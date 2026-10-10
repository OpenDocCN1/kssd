

# Win32 asm 第二十五课：简单的位图

> 原文：[`www.pediy.com/kssd/pediy10/60534.html`](https://www.pediy.com/kssd/pediy10/60534.html)

Tutorial 25: Simple Bitmap  
第二十五课：简单的位图  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
In this tutorial, we will learn how to use bitmap in our program. To be exact, we will learn how to display a bitmap in the client area of our window.  
在这一课中，我们将学习如何在我们的程序中使用位图。为了更精确，我们将学习如何在我们窗口客户区显示一个位图。  
 Download the example.   
Theory  
原理  
Bitmaps can be thought of as pictures stored in computer. There are many picture formats used with computers but Windows only natively supports Windows Bitmap Graphics files (.bmp). The bitmaps I'll refer to in this tutorial are Windows bitmap graphics files. The easiest way to use a bitmap is to use it as a resource. There are two ways to do that. You can include the bitmap in the resource definition file (.rc) as follows:   
位图就是存储在计算机中的图像。计算机中的图像可以使用很多的格式，但是 windows 仅支持 windows Bitmap Graphics 文件（.bmp--位图文件）。我将在本课中涉及的就是位图文件。使用一个位图最简单的方式就是将它作为一种资源文件。有两种方式可以这样做。你能用下面的定义来在资源文件中包含一个位图:   
    
#define IDB\_MYBITMAP   100   
IDB\_MYBITMAP  BITMAP  "c:\\project\\example.bmp"  
  
This method uses a constant to represent the bitmap. The first line just creates a constant named IDB\_MYBITMAP which has the value of 100. We will use this label to refer to the bitmap in the program. The next line declares a bitmap resource. It tells the resource compiler where to find the actual bmp file.   
这种方法使用一个常数来表示位图（宏定义）。第一行仅仅是创建一名为 IDB\_MYBITMAP，值为 100 的常量。我们将在程序中用这个标号来引用这个位图。下一行声明一位图资源。它告诉资源编译器在那里能找到真实的位图文件。  
  
The other method uses a name to represent the bitmap as follows:   
另外一种方法是使用一名称（字符串）来表示位图，向下面这样：  
MyBitMap  BITMAP "c:\\project\\example.bmp"  
This method requires that you refer to the bitmap in your program by the string "MyBitMap" instead of a value.   
这种方法要求在程序中引用位图时，是通过一字符串“MyBitMap”引用而不在是一数值。  
  
Either method works fine as long as you know which method you're using.   
Now that we put the bitmap in the resource file, we can go on with the steps in displaying it in the client area of our window.   
两种方法都工作的很好，只要你知道你正在使用的是哪一种方法。现在我们在资源文件中放置一位图， 下一步就是让它显示在我们窗口的客户区上。  
1.  call LoadBitmap to get the bitmap handle. LoadBitmap has the following definition:   
调用 LoadBitmap 来获得一位图句柄。LoadBitmap 定义如下：  
LoadBitmap proto hInstance:HINSTANCE, lpBitmapName:LPSTR   
  
This function returns a bitmap handle. hInstance is the instance handle of our program. lpBitmapName is a pointer to the string that is the name of the bitmap (incase you use the second method to refer to the bitmap). If you use a constant to refer to the bitmap (like IDB\_MYBITMAP), you can put its value here. (In the example above it would be 100). A short example is in order:   
这个函数返回一位图句柄。 hInstance 是我们程序的实例句柄。LpBitmapName 是指向位图名称的指针（包括用第二种方法引用位图）。如果你想使用一个常数来引用位图，你能把它的值放在这儿。（在上面的例子中，它的值就是 100）。一个简短的例子如下：   
    
First Method:    
第一种方法：  
.386   
.model flat, stdcall   
................   
.const   
IDB\_MYBITMAP    equ 100   
...............   
.data?   
hInstance  dd ?   
..............   
.code   
.............   
    invoke GetModuleHandle,NULL   
    mov hInstance,eax   
............   
    invoke LoadBitmap,hInstance,IDB\_MYBITMAP   
...........   
Second Method:   
第二种方法：  
.386   
.model flat, stdcall   
................   
.data   
BitmapName  db "MyBitMap",0   
...............   
.data?   
hInstance  dd ?   
..............   
.code   
.............   
    invoke GetModuleHandle,NULL   
    mov hInstance,eax   
............   
    invoke LoadBitmap,hInstance,addr BitmapName   
...........   
1.  Obtain a handle to device context (DC). You can obtain this handle by calling BeginPaint in response to WM\_PAINT message or by calling GetDC anywhere.   
获得一设备描述表的句柄 （DC）。你能在响应 WM\_PAINT 消息时调用 BeginPaint 函数获得这个句柄。也可以在任何地方调用 GetDC 获取 。  
2.  Create a memory device context which has the same attribute as the device context we just obtained. The idea here is to create a kind of "hidden" drawing surface which we can draw the bitmap on. When we are finished with the operation, we just copy the content of the hidden drawing surface to the actual device context in one function call. It's an example of double-buffer technique used for fast display of pictures on the screen. You can create this "hidden" drawing surface by calling CreateCompatibleDC.   
创建一内存设备环境，作为设备描述表的它们都有相同的属性，我们仅仅是获得它。这儿的一种观念是创建一种隐藏画纸，我们能将位图画在上面。当我们完成这个操作后，我们仅仅是在一个函数调用中，将隐藏的画纸复制到实际的设备现场（设备描述表）。这就是用于在屏幕上快速显示图像的双缓冲技术的一种典型。你能通过调用 CreateCompatibleDC 函数来创建这张隐藏的画纸。  
CreateCompatibleDC  proto  hdc:HDC   
  
If this function succeeds, it returns the handle of the memory device context in eax. hdc is the handle to the device context that you want the memory DC to be compatible with.   
如果这个函数成功，它在 eax 中返回内存设备描述表的句柄。Hdc 是设备描述表句柄，它和你想要的内存 DC 是一致的。   
1.  Now that you got a hidden drawing surface, you can draw on it by selecting the bitmap into it. This is done by calling SelectObject with the handle to the memory DC as the first parameter and the bitmap handle as the second parameter. SelectObject has the following definition:   
现在，你得到了那张隐藏的画纸，你能将你选择的位图画在它上面。这可以通过调用 SelectObject 函数来完成。其中，传入内存 DC 的句柄作为 SelectObject 函数的第一个参数，并且用位图句柄做为函数的第二个参数。SelectObject 函数定义如下：  
SelectObject   proto  hdc:HDC, hGdiObject:DWORD   
1.  The bitmap is drawn on the memory device context now. All we need to do here is to copy it to the actual display device, namely the true device context. There are several functions that can perform this operation such as BitBlt and StretchBlt. BitBlt just copies the content of one DC to another so it's fast while StretchBlt can stretch or compress the bitmap to fit the output area. We will use BitBlt here for simplicity. BitBlt has the following definition:   
现在，位图已经画在内存设备描述表上了。在这儿我们需要做的是将位图复制到真正的显示设备中，也就是真正的设备描述表。这里有几个函数能完成这个任务，诸如，BitBlt 和 StretchBlt。虽然 StretBlt 能伸展或压缩位图以便让它适合输出区域的大小，而 BitBlt 仅仅是将一个 DC 的内容复制到另一个 DC 中，但是 BitBlt 更快速。为了简单，我们在这里使用 BitBlt 函数。 BitBlt 函数定义如下:  
BitBlt  proto  hdcDest:DWORD, nxDest:DWORD, nyDest:DWORD, nWidth:DWORD, nHeight:DWORD, hdcSrc:DWORD, nxSrc:DWORD, nySrc:DWORD, dwROP:DWORD   
    
hdcDest is the handle of the device context that serves as the destination of bitmap transfer operation   
hdcDest  作为位图转移操作目的地的设备描述表的句柄。  
nxDest, nyDest are the coordinate of the upper left corner of the output area   
nxDest ，nyDest 是输出区域左上角的坐标值。  
nWidth, nHeight are the width and height of the output area   
nWidth ，nHeight 是输出区域的高度和宽度。  
hdcSrc is the handle of the device context that serves as the source of bitmap transfer operation   
hdcSrc 作为位图转移操作源的设备描述表的句柄。  
nxSrc, nySrc are the coordinate of the upper left corner of the source rectangle.   
NxSrc，nySrc 源 DC 中需要复制的矩形区域的左上角坐标。  
dwROP is the raster-operation code (hence the acronym ROP) that governs how to combine the color data of the bitmap to the existing color data on the output area to achieve the final result. Most of the time, you only want to overwrite the existing color data with the new one.   
DwRop 是光栅操作码。它用于确定如何将位图的颜色数据和在输出区域上已经存在的颜色数据组合，以便让输出区域达到最终结果。大多数时候，你只应该用新的数据重写已经存在的颜色数据。  
1.  When you're done with the bitmap, delete it with DeleteObject API call.   
当你完成了位图的操作，调用 DeleteObject API 函数将它删除。  
That's it! To recapitulate, you need to put the bitmap into the resource scipt. Then load it from the resource with LoadBitmap. You'll get the bitmap handle. Next you obtain the handle to the device context of the area you want to paint the bitmap on. Then you create a memory device context that is compatible with the device context you just obtained. Select the bitmap into the memory DC then copy the content of the memory DC to the real DC.   
就是这样！现在，重述要点，你需要放置位图到资源脚本中。然后用 LoadBitmap 函数从资源文件中装载它。你将得到该位图的句柄。下一步，获取你想把位图画在上面的那块区域的设备描述表句柄。然后创建一内存设备描述表，这一设备描述表和你刚刚获得的设备描述表一致。选取位图到内存设备描述表中，然后将内存 DC 的内容复制到真正的 DC 中。  
Example Code:  
例子代码：  
.386   
.model flat,stdcall   
option casemap:none   
include \\masm32\\include\\windows.inc   
include \\masm32\\include\\user32.inc   
include \\masm32\\include\\kernel32.inc   
include \\masm32\\include\\gdi32.inc   
includelib \\masm32\\lib\\user32.lib   
includelib \\masm32\\lib\\kernel32.lib   
includelib \\masm32\\lib\\gdi32.lib   
WinMain proto :DWORD,:DWORD,:DWORD,:DWORD   
IDB\_MAIN   equ 1   
.data   
ClassName db "SimpleWin32ASMBitmapClass",0   
AppName  db "Win32ASM Simple Bitmap Example",0   
.data?   
hInstance HINSTANCE ?   
CommandLine LPSTR ?   
hBitmap dd ?   
.code   
start:   
 invoke GetModuleHandle, NULL   
 mov    hInstance,eax   
 invoke GetCommandLine   
 mov    CommandLine,eax   
 invoke WinMain, hInstance,NULL,CommandLine, SW\_SHOWDEFAULT   
 invoke ExitProcess,eax   
WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD   
 LOCAL wc:WNDCLASSEX   
 LOCAL msg:MSG   
 LOCAL hwnd:HWND   
 mov   wc.cbSize,SIZEOF WNDCLASSEX   
 mov   wc.style, CS\_HREDRAW or CS\_VREDRAW   
 mov   wc.lpfnWndProc, OFFSET WndProc   
 mov   wc.cbClsExtra,NULL   
 mov   wc.cbWndExtra,NULL   
 push  hInstance   
 pop   wc.hInstance   
 mov   wc.hbrBackground,COLOR\_WINDOW+1   
 mov   wc.lpszMenuName,NULL   
 mov   wc.lpszClassName,OFFSET ClassName   
 invoke LoadIcon,NULL,IDI\_APPLICATION   
 mov   wc.hIcon,eax   
 mov   wc.hIconSm,eax   
 invoke LoadCursor,NULL,IDC\_ARROW   
 mov   wc.hCursor,eax   
 invoke RegisterClassEx, addr wc   
 INVOKE CreateWindowEx,NULL,ADDR ClassName,ADDR AppName,\\   
           WS\_OVERLAPPEDWINDOW,CW\_USEDEFAULT,\\   
           CW\_USEDEFAULT,CW\_USEDEFAULT,CW\_USEDEFAULT,NULL,NULL,\\   
           hInst,NULL   
 mov   hwnd,eax   
 invoke ShowWindow, hwnd,SW\_SHOWNORMAL   
 invoke UpdateWindow, hwnd   
 .while TRUE   
  invoke GetMessage, ADDR msg,NULL,0,0   
  .break .if (!eax)   
  invoke TranslateMessage, ADDR msg   
  invoke DispatchMessage, ADDR msg   
 .endw   
 mov     eax,msg.wParam   
 ret   
WinMain endp   
WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM   
   LOCAL ps:PAINTSTRUCT   
   LOCAL hdc:HDC   
   LOCAL hMemDC:HDC   
   LOCAL rect:RECT   
   .if uMsg==WM\_CREATE   
      invoke LoadBitmap,hInstance,IDB\_MAIN   
      mov hBitmap,eax   
   .elseif uMsg==WM\_PAINT   
      invoke BeginPaint,hWnd,addr ps   
      mov    hdc,eax   
      invoke CreateCompatibleDC,hdc   
      mov    hMemDC,eax   
      invoke SelectObject,hMemDC,hBitmap   
      invoke GetClientRect,hWnd,addr rect   
      invoke BitBlt,hdc,0,0,rect.right,rect.bottom,hMemDC,0,0,SRCCOPY   
      invoke DeleteDC,hMemDC   
      invoke EndPaint,hWnd,addr ps   
 .elseif uMsg==WM\_DESTROY   
  invoke DeleteObject,hBitmap   
  invoke PostQuitMessage,NULL   
 .ELSE   
  invoke DefWindowProc,hWnd,uMsg,wParam,lParam   
  ret   
 .ENDIF   
 xor eax,eax   
 ret   
WndProc endp   
end start   
;---------------------------------------------------------------------   
;                            The resource script   
资源脚本  
;---------------------------------------------------------------------   
#define IDB\_MAIN 1   
IDB\_MAIN BITMAP "tweety78.bmp"   
Analysis:  
分析：  
There is not much to analyze in this tutorial ;)   
在这一课中并不需要很多的分析。  
    
#define IDB\_MAIN 1   
IDB\_MAIN BITMAP "tweety78.bmp"   
  
Define a constant named IDB\_MAIN, assign 1 as its value. And then use that constant as the bitmap resource identifier. The bitmap file to be included in the resource is "tweety78.bmp" which resides in the same folder as the resource script.   
定义了一个名为 IDB\_MAIN 的常量，并指派 1 作为它的值。然后用这个常量作为位图资源标识符。包含在资源中的位图是“tweety.bmp”，它和资源脚本文件应该保存在同一个文件夹中。  
   .if uMsg==WM\_CREATE   
      invoke LoadBitmap,hInstance,IDB\_MAIN   
      mov hBitmap,eax   
In response to WM\_CREATE, we call LoadBitmap to load the bitmap from the resource, passing the bitmap's resource identifier as the second parameter to the API. We get the handle to the bitmap when the function returns.   
在响应 WM\_CREAT 消息时，我们调用 LoadBitmap 函数来从资源中装载位图，并传递资源标识符作为 API 函数的第二个参数。当函数返回时，我们获得位图的句柄。  
Now that the bitmap is loaded, we can paint it in the client area of our main window.   
现在，位图已经被装载，我们能将它画在我们主窗口的客户区上。  
   .elseif uMsg==WM\_PAINT   
      invoke BeginPaint,hWnd,addr ps   
      mov    hdc,eax   
      invoke CreateCompatibleDC,hdc   
      mov    hMemDC,eax   
      invoke SelectObject,hMemDC,hBitmap   
      invoke GetClientRect,hWnd,addr rect   
      invoke BitBlt,hdc,0,0,rect.right,rect.bottom,hMemDC,0,0,SRCCOPY   
      invoke DeleteDC,hMemDC   
      invoke EndPaint,hWnd,addr ps   
We choose to paint the bitmap in response to WM\_PAINT message. We first call BeginPaint to obtain the handle to the device context. Then we create a compatible memory DC with CreateCompatibleDC. Next select the bitmap into the memory DC with SelectObject. Determine the dimension of the client area with GetClientRect. Now we can display the bitmap in the client area by calling BitBlt which copies the bitmap from the memory DC to the real DC. When the painting is done, we have no further need for the memory DC so we delete it with DeleteDC. End painting session with EndPaint.   
我们选择在响应 WM\_PAINT 消息的时候绘画位图。首先，我们调用 BeginPaint 函数来获得设备描述表的句柄，然后，我们用 CreateCompatibleDC 函数来创建与之一致的内存 DC。下一步，用 SelectObject 函数将位图选进内存 DC。再用 GetClientRect 函数确定客户区的尺寸大小。现在，我们通过调用 BitBlt 函数将位图从内存 DC 复制到真正的 DC 中以便让位图能显示在客户区中。当绘画完成时，我们以后再也用不上内存 DC 所以我们用 DeleteDc 删除它。然后用 EndPaint 函释放客户区 DC，结束绘画。  
 .elseif uMsg==WM\_DESTROY   
  invoke DeleteObject,hBitmap   
  invoke PostQuitMessage,NULL   
When we don't need the bitmap anymore, we delete it with DeleteObject   
当我们在也不需要位图时，我们用 DeleteObject 函数将它删除。  
  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  
This article come from Iczelion's asm page  
风向改变翻译于 2008-3-2