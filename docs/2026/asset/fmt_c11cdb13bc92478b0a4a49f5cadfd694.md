
# [原创]自己实现的一个基于x86的操作系统

祝各位坛友圣诞节快乐！

不知道还有人对这些东西有兴趣么![](https://bbs.kanxue.com/view/img/face/16.gif)

很早就被x86的一些底层细节idt啊gdt啊cpl、dpl、rpl之类的搞的晕头转向，后来渐渐的对这些东西有了概念，但还是不清晰。断断续续的写了很多汇编程序来验证理解的是否正确，慢慢的发现自己去实现一个实验os好像也没有太多技术上的难点了。于是就搞了这个东西。

真正做的时候发现也挺烦的，r0下每写一行都要考虑要不要关中断之类的，好在本身就简单，目前好像还没发现啥重大的bug![](https://bbs.kanxue.com/view/img/face/27.gif)

虽然小，但这真的是一个多任务操作系统，而不是一个bootloader之类的。他能同时跑多个任务，且每个任务有独立的进程空间。任务运行于r3，支持十几个系统调用，可以完成一些简单的功能。`src`里有为他编写的应用范例，比如`shell`

附件的`src`可以在windows下用cygwin编译（要把cygwin的bin目录加到windows环境变量里，然后直接运行`build.bat`即可，`oskernel`即为生成的内核），也可以在linux下运行`sh`脚本`lbuild`编译

因为内核的编写遵循了grub的muiltboot规范，所以编译成的`oskernel`须有grub引导。

`tinix.img`这个软盘镜像已经安装了grub,把`oskernel`放到`boot`目录下，然后在虚拟机下就可以跑了

在实际的物理机上也可以用，修改`makefile` `CCFLAGS`里加上`-D NO_FLOPPY`,编译出`oskernel`直接用现有操作系统里的grub或grub4dos就可以引导

这是附件压缩文件的目录

```
|-- floppy.img                                        #最终完成的软盘镜像
|-- tinix                                                        #内核目录
|   |-- boot                                                #内核加载器目录
|   |-- kernel                                        #内核代码目录
|   |   |-- dev                                        #设备驱动目录
|   |   |-- fs                                        #文件系统目录
|   |   |-- include                                #头文件
|   |   |-- init                                        #内核代码入口
|   |   |-- kernel                                #任务管理、陷阱门、时钟、系统调用等
|   |   |-- lib                                        #内核lib
|   |   |-- mm                                        #内存管理目录
|   |   |-- Makefile                                #用于Cygwin下编译的makefile
|   |   `-- Makefile.linux                #用于linux下编译的makefile
|   |-- build.bat                                #用于Cygwin下编译的两个批处理
|   |-- clean.bat
|   |-- lbuild                                        #用于linux的shell脚本
|   |-- lclean
|   |-- build.log                                #gcc编译生成的错误输出
|   |-- loader.map                                #内核加载器符号文件
|   |-- system.map                                #内核符号文件
|   |-- system.idc                                #用于IDA的内核符号脚本
|   |-- map2idc.pl                                #生成IDA脚本的perl脚本
|   `-- oskernel                                        #最终编译出的内核镜像
`-- user                                                        #用户态代码目录
    |-- app                                                #用户态应用源代码
    |-- include                                        #用户态头文件
    |-- lib                                                #用户态lib函数源代码
    |-- Makefile.eval                        #用户态应用的几个makefile
    |-- Makefile.pi
    |-- Makefile.queens
    |-- Makefile.sh
`-- objs                                                #避免重新编译的obj文件
```

具体的一些实现细节见附件

贴些应用演示

![](https://bbs.kanxue.com/upload/attach/201312/321334_ajc5hlcb9ay2py5.png)

![](https://bbs.kanxue.com/upload/attach/201312/321334_0gvgc2whur7f4fs.png)

![](https://bbs.kanxue.com/upload/attach/201312/321334_ade10d81ehcyypk.png)

![](https://bbs.kanxue.com/upload/attach/201312/321334_3ako1m54kqe5l55.png)

>>>>>>>>>>>以下是帖子doc附件的内容<<<<<<<<<<<

## 简介

这是一个实验性质的操作系统，起初目的是为验证我对x86底层的一些了解是否正确，以及尝试一些不同于现有操作系统的想法。因为一开始学写`makefile`参考了于渊的《自己动手写操作系统》，名称也没有改，就叫`tinix`。但已经和原来的`tinix`没有任何关系了。为了将精力尽量投入在内核本身的逻辑上，一些和硬件相关的部分尽量简化或尽量移植，一些libc功能性函数则是直接完全移植，如`vsprintf`等。

至今为止，仅实现了多任务管理、任务特权级管理（使用0和3两级，普通程序运行于ring3内核运行于ring0）、多控制台管理（F1~F4可切换）、不高于4g的内存管理（使用两级分页）、有限的文件系统管理、fat12文件系统、虚拟文件系统、以及软驱驱动和键盘驱动。目前还没有实现的包括多用户、文件权限、硬盘驱动、和其他外设驱动。

对于应用程序的支持，仅实现了单个任务对应单个ELF格式文件的加载，不支持写时复制机制和动态库加载机制。

此项目的目录结构如下

```
|-- floppy.img                                        #最终完成的软盘镜像
|-- tinix                                                        #内核目录
|   |-- boot                                                #内核加载器目录
|   |-- kernel                                        #内核代码目录
|   |   |-- dev                                        #设备驱动目录
|   |   |-- fs                                        #文件系统目录
|   |   |-- include                                #头文件
|   |   |-- init                                        #内核代码入口
|   |   |-- kernel                                #任务管理、陷阱门、时钟、系统调用等
|   |   |-- lib                                        #内核lib
|   |   |-- mm                                        #内存管理目录
|   |   |-- Makefile                                #用于Cygwin下编译的makefile
|   |   `-- Makefile.linux                #用于linux下编译的makefile
|   |-- build.bat                                #用于Cygwin下编译的两个批处理
|   |-- clean.bat
|   |-- lbuild                                        #用于linux的shell脚本
|   |-- lclean
|   |-- build.log                                #gcc编译生成的错误输出
|   |-- loader.map                                #内核加载器符号文件
|   |-- system.map                                #内核符号文件
|   |-- system.idc                                #用于IDA的内核符号脚本
|   |-- map2idc.pl                                #生成IDA脚本的perl脚本
|   `-- oskernel                                        #最终编译出的内核镜像
|-- tools
|   `-- WinImage.exe                        #用于在windows下编辑软盘镜像的工具
`-- user                                                        #用户态代码目录
    |-- app                                                #用户态应用源代码
    |-- include                                        #用户态头文件
    |-- lib                                                #用户态lib函数源代码
    |-- Makefile.eval                        #用户态应用的几个makefile
    |-- Makefile.pi
    |-- Makefile.queens
    |-- Makefile.sh
`-- objs                                                #避免重新编译的obj文件
```

下文将对内核态的设计与实现进行解释，并对该系统的实际应用进行演示。

## 内核加载器

### 主要文件

- `/tinix/boot/boot.S`
- `/tinix/kernel/loader.c`

为方便内核设计和编译，内核的加载位置位于高地址`0xC0000000`，但物理内存无法保证有这个物理地址，因此内核需要通过一个位于低地址的加载器将其转移并映射到高地址。这就是内核加载器的作用。

内核加载器自身的加载则需要借助grub，因此内核加载器的编写遵循了grub的ultiboot规范。同时内核加载器还有一个任务，即传递grub探测到的硬件信息。这个项目里只使用到了grub探测到的物理内存上限。

在`loader.c`中，内核加载器会将`kernel`文件复制到物理0地址。同时将物理内存上限保存到`0x00090000`，将当前光标位置保存到`0x00090004`。加载工作完成之后，内核加载器将直接跳转到0地址，将控制权移交实际内核。

## 内核初始化

### 主要文件

- `/tinix/kernel/init/head.S`
- `/tinix/kernel/init/main.c`

在`head.S`中，`start`为内核的入口，在这里重新设置`esp`为`0x90000`，然后依次开分页、初始化gdt、初始化中断控制器、初始化idt。

开分页时，为保证代码地址和数据地址正确。这里将线性地址`0~0x400000`和`0xC0000000~0xC0400000`映射到相同的物理地址0~4m。映射关系如图。建立完映射关系后，`init_page`通过调整`esp`和修改返回地址转移到高3g执行。

![](https://bbs.kanxue.com/upload/attach/201312/321334_wrcdht1l5eog5x1.png)

全局描述符表中初始化了4个描述符，分别为内核代码段，内核数据段，用户代码段和用户数据段。这4个描述符在表中是连续的，这是为了兼容`sysenter`、`sysexit`的规定。中断描述符表初始状态均为空。

`head.S`完成基本的初始化工作后，将进入`k_main`函数，在这里调用各个模块的初始化接口。操作系统所有的初始化工作完成之后，0号任务将依次在0~2三个空闲控制台上启动三个子任务`shell`，路径为`/bin/sh`。

之后，0号任务将进入半睡眠状态，最大限度的让出cpu资源。

## 内存管理

### 主要文件

- `/tinix/kernel/mm/memory.c`

这里的内存管理指狭义的内存管理，及映射关系的建立和映射关系的取消，是页级的。内核态内存的管理，将从物理内存1M位置开始，在1M位置，依次为内核页目录表、内核页表0~3。尾随其后的是根据loader传来的物理内存大小计算得出的一个`mem_map`数组。这个数组用于管理物理内存，数组地址为`0x00105000`。`mem_map`之后的内存将用于动态管理。建立完`mem_map`之后，为避免其他问题，将取消掉低4M的映射关系。

因为只为内核保留了4张页表，所以内核本身能使用的内存将不能大于16M。另外，为方便内核态线性地址和物理地址之间的相互转换，内核态内存和物理内存的映射关系必须为“`线性地址 - 物理地址 = PAGE_OFFSET`”。因此，内核内存申请和用户内存申请的策略是不同的。内核态内存申请将在`mem_map`中从前往后找，用户态相反。

最终形成的映射关系，如下图所示

![](https://bbs.kanxue.com/upload/attach/201312/321334_7pevgmb3f2zjakr.png)

内核态内存申请与释放接口为`k_get_page`和`k_free_page`。用户态接口为`u_get_pages`和`u_free_page_table`。

## 文件管理

### 主要文件

- `/tinix/kernel/fs/fs.c`

文件系统的管理基于`fs.c`中维护的一张内核文件表`kfile_table`。向这张表中添加文件或添加目录通过`fs_add_file`和`fs_add_dir`进行。

文件表中每一项均为一个`file`结构，定义如下

```
typedef struct _file {
    u32 f_type;
    u32 f_father;					//父目录索引
    u32 f_time;						//最后修改时间
    u32 f_base;						//文件基址，不同的文件含义不同
    s32 f_size;						//文件大小
    s32 f_pos[MAX_TASK];			//读写位置,每个任务有自己的读写位置
    struct _file_operations *f_op;	//操作文件跳转指针 不能空！！！
    struct _file_req *f_req;		//操作请求结构，可以空
    u8  f_name[MAX_NAME];
}__attribute__((packed)) file;
```

其中比较重要的成员一是`f_op`这个指针指向操作该文件的各种接口地址。不同的文件系统在向`kfile_table`中添加文件时，须指定该结构，这样在操作这个文件时，将会根据这个指针找到正确的操作接口。
