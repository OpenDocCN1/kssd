
# 定式详解 V2.0[围棋]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-107.htm`](https://www.pediy.com/kssd/pediy08/pediy8-107.htm)

【文章标题】: 定式详解 V2.0[围棋]去功能限制及爆破注册——菜鸟教学【2】  
【作者】: `laoqian[FCG]`  
【邮箱】: -  
【主页】: `www.fcgchina.com`  
【QQ 号】: -  
【软件名称】: 定式详解 V2.0 [繁体版]  
【下载地址】: `http://www.lshuzhi.com`, 但是作者已经不提供此版本下载，有需要的告诉我！  
【加壳方式】: 无  
【编写语言】: `delphi`  
【工具】: `OllyDbg1.10 fly`, `W32DSM`  
【操作平台】: `windows` 系列  
【软件介绍】: 定式详解包含了几乎所有常用定式，带有详细的解说。有了它，查询定式、学习定式将不再是麻烦事。你只要走一手，计算机就会告诉你下一步所有可能的应法，包括不正确的应法（解说中告诉你不正确的原因）。你也可以随时进入研究模式自己再琢磨新的走法，研究时进退自如，免除在传统棋盘上反复移挪棋子之苦。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

【详细过程】

狗年就不打狗了，我们再来一个简单的爆破教学吧！

本文只适合初学者，高手略过！

前两天下了几次围棋，老是输，主要是定式都不会了，开局就落后！搞的人比较烦，我遂在网上找定式的软件，但都是一些试用功能不全的，比如 `http://www.lshuzhi.com/joseki.htm` 上的定式详解 V3.0，使用起来倒是很方便，只是代码不全，无法看到所有定式！

忽然想起在 4、5 年前，有人给过我他的繁体英文版，要我破解，据说里面是全的，当时我初入破解，没有搞定，就放下了，没想到一搜硬盘，居然还找到了，那就干他，都是 5 年前的东西了，`lshuzhi` 不会找我的茬吧？！

版本是 定式详解 V2.0，其实比 3.0 差不了许多，围棋定式几百年也变化不了多少，呵呵！

但他有限制，主要是下面几个：

1. 必须在繁体中文或者英文操作系统下使用！[显示繁体乱码，别扭吧！]
2. 不注册，无法使用全部，但代码是全的！

由于分析得知，有代码的，。那我们就来爆破他实现，工作很简单，我只是说一下如何入手。其实对于新手破解时最困难的是如何入手，如何快速的分析，如何最恰当的下断点！

```
代码:

* * *

:00484916 8BC0                     mov eax, eax
:00484918 55                       push ebp
:00484919 8BEC                     mov ebp, esp
:0048491B B91C000000               mov ecx, 0000001C

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484925(C)
|
:00484920 6A00                     push 00000000
:00484922 6A00                     push 00000000
:00484924 49                       dec ecx
:00484925 75F9                     jne 00484920
:00484927 51                       push ecx
:00484928 53                       push ebx
:00484929 56                       push esi
:0048492A 8BD8                     mov ebx, eax
:0048492C 33C0                     xor eax, eax
:0048492E 55                       push ebp
:0048492F 68BF4D4800               push 00484DBF
:00484934 64FF30                   push dword ptr fs:[eax]
:00484937 648920                   mov dword ptr fs:[eax], esp

\* Reference To: kernel32.GetSystemDefaultLangID, Ord:0000h   ;判断操作系统语言版本的 API
                                        |
:0048493A E89921F8FF                 Call 00406AD8
:0048493F 8BD0                       mov edx, eax    ；我们简体版返回是 eax＝0804，繁体是 0404 了
:00484941 6683E23F                   and dx, 003F    ；dx＝4
:00484945 0FB7F0                     movzx esi, ax   ；传 0804
:00484948 C1EE0A                     shr esi, 0A     ；si＝2
:0048494B 6683FA04                   cmp dx, 0004    ；比较 4 是中文简体或繁体版，其余还是其他系统
:0048494F 7506                       jne 00484957    ；中文版相等不跳
:00484951 6683FE01                   cmp si, 0001    ；比较 2 是简体，1 是繁体
:00484955 7404                       je  0048495B    ；相等跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048494F(C)
|
:00484957 33C0                       xor eax, eax   ；标志置 0。来这里是非中文版，即英语版，但是解说是乱码！
:00484959 EB02                       jmp 0048495D   ；跳到标志位 A

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484955(U)
|
:0048495B B001                       mov al, 01   ；标志置 1 繁体版

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484959(U)
|
:0048495D 88832C030000               mov byte ptr [ebx+0000032C], al    ；标志 A ！！我们这里 al＝0
:00484963 6683FA04                   cmp dx, 0004   ；比较 4 是中文简体或繁体版，其余还是其他系统
:00484967 7506                       jne 0048496F   ；中文版相等不跳
:00484969 6683FE02                   cmp si, 0002   ；比较 2 是简体，1 是繁体
:0048496D 7404                       je 00484973    ；相等跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484967(C)
|
:0048496F 33C0                       xor eax, eax  ；标志置 0。来这里是非中文版，即英语版，但是解说是乱码！
:00484971 EB02                       jmp 00484975  ；跳到标志位 B

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048496D(C)
|
:00484973 B001                       mov al, 01  ；标志置 1 简体版

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484971(U)
|
:00484975 88832D030000               mov byte ptr [ebx+0000032D], al  ；标志 B ！简体版我们这里 al＝1
:0048497B 84C0                       test al, al   ；关键比较，
:0048497D 7416                       je 00484995   ；al＝0 才跳 ！我们强制 jmp  EB16 
> 
> \* Possible StringData Ref from Code Obj ->"该版本不是简体版，简体版请到 http://www.lshuzhi"
>                                        ->".com 下载"
>                                        |；向上看避过这里吧！
:0048497F B8D44D4800                 mov eax, 00484DD4
:00484984 E887CDFCFF                 call 00451710
:00484989 A11C924900                 mov eax, dword ptr [0049921C]
:0048498E 8B00                       mov eax, dword ptr [eax]
:00484990 E8E76AFCFF                 call 0044B47C        ；出错提示，\*进去看是调用 user32.PostQuitMessage

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048497D(U)
|
:00484995 80BB2D03000000             cmp byte ptr [ebx+0000032D], 00   ；这里＝1
:0048499C 750D                       jne 004849AB  ；跳了！！
:0048499E 80BB2C03000000             cmp byte ptr [ebx+0000032C], 00
:004849A5 7504                       jne 004849AB
:004849A7 33C0                       xor eax, eax
:004849A9 EB02                       jmp 004849AD

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0048499C(C), :004849A5(C)
|
:004849AB B001                       mov al, 01

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004849A9(U)
|
:004849AD 88832E030000               mov byte ptr [ebx+0000032E], al  ；标志 C ！我们这里 al＝1
:004849B3 B205                       mov dl, 05
:004849B5 8B83D4020000               mov eax, dword ptr [ebx+000002D4]  ；一路走下去出错提示消失
:004849BB E8D871FAFF                 call 0042BB98
:004849C0 B201                       mov dl, 01
:004849C2 8B83D8020000               mov eax, dword ptr [ebx+000002D8]
:004849C8 E8CB7AFAFF                 call 0042C498
:004849CD 33D2                       xor edx, edx
:004849CF 8B8324030000               mov eax, dword ptr [ebx+00000324]
:004849D5 E8BE7AFAFF                 call 0042C498
:004849DA B205                       mov dl, 05
:004849DC 8B83D8020000               mov eax, dword ptr [ebx+000002D8]
:004849E2 E8B171FAFF                 call 0042BB98
:004849E7 B201                       mov dl, 01    
> 
> \* Possible StringData Ref from Code Obj ->"DA"
>                                        |
:004849E9 A1C4DF4000                 mov eax, dword ptr [0040DFC4]
:004849EE E825E5F7FF                 call 00402F18
:004849F3 898330030000               mov dword ptr [ebx+00000330], eax
:004849F9 C78564FFFFFF94000000       mov dword ptr [ebp+FFFFFF64], 00000094
:00484A03 8D8564FFFFFF               lea eax, dword ptr [ebp+FFFFFF64]
:00484A09 50                         push eax

\* Reference To: kernel32.GetVersionExA, Ord:0000h
                                        |
:00484A0A E8F120F8FF                 Call 00406B00
:00484A0F 83BD74FFFFFF02             cmp dword ptr [ebp+FFFFFF74], 00000002
:00484A16 0F94C0                     sete al
:00484A19 888334030000               mov byte ptr [ebx+00000334], al
:00484A1F 80BB2E03000000             cmp byte ptr [ebx+0000032E], 00
:00484A26 0F84E3010000               je 00484C0F    ；不跳，一路走下去退出！！呵呵，还有暗桩！
:00484A2C 8D8D60FFFFFF               lea ecx, dword ptr [ebp+FFFFFF60]

* * *
```

此刻我们想起一般退出都是调用 `user32.PostQuitMessage`，正好前面也有调用这个的，我们进去看看会发现什么？



```
代码:

* * *

\* Referenced by a CALL at Addresses:
|:00448263   , :00484990   , :00485034   ;发现有 3 处调用这里，我们一一进去看看！
|
:0044B47C E86303FCFF            call 0040B7E4
:0044B481 84C0                  test al, al
:0044B483 7407                  je 0044B48C
:0044B485 6A00                  push 00000000

\* Reference To: user32.PostQuitMessage, Ord:0000h
                            |
:0044B487 E864BDFBFF           Call 004071F0

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B483(C)
|
:0044B48C C3                  ret

在第三处我们看到了：


:00485014 803D648F490000       cmp byte ptr \[00498F64\], 00
:0048501B 741C                je 00485039
:0048501D C605648F490000       mov byte ptr \[00498F64\], 00
:00485024 80B82D03000000       cmp byte ptr \[eax+0000032D\], 00 ；就是这里，我们前面简体版标志为 1,当然下面不跳！！
:0048502B 740C                je  00485039   ；我们当然要强制跳！改为 jmp
:0048502D A11C924900          mov eax, dword ptr \[0049921C\]
:00485032 8B00                mov eax, dword ptr \[eax\]
:00485034 E84364FCFF          call 0044B47C   ； 调用 user32.PostQuitMessage 的地方

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0048501B(C), :0048502B(U)
|
:00485039 C3                  ret
:0048503A 8BC0                mov eax, eax

* * *


```

好到此，我们进去了！可是，显示是繁体，就是我们看到的是乱码！！怎么办，解决他，这时要请出`od`来了。

我们接着程序往下走，各位看到下面代码，明明看到的都是简体的字符串啊！

```
代码:

* * *

:004849F9 C78564FFFFFF94000000   mov dword ptr \[ebp+FFFFFF64\], 00000094
:00484A03 8D8564FFFFFF          lea eax, dword ptr \[ebp+FFFFFF64\]
:00484A09 50                  push eax

\* Reference To: kernel32.GetVersionExA, Ord:0000h
                            |
:00484A0A E8F120F8FF          Call 00406B00
:00484A0F 83BD74FFFFFF02      cmp dword ptr \[ebp+FFFFFF74\], 00000002
:00484A16 0F94C0              sete al
:00484A19 888334030000        mov byte ptr \[ebx+00000334\], al
:00484A1F 80BB2E03000000      cmp byte ptr \[ebx+0000032E\], 00
:00484A26 0F84E3010000        je 00484C0F    ；不跳，一路走下去退出！！呵呵，还有暗桩！
:00484A2C 8D8D60FFFFFF        lea ecx, dword ptr \[ebp+FFFFFF60\]

\* Possible StringData Ref from Code Obj ->"定式详解"
                            |
:00484A32 BA144E4800          mov edx, 00484E14  ；就是这些地方！明明看到的都是简体的字符串啊！存 edx＝00484E14
:00484A37 8BC3                mov eax, ebx
:00484A39 E892FAFFFF          call 004844D0   ；看来是这里可疑，我们进去看看吧！
:00484A3E 8B9560FFFFFF        mov edx, dword ptr \[ebp+FFFFFF60\]
:00484A44 8BC3                mov eax, ebx
:00484A46 E8657BFAFF          call 0042C5B0
:00484A4B 8D8D5CFFFFFF        lea ecx, dword ptr \[ebp+FFFFFF5C\]

\* Possible StringData Ref from Code Obj ->"请单击 A、B、C 等点"
                            |
:00484A51 BA284E4800          mov edx, 00484E28  ；就是这些地方！明明看到的都是简体的字符串啊！
:00484A56 8BC3                mov eax, ebx
:00484A58 E873FAFFFF          call 004844D0   ；看来是这里可疑，
:00484A5D 8B955CFFFFFF        mov edx, dword ptr \[ebp+FFFFFF5C\]
:00484A63 8B83F0020000        mov eax, dword ptr \[ebx+000002F0\]
:00484A69 E8427BFAFF          call 0042C5B0
:00484A6E 8D8D58FFFFFF        lea ecx, dword ptr \[ebp+FFFFFF58\]


\* Possible StringData Ref from Code Obj ->"解说"
                            |
:00484A74 BA444E4800          mov edx, 00484E44  ；就是这些地方！明明看到的都是简体的字符串啊！
:00484A79 8BC3                mov eax, ebx
:00484A7B E850FAFFFF          call 004844D0     ；看来是这里可疑，
:00484A80 8B9558FFFFFF        mov edx, dword ptr \[ebp+FFFFFF58\]
:00484A86 8B8320030000        mov eax, dword ptr \[ebx+00000320\]
:00484A8C E81F7BFAFF          call 0042C5B0
:00484A91 8D8D54FFFFFF        lea ecx, dword ptr \[ebp+FFFFFF54\]

* * *


```

我们进去`call 004844D0`看看吧！好多地方调用

```
代码:

* * *

\* Referenced by a CALL at Addresses:
|:00482A8C   , :00482AAA   , :00482ACC   , :00482B13   , :00482B57
|:0048309B   , :004830B9   , :004839E5   , :00483A03   , :00483A25
|:00483A47   , :00483A69   , :00483CB8   , :00483D03   , :00484A39
|:00484A58   , :00484A7B   , :00484A9E   , :00484AC1   , :00484AE4
|:00484B07   , :00484B2A   , :00484B4D   , :00484B73   , :00484B96
|:00484BC8   , :00484BF9   , :004852FD   , :0048543A   , :0048589C
|:00485A51   , :00485B70   , :0048639B   , :0048673F   , :00486766
|
:004844D0 53                  push ebx
:004844D1 56                  push esi
:004844D2 57                  push edi
:004844D3 8BF9                mov edi, ecx
:004844D5 8BF2                mov esi, edx  ；明明看到的都是简体的字符串啊！
:004844D7 8BD8                mov ebx, eax
:004844D9 8BCF                mov ecx, edi
:004844DB 8BD6                mov edx, esi  ；下 d edx，看到"定式详解"
:004844DD 8B8318030000        mov eax, dword ptr \[ebx+00000318\]
:004844E3 E878D2FFFF          call 00481760   ；看来是这里可疑，我们进去看看吧！
:004844E8 5F                  pop edi
:004844E9 5E                  pop esi
:004844EA 5B                  pop ebx
:004844EB C3                  ret

* * *


```

我们进去`call 00481760`看看吧！

```
代码:

* * *

\* Referenced by a CALL at Address:
|:004844E3
|
:00481760 53                  push ebx
:00481761 56                  push esi
:00481762 57                  push edi
:00481763 8BF9                mov edi, ecx
:00481765 8BF2                mov esi, edx  ；明明看到的都是简体的字符串啊！
:00481767 8BD8                mov ebx, eax
:00481769 57                  push edi
:0048176A 33C9                xor ecx, ecx
:0048176C 8BD6                mov edx, esi  ；下 d edx，看到"定式详解"
:0048176E 8B4324              mov eax, dword ptr \[ebx+24\]
:00481771 E806000000          call 0048177C    ；看来是这里可疑，我们进去看看吧！
:00481776 5F                  pop edi
:00481777 5E                  pop esi
:00481778 5B                  pop ebx
:00481779 C3                  ret

* * *


```

我们进去`call 0048177C`看看吧！这里是真正的重点了



```
代码:

* * *

\* Referenced by a CALL at Addresses:
|:004815FC   , :00481618   , :00481771    
|
:0048177C 55                   push ebp
:0048177D 8BEC                 mov ebp, esp
:0048177F 51                   push ecx
:00481780 B904000000           mov ecx, 00000004

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048178A(C)
|
:00481785 6A00                 push 00000000   ；准备堆栈
:00481787 6A00                 push 00000000   ；准备堆栈
:00481789 49                   dec ecx
:0048178A 75F9                 jne 00481785     ；循环
:0048178C 874DFC               xchg dword ptr [ebp-04], ecx
:0048178F 53                   push ebx
:00481790 56                   push esi   ；入栈，下 d esi，看到简体"定式详解" 
:00481791 57                   push edi
:00481792 884DFF               mov byte ptr [ebp-01], cl
:00481795 8BFA                 mov edi, edx  ；下 d edx，看到简体"定式详解" 
:00481797 33C0                 xor eax, eax
:00481799 55                   push ebp
:0048179A 6855194800           push 00481955
:0048179F 64FF30               push dword ptr fs:[eax]
:004817A2 648920               mov dword ptr fs:[eax], esp
:004817A5 8BC7                 mov eax, edi  ；走过下 d eax，看到简体"定式详解" 
:004817A7 E8A426F8FF           call 00403E50  ；获得"定式详解"长度的 call，返回 eax＝8

走过下 d eax 此时看内存：
00484E10  00000008  ...                ；长度 8
00484E14  BDCAA8B6  定式                ；字符
00484E18  E2BDEACF  详解
00484E1C  00000000  ....

:004817AC 8BF0                 mov esi, eax         ；esi＝eax＝8
:004817AE 83FE02               cmp esi, 00000002    ；比较
:004817B1 7D0F                 jge 004817C2         ；大于就跳，这里是关键，后面再说吧！！！
:004817B3 8B4508               mov eax, dword ptr [ebp+08]
:004817B6 8BD7                 mov edx, edi
:004817B8 E86724F8FF           call 00403C24   ；这里也后面说
:004817BD E970010000           jmp 00481932

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004817B1(C)
|
:004817C2 8D45F8               lea eax, dword ptr [ebp-08]   ；跳到这里，注意 edi 里是 00484E14，看到简体"定式详解" 
:004817C5 E80624F8FF           call 00403BD0
:004817CA BB01000000           mov ebx, 00000001    ；置 1
:004817CF 3BF3                 cmp esi, ebx   ；比较，此时 esi＝8，前面得到的
:004817D1 0F8C50010000          jl 00481927    ；当然不跳，接下面吧

* * *


```

；好这里开始就是简体转繁体的一段代码！！！




```
Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00481921(C)

:004817D7  3BF3                cmp esi, ebx    ; 比较转换到第几位
:004817D9  7522                jne 004817FD    ; 没结束就跳下去
:004817DB  8D45EC              lea eax, dword ptr [ebp-14]
:004817DE  50                  push eax
:004817DF  B901000000          mov ecx, 00000001
:004817E4  8BD3                mov edx, ebx
:004817E6  8BC7                mov eax, edi
:004817E8  E86B28F8FF          call 00404058
:004817ED  8B55EC              mov edx, dword ptr [ebp-14]
:004817F0  8D45F8              lea eax, dword ptr [ebp-08]
:004817F3  E86026F8FF          call 00403E58
:004817F8  E921010000          jmp 0048191E

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004817D9(C)

:004817FD  33C0                xor eax, eax    ; 清零开始
:004817FF  8A441FFF            mov al, byte ptr [edi+ebx-01]  ; edi 里是 00484E14，从第一位开始依次给 al
:00481803  8945F4              mov dword ptr [ebp-0C], eax     ; 传给 ebp-0C
:00481806  33C0                xor eax, eax    ; 清零
:00481808  8A041F              mov al, byte ptr [edi+ebx]  ; 汉字是双字节，当然要 2 位一起计算
:0048180B  807DFF00            cmp byte ptr [ebp-01], 00
:0048180F  0F8499000000        je 004818AE     ; 跳
:00481815  8B55F4              mov edx, dword ptr [ebp-0C]
:00481818  81C25FFFFFFF        add edx, FFFFFF5F
:0048181E  83EA59              sub edx, 00000059
:00481821  736C                jnb 0048188F
:00481823  8BD0                mov edx, eax
:00481825  83C2C0              add edx, FFFFFFC0
:00481828  83EA3F              sub edx, 0000003F
:0048182B  7208                jb 00481835
:0048182D  83C2DE              add edx, FFFFFFFE
:00481830  83EA5E              sub edx, 0000005E
:00481833  735A                jnb 0048188F

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0048182B(C)

:00481835  83F87F              cmp eax, 0000007F
:00481838  7D19                jge 00481853
:0048183A  8B55F4              mov edx, dword ptr [ebp-0C]
:0048183D  81EAA1000000        sub edx, 000000A1
:00481843  69D29D000000        imul edx, 0000009D
:00481849  83E840              sub eax, 00000040
:0048184C  03D0                add edx, eax
:0048184E  8955F0              mov dword ptr [ebp-10], edx
:00481851  EB17                jmp 0048186A

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00481838(C)

:00481853  8B55F4              mov edx, dword ptr [ebp-0C]
:00481856  81EAA1000000        sub edx, 000000A1
:0048185C  69D29D000000        imul edx, 0000009D
:00481862  83E862              sub eax, 00000062
:00481865  03D0                add edx, eax
:00481867  8955F0              mov dword ptr [ebp-10], edx

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00481851(U)

:0048186A  8D45E8              lea eax, dword ptr [ebp-18]
:0048186D  8B55F0              mov edx, dword ptr [ebp-10]
:00481870  8D1452              lea edx, dword ptr [edx+2*edx]
:00481873  81C2A4EB4800        add edx, 0048EBA4
:00481879  E87625F8FF          call 00403DF4
:0048187E  8B55E8              mov edx, dword ptr [ebp-18]
:00481881  8D45F8              lea eax, dword ptr [ebp-08]
:00481884  E8CF25F8FF          call 00403E58
:00481889  43                  inc ebx
:0048188A  E98F000000          jmp 0048191E

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00481821(C), :00481833(C)

:0048188F  8D45E4              lea eax, dword ptr [ebp-1C]
:00481892  50                  push eax
:00481893  B901000000          mov ecx, 00000001
:00481898  8BD3                mov edx, ebx
:0048189A  8BC7                mov eax, edi
:0048189C  E8B727F8FF          call 00404058
:004818A1  8B55E4              mov edx, dword ptr [ebp-1C]
:004818A4  8D45F8              lea eax, dword ptr [ebp-08]
:004818A7  E8AC25F8FF          call 00403E58
:004818AC  EB70                jmp 0048191E

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0048180F(C)

:004818AE  8B55F4              mov edx, dword ptr [ebp-0C]   ; 第一位
:004818B1  81C25FFFFFFF        add edx, FFFFFF5F    ; add
:004818B7  83EA57              sub edx, 00000057    ; sub
:004818BA  7345                jnb 00481901         ; 不跳
:004818BC  8BD0                mov edx, eax         ; 第二位
:004818BE  81C25FFFFFFF        add edx, FFFFFF5F    ; add
:004818C4  83EA5E              sub edx, 0000005E    ; sub
:004818C7  7338                jnb 00481901
:004818C9  8B55F4              mov edx, dword ptr [ebp-0C]
:004818CC  81EAA1000000        sub edx, 000000A1
:004818D2  6BD25E              imul edx, 0000005E
:004818D5  2DA1000000          sub eax, 000000A1
:004818DA  03D0                add edx, eax
:004818DC  8955F0              mov dword ptr [ebp-10], edx
:004818DF  8D45E0              lea eax, dword ptr [ebp-20]
:004818E2  8B55F0              mov edx, dword ptr [ebp-10]
:004818E5  8D1452              lea edx, dword ptr [edx+2*edx]
:004818E8  81C2CC8B4800        add edx, 00488BCC
:004818EE  E80125F8FF          call 00403DF4
:004818F3  8B55E0              mov edx, dword ptr [ebp-20]
:004818F6  8D45F8              lea eax, dword ptr [ebp-08]
:004818F9  E85A25F8FF          call 00403E58
:004818FE  43                  inc ebx
:004818FF  EB1D                jmp 0048191E

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004818BA(C), :004818C7(C)

:00481901  8D45DC              lea eax, dword ptr [ebp-24]
:00481904  50                  push eax
:00481905  B901000000          mov ecx, 00000001
:0048190A  8BD3                mov edx, ebx
:0048190C  8BC7                mov eax, edi
:0048190E  E84527F8FF          call 00404058
:00481913  8B55DC              mov edx, dword ptr [ebp-24]
:00481916  8D45F8              lea eax, dword ptr [ebp-08]
:00481919  E83A25F8FF          call 00403E58         ; 以上分析略过了，反正是简体转繁体！

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004817F8(U), :0048188A(U), :004818AC(U), :004818FF(U)

:0048191E  43                  inc ebx    ; 加 1
:0048191F  3BF3                cmp esi, ebx  ; 比较到了第几位了！
:00481921  0F8DB0FEFFFF        jnl 004817D7  ; 没转完，就跳去循环！
```

计算转换完毕来到下面

```
Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004817D1(C)

:00481927  8B4508              mov eax, dword ptr [ebp+08]
:0048192A  8B55F8              mov edx, dword ptr [ebp-08]  ; 此时就是转好的繁体"定式详解" ，下 d edx
:0048192D  E8F222F8FF          call 00403C24   ; 这个就是显示菜单的 call 了

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004817BD(U)

:00481932  33C0                xor eax, eax
:00481934  5A                  pop edx
:00481935  59                  pop ecx
:00481936  59                  pop ecx
:00481937  648910              mov dword ptr fs:[eax], edx
:0048193A  685C194800          push 0048195C

Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0048195A(U)

:0048193F  8D45DC              lea eax, dword ptr [ebp-24]
:00481942  BA05000000          mov edx, 00000005
:00481947  E8A822F8FF          call 00403BF4
:0048194C  8D45F8              lea eax, dword ptr [ebp-08]
:0048194F  E87C22F8FF          call 00403BD0
:00481954  C3                  ret
```

那我们剩下的工作，就是不要让他转换！我们跳过他吧



:004817AC 8BF0 mov esi, eax ；`esi＝eax＝8`

:004817AE 83FE02 cmp esi, 00000002 ；比较

:004817B1 7D0F jge 004817C2 ；我们强制不跳 `nop`，`90` 大法

:004817B3 8B4508 mov eax, dword ptr [ebp+08]

:004817B6 8BD7 mov edx, edi ；呵呵，此时是简体的了！

:004817B8 E86724F8FF call 00403C24 ；这个就是显示菜单的 `call` 了

:004817BD E970010000 jmp 00481932

修改，运行进入简体显示，好就此完工，下面我们来注册他吧！如果大家需要看他的简体转繁体的代码，可以仔细学习一下！这我可不敢乱说了！

```
代码:

* * *

* Possible StringData Ref from Code Obj ->"?C"
                             |
:00486704 A18C2D4800 mov eax, dword ptr [00482D8C]
:00486709 E806DDFBFF call 00444414         ；分析此处可以，下 bp 00444414
:0048670E 8B15C8914900 mov edx, dword ptr [004991C8]
:00486714 8902 mov dword ptr [edx], eax
:00486716 80BB5903000000 cmp byte ptr [ebx+00000359], 00
:0048671D 7427 je 00486746
:0048671F 8D45FC lea eax, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"Registered version"
                             |
:00486722 BAD0674800 mov edx, 004867D0
:00486727 E83CD5F7FF call 00403C68
:0048672C 80BB2E03000000 cmp byte ptr [ebx+0000032E], 00
:00486733 7436 je 0048676B
:00486735 8D4DFC lea ecx, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"注册用户"
                             |
:00486738 BAEC674800 mov edx, 004867EC
:0048673D 8BC3 mov eax, ebx
:0048673F E88CDDFFFF call 004844D0
:00486744 EB25 jmp 0048676B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048671D(C)
|
:00486746 8D45FC lea eax, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"Unregistered version"
                             |
:00486749 BA00684800 mov edx, 00486800
:0048674E E815D5F7FF call 00403C68
:00486753 80BB2E03000000 cmp byte ptr [ebx+0000032E], 00
:0048675A 740F je 0048676B
:0048675C 8D4DFC lea ecx, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"非注册用户"
                             |
:0048675F BA20684800 mov edx, 00486820
:00486764 8BC3 mov eax, ebx
:00486766 E865DDFFFF call 004844D0

* * *
```

用 `OllyDbg1.10` 打开我们修改后的 `.exe`，中断在入口，因为没有加壳，因此直接调试，忽略所有异常，取消所有断点，`F9` 运行出界面。

然后下 `bp 00444414`（或者干脆把调用这里的地方都下断），点击注册按钮！中断如下：

```
代码:

* * *

........
00484848  |. 8BCB mov ecx, ebx
0048484A  |. B2 01 mov dl, 1
0048484C  |. A1 00274800 mov eax, [482700]
00484851  |. E8 BEFBFBFF call 00444414       ；我们的断点返回到这里
00484856  |. 8B15 D8914900 mov edx, [4991D8]    ;  GoFormul.0049A988
0048485C  |. 8902 mov [edx], eax
0048485E  |. A1 D8914900 mov eax, [4991D8]
00484863  |. 8B00 mov eax, [eax]
00484865  |. 8B10 mov edx, [eax]
00484867  |. FF92 D8000000 call [edx+D8]    ；显示对话框，我们输入 787878，点 ok，中断下面
0048486D  |. 48 dec eax          ；这里 F2 设断吧
0048486E  |. 75 41 jnz short 004848B1
00484870  |. 8D55 FC lea edx, [ebp-4]
00484873  |. A1 D8914900 mov eax, [4991D8]
00484878  |. 8B00 mov eax, [eax]
0048487A  |. 8B80 DC020000 mov eax, [eax+2DC]
00484880  |. E8 FB7CFAFF call 0042C580
00484885  |. 8B55 FC mov edx, [ebp-4]   ；我们的 787878
00484888  |. 8BC3 mov eax, ebx
0048488A  |. E8 D1FEFFFF call 00484760    ；关键 call
0048488F  |. 84C0 test al, al        ；返回 0，失败
00484891  |. 74 1E je short 004848B1   ；不跳注册爆破，但是重启呢？
00484893  |. C683 59030000 01 mov byte ptr [ebx+359], 1   ；注册标志
0048489A  |. 33D2 xor edx, edx
0048489C  |. 8B83 E4020000 mov eax, [ebx+2E4]
004848A2  |. E8 F17BFAFF call 0042C498
004848A7  |. 8B55 FC mov edx, [ebp-4]
004848AA  |. 8BC3 mov eax, ebx
004848AC  |. E8 23FEFFFF call 004846D4
004848B1  |> A1 D8914900 mov eax, [4991D8]

进入 call 00484760

00484760  /$ 55 push ebp
00484761  |. 8BEC mov ebp, esp
00484763  |. 83C4 F0 add esp, -10
00484766  |. 53 push ebx
00484767  |. 56 push esi
00484768  |. 33C9 xor ecx, ecx
0048476A  |. 894D F0 mov [ebp-10], ecx
0048476D  |. 8955 FC mov [ebp-4], edx
00484770  |. 8BF0 mov esi, eax
00484772  |. 8B45 FC mov eax, [ebp-4]
00484775  |. E8 8AF8F7FF call 00404004
0048477A  |. 33C0 xor eax, eax
0048477C  |. 55 push ebp
0048477D  |. 68 16484800 push 00484816
00484782  |. 64:FF30 push dword ptr fs:[eax]
00484785  |. 64:8920 mov fs:[eax], esp
00484788  |. B2 01 mov dl, 1
0048478A  |. A1 18DD4000 mov eax, [40DD18]
0048478F  |. E8 84E7F7FF call 00402F18
00484794  |. 8945 F4 mov [ebp-C], eax
00484797  |. 33C0 xor eax, eax
00484799  |. 55 push ebp
0048479A  |. 68 F1474800 push 004847F1
0048479F  |. 64:FF30 push dword ptr fs:[eax]
004847A2  |. 64:8920 mov fs:[eax], esp
004847A5  |. BB 2D010000 mov ebx, 12D   ；12d
004847AA  |> 8D4D F0 /lea ecx, [ebp-10]
004847AD  |. 8BD3 |mov edx, ebx
004847AF  |. 8BC6 |mov eax, esi
004847B1  |. E8 FAFDFFFF |call 004845B0     ；此处计算得到注册码，可以计算得到 6 个！！
004847B6  |. 8B55 F0 |mov edx, [ebp-10]  ；[ebp-10]为真注册码！
004847B9  |. 8B45 F4 |mov eax, [ebp-C]
004847BC  |. 8B08 |mov ecx, [eax]
004847BE  |. FF51 34 |call [ecx+34]
004847C1  |. 43 |inc ebx
004847C2  |. 81FB 33010000 |cmp ebx, 133   ；133-12d＝6
004847C8  |.^ 75 E0 \\jnz short 004847AA
004847CA    8B55 FC mov edx, [ebp-4]   ；此处为假注册码！！
004847CD  |. 8B45 F4 mov eax, [ebp-C]
004847D0  |. 8B08 mov ecx, [eax]
004847D2  |. FF51 50 call [ecx+50]
004847D5  |. 85C0 test eax, eax
004847D7  |. 0F9D45 FB setge [ebp-5]
004847DB  |. 33C0 xor eax, eax
004847DD  |. 5A pop edx
004847DE  |. 59 pop ecx
004847DF  |. 59 pop ecx
004847E0  |. 64:8910 mov fs:[eax], edx
004847E3  |. 68 F8474800 push 004847F8
004847E8  |> 8B45 F4 mov eax, [ebp-C]
004847EB  |. E8 58E7F7FF call 00402F48
004847F0  \\. C3 retn

* * *
```

你可以进入 `call 004845B0` 看看，好复杂的算法，但是明码出现了，还管他干什么，做内存注册机吧。

但是我不作内存注册机，要做补丁，怎么办呢，

看到

`004847CA 8B55 FC mov edx, [ebp-4] ；此处为假注册码！！`

我们知道此时`[ebp-10]`为真注册码！，那我们把它替换！改为：

`004847CA 8B55 F0 mov edx, [ebp-10]`

这样只要注册一次，就可以了！

现在爽了，这些定式够我用的了！



# 总结

1. `0048497D 7416`: `je 00484995` ；我们强制跳 改为 `jmp 00484995` `[EB16]`
2. `0048502B 740C`: `je 00485039` ；我们强制跳！改为 `jmp 00485039` `[EB0C]`
3. `004817B1 7D0F`: `jge 004817C2` ；我们强制不跳 改为 `nop`， `[9090]`
4. `004847CA 8B55 FC`: `mov edx, [ebp-4]` ；我们把它替换！改为：
   `004847CA 8B55 F0`: `mov edx, [ebp-10]`

只改 5 个字节！爆破的艺术就是要用改代码最少的方法实现成功！

没什么可总结的，此程序好像就 6 个注册码呢！不知道当初他是为何，好几年了。找好 `api` 函数，断点，跟踪，爆破！主要是给大家一个过程，一个思路！

但是作者已经不提供此版本下载，有需要的告诉我！

唯一可以总结的就是，我们可以逆向得到他的简体转繁体的代码，倒是很实用呢：

1. 通过 `kernel32.GetSystemDefaultLangID` 判断操作系统，决定显示版本字体；
2. 把菜单，按钮、说明等的字符串依次用简体转繁体的 `call` 转一下，存下；
3. 再显示。

# 原版下载地址

http://free.ys168.com/?laoqian

解压密码：`fcg`

# 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
