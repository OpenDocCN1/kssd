
# 如何显示 Windows 标准"运行"对话框

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-238.htm`](https://www.pediy.com/kssd/pediy09/pediy09-238.htm)

> 对于喜欢以命令行方式执行程序人来说，Windows 中的标准“运行”对话框一定是再熟悉不过了。当我们看到 Windows 自带的任务管理器以及著名的 Process Explorer 中都能产生这个对话框时，我们会不会联想到这本身就是 Windows 提供的一个功能呢？作者深入分析了这个问题，找到了一个只以序号方式导出的未公开的 API。本文对反向工程初学者也会有一定帮助。

> 注：附件中是本文的 PDF 格式，可读性好,下载：[`bbs.pediy.com/attachment.php?attachmentid=7372&d=1187391359`](http://bbs.pediy.com/attachment.php?attachmentid=7372&d=1187391359)

## 探索如何显示 Windows 标准“运行”对话框

作者：SmartTech

我们有时在编写一些实用工具时可能想要显示一个类似与 Windows 中的标准“运行”对话框。虽然我们可以自己手工实现这个对话框，但是标准的“运行“对话框中却可以列出你曾经键入的命令。并且，最后一次输入的命令，如果它能够正常运行，那么它总是出现在组合框的最上端。在 Windows 中很多数据都是保存在系统注册表中的，我们当然有理由认为像这样的数据就被保存在注册表中。通过键入一些可用命令并在注册表中搜索，我们可以找到这个存储位置为 `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU` 这个子键（SubKey）。这个子键中的每个值（Value）中的数据（Data）就是你曾经键入的命令。其中有个值 `MRUList` 保存的是你键入的命令的顺序。不过它保存的是键入的相反顺序，也就是说，最后一次键入并成功运行的命令总是出现在这个值的数据中的第一个位置。当然，我们可以枚举 `RunMRU` 这个子键然后将曾经运行过的各个命令添加到自己手工实现的运行对话框中的组合框中，并分析 `MRUList` 这个值中的数据来对枚举出的各个命令字符串进行排序。这样就能完全模仿系统的“运行”对话框。这不失为一种方法。但是，一定有许多人像我一样，确实想知道到底系统是如何显示这个对话框的。那就让我们一步一步来分析吧！

在调用这个对话框的系统程序中，我们首先想到就是系统 Shell（即 `explorer.exe`）。本想拿它来做实验，但是它太大，也太复杂，并不合适。接下来能想到的程序可能就是任务管理器了（即 `taskmgr.exe`）。当你启动任务管理器后，单击“文件”菜单，就会看到“新建任务”命令，单击它就会出现“运行“对话框。另外一个会显示“运行”对话框的程序就是下面要提到的 Process Explorer，并且它的“运行”对话框有标准的，也有定制的。那任务管理器是如何显示这个对话框的呢？这里，我要提到一个工具，那就是 Process Explorer，它是由著名的 Windows NT 黑客 Mark E. Russinovich 编写的。现在 Mark 已经进入 Microsoft，成为核心开发成员之一。它的网站 `www.sysinternals.com`（现在已经合并到 Microsoft 网站中）中免费公开许多实用工具（甚至它们的源代码），像著名的 `FileMon` 和 `RegMon` 等等。在他与 David A. Solomon 合著的《Inside Windows 2000, Third Edition》中，Solomon 曾经提到 Mark 不看 Windows 源代码，仅用 `ntoskrnl.exe`（Windows 执行内核）的反汇编代码并配合 `SoftICE` 就能很快解决遇到的问题。就连 Windows NT 之父 David N. Cutler 也佩服他竟然能在正在运行的 Windows 上运行内核调试器（Microsoft 虽然支持内核调试，但是一定要在启动选项加入 `/DEBUG` 并重新启动之后才能有效地使用内核调试器），以及能让 Windows 9x 支持 NTFS 分区。这些都是十分棘手的问题。否则，Microsoft 没有必要不自己实现。Mark 的水平由此可见一斑。Process Explorer 本来就是被设计用来替代任务管理器的，它实现了任务管理器的一切功能，并且又加入了许多有用的功能。今天我们要用到的就是其中的一个功能。

### 寻找切入点

当任务管理器显示“运行”对话框后，我们启动 Process Explorer，双击其中的 `taskmgr.exe` 进程，在弹出的属性对话框中单击“Thread”选项卡，如果你正确配置了系统文件的调试符号，你会看到 `taskmgr.exe` 的各个线程。在“Start Address”一栏中，双击最上面的 `taskmgr.exe!ModuleEntry`，会弹出一个关于这个线程的堆栈的对话框，查找此对话框，我们会看到这样两行：

```
SHELL32.dll!RunFileDlg+0xc4
taskmgr.exe!RunDlg+0x99
```

只所以会注意到这两行，是因为只有它们与运行（Run）对话框（Dialog，在 Windows 编程中常简称为 Dlg）有关。这表明 `taskmgr.exe` 中的 `RunDlg` 函数调用了 `Shell32.dll` 中的 `RunFileDlg` 函数。由于各种程序（`explorer.exe`、`taskmgr.exe`，以及 Process Explorer）所显示的“运行”对话框基本一样，所以我们有理由猜测它是由 Windows 自身实现的标准对话框。由上面堆栈中的两行内容我们可以初步猜测系统中提供的标准“运行”对话框可能是由 `Shell32.dll` 中的 `RunFileDlg` 函数实现的。如何验证我们的想法呢？首先，我们可以查看 `Shell32.dll` 的导出函数，看其中是否有 `RunFileDlg` 函数。令人遗憾的是，`Shell32.dll` 并没有导出以这个名字命名的函数，但是它却导出了许多没有名字的函数，也就是说，只能以序号来调用这些函数。根据 Microsoft 向来好隐藏一些 API 函数这一点，我们可以猜测这个函数可能是仅由序号导出的。为了找出到底是如何调用这个函数的，读者当然可以调试这些程序，或者使用一些监视软件，但是我更喜欢使用 `SoftICE` 来设置断点。我们用 `Loader32` 载入 `Shell32.dll` 的调试符号，然后用 `bpx RunFileDlg` 来设置断点。但是在函数名还没有打完时，按 TAB 键让 `SoftICE` 将命令补全，它却提示“找不到匹配的符号”。由于 Win32 API 都是 `stdcall` 调用约定，而 `stdcall` 调用约定会改写函数名（在前面加上下划线，后面加上@符号以及此函数所需参数的总字节数。要详细了解函数调用习惯及其对函数名称与参数的影响，可以参考 John Robbins 的《Debugging Applications》的 Power Debugging with x86 Assembly Language and the Visual C++ Debugger Disassembly Window 一章），所以猜测这个函数的名称已经被改写，改用 `bpx _RunFileDlg` 来设置断点。在输入 `_RunF` 后按 TAB 键，`SoftICE` 就自动将命令补全了。然后，单击“开始”菜单中的“运行”，`SoftICE` 弹出，中断在函数 `_RunFileDlg` 的入口处。此时观察堆栈，发现最上面一行为 `shell32!_RunFileDlg`。单击 Process Explorer 的“File”菜单下的“Run”，结果与上述情况完全一样。单击“任务管理器”中的“新建任务”，情况依然如此。这就充分说明了 `Shell32.dll` 中确实存在这个产生标准“运行”对话框的函数 `_RunFileDlg`。由任务管理器中弹出的“运行”对话框的标题与图标与其它两个程序不同，这是不是表示这个对话框可以被定制呢？接下来的任务就是找出它的导出序号和各个参数及其功能了。

### 寻找导出序号

我们用 IDA Pro 来反汇编 `taskmgr.exe` 程序。载入调试符号后，观察其 imports 窗口。其中有这样一行：

```
010011C4 61  __imp__RunFileDlg@24                 SHELL32
```

很明显，`RunFileDlg` 函数的导出序号为 61。要验证这一点，可以使用 Visual Studio 2005 自带的 `DUMPBIN` 实用工具。用下面的命令行：

```
DUMPBIN /EXPORTS SHELL32.DLL | more
```

使用管道（`|`）和 `more` 可以让输出满屏后暂停，然后按空格滚动一屏，或者使用回车滚动一行（当然，读者也可以让输出重定向到一个文件再查看）。在其输出中可以看到这样一行：

```
61  00077077 [NONAME] _RunFileDlg@24
```

很明显，`RunFileDlg` 函数的导出序号为 61，并且带 6 个参数（Win32 API 函数的每个参数均为 4 字节宽）。

### 判断参数类型及用途

现在的任务只剩下判断参数的类型和用途了。回过头来看 `taskmgr.exe` 的反汇编代码。在 `taskmgr.exe` 中，是它的 `RunDlg` 函数调用了 `Shell32.dll` 中的 `RunFileDlg` 函数。我们可以根据它对参数的赋值情况来判断 `RunFileDlg` 函数的各个参数。`RunDlg` 函数的反汇编代码如下：

```
.text:01003110                 unsigned long __stdcall RunDlg(void) proc near
.text:01003110
.text:01003110                         var_61C= dword ptr -61Ch
.text:01003110                         var_414= dword ptr -414h
.text:01003110                         Buffer= word ptr -20Ch
.text:01003110                         var_4= dword ptr -4
.text:01003110
.text:01003110 8B FF                   mov     edi, edi
.text:01003112 55                      push    ebp
.text:01003113 8B EC                   mov     ebp, esp
.text:01003115 81 EC 1C 06 00 00       sub     esp, 61Ch
.text:0100311B A1 94 54 01 01          mov     eax, ___security_cookie
.text:01003120 53                      push    ebx
.text:01003121 6A 40                   push    40h     ; UINT
.text:01003123 6A 00                   push    0       ; int
.text:01003125 6A 00                   push    0       ; int
.text:01003127 6A 01                   push    1       ; UINT
.text:01003129 6A 6B                   push    6Bh     ; LPCWSTR
.text:0100312B FF 35 28 5E 01 01       push    HINSTANCE__ * g_hInstance ; HINSTANCE
.text:01003131 89 45 FC                mov     [ebp+var_4], eax
.text:01003134 FF 15 F0 11 00 01       call    ds:LoadImageW(x,x,x,x,x,x)
.text:0100313A 8B D8                   mov     ebx, eax
.text:0100313C 85 DB                   test    ebx, ebx
.text:0100313E 74 72                   jz      short loc_10031B2
.text:0100313E
.text:01003140 56                      push    esi
.text:01003141 57                      push    edi
.text:01003142 8B 3D BC 13 00 01       mov     edi, ds:LoadStringW(x,x,x,x)
.text:01003148 BE 04 01 00 00          mov     esi, 104h
.text:0100314D 56                      push    esi       ; nBufferMax
.text:0100314E 8D 85 F4 FD FF FF       lea     eax, [ebp+Buffer]
.text:01003154 50                      push    eax       ; lpBuffer
.text:01003155 68 11 27 00 00          push    2711h     ; uID
.text:0100315A FF 35 28 5E 01 01       push    HINSTANCE__ * g_hInstance ; hInstance
.text:01003160 FF D7                   call    edi ; LoadStringW(x,x,x,x)
.text:01003162 56                      push    esi       ; nBufferMax
.text:01003163 8D 85 EC FB FF FF       lea     eax, [ebp+var_414]
.text:01003169 50                      push    eax       ; lpBuffer
.text:0100316A 68 12 27 00 00          push    2712h     ; uID
.text:0100316F FF 35 28 5E 01 01       push    HINSTANCE__ * g_hInstance ; hInstance
.text:01003175 FF D7                   call    edi ; LoadStringW(x,x,x,x)
.text:01003177 8D 85 E4 F9 FF FF       lea     eax, [ebp+var_61C]
.text:0100317D 50                      push    eax       ; lpBuffer
.text:0100317E 56                      push    esi       ; nBufferLength
.text:0100317F FF 15 F0 10 00 01       call    ds:GetCurrentDirectoryW(x,x)
.text:01003185 6A 14                   push    14h
.text:01003187 8D 85 EC FB FF FF       lea     eax, [ebp+var_414]
.text:0100318D 50                      push    eax
.text:0100318E 8D 85 F4 FD FF FF       lea     eax, [ebp+Buffer]
.text:01003194 50                      push    eax
.text:01003195 8D 85 E4 F9 FF FF       lea     eax, [ebp+var_61C]
.text:0100319B 50                      push    eax
.text:0100319C 53                      push    ebx
```



```asm
.text:0100319D FF 35 1C 5E 01 01       push    HWND__ * g_hMainWnd
.text:010031A3 FF 15 C4 11 00 01       call    ds:RunFileDlg(x,x,x,x,x,x)
.text:010031A9 53                       push    ebx         ; hIcon
.text:010031AA FF 15 EC 11 00 01       call    ds:DestroyIcon(x)
.text:010031B0 5F                       pop     edi
.text:010031B1 5E                       pop     esi
.text:010031B1
.text:010031B2
.text:010031B2                  loc_10031B2:     ; CODE XREF: RunDlg(void)+2E j
.text:010031B2 8B 4D FC             mov     ecx, [ebp+var_4]
.text:010031B5 33 C0               xor     eax, eax
.text:010031B7 40                   inc     eax
.text:010031B8 5B                   pop     ebx
.text:010031B9 E8 1C EF 00 00       call    __security_check_cookie(x)
.text:010031B9
.text:010031BE C9                   leave
.text:010031BF C3                   retn
.text:010031BF
.text:010031BF                   unsigned long __stdcall RunDlg(void) endp
```

在这个函数中，除了 `prolog` 代码和 `epilog` 代码以及编译器生成的安全检查代码外，主体是对 `RunFileDlg` 这个函数的调用。下面我们来具体分析。

识别出第一个参数

我们先来看 `RunDlg` 函数在调用 `RunFileDlg` 函数之前向堆栈中都压入了哪些参数。压入的最后一个参数（也就是 `RunFileDlg` 函数的第一个参数）已经被 `IDA Pro` 识别出来了。根据匈牙利命名法，它应该是一个全局的窗口句柄（即 `HWND` 类型的变量）。很明显，这里应该是指任务管理器的主窗口句柄。也就是“运行”对话框的父窗口。为了验证这一点，现在来看 `Process Explorer` 的反汇编代码。在其中发现了如下代码：

```asm
.text:0041C780 6A 3D               push    3Dh
.text:0041C782 68 38 56 48 00      push    offset s_Shell32_dll_0 ; "shell32.dll"
.text:0041C787 FF 15 1C 93 46 00   call    ds:LoadLibraryA
.text:0041C78D 50                 push    eax
.text:0041C78E FF 15 24 93 46 00  call    ds:GetProcAddress
.text:0041C794 85 C0               test    eax, eax
.text:0041C796 A3 C0 FA 49 00      mov     dword_49FAC0, eax
.text:0041C79B 74 14               jz      short loc_41C7B1
.text:0041C79B
.text:0041C79D 8B 4C 24 04         mov     ecx, [esp+4]
.text:0041C7A1 6A 00               push    0
.text:0041C7A3 6A 00               push    0
.text:0041C7A5 6A 00               push    0
.text:0041C7A7 6A 00               push    0
.text:0041C7A9 6A 00               push    0
.text:0041C7AB 51                 push    ecx
.text:0041C7AC FF D0               call    eax
.text:0041C7AE 33 C0               xor     eax, eax
.text:0041C7B0 C3                 retn
```

在这段代码中，先用 `LoadLibrary` 函数加载 `Shell32.dll`，获取其模块句柄。然后调用 `GetProcAddress` 函数获取 `Shell32.dll` 中某个导出函数的入口地址，注意到 `3Dh` 就是 `61`，可以肯定它获取的是 `RunFileDlg` 函数的入口地址。再看最后对这个函数的调用，压入的参数除了最后一个外全为 `0`。我们在 `taskmgr.exe` 的反汇编代码中已经知道这最后一个压入的参数可能为窗口句柄。既然其它参数都可以为 `0`，我们先来实验一下。

插曲—我是如何找到上面那段代码的呢？首先在 `IDA Pro` 的 `Names` 窗口搜索字符串“run”，会看到有个字符串名称为 `s_Rundlg`。根据 `IDA Pro` 的命名习惯，这个字符串是“Rundlg”。由前面我们得到的信息，可以猜测它与我们要找的“运行”对话框有关。双击此字符串，`IDA Pro` 转到程序中此字符串的位置处。在此字符串右边的提示信息中，我们可以看到有三段代码使用了这个字符串。如下图所示。

联想到 `Process Explorer` 的“File”菜单中有三个不同的“Run”命令，猜测它们可能就对应着这三段代码。在提示信息的第一行上双击转到相应的代码处。但首先映入我们眼帘的是对 `DialogBoxParam` 函数的三次调用。而“RUNDLG”是这三个模式对话框的资源名称。很明显，`Process Explorer` 是自己实现“运行”对话框的。既然这样，那为什么刚才我们在 `SoftICE` 中设置断点时它会被中断呢？向前滚动几行，我们就会看到前面的那段代码。原来，`Process Explorer` 首先尝试获取 `Shell32.dll` 提供的标准“运行”对话框，如果失败，它就自己生成此对话框。而

```asm
jz      short loc_41C7B1
```

这条指令就是在获取系统的“运行”对话框失败时转到自己生成对话框的代码处。如果你仔细观察由系统生成的“运行”对话框与 `Process Explorer` 生成的“运行”对话框就会发现，前者中的组合框中始终都显示滚动条，即使不需要也显示为灰化状态，而后者中的组合框中在不需要时并不显示滚动条。

实验程序如下：

`RunFileDlg1.cpp` 程序

```cpp
#define WIN32_LEAN_AND_MEAN
#ifdef UNICODE
#undef UNICODE
#endif

#include <windows.h>

char szShell32[]="shell32.dll";

#pragma comment(linker, "/entry:main")

void __declspec(naked) main()
{
  __asm
  {
    xor eax, eax
    push eax
    push eax
    push eax
    push eax
    push eax    //以上是 RunFileDlg 函数的最后五个参数

    call DWORD PTR [GetDesktopWindow]
    push eax    //这是它的第一个参数

    mov eax, offset szShell32
    push eax
    call DWORD PTR [LoadLibrary]

    push 3Dh

    push eax
    call DWORD PTR [GetProcAddress]
    call eax    //eax 寄存器中即为 RunFileDlg 函数的入口地址
    call DWORD PTR [ExitProcess]
  }
}
```

编译后运行它，你会看到桌面左上角真的出现了标准的“运行”对话框！（笔者使用的是英文版的 `Windows XP SP2`）如图 1 所示。

图 1 `RunFileDlg1` 运行结果

根据 `Windows` 命名习惯，可以将此参数命名为 `hwndParent`。那现在要问，这个参数是否可以为 `NULL` 呢？我们将 `RunFileDlg1.cpp` 中的

```asm
call DWORD PTR [GetDesktopWindow]
```

这一句注释掉，重新编译运行，结果同上。这表明，这个函数的所有参数都可以为 `NULL`。也就是说，只要给这个函数传递六个 `NULL` 作为参数，它就会显示一个 `Windows` 标准“运行”对话框！可能还有比较细心的读者会说，在你的例子中并没有对函数返回值进行检查，如果函数运行失败，而它的返回值刚好是 `NULL`，那以上两种情况不就一样了吗？对于这样简单的程序，验证返回值的语句说不定比有用的语句还多，所以我没有对返回值进行验证。如果读者想确认，方法如下：单步跟踪到

```asm
call DWORD PTR [GetDesktopWindow]
```

这条指令执行完毕，此时观察 `EAX` 寄存器的值，在笔者的机器上，它的值为 `00010014`。这时，打开 `Spy++`，你会看到最上面一行显示的桌面的窗口句柄也为此值，如下图所示。

此证明笔者所言不虚。

识别第二个参数

现在再来看其它五个参数。在 `taskmgr.exe` 的反汇编代码中，倒数第二个被压入堆栈中的参数是 `ebx`,分析前面 `ebx` 寄存器中值的变化，发现 `ebx` 中的值是调用 `LoadImageW` 函数的返回值。而 `LoadImageW` 函数的第三个参数为 `1`，也就是 `IMAGE_ICON`，实际上这个函数就是加载一个图标，`ebx` 中就是此图标的句柄。此时再看其第二个参数。根据 `MSDN` 中的描述，很明显这是一个资源的序号。现在看第一个参数，`IDA Pro` 已经认出它是一个全局的实例句柄。显然，这里指的是任务管理器的实例句柄。我们用资源查看工具（例如 `Visual Studio 2005`）来看一下，很容易发现它就是任务管理器的图标 `ID`。当你查看由任务管理器产生的“运行”对话框时，你会发现对话框左边的图标确实是任务管理器的图标。这说明，第二个参数是用来控制这个图标的，并且它是一个图标句柄（即 `HICON` 类型的变量）。再看 `RunFileDlg` 函数后面对 `DestroyIcon` 函数的调用，以 `ebx` 为参数，更证明了我们的推断。根据 `Windows` 命名习惯，可以将此参数命名为 `hIcon`。

识别第三、四、五个参数

再来看对 `LoadString` 函数的第一次调用。此函数的第一个参数同样为任务管理器的实例句柄。第二个参数为一个字符串的 `ID`。同样用资源查看工具，可以发现这个字符串为“Create New Task”，第三个参数 `eax` 用来保存这个字符串的地址。而 `eax` 中实际是一个局部变量的偏移地址（用 `ebp` 寻址这个变量，编译器一般为未使用栈帧指针省略—`FPO` 优化的函数生成这样的代码）。也就是说，这个变量指向了这个字符串。从后面可以看出，这个局部变量作为 `RunFileDlg` 函数的第四个参数被压入堆栈。观察由任务管理器产生的“运行”对话框，你会发现此对话框的标题正是这个字符串。根据 `Windows` 命名习惯，我们可以将这个参数命名为 `lpszCaption`。但是现在还不知道这个字符串可不可以是 `ANSI` 字符串（由 `taskmgr.exe` 中的用法知道它可以是 `UNICODE` 字符串）。

接下来看对 `LoadString` 函数的第二次调用。使用与上面相同的方法分析，可以知道 `RunFileDlg` 函数的第五个参数也是一个指向字符串的指针。在任务管理器它指向“Type the name of a program, folder, document, or Internet resource, and Windows will open it for you.”，这个字符串正是运行对话框中显示的说明文字。根据 `Windows` 命名习惯，可以将其命名为 `lpszText`。与上面一样，我们现在还不知道它指向的字符串可不可以是 `ANSI` 类型的。

接着看对 `GetCurrentDirectory` 函数的调用。同样，它将获得的当前目录字符串放入一个局部变量中，然后将此局部变量的地址作为第三个参数传递给 `RunFileDlg` 函数。显然，这个参数是一个指向目录字符串的指针，但是现在还不知道它必须指向当前目录，还是可以指向其它目录。根据 `Windows` 命名习惯，将其命名为 `lpszDestDirectory`。现在，同样不知道这个参数指向的字符串可不可以是 `ANSI` 类型的。

以上三个参数的不确定性可以通过实验来解决。

识别第六个参数

只剩下最后一个参数了。但是我们无法这样判断了。因为 `taskmgr.exe` 并未对这个参数进行操作，仅将 `14h` 作为这个参数的值压入堆栈。根据 `Windows` 编程经验，这很可能是一个标志位（`Windows API` 中经常把标志作为最后一个参数，而在反汇编代码中，它就是一个常量）。现在只能通过反汇编 `Shell32.dll` 来寻找 `RunFileDlg` 函数是如何使用这一参数的。不幸的是，`RunFileDlg` 函数并没有很明确地使用这个参数。所以我们最好根据实验来确定。根据 `Windows` 命名习惯，将此参数命名为 `dwFlags`。

利用实验消除不确定性因素

下面用实验来具体确定各个参数的类型。

首先，我们来确定其中指向字符串的参数可不可以指向 `ANSI` 字符串。程序如下：

`RunFileDlg2.cpp` 程序

```cpp
#define  WIN32_LEAN_AND_MEAN
#ifndef UNICODE
#define UNICODE
#endif

#include <windows.h>
#include <tchar.h>

TCHAR szShell32[]=TEXT("shell32.dll");
char szCaption[]="测试\\"运行\\"对话框";
TCHAR szText[]=TEXT("这是一个对\\"运行\\"对话框进行实验的程序");
TCHAR szDestDirectory[MAX_PATH];

#pragma comment(linker, "/entry:main")

void __declspec(naked) main()
{
  __asm
  {
    xor eax, eax
    push eax    ;dwFlags

    mov eax, offset szText
    push eax

    mov eax, offset szCaption
    push eax

    mov eax, offset szDestDirectory
    push eax
    push MAX_PATH
    call DWORD PTR [GetCurrentDirectory]
    mov eax, offset szDestDirectory
    push eax    ;lpszCurrentDirectory

    push 32516    ;IDI_INFORMATION 的硬编码
    xor eax, eax
    push eax
    call DWORD PTR [LoadIcon]
    push eax    ;hIcon
```



### 汇编代码示例

```asm
    call DWORD PTR [GetDesktopWindow]  
    push eax    ;hwndParent  
    
    mov eax, offset szShell32  
    push eax  
    call DWORD PTR [LoadLibrary]  
    
    push 3Dh  
    push eax  
    call DWORD PTR [GetProcAddress]  
    
    call eax  
    
    call DWORD PTR [ExitProcess]  
  }  
}
```

编译运行它，结果如图 2 所示。

图 2 `RunFileDlg2.cpp` 运行结果

注意到程序中，我给第四个参数 `lpszCaption` 传递的是一个指向 ANSI 字符串的指针。结果，“运行”对话框的标题变成了乱码。给第五个参数传递的是一个指向 UNICODE 字符串的指针，显示一切正常。而且图标也换成了指定的图标。大家还可以继续做其它实验，都可以证明这三个指向字符串的指针必须指向 UNICODE 字符串。

现在来看一下 `lpDestDirectory` 参数可否指向其它目录。程序如下：

### RunFileDlg3.cpp 程序

```cpp
#define  WIN32_LEAN_AND_MEAN  
#ifndef UNICODE  
#define UNICODE  
#endif  

#include <windows.h>  
#include <tchar.h>  

TCHAR szShell32[]=TEXT("shell32.dll");  
TCHAR szCaption[]=TEXT("测试\"运行\"对话框");  
TCHAR szText[]=TEXT("这是一个对\"运行\"对话框进行实验的程序");  
TCHAR szDestDirectory[]=TEXT("C:\\windows");  

#pragma comment(linker, "/entry:main")  

void __declspec(naked) main()  
{  
  __asm  
  {  
    xor eax, eax  
    push eax    ;dwFlags  
    
    mov eax, offset szText  
    push eax  
    
    mov eax, offset szCaption  
    push eax  
    
    mov eax, offset szDestDirectory  
    push eax  
    
    push 32516      ;IDI_INFORMATION 的硬编码  
    xor eax, eax  
    push eax  
    call DWORD PTR [LoadIcon]  
    push eax    ;hIcon  
    
    call DWORD PTR [GetDesktopWindow]  
    push eax    ;hwndParent  
    
    mov eax, offset szShell32  
    push eax  
    call DWORD PTR [LoadLibrary]  
    
    push 3Dh  
    push eax  
    call DWORD PTR [GetProcAddress]  
    
    call eax  
    
    call DWORD PTR [ExitProcess]  
  }  
}
```

在第二个实验中，如果我们在“运行”对话框中输入“CMD”，单击“确定”按钮，结果如下图所示。

在本次实验中，结果如下图所示：

即命令行解释器的当前目录已经被设置为我们指定的目录。这说明第三个参数可以为其它目录。

下面来确定最后一个参数。

Windows 中的标志参数往往都是每个标志占一个位，最后把它们用位或组合起来。因此我们可以手工为这个参数赋值，以观察效果。

当为此参数赋值 0 时，结果如下图所示。

当为此参数赋值 1 时，结果如下图所示（不显示“浏览”按钮）。

当为此参数赋值 2 时，结果如下图所示（不显示最后一次输入的命令）。

当为此参数赋值 3 时，结果如下图所示（以上两种情况的组合）。

其它情况读者可以自行实验。可见，这个参数可能控制着“运行”对话框的外观。

### 结论

根据 `IDA Pro` 对 `Shell32.dll` 的反汇编结果可以看出，它的返回值为 `int` 类型，但是对于这个函数来说，返回值并不重要。

最后，我们得到 `RunFileDlg` 函数的原型为

```cpp
int RunFileDlg( HWND hwndParent,  
HICON hIcon,  
LPWSTR lpszDestDirectory ,  
LPWSTR lpszCaption,  
LPWSTR lpszText,  
DWORD dwFlags);
```

参数：

`hwndParent`  
“运行”对话框父窗口句柄，如果为 `NULL`，则父窗口为桌面。

`hIcon`  
“运行”对话框中图标句柄，如果为 `NULL`，则由系统提供默认图标。

`lpszDestDirectory`  
设置要运行的程序的当前目录，如果为 `NULL`，则设置为程序所在目录。

`lpszCaption`  
指向“运行”对话框标题文本字符串，如果为 `NULL`，则由系统提供默认标题。

`lpszText`  
指向“运行”对话框内容文本字符串，如果为 `NULL`，则由系统提供默认内容。

`dwFlags`  
在某些方面控制着“运行”对话框外观，如以上各图所示。如果为 `NULL`，则显示系统默认的三个按钮。

它由 `Shell32.dll` 导出，导出序号为 61，也就是 `3Dh`。

### 在 C 语言中使用

现在根据以上原型，用 C 语言编写一个样例程序。如下所示。

#### RunFileDlg4.cpp 文件

```cpp
#define WIN32_LEAN_AND_MEAN  
#ifndef UNICODE  
#define UNICODE  
#endif  

#include <Windows.h>  
#include "resource.h"  

INT_PTR CALLBACK DialogProc( HWND hwndDlg,  
            UINT uMsg,  
          WPARAM wParam,  
          LPARAM lParam  
          );  


typedef int (* pfnRunFileDlg)(HWND, HICON, LPWSTR, LPWSTR, LPWSTR, DWORD);  

#pragma comment(linker, "/entry:main")  

void ShowRunDlg(HWND hwndParent, DWORD dwFlags)  
{  
  pfnRunFileDlg pRunFileDlg=(pfnRunFileDlg)GetProcAddress(LoadLibrary(TEXT("Shell32.dll")), (LPCSTR)61);  
  pRunFileDlg(hwndParent, LoadIcon(NULL, IDI_INFORMATION), TEXT("c:\\windows"), TEXT("测试\"运行\"对话框"), TEXT("这是一个\"运行\"对话框测试程序。"), dwFlags);  
}  

void main(void)  
{  
  DialogBoxParam(GetModuleHandle(NULL), MAKEINTRESOURCE(IDD_DIALOGTEST), NULL, DialogProc, NULL);  

  ExitProcess(0);  
}  

INT_PTR CALLBACK DialogProc(HWND hwndDlg, UINT uMsg, WPARAM wParam, LPARAM lParam)  
{  
  switch(uMsg)  
  {  
    case WM_INITDIALOG:  
      return TRUE;  

    case WM_COMMAND:  
      switch(LOWORD(wParam))  
      {  
        case IDOK:  
          ShowRunDlg(hwndDlg, 0);  
          break;  

        case IDCANCEL:  
          EndDialog(hwndDlg, 0);   
          break;  
      }  
    break;  

    case WM_CLOSE:  
      EndDialog(hwndDlg, 0);  
      break;  
     
    default:  
      return FALSE;  
  }  

  return TRUE;  
}
```

#### RunFileDlg4.rc 文件

```rc
#include "resource.h"  

IDD_DIALOGTEST DIALOGEX 0, 0, 256, 122  
STYLE DS_CENTER | DS_FIXEDSYS | DS_MODALFRAME | WS_POPUP | WS_CAPTION | WS_SYSMENU  
CAPTION "Windows 运行对话框测试程序"  
FONT 8, "MS Shell Dlg", 400, 0, 0x1  
BEGIN  
    DEFPUSHBUTTON   "运行(&R)...",IDOK,47,54,50,14  
    PUSHBUTTON      "退出(&X)",IDCANCEL,158,54,50,14  
END
```

#### resource.h 文件

```cpp
#include <windows.h>  

#define IDD_DIALOGTEST    301
```

在编译上述文件时，记得使用 `stdcall` 调用约定（加上 `/Gz` 编译器选项）。运行结果如下图所示。

单击“运行”按钮，结果如下图所示。

### 说明

读者可能已经看到了，我在所有的示例程序中都使用了

```cpp
#pragma comment(linker, "/entry:main")
```

这条链接器指令。因为 Visual C++ 默认生成的程序要在调用你的入口函数前进行许多初始化操作，包括初始化一些全局变量、堆等，你可以直接在 C 语言中使用的 `argc` 和 `argv` 变量都是由它处理生成的。（详细信息可以参考 Jeffrey Richter 的《Programming Applications for Microsoft Windows, Fourth Edition》一书第四章 Processes 以及 Microsoft Systems Journal—现在已合并到 MSDN Magazine 中 Under the Hood 专栏的部分文章）它将这些代码都静态链接到你的程序中（这也是你用 C 语言写一个 Hello World 程序也会很大的一个主要原因）。而在我们的程序中，我们只调用 Win32 API，并不调用任何 C/C++运行时库函数，也不使用任何它的全局变量，当然不需要它进行任何的初始化操作。这样，便于我们调试自己的发行版程序，而不是在由它生成的指令堆中挣扎了半天，还没有到我们自己写的代码中。

### 思考

在前面找 `RunFileDlg` 函数的导出序号时，我是从 `IDA Pro` 显示的 `taskmgr.exe` 的导入表中的

```text
010011C4 61  __imp__RunFileDlg@24           SHELL32
```

这一行得到线索的。不知读者有没有注意到这样一个事实，那就是：`Shell32.dll` 中的 `RunFileDlg` 函数并没有使用名称导出，在我们使用的导入库文件（`Shell32.lib`）中并没有 `RunFileDlg` 这个函数的相关导入信息（可以使用 `DUMPBIN` 工具来验证）。因此，在我们的程序中就不可能直接调用这个函数，即使你自己声明这个函数的原型。但是，Microsoft 的程序 `taskmgr.exe` 中却有此函数的相关导入信息，也就是这个函数的导入序号 `3Dh`。也就是说，`taskmgr.exe` 中的这个函数是通过导入库引用的。这到底是为什么呢？是不是说明了 Microsoft 使用的导入库与他分发给我们使用的导入库不一样呢？我将这个问题留给读者思考。

作者：SmartTech    电子信箱：zhzhtst@163.com
