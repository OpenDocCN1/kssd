
# 逆向 `hwnd_pwd`

> 原文：[`www.pediy.com/kssd/pediy06/pediy6841.htm`](https://www.pediy.com/kssd/pediy06/pediy6841.htm)

一款工具的逆向分析与再实现

作者：`nbw` [www.vxer.com](http://www.vxer.com)

目的：逆向分析一款窗口句柄察看工具（`dRag0nMa` 大侠写的 `hwnd_pwd`）并重新写出来

分析工具：`IDA`，`fly_OD`

编写语言：`Masm32`（原软件估计是 `VC`）

备注：刚看了 `firstrose` 兄弟写的逆向文章，自己也想来练练手。未曾征得原作者的同意就发出笔记，请多原谅！

感谢悠长假期的帮助。

原软件用 `upx` 加壳，脱掉。

用 `HackRes` 查看一下资源，可以获得资源文件：

```
101 DIALOGEX 0, 0, 181, 53

STYLE DS_MODALFRAME | DS_CENTER | WS_MINIMIZEBOX | WS_POPUP | WS_CAPTION | WS_SYSMENU

CAPTION "WinClass&HWND+viEw\*pwd"

LANGUAGE LANG_ENGLISH, SUBLANG_ENGLISH_US

FONT 8, "Times New Roman"

{
   CONTROL "", 1000, EDIT, ES_LEFT | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_GROUP | WS_TABSTOP, 41, 13, 122, 12

   CONTROL "", 1005, EDIT, ES_LEFT | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_TABSTOP, 40, 31, 91, 12

   CONTROL "Win.Class:", 1037, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 7, 13, 35, 8 , 0x00000020

   CONTROL "HWND:", -1, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 8, 31, 29, 9

   CONTROL "", -1, BUTTON, BS_GROUPBOX | WS_CHILD | WS_VISIBLE, 2, 0, 174, 47

   CONTROL "cOde:dRag0nMa", -1, STATIC, SS_RIGHT | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_GROUP | WS_TABSTOP, 113, 41, 58, 8
}
```

需要注意的是：

```
CAPTION "WinClass&HWND+viEw\*\*pwd"
```

这里在 `asm` 中编译不通过，直接去掉 `\\` 就可以，改为：

```
CAPTION "WinClass&HWND+viEw\*pwd"
```

但是它这里的 `LANGUAGE` 和 `FONT` 我这边不能用（估计我的机器太烂，装配不全），所以最后看到的界面比原来的软件要大。

代码初始：

```
mBC:00401048               push    1
mBC:0040104A               call    ds:GetCommandLineA
mBC:00401050               push    eax
mBC:00401051               push    0
mBC:00401053               push    0               ; lpModuleName
mBC:00401055               call    ds:GetModuleHandleA
mBC:0040105B               push    eax
mBC:0040105C               call    sub_401213
mBC:00401061               push    eax             ; uExitCode
mBC:00401062               call    ds:ExitProcess
mBC:00401068               retn
```

其中，`call sub_401213` 为：

```
mov     eax, [esp+hInstance]
push    0               ; dwInitParam
push    offset DialogFunc ; lpDialogFunc
push    0               ; hWndParent
push    65h             ; lpTemplateName
push    eax             ; hInstance
mov     ds:dword_401344, eax
call    ds:DialogBoxParamA ; Create a modal dialog box from a
xor     eax, eax
```

这里需要注意的是：

`push 65h`
这个地方为主窗口资源 ID

`mov ds:dword_401344, eax`
由于 `eax` 为进程句柄，因此，我们用 `mov hInstance, eax` 代替

`xor eax, eax`
故，下面的 `call ExitProcess` 参数为 `NULL`

理解了 `call sub_401213` 的含义，故此，把该函数去掉，只保留该函数的主体。

进入主消息处理瞅瞅：

```
DialogFunc  proc near               ; DATA XREF: sub_401213+6o
mBC:004010DE hDlg            = dword ptr  8
mBC:004010DE arg_4           = dword ptr  0Ch
mBC:004010DE                 push    ebp
mBC:004010DF                 mov     ebp, esp
mBC:004010E1                 mov     eax, [ebp+arg_4] ; wMsg
```

其中 `ebp + hDlg` 相当于我们的 `hWnd`，`ebp+arg_4` 相当于我们的 `wMsg`。代码开头就是编译器的经典 `call` 处理，我们作汇编的，不管这个。

```
mBC:004010E4                 sub     eax, 0Fh
mBC:004010E7                 jz      loc_40120D
```

这 2 句是错误消息处理，如果消息是 `0Fh`，那么直接返回。代码中应该做如下处理：

```
sub  eax, 0Fh
cmp  eax, 0
jne  @F
xor  eax, eax
ret
@@:        ;normal Message
```

那么 `0Fh` 究竟是什么消息呢？一查查了一大堆，也懒得去猜。因此我也不晓得。

继续向下看：

```
mBC:004010ED                 dec     eax
mBC:004010EE                 jz      loc_4011F7
```

如果传来的消息为 `0F + 1`（注意上面已经 `sub eax, 0Fh`），那么：

```
mBC:004011F7                 push    1               ; uIDEvent
mBC:004011F9                 push    [ebp+hDlg]     ; hWnd
mBC:004011FC                 call    ds:KillTimer
mBC:00401202                 push    0               ; nResult
mBC:00401204                 push    [ebp+hDlg]     ; hDlg
mBC:00401207                 call    ds:EndDialog
mBC:0040120D                 xor     eax, eax
mBC:0040120F                 pop     ebp
mBC:00401210                 retn    10h
```

就是：

```
push 1
push hWnd
call KillTimer
push 0
push hWnd
call EndDialog
xor  eax, eax
```

很明显是 close 事件。那么处理的消息是不是 `WM_CLOSE` 呢？可以认为是的，因为 `(0F + 1) equ 10h equ WM_CLOSE`。

因此，这里可以翻译成：

```
.if  wMsg == WM_CLOSE
   invoke  KillTimer,hWnd,1
   invoke  EndDialog,hWnd,NULL
   xor eax, eax
.endif
```

再往下看下一条消息吧：

```
push    0               ; lpTimerFunc
push    12Ch            ; uElapse
push    1               ; nIDEvent
push    hWnd
call    SetTimer
push    offset Rect     ; lpRect
push    hWnd
call    GetWindowRect
push    41h             ; uFlags
push    Rect.bottom  ; cy
push    Rect.right   ; cx
push    Rect.top     ; Y
push    Rect.left    ; X
push    0FFFFFFFFh     ; hWndInsertAfter
push    hWnd
call    SetWindowPos
push    1
pop     eax
jmp     short loc_40120F
```

short `loc_40120F`：

```
pop   ebp
ret
```

`SetTimer` 设定一个时间，300 毫秒监测一次，`lpTimerFunc` 为 0，也就是这个 timer 专门供主进程用。

`GetWindowRect` 获取主窗口位置。`Rect` 结构定义要注意以下。具体参考代码。

`SetWindowPos` 设定窗体显示属性，比如说总置顶显示。

最后是 `mov eax, 1`，然后 `ret`

这个消息数值为 `110h`，参考以下功能，可以判定为 `WM_INITDIALOG`。

再看下一条消息处理：

```
mBC:004011F2                 push    1
mBC:004011F4                 pop     eax
mBC:004011F5                 jmp     short loc_40120F
```

完成 `mov eax, 1` 然后退出。

该消息值为 `111h`，判定为 `WM_COMMAND`。因为资源定义中使用了 menu，因此这里怀疑是否为用来处理 menu 消息的部分。但是由于没用到，所以程序中我删除了这个。

继续看下一条：

消息检测为：

```
mBC:00401106                 dec     eax
mBC:00401107                 dec     eax
mBC:00401108                 jz      short loc_401132
```

消息数值为：`111h + 2 = 113h`，查一下文档，就 2 个消息为该值，毋庸置疑，该消息是 `WM_TIMER`，总算到了核心部分。

消息处理如下：

```
mBC:00401132 loc_401132:                        ; CODE XREF: DialogFunc+2Aj
mBC:00401132                 push    esi
mBC:00401133                 push    edi
mBC:00401134                 push    offset Point    ; lpPoint
mBC:00401139                 call    ds:GetCursorPos
mBC:0040113F                 push    ds:Point.y
mBC:00401145                 push    ds:Point.x      ; Point
mBC:0040114B                 call    ds:WindowFromPoint ; Get handle of the window that
mBC:0040114B                                            ; contains the specified poin
mBC:00401151                 push    eax
mBC:00401152                 push    50h          ;注意这里是显示数据的 label 的 ID
mBC:00401154                 mov     ds:hWnd, eax    ;保存鼠标所在的窗口句柄，用于下一步调用
mBC:00401159                 call    sub_401069
mBC:0040115E                 pop     ecx
mBC:0040115F                 mov     esi, ds:SetDlgItemTextA
mBC:00401165                 pop     ecx
mBC:00401166                 push    offset String   ; lpString
mBC:0040116B                 push    3EDh            ; nIDDlgItem
mBC:00401170                 push    [ebp+hDlg]      ; hDlg
mBC:00401173                 call    esi ; SetDlgItemTextA
mBC:00401175                 mov     edi, offset ClassName
mBC:0040117A                 push    10h             ; nMaxCount
mBC:0040117C                 push    edi             ; lpClassName
mBC:0040117D                 push    ds:hWnd         ; hWnd
mBC:00401183                 call    ds:GetClassNameA
mBC:00401189                 push    edi             ; lpString
mBC:0040118A                 push    3E8h            ;注意这里是显示数据的 label 的 ID
mBC:0040118F                 push    [ebp+hDlg]      ; hDlg
mBC:00401192                 call    esi ; SetDlgItemTextA
mBC:00401194                 push    0               ; lParam
mBC:00401196                 push    0               ; wParam
mBC:00401198                 push    0CCh            ; Msg
mBC:0040119D                 push    ds:hWnd         ; hWnd
```



`mBC:004011A3 call ds:PostMessageA`

首先，通过`GetCursorPos`获取鼠标所在位置，然后通过`WindowFromPoint`函数获取该位置处窗口句柄，利用`SetDlgItemTextA`显示出来。

再用`GetClassNameA`获取该窗口类，显示出来。

其中有函数：

```
push    eax
push    50h
mov     hWnd, eax
call    sub_401069
```

去瞅瞅：

```
mBC:00401070             mov     esi, [ebp+arg_4]        ;取获得的窗口句柄
mBC:00401073             xor     ecx, ecx
mBC:00401075             test    esi, esi
mBC:00401077             jz      short loc_4010A7        ;如果没取到，便退出
mBC:00401079             push    edi
mBC:0040107A

;********************************************************************

;这个小循环，通过除以 10h，依次取出来窗口句柄的每一位（肯定小于 0x10 了）。
;如果小于 9，就加上 30h，转换成真正的 9，否则，加上 37h，转换成字母。
;不过转换过来的顺序颠倒过来了
;比如 B0638 转换成：
; 38 33 36 30 42 00

mBC:0040107A             mov     eax, esi
mBC:0040107C             push    10h
mBC:0040107E             cdq
mBC:0040107F             pop     edi
mBC:00401080             idiv    edi
mBC:00401082             lea     eax, [ebp+ecx+var_C]
mBC:00401086             cmp     dl, 9
mBC:00401089             mov     [eax], dl
mBC:0040108B             jle     short loc_401092
mBC:0040108D             add     dl, 37h
mBC:00401090             jmp     short loc_401095
mBC:00401092

mBC:00401092 loc_401092:                         ; CODE XREF: sub_401069+22j
mBC:00401092             add     dl, 30h
mBC:00401095

mBC:00401095 loc_401095:                         ; CODE XREF: sub_401069+27j
mBC:00401095             mov     [eax], dl
mBC:00401097             mov     eax, esi
mBC:00401099             push    10h
mBC:0040109B             cdq
mBC:0040109C             pop     esi
mBC:0040109D             idiv    esi
mBC:0040109F             inc     ecx
mBC:004010A0             mov     esi, eax
mBC:004010A2             test    esi, esi
mBC:004010A4             jnz     short loc_40107A

;********************************************************************

mBC:004010AF             mov     dl, byte ptr [ebp+ecx+var_C]    ;把转换后的数据移动到输出缓冲区
mBC:004010B3             mov     ds:String[eax], dl
mBC:004010B9             inc     eax
mBC:004010BA             dec     ecx
mBC:004010BB             jns     short loc_4010AF

;在窗口句柄输出缓冲区最后添加“(H)”

mBC:004010BD             mov     ds:String[eax], 28h        ;28h 为“(”的 ASCII 码
mBC:004010C4             mov     ds:byte_401239[eax], 48h    ;48h 为“H”的 ASCII 码
mBC:004010CB             mov     ds:byte_40123A[eax], 29h    ;29h 为“)”的 ASCII 码
mBC:004010D2             and     ds:byte_40123B[eax], 0
```

现在看来这个函数 `sub_401069` 就是把获取的窗口转换成可以让用户看的数据，然后在后面添加“(H)”，表示是 16 进制。

我翻译一下：

```
    xor  ecx, ecx
    mov  esi, offset szString1      
@@:              ;这个小循环用来把句柄数值转换成字符串，存放在 szString1，不过存放的数值是倒的
    test  eax, eax
    jz    @lExit
    xor  edx, edx
    mov  edi, 10h
    idiv  edi
    .if    dl >  9
      add  dl, 37h
    .else
      add  dl, 30h
    .endif
    mov  byte ptr [esi], dl
    inc  esi
    inc  ecx
    test  eax, eax
    jnz  @B
  @lExit:
    mov  edi, offset szString2
    dec  esi
@@:            ;把倒过来的字符串再修正回来，放在 szString2
    mov  al, BYTE ptr [esi]
    mov  BYTE ptr [edi], al
    dec  ecx
    dec  esi
    inc  edi
    test  ecx, ecx
    jnz  @B
    mov     BYTE ptr [edi ], 28h     ;填写句柄字符串最后的"(H)"
    mov     BYTE ptr [edi+1], 48h
    mov     BYTE ptr [edi +2 ], 29h
    mov     BYTE ptr [edi +3 ], 00
```

写得比较垃圾，凑合用吧。

看下一个消息：

```
mBC:0040110A             sub     eax, 0EEh
mBC:0040110F             jnz     loc_40120D
```

`WM_TIMER` = 113h + 0EEh，估计是 `WM_LBUTTONDOWN`。

而 `loc_40120D`：

```
xor     eax, eax
ret
```

其实就是消息 case 的返回。

由于这里是 `jnz`，就是说如果不是 `WM_LBUTTONDOWN`，就返回。

结合上面的代码，`WM_LBUTTONDOWN` 是最后一条需要专门处理的消息。处理如下：

```
mBC:00401115             call    ds:ReleaseCapture
mBC:0040111B             push    0               ; lParam
mBC:0040111D             push    2               ; wParam
mBC:0040111F             push    0A1h            ; Msg
mBC:00401124             push    [ebp+hDlg]       ; hWnd
mBC:00401127             call    ds:SendMessageA
```

解释一下，就是当本窗口发生鼠标事件时候，不接受鼠标的位置（减轻程序的处理负担）。

具体翻译参考以下代码就行。

## 总结

总体来说，程序流程规整，便于分析。但是也存在一些问题，比如：

1. 他这里每隔 300 毫秒，利用鼠标 API 和句柄 API 来检测窗口句柄和类，虽然程序实现比较容易，但是效率不高，一般来说应该用 hook 拦截鼠标消息。
2. 另外，多处理了 `WM_COMMAND` 消息（或许有用）。
3. 对字符串的处理（`:0040107A`），没有处理好窗口句柄，因为存在本进程句柄和获取的窗口句柄，最好多添加一个 `hWnd` 变量，用起来也方便。

我这里代码写得比较烂，时间仓促，自己也太垃圾，不足之处多包涵。

## 附程序代码

### 资源文件

`"Main.rc"`文件：

```
#define DLG_MAIN 101

#include "resource.h"

DLG_MAIN DIALOGEX 0, 0, 181, 53
CAPTION "WinClass&HWND+viEw*pwd"
STYLE DS_MODALFRAME | DS_CENTER | WS_MINIMIZEBOX | WS_POPUP | WS_CAPTION | WS_SYSMENU
BEGIN
   CONTROL "", 1000, EDIT, ES_LEFT | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_GROUP | WS_TABSTOP, 41, 13, 122, 12   
   CONTROL "", 1005, EDIT, ES_LEFT | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_TABSTOP, 40, 31, 91, 12  
   CONTROL "Win.Class:", 1037, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 7, 13, 35, 8 , 0x00000020
   CONTROL "HWND:", -1, STATIC, SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 8, 31, 29, 9  
   CONTROL "", -1, BUTTON, BS_GROUPBOX | WS_CHILD | WS_VISIBLE, 2, 0, 174, 47  
   CONTROL "cOde:dRag0nMa", -1, STATIC, SS_RIGHT | WS_CHILD | WS_VISIBLE | WS_DISABLED | WS_GROUP | WS_TABSTOP, 113, 41, 58, 8  
END

IDM_MAIN MENUEX
BEGIN
    MENUITEM "",,0x00000800
END
```

### `Main.asm`文件

```
.386
.model flat, stdcall
option casemap :none

;---------------------------------------------------------------------------
include    nbwFun.inc
_ProcDlgMain  PROTO :DWORD,:DWORD,:DWORD,:DWORD

;---------------------------------------------------------------------------

.data
  lRect    RECT <>  
  lPoint  POINT <>
  szClassName  db  16 dup(?)

.data?
  lhWnd      dd  ?
  szString1  db  16 dup(?)
  szString2  db  16 dup(?)

.code

start:
  push    1
  call    GetCommandLineA
  push    eax
  push    0
  push    0               ; lpModuleName
  call    GetModuleHandleA
  push    0               ; dwInitParam
  push    offset _ProcDlgMain ; lpDialogFunc
  push    0               ; hWndParent
  push    65h        ; lpTemplateName
  push    eax             ; hInstance
  call    DialogBoxParam  ; Create a modal dialog box
  push    NULL
  call    ExitProcess

;---------------------------------------------------------------------------

_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam
  mov  eax,wMsg
  .if  eax == WM_CLOSE
    invoke  EndDialog,hWnd,NULL
  .elseif  eax == WM_INITDIALOG
      push    0               ; lpTimerFunc
      push    12Ch            ; uElapse
      push    1               ; nIDEvent
      push    hWnd
      call    SetTimer
      push    offset lRect     ; lpRect
      push    hWnd
      call    GetWindowRect
      push    41h             ; uFlags
      push    lRect.bottom  ; cy
      push    lRect.right   ; cx
      push    lRect.top     ; Y
      push    lRect.left    ; X
      push    0FFFFFFFFh    ; hWndInsertAfter = HWND_TOPMOST
      push    hWnd
      call    SetWindowPos
      push    1
      pop     eax
      ret

;  .elseif  eax == WM_COMMAND    ;原来软件中对此进行处理。我这里注释掉了
;      mov  eax, TRUE
;      ret
  .elseif  eax == WM_TIMER
      push    offset lPoint    ; lpPoint
      call    GetCursorPos
      push    lPoint.y
      push    lPoint.x     ; Point
```



```asm
call    WindowFromPoint ; Get handle of the window that  
push    eax       ;获取的窗口句柄  
pop     lhWnd       ;保存，供下一步应用  
xor   ecx, ecx  
mov   esi, offset szString1     
  
@@:             ;这个小循环用来把句柄数值转换成字符串，存放在 szString1，不过存放的数值是倒的  
  test  eax, eax  
  jz    @lExit  
  xor   edx, edx  
  mov   edi, 10h  
  idiv  edi  
  .if   dl >  9  
    add   dl, 37h  
  .else  
    add   dl, 30h  
  .endif  
  mov   byte ptr [esi], dl  
  inc   esi  
  inc   ecx  
  test  eax, eax  
  jnz   @B  
  
@lExit:  
  mov   edi, offset szString2  
  dec   esi  
  
@@:           ;把倒过来的字符串再修正回来，放在 szString2  
  mov   al, BYTE ptr [esi]  
  mov   BYTE ptr [edi], al  
  dec   ecx  
  dec   esi  
  inc   edi  
  test  ecx, ecx  
  jnz   @B  
  mov     BYTE ptr [edi ], 28h      ;填写句柄字符串最后的"(H)"  
  mov     BYTE ptr [edi+1], 48h  
  mov     BYTE ptr [edi +2 ], 29h  
  mov     BYTE ptr [edi +3 ], 00  
  push    offset szString2  
  push    3EDh  
  push    hWnd  
  call    SetDlgItemText  
  mov     edi, offset szClassName  
  push    10h           ; nMaxCount  
  push    edi           ; lpClassName  
  push    lhWnd        ; hWnd  
  call    GetClassName  
  mov     edi, offset szClassName  
  push    edi           ; lpString  
  push    3E8h          ; nIDDlgItem  
  push    hWnd  
  call    SetDlgItemText      ; SetDlgItemTextA  
  push    0             ; lParam  
  push    0             ; wParam  
  push    0CCh          ; Msg  
  push    hWnd  
  call    PostMessageA  
.elseif eax == WM_LBUTTONDOWN  
  call    ReleaseCapture  
  push    0             ; lParam  
  push    2             ; wParam  
  push    0A1h          ; Msg  
  push    hWnd  
  call    SendMessageA  
.else  
  mov eax,FALSE  
  ret  
.endif  
xor eax, eax  
ret  
_ProcDlgMain  endp  
;---------------------------------------------------------------------------  
end  start  
```

编译成功后 `4.5k`，比原来软件 `12k` 稍微小点

`4dd466d0e1e0cc123afbe504da2462ca`
