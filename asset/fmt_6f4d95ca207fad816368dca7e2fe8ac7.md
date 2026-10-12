
# 散谈游戏保护那点事~就从 TP 开始入手吧

> 原文：[`www.pediy.com/kssd/pediy11/126802.html`](https://www.pediy.com/kssd/pediy11/126802.html)

声明：本文只为研究技术，请所有童鞋切勿使用本文之方法做下那天理难容罪恶不舍之坏事。

既是研究游戏保护，那么总要有一个研究对象。本文就以 `TMD_TP` 这款游戏保护为例进行分析讲解。请勿对号入座，如有雷同之处。纯属反汇编引擎之错误，不关我的事！

转载请注明出处

关键字：`DNF` 驱动保护

鉴于最近很多同学找上门来求解这那问题，反正这东西又不是绝密档案，放在我手里大半个月了，还不如放出来让大家一起进步算了。另外都是取之看雪还之看雪罢了。

索性我也就公布一个全套的方案。绝无其他意思，所以还请同道中人嘴下留情。切勿背地使坏！`![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")`

在正式开篇之前我要感谢看雪 ID:`十年寒窗` 在我最困惑的时候，他给予了最大的帮助！另外还有一位和我同岁的神秘人物也给予了不小的帮助，感谢你们。

废话了半天，正式开始吧。

`tmd_TP` 也就是国内比较流行的游戏 `D_N*F` 的游戏保护。

它在 `ring0` 层一共 HOOK 了几个地方和一些其他的工作。来达到保护的目的

下面是简报:

引用:
- `NtOpenThread`    //防止调试器在它体内创建线程
- `NtOpenProcess`   //防止 `OD` 等在进程列表看到它
- `KiAttachProcess`   //防止其他软件附加它
- `NtReadVirtualMemory`  //防止别人读取它的内存
- `NtWriteVirtualMemory`  //防止别人在它的内存里面乱写乱画
- `KDCOM.dll:KdReceivePacket`  //这两个是 COM 串口的接受和发送数据
- `KDCOM.dll:KdSendPacket`     //主要用来方式别人双机调试

使用了 `KdDisableDebugger` 来禁用双机调试

代码:

```
.text:010025F0                 jz      short loc_1002622
.text:010025F2                 call    sub_10022A4
.text:010025F7                 call    ds:KdDisableDebugger
.text:010025FD                 push    offset byte_10022EC
.text:01002602                 push    esi
.text:01002603                 push    offset byte_10022DC
.text:01002608                 push    edi
.text:01002609                 push    dword_100CF24
```

并对 `debugport` 进行了疯狂的清零操作

甚至包括 `EPROCESS+70/+74/+78` 等几处位置

`![](img/d29155bd42659779e03d1162c616ce04.jpg)`

处理的手段通常都是向 64 端口写入 `FE` 导致计算机被重启

代码:

```
.text:01001665                 mov     al, 0FEh
.text:01001667                 out     64h, al         ; AT Keyboard controller 8042.
.text:01001667                                         ; Resend the last transmission
.text:01001669                 popa
.text:0100166A                 retn
```

下面简单看下他关键的几个 HOOK：

`KiAttachProcess`

`![](img/2aaf6f41693e00e546961f1e17e3a7b1.jpg)`

`NtReadVirtualMemory`

`![](img/3089b0c3f18d1722917e19b955c3dd69.jpg)`

`NtWriteVirtualMemory`

`![](img/a83ffd375af6a47099d271eff7192814.jpg)`

`NtOpenThread`

`![](img/b4684f120b63c763fa2ce079fe591bbe.jpg)`

`NtOpenProcess`

`![](img/25fbcd1f3e48d9a3d13de90542f5ca2a.jpg)`

引用: 其中，前 3 个直接恢复即可。

第 4 个有监视，直接恢复即刻重启

第 5 个和 `ring3` 有通信，直接恢复 1 分钟内 `SX` 非法模块

根据上面的分析，下面给出相应的解决方案

1.直接恢复 第 1、2、3 处 HOOK

2.绕过 4、5 处 HOOK

3.将 `debugport` 清零的内核线程干掉

4.恢复硬件断点

但是要有一个先后的逻辑顺序

因为内核有一个线程负责监视几个地方，必须要先干掉它。

但是这个内容我写在了处理 `debugport` 清零的一起，也就是第 3 步。所以大家在照搬源码的时候

注意代码执行次序

先从简单的工作讲起，恢复 1、2、3 处的 HOOK

### KiAttachProcess 的处理

代码:

```
//////////////////////////////////////////////////////////////////////
//  名称:  Nakd_KiAttachProcess
//  功能:  My_RecoveryHook_KiAttachProcess 的中继函数
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
static NAKED VOID  Nakd_KiAttachProcess()
{
  __asm
  {
    mov     edi,edi
    push    ebp
    mov     ebp,esp
    push    ebx
    push    esi
    mov    eax,KiAttachProcessAddress  //注意这个是全局变量 BYTE*
    add    eax,7
    jmp    eax
  }
}
//////////////////////////////////////////////////////////////////////
//  名称:  RecoveryHook_KiAttachProcess
//  功能:  解除游戏保护对_KiAttachProcess 函数的 HOOK(DNF)
//  参数:  
//  返回:  状态
//////////////////////////////////////////////////////////////////////
NTSTATUS My_RecoveryHook_KiAttachProcess()
{
  BYTE    *KeAttachProcessAddress = NULL;  //KeAttachProcess 函数地址
  BYTE    *p;
  BYTE    MovEaxAddress[5]  = {0xB8,0,0,0,0};  
  BYTE    JmpEax[2]      = {0xff,0xe0};
  KIRQL   Irql;
  //特征码
  BYTE  Signature1 = 0x56,  //p-1
      Signature2 = 0x57,  //p-2
      Signature3 = 0x5F,  //p-3
      Signature4 = 0x5E,  //p+5
      Signature5 = 0xE8;  //p 第一个字节
 
  //获得 KeAttachProcess 地址,然后通过特征码找到
  //KiAttachProcess 的地址
  KeAttachProcessAddress = (BYTE*)MyGetFunAddress(L"KeAttachProcess");
  if (KeAttachProcessAddress == NULL)
  {
    KdPrint(("KeAttachProcess 地址获取失败\n"));
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
  //将 p 指向 KeAttachProcess 函数开始处
  p = KeAttachProcessAddress;
  while (1)
  {
    if ((*(p-1) == Signature1) &&
      (*(p-2) == Signature2) &&
      (*(p+5) == Signature3) &&
      (*(p+6) == Signature4) &&
      (*p    == Signature5))
    {
      //定位成功后取地址
      KiAttachProcessAddress = *(PULONG)(p+1)+(ULONG)(p+5);
      break;
    }
 
    //推动指针
    p++;
  }
 
  //计算中继函数地址
  *(ULONG *)(MovEaxAddress+1)=(ULONG)Nakd_KiAttachProcess;
 
  WPOFF();  //清除 CR0
  //提升 IRQL 中断级
  Irql=KeRaiseIrqlToDpcLevel();
  //写入
  RtlCopyMemory(KiAttachProcessAddress,MovEaxAddress,5);
  RtlCopyMemory(KiAttachProcessAddress+5,JmpEax,2);
  //恢复 Irql
  KeLowerIrql(Irql);
  WPON();    //恢复 CR0
 
  return  STATUS_SUCCESS;
}
```

### NtReadVirtualMemory 和 NtWriteVirtualMemory 的处理

注意这里，我对他们俩开头的第 2 句 `PUSH` 的处理

我直接写入了 `push 0x78563412`

大家可以根据自己的地址来硬编码一次。

或者干脆这样使用

代码:

```
//////////////////////////////////////////////////////////////////////
//  名称:  My_RecoveryHook_NtReadAndWriteMemory
//  功能:  解除游戏保护对 NtReadVirtualMemory 和
//      NtWriteVirtualMemory 的 HOOK
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
NTSTATUS My_RecoveryHook_NtReadAndWriteMemory()
{
  BYTE  Push1Ch[2]  = {0x6a,0x1c};  //0~2 字节
  BYTE  PushAdd[5]  = {0x68,0x12,0x34,0x56,0x78};  //NtReadVirtualMemory[物理机]
  //BYTE  PushAdd2[5]  = {0x68,0xf0,0x6f,0x4f,0x80};  //NtWriteVirtualMemory[物理机]
  KIRQL  Irql;
  BYTE  *NtReadVirtualMemoryAddress    = NULL;  //NtReadVirtualMemory 的地址
  BYTE  *NtWriteVirtualMemoryAddress  = NULL;  //NtWriteVirtualMemory 的地址
 
  //从 SSDT 表中获取 NtReadVirtualMemory 函数地址
  NtReadVirtualMemoryAddress = (BYTE*)myGetCurrentAddress(0xBA);
  if (NtReadVirtualMemoryAddress == NULL)
  {
    KdPrint(("NtReadVirtualMemory 函数地址获取失败! \n"));
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
  //从 SSDT 表中获取 NtWriteVirtualMemory 函数地址
  NtWriteVirtualMemoryAddress = (BYTE*)myGetCurrentAddress(0x115);
  if (NtWriteVirtualMemoryAddress == NULL)
  {
    KdPrint(("NtWriteVirtualMemory 函数地址获取失败! \n"));
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
 
  WPOFF();  //清除 CR0
  //提升 IRQL 中断级
  Irql=KeRaiseIrqlToDpcLevel();
  //写入
  RtlCopyMemory(NtReadVirtualMemoryAddress,Push1Ch,2);
  RtlCopyMemory(NtReadVirtualMemoryAddress+2,PushAdd,5);
 
  RtlCopyMemory(NtWriteVirtualMemoryAddress,Push1Ch,2);
  RtlCopyMemory(NtWriteVirtualMemoryAddress+2,PushAdd,5);
  //恢复 Irql
  KeLowerIrql(Irql);
  WPON();    //恢复 CR0
 
  return  STATUS_SUCCESS;
}
```

好了，下面来处理

`NtOpenProcess` 和 `NtOpenThread`

这两个函数的处理上不能太鲁莽了。

手法要风骚一点细腻一点了

介于篇幅的原因，我只贴出来前者的处理方法，后者雷同

细微之处大家自行修改。我总不能真的给你方法又给你工具。眼看着自己变成教唆犯

代码:
