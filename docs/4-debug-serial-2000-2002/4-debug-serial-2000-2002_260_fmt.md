
# CPUCOOL 5.1000

> 原文：[`www.pediy.com/kssd/pediy03/forum543.htm`](https://www.pediy.com/kssd/pediy03/forum543.htm)

**文章名称**：`CPUCOOL 5.1000` 注册码分析

**作者**：夜月

**e_mail**: `luoyi.ly@yeah.net`

**oicq**: `36606500`

**目的**：帮助和我一样的初学者们。

**软件名称**：`CPUCOOL`

**软件简介**：一个 CPU 降温软件，也可以定时优化内存。具体的效果怎么样，我也不太清楚。（呵呵...没用温度计量过...)。

**保护分析**：这个软件在你输入注册码的时候会进行一次判断。如果你就只找到这一处判断的话，还不能注册成功。

另一个检测点说实在的，我也找了好久，主要原因有兴趣的朋友用 `W32dasm` 分析一下就知道了---- 虽然所有的对话框中的字符串，都能够在反汇编后找到，可是，它们的附近一个有用的跳转都没有，后来，找了好久用了 `bpm`，全局 flag,注册表监视，反正能想的招我都想了，才找到了第二个检测点。

还是有点运气的成分在里面的。它判断注册码的算法在反汇编后唯一的特点就是一个“%d”，两处检测点的字符串参考（`w32dasm` 内）都有这个“%d”。两个检测点找到了以后分析就是很简单的事了，不过这个软件的算法，需要一点浮点指令的知识。我以前没看过浮点指令的介绍，哪位大虾能 `e_mail` 给小弟一份的话，小弟感激不尽！不过，这里的两个浮点指令都比较简单，我看着它们的形式也就猜出意思了所以，如果你现在也不懂浮点指令的话，没关系的，猜吧！

## 教程正文

输入注册码，用 `trw2000` 的万能中断 `hmemcpy+pmodule` 一会儿还找不出关键的跳转，我是用 `bpm` 的方法找到了下面的这一处：

```
* Possible StringData Ref from Data Obj ->"%d"
|
:00414039 687C854600           push 0046857C
:0041403E 50                   push eax
:0041403F E80C2C0200           call 00436C50
:00414044 8B4C2414             mov ecx, dword ptr [esp+14]
:00414048 BF83000000           mov edi, 00000083
:0041404D 8BC1                 mov eax, ecx    <------eax 中保存了你输入的注册码（16 进制）
:0041404F 83C40C               add esp, 0000000C
:00414052 99                   cdq
:00414053 F7FF                 idiv edi
:00414055 85D2                 test edx, edx
:00414057 0F85C2000000         jne 0041411F    <-----注册码要是 83H 的整数倍
:0041405D B83DCB883E           mov eax, 3E88CB3D
:00414062 F7E9                 imul ecx
:00414064 C1FA05               sar edx, 05
:00414067 8BC2                 mov eax, edx
:00414069 C1E81F               shr eax, 1F
:0041406C 03D0                 add edx, eax
:0041406E 3BD7                 cmp edx, edi
:00414070 0F8EA9000000         jle 0041411F    <-----这一段是变相对注册码的大小进行判断
:00414076 81F980470100         cmp ecx, 00014780 <------14780H 是一个不能使用的号码！
:0041407C 7467                 je 004140E5
:0041407E 8BC1                 mov eax, ecx
:00414080 B97F000000           mov ecx, 0000007F
:00414085 99                   cdq
:00414086 F7F9                 idiv ecx    <------注册码不能是 7FH 的整数倍
:00414088 85D2                 test edx, edx
:0041408A 7459                 je 004140E5
:0041408C 8D542468             lea edx, dword ptr [esp+68]
:00414090 B9DC8D4700           mov ecx, 00478DDC
:00414095 52                   push edx
```

这里就是我说的第一处判断了。详细的分析，我已经写在旁边，相信有一点汇编基础的朋友就能够很轻松的看懂了。不过在那个变相判断注册码大小的地方，我还编了一个小程序算出来注册码必须大于 `252*131=33012`。有兴趣的朋友可以验证一下。

经过了这个“卡子”我们可以得出注册码必须要是 131 大于 251 的整数倍，而且，还不能被 `7FH` 整除，当然，还得排除一个作者已知的 `CRACK` 号码。下面，我们来到第二个检测点：

```
* Possible StringData Ref from Data Obj ->" %d"
|
:00421FA0 6864974600           push 00469764
:00421FA5 50                   push eax
:00421FA6 E8A54C0100           call 00436C50
:00421FAB 8B442424             mov eax, dword ptr [esp+24]
:00421FAF 83C40C               add esp, 0000000C
:00421FB2 99                   cdq
:00421FB3 F73DA88D4700         idiv dword ptr [00478DA8]      [478DA8]=1A19H=6681
:00421FB9 8BE8                 mov ebp, eax
:00421FBB 99                   cdq
:00421FBC 33C2                 xor eax, edx
:00421FBE 896C2418             mov dword ptr [esp+18], ebp
:00421FC2 2BC2                 sub eax, edx
:00421FC4 83E001               and eax, 00000001
:00421FC7 33C2                 xor eax, edx
:00421FC9 2BC2                 sub eax, edx
:00421FCB 742D                 je 00421FFA      上面一连串的位运算就是看是否是 6681 的奇数倍
:00421FCD DB442418             fild dword ptr [esp+18]  <----我猜应该是操作数被 LOAD
:00421FD1 32DB                 xor bl, bl
:00421FD3 D9FA                 fsqrt          <----我猜应该是开方的意思
:00421FD5 E8464B0100           call 00436B20
:00421FDA 8BF0                 mov esi, eax
:00421FDC B903000000           mov ecx, 00000003
:00421FE1 46                   inc esi
:00421FE2 3BF1                 cmp esi, ecx
:00421FE4 7E14                 jle 00421FFA
:00421FE6 8BC5                 mov eax, ebp
:00421FE8 99                   cdq
:00421FE9 F7F9                 idiv ecx
:00421FEB 85D2                 test edx, edx
:00421FED 7409                 je 00421FF8
:00421FEF 83C102               add ecx, 00000002
:00421FF2 3BCE                 cmp ecx, esi
:00421FF4 7CF0                 jl 00421FE6
:00421FF6 EB02                 jmp 00421FFA

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00421FED(C)
|
:00421FF8 B301                 mov bl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00421FCB(C), :00421FE4(C), :00421FF6(U)
|
:00421FFA 84DB                 test bl, bl
:00421FFC 7512                 jne 00422010
:00421FFE 6A20                 push 00000020
:00422000 8D4C2418             lea ecx, dword ptr [esp+18]
:00422004 E8B0D80200           call 0044F8B9
:00422009 68DC8D4700           push 00478DDC
:0042200E EB1E                 jmp 0042202E
```

上面这段代码很简单吧？呵呵...我的注释也就加的不多了。下面我就把注册码应该满足的条件列出来：

假设输入的数字为 `N`。则其必须满足：(用 C 语言的运算符表示）

1. `(N%131)=0`，`(N/131)>251;`
2. `(N%6681)=0.`
3. 设 `N/6681=M`.则 `M` 必须为奇数。
4. 设 `INT(SQRT(M))=K`,则：`K>2` 且从 3 到 `K+1` 所有的奇数都不能被 `M` 整除。

这就是我总结出来的 `CPUCOOL` 的注册码应该满足的规律，如有疏漏，尽请执教！
