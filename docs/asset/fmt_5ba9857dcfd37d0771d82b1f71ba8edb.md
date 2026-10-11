
## 第十六章 Pic2Ico 2.1 注册主程序的功能模块简要理解

粗粗的阅读后，我们大致了解了此段的大致含义和重要程序功能块：

1. 判断用户名是否为空。

```
00427F19  |.  E>call    <sub_509664>                   ; \001.00509664
00427F1E  |.  5>pop     ecx
00427F1F  |.  8>test    cl, cl
00427F21  |.  7>je      short <loc_427F5F>
00427F23  |.  6>mov     word ptr [ebp-48], 14
00427F29  |.  B>mov     edx, 00523BDE                 ;  ASCII "Please input your Full Name!"
00427F2E  |.  8>lea     eax, dword ptr [ebp-8]
00427F31  |.  E>call    <sub_509504>
```

2. 注册成功的关键判断：

```
00427F8F  |.  E>call    <sub_4262FC>                   ; \001.004262FC
00427F94  |.  5>pop     ecx
00427F95  |.  8>mov     ecx, dword ptr [52E328]        ;  001._IconConverter
00427F9B  |.  8>mov     edx, dword ptr [ecx]
00427F9D  |.  8>mov     byte ptr [edx+3E4], al
00427FA3  |.  F>dec     dword ptr [ebp-3C]
00427FA6  |.  8>lea     eax, dword ptr [ebp-C]
00427FA9  |.  B>mov     edx, 2
00427FAE  |.  E>call    <sub_509664>
00427FB3  |.  A>mov     eax, dword ptr [52E328]
00427FB8  |.  8>mov     ecx, dword ptr [eax]
00427FBA  |.  8>cmp     byte ptr [ecx+3E4], 0
00427FC1  |.  0>je      <loc_4281E6>
```

3. 经典的语句，判断是否为数字。后来调试才证实是注册码的 20-24 位必须为数字。

关键 `call` `<sub_4262FC>` 里对 20-24 位没有交代。这个软件的巧妙就在于：在程序的两处分别进行判断，迷惑了一些人。那些以注册码的获得为目的的人可能会蒙对，虽然错误地理解为 20-24 位为任意字符，却没有输入字母，而是运气地输入数字。但写注册机时，这种理解的粗心是致命的。因此将注册机的编写作为比算法分析（并追出注册码）层次更高的境界是有理由的。

```
00428010  |.  E>call    <System::AnsiString::c_str(void)>
00428015  |.  0>movsx   edx, byte ptr [eax+17]
00428019  |.  8>cmp     edx, 30
0042801C  |.  7>jl      short <loc_428034>
0042801E  |.  8>mov     eax, dword ptr [ebp-5C]
00428021  |.  0>add     eax, 320
00428026  |.  E>call    <System::AnsiString::c_str(void)>
0042802B  |.  0>movsx   edx, byte ptr [eax+17]
0042802F  |.  8>cmp     edx, 39
00428032  |.  7>jle     short <loc_428043>
00428034  |>  8>mov     ecx, dword ptr [52E328]        ;  001._IconConverter
0042803A  |.  8>mov     eax, dword ptr [ecx]
0042803C  |.  C>mov     byte ptr [eax+3E4], 0
00428043  |>  B>mov     dl, 1
```

4. 将注册信息写入注册表，形式：

```
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Software\XTZY\Pic2Ico]
"NO"="520NBA2005YH*QH*UK*12345C*AN*PD*MO*OAD*RP2I3"
"Name"="NBA2005"
```

其中 `NO` 就是注册码。

```
0042804A  |.  E>call    <Registry::TRegistry::TRegistry(>
0042804F  |.  8>mov     dword ptr [ebp-64], eax
00428052  |.  B>mov     edx, 80000001
00428057  |.  8>mov     eax, dword ptr [ebp-64]
0042805A  |.  E>call    <Registry::TRegistry::SetRootKey>
0042805F  |.  8>mov     edx, dword ptr [52E328]        ;  001._IconConverter
00428065  |.  8>mov     ecx, dword ptr [edx]
00428067  |.  8>cmp     byte ptr [ecx+3E4], 0
0042806E  |.  0>je      <loc_42817A>
00428074  |.  6>mov     word ptr [ebp-48], 38
0042807A  |.  B>mov     edx, 00523BFB                 ;  ASCII "Software\XTZY\Pic2Ico"
0042807F  |.  8>lea     eax, dword ptr [ebp-14]
00428082  |.  E>call    <sub_509504>
00428087  |.  F>inc     dword ptr [ebp-3C]
0042808A  |.  8>mov     edx, dword ptr [eax]
0042808C  |.  B>mov     cl, 1
0042808E  |.  8>mov     eax, dword ptr [ebp-64]
00428091  |.  E>call    <Registry::TRegistry::OpenKey(Sy>
00428096  |.  8>test    al, al
00428098  |.  0>setne   al
0042809B  |.  8>and     eax, 1
0042809E  |.  5>push    eax                            ; /Arg1
0042809F  |.  F>dec     dword ptr [ebp-3C]             ; |
004280A2  |.  8>lea     eax, dword ptr [ebp-14]        ; |
004280A5  |.  B>mov     edx, 2                         ; |
004280AA  |.  E>call    <sub_509664>                   ; \001.00509664
004280AF  |.  5>pop     ecx
004280B0  |.  8>test    ecx, ecx
004280B2  |.  0>je      <loc_42817A>
004280B8  |.  8>lea     eax, dword ptr [ebp-1C]
004280BB  |.  E>call    <unknown_libname_44>
004280C0  |.  8>mov     edx, eax
004280C2  |.  F>inc     dword ptr [ebp-3C]
004280C5  |.  8>mov     ecx, dword ptr [ebp-5C]
004280C8  |.  8>mov     eax, dword ptr [ecx+304]
004280CE  |.  E>call    <TControl::GetText(void)>
004280D3  |.  8>lea     edx, dword ptr [ebp-1C]
004280D6  |.  F>push    dword ptr [edx]
004280D8  |.  6>mov     word ptr [ebp-48], 44
004280DE  |.  B>mov     edx, 00523C11                 ;  ASCII "NO"
004280E3  |.  8>lea     eax, dword ptr [ebp-18]
004280E6  |.  E>call    <sub_509504>
004280EB  |.  F>inc     dword ptr [ebp-3C]
004280EE  |.  8>mov     edx, dword ptr [eax]
004280F0  |.  8>mov     eax, dword ptr [ebp-64]
004280F3  |.  5>pop     ecx
004280F4  |.  E>call    <sub_47E824>
004280F9  |.  F>dec     dword ptr [ebp-3C]
004280FC  |.  8>lea     eax, dword ptr [ebp-1C]
004280FF  |.  B>mov     edx, 2
00428104  |.  E>call    <sub_509664>
00428109  |.  F>dec     dword ptr [ebp-3C]
0042810C  |.  8>lea     eax, dword ptr [ebp-18]
0042810F  |.  B>mov     edx, 2
00428114  |.  E>call    <sub_509664>
00428119  |.  8>lea     eax, dword ptr [ebp-24]
0042811C  |.  E>call    <unknown_libname_44>
00428121  |.  8>mov     edx, eax
00428123  |.  F>inc     dword ptr [ebp-3C]
00428126  |.  8>mov     ecx, dword ptr [ebp-5C]
00428129  |.  8>mov     eax, dword ptr [ecx+300]
0042812F  |.  E>call    <TControl::GetText(void)>
00428134  |.  8>lea     edx, dword ptr [ebp-24]
00428137  |.  F>push    dword ptr [edx]
00428139  |.  6>mov     word ptr [ebp-48], 50
0042813F  |.  B>mov     edx, 00523C14                 ;  ASCII "Name"
00428144  |.  8>lea     eax, dword ptr [ebp-20]
00428147  |.  E>call    <sub_509504>
0042814C  |.  F>inc     dword ptr [ebp-3C]
0042814F  |.  8>mov     edx, dword ptr [eax]
00428151  |.  8>mov     eax, dword ptr [ebp-64]
00428154  |.  5>pop     ecx
00428155  |.  E>call    <sub_47E824>
0042815A  |.  F>dec     dword ptr [ebp-3C]
0042815D  |.  8>lea     eax, dword ptr [ebp-24]
00428160  |.  B>mov     edx, 2
00428165  |.  E>call    <sub_509664>
```

## 第十七章 程序功能段的功能理解和提高方法

这里详细说明的原因，是因为对程序功能段的功能理解是破解水平和效率的表现，是初学算法分析的难点，也是众多破文根本不重视的地方。对程序功能段的功能的研判能力如何进行提高？

我个人的建议：

1. 初学者注意记忆和积累一些经典的功能段。如上述的判断是否为数字和写入注册信息的功能段，都是最常用和最基本的。在高手的破文中，有许多这方面的精彩，可以借鉴记忆。《加密和解密 II》中有较全面而系统的基本和常用的汇编功能段的阐述，熟读并理解记忆后，你再多做练习和总结，相信士别三日当刮目相看。

举例

小写转大写的程序段：

```
00401126  |.  0FBE8C05 44FF>movsx   ecx, byte ptr [ebp+eax-BC]       ;  用户名
0040112E  |.  83F9 61       cmp     ecx, 61
00401131  |.  7C 07         jl      short 0040113A                 ;  不是大写就转换成大写
00401133  |.  90            nop
00401134  |.  90            nop
00401135  |.  90            nop
00401136  |.  90            nop
00401137  |.  83E9 20       sub     ecx, 20
0040113A  |>  8BF1          mov     esi, ecx
```

2. 人门者最好学一门编程语言。自己编程一段单一的功能，自己反汇编研究，不断的总结、记忆和积累。

推荐一位破解模范的痴情之作：

> 【原创】菜鸟看懂算法以后之一：头痛的 64 次左移
> --------------------------------------------------------------------------------
> 标 题: 【原创】菜鸟看懂算法以后之一：头痛的 64 次左移
> 作 者: jney2
> 时 间: 2005-04-22,13:07
> 链 接: `http://bbs.pediy.com/showthread.php?t=13072`
> 菜鸟看懂算法以后之一：头痛的 64 次左移

好了，关于算法分析的要点和难点、相应的学习提高方法基本介绍差不多了。

## 第十八章 关键 `CALL` 的被调用情况分析方法

对于这个软件，我再进入关键 `CALL`，进行简要的分析。

首先，我们要了解此关键 `CALL` 的调用情况。

`OD` 的方法：

```
004262FC  >/$  5>push    ebp
```

信息窗口：

```
本地调用来自 00426B07, _TForm3_suiButton2Click+0DF
001.sub_4262FC
```

```
004262FD  |.  8>mov     ebp, esp
004262FF  |.  8>add     esp, -90
00426305  |.  5>push    esi
00426306  |.  5>push    edi
```
