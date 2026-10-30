
# [原创]内核漏洞挖掘技术系列(1)——trinity

这是内核漏洞挖掘技术系列的第一篇。

# 前言

提到linux内核fuzz目前最流行的工具是syzkaller，不过在syzkaller出现之前(github上首次commit是2015年10月)linux内核fuzz用到最多的工具是trinity(github上首次commit是2006年3月，1.0版本发布于2012年8月)，并且就在2019年1月刚刚发布了1.9版本(<a href="elink@55cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4u0F1k6h3I4K6L8r3q4U0K9$3g2J5i4K6u0r3N6s2u0A6L8X3W2@1P5b7`.`."><mark class="encrypted">841K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4u0F1k6h3I4K6L8r3q4U0K9$3g2J5i4K6u0r3N6s2u0A6L8X3W2@1P5b7`.`.</mark></a>)。trinity网上也有各种二次开发的版本，有各种修改后在android下面跑的([https://bbs.pediy.com/thread-218969.htm](https://bbs.pediy.com/thread-218969.htm))，有用来fuzz `perf_event_open`系统调用的perf_fuzzer(<a href="elink@abfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6h3q4@1k6i4u0Q4x3V1k6H3k6i4u0X3i4K6g2X3k6i4k6W2L8Y4c8Q4y4h3k6@1k6i4y4@1M7H3`.`."><mark class="encrypted">1e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6h3q4@1k6i4u0Q4x3V1k6H3k6i4u0X3i4K6g2X3k6i4k6W2L8Y4c8Q4y4h3k6@1k6i4y4@1M7H3`.`.</mark></a>)等等。比起来trinity算得上是元老级的fuzz工具了。本文会详细分析trinity目前最新的1.9版本的实现。

# 整体架构

下面是《LCA: The Trinity fuzz tester》这篇文章中给的一张图。

![](upload/attach/202002/734571\_EEYFMFXMXJY6PET.png)

`trinity-main`执行各种初始化，然后创建执行系统调用的子进程。`trinity-main`创建的共享内存区域用于记录各种全局信息(打开文件描述符号、执行的系统调用总数以及成功和失败的系统调用数等等)和每个子进程的各种信息(pid和执行的系统调用信息等等)。

`trinity-watchdog`确保系统正常工作。它会检查子进程是否正在运行(可能会在系统调用中被暂停)，如果没有运行，则会将其杀死。当主进程检测到其中一个子进程已终止时(因为`trinity-watchdog`将其终止或出于其它原因)会启动一个新的子进程来替换它。`trinity-watchdog`还监视共享内存区域的完整性。

(PS:《LCA: The Trinity fuzz tester》这篇文章是几年以前的了，根据后文的源码分析目前`trinity-watchdog`的功能已经被整合到`trinity-main`中)

在`trinity`文件夹下除了子目录，还有下面这些源代码文件。

- `blockdevs.c`：块设备相关(目前没有用到)

![](upload/attach/202002/734571\_3FR89EZSYKKAYQT.png)

- `child.c`：执行fuzz的子进程

- `debug.c`：调试功能

- `devices.c`：解析`/proc/devices`和`/proc/misc`以对ioctl进行fuzz

- `ftrace.c`：ftrace功能，记录到`/boot/trace.txt`文件

- `generate-args.c`：系统调用参数的生成释放等处理

- `kcov.c`：代码覆盖率功能(目前没有用到)

- `locks.c`：锁功能，根据前面的叙述我们知道有多个进程操作共享内存区域，所以需要加锁

- `log-files.c`：将日志记录到本地文件

- `udp.c`：将日志记录到远程服务器

- `log.c`：对`udp.c`和`log-files.c`的封装

可以通过命令行参数选择将日志记录到远程服务器或者本地文件，也可以禁用日志功能。默认将日志记录到本地文件。将日志记录到服务器相关实现在`server`文件夹中。

![](upload/attach/202002/734571\_YQ6CEDYXY5A8QH6.png)

![](upload/attach/202002/734571\_CFJG4SYWPTWZGH3.png)

![](upload/attach/202002/734571\_8WXHQ5AUDWZR3F6.png)

这里我们只看将日志记录到本地文件中的情况。主进程的日志记录到`trinity.log`。

![](upload/attach/201903/734571\_JHSDEGJXB4DWN86.png)

每个子进程都将它执行的系统调用信息写入一个单独的日志文件`trinity-child1.log`/`trinity-child2.log`……。

![](upload/attach/202002/734571\_QN4KN75GMKQ7K6X.jpg)

- `arg-decoder.c`：子进程在执行系统调用前后会调用到其中的函数记录信息到`syscallrecord`结构体中的`prebuffer`(子进程child编号/pid/每次执行系统调用依次递增的编号/是否是32位/系统调用名和参数)和`postbuffer`(返回值/错误代码)中，然后写入到`trinity-childx.log`(如上图中所示)

![](upload/attach/202002/734571\_RZP8T2UB8HWVB4Z.png)

- `main.c`：被`trinity.c`通过`main_loop`函数调用，执行主要功能

- `objects.c`：管理系统调用中用到的文件描述符，每一种文件描述符具体的操作在`fds`文件夹中

![](upload/attach/202002/734571\_SHXCA4X7XTZ2M7B.png)

- `output.c`：打印信息

- `params.c`：命令行参数处理

- `pathnames.c`：系统调用参数如果是路径名提供相关参数

- `pids.c`：pid管理

- `post-mortem.c`：检测到taint之后向`trinity-post-mortem.log`写入每个子进程最后一个系统调用信息

- `results.c`：系统调用执行成功之后的处理

- `shm.c`：共享内存区域的初始化和创建

- `signals.c`：信号功能

- `stats.c`：打印系统调用执行总数，成功的数目，失败的数目和错误代码

- `syscall.c`：执行系统调用

- `sysv-shm.c`：初始化时创建供系统调用使用的共享内存(是用`shmget`函数创建的共享内存，不是前面说的`shm.c`中用`mmap`函数创建的用来记录各种全局信息的模拟的共享内存，后面没有特殊说明提到共享内存指的是后者)

- `tables-biarch.c`：同时使用64位和32位两种架构的系统调用

- `tables-uniarch.c`：只使用一种架构的系统调用

- `tables.c`：对`tables-biarch.c`和`tables-uniarch.c`的封装

可以通过命令行参数指定一种架构。

![](upload/attach/202002/734571\_5KK232XE7QZHV2F.png)

![](upload/attach/201903/734571\_4RC3QWG9N4TM8JV.png)

- `taint.c`：taint功能

- `trinity.c`：执行一些初始化操作并调用`main.c`中的`main_loop`函数

- `uid.c`：uid的初始化和检查等功能

- `utils.c`：一些辅助函数

`trinity`文件夹下含有源代码文件的子目录如下。

- `childops`：该目录下有四个文件，目前用到的只有`random-syscall.c`这一个文件，其它的在`child.c`中被注释掉了。`random_syscall`函数当然就是随机进行系统调用了，后面再详细介绍

![](upload/attach/202002/734571\_EDCEVS3FKVCQ7Q2.png)

- `fds`：前面已经介绍

- `include`：用到的头文件

- `ioctls`：ioctl相关fuzz。每组ioctl都用`ioctl_group`结构体表示，`ioctls`中的`ioctls.c`提供`get_random_ioctl_group`等函数，通过`syscalls`中的`ioctl.c`中的`syscallentry`结构体进行调用

![](upload/attach/202002/734571\_CSNQFJSNQSPP78A.png)

![](upload/attach/202002/734571\_BJV588DTNWZRVER.png)

![](upload/attach/202002/734571\_KRFJRGGD2RGEAP8.png)

`syscallentry`结构体中的`sanitise`函数用来在生成系统调用参数之后对参数进行调整。这里就是调用的`sanitise_ioctl`函数，`sanitise_ioctl`函数会调用`ioctl_group`结构体中的`sanitise`函数，对于`sgx_grp`调用的是`pick_random_ioctl`函数从这一组ioctl中随机选择一个。

![](upload/attach/202002/734571\_WXJX6JACRKHZ9PH.png)

- `mm`：该目录下有四个文件，`maps-initial.c`创建初始的内存映射，每个子进程随后会将它们复制为自己的私有副本

![](upload/attach/202002/734571\_X4KP6D2D935CHVB.png)

![](upload/attach/202002/734571\_4X8KTYED3T42KUC.png)

![](upload/attach/202002/734571\_7GWJPVHJB5BH3VD.png)

`fault-write.c`和`fault-read.c`分别提供`random_map_writefn`函数和`random_map_readfn`函数供`dirty_mapping`函数随机对映射做一些操作。前面可以看到创建子进程调用`init_child_mappings`函数之后就会调用`dirty_mapping`函数，此外在`mmap`和`mmap2`系统调用返回之后也会随机调用`dirty_mapping`函数。

![](upload/attach/202002/734571\_W8H8UHJQBN2HYY7.png)

![](upload/attach/202002/734571\_JJGYH7A3ZW675GW.png)

![](upload/attach/202002/734571\_6NW42ERX6WGW22R.png)

`syscallentry`结构体中的`post`函数在系统调用返回之后被调用。

![](upload/attach/202002/734571\_ED575XXW589MBNP.png)

`maps.c`提供了前面所说的`dirty_mapping`函数和`init_child_mappings`函数，还有其它一些相关函数。

- `net`：net相关fuzz。和`ioctls`差不多，在`syscalls`中的`send.c`/`setsockopt.c`/`socket.c`中调用

- `rand`：生成随机数，随机长度，随机地址等

- `server`：前面已经介绍

- `syscalls`：被fuzz的系统调用，使用`syscallentry`结构体描述

- `tools`：只有一个`analyze-sockets.c`源代码文件，用来分析socket文件

# trinity-main

下面我们从`trinity.c`的`main`函数开始分析。

首先设置最大子进程数`max_children`为CPU核心数的4倍。然后处理参数，除了前面说到的还有几个比较有用的选项，比如`-c`表示fuzz指定的系统调用；`-N`表示指定fuzz的系统调用数量；`-V`接受目录参数，程序随机打开该目录中的文件，并将生成的文件描述符传递给系统调用，有助于发现特定文件系统类型中的漏洞等等。

接下来创建并初始化共享内存区域。共享内存区域`shm_s`结构体定义在`include\shm.h`中，该结构体中的`children`是一个二维数组，数组中的每一个元素都是指向子进程使用的`childdata`结构体的指针。

![](upload/attach/202002/734571\_7Q4XDC697P8ARAA.png)

![](upload/attach/202002/734571\_TXT7F3J8EDXNK4H.png)

![](upload/attach/202002/734571\_X43FDCV3EMFWQ6Z.png)

初始化系统调用，整体架构中已经提到了系统调用是通过`include\syscallentry.h`中的`syscallentry`结构体定义的，`syscalltable`结构体中的`entry`是一个指向`syscallentry`结构体的指针，所以通过指向`syscalltable`结构体的指针`table`通过`table[i].entry`可以取到所有的`syscallentry`。在`include`目录下`arch-xxx.h`定义了不同架构的`syscalltable`。

![](upload/attach/201903/734571\_EPXFREAEREVEZ2B.png)

比如当前操作系统是`__x86_64__`，那么include的是`arch-x86-64.h`。

![](upload/attach/201903/734571\_QB2BCB5UQT2GFPW.png)

在`arch-x86-64.h`里面定义了当前同时使用64位和32位两种架构的系统调用。

![](upload/attach/201903/734571\_2P8GXRUJVWSBKDA.png)

![](upload/attach/201903/734571\_AT5GSYWB4BMGBCM.png)

在`syscallentry`结构体中可以看到每个系统调用中包含系统调用名、参数个数、返回值类型和每一个参数的参数名、类型、取值范围等等。

![](upload/attach/202002/734571\_WT9X33YRRB55ZB3.png)

![](upload/attach/202002/734571\_627MK2QYGBEQ8ZA.png)

在初始化系统调用的过程中会调用它们的`init`函数。只有`perf_event_open`这个系统调用定义了相应的`init`函数。

![](upload/attach/202002/734571\_MSAJJ794ZN36P6Q.png)

![](upload/attach/201903/734571\_46NEHGTWHBE4GGH.png)

初始化文件描述符，整体架构中已经提到了`objects.c`管理系统调用中用到的文件描述符，每一种文件描述符具体的操作在`fds`文件夹中，都用`fd_provider`结构体表示，初始化时调用它们的`open`函数。

![](upload/attach/201903/734571\_XRDNB4Y6C9NMHEA.png)

![](upload/attach/201903/734571\_DF3HRE85U7WA2RM.png)

还有其它的一些初始化操作，之后重点就在于`main_loop`函数，从`main_loop`函数退出以后执行一些清理善后的操作。在`main_loop`函数中首先记录下`main`函数已经开始，然后调用`fork_children`函数创建执行系统调用的子进程。在`while`循环中，只要子进程还在运行，就一直执行下面这些操作：

1.首先在`handle_children`函数中等待子进程停止的信号。如果1秒之后没有接收到则返回。如果接收到则找到相应的子进程然后调用`handle_child`函数处理。

![](upload/attach/201903/734571\_XU2KNNPW6GJEZAV.png)

在`handle_child`函数中如果子进程是正常终止则记录该子进程已经退出，删除它的所有有关引用，重新创建子进程；如果子进程是异常终止或者暂停则调用相应的处理函数`handle_childsig`，`handle_childsig`函数除了做一些记录，其它的处理和`handle_child`函数大致相同。

![](upload/attach/201903/734571\_MBD6Y742BEZVF4G.png)

2.检查内核是否taint，是则调用`stop_ftrace`函数停止ftrace并调用`post-mortem.c`中的`tainted_postmortem`函数，整体架构中已经提到了其功能是检测到taint之后向`trinity-post-mortem.log`写入每个子进程最后一个系统调用信息。

![](upload/attach/201903/734571\_P7GWJZ48RCGHVZZ.png)

3.检查共享内存区域是否损坏，是否持有共享内存区域的锁和每一个子进程使用的`childdata`结构体中的`syscallrecord`的锁。

![](upload/attach/201903/734571\_5N2RBS54UGA5WR2.png)

4.如果通过`-N`参数设定了fuzz系统调用的数量那么检查是否达到该数量。

![](upload/attach/201903/734571\_6T7MGNBT7DN54X4.png)
