
# 为 OllyDbg 增加 LastBranchRecord 功能

> 原文：[`www.pediy.com/kssd/pediy10/80472.html`](https://www.pediy.com/kssd/pediy10/80472.html)

为 OllyDbg 增加 LastBranchRecord 功能

关于 CPU 的分枝监视功能,已经有牛人写过文章,<软件调试>书中也有详细介绍,就不多说了。直接给代码。

另外,我的代码是写在自己的插件中的,一堆乱七八糟的东西,就不给 bin 了,代码还算简单。

测试机器为 WinXP Sp2,CPU 为 P4 Family0F Model6。

## 在驱动中创建与 Ring3 的共享内存

这里的代码直接从 sudami 的文章中 copy 的哈。

// 创建共享内存向 Ring3 传递数据
```
g_SharedMem = ExAllocatePool(NonPagedPool, PAGE_SIZE);
if (!g_SharedMem)
    goto __Fail;
g_MdlShared = IoAllocateMdl(g_SharedMem,  
          PAGE_SIZE,  
          FALSE,  
          FALSE,  
          NULL);
if (!g_MdlShared)
{
   ExFreePool(g_SharedMem);
   goto __Fail;
}
MmBuildMdlForNonPagedPool(g_MdlShared);
```

## 将内存映射到 OllyDbg 用户空间

```
case IRP_MJ_DEVICE_CONTROL:
     switch (IrpSp->Parameters.DeviceIoControl.IoControlCode)  
     {
  case IOCTL_INSTALL_HOOK:
      ...
         if (OutputLen >= sizeof(PVOID))
      {
    MapAddress = MmMapLockedPagesSpecifyCache  (
          g_MdlShared,
        UserMode,
        MmNonCached,
        NULL,
        FALSE,
        NormalPagePriority );
    if (MapAddress)
    {
        //DbgPrint("Map ShareMem ok, MapAddress = %08X", MapAddress);
        *(PVOID *)OutputBuffer = MapAddress;
        InfoSize = sizeof(PVOID);
    }
      }
      ...
```

## 挂 INT1

```
volatile __declspec(naked) void NewInt01()
{
  //  - Interrupt 1 Handler -
  //
  //  offset   | contains
  //  ---------+---------------------------------
  //  esp     : EIP Context
  //  esp + 4  : CS  Context
  //  esp + 8  : EFLAGS Context
  __asm
  {
      pushad
      push    fs
      push    ds
      push    es
      mov     eax, 30h
      mov     fs, ax
      mov     eax, 23h
      mov     ds, ax
      mov     es, ax
      call  dword ptr [IoGetCurrentProcess]
      cmp  eax, g_ObjProc  // Debuggee?
      jne  __oldint01
      mov  eax, dr6
      bt  eax, 0Eh  //单步?
      jnc  __oldint01
      mov    eax, g_FastTrace //配置选项
      cmp    eax, 1
      jne    __oldint01
      mov    edi, g_SharedMem
      mov    ecx, MSR_LASTBRANCH_TOS
      rdmsr
      mov    ebx, eax
      and    ebx, 0Fh    //TOS = 4 位
      mov    ecx, MSR_LASTBRANCH_0_FROM_LIP
      add    ecx, ebx
      rdmsr
      mov    [edi], eax
      mov    ecx, MSR_LASTBRANCH_0_TO_LIP
      add    ecx, ebx
      rdmsr
      mov    [edi+4], eax
      mov    ecx, MSR_DEBUGCTLA
      rdmsr
      or     eax, 3  // BTF & LBR
      wrmsr
__oldint01:  
      pop    es     
      pop    ds
      pop    fs
      popad
      jmp    g_KiTrap01;
  }
}
```

这里没有用 DebugStore,直接从 LBR 栈取数据,省事,而且支持多处理器。

## Ring3 部分的代码

Hook 了 OllyDbg 的 `WaitForDebugEvent`。

```
case EXCEPTION_SINGLE_STEP: //单步异常
     // 判断一下是否处于 TraceInto 或 TraceOver,这个没写,再说
     // 有时第 1 条数据记录的是内核地址,这里可以判断一下
    if (g_Ring0Options.FastTrace)
    {
  if (g_dwLBRCount < MAX_LBR_NUMBER)
  {
     g_LBRRecord[g_dwLBRCount].dwIndex = g_dwLBRCount;
     g_LBRRecord[g_dwLBRCount].dwThreadId = lpDbgEvent->dwThreadId;
     g_LBRRecord[g_dwLBRCount].dwFrom = ((PDWORD)g_SharedMem)[0];
     g_LBRRecord[g_dwLBRCount].dwTo   = ((PDWORD)g_SharedMem)[1];
     // 在这里读取代码,以防备 SMC 破坏代码
     if (_Readcommand(((PDWORD)g_SharedMem)[0], g_LBRRecord[g_dwLBRCount].CmdFrom) &&
        _Readcommand(((PDWORD)g_SharedMem)[1], g_LBRRecord[g_dwLBRCount].CmdTo))
     {
    _Addsorteddata(&g_LBRTable.data, &g_LBRRecord[g_dwLBRCount++]);
     }  
     else
     {
    _Addtolist(0, 1, "LBR Buffer Overrun");
     }
        }
    }
    ...
```

剩下的就是 OD 插件界面相关的,从 Conditional Branch Logger 抄的。

需要说明的是,OD 自己的 Trace 功能,若选择 Always trace over system DLLs,估计是用 int3 断点来获取调用后的控制。

用 BTF 位,则任何控制转移都会被记录,不过可以在 `WaitForDebugEvent` 中筛选一下记录的数据。另外,也可以在驱动中做

得更好点,只在运行被调试进程时才打开 LBR。不过我的测试似乎性能下降不大,至少不比 OD 自己的跟踪差。

上个图。加载一个用 UPX 加壳的程序,用 OllyBone 对第 1 区段下执行断点。OllyBone 我也抄了,不过功能有点

问题,这里先不管它了,对这个程序是可以的。

`Ctrl-F11` 开始跟。到 OEP 停下显示数据。共 90000 多条数据。可以清楚看到是怎样跳到 OEP 的,对于有花指令的代码有点用。

当然,最好不要加载完就这样干,那样数据太多了。

本来想把 OD 的语法配色也搞出来,感觉比较复杂,就算啦。图标也用的 CBL 插件的。

![](img/74530639cc516a3644476de22fc38df9.jpg)
