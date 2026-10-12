
# CsrssWalker 学习笔记 (附源代码)

> 原文：[`www.pediy.com/kssd/pediy10/89708.html`](https://www.pediy.com/kssd/pediy10/89708.html)

学习了一下古老的 `CsrssWalker`，写点笔记。

在 `Csrss.exe` 中，保存着所有 Win32 子系统进程的进程信息，这些信息以链表的形式保存。

正常情况下，每一个新创建的进程都会通知 `Csrss.exe`，`Csrss.exe` 接收这些信息然后保存起来，所以遍历这个链表就可以得到所有 Win32 子系统进程的信息。首先就是找链表头了，链表头为 `CsrssRootProcess`，在 `CSRSRV.DLL` 导出的函数中有对 `CsrssRootProcess` 的操作，因此可以通过 `CSRSRV.DLL` 的导出函数找到 `CsrssRootProcess`。

比较方便一点的，是从 `CsrLockProcessByClientId` 中找到 `CsrssRootProcess`。

代码:

```
mov     edx, [ebp+arg_4]
and     dword ptr [edx], 0
mov     esi, dword_75AA891C ; Base=75AA0000
add     esi, 8
mov     [ebp+arg_4], 0C0000001h
```

`75AA891C` 处就是 `CsrssRootProcess` 的指针了，这个指针的值可以通过特征匹配找到，具体请参考代码。读取其内容，得到 `CsrssRootProcess=0x001629C0` (这个只是我的系统上的)

从这里读就可以得到 `CsrssRootProcess` 的内容了，然后遍历 `Link` 即可

涉及到的一些数据结构在 `CsrssStruct.h` 中。

进程信息的结构如下：

代码:

```
typedef struct _CSR_PROCESS
{
    CLIENT_ID ClientId; //这里可以得到进程 PID 和主线程 TID
    LIST_ENTRY ListLink; //就是这个链表
  LIST_ENTRY ThreadList;
    struct _CSR_PROCESS *Parent;
    PCSR_NT_SESSION NtSession;
    ULONG ExpectedVersion;
    HANDLE ClientPort;
    ULONG_PTR ClientViewBase;
    ULONG_PTR ClientViewBounds;
    HANDLE ProcessHandle;
    ULONG SequenceNumber;
    ULONG Flags;
    ULONG DebugFlags;
    CLIENT_ID DebugCid;
    ULONG ReferenceCount;
    ULONG ProcessGroupId;
    ULONG ProcessGroupSequence;
    ULONG fVDM;
    ULONG ThreadCount;
    ULONG PriorityClass;
    ULONG Reserved;
    ULONG ShutdownLevel;
    ULONG ShutdownFlags;
    PVOID ServerData[];
} CSR_PROCESS, *PCSR_PROCESS;
```

但是因为这不是在当前进程中，所以每次都要先读取出来，遍历时与普通的遍历双链表操作稍有差别，但是也很容易实现。

具体的步骤为：

1. 找到 `Csrss.exe` 进程 (这个很简单，Vista 要注意有不止一个 `Csrss` 进程)
2. 遍历 `Csrss.exe` 中的模块列表，找到 `CSRSRV.DLL` 的基址。
3. 在 `CSRSRV.DLL` 中根据导出函数 `CsrLockProcessByClientId` 找 `CsrssRootProcess` 指针
4. 构建当前进程名称列表，为输出作准备
5. 根据 `CsrssRootProcess` 指针的地址，从 `Csrss.exe` 进程中读取和遍历每个进程的信息并输出

在 `Csrss.exe` 中同样还保存着所有 Win32 线程的信息，由于线程数目较多，`Csrss` 中采用 Hash 表的形式来保存线程信息。同样从 `CSRSRV.DLL` 的导出函数 `CsrLockThreadByClientId` 可以得到 `CsrThreadHashTable` 的地址，这是一个 Hash 表，定义为：

代码:

```
LIST_ENTRY CsrThreadHashTable[256];

Hash 算法为:

#define CsrHashThread(t) \
    (HandleToUlong(t)&(256 - 1))
```

很简单的算法，查找 `CsrThreadHashTable` 的方法及遍历方法与前面遍 `CsrssRootProcessLink` 基本相同，不多说。这部分我并未在代码中实现，有兴趣的自己写一写吧，很简单。

对于 Vista 等较新的系统，由于 Session 隔离，系统中会有不止一个 `Csrss` 进程，这样就需要对这几个 `Csrss` 进程都进行处理。就说这么多了，具体地看代码吧。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `CsrssWalker_Achillis_Src.rar`  

 |
