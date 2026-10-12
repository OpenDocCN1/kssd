
# Dll2Lib 3.00 脱壳后修复及优化 (Armadillo 4.66)

> 原文：[`www.pediy.com/kssd/pediy10/67619.html`](https://www.pediy.com/kssd/pediy10/67619.html)

**【破文标题】** Dll2Lib 3.00 脱壳后修复及优化 (Armadillo 4.66)

**【破文作者】** Ptero

**【破解工具】** `Ollydbg`, `PEiD`, `LordPE`, `ImportREC`, `FixRes`, `Armadillo Find Protect 1.8`, `Armadillo Process Detach 1.3`, `dilloDIE 1.6`, `ArmInline 0.96`

**【注册方式】** 未知

**【加壳方式】** `Armadillo 4.66`, `Debug-Blocker` + `CopyMem-II` + `Import Table Elimination` + `Strategic Code Splicing` + `Nanomites Processing` + `Memory-Patching Protections`

**【软件限制】** 30 天试用

**【下载地址】** `http://www.binary-soft.com/dll2lib/d2l.exe`

**【作者声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教。

---

## 一、查壳

Dll2Lib 就不用介绍了吧，看名字就知道是什么软件了。今年刚出了 3.00 版本，支持 Vista 系统。

首先用 `PEiD` 扫描，显示：`Armadillo 3.78 - 4.xx -> Silicon Realms Toolworks`。

Arm 的壳。再用 `Armadillo Process Detach` 查看一下：

```
!- Protected Armadillo

Protection system (Professional)

!- <Protection Options>

Debug-Blocker

CopyMem-II

Enable Import Table Elimination

Enable Strategic Code Splicing

Enable Nanomites Processing

Enable Memory-Patching Protections

!- <Backup Key Options>

Variable Backup Keys

!- <Compression Options>

Minimal/Fastest Compression

!- <Other Options>

45CBB980 Version 4.66 09-02-2007
```

用脱壳机 `dilloDIE` 试脱一下，因为 `dilloDIE` 不能正确修复 `Arm` 4.66 版的一些 `CC`，脱壳后的程序无法运行。

但是 `dilloDIE` 还是帮我们完成了很多工作。它修复了绝大部分的 `IAT`。下面我们要用到。

## 二、直飞 OEP

到达 `OEP`，前辈提供了很多方法。这里还是简单点，使用工具。

在 `Armadillo Process Detach` 当中选上 `CopyMem-II`， 然后载入目标程序。 显示：

```
Parent process iD:  [000020C8]

Processing...

[PROTECTiON SYSTEM]

Professional Edition

[PROTECTiON OPTiONS]

Debug-Blocker protection detected

CopyMem-II protection detected

Memory-Patching Protections enabled

Strategic Code Splicing enabled

Import Table Elimination enabled

Nanomites Processing enabled

[CHiLD iNFO]

Crypto call found:  [00499D42]

Child process iD:   [00000DE4]

Entry point:        [00427606]

Original bytes:     [558BEC6A]

Detached successfully :)
```

用 `OD` 附加 `00000DE4` 进程，`F9` 之后再 `F12`， 停在 `OEP`。 将 `OEP` 前 4 个字节改回到 `558BEC6A` 即可。

## 三、修复 Code Splicing

打开 `ArmInline`，选择进程 `00000DE4`，它会自动分析出 `Code Splicing` 的地址和大小，直接 `Remove Splices` 即可。

## 四、修复 IAT

这部分是比较麻烦的了。

![](img/9694b3ad9d249357d001b654196b989e.jpg)

首先，`ArmInline` 没有给对 `IAT` 的大小，手动查找一下很容易得出正确的大小是 `A38`。

新建 `IAT` 的存放位置嘛，还是在 `.rdata` 段里面找 `KERNEL32.DLL` 后面的大段空白。找到 `0045D250`：

```
0045D230  00 00 00 00 00 00 00 00 00 00 00 00 4B 45 52 4E  ............KERN
0045D240  45 4C 33 32 2E 64 6C 6C 00 00 00 00 00 00 00 00  EL32.dll........
0045D250  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

修改好参数以后，点 `Rebase IAT` 会跳出下面的对话框：

![](img/e3370ea72400373f6097bd951cb4b587.jpg)

这是因为 `ArmInline` 只支持到 4.4 版的 `Arm`， 4.66 版 `Arm` 有很多函数它不能识别。只有手动修复了。

在数据窗口按 `Ctrl+G` 来到 `156E6B8`， 也就是 `IAT` 开始的地方。按 `Ctrl+B` 搜索 `43713700`， 找到在这里：

```
0156EB08  43 71 37 00 FF C5 37 76 1C 7D 37 00 C2 59 E7 75  Cq7.?v}7.
```

然后按 `Ctrl+R` 显示调用窗口：

参考位于 `Dll2Lib:.text` 到 `0156EB08..0156EB0B`

```
地址       反汇编                                 注释

0041A211   call dword ptr [156EB08]     ds:[0156EB08]=00377143
0041DB3B   mov esi,dword ptr [156EB08]   ds:[0156EB08]=00377143
0042F423   call dword ptr [156EB08]     ds:[0156EB08]=00377143
004305D0   mov esi,dword ptr [156EB08]   ds:[0156EB08]=00377143
004366C0   call dword ptr [156EB08]     ds:[0156EB08]=00377143
0043CA21   call dword ptr [156EB08]     ds:[0156EB08]=00377143
00444FC9   call dword ptr [156EB08]     ds:[0156EB08]=00377143
```

现在在另一个 `OD` 中打开 `dilloDIE` 脱壳后的半成品，定位到 `0041A211`：

```
0041A211    FF15 04D24400   call dword ptr [&lt;KERNEL32.GetProcAddress&gt;]
```

呵呵，很明显了。右击这行，在数据窗口中跟随内存地址，二进制复制，再回到另一个 `OD`，在数据窗口 `0156EB08` 的位置二进制粘贴。这样 `00377143` 的这个 `Call` 就修复好了。

然后继续用 `ArmInline` 修复 `IAT`，还是会报错，按照类似方法一一修复。大概有几十个 `Call` 要修复。大部分都是像这样查找复制粘贴，比较简单。有极个别的 `dilloDIE` 也不能正确识别的，就需要手动跟踪一下了。

所有 `Call` 修复好以后，再用 `ArmInline` 重建 `IAT`。这时就可以用 `LordPE` 把进程 `dump` 下来了。

## 五、PE 文件修复及优化

将 `dump` 下来的文件复制一份，下面拿它开刀。

![](img/9dcd10e078154071733bb949c8313080.jpg)

将入口点修正为 `27606`， 文件对齐修正为 `200`。

![](img/0b33bbe70948769518fbc2292827aad6.jpg)

去掉 `.text1` `.adata` `.data1` `.pdata` `.rsrc` 区段。

在 `LordPE` 选项当中选中重新排列文件、清除重定位表、使 `PE` 有效，然后重建 `PE`。

编辑修改后文件的 `.data` 区段，修正其 `VSize` 大小为 `0000B000`， 不然 `ArmInline` 不能正确修复。

用 `ArmInline` 修复 `Nanomites`，修复刚才修改的 `PE` 文件。

用 `ImportREC` 重建输入表。

![](img/d252ff85ad6b20ab9fc006c1e3544583.jpg)

用 `LordPE` 打开重建后的文件，修改 `.mactk` 段 `RSize` 为 `1C00`，然后截去段尾，再修正 `RSize` 为 `1AB2` （这个数值是 `ImportREC` 给出的）。

计算一下下一个段的 `RVA` = `6E000` + `2000` = `70000`。

用 `FixRes` 抓取 `dump` 下的原始文件的资源， `RVA` 填 `70000`， 文件对齐填 `200`。

用 `LordPE` 载入资源区段，修复一下目录表。

![](img/dfd8d9ab9bc1d2a9b6e0acaac48906d4.jpg)

将入口点改到 `6A070` （`.nano` 段 `+70`），然后在 `LordPE` 选项当中选择使 `PE` 文件有效（万万不可选中重新排列 `PE` 文件！），然后重建 `PE`。

现在可以在 `XP` 上正常运行了，但在 `Vista` 上会出错。`Dll2Lib` 3.00 可是支持 `Vista` 的呀！我们来修复一下。

这里的代码是为了获取 `GetProcAddress` 的地址，在 `Vista` 上会出错。

```
0046A097  |.  33ED          xor ebp,ebp
0046A099  |.  8B58 3C       mov ebx,dword ptr [eax+3C]
0046A09C  |.  03D8          add ebx,eax
0046A09E  |.  8B53 78       mov edx,dword ptr [ebx+78]
0046A0A1  |.  03D0          add edx,eax
0046A0A3  |.  5B            pop ebx
0046A0A4  |.  8B4A 20       mov ecx,dword ptr [edx+20]
0046A0A7  |.  03C8          add ecx,eax
0046A0A9  |.  8B72 1C       mov esi,dword ptr [edx+1C]
0046A0AC  |.  03F0          add esi,eax
0046A0AE  |>  8B39          /mov edi,dword ptr [ecx]
0046A0B0  |.  03F8          |add edi,eax
0046A0B2  |.  8B7F 04       |mov edi,dword ptr [edi+4]
0046A0B5  |.  3B7B 20       |cmp edi,dword ptr [ebx+20]
0046A0B8  |.  74 08         |je short Dll2Lib.0046A0C2
0046A0BA  |.  83C1 04       |add ecx,4
0046A0BD  |.  83C5 04       |add ebp,4
0046A0C0  |.^ EB EC         \jmp short Dll2Lib.0046A0AE
0046A0C2  |>  03F5          add esi,ebp
0046A0C4  |.  8B36          mov esi,dword ptr [esi]
0046A0C6  |.  03F0          add esi,eax
```

我们绕过去自己写一个：

```
0046A097      5B            pop ebx
0046A098      8B35 F8D34500 mov esi,dword ptr [45D3F8]    ； 这里放的是 GetProcAddress 的地址，我们直接拿来用
0046A09E      EB 28         jmp short Dll2Lib.0046A0C8
```

修复后的文件 913 KB，比脱壳前的文件还要小。用 `PEiD` 查一下壳，显示： `Microsoft Visual C++ 6.0`。

到此修复完毕！在 `XP` 和 `Vista` 平台上通过测试，可以正常运行！

原本以为已经完美修复了，后来又发现一点问题。`ArmInline` 对一些 `CC` 断点修复错误，还要手动修复一下才行。

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢！
