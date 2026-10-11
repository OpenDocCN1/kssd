
# RamDisk9x/Me V1.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50495.htm`](https://www.pediy.com/kssd/pediy05/pediy50495.htm)

另类爆破——`RamDisk9x/Me V1.5`

下载页面：[`www.skycn.com/soft/4649.html`](http://www.skycn.com/soft/4649.html)

软件大小：626 KB

软件语言：英文

软件类别：国外软件 / 共享版 / 内存工具

应用平台：`Win95/98/Me`

加入时间：2001-12-17 10:54:27

下载次数：1520

推荐等级：\*\*\*\*

**【软件简介】**：利用内存模拟技术产生和硬盘以及 1.44M/2.88M 软盘功能一模一样的虚拟硬盘/软盘机，可依个人电脑内存大小设定所模拟的硬盘空间大小，最大支持到 4GB。

**【软件限制】**：次数限制、`NAG`

**【作者声明】**：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`Windows Enabler`、`FI2.5`、`W32Dasm8.93` 黄金版

—————————————————————————————————

**【过 程】**：

主文件 `0RAMDSK9XME.EXE` 在 `C:\WINDOWS\SYSTEM` 目录下。无壳。

呵呵，`VB` 的东西。输入注册码时逐个检验，错误则“OK”不可用。

—————————————————————————————————

```
:0041699F FF15DC114000           Call dword ptr [004011DC]

:004169A5 663BFE               cmp di, si

:004169A8 741D                 je 004169C7

:004169AA BAA86A4000           mov edx, 00406AA8

:004169AF 8D4D88               lea ecx, dword ptr [ebp-78]

* Reference To: MSVBVM60.__vbaStrCopy, Ord:0000h

                       |

:004169B2 FF1560114000         Call dword ptr [00401160]

:004169B8 8B03                 mov eax, dword ptr [ebx]

:004169BA 8D4D88               lea ecx, dword ptr [ebp-78]

:004169BD 51                   push ecx

:004169BE 53                   push ebx

:004169BF FF9014070000         call dword ptr [eax+00000714]

:004169C5 EB1B                jmp 004169E2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004169A8(C)

|

* Possible StringData Ref from Code Obj ->"YYour email must contain an "@""

                       |

:004169C7 BAAC704000           mov edx, 004070AC

:004169CC 8D4D88               lea ecx, dword ptr [ebp-78]

* Reference To: MSVBVM60.__vbaStrCopy, Ord:0000h

                       |

:004169CF FF1560114000         Call dword ptr [00401160]

:004169D5 8B13                 mov edx, dword ptr [ebx]

:004169D7 8D4588               lea eax, dword ptr [ebp-78]

:004169DA 50                   push eax

:004169DB 53                   push ebx

:004169DC FF9214070000         call dword ptr [edx+00000714]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004169C5(U)

|

:004169E2 8D4D88               lea ecx, dword ptr [ebp-78]

* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h

                       |

:004169E5 FF15E0114000         Call dword ptr [004011E0]

:004169EB 663BFE               cmp di, si

:004169EE 0F8425030000         je 00416D19

                       ====>不跳则 OVER！改为：JMP

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

爆破①：`0F8425030000`  改为：`E92603000090`（补 1 个 `NOP`）

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

…… …… 省 略 …… ……

* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h

                       |

:00416D19 8B35BC114000         mov esi, dword ptr [004011BC]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00416CEF(U), :00416D17(U)

|

:00416D1F 6685FF               test di, di

:00416D22 0F84D4010000         je 00416EFC

                       ====>不跳则 OVER！改为：JMP

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

爆破②：`0F84D4010000`  改为：`E9D501000090`（补 1 个 `NOP`）

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
```

改动以上 2 处后，“OK”已经 `Enable` 了。再次启动没有 `NAG`。

—————————————————————————————————

**【另 类 破 解】**：

上面是 爆破 ，还有一种比较简便而且有效的方法，这才是我这篇小文的主角呀。

主角：`Windows Enabler`

运行 `enabler.exe`，选上：`EnableWindows`

然后运行目标程序：`RAMDSK9XME.EXE`

输入 `E-mail`：`fly4099@sohu.com`

随意输入 `Password`:`13572468`

鼠标移动到“OK”上，呵呵，怎么样？“OK”按纽早已 `Enable` 了吧？！

点“OK”，进入程序。再重新运行 `RAMDSK9XME.EXE`，OK 了！`NAG` 不见了。

呵呵，程序已把注册标志信息写入了注册表中了！^O^^O^

其实这个小工具对付这种限制的软件可谓是“手到病除”，例如：`杀毒软件`就可以搞定。谢谢作者：`deuce`

虽然简单方便，但是对于想不断提高 `PJ` 水平的 `Cracker` 却没必要常用，毕竟我们是为了不断学习新的知识，而不是仅仅为了破解软件的。

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

**【软 件 说 明】**：

```
Author: deuce
Target: n0va's crackme v3

The hardest part of this crackme is enabling the window controls. Once you accomplish this, the rest is cake

There may be other ways to do this, but I decided to write a little program that does it simply by enumerating all child windows of the foreground window and sending WM_ENABLE message to each.  Examine enabler.asm to see exactly how this is done in assembly (to use enabler.exe, check "EnableWindows" and then click the parent window that contains the child window you want to enable).

This is coded in VB, so load up the crackme in SmartCheck, use enabler.exe, then type in a test serial and click OK.  By looking through the events under "_Click" you will see a call to vbaStrCmp.  Click on it and you will see the hard-coded serial

Later, Deuce
```

—————————————————————————————————

**【注册信息保存】**：

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\Microsoft\7R]

"ID"="fly4099@sohu.com"

"Password"="7174602E2B25245E6C4F49570D474A4B6A61676F"
```

—————————————————————————————————

Cracked By 巢水工作坊——fly【`OCN`】

2003-03-18 21:30:25  `4dd466d0e1e0cc123afbe504da2462ca`
