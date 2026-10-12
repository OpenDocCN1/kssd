
08 `0007bc4c` `6b46938e` `00390000` `00000000` `0007bc9c` `dbgeng!UserTargetInfo::ReadVirtualUncached+0x4d` (FPO: [Non-Fpo])  

09 `0007bc68` `6b467e7f` `00390000` `00000000` `0007bc9c` `dbgeng!LocalUserTargetInfo::ReadVirtual+0x17` (FPO: [Non-Fpo])  

0a `0007bc8c` `6b48d15c` `6b4eea10` `003a0000` `00000000` `dbgeng!TargetInfo::ReadPointer+0x3a` (FPO: [Non-Fpo])  

0b `0007bcc0` `6b48e36f` `0007dd94` `00000000` `00000001` `dbgeng!BaseX86MachineInfo::DIdoModrm+0x7cd` (FPO: [Non-Fpo])  

0c `0007de08` `6b4cc804` `0007eea8` `0007de28` `00000001` `dbgeng!BaseX86MachineInfo::Disassemble+0x11d0` (FPO: [Non-Fpo])  

0d `0007eed8` `6b464264` `0000001f` `00000009` `00000020` `dbgeng!OutCurInfo+0x23b` (FPO: [Non-Fpo])  

0e `0007ef0c` `01005228` `002c4980` `00000001` `0000000f` `dbgeng!DebugClient::OutputCurrentState+0xac` (FPO: [Non-Fpo])  

0f `0007ff38` `01006a30` `00000000` `0007ffc0` `01006b66` `ntsd!MainLoop+0x11b` (FPO: [Non-Fpo])  

10 `0007ff44` `01006b66` `00000002` `002c4930` `002c2f70` `ntsd!main+0x10f` (FPO: [Non-Fpo])  

11 `0007ffc0` `77e830be` `38633437` `00000000` `7ffde000` `ntsd!mainCRTStartup+0x125` (FPO: [Non-Fpo])  

12 `0007fff0` `00000000` `01006a41` `00000000` `78746341` `kernel32!BaseProcessStart+0x23` (FPO: [Non-Fpo])  

这次看到了 `ReadProcessMemory` 进入内核后的执行过程。  

观察一下 `nt!MiDoPoolCopy` 函数的代码，在其起始处果然安装了一个 `SEH` 异常捕捉（《软件调试》24.5.6）：  

`nt!MiDoPoolCopy`:  

```
80b9706e 6848020000     push    248h
80b97073 68e056a080     push    offset nt!MmClaimParameterAdjustDownTime+0x80 (80a056e0)
80b97078 e8336df3ff     call    nt!_SEH_prolog (80acddb0)
```

其中 `80a056e0` 便是所谓的异常注册结构（参见《软件调试》24 章，24.2.3，24.5.3），使用 `dds` 命令观察其中的异常处理代码入口：  

```
kd> dds 80a056e0

80a056e0  ffffffff
80a056e4  80b9723c nt!MiDoPoolCopy+0x1ce
80a056e8  80b97255 nt!MiDoPoolCopy+0x1e7
```

其中，`80b9723c` 是异常的过滤表达式地址，`80b97255` 是异常的处理块地址。对这个异常的过滤表达式 `nt!MiDoPoolCopy+0x1ce` 设置断点：  

```
bp nt!MiDoPoolCopy+0x1ce
```

恢复执行后，断点迅速命中，果然有异常发生，观察栈回溯：  

```
kd> kvn

 # ChildEBP RetAddr  Args to Child                   

00 f8af94b0 80acbd41 f8af94d8 00000000 f8af94d8 nt!MiDoPoolCopy+0x1ce (FPO: [Non-Fpo])

01 f8af94d8 80adab26 f8af9984 f8af9ca0 f8af9680 nt!_except_handler3+0x61 (FPO: [Uses EBP] [3,0,7])

02 f8af94fc 80adaaf8 f8af9984 f8af9ca0 f8af9680 nt!ExecuteHandler2+0x26

03 f8af95ac 80a3939c f8af9984 f8af9680 00390000 nt!ExecuteHandler+0x24

04 f8af9968 80ad5199 f8af9984 00000000 f8af99d8 nt!KiDispatchException+0x166 (FPO: [Non-Fpo])

05 f8af99d0 80ad514a f8af9cb0 80b971b3 badb0d00 nt!CommonDispatchException+0x4d (FPO: [0,20,0])

06 f8af9a00 80ad90a4 00000000 00000023 00000023 nt!KiExceptionExit+0x1ee

07 f8af9cb0 80b97392 8278f020 00390000 8274a020 nt!KiUnlockDispatcherDatabase+0x1c

08 f8af9ce0 80b9756d 8278f0a0 00390000 8274a020 nt!MmCopyVirtualMemory+0x9c (FPO: [Non-Fpo])

09 f8af9d3c 80ad4568 00000438 00390000 0007bc9c nt!NtReadVirtualMemory+0x125 (FPO: [Non-Fpo])

0a f8af9d3c 7c8343a4 00000438 00390000 0007bc9c nt!KiFastCallEntry+0x158 (FPO: [0,3] TrapFrame @ f8af9d64)

0b 0007bbf8 00000000 00000000 00000000 00000000 ntdll!KiFastSystemCallRet (FPO: [0,0,0])
```

从栈帧 `01`～`05` 可以清楚的看到处理异常的过程。我们知道 `KiDispatchException` 的第一个参数是一个 `EXCEPTION_RECORD` 结构，因此可以使用 `dt` 命令观察这个异常的详情：  

```
kd> dt _EXCEPTION_RECORD f8af9984 

ntdll!_EXCEPTION_RECORD
  +0x000 ExceptionCode    : -2147483647
  +0x004 ExceptionFlags   : 0
  +0x008 ExceptionRecord  : (null)
  +0x00c ExceptionAddress : 0x80b971b3
  +0x010 NumberParameters : 2
  +0x014 ExceptionInformation : [15] 0
```

其中的 `-2147483647` 是异常代码，其实它就是 `0x80000001`，也就是 `EXCEPTION_GUARD_PAGE`。至此，我们证明了调试器去偷看保护页时其实也会触发 `EXCEPTION_GUARD_PAGE` 异常。  

```
kd> .formats 80000001

Evaluate expression:
  Hex:     80000001
  Decimal: -2147483647
  Octal:   20000000001
  Binary:  10000000 00000000 00000000 00000001
  Chars:   ....
  Time:    ***** Invalid
  Float:   low -1.4013e-045 high -1.#QNAN
  Double:  -1.#QNAN
```

执行 `.process` 观察当前的进程：  

```
kd> .process

Implicit process is now 8278f020

kd> !process 0 0 gdpage.exe

PROCESS 8278f020  SessionId: 0  Cid: 015c    Peb: 7ffdb000  ParentCid: 02b0
    DirBase: 1c25c000  ObjectTable: e16048d0  HandleCount:   8.
    Image: gdpage.exe
```

睁大眼睛，上面显示的是当前的进程是被调试 `GdPage`，而不是 `NTSD`。要知道，刚才要读内存的是 `NTSD`。这是怎么回事呢？  

这是因为 `MiDoPoolCopy` 在读 `GdPage` 进程的内存前执行了 `KeStackAttachProcess` 函数，所以它目前是在一种把灵魂化身到 `GdPage` 进程的状态。`MiDoPoolCopy` 异常处理代码会调用 `nt!KeUnstackDetachProcess` 来“显原身”。  

总结一下，这个海森伯效应的确容易让人困惑不解。会对调试有这样的影响，应该是 `NT` 的设计者本来也没有想到的吧。对于大多数内存，调试器“偷看”多少次都是无大碍的，但是看过了和没有看还是可能有差异，比如看的过程可能导致本来在磁盘上的内存交换到物理内存中。但是这个交换对应用程序来说通常是没有影响的。而对于本例中的保护页就不一样了，看一眼性质就变了，因此“导致了问题”。不过我们知道了其中的原委后，也就不困惑了。
