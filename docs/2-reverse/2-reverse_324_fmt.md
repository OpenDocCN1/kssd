
# Debug Tutorial(完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-5.html`](https://www.pediy.com/kssd/pediy12/104828-5.html)

## 句柄泄露

### 介绍

欢迎来到这个调试教程系列的第 5 篇。在这篇文章，我们会介绍 Windows 中的句柄，什么是句柄以及怎么调试句柄泄露。在读这篇文章之前，我希望你对前 4 篇文章掌握的都还不错。这一系列的文章并不是在重复 `WinDbg`/`NTSD` 的帮助文件，而是介绍实实在在的问题，它们是怎么一回事，以及如何解决它们。

### 什么是句柄

对于一个程序来说，句柄就是设备，文件，或其它一些系统对象或资源的实例。程序可以通过 `CreateFile`，`RegOpenKey` 等类似函数来创建一个资源的实例，然后使用这个句柄来进行接下来对这些资源的操作。

一般来说，你不用关心这个句柄值是多少。那么句柄到底是什么，代表什么？

为了理解的更清晰，我们来看看 `CreateFile` 的流程。如果你调试 `CreateFile`，你会注意到它最终会调用 `NtCreateFile`。

`kernel32!CreateFileW+0x34a:`

```
77e7b24c  ff150810e677  call  dword ptr [kernel32!_imp__NtCreateFile (77e61008)]
```

```
0:000> kb
ChildEBP  RetAddr  Args to Child
0012f728  77e7b4a3  000007c4 80000000 00000000 kernel32!CreateFileW+0x40e
0012f74c  00401ba9  00406760 80000000 00000000 kernel32!CreateFileA+0x2e
```

`NtCreateFile` 会导致一个内核调用。这个调用是如何实现的要看系统，可能是 `sysenter` 或一个软中断 `int 2eh`。不管哪种方法，这个调用都会进入内核，并会被系统分配到对应的驱动。然后，在内核中就会创建了一个对象来代表这个资源，在这里就是文件。如果你去观察 `NtCreateFile` 的参数，你会发现第一个参数就是将会返回的句柄。我们来看看第一个参数：

```
eax=0012f730 ebx=00000000 ecx=80100080 edx=00200000 esi=77f58a3e edi=00000000
eip=77e7b24b esp=0012f69c ebp=0012f728 iopl=0 nv up ei ng nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=0038 gs=0000 efl=00000282
```

`kernel32!CreateFileW+0x349:`

```
77e7b24b  50             push    eax
0:000>
```

可以看到，第一个参数是一个地址，就是函数中的一个局部变量。返回后，我们可以检查它的值。

```
0:000> dd 0012f730
0012f730  000007c4
```

返回的值是 `7c4h`，并不是程序中指向任何一个地址的指针，更不是内核内存中的指针。为了更深入，有一个调试器命令可以显示句柄信息，就是 `!handle`。

```
0:000> !handle 7c4 ff
Handle 7c4
  Type          File
  Attributes    0
  GrantedAccess 0x120089:
         ReadControl,Synch
         Read/List,ReadEA,ReadAttr
  HandleCount   2
  PointerCount  3
  No Object Specific Information available
```

我们可以看到，这确实是一个文件对象，还显示了它的属性和访问权限。但是我们并不知道这个句柄到底指向哪个文件，而我们想知道的就是这个信息。我们看不到是有原因的，但在我告诉你原因之前，我们来看看系统到底干了什么。

### 句柄值代表什么？

为了给你展示这个东西，我们需要内核调试器。我在另外一台 `Windows 2000` 的机器上调试过一个进程。我需要打开记事本，然后需要获得这个进程中的一些句柄。

注意：在 `Windows XP/2003` 中，调试器会显示系统的更多信息。比如，对一个线程使用 `!handle` 将会显示线程的入口函数，而 `Windows 2000` 不会显示。如果你想将一个线程和一个句柄匹配，这将非常有用，特别是当句柄已经泄露了或线程也不再运行了。教程中我还介绍过 `dt` 命令，`Windows 2000` 有时候不能正确显示一些结构(比如系统结构 `_EPROCESS`)。这可能是因为 `DBGs` 和 `PDBs` 在 `Windows 2000` 上被处理的方法不同，或者是信息丢失了。在 `Windows XP/2003` 上调试还有更多其他的好处。

因此，我选择了一个文件，并在 `CreateFile` 上下断点。当我选择完这个文件之后，断点触发，然后我再返回地址上下一个断点。然后，我打印出 `EAX` 中的值来看看句柄信息。

```
0:000> !handle eax ff
Handle 120
  Type          File
  Attributes    0
  GrantedAccess 0x120089:
         ReadControl,Synch
         Read/List,ReadEA,ReadAttr
  HandleCount   2
  PointerCount  3
```

这和上面一样，只是显示了它是一个文件。下面我们来看看第一个秘密。这些句柄都是针对特定进程的。`120h` 这个值只在这个进程空间中有用。如果你在另外一个进程中使用这个句柄值，它可能并不存在，也可能代表了另外一个不同的对象。这些句柄和窗口句柄不一样，它们的作用范围是进程。

句柄还是内核对象的一个代表，这意味着每个进程在内核中都有一个句柄表，其中每一项都指向内核的一块内存。这是第二个秘密，也是我为什么要使用内核调试器的原因。现在，我们在内核中中断下来，并来看看这个句柄值。

在内核调试器中，我们使用 `!process 0 0`，列举所有的进程，然后我们使用 `!handle` 命令，语法会有一些不一样，为了列举正确的句柄，我们需要指定相应的进程。

```
kd> !process 0 0
**** NT ACTIVE PROCESS DUMP ****
...
PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8
    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.
    Image: notepad.exe
```

```
kd> !handle 120 ff fcc77200
processor number 0
PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8
    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.
    Image: notepad.exe
Handle Table at e1e5f000 with 74 Entries in use
0120: Object: fcd32448  GrantedAccess: 00120089
Object: fcd32448  Type: (fced7c40) File
    ObjectHeader: fcd32430
        HandleCount: 1  PointerCount: 1
        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}
```

第一个加粗的数字是进程对象，它是内存中的一个结构体，包含了进程中的一些信息。既然句柄是对应于特定进程的，我们就要告诉 `!handle` 它对应于哪个进程。`"ff"` 就是说显示可能的所有信息，帮助文件中有具体的解释。我总是使用 `"ff"`，因为我需要所有的信息。

第二个加粗的地址是进程中的句柄表，这块内存代表进程中所有的句柄项。可以看到现在有 74 个句柄被打开。最后一个加粗的地址是对象的地址，这是所有数据的来源。`"Name"` 属性告诉我们文件名。为什么我们不能在用户态调试器下得到这些信息呢？

### 在用户态下显示句柄信息

很明显，这个表是在内核中的，所以你不能直接在用户态下看到。`NT` 系统提供了一些 `API` 可以让你得到这些句柄的信息。如果你看了第 3 篇文章，我写的那个 `QuickView:System Explorer` 就可以完成这样的功能，并显示其中的一些信息，使用的 `API` 是 `NtQueryObject`。

这个 `API` 的不足之处就是它可能会使你的进程挂住。这是因为有一些对象是以 `SYNC` 的方式打开的。系统中的管道就是这样的情况，如果你请求它们的信息，你将无限期挂住。为了防止出现这样的情况，调试器以及我的程序都不会去想请求这样的对象，以防死锁。在我的程序中，我会检查 `SYNC` 标志，我还发现其他一些不会挂起的访问权限。

`SysInternals` 中的 `HANDLE.EXE` 工具可以显示所有的信息，那么，它是怎么做到的呢？因为它会使用内核驱动，并且驱动可以访问所有的内核空间，因此可以直接读内核对象的内存，从而不会使 `NtQueryObject` 死锁。我也在想，在将来的 `QuickView` 版本中增加一个驱动，从而不会死锁，并显示更加具体的信息。

### 多个句柄

在上面的例子中，我打开了 `\\TripItinerary.txt` 文件。那么我在另外一个记事本中再打开它会怎么样呢？我们来看看会发生什么。

在用户态调试器中可以看到：

```
0:000> !handle eax ff
Handle 58
  Type          File
  Attributes    0
  GrantedAccess 0x120089:
         ReadControl,Synch
         Read/List,ReadEA,ReadAttr
  HandleCount   2
  PointerCount  3
```

在内核调试器中，列举句柄：

```
kd> !handle 58 ff fcd8ace0
processor number 0
PROCESS fcd8ace0  SessionId: 0  Cid: 0258    Peb: 7ffdf000  ParentCid: 0198
    DirBase: 04b19000  ObjectTable: fccc9648  TableSize:  22.
    Image: notepad.exe
Handle Table at e1e89000 with 22 Entries in use
0058: Object: fcce7028  GrantedAccess: 00120089
Object: fcce7028  Type: (fced7c40) File
    ObjectHeader: fcce7010
        HandleCount: 1  PointerCount: 1
        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}
```

我们还可以在原来那个记事本中列举句柄：

```
kd> !handle 120 ff fcc77200
processor number 0
PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8
    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.
    Image: notepad.exe
Handle Table at e1e5f000 with 74 Entries in use
0120: Object: fcd32448  GrantedAccess: 00120089
Object: fcd32448  Type: (fced7c40) File
    ObjectHeader: fcd32430
        HandleCount: 1  PointerCount: 1
        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}
```

在这个例子中，我们可以看到它们不仅有各自的句柄，还有各自的内核对象。但并不总是这样的。有时候内核对象能在不同进程之间共享。如果同一个进程打开了文件，一般来说它们会得到两个句柄，但是指向同一个内核对象。你可以使用我的工具或 `HANDLE.EXE` 来显示句柄信息，并将他们分类排列，这会让你学习到更多关于句柄的知识。

### 为什么用户态调试器显示的 `HandleCount` 是 2，而 `PointerCount` 是 3？

你会看到，在用户态调试器下显示句柄信息，`"HandleCount"` 是 2，而 `"PointerCount"` 是 3。这是因为当显示信息的时候，我们需要作一些变动。调试器为了得到信息，会使用 `DuplicateHandle` 复制一个句柄。这个复制会将 `HandleCount` 加 1，`PointerCount` 加 2。

我们来看看上面那个 `ObjectHeader:"fcd32430"`。

```
kd> !handle 120 ff fcc77200
processor number 0
PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8
    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.
    Image: notepad.exe
Handle Table at e1e5f000 with 74 Entries in use
0120: Object: fcd32448  GrantedAccess: 00120089
Object: fcd32448  Type: (fced7c40) File
    ObjectHeader: fcd32430
        HandleCount: 1  PointerCount: 1
        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}
```

```
kd> dd fcd32430
fcd32430  00000001 00000001 fced7c40 00000800
fcd32440  fcce5fc8 00000000 00700005 fceccbd0
fcd32450  fcecc248 e1e730d8 e1e73250 fcc790b4
fcd32460  00000000 00000000 00000000 00010000
fcd32470  00010100 00040042 00380024 e1bf2568
fcd32480  00000000 00000000 00000000 00000000
fcd32490  00000000 00040001 00000000 fcd3249c
fcd324a0  fcd3249c 00040000 00000001 fcd324ac
```

```
kd> ba r1 fcd32430
kd> ba r1 fcd32434
kd> g
```

我们可以看到，`handle count` 是 1，`pointer count` 是 1。然后我使用 `"ba r1"` 在 `handle count` 和 `pointer count` 的地址上下断点。`BA` 的意思就是当地址被访问是就断下。`R` 表示它是读还是写。`R1` 中的 1 表示 1 字节。

然后我们使用 `g` 命令让其运行。而后我会在用户态调试器下使用 `!handle 120 ff`，这将会使用户态调试器访问这个对象。我们来看看这个指针是在什么时候增长的。

```
kd> kb
ChildEBP  RetAddr  Args to Child
fb66ebf0  8049ff0d 00000120 00000000 00000000 nt!ObReferenceObjectByHandle+0x1af
fb66ed40  80461691 00000074 00000120 ffffffff nt!NtDuplicateObject+0x12d
fb66ed40  77f83f85 00000074 00000120 ffffffff nt!KiSystemService+0xc4
0006ef28  00000000 00000000 00000000 00000000 ntdll!NtDuplicateObject+0xb
```

我们可以看到，第一个函数是 `ObReferenceObjectByHandle`。这会增加指针的引用计数。我们来看看。

```
kd> !handle 120 ff fcc77200
processor number 0
```



# 调试句柄泄露

## 分析句柄引用计数

以下显示了一个进程 `notepad.exe` 的调试信息，其中句柄 `0120` 对应文件对象 `\\TripItinerary.txt`。初始状态下，`HandleCount` 为 1，`PointerCount` 为 2。

```
PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8  

    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.  

    Image: notepad.exe  

Handle Table at e1e5f000 with 74 Entries in use  

0120: Object: fcd32448  GrantedAccess: 00120089  

Object: fcd32448  Type: (fced7c40) File  

    ObjectHeader: fcd32430  

        HandleCount: 1  PointerCount: 2  

        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}  
```

我们可以看到 `PointerCount` 已经增加到 2 了，我们看看下面会发生什么。

调用 `kd> kb` 查看调用栈，可以看到 `nt!ObpIncrementHandleCount` 被调用，进而涉及 `nt!NtDuplicateObject` 和 `KERNEL32!DuplicateHandle`。

```
kd> kb  

ChildEBP RetAddr  Args to Child          

fb66ebec 8049fffb 00000002 fcc77800 fcd32448 nt!ObpIncrementHandleCount+0x236  

fb66ed40 80461691 00000074 00000120 ffffffff nt!NtDuplicateObject+0x3c3  

fb66ed40 77f83f85 00000074 00000120 ffffffff nt!KiSystemService+0xc4  

0006eed0 77e846ed 00000074 00000120 ffffffff ntdll!NtDuplicateObject+0xb  

0006ef28 69b22188 00000074 00000120 ffffffff KERNEL32!DuplicateHandle+0xd4  

0006f454 69b2252e 00000074 00000120 000000ff ntsdexts!GetHandleInfo+0x29  

0006f4d0 0100d562 00000074 00000050 01001dec ntsdexts!handle+0x10f  

0006f52c 0100e497 00233848 0006f54c 0006f553 ntsd!CallExtension+0x77  

0006f65c 0100c2bb 01089c81 010241d8 01089138 ntsd!fnBangCmd+0x377  

0006f868 0100be0f 80000003 00000001 0006fc2c ntsd!ProcessCommands+0x2ac  

0006fab4 01008406 00000000 00000000 ffffffff ntsd!ProcessStateChange+0x687  

0006fc2c 01008a14 0006fc4c 00000000 00000000 ntsd!DebugEventHandler+0x6a5  

0006fca8 010076dc 00000000 00000000 7ffdf000 ntsd!NtsdExecution+0x13b  

0006ff70 010226bf 00000002 002337b0 00232978 ntsd!main+0x3d7  

0006ffc0 77e87903 00000000 00000000 7ffdf000 ntsd!mainCRTStartup+0xff  

0006fff0 00000000 010225c0 00000000 000000c8 KERNEL32!BaseProcessStart+0x3d  
```

再次执行 `kd> !handle 120 ff fcc77200` 查看句柄信息，可以看到 `HandleCount` 现在也是 2 了。

```
kd> !handle 120 ff fcc77200   

processor number 0  

PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8  

    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.  

    Image: notepad.exe  

Handle Table at e1e5f000 with 74 Entries in use  

0120: Object: fcd32448  GrantedAccess: 00120089  

Object: fcd32448  Type: (fced7c40) File  

    ObjectHeader: fcd32430  

        HandleCount: 2  PointerCount: 2  

        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}  
```

我们可以看到会调用 `ObpIncrementHandleCount`，并且 `handle count` 现在也是 2 了。注意到，这两个调用都来自于 `NtDuplicateObject`，也就是用户态下的 `DuplicateHandle`。因此，复制对象会增加这块内核内存的引用计数，并增加句柄数量，这就是两个进程使用同一个内核对象的一个例子。那么第 3 个指针引用是从哪来的？

再次调用 `kd> kb`，可以看到调用栈中涉及 `nt!ObReferenceObjectByHandle` 和 `nt!NtQueryObject`。

```
kd> kb  

ChildEBP RetAddr  Args to Child          

fb66ec68 804baed8 00000100 00000000 00000000 nt!ObReferenceObjectByHandle+0x1af  

fb66ed48 80461691 00000100 00000002 0006ef4c nt!NtQueryObject+0xc1  

fb66ed48 77f8c4e1 00000100 00000002 0006ef4c nt!KiSystemService+0xc4  

0006ef24 69b22203 00000100 00000002 0006ef4c ntdll!NtQueryObject+0xb  

0006f454 69b2252e 00000000 00000120 000000ff ntsdexts!GetHandleInfo+0xa4  

0006f4d0 0100d562 00000074 00000050 01001dec ntsdexts!handle+0x10f  

0006f52c 0100e497 00233848 0006f54c 0006f553 ntsd!CallExtension+0x77  

0006f65c 0100c2bb 01089c81 010241d8 01089138 ntsd!fnBangCmd+0x377  

0006f868 0100be0f 80000003 00000001 0006fc2c ntsd!ProcessCommands+0x2ac  

0006fab4 01008406 00000000 00000000 ffffffff ntsd!ProcessStateChange+0x687  

0006fc2c 01008a14 0006fc4c 00000000 00000000 ntsd!DebugEventHandler+0x6a5  

0006fca8 010076dc 00000000 00000000 7ffdf000 ntsd!NtsdExecution+0x13b  

0006ff70 010226bf 00000002 002337b0 00232978 ntsd!main+0x3d7  

0006ffc0 77e87903 00000000 00000000 7ffdf000 ntsd!mainCRTStartup+0xff  

0006fff0 00000000 010225c0 00000000 000000c8 KERNEL32!BaseProcessStart+0x3d  
```

再次执行 `kd> !handle 120 ff fcc77200`，可以看到 `PointerCount` 增加到了 3。

```
kd> !handle 120 ff fcc77200   

processor number 0  

PROCESS fcc77200  SessionId: 0  Cid: 0338    Peb: 7ffdf000  ParentCid: 02c8  

    DirBase: 079de000  ObjectTable: fccc30c8  TableSize:  74.  

    Image: notepad.exe  

Handle Table at e1e5f000 with 74 Entries in use  

0120: Object: fcd32448  GrantedAccess: 00120089  

Object: fcd32448  Type: (fced7c40) File  

    ObjectHeader: fcd32430  

        HandleCount: 2  PointerCount: 3  

        Directory Object: 00000000  Name: \\TripItinerary.txt {HarddiskVolume1}  
```

看到了吗？我前面提到过，调试器会调用 `NtQueryObject` 来得到信息，这个函数读这块内存会调用 `ObReferenceObjectByHandle`，这又会将 `pointer count` 增加到 3。然而，`NtQueryObject` 并不需要创建或复制另外一个句柄，它只是增加引用计数，取得信息，然后释放引用计数。

另外一个进程也可能释放它的句柄，并且只显示调试器的引用。调试器返回的信息关系到 1 个句柄引用和 2 个指针引用。调试器显示完信息后，就会将这个数减去。因为 `NtQueryObject` 调用完之后，句柄将关闭，指针引用将不再存在，这就让这个对象回复到原来的状态。

顺便说一下，`DuplicateHandle` 需要一个源进程句柄作为参数，因此，你必须使用 `OpenProcess`，并指定复制权限来复制句柄。

## 确定是否是泄露

我们已经了解了句柄，那么，我们该怎么找到句柄泄露呢？除了一些现成的工具可以使用，比如 `Bounds Checker`，我们还可以自己去找。我们要做的第一件事就是在恰当的时机检查程序的句柄基数。任务管理器有一个选项可以显示句柄。下面，我们来对程序做一些操作，并看看句柄是增加还是减少。如果句柄在不断增加，你就要注意是否是有句柄泄露，这都必须靠你自己去确定。

比如，如果你有一个服务端程序，并且你在检查网络连接。你注意到你正在和很多客户端连接，服务端的连接数不断增加，这是在意料中的。当然你不仅仅只是判断这些东西，当所有的客户端都断开连接了，所有的网络的连接都应该被清除掉。因此，如果你知道你的句柄计数在不正常的增加，那你就要测试了。

### 第一步: 确定是否是泄露

如果你正在监视一个程序，确定句柄计数是否正常，观察它是否稳定增加，快或慢。如果你确定这就是一个泄露或在这个时间点你不确定，那么我们就可以进入下一步，找出可能正在泄露的句柄。如果可能的话，泄露可能会是一种固定的模式，比如你每次按菜单或打开一个文件。这可以帮助我们减小检查范围。

### 第二步: 确定类型和对象信息

完成这一步最好的方法就是使用 `Handle.exe` 这样的工具或调试器。我写了一个句柄泄露速度很快的程序，运行之后观察任务管理器，发现有超过 65000 个句柄泄露。因此，我们需要确定泄露的句柄类型，这样的话，我们就可以确定哪个 API 导致这样的问题。

执行 `!handle 0 0` 查看句柄统计：

```
0:001> !handle 0 0  

65545 Handles  

Type            Count  

Event           3  

Section         1  

File            1  

Port            1  

Directory       3  

WindowStation   2  

Semaphore       2  

Key             65530  

Desktop         1  

KeyedEvent      1  
```

在这个夸张的例子中，我们可以看到泄露的句柄类型是 `Key`，也就是注册表句柄。下面我们要确定发生问题的位置。如果我们能找到这个键的信息，我们就能知道哪个键被打开了，然后缩小检查范围。

执行 `!handle 0 ff Key` 查看 `Key` 类型句柄的详细信息：

```
!handle 0 ff Key  

Handle 2de0  

  Type          Key  

  Attributes    0  

  GrantedAccess 0xf003f:  

         Delete,ReadControl,WriteDac,WriteOwner  

         QueryValue,SetValue,CreateSubKey,EnumSubKey,Notify,CreateLink  

  HandleCount   2  

  PointerCount  3  

  Name          \\REGISTRY\\USER\\S-1-5-21-789336058-706699826-1202660629-1003\\Software  

  Object Specific Information  

    Key last write time:  01:10:03. 5/9/2004  

Key name Software  
```

我们看到，调试器给我们显示键的信息，并且 `HKCU\Software` 被打开的次数最多。我们可以确定这是一个 `key` 对象，也就是说可能使用 `RegOpenKey` 或 `RegOpenKeyEx` 打开的。我们已经确定了被打开的 `key`，下面看第三步。

### 第三步: 浏览源码，调试程序

我们现在已经知道了类型，使用的 API，还有被打开的资源。下面，我们就可以查看源码，找到这个地方或在 `RegOpenKey`/`RegOpenKeyEx` 上下断点来看栈回溯。然后我们就可以跟踪句柄，确定源头。`Bounds Checker` 也能完成类似这样功能。

另外，我们还可以使用在“堆”那节讲述的寻找内存泄露的方法。我们可以将创建句柄的 API 包装成一些函数，然后将句柄放入链表中，释放的时候将其从链表中删除即可。缺点是，删除的时候遍历链表会很慢。

```c
//This method could return the actual key, but would require  
//a search of the linked list on a close.  

DWORD MyOpenKey(..., phKey)  

{  

    dwResult = RegOpenKey(... phKey);  

    pTemp = Allocate();  

    pTemp->pNext = gpHead;  

    gpHead = pTemp;  

    gpHead->hKey = phKey;  

    return dwResult;  

}  

//This method would allow faster look up in the close but  

//would also require a wrapper for all functions.  

PMYKEY MyOpenKey(...)  

{  

    hKey = RegOpenKey(...);  

    pTemp = Allocate();  

    pTemp->pNext = gpHead;  

    gpHead = pTemp;  

    gpHead->hKey = hKey;  

    return gpHead;  

}  
```

上面给出的只是一些伪代码。使用的时候必须加上临界区，或者其它一些方法。增加这个全局链表的目的是让调试扩展可以遍历它。这对大部分问题来说可能都太不需要了，并且可能使用在一些设置好的工程中。

## 轻量泄露

不是所有的句柄泄露都像上面一样那么夸张。比如，可能一次就只有 1 到 2 个句柄泄露。你可以在程序中中断，记住所有的句柄数，然后运行泄露程序，然后再得到一次快照。如果确实有问题存在，你可以再重现一次问题，但是这次你需要在创建对象的地方下断。

## 其他提示

下面是其他相关技巧。

### 对线程的不正确的退出等待

我看到很多程序都使用 `Sleep()`/`GetExitCodeThread()` 循环来等待线程退出。比如：

```c
do {  
  Sleep(10);  
  bReturn = GetExitCodeThread(hThread, &ExitCode);  
} while(bReturn && ExitCode == STILL_ACTIVE) ;  
```

`MSDN` 上面说到，线程会将 `STILL_ACTIVE` 作为它的返回值。如果这是真的，那我就不用担心这么多了。我发现有些程序使用 `MFC` 的 `CThread` 库，并使用这个循环。问题是 `AfxBeginThread()` 会传递一个已经创建的线程的句柄，这个线程会使用 `AfxEndThread()` 退出，关闭句柄，如果你没有复制这个句柄的话，这个句柄这时就是无效的。

一般来说，这也不会造成问题，因为出现错误的话，循环也会退出。但是如果另外一个被创建的对象也使用这个句柄值，问题就来了。记住，这些句柄值是可以重复使用的。这就意味着这个句柄这时代表另外一个对象，这可能是另外一个线程对象，从而导致这个循环一直持续。

在 `Afx` 之外的其他地方，你也可能会关闭句柄，因此保证一定要复制它。你也不需要一直循环，因为线程退出后，句柄会变为有信号状态。

```c
WaitForSingleObject(hThread, INFINITE);  

GetExitCodeThread(hThread, &ExitCode);  
```



`CloseHandle(hThread);`

因此，上面这种方法将会是更好的选择。

### `.DMP` 文件丢失句柄信息？

如果你有一个`.DMP` 文件，并且使用`!handle` 命令，你可能会得到一个错误信息。因为对于`.DMP` 文件，你不能使用`NtQueryObject` 来得到句柄信息了。在这种情况下，调试器需要取得所有的信息，并把信息存储在`.DMP` 文件中。设置一些`.dump` 标志可以不存储，在所有的`NTSD`/`CDB`/`WinDbg` 版本中，使用`.dump /f x.dmp (full dump)` 将不存储句柄信息。另外一种选择就是使用`.dump /mh x2.dmp`，但是这不是完全转储，而是一个”mini dump”。

如果你从官网下载最新的调试器的话，有一个新选项`/ma` 可以生成带有句柄信息的完整转储，像这样`.dump /ma x3.dmp`。这就是我推荐你使用的能生成句柄信息的用户态`DUMP`。

### 总结

句柄是 Windows 中的一部分，我们必须了解它，并使用它。当你检查内存泄露或其他一些问题的时候，记住不要忘记检查句柄计数。希望这篇文章帮助你了解什么是句柄，以及如何检查它们。
