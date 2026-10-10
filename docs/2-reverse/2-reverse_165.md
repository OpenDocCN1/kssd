

# Pediy 之给 OllyICE 添加全屏功能

> 原文：[`www.pediy.com/kssd/pediy10/73562.html`](https://www.pediy.com/kssd/pediy10/73562.html)

看了《加密与解密》第十九章觉得手痒所以做了这个。废话少说，直奔主题。  
这里采用的是增加接口的方法，一共分为 3 步：  
1.编写 DLL  
2.添加导入函数  
3.修改和添加代码  
  
1.编写 DLL  
实现全屏的代码如下：  

代码:

```
                .386
                .model flat, stdcall
                option casemap:none
include         windows.inc
include         user32.inc
include         gdi32.inc
include         kernel32.inc
includelib      user32.lib
includelib      gdi32.lib
includelib      kernel32.lib

                .data
szErrlWinMode   db    "set window window mode faild", 0
szErrlFullScr   db    "set window full screen mode faild", 0
szCaption       db    "Info",0
bIsFullMode     dword FALSE
stOriginPos     WINDOWPLACEMENT <0>
ESC\_HOTKEY      equ   1
F11\_HOTKEY      equ   2

                .code
DllEntry    proc    \_hInstance, \_dwReason, \_dwReserved
    mov     eax, TRUE
    ret
DllEntry    endp

SetFullScr  proc   \_hWnd
    local   @hDC        
    local   @dwHorzres
    local   @dwVertres
    
    invoke  GetWindowPlacement, \_hWnd, addr stOriginPos
    invoke  GetWindowLong, \_hWnd, GWL\_STYLE

    xor     eax, WS\_CAPTION
    invoke  SetWindowLong, \_hWnd, GWL\_STYLE, eax        ;去掉标题栏
    
    invoke  GetDC, \_hWnd
    mov     @hDC, eax
    invoke  GetDeviceCaps, @hDC, HORZRES
    mov     @dwHorzres, eax
    invoke  GetDeviceCaps, @hDC, VERTRES
    mov     @dwVertres, eax

    xor     eax, eax
    invoke  SetWindowPos, \_hWnd, HWND\_TOPMOST, eax, eax, @dwHorzres, @dwVertres, SWP\_SHOWWINDOW ;全屏显示
    ret
SetFullScr  endp

SetWinMode  proc \_hWnd
    xor     eax, eax
    invoke  SetWindowPos, \_hWnd, HWND\_NOTOPMOST, eax, eax, eax, eax, SWP\_NOSIZE ;添加此两行，修正 bug

    invoke  GetWindowLong, \_hWnd, GWL\_STYLE
    or      eax, WS\_CAPTION
    invoke  SetWindowLong, \_hWnd, GWL\_STYLE, eax  ;恢复标题栏
    invoke  ShowWindow, \_hWnd, SW\_HIDE
    invoke  SetWindowPlacement, \_hWnd, addr stOriginPos
    ret
SetWinMode ENDP


SwitchMode proc \_hWnd
    .if bIsFullMode == TRUE
        invoke SetWinMode, \_hWnd
        .IF eax==0
            invoke MessageBox, \_hWnd, addr szErrlWinMode, addr szCaption, MB\_ICONERROR
            ret
        .ENDIF
        mov        bIsFullMode, FALSE
    .else
        invoke SetFullScr, \_hWnd
        .IF eax==0
            invoke MessageBox, \_hWnd, addr szErrlFullScr, addr szCaption, MB\_ICONERROR
            ret
        .ENDIF
        mov bIsFullMode, TRUE
    .endif
    ret
SwitchMode endp

WndProc proc C dwReserved, hWnd, uMsg, wParam, lParam
    mov    eax, uMsg    
    .if         eax == WM\_HOTKEY
        .if     wParam == ESC\_HOTKEY && bIsFullMode == TRUE
                invoke SwitchMode, hWnd
        .elseif wParam == F11\_HOTKEY && bIsFullMode == FALSE
                invoke SwitchMode, hWnd
        .endif
    .elseif     eax == WM\_CREATE
        invoke  RegisterHotKey, hWnd, F11\_HOTKEY, 0, VK\_F11
        invoke  RegisterHotKey, hWnd, ESC\_HOTKEY, 0, VK\_ESCAPE
    .elseif     eax == WM\_DESTROY
        invoke  UnregisterHotKey, hWnd, F11\_HOTKEY
        invoke  UnregisterHotKey, hWnd, ESC\_HOTKEY
    .endif
    ret
WndProc endp

end DllEntry

```

汇编链接后，就生成了需要的 DLL。  
编写的 DLL 只导出了一个函数 WndProc，需要特别注意的是  
WndProc proc C dwReserved, hWnd, uMsg, wParam, lParam，对了就是指定函数的调用类型为 C，Tnnd 当初我就是栽在这儿了  
  
2.添加导入函数  
用 LordPE 打开 OllyICE，点击“Directories”->在弹出的 DirectoryTable 窗口中的 import 项后面的“...”按钮上单击->在弹出的 ImportTable 窗口中单击右键->“add import”->在弹出的 AddImport 窗口中输入上面创建的 DLL 文件名（这里是 od.dll）->API 对应的文本框填写此 DLL 到处的函数（这里是 WndProc）->单击“+”按钮，再单击 OK 搞定。  
这样我们就添加好了输入函数。我们需要记住的唯一一个数据是 ThunkRVA（这里是 1AA00D），在 400000（基址）+ 1AA00D = 5AA00D 该处，程序加载完成时这里的双字就是 DLL 中的 WndProc 函数地址。  
最好是将 DLL 文件 Copy 至 OllyICE 所在目录，否则 LordPE 会有“API not found！”的提示。  
  
3.修改和添加代码  
我们应当首先找到主窗体的窗口过程。我这里是对在注册窗口类时断下，这样就可以通过窗口类得到窗口过程了。按 CTRL + N，在弹出的函数名窗口中找到 user32 的导出函数 RegisterClassA，在该行单击回车就到了 RegisterClassA 的函数参考窗口，可以看到一共是有 7 条记录，按 F2 全部设断。回到 CPU 窗口，按 F9 执行就断在  
代码:

```
00435CDE   .  E8 69980700   call    <jmp.&USER32.RegisterClassA>     ; \\RegisterClassA

```

往上面一点就找到了窗口过程的地址，  
代码:

```
00435C6A   .  C785 94F6FFFF>mov     dword ptr \[ebp-96C\], OI.004323D4 ;  入口地址

```

就是 4323D4 啦。我们还应该找块地放我们自己的代码，我们选择 4AF7BB。所以我们转到 4323D4 处，看到开头的 3 条指令是：  
代码:

```
004323D4  /$  55            push    ebp
004323D5  |.  8BEC          mov     ebp, esp
004323D7  |.  81C4 04F0FFFF add     esp, -0FFC

```

我们得改成这样：  
代码:

```
004323D4     /E9 E2D30700   jmp     OllyICE.004AF7BB
004323D9     |90            nop
004323DA     |90            nop
004323DB     |90            nop
004323DC     |90            nop

```

在 004323D4 行按回车就到了  
代码:

```
004AF7BB      00            db      00
004AF7BC      00            db      00
004AF7BD      00            db      00
004AF7BE      00            db      00
004AF7BF      00            db      00
004AF7C0      00            db      00

```

添加代码如下：  
代码:

```
004AF7BB      FF15 0DA05A00 call    dword ptr \[<&od.WndProc>\]        ;  od.WndProc
004AF7C1      55            push    ebp
004AF7C2      8BEC          mov     ebp, esp
004AF7C4      81C4 04F0FFFF add     esp, -0FFC
004AF7CA    ^ E9 0E2CF8FF   jmp     OllyICE.004323DD

```

这样就整个流程就差不多了，最后在反汇编窗口中点击右键->“复制到可执行文件”->“所有修改”。如果有弹出“在可执行文件中无法定位数据”的对话框，那么你的把最后添加的代码换个地点，并作相应的修改。  
  
这样，我们就可以按 F11 键实现全屏，而按 Esc 键退出全屏。哈哈,Pediy 够简单的吧。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  
附件里是修改好的 OllyICE 和 od.asm,od.def.od.dll，请注意你的 OllyICE 版本是否和我的一样。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || OllyICE.rar  
 |