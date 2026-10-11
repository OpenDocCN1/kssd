
# 广电节目管理系统 V0.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50642.htm`](https://www.pediy.com/kssd/pediy05/pediy50642.htm)

下载页面： [`liyueqi.myetang.com/`](http://liyueqi.myetang.com/)  
软件大小： 148K  
加入时间： 2003-04-21  
适用平台: Win9x, WinME, WinNT

**【软件简介】：** 用于电视台、广播电台、影剧院、电影院或录像厅的各类节目分类、记录、管理、编排、查询。

**【软件限制】：** 功能限制

**【作者声明】：** 初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】：** `TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

---

**【过程】：**

呵呵，`showha` 朋友已经追到了注册码，我也沾点光，找找算法吧。

`BroadAct.exe` 无壳。`Visual Basic 6.0` 编写。

可以用 `VB 6.0` 的“经典”比较语句 `BPX MSVBVM60.__vbaStrCmp` 拦截，很方便呀。

用户号：`894112337`  
试炼码：`13572468`

---

```
* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
|
:0042BC48 FF1568104000         Call dword ptr [00401068]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042BC3A(C)
|
:0042BC4E 8B55D8               mov edx, dword ptr [ebp-28]
====>EDX=[ebp-28]=894112337  用户号！

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
|
:0042BC51 8B1DD4114000         mov ebx, dword ptr [004011D4]
:0042BC57 8D4DD4               lea ecx, dword ptr [ebp-2C]
:0042BC5A 897DD8               mov dword ptr [ebp-28], edi
:0042BC5D FFD3                 call ebx
:0042BC5F 8B4DDC               mov ecx, dword ptr [ebp-24]
====>ECX=[ebp-24]=13572468  试炼码

:0042BC62 8D55D4               lea edx, dword ptr [ebp-2C]
:0042BC65 51                   push ecx
:0042BC66 52                   push edx
:0042BC67 E854180000           call 0042D4C0
====>算法 CALL！进入！

:0042BC6C 8BD0                 mov edx, eax
====>EDX=EAX=295129513        注册码！

:0042BC6E 8D4DD0               lea ecx, dword ptr [ebp-30]
:0042BC71 FFD3                 call ebx
:0042BC73 50                   push eax

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
|
:0042BC74 FF15CC104000         Call dword ptr [004010CC]
====>比较 CALL!

:0042BC7A 8BF0                 mov esi, eax
:0042BC7C 8D45D0               lea eax, dword ptr [ebp-30]
:0042BC7F F7DE                 neg esi
:0042BC81 8D4DDC               lea ecx, dword ptr [ebp-24]
:0042BC84 50                   push eax
:0042BC85 1BF6                 sbb esi, esi
====>爆破点！

:0042BC87 8D55D4               lea edx, dword ptr [ebp-2C]
:0042BC8A 51                   push ecx
:0042BC8B 46                   inc esi
:0042BC8C 52                   push edx
:0042BC8D 6A03                 push 00000003
:0042BC8F F7DE                 neg esi

* Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h
|
:0042BC91 FF1588114000         Call dword ptr [00401188]
:0042BC97 8D45C8               lea eax, dword ptr [ebp-38]
:0042BC9A 8D4DCC               lea ecx, dword ptr [ebp-34]
:0042BC9D 50                   push eax
:0042BC9E 51                   push ecx
:0042BC9F 6A02                 push 00000002

* Reference To: MSVBVM60.__vbaFreeObjList, Ord:0000h
|
:0042BCA1 FF1540104000         Call dword ptr [00401040]
:0042BCA7 83C41C               add esp, 0000001C
:0042BCAA 663BF7               cmp si, di
:0042BCAD 0F847C010000         je 0042BE2F
====>跳则 OVER！

:0042BCB3 E888130000           call 0042D040
:0042BCB8 668B1544804300       mov dx, word ptr [00438044]
:0042BCBF 52                   push edx
:0042BCC0 6A01                 push 00000001
:0042BCC2 6854804300           push 00438054

* Possible StringData Ref from Code Obj ->"LL?
|
:0042BCC7 68E0D54000           push 0040D5E0

* Reference To: MSVBVM60.__vbaRecDestruct, Ord:0000h
|
:0042BCCC FF1564104000         Call dword ptr [00401064]
:0042BCD2 50                   push eax
:0042BCD3 683CEE4000           push 0040EE3C

* Reference To: MSVBVM60.__vbaGetOwner4, Ord:0000h
|
:0042BCD8 FF154C114000         Call dword ptr [0040114C]
:0042BCDE 8D8544FFFFFF         lea eax, dword ptr [ebp+FFFFFF44]
:0042BCE4 C705C0804300CA546842  mov dword ptr [004380C0], 426854CA
:0042BCEE 50                   push eax
:0042BCEF C78544FFFFFF12000000  mov dword ptr [ebp+FFFFFF44], 00000012
:0042BCF9 E8721B0000           call 0042D870
:0042BCFE 8BD0                 mov edx, eax
:0042BD00 8D4DE4               lea ecx, dword ptr [ebp-1C]
:0042BD03 FFD3                 call ebx
:0042BD05 8B4DE4               mov ecx, dword ptr [ebp-1C]
:0042BD08 51                   push ecx

====>下面写注册信息！

* Possible StringData Ref from Code Obj ->"KKey02"
|
:0042BD09 68ACEE4000           push 0040EEAC

* Possible StringData Ref from Code Obj ->"SSection1"
|
:0042BD0E 6894EE4000           push 0040EE94

* Possible StringData Ref from Code Obj ->"FFbuklvkTtsjqccQvapzoy"
|
:0042BD13 680CD24000           push 0040D20C

* Reference To: MSVBVM60.rtcSaveSetting, Ord:02B2h
|
:0042BD18 FF1504104000         Call dword ptr [00401004]
:0042BD1E 8D55E4               lea edx, dword ptr [ebp-1C]
:0042BD21 52                   push edx
:0042BD22 E8C91C0000           call 0042D9F0
:0042BD27 8BD0                 mov edx, eax
:0042BD29 8D4DDC               lea ecx, dword ptr [ebp-24]
:0042BD2C FFD3                 call ebx
:0042BD2E 50                   push eax
:0042BD2F 68C4804300           push 004380C4
:0042BD34 6A12                 push 00000012

* Reference To: MSVBVM60.__vbaLsetFixstr, Ord:0000h
|
:0042BD36 FF1554104000         Call dword ptr [00401054]
:0042BD3C 8D4DDC               lea ecx, dword ptr [ebp-24]

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
|
:0042BD3F FF15F0114000         Call dword ptr [004011F0]
:0042BD45 66A144804300         mov ax, word ptr [00438044]
:0042BD4B 50                   push eax
:0042BD4C 6A01                 push 00000001
:0042BD4E 6854804300           push 00438054
:0042BD53 683CEE4000           push 0040EE3C

* Reference To: MSVBVM60.__vbaPutOwner4, Ord:0000h
|
:0042BD58 FF15D4104000         Call dword ptr [004010D4]
:0042BD5E 668B0D44804300       mov cx, word ptr [00438044]
:0042BD65 51                   push ecx

* Reference To: MSVBVM60.__vbaFileClose, Ord:0000h
|
:0042BD66 FF15C4104000         Call dword ptr [004010C4]
:0042BD6C B904000280           mov ecx, 80020004
:0042BD71 B80A000000           mov eax, 0000000A
:0042BD76 894D90               mov dword ptr [ebp-70], ecx
:0042BD79 894DA0               mov dword ptr [ebp-60], ecx
:0042BD7C 894DB0               mov dword ptr [ebp-50], ecx
:0042BD7F 8D9578FFFFFF         lea edx, dword ptr [ebp+FFFFFF78]
:0042BD85 8D4DB8               lea ecx, dword ptr [ebp-48]
:0042BD88 894588               mov dword ptr [ebp-78], eax
:0042BD8B 894598               mov dword ptr [ebp-68], eax
```



`0042BD8E 8945A8` | `mov dword ptr [ebp-58], eax`

* Possible StringData Ref from Code Obj -> `"騗鑜孮."`

`:0042BD91 C74580BCEE4000` | `mov [ebp-80], 0040EEBC`

`:0042BD98 C78578FFFFFF08000000` | `mov dword ptr [ebp+FFFFFF78], 00000008`

* Reference To: `MSVBVM60.__vbaVarDup`, Ord:`0000h`

`:0042BDA2 FF15B8114000` | `Call dword ptr [004011B8]`

`:0042BDA8 8D5588` | `lea edx, dword ptr [ebp-78]`

`:0042BDAB 8D4598` | `lea eax, dword ptr [ebp-68]`

`:0042BDAE 52` | `push edx`

`:0042BDAF 8D4DA8` | `lea ecx, dword ptr [ebp-58]`

`:0042BDB2 50` | `push eax`

`:0042BDB3 51` | `push ecx`

`:0042BDB4 8D55B8` | `lea edx, dword ptr [ebp-48]`

`:0042BDB7 6A40` | `push 00000040`

`:0042BDB9 52` | `push edx`

* Reference To: `MSVBVM60.rtcMsgBox`, Ord:`0253h`

`:0042BDBA FF158C104000` | `Call dword ptr [0040108C]`

====> 呵呵，胜利女神！

…… ……省 略…… ……

* Reference To: `MSVBVM60.rtcMsgBox`, Ord:`0253h`

`:0042BE7D FF158C104000` | `Call dword ptr [0040108C]`

====> BAD BOY！

—————————————————————————————————

进入算法 `CALL`：`0042BC67 call 0042D4C0`

* Referenced by a CALL at Address:

`:0042BC67`

`:0042D4C0 55` | `push ebp`

`:0042D4C1 8BEC` | `mov ebp, esp`

`:0042D4C3 83EC0C` | `sub esp, 0000000C`

* Possible StringData Ref from Code Obj -> `"“%$@"`

`:0042D4C6 68361C4000` | `push 00401C36`

`:0042D4CB 64A100000000` | `mov eax, dword ptr fs:[00000000]`

`:0042D4D1 50` | `push eax`

`:0042D4D2 64892500000000` | `mov dword ptr fs:[00000000], esp`

`:0042D4D9 81EC44010000` | `sub esp, 00000144`

`:0042D4DF 53` | `push ebx`

`:0042D4E0 56` | `push esi`

`:0042D4E1 57` | `push edi`

`:0042D4E2 8965F4` | `mov dword ptr [ebp-0C], esp`

`:0042D4E5 C745F830194000` | `mov [ebp-08], 00401930`

`:0042D4EC 8B5D08` | `mov ebx, dword ptr [ebp+08]`

`:0042D4EF 33C0` | `xor eax, eax`

* Reference To: `MSVBVM60.rtcMidCharVar`, Ord:`0278h`

`:0042D4F1 8B35B8104000` | `mov esi, dword ptr [004010B8]`

`:0042D4F7 8945E4` | `mov dword ptr [ebp-1C], eax`

`:0042D4FA 8945E0` | `mov dword ptr [ebp-20], eax`

`:0042D4FD 8945DC` | `mov dword ptr [ebp-24], eax`

`:0042D500 8945D8` | `mov dword ptr [ebp-28], eax`

`:0042D503 8945B8` | `mov dword ptr [ebp-48], eax`

`:0042D506 8945A8` | `mov dword ptr [ebp-58], eax`

`:0042D509 894598` | `mov dword ptr [ebp-68], eax`

`:0042D50C 894588` | `mov dword ptr [ebp-78], eax`

`:0042D50F 898578FFFFFF` | `mov dword ptr [ebp+FFFFFF78], eax`

`:0042D515 898568FFFFFF` | `mov dword ptr [ebp+FFFFFF68], eax`

`:0042D51B 898558FFFFFF` | `mov dword ptr [ebp+FFFFFF58], eax`

`:0042D521 898548FFFFFF` | `mov dword ptr [ebp+FFFFFF48], eax`

`:0042D527 898538FFFFFF` | `mov dword ptr [ebp+FFFFFF38], eax`

`:0042D52D 898508FFFFFF` | `mov dword ptr [ebp+FFFFFF08], eax`

`:0042D533 8985E8FEFFFF` | `mov dword ptr [ebp+FFFFFEE8], eax`

`:0042D539 8985C8FEFFFF` | `mov dword ptr [ebp+FFFFFEC8], eax`

`:0042D53F 8985B8FEFFFF` | `mov dword ptr [ebp+FFFFFEB8], eax`

`:0042D545 8D45C8` | `lea eax, dword ptr [ebp-38]`

`:0042D548 8D8D28FFFFFF` | `lea ecx, dword ptr [ebp+FFFFFF28]`

`:0042D54E 50` | `push eax`

`:0042D54F 6A04` | `push 00000004`

`:0042D551 8D55B8` | `lea edx, dword ptr [ebp-48]`

`:0042D554 51` | `push ecx`

`:0042D555 52` | `push edx`

`:0042D556 C745D004000280` | `mov [ebp-30], 80020004`

`:0042D55D C745C80A000000` | `mov [ebp-38], 0000000A`

`:0042D564 899D30FFFFFF` | `mov dword ptr [ebp+FFFFFF30], ebx`

`:0042D56A C78528FFFFFF08400000` | `mov dword ptr [ebp+FFFFFF28], 00004008`

`:0042D574 FFD6` | `call esi`

====>取字符！取用户号 `894112337` 的前 3 位：`894`

`:0042D576 8D45A8` | `lea eax, dword ptr [ebp-58]`

`:0042D579 8D8D08FFFFFF` | `lea ecx, dword ptr [ebp+FFFFFF08]`

`:0042D57F 50` | `push eax`

`:0042D580 6A01` | `push 00000001`

`:0042D582 8D5598` | `lea edx, dword ptr [ebp-68]`

`:0042D585 BF02000000` | `mov edi, 00000002`

`:0042D58A 51` | `push ecx`

`:0042D58B 52` | `push edx`

`:0042D58C C745B003000000` | `mov [ebp-50], 00000003`

`:0042D593 897DA8` | `mov dword ptr [ebp-58], edi`

`:0042D596 899D10FFFFFF` | `mov dword ptr [ebp+FFFFFF10], ebx`

`:0042D59C C78508FFFFFF08400000` | `mov dword ptr [ebp+FFFFFF08], 00004008`

`:0042D5A6 FFD6` | `call esi`

====>取字符！取用户号 `894112337` 的后 6 位：`112337`

`:0042D5A8 8D45B8` | `lea eax, dword ptr [ebp-48]`

`:0042D5AB 8D4D98` | `lea ecx, dword ptr [ebp-68]`

`:0042D5AE 50` | `push eax`

`:0042D5AF 8D5588` | `lea edx, dword ptr [ebp-78]`

`:0042D5B2 51` | `push ecx`

`:0042D5B3 52` | `push edx`

* Reference To: `MSVBVM60.__vbaVarCat`, Ord:`0000h`

`:0042D5B4 FF1550114000` | `Call dword ptr [00401150]`

`:0042D5BA 50` | `push eax`

* Reference To: `MSVBVM60.__vbaStrVarMove`, Ord:`0000h`

`:0042D5BB FF1528104000` | `Call dword ptr [00401028]`

====>移动前 3 位到末尾！

* Reference To: `MSVBVM60.__vbaStrMove`, Ord:`0000h`

`:0042D5C1 8B1DD4114000` | `mov ebx, dword ptr [004011D4]`

`:0042D5C7 8BD0` | `mov edx, eax`

====>`EDX=EAX=112337894`

`:0042D5C9 8D4DE4` | `lea ecx, dword ptr [ebp-1C]`

`:0042D5CC FFD3` | `call ebx`

`:0042D5CE 8D4588` | `lea eax, dword ptr [ebp-78]`

`:0042D5D1 8D4D98` | `lea ecx, dword ptr [ebp-68]`

`:0042D5D4 50` | `push eax`

`:0042D5D5 8D55B8` | `lea edx, dword ptr [ebp-48]`

`:0042D5D8 51` | `push ecx`

`:0042D5D9 8D45A8` | `lea eax, dword ptr [ebp-58]`

`:0042D5DC 52` | `push edx`

`:0042D5DD 8D4DC8` | `lea ecx, dword ptr [ebp-38]`

`:0042D5E0 50` | `push eax`

`:0042D5E1 51` | `push ecx`

`:0042D5E2 6A05` | `push 00000005`

* Reference To: `MSVBVM60.__vbaFreeVarList`, Ord:`0000h`

`:0042D5E4 FF1530104000` | `Call dword ptr [00401030]`

`:0042D5EA 83C418` | `add esp, 00000018`

`:0042D5ED BA04DE4000` | `mov edx, 0040DE04`

`:0042D5F2 8D4DE0` | `lea ecx, dword ptr [ebp-20]`

* Reference To: `MSVBVM60.__vbaStrCopy`, Ord:`0000h`

`:0042D5F5 FF1580114000` | `Call dword ptr [00401180]`

`:0042D5FB 8B55E4` | `mov edx, dword ptr [ebp-1C]`

====>`EDX=[ebp-1C]=112337894`

`:0042D5FE 52` | `push edx`

* Reference To: `MSVBVM60.__vbaLenBstr`, Ord:`0000h`



## 汇编代码分析

### 初始化与调用
```
:0042D5FF FF1520104000       Call dword ptr [00401020]

====>取 112337894 长度

:0042D605 8BC8               mov ecx, eax

====>ECX=EAX=9

* Reference To: MSVBVM60.__vbaUI1I4, Ord:0000h

:0042D607 FF1518114000       Call dword ptr [00401118]

:0042D60D 660FB6C0           movzx ax, al

:0042D611 668985B0FEFFFF     mov word ptr [ebp+FFFFFEB0], ax

:0042D618 66B80100           mov ax, 0001
```

### 比较与分支
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:0042D7C6(U)

:0042D61C 663B85B0FEFFFF     cmp ax, word ptr [ebp+FFFFFEB0]

:0042D623 66A340814300       mov word ptr [00438140], ax

:0042D629 0F8F9C010000       jg 0042D7CB

:0042D62F 8D4DE4             lea ecx, dword ptr [ebp-1C]

:0042D632 8D55C8             lea edx, dword ptr [ebp-38]

:0042D635 0FBFC0             movsx eax, ax

:0042D638 898D30FFFFFF       mov dword ptr [ebp+FFFFFF30], ecx

:0042D63E 52                 push edx

:0042D63F 8D8D28FFFFFF       lea ecx, dword ptr [ebp+FFFFFF28]

:0042D645 50                 push eax

:0042D646 8D55B8             lea edx, dword ptr [ebp-48]

:0042D649 51                 push ecx

:0042D64A 52                 push edx

:0042D64B C745D001000000     mov [ebp-30], 00000001

:0042D652 897DC8             mov dword ptr [ebp-38], edi

:0042D655 C78528FFFFFF08400000 mov dword ptr [ebp+FFFFFF28], 00004008

:0042D65F FFD6               call esi
```
====>依次取变化后的用户号 `112337894` 数字
1、====>取 1
2、====>取 1
3、====>取 2
4、====>取 3
5、====>取 3
6、====>取 7
7、====>取 8
8、====>取 9
9、====>取 4

```
:0042D668 8D45E4             lea eax, dword ptr [ebp-1C]

:0042D66B 8D4DA8             lea ecx, dword ptr [ebp-58]

:0042D66E 898510FFFFFF       mov dword ptr [ebp+FFFFFF10], eax

:0042D674 51                 push ecx

:0042D675 8D8508FFFFFF       lea eax, dword ptr [ebp+FFFFFF08]

:0042D67B 52                 push edx

:0042D67C 8D4D98             lea ecx, dword ptr [ebp-68]

:0042D67F 50                 push eax

:0042D680 51                 push ecx

:0042D681 C745B001000000     mov [ebp-50], 00000001

:0042D688 897DA8             mov dword ptr [ebp-58], edi

:0042D68B C78508FFFFFF08400000 mov dword ptr [ebp+FFFFFF08], 00004008

:0042D695 FFD6               call esi
```
====>依次取变化后的用户号 `112337894` 数字
1、====>取 1
2、====>取 1
3、====>取 2
4、====>取 3
5、====>取 3
6、====>取 7
7、====>取 8
8、====>取 9
9、====>取 4

```
:0042D697 0FBF0D40814300     movsx ecx, word ptr [00438140]

:0042D69E 8D55E4             lea edx, dword ptr [ebp-1C]

:0042D6A1 8D8578FFFFFF       lea eax, dword ptr [ebp+FFFFFF78]

:0042D6A7 8995F0FEFFFF       mov dword ptr [ebp+FFFFFEF0], edx

:0042D6AD 50                 push eax

:0042D6AE 8D95E8FEFFFF       lea edx, dword ptr [ebp+FFFFFEE8]

:0042D6B4 51                 push ecx

:0042D6B5 8D8568FFFFFF       lea eax, dword ptr [ebp+FFFFFF68]

:0042D6BB 52                 push edx

:0042D6BC 50                 push eax

:0042D6BD C7458001000000     mov [ebp-80], 00000001

:0042D6C4 89BD78FFFFFF       mov dword ptr [ebp+FFFFFF78], edi

:0042D6CA C785E8FEFFFF08400000 mov dword ptr [ebp+FFFFFEE8], 00004008

:0042D6D4 FFD6               call esi
```
====>依次取变化后的用户号 `112337894` 数字
1、====>取 1
2、====>取 1
3、====>取 2
4、====>取 3
5、====>取 3
6、====>取 7
7、====>取 8
8、====>取 9
9、====>取 4

### 循环计算
```
:0042D6D6 66A140814300       mov ax, word ptr [00438140]

====>AX 是循环次数！依次增 1。

:0042D6DC 8D55B8             lea edx, dword ptr [ebp-48]

:0042D6DF 668BC8             mov cx, ax

====>CX=AX

:0042D6E2 52                 push edx

:0042D6E3 660FAFC8           imul cx, ax
```
====>第一次：求循环次数的平方
1、====>CX=1 * 1=1
2、====>CX=2 * 2=4
3、====>CX=3 * 3=9
4、====>CX=4 * 4=10
5、====>CX=5 * 5=19
6、====>CX=6 * 6=24
7、====>CX=7 * 7=31
8、====>CX=8 * 8=40
9、====>CX=9 * 9=51

```
:0042D6E7 0F806F010000       jo 0042D85C

:0042D6ED 660FAFC8           imul cx, ax
```
====>第二次：循环次数的平方值和循环次数相乘！其实这两步求循环次数的立方值！
1、====>CX=1 * 1=1 （H）=1 （D）
2、====>CX=4 * 2=8 （H）=8 （D）
3、====>CX=9 * 3=1B （H）=27 （D）
4、====>CX=10 * 4=40 （H）=64 （D）
5、====>CX=19 * 5=7D （H）=125（D）
6、====>CX=24 * 6=D8 （H）=216（D）
7、====>CX=31 * 7=157（H）=343（D）
8、====>CX=40 * 8=200（H）=512（D）
9、====>CX=51 * 9=2D9（H）=729（D）

```
:0042D6F1 0F8065010000       jo 0042D85C

:0042D6F7 66898DD0FEFFFF     mov word ptr [ebp+FFFFFED0], cx

:0042D6FE 8D4598             lea eax, dword ptr [ebp-68]

:0042D701 8D4D88             lea ecx, dword ptr [ebp-78]

:0042D704 50                 push eax

:0042D705 51                 push ecx

:0042D706 89BDC8FEFFFF       mov dword ptr [ebp+FFFFFEC8], edi

:0042D70C C785C0FEFFFF0A000000 mov dword ptr [ebp+FFFFFEC0], 0000000A

:0042D716 89BDB8FEFFFF       mov dword ptr [ebp+FFFFFEB8], edi

* Reference To: MSVBVM60.__vbaVarMul, Ord:0000h

:0042D71C FF151C114000       Call dword ptr [0040111C]
```
====>第三次：求字符值的平方
呵呵，里面是浮点运算： `77A10621` `DC4E 08` `FMUL QWORD PTR DS:[ESI+8]`
1、====>1.0000000000000000000 * 1.0000000000000000000=1.0000000000000000000
2、====>1.0000000000000000000 * 1.0000000000000000000=1.0000000000000000000
3、====>2.0000000000000000000 * 2.0000000000000000000=4.0000000000000000000
4、====>3.0000000000000000000 * 3.0000000000000000000=9.0000000000000000000
5、====>3.0000000000000000000 * 3.0000000000000000000=9.0000000000000000000
6、====>7.0000000000000000000 * 7.0000000000000000000=49.000000000000000000
7、====>8.0000000000000000000 * 8.0000000000000000000=64.000000000000000000




## 计算过程分析

### 第四步：字符值的平方

`8、 ====>9.0000000000000000000 * 9.0000000000000000000=81.000000000000000000`
`9、 ====>4.0000000000000000000 * 4.0000000000000000000=16.000000000000000000`

### 汇编指令：乘法运算

```
:0042D722 50                             push eax
:0042D723 8D9568FFFFFF                  lea edx, dword ptr [ebp+FFFFFF68]
:0042D729 8D8558FFFFFF                  lea eax, dword ptr [ebp+FFFFFF58]
:0042D72F 52                             push edx
:0042D730 50                             push eax
```

* Reference To: `MSVBVM60.__vbaVarMul`, `Ord:0000h`

```
:0042D731 FF151C114000                  Call dword ptr [0040111C]
```

====> 第四次：字符值 与 字符值的平方 相乘！

其实这两步求字符数字的立方值！
`77A10621 DC4E 08 FMUL QWORD PTR DS:[ESI+8]`

### 计算过程分析：立方值

`1、 ====>1.0000000000000000000 * 1.0000000000000000000=1.0000000000000000000`
`2、 ====>1.0000000000000000000 * 1.0000000000000000000=1.0000000000000000000`
`3、 ====>2.0000000000000000000 * 4.0000000000000000000=8.0000000000000000000`
`4、 ====>3.0000000000000000000 * 9.0000000000000000000=27.000000000000000000`
`5、 ====>3.0000000000000000000 * 9.0000000000000000000=27.000000000000000000`
`6、 ====>7.0000000000000000000 * 49.000000000000000000=343.00000000000000000`
`7、 ====>8.0000000000000000000 * 64.000000000000000000=512.00000000000000000`
`8、 ====>9.0000000000000000000 * 81.000000000000000000=729.00000000000000000`
`9、 ====>4.0000000000000000000 * 16.000000000000000000=64.000000000000000000`

### 汇编指令：加法运算

```
:0042D737 8D8DC8FEFFFF                  lea ecx, dword ptr [ebp+FFFFFEC8]
:0042D73D 50                             push eax
:0042D73E 8D9548FFFFFF                  lea edx, dword ptr [ebp+FFFFFF48]
:0042D744 51                             push ecx
:0042D745 52                             push edx
```

* Reference To: `MSVBVM60.__vbaVarAdd`, `Ord:0000h`

```
:0042D746 FF15B0114000                  Call dword ptr [004011B0]
```

====> 第五次：循环次数的立方值 和 字符的立方值 相加!
`77A0F5A4 DC46 08 FADD QWORD PTR DS:[ESI+8]`

### 计算过程分析：累加

`1、 ====>1.0000000000000000000 + 1.0000000000000000000=2.0000000000000000000`
`2、 ====>8.0000000000000000000 + 1.0000000000000000000=9.0000000000000000000`
`3、 ====>27.000000000000000000 + 8.0000000000000000000=35.000000000000000000`
`4、 ====>64.000000000000000000 + 27.000000000000000000=91.000000000000000000`
`5、 ====>125.00000000000000000 + 27.000000000000000000=152.00000000000000000`
`6、 ====>216.00000000000000000 + 343.00000000000000000=559.00000000000000000`
`7、 ====>343.00000000000000000 + 512.00000000000000000=855.00000000000000000`
`8、 ====>512.00000000000000000 + 729.00000000000000000=1241.0000000000000000`
`9、 ====>729.00000000000000000 + 64.000000000000000000=793.00000000000000000`

### 汇编指令：取模运算

```
:0042D74C 50                             push eax
:0042D74D 8D85B8FEFFFF                  lea eax, dword ptr [ebp+FFFFFEB8]
:0042D753 8D8D38FFFFFF                  lea ecx, dword ptr [ebp+FFFFFF38]
:0042D759 50                             push eax
:0042D75A 51                             push ecx
```

* Reference To: `MSVBVM60.__vbaVarMod`, `Ord:0000h`

```
:0042D75B FF15C0114000                  Call dword ptr [004011C0]
```

====> 第六次：上面所得的值的 16 进制值 与 A 求模

呵呵，用 `Ollydbg` 复制的代码
```
77A11A79 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]
77A11A7C 99             CDQ
77A11A7D F77D F8        IDIV DWORD PTR SS:[EBP-8]
77A11A80 8956 08        MOV DWORD PTR DS:[ESI+8],EDX
```

### 取模结果计算

`1、 ====>EDX=2 % A=2`
`2、 ====>EDX=9 % A=9`
`3、 ====>EDX=23 % A=5`
`4、 ====>EDX=5B % A=1`
`5、 ====>EDX=98 % A=2`
`6、 ====>EDX=22F % A=9`
`7、 ====>EDX=357 % A=5`
`8、 ====>EDX=4D9 % A=1`
`9、 ====>EDX=319 % A=3`

====> 九次循环运算得出：`295129513`

### 汇编指令：变量释放

```
:0042D761 50                             push eax
```

* Reference To: `MSVBVM60.__vbaStrVarMove`, `Ord:0000h`

```
:0042D762 FF1528104000                  Call dword ptr [00401028]
:0042D768 8BD0                           mov edx, eax
:0042D76A 8D4DD8                         lea ecx, dword ptr [ebp-28]
:0042D76D FFD3                           call ebx
:0042D76F 8D9548FFFFFF                  lea edx, dword ptr [ebp+FFFFFF48]
:0042D775 8D8568FFFFFF                  lea eax, dword ptr [ebp+FFFFFF68]
:0042D77B 52                             push edx
:0042D77C 8D8D78FFFFFF                  lea ecx, dword ptr [ebp+FFFFFF78]
:0042D782 50                             push eax
:0042D783 8D5598                         lea edx, dword ptr [ebp-68]
:0042D786 51                             push ecx
:0042D787 8D45B8                         lea eax, dword ptr [ebp-48]
:0042D78A 52                             push edx
:0042D78B 8D4DA8                         lea ecx, dword ptr [ebp-58]
:0042D78E 50                             push eax
:0042D78F 8D55C8                         lea edx, dword ptr [ebp-38]
:0042D792 51                             push ecx
:0042D793 52                             push edx
:0042D794 6A07                           push 00000007
```

* Reference To: `MSVBVM60.__vbaFreeVarList`, `Ord:0000h`

```
:0042D796 FF1530104000                  Call dword ptr [00401030]
:0042D79C 8B45E0                         mov eax, dword ptr [ebp-20]
:0042D79F 8B4DD8                         mov ecx, dword ptr [ebp-28]
:0042D7A2 83C420                         add esp, 00000020
:0042D7A5 50                             push eax
:0042D7A6 51                             push ecx
```

* Reference To: `MSVBVM60.__vbaStrCat`, `Ord:0000h`

```
:0042D7A7 FF1550104000                  Call dword ptr [00401050]
```

====> 依次把所得结果连接起来！

```
:0042D7AD 8BD0                           mov edx, eax
```

最后保存的结果 `====>EDX=EAX=295129513` 这就是注册码了！

```
:0042D7AF 8D4DE0                         lea ecx, dword ptr [ebp-20]
:0042D7B2 FFD3                           call ebx
:0042D7B4 B801000000                     mov eax, 00000001
:0042D7B9 66030540814300                add ax, word ptr [00438140]
:0042D7C0 0F8096000000                   jo 0042D85C
:0042D7C6 E951FeFFFF                     jmp 0042D61C
```

====> 继续循环！共循环用户号位数次！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0042D629(C)`

```
:0042D7CB 8B55E0                         mov edx, dword ptr [ebp-20]
:0042D7CE 8D4DDC                         lea ecx, dword ptr [ebp-24]
```

* Reference To: `MSVBVM60.__vbaStrCopy`, `Ord:0000h`

```
:0042D7D1 FF1580114000                  Call dword ptr [00401180]
:0042D7D7 6846D84200                     push 0042D846
:0042D7DC EB52                           jmp 0042D830
:0042D7DE F645FC04                       test [ebp-04], 04
:0042D7E2 7409                           je 0042D7ED
:0042D7E4 8D4DDC                         lea ecx, dword ptr [ebp-24]
```

* Reference To: `MSVBVM60.__vbaFreeStr`, `Ord:0000h`

```
:0042D7E7 FF15F0114000                  Call dword ptr [004011F0]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0042D7E2(C)`

```
:0042D7ED 8D9538FFFFFF                  lea edx, dword ptr [ebp+FFFFFF38]
:0042D7F3 8D8548FFFFFF                  lea eax, dword ptr [ebp+FFFFFF48]
:0042D7F9 52                             push edx
:0042D7FA 8D8D58FFFFFF                  lea ecx, dword ptr [ebp+FFFFFF58]
:0042D800 50                             push eax
```



```assembly
:0042D801 8D9568FFFFFF           lea edx, dword ptr [ebp+FFFFFF68]  
:0042D807 51                    push ecx  
:0042D808 8D8578FFFFFF          lea eax, dword ptr [ebp+FFFFFF78]  
:0042D80E 52                    push edx  
:0042D80F 8D4D88                lea ecx, dword ptr [ebp-78]  
:0042D812 50                    push eax  
:0042D813 8D5598                lea edx, dword ptr [ebp-68]  
:0042D816 51                    push ecx  
:0042D817 8D45A8                lea eax, dword ptr [ebp-58]  
:0042D81A 52                    push edx  
:0042D81B 8D4DB8                lea ecx, dword ptr [ebp-48]  
:0042D81E 50                    push eax  
:0042D81F 8D55C8                lea edx, dword ptr [ebp-38]  
:0042D822 51                    push ecx  
:0042D823 52                    push edx  
:0042D824 6A0A                  push 0000000A
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h  
                                  |  
:0042D826 FF1530104000          Call dword ptr [00401030]  
:0042D82C 83C42C                add esp, 0000002C  
:0042D82F C3                   ret
```

### 算法总结

一、取用户号，将前 3 位换到最后。如 `894112337` -> `112337894`

二、依次求循环次数的立方值

三、依次取变化后的用户号的数字，求其立方值

四、两者依次相加

五、相加之和 依次 与 `A` 求模！每次求模的结果连接起来就是注册码了！

### 完美爆破

`0042BC85`
```assembly
1BF6                  sbb esi, esi  
```
改为：
```assembly
33F6                  xor esi, esi  
```

### KeyMake 之{68th}内存注册机

`中断地址`：`0042BC6C`  
`中断次数`：`1`  
`第一字节`：`8B`  
`指令长度`：`2`

`内存方式`：`EAX`  

`宽字符串`

### 注册信息保存

`C:\WINDOWS\SYSTEM` 下的 `vfnlubfvt.sys` 文件

### 整理

`用户号`：`894112337`  
`注册码`：`295129513`

```text
        ,     _/   
      /| \_.-~/           \_     ,        青春都一饷  
      ( /~   /              ~-._ |
      `\\   /_/               ~\\ )          忍把浮名   
      _-~~~-. )  )\_/;;,.          \_  //'  
      /'\_,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.          换了破解轻狂  
      `~ _( ,_..--\ (      ,;'' /    ~--   /._`\  
      /~~//'   /' `~\           ) /--.._ , )_  `~  
      "  `~"  "       `"       /~'`\   `\\~~\   
                               "      "   "~'  ""
```

Cracked By 巢水工作坊——fly [OCN][FCG]

2003-04-23 2:00

```pascal
var  
 L, i, Key1, Key: Integer;  
 S, Temp, Temp1, Temp2, RegCode: String;

begin  
 S := Edit1.Text;  
 L := Length(S);  
 Temp := Copy(S, 1, 3);  
 Temp1 := Copy(S, 4, L);  
 Temp2 := Temp1 + Temp;  
 S := Temp2;  
 SetLength(RegCode, L);  
 RegCode := '';  
 for i := 1 to L do  
 begin  
 Key1 := StrToInt(s[i]);  
 Key := i * i * i + Key1 * Key1 * Key1;  
 Key := Key mod $0a;  
 RegCode := RegCode + IntToStr(Key);  
 end;  
 Edit2.Text := RegCode;  
end; 
```

`4dd466d0e1e0cc123afbe504da2462ca`
