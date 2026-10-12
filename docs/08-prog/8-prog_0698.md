
# Delphi 驱动开发研究之内核同步对象 Mutex

> 原文：[`www.pediy.com/kssd/pediy10/105373.htm`](https://www.pediy.com/kssd/pediy10/105373.htm)

在上篇教程中，我们讲解了内核同步对象中的计时器对象的使用方法，有关同步的另一个常见的用法就是对数据的独占访问。

在本教程中，我们将同时启动多个线程，所有这些线程都会数次对一个 `ULONG` 类型的共享变量进行累加操作，最终这个共享变量的值将会等于所有线程工作次数的总和。

这个共享变量可以是任意类型的数据，比如我们拦截到的系统服务（这些将在后面的教程里讨论）的相关统计数据。如果没有同步机制，这些共享数据的结果就无法预知了，我们可能得不到正确的统计数据甚至严重的话可能会导致系统崩溃。

解决上述问题最好的方案就是使用互斥（`Mutex`）对象了，在内核中，`Mutex` 又被称为突变体(mutants)。互斥，顾名思义，就是排它性访问，也就是同一时间内只允许一个线程访问共享数据，同一驱动的所有线程共同拥有一个 `Mutex` 对象，如果一个线程要访问共享变量，它首先要获取这个 `Mutex` 对象，一旦某个线程取得了 `Mutex` 对象，就可以对共享变量进行存取操作，否则就只能等待直到其他线程释放 `Mutex` 对象。通过 `Mutex` 机制就能确保同一时间只能有一个线程访问共享变量。

## 11.1 教程源码

代码:

```
unit MutualExclusion;
{$POINTERMATH ON}

interface

uses
  nt_status, ntoskrnl, fcall;

function _DriverEntry(pDriverObject:PDRIVER_OBJECT;
                      pusRegistryPath:PUNICODE_STRING): NTSTATUS; stdcall;

implementation

const
  { 不能超过 MAXIMUM_WAIT_OBJECTS (64)  - 最大等待对象数 }
  NUM_THREADS  = 5;
  NUM_WORKS    = 10;

var
  g_usDeviceName, g_usSymbolicLinkName: UNICODE_STRING;
  g_pkWaitBlock: PKWAIT_BLOCK;
  { PKTHREAD 数组 }
  g_apkThreads: array[0..NUM_THREADS - 1] of PVOID;
  g_dwCountThreads: DWORD;
  g_kMutex: KMUTEX;
  g_dwWorkElement: DWORD;

function ThreadProc: NTSTATUS; stdcall;
var
  liDelayTime:LARGE_INTEGER;
  pkThread: PVOID;  { PKTHREAD }
  dwWorkElement: ULONG;
  dwCount, dwTmp: ULONG;
begin
  pkThread := PsGetCurrentThread;
  DbgPrint('MutualExclusion: Thread %08X is entering ThreadProc'#13#10, pkThread);
  dwCount := 0;
  while dwCount < NUM_WORKS do
  begin
    DbgPrint('MutualExclusion: Thread %08X is working on #%d'#13#10, pkThread, dwCount);
    KeWaitForMutexObject(@g_kMutex, Executive, KernelMode,
                         False, nil);
    { 读取线程间共享资源的值 }
    dwWorkElement := g_dwWorkElement;
    { 这里做些其他的事情 }
    liDelayTime.HighPart := liDelayTime.HighPart or -1;
    liDelayTime.LowPart := -(rand shl 4);
    KeDelayExecutionThread(KernelMode, false, @liDelayTime);
    { 设置新的线程间共享资源的值 }
    Inc(dwWorkElement);
    g_dwWorkElement := dwWorkElement;
    KeReleaseMutex(@g_kMutex, False);
    dwTmp := (((ULONG(-liDelayTime.LowPart) * 3518437209) and $FFFF0000) shr 16) shr 13;
    DbgPrint('MutualExclusion: Thread %08X work #%d is done (%02dms)'#13#10,
             pkThread, dwCount, dwTmp);
    { 计数器加一，做下一次循环 }
    Inc(dwCount);
  end;
  DbgPrint('MutualExclusion: Thread %08X is about to terminate'#13#10, pkThread);
  Result := PsTerminateSystemThread(STATUS_SUCCESS);
end;

procedure CleanUp(pDriverObject:PDRIVER_OBJECT);
begin
  IoDeleteSymbolicLink(@g_usSymbolicLinkName);
  IoDeleteDevice(pDriverObject^.DeviceObject);
  if g_pkWaitBlock <> nil then
  begin
    ExFreePool(g_pkWaitBlock);
    g_pkWaitBlock := nil;
  end;
end;

procedure DriverUnload(pDriverObject:PDRIVER_OBJECT); stdcall;
begin
  DbgPrint('MutualExclusion: Entering DriverUnload'#13#10);
  DbgPrint('MutualExclusion: Wait for threads exit...'#13#10);

  { 因为 ThreadProc 存在于我们的驱动主体中，因此只要还有一个
  { 线程在运行，就不能卸载驱动，必须要等到所有的线程都退出 }
  if g_dwCountThreads > 0 then
  begin
    { 没有设置超时时间，一直等待到所有线程退出 }
    { 因为有多个线程对象，所以使用 KeWaitForMultipleObjects }
    KeWaitForMultipleObjects(g_dwCountThreads, @g_apkThreads,
                             WaitAll, Executive, KernelMode,
                             False, nil, g_pkWaitBlock);
    { 执行到这里时，所有线程均已退出，就可以释放所有线程对象了 }
    while g_dwCountThreads > 0 do
    begin
      Dec(g_dwCountThreads);
      ObfDereferenceObject(g_apkThreads[g_dwCountThreads]);
    end;
  end;
  CleanUp(pDriverObject);
  { 打印结果. 这里 g_dwWorkElement 的值应该等于 NUM_THREADS * NUM_WORKS }
  DbgPrint('MutualExclusion: WorkElement = %d'#13#10, g_dwWorkElement);
  DbgPrint('MutualExclusion: Leaving DriverUnload'#13#10);
end;

function StartThreads: NTSTATUS;
var
  hThread:HANDLE;
  i, dwCount:ULONG;
  rtnCode: NTSTATUS;
begin
  i := 0;
  { dwCount 保存实际运行的线程数 }
  dwCount := 0;
  while i < NUM_THREADS do
  begin
    { 启动 NUM_THREADS 个线程 }
    rtnCode := PsCreateSystemThread(@hThread, THREAD_ALL_ACCESS,
                                    nil, 0, nil,
                                    @ThreadProc, nil);
    if rtnCode = STATUS_SUCCESS then
    begin
      { 我们不需要 PsCreateSystemThread 返回的线程句柄. }
      { 但是我们需要指向它的指针. 以便我们引用线程对象并且关闭它. }
      ObReferenceObjectByHandle(hThread, THREAD_ALL_ACCESS,
                                nil, KernelMode,
                                @g_apkThreads[dwCount], nil);
      ZwClose(hThread);
      DbgPrint('MutualExclusion: System thread created. Thread Object: %08X'#13#10,
               g_apkThreads[dwCount]);
      Inc(dwCount);
    end else
    begin
      DbgPrint('MutualExclusion: Can't create system thread. Status: %08X'#13#10, rtnCode);
    end;
    Inc(i);
  end;

  g_dwCountThreads := dwCount;
  if dwCount <> 0 then
  begin
    Result := STATUS_SUCCESS; { 返回成功说明至少有一个线程在运行 }
  end else
  begin
    Result := STATUS_UNSUCCESSFUL; { 无法启动任何线程 }
  end;
end;

function _DriverEntry(pDriverObject:PDRIVER_OBJECT;
                      pusRegistryPath:PUNICODE_STRING): NTSTATUS; stdcall;
var
  status:NTSTATUS;
  pDeviceObject:PDEVICE_OBJECT;
  liTickCount:LARGE_INTEGER;
begin
  status := STATUS_DEVICE_CONFIGURATION_ERROR;
  RtlInitUnicodeString(@g_usDeviceName, '\\Device\\MutualExclusion');
  RtlInitUnicodeString(@g_usSymbolicLinkName, '\\DosDevices\\MutualExclusion');
  if IoCreateDevice(pDriverObject, 0, @g_usDeviceName,
                    FILE_DEVICE_UNKNOWN, 0, False,
                    @pDeviceObject) = STATUS_SUCCESS then
  begin
    if IoCreateSymbolicLink(@g_usSymbolicLinkName,
                            @g_usDeviceName) = STATUS_SUCCESS then
    begin
      { 因为 ThreadProc 存在于我们的驱动主体中，因此只要还有一个
      { 线程在运行，就不能卸载驱动，必须要等到所有的线程都退出
      { 方可卸载驱动。为了达到这个目的，我们需要一些内存。必须
      { 在这里分配这些内存，因为如果我们在 DriverUnload 中去分配，
      { 一旦分配失败，就没有办法停止驱动了。}

      { 每个线程对象都有一个内建的等待块(wait block)数组(缺省一
      { 个数组中有 3 个等待块)用于有几个对象并存时的等待操作。由于
      { 没有额外的等待块可供使用，因此通常情况下，这些内建的等待
      { 块数组被用于多个等待操作。当然，如果需要并存的对象的数目
      { 超过了内建等待块的数目, 可以通过设置 WaitBlockArray 参数指
      { 定一个替代的等待块用于等待操作。}

      { 在本例中，我们的 NUM_THREADS 大于 THREAD_WAIT_OBJECTS(3). }
      { 因此就必须要使用自己的 Wait Block 了. }
      g_pkWaitBlock := ExAllocatePool(NonPagedPool, NUM_THREADS * SizeOf(KWAIT_BLOCK));
      if g_pkWaitBlock <> nil then
      begin
        { 初始化 mutex }
        KeInitializeMutex(@g_kMutex, 0);
        { 出于性能方面的考虑, 可以使用 Ex..FastMutex 函数替代
        { Ke..Mutex. 当然, 快速 mutex 无法递归取得而内核 mutex
        { 却可以；另一个缺点是 ExAcquireFastMutex 设置 IRQL＝APC_LEVEL,
        { 并且调用 ExAcquireFastMutex 返回后，调用者函数也将运行在
        { APC_LEVEL. }
        KeQueryTickCount(@liTickCount);
        { 初始化随机数发生器种子 }
        srand(liTickCount.LowPart);
        g_dwWorkElement := 0;
        if StartThreads = STATUS_SUCCESS then
        begin
          pDriverObject^.DriverUnload := @DriverUnload;
          status := STATUS_SUCCESS;
        end else
        begin
          CleanUp(pDriverObject);
        end;
      end else
      begin
        CleanUp(pDriverObject);
        DbgPrint('MutualExclusion: Couldn't allocate memory for Wait Block'#13#10);
      end;
    end else
    begin
      IoDeleteDevice(pDeviceObject);
    end;
  end;
  result :=  status;
end;

end.
```

## 11.2 DriverEntry

代码:

```
g_pkWaitBlock := ExAllocatePool(NonPagedPool, NUM_THREADS * SizeOf(KWAIT_BLOCK));
```

分配 `NUM_THREADS * SizeOf(KWAIT_BLOCK)`大小的内存，常量 `NUM_THREADS` 决定我们一共启动多少个线程。分配的内存块指针保存在全局变量 `g_pkWaitBlock` 中，我们将在卸载驱动时使用这块内存，为什么需要分配这块内存以及为什么要在驱动初始化时分配这块内存的原因稍后再做解释，在这里我们并不使用这块内存。

代码:

```
KeInitializeMutex(@g_kMutex, 0);
```

这里我们将使用 `Mutex` 对象来同步，所以首先要调用 `KeInitializeMutex` 函数来初始化 `Mutex`，`KeInitializeMutex` 函数只是简单的填充一个 `KMUTANT` 变量，`KMUTANT` 结构描述一个 `Mutex` 对象，定义如下：

代码:

```
KMUTANT = packed record
    Header:TDispatcherHeader;
    MutantListEntry:TListEntry;
    OwnerThread:PKThread;
    Abandoned:Boolean;
    ApcDisable:Byte;
    Alignment0:Word;
end;
```



初始化完成后，`Mutex` 处于自由状态可供使用了。

代码:

```
KeQueryTickCount(@liTickCount);
```

为了最大程度地模拟真实环境，需要让各个线程乱序访问共享资源，为了达到这个目的，我们将生成一些随机数，并使用这些随机数来控制每个线程的延迟时间间隔。`Ntoskrnl.exe` 导出了标准库函数 `rand`，这个函数产生 `0--$7FFF` 之间的一个整数。产生随机数需要一个种子数（有些地方称之为主数），在内核中，这个种子数是一个全局变量，在系统引导后就被初始化并且始终存在于内存中了，所以我们可以不用调用 `srand` 去初始化随机数发生器的种子的，不过我们的程序还是依照常规调用了 `srand` 函数去初始化随机数发生器种子，首先我们调用 `KeQueryTickCount` 函数返回自系统启动以来发生过的计时器中断的次数，并以这个数为参数调用 `srand` 初始化随机数发生器种子。

代码:

```
g\_dwWorkElement := 0;
```

`g\_dwWorkElement` 就是本例中所有线程的共享资源。

### 11.3 创建线程

代码:

```
i := 0;
  { dwCount 保存实际运行的线程数 }
  dwCount := 0;
  while i < NUM\_THREADS do
  begin
    { 启动 NUM\_THREADS 个线程 }
    rtnCode := PsCreateSystemThread(@hThread, THREAD\_ALL\_ACCESS,
                                    nil, 0, nil,
                                    @ThreadProc, nil);
    if rtnCode = STATUS\_SUCCESS then
    begin
      { 我们不需要 PsCreateSystemThread 返回的线程句柄. }
      { 但是我们需要指向它的指针. 以便我们引用线程对象并且关闭它. }
      ObReferenceObjectByHandle(hThread, THREAD\_ALL\_ACCESS,
                                nil, KernelMode,
                                @g\_apkThreads\[dwCount\], nil);
      ZwClose(hThread);
      DbgPrint('MutualExclusion: System thread created. Thread Object: %08X'#13#10,
               g\_apkThreads\[dwCount\]);
      Inc(dwCount);
    end else
    begin
      DbgPrint('MutualExclusion: Can''t create system thread. Status: %08X'#13#10, rtnCode);
    end;
    Inc(i);
  end;
  g\_dwCountThreads := dwCount;
```

这里创建线程的方法与我们在 `TimerWorker` 里的方法没有什么大的不同，我们通过循环调用 `PsCreateSystemThread` 创建了 `NUM\_THREADS` 个线程，并将指向线程的指针保存在 `g\_apkThreads` 数组中，实际创建的线程数保存在 `g\_dwCountThreads` 变量中，线程函数依旧是 `ThreadProc`。这里唯一要注意的就是创建的线程数不能大于 `MAXIMUM\_WAIT\_OBJECTS`，`MAXIMUM\_WAIT\_OBJECTS` 是一个常量，其值等于 64，也就是说，同一时间并存的线程数不能超过 64 个。在 `DriverUnload` 里，我们将等待所有这些创建的线程正常退出后再卸载驱动。

### 11.4 线程函数 ThreadProc

我们创建了 `NUM\_THREADS` 个线程，这些线程的线程函数最终都会获得执行的权限，其 `ThreadProc` 函数将会执行。

代码:

```
pkThread := PsGetCurrentThread;
DbgPrint('MutualExclusion: Thread %08X is entering ThreadProc'#13#10, pkThread);
```

函数 `PsGetCurrentThread` 返回一个当前线程结构的指针，我们用 `DbgPrint` 输出该指针的地址。

代码:

```
dwCount := 0;
  while dwCount < NUM\_WORKS do
  begin
```

主循环重复执行 `NUM\_WORKS` 次，每次循环都会累加共享资源--全局变量 `g\_dwWorkElement`，每个线程累加 `NUM\_WORK` 次。

首先调用 `KeWaitForMutexObject` 竞争 `Mutex`，一旦取得 `Mutex`，线程开始运行，并可以自由的操作共享资源，其余 `NUM\_THREADS-1` 个线程将处于等待状态，直到这个线程释放 `Mutex`。

为了模拟线程对共享资源的访问，我们简单地随机停止线程执行 `050` 毫秒。

代码:

```
liDelayTime.HighPart := liDelayTime.HighPart or -1;
liDelayTime.LowPart := -(rand shl 4);
KeDelayExecutionThread(KernelMode, false, @liDelayTime);
```

函数 `rand` 产生一个 `0--$7FFF` 之间的随机数，再乘以 16 就是我们想要的延迟值。`KeDelayExecutionThread` 和 `KeSetTimerEx` 都传递一个 64 位的延迟值去设置延迟参数，不同之处在于 `KeSetTimerEx` 传递的是一个 `LARGE\_INTEGER`，而 `KeDelayExecutionThread` 传递的是一个指向 `LARGE\_INTEGER` 的指针。`KeDelayExecutionThread` 的作用是将当前线程置于可报警或者不可报警的等待状态给定的时间，此函数必须执行在 `IRQL = PASSIVE\_LEVEL`，内部是通过计时器实现的。

代码:

```
Inc(dwWorkElement);
g\_dwWorkElement := dwWorkElement;
```

改变共享资源的值并写回。

代码:

```
KeReleaseMutex(@g\_kMutex, False);
```

工作完成，我们必须释放 `Mutex` 以便其他正在等待的线程捕获它并完成自己的工作。`KeReleaseMutex` 的函数原型如下：

代码:

```
function KeReleaseMutex(Mutex:PKMutex; Wait:LongBool):LONG; stdcall;
```

其作用是释放一个由 `KeWaitXXX` 函数占用的 `Mutex` 对象，此函数可临时提升 `IRQL`。当参数 `Wait=false` 时，函数返回前会把 `IRQL` 恢复到调用它之前的原始值；如果 `Wait=true`，函数返回时不会降低 `IRQL`，在这种情况下，`KeReleaseMutex` 调用必须跟随在一个 `KeWaitXXX` 函数调用之后。如果设置 `Wait=TRUE`，调用者可以阻止在 `KeWaitXXX` 调用和 `KeReleaseMutex` 调用之间不必要的上下文切换。`KeWaitXXX` 函数在返回前，会把 `IRQL` 恢复到开始 `KeReleaseMutex` 调用之前的值，尽管 `IRQL` 不允许在两个函数调用之间进行上下文切换，但是这些调用在用于开始和结束一个原子操作时并不是很可靠。比如在这两个函数调用之间，同时在另一个处理器上运行的线程可能会改变事件对象或者等待的目的对象的状态。如果调用者正运行在 `IRQL=DISPATCH\_LEVEL` 或正位于一个任意的线程上下文中，传递给 `KeWaitXXX` 函数的 `Timeout` 参数必须为 0。

代码:

```
Result := PsTerminateSystemThread(STATUS\_SUCCESS);
```

结束本线程，让其他线程执行。如果所有线程都执行完毕则销毁线程对象。

### 11.5 DriverUnload

与 `TimerWorks` 驱动类似，我们必须要等待所有线程结束方可卸载驱动，因为所有线程都位于驱动主体中，如果我们在不合适的时候卸载驱动可能导致系统崩溃。解决方法就是等待所有的线程结束。

代码:

```
if g\_dwCountThreads > 0 then
  begin
    { 没有设置超时时间，一直等待到所有线程退出 }
    { 因为有多个线程对象，所以使用 KeWaitForMultipleObjects }
    KeWaitForMultipleObjects(g\_dwCountThreads, @g\_apkThreads,
                            WaitAll, Executive, KernelMode,
                            False, nil, g\_pkWaitBlock);
```

因为有多个线程，所以我在要调用 `KeWaitForMultipleObjects` 而不是 `KeWaitForSingleObject`。`KeWaitForMultipleObjects` 的第一个参数指定要等待对象的数量，也就是第二个参数传送的对象指针数组的个数，第三个参数决定是否所有的对象或者任意一个对象在需要满足等待条件前必须达到信号态，如果取值为 `WaitAny`，`KeWaitForMultipleObjects` 会返回 `STATUS\_WAIT\_0` 到 `STATUS\_WAIT\_63`，如果对象指针数组的第一个对象被置为信号态，则返回 `STATUS\_WAIT\_0`，第二个则返回 `STATUS\_WAIT\_1`，依此类推，返回值的低 6 位则是对象在对象指针数组中的下标，下标从 0 开始。最后一个参数是一个指向 `KWAIT\_BLOCK` 结构的指针，如果参数一小于等于 `THREAD\_WAIT\_OBJECTS`，此参数可以置为 `nil`，否则，此参数必须指向一个 `NUM\_THREADS * SizeOf(KWAIT\_BLOCK)`大小的 `KWAIT\_BLOCK` 结构数组。我们的程序里线程数大于 `THREAD\_WAIT\_OBJECTS`，因此我们就无法使用线程内建的等待块，必须要创建自己的等待块。

代码:

```
while g\_dwCountThreads > 0 do
    begin
      Dec(g\_dwCountThreads);
      ObfDereferenceObject(g\_apkThreads\[g\_dwCountThreads\]);
  end;
```

这段代码销毁我们创建的线程对象。

代码:

```
DbgPrint('MutualExclusion: WorkElement = %d'#13#10, g\_dwWorkElement);
```

`DbgPrint` 输出 `g\_dwWorkElement` 的值，这个值应该等于 `NUM\_THREADS * NUM\_WORKS`。
