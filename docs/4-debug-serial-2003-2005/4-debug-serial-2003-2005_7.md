
# 1toX 2.56

> 原文：[`www.pediy.com/kssd/pediy05/pediy50438.htm`](https://www.pediy.com/kssd/pediy05/pediy50438.htm)

简单算法分析——1toX 2.56

软件大小：约 800K

下载地址：`http://www.logipole.com`

【软件简介】：`1toX` is a 32 bit software for `Windows 9x`, `ME`, `2000` and `NT 4.x` used to split big files or a huge set of files into several smaller files. 文件分割与合并工具。

【软件限制】：30 天试用

【作者声明】：小弟初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请各大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`FI2.5`、`W32Dasm8.93` 黄金版

【过程】：

呵呵，从某张光盘里看见了这个家伙，想起 `xiA Qin` 大侠在《看雪论坛精华 3》中曾追过 1.63 版的注册码，却没有分析算法。虽然 2.56 版本也挺旧了，但是不妨碍我练练手呀。OK，开工吧！

`1toX.exe` 无壳，`VC++6.0` 编写。反汇编。`TRW` 伺候！

输入试炼信息：

Name: `fly`
First Name: `sky`
Key: `13572468`

`CTR+D` 切入 `TRW`，下万能断点 `BPX HMEMCPY`。`F5` 返回 `WINDOWS`，点“OK”，拦下！

`BD`，暂停断点。`PMODULE` 直达程序领空。我们来到 `425792` 处。

```
:00415792 85C0                    test eax, eax
=================================
====>我们来到这儿！

:00415794 755B                    jne 004157F1

…… ……省略…… ……
```

`F10` 走，呵呵，很快的，我们就来到了核心！

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041589E(C)
|
:004158FB 8D442418                lea eax, dword ptr [esp+18]
=================================
====>D EAX=fly

:004158FF 8D4C246C                lea ecx, dword ptr [esp+6C]
=================================
====>D ECX=sky

:00415903 50                      push eax

\* Possible StringData Ref from Data Obj ->"1toX"
=================================
====>注意此字符串！

:00415904 68F8834300              push 004383F8
=================================
====>D 4383F8=1toX

:00415909 51                      push ecx
:0041590A 8D942410070000          lea edx, dword ptr [esp+00000710]

\* Possible StringData Ref from Data Obj ->"%s%s%s"
=================================
|
:00415911 688C884300              push 0043888C
:00415916 52                      push edx
:00415917 FFD5                    call ebp
=================================
====>此 CALL 把 sky、1toX、fly 连接起来！

:00415919 8DBC2418070000          lea edi, dword ptr [esp+00000718]
=================================
====>D EDI=sky1toXfly

:00415920 83C9FF                  or ecx, FFFFFFFF
:00415923 33C0                    xor eax, eax
:00415925 83C414                  add esp, 00000014
:00415928 F2                      repnz
:00415929 AE                      scasb
:0041592A F7D1                    not ecx
:0041592C 2BF9                    sub edi, ecx
:0041592E 8D9424EC020000          lea edx, dword ptr [esp+000002EC]
:00415935 8BC1                    mov eax, ecx
:00415937 8BF7                    mov esi, edi
:00415939 8BFA                    mov edi, edx
:0041593B C1E902                  shr ecx, 02
:0041593E F3                      repz
:0041593F A5                      movsd
:00415940 8BC8                    mov ecx, eax
:00415942 8D8424EC020000          lea eax, dword ptr [esp+000002EC]
:00415949 83E103                  and ecx, 00000003
:0041594C F3                      repz
:0041594D A4                      movsb
:0041594E 8A8C24EC020000          mov cl, byte ptr [esp+000002EC]
:00415955 84C9                    test cl, cl
:00415957 741F                    je 00415978

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
下面开始循环了！

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00415976(C)
|
:00415959 80385F                  cmp byte ptr [eax], 5F
:0041595C 7503                    jne 00415961
:0041595E C60020                  mov byte ptr [eax], 20

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041595C(C)
|
:00415961 0FBE08                  movsx ecx, byte ptr [eax]
=================================
====>依次取 sky1toXfly 的 HEX 值
====>1、ECX=73
====>2、ECX=6B
====>3、ECX=79
…… ……省略…… ……

:00415964 334C2410                xor ecx, dword ptr [esp+10]
=================================
====>依次与[esp+10]异或！
请教一下[esp+10]的初始值是如何得出的？

====>1、ECX=73 XOR FFFFFFFF=FFFFFF8C
====>2、ECX=6B XOR ECA86542=ECA86529
====>3、ECX=79 XOR FFFFFFE7=FFFFFF9E
…… ……省略…… ……

:00415968 81F1CE9A5713            xor ecx, 13579ACE
=================================
====>上面得出的值依次与 13579ACE 异或！

====>1、ECX=FFFFFF8C XOR 13579ACE=ECA86542
====>2、ECX=ECA86529 XOR 13579ACE=FFFFFFE7
====>3、ECX=FFFFFF9E XOR 13579ACE=ECA86550
…… ……省略…… ……

:0041596E 40                      inc eax
:0041596F 894C2410                mov dword ptr [esp+10], ecx
=================================
====>ECX 依次入[esp+10]

====>循环最后 ECX=FFFFFF9F

:00415973 803800                  cmp byte ptr [eax], 00
:00415976 75E1                    jne 00415959
=================================
====>没取完？继续循环！

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00415957(C)
|
:00415978 8B442410                mov eax, dword ptr [esp+10]
=================================
====>FFFFFF9F 入 EAX

:0041597C 8D942414010000          lea edx, dword ptr [esp+00000114]
=================================
====>D EDX=13572468

:00415983 35F0BD6824              xor eax, 2468BDF0
=================================
====>EAX=FFFFFF9F XOR 2468BDF0=DB97426F
DB97426F 的十进制 3684123247 就是我们的真码了！

:00415988 52                      push edx
:00415989 A380EA4300              mov dword ptr [0043EA80], eax
=================================
====>DB97426F 入[0043EA80]

:0041598E E8E4400100              call 00429A77
:00415993 8B0D80EA4300            mov ecx, dword ptr [0043EA80]
=================================
====>DB97426F 入 ECX

:00415999 83C404                  add esp, 00000004
:0041599C 3BC1                    cmp eax, ecx
=================================
====>真假码比较！
====>？EAX=13572468    试炼码！
====>？ECX=3684123247  真码！！

:0041599E 7455                    je 004159F5
=================================
====>不跳则 OVER！

\* Reference To: USER32.MessageBoxA, Ord:01C3h

:004159D4 FF1590524300            Call dword ptr [00435290]
=================================
====>BAD BOY！
```

【KeyMake 之内存注册机】：

中断地址：`41599C`
中断次数：`1`
第一字节：`3B`
指令长度：`2`
寄存器方式：`ECX`

【注册信息保存】：

程序文件夹下的 `1toX.lic` 中。

```ini
[ENREGISTREMENT]
licence=3684123247
first name=fly
last name=sky
```

【整理】：

Name: `fly`
First Name: `sky`
Key: `3684123247`

Cracked By 巢水工作坊——fly【OCN】
2003-2-15 16:46 `4dd466d0e1e0cc123afbe504da2462ca`
