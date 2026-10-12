
# 分享比较完整的 ROOTKIT DEMO! 原来 Shadow Hook 和 SSDT Hook 一样容易！

> 原文：[`www.pediy.com/kssd/pediy10/70083.html`](https://www.pediy.com/kssd/pediy10/70083.html)

这里写的 `ROOTKIT` 比较简单(有些代码是消化别人的代码后改写过来的)，高手.......

包含以下内容：(详细请看源代码)

## SSDT Hook

```
//hook system call

#define HOOK_SYSCALL(FuncName, pHookFunc, pOrigFunc ) \
          pOrigFunc = (PVOID)InterlockedExchange( \
          (PLONG)&MappedSystemCallTable[ SYSCALL_INDEX(FuncName) ], \
          (LONG)pHookFunc)

//unhook system call

#define UNHOOK_SYSCALL(FuncName, pHookFunc, pOrigFunc ) \
          InterlockedExchange( \
          (PLONG)&MappedSystemCallTable[ SYSCALL_INDEX(FuncName) ],\
          (LONG)pOrigFunc)
```

`SSDT` HOOK 了如下函数：

* `ZwQueryValueKey`
* `ZwEnumerateValueKey`
* `ZwQueryDirectoryFile`
* `ZwOpenProcess`
* `ZwDeleteKey`
* `ZwDeleteValueKey`
* `ZwSaveKey`
* `ZwLoadDriver`
* `ZwSetSystemInformation`
* `ZwTerminateProcess`

## Shadow Hook

仿照 `SSDT` HOOK，下面定义两个宏，让 `Shadow` Hook 和 `SSDT` Hook 一样简单!

```
//hook shadow system call

#define HOOK_SHADOW_SYSCALL(SysCallIndex, pHookFunc, pOrigFunc ) \
          pOrigFunc = (PVOID)InterlockedExchange( \
          (PLONG)&MappedSystemCallTable[ (SysCallIndex) ], \
          (LONG)pHookFunc)

//unhook shadow system call

#define UNHOOK_SHADOW_SYSCALL(SysCallIndex, pHookFunc, pOrigFunc ) \
          InterlockedExchange( \
          (PLONG)&MappedSystemCallTable[ (SysCallIndex) ],\
          (LONG)pOrigFunc)
```

上周一接到腾讯的电话面试，由于有一段时间没研究 HOOK 了，问到 `Shadow` Hook 时没回答好！汗！

现在把 `Shadow` Hook 重新整理了一下！

`Shadow` Hook 了如下函数，程序框架比较好，容易加入新挂钩函数

* `NtUserFindWindowEx`
* `NtUserGetForegroundWindow`
* `NtUserQueryWindow`
* `NtUserBuildHwndList`
* `NtUserWindowFromPoint`
* `NtUserSetWindowsHookEx`
* `NtUserGetDC`
* `NtUserGetDCEx`
* `NtUserSendInput`

为了保护进程，研究了终止进程的方法

* `WINDOWS` 内核定时器
* 老土的文件/目录隐藏
* 注册表键值隐藏
* 驱动隐藏
* 系统线程
* `IRP` 文件操作
* 多种加载内核级 `ROOTKIT` 方法
* `Ring3` 中恢复 `SSDT`(`ZwSystemDebugControl`)
* 从资源释放文件
* 远程进程注入
* 消息钩子注入 DLL
* 查找窗口
* 查找进程

注意：不要随便运行程序，最好在虚拟机下运行！此程序仅供学习 `WINDOWS` 内核驱动编程用！

在自己的 `ntddk.h` 中的结构 `IO_STACK_LOCATION` 添加如下代码才能顺利通过编译:

```
//Parameters for IRP_MJ_DIRECTORY_CONTROL
struct {
   ULONG Length;
   PUNICODE_STRING FileName;
   FILE_INFORMATION_CLASS POINTER_ALIGNMENT \
   FileInformationClass;
} QueryDirectory;
```

下面两图分别是程序启动信息和在 `Ring0` 恢复 `SSDT`:

![](img/fc3f3e61580e6230618ac3a6233928e9.jpg)

![](img/7d0435777b3d03509c12c8f66f64a49b.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `CCRootkit-V0.1.rar` [解压密码：`pediy`] |
| :--- | :--- |
