
`HINSTANCE` `hInstance`,  // 当前应用程序实例句柄,由系统提供  
`HINSTANCE` `hPrevInstance`,  // 上一个实例句柄 ,CreateMutex 可创建实例互斥体  
`LPSTR` `lpCmdLine`,      // cmd 命令行,可以使用 `GetCommandLine` 获得完整的命令行参数  
`int` `nCmdShow`          // 窗口显示方式,以 `SW_XXXX` 开始,例:`SW_NORMAL` 默认显示方式  
);

`WinMain` 中第三个参数在 `WINBASE.H` 中定义为 `LPSTR`，我将它改为 `PSTR`。这两种数据型态都定义在 `WINNT.H` 中，作为指向字符串的指针。`LP` 前缀代表「长指针」，这是 16 位 Windows 下的产物。在 `WinMain` 声明中改变了两个参数的名称。许 Windows 程序中的变量名使用一种称作「匈牙利表示法」的命名系统，该系统在变量名称前面增加了表示变量数据型态的短前缀，现在仅需记住前缀 `i` 表示 `int`、`sz` 表示 `String of Zero`「以零结束的字符串」。

-----------------------------------------------------------------------------------------------

### -Unicode 的简介

#### 宽字符和 C 语言

C 中的宽字符基于 `wchar_t` 数据型态，它在几个表头文件包括 `WCHAR.H` 中都有定义，像这样：`typedef unsigned short wchar_t ;` 因此，`wchar_t` 数据型态与无符号短整数型态相同，都是 16 位宽。

`strlen` 接受的是 `char` 类型字符,如果接受 Unicode 字符,编译器会把每个字符都分配为两个字节,如果第一个字符为字母,则被分配为这个字母的 ASCII 字符和 0，如果此时再次用 `strlen()` 命令将返回为 1[第二个字符为 0 则结束]，并不会返回定义的 Unicode 字符的长度.`strlen` 函数的宽字符版是 `wcslen`（wide-character string length：宽字符串长度），而 `wcslen` 函数则说明如下：`size_t __cdecl wcslen (const wchar_t *)` ;`size_t` 在 `stddef.h` 中定义:`typedef unsigned int size_t;``TCHAR.H` 还用一个新的数据型态 `TCHAR` 来解决两种字符数据型态的问题.

```
if ANSI:
typedef char CHAR ;
```
```
if UNICODE:
typedef wchar_t WCHAR ;
```

----------------------------------------------------------------------------
```
#ifdef ANSI
typedef CHAR * PCHAR, * LPCH, * PCH, * NPSTR, * LPSTR, * PSTR ;
typedef CONST CHAR * LPCCH, * PCCH, * LPCSTR, * PCSTR ;      ANSI 字符定义,8_bit
#endif
```
前缀 `N` 和 `L` 表示「near」和「long」，指的是 16 位 Windows 中两种大小不同的指标。在 Win32 中 near 和 long 指标没有区别

------------------------------------------------------------------------------------

```
#IFDEF UNICODE
typedef WCHAR * PWCHAR, * LPWCH, * PWCH, * NWPSTR, * LPWSTR, * PWSTR ;
typedef CONST WCHAR * LPCWCH, * PCWCH, * LPCWSTR, * PCWSTR ;   UNCODE 字符定义 16_bit
#ENDIF
```
-----------------------------------------------------------------------------------------------

### -窗口和消息

#### Windows-Hello World 函数

`HICON LoadIcon(HINSTANCE hInstance,LPCTSTR lpIconName)` 加载图标供程序使用。

`HCURSOR LoadCursor(HINSTANCE hInstance,LPCTSTR lpCursorName)` 加载鼠标光标供程序使用。

`HGDIOBJ GetStockObject(int fnObject)` 取得一个图形对象（在这个例子中，是取得绘制窗口背景的画刷对象）。

`ATOM RegisterClass (CONST WNDCLASS *lpWndClass)` 为程序窗口注册窗口类别。

`int MessageBox(HWND,LPCTSTR,LPCTSTR,UINT)` 显示消息框。

```
HWND CreateWindow(   //根据窗口类别建立一个窗口。
  LPCTSTR lpClassName,  // 窗口类名
  LPCTSTR lpWindowName, // 窗口标题
  DWORD dwStyle,        // 窗口风格
  int x,                // X 坐标
  int y,                // Y 坐标
  int nWidth,           // 宽度
  int nHeight,          // 高度
  HWND hWndParent,      // 父窗口句柄
  HMENU hMenu,          // 菜单句柄
  HANDLE hInstance,     // 单签实例句柄
  LPVOID lpParam        // 指向 CREATESTRUCT 结构
);
```

`ShowWindow` 在屏幕上显示窗口,第二个参数是作为参数传给 `WinMain` 的 `iCmdShow`。它确定最初如何在屏幕上显示窗口，是一般大小、最小化还是最大化。

`UpdateWindow` 指示窗口自我更新。

`GetMessage` 从消息队列中取得消息,第二、第三和第四个参数设定为 `NULL` 或者 `0`，表示程序接收它自己建立的所有窗口的所有消息。

`TranslateMessage` 转译某些键盘消息。

`DispatchMessage` 将消息发送给窗口消息处理程序。

`PlaySound` 播放一个声音文件。

`BeginPaint` 开始绘制窗口。

`GetClientRect` 取得窗口显示区域的大小。

`DrawText` 显示字符串。

`EndPaint` 结束绘制窗口。

`PostQuitMessage` 在消息队列中插入一个「退出程序」消息。

`DefWindowProc` 执行内定的消息处理。

窗口消息处理程序总是定义为如下形式：`LRESULT CALLBACK WndProc (HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam);`

`LRESULT` 在 `winuser.h` 定义为:`typedef LRESULT (CALLBACK* WNDPROC)(HWND, UINT, WPARAM, LPARAM);`

这里 `typedef` 的用法是:把 `LRESULT` 作为 `WNDPROC` 的函数指针

窗口消息处理程序是命名为 `WndProc` 的函数。窗口消息处理程序可任意命名（只要求不和其它名字发生冲突）。一个 Windows 程序可以包含多个窗口消息处理程序。一个窗口消息处理程序总是与呼叫 `RegisterClass` 注册的特定窗口类别相关联

一般来说，Windows 程序写作者使用 `switch` 和 `case` 结构来确定窗口消息处理程序接收的是什么消息，以及如何适当地处理它。窗口消息处理程序在处理消息时，必须传回 0。窗口消息处理程序不予处理的所有消息应该被传给名为 `DefWindowProc` 的 Windows 函数。从 `DefWindowProc` 传回的值必须由窗口消息处理程序传回。

对 `WM_PAINT` 的处理几乎总是从一个 `BeginPaint` 呼叫开始,而以一个 `EndPaint` 呼叫结束.

#### Windows 中使用的值常数在表头文件中均有相应的标识符定义。

`CS` 窗口类别样式[ClassStyle]  
`CW` 建立窗口[CreateWindow]  
`DT` 绘制文字[DrawText]  
`IDI` 图示 ID[IDIcon]  
`IDC` 游标 ID[IDCursor]  
`MB` 消息框[MessageBox]  
`SND` 声音[Sound]  
`WM` 窗口消息[WindowMessage]  
`WS` 窗口样式[WindowStyle]

### WNDCLASS[窗口类],表头文件 WINUSER.H 中定义

```
typedef struct _WNDCLASS {
    UINT    style;    //显示风格,以 CS_XXXX 形式
    WNDPROC lpfnWndProc;  //WndProc 函数子程
    int     cbClsExtra;  //初始化为 0
    int     cbWndExtra;  //初始化为 0
    HANDLE  hInstance;   //当前实例句柄
    HICON   hIcon; //图标句柄
    HCURSOR hCursor; //光标句柄
    HBRUSH  hbrBackground; //背景颜色,COLOR_XXXXX 形式
    LPCTSTR lpszMenuName;   //菜单名称,填 NULL,则使用默认名称
    LPCTSTR lpszClassName;  //窗口类名
} WNDCLASS;
```

其中的 `lpfn` 前缀代表「指向函数的指标」。（在 Win32 API 中，长指标和短指标（或者近程指标）没有区别。这只是 16 位 Windows 的遗物。）`cb` 前缀代表「字节数 CountOfByte」而且通常作为一个常数来表示一个字节的大小。`h` 前缀是一个句柄，而 `hbr` 前缀代表「一个画刷的代号」。`lpsz` 前缀代表「指向以 0 结尾字符串的指针」。

### 普通的迭代窗口
各窗口左上角的垂直和水平距离在屏幕上按一定的大小递增

### 消息类型

```
typedef struct tagMSG {     // 在 WinUser.h 中定义
    HWND   hwnd;     //接受消息的窗口句柄
    UINT   message;  //消息类型,一般以 WM_XXXX 形式
    WPARAM wParam;   //消息参数值,据消息而定
    LPARAM lParam;   //消息参数值,据消息而定
    DWORD  time;     //消息放入消息队列中的时间
    POINT  pt;       //消息发布时,光标的坐标
} MSG;
```

### -输出文字

要得到窗口显示区域的设备内容句柄，可以呼叫 `HDC GetDC(HWND hWnd)` 来取得句柄，在使用完后呼叫 `ReleaseDC`

`GetSystemMetrics` 函数以取使用者接口上各类视觉组件大小的信息,取得与系统有关的度量信息

```
BOOL GetTextMetrics(   //取得字体大小
  HDC hdc,            // 指向设备上下文句柄
  LPTEXTMETRIC lptm   // 指向 TEXTMETRIC 结构
);
```

`TEXTMETRIC` 结构定义如下：

```
typedef struct tagTEXTMETRIC { // tm
    LONG tmHeight;
    LONG tmAscent;
    LONG tmDescent;
    LONG tmInternalLeading;
    LONG tmExternalLeading;
    LONG tmAveCharWidth;
    LONG tmMaxCharWidth;
    LONG tmWeight;
    LONG tmOverhang;
    LONG tmDigitizedAspectX;
    LONG tmDigitizedAspectY;
    BCHAR tmFirstChar;
    BCHAR tmLastChar;
    BCHAR tmDefaultChar;
    BCHAR tmBreakChar;
    BYTE tmItalic;
    BYTE tmUnderlined;
    BYTE tmStruckOut;
    BYTE tmPitchAndFamily;
    BYTE tmCharSet;
} TEXTMETRIC;
```

### 滚动条的范围和位置
`BOOL SetScrollRange (hwnd, iBar, iMin, iMax, bRedraw) ;` 参数 `iBar` 为 `SB_VERT` 或者 `SB_HORZ`，`iMin` 和 `iMax` 分别是范围的最小值和最大值。如果想要 Windows 根据新范围重画滚动条，则设置 `bRedraw` 为 `TRUE`（如果在呼叫 `SetScrollRange` 后，呼叫了影响滚动条位置的其它函数，则应该将 `bRedraw` 设定为 `FALSE` 以避免过多的重画）。Windows 提供了类似的函数（`GetScrollRange` 和 `GetScrollPos/ShowScrollBar`）来取得滚动条的目前范围和位置以及显示滚动条.

### -图形基础

`BeginPaint`、`GetDC` 和 `GetWindowDC` 获得的设备内容都与视讯显示器上的某个特定窗口相关。取得设备内容句柄的另一个更通用的函数是 `CreateDC`：`hdc = CreateDC (pszDriver, pszDevice, pszOutput, pData) ;` 可以通过下面的呼叫来取得整个屏幕的设备内容句柄：`hdc = CreateDC (TEXT ("DISPLAY"), NULL, NULL, NULL) ;`

`metafile` 是一些 GDI 呼叫的集合，以二进制形式编码。您可以通过取得 `metafile` 设备内容来建立 `metafile`,`hdcMeta = CreateMetaFile (pszFilename) ;`

### -键盘

#### 按键消息

```
----------------------------------------
|        |  键按下       |     键释放    |
-----------------------------------     |
|非系统键 |  WM_KEYDOWN    |  WM_KEYUP    |
---------------------------------------  |
|系统键   |  WM_SYSKEYDOWN |  WM_SYSKEYUP |
---------------------------------------
```

在四个按键消息（`WM_KEYDOWN`、`WM_KEYUP`、`WM_SYSKEYDOWN` 和 `WM_SYSKEYUP`）中，`wParam` 消息参数含有上面所讨论的虚拟键码，而 `lParam` 消息参数则含有对了解按键非常有用的其它信息。`lParam` 的 32 位分为 6 个字段,0-15 位记录重复计数,16-23 位指定 OEM 扫描码(键盘硬件产生的代码),第 24 为保存扩充键旗标,第 29 位保存内容代码,第 30 位保存此按键的先前状态,第 31 为保存转换状态.

重复计数:重复计数是该消息所表示的按键次数，大多数情况下，重复计数设定为 1。不过，如果按下一个键之后，您的窗口消息处理程序不够快，以致不能处理自动重复速率（您可以在「控制台」的「键盘」中进行设定）下的按键消息，Windows 就把几个 `WM_KEYDOWN` 或者 `WM_SYSKEYDOWN` 消息组合到单个消息中，并相应地增加重复计数。`WM_KEYUP` 或 `WM_SYSKEYUP` 消息的重复计数总是为 1。因为重复计数大于 1 指示按键速率大于您程序的处理能力，所以您也可能想在处理键盘消息时忽略重复计数。几乎每个人都有文书处理或执行电子表格时画面卷过头的经验，因为多余的按键堆满了键盘缓冲区，所以当程序用一些时间来处理每一次按键时，如果忽略您程序中的重复计数，就能够解决此问题。不过，有时可能也会用到重复计数，您应该尝试使用两种方法执行程序，并从中找出一种较好的方法。

OEM 扫描码:OEM 扫描码是由硬件（键盘）产生的代码。这对中古时代的汇编程序写作者来说应该很熟悉，它是从 PC 相容机种的 ROM BIOS 服务中所获得的值（OEM 指的是 PC 的原始设备制造商（Original Equipment Manufacturer）及其与「IBM 标准」同步的内容）。在此我们不需要更多的信息。除非需要依赖实际键盘布局的样貌，不然 Windows 程序可以忽略掉几乎所有的 OEM 扫描码信息
