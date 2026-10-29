
```text
    d42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2H3k6h3c8A6P5h3y4G2L8g2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5k6o6t1I4x3o6f1H3x3#2)9J5k6h3S2@1L8b7`.`.
```

这个大神也很感谢，我看了不下20遍，我把他的代码拿过来了，可以他是３２位的，而且是arm,天真的我以为小改一下就能跑

```text
   71dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3u0A6L8r3g2D9K9h3&6#2P5q4)9J5c8X3W2G2N6Y4W2Q4y4h3k6J5L8$3!0@1i4K6g2X3M7X3g2K6k6h3q4J5j5$3R3`.
```

还有这个代码，他们的区别我下面会讲，这个工具，好像是个著名的提权工具

# 漏洞具体分析

漏洞描述什么的，不好意思，看不懂，代码修改完了，提权不了，我去，蒙b了，心碎了。

第二天，天晴了，雨停了，我又行了。开始从头分析

## readv是什么

`readv`函数，是一种高级I/O函数，就是加快读写速度的，每种I/O驱动都可以实现，也可以不实现，如果不实现，应该就没法使用`readv`。

这里使用pipe管道驱动的`readv`函数，最后调用到了`pipe_read`，这个函数有堆数组越界，导致任意读写，然后在进行一系列堆喷，溢出计算，等等看不懂的操作，据说就能提权，我们先看`readv`函数调用顺序。

```text
read->SYSCALL_DEFINE3(readv...）->vfs_readv->do_readv_writev-> do_sync_readv_writev/do_loop_readv_writev -> pipe_read
```

（`do_sync_readv_writev`/`do_loop_readv_writev`, 这两个函数我没有具体研究。。。。）

代码我还是贴上吧，里面的注释，如果看不懂，可以看完在慢慢分析

```text
static ssize_t

pipe_read(struct kiocb *iocb, const struct iovec *_iov,

       unsigned long nr_segs, loff_t pos)

{

    struct file *filp = iocb->ki_filp;

    struct pipe_inode_info *pipe = filp->private_data;

    int do_wakeup;

    ssize_t ret;

    struct iovec *iov = (struct iovec *)_iov;

    size_t total_len;



    total_len = iov_length(iov, nr_segs);　　　　//读取iov－>leng数据的长度的总长度

    /* Null read succeeds. */

    if (unlikely(total_len == 0))

        return 0;



    do_wakeup = 0;

    ret = 0;

    __pipe_lock(pipe);

    for (;;) {

        int bufs = pipe->nrbufs;

        if (bufs) {

            int curbuf = pipe->curbuf;

            struct pipe_buffer *buf = pipe->bufs + curbuf;

            const struct pipe_buf_operations *ops = buf->ops;

            void *addr;

            size_t chars = buf->len;

            int error, atomic;



            if (chars > total_len)　　　　　　　　　　　　　　　　　　//chars 最大0x1000 也就是１页大小

                chars = total_len;



            error = ops->confirm(pipe, buf);

            if (error) {

                if (!ret)

                    ret = error;

                break;

            }

   //参数漏洞的位置主要在这里，这里有页错误然后goto的初衷，是因为高端内存映射不能一直驻留，pipe使用了高端内存，所以它本身会有页错误处理，而攻击者利用他的页错误处理

   //将本该是驱动本身的页，没有映射到高端内存的错误，变成了用户空间接收数据的页的错误，强行goto

            atomic = !iov_fault_in_pages_write(iov, chars);　　　//这一行也很很重要，堆喷的内存可以必须可读

redo:

            addr = ops->map(pipe, buf, atomic);

            error = pipe_iov_copy_to_user(iov, addr + buf->offset, chars, atomic);　　　//拷贝数据，无论怎么拷贝，抖不会超过一页的大小

                                                                                                                                                   //在第一次进入pipe_iov_copy_to_user这个函数以后，将界面释放，造成错误

            ops->unmap(pipe, buf, addr);

            if (unlikely(error)) {

                /*

                 * Just retry with the slow path if we failed.

                 */

                if (atomic) {

                    atomic = 0;

                    goto redo;　　　　　　　　　　　　　　　　　　//构造页错误，回强行执行到这个位置，导致　total_len 也就是接收数据的长度没更新

                                                                                                                     //第一次执行pipe_iov_copy_to_user，故意构造错误，返回，会执行到这里

                }

                if (!ret)

                    ret = error;

                break;

            }

            ret += chars;

            buf->offset += chars;

            buf->len -= chars;



            /* Was it a packet buffer? Clean up and exit */

            if (buf->flags & PIPE_BUF_FLAG_PACKET) {

                total_len = chars;

                buf->len = 0;

            }



            if (!buf->len) {　　　　　　　//这个页读完了，更新到下一个页，来接着读

                buf->ops = NULL;

                ops->release(pipe, buf);

                curbuf = (curbuf + 1) & (pipe->buffers - 1);   //从１－１０的读取顺序

                pipe->curbuf = curbuf;

                pipe->nrbufs = --bufs;

                do_wakeup = 1;

            }

                total_len -= chars;　　　　　　　　　　　　　//第二次执行完　pipe_iov_copy_to_user　函数以后，会执行到这里更新长度，这个时候，

                                                                                               //iov已经消耗完毕了，但是total_len,还没有读完 　溢出部分　＝　total_len　－　chars　－　第一次执行时消耗的个数

            if (!total_len)

                break;    /* common path: read succeeded */

        }

        if (bufs)    /* More to do? */

            continue;

        if (!pipe->writers)

            break;

        if (!pipe->waiting_writers) {

            /* syscall merging: Usually we must not sleep

             * if O_NONBLOCK is set, or if we got some data.

             * But if a writer sleeps in kernel space, then

             * we can wait for that data without violating POSIX.

             */

            if (ret)

                break;

            if (filp->f_flags & O_NONBLOCK) {

                ret = -EAGAIN;

                break;

            }

        }

        if (signal_pending(current)) {

            if (!ret)

                ret = -ERESTARTSYS;

            break;

        }

        if (do_wakeup) {

            wake_up_interruptible_sync_poll(&pipe->wait, POLLOUT | POLLWRNORM);

             kill_fasync(&pipe->fasync_writers, SIGIO, POLL_OUT);

        }

        pipe_wait(pipe);

    }

    __pipe_unlock(pipe);



    /* Signal writers asynchronously that there is more room. */

    if (do_wakeup) {

        wake_up_interruptible_sync_poll(&pipe->wait, POLLOUT | POLLWRNORM);

        kill_fasync(&pipe->fasync_writers, SIGIO, POLL_OUT);

    }

    if (ret > 0)

        file_accessed(filp);

    return ret;

}



static int　pipe_iov_copy_to_user(struct iovec *iov, const void *from, unsigned long len,

              int atomic)

{

    unsigned long copy;



    while (len > 0) {

        while (!iov->iov_len)

            iov++;

        copy = min_t(unsigned long, len, iov->iov_len);



        if (atomic) {

            if (__copy_to_user_inatomic(iov->iov_base, from, copy))

                return -EFAULT;

        } else {

            if (copy_to_user(iov->iov_base, from, copy))

                return -EFAULT;

        }

        from += copy;

        len -= copy;

        iov->iov_base += copy;

        iov->iov_len -= copy;　　拷贝过的项，iov_len=0   copy的值，看上面

    }

    return 0;

}
```

我们通过这个函数来分析，为什么产生漏洞，搞清楚，不是如何利用，如何进行任意地址写，只是分析为什么会产生

如果造成`atomic`＝１，`pipe_iov_copy_to_user`执行错误返回，这个时候，`iov`数组就会已经copy过的项的`iov_len`已经等于０了，但是这个时候，函数的逻辑是将`atomic=0`，然后继续`goto`到这个函数的位置，继续copy，就从这个地方看，再次进入`iov`函数，只有`atomic`和`iov`的数据产生了变化，拷贝的个数`chars`和源地址`addr + buf->offset`均未发生变化，不考虑漏洞的问题，这本身就是bug了，内存拷贝，目的地址变小了，但是拷贝的个数和源缓冲区大小都没变的情况下，必然导致溢出。

### 第一个点，缓冲区溢出了。

溢出的位置是哪里，我们看这个函数，`iov`是否数组，但是是通过指针使用的，如果要写入的数据不为0，`iov`数组就会越界，直到`len`小于0为止

### 第二个点，`iov`数组会一直越界，直到`len`小于0位置，缓冲去溢出的位置就是越界的`iov`数组元素的`iov_base`，溢出的长度是`iov_len`

这个漏洞是拷贝数据函数是`pipe_iov_copy_to_user`里面的`__copy_to_user_inatomic`，要达到地任意址写入任意大小的任意数据，我们必须同时控制他的三个参数（`iov->iov_base`，`from`，`copy`），`copy`大部分情况下等于`iov->iov_le`，`from`，是我们要读的缓冲区的数据，也就是我们写进来的数据，通过`readv`读出去。而`iov`是会发生数组越界的。

### 第三个点，`copy`函数三大参数，现在源缓冲区是我们写进来的数据，而长度和目的缓冲区都是`iov`，而且`iov`是会越界的

我们现在需要跟踪分析这个`iov`的前世今生了。看看能不能控制他。

```text
    readv ->  vfs_readv  -> do_readv_writev-> rw_copy_check_uvector
```

`iov`数组，只要大于８组，就可以在`rw_copy_check_uvector`这个函数通过`kmalloc`分配，然后将用户的数据拷贝过来的，小于８组看上个函数
