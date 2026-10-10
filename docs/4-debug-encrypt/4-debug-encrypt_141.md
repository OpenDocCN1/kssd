

# Windows 优化大师 V7.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-143.htm`](https://www.pediy.com/kssd/pediy08/pediy8-143.htm)

【破解日期】 2006 年 5 月 20 日  
【破解作者】 Ryosuke(QQ:448089717)  
【作者邮箱】 hexx@ics.ict.ac.cn  
【使用工具】 OD,RSA-TOOL  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 Windows\*\*\*\* V7.2 Build 6.516  
【加入时间】 2006-05-17 16:22:00  
【下载地址】 http://www.skycn.com/soft/2988.html  
【加壳方式】 ASPack 2.12 -> Alexey Solodovnikov  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
先脱壳,ASPack 2.12 -> Alexey Solodovnikov 壳没有任何难度。脱完壳进行分析。  
输入姓名：nightfox  
获得申请码：666018  
运行，4 个注册码窗口都输入 12345678，没反应。  
找字符串中"windows 优化大师 v7.2 (已注册)"，来到:  
00599D98   .  55            push    ebp                          ;  (initial cpu selection)  
00599D99   .  8BEC          mov     ebp, esp  
00599D9B   .  B9 0C000000   mov     ecx, 0C  
00599DA0   >  6A 00         push    0  
00599DA2   .  6A 00         push    0  
00599DA4   .  49            dec     ecx  
00599DA5   .^ 75 F9         jnz     short 00599DA0  
00599DA7   .  51            push    ecx  
00599DA8   .  53            push    ebx  
00599DA9   .  56            push    esi  
00599DAA   .  57            push    edi  
00599DAB   .  8945 FC       mov     \[ebp-4\], eax  
00599DAE   .  33C0          xor     eax, eax  
00599DB0   .  55            push    ebp  
00599DB1   .  68 5FA25900   push    0059A25F  
00599DB6   .  64:FF30       push    dword ptr fs:\[eax\]  
00599DB9   .  64:8920       mov     fs:\[eax\], esp  
00599DBC   .  8D55 EC       lea     edx, \[ebp-14\]  
00599DBF   .  8B45 FC       mov     eax, \[ebp-4\]  
00599DC2   .  8B80 08030000 mov     eax, \[eax+308\]  
00599DC8   .  E8 8306EDFF   call    0046A450                     ;  获得 name  
00599DCD   .  8B45 EC       mov     eax, \[ebp-14\]  
00599DD0   .  E8 23B9E6FF   call    004056F8  
00599DD5   .  83F8 04       cmp     eax, 4  
00599DD8   .  7D 1F         jge     short 00599DF9  
00599DDA   .  6A 10         push    10  
00599DDC   .  68 70A25900   push    0059A270                     ;  windows 优化大师  
00599DE1   .  68 80A25900   push    0059A280                     ;  错误！注册姓名长度不足。注：注册姓名至少应为 4 位字母或两个汉字。  
00599DE6   .  8B45 FC       mov     eax, \[ebp-4\]  
00599DE9   .  E8 966FEDFF   call    00470D84  
00599DEE   .  50            push    eax                          ; |hOwner  
00599DEF   .  E8 5CEBE6FF   call    <jmp.&user32.MessageBoxA>    ; \\MessageBoxA  
00599DF4   .  E9 0C040000   jmp     0059A205  
00599DF9   >  A1 74A86300   mov     eax, \[63A874\]  
00599DFE   .  8338 01       cmp     dword ptr \[eax\], 1  
00599E01   .  0F85 00020000 jnz     0059A007  
00599E07   .  8D55 E4       lea     edx, \[ebp-1C\]  
00599E0A   .  8B45 FC       mov     eax, \[ebp-4\]  
00599E0D   .  8B80 20030000 mov     eax, \[eax+320\]  
00599E13   .  E8 3806EDFF   call    0046A450                     ;  获得注册码第一个字符串 A  
00599E18   .  FF75 E4       push    dword ptr \[ebp-1C\]  
00599E1B   .  8D55 E0       lea     edx, \[ebp-20\]  
00599E1E   .  8B45 FC       mov     eax, \[ebp-4\]  
00599E21   .  8B80 34030000 mov     eax, \[eax+334\]  
00599E27   .  E8 2406EDFF   call    0046A450                     ;  获得注册码第 2 个字符串 B  
00599E2C   .  FF75 E0       push    dword ptr \[ebp-20\]  
00599E2F   .  8D55 DC       lea     edx, \[ebp-24\]  
00599E32   .  8B45 FC       mov     eax, \[ebp-4\]  
00599E35   .  8B80 28030000 mov     eax, \[eax+328\]  
00599E3B   .  E8 1006EDFF   call    0046A450                     ;  获得注册码第 3 个字符串 C  
00599E40   .  FF75 DC       push    dword ptr \[ebp-24\]  
00599E43   .  8D55 D8       lea     edx, \[ebp-28\]  
00599E46   .  8B45 FC       mov     eax, \[ebp-4\]  
00599E49   .  8B80 30030000 mov     eax, \[eax+330\]  
00599E4F   .  E8 FC05EDFF   call    0046A450                     ;  获得注册码第 4 个字符串 D  
00599E54   .  FF75 D8       push    dword ptr \[ebp-28\]  
00599E57   .  8D45 E8       lea     eax, \[ebp-18\]  
00599E5A   .  BA 04000000   mov     edx, 4  
00599E5F   .  E8 54B9E6FF   call    004057B8                     ;  连接 A,B,C,D 成 str  
00599E64   .  8B45 E8       mov     eax, \[ebp-18\]                ;  保存地址  
00599E67   .  8D55 F8       lea     edx, \[ebp-8\]  
00599E6A   .  E8 2DE5FBFF   call    0055839C       ;跟进去知道，V7.2 采用 RC6 进行加密，一会分析这个 CALL  
00599E6F   .  8B45 F8       mov     eax, \[ebp-8\]                 ;  对注册码加密的结果\[ebp-8\]  
00599E72   .  E8 81B8E6FF   call    004056F8  
00599E77   .  83F8 18       cmp     eax, 18  
00599E7A   .  0F8E 7D030000 jle     0059A1FD  
00599E80   .  8D55 D4       lea     edx, \[ebp-2C\]  
00599E83   .  8B45 FC       mov     eax, \[ebp-4\]  
00599E86   .  8B80 08030000 mov     eax, \[eax+308\]  
00599E8C   .  E8 BF05EDFF   call    0046A450                     ;  获得 name  
00599E91   .  8B45 D4       mov     eax, \[ebp-2C\]                ;  name  
00599E94   .  8B55 F8       mov     edx, \[ebp-8\]                 ;  注册码加密结果 serialstr  
00599E97   .  E8 98DBFBFF   call    00557A34       ;F8 这个程序就运行了，没有到下面比较，这个 CALL 有问题，一会分析  
00599E9C   .  83F8 0A       cmp     eax, 0A  
00599E9F   .  0F85 58030000 jnz     0059A1FD  
00599EA5   .  A1 90AA6300   mov     eax, \[63AA90\]  
00599EAA   .  BA C8A25900   mov     edx, 0059A2C8                ;  the pig not exists!  (哈哈，作者太搞笑了)  
00599EAF   .  E8 D8B5E6FF   call    0040548C  
00599EB4   .  6A 00         push    0                            ; /lParam = 0  
00599EB6   .  68 689C5900   push    00599C68                     ; |Callback = dumped\_.00599C68  
00599EBB   .  E8 00E8E6FF   call    <jmp.&user32.EnumWindows>    ; \\EnumWindows  
00599EC0   .  8D55 CC       lea     edx, \[ebp-34\]  
00599EC3   .  A1 C8AC6300   mov     eax, \[63ACC8\]  
00599EC8   .  8B00          mov     eax, \[eax\]  
00599ECA   .  E8 310CEFFF   call    0048AB00  
00599ECF   .  8B45 CC       mov     eax, \[ebp-34\]  
00599ED2   .  8D55 D0       lea     edx, \[ebp-30\]  
00599ED5   .  E8 7237E7FF   call    0040D64C  
00599EDA   .  8B45 D0       mov     eax, \[ebp-30\]  
00599EDD   .  33D2          xor     edx, edx  
00599EDF   .  E8 44D7FBFF   call    00557628  
00599EE4   .  A1 90AA6300   mov     eax, \[63AA90\]  
00599EE9   .  8B00          mov     eax, \[eax\]  
00599EEB   .  E8 08B8E6FF   call    004056F8  
00599EF0   .  83F8 13       cmp     eax, 13  
00599EF3   .  0F85 04030000 jnz     0059A1FD  
00599EF9   .  A1 CCAC6300   mov     eax, \[63ACCC\]  
00599EFE   .  8B00          mov     eax, \[eax\]  
00599F00   .  8B80 38090000 mov     eax, \[eax+938\]  
00599F06   .  BA E4A25900   mov     edx, 0059A2E4                ;  windows 优化大师 v7.2 (已注册)  
00599F0B   .  E8 7005EDFF   call    0046A480  
00599F10   .  A1 CCAC6300   mov     eax, \[63ACCC\]  
00599F15   .  8B00          mov     eax, \[eax\]  
00599F17   .  8B80 30030000 mov     eax, \[eax+330\]  
00599F1D   .  BA 0CA35900   mov     edx, 0059A30C                ;  升级  
00599F22   .  E8 5905EDFF   call    0046A480  
00599F27   .  8D55 C8       lea     edx, \[ebp-38\]  
00599F2A   .  B8 1CA35900   mov     eax, 0059A31C                ;  f812930116634e5a9ec7e2a7270ebf6d  
  
V7.2 采用 RC6 加密分析  
0055839C  /$  55            push    ebp  
0055839D  |.  8BEC          mov     ebp, esp  
0055839F  |.  81C4 14FFFFFF add     esp, -0EC  
005583A5  |.  53            push    ebx  
005583A6  |.  56            push    esi  
005583A7  |.  33C9          xor     ecx, ecx  
005583A9  |.  898D 14FFFFFF mov     \[ebp-EC\], ecx  
005583AF  |.  894D FC       mov     \[ebp-4\], ecx  
005583B2  |.  894D F8       mov     \[ebp-8\], ecx  
005583B5  |.  8BF2          mov     esi, edx  
005583B7  |.  8BD8          mov     ebx, eax  
005583B9  |.  33C0          xor     eax, eax  
005583BB  |.  55            push    ebp  
005583BC  |.  68 AB845500   push    005584AB  
005583C1  |.  64:FF30       push    dword ptr fs:\[eax\]  
005583C4  |.  64:8920       mov     fs:\[eax\], esp  
005583C7  |.  8BC6          mov     eax, esi  
005583C9  |.  E8 6AD0EAFF   call    00405438  
005583CE  |.  85DB          test    ebx, ebx  
005583D0  |.  0F84 AF000000 je      00558485  
005583D6  |.  8D45 FC       lea     eax, \[ebp-4\]  
005583D9  |.  8BD3          mov     edx, ebx  
005583DB  |.  E8 F0D0EAFF   call    004054D0  
005583E0  |.  33DB          xor     ebx, ebx  
005583E2  |>  6A 00         /push    0                           ; /Arg1 = 00000000  
005583E4  |.  BA EC9E6300   |mov     edx, 00639EEC               ; |  
005583E9  |.  8D85 28FFFFFF |lea     eax, \[ebp-D8\]               ; |  
005583EF  |.  B9 10000000   |mov     ecx, 10                     ; |  
005583F4  |.  E8 4FBEFFFF   |call    00554248                    ; \\RC6 Init Key:0x00639EEC Length:0x10 见下面  
          //00639EEC  B2 3D C5 26 EB 5B CD F8 3B 5E 23 C1 72 43 A6 82  ??隱网;^#羠 C  
005583F9  |.  8B45 FC       |mov     eax, \[ebp-4\]  
005583FC  |.  E8 F7D4EAFF   |call    004058F8  
00558401  |.  8BD0          |mov     edx, eax  
00558403  |.  03D3          |add     edx, ebx  
00558405  |.  8D8D 18FFFFFF |lea     ecx, \[ebp-E8\]  
0055840B  |.  8D85 28FFFFFF |lea     eax, \[ebp-D8\]  
00558411  |.  E8 F6BFFFFF   |call    0055440C                    ;  RC6 Encipher  
00558416  |.  8D45 F8       |lea     eax, \[ebp-8\]  
00558419  |.  BA 20000000   |mov     edx, 20  
0055841E  |.  E8 61D6EAFF   |call    00405A84  
00558423  |.  8B45 F8       |mov     eax, \[ebp-8\]  
00558426  |.  E8 CDD4EAFF   |call    004058F8  
0055842B  |.  8BD0          |mov     edx, eax  
0055842D  |.  8D85 18FFFFFF |lea     eax, \[ebp-E8\]  
00558433  |.  B9 10000000   |mov     ecx, 10  
00558438  |.  E8 77EBECFF   |call    00426FB4                    ;  加密结果转成 32 字符 str  
0055843D  |.  8B45 F8       |mov     eax, \[ebp-8\]                ;  str  
00558440  |.  E8 B3D4EAFF   |call    004058F8  
00558445  |.  8BD0          |mov     edx, eax  
00558447  |.  8D85 14FFFFFF |lea     eax, \[ebp-EC\]  
0055844D  |.  E8 DED1EAFF   |call    00405630                    ;  str 存到\[ebp-ec\]  
00558452  |.  8B95 14FFFFFF |mov     edx, \[ebp-EC\]  
00558458  |.  8BC6          |mov     eax, esi  
0055845A  |.  E8 A1D2EAFF   |call    00405700  
0055845F  |.  8D45 F8       |lea     eax, \[ebp-8\]  
00558462  |.  E8 D1CFEAFF   |call    00405438  
00558467  |.  83C3 10       |add     ebx, 10  
0055846A  |.  8B45 FC       |mov     eax, \[ebp-4\]  
0055846D  |.  E8 86D2EAFF   |call    004056F8  
00558472  |.  3BD8          |cmp     ebx, eax  
00558474  |.^ 0F8C 68FFFFFF \\jl      005583E2                    ;  rc6 加密注册码  
//上面是初始化 RC6 和加密注册码，以后的分析知道注册码连起来的长度是 32 个字符。  
0055847A  |.  8D85 28FFFFFF lea     eax, \[ebp-D8\]  
00558480  |.  E8 77BFFFFF   call    005543FC  
00558485  |>  33C0          xor     eax, eax  
00558487  |.  5A            pop     edx  
00558488  |.  59            pop     ecx  
00558489  |.  59            pop     ecx  
0055848A  |.  64:8910       mov     fs:\[eax\], edx  
0055848D  |.  68 B2845500   push    005584B2  
00558492  |>  8D85 14FFFFFF lea     eax, \[ebp-EC\]  
00558498  |.  E8 9BCFEAFF   call    00405438  
0055849D  |.  8D45 F8       lea     eax, \[ebp-8\]  
005584A0  |.  BA 02000000   mov     edx, 2  
005584A5  |.  E8 B2CFEAFF   call    0040545C  
005584AA  \\.  C3            retn  
  
分析 00599E97   .  E8 98DBFBFF   call    00557A34 F8  
00427328  /$  55            push    ebp  
00427329  |.  8BEC          mov     ebp, esp  
0042732B  |.  83C4 F0       add     esp, -10  
0042732E  |.  53            push    ebx  
0042732F  |.  56            push    esi  
00427330  |.  33DB          xor     ebx, ebx  
00427332  |.  895D F0       mov     \[ebp-10\], ebx  
00427335  |.  84D2          test    dl, dl  
00427337  |.  74 08         je      short 00427341  
00427339  |.  83C4 F0       add     esp, -10  
0042733C  |.  E8 03D6FDFF   call    00404944  
00427341  |>  8BD9          mov     ebx, ecx  
00427343  |.  8855 FF       mov     \[ebp-1\], dl  
00427346  |.  8BF0          mov     esi, eax  
00427348  |.  33C0          xor     eax, eax  
0042734A  |.  55            push    ebp  
0042734B  |.  68 D2734200   push    004273D2  
00427350  |.  64:FF30       push    dword ptr fs:\[eax\]  
00427353  |.  64:8920       mov     fs:\[eax\], esp  
00427356  |.  33D2          xor     edx, edx  
00427358  |.  8BC6          mov     eax, esi  
0042735A  |.  E8 3DD2FDFF   call    0040459C  
0042735F  |.  E8 64FDFFFF   call    004270C8  
00427364  |.  885E 0E       mov     \[esi+E\], bl  
00427367  |.  885E 0C       mov     \[esi+C\], bl  
0042736A  |.  56            push    esi                           ; /Arg3  
0042736B  |.  6A 04         push    4                             ; |Arg2 = 00000004  
0042736D  |.  8D46 08       lea     eax, \[esi+8\]                  ; |  
00427370  |.  50            push    eax                           ; |Arg1  
00427371  |.  B9 80724200   mov     ecx, 00427280                 ; |  
00427376  |.  33D2          xor     edx, edx                      ; |  
00427378  |.  33C0          xor     eax, eax                      ; |  
0042737A  |.  E8 6DE0FDFF   call    004053EC                      ; \\dumped\_.004053EC  //这个是关键，里面创建了一个线程，进行注册码验证，跟进。  
0042737F  |.  8BD8          mov     ebx, eax  
00427381  |.  895E 04       mov     \[esi+4\], ebx  
00427384  |.  85DB          test    ebx, ebx  
00427386  |.  75 34         jnz     short 004273BC  
00427388  |.  E8 4B0BFEFF   call    <jmp.&kernel32.GetLastError>  ; \[GetLastError  
0042738D  |.  8D55 F0       lea     edx, \[ebp-10\]  
00427390  |.  E8 1394FEFF   call    004107A8  
00427395  |.  8B45 F0       mov     eax, \[ebp-10\]  
00427398  |.  8945 F4       mov     \[ebp-C\], eax  
0042739B  |.  C645 F8 0B    mov     byte ptr \[ebp-8\], 0B  
0042739F  |.  8D45 F4       lea     eax, \[ebp-C\]  
004273A2  |.  50            push    eax  
004273A3  |.  6A 00         push    0  
004273A5  |.  8B0D ECAE6300 mov     ecx, \[63AEEC\]                 ;  dumped\_.0041A4A8  
004273AB  |.  B2 01         mov     dl, 1  
004273AD  |.  A1 08CA4100   mov     eax, \[41CA08\]  
004273B2  |.  E8 319DFEFF   call    004110E8  
004273B7  |.  E8 F0D9FDFF   call    00404DAC  
004273BC  |>  33C0          xor     eax, eax  
004273BE  |.  5A            pop     edx  
004273BF  |.  59            pop     ecx  
004273C0  |.  59            pop     ecx  
004273C1  |.  64:8910       mov     fs:\[eax\], edx  
004273C4  |.  68 D9734200   push    004273D9  
004273C9  |>  8D45 F0       lea     eax, \[ebp-10\]  
004273CC  |.  E8 67E0FDFF   call    00405438  
004273D1  \\.  C3            retn  
  
004053EC  /$  55            push    ebp  
004053ED  |.  8BEC          mov     ebp, esp  
004053EF  |.  53            push    ebx  
004053F0  |.  56            push    esi  
004053F1  |.  57            push    edi  
004053F2  |.  8BF9          mov     edi, ecx  
004053F4  |.  8BF2          mov     esi, edx  
004053F6  |.  8BD8          mov     ebx, eax  
004053F8  |.  B8 08000000   mov     eax, 8  
004053FD  |.  E8 32DAFFFF   call    00402E34  
00405402  |.  8938          mov     \[eax\], edi  
00405404  |.  8B55 10       mov     edx, \[ebp+10\]  
00405407  |.  8950 04       mov     \[eax+4\], edx  
0040540A  |.  C605 4DC06300>mov     byte ptr \[63C04D\], 1  
00405411  |.  8B55 08       mov     edx, \[ebp+8\]  
00405414  |.  52            push    edx                           ; /pThreadId  
00405415  |.  8B55 0C       mov     edx, \[ebp+C\]                  ; |  
00405418  |.  52            push    edx                           ; |CreationFlags  
00405419  |.  50            push    eax                           ; |pThreadParm  
0040541A  |.  B8 B4534000   mov     eax, 004053B4                 ; |  
0040541F  |.  50            push    eax                           ; |ThreadFunction => dumped\_.004053B4  
00405420  |.  56            push    esi                           ; |StackSize  
00405421  |.  53            push    ebx                           ; |pSecurity  
00405422  |.  E8 D1BEFFFF   call    <jmp.&kernel32.CreateThread>  ; \\CreateThread  //呵呵，作者保护的核心到了，在 ThreadFunction 下断。  
00405427  |.  5F            pop     edi  
00405428  |.  5E            pop     esi  
00405429  |.  5B            pop     ebx  
0040542A  |.  5D            pop     ebp  
0040542B  \\.  C2 0C00       retn    0C  
  
不一会来到，验证的部分了。  
00558DDC   .  55            push    ebp  
00558DDD   .  8BEC          mov     ebp, esp  
00558DDF   .  B9 07000000   mov     ecx, 7  
00558DE4   >  6A 00         push    0  
00558DE6   .  6A 00         push    0  
00558DE8   .  49            dec     ecx  
00558DE9   .^ 75 F9         jnz     short 00558DE4  
00558DEB   .  53            push    ebx  
00558DEC   .  56            push    esi  
00558DED   .  57            push    edi  
00558DEE   .  8BD8          mov     ebx, eax  
00558DF0   .  8D45 F8       lea     eax, \[ebp-8\]  
00558DF3   .  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00558DF9   .  E8 02D1EAFF   call    00405F00  
00558DFE   .  8D45 F0       lea     eax, \[ebp-10\]  
00558E01   .  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00558E07   .  E8 F4D0EAFF   call    00405F00  
00558E0C   .  8D45 E8       lea     eax, \[ebp-18\]  
00558E0F   .  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00558E15   .  E8 E6D0EAFF   call    00405F00  
00558E1A   .  33C0          xor     eax, eax  
00558E1C   .  55            push    ebp  
00558E1D   .  68 538F5500   push    00558F53  
00558E22   .  64:FF30       push    dword ptr fs:\[eax\]  
00558E25   .  64:8920       mov     fs:\[eax\], esp  
00558E28   .  C705 50EA6400>mov     dword ptr \[64EA50\], 64  
00558E32   .  8D55 D4       lea     edx, \[ebp-2C\]  
00558E35   .  8B43 40       mov     eax, \[ebx+40\]                 ;  name  
00558E38   .  E8 8F39EBFF   call    0040C7CC                      ;  name=>ebp-2c  
00558E3D   .  837D D4 00    cmp     dword ptr \[ebp-2C\], 0  
00558E41   .  0F84 DE000000 je      00558F25  
00558E47   .  8D55 D0       lea     edx, \[ebp-30\]  
00558E4A   .  8B43 44       mov     eax, \[ebx+44\]  
00558E4D   .  E8 7A39EBFF   call    0040C7CC                      ;  serial=>\[ebp-30\]  
00558E52   .  837D D0 00    cmp     dword ptr \[ebp-30\], 0  
00558E56   .  0F84 C9000000 je      00558F25  
00558E5C   .  8D55 CC       lea     edx, \[ebp-34\]  
00558E5F   .  B8 6C8F5500   mov     eax, 00558F6C                 ;  ASCII "AF014BFCE79DD52DF3DF2E2DDF46585E0ABC524CA40814A5EFE7FED006BFC95FFA5655894A053722151B220320417E7A"  
00558E64   .  E8 1FF4FFFF   call    00558288                      ;  RC6 解密上面的字符串=>a 解密结果是"260853666017218556428756318395867010513"，看到这里，估计验证用到了 RSA 了，随即用 RSAtool 分解了一下 260853666017218556428756318395867010513=17108966812627532699\*15246605413056944387，呵呵，接下来的分析有了基础了，这个值即是 RSA 中 N  
00558E69   .  8B45 CC       mov     eax, \[ebp-34\]  
00558E6C   .  8D4D DC       lea     ecx, \[ebp-24\]  
00558E6F   .  BA 27000000   mov     edx, 27  
00558E74   .  E8 CB3AEEFF   call    0043C944  
00558E79   .  8D55 C8       lea     edx, \[ebp-38\]  
00558E7C   .  B8 D88F5500   mov     eax, 00558FD8                 ;  ASCII "0E03F9934B8E9786E0CE1BA0B4B97DE944AEC4C644E669573CB18EA0810941F4C28B59545FA2E8136603F274BCF7E986"  
00558E81   .  E8 02F4FFFF   call    00558288                      ;  RC6 解密=>b 解密结果"133628944400812954421111162414689994885"，这个是 RSA 中的 E，那么 RSA 中的 D 是 65537(0x10001)，到这里已经坚定 RSA 的决心了。  
00558E86   .  8B45 C8       mov     eax, \[ebp-38\]  
00558E89   .  8D4D D8       lea     ecx, \[ebp-28\]  
00558E8C   .  BA 27000000   mov     edx, 27  
00558E91   .  E8 AE3AEEFF   call    0043C944  
00558E96   .  8D55 E4       lea     edx, \[ebp-1C\]  
00558E99   .  8B43 40       mov     eax, \[ebx+40\]      ;通过 name 生成申请号，我的 name "nightfox"  
00558E9C   .  E8 73EEFFFF   call    00557D14                      ;  产生注册申请号 "666018" 由于后面直接明码用到申请号。我就不跟了，注册时候申请一下就可以了。  
00558EA1   .  8D55 F0       lea     edx, \[ebp-10\]  
00558EA4   .  8B45 DC       mov     eax, \[ebp-24\]  
00558EA7   .  E8 08BEFFFF   call    00554CB4                      ;  a=>二进制 BigInt  
00558EAC   .  8D55 F8       lea     edx, \[ebp-8\]  
00558EAF   .  8B45 D8       mov     eax, \[ebp-28\]  
00558EB2   .  E8 FDBDFFFF   call    00554CB4                      ;  b=>二进制 BigInt  
00558EB7   .  33C0          xor     eax, eax  
00558EB9   .  55            push    ebp  
00558EBA   .  68 1B8F5500   push    00558F1B  
00558EBF   .  64:FF30       push    dword ptr fs:\[eax\]  
00558EC2   .  64:8920       mov     fs:\[eax\], esp  
00558EC5   .  8D45 E8       lea     eax, \[ebp-18\]  
00558EC8   .  E8 63C0FFFF   call    00554F30  
00558ECD   .  8D55 E0       lea     edx, \[ebp-20\]  
00558ED0   .  8B43 44       mov     eax, \[ebx+44\]     ;输入的注册码"12345678123456781234567812345678"  
00558ED3   .  E8 1CBCFFFF   call    00554AF4                      ;  转换成 HEX 这里是 RSA 中的明文 M  
00558ED8   .  8D45 E8       lea     eax, \[ebp-18\]  
00558EDB   .  50            push    eax  
00558EDC   .  8D45 E8       lea     eax, \[ebp-18\]  
00558EDF   .  50            push    eax  
00558EE0   .  8D45 E8       lea     eax, \[ebp-18\]  
00558EE3   .  50            push    eax  
00558EE4   .  8D45 E8       lea     eax, \[ebp-18\]  
00558EE7   .  50            push    eax  
00558EE8   .  8D45 E0       lea     eax, \[ebp-20\]  
00558EEB   .  50            push    eax  
00558EEC   .  8D4D F0       lea     ecx, \[ebp-10\]  
00558EEF   .  8D55 F8       lea     edx, \[ebp-8\]  
00558EF2   .  8B45 E0       mov     eax, \[ebp-20\]  
00558EF5   .  E8 12E2FFFF   call    0055710C      ;求 C=M^E mod N 这里作者做了点小手脚，待会讲。  
00558EFA   .  8B45 E4       mov     eax, \[ebp-1C\]    ;申请号  
00558EFD   .  8B55 E0       mov     edx, \[ebp-20\]    ;C  
00558F00   .  E8 3FC9EAFF   call    00405844                      ;  判断是否申请号 和 加密的 C 是否相等  
00558F05   .  75 0A         jnz     short 00558F11      ;爆破点了。  
00558F07   .  C705 50EA6400>mov     dword ptr \[64EA50\], 0A        ;  注册标志，呵呵，  
00558F11   >  33C0          xor     eax, eax  
00558F13   .  5A            pop     edx  
00558F14   .  59            pop     ecx  
00558F15   .  59            pop     ecx  
00558F16   .  64:8910       mov     fs:\[eax\], edx  
00558F19   .  EB 0A         jmp     short 00558F25  
00558F1B   .^ E9 A0BBEAFF   jmp     00404AC0  
00558F20   .  E8 03BFEAFF   call    00404E28  
00558F25   >  33C0          xor     eax, eax  
00558F27   .  5A            pop     edx  
00558F28   .  59            pop     ecx  
00558F29   .  59            pop     ecx  
00558F2A   .  64:8910       mov     fs:\[eax\], edx  
00558F2D   .  68 5A8F5500   push    00558F5A  
00558F32   >  8D45 C8       lea     eax, \[ebp-38\]  
00558F35   .  BA 08000000   mov     edx, 8  
00558F3A   .  E8 1DC5EAFF   call    0040545C  
00558F3F   .  8D45 E8       lea     eax, \[ebp-18\]  
00558F42   .  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00558F48   .  B9 03000000   mov     ecx, 3  
00558F4D   .  E8 CAD0EAFF   call    0040601C  
00558F52   .  C3            retn  
00558F53   .^ E9 1CBEEAFF   jmp     00404D74  
00558F58   .^ EB D8         jmp     short 00558F32  
00558F5A   .  5F            pop     edi  
00558F5B   .  5E            pop     esi  
00558F5C   .  5B            pop     ebx  
00558F5D   .  8BE5          mov     esp, ebp  
00558F5F   .  5D            pop     ebp  
00558F60   .  C3            retn  
  
现在讲作者做的小手脚。  
00558EF5   .  E8 12E2FFFF   call    0055710C 跟进  
0055710C  /$  55            push    ebp  
0055710D  |.  8BEC          mov     ebp, esp  
0055710F  |.  83C4 A4       add     esp, -5C  
00557112  |.  53            push    ebx  
00557113  |.  56            push    esi  
00557114  |.  57            push    edi  
00557115  |.  33DB          xor     ebx, ebx  
00557117  |.  895D A4       mov     \[ebp-5C\], ebx  
0055711A  |.  895D A8       mov     \[ebp-58\], ebx  
0055711D  |.  895D AC       mov     \[ebp-54\], ebx  
00557120  |.  895D B8       mov     \[ebp-48\], ebx  
00557123  |.  895D B4       mov     \[ebp-4C\], ebx  
00557126  |.  895D B0       mov     \[ebp-50\], ebx  
00557129  |.  894D FC       mov     \[ebp-4\], ecx  
0055712C  |.  8BFA          mov     edi, edx  
0055712E  |.  8BF0          mov     esi, eax  
00557130  |.  8D45 F4       lea     eax, \[ebp-C\]  
00557133  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00557139  |.  E8 C2EDEAFF   call    00405F00  
0055713E  |.  8D45 EC       lea     eax, \[ebp-14\]  
00557141  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00557147  |.  E8 B4EDEAFF   call    00405F00  
0055714C  |.  8D45 E4       lea     eax, \[ebp-1C\]  
0055714F  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00557155  |.  E8 A6EDEAFF   call    00405F00  
0055715A  |.  8D45 DC       lea     eax, \[ebp-24\]  
0055715D  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00557163  |.  E8 98EDEAFF   call    00405F00  
00557168  |.  8D45 D4       lea     eax, \[ebp-2C\]  
0055716B  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
00557171  |.  E8 8AEDEAFF   call    00405F00  
00557176  |.  8D45 CC       lea     eax, \[ebp-34\]  
00557179  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
0055717F  |.  E8 7CEDEAFF   call    00405F00  
00557184  |.  8D45 C4       lea     eax, \[ebp-3C\]  
00557187  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
0055718D  |.  E8 6EEDEAFF   call    00405F00  
00557192  |.  8D45 BC       lea     eax, \[ebp-44\]  
00557195  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
0055719B  |.  E8 60EDEAFF   call    00405F00  
005571A0  |.  33C0          xor     eax, eax  
005571A2  |.  55            push    ebp  
005571A3  |.  68 CA745500   push    005574CA  
005571A8  |.  64:FF30       push    dword ptr fs:\[eax\]  
005571AB  |.  64:8920       mov     fs:\[eax\], esp  
005571AE  |.  8D55 BC       lea     edx, \[ebp-44\]  
005571B1  |.  B8 E4745500   mov     eax, 005574E4  
005571B6  |.  E8 91EAFFFF   call    00555C4C  
005571BB  |.  8D55 B8       lea     edx, \[ebp-48\]  
005571BE  |.  8B45 FC       mov     eax, \[ebp-4\]  
005571C1  |.  E8 A2E9FFFF   call    00555B68  
005571C6  |.  8B45 B8       mov     eax, \[ebp-48\]                 ;  N  
005571C9  |.  E8 2AE5EAFF   call    004056F8  
005571CE  |.  8BD8          mov     ebx, eax                      ;  128 位  
005571D0  |.  8D55 B8       lea     edx, \[ebp-48\]  
005571D3  |.  8BC6          mov     eax, esi  
005571D5  |.  E8 72D7FFFF   call    0055494C  
005571DA  |.  EB 12         jmp     short 005571EE  
005571DC  |>  8D45 B8       /lea     eax, \[ebp-48\]  
005571DF  |.  B9 01000000   |mov     ecx, 1  
005571E4  |.  BA 01000000   |mov     edx, 1  
005571E9  |.  E8 AAE7EAFF   |call    00405998  
005571EE  |>  8D45 AC        lea     eax, \[ebp-54\]  
005571F1  |.  50            |push    eax  
005571F2  |.  B9 01000000   |mov     ecx, 1  
005571F7  |.  BA 01000000   |mov     edx, 1  
005571FC  |.  8B45 B8       |mov     eax, \[ebp-48\]                ;  M 表示的二进制字符串  
005571FF  |.  E8 54E7EAFF   |call    00405958  
00557204  |.  8B45 AC       |mov     eax, \[ebp-54\]  
00557207  |.  BA E4745500   |mov     edx, 005574E4  
0055720C  |.  E8 33E6EAFF   |call    00405844  
00557211  |.^ 74 C9         \\je      short 005571DC  
00557213  |.  EB 10         jmp     short 00557225  
00557215  |>  8D45 B8       /lea     eax, \[ebp-48\]  
00557218  |.  8B4D B8       |mov     ecx, \[ebp-48\]  
0055721B  |.  BA E4745500   |mov     edx, 005574E4  
00557220  |.  E8 1FE5EAFF   |call    00405744  
00557225  |>  8B45 B8        mov     eax, \[ebp-48\]  
00557228  |.  E8 CBE4EAFF   |call    004056F8  
0055722D  |.  99            |cdq  
0055722E  |.  F7FB          |idiv    ebx  
00557230  |.  85D2          |test    edx, edx  
00557232  |.^ 75 E1         \\jnz     short 00557215  
00557234  |.  837F 04 00    cmp     dword ptr \[edi+4\], 0  
00557238  |.  75 48         jnz     short 00557282  
0055723A  |.  8D4D E4       lea     ecx, \[ebp-1C\]  
0055723D  |.  8B55 10       mov     edx, \[ebp+10\]  
00557240  |.  8B45 0C       mov     eax, \[ebp+C\]  
00557243  |.  E8 68FCFFFF   call    00556EB0  
00557248  |.  8D4D C4       lea     ecx, \[ebp-3C\]  
0055724B  |.  8D55 E4       lea     edx, \[ebp-1C\]  
0055724E  |.  8B45 0C       mov     eax, \[ebp+C\]  
00557251  |.  E8 1AE5FFFF   call    00555770  
00557256  |.  8D45 E4       lea     eax, \[ebp-1C\]  
00557259  |.  E8 D2DCFFFF   call    00554F30  
0055725E  |.  8D4D E4       lea     ecx, \[ebp-1C\]  
00557261  |.  8B55 0C       mov     edx, \[ebp+C\]  
00557264  |.  8B45 10       mov     eax, \[ebp+10\]  
00557267  |.  E8 44FCFFFF   call    00556EB0  
0055726C  |.  8D4D CC       lea     ecx, \[ebp-34\]  
0055726F  |.  8D55 E4       lea     edx, \[ebp-1C\]  
00557272  |.  8B45 10       mov     eax, \[ebp+10\]  
00557275  |.  E8 F6E4FFFF   call    00555770  
0055727A  |.  8D45 E4       lea     eax, \[ebp-1C\]  
0055727D  |.  E8 AEDCFFFF   call    00554F30  
00557282  |>  8B45 B8       mov     eax, \[ebp-48\]  
00557285  |.  E8 6EE4EAFF   call    004056F8  
0055728A  |.  99            cdq  
0055728B  |.  F7FB          idiv    ebx  
0055728D  |.  8BF0          mov     esi, eax  
0055728F  |.  8D45 B4       lea     eax, \[ebp-4C\]  
00557292  |.  E8 A1E1EAFF   call    00405438  
00557297  |.  85F6          test    esi, esi  
00557299  |.  0F8E 7C010000 jle     0055741B  
0055729F  |>  8D45 B0       /lea     eax, \[ebp-50\]  
005572A2  |.  50            |push    eax  
005572A3  |.  8BCB          |mov     ecx, ebx  
005572A5  |.  BA 01000000   |mov     edx, 1  
005572AA  |.  8B45 B8       |mov     eax, \[ebp-48\]                ;  M  
005572AD  |.  E8 A6E6EAFF   |call    00405958  
005572B2  |.  EB 12         |jmp     short 005572C6  
005572B4  |>  8D45 B0       |/lea     eax, \[ebp-50\]  
005572B7  |.  B9 01000000   ||mov     ecx, 1  
005572BC  |.  BA 01000000   ||mov     edx, 1  
005572C1  |.  E8 D2E6EAFF   ||call    00405998  
005572C6  |>  8D45 A8       | lea     eax, \[ebp-58\]  
005572C9  |.  50            ||push    eax  
005572CA  |.  B9 01000000   ||mov     ecx, 1  
005572CF  |.  BA 01000000   ||mov     edx, 1  
005572D4  |.  8B45 B0       ||mov     eax, \[ebp-50\]  
005572D7  |.  E8 7CE6EAFF   ||call    00405958  
005572DC  |.  8B45 A8       ||mov     eax, \[ebp-58\]  
005572DF  |.  BA E4745500   ||mov     edx, 005574E4  
005572E4  |.  E8 5BE5EAFF   ||call    00405844  
005572E9  |.  75 0B         ||jnz     short 005572F6  
005572EB  |.  8B45 B0       ||mov     eax, \[ebp-50\]               ;  M  
005572EE  |.  E8 05E4EAFF   ||call    004056F8  
005572F3  |.  48            ||dec     eax  
005572F4  |.^ 7F BE         |\\jg      short 005572B4  
005572F6  |>  8D55 F4       |lea     edx, \[ebp-C\]  
005572F9  |.  8B45 B0       |mov     eax, \[ebp-50\]  
005572FC  |.  E8 4BE9FFFF   |call    00555C4C  
00557301  |.  8D45 B8       |lea     eax, \[ebp-48\]  
00557304  |.  8BCB          |mov     ecx, ebx  
00557306  |.  BA 01000000   |mov     edx, 1  
0055730B  |.  E8 88E6EAFF   |call    00405998  
00557310  |.  8B45 B0       |mov     eax, \[ebp-50\]  
00557313  |.  BA E4745500   |mov     edx, 005574E4  
00557318  |.  E8 27E5EAFF   |call    00405844  
0055731D  |.  75 10         |jnz     short 0055732F  
0055731F  |.  8D55 EC       |lea     edx, \[ebp-14\]  
00557322  |.  8D45 BC       |lea     eax, \[ebp-44\]  
00557325  |.  E8 56E2FFFF   |call    00555580  
0055732A  |.  E9 91000000   |jmp     005573C0  
0055732F  |>  837F 04 00    |cmp     dword ptr \[edi+4\], 0  
00557333  |.  74 13         |je      short 00557348  
00557335  |.  8D45 EC       |lea     eax, \[ebp-14\]  
00557338  |.  50            |push    eax  
00557339  |.  8B4D FC       |mov     ecx, \[ebp-4\]  
0055733C  |.  8BD7          |mov     edx, edi  
0055733E  |.  8D45 F4       |lea     eax, \[ebp-C\]  
00557341  |.  E8 4EF7FFFF   |call    00556A94  
00557346  |.  EB 78         |jmp     short 005573C0  
00557348  |>  8D45 E4       |lea     eax, \[ebp-1C\]  
0055734B  |.  50            |push    eax  
0055734C  |.  8B4D 10       |mov     ecx, \[ebp+10\]  
0055734F  |.  8B55 18       |mov     edx, \[ebp+18\]  
00557352  |.  8D45 F4       |lea     eax, \[ebp-C\]  
00557355  |.  E8 3AF7FFFF   |call    00556A94  
0055735A  |.  8D4D D4       |lea     ecx, \[ebp-2C\]  
0055735D  |.  8D55 C4       |lea     edx, \[ebp-3C\]  
00557360  |.  8D45 E4       |lea     eax, \[ebp-1C\]  
00557363  |.  E8 08E4FFFF   |call    00555770  
00557368  |.  8D55 E4       |lea     edx, \[ebp-1C\]  
0055736B  |.  8D45 D4       |lea     eax, \[ebp-2C\]  
0055736E  |.  E8 0DE2FFFF   |call    00555580  
00557373  |.  8D45 DC       |lea     eax, \[ebp-24\]  
00557376  |.  50            |push    eax  
00557377  |.  8B4D 0C       |mov     ecx, \[ebp+C\]  
0055737A  |.  8B55 14       |mov     edx, \[ebp+14\]  
0055737D  |.  8D45 F4       |lea     eax, \[ebp-C\]  
00557380  |.  E8 0FF7FFFF   |call    00556A94  
00557385  |.  8D4D D4       |lea     ecx, \[ebp-2C\]  
00557388  |.  8D55 CC       |lea     edx, \[ebp-34\]  
0055738B  |.  8D45 DC       |lea     eax, \[ebp-24\]  
0055738E  |.  E8 DDE3FFFF   |call    00555770  
00557393  |.  8D55 DC       |lea     edx, \[ebp-24\]  
00557396  |.  8D45 D4       |lea     eax, \[ebp-2C\]  
00557399  |.  E8 E2E1FFFF   |call    00555580  
0055739E  |.  8D45 EC       |lea     eax, \[ebp-14\]  
005573A1  |.  50            |push    eax  
005573A2  |.  8B4D FC       |mov     ecx, \[ebp-4\]  
005573A5  |.  8D55 DC       |lea     edx, \[ebp-24\]  
005573A8  |.  8D45 E4       |lea     eax, \[ebp-1C\]  
005573AB  |.  E8 A4F1FFFF   |call    00556554  
005573B0  |.  8D45 E4       |lea     eax, \[ebp-1C\]  
005573B3  |.  E8 78DBFFFF   |call    00554F30  
005573B8  |.  8D45 DC       |lea     eax, \[ebp-24\]  
005573BB  |.  E8 70DBFFFF   |call    00554F30  
005573C0  |>  8D45 F4       |lea     eax, \[ebp-C\]  
005573C3  |.  E8 68DBFFFF   |call    00554F30  
005573C8  |.  8D45 B0       |lea     eax, \[ebp-50\]  
005573CB  |.  E8 68E0EAFF   |call    00405438  
005573D0  |.  8D55 B0       |lea     edx, \[ebp-50\]  
005573D3  |.  8D45 EC       |lea     eax, \[ebp-14\]  
005573D6  |.  E8 8DE7FFFF   |call    00555B68  
005573DB  |.  EB 10         |jmp     short 005573ED  
005573DD  |>  8D45 B0       |/lea     eax, \[ebp-50\]  
005573E0  |.  8B4D B0       ||mov     ecx, \[ebp-50\]  
005573E3  |.  BA E4745500   ||mov     edx, 005574E4  
005573E8  |.  E8 57E3EAFF   ||call    00405744  
005573ED  |>  8B45 B0       | mov     eax, \[ebp-50\]  
005573F0  |.  E8 03E3EAFF   ||call    004056F8  
005573F5  |.  8BD3          ||mov     edx, ebx  
005573F7  |.  4A            ||dec     edx  
005573F8  |.  8BCA          ||mov     ecx, edx  
005573FA  |.  99            ||cdq  
005573FB  |.  F7F9          ||idiv    ecx  
005573FD  |.  85D2          ||test    edx, edx  
005573FF  |.^ 75 DC         |\\jnz     short 005573DD  
00557401  |.  8D45 B4       |lea     eax, \[ebp-4C\]  
00557404  |.  8B55 B0       |mov     edx, \[ebp-50\]  
00557407  |.  E8 F4E2EAFF   |call    00405700  
0055740C  |.  8D45 EC       |lea     eax, \[ebp-14\]  
0055740F  |.  E8 1CDBFFFF   |call    00554F30  
00557414  |.  4E            |dec     esi  
00557415  |.^ 0F85 84FEFFFF \\jnz     0055729F  
//分析可知上面是在进行 M^E mod N 的计算。  
0055741B  |>  837F 04 00    cmp     dword ptr \[edi+4\], 0  
0055741F  |.  75 24         jnz     short 00557445  
00557421  |.  8D45 CC       lea     eax, \[ebp-34\]  
00557424  |.  E8 07DBFFFF   call    00554F30  
00557429  |.  8D45 C4       lea     eax, \[ebp-3C\]  
0055742C  |.  E8 FFDAFFFF   call    00554F30  
00557431  |.  EB 12         jmp     short 00557445  
00557433  |>  8D45 B4       /lea     eax, \[ebp-4C\]  
00557436  |.  B9 01000000   |mov     ecx, 1  
0055743B  |.  BA 01000000   |mov     edx, 1  
00557440  |.  E8 53E5EAFF   |call    00405998  
00557445  |>  8D45 A4        lea     eax, \[ebp-5C\]  
00557448  |.  50            |push    eax  
00557449  |.  B9 03000000   |mov     ecx, 3  
0055744E  |.  BA 01000000   |mov     edx, 1  
00557453  |.  8B45 B4       |mov     eax, \[ebp-4C\]                ;  M^E 计算结果  
00557456  |.  E8 FDE4EAFF   |call    00405958  
0055745B  |.  8B45 A4       |mov     eax, \[ebp-5C\]  
0055745E  |.  BA F0745500   |mov     edx, 005574F0                ;  ASCII "111"  
00557463  |.  E8 DCE3EAFF   |call    00405844                     ;  比较是不是 111  
00557468  |.  74 0D         |je      short 00557477  
0055746A  |.  8B45 B4       |mov     eax, \[ebp-4C\]  
0055746D  |.  E8 86E2EAFF   |call    004056F8  
00557472  |.  83F8 03       |cmp     eax, 3  
00557475  |.^ 7F BC         \\jg      short 00557433  
作者做的手脚就是对 M^E mod N 的结果进行了变换，对其二进制字符表示的，找到第一个"111"，然后去其后面的串作为比较的依据。  
所以我在这里就在前面加了一个 7，比如我的申请号是"666018",他的 16 进制表示"363636303138"，我做手脚后"7363636303138"  
00557477  |>  8D45 B4       lea     eax, \[ebp-4C\]                 ;  变换的 M^E 结果  
0055747A  |.  B9 03000000   mov     ecx, 3  
0055747F  |.  BA 01000000   mov     edx, 1  
00557484  |.  E8 0FE5EAFF   call    00405998                      ;  头三位为 0  
00557489  |.  8B55 08       mov     edx, \[ebp+8\]  
0055748C  |.  8B45 B4       mov     eax, \[ebp-4C\]  
0055748F  |.  E8 64D5FFFF   call    005549F8  
00557494  |.  8D45 BC       lea     eax, \[ebp-44\]  
00557497  |.  E8 94DAFFFF   call    00554F30  
0055749C  |.  33C0          xor     eax, eax  
0055749E  |.  5A            pop     edx  
0055749F  |.  59            pop     ecx  
005574A0  |.  59            pop     ecx  
005574A1  |.  64:8910       mov     fs:\[eax\], edx  
005574A4  |.  68 D1745500   push    005574D1  
005574A9  |>  8D45 A4       lea     eax, \[ebp-5C\]  
005574AC  |.  BA 06000000   mov     edx, 6  
005574B1  |.  E8 A6DFEAFF   call    0040545C  
005574B6  |.  8D45 BC       lea     eax, \[ebp-44\]  
005574B9  |.  8B15 28475500 mov     edx, \[554728\]                 ;  dumped\_.0055472C  
005574BF  |.  B9 08000000   mov     ecx, 8  
005574C4  |.  E8 53EBEAFF   call    0040601C  
005574C9  \\.  C3            retn  
  
大体分析到这里，作者这次用到了 RC6,RSA,和多线程验证，效果还是不错的。  
我的注册码是:25AEB2974F2057788538081FFD8FF087  
在一个窗口中输入就可以了。  
  
///////////////////////////////////////////////////////////////////////////////////////////  
注册机代码  
#include "stdafx.h"  
#include "BigInt.h"  
  
int main(int argc, char\* argv\[\])  
{  
  printf("Windows 优化大师 V7.2 破解\\n");  
  printf("By NightFox 2006.05.20\\n");  
  
  CBigInt N;  //RSA 中的 N  
  string sn="260853666017218556428756318395867010513";  
  N.Get(sn,DEC);  
  
  CBigInt E;  //RSA 中的 E  
  string se="133628944400812954421111162414689994885";  
  E.Get(se,DEC);  
    
  CBigInt D;  //RSA 中的 D  
  string sd="65537";  
  D.Get(sd,DEC);  
    
  char shenqing\[7\];  
  printf("输入申请号:");  
  scanf("%s",shenqing);  
  
  char hex\[20\];  
  sprintf(hex,"%02x%02x%02x%02x%02x%02x",shenqing\[0\],shenqing\[1\],shenqing\[2\],shenqing\[3\],shenqing\[4\],shenqing\[5\]);  
    
  CBigInt C;  
  string strc=hex;  
  strc.insert(0,'7');    //破坏作者做的手脚  
  C.Get(strc,HEX);  
    
  CBigInt M;  
  M=C.RsaTrans(D,N);    //RSA 解密计算  
  string m;  
  M.Put(m,HEX);  
  cout<<"注册码:"<<m<<endl;        
  
  return 0;  
}  
///////////////////////////////////////////////////////////////////////////////////////////  
\--------------------------------------------------------------------------------  
【破解总结】  
  
谢谢你能看到这里。  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!\[B\] 

其中有段代码并不是作者做的手脚。  
整个 RSA 代码是使用的 FGIntRSA。  
其中使用了 Procedure RSADecrypt(E : String; Var exp, modb, d\_p, d\_q, p, q : TFGInt; Var D : String);等一些基本函数。  
建议楼主看一下 FGIntRSA 的源代码。