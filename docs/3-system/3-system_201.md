

# 中断和异常

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-235.htm`](https://www.pediy.com/kssd/pediy09/pediy09-235.htm)

> 本文是 Matt Pietrek 在 1997 年月 10 月的 MSJ 杂志 Under The Hood 专栏上发表的文章。中断和异常在 DOS 时代是整个系统的灵魂，但 Windows 已将其隐藏到了系统深处。Matt Pietrek 详细剖析了 Windows 下的中断和异常及其处理机制以及内核模式与用户模式代码之间调用的问题。作者还提供了一个比较有意思的实验程序。  
>   
> 注：附件中是本文的 PDF 格式，可读性好：[`bbs.pediy.com/showthread.php?t=49871`](http://bbs.pediy.com/showthread.php?t=49871)   
>   
> Win32 下的中断和异常  
> 作者：Matt Pietrek  
> 你可能感觉一切都好。但当你写了一些新代码并运行它时才知道你被感觉欺骗了！又出现了令人烦恼的访问违规（Access Violation）。你可能也看到了那个令人害怕的代码 0xC0000005，也就是 STATUS\_ACCESS\_VIOLATION。0xC0000005 是如何表示“刚刚出错了”的，以及 Win32侨绾沃С植煌嘈偷囊斐５模庑┎⒉晃芏嗳怂Ｔ诒驹碌淖ɡ钢校乙诰騑 in32 下的异常以及它们是如何与硬件异常相关联的。在讨论硬件时主要针对的是 Intel x86 平台。  
> 如果你曾经为 Windows?nbsp;3.x 编写过程序或者编写过 MS-DOS?nbsp;extender，你一定遇到过 0xD 这个异常（一般保护性错误，简称为 GPF）。你也可能看到过其它错误，例如非法指令错误（异常 6）。这些代码并不是人工赋予的。任何 Intel 手册上都说过，这些异常代码是 CPU 用来通知各种问题或事件用的。在 Win32 中你看不到这些代码，因为 Windows?nbsp;NT，这个 Win32 操作系统家族的旗舰产品，被设计运行于多种平台上。它没有简单地让 Alpha 或 MIPS 版本的 Windows NT 使用 Intel CPU 的异常代码。  
> 相反，Win32 使用它自己的一套代码系统来表示各种异常。在任何给定的 Win32 平台上，系统把相应的 CPU 的异常代码映射到一个或多个通用的 Win32 异常代码上。例如，Intel CPU 上的异常代码 0xD 可能变成 STATUS\_ACCESS\_VIOLATION（0xC0000005）。同样，异常代码 0xD 也可能变成 Win32 的 STATUS\_PRIVILEGED\_INSTRUCTION（0xC0000096）异常。底层的硬件异常决定了它应该被映射到哪个 Win32 异常上。  
> 让我们从 CPU 异常和中断出发，开始我们的 Win32 异常之旅。异常（Exception）和中断（Interrupt）是一种手段, 当正在执行代码时 CPU 通过它切换到一个完全不同的代码路径上以处理一些外部的刺激或条件。中断通常是由外部的刺激引起的，例如按下了一个键。而异常则是代码或数据中的条件导致处理器生成的。CPU 试图读取一个没有物理内存映射到的地址时会产生异常，这是最经典的一个异常的例子。  
> Intel CPU 保留了 32 个中断/异常号以处理各种情形。图 1 是一些常用的代码。它们中很多意义很清楚，但是还有很多你没有遇到过（至少是在运行本专栏的样例程序之前）。MS-DOS 上的老手可能奇怪竟然列出的 INT 5H 不是打印屏幕，INT 8H 也不是计时器中断。这是为什么？图 1 的描述是 Intel 对异常和中断的定义。但不幸的是，在 Intel 迅猛发展之前，MS-DOS 的作者已经把其中的一些中断号用作其它用途。结果导致当程序员使用 BOUND 指令时竟然意外到得到了屏幕的输出内容！  
> 图 1：Intel 定义的异常和中断  
> 代码  定义  
> 00  除法错  
> 01  调试异常（单步和硬件调试）  
> 02  不可屏敝中断（NMI）  
> 03  断点中断  
> 04  溢出中断（INTO）  
> 05  越界中断  
> 06  非法指令  
> 07  协处理器不可用  
> 08  异常嵌套  
> 0A  非法任务状态段(TSS)  
> 0B  段不存在  
> 0C  堆栈错误  
> 0D  一般保护性错  
> 0E  页错误  
>   
> 为了简单起见，本专栏以下的部分中我就用异常来代表异常或中断。就像我前面说的，中断和异常在技术上是不同的。另外，异常可以被进一步分成故障（Fault）、自陷（Trap）和终止（Abort）。我不想在这里对它们做详细描述，你可以简单地认为它们是一样的。  
>   
> 当异常发生时，CPU 挂起当前的执行路径，把控制权交给异常处理程序。CPU 把标志寄存器（EFLAGS）、代码段寄存器（CS）、指令指针寄存器（EIP）压入堆栈以保护当前的执行状态。接着，根据异常代码查找事先设计好的处理这个异常的程序的地址，并把控制权转交给它。实际上，异常代码就是中断描述符表（Interrupt Descriptor Talbe，IDT）的索引，而中断描述符表指出异常应该交由谁处理。  
>   
> IDT 是 Intel CPU 使用的基本数据结构，它由多达 256 个中断描述符组成，每个长为 8 字节。中断描述符表由操作系统创建和维护，因此虽然被理解为是 CPU 的数据结构，但它也受到操作系统的控制。如果操作系统把 IDT 搞错了，那整个系统立马崩溃。  
>   
> 在大多数操作系统上，包括基于 Win32 的系统，IDT 被放在高特权级内存上，低特权级的应用程序根本不能访问它。这与实模式的 MS-DOS 程序有很大不同，在那里，应用程序通常替换中断向量表（IDT 在实模式下的一种版本）。由于多个基于 MS-DOS 的程序、驱动程序、TSR（终止并驻留程序）缺乏协调，导致 MS-DOS 系统和 16 位的 Windows 系统特别不稳定。在最新的 32 位操作系统上，CPU 严格限制对 IDT 的访问，相应地增加了稳定性。然而 Win32 设备驱动程序（高特权级）可以访问 IDT，并且可以修改它在 IDT 中的相应项。  
>   
> 现在让我们回到异常发生时的情形。CPU 把异常号作为索引获取 8 字节的描述符。在描述符中包括各种域。图 2 显示的是中断描述符的一种简化形式。注意，对于每个异常来说，都有一个相应的异常处理程序地址（CS:EIP），控制权就是要转到这个地址。图 3 显示了 GPF（异常 0xD）发生时的事件顺序。  
> 图 2：中断描述符  
>     
> 图 3：异常发生时的事件顺序  
>     
> 要是在平时，到这里我一定会写一个能显示 IDT 内容的试验程序。但不幸的是（至少对于我来说），应用程序不能访问 IDT。这是因为在 Win32 下，应用程序运行在 Ring 3，这是最低的特权级。Win32 操作系统内核运行在 Ring 0（内核或管理模式），这是最高的特权级。同时，关键的操作系统数据结构，例如 IDT，只能通过 Ring 0 的代码进行访问。（Ring 1 和 2 在 Win32 中没有使用。从 80286 开始起它们就存在，但据我所知，还没有人使用这些特权级。）  
> 既然我不能写一个可以读取 IDT 的程序，那就拿一些其它资料吧。图 4 是用 SoftICE/NT 的 IDT 命令得到的前 30 个中断描述符表项。SoftICE 作为 Ring 0 下的驱动程序运行，所以它对 IDT 有读/写权。  
> 图 4：SoftICE 的 IDT 命令输出结果  
> Int  Type  Sel:Offset  Attributes  Symbol/Owner  
> IDTbase=80036400 Limit=07FF  
> 0000  IntG32  0008:8013C354  DPL=0  P  \_KiTrap00  
> 0001  IntG32  0008:8013C49C  DPL=3  P  \_KiTrap01  
> 0002  IntG32  0008:0000137E  DPL=0  P    
> 0003  IntG32  0008:8013C764  DPL=3  P  \_KiTrap03  
> 0004  IntG32  0008:8013C8B8  DPL=3  P  \_KiTrap04  
> 0005  IntG32  0008:8013C9F4  DPL=0  P  \_KiTrap05  
> 0006  IntG32  0008:8013CB4C  DPL=0  P  \_KiTrap06  
> 0007  IntG32  0008:8013D068  DPL=0  P  \_KiTrap07  
> 0008  TaskG  0050:000013D8  DPL=0  P    
> 0009  IntG32  0008:8013D3A8  DPL=0  P  \_KiTrap09  
> 000A  IntG32  0008:8013D4A8  DPL=0  P  \_KiTrap0A  
> 000B  IntG32  0008:8013D5CC  DPL=0  P  \_KiTrap0B  
> 000C  IntG32  0008:8013D8BC  DPL=0  P  \_KiTrap0C  
> 000D  IntG32  0008:8013DABC  DPL=0  P  \_KiTrap0D  
> 000E  IntG32  0008:8013E468  DPL=0  P  \_KiTrap0E  
> 000F  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0010  IntG32  0008:8013E8D4  DPL=0  P  \_KiTrap10  
> 0011  IntG32  0008:8013E9E8  DPL=0  P  \_KiTrap11  
> 0012  TaskG  00A0:8013E7D4  DPL=0  P    
> 0013  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0014  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0015  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0016  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0017  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0018  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0019  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001A  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001B  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001C  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001D  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001E  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 001F  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
> 0020  Reserved  0008:00000000  DPL=0  NP    
> 0021  TrapG16  00C7:00000696  DPL=3  P    
> 0022  Reserved  0008:00000000  DPL=0  NP    
> 0023  Reserved  0008:00000000  DPL=0  NP    
> 0024  Reserved  0008:00000000  DPL=0  NP    
> 0025  Reserved  0008:00000000  DPL=0  NP    
> 0026  Reserved  0008:00000000  DPL=0  NP    
> 0027  Reserved  0008:00000000  DPL=0  NP    
> 0028  Reserved  0008:00000000  DPL=0  NP    
> 0029  Reserved  0008:00000000  DPL=0  NP    
> 002A  IntG32  0008:8013B8A6  DPL=3  P  \_KiGetTickCount  
> 002B  IntG32  0008:8013B990  DPL=3  P  \_KiCallbackReturn  
> 002C  IntG32  0008:8013BAA0  DPL=3  P  \_KiSetLowWaitHighThread  
> 002D  IntG32  0008:8013C65C  DPL=3  P  \_KiDebugService  
> 002E  IntG32  0008:8013B440  DPL=3  P  \_KiSystemService  
> 002F  IntG32  0008:8013E7D4  DPL=0  P  \_KiTrap0F  
>   
> 首先看到的是，Windows NT 的 IDT 中所有的异常处理程序地址都在 0x80000000 之上。0x80000000 之上的地址被 Windows NT 保留用于特权级（Ring 0）访问。尽管从图上看可能不明显，但是确实几乎所有的异常处理程序地址都在 NTOSKRNL.EXE 中，它是 Windows NT 中运行于 Ring 0 的核心组件。由于我事先已经从 NTOSKRNL 的 DBG 文件中加载了调试符号，所以 SoftICE 查找异常处理程序地址并且找到了大部分异常处理程序的名称。前 0x20 个异常被一系列名字为\_KiTrap00，\_KiTrap01 等的例程处理。“Ki”代表内核中断（Kernel Interrupt）。  
>   
> 还有一个应该注意的是 IDT 中的描述符特权级（Descriptor Privilege Level，DPL）域。它指定了允许调用特定软件中断的最低特权级。例如，INT 2EH 可以被从 Ring 3（最低特权级）到 Ring 0（最高特权级）中任何一级调用。同样，用于断点的 INT 3H，也可以被 Ring 3 及更高特权级的代码调用。  
>   
> 从 0x2A 到 0x2E 的异常被 NTOSKRNL.EXE 中的其它例程处理。例如，在我 1996 年八月的文章“Poking Around Under the Hood: A Programmer’s View of Windows NT 4.0”，我讲到了 Ring 3 级的应用程序代码传递控制权到 Ring 0 级的系统代码以完成诸如创建一个新进程之类的特殊操作的机制，那就是调用 INT 2E。INT 2E 被系统 DLL，例如 NTDLL.DLL、USER32.DLL 和 GDI32.DLL 从 Ring 3 调用。看一下 IDT 的 0x2E 这一项，你会看到它的地址指向 NTOSKRNL 中的\_KiSystemService 函数。正是这个函数把控制权转到了相应的代码。  
>   
> INT 2EH 之后，在前面的表中接下来最经常使用使用的中断当属 INT 2BH。这个中断在 IDT 中的项的名称叫\_KiCallbackReturn，这个名字提示了它的作用。当 Ring 3 的回调函数被 Ring 0 的代码调用后，需要一种回到 Ring 0 的调用者中的方法。INT 2BH 正用于此目的。这方面的一个典型例子是调用 SetWindowsHookEx 来安装的 Windows 钩子回调函数。用户功能中的真正实现部分在 Ring 0 的 WIN32K.SYS 驱动程序中，正是它调用了在 Ring 3 中的钩子回调函数。当回调函数执行完毕，系统执行一个 INT 2BH 返回到 Ring 0。  
>   
> 关于中断讲的已经够多了。那异常怎么样呢，特别是像访问违规之类令人讨厌的异常？处理器级别最经常出现的两个异常是异常 0xD（GPF）和 0xE（页错误）。从 CPU 产生这些异常到你的应用程序得到机会处理它们这段时间内，操作系统把异常代码改成它喜欢的更一般的代码。  
>   
> 假设你想运行下面这个有错误的程序，它试图把 2 写到内存偏移 0 处：  
> int main()  
> {  
>     \*(int \*) 0 = 2;  
> }  
> 正如你所料，偏移 0 不是一个可用的程序地址。例如，在 Windows NT 中，内存的第一个 4KB 页面被标记为“不存在”，用以阻止使用 NULL 指针的程序问题。试图写这个地址将引发一个页错误（异常 0xE）。看一下上面的 IDT 图，你会看到这个异常是由 NTOSKRNL.EXE 中的\_KiTrap0E 处理的。  
>   
> 我已经多次在调试器中跟踪到\_KiTrap0E 的代码中，但这个代码相当复杂，想全面描述得另用一篇文章才行。眼下，只要知道 Ring 0 的\_KiTrap0E 代码检查各种各样的特殊条件就足够了。因此，KiTrap0E 调用了 IRETD 指令把控制权传到了 Ring 3 的 NTDLL 开头的 KiUserExceptionDispatcher 函数中。我在这里不讲 KiUserExceptionDispatcher，因为我已经在我的文章“A Crash Course on the Depths of Win32 Structured Exception Handling”（MSJ，1997 年一月）中详细讲了这个函数。重点是要知道 KiUserExceptionDispatcher 被告知异常代码是 0xC0000005（STATUS\_ACCESS\_VIOLATION），并不是由 CPU 产生的那个异常代码 0xE。  
>   
> 像 0xC0000005 之类的 Win32 异常代码是哪里来的？答案可以在 Win32 SDK 或你的 C++编译器中的 WINERROR.H 头文件中找到。几乎在最上面，你会看到一个注释：  
> //  Values are 32 bit values layed out as follows:  
> 继续读这个注释，你就会知道，最高的两位（位 31 和 30）代表严重程度。接下来的位（29）表示定义者。位 28 是保留的。高位字中剩下的 12 位是设备代码。低位字（位 0 到 15）是异常代码。  
>   
> 比较有趣的一点是，Win32 的 Last Error 代码也是通过用位域来分类信息的。因此，你会知道像 0x80010002（RPC\_E\_CALL\_CANCELED）之类的错误代码来自哪里。顺便说一下，使用严重程度，定义者和设备位域并不是起源于 Windows NT。IBM 的 OS/2 使用了相同的机制，它是在 20 世纪 80 年代后期合并分别由 Microsoft 和 IBM 完成的操作系统的工作的一个副产品。  
>   
> 回到异常中，看一下严重程度位，位 31 和 30。值 0 代表成功，1 代表信息，2 代表警告，3（两个位均置位）代表错误。一个致命的异常相当于一个错误，因此任何 32 位的致命异常代码最高的两位都是置位的。接下来的两个位，定义者和保留位，通常都被设置为 0，因为很少使用它们。  
>   
> 仅仅知道上面那些有限的异常代码构造方面的知识，你就能推断出致命异常代码都是以 0xC 开头的。因此，遇到像 0xC0000005（STATUS\_ACCESS\_VIOLATION）和 0xC000001D（STATUS\_ILLEGAL\_INSTRUCTION）之类的异常代码，你知道它们就属于这一类。比这严重程序低一些的异常，也就是警告，它的严重系数是 2，因此你看到类似 0x80000003（STATUS\_BREAKPOINT）和 0x80000004（STATUS\_SINGLE\_STEP）之类的代码，你知道它们就属于这一类。在 WINNT.H 中搜索 STATUS\_ 可以找到一份相当完整的可能的异常代码列表。当你看这个列表时要记住，并不是支持 Win32 的每一个处理器都可以生成所有 Win32 异常代码。  
>   
> 在写这个专栏时，我到底能导致多少个 Win32 异常引起了我的兴趣。我对操作系统到底能赋予我有意导致的许多错误什么样的异常代码也充满好奇。为了帮助解决这些问题，我写了一个能以各种方式产生处理器错误并且报告它们被映射到的 Win32 异常代码的程序框架。这就是我的 GenException 程序（见图 5）。  
> 图 5 GenException.CPP  
> //==========================================  
> // Matt Pietrek  
> // Microsoft Systems Journal, October 1997  
> // FILE: GenException.CPP  
> // 使用命令行 CL GenException.CPP 编译  
> //==========================================  
> #define WIN32\_LEAN\_AND\_MEAN  
> #include <windows.h>  
> #include <stdio.h>  
> #include <float.h>  
> #include <assert.h>  
>   
> typedef void (\* PFNGENERATEEXCEPTION)(void);  
> void GenerateSTATUS\_BREAKPOINT( void )  
> {  
>     \_\_asm   int 3   // 普通的断点指令  
> }  
> void GenerateSTATUS\_SINGLE\_STEP( void )  
> {  
>     // 这比使用硬件断点寄存器生成 int 1 更容易  
>     \_\_asm   int 1  
> }  
> void GenerateSTATUS\_ACCESS\_VIOLATION( void )  
> {  
> // 通过读取地址在 2GB 以上的内存来产生  
> // 一个页错误（异常代码 0xE）  
>     int i = \*(int \*)0xFFFFFFF0;  
> }  
> void GenerateSTATUS\_ILLEGAL\_INSTRUCTION( void )  
> {  
>     \_\_asm \_emit 0x0F    // 无效指令导致产生异常 0xD  
>     \_\_asm \_emit 0xFF  
> }  
> void GenerateSTATUS\_ARRAY\_BOUNDS\_EXCEEDED( void )  
> {  
>     DWORD arrayBounds\[2\] = { 10, 48 };  
>   
>     \_\_asm   mov eax, 12  
>     \_\_asm   bound eax, arrayBounds  // 这条 BOUND 指令运行正常  
>     \_\_asm   mov eax, 7  
>     \_\_asm   bound eax, arrayBounds  // 这条 BOUND 指令会产生异常 0x5  
> }  
> void UnmaskFPExceptionBits( void )  
> {  
>     unsigned short cw;  
>   
>     \_\_asm   fninit      // 初始化数值协处理器  
>     \_\_asm   fstcw \[cw\]  
>     cw &= 0xFFE0;       // 关闭大部分异常位（除了精度异常）  
>     \_\_asm   fldcw \[cw\]  
>   
> }  
> void GenerateSTATUS\_FLOAT\_DIVIDE\_BY\_ZERO( void )  
> {  
>     double a = 0;  
>       
>     a = 1 / a;  
>     \_\_asm fwait;          
> }  
> void GenerateSTATUS\_FLOAT\_OVERFLOW( void )  
> {  
>     double a = DBL\_MAX;  
>   
>     a \*= a;  
>     \_\_asm fwait;  
>           
> }  
> void GenerateSTATUS\_FLOAT\_STACK\_CHECK( void )  
> {  
>     unsigned a;  
>   
>     \_\_asm   fistp \[a\]  
>     \_\_asm   fwait;  
>           
> }  
> void GenerateSTATUS\_FLOAT\_UNDERFLOW( void )  
> {  
>     double a = DBL\_MIN;  
>       
>     a /= 10;  
>     \_\_asm fwait;  
>           
> }  
> void GenerateSTATUS\_INTEGER\_DIVIDE\_BY\_ZERO( void )  
> {  
>     // 除以 0 导致异常 0x0  
>     int i = 0;  
>     i = 2 / i;  
> }  
> void GenerateSTATUS\_INTEGER\_OVERFLOW( void )  
> {  
>     \_\_asm   mov eax, 07FFFFFFFh     // 带符号数的最大值  
>     \_\_asm   add eax, 2              // 结果 = 0x80000001 -> 溢出！  
>     \_\_asm   into                    // 产生异常 0x4  
> }  
> void GenerateSTATUS\_PRIVILEGED\_INSTRUCTION( void )  
> {  
>     // HLT 指令只能在 ring 0 下执行  
>     \_\_asm   hlt  
> }  
> void GenerateSTATUS\_STACK\_OVERFLOW( void )  
> {  
>     DWORD myArray\[512\];  
>       
>     // “无穷”递归导致堆栈溢出  
>     GenerateSTATUS\_STACK\_OVERFLOW();  
> }  
> DWORD GetExceptionNumber( PFNGENERATEEXCEPTION pfn )  
> {  
>     DWORD exceptionCode = 0;  
>     \_\_try  
>     {  
>         pfn();    
>     }  
>     \_\_except( exceptionCode = GetExceptionCode(), EXCEPTION\_EXECUTE\_HANDLER )  
>     {  
>     }     
>     return exceptionCode;  
> }  
> #define SHOW\_EXCEPTION( x )                                 \\  
>     dwExceptionNumber = GetExceptionNumber( Generate##x );  \\  
>     printf( "%X %s\\n", dwExceptionNumber, #x );             \\  
>     assert( dwExceptionNumber == x );  
> int main(int argc, char \*argv\[\])  
> {  
>     DWORD dwExceptionNumber;  
>       
>     SHOW\_EXCEPTION( STATUS\_BREAKPOINT )  
>     SHOW\_EXCEPTION( STATUS\_SINGLE\_STEP )  
>     SHOW\_EXCEPTION( STATUS\_ACCESS\_VIOLATION )  
>     SHOW\_EXCEPTION( STATUS\_ILLEGAL\_INSTRUCTION )  
>     SHOW\_EXCEPTION( STATUS\_ARRAY\_BOUNDS\_EXCEEDED )  
>       
>     UnmaskFPExceptionBits();  
>     SHOW\_EXCEPTION( STATUS\_FLOAT\_DIVIDE\_BY\_ZERO )  
>   
>     UnmaskFPExceptionBits();  
>     SHOW\_EXCEPTION( STATUS\_FLOAT\_OVERFLOW )  
>   
>     UnmaskFPExceptionBits();  
>     SHOW\_EXCEPTION( STATUS\_FLOAT\_STACK\_CHECK )  
>   
>     UnmaskFPExceptionBits();  
>     SHOW\_EXCEPTION( STATUS\_FLOAT\_UNDERFLOW )  
>   
>     SHOW\_EXCEPTION( STATUS\_INTEGER\_DIVIDE\_BY\_ZERO )   
>     SHOW\_EXCEPTION( STATUS\_INTEGER\_OVERFLOW )  
>     SHOW\_EXCEPTION( STATUS\_PRIVILEGED\_INSTRUCTION )  
>   
>     SHOW\_EXCEPTION( STATUS\_STACK\_OVERFLOW );  
>       
>     return 0;  
> }  
>   
> GetException 程序的代码被分成三部分。第一部分是一系列函数，它们的名字以 Generate 开头，后面是它们要产生的 Win32 异常的名字。例如，GenerateSTATUS\_ILLEGAL\_INSTRUCTION 引起一个非法指令异常。第二部分是 GetExceptionNumber 函数。它使用 Win32 结构化异常处理（SEH）来确定各个 GenerateXXX 函数引起的 Win32 异常代码，并且将这个异常代码返回它的调用者。GetExceptionNumber 函数带有一个参数，这个参数是指向它要调用的 GenerateXXX 函数的指针。  
>   
> GenException.CPP 的最后一部分是 main 函数。它是一系列 C++预处理器宏的调用，这个宏被我命名为 SHOW\_EXCEPTION。对 SHOW\_EXCEPTION 的每一次调用就会产生一个 Win32 异常。SHOW\_EXCEPTION 带一个预定义的异常名称（例如 STATUS\_ACCESS\_VIOLATION），然后将它合成一个与其相应的 GenerateXXX 函数的调用。我使用 SHOW\_EXCEPTION 宏来省略大量模板代码，这些模块代码只有实际调用的异常代码不同。通过使用预处理器符号粘贴（preprocessor token pasting）和字符串化（stringizing）宏，这一行  
>   
> SHOW\_EXCEPTION( STATUS\_BREAKPOINT )  
> 被扩展成：  
> dwExceptionNumber = GetExceptionNumber( GenerateSTATUS\_BREAKPOINT );  
> printf( "%X %s\\n", dwExceptionNumber, "STATUS\_BREAKPOINT" );  
> assert( dwExceptionNumber == STATUS\_BREAKPOINT );  
>   
> 在写 GetException 时，一些异常非常容易产生，例如 STATUS\_ACCESS\_VIOLATION。创建那些不常见的异常也很重要，例如 STATUS\_ILLEGAL\_INSTRUCTION。许多情况下，我不得不借助于内联汇编。两个比较好的例子是 CPU 异常 4 和 5，它们分别由 INTO 指令和 BOUND 指令产生。我不详细讲述各种异常是如何产生的，GenException.CPP 代码中包含了许多相关注释。  
>   
> 生成浮点异常需要一些技巧，因为 Win32 初始化浮点单元时不会产生异常。我不得不明确关闭协处理器控制字中的某些位来产生浮点异常，像 STATUS\_FLOAT\_DIVIDE\_BY\_ZERO。如果你对此好奇，可以看 UnmaskFPExceptionBits 函数，它包含了处理那些位的代码。因为在执行浮点指令时，只有执行到实际出错指令的下一条指令时才引发异常，因此我使用\_\_asm fwait 指令强制在一个有意出错的指令后引发一个异常。  
>   
> 可能 GetException 不是你曾经运行过的程序中最令人兴奋或最有用的程序，但是我相信你一定能从如何产生各种 Win32 异常中受到启发。在大多数情况下，CPU 生成一个异常 0xD，然后 Win32 异常处理程序分析这个代码并构造一个更有意义，更加明确的异常代码。我的目的是描述这些机制，解释硬件级别和操作系统级别的异常，并且向你展示它们之间的联系。  
>   
> [（Microsoft System Journal 1997 年 10 月 Under The Hood 专栏）](http://www.microsoft.com/msj/defaultframe.asp?page=/msj/1097/hood1097.htm)  
> 译者：SmartTech   电子信箱：zhzhtst@163.com 

 4dd466d0e1e0cc123afbe504da2462ca