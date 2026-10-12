
# Debug Tutorial(完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-2.html`](https://www.pediy.com/kssd/pediy12/104828-2.html)

## 栈

### 介绍

欢迎来到这一系列教程的第二篇。在这篇文章，我们将探讨一些关于栈的问题和它在调试中的作用。每当你问别人”如果程序出现错误了该怎么办？”，最常见的回答就是”看栈回溯”。这是很对的，当你分析一个 CrashDump 的时候，你要做的第一件事就是得到当时的栈回溯。

### 栈是什么？

这是第一个，也是非常明显的一个问题。不幸的是，在上一篇文章中，我并没有回答这个问题，因为我想当然的以为每个人都很熟悉它。为了解释什么是栈，首先我们来看看什么是进程。

### 进程是什么？

进程就是应用程序在内存中的一个实例。可执行文件和需要的库都被映射到进程的地址空间中，进程并不执行，它只是定义了内存边界，资源，哪些用户可以操作哪些东西。

### 线程是什么？

线程是进程中可执行的一个实例。进程并不会执行，真正执行的是进程中的线程，一个进程中会有许多线程在执行。虽然一个线程会有线程自己特定的存储区，一般情况下，进程中的所有内存和资源都是可以被执行的线程使用的。

### 全局资源和局部资源

资源可在全局创建，也可以在局部创建，这也就是说这些资源可以在创建它的进程之外被使用。其中一个例子就是窗口句柄。这些资源有他们自己的作用范围。有些资源是系统范围的，有些是桌面范围内的或会话范围内的。另外，不同的进程也可以协商它们自己定义好的共享资源。

### 虚拟内存是什么？

虚拟内存的意思就是系统以为自己有那么多内存，但实际上物理内存没有那么多。这可以说是对的，也是错的，因为这主要看你的系统到底什么样的。

系统事实上并没有被欺骗，硬件知道自己到底有多少内存，所以我们需要一种机制来实现”虚拟内存”。操作系统是实现这一机制的源头，它不可能被欺骗，所以我们说真正被欺骗的也就是运行在系统中的进程。

那么，”虚拟内存”所做的就是将物理内存抽象，这意味着应用程序看不见，也不知道物理内存，它们只知道虚拟的内存。有操作系统的帮助，`CPU` 就可以把虚拟地址转化为物理地址。更细节的东西，这里就不讨论了，你只要知道应用程序只接受虚拟地址，处理器会把它映射为物理地址。

虚拟地址也不一定都指向物理地址，操作系统可以在一个交换文件中存储虚拟地址中的内容，这样，整个程序就不需要同时都在物理内存中了。这也就可以让许多程序在内存中并执行，如果一个程序想要访问一个地址，但是这个地址又不在物理内存中，`CPU` 知道，然后 `CPU` 就会产生缺页中断，并从磁盘上将内容加载入物理内存，这样这个程序就能访问这个地址，然后继续执行。

页交换的算法有很多种，最简单常用的就是最不经常使用页。当你写程序的时候，数据和代码最好都是页对齐的，这样就不会出现太频繁的页换入和页换出。

虚拟内存的一个优点就是能起到保护作用。一个进程不能直接操作另外一个进程的内存空间，在一个时刻，`CPU` 只认识当前进程的地址空间，而不能操作其他进程的地址空间。这是很有意义的，因为它们是分开映射的，同一个地址对应的是不同的地方。

但这并不意味着我们不可能访问其他进程的内存。`Windows` 内部提供这样一种机制，你可以通过它来访问其他内存的空间。你可以操作和虚拟内存相关的 `CPU` 寄存器(`CR3`)来达到目的，但这不可能，因为用户态下，你并没有这样的权限。

### 栈是什么?

我们已经解释了系统中一些基本的概念，然后我们回到正题，什么是栈？栈是我们通常意义上所说的数据结构，它允许一个东西压进去和弹出来。你可以把它看作一堆盘子，你可以把一个盘子放到这堆的最上面，你也只能从最上面拿走盘子，根据这个后进先出的规则，就构成了一个栈。

程序通常把栈当作暂时存储器。非汇编程序员通常不知道这个细节，因为编译器会向你隐藏这个细节。程序内部还是会使用到内部已经建好的栈。

在 `Intel` 平台上，压栈和出栈的汇编指令分别是 `PUSH` 和 `POP`，有些处理器会用到 `PUSH/PULL`。汇编指令只是 `CPU` 内部制定好的一些你可以使用的指令，这些英文代表 `CPU` 需要做怎样的操作，也就是相应的机器码，你可以将这些指令结合起来做任何你想要做的事情。

每个线程都有自己的栈，因为多个线程用同一个存储空间将出现错乱。

## 函数调用是如何实现的？

函数调用依赖于调用约定。调用约定就是调用者与被调用者达成协议，确定参数如何传递，以及参数如何清除。`Windows` 平台一般支持 3 种调用约定，”this call”，”standard call”，”CDECL or C Call”。

### This Call

这是 C++中的调用约定。如果你熟悉 C++的话，你会知道成员函数都有一个 `this` 指针作为参数，并且这个指针通常是栈中的第一个参数。但这并不是这里说的 This Call，This Call 中的 `this` 指针是存在寄存器中的，确切的说是 `ECX`，参数以反向压入栈，最后由被调用者清理栈。

### Standard call

“Standard Call”就是参数以反向压入栈，然后被调用者清理栈。

### CDECL or C Calling Convention

“C Calling”调用约定就是参数以反向压入栈，然后调用者清理栈。

### Pascal Calling Convention

如果你见过以前的程序，你可能会建国 PASCAL 调用约定。在 `WIN32` 中，`_pascal` 不再允许使用了，PASCAL 其实就是一个宏，被定义为 Standard Call。然后，Pascal 调用约定的参数是以正向压入栈众的，由被调用者清理栈。

### 清理栈？

由谁来清理栈是很重要的一件事。首先就是保存内容。若果被调用者清理栈，就不需要在每个函数调用中产生一些代码来清理栈，缺点是你不能使用可变参数。像 `printf` 这样的函数就是使用可变参数，被调用者并不知道有多少参数压入了栈，它只能靠格式化的字符串猜测。如果这个字符串是 `printf "%i %i %i"`，它将适用 3 个栈中的参数来完成，而不管你是否真的压入了这个参数，这就可能产生错误。如果你压入了比这更多的参数，这将不会出现问题，因为调用者会负责清理栈。记住，可变参函数并不知道到底有多少参数，它们必须靠调用者告诉他们参数列表，`Printf` 就是通过格式化字符串来知道到底有多少个参数的。

虽然它有可能可以清理栈，但是这还是不可行，因为函数在编译的时候不能确定有几个参数传给它，让调用者来清理栈会变得更加容易。

`Intel` 提供了让被调用者清理栈的指令，这个指令是 `RET <Byte Count>`，`Byte Count` 就是压入栈中的参数的总字节数。

## 那么，栈到底是什么？

栈就是一个临时存储区，参数可以压入栈，然后返回地址压入栈，`CPU` 是很傻的，它只会一条指令接着一条指令执行，你必须告诉它该往哪里执行，为了让它知道改怎么返回，我们必须保存返回地址。汇编指令 `CALL` 和 `RET` 就是函数调用和返回的指令。另外你还可能会在栈中分配一些局部变量，然后在函数返回时会自动清除，这就是你不能返回局部变量的地址的原因。

一个栈的布局如下:

```
[Parameter n          ]

...

[Parameter 2          ]

[Parameter 1          ]

[Return Address       ]

[Previous Base Pointer]

[Local Variables      ]
```

在返回之前，栈被清理，并且返回到返回地址。如果栈不按合适顺序存放，我们将会得到错误的返回地址，这将很明显会产生一个错误。

## 基址指针什么？

基址指针就是 `Intel` 平台上的 `EBP` 寄存器。`ESP` 是经常变动的，你必须保存以前函数的 `EBP`，然后将 `EBP` 设为当前的栈地址，然后你就可以直接用一个标准来取得栈里的变量，也就是说第一个参数总是 `EBP+XX`。如果你不保存 `ESP`，并且总是引用 `ESP`，你必须跟踪当前栈中的数据，如果你以后又想在栈中压入数据，那么第一个参数的偏移就会改变。所以我们会在函数调用时，总是将 `EBP` 设为当前的 `ESP` 值，而不直接使用 `ESP`，然后就能通过 `EBP+Value` 来得到参数，`EBP-Value` 来的到局部变量。

所以，这就是每个线程都有自己的栈的原因，如果它们共享一个栈，它们将会出现互相覆盖数据的情况。

## 栈溢出

栈溢出的意思就是说你已经用完了你的栈，没有可以再用的空间了。`Windows` 通常给一个程序分配的用户态栈空间大小都是固定的，内核都另外自己的栈。递归就经常会将栈空间用完。`Windows` 不会一次就分配你需要的所有栈，当你需要的时候，它会再给你分配，这是一个优化选项。

我们可以编写一个小程序来看看演示栈溢出，并看看 `Windows` 给我们分配了多少栈空间。

```
0:000> g

(928.898): Stack overflow - code c00000fd (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=00131ad8 ebx=7ffdf000 ecx=00131ad8 edx=00430df0 esi=00000000 edi=0003347c

eip=00401029 esp=00032ffc ebp=00033230 iopl=0         nv up ei pl nz ac po nc

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00010216

*** WARNING: Unable to verify checksum for COMSTRESS.exe

COMSTRESS!func+0x9:

00401029 53             push    ebx

0:000> !teb

TEB at 7ffde000

    ExceptionList:        0012ffb0

    StackBase:            00130000

    StackLimit:           00031000

    SubSystemTib:         00000000

    FiberData:           00001e00

    ArbitraryUserPointer: 00000000

    Self:                 7ffde000

    EnvironmentPointer:   00000000

    ClientId:             00000928 . 00000898

    RpcHandle:           00000000

    Tls Storage:         00000000

    PEB Address:          7ffdf000

    LastErrorValue:       0

    LastStatusValue:      0

    Count Owned Locks:    0

    HardErrorMode:        0

0:000> ? 130000 - 31000

Evaluate expression: 1044480 = 000ff000

0:000>
```

我使用`!teb` 命令来显示 `TEB` 也就是 `Thread Environment Block`。我们可以看到，`1044480 bytes` 就是 `Windows` 给我们分配的栈空间。

## 栈上溢

栈上溢和栈溢出是相反的。你以为你往栈中压入了很多数据，然后弹出了这么多数据，而事实上并没有这么多，所以你就超过了栈的上边界，从而造成了栈上溢。

## 溢出和上溢

溢出和上溢其实就是对栈的处理不当，并且在最终使崩溃发生的不同的位置。如果你清理了过多并且返回将有可能发生上溢，出现这种情况的原因是，你以为你有这么多数据在栈中，而实际上没有。相反地，溢出和这类似。

## 调试器如何得到栈回溯

下一个话题就是，如何得到栈回溯？第一个答案就是符号。符号能告诉调试器栈中有多少参数，多少局部变量等等，因此调试器可以使用符号来确定怎样遍历栈以及显示信息。

如果没有符号，我们可以用基址指针。每个基址指针都指向以前的基址，基址+4 指向返回地址，按这种规则我们就能遍历栈。虽然调试器不知道由多少参数，它只是 Dump 出来可能的参数的，最后依靠你自己来判定正确的参数。

下面是一个简单的函数调用表。

```
0:000> kb

ChildEBP RetAddr   Args to Child

0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18

0012ff38 00401044 00000000 77f944a8 00000007 MSVCRT!printf+0x35

0012ff4c 00401147 00000001 00323d70 00322ca8 temp!main+0x44

0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3

0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23
```

我将会使用”DDS”命令，这个命令会将栈中的值和最近的符号匹配。

我们当前的 `EBP` 值是 `0012fef4`，这是栈中的一个指针，这个值指向以前的 `EBP`，记住 `EBP+4==返回地址`，`EBP+8==参数`。

```
[Stack Address |  Value  | Description]

0012fef4  0012ff38

0012fef8  77c3e68d MSVCRT!printf+0x35

0012fefc  77c5aca0 MSVCRT!_iob+0x20

0012ff00  00000000

0012ff04  0012ff44

0012ff08  77c5aca0 MSVCRT!_iob+0x20

0012ff0c  00000000

0012ff10  000007e8

0012ff14  7ffdf000

0012ff18  0012ffb0

0012ff1c  00000001

0012ff20  0012ff0c

0012ff24  0012f8c8

0012ff28  0012ffb0

0012ff2c  77c33eb0 MSVCRT!_except_handler3

0012ff30  77c146e0 MSVCRT!`string'+0x16c

0012ff34  00000000

0012ff38  0012ffc0

0012ff3c  00401044 temp!main+0x44

0012ff40  00000000

0012ff44  77f944a8 ntdll!RtlpAllocateFromHeapLookaside+0x42

0012ff48  00000007

0012ff4c  00000000

0012ff50  00401147 temp!mainCRTStartup+0xe3

0012ff54  00000001

0012ff58  00323d70

0012ff5c  00322ca8

0012ff60  00403000 temp!__xc_a

0012ff64  00403004 temp!__xc_z

0012ff68  0012ffa4

0012ff6c  0012ff94

0012ff70  0012ffa0

0012ff74  00000000

0012ff78  0012ff98

0012ff7c  00403008 temp!__xi_a

0012ff80  0040300c temp!__xi_z

0012ff84  77f944a8 ntdll!RtlpAllocateFromHeapLookaside+0x42

0012ff88  00000007

0012ff8c  7ffdf000

0012ff90  c0000005

0012ff94  00323d70

0012ff98  00000000

0012ff9c  8053476f

0012ffa0  00322ca8

0012ffa4  00000001

0012ffa8  0012ff84

0012ffac  0012f8c8

0012ffb0  0012ffe0

0012ffb4  00401210 temp!except_handler3

0012ffb8  004020d0 temp!⌂MSVCRT_NULL_THUNK_DATA+0x80

0012ffbc  00000000

0012ffc0  0012fff0

0012ffc4  77e814c7 kernel32!BaseProcessStart+0x23

0012ffc8  77f944a8 ntdll!RtlpAllocateFromHeapLookaside+0x42

0012ffcc  00000007

0012ffd0  7ffdf000

0012ffd4  c0000005

0012ffd8  0012ffc8

0012ffdc  0012f8c8
```



```
0012ffe0  ffffffff
0012ffe4  77e94809  kernel32!_except_handler3
0012ffe8  77e91210  kernel32!`string'+0x98
0012ffec  00000000
0012fff0  00000000
0012fff4  00000000
0012fff8  00401064  temp!mainCRTStartup
```

因此，`EBP`(`0012fef4`)指向以前的 `EBP`(`0012ff38`)。`EIP`==`77c3f10b`，也就是 `MSVCRT!_output+0x18`。`KB` 命令一般只 Dump 出前 3 个参数，它并不知道这是否是正确的参数，仅仅是一种预测，如果你想知道其他的参数，你只要找到地址，然后 Dump 出相应栈空间即可。

```
0012fefc  77c5aca0  MSVCRT!_iob+0x20
0012ff00  00000000
0012ff04  0012ff44
```

然后，我们可以汇编第一个函数：

`MSVCRT!_output+0x18(77c5aca0, 00000000, 0012ff44);`

第二个函数是 `EBP+4`，也就是返回地址。记住，它并不知道这个函数从哪开始，因此，最好的办法就是将返回地址和函数匹配。

下面是调用函数：

```
0012fef8  77c3e68d  MSVCRT!printf+0x35
```

然后我们得到了以前的 `EBP`，`0012ff38`，加 8 就是参数。

```
0012ff40  00000000
0012ff44  77f944a8  ntdll!RtlpAllocateFromHeapLookaside+0x42
0012ff48  00000007
```

这是调用函数和参数。

`MSVCRT!printf+0x35(00000000, 77f944a8, 00000007);`

你可以看到，如果有任何不对，这些信息都是错误的，所以这都要看你自己的判断。

下一个 `EBP` 是 `0012ffc0`，它指向内存地址 `0012ff38`，返回值为：

```
0012ff3c  00401044  temp!main+0x44
```

这里的参数是 `0012ffc0+8`，记住，我们总是假设 `EBP` 是栈上的第一个值。如果调试器足够智能，那么它就能够遍历栈直到第一个可识别的符号，然后让它作为返回值。

下面这些是参数：

```
0012ffc8  77f944a8  ntdll!RtlpAllocateFromHeapLookaside+0x42
0012ffcc  00000007
0012ffd0  7ffdf000
```

`temp!main+0x44(77f944a8, 00000007, 7ffdf000)`

我们的下一个 `EBP` 是 `0012ffc0`，因此加上 4 就是返回值。

```
0012ffc4  77e814c7  kernel32!BaseProcessStart+0x23
```

`EBP`=`0012ffc0`，指向上一个 `EBP` `0012fff0`。

`0:000> dds 0012fff0`

```
0012fff0  00000000
0012fff4  00000000  <-- Previous return value is NULL so stop here.
0012fff8  00401064  temp!mainCRTStartup  <-- + 8
0012fffc  00000000
00130000  78746341
```

`kernel32!BaseProcessStart+0x23(00401064, 00000000, 78746341)`

可以看到，上一个返回值为 `NULL`，所以我们人工构造的栈如下：

```
MSVCRT!_output+0x18          (77c5aca0, 00000000, 0012ff44);
MSVCRT!printf+0x35          (00000000, 77f944a8, 00000007);
temp!main+0x44              (77f944a8, 00000007, 7ffdf000);
kernel32!BaseProcessStart+0x23 (00401064, 00000000, 78746341);
```

下面是调试器的栈回溯：

`ChildEBP RetAddr Args to Child`

```
0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18
0012ff38 00401044 00000000 77f944a8 00000007 MSVCRT!printf+0x35
0012ff4c 00401147 00000001 00323d70 00322ca8 temp!main+0x44
0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23
```

为什么会有些不同呢？我们遵循的规则是 `EBP` 指向以前的 `EBP`，并且我们没有使用符号信息。如果我将符号删除，得到的栈回溯如下：

`0:000> kb`

`ChildEBP RetAddr Args to Child`

```
0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18
0012ff38 00401044 00000000 77f944a8 00000007 MSVCRT!printf+0x35
WARNING: Stack unwind information not available. Following frames may be wrong.
0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp+0x1044
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23
0:000>
```

可以看到，和我们得到的信息一样。因此，调试器使用符号信息来遍历栈，并显示准确的信息，然而，如果没有符号信息，这些函数调用都将丢失，也就是说，我们不能总是相信栈回溯，如果符号是错误的，这将引起很大的麻烦。

### 验证函数调用

- 再次运行程序并得到栈回溯。

`0:000> kb`

`ChildEBP RetAddr Args to Child`

```
0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18
0012ff38 00401044 00000000 00000000 00000000 MSVCRT!printf+0x35
WARNING: Stack unwind information not available. Following frames may be wrong.
0012ffc0 77e814c7 00000000 00000000 7ffdf000 temp+0x1044
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23
0:000>
```

栈中的一些值会和以前不同，因为你不能保证每次程序的运行都和以前一模一样。

第一个返回地址：`77c3e68d`

如果你反汇编这个地址，将得到：

`0:000> u 77c3e68d`

`MSVCRT!printf+0x35:`

```
77c3e68d 8945e0    mov     [ebp-0x20],eax
77c3e690 56        push    esi
77c3e691 ff75e4    push    dword ptr [ebp-0x1c]
```

这些代码可以这样理解：

`<address> <opcode> <assembly instruction in english or mnemonic>`

```
77c3e691 ff75e4  push dword ptr [ebp-0x1c]

77c3e691 == Address
ff75e4   == Opcode or machine code. This is what the CPU understands
push dword ptr [ebp-0x1c] == Assembly instruction in english. The mnemonic.
```

这是返回值。

`0:000> u 77c3e68d - 20`

`MSVCRT!printf+0x15:`

```
77c3e66d bdadffff59    mov     ebp,0x59ffffad
77c3e672 59            pop     ecx
77c3e673 8365fc00      and     dword ptr [ebp-0x4],0x0
77c3e677 56            push    esi
77c3e678 e8c7140000    call    MSVCRT!_stbuf (77c3fb44)
77c3e67d 8945e4        mov     [ebp-0x1c],eax
77c3e680 8d450c        lea     eax,[ebp+0xc]
77c3e683 50            push    eax
```

`0:000> u`

`MSVCRT!printf+0x2c:`

```
77c3e684 ff7508        push    dword ptr [ebp+0x8]
77c3e687 56            push    esi
77c3e688 e8660a0000    call    MSVCRT!_output (77c3f0f3)
77c3e68d 8945e0        mov     [ebp-0x20],eax
```

你可以看到，返回地址是 `77c3e68d`，因此，`77c3e688` 是函数调用，也就是在调用 `_output`。

栈回溯中下一个返回地址是 `00401044`，用相同的步骤来试验一下：

`0:000> u 00401044 - 20`

`temp+0x1024:`

```
00401024 2408        and     al,0x8
00401026 57          push    edi
00401027 50          push    eax
00401028 6a04        push    0x4
0040102a 6820304000  push    0x403020
0040102f 56          push    esi
00401030 ff1500204000 call  dword ptr [temp+0x2000 (00402000)]
00401036 56          push    esi
```

`0:000> u`

`temp+0x1037:`

```
00401037 ff1508204000 call  dword ptr [temp+0x2008 (00402008)]
0040103d 57          push    edi
0040103e ff1510204000 call  dword ptr [temp+0x2010 (00402010)]
00401044 59          pop     ecx
```

从这些汇编语句中，我们看到，正在调用地址 `00402010` 处的函数。

用 `DD` 命令得到地址中的值。

`0:000> dd 00402010`

```
00402010  77c3e658
```

然后，我们来反汇编这个函数。

`0:000> u 77c3e658`

`MSVCRT!printf:`

```
77c3e658 6a10        push    0x10
```

是的，我们在调用 `printf` 函数。

下一个返回值是 `77e814c7`。

`0:000> u 77e814c7 - 20`

`kernel32!BaseProcessStart+0x3:`

```
77e814a7 1012        adc     [edx],dl
77e814a9 e977e8288e  jmp     0610fd25
77e814ae ffff        ???
77e814b0 8365fc00    and     dword ptr [ebp-0x4],0x0
77e814b4 6a04        push    0x4
77e814b6 8d4508      lea     eax,[ebp+0x8]
77e814b9 50          push    eax
77e814ba 6a09        push    0x9
```

`0:000> u`

`kernel32!BaseProcessStart+0x18:`

```
77e814bc 6afe        push    0xfe
77e814be ff159c13e677 call  dword ptr [kernel32!__imp__NtSetInformationThread (77e6139c)]
77e814c4 ff5508      call    dword ptr [ebp+0x8]
77e814c7 50          push    eax
```

它正在调用第一个参数，我们来看看。

```
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23
```

`0:000> u 00401064`

`temp+0x1064:`

```
00401064 55          push    ebp
00401065 8bec        mov     ebp,esp
00401067 6aff        push    0xff
```

可以看到，它正在调用 `temp` 中的某个函数，但是我们不能确定这是同一个函数，因此需要反汇编来一步一步看。记住，`Printf` 的调用在这里：

```
0040103e ff1510204000 call  dword ptr [temp+0x2010 (00402010)]
```

因此我们需要反汇编这个函数一直到 `0401064`。另外一种方法就是使用 `DDS` 命令来找到栈中的相应符号并验证。

我们在 `EBP` 上使用 `DDS`：

`0:000> dds ebp`

```
0012fef4  0012ff38
0012fef8  77c3e68d MSVCRT!printf+0x35
0012fefc  77c5aca0 MSVCRT!_iob+0x20
0012ff00  00000000
0012ff04  0012ff44
0012ff08  77c5aca0 MSVCRT!_iob+0x20
0012ff0c  00000000
0012ff10  000007e8
0012ff14  7ffdf000
0012ff18  0012ffb0
0012ff1c  00000001
0012ff20  0012ff0c
0012ff24  ffffffff
0012ff28  0012ffb0
0012ff2c  77c33eb0 MSVCRT!_except_handler3
0012ff30  77c146e0 MSVCRT!`string'+0x16c
0012ff34  00000000
0012ff38  0012ffc0
0012ff3c  00401044 temp+0x1044
0012ff40  00000000
0012ff44  00000000
0012ff48  00000000
0012ff4c  00000000
0012ff50  00401147 temp+0x1147
0012ff54  00000001
0012ff58  00322470
0012ff5c  00322cf8
0012ff60  00403000 temp+0x3000
0012ff64  00403004 temp+0x3004
0012ff68  0012ffa4
0012ff6c  0012ff94
0012ff70  0012ffa0
```

`0:000> dds`

```
0012ff74  00000000
0012ff78  0012ff98
0012ff7c  00403008 temp+0x3008
0012ff80  0040300c temp+0x300c
0012ff84  00000000
0012ff88  00000000
0012ff8c  7ffdf000
```



```
0012ff90   00000001  
0012ff94   00322470  
0012ff98   00000000  
0012ff9c   8053476f  
0012ffa0   00322cf8  
0012ffa4   00000001  
0012ffa8   0012ff84  
0012ffac   e3ce0b30  
0012ffb0   0012ffe0  
0012ffb4   00401210 temp+0x1210  
0012ffb8   004020d0 temp+0x20d0  
0012ffbc   00000000  
0012ffc0   0012fff0  
0012ffc4   77e814c7 kernel32!BaseProcessStart+0x23
```

在栈中有很多未知的 `TEMP+XXX`，其中一个就是 `printf` 的返回值。

我们来看看最近的这个符号：

```
0012ff50   00401147 temp+0x1147  
```

```
0:000> u 00401147 - 20  
temp+0x1127:  

00401127 40                inc     eax  
00401128 00e8              add     al,ch  
0040112a 640000            add     fs:[eax],al  
0040112d 00ff              add     bh,bh  
0040112f 1520204000        adc     eax,0x402020  
00401134 8b4de0            mov     ecx,[ebp-0x20]  
00401137 8908              mov     [eax],ecx  
00401139 ff75e0            push    dword ptr [ebp-0x20]  
```

```
0:000> u  
temp+0x113c:  

0040113c ff75d4            push    dword ptr [ebp-0x2c]  
0040113f ff75e4            push    dword ptr [ebp-0x1c]  
00401142 e8b9feffff        call    temp+0x1000 (00401000)  
00401147 83c430            add     esp,0x30  
```

我们找到了这个函数调用，并且似乎是一个有效的地址。

我们来反汇编这个函数：

```
0:000> u 00401000  
temp+0x1000:  

00401000 51                push    ecx  
00401001 56                push    esi  
00401002 57                push    edi  
00401003 33ff              xor     edi,edi  
00401005 57                push    edi  
00401006 57                push    edi  
00401007 6a03              push    0x3  
00401009 57                push    edi  
```

```
0:000> u  
temp+0x100a:  

0040100a 57                push    edi  
0040100b 6800000080        push    0x80000000  
00401010 6810304000        push    0x403010  
00401015 ff1504204000      call    dword ptr [temp+0x2004 (00402004)]  
0040101b 8bf0              mov     esi,eax  
0040101d 83feff            cmp     esi,0xffffffff  
00401020 741b              jz      temp+0x103d (0040103d)  
00401022 8d442408          lea     eax,[esp+0x8]  
```

```
0:000> u  
temp+0x1026:  

00401026 57                push    edi  
00401027 50                push    eax  
00401028 6a04              push    0x4  
0040102a 6820304000        push    0x403020  
0040102f 56                push    esi  
00401030 ff1500204000      call    dword ptr [temp+0x2000 (00402000)]  
00401036 56                push    esi  
00401037 ff1508204000      call    dword ptr [temp+0x2008 (00402008)]  
```

```
0:000>  
temp+0x103d:  

0040103d 57                push    edi  
0040103e ff1510204000      call    dword ptr [temp+0x2010 (00402010)]  
00401044 59                pop     ecx  
00401045 5f                pop     edi  
00401046 33c0              xor     eax,eax  
00401048 5e                pop     esi  
00401049 59                pop     ecx  
0040104a c3                ret  
0:000>  
```

这看起来是一个有效的函数调用，并且似乎会调用 `Printf`。因此，我们可以反汇编原始函数调用，直到我们到达这个调用。

```
0:000> u 0401064  
temp+0x1064:  

00401064 55                push    ebp  
00401065 8bec              mov     ebp,esp  
00401067 6aff              push    0xff  
00401069 68d0204000        push    0x4020d0  
0040106e 6810124000        push    0x401210  
00401073 64a100000000      mov     eax,fs:[00000000]  
00401079 50                push    eax  
0040107a 64892500000000    mov     fs:[00000000],esp  
```

```
0:000> u  
temp+0x1081:  

00401081 83ec20            sub     esp,0x20  
00401084 53                push    ebx  
00401085 56                push    esi  
00401086 57                push    edi  
00401087 8965e8            mov     [ebp-0x18],esp  
0040108a 8365fc00          and     dword ptr [ebp-0x4],0x0  
0040108e 6a01              push    0x1  
00401090 ff153c204000      call    dword ptr [temp+0x203c (0040203c)]  
```

```
0:000>  
temp+0x1096:  

00401096 59                pop     ecx  
00401097 830d40304000ff    or    dword ptr [temp+0x3040 (00403040)],0xffffffff  
0040109e 830d44304000ff    or    dword ptr [temp+0x3044 (00403044)],0xffffffff  
004010a5 ff1538204000      call    dword ptr [temp+0x2038 (00402038)]  
004010ab 8b0d3c304000      mov     ecx,[temp+0x303c (0040303c)]  
004010b1 8908              mov     [eax],ecx  
004010b3 ff1534204000      call    dword ptr [temp+0x2034 (00402034)]  
004010b9 8b0d38304000      mov     ecx,[temp+0x3038 (00403038)]  
```

```
0:000>  
temp+0x10bf:  

004010bf 8908              mov     [eax],ecx  
004010c1 a130204000        mov     eax,[temp+0x2030 (00402030)]  
004010c6 8b00              mov     eax,[eax]  
004010c8 a348304000        mov     [temp+0x3048 (00403048)],eax  
004010cd e8e1000000        call    temp+0x11b3 (004011b3)  
004010d2 833d2830400000    cmp     dword ptr [temp+0x3028 (00403028)],0x0  
004010d9 750c              jnz     temp+0x10e7 (004010e7)  
004010db 68b0114000        push    0x4011b0  
```

```
0:000>  
temp+0x10e0:  

004010e0 ff152c204000      call    dword ptr [temp+0x202c (0040202c)]  
004010e6 59                pop     ecx  
004010e7 e8ac000000        call    temp+0x1198 (00401198)  
004010ec 680c304000        push    0x40300c  
004010f1 6808304000        push    0x403008  
004010f6 e897000000        call    temp+0x1192 (00401192)  
004010fb a134304000        mov     eax,[temp+0x3034 (00403034)]  
00401100 8945d8            mov     [ebp-0x28],eax  
```

```
0:000>  
temp+0x1103:  

00401103 8d45d8            lea     eax,[ebp-0x28]  
00401106 50                push    eax  
00401107 ff3530304000      push    dword ptr [temp+0x3030 (00403030)]  
0040110d 8d45e0            lea     eax,[ebp-0x20]  
00401110 50                push    eax  
00401111 8d45d4            lea     eax,[ebp-0x2c]  
00401114 50                push    eax  
00401115 8d45e4            lea     eax,[ebp-0x1c]  
```

```
0:000>  
temp+0x1118:  

00401118 50                push    eax  
00401119 ff1524204000      call    dword ptr [temp+0x2024 (00402024)]  
0040111f 6804304000        push    0x403004  
00401124 6800304000        push    0x403000  
00401129 e864000000        call    temp+0x1192 (00401192)  
0040112e ff1520204000      call    dword ptr [temp+0x2020 (00402020)]  
00401134 8b4de0            mov     ecx,[ebp-0x20]  
00401137 8908              mov     [eax],ecx  
```

```
0:000>  
temp+0x1139:  

00401139 ff75e0            push    dword ptr [ebp-0x20]  
0040113c ff75d4            push    dword ptr [ebp-0x2c]  
0040113f ff75e4            push    dword ptr [ebp-0x1c]  
00401142 e8b9feffff        call    temp+0x1000 (00401000)  
00401147 83c430            add     esp,0x30  
```

如果你懂汇编，你可以简单的通读一遍。你会发现，有两个函数调用丢失了，所以我们必须验证这个跟踪是否正确。可以看到，前面的函数没有调用 `printf`，而另外一个函数却调用了。

`00401147` 是丢失的返回值，如果我们在栈中找到它，我们就能得到正确的参数:

```
00000000  
00401147 temp+0x1147  
00000001  
00322470  
00322cf8  
```

下面是 `kb` 命令的到的：

```
0:000> kb  
ChildEBP RetAddr  Args to Child  
0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18  
0012ff38 00401044 00000000 00000000 00000000 MSVCRT!printf+0x35  
WARNING: Stack unwind information not available. Following frames may be wrong.  
0012ffc0 77e814c7 00000000 00000000 7ffdf000 temp+0x1044  
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23  
0:000>  
```

下面是我们改正后得到的：

```
ChildEBP RetAddr  Args to Child  
0012fef4 77c3e68d 77c5aca0 00000000 0012ff44 MSVCRT!_output+0x18  
0012ff38 00401044 00000000 00000000 00000000 MSVCRT!printf+0x35  
WARNING: Stack unwind information not available. Following frames may be wrong.  
xxxxxxxx  0401147  00000001 00322470 00322cf8 temp+0x1044  
0012ffc0 77e814c7 00000000 00000000 7ffdf000 temp+0x1147  
0012fff0 00000000 00401064 00000000 78746341 kernel32!BaseProcessStart+0x23  
0:000>  
```

我们知道，`TEMP` 中调用 `Printf` 的是 `main`，因此，`argc=1`，`*argv[]=322470`。

`*argv[]` 是一个字符串数组指针。

```
0:000> dd 322470  
00322470  00322478 00000000 706d6574 ababab00  
00322480  abababab feeefeab 00000000 00000000  
00322490  000500c5 feee0400 00325028 00320178  
003224a0  feeefeee feeefeee feeefeee feeefeee  
003224b0  feeefeee feeefeee feeefeee feeefeee  
```



```
003224c0   feeefeee feeefeee feeefeee feeefeee  
003224d0   feeefeee feeefeee feeefeee feeefeee  
003224e0   feeefeee feeefeee feeefeee feeefeee  

0:000> da 00322478  
00322478   "temp"  
```

Dump 处这个数组，只有一个元素，使用`da`命令得到。

### 栈中的多返回地址？

为什么栈中会有多个返回地址呢？栈首先被初始化为`0`，然后逐渐使用，并被填满各种数据。局部变量并不总是会初始化的，因此如果你有一个函数调用，这些值不会被重置为`0`。如果你从栈中弹出一个值，栈指针变化，但是栈中的这个值还在，除非你手动做清除工作。而一般优化是不会做清楚工作的，因此你会在栈中看到很多没用的数据。

把值留在栈中并不总是我们所期望的，比如如果你把密码放入栈中，然后出现崩溃，我们只要将栈 Dump 出来就有可能得到你的密码。因此，对于一些敏感信息，在你返回时，你应该做清理工作。其中一个方法就是使用`SecureZeroMemory()` API。这个函数可以帮助你安全的清空内存，比如，当你返回前调用它，编译器就知道你不想再使用这些变量了，就会做一些清理工作。

### 缓冲区溢出

缓冲区溢出是发生在栈中很常见的现象。栈在内存中是向下增长的，而数组是向上增长的。下面我们来看看这个 C 函数：

```c
DWORD MyArray[4];
int Index;
```

这在栈中的布局如下：

```
424 [Return Address           ]
420 [ Previous Base Pointer    ]
416 [ Local Array Variable Index 3]
412 [ Local Array Variable Index 2]
408 [ Local Array Variable Index 1]
404 [ Local Array Variable Index 0]
400 [ Local Integer Value      ]
```

你可以看到，如果你用索引这样使用你的数组，`MyArray[4]`，`MyArray[5]`，这将会覆盖掉一些重要数据，并可能产生崩溃。如果调用函数不再使用前一个`EBP`，覆盖掉它可能不会出现什么问题，但是覆盖掉返回地址，就非常确定的回产生一个崩溃。因此你必须小心使用你的局部变量，因为你可以对他们做你想做的任何操作。

### Windows 2003

Windows 2003 有一个方法可以防止你出现缓冲区溢出。在`VS.NET`编译时设置`GS`标志即可。程序启动时，一个随机的数会产生来作为`cookie`，这个`cookie`会合返回地址作`XOR`运算，然后被放在基址指针后面，下面是一个简单示例:

```
[Return Address            ]
[Previous Base Pointer     ]
[Cookie XOR Return Address ]
```

在返回时，会检查这个返回值，如果没有变化，就直接返回，否则，就说明有问题发生。这个安全措施并不是用来防止代码崩溃的，而是用来防止注入代码的。如果有人找到了这个缓冲区溢出，还有相应的代码和地址，那么他就可以在相应返回地址构造代码来使程序运行它。

下面的网址提供了更具体的东西：

`http://msdn.microsoft.com/library/?url=/library/en-us/dv_vstechart/html/vctchCompilerSecurityChecksInDepth.asp?frame=true`

### 总结

我可能已经让初学者很困惑，并且高级编程人员会觉得厌烦，然而，确实我很难用很简单的方式来描述这些高级的概念，我已经尽力了。不管你喜欢或不喜欢，请留下你的评论。

上一篇可能很简单，而这篇可能又太深入了，但是没有办法，程序员应该学会自己寻找资料来充实自己。不要把写下的东西当作既定的事实。我们都是人，都会犯错误，并且我们会什么都知道。网站是给每个人发表评论的，因此需要总是抱着怀疑的态度来看，如果你发现了错误，请告诉我，谢谢。
