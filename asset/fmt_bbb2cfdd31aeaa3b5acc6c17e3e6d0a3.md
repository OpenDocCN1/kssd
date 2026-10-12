
# exploitme 溢出分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-365.htm`](https://www.pediy.com/kssd/pediy09/pediy09-365.htm)

> **试题 A 分析**PE 分析：

我们运行 `exploit_me_A.exe`，发现它开了 `7777` 端口，于是用 `telnet` 连接该端口，随便敲入字母，`exploit_me_A.exe` 将我们键入的字母打印了出来。

该程序功能就是：接收客户端传来的字符串并将其打印出来。

漏洞描述：

用 `od` 载入 `exploit_me_A.exe`，来到程序入口主要代码如下：

```
004010B0  /$  81EC B4030000 sub     esp, 3B4
004010B6  |.  8D8424 240200>lea     eax, dword ptr [esp+224]
004010BD  |.  55            push    ebp
004010BE  |.  50            push    eax                         ; /pWSAData
004010BF  |.  68 01010000   push    101                         ; |RequestedVersion = 101 (1.1.)
004010C4  |.  FF15 BC804000 call    dword ptr [<&WS2_32.#115>]   ; \\WSAStartup
004010CA  |.  6A 00         push    0                           ; /Protocol = IPPROTO_IP
004010CC  |.  6A 01         push    1                           ; |Type = SOCK_STREAM
004010CE  |.  6A 02         push    2                           ; |Family = AF_INET
004010D0  |.  FF15 C0804000 call    dword ptr [<&WS2_32.#23>]    ; \\socket
004010D6  |.  8BE8          mov     ebp, eax
004010D8  |.  85ED          test    ebp, ebp
004010DA  |.  7D 33         jge     short 0040110F
004010DC  |.  68 00914000   push    00409100                    ;  ASCII "socket creating error!"
004010E1  |.  55            push    ebp                         ; /Arg1
004010E2  |.  B9 689A4000   mov     ecx, 00409A68               ; |
004010E7  |.  E8 0C070000   call    004017F8                    ; \\exploit_.004017F8
004010EC  |.  8BC8          mov     ecx, eax
004010EE  |.  E8 34030000   call    00401427
004010F3  |.  68 D0124000   push    004012D0
004010F8  |.  6A 0A         push    0A                          ; /Arg1 = 0000000A
004010FA  |.  8BC8          mov     ecx, eax                    ; |
004010FC  |.  E8 DF010000   call    004012E0                    ; \\exploit_.004012E0
00401101  |.  8BC8          mov     ecx, eax
00401103  |.  E8 A8010000   call    004012B0
00401108  |.  6A 01         push    1
0040110A  |.  E8 430E0000   call    00401F52
0040110F  |>  68 611E0000   push    1E61                        ; /NetShort = 1E61
00401114  |.  66:C74424 0C >mov     word ptr [esp+C], 2         ; |
0040111B  |.  FF15 C4804000 call    dword ptr [<&WS2_32.#9>]     ; \\ntohs
00401121  |.  6A 00         push    0                           ; /NetLong = 0
00401123  |.  66:894424 0E  mov     word ptr [esp+E], ax        ; |
00401128  |.  FF15 C8804000 call    dword ptr [<&WS2_32.#8>]     ; \\ntohl
0040112E  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]
00401132  |.  6A 10         push    10                          ; /AddrLen = 10 (16.)
00401134  |.  51            push    ecx                         ; |pSockAddr
00401135  |.  55            push    ebp                         ; |Socket
00401136  |.  894424 18     mov     dword ptr [esp+18], eax     ; |
0040113A  |.  FF15 CC804000 call    dword ptr [<&WS2_32.#2>]     ; \\bind
00401140  |.  85C0          test    eax, eax
00401142  |.  74 24         je      short 00401168
00401144  |.  68 E0904000   push    004090E0                    ;  ASCII "binding stream socket error!"
00401149  |.  B9 689A4000   mov     ecx, 00409A68
0040114E  |.  E8 D4020000   call    00401427
00401153  |.  68 D0124000   push    004012D0
00401158  |.  6A 0A         push    0A                          ; /Arg1 = 0000000A
0040115A  |.  8BC8          mov     ecx, eax                    ; |
0040115C  |.  E8 7F010000   call    004012E0                    ; \\exploit_.004012E0
00401161  |.  8BC8          mov     ecx, eax
00401163  |.  E8 48010000   call    004012B0
00401168  |>  53            push    ebx
00401169  |.  68 B8904000   push    004090B8                    ;  ASCII "********************************"
0040116E  |.  B9 689A4000   mov     ecx, 00409A68
00401173  |.  E8 AF020000   call    00401427
00401178  |.  68 D0124000   push    004012D0
0040117D  |.  6A 0A         push    0A                          ; /Arg1 = 0000000A
0040117F  |.  8BC8          mov     ecx, eax                    ; |
00401181  |.  E8 5A010000   call    004012E0                    ; \\exploit_.004012E0
00401186  |.  8BC8          mov     ecx, eax
00401188  |.  E8 23010000   call    004012B0
0040118D  |.  68 94904000   push    00409094                    ;  ASCII "     exploit target server 1.0",TAB,"   "
00401192  |.  B9 689A4000   mov     ecx, 00409A68
00401197  |.  E8 8B020000   call    00401427
0040119C  |.  68 D0124000   push    004012D0
004011A1  |.  6A 0A         push    0A                          ; /Arg1 = 0000000A
004011A3  |.  8BC8          mov     ecx, eax                    ; |
004011A5  |.  E8 36010000   call    004012E0                    ; \\exploit_.004012E0
004011AA  |.  8BC8          mov     ecx, eax
004011AC  |.  E8 FF000000   call    004012B0
004011B1  |.  68 B8904000   push    004090B8                    ;  ASCII "********************************"
004011B6  |.  B9 689A4000   mov     ecx, 00409A68
004011BB  |.  E8 67020000   call    00401427
004011C0  |.  68 D0124000   push    004012D0
004011C5  |.  6A 0A         push    0A                          ; /Arg1 = 0000000A
004011C7  |.  8BC8          mov     ecx, eax                    ; |
004011C9  |.  E8 12010000   call    004012E0                    ; \\exploit_.004012E0
004011CE  |.  8BC8          mov     ecx, eax
004011D0  |.  E8 DB000000   call    004012B0
004011D5  |.  6A 04         push    4                           ; /Backlog = 4
004011D7  |.  55            push    ebp                         ; |Socket
004011D8  |.  FF15 D0804000 call    dword ptr [<&WS2_32.#13>]    ; \\listen
004011DE  |.  8D5424 08     lea     edx, dword ptr [esp+8]
004011E2  |.  8D4424 1C     lea     eax, dword ptr [esp+1C]
004011E6  |.  52            push    edx                         ; /pAddrLen
004011E7  |.  50            push    eax                         ; |pSockAddr
004011E8  |.  55            push    ebp                         ; |Socket
004011E9  |.  C74424 14 100>mov     dword ptr [esp+14], 10      ; |
004011F1  |.  FF15 D4804000 call    dword ptr [<&WS2_32.#1>]     ; \\accept
004011F7  |.  8BD8          mov     ebx, eax
004011F9  |.  83FB FF       cmp     ebx, -1
004011FC  |.  74 7F         je      short 0040127D
004011FE  |.  56            push    esi
004011FF  |.  57            push    edi
00401200  |>  B9 80000000   /mov     ecx, 80
00401205  |.  33C0          |xor     eax, eax
00401207  |.  8D7C24 34     |lea     edi, dword ptr [esp+34]
0040120B  |.  50            |push    eax                        ; /Flags => 0
0040120C  |.  F3:AB         |rep     stos dword ptr es:[edi]    ; |
0040120E  |.  8D4C24 38     |lea     ecx, dword ptr [esp+38]    ; |
00401212  |.  68 00020000   |push    200                        ; |BufSize = 200 (512.)
00401217  |.  51            |push    ecx                        ; |Buffer
00401218  |.  53            |push    ebx                        ; |Socket
00401219  |.  FF15 D8804000 |call    dword ptr [<&WS2_32.#16>]   ; \\recv
0040121F  |.  8BF0          |mov     esi, eax
00401221  |.  85F6          |test    esi, esi
00401223  |.  7D 26         |jge     short 0040124B
00401225  |.  68 74904000   |push    00409074                   ;  ASCII "reading stream message erro!"
0040122A  |.  B9 689A4000   |mov     ecx, 00409A68
```
