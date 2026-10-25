
# 古今大战 80 分 V3.0 Beta

> 原文：`www.pediy.com/kssd/pediy05/pediy50582.htm` ([http://www.pediy.com/kssd/pediy05/pediy50582.htm](http://www.pediy.com/kssd/pediy05/pediy50582.htm))
>
> 下载页面： `www.skycn.com/soft/1516.html` ([http://www.skycn.com/soft/1516.html](http://www.skycn.com/soft/1516.html))
>
> 软件大小： 442 KB
>
> 软件语言： 简体中文
>
> 软件类别： 国产软件 / 共享版 / 棋牌游戏
>
> 应用平台： `Win9x/NT/2000/XP`
>
> 加入时间： 2001-01-05 00:00:00
>
> 下载次数： 43870
>
> 推荐等级： *****
>
> 开发 商： `www.wj2000.50megs.com/` ([http://www.wj2000.50megs.com/](http://www.wj2000.50megs.com/))

## 软件简介
80 分，又叫拖拉机、双升等，相信大家不陌生。本游戏可说是此种扑克游戏的精品，囊括各种翻主、打牌、计分、升级的方法。玩家可以控制翻主过程，机器玩家采用人工智能出牌打法，有不同难度级别，试玩级别相信你是可以战胜它的。另外，界面美观并可设置背景、牌张、发、收牌速度；可以提取、保存进度；播放背景音乐；等等。

## 软件限制
功能限制

## 作者声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具
`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0` 白金版

---

## 过程
呵呵，看到 透明的 朋友有篇教程，我也写一下算法吧。沾光了，不知道 透明的 朋友是否有意见？

`cents80.exe` 无壳。`Borland Delphi` 编写。呵呵，我等菜鸟喜欢的类型呀。

不明白 `W32Dasm 10` 修改版为何许多“参考”反汇编不出来？换用 `pll621[CCG]` 大侠修改的白金版了。

- 序列号：`456`
- 试炼码：`13572468`

```
—————————————————————————————————

:0045578A 8BC0                   mov eax, eax
:0045578C 55                     push ebp
:0045578D 8BEC                   mov ebp, esp
:0045578F 6A00                   push 00000000
:00455791 6A00                   push 00000000
:00455793 53                     push ebx
:00455794 56                     push esi
:00455795 57                     push edi
:00455796 8BF0                   mov esi, eax
:00455798 33C0                   xor eax, eax
:0045579A 55                     push ebp
:0045579B 68AE584500             push 004558AE
:004557A0 64FF30                 push dword ptr fs:[eax]
:004557A3 648920                 mov dword ptr fs:[eax], esp
:004557A6 33C0                   xor eax, eax
:004557A8 55                     push ebp
:004557A9 68D6574500             push 004557D6
:004557AE 64FF30                 push dword ptr fs:[eax]
:004557B1 648920                 mov dword ptr fs:[eax], esp
:004557B4 8D55F8                 lea edx, dword ptr [ebp-08]
:004557B7 8B86EC010000           mov eax, dword ptr [esi+000001EC]
:004557BD E8B690FCFF             call 0041E878
                                 ==== > 取 序列号

:004557C2 8B45F8                 mov eax, dword ptr [ebp-08]
                                 ==== > EAX = 456

:004557C5 E8B218FBFF             call 0040707C
                                 ==== > 把 序列号 转换成用 16 进制值表示

:004557CA 8BD8                   mov ebx, eax
                                 ==== > EAX = 1C8 (H) = 456 (D)

:004557CC 33C0                   xor eax, eax
:004557CE 5A                     pop edx
:004557CF 59                     pop ecx
:004557D0 59                     pop ecx
:004557D1 648910                 mov dword ptr fs:[eax], edx
:004557D4 EB14                   jmp 004557EA
:004557D6 E961DBFAFF             jmp 0040333C
:004557DB E800DEFAFF             call 004035E0
:004557E0 E9AB000000             jmp 00455890
:004557E5 E8F6DDFAFF             call 004035E0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :004557D4(U)
|

:004557EA 81FB48890100           cmp ebx, 00018948
                                 ==== > 序列号的 16 进制值大于 18948？

:004557F0 7C14                   jl 00455806
:004557F2 81FB52890100           cmp ebx, 00018952
:004557F8 7F0C                   jg 00455806
:004557FA 8D55FC                 lea edx, dword ptr [ebp-04]
:004557FD 8BC3                   mov eax, ebx
:004557FF E890FBFFFF             call 00455394
:00455804 EB0A                   jmp 00455810
                                 ==== > 这里跳走就 OVER 了！呵呵

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
| :004557F0(C), :004557F8(C)
|

:00455806 8D55FC                 lea edx, dword ptr [ebp-04]
:00455809 8BC3                   mov eax, ebx
:0045580B E804FCFFFF             call 00455414
                                 ==== > 算法 CALL！进入！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00455804(U)
|

:00455810 8D55F8                 lea edx, dword ptr [ebp-08]
:00455813 8B86F0010000           mov eax, dword ptr [esi+000001F0]
:00455819 E85A90FCFF             call 0041E878
:0045581E 8B45F8                 mov eax, dword ptr [ebp-08]
                                 ==== > EAX = 13572468          试炼码!

:00455821 8B55FC                 mov edx, dword ptr [ebp-04]
                                 ==== > EDX = CJBHWRQG          注册码!

:00455824 E87BE6FAFF             call 00403EA4
                                 ==== > 比较 CALL！

:00455829 7554                   jne 0045587F
                                 ==== > 跳则 OVER！
                                 ==== > 下面是保存注册信息

* Possible StringData Ref from Code Obj -> "wjupgrad.ini"
                                |
:0045582B B9C4584500             mov ecx, 004558C4
:00455830 B201                   mov dl, 01
:00455832 A110A54400             mov eax, dword ptr [0044A510]
:00455837 E8304DFFFF             call 0044A56C
:0045583C 8BF0                   mov esi, eax
:0045583E 53                     push ebx

* Possible StringData Ref from Code Obj -> "SerialNo"
                                |
:0045583F B9DC584500             mov ecx, 004558DC

* Possible StringData Ref from Code Obj -> "Register"
                                |
:00455844 BAF0584500             mov edx, 004558F0
:00455849 8BC6                   mov eax, esi
:0045584B E8144FFFFF             call 0044A764
:00455850 8B45FC                 mov eax, dword ptr [ebp-04]
:00455853 50                     push eax

* Possible StringData Ref from Code Obj -> "Code"
                                |
:00455854 B904594500             mov ecx, 00455904

* Possible StringData Ref from Code Obj -> "Register"
                                |
:00455859 BAF0584500             mov edx, 004558F0
:0045585E 8BC6                   mov eax, esi
:00455860 E89B4DFFFF             call 0044A600
:00455865 A1348D4600             mov eax, dword ptr [00468D34]
:0045586A 8B00                   mov eax, dword ptr [eax]
:0045586C C6807716000001         mov byte ptr [eax+00001677], 01

* Possible StringData Ref from Code Obj -> "注册成功"
                                 ==== > 呵呵，胜利女神！

:00455873 B814594500             mov eax, 00455914
:00455878 E8E73EFEFF             call 00439764
:0045587D EB11                   jmp 00455890

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00455829(C)
|

* Possible StringData Ref from Code Obj -> "注册码不正确"

:0045587F B828594500             mov eax, 00455928
                                 ==== > BAD BOY！
```

---

进入算法 `CALL`：`45580B E804FCFFFF call 00455414`

```
* Referenced by a CALL at Addresses:
| :0045580B, :004559B4, :00456A87
|

:00455414 55                     push ebp
:00455415 8BEC                   mov ebp, esp
:00455417 6A00                   push 00000000
:00455419 6A00                   push 00000000
:0045541B 6A00                   push 00000000
:0045541D 53                     push ebx
:0045541E 56                     push esi
:0045541F 8BF2                   mov esi, edx
:00455421 8BD8                   mov ebx, eax
:00455423 33C0                   xor eax, eax
:00455425 55                     push ebp
:00455426 6887544500             push 00455487
:0045542B 64FF30                 push dword ptr fs:[eax]
:0045542E 648920                 mov dword ptr fs:[eax], esp
:00455431 8D55FC                 lea edx, dword ptr [ebp-04]
:00455434 8BC3                   mov eax, ebx
                                 ==== > EAX = EBX = 1C8

:00455436 03C0                   add eax, eax
```



### 第一步

```
:00455438 E8DFFEFFFF                call 0045531C
```
> `EAX`=1C8 + 1C8=390

> 子运算 `CALL`！得出注册码的前几位

```
:0045543D FF75FC                    push [ebp-04]
```
> `[ebp-04]`=CJB

```
:00455440 8D55F8                    lea edx, dword ptr [ebp-08]

:00455443 8BC3                      mov eax, ebx
```
> `EAX`=`EBX`=1C8

```
:00455445 C1E806                    shr eax, 06
```

### 第二步

> `EAX`=1C8 SHR 06=7

```
:00455448 E8CFFEFFFF                call 0045531C
```
> 子运算 `CALL`！得出注册码中间几位

```
:0045544D FF75F8                    push [ebp-08]
```
> `[ebp-08]`=H

```
:00455450 8D55F4                    lea edx, dword ptr [ebp-0C]

:00455453 8BC3                      mov eax, ebx
```
> `EAX`=`EBX`=1C8

```
:00455455 C1E008                    shl eax, 08
```

### 第三步

> `EAX`=1C8 SHL 08=1C800

```
:00455458 E8BFFEFFFF                call 0045531C
```
> 子运算 `CALL`！得出注册码的后几位

```
:0045545D FF75F4                    push [ebp-0C]
```
> `[ebp-0C]`=WRQG

```
:00455460 8BC6                      mov eax, esi

:00455462 BA03000000                mov edx, 00000003

:00455467 E8E8E9FAFF                call 00403E54
```
> 此 `CALL` 把以上 3 步所得字符连接起来！

```
:0045546C 33C0                      xor eax, eax

:0045546E 5A                        pop edx

:0045546F 59                        pop ecx

:00455470 59                        pop ecx

:00455471 648910                    mov dword ptr fs:[eax], edx

:00455474 688E544500                push 0045548E
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0045548C(U)

```
:00455479 8D45F4                    lea eax, dword ptr [ebp-0C]

:0045547C BA03000000                mov edx, 00000003

:00455481 E8B6E6FAFF                call 00403B3C

:00455486 C3                         ret
```

---

进入子运算 `CALL`：`455438` -> `call 0045531C`

因为 3 部分的运算流程都是相同的，只是参数不同。所以我只记录了第一步的过程。

> Referenced by a `CALL` at Addresses:
> | :004553B9, :004553C8, :004553D8, :00455438, :00455448
> | :00455458

```
:0045531C 55                        push ebp

:0045531D 8BEC                      mov ebp, esp

:0045531F 6A00                      push 00000000

:00455321 53                        push ebx

:00455322 56                        push esi

:00455323 8BF2                      mov esi, edx

:00455325 8BD8                      mov ebx, eax

:00455327 33C0                      xor eax, eax

:00455329 55                        push ebp

:0045532A 6885534500                push 00455385
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :004552BF(C)

```
:0045532F 64FF30                    push dword ptr fs:[eax]

:00455332 648920                    mov dword ptr fs:[eax], esp

:00455335 8BC6                      mov eax, esi

:00455337 E8DCE7FAFF                call 00403B18

:0045533C 85DB                      test ebx, ebx

:0045533E 7E2F                      jle 0045536F
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0045536D(C)

```
:00455340 8BC3                      mov eax, ebx
```
> `EAX`=`EBX`=390

```
:00455342 B91A000000                mov ecx, 0000001A
```
> `ECX`=1A

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :004552D1(C)

```
:00455347 99                        cdq

:00455348 F7F9                       idiv ecx
```
> 循环与 1A 求模，直至商为 0！
> 1、`EDX`=390 % 1A=2
> 2、`EDX`=23 % 1A=9
> 3、`EDX`=1 % 1A=1

```
:0045534A 83C241                    add edx, 00000041
```
> 余数加 41
> 1、`EDX`=2 + 41=43 即：字符 C
> 2、`EDX`=9 + 41=4A 即：字符 J
> 3、`EDX`=1 + 41=42 即：字符 B

```
:0045534D 8D45FC                    lea eax, dword ptr [ebp-04]

:00455350 E867E9FAFF                call 00403CBC

:00455355 8B55FC                    mov edx, dword ptr [ebp-04]

:00455358 8BC6                      mov eax, esi

:0045535A E83DEAFAFF                call 00403D9C
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :00455309(C)

```
:0045535F 8BC3                      mov eax, ebx

:00455361 B91A000000                mov ecx, 0000001A

:00455366 99                        cdq

:00455367 F7F9                       idiv ecx
```
> 循环除以 1A，求商！
> 1、`EDX`=390 / 1A=23
> 2、`EDX`=23 / 1A=1
> 3、`EDX`=1 / 1A=0

```
:00455369 8BD8                      mov ebx, eax
```
> 商入 `EBX`，继续下次求模！

```
:0045536B 85DB                      test ebx, ebx

:0045536D 7FD1                      jg 00455340
```
> 循环

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0045533E(C)

```
:0045536F 33C0                      xor eax, eax

:00455371 5A                        pop edx

:00455372 59                        pop ecx

:00455373 59                        pop ecx
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :00455305(C)

```
:00455374 648910                    mov dword ptr fs:[eax], edx

:00455377 688C534500                push 0045538C
```

> Referenced by a (U)nconditional or (C)onditional Jump at Address:
> | :0045538A(U)

```
:0045537C 8D45FC                    lea eax, dword ptr [ebp-04]

:0045537F E894E7FAFF                call 00403B18

:00455384 C3                         ret
```

---

### 算法总结

1. 将用户输入的序列号转化为 16 进制值。
2. 用序列号的 16 进制值的平方循环和 `1A` 求模，直至商为 0。余数 + `41`。
3. 用序列号的 16 进制值逻辑右移 6 位后的值，循环和 `1A` 求模，直至商为 0。余数 + `41`。
4. 用序列号的 16 进制值逻辑左移 8 位后的值，循环和 `1A` 求模，直至商为 0。余数 + `41`。
5. 以上 3 部分运算所得字符连接起来就是注册码了。

---

### 完美爆破

```
0045581E 8B45F8                    mov eax, dword ptr [ebp-08]
```
改为：
```
8B45FC                              mov eax, dword ptr [ebp-04]
```
呵呵，和下面的 `00455821` 处相映成趣！让真的注册码去和真注册码比较，岂有不 OK 的？

---

### KeyMake 之{56th}内存注册机

- 中断地址：`455824`
- 中断次数：1
- 第一字节：`E8`
- 指令长度：5
- 内存方式：`EDX`

---

### 注册信息保存

`C:\WINDOWS` 下的 `wjupgrad.ini` 文件中：

```ini
[Register]
SerialNo=456
Code=CJBHWRQG
```

---

### 整理

- 序列号：`456`
- 注册码：`CJBHWRQG`

---

Cracked By 巢水工作坊——fly【OCN】

2003-4-13 18:18 `4dd466d0e1e0cc123afbe504da2462ca`
