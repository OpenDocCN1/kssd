
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
