
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



# 反汇编代码分析

## 逻辑判断与循环

```
:004C26DB 8B55F8               mov edx, dword ptr [ebp-08]
:004C26DE 8A5402FF             mov dl, byte ptr [edx+eax-01]
:004C26E2 80C2D0               add dl, D0
:004C26E5 80EA0A               sub dl, 0A
:004C26E8 7207                 jb 004C26F1
:004C26EA C605BC9E4C0001       mov byte ptr [004C9EBC], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C26E8(C)
|
:004C26F1 40                   inc eax
:004C26F2 83F80A               cmp eax, 0000000A
:004C26F5 75E4                 jne 004C26DB
```

`///////////上面是当用户编号前面三位不通过就进行比较后面是否全是数字,但这比较没用的\\\\\\`

## 标志比较与字符串验证

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004C26C7(C), :004C26CD(C)
|
:004C26F7 803DBC9E4C0000       cmp byte ptr [004C9EBC], 00======>比较标志是否 0
:004C26FE 757B                 jne 004C277B=====================>不为 0,OVER!!!!
:004C2700 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C2705 8B00                 mov eax, dword ptr [eax]
:004C2707 8B8020030000         mov eax, dword ptr [eax+00000320]
:004C270D 8B55FC               mov edx, dword ptr [ebp-04]
:004C2710 E8E7D3F7FF           call 0043FAFC
:004C2715 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C271A 8B00                 mov eax, dword ptr [eax]
:004C271C 8B8028030000         mov eax, dword ptr [eax+00000328]
:004C2722 8B55F8               mov edx, dword ptr [ebp-08]
:004C2725 E8D2D3F7FF           call 0043FAFC
:004C272A A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C272F 8B00                 mov eax, dword ptr [eax]
:004C2731 8B8024030000         mov eax, dword ptr [eax+00000324]
:004C2737 8B55F4               mov edx, dword ptr [ebp-0C]
:004C273A E8BDD3F7FF           call 0043FAFC
:004C273F A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C2744 8B00                 mov eax, dword ptr [eax]
:004C2746 8B8020030000         mov eax, dword ptr [eax+00000320]
:004C274C 33D2                 xor edx, edx
:004C274E 8B08                 mov ecx, dword ptr [eax]
:004C2750 FF5164               call [ecx+64]
:004C2753 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C2758 8B00                 mov eax, dword ptr [eax]
:004C275A 8B8028030000         mov eax, dword ptr [eax+00000328]
:004C2760 33D2                 xor edx, edx
:004C2762 8B08                 mov ecx, dword ptr [eax]
:004C2764 FF5164               call [ecx+64]
:004C2767 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C276C 8B00                 mov eax, dword ptr [eax]
:004C276E 8B8024030000         mov eax, dword ptr [eax+00000324]
:004C2774 33D2                 xor edx, edx
:004C2776 8B08                 mov ecx, dword ptr [eax]
:004C2778 FF5164               call [ecx+64]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004C2690(U), :004C26A0(C), :004C26FE(C)
|
:004C277B 8D55C8               lea edx, dword ptr [ebp-38]
:004C277E A1F4A24C00           mov eax, dword ptr [004CA2F4]
:004C2783 8B00                 mov eax, dword ptr [eax]
:004C2785 E8DED8F9FF           call 00460068
:004C278A 8B45C8               mov eax, dword ptr [ebp-38]
:004C278D 8D55CC               lea edx, dword ptr [ebp-34]
:004C2790 E8FB71FCFF           call 00489990
:004C2795 8B4DCC               mov ecx, dword ptr [ebp-34]
:004C2798 8D45D0               lea eax, dword ptr [ebp-30]
```

### 字符串引用

* Possible StringData Ref from Code Obj -> "当前版本号："

```
:004C279B BAE4294C00           mov edx, 004C29E4
:004C27A0 E8D725F4FF           call 00404D7C
:004C27A5 8B55D0               mov edx, dword ptr [ebp-30]
:004C27A8 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C27AD 8B00                 mov eax, dword ptr [eax]
:004C27AF 8B801C030000         mov eax, dword ptr [eax+0000031C]
:004C27B5 E842D3F7FF           call 0043FAFC
:004C27BA 33F6                 xor esi, esi
:004C27BC 803DBC9E4C0000       cmp byte ptr [004C9EBC], 00=======>再次比较标志
:004C27C3 743A                 je 004C27FF=======================>这里要跳!!!
:004C27C5 A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C27CA 8B00                 mov eax, dword ptr [eax]
:004C27CC 8B8018030000         mov eax, dword ptr [eax+00000318]
```

### 未注册试用版提示

* Possible StringData Ref from Code Obj -> "本版为未注册试用版，部分功能只能在注册后才能使" -> "用！"

```
:004C27D2 BAFC294C00           mov edx, 004C29FC
:004C27D7 E820D3F7FF           call 0043FAFC
:004C27DC 8D55C4               lea edx, dword ptr [ebp-3C]
:004C27DF 8BC7                 mov eax, edi
:004C27E1 E8E6D2F7FF           call 0043FACC
:004C27E6 8D45C4               lea eax, dword ptr [ebp-3C]
```

* Possible StringData Ref from Code Obj -> "[未注册功能限制版]"

```
:004C27E9 BA382A4C00           mov edx, 004C2A38
:004C27EE E84525F4FF           call 00404D38
:004C27F3 8B55C4               mov edx, dword ptr [ebp-3C]
:004C27F6 8BC7                 mov eax, edi
:004C27F8 E8FFD2F7FF           call 0043FAFC
:004C27FD EB55                 jmp 004C2854
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `|:004C27C3(C)`

## 注册成功提示

```
:004C27FF A1D49F4C00           mov eax, dword ptr [004C9FD4]
:004C2804 8B00                 mov eax, dword ptr [eax]
:004C2806 8B8018030000         mov eax, dword ptr [eax+00000318]
```

* Possible StringData Ref from Code Obj -> "恭喜您成为完全版的荣誉注册用户！"

```
:004C280C BA542A4C00           mov edx, 004C2A54
:004C2811 E8E6D2F7FF           call 0043FAFC
:004C2816 8D55BC               lea edx, dword ptr [ebp-44]
:004C2819 8BC7                 mov eax, edi
:004C281B E8ACD2F7FF           call 0043FACC
:004C2820 FF75BC               push [ebp-44]
```

* Possible StringData Ref from Code Obj -> " [荣誉注册用户："

```
:004C2823 68802A4C00           push 004C2A80
:004C2828 FF75FC               push [ebp-04]
:004C282B 689C2A4C00           push 004C2A9C
:004C2830 8D45C0               lea eax, dword ptr [ebp-40]
:004C2833 BA04000000           mov edx, 00000004
:004C2838 E8B325F4FF           call 00404DF0
:004C283D 8B55C0               mov edx, dword ptr [ebp-40]
:004C2840 8BC7                 mov eax, edi
:004C2842 E8B5D2F7FF           call 0043FAFC
:004C2847 33DB                 xor ebx, ebx
:004C2849 8B8734030000         mov eax, dword ptr [edi+00000334]
:004C284F E8A0D7F7FF           call 0043FFF4
```

## 跳转与最终检查

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `|:004C27FD(U)`

```
:004C2854 84DB                 test bl, bl
:004C2856 7413                 je 004C286B
:004C2858 6A00                 push 00000000
```

* Possible StringData Ref from Code Obj -> "提示！"

```
:004C285A 68A02A4C00           push 004C2AA0
```

* Possible StringData Ref from Code Obj -> "本软件的试用版只能使用 15 次! 如果您对试用结果满意，可以向我们注册。 注册信息请访问“帮助”菜? サ摹白⒉帷弊酉?注册费用为 12 元。"

```
:004C285F 68A82A4C00           push 004C2AA8
:004C2864 6A00                 push 00000000
```



* Reference To: `user32.MessageBoxA`, Ord:`0000h`

`004C2866 E8B153F4FF Call 00407C1C`

`END`

总结:

1.注册名是任意

2.注册码永远是 0

3.用户编号有两种情况

A.用户编号可以空值

B.用户编号第一位在:`A-Z` 范围

用户编号第二位在:`A-L` 范围

用户编号第三位在:`A-L` 或 `a-f` 范围

还有用户编号一定不能为`'AHF000186'`

注册表:

`"SOFTWARE\Microsoft\tpcip\CurrentVersion"`

怎么还要写注册机..不用吧...哈哈...

`===================Open Cracking Group========================`

`=`

`= 邮件先知 v2.5.2.50 注册算法分析`

`=`

`= ssljx/OCG`

`= http://www.newclw.com/lllufh/cgi-bin/leoboard.cgi`

`=`

`===================Open Cracking Group======================== 4dd466d0e1e0cc123afbe504da2462ca`
