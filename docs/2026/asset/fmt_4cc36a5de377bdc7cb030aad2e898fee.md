
此时堆块已经分配完毕，对应的分配地址位于`0x007104a0`，`0x007104a0`是堆块数据的起始地址，并非堆头信息的起始地址。对于已经分配的堆块，开头有8字节的`HEAP_ENTRY`结构，因此`heap`的`HEAP_ENTRY`结构位于`0x007104a0 - 8 = 0x710498`。

![堆块信息图](https://bbs.kanxue.com/upload/attach/201907/823237_6G2Y98UQQUXW3W4.png)

在`windbg`上查看两个堆块的信息，这两个堆块目前处于占用状态，共有`0x10`大小空间：

```
0:000> !heap -p -a 0x710498

    address 00710498 found in

    _HEAP @ 710000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        00710498 0005 0000  [00]   007104a0    00010 - (busy)
```

在`windbg`中，使用`!heap`查看`HeapCreate`创建的整个堆块信息，可以发现堆块`heap`后面还有一个空闲堆块`0x007104c0`：

```
0:000> !heap

        Heap Address      NT/Segment Heap

              560000              NT Heap

              800000              NT Heap

              710000              NT Heap

0:000> !heap -a 710000

HEAPEXT: Unable to get address of ntdll!RtlpHeapInvalidBadAddress.

Index   Address  Name      Debugging options enabled

  3:   00710000 

    Segment at 00710000 to 00720000 (00001000 bytes committed)

    Flags:                40001064

    ForceFlags:           40000064

    Granularity:          8 bytes

    Segment Reserve:      00100000

    Segment Commit:       00002000

    DeCommit Block Thres: 00000200

    DeCommit Total Thres: 00002000

    Total Free Size:      00000164

    Max. Allocation Size: 7ffdefff

    Lock Variable at:     00710248

    Next TagIndex:        0000

    Maximum TagIndex:     0000

    Tag Entries:          00000000

    PsuedoTag Entries:    00000000

    Virtual Alloc List:   0071009c

    Uncommitted ranges:   0071008c

            00711000: 0000f000  (61440 bytes)

    FreeList[ 00 ] at 007100c0: 007104c8 . 007104c8  

        007104c0: 00028 . 00b20 [104] - free

    Segment00 at 00710000:

        Flags:           00000000

        Base:            00710000

        First Entry:     00710498

        Last Entry:      00720000

        Total Pages:     00000010

        Total UnCommit:  0000000f

        Largest UnCommit:00000000

        UnCommitted Ranges: (1)

    Heap entries for Segment00 in Heap 00710000

         address: psize . size  flags   state (requested size)

        00710000: 00000 . 00498 [101] - busy (497)

        00710498: 00498 . 00028 [107] - busy (10), tail fill //heap的占用堆块

        007104c0: 00028 . 00b20 [104] free fill  //空闲堆块

        00710fe0: 00b20 . 00020 [111] - busy (1d)

        00711000:      0000f000      - uncommitted bytes.
```

在复制字符串的时候，原本只有`0x10`大小的堆块，填充过多的字符串时就会覆盖到下方的空闲堆块`007104c0`。在复制前，`007104c0`空闲堆块的`HEAP_FREE_ENTRY`结构数据如下：

```
0:000> dt _HEAP_FREE_ENTRY 0x007104c0

ntdll!_HEAP_FREE_ENTRY

   +0x000 HeapEntry        : _HEAP_ENTRY

   +0x000 UnpackedEntry    : _HEAP_UNPACKED_ENTRY

   +0x000 Size             : 0x6298

   +0x002 Flags            : 0x16 ''

   +0x003 SmallTagIndex    : 0xac ''

   +0x000 SubSegmentCode   : 0xac166298

   +0x004 PreviousSize     : 0xcfb9

   +0x006 SegmentOffset    : 0 ''

   +0x006 LFHFlags         : 0 ''

   +0x007 UnusedBytes      : 0 ''

   +0x000 ExtendedEntry    : _HEAP_EXTENDED_ENTRY

   +0x000 FunctionIndex    : 0x6298

   +0x002 ContextValue     : 0xac16

   +0x000 InterceptorValue : 0xac166298

   +0x004 UnusedBytesLength : 0xcfb9

   +0x006 EntryOffset      : 0 ''

   +0x007 ExtendedBlockSignature : 0 ''

   +0x000 Code1            : 0xac166298

   +0x004 Code2            : 0xcfb9

   +0x006 Code3            : 0 ''

   +0x007 Code4            : 0 ''

   +0x004 Code234          : 0xcfb9

   +0x000 AgregateCode     : 0x0000cfb9`ac166298

   +0x008 FreeList         : _LIST_ENTRY [ 0x7100c0 - 0x7100c0 ]
```

```
0:000> dt _LIST_ENTRY 0x007104c0+8

ntdll!_LIST_ENTRY

 [ 0x7100c0 - 0x7100c0 ]

   +0x000 Flink            : 0x007100c0 _LIST_ENTRY [ 0x7104c8 - 0x7104c8 ]

   +0x004 Blink            : 0x007100c0 _LIST_ENTRY [ 0x7104c8 - 0x7104c8 ]
```

覆盖后，`0x007104c0`空闲块的`HEAP_FREE_ENTRY`结构数据如下：

```
0:000> g

(2c08.234): Access violation - code c0000005 (!!! second chance !!!)

eax=007e04a0 ebx=007e0498 ecx=41414141 edx=007e0260 esi=007e04b8 edi=007e0000

eip=7775919d esp=0019fdb0 ebp=0019fea8 iopl=0         nv up ei ng nz na po cy

cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00010283

ntdll!RtlpFreeHeap+0x5bd:

7775919d 8b11            mov     edx,dword ptr [ecx]  ds:002b:41414141=????????

0:000> dt _HEAP_FREE_ENTRY 0x007104c0

ntdll!_HEAP_FREE_ENTRY

   +0x000 HeapEntry        : _HEAP_ENTRY

   +0x000 UnpackedEntry    : _HEAP_UNPACKED_ENTRY

   +0x000 Size             : ??

   +0x002 Flags            : ??

   +0x003 SmallTagIndex    : ??

   +0x000 SubSegmentCode   : ??

   +0x004 PreviousSize     : ??

   +0x006 SegmentOffset    : ??

   +0x006 LFHFlags         : ??

   +0x007 UnusedBytes      : ??

   +0x000 ExtendedEntry    : _HEAP_EXTENDED_ENTRY

   +0x000 FunctionIndex    : ??

   +0x002 ContextValue     : ??

   +0x000 InterceptorValue : ??

   +0x004 UnusedBytesLength : ??

   +0x006 EntryOffset      : ??

   +0x007 ExtendedBlockSignature : ??

   +0x000 Code1            : ??

   +0x004 Code2            : ??

   +0x006 Code3            : ??

   +0x007 Code4            : ??

   +0x004 Code234          : ??

   +0x000 AgregateCode     : ??

   +0x008 FreeList         : _LIST_ENTRY

Memory read error 007104c0
```

整个空闲堆头信息都被覆盖了，包括最后的空闲链表中的前后向指针都被改成了`0x41414141`。后面调用`HeapFree`释放堆块的时候，就会将`buf2`和后面的空闲堆块`0x007104c0`合并，修改两个空闲堆块的前后向指针就会引用`0x41414141`，最后造成崩溃。

如果把上面释放堆块的操作换成分配堆块`HeapAlloc()`，也会导致崩溃。因为在分配堆块的时候会去遍历空闲链表指针，会造成地址引用异常。当内存中已经分配多个堆块的时候，可能覆盖到的就是已经分配到的堆块，此时可能就是覆盖`HEAP_ENTRY`结构，而不是`HEAP_FREE_ENTRY`结构。

![堆溢出原理图](https://bbs.kanxue.com/upload/attach/201907/823237_7JVGRQPDWBKQ9P8.png)

堆溢出原理图

### 0x03 堆调试技巧

微软提供了一些调试选项用于辅助堆调试，可以通过`windbg`提供的`gflag.exe`或`!gflag`命令来设置：

- `htc`：堆尾检查，是否发生溢出
- `hfc`：堆释放检查
- `hpc`：堆参数检查
- `hpa`：启用页堆
- `htg`：堆标志
- `ust`：用户态栈回溯

对`heapoverflow.exe`添加堆尾检查和页堆，去掉堆标志：

```
gflags.exe -i F:\vulns\Release\heapoverflow +htc +hpa +htg
```

#### 堆尾检查

主要是在每个堆块的尾部，用户数据之后添加8字节，通常是连续的2个`0xabababab`，该数据段被破坏就可能发生了溢出。

对`heapoverflow.exe`开启`hpc`和`htc`，用`windbg`加载对`heapoverflow`程序。附加进程无法在堆尾添加额外标志，使用以下命令开启堆尾检查和堆参数检查：

```
0:000> !gflag +htc +hpc

Current NtGlobalFlag contents: 0x00000070

    htc - Enable heap tail checking

    hfc - Enable heap free checking

    hpc - Enable heap parameter checking
```

```
0:000:x86> g

HEAP[heapoverflow.exe]: Heap block at 001E0498 modified at 001E04B0 past requested size of 10

(13d0.3c9c): WOW64 breakpoint - code 4000001f (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

ntdll_77710000!RtlpBreakPointHeap+0x13:

777f07c7 cc              int     3
```

执行命令`g`后，按下回车键程序会断下来：

```
0:000:x86> kb

 # ChildEBP RetAddr  Args to Child              

00 0019fd18 777dd85b 00000000 001e0000 001e0498 ntdll_77710000!RtlpBreakPointHeap+0x13

01 0019fd30 77793e9c 001e0000 00000000 77786780 ntdll_77710000!RtlpCheckBusyBlockTail+0x1a2

02 0019fd4c 777ef9f1 7772e4dc 9ceeef49 001e0000 ntdll_77710000!RtlpValidateHeapEntry+0x633d9

03 0019fda4 7775991d 001e04a0 9ceeec45 001e0498 ntdll_77710000!RtlDebugFreeHeap+0xbf

04 0019fea8 77758b98 001e0498 001e04a0 001e04c1 ntdll_77710000!RtlpFreeHeap+0xd3d

*** WARNING: Unable to verify checksum for F:\vulns\Release\heapoverflow.exe

05 0019fefc 00401094 001e0000 00000000 001e04a0 ntdll_77710000!RtlFreeHeap+0x758

WARNING: Stack unwind information not available. Following frames may be wrong.

06 001e0000 01000709 ffeeffee 00000000 001e00a4 heapoverflow+0x1094

07 001e0004 ffeeffee 00000000 001e00a4 001e00a4 0x1000709

08 001e0008 00000000 001e00a4 001e00a4 001e0000 0xffeeffee
```

```
HEAP[heapoverflow.exe]: Heap block at 001E0498 modified at 001E04B0 past requested size of 10
```

上面一句调试输出信息的意思是，在大小为`0x10`的堆块`0x001E0498`的`0x001E04B0`覆盖破坏了。`0x10`大小的空间加上堆头的`8`字节一共`0x18`字节，`0x001E04B0 - 0x001E0498 = 0x18`，也就是说`0x001E04B0`位于堆块数据的最后一个字节上。基于上面的信息，可以分析出程序主要是因为向`0x10`的堆块中复制过多数据导致的堆溢出。

#### 页堆

在调试漏洞的时候，经常需要定位导致漏洞的代码和函数，比如导致堆溢出的字节复制指令`rep movsz`等。前面的堆尾检查方式主要是堆被破坏的场景，不利于定位导致漏洞的代码。为此，引入了页堆的概念。开启后，会在堆块中增加不可访问的栅栏页，溢出覆盖到栅栏页就会触发异常。

开启页堆：
