
| 条件 | 模式 | 说明 |
| --- | --- | --- |
| `<0` | `SEARCH_LESSEQUAL` | 找到一个`msg->m_type`小于`msgtyp`且`msg->m_type`**最小的**`msg_msg` |
| `=0` | `SEARCH_ANY` | 找到`msg_msg`队列中**第一个**`msg_msg` |
| `>0` | `SEARCH_EQUAL` \| `SEARCH_NOTEQUAL` | 找到**第一个**`msg->m_type`等于/不等于`msgtyp`的`msg_msg` |

**特例：`MSG_COPY`位为1的时候，`mode`为`SEARCH-NUMBER`,在`find_msg`中会返回`msg_msg`双向循环链表中，第`msgtyp`个`msg_msg`，也就是返回第`msgtyp`条消息，而不是上述表格中根据`msgtyp`去和`msg->m_type`进行匹配。**

#### ②链表unlink && 释放结构体

在`/ipc/msg.c`中`do_msgrcv`：

```c
static long do_msgrcv(int msqid, void __user *buf, size_t bufsz, long msgtyp, int msgflg,
                      long (*msg_handler)(void __user *, struct msg_msg *, size_t))
{
    int mode;
    struct msg_queue *msq;
    struct ipc_namespace *ns;
    struct msg_msg *msg, *copy = NULL;
    ...........
    ...........
    list_del(&msg->m_list);
    ...........
    ...........
    free_msg(msg);

    return bufsz;
}
```

内核首先会调用 `list_del()` 将其从 `msg_queue` 的双向链表上 unlink，之后再调用 `free_msg()` 释放 `msg_msg` 单向链表上的所有消息。

#### ③内核-->用户态的消息拷贝

在`do_msg_rcv`函数最后，调用了`msg_handler`，看参数像是进行内核-->用户的数据拷贝。

```c
bufsz = msg_handler(buf, msg, bufsz);
```

其中`msg_handler`是`do_msgrcv`传进来的参数，是一个函数指针，向上看调用`do_msgrcv`的调用链：

```c
long ksys_msgrcv(int msqid, struct msgbuf __user *msgp, size_t msgsz,
                 long msgtyp, int msgflg)
{
    return do_msgrcv(msqid, msgp, msgsz, msgtyp, msgflg, do_msg_fill);
}
```

可知`msg_handler`具体函数指针为`do_msg_fill`：

```c
static long do_msg_fill(void __user *dest, struct msg_msg *msg, size_t bufsz)
{
    struct msgbuf __user *msgp = dest;
    size_t msgsz;

    if (put_user(msg->m_type, &msgp->mtype))
        return -EFAULT;

    msgsz = (bufsz > msg->m_ts) ? msg->m_ts : bufsz;
    if (store_msg(msgp->mtext, msg, msgsz))
        return -EFAULT;
    return msgsz;
}
```

其中调用`store_msg`进行数据拷贝：

```c
int store_msg(void __user *dest, struct msg_msg *msg, size_t len)
{
    size_t alen;
    struct msg_msgseg *seg;

    alen = min(len, DATALEN_MSG);
    if (copy_to_user(dest, msg + 1, alen))
        return -1;

    for (seg = msg->next; seg != NULL; seg = seg->next) {
        len -= alen;
        dest = (char __user *)dest + alen;
        alen = min(len, DATALEN_SEG);
        if (copy_to_user(dest, seg + 1, alen))
            return -1;
    }
    return 0;
}
```

可以看到拷贝过程和之前`msg_msg`结构内存申请相对应：

- 若拷贝数据总长度小于`DATALEN_MSG`，则直接拷贝后结束。

- 若拷贝数据总长度小于`DATALEN_MSG`，则继续拷贝`msg_msg`单向链表后面的`msg_msgseg`结构体内容，直到拷贝结束。

**ps：拷贝结束的标志均为`seg->next`指针为NULL**

拷贝的总长度则由`msgsz`决定，而`msgsz`：

```c
msgsz = (bufsz > msg->m_ts) ? msg->m_ts : bufsz;
```

可以看到若`bufsz`足够的情况下，**拷贝数据总长度是由`msg->m_ts`决定的。**

#### ④特例：`MSG_COPY`位

##### not unlink

**需要注意的是，若我们带有`MSG_COPY`标志，则不会在双向链表上unlink，只会进行copy操作**，具体实现在`do_msgrcv`中部分源码：

```c
if (!IS_ERR(msg)) {
    /*
     * Found a suitable message.
     * Unlink it from the queue.
     */
    if ((bufsz < msg->m_ts) && !(msgflg & MSG_NOERROR)) {
        msg = ERR_PTR(-E2BIG);
        goto out_unlock0;
    }
    /*
     * If we are copying, then do not unlink message and do
     * not update queue parameters.
     */
    if (msgflg & MSG_COPY) {
        msg = copy_msg(msg, copy);
        goto out_unlock0;
    }

    list_del(&msg->m_list);
    msq->q_qnum--;
    msq->q_rtime = ktime_get_real_seconds();
    ipc_update_pid(&msq->q_lrpid, task_tgid(current));
    msq->q_cbytes -= msg->m_ts;
    atomic_sub(msg->m_ts, &ns->msg_bytes);
    atomic_dec(&ns->msg_hdrs);
    ss_wakeup(msq, &wake_q, false);

    goto out_unlock0;
}
```

##### `msgtyp`意义转变

在通过 `find_msg` 定位一节已经讲过：

`MSG_COPY`位为1的时候，`mode`为`SEARCH-NUMBER`,在`find_msg`中会返回`msg_msg`双向循环链表中，第`msgtyp`个`msg_msg`，也就是返回第`msgtyp`条消息，而不是上述表格中根据`msgtyp`去和`msg->m_type`进行匹配。

##### `msgsz`检测

由于`MSG_COPY`位为1的时候，内核会调用`prepare_copy`再申请一块内存出来。

```c
static inline struct msg_msg *prepare_copy(void __user *buf, size_t bufsz)
{
    struct msg_msg *copy;

    /*
     * Create dummy message to copy real message to.
     */
    copy = load_msg(buf, bufsz);
    if (!IS_ERR(copy))
        copy->m_ts = bufsz;
    return copy;
}
```

申请内存大小为我们传入`do_msgrcv`的`bufsz`。

两个`msg_msg`之间的拷贝则由`copy_msg`负责，而在`copy_msg`函数中有一段代码如下：

```c
if (src->m_ts > dst->m_ts)
    return ERR_PTR(-EINVAL);
```

若源`src->m_ts`大于目标`dst->m_ts`,则会发生溢出，因此会直接返回不会拷贝。

同时`copy_msg`函数末尾还有赋值操作：

```c
dst->m_ts = src->m_ts;
```

因此我们要满足的条件是 `src->m_ts <= dst->m_ts` 即可。

即 `bufsz >= src->m_ts`。

## 具体利用

### ①地址泄露

经过对源码的阅读和分析，可以想到我们可以做到的事情：

(1)改掉`msg_msg->m_ts`。

可以读取最多一页的内存，**实现越界读。**

1. 若单项链表只有`msg_msg`，则可以读取该`msg_msg`附近的数据（最多将近一页内存）。

2. 若单向链表中含有`msg_msgseg`，则可以读取单向链表尾节点`msg_msgseg`中附近的数据（最多将近一页内存）。

(2)改掉`msg_msg->m_ts`和`msg_msg->m_list`中的`next`指针。

**可以利用堆喷其他结构体+`msg_msg`越界读，获得一些堆地址 or 内核地址。**

可以堆喷一些消息队列，每个消息队列上只有一条消息：即`msg->queue`双向循环链表里只有一个节点：

![](https://bbs.kanxue.com/upload/attach/202402/959503_RVGSH489CJGAASF.jpg)

可以通过某个`msg_msg`的越界读，有几率读到其他消息队列的`msg_msg`的`m_list`字段，而我们构造**每条消息队列上只有一条消息**，泄露其`m_list`，即为`msg_queue`的地址。泄露完之后，继续伪造`msg_msg->next`字段可泄露整个该消息队列中每个结构体的地址。

**可以实现任意地址读。**

但是需要注意的是，我们需要伪造我们需要读的地址 `target` 的 `next` 指针为 `NULL`，不然在`store_msg`进行数据拷贝的时候，是以`NULL`指针为结束判断条件，因此我们需要满足`target->next==NULL` or `target->next->next==NULL`，反正需要我们伪造的任意读链表存在一个`NULL`节点，且中途不能到达不可读地址，否则会造成 kernel panic。

### ②任意地址写

在`do_msgsnd`函数中调用了`load_msg`进行用户到内核的数据拷贝，若我们利用`userfaultfd`机制暂停一个线程，再在另一个线程中篡改掉`msg->next`指针，则可以**实现任意地址写。**

模板采用的`arttnba3`师傅的模板：
