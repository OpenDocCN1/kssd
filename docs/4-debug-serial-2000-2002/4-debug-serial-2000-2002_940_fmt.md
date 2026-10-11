
# 美萍反黄专家 版本 3.2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue278.htm`](https://www.pediy.com/kssd/pediy04/kanxue278.htm)

美萍反黄专家 版本 3.2 破解实录

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

破解时间：2001-12-8

软件简介：该软件对抗 `TRW`、`SOFTICE`、`WD32SM`、`CRACKCODE` 等破解工具！

破解形式：注册码

破解工具：`trw`(调试工具）、`pw32dasm`（反汇编）

破解作者：`ybyb`

## 第一部分，破解对抗

＝＝＝＝＝＝＝＝＝＝＝＝

### 一、用 FI 侦测，发现它用 ASPACK 壳,于是用 caspr110 解压.

### 二、解除发现破解工具立即关机问题！

先运行 `shield.exe`，它会自动检测你的硬盘是否存在以下文件，如果存在就立即重新启动电脑！黑名单如下：

内存中的：

1. `softice`
2. `trw`

当然目录下的：

1. `CRACKCODE`
2. `WDAS`

解决办法：

\* Referenced by a CALL at Addresses:

| :004740BB , :00474139 , :00476E3C , :00477647 , :0047853C \\
| :--- |
| 所有发现黑名单的地方都调用这个关机函数， |

\* Reference To: `user32.ExitWindowsEx`, Ord:0000h

```
:00407000 FF25F4164800            Jmp dword ptr [004816F4]==> NOP 掉
:00407006 8BC0                    mov eax, eax
```

好了不再关机了，大家可以追注册码了。

\=============================================================

## 第二部分，找注册码

### 1. `w32dasm` 反汇编,串式参考"未注册版本只能使用 30 天，现在还剩" 具体是：

\*\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0047735B(C)
| --- |

```
:00477384 A1D4F24700            mov eax, dword ptr [0047F2D4]<====== eax ,注册标志位
:00477389 3B05540C4800          cmp eax, dword ptr [00480C54]<======这个不可忽略 00480c54 为 B
:0047738F 0F8404010000          je 00477499<====关键跳转
:00477395 E85ABDFFFF            call 004730F4
:0047739A A3E4F24700            mov dword ptr [0047F2E4], eax

* Possible StringData Ref from Code Obj ->"未注册版本只能使用 30 天，现在还剩"

:0047739F 68287A4700            push 00477A28
:004773A4 8D55A8                lea edx, dword ptr [ebp-58]
:004773A7 A1E8F24700            mov eax, dword ptr [0047F2E8]
:004773AC 2B05E4F24700          sub eax, dword ptr [0047F2E4]
:004773B2 40                    inc eax
:004773B3 E85415F9FF            call 0040890C
:004773B8 FF75A8                push [ebp-58]
:004773BB 68547A4700            push 00477A54
:004773C0 8D45AC                lea eax, dword ptr [ebp-54]
:004773C3 BA03000000            mov edx, 00000003
:004773C8 E87BCBF8FF            call 00403F48
:004773CD 8B55AC                mov edx, dword ptr [ebp-54]
:004773D0 8B8380030000          mov eax, dword ptr [ebx+00000380]
:004773D6 E8CD87FBFF            call 0042FBA8
:004773DB 8B15E4F24700          mov edx, dword ptr [0047F2E4]
:004773E1 8B8378030000          mov eax, dword ptr [ebx+00000378]
:004773E7 E85442FEFF            call 0045B640
:004773EC A1E8F24700            mov eax, dword ptr [0047F2E8]
:004773F1 83E80F                sub eax, 0000000F
:004773F4 3B05E4F24700          cmp eax, dword ptr [0047F2E4]
:004773FA 7D50                  jge 0047744C
:004773FC 6A40                  push 00000040

* Possible StringData Ref from Code Obj ->"注册信息"

:004773FE 68587A4700            push 00477A58

* Possible StringData Ref from Code Obj ->"软件试用期还剩"

:00477403 686C7A4700            push 00477A6C
:00477408 8D55A0                lea edx, dword ptr [ebp-60]
:0047740B A1E8F24700            mov eax, dword ptr [0047F2E8]
:00477410 40                    inc eax
:00477411 2B05E4F24700          sub eax, dword ptr [0047F2E4]
:00477417 E8F014F9FF            call 0040890C
:0047741C FF75A0                push [ebp-60]
:0047741F 68547A4700            push 00477A54

* Possible StringData Ref from Code Obj ->",请赶快向美萍公司注册(0371-8749676)"

:00477424 68847A4700            push 00477A84
:00477429 8D45A4                lea eax, dword ptr [ebp-5C]
:0047742C BA04000000            mov edx, 00000004
:00477431 E812CBF8FF            call 00403F48
:00477436 8B45A4                mov eax, dword ptr [ebp-5C]
:00477439 E80ECCF8FF            call 0040404C
:0047743E 50                    push eax
:0047743F 8BC3                  mov eax, ebx
:00477441 E80AE8FBFF            call 00435C50
:00477446 50                    push eax
```

### 2. `w32dasm` 查找菜单,从头查找 0047F2D4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| :0047705C(C)
| --- |

```
:0047706B 8B153C0A4800          mov edx, dword ptr [00480A3C]
:00477071 A14C0A4800            mov eax, dword ptr [00480A4C]
:00477076 E8F9D0F8FF            call 00404174<====d edx 见一长注册码,我的是 162778260300854525709 比方我输入的是 62778
:0047707B 85C0                  test eax, eax
:0047707D 7E24                  jle 004770A3
:0047707F A14C0A4800            mov eax, dword ptr [00480A4C]
:00477084 E8FFCDF8FF            call 00403E88
:00477089 83F805                cmp eax, 00000005
:0047708C 7515                  jne 004770A3
:0047708E 8B153C0A4800          mov edx, dword ptr [00480A3C]
:00477094 A14C0A4800            mov eax, dword ptr [00480A4C]
:00477099 E8D6D0F8FF            call 00404174 <=====计算你输入注册码的开始位数
:0047709E A3D4F24700            mov dword ptr [0047F2D4], eax <===EAX 为 2.而比较的是 B 所以我们知道我们该从第几位输了吧。
```

呵呵~~~~~~~~~可是这样做了仍然不对，为什么？？？记得在 3.0 以前我们知道黑名单里有两个 `TRW`,后一个参与注册码运算，前一个被发现后关机。可是这个版本是发现 `TRW` 后不仅关机还来干扰注册码的运算。所以我门该把第一个 `TRW` 改为“ttt”（为任意三个字母)

再来看 00477084 处那个长码，这次为 11244327843548534316722 从 B 位开始取，为 35485

所以:注册名 `ybyb`

注册码 35485

写不有错的地方请大家指正.

(后话虽然快考试了,可是手痒的很啊!!!于是下载了美萍网管大师 7.2,美萍安全卫士 8.5 来练习一下 crackYY\[CNCG\] 解的网管大师也是从 B 位开始的. 安全卫士和网管大师大同小异,前者在 00475a13 处 ? ebx 就见你的注册码了. )

2001.12.08 4dd466d0e1e0cc123afbe504da2462ca
