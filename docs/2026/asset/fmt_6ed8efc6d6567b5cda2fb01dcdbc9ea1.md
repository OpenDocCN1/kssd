
# [原创]Windows内核学习笔记之系统调用

## 一.基本概念

NT内核会把操作系统的代码和数据映射到系统中所有进程的内核空间中。这样，每个进程内的应用程序代码便可以很方便地调用内核空间中的系统服务。这里的”很方便“有多层含义，一方面是内核代码和用户代码在一个地址空间中，应用程序调用系统服务时不用切换地址空间，另一方面是整个系统中内核空间的地址是统一的，编写内核空间的代码时会简单很多。但是，如此设计带来了一个很大的问题，那就是用户空间中的程序指针可以指向内核空间中的数据和代码，因此必须防止用户代码破坏内核空间中的操作系统。做法就是利用权限控制来实现对内核空间的保护。

### 1.访问模式

Windows定义了两种访问模式，分别是用户模式和内核模式。应用程序运行在用户模式下，操作系统运行在内核模式下。内核模式对应于处理器的最高级别权限（ring 0），在内核模式下执行的代码可以访问所有系统资源并具有使用所有特权指令的权利。相对而言，用户模式对应于较低的处理器优先级（ring 3），在用户模式下执行的代码只可以访问系统运行其访问的内存空间且没有使用特权指令的权力。

因为内核模式下的数据和代码具有更高的优先级，所以用户模式下的代码不可以直接访问内核空间中的数据，也不可以直接调用内核空间中的任何函数或例程。任何这样的尝试都会导致保护性错误。也就是说，即使用户空间中的代码指针正确指向了要访问的数据或代码，但一旦访问发生，那么处理器就会检测到该访问是违法的，会停止该访问并产生保护性异常（#GP）。

### 2.系统调用

虽然不能直接访问，但是应用程序可以通过调用系统服务来间接访问内核空间中的数据或间接调用，执行内核空间中的代码。当调用系统服务的时候，主调线程会从用户模式切换到内核模式，调用结束后在返回用户模式，也就是所谓的模式切换。在线程的`KTHEREAD`结构中，定义了`UserTime`和`KernelTime`两个字段，分别用来记录这个线程在用户模式和内核模式的运行事件（以时钟中断计数为单位）。

对于应用程序的运行即进程而言，操作系统内核的作用体现在一组可以供其调用的函数，称为”系统调用“，正是这些系统调用加上一些辅助的手段构成了应用软件的运行环境，即日常所说的”运行平台“。从应用软件的角度看，这些系统调用都是操作系统为其提供的服务，所以也称为”系统服务“。

系统调用所提供的服务是在内核中，一般是在”系统空间“中实现的，而应用软件则都在用户空间运行，二者之间有着空间的间隔。所以，在应用软件和内核之间必定存在一个明确定义的”系统调用界面”。

Windows系统服务界面是隐藏不公开的，公开的是由用户空间一些高层库函数构成的界面，称为“Win32应用程序界面”，即Win32 API。用户模式下的代码通过调用这些Win32 API来调用系统服务完成需要的操作。

因运行状态和执行的程序所在的内存区间的不同，CPU既可以运行于非特权的“用户态”，或者说运行于“用户空间”，也可也运行于特权的“系统态”，或者说运行于“系统空间”。CPU要从系统空间转入用户空间是容易的，因为运行于系统态的CPU可以通过一些特权指令来改变其运行状态。但是，反过来要从用户空间转入系统空间就不容易了，因为运行于用户态的CPU是不能执行特权指令的。所以，一般而言只有三种手段或原因可以使运行于用户空间的CPU转入系统空间：

- 中断：在开启中断机制的情况下，只要有外部设备的中断请求到来，CPU就会自动转入系统空间，并从（系统空间中）某个预定的地址开始执行指令，从而可以在系统空间对外部设备的中断请求作出反应，或者说提供服务。中断只发生在两条指令之间，因而不会使正在执行的指令半途而废。对于CPU而言，因中断而进入系统空间是被动的，并且CPU是完全无法预知何时将发生中断，所以中断的发生又是“异步”的。

- 异常：不管是用户空间或系统空间，执行指令时的失败都会引起一次”异常“，CPU也会因此而转入系统空间（如果原来不再系统空间），并从某个预定的地址开始执行指令，从而可以在系统空间对发生的失败做出反应。异常在形式上与中断十分相似，只是异常发生在执行一条指令的过程中，而不是两条指令之间，所以当前指令的执行已经半途而废。同样，对于CPU而言，因异常而进入系统空间也是被动的，无法预知的。但是，这只是一般而言，实践中也可也通过故意引起异常而进入内核。例如，在用户空间故意访问系统某个特定地址，就会因访问权限的问题而引起主动的异常，而内核空间中的异常处理程序则可以根据所访问的地址判定其目的，从而做出预定的反应。

- 自陷：为了让CPU能够主动地进入系统空间，绝大多数CPU都设有专门地”自陷“指令，系统调用通常就是靠自陷指令实现地。一执行这样的指令，CPU就转入系统空间并从某个预定地地址开始执行指令，就像掉进了陷阱一样。自陷指令在形式上与中断相似，就像是一次由CPU主动发出的中断请求。对于CPU而言，这是成功执行了一条指令的结果，所以是主动的，尽管该指令其实是由程序员安排的。对于程序而言，则一条自陷指令的作用相当于一次子程序调用，只是这个子程序存在于系统空间中。

传统的Windows系统调用正是通过自陷指令`int 0x2E`进入内核实现系统调用的，但是，从Pentium II开始，Intel又在x86系列的CPU中增加了一对指令`sysenter`和`sysexit`，用来实现一种称为“快速系统调用”的机制，并为此增加了三个寄存器。

## 二.Intel x86的用户模式-内核模式的切换

应用程序为了调用系统服务，就需要进行模式切换，将处理器从用户模式切换到内核模式下。模式切换的工作并不需要由应用程序自己来完成，Windows提供了一个系统模块`ntdll.dll`，已经实现了所有系统服务的模式切换工作，在用户模式下只需要通过调用Win32 API就可以由`ntdll.dll`实现相应的模式切换工作，切换到内核模式以后就可以使用系统提供的系统服务。模式切换依赖于硬件系统结构，接下来以`ReadProcessMemory`为例，来具体看看切换的过程。

`ReadProcessMemory`是`kernel32.dll`中的一个导出函数，根据反汇编的结果可以看到，该函数的主要行为就是调用`NtReadVirtualMemory`函数来完成功能

```nasm
.text:7C8021D0 ; BOOL \_\_stdcall ReadProcessMemory(HANDLE hProcess, LPCVOID lpBaseAddress, LPVOID lpBuffer, SIZE\_T nSize, SIZE\_T \*lpNumberOfBytesRead)
.text:7C8021D0 public \_ReadProcessMemory@20
.text:7C8021D0 \_ReadProcessMemory@20 proc near ; CODE XREF: GetProcessVersion(x)+2F18B↓p
.text:7C8021D0 ; GetProcessVersion(x)+2F1AA↓p ...
.text:7C8021D0
.text:7C8021D0 hProcess = dword ptr 8
.text:7C8021D0 lpBaseAddress = dword ptr 0Ch
.text:7C8021D0 lpBuffer = dword ptr 10h
.text:7C8021D0 nSize = dword ptr 14h
.text:7C8021D0 lpNumberOfBytesRead= dword ptr 18h
.text:7C8021D0
.text:7C8021D0 mov edi, edi
.text:7C8021D2 push ebp
.text:7C8021D3 mov ebp, esp
.text:7C8021D5 lea eax, [ebp+nSize]
.text:7C8021D8 push eax ; NumberOfBytesRead
.text:7C8021D9 push [ebp+nSize] ; NumberOfBytesToRead
.text:7C8021DC push [ebp+lpBuffer] ; Buffer
.text:7C8021DF push [ebp+lpBaseAddress] ; BaseAddress
.text:7C8021E2 push [ebp+hProcess] ; ProcessHandle
.text:7C8021E5 call ds:\_\_imp\_\_NtReadVirtualMemory@20 ; NtReadVirtualMemory(x,x,x,x,x)
.text:7C8021EB mov ecx, [ebp+lpNumberOfBytesRead]
.text:7C8021EE test ecx, ecx
.text:7C8021F0 jnz short loc\_7C8021FD
.text:7C8021F2
.text:7C8021F2 loc\_7C8021F2: ; CODE XREF: ReadProcessMemory(x,x,x,x,x)+32↓j
.text:7C8021F2 test eax, eax
.text:7C8021F4 jl short loc\_7C802204
.text:7C8021F6 xor eax, eax
.text:7C8021F8 inc eax
.text:7C8021F9
.text:7C8021F9 loc\_7C8021F9: ; CODE XREF: ReadProcessMemory(x,x,x,x,x)+3C↓j
.text:7C8021F9 pop ebp
.text:7C8021FA retn 14h
.text:7C8021FD ; ---------------------------------------------------------------------------
.text:7C8021FD
.text:7C8021FD loc\_7C8021FD: ; CODE XREF: ReadProcessMemory(x,x,x,x,x)+20↑j
.text:7C8021FD mov edx, [ebp+nSize]
.text:7C802200 mov [ecx], edx
.text:7C802202 jmp short loc\_7C8021F2
.text:7C802204 ; ---------------------------------------------------------------------------
.text:7C802204
.text:7C802204 loc\_7C802204: ; CODE XREF: ReadProcessMemory(x,x,x,x,x)+24↑j
.text:7C802204 push eax ; Status
.text:7C802205 call \_BaseSetLastNTError@4 ; BaseSetLastNTError(x)
.text:7C80220A xor eax, eax
.text:7C80220C jmp short loc\_7C8021F9
.text:7C80220C \_ReadProcessMemory@20 endp
```

`NtReadVirtualMemory`的功能很简单，将服务号`0x0BA`赋给`eax`，该服务号决定了进入内核模式以后使用的系统服务，随后将`0x7FFE0300`赋给`edx`，在调用`edx`保存的地址中保存的函数地址

```nasm
.text:7C92D9E0 public NtReadVirtualMemory
.text:7C92D9E0 NtReadVirtualMemory proc near ; CODE XREF: .text:7C93FFD1↓p
.text:7C92D9E0 ; LdrCreateOutOfProcessImage+7C↓p ...
.text:7C92D9E0 mov eax, 0BAh ; 服务号赋给eax
.text:7C92D9E5 mov edx, 7FFE0300h
.text:7C92D9EA call dword ptr [edx]
.text:7C92D9EC retn 14h
.text:7C92D9EC NtReadVirtualMemory endp
```

要理解`edx`保存的内容，首先要知道，在Windows的虚拟地址空间管理中，系统空间有一个特殊的页面会映射到每个进程地址空间中。也就是说，该页面是系统地址空间和进程地址空间共享的。

它在系统空间的地址由以下宏定义决定

```c
#define KI\_USER\_SHARED\_DATA 0xFFDF0000
```

在进程空间的地址则是由以下的宏定义决定

```c
#define MM\_SHARED\_USER\_DATA\_VA 0x7FFE0000
```

该共享页面是在内存管理器初始化时分配的，记录了当前系统的一些关键状态信息，数据结构`KUSER_SHARED_DATA`定义了该页面内部的结构，该结构定义如下：

```php
kd> dt \_KUSER\_SHARED\_DATA
nt!\_KUSER\_SHARED\_DATA
+0x000 TickCountLow : Uint4B
+0x004 TickCountMultiplier : Uint4B
+0x008 InterruptTime : \_KSYSTEM\_TIME
+0x014 SystemTime : \_KSYSTEM\_TIME
+0x020 TimeZoneBias : \_KSYSTEM\_TIME
+0x02c ImageNumberLow : Uint2B
+0x02e ImageNumberHigh : Uint2B
+0x030 NtSystemRoot : [260] Uint2B
+0x238 MaxStackTraceDepth : Uint4B
+0x23c CryptoExponent : Uint4B
+0x240 TimeZoneId : Uint4B
+0x244 Reserved2 : [8] Uint4B
+0x264 NtProductType : \_NT\_PRODUCT\_TYPE
+0x268 ProductTypeIsValid : UChar
+0x26c NtMajorVersion : Uint4B
+0x270 NtMinorVersion : Uint4B
+0x274 ProcessorFeatures : [64] UChar
+0x2b4 Reserved1 : Uint4B
+0x2b8 Reserved3 : Uint4B
+0x2bc TimeSlip : Uint4B
+0x2c0 AlternativeArchitecture : \_ALTERNATIVE\_ARCHITECTURE\_TYPE
+0x2c8 SystemExpirationDate : \_LARGE\_INTEGER
+0x2d0 SuiteMask : Uint4B
+0x2d4 KdDebuggerEnabled : UChar
+0x2d5 NXSupportPolicy : UChar
+0x2d8 ActiveConsoleId : Uint4B
+0x2dc DismountCount : Uint4B
+0x2e0 ComPlusPackage : Uint4B
+0x2e4 LastSystemRITEventTickCount : Uint4B
+0x2e8 NumberOfPhysicalPages : Uint4B
+0x2ec SafeBootMode : UChar
+0x2f0 TraceLogging : Uint4B
+0x2f8 TestRetInstruction : Uint8B
+0x300 SystemCall : Uint4B
+0x304 SystemCallReturn : Uint4B
+0x308 SystemCallPad : [3] Uint8B
+0x320 TickCount : \_KSYSTEM\_TIME
+0x320 TickCountQuad : Uint8B
+0x330 Cookie : Uint4B
```

其中偏移`0x300`的`SystemCall`指示了从用户模式切换到内核模式的函数地址，而偏移`0x304`的`SystemCallReturn`则指示了从内核模式返回至用户模式的函数地址。因此，在`ntdll.dll`中的`NtReadVirtualMemory`的`edx`被赋予了`SystemCall`的值。

在初始化阶段，系统会根据当前处理器的特征信息来决定`SystemCall`保存的函数。当通过`eax=1`来执行`cpuid`指令的时候，处理器的特征信息会被存放到`ecx`和`edx`寄存器中，其中`edx`包含了一个SEP位（第11位），当该位为1说明当前处理器支持快速系统调用，此时`SystemCall`中保存的就会是通过快速系统调用机制切换到内核模式的`KiFastSystemCall`的函数地址，否则保存的就是通过自陷指令进入系统内核的`KiIntSystemCall`。

### 1.通过自陷指令切换到内核模式

根据`KiIntSystemCall`反汇编的结果可以知道，函数首先会将传入的参数的地址赋给`edx`，随后使用"`int 0x2E`"指令通过中断门来切换到内核模式

```nasm
.text:7C92E500 public KiIntSystemCall
.text:7C92E500 KiIntSystemCall proc near ; DATA XREF: .text:off\_7C923428↑o
.text:7C92E500
.text:7C92E500 arg\_4 = byte ptr 8
.text:7C92E500
.text:7C92E500 lea edx, [esp+arg\_4] ;将参数地址赋给edx
.text:7C92E504 int 2Eh ; DOS 2+ internal - EXECUTE COMMAND
.text:7C92E504 ; DS:SI -> counted CR-terminated command string
.text:7C92E506 retn
.text:7C92E506 KiIntSystemCall endp
```
