
```C
/*
* The caller must ensure that fd table isn't shared or hold rcu or file lock
*/
static inline struct file *files_lookup_fd_raw(struct files_struct *files, unsigned int fd)
{
struct fdtable *fdt = rcu_dereference_raw(files->fdt);
if (fd < fdt->max_fds) {
fd = array_index_nospec(fd, fdt->max_fds);
return rcu_dereference_raw(fdt->fd[fd]);
}
return NULL;
}
```

根据注释也可以认为，需要保证文件描述符表没有被共享过，或者是持有文件锁。会返回一个 `fd` 表中的 `struct file` 结构（`fdt->fd[fd]`）。

如果引用计数不为 `1`，则调用 `__fget` 去获取指针，其中主要是调用了 `__fget_files` 函数。

```C
#define get_file_rcu_many(x, cnt) \
atomic_long_add_unless(&(x)->f_count, (cnt), 0)
static struct file *__fget_files(struct files_struct *files, unsigned int fd,
fmode_t mask, unsigned int refs)
{
struct file *file;
rcu_read_lock();
loop:
file = files_lookup_fd_rcu(files, fd);
if (file) {
/* File object ref couldn't be taken.
* dup2() atomicity guarantee is the reason
* we loop to catch the new file (or NULL pointer)
*/
if (file->f_mode & mask)
file = NULL;
else if (!get_file_rcu_many(file, refs))
goto loop;
}
rcu_read_unlock();
return file;
}
static inline struct file *__fget(unsigned int fd, fmode_t mask,
unsigned int refs)
{
return __fget_files(current->files, fd, mask, refs);
}
```

这里的 `files_lookup_fd_rcu` 直接可以认为是获取文件结构体的，随后判断里面是否包含禁止的模式，然后增加文件计数引用 （`get_file_rcu_many`）。

回过头来看看 `__fdget_pos` 函数

```C
unsigned long __fdget_pos(unsigned int fd)
{
unsigned long v = __fdget(fd);
struct file *file = (struct file *)(v & ~3);
if (file && (file->f_mode & FMODE_ATOMIC_POS)) {
if (file_count(file) > 1) {
v |= FDPUT_POS_UNLOCK;
mutex_lock(&file->f_pos_lock);
}
}
return v;
}
```

获取到的文件指针将最低两位置为 0（对齐），如果被设置了 `FMODE_ATOMIC_POS` 且 文件引用大于 1，那么上锁，到这里，才分析完 `do_writev` 的第一句话，来看看接下来的语句，重点是 `vfs_writev` 函数。

```C
static ssize_t do_iter_write(struct file *file, struct iov_iter *iter,
loff_t *pos, rwf_t flags)
{
size_t tot_len;
ssize_t ret = 0;
if (!(file->f_mode & FMODE_WRITE))
return -EBADF;
if (!(file->f_mode & FMODE_CAN_WRITE))
return -EINVAL;
tot_len = iov_iter_count(iter);
if (!tot_len)
return 0;
ret = rw_verify_area(WRITE, file, pos, tot_len);
if (ret < 0)
return ret;
if (file->f_op->write_iter)
ret = do_iter_readv_writev(file, iter, pos, WRITE, flags);
else
ret = do_loop_readv_writev(file, iter, pos, WRITE, flags);
if (ret > 0)
fsnotify_modify(file);
return ret;
}
static ssize_t vfs_writev(struct file *file, const struct iovec __user *vec,
unsigned long vlen, loff_t *pos, rwf_t flags)
{
struct iovec iovstack[UIO_FASTIOV];
struct iovec *iov = iovstack;
struct iov_iter iter;
ssize_t ret;
ret = import_iovec(WRITE, vec, vlen, ARRAY_SIZE(iovstack), &iov, &iter);
if (ret >= 0) {
file_start_write(file);
ret = do_iter_write(file, &iter, pos, flags);
file_end_write(file);
kfree(iov);
}
return ret;
}
```

首先根据 `writev` 的结构体解出数据和长度，然后调用 `do_iter_write` 去写文件，而在 `do_iter_write` 中可以发现，这里作权限校验了，校验了是否可写以及文件描述符是否可写，这里的两层意思分别是文件本身是否具有可写权限以及你打开的文件描述符是否包含了 `O_WRITE` 权限位。

随后进行写，写的过程会根据文件系统调用对应的写函数（`write_iter`）

```C
static inline ssize_t call_write_iter(struct file *file, struct kiocb *kio,
struct iov_iter *iter)
{
return file->f_op->write_iter(kio, iter);
}
```

下面是完整的调用链，感兴趣可以跟一下。

`do_writev`->`vfs_writev`->`do_iter_write`->`do_iter_readv_writev`->`call_write_iter`->`.write_iter` ->`ext4_file_write_iter` -> `ext4_buffered_write_iter`

在这个函数里面可以看到我注释的两个位置分别对文件节点进行了上锁和解锁。

```C
static ssize_t
ext4_file_write_iter(struct kiocb *iocb, struct iov_iter *from)
{
struct inode *inode = file_inode(iocb->ki_filp);
if (unlikely(ext4_forced_shutdown(EXT4_SB(inode->i_sb))))
return -EIO;
#ifdef CONFIG_FS_DAX
if (IS_DAX(inode))
return ext4_dax_write_iter(iocb, from);
#endif
if (iocb->ki_flags & IOCB_DIRECT)
return ext4_dio_write_iter(iocb, from);
else
return ext4_buffered_write_iter(iocb, from);
}
static ssize_t ext4_buffered_write_iter(struct kiocb *iocb,
struct iov_iter *from)
{
ssize_t ret;
struct inode *inode = file_inode(iocb->ki_filp);
if (iocb->ki_flags & IOCB_NOWAIT)
return -EOPNOTSUPP;
ext4_fc_start_update(inode);
inode_lock(inode);//这里会上锁
ret = ext4_write_checks(iocb, from);
if (ret <= 0)
goto out;
current->backing_dev_info = inode_to_bdi(inode);
ret = generic_perform_write(iocb->ki_filp, from, iocb->ki_pos);
current->backing_dev_info = NULL;
out:
inode_unlock(inode);//这里会解锁
ext4_fc_stop_update(inode);
if (likely(ret > 0)) {
iocb->ki_pos += ret;
ret = generic_write_sync(iocb, ret);
}
return ret;
}
```

此时两个线程会卡在这个锁里，翻一翻时间节点，此时权限校验已经完了，第一个线程写入大量数据将第二个线程获取锁的时间，趁此机会第三个线程将 `/etc/passwd` 打开并将文件页面以这个 `uaf` 的页面使用，第二个线程获取锁之后直接将数据写入 `/etc/passwd`。

所以要彻底明白这个漏洞，还需要理解前面 UAF 的成因。

### fsconfig

这个系统调用太大了，只介绍它原有的含义和触发漏洞的位置。
