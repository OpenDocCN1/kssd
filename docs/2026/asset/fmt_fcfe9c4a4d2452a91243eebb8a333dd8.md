
# lichsword的《加密解密》学习笔记...更新中

标 题: 【分享】lichsword的《加密解密》学习笔记...更新中

作 者: `lichsword`

时 间: 2010-01-08 11:30 ---?

链 接: <http://bbs.pediy.com/showthread.php?t=104782&prefixid=139_xuexi>

从2010\1\8开始，在这此帖整理自己的学习笔记：

1>2009\2\7就买到了《加密与解密》第三版，可到现在只会对无壳的软件暴破（太懒了！），自己都无颜了，再这样下去，怕是走不进逆向的世界了。
2>在此帖中得到无形的监督
3>相信也有雪友在学习中，希望能相互交流，共同进步。

故，开始《加》的学习。

----------------------------------------------开始-------------------------------------------------------------

## 第一章 基础知识

常见的`intel`体系芯片用`Little-Endian`类(逆序)

某些`RISC`架构的`CPU`，eg:`IBM`的`POWER-PC`等为`Big-Endian`类(正序)

`Big-Endian`:高字节存入低地址，低字节存入高地址。(少见的情况)

`Little-Endian`:低字节存入低地址，高字节存入高地址。(这种是我们的机子的顺序)

所以当我们`PEDIY`时，就是用逆序存放的。

### 1.2 什么是UNICODE？

额，区别于`ASCII`，大二学`C`时，总说“啊斯克码”，全称 美国信息交换标准码

`American Standard Code for Information Interchange`

`Unicode`是`ASCII`的扩展，eg:

`70h` `65h` `64h` `69h` `79h`(`ASCII`的“pediy”) 7位编码

`0070h` `0065h` `0064h` `0069h` `0079h`(`Unicode`的“pediy”) 16位编码

### 1.3 什么是API？

额，我用了不少`API`了，应用程序编程接口(`Application Programming Interface`)

`windows`提供的丰富的`API`给程序员使用，这些`API`封装到`DLL`中。

系统最常用的`DLL`：

`Kernel`:16bit `KRNL386.EXE` + 32bit `KERNEL32.DLL`，包括进程与线程控制、内存管理、文件访问等核心功能服务

听名字就知道`kernel`--1.谷粒;仁,核2.要点,中心,核心

难怪，我们用`OD`时，经常一个`CALL`跑到系统领空，那就是到这些`DLL`中去了。

`User`:16bit `USER.EXE` + 32bit `USER32.DLL`,处理 键盘鼠标输入、窗口和菜单管理等用户接口。

`GDI`:16bit `GDI.EXE` + 32bit `GDI32.DLL`,这个也很常用，如我们用画直线画点等`API`。作用是 允许用户在屏幕和打印机上显示文本和图形。

英文是`Graphics Device Interface`图形设备接口。

以上三个几乎是用汇编写程序必用的`DLL`：

```
;--------include files--------
include gdi32.inc
includelib gdi32.lib
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib
...
```

之外还有一些`DLL`，如下所示，先混个脸熟:

`COMDLG32.DLL`:通用对话框`DLL`。这个也相当常用了，

使用

打开文件对话框

保存文件对话框

字体选择对话框

颜色选择对话框

查找和替换文本对话框

页面设置对话框

浏览目录对话框 等相关`API`，就要用到`comdlg32.dll`

`Comctl32.dll`:标题栏、进度条、状态栏、滚动条、工具栏、提示文本、树型视图、列表视图等控件的相关`API`，都要用到这个库。

`wsock32.dll`:网络编程接口库。对网络数据包的操作用到。

### 1.4 常用API函数

参考：`MSDN`或百度`google`或..

由于字符分`ASCII`和`Unicode`编码，所以一些与字符相关的`API`，就分成2类。

以`A`结尾(单字节方式)以`W`结尾(宽字节方式)

eg:

`GetWindowText`:获取窗口文本 `GetWindowTextA`/`GetWindowTextW`

`GetDlgItemText`:获取对话框文本 `GetDlgItemTextA`/`GetDlgItemTextW`

`MessageBox`:创建和显示信息框 `MessageBoxA`/`MessageBoxW`

我们写程序时，只用调用`MessageBox`就可以,编译器会根据设置来采用`A`/`W`。

[思考]但如果用`OD`对`API`下断点时，我们是对以`A`结尾的`API`下断 还是 对以`W`结尾的`API`下断呢？

我的想法是，先用`Ctrl+N`打开导入库，查看是什么样的`API`，再下断，就可以了。

当然，也可以先下`A`型的，不行，再下`W`型的。一次次试。

### 1.5 什么是句柄

句柄就是一个标识符，本质是一个整数。由于表示的东东的类型不同，分为：

窗口句柄

资源句柄(细分为 字符串句柄、声音`wav`资源句柄、图片句柄等等)

总之就是 一个用于保存每个东东的索引的`DWORD`型整数。比如：

```
;----------------------------------------------
;EQU define
;----------------------------------------------
DLG_MAIN EQU 1000 对话框
ICO_MAIN EQU 100 程序图标
;... ...s
RED_TXT EQU 1001 Richedit控件
IDM_OPENFILE EQU 10002 以下都是菜单项的句柄
IDM_EXIT EQU 10003
IDM_BASICINFO EQU 10004
IDM_IMPORT EQU 10005
IDM_EXPORT EQU 10006
IDM_RESOURCE EQU 10007
IDM_RELOCATION EQU 10008
IDM_VERSION EQU 10009
```

### 1.6 windows各操作系统与Unicode的处理

`ASCIId`码只有7位，保存的信息少，而`Unicode`为16位，可以保存更多的信息，如我们的汉字就是2个`UNICODE`大小。

一、9x系统，即`win95-win98`(不是很严谨)的处理如下：

将`UNICODE`字符参数先转换成`ASCII`类型--->再调用`ASCII`型的`API`

eg:

```
windows98里MessageBoxW函数的内部定义
int MessageBoxW(
MessageBoxExW{ // 调用MessageBoxExW()函数
WideCharToMultiByte(); // 取得要显示文本的长度
GlobalAlloc(); // 按字符串长度分配内存
WideCharToMultiByte(); // 将UNICODE文本转换成ANSI字符串
WideCharToMultiByte(); // 取得消息框标题文本的长度
GlobalAlloc(); // 按字符串长度分配内存
WideCharToMultiByte(); // 将UNICODE文本转换成ANSI字符串
MessageBoxExA(); // 最终还是调用ANSI版的MessageBoxExA();函数显示窗口
GlobalFree(); // 释放内存
GlobalFree(); // 释放内存
}
);
```

二、`NT/2000/XP`系统如何处理`Unicode`

`NT`及以上系统好像完全反过来了。

将`ASNI`字符串先转换成`UNICODE`，再调用`W`型`API`，因为`NT`及以上系统是的系统核心完全是用`Unicode`函数工作的。

```
eg: int MessageBoxA(
MessageBoxA{ // 调用MessageBoxExA()函数
MBToWCSEx(); // 将消息框主体文字转换成Unicode字符串
MBToWCSEx(); // 将消息框标题文字转换成Unicode字符串
MessageBoxExW() // 调用MessageBoxExW()函数
HeapFree() // 释放内存
}
);
```

### 1.7 windows消息机制

［笑话一则］学到这里，我想起最近发生的事。是在`QQ`上和朋友聊天，朋友是学软件工程的，他居然不知道`C`可以写窗口程序，他理解为`C`只能写`console`这种黑框框的控制台程序。

并和他同学一致讨论，认为，`C++`才能写窗口程序！

我的理解是：只要在理解`windows`消息机制的基础上，调用相关`API`，就可以写出窗口程序。`C`当然可以写窗口，因为`C`可以调用相关`API`，我也写过不少了。再说，好多网络游戏，单机游戏就是用`C`写的，有名的拉莫斯的游戏引擎就是用`C`写的，额，说远了，打住，说回来。

这里总结 `C`语言 和汇编语言 的`window`消息机制

先写汇编的框架:>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```
;--------------------------------------------------------
;time:2009-11-3 13:46
;author:lichsword
;function:
;
;--------------------------------------------------------
.386
.model flat,stdcall
option casemap:none
;----------------------------------------------
;include file define
;----------------------------------------------
include windows.inc
include gdi32.inc
includelib gdi32.lib
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib
;----------------------------------------------
;EQU define
;----------------------------------------------
ICO_MAIN EQU 100
;... ...

;----------------------------------------------
;.data segment
;----------------------------------------------
.data?

hInstance dd ?
hWinMain dd ?
hIconMain dd ?
.data
;... ...
.const
szClassName db 'MyClassName',0
szCaptionName db 'My Window Caption',0
szText db '这是一个窗口的框架。',0
;---------------------------------------------
;.code segment
;---------------------------------------------
.code
;---------------------------------------------
;Main Window's process
;---------------------------------------------
_ProcWinMain proc uses ebx edi esi hWnd,uMsg,wParam,lParam

LOCAL @stPs:PAINTSTRUCT
LOCAL @stRect:RECT
LOCAL @hDc

mov eax,uMsg
;-------------------------------------------
.if eax==WM_PAINT
invoke BeginPaint,hWnd,addr @stPs
mov @hDc,eax

invoke GetClientRect,hWnd,addr @stRect
invoke DrawText,@hDc,addr szText,-1,addr @stRect,DT_SINGLELINE or DT_CENTER or DT_VCENTER
invoke EndPaint,hWnd,addr @stPs
;--------------------------------
;add your code here...
;--------------------------------
.elseif eax==WM_COMMAND
;--------------------------------
.elseif eax==WM_CREATE
invoke SendMessage,hWnd,WM_SETICON,ICON_BIG,hIconMain



.elseif eax==WM_CLOSE
invoke DestroyWindow,hWinMain
invoke PostQuitMessage,NULL
.else
invoke DefWindowProc,hWnd,uMsg,wParam,lParam
ret
.endif

xor eax,eax
ret

_ProcWinMain endp
;---------------------------------------------
;Main Window
;---------------------------------------------
_WinMain proc

LOCAL @stWndClass:WNDCLASSEX
LOCAL @stMsg:MSG

invoke GetModuleHandle,NULL
mov hInstance,eax

invoke RtlZeroMemory,addr @stWndClass,sizeof @stWndClass

;---------------------------------
;注册窗口类
;---------------------------------
invoke LoadCursor,0,IDC_ARROW
mov @stWndClass.hCursor,eax

invoke LoadIcon,hInstance,ICO_MAIN
mov hIconMain,eax

push hInstance
pop @stWndClass.hInstance

mov @stWndClass.cbSize,sizeof WNDCLASSEX
mov @stWndClass.style,CS_HREDRAW or CS_VREDRAW
mov @stWndClass.lpfnWndProc,offset _ProcWinMain
mov @stWndClass.hbrBackground,COLOR_WINDOW+1
mov @stWndClass.lpszClassName,offset szClassName

invoke RegisterClassEx,addr @stWndClass
;---------------------------------
;Create and show window
;---------------------------------
invoke CreateWindowEx,WS_EX_CLIENTEDGE,\
offset szClassName,offset szCaptionName,\
WS_OVERLAPPEDWINDOW,\
100,100,600,400,\
NULL,NULL,hInstance,NULL
mov hWinMain,eax;save handle of window

invoke ShowWindow,hWinMain,SW_SHOWNORMAL;show window
invoke UpdateWindow,hWinMain ;update window
;---------------------------------
;message circle 消息循环
;---------------------------------
.while TRUE
invoke GetMessage,addr @stMsg,NULL,0,0
.break .if eax==0;when @stMsg = WM_QUIT , eax=0

invoke TranslateMessage,addr @stMsg
invoke DispatchMessage,addr @stMsg

.endw
ret
_WinMain endp
;-------------------------------------------------
start:
call _WinMain
invoke ExitProcess,NULL
end start
```

下面是C的框架：>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
