
# 是男人就撑过 20 秒的游戏分析+无敌 Patch 的实现

> 原文：[`www.pediy.com/kssd/pediy10/70037.html`](https://www.pediy.com/kssd/pediy10/70037.html)

在论坛潜水了好长时间，终于忍不住想发表点什么了。（不然良心不安啊！`![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")` ）

这一篇算是偶的处女原创帖了，给大家带来的是一片关于《是男人就撑过 20 秒》这个游戏的分析和 Patch。本人菜鸟一只，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

先简单说说《是男人就撑过 20 秒》这个游戏，就是操纵一架飞机躲避飞来的 N 多子弹，看你能支撑多长时间。（真的很变态`![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")` ）

好，言归正传。

首先，观察游戏，发现一点，那就是每当进入游戏后鼠标就会消失。当游戏结束之后，显示你支撑的时间时，鼠标又出现了。`OK`，这是一个很好的线索。

用 `OllyDBG` 加载游戏，在函数 `ShowCursor` 下断。运行游戏后，游戏第一次断这里

代码:

```
004042F0  /$  53            push    ebx
004042F1  |.  56            push    esi
004042F2  |.  57            push    edi
004042F3  |.  55            push    ebp
004042F4  |.  83C4 D0       add     esp, -30
004042F7  |.  6A 00         push    0                           ; /Show = FALSE
004042F9  |.  E8 24050000   call    <jmp.&USER32.ShowCursor>     ;
\ShowCursor
004042FE  |.  E8 5D030000   call    00404660
00404303  |.  A1 C86D4000   mov     eax, dword ptr [406DC8]

```

那么，这里就应该是游戏开始，鼠标消失用的那个 `ShowCursor`，他的参数为 `FALSE` 说明分析没错。好，继续运行，开始游戏，当游戏结束时，不出所料，游戏断下来了。

代码:

```
00404590  /$  53            push    ebx
00404591  |.  83C4 E4       add     esp, -1C
00404594  |.  8BD8          mov     ebx, eax
00404596  |.  6A 01         push    1                           ; /Show = TRUE
00404598  |.  E8 85020000   call    <jmp.&USER32.ShowCursor>     ; \ShowCursor
0040459D  |.  833D CC6D4000>cmp     dword ptr [406DCC], 0

```

跟我想的一样，参数为 `TRUE`，鼠标显示出来，这样应该离目标不远了。接着我们看看是哪来在 `CALL 00404590` 。

代码:

```
00403614  |.  85C0          test    eax, eax
00403616  |.  74 3B         je      short 00403653
00403618  |.  83F8 11       cmp     eax, 11
0040361B  |.  75 17         jnz     short 00403634
0040361D  |.  33D2          xor     edx, edx
0040361F  |.  8915 906D4000 mov     dword ptr [406D90], edx
00403625  |.  A1 DC694000   mov     eax, dword ptr [4069DC]
0040362A  |.  E8 610F0000   call    00404590                  ;   就是这里
0040362F  |.  E9 83040000   jmp     00403AB7
00403634  |>  8B15 806D4000 mov     edx, dword ptr [406D80]
0040363A  |.  0FB71455 125C>movzx   edx, word ptr [edx*2+405C12]
00403642  |.  81C2 00504000 add     edx, 00405000
00403648  |.  FF05 806D4000 inc     dword ptr [406D80]

```

于是，我将这个函数段跑了一遍，其中有个大循环，应该是初始化子弹的工作（由于那段代码太多，就不贴上来了`![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")` ）。其实上面的代码就是游戏是否结束的关键。我想大家也都看出来了。看这代码的头两句。

`00403614  |.  85C0          test    eax, eax`

`00403616  |.  74 3B         je      short 00403653              ;   很可疑的跳转！`

这个跳转刚好跳过我们的结束代码地址 `00404590`，所以不妨吧 `je` 改为 `jmp`。保存后，运行游戏。发现飞机已经能够遨游在子弹群之中！！！`![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")`

好了，既然已经将关键的地方找出来了，接下来就是写一个程序 Patch 了。（程序很简单，也没有什么注释~~`![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")` ）

代码:

```
     .386
     .model flat , stdcall
     option casemap : none

;--------------------------------------------------------------------------------

include      windows.inc
include      kernel32.inc
include      user32.inc
include      advapi32.inc
includelib    kernel32.lib
includelib    user32.lib
includelib    advapi32.lib

;--------------------------------------------------------------------------------

IDD_DLG      equ  1000
IDC_BTN      equ  1001
IDC_STC      equ  1002

;--------------------------------------------------------------------------------

     .data?

hInstance    dd  ?
hToken      dd  ?
luid      LUID  <>
TokenPri    TOKEN_PRIVILEGES  <>

;--------------------------------------------------------------------------------

     .data

szPriText    db  "提权失败",0
szFWText    db  "请先运行游戏程序",0
dwAddress    dd  00403616h
bPatch      db  0ebh
bRestore    db  74h
szWindowName    db  "",0
szStart      db  "Start Cheat",0
szStop      db  "Stop Cheat",0
bCheat      db  0
szDebug      db  "SeSecurityPrivilege",0
;--------------------------------------------------------------------------------


     .code

_WriteProc    proc  hwndDlg,lpBuffer,dwSize

     LOCAL  pid,hHandle,tghwnd

     invoke  FindWindow,NULL,addr szWindowName
     mov  tghwnd,eax
     .if  eax == NULL
       invoke  MessageBox,hwndDlg,addr szFWText,NULL,MB_OK
       mov  eax,FALSE
       ret
     .endif

     invoke  GetWindowThreadProcessId,tghwnd,addr pid
     invoke  OpenProcess,PROCESS_ALL_ACCESS,FALSE,pid
     mov  hHandle,eax

     invoke  WriteProcessMemory,hHandle,dwAddress,lpBuffer,dwSize,NULL

     ret

_WriteProc endp

_Privilege    proc  hwndDlg


     invoke  GetCurrentProcess
     invoke  OpenProcessToken,eax,TOKEN_ADJUST_PRIVILEGES or TOKEN_QUERY ,addr hToken
     invoke  LookupPrivilegeValue,NULL,addr szDebug,addr TokenPri.Privileges.Luid

     mov  TokenPri.PrivilegeCount,1
     mov  TokenPri.Privileges.Attributes,SE_PRIVILEGE_ENABLED

     invoke  AdjustTokenPrivileges,hToken,FALSE,addr TokenPri,sizeof TokenPri,NULL,NULL

     .if  eax == FALSE
       invoke  MessageBox,hwndDlg,addr szPriText,NULL,MB_OK
     .endif

     ret

_Privilege endp

_ProcDlg    proc  uses ebx edi esi hwndDlg,uMsg,wParam,lParam

     mov  eax,uMsg
     .if  eax == WM_CLOSE
       invoke  EndDialog,hwndDlg,NULL
     .elseif  eax == WM_INITDIALOG
       invoke  _Privilege,hwndDlg
     .elseif eax == WM_COMMAND
       mov  eax,wParam
       .if  ax == IDC_BTN
         .if  bCheat == 0
           invoke  _WriteProc,hwndDlg,addr bPatch,1
           .if  eax == TRUE
             invoke SetDlgItemText,hwndDlg,IDC_BTN,addr szStop
             mov  bCheat,1
           .endif
         .else
           invoke  _WriteProc,hwndDlg,addr bRestore,1
             invoke  SetDlgItemText,hwndDlg,IDC_BTN,addr szStart
             mov  bCheat,0
         .endif
       .endif
     .else
       mov  eax,FALSE
       ret
     .endif
     mov  eax,TRUE
     ret

_ProcDlg    endp


start:

     invoke  GetModuleHandle,NULL
     mov  hInstance,eax
     invoke  DialogBoxParam,hInstance,IDD_DLG,NULL,addr _ProcDlg,NULL
     invoke  ExitProcess,NULL

     end  start

```

这样，我们就随随便便撑过 20 秒了。`o(∩_∩)o`...哈哈！

程序的源码和游戏我都放在附件里了，有兴趣的朋友可以玩玩~~~

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` || 程序+游戏.rar

 |
