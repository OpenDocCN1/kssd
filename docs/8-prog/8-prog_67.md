
# Win32 asm 第二十五课：简单的位图

> 原文：[`www.pediy.com/kssd/pediy10/60534.html`](https://www.pediy.com/kssd/pediy10/60534.html)

## Tutorial 25: Simple Bitmap
第二十五课：简单的位图

---

在这一个教程中，我们将学习如何在程序中使用了位图。准确地说，我们将学习如何在我们窗口客户区显示一个位图。

在这一课中，我们将学习如何在我们的程序中使用位图。为了更精确，我们将学习如何在我们窗口客户区显示一个位图。

下载示例代码。

## Theory
## 原理

位图可以被认为是存储在计算机中的图片。计算机使用许多图片格式，但 Windows 仅原生支持 Windows Bitmap Graphics 文件（`.bmp`）。我在本教程中提到的位图都是 Windows 位图图形文件。使用位图最简单的方法是将其作为资源使用。有两种方法可以做到这一点。你可以按照以下方式在资源定义文件（`.rc`）中包含位图：

位图就是存储在计算机中的图像。计算机中的图像可以使用很多的格式，但是 `windows` 仅支持 `windows` `Bitmap` `Graphics` 文件（`.bmp`--位图文件）。我将在本课中涉及的就是位图文件。使用一个位图最简单的方式就是将它作为一种资源文件。有两种方式可以这样做。你能用下面的定义来在资源文件中包含一个位图:

```
#define IDB_MYBITMAP   100

IDB_MYBITMAP  BITMAP  "c:\project\example.bmp"
```

这种方法使用一个常量来表示位图。第一行仅仅是创建一个名为 `IDB_MYBITMAP` 的常量，其值为 100。我们将在程序中使用这个标签来引用位图。下一行声明一个位图资源。它告诉资源编译器在哪里可以找到实际的 `bmp` 文件。

这种方法使用一个常数来表示位图（宏定义）。第一行仅仅是创建一名为 `IDB_MYBITMAP`，值为 100 的常量。我们将在程序中用这个标号来引用这个位图。下一行声明一位图资源。它告诉资源编译器在那里能找到真实的位图文件。

另一种方法使用名称来表示位图，如下所示：

另外一种方法是使用一名称（字符串）来表示位图，向下面这样：

```
MyBitMap  BITMAP "c:\project\example.bmp"
```

这种方法要求在程序中通过字符串 "MyBitMap" 来引用位图，而不是数值。

这种方法要求在程序中引用位图时，是通过一字符串 "MyBitMap" 引用而不在是一数值。

只要你清楚你正在使用哪种方法，两种方法都运行得很好。

既然我们将位图放入资源文件，我们就可以继续进行在窗口客户区显示它的步骤。

两种方法都工作的很好，只要你知道你正在使用的是哪一种方法。现在我们在资源文件中放置一位图， 下一步就是让它显示在我们窗口的客户区上。

1.  调用 `LoadBitmap` 来获取位图句柄。`LoadBitmap` 的定义如下：

    调用 `LoadBitmap` 来获得一位图句柄。`LoadBitmap` 定义如下：

    ```
    LoadBitmap proto hInstance:HINSTANCE, lpBitmapName:LPSTR
    ```

    这个函数返回一个位图句柄。`hInstance` 是我们程序的实例句柄。`lpBitmapName` 是指向位图名称字符串的指针（如果你使用第二种方法来引用位图）。如果你使用常量来引用位图（如 `IDB_MYBITMAP`），你可以将它的值放在这里。（在上面的例子中，它的值就是 100）。一个简短的例子如下：

    这个函数返回一位图句柄。 `hInstance` 是我们程序的实例句柄。`LpBitmapName` 是指向位图名称的指针（包括用第二种方法引用位图）。如果你想使用一个常数来引用位图，你能把它的值放在这儿。（在上面的例子中，它的值就是 100）。一个简短的例子如下：

    **第一种方法：**
    **First Method:**

    ```
    .386

    .model flat, stdcall
    ...............
    .const

    IDB_MYBITMAP    equ 100
    ...............
    .data?

    hInstance  dd ?
    ............
    .code
    .............
        invoke GetModuleHandle,NULL
        mov hInstance,eax
    ............
        invoke LoadBitmap,hInstance,IDB_MYBITMAP
    ...........
    ```

    **第二种方法：**
    **Second Method:**

    ```
    .386

    .model flat, stdcall
    ...............
    .data

    BitmapName  db "MyBitMap",0
    ...............
    .data?

    hInstance  dd ?
    ............
    .code
    .............
        invoke GetModuleHandle,NULL
        mov hInstance,eax
    ............
        invoke LoadBitmap,hInstance,addr BitmapName
    ...........
    ```

2.  获取设备上下文（DC）的句柄。你可以通过响应 `WM_PAINT` 消息调用 `BeginPaint` 或在任何地方调用 `GetDC` 来获取此句柄。

    获得一设备描述表的句柄 （DC）。你能在响应 `WM_PAINT` 消息时调用 `BeginPaint` 函数获得这个句柄。也可以在任何地方调用 `GetDC` 获取 。

3.  创建一个与刚刚获取的设备上下文属性相同的内存设备上下文。这里的想法是创建一种“隐藏”的绘图表面，我们可以将位图画在上面。当我们完成操作后，我们只需在函数调用中将该隐藏绘图表面的内容复制到实际的设备上下文。这是双缓冲技术的一个例子，用于在屏幕上快速显示图片。你可以通过调用 `CreateCompatibleDC` 来创建这个“隐藏”的绘图表面。

    创建一内存设备环境，作为设备描述表的它们都有相同的属性，我们仅仅是获得它。这儿的一种观念是创建一种隐藏画纸，我们能将位图画在上面。当我们完成这个操作后，我们仅仅是在一个函数调用中，将隐藏的画纸复制到实际的设备现场（设备描述表）。这就是用于在屏幕上快速显示图像的双缓冲技术的一种典型。你能通过调用 `CreateCompatibleDC` 函数来创建这张隐藏的画纸。

    ```
    CreateCompatibleDC  proto  hdc:HDC
    ```

    如果此函数成功，它在 `eax` 中返回内存设备上下文的句柄。`hdc` 是你希望内存 DC 与其兼容的设备上下文的句柄。

    如果这个函数成功，它在 `eax` 中返回内存设备描述表的句柄。`Hdc` 是设备描述表句柄，它和你想要的内存 DC 是一致的。

4.  既然你获得了隐藏绘图表面，你可以通过选择位图到其中来在它上面绘制。这是通过调用 `SelectObject` 来完成的，将内存 DC 的句柄作为第一个参数，位图句柄作为第二个参数。`SelectObject` 的定义如下：

    现在，你得到了那张隐藏的画纸，你能将你选择的位图画在它上面。这可以通过调用 `SelectObject` 函数来完成。其中，传入内存 DC 的句柄作为 `SelectObject` 函数的第一个参数，并且用位图句柄做为函数的第二个参数。`SelectObject` 函数定义如下：

    ```
    SelectObject   proto  hdc:HDC, hGdiObject:DWORD
    ```

5.  现在位图已经被画在内存设备上下文上了。我们在这里需要做的只是将其复制到实际的显示设备，即真正的设备上下文。有几个函数可以执行此操作，如 `BitBlt` 和 `StretchBlt`。`BitBlt` 仅仅将一个 DC 的内容复制到另一个 DC，所以它很快，而 `StretchBlt` 可以拉伸或压缩位图以适应输出区域。为了简单起见，我们将在这里使用 `BitBlt`。`BitBlt` 的定义如下：

    现在，位图已经画在内存设备描述表上了。在这儿我们需要做的是将位图复制到真正的显示设备中，也就是真正的设备描述表。这里有几个函数能完成这个任务，诸如，`BitBlt` 和 `StretchBlt`。虽然 `StretBlt` 能伸展或压缩位图以便让它适合输出区域的大小，而 `BitBlt` 仅仅是将一个 DC 的内容复制到另一个 DC 中，但是 `BitBlt` 更快速。为了简单，我们在这里使用 `BitBlt` 函数。 `BitBlt` 函数定义如下:

    ```
    BitBlt  proto  hdcDest:DWORD, nxDest:DWORD, nyDest:DWORD, nWidth:DWORD, nHeight:DWORD, hdcSrc:DWORD, nxSrc:DWORD, nySrc:DWORD, dwROP:DWORD
    ```

    `hdcDest` 是作为位图传输操作目的地的设备上下文的句柄。
    `hdcDest` 作为位图转移操作目的地的设备描述表的句柄。

    `nxDest`, `nyDest` 是输出区域左上角的坐标。
    `nxDest` ，`nyDest` 是输出区域左上角的坐标值。

    `nWidth`, `nHeight` 是输出区域的宽度和高度。
    `nWidth` ，`nHeight` 是输出区域的高度和宽度。

    `hdcSrc` 是作为位图传输操作源的设备上下文的句柄。
    `hdcSrc` 作为位图转移操作源的设备描述表的句柄。

    `nxSrc`, `nySrc` 是源矩形区域左上角的坐标。
    `NxSrc`，`nySrc` 源 DC 中需要复制的矩形区域的左上角坐标。

    `dwROP` 是光栅操作码（因此缩写为 ROP），它控制如何组合位图的颜色数据与输出区域上现有的颜色数据以产生最终结果。大多数时候，你只希望用新数据覆盖现有颜色数据。
    `DwRop` 是光栅操作码。它用于确定如何将位图的颜色数据和在输出区域上已经存在的颜色数据组合，以便让输出区域达到最终结果。大多数时候，你只应该用新的数据重写已经存在的颜色数据。

6.  当你完成位图的操作后，使用 `DeleteObject` API 调用将其删除。

    当你完成了位图的操作，调用 `DeleteObject` API 函数将它删除。

就是这样！总结一下，你需要将位图放入资源脚本中。然后使用 `LoadBitmap` 从资源中加载它。你将获得位图句柄。接下来，获取你想将位图画在上面的区域的设备上下文句柄。然后创建一个与刚刚获取的设备上下文兼容的内存设备上下文。将位图选择到内存 DC 中，然后将内存 DC 的内容复制到真实的 DC 中。

就是这样！现在，重述要点，你需要放置位图到资源脚本中。然后用 `LoadBitmap` 函数从资源文件中装载它。你将得到该位图的句柄。下一步，获取你想把位图画在上面的那块区域的设备描述表句柄。然后创建一内存设备描述表，这一设备描述表和你刚刚获得的设备描述表一致。选取位图到内存设备描述表中，然后将内存 DC 的内容复制到真正的 DC 中。

## Example Code
## 例子代码：

```
.386

.model flat,stdcall
```



## 汇编代码

```asm
option casemap:none

include \masm32\include\windows.inc

include \masm32\include\user32.inc

include \masm32\include\kernel32.inc

include \masm32\include\gdi32.inc

includelib \masm32\lib\user32.lib

includelib \masm32\lib\kernel32.lib

includelib \masm32\lib\gdi32.lib

WinMain proto :DWORD,:DWORD,:DWORD,:DWORD

IDB_MAIN   equ 1

.data

ClassName db "SimpleWin32ASMBitmapClass",0

AppName  db "Win32ASM Simple Bitmap Example",0

.data?

hInstance HINSTANCE ?

CommandLine LPSTR ?

hBitmap dd ?

.code

start:

 invoke GetModuleHandle, NULL

 mov     hInstance,eax

 invoke GetCommandLine

 mov     CommandLine,eax

 invoke WinMain, hInstance,NULL,CommandLine, SW_SHOWDEFAULT

 invoke ExitProcess,eax

WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD

 LOCAL wc:WNDCLASSEX

 LOCAL msg:MSG

 LOCAL hwnd:HWND

 mov    wc.cbSize,SIZEOF WNDCLASSEX

 mov    wc.style, CS_HREDRAW or CS_VREDRAW

 mov    wc.lpfnWndProc, OFFSET WndProc

 mov    wc.cbClsExtra,NULL

 mov    wc.cbWndExtra,NULL

 push   hInstance

 pop    wc.hInstance

 mov    wc.hbrBackground,COLOR_WINDOW+1

 mov    wc.lpszMenuName,NULL

 mov    wc.lpszClassName,OFFSET ClassName

 invoke LoadIcon,NULL,IDI_APPLICATION

 mov    wc.hIcon,eax

 mov    wc.hIconSm,eax

 invoke LoadCursor,NULL,IDC_ARROW

 mov    wc.hCursor,eax

 invoke RegisterClassEx, addr wc

 INVOKE CreateWindowEx,NULL,ADDR ClassName,ADDR AppName,\

            WS_OVERLAPPEDWINDOW,CW_USEDEFAULT,\

            CW_USEDEFAULT,CW_USEDEFAULT,CW_USEDEFAULT,NULL,NULL,\

            hInst,NULL

 mov    hwnd,eax

 invoke ShowWindow, hwnd,SW_SHOWNORMAL

 invoke UpdateWindow, hwnd

 .while TRUE

  invoke GetMessage, ADDR msg,NULL,0,0

  .break .if (!eax)

  invoke TranslateMessage, ADDR msg

  invoke DispatchMessage, ADDR msg

 .endw

 mov      eax,msg.wParam

 ret

WinMain endp

WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM

   LOCAL ps:PAINTSTRUCT

   LOCAL hdc:HDC

   LOCAL hMemDC:HDC

   LOCAL rect:RECT

   .if uMsg==WM_CREATE

      invoke LoadBitmap,hInstance,IDB_MAIN

      mov hBitmap,eax

   .elseif uMsg==WM_PAINT

      invoke BeginPaint,hWnd,addr ps

      mov    hdc,eax

      invoke CreateCompatibleDC,hdc

      mov    hMemDC,eax

      invoke SelectObject,hMemDC,hBitmap

      invoke GetClientRect,hWnd,addr rect

      invoke BitBlt,hdc,0,0,rect.right,rect.bottom,hMemDC,0,0,SRCCOPY

      invoke DeleteDC,hMemDC

      invoke EndPaint,hWnd,addr ps

 .elseif uMsg==WM_DESTROY

  invoke DeleteObject,hBitmap

  invoke PostQuitMessage,NULL

 .ELSE

  invoke DefWindowProc,hWnd,uMsg,wParam,lParam

  ret

 .ENDIF

 xor eax,eax

 ret

WndProc endp

end start

;---------------------------------------------------------------------

;                     The resource script

资源脚本

;---------------------------------------------------------------------

#define IDB_MAIN 1

IDB_MAIN BITMAP "tweety78.bmp"
```

## 分析

在这一课中并不需要很多的分析。

```c
#define IDB_MAIN 1

IDB_MAIN BITMAP "tweety78.bmp"
```

定义了一个名为 `IDB_MAIN` 的常量，并指派 1 作为它的值。然后用这个常量作为位图资源标识符。包含在资源中的位图是 `tweety.bmp`，它和资源脚本文件应该保存在同一个文件夹中。

```asm
   .if uMsg==WM_CREATE

      invoke LoadBitmap,hInstance,IDB_MAIN

      mov hBitmap,eax
```

在响应 `WM_CREATE` 消息时，我们调用 `LoadBitmap` 函数来从资源中装载位图，并传递资源标识符作为 API 函数的第二个参数。当函数返回时，我们获得位图的句柄。

现在，位图已经被装载，我们能将它画在我们主窗口的客户区上。

```asm
   .elseif uMsg==WM_PAINT

      invoke BeginPaint,hWnd,addr ps

      mov    hdc,eax

      invoke CreateCompatibleDC,hdc

      mov    hMemDC,eax

      invoke SelectObject,hMemDC,hBitmap

      invoke GetClientRect,hWnd,addr rect

      invoke BitBlt,hdc,0,0,rect.right,rect.bottom,hMemDC,0,0,SRCCOPY

      invoke DeleteDC,hMemDC

      invoke EndPaint,hWnd,addr ps
```

我们选择在响应 `WM_PAINT` 消息的时候绘画位图。首先，我们调用 `BeginPaint` 函数来获得设备描述表的句柄，然后，我们用 `CreateCompatibleDC` 函数来创建与之一致的内存 DC。下一步，用 `SelectObject` 函数将位图选进内存 DC。再用 `GetClientRect` 函数确定客户区的尺寸大小。现在，我们通过调用 `BitBlt` 函数将位图从内存 DC 复制到真正的 DC 中以便让位图能显示在客户区中。当绘画完成时，我们以后再也用不上内存 DC 所以我们用 `DeleteDC` 删除它。然后用 `EndPaint` 函数释放客户区 DC，结束绘画。

```asm
 .elseif uMsg==WM_DESTROY

  invoke DeleteObject,hBitmap

  invoke PostQuitMessage,NULL
```

当我们在也不需要位图时，我们用 `DeleteObject` 函数将它删除。

***

本文档源自 Iczelion 的汇编页面

风向改变翻译于 2008-3-2
