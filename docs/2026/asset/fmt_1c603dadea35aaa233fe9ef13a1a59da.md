
```nasm
.text:004069CD loc\_4069CD: ; CODE XREF: \_KiServiceExit+8↑j
.text:004069CD ; \_KiServiceExit+64↓j
.text:004069CD mov ebx, large fs:124h
.text:004069D4 mov [ebx+\_KTHREAD.Alerted], 0
.text:004069D8 cmp [ebx+\_KTHREAD.ApcState.UserApcPending], 0 ; 判断是否有用户APC需要执行
.text:004069DC jz short loc\_406A23
.text:004069DE mov ebx, ebp ; 将陷阱帧栈顶地址赋给ebx
.text:004069E0 mov [ebx+44h], eax ; 为eax赋值
.text:004069E3 mov dword ptr [ebx+50h], 3Bh ; 为fs寄存器赋值
.text:004069EA mov dword ptr [ebx+38h], 23h ; 为ds寄存器赋值
.text:004069F1 mov dword ptr [ebx+34h], 23h ; 为es寄存器赋值
.text:004069F8 mov dword ptr [ebx+30h], 0 ; 为gs寄存器赋值
.text:004069FF mov ecx, 1 ; NewIrql
.text:00406A04 call ds:\_\_imp\_@KfRaiseIrql@4 ; KfRaiseIrql(x)
.text:00406A0A push eax
.text:00406A0B sti
.text:00406A0C push ebx
.text:00406A0D push 0
.text:00406A0F push 1
.text:00406A11 call \_KiDeliverApc@12 ; KiDeliverApc(x,x,x)
.text:00406A16 pop ecx ; NewIrql
.text:00406A17 call ds:\_\_imp\_@KfLowerIrql@4 ; KfLowerIrql(x)
.text:00406A1D mov eax, [ebx+44h]
.text:00406A20 cli
.text:00406A21 jmp short loc\_4069CDKfRaiseIrql@4
```

如果不用执行APC代码，就会接着执行以下的代码，将陷阱帧中的保存的数据复制到线程中，这样就恢复了切换进内核模式前的一部分数据，然后判断是否处于调试模式

```nasm
.text:00406A23 loc\_406A23: ; CODE XREF: \_KiServiceExit+E↑j
.text:00406A23 ; \_KiServiceExit+1F↑j
.text:00406A23 mov edx, [esp+4Ch] ; 将ExceptionList赋给edx
.text:00406A27 mov ebx, large fs:50h ; 将DebugActive赋给ebx
.text:00406A2E mov large fs:0, edx ; 将ExceptionList赋给KPCR的ExceptionList
.text:00406A35 mov ecx, [esp+48h] ; 将PreviousMode赋值给ecx
.text:00406A39 mov esi, large fs:124h
.text:00406A40 mov [esi+\_KTHREAD.PreviousMode], cl
.text:00406A46 test ebx, 0FFh ; 判断是否处于调试模式
.text:00406A4C jnz short loc\_406AC7
```

从陷阱帧中恢复几个寄存器的值

```nasm
.text:00406A89 loc\_406A89: ; CODE XREF: \_KiServiceExit+C5↑j
.text:00406A89 lea esp, [ebp+54h] ; 将陷阱帧中edi的地址赋给esp
.text:00406A8C pop edi ; 恢复edi
.text:00406A8D pop esi ; 恢复esi
.text:00406A8E pop ebx ; 恢复ebx
.text:00406A8F pop ebp ; 恢复ebp
```

此时esp指向陷阱帧中的ErrorCode，然后继续判断保存的cs寄存器的值，如果cs寄存器的值小于0x80就会判断是否是用户模式发起的调用还是内核模式发起的调用，如果是从内核模式发起的调用，则会把返回地址赋值到edx中，然后通过jmp指令跳转

```nasm
.text:00406A90 cmp word ptr [esp+8], 80h ; 判断cs寄存器是否高于0x80
.text:00406A97 ja loc\_407376
.text:00406A9D add esp, 4 ; esp加4，指向EIP
.text:00406AA0 test dword ptr [esp+4], 1 ; 判断保存的cs寄存器是否等于1
.text:00406AA0 \_KiServiceExit endp ; sp-analysis failed
.text:00406AA0
.text:00406AA8
.text:00406AA8 ; =============== S U B R O U T I N E =======================================
.text:00406AA8
.text:00406AA8
.text:00406AA8 \_KiSystemCallExitBranch proc near ; DATA XREF: KiEnableFastSyscallReturn():loc\_43AAE3↓r
.text:00406AA8 ; KiEnableFastSyscallReturn()+26↓w ...
.text:00406AA8 jnz short \_KiSystemCallExit
.text:00406AAA pop edx ; 返回地址赋给edx
.text:00406AAB pop ecx ; cs寄存器赋给ecx
.text:00406AAC popf ; 恢复eflags寄存器
.text:00406AAD jmp edx
```

如果是用户模式发起的调用，则会进行跳转，而跳转的地址会根据是否支持快速调用而不同，如果不支持快速调用，跳转的地址就会是`KiSystemCallExit`，如果支持快速调用，系统就会将跳转地址从`KiSystemCallExit`修改为`KiSystemCallExit2`。

如果执行的是`KiSystemCallExit`，就会通过`iret`指令返回用户层

```nasm
.text:00406AAF \_KiSystemCallExit: ; CODE XREF: \_KiSystemCallExitBranch↑j
.text:00406AAF ; \_KiSystemCallExit2+5↓j
.text:00406AAF ; DATA XREF: ...
.text:00406AAF iret
.text:00406AAF \_KiSystemCallExitBranch
```

如果执行的是`KiSystemCallExit2`，就会将eip和esp分别赋值给edx和ecx以后调用`sysexit`指令来返回到用户层

```nasm
.text:00406AB0 \_KiSystemCallExit2 proc near ; DATA XREF: KiRestoreFastSyscallReturnState()+16↓o
.text:00406AB0
.text:00406AB0 arg\_5 = byte ptr 9
.text:00406AB0
.text:00406AB0 test byte ptr [esp+9], 1
.text:00406AB5 jnz short \_KiSystemCallExit
.text:00406AB7 pop edx ; 将陷阱帧中eip赋值给edx
.text:00406AB8 add esp, 4 ; esp加4，指向陷阱帧中的EFlags
.text:00406ABB and byte ptr [esp+1], 0FDh
.text:00406AC0 popf ; 恢复eflags寄存器
.text:00406AC1 pop ecx ; 将陷阱帧中保存的esp赋给ecx
.text:00406AC2 sti
.text:00406AC3 sysexit
.text:00406AC5 iret
.text:00406AC5 \_KiSystemCallExit2 endp ; sp-analysis failed
```

# 六.参考资料

> - 《Windows内核原理与实现》
> - 《Windows内核情景分析》（上册）
> - 《软件调试》（第二版）卷2

# 一.基本概念

NT内核会把操作系统的代码和数据映射到系统中所有进程的内核空间中。这样，每个进程内的应用程序代码便可以很方便地调用内核空间中的系统服务。这里的”很方便“有多层含义，一方面是内核代码和用户代码在一个地址空间中，应用程序调用系统服务时不用切换地址空间，另一方面是整个系统中内核空间的地址是统一的，编写内核空间的代码时会简单很多。但是，如此设计带来了一个很大的问题，那就是用户空间中的程序指针可以指向内核空间中的数据和代码，因此必须防止用户代码破坏内核空间中的操作系统。做法就是利用权限控制来实现对内核空间的保护。

## 1.访问模式

Windows定义了两种访问模式，分别是用户模式和内核模式。应用程序运行在用户模式下，操作系统运行在内核模式下。内核模式对应于处理器的最高级别权限（ring 0），在内核模式下执行的代码可以访问所有系统资源并具有使用所有特权指令的权利。相对而言，用户模式对应于较低的处理器优先级（ring 3），在用户模式下执行的代码只可以访问系统运行其访问的内存空间且没有使用特权指令的权力。

因为内核模式下的数据和代码具有更高的优先级，所以用户模式下的代码不可以直接访问内核空间中的数据，也不可以直接调用内核空间中的任何函数或例程。任何这样的尝试都会导致保护性错误。也就是说，即使用户空间中的代码指针正确指向了要访问的数据或代码，但一旦访问发生，那么处理器就会检测到该访问是违法的，会停止该访问并产生保护性异常（#GP）。

## 2.系统调用

虽然不能直接访问，但是应用程序可以通过调用系统服务来间接访问内核空间中的数据或间接调用，执行内核空间中的代码。当调用系统服务的时候，主调线程会从用户模式切换到内核模式，调用结束后在返回用户模式，也就是所谓的模式切换。在线程的`KTHEREAD`结构中，定义了`UserTime`和`KernelTime`两个字段，分别用来记录这个线程在用户模式和内核模式的运行事件（以时钟中断计数为单位）。

对于应用程序的运行即进程而言，操作系统内核的作用体现在一组可以供其调用的函数，称为”系统调用“，正是这些系统调用加上一些辅助的手段构成了应用软件的运行环境，即日常所说的”运行平台“。从应用软件的角度看，这些系统调用都是操作系统为其提供的服务，所以也称为”系统服务“。

系统调用所提供的服务是在内核中，一般是在”系统空间“中实现的，而应用软件则都在用户空间运行，二者之间有着空间的间隔。所以，在应用软件和内核之间必定存在一个明确定义的”系统调用界面”。

Windows系统服务界面是隐藏不公开的，公开的是由用户空间一些高层库函数构成的界面，称为“Win32应用程序界面”，即`Win32 API`。用户模式下的代码通过调用这些`Win32 API`来调用系统服务完成需要的操作。

因运行状态和执行的程序所在的内存区间的不同，CPU既可以运行于非特权的“用户态”，或者说运行于“用户空间”，也可也运行于特权的“系统态”，或者说运行于“系统空间”。CPU要从系统空间转入用户空间是容易的，因为运行于系统态的CPU可以通过一些特权指令来改变其运行状态。但是，反过来要从用户空间转入系统空间就不容易了，因为运行于用户态的CPU是不能执行特权指令的。所以，一般而言只有三种手段或原因可以使运行于用户空间的CPU转入系统空间：

- 中断：在开启中断机制的情况下，只要有外部设备的中断请求到来，CPU就会自动转入系统空间，并从（系统空间中）某个预定的地址开始执行指令，从而可以在系统空间对外部设备的中断请求作出反应，或者说提供服务。中断只发生在两条指令之间，因而不会使正在执行的指令半途而废。对于CPU而言，因中断而进入系统空间是被动的，并且CPU是完全无法预知何时将发生中断，所以中断的发生又是“异步”的。
- 异常：不管是用户空间或系统空间，执行指令时的失败都会引起一次”异常“，CPU也会因此而转入系统空间（如果原来不再系统空间），并从某个预定的地址开始执行指令，从而可以在系统空间对发生的失败做出反应。异常在形式上与中断十分相似，只是异常发生在执行一条指令的过程中，而不是两条指令之间，所以当前指令的执行已经半途而废。同样，对于CPU而言，因异常而进入系统空间也是被动的，无法预知的。但是，这只是一般而言，实践中也可也通过故意引起异常而进入内核。例如，在用户空间故意访问系统某个特定地址，就会因访问权限的问题而引起主动的异常，而内核空间中的异常处理程序则可以根据所访问的地址判定其目的，从而做出预定的反应。
- 自陷：为了让CPU能够主动地进入系统空间，绝大多数CPU都设有专门地”自陷“指令，系统调用通常就是靠自陷指令实现地。一执行这样的指令，CPU就转入系统空间并从某个预定地地址开始执行指令，就像掉进了陷阱一样。自陷指令在形式上与中断相似，就像是一次由CPU主动发出的中断请求。对于CPU而言，这是成功执行了一条指令的结果，所以是主动的，尽管该指令其实是由程序员安排的。对于程序而言，则一条自陷指令的作用相当于一次子程序调用，只是这个子程序存在于系统空间中。

传统的Windows系统调用正是通过自陷指令”`int 0x2E`“进入内核实现系统调用的，但是，从Pentium II开始，Intel又在x86系列的CPU中增加了一对指令`sysenter`和`sysexit`，用来实现一种称为“快速系统调用”的机制，并为此增加了三个寄存器。

# 二.Intel x86的用户模式-内核模式的切换

应用程序为了调用系统服务，就需要进行模式切换，将处理器从用户模式切换到内核模式下。模式切换的工作并不需要由应用程序自己来完成，Windows提供了一个系统模块`ntdll.dll`，已经实现了所有系统服务的模式切换工作，在用户模式下只需要通过调用`Win32 API`就可以由`ntdll.dll`实现相应的模式切换工作，切换到内核模式以后就可以使用系统提供的系统服务。模式切换依赖于硬件系统结构，接下来以`ReadProcessMemory`为例，来具体看看切换的过程。

`ReadProcessMemory`是`kernel32.dll`中的一个导出函数，根据反汇编的结果可以看到，该函数的主要行为就是调用`NtReadVirtualMemory`函数来完成功能

|  |  |

| --- | --- |
