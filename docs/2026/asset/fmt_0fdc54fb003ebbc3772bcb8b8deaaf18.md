
# [原创]cve-2022-0847复现

## 漏洞介绍

在 Linux 内核的 `copy_page_to_iter_pipe` 和 `push_pipe` 函数中，管道缓冲区结构的 `flags` 成员缺乏正确的初始化，因此可能包含过期的值。无权限的本地用户可利用此漏洞任意写入文件，从而完成提权。

## 漏洞详细

### pipe机制

`pipe` 是内核提供的一种通讯机制，返回两个文件描述符，一个用于发送数据，另一个用于接受数据。实现方式是**循环队列**，**它只能用于有血缘关系的进程间通信**。

`pipe` 结构体如下：

```c
struct pipe_inode_info {
struct mutex mutex;
wait_queue_head_t rd_wait, wr_wait;
unsigned int head;
unsigned int tail;
unsigned int max_usage;
unsigned int ring_size;
#ifdef CONFIG_WATCH_QUEUE
bool note_loss;
#endif
unsigned int nr_accounted;
unsigned int readers;
unsigned int writers;
unsigned int files;
unsigned int r_counter;
unsigned int w_counter;
struct page *tmp_page;
struct fasync_struct *fasync_readers;
struct fasync_struct *fasync_writers;
struct pipe_buffer *bufs;
struct user_struct *user;
#ifdef CONFIG_WATCH_QUEUE
struct watch_queue *watch_queue;
#endif
};
```

`bufs` 是循环数组，`head` 用来写管道，指向 `buffer` 头部，`tail` 用来读管道，指向 `buffer` 尾部。

`pipe_buffer` 结构体如下：

```c
struct pipe_buffer {
struct page *page;
unsigned int offset, len;
const struct pipe_buf_operations *ops;
unsigned int flags;
unsigned long private;
};
```

### pipe_write

向管道中写数据时会调用函数 `pipe_write()`

```c
pipe_write(struct kiocb *iocb, struct iov_iter *from)
{
....
head = pipe->head;
was_empty = pipe_empty(head, pipe->tail);
chars = total_len & (PAGE_SIZE-1);
if (chars && !was_empty) {
unsigned int mask = pipe->ring_size - 1;
struct pipe_buffer *buf = &pipe->bufs[(head - 1) & mask];
int offset = buf->offset + buf->len;
if ((buf->flags & PIPE_BUF_FLAG_CAN_MERGE) &&
offset + chars <= PAGE_SIZE) {
ret = pipe_buf_confirm(pipe, buf);
if (ret)
goto out;
ret = copy_page_from_iter(buf->page, offset, chars, from);
if (unlikely(ret < chars)) {
ret = -EFAULT;
goto out;
}
buf->len += ret;
if (!iov_iter_count(from))
goto out;
}
}
for (;;) {
....
if (!pipe_full(head, pipe->tail, pipe->max_usage)) {
unsigned int mask = pipe->ring_size - 1;
struct pipe_buffer *buf = &pipe->bufs[head & mask];
struct page *page = pipe->tmp_page;
int copied;
if (!page) {
page = alloc_page(GFP_HIGHUSER | __GFP_ACCOUNT);
if (unlikely(!page)) {
ret = ret ? : -ENOMEM;
break;
}
pipe->tmp_page = page;
}
/* Allocate a slot in the ring in advance and attach an
* empty buffer. If we fault or otherwise fail to use
* it, either the reader will consume it or it'll still
* be there for the next write.
*/
spin_lock_irq(&pipe->rd_wait.lock);
head = pipe->head;
if (pipe_full(head, pipe->tail, pipe->max_usage)) {
spin_unlock_irq(&pipe->rd_wait.lock);
continue;
}
pipe->head = head + 1;
spin_unlock_irq(&pipe->rd_wait.lock);
/* Insert it into the buffer array */
buf = &pipe->bufs[head & mask];
buf->page = page;
buf->ops = &anon_pipe_buf_ops;
buf->offset = 0;
buf->len = 0;
if (is_packetized(filp))
buf->flags = PIPE_BUF_FLAG_PACKET;
else
buf->flags = PIPE_BUF_FLAG_CAN_MERGE;
pipe->tmp_page = NULL;
....
}
}
```

`pipe_write()` 关键功能如下：

1. 如果 `pipe` 不为空，则说明 `pipe` 中还有未被读取的部分，如果未被读取的数据所在的 `buffer` 标识为 `PIPE_BUF_FLAG_CAN_MERGE` 且要写入的数据加上原来的数据不超过一个页，即可在原来的 `buffer` 上续写。
2. 不符合上述条件就会新申请一个页面，将新申请的页面插入 `pipe` 中，默认初始化 `flag` 为 `PIPE_BUF_FLAG_CAN_MERGE`，即默认状态是允许 `buffer` 续写的。

### 零拷贝

Linux 系统中一切皆文件，Linux 系统的很多活动无外乎**读操作**和**写操作**，零拷贝就是为了提高读写性能而出现的。

#### 页面高速缓存

我们知道文件一般存放在硬盘（机械硬盘或固态硬盘）中，CPU 并不能直接访问硬盘中的数据，而是需要先将硬盘中的数据读入到内存中，然后才能被 CPU 访问。由于读写硬盘的速度比读写内存要慢很多（DDR4 内存读写速度是机械硬盘500倍，是固态硬盘的200倍），所以为了避免每次读写文件时，都需要对硬盘进行读写操作，Linux 内核使用 页缓存（Page Cache） 机制来对文件中的数据进行缓存。

当用户对文件进行读写时，实际上是对文件的 页缓存 进行读写。所以对文件进行读写操作时，会分以下两种情况进行处理：

- 当从文件中读取数据时，如果要读取的数据所在的页缓存已经存在，那么就直接把页缓存的数据拷贝给用户即可。否则，内核首先会申请一个空闲的内存页（页缓存），然后从文件中读取数据到页缓存，并且把页缓存的数据拷贝给用户。
- 当向文件中写入数据时，如果要写入的数据所在的页缓存已经存在，那么直接把新数据写入到页缓存即可。否则，内核首先会申请一个空闲的内存页（页缓存），然后从文件中读取数据到页缓存，并且把新数据写入到页缓存中。对于被修改的页缓存，内核会定时把这些页缓存刷新到文件中。

将 `pipe_buffer` 直接指向页高速缓存作为 `pipe` 的 `buf` 页使用即可实现以零拷贝的方式在文件描述符之间移动数据。

### copy_page_to_iter_pipe

源码：

```c
static size_t copy_page_to_iter_pipe(struct page *page, size_t offset, size_t bytes,
struct iov_iter *i)
{
struct pipe_inode_info *pipe = i->pipe;
struct pipe_buffer *buf;
unsigned int p_tail = pipe->tail;
unsigned int p_mask = pipe->ring_size - 1;
unsigned int i_head = i->head;
size_t off;
if (unlikely(bytes > i->count))
bytes = i->count;
if (unlikely(!bytes))
return 0;
if (!sanity(i))
return 0;
off = i->iov_offset;
buf = &pipe->bufs[i_head & p_mask];
if (off) {
if (offset == off && buf->page == page) {
/* merge with the last one */
buf->len += bytes;
i->iov_offset += bytes;
goto out;
}
i_head++;
buf = &pipe->bufs[i_head & p_mask];
}
if (pipe_full(i_head, p_tail, pipe->max_usage))
return 0;
buf->ops = &page_cache_pipe_buf_ops;
get_page(page);
buf->page = page;
buf->offset = offset;
buf->len = bytes;
pipe->head = i_head + 1;
i->iov_offset = offset + bytes;
i->head = i_head;
out:
i->count -= bytes;
return bytes;
}
```

关键功能如下：

1. 找到 `pipe` 的写指针 `head`
2. 将 `head` 指向的 `pipe_buffer` 的 `page` 改为准备好的文件缓存页

这里函数没有初始化 `pipe_buffer` 的 `flag`，如果用户调用 `copy_page_to_iter_pipe` 之前，`head` 指向 `pipe_buffer` 是刚申请来的，那此时的 `pipe_buffer` 的 `flag` 应该为 `PIPE_BUF_FLAG_CAN_MERGE`，如果再次调用 `pipe_write` 函数用户就可以向 `pipe_buffer` 指向的文件缓存页中写数据，这就造成了一个任意文件写。

`copy_page_to_iter_pipe` 可以由 `splice` 调用

## 调试漏洞

### poc逻辑
