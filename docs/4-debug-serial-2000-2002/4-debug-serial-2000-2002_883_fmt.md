
# Turbolaunch4.04

> 原文：[`www.pediy.com/kssd/pediy04/kanxue399.htm`](https://www.pediy.com/kssd/pediy04/kanxue399.htm)

使用工具：`W32Dasm8.93` 中文版

注：`Turbolaunch4.04` 未经过 `UPX` 的压缩，而其 5.0 版则是经过了 `UPX1.20` 的压缩若要顺利 pj 则需要事先脱壳,但是注册算法是相同的，故此可用同一个注册口令。

首先，用 `W32Dasm` 将目标程序加载，反汇编结束后发现 `String reference` 处于空白状态，没有什么有用的信息可以利用。无奈之间，调试运行用户程序，直到注册窗口出现，而此时想要切回 `W32Dasm` 的领空却是不能，故而继续运行客户程序直到注册出错的窗口出现，此时则可以切回 `W32Dasm` 的领空，于是按 `F8`[`STEP OVER`]或 `F7`[`STEP INTO`]使 `W32Dasm` 处于单步运行状态，然后再切换回用户程序的领空按`[OK]`键令程序运行，这时控制权又一次回到的 `W32Dasm` 手中。从此我们可以控制程序的运作流程了。

令 `W32Dasm` 处于`[ATUO STEP OVER]`状态，自动单步执行用户程序，直到再一次出现注册窗口，此时回到 `W32Dasm` 的领空，按`[STEP INTO]`令 `W32Dasm` 处于单步状态，再回到用户程序的领空，键入用户名`[dola]`和注册码`[77887788]`，完毕后按`[OK]`键，程序被拦停与以下的程序段：

```
:00463B04 mov eax, ebx
:00463B06 call TLAUNCH.00462C48
:00463B0B mov eax, dword ptr [0046CADC]
:00463B10 call TLAUNCH.00425B2C
:00463B15 dec eax <---当前 EIP 的所在位置
:00463B16 je 00463A5A
:00463B1C mov eax, dword ptr [0046CADC]
:00463B21 call TLAUNCH.00402BA0
:00463B26 mov eax, dword ptr [0046CAE0]
:00463B2B call TLAUNCH.00402BA0
:00463B30 xor eax, eax
```

此时按 `F8`[`STEP OVER`]直到以下的程序段：

```
:00463A90 mov eax, 005B7FEC
:00463A95 call TLAUNCH.0040351C
:00463A9A mov ecx, dword ptr [005B7FEC]
:00463AA0 mov edx, dword ptr [005B7FE8]
:00463AA6 mov eax, ebx
:00463AA8 call TLAUNCH.00461674 <---当前 EIP 的所在位置（关键 CALL）STEP INTO IT
:00463AAD mov byte ptr [005B7FF0], al <---放入旗标
:00463AB2 cmp byte ptr [005B7FF0], 00 <---判断旗帜
:00463AB9 je 00463AFB
:00463ABB mov edx, 00463BC4
:00463AC0 lea eax, dword ptr [ebp-04]
```

单步进入上面所视的 CALL，直到如下的程序段：

```
:004617A7 test edx, edx
:004617A9 jne 004617BD
:004617AB cmp edi, 00000012
:004617AE jge 004617BD
:004617B0 lea eax, dword ptr [ebp-0C]
:004617B3 mov edx, 0046186C  <---当前 EIP 的所在位置
:004617B8 call TLAUNCH.00403648
:004617BD inc edi
:004617BE cmp edi, 00000013
:004617C1 jne 004616FE
:004617C7 mov eax, dword ptr [ebp-0C]
```

此时见到 `EDX` 对应的数据区域为（其具体的作用稍后再说）：

```
[edx-00000014] - 47000000  ...G
[edx-00000010] - 53443246  F2DS
[edx-0000000C] - 48383341  A38H
[edx-00000008] - 374c4b4a  JKL7
[edx-00000004] - 5a4e344d  M4NZ
[edx+00000000] - 35564358  XCV5
[edx+00000004] - 55395942  BY9U
[edx+00000008] - 52365450  PT6R
[edx+0000000C] - 51574531  1EWQ
[edx+00000010] - 31493034  40I1
[edx+00000014] - 5a375043  CP7Z
[edx+00000018] - 454f4737  7GOE
[edx+0000001C] - 5a4c5150  PQLZ
```

此时继续按 `F8`[`STEP OVER`]直到以下的程序段：

```
:004617B0 lea eax, dword ptr [ebp-0C]
:004617B3 mov edx, 0046186C
:004617B8 call TLAUNCH.00403648
:004617BD inc edi
:004617BE cmp edi, 00000013 <---检验是否到了 13h 个字节
:004617C1 jne 004616FE
:004617C7 mov eax, dword ptr [ebp-0C]<---真的注册码
:004617CA mov edx, dword ptr [ebp-08]<---假的注册码
:004617CD call TLAUNCH.00403750<---判断函数（为看到真的注册码在此设断点）
:004617D2 sete bl
:004617D5 mov edi, 00000019
```

在断点处停下来后见到如下的数据区：

```
[eax-00000014] - 38383737  7788
[eax-00000010] - 00000000  ....
[eax-0000000C] - 00000026  &...
[eax-00000008] - 00000001  ....
[eax-00000004] - 00000014  ....
[eax+00000000] - 324e4441  ADN2<---注册码
[eax+00000004] - 372d3835  58-7
[eax+00000008] - 3956324a  J2V9
[eax+0000000C] - 53532d54  T-SS
[eax+00000010] - 44384537  7E8D
[eax+00000014] - 0046c400  ..F.
```

上面见到的数据区为计算用的密码表。

由此可以得到正确的注册码为：

- 用户名：`dola`
- 注册码：`ADN258-7J2V9T-SS7E8D`

（同样适用于 5.0 版本）

`4dd466d0e1e0cc123afbe504da2462ca`
