
# 一个木马分析，请大家多多指教，提高分析

> 原文：[`www.pediy.com/kssd/pediy12/141472.html`](https://www.pediy.com/kssd/pediy12/141472.html)

脱壳后：

```
10015DE8           8BFF               mov edi, edi   //OEP 入口点

10015DEA           55                 push ebp

10015DEB           8BEC               mov ebp, esp

10015DED           837D 0C 01         cmp dword ptr [ebp+C], 1

10015DF1           75 05              jnz short 10015DF8

10015DF3           E8 56780000        call 1001D64E

10015DF8           FF75 08            push dword ptr [ebp+8]

10015DFB           8B4D 10            mov ecx, dword ptr [ebp+10]

10015DFE           8B55 0C            mov edx, dword ptr [ebp+C]

10015E01           E8 ECFEFFFF        call 10015CF2          关键 call

10015E06           59                 pop ecx

10015E07           5D                 pop ebp

10015E08           C2 0C00            retn 0C

10009916           FF75 08            push dword ptr [ebp+8]

10009919           FF15 A4200210      call dword ptr [100220A4]                         ; kernel32.DisableThreadLibraryCalls

1000991F           6A 37              push 37

10009921           68 B4250210        push 100225B4                                          ; ASCII "LpkEditControl"

10009926           E8 1C81FFFF        call 10001A47       LpkEditControl 是一个数组，里面存放了一些函数，构造 lpk

通过劫持系统文件 lpk.dll 的方法，使游戏优先加载游戏目录下的病毒文件 lpk.dll

100019F4           E8 37030100        call 10011D30   //调用 lpk

100019F9           83C4 0C            add esp, 0C

100019FC           56                 push esi

100019FD           8D85 F8FEFFFF      lea eax, dword ptr [ebp-108]

10001A03           50                 push eax

10001A04           FF15 48200210      call dword ptr [10022048]                         ; kernel32.GetSystemDirectoryA

10001A0A           68 54250210        push 10022554                                          ; ASCII "\\lpk.dll"

10001A0F           8D85 F8FEFFFF      lea eax, dword ptr [ebp-108]

10001A15           50                 push eax

10001A16           FF15 44200210      call dword ptr [10022044]                         ; kernel32.lstrcatA

10001A1C           833D 1C170310 00   cmp dword ptr [1003171C], 0

10001A23           5E                 pop esi

10001A24           75 12              jnz short 10001A38

10001A26           8D85 F8FEFFFF      lea eax, dword ptr [ebp-108]

10001A2C           50                 push eax

10001A2D           FF15 38200210      call dword ptr [10022038]                         ; kernel32.LoadLibraryA

加载文件目录下的 usp10 和 lpk

10009A4A           68 B03B0210        push 10023BB0                                          ; ASCII "%s\\Thumbs.db"

10009A4F           68 18160310        push 10031618                                          ; ASCII "e:\\ollyice\\Thumbs.db"

10009A54           E8 6D9D0000        call 100137C6                                          ; 连接本路径和此文件名

10009A59           53                 push ebx

10009A5A           68 A03B0210        push 10023BA0                                          ; ASCII "%s\\Usp10.dll"

10009A5F           BF C0110310        mov edi, 100311C0                                      ; ASCII "e:\\ollyice\\Usp10.dll"

10009A64           57                 push edi

10009A65           E8 5C9D0000        call 100137C6

10009A6A           53                 push ebx

10009A6B           68 402F0210        push 10022F40                                          ; ASCII "%s\\Lpk.dll"

10009A70           56                 push esi

10009A71           E8 509D0000        call 100137C6

10009A76           83C4 24            add esp, 24

10009A79           FF15 5C200210      call dword ptr [1002205C]                         ; kernel32.GetTickCount

10009A7F           53                 push ebx

10009A80           A8 01              test al, 1

10009A82           75 12              jnz short 10009A96

10009A84           68 402F0210        push 10022F40                                          ; ASCII "%s\\Lpk.dll"

10009A89           BB A0C00210        mov ebx, 1002C0A0                                      ; ASCII "e:\\ollyice\\Usp10.dll"

10009A8E           53                 push ebx

10009A8F           E8 329D0000        call 100137C6

10009A94           EB 14              jmp short 10009AAA

10009A96           68 A03B0210        push 10023BA0                                          ; ASCII "%s\\Usp10.dll"

10009A9B           68 A0C00210        push 1002C0A0                                          ; ASCII "e:\\ollyice\\Usp10.dll"

10009AA0           E8 219D0000        call 100137C6

10009AA5           BB A0C00210        mov ebx, 1002C0A0                                      ; ASCII "e:\\ollyice\\Usp10.dll"

10009AAA           83C4 0C            add esp, 0C

10009AAD           57                 push edi

10009AAE           FF15 84200210      call dword ptr [10022084]                         ; kernel32.GetFileAttributesA

创建一个线程函数 1，地址 10009678

10009B0D           E8 97B10000        call 10014CA9

10009B12           59                 pop ecx

10009B13           59                 pop ecx

10009B14           33C0               xor eax, eax

10009B16           50                 push eax

10009B17           50                 push eax

10009B18           50                 push eax

10009B19           68 78960010        push 10009678        线程函数 1

10009B1E           50                 push eax

10009B1F           50                 push eax

10009B20           FF15 94200210      call dword ptr [10022094]                         ; kernel32.CreateThread

线程函数 1：

10009678   /.  55                push ebp                                                   ;  线程函数 1

10009679   |.  8BEC              mov ebp, esp

1000967B   |.  81EC 94010000    sub esp, 194

10009681   |.  A1 60B00210      mov eax, dword ptr [1002B060]

10009686   |.  33C5              xor eax, ebp

10009688   |.  8945 FC           mov [local.1], eax

1000968B   |.  53                push ebx

1000968C   |.  56                push esi

1000968D   |.  57                push edi

1000968E   |.  E8 657DFFFF       call 100013F8          ;  动态加载函数，获取用到的函数的地址 关键 call 1

10009693   |.  8D85 6CFEFFFF     lea eax, [local.101]

10009699   |.  50                push eax

1000969A   |.  33F6              xor esi, esi

1000969C   |.  68 02020000      push 202

100096A1   |.  8935 78170310    mov dword ptr [10031778], esi

100096A7   |.  8935 74170310    mov dword ptr [10031774], esi

100096AD   |.  FF15 F4F70210     call dword ptr [1002F7F4]                                 ;  ws2_32.WSAStartup

100096B3   |.  BB 00010000      mov ebx, 100

100096B8   |.  53                push ebx

100096B9   |.  56                push esi

100096BA   |.  BF E0C10210      mov edi, 1002C1E0                                          ;  ASCII "c06a104c7ee76da815861809b2cf7925"

100096BF   |.  57                push edi

100096C0   |.  E8 6B860000      call <清理一块内存空间出来>

100096C5   |.  53                push ebx

100096C6   |.  56                push esi

100096C7   |.  BB F8120310      mov ebx, 100312F8                                          ;  ASCII "d41d8cd98f00b204e9800998ecf8427e"

100096CC   |.  53                push ebx

100096CD   |.  E8 5E860000      call <清理一块内存空间出来>

100096D2   |.  68 04010000      push 104

100096D7   |.  56                push esi

100096D8   |.  68 78DC0210      push 1002DC78

100096DD   |.  E8 4E860000      call <清理一块内存空间出来>

100096E2   |.  57                push edi

100096E3   |.  68 50C50210      push 1002C550                                              ;  ASCII "d:\\tools\\ollyice\\loaddll.exe"

100096E8   |.  E8 EA8FFFFF      call <计算出一个 32 位的字符串>

100096ED   |.  53                push ebx

100096EE   |.  68 A0C00210      push 1002C0A0                                              ;  ASCII "d:\\tools\\ollyice\\Lpk.dll"

100096F3   |.  E8 DF8FFFFF      call <计算出一个 32 位的字符串>                             ;  计算"d:\\tools\\ollyice\\Lpk.dll"对应的 32 位字符

100096F8   |.  68 603B0210      push 10023B60                                              ;  d41d8cd98f00b204e9800998ecf8427e
```
