
```c
typedef
BOOLEAN
(\*fn\_PicoExceptionHandler)(
\_In\_ PEXCEPTION\_RECORD ExceptionRecord,
\_In\_ PKEXCEPTION\_FRAME ExceptionFrame,
\_In\_ PKTRAP\_FRAME TrapFrame,
\_In\_ ULONG Unknown,
\_In\_ KPROCESSOR\_MODE PreviousMode
);
BOOLEAN
InstallPicoExceptionHook(
\_In\_ fn\_PicoExceptionHandler ExceptionHandler
)
{
BOOLEAN Ret = FALSE;
NTSTATUS Status;
do
{
if (NULL == ExceptionHandler)
{
break;
}
UNICODE\_STRING FuncName;
RtlInitUnicodeString(&FuncName, L"PsRegisterPicoProvider");
\_PsRegisterPicoProvider = (fn\_PsRegisterPicoProvider)MmGetSystemRoutineAddress(&FuncName);
if (NULL == \_PsRegisterPicoProvider)
{
break;
}
\_pPspPicoRegistrationDisabled = SearchPspPicoRegistrationDisabled(\_PsRegisterPicoProvider);
if (NULL == \_pPspPicoRegistrationDisabled)
{
break;
}
PS\_PICO\_EXCEPTION\_ARG1 ProviderRoutines = { 0 };
ProviderRoutines.Size = 88;
ProviderRoutines.DispatchException = ExceptionHandler;
ProviderRoutines.ProtectedRanges[0] = 0;
ProviderRoutines.ProtectedRanges[1] = 0;
PS\_PICO\_EXCEPTION\_ARG2 PicoRoutines = { 0 };
PicoRoutines.Size = 96;
\*\_pPspPicoRegistrationDisabled = 0;
Status = \_PsRegisterPicoProvider(&ProviderRoutines, &PicoRoutines);
\*\_pPspPicoRegistrationDisabled = 1;
if (!NT\_SUCCESS(Status))
{
break;
}
Ret = TRUE;
} while (FALSE);
return Ret;
}
```

### 5.4 将目标进程设置为WSL子进程

我们现在注册好了`Pico`异常处理回调，但是该回调只能提前接管`WSL`子进程的异常，那么如何将目标进程变为`WSL`子进程呢？非常检查，只需要将`EPROCESS.PicoContext`字段成员设置一个任意非零值即可，笔者这里直接将其设置为1

```c
//0x840 bytes (sizeof)
struct \_EPROCESS
{
struct \_KPROCESS Pcb; //0x0
struct \_EX\_PUSH\_LOCK ProcessLock; //0x1c8
VOID\* UniqueProcessId; //0x1d0
// ...
VOID\* PicoContext; //0x640
// ...
};
```

```c
VOID SetProcessPicoContext(
\_In\_ PEPROCESS Process
)
{
ULONG PicoContextOffset = GetPicoContextOffset();
\*(PUCHAR)((PUCHAR)Process + PicoContextOffset) = 1;
}
```

### 5.5 利用Pico异常处理回调实现无痕Hook

实现以上步骤后，当目标进程发生用户态异常后，我们的`Pico`回调就能实现最高接管该异常，早于`内核调试器`、`用户调试器`,至于`VEH`、`SEH`更不必多说，此时我们可以将前文的`无痕Hook`实现，异常处理部分改动一下，就能实现更加隐蔽的无痕Hook效果，关键代码、效果图如下所示：

```c
BOOLEAN
ExceptionHandler(
PEXCEPTION\_RECORD ExceptionRecord,
PKEXCEPTION\_FRAME ExceptionFrame,
PKTRAP\_FRAME TrapFrame,
ULONG Unknown,
KPROCESSOR\_MODE PreviousMode
)
{
UNREFERENCED\_PARAMETER(ExceptionFrame);
UNREFERENCED\_PARAMETER(Unknown);
UNREFERENCED\_PARAMETER(PreviousMode);
if (g\_TargetProcess == PsGetCurrentProcess() && STATUS\_ACCESS\_VIOLATION == ExceptionRecord->ExceptionCode)
{
SHADOW\_EXCEPTION\_INFO ShadowInfo = { 0 };
if (!QueryShadowModule((ULONG64)ExceptionRecord->ExceptionAddress, &ShadowInfo))
{
return FALSE;
}
ULONG64 FuncOffset = (ULONG64)ExceptionRecord->ExceptionAddress - ShadowInfo.OriginalImageBase;
TrapFrame->Rip = ShadowInfo.ShadowImageBase + FuncOffset;
return TRUE;
}
return FALSE;
}
```

![Pico异常回调实现无痕Hook](image/1.59PsPicoDispatchException.png)

这部分代码已经整理好，Git仓库地址见文末

### 5.6 利用Pico异常回调实现反调试效果

既然我们目前可以最早接管目标进程的用户态异常，那么就很容易实现各种反调试手法

第一种常见的反调试思路是**破坏调试现场**，因为攻击方使用调试器分析我们的代码时，一个非常重要的过程就是看寄存器列表，反推参数的意义，但是暴力清空所有寄存器，这是一个非常明显、且强烈的反调试信号。这相当于提示调试者，当前程序有反调试措施，要先把反调试干掉！所以笔者的反调试的思路是，魔改一份寄存器现场，让寄存器中的内容看起来`非常正常`，但是调试者尝试断点、单步走时，就会出现各种乱七八糟的情况，误导调试者以为这个程序的数据结构怎么乱七八糟的，从而增加调试者的时间成本

第二种反调试的方式，就直接退出自身进程呗，代码如下：

调试者看到的效果就是，怎么断点一命中，程序就退出了？

```c
BOOLEAN
ExceptionHandler(
PEXCEPTION\_RECORD ExceptionRecord,
PKEXCEPTION\_FRAME ExceptionFrame,
PKTRAP\_FRAME TrapFrame,
ULONG Unknown,
KPROCESSOR\_MODE PreviousMode
)
{
UNREFERENCED\_PARAMETER(ExceptionFrame);
UNREFERENCED\_PARAMETER(Unknown);
UNREFERENCED\_PARAMETER(PreviousMode);
if (g\_TargetProcess == PsGetCurrentProcess() &&
(STATUS\_BREAKPOINT == ExceptionRecord->ExceptionCode || STATUS\_SINGLE\_STEP == ExceptionRecord->ExceptionCode))
{
// 结束自身进程，报一个栈溢出的假退出码
ZwTerminateProcess((HANDLE)-1, STATUS\_STACK\_OVERFLOW);
return TRUE;
}
return FALSE;
}
```

### 5.7 WSL子进程进程保护

笔者这里测试，将任务管理器进程`Process.PicoContext`置1，将其变为`WSL`子进程，此时使用普通的调试器，例如`x64Dbg`、`CE`是无法打开进程的

1. `x64Dbg`进程列表找不到目标程序
2. `CE`无法进程目标进程报错，如图所示

![图片描述](upload/attach/202609/953262\_HF8JUWYNM3ER75C.webp)

关于这种**进程保护**的原理，碍于当前文章篇幅问题，这里就不展开详细分析过程了，笔者之前在IDA中粗略分析了一下，原理如下：

> NtOpenProcess尝试打开进程时，会判断目标进程是不是WSL子进程，如果是就会返回错误码`STATUS\_ACCESS\_DENIED`，所以达到了进程保护的效果

## 6. 提前接管用户态、内核态异常方法

### 6.1 Win11 26H1

笔者在前文介绍了2种提前接管用户态异常的方法，但是局限在于仅能提前接管用户态异常，所以笔者这里将会介绍在`Win11 26H1`下，可以同时提前接管用户态、内核态异常的思路

还是回去分析我们的老熟人`KiDispatchException`函数，已知需要将异常派发到`内核调试器`时，会调用`KdTrap`或者`KdStub`函数，如图所示

![图片描述](upload/attach/202609/953262\_MWD6565YB8JG2VW.webp)

利用点在`KdTrap`中，大量的无关中间调用这里不展开分析，直到最后的`KeStallExecutionProcessor`函数

![图片描述](upload/attach/202609/953262\_BAWG55B5FB5CJPD.webp)

在`KeStallExecutionProcessor`函数中，会取出`HalpStallCounter`这个对象下`0x70`的函数指针，最后通过`CFG`发起一次间接函数调用，看到这里，对`ETW Hook`有所了解的朋友应该猜到劫持异常的原理了，没错，又是`Halxxx`指针导致的问题

已知不管是用户态还是内内核态异常，系统都先尝试将异常派发到内核调试器，此时替换`Halxxx`中的函数指针，并配合修改沿途的一些标志位，使得最终调用到我们处理回调中，让我们有机会提前接管异常。这个实现思路跟`ETW Hook`（或者`InfinityHook`）可以说非常相似了

![图片描述](upload/attach/202609/953262\_TKVJA5MM7PSN89R.webp)

![图片描述](upload/attach/202609/953262\_V5Q6BRGGEQGKX6S.webp)

### 6.2 Win11 25H2及以下

着手点还在`KdTrap`中，大量的无关中间调用这里不展开分析，直到最后的`KeQueryPerformanceCounter`函数

![图片描述](upload/attach/202609/953262\_K34R8R4MNFCUS7V.webp)

在``KeQueryPerformanceCounter``函数中，会取出`HalpPerformanceCounter`这个对象下`0x70`的函数指针，最后通过`CFG`发起一次间接函数调用，没错，又又是`Halxxx`指针导致的问题

![图片描述](upload/attach/202609/953262\_P74JZTGD87KFPB7.webp)

![图片描述](upload/attach/202609/953262\_MGDXD6Z3NPQ6V8Y.webp)

#### 6.2.1 特征码定位HalpPerformanceCounter

接下来开始简单实验，首先特征码定位到`HalpPerformanceCounter`对象

```c
ULONG64
SearchHalpPerformanceCounter()
{
static ULONG64 HalpPerformanceCounter = 0;
if (HalpPerformanceCounter)
{
return HalpPerformanceCounter;
}
ULONG64 Addr = SearchFeatureCode(
(PCCHAR)"ntoskrnl.exe",
(PCCHAR)".text",
(PCUCHAR)"\x48\x8B\x35\xFC\xE3\xD6\x00\x48\x89\x7C\x24\x28\x4C\x89\x74\x24\x20\x83\xBE\xE4\x00\x00\x00\x05\x0F\x84\x0A\x01\x00\x00\x83\xBE\xDC\x00\x00\x00\x40\x48\x8B\x9E\xC0\x00\x00\x00",
"xxx????xxxx?xxxx?xx?????xx????xx?????xxx????",
0
);
if (NULL == Addr)
{
return NULL;
}
ULONG64\* pHalpPerformanceCounter = DEREF\_RELATIVE\_ADDR(ULONG64\*, Addr, 3);
HalpPerformanceCounter = \*pHalpPerformanceCounter;
return HalpPerformanceCounter;
}
```

#### 6.2.2 替换Hal函数指针

搜索成功后，替换其中`0x70`处的Hal函数指针，替换为自己的异常处理函数：

```c
g\_HalpHvCounterQueryCounter =
(fn\_HalpHvCounterQueryCounter)\_InterlockedExchangePointer((volatile PVOID\*)(HalpPerformanceCounter + 0x70), HalpHvCounterQueryCounter\_Hook);
```

```c
\_\_int64
HalpHvCounterQueryCounter\_Hook(
PVOID Arg1,
PVOID Arg2
)
{
\_InterlockedIncrement64((volatile LONG64\*)&g\_Counter);
// 筛选出异常调用来源
if (ExceptionStackTrace())
{
\_InterlockedIncrement64((volatile LONG64\*)&g\_ExceptionCounter);
// 从堆栈中恢复丢失的ExceptionRecord、Context等
CONTEXT\* Context = NULL;
EXCEPTION\_RECORD\* ExceptionRecord = NULL;
ResumeExceptionInfoFromStack(&Context, &ExceptionRecord);
if (STATUS\_ACCESS\_VIOLATION == ExceptionRecord->ExceptionCode)
{
// ...
}
// 构造中断返回栈，恢复异常派发
ContinueExecution();
}
return g\_HalpHvCounterQueryCounter(Arg1, Arg2);
}
```

#### 6.2.3 堆栈回溯筛选出异常派发调用

由于我们的Hook点位于`KeQueryPerformanceCounter`，就导致大量的**非异常调用**来源，所以我们要需要回溯堆栈信息，筛选出异常派发来源，笔者这里堆栈回溯判定比较简单，在堆栈中找到`KeThawExecution`、`KdpReport`、`KiDispatchException`范围附近的返回地址，便认为本次调用来源属于异常派发，部分代码如下：
