
# 腾龙备份大师 2003 V3.05.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50651.htm`](https://www.pediy.com/kssd/pediy05/pediy50651.htm)

腾龙备份大师 2003 V3.05.01 专业版算法分析

**作者**：`wzh123`

**软件大小**：3030 KB

**软件语言**：简体中文

**软件类别**：国产软件 / 共享版 / 数据备份

**应用平台**：`Win9x`/`NT`/`2000`/`XP`

**软件介绍**：
全方位的数据备份保护系统“腾龙备份大师 2003”专业版隆重出场。适用于个人用户、企事业单位及政府机关使用的全新版本！最新编制的监控引擎，更低的系统资源占用率(1%-5%根据计算机配置)。为政府企业特别设计的自动数据锁定系统，可以广泛应用于政府网站保护及企业数据保护，有效防止因防火墙及操作系统漏洞而造成黑客成功入侵的数据损失！针对性的为用户设计了三大类十小类数据备份保护方法，以适应不同场合及不同人员对数据备份保护的需要！全新编写的内核代码、全新的操作界面、全新的向导界面，让每一个用户体验最便捷的操作感！最优惠的注册价格，让每一位用户都能够拥有安全的信息空间！

**PJ 工具**：`softice`，`W32Dasm8.93` 黄金版，`FI2.5`

**作者申明**：只是学习，无其他目的。

本人刚刚学破解，错误在所难免，写的也很乱，请各位包涵，也请各位高手指教

1、软件没有加壳，用 `delphi` 编的；

2、这是一个重启验证的软件，注册文件放在`\\winnt\\system32\\SYSTEMWIN32.dll`，可以用记事本打开。用 `softice` 下断，

序列号：`3781489924572`
注册名：`wzh123`
注册码：`a1234-b2345-c3456-d4567-5678`

你一定可以来到以下地方：(以下的分析都以我的注册信息为例子，大家可以根据自己的情况算出自己的注册码)

----------------------------------注册码第一部分计算----------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00506DE6(C)
|

:00506D94 8BC3                  mov eax, ebx
:00506D96 2501000080           and eax, 80000001
:00506D9B 7905                  jns 00506DA2
:00506D9D 48                    dec eax
:00506D9E 83C8FE               or eax, FFFFFFFE
:00506DA1 40                    inc eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00506D9B(C)
|

:00506DA2 85C0                  test eax, eax
:00506DA4 751F                  jne 00506DC5
:00506DA6 8D45CC                lea eax, dword ptr [ebp-34]
:00506DA9 50                    push eax
:00506DAA B901000000            mov ecx, 00000001
:00506DAF 8BD3                  mov edx, ebx
:00506DB1 8B45FC                mov eax, dword ptr [ebp-04]
:00506DB4 E83349F3FF            call 0043B6EC
:00506DB9 8B45CC                mov eax, dword ptr [ebp-34]
:00506DBC E89B2BF0FF            call 0040995C
:00506DC1 03F8                  add edi, eax
:00506DC3 EB1D                  jmp 00506DE2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00506DA4(C)
|

:00506DC5 8D45C8                lea eax, dword ptr [ebp-38]
:00506DC8 50                    push eax
:00506DC9 B901000000            mov ecx, 00000001
:00506DCE 8BD3                  mov edx, ebx
:00506DD0 8B45FC                mov eax, dword ptr [ebp-04]
:00506DD3 E81449F3FF            call 0043B6EC
:00506DD8 8B45C8                mov eax, dword ptr [ebp-38]
:00506DDB E87C2BF0FF            call 0040995C
:00506DE0 03F0                  add esi, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00506DC3(U)
|

:00506DE2 43                    inc ebx
:00506DE3 83FB0E                cmp ebx, 0000000E
:00506DE6 75AC                  jne 00506D94  ----------------以上将给定的序列号的奇、偶数位分别相加，将奇数位相加的结果-->`esi`，将偶数位相加的结果-->`edi`

(以我的序列号为例 `3781489924572`，`3+8+4+9+2+5+2=0x21`==>`esi`, `7+1+8+9+4+7=0x24`==>`edi`)

:00506DE8 8D55C4                lea edx, dword ptr [ebp-3C]
:00506DEB 8BC7                  mov eax, edi             偶数位相加的结果 `0x24`-->`eax`
:00506DED 0FAFC6                imul eax, esi            偶数位相加的结果*奇数位相加的结果 `0x4a4`-->`eax`
:00506DF0 E8FB29F0FF            call 004097F0            `0x4a4`-->`1188(H)`
:00506DF5 8B45C4                mov eax, dword ptr [ebp-3C]
:00506DF8 8D4DE8                lea ecx, dword ptr [ebp-18]
:00506DFB BA05000000            mov edx, 00000005
:00506E00 E8B3EDFFFF            call 00505BB8            `1188`-->`11880`
:00506E05 8D55BC                lea edx, dword ptr [ebp-44]
:00506E08 8B45F8                mov eax, dword ptr [ebp-08]  注册名`"wzh123"`-->`eax`
:00506E0B E8CCEEFFFF            call 00505CDC            注册名转换
:00506E10 8B45BC                mov eax, dword ptr [ebp-44]  `71610`-->`eax`
:00506E13 8D55C0                lea edx, dword ptr [ebp-40]
:00506E16 E89DECFFFF            call 00505AB8
:00506E1B 8B45C0                mov eax, dword ptr [ebp-40]  `71610`-->`eax`
:00506E1E 8D4DE4                lea ecx, dword ptr [ebp-1C]
:00506E21 BA05000000            mov edx, 00000005
:00506E26 E88DEDFFFF            call 00505BB8
:00506E2B 8B45E8                mov eax, dword ptr [ebp-18]  `11880`-->`eax`
:00506E2E E8292BF0FF            call 0040995C            `11880(D)`-->`2E68(H)`
:00506E33 50                    push eax
:00506E34 8B45E4                mov eax, dword ptr [ebp-1C]  `71610`-->`eax`
:00506E37 E8202BF0FF            call 0040995C            `71610(D)`-->`117BA(H)`
:00506E3C 5A                    pop edx
:00506E3D 92                    xchg eax,edx             `2E68(H)`-->`eax`, `117BA(H)`-->`edx`
:00506E3E 8BCA                  mov ecx, edx
:00506E40 99                    cdq
:00506E41 F7F9                  idiv ecx                 `2E68(H)/117BA`
:00506E43 8BC2                  mov eax, edx             余数(`0x2E68`)-->`eax`
:00506E45 05E7030000            add eax, 000003E7        `0x2E68+0x3E7=0x324F`-->`eax`
:00506E4A 8D55B4                lea edx, dword ptr [ebp-4C]
:00506E4D E89E29F0FF            call 004097F0            `0x324F`-->`12879(D)`
:00506E52 8B45B4                mov eax, dword ptr [ebp-4C]  `12879(D)`-->`eax`
:00506E55 8D4DB8                lea ecx, dword ptr [ebp-48]
:00506E58 BA04000000            mov edx, 00000004
:00506E5D E856EDFFFF            call 00505BB8            `12879(D)`-->`1287`
:00506E62 8B45B8                mov eax, dword ptr [ebp-48]  `1287`-->`eax`
:00506E65 8D55EC                lea edx, dword ptr [ebp-14]
:00506E68 E87F090000            call 005077EC
:00506E6D 8D55B0                lea edx, dword ptr [ebp-50]
:00506E70 8B45EC                mov eax, dword ptr [ebp-14]
:00506E73 E874090000            call 005077EC
:00506E78 8B45B0                mov eax, dword ptr [ebp-50]
:00506E7B E8DC2AF0FF            call 0040995C            `1287(D)`-->`507(H)`
:00506E80 8945D0                mov dword ptr [ebp-30], eax
:00506E83 8D55A0                lea edx, dword ptr [ebp-60]
:00506E86 8B45EC                mov eax, dword ptr [ebp-14]
:00506E89 E85E090000            call 005077EC            取 `507` 最后一位`"7"`
:00506E8E 8B45A0                mov eax, dword ptr [ebp-60]
:00506E91 8D4DA4                lea ecx, dword ptr [ebp-5C]
:00506E94 BA01000000            mov edx, 00000001
:00506E99 E8EE22FEFF            call 004E918C
:00506E9E 8B45A4                mov eax, dword ptr [ebp-5C]  `"7"`-->[`eax`]
:00506EA1 E8B62AF0FF            call 0040995C            `7`-->`eax`
:00506EA6 8BD0                  mov edx, eax
:00506EA8 83C241                add edx, 00000041        `7+41=0x48` 即`"H"`-->`edx`
:00506EAB 8D45A8                lea eax, dword ptr [ebp-58]
:00506EAE E89DDFEFFF            call 00404E50
:00506EB3 8D45A8                lea eax, dword ptr [ebp-58]
:00506EB6 50                    push eax
:00506EB7 8D559C                lea edx, dword ptr [ebp-64]
:00506EBA 8B45EC                mov eax, dword ptr [ebp-14]
:00506EBD E82A090000            call 005077EC
:00506EC2 8B559C                mov edx, dword ptr [ebp-64]  `1287`-->`edx`
:00506EC5 58                    pop eax
:00506EC6 E865E0EFFF            call 00404F30            将 `1287` 与`"H"`连起来得到字串 `"H1287"`--------第一部分的真注册码出现
:00506ECB 8B45A8                mov eax, dword ptr [ebp-58]
:00506ECE 8D55AC                lea edx, dword ptr [ebp-54]
:00506ED1 E816090000            call 005077EC
:00506ED6 8B55AC                mov edx, dword ptr [ebp-54]



:00506ED9 8D45EC `lea eax, dword ptr [ebp-14]`
:00506EDC E81FDEEFFF `call 00404D00`
:00506EE1 8D5590 `lea edx, dword ptr [ebp-70]`
:00506EE4 8B45EC `mov eax, dword ptr [ebp-14]`
:00506EE7 E800090000 `call 005077EC`
:00506EEC 8B4590 `mov eax, dword ptr [ebp-70]`
:00506EEF 8D4D94 `lea ecx, dword ptr [ebp-6C]`
:00506EF2 BA01000000 `mov edx, 00000001`
:00506EF7 E89022FeFF `call 004E918C`
:00506EFC 8B4594 `mov eax, dword ptr [ebp-6C]`
:00506EFF E8582AF0FF `call 0040995C`
:00506F04 83C041 `add eax, 00000041`
:00506F07 8D5598 `lea edx, dword ptr [ebp-68]`
:00506F0A E8E128F0FF `call 004097F0`
:00506F0F 8D4598 `lea eax, dword ptr [ebp-68]`
:00506F12 50 `push eax`
:00506F13 8D558C `lea edx, dword ptr [ebp-74]`
:00506F16 8B45D0 `mov eax, dword ptr [ebp-30]`
:00506F19 E8D228F0FF `call 004097F0`
:00506F1E 8B558C `mov edx, dword ptr [ebp-74]`
:00506F21 58 `pop eax`
:00506F22 E809E0EFFF `call 00404F30` 1287-->721287(下面有用)
:00506F27 8B4598 `mov eax, dword ptr [ebp-68]`
:00506F2A E82D2AF0FF `call 0040995C` 187-->`eax`
:00506F2F 8945D0 `mov dword ptr [ebp-30], eax`
:00506F32 8D5588 `lea edx, dword ptr [ebp-78]`
:00506F35 8B45D0 `mov eax, dword ptr [ebp-30]`
:00506F38 E8B328F0FF `call 004097F0`
:00506F3D 8B4588 `mov eax, dword ptr [ebp-78]`
:00506F40 8D55DC `lea edx, dword ptr [ebp-24]`
:00506F43 E8A4080000 `call 005077EC` 将真注册码的第一部分各位取反
:00506F48 8D4D80 `lea ecx, dword ptr [ebp-80]`
:00506F4B BA05000000 `mov edx, 00000005`
:00506F50 8B45F4 `mov eax, dword ptr [ebp-0C]` 取第一部分的假码
:00506F53 E8A046F3FF `call 0043B5F8`
:00506F58 8B4580 `mov eax, dword ptr [ebp-80]`
:00506F5B 8D5584 `lea edx, dword ptr [ebp-7C]`
:00506F5E E889080000 `call 005077EC` 将输入注册码的第一部分各位取反
:00506F63 8B5584 `mov edx, dword ptr [ebp-7C]`
:00506F66 8B45EC `mov eax, dword ptr [ebp-14]`
:00506F69 E806E1EFFF `call 00405074` 第一部分的经过变换的真假注册码相比
:00506F6E 7409 `je 00506F79` 相等就跳到注册码第二部分的计算，否则去死(爆破点)
:00506F70 C645F300 `mov [ebp-0D], 00`
:00506F74 E946060000 `jmp 005075BF`

－－－－－－－－－－－－注册码第二部分计算－－－－－－－－－－－－－－－

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00506F6E(C)

|

:00506F79 8D45E0 `lea eax, dword ptr [ebp-20]`
:00506F7C E8E7DCEFFF `call 00404C68`
:00506F81 BB01000000 `mov ebx, 00000001`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00506FF6(C)

|

:00506F86 8D8578FFFFFF `lea eax, dword ptr [ebp+FFFFFF78]`
:00506F8C 50 `push eax`
:00506F8D B901000000 `mov ecx, 00000001`
:00506F92 8BD3 `mov edx, ebx`
:00506F94 8B45E8 `mov eax, dword ptr [ebp-18]` 11880(见上)-->`eax`
:00506F97 E85047F3FF `call 0043B6EC`
:00506F9C 8B8578FFFFFF `mov eax, dword ptr [ebp+FFFFFF78]`
:00506FA2 E8B529F0FF `call 0040995C`
:00506FA7 8BF0 `mov esi, eax`
:00506FA9 8D8574FFFFFF `lea eax, dword ptr [ebp+FFFFFF74]`
:00506FAF 50 `push eax`
:00506FB0 8D5301 `lea edx, dword ptr [ebx+01]`
:00506FB3 B901000000 `mov ecx, 00000001`
:00506FB8 8B45E8 `mov eax, dword ptr [ebp-18]`
:00506FBB E82C47F3FF `call 0043B6EC`
:00506FC0 8B8574FFFFFF `mov eax, dword ptr [ebp+FFFFFF74]`
:00506FC6 E89129F0FF `call 0040995C`
:00506FCB 03F0 `add esi, eax`
:00506FCD 8BC6 `mov eax, esi`
:00506FCF B90A000000 `mov ecx, 0000000A`
:00506FD4 99 `cdq`
:00506FD5 F7F9 `idiv ecx`
:00506FD7 8BC2 `mov eax, edx`
:00506FD9 8D957CFFFFFF `lea edx, dword ptr [ebp+FFFFFF7C]`
:00506FDF E80C28F0FF `call 004097F0`
:00506FE4 8B957CFFFFFF `mov edx, dword ptr [ebp+FFFFFF7C]`
:00506FEA 8D45E0 `lea eax, dword ptr [ebp-20]`
:00506FED E83EDFEFFF `call 00404F30`
:00506FF2 43 `inc ebx`
:00506FF3 83FB05 `cmp ebx, 00000005`
:00506FF6 758E `jne 00506F86`
以上构成循环，将 11880 两位一组合，然后除 `0xA`，余数保存起来，如
1、`(1+1)%0xA="2"`
2、`(1+8)%0xA="9"`
3、`(8+8)%0xA="6"`
4、`(8+0)%0xA="8"`

:00506FF8 33F6 `xor esi, esi`
:00506FFA BB01000000 `mov ebx, 00000001`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00507026(C)

|

:00506FFF 8D8570FFFFFF `lea eax, dword ptr [ebp+FFFFFF70]`
:00507005 50 `push eax`
:00507006 B901000000 `mov ecx, 00000001`
:0050700B 8BD3 `mov edx, ebx`
:0050700D 8B45E0 `mov eax, dword ptr [ebp-20]`
:00507010 E8D746F3FF `call 0043B6EC`
:00507015 8B8570FFFFFF `mov eax, dword ptr [ebp+FFFFFF70]`
:0050701B E83C29F0FF `call 0040995C`
:00507020 03F0 `add esi, eax`
:00507022 43 `inc ebx`
:00507023 83FB05 `cmp ebx, 00000005`
:00507026 75D7 `jne 00506FFF`
又一个循环，将以上得到的余数相加，即 `2+9+6+8=0x19`---->`esi`

:00507028 8BC6 `mov eax, esi`
:0050702A B90A000000 `mov ecx, 0000000A`
:0050702F 99 `cdq`
:00507030 F7F9 `idiv ecx` `0x19/0xA`
:00507032 8BC2 `mov eax, edx` 余数"5"-->`eax`
:00507034 8D9564FFFFFF `lea edx, dword ptr [ebp+FFFFFF64]`
:0050703A E8B127F0FF `call 004097F0`
:0050703F 8B8D64FFFFFF `mov ecx, dword ptr [ebp+FFFFFF64]`
:00507045 8D8568FFFFFF `lea eax, dword ptr [ebp+FFFFFF68]`
:0050704B 8B55E0 `mov edx, dword ptr [ebp-20]`
:0050704E E821DFEFFF `call 00404F74` 将"2968"与"5"相连得到第二部分的真注册码"29685"
:00507053 8B8568FFFFFF `mov eax, dword ptr [ebp+FFFFFF68]`
:00507059 8D956CFFFFFF `lea edx, dword ptr [ebp+FFFFFF6C]`
:0050705F E888070000 `call 005077EC`
:00507064 8B956CFFFFFF `mov edx, dword ptr [ebp+FFFFFF6C]`
:0050706A 8D45E0 `lea eax, dword ptr [ebp-20]`
:0050706D E88EDCEFFF `call 00404D00`
:00507072 8D45EC `lea eax, dword ptr [ebp-14]`
:00507075 8B55E0 `mov edx, dword ptr [ebp-20]`
:00507078 E883DCEFFF `call 00404D00`
:0050707D 8D9560FFFFFF `lea edx, dword ptr [ebp+FFFFFF60]`
:00507083 8B45EC `mov eax, dword ptr [ebp-14]`
:00507086 E861070000 `call 005077EC`
:0050708B 8B8560FFFFFF `mov eax, dword ptr [ebp+FFFFFF60]`
:00507091 E8C628F0FF `call 0040995C`
:00507096 0145D0 `add dword ptr [ebp-30], eax`



:00507099 8D45D8              lea eax, dword ptr [ebp-28]

:0050709C 8B55EC              mov edx, dword ptr [ebp-14]

:0050709F E85CDCEFFF          call 00404D00

:005070A4 8D8558FFFFFF        lea eax, dword ptr [ebp+FFFFFF58]

:005070AA 50                  push eax

:005070AB B905000000          mov ecx, 00000005

:005070B0 BA07000000          mov edx, 00000007

:005070B5 8B45F4              mov eax, dword ptr [ebp-0C]

:005070B8 E82F46F3FF          call 0043B6EC                          取第二部分的假码

:005070BD 8B8558FFFFFF        mov eax, dword ptr [ebp+FFFFFF58]

:005070C3 8D955CFFFFFF        lea edx, dword ptr [ebp+FFFFFF5C]

:005070C9 E81E070000          call 005077EC                          假码各位取反

:005070CE 8B955CFFFFFF        mov edx, dword ptr [ebp+FFFFFF5C]

:005070D4 8B45EC              mov eax, dword ptr [ebp-14]

:005070D7 E898DFEFFF          call 00405074                          第二部分的经过变换的真假注册码相比

:005070DC 7409                je 005070E7                            相等就跳到注册码第三部分的计算，否则去死(爆破点)

:005070DE C645F300            mov [ebp-0D], 00

:005070E2 E9D8040000          jmp 005075BF

－－－－－－－－－－－－注册码第三部分计算－－－－－－－－－－－－－－－

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005070DC(C)

|

:005070E7 8D45E0              lea eax, dword ptr [ebp-20]

:005070EA E879DBEFFF          call 00404C68

:005070EF BB01000000          mov ebx, 00000001

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0050715E(C)

|

:005070F4 8D8550FFFFFF        lea eax, dword ptr [ebp+FFFFFF50]

:005070FA 50                  push eax

:005070FB B901000000          mov ecx, 00000001

:00507100 8BD3                mov edx, ebx

:00507102 8B45E8              mov eax, dword ptr [ebp-18]           11880-->eax

:00507105 E8E245F3FF          call 0043B6EC

:0050710A 8B8550FFFFFF        mov eax, dword ptr [ebp+FFFFFF50]

:00507110 E84728F0FF          call 0040995C

:00507115 50                  push eax

:00507116 8D854CFFFFFF        lea eax, dword ptr [ebp+FFFFFF4C]

:0050711C 50                  push eax

:0050711D 8D5301              lea edx, dword ptr [ebx+01]

:00507120 B901000000          mov ecx, 00000001

:00507125 8B45E8              mov eax, dword ptr [ebp-18]

:00507128 E8BF45F3FF          call 0043B6EC

:0050712D 8B854CFFFFFF        mov eax, dword ptr [ebp+FFFFFF4C]

:00507133 E82428F0FF          call 0040995C

:00507138 5A                  pop edx

:00507139 92                  xchg eax,edx

:0050713A 2BC2                sub eax, edx

:0050713C 99                  cdq

:0050713D 33C2                xor eax, edx

:0050713F 2BC2                sub eax, edx

:00507141 8D9554FFFFFF        lea edx, dword ptr [ebp+FFFFFF54]

:00507147 E8A426F0FF          call 004097F0

:0050714C 8B9554FFFFFF        mov edx, dword ptr [ebp+FFFFFF54]

:00507152 8D45E0              lea eax, dword ptr [ebp-20]

:00507155 E8D6DDEFFF          call 00404F30

:0050715A 43                  inc ebx

:0050715B 83FB05              cmp ebx, 00000005

:0050715E 7594                jne 005070F4------------------------以上构成循环，将 11880 各位两两相减，得出一组数字，

1、1-1=0

2、8-1=7

3、8-8=0

4、8-0=8 (0708)

:00507160 BE01000000          mov esi, 00000001

:00507165 BB01000000          mov ebx, 00000001

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005071B8(C)

|

:0050716A 8D8548FFFFFF        lea eax, dword ptr [ebp+FFFFFF48]

:00507170 50                  push eax

:00507171 B901000000          mov ecx, 00000001

:00507176 8BD3                mov edx, ebx

:00507178 8B45E0              mov eax, dword ptr [ebp-20]            0708--->eax

:0050717B E86C45F3FF          call 0043B6EC

:00507180 8B8548FFFFFF        mov eax, dword ptr [ebp+FFFFFF48]

:00507186 E8D127F0FF          call 0040995C

:0050718B 85C0                test eax, eax

:0050718D 7425                je 005071B4

:0050718F 8D8544FFFFFF        lea eax, dword ptr [ebp+FFFFFF44]

:00507195 50                  push eax

:00507196 B901000000          mov ecx, 00000001

:0050719B 8BD3                mov edx, ebx

:0050719D 8B45E0              mov eax, dword ptr [ebp-20]

:005071A0 E84745F3FF          call 0043B6EC

:005071A5 8B8544FFFFFF        mov eax, dword ptr [ebp+FFFFFF44]

:005071AB E8AC27F0FF          call 0040995C

:005071B0 F7EE                imul esi

:005071B2 8BF0                mov esi, eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0050718D(C)

|

:005071B4 43                  inc ebx

:005071B5 83FB05              cmp ebx, 00000005

:005071B8 75B0                jne 0050716A------------------------又是一个循环，将 0708 进行处理，如果遇到 0，则不处理，遇到其他数字，进行如下处理：

:005071BA 8BC6                mov eax, esi

:005071BC B90A000000          mov ecx, 0000000A

:005071C1 99                  cdq

:005071C2 F7F9                idiv ecx

:005071C4 8BC2                mov eax, edx                            如：(7*8)%0xA=6

:005071C6 8D953CFFFFFF        lea edx, dword ptr [ebp+FFFFFF3C]

:005071CC E81F26F0FF          call 004097F0

:005071D1 8D853CFFFFFF        lea eax, dword ptr [ebp+FFFFFF3C]

:005071D7 8B55E0              mov edx, dword ptr [ebp-20]

:005071DA E851DDEFFF          call 00404F30                            将 0708 与 6 连接起来得到第三部分真注册码"60708"

:005071DF 8B853CFFFFFF        mov eax, dword ptr [ebp+FFFFFF3C]

:005071E5 8D9540FFFFFF        lea edx, dword ptr [ebp+FFFFFF40]

:005071EB E8FC050000          call 005077EC

:005071F0 8B9540FFFFFF        mov edx, dword ptr [ebp+FFFFFF40]

:005071F6 8D45E0              lea eax, dword ptr [ebp-20]

:005071F9 E802DBEFFF          call 00404D00

:005071FE 8D45EC              lea eax, dword ptr [ebp-14]

:00507201 8B55E0              mov edx, dword ptr [ebp-20]

:00507204 E8F7DAEFFF          call 00404D00

:00507209 8D45D4              lea eax, dword ptr [ebp-2C]

:0050720C 8B55EC              mov edx, dword ptr [ebp-14]

:0050720F E8ECDAEFFF          call 00404D00

:00507214 8D9538FFFFFF        lea edx, dword ptr [ebp+FFFFFF38]

:0050721A 8B45EC              mov eax, dword ptr [ebp-14]

:0050721D E8CA050000          call 005077EC

:00507222 8B8538FFFFFF        mov eax, dword ptr [ebp+FFFFFF38]

:00507228 E82F27F0FF          call 0040995C

:0050722D 0145D0              add dword ptr [ebp-30], eax

:00507230 8D8530FFFFFF        lea eax, dword ptr [ebp+FFFFFF30]

:00507236 50                  push eax

:00507237 B905000000          mov ecx, 00000005

:0050723C BA0D000000          mov edx, 0000000D

:00507241 8B45F4              mov eax, dword ptr [ebp-0C]

:00507244 E8A344F3FF          call 0043B6EC                            取第三部分的假码

:00507249 8B8530FFFFFF        mov eax, dword ptr [ebp+FFFFFF30]

:0050724F 8D9534FFFFFF        lea edx, dword ptr [ebp+FFFFFF34]

:00507255 E892050000          call 005077EC

:0050725A 8B9534FFFFFF        mov edx, dword ptr [ebp+FFFFFF34]

:00507260 8B45EC              mov eax, dword ptr [ebp-14]

:00507263 E80CDEEFFF          call 00405074                            第三部分的经过变换的真假注册码相比



# 00507268

```
:00507268 7409                 je 00507273
```
相等就跳到注册码第四部分的计算，否则去死（爆破点）

# 0050726A

```
:0050726A C645F300             mov [ebp-0Dh], 00h
:0050726E E94C030000           jmp 005075BF
```

---

## 注册码第四部分计算

> **Referenced by a (U)nconditional or (C)onditional Jump at Address:**
>
> |:00507268(C)

```
:00507273 8D9528FFFFFF         lea edx, dword ptr [ebp+FFFFFF28]
:00507279 8B45D0               mov eax, dword ptr [ebp-30h]
:0050727C E86F25F0FF           call 004097F0
:00507281 8B8528FFFFFF         mov eax, dword ptr [ebp+FFFFFF28] ; 811680-->eax
:00507287 8D8D2CFFFFFF         lea ecx, dword ptr [ebp+FFFFFF2C]
:0050728D BA05000000           mov edx, 00000005
:00507292 E821E9FFFF           call 00505BB8 ; 811680-->81168
:00507297 8B852CFFFFFF         mov eax, dword ptr [ebp+FFFFFF2C] ; "81168"-->eax, 即第四部分真注册码
:0050729D 8D55EC               lea edx, dword ptr [ebp-14h]
:005072A0 E847050000           call 005077EC ; 真注册码各位取反
:005072A5 8D8520FFFFFF         lea eax, dword ptr [ebp+FFFFFF20]
:005072AB 50                   push eax
:005072AC B905000000           mov ecx, 00000005
:005072B1 BA13000000           mov edx, 00000013
:005072B6 8B45F4               mov eax, dword ptr [ebp-0Ch]
:005072B9 E82E44F3FF           call 0043B6EC ; 取第四部分的假码
:005072BE 8B8520FFFFFF         mov eax, dword ptr [ebp+FFFFFF20]
:005072C4 8D9524FFFFFF         lea edx, dword ptr [ebp+FFFFFF24]
:005072CA E81D050000           call 005077EC ; 第四部分的假码各位取反
:005072CF 8B9524FFFFFF         mov edx, dword ptr [ebp+FFFFFF24]
:005072D5 8B45EC               mov eax, dword ptr [ebp-14h]
:005072D8 E897DDEFFF           call 00405074 ; 第四部分的经过变换的真假注册码相比
:005072DD 7409                 je 005072E8 ; 相等就跳到注册码第五部分的计算，否则去死（爆破点）
:005072DF C645F300             mov [ebp-0Dh], 00h
:005072E3 E9D7020000           jmp 005075BF
```

---

## 注册码第五部分计算

> **Referenced by a (U)nconditional or (C)onditional Jump at Address:**
>
> |:005072DD(C)

```
:005072E8 33F6                 xor esi, esi ; esi 清零
:005072EA BB01000000           mov ebx, 00000001
```

> **Referenced by a (U)nconditional or (C)onditional Jump at Address:**
>
> |:00507327(C)

```
:005072EF 8D851CFFFFFF         lea eax, dword ptr [ebp+FFFFFF1C]
:005072F5 50                   push eax
:005072F6 8D9518FFFFFF         lea edx, dword ptr [ebp+FFFFFF18]
:005072FC 8B45EC               mov eax, dword ptr [ebp-14h]
:005072FF E8E8040000           call 005077EC
:00507304 8B8518FFFFFF         mov eax, dword ptr [ebp+FFFFFF18] ; 81168-->eax
:0050730A B901000000           mov ecx, 00000001
:0050730F 8BD3                 mov edx, ebx
:00507311 E8D643F3FF           call 0043B6EC
:00507316 8B851CFFFFFF         mov eax, dword ptr [ebp+FFFFFF1C]
:0050731C E83B26F0FF           call 0040995C
:00507321 03F0                 add esi, eax
:00507323 43                   inc ebx
:00507324 83FB06               cmp ebx, 00000006
:00507327 75C6                 jne 005072EF ; 以上构成循环，将 81168 各位相加，即 8+1+1+6+8=0x18---->esi
:00507329 8BC6                 mov eax, esi
:0050732B B90A000000           mov ecx, 0000000A
:00507330 99                   cdq
:00507331 F7F9                 idiv ecx ; 0x18/0xA
:00507333 8BF2                 mov esi, edx ; 余数为"4"-->esi
:00507335 8D55E0               lea edx, dword ptr [ebp-20h]
:00507338 8BC6                 mov eax, esi
:0050733A E8B124F0FF           call 004097F0
:0050733F 33F6                 xor esi, esi
:00507341 BB01000000           mov ebx, 00000001
```

> **Referenced by a (U)nconditional or (C)onditional Jump at Address:**
>
> |:0050737E(C)

```
:00507346 8D8514FFFFFF         lea eax, dword ptr [ebp+FFFFFF14]
:0050734C 50                   push eax
:0050734D 8D9510FFFFFF         lea edx, dword ptr [ebp+FFFFFF10]
:00507353 8B45DC               mov eax, dword ptr [ebp-24h]
:00507356 E891040000           call 005077EC
:0050735B 8B8510FFFFFF         mov eax, dword ptr [ebp+FFFFFF10] ; 721287(见上)-->eax
:00507361 B901000000           mov ecx, 00000001
:00507366 8BD3                 mov edx, ebx
:00507368 E87F43F3FF           call 0043B6EC
:0050736D 8B8514FFFFFF         mov eax, dword ptr [ebp+FFFFFF14]
:00507373 E8E425F0FF           call 0040995C
:00507378 03F0                 add esi, eax
:0050737A 43                   inc ebx
:0050737B 83FB07               cmp ebx, 00000007
:0050737E 75C6                 jne 00507346 ; 又一个循环，将 721287 各位相加，即 7+2+1+2+8+7=0x1B----->esi
:00507380 8BC6                 mov eax, esi
:00507382 B90A000000           mov ecx, 0000000A
:00507387 99                   cdq
:00507388 F7F9                 idiv ecx ; 0x1B/0xA
:0050738A 8BF2                 mov esi, edx ; 余数为"7"-->esi
:0050738C 8D950CFFFFFF         lea edx, dword ptr [ebp+FFFFFF0C]
:00507392 8BC6                 mov eax, esi
:00507394 E85724F0FF           call 004097F0
:00507399 8B850CFFFFFF         mov eax, dword ptr [ebp+FFFFFF0C]
:0050739F 8D55EC               lea edx, dword ptr [ebp-14h]
:005073A2 E845040000           call 005077EC
:005073A7 33F6                 xor esi, esi
:005073A9 BB01000000           mov ebx, 00000001
```

> **Referenced by a (U)nconditional or (C)onditional Jump at Address:**
>
> |:005073E6(C)

```
:005073AE 8D8508FFFFFF         lea eax, dword ptr [ebp+FFFFFF08]
:005073B4 50                   push eax
:005073B5 8D9504FFFFFF         lea edx, dword ptr [ebp+FFFFFF04]
:005073BB 8B45D8               mov eax, dword ptr [ebp-28h]
:005073BE E829040000           call 005077EC ; 得到"29685"(见上)
:005073C3 8B8504FFFFFF         mov eax, dword ptr [ebp+FFFFFF04] ; 29685-->eax
:005073C9 B901000000           mov ecx, 00000001
:005073CE 8BD3                 mov edx, ebx
:005073D0 E81743F3FF           call 0043B6EC
:005073D5 8B8508FFFFFF         mov eax, dword ptr [ebp+FFFFFF08]
:005073DB E87C25F0FF           call 0040995C
:005073E0 03F0                 add esi, eax
:005073E2 43                   inc ebx
:005073E3 83FB06               cmp ebx, 00000006
:005073E6 75C6                 jne 005073AE ; 又一个循环,将 29685 各位相加，即 2+9+6+8+5=0x1E----->esi
:005073E8 8BC6                 mov eax, esi
:005073EA B90A000000           mov ecx, 0000000A
:005073EF 99                   cdq
:005073F0 F7F9                 idiv ecx ; 0x1E/0xA
:005073F2 8BF2                 mov esi, edx ; 余数为"0"-->esi
:005073F4 8D95FCFEFFFF         lea edx, dword ptr [ebp+FFFFFEFC]
:005073FA 8B45EC               mov eax, dword ptr [ebp-14h]
:005073FD E8EA030000           call 005077EC
:00507402 8D85FCFEFFFF         lea eax, dword ptr [ebp+FFFFFEFC]
:00507408 50                   push eax
:00507409 8D95F8FEFFFF         lea edx, dword ptr [ebp+FFFFFEF8]
:0050740F 8BC6                 mov eax, esi
:00507411 E8DA23F0FF           call 004097F0
:00507416 8B95F8FEFFFF         mov edx, dword ptr [ebp+FFFFFEF8]
:0050741C 58                   pop eax
```



:0050741D `E80EDBEFFF` `call 00404F30` 将余数"7"与余数"0"连接起来----->"70"

:00507422 `8B85FCFEFFFF` `mov eax, dword ptr [ebp+FFFFFEFC]`

:00507428 `8D9500FFFFFF` `lea edx, dword ptr [ebp+FFFFFF00]`

:0050742E `E8B9030000` `call 005077EC`

:00507433 `8B9500FFFFFF` `mov edx, dword ptr [ebp+FFFFFF00]`

:00507439 `8D45EC` `lea eax, dword ptr [ebp-14]`

:0050743C `E8BFD8EFFF` `call 00404D00`

:00507441 `33F6` `xor esi, esi` `esi` 清零

:00507443 `BB01000000` `mov ebx, 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00507480(C)`
|

:00507448 `8D85F4FEFFFF` `lea eax, dword ptr [ebp+FFFFFEF4]`

:0050744E `50` `push eax`

:0050744F `8D95F0FEFFFF` `lea edx, dword ptr [ebp+FFFFFEF0]`

:00507455 `8B45D4` `mov eax, dword ptr [ebp-2C]`

:00507458 `E88F030000` `call 005077EC`

:0050745D `8B85F0FEFFFF` `mov eax, dword ptr [ebp+FFFFFEF0]` "60708"(见上)--->`eax`

:00507463 `B901000000` `mov ecx, 00000001`

:00507468 `8BD3` `mov edx, ebx`

:0050746A `E87D42F3FF` `call 0043B6EC`

:0050746F `8B85F4FEFFFF` `mov eax, dword ptr [ebp+FFFFFEF4]`

:00507475 `E8E224F0FF` `call 0040995C`

:0050747A `03F0` `add esi, eax`

:0050747C `43` `inc ebx`

:0050747D `83FB06` `cmp ebx, 00000006`

:00507480 `75C6` `jne 00507448` 又一个循环,将 60708 各位相加，即 6+0+7+0+8=0x15----->`esi`

:00507482 `8BC6` `mov eax, esi`

:00507484 `B90A000000` `mov ecx, 0000000A`

:00507489 `99` `cdq`

:0050748A `F7F9` `idiv ecx` 0x15/0xA

:0050748C `8BF2` `mov esi, edx` 余数为"1"--->`esi`

:0050748E `8D95E4FEFFFF` `lea edx, dword ptr [ebp+FFFFFEE4]`

:00507494 `8B45EC` `mov eax, dword ptr [ebp-14]`

:00507497 `E850030000` `call 005077EC`

:0050749C `FFB5E4FEFFFF` `push dword ptr [ebp+FFFFFEE4]`

:005074A2 `8D95E0FEFFFF` `lea edx, dword ptr [ebp+FFFFFEE0]`

:005074A8 `8BC6` `mov eax, esi`

:005074AA `E84123F0FF` `call 004097F0`

:005074AF `FFB5E0FEFFFF` `push dword ptr [ebp+FFFFFEE0]`

:005074B5 `FF75E0` `push [ebp-20]`

:005074B8 `8D85E8FEFFFF` `lea eax, dword ptr [ebp+FFFFFEE8]`

:005074BE `BA03000000` `mov edx, 00000003`

:005074C3 `E820DBEFFF` `call 00404FE8` 将以上得到的余数连接起来得到数"7014"

:005074C8 `8B85E8FEFFFF` `mov eax, dword ptr [ebp+FFFFFEE8]` "7014"--->`eax`

:005074CE `8D95ECFEFFFF` `lea edx, dword ptr [ebp+FFFFFEEC]`

:005074D4 `E813030000` `call 005077EC`

:005074D9 `8B95ECFEFFFF` `mov edx, dword ptr [ebp+FFFFFEEC]`

:005074DF `8D45EC` `lea eax, dword ptr [ebp-14]`

:005074E2 `E819D8EFFF` `call 00404D00`

:005074E7 `33F6` `xor esi, esi` `esi` 清零

:005074E9 `BB01000000` `mov ebx, 00000001`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00507526(C)`
|

:005074EE `8D85DCFEFFFF` `lea eax, dword ptr [ebp+FFFFFEDC]`

:005074F4 `50` `push eax`

:005074F5 `8D95D8FEFFFF` `lea edx, dword ptr [ebp+FFFFFED8]`

:005074FB `8B45EC` `mov eax, dword ptr [ebp-14]`

:005074FE `E8E9020000` `call 005077EC`

:00507503 `8B85D8FEFFFF` `mov eax, dword ptr [ebp+FFFFFED8]` "7014"--->`eax`

:00507509 `B901000000` `mov ecx, 00000001`

:0050750E `8BD3` `mov edx, ebx`

:00507510 `E8D741F3FF` `call 0043B6EC`

:00507515 `8B85DCFEFFFF` `mov eax, dword ptr [ebp+FFFFFEDC]`

:0050751B `E83C24F0FF` `call 0040995C`

:00507520 `03F0` `add esi, eax`

:00507522 `43` `inc ebx`

:00507523 `83FB05` `cmp ebx, 00000005`

:00507526 `75C6` `jne 005074EE` 又一个循环,将 7014 各位相加，即 7+0+1+4=0xC----->`esi`

:00507528 `8BC6` `mov eax, esi`

:0050752A `B90A000000` `mov ecx, 0000000A`

:0050752F `99` `cdq`

:00507530 `F7F9` `idiv ecx` 0xC/0xA

:00507532 `8BF2` `mov esi, edx` 余数为"2"--->`esi`

:00507534 `8D95D0FEFFFF` `lea edx, dword ptr [ebp+FFFFFED0]`

:0050753A `8B45EC` `mov eax, dword ptr [ebp-14]`

:0050753D `E8AA020000` `call 005077EC`

:00507542 `8D85D0FEFFFF` `lea eax, dword ptr [ebp+FFFFFED0]`

:00507548 `50` `push eax`

:00507549 `8D95CCFEFFFF` `lea edx, dword ptr [ebp+FFFFFECC]`

:0050754F `8BC6` `mov eax, esi`

:00507551 `E89A22F0FF` `call 004097F0`

:00507556 `8B95CCFEFFFF` `mov edx, dword ptr [ebp+FFFFFECC]`

:0050755C `58` `pop eax`

:0050755D `E8CED9EFFF` `call 00404F30` 将"2"与"7014"连接起来，得到第五部分真注册码，即"70142"

:00507562 `8B85D0FEFFFF` `mov eax, dword ptr [ebp+FFFFFED0]`

:00507568 `8D95D4FEFFFF` `lea edx, dword ptr [ebp+FFFFFED4]`

:0050756E `E879020000` `call 005077EC` 真码各位取反

:00507573 `8B95D4FEFFFF` `mov edx, dword ptr [ebp+FFFFFED4]`

:00507579 `8D45EC` `lea eax, dword ptr [ebp-14]`

:0050757C `E87FD7EFFF` `call 00404D00`

:00507581 `8D85C4FEFFFF` `lea eax, dword ptr [ebp+FFFFFEC4]`

:00507587 `50` `push eax`

:00507588 `B905000000` `mov ecx, 00000005`

:0050758D `BA19000000` `mov edx, 00000019`

:00507592 `8B45F4` `mov eax, dword ptr [ebp-0C]` 取第五部分的假码

:00507595 `E85241F3FF` `call 0043B6EC`

:0050759A `8B85C4FEFFFF` `mov eax, dword ptr [ebp+FFFFFEC4]`

:005075A0 `8D95C8FEFFFF` `lea edx, dword ptr [ebp+FFFFFEC8]`

:005075A6 `E841020000` `call 005077EC` 第五部分的假码取反

:005075AB `8B95C8FEFFFF` `mov edx, dword ptr [ebp+FFFFFEC8]`

:005075B1 `8B45EC` `mov eax, dword ptr [ebp-14]`

:005075B4 `E8BBDAEFFF` `call 00405074` 第五部分的经过变换的真假注册码相比

:005075B9 `7404` `je 005075BF` 相等就跳，注册成功，否则去死(爆破点)

:005075BB `C645F300` `mov [ebp-0D], 00`

所以注册信息为：

序列号：`3781489924572`

注册名：`wzh123`

注册码：`H1287-29685-60708-81168-70142`

由于注册算法用到了序列号，所以，一个注册码只对应一台机器，你只好自己算算了^-^

`4dd466d0e1e0cc123afbe504da2462ca`
