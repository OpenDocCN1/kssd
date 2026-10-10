

# 翻译九月 022 深入分析:在 linux 系统 Win32 特洛伊木马逆向工程

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-759.htm`](https://www.pediy.com/kssd/pediy08/pediy8-759.htm)

原文地址  
022.Alien Autopsy, reverse engineering Win32 Trojans on Linux   http://www.lurhq.com/alien.pdf  
由于文件大没法上传  
深入分析:在 linux 系统 Win32 特洛伊木马逆向工程  
作者 Joe Stewart，GCIH  
在我的上一篇文章中，敌意代码的逆向工程，描述了在简单 Trojan 木马的基本逆向工程利用的工具以及过程步骤。本文利用（in the wild）流行的 Trojan 提供了一个更详细的逆向工程的检测。同时，本文将讨论一些在 Linux 系统下本质完全为 Windows 代码的逆向工程技术。作为附加的益处，在本文中所利用工具或者是免费软件或者是自由软件。它们是：  
   Wine-UNIX 系统的 Win32API 执行软件  
   gdb-我们最爱 Unix 调试器和反汇编环境  
   IDA Pro 免费版本-Win32 反汇编工具（在 Linux 系统中 20021007 发行版 Wine 环境下运行，也可在其它版本下运行）。  
注意：对于没有阅读前一篇文章即 Reverse Engineering Hostile Code 且已经不具备 C 和汇编语言的知识的读者，最好就此停止并阅读该文，而读 Reverse Engineering Hostile Code。  
捕获静态代码列表  
静态代码列表（a deadlisting） 仅仅是存储木马的汇编语言代码的堆栈。我们将利用 IDA Pro 的免费版本完成该项任务。这个软件标注了大量的交叉参考跳转，调用以及字符串，确实使汇编代码更容易读懂。最成功的是它现在能运行在 Linux 系统中 Wine 环境下。你仅仅需要从数据恢复网站下载这个版本并将它解压在一个空目录下。然后返回执行解压 IDA 的目录并从 Wine 源代码的程序的子目录下运行 Wine 命令窗口（utility）wcmd。  
如图 1 所示，打开了一个新的控制台窗口。这个环境与 Windows 系统中的 cmd.exe 或者 COMMAND.COM 比较相似。如果你已启动了 wcmd 并在 IDA Pro 目录下，现在只需在 wcmd 的窗口中简单键入“idaw.exe”并敲回车。如果 IDAPro 成功启动，你就能看到 wcmd 窗口发生变化为如图 2 所示。你现在已经在 Linux 中运行了 IDAPro。  
如果点击 OK，将看见文件选择对话框。浏览目录树寻找你想反汇编的 Win32 可执行文件然后点击 OK。下面，它将要求选择一些文件格式选项。通常默认比较健全的，所以你仅需点击对话框上 OK 即可。IDA 将去分析处理它在可执行文件格式中发现的所有信息，并且当它在分析处理时，工具栏将出现“thinking”的标示。由于文件的大小的不同，这项工作将花费从几秒到几分钟不等的时间。  
  
当 IDA 完成分析后，选择菜单中的“File|Produce output file}Produce LST file”并将文件存盘。现在就可以得到 Win32PE 文件的静态代码列表。  
读这个静态代码列表  
利用任意一个文本编辑器，打开这个存盘.LST 文件，并找到这个程序的入口点。它通常不在文件的顶部；你可以搜索“start”或者有时“WinMain”。这个木马程序在偏移量 0x40A00C 开始执行。这也就是我们开始读代码的地方。在 DEADLIST 中最重要可用的线索是 Win32API 调用（IDA Pro 已完美为我们标示出来）以及字符串的偏移量。下面就是主子程序的开始部分，所有的初始 API 调用都一样：  
CODE:0040A00C   
CODE:0040A00C ; UUUUUUUUUUUUUUU S U B R O U T I N E   
UUUUUUUUUUUUUUUUUUUUUUUUU   
CODE:0040A00C   
CODE:0040A00C ; Attributes: bp-based frame   
CODE:0040A00C   
CODE:0040A00C                 public start   
CODE:0040A00C start           proc near   
CODE:0040A00C   
CODE:0040A00C var\_10          = dword ptr -10h   
CODE:0040A00C   
CODE:0040A00C                 push    ebp   
CODE:0040A00D                 mov     ebp, esp   
CODE:0040A00F                 add     esp, 0FFFFFFF0h   
CODE:0040A012                 push    ebx   
CODE:0040A013                 push    esi   
CODE:0040A014                 push    edi   
CODE:0040A015                 xor     eax, eax   
CODE:0040A017                 mov     \[ebp+var\_10\], eax   
CODE:0040A01A                 mov     eax, offset dword\_0\_409FCC   
CODE:0040A01F                 call    sub\_0\_404F74   
CODE:0040A024                 mov     edi, offset unk\_0\_40C7B8   
CODE:0040A029                 xor     eax, eax   
CODE:0040A02B                 push    ebp   
CODE:0040A02C                 push    offset loc\_0\_40A15E   
CODE:0040A031                 push    dword ptr fs:\[eax\]   
CODE:0040A034                 mov     fs:\[eax\], esp   
CODE:0040A037                 push    offset unk\_0\_40C604   
CODE:0040A03C                 push    2   
CODE:0040A03E                 call    j\_WSAStartup  
 j\_WSAStartup 是一个局部子程序作为 Win32API 调用的封装程序。如果是这样，我们分析程序这个可能是为网络通讯创作一个接口。  
 CODE:0040A043                 push    6   
CODE:0040A045                 push    1   
CODE:0040A047                 push    2   
CODE:0040A049                 call    j\_socket   
CODE:0040A04E                 mov     ebx, eax   
CODE:0040A050                 mov     ds:word\_0\_40C794, 1   
CODE:0040A059                 mov     ds:word\_0\_40C796, 0   
CODE:0040A062                 push    4   
CODE:0040A064                 push    offset word\_0\_40C794   
CODE:0040A069                 push    80h   
CODE:0040A06E                 push    0FFFFh   
CODE:0040A073                 push    ebx   
CODE:0040A074                 call    j\_setsockopt   
还有两个封装的 API 调用，setsockopt。我们将按照这样方式一步一步查找 Win32API 调用，所以让我们来讨论这个区域的资源。有些关于这个主题的好书，你可以在线找到这些书，当想深入研究这些代码时，MSDN 可检索文档是一个最好的资源。   
查找 socket 函数的原型，我们发现这些程序将作如下的调用准备：  
SOCKET socket(int af, int type, int protocol);    
在汇编层次来讲，这些变量将传递给系统并由系统以先进后出的原则压入堆栈。关注我们找到的上面偏移量为 0x40A0043 的开始指令行，仅仅在 j\_socket 调用的前面：  
CODE:0040A043                 push    6   
CODE:0040A045                 push    1   
CODE:0040A047                 push    2   
CODE:0040A049                 call    j\_socket   
因此，当我们调用 socket 时，af 值为 2，type 值为 1 并且 protocol 值为 6。参考 MSDN 帮助文件我们发现 af 值为 2 代表 AF\_INET，即网络地址（famliy）。socket 中 type 值为 1 被定义为 SOCK\_STREAM,并且 protocol 为 6 表示 TCP。因此我们的木马建立一个 TCP 接口。我们仍然不知道这个接口在那个端口开放，也不知道它是客户机或者监听程序。继续阅读这个静态代码列表。  
CODE:0040A074                 call    j\_setsockopt   
CODE:0040A079                 mov     ds:word\_0\_40C798, 2   
CODE:0040A082                 call    sub\_0\_402700   
CODE:0040A087                 dec     eax   
CODE:0040A088                 jnz     short loc\_0\_40A0A5   
CODE:0040A08A                 lea     edx, \[ebp+var\_10\]   
CODE:0040A08D                 mov     eax, 1   
CODE:0040A092                 call    sub\_0\_402760   
CODE:0040A097                 mov     eax, \[ebp+var\_10\]   
CODE:0040A09A                 call    sub\_0\_405EA0   
CODE:0040A09F                 mov     ds:word\_0\_40B2C0, ax   
CODE:0040A0A5   
CODE:0040A0A5 loc\_0\_40A0A5:                          ; CODE XREF: start+7C^Xj   
CODE:0040A0A5                 mov     ax, ds:word\_0\_40B2C0   
CODE:0040A0AB                 push    eax   
CODE:0040A0AC                 call    j\_htons   
CODE:0040A0B1                 mov     ds:word\_0\_40C79A, ax   
CODE:0040A0B7                 push    10h   
CODE:0040A0B9                 push    offset word\_0\_40C798   
CODE:0040A0BE                 push    ebx   
CODE:0040A0BF                 call    j\_bind   
这个 bind 调用的原型如下：  
int bind(SOCKET s, const struct SOCK\_ADDR\* name, int namelen);    
因次这里，EBX 寄存器包含了一个指向 socket 的指针，并且数据段的偏移量 0x40C798 应当包含我们的 16 个字节长的 SOCK\_ADDR 结构。TCP 中 SOCK\_ADDR 结构的定义如下：  struct sockaddr\_in {   
    short sin\_family;   
    u\_short sin\_port;   
    struct in\_addr sin\_addr;   
    char sin\_zero\[8\];   
};   
在这个结构中 sin\_port 包含了端口号.它是以网络字节序（big endian）存储的，所以如果端口号存储在一个变量中，则必须它必须从 x86‘s 的正常的主字符序（里 little-endian）。十分确定的是，如果我们返回看 0x40A0AC 偏移量时，看见 API 调用 htons，这个调用将 16-位的短整数从主字符序转化为网络字符序。因此调用的返回量（在 0x40A0AC 的 EAX）是我们的端口号。为了看到我们的端口变量如何变化的，让我们跳转并查看 htons 调用的前面：  
CODE:0040A09A                 call    sub\_0\_405EA0   
CODE:0040A09F                 mov     ds:word\_0\_40B2C0, ax  
CODE:0040A0A5                 mov     ax, ds:word\_0\_40B2C0  
CODE:0040A0AB                 push    eax   
CODE:0040A0AC                 call    j\_htons   
追溯到 0x40A0AC 跟踪 EAX 的原始值，表明其中包含在 0x40A09A 调用的字函数 0x405EA0 的返回值。这就意味着我们必须追溯至少一个子函数，或许更多，找到端口号。但是我们并不介意作所有那些工作，因此我们从另外一种方法得到那些信息。从现在起，让我们返回原来的地方并继续：  
CODE:0040A0C4   
CODE:0040A0C4 loc\_0\_40A0C4:                          ; CODE XREF: start+125^Yj   
CODE:0040A0C4                 push    5   
CODE:0040A0C6                 push    ebx   
CODE:0040A0C7                 call    j\_listen   
CODE:0040A0CC                 mov     dword ptr \[edi\], 10h   
CODE:0040A0D2                 push    edi   
CODE:0040A0D3                 push    offset unk\_0\_40C7A8   
CODE:0040A0D8                 push    ebx   
CODE:0040A0D9                 call    j\_accept   
在这个地方,accept 调用说明我们将成为一个监听器。现在我们最想知道就是在那个端口。  
运行代码:wine+gdb  
这个 Wine 工程已经将大量的 Win32 API 调用装载入 Linux 系统中。这就使得我们可以在 Linux 中运行我们的木马，并且如果它本质上是 Linux 可执行的，就可以利用 gdb 来调试。Wine 也有它自己的调试器，也能完成同样调试目的；然而，gdb 的功能非常丰富。唯一的缺点是利用 gdb 调试前你必须首先在 Wine 代码跟踪方法，而不能直接在 Win32 代码下操作。在下面我会介绍一对断点可以使你快速跳过 Wine 常规方法直接找到 Win32 程序的入口点。  
  
不能忘记逆向未知/恶意代码的基本规则。Always work on non-production machines, on a network segment specifically designed to contain the trojan's traffic. 安全忠告到此结束，利用到 Wine 可执行文件的路径作为 gdb 的变量，启动 gdb。在我的系统上，我将运行如下命令：  
gdb /usr/bin/wine.bin   
接下来将产生下面的结果。（敲入命令回车后，在 gdb 启动后在下面以粗体显示，红色内容是作者的笔记）  
GNU gdb 5.2.1-2mdk (Mandrake Linux)   
Copyright 2002 Free Software Foundation, Inc.   
GDB is free software, covered by the GNU General Public License, and you are    
welcome to change it and/or distribute copies of it under certain conditions.    
Type "show copying" to see the conditions.   
There is absolutely no warranty for GDB.  Type "show warranty" for details.    
This GDB was configured as "i586-mandrake-linux-gnu"...(no debugging symbols found)...   
(gdb) set args ./trojan.exe   
        为 gdb 提供一个变量；载入木马名。  
(gdb) b PROCESS\_InitWine   
翻译(Set a breakpoint when Wine begins. This is  to allow us to set breakpoints later on which are )  
currently   
inaccessible since the code has not been loaded into memory yet)   
在 wine 开始运行时，设置断点。因为这些代码还没有载入内存中，这样就允许以后我们在不可接近的代码处设置断点。  
断点 1：0x80486d0  
（gdb）运行  
程序运行  
启动程序: /usr/bin/wine.bin ./trojan.exe   
(no debugging symbols found)...(no debugging symbols found)...   
(no debugging symbols found)...(no debugging symbols found)...   
(no debugging symbols found)...(no debugging symbols found)...   
Breakpoint 1, 0x400eea46 in PROCESS\_InitWine () from /usr/lib/libntdll.dll.so   
(gdb) b SYSDEPS\_SwitchToThreadStack   
在 Win32 可执行程序入口点设置断点。在到达这个断点后，我们就可以在 0x40a0ab 设置目标断点。  
断点 2 ：0x400f4d66   
（gdb）c  
继续执行这个程序  
继续。  
Could not stat /floppy (No such file or directory), ignoring drive A:   
Breakpoint 2, 0x400f4d66 in SYSDEPS\_SwitchToThreadStack () from /usr/lib/libntdll.dll.so   
(gdb) b \* 0x0040a0ab   
（在 htonsAPI 调用前的偏移地址处设置断点。当在偏移地址处设置断点后，不要忘记偏移地址的前缀星号\*）  
断点 3：0x0040a0ab  
（gdb）c  
（继续执行程序）  
Continuing.   
Breakpoint 3, 0x0040a0ab in ?? ()   
（我们已找到最后一个断点并且停止在木马的主子程序，这个子程序仅仅就在 htons 调用之前。EAX 中应当包含我们的端口号）  
(gdb) print/x (short) $eax   
（显示出寄存器 EAX 中的值）  
$1 = 0x4b8c   
（我们的端口号，以十六进制数表示。0x4b8c 转化成十进制为 19340）  
（gdb）quit  
这样，不需要在 deadlisting 跟踪无数的子程序，我们就可以确定出木马将在端口 19340 开放一个监听器。  
coroner 的报告  
这里，我们仍可继续读 the deadlisting 并通过设置断点直到程序的每个函数都运行后，来获取 gdb 中的变量内存信息填充我们对程序认识的空白。很明显，这篇短文不能涉及到即使一个很小的木马的所有函数，但令人欣慰的是，它为你提供了如何自己亲自去分析木马的信息，使得仅需一点汇编、C 知识、一点运气以及再多一点耐心就可以征服一个木马。刚开始，你看到 thousands upon thousands 数千汇编代码后，就感到困惑，但实际上可以凭经验容易地来处理这些代码。最终你将会从整体角度找到程序的范式，从而从一行一行代码分析中解脱出来，似乎就好像是你不用读代码就能‘感受’到它。  
关于木马  
上面代码是从建立在约定系统上的 Win32 后门得到的。连接监听器将产生一条提示“Barvinok NT Shell v1.0”以及密码输入框。密码是以二进制形式的硬编码并可以在“字符串”输出是可读的。在键入密码后，你就能进入基本的 NT 命令行核。 

 4dd466d0e1e0cc123afbe504da2462ca