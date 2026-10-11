
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



```
//NtOpenProcess 用到的全局变量\[为了方便堆栈平衡的处理使用全局变量\]
PEPROCESS  processEPROCESS = NULL;  //保存访问者的 EPROCESS
ANSI\_STRING  p\_str1,p\_str2;      //保存进程名称
BYTE    \*ObOpenObjectByPointerAddress  = NULL; //ObOpenObjectByPointer 的地址
BYTE    \*p\_TpHookAddress = NULL;        //TP 的 HOOK 函数地址
BYTE    \*p\_ReturnAddress = NULL;        //返回到的地址
BYTE    \*p\_MyHookAddress = NULL;        //我们的 HOOK 函数在哪写入
#define DNF\_EXE  "DNF.exe"  //要检索的进程名
//////////////////////////////////////////////////////////////////////
//  名称:  Nakd\_NtOpenProcess
//  功能:  My\_RecoveryHook\_NtOpenProcess 的中继函数
//  参数:   
//  返回:   
//////////////////////////////////////////////////////////////////////
static NAKED VOID  Nakd\_NtOpenProcess()
{
  //获得调用者的 EPROCESS
  processEPROCESS = IoGetCurrentProcess();
  //将调用者的进程名保存到 str1 中
  RtlInitAnsiString(&p\_str1,(ULONG)processEPROCESS+0x174);
  //将我们要比对的进程名放入 str2
  RtlInitAnsiString(&p\_str2,DNF\_EXE);
  if (RtlCompareString(&p\_str1,&p\_str2,TRUE) == 0)
  {
    //说明是 DNF 进程访问了这里
    \_\_asm
    {
      push    dword ptr \[ebp-38h\]
      push    dword ptr \[ebp-24h\]
      push  p\_ReturnAddress
      mov    eax,p\_TpHookAddress
      jmp    eax
    }
  }
  else
  {
    \_\_asm
    {
      push    dword ptr \[ebp-38h\]
      push    dword ptr \[ebp-24h\]
      push  p\_ReturnAddress
      mov    eax,ObOpenObjectByPointerAddress
      jmp    eax
    }
  }
}
 
//////////////////////////////////////////////////////////////////////
//  名称:  My\_RecoveryHook\_NtOpenProcess
//  功能:  解除游戏保护对 NtOpenProcess 的 HOOK
//  参数:   
//  返回:  状态
//////////////////////////////////////////////////////////////////////
NTSTATUS My\_RecoveryHook\_NtOpenProcess()
{
  BYTE    \*NtOpenProcessAddress      = NULL;  //NtOpenProcess 的地址
  BYTE    \*p = NULL;      //临时
  TOP5CODE  \*top5code = NULL;  //保存 5 字节内容
  BYTE    JmpAddress\[6\] = {0xE9,0,0,0,0,0x90};
  KIRQL    Irql;
 
    //获取 NtOpenProcess 的地址
    NtOpenProcessAddress = (BYTE\*)MyGetFunAddress(L"NtOpenProcess");
    if (NtOpenProcessAddress == NULL)
    {
      KdPrint(("NtOpenProcess 地址获取失败\\n"));
      return  FAILED\_TO\_OBTAIN\_FUNCTION\_ADDRESSES;
    }
    //获取 ObOpenObjectByPointer 的地址
    ObOpenObjectByPointerAddress = (BYTE\*)MyGetFunAddress(L"ObOpenObjectByPointer");
    if (ObOpenObjectByPointerAddress == NULL)
    {
      KdPrint(("ObOpenObjectByPointer 地址获取失败\\n"));
      return  FAILED\_TO\_OBTAIN\_FUNCTION\_ADDRESSES;
    }
 
    //将 p 指向 NtOpenProcess 函数开始处
    p = NtOpenProcessAddress;
    //用一个无限循环来判断给定的特征码来确定被 HOOK 位置
    while (1)
    {
      if ((\*(p-7)    == 0x50) && 
        (\*(p-0xE)  == 0x56) &&
        (\*(p+0xd)  == 0x50) &&
        (\*(p+0x16)  == 0x3b) &&
        (\*(p+0x17)  == 0xce) &&
        (\*p      == 0xE8) &&
        (\*(p+5)    == 0x8b) &&
        (\*(p+6)    == 0xf8))
      {
        KdPrint(("%0X \\n",(ULONG)p));
        break;
      }
      //推动指针向前走
      p++;
    }
 
    //将 top5code 指向 p 的当前处
    //用以取出 call \[地址\] 这 5 字节里面的地址
    top5code = (TOP5CODE\*)p;
    p\_TpHookAddress = (BYTE\*)((ULONG)p+5+top5code->address);
 
    //找到我们写入自定义函数的地址
    p\_MyHookAddress = p-6;
    //保存调用 ObOpenObjectByPointer 函数以后的返回地址
    p\_ReturnAddress = p+5;
 
    //将一条 JMP Nakd\_NtOpenProcess 写入到数组中
    \*(ULONG \*)(JmpAddress+1)=(ULONG)Nakd\_NtOpenProcess - ((ULONG)p\_MyHookAddress+5);
 
    WPOFF();  //清除 CR0
    //提升 IRQL 中断级
    Irql=KeRaiseIrqlToDpcLevel();
    //写入
    RtlCopyMemory(p\_MyHookAddress,JmpAddress,6);
    //恢复 Irql
    KeLowerIrql(Irql);
    WPON();    //恢复 CR0
   
  return  STATUS\_SUCCESS;
}
```

![](img/1e2e6c1b48bf369c87082c470b3e5d46.jpg)

简而言之其原理就是，任何人调用了 `NtOpenProcess` 的时候会先进入 `Nakd_NtOpenProcess` 函数，我们判断。如果是游戏进程访问的话，就有可能是验证之类的，我们转到它自己的函数里面。让它保持与 ring3 层的通信。否则的话，嘿嘿……

接下来是第 3 步处理 `debugport` 清零的这块了。

我想绝大多数人关心的都是这里了，网络上能搜多到的办法几乎都失效了，有办法的人又不肯放出来，急眼了就自己想了个土办法，虽然不那么时尚，但是绝对的奏效。

由于代码凌乱不堪，简单说下其原理。我们定位内核模块 `TxxxSxxx.sys` 的首地址，然后根据特征码遍历整个模块找到我们需要的地方，然后干掉他们。

那么我们又如何能够通过人工的判断出来到底是哪里在作怪呢？利用 `syser` 或 `Start SoftICE` 对 `EPROCESS+BC` 处设置断点。就可以一层一层的追溯上去了。

到底如何用他们，我想大家自己多花点时间在看雪和 `GOOGLE` 或者 `BAIDU` 上面是不会吃亏的。

由于 `ZwQuerySystemInformation` 函数的使用非常繁琐。而且篇幅有限。所以我只给出关键代码，至于这个函数如何使用。大家可以自己在搜索引擎找“枚举内核模块”。

代码:



# 枚举内核模块与恢复调试端口

以下代码展示了如何枚举内核模块以及如何恢复游戏对 `debugport` 的清零操作。

```c
//////////////////////////////////////////////////////////////////////
//  名称:  MyEnumKernelModule
//  功能:  枚举内核模块
//  参数:  str:内核模块名称
//      moduleadd:该模块地址[传出]
//      modulesie:该模块大小[传出]
//  返回:  
//////////////////////////////////////////////////////////////////////
NTSTATUS MyEnumKernelModule(IN CHAR* str,OUT ULONG *moduleadd,OUT ULONG *modulesie)
{
  NTSTATUS status = STATUS_SUCCESS;
  ULONG   n       = 0;
  ULONG   i       = 0;
  PSYSTEM_MODULE_INFORMATION_ENTRY   module = NULL;
  PVOID   pbuftmp = NULL;
  ANSI_STRING    ModuleName1,ModuleName2;
  BOOLEAN  tlgstst= FALSE;  //如果找到了指定模块则设置为 TRUE
 
  //利用 11 号功能枚举内核模块
  status = ZwQuerySystemInformation(11, &n, 0, &n);
 
  //申请内存
  pbuftmp = ExAllocatePool(NonPagedPool, n);
 
  //再次执行,将枚举结果放到指定的内存区域
  status = ZwQuerySystemInformation(11, pbuftmp, n, NULL);
 
  module = (PSYSTEM_MODULE_INFORMATION_ENTRY)((PULONG )pbuftmp + 1 );
 
  //初始化字符串 
  RtlInitAnsiString(&ModuleName1,str);
  //
  n       = *((PULONG)pbuftmp );
  for ( i = 0; i < n; i++ )
  {
    RtlInitAnsiString(&ModuleName2,&module[i].ImageName);
    //DbgPrint("%d\t0x%08X 0x%08X %s\n",module[i].LoadOrderIndex,module[i].Base,module[i].Size,module[i].ImageName);
 
    if (RtlCompareString(&ModuleName1,&ModuleName2,TRUE) == 0)
    {
      DbgPrint("MyEnumKernelModule:%s:%0X \n",ModuleName2.Buffer,module[i].Base);
      *moduleadd  = module[i].Base;
      *modulesie  = module[i].Size;
      tlgstst = TRUE;
      break;
    }
  }
  ExFreePool(pbuftmp);
  if tlgstst == FALSE)
  {
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
  return status;
}
 
//////////////////////////////////////////////////////////////////////
//  名称:  My_Recovery_Debugport
//  功能:  恢复游戏对 debugport 的清零操作
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
NTSTATUS My_Recovery_Debugport()
{
  NTSTATUS stats;
  BYTE  *sd1 = NULL,*sd2 = NULL,*pd = NULL;
  ULONG  ModuleSize,ModuleAddress,i,number = 0;
  BYTE  *p;
  KIRQL  Irql;
  BYTE  C390[2] = {0xc3,0x90};
 
  //获取指定的内核模块地址和字节数
  stats = MyEnumKernelModule("\\\\??\\c:\\windows\\system32\\tessafe.sys",&ModuleAddress,&ModuleSize);
  if (stats == FAILED_TO_OBTAIN_FUNCTION_ADDRESSES)
  {
    return  FAILED_TO_OBTAIN_FUNCTION_ADDRESSES;
  }
  KdPrint(("Address:%0X Sie:%d \n",ModuleAddress,ModuleSize));
  //特征码
  /*  sd1 特征
  p-1:18 p-2:87 p-3:DB p-4:33 p-5:07 
  p-6:03 p  :33 p+1:C0 p+7:3B p+8:D8
    sd2 特征
  p-1:07 p-2:87 p-3:c0 p-4:33 p+14:89
  p+15:1c p+16:38
  */
 
  //将 P 指向内核模块开始处
  p = (BYTE*)ModuleAddress + 20;
  for (i = 0; i < ModuleSize - 20; i++,p++)
  {
    //sd1
    if ((*(p-1) == 0x18) &&
      (*(p-2) == 0x87) &&
      (*(p-3) == 0xDB) &&
      (*(p-4) == 0x33) &&
      (*(p-5) == 0x07) &&
      (*(p-6) == 0x03) &&
      (*p    == 0x33) &&
      (*(p+1) == 0xC0) &&
      (*(p+7) == 0x3B) &&
      (*(p+8) == 0xD8) )
    {
      KdPrint(("--SD1 -- %0X \n",(ULONG)p));
      sd1 = p;
      number+=1;  //记录已经获取一个特征
    }
    //sd2
    if ((*(p-1) == 0x07) &&
      (*(p-2) == 0x87) &&
      (*(p-3) == 0xC0) &&
      (*(p-4) == 0x33) &&
      (*(p+14)== 0x89) &&
      (*(p+15)== 0x1C) &&
      (*(p+16)== 0x38) &&
      (*p    == 0xA1))
    {
      KdPrint(("--SD2 -- %0X \n",(ULONG)p));
      sd2 = p;
      number+=1;  //记录已经获取一个特征
    }
    //pd
    if ((*(p-2) == 0xE3) &&
      (*(p-3) == 0xC1) &&
      (*(p-7) == 0xF3) &&
      (*(p-8) == 0x33) &&
      (*(p-10)== 0xEB) &&
      (*(p-11)== 0xC1) &&
      (*(p+1) == 0xF3) &&
      (*(p+2) == 0x42) &&
      (*(p+3) == 0x3B) &&
      (*(p+4) == 0xD1) &&
      (*p    == 0x33))
    {
      KdPrint(("--PD -- %0X \n",(ULONG)p));
      pd = p;
      number+=1;  //记录已经获取一个特征
    }
    if (number >= 3)
    {
      KdPrint(("特征 %d ---退出\n",number));
      break;
    }
  }
 
  //首先干掉监视函数
  while (1)
  {
    if ((*(pd-1) == 0xcc) && (*(pd-2) == 0xcc))
    {
      KdPrint(("pd 首地址:%0X \n",(ULONG)pd));
      WPOFF();  //清除 CR0
      //提升 IRQL 中断级
      Irql=KeRaiseIrqlToDpcLevel();
      //写入
      RtlCopyMemory(pd,C390,2);
      //恢复 Irql
      KeLowerIrql(Irql);
      WPON();    //恢复 CR0
      break;
    }
    pd--;
  }
  //干掉 2 个 SD
  while (1)
  {
    if ((*(sd1-1) == 0xcc) && (*(sd1-2) == 0xcc))
    {
      KdPrint(("sd1 首地址:%0X \n",(ULONG)sd1));
      WPOFF();  //清除 CR0
      //提升 IRQL 中断级
      Irql=KeRaiseIrqlToDpcLevel();
      //写入
      RtlCopyMemory(sd1,C390,2);
      //恢复 Irql
      KeLowerIrql(Irql);
      WPON();    //恢复 CR0
      break;
    }
    sd1--;
  }
  while (1)
  {
    if ((*(sd2-1) == 0xcc) && (*(sd2-2) == 0xcc))
    {
      KdPrint(("sd2 首地址:%0X \n",(ULONG)sd2));
      WPOFF();  //清除 CR0
      //提升 IRQL 中断级
      Irql=KeRaiseIrqlToDpcLevel();
      //写入
      RtlCopyMemory(sd2,C390,2);
      //恢复 Irql
      KeLowerIrql(Irql);
      WPON();    //恢复 CR0
      break;
    }
    sd2--;
  }
 
  return  STATUS_SUCCESS;
}
```

最后，处理一下硬件断点就可以了。

# SSDT HOOK 实现硬件断点恢复

这里我们使用到了 `SSDT HOOK`。

分别 `HOOK` 了 `SSDT` 表中索引为 `0xD5` 和 `0x55` 的函数。由于这里比较简单，我想 10 个人有 9 个人懂得 `SSDT HOOK` 的。所以直接给出源码，不做原理分析了。

代码:

```c
//处理硬件断点时
ULONG    uNtSetContextThreadAddress;
ULONG    uNtGetContextThreadAddress;
ULONG    TenNtSetContextThread, 
      TenNtGetContextThread;
//////////////////////////////////////////////////////////////////////
//  名称:  _MyNtGetThreadContext
//  功能:  两个 SSDT HOOK 伪造函数的中继函数
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
static NAKED NTSTATUS Nakd_NtGetThreadContext(HANDLE hThread, PCONTEXT pContext)
{
  __asm
  {
    jmp    dword ptr[TenNtGetContextThread]
  }
}
 
static NAKED NTSTATUS Nakd_NtSetThreadContext(HANDLE hThread, PCONTEXT pContext)
{
  __asm
  {
    jmp    dword ptr[TenNtSetContextThread]
  }
}
//////////////////////////////////////////////////////////////////////
//  名称:  MyNtGetThreadContext && MyNtSetThreadContext
//  功能:  NtGetThreadContext 与 NtSetThreadContext 函数被 SSDT HOOK 的伪造函数
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
NTSTATUS MyNtGetThreadContext(HANDLE hThread, PCONTEXT pContext)
{
  if ( _stricmp((const char*)PsGetProcessImageFileName(PsGetCurrentProcess()),DNF_EXE) )
  {
    return Nakd_NtGetThreadContext(hThread, pContext);
  }
  return STATUS_UNSUCCESSFUL;
}
 
 
NTSTATUS MyNtSetThreadContext(HANDLE hThread, PCONTEXT pContext)
{
  if ( _stricmp((const char*)PsGetProcessImageFileName(PsGetCurrentProcess()),DNF_EXE) )
  {
    return Nakd_NtSetThreadContext(hThread, pContext); 
  }
  //DbgPrint("Dr7:%08X\n", pContext->Dr7);
  if ( pContext->Dr7 == 0x101 )
  {
    return Nakd_NtSetThreadContext(hThread, pContext); 
  }
  return STATUS_UNSUCCESSFUL; 
}
 
//////////////////////////////////////////////////////////////////////
//  名称:  My_Recovery_HardwareBreakpoint
//  功能:  通过对 set 与 get 进行 SSDT HOOK 来恢复硬件断点
//  参数:  
//  返回:  
//////////////////////////////////////////////////////////////////////
NTSTATUS My_Recovery_HardwareBreakpoint()
{
  KIRQL    Irql;
  //获取地址
  uNtSetContextThreadAddress = (ULONG)KeServiceDescriptorTable->ServiceTableBase+0xD5 * 4;
  uNtGetContextThreadAddress = (ULONG)KeServiceDescriptorTable->ServiceTableBase+0x55 * 4;
 
  TenNtSetContextThread = *(ULONG*)uNtSetContextThreadAddress;
  TenNtGetContextThread = *(ULONG*)uNtGetContextThreadAddress;
 
  KdPrint(("Set 地址:%0X\n",TenNtSetContextThread));
  KdPrint(("Get 地址:%0X\n",TenNtGetContextThread));
 
  KdPrint(("Process:%0X \n",(ULONG)p_MyHookAddress));
  KdPrint(("Thread:%0X \n",(ULONG)t_MyHookAddress));
 
  WPOFF();  //清除 CR0
  //提升 IRQL 中断级
  Irql=KeRaiseIrqlToDpcLevel();
 
  //完成 SSDT HOOK
  *(ULONG*)uNtGetContextThreadAddress = (ULONG)MyNtGetThreadContext;
  *(ULONG*)uNtSetContextThreadAddress = (ULONG)MyNtSetThreadContext;
 
  //恢复 Irql
  KeLowerIrql(Irql);
  WPON();    //恢复 CR0
 
  return STATUS_UNSUCCESSFUL; 
}
```

另外还有一些功能型的函数一并给出，省的大家迷糊。

我也算服务到位了，再看上面代码迷糊的时候。看这里找找。



看看有没有能用到的，或者翻一下我以往的帖子。里面应该有

代码:

```c
//保存 5 字节代码的结构
#pragma pack(1)
typedef struct _TOP5CODE
{
  UCHAR  instruction;  //指令
  ULONG  address;    //地址
}TOP5CODE,*PTOP5CODE;
#pragma pack( )

//ssdt 表结构
typedef struct _ServiceDescriptorTable {
  PVOID ServiceTableBase;    //System Service Dispatch Table 的基地址   
  PVOID ServiceCounterTable;
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。 
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。   
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表 
}*PServiceDescriptorTable;
//由 SSDT 索引号获取当前函数地址
//NtOpenProcess  [[KeServiceDescriptorTable]+0x7A*4]
extern PServiceDescriptorTable KeServiceDescriptorTable;

//////////////////////////////////////////////////////////////////////
//  名称:  MyGetFunAddress
//  功能:  获取函数地址
//  参数:  函数名称字符串指针
//  返回:  函数地址
//////////////////////////////////////////////////////////////////////
ULONG MyGetFunAddress( IN PCWSTR FunctionName)
{
  UNICODE_STRING UniCodeFunctionName;
  RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );
  return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );    
}

//////////////////////////////////////////////////////////////////////
//  名称:  myGetCurrentAddress
//  功能:  获取 SSDT 表中指定函数的当前地址
//  参数:  index:指定函数在表中的索引号
//  返回:  地址
//////////////////////////////////////////////////////////////////////
ULONG myGetCurrentAddress(IN ULONG index)
{
  ULONG    SSDT_Cur_Addr;
  __asm
  {
    push  ebx
    push  eax
    mov   ebx,KeServiceDescriptorTable
    mov   ebx,[ebx]
    mov   eax,index
    shl   eax,2
    add   ebx,eax
    mov   ebx,[ebx]
    mov   SSDT_Cur_Addr,ebx
    pop   eax
    pop   ebx
  }

  return  SSDT_Cur_Addr;
}

VOID WPOFF()
{
  __asm
  {
    cli
    mov eax,cr0
    and eax,not 10000h
    mov cr0,eax
  }
}

VOID WPON()
{
  __asm
  {
    mov eax,cr0
    or eax,10000h
    mov cr0,eax
    sti
  }
}
```

记在最后面的话：大家要善用搜索引擎(建议学习`google hacking`技巧)，勤做笔记，最后要说的依然是感谢，感谢`GOOGLE\BAIDU\PEDIY\DEBUGMAN`。还有那些默默发帖的人~

如果有时间的话，我会将其他几个游戏保护的分析资料也放出来

什么`GPK\HP\HS`的。大家不要催不要急，一定会放出来的。等到我觉得这些东西都没有挑战性的时候那么也就不会再有资料陆续放出来了……

如果有好东西记得与我分享![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

![](img/b56ec7ac43f57e87ff7d76f3e935a064.jpg)
