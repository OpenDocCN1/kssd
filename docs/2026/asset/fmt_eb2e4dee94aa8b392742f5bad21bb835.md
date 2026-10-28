
- 扩展键标记：如果击键结果来自于IBM加强型键盘的附加减则扩展建标记为1，此项通常被忽略
- 内容代码：如果在击键的同时页按下了Alt键，则内容代码为1
- 键的先前状态：如果键以前是处于释放的，则键的先前状态为0。如果键以前是按下的，则键的先前状态为1
- 转换状态：如果键正在被按下，转换状态为0，如果键正在被释放，转换状态为1.

### 6.4.转义状态

当处理击键消息时，可能需要知道是否有转义键（shift、ctrl、alt键）或切换键（caps lock、num lock、和scroll lock键）被按下，可以通过`GetKeyState()`函数获得此信息。

```c
//如果shift键被按下，则iState变量为负（即最高位为1）
iState = GetKeyState(VK_SHIFT);
//如果Caps Lock键打开，则返回的值最低位置为1
iState = GetKeyState(VK_CAPITAL);
```

### 6.5.为SYSMETS加上键盘处理

给第4章`SYSMETS`程序添加键盘消息处理，使用`SengMessage()`函数，可以通过给窗口过程发送假冒的消息欺骗`WndProc()`函数，使它认为收到了滚动条消息。

```c
case WM_KEYDOWN:
switch (wParam)
{
case VK_HOME:
SendMessage (hwnd, WM_VSCROLL, SB_TOP, 0) ;
break ;
case VK_END:
SendMessage (hwnd, WM_VSCROLL, SB_BOTTOM, 0) ;
break ;
case VK_PRIOR:
SendMessage (hwnd, WM_VSCROLL, SB_PAGEUP, 0) ;
break ;
case VK_NEXT:
SendMessage (hwnd, WM_VSCROLL, SB_PAGEDOWN, 0) ;
break ;
case VK_UP:
SendMessage (hwnd, WM_VSCROLL, SB_LINEUP, 0) ;
break ;
case VK_DOWN:
SendMessage (hwnd, WM_VSCROLL, SB_LINEDOWN, 0) ;
break ;
case VK_LEFT:
SendMessage (hwnd, WM_HSCROLL, SB_PAGEUP, 0) ;
break ;
case VK_RIGHT:
SendMessage (hwnd, WM_HSCROLL, SB_PAGEDOWN, 0) ;
break ;
}
return 0 ;
```

### 6.6.字符消息

```c
while (GetMessage(&msg, NULL, 0, 0))
{
TranslateMessage(&msg);
DispatchMessage(&msg);
}
```

这是`WinMain()`函数中的消息循环。`GetMessage()`函数从消息队列总取出下一条消息，天填入`msg`结构的字段。

`DispatchMessage()`函数调用此消息的窗口过程。

`TranslateMessage()`负责把击键消息转换为字符消息，如果击键消息时`WM_KEYDOWN`或`WM_SYSKEYDOWN`，且击键和转移状态组合产生了一个字符，则`TranslateMessage()`函数把字符消息放入应用程序的消息队列。这个字符消息将被放在击键消息之后，`GetMessage()`函数可从消息队列中获取此字符消息。

**四类字符消息**

windows程序会忽略其它三占用字符消息，仅处理`WM_CHAR`消息

|  | 字符 | 死字符 |
| --- | --- | --- |
| 非系统字符 | `WM_CHAR` | `WM_DEADCHAR` |
| 系统字符 | `WM_SYSCHAR` | `WM_SYSDEADCHAR` |

### 6.7.消息排序

例如Caps Lock键没有锁定，则按下再释放A键时，相应的窗口过程会接收一下三个消息

| 消息 | 击键或代码 |
| --- | --- |
| `WM_KEYDOWN` | A的虚拟键代码(`0x41`) |
| `WM_CHAR` | A的字符编码（`0x61`） |
| `WM_KEYUP` | A的虚拟键代码（`0x41`） |

如果输入大写字母A：按下shift键，再按下A键，释放A键，再释放shift键，则窗口过程接收五个消息

| 消息 | 击键或代码 |
| --- | --- |
| `WM_KEYDOWN` | 虚拟键代码`VK_SHIFT`(`0X10`) |
| `WM_KEYDOWN` | A的虚拟键代码（`0x41`） |
| `WM_CHAR` | A的字符编码（`0x41`） |
| `WM_KEYUP` | A的虚拟键代码（`0x41`） |
| `WM_KEYUP` | 虚拟键代码`VK_SHIFT`(`0X10`) |

### 6.8.控制字符的处理

ctrl和字母键的组合会产生ASCII控制字符

| 击键 | 字符码 | 产生方法 | ANSI C 转义码 |
| --- | --- | --- | --- |
| 空格键 | `0x08` | Ctrl-H | `\b` |
| Tab键 | `0x09` | Ctrl-I | `\t` |
| ctrl+回车 | `0x0A` | Ctrl-J | `\n` |
| 回车键 | `0X0D` | Ctrl-M | `\r` |
| Esc键 | `0X1B` | Ctrl-[ |  |

我们按照一下的基本规则来处理击键和字符消息

- 如果需要读取输入到窗口中的键盘字符，就处理`WM_CHAR`消息
- 如果需要读取光标键、功能键、Delete键、Insert键、shift、ctrl和alt键则处理`WM_KEYDOWN`

Tab键、回车键、空格键和esc键看作控制字符，而不是虚拟按键，通常这样处理`WM_CHAR`消息

```c
case WM_CHAR:
switch(wParam)
{
case '\b': //backspace
case '\t': //tab
case '\n': //linefeed
case '\r': //carriage return
}
```

### 6.9.插入符号

- 插入符号：当你向程序中输入文本时，通常会有下划线、竖线或方框指示你输入的下一个字符将出现在屏幕上的位置
- 光标：特指表示鼠标位置的位图图像，即鼠标指针

**关于插入符号的函数**

- `CreateCaret()`：创建和窗口关联的插入符号
- `SetCaretPos()`：设置窗口内的插入符号的位置
- `ShowCaret()`：显示插入符号
- `HideCaret()`：隐藏插入符号
- `DestoryCaret()`：销毁插入符号

仅当窗口具有输入焦点时，窗口中插入符号才有意义。程序中通过处理`WM_SETFOCUS`消息和`WM_KILLFOCUS`消息来决定它是否具有输入焦点。

使用插入符号的主要规则：在窗口过程处理`WM_SETFOCUS`消息时调用`CreateCaret()`函数，处理`WM_KILLFOCUS`消息时调用`DestoryCaret()`函数。

## 七、鼠标

### 7.1.客户区鼠标消息

当鼠标移经窗口客户区时，窗口过程接收`WM_MOUSEMOVE`消息。

| 按钮 | 按下 | 释放 | 第二次按下按钮 |
| --- | --- | --- | --- |
| 左键 | `WM_LBUTTONDOWN` | `WM_LBUTTONUP` | `WM_LBUTTONDBLCLK` |
| 中键 | `WM_MBUTTONDOWN` | `WM_MBUTTONUP` | `WM_MBUTTONDBLCLK` |
| 右键 | `WM_RBUTTONDOWN` | `WM_RBUTTONUP` | `WM_RBUTTONDBLCLK` |

对所有消息来说，参数`IParam`包含了鼠标的位置消息

- `x = LOWORD(IParam);`
- `y = HIWORD(IParam);`

参数`wParam`表示鼠标按钮、shift和ctrl键的状态

- `MK_LBUTTON` 按下左键
- `MK_MBUTTON` 按下中键
- `MK_RBUTTON` 按下右键
- `MK_SHIFT` 按下shift键
- `MK_CONTROL` 按下ctrl键

### 7.2.简单的鼠标处理示例

`connect.c`

```c
#include <windows.h>
#include <windowsx.h>

#define MAXPOINTS 1000

LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
                   PSTR szCmdLine, int iCmdShow)
{
    static TCHAR szAppName[] = TEXT("Connect");
    HWND         hwnd;
    MSG          msg;
    WNDCLASS     wndclass;

    wndclass.style = CS_HREDRAW | CS_VREDRAW;
    wndclass.lpfnWndProc = WndProc;
    wndclass.cbClsExtra = 0;
    wndclass.cbWndExtra = 0;
    wndclass.hInstance = hInstance;
    wndclass.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wndclass.hCursor = LoadCursor(NULL, IDC_ARROW);
    wndclass.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
    wndclass.lpszMenuName = NULL;
    wndclass.lpszClassName = szAppName;

    if (!RegisterClass(&wndclass))
    {
        MessageBox(NULL, TEXT("Program requires Windows NT!"),
                   szAppName, MB_ICONERROR);
        return 0;
    }

    hwnd = CreateWindow(szAppName, TEXT("Connect-the-Points Mouse Demo"),
                        WS_OVERLAPPEDWINDOW,
                        CW_USEDEFAULT, CW_USEDEFAULT,
                        CW_USEDEFAULT, CW_USEDEFAULT,
                        NULL, NULL, hInstance, NULL);

    ShowWindow(hwnd, iCmdShow);
    UpdateWindow(hwnd);

    while (GetMessage(&msg, NULL, 0, 0))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return msg.wParam;
}

LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    static POINT pt[MAXPOINTS];
    static int   iCount;
    HDC          hdc;
    int          i, j;
    PAINTSTRUCT  ps;

    switch (message)
    {
        case WM_LBUTTONDOWN:
            iCount = 0;
            InvalidateRect(hwnd, NULL, TRUE); //清空客户区
            return 0;

        case WM_MOUSEMOVE:
            if (wParam & MK_LBUTTON && iCount < 1000) //判断是否按下了左键
            {
                pt[iCount].x = GET_X_LPARAM(lParam);
                pt[iCount++].y = GET_Y_LPARAM(lParam);

                hdc = GetDC(hwnd);
                //画一个黑点，并保存点的坐标
                SetPixel(hdc, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam), 0);
                ReleaseDC(hwnd, hdc);
            }
            return 0;

        case WM_LBUTTONUP:
            InvalidateRect(hwnd, NULL, FALSE);
            return 0;

        case WM_PAINT:
            hdc = BeginPaint(hwnd, &ps);

            SetCursor(LoadCursor(NULL, IDC_WAIT));
            ShowCursor(TRUE);

            for (i = 0; i < iCount - 1; i++)
                for (j = i + 1; j < iCount; j++)
                {
                    MoveToEx(hdc, pt[i].x, pt[i].y, NULL);
                    LineTo(hdc, pt[j].x, pt[j].y);
                }

            ShowCursor(FALSE);
            SetCursor(LoadCursor(NULL, IDC_ARROW));

            EndPaint(hwnd, &ps);
            return 0;

        case WM_DESTROY:
            PostQuitMessage(0);
            return 0;
    }
    return DefWindowProc(hwnd, message, wParam, lParam);
}
```

### 7.3.鼠标双击

如果想让窗口过程接收鼠标双击消息，那么在调用`RegisterClass()`初始化窗口类结构时，必须在窗口风格字段中包含标识符`CS_DBLCLKS`

- `wndclass.style = CS_HREDRAW | CS_VREDRAW | CS_DBLCLKS;`

若窗口类的风格包含`CS_DBLCLKS`，那么双击后，窗口过程会接收如下消息：

- `WM_LBUTTONDOWN`
- `WM_LBUTTONUP`
- `WM_LBUTTONDBLCLK`
- `WM_LBUTTONUP`

### 7.3.非客户区鼠标消息

如果鼠标位于窗口内部除客户区外的其它区域，windows就会向窗口过程发送一个非客户取区消息。窗口的非客户区消息包括标题栏、菜单和滚动条。

系统一般不需要用户处理非客户区消息，用户只需要将这些消息发送给`DefWindowProc()`.

如果鼠标在窗口的非客户区内移动，窗口过程就会接收`WM_NCMOUSEMOVE`(NC：nonclient)消息

| 按钮 | 按下 | 释放 | 第二次按下按钮 |
| --- | --- | --- | --- |
| 左键 | `WM_NCLBUTTONDOWN` | `WM_NCLBUTTONUP` | `WM_NCLBUTTONDBLCLK` |
| 中键 | `WM_NCMBUTTONDOWN` | `WM_NCMBUTTONUP` | `WM_NCMBUTTONDBLCLK` |
| 右键 | `WM_NCRBUTTONDOWN` | `WM_NCRBUTTONUP` | `WM_NCRBUTTONDBLCLK` |

非客户区鼠标消息参数
