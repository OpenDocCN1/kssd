
# 调试笔记之 thread

> 原文：[`www.pediy.com/kssd/pediy11/111038.html`](https://www.pediy.com/kssd/pediy11/111038.html)

## 说明

关于内核相关的，借助外部资料去阅读理解，效果可能会更好点；如 `wrk`, `reactOS`, `windows internals`。

必要的时候，也需要用调试器去跟踪内核，发现问题之所在，并体会其中的乐趣。

关于线程切换是一个很基础的话题，发这个帖子主要是为了分享一些实际调试经验。很多人都是直接将调试结论写出来，但是一般不会写怎么调试的，可能觉得调试的过程比较低级乏味吧。当然，结论的来源，无非就是两点：第一，阅读类系统内核代码，如上；第二，就是调试。

关于进程和线程切换的原理性说明，就不罗嗦，直接切入主题。

## 调试目的

1. 调试线程控制块 `TCB`，获知哪些字段指示了线程每获得一次 `CPU` 时间片时，实际执行的时间；因为当线程获得 `CPU` 的时候，并不一定都是在执行代码。
2. 了解线程切换时的工作流程；

## 调试环境

`host+vmware`; 或 `host+1394(target)`

## 调试方法

写一个 `application console` 程序，名为 `dbgTCB`，为了便于调试，程序内部只执行一个简单的死循环，如下：

```c
int main(int argc, char ** argv)
{
    int num = 0;
    _asm int 3;
    while(1)
    {
        num ++;
    }
    return 0;
}
```

先编译这段程序（`debug` 模式），然后将编译后的 `EXE`, `.PDB` 文件拷贝到目标机（`VM`），用 `WINDBG` 联机调试。在目标机里执行 `exe`，当执行到 `_asm int 3;` 时被中断，控制权返还到 `windbg`。调试开始：

### 步骤

1. `!analyze -v` 分析
   如果在调试机上编译，在 `VM`（被调试机）上执行，`windbg` 加载工程的符号表 `dbgTCB.pdb`，那么该命令执行过后，就会出现源代码窗口，反汇编窗口，并且高亮显示断点位置。

2. 查看当前堆栈：`kb`

   ```
   kd> kb

   ChildEBP RetAddr  Args to Child

   0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x1f [e:\temp program\dbgtcb\dbgtcb.cpp @ 5]
   0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]
   0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23
   ```

3. 定位进程地址

   `!process 0 0` 注：用 `!process 0 7` 可以得到更加详细的信息，但是耗时，在我们已经知道进程名字的情况下，选用前者，然后用命令：

   `!process addr 7` 得到更加详细的信息，其中 `addr` 是要关注的进程的内存加载地址

   ```
   PROCESS 81f86520  SessionId: 0  Cid: 05a8    Peb: 7ffde000  ParentCid: 05e8
   DirBase: 04f002e0  ObjectTable: e143e718  HandleCount:   7.
   Image: dbgTCB.exe
   ```

   //得到进程 `dbgTCB.exe` 的详细信息

   ```
   kd> !process 81f86520   7

   PROCESS 81f86520  SessionId: 0  Cid: 05a8    Peb: 7ffde000  ParentCid: 05e8
   DirBase: 04f002e0  ObjectTable: e143e718  HandleCount:   7.
   Image: dbgTCB.exe
   VadRoot 820674b8 Vads 22 Clone 0 Private 43. Modified 0. Locked 0.
   DeviceMap e1dfb5c8
   Token                         e14988c0
   ElapsedTime                   00:00:00.046
   UserTime                      00:00:00.015
   KernelTime                    00:00:00.000
   QuotaPoolUsage[PagedPool]      12364
   QuotaPoolUsage[NonPagedPool]    880
   Working Set Sizes (now,min,max)  (174, 50, 345) (696KB, 200KB, 1380KB)
   PeakWorkingSetSize            174
   VirtualSize                   7 Mb
   PeakVirtualSize               7 Mb
   PageFaultCount                167
   MemoryPriority                BACKGROUND
   BasePriority                  8
   CommitCharge                  51
   ```

   //得到 `TCB` 地址

   ```
        THREAD 81f862a8  Cid 05a8.0588  Teb: 7ffdd000 Win32Thread: 00000000 RUNNING on processor 0
        Not impersonating
        DeviceMap             e1dfb5c8
        Owning Process        0       Image:         <Unknown>
        Attached Process      81f86520      Image:         dbgTCB.exe
        Wait Start TickCount  9170            Ticks: 1 (0:00:00:00.015)
        Context Switch Count  23

        UserTime              00:00:00.000
        KernelTime            00:00:00.000
        Win32 Start Address dbgTCB!mainCRTStartup (0x004010b0)
        Start Address kernel32!BaseProcessStartThunk (0x7c8106f5)
        Stack Init f0a95000 Current f0a94b8c Base f0a95000 Limit f0a92000 Call 0
        Priority 10 BasePriority 8 PriorityDecrement 2 DecrementCount 16
        ChildEBP RetAddr  Args to Child

        0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x1f [e:\temp program\dbgtcb\dbgtcb.cpp @ 5]
        0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]
        0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23 (FPO: [Non-Fpo])
   ```

4. 查看线程 `ETHREAD` 信息

   ```
   kd> dt nt!_ethread 81f862a8

     +0x000 Tcb                : _KTHREAD
     +0x1c0 CreateTime         : _LARGE_INTEGER 0xe543bc6`8be3c610
     +0x1c0 NestedFaultCount   : 0y00
     +0x1c0 ApcNeeded          : 0y0
     +0x1c8 ExitTime           : _LARGE_INTEGER 0x81f86470`81f86470
     +0x1c8 LpcReplyChain      : _LIST_ENTRY [ 0x81f86470 - 0x81f86470 ]
     +0x1c8 KeyedWaitChain     : _LIST_ENTRY [ 0x81f86470 - 0x81f86470 ]
     +0x1d0 ExitStatus         : 0
     +0x1d0 OfsChain           : (null)
     +0x1d4 PostBlockList      : _LIST_ENTRY [ 0x81f8647c - 0x81f8647c ]
     +0x1dc TerminationPort    : 0xe11c8298 _TERMINATION_PORT
     +0x1dc ReaperLink         : 0xe11c8298 _ETHREAD
     +0x1dc KeyedWaitValue     : 0xe11c8298

     ...  ...

     ETHREAD 偏移 0 字节，查看_kthread 数据结构

   kd> dt nt!_kthread 81f862a8+0

     +0x000 Header             : _DISPATCHER_HEADER
     +0x010 MutantListHead     : _LIST_ENTRY [ 0x81f862b8 - 0x81f862b8 ]
     +0x018 InitialStack       : 0xf0a95000
     +0x01c StackLimit         : 0xf0a92000
     +0x020 Teb                : 0x7ffdd000
     +0x024 TlsArray           : (null)
     +0x028 KernelStack        : 0xf0a94b8c
     +0x02c DebugActive        : 0 ''
     +0x02d State              : 0x2 ''
     +0x02e Alerted            : [2]  ""
     +0x030 Iopl               : 0 ''
     +0x031 NpxState           : 0xa ''
     +0x032 Saturation         : 0 ''
     +0x033 Priority           : 10 ''
     +0x034 ApcState           : _KAPC_STATE
     +0x04c ContextSwitches    : 0x17
     +0x050 IdleSwapBlock      : 0 ''
     +0x051 Spare0             : [3]  ""
     +0x054 WaitStatus         : 0
     +0x058 WaitIrql           : 0x1 ''
     +0x059 WaitMode           : 1 ''
     +0x05a WaitNext           : 0 ''
     +0x05b WaitReason         : 0x11 ''
     +0x05c WaitBlockList      : 0x81f86318 _KWAIT_BLOCK
     +0x060 WaitListEntry      : _LIST_ENTRY [ 0x80554870 - 0x80554870 ]
     +0x060 SwapListEntry      : _SINGLE_LIST_ENTRY
     +0x068 WaitTime           : 0x23d2
     +0x06c BasePriority       : 8 ''
     +0x06d DecrementCount     : 0x10 ''
     +0x06e PriorityDecrement  : 2 ''
     +0x06f Quantum            : 4 ''
     +0x070 WaitBlock          : [4] _KWAIT_BLOCK
     +0x0d0 LegoData           : (null)
     +0x0d4 KernelApcDisable   : 0
     +0x0d8 UserAffinity       : 1
     +0x0dc SystemAffinityActive : 0 ''
     +0x0dd PowerState         : 0 ''
     +0x0de NpxIrql            : 0 ''
     +0x0df InitialNode        : 0 ''

     ...  ...
   ```

5. 线程工作状态的决定因子 `nt!_ethread._kthread.State`
   如上结构体，`nt!_kthread` `81f862a8+0+2d` 处，有一个 `State` 的字段，取值为 `0x2`。

   此时，通过 `!process 81b68020 7` 命令，得知，线程工作状态为：`running`

   打开内存窗口，定位到地址 `81f862a8+0+2d`，将内存数据改为 `01`

   此时用 `!process 81b68020 7` 观察：

   ```
    ...  ...

   THREAD 81e03318  Cid 0204.04e4  Teb: 7ffdf000 Win32Thread: 00000000 READY

    ...  ...
   ```

   线程状态由 `running`-->`ready`。

   那么，现在可以确定，`nt!_ethread._kthread.State` 数据值就是指示当前线程工作状态的因子

6. 跟踪线程状态切换流程
   由前面的分析，得出 `nt!_ethread._kthread.State` 就是线程工作状态的地址，当线程切换的时候，一定会改变线程状态，那么在状态内存地址处下断点，就可以跟踪到修改这一断点的函数了，进而可能跟踪到其他线程切换时的函数了。

   一个线程获取 `CPU`，但并非一定处于执行状态，我们通知跟踪这个状态值，可以精确地得到线程在 `CPU` 上的执行时间。

   注：准备工作，单步执行，让被调试程序进入 `while(1)` 循环：

   查看堆栈：

   ```
   kd> kb

   ChildEBP RetAddr  Args to Child

   0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x29 [e:\temp program\dbgtcb\dbgtcb.cpp @ 9]
   0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]
   0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23
   ```

   然后让程序运行

   ```
   g
   ```

   在 `ctrl+break` 中断

   再次查看堆栈：

   ```
   kd> kb

   ChildEBP RetAddr  Args to Child

   f0a94d50 8054209d 00000001 00000000 000000d1 nt!RtlpBreakWithStatusInstruction
   f0a94d50 0040103c 00000001 00000000 000000d1 nt!KeUpdateSystemTime+0x165
   0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x2c [e:\temp program\dbgtcb\dbgtcb.cpp @ 9]
   ```



0012ffc0 7c817067 00390038 00370036 7ffde000 `dbgTCB!mainCRTStartup+0xe9` `[crt0.c @ 206]`

0012fff0 00000000 004010b0 00000000 78746341 `kernel32!BaseProcessStart+0x23`

在 `nt!_ethread._kthread.State` 地址处下断点：

`ba w4 81f862d4` ,其实地址为：`81f862d5`，但是 Data breakpoint must be aligned，所以取地址为：`81f862d4`

OK，再次运行程序:`g`

下面关系到线程切换的若干函数调用过程：理论上，程序会断在修改 `nt!_ethread._kthread.State` 状态值的地方，

那么，也就可以间接地知道是哪个函数在修改这个值了。

```
kd> g

Breakpoint 0 hit

nt!KiReadyThread+0x3a:

80501942 833d84bd548000  cmp     dword ptr [nt!KiIdleSummary (8054bd84)],0
```

!!!程序断在了这里，可以推测：`nt!KiReadyThread` 这个函数就是改变线程工作状态的函数

`nt!KiReadyThread`:

```
80501908 8bff            mov     edi,edi
8050190a 55              push    ebp
8050190b 8bec            mov     ebp,esp
8050190d 51              push    ecx
8050190e 51              push    ecx
8050190f 8bc1            mov     eax,ecx
80501911 8d8828010000    lea     ecx,[eax+128h]
80501917 53              push    ebx
80501918 8a19            mov     bl,byte ptr [ecx]
8050191a c60100          mov     byte ptr [ecx],0
8050191d 8b15a0bd5480    mov     edx,dword ptr [nt!KeTickCount (8054bda0)]
80501923 0fbe4833        movsx   ecx,byte ptr [eax+33h]
80501927 56              push    esi
80501928 895068          mov     dword ptr [eax+68h],edx
8050192b 8b5044          mov     edx,dword ptr [eax+44h]
8050192e 57              push    edi
8050192f eb5e            jmp     nt!KiReadyThread+0x87 (8050198f)
80501931 80b82a01000000  cmp     byte ptr [eax+12Ah],0
80501938 0f84ac000000    je      nt!KiReadyThread+0xe2 (805019ea)
      //2dh，状态值为 3 是 standby 状态 ，至此线程的四个状态应该清楚：00 initialized,01 ready,02 running,03 standby   
8050193e c6402d03        mov     byte ptr [eax+2Dh],3     
       //被中断的指令处，在此之后 p 单步运行 
80501942 833d84bd548000  cmp     dword ptr [nt!KiIdleSummary (8054bd84)],0 ds:0023:8054bd84=00000000
80501949 8b3d403e5580    mov     edi,dword ptr [nt!KiProcessorBlock (80553e40)]
8050194f 0f85cb000000    jne     nt!KiReadyThread+0x118 (80501a20)
80501955 8b5708          mov     edx,dword ptr [edi+8]
80501958 85d2            test    edx,edx
8050195a 0f84c9000000    je      nt!KiReadyThread+0x121 (80501a29)
80501960 0fbe7233        movsx   esi,byte ptr [edx+33h]
80501964 3bce            cmp     ecx,esi
80501966 0f8ed4000000    jle     nt!KiReadyThread+0x138 (80501a40)
8050196c 8db228010000    lea     esi,[edx+128h]
80501972 c60601          mov     byte ptr [esi],1
80501975 894708          mov     dword ptr [edi+8],eax
80501978 8a1e            mov     bl,byte ptr [esi]
8050197a c60600          mov     byte ptr [esi],0
8050197d 0fbe4a33        movsx   ecx,byte ptr [edx+33h]
80501981 8b35a0bd5480    mov     esi,dword ptr [nt!KeTickCount (8054bda0)]
80501987 8bc2            mov     eax,edx
80501989 897268          mov     dword ptr [edx+68h],esi
8050198c 8b5244          mov     edx,dword ptr [edx+44h]
8050198f 807a6500        cmp     byte ptr [edx+65h],0
80501993 749c            je      nt!KiReadyThread+0x29 (80501931)
80501995 c6402d01        mov     byte ptr [eax+2Dh],1   
80501999 c6802901000001  mov     byte ptr [eax+129h],1
805019a0 83c060          add     eax,60h
805019a3 8d4a40          lea     ecx,[edx+40h]
805019a6 8b7104          mov     esi,dword ptr [ecx+4]
805019a9 8908            mov     dword ptr [eax],ecx
805019ab 897004          mov     dword ptr [eax+4],esi
805019ae 8906            mov     dword ptr [esi],eax
805019b0 894104          mov     dword ptr [ecx+4],eax
805019b3 807a6501        cmp     byte ptr [edx+65h],1
805019b7 0f85bb000000    jne     nt!KiReadyThread+0x170 (80501a78)
805019bd c6426502        mov     byte ptr [edx+65h],2
805019c1 a1e03d5580      mov     eax,dword ptr [nt!KiProcessInSwapListHead (80553de0)]
805019c6 8d7248          lea     esi,[edx+48h]
805019c9 8975f8          mov     dword ptr [ebp-8],esi
805019cc 8945fc          mov     dword ptr [ebp-4],eax
805019cf 8906            mov     dword ptr [esi],eax
805019d1 8bf8            mov     edi,eax
805019d3 8b45fc          mov     eax,dword ptr [ebp-4]
805019d6 b9e03d5580      mov     ecx,offset nt!KiProcessInSwapListHead (80553de0)
805019db 8b55f8          mov     edx,dword ptr [ebp-8]
805019de 0fb111          cmpxchg dword ptr [ecx],edx
805019e1 3bc7            cmp     eax,edi
805019e3 8945fc          mov     dword ptr [ebp-4],eax
805019e6 75e7            jne     nt!KiReadyThread+0xc7 (805019cf)
805019e8 eb2f            jmp     nt!KiReadyThread+0x111 (80501a19)
805019ea 66ff4260        inc     word ptr [edx+60h]
805019ee 8d7060          lea     esi,[eax+60h]
805019f1 c6402d06        mov     byte ptr [eax+2Dh],6
805019f5 a1d83d5580      mov     eax,dword ptr [nt!KiStackInSwapListHead (80553dd8)]
805019fa 8975f8          mov     dword ptr [ebp-8],esi
805019fd 8945fc          mov     dword ptr [ebp-4],eax
80501a00 8906            mov     dword ptr [esi],eax
80501a02 8bf8            mov     edi,eax
80501a04 8b45fc          mov     eax,dword ptr [ebp-4]
80501a07 b9d83d5580      mov     ecx,offset nt!KiStackInSwapListHead (80553dd8)
80501a0c 8b55f8          mov     edx,dword ptr [ebp-8]
80501a0f 0fb111          cmpxchg dword ptr [ecx],edx
80501a12 3bc7            cmp     eax,edi
80501a14 8945fc          mov     dword ptr [ebp-4],eax
80501a17 75e7            jne     nt!KiReadyThread+0xf8 (80501a00)
80501a19 e8446bffff      call    nt!KiSetSwapEvent (804f8562)
80501a1e eb58            jmp     nt!KiReadyThread+0x170 (80501a78)
80501a20 832584bd548000  and     dword ptr [nt!KiIdleSummary (8054bd84)],0
80501a27 eb12            jmp     nt!KiReadyThread+0x133 (80501a3b)
80501a29 8b5704          mov     edx,dword ptr [edi+4]
80501a2c 0fbe7233        movsx   esi,byte ptr [edx+33h]
80501a30 3bce            cmp     ecx,esi
80501a32 7e0c            jle     nt!KiReadyThread+0x138 (80501a40)
80501a34 c6822801000001  mov     byte ptr [edx+128h],1
80501a3b 894708          mov     dword ptr [edi+8],eax
80501a3e eb38            jmp     nt!KiReadyThread+0x170 (80501a78)
80501a40 c6402d01        mov     byte ptr [eax+2Dh],1   
80501a44 83c060          add     eax,60h
80501a47 84db            test    bl,bl
       //将 DispatcherReadyListHead 所有状态为 ready 的线程链表头地址保存到 edx
       //
80501a49 8d14cd20485580  lea     edx,nt!KiDispatcherReadyListHead (80554820)[ecx*8]
       // [br =1]跳转， 
80501a50 740e            je      nt!KiReadyThread+0x158 (80501a60)
80501a52 8b32            mov     esi,dword ptr [edx]
80501a54 8930            mov     dword ptr [eax],esi
80501a56 895004          mov     dword ptr [eax+4],edx
80501a59 894604          mov     dword ptr [esi+4],eax
80501a5c 8902            mov     dword ptr [edx],eax
80501a5e eb0d            jmp     nt!KiReadyThread+0x165 (80501a6d)
       //将状态为 ready 的线程链表的最顶层元素出栈(堆栈原理)，发送到 esi
80501a60 8b7204          mov     esi,dword ptr [edx+4]
80501a63 8910            mov     dword ptr [eax],edx
80501a65 897004          mov     dword ptr [eax+4],esi
80501a68 8906            mov     dword ptr [esi],eax
80501a6a 894204          mov     dword ptr [edx+4],eax
80501a6d 33c0            xor     eax,eax
80501a6f 40              inc     eax
80501a70 d3e0            shl     eax,cl
       // inc eax，函数 KiReadySummary 可能是统计有多少 read 线程
80501a72 09058cbd5480    or      dword ptr [nt!KiReadySummary (8054bd8c)],eax
80501a78 5f              pop     edi
80501a79 5e              pop     esi
80501a7a 5b              pop     ebx
80501a7b c9              leave
80501a7c c3              ret
80501a7d cc              int     3
80501a7e cc              int     3
```



80501a7f cc             int     3  

80501a80 cc             int     3  

80501a81 cc             int     3  

  `KiReadyThread` 这个函数返回以后呢，回到 `nt!KiDispatchInterrupt` 函数。  

  查看堆栈：  

```
kd> kb  

ChildEBP RetAddr  Args to Child          

f0a94d48 806d3ca4 00000000 ffdff980 80542076 nt!KiDispatchInterrupt+0x78  
f0a94d54 80542076 00000000 000000d1 0012ff80 hal!HalEndSystemInterrupt+0x54  
f0a94d54 0040103c 00000000 000000d1 0012ff80 nt!KeUpdateSystemTime+0x13e  
0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x2c [e:\\temp program\\dbgtcb\\dbgtcb.cpp @ 9]  
0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]  
0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23  
```

  `KiDispatchInterrupt` 函数的部分反汇编代码如下：  

```
8054287a 89742408        mov     dword ptr [esp+8],esi  
8054287e 897c2404        mov     dword ptr [esp+4],edi  
80542882 892c24          mov     dword ptr [esp],ebp  
80542885 8bf0            mov     esi,eax  
80542887 8bbb24010000    mov     edi,dword ptr [ebx+124h]  
8054288d c7832801000000000000 mov dword ptr [ebx+128h],0  
80542897 89b324010000    mov     dword ptr [ebx+124h],esi  
8054289d 8bcf            mov     ecx,edi  
8054289f c6475001        mov     byte ptr [edi+50h],1  

      //调用 nt!KiReadyThread 函数  
805428a3 e860f0fbff      call    nt!KiReadyThread (80501908)  
805428a8 b101            mov     cl,1  

      //下面调用 nt!SwapContext,这里才是线程切换的真正函数。  

      //这里我们要 F11 进去看看，线程切换的精华就是这个函数，下面会有分析，这里暂时跳过。         

805428aa e831000000      call    nt!SwapContext (805428e0)  
805428af 8b2c24          mov     ebp,dword ptr [esp]  
805428b2 8b7c2404        mov     edi,dword ptr [esp+4]  
805428b6 8b742408        mov     esi,dword ptr [esp+8]  
805428ba 83c40c          add     esp,0Ch  
805428bd c3              ret  
805428be c783ac09000000000000 mov dword ptr [ebx+9ACh],0  
805428c8 e8e9e2fbff      call    nt!KiQuantumEnd (80500bb6)  
805428cd 0bc0            or      eax,eax  
805428cf 75a6            jne     nt!KiDispatchInterrupt+0x47 (80542877)  
805428d1 c3              ret  
805428d2 8da42400000000  lea     esp,[esp]  
805428d9 8da42400000000  lea     esp,[esp]  

nt!SwapContext:  

805428e0 0ac9            or      cl,cl  
805428e2 26c6462d02      mov     byte ptr es:[esi+2Dh],2  
805428e7 9c              pushfd  
805428e8 8b0b            mov     ecx,dword ptr [ebx]  
805428ea 83bb9409000000 cmp     dword ptr [ebx+994h],0  
805428f1 51              push    ecx  
805428f2 0f8535010000    jne     nt!SwapContext+0x14d (80542a2d)  
```

  ......  

```
805428aa e831000000      call    nt!SwapContext (805428e0)  

      //前面 f11 跟进 nt!SwapContext，走了一通，回来的时候 ret 在下面的指令上  
      
      //也就是说，在这个函数里，根据不同情况，在不同位置调用 swapcontext  
      
      //保存堆栈栈顶到 ebp  
805428af 8b2c24          mov     ebp,dword ptr [esp]  
805428b2 8b7c2404        mov     edi,dword ptr [esp+4]  
805428b6 8b742408        mov     esi,dword ptr [esp+8]  
805428ba 83c40c          add     esp,0Ch  
805428bd c3              ret  
```

      //执行 `ret` 后，返回到  

      `hal!HalEndSystemInterrupt` 函数  

      查看堆栈  

```
kd> kb  

ChildEBP RetAddr  Args to Child          

f0a94d54 80542076 00000000 000000d1 0012ff80 hal!HalEndSystemInterrupt+0x27  
f0a94d54 0040103c 00000000 000000d1 0012ff80 nt!KeUpdateSystemTime+0x13e  
0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x2c [e:\\temp program\\dbgtcb\\dbgtcb.cpp @ 9]  
0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]  
0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23  
```

      `ret` 返回后，到函数 `nt!KeUpdateSystemTime`  

      查看堆栈：  

```
kd> kb  

ChildEBP RetAddr  Args to Child          

f0a94d64 0040103c badb0d00 00430a00 f0a94d98 nt!KeUpdateSystemTime+0x13e  
0012ff80 00401199 00000001 00430aa0 00430a00 dbgTCB!main+0x2c [e:\\temp program\\dbgtcb\\dbgtcb.cpp @ 9]  
0012ffc0 7c817067 00390038 00370036 7ffde000 dbgTCB!mainCRTStartup+0xe9 [crt0.c @ 206]  
0012fff0 00000000 004010b0 00000000 78746341 kernel32!BaseProcessStart+0x23  
```

  接上，又进行一系列的工作指令之后，回到了 `nt!KiReadyThread` 函数，  

  ..大致如上，可以看出线程切换的一个大概过程  

  分析 `nt!SwapContext` 函数  

  查看当前堆栈：  

```
kd> kb  

ChildEBP RetAddr  Args to Child          

f0a94d64 0040103c badb0d00 00430a00 f0a94d98 nt!SwapContext  
f0a94ddc 80542dd2 f956ab85 82064280 00000000 dbgTCB!main+0x2c [e:\\temp program\\dbgtcb\\dbgtcb.cpp @ 9]  
f0a94e2c 7c93017b 7c9301bb 0000dbbc 00000000 nt!KiThreadStartup+0x16  

WARNING: Process directory table base 04F00260 doesn't match CR3 04F002E0  
WARNING: Process directory table base 04F00260 doesn't match CR3 04F002E0  

00000000 00000000 00000000 00000000 00000000 ntdll!RtlAllocateHeap+0x1c2  
```

  `nt!SwapContex` 函数的部分反汇编如下：  

  `nt!SwapContext`:  

```
805428e0 0ac9            or      cl,cl  

      // 这里就比较熟悉了，0x2 状态值应该就是线程工作状态：running  
      
      // esi 应该就是指向一个新线程的 \_ethread.kthread 结构，然后偏移 2d，就是 state 字段了  
      
      // edi 指向旧线程的 \_ethread.kthread 结构 edx 指向 KPCR  
805428e2 26c6462d02      mov     byte ptr es:[esi+2Dh],2    es:0023:82025b5d=03  
805428e7 9c              pushfd  
805428e8 8b0b            mov     ecx,dword ptr [ebx]  
805428ea 83bb9409000000 cmp     dword ptr [ebx+994h],0  
805428f1 51              push    ecx  
805428f2 0f8535010000    jne     nt!SwapContext+0x14d (80542a2d)  
805428f8 833d0cbf558000  cmp     dword ptr [nt!PPerfGlobalGroupMask (8055bf0c)],0  
805428ff 0f85ff000000    jne     nt!SwapContext+0x124 (80542a04)  
80542905 0f20c5          mov     ebp,cr0  
80542908 8bd5            mov     edx,ebp  

      //和上面的分析一样，2c 偏移处的字段需要关注 ,DebugActive :UChar  
8054290a 8a4e2c          mov     cl,byte ptr [esi+2Ch]  
8054290d 884b50          mov     byte ptr [ebx+50h],cl  

    //这里关闭中断，看来要进行一些修改属性的操作了  
80542910 fa              cli  

    //保存当前栈到旧线程的 Kthread->KernelStack 中  

80542911 896728          mov     dword ptr [edi+28h],esp  

    // 18 偏移 nitialStack:Ptr32 Void 初始化堆栈 新线程 Kthread->InitialStack 系统空间栈底  
80542914 8b4618          mov     eax,dword ptr [esi+18h]  

    // 1c 偏移 StackLimit:Ptr32 Void  
80542917 8b4e1c          mov     ecx,dword ptr [esi+1Ch]  
8054291a 2d10020000      sub     eax,210h  
8054291f 894b08          mov     dword ptr [ebx+8],ecx  
80542922 894304          mov     dword ptr [ebx+4],eax  
80542925 33c9            xor     ecx,ecx  

    //31 偏移 NpxState:UChar   新线程 KThread->NpxState    

    ////清除 NPX 标志位  
8054292a 83e2f1          and     edx,0FFFFFFF1h   
8054292d 0bca            or      ecx,edx  
8054292f 0b880c020000    or      ecx,dword ptr [eax+20Ch]  
80542935 3be9            cmp     ebp,ecx  
80542937 0f85bf000000    jne     nt!SwapContext+0x11c (805429fc)  
8054293d 8d4900          lea     ecx,[ecx]  
80542940 f740e400000200  test    dword ptr [eax-1Ch],20000h  
80542947 7503            jne     nt!SwapContext+0x6c (8054294c)  
80542949 83e810          sub     eax,10h  
8054294c 8b4b40          mov     ecx,dword ptr [ebx+40h]  //KPCR->TSS   
8054294f 894104          mov     dword ptr [ecx+4],eax     
80542952 8b6628          mov     esp,dword ptr [esi+28h]  //开始切换，装载新线程 KTHREAD->KernelStack 到 esp    

    //偏移 20，Teb  
80542955 8b4620          mov     eax,dword ptr [esi+20h]  
80542958 894318          mov     dword ptr [ebx+18h],eax  //KPCR->NT\_TIB->Self，更新 KPCR 线程环境快(TEB)  

    //执行完上面的操作后，重新开启中断，看来上面的操作过程中，不希望有任何操作打断  
      
    //从而保证这个原子操作的完整性  
8054295b fb              sti  
8054295c 8b4744          mov     eax,dword ptr [edi+44h]  //旧线程的 PROCESS    
8054295f 3b4644          cmp     eax,dword ptr [esi+44h]  //比较新旧线程所属的 PROCESS 是否一样   
80542962 c6475000        mov     byte ptr [edi+50h],0  
80542966 742c            je      nt!SwapContext+0xb4 (80542994) //一样就跳转  
80542968 8b7e44          mov     edi,dword ptr [esi+44h]     
```



```assembly
8054296b 66f74720ffff    test    word ptr [edi+20h],0FFFFh
80542971 755b            jne     nt!SwapContext+0xee (805429ce)
80542973 33c0            xor     eax,eax
80542975 0f00d0          lldt    ax
80542978 33c0            xor     eax,eax
8054297a 8ee8            mov     gs,ax
8054297c 8b4718          mov     eax,dword ptr [edi+18h]
8054297f 8b6b40          mov     ebp,dword ptr [ebx+40h]
80542982 8b4f30          mov     ecx,dword ptr [edi+30h]
80542985 89451c          mov     dword ptr [ebp+1Ch],eax
80542988 0f22d8          mov     cr3,eax      ////当切换的线程属于不同的进程时，切换 CR3。
8054298b 66894d66        mov     word ptr [ebp+66h],cx
8054298f eb03            jmp     nt!SwapContext+0xb4 (80542994)
80542991 8d4900          lea     ecx,[ecx]
80542994 8b4318          mov     eax,dword ptr [ebx+18h]
80542997 8b4b3c          mov     ecx,dword ptr [ebx+3Ch]
8054299a 6689413a        mov     word ptr [ecx+3Ah],ax
8054299e c1e810          shr     eax,10h
805429a1 88413c          mov     byte ptr [ecx+3Ch],al
805429a4 88613f          mov     byte ptr [ecx+3Fh],ah
805429a7 ff464c          inc     dword ptr [esi+4Ch]
805429aa ff831c060000    inc     dword ptr [ebx+61Ch]
805429b0 59              pop     ecx
805429b1 890b            mov     dword ptr [ebx],ecx
805429b3 807e4900        cmp     byte ptr [esi+49h],0
805429b7 7504            jne     nt!SwapContext+0xdd (805429bd)
805429b9 9d              popfd
805429ba 33c0            xor     eax,eax
805429bc c3              ret    //切换结束
```

简单分析了下 `SwapContext` 中感兴趣的字段；

综上，`SwapContext` 正如函数其名，就是将新老线程的 `EThread` 的某些状态值改变，保存旧线程的堆栈，

初始化新线程的工作环境，如此而已；其余的细节可以自己详细分析。

现在查看堆栈：

`nt!SwapContext` 执行 `ret` 后，返回到 `nt!KiSwapContext`

```text
kd> kb

ChildEBP RetAddr  Args to Child
f08ce974 80501cd6 00000000 82025b30 804faae2 nt!KiSwapContext+0x2e
f08ce980 804faae2 82025b30 00000001 00000000 nt!KiSwapThread+0x46
f08ce9b8 805b7418 00000001 f08cebec 00000001 nt!KeWaitForMultipleObjects+0x284
f08ced48 8053e638 00000001 00b8fea0 00000001 nt!NtWaitForMultipleObjects+0x2a2
f08ced48 7c92e4f4 00000001 00b8fea0 00000001 nt!KiFastCallEntry+0xf8
00b8fe74 7c92df2c 7c809574 00000001 00b8fea0 ntdll!KiFastSystemCallRet
00b8fe78 7c809574 00000001 00b8fea0 00000001 ntdll!NtWaitForMultipleObjects+0xc
00b8ff14 00401c39 00000001 003bad10 00000000 kernel32!WaitForMultipleObjectsEx+0x12c
00b8ff5c 004024f9 000186a0 00000000 003b75a0 dbgTCB!_XcptFilter+0x79 [winxfltr.c @ 267]
00b8ff98 0040e1fe 00158b50 77dc352b 00000001 dbgTCB!__crtGetEnvironmentStringsA+0xc9 [a_env.c @ 96]
00b8ffb4 7c80b713 00158b50 00000000 0012e814 dbgTCB!CloseHandle+0x2d52
00b8ffec 00000000 77dc3519 00158b50 00000000 kernel32!BaseThreadStart+0x37
```

跳转到 `nt!KiSwapContext` 函数之后，又 `ret` 到 `nt!KiSwapThread` 函数

大概流程就这样。细节并没有照顾太多，主要是说明，如何根据自己的需要去调试系统内核，找到自己关心的。

### 关于总结

1. 我们关心的是一个线程获得 `cpu` 以后，实际执行的时间；在用户层，我们知道精确获得一个线程获取 `CPU` 的时间，可以用 `TLS` 机制；在内核里，可以就可以更加精确地知道，当线程获取 `CPU` 以后，只有 `nt!_kthread` `81f862a8+0+2d` 处的 `State` 字段为 `running` 的时候，才算是实际实行；只要这个值改变，实际的一段执行时间就被切换或其他原因 `interrupt` 掉了。

2. 关于线程切换，实在是一个大的流程，上面只是涉及到了一定如何跟踪的实际技巧。如果想详细研究，建议先看看 `Windows Internal` 的线程切换相关部分，在自己调试，就容易多了。

以上权作学习笔记，错误之处，勿见笑。

`dbgEthread.txt`
