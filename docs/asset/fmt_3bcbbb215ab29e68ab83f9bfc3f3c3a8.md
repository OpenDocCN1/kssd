
# Visual CHM 4.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50483.htm`](https://www.pediy.com/kssd/pediy05/pediy50483.htm)

算法浅探！——Visual CHM 4.0

下载页面： [`www.skycn.com/soft/6376.html`](http://www.skycn.com/soft/6376.html)

软件大小： 1570 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 安装制作

应用平台： `Win9x/NT/2000/XP`

加入时间： 2002-11-18 09:39:16

下载次数： 12464

推荐等级： `*****`

开发商： [`cn.geocities.com/vchm2000/`](http://cn.geocities.com/vchm2000/)

**【软件简介】：** 一级棒的制作 CHM 文件的工具。

Visual CHM 将帮助您非常容易的制作出具有非常专业水准的 CHM 文件，而且是“所见即所得”。

**【软件限制】：** 功能限制

**【作者声明】：** 初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】：** `TRW2000` 娃娃修改版、`Ollydbg1.09`、`FI2.5`、`AspackDie`、`W32Dasm8.93` 黄金版

—————————————————————————————————

**【过 程】：**

呵呵，记得刚学破解时想把《看雪论坛精华 3、4》合并为 1 个 CHM 文件，于是找到了这个软件，Visual CHM 的确是一款一级棒的制作软件，但是未注册版只能编译 15 个节点的文件。当时我调试了好几次均无功而返。

后来得知软件作者葛泽华先生就是一位 CRACK 高手！前些日从看雪精华里发现 `heXer/iPB` 老师分析 3.10 版的算法解文，非常高兴，于是这 3 天来静下心又重新试试 4.0 的破解，居然饶幸找到了一点眉目！

我很菜，许多地方分析的有错误，恳请诸位老师指正！另外：特别感谢 `heXer/iPB` 老师和葛泽华先生。冒犯之处，还请葛先生海涵！

近 1 个月我把学破解以来所作的笔记整理出来一部分，已经全部放到论坛上了，谢谢老师、朋友们的关心和帮助！以后我或许没有充裕的时间破解了，就以此篇解文作为我这段学习日子的小结吧！东方欲晓了，呵呵，又一个不眠之夜。~\_~~\_~

—————————————————————————————————

`Vchm.exe` 是 `ASPack 2.12` 壳，用 `AspackDie` 脱之。407K->1.28M。反汇编，便于静态分析。

用户名：`fly\[OCN\]` （用户名长度须在 5-32 位间）

试炼码：`BCDEFGHIJK` （注册码 10 位）

设：
1. 用户名 `fly\[OCN\]` 为 `N0`；
2. 对 `N0` 进行运算后得出的字符 `TJYIPJFB` 为 `N1`；
3. 对 `N1` 运算后得出的字符 `TJYIPJFBFW` 为 `N2`；
4. 试炼码 `BCDEFGHIJK` 为 `K0`；
5. 对 `K0` 进行运算后得出的字符 `LJPJLJXJLJ` 为 `K1`；
6. 对 `K1` 进行运算后得出的字符 `RSTUVWXYZJ` 为 `K2`。

呵呵，我也是晕头转向呀！~@~

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

## 一、对用户名 N0 进行运算后得出 N1

`Possible StringData Ref from Code Obj -> "[www.vchm.com/](http://www.vchm.com/) convenient "`

```
->"CHM editor,WYSIWYG."
|
:004E7684 BA3C8B4E00           mov edx, 004E8B3C
                              ====>EDX=http://www.vchm.com/ convenient CHM editor,WYSIWYG.
:004E7689 E8BAD5F1FF           call 00404C48
:004E768E 8B45FC               mov eax, dword ptr [ebp-04]
:004E7691 0550060000           add eax, 00000650
:004E7696 8B55FC               mov edx, dword ptr [ebp-04]
:004E7699 8B9248060000         mov edx, dword ptr [edx+00000648]
:004E769F E860D5F1FF           call 00404C04

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E764A(C)
|
:004E76A4 8D45E8               lea eax, dword ptr [ebp-18]
:004E76A7 BA788B4E00           mov edx, 004E8B78
:004E76AC E897D5F1FF           call 00404C48
:004E76B1 8B45FC               mov eax, dword ptr [ebp-04]
:004E76B4 8B8050060000         mov eax, dword ptr [eax+00000650]
                              ====>EAX=fly[OCN]
:004E76BA E8C5D7F1FF           call 00404E84
                              ====>取用户名长度
:004E76BF 8BF8                 mov edi, eax
                              ====>EDI=8
:004E76C1 85FF                 test edi, edi
:004E76C3 7E66                 jle 004E772B
:004E76C5 BE01000000           mov esi, 00000001
                              ====>ESI=1

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E7729(C)
|
:004E76CA 8B45FC               mov eax, dword ptr [ebp-04]
:004E76CD 8B8050060000         mov eax, dword ptr [eax+00000650]
                              ====>EAX=fly[OCN]
:004E76D3 8A5C30FF             mov bl, byte ptr [eax+esi-01]
                              ====>依次取用户名字符的 HEX 值
                              1、 ====>BL=66
                              …… …… 省 略 …… ……
                              8、 ====>BL=5D
:004E76D7 8B45EC               mov eax, dword ptr [ebp-14]
                              ====>EAX=http://www.vchm.com/ convenient CHM editor,WYSIWYG.
:004E76DA 8A4430FF             mov al, byte ptr [eax+esi-01]
                              ====>依次取 http://www.vchm.com/ convenient CHM editor,WYSIWYG.
                              1、 ====>AL=68
                              …… …… 省 略 …… ……
                              8、 ====>AL=77
:004E76DE 32D8                 xor bl, al
                              1、 ====>BL=66 XOR 68=0E
                              …… …… 省 略 …… ……
                              8、 ====>BL=5D XOR 77=2A
:004E76E0 81E3FF000000         and ebx, 000000FF
:004E76E6 33DE                 xor ebx, esi
                              1、 ====>EBX=0E XOR 01=0F
                              …… …… 省 略 …… ……
                              8、 ====>EBX=2A XOR 08=22
:004E76E8 83FB41               cmp ebx, 00000041
                              ==== >EBX 是否 小于 41？
:004E76EB 7D0B                 jge 004E76F8
                              ==== >小于则下面相加

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E76F6(C)
|
:004E76ED 8D441E16             lea eax, dword ptr [esi+ebx+16]
                              1、①  ==== >EAX=01 + 0F + 16=26
                              1、②  ==== >EAX=01 + 26 + 16=3D
                              1、③  ==== >EAX=01 + 3D + 16=54
                              …… …… 省 略 …… ……
                              8、①  ==== >EAX=08 + 22 + 16=40
                              8、②  ==== >EAX=08 + 40 + 16=5E
:004E76F1 8BD8                 mov ebx, eax
                              ==== >EBX=EAX
:004E76F3 83FB41               cmp ebx, 00000041
                              ==== >EBX 是否 小于 41？
:004E76F6 7CF5                 jl 004E76ED
                              ==== >是则跳上去继续相加，直至不小于 41

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E76EB(C)
|
:004E76F8 83FB7A               cmp ebx, 0000007A
:004E76FB 7E0F                 jle 004E770C
                              ==== >若大于 7A ，则下面相减！

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E7705(C)
|
:004E76FD 83EB1B               sub ebx, 0000001B
:004E7700 2BDE                 sub ebx, esi
:004E7702 83FB7A               cmp ebx, 0000007A
:004E7705 7FF6                 jg 004E76FD
:004E7707 EB03                 jmp 004E770C

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E7714(C)
|
:004E7709 83C304               add ebx, 00000004
                              …… …… 省 略 …… ……
                              8、①  ==== >EAX=5E + 04=62

Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E76FB(C), :004E7707(U)
|
:004E770C 83FB61               cmp ebx, 00000061
                              ==== >EBX 是否 小于 61？
:004E770F 7D05                 jge 004E7716
:004E7711 83FB5A               cmp ebx, 0000005A
:004E7714 7FF3                 jg 004E7709

Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E770F(C)
|
:004E7716 8B45FC               mov eax, dword ptr [ebp-04]
:004E7719 0550060000           add eax, 00000650
:004E771E E8B1D9F1FF           call 004050D4
:004E7723 885C30FF             mov byte ptr [eax+esi-01], bl
                              ==== >结果入 [eax+esi-01] 处
                              1、 ====>BL=54
                              2、 ====>BL=4A
                              3、 ====>BL=59
                              4、 ====>BL=49
                              5、 ====>BL=70
                              6、 ====>BL=6A
                              7、 ====>BL=66
                              8、 ====>BL=62
循环结束后[eax+esi-01] 处是 fly\OCN\经过以上运算转换后的字符：TJYIpjfb（设为 N1）
:004E7727 46                   inc esi
                              ==== >ESI 逐次增 1
:004E7728 4F                   dec edi
                              ==== >8 次！用户名长度
:004E7729 759F                 jne 004E76CA
```
