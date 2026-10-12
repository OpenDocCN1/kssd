
# 驱动反调试：点 SoftICE 死穴

> 原文：[`www.pediy.com/kssd/pediy11/104908.html`](https://www.pediy.com/kssd/pediy11/104908.html)

## 1. 前言

为表达对看雪论坛的支持，特把这两天研究驱动的一点心得变成文字，以作留念。实事求是地讲，这是我写得第一个驱动，希望大家喜欢。

作为一个驱动菜鸟，首先恶补了一些驱动理论，像《Windows 驱动开发技术详解》、《软件调试》、《从汇编语言到 windows 内核编程》等电子书网上都能下到。看书就要看好书，我可不想一上来就被误导，当然书上的东西我们还是不能全信。

## 2. 开发环境与工具

其次初步熟悉了驱动开发环境和各种调试工具的使用，`windbg`、`softice` 我是都装电脑里了，就差熟练使用了。编译驱动，我是一键编译（`ddk`），当然只有懒人才想得出来。在论坛上我写过一篇相关文章，如果感兴趣的话，可以找出来看看。用 `vc++6.0` 编译也很简单，照《Windows 驱动开发技术详解》一书里设置一下编译、调试选项即可，根本不需要装什么乱七八糟的软件。再用个能显示行号的记事本编辑源文件就 `ok` 了，有行号便于由编译出错信息中提示的行号定位源文件中出错的地方。用 `windbg` 的 `open crash dump` 菜单看看蓝屏后的系统生成的 `*.dmp` 文件也可以快速找到错误（默认不保存 `*.dmp` 文件，当然你要在 `windows` 里先设置一下）。

![](img/cfcc5b78777c2c248caf569aa74fc91f.jpg)

## 3. SoftICE 的反调试原理

废话这么多，下面讲点 `softice` 死穴。在调试驱动过程中，我发现了 `softice` 的一个敏感穴位，在这里我就不保密了。`Softice` 对键盘中断进行了 `hook`，见图 `hook,jpg`，先 `push` 中断服务的真正入口，然后 `jmp` 到 `softice` 的处理代码。在处理的开始先读取 `60` 端口的 `scan code` 而后保存到某处。这里就是关键，如果这里不保存，那么将不会回显键盘输入的字符，其效果就是键盘无法输入。这样就可以被我们用来反调试，当我们的内核代码检测到 `softice` 后就把该处替换成垃圾指令，那么键盘就不起作用了，当然就无法继续调试。原理很简单，具体实现如下：

![](img/5c326a7168716e1229ad7980fa8263b5.jpg)

```c
#include "ntddk.h"

#include <stdio.h>

#define MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16))

int  kb_int = 0;

#pragma pack(1)

typedef struct
{
  unsigned short LowOffset;
  unsigned short selector;
  unsigned char unused_lo;
  unsigned char segment_type:4;    
  unsigned char system_segment_flag:1;
  unsigned char DPL:2;    
  unsigned char P:1;  
  unsigned short HiOffset;
} IDTENTRY;

typedef struct
{
  unsigned short IDTLimit;
  unsigned short LowIDTbase;
  unsigned short HiIDTbase;
} IDTINFO;

typedef struct
{
  unsigned char  E9_BYTE;
  unsigned long  JMP_ADDR;
} MYPATCH;

#pragma pack()

unsigned long ISR_pointer,tt;   

unsigned char ss,Sice_Protected;
MYPATCH*   k;

int irq1_svcnumber()
{
  unsigned char *pIoRegSel;
  unsigned char *pIoWin;
  unsigned char ch;
  PHYSICAL_ADDRESS  phys;
  PVOID        pAddr;
  phys.u.LowPart = 0xFEC00000;   
  pAddr = MmMapIoSpace(phys, 0x14, MmNonCached);//读 0x14 字节即可,最后一个 dword 就是 I/O WINDOW REGISTER
  if (pAddr == NULL)
    return 0;
  
  pIoRegSel = (unsigned char *)pAddr;
  pIoWin = (unsigned char *)(pAddr) + 0x10;
  
  *pIoRegSel = 0x12;    
  ch = *pIoWin;
  MmUnmapIoSpace(pAddr, 0x14);
  
  return (int)ch;
}

/*
The PIIX3 decodes the 82093AA(IOAPIC) in memory space.
Memory Address: FEC0xy00h (xy=See APICBASE Register in the PIIX3)
Typically, IOREGSEL Register is at 00h and IOWIN Register is at 10h.
interl 82093AA has 24 I/O Redirection Table entry registers.  IOREDTBL[0]-IOREDTBL[23]
Each register is a dedicated entry for each interrupt input.
IOREDTBL[1] is the entry for keyboard interrupt (IRQ1), which address offset is 12-13h.  
Here IOWIN Register is used to read vector from the IOAPIC register(IOREDTBL[1])
specified by 0x12 which was written to the IOREGSEL Register.
Redirection table entries 的低 8 位就是 vector
*/

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{           
   if (Sice_Protected==1) {   
       __asm  cli
       k->E9_BYTE=ss;
       k->JMP_ADDR=tt;
       __asm sti
      DbgPrint("Anti_Softice end.");    
   };
      
      DbgPrint("Welcome! by 天易 love 2010-1-10");   
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
  IDTINFO    idt_info;     
  IDTENTRY*  idt_entries;    
  unsigned long   addr;
  unsigned long* addr1;   
  unsigned char*  ch;
  Sice_Protected=0;
  theDriverObject->DriverUnload  = OnUnload; 
  kb_int = irq1_svcnumber();
  DbgPrint("kb_int = 0x%02X\n", kb_int);
  
  __asm  sidt  idt_info
  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);
  ISR_pointer = MAKELONG(idt_entries[kb_int].LowOffset,idt_entries[kb_int].HiOffset);
    ch=(unsigned  char*)(ISR_pointer);  //68 表示 push 指令
    if (*ch!=0x68) {                   //判断是否运行 softice 有无 hook
      DbgPrint("There is no Softice.");
      return STATUS_SUCCESS;
    }
      addr1=(unsigned  long*)(ISR_pointer+6);
      __asm{                  //定位 patch 的地方  
      cli
      mov  eax,addr1  
      mov  eax,[eax]  
      add  eax,addr1
      add  eax,0x3d   
      mov  addr,eax  //     
      };
      k = (MYPATCH*) addr;
      ss=k->E9_BYTE;      //unload 时需要
      tt=k->JMP_ADDR;
    k->E9_BYTE=0x83;         //83C0 00         ADD EAX,0  填上垃圾指令
    k->JMP_ADDR=0xc08b00c0; //8BC0           MOV EAX,EAX
       __asm sti
    Sice_Protected=1;
  DbgPrint("0x%08X is the address patched  within sice",addr);
    DbgPrint("Anti_Softice start....");
  return STATUS_SUCCESS;
}
```

![](img/2e8571637be0c623866ff1b91d21c695.jpg)

代码在灵想技术 `GHOSTXPSP3` 电脑城装机版 `V7.0` `Build` `20090820` 下测试通过，截图留念。

![](img/e5ab0fc831acd62d798783ea33dc64bb.jpg)

## 4. 结语

代码有些地方你不一定能看懂，必须要通过调试才有感性认识，我就不详细注释了。注意中断服务号即 `vector` 的取得也不是很简单，我参考了资料一并奉上。

最后感谢大家！

<div align="right">
By 天易 love  2010-1-10
</div>

## 5. 附件

上传的附件

| 图片 | 文件 |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `antisice.rar` |
