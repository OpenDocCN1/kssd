
# [原创]刚看到的，刚下的，刚写的破文－Nice PDF Creato

一个制作PDF文件的好程序！！！

试用期14天。

本软件是在看雪论坛里被推荐使用的。

下载地址：`91bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2L8Y4A6A6k6Y4W2Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3@1k6s2L8#2)9K6c8q4y4Z5L8%4N6Q4x3@1q4Q4x3@1q4x3K9i4y4@1i4K6t1$3d9f1c8Q4x3@1b7&6x3e0f1^5i4K6t1$3c8r3!0%4L8W2)9K6c8o6q4Q4x3U0k6x3i4K6y4p5j5$3^5`.

主程序安装在：`C:\WINDOWS\system32\nicepdf.exe`

本汉化版不是破解版。

决定亲自动手。

汉化版未加壳。

Borland Delphi 6.0 - 7.0

如果不填任何信息，就弹出： 注册信息不完整！！

于是在`C32Asm.exe`中可以找到：

``` 
::004AFD9C::  E8 1360FBFF              CALL 00465DB4                           \:JMPUP

::004AFDA1::  837D F4 00               CMP DWORD PTR [EBP-C],0                      －－－－－这里就应该是检查注册名，可以在这里下断。

::004AFDA5::  74 14                    JE SHORT 004AFDBB                       \:JMPDOWN

::004AFDA7::  8D55 F0                  LEA EDX,[EBP-10]

::004AFDAA::  8B87 84060000            MOV EAX,[EDI+684]

::004AFDB0::  E8 FF5FFBFF              CALL 00465DB4                           \:JMPUP

::004AFDB5::  837D F0 00               CMP DWORD PTR [EBP-10],0                －－－－－－检查是否填入了注册码！

::004AFDB9::  75 0F                    JNZ SHORT 004AFDCA                      \:JMPDOWN

::004AFDBB::  B8 74FF4A00              MOV EAX,4AFF74                          \:BYJMP JmpBy:004AFDA5,    \->: 注册信息不完整！！
```

注册名：`wofan`

注册码：`12345678`

来到这里：

``` 
004AFDFC   |.  BE 01000000                       mov esi,1

004AFE01   |>  8D55 E8                           /lea edx,dword ptr ss:[ebp-18]－－－－－这个循环处理注册名

004AFE04   |.  8B45 F8                           |mov eax,dword ptr ss:[ebp-8]

004AFE07   |.  0FB64430 FF                       |movzx eax,byte ptr ds:[eax+esi-1]－－注册名依次取

004AFE0C   |.  E8 7395F5FF                       |call nicepdf.00409384－－－－－－－－取字符的十进制

004AFE11   |.  8B55 E8                           |mov edx,dword ptr ss:[ebp-18]

004AFE14   |.  8D45 FC                           |lea eax,dword ptr ss:[ebp-4]

004AFE17   |.  E8 CC4DF5FF                       |call nicepdf.00404BE8－－－－－－－－联接起来，存于F19CB0为起始地址的内存中

004AFE1C   |.  46                                |inc esi

004AFE1D   |.  4B                                |dec ebx

004AFE1E   |.^ 75 E1                             \jnz short nicepdf.004AFE01

004AFE20   |>  8D55 E4                           lea edx,dword ptr ss:[ebp-1C]
```

``` 
注册名：  w        o        f        a         n
十进制：  119     111      102      97        110
内存中：

00F19CB0  31 31 39 31 31 31 31 30  11911110

00F19CB8  32 39 37 31 31 30        297110
```

……

``` 
004AFE31   |.  B8 A4FF4A00                       mov eax,nicepdf.004AFFA4 ;  ASCII "20050424"－－－应该是软件创作的日期？却是一个很重要的数据！长度为8位

……

004AFE36   |.  E8 E950F5FF                       call nicepdf.00404F24－－－－返回EAX=0 或者1，就完了！！！跟进去看一看！

004AFE3B   |.  83F8 02                           cmp eax,2

004AFE3E   |.  0F8C B8000000                     jl nicepdf.004AFEFC  －－－－只要不少于2，就继续下去！是如何得到2的呢？

004AFE44   |.  8D45 E0                           lea eax,dword ptr ss:[ebp-20]

004AFE47   |.  50                                push eax

004AFE48   |.  B9 08000000                       mov ecx,8

004AFE4D   |.  BA 01000000                       mov edx,1

004AFE52   |.  8B45 FC                           mov eax,dword ptr ss:[ebp-4]

004AFE55   |.  E8 E64FF5FF                       call nicepdf.00404E40

004AFE5A   |.  8B45 E0                           mov eax,dword ptr ss:[ebp-20]

004AFE5D   |.  50                                push eax

004AFE5E   |.  8D55 DC                           lea edx,dword ptr ss:[ebp-24]

004AFE61   |.  8B87 84060000                     mov eax,dword ptr ds:[edi+684]

004AFE67   |.  E8 485FFBFF                       call nicepdf.00465DB4

004AFE6C   |.  8B55 DC                           mov edx,dword ptr ss:[ebp-24]－－－假码

004AFE6F   |.  58                                pop eax－－－－－－－－－－－－ 注册名生成的数据的前部分：11911110

004AFE70   |.  E8 AF50F5FF                       call nicepdf.00404F24－－－－－－比较假码的后八位，与注册名生成的数据的前八位相等

004AFE75   |.  48                                dec eax

004AFE76   |.  0F8C 80000000                     jl nicepdf.004AFEFC－－－－－－－以后就成功注册！！！写注册信息

004AFE7C   |.  B2 01                             mov dl,1

004AFE7E   |.  A1 34BB4300                       mov eax,dword ptr ds:[43BB34]

004AFE83   |.  E8 ACBDF8FF                       call nicepdf.0043BC34

004AFE88   |.  8BD8                              mov ebx,eax

004AFE8A   |.  BA 02000080                       mov edx,80000002

004AFE8F   |.  8BC3                              mov eax,ebx

004AFE91   |.  E8 3EBEF8FF                       call nicepdf.0043BCD4

004AFE96   |.  B1 01                             mov cl,1

004AFE98   |.  BA B8FF4A00                       mov edx,nicepdf.004AFFB8              ;  ASCII "Software\Microsoft\Windows\CurrentVersion\App Paths\NicePDF.dll"

004AFE9D   |.  8BC3                              mov eax,ebx

004AFE9F   |.  E8 94BEF8FF                       call nicepdf.0043BD38

004AFEA4   |.  8D45 D8                           lea eax,dword ptr ss:[ebp-28]

004AFEA7   |.  E8 8C60FEFF                       call nicepdf.00495F38

004AFEAC   |.  8D45 D8                           lea eax,dword ptr ss:[ebp-28]

004AFEAF   |.  BA 00004B00                       mov edx,nicepdf.004B0000              ;  ASCII "nicepdf.dll"

004AFEB4   |.  E8 2F4DF5FF                       call nicepdf.00404BE8

……

004AFF5D   \.  C3                                retn
```

×××××××××××××××××××××××××××××××××××××××××××××××××××××××××

``` 
004AFE36   |.  E8 E950F5FF   call nicepdf.00404F24－－－－返回EAX=0 或者1，就完了！！！跟进去看一看！
```

跟进看一看：

``` 
00404F2A   |. /74 31                             je short nicepdf.00404F5D

00404F2C   |. |53                                push ebx

00404F2D   |. |56                                push esi－－－－－ESI=6（是个固定值）

00404F2E   |. |57                                push edi

00404F2F   |. |89C6                              mov esi,eax－－－－－－－－－－字串20050424

00404F31   |. |89D7                              mov edi,edx－－－－－－－－－注册码12345678

00404F33   |. |8B4F FC                           mov ecx,dword ptr ds:[edi-4]－－－注册码长度8

00404F36   |. |57                                push edi

00404F37   |. |8B56 FC                           mov edx,dword ptr ds:[esi-4]－－－字串长度8

00404F3A   |. |4A                                dec edx－－－－－－－－－－－－－－8-1=7

00404F3B   |. |78 1B                             js short nicepdf.00404F58

00404F3D   |.  8A06                              mov al,byte ptr ds:[esi]－－－－怎么会从20050424取字符？取得它的第一个字符2(32)

00404F3F   |.  46                                inc esi

00404F40   |.  29D1                              sub ecx,edx－－－－－－－7-8（8是注册码位数）看来注册码该至少是8位！

00404F42   |.  7E 14                             jle short nicepdf.00404F58

00404F44   |>  F2:AE                             /repne scas byte ptr es:[edi]－－－－注册码12345678－－－－注册码第一个字符必须是2

00404F46   |.  75 10                             |jnz short nicepdf.00404F58

00404F48   |.  89CB                              |mov ebx,ecx

00404F4A   |.  56                                |push esi

00404F4B   |.  57                                |push edi

00404F4C   |.  89D1                              |mov ecx,edx

00404F4E   |.  F3:A6                             |repe cmps byte ptr es:[edi],byte ptr>－－－循环比较，后八位必须是：20050424

00404F50   |.  5F                                |pop edi

00404F51   |.  5E                                |pop esi

00404F52   |.  74 0C                             |je short nicepdf.00404F60

00404F54   |.  89D9                              |mov ecx,ebx

00404F56   |.^ EB EC                             \jmp short nicepdf.00404F44

00404F58   |>  5A                                pop edx

00404F59   |.  31C0                              xor eax,eax

00404F5B   |.  EB 08                             jmp short nicepdf.00404F65

00404F5D   |>  31C0                              xor eax,eax

00404F5F   |.  C3                                retn

00404F60   |> \5A                                pop edx

00404F61   |.  89F8                              mov eax,edi
```
