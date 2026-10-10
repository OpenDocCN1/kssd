
# 献给新手(一)--不用脱壳就能调试加壳软件(SoftICE 还旧之旅)

> 原文：`www.pediy.com/kssd/pediy12/127562.html`

**【原创】献给新手(一)--不用脱壳就能调试加壳软件(SoftICE 还旧之旅)**

於版,我己不能在新人交流投稿发贴,但贴是要新手入看的,因是自己加入论坛,搞不清楚,所以就先到了『求助问答区』,但因不是真的悬赏贴,所以发得心理怕怕的,在发

『伴你成长』园地,就快把之前的贴移到,不是要灌水重发贴,先前不懂版的地方,版主及各位雪友多多原!!

【作者】`ninymay`

【说明】我是,了一篇於 `OCX` 控件的破解文章

(`bbs.pediy.com/showthread.php?p=905878#post905878`)

除了得版主了邀请码外,也勾起大家的,坐光机回到 `SoftICE` 的年代,雪友 `whypro` 希望我再多篇,所以我就抱著回大家的心了,我水平不高,的不好多多包涵![](img/8665dab461df9ff5e20e52adf9adf6de.jpg "伤心")

老,`OllyDBG` 真的太好用了,不用安,什作系都能 `Run`,界面又漂亮,插件多到翻...不用它真是笨蛋咧![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

但如果你是初软件调试的新手,恐怕 `OllyDBG` 的美好光很快就去了,因它有很多事情不是您想的那美好,在扒一下文就知道,新手的汇编能力不好,遇到加壳软件,如果有脱壳机或脱壳插件、脱壳本...,那程序入跟不了多久、看不了多少机就放了,即使不放也是白做工,去不知道要做什？所以老得破解入，调试软件真不是人的...![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

如果新人先 `SoftICE` 下手就不一了,它然不美,而且很挑作系,最重要是得一步一步去安...然,但起,上都有明:

`www.pediy.com/tools/Debuggers...CE/SoftICE.htm`

`bbs.pediy.com/showthread.php?t=85389`

如果有,我也把我的安心得再上...

一旦起,了,您就了:

您不用先去付的 `PE` 格式、`IAT` 表格、不用去管什演算法、加解密法，您可以先比人性的 `Windows API` 下手,配合常用的机,就能有小成就,什呢？因:

『`SoftICE` 可以加壳软件直接下 `Windows API`』(比如框出用的 `MessageBox...`),不用先脱壳,就能在要害的地方,段乎有程式可以『完全』不用 `Windows API`,只要用 `API`, `SoftICE` 就能您下,而且不管再害的壳它是得行,一旦行起,它自己得先脱壳,您就等著它自投就好了

接下您就可以好好分析了,才有趣,才不把新手跑嘛>\_<,而且又回,如果用 `SoftICE` 中在的地方都法往下分析,那就算完壳你,不是一看不懂,所以尤其是您要付的是『有壳的意程序』,重是要分析意程序,而不是壳,

如果可以不壳就分析,就不用多一道手了不![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

好了,不多,才是真的,先好了,是新手入的,大牛千挑毛病啊!!有 `OllyDBG` 有不好,`OllyDBG` 的死忠者,千不要找我,我只是希望借 `SoftICE` 的方便,

更多的新手入之

【破解等】入

【使用工具】`SoftICE`(`Driver Studio V3.2`)

【破解平台】`VM6.04/XP SP2`

【软件名称】`RenameRegKey UPX.exe MyReg1.reg`

【下载地址】`RenameRegKey.rar`

【软件简介】非常的程序,供用,有 `Anti-Debug`,不是否在 `VM` 下行...

【软件特性】`Windows` 登(`Registry`)做修改的作

【软件大小】`184KB`(解後)

【保护方式】`UPX 2.92b`--最的壳,丈高平地起,一步一步慢慢

【破解目的】用 `SoftICE` 在未壳的情下就可以分析程序是如何修改登加以阻止,

以此了解存取登的相 `API`

【破解声明】本例程式如果您用壳机壳後,可以再用任何加壳方式加壳回去,都可以再用本篇

介的方式在未壳的情下成功下喔

【破解内容】

因是教程，所以也大家摸索了，更不必去放在沙盒中或什捕境中，我先告大家程式要嘛？

大家先用事本把 `MyReg1.reg` 打看看，

![](img/5437b3d1689c880d01b44b4892392151.jpg)

它在

`HKEY_LOCAL_MACHINE\SOFTWARE\`

新增一机名字叫做 `MyReg1`

且在 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`

新增一机叫做 “`Virus`” 机值 “`Virus.exe`”

我把事本，用滑鼠 `MyReg1.reg`，不要理系的警告，按下 “定” 机入登，如果您有安防毒程式

也它通，否後面法。

在打登程式(`RegEdit.EXE`)到 `HKEY_LOCAL_MACHINE\SOFTWARE\`

看到左方新增了一名叫 `MyReg1` 的机

![](img/256e25183a3eae4dd85c792eda3cc7a4.jpg)

再到 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`

看到右方多了一行 `Virus REG_SZ Virus.exe`

放在位置的程序是每次 `Windows` 机都行的,所以是意程式兵家必之地！

![](img/46cb5d39c69724d4fba6fc3d05beb3fb.jpg)

※我例程序是示用的,所以很友善,真的意程式就去除(不是改名而已)你重要的机,且在增加(而不是除)一机,以便下次 `Windows` 可以入它的意程序!!!!!

但透例,就可以了解登的 `API` 操作手法,日後可以通用![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

一切就就先行 `RenameRegKey UPX.exe` 程式

![](img/1608eb5e7a772f9f7dd3dab203c308cd.jpg)

我看到主面有大 `Button` 如果按下 `Button1` 看到一框告你 `MyReg1` 成 `MyReg2` 了

![](img/7155f38dcfc86f2834d31aa2a41c27a8.jpg)

再到登程式，按一下 `F5` 它重新整理，在再去看看 `HKEY_LOCAL_MACHINE\SOFTWARE\`下的 `MyReg1` 不了成 `MyReg2`

![](img/d88f45fcf20c36f6eae7c885bb6cd99a.jpg)

而按下 `Button2` 出值己清除完成的框，

![](img/05471587a91b083790f9499d58f134fd.jpg)

再到 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`

`Virus` 那一行值己不了不？

好了，在假你是初者不壳，但要付程序或是分析它的程序在哪要怎做呢？

如果你叫出 `OllyDBG` 假是全新的壳或是壳有插件可以忙壳，那入 `RenameRegKey UPX.exe` 程式就出警告息

![](img/992f62bdb943eecff51f12e07162183c.jpg)

不理它入，由於我已知道它是在登做手，所以我可以下 `RegOpenKeyA` 做中，在 `Windows` 要存取登都必需要靠 `API` 登,如果手有 `MSDN`,可以查一下它的原型宣告:

```c
LONG RegOpenKey(
  HKEY hKey,        // handle to open key 在本教程中用 HKEY_LOCAL_MACHINE
  LPCTSTR lpSubKey, // name of subkey to open
  PHKEY phkResult   // handle to open key
);
```

中有 `RegOpenKeyA`, `RegOpenKeyW`, `RegOpenKeyExA`, `RegOpenKeyExW` 四 `A` 是 For `ANSI`, `W` 是 For `Unicode` 版本

在本教程中用的是 `RegOpenKeyA`

在我在下方的 `Command` 入 `BPX RegOpenKeyA`

![](img/d8f8243cbfc313b7de2a63b1be0725e9.jpg)

出的框怪怪的？！不像是有效的中，因若是有效的中是出的框才

![](img/4bbd69f5f57a32e581ded34eaa63108f.jpg)

才算是承 `RegOpenKeyA` 是有效的中，但在先不管那多了，全速行下去看不得到就知道了！在 `OllyDBG` 全速行後，主面出了，按一下 `Button1` 果告你 `MyReg1` 成 `MyReg2` 的框出了，在壳的情下，`OllyDBG` 什也到？？

如果到的面是才啦！

![](img/7f50d859a82f6e9589434244fd1891c8.jpg)

如果不想慢慢的起,您可以有另外一

在 `SoftICE` 上了，先把 `VMware` 打，先 `SoftICE`，先到始程式集 `Compuware DriverStudioSettings` 叫出 `SoftICE` 的定程式在左 `SoftICE Initialzation` 中的 `Exports` 右方是不是有 `C:\WINDOWS\system32\advapi32.dll` 字，如果有要自己按下右的 “Add…”自行加入，才能使 `SoftICE` `RegOpenKeyA` `API` 等一下才能正入做中。

![](img/0ac1cbbca0d139b3c80faba889d1b72b.jpg)

在可以 `SoftICE` 了，因 `SoftICE` 是 `Ring 0` 的除工具，了避免等一下做任何作都 `SoftICE` 我最好先行 `RenameRegKey UPX.exe` 之後才去按 `Ctrl+D` 叫出 `SoftICE` 然後入 `BPX RegOpenKeyA` 了保起可以入 `BL` 指令已被接受

![](img/434713e8c00ed7da952d042adba663c9.jpg)

各位可以在截中看到 `RenameRegKey.exe` 已了，在按 `F5` `SoftICE` 先消失，之後就可以按下 `RenameRegKey.exe` 中的 `Button1` 了一下子 `SoftICE` 就跳起，它已到 `RenameRegKey UPX.exe` 登表的作了，

![](img/198bdf05d392c12c39d4ba9642827a77.jpg)

只是在在 `Advapi32.dll` 的空上，我可以中下方色那行字:

`advapi32!.text+0002B41A` 的字得知,

在我要按一下 `F12` 它可以回到

`RenameRegKey UPX.exe` 的空

![](img/b86666e21e835ca85132f3f2b7579563.jpg)

在回到 `RenameRegKey UPX.exe` 的空，往下看就可以知道程式是怎去更改登表或是除登表的：原登表不像案管那可以直接下 `Rename` 的指令去修改案或目名，至少有 `RegRenameKey` `API` 嘛！它必需先重新造一新的名，然後把原的容一份到新的中，最後再把原的掉。

考到新手可能械的不是熟，在是稍微一下追的程好了。我程式步行(`F10`)下去，要特注意分岐的地方，像第一分岐是 `4019D1` 的地方(以我的截，因不同的器上跑程式地址可能有所不同！)，`JZ 4019D8`，`SoftICE` 告我件

成立所以程序跳到 `4019D8`(`SoftICE` 有一往下的小箭提示)，在做了最重要的一件是就是呼叫 `RegCreateKeyA`，建立一名 `MyReg2` 的，之後到了第二分岐 `4019EE JZ 4019FD`，`SoftICE` 同告我件成立，所以跳到 `4019FD`，到了 `4019FD` 就去呼叫 `401A38`

![](img/205d48c0247becfc78b77fdab4508923.jpg)

是 `RegCreateKey` 的原型宣告

```c
LONG RegCreateKey(
  HKEY hKey,        // handle to an open key
  LPCTSTR lpSubKey, // subkey name
  PHKEY phkResult   // buffer for key handle
);
```

我按 `F8` 跟看看，一始看不到什 `API`，如果有趣可以自己分析看看，一直走到 `401A69 JNZ 401A73`，`SoftICE` 告我要跳，接下情就明朗了，程式先叫用了 `RegEnumValueA` 去列原 `MyReg1` 面的值，然後到 `401AB8 JNZ 401ADE` `SoftICE` 回不跳，不跳就了，下面才是重：

先叫用 `RegSetValueExA` 入和 `MyReg1` 列出一的值到 `MyReg2` 面，再呼叫

`RegDeleteValueA` 去掉 `MyReg1` 面的值，在 `401AE2` 的地方查 `MyReg1` 的机值都列完了？如果有就回跳到 `401A84` 重上面的作直到整 `MyReg1` 的西都到

`MyReg2` 面止。

是 `RegEnumKey` 的原型宣告

```c
LONG RegEnumKey(
  HKEY hKey,      // handle to key to query
  DWORD dwIndex,  // index of subkey to query
  LPTSTR lpName,  // buffer for subkey name
  DWORD cchName   // size of subkey name buffer
);
```

是 `RegSetValueEx` 的原型宣告

```c
LONG RegSetValueEx(
  HKEY hKey,           // handle to key
  LPCTSTR lpValueName, // value name
  DWORD Reserved,      // reserved
  DWORD dwType,        // value type
  CONST BYTE *lpData,  // value data
  DWORD cbData         // size of value data
);
```

是 `RegDeleteValue` 的原型宣告

```c
LONG RegDeleteValue(
  HKEY hKey,           // handle to key
  LPCTSTR lpValueName  // value name
);
```

![](img/ccb3578fa81e95a88e298afb7ad3bc9b.jpg)

在您是不是恍然大悟，`Windows` 成的 `API` 真的很重要不？熟悉了之後要分析意程式真的很方便，配合 `SoftICE` 就算不壳也可以截下

且把它分析得清清楚楚啊！

如果不放心，可以把在看到械便抄下然後去 `HexEdit` 或是 `UltraEdit` 把 `RenameRegKey UPX.exe` 入後搜看看，是不是根本找不到些械？所以可以明程序案仍是未壳的,但一旦行起被 `SoftICE` 下的是已壳乾的，如果有用保的，也保一定被原成 `x86` 的械了啦^^

如同一始的候的,本例程式如果您用壳机壳後,可以再用任何加壳方式加壳回去,都可以再用本篇介的方式在未壳的情下成功下分析喔!各位不妨![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

而再害的客不管怎小心，都很完全不用 `Windows API` 出一像的意程式，多是用冷的、低的 `API`(例如 `zwOpenkey...`以後有的文介)，且去 `Hide` 一些知名的追，再不就是 `Hook` 住 `API` `API` 先做自己想做的事情，做完了再去做原本 `API` 做的事情，而些都很逃 `SoftICE` 的法眼，上面的明和示相信者可以知道 `SoftICE` 的的有它到的地方是 `OllyDBG` 所不及的，在入可以新手一些助。

【後】

挑剔的者也到如果不壳，那在找到性的程式要修改，但是到了案中找又找不到，那又如何能修改呢？心，不壳不但可以分析也可以修改，才是完美的破解方案，期待下一篇:新手(二)--不用脱壳就能修改加壳软件(SoftICE 之旅)。

看完![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

# 献给新手(二)--不用脱壳就能修改加壳软件(SoftICE 还旧之旅)

**【原创】献新手(二)--不用脱壳就能修改加壳软件(SoftICE 还旧之旅)**

抱歉各位久等了,新年期,路不太,看雪的老不太起,所以至今日才第二篇,大家![](img/e06ce445d64470eaf3496ba167dfb4ac.jpg "碌眼")

在始第二篇的正:

挑剔的者也到如果不壳，那在找到性的程式要修改，但是到了案中找又找不到，那又如何能修改呢？心，不壳不但可以分析也可以修改，才是完美的破解方案。
