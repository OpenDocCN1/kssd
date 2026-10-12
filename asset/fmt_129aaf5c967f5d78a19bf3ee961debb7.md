
# SN Calculator v2.2

> 原文：[`www.pediy.com/kssd/pediy03/forum676.htm`](https://www.pediy.com/kssd/pediy03/forum676.htm)

## 概述

初学者看看。

- **软件名称**：SN Calculator v2.2（功能强大的科学计算器，可惜是 E 文的）
- **注册方式**：Name&Key
- **下载地址**：`http://61.134.4.193:8080/appsoftware/calculator/sncalc.exe`

## 破解背景

我是一个新手~在论坛扮演了几次 1212~也将看雪主页里的教程看了几次~跟着就是实践啦~于是先拿我手头上用着的一些软件来开刀啦~~虽然有的都已经有注册码啦~~现在我已破了 `Boxman 3.0`~~~`Win-eXpose-Registry` for Windows 95(同 `regmon` 功能一样~~有程序防 `regmon` 的用它就行了)~~~`Readbook 1.42` 和其它一些小软件(我正在破 `Nktools` 手机工具箱~~请高手们提点一下)~~这些都是看着教程去破的~~但我在破解过程中都有一些心得~~很想拿出来同大家分享~~也想高手指点一二~~~如果我有时间就将破解过程写出来吧~~~在破解时我最头痛的就是程序用我输入的注册码经过一大堆运算后再同内存里的数据比较~~~对于我这种新手来说很难找到比较的地方~~就算找到了也看不到真的注册码~~写注册机就更别说啦~~~所以我有了个蠢想法~~找一个功能强大的计算器来~~将注册码算一算~~于是上网就找了一大堆~~上面这个就是其一~~安装后一用~~呵呵~~E 文~~~好难明~~删~~~~~但删之前当然是先搞它一下啦~~~哈哈~~进去一看~~~太简单啦~~~所以就写一写它咯~~~~~~~~~~~~~~~

## 调试过程

开始：

```
Trw2000                           |
Ctrl+N                            |
bpx hmemcpy                       |
输入 Name:Sam Von & Key:67676767  |------看得懂吧？？？？？
Trw2000 弹出                      |
bd                                |
pmodule                           -------|
```

好啦我们来到 `SN Calculator` 的代码中啦~~~第一次调试当然是一真按 `F10` 啦~~因为这个软件不会有出错信息~~所以我没有用 `F12`~~~一直按下去~~~没太多 `call` 的地方~~一路都是 `ret`~~当程序来到 `00488F95` 的时候~~立即留意啦~~看看 `EAX` 的值等于 `7`~~~呵呵~~~是我注册名“`Sam Von`"的个数~~~继续~~~`00488FA2` 是我输入的假注册码的个数~~下面就是比较核心啦~~看我的注解吧~~~

```asm
:00488F8D 8B45FC                  mov eax, dword ptr [ebp-04]
:00488F90 E8C7ADF7FF              call 00403D5C
; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :00488F25(C)
:00488F95 83F804                  cmp eax, 00000004   <-----注册名要大于 4
:00488F98 7C0D                    jl 00488FA7
:00488F9A 8B45F8                  mov eax, dword ptr [ebp-08]
:00488F9D E8BAADF7FF              call 00403D5C
:00488FA2 83F804                  cmp eax, 00000004   <-----注册码要大于 4
:00488FA5 7D0C                    jge 00488FB3
; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :00488F98(C)
:00488FA7 8BC3                    mov eax, ebx
:00488FA9 E80A06FCFF              call 004495B8
:00488FAE E9BB000000              jmp 0048906E
; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :00488FA5(C)
:00488FB3 8B45FC                  mov eax, dword ptr [ebp-04]    <----注册名的首地址
:00488FB6 0FB64001                movzx eax, byte ptr [eax+01]   <---注册名的第二个字符 a
:00488FBA 8B55F8                  mov edx, dword ptr [ebp-08]    <----注册码的首地址
:00488FBD 0FB65202                movzx edx, byte ptr [edx+02]   <---注册码的第三个字符 6
:00488FC1 33C2                    xor eax, edx     <-------61 xor 36=57=>eax
:00488FC3 B90A000000              mov ecx, 0000000A
:00488FC8 99                      cdq
:00488FC9 F7F9                    idiv ecx         <-------57 除以 A 余数 7 放入 edx
:00488FCB 83FA03                  cmp edx, 00000003
:00488FCE 0F859A000000            jne 0048906E     <-------不等于 3 就跳
:00488FD4 68E9030000              push 000003E9
; Reference To: kernel32.Sleep, Ord:0000h
:00488FD9 E862D9F7FF              Call 00406940
:00488FDE 8B45FC                  mov eax, dword ptr [ebp-04]
:00488FE1 0FB64002                movzx eax, byte ptr [eax+02]   <---注册名的第三个字符 m
:00488FE5 8B55F8                  mov edx, dword ptr [ebp-08]
:00488FE8 0FB65201                movzx edx, byte ptr [edx+01]   <---注册码的第二个字符 7
:00488FEC 33C2                    xor eax, edx     <-------6D xor 37=5A=>eax
:00488FEE B90A000000              mov ecx, 0000000A
:00488FF3 99                      cdq
:00488FF4 F7F9                    idiv ecx         <-------5A 除以 A 余数 0 放入 edx
:00488FF6 83FA05                  cmp edx, 00000005
:00488FF9 7573                    jne 0048906E     <-------不等于 5 就跳
:00488FFB 68E7030000              push 000003E7
```

### 分析结论

好啦~~首先我将 `edx` 的值在比较前改为 `3` 和 `5`~~~~程序运行下去就 `Say Ok` 啦~~~~~就是说程序只将我的注册名和注册码的第二三位运算比较~~~但两者都要 `4` 位以上~~~~那太简单啦嘛~~~还好程序的运算过程不复杂~~注册机都不用写啦~~就用它本身来算一算吧~~~计算器软件~~~~真搞笑~~~程序要求两个数相 `xor` 后的值除以 `A` 的余数为 `3` 和 `5`~~~上面的是 `57` 和 `5A`~~~我就改成 `53` 和 `55` 吧~~~分别和 `61` 和 `6D` 相 `xor` 得到 `8` 和 `2`~~~就是说注册码的第二三位是 `82` 就行啦~~其它的位就补上任意数吧~~~呵呵~~~以上是我的蠢办法~~我是新手嘛~~~~~请高手指正啦~~~~多谢~~~~另外注册成功后要退出程序再进入~~不然在 `About` 里仍然是没注册的~~~

我写得不好~~~~~~大家多多包涵~~~~~~~~

### 补充说明

上面的分析基本正确：

先设姓名为 `NAME`、注册码为 `CODE`，姓名与注册码要求均大于 `4` 位。

- `NAME[2]` 的值 `XOR` `CODE[3]` 的值 再除以 `0A` 的余数要等于 `3`
- `NAME[3]` 的值 `XOR` `CODE[2]` 的值 再除以 `0A` 的余数要等于 `5`
- 注册码格式：`*CODE[2]+CODE[3]*`（`*` 为任意数字，长度任意）

一些地方不够明确，例：

```asm
:00488FC9 F7F9                    idiv ecx         <-------57 除以 A 余数 7 放入 edx
```

（这是将 `EAX` 的值除以 `ECX` 的值，`EAX` 等于商，`EDX` 为余数）

```asm
:00488FCB 83FA03                  cmp edx, 00000003
```

（比较余数是否为 `3`）

是否这样说更能明白点，总得来说是不错的。但如果能写得更详细、更明白一点就是一篇好教材了。

再接再历... `4dd466d0e1e0cc123afbe504da2462ca`
