
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
