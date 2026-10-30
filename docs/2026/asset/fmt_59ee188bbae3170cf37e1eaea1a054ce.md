
![图 5-1 构造漏洞数据包](https://bbs.kanxue.com/upload/attach/202105/741085_A5AQZBQCPVFE5C7.png)

实际发现`rsp`这块内存之前保存了两个位于内核代码段的地址，第一个长字的低2字节被设置为`0x42`和`0x01`，第二个长字则原封不动包含在`rsp`里面被返回给攻击者。

![图 5-2](https://bbs.kanxue.com/upload/attach/202105/741085_QRAGQ8ZM7QX62MK.png)

攻击者拿到响应数据后解析出`rsp+0x8`即为内核代码段地址。低端对齐后减去`0xffff_ffff_8100_0000`得到KASLR偏移，该偏移用于计算后续rop gadget的实际地址。

![图5-3 栈变量泄漏](https://bbs.kanxue.com/upload/attach/202105/741085_PF942V3E8HJVRQ8.png)

# 六、泄漏内核堆地址

若在泄漏内核代码段地址之前，先发送`cid=1`的信道控制包试图将a2mp channel设置为ERTM模式，再次发送图5-1所示漏洞数据包，则会在`rsp+0x8`的位置留下一个位于内核堆区域的指针。内核将`0xffff_8800_0000_0000 - 0xffff_c7ff_ffff_ffff`这段虚拟地址用于物理内存直接映射，使用`kmalloc`分配的小块内存位于此区域。

![图 6-1 遗留的堆指针](https://bbs.kanxue.com/upload/attach/202105/741085_PCVBT55GDCTNQ2D.png)

拿到堆地址就可以根据堆块之间的相对偏移计算出任意一个堆块的地址。如果能在已知地址的堆块上布置fake `sk_filter`、fake `bpf_prog`和rop链就可以控制程序流。

这里将目标瞄准`l2cap_chan`结构。当`cid=3`会创建a2mp channel，

```c
//代码6-1 linux-5.4\net\bluetooth\l2cap_core.c
static void l2cap_data_channel(struct l2cap_conn *conn, u16 cid, struct sk_buff *skb)
{
    struct l2cap_chan *chan;
    chan = l2cap_get_chan_by_scid(conn, cid); //寻找对应的进程
    if (!chan) {
        if (cid == L2CAP_CID_A2MP) {
            chan = a2mp_channel_create(conn, skb);
            ……
        }
    }
}
```

`a2mp_channel_create -> amp_mgr_create -> a2mp_channel_open -> l2cap_chan_create`，最终调用`kzalloc`分配堆块用于保存a2mp channel。

```c
//代码6-2 linux-5.4\net\bluetooth\l2cap_core.c
struct l2cap_chan *l2cap_chan_create(void)
{
    struct l2cap_chan *chan;

    chan = kzalloc(sizeof(*chan), GFP_ATOMIC);
    if (!chan)
        return NULL;
    ……
}
```

a2mp channel大小为0x318，slab系统实际分配大小为0x400的堆块。

![图 6-2 l2cap_chan结构](https://bbs.kanxue.com/upload/attach/202105/741085_59R6D8PF4XJQBMA.png)

a2mp channel地址为`0xffff_8881_cdfd_d000`，栈上遗留堆指针为`0xffff_8881_cdfd_d110`，遗留指针减去固定偏移`0x110`即为a2mp channel地址。

![图 6-3](https://bbs.kanxue.com/upload/attach/202105/741085_SBV7FHUZB5YNH55.png)

# 七、释放后重引用

攻击者可以向目标机发送`cid=1`的信道控制包断开a2mp连接，a2mp channel会被释放。`disconn_req.disconn_req.dcid=AMP_MGR_CID`（3），`l2cap_disconnect_req`根据`dcid`找到要释放的channel。

![图 7-1](https://bbs.kanxue.com/upload/attach/202105/741085_GFPDYS4R4FQJ32N.png)

函数调用路径为`l2cap_sig_channel -> l2cap_bredr_sig_cmd -> l2cap_disconnect_req -> l2cap_chan_put -> l2cap_chan_destroy`，最终调用`kfree`释放a2mp channel。

```c
//代码7-1 linux-5.4\net\bluetooth\l2cap_core.c
static void l2cap_chan_destroy(struct kref *kref)
{
    struct l2cap_chan *chan = container_of(kref, struct l2cap_chan, kref);
    write_lock(&chan_list_lock);
    list_del(&chan->global_l);
    write_unlock(&chan_list_lock);

    kfree(chan);
}
```

现在得到一个地址已知的空闲堆块，下面获取该堆块并向里面写入fake `sk_filter`、fake `bpf_prog`和rop链。基本思路是试图在目标机中分配得到该堆块并可以向里面写入任意数据。

通过搜索堆分配函数发现了`a2mp_getampassoc_rsp()`函数提供了这个功能。

```c
//代码7-2 linux-5.4\net\bluetooth\a2mp.c
static int a2mp_getampassoc_rsp(struct amp_mgr *mgr, struct sk_buff *skb, struct a2mp_cmd *hdr)
{
    struct a2mp_amp_assoc_rsp *rsp = (void *) skb->data;
    u16 len = le16_to_cpu(hdr->len); //rsp+assoc_data
    struct amp_ctrl *ctrl;
    size_t assoc_len;

    assoc_len = len - sizeof(*rsp); //assoc_data长度

    if (rsp->status) //rsp->status字段保持为0
        return -EINVAL;
    ctrl = amp_ctrl_lookup(mgr, rsp->id);
    if (ctrl) {
        u8 *assoc;
        assoc = kmemdup(rsp->amp_assoc, assoc_len, GFP_KERNEL); //分配堆块
        if (!assoc) {
            amp_ctrl_put(ctrl);
            return -ENOMEM;
        }
        ctrl->assoc = assoc;
        ctrl->assoc_len = assoc_len;
        ctrl->assoc_rem_len = assoc_len;
        ctrl->assoc_len_so_far = 0;
        amp_ctrl_put(ctrl);
    }
}
```

调用路径为`l2cap_data_channel -> a2mp_chan_recv_cb -> a2mp_getampassoc_rsp`，根据图4-1可知要到达这里需要重建a2mp channel并设置channel为BASIC模式。该函数将数据包中`amp_assoc`部分复制到堆上。攻击机可以构造发送携带0x400字节数据的`assoc_rsp`数据包，有一定的概率会分配到刚释放的堆块上。

![图 7-2](https://bbs.kanxue.com/upload/attach/202105/741085_DZBTQHFFSCYWWGM.png)

代码7-2中`a2mp_getampassoc_rsp`调用`kmemdup`分配堆块必须要`amp_ctrl_lookup`返回有效结果。`amp_ctrl_lookup`在`mgr->amp_ctrls`链表中搜索并返回`id`值为`rsp->id`的ctrl控制结构，因此必然有一处要向`mgr->amp_ctrls`里面添加ctrl。

`a2mp_getinfo_rsp`调用`amp_ctrl_add`向`mgr->amp_ctrls`链表中添加攻击者构造的特定`id`的ctrl结构。调用路径为`l2cap_data_channel -> a2mp_chan_recv_cb -> a2mp_getinfo_rsp`。

```c
//代码7-3 linux-5.4\net\bluetooth\a2mp.c
static int a2mp_getinfo_rsp(struct amp_mgr *mgr, struct sk_buff *skb, struct a2mp_cmd *hdr)
{
    struct a2mp_info_rsp *rsp = (struct a2mp_info_rsp *) skb->data;
    struct a2mp_amp_assoc_req req;
    struct amp_ctrl *ctrl;

    if (rsp->status)
        return -EINVAL;

    ctrl = amp_ctrl_add(mgr, rsp->id);
}
```

攻击机发送info rsp数据包，其中`info_rsp.id`字段可以是任意整型值，目标机会将对应`id`的ctrl结构添加到链表中，`a2mp_getampassoc_rsp()`就可以顺利调用`kmemdup`。

![图 7-3](https://bbs.kanxue.com/upload/attach/202105/741085_4V3G6AQH5Y4G97T.png)

## 1、构造堆喷射

`a2mp_getampassoc_rsp()`可以分配任意大小且内容可控的堆块，但是分配堆块的位置是不可预知的，也就是不一定分配到已知地址的堆块上。且内核堆是多线程的，无法通过排列组合构造堆布局。这种情况下可以使用堆喷射技术向堆上重复喷洒大量数据以期能够将载荷喷洒到已知地址上，最直接的做法就是分配大量的堆块并重复写入大量数据。利用`a2mp_getampassoc_rsp()`分配任意大小且内容可控的特点构造堆喷射。

要注意的是ACL数据的最大传输单元为1021字节，也就是每次传输的L2CAP数据分组不超过1021字节，对于要喷射0x400（1024字节）大小的载荷需要分组传输。

![图 7-4](https://bbs.kanxue.com/upload/attach/202105/741085_HW4MXH4HH5ZSNKM.png)

若hci头部flag字段为ACL_START（2）表示hci的载荷为l2cap第一个分组。若l2cap头部len字段等于当前载荷长度，说明只有l2cap只有一个分组且被正确接收，直接交给l2cap层处理；若len字段大于当前载荷长度说明有多个l2cap分组，调用`bt_skb_alloc`分配缓冲区，等所有分组接收组装完毕后交给l2cap处理。

同时要注意a2mp channel缓冲区默认大小为`L2CAP_DEFAULT_MIN_MTU`（670字节），也就是l2cap携带的amp载荷不超过670字节。

```c
//代码7-4 linux-5.4\net\bluetooth\a2mp.c
static struct l2cap_chan *a2mp_chan_open(struct l2cap_conn *conn, bool locked)
{
    struct l2cap_chan *chan;
    chan = l2cap_chan_create();
    ……
    chan->imtu = L2CAP_A2MP_DEFAULT_MTU;
    ……
}
```

对于喷射0x400大小的堆块，amp载荷其实不必大于670字节，因为slab会为介于512-1024之间的数据分配0x400的堆块，但是为了后面调试方便观察堆数据的布局，我们还是将0x400填满确保堆块之间没有空隙。

```c
//代码7-5 linux-5.4\net\bluetooth\l2cap_core.c
static void l2cap_data_channel(struct l2cap_conn *conn, u16 cid, struct sk_buff *skb)
{
    switch (chan->mode) {
        ……
    case L2CAP_MODE_BASIC:
        if (chan->imtu < skb->len) { //检查amp包长度是否大于mtu
            goto drop;
        }
        if (!chan->ops->recv(chan, skb)) //调用a2mp_chan_recv_cb
            goto done;
}
```

为绕过AMP MTU和ACL MTU限制，要在建立a2mp channel之后紧跟着发送信道控制包（`cid=1`）修改AMP MTU为0xffff，并且每次发送的l2cap分组控制在ACL MTU范围内即可。

![图 7-5](https://bbs.kanxue.com/upload/attach/202105/741085_UV3ZDHR376RYANF.png)

## 2、第一次堆喷射

向内核堆喷洒大量0x400堆块以覆盖刚刚释放的a2mp channel结构。

![图 7-6](https://bbs.kanxue.com/upload/attach/202105/741085_ZFEDBPATA992X73.png)

由于slab空闲链表遵循先进后出的原则（队列），可以看到刚刚被释放的a2mp channel又被重建的a2mp channel占用，但这个没有影响。slab不像glibc堆管理器那样，slab不使用堆块头部的元数据管理堆块，因此内核堆块之间是没有空洞的，a2mp channel + 0x400即是下一个堆块，可以看到被填充了标记字节。

由于旧的a2mp channel极有可能被再次占用，所以选择偏移0x400的位置也就是紧邻的下一个堆块保存fake `bpf_prog`和ROP链。

## 3、构造fake `sk_filter`、`bpf_prog`和ROP链

验证堆喷成功之后，将部分标记字节替换成fake `sk_filter`、fake `bpf_prog`和ROP链。

fake `sk_filter`位于堆块偏移0x300处，`sk_filter->prog`指向`0xffff_8882_090d_ef20`即为

![图 7-7](https://bbs.kanxue.com/upload/attach/202105/741085_MZYFYMDZEQJXEXM.png)

fake `bpf_prog`的起始地址。`bpf_prog->bpf_func`保存指向如下指令片段的地址，该地址由第5节泄漏的内核代码段地址计算而来。

```asm
0xffffffff8155528d        push rsi
add byte ptr [rbx + 0x41], bl
pop rsp
pop rbp
ret
```
