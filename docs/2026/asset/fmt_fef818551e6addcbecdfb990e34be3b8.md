
```
0: kd> u KdEnterDebugger l50

nt!KdEnterDebugger:

809cd36c 8bff            mov     edi,edi

809cd36e 55              push    ebp

809cd36f 8bec            mov     ebp,esp

809cd371 837d0800        cmp     dword ptr [ebp+8],0

809cd375 742c            je      nt!KdEnterDebugger+0x37 (809cd3a3)

809cd377 ff7508          push    dword ptr [ebp+8]

809cd37a e83f200000      call    nt!KdpQueryPerformanceCounter (809cf3be)

809cd37f a388869e80      mov     dword ptr [nt!KdTimerStop (809e8688)],eax

809cd384 2b0580869e80    sub     eax,dword ptr [nt!KdTimerStart (809e8680)]

809cd38a 89158c869e80    mov     dword ptr [nt!KdTimerStop+0x4 (809e868c)],edx

809cd390 1b1584869e80    sbb     edx,dword ptr [nt!KdTimerStart+0x4 (809e8684)]

809cd396 a390869e80      mov     dword ptr [nt!KdTimerDifference (809e8690)],eax

809cd39b 891594869e80    mov     dword ptr [nt!KdTimerDifference+0x4 (809e8694)],edx

809cd3a1 eb0e            jmp     nt!KdEnterDebugger+0x45 (809cd3b1)

809cd3a3 832588869e8000  and     dword ptr [nt!KdTimerStop (809e8688)],0

809cd3aa 83258c869e8000  and     dword ptr [nt!KdTimerStop+0x4 (809e868c)],0

809cd3b1 53              push    ebx

809cd3b2 56              push    esi

809cd3b3 648b3520000000  mov     esi,dword ptr fs:[20h]

809cd3ba ff1594108080    call    dword ptr [nt!_imp__KeGetCurrentIrql (80801094)]

809cd3c0 ff750c          push    dword ptr [ebp+0Ch]

809cd3c3 888645050000    mov     byte ptr [esi+545h],al

809cd3c9 ff7508          push    dword ptr [ebp+8]

809cd3cc e8ab31e6ff      call    nt!KeFreezeExecution (8083057c)

809cd3d1 b980438b80      mov     ecx,offset nt!KdpDebuggerLock (808b4380)

809cd3d6 8ad8            mov     bl,al

809cd3d8 e83fb8ebff      call    nt!KeTryToAcquireSpinLockAtDpcLevel (80888c1c)

809cd3dd 6a00            push    0

809cd3df a220879e80      mov     byte ptr [nt!KdpPortLocked (809e8720)],al

809cd3e4 e81597ebff      call    nt!KdSave (80886afe)

809cd3e9 5e              pop     esi

809cd3ea 8ac3            mov     al,bl

809cd3ec c705885b8a8001000000 mov dword ptr [nt!KdEnteredDebugger (808a5b88)],1    // 重点在这里

809cd3f6 5b              pop     ebx

809cd3f7 5d              pop     ebp

809cd3f8 c20800          ret     8
```

我们就直接`asm` `KdEnterDebugger` 这个函数 让他对 `nt!KdEnteredDebugger` 置 0

```
//windbg 执行修改代码

0: kd> eb 809cd3ec c7 05 88 5b 8a 80 00

//结果

809cd3ec c705885b8a8000000000 mov dword ptr [nt!KdEnteredDebugger (808a5b88)],0
```

大家要注意 我这样修改是为了方便马上得到结果才这样做的 这样可能会对系统照成不稳定 所以希望大家不要学我

//已经验证了 `KdEnteredDebugger` 不会被置 1 了 我们运行游戏看看

0: kd> g

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

Tenio[Error]: To remove a component not exist

Tenio[Error]: IComponent\* = 0x0130C00C

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

[Tenio_Error] Can't find Component(33554448) Creator From Dll List!

Rule ModuleEntry Start---------------------------

LPDispachIoRequest=F662C008

Rule ModuleEntry end-----------------------------

![](https://bbs.kanxue.com/upload/attach/201403/144390_5f2w6vadq3sysgp.jpg)

某驱动已经加载了 而且并没有蓝屏  这里我们得出结论了 某驱动就是对`KdEnteredDebugger` 的值进行判断 是否开启了内核

调试引擎的 因为我直接修改`KdEnteredDebugger()` 函数 导致某驱动可能初始化出问题 没出现游戏界面一直卡着了

但是能定位到他是检测什么的了 我相信朋友们要绕过它 PASS它 FUCK它 都有各自的奇招 我就不献丑我如何逆向他的了

定位到他关键代码：

![](https://bbs.kanxue.com/upload/attach/201403/144390_qh13bsnnmlzjst2.jpg)

大概的流程是获取到 `KdEnteredDebugger` 地址 然后创建`MDL`   然后判断用`MDL`返回的指针 来判断 `KdEnteredDebugger` 的值 所以我用了个很简单的方法来绕过他 就是最简单的`HOOK`

```
typedef PMDL (__stdcall *_MyIoAllocateMdl)(

           __in_opt PVOID  VirtualAddress,

           __in ULONG  Length,

           __in BOOLEAN  SecondaryBuffer,

           __in BOOLEAN  ChargeQuota,

           __inout_opt PIRP  Irp  OPTIONAL);



_MyIoAllocateMdl old_IoAllocateMdl;
```

```
PMDL MyIoAllocateMdl(

        __in_opt PVOID  VirtualAddress,

        __in ULONG  Length,

        __in BOOLEAN  SecondaryBuffer,

        __in BOOLEAN  ChargeQuota,

        __inout_opt PIRP  Irp  OPTIONAL)

{

  if (pKdEnteredDebugger == VirtualAddress)

  {

    VirtualAddress = pKdEnteredDebugger + 0x20;  //+0x20  是让他读到其他的位置

  }

  

  return old_IoAllocateMdl(VirtualAddress,Length,SecondaryBuffer,ChargeQuota,Irp);

}





//////////////////////////////////////////////////////////////////////////
//模块加载回调函数例程

VOID LoadImageRoutine(IN PUNICODE_STRING  FullImageName,

            IN HANDLE  ProcessId, // where image is mapped

            IN PIMAGE_INFO  ImageInfo)

{

  PMDL pMdl;

  ULONG uHookAddr,uJmpAddr;

  if (wcsstr(FullImageName->Buffer,L"TesSafe.sys") != NULL)

  {

    KdPrint(("TesSafe.sys -------------加载-------------\r\n"));

    KdPrint(("TesSafe.sys --->ImageBase 0x%x\r\n",(ULONG)ImageInfo->ImageBase));

    KdPrint(("TesSafe.sys --->ImageSize 0x%x\r\n",(ULONG)ImageInfo->ImageSize));

    KdPrint(("断点命令: ba e 1 0x%x+ \r\n",(ULONG)ImageInfo->ImageBase));

    KdBreakPoint();



    InlineHook(IoAllocateMdl,MyIoAllocateMdl,(void**)&old_IoAllocateMdl);

　　}

　　return ;

}
```

编译后运行 成功绕过了 内核调试引擎检测

![](https://bbs.kanxue.com/upload/attach/201403/144390_974jgo7el3sozsq.jpg)

但是虚拟机不能下断点了 呵呵 老样子 它还是回阻止你调试 但这已经不是本章的内容了 不蓝屏了 朋友们可以慢慢一步一步的分析

总结：

1.通过了解win内核调试引擎的加载机制 解决了我们想要解决的问题

2.

`KdPitchDebugger`

`KdDebuggerEnabled`

`KiDebugRoutine`

`KdDebuggerNotPresent`

`KdEnteredDebugger`

3.知道了 以上5个全局变量 都可以来判断 内核调试是否开启

了解`WINDBG` 和 内核引擎是如何完成初始化操作的

本章内容多数采自《软件调试》一书

![](https://bbs.kanxue.com/upload/attach/201403/144390_7cmjtojtm0er0l1.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

#系统底层

上传的附件：

- [1.jpg](javascript:void(0);)
  （84.70kb，41次下载）
- [2.jpg](javascript:void(0);)
  （167.49kb，348次下载）
- [3.jpg](javascript:void(0);)
  （123.09kb，155次下载）
- [4.jpg](javascript:void(0);)
  （108.86kb，109次下载）
- [5.jpg](javascript:void(0);)
  （117.59kb，64次下载）
- [6.jpg](javascript:void(0);)
  （278.22kb，59次下载）
- [7.jpg](javascript:void(0);)
  （71.34kb，14次下载）
- [8.jpg](javascript:void(0);)
  （151.22kb，126次下载）
- [9.jpg](javascript:void(0);)
  （408.75kb，43次下载）
- [10.jpg](javascript:void(0);)
  （157.71kb，235次下载）
