
# 海森伯效应一例

> 原文：[`www.pediy.com/kssd/pediy10/76698.html`](https://www.pediy.com/kssd/pediy10/76698.html)

我们把调试过程对被调试程序所产生的影响称为海森伯效应（详见《软件调试》28.6.1）。设计调试工具时，当然应该努力降低可能的海森伯效应，以便可以在调试器中调试时看到的症状与没有被调试时是一样的。

但正像这个名字的由来（海森伯的测不准原理）那样，海森伯效应一定是存在的。只不过是很小，大多数时候都不会对我们的调试造成明显的影响。

前两天，`jlflyfox` 提出了一个问题，这个问题其实是一种非常典型的海森伯效应。

为了讨论简单，可以使用下面的代码来重现 `jlflyfox` 所提出的问题。

```
/*-------------------------------------------------------------- 

GdPage.cpp : a real example to illustrate Heisenberg Effect 

   related with guard page by Raymond Zhang, Oct. 2008 

--------------------------------------------------------------*/ 

#include 

#include 

#include 

int main() 

{ 

  LPVOID lpvAddr;     

  lpvAddr = VirtualAlloc(NULL, 0x4096, 

                         MEM_RESERVE | MEM_COMMIT, 

                         PAGE_READONLY | PAGE_GUARD); 

  if(lpvAddr == NULL) 

  { 

    printf("VirtualAlloc failed with %ld\n", GetLastError()); 

    return -1; 

  } 

  return *(long *)lpvAddr;   

} 
```

简单来说，如果把上面的代码编译后直接运行，那么会发生异常而被强行终止，无论是调试版本，还是发布版本都会这样。如果有兴趣，那么你可以下载下面的文件包自己尝试一下：[`advdbg.org/books/download/gdpage.zip`](http://advdbg.org/books/download/gdpage.zip) 

但如果在调试器中运行这个程序，并且单步跟踪执行 `return *(long *)lpvAddr;` 这一行所对应的汇编指令，那么程序会很正常的退出，不会有异常发生，笔者尝试了 `VC6` 调试器、`WinDBG` 调试器，以及 `VS2005` 的集成调试器，现象都如此。

看来对于这个程序，在被调试时和不在被调试时，运行的结果很不一样，这是什么原因呢？

观察直接运行崩溃时的异常代码，其值为 `0x80000001`，搜索 `Windows` 的错误码定义或者《软件调试》的表 11-2，对应的常量是 `EXCEPTION_GUARD_PAGE`。它的含义是访问了具有 `PAGE_GUARD` 属性的保护页面。这样的保护页面的一个典型用途就是放在栈的顶部（低地址端），用来实现栈的自动增长（参见《软件调试》22.2.3 和 22.8）。

考虑到这个程序的开始处使用 `VirtualAlloc` 分配了 `4096` 个字节（一个页面）的内存区，并在分配时指定了 `PAGE_GUARD` 属性，而在 `main` 函数的最后一行却是故意去访问这个保护页，所以得到 `EXCEPTION_GUARD_PAGE` 异常是“合情合理”的。

那么现在的问题是在单步跟踪调试时，为什么就不发生异常了呢？

答案是调试器先“偷看了”这个页面。保护页的一个特征是访问一次后，系统会自动去除保护属性，这样第二次访问后就不会再有 `EXCEPTION_GUARD_PAGE` 异常。因为我们在单步跟踪时，调试器先于被调试程序偷看了这个内存，触发系统去除了 `PAGE_GUARD` 属性，所以当被调试程序再去访问时，就安然无事了。

说到这里，似乎问题就清楚了。但是可能还有些疑问。首先是调试器为什么要去“偷看”？回答是为了实现调试功能。事实上，调试时，特别是每次中断到调试器之前以及恢复执行（有新断点）时，调试器是需要“看”被调试进程的内存的。

以 `WinDBG` 为例，但单步跟踪 `return` 语句所对应的汇编指令时，`WinDBG` 会帮助我们显示出操作数的内容，如果是指针，它会显示出这个指针指向的数据：

```
0040106c 8b00             mov     eax,dword ptr [eax]  ds:0023:003a0000=00000000 
```

上面 `eax` 就是 `lpvAddr`，`WinDBG` 显示出了它的值是 `00000000`，要显示这个值，`WinDBG` 显然要去看一下的。

`VC6` 和 `VS2005` 的反汇编虽然没有做类似上面的显示，但是它实际上也会去观察这个地址。如果变量观察窗口中在观察有关的变量，那么就更是顺理成章了。

可以通过一个实验来观察一下以加深理解，在 `VC2005` 中调试上面的 `GdPage` 小程序，然后用 `WinDBG` 附加到 `devenv.exe`（`VS2005` 的进程）上，并设置一个下面这样的条件断点：

```
kernel32!ReadProcessMemory+0x5 ".if poi(@ebp+c)=0x3a0000 {} .else {kv 1;g}" 
```

根据《软件调试》的表 9-4，`ReadProcessMemory` API 是调试器访问被调试程序内存的方法。因此我们对其埋下断点。后面的条件是为了避免反复命中。其中的 `0x3a0000` 是 `lpvAddr` 的值，可能变化。

当我们单步执行到 `return` 语句所对应的第一行汇编时，

```
  return *(long *)lpvAddr;   

00401069   mov         eax,dword ptr [lpvAddr] 

0040106C   mov         eax,dword ptr [eax] 

} 
```

上面的断点果然命中，观察栈回溯：

```
0:019> kvn 100 

 # ChildEBP RetAddr  Args to Child                   

00 0c41c73c 5be240c8 00000e50 003a0000 0f592afc kernel32!ReadProcessMemory+0x5 (FPO: [Non-Fpo]) 

01 0c41c758 5be1785d 003a0000 0f592afc 00001000 NatDbgDE!Win32Debugger::RawReadProcessMemoryEx+0x19 (FPO: [Non-Fpo]) 

02 0c41c77c 5be1782a 0f453b98 003a0000 0f592afc NatDbgDE!RawReadProcessMemoryHelper+0x36 (FPO: [Non-Fpo]) 

03 0c41c798 5be178af 0f453b98 003a0000 0f592afc NatDbgDE!DbgReadMemory+0x1a (FPO: [Non-Fpo]) 

04 0c41c7e0 5be19b86 0f453b98 00000000 0f49bff8 NatDbgDE!AddrReadMemory+0x38 (FPO: [Non-Fpo]) 

05 0c41c838 5be19b2d 0f453b98 00000000 0f49bfd8 NatDbgDE!ProcessReadMemoryCmd+0x69 (FPO: [Non-Fpo]) 

06 0c41c85c 5be12fa2 00000048 00000019 0c41c890 NatDbgDE!DMLib::DMFunc+0xfb (FPO: [Non-Fpo]) 

07 0c41c86c 5be124e9 00000000 00000006 0645a368 NatDbgDE!TLClientLib::Local_TLFunc+0x8c 

08 0c41c890 5be12510 00000006 0645a368 00000000 NatDbgDE!CallTL+0x33 

09 0c41c8b0 5be126c2 00000006 0645a368 00000000 NatDbgDE!EMCallBackTL+0x18 

0a 0c41c8d8 5be197ca 00000019 0645a368 00000000 NatDbgDE!SendRequestX+0x7d 

0b 0c41c940 5be1989f 0645a368 00000000 00001000 NatDbgDE!LimitedReadPhysical+0x4c 

0c 0c41e7b0 5be199c5 0645a368 00000000 00001000 NatDbgDE!ReadPhysical+0xa3 

0d 0c41e800 5be19a74 0645a368 00000000 00000004 NatDbgDE!ReadForCache+0xa9 

0e 0c41e880 5be23db9 0645a368 00000000 5bedbcc0 NatDbgDE!ReadBuffer+0xf3 

0f 0c41f584 5be12496 0000002d 0645a368 00000000 NatDbgDE!EMFunc+0x5da 

10 0c41f5ac 5be23def 0000002d 0645a368 00000000 NatDbgDE!CallEM+0x20 

11 0c41f5e4 5be23e15 0645a368 00000000 0c41f644 NatDbgDE!OSDReadMemory+0x31 

12 0c41f600 5be48458 0f451310 5bedbcc0 0c41f644 NatDbgDE!CNativeProcess::ReadMemory+0x22 

13 0c41f664 5be485e2 0f48da90 0c41f684 0c41f7dc NatDbgDE!TRegisterTraits<_EA>::FormatValue+0xe8 

14 0c41f7b4 5be4860e 0c41f7dc 0c41f7d8 1d959456 NatDbgDE!TRegisterTraits<_EA>::GetRegisterInfo+0x66 

15 0c41f7fc 5be51ef3 ffffffff 0c9dbf00 1d959bb2 NatDbgDE!CRegisterProperty::GetPropertyInfo+0xb2 

16 0c41f82c 5be1fce4 00000000 0c9dbf00 0c41fcb8 NatDbgDE!CEnumRegisterGroup::GetChild+0x46 

17 0c41f84c 77e7a1ac 0f501568 00000001 0c9dbf00 NatDbgDE!CTreeEnumerator::Next+0x50 

18 0c41f870 77ef421a 5be16d16 0c41f884 00000004 RPCRT4!Invoke+0x30 

19 0c41fc7c 77ef4bf3 0c9a8dc0 0c8d5b84 01ddbb54 RPCRT4!NdrStubCall2+0x297 (FPO: [Non-Fpo]) 

1a 0c41fcd4 77600c31 0c9a8dc0 01ddbb54 0c8d5b84 RPCRT4!CStdStubBuffer_Invoke+0xc6 (FPO: [Non-Fpo]) 

1b 0c41fd14 77600bdb 01ddbb54 0c8c44cc 0c8b5760 ole32!SyncStubInvoke+0x33 (FPO: [Non-Fpo]) 

1c 0c41fd5c 7750f237 01ddbb54 0c9848f0 0c9a8dc0 ole32!StubInvoke+0xa7 (FPO: [Non-Fpo]) 

1d 0c41fe34 7750f15c 0c8d5b84 00000000 0c9a8dc0 ole32!CCtxComChnl::ContextInvoke+0xe3 (FPO: [Non-Fpo]) 

1e 0c41fe50 77600b11 01ddbb54 00000001 0c9a8dc0 ole32!MTAInvoke+0x1a (FPO: [Non-Fpo]) 

1f 0c41fe80 776009bc 01ddbb00 0c8d5b84 0c9a8dc0 ole32!AppInvoke+0x9c (FPO: [Non-Fpo]) 

20 0c41ff54 77600df2 01ddbb00 068f6238 00000000 ole32!ComInvokeWithLockAndIPID+0x2e0 (FPO: [Non-Fpo]) 

21 0c41ff80 7750fcb3 01ddbb00 7c802540 0c9a5fa8 ole32!ComInvoke+0x60 (FPO: [Non-Fpo]) 

22 0c41ff94 774fe3dc 01ddbb00 00001000 0c9a5fa8 ole32!ThreadDispatch+0x23 (FPO: [Non-Fpo]) 

23 0c41ffa8 774fe444 00000088 0c41ffec 7c80b6a3 ole32!CRpcThread::WorkerLoop+0x1e (FPO: [0,0,0]) 

24 0c41ffb4 7c80b6a3 0c9a5fa8 00001000 00000088 ole32!CRpcThreadCache::RpcWorkerThreadEntry+0x1b (FPO: [Non-Fpo]) 

25 0c41ffec 00000000 774fe429 0c9a5fa8 00000000 kernel32!BaseThreadStart+0x37 (FPO: [Non-Fpo]) 
```

从 `ReadProcessMemory` 函数的第二个参数可以看到，确实是在访问 `0x3a0000` 这个地址，铁证。

接下来，另一个问题时，调试器看时会不会也触发异常呢？这个异常又发到哪里去了呢？正确的答案是，也会触发异常，但是被内核中的函数捕捉到并处理了。因此，调试器其实也没有收到 `EXCEPTION_GUARD_PAGE` 异常。

那么如何才能观察到调试器“偷看时”触发异常和异常被处理的过程呢？这就需要使用内核调试器了。下面是实验过程。

将上面的小程序 `GdPage.exe` 复制到另一个系统中，笔者使用的是虚拟机中的 `XP SP3`，然后与其建立内核调试会话。然后在目标系统中使用系统自带的 `ntsd` 调试 `gdpage.exe`。这里体现出了系统自带一个调试器的价值，安装其它调试器毕竟需要时间。

```
ntsd gdpage.exe 
```

接下来需要在内核调试器中埋一个断点，长话短说便是下面这样：

```
bp /p 8274a020 nt!MiDoPoolCopy    

8274a020 是 ntsd 进程的 EPROCESS 结构地址，有了这个指定后，只有这个进程执行 MiDoPoolCopy 时才会中断下来。 

```

而后，在 `ntsd` 中对 `return` 语句前面设置一个断点，执行到这里后，使用 `p` 命令单步执行。内核调试器中的断点果然命中，显示栈回溯：

```
kd> kvn 

 # ChildEBP RetAddr  Args to Child                   

00 f8af9cb0 80b97392 8278f020 003a0000 8274a020 nt!MiDoPoolCopy (FPO: [Non-Fpo]) 

01 f8af9ce0 80b9756d 8278f0a0 003a0000 8274a020 nt!MmCopyVirtualMemory+0x9c (FPO: [Non-Fpo]) 

02 f8af9d3c 80ad4568 00000438 003a0000 0007bc9c nt!NtReadVirtualMemory+0x125 (FPO: [Non-Fpo]) 

03 f8af9d3c 7c8343a4 00000438 003a0000 0007bc9c nt!KiFastCallEntry+0x158 (FPO: [0,3] TrapFrame @ f8af9d64) 

04 0007bbd8 7c80e9a4 77e7e361 00000438 00390000 ntdll!KiFastSystemCallRet (FPO: [0,0,0]) 

05 0007bbdc 77e7e361 00000438 003a0000 0007bc9c ntdll!NtReadVirtualMemory+0xc (FPO: [5,0,0]) 

06 0007bbf8 6b4d7348 00000438 003a0000 0007bc9c kernel32!ReadProcessMemory+0x1b (FPO: [Non-Fpo]) 

07 0007bc14 6b4690d4 002c4aa0 00000438 00000000 dbgeng!LiveUserDebugServices::ReadVirtual+0x19 (FPO: [Non-Fpo]) 
```



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
