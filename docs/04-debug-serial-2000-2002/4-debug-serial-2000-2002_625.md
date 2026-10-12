
# 光驱精灵.NET

> 原文：[`www.pediy.com/kssd/pediy03/forum1134.htm`](https://www.pediy.com/kssd/pediy03/forum1134.htm)

- **作者**：`Crack007[BCG]`
- **主页**：`http://www.crack007.com` (^-^)
- **破解日期**：`2001-7-4`
- **破解工具**：`TRW2000 1.23`、`W32dasm`
- **难度**：易
- **软件下载地址**：`http://www.csdn.net/soft/openfile.asp?kind=1&id=11092` 或 `http://yuxuguang.at.china.com/software/cnpu/CDWizard.zip` （124K）

## 软件简介

光驱作为一种计算机常用外设是极其容易损坏的。对于普通光驱当读盘能力尚可，但是控制面板中的弹出键失灵时，若想使用光驱是一件很头疼的事。`光驱精灵`是解决这种难题的优秀软件之一。`光驱精灵`是一款小巧的光驱控制软件，`光驱精灵.NET`是其最新的版本。它能使用热键自由的控制光驱的打开和关闭，相对于同类软件，`光驱精灵.NET`短小精悍，功能强大，勿需任何动态链接库的支持，从内核真正的支持多光驱，并且支持虚拟光驱及特殊的刻录光驱等；它还拥有优化光驱、关机时检测光驱、隐藏光驱以及热键控制音量等功能，用户需要做的仅仅是简单的设置。

**软件注册费**：`10.00￥/每注册用户`

## 过程

```
:00406FE0 6AFF                  push FFFFFFFF
:00406FE2 68D8454200           push 004245D8
:00406FE7 64A100000000         mov eax, dword ptr fs:[00000000]
:00406FED 50                    push eax
:00406FEE 64892500000000       mov dword ptr fs:[00000000], esp
:00406FF5 81EC4C040000         sub esp, 0000044C
:00406FFB A128EB4200           mov eax, dword ptr [0042EB28]
:00407000 53                    push ebx
:00407001 55                    push ebp
:00407002 56                    push esi
:00407003 57                    push edi
:00407004 8BF1                  mov esi, ecx
:00407006 89442410             mov dword ptr [esp+10], eax
:0040700A 6A01                  push 00000001
:0040700C C784246804000000000000 mov dword ptr [esp+00000468], 00000000
:00407017 E8D7530100           call 0041C3F3
:0040701C 51                    push ecx
:0040701D 8D6E5C               lea ebp, dword ptr [esi+5C]
:00407020 8BCC                 mov ecx, esp
:00407022 89642418             mov dword ptr [esp+18], esp
:00407026 55                    push ebp
:00407027 E8366F0100           call 0041DF62
:0040702C 8D4C2418             lea ecx, dword ptr [esp+18]
:00407030 51                    push ecx
:00407031 8BCE                 mov ecx, esi
:00407033 E868020000           call 004072A0
:00407038 50                    push eax
:00407039 8D4C2414             lea ecx, dword ptr [esp+14]
:0040703D C684246804000001     mov byte ptr [esp+00000468], 01
:00407045 E84C720100           call 0041E296
:0040704A 8D4C2414             lea ecx, dword ptr [esp+14]
:0040704E C684246404000000     mov byte ptr [esp+00000464], 00
:00407056 E892710100           call 0041E1ED
:0040705B 8B4500               mov eax, dword ptr [ebp+00]
:0040705E 6880014300           push 00430180
:00407063 50                    push eax
:00407064 E840260000           call 004096A9
:00407069 83C408               add esp, 00000008
:0040706C 85C0                 test eax, eax
:0040706E 7521                 jne 00407091
:00407070 8B4660               mov eax, dword ptr [esi+60]
:00407073 6880014300           push 00430180
:00407078 50                    push eax
:00407079 E82B260000           call 004096A9
:0040707E 83C408               add esp, 00000008
:00407081 85C0                 test eax, eax
:00407083 750C                 jne 00407091

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004070DE(C)
|

:00407085 8BCE                 mov ecx, esi
:00407087 E8F12F0100           call 0041A07D
:0040708C E9E0010000           jmp 00407271

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040706E(C), :00407083(C)
|

:00407091 8B7E60               mov edi, dword ptr [esi+60]    <-----可在这里设断.d edi 显示假的注册码
:00407094 8B442410             mov eax, dword ptr [esp+10]    <-----d eax 显示正确注册码
:00407098 8D5E60               lea ebx, dword ptr [esi+60]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004070B9(C)
|

:0040709B 8A10                 mov dl, byte ptr [eax]
:0040709D 8ACA                 mov cl, dl
:0040709F 3A17                 cmp dl, byte ptr [edi]
:004070A1 751C                 jne 004070BF
:004070A3 84C9                 test cl, cl
:004070A5 7414                 je 004070BB
:004070A7 8A5001               mov dl, byte ptr [eax+01]
:004070AA 8ACA                 mov cl, dl
:004070AC 3A5701               cmp dl, byte ptr [edi+01]
:004070AF 750E                 jne 004070BF
:004070B1 83C002               add eax, 00000002
:004070B4 83C702               add edi, 00000002
:004070B7 84C9                 test cl, cl
:004070B9 75E0                 jne 0040709B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004070A5(C)
|

:004070BB 33C0                 xor eax, eax       异或对 eax 清零
:004070BD EB05                 jmp 004070C4

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004070A1(C), :004070AF(C)
|

:004070BF 1BC0                 sbb eax, eax       eax 清零
:004070C1 83D8FF               sbb eax, FFFFFFFF  设置 eax＝1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004070BD(U)
|

:004070C4 85C0                 test eax, eax      <------软件是否注册成功的标记
:004070C6 743E                 je 00407106        <------1 则死，0 则活
:004070C8 6A04                 push 00000004

* Possible StringData Ref from Data Obj ->"警告"
                     |

:004070CA 68ECE24200           push 0042E2EC

* Possible StringData Ref from Data Obj ->"注册失败，重新注册吗？"
                     |

:004070CF 6870E54200           push 0042E570
:004070D4 8BCE                 mov ecx, esi
:004070D6 E8BC4B0100           call 0041BC97
:004070DB 83F806               cmp eax, 00000006
:004070DE 75A5                 jne 00407085
:004070E0 6880014300           push 00430180
:004070E5 8BCD                 mov ecx, ebp
:004070E7 E8FA710100           call 0041E2E6
:004070EC 6880014300           push 00430180
:004070F1 8BCB                 mov ecx, ebx
:004070F3 E8EE710100           call 0041E2E6
:004070F8 6A00                 push 00000000
:004070FA 8BCE                 mov ecx, esi
:004070FC E8F2520100           call 0041C3F3
:00407101 E96B010000           jmp 00407271

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004070C6(C)
|

:00407106 8D4C2418             lea ecx, dword ptr [esp+18]
:0040710A E80F790100           call 0041EA1E
:0040710F 8D4C2428             lea ecx, dword ptr [esp+28]
:00407113 C684246404000002     mov byte ptr [esp+00000464], 02
:0040711B E8D7740100           call 0041E5F7
:00407120 A128EB4200           mov eax, dword ptr [0042EB28]
:00407125 89442438             mov dword ptr [esp+38], eax
:00407129 6A00                 push 00000000
:0040712B 8D4C243C             lea ecx, dword ptr [esp+3C]
:0040712F C684246804000004     mov byte ptr [esp+00000468], 04
:00407137 C744242CFC814200     mov [esp+2C], 004281FC
:0040713F C744243400000000     mov [esp+34], 00000000
:00407147 C7442438FFFFFFFF     mov [esp+38], FFFFFFFF
:0040714F E892710100           call 0041E2E6
:00407154 8B7D00               mov edi, dword ptr [ebp+00]
:00407157 8B13                 mov edx, dword ptr [ebx]
:00407159 C684246404000005     mov byte ptr [esp+00000464], 05
:00407161 33C0                 xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407186(C)
|
```



```assembly
:00407163 8A0C07          mov cl, byte ptr [edi+eax]
:00407166 80C112          add cl, 12
:00407169 884C043C        mov byte ptr [esp+eax+3C], cl
:0040716D 8A0C02          mov cl, byte ptr [edx+eax]
:00407170 80C112          add cl, 12
:00407173 884C0448        mov byte ptr [esp+eax+48], cl
:00407177 8A4C0208        mov cl, byte ptr [edx+eax+08]
:0040717B 80C112          add cl, 12
:0040717E 884C0450        mov byte ptr [esp+eax+50], cl
:00407182 40              inc eax
:00407183 83F808          cmp eax, 00000008
:00407186 7CDB            jl 00407163
:00407188 8D4C2428        lea ecx, dword ptr [esp+28]
:0040718C C644244400      mov [esp+44], 00
:00407191 51              push ecx
:00407192 6841800000      push 00008041
* Possible StringData Ref from Data Obj ->".\data.cwd"
:00407197 6824E54200      push 0042E524
:0040719C 8D4C2424        lea ecx, dword ptr [esp+24]
:004071A0 C644246400      mov [esp+64], 00
:004071A5 E886790100      call 0041EB30
:004071AA 85C0            test eax, eax
:004071AC 6A00            push 00000000
:004071AE 753C            jne 004071EC
:004071B0 8D542460        lea edx, dword ptr [esp+60]
:004071B4 6800040000      push 00000400
:004071B9 52              push edx
:004071BA 8D4C2434        lea ecx, dword ptr [esp+34]
:004071BE E8D2740100      call 0041E695
:004071C3 6A00            push 00000000
:004071C5 8D442460        lea eax, dword ptr [esp+60]
:004071C9 6A00            push 00000000
:004071CB 50              push eax
:004071CC E87F980100      call 00420A50
:004071D1 8D4C2418        lea ecx, dword ptr [esp+18]
:004071D5 E8717B0100      call 0041ED4B
:004071DA C7442428FC814200 mov [esp+28], 004281FC
:004071E2 C684246404000006 mov byte ptr [esp+00000464], 06
:004071EA EB6B            jmp 00407257

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004071AE(C)

:004071EC 6A10            push 00000010
:004071EE 8D4C2420        lea ecx, dword ptr [esp+20]
:004071F2 E8DB7A0100      call 0041ECD2
:004071F7 8D4C243C        lea ecx, dword ptr [esp+3C]
:004071FB 6A08            push 00000008
:004071FD 51              push ecx
:004071FE 8D4C2420        lea ecx, dword ptr [esp+20]
:00407202 E8807A0100      call 0041EC87
:00407207 6A00            push 00000000
:00407209 6A28            push 00000028
:0040720B 8D4C2420        lea ecx, dword ptr [esp+20]
:0040720F E8BE7A0100      call 0041ECD2
:00407214 8D542448        lea edx, dword ptr [esp+48]
:00407218 6A10            push 00000010
:0040721A 52              push edx
:0040721B 8D4C2420        lea ecx, dword ptr [esp+20]
:0040721F E8637A0100      call 0041EC87
:00407224 6A00            push 00000000
* Possible StringData Ref from Data Obj ->"恭喜"
:00407226 6868E54200      push 0042E568
* Possible StringData Ref from Data Obj ->"注册成功,请重新启动本程序"
:0040722B 684CE54200      push 0042E54C
:00407230 8BCE            mov ecx, esi
:00407232 E8604A0100      call 0041BC97
:00407237 8D4C2418        lea ecx, dword ptr [esp+18]
:0040723B E80B7B0100      call 0041ED4B
:00407240 8BCE            mov ecx, esi
:00407242 E8362E0100      call 0041A07D
:00407247 C7442428FC814200 mov [esp+28], 004281FC
:0040724F C684246404000007 mov byte ptr [esp+00000464], 07
```

跟一阵子之后，我觉得这好像是生成注册码的核心。主要是根据不同的名字（比如 `12121212`）分别取每个字符跟 `dl`、`bl` 进行一些简单的运算，并把结果保存在 `al` 里，并根据不同的 `al` 值（应该是字符的 ASCII 码）的范围再进行一些运算。就可得到一个 `16` 位的注册码。

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004073C8(C)

:00407351 8B07            mov eax, dword ptr [edi] <-----在这设断注意一下 al、bl、dl、ecx、[esp+ecx+14]之间的关系，大家可以可清楚的看到注册码的生成。
:00407353 8A540C14        mov dl, byte ptr [esp+ecx+14]
:00407357 8A1C28          mov bl, byte ptr [eax+ebp]
:0040735A 8D3428          lea esi, dword ptr [eax+ebp]
:0040735D 8AC2            mov al, dl
:0040735F 02C3            add al, bl
:00407361 3C41            cmp al, 41
:00407363 88440C24        mov byte ptr [esp+ecx+24], al    生成第 ecx+1 位注册码（因为 ecx 初始值为 0），不符合范围再进行调整。
                        ^^^ ECX 的值是指现在取得是第几位字符
:00407367 7D06            jge 0040736F
:00407369 8AC2            mov al, dl
:0040736B 0441            add al, 41
:0040736D EB18            jmp 00407387

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407367(C)

:0040736F 3C5A            cmp al, 5A
:00407371 7E0A            jle 0040737D
:00407373 3C61            cmp al, 61
:00407375 7D06            jge 0040737D
:00407377 B05A            mov al, 5A
:00407379 2AC2            sub al, dl
:0040737B EB0A            jmp 00407387

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00407371(C), :00407375(C)

:0040737D 3C7A            cmp al, 7A
:0040737F 7E0A            jle 0040738B
:00407381 8A440C1C        mov al, byte ptr [esp+ecx+1C]
:00407385 0461            add al, 61

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040736D(U), :0040737B(U)

:00407387 88440C24        mov byte ptr [esp+ecx+24], al    <-----生成第 ecx+1 位注册码

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040737F(C)

:0040738B 8A06            mov al, byte ptr [esi]
:0040738D 8A5C0C1C        mov bl, byte ptr [esp+ecx+1C]
:00407391 2AC3            sub al, bl
:00407393 3C41            cmp al, 41
:00407395 88440C2C        mov byte ptr [esp+ecx+2C], al    <-----生成第 ecx+9 位注册码，不符合范围再进行调整。
:00407399 7D0A            jge 004073A5
:0040739B B05A            mov al, 5A
:0040739D 2AC2            sub al, dl
:0040739F 88440C2C        mov byte ptr [esp+ecx+2C], al    <-----生成第 ecx+9 位注册码
:004073A3 EB1C            jmp 004073C1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407399(C)

:004073A5 3C5A            cmp al, 5A
:004073A7 7E09            jle 004073B2
:004073A9 3C61            cmp al, 61
:004073AB 7D05            jge 004073B2
:004073AD 80C261          add dl, 61
:004073B0 EB0B            jmp 004073BD

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004073A7(C), :004073AB(C)

:004073B2 807C0C247A      cmp byte ptr [esp+ecx+24], 7A
:004073B7 7E08            jle 004073C1
:004073B9 B27A            mov dl, 7A
:004073BB 2AD3            sub dl, bl

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004073B0(U)

:004073BD 88540C2C        mov byte ptr [esp+ecx+2C], dl
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
```



|:004073A3(U), :004073B7(C)

:004073C1 41 `inc ecx` `<-------ecx=ecx+1`

:004073C2 83C704 `add edi, 00000004` `<-------改变 edi 的值用于下次计算`

:004073C5 83F908 `cmp ecx, 00000008` `<-------供取八次（由此可知，注册名用八位以上的字符毫无意义，只取前八位。）`

:004073C8 7C87 `jl 00407351` `<-------循环`

整理一下：

`Name: Crack007(Crack007\[BCG\]也可，不过没什么实际意义)`

`Code: DosHAedFWgpBZcdU`

似乎是万事大吉了，但，但最初的 `edi` 等值是怎么来的，我一直没看出来。于是对写注册机而言以上变得毫无意义。大家帮帮忙指点一下吧。 `4dd466d0e1e0cc123afbe504da2462ca`
