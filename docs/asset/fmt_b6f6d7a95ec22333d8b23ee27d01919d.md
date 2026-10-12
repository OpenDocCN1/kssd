
# 汇编学习之多线程技术的实现

> 原文：[`www.pediy.com/kssd/pediy10/78004.html`](https://www.pediy.com/kssd/pediy10/78004.html)

现在调试程序时经常会碰到多线程程序。由于多线程可以在一个程序中实现多个任务，所以在今天的程序开发中运用较多。但实现它时，高级语言和汇编有什么区别呢？怎么实现呢？

首先说明一下，每一个线程都有自己独立的堆栈，局部变量也是私有的。

创建线程的语法是：

代码:

```
CreateThread proto lpThreadAttributes:DWORD,
        dwStackSize:DWORD,
        lpStartAddress:DWORD,
        lpParameter:DWORD,
        dwCreationFlags:DWORD,
        lpThreadID:DWORD
```

线程函数通过`ret`返回，`Windows`会隐含调用`ExitThread`，所以可以不用手动调用`ExitThread`。类似于`VC`中`CreateThread`函数的结束过程。极力不推荐用`TerminateThread`函数来结束线程，因为这个函数直接结束线程，而线程没有进行清理，容易丢失数据或内存泄露。

关于事件对象问题：事件对象处于无信号状态，表示线程“睡眠”，线程不接受`CPU`时间片；当处于有信号状态，`Windows`唤醒线程，进入`CPU`时间片。

言归正传，实现代码：

代码:

```
WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM

  .IF uMsg == WM_COMMAND
    mov eax, wParam
    .IF lParam == 0
      .IF ax == IDM_CREATE_THREAD
        mov eax, OFFSET ThreadProc
        invoke CreateThread, NULL, eax, 0, ADDR ThreadID
        invoke CloseHandle, eax
      .ELSE
        invoke DestroyWindow, hWnd
      .ENDIF
    .ENDIF
  .ENDIF
  ret

WndProc endp
```

程序中调用了关闭句柄，但是关闭线程句柄是不会关闭线程运行的，只是说以后不用这个句柄了，所以要注意！
