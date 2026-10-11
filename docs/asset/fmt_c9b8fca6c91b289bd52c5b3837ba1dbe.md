
# 邮件先知 v2.5.2.50

> 原文：[`www.pediy.com/kssd/pediy05/pediy50055.htm`](https://www.pediy.com/kssd/pediy05/pediy50055.htm)

===================Open Cracking Group========================

```
=
=        邮件先知 v2.5.2.50 注册算法分析
=
=                            ssljx/OCG
=     http://www.newclw.com/lllufh/cgi-bin/leoboard.cgi
=
===================Open Cracking Group========================
```

\* Possible StringData Ref from Code Obj -> `"SOFTWARE\\Microsoft\\tpcip\\CurrentVersion"`

```
:004C25CE 6848294C00               push 004C2948
:004C25D3 6802000080               push 80000002
* Reference To: advapi32.RegCreateKeyExA, Ord:0000h
:004C25D8 E85F4DF4FF              Call 0040733C
:004C25DD 85C0                    test eax, eax
:004C25DF 7548                    jne 004C2629
:004C25E1 8D45FC                 lea eax, dword ptr [ebp-04]
:004C25E4 50                      push eax
* Possible StringData Ref from Code Obj -> "PPPPP"
:004C25E5 B978294C00             mov ecx, 004C2978
* Possible StringData Ref from Code Obj -> "Rotescode"=========>注册表的注册名
:004C25EA BA88294C00             mov edx, 004C2988
:004C25EF 8B45F0                 mov eax, dword ptr [ebp-10]
:004C25F2 E8C182FCFF             call 0048A8B8
:004C25F7 8D45F8                 lea eax, dword ptr [ebp-08]
:004C25FA 50                      push eax
* Possible StringData Ref from Code Obj -> "H012123"
:004C25FB B99C294C00             mov ecx, 004C299C
* Possible StringData Ref from Code Obj -> "RotesNum"==========>注册表的用户编号
:004C2600 BAAC294C00             mov edx, 004C29AC
:004C2605 8B45F0                 mov eax, dword ptr [ebp-10]
:004C2608 E8AB82FCFF             call 0048A8B8
:004C260D 8D45F4                 lea eax, dword ptr [ebp-0C]
:004C2610 50                      push eax
:004C2611 33C9                    xor ecx, ecx
* Possible StringData Ref from Code Obj -> "Object"===========>注册表的注册码
:004C2613 BAC0294C00             mov edx, 004C29C0
:004C2618 8B45F0                 mov eax, dword ptr [ebp-10]
:004C261B E89882FCFF             call 0048A8B8
:004C2620 8B45F0                 mov eax, dword ptr [ebp-10]
:004C2623 50                      push eax
* Reference To: advapi32.RegCloseKey, Ord:0000h
:004C2624 E80B4DF4FF             Call 00407334
```

////////////////////////////////读去注册表过程结束\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C25DF(C)
|
:004C2629 8B45F8                 mov eax, dword ptr [ebp-08]
* Possible StringData Ref from Code Obj -> "AHF000186"
:004C262C BAD0294C00             mov edx, 004C29D0
:004C2631 E83E28F4FF             call 00404E74========>用户编号与'AHF000186'比较
:004C2636 7516                    jne 004C264E=========>一定要不等,转向!!
:004C2638 8D55D8                 lea edx, dword ptr [ebp-28]
:004C263B 8B45F8                 mov eax, dword ptr [ebp-08]
:004C263E E8196AF4FF             call 0040905C
:004C2643 8B55D8                 mov edx, dword ptr [ebp-28]
:004C2646 8D45F8                 lea eax, dword ptr [ebp-08]
:004C2649 E8C224F4FF             call 00404B10
```

\\\\\\\\\\\\\用户编号与'AHF000186'相等就把'AHF000186'转为小写字母,这样到后面判断就会出错\\\\\\\\\\\\\\\

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C2636(C)
|
:004C264E A1D49F4C00             mov eax, dword ptr [004C9FD4]
:004C2653 833800                 cmp dword ptr [eax], 00000000
:004C2656 7516                    jne 004C266E
:004C2658 8BCF                    mov ecx, edi
:004C265A B201                    mov dl, 01
* Possible StringData Ref from Code Obj -> "癿 D"
:004C265C A1C0694B00             mov eax, dword ptr [004B69C0]
:004C2661 E87A5DF9FF             call 004583E0
:004C2666 8B15D49F4C00           mov edx, dword ptr [004C9FD4]
:004C266C 8902                    mov dword ptr [edx], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C2656(C)
|
:004C266E 8D4DD4                 lea ecx, dword ptr [ebp-2C]
:004C2671 8B55F8                 mov edx, dword ptr [ebp-08]
:004C2674 8B45FC                 mov eax, dword ptr [ebp-04]
:004C2677 E80074FCFF             call 00489A7C
:004C267C 8B45D4                 mov eax, dword ptr [ebp-2C]=>根据注册名计算出来的注册码,怎么计算注册码都是零
:004C267F 8B55F4                 mov edx, dword ptr [ebp-0C]=>输入的注册码,只要输入 0 就行
:004C2682 E8ED27F4FF             call 00404E74===============>比较
:004C2687 740C                    je 004C2695=================>要等,转到下面比较
:004C2689 C605BC9E4C0001          mov byte ptr [004C9EBC], 01==>不等,给标志赋值,1 表示失败
:004C2690 E9E6000000             jmp 004C277B
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C2687(C)
|
:004C2695 C605BC9E4C0000          mov byte ptr [004C9EBC], 00==>注册码相同,标志为赋值 0,0 表示成功
:004C269C 837DF800                cmp dword ptr [ebp-08], 00000000===>用户编号是空的,直接结束比较
:004C26A0 0F84D5000000            je 004C277B
```

//////////用户编号不是空值,进行下面比较\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\

```
:004C26A6 8B45F8                 mov eax, dword ptr [ebp-08]
:004C26A9 8A00                    mov al, byte ptr [eax]====>用户编号第一位(N1)
:004C26AB 04BF                    add al, BF===============>这里控制要大于'A'
:004C26AD 2C1A                    sub al, 1A===============>这里控制要小于'Z'
:004C26AF 731E                    jnb 004C26CF=============>不在 A-Z 范围就出错
```

//////////////用户编号第一位要在 A-Z 范围,$1A-$BF=$5B 注意这是字节运算,下面简单说说这算法\\\\\\\\\\\

我们先了解 `jnb` 的转向条件是 `CF=0`,那么 `CF` 受什么影响,`CF==>进位标志,在进行字/字节运算产生进位或借位是置 1,这是我以前的书上讲的,不知现在有没改变..`

举例：

当 `AL=$40` 时,`CF=0`

```
AL\=$40+$BF=$FF CF=0(没变)
AL\=$FF-$1A=$E6 CF=0(没变)
```

所以 `jnb 004C26CF` 合乎转向条件,跳转出错!!

当 `AL=$5B,CF=0`

```
AL\=$5B+$BF=$11A CF=1(改变)
AL\=$11A-$1A=100 CF=0(改变)
```

这样 `jnb 004C26CF` 合乎转向条件,跳转出错!!

当 `AL=$41, CF=0`

```
AL\=$41+$BF=101  CF=1(改变)
AL\=$101-$1A=$E6 CF=1(没变)
```

这样 `jnb 004C26CF` 不合乎转向条件,继续比较!!

```
:004C26B1 8B45F8                 mov eax, dword ptr [ebp-08]
:004C26B4 8A4001                  mov al, byte ptr [eax+01]
:004C26B7 04BF                    add al, BF
:004C26B9 2C0C                    sub al, 0C
:004C26BB 7312                    jnb 004C26CF=======>第二位用户编号范围 A-L
:004C26BD 8B45F8                 mov eax, dword ptr [ebp-08]
:004C26C0 8A4002                  mov al, byte ptr [eax+02]
:004C26C3 04BF                    add al, BF
:004C26C5 2C1A                    sub al, 1A
:004C26C7 722E                    jb 004C26F7=======>第三位用户编码在 A-Z 就转向,不再继续
:004C26C9 04FA                    add al, FA
:004C26CB 2C06                    sub al, 06
:004C26CD 7228                    jb 004C26F7======>第三位用户编码在 a-f 就转向
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
|:004C26AF(C), :004C26BB(C)
|
:004C26CF C605BC9E4C0001          mov byte ptr [004C9EBC], 01==>上面不通过设标志为 1
:004C26D6 B804000000             mov eax, 00000004
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004C26F5(C)
|
```
