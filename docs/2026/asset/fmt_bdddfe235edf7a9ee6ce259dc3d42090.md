
`➜  sample lsmod`
`Module                  Size  Used by`
`sample` `16384`  `0`
`nls_iso8859_1` `16384`  `1`
`nvidia_uvm` `1028096`  `0`
`intel_rapl_msr` `20480`  `0`
`intel_rapl_common` `24576`  `1` `intel_rapl_msr`
`nvidia_drm` `61440`  `10`
`nvidia_modeset` `1150976`  `13` `nvidia_drm`
`x86_pkg_temp_thermal` `20480`  `0`
`......`
`sample` `16384`  `0`
`......`

- `dmesg`命令可以查看模块输出的消息（部分情况需要root权限）

```
➜  sample dmesg
[ 0.000000] microcode: microcode updated early to revision 0xea, date = 2021-01-05
[ 0.000000] Linux version 5.11.0-46-generic (buildd@lgw01-amd64-010) (gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0, GNU ld (GNU Binutils for Ubuntu) 2.34) #51~20.04.1-Ubuntu SMP Fri Jan 7 06:51:40 UTC 2022 (Ubuntu 5.11.0-46.51~20.04.1-generic 5.11.22)
......
[ 20.474554] audit: type=1400 audit(1642506075.478:44): apparmor="DENIED" operation="open" profile="snap.snap-store.ubuntu-software" name="/etc/PackageKit/Vendor.conf" pid=2760 comm="snap-store" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
[ 6287.886473] perf: interrupt took too long (2501 > 2500), lowering kernel.perf_event_max_sample_rate to 79750
[ 8920.311392] Hello world - unr4v31.
```

- `rmmod`命令从内核移除已注册的模块

```
➜  sample sudo rmmod sample
➜  sample dmesg
[ 0.000000] microcode: microcode updated early to revision 0xea, date = 2021-01-05
[ 0.000000] Linux version 5.11.0-46-generic (buildd@lgw01-amd64-010) (gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0, GNU ld (GNU Binutils for Ubuntu) 2.34) #51~20.04.1-Ubuntu SMP Fri Jan 7 06:51:40 UTC 2022 (Ubuntu 5.11.0-46.51~20.04.1-generic 5.11.22)
......
[ 8920.311392] Hello world - unr4v31.
[ 8961.827092] Goodbye world - unr4v31.
```

# 字符设备模块

字符设备驱动是一种在不使用缓冲区高速缓存的情况下一次读取和写入一个字符数据的驱动程序，例如：键盘、声卡、打印机驱动程序。

此外还有块设备和网络驱动程序：

- 块设备驱动程序允许通过缓冲区告诉缓存和块单元中的I/O进行随机访问，例如硬盘。
- 网络设备驱动程序位于网络堆栈和网络硬件之间，负责发送和接受数据，例如以太网、网卡。

`file_operations`结构是为字符设备、块设备驱动程序与通用程序之间的通信提供的接口。可以使用结构体内的函数指针，例如：`read`, `write`, `open`, `release`, `unlocked_ioctl`。而网络设备不使用`file_operations`结构，应当使用`include/linux/netdevice.h`中的`net_device`结构体。

下面是Linux 5.11的`file_operations`结构体内容：

```c
struct file_operations {
struct module *owner;
loff_t (*llseek) (struct file *, loff_t, int);
ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
int (*iopoll)(struct kiocb *kiocb, bool spin);
int (*iterate) (struct file *, struct dir_context *);
int (*iterate_shared) (struct file *, struct dir_context *);
__poll_t (*poll) (struct file *, struct poll_table_struct *);
long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
int (*mmap) (struct file *, struct vm_area_struct *);
unsigned long mmap_supported_flags;
int (*open) (struct inode *, struct file *);
int (*flush) (struct file *, fl_owner_t id);
int (*release) (struct inode *, struct file *);
int (*fsync) (struct file *, loff_t, loff_t, int datasync);
int (*fasync) (int, struct file *, int);
int (*lock) (struct file *, int, struct file_lock *);
ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
int (*check_flags)(int);
int (*flock) (struct file *, int, struct file_lock *);
ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
int (*setlease)(struct file *, long, struct file_lock **, void **);
long (*fallocate)(struct file *file, int mode, loff_t offset, loff_t len);
void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
unsigned (*mmap_capabilities)(struct file *);
#endif
ssize_t (*copy_file_range)(struct file *, loff_t, struct file *, loff_t, size_t, unsigned int);
loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in, struct file *file_out, loff_t pos_out, loff_t len, unsigned int remap_flags);
int (*fadvise)(struct file *, loff_t, loff_t, int);
} __randomize_layout;
```

使用下面的方式绑定设备模块中的`open`函数，在使用`open`系统调用时内核会输出“chardev_open”：

```c
static int chardev_open(struct inode *inode, struct file *file)
{
printk("chardev_open");
return 0;
}
struct file_operations chardev_fops =
{
.open = chardev_open,
};
```

## 一个简单的例子

环境信息（这里我换了个ubuntu的环境，不过小版本差别不大）：

```
➜  ~ uname -a
Linux unravel 5.11.0-43-generic #47~20.04.2-Ubuntu SMP Mon Dec 13 11:06:56 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
➜  ~ gcc --version
gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
➜  ~ make --version
GNU Make 4.2.1
```

编写如下模块源码，保存为`chardev.c`：
