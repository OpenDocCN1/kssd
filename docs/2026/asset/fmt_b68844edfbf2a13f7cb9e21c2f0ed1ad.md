
# [原创]强网杯出题思路-solid_core-HijackPrctl

[toc]

## 0x00 概述

`solid_core`出题时候，基本上是以`CSAW-2015-CTF`的`stringipc`题目为基础进行`内存读写`的限制，`CSAW-2015-CTF`中的题目是一个`krealloc()`的利用。
在`krealloc()`的定义中`mm/slab_common.c`中 ![krealloc](upload/attach/201803/768814_A699HCGNUEGJSKD.png)
如果`size = 0`，`krealloc()`返回值为`0x10`
![krealloc](upload/attach/201803/768814_NGAJYNFJFC4U99V.png)
通过修改`size = -1`，使得`krealloc()`的返回值变成`0x10`，同时`size`因为是`0xFFFFFFFFFFFFFFFF`所以可以进行任意地址读写。
![krealloc](upload/attach/201803/768814_B5T2RJMKS89P99D.png)
那么传统思路下一步提权的方法有两种，下面进行介绍。

## 0x01 任意地址读写到权限提升(传统思路)

下面以`csaw-2015`的`stringipc`为例子来介绍两种从任意地址读写到权限提升到权限的传统思路

### 1. 爆破cred结构位置并篡改

做过内核漏洞利用的同学应该都了解`task_struct`中的`cred`结构代表了该进程的`权限结构`。
![krealloc](upload/attach/201803/768814_GER2JBZYXDP4JN8.png)
如果我们能够修改`cred`结构的值那么就可以进行提权操作。这是一个很正常的思路，但是我们的`cred`结构地址在哪里呢？这里`CSAW`给出的思路是通过`prctl()`设置`comm`结构为一个`Random`的字符串是，然后通过爆破这个`Random`的字符串，每八个字节进行遍历，耗时比较久，但是是可行的。
链接如下 <mark class="encrypted">8a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8X3y4G2M7s2m8G2L8r3q4Q4x3V1k6e0N6s2u0A6L8X3N6u0f1p5y4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7$3!0D9N6i4c8A6L8$3&6Q4x3V1k6K6L8$3I4#2N6r3W2G2L8W2)9J5k6h3x3`.</mark>

### 2. RET2DIR攻击(劫持VDSO)

第二种方法是使用`RET2DIR`攻击，在这里`CSAW-2015-stringipc`可以通过攻击`VDSO`来劫持用户态的代码执行流程。下面我们先来补充一下`VDSO`的知识：

#### 1.linux下的VDSO

`VDSO`(`Virtual Dynamically-linked Shared Object`)是个很有意思的东西, 它将内核态的调用映射到用户态的地址空间中, 使得调用开销更小, 路径更好.
开销更小比较容易理解, 那么路径更好指的是什么呢? 拿x86下的系统调用举例, 传统的`int 0x80`有点慢, Intel和AMD分别实现了`sysenter`、`sysexit`和`syscall`、`sysret`, 即所谓的快速系统调用指令, 使用它们更快, 但是也带来了兼容性的问题. 于是Linux实现了`vsyscall`, 程序统一调用`vsyscall`, 具体的选择由内核来决定. 而`vsyscall`的实现就在`VDSO`中.
Linux(kernel 2.6 or upper)环境下执行`ldd /bin/sh`, 会发现有个名字叫`linux-vdso.so.1`(`老点的版本是linux-gate.so.1`)的动态文件, 而系统中却找不到它, 它就是`VDSO`. 例如:
```
$ ldd /bin/sh
linux-vdso.so.1 => (0x00007fff2f9ff000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f28d5b36000)
/lib64/ld-linux-x86-64.so.2 (0x00007f28d5eca000)
```
所以，不难理解，`VDSO`是一段内核空间的代码，用来提供给用户态下更快地调用系统调用。正是因为这个特殊的机制，使得我们可以进行攻击。
在`CSAW-CTF-2015`的`stringipc`题目中，内核态下`VDSO`页的权限是`RW`，这个代码页会被直接映射到用户态下的进程中，以满足`gettime`等频繁系统调用的速度。这个代码页映射到用户态的权限属性是`RX`，就是可以执行。所以，我们可以通过在内核态下通过任意写来修改`VDSO`的代码，譬如我们修改代码成为`prepare_cred`+`committed_cred`等，这样在用户调用`VDSO`时，就可以劫持控制流了。
具体链接如下: <mark class="encrypted">b75K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2@1M7%4A6F1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8W2)9K6c8Y4m8Q4x3@1b7J5x3b7`.`.</mark>
这里也有一个问题:`VDSO`位置在哪里呢？这个可以爆破，因为我们有了任意地址读的权限，不过和上面爆破`cred`结构的技术不一样，我们可以更快的爆破，因为`VDSO`必定是被安防到一个内存页里面，也就是页对其的，同时它是一个`ELF`文件，是有`ELF Signurte`，所以我们可以按照内存页的偏移来进行爆破，这样爆破速度会很快，大概是`256`倍,而且它的映射位置离内核基址并不是太远，可以很快就出来了。

## 0x02 题目修改思路

题目为了限制大家不能使用`cred覆盖`和`ret2dir`攻击利用方法，我对`csaw-ctf-2015`进行了修改，限制了内存读写范围。
我进行了限制，限制写入范围必须大于`0xffffffff80000000`，这个地址在`linux`内存分布中，是`kernel base`以上。
附上`linux 内核内存分布图`
![krealloc](upload/attach/201803/768814_3GHS4DNBE4KVPM3.png)
所以可以限制`cred被覆盖`，同时编译最新内核以限制`VDSO`在内核情况下不能被修改。
![krealloc](upload/attach/201803/768814_4CEDBJR55B9DS7X.png)
可以使得上两种的利用方法失效，以期待有高手能够给出一种另外的利用思路，同时我也给出了一种限制之后可以成功利用思路，也就是`HijackPrctl`，下面进行详细介绍。

## 0x03 HijackPrctl- A reliable linux root Techniques

下面介绍从内核地址任意读写(或者可以限制为局部地址读写)到任意代码执行的一种新型的`Reliable Linux Root`技术，这个思路其实也不算最新的，是来自于`INetCop Security` 分享的`New Reliable Android Kernel Root Exploitation Techniques`,这种漏洞利用思路主要是要劫持`Prctl`函数，最后跳转到`call_usermodehelper`,我姑且把这种方法称作`HijackPrctl`。
我们再来确定一下现在能做的事情,我们可以进行局部地址的任意读写，题目限制了读写范围必须大于`0xffffffff80000`也就是`kernel base`以上。为了达到代码执行，我们先找一个内核函数看能不能劫持的。

### 0x01 Prctl(用户和内核沟通的一个绝佳函数)

这里要介绍一下`Prctl`函数，这个函数可以对进程进行一些设置，通过查看`linux内核`源码，可以知道这个函数是一个内核漏洞利用中的绝佳函数。
在`include/linux/security.h`中可以看到`cap_task_prctl`拥有6个参数。
![krealloc](upload/attach/201803/768814_E7W36VQZMQN4E6J.png)
同时在`kernel/sys.c`中，可以看到对于`prctl`这个系统调用的处理过程中，将参数原封不动的传给了`security_task_prctl()`函数进行处理。
![krealloc](upload/attach/201803/768814_WCKEENDY4MC72Q5.png)
继续往下跟进，我们发现`security_task_prctl()`这个函数其实最后是定位到了一个虚表里面去，
![krealloc](upload/attach/201803/768814_P6DQEXPHJHCZ6QY.png)
在调试时候我们会找到这个`hook`的位置在哪里，在这里是`capability+0x520+0x18`这个偏移，这个偏移在`IDA`中也能分析出来。
![krealloc](upload/attach/201803/768814_GBNQSYTFYYYGVNC.png)
![krealloc](upload/attach/201803/768814_3U8VWQNY66GN7UN.png)
这样的话，我们就找到了一个可以通过用户态传最多6个参数并且到内核态原封不动执行的虚函数。也就是意味我们可以通过修改这个指针，可以任意执行一个函数了。

### 0x02 32位和64位区别

基于上面的分析我们可以通过用户态传递参数，在内核态下任意执行6个参数以下的函数。再仔细分析一下，对吗？`在32位下是可以的，在64位下并不是`。我们再看这个函数
`int security_task_prctl(int option, unsigned long ...)`发现第一个参数`option`是`int`类型，也就是我们传入的`64位会被截断成为32位`，这也就导致了64位，第一个参数不能用了。
这个不影响32位下的漏洞利用，我们继续来分析32位如何利用。
其实，这里有很多思路，`INetCop Security`的`Slide`里面有，我这里介绍另外一种基于`VDSO`变形的方法，也是看雪论坛上(https://bbs.pediy.com/thread-220057.htm)，提出的一种方法。
具体思路是:
1. 先通过劫持`task_prctl`，将其修改成为`set_memory_rw`
2. 然后传入`VDSO`的地址，将`VDSO`修改成为可写的属性，
3. 然后之后的步骤就和劫持`VDSO`方法是一样的了。
这种方法我进行了验证`64位内核`上是不可行的，就是因为第一个参数被截断的原因。

### 0x03 call_usermoderhelper内核线程执行

`call_usermodehelper`是内核运行用户程序的一个`api`,并且该程序有`root`的权限。如果我们能够控制性的调用它，就能以`Root`权限执行我们想要执行的程序了。
定义在`kernel/umh.c`中
![krealloc](upload/attach/201803/768814_9EKNXZPVWV49ZVD.png)
`subprocess_info`如下
![krealloc](upload/attach/201803/768814_HWFCJD8GGU5XX8Z.png)
我们要劫持`task_prctl`到`call_usermodehelper`吗，不是的，因为这里的第一个参数也是`64位`的，也不能直接劫持过来。但是内核中有些代码片段是调用了`Call_usermodehelper`的，可以转化为我们所用。
譬如`kernel/reboot.c`中的`orderly_poweroff`函数中调用了`run_cmd`参数是`poweroff_cmd`,而且`poweroff_cmd`是一个全局变量，可以修改。
![krealloc](upload/attach/201803/768814_JKKXQBND4TFHCRB.png)
那么我们就先篡改`poweroff_cmd`=我们预期执行的命令，然后直接劫持`task_prctl`到`orderly_poweroff`函数，这样就任意命令执行了，同时按照`INetCop Security`给出的思路，需要先关闭`selinux`。
所以再整理一下整体思路:
1. 利用`kremalloc`的问题，达到`任意地址读写`的能力
2. 通过快速爆破，泄露出`VDSO`地址。
3. 利用`VDSO`和`kernel_base`相差不远的特性，泄露出`内核基址`
4. 篡改`prctl`的`hook`为`selinux_disable`函数的地址
5. 调用`prctl()`使得`selinux`失效
6. 篡改`poweroff_cmd`使其等于我们预期执行的命令。
7. 篡改`prctl`的`hook`为`orderly_poweroff`
8. 调用`prctl()`执行我们预期的命令，达到内核提权的效果。
最后给出利用程序如下
