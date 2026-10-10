

# OllyDbg 的 help

> 原文：[`www.pediy.com/kssd/pediy06/pediy6862.htm`](https://www.pediy.com/kssd/pediy06/pediy6862.htm)

OllyDbg 的 help-怎样开始调试（翻译）  
  
最简单的方法是启动 OllyDbg，点击 File|Open，然后选择你想调试的程序。程序需要命令行参数输入对话框下方的文本栏。  
重新开始调试最后一个程序的快捷键是 Ctrl+F2，并且 OllyDbg 使用相同的参数。  
你也可以点选历史记录。把程序拖入 OllyDbg 也可以开始调试。  
当然，当启动 OllyDbg 时，你在命令行中也能指定被调试的程序名和参数。比如：  
你可以创建桌面快捷方式指向 OllyDbg，选择属性，到快捷方式，把程序名加入目  
标栏。每次你双击这个快捷图标，OllyDbg 自动装载被调试程序。  
你可以 attach OllyDbg 到某个正在运行的进程。点击 File|Attach，从列表中选择  
该进程。注意：当你关闭 OllyDbg，这个进程也会终止。不要试图 attach 系统进程  
，这很可能使系统完全死机。（事实上，大多数情况下，OS 不允许 attach 敏感进  
程）  
OllyDbg 能作为 just-in-time debugger。这需要在注册表中登记。点击  
Options|Just-in-time debugging，在对话框中按“Make OllyDbg just-in-time   
debugger”。现在，当一些程序崩溃，你会被询问是否调试它。这样，操作系统  
可以启动 OllyDbg 并直接停在异常发生处。如果你选择 attaching without   
confirmation，OllyDbg 不会询问直接启动。要恢复原来的 just-in-time   
debugger，在出现的对话框中按相应的按纽就行了。  
其它方法还有，把 OllyDbg 加入可执行文件的弹出式菜单。点击 Options|Add to   
Explorer，按“Add OllyDbg to menu in Windows Explorer”。以后你可以右击  
可执行文件点选 OllyDbg。这个选项创建注册表键  
HKEY\_CLASSES\_ROOT\\exefile\\shell\\Open with OllyDbg and  
HKEY\_CLASSES\_ROOT\\exefile\\shell\\Open with OllyDbg\\command  
OllyDbg 可以调试控制台（基于文本）程序。  
注意：WindowsNT 或 Windows2000 下，你必须有管理员权限  
OllyDbg 的 help-通用快捷键（翻译）  
Golbal shortcuts（通用快捷键）  
这些快捷键在 OllyDbg 中通用，不依赖于当前活动窗口。  
Ctrl+F2---OllyDbg 重置，重新开始调试。如果没有活动程序，OllyDbg 装入历史列表中的第一个程序。OllyDbg 重置会释放内存移除硬断点。  
Alt+F2－－关闭被调试的程序。如果程序还在活动状态，你会被询问是否执行该操作。  
F3－－显示“Open 32-bit .exe file”对话框，这里可以选择可执行文件和指定参数  
Alt+F5－－使 OllyDbg 显示在屏幕最前方。如果被调试程序中断时显示窗口遮住 OllyDbg 的一些区域，不继续运行又不能移动或最小化它。激活 OllyDbg 按 Alt+F5 可以解决。如果你再次按 Alt+F5，OllyDbg 会恢复为普通窗口。OllyDbg 的当前状态显示在状态栏。  
F7－－step into，执行下一条简单命令。如果该命令是一个函数调用，进入调用函数内部。如果命令有 REP 前缀，执行该命令的一步操作。  
Shift+F7－－与 F7 相同，除了当被调试程序遇到某些异常时，首先尝试调用程序自己的异常处理过程。  
Ctrl＋F7－－animate into，一步一步执行程序，也进入函数调用（就象你一直按着 F7，不过更快）。当你执行一些单步或继续命令，程序到达有效断点，一些异常发生时，Animation 终止。每步执行，OllyDbg 重画所有窗口。要加速 animation，关闭所有窗口而不要只改变现有窗口的大小。终止 animation，也可按 Esc。  
F8－step over，执行下一条简单命令。如果该命令是一个函数调用，立刻执行完该函数（除非函数内部有断点或产生异常）如果命令有 REP 前缀，执行所有重复操作，并停于下一条命令。  
Shift+F8－－与 F8 相同，除了当被调试程序遇到某些异常时，首先尝试调用程序自己的异常处理过程。  
Ctrl+F8－－animate over，一步一步执行程序，但不进入函数调用（就象你一直按着 F8，不过更快）。当你执行一些单步或继续命令，程序 Animation 到达有效断点或异常发生时，Animation 终止。每步执行，OllyDbg 重画所有窗口。要加速 animation，关闭所有窗口而不要只改变现有窗口的大小。终止 animation 也可按 Esc。  
F9－－继续执行程序。  
Shift+F9－－与 F9 相同。除了当被调试程序遇到某些异常时，首先尝试调用程序自己的异常处理过程。  
Ctrl+F11－－run trace into，一步一步执行程序，要进入函数调用，记录寄存器内容。Run trace 不重绘 CPU 窗口.  
F12-悬挂所有线程以停止程序执行。最好用继续键和菜单命令（像 F9）恢复执行线程，而不要用其它手动方法恢复。  
Ctrl+F12－－run trace over，一步一步执行程序，不进入函数调用，记录寄存器内容。Run trace 不重绘 CPU 窗口。  
ESC－－如果 animation 或跟踪正在进行，将被终止。如果 CPU 窗口显示跟踪时的数据，此时转为显示实际数据。  
Alt+B－－打开或恢复 Breakpoint 窗口。这里可以编辑，删除和观察断点。  
Alt+C－－打开或恢复 CPU 窗口。  
Alt+E－－打开或恢复模块列表  
Alt+E－－打开或恢复 Call stack 窗口。  
Alt+L－－打开或恢复 Log 窗口。  
Alt+M－－打开或恢复 Memory 窗口。  
Alt+O－－打开 Option 对话框  
Ctrl+P－－打开 Patch 窗口。  
Ctrl+T－－打开 Pause run trace 对话框  
Alt＋X－中断 OllyDbg。  
多数窗口可以使用下列快捷键：  
Alt+F3－－关闭活动窗口。  
Ctrl+F4－－关闭活动窗口。  
F5－－最大化或恢复活动窗口。  
F6－－激活下一个窗口。  
Shift+F6－－激活前一个窗口。  
F10－－打开激活窗口或面板的右键菜单。  
LeftArrow－－左移一个字符。  
Ctrl+LeftArrow－－左移一行。  
RightArrow－－右移一个字符。  
Ctrl+RightArrow－－右移一行。   
OllyDbg 的 help－分析模块介绍（翻译）  
分析  
OllyDbg 集成了快速超强的代码分析器。装载它，可以用弹出式菜单或者 CPU 窗口  
的反汇编栏按 Ctrl+A 或者在执行模块选“Analyze all modules”。  
分析器非常有用。它在数据中分辨代码，标记入口点和 jump 的目标，辨认 switch   
tables，ASCII 和 UNICODE 字符串，定位过程、循环、高级 switch 语句和解码标准  
API 函数参数（看范例）。OllyDbg 的其它部分也广泛用于数据分析。  
这怎么可能呢？我稍微介绍一下原理。首先，OllyDbg 反汇编代码区所有可能的地  
址，记下所有发现的调用及指向的目标。当然，许多这样的调用不正确，但是未  
必会有两个错误的调用指向同一条命令，并且三个或三个以上更不可能。这样，  
三个或更多调用指向同一地址，我就确信该地址是经常使用的子过程入口。从这  
个入口开始，我跟踪所有的跳转和调用，反复操作。这种方法使我 99.9％确定所  
有命令。然而，某些字节不在这条链中，我采用大约 20 个更高效的方法（最简单  
的如：“MOV \[RA\],Ra"是无效命令).  
过程检测也简单。过程，从分析器的角度看，就是从入口开始的代码的连续区域  
，（理论上）可以到达其它命令（除 NOP 或对齐填充外）。你可以指定三种识别级  
。Strict 级严格确认一个入口和至少一个出口。heuristical 级，分析器尝试确认  
入口。如果你选择 fuzzy 级，或多或少相容的代码区被分离为过程。现代编译器把  
过程分成几个部分作公用代码优化。这种情况下，fuzzy 级特别有用！然而，误解  
的概率也相当高。  
类似的，loop 是一个封闭的连续命令序列，这里最后的命令跳到开头、一个入口  
几个出口。Loop 相当于高级操作语言中的 do,while 和 for。OllyDbg 能认识任何复  
杂的嵌套循环。在反汇编区，他们用长长的大括号标记。如果入口不是循环的第  
一条命令，OllyDbg 用个小三角形作标记。  
实现 switch 语句，多数编译器把 switch 变量装入寄存器，然后减去一部分，像下  
面代码序列一样：  
    MOV EDX,<switch variable>  
    SUB EDX,100  
    JB DEFAULTCASE  
    JE CASE100        ; Case 100  
    DEC EDX  
    JNE DEFAULTCASE  
    ...               ; Case 101  
这些序列也可能包含一级或两级 switch 表，直接比较，优化，填充等等。如果你  
深入研究比较和跳转树，很难说哪一条命令是某个 case 语句。OllyDbg 为你代劳。  
它标记所有的 case 语句，包括 default，甚至尝试猜测 case 的意思，如'A',   
WM\_PAINT 或者 EXCEPTION\_ACCESS\_VIOLATION。如果命令序列不修改寄存器（只  
由比较命令构成），那么不大可能是 switch 语句，但可能是 if 嵌套：  
    if (i==0) {...}  
    else if (i==5) {...}  
    else if (i==10) {...}  
让 OllyDbg 解码 if 嵌套为 switch，选择 Analysis1 中相应的选项。  
OllyDbg 预置超过 1900 个常用 API 函数的描述。包括 KERNEL32, GDI32, USER32,   
ADVAPI32, COMDLG32, SHELL32, VERSION, SHLWAPI, COMCTL32, WINSOCK,   
WS2\_32 和 MSVCRT。你也可以加入自己的描述。如果分析器遇到已知函数名的调  
用（或跳转到该函数），它尝试解码调用跟前的 PUSH 命令。因此，你可以粗略翻  
译该调用的功能。OllyDbg 也内置大约 400 个标准 C 函数的描述。如果你利用新库，  
我建议你分析前先扫描对象文件。这种情况下，OllyDbg 也会解码已知的 C 函数参  
数。  
如果选项“Guess number of arguments of unknown functions”被设置，分析  
器尝试确认调用过程进栈的双字数目，并标记他们为参数 Arg1,Arg2 等等。注意：  
如果有寄存器参数，OllyDbg 还是不认识也不包括在上面的统计中。分析器采用了  
一个安全的方法。例如，它不分辨无参数过程和返回前用 POP 恢复寄存器代替舍弃  
参数的 case 语句。然而，能分辨出的函数数目相当多，并非常有助于提高代码的  
可读性。  
分析器能跟踪寄存器的值。现代优化编译器，特别是面向 Pentium 的，经常把常量  
和地址装入寄存器便于重复使用或减小内存占用空间。如果一些常量装入寄存器  
，分析器会注意它，并用于解码函数及其参数。还能执行简单的算术计算跟踪  
push 和 pop 命令。  
分析器不能区别不同类型的名字。如果你用已有的名字命名一些函数，OllyDbg 会  
解码所有该地址的调用为原过程。WinMain，DllEntryPoint 和 WinProc 是特殊的预  
定义名。你可以使用这些标号标记主程序入口，DLL 入口指针和 window 过程（注意  
：OllyDbg 不会检查用户定义标号的唯一性）。当然，最好的方法是显示已定义的  
参数。  
非常不幸，没有一般的规则用于 100％的准确分析。有些情况下，例如当模块包含  
p-代码或者代码区嵌入大量数据，分析器可能认为部分数据是代码。如果统计分  
析显示代码可能被打包或加密，分析器会警告你。如果你想用 hit 跟踪方式，我建  
议你不要用 fuzzy 分析方式，否则断点被设置在被误认为代码的数据上的几率很高。  
自解压文件通常在主体代码外有解压代码。如果你选择 SFX 选项“Extend code   
section to include self-extractor”，OllyDbg 会扩大代码区，形式上允许分  
析它并跟踪。   
OllDbg 的一般原理（翻译）部分  
我希望你熟悉 80x86 兼容 CPU 的内部结构，并且有汇编写程序的经历。我也希望你  
熟悉 Microsoft Windows.  
OllyDbg 是一个单进程多线程的“机器代码级”debugger，用于 Windows 环境下的 32 位程序。它允许你 debug 和 patch PE 格式的可执行程序。OllDbg 仅仅使用列入文档的 Win32 API 调用，所以可用于下一代 32 位 Windows 系统。OllDbg 看来也可工作于 Windows XP，但是我没有详尽的测试，因此不能保证功能完整。  
OllyDbg 不是面向编译器。它不含特别的规则显示某些情况下特定的编译程序生成哪些代码序列。因此，你可以一样对待任何编译器编译的，或者汇编书写的任何代码。  
OllyDbg 与被调试的程序同时工作。你可以浏览代码和数据、设置断点、停止或继续线程，甚至运行期修改内存（有时这叫作软调试方式）。当然，如果被请求的操作不是最基本的，OllyDbg 就会短时暂停程序，但这对用户透明。有时不在调试状态运行的程序会意外崩溃。OllyDbg，这个“及时”debugger，会指出异常发生的位置。  
OllyDbg 强烈面向模块。模块这里指启动时加载的或动态加载的主执行文件或动态连接库。在调试区，你可以设置断点，定义新标号和注释汇编语句。当一些模块从内存卸载后，OllyDbg 保存这些信息到扩展名为.UDD 名字同被调试模块的文件中。下次当装载这些模块时，OllyDbg 自动复原所有调试信息，不论程序是否使用这些模块。比如：你调试使用 Mydll 的程序 Myprog1，并且在 Mydll 中设置一些断点。那么当你调试 Myprog2 时，也使用 Mydll，你会发现所有在 Mydll 中的断点还在那里，不管 Mydll 是否装载在不同的位置。  
一些调试器把被调试进程的内存视为单个 2\*\*32 字节区域。OllyDbg 做了别的处理方法。内存由几个独立的块组成。任何内存操作都限制于块内。在大多数案例中，这工作优良并且容易调试。但是模块包含几个执行部分等等，你将不能立刻看到整个代码。然而这些例外不常见。  
OllyDbg 是内存消耗大户。启动时就要分配 3MB 内存，甚至更多。每次分析，备份，跟踪或文件转储另外再分配。所以当你调试大工程时消耗 40 或 60M 内存很正常。  
要有效的调试一些无源代码的程序，你首先必须理解它是如何工作的。OllyDbg 提供了大量的手段使理解更容易![](img/620448a77d884bd6b2c4a7cfef437501.jpg)   4dd466d0e1e0cc123afbe504da2462ca