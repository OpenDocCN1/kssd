
# Flashsoft 1.05

> 原文：[`www.pediy.com/kssd/pediy03/forum875.htm`](https://www.pediy.com/kssd/pediy03/forum875.htm)

软件管理专家（`Flashsoft`）1.05 的破解

作者：`Crack007`

主页：`http://www.crack007.com` ( ^-^ )

破解日期：2001-4-22

破解工具：`TRW2000` 1.22

难度：易

软件下载地址：`http://www.newhua.com/FLASHSOFT.HTM`

软件简介：软件管理专家是用来管理下载文件的，你只要选中一个软件，将它拖拽到 `FlashSoft` 的窗口中，填写一些备忘的信息，就把这个下载软件分门别类地放到文件盒中了，下次需要的话，不用去找，直接在软件中调用就 ok 了，是不是挺方便的？通过本软件你可以很方便的知道您下载了什么文件、大小、位置和下载地址以及软件功能等信息，如果您是软件收藏爱好者的话，本软件也可以作为一个您很好的软件收藏目录，象放书的书架一样，随取随用。对于制作软件光盘来说，这可以很方便的做光盘软件目录。

软件注册费：25.00￥/每注册用户

## 1、启动 `TRW2000`

## 2、运行 `Flashsoft`

随便输入注册信息：

用户名: `Crack007`

注册码: `787878`

## 3、按 `CTRL+N` 激活 `TRW2000`

输入如下 2 条命令

`bpx hmemcpy` 设置中断条件

## 4、按 `F5` 键，回到 `Flashsoft.exe` 的注册窗口

按 " OK " 按钮, `TRW2000` 窗口被激活自动弹出

```
bc *
pmodule
```
从系统内核中返回到 `Flashsoft.exe`

中断至以下代码：

```
:004372CB 89430C          mov dword ptr [ebx+0C], eax
```

按 `F10` 键，单步跟踪至以下代码：

```
:004D209E E85D64F8FF      call 00458500
:004D20A3 8B55F8          mov edx, dword ptr [ebp-08]      -|
:004D20A6 8B45FC          mov eax, dword ptr [ebp-04]       |
:004D20A9 E8E679FFFF      call 004C9A94 ←可疑 CALL，按 F8 跟进。 |
:004D20AE 84C0            test al, al                       | 经典语句！
:004D20B0 743C            je 004D20EE                       _|
:004D20B2 8B55FC          mov edx, dword ptr [ebp-04]
```

跟进以后，有

```
* Referenced by a CALL at Addresses:
| :004D20A9  , :004D3AA3
|
:004C9A94 55              push ebp
:004C9A95 8BEC            mov ebp, esp
:004C9A97 83C4F4          add esp, FFFFFFF4
:004C9A9A 53              push ebx
:004C9A9B 33C9            xor ecx, ecx
:004C9A9D 894DF4          mov dword ptr [ebp-0C], ecx
:004C9AA0 8955F8          mov dword ptr [ebp-08], edx
:004C9AA3 8945FC          mov dword ptr [ebp-04], eax
:004C9AA6 8B45FC          mov eax, dword ptr [ebp-04]
:004C9AA9 E876A7F3FF      call 00404224
:004C9AAE 8B45F8          mov eax, dword ptr [ebp-08]
:004C9AB1 E86EA7F3FF      call 00404224
:004C9AB6 33C0            xor eax, eax
:004C9AB8 55              push ebp
:004C9AB9 68FD9A4C00      push 004C9AFD
:004C9ABE 64FF30          push dword ptr fs:[eax]
:004C9AC1 648920          mov dword ptr fs:[eax], esp
:004C9AC4 8D55F4          lea edx, dword ptr [ebp-0C]
:004C9AC7 8B45FC          mov eax, dword ptr [ebp-04]
:004C9ACA E83D000000      call 004C9B0C
:004C9ACF 8B45F4          mov eax, dword ptr [ebp-0C]  ←过此处可下 d eax 显示真的注册码
:004C9AD2 8B55F8          mov edx, dword ptr [ebp-08]  ←过此处可下 d edx 显示注册用户名
:004C9AD5 E8A6A6F3FF      call 00404180
:004C9ADA 7504            jne 004C9AE0
:004C9ADC B301            mov bl, 01
:004C9ADE EB02            jmp 004C9AE2
```

整理一下：

用户名: `Crack007`

注册码: `859158116`

结束。

希望大侠们能贴出注册码的生成过程，THANKS！  `4dd466d0e1e0cc123afbe504da2462ca`
