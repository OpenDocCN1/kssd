

# 英宇职业介绍管理系统 V5.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6086.htm`](https://www.pediy.com/kssd/pediy06/pediy6086.htm)

简单算法——英宇职业介绍管理系统 V5.0（Softsentry 2.11 保护）  
  
  
  
下载页面：  [`www.downloadsky.com/soft/9190.html`](http://www.downloadsky.com/soft/9190.html)  
软件大小：  2624 KB  
软件语言：  简体中文  
软件类别：  国产软件 / 共享版 / 其它行业  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2003-05-09 10:53:12  
下载次数：  859  
推荐等级：  \*\*\* 

【软件简介】：英宇职业介绍管理系统是针对职业介绍（人才服务）行业开发制作的管理系统。当前程序版本 V3.10（单机版本），由招聘信息管理、人才库管理、职介管理、介绍信打印、系统维护等功能组成。软件操作界面友好、功能多、操作简单，查询功能强大，并具有方便快捷的数据导出功能，各种数据表格可以所见即所得地转换为 EXCEL 格式，还可以导出介绍信到您自定义的 Word 模版中，打印您所需要的样式。 

【软件限制】：30 天试用 

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！ 

【破解工具】：TRW2000 娃娃修改版、Ollydbg1.09、FI2.5、W32Dasm 9.0 白金版 

—————————————————————————————————   
【过    程】： 

  
英宇职介管理 V5.exe 用 FI2.5 看是 Softsentry 2.11 壳，晕，现在居然还用 V2.11 加壳。 有专用的 For Softsentry2.11 的脱壳工具：Crkss211.com，脱完壳后就取消一切限制了。这篇我写的稍微简单点，其实 Softsentry 壳的算法都大同小异，具体的可以看我以前分析过的笔记。这个程序不同的是取了用户名和单位名进行运算。 

序列号：95065  
用户名：fly  
单位名：\[OCN\]\[FCG\]  
试炼码：ABCDEFGH-12345678-KLMNOPQ  
—————————————————————————————————  
可以下 bpx getdlgitemtexta   一般 Softsentry 壳下这个断点挺好用。 

  
拦下后返回程序细心跟踪会来到下面： 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721ABD(C)  
|  
:00721B55 8B3D44BC7200            mov edi, dword ptr \[0072BC44\]  
                                 ====>EDI\=YYG-YYZJ-           这就是 String 1 

:00721B5B B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721B60 2BC0                    sub eax, eax  
:00721B62 F2                      repnz  
:00721B63 AE                      scasb  
:00721B64 F7D1                    not ecx  
:00721B66 49                      dec ecx  
                                 ====>取长度  ECX\=9 

:00721B67 6649                    dec cx  
:00721B69 6683F9FF                cmp cx, FFFF  
:00721B6D 7426                    je 00721B95  
:00721B6F 6685C9                  test cx, cx  
:00721B72 7C1B                    jl 00721B8F 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721B8D(C)  
|  
:00721B74 8B1544BC7200            mov edx, dword ptr \[0072BC44\]  
                                 ====>EDX\=YYG-YYZJ- 

:00721B7A 0FBFC1                  movsx eax, cx  
:00721B7D 8A1402                  mov dl, byte ptr \[edx+eax\]  
                                 ====>DI\=依次倒序取 YYG-YYZJ- 

:00721B80 80FA3F                  cmp dl, 3F  
:00721B83 7406                    je 00721B8B  
:00721B85 3854041C                cmp byte ptr \[esp+eax+1C\], dl  
                                 ====>逐位比较试炼码前 9 位是否是 YYG-YYZJ- 

:00721B89 7504                    jne 00721B8F  
                                 ====>跳则 OVER！可以 NOP 掉，方便调试 ^O^ ^O^  
                       一、      ====>所以注册码前 9 位固定是 YYG-YYZJ- 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721B83(C)  
|  
:00721B8B 6649                    dec cx  
:00721B8D 79E5                    jns 00721B74  
                                 ====>循环比较！ 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00721B72(C), :00721B89(C)  
|  
:00721B8F 6683F9FF                cmp cx, FFFF  
:00721B93 7505                    jne 00721B9A 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721B6D(C)  
|  
:00721B95 BD01000000              mov ebp, 00000001  
                                 ====>EBP\=1 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721B93(C)  
|  
:00721B9A 8B3DCCBB7200            mov edi, dword ptr \[0072BBCC\]  
                                 ====>EDI\=-1002002            这就是 String 2 

:00721BA0 B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721BA5 2BC0                    sub eax, eax  
:00721BA7 F2                      repnz  
:00721BA8 AE                      scasb  
:00721BA9 F7D1                    not ecx  
:00721BAB 49                      dec ecx  
                                 ====>取长度  ECX\=8 

:00721BAC 8D7C241C                lea edi, dword ptr \[esp+1C\]  
                                 ====>EDI\=ABCDEFGH-12345678-KLMNOPQ  试炼码 

:00721BB0 668BD1                  mov dx, cx  
                                 ====>DX\=CX\=8 

:00721BB3 2BC0                    sub eax, eax  
:00721BB5 B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721BBA F2                      repnz  
:00721BBB AE                      scasb  
:00721BBC F7D1                    not ecx  
:00721BBE 49                      dec ecx  
                                 ====>取长度  ECX\=19 

:00721BBF 662BCA                  sub cx, dx  
                                 ====>CX\=19 - 8=11 

:00721BC2 6685C9                  test cx, cx  
:00721BC5 7E2F                    jle 00721BF6  
:00721BC7 6633F6                  xor si, si  
:00721BCA 6685D2                  test dx, dx  
:00721BCD 7E21                    jle 00721BF0 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721BEE(C)  
|  
:00721BCF A1CCBB7200              mov eax, dword ptr \[0072BBCC\]  
:00721BD4 0FBFFE                  movsx edi, si  
:00721BD7 8A0438                  mov al, byte ptr \[eax+edi\]  
                                 ====>AI=依次倒序取-1002002 

:00721BDA 3C3F                    cmp al, 3F  
:00721BDC 740B                    je 00721BE9  
:00721BDE 0FBFD9                  movsx ebx, cx  
:00721BE1 03DF                    add ebx, edi  
:00721BE3 38441C1C                cmp byte ptr \[esp+ebx+1C\], al  
                                 ====>逐位比较试炼码最后 8 位是否是-1002002 

:00721BE7 7507                    jne 00721BF0  
                                 ====>跳则 OVER！可以 NOP 掉，方便调试 ^O^ ^O^  
                       二、      ====>所以注册码最后 8 位固定是 -1002002 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721BDC(C)  
|  
:00721BE9 6646                    inc si  
:00721BEB 663BD6                  cmp dx, si  
:00721BEE 7FDF                    jg 00721BCF  
                                 ====>循环比较！ 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00721BCD(C), :00721BE7(C)  
|  
:00721BF0 663BD6                  cmp dx, si  
:00721BF3 7501                    jne 00721BF6  
:00721BF5 45                      inc ebp  
                                 ====>EBP\=1 + 1=2 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00721BC5(C), :00721BF3(C)  
|  
:00721BF6 83FD02                  cmp ebp, 00000002  
                                 ====>是否已比较 2 次？ 

:00721BF9 740A                    je 00721C05  
                                 ====>跳下去 

:00721BFB BDFEFFFFFF              mov ebp, FFFFFFFE  
:00721C00 E900010000              jmp 00721D05 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721BF9(C)  
|  
:00721C05 8B3D44BC7200            mov edi, dword ptr \[0072BC44\]  
:00721C0B B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721C10 2BC0                    sub eax, eax  
:00721C12 F2                      repnz  
:00721C13 AE                      scasb  
:00721C14 F7D1                    not ecx  
:00721C16 2BC0                    sub eax, eax  
:00721C18 8D740C1B                lea esi, dword ptr \[esp+ecx+1B\]  
:00721C1C 8BFE                    mov edi, esi  
:00721C1E B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721C23 F2                      repnz  
:00721C24 AE                      scasb  
:00721C25 F7D1                    not ecx  
:00721C27 8B3DCCBB7200            mov edi, dword ptr \[0072BBCC\]  
:00721C2D 2BC0                    sub eax, eax  
:00721C2F 8D51FF                  lea edx, dword ptr \[ecx\-01\]  
:00721C32 B9FFFFFFFF              mov ecx, FFFFFFFF  
:00721C37 F2                      repnz  
:00721C38 AE                      scasb  
:00721C39 F7D1                    not ecx  
:00721C3B 49                      dec ecx  
:00721C3C 8BC6                    mov eax, esi  
:00721C3E 2BC1                    sub eax, ecx  
:00721C40 8BCE                    mov ecx, esi  
:00721C42 C6041000                mov byte ptr \[eax+edx\], 00  
:00721C46 E8C54D0000              call 00726A10  
                                 ====>测试试炼码中间的 12345678 是否是数字？ 

:00721C4B 85C0                    test eax, eax  
:00721C4D 750A                    jne 00721C59  
                                 ====>是则跳下去 

:00721C4F BDFDFFFFFF              mov ebp, FFFFFFFD  
:00721C54 E9AC000000              jmp 00721D05 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721C4D(C)  
|  
:00721C59 BAE8807200              mov edx, 007280E8  
                                 ====>EDX\=0604 

:00721C5E 8BCE                    mov ecx, esi  
                                 ====>ECX\=12345678     试炼码中间的 8 位 

:00721C60 BDFCFFFFFF              mov ebp, FFFFFFFC  
:00721C65 E8F64D0000              call 00726A60  
                                 ====>取 12345678 的 16 进制值=00BC614E 

:00721C6A 66833D38BC720001        cmp word ptr \[0072BC38\], 0001  
:00721C72 8BF0                    mov esi, eax  
                                 ====>ESI\=00BC614E（H）=12345678（D） 

:00721C74 7559                    jne 00721CCF  
                                 ====>跳下去 

:00721C76 668B3D3EBC7200          mov di, word ptr \[0072BC3E\]  
:00721C7D 8B15C0BB7200            mov edx, dword ptr \[0072BBC0\]  
:00721C83 66C1EF08                shr di, 08  
:00721C87 668B0D3EBC7200          mov cx, word ptr \[0072BC3E\]  
:00721C8E 6681E1FF00              and cx, 00FF  
:00721C93 E8F8FAFFFF              call 00721790  
:00721C98 03F0                    add esi, eax  
:00721C9A 6685FF                  test di, di  
:00721C9D 750A                    jne 00721CA9  
:00721C9F 8B15C4BB7200            mov edx, dword ptr \[0072BBC4\]  
:00721CA5 8BCF                    mov ecx, edi  
:00721CA7 EB0B                    jmp 00721CB4 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721C9D(C)  
|  
:00721CA9 668BCF                  mov cx, di  
:00721CAC 8B15C4BB7200            mov edx, dword ptr \[0072BBC4\]  
:00721CB2 6641                    inc cx 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721CA7(U)  
|  
:00721CB4 E8D7FAFFFF              call 00721790  
:00721CB9 8BC8                    mov ecx, eax  
:00721CBB 85C9                    test ecx, ecx  
:00721CBD 7507                    jne 00721CC6  
:00721CBF BDFBFFFFFF              mov ebp, FFFFFFFB  
:00721CC4 EB36                    jmp 00721CFC 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721CBD(C)  
|  
:00721CC6 8BC6                    mov eax, esi  
:00721CC8 99                      cdq  
:00721CC9 F7F9                    idiv ecx  
:00721CCB 8BEA                    mov ebp, edx  
:00721CCD EB2D                    jmp 00721CFC 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721C74(C)  
|  
:00721CCF 66833D38BC720002        cmp word ptr \[0072BC38\], 0002  
:00721CD7 7523                    jne 00721CFC  
:00721CD9 668B153EBC7200          mov dx, word ptr \[0072BC3E\]  
                                 ====>DX\=3221            这个似乎是固定值 

:00721CE0 A1C4BB7200              mov eax, dword ptr \[0072BBC4\]  
                                 ====>EAX\=\[OCN\]\[FCG\]               单位名 

:00721CE5 50                      push eax  
:00721CE6 8B0DC0BB7200            mov ecx, dword ptr \[0072BBC0\]  
                                 ====>ECX\=fly                      用户名 

:00721CEC 51                      push ecx  
:00721CED 8B0DD4B97200            mov ecx, dword ptr \[0072B9D4\]  
                                 ====>ECX\=00017359（H）=95065（D）  序列号 

:00721CF3 E828FBFFFF              call 00721820  
                                 ====>关键 CALL！进入！对用户名、单位和序列号进行运算 

:00721CF8 8BE8                    mov ebp, eax  
                                 ====>EBP\=EAX\=0002B750（H）=178000（D）   运算的结果 

:00721CFA 2BEE                    sub ebp, esi  
                                 ====>EBX\=0002B750 - 00BC614E=FF465602  
                                 ====>其实就是比较注册码中间几位是否和上面运算的结果相等！  
                       三、      ====>所以我的注册码中间几位是 178000 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00721CC4(U), :00721CCD(U), :00721CD7(C)  
|  
:00721CFC 85ED                    test ebp, ebp  
:00721CFE 7429                    je 00721D29 

…… ……省 略…… …… 

:00721E74 FF15B0C57200            call dword ptr \[0072C5B0\]  
                                 ====>BAD BOY！ 

  
—————————————————————————————————   
进入关键 CALL：00721CF3   call 00721820 

  
\* Referenced by a CALL at Address:  
|:00721CF3     
|  
:00721820 53                      push ebx  
:00721821 56                      push esi  
:00721822 57                      push edi  
:00721823 8BD9                    mov ebx, ecx  
:00721825 668BCA                  mov cx, dx  
                                 ====>CX\=DX\=3221 

:00721828 668BFA                  mov di, dx  
                                 ====>DI\=DX\=3221 

:0072182B 8B542410                mov edx, dword ptr \[esp+10\]  
                                 ====>EDX\=fly 

:0072182F 6681E1FF00              and cx, 00FF  
                                 ====>CX\=3221 AND FF=21 

:00721834 66C1EF08                shr di, 08  
                                 ====>DI\=3221 SHR 08=32 

:00721838 E853FFFFFF              call 00721790  
                                 ====>关键 CALL！进入！对用户名 fly 进行运算 

:0072183D 668BCF                  mov cx, di  
:00721840 8BF0                    mov esi, eax  
:00721842 6685C9                  test cx, cx  
:00721845 7517                    jne 0072185E  
:00721847 8B542414                mov edx, dword ptr \[esp+14\]  
:0072184B E840FFFFFF              call 00721790  
:00721850 8D0C33                  lea ecx, dword ptr \[ebx+esi\]  
:00721853 5F                      pop edi  
:00721854 0FAFC8                  imul ecx, eax  
:00721857 8BC1                    mov eax, ecx  
:00721859 5E                      pop esi  
:0072185A 5B                      pop ebx  
:0072185B C20800                  ret 0008 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00721845(C)  
|  
:0072185E 6641                    inc cx  
:00721860 8B542414                mov edx, dword ptr \[esp+14\]  
                                 ====>EDX\=\[OCN\]\[FCG\] 

:00721864 E827FFFFFF              call 00721790  
                                 ====>对单位名\[OCN\]\[FCG\]进行运算！ 

:00721869 03C6                    add eax, esi  
                                 ====>对用户名和单位名运算的结果相加  
                                 ====>EAX\=00006760 + 0000DC97=000143F7 

:0072186B 5F                      pop edi  
:0072186C 03C3                    add eax, ebx  
                                 ====>EBX\=00017359（H）=95065（D） 即：序列号  
                                 ====>EAX\=000143F7 + 00017359=0002B750 

:0072186E 5E                      pop esi  
:0072186F 5B                      pop ebx  
:00721870 C20800                  ret 0008 

  
—————————————————————————————————  
进入 0072184B   call  00721790  
因为对用户名和单位名的运算流程是相同的，所以只是记录了用户名的运算数据。 

  
\* Referenced by a CALL at Addresses:  
|:00721838   , :0072184B   , :00721864   , :00721C93   , :00721CB4     
|  
:00721790 53                      push ebx  
:00721791 56                      push esi  
:00721792 668BD9                  mov bx, cx  
                                 ====>BX\=21 

:00721795 57                      push edi  
:00721796 55                      push ebp  
:00721797 8BF2                    mov esi, edx  
:00721799 85F6                    test esi, esi  
                                 ====>ESI\=fly 

:0072179B 7475                    je 00721812  
:0072179D 803E00                  cmp byte ptr \[esi\], 00  
:007217A0 7470                    je 00721812  
:007217A2 8BFE                    mov edi, esi  
:007217A4 B9FFFFFFFF              mov ecx, FFFFFFFF  
:007217A9 2BC0                    sub eax, eax  
:007217AB F2                      repnz  
:007217AC AE                      scasb  
:007217AD F7D1                    not ecx  
:007217AF 49                      dec ecx  
                                 ====>取 fly 长度   ECX\=3 

:007217B0 6685DB                  test bx, bx  
:007217B3 7444                    je 007217F9  
:007217B5 6683FB01                cmp bx, 0001  
:007217B9 743E                    je 007217F9  
:007217BB 0FB7FB                  movzx edi, bx  
:007217BE 8BC7                    mov eax, edi  
                                 ====>EAX\=21 

:007217C0 99                      cdq  
:007217C1 F7F9                    idiv ecx  
                                 ====>EDX\=21 % 3=0 

:007217C3 0FBE0416                movsx eax, byte ptr \[esi+edx\]  
                                 ====>EAX\=66   根据余数 EDX 的值 0 取 fly 的第一位 

:007217C7 0FAFC2                  imul eax, edx  
                                 ====>EAX\=66 \* 0=0 

:007217CA 0FAFC7                  imul eax, edi  
                                 ====>EAX\=0 \* 21=0 

:007217CD 03C1                    add eax, ecx  
                                 ====>EAX\=0 + 3=3 

:007217CF 33D2                    xor edx, edx  
:007217D1 85C9                    test ecx, ecx  
:007217D3 7E19                    jle 007217EE  
:007217D5 8BD9                    mov ebx, ecx  
                                 ====>EBX\=ECX\=3 

:007217D7 2BDF                    sub ebx, edi  
                                 ====>EBX\=3 - 21=FFFFFFE2 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:007217EC(C)  
|  
:007217D9 0FBE3C16                movsx edi, byte ptr \[esi+edx\]  
                                 ====>EDI\=依次取 fly 字符的 HEX 值：66、6C、79 

:007217DD 8BEB                    mov ebp, ebx  
                                 ====>EBP\=EBX\=FFFFFFE2 

:007217DF 2BEA                    sub ebp, edx  
                          1、    ====>EBP\=FFFFFFE2 - 0=FFFFFFE2  
                          2、    ====>EBP\=FFFFFFE2 - 1=FFFFFFE1  
                          3、    ====>EBP\=FFFFFFE2 - 2=FFFFFFE0 

:007217E1 42                      inc edx  
                                 ====>EDX 依次增 1 

:007217E2 83C56F                  add ebp, 0000006F  
                          1、    ====>EBP\=FFFFFFE2 + 6F=51  
                          2、    ====>EBP\=FFFFFFE1 + 6F=50  
                          3、    ====>EBP\=FFFFFFE0 + 6F=4F 

:007217E5 0FAFFD                  imul edi, ebp  
                          1、    ====>EDI\=00000066 \* 51=00002046  
                          2、    ====>EDI\=0000006C \* 50=000021C0  
                          3、    ====>EDI\=00000079 \* 4F=00002557 

:007217E8 03C7                    add eax, edi  
                          1、    ====>EAX\=00000003 + 00002046=00002049  
                          2、    ====>EAX\=00002049 + 000021C0=00004209  
                          3、    ====>EAX\=00004209 + 00002557=00006760 

:007217EA 3BCA                    cmp ecx, edx  
:007217EC 7FEB                    jg 007217D9  
                                 ====>继续循环 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:007217D3(C)  
|  
:007217EE 85C0                    test eax, eax  
          对用户名 fly 运算得出   ====>EAX\=00006760  
          对\[OCN\]\[FCG\]运算得出   ====>EAX\=0000DC97 

:007217F0 7D25                    jge 00721817  
:007217F2 F7D8                    neg eax  
:007217F4 5D                      pop ebp  
:007217F5 5F                      pop edi  
:007217F6 5E                      pop esi  
:007217F7 5B                      pop ebx  
:007217F8 C3                      ret 

  
—————————————————————————————————  
【算 法  总 结】： 

  
1、注册码前 9 位固定为：YYG-YYZJ- 

2、注册码最后 8 位固定：-1002002 

3、注册码中间几位是通过对用户名、单位名、序列号运算得出的。     

  
—————————————————————————————————   
【注册信息保存】： 

  
1、REGEDIT4  
\[HKEY\_CLASSES\_ROOT\\{1N1AXAvCav}\]  
@="NUQ=&!!9!(Q!!!#!!#!\\"G!T5Q.4)U!!!!!!\\"=R1!!>\`^:75=N76F;3CUR.TAQN-$!N-4!Q-D!Q-A!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!#!!!!!!!!N!!!!(A!!!.-(\\"1!'!\\"\]!!!!A!!A!:A-!!!)!!!!!!!!!!+(\`<1&G<(E!7U^$;4FV<2E.(81!!!!!!!!!!!!!!!!!!!!!!!!!!" 

2、REGEDIT4  
\[HKEY\_CLASSES\_ROOT\\SystemAppIDs\]  
@="B!A!!!!!!!!\\"\\\\-XJ';E>04W\*638V\\\\-5YR16B\\">E.B>HU!" 

3、C:\\WINDOWS\\SYSTEM 下的 access.ctl 文件。 

  
—————————————————————————————————   
【整        理】： 

  
序列号：95065  
用户名：fly  
单位名：\[OCN\]\[FCG\]  
注册码：YYG-YYZJ-178000-1002002 

—————————————————————————————————  
     
                                 
        ,     \_/   
       /| \_.-~/            \\\_     ,        青春都一饷  
      ( /~   /              \\~-.\_ |\\  
      \`\\\\  \_/                \\   ~\\ )          忍把浮名   
  \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
 /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
\`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
 /~~//'    /' \`~\\         ) /--..\_, )\_  \`~  
 "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                        "     "   "~'  "" 

    

                   Cracked By 巢水工作坊——fly \[OCN\]\[FCG\] 

                          2003-05-31   03:03

  
 4dd466d0e1e0cc123afbe504da2462ca