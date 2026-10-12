
# MagicTweak

> 原文：[`www.pediy.com/kssd/pediy04/kanxue329.htm`](https://www.pediy.com/kssd/pediy04/kanxue329.htm)

**软件简介**:

`MagicTweak` 是一个系统优化和设置工具，可以更改上百种 `Windows` 隐藏设定，让你也变成了 `Windows` 修改设置专家。

**下载地址**: `http://nj.onlinedown.net/MAGICTWEAK.HTM`

**破解工具**:
* `Trw2000`
* `Filemon`
* `PEiD`
* `冲击波 V0.2`

**过程**:

`软件启动时会弹出一个注册框，让你注册。输入假的用户名和注册码，都会恭喜你注册成功，点击确定退出软件。`

`用 Filemon 查看软件启动过程，发现读取了 C:\Windows\mgwin.ini 这个文件，打开该文件，看到假的用户名和注册码。初步判断软件是启动过程时比较注册码。(后来用以下的断点均不成功，怀疑是错的.55555555)`

`用 PEiD 查看软件是用 PECompact 1.66 -> 1.67 加壳的，冲击波找到软件的入口点 00439292，用 Trw2000 载入脱壳。`

**下断点**:

```
bpx ReadFile
bpx WriteFile
bpx CreateFile
bpx SetFilePointer
bpx GetSystemDirectory
bpx GetPrivateProfileString
bpx GetPrivateProfileInt
bpx WritePrivateProfileString
bpx WritePrivateProfileInt
bpx findfirstfilea
```

不是不成功就是在代码堆里打转，看到一片的云山雾海。索性破罐子破摔，下 `Messageboxa`，看看能不能跟到些什么。点击 `Register` 拦住了：

```
167:BFF541B7 RET 04
USER32!MessageBoxA
167:BFF541BA PUSH EBP <=断点停在这里
167:BFF541BB EBP,ESP
167:BFF541BD BYTE +OO
167:BFF541BF DWORD[EBP+14]
167:BFF541C2 DWORD[EBP+10]
167:BFF541C5 DWORD[EBP+0C]
167:BFF541C8 DWORD[EBP+08]
167:BFF541CB CALL USER32!MessageBoxA
167:BFF541D0 POP EBP
167:BFF541D1 RET 10
```

一个 `Pmodule` 弹出假的注册成功对话框，点击确定，再次拦下:

```
:0044BAF3 8B442408 mov eax, dword ptr [esp+08]<=在这里重新设置断点，拦下
:0044BAF7 56 push esi <=在这里用 Alt+PaDn 查看，一大堆注册码
:0044BAF8 85C0 test eax, eax
:0044BAFA 8BF1 mov esi, ecx
:0044BAFC 7508 jne 0044BB06
:0044BAFE E842EE0000 call 0045A945
:0044BB03 8B4010 mov eax, dword ptr [eax+10]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0044BAFC(C)

:0044BB06 85F6 test esi, esi
:0044BB08 7504 jne 0044BB0E
:0044BB0A 33C9 xor ecx, ecx
:0044BB0C EB03 jmp 0044BB11

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0044BB08(C)

:0044BB0E 8B4E1C mov ecx, dword ptr [esi+1C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0044BB0C(U)

:0044BB11 FF742410 push [esp+10]
:0044BB15 50 push eax
:0044BB16 FF742410 push [esp+10]
:0044BB1A 51 push ecx
:0044BB1B FF15DC544600 call dword ptr [004654DC]
:0044BB21 5E pop esi <= 断在这里
```

`F6 + PaUp` 在 `0044BAF3` 处用 `F9` 重新设置断点。启动软件注册，断下了。用 `Alt+PaDn`，哇，软件贼大方，注册码居然有 `N` 多个，而且还是万能注册码（跟用户名无关）!!!

**童言无忌**：

非常感谢看雪老师为了这个论坛，让我学到了很多东西。因为太早的走出校门，接触到计算机也不过是一年多的时间。在购买了《加密与解密》后，居然发现自己连教程里最简单的加壳软件，一个都脱不了。或许这跟我是个"汇编瞎子"有关吧。

我所破的软件（其实也没几个），大多数是明码比较或照着葫芦画瓢（跟着别人的教程破的）。这其中有 99% 的运气和 1% 的耐心。所以能不能请看雪老师，再为我们这些"汇编瞎子"出版一本各个破解工具使用的详细教程呢。 `4dd466d0e1e0cc123afbe504da2462ca`
