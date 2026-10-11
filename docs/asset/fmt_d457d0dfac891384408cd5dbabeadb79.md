
# 输入数据输入函数表之二：`.rdata` 节的输入函数

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-711.htm`](https://www.pediy.com/kssd/pediy07/pediy7-711.htm)

## `.rdata` 节的输入函数

（对上个篇输入函数文章的补充）

原文：caracol
译文：声声慢
致谢：谢谢 fly, tianwei 以及看雪的朋友们.

### 前言

离我开始收集材料为了补充上输入函数表的文章已经一个月了.

实际上, 在一篇文章, 我只是满足于指出: 不总是只有一个 `.idata` 节为了储存输入表数据信息.

在第一篇文章出炉之后, 我发现了一些朋友却遇到了加壳软件将输入函数储存在 `.rdata` 节里的疑难.

在这种情况下遇到的主要难题是: 哪些输入函数表的数值必须交给 `pe` `header`, 为了获得一个可执行的文件? 怎样去求取这些数值?

这也就是发布一篇关于输入表文章的续集主意来源, 也借同一机会, 将一些难点弄清楚, 或者, 用另一个角度来陈述一下同样的问题 (希望这次会更清楚一点)

为了以下的分析更为具象化, 我在这篇文章里增加了实践操作的部分. (两个加壳后, 输入表里包含 `.rdata` 节的程序: `ACDSEE3.0` 法文版和 `AVOLUME3.1`)

在很长的一段休息后, 我重整勇气 (特别我的时间) 去完成这篇文章.

我希望它能给大家带来一些益处.

好了, 开始!

## 在 `.idata` 和 `.rdata` 节里的函数表

大部分时候, 在一个执行程序里, 所有的关于输入函数表的信息都存放在一个命名为: `.idata` 的节里.

但是有些编程工具/linker/complilator 会将这些信息存放在一个名为 `.rdata` 的节里, 与其他数据混在一起.

我注意到那些将输入函数表存在 `.rdata` 里的通常是一些由 microsoftde 的工具 (c++, viiisual c++ 等...) 编写的程序.

无论我们遇到什么情况, 它的输入函数表的逻辑, 组织, 结构... 除了一些小的细节外, 基本上与包含有 `.idata` 节的程序一致.

### 总结重温一下在 `.idata` 节里, 在一个程序里的输入表的组织结构的信息

(`.idata` 节)

![](img/802aa974f8f50ce0e59c711fe054a056.jpg)

图 01.

当然, 要是您已经阅读过我的上一篇关于输入函数表的文章, 或者您对输入函数很熟悉, 您应该能认出 Microsoft 的类型!

我不再重提 Borland 的例子, 是一样的, 但比较简单: 没有符合输入表的那个块段的数据.

Microsoft 的类型比 Borland 的要复杂, 我们来讲解 Microsoft 的, 这样一来, 下次遇到 Borland 的, 那就是白送给您的礼物.

无论是在硬盘 (文件里) 上, 还是被 windows 的装载器装载在内存里, 这 4 个区域的内存, (image import descriptor, hint name array, import address table, DLL names 和 functions names) 构成一个的连续的块段.

这个内存的块段也就是 `.idata` 节.

当那些输入函数处于 `.rdata` 的节时, 我们将遇到这 4 个区域的内存, 但不一定以一样的顺序, 和处于连续的一个块段中.

在小结后, 我们来看一下.

### 对在 Image Import Descriptor 里, 每个 DLL 的入口格式的简单小结

在 image import descriptor 区域的每个入口符合一个 DLL

每个 DLL 占用 5 个 `Dword` (双词)

对我们来说, 最重要的 `Dword` 是:

Microsoft 类型: `Dword 1`, `Dword 4`, `Dword 5`

Borland 类型: `Dword 4`, `Dword 5`

![](img/8639f27e375c4d25552e6b79154156b5.jpg)

图 02.

(\*) Entrée pour la DLL n°1 (例子 : `KERNEL32.dll`)

(\*\*) Entrée pour la DLL n°2 (例子 : `SHELL32.dll`)

`Dword1`:

它指向表格的第一个 dword.

这个表包含 `Dword 4` 指向的 DLL 的所有的有关函数的指针.

这个表是 ‘Hint Name Array’ 内存块段的一部分.

`Dword 5`:

它指向表格的第一个 dword.

这个表, 包含在 ‘Hint Name Array‘ 里的函数的所有调用地址. 它与 `Hint Name Array` 并行.

这个表格是 ‘Import Address Table’ 内存块段的一部分.

在这, 最有用的是 `Dword 4`, 能帮助我们在 `.rdata` 节里找到我们的小朋友们 :

`Dword 4` 包含 DLL 明文名字的地址 (Raw Virtual Address).

比如:

符合 `KERNEL32.dll` 的入口.

假设 `dword 4` 含有‘8A 65 01 00’的值, 就是表示它指向’00 01 65 8A’的地址.

要是 `Raw Offset` 与 `Virtual Offset` 相等 (也就是说在硬盘上的地址与内存里的地址相符), 我们将在 offset ’00 01 65 8A’ (在文件) 找到‘KERNEL32.dll’的字串.

## 在 `.rdata` 节内, 输入函数组织的例子

注意, 这只是一个例子, 只是我有幸观察过几个包含 `.rdata` 节的程序后的结果.

因此, 我不敢担保所有的 `.rdata` 节都与此相同.

![](img/aff194ae99619570ab045c0ae258716a.jpg)

图 03.

您看到了吗? 除了 import address table 的块段现在处于 `.rdata` 节的开端, 跟在一段数据块后, 其它基本上是一样的.

唯一的盲点对我们来说是, `Image Import Descriptor` 的块不处在 rdata 节的开端.

因此, 当我们遇到象被 `ASPack` 或 `ASProtect` 加壳的程序, 并且我们成功的 `DUMP` 出脱壳程序, 我们还要用可靠快捷的方法成功辨认出这个块段 (地址的开端及其大小), 为了将这些信息交给 `PE header`.

由此, 我们可以得到一个可运行的程序, 并且具有输入函数表的反编代码.

## 怎样找回内存里包含 Image Import Descriptor 信息的区域, 当在 PE header 没有具备这些信息时

我们在上面看到过了, 最有用的是 `Dword 4`, 能帮助我们在 `.rdata` 节里找回我们的小朋友们.

事实上, 因为 `dword 4` 指向 DLL 名字的明文. 只要找出在 rdata 节 DLL 的名字 (跟在与之有关的函数名后), 记下 offset, 然后在 rdata 寻找与之相应的 offset 值.

一旦我们找到, 就是说在目录 image import descriptor 里.

剩下的就是判断出这个区域的开端和结尾.

`REGMON.EXE` 例子的节选

```
section rdata : 0000A000 - 0000C000

RVA Import table = A624

Size Import table = 008C
```

![](img/0c6a4f8962c1f93ebc9fb38c3dcdbfe8.jpg)

图 04.

‘KERNEL32.dll’在 offset ‘00 00 AA 22’ (文件里) 里找到.

这表示 image import descriptor, 应该有一个 `dword 4` 包含‘22AA 0000’的值, 符合 `KERNEL32.dll` 的入口.

只需要在 rdata 节里寻找这个 hexa 的值, 以进入 image import descriptor.

![](img/0d8c63859b2456b04c2acd32fac961da.jpg)

图 05.

从这个标记开始 (在 `0000A630`, 符合 `dword4` 目录表的一个入口), 生米已煮成熟饭!

剩下要做的就是在这个内存的区域内, 往上或往下移, 边数着这些 `dword` , 边看看它们的内容是否正确.

例如 : 我们看见 `dword1` 由需线及斜体表示, 含有在 `.rdata` 区域外的地址, 第一个有效的 `dword1` 处于: ‘0000A624’.

这是 image import descriptor 的开端.

image import descriptor 的结尾, 在视觉上, 能被连续的 ’00 00 00 00’ 所提示.

有时, 没有这些连续的’00 00 00 00’, 那么就要使用别的方法.

例如, 看到 `dword 4` 不包含有效的地址 (我们看 `0000A6A0` 的那行 !)

一旦我们辩认出这个内存区域的开端和结尾, 我们就可以计算它的大小.

这个区域的开头是在 ‘000A624’ ; 结尾在 ‘0000A6B0‘ 因而, 大小 = ‘0000008C’ (`A6B0` - `A624`)

这两个信息是用来放在 `PE directory information / Import Table / RVA & Size` 里的.

(我知道我在不断的在重复...)

## 这两个信息 (Image Import Descriptor 的开端和大小) 对 windows 的装载器来说足够了.

对, 对, 我向你保证!... 无须再指出 `Import Adress Table` 的值, 因为这些信息都包含在每个入口的 `dword1`.

另外, 我们可以用以下的方法测试一下:

借助 `PROCDUMP.EXE` (再次感谢 `G-Rom`, `Lorian` & `Stone` !!), 删除程序 ‘REGMON.EXE’ 或 ‘NOTEPAD.EXE’ 的 `PE directory information / Import AddressTable` 里显示的信息.

我们会看到对这些程序的运行不带来任何的影响… &;-)

## 在实际操作前的总结

逻辑上, 在 `.rdata` 节里, 输入表的组织, 结构与我们在 idata 节遇到的基本一致.

唯一的盲点对我们来说是, `Image Import Descriptor` (目录表) 的块段不处在 rdata 节的开端.

我们要利用一个 DLL 的名字 (最常用的是 `KERNEL32.dll’`), 来找出这个块段的位置, 然后, 正确判断出 `Image Import Descriptor` 开头的地址和整个块段的大小.

这两个信息将被放在 `PE directory information / Import Table / RVA & Size`.

在实际操作中, 我将演示带有一个 `.rdata` 接段的两个加壳程序.

* `ASPack` 加壳的 `ACDSee 3.0` 法文版 (不去管它的版本...)

* `ASProtect` 确加壳的 `volume 3.1` , (这个也是, 它的版本没有什么关系... 听说是 `0.9` 版)

为什么特别选择这两个例子?

* `ACDsee 3.0 fr` : 很简单, 是因为一天, 有朋友在手动脱壳后, 叫我帮忙找回要放进 `PE directory information` 里的正确数值, 在这, 唯一难的是我们将要对付一个包含 `.rdata` 的节.

* `Zvolume 3.1` : 是因为不久前有一篇关于这个程序的解文, 并且文中提出这是关于 `ASProtect` 的一个新版本. 我想要看看是怎么样的, 可我很惊奇的在里面发现了一个 rdata 的节…我就把它用于这篇文章里.

在 `Zvolume 3.1` 的例子里, 获得一个能运行的脱壳程序, 就是自动删除了 `ASProtect` 对程序设置的各种保护措施. (特别是对 checksum 的检验).

### `ASPack` 与 `ASProtect` 间有什么不同?

在这方面已经有不少文章, `Christal` 肯定是第一个带头研究的, 并且指出这两个产品的区别和近似之处.

`ASxxxx` 就是 `Alexei Solodovnikov`.

`ASPack` 仅仅是一个压缩/解压的壳, 它只修改一部分的输入函数, 不包含别的附加保护措施.

`ASProtect` 却是一个做好了的保护措施程序, 它除了用 `ASPack` 的压缩算法, 还有别的东西.

也就是一个包含别的附加保护措施的 `ASPack`, 如: anti-softice, checksum, time-limit 等等...

在 `ASPack.com` 的网址上, `Alexei Solodvnikov` 指出他的产品也包含 anti-procdump.

« counteraction to dumping application memory with the tool like ProcDump »

« API for interaction between application and protection routines »

我不知道它所谓的 anti-procdump 是什么含义 : 是让 dump 无法进行 (也就是说这个程序无法被 DUMP), 还是 DUMP 是可以的, 可是被 DUMP 出来的执行文件将无法运行? (比如, 在输入函数方面做了手脚...)

无论如何, 在实际操作部分您将看到, 我们将不会遇到这种麻烦, 因为我们要对输入函数下点工夫.

在实际操作部分的目标是: 获得执行文件的原件, 也就是说, 在它被防盗的保护措施修改前.

得到的程序将是可反编译的 (当然包含输入函数!), 还有就是, 要是我们愿意, 可对它进行修改.

这要靠 `Icedump` 和 `Procdump` 对它手动 `DUMP` 来取得.

好了, 说了很多, 我们来点实际行动吧!

### 第一个实际操作的目标: `ACDSEE 3.0 fr`

( 被 `ASPack` 传染的 )

我们先借助 `Procdump` 对 `PE header` 进行观察. (再次感谢 `G-Rom Lorian & Stone` !)

![](img/96ceedee9c73256f8cc19cdda2b5bff8.jpg)

图 06.

`Entry point` 让我们来到 `ASPack` 节.

![](img/09ee3fbdaf3f9859d6c97fdd9635c436.jpg)

图 07.

这里很清楚, 我们对付的是 `ASPack` !

哪个版本? 无关紧要的!

哎! 没有 `.idata` 节, 这次输入函数都在 `.rdata` 节.

这不要紧, 您刚才看见了这对我们重新找回我们的小朋友们没有什么困难的.

为了有一个好的开始, 只须找到一个好的定点, 然后一切就会水到渠成.

我们继续对 PE 里的信息进行观察.

![](img/b88ac1cb1701c5fe9be7dac38e1b146f.jpg)

图 08.

import image descriptor 的地址是 ‘15AFC0’ (也就是说在 ASPack 节正里面.)

要把这个地址修改为指向原来程序的 import image descriptor, 即 `.rdata` 节. (在 `CF000` 与 `E3FFF` 之间) .

还要修改它的大小, 为了 windows 的装载器能重新找回它的小朋友们, 以免死机.

我们留意到没有任何输入地址表里的信息 (包含内存里每个函数的调用地址 (举例: `7FD47579`))

我们面对的是一份 Borland 类型的输入表吗? (其实不是的, 但我们慢慢的意识到这点.)

总之, 我们已经说过: 这些信息对 windows 的装载器没有什么作用 .

以下是为了得到一个有效的脱壳后可执行文件的几个步骤:

* DUMP 出 rdata 节, 当它被 ASPack 解压, 还是'干净'时.

* 找出被解压程序的入口处 entry point , 并手动 DUMP 出程序.

* 将 DUMP 出的. rdata 节的文件插入解压后的程序, 并找出 Image Import Descriptor 的区域.

* 用 Entry Point 的值, 输入表的开端的地址, 及大小为 PE header 升级.

## 第一步, DUMP 出 `.rdata` 节, 当它被 `ASPack` 解压, 还是'干净'时.

嗯 !… 怎样才算是'干净 '?

当然, 在这, 我们不能以检测 import image descriptor 或 输入表的地址是否正确为乐, 因为我们连它们的内存区域也不清楚...

那么, 我们要找出比较有代表性, 简单的一个定点, 它将向我们指出在某时某刻, 输入表的信息将被解压, 还原, 而且是正确的...

我们现在已经知道, 只需在 `.rdata` 节里, 寻找当前函数的名字.

例如 `KERNEL32`.

一旦我们看见这个 DLL 名字的明文, (已及与其相应的函数名), 我们就可以 DUMP. rdata 节.

然后, 我们可以慢慢的检查输入表里的信息, 和找回我们的小朋友.

来 ! 一等 .rdata 节解压, 我们来 Dump 它.

(`.rdata` 节从 `4CF000` 到 `4E3FFF` , 有 `15000` 字节.)

在对 `.text` 节的属性修改成 `E0…..20` 后, 我们用 `WinIce` 的 `symbol loader` 装载程序. (感谢 `Numega` !)

`Break due to Symbol Loader`

`0177:0055A001 60 PUSHAD`

`:bpm cs:4cf000 W <`-- 这样一来, 每次 ASPack 的 loader 要对 `.rdata` 节进行写入时, 我们就会有一个截停.

`:x <`-- 我们让 ASPack 完成任务它的任务: 分块给程序解压.

`POP` ! (ASPack 刚在 `.rdata` 段写入了 !)
