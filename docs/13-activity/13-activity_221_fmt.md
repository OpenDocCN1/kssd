
# 看雪论坛读书月第一题 CrackMe01 分析

> 原文：[`www.pediy.com/kssd/pediy10/68438.html`](https://www.pediy.com/kssd/pediy10/68438.html)

【文章标题】：看雪论坛读书月第一题 CrackMe01 分析  
【文章作者】：hflywolf  
【作者邮箱】：6785623@qq.com  
【软件名称】：看雪论坛读书月第一题 CrackMe01  
【下载地址】：http://bbs.pediy.com/announcement.php?f=37&a=94  
【使用工具】：`OD`  
【操作平台】：`XpSp2`  
【作者声明】：俺是菜鸟，第一次写破文，如有错误之处，还望包涵和指教。

## 前言

：）第一次正儿八经写破文，感觉还满紧张滴。。文笔不太好，大侠们看了不要笑我哈，不多说了。

2008 看雪读书月第一题 CrackMe01 初探

一、简单的花指令  
二、枚举进程,查找 `explorer.exe` 进程。  
三、算法分三部份

## 正文

### 一、去简单花指令

用 `OD` 载入 `CM`

```
00402418  >/$  55            push    ebp                          ; 停在这里
00402419  |.  8BEC          mov     ebp, esp
0040241B  |.  6A FF         push    -1
0040241D  |.  68 E0354000   push    004035E0
00402422  |.  68 9E254000   push    <jmp.&MSVCRT.\_except\_handler3>   ;  SE 处理程序安装
00402427  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]
0040242D  |.  50            push    eax
0040242E  |.  64:8925 00000>mov     dword ptr fs:[0], esp
00402435  |.  83EC 68       sub     esp, 68
00402438  |.  53            push    ebx
00402439  |.  56            push    esi
0040243A  |.  57            push    edi
0040243B  |.  8965 E8       mov     dword ptr [ebp-18], esp
0040243E  |.  33DB          xor     ebx, ebx
00402440  |.  895D FC       mov     dword ptr [ebp-4], ebx
00402443  |.  6A 02         push    2
00402445  |.  FF15 B8314000 call    dword ptr [<&MSVCRT.\__set_app_ty>;  MSVCRT.__set_app_type
0040244B  |.  59            pop     ecx
0040244C  |.  830D 20424000>or      dword ptr [404220], FFFFFFFF
00402453  |.  830D 24424000>or      dword ptr [404224], FFFFFFFF
0040245A  |.  FF15 B4314000 call    dword ptr [<&MSVCRT.__p__fmode>] ;  MSVCRT.__p__fmode
```

按 `CTRL+B`，输入 `EB 05 F9 73 01 C3 FF E8 F6 FF FF FF`。查找二进制字串。将查找到的字符串用 `NOP`（`90`）填充,将九处的花指令填充后，保存后花指令就去掉了。

```
00401829   .  /EB 05         jmp     short 00401830
0040182B   $  |F9            stc
0040182C   .  |73 01         jnb     short 0040182F
0040182E   .  |C3            retn
0040182F   >  |FFE8          jmp     far eax                          ;  非法使用寄存器
00401831   ?  F6FF          idiv    bh
00401833   ?  FFFF          ???                                      ;  未知命令
```

### 二、算法分析

先运行 `CM`，输入用户名和注册码后，点击确定，弹出"非法用户！"对话框。用 `OD` 载入去掉花指令的 `CM` 程序。用查找字符串插件--》发现有"非法用户！"的字串，点击来到反汇编窗口。

```
00401DC7  |.  68 B4354000   push    004035B4                         ;  非法用户!   ；来到这，向上找到断首，在断首 F2 下断。F9
运行。
00401DCC  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401DCF  |.  E8 AE050000   call    <jmp.&MFC42.#4224>
00401DD4  |.  EB 39         jmp     short 00401E0F
00401DD6  |>  8B55 F4       mov     edx, dword ptr [ebp-C]
00401DD9  |.  52            push    edx                             ;  /src
00401DDA  |.  68 6C414000   push    0040416C                         ;  |dest = cm5.0040416C
00401DDF  |.  E8 2E060000   call    <jmp.&MSVCRT.strcpy>             ;  \\strcpy
00401DE4  |.  83C4 08       add     esp, 8
00401DE7  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00401DEA  |.  50            push    eax                             ;  /src
00401DEB  |.  68 EC404000   push    004040EC                         ;  |dest = cm5.004040EC
00401DF0  |.  E8 1D060000   call    <jmp.&MSVCRT.strcpy>             ;  \\strcpy
00401DF5  |.  83C4 08       add     esp, 8
00401DF8  |.  8965 EC       mov     dword ptr [ebp-14], esp
00401DFB  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]
00401DFE  |.  51            push    ecx
00401DFF  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
00401E02  |.  52            push    edx
00401E03  |.  8B45 EC       mov     eax, dword ptr [ebp-14]
00401E06  |.  50            push    eax
00401E07  |.  E8 A4FDFFFF   call    00401BB0
00401E0C  |.  83C4 0C       add     esp, 0C
00401E0F  |>  5F            pop     edi
00401E10  |.  5E            pop     esi
00401E11  |.  5B            pop     ebx
00401E12  |.  8BE5          mov     esp, ebp
00401E14  |.  5D            pop     ebp
00401E15  \.  C3            retn
```

输入用户名和注册码后，按确定，`OD` 就断下了。

```
00401CC0  /.  55            push    ebp                             ；OD 断在这。
00401CC1  |.  8BEC          mov     ebp, esp
00401CC3  |.  83EC 54       sub     esp, 54
00401CC6  |.  53            push    ebx
00401CC7  |.  56            push    esi
00401CC8  |.  57            push    edi
00401CC9  |.  894D FC       mov     dword ptr [ebp-4], ecx
00401CCC  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00401CCF  |.  83C0 60       add     eax, 60
00401CD2  |.  50            push    eax
00401CD3  |.  68 E9030000   push    3E9
00401CD8  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401CDB  |.  E8 B4060000   call    <jmp.&MFC42.#3092>
00401CE0  |.  8BC8          mov     ecx, eax
00401CE2  |.  E8 A7060000   call    <jmp.&MFC42.#3874>
00401CE7  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401CEA  |.  83C1 60       add     ecx, 60
00401CED  |.  E8 2E010000   call    00401E20
00401CF2  |.  50            push    eax
00401CF3  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401CF6  |.  83C1 60       add     ecx, 60
00401CF9  |.  E8 8A060000   call    <jmp.&MFC42.#2915>               ；取注册码
00401CFE  |.  8945 F8       mov     dword ptr [ebp-8], eax           ；将注册码送入 ESP-8 地址中
00401D01  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401D04  |.  83C1 64       add     ecx, 64
00401D07  |.  51            push    ecx
00401D08  |.  68 EA030000   push    3EA
00401D0D  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401D10  |.  E8 7F060000   call    <jmp.&MFC42.#3092>
00401D15  |.  8BC8          mov     ecx, eax
00401D17  |.  E8 72060000   call    <jmp.&MFC42.#3874>
00401D1C  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401D1F  |.  83C1 64       add     ecx, 64
00401D22  |.  E8 F9000000   call    00401E20
00401D27  |.  50            push    eax
00401D28  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401D2B  |.  83C1 64       add     ecx, 64
00401D2E  |.  E8 55060000   call    <jmp.&MFC42.#2915>               ；取用户名
00401D33  |.  8945 F4       mov     dword ptr [ebp-C], eax           ；将用户名送入 ESP-C 地址中
00401D36  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
00401D39  |.  52            push    edx
00401D3A  |.  E8 C1060000   call    <jmp.&MSVCRT.strlen>             ；取用户名长度
00401D3F  |.  83C4 04       add     esp, 4
00401D42  |.  85C0          test    eax, eax                         ；判断用户名是否为空
00401D44  |.  75 16         jnz     short 00401D5C                   ；不是则跳走。
00401D46  |.  6A 00         push    0
00401D48  |.  6A 00         push    0
00401D4A  |.  68 D0354000   push    004035D0                         ;  请输入用户名!
00401D4F  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
00401D52  |.  E8 2B060000   call    <jmp.&MFC42.#4224>
00401D57  |.  E9 B3000000   jmp     00401E0F
00401D5C  |>  8B45 F8       mov     eax, dword ptr [ebp-8]
00401D5F  |.  50            push    eax
00401D60  |.  E8 9B060000   call    <jmp.&MSVCRT.strlen>             ；取用注册码长度
00401D65  |.  83C4 04       add     esp, 4
00401D68  |.  85C0          test    eax, eax                         ；判断注册码是否为空
00401D6A  |.  75 16         jnz     short 00401D82                   ；不是则跳走。
00401D6C  |.  6A 00         push    0
00401D6E  |.  6A 00         push    0
00401D70  |.  68 C0354000   push    004035C0                         ;  请输入注册码!
00401D75  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
```



上面的分析可以明显的知道用户名长度是六位，用户名只能取“b”-“y”之间的字符。得到这结果后，我们 `F7` 跟入 `CALL` `401BB0`

``` 
00401D78  |.  E8 05060000   call    <jmp.&MFC42.#4224>  
00401D7D  |.  E9 8D000000   jmp     00401E0F  
00401D82  |>  C745 F0 00000>mov     dword ptr [ebp-10], 0            ；初始 EBP-10 地址的值为 0  
00401D89  |>  8B4D F4       /mov     ecx, dword ptr [ebp-C]          ；将[EBP-C]地址里的内容（即用户名）送入 ECX  
00401D8C  |.  034D F0       |add     ecx, dword ptr [ebp-10]         ；  
00401D8F  |.  0FBE11        |movsx   edx, byte ptr [ecx]             ；依次取用户名各位字符的 ASCALL 值  
00401D92  |.  85D2          |test    edx, edx                        ；如果取不到 ASCALL 值就跳走  
00401D94  |.  74 27         |je      short 00401DBD  
00401D96  |.  8B45 F4       |mov     eax, dword ptr [ebp-C]          
00401D99  |.  0345 F0       |add     eax, dword ptr [ebp-10]  
00401D9C  |.  0FBE08        |movsx   ecx, byte ptr [eax]             ；依次取用户名各位字符的 ASCALL 值送入 ECX  
00401D9F  |.  83F9 61       |cmp     ecx, 61                         ；比较 ASCALL 值与 61（a）比较  
00401DA2  |.  7E 19         |jle     short 00401DBD                  ；判断是否大于 61，不是则是“非法用户”  
00401DA4  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]  
00401DA7  |.  0355 F0       |add     edx, dword ptr [ebp-10]  
00401DAA  |.  0FBE02        |movsx   eax, byte ptr [edx]  
00401DAD  |.  83F8 7A       |cmp     eax, 7A                         ；比较 ASCALL 值与 7A（z）比较  
00401DB0  |.  7D 0B         |jge     short 00401DBD                  ；判断是否小于 7A，不是则是“非法用户”  
00401DB2  |.  8B4D F0       |mov     ecx, dword ptr [ebp-10]  
00401DB5  |.  83C1 01       |add     ecx, 1  
00401DB8  |.  894D F0       |mov     dword ptr [ebp-10], ecx  
00401DBB  |.^ EB CC        \\jmp     short 00401D89                  ；依次取用户名字符的 ASCALL 值判断。CM 算法第一部份  
00401DBD  |>  837D F0 06    cmp     dword ptr [ebp-10], 6            ；判断用户名长度是否为 6，不为 6 则是“非法用户”  
00401DC1  |.  74 13         je      short 00401DD6  
00401DC3  |.  6A 00         push    0  
00401DC5  |.  6A 00         push    0  
00401DC7  |.  68 B4354000   push    004035B4                         ;  非法用户!  
00401DCC  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]  
00401DCF  |.  E8 AE050000   call    <jmp.&MFC42.#4224>  
00401DD4  |.  EB 39         jmp     short 00401E0F  
00401DD6  |>  8B55 F4       mov     edx, dword ptr [ebp-C]           ；用户名送入 DEX  
00401DD9  |.  52            push    edx                             ;  将用户名压栈  
00401DDA  |.  68 6C414000   push    0040416C                         ;  将 40416C 地址压栈  
00401DDF  |.  E8 2E060000   call    <jmp.&MSVCRT.strcpy>             ;  将用户名压入 40416C 地址空间里去。  
00401DE4  |.  83C4 08       add     esp, 8  
00401DE7  |.  8B45 F8       mov     eax, dword ptr [ebp-8]           ；注册码送入 EAX  
00401DEA  |.  50            push    eax                             ;  将注册码压栈  
00401DEB  |.  68 EC404000   push    004040EC                         ;  将 4040EC 地址压栈  
00401DF0  |.  E8 1D060000   call    <jmp.&MSVCRT.strcpy>             ;  将注册码压入 4040EC 地址空间里去。  
00401DF5  |.  83C4 08       add     esp, 8  
00401DF8  |.  8965 EC       mov     dword ptr [ebp-14], esp  
00401DFB  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]  
00401DFE  |.  51            push    ecx  
00401DFF  |.  8B55 F4       mov     edx, dword ptr [ebp-C]  
00401E02  |.  52            push    edx  
00401E03  |.  8B45 EC       mov     eax, dword ptr [ebp-14]  
00401E06  |.  50            push    eax  
00401E07  |.  E8 A4FDFFFF   call    00401BB0                         ；关键 CALL，F7 跟入  
00401E0C  |.  83C4 0C       add     esp, 0C  
00401E0F  |>  5F            pop     edi  
00401E10  |.  5E            pop     esi  
00401E11  |.  5B            pop     ebx  
00401E12  |.  8BE5          mov     esp, ebp  
00401E14  |.  5D            pop     ebp  
00401E15  \\.  C3            retn  
```

``` 
00401BB0   $  55            push    ebp                             ；来到这。  
00401BB1   .  8BEC          mov     ebp, esp  
00401BB3   .  6A FF         push    -1  
00401BB5   .  68 F0264000   push    004026F0                         ;  给 6@;  SE 处理程序安装  
00401BBA   .  64:A1 0000000>mov     eax, dword ptr fs:[0]  
00401BC0   .  50            push    eax  
00401BC1   .  64:8925 00000>mov     dword ptr fs:[0], esp  
00401BC8   .  51            push    ecx  
00401BC9   .  83EC 50       sub     esp, 50  
00401BCC   .  53            push    ebx  
00401BCD   .  56            push    esi  
00401BCE   .  57            push    edi  
00401BCF   .  8965 F0       mov     dword ptr [ebp-10], esp  
00401BD2   .  8B45 08       mov     eax, dword ptr [ebp+8]  
00401BD5   .  83E8 10       sub     eax, 10  
00401BD8   .  8945 EC       mov     dword ptr [ebp-14], eax  
00401BDB   .  8B4D EC       mov     ecx, dword ptr [ebp-14]  
00401BDE   .  C701 F0174000 mov     dword ptr [ecx], 004017F0  
00401BE4   .  C745 E8 00000>mov     dword ptr [ebp-18], 0  
00401BEB   .  C745 E4 00000>mov     dword ptr [ebp-1C], 0  
00401BF2   .  90            nop  
00401BF3   .  90            nop  
00401BF4   .  90            nop  
00401BF5   .  90            nop  
00401BF6   .  90            nop  
00401BF7   .  90            nop  
00401BF8   .  90            nop  
00401BF9   .  90            nop  
00401BFA   .  90            nop  
00401BFB   .  90            nop  
00401BFC   .  90            nop  
00401BFD   .  90            nop  
00401BFE   >  837D E8 06    cmp     dword ptr [ebp-18], 6         ；EBP-18 地址的值与 6 比较  
00401C02   .  7D 6D         jge     short 00401C71                ；大于则跳走。  
00401C04   .  90            nop  
00401C05   .  90            nop  
00401C06   .  90            nop  
00401C07   .  90            nop  
00401C08   .  90            nop  
00401C09   .  90            nop  
00401C0A   .  90            nop  
00401C0B   .  90            nop  
00401C0C   .  90            nop  
00401C0D   .  90            nop  
00401C0E   .  90            nop  
00401C0F   .  90            nop  
00401C10   .  8B55 0C       mov     edx, dword ptr [ebp+C]       ；用户名送入 EDX  
00401C13   .  0355 E8       add     edx, dword ptr [ebp-18]       
00401C16   .  0FBE02        movsx   eax, byte ptr [edx]          ；依次取用户名字符的 ASCALL 值  
00401C19   .  8B4D 10       mov     ecx, dword ptr [ebp+10]      ；注册码送入 ECX  
00401C1C   .  034D E4       add     ecx, dword ptr [ebp-1C]  
00401C1F   .  0FBE11        movsx   edx, byte ptr [ecx]          ；依次取注册码各字符的 ASCALL 值  
00401C22   .  83C2 1B       add     edx, 1B                      ；将取得的 ASCALL 值加 1B（十进制就是 27）  
00401C25   .  3BC2          cmp     eax, edx                     ；将用户名各字符的 ASCALL 值依次与上面得的结果比较  
00401C27   .  74 05         je      short 00401C2E               ；相等则跳走。反之则跳向失败的地方  
00401C29   .  E8 82FBFFFF   call    004017B0                     ；错误 CALL  
00401C2E   >  90            nop  
00401C2F   .  90            nop  
00401C30   .  90            nop  
00401C31   .  90            nop  
00401C32   .  90            nop  
00401C33   .  90            nop  
00401C34   .  90            nop  
00401C35   .  90            nop  
00401C36   .  90            nop  
00401C37   .  90            nop  
00401C38   .  90            nop  
00401C39   .  90            nop  
00401C3A   .  8B45 0C       mov     eax, dword ptr [ebp+C]       ；用户名送入 EDX  
00401C3D   .  0345 E8       add     eax, dword ptr [ebp-18]  
00401C40   .  0FBE08        movsx   ecx, byte ptr [eax]          ；依次取用户名字符的 ASCALL 值  
00401C43   .  8B55 10       mov     edx, dword ptr [ebp+10]      ；注册码送入 ECX  
00401C46   .  0355 E4       add     edx, dword ptr [ebp-1C]  
00401C49   .  0FBE42 01     movsx   eax, byte ptr [edx+1]        ；依次取注册码各字符的 ASCALL 值  
00401C4D   .  83C0 20       add     eax, 20                      ；将取得的 ASCALL 值加 20（十进制就是 32），就是将字符转小写）  
00401C50   .  3BC8          cmp     ecx, eax                     ；将用户名各字符的 ASCALL 值依次与上面得的结果比较  
```



```asm
00401C52   .  7E 09         jle     short 00401C5D               ;不大于则跳走。反之则跳向失败的地方  
00401C54   .  8B4D EC       mov     ecx, dword ptr [ebp-14]  
00401C57   .  C701 B0174000 mov     dword ptr [ecx], 004017B0      
00401C5D   >  8B55 E4       mov     edx, dword ptr [ebp-1C]  
00401C60   .  83C2 02       add     edx, 2  
00401C63   .  8955 E4       mov     dword ptr [ebp-1C], edx  
00401C66   .  8B45 E8       mov     eax, dword ptr [ebp-18]  
00401C69   .  83C0 01       add     eax, 1  
00401C6C   .  8945 E8       mov     dword ptr [ebp-18], eax  
00401C6F   .^ EB 8D         jmp     short 00401BFE               ;循环依次判断注册码是否合法。CM 算法的第二部分  
00401C71   >  C745 FC 00000>mov     dword ptr [ebp-4], 0  
00401C78   .  90            nop  
00401C79   .  90            nop  
00401C7A   .  90            nop  
00401C7B   .  90            nop  
00401C7C   .  90            nop  
00401C7D   .  90            nop  
00401C7E   .  90            nop  
00401C7F   .  90            nop  
00401C80   .  90            nop  
00401C81   .  90            nop  
00401C82   .  90            nop  
00401C83   .  90            nop  
00401C84   .  C745 E0 9C020>mov     dword ptr [ebp-20], 29C    ;将 29C 送入 EBP-20 地址里。  
00401C8B   .  8B4D E0       mov     ecx, dword ptr [ebp-20]    ;将 29C 送入 ECX  
00401C8E   .  C601 06       mov     byte ptr [ecx], 6          ;此处产生异常。调用 seh  
00401C91   .  EB 0B         jmp     short 00401C9E              
00401C93   .  E8 58FDFFFF   call    004019F0  
00401C98   .  B8 9E1C4000   mov     eax, 00401C9E  
00401C9D   .  C3            retn  
00401C9E   >  C745 FC FFFFF>mov     dword ptr [ebp-4], -1  
00401CA5   .  8B4D F4       mov     ecx, dword ptr [ebp-C]  
00401CA8   .  64:890D 00000>mov     dword ptr fs:[0], ecx  
00401CAF   .  5F            pop     edi  
00401CB0   .  5E            pop     esi  
00401CB1   .  5B            pop     ebx  
00401CB2   .  8BE5          mov     esp, ebp  
00401CB4   .  5D            pop     ebp  
00401CB5   .  C3            retn  
```

上面的分析可以明显知道注册码长度为 12 位。注册码分为两部份。注册码的 1，3，5，7，9，11 位的 ASCALL 分别等于用户名各位 ASCALL 值减去 27。 注册码的 2，4，6，8，10，12 位的 ASCALL 值分别不小于用户名转成大写后的 ASCALL 值。

我们再来看看上面分析产生异常的地方，这里可以有两种做法，一是将其 NOP 掉后，就可以走向算法最后一部份了。二是先在 `4019F0` 地址处 F2 下断。也就是枚举进程的地方。为了很好的了解本 CM 的原理，我选择了第二种做法。直接在 `4019F0` 地址下断。

```asm
77F91BBC    8B1C24          mov     ebx, dword ptr [esp]       ;F8 跟进上面那个异常的地方后就来到这里了。  
77F91BBF    51              push    ecx  
77F91BC0    53              push    ebx  
77F91BC1    E8 EC AF0100     call    77FACBB2                   ;如果在 4019F0 上面下断，F8 执行到这里后就会断在 4019F0 上。  
77F91BC6    0AC0            or      al, al  
77F91BC8    74 0C           je      short 77F91BD6  
77F91BCA    5B              pop     ebx  
77F91BCB    59              pop     ecx  
77F91BCC    6A 00           push    0  
77F91BCE    51              push    ecx  
77F91BCF    E8 64 66FFFF     call    ZwContinue  
77F91BD4    EB 0B           jmp     short 77F91BE1  
77F91BD6    5B              pop     ebx  
77F91BD7    59              pop     ecx  
77F91BD8    6A 00           push    0  
77F91BDA    51              push    ecx  
77F91BDB    53              push    ebx  
77F91BDC    E8 7B 6EFFFF     call    ZwRaiseException  
77F91BE1    83C4 EC         add     esp, -14  
77F91BE4    890424          mov     dword ptr [esp], eax  
77F91BE7    C74424 04 01000>mov     dword ptr [esp+4], 1  
77F91BEF    895C24 08       mov     dword ptr [esp+8], ebx  
77F91BF3    C74424 10 00000>mov     dword ptr [esp+10], 0  
77F91BFB    54              push    esp  
77F91BFC    E8 9FA80100     call    RtlRaiseException  
77F91C01    C2 0800         retn    8  
```

```asm
004019F0  /$  55            push    ebp                         ;断在这。  
004019F1  |.  8BEC          mov     ebp, esp  
004019F3  |.  81EC 88050000 sub     esp, 588  
004019F9  |.  53            push    ebx  
004019FA  |.  56            push    esi  
004019FB  |.  57            push    edi  
004019FC  |.  C785 D4FEFFFF>mov     dword ptr [ebp-12C], 0  
00401A06  |.  C785 C8FEFFFF>mov     dword ptr [ebp-138], 0  
00401A10  |.  C785 CCFEFFFF>mov     dword ptr [ebp-134], 0  
00401A1A  |.  68 00040000   push    400                          ; /n = 400 (1024.)  
00401A1F  |.  6A 00         push    0                            ; |c = 00  
00401A21  |.  8D85 C4FAFFFF lea     eax, dword ptr [ebp-53C]         ; |  
00401A27  |.  50            push    eax                          ; |s  
00401A28  |.  E8 DF090000   call    <jmp.&MSVCRT.memset>             ; \\memset  
00401A2D  |.  83C4 0C       add     esp, 0C  
00401A30  |.  90            nop  
00401A31  |.  90            nop  
00401A32  |.  90            nop  
00401A33  |.  90            nop  
00401A34  |.  90            nop  
00401A35  |.  90            nop  
00401A36  |.  90            nop  
00401A37  |.  90            nop  
00401A38  |.  90            nop  
00401A39  |.  90            nop  
00401A3A  |.  90            nop  
00401A3B  |.  90            nop  
00401A3C  |.  6A 00         push    0                            ; /ProcessID = 0            ;参数（当前快照的进程 ID）  
00401A3E  |.  6A 0F         push    0F                           ; |Flags = TH32CS_SNAPALL    ;参数（全部进程和线程）  
00401A40  |.  E8 710B0000   call    <jmp.&KERNEL32.CreateToolhelp32S>; \\CreateToolhelp32Snapshot    ;为指定进程建立快照  
00401A45  |.  8985 D0FEFFFF mov     dword ptr [ebp-130], eax         ;将要快照的进程 ID 送入 DBP-130  
00401A4B  |.  C785 D8FEFFFF>mov     dword ptr [ebp-128], 128         ;将 128-》EBP-128 地址  
00401A55  |.  8D8D D8FEFFFF lea     ecx, dword ptr [ebp-128]         ;将 EBP-128 地址-》ECX  
00401A5B  |.  51            push    ecx                          ; /lppe  
00401A5C  |.  8B95 D0FEFFFF mov     edx, dword ptr [ebp-130]         ; |  
00401A62  |.  52            push    edx                          ; |hSnapshot  
00401A63  |.  E8 480B0000   call    <jmp.&KERNEL32.Process32First>   ; \\Process32First  ;建立快照后获得第一个进程的句柄  
00401A68  |.  8985 C4FEFFFF mov     dword ptr [ebp-13C], eax         ;获取成功致 EAX 为 1，将 EAX 送入 EBP-13C  
00401A6E  |.  83BD C4FEFFFF>cmp     dword ptr [ebp-13C], 0           ;将 EBP-13C 的值与 0 比较  
00401A75  |.  75 1B         jnz     short 00401A92                   ;不相等则跳。  
00401A77  |.  FF15 20304000 call    dword ptr [<&KERNEL32.GetCurrent>; [GetCurrentProcess ;获取 CM 进程的一个伪句柄  
00401A7D  |.  8985 C0FAFFFF mov     dword ptr [ebp-540], eax  
00401A83  |.  6A 00         push    0                            ; /ExitCode = 0     ;参数，（进程退出代码）  
00401A85  |.  8B85 C0FAFFFF mov     eax, dword ptr [ebp-540]         ; |                  ;进程句柄  
00401A8B  |.  50            push    eax                          ; |hProcess  
00401A8C  |.  FF15 00304000 call    dword ptr [<&KERNEL32.TerminateP>; \\TerminateProcess ;结束 CM 进程。  
00401A92  |>  C785 BCFAFFFF>mov     dword ptr [ebp-544], 0           ;初始 EBP-544 地址的值为 0  
00401A9C  |>  83BD C4FEFFFF>/cmp     dword ptr [ebp-13C], 0          ;将 EBP-13C 的值与 0 比较  
00401AA3  |.  0F84 CE000000 |je      00401B77                        ;相等则跳  
00401AA9  |.  83BD C8FEFFFF>|cmp     dword ptr [ebp-138], 0          ;将 EBP-138 的值与 0 比较  
00401AB0  |.  0F85 C1000000 |jnz     00401B77                        ;不相等则跳  
00401AB6  |.  83BD CCFEFFFF>|cmp     dword ptr [ebp-134], 0          ;将 EBP-134 的值与 0 比较  
00401ABD  |.  0F85 B4000000 |jnz     00401B77                        ;不相等则跳  
00401AC3  |.  FF15 0C304000 |call    dword ptr [<&KERNEL32.GetCurren>; [GetCurrentProcessId  ;取 CM 进程 ID  
00401AC9  |.  3985 E0FEFFFF |cmp     dword ptr [ebp-120], eax        ;将 EBP-120 与 EAX（CM 进程 ID 值）比较  
00401ACF  |.  75 28         |jnz     short 00401AF9                  ;不相等则跳走。  
00401AD1  |.  8B8D F0FEFFFF |mov     ecx, dword ptr [ebp-110]  
00401AD7  |.  898D CCFEFFFF |mov     dword ptr [ebp-134], ecx  
```



### 汇编分析

```
00401ADD  |. 8B95 F0FEFFFF |mov     edx, dword ptr [ebp-110]  
00401AE3  |. 52            |push    edx                             ; /ProcessID          ；已存在进程的 ID  
00401AE4  |. 6A 01         |push    1                               ; |Inheritable = TRUE  
00401AE6  |. 68 FF0F1F00   |push    1F0FFF                          ; |Access = PROCESS_ALL_ACCESS  
00401AEB  |. FF15 08304000 |call    dword ptr [<&KERNEL32.OpenProce>; \\OpenProcess        ；打开已存在的进程  
00401AF1  |. 8985 D4FEFFFF |mov     dword ptr [ebp-12C], eax        ；进程  
00401AF7  |. EB 24         |jmp     short 00401B1D  
00401AF9  |> 68 A4354000   |push    004035A4                        ; /explorer.exe      ；固定字符串。  
00401AFE  |. 8D85 FCFEFFFF |lea     eax, dword ptr [ebp-104]        ; |  
00401B04  |. 50            |push    eax                             ; |s1            ；当前的进程名  
00401B05  |. E8 FC080000   |call    <jmp.&MSVCRT.strcmp>            ; \\strcmp        ；固定字符串与当前进程名比较。  
00401B0A  |. 83C4 08       |add     esp, 8                          ；  
00401B0D  |. 85C0          |test    eax, eax                        ；不相等则跳  
00401B0F  |. 75 0C         |jnz     short 00401B1D  
00401B11  |. 8B8D E0FEFFFF |mov     ecx, dword ptr [ebp-120]        ；将 EBP-120 的值--》ECX  
00401B17  |. 898D C8FEFFFF |mov     dword ptr [ebp-138], ecx        ；将 ECX---》EBP-138  
00401B1D  |> 8D95 D8FEFFFF |lea     edx, dword ptr [ebp-128]        ；将 EBP-128 地址--》EDX  
00401B23  |. 52            |push    edx                             ; /lppe  
00401B24  |. 8B85 D0FEFFFF |mov     eax, dword ptr [ebp-130]        ; |  
00401B2A  |. 50            |push    eax                             ; |hSnapshot  
00401B2B  |. E8 7A0A0000   |call    <jmp.&KERNEL32.Process32Next>   ; \\Process32Next  ；取下一进程的 ID，ID 送入 EBP-120 地址  
00401B30  |. 8985 C4FEFFFF |mov     dword ptr [ebp-13C], eax        ；取下一进程成功 EAX 值为 1，不成功为 0，将 EAX--》EBP-13C  
00401B36  |. 83BD C4FEFFFF>|cmp     dword ptr [ebp-13C], 0          ；比较 EBP-13C 与 O  
00401B3D  |. 75 24         |jnz     short 00401B63                  ；不相等则跳  
00401B3F  |. 83BD BCFAFFFF>|cmp     dword ptr [ebp-544], 0          ；比较 EBP-544 与 0  
00401B46  |. 75 1B         |jnz     short 00401B63                  ；不相等跳则  
00401B48  |. FF15 20304000 |call    dword ptr [<&KERNEL32.GetCurren>; \[GetCurrentProcess  ；取 CM 进程伪句柄  
00401B4E  |. 8985 B8FAFFFF |mov     dword ptr [ebp-548], eax        ；进程句柄送 EBP-548  
00401B54  |. 6A 00         |push    0                               ; /ExitCode = 0  
00401B56  |. 8B8D B8FAFFFF |mov     ecx, dword ptr [ebp-548]        ; |                   ；进程句柄  
00401B5C  |. 51            |push    ecx                             ; |hProcess  
00401B5D  |. FF15 00304000 |call    dword ptr [<&KERNEL32.Terminate>; \\TerminateProcess  ；结束进程  
00401B63  |> 8B95 BCFAFFFF |mov     edx, dword ptr [ebp-544]        ；EBP-544 内容-》EDX  
00401B69  |. 83C2 01       |add     edx, 1                          ；EDX+1  
00401B6C  |. 8995 BCFAFFFF |mov     dword ptr [ebp-544], edx        ；EDX-》EBP-544 地址  
00401B72  |.^ E9 25FFFFFF  \\jmp     00401A9C                        ；循环枚举进程。  
00401B77  |> 90            nop  
00401B78  |. 90            nop  
00401B79  |. 90            nop  
00401B7A  |. 90            nop  
00401B7B  |. 90            nop  
00401B7C  |. 90            nop  
00401B7D  |. 90            nop  
00401B7E  |. 90            nop  
00401B7F  |. 90            nop  
00401B80  |. 90            nop  
00401B81  |. 90            nop  
00401B82  |. 90            nop  
00401B83  |. 8B85 CCFEFFFF mov     eax, dword ptr [ebp-134]         ；EBP-134 内容--》EAX  
00401B89  |. 3B85 C8FEFFFF cmp     eax, dword ptr [ebp-138]         ；EAX 与 EBP-138 内容比较  
00401B8F  |. 74 0F         je      short 00401BA0                   ；相等则跳走。  
00401B91  |. 6A 00         push    0                                ; /ExitCode = 0  
00401B93  |. 8B8D D4FEFFFF mov     ecx, dword ptr [ebp-12C]         ; |                    ；进程句柄  
00401B99  |. 51            push    ecx                              ; |hProcess  
00401B9A  |. FF15 00304000 call    dword ptr [<&KERNEL32.TerminateP>; \\TerminateProcess    ；结束进程  
00401BA0  |> 5F            pop     edi  
00401BA1  |. 5E            pop     esi  
00401BA2  |. 5B            pop     ebx  
00401BA3  |. 8BE5          mov     esp, ebp  
00401BA5  |. 5D            pop     ebp  
00401BA6  \. C3            retn                                    ；返回  
```

上面的分析可以明显的知道 `CM` 通过枚举系统当前全部进程的进程名，查找是否存在 `explorer.exe` 进程（有大小写之分,555 俺的 XPSP2 的这个进程是大写的。刚开始分析的时候走了一些弯路），如果不存在就结束进程并刷新进程。

上面执行到返回后，会来到下面这里。

```
7800925D    5F              pop     edi                           ；F8 一直下去。  
7800925E    5E              pop     esi  
7800925F    8BDD            mov     ebx, ebp  
78009261    5D              pop     ebp  
78009262    8B4D 10         mov     ecx, dword ptr [ebp+10]  
78009265    55              push    ebp  
78009266    8BEB            mov     ebp, ebx  
78009268    81F9 00010000   cmp     ecx, 100  
7800926E    75 05           jnz     short 78009275  
78009270    B9 02000000     mov     ecx, 2  
78009275    51              push    ecx  
78009276    E8 CE2A0000     call    7800BD49  
7800927B    5D              pop     ebp  
7800927C    59              pop     ecx  
7800927D    5B              pop     ebx  
7800927E    C9              leave  
7800927F    C2 0C00         retn    0C                             ；返回，一直 F8 下去。  
78006C6E    83C4 2C         add     esp, 2C                        ；F8 一直跟到这。  
78006C71    85C0            test    eax, eax  
78006C73    74 07           je      short 78006C7C  
78006C75    57              push    edi  
78006C76    50              push    eax  
78006C77    E8 92040000     call    7800710E                       ；这个 CALL，F7 跟进就可以到 CM 算法和第三部份了。  
78006C7C    5F              pop     edi  
78006C7D    5E              pop     esi  
78006C7E    5B              pop     ebx  
78006C7F    5D              pop     ebp  
78006C80    C3              retn  
```

```
004017F0   . 55            push    ebp                       ；上面的 CALL 跟进后，一直 F8 没多久就到这里。本 CM 最经典的部份。  
004017F1   . 8BEC          mov     ebp, esp  
004017F3   . 81EC 80000000 sub     esp, 80  
004017F9   . 53            push    ebx  
004017FA   . 56            push    esi  
004017FB   . 57            push    edi  
004017FC   . C745 FC 88354>mov     dword ptr [ebp-4], 00403588   ;  abcdefghijklmnopqrstuvwxy（固定串）  
00401803   . 50            push    eax  
00401804   . 33C0          xor     eax, eax  
00401806   . E8 00000000   call    0040180B  
0040180B  /$ 58            pop     eax  
0040180C  |. 57            push    edi  
0040180D  |. 8D78 04       lea     edi, dword ptr [eax+4]  
00401810  |. 58            pop     eax  
00401811  |. 5F            pop     edi  
00401812  |. 8B45 FC       mov     eax, dword ptr [ebp-4]        ；将固定字符串送入 EAX  
00401815  |. 8945 F8       mov     dword ptr [ebp-8], eax        ；EAX-》EBP-8 地址  
00401818  |. C745 D8 00000>mov     dword ptr [ebp-28], 0         ；初始化 EBP-28 的值为 0  
0040181F  |> 837D D8 18    /cmp     dword ptr [ebp-28], 18       ；比较 EBP-28 是否等于 24（16 进制是 18）  
00401823  |. 0F84 92000000 |je      004018BB                     ；相等跳走。  
00401829  |. 90            |nop  
0040182A  |. 90            |nop  
0040182B  |. 90            |nop  
0040182C  |. 90            |nop  
0040182D  |. 90            |nop  
0040182E  |. 90            |nop  
0040182F  |. 90            |nop  
00401830  |. 90            |nop  
00401831  |. 90            |nop  
00401832  |. 90            |nop  
00401833  |. 90            |nop  
00401834  |. 90            |nop  
00401835  |. 0FBE0D 6C4140>|movsx   ecx, byte ptr [40416C]          ；用户名首位字符的 ASCALL 值-》ECX  
```



```
0040183C  |.  8B55 FC       |mov     edx, dword ptr [ebp-4]          ；将固定字符串送入 EDX  
0040183F  |.  0FBE02        |movsx   eax, byte ptr [edx]             ；依次取固定字符的 ACSCALL 值-》EAX  
00401842  |.  83C0 20       |add     eax, 20                         ；EAX 的值+32  
00401845  |.  3BC8          |cmp     ecx, eax                        ；ECX 与 EAX 比较  
00401847  |.  74 21         |je      short 0040186A                  ；相等则跳  
00401849  |.  68 6C414000   |push    0040416C                        ;  用户名        
0040184E  |.  E8 AD0B0000   |call    <jmp.&MSVCRT.strlen>            ;  取用户名长度，长度值---》EAX  
00401853  |.  83C4 04       |add     esp, 4  
00401856  |.  0FBE88 6B4140>|movsx   ecx, byte ptr [eax+40416B]      ；用户最后一位字符的 ASCALL 值-》ECX  
0040185D  |.  8B55 FC       |mov     edx, dword ptr [ebp-4]          ；将固定字符串送入 EDX  
00401860  |.  0FBE02        |movsx   eax, byte ptr [edx]             ；依次取固定字符的 ACSCALL 值-》EAX  
00401863  |.  83C0 20       |add     eax, 20                         ；EAX 的值+32  
00401866  |.  3BC8          |cmp     ecx, eax                        ；ECX 与 EAX 比较  
00401868  |.  75 09         |jnz     short 00401873                  ；不相等则跳  
0040186A  |>  8B4D FC       |mov     ecx, dword ptr [ebp-4]          ；将固定字符串送入 ECX  
0040186D  |.  83C1 01       |add     ecx, 1                          ；ECX 删去前一位字符  
00401870  |.  894D FC       |mov     dword ptr [ebp-4], ecx          ；将新的字符串-》EBP-4 地址  
00401873  |>  8B55 D8       |mov     edx, dword ptr [ebp-28]         ；EBP-28 地址--》EDX  
00401876  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]          ；固定字符到 EAX  
00401879  |.  8A08          |mov     cl, byte ptr [eax]              ；依次取固定字符串指定位置的值送到 CL  
0040187B  |.  884C15 DC     |mov     byte ptr [ebp+edx-24], cl       ；保存字符，作用是得到一个字符串，算法计算要用到。  
0040187F  |.  8B55 D8       |mov     edx, dword ptr [ebp-28]         ；EBP-28 内容-》到 EDX  
00401882  |.  83C2 01       |add     edx, 1                          ；EDX+1  
00401885  |.  8955 D8       |mov     dword ptr [ebp-28], edx         ；EDX-》EBP-28 地址  
00401888  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]          ；固定字符串送入 EAX  
0040188B  |.  83C0 02       |add     eax, 2                          ；将固定字符串删去前两位。  
0040188E  |.  8945 FC       |mov     dword ptr [ebp-4], eax          ；将新的字符串-》EBP-4 地址  
00401891  |.  90            |nop  
00401892  |.  90            |nop  
00401893  |.  90            |nop  
00401894  |.  90            |nop  
00401895  |.  90            |nop  
00401896  |.  90            |nop  
00401897  |.  90            |nop  
00401898  |.  90            |nop  
00401899  |.  90            |nop  
0040189A  |.  90            |nop  
0040189B  |.  90            |nop  
0040189C  |.  90            |nop  
0040189D  |.  8B4D FC       |mov     ecx, dword ptr [ebp-4]         ；将新的字符串-》ECX  
004018A0  |.  0FBE11        |movsx   edx, byte ptr [ecx]            ；取第一位字符串  
004018A3  |.  85D2          |test    edx, edx                       ；判断是否取成功  
004018A5  |.  75 0F         |jnz     short 004018B6                 ；成功就跳走。  
004018A7  |.  837D D8 18    |cmp     dword ptr [ebp-28], 18         ；比较 EBP-28 内容与 18  
004018AB  |.  7D 09         |jge     short 004018B6                 ；小于 18 就不跳  
004018AD  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]         ；将固定字符串---》EAX  
004018B0  |.  83C0 01       |add     eax, 1                         ；删去固定字符串前一位。  
004018B3  |.  8945 FC       |mov     dword ptr [ebp-4], eax         ；将新的字符串送 EBP-4 地址。  
004018B6  |>^ E9 64FFFFFF  \\jmp     0040181F                       ；循环生成一个字符串做为算法计算的一个 KEY  
004018BB  |>  6A 00         push    0                                ;  
004018BD  |.  68 60354000   push    00403560                         ;  
004018C2  |.  68 54354000   push    00403554                         ;  
004018C7  |.  0FBE0D F64040>movsx   ecx, byte ptr [4040F6]           ;  取注册码的 11 位字符  
004018CE  |.  83E9 55       sub     ecx, 55                          ;  将 11 位的字符的 ASCALL 值与 85（16 进制 55）比较  
004018D1  |.  F7D9          neg     ecx                              ;  ECX 求补  
004018D3  |.  1BC9          sbb     ecx, ecx                         ;  有符号相减  
004018D5  |.  41            inc     ecx                              ;  ECX+1  
004018D6  |.  51            push    ecx                              ;  将 ECX 压栈  
004018D7  |.  FF15 00324000 call    dword ptr [<&USER32.MessageBoxA>>;  如果 11 位字符 ASCALL 值等于 85，不弹出对话框。  
004018DD  |.  8945 D8       mov     dword ptr [ebp-28], eax          ；不弹出对话框 EAX 变就为 O，反之为 1  
004018E0  |.  837D D8 00    cmp     dword ptr [ebp-28], 0            ；比较 EBP-28 是否为 0  
004018E4  |.  74 15         je      short 004018FB                   ；相等则跳  
004018E6  |.  FF15 20304000 call    dword ptr [<&KERNEL32.GetCurrent>;  \[GetCurrentProcess  ；取 CM 进程伪句柄  
004018EC  |.  8945 D4       mov     dword ptr [ebp-2C], eax  
004018EF  |.  6A 00         push    0                                ;  /ExitCode = 0       ；句柄  
004018F1  |.  8B55 D4       mov     edx, dword ptr [ebp-2C]          ;  |                    
004018F4  |.  52            push    edx                             ;  |hProcess  
004018F5  |.  FF15 00304000 call    dword ptr [<&KERNEL32.TerminateP>;  \\TerminateProcess   ；结束进程  
004018FB  |>  C745 D8 00000>mov     dword ptr [ebp-28], 0            ；初始化 EBP-28 的值为 0  
00401902  |.  C745 D0 05000>mov     dword ptr [ebp-30], 5            ；初始化 EBP-30 的值为 5  
00401909  |.  C745 CC 00000>mov     dword ptr [ebp-34], 0            ；初始化 EBP-34 的值为 0  
00401910  |>  837D D8 0C    /cmp     dword ptr [ebp-28], 0C          ；EBP-28 的值是否等于 12  
00401914  |.  0F84 BB000000 |je      004019D5                        ；相等则跳走。  
0040191A  |.  8B45 D0       |mov     eax, dword ptr [ebp-30]         ；5--》EAX  
0040191D  |.  8D0C85 FCFFFF>|lea     ecx, dword ptr [eax*4-4]        ；ECX=EAX*4-4  
00401924  |.  894D C8       |mov     dword ptr [ebp-38], ecx         ；ECX-》EBP-38 地址  
00401927  |>  8B55 D8       |/mov     edx, dword ptr [ebp-28]        ；EBP-28 的值-EDX  
0040192A  |.  0FBE82 EC4040>||movsx   eax, byte ptr [edx+4040EC]     ；依次取注册码各位字符的 ASCALL 值  
00401931  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；EBP-38 的值-》ECX  
00401934  |.  0FBE540D DC   ||movsx   edx, byte ptr [ebp+ecx-24]     ；依次取上面得到的 KEY 指定位置上的字符的 ASCALL 值  
00401939  |.  8B4D C8       ||mov     ecx, dword ptr [ebp-38]        ；将 EBP-38 的值-》ECX  
0040193C  |.  83C1 01       ||add     ecx, 1                         ；ECX+1  
0040193F  |.  894D C8       ||mov     dword ptr [ebp-38], ecx        ；ECX-》EBP-38 地址  
00401942  |.  3BC2          ||cmp     eax, edx                       ；注册码各位的字符与 KEY 指定位置上的字符比较  
00401944  |.  74 13         ||je      short 00401959                 ；相等就跳走  
00401946  |.  8B55 CC       ||mov     edx, dword ptr [ebp-34]        ；EBP-34 的值-》EDX  
00401949  |.  83C2 01       ||add     edx, 1                         ；EDX+1  
0040194C  |.  8955 CC       ||mov     dword ptr [ebp-34], edx        ；EDX-》EBP-34 地址  
0040194F  |.  837D CC 04    ||cmp     dword ptr [ebp-34], 4          ；EBP-34 的值与 4 比较  
00401953  |.  7E 02         ||jle     short 00401957                 ；小于等于则跳。  
00401955  |.  EB 02         ||jmp     short 00401959                 ；跳走  
00401957  |>^ EB CE         |\\jmp     short 00401927                 ；循环判断输入的注册码是否合法。  
00401959  |>  837D CC 05    |cmp     dword ptr [ebp-34], 5           ；EBP-34 的值与 5 比较  
0040195D  |.  75 3F         |jnz     short 0040199E                 ；不等于则跳，反之就走向失败的地方。  
0040195F  |.  C745 C4 78354>|mov     dword ptr [ebp-3C], 00403578    ;  abcdefghijklmn  
00401966  |.  C745 C0 68354>|mov     dword ptr [ebp-40], 00403568    ;  opqrstuvwxyz  
0040196D  |.  50            |push    eax  
0040196E  |.  8D85 C4FFFFFF |lea     eax, dword ptr [ebp-3C]  
00401974  |.  50            |push    eax  
```



### 汇编代码片段

```
00401975  |.  FF75 C0       |push    dword ptr [ebp-40]  
00401978  |.  58            |pop     eax  
00401979  |.  58            |pop     eax  
0040197A  |.  58            |pop     eax  
0040197B  |.  8945 CC       |mov     dword ptr [ebp-34], eax  
0040197E  |.  837D CC 00    |cmp     dword ptr [ebp-34], 0  
00401982  |.  7E 07         |jle     short 0040198B  
00401984  |.  E8 27FEFFFF   |call    004017B0                       ；错误 CALL  
00401989  |.  EB 0E         |jmp     short 00401999  
0040198B  |>  50            |push    eax  
0040198C  |.  8D85 C4FFFFFF |lea     eax, dword ptr [ebp-3C]  
00401992  |.  50            |push    eax  
00401993  |.  FF75 C0       |push    dword ptr [ebp-40]  
00401996  |.  58            |pop     eax  
00401997  |.  58            |pop     eax  
00401998  |.  58            |pop     eax  
00401999  |>  E8 12FEFFFF   |call    004017B0                      ；错误 CALL  
0040199E  |>  8B45 D8       |mov     eax, dword ptr [ebp-28]  
004019A1  |.  83C0 02       |add     eax, 2  
004019A4  |.  8945 D8       |mov     dword ptr [ebp-28], eax  
004019A7  |.  8B4D D0       |mov     ecx, dword ptr [ebp-30]  
004019AA  |.  83E9 01       |sub     ecx, 1  
004019AD  |.  894D D0       |mov     dword ptr [ebp-30], ecx  
004019B0  |.  90            |nop  
004019B1  |.  90            |nop  
004019B2  |.  90            |nop  
004019B3  |.  90            |nop  
004019B4  |.  90            |nop  
004019B5  |.  90            |nop  
004019B6  |.  90            |nop  
004019B7  |.  90            |nop  
004019B8  |.  90            |nop  
004019B9  |.  90            |nop  
004019BA  |.  90            |nop  
004019BB  |.  90            |nop  
004019BC  |.  837D D0 00    |cmp     dword ptr [ebp-30], 0           ；EBP-30 的值与 0 比较  
004019C0  |.  75 07         |jnz     short 004019C9                  ；不相等则跳走  
004019C2  |.  C745 D0 0600  |mov     dword ptr [ebp-30], 6           ；6-》EBP-30 地址  
004019C9  |>  C745 CC 0000  |mov     dword ptr [ebp-34], 0           ；0-》EBP-34 地址  
004019D0  |.^ E9 3BFFFFFF   |jmp     00401910                        ；循环判断  
004019D5  |>  E8 96FDFFFF   |call    00401770                         ；成功 CALL  
004019DA  |.  5F            |pop     edi  
004019DB  |.  5E            |pop     esi  
004019DC  |.  5B            |pop     ebx  
004019DD  |.  8BE5          |mov     esp, ebp  
004019DF  |.  5D            |pop     ebp  
004019E0  |\ .  C3            |retn  
```

### 破解总结

上面的分析可以明显的知道这里算法的原理。大至可以概括如下：

#### 一、用户名规则
用户名长度为 6 位，用户名只能取“b”-“y”之间的字符。

#### 二、注册码规则
注册码长度为 12 位，其要分为两部份。
a）注册码的 1，3，5，7，9，11 位的 ASCALL 分别等于用户名各位 ASCALL 值减去 27。
b）注册码的 2，4，6，8，10，12 位的 ASCALL 值分别不小于用户名转成大写后的 ASCALL 值

#### 三、固定字符串运算
固定字符串 `ABCDEFGHIJKLMNOPQRSTUVWXY` 的运算由用户名的第一位和最后一位来决定。

（一）将用户的第一位和最后一位的字符转成大写分别与固定字符串各位的字符依次比较。
1）如果不相等
那就依次截取固定字符串前一个字符连接一个新的变量里。然后删去固定字符串的前两位字符
再进行同样的比较，以此循环，直到固定字符串比较完。再将原固定字符串删去前一位字符，
再进行同样的比较，以此循环。
2）如果相等
那将固定字符串先删去相等的字符，再依次截取前一个字符连接到变理里。后面的和上面的一样。

（二）最后产生一个 `KEY`。

#### 四、用户名末位固定
用户名的最后一位是固定的。根据注册码 11 位的 ASCALL 值是 85，知道字符是大写的"U"。再根据算法第二部份可以知道用户名最后一
位的字符是 85+27=112，即小写的"p"

#### 五、注册码与 KEY 的对应关系
注册码各位字符是根据固定字符串 `ABCDEFGHIJKLMNOPQRSTUVWXY` 经过运算后得到新的字符串 `KEY` 的特定位置来定的。

具体如下：
a）注册码的第一位字符由新字符串的 17-20 位来决定。
b) 注册码第三位字符由新字符串的 13-16 位来决定。
c) 注册码第五位字符由新字符串的 9-12 位来决定。
d) 注册码第七位字符由新字符串的 5-8 位来决定。
e) 注册码第九位字符由新字符串的 1-4 位来决定。
f) 注册码第十一位字符由新字符串的 21-24 位来决定。

#### 六、算法结论
根据以上的结论。算法可以这样理解。

用户名只能由“b”-“y"之间的字符组成，用户名最后一位只能为“p”,根据三的结论我们可以得到这样的一些 KEY

用户名是:(用程序枚举出来的)

b****p KEY(ACEGIKMOQSUWYCEG IKMO QSUW)
c****p KEY(ADFHJLNQSUWYBDFH JLNQ SUWY)
d****p KEY(ACEGIKMOQSUWYBEG IKMO QSUW)
e****p KEY(ACFHJLNQSUWYBDFH JLNQ SUWY)
f****p KEY(ACEGIKMOQSUWYBDG IKMO QSUW)
g****p KEY(ACEHJLNQSUWYBDFH JLNQ SUWY)
h****p KEY(ACEGIKMOQSUWYBDF IKMO QSUW)
i****p KEY(ACEGJLNQSUWYBDFH JLNQ SUWY)
j****p KEY(ACEGIKMOQSUWYBDF HKMO QSUW)
k****p KEY(ACEGILNQSUWYBDFH JLNQ SUWY)
l****p KEY(ACEGIKMOQSUWYBDF HJMO QSUW)
m****p KEY(ACEGIKNQSUWYBDFH JLNQ SUWY)
n****p KEY(ACEGIKMOQSUWYBDF HJLO QSUW)
o****p KEY(ACEGIKMPRTVXBDFH JLNQ SUWY)
p****p KEY(ACEGIKMOQSUWYBDF HJLN QSUW)
q****p KEY(ACEGIKMORTVXBDFH JLNQ SUWY)
r****p KEY(ACEGIKMOQSUWYBDF HJLN QSUW)
s****p KEY(ACEGIKMOQTVXBDFH JLNQ TVXB)
t****p KEY(ACEGIKMOQSUWYBDF HJLN QSUW)
u****p KEY(ACEGIKMOQSVXBDFH JLNQ SVXB)
v****p KEY(ACEGIKMOQSUWYBDF HJLN QSUW)
w****p KEY(ACEGIKMOQSUXBDFH JLNQ SUXB)
x****p KEY(ACEGIKMOQSUWYBDF HJLN QSUW)
y****p KEY(ACEGIKMOQSUWB0DF HJLN QSUW)

再根据结论五用户名的第一位是由 KEY 的 17-20 位来决定的。根据上面枚举出来的 KEY，发现 KEY 的 17-20 位只有以下的字符
（H I J K L M N O Q）组成，根据结论二得出只有（I，J，K ，L，M，N，O）这七个字符是符合要求的。所以终上所述

算法最终可以描述成如下：
'a）、用户名第一位只能是 7 个字符“d”，“e”，“f”，“g”，“h”，“i”，“j”
'b）、用户名第二位根据第一位的字符来决定
当第一位字符是“d”和“f”的时候，第二位只能是“b”和“t”
当第一位字符是“e”，“g”，“i”的时候，第二位只能是“c”
当第一位字符是“h”，“j”的时候，第二位只能是“t”
'c）、用户名第三位根据第一位的字符来决定
当第一位字符是“d”，“f”，“h”，“j”的时候，第二位只能是“l”，“n”，“p”，“r”
当第一位字符是“e”，“g”，“i”，的时候，第二位只能是“n”，“p”，“r”，“t”
'd）、用户名第四位根据第一位的字符来决定
当第一位字符是“d”，“f”，“h”，“j”的时候，第二位只能是“d”，“f”，“h”，“j”
当第一位字符是“e”，“g”，“i”，的时候，第二位只能是“e”，“g”，“i”，“l”
'e）、用户名第四位根据第一位的字符来决定
当第一位字符是“d”，“f”，“h”，“i”，“j”的时候，第二位只能是“b”
当第一位字符是“e”，“g”的时候，第二位只能是“c”
'f）、用户名第六位是固定字符“p”

PS:嘿嘿,不好意思,今天才有时间写下破文.不知道晚了没?555 第一次写破文好烂哦..大家不要笑哈.

### 版权声明
本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整，谢谢！

### 附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || KXCmkeygen.rar |
