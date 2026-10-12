
# 第二阶段第二题目的答案(ring3 下 10 种,ring0 下 5 种)(看雪金山逆向比赛)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-260.htm`](https://www.pediy.com/kssd/pediy09/pediy09-260.htm)

> 首先说下这个程序干嘛了~~~

> 释放驱动 HOOK 了 SSDT 表中的 `NtOpenProcess`,但是对参数过滤不严密,只过滤 `PROCESS_TERMINATE`, `PROCESS_WRITE`, `XXXX`(忘记了),,三种.然后比较的 PID 是放在全局变量中,可以通过发送 `DeviceIoControl` 改变这个值~

> 所以我们还是可以以其他的参数打开这个进程的(只要不含有那三个参数)~

> 不过大家似乎都不感兴趣这个题目,于是代码就不放了,想要可以直接给我消息.顺便要求刀客公开他的方法~~

## 方法列表

1. 使用 `FindWindow` 的方法,然后想窗口发送 `WM_CLOSE` 的消息

2. 找到进程的线程, `PostThreadMessage` 发送 `WM_QUIT` 消息结束主线程

3. `OpenThread` ---> `TerminateThread`

4. `RemoteExitProcess`,以 `PROCESS_CREATE_THREAD|VM_OPERATION` 方式打开进程,然后创建个远程线程,线程开始地址是 `ExitProcess` 参数 0 ,程序自行了断.

5. 首先确保进程能被打开(8 和 9 介绍),然后创建一个线程去 `DebugActiveProcess` 他,然后什么也不做退出线程,程序无人接管,于是挂掉~

6. 以 `VM_OPERATION` 方式打开进程, 然后调用 `NtUnmapViewOfSection` 卸载掉他的 `ntdll.dll` 这个时候程序还未崩溃,然后再随意发送个消息给它,它的程序一经调用直接就会异常,但是 `ntdll` 已经挂掉,无法用户空间异常处理,自行了断~(我认为这个方法应和往进程空间写垃圾数据属于同一方法)

7. 发送键盘 `ESC` 单击的消息(这个和发送窗口消息不能算同一个的).

8. 下面是重点要介绍的:-----------------------------

由于 `CrackMeApp` 打开了驱动的文件句柄,所以直接不能卸载驱动的.因此需要把这个句柄给关闭.

于是怎么去获取远程的文件句柄呢?首先想到的是 `DuplicateHandle`

```c
BOOL DuplicateHandle(
  HANDLE hSourceProcessHandle,  // handle to source process
  HANDLE hSourceHandle,         // handle to duplicate
  HANDLE hTargetProcessHandle,  // handle to target process
  LPHANDLE lpTargetHandle,      // duplicate handle
  DWORD dwDesiredAccess,        // requested access
  BOOL bInheritHandle,          // handle inheritance option
  DWORD dwOptions               // optional actions
);
```

先看最后一个参数:

`DUPLICATE_CLOSE_SOURCE`

Closes the source handle. This occurs regardless of any error status returned.

`DUPLICATE_SAME_ACCESS`

Ignores the `dwDesiredAccess` parameter. The duplicate handle has the same access as the source handle.

也就是说当我们选择 `DUPLICATE_CLOSE_SOURCE` 时,远程的句柄就会自动关闭了

第一个参数,我们可以直接 `OpenProcess(PROCESS_DUP_HANDLE ,XXX)`获得,

第二个参数比较难找,就是 `CrackMeApp` 进程里打开驱动的句柄,要想获得这个我们得借助个 `ntdll.dll` 中的一个 API, `ZwQuerySystemInformation` 这个可以获得整个系统中所有的句柄信息,我们可以全部找出来,然后判断哪些句柄是属于 `CrackMeApp` 进程的,并且 `ObjectTypeNumber` 类型为文件类型的.关于这个的具体实现可以看我的实现代码.

```c
typedef struct _SYSTEM_HANDLE_INFORMATION { // Information Class 16
  ULONG ProcessId;
  UCHAR ObjectTypeNumber;
  UCHAR Flags;  // 0x01 = PROTECT_FROM_CLOSE, 0x02 = INHERIT
  USHORT Handle;
  PVOID Object;
  ACCESS_MASK GrantedAccess;
} SYSTEM_HANDLE_INFORMATION, *PSYSTEM_HANDLE_INFORMATION;
```

这样 `CrackMeApp` 进程中通过 `CreateFile` 打开的句柄就全找到了,一般有三个,我们可以把它全部都关闭,也可以只关闭最后一个(因为最后就是打开驱动的句柄,打开驱动是最后一次调用 `CreateFile`).

这个我就得到了第二个参数,下面的参数都很简单了~

于是一调用这个 `DuplicateHandle` ,我们不仅关闭了远程的句柄,而且还获得了驱动的句柄,为所欲为了~~

于是我的第 8 个方法就是发送 IO 请求到 `CrackMeApp` 的驱动里,请求它修改被保护的进程 PID 为 0,这样我们就可以用常规方法结束它了(`Open+Terminate`)

9. 上面方法的远程句柄关闭了,我们就自然可以去卸载驱动了~于是这个就是去 `ZwUnloadDriver` 或者停止了它的 Service.这样再用常规方法结束~~

10. 所有的 win32 子系统的进程都会有一个句柄在 `csrss.exe` 进程里面,`CrackMeApp.exe` 自然也在,于是我们就可以把 `csrss.exe` 里面的句柄给 `dup` 过来自己使用,DUP 过来的方法大致和上面相同,不过判断的进程该为 `csrss`,类型要该为 5(进程句柄的类型),然后呢对每个进程句柄做一个 `ZwQueryInformationProcess()` 参数为 `BASIC_INFORMATION=0` 的查询,看看这个句柄是不是 `CrackMeApp` 的 ,如果是就直接拿来用,`TerminateProcess` 之~

> ---------------以下是用借助驱动来结束的-----------

11. 用驱动 patch 了它的保护的 PID 值,然后用户空间里常规方法~

12. 用驱动恢复 SSDT 中 `ZwOpenProcess`,这里我偷懒直接硬编码,调用那个驱动自身的恢复 SSDT 的函数,嘿嘿~~之后用户空间里常规方法(这个和 11 不算一种的)

13. 直接在驱动里调用 `NtOpenProcess` 绕过它的 SSDT HOOK 就可以了,不过应该注意的是需要先把 `KTHREAD` 结构体里的 `PreviousMode` 值改为 `KernelMode`,否则有可能会失败的~

获取句柄后调用 `ZwTerminateProcess`,这个不需要修改 `PreviousMode`,直接用就可以~

14. 和 13 的方法不同的是打开句柄的方法,使用了 `PsLookupProcessByProcessId+ObOpenObjectByPointer` 的方法,然后还是调用 `ZwTerminateProcess`~(这个也要算不同的,,因为规则上曾经说过 `ObXXX` 和 `NtXX` 算不同的 API 的)

15. 最后一种了,首先获取 `CrackMeApp` 线程的 TID,然后根据 `PsLookupThreadByThreadId` 获取 `EThread` 指针,然后向这个线程插入 APC 调用,在 APC 过程函数中,主要是执行的 `ZwTerminateProcess( 0xffffffff, 0 );`这个东西就是结束自己的意思(嘿嘿,,这个方法是我从 `IceSword` 里面抠出来的)~~本来想去调用 `Nt/ZwTerminateThread`,结果那个函数没导出还得自己找,于是索性就简单点 `ZwTerminateProcess` 了.

我的方法都较普通,跟你差不多

驱动中只用了 4 种方法

方法 11：

在驱动中 `NtOpenProcess` 取得句柄, 然后在驱动中 `TerminateProcess`

方法 12：

`PsLookupProcessByProcessId` 取得 Process，然后切换到进程空间，最后

`ZwTerminateProcess(NULL, 0);`

方法 13：

通过 `ObOpenObjectByPointer` 获取句柄，然后 `Terminate`

方法 14：

恢复 SSDT(`ZwOpenProcess`)，然后在应用层直接 `OpenProcess` 并 `Terminate`

> 5.首先确保进程能被打开(8 和 9 介绍),然后创建一个线程去 `DebugActiveProcess` 他,然后什么也不做退出线程,程序无人接管,于是挂掉~

> 10.所有的 win32 子系统的进程都会有一个句柄在 `csrss.exe`

> 这两个没想到,顶.

4dd466d0e1e0cc123afbe504da2462ca
