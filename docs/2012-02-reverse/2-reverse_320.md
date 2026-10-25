
# Debug Tutorial (完全翻译)

> 原文：`[www.pediy.com/kssd/pediy12/104828-1.html](https://www.pediy.com/kssd/pediy12/104828-1.html)`

使用 CDB 和 NTSD 开始调试旅程

## 介绍

在软件开发和维护的过程中，调试是最有具有价值的技能之一，它几乎被用在产品生命周期的每个阶段。首先，开发人员很明显会遇到很多错误。这些错误多种多样，有逻辑错误、语法错误、编译器方面的错误等等。其次，当测试更多的高级的情况或当软件在和其他环境交互的时候，质量保证部门人员可能会遇到困难。最后，产品发布之后，公司必须对它提供一些支持。顾客拿到软件产品之后，调试并没有结束，错误通常都会升级，并被返回公司等待调试。

## 这篇教程的目的

这篇教程仅仅是关于调试的一个介绍。这篇是”教程 #1”，如果反馈好的话，我会继续写。有许多复杂的调试技术以及问题，以至于我们不知道从哪里开始着手。这篇教程从最初开始帮助你了解调试是怎么一回事。我希望把高级调试的世界展现给初级和中级程序员，而不是简单的重新编译，或者简单的 `MessageBox` 和 `Printf` 调试。

## 调试器和操作系统

你可以到下面这个网站下载最新的微软提供的调试器。

`http://www.microsoft.com/whdc/ddk/debugging`

## CDB，NTSD 和 Windbg

这篇文章讨论的情况一般都是 Windows 2000 以及更高版本。我们将要谈论的这三个调试器是 CDB，NTSD 和 WinDbg。Windows 2000 以及更高版本一般系统内都内置 NTSD!所以，如果你想快速调试，就不需要另外再安装软件了。

那么这又有什么不一样呢？文档上说”NTSD 不需要控制台支持，而 CDB 需要”。这是真的，NTSD 确实不需要，而 CDB 需要。然而，我发现还有更多的不同之处。第一，老版本的 NTSD 不支持 PDB 符号文件，它们只支持 DBG 符号文件!我还发现 NTSD 不支持符号服务器，而 CDB 支持。老版本的 NTSD 不能创建内存 dump，还有其他一些问题，比如 NTSD 只支持 2 种断点命令。NTSD 相对于 CDB 的一个优势就是它不需要控制台。

当你正在调试用户态服务或者登陆到系统之前的进程时，不需要控制台窗口这个特点是非常重要的。如果没有用户登陆到系统，你就不能创建控制台窗口。你可以设置一个命令选项`-d`，让 NTSD 和已经连接上的内核调试器通信(CDB 也有相同的选项)。这样就能通过内核调试器来调试系统启动期间的进程。当你已经可以用内核调试器调试进程，用户态调试器能给你更多的灵活性。这已经超出了介绍章节的范围，只要消化这个概念就行了。

除了很少的一些差异，WinDbg 和 CDB 几乎一样。WinDbg 是 GUI 程序，CDB 是控制台程序，这是第一个不同。WinDbg 还支持内核调试和源码级调试。

## Visual C++ 调试器

我不使用这个调试器，并且我不推荐使用它。第一个原因就是它非常消耗资源。它加载的非常慢，并且包含了很多没用的东西，以至于成为累赘。第二个原因就是安装完这个调试器后，你需要重启，我一般都是在没有安装调试器的机器上工作。并且 VC++ 非常大，安装费时。

## Windows 9x/ME

在 Windows 9x/ME 的机器上，我们该怎么办呢？你可以使用 WinDbg。对所有系统，和调试有关的 API 都是一样的，因此 Windbg 能运行在 Windows 9x/ME 上。我唯一的担心就是 WinDbg 会检测当前系统是否 Windows 9x 并且会不允许调试。我最近发现事实上不会这样。剩下的问题就是，最新的 WinDbg 是 MSI 安装包，并且不允许安装在 Windows 9x 系统上。我们可以在 NT 系统上安装，然后共享这个目录或者拷贝到 CD 上共享。这会有一些影响，比如若 NT 和 9x 系统放置的数据在内存的不同地方，你就不能使用所有的`!xxx` 命令。那么符号能使用吗？是的，PDB 可以使用。但是当设置了`ba r1 xxxxx` 之后，单步走非常慢。这篇文章的内容不包括 Windows 9x/ME。

## 设置环境

开始调试前，这是非常重要的一步。把系统配置成你喜欢的状态，并且包含所有你需要的工具。

### 符号和符号服务器

符号是调试操作中很重要的一步。你可以从微软的一个地址下载相应操作系统的所有符号。问题是，你需要很大的硬盘空间，并且不如你在一台机器上调试许多操作系统，这可能会非常麻烦。

为了适应这个需要，微软提供了”符号服务器”这个功能。这会帮助你得到正确的符号。符号服务器的地址`http://msdl.microsoft.com/download/symbols`。如果你把符号路径设置为这个地址，调试器将会自动你需要的系统符号。你自己的程序的符号需要你自己设置。

### 映像文件执行选项

这是注册表中的一个位置，当一个程序开始运行，这个注册表位置会自动将调试器附加到程序。这个注册表位置如下:

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options`

在这个键下，你只要创建一个子键，取名为你想要调试的程序，比如”myapplication.exe”。如果你以前没有使用过这个功能，可能会有一个默认键值”Your Application Here”或类似的值，重命名它即可。

这个键下的一个值是”Debugger”，你可以在这里设置需要开启的调试器，”Your Application Here”下面默认的这个值为”ntsd -d”。你不能使用这个，除非已经有内核调试器附加在系统上，所以要去掉”-d”选项。

**注意**: 使用”-d”选项，并且当前没有内核调试器附加在系统上，每次启动程序都有可能导致系统锁死。必须非常小心。如果内核调试器已经设置好了，你可以使用”g”命令解锁系统。

另外一个值为”GlobalFlags”。这是另外一个可以用于调试的工具，然而它超出了本篇的范围。如果你想知道更多，可以看”gflags.exe”。

### 内核调试设置

如果需要内核调试，首先你需要以调试模式启动系统。虽然在系统属性里面可以用 GUI 的方式设置，我还是建议直接编辑`boot.ini` 文件。在`C:\`盘找到`boot.ini` 文件，它是一个隐藏的系统文件。

**小心**: 不正确的编辑这个文件将会让你无法启动。

这个启动文件内容类似下面:

```
timeout=30

default=multi(0)disk(0)rdisk(0)partition(1)\WINDOWS

[operating systems]

multi(0)disk(0)rdisk(0)partition(1)\WINDOWS.0=

"Microsoft Windows XP Professional" /fastdetect
```

我将复制 Operating Systems 下面的第一行:

```
timeout=30

default=multi(0)disk(0)rdisk(0)partition(1)\WINDOWS

[operating systems]

multi(0)disk(0)rdisk(0)partition(1)\WINDOWS.0=

"Microsoft Windows XP Professional" /fastdetect

multi(0)disk(0)rdisk(0)partition(1)\WINDOWS.0=

"Microsoft Windows XP Professional"

/fastdetect /debug /debugport=COM1 /baudrate=115200
```

复制的这行包含了你的设置。`/debug`, 然后是`/debugport=port`, 最后是`/baudrate=baudrate`。调试端口是你需要使用的串行端口，这是你需要设置的硬件，你还需要另外一台机器来完成设置。除了使用 COM 端口，你还可以使用 IEEE 1394，这个速度会更快一些。

当你再次启动后，选择”Debugger Enabled”选项来启动调试模式。

(其实我们一般都用虚拟机来完成这部分的设置，具体可以在论坛上搜，有很多教程。)

### 环境变量

我一般会把`_NT_SYMBOL_PATH` 环境变量设置为微软的符号服务器和我自己的符号目录。你可以在系统属性->高级->环境变量 里面设置这个值。

### 默认调试器

当有任何崩溃出现在系统上，将会调用缺省调试器。默认情况下，它被设置为”Doctor Watson”。这个注册表键在如下位置:

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AeDebug`

我会把”Auto”设为 1，”Debugger”设为需要的调试器。

## 汇编

我十分推荐你学习汇编语言。这篇教程不会给你展示源码级调试，因为我从来不这样做并且我也不会。源码级调试带来的问题是，你不会总是有源代码，并且有时候从源代码看不出问题在哪。如果你了解系统的组成，你可以很容易的逆向系统来找到你需要的信息，这个源码级调试不能带给你的。

我讨厌源码级调试的另外一个原因是，如果源码和符号不符合，调试器将会给你错误的信息。这意味着如果你为你的项目创建了多个版本，你不得不找到和你正在调试的程序符合的版本。

## 让我们开始吧

这篇教程是第一部分，如果你们喜欢，我们写更多，并且会更深入。这篇将会解决两个简单的用户态编程问题。

### 发布版本的符号

首先，怎样为发布版程序创建符号呢？很简单，创建一个 make 文件。

我一般使用的编译选项如下:

```
/nologo /MD /W3 /Oxs /Zi /I "..\..\inc" /D "WIN32" /D "_WINDOWS"
/Fr$(OBJDIR)\ /Fo$(OBJDIR)\ /Fd$(OBJDIR)\ /c
```

我一般使用的链接选项如下:

```
/nologo /subsystem:console
/out:$(TARGETDIR)\$(TARGET)/pdb:<YourProjectName>.pdb
/debug /debugtype:both
/LIBPATH:"..\..\..\bin\lib"
```

这将会为你的工程创建.pdb 文件。当然，根据 VC++7 的介绍，他们已经放弃使用.DBG(因此`/debugtype:both` 在这个编译器上可能会出现错误)。.DBG 是.PDB 的简单版本并且它不包含源码信息，精确的符号察看。它甚至不包含参数或其他一些东西。如果你还在使用这样的编译器，你需要做下面的工作:

```
rebase -b 0x00100000 -x $(TARGETDIR) -a $(TARGETDIR)\$(TARGET)
```

`-b` 选项后面是重定位的新的可执行文件的内存地址。如果你使用默认的 Visual Studio 方式创建文件，那可能会比这个更小一点，不过，你不会获得符号。产生的代码是一样的，并且会有相应的优化。不同的是，这些文件会更有用，不论你在哪里使用，你都能得到符号信息。

记住，最棒的调试时机总是在你还没有重新生成可执行文件之前。一旦你不得不重新生成可执行文件，你必须知道，你已经改变了这个文件在内存中的位置。你也可能改变了文件执行的速度。如果你需要重现这个错误，这将是非常重要的!如果需要 4 天才能引起这个错误，那该怎么办呢？如果能的话，最好在发生的时候就去处理它。

## 简单的 Access Violation 错误

我们来看一个简单的”Access Violation”错误，这很常见。解决这个问题可以分为三步。

1. 谁触发了这个访问？哪个模块？
2. 它要访问哪里？这块内存在哪？
3. 为什么它要访问这块内存？它要干什么?

这些是一般情况下解决这个问题的方法，其中第 2 条又是最重要的。然而，解决 1 和 3 的问题可以帮助解决 2 的问题。

我创建了一个简单的崩溃的程序。我已经把我的默认调试器设置为 CDB，并且我运行了这个程序。我也为这个可执行文件创建了符号并把`_NT_SYMBOL_PATH` 设置为微软的符号服务器。

当我们运行这个程序，我们将会看到下面的情况:

```
C:\programs\DirectX\Games\src\Games\temp\bin>temp

Microsoft (R) Windows Debugger Version 6.3.0005.1

Copyright (c) Microsoft Corporation. All rights reserved.

*** wait with pending attach

Symbol search path is:

SRV*c:\symbols*http://msdl.microsoft.com/download/symbols

Executable search path is:

ModLoad: 00400000 00404000 C:\programs\DirectX\Games\src\Games\temp\bin\temp.exe

ModLoad: 77f50000 77ff7000 C:\WINDOWS.0\System32\ntdll.dll

ModLoad: 77e60000 77f46000 C:\WINDOWS.0\system32\kernel32.dll

ModLoad: 77c10000 77c63000 C:\WINDOWS.0\system32\MSVCRT.dll

ModLoad: 77dd0000 77e5d000 C:\WINDOWS.0\system32\ADVAPI32.DLL

ModLoad: 78000000 78086000 C:\WINDOWS.0\system32\RPCRT4.dll

(ee8.c38): Access violation - code c0000005 (!!! second chance !!!)

eax=00000000 ebx=7ffdf000 ecx=00001000 edx=00320608 esi=77c5aca0 edi=77f944a8

eip=77c3f10b esp=0012fb0c ebp=0012fd60 iopl=0 nv up ei pl zr na po nc

cs=001b ss=0023 ds=0023 es=0023 fs=0038 gs=0000 efl=00000246

MSVCRT!_output+0x18:

77c3f10b 8a18 mov bl,[eax] ds:0023:00000000=??

0:000>
```

第一个要注意的就是，这个错误发生在 MSVCRT.DLL 当中。这很明显，因为调试器给我们显示了类似的信息，格式为`<module>!<nearest symbol>+offset`。这意味着最近的符号是`_output`，并且我们运行到了里面的+18h 处。因此我们假设自己正处于`_output` 函数当中。

```
(ee8.c38): Access violation - code c0000005 (!!! second chance !!!)

eax=00000000 ebx=7ffdf000 ecx=00001000 edx=00320608 esi=77c5aca0 edi=77f944a8

eip=77c3f10b esp=0012fb0c ebp=0012fd60 iopl=0 nv up ei pl zr na po nc

cs=001b ss=0023 ds=0023 es=0023 fs=0038 gs=0000 efl=00000246

MSVCRT!_output+0x18:

77c3f10b 8a18 mov bl,[eax] ds:0023:00000000=??

0:000>
```

如果我们想验证这个信息，我们应该怎么做？

```
<0:000> x *!

start end module name

00400000 00404000 temp (deferred)

77c10000 77c63000 MSVCRT (pdb symbols)
c:\symbols\msvcrt.pdb\3D6DD5921\msvcrt.pdb

77dd0000 77e5d000 ADVAPI32 (deferred)

77e60000 77f46000 kernel32 (deferred)

77f50000 77ff7000 ntdll (deferred)

78000000 78086000 RPCRT4 (deferred)
```

这个命令展示所有模块的列表以及它们的开始和结束位置。我们的错误地址是 77c3f10b，77c10000<=77c3f10b<=77c63000，因此可以确认错误发生在 MSVCRT。下面我们来确定这个地址在哪。

有几种方法可以完成，我们可以反汇编代码，找出这个地址，还可以看栈回溯。首先我们来看看`_output` 函数的反汇编代码。

```
0:000> u MSVCRT!_output

MSVCRT!_output:
```



# 77c3f0f3 55 push ebp

```
77c3f0f3 55             push    ebp  
77c3f0f4 8bec           mov     ebp,esp  
77c3f0f6 81ec50020000   sub     esp,0x250  
77c3f0fc 33c0           xor     eax,eax  
77c3f0fe 8945d8         mov     [ebp-0x28],eax  
77c3f101 8945f0         mov     [ebp-0x10],eax  
77c3f104 8945ec         mov     [ebp-0x14],eax  
77c3f107 8b450c         mov     eax,[ebp+0xc]  
```

```
0:000> u  
```

`MSVCRT!_output+0x17:`

```
77c3f10a 53             push    ebx  
77c3f10b 8a18           mov     bl,[eax]  
```

即使你不懂汇编，你也会发现一些东西。首先，我们可以发现这个内存地址在`EAX`当中。它是CPU的一个寄存器，但是我们可以把它看作一个变量。环绕`EAX`的`[]`符号相当于C语言中的`*MyPointer`。这意味着我们正在引用`EAX`指向的地址。那么`EAX`又是怎么来的？`EAX`来自`[EBP+0Ch]`，你可以把它看作`DWORD *EBP, EAX=EBP[3]`。这是因为在汇编语言中，没有类型。`EAX`是一个32位寄存器，`EBP+12`相当于一个`DWORD`指针加3。

下面我们可以看到`MOV EBP, ESP`。`ESP`是栈指针。参数都是压入栈中，返回地址和局部变量都是在栈中。`ESP`指向栈的位置。在内存中，一个C函数调用约定的存放如下:

```
[Parameter n]  

...  

[Parameter 2]  

[Parameter 1]  

[Return Address]  
```

并且，我们看到了`PUSH EBP`。`PUSH`就是把某个东西压入栈，因此我们在栈中保存了`EBP`以前的值。此时，栈的状态如下:

```
[Parameter n]  

...  

[Parameter 2]  

[Parameter 1]  

[Return Address]  

[Previous EBP]  
```

然后我们又把`EBP`设为`ESP`，我们可以把它看作是一个指针，栈就相当于一个`DWORD`类型的数组。因此，栈中各个变量与`EBP`的对应如下:

```
[Parameter n]     ==  [EBP + n*4 + 4]  (The formula)  

...  

[Parameter 2]     ==  [EBP + 12]  

[Parameter 1]     ==  [EBP + 8]  

[Return Address]  ==  [EBP + 4]  

[Previous EBP]    ==  [EBP + 0]  
```

对于我们这个例子来说，我们的变量是`_output`的第二个参数。那么，下面该怎么办呢？我们来反汇编调用函数!我们知道`EBP+4`指向返回地址，或者我们也可以得到栈回溯。

```
0:000> kb  

ChildEBP RetAddr   Args to Child  

0012fd60 77c3e68d 77c5aca0 00000000 0012fdb0 MSVCRT!_output+0x18  

0012fda4 0040102f 00000000 00000000 00403010 MSVCRT!printf+0x35  

0012ff4c 00401125 00000001 00323d70 00322ca8 temp!main+0x2f  

0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3  

0012fff0 00000000 00401042 00000000 78746341 kernel32!BaseProcessStart+0x23  

0:000>  
```

“`KB`”命令能得到栈回溯。我们不会总是得到完整的栈回溯，我们会在更加深入的教程中讲解这点。在这篇简单的教程里，我们假定我们得到了完整的栈回溯。我们注意到，这个函数是`printf`，并且`printf`调用`_output`。我们来反汇编`printf`，注意我们不用每次都反汇编整个函数，将其分割成几段即可。有时，我们能从栈回溯中很简单就找到错误发生点，而这些函数也都非常简单，我们可以非常轻松的跟踪他们。

```
0:000> u MSVCRT!_output  

MSVCRT!_output:  

77c3f0f3 55             push    ebp  
77c3f0f4 8bec           mov     ebp,esp  
77c3f0f6 81ec50020000   sub     esp,0x250  
77c3f0fc 33c0           xor     eax,eax  
77c3f0fe 8945d8         mov     [ebp-0x28],eax  
77c3f101 8945f0         mov     [ebp-0x10],eax  
77c3f104 8945ec         mov     [ebp-0x14],eax  
77c3f107 8b450c         mov     eax,[ebp+0xc]  
```

```
0:000> u  
```

`MSVCRT!_output+0x17:`

```
77c3f10a 53             push    ebx  
77c3f10b 8a18           mov     bl,[eax]  
77c3f10d 33c9           xor     ecx,ecx  
77c3f10f 84db           test    bl,bl  
77c3f111 0f8445070000   je      MSVCRT!_output+0x769 (77c3  
77c3f117 56             push    esi  
77c3f118 57             push    edi  
77c3f119 8bf8           mov     edi,eax  
```

```
0:000> u MSVCRT!printf  

MSVCRT!printf:  

77c3e658 6a10           push    0x10  
77c3e65a 68e046c177     push    0x77c146e0  
77c3e65f e8606effff     call    MSVCRT!_SEH_prolog (77c354  
77c3e664 bea0acc577     mov     esi,0x77c5aca0  
77c3e669 56             push    esi  
77c3e66a 6a01           push    0x1  
77c3e66c e8bdadffff     call    MSVCRT!_lock_file2 (77c394  
77c3e671 59             pop     ecx  
```

```
0:000> u  
```

`MSVCRT!printf+0x1a:`

```
77c3e672 59             pop     ecx  
77c3e673 8365fc00       and     dword ptr [ebp-0x4],0x0  
77c3e677 56             push    esi  
77c3e678 e8c7140000     call    MSVCRT!_stbuf (77c3fb44)  
77c3e67d 8945e4         mov     [ebp-0x1c],eax  
77c3e680 8d450c         lea     eax,[ebp+0xc]  
77c3e683 50             push    eax  
77c3e684 ff7508         push    dword ptr [ebp+0x8]  
```

```
0:000> u  
```

`MSVCRT!printf+0x2f:`

```
77c3e687 56             push    esi  
77c3e688 e8660a0000     call    MSVCRT!_output (77c3f0f3)  
```

`_output`的第二个参数是`[EBP+8]`,并且有`PUSH EBP`和`MOV EBP, ESP`，因此这和我之前说的情况一样。但是也不总是这样的，要视具体情况而定，我们慢慢再深入讲解。

因此，我们可以确定`printf`的第一个参数在内存中的哪个地方，并且`printf`是我们的程序发出的调用。从错误信息，我们知道`EAX`是0，因此我们在对一个空指针解引用。

```
77c3f10b 8a18    mov bl,[eax]   ds:0023:00000000=??  
```

下面是我们写的代码:

```
int main(int argc, char *argv[])  
 {    
  char *TheLastParameter[100];  
  sprintf(*TheLastParameter, "The last parameter is %s", argv[argc]);  
  printf(*TheLastParameter);  
  return 0;  
 }  
```

这段代码有很多问题，然后因为空指针，错误发生在`printf`。奇怪的是，并没有在`sprintf()`处出现错误。那么，我们该怎样只用`KB`命令解决这个问题呢？

```
0:000> kb  

ChildEBP RetAddr   Args to Child  

0012fd60 77c3e68d 77c5aca0 00000000 0012fdb0 MSVCRT!_output+0x18  

0012fda4 0040102f 00000000 00000000 00403010 MSVCRT!printf+0x35  

0012ff4c 00401125 00000001 00323d70 00322ca8 temp!main+0x2f  

0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3  

0012fff0 00000000 00401042 00000000 78746341 kernel32!BaseProcessStart+0x23  

0:000>  
```

我们有符号和栈回溯，第一个参数是0，并且我们调用了这个函数。这是很简单的情况，我会试着讲述一些技巧，让你发现问题的所在之处。知道栈是如何组成的，在内存中的状态，这对你找出问题所在之处非常有帮助，因为仅仅一个“`kb`”命令不会让你一眼就能看出问题。

## 不按预期运行的程序

这也是一个很常见的错误。你运行了程序，但是你并没有看到正确的输出，或者总是有一些错误信息。这个常见的问题有可能非常容易解决，也可能很复杂。那么解决这种问题的步骤如何呢？

1. 什么东西没有工作？
2. 哪些API或模块可能会和这个问题有关？
3. 什么原因导致这些API不正常的工作？

这些步骤不是必须的。下面让我们来看一个例子，关于打开文件的。

```
  HANDLE hFile;  
  DWORD dwWritten;  
  hFile = CreateFile("c:\\MyFile.txt", GENERIC_READ,  
                      0, NULL, OPEN_EXISTING, 0, NULL);  
  if(hFile != INVALID_HANDLE_VALUE)  
  {  
   WriteFile(hFile, "Test", strlen("Test"), &dwWritten, NULL);  
   CloseHandle(hFile);  
  }  
```

这是我们的代码。你可能会想使用`GetLastError()`，然后重新编译，并将错误打印出来。但是，你并不需要这样做，在这个例子中非常简单。下面我们来看看，打开调试器，并且会停在这个函数。有符号的帮助，一切都很简单，`CreateFile`是一个导出的符号，我们总是能断在这个函数上。

```
C:\programs\DirectX\Games\src\Games\temp\bin>cdb temp  

Microsoft (R) Windows Debugger  Version 6.3.0005.1  

Copyright (c) Microsoft Corporation. All rights reserved.  

CommandLine: temp  

Symbol search path is:  
    SRV*c:\symbols*http://msdl.microsoft.com/download/symbols  

Executable search path is:  

ModLoad: 00400000 00404000   temp.exe  

ModLoad: 77f50000 77ff7000   ntdll.dll  

ModLoad: 77e60000 77f46000   C:\WINDOWS.0\system32\kernel32.dll  

ModLoad: 77c10000 77c63000   C:\WINDOWS.0\system32\MSVCRT.dll  

(2a0.94): Break instruction exception - code 80000003 (first chance)  

eax=00241eb4 ebx=7ffdf000 ecx=00000004 edx=77f51310 esi=00241eb4 edi=00241f48  

eip=77f75a58 esp=0012fb38 ebp=0012fc2c iopl=0         nv up ei pl nz na pe nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202  

ntdll!DbgBreakPoint:  

77f75a58 cc             int     3  
```

```
0:000> bp temp!main  

0:000> g  
```

我们在`main`函数上下一个断点，然后使用“`g`”命令让其运行。当我们到达断点，用“`p`”命令单步走，直到`CreateFile`函数。

```
Breakpoint 0 hit  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401000 esp=0012ff50 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main:  

00401000 51             push    ecx  
```

```
0:000> p  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401001 esp=0012ff4c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x1:  

00401001 56             push    esi  
```

```
0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401002 esp=0012ff48 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x2:  

00401002 57             push    edi  
```

```
0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401003 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x3:  
```



`temp!main+0x3`:

```
00401003 33ff             xor     edi,edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401005 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x5`:

```
00401005 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401006 esp=0012ff40 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x6`:

```
00401006 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401007 esp=0012ff3c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x7`:

```
00401007 6a03            push    0x3
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401009 esp=0012ff38 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x9`:

```
00401009 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=0040100a esp=0012ff34 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0xa`:

```
0040100a 57              push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=0040100b esp=0012ff30 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0xb`:

```
0040100b 6800000080      push    0x80000000
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401010 esp=0012ff2c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x10`:

```
00401010 6810304000      push    0x403010
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401015 esp=0012ff28 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x15`:

```
00401015 ff1504204000    call    dword ptr [temp!__imp__CreateFileA (00402004)]{kernel32!CreateFileA (77e7b476)} ds:0023:00402004=77e7b476
```

`0:000> p`

```
eax=ffffffff ebx=7ffdf000 ecx=77f939e3 edx=00000002 esi=00000000 edi=00000000
eip=0040101b esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei ng nz na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000286
```

`temp!main+0x1b`:

```
0040101b 8bf0            mov     esi,eax
```

在调用 `CreateFile` 函数之后，`EAX` 中将存储返回值。我们注意到其值为 `ffffffff`，也就是”Invalid Handle Value”，我们还想知道 `GetLastError` 的值，它存储在 `fs:34` 这个位置。`FS` 是 `TEB` 选择子，我们可以把它 dump 出来。

`0:000> dd fs:34`

```
0038:00000034  00000002 00000000 00000000 00000000
0038:00000044  00000000 00000000 00000000 00000000
0038:00000054  00000000 00000000 00000000 00000000
0038:00000064  00000000 00000000 00000000 00000000
0038:00000074  00000000 00000000 00000000 00000000
0038:00000084  00000000 00000000 00000000 00000000
0038:00000094  00000000 00000000 00000000 00000000
0038:000000a4  00000000 00000000 00000000 00000000
```

`CDB` 还有一种更快速的方式能做到这点，`!gle`:

`0:000> !gle`

```
LastErrorValue: (Win32) 0x2 (2) - The system cannot find the file specified.
LastStatusValue: (NTSTATUS) 0xc0000034 - Object Name not found.
```

`0:000>`

找不到对应的文件。那么问题出在哪呢？我们需要进一步调试，我们来看看传递给 `CreateFile` 的参数。

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401010 esp=0012ff2c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x10`:

```
00401010 6810304000      push    0x403010
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401015 esp=0012ff28 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x15`:

```
00401015 ff1504204000    call    dword ptr [temp!__imp__CreateFileA
   (00402004)]{kernel32!CreateFileA (77e7b476)} ds:0023:00402004=77e7b476
```

幸运的是，这是内存中的一个常量，它不太可能会改变，因为我们并没有运行到离 `CreateFile` 太远。

然后，我们可以使用”`da`”，”`dc`”，“`du`”命令。”`da`”命令打印出 ANSI 字符串，”`du`”打印出 Unicode 字符串，”`dc`”和”`dd`”类似，不过它是打印出所有的字符，包括不可显示的。我们知道这是一个 ANSI 字符串，”`da`”命令:

`0:000> da 403010`

```
00403010  "c:MyFile.txt"
```

`0:000>`

我们看到这是错误的字符串，我们应该用 `c:\\MyFile.txt` 来代替它。

因此，我们找到并更正这个错误，但是，我们还是不能写入。我们再进一步调试。

重新载入一遍。

`C:\programs\DirectX\Games\src\Games\temp\bin>cdb temp`

```
Microsoft (R) Windows Debugger  Version 6.3.0005.1
Copyright (c) Microsoft Corporation. All rights reserved.
```

```
CommandLine: temp
Symbol search path is:
  SRV*c:\symbols*http://msdl.microsoft.com/download/symbols
Executable search path is:
ModLoad: 00400000 00404000    temp.exe
ModLoad: 77f50000 77ff7000    ntdll.dll
ModLoad: 77e60000 77f46000    C:\WINDOWS.0\system32\kernel32.dll
ModLoad: 77c10000 77c63000    C:\WINDOWS.0\system32\MSVCRT.dll
(80c.c94): Break instruction exception - code 80000003 (first chance)
eax=00241eb4 ebx=7ffdf000 ecx=00000004 edx=77f51310 esi=00241eb4 edi=00241f48
eip=77f75a58 esp=0012fb38 ebp=0012fc2c iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202
```

`ntdll!DbgBreakPoint`:

```
77f75a58 cc             int     3
```

`0:000> bp temp!main`

`0:000> g`

```
Breakpoint 0 hit
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401000 esp=0012ff50 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main`:

```
00401000 51             push    ecx
```

`0:000> p`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401001 esp=0012ff4c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x1`:

```
00401001 56             push    esi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401002 esp=0012ff48 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x2`:

```
00401002 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401003 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x3`:

```
00401003 33ff            xor     edi,edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401005 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x5`:

```
00401005 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401006 esp=0012ff40 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246
```

`temp!main+0x6`:

```
00401006 57             push    edi
```

`0:000>`

```
eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000
eip=00401007 esp=0012ff3c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc
```



```
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0x7:  

00401007 6a03            push    0x3  

0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401009 esp=0012ff38 ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0x9:  

00401009 57              push    edi  

0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=0040100a esp=0012ff34 ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0xa:  

0040100a 57              push    edi  

0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=0040100b esp=0012ff30 ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0xb:  

0040100b 6800000080      push    0x80000000  

0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401010 esp=0012ff2c ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0x10:  

00401010 6810304000      push    0x403010  

0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401015 esp=0012ff28 ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0x15:  

00401015 ff1504204000 call dword ptr [temp!_imp__CreateFileA (00402004)]{kernel3  

2!CreateFileA (77e7b476)} ds:0023:00402004=77e7b476  

0:000>  

eax=000007e8 ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=00000000 edi=00000000  

eip=0040101b esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei ng nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000293  

temp!main+0x1b:  

0040101b 8bf0            mov     esi,eax  

0:000> p  
```

走到这里，我们看到 `EAX` 是一个有效的 `handle`，继续。

```
eax=000007e8 ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=0040101d esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei ng nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000293  

temp!main+0x1d:  

0040101d 83feff          cmp     esi,0xffffffff  

0:000>  

eax=000007e8 ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401020 esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x20:  

00401020 741b            jz      temp!main+0x3d (0040103d)         

0:000>  

eax=000007e8 ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401022 esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x22:  

00401022 8d442408        lea     eax,[esp+0x8]     ss:0023:0012ff4c=00322cf8  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401026 esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x26:  

00401026 57              push    edi  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401027 esp=0012ff40 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x27:  

00401027 50              push    eax  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401028 esp=0012ff3c ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x28:  

00401028 6a04            push    0x4  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=0040102a esp=0012ff38 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x2a:  

0040102a 6820304000      push    0x403020  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=0040102f esp=0012ff34 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x2f:  

0040102f 56              push    esi  

0:000>  

eax=0012ff4c ebx=7ffdf000 ecx=77f59037 edx=00140608 esi=000007e8 edi=00000000  

eip=00401030 esp=0012ff30 ebp=0012ffc0 iopl=0          nv up ei pl nz ac pe cy  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000213  

temp!main+0x30:  

00401030 ff1500204000 call dword ptr [temp!_imp__WriteFile (00402000)]{kernel32!  

WriteFile (77e7f13a)} ds:0023:00402000=77e7f13a  

0:000> p  

eax=00000000 ebx=7ffdf000 ecx=77e7f1c9 edx=00000015 esi=000007e8 edi=00000000  

eip=00401036 esp=0012ff44 ebp=0012ffc0 iopl=0          nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000          efl=00000246  

temp!main+0x36:  

00401036 56              push    esi  
```

我们调用了 `WriteFile` 并且 `EAX==0`，这意味着调用返回错误，我们来看看其他变量。

第二个参数是对的，长度为 4:

```
0:000> da 403020  

00403020  "Test"  
```

第 4 个参数代表写入的地字节数，为 0。

```
0:000> dd 012ff4c  

0012ff4c  00000000 00401139 00000001 00322470  

0012ff5c  00322cf8 00403000 00403004 0012ffa4  

0012ff6c  0012ff94 0012ffa0 00000000 0012ff98  

0012ff7c  00403008 0040300c 00000000 00000000  

0012ff8c  7ffdf000 00000001 00322470 00000000  

0012ff9c  8053476f 00322cf8 00000001 0012ff84  

0012ffac  e1176590 0012ffe0 00401200 004020c0  

0012ffbc  00000000 0012fff0 77e814c7 00000000  
```

下面我们检查一下 `GetLastError` 的值。

```
0:000> !gle  

LastErrorValue: (Win32) 0x5 (5) - Access is denied.  

LastStatusValue: (NTSTATUS) 0xc0000022 - {Access Denied}   

              A process has requested access to an object,  

              but has not been granted those access rights.  

0:000>  
```

拒绝访问？怎么会这样!检查一下，我们只以 `READ` 权限打开这个文件，没有 `WRITE` 权限，这就是问题所在。然后我们在代码中修改这个错误。

```
hFile = CreateFile("c:\\MyFile.txt", GENERIC_READ,  

        0, NULL, OPEN_EXISTING, 0, NULL);  
```

## 总结

这仅仅是对基本调试技术的一个介绍。例子都很简单，但是其中展示的这些技术都是很有价值的。这是调试教程的第一篇，如果有人感兴趣，我们继续补上一些更加高级的内容。

对有些人来说，这篇教程可能会很简单，对其他人来说又可能太难了。你不可能一天就变成一个熟练的调试专家，这是需要练习的。我建议大家，即使是很简单的问题，也要试着去用调试器解决。练习得越多，工具使用的越熟练，你将会学习到更多。
