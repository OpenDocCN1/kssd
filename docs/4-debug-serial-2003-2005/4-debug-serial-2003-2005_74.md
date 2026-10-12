
# Becky! Internet Mail Ver.2.05.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50479.htm`](https://www.pediy.com/kssd/pediy05/pediy50479.htm)

简单算法——Becky! Internet Mail Ver.2.05.2

【软件简介】：Becky 是一款由日本人编制的邮件软件， 此是著名汉化人小鱼儿制作的完美汉化版本。它具备比 `OE` 更为强大的功能，可以完美支持多内码，可以完美支持微软 `Hotmail` 邮箱（包括发送，这点可是别的 E-mail 工具所不具备的），完美无缺的远程邮箱管理（可选择性地下载其中某个附件）功能等诸多功能。

【软件限制】：30 天试用。其实机子里有 `OE`、`FoxMail`，不会用它的。^-^

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

—————————————————————————————————

【过 程】：很久以前写的，呵呵，索性也贴上来吧。

`B2.exe` 无壳，可能让汉化者脱了？Visual C++ 6.0 编写。

填好试炼信息：

名称：`fly`

注册的通行码：`9912-4444-WXYZ`

E-Mail：`fly4099@sohu.com`

反汇编后的提示很多乱码，所以我立即祭出屠龙刀 `TRW2000`！

填好注册信息后，`CTR+N`，下 `BPX HMEMCPY`，`F5` 返回，点“确定”，被拦。

`PMODULE` 直达程序领空。`BD`,暂停断点。`F12` 三次，`F10` 至 `525CB4`。

—————————————————————————————————

```
:00525CB4 A1F02D5B00                 mov eax, dword ptr [005B2DF0]
                                     ==== >停在这儿！

F10 一直走。

* Possible Reference to String Resource ID=00010: "蕙 1%"
                                     |
:00525D37 6A0A                       push 0000000A
:00525D39 52                         push edx
:00525D3A 8BCB                       mov ecx, ebx
:00525D3C E8FEDD0200                 call 00553B3F

* Possible StringData Ref from Data Obj ->"RBK"
                                     |
:00525D41 6880975A00                 push 005A9780
:00525D46 8D4C2420                   lea ecx, dword ptr [esp+20]
:00525D4A E854690300                 call 0055C6A3
:00525D4F 53                         push ebx
:00525D50 8D442420                   lea eax, dword ptr [esp+20]

* Possible StringData Ref from Data Obj ->"--"
                                     |
:00525D54 68A0DE5A00                 push 005ADEA0
:00525D59 8D4C2420                   lea ecx, dword ptr [esp+20]
:00525D5D 50                         push eax
:00525D5E 51                         push ecx
:00525D5F E84B6A0300                 call 0055C7AF
:00525D64 8D542414                   lea edx, dword ptr [esp+14]
:00525D68 50                         push eax
:00525D69 52                         push edx
:00525D6A E8DA690300                 call 0055C749
:00525D6F 8D4C2418                   lea ecx, dword ptr [esp+18]
:00525D73 E8A2670300                 call 0055C51A
:00525D78 51                         push ecx
:00525D79 8D442414                   lea eax, dword ptr [esp+14]
:00525D7D 8BCC                       mov ecx, esp
:00525D7F 50                         push eax
:00525D80 E80A650300                 call 0055C28F
                                     ==== >取得注册信息
:00525D85 E8E6F6EEFF                 call 00415470
                                     ==== >关键 CALL！
:00525D8A 85C0                       test eax, eax
                                     ==== >EAX 为 0 则注册成功！
:00525D8C 0F85E5000000               jne 00525E77
                                     ==== >跳则 OVER！
:00525D92 8B2F                       mov ebp, dword ptr [edi]
:00525D94 E838F10400                 call 00574ED1
:00525D99 8B4004                     mov eax, dword ptr [eax+04]
:00525D9C 55                         push ebp

* Possible StringData Ref from Data Obj ->"User"
                                     |
:00525D9D 6870995A00                 push 005A9970

* Possible StringData Ref from Data Obj ->"License"
                                     |
:00525DA2 68089A5A00                 push 005A9A08
:00525DA7 8BC8                       mov ecx, eax
:00525DA9 E80E100400                 call 00566DBC
:00525DAE E81EF10400                 call 00574ED1
:00525DB3 8B4C2410                   mov ecx, dword ptr [esp+10]
:00525DB7 8B4004                     mov eax, dword ptr [eax+04]
:00525DBA 51                         push ecx

* Possible StringData Ref from Data Obj ->"Code"
                                     |
:00525DBB 6868995A00                 push 005A9968

* Possible StringData Ref from Data Obj ->"License"
```

—————————————————————————————————

`F8` 进入关键 CALL： `00525D85 call 00415470`

```
:00415470 8B442404                   mov eax, dword ptr [esp+04]
                                     ==== >过此 D EAX\=RBK-9912-4444-WXYZ 假码前加 RBK-
:00415474 83EC14                     sub esp, 00000014
:00415477 8B48F8                     mov ecx, dword ptr [eax\-08]
:0041547A 57                         push edi
:0041547B 33FF                       xor edi, edi
:0041547D 83F912                     cmp ecx, 00000012
                                     ==== >比较是否为 18 位，即 注册码=18-4=14 位！
:00415480 0F85AC010000               jne 00415632
                                     ==== >跳则 OVER！
:00415486 8A5003                     mov dl, byte ptr [eax+03]
:00415489 B12D                       mov cl, 2D
                                     ==== > - 移入 CL
:0041548B 3AD1                       cmp dl, cl
                                     ==== >比较第四位（RBK“-”）是否为-
:0041548D 0F859F010000               jne 00415632
                                     ==== >这次当然不跳了，它自己加的嘛
:00415493 384808                     cmp byte ptr [eax+08], cl
                                     ==== >比较第九位（RBK-9912“-”）是否为-
:00415496 0F8596010000               jne 00415632
                                     ==== >跳则 OVER！
:0041549C 38480D                     cmp byte ptr [eax+0D], cl
                                     ==== >比较第十四位（-4444“-”）是否为-
:0041549F 0F858D010000               jne 00415632
                                     ==== >跳则 OVER！
:004154A5 53                         push ebx
:004154A6 56                         push esi
:004154A7 8D442418                   lea eax, dword ptr [esp+18]

* Possible Reference to Dialog: DialogID\_006B, CONTROL\_ID:0003, "h??&L)"
                                     |
:004154AB 6A03                       push 00000003
:004154AD 50                         push eax
:004154AE 8D4C242C                   lea ecx, dword ptr [esp+2C]
:004154B2 E89AE71300                 call 00553C51

* Possible Reference to String Resource ID=00004: ">................"
                                     |
:004154B7 6A04                       push 00000004
:004154B9 8D4C2414                   lea ecx, dword ptr [esp+14]

* Possible Reference to String Resource ID=00004: ">................"
                                     |
:004154BD 6A04                       push 00000004
:004154BF 51                         push ecx
:004154C0 8D4C2430                   lea ecx, dword ptr [esp+30]
:004154C4 E876E61300                 call 00553B3F

* Possible Reference to String Resource ID=00004: ">................"
                                     |
:004154C9 6A04                       push 00000004
:004154CB 8D542418                   lea edx, dword ptr [esp+18]

* Possible Reference to Dialog: DialogID\_00A5, CONTROL\_ID:0009, ""
                                     |
* Possible Reference to String Resource ID=00009: "蕙蝥?
                                     |
:004154CF 6A09                       push 00000009
:004154D1 52                         push edx
:004154D2 8D4C2430                   lea ecx, dword ptr [esp+30]
:004154D6 E864E61300                 call 00553B3F

* Possible Reference to String Resource ID=00004: ">................"
                                     |
:004154DB 6A04                       push 00000004
:004154DD 8D442410                   lea eax, dword ptr [esp+10]

* Possible Reference to String Resource ID=00014: " "
                                     |
:004154E1 6A0E                       push 0000000E
:004154E3 50                         push eax
:004154E4 8D4C2430                   lea ecx, dword ptr [esp+30]
:004154E8 E852E61300                 call 00553B3F

* Possible StringData Ref from Data Obj ->"RBK"
                                     |
:004154ED BE80975A00                 mov esi, 005A9780
:004154F2 8B442418                   mov eax, dword ptr [esp+18]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00415518(C)
|
:004154F6 8A10                       mov dl, byte ptr [eax]
```



```asm
:004154F8 8A1E                 mov bl, byte ptr [esi]

:004154FA 8ACA                 mov cl, dl

:004154FC 3AD3                 cmp dl, bl

:004154FE 751E                 jne 0041551E

:00415500 84C9                 test cl, cl

:00415502 7416                 je 0041551A

:00415504 8A5001               mov dl, byte ptr [eax+01]

:00415507 8A5E01               mov bl, byte ptr [esi+01]

:0041550A 8ACA                 mov cl, dl

:0041550C 3AD3                 cmp dl, bl

:0041550E 750E                 jne 0041551E

:00415510 83C002               add eax, 00000002

:00415513 83C602               add esi, 00000002

:00415516 84C9                 test cl, cl

:00415518 75DC                 jne 004154F6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00415502(C)

|

:0041551A 33C0                 xor eax, eax

:0041551C EB05                 jmp 00415523

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004154FE(C), :0041550E(C)

|

:0041551E 1BC0                 sbb eax, eax

:00415520 83D8FF               sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041551C(U)

|

:00415523 85C0                 test eax, eax

:00415525 0F85DA000000         jne 00415605

:0041552B 8D44241C             lea eax, dword ptr [esp+1C]

:0041552F 6A02                 push 00000002

:00415531 50                   push eax

:00415532 8D4C2418             lea ecx, dword ptr [esp+18]

:00415536 E89AE61300           call 00553BD5

:0041553B 8B00                 mov eax, dword ptr [eax]

:0041553D 50                   push eax

:0041553E E8C2D21200           call 00542805

:00415543 83C404               add esp, 00000004

:00415546 8D4C241C             lea ecx, dword ptr [esp+1C]

:0041554A 8BF0                 mov esi, eax

:0041554C E8C96F1400           call 0055C51A

:00415551 8B4C2410             mov ecx, dword ptr [esp+10]

:00415555 51                   push ecx
                                ====>D ECX=9912

:00415556 E8AAD21200           call 00542805
                                ====>检测假码前 4 位是否为数字？且 3、4 位要大于 00

:0041555B 83C404               add esp, 00000004

:0041555E 85C0                 test eax, eax

:00415560 0F849F000000         je 00415605
                                ====>不能跳！

:00415566 83FE01               cmp esi, 00000001

:00415569 0F8C96000000         jl 00415605

:0041556F 83FE0C               cmp esi, 0000000C
                                ====>比较第三、四位是否小于 或 等于“12”

:00415572 0F8F8D000000         jg 00415605
                                ====>不能跳！

:00415578 8B442414             mov eax, dword ptr [esp+14]

* Possible StringData Ref from Data Obj ->"3437"

|

:0041557C BE78975A00           mov esi, 005A9778

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004155A3(C)

|

:00415581 8A10                 mov dl, byte ptr [eax]
                                ====>D EAX=4444

:00415583 8A1E                 mov bl, byte ptr [esi]
                                ====>D ESI=3437    第 2 组数固定为 3437

:00415585 8ACA                 mov cl, dl

:00415587 3AD3                 cmp dl, bl
                                ====>逐位比较。 因此：改 4444 为 3437

:00415589 751E                 jne 004155A9

:0041558B 84C9                 test cl, cl

:0041558D 7416                 je 004155A5

:0041558F 8A5001               mov dl, byte ptr [eax+01]

:00415592 8A5E01               mov bl, byte ptr [esi+01]

:00415595 8ACA                 mov cl, dl

:00415597 3AD3                 cmp dl, bl

:00415599 750E                 jne 004155A9

:0041559B 83C002               add eax, 00000002

:0041559E 83C602               add esi, 00000002

:004155A1 84C9                 test cl, cl

:004155A3 75DC                 jne 00415581

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041558D(C)

|

:004155A5 33C0                 xor eax, eax

:004155A7 EB05                 jmp 004155AE

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00415589(C), :00415599(C)

|

:004155A9 1BC0                 sbb eax, eax

:004155AB 83D8FF               sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004155A7(U)

|

:004155AE 85C0                 test eax, eax

:004155B0 7553                 jne 00415605

:004155B2 8B44240C             mov eax, dword ptr [esp+0C]

:004155B6 0FBE4801             movsx ecx, byte ptr [eax+01]

:004155BA 51                   push ecx
                                ====> ? ECX=58  即：X

:004155BB E8E6D61200           call 00542CA6
                                ====>检测第十六位（即真码第 12 位）是否为数字？

:004155C0 83C404               add esp, 00000004

:004155C3 85C0                 test eax, eax

:004155C5 743E                 je 00415605
                                ====>不能跳！ r fl z

:004155C7 8B54240C             mov edx, dword ptr [esp+0C]

:004155CB 0FBE4202             movsx eax, byte ptr [edx+02]

:004155CF 50                   push eax
                                ====> ? EAX=59  即：Y

:004155D0 E8D1D61200           call 00542CA6
                                ====>检测第十七位（即真码第 13 位）是否为数字？

:004155D5 83C404               add esp, 00000004

:004155D8 85C0                 test eax, eax

:004155DA 7429                 je 00415605
                                ====>不能跳！ r fl z

:004155DC 8B4C240C             mov ecx, dword ptr [esp+0C]

:004155E0 0FBE5103             movsx edx, byte ptr [ecx+03]

:004155E4 52                   push edx
                                ====>？EDX=5a  即：Z

:004155E5 E8BCD61200           call 00542CA6
                                ====>检测第十八位（即真码第 14 位）是否为数字？

:004155EA 83C404               add esp, 00000004

:004155ED 85C0                 test eax, eax

:004155EF 7414                 je 00415605
                                ====>不能跳！ r fl z

:004155F1 8B44240C             mov eax, dword ptr [esp+0C]

:004155F5 0FBE08               movsx ecx, byte ptr [eax]

:004155F8 51                   push ecx
                                ====> ？ECX=57  即：W

:004155F9 E852D61200           call 00542C50
                                ====>检测第十五位（即真码第 11 位）是否为字母？

:004155FE 83C404               add esp, 00000004

:00415601 85C0                 test eax, eax

:00415603 7505                 jne 0041560A
                                ====>正确则跳！！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00415525(C), :00415560(C), :00415569(C), :00415572(C), :004155B0(C)

|:004155C5(C), :004155DA(C), :004155EF(C)

|

:00415605 BF01000000           mov edi, 00000001
                                ====>EDI 置 1。暴力破解改此处

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00415603(C)

|

:0041560A 8D4C240C             lea ecx, dword ptr [esp+0C]

:0041560E E8076F1400           call 0055C51A

:00415613 8D4C2414             lea ecx, dword ptr [esp+14]

:00415617 E8FE6E1400           call 0055C51A

:0041561C 8D4C2410             lea ecx, dword ptr [esp+10]

:00415620 E8F56E1400           call 0055C51A

:00415625 8D4C2418             lea ecx, dword ptr [esp+18]

:00415629 E8EC6E1400           call 0055C51A

:0041562E 5E                   pop esi

:0041562F 5B                   pop ebx
```



#### 汇编代码分析

```
:00415630 EB05                 jmp 00415637
```

- Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
  `:00415480(C)`, `:0041548D(C)`, `:00415496(C)`, `:0041549F(C)`  

```
:00415632 BF01000000           mov edi, 00000001
```
> =====> `EDI` 置 1。暴力破解改此处！

- Referenced by a (U)nconditional or (C)onditional Jump at Address:  
  `:00415630(U)`  

```
:00415637 8D4C241C             lea ecx, dword ptr [esp+1C]
:0041563B E8DA6E1400           call 0055C51A
:00415640 8BC7                 mov eax, edi
:00415642 5F                   pop edi
:00415643 83C414               add esp, 00000014
:00415646 C20400               ret 0004
```

#### 总结

注册码共 14 位，与名称、E-Mail 无关。形式为：`??12-3437-????`

- 第 1、2 位为数字。
- 3、4 位介于 `00-13` 之间。
- `3437` 固定。
- 第 11 位为字母。
- 第 12、13、14 位为数字。

一个可用之注册码：`9912-3437-X444`

#### 注册信息保存

```
REGEDIT4

[HKEY_CURRENT_USER\Software\RimArts\B2\License]
"Agreed"=dword:00000001
"User"="fly"
"Code"="RBK-9912-3437-X444"
"EMail"="fly4099@sohu.com"
```

#### 完美爆破

用 `HIEW` 吧！`F5` 去修改地址，`F3` 进入修改状态，直接改完后 `F9` 保存，`F10` 退出。

1. `00415605 BF01000000` `mov edi, 00000001` <---- `EDI` 置 1
    - 修改为：`MOV EDI，00000000`，`BF01000000` 改为 `BF00000000`

2. `00415632 BF01000000` `mov edi, 00000001` <---- `EDI` 置 1
    - 修改为：`MOV EDI，00000000`，`BF01000000` 改为 `BF00000000`

---

Cracked By 巢水工作坊——fly  
2002-9-10 `4dd466d0e1e0cc123afbe504da2462ca`
