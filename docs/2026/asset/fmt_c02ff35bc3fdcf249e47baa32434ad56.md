
`int` `ip_output(` `struct` `net *net,` `struct` `sock *sk,` `struct` `sk_buff *skb)`  `{`  `struct` `net_device *dev = skb_dst(skb)->dev, *indev = skb->dev;`    `IP_UPD_PO_STATS(net, IPSTATS_MIB_OUT, skb->len);`    `skb->dev = dev;`  `skb->protocol = htons(ETH_P_IP);`    `return` `NF_HOOK_COND(NFPROTO_IPV4, NF_INET_POST_ROUTING,`  `net, sk, skb, indev, dev,`  `ip_finish_output,`  `!(IPCB(skb)->flags & IPSKB_REROUTED));`  `}`  `EXPORT_SYMBOL(ip_output);`

1. 此时如果你发送的目的地址环回地址，则会触发软中断，软中断调用`__netif_receive_skb_one_core`来触发，经过一系列调用链条最终调用`ip_rcv()`

2. `ip_rcv()`首先调用`ip_rcv_core()`处理接收到的skb,然后触发`NF_INET_PRE_ROUTING`hook, 在一些相应的hook函数完毕后调用回调函数`ip_rcv_finish()`

```
 /*  * IP receive entry point  */  int ip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt,  struct net_device *orig_dev)  {  struct net *net = dev_net(dev);    skb = ip_rcv_core(skb, net);  if (skb == NULL)  return NET_RX_DROP;    return NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING,  net, NULL, skb, dev, NULL,  ip_rcv_finish);  }
```

1. `ip_rcv_finish()`调用函数`ip_rcv_finish_core()`函数，主要判断目的地址的规范性一类然后来判断是否需要丢弃

2. 如果不需要丢弃，则调用`dst_input()`

3. `dst_input()`会通过间接调用到`ip_local_deliver()`,其首先判断该ip包是否是分片包，如果是则调用`ip_defrag()`,否则触发钩子`NF_INET_LOCAL_IN`

```
 /*  *  Deliver IP Packets to the higher protocol layers.  */  int ip_local_deliver(struct sk_buff *skb)  {  /*  *  Reassemble IP fragments.  */  struct net *net = dev_net(skb->dev);    if (ip_is_fragment(ip_hdr(skb))) {  if (ip_defrag(net, skb, IP_DEFRAG_LOCAL_DELIVER))  return 0;  }    return NF_HOOK(NFPROTO_IPV4, NF_INET_LOCAL_IN,  net, NULL, skb, skb->dev, NULL,  ip_local_deliver_finish);  }  EXPORT_SYMBOL(ip_local_deliver);
```

1. `ip_defrag()`该函数用来处理分片包,该函数会调用`ip_find()`来查找是否已经有同一个IP包的分片组成的队列，如果有则返回队列，否则创建新的队列

2. `ip_defrag()`继续调用`ip_frag_queue()`来真正处理该分片的信息

3. `ip_frag_queue()`来查看该ip头部的字段长度是否安全，如果出现错误则会直接丢弃该skb并且释放整个队列

4. `ip_frag_queue()`会检查几个条件来判断是否ip包是否全部分片均到达，如果到达则调用`ip_frag_reasm()`来重组新的skb,然后将skb重新注入到网络协议栈的接收路径

5. 之后通过返回0，返回到`ip_local_deliver()`然后触发钩子`NF_INET_LOCAL_IN`,之后继续调用回调函数

所以，如果我们发送一个数据包分片的时候，经过的netfilter hook点顺序可能是

`NF_INET_LOCAL_OUT -> NF_INET_POST_ROUTING -> NF_INET_PRE_ROUTING -> NF_INET_LOCAL_IN`

# 漏洞原因

漏洞出自于`nf_hook_slow()`

```
int nf_hook_slow(struct sk_buff *skb, struct nf_hook_state *state,  const struct nf_hook_entries *e, unsigned int s)  {  unsigned int verdict;  int ret;    for (; s < e->num_hook_entries; s++) {  verdict = nf_hook_entry_hookfn(&e->hooks[s], skb, state);  switch (verdict & NF_VERDICT_MASK) {  case NF_ACCEPT:  break;  case NF_DROP:  kfree_skb_reason(skb,  SKB_DROP_REASON_NETFILTER_DROP);  ret = NF_DROP_GETERR(verdict);  if (ret == 0)  ret = -EPERM;  return ret;  case NF_QUEUE:  ret = nf_queue(skb, state, s, verdict);  if (ret == 1)  continue;  return ret;  default:  /* Implicit handling for NF_STOLEN, as well as any other  * non conventional verdicts.  */  return 0;  }  }    return 1;  }  EXPORT_SYMBOL(nf_hook_slow);
```

在switch case 语句中`NF_DROP`的选项,这是由于当用户所设计的netfilter hook函数最后返回`NF_DROP`时即将执行的程序流程

这里的流程将会首先释放这个skb，但是这里程序再次判断了`NF_DROP_GETERR`来获取返回参数，

这就给了用户一定的操作空间，如果可以通过这个函数将ret修改为NF\_ACCEPT, 也就是1,

那么当这个函数返回后将回到下面的`nf_hook()`函数

`nf_hook()`函数返回1则会导致调用后续的`okfn()`,这里使用`ip_forward()`进行举例

该函数将在最后调用`NF_HOOK()`函数

```
...  return NF_HOOK(NFPROTO_IPV4, NF_INET_FORWARD,  net, NULL, skb, skb->dev, rt->dst.dev,  ip_forward_finish);  ...
```

而`NF_HOOK()`函数将会根据`nf_hook()`函数的返回值来决定是否调用`okfn`参数指针所指向的地址

返回值为1则说明hook函数处理的返回值为`NF_ACCEPT`

```
static inline int  NF_HOOK(uint8_t pf, unsigned int hook, struct net *net, struct sock *sk, struct sk_buff *skb,  struct net_device *in, struct net_device *out,  int (*okfn)(struct net *, struct sock *, struct sk_buff *))  {  int ret = nf_hook(pf, hook, net, sk, skb, in, out, okfn);  if (ret == 1)  ret = okfn(net, sk, skb);  return ret;  }
```

那么就能在之后对skb进行二次释放,因此导致了double free

# 触发方式

知道了漏洞出处，接下来的步骤便是尝试触发，在`nf_hook_slow()`函数当中

```
int nf_hook_slow(struct sk_buff *skb, struct nf_hook_state *state,  const struct nf_hook_entries *e, unsigned int s)  {  ...  for (; s < e->num_hook_entries; s++) {  verdict = nf_hook_entry_hookfn(&e->hooks[s], skb, state);  switch (verdict & NF_VERDICT_MASK) {  case NF_ACCEPT:  break;  case NF_DROP:  kfree_skb_reason(skb,  SKB_DROP_REASON_NETFILTER_DROP);  ret = NF_DROP_GETERR(verdict);  ...  
```

这里的for循环是调用当前hook点里面所有的hook,并且返回值是可以由用户构造的

所以为了满足我们的目的，需要verdict满足:

```
static inline int NF_DROP_GETERR(int verdict)  {  return -(verdict >> NF_VERDICT_QBITS);  }    #define NF_DROP 0  #define NF_ACCEPT 1  #define NF_VERDICT_MASK 0x000000ff  #define NF_VERDICT_QBITS 16
```

```
1. verdict & NF_VERDICT_MASK == NF_DROP;  2. NF_DROP_GETERR(verdict) == NF_ACCEPT;
```

因此计算得出可以构造`verdict = 0xffff0000`即可满足这样一个条件,而构造verdict我们可以通过向内核传递在用户态构造的netfilter tables-chains-rules来达成，这一过程可以通过netlink来完成传递。

# 相关技巧

## Page Double Free

从`__free_pages()`函数可以看出

```
void __free_pages(struct page *page, unsigned int order)  {  /* get PageHead before we drop reference */  int head = PageHead(page);    if (put_page_testzero(page))  free_the_page(page, order);  else if (!head)  while (order-- > 0)  free_the_page(page + (1 << order), order);  }  EXPORT_SYMBOL(__free_pages);
```

当使用`put_page_testzero()`发现页面的引用计数为1,则将释放他

而如果此时我们不做任何事直接将他应用在double free上，则`put_page_testzero()`将会发现`page.refcount == 0`从而报错

```
static inline int put_page_testzero(struct page *page)  {  VM_BUG_ON_PAGE(page_ref_count(page) == 0, page);  return page_ref_dec_and_test(page);  }
```

因此可以在double free中途插入重分配，使得refcount++,这样就可以绕过这部分检查

作者的例子如下:

```
static void kref_juggling(void)  {  struct page *skb1, *pmd, *pud;    skb1 = alloc_page(GFP_KERNEL); // refcount 0 -> 1  __free_page(skb1); // refcount 1 -> 0  pmd = alloc_page(GFP_KERNEL); // refcount 0 -> 1  __free_page(skb1); // refcount 1 -> 0  pud = alloc_page(GFP_KERNEL); // refcount 0 -> 1    pr_err("[*] skb1: %px (phys: %016llx), pmd: %px (phys: %016llx), pud: %px (phys: %016llx)\n", skb1, page_to_phys(skb1), pmd, page_to_phys(pmd), pud, page_to_phys(pud));  }
```

## Order 4 -> Order 0

在`__do_kmalloc_node()`函数当中，

```
#define KMALLOC_MAX_CACHE_SIZE  (1UL << KMALLOC_SHIFT_HIGH)  #define KMALLOC_SHIFT_HIGH  (PAGE_SHIFT + 1)  #define PAGE_SHIFT      12    static __always_inline  void *__do_kmalloc_node(size_t size, gfp_t flags, int node, unsigned long caller)  {  ...  if (unlikely(size > KMALLOC_MAX_CACHE_SIZE)) {  ret = __kmalloc_large_node(size, flags, node);  ...  ret = __kmem_cache_alloc_node(s, flags, node, size, caller);  ...  }
```

在使用`kmalloc`分配内存堆块的过程中，

如果分配size大小大于`KMALLOC_MAX_CACHE_SIZE`

那么将会使用`alloc_page()`直接从伙伴系统申请页面，kfree也是同理
