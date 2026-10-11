
# 在 Visual C++中使用内联汇编

> 原文：[`www.pediy.com/kssd/pediy05/pediy50279.htm`](https://www.pediy.com/kssd/pediy05/pediy50279.htm)

## 一、内联汇编的优缺点

因为在 Visual C++中使用内联汇编不需要额外的编译器和联接器，且可以处理 Visual C++中不能处理的一些事情，而且可以使用在 C/C++中的变量，所以非常方便。内联汇编主要用于如下场合:

1. 使用汇编语言写函数；
2. 对速度要求非常高的代码；
3. 设备驱动程序中直接访问硬件；
4. "Naked" Call 的初始化和结束代码。

//(".Naked"，理解了意思，但是不知道怎么翻译^\_^，大概就是不需要 C/C++的编译器(自作聪明)生成的函数初始化和收尾代码，请参看 MSDN 的"Naked Functions"的说明)

内联汇编代码不易于移植，如果你的程序打算在不同类型的机器（比如 `x86` 和 `Alpha`）上运行，应当尽量避免使用内联汇编。这时候你可以使用 `MASM`，因为 `MASM` 支持更方便的的宏指令和数据指示符。

## 二、内联汇编关键字

在 Visual C++使用内联汇编用到的是`__asm` 关键字，这个关键字有两种使用方法:

1. 简单`__asm` 块

```
__asm
{
    MOV     AL, 2
    MOV     DX, 0XD007
    OUT     AL, DX
}
```

2. 在每条汇编指令之前加`__asm` 关键字

```
__asm MOV    AL, 2
__asm MOV    DX, 0xD007
__asm OUT    AL, DX
```

因为`__asm` 关键字是语句分隔符，因此你可以把汇编指令放在同一行:

```
__asm MOV AL, 2     __asm MOV DX, 0XD007    __asm OUT AL, DX
```

显然，第一种方法和 C/C++的风格很一致，并且有很多其它优点，因此推荐使用第一种方法。

不象在 C/C++中的"{}"，`__asm` 块的"{}"不会影响 C/C++变量的作用范围。同时，`__asm` 块可以嵌套，嵌套也不会影响变量的作用范围。

## 三、在`__asm` 块中使用汇编语言

1. 内联汇编指令集

   内联汇编完全支持的 `Intel 486` 指令集，允许使用 `MMX` 指令。不支持的指令可以使用`_EMIT` 伪指令定义(`_EMIT` 伪指令说明见下文)。

2. MASM 表达式

   内联汇编可以使用 `MASM` 中的表达式。比如: `MOV EAX, 1`。

3. 数据指示符和操作符

   虽然`__asm` 块中允许使用 C/C++的数据类型和对象，但它不能用 MASM 指示符和操作符定义数据对象。这里特别指出，`__asm` 块中不允许 MASM 中的定义指示符: `DB`、`DW`、`DD`、`DQ`、`DT` 和 `DF`，也不允许 `DUP` 和 `THIS` 操作符。MASM 结构和记录也不再有效，内联汇编不接受 `STRUC`、`RECORD`、`WIDTH` 或者 `MASK`。

4. EVEN 和 ALIGN 指示符

   尽管内联汇编不支持大多数 MASM 指示符，但它支持 `EVEN` 和 `ALIGN`，当需要的时候，这些指示符在汇编代码里面加入 `NOP`(空操作)指令使标号对齐到特定边界。这样可以使某些处理器取指令时具有更高的效率。

5. MASM 宏指示符

   内联汇编不是宏汇编，不能使用 MASM 宏指示符(`MACRO`、`REPT`、`IRC`、`IRP` 和 `ENDM`)和宏操作符(`<>`、`!`、`&`、`%和.TYPE`)。

6. 段说明

   必须使用寄存器来说明段，跨越段必须显式地说明，如 `ES:[BX]`。

7. 类型和变量大小

   我们可以使用 `LENGTH` 来取得 C/C++中的数组中的元素个数，如果不是一个数组，则结果为一。使用 `SIZE` 来取得 C/C++中变量的大小，一个变量的大小是 `LENGTH` 和 `TYPE` 的乘积。`TYPE` 用来取得一个变量的大小，如果是一个数组，它得到的一个数组中的单个元素的大小。

8. 注释

   可以使用 C/C++的注释，但推荐用 ASM 的注释，即`;`号。

9. `_EMIT` 伪指令

   `_EMIT` 伪指令相当于 MASM 中的 `DB`，但一次只能定义一个字节，比如:

```
__asm
{
    JMP     _CodeOfAsm
    _EMIT   0x00    ; 定义混合在代码段的数据
    _EMIT   0x01
_CodeOfAsm:
    ; 这里是代码
    _EMIT   0x90    ; NOP 指令
}
```

## 四、在`__asm` 块中使用 C/C++语言元素

C/C++与汇编可以混合使用，在内联汇编可以使用 C/C++的变量和很多其它 C/C++的元素。在`__asm` 块中可以使用以下 C/C++元素:

1. 符号，包括标号、变量和函数名；
2. 常量，包括符号常量和枚举型(`enum`)成员；
3. 宏定义和预处理指示符；
4. 注释，包括`/`/`/`和`//`；
5. 类型名，包括所有 MASM 中合法的类型
6. `typedef` 名称， 像 `PTR`、`TYPE`、特定的结构成员或枚举成员这样的通用操作符。

在`__asm` 块中，可以使用 C/C++或 ASM 的基数计数法(比如: `0x100` 和 `100H` 是相等的)。

`__asm` 块中不能使用像`<<`一类的 C/C++操作符。C/C++和 MASM 通用的操作符，比如`*`和`[]`操作符，都被认为是汇编语言的操作符。举个例子:

```
int array[10];
__asm MOV array[6], BX ;  Store BX at array+6 (not scaled)
array[6] = 0;         /* Store 0 at array+12 (scaled) */
```

小技巧: 内联汇编中，你可以使用 `TYPE` 操作符使作其与 C 一致。比如，下面两条语句是一样的:

```
__asm MOV array[6 * TYPE int], 0 ; Store 0 at array + 12
array[6] = 0;                    /* Store 0 at array + 12 */
```

内联汇编能通过变两名直接引用 C/C++的变量。`__asm` 块中可以引用任何符号，包括变量名。

如果 C/C++中的类、结构或者枚举成员具有唯一的名称，如果在`.`操作符之前不指定变量或者 `typedef` 名称，则`__asm` 块中只能引用成员名称。然而，如果成员不是唯一的，你必须在`.`操作符之前加上变量名或 `typedef` 名称。例如，下面的两个结构都具有 `same_name` 这个成员变量:

```
struct first_type
{
    char *weasel;
    int same_name;
};

struct second_type
{
    int wonton;
    long same_name;
};
```

如果按下面声明变量:

```
struct first_type hal;
struct second_type oat;
```

那么，所有引用 `same_name` 成员的地方都必须使用变量名，因为 `same_name` 不是唯一的。另外，上面的 `weasel` 变量具有唯一的名称，你可以仅仅使用它的成员名称来引用它:

```
__asm
{
    MOV EBX, OFFSET hal
    MOV ECX, [EBX]hal.same_name ; 必须使用 'hal'
    MOV ESI, [EBX].weasel       ; 可以省略 'hal'
}
```

注意，省略了变量名仅仅是为了写代码的方便，生成的汇编指令的还是一样的。

可以不受限制地访问 C++成员变量，但是不能调用 C++的成员函数。

## 五、寄存器使用

一般来说，在`__asm` 块开始的时候，寄存器是空的，不能在两个`__asm` 之间保存寄存器的值。(这是 MSDN 上说的，我在实际使用时发现，好像并不是这样。不过它是说"一般"，我是特殊:))

如果一个函数被声明成了`__fastcall`，则其参数将放在寄存器中，这将给寄存器的管理带来问题。所以，如果要将一个函数声明成`__fastcall`，必须保存 `ECX` 寄存器。为了避免以上的冲突，在声明为`__fastcall` 的函数中不要有`__asm` 块。如果用了`/Gr` 编译选项(它全局的变成`__fastcall`)，将每个函数声明成`__cdecl` 或者`__stdcall`，这个属性告诉编译器用传统的 C 方法。

如果使用 `EAX`、`EBX`、`ECX`、`EDX`、`ESI` 和 `EDI` 寄存器，你不需要保存它；但如果你用到了 `DS`、`SS`、`SP`、`BP` 和标志寄存器，那就应该 `PUSH` 保存这些寄存器。

如果程序中改变了用于 `STD` 和 `CLD` 的方向标志，你必须将其恢复到原来的值。

## 六、转跳

可以在 C 里面使用 `goto` 调到`__asm` 块中的标号处，也可以在`__asm` 块中转跳到`__asm` 块里面和外面的标号处。`__asm` 块内的标号是不区分大小写的(指令、指示符等也是不区分大小写的)。例:

```
void func()
{
        goto C_Dest;   /* 合法 */
        goto c_dest;   /* 错误 */
        goto A_Dest;   /* 合法 */
        goto a_dest;   /* 合法 */
        __asm
        {
                JMP C_Dest ; 合法
                JMP c_dest ; MSDN 上说合法，但是我在 VS.NET 中编译，认为这样不合法
                JMP A_Dest ; 合法
                JMP a_dest ; 合法
a_dest:     ; __asm 标号
        }
C_Dest:     /* C 的标号 */
    return;
}
```

不要使用函数名称当作标号，否则将使其跳到函数执行而不是标号处。如下所示:

```
; 错误: 使用函数名作为标号
JNE exit
.
.
.
exit:
; 下面是更多的 ASM 代码
```

美元符号`$`用于指定当前位置，如下所用，常用于条件跳转:

```
JNE $+5 ; 下面这条指令的长度是 5 个字节
JMP farlabel
;$+5，跳到了这里
.
.
.
farlabel:
```

## 七、调用函数

内联汇编调用 C/C++函数必须自己清除堆栈，下面是一个调用 C/C++函数例子:

```
#include <stdio.h>
char szformat[] = "%s %s\n";
char szHello[] = "Hello";
char szWorld[] = " world";

void main()
{
    __asm
    {
        MOV     EAX, OFFSET szWorld
        PUSH    EAX
        MOV     EAX, OFFSET szHello
        PUSH    EAX
        MOV     EAX, OFFSET szformat
        PUSH    EAX
        CALL    printf
        //内联汇编调用 C 函数必须自己清除堆栈
        //用不使用的 EBX 寄存器清除堆栈，或 ADD ESP, 12
        POP     EBX
        POP     EBX
        POP     EBX
    }
}
```

注意:函数参数是从右向左压栈。

不能够访问 C++中的类成员函数，但是可以访问 `extern "C"` 函数。

如果调用 Windows API 函数，则不需要自己清除堆栈，因为 API 的返回指令是 `RET n`，会自动清除堆栈

比如下面的例子:

```
#include <windows.h>
char szAppName[] = "API Test";

void main()
{
    char szHello[] = "Hello, world!";
    __asm
    {
        PUSH    MB_OK OR MB_ICONINformATION
        PUSH    OFFSET szAppName   ; 全局变量用 OFFSET
        LEA     EAX, szHello      ; 局部变量用 LEA
        PUSH    EAX
        PUSH    0
        CALL    DWORD PTR [MessageBoxA]     ; 注意这里，我费了好大周折才发现不是 CALL MessageBoxA
    }
}
```

一般来说，在 Visual C++中使用内联汇编是为了提高速度，因此这些函数调用尽可能用 C/C++写。

## 八、一个例子

下面的例子是在 `VS.NET`(即 `VC7`)中 C 语言写的。先建一个工程，将下列代码放到工程中的`.c` 文件中编译，无需作特别的设置，即可编译通过。

////////////////////////////////////////////////////////////////////////////////////////////////////

//预处理

```
#include <Windows.h>
```

////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////

//全局变量

```
HWND g_hWnd;
HINSTANCE g_hInst;
TCHAR szTemp[1024];
TCHAR szAppName[] = "CRC32 Sample";
```



## 函数声明

```
DWORD GetCRC32(const BYTE *pbData, int nSize);
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int iCmdShow);
LRESULT CALLBACK WindowProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam);
```

## 主函数

```
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int iCmdShow)
{
    MSG msg;
    WNDCLASSEX wndClassEx;
    g_hInst = hInstance;
    wndClassEx.cbSize = sizeof(WNDCLASSEX);
    wndClassEx.style = CS_VREDRAW | CS_HREDRAW;
    wndClassEx.lpfnWndProc = (WNDPROC) WindowProc;
    wndClassEx.cbClsExtra = 0;
    wndClassEx.cbWndExtra = 0;
    wndClassEx.hInstance = g_hInst;
    wndClassEx.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wndClassEx.hCursor = LoadCursor(NULL, IDC_ARROW);
    wndClassEx.hbrBackground = (HBRUSH) (COLOR_WINDOW);
    wndClassEx.lpszMenuName = NULL;
    wndClassEx.lpszClassName = szAppName;
    wndClassEx.hIconSm = NULL;
    RegisterClassEx(&wndClassEx);
    g_hWnd = CreateWindowEx(0, szAppName, szAppName, WS_OVERLAPPED | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME | WS_MINIMIZEBOX,
        CW_USEDEFAULT, CW_USEDEFAULT, 300, 70,
        NULL, NULL, g_hInst, NULL);
    ShowWindow(g_hWnd, iCmdShow);
    UpdateWindow(g_hWnd);
    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return ((int) msg.wParam);
}
```

## 主窗口回调函数

```
LRESULT CALLBACK WindowProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
    case WM_CREATE:
        CreateWindowEx(WS_EX_CLIENTEDGE, "EDIT", NULL, WS_CHILD | WS_VISIBLE | WS_BORDER | ES_AUTOHSCROLL | ES_AUTOVSCROLL | ES_NOHIDESEL | WS_OVERLAPPED,
            7, 12, 220, 22,
            hWnd, (HMENU)1000, g_hInst, NULL);
        CreateWindowEx(0, "BUTTON", "&OK", WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON | WS_OVERLAPPED | BS_FLAT,
            244, 12, 40, 20,
            hWnd, (HMENU)IDOK, g_hInst, NULL);
        break;
    case WM_COMMAND:
        switch (LOWORD(wParam))
        {
        case IDOK:
            GetDlgItemText(g_hWnd, 1000, szTemp + 100, 800);
            wsprintf(szTemp, "当前文本框内的字符串的 CRC32 校验码是: 0x%lX", GetCRC32(szTemp + 100, (int)strlen(szTemp + 100)));
            MessageBox(g_hWnd, szTemp, szAppName, MB_OK|MB_ICONINformATION);
        }
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    default:
        return (DefWindowProc(hWnd, uMsg, wParam, lParam));
    }
    return (0);
}
```

## GetCRC32: 求字节流的 CRC32 校验码

- **参数**:
    - `pbData`: 指向字节流缓冲区首地址
    - `nSize`: 字节流长度
- **返回值**:
    - 字节流的 `CRC32` 校验码

这里使用查表法求 `CRC32` 校验码，这部分是参考老罗的文章《矛与盾的较量（2）——CRC 原理篇》该写的。原文的具体内容请参看: [asp.7i24.com/netcool/laoluo/articles/show_article.asp?Article_ID=15](http://asp.7i24.com/netcool/laoluo/articles/show_article.asp?Article_ID=15)

下面使用内联汇编求 `CRC32` 校验码，充分使用了 `CPU` 中的寄存器，速度和方便性都是使用 C/C++所不能比拟的

```
DWORD GetCRC32(const BYTE *pbData, int nSize)
{
    DWORD dwCRC32Table[256];
    __asm   //这片内联汇编是初始化 CRC32 表
    {
        MOV     ECX, 256
    _NextTable:
        LEA     EAX, [ECX-1]
        PUSH    ECX
        MOV     ECX, 8
    _NextBit:
        SHR     EAX, 1
        JNC     _NotCarry
        XOR     EAX, 0xEDB88320
    _NotCarry:
        DEC     ECX
        JNZ     _NextBit
        POP     ECX
        MOV     [dwCRC32Table + ECX*4 - 4], EAX
        DEC     ECX
        JNZ     _NextTable
    }
    __asm   //下面是求 CRC32 校验码
    {
        MOV     EAX, -1
        MOV     EBX, pbData
        OR      EBX, EBX
        JZ      _Done
        MOV     ECX, nSize
        OR      ECX, ECX
        JZ      _Done
    _NextByte:
        MOV     DL, [EBX]
        XOR     DL, AL
        MOVZX   EDX, DL
        SHR     EAX, 8
        XOR     EAX, [dwCRC32Table + EDX*4]
        INC     EBX
        LOOP    _NextByte
    _Done:
        NOT     EAX
    }
}
```

## 九、后话

本来打算写个综合各种内联汇编用法的例子，由于本人拖拖拉拉的习惯，虽经 `dREAMtHEATER` 多次催促，到最后还是不免草草了事。如果有什么谬误或不明之处，欢迎批评指正并一起探讨(QQ: 123018)。

Yonsm@163.com
2002.9.3

## 参考资料

1. Sunwen，《VC 内联 ASM 汇编学习笔记》
2. 老罗，《矛与盾的较量（2）——CRC 原理篇》
3. Microsoft，《MSDN Library - January 2001》

～· 全文完 ·～
