
# Android VM 调试与 JDWP 机制解析

## 一、`AndroidRuntime::startVm()` 与 JDWP 启动参数

在 Android 源码中，`AndroidRuntime::startVm()` 函数负责创建并启动 Java 虚拟机，其代码片段如下：

```cpp
// 601-602
int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv, bool zygote) {
  // ...
  // 773-779
  /*
   * Enable debugging only for apps forked from zygote.
   * Set suspend=y to pause during VM init and use android ADB transport.
   */
  if (zygote) {
    addOption("-agentlib:jdwp=transport=dt_android_adb,suspend=n,server=y");
  }
  // ...
  // 995-1001
  if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
    ALOGE("JNI_CreateJavaVM failed\n");
    return -1;
  }
  return 0;
}
```

所以直接转发 8100 端口会失败，因为 Android VM 的启动模式是 `dt_android_adb`，JDWP 线程也没有监听 8100 端口，这条日志给很多人带来了困扰。

最后，如果想要实现通过 socket 直接连接到 JDWP 线程，有没有办法呢？答案是有的。有两种可行的方式，一种是安装 hook 框架直接修改参数，但这种做法意义不大；另外一种是通过操作动态库的方式（`dlopen`、`dlsym`），修改调试启动参数，然后重启 JDWP 线程，这种适合实现生产环境下的远程调试。

## 二、`jdb connect` 命令在 SO 调试中的作用

`jdb connect` 命令是 Java VM 调试相关的命令，在 SO 调试中为什么要用它？它在和哪个模块进行通信？IDA 附加上进程之后为什么不能直接开始调试？

在前面一章中，我们明确了通过原生 JDWP 的方式调试与使用 IDA 调试是两个层面的调试手段。使用 `jdb` 的目的，是为了通知 VM 继续运行，因为应用以调试模式启动（以调试模式启动的原因是为了在程序早期下断点），启动之后一直在等待调试连接，`jdb` 通过 `dt_android_adb` 的方式，连接到了目标进程的 VM，VM 继续运行，这样 IDA 才可以在运行着的进程上进行调试。

> 归纳而言之，对 JDWP 调试手段的运用，是一种辅助手段，这种辅助手段使得我们能够在进程早期下断点。

## 三、`adb`、`android_server`、IDA 之间的关系

`adb`、`android_server`、IDA 之间的关系是什么？IDA 与应用进程之间的调试连接到底是如何建立的？IDA 调试 Android SO 的技术原理是什么？

根据前两个小节的内容，我们知道了 `adb` 套件实际上是在辅助建立调试连接：

a）在 JDWP 调试连接中，提供 `dt_android_adb` 模式所需的通信功能；

b）在 IDA 调试连接中，提供端口转发、远程启动进程的手段。

而 `android_server`、IDA 之间的关系，基本上与原生调试模型保持着一致，它是 IDA 对 Android 逆向调试的实现：

```text
____________________________________
|                                    |
|     IDA (host)                     |
|                                    |
|                      RemoteSocket  |
|                           ^^       |
|___________________________||_______|
                             ||
Transport                     ||
(TCP for emulator - USB for device)
                             ||
_____________________________||_____
|                           ||       |
|   android_server (device) ||       |
|                           VV       |
PROCESS <================>  ptrace <---->    RemoteSocket  |
|                                    |
|____________________________________|
```

`android_server` 的作用类似于 `adbd`，区别在于，`adbd` 通过本地 socket 与 JDWP 进行通信，以转发调试信号；而 `android_server` 此处实际上是基于 `ptrace` 实现的一个调试器，它一端通过 socket 与 IDA 相连接，传输调试指令和数据，另一端通过 `ptrace` 直接操控调试进程。

## 七、总结

经过以上几个章节的阐述，我希望已经讲清楚我提出的那些问题，也希望这篇文章能够达到它的目的。如果您在阅读过程中有任何问题，既可以在评论区留言，也可以发送邮件至 1054624587@qq.com 与我进一步交流。

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-3-9 21:50 被 kxliping 编辑，原因：脱敏处理

- [#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
- [#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
- [#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)
- [#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
- [#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)
- [#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
