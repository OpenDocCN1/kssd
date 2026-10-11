
# 网页特效梦工厂 XP 1.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50604.htm`](https://www.pediy.com/kssd/pediy05/pediy50604.htm)

下载页面： [`www.skycn.com/soft/8556.html`](http://www.skycn.com/soft/8556.html)

软件大小： 815 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 网页辅助

应用平台： `Win9x`/`NT`/`2000`/`XP`

加入时间： 2002-12-26 11:00:02

下载次数： 15888

推荐等级： \*\*\*

开发商： [`remotion.myetang.com/`](http://remotion.myetang.com/)

**【软件简介】**：你在做网页嘛，如果是的话，这是你不可多得的超 Cool 工具。这是可以自动生成网页特效的软件，每个特效都可以由您进行参数设置，100% 傻瓜性。收集了包括时间特效，文字特效，图像处理，鼠标特效，页面特效，菜单特效，在线游戏，其它特效在内的八类上百个精彩特效。这些特效都是使用率比较高的 `javascript` 代码，您可以直接使用。软件内置浏览器，您可以随时预览特效效果；特效制作好之后，您可以把它复制到剪贴板，或者保存到文件中；软件的界面美观新颖，操作简单明了，极易上手。轻轻点几下鼠标，又炫又酷的网页任你选！

**【软件限制】**：试用 40 次。

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`ProDump`、`W32Dasm 9.0` 白金版

—————————————————————————————————

**【过程】**：

`JSbuilder.exe` 是 `UPX 1.23` 壳，用 `ProDump` 脱之。429K->1.33M。`Delphi` 编写。

申请码：61007649

试炼码：13572468

很简单，反汇编就轻松找到下面的核心了。

—————————————————————————————————

```asm
* Possible StringData Ref from Data Obj ->"请输入注册认证码!"

                       |
:004D695C B8D46A4D00            mov eax, 004D6AD4
:004D6961 E83EF5F5FF            call 00435EA4
:004D6966 8B96200A0000          mov edx, dword ptr [esi+00000A20]
:004D696C A1EC405000            mov eax, dword ptr [005040EC]
:004D6971 E82613F8FF            call 00457C9C
:004D6976 E91E010000            jmp 004D6A99

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D695A(C)
|
:004D697B 8D55FC                lea edx, dword ptr [ebp-04]
:004D697E 8B86200A0000          mov eax, dword ptr [esi+00000A20]
:004D6984 E85F62F6FF            call 0043CBE8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D6918(C)
|
:004D6989 8D55F4                lea edx, dword ptr [ebp-0C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D6916(C)
|
:004D698C 8B86200A0000          mov eax, dword ptr [esi+00000A20]
:004D6992 E85162F6FF            call 0043CBE8
:004D6997 8B45F4                mov eax, dword ptr [ebp-0C]
:004D699A E88DE3F2FF            call 00404D2C
:004D699F 8BD0                  mov edx, eax
:004D69A1 85D2                  test edx, edx
:004D69A3 7E3C                  jle 004D69E1
:004D69A5 B801000000            mov eax, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D69DF(C)
|
:004D69AA 8B4DFC                mov ecx, dword ptr [ebp-04]
                       ====>ECX=13572468           试炼码
:004D69AD 8A4C01FF             mov cl, byte ptr [ecx+eax-01]
:004D69B1 80F930                cmp cl, 30
:004D69B4 7208                  jb 004D69BE
:004D69B6 8B5DFC                mov ebx, dword ptr [ebp-04]
:004D69B9 80F939                cmp cl, 39
:004D69BC 761F                  jbe 004D69DD
                       ====>不是数字则 GAME OVER！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D69B4(C)
|

* Possible StringData Ref from Data Obj ->"注册码输入有误,请重新输入!"

                       |
:004D69BE B8F06A4D00            mov eax, 004D6AF0
:004D69C3 E8DCF4F5FF            call 00435EA4
:004D69C8 8B96200A0000          mov edx, dword ptr [esi+00000A20]
:004D69CE A1EC405000            mov eax, dword ptr [005040EC]
:004D69D3 E8C412F8FF            call 00457C9C
:004D69D8 E9BC000000            jmp 004D6A99

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D69BC(C)
|
:004D69DD 40                    inc eax
:004D69DE 4A                    dec edx
:004D69DF 75C9                  jne 004D69AA
                       ====>循环检测输入的注册码是否是数字？

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D69A3(C)
|
:004D69E1 8D55F0                lea edx, dword ptr [ebp-10]
:004D69E4 8B861C0A0000          mov eax, dword ptr [esi+00000A1C]
:004D69EA E8F961F6FF            call 0043CBE8
:004D69EF 8B45F0                mov eax, dword ptr [ebp-10]
                       ====>EAX=61007649            申请码
:004D69F2 E8F126F3FF            call 004090E8
                       ====>把 61007649 转化成 16 进制值 03A2E721
:004D69F7 E88829FFFF            call 004C9384
                       ====>算法 CALL！进入！
:004D69FC 8BD8                  mov ebx, eax
                       ====>EBX=5E9B5488
:004D69FE 8B45FC                mov eax, dword ptr [ebp-04]
                       ====>EAX=13572468
:004D6A01 E8E226F3FF            call 004090E8
                       ====>把 13572468 转化成 16 进制值 00CF1974
:004D6A06 3BD8                  cmp ebx, eax
                       ====>比较注册码！
                       ====>EBX=5E9B5488
                       ====>EAX=00CF1974
                       呵呵，所以我的注册码就是 5E9B5488 的 10 进制值 1587238024
:004D6A08 0F8581000000          jne 004D6A8F
                       ====>跳则 OVER！
:004D6A0E B201                  mov dl, 01

* Possible StringData Ref from Data Obj ->""

                       |
:004D6A10 A140B34600            mov eax, dword ptr [0046B340]
:004D6A15 E8264AF9FF            call 0046B440
:004D6A1A 8BD8                  mov ebx, eax
:004D6A1C BA02000080            mov edx, 80000002
:004D6A21 8BC3                  mov eax, ebx
:004D6A23 E8B84AF9FF            call 0046B4E0
:004D6A28 33C9                  xor ecx, ecx

* Possible StringData Ref from Data Obj ->"System\\CurrentControlSet\\Services\\Class\\knight"
                          ->"soft\\JSBuilder"
                       |
:004D6A2A BA146B4D00            mov edx, 004D6B14
:004D6A2F 8BC3                  mov eax, ebx
:004D6A31 E8EA4BF9FF            call 0046B620
:004D6A36 8D55EC                lea edx, dword ptr [ebp-14]
:004D6A39 8B861C0A0000          mov eax, dword ptr [esi+00000A1C]
:004D6A3F E8A461F6FF            call 0043CBE8
:004D6A44 8B45EC                mov eax, dword ptr [ebp-14]
:004D6A47 E89C26F3FF            call 004090E8
:004D6A4C E83329FFFF            call 004C9384
:004D6A51 8BC8                  mov ecx, eax

* Possible StringData Ref from Data Obj ->"registecode"
                       ====>在注册表中保存注册信息
:004D6A53 BA5C6B4D00            mov edx, 004D6B5C
:004D6A58 8BC3                  mov eax, ebx
:004D6A5A E8114DF9FF            call 0046B770
:004D6A5F 8BC3                  mov eax, ebx
:004D6A61 E84A4AF9FF            call 0046B4B0
:004D6A66 8BC3                  mov eax, ebx
:004D6A68 E89BD2F2FF            call 00403D08

* Possible StringData Ref from Data Obj ->"网页特效梦工厂 XP 注册成功,谢谢您对本软件的支?
                          ->"?您的认可是我最大的动力!"
                       ====>呵呵，胜利女神！
:004D6A6D B8706B4D00            mov eax, 004D6B70
:004D6A72 E82DF4F5FF            call 00435EA4

* Possible StringData Ref from Data Obj ->"请重新启动本软件,以完成软件的注册"

                       |
:004D6A77 B8C06B4D00            mov eax, 004D6BC0
:004D6A7C E823F4F5FF            call 00435EA4
:004D6A81 A1FC7C4F00            mov eax, dword ptr [004F7CFC]
:004D6A86 8B00                  mov eax, dword ptr [eax]
:004D6A88 E81F62F8FF            call 0045CCAC
:004D6A8D EB0A                  jmp 004D6A99

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D6A08(C)
|

* Possible StringData Ref from Data Obj ->"错误,网页特效梦工厂 XP 注册认证失败"
                       ====>BAD BOY！ 
:004D6A8F B8EC6B4D00            mov eax, 004D6BEC
```



## 反汇编代码分析：入口点相关

`:004D6A94 E80BF4F5FF` `call 00435EA4`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:004D6976`(U), `:004D69D8`(U), `:004D6A8D`(U)

`:004D6A99 33C0` `xor eax, eax`
`:004D6A9B 5A` `pop edx`
`:004D6A9C 59` `pop ecx`
`:004D6A9D 59` `pop ecx`
`:004D6A9E 648910` `mov dword ptr fs:[eax], edx`
`:004D6AA1 68C36A4D00` `push 004D6AC3`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004D6AC1`(U)

`:004D6AA6 8D45EC` `lea eax, dword ptr [ebp-14]`
`:004D6AA9 BA04000000` `mov edx, 00000004`
`:004D6AAE E8E5DFF2FF` `call 00404A98`
`:004D6AB3 8D45FC` `lea eax, dword ptr [ebp-04]`
`:004D6AB6 E8B9DFF2FF` `call 00404A74`
`:004D6ABB C3` `ret`

---

## 进入算法 CALL：`4D69F7` `call 004C9384`

> \* Referenced by a CALL at Addresses: `:004C9BA6`, `:004D69F7`, `:004D6A4C`

`:004C9384 55` `push ebp`
`:004C9385 8BEC` `mov ebp, esp`
`:004C9387 6A00` `push 00000000`
`:004C9389 53` `push ebx`
`:004C938A 56` `push esi`
`:004C938B 8BF0` `mov esi, eax`
`:004C938D 33C0` `xor eax, eax`
`:004C938F 55` `push ebp`
`:004C9390 6852944C00` `push 004C9452`
`:004C9395 64FF30` `push dword ptr fs:[eax]`
`:004C9398 648920` `mov dword ptr fs:[eax], esp`
`:004C939B BBD9D10E00` `mov ebx, 000ED1D9`
> `====>` `EBX`= `000ED1D9`  呵呵，971225，97 年的圣诞？

`:004C93A0 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"wangshuang"`
> `====>` 呵呵，作者的朋友？

`:004C93A3 BA68944C00` `mov edx, 004C9468`
> `====>` `EDX`= `wangshuang`

`:004C93A8 E85FB7F3FF` `call 00404B0C`
`:004C93AD 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C93B0 E877B9F3FF` `call 00404D2C`
> `====>` 取 `wangshuang` 的长度

`:004C93B5 85C0` `test eax, eax`
> `====>` `EAX`= `A`

`:004C93B7 7E1B` `jle 004C93D4`
`:004C93B9 BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93D2`(C)

`:004C93BE 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C93C1 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `wangshuang` 字符的 HEX 值
> 1、 `====>` `ECX`= `77`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `67`

`:004C93C6 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `77` \* `5`= `00000253`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `67` \* `5`= `00000203`

`:004C93C9 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `00000253` \* `5`= `00000B9F`
> …… …… 省 略 …… ……
> 10、 `====>` `ECX`= `00000203` \* `5`= `00000A0F`

`:004C93CC 03D9` `add ebx, ecx`
> 1、 `====>` `EBX`= `000ED1D9` + `00000B9F`= `000EDD78`
> …… …… 省 略 …… ……
> 10、 `====>` `EBX`= `20C950EE` + `00000A0F`= `20C95AFD`

`:004C93CE 03DE` `add ebx, esi`
> 1、 `====>` `EBX`= `000EDD78` + `03A2E721`= `03B1C499`
> …… …… 省 略 …… ……
> 10、 `====>` `EBX`= `20C95AFD` + `03A2E721`= `246C421E`

`:004C93D0 42` `inc edx`
`:004C93D1 48` `dec eax`
`:004C93D2 75EA` `jne 004C93BE`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93B7`(C)

`:004C93D4 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"yaoyuan"`
> `====>` 呵呵，作者的大名

`:004C93D7 BA7C944C00` `mov edx, 004C947C`
> `====>` `EDX`= `yaoyuan`

`:004C93DC E82BB7F3FF` `call 00404B0C`
`:004C93E1 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C93E4 E843B9F3FF` `call 00404D2C`
> `====>` 取 `yaoyuan` 的长度

`:004C93E9 85C0` `test eax, eax`
> `====>` `EAX`= `7`

`:004C93EB 7E1B` `jle 004C9408`
`:004C93ED BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C9406`(C)

`:004C93F2 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C93F5 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `yaoyuan` 字符的 HEX 值
> 1、 `====>` `ECX`= `79`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `6E`

`:004C93FA 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `79` \* `5`= `0000025D`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `6E` \* `5`= `00000226`

`:004C93FD 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `0000025D` \* `5`= `00000BD1`
> …… …… 省 略 …… ……
> 7、 `====>` `ECX`= `00000226` \* `5`= `00000ABE`

`:004C9400 03D9` `add ebx, ecx`
> 1、 `====>` `EBX`= `246C421E` + `00000203`= `246C4DEF`
> …… …… 省 略 …… ……
> 7、 `====>` `EBX`= `3A3DEDBC` + `00000ABE`= `20C95AFD`

`:004C9402 03DE` `add ebx, esi`
> 1、 `====>` `EBX`= `246C4DEF` + `03A2E721`= `280F3510`
> …… …… 省 略 …… ……
> 7、 `====>` `EBX`= `3A3DF87A` + `03A2E721`= `3DE0DF9B`

`:004C9404 42` `inc edx`
`:004C9405 48` `dec eax`
`:004C9406 75EA` `jne 004C93F2`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C93EB`(C)

`:004C9408 8D45FC` `lea eax, dword ptr [ebp-04]`

> \* Possible StringData Ref from Data Obj -> `"JSBuilder"`
> `====>` 呵呵，软件的大名

`:004C940B BA8C944C00` `mov edx, 004C948C`
> `====>` `EDX`= `JSBuilder`

`:004C9410 E8F7B6F3FF` `call 00404B0C`
`:004C9415 8B45FC` `mov eax, dword ptr [ebp-04]`
`:004C9418 E80FB9F3FF` `call 00404D2C`
> `====>` 取 `JSBuilder` 的长度

`:004C941D 85C0` `test eax, eax`
> `====>` `EAX`= `9`

`:004C941F 7E1B` `jle 004C943C`
`:004C9421 BA01000000` `mov edx, 00000001`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004C943A`(C)

`:004C9426 8B4DFC` `mov ecx, dword ptr [ebp-04]`
`:004C9429 0FB64C11FF` `movzx ecx, byte ptr [ecx+edx-01]`
> `====>` 依次取 `JSBuilder` 字符的 HEX 值
> 1、 `====>` `ECX`= `4A`
> …… …… 省 略 …… ……
> 9、 `====>` `ECX`= `72`

`:004C942E 8D0C89` `lea ecx, dword ptr [ecx+4*ecx]`
> 1、 `====>` `ECX`= `4A` \* `5`= `00000172`
> …… …… 省 略 …… ……
> 9、 `====>` `ECX`= `72` \* `5`= `0000023A`



`:004C9431 8D0C89` 指令对应汇编代码：
```
lea ecx, dword ptr [ecx+4*ecx]
```
计算过程示例：
- 第 1 次循环：`ECX=00000172`，`* 5 = 0000073A`
- ...
- 第 9 次循环：`ECX=0000023A`，`* 5 = 00000B22`

`:004C9434 03D9` 指令对应汇编代码：
```
add ebx, ecx
```
计算过程示例：
- 第 1 次循环：`EBX=3DE0DF9B + 0000073A = 3DE0E6D5`
- ...
- 第 9 次循环：`EBX=5AF86245 + 00000B22 = 5AF86D67`

`:004C9436 03DE` 指令对应汇编代码：
```
add ebx, esi
```
计算过程示例：
- 第 1 次循环：`EBX=3DE0E6D5 + 03A2E721 = 4183CDF6`
- ...
- 第 9 次循环：`EBX=5AF86D67 + 03A2E721 = 5E9B5488`

注册码即为 `5E9B5488` 的 10 进制值。

后续指令流：
`:004C9438 42`
```
inc edx
```
`:004C9439 48`
```
dec eax
```
`:004C943A 75EA`
```
jne 004C9426
```
* 被无条件或条件跳转引用的地址：`:004C941F` (条件跳转 C)

`:004C943C 33C0`
```
xor eax, eax
```
`:004C943E 5A`
```
pop edx
```
`:004C943F 59`
```
pop ecx
```
`:004C9440 59`
```
pop ecx
```
`:004C9441 648910`
```
mov dword ptr fs:[eax], edx
```
`:004C9444 6859944C00`
```
push 004C9459
```
* 被无条件或条件跳转引用的地址：`:004C9457` (无条件跳转 U)

`:004C9449 8D45FC`
```
lea eax, dword ptr [ebp-04]
```
`:004C944C E823B6F3FF`
```
call 00404A74
```
`:004C9451 C3`
```
ret
```

---

## C++ KeyGen

虽然有不少循环，但是主要参数最终是固定值。算法非常简单。使用以下 C++ 代码作为注册机：

```cpp
#include<iostream.h>
#include<math.h>

void main()
{
    int m;
    cout<<"★★★★网页特效梦工厂 XP 1.5 KeyGen{8th}★★★★\n\n\n\n";
    cout<<"请输入申请码:";
    cin >>m;
    m *= 0X1A;
    m += 0XFDB2E;
    cout<<"\n 呵呵，注册码:"<<m<<endl;
    cout<<"\n\n\nCracked By 巢水工作坊——fly [OCN][FCG] 2003-4-15 21:21 COMPILE";
    cout<<"\n\n\n             * * * 按回车退出！* * *";
    cin.get();
    cin.get();
}
```

---

## 完美爆破

将地址 `004D6A06` 处的指令：
```
cmp ebx, eax
```
修改为：
```
cmp ebx, ebx
```
让真的去和真的比较。

---

## KeyMake 之{60th}内存注册机

- 中断地址：`4D6A06`
- 中断次数：`1`
- 第一字节：`3D`
- 指令长度：`2`
- 寄存器方式：`EBX`
- 填入的试炼码必须是数字！

---

## 注册信息保存

```regedit
REGEDIT4

[HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Class\knightsoft\JSBuilder]
"version"="1.1"
"times"=dword:00000006 ; 使用次数
"applycode"=dword:03a2e721
"registecode"=dword:5e9b5488
```

---

## 整理

- 申请码：`61007649`
- 注册码：`1587238024`

---

Cracked By 巢水工作坊——fly `[OCN]` `[FCG]`
2003-4-15 21:18
MD5: `4dd466d0e1e0cc123afbe504da2462ca`
