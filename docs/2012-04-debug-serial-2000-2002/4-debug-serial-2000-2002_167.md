
# 象棋桥 2000

> 原文：[`www.pediy.com/kssd/pediy02/forum277.htm`](https://www.pediy.com/kssd/pediy02/forum277.htm)

目标程序：象棋桥最新版

保护方式：采用注册名与注册码形式，不是旧版采用机器码与注册码的形式。

难度：易

工具：`softice4.05`

下载：CHINA 主页共享软件栏目或华军软件园 `WWW.NEWHUA.COM`

填入用户名：`backboy`

注册码：`78787878`

下 `BPX HMEMCPY` `F5` 返回，按确定后中断返回程序空间，按 `F10` 来到如下代码处：

```
:0040860C E8D3BAFFFF call 004040E4
:00408611 8BD6 mov edx, esi
:00408613 8B37 mov esi, dword ptr [edi]
:00408615 85DB test ebx, ebx
:00408617 7415 je 0040862E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040862C(C)

:00408619 8A02 mov al, byte ptr [edx]
:0040861B 3C61 cmp al, 61
:0040861D 7206 jb 00408625
:0040861F 3C7A cmp al, 7A
:00408621 7702 ja 00408625
:00408623 2C20 sub al, 20

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:0040861D(C), :00408621(C)

:00408625 8806 mov byte ptr [esi], al
:00408627 42 inc edx
:00408628 46 inc esi
:00408629 4B dec ebx
:0040862A 85DB test ebx, ebx
:0040862C 75EB jne 00408619
```

以上代码逐个比较输入的注册码是否为小写字母，是则出错

============================================================

```
:0049E2FC E8AF5AF6FF call 00403DB0
:0049E301 83F80E cmp eax, 0000000E 此处比较输入注册码的位数是否为 14 位
:0049E304 7525 jne 0049E32B 不是 14 位就死，重新填入 14 位注册码 12345678901234 再来一次。
:0049E306 8D45F4 lea eax, dword ptr [ebp-0C]
:0049E309 B905000000 mov ecx, 00000005
:0049E30E BA0A000000 mov edx, 0000000A
:0049E313 E8E05CF6FF call 00403FF8
:0049E318 8B45F4 mov eax, dword ptr [ebp-0C]
一路 F10 来到
:0049E31B BA38E54900 mov edx, 0049E538
:0049E320 E89B5BF6FF call 00403EC0 此 CALL 为比较核心 F8 跟入
:0049E325 7406 je 0049E32D
:0049E327 33DB xor ebx, ebx
:0049E329 EB02 jmp 0049E32D
```

============================================================

```
:00403EC0 53 push ebx
:00403EC1 56 push esi
:00403EC2 57 push edi
:00403EC3 89C6 mov esi, eax
:00403EC5 89D7 mov edi, edx
:00403EC7 39D0 cmp eax, edx 此时的 D EAX 为输入的假码，D EDX 为真注册码
程序只作前 9 位比较
此时的 EDX 内容为 CCB2000R-
EAX 内容为 12345678901234
:00403EC9 0F848F000000 je 00403F5E 相等则注册成功
```

至此注册码已出来，象棋桥 2000 最新版的注册码与用户名无关，注册码形式为 `CCB2000R-*****`（此处的 `*` 为除小写字母外的其它数字或字母）

`backboy/CCB2000R-12345`

另 `winxfile` 同一公司出品的 `ABSOLUTE SECURITY PRO3.7`（能加密各种文件）的注册比较核心在 `0137:0048c4cb` `call 000403914` 时 `F8` 进入。

一路 `F10` 到：

```
0137:00403917 mov esi,eax
0137:00403919 mov edi,edx
0137:0040391b cmp eax,edx 此时的 D EDX 和 D EAX 即可见到注册码。
0137:0040391d jz 004039b2
```

一组注册码如下：

`backboy/PHJCRLAPDCAFTGV`

`WWW.PEPSOFT.COM` `4dd466d0e1e0cc123afbe504da2462ca`
