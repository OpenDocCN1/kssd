
# PC-Washer 2.0 最新版 算法分析及 Keygen 源码

> 原文：[`www.pediy.com/kssd/pediy10/69360.html`](https://www.pediy.com/kssd/pediy10/69360.html)

【破文标题】PC-Washer 2.0 最新版 算法分析  

【破文作者】Playboysen  

【作者邮箱】playboysen@126.com  

【作者主页】playboysen2.photo.163.com  

【破解工具】`PEiD`, `OD`  

【破解平台】`Windows XP`  

【软件名称】PC Washer - ACleaner 2.0 build 07-24-08  

【软件大小】1.01 MB  

【软件类别】国外软件/系统辅助  

【软件授权】共享版  

【软件语言】英文  

【更新时间】2008-7-24  

【原版下载】`http://www.PC-Washer.com/`  

【保护方式】注册码  

【软件简介】PC Washer 是一款系统辅助软件，可以清理广告软体、注册表、快捷方式、程序垃圾等等，可以使你的系统更快速更健康！  

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

初学破解与编程，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

每天都要找一些软件来练手，从上次发表文章到现在已经一个月了，到时间遛遛了！这破解就和做数学题一样，几天不摸就手生，应该勤加练习啊 o(∩\_\)o...

试运行程序发现是单一注册码保护，有错误提示“Invalid Regisered...”，`PEID` 查壳得知是 `VC++`的程序，无壳省了不少劲。经过两遍初步跟踪，我不断调整假码，最终我们选择输入这个假码“playboysen8419876-54321-25814"（具体原因下面会详细分析）。

`OD` 加载查找字符串“invalid”找到下面的关键点：

代码:

```
0041AA40  /$  6A FF          push    -1
0041AA42  |.  68 182C4300   push    00432C18                  ;  SE 处理程序安装
0041AA47  |.  64:A1 0000000>mov     eax, dword ptr fs:\[0\]
0041AA4D  |.  50            push    eax
0041AA4E  |.  64:8925 00000>mov     dword ptr fs:\[0\], esp
0041AA55  |.  83EC 3C       sub     esp, 3C
0041AA58  |.  53            push    ebx
0041AA59  |.  55            push    ebp
0041AA5A  |.  8D4424 54     lea     eax, dword ptr \[esp+54\]
0041AA5E  |.  8D4C24 08     lea     ecx, dword ptr \[esp+8\]
0041AA62  |.  50            push    eax
0041AA63  |.  C74424 50 000>mov     dword ptr \[esp+50\], 0
0041AA6B  |.  E8 A2480100   call    <jmp.&MFC42.#535\_CString::CString>
0041AA70  |.  8D4C24 0C     lea     ecx, dword ptr \[esp+C\]
0041AA74  |.  C64424 4C 01  mov     byte ptr \[esp+4C\], 1
0041AA79  |.  E8 1A470100   call    <jmp.&MFC42.#540\_CString::CString>
0041AA7E  |.  8B4C24 08     mov     ecx, dword ptr \[esp+8\]               ;  取假码放入 ECX 中
0041AA82  |.  C64424 4C 02  mov     byte ptr \[esp+4C\], 2
0041AA87  |.  8379 F8 19    cmp     dword ptr \[ecx-8\], 19               ;  注册码大于等于 25 位，否则跳向错误处
0041AA8B  |.  0F8C C4020000 jl      0041AD55
0041AA91  |.  8D5424 1C     lea     edx, dword ptr \[esp+1C\]
0041AA95  |.  6A 0C         push    0C
0041AA97  |.  52            push    edx
0041AA98  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]             ;  假码内存地址放入 ECX
0041AA9C  |.  E8 2B490100   call    <jmp.&MFC42.#4129\_CString::Left>
0041AAA1  |.  50            push    eax
0041AAA2  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]
0041AAA6  |.  C64424 50 03  mov     byte ptr \[esp+50\], 3
0041AAAB  |.  E8 DC460100   call    <jmp.&MFC42.#858\_CString::operator=>
0041AAB0  |.  8D4C24 1C     lea     ecx, dword ptr \[esp+1C\]
0041AAB4  |.  C64424 4C 02  mov     byte ptr \[esp+4C\], 2
0041AAB9  |.  E8 C8460100   call    <jmp.&MFC42.#800\_CString::~CString>
0041AABE  |.  8D4424 1C     lea     eax, dword ptr \[esp+1C\]
0041AAC2  |.  6A 0C         push    0C
0041AAC4  |.  50            push    eax
0041AAC5  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]
0041AAC9  |.  E8 0A490100   call    <jmp.&MFC42.#4277\_CString::Mid>
0041AACE  |.  50            push    eax
0041AACF  |.  8D4C24 0C     lea     ecx, dword ptr \[esp+C\]
0041AAD3  |.  C64424 50 04  mov     byte ptr \[esp+50\], 4
0041AAD8  |.  E8 AF460100   call    <jmp.&MFC42.#858\_CString::operator=>
0041AADD  |.  8D4C24 1C     lea     ecx, dword ptr \[esp+1C\]
0041AAE1  |.  C64424 4C 02  mov     byte ptr \[esp+4C\], 2
0041AAE6  |.  E8 9B460100   call    <jmp.&MFC42.#800\_CString::~CString>
0041AAEB  |.  8B4C24 08     mov     ecx, dword ptr \[esp+8\]
0041AAEF  |.  8379 F8 11    cmp     dword ptr \[ecx-8\], 11               ;  上面一堆操作是去除假码的前 12 位；剩余的假码设为 K2，其实 K2 还要大于等于 17 位
0041AAF3  |.  0F8C 5C020000 jl      0041AD55
0041AAF9  |.  6A 05         push    5                                     ;  参数“5”
0041AAFB  |.  6A 2D         push    2D                                    ;  参数“-”
0041AAFD  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]
0041AB01  |.  E8 544B0100   call    <jmp.&MFC42.#6662\_CString::Find>     ;  此处在验证假码 K2；进入可知 K2 中每隔 5 位必须有个连字符“-”
0041AB06  |.  83CD FF       or      ebp, FFFFFFFF
0041AB09  |.  3BC5          cmp     eax, ebp
0041AB0B  |.  0F84 6A020000 je      0041AD7B
```

从 `0041AB01` 的 `call` 进入至此：

代码:

```
73D46911 >  8BFF          mov     edi, edi
73D46913    56            push    esi
73D46914    8BF1          mov     esi, ecx
73D46916    8B06          mov     eax, dword ptr \[esi\]
73D46918    8B4C24 0C     mov     ecx, dword ptr \[esp+C\]
73D4691C    3B48 F8       cmp     ecx, dword ptr \[eax-8\]
73D4691F    7D 15         jge     short 73D46936
73D46921    03C1          add     eax, ecx
73D46923    0FB64C24 08   movzx   ecx, byte ptr \[esp+8\]
73D46928    51            push    ecx
73D46929    50            push    eax
73D4692A    FF15 3467DD73 call    dword ptr \[<&msvcrt.\_mbsc>; 此处看右下角堆栈处已经显示了相关参数，要进入看看
73D46930    85C0          test    eax, eax                ; 若 ECX 的第一位是“-”，则此处 EAX 的值就是“-54321-25814”
73D46932    59            pop     ecx
73D46933    59            pop     ecx
73D46934    75 05         jnz     short 73D4693B
73D46936    83C8 FF       or      eax, FFFFFFFF
73D46939    EB 02         jmp     short 73D4693D
73D4693B    2B06          sub     eax, dword ptr \[esi\]    ; ESI 的指针是指向假码 K2 的
73D4693D    5E            pop     esi
73D4693E    C2 0800       retn    8
```

返回到 `0041AB06`。

从 `73D4692A` 处的 `call` 进来至此：

代码:



```
77C017FF >   8BFF            mov     edi, edi
77C01801    55              push    ebp
77C01802    8BEC            mov     ebp, esp
77C01804    E8 1C870000     call    77C09F25
77C01809    8B40 60         mov     eax, dword ptr [eax+60]
77C0180C    3B05 4425C377   cmp     eax, dword ptr [77C32544]
77C01812    74 05           je      short 77C01819
77C01814    E8 14EEFFFF     call    77C0062D
77C01819    8378 08 00      cmp     dword ptr [eax+8], 0
77C0181D    75 06           jnz     short 77C01825
77C0181F    5D              pop     ebp
77C01820    E9 3B5E0100     jmp     strchr
77C01825    8B4D 08         mov     ecx, dword ptr [ebp+8]    ; 把 K2 的后半部分"-54321-25814"放入 ECX
77C01828    53              push    ebx
77C01829    56              push    esi
77C0182A    EB 2F           jmp     short 77C0185B
77C0182C    0FB6F2          movzx   esi, dl
77C0182F    F64406 1D 04    test    byte ptr [esi+eax+1D], 4
77C01834    74 1C           je      short 77C01852
77C01836    41              inc     ecx
77C01837    8A19            mov     bl, byte ptr [ecx]
77C01839    84DB            test    bl, bl
77C0183B    74 3B           je      short 77C01878
77C0183D    0FB7D2          movzx   edx, dx
77C01840    0FB6F3          movzx   esi, bl
77C01843    C1E2 08         shl     edx, 8
77C01846    0BD6            or      edx, esi
77C01848    3955 0C         cmp     dword ptr [ebp+C], edx
77C0184B    75 0D           jnz     short 77C0185A
77C0184D    8D41 FF         lea     eax, dword ptr [ecx-1]
77C01850    EB 22           jmp     short 77C01874
77C01852    0FB7F2          movzx   esi, dx
77C01855    3975 0C         cmp     dword ptr [ebp+C], esi    ; 比较 ECX 的第一位是不是“-”
77C01858    74 0A           je      short 77C01864
77C0185A    41              inc     ecx
77C0185B    66:0FB611       movzx   dx, byte ptr [ecx]        ; 取 ECX 的第一位
77C0185F    66:85D2         test    dx, dx
77C01862   75 C8           jnz     short 77C0182C
77C01864    8B45 0C         mov     eax, dword ptr [ebp+C]
77C01867    0FB7D2          movzx   edx, dx
77C0186A    2BC2            sub     eax, edx                  ; 如果 ECX 第一位是“-”，则此处 EAX 位 0
77C0186C    F7D8            neg     eax                       ; NEG ( NEGate ) 求补指令
77C0186E    1BC0            sbb     eax, eax                  ; SBB ( SuBtract with Borrow) 带借位减法指令
77C01870    F7D0            not     eax                       ; EAX 取反
77C01872    23C1            and     eax, ecx
77C01874    5E              pop     esi
77C01875    5B              pop     ebx
77C01876    5D              pop     ebp
77C01877    C3              retn
```

返回到上面的 `73D46930` 处。  

代码:



```
0041AB11  |.  6A 0B        push    0B
0041AB13  |.  6A 2D        push    2D
0041AB15  |.  8D4C24 10    lea     ecx, dword ptr [esp+10]
0041AB19  |.  E8 3C4B0100  call    <jmp.&MFC42.#6662_CString>;  同上，比较 K2 的第十二位是不是“-”
0041AB1E  |.  3BC5        cmp     eax, ebp
0041AB20  |.  0F84 55020000 je     0041AD7B
0041AB26  |.  33DB        xor     ebx, ebx
0041AB28  |>  8AD3        /mov    dl, bl                   ;  这一段在比较 K2 有没有哪一位包含小写的 26 个字母，若包含则错
0041AB2A  |.  6A 01        |push    1
0041AB2C  |.  80C2 61     |add     dl, 61
0041AB2F  |.  8D4C24 0C   |lea     ecx, dword ptr [esp+C]
0041AB33  |.  885424 20   |mov     byte ptr [esp+20], dl
0041AB37  |.  8B4424 20   |mov     eax, dword ptr [esp+20]
0041AB3B  |.  50          |push    eax
0041AB3C  |.  E8 194B0100 |call    <jmp.&MFC42.#6662_CStrin>
0041AB41  |.  3BC5        |cmp     eax, ebp
0041AB43  |.  0F85 32020000 |jnz    0041AD7B
0041AB49  |.  43          |inc     ebx
0041AB4A  |.  83FB 1A     |cmp     ebx, 1A
0041AB4D  |.^ 7C D9      \jl     short 0041AB28
0041AB4F  |.  33DB        xor     ebx, ebx
0041AB51  |>  8ACB        /mov    cl, bl                   ;  这一段在比较 K2 有没有哪一位包含大写的 26 个字母，若包含则错
0041AB53  |.  6A 01        |push    1
0041AB55  |.  80C1 41     |add     cl, 41
0041AB58  |.  884C24 20   |mov     byte ptr [esp+20], cl
0041AB5C  |.  8B5424 20   |mov     edx, dword ptr [esp+20]
0041AB60  |.  52          |push    edx
0041AB61  |.  8D4C24 10   |lea     ecx, dword ptr [esp+10]
0041AB65  |.  E8 F04A0100 |call    <jmp.&MFC42.#6662_CStrin>
0041AB6A  |.  3BC5        |cmp     eax, ebp
0041AB6C  |.  0F85 09020000 |jnz    0041AD7B
0041AB72  |.  43          |inc     ebx
0041AB73  |.  83FB 1A     |cmp     ebx, 1A
0041AB76  |.^ 7C D9      \jl     short 0041AB51
0041AB78  |.  56          push    esi                       ;  也就是说 K2 必须全部是数字
0041AB79  |.  57          push    edi
0041AB7A  |.  8D4C24 20   lea     ecx, dword ptr [esp+20]
0041AB7E  |.  E8 15460100 call    <jmp.&MFC42.#540_CString:>
0041AB83  |.  8D4C24 1C   lea     ecx, dword ptr [esp+1C]
0041AB87  |.  C64424 54 05 mov    byte ptr [esp+54], 5
0041AB8C  |.  E8 07460100 call    <jmp.&MFC42.#540_CString:>
0041AB91  |.  8D4C24 18   lea     ecx, dword ptr [esp+18]
0041AB95  |.  C64424 54 06 mov    byte ptr [esp+54], 6
0041AB9A  |.  E8 F9450100 call    <jmp.&MFC42.#540_CString:>
0041AB9F  |.  8D4424 24   lea     eax, dword ptr [esp+24]
0041ABA3  |.  B3 07       mov     bl, 7
0041ABA5  |.  6A 05       push    5
0041ABA7  |.  50          push    eax
0041ABA8  |.  8D4C24 18   lea     ecx, dword ptr [esp+18]
0041ABAC  |.  885C24 5C   mov     byte ptr [esp+5C], bl
0041ABB0  |.  E8 17480100 call    <jmp.&MFC42.#4129_CString>
0041ABB5  |.  50          push    eax
0041ABB6  |.  8D4C24 24   lea     ecx, dword ptr [esp+24]
0041ABBA  |.  C64424 58 08 mov    byte ptr [esp+58], 8
0041ABBF  |.  E8 C8450100 call    <jmp.&MFC42.#858_CString:>
0041ABC4  |.  8D4C24 24   lea     ecx, dword ptr [esp+24]
0041ABC8  |.  885C24 54   mov     byte ptr [esp+54], bl
0041ABCC  |.  E8 B5450100 call    <jmp.&MFC42.#800_CString:>
0041ABD1  |.  8D4C24 24   lea     ecx, dword ptr [esp+24]
0041ABD5  |.  6A 05       push    5
0041ABD7  |.  51          push    ecx
0041ABD8  |.  8D4C24 18   lea     ecx, dword ptr [esp+18]
0041ABDC  |.  E8 2B4A0100 call    <jmp.&MFC42.#5710_CString>
0041ABE1  |.  50          push    eax
0041ABE2  |.  8D4C24 1C   lea     ecx, dword ptr [esp+1C]
0041ABE6  |.  C64424 58 09 mov    byte ptr [esp+58], 9
0041ABEB  |.  E8 9C450100 call    <jmp.&MFC42.#858_CString:>
0041ABF0  |.  8D4C24 24   lea     ecx, dword ptr [esp+24]
0041ABF4  |.  885C24 54   mov     byte ptr [esp+54], bl
0041ABF8  |.  E8 89450100 call    <jmp.&MFC42.#800_CString:>
0041ABFD  |.  6A 05       push    5
0041ABFF  |.  8D5424 28   lea     edx, dword ptr [esp+28]
0041AC03  |.  6A 06       push    6
0041AC05  |.  52          push    edx
0041AC06  |.  8D4C24 1C   lea     ecx, dword ptr [esp+1C]
0041AC0A  |.  E8 C1490100 call    <jmp.&MFC42.#4278_CString>
0041AC0F  |.  50          push    eax
0041AC10  |.  8D4C24 20   lea     ecx, dword ptr [esp+20]
0041AC14  |.  C64424 58 0A mov    byte ptr [esp+58], 0A
0041AC19  |.  E8 6E450100 call    <jmp.&MFC42.#858_CString:>
0041AC1E  |.  8D4C24 24   lea     ecx, dword ptr [esp+24]
0041AC22  |.  885C24 54   mov     byte ptr [esp+54], bl
0041AC26  |.  E8 5B450100 call    <jmp.&MFC42.#800_CString:>
0041AC2B  |.  8B7C24 20   mov     edi, dword ptr [esp+20]   ;  把 K2 中的三组数字分别设为 N1、N2、N3，则此处取 N1 进行运算
0041AC2F  |.  8BCD        mov     ecx, ebp
0041AC31  |.  33C0        xor     eax, eax
0041AC33  |.  8D5424 28   lea     edx, dword ptr [esp+28]
0041AC37  |.  F2:AE       repne   scas byte ptr es:[edi]    ;  相当于计算 N1 长度，不明白请参看

http://bbs.pediy.com/showthread.php?p=404309
0041AC39  |.  F7D1        not     ecx                       ;  经过此次 NOT 运算其实 ECX=6
0041AC3B  |.  2BF9        sub     edi, ecx
0041AC3D  |.  8BC1        mov     eax, ecx                  ;  把 ECX 的值放入 EAX 保存
0041AC3F  |.  8BF7        mov     esi, edi                  ;  把 N1 放入 ESI
0041AC41  |.  8BFA        mov     edi, edx
0041AC43  |.  8D5424 34   lea     edx, dword ptr [esp+34]
0041AC47  |.  C1E9 02     shr     ecx, 2                    ;  ECX 右移 2，结果放入 ECX
0041AC4A  |.  F3:A5       rep    movs dword ptr es:[edi], >;  同上
0041AC4C  |.  8BC8        mov     ecx, eax
0041AC4E  |.  33C0        xor     eax, eax
0041AC50  |.  83E1 03     and     ecx, 3                    ;  ECX 和 3 进行 AND 运算
0041AC53  |.  F3:A4       rep    movs byte ptr es:[edi], b>
0041AC55  |.  8B7C24 1C   mov     edi, dword ptr [esp+1C]   ;  把 N2 放入 EDI 进行运算
0041AC59  |.  8BCD        mov     ecx, ebp
0041AC5B  |.  F2:AE       repne   scas byte ptr es:[edi]
0041AC5D  |.  F7D1        not     ecx
0041AC5F  |.  2BF9        sub     edi, ecx
0041AC61  |.  8BC1        mov     eax, ecx                  ;  把 ECX 的值放入 EAX 保存
0041AC63  |.  8BF7        mov     esi, edi                  ;  N2 放入 ESI
0041AC65  |.  8BFA        mov     edi, edx
0041AC67  |.  C1E9 02     shr     ecx, 2
0041AC6A  |.  F3:A5       rep    movs dword ptr es:[edi], >;  同上
0041AC6C  |.  8BC8        mov     ecx, eax
0041AC6E  |.  33C0        xor     eax, eax
0041AC70  |.  83E1 03     and     ecx, 3
0041AC73  |.  8D5424 40   lea     edx, dword ptr [esp+40]
0041AC77  |.  F3:A4       rep    movs byte ptr es:[edi], b>
0041AC79  |.  8B7C24 18   mov     edi, dword ptr [esp+18]   ;  N3 放入 EDI
0041AC7D  |.  8BCD        mov     ecx, ebp
0041AC7F  |.  F2:AE       repne   scas byte ptr es:[edi]    ;  同上
0041AC81  |.  F7D1        not     ecx
0041AC83  |.  2BF9        sub     edi, ecx
0041AC85  |.  8BC1        mov     eax, ecx
0041AC87  |.  8BF7        mov     esi, edi                  ;  把 N3 放入 ESI 保存
0041AC89  |.  8BFA        mov     edi, edx
0041AC8B  |.  C1E9 02     shr     ecx, 2
0041AC8E  |.  F3:A5       rep    movs dword ptr es:[edi], >
0041AC90  |.  8BC8        mov     ecx, eax
0041AC92  |.  83E1 03     and     ecx, 3
0041AC95  |.  F3:A4       rep    movs byte ptr es:[edi], b>
0041AC97  |.  8B3D 10574300 mov    edi, dword ptr [<&MSVCRT.>;  C 函数 atoi(char *str) 将字符串 str 转换成整数并返回这个整数到

EAX 中
0041AC9D  |.  8D4C24 28   lea     ecx, dword ptr [esp+28]   ;  N1 的内存地址放入 ECX
0041ACA1  |.  51          push    ecx                       ; /s
0041ACA2  |.  FFD7        call    edi                       ; \\函数 atoi
0041ACA4  |.  8D5424 38   lea     edx, dword ptr [esp+38]   ;  此处看 EAX 的值为 4DA4h，十进制正好是 19876（我们假码中 N1 的值）
0041ACA8  |.  8BE8        mov     ebp, eax                  ;  这句指令是把 N1 的十六进制值储存到 EBP，下面关键点要用到
0041ACAA  |.  52          push    edx
0041ACAB  |.  FFD7        call    edi
0041ACAD  |.  8BF0        mov     esi, eax                  ;  此处看 EAX 的值为 D431h，十进制正好是 54321（我们假码中 N2 的值）

；这句指令是把 N2 的十六进制值储存到 ESI，下面关键点要用到
0041ACAF  |.  8D4424 48   lea     eax, dword ptr [esp+48]
0041ACB3  |.  50          push    eax
0041ACB4  |.  FFD7        call    edi                       ;  此处看 EAX 的值为 64D6h，十进制正好是 25814（我们假码中 N3 的值）
0041ACB6  |.  33F5        xor     esi, ebp                  ;  XOR N2,N1
0041ACB8  |.  83C4 0C     add     esp, 0C
0041ACBB  |.  81F6 B0410000 xor    esi, 41B0                ;  ESI 与 41B0h（十进制就是 16816）异或，值放入 ESI
0041ACC1  |.  C64424 54 06 mov    byte ptr [esp+54], 6
0041ACC6  |.  5F          pop     edi
0041ACC7  |.  3BF0        cmp     esi, eax                  ;  还记得吗？EAX 此时的值是假码中 N3 的十六进制哦！
0041ACC9  |.  5E          pop     esi                      
0041ACCA  |.  8D4C24 10   lea     ecx, dword ptr [esp+10]
0041ACCE  |.  75 64       jnz     short 0041AD34            ;  此处如果没有跳，则注册成功了！！！
0041ACD0  |.  E8 B1440100 call    <jmp.&MFC42.#800_CString:>
```



近来正好在自学 Delphi 编程，就顺手自己写个注册机，关键源码公开如下（Delphi 7 编译通过）：

代码:

```
procedure TForm1.btn1Click(Sender: TObject);
var
b,c,Temp:integer;
begin
    randomize; //初始化随机数
    repeat
    b:=10000+Random(90000);//算出一个随机 5 位数，即 N1
    c:=10000+Random(90000);//算出一个 N2
    Temp:=16816 xor (b xor c);//计算出 N3
    until Length(IntToStr(Temp))=5;//经测试发现 N3 必须是 5 位，否则会提示注册错误
    Form1.Edit1.Text:='Senhuanpediy'+ inttostr(b) + '-' + inttostr(c) + '-' + inttostr(Temp);
end;

end.
```

注意：软件的注册信息保存在这里

`Windows Registry Editor Version 5.00`

```
[HKEY_LOCAL_MACHINE\SOFTWARE\1KeyClean\]
"LastDate"="07-26-2008 11:11"
"VerifyKey"="playboysen8419876-54321-55333"
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `注册机及其源码.rar` |
| :--- | :--- |
