
# 写了个玩具编译器

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-629.htm`](https://www.pediy.com/kssd/pediy09/pediy09-629.htm)

前几天有个朋友做编译原理课程设计，其中一个题目要求 4 天内编写一个编译器。

当时觉得不可能，但考虑了下是否可能简化结构，做个简单的出来，后来就写了这个东西。

这个程序编译过程已经简化，不需要语法树，甚至不需要汇编器，用了 3000 多行代码。

如果语法更简单点，大概确实可以 4 天内写出来，但无法优化。

编译的方法是，在 `yacc` 分析文法时生成代码，`yacc` 分析表达式时内部有一个分析栈，文法单元入栈顺序和执行顺序是一样的。

比如，`2+3*4`，这个式子 `2` `3` `4` 依次入栈，然后 `3*4` 规约，再和 `2` 规约，这正是这个式子执行时的顺序。

因此可以边分析文法边生成代码，代码使用“双栈”结构，一个用于运算表达式，另一个就是存放局部变量，正常用途。

实际上就在一个栈中就可以实现，因为一个函数局部变量的大小是可以确定的。

所以，`EBP` 作为栈的基址用于索引，而 `ESP` 指向栈顶，用于运算表达式，栈区布局如下：

`operand operand .... var(-8) var(-4) ebp(+0) ret(+4) arg(+8) arg(+0c) arg(+10)`

这样对于一个运算，比如加法，其功能是明确的，就是取栈顶的两个数相加。

所以，可以用固定的代码模版来实现，避免分配寄存器和汇编的麻烦。

比如处理 `a+b*c` 这个表达式，`a` `b` `c` 三个变量依次入栈，栈顶两个数相乘结果入栈，然后栈顶两个数相加结果入栈。

原理就是这样，比一个解释器复杂不了多少，剩下的基本是体力活。

`C` 语法有点麻烦，要处理各种类型变量，如果基本类型就是 `VARIANT`，运算都放 `DLL` 库里，肯定还要简单许多。

我写这个东西，实现了三种基本类型 `char` `int` `float`，支持结构体，多维数组，指针，`API`，`if` `while` `for` `switch`。

函数指针单独处理，有两类 `stdproc` `cproc`，分别是 `STDCALL` `CDECL` 调用方式。

但函数调用不检查类型，一方面是我想不出一个幽雅简洁的方法处理自动类型转化，

另一面如果检查类型，调用 `API` 就要先声明，比较麻烦。

测试代码：

```
#import "user32.dll"
#import "kernel32.dll"

int ss[200000];

int main()
{
    char msg[100];
    int t1, t2, n1, n2, n3, fg;
    t1 = GetTickCount();
    n1 = 12;
    fg = 1;
    ss[0] = 2;
    ss[1] = 3;
    ss[2] = 5;
    ss[3] = 7;
    ss[4] = 11;
    n3 = 4;
    while (n1 <= 2000000)
    {
        for (n2 = 0; n2 <= n3; n2++)
        {
            if (n1 < ss[n2] * ss[n2])
                break;
            if (n1 % ss[n2] == 0)
            {
                fg = 0;
                break;
            }
            else
                fg = 1;
        }
        if (fg == 1)
        {
            n3++;
            ss[n3] = n1;
            fg = 0;
        }
        n1++;
    }
    t2 = GetTickCount();

    ((cproc)wsprintfA)(msg, "time=%d\r\n", t2 - t1);
    MessageBoxA(0, msg, "Dynasty", 0);

    int i;
    for (i = 0; i < 10; i++)
    {
        ((cproc)wsprintfA)(msg, "%d\r\n", ss[i]);
        MessageBoxA(0, msg, "Dynasty", 0);
    }
}
```

用 `kflnig` 的计算素数代码改的，`API` 默认为 `stdcall`，`wsprintfA` 要转化成 `cproc` 调用，计算的结果，比 `VC` 慢了 2.5 倍。

另一个例子，是生成窗口的 `hello world`。

```
#import "user32.dll"
#import "kernel32.dll"
#import "gdi32.dll"

struct POINT
{
    int x;
    int y;
};

struct MSG
{
    int hwnd;
    int message;
    int wParam;
    int lParam;
    int time;
    POINT pt;
};

struct WNDCLASSEXA
{
    int cbSize;
    int style;
    stdproc lpfnWndProc;
    int cbClsExtra;
    int cbWndExtra;
    int hInstance;
    int hIcon;
    int hCursor;
    int hbrBackground;
    char* lpszMenuName;
    char* lpszClassName;
    int hIconSm;
};

struct RECT
{
    int left;
    int top;
    int right;
    int bottom;
};

struct PAINTSTRUCT
{
    int hdc;
    int fErase;
    RECT rcPaint;
    int fRestore;
    int fIncUpdate;
    char rgbReserved[32];
};

int WndProc(int hWnd, int message, int wParam, int lParam);

int MyRegisterClass(int hInstance)
{
    WNDCLASSEXA wcex;
    wcex.cbSize = 48;
    wcex.style = 3; // CS_HREDRAW|CS_VREDRAW;
    wcex.lpfnWndProc = WndProc;
    wcex.cbClsExtra = 0;
    wcex.cbWndExtra = 0;
    wcex.hInstance = hInstance;
    wcex.hIcon = 0;
    wcex.hCursor = LoadCursorA(0, 32512); // IDC_ARROW
    wcex.hbrBackground = 6; // (HBRUSH)(COLOR_WINDOW+1);
    wcex.lpszMenuName = (char*)0;
    wcex.lpszClassName = "Class_Name_Dynasty_HelloWorld";
    wcex.hIconSm = 0;
    return RegisterClassExA(&wcex);
}

int hInst;

int InitInstance(int hInstance, int nCmdShow)
{
    int hWnd;
    hInst = hInstance;

    hWnd = CreateWindowExA(0,
        "Class_Name_Dynasty_HelloWorld",
        "HelloWorld",
        0xCF0000, // WS_OVERLAPPEDWINDOW,
        0x80000000, // CW_USEDEFAULT,
        0,
        0x80000000, // CW_USEDEFAULT,
        0, 0, 0, hInstance, 0);

    if (!hWnd)
        return 0;

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    return 1;
}

int WinMain(int hInstance, int hPrevInstance, char* lpCmdLine, int nCmdShow)
{
    MSG msg;
    MyRegisterClass(hInstance);
    if (!InitInstance(hInstance, nCmdShow))
        return 0;
    while (GetMessageA(&msg, 0, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessageA(&msg);
    }
    return msg.wParam;
}

int main()
{
    return ExitProcess(WinMain(GetModuleHandleA(0), 0, GetCommandLineA(), 1)); // SW_SHOWNORMAL
}

int LOWORD(int dword)
{
    int h1, h2, h3, h4;
    h4 = *((char*)&dword + 0);
    h3 = *((char*)&dword + 1);
    h2 = *((char*)&dword + 2);
    h1 = *((char*)&dword + 3);
    return h3 * 0xFF + h4;
}

int HIWORD(int dword)
{
    int h1, h2, h3, h4;
    h4 = *((char*)&dword + 0);
    h3 = *((char*)&dword + 1);
    h2 = *((char*)&dword + 2);
    h1 = *((char*)&dword + 3);
    return h1 * 0xFF + h2;
}

int WndProc(int hWnd, int message, int wParam, int lParam)
{
    int wmId, wmEvent;
    PAINTSTRUCT ps;
    int hdc;

    switch (message)
    {
    case 0x0111: // WM_COMMAND
        wmId = LOWORD(wParam);
        wmEvent = HIWORD(wParam);
        switch (wmId)
        {
        case 105: // IDM_EXIT:
            DestroyWindow(hWnd);
            break;
        default:
            return DefWindowProcA(hWnd, message, wParam, lParam);
        }
        break;
    case 0x000F: // WM_PAINT
        hdc = BeginPaint(hWnd, &ps);
        RECT rt;
        GetClientRect(hWnd, &rt);
        rt.top = rt.top + 30;
        DrawTextA(hdc, "Hello World", 11, &rt, 1); // DT_CENTER
        rt.top = rt.top + 30;
        DrawTextA(hdc, "build by DynastyCompiler", 24, &rt, 1); // DT_CENTER
        EndPaint(hWnd, &ps);
        break;
    case 0x0002: // WM_DESTROY
        PostQuitMessage(0);
        break;
    default:
        return DefWindowProcA(hWnd, message, wParam, lParam);
    }
    return 0;
}
```

这种结构写出来的东西，用于编译的话，代码太烂，估计没多少价值。

不过，这个可以用来实现高速脚本，不生成 `PE`，直接编译在内存中运行。

虽然比 `VC` 慢，但毕竟是编译级别的速度，解释执行据说最好的优化也要慢 10 倍。

另外，也许可以用来做壳，改进下，生成 `LIB`，就可以和 `VC` 联合编译。

在编译器级别“编织”代码，可以编出的花纹就肯定漂亮得多，只是这样估计要生成语法树。

原代码可以直接编译，但如果要修改 `yacc.y` 需要 `bison2.1`，其他版本也行，但必须是支持 `%glr-parser` 之后的版本。

下载 `bison` 后要在 `VC6 Project Settings` 里设置 `yacc.y` 的 `custom build` 中调用 `bison` 的路径才能编译。

`bison2.1` 下载地址和其他一些参考资料：

`http://gnuwin32.sourceforge.net/packages/bison.htm`

`bison2.1` 官方下载

`http://docs.huihoo.com/vm/tut_script/tut_script0.htm`

实现一个脚本引擎

`http://wiki.hit.edu.cn/index.php/Lex%E5%92%8Cyacc%E5%B7%A5%E5%85%B7%E4%BB%8B%E7%BB%8D`

`Lex` 和 `yacc` 工具介绍

`http://blog.csdn.net/sirouni2003/archive/2005/06/22/400672.aspx`

`20050620 GNU Bison` 中文手册翻译完成

`http://blog.csdn.net/sirouni2003/archive/2006/02/01/590661.aspx`

`20060121 GNU Bison 2.1` 中文手册翻译完成

`https://www6.software.ibm.com/developerworks/cn/education/aix/au-lexyacc/index.html`

`IBM developerWorks`:使用 `yacc` 和 `lex` 编写文本分析器

`http://www.ibm.com/developerworks/cn/linux/sdk/lex/index.html`

`IBM developerWorks`: `Yacc` 与 `Lex` 快速入门

`http://www.51delphi.com/wz/8.html`

用 `YACC/LEX` 设计计算机语言:`BASIC`

`http://tag.csdn.net/author/a0786de3-9b05-437a-848a-1b0611916677/pandaxcl/lex 和 yacc/1.html`

`Lex` 和 `Yacc` 从入门到精通(1)－－环境配置篇

`http://blog.csdn.net/dawndu/archive/2006/04/13/662260.aspx`

递归下降纯解释器编写的困惑 (关于纯解释中的分支，循环)

`http://www.cublog.cn/u/13392/showart_203707.html#13.%20Caveats%20and%20Bugs`

`Lex`-词法分析器生成器(`M.E.Lesk`,`E.Schmidt` 著,中文翻译

`http://www.ibm.com/developerworks/cn/linux/game/sdl/pirates-4/index.html`

`SDL` 用法，第 4 部分: `lex` 和 `yacc`

构建用于脚本和 `GUI` 设计的语法分析器
