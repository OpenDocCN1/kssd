
# CDR Inspector v1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum735.htm`](https://www.pediy.com/kssd/pediy03/forum735.htm)

## 前言

今天在朋友的软磨硬泡（电话骚扰）、威逼（绝交警告）利诱（请客答谢）下，我不得不答应帮他破解这个软件。由于我工作特别忙，开始我还很担心软件采用“马拉松”式的保护机制（典型的代表为“SmyNes”），着手后才发现软件的作者简直太善良了;-)

这个软件的破解过程十分简单，我希望破解初学者更多地注意教程中的“★★★ 破解经验提示 ★★★”，那里面大多是一些总结性的文字，对初学者迅速进入破解大门多少会有些帮助的;-)

## 正文

软件名称：`CDR Inspector v1.0`

下载地址：`http://www.ocloud.com/`

简单介绍：`CDR Inspector` 可以读出 CDR 盘片信息，自动识别生产商，并可读出 CDR 中记录的有关记录信息，如容量、介质类型、支持的刻录速度等，让你对你购卖的 CDR 心中有数，同时可以读取光驱设备性能参数。

作者：`Sun Bird [CCG]`

工具：`FileInfo v2.43`、`DeASPack v2.11`、`TRW2000 v1.22`

日期：2001 年 3 月 23 日

### 破解过程

(1) 使用 `FileInfo v2.43` 检测一下软件主程序“`CDRInspector.exe`”，果然有料，编程语言是 `Delphi`，还加了壳，是 `ASPack v2.11` :-(

#### ★★★ 破解经验提示 ★★★

现在的软件为了防破解大多经过了加压或加壳，而且目前很多外壳软件均防常见的破解工具。所以在破解前应首先使用文件外壳检测软件检测一下主程序，以免跟踪时在软件外壳部分转来转去浪费时间和精力或者直接被软件外壳搞死:-( 这可是破解初学者应该养成的良好习惯^\_^

推荐使用 `FileInfo v2.43`、`Get Type v2.60`

(2) 使用 `DeASPack v2.11` 将主程序“`CDRInspector.exe`”脱壳（注意 `DeASPack` 默认将脱壳后的程序命名为“`out.exe`”）。

#### ★★★ 破解经验提示 ★★★

要想在破解过程中不“受制于人”，那么必须掌握手动脱壳技术！现在手动脱壳的极品工具组合为 `BlastWave2000 v0.2` 和 `TRW2000 v1.22` （向 `Ding Boy`，`刘涛涛`、`朱南灏` 大虾致敬！）

常有初学者提问“如何脱 `XXXX` 壳”。在此建议初学者，既然在手动脱壳方面比较“手潮”，那么一定要密切关注各类破解工具站点，这些站点会及时更新各种专用解压脱壳工具。

推荐站点：
`http://www.exetools.com/`
`http://www.suddendischarge.com/`
`http://protools.cjb.net/`

(3) 运行“`out.exe`”，嘣－－“`Unregistered version. Please register.`”（由于软件没有找到“`out.CHS`”，所以显示英文提示信息，这样对于破解非常有利），不用考虑了，点“确定”吧;-)

#### ★★★ 破解经验提示 ★★★

`TRW2000`、`SoftICE` 或者标准版的 `W32Dasm` 等常用破解工具均不支持汉字，而我们在破解国产软件时又经常需要查找提示信息，常有初学者对此一筹莫展:-( 其实为了方便破解，我们可以使用 `UltraEDIT`、`WinHex` 等编辑软件将汉字提示信息更改为英文，这样常用破解工具就可以使用了^\_^

(4) 进入软件主界面，点“`About`”，在 `About` 窗口点“`Register Now`”，在 `Register` 窗口输入相关信息：

`Username`：`Sun Bird [CCG]`
`Registration Code`：`31415926`

#### ★★★ 破解经验提示 ★★★

现在很多软件在防破解方面下了很大的功夫，有的软件发现输入的注册码为“`12345678`”、“`78787878`”或者“`88888888`”等简单重复时会直接使注册失败，根本不去判断注册名和注册码之间的逻辑关系；有的软件则对注册码采用实时判断，这样如果输入简单重复的注册码在破解时会郁闷之极，因为很难判断现在正确的注册码对应的是第几个“`8`”:-(

使用圆周率或者自己熟悉的字符串（例如身份证号码、出生年月日等）做为假注册码，可以避免类似的麻烦^\_^

(5) 运行 `TRW2000 v1.22` ，按 `Ctrl-N` 进入 `TRW2000`

```
BPX HMEMCPY <--- 下“万能”断点
G <--- 继续运行（即回到 out）
```

#### ★★★ 破解经验提示 ★★★

前面我们已经提到过，现在很多软件反破解工具，有些软件在启动后检测到有 `SoftICE`、`TRW2000` 甚至 `天意 II v0.46` 这类破解工具时拒绝继续运行直接退出，而有些恶毒的软件则干脆导致死机:-(

在软件启动后再运行 `TRW2000` 可以避免部分软件对破解工具的检测^\_^

(6) 在 `Register` 窗口点“`OK`”，马上被 `TRW2000` 中断

```
BC * <--- 清除断点
PMODULE <--- 回到 out 领空
```

(7) 按 `F12` 7 次、`F10` 1 次后来到这里：

```
:0045DEEA 8B45E4 mov eax, dword ptr [ebp-1C] <--- 光棒停在这里
:0045DEED 8D55E8 lea edx, dword ptr [ebp-18]
:0045DEF0 E8FF020000 call 0045E1F4
:0045DEF5 8B45E8 mov eax, dword ptr [ebp-18]
:0045DEF8 50 push eax
:0045DEF9 8D55D4 lea edx, dword ptr [ebp-2C]
:0045DEFC 8B45FC mov eax, dword ptr [ebp-04]
:0045DEFF E820010000 call 0045E024
:0045DF04 8D45D4 lea eax, dword ptr [ebp-2C]
:0045DF07 8D4DF4 lea ecx, dword ptr [ebp-0C]
:0045DF0A 5A pop edx
:0045DF0B E8ECF5FFFF call 0045D4FC
:0045DF10 8D4DF0 lea ecx, dword ptr [ebp-10]
:0045DF13 8D45F4 lea eax, dword ptr [ebp-0C]
:0045DF16 BA08000000 mov edx, 00000008
:0045DF1B E8A0F0FFFF call 0045CFC0
:0045DF20 8D55D0 lea edx, dword ptr [ebp-30]
:0045DF23 8B45FC mov eax, dword ptr [ebp-04]
:0045DF26 8B80E0020000 mov eax, dword ptr [eax+000002E0]
:0045DF2C E8D7ACFCFF call 00428C08
:0045DF31 8B55D0 mov edx, dword ptr [ebp-30]
:0045DF34 8B45F0 mov eax, dword ptr [ebp-10]
:0045DF37 E8A85EFAFF call 00403DE4 <--- 关键 Call，需要 F8 跟进
:0045DF3C 7541 jne 0045DF7F
:0045DF3E 8D55CC lea edx, dword ptr [ebp-34]
:0045DF41 8B45FC mov eax, dword ptr [ebp-04]
:0045DF44 8B80E0020000 mov eax, dword ptr [eax+000002E0]
:0045DF4A E8B9ACFCFF call 00428C08
:0045DF4F 8B45CC mov eax, dword ptr [ebp-34]
:0045DF52 50 push eax
:0045DF53 8D55C8 lea edx, dword ptr [ebp-38]
:0045DF56 8B45FC mov eax, dword ptr [ebp-04]
:0045DF59 8B80D8020000 mov eax, dword ptr [eax+000002D8]
:0045DF5F E8A4ACFCFF call 00428C08
:0045DF64 8B55C8 mov edx, dword ptr [ebp-38]
:0045DF67 8B45FC mov eax, dword ptr [ebp-04]
:0045DF6A 59 pop ecx
:0045DF6B E854010000 call 0045E0C4
:0045DF70 8B45FC mov eax, dword ptr [ebp-04]
:0045DF73 C7803402000001000000 mov dword ptr [ebx+00000234], 00000001
:0045DF7D EB57 jmp 0045DFD6
```

(8) 按 `F8` 跟进关键 `Call`：

```
:00403DE4 53 push ebx
:00403DE5 56 push esi
:00403DE6 57 push edi
:00403DE7 89C6 mov esi, eax
:00403DE9 89D7 mov edi, edx
:00403DEB 39D0 cmp eax, edx <--- 典型比较
:00403DED 0F848F000000 je 00403E82
:00403DF3 85F6 test esi, esi
:00403DF5 7468 je 00403E5F
:00403DF7 85FF test edi, edi
:00403DF9 746B je 00403E66
:00403DFB 8B46FC mov eax, dword ptr [esi-04]
:00403DFE 8B57FC mov edx, dword ptr [edi-04]
:00403E01 29D0 sub eax, edx
:00403E03 7702 ja 00403E07
:00403E05 01C2 add edx, eax
```

当光棒停在 `:00403DEB cmp eax, edx` 处时，`D EAX` 可得到正确的注册码、`D EDX` 可得到输入的假注册码。

其实当光棒停在第 (7) 步 `:0045DF37 call 00403DE4` 处时，即在按 `F8` 跟进关键 `Call` 之前，`D EAX` 已经可以得到正确的注册码、`D EDX` 可得到输入的假注册码。

#### ★★★ 破解经验提示 ★★★

常有初学者提问“怎么才能知道需要按 `F12` 多少次”。其实通常情况是，当按了 `F12` N+1 次后软件就提示注册码错误的信息了，第一反应自然是“注册码判断的过程可能在 `F12` N 次后的部分”，所以下次就少按 1 次 `F12` 喽。

在什么时候需要 `F8` 跟进 `Call`、什么时候使用 `D`、？ 命令则需要有一定的破解经验了。对于破解经验比较少的初学者可以采用最原始的方法，当碰到对寄存器进行操作的命令后就 `D` 或 ？ 一下、碰到 `Call` 就跟进去、碰到比较命令就对比较的寄存器 `D` 或 ？ 一下……一定要勤记笔记！破解时一定会失败若干次，通常是没有跟进某个 `Call` 或者是对某个判断、跳转没有注意，那么下次可以对照笔记快速定位，否则每次都要从头来过可就太惨了:-(

每一次破解都是一次经验的积累，当破解经验越来越丰富的时候，自然而然会对某些命令比较敏感，直觉上认为这些 `Call`、判断、跳转一定有问题，而事实往往证明这些就是关键点。因为软件的保护机制是有规律可循的，所以有经验的破解者能够比较迅速的找到关键点。

这些直觉是任何书本、教程都无法讲清楚的，所谓“只可意会，不可言传”。要想获得这些直觉，没有任何捷径，只有不停地实践！

### 注册信息

注册名：`Sun Bird [CCG]`
注册码：`AF40E7648DD90464`

现在可以运行原版软件“`CDRInspector.exe`”进行注册了（软件的注册信息存储在注册表“`HKEY_CURRENT_USER\Software\OCloud\CDR Inspector`”处）。

## 后记

我用了 5 分钟左右的时间破解，而写这篇教程却断断续续花费了大约 NN 个小时，其间包括完成本职工作、外出开会、陪客户吃饭、加班以及和老板“谈心”……搞得我十分郁闷:-(

还有，今天早上在“白菜乐园”处看到已经有 `CCG` 成员提供了 `CDR Inspector v1.0` 的注册码^\_^ `4dd466d0e1e0cc123afbe504da2462ca`
