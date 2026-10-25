
# [分享]利用OllyDbg进行源码级调试(Win32汇编语言)

利用OllyDbg进行源码级调试(Win32汇�语言)

OllyDbg是一款能够在Windows环境下的动态调试软件，与Softice不同的是，它运行在用户模式下，且结合了动态调试与静态分析的功能。

OllyDbg通常用于反汇编调试，但事实上，它也能进行源码级调试，这对众多程序员来说是个福音。

下面我就以Win32汇编语言为例，简单介绍一下OllyDbg的源码级调试方法。

## 示例代码

代码文件列表：

`ODbgTest.Asm`

`ODbgTest.Inc`

`DlgProc\DlgProc.Asm`

`ODbgTest.Rc`

主要代码如下：

### `ODbgTest.Asm`

```asm
.386
.model flat, stdcall
option casemap :none
include ODbgTest.inc
include DlgProc\DLgProc.asm
.code
start:
  invoke GetModuleHandle,NULL
  mov    hInstance,eax
  invoke InitCommonControls
  invoke DialogBoxParam,hInstance,IDD\_DIALOG1,NULL,addr DlgProc,NULL
  invoke ExitProcess,0
end start
```

### `DlgProc\DlgProc.Asm`

```asm
.code
DlgProc proc hWin:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM
   mov eax,uMsg
   .if eax==WM\_INITDIALOG
   .elseif eax==WM\_COMMAND
      mov eax,wParam
      mov edx,eax
      shr edx,16
      and eax,0FFFFh
      .if edx==BN\_CLICKED
         .if eax==IDC\_BTN1
             invoke EndDialog,hWin,NULL
         .endif
      .endif
   .elseif eax==WM\_CLOSE
      invoke EndDialog,hWin,0
   .else
      mov eax,FALSE
      ret
   .endif
   mov        eax,TRUE
  ret
DlgProc endp
```

下面的编译链接所用的选项比较重要，就是要添加调试信息供OllyDbg调试：

```
rc ODbgTest.Rc
ml /c /coff /Cp /Zi ODbgTest.Asm
link /SUBSYSTEM:WINDOWS /DEBUG /DEBUGTYPE:CV ODbgTest.obj ODbgTest.res
```

完成后，我们便可以得到可执行程序`ODbgTest.exe`。

## OllyDbg 反汇编对比

接下来，我们用OllyDbg打开刚才得到的可执行程序`ODbgTest.exe`。

如一下就是OllyDbg中的一段反汇编代码(带调试符)：

```
0040101C >/> \55            push    ebp
0040101D  |.  8BEC          mov     ebp,esp
0040101F  |.  8B45 0C       mov     eax,[arg.uMsg]
00401022  |.  3D 10010000   cmp     eax,110                  ;  Switch (cases 10..111)
00401027  |.  75 02         jnz     short ODbgTest.0040102B
00401029  |.  EB 45         jmp     short ODbgTest.00401070  ;  Case 110 (WM\_INITDIALOG) of switch 00401022
0040102B  |>  3D 11010000   cmp     eax,111
00401030  |.  75 24         jnz     short ODbgTest.00401056
00401032  |.  8B45 10       mov     eax,[arg.wParam]         ;  Case 111 (WM\_COMMAND) of switch 00401022
00401035  |.  8BD0          mov     edx,eax
00401037  |.  C1EA 10       shr     edx,10
0040103A  |.  25 FFFF0000   and     eax,0FFFF
0040103F  |.  0BD2          or      edx,edx
00401041  |.  75 2D         jnz     short ODbgTest.00401070
00401043  |.  3D E9030000   cmp     eax,3E9
00401048  |.  75 0A         jnz     short ODbgTest.00401054
0040104A  |.  6A 00         push    0                        ; /Result = 0
0040104C  |.  FF75 08       push    [arg.hWin]               ; |hWnd
0040104F  |.  E8 8E000000   call    ODbgTest.EndDialog       ; \EndDialog
00401054  |>  EB 1A         jmp     short ODbgTest.00401070
00401056  |>  83F8 10       cmp     eax,10
00401059  |.  75 0C         jnz     short ODbgTest.00401067
0040105B  |.  6A 00         push    0                        ; /Result = 0; Case 10 (WM\_CLOSE) of switch 00401022
0040105D  |.  FF75 08       push    [arg.hWin]               ; |hWnd
00401060  |.  E8 7D000000   call    ODbgTest.EndDialog       ; \EndDialog
00401065  |.  EB 09         jmp     short ODbgTest.00401070
00401067  |>  B8 00000000   mov     eax,0                    ;  Default case of switch 00401022
0040106C  |.  C9            leave
0040106D  |.  C2 1000       retn    10
00401070  |>  B8 01000000   mov     eax,1
00401075  |.  C9            leave
00401076  \.  C2 1000       retn    10
```

我们在来看一下用普通方式编译链接的OllyDbg反汇编代码：

```
00401000  /.  55            push    ebp
00401001  |.  8BEC          mov     ebp,esp
00401003  |.  8B45 0C       mov     eax,[arg.2]
00401006  |.  3D 10010000   cmp     eax,110                          ;  Switch (cases 10..111)
0040100B  |.  75 02         jnz     short ODbgTest.0040100F
0040100D  |.  EB 45         jmp     short ODbgTest.00401054          ;  Case 110 (WM\_INITDIALOG) of switch 00401006
0040100F  |>  3D 11010000   cmp     eax,111
00401014  |.  75 24         jnz     short ODbgTest.0040103A
00401016  |.  8B45 10       mov     eax,[arg.3]                      ;  Case 111 (WM\_COMMAND) of switch 00401006
00401019  |.  8BD0          mov     edx,eax
0040101B  |.  C1EA 10       shr     edx,10
0040101E  |.  25 FFFF0000   and     eax,0FFFF
00401023  |.  0BD2          or      edx,edx
00401025  |.  75 2D         jnz     short ODbgTest.00401054
00401027  |.  3D E9030000   cmp     eax,3E9
0040102C  |.  75 0A         jnz     short ODbgTest.00401038
0040102E  |.  6A 00         push    0                                ; /Result = 0
00401030  |.  FF75 08       push    [arg.1]                          ; |hWnd
00401033  |.  E8 66000000   call    <jmp.&user32.EndDialog>          ; \EndDialog
00401038  |>  EB 1A         jmp     short ODbgTest.00401054
0040103A  |>  83F8 10       cmp     eax,10
0040103D  |.  75 0C         jnz     short ODbgTest.0040104B
0040103F  |.  6A 00         push    0                                ; /Result = 0; Case 10 (WM\_CLOSE) of switch 00401006
00401041  |.  FF75 08       push    [arg.1]                          ; |hWnd
00401044  |.  E8 55000000   call    <jmp.&user32.EndDialog>          ; \EndDialog
00401049  |.  EB 09         jmp     short ODbgTest.00401054
0040104B  |>  B8 00000000   mov     eax,0                            ;  Default case of switch 00401006
00401050  |.  C9            leave
00401051  |.  C2 1000       retn    10
00401054  |>  B8 01000000   mov     eax,1
00401059  |.  C9            leave
0040105A  \.  C2 1000       retn    10
```

我们可以对两者做一些对比，发现前者的可读性好，便于理解

例如：

`0040101F  |.  8B45 0C       mov     eax,[arg.uMsg]`

`00401003  |.  8B45 0C       mov     eax,[arg.2]`

很明显，上面一句一看就知道是干什么!

## 源码文件读取

好，下面再讲源代码的读取：

在OllyDbg主菜单中点选：

查看->源码文件，打开“源码文件”窗口，在窗口中我们可以看到如下内容：

```
模块     |  源码         | 源码路径
---------------------------------------------------------
ODbgTest |  (缺位)       | DlgProc\DLgProc.asm
ODbgTest |  ODBGTEST.ASM | E:\debug\ODbgTest\ODbgTest.asm
```

此时，我们用鼠标双击想要查看的源码文件，便可打开“源码文件窗口”

例如我们双击`ODBGTEST.ASM`，看到的结果就是这样的：

```
  1.|.386
  2.|.model flat, stdcall
  3.|option casemap :none
  4.|include ODbgTest.inc
  5.|include DlgProc\DLgProc.asm
  6.|.code
  7.|start:
> 8.|  invoke GetModuleHandle,NULL
> 9.|  mov    hInstance,eax
>10.|  invoke InitCommonControls
>11.|  invoke DialogBoxParam,hInstance,IDD\_DIALOG1,NULL,addr DlgProc,NULL
>12.|  invoke ExitProcess,0
13.|end start
```

是不是与我们编写的源码一摸一样？

但这里有个问题，`DLgProc.asm`文件被显示为“ (缺位)”，且当我们双击它时，根本就看不到源码。

做个对比发现，`ODbgTest.asm`的“ 源码路径”是一个完整路径，而`DLgProc.asm`的不是。

那么，再看一下我们自己编写的源代码：

```asm
ODbgTest.Asm:
.386
.model flat, stdcall
option casemap :none
include ODbgTest.inc
include DlgProc\DLgProc.asm    ; <--这里，改成完整路径
.code
start:
....
```

将第5行改成`include E:\debug\ODbgTest\DlgProc\DlgProc.Asm`

然后再重新编译链接，再用OllyDbg打开“源码文件”窗口看到如下：

```
模块     |  源码         | 源码路径
----------------------------------------------------------------
ODbgTest |  DLgProc.asm  | E:\debug\ODbgTest\DlgProc\DlgProc.Asm
ODbgTest |  ODBGTEST.ASM | E:\debug\ODbgTest\ODbgTest.asm
```

怎么样，正常了吧。再双击`DLgProc.asm`文件，便能看到它的源码了。
