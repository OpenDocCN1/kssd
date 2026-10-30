
```c
skb = sock_alloc_send_skb(sk,
        alloclen + hh_len + 15,
        (flags & MSG_DONTWAIT), &err);
} else {
    //如果不是第一个分片
    skb = NULL;
    if (atomic_read(&sk->sk_wmem_alloc) <=
        2 * sk->sk_sndbuf)
        skb = sock_wmalloc(sk,
                alloclen + hh_len + 15, 1,
                sk->sk_allocation);
    if (unlikely(!skb))
        err = -ENOBUFS;
}
if (!skb)
    goto error; //分配失败跳转到error
```

当分配成功后，首先初始化`skb`中的一些控制数据

```c
/*
 * Fill in the control structures
 */
skb->ip_summed = csummode;
skb->csum = 0;
skb_reserve(skb, hh_len);
```

接下来初始化时间戳

```c
/* only the initial fragment is time stamped */
skb_shinfo(skb)->tx_flags = cork->tx_flags;
cork->tx_flags = 0;
skb_shinfo(skb)->tskey = tskey;
tskey = 0;
```

```c
/*
 * Find where to start putting bytes.
 */
data = skb_put(skb, fraglen + exthdrlen);//预留L2，L3首部空间
skb_set_network_header(skb, exthdrlen); //设置L3层的指针
skb->transport_header = (skb->network_header +
        fragheaderlen);
data += fragheaderlen + exthdrlen;

if (fraggap) { //填充原来skb的尾部
    skb->csum = skb_copy_and_csum_bits(
            skb_prev, maxfraglen,
            data + transhdrlen, fraggap, 0);
    // skb_copy_and_csum_bits函数将数据从第一个创建的sk_buff复制到新分配的sk_buff
    skb_prev->csum = csum_sub(skb_prev->csum,
            skb->csum);
    data += fraggap;
    pskb_trim_unique(skb_prev, maxfraglen);
}
copy = datalen - transhdrlen - fraggap;
if (copy > 0 && getfrag(from, data + transhdrlen, offset, copy, fraggap, skb) < 0) {
    err = -EFAULT;
    kfree_skb(skb);
    goto error;
}
offset += copy;
length -= datalen - fraggap;
transhdrlen = 0;
exthdrlen = 0;
csummode = CHECKSUM_NONE;
/*
 * Put the packet on the pending queue.
 */
__skb_queue_tail(queue, skb); //把新skb插入skb队列队尾
continue;
}
```

在这里可能产生两种情况：
（1）分配的新`skb->len`刚好是MTU
![](upload/attach/202101/876323_RV6KCT2F3DMB97V.jpg)
（2）分配的新`skb->len`刚好是确切的大小（小于mtu）
![](upload/attach/202101/876323_9HK5J4D37HJZH3X.jpg)

在这里我们只讨论最复杂的这种情况（并且也是跟我们漏洞最相关的这种情况），其余的更加细节的可以看：
https://blog.csdn.net/minghe_uestc/article/details/7836920?utm_source=blogxgwz2

# 漏洞分析与利用

### poc/exp

来自国外某带师傅
![](upload/attach/202101/876323_PB84GGEUURYZ592.jpg)

### poc/exp分析

最主要的其实就是 **两次`send`**

```c
#define SHINFO_OFFSET 3164
int size = SHINFO_OFFSET + sizeof(struct skb_shared_info);
int rv = send(s, buffer, size, MSG_MORE);
int val = 1;
rv = setsockopt(s, SOL_SOCKET, SO_NO_CHECK, &val, sizeof(val));
send(s, buffer, 1, 0); //第二次send的size为1
close(s);
```

- `SOCK_DGRAM` 代表了UDP
- `AF_INET` 代表TCP/IP协议族，在socket编程中只能是`AF_INET`。
- `s_addr` 代表ip地址，`INADDR_LOOPBACK`代表绑定地址LOOPBAC, 往往是127.0.0.1, 只能收到127.0.0.1上面的连接请求。`htons`将其转换成一个转换成网络数据格式的数字。

当我们建立好`socket`并初始化之后，**第一次`send`，带上标记为`MSG_MORE`告诉系统我们接下来还有数据要发送。此时走UFO路径**

如果要发送的是UDP数据包，且系统支持UFO，并且需要分片（`length > mtu`），那么`send()`最终会进入：`ip_ufo_append_data`

![](upload/attach/202101/876323_CMNK44JKTJM8ZQC.jpg)

源码如下：

```c
static inline int ip_ufo_append_data(struct sock *sk,
        struct sk_buff_head *queue,
        int getfrag(void *from, char *to, int offset, int len,
            int odd, struct sk_buff *skb),
        void *from, int length, int hh_len, int fragheaderlen,
        int transhdrlen, int maxfraglen, unsigned int flags)
{
    struct sk_buff *skb;
    int err;
    /* There is support for UDP fragmentation offload by network
     * device, so create one single skb packet containing complete
     * udp datagram
     */
    skb = skb_peek_tail(queue); //取skb队列的队尾
    if (!skb) {
        skb = sock_alloc_send_skb(sk,
                hh_len + fragheaderlen + transhdrlen + 20,
                (flags & MSG_DONTWAIT), &err);
        if (!skb)
            return err;
        /* reserve space for Hardware header */
        skb_reserve(skb, hh_len);
        /* create space for UDP/IP header */
        skb_put(skb, fragheaderlen + transhdrlen);
        /* initialize network header pointer */
        skb_reset_network_header(skb);
        /* initialize protocol header pointer */
        skb->transport_header = skb->network_header + fragheaderlen;
        skb->csum = 0;
        __skb_queue_tail(queue, skb);
    } else if (skb_is_gso(skb)) {
        goto append;
    }
    skb->ip_summed = CHECKSUM_PARTIAL;
    /* specify the length of each IP datagram fragment */
    skb_shinfo(skb)->gso_size = maxfraglen - fragheaderlen;
    skb_shinfo(skb)->gso_type = SKB_GSO_UDP;
append:
    return skb_append_datato_frags(sk, skb, getfrag, from,
            (length - transhdrlen));
}
```

调用`sock_alloc_send_skb`分配一个新的`skb`，然后把数据放到新的`skb`的非线性区域中。（`skb_share_info`）
![](upload/attach/202101/876323_ME34M3FYCVAAW3K.jpg)

其结构体定义如下：

```c
struct skb_shared_info {
    unsigned char nr_frags;
    __u8 tx_flags;
    unsigned short gso_size;
    /* Warning: this field is not always filled in (UFO)! */
    unsigned short gso_segs;
    unsigned short gso_type;
    struct sk_buff *frag_list;
    struct skb_shared_hwtstamps hwtstamps;
    u32 tskey;
    __be32 ip6_frag_id;
    /*
     * Warning : all fields before dataref are cleared in __alloc_skb()
     */
    atomic_t dataref;
    /* Intermediate layers must ensure that destructor_arg
     * remains valid until skb destructor
     */
    void * destructor_arg;
    /* must be last field, see pskb_expand_head() */
    skb_frag_t frags[MAX_SKB_FRAGS];
};
```

最后新的`skb`入队。

**注意在本条UFO路径中我们`skb`中数据的大小是大于`mtu`的！**

更细节的：
![](upload/attach/202101/876323_2C4SD9KTEMKSW9K.jpg)

通过`skb_shinfo(SKB)`宏也可以看出来`skb_shared_info`与`skb`之间的关系

```c
#define skb_shinfo(SKB) ((struct skb_shared_info *)(skb_end_pointer(SKB)))
```

在第二次`send`之前，我们调用`setsockopt`来设置了`SO_NO_CHECK`标志，即不校验checksum。（**内核是通过`SO_NO_CHECK`的标志来判断用UFO机制还是non-UFO机制，这一点在刚刚的源码中并不明显。请直接看下面漏洞补丁那里的patch**）

造成接下来 **第二次`send`的时候越过UFO路径而执行non-UFO的代码。**

在non-UFO中，此时计算的`copy = mtu - skb_len`小于0，此时的`skb`是直接从`skb`队尾取出来的，也就是第一次`send`时new alloc出来的`len > mtu`的`skb`。

由于`copy < 0`，那么在 **non-UFO** 路径上触发了重新分配`skb`的操作。

而在重新分配结束后调用了：

```c
copy = mtu - skb->len;
if(copy<0){
    char *data;
    unsigned int datalen;
    unsigned int fraglen;
    unsigned int fraggap;
    unsigned int alloclen;
    struct sk_buff *skb_prev = skb;
    fraggap = skb_prev->len - maxfraglen;//这里其实就是负的copy
    datalen = length + fraggap;
    skb_copy_and_csum_bits(skb_prev, maxfraglen,data + transhdrlen, fraggap, 0);
}
```

**其中的`skb_copy_and_csum_bits`将旧的`skb_prev`中的数据（UFO路径中的`skb`）复制到新分配的`sk_buff`中（即`skb_shared_info->frags[]`中的`page_frag`），从而造成溢出。**

而对于`skb_shared_info`存在一个成员`void * destructor_arg`，他是`skb`释放时的在`kfree_skb`中底层对于其产生的一个析构函数的调用。
![](upload/attach/202101/876323_P2NJRMMM2AVQ4NU.jpg)

这里很有意思的一个处理，直接将一个`void *`赋给一个`ubuf_info *`类型。

```c
struct ubuf_info {
    void (*callback)(struct ubuf_info *, bool zerocopy_success);
    void *ctx;
    unsigned long desc;
};
```

其作用是：
- 当完成了`skb` DMA时，通过他调用回调函数做析构，释放缓冲区。并且此时`skb`引用计数为0。
- 而`ctx`负责跟踪设备上下文。`desc`负责跟踪用户空间的缓冲区索引。
- `zerocopy_success`代表是否发生 [零拷贝](https://zhuanlan.zhihu.com/p/85571977)

至此，通过覆盖`skb_shared_info.destructor_arg`就可以实现程序流劫持了。具体的打法有很多：

1.如果不开smep/kaslr，那么直接ret2usr即可。

2.如果开了smep，可以做内核rop先劫持cr4来关闭smep。然后jmp到payload
