
# Debug Tutorial(完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-3.html`](https://www.pediy.com/kssd/pediy12/104828-3.html)

## 堆

### 介绍

在这个调试系列的前一章节我们学习了栈。栈是一个临时存储的区域，用于存储局部变量，参数，返回地址和其它一些编译器需要的东西。在这一篇当中，我们将学习用户态下的堆。

### 堆是什么？

堆就是当程序需要时，进程空间中用来提供的一大块内存。当需要时，系统 API 会分配一块内存，然后在每块内存地址前加上一个头部，表示这块地址当前正在使用以及它的大小。然后当需要释放这些内存时，系统会以这个为依据。(全局变量也在堆中)

我现在所说的都是用户态的情况。这一块内存已经分配在进程空间中分配给你的程序来使用。当特定的库开始运行时，比如 `MSVCRT.DLL` 中的 `DllMain()`。`Malloc()`函数就是以这块已经分配好的内存来工作的。

你可以使用 `HeapCreate` 来在你的进程中分配一个内存堆。这个函数分配一个堆段，然后返回这个段号。你的程序可以将这个段作为参数传递给 `HeapAlloc` 来在这个堆中分配内存。这就是我前面说的 `malloc` 的功工作原理，类似于 `Heap*`这样的函数可以完成所有你分配的段中的内存管理，这也是 `malloc` 内部的实现。

另外还有一个函数 `VirtualAlloc`，这个函数可以在程序的更大的范围内分配内存，并且能为内存提交页。这个函数不需要像 `HeapAlloc` 这样函数预先分配好的堆，并且你甚至可以自己指定地址的位置，但并不推荐。这是更加高级的分配内存的方法，但在大部分程序中不使用，也不需要。

所以，你也就知道，用一种函数分配内存，而用另一种函数释放错误将是不允许的(例如用 `LocalAlloc` 分配，而用 `free` 来释放)。因为它们使用了不同的方法来分配，并且他们使用的是不同的堆。内部的代码不知道这种情况，并且总是企图去释放它，从而导致崩溃。这也是为什么任何模块分配了内存都需要释放的原因。如果你在一个 `DLL` 中分配了内存，而在另外一个 `DLL` 中释放它，会怎么样呢？那有时候会起作用，但是如果有一天你用一个调试版本的 `DLL` 替换其中一个，我保证会出现问题。因为调试版本的堆分配和发布版本的堆分配包含的信息不同。这是一个坏习惯，我们应该在同一个模块中分配和释放。

### 分配的内存不是零初始化的？

在前一篇教程中，我提到过，栈中会有一些没有意义的值。栈初始化的时候是全零的，随着使用会逐渐增添一些数据。并且局部变量并不总是会初始化的，因此如果你进行一次函数调用，这些值并不会重置为 0。如果你将一个值弹出栈，栈指针虽然移动了，但是那个地址的值并不会改变，除非你手动做清理。因此，在栈中看到一些无用的数据是很正常的。相同的事情也会发生在堆中。

释放堆中的内存并不会将其中的内容清空为 0，除非在释放内存前你已经这么做了。因此，在你分配了内存之后，得到的指针会指向一些垃圾数据，这也就是为什么我们在分配了内存后通常都需要将之初始化为 0。

在释放内存之前将它置为 0 看起来是很可笑的一件事，但是，如果你的程序包含的了一些敏感的信息，比如密码，这样的工作将会很有意义。因为你肯定不想自己的程序出错之后，栈中或堆中存有用户的密码。

## 堆问题

在这篇文章中，我将会讲述两个与堆相关的常见问题，分别是内存泄露和堆损坏。

### 内存泄露

当你用任务管理器去观察，并发现你的程序的内存使用在不停的增长，这时你要意识到这可能就是内存泄露。内存泄露的意思就是说你分配了内存，但是忘记了释放它。当你得到了你分配的内存的地址，但是你却不小心丢失了它，这就是产生一个泄露。但是程序的内存增长并不就一定代表了内存泄露，有时你的程序确实需要那么多内存。那么，我们该如何确定这种情况呢？

首先，需要做的一件事就是查看任务管理器。如果是很快速的泄露，内存增长速度会非常快；如果是缓慢的泄露，内存会在一段时间内逐渐增长。所以，你需要知道如何去看任务管理器。

1.  **虚拟内存** 这个地方显示了这个进程所需要的内存。如果这个进程空间被页换出到磁盘上了，这个值就是你需要的耗费的分页文件的大小。
2.  **内存使用** 这表示工作集，或者说你所占有的物理内存的量。有人会很奇怪，为什么这个值会比虚拟内存的值还大，这是因为这个值代表的不是只有这个进程使用的内存量，还包含了和其他进程共享的内存量。比如，许多进程都会使用 `kernel32.dll`，这个就是进程之间共享的内存量，如果每个进程都复制一份将会很浪费。

因此，一旦你确定这是一个泄露问题，你就需要找到问题所在。堆的问题是最难追踪的问题之一。如果你确信这是一个泄露问题，那么这些泄露可能都来自同一段代码。即使有很多段代码都在发生泄露问题，那么每一段代码也都会泄露很多次，也就是说泄露的内存中的数据都是相似的。

既然数据都是相似的，你只要找到那么大量分配的内存，然后检查就可以了。下面是一些建议：

1.  一般来说，同一段代码分配的内存大小都是相同的，这对固定大小的结构体来说是可行的。因此你需要找到那些大小相同的内存分配。但这并不总是正确的。
2.  同一段代码分配的内存，其中包含的信息或信息的种类应该是相同的或相似的。也就是说，当你找到了这些内存块，检查它们是否是否有相似的地方。如果你知道其中包含的数据结构或者其他东西，你可以验证大小，匹配它们的数据。这将帮助你缩小所要检查的代码的范围。

我已经写了一个会造成内存泄露的程序。这是一个很简单的程序，但是可以帮助你熟悉堆。

有时候你在程序中发现了内存泄露，这并不意味着那是你写的代码造成的问题，如果你使用了第三方库或 `DLL`，这个问题很可能来自于它们。因此，如果你发现了内存泄露，但是你并没有自己直接分配过内存，那就有可能你使用了别的东西造成了间接的内存分配。比如，你使用 `RegOpenKey` 函数打开一个注册表键，你这道这个函数是如何实现的吗？不知道的话，你就可能会泄露这个句柄，这个句柄会和其他一些内存有关吗？会的。句柄泄露的问题我会在后面的教程中再讲解。

我并不是说注册表键的泄露会导致内存使用量的增加，我只是在阐述一个道理，当你使用第三方模块的时候，可能会间接导致内存泄露，而你是要对此负责的。也就是说，你必须按照函数的使用规则来使用它们，比如最后要使用一些释放资源的函数。

我们开始运行这个程序，并发现内存在一直增长。我们来看看是否能找出源头。首先，找到这个进程的 `PID`，然后使用"`CDB P <PID>`"。当然，你也可以使用 GUI 界面的 `WinDbg`，然后选择你要调试的进程。当我们在进程中中断下来后，要做的第一件事情就是"`!heap`"，显示进程中的堆。

```
0:000> !heap

NtGlobalFlag enables following debugging aids for new heaps:    tail checking

    disable coalescing of free blocks

Index   Address  Name      Debugging options enabled

  1:   00140000                    tail checking free checking validate parameters

  2:   00240000                    tail checking free checking validate parameters

  3:   00250000                    tail checking free checking validate parameters

  4:   00320000                    tail checking free checking validate parameters

0:000>
```

下面，我们来检查到底哪个堆在造成内存泄露。

```
0:000> !heap 00140000

Index   Address  Name      Debugging options enabled

  1:   00140000

    Segment at 00140000 to 00240000 (00100000 bytes committed)

    Segment at 00510000 to 00610000 (00100000 bytes committed)

    Segment at 00610000 to 00810000 (00051000 bytes committed)

  2:   00240000

  3:   00250000

  4:   00320000

0:000> !heap 00240000

Index   Address  Name      Debugging options enabled

  1:   00140000

  2:   00240000

    Segment at 00240000 to 00250000 (00006000 bytes committed)

  3:   00250000

  4:   00320000

0:000> !heap 00250000

Index   Address  Name      Debugging options enabled

  1:   00140000

  2:   00240000

  3:   00250000

    Segment at 00250000 to 00260000 (00001000 bytes committed)

  4:   00320000

0:000> !heap 00320000

Index   Address  Name      Debugging options enabled

  1:   00140000

  2:   00240000

  3:   00250000

  4:   00320000

    Segment at 00320000 to 00330000 (00010000 bytes committed)

    Segment at 00410000 to 00510000 (000ee000 bytes committed)

0:000>
```

我们看看提交内存最多的段(segment)。我们从第一个堆开始。

```
0:000> !heap 00140000 -a

Index   Address  Name      Debugging options enabled

  1:   00140000

    Segment at 00140000 to 00240000 (00100000 bytes committed)

    Segment at 00510000 to 00610000 (00100000 bytes committed)

    Segment at 00610000 to 00810000 (00051000 bytes committed)

    Flags:                 50000062

    ForceFlags:            40000060

    Granularity:           8 bytes

    Segment Reserve:       00400000

    Segment Commit:        00002000

    DeCommit Block Thres: 00000200

    DeCommit Total Thres: 00002000

    Total Free Size:       00000226

    Max. Allocation Size: 7ffdefff

    Lock Variable at:      00140608

    Next TagIndex:         0000

    Maximum TagIndex:      0000

    Tag Entries:           00000000

    PsuedoTag Entries:     00000000

    Virtual Alloc List:    00140050

    UCR FreeList:          00140598

    FreeList Usage:      00040000 00400000 00000000 00000000

    FreeList[ 00 ] at 00140178: 00660118 . 00660118

    Unable to read nt!_HEAP_FREE_ENTRY structure at 00660118

    FreeList[ 12 ] at 00140208: 0023ff78 . 0023ff78

    Unable to read nt!_HEAP_FREE_ENTRY structure at 0023ff78

    FreeList[ 36 ] at 00140328: 0060fe58 . 0060fe58

    Unable to read nt!_HEAP_FREE_ENTRY structure at 0060fe58

    Segment00 at 00140640:

        Flags:            00000000

        Base:             00140000

        First Entry:      00140680

        Last Entry:       00240000

        Total Pages:      00000100

        Total UnCommit:   00000000

        Largest UnCommit:00000000

        UnCommitted Ranges: (0)

    Heap entries for Segment00 in Heap 00140000

        00140000: 00000 . 00640 [01] - busy (640)

        00140640: 00640 . 00040 [01] - busy (40)

        00140680: 00040 . 01818 [07] - busy (1800),

            tail fill - unable to read heap entry extra at 00141e90

        00141e98: 01818 . 00040 [07] - busy (22),

            tail fill - unable to read heap entry extra at 00141ed0

        00141ed8: 00040 . 00020 [07] - busy (5),

            tail fill - unable to read heap entry extra at 00141ef0

        00141ef8: 00020 . 002f0 [07] - busy (2d8),

            tail fill - unable to read heap entry extra at 001421e0

        001421e8: 002f0 . 00330 [07] - busy (314),

            tail fill - unable to read heap entry extra at 00142510

        00142518: 00330 . 00330 [07] - busy (314),

            tail fill - unable to read heap entry extra at 00142840

        00142848: 00330 . 00040 [07] - busy (24),

            tail fill - unable to read heap entry extra at 00142880

        00142888: 00040 . 00040 [07] - busy (24),

            tail fill - unable to read heap entry extra at 001428c0

        001428c8: 00040 . 00028 [07] - busy (10),

            tail fill - unable to read heap entry extra at 001428e8

        001428f0: 00028 . 00058 [07] - busy (40),

            tail fill - unable to read heap entry extra at 00142940

        00142948: 00058 . 00058 [07] - busy (40),

            tail fill - unable to read heap entry extra at 00142998

        001429a0: 00058 . 00060 [07] - busy (44),

            tail fill - unable to read heap entry extra at 001429f8
```



00142a00: 00060 . 00020 [07] - busy (1),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00142a18  

00142a20: 00020 . 00028 [07] - busy (10),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00142a40  

00142a48: 00028 . 00050 [07] - busy (36),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00142a90  

00142a98: 00050 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00142ca0  

00142ca8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00142eb0  

00142eb8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001430c0  

001430c8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001432d0  

001432d8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001434e0  

001434e8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001436f0  

001436f8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00143900  

00143908: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00143b10  

00143b18: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00143d20  

00143d28: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00143f30  

00143f38: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144140  

00144148: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144350  

00144358: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144560  

00144568: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144770  

00144778: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144980  

00144988: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144b90  

00144b98: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144da0  

00144da8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00144fb0  

00144fb8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001451c0  

001451c8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001453d0  

001453d8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001455e0  

001455e8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 001457f0  

001457f8: 00210 . 00210 [07] - busy (1f4),  
&nbsp;&nbsp;&nbsp;&nbsp;tail fill - unable to read heap entry extra at 00145a00  

你可以使用 `Ctrl+Break` 来暂停列表的输出，以方便观看。从后面几行，你会发现这些内存块的大小都是相同的，并且都非常大。这个列表可以如下这样看：  

`<ADDRESS>: <Current Size> . <PREVIOUS Size>`  

我打印出其中一个地址中的值，我们来看看：  

```
0:000> dd 001457f8

001457f8  00420042 001c0700 00006968 00000000
00145808  00000000 00000000 00000000 00000000
00145818  00000000 00000000 00000000 00000000
00145828  00000000 00000000 00000000 00000000
00145838  00000000 00000000 00000000 00000000
00145848  00000000 00000000 00000000 00000000
00145858  00000000 00000000 00000000 00000000
00145868  00000000 00000000 00000000 00000000
```

第一个 `DWORD` 代表的是大小，但是它会分成两部分，低 2 个字节代表现在的大小，高两个字节代表以前的大小。把这个值左移 3 位就能得到真实的大小。所有的内存分配都是以 8 为分配粒度的。`42<<3=210`，也就是十进制的 528。第二个 `DWORD` 是标志(`flags`)，剩下的就是分配给程序的内存。  

```
0:000> dc 001457f8

001457f8  00420042 001c0700 00006968 00000000  B.B.....hi......
00145808  00000000 00000000 00000000 00000000  ................
00145818  00000000 00000000 00000000 00000000  ................
00145828  00000000 00000000 00000000 00000000  ................
00145838  00000000 00000000 00000000 00000000  ................
00145848  00000000 00000000 00000000 00000000  ................
00145858  00000000 00000000 00000000 00000000  ................
00145868  00000000 00000000 00000000 00000000  ................
0:000>
```

像上面这样，使用`DC`命令，以字符方式显示，你会注意到内存中包含`hi`。如果你继续显示这些内存的内容，这些分配的 528 字节的地方都会有`hi`。下面我们来看看下一个堆。  

```
0:000> !heap

NtGlobalFlag enables following debugging aids for new heaps:    tail checking
    disable coalescing of free blocks

Index   Address  Name       Debugging options enabled
  1:   00140000                   tail checking free checking validate parameters
  2:   00240000                   tail checking free checking validate parameters
  3:   00250000                   tail checking free checking validate parameters
  4:   00320000                   tail checking free checking validate parameters

0:000> !heap 00320000 -a

Index   Address  Name       Debugging options enabled
  1:   00140000
  2:   00240000
  3:   00250000
  4:   00320000

    Segment at 00320000 to 00330000 (00010000 bytes committed)
    Segment at 00410000 to 00510000 (000ee000 bytes committed)
    Flags:                50001062
    ForceFlags:           40000060
    Granularity:          8 bytes
    Segment Reserve:      00200000
    Segment Commit:       00002000
    DeCommit Block Thres: 00000200
    DeCommit Total Thres: 00002000
    Total Free Size:      000000b3
    Max. Allocation Size: 7ffdefff
    Lock Variable at:     00320608
    Next TagIndex:        0000
    Maximum TagIndex:     0000
    Tag Entries:          00000000
    PsuedoTag Entries:    00000000
    Virtual Alloc List:   00320050
    UCR FreeList:         00320598
    FreeList Usage:       00000800 00000000 00000000 00000000
    FreeList[ 00 ] at 00320178: 004fdac8 . 004fdac8
    Unable to read nt!_HEAP_FREE_ENTRY structure at 004fdac8
    FreeList[ 0b ] at 003201d0: 0032ffb0 . 0032ffb0
    Unable to read nt!_HEAP_FREE_ENTRY structure at 0032ffb0

    Segment00 at 00320640:
        Flags:           00000000
        Base:            00320000
        First Entry:     00320680
        Last Entry:      00330000
        Total Pages:     00000010
        Total UnCommit:  00000000
        Largest UnCommit:00000000
        UnCommitted Ranges: (0)

    Heap entries for Segment00 in Heap 00320000
        00320000: 00000 . 00640 [01] - busy (640)
        00320640: 00640 . 00040 [01] - busy (40)
        00320680: 00040 . 01818 [07] - busy (1800), 
            tail fill - unable to read heap entry extra at 00321e90
        00321e98: 01818 . 000a0 [07] - busy (88), 
            tail fill - unable to read heap entry extra at 00321f30
        00321f38: 000a0 . 00498 [07] - busy (480), 
            tail fill - unable to read heap entry extra at 003223c8
        003223d0: 00498 . 00098 [07] - busy (80), 
            tail fill - unable to read heap entry extra at 00322460
        00322468: 00098 . 00028 [07] - busy (d), 
            tail fill - unable to read heap entry extra at 00322488
        00322490: 00028 . 000e0 [07] - busy (c8), 
            tail fill - unable to read heap entry extra at 00322568
        00322570: 000e0 . 000e0 [07] - busy (c8), 
            tail fill - unable to read heap entry extra at 00322648
        00322650: 000e0 . 000e0 [07] - busy (c8), 
            tail fill - unable to read heap entry extra at 00322728
        00322730: 000e0 . 000e0 [07] - busy (c8),
```



tail fill - unable to read heap entry extra at `00322808`

`00322810: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003228e8`

`003228f0: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003229c8`

`003229d0: 000e0 . 000e8 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00322ab0`

`00322ab8: 000e8 . 00238 [07] - busy (220),`

tail fill - unable to read heap entry extra at `00322ce8`

`00322cf0: 00238 . 000a8 [07] - busy (90),`

tail fill - unable to read heap entry extra at `00322d90`

`00322d98: 000a8 . 00058 [07] - busy (3e),`

tail fill - unable to read heap entry extra at `00322de8`

`00322df0: 00058 . 00060 [07] - busy (41),`

tail fill - unable to read heap entry extra at `00322e48`

`00322e50: 00060 . 00050 [07] - busy (31),`

tail fill - unable to read heap entry extra at `00322e98`

`00322ea0: 00050 . 00038 [07] - busy (1b),`

tail fill - unable to read heap entry extra at `00322ed0`

`00322ed8: 00038 . 00040 [07] - busy (26),`

tail fill - unable to read heap entry extra at `00322f10`

`00322f18: 00040 . 00030 [07] - busy (11),`

tail fill - unable to read heap entry extra at `00322f40`

`00322f48: 00030 . 00030 [07] - busy (17),`

tail fill - unable to read heap entry extra at `00322f70`

`00322f78: 00030 . 00028 [07] - busy (d),`

tail fill - unable to read heap entry extra at `00322f98`

`00322fa0: 00028 . 00048 [07] - busy (2f),`

tail fill - unable to read heap entry extra at `00322fe0`

`00322fe8: 00048 . 000d0 [07] - busy (b1),`

tail fill - unable to read heap entry extra at `003230b0`

`003230b8: 000d0 . 00080 [07] - busy (61),`

tail fill - unable to read heap entry extra at `00323130`

`00323138: 00080 . 00038 [07] - busy (1c),`

tail fill - unable to read heap entry extra at `00323168`

`00323170: 00038 . 00048 [07] - busy (2d),`

tail fill - unable to read heap entry extra at `003231b0`

`003231b8: 00048 . 00040 [07] - busy (22),`

tail fill - unable to read heap entry extra at `003231f0`

`003231f8: 00040 . 00030 [07] - busy (17),`

tail fill - unable to read heap entry extra at `00323220`

`00323228: 00030 . 00028 [07] - busy (e),`

tail fill - unable to read heap entry extra at `00323248`

`00323250: 00028 . 00168 [07] - busy (149),`

tail fill - unable to read heap entry extra at `003233b0`

`003233b8: 00168 . 00058 [07] - busy (39),`

tail fill - unable to read heap entry extra at `00323408`

`00323410: 00058 . 00038 [07] - busy (1b),`

tail fill - unable to read heap entry extra at `00323440`

`00323448: 00038 . 00060 [07] - busy (43),`

tail fill - unable to read heap entry extra at `003234a0`

`003234a8: 00060 . 00030 [07] - busy (12),`

tail fill - unable to read heap entry extra at `003234d0`

`003234d8: 00030 . 00030 [07] - busy (18),`

tail fill - unable to read heap entry extra at `00323500`

`00323508: 00030 . 00038 [07] - busy (1e),`

tail fill - unable to read heap entry extra at `00323538`

`00323540: 00038 . 00028 [07] - busy (c),`

tail fill - unable to read heap entry extra at `00323560`

`00323568: 00028 . 00030 [07] - busy (14),`

tail fill - unable to read heap entry extra at `00323590`

`00323598: 00030 . 00028 [07] - busy (f),`

tail fill - unable to read heap entry extra at `003235b8`

`003235c0: 00028 . 00030 [07] - busy (18),`

tail fill - unable to read heap entry extra at `003235e8`

`003235f0: 00030 . 00040 [07] - busy (28),`

tail fill - unable to read heap entry extra at `00323628`

`00323630: 00040 . 00040 [07] - busy (27),`

tail fill - unable to read heap entry extra at `00323668`

`00323670: 00040 . 00038 [07] - busy (19),`

tail fill - unable to read heap entry extra at `003236a0`

`003236a8: 00038 . 00030 [07] - busy (17),`

tail fill - unable to read heap entry extra at `003236d0`

`003236d8: 00030 . 00050 [07] - busy (34),`

tail fill - unable to read heap entry extra at `00323720`

`00323728: 00050 . 00030 [07] - busy (11),`

tail fill - unable to read heap entry extra at `00323750`

`00323758: 00030 . 00030 [07] - busy (14),`

tail fill - unable to read heap entry extra at `00323780`

`00323788: 00030 . 00068 [07] - busy (4a),`

tail fill - unable to read heap entry extra at `003237e8`

`003237f0: 00068 . 00818 [07] - busy (800),`

tail fill - unable to read heap entry extra at `00324000`

`00324008: 00818 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003240e0`

`003240e8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003241c0`

`003241c8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003242a0`

`003242a8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324380`

`00324388: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324460`

`00324468: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324540`

`00324548: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324620`

`00324628: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324700`

`00324708: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003247e0`

`003247e8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003248c0`

`003248c8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003249a0`

`003249a8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324a80`

`00324a88: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324b60`

`00324b68: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324c40`

`00324c48: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324d20`

`00324d28: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324e00`

`00324e08: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324ee0`

`00324ee8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00324fc0`

`00324fc8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `003250a0`

`003250a8: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00325180`

`00325188: 000e0 . 000e0 [07] - busy (c8),`

tail fill - unable to read heap entry extra at `00325260`



在这个堆中，可以看到很多 `<address>:e0.e0` 这样的分配，这很有可能是一个泄露。下面我们来看看：

```
0:000> dc 00325188

00325188  001c001c 00180700 66647361 61667361  ........asdfasfa
00325198  73666473 73666461 73666461 61666164  sdfsadfsadfsdafa
003251a8  61736673 73616664 61736664 73616664  sfsadfasdfsadfas
003251b8  66736166 73666473 66736661 66647361  fasfsdfsafsfasdf
003251c8  00617364 baadf00d baadf00d baadf00d  dsa.............
003251d8  baadf00d baadf00d baadf00d baadf00d  ................
003251e8  baadf00d baadf00d baadf00d baadf00d  ................
003251f8  baadf00d baadf00d baadf00d baadf00d  ................
```

除了 `!heap` 命令，你还可以从所分配的堆的开始地址一直使用 `DC` 命令来查看，是否有一些固定的模式。

`01c<<3=e0`，十进制的 `224`。下面我们来看看源代码：

```
char *p, *x;
while(1)
{
    p = malloc(200);
    strcpy(p, "asdfasfasdfsadfsadfsdafasfsadfasdfsadfasfasfsdfsafsfasdfdsa");
    x = LocalAlloc(LMEM_ZEROINIT, 500);
    strcpy(x, "hi");
    Sleep(1);
}
```

很显然，这里造成了一个快速的泄露。你们会注意到，分配的是 `224` 字节，而不是我们指定的 `200`。这是因为分配的大小包括了前面说的 `2` 个 `DWORD` 头，还有必须是 `8` 字节为边界的。如果你得到了分配给你的内存地址，然后将它减去 `8`，你就可以得到头部信息了。将分配的这个大小左移 `3` 位，然后加上这个地址，你就可以得到下一个分配的内存块。下面举例说明：

```
0:000> dc 0325188 + e0

00325268  001c001c 00180700 66647361 61667361  ........asdfasfa
00325278  73666473 73666461 73666461 61666164  sdfsadfsadfsdafa
00325288  61736673 73616664 61736664 73616664  sfsadfasdfsadfas
00325298  66736166 73666473 66736661 66647361  fasfsdfsafsfasdf
003252a8  00617364 baadf00d baadf00d baadf00d  dsa.............
003252b8  baadf00d baadf00d baadf00d baadf00d  ................
003252c8  baadf00d baadf00d baadf00d baadf00d  ................
003252d8  baadf00d baadf00d baadf00d baadf00d  ................
0:000>
```

我不会一个一个区比较堆标志，因为这是没有必要的。我觉得最重要的标志就当前这个内存块是否分配了。也就是说，当你使用 `!heap <heap> -a` 命令之后，看到了 `busy`，就表示这块内存没有被释放，而 `free` 则表示这块内存已经被释放了。分配之后标志为 `00180700`，那么释放之后就是 `00180400`，你可以写一个程序测试一下，但是要注意，你释放之后没有别的线程启动或其他的内存分配。

### 私有堆和全局堆

上面我已经介绍了 `!heap` 命令，但是要注意的是，它不会显示全局堆。如果你有一个全局变量，`!heap` 命令显示的堆中不会包含这个全局变量。全局堆也不能摧毁，而私有堆可以。

全局堆还会有损坏的问题，我们将在下一个话题讨论。

### 跟踪内存分配

还有另外一种跟踪内存泄露的方法，那就是自己写一个内存分配函数。如下：

```
PVOID MyAllocationRoutine(DWORD dwSize)
{
  PVOID pMem = malloc(dwSize + sizeof(_DEBUG_STRUCTURE));
  if(pMem)
  {
    _DEBUG_STRUCTURE *pDebugStruc = (_DEBUG_STRUCTURE *)pMem;
    /* Fill In Your Debug Information Here */
    /* Make Sure You Give the Application the memory AFTER your debug structure */
    pMem = pDebugStruc + 1;
  }
  return pMem;
}
```

你只要将你自己的头部加到所有的分配中去就可以了。这也是类似于 `boundschecker` 这样的程序运行的原理。它们能够将你的分配替换，然后找出是谁分配的这块内存，并跟踪它，你也可以这样做。你甚至可以创建一个全局变量，然后把所有的内存加入到链表中去，然后自己写一个调试扩展将其中的信息都显示出来。以后的章节我们将讲述调试扩展的内容。

所以，你可以看到，你可以创建自己的函数来分配内存，并向其中加入任何信息。你甚至可以使用 `#define` 来讲一些函数定义为你自己想要的，比如 `LocalAlloc` 和 `malloc`。你还可以在注册表中增加一些标志来做这些事情。另外你还要记住写一个相应的释放函数。

### 堆损坏

堆损坏和内存损坏一样，基本上就是说所写的边界超过了给你分配的大小。造成堆损坏的最常见的原因就是使用了错误的方式释放内存。比如，如果你使用 `malloc` 分配内存，而用 `LocalFree` 释放内存。由于它们使用不同的堆，并且甚至使用底层不同的方法来分配和跟踪内存，而这些释放函数都会用它们自己的算法来释放内存，并且不会对所要释放的堆做一些验证，这就会造成这里所说的堆损坏问题。上一节我们用 `malloc` 创建了自己的内存分配函数，所以为了避免这样的问题，我们应该也创建自己的相应正确的释放函数。

导致这个问题的其他原因，也可能是你就是超过了所拥有的内存边界，比如向内存中写入了过多的数据，以至于超过边界，或者是向随机的一个地址写入了数据。堆损坏比内存泄露要更难跟踪。”跟踪分配的过程”这个策略并不会给你带来任何帮助，特别是当你重新编译之后。因为了你可以能改变了分配的大小，这就可能不会再发生堆损坏的问题，因为这次堆的大小可能满足你的条件。

我写了一个有一些堆问题的程序。堆问题并不会很快的显现，这通常需要经过一段时间。当程序中的其他部分使用损坏的内存，或释放或分配另一个变量的时候，就可能显现出问题。

我运行这个程序之后，出现了几个对话框，告诉你不正确的内存引用。下面就该我们的调试器登场了。

```
C:\programs\DirectX\Games\src\Games\temp\bin>cdb temp

Microsoft (R) Windows Debugger  Version 6.3.0005.1
Copyright (c) Microsoft Corporation. All rights reserved.

CommandLine: temp
Symbol search path is: SRV*c:\symbols*http://msdl.microsoft.com/download/symbols

Executable search path is:
ModLoad: 00400000 00404000   temp.exe
ModLoad: 77f50000 77ff7000   ntdll.dll
ModLoad: 77e60000 77f46000   C:\WINDOWS.0\system32\kernel32.dll
ModLoad: 77c10000 77c63000   C:\WINDOWS.0\system32\MSVCRT.dll

(a20.710): Break instruction exception - code 80000003 (first chance)
eax=00241eb4 ebx=7ffdf000 ecx=00000004 edx=77f51310 esi=00241eb4 edi=00241f48
eip=77f75a58 esp=0012fb38 ebp=0012fc2c iopl=0          nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000          efl=00000202

ntdll!DbgBreakPoint:
77f75a58 cc            int     3
0:000> g

(a20.710): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.

eax=61736664 ebx=00000004 ecx=73616664 edx=00142ab8 esi=00142ab8 edi=00140000
eip=77f8452d esp=0012f7e4 ebp=0012f9fc iopl=0          nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00010246

ntdll!RtlAllocateHeapSlowly+0x6bd:
77f8452d 8901           mov     [ecx],eax            ds:0023:73616664=????????
0:000>
```

这里我们得到了 `first chance` 异常，这通常意味着，如果我键入 `g` 命令，程序将会继续运行。如果我们得到了 `second chance` 异常，程序将会出错暂停。我们看到，这是内存损坏的标志。记得我们在第一篇教程中说的，遇到这样的问题，我们要做的第一件事就是找到为什么我们会使用这块内存，谁在使用它，和它在哪里。那该怎么办呢？那就是我们的老朋友了，栈回溯。

```
0:000> kb

ChildEBP RetAddr  Args to Child
0012f9fc 77f9d959 00140000 50140169 00000006 ntdll!RtlAllocateHeapSlowly+0x6bd
0012fa80 77f83eb1 00140000 50140169 00000006 ntdll!RtlDebugAllocateHeap+0xaf
0012fcac 77f589f2 00140000 40140068 00000006 ntdll!RtlAllocateHeapSlowly+0x41
0012fee4 77e7a6d4 00140000 40140068 00000006 ntdll!RtlAllocateHeap+0xe44
0012ff30 00401024 00000040 00000006 00000000 kernel32!LocalAlloc+0x58
0012ff4c 0040113b 00000001 00322470 00322cf8 temp!main+0x24
0012ffc0 77e814c7 00000000 00000000 7ffdf000 temp!mainCRTStartup+0xe3
0012fff0 00000000 00401058 00000000 78746341 kernel32!BaseProcessStart+0x23
0:000>
```

可以看到，我们在分配内存，并在 `NTDLL` 中发生了错误。我们再继续看，被引用的内存在哪里。

```
0:000> u eip - 20

ntdll!RtlAllocateHeapSlowly+0x69d:
77f8450d 058845b356     add     eax,0x56b34588
77f84512 8b7de4         mov     edi,[ebp-0x1c]
77f84515 57             push    edi
77f84516 e85eeaffff     call    ntdll!RtlpUpdateIndexRemoveBlock (77f82f79)
77f8451b 8b4608         mov     eax,[esi+0x8]
77f8451e 89855cffffff   mov     [ebp-0xa4],eax
77f84524 8b4e0c         mov     ecx,[esi+0xc]
77f84527 898d58ffffff   mov     [ebp-0xa8],ecx
0:000> u

ntdll!RtlAllocateHeapSlowly+0x6bd:
77f8452d 8901           mov     [ecx],eax
```

看上去像是某种链表或类似的数据结构。我们看到 `ECX` 是作为一个指针在使用，类似于这样的语句，`DWORD *pECX; *pECX=EAX;`。因此我们需要找出 `ECX` 的值从哪来。看到了 `ECX, [ESI+0ch]` 吗？等价于下面这条语句：

`DWORD *pECX, *pESI; pECX = pESI[12/4];`

记住，在汇编语言中没有类型，因此数组都是以字节来索引的，而不是以数据类型的大小来索引的。因此，我们来 `Dump` 出 `[ESI+C]` 看看。

```
0:000> dc esi + c

00142ac4  73616664 66736166 73666473 66736661  dfasfasfsdfsafsf
00142ad4  66647361 00617364 feeefeee feeefeee  asdfdsa.........
00142ae4  feeefeee feeefeee feeefeee feeefeee  ................
00142af4  feeefeee feeefeee feeefeee feeefeee  ................
00142b04  feeefeee feeefeee feeefeee feeefeee  ................
00142b14  feeefeee feeefeee feeefeee feeefeee  ................
00142b24  feeefeee feeefeee feeefeee feeefeee  ................
00142b34  feeefeee feeefeee feeefeee feeefeee  ................
```

错误发生的地方如下：

`77f8452d 8901  mov  [ecx],eax  ds:0023:73616664=????????`

现在就很明了了，这是一个字符串，我们在程序里找找分配这段内存的地方。

```
x = LocalAlloc(LMEM_ZEROINIT, 5);
strcpy(x, "asdfasfasdfsadfsadfsdafasfsadfasdfsadfasfasfsdfsafsfasdfdsa");
p = LocalAlloc(LMEM_ZEROINIT, 6);
strcpy(p, "hi");
LocalFree(x);
free(p);
```

可以看到，我们写入的数据明显超过了 `5` 个字节。这是很简单的一个例子。有时，可能就是多写了一个字节，你必须一点一点往回找，这可能是字符串的 `null` 结尾，很多时候你都会忘了要多留一个字节给 `null`。很多时候，我们可能不会注意到这些问题，因为内存分配是以 `8` 为粒度的。但是有些时候，追踪这些问题可能就是恶梦，因为内存一旦超出范围，罪魁祸首可能就逃之夭夭。

另外一种方法就是一点一点的跟进程序，如果超出范围的地方没有改变，跟进和检查它很简单。你可以一步一步减小范围，这些都是假定我们遇到的都是简单问题，复杂的话，你必须检查各个地方。

你还可以使用断点来检查。`ba r1 xxxx` 的意思是 `如果有人想读或写这个地址的话就中断下来`。如果地址是常量的话，这将会很有用。另外，你还可以降低发生问题的可能性，比如开启 `global flags` 等等。



第一段中说的检查内存溢出的代码不能用来检查内存破坏，这么说是对的，也是错的。在当前这样的情况，你不要期望它能帮你找出内存破坏。但是你做一些调整之后，你可以在分配的内存的开始和结束处放入一些数据，然后你可以释放这些数据，并检查这些数据来验证是否有问题出现，如果发生改变了，就可能有问题出现。这当然是假设损坏是连续的，并且会触及到边界，并且不是随机的一个内存地址。

## 其他工具

还有其他一些工具可以帮助你跟踪内存损坏和泄漏。

### Performance Monitor

这是 Windows 自带的工具`perfmon`。它可以帮助你监视和记录系统性能，你可以用它来跟踪进程中内存的使用轨迹。这个也可以用来发现一些小的泄漏情况。

### Bounds Checker

这是我上面提到的一个工具。它会在你的程序结束的时候，帮助你跟踪没有释放的内存。当然还可以发现其他种类的泄漏，甚至可以发现内存损坏。这个工具使用非常简单，并会告诉你哪段代码分配了内存，但是并没有释放，还会告诉你有多少内存没有释放。

### Global Flags

注册表中的一些选项可以开启堆检查和堆验证的功能。调试工具集中有一个`gflags`工具可以帮助设置这些标志。以后的教程中我可能会讲到这个工具。

### QuickView:System Explorer

这是我写的一个工具，工作的 `Windows2000` 或更高版本上。它不会得到实时的数据，它只是帮你检查系统地不同方面来帮助你跟踪问题。你可以去这个页面下载安装程序。

## 总结

这篇教程主要讲述了一些用户太下关于内存泄漏和内存损坏的问题，其中还教了你一些解决问题的方法。
