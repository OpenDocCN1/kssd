
```asm
.386
.model flat,stdcall
option casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; Include 文件定义
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include          windows.inc
include          gdi32.inc
includelib       gdi32.lib
include          user32.inc
includelib       user32.lib
include          kernel32.inc
includelib       kernel32.lib

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; 数据段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.data?

hInstance        dd         ?

hWinMain         dd         ?

.const

szClassName      db    'MyClass',0

szCaptionMain    db    'My first Window !',0

szText           db    'Win32 Assembly, Simple and powerful !',0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; 代码段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; 窗口过程
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
_ProcWinMain     proc  uses ebx edi esi,hWnd,uMsg,wParam,lParam

        local @stPs:PAINTSTRUCT

        local @stRect:RECT

        local @hDc

        mov   eax,uMsg

;*********************************************************************
        .if      eax ==    WM_PAINT

                invoke    BeginPaint,hWnd,addr @stPs

                mov       @hDc,eax

                invoke    GetClientRect,hWnd,addr @stRect

                invoke    DrawText,@hDc,addr szText,-1,\
                           addr @stRect,\
                           DT_SINGLELINE or DT_CENTER or DT_VCENTER

                invoke    EndPaint,hWnd,addr @stPs

;*********************************************************************
        .elseif  eax ==    WM_CLOSE

                invoke    DestroyWindow,hWinMain

                invoke    PostQuitMessage,NULL

;*********************************************************************
        .else
                invoke    DefWindowProc,hWnd,uMsg,wParam,lParam

                        ret

                .endif
;*********************************************************************
        xor      eax,eax

        ret
_ProcWinMain     endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
_WinMain         proc
        local     @stWndClass:WNDCLASSEX

        local     @stMsg:MSG

        invoke   GetModuleHandle,NULL

        mov              hInstance,eax

        invoke   RtlZeroMemory,addr @stWndClass,sizeof @stWndClass

;*********************************************************************
; 注册窗口类
;*********************************************************************
        invoke   LoadCursor,0,IDC_ARROW

        mov      @stWndClass.hCursor,eax

        push     hInstance

        pop      @stWndClass.hInstance

        mov      @stWndClass.cbSize,sizeof WNDCLASSEX

        mov      @stWndClass.style,CS_HREDRAW or CS_VREDRAW

        mov      @stWndClass.lpfnWndProc,offset _ProcWinMain

        mov      @stWndClass.hbrBackground,COLOR_WINDOW + 1

        mov      @stWndClass.lpszClassName,offset szClassName

        invoke   RegisterClassEx,addr @stWndClass

;*********************************************************************
; 建立并显示窗口
;*********************************************************************
        invoke   CreateWindowEx,WS_EX_CLIENTEDGE,\
                 offset szClassName,offset szCaptionMain,\
                 WS_OVERLAPPEDWINDOW,\
                 100,100,600,400,\
                 NULL,NULL,hInstance,NULL

        mov      hWinMain,eax

        invoke   ShowWindow,hWinMain,SW_SHOWNORMAL

        invoke   UpdateWindow,hWinMain

;*********************************************************************
; 消息循环
;*********************************************************************
        .while   TRUE

                invoke    GetMessage,addr @stMsg,NULL,0,0

                .break    .if eax  == 0

                invoke    TranslateMessage,addr @stMsg

                invoke    DispatchMessage,addr @stMsg

        .endw

        ret
_WinMain         endp
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
start:
        call     _WinMain

        invoke   ExitProcess,NULL
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
        end      start
```

### 第四章 第一个窗口程序
### 4.1 开始了解窗口(3)

让我们打开一个 DOS 窗口，切换到 `FistWindow` 所在的目录，运行环境设置的批处理文件 `var.bat`，再键入 `nmake` 编译出 `FirstWindow.exe`，这个程序只有 2 560 字节，运行后窗口出来了，如图 4.3 所示。对于这个窗口，用户可以拖动边框去改变大小、按标题栏上的按钮来最大化和最小化，当光标移到边框的时候，会自动变成双箭头……总之，这个窗口包括了一个典型窗口的所有特征。(当用户去改变这个窗口的时候，系统自动调用缺省消息处理函数来处理相应的事件。在这个程序中我们只对 `WM_PAINT`、`WM_CLOSE` 这两个消息感兴趣而已,故我们为处理它们而添加了相应的处理消息的代码进去。)

![](img/08c18fea8c3aecf1497d68e90571a761.jpg)

图 4.3 `FirstWindow` 的运行结果

接下来开始分析源代码，看了这三页多的源代码，第一个感觉是什么？是不是想撤退了？笔者刚开始编 Win32 程序的时候就是这种感觉，可能 90％的人有同样的感觉，别急，过了这一关，Win32 汇编的入门就成功了一半，所以千万要挺住！有个振奋人心的消息是，这个程序是大部分窗口程序的模板，以后要写一个新的程序，把它拷贝过来再往中间添砖加瓦就是了，功夫一点都不白费。(三页代码确实挺多的，容易让人产生恐惧感。如老罗所说，再难再苦也要咬着牙挺过来，很快就会好的。不过我对这个有疑问，如果实在是看不下去的话，我宁可把它放一边先，去学点自己更容易掌握和吸收的技术，等时机成熟了再来看。因为这本书如果让大多数人实在无法看下去的话，那还不如去找另一本更适合自己看的书去。)

先静下心来分析一下程序的结构，还看得懂，很好！其实源程序的结构在第三章里已经了解过了，首先是注释……模式定义……`include`…… `.data` 数据段，都没有问题，这些已经占去了近 40 行了，好了，终于是关键的代码段了，统计一下，只剩 80 行代码了。(消息处理机制下的程序确实是有个结构的，但并不是说它就是很死板的，我们大可不必把所有的代码放在窗口消息处理函数里，如果代码堆放得太多的话，程序会显得极度不平衡的。)

分析一下程序的结构，发现入口是 `start`，然后执行了一个`_WinMain` 子程序，完成后就是程序退出的函数 `ExitProcess`，再看`_WinMain` 的结构，前面是顺序下来的几个 API：

```
GetModuleHandle → RtlZeroMemory → LoadCursor → RegisterClassEx

→ CreateWindowEx → ShowWindow → UpdateWindow
```

从名称上就能看出它们的用途，很明显，窗口是在 `CreateWindowEx` 处建立的，`ShowWindow` 则是把窗口显示在屏幕上，这些代码是窗口的建立过程。(确实，如众所周知的一样，API 的名字已经很好的描述了它的功能了。因为我们很有必要仔细研究一下 API 的名字和它的功能的关系，这样就容易理解一点，而不会对 API 产生恐惧感了。即使不能完全理解，但看到名字至少也知道点意思。)

接下来，就是一个由 3 个 API 组成的循环了：

```
GetMessage → TranslateMessage → DispatchMessage
```

很明显，这是和消息有关的循环，因为 API 名称中都带有 `Message` 字样，如果退出这个循环，程序也就结束了，这个循环叫做消息循环。设置`_WinMain` 子程序并不是必须的，可以把`_WinMain` 的所有代码放到主程序中，没有任何影响，之所以这样只是为了将这里使用的变量定义成局部变量，这样可以方便移植。(这里表达了好几个重要的信息：一是处理消息的三个极其重要的 API，它们名字里都带有 `Message` 字样；二是由这几个 API 构成的循环为消息循环，几乎没有哪个 WINDOWS 程序里少了这个循环的，这非常好的表明了这个循环的重要性；三是消息处理函数里的所有代码可以放到主函数内部，但把它做成一个函数的好处是方便移植。如果真将消息处理函数里的所有代码放到主程序内部，那我们在注册窗口类时就没必要将消息处理函数的地址传递到窗口类成员中去了，我这样做了，结果运行程序没有显示窗口，另外，我想将消息处理函数里的代码移到主程序中去，发现这样做很不妥。还是保持原状的好。)

看了程序的流程，似乎没有什么地方涉及窗口的行为，如改变大小和移动位置的处理等。再看源程序，除了`_WinMain`，还有一个子程序`_ProcWinMain`，但除了在 `WNDCLASSEX` 结构的赋值中提到过它，好像就没有什么地方要用到这个子程序，起码在自己编写的源代码中没有任何一个地方调用过它。(表面上我们一次也没有调用`_ProcWinMain` 这个函数,实际上这个函数一直在被调用，只要程序接受到了消息，它就会被调用，只是这个函数不是由我们调用而是由程序自动调用而已。这里面涉及了一个思想，那就是什么函数来消息用户触发的事件所发出的消息的思想。)
