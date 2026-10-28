
# [原创]驱动注入用户线程之跨session通知csrss之真正解决

首先这事得搜索网络，发现网络上的看起来很牛的文章，咱们得承认在之前是还算不错的，但是也没有真正实现。

或许有实现的，但是网上肯定是搜不到具体的，肯定都当私货处理了，拿到现在这年代来说。都算是被淘汰了。

为什么这么说呢，因为这得分具体情况。在vista以及以后的系统中，未导出函数`ZwCreateThreadEx`已经自己内置实现了

跨session 通知`csrss`等操作，所以用起来还是很省心的。填充好参数，直接就不管了，也很顺利的运行。

但是吧，咱们写程序就是为了兼容性（不然内心一万匹草泥马在翻滚）。在国内`xp`可还是一大堆的~

但是`xp`却没有`ZwCreateThreadEx`，只有`ZwCreateThread`

（请自行`ida`一下`xp`的`ntkrnlpa.exe`，

至于为什么是这个而不是`ntoskrnl.exe`当然这个`pchunter`看下驱动的加载就好了啊，本文不是扫盲篇，这些不具体解释，当然我也是大菜鸟）

然而`ZwCreateThread`这个家伙 可真是让人操碎了心。

因为按照`ZwCreateThread`的参数 我填充好之后，编译，然后直接往`taskmgr.exe`里面注入`dll`，而`dll`的代码就是`dllmain`里`MessageBox`了一下

大家可以想到结果是什么。

结果是：弹框出来了，哎呀，万事大吉，收工保存，抽只烟，然后休息下，真爽。

等过了些时候，打算用用了。于是就真正的写功能`dll`，至于`dll`的功能当然是有点小复杂。

然后测试下。

哎，不对啊，怎么有点问题。是不是我系统坏了，或者`dll`那里有问题？真是奇怪。

仔细检查`dll`，没发现啥错误啊，为啥结果不正确。

得仔细定位`dll`。加了点调试信息输出。最终定位到`dll`里 线程创建失败了？然后，又仔细核对了下创建线程的函数

也没发现什么问题啊。真是莫名其妙。

还是网上搜搜前辈们吧。这一搜不要紧，原来这事 绝没这么简单，复杂的在后面呢。

所以本文的重点是针对版本号`3790`（`2003`和`64`位`xp`）以及以下的所有的系统的兼容性，核心问题也就是：如何正确处理`ZwCreateThread`

那么到底如何正确处理呢？

我们为了解决这个问题，自然会网上搜，但是真正会思考这个问题的一定会联想到`CreateRemoteThread`这个`r3`的`api`因为这个大家肯定太熟悉了。

好吧，先说个题外话。鲁迅曾经说过："不想吃天鹅的蛤蟆不是好的蝌蚪"，还说过："电脑不备好`ida`,`windbg`,`od`,`wrk`,`reatos`的程序员不是好的厨师"

那么我最开始的想法是：先看看`reatos`里有没有，实在没有就`ida`逆`CreateRemoteThread`（反正有符号，没符号的话真没太大勇气逆）

天公做美竟然给搜出来了，路径在这里

`dll\win32\kernel32\thread\thread.c`

我们来看下内容

```c
HANDLE
WINAPI
CreateRemoteThread(HANDLE hProcess,
LPSECURITY_ATTRIBUTES lpThreadAttributes,
DWORD dwStackSize,
LPTHREAD_START_ROUTINE lpStartAddress,
LPVOID lpParameter,
DWORD dwCreationFlags,
LPDWORD lpThreadId)
{
NTSTATUS Status;
INITIAL_TEB InitialTeb;
CONTEXT Context;
CLIENT_ID ClientId;
OBJECT_ATTRIBUTES LocalObjectAttributes;
POBJECT_ATTRIBUTES ObjectAttributes;
HANDLE hThread;
ULONG Dummy;
DPRINT("CreateRemoteThread: hProcess: %ld dwStackSize: %ld lpStartAddress"
": %p lpParameter: %lx, dwCreationFlags: %lx\n", hProcess,
dwStackSize, lpStartAddress, lpParameter, dwCreationFlags);

/* Clear the Context */
RtlZeroMemory(&Context, sizeof(CONTEXT));

/* Write PID */
ClientId.UniqueProcess = hProcess;

/* Create the Stack */
Status = BasepCreateStack(hProcess,
dwStackSize,
dwCreationFlags & STACK_SIZE_PARAM_IS_A_RESERVATION ?
dwStackSize : 0,
&InitialTeb);
if(!NT_SUCCESS(Status))
{
SetLastErrorByStatus(Status);
return NULL;
}

/* Create Initial Context */
BasepInitializeContext(&Context,
lpParameter,
lpStartAddress,
InitialTeb.StackBase,
1);

/* initialize the attributes for the thread object */
ObjectAttributes = BasepConvertObjectAttributes(&LocalObjectAttributes,
lpThreadAttributes,
NULL);

/* Create the Kernel Thread Object */
Status = NtCreateThread(&hThread,
THREAD_ALL_ACCESS,
ObjectAttributes,
hProcess,
&ClientId,
&Context,
&InitialTeb,
TRUE);
if(!NT_SUCCESS(Status))
{
BasepFreeStack(hProcess, &InitialTeb);
SetLastErrorByStatus(Status);
return NULL;
}

/* Are we in the same process? */
if (hProcess == NtCurrentProcess())
{
PTEB Teb;
PVOID ActivationContextStack;
THREAD_BASIC_INFORMATION ThreadBasicInfo;
#ifndef SXS_SUPPORT_FIXME
ACTIVATION_CONTEXT_BASIC_INFORMATION ActivationCtxInfo;
ULONG_PTR Cookie;
#endif
ULONG retLen;

/* Get the TEB */
Status = NtQueryInformationThread(hThread,
ThreadBasicInformation,
&ThreadBasicInfo,
sizeof(ThreadBasicInfo),
&retLen);
if (NT_SUCCESS(Status))
{
/* Allocate the Activation Context Stack */
Status = RtlAllocateActivationContextStack(&ActivationContextStack);
}
if (NT_SUCCESS(Status))
{
Teb = ThreadBasicInfo.TebBaseAddress;
/* Save it */
Teb->ActivationContextStackPointer = ActivationContextStack;
#ifndef SXS_SUPPORT_FIXME
/* Query the Context */
Status = RtlQueryInformationActivationContext(1,
0,
NULL,
ActivationContextBasicInformation,
&ActivationCtxInfo,
sizeof(ActivationCtxInfo),
&retLen);
if (NT_SUCCESS(Status))
{
/* Does it need to be activated? */
if (!ActivationCtxInfo.hActCtx)
{
/* Activate it */
Status = RtlActivateActivationContext(1,
ActivationCtxInfo.hActCtx,
&Cookie);
if (!NT_SUCCESS(Status))
DPRINT1("RtlActivateActivationContext failed %x\n", Status);
}
}
else
DPRINT1("RtlQueryInformationActivationContext failed %x\n", Status);
#endif
}
else
DPRINT1("RtlAllocateActivationContextStack failed %x\n", Status);
}

/* Notify CSR */
Status = BasepNotifyCsrOfThread(hThread, &ClientId);
if (!NT_SUCCESS(Status))
{
ASSERT(FALSE);
}

/* Success */
if(lpThreadId) *lpThreadId = HandleToUlong(ClientId.UniqueThread);

/* Resume it if asked */
if (!(dwCreationFlags & CREATE_SUSPENDED))
{
NtResumeThread(hThread, &Dummy);
}

/* Return handle to thread */
return hThread;
}
```

我们来简化一下代码方便继续。

```c
CreateRemoteThread()
{
    //栈
    BasepCreateStack
    //上下文
    BasepInitializeContext
    //创建挂起线程
    NtCreateThread
    //判断是否自身,我们不管忽略过去,我们不会往csrss里面注
    //通知
    BasepNotifyCsrOfThread
    //恢复
    NtResumeThread
}
```

从上面的伪代码中我们可以看到除了通知这个，其他的都很好处理。所以我们直接将重点精力继续追下去，也就是说看`BasepNotifyCsrOfThread`的具体实现

当然还是`reatos`里面搜，路径如下

`dll\win32\kernel32\process\procsup.c`

内容如下:

```c
NTSTATUS
WINAPI
BasepNotifyCsrOfThread(IN HANDLE ThreadHandle,
IN PCLIENT_ID ClientId)
{
ULONG Request = CREATE_THREAD;
CSR_API_MESSAGE CsrRequest;
NTSTATUS Status;
DPRINT("BasepNotifyCsrOfThread: Thread: %lx, Handle %lx\n",
ClientId->UniqueThread, ThreadHandle);

/* Fill out the request */
CsrRequest.Data.CreateThreadRequest.ClientId = *ClientId;
CsrRequest.Data.CreateThreadRequest.ThreadHandle = ThreadHandle;

/* Call CSR */
Status = CsrClientCallServer(&CsrRequest,
NULL,
MAKE_CSR_API(Request, CSR_NATIVE),
sizeof(CSR_API_MESSAGE));
if (!NT_SUCCESS(Status) || !NT_SUCCESS(CsrRequest.Status))
{
DPRINT1("Failed to tell csrss about new thread\n");
return CsrRequest.Status;
}

/* Return Success */
return STATUS_SUCCESS;
}
```

一看就知道，实际上这个函数算是个小封装，没意思，重点还是在`CsrClientCallServer`里面。

那么继续搜，路径在

`dll\ntdll\csr\connect.c`

看下内容如下:

```c
NTSTATUS
NTAPI
CsrClientCallServer(PCSR_API_MESSAGE ApiMessage,
PCSR_CAPTURE_BUFFER CaptureBuffer OPTIONAL,
CSR_API_NUMBER ApiNumber,
ULONG RequestLength)
{
NTSTATUS Status;
ULONG PointerCount;
PULONG_PTR Pointers;
ULONG_PTR CurrentPointer;
DPRINT("CsrClientCallServer\n");

/* Fill out the Port Message Header */
ApiMessage->Header.u2.ZeroInit = 0;
ApiMessage->Header.u1.s1.DataLength = RequestLength - sizeof(PORT_MESSAGE);
ApiMessage->Header.u1.s1.TotalLength = RequestLength;

/* Fill out the CSR Header */
ApiMessage->Type = ApiNumber;
//ApiMessage->Opcode = ApiNumber; <- Activate with new CSR
ApiMessage->CsrCaptureData = NULL;
DPRINT("API: %lx, u1.s1.DataLength: %x, u1.s1.TotalLength: %x\n",
ApiNumber,
ApiMessage->Header.u1.s1.DataLength,
ApiMessage->Header.u1.s1.TotalLength);

/* Check if we are already inside a CSR Server */
if (!InsideCsrProcess)
{
/* Check if we got a a Capture Buffer */
if (CaptureBuffer)
{
/* We have to convert from our local view to the remote view */
ApiMessage->CsrCaptureData = (PVOID)((ULONG_PTR)CaptureBuffer +
CsrPortMemoryDelta);

/* Lock the buffer */
CaptureBuffer->BufferEnd = 0;

/* Get the pointer information */
PointerCount = CaptureBuffer->PointerCount;
Pointers = CaptureBuffer->PointerArray;

/* Loop through every pointer and convert it */
DPRINT("PointerCount: %lx\n", PointerCount);
while (PointerCount--)
{
/* Get this pointer and check if it's valid */
DPRINT("Array Address: %p. This pointer: %p. Data: %lx\n",
&Pointers, Pointers, *Pointers);
if ((CurrentPointer = *Pointers++))
{
/* Update it */
DPRINT("CurrentPointer: %lx.\n", *(PULONG_PTR)CurrentPointer);
*(PULONG_PTR)CurrentPointer += CsrPortMemoryDelta;
Pointers[-1] = CurrentPointer - (ULONG_PTR)ApiMessage;
DPRINT("CurrentPointer: %lx.\n", *(PULONG_PTR)CurrentPointer);
}
}
}

/* Send the LPC Message */
Status = NtRequestWaitReplyPort(CsrApiPort,
&ApiMessage->Header,
&ApiMessage->Header);

/* Check if we got a a Capture Buffer */
if (CaptureBuffer)
{
/* We have to convert from the remote view to our remote view */
```
