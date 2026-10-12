
# 学友计算机等级考试试题汇编 2002A+

> 原文：[`www.pediy.com/kssd/pediy05/pediy50437.htm`](https://www.pediy.com/kssd/pediy05/pediy50437.htm)

简单算法——学友计算机等级考试试题汇编 2002A+

下载页面： [`www.skycn.com/soft/8393.html`](http://www.skycn.com/soft/8393.html)  
软件大小： 769 KB  
软件语言： 简体中文  
软件类别： 国产软件 / 共享版 / 考试系统  
应用平台： `Win9x/NT/2000/XP`  
加入时间： 2002-11-05 15:25:55  
下载次数： 11521  
推荐等级： ***

【软件简介】：这是一个全国计算机等级考试辅助学习软件，本套软件对用户的计算机配置要求极低，能正常运行 `Win98` 的电脑均可流畅的运行它，且支持最新的 `Windows XP` 操作系统。本套软件适用于个人自学、学校或电脑培训部教学时使用，当前的 `2002A+` 版本已经包含了 一级 `Windows` 考试从 1999 年至 2002 年上半年 的考试试题，二级考试中的 `C` 语言 和 `FOXBASE` 这两门程序设计语言从 1994 年至 2002 年上半年 的考试试题，题库中的试题数量达到 1870 多题，均为百分之百的考试试卷原题。本软件的主要功能：用户可根据自己的需要选择每种程序设计语言在某年的 4 月份或 9 月份的考试试卷，并可在答卷过程中随时打开[试卷评分面板]，系统将自动对当前试卷进行评分，并显示出每道试题的对错情况和用户所选择的答案以及标准答案，此时用户仍可继续答题，对做错的试题还可加以改正，以利于加深用户对做错的试题的印象，此[试卷评分面板]可被随时关闭。用户可随意修改试题显示区域的背景色或前景色。用户在查看试卷评分后如想退出答卷环境则可使用交卷功能。交卷后即可再选做其它试卷。

【软件限制】：功能限制

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000` 娃娃修改版、`FI2.5`、`AspackDie`、`W32Dasm8.93` 黄金版

—————————————————————————————

【过  程】：

`Windjks.exe` 为 `ASPACK 2.12` 壳。用 `AspackDie` 脱之。反汇编。

`TRW2000` 装入目标程序。  
填入试炼码（即：假码，试炼码是我的习惯称呼）：`13572468`  
注意：至少要 8 位！！！

`CTR+N` 切入 `TRW`，下万能断点：`BPX HMEMCPY`，`F5` 返回，点注册，立即拦下。

`BD`，暂停断点。  
下 `PMODULE` 指令，返回程序领空，按 `F12` 七次，`DELPHI` 编的东西许多都是按 `F12` 七次到达核心。

—————————————————————————————

`F10` 走，多看看，多留意。  

请看：

```
:004BB1BA E8A999F4FF                 call 00404B68  
:004BB1BF 83F808                     cmp eax, 00000008  
                                     ====>比较试炼码位数是否不小于 8 位？

:004BB1C2 7D2C                       jge 004BB1F0   

                                     ====>不跳则 OVER！

:004BB1C4 8D45FC                     lea eax, dword ptr [ebp-04]  
:004BB1C7 BA0A000000                 mov edx, 0000000A  
:004BB1CC E81B9DF4FF                 call 00404EEC  
:004BB1D1 BB01000000                 mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004BB1E9(C)  
|  
:004BB1D6 8D45FC                     lea eax, dword ptr [ebp-04]  
:004BB1D9 E8DA9BF4FF                 call 00404DB8  
:004BB1DE 6BD30D                     imul edx, ebx, 0000000D  
:004BB1E1 885418FF                   mov byte ptr [eax+ebx-01], dl  
:004BB1E5 43                         inc ebx  
:004BB1E6 83FB0B                     cmp ebx, 0000000B  
:004BB1E9 75EB                       jne 004BB1D6  
:004BB1EB E9A7000000                 jmp 004BB297
```

跳到这：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004BB1C2(C)  
|  
:004BB1F0 8D55E4                     lea edx, dword ptr [ebp-1C]  
:004BB1F3 8B862C030000               mov eax, dword ptr [esi+0000032C]  
:004BB1F9 E84A55F8FF                 call 00440748  
:004BB1FE 8B45E4                     mov eax, dword ptr [ebp-1C]  
:004BB201 8D55E8                     lea edx, dword ptr [ebp-18]  
:004BB204 E81BDDF4FF                 call 00408F24  
:004BB209 8B45E8                     mov eax, dword ptr [ebp-18]  
:004BB20C 8D55EC                     lea edx, dword ptr [ebp-14]  
:004BB20F E8DCDCF4FF                 call 00408EF0  
:004BB214 8B45EC                     mov eax, dword ptr [ebp-14]  
:004BB217 50                         push eax  
:004BB218 8D55E0                     lea edx, dword ptr [ebp-20]  
:004BB21B 8BC6                       mov eax, esi  
:004BB21D E836FBFFFF                 call 004BAD58  
                                     ===>算法 CALL！

:004BB222 8B55E0                     mov edx, dword ptr [ebp-20]  
:004BB225 58                         pop eax

:004BB226 E8819AF4FF                 call 00404CAC  
                                     ===>比较注册码！

:004BB22B 7543                       jne 004BB270  
:004BB22D 8BC6                       mov eax, esi  
:004BB22F E848FEFFFF                 call 004BB07C  
:004BB234 8D45FC                     lea eax, dword ptr [ebp-04]  
:004BB237 BA0A000000                 mov edx, 0000000A  
:004BB23C E8AB9CF4FF                 call 00404EEC  
:004BB241 BB01000000                 mov ebx, 00000001
```

—————————————————————————————

`F8` 进入算法 `CALL`：`:004BB21D call 004BAD58`  
即比较注册码的上一个 `CALL`

```
* Referenced by a CALL at Address:

|:004BB21D   

|  
:004BAD58 55                         push ebp  
:004BAD59 8BEC                       mov ebp, esp  
:004BAD5B B906000000                 mov ecx, 00000006

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004BAD65(C)  
|  
:004BAD60 6A00                       push 00000000  
:004BAD62 6A00                       push 00000000  
:004BAD64 49                         dec ecx  
:004BAD65 75F9                       jne 004BAD60  
:004BAD67 51                         push ecx  
:004BAD68 53                         push ebx  
:004BAD69 56                         push esi  
:004BAD6A 8BF2                       mov esi, edx  
:004BAD6C 8BD8                       mov ebx, eax  
:004BAD6E 33C0                       xor eax, eax  
:004BAD70 55                         push ebp  
:004BAD71 68D3AE4B00                 push 004BAED3  
:004BAD76 64FF30                     push dword ptr fs:[eax]  
:004BAD79 648920                     mov dword ptr fs:[eax], esp  
:004BAD7C 8D55F8                     lea edx, dword ptr [ebp-08]  
:004BAD7F 8BC3                       mov eax, ebx  
:004BAD81 E80AFDFFFF                 call 004BAA90  
:004BAD86 8D45FC                     lea eax, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"8888"  
                                     ===>注意此数！  
                                     |  
:004BAD89 BAE8AE4B00                 mov edx, 004BAEE8  
                                     ===>8888 移入 EDX

:004BAD8E E8B59BF4FF                 call 00404948  
:004BAD93 8D45FC                     lea eax, dword ptr [ebp-04]

:004BAD96 E81DA0F4FF                 call 00404DB8  
                                     ===>此 CALL 取得你的 BIOS 的日期，  
我的机子是“老古董”了，BIOS 日期是 020598，即 98 年 5 月 2 日的

:004BAD9B 8B55F8                     mov edx, dword ptr [ebp-08]  
                                     ===>EDX=020598

:004BAD9E 8A5201                     mov dl, byte ptr [edx+01]  
                                     ===>EDX 的第二位移入 DL ？DL=32（即 2）

:004BADA1 8810                       mov byte ptr [eax], dl  
                                     ===>把 DL 移入[EAX]，即 2 移入[EAX]

:004BADA3 8D45FC                     lea eax, dword ptr [ebp-04]  
                                     ===>D EAX=2888，装入[EBP-04]  
                                     即 8888 的第一位“让位”给 2 了

:004BADA6 E80DA0F4FF                 call 00404DB8  
:004BADAB 8B55F8                     mov edx, dword ptr [ebp-08]  
                                     ===>[EBP-08]里放的 BIOS 日期，  
                                     即把 020598 又移入 EDX

:004BADAE 8A5203                     mov dl, byte ptr [edx+03]  
                                     ===>把 EDX 的第四位移入 DL，DL=5

:004BADB1 885001                     mov byte ptr [eax+01], dl  
                                     ===>把 5 移入[EAX+01]

:004BADB4 8D45F4                     lea eax, dword ptr [ebp-0C]  
                                     ===>D EAX=2588，装入[EBP-0C]  
                                     即 2888 的第二位又“让”给 5 了

:004BADB7 8B55FC                     mov edx, dword ptr [ebp-04]  
                                     ===>此时[EBP-4]=2588

:004BADBA 8A5201                     mov dl, byte ptr [edx+01]  
                                     ===>把 2588 的第二位“5”移入 DL，DL=35

:004BADBD E8CE9CF4FF                 call 00404A90  
:004BADC2 8B45F4                     mov eax, dword ptr [ebp-0C]  
:004BADC5 E8BEE4F4FF                 call 00409288  
:004BADCA 8BD8                       mov ebx, eax  
                                     ===>EAX=5，即上面 DL=35 的 HEX 值对应的字符  
                                     把 5 移入 EBX

:004BADCC 83C348                     add ebx, 00000048  
                                     ===>EBX=5+48=4D，16 进制相加。  
:004BADCF 8D45FC                     lea eax, dword ptr [ebp-04]  
:004BADD2 E8E19FF4FF                 call 00404DB8  
                                     ===>4D 对应的字符为 M 
```



### 汇编代码分析与注释

`:004BADD7` `885802` `mov byte ptr [eax+02], bl`
`====>` 把 `BL` 中的 `M` 移入 `[EAX+2]`

`:004BADDA` `8D45F0` `lea eax, dword ptr [ebp-10]`
`====>` `EAX`=25M8
`====>` 即 `2588` 的第三位 `8` 变为 `M` 了

`:004BADDD` `8B55FC` `mov edx, dword ptr [ebp-04]`
`====>` `EDX`=25M8

`:004BADE0` `8A12` `mov dl, byte ptr [edx]`
`====>` `DL`=32，即 `2` 的 `HEX` 值

`:004BADE2` `E8A99CF4FF` `call 00404A90`

`:004BADE7` `8B45F0` `mov eax, dword ptr [ebp-10]`

`:004BADEA` `E899E4F4FF` `call 00409288`

`:004BADEF` `8BD8` `mov ebx, eax`
`====>` `EAX`=2，移入 `EBX`

`:004BADF1` `83C343` `add ebx, 00000043`
`====>` `EBX`=2+43=45，16 进制相加

`:004BADF4` `8D45FC` `lea eax, dword ptr [ebp-04]`

`:004BADF7` `E8BC9FF4FF` `call 00404DB8`
`====>` 此 `CALL` 把 `45` 转化为对应的字符：`E`

`:004BADFC` `885803` `mov byte ptr [eax+03], bl`
`====>` 把 `BL` 中的 `E` 移入 `[EAX+03]`

`:004BADFF` `BBC5440D00` `mov ebx, 000D44C5`
`====>` `D44C5` 移入 `EBX`。注意程序给的此数！

`:004BAE04` `8D45E8` `lea eax, dword ptr [ebp-18]`

`:004BAE07` `8B55FC` `mov edx, dword ptr [ebp-04]`
`====>` `25ME` 移入 `EDX`

`:004BAE0A` `8A5202` `mov dl, byte ptr [edx+02]`
`====>` 把 `EDX` 的第三个字符移入 `DL`，即 `M`

`:004BAE0D` `885001` `mov byte ptr [eax+01], dl`
`====>` `M` 移入 `[EAX+01]`

`:004BAE10` `C60001` `mov byte ptr [eax], 01`

`:004BAE13` `8D55E8` `lea edx, dword ptr [ebp-18]`

`:004BAE16` `8D45E4` `lea eax, dword ptr [ebp-1C]`

`:004BAE19` `E8D681F4FF` `call 00402FF4`

`:004BAE1E` `8D45E0` `lea eax, dword ptr [ebp-20]`

`:004BAE21` `8B55FC` `mov edx, dword ptr [ebp-04]`
`====>` `25ME` 移入 `EDX`

`:004BAE24` `8A5203` `mov dl, byte ptr [edx+03]`
`====>` `25ME` 的第四位移入 `DL`，即 `E`

`:004BAE27` `885001` `mov byte ptr [eax+01], dl`
`====>` `E` 移入 `[EAX+01]`

`:004BAE2A` `C60001` `mov byte ptr [eax], 01`

`:004BAE2D` `8D55E0` `lea edx, dword ptr [ebp-20]`

`:004BAE30` `8D45E4` `lea eax, dword ptr [ebp-1C]`

`:004BAE33` `B102` `mov cl, 02`

`:004BAE35` `E88A81F4FF` `call 00402FC4`

`:004BAE3A` `8D55E4` `lea edx, dword ptr [ebp-1C]`

`:004BAE3D` `8D45EC` `lea eax, dword ptr [ebp-14]`

`:004BAE40` `E8C79CF4FF` `call 00404B0C`
`====>` 此 `CALL` 取得上面两次移入的字符 `ME`

`:004BAE45` `FF75EC` `push [ebp-14]`
`====>` `DD*`（`EBP-14`）=`ME`
`********** 真码的第一部分！！！`
`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`

`:004BAE48` `8B45F8` `mov eax, dword ptr [ebp-08]`
`====>` `D` `EAX`=020598 我的 `BIOS` 日期

`:004BAE4B` `E838E4F4FF` `call 00409288`
`====>` `？EAX=5076`（十进制=20598）

`:004BAE50` `33C3` `xor eax, ebx` 异或运算
`====>` `EBX`=D44C5，即 `004BADFF` 处移入的值
`====>` `5076 XOR D44C5=D14B3`
`********** D14B3` 的十进制是 `857267`，真码的第二部分！！！
`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`

`:004BAE52` `8D55D4` `lea edx, dword ptr [ebp-2C]`

`:004BAE55` `E8CAE3F4FF` `call 00409224`

`:004BAE5A` `8B45D4` `mov eax, dword ptr [ebp-2C]`

`:004BAE5D` `8D55D8` `lea edx, dword ptr [ebp-28]`

`:004BAE60` `E8BFE0F4FF` `call 00408F24`

`:004BAE65` `8B45D8` `mov eax, dword ptr [ebp-28]`

`:004BAE68` `8D55DC` `lea edx, dword ptr [ebp-24]`

`:004BAE6B` `E880E0F4FF` `call 00408EF0`

`:004BAE70` `FF75DC` `push [ebp-24]`

`:004BAE73` `8D45D0` `lea eax, dword ptr [ebp-30]`

`:004BAE76` `8B55FC` `mov edx, dword ptr [ebp-04]`
`====>` `25ME` 移入 `EDX`

`:004BAE79` `8A5201` `mov dl, byte ptr [edx+01]`
`====>` 第二位 `5` 移入 `DL`

`:004BAE7C` `E80F9CF4FF` `call 00404A90`

`:004BAE81` `FF75D0` `push [ebp-30]`
`====>` `DD*`（`EBP-30`）=`5`
`********** 真码的第三部分！！！`
`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`

`:004BAE84` `8D45CC` `lea eax, dword ptr [ebp-34]`

`:004BAE87` `8B55FC` `mov edx, dword ptr [ebp-04]`

`:004BAE8A` `8A12` `mov dl, byte ptr [edx]`
`====>` `DL`=32，即 `2`

`:004BAE8C` `E8FF9BF4FF` `call 00404A90`

`:004BAE91` `FF75CC` `push [ebp-34]`
`====>` `DD*`（`EBP-34`）=`2`
`********** 真码的第四部分！！！`
`@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@`

`:004BAE94` `8D45F8` `lea eax, dword ptr [ebp-08]`

`:004BAE97` `BA04000000` `mov edx, 00000004`

`:004BAE9C` `E8879DF4FF` `call 00404C28`
`====>` 此 `CALL` 把 `1`、`2`、`3`、`4` 部分连接起来！
即把 `ME`、`857267`、`5`、`2`-->`ME85726752`
放在 `[EBP-08]`

`:004BAEA1` `8BC6` `mov eax, esi`

`:004BAEA3` `8B55F8` `mov edx, dword ptr [ebp-08]`
`====>` 真码移入 `EDX`

`:004BAEA6` `E8599AF4FF` `call 00404904`

---

### 算法总结

1. 程序先给出 `4` 个字符：`8888`
   - 取得你的机子的 `BIOS` 日期。如我的是 `020598`，从中取第二个、第四个数，移入`"8888"`的第一、第二位。`8888`-->`2588`。
   - 把新得出的 `2588` 的第二位`"5"`作为 `16` 进制数和程序给的 `48` 相加。`5+48=4D`,`4D` 所对应的字符为`"M"`。把 `M` 移入 `2588` 的第三位，`2588`-->`25M8`
   - 同样，取第一位`"2"`作为 `16` 进制数和程序给的 `43` 相加。`2+43=45`，`45` 所对应的字符为`"E"`。把 `E` 移入 `2588` 的第三位，`25M8`-->`25ME`。
   - 至此：`8888`-->`25ME`。

2. 取新得出的 `25ME` 的第三、四位作为真码的第一、二位。即：`ME`

3. 取 `BIOS` 日期作为 `16` 进制数与程序给的 `D44C5` 进行异或运算，`5076 XOR D44C5=D14B3`。
   - `D14B3` 的十进制是 `857267`，真码的第二部分。

4. 取 `25ME` 的第二位`"5"`，作为真码的第三部分。

5. 取 `25ME` 的第一位`"2"`，作为真码的第四部分。

最后得出完整的真码：`ME85726752`

---

记得 有声有色 `7.0` 也是把 `BIOS` 的日期作为一个主要参数。

`BIOS` 日期：

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Enum\Root\*PNP0C01\0000]

"BIOSName"="Award"
"BIOSVersion"="Award Modular BIOS v4.51PG"
"BIOSDate"="02/05/98"
```

---

`F8` 进入比较 `CALL`：`004BB226` `call 00404CAC`

`:00404CAC` `53` `push ebx`

`:00404CAD` `56` `push esi`

`:00404CAE` `57` `push edi`

`:00404CAF` `89C6` `mov esi, eax`

`:00404CB1` `89D7` `mov edi, edx`

`:00404CB3` `39D0` `cmp eax, edx`
`====>` 比较了！
`====>` `D` `EAX`=13572468（假码）
`====>` `D` `EDX`=真码！！！

`:00404CB5` `0F848F000000` `je 00404D4A`

`:00404CBB` `85F6` `test esi, esi`

`:00404CBD` `7468` `je 00404D27`

`:00404CBF` `85FF` `test edi, edi`

`:00404CC1` `746B` `je 00404D2E`

`:00404CC3` `8B46FC` `mov eax, dword ptr [esi-04]`

`:00404CC6` `8B57FC` `mov edx, dword ptr [edi-04]`

`:00404CC9` `29D0` `sub eax, edx`

`:00404CCB` `7702` `ja 00404CCF`

---

`【KeyMake 之内存注册机】`：

中断地址：`406751`

中断次数：`1`

第一字节：`51`

指令长度：`1`



# 中断地址

`406751`

# 中断次数

`1`

# 第一字节

`51`

# 指令长度

`1`

# 内存方式

`ECX`

---

## 注册信息保存

在 `C:\WINDOWS\INF` 目录下有一个文件：

`{F982X26B-19PA-81C5-32EG-O096345608NK}`

该文件没有后缀名，删除它即可变回未注册版。

---

## 整理

# 机器码

`28771035`

# 真码

`ME85726752`

---

Cracked By 巢水工作坊——fly【OCN】

`2003-2-4`

`4dd466d0e1e0cc123afbe504da2462ca`
