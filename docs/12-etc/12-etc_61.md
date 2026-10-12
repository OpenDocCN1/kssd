
# Android 提权代码 zergRush 分析

> 原文：[`www.pediy.com/kssd/pediy12/141661.html`](https://www.pediy.com/kssd/pediy12/141661.html)

原文地址：[`blog.claudxiao.net/2011/10/zergrush/`](http://blog.claudxiao.net/2011/10/zergrush/)

## 一、背景和原理

Revolutionary 工具开发小组在 2011 年 10 月发布了一个在 Android 2.2 和 2.3 上获得 root 权限的方法\[1\]，并公布了漏洞利用代码 `zergRush.c`\[2\]。`tomken_zhang` 已经在其博客上发表了两篇文章\[3\]\[4\]对其分析。本文做进一步梳理和补充。

产生漏洞的主要原因是：具有 root 权限的 `vold` 进程使用了 `libsysutils.so` 库，该库的一个函数存在栈溢出，因此可以在 root 权限执行输入的 shellcode。

存在漏洞的函数为 `FrameworkListener::dispatchCommand`，位于源码的 `\system\core\libsysutils\src\FrameworkListener.cpp` 中，其中的局部变量 `argv` 为固定大小的指针数组，当输入参数的数量超过其大小时，会越界写入栈中。

`zergRush.c` 成功地利用了这一漏洞，并进一步：

1、在 `/data/local/tmp/` 下增加一个置了 S 位的 shell；
2、使 Android 中后续启动的 adb 进程以 root 权限运行。

其中第二步的方法是：`adb` 进程最初以 root 运行，之后调用 `setuid()` 降低权限\[5\]。降权之前，会判断系统属性 `ro.kernel.qemu`，如果该属性位 1，则不降权。

## 二、函数功能概要

`die` 打印出错信息，退出程序
`copy` 将一个文件拷贝为另一个文件
`remount_data` 重新 mount 一个分区
`find_symbol` 查找 `libc.so` 中导出函数的内存地址
`check_addr` 确定一个地址中是否包含被禁止的字节
`do_fault` 构造溢出数据和 exploit，并通过 socket 发送给 `vold` 进程
`find_rop_gadgets` 从 `libc.so` 中寻找两个特殊指令序列的地址
`checkcrash` 调用 `do_fault`，判断其溢出产生的调试信息中是否包含 `sp`
`find_stack_addr` 调用 `do_fault`，从其溢出产生的调试信息中定位栈地址
`do_root` 将 shell 文件的 S 位置上，并设置 `ro.kernel.qemu` 属性为 1
`main` 主函数，完成漏洞利用的所有步骤

## 三、`main` 函数

395-396: 如果当前程序是以 root 权限运行，并且程序名为 `boomsh`，则调用 `do_root`，执行附加的两步操作
402-405: 将自身拷贝至 `/data/local/tmp/boomsh`，并设置其权限为 0711，将 `/system/bin/sh` 拷贝至 `/data/local/tmp/sh`。
407-408: 根据 `/system/bin/vold` 文件的大小获得其对应进程中堆的大概地址 `heap_addr`。
410-421: 根据系统版本对 `heap_addr` 做微调。如果不是 2.2 或 2.3 系统，退出。
423-428: 查询 `libc.so` 中 `system` 调用的地址，保存至 `system_ptr`。
430-443: 通过 `checkcrash` 函数，判断 `buffsz` 为 16 或 24 时能否成功利用。这里 `buffsz` 实际指 `libsysutils` 中造成栈溢出的指针数组 `argv` 的容量。
445-484: 调用 `find_stack_addr` 函数，确定栈地址。反复尝试五次，每次对堆地址 `heap_addr` 做微调，直至成功。判断得到的栈地址是否有效。
486-487: kill 掉当前的 `logcat` 进程，删除 `/data/local/tmp/crashlog` 文件。
489-491: 调用 `find_rop_gadgets` 函数，在 `libc.so` 中寻找指令序列 `add sp, #108; pop {r4-r7, pc}`，将地址保存在 `stack_pivot`；寻找指令 `pop {r0, pc}`，将地址保存在 `pop_r0`。
493-514: 尝试三次，每次调用 `do_fault`，之后判断 `/data/local/tmp/sh` 的 S 位是否置上，一旦置上，则利用成功；否则，微调栈地址 `heap_addr`（加减 16）。
516-533: 一旦利用成功，并且系统的 `ro.kernel.qemu` 属性已经被置为 1，则利用完成，重启的 adb 进程即可获得 root 权限。

## 四、`do_root` 函数

395-396: 若当前程序是以 root 权限执行的 `/data/local/tmp/boomsh`，则调用 `do_root` 函数。
379: 重新 mount 目录 `/data`。
380: 将 `/data/local/tmp/sh` 的所有者设置为 root。
381: 将 `/data/local/tmp/sh` 的属性设置为 04711，注意其 S 位被置位。
382: 设置系统的 `ro.kernel.qemu` 属性为 1。

## 五、`find_stack_addr` 函数

332-333: 清空 `logcat` 缓存，删除老的 `/data/local/tmp/crashlog` 日志文件。
335-340: 重启一个 `logcat`，将其日志输出至 `/data/local/tmp/crashlog` 文件。
342-349: 调用一次 `do_fault`，等待 3 秒后，读取 `crashlog` 文件中的 `logcat` 日志。
350-366: 搜索 `logcat` 日志中的 debug 信息，`" 4752455a"` 之前 8 个字节为栈基址 `stack_addr`，`" 5245564f"` 往之前 8 个字节为 `over`？，`" sp "` 之后 5 个字节为栈顶 `sp`。
370-371: `jumpsz = over - sp`

## 六、`do_fault` 函数

165: `buf` 是最后发送至 `vold` 的 shellcode。
169-181: `padding` 是 shellcode 中的一段填充内容，全部为 Z，无意义。长度为 `padding_sz + 12`。`padding_sz` 由 108 减去 `jumpsz` 计算得到。
183-184: 通过 socket 连到本地的 `vold` 进程。
186-190: 将栈地址 `stack_addr`、指令序列 1 地址 `stack_pivot`、指令序列 2 地址 `pop_r0`、`system` 调用地址 `system_ptr`、堆地址 `heap_addr`，分别填充到相应的字符串中。
192-198: 开始构造 shellcode。注意第 195 行，这里根据 `buffsz`，也就是尝试出来的溢出数组 `argv` 的大小，构造相应数量的输入参数。
200-201: 计算一下 `/data/local/tmp/boomsh` 字符串将会出现的地址，这个字符串会作为 shellcode 的一部分发到栈中，因此可以根据栈地址和偏移计算出来，最后作为 `system` 调用的参数。
208: 把上述地址转为字符串 `s_bsh_addr`。
209: 进一步构造 shellcode，包括栈地址、堆地址、填充、指令序列 2 地址、`boomsh` 字符串地址、`system` 调用地址、`boomsh` 字符串等。
214: 将 shellcode 发送至 `vold` 进程。

## 七、总结

综合来看，`zergRush.c` 的思路如下：

1. 计算出 `vold` 的堆地址
2. 查到 `system` 调用的地址
3. 尝试出栈缓冲区大小
4. 通过崩溃产生的调试信息，取得栈地址和栈结构信息
5. 在 `libc.so` 中找寻跳板指令
6. 根据缓冲区大小、栈结构和上述各种地址，构造出有效的 shellcode 来，发送到 `vold`
7. `shellcode` 在 `vold` 中以 root 权限运行，它通过 `system` 调用运行该利用程序的一个副本 `boomsh`
8. 程序副本 `boomsh` 以 root 权限运行时，会置上 shell 程序的 S 位，并设置系统属性 `ro.kernel.qemu`
9. 结束掉 `adb`，后续开启的 `adb` 进程将具有 root 权限

非常典型的缓冲区溢出利用思路，但与 PC 相比，利用了 android 中几个特殊之处：

1. `vold` 的溢出会在 `adb logcat` 中输出调试信息，这些信息说明了其内存结构，而其他程序可以读取到这些信息；
2. 在 ARM 架构下，跳板指令有了更多的选择，`ret2libc` 的攻击也可能更容易实现
3. `adb` 的降低权限过程又一次被利用。

最后，我们没有进一步分析 shellcode 的详细结构和跳转过程，难度已经不大。反而是 `libsysutils.so` 这个通用库中的溢出有没有可能造成其他问题，需要进一步分析。

## 八、参考链接

\[1\] ieftm. Revolutionary - zergRush local root 2.2/2.3. [`forum.xda-developers.com/show....php?t=1296916`](http://forum.xda-developers.com/showthread.php?t=1296916)
\[2\] revolutionary. `zergRush.c`. [`github.com/revolutionary/zer...ter/zergRush.c`](https://github.com/revolutionary/zergRush/blob/master/zergRush.c)
\[3\] `tomken_zhang`. 漏洞 zergRush. [`blog.csdn.net/tomken_zhang/ar...etails/6866260`](http://blog.csdn.net/tomken_zhang/article/details/6866260)
\[4\] `tomken_zhang`. 漏洞 zergRush (补充). [`blog.csdn.net/tomken_zhang/ar...etails/6870104`](http://blog.csdn.net/tomken_zhang/article/details/6870104)
\[5\] Claud Xiao. Android adb setuid 提权漏洞的分析. [`blog.claudxiao.net/2011/04/android-adb-setuid/`](http://blog.claudxiao.net/2011/04/android-adb-setuid/)
