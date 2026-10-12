
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
