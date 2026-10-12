
# 英宇职业介绍管理系统 V5.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6086.htm`](https://www.pediy.com/kssd/pediy06/pediy6086.htm)

简单算法——英宇职业介绍管理系统 `V5.0`（`Softsentry 2.11` 保护）

下载页面： [`www.downloadsky.com/soft/9190.html`](http://www.downloadsky.com/soft/9190.html)

软件大小： 2624 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 其它行业

应用平台： `Win9x/NT/2000/XP`

加入时间： 2003-05-09 10:53:12

下载次数： 859

推荐等级： \*\*\*

【软件简介】：英宇职业介绍管理系统是针对职业介绍（人才服务）行业开发制作的管理系统。当前程序版本 `V3.10`（单机版本），由招聘信息管理、人才库管理、职介管理、介绍信打印、系统维护等功能组成。软件操作界面友好、功能多、操作简单，查询功能强大，并具有方便快捷的数据导出功能，各种数据表格可以所见即所得地转换为 `EXCEL` 格式，还可以导出介绍信到您自定义的 `Word` 模版中，打印您所需要的样式。

【软件限制】：30 天试用

【作者声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`W32Dasm 9.0` 白金版

—————————————————————————————————

## 【过 程】

`英宇职介管理 V5.exe` 用 `FI2.5` 看是 `Softsentry 2.11` 壳，晕，现在居然还用 `V2.11` 加壳。 有专用的 For `Softsentry2.11` 的脱壳工具：`Crkss211.com`，脱完壳后就取消一切限制了。这篇我写的稍微简单点，其实 `Softsentry` 壳的算法都大同小异，具体的可以看我以前分析过的笔记。这个程序不同的是取了用户名和单位名进行运算。

序列号：95065

用户名：`fly`

单位名：\[OCN\]\[FCG\]

试炼码：`ABCDEFGH-12345678-KLMNOPQ`

—————————————————————————————————

可以下 `bpx getdlgitemtexta` 一般 `Softsentry` 壳下这个断点挺好用。

拦下后返回程序细心跟踪会来到下面：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721ABD(C)

|

:00721B55 `8B3D44BC7200`  mov `edi`, `dword ptr [0072BC44]`  ==> `EDI=YYG-YYZJ-`  这就是 String 1

:00721B5B `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721B60 `2BC0`  sub `eax`, `eax`

:00721B62 `F2`  repnz

:00721B63 `AE`  scasb

:00721B64 `F7D1`  not `ecx`

:00721B66 `49`  dec `ecx`  ==> 取长度 `ECX=9`

:00721B67 `6649`  dec `cx`

:00721B69 `6683F9FF`  cmp `cx`, `FFFF`

:00721B6D `7426`  je `00721B95`

:00721B6F `6685C9`  test `cx`, `cx`

:00721B72 `7C1B`  jl `00721B8F`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721B8D(C)

|

:00721B74 `8B1544BC7200`  mov `edx`, `dword ptr [0072BC44]`  ==> `EDX=YYG-YYZJ-`

:00721B7A `0FBFC1`  movsx `eax`, `cx`

:00721B7D `8A1402`  mov `dl`, `byte ptr [edx+eax]`  ==> `DI=`依次倒序取 `YYG-YYZJ-`

:00721B80 `80FA3F`  cmp `dl`, `3F`

:00721B83 `7406`  je `00721B8B`

:00721B85 `3854041C`  cmp `byte ptr [esp+eax+1C]`, `dl`  ==> 逐位比较试炼码前 9 位是否是 `YYG-YYZJ-`

:00721B89 `7504`  jne `00721B8F`  ==> 跳则 OVER！可以 NOP 掉，方便调试 ^O^ ^O^ 一、  ==> 所以注册码前 9 位固定是 `YYG-YYZJ-`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721B83(C)

|

:00721B8B `6649`  dec `cx`

:00721B8D `79E5`  jns `00721B74`  ==> 循环比较！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00721B72(C), :00721B89(C)

|

:00721B8F `6683F9FF`  cmp `cx`, `FFFF`

:00721B93 `7505`  jne `00721B9A`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721B6D(C)

|

:00721B95 `BD01000000`  mov `ebp`, `00000001`  ==> `EBP=1`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721B93(C)

|

:00721B9A `8B3DCCBB7200`  mov `edi`, `dword ptr [0072BBCC]`  ==> `EDI=-1002002`  这就是 String 2

:00721BA0 `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721BA5 `2BC0`  sub `eax`, `eax`

:00721BA7 `F2`  repnz

:00721BA8 `AE`  scasb

:00721BA9 `F7D1`  not `ecx`

:00721BAB `49`  dec `ecx`  ==> 取长度 `ECX=8`

:00721BAC `8D7C241C`  lea `edi`, `dword ptr [esp+1C]`  ==> `EDI=ABCDEFGH-12345678-KLMNOPQ` 试炼码

:00721BB0 `668BD1`  mov `dx`, `cx`  ==> `DX=CX=8`

:00721BB3 `2BC0`  sub `eax`, `eax`

:00721BB5 `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721BBA `F2`  repnz

:00721BBB `AE`  scasb

:00721BBC `F7D1`  not `ecx`

:00721BBE `49`  dec `ecx`  ==> 取长度 `ECX=19`

:00721BBF `662BCA`  sub `cx`, `dx`  ==> `CX=19 - 8=11`

:00721BC2 `6685C9`  test `cx`, `cx`

:00721BC5 `7E2F`  jle `00721BF6`

:00721BC7 `6633F6`  xor `si`, `si`

:00721BCA `6685D2`  test `dx`, `dx`

:00721BCD `7E21`  jle `00721BF0`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721BEE(C)

|

:00721BCF `A1CCBB7200`  mov `eax`, `dword ptr [0072BBCC]`

:00721BD4 `0FBFFE`  movsx `edi`, `si`

:00721BD7 `8A0438`  mov `al`, `byte ptr [eax+edi]`  ==> `AI=`依次倒序取 `-1002002`

:00721BDA `3C3F`  cmp `al`, `3F`

:00721BDC `740B`  je `00721BE9`

:00721BDE `0FBFD9`  movsx `ebx`, `cx`

:00721BE1 `03DF`  add `ebx`, `edi`

:00721BE3 `38441C1C`  cmp `byte ptr [esp+ebx+1C]`, `al`  ==> 逐位比较试炼码最后 8 位是否是 `-1002002`

:00721BE7 `7507`  jne `00721BF0`  ==> 跳则 OVER！可以 NOP 掉，方便调试 ^O^ ^O^ 二、  ==> 所以注册码最后 8 位固定是 `-1002002`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721BDC(C)

|

:00721BE9 `6646`  inc `si`

:00721BEB `663BD6`  cmp `dx`, `si`

:00721BEE `7FDF`  jg `00721BCF`  ==> 循环比较！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00721BCD(C), :00721BE7(C)

|

:00721BF0 `663BD6`  cmp `dx`, `si`

:00721BF3 `7501`  jne `00721BF6`

:00721BF5 `45`  inc `ebp`  ==> `EBP=1 + 1=2`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00721BC5(C), :00721BF3(C)

|

:00721BF6 `83FD02`  cmp `ebp`, `00000002`  ==> 是否已比较 2 次？

:00721BF9 `740A`  je `00721C05`  ==> 跳下去

:00721BFB `BDFEFFFFFF`  mov `ebp`, `FFFFFFFE`

:00721C00 `E900010000`  jmp `00721D05`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00721BF9(C)

|

:00721C05 `8B3D44BC7200`  mov `edi`, `dword ptr [0072BC44]`

:00721C0B `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721C10 `2BC0`  sub `eax`, `eax`

:00721C12 `F2`  repnz

:00721C13 `AE`  scasb

:00721C14 `F7D1`  not `ecx`

:00721C16 `2BC0`  sub `eax`, `eax`

:00721C18 `8D740C1B`  lea `esi`, `dword ptr [esp+ecx+1B]`

:00721C1C `8BFE`  mov `edi`, `esi`

:00721C1E `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721C23 `F2`  repnz

:00721C24 `AE`  scasb

:00721C25 `F7D1`  not `ecx`

:00721C27 `8B3DCCBB7200`  mov `edi`, `dword ptr [0072BBCC]`

:00721C2D `2BC0`  sub `eax`, `eax`

:00721C2F `8D51FF`  lea `edx`, `dword ptr [ecx\-01]`

:00721C32 `B9FFFFFFFF`  mov `ecx`, `FFFFFFFF`

:00721C37 `F2`  repnz

:00721C38 `AE`  scasb

:00721C39 `F7D1`  not `ecx`

:00721C3B `49`  dec `ecx`

:00721C3C `8BC6`  mov `eax`, `esi`
