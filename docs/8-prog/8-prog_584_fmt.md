
# Win32 窗口创建和消息处理小程序(MASM 附源代码)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-578.htm`](https://www.pediy.com/kssd/pediy09/pediy09-578.htm)

> Author: Jacky Chou

> 引用: 理论：
>
> Windows 程序中，在写图形用户界面时需要调用大量的标准 Windows Gui 函数。其实这对用户和程序员来说都有好处，对于用户，面对的是
> 同一套标准的窗口，对这些窗口的操作都是一样的，所以使用不同的应用程序时无须重新学习操作。对程序员来说，这些 Gui 源代码都是经过
> 了微软的严格测试，随时拿来就可以用的。当然至于具体地写程序对于程序员来说还是有难度的。为了创建基于窗口的应用程序，必须严格遵
> 守规范。作到这一点并不难，只要用模块化或面向对象的编程方法即可。

下面我就列出在桌面显示一个窗口的几个步骤：

1.  得到您应用程序的句柄(必需)；
2.  得到命令行参数(如果您想从命令行得到参数，可选)；
3.  注册窗口类(必需，除非您使用 Windows 预定义的窗口类，如 `MessageBox` 或 `dialog box`；
4.  产生窗口(必需)；
5.  在桌面显示窗口(必需，除非您不想立即显示它)；
6.  刷新窗口客户区；
7.  进入无限的获取窗口消息的循环；
8.  如果有消息到达，由负责该窗口的窗口回调函数处理；
9.  如果用户关闭窗口，进行退出处理。

相对于单用户的 DOS 下的编程来说，Windows 下的程序框架结构是相当复杂的。但是 Windows 和 DOS 在系统架构上是截然不同的。Windows
是一个多任务的操作系统，故系统中同时有多个应用程序彼此协同运行。这就要求 Windows 程序员必须严格遵守编程规范，并养成良好的编程
风格。

下面列出程序源代码：

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
以下代码 经 RadASM 编译通过，测试成功! (程序只有 4K) 我用 VC++ Win32 Application 做一个完全一样功能的程序，要 168K,果然汇编牛啊
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Programmed by Jacky Chou
;
;     Win32 ASM is Masm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

**代码:**

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;WinMain\_Asm.Inc
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.386                 ;
.model flat,stdcall     ;        
option casemap:none     ;这 3 条不用说了

include windows.inc     ;需要包含的头文件和库文件
include kernel32.inc
includelib kernel32.lib
include user32.inc
includelib user32.lib
include gdi32.inc
includelib gdi32.lib
include comctl32.inc
includelib comctl32.lib

```

**代码:**

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;WinMain\_Asm.Asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include WinMain\_Asm.inc

WinMain proto :DWORD,:DWORD,:DWORD,:DWORD    ;WinMain 函数的申明

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;Data Segment --- initialized data
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.data
WndClassName db "MyFirstWin",0          ;类名
WndAppName   db "WinMainTest",0    ;窗口名
MsgText      db "hello JackyChou",0  ;消息内容
MsgCaption   db "Hello",0    ;消息标题
btnkeyinfo   db "You Have Entered Key ",?,0    
char         db "%c",0
LButtonDown  db "Left Button clicked!",0
RButtonDown  db "Right Buttond clicked!",0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;Data Segment --- Uninitialized data 
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.data?
hInstance  HINSTANCE ?  ;句柄 在 Win32 下，应用程序的句柄和模块的句柄是一样的。您可以把实例句柄看成是您的应用程序的 ID 号。
CommandLine LPSTR ?


;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;Code Segment
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code
start:
invoke GetModuleHandle,NULL  ;获取我们应用程序的句柄
mov hInstance,eax
invoke GetCommandLine  ;您的应用程序不处理命令行那么就无须调用 GetCommandLine，这里只是告诉您如果要调用应该怎么做。
mov CommandLine,eax
invoke WinMain,hInstance,NULL,CommandLine,SW\_SHOWNORMAL    ;调用 WinMain 函数
invoke ExitProcess,eax         ;退出我们的程序，退出代码是从 WinMain 函数返回的 EAX

WinMain proc hInStance:HINSTANCE,hPreInstance:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD

  LOCAL hWnd:HWND      ;创建局部变量
  LOCAL msg:MSG
  LOCAL myWndClass:WNDCLASS
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;窗口类是一个结构题，我们只要像做填空题一样，把结构体中需要填写的内容全部进行相应赋值就可以了。
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  mov myWndClass.style,CS\_HREDRAW or CS\_VREDRAW
  mov myWndClass.lpfnWndProc,offset myWndProc
  mov myWndClass.cbClsExtra,NULL
  mov myWndClass.cbWndExtra,NULL
  push hInStance
  pop myWndClass.hInstance
  invoke LoadIcon,NULL,IDI\_WINLOGO    ;装载图标
  mov myWndClass.hIcon,eax
  invoke LoadCursor,NULL,IDC\_HAND     ;装载光标
  mov myWndClass.hCursor,eax
  invoke GetStockObject,WHITE\_BRUSH
  mov myWndClass.hbrBackground,eax       ;背景
  mov myWndClass.lpszMenuName,NULL
  mov myWndClass.lpszClassName,offset WndClassName     
  
  invoke RegisterClass,addr myWndClass          ;注册窗口
  invoke CreateWindowEx,NULL,\     ;创建窗口
                    addr WndClassName,\
                    addr WndAppName,\
                    WS\_OVERLAPPEDWINDOW, \            ;and (not WS\_MAXIMIZEBOX),\  ;如何去除一个其中一个选项
                    CW\_USEDEFAULT,CW\_USEDEFAULT,CW\_USEDEFAULT,CW\_USEDEFAULT,\
                    NULL,\
                    NULL,\
                    hInStance,\
                    NULL 
  mov hWnd,eax
  invoke ShowWindow,hWnd,SW\_SHOWNORMAL    ;显示窗口
  invoke UpdateWindow,hWnd
  
  .while TRUE        ;循环处理消息
    invoke GetMessage,addr msg,NULL,0,0
    invoke TranslateMessage,addr msg
    invoke DispatchMessage,addr msg
  .endw
  mov eax,msg.wParam
  ret

WinMain endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;消息处理部分
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
myWndProc proc hWnd:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
  
  LOCAL ps:PAINTSTRUCT
  LOCAL hDc:HDC   
  .if uMsg == WM\_DESTROY            ;窗口销毁消息           
    invoke PostQuitMessage,0
  .elseif uMsg == WM\_PAINT            ;重画消息
      invoke BeginPaint,hWnd,addr ps
      invoke GetDC,hWnd
      mov hDc,eax
      invoke TextOut,hDc,0,0,addr MsgText,(sizeof MsgText)-1
      invoke ReleaseDC,hWnd,hDc
      invoke EndPaint,hWnd,addr ps
  .elseif uMsg == WM\_KEYDOWN            ;键盘键按下消息
    .if wParam == VK\_F8      ;F8 关闭程序，相当于给程序留一个关闭的后门
      invoke ExitProcess,0
    .endif
    invoke wsprintf,addr btnkeyinfo+21,addr char,wParam      ;参考 wsprintf 的用法
    invoke MessageBox,hWnd,addr btnkeyinfo,addr MsgCaption,MB\_OK
        .elseif uMsg == WM\_LBUTTONDOWN         ;鼠标左键按下消息
          invoke MessageBox,hWnd,addr LButtonDown,addr MsgCaption,MB\_OK
  .elseif uMsg == WM\_RBUTTONDOWN         ;鼠标右键按键消息
    invoke MessageBox,hWnd,addr RButtonDown,addr MsgCaption,MB\_OK
  .elseif uMsg == WM\_CLOSE            ;关闭消息
          invoke ExitProcess,0
  .else 
          invoke DefWindowProc,hWnd,uMsg,wParam,lParam           ;默认处理
          ret
  .endif
  xor eax,eax
  ret

myWndProc endp
end start

```

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
以上代码 经 RadASM 编译通过，测试成功! (程序只有 4K) 我用 VC++ Win32 Application 做一个完全一样功能的程序，要 168K,果然汇编牛啊
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

**引用: 程序心得与总结:**

这个小程序的代码其实很容易理解，如果我已经掌握一门语言，看这个代码其实比较容易的，自己也是比较容易亲自动手写的，其实我一
开始不是很会汇编，特别在 16 位汇编学习时，觉得挺吃力，当我再次学习 Win32 汇编时，程序的设计过程和其他高级语言也是有很大相似的，就
拿这个程序来说，我是先会在 VC++ 下如何进行建立一个简单的窗口已经循环处理消息，在 VC++ 中的建立过程和 `MASM32` 中其实是一摸一样的顺序
。窗口类的赋值也是一样的，其实语言的表达稍微有点不同而已。如想知道在 VC++ 中如何建立上述的程序，如不知道的话，感兴趣的朋友可以
与我联系，我就不再贴代码了，代码的方法几乎一样。

下面谈谈这个程序的关键部分或者说需要再次说明的部分。

数据段，我们有 2 个段，`.data` 和 `.data?` 区别就是前者中的数据已经初始化，后者没有。也就是说在 `.DATA?` 中的变量都是未经初始化的
，这也就是说在程序刚启动时它们的值是什么无关紧要，只不过占有了一块内存，以后可以再利用而已。

**特别注意**：WIN32 下的实例句柄实际上是您应用程序在内存中的线性地址。

`WNDCLASSEX` 中最重要的成员莫过于 `lpfnWndProc` 了。前缀 `lpfn` 表示该成员是一个指向函数的长指针。在 Win32 中由于内存模式是 `FLAT`



由于没有 `near` 或 `far` 的区别，每一个窗口类必须有一个窗口过程。当 `Windows` 把属于特定窗口的消息发送给该窗口时，该窗口的窗口类负责处理所有的消息，如键盘消息或鼠标消息。由于窗口过程差不多智能地处理了所有的窗口消息循环，所以您只要在其中加入消息处理过程即可。

在消息循环中，一定要定义退出消息，否则当你销毁窗口后，程序并没有退出，在任务管理器中进程仍然在的。

`invoke DefWindowProc,hWnd,uMsg,wParam,lParam` 这个默认的处理是必须要的，否则你创建的窗口你是看不到的。

最后我们看看 `offset` 和 `addr` 之间的区别：

1. `addr` 不可以处理向前引用，`offset` 则能。所谓向前引用是指：标号的定义是在 `invoke` 语句之后，譬如在如下的例子：
    ```
    invoke MessageBox，NULL， addr MsgBoxText，addr MsgBoxCaption，MB_OK
    ......
    ```
   如果您是用 `addr` 而不是 `offset` 的话，那 `MASM` 就会报错。

`addr` 操作符用来把标号的地址传递给被调用的函数，它只能用在 `invoke` 语句中，譬如您不能用它来把标号的地址赋给寄存器或变量，如果想这样做则要用 `offset` 操作符。

2. `addr` 可以处理局部变量而 `offset` 则不能。局部变量只是在运行时在堆栈中分配内存空间。而 `offset` 则是在编译时由编译器解释，这显然不能用 `offset` 在运行时来分配内存空间。编译器对 `addr` 的处理是先检查处理的是全局还是局部变量，若是全局变量则把其地址放到目标文件中，这一点和 `offset` 相同，若是局部变量，就在执行 `invoke` 语句前产生如下指令序列：
    ```
    lea eax， LocalVar
    push eax
    ```
   因为 `lea` 指令能够在运行时决定标号的有效地址，所以有了上述指令序列，就可以保证 `invoke` 的正确执行了。

就谈这些吧，让我们共同学习，共同进步！

引用: 源代码下载

Download-Link:
`http://www.live-share.com/files/153022/WinMain_Asm.rar.html`

`4dd466d0e1e0cc123afbe504da2462ca`
