
# Unicode的简介

## 宽字符和C语言

C中的宽字符基于`wchar_t`数据型态，它在几个表头文件包括`WCHAR.H`中都有定义，像这样：

```c
typedef unsigned short wchar_t;
```

因此，`wchar_t`数据型态与无符号短整数型态相同，都是16位宽。

`strlen`接受的是`char`类型字符,如果接受Unicode字符,编译器会把每个字符都分配为两个字节,如果第一个字符为字母,则被分配为这个字母的ASCII字符和0，如果此时再次用`strlen()`命令将返回为1[第二个字符为0则结束]，并不会返回定义的Unicode字符的长度。`strlen`函数的宽字符版是`wcslen`（wide-character string length：宽字符串长度），而`wcslen`函数则说明如下：

```c
size_t __cdecl wcslen (const wchar_t *);
```

`size_t`在`stddef.h`中定义:

```c
typedef unsigned int size_t;
```

`TCHAR.H`还用一个新的数据型态`TCHAR`来解决两种字符数据型态的问题.

```c
if ANSI:
typedef char CHAR;
if UNICODE:
typedef wchar_t WCHAR;
```

### ANSI 定义

```c
#ifdef ANSI
typedef CHAR * PCHAR, * LPCH, * PCH, * NPSTR, * LPSTR, * PSTR;
typedef CONST CHAR * LPCCH, * PCCH, * LPCSTR, * PCSTR;
#endif
```

`ANSI`字符定义,8_bit

前缀`N`和`L`表示「near」和「long」，指的是16位Windows中两种大小不同的指标。在Win32中near和long指标没有区别

### UNICODE 定义

```c
#IFDEF UNICODE
typedef WCHAR * PWCHAR, * LPWCH, * PWCH, * NWPSTR, * LPWSTR, * PWSTR;
typedef CONST WCHAR * LPCWCH, * PCWCH, * LPCWSTR, * PCWSTR;
#ENDIF
```

`UNCODE`字符定义 16_bit

# 窗口和消息

## Windows-Hello World函数

`HICON LoadIcon(HINSTANCE hInstance,LPCTSTR lpIconName)` 加载图标供程序使用。

`HCURSOR LoadCursor(HINSTANCE hInstance,LPCTSTR lpCursorName)` 加载鼠标光标供程序使用。

`HGDIOBJ GetStockObject(int fnObject)` 取得一个图形对象（在这个例子中，是取得绘制窗口背景的画刷对象）。

`ATOM RegisterClass (CONST WNDCLASS *lpWndClass)`为程序窗口注册窗口类别。

`int MessageBox(HWND,LPCTSTR,LPCTSTR,UINT)` 显示消息框。

`HWND CreateWindow(` //根据窗口类别建立一个窗口。
`LPCTSTR lpClassName,` // 窗口类名
`LPCTSTR lpWindowName,` // 窗口标题
`DWORD dwStyle,` // 窗口风格
`int x,` // X坐标
`int y,` // Y坐标
`int nWidth,` // 宽度
`int nHeight,` // 高度
`HWND hWndParent,` // 父窗口句柄
`HMENU hMenu,` // 菜单句柄
`HANDLE hInstance,` // 单签实例句柄
`LPVOID lpParam` // 指向CREATESTRUCT结构
`);

`ShowWindow` 在屏幕上显示窗口,第二个参数是作为参数传给`WinMain`的`iCmdShow`。它确定最初如何在屏幕上显示窗口，是一般大小、最小化还是最大化。

`UpdateWindow` 指示窗口自我更新。

`GetMessage` 从消息队列中取得消息,第二、第三和第四个参数设定为NULL或者0，表示程序接收它自己建立的所有窗口的所有消息。

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

`LRESULT`在`winuser.h`定义为:`typedef LRESULT (CALLBACK* WNDPROC)(HWND, UINT, WPARAM, LPARAM);`

这里`typedef`的用法是:把`LRESULT`作为`WNDPROC`的函数指针

窗口消息处理程序是命名为`WndProc`的函数。窗口消息处理程序可任意命名（只要求不和其它名字发生冲突）。一个Windows程序可以包含多个窗口消息处理程序。一个窗口消息处理程序总是与呼叫`RegisterClass`注册的特定窗口类别相关联

一般来说，Windows程序写作者使用`switch`和`case`结构来确定窗口消息处理程序接收的是什么消息，以及如何适当地处理它。窗口消息处理程序在处理消息时，必须传回0。窗口消息处理程序不予处理的所有消息应该被传给名为`DefWindowProc`的Windows函数。从`DefWindowProc`传回的值必须由窗口消息处理程序传回。

对`WM_PAINT`的处理几乎总是从一个`BeginPaint`呼叫开始,而以一个`EndPaint`呼叫结束.

## Windows中使用的值常数

`Windows`中使用的值常数在表头文件中均有相应的标识符定义。

`CS` 窗口类别样式[ClassStyle]

`CW` 建立窗口[CreateWindow]

`DT` 绘制文字[DrawText]

`IDI` 图示ID[IDIcon]

`IDC` 游标ID[IDCursor]

`MB` 消息框[MessageBox]

`SND` 声音[Sound]

`WM` 窗口消息[WindowMessage]

`WS` 窗口样式[WindowStyle]

### WNDCLASS 窗口类

`WNDCLASS`[窗口类],表头文件`WINUSER.H`中定义

```c
typedef struct _WNDCLASS {
    UINT    style;    //显示风格,以CS_XXXX形式
    WNDPROC lpfnWndProc;  //WndProc函数子程
    int     cbClsExtra;  //初始化为0
    int     cbWndExtra;  //初始化为0
    HANDLE  hInstance;   //当前实例句柄
    HICON   hIcon; //图标句柄
    HCURSOR hCursor; //光标句柄
    HBRUSH  hbrBackground; //背景颜色,COLOR_XXXXX形式
    LPCTSTR lpszMenuName;   //菜单名称,填NULL,则使用默认名称
    LPCTSTR lpszClassName; //窗口类名
} WNDCLASS;
```

其中的`lpfn`前缀代表「指向函数的指标」。 （在Win32 API中，长指标和短指标（或者近程指标）没有区别。这只是16位Windows的遗物。）`cb`前缀代表「字节数CountOfByte」而且通常作为一个常数来表示一个字节的大小。`h`前缀是一个句柄，而`hbr`前缀代表「一个画刷的代号」。`lpsz`前缀代表「指向以0结尾字符串的指针」。

### 普通的迭代窗口

普通的迭代窗口:各窗口左上角的垂直和水平距离在屏幕上按一定的大小递增

### 消息类型

```c
typedef struct tagMSG {     // 在WinUser.h中定义
    HWND   hwnd;     //接受消息的窗口句柄
    UINT   message;  //消息类型,一般以WM_XXXX形式
    WPARAM wParam;   //消息参数值,据消息而定
    LPARAM lParam;   //消息参数值,据消息而定
    DWORD  time;     //消息放入消息队列中的时间
    POINT  pt;       //消息发布时,光标的坐标
} MSG;
```

# 输出文字

要得到窗口显示区域的设备内容句柄，可以呼叫 `HDC GetDC(HWND hWnd)`来取得句柄，在使用完后呼叫`ReleaseDC`

`GetSystemMetrics`函数以取使用者接口上各类视觉组件大小的信息,取得与系统有关的度量信息

```c
BOOL GetTextMetrics(   //取得字体大小
  HDC hdc,            // 指向设备上下文句柄
  LPTEXTMETRIC lptm   // 指向TEXTMETRIC结构
);
```

`TEXTMETRIC`结构定义如下:

```c
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

`BOOL SetScrollRange (hwnd, iBar, iMin, iMax, bRedraw)` ;参数`iBar`为`SB_VERT`或者`SB_HORZ`，`iMin`和`iMax`分别是范围的最小值和最大值。如果想要Windows根据新范围重画滚动条，则设置`bRedraw`为`TRUE`（如果在呼叫`SetScrollRange`后，呼叫了影响滚动条位置的其它函数，则应该将`bRedraw`设定为`FALSE`以避免过多的重画）。Windows提供了类似的函数（`GetScrollRange`和`GetScrollPos`/`ShowScrollBar`）来取得滚动条的目前范围和位置以及显示滚动条.

# 图形基础

`BeginPaint`、`GetDC`和`GetWindowDC`获得的设备内容都与视讯显示器上的某个特定窗口相关。取得设备内容句柄的另一个更通用的函数是`CreateDC`：

```c
hdc = CreateDC (pszDriver, pszDevice, pszOutput, pData);
```

可以通过下面的呼叫来取得整个屏幕的设备内容句柄：

```c
hdc = CreateDC (TEXT ("DISPLAY"), NULL, NULL, NULL);
```

`metafile`是一些`GDI`呼叫的集合，以二进制形式编码。您可以通过取得`metafile`设备内容来建立`metafile`，

```c
hdcMeta = CreateMetaFile (pszFilename);
```

# 键盘

## 按键消息

|        |  键按下       |     键释放    |
|        |  `WM_KEYDOWN`    |  `WM_KEYUP`    |
|系统键   | `WM_SYSKEYDOWN` |  `WM_SYSKEYUP` |

在四个按键消息（`WM_KEYDOWN`、`WM_KEYUP`、`WM_SYSKEYDOWN`和`WM_SYSKEYUP`）中，`wParam`消息参数含有上面所讨论的虚拟键码，而`lParam`消息参数则含有对了解按键非常有用的其它信息。`lParam`的32位分为6个字段,0-15位记录重复计数,16-23位指定`OEM`扫描码(键盘硬件产生的代码),第24为保存扩充键旗标,第29位保存内容代码,第30位保存此按键的先前状态,第31为保存转换状态.

重复计数:重复计数是该消息所表示的按键次数，大多数情况下，重复计数设定为1。不过，如果按下一个键之后，您的窗口消息处理程序不够快，以致不能处理自动重复速率（您可以在「控制台」的「键盘」中进行设定）下的按键消息，Windows就把几个`WM_KEYDOWN`或者`WM_SYSKEYDOWN`消息组合到单个消息中，并相应地增加重复计数。`WM_KEYUP`或`WM_SYSKEYUP`消息的重复计数总是为1。因为重复计数大于1指示按键速率大于您程序的处理能力，所以您也可能想在处理键盘消息时忽略重复计数。几乎每个人都有文书处理或执行电子表格时画面卷过头的经验，因为多余的按键堆满了键盘缓冲区，所以当程序用一些时间来处理每一次按键时，如果忽略您程序中的重复计数，就能够解决此问题。不过，有时可能也会用到重复计数，您应该尝试使用两种方法执行程序，并从中找出一种较好的方法。

`OEM`扫描码:`OEM`扫描码是由硬件（键盘）产生的代码。这对中古时代的汇编程序写作者来说应该很熟悉，它是从`PC`相容机种的`ROM BIOS`服务中所获得的值（`OEM`指的是`PC`的原始设备制造商（Original Equipment Manufacturer）及其与「`IBM`标准」同步的内容）。在此我们不需要更多的信息。除非需要依赖实际键盘布局的样貌，不然Windows程序可以忽略掉几乎所有的`OEM`扫描码信息

扩充键旗标:如果按键结果来自`IBM`增强键盘的附加键之一，那么扩充键旗标为1（`IBM`增强型键盘有101或102个键。功能键在键盘顶端，光标移动键从数字键盘中分离出来，但在数字键盘上还保留有光标移动键的功能）。对键盘右端的`Alt`和`Ctrl`键，以及不是数字键盘那部分的光标移动键（包括`Insert`和`Delete`键）、数字键盘上的斜线（／）和`Enter`键以及`Num Lock`键等，此旗标均被设定为1。Windows程序通常忽略扩充键旗标。

内容代码:右按键时，假如同时压下`ALT`键，那么内容代码为1。对`WM_SYSKEYUP`与`WM_SYSKEYDOWN`而言，此位总视为1；而对`WM_SYSKEYUP`与`WM_KEYDOW`消息而言，此位为0。除了两个之外：如果活动窗口最小化了，则它没有输入焦点。这时候所有的按键都会产生`WM_SYSKEYUP`和`WM_SYSKEYDOWN`消息。如果`Alt`键未被按下，则内容代码字段被设定为0。Windows使用`WM_SYSKEYUP`和`WM_SYSKEYDOWN`消息，从而使最小化了的活动窗口不处理这些按键。对于一些外国语文（非英文）键盘，有些字符是通过`Shift`、`Ctrl`或者`Alt`键与其它键相组合而产生的。这时内容代码为1，但是此消息并非系统按键消息。

键的先前状态:如果在此之前键是释放的，则键的先前状态为0，否则为1。对`WM_KEYUP`或者`WM_SYSKEYUP`消息，它总是设定为1；但是对`WM_KEYDOWN`或者`WM_SYSKEYDOWN`消息，此位可以为0，也可以为1。如果为1，则表示该键是自动重复功能所产生的第二个或者后续消息。

转换状态:如果键正被按下，则转换状态为0；如果键正被释放，则转换状态为1。对`WM_KEYDOWN`或者`WM_SYSKEYDOWN`消息，此字段为0；对`WM_KEYUP`或者`WM_SYSKEYUP`消息，此字段为1。

## 虚拟键码

虚拟键码保存在`WM_KEYDOWN`、`WM_KEYUP`、`WM_SYSKEYDOWN`和`WM_SYSKEYUP`消息的`wParam`参数中,此代码标识按下或释放的键,以`VK_XXX`形式存定义在`WinUser.h`中,其中`VK_RETURN`为`Enter`键,`VK_MENU`为`Alt`键,`VK_PRIOR`为`Page Up`键,`VK_NEXT`为`Page Down`键,`VK_SNAPSHOT`为`Print Screen`键,`VK_CLEAR`为`Num Lock`关闭时的数字键盘5

## 位移状态
