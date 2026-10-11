
# Advanced Administrative Tools v4.0.0.596

> 原文：[`www.pediy.com/kssd/pediy02/forum222.htm`](https://www.pediy.com/kssd/pediy02/forum222.htm)

下载：`http://newhua.myrice.com/down/aat40a.zip`

脱！脱！脱！ ------ 众人曰："还在脱啊！脱光了没！" 答："还剩一条裤衩"

脱壳 ----- `AATools v4.0.0.596` (全称：`Advanced Administrative Tools`)

## 【前言】

我写文章以交流为主，希望大家在转载时保持文章的完整性。

多日对 `Asprotect v1.0` 加壳的程序做战未果，好容易有点收获。兴奋到了极点。晚上奋起神勇，挑灯做战。（老爸在一旁怒吼："几点了，还不睡，想成仙呀！"）。俺姓李，最多只能成半仙，如果俺姓胡就不同了。胡仙......呵呵。

样例文件：    `AATools.exe`    （主程序文件）

加壳方式：    `ASProtect + ASPack v1.084`

检测工具：    `FileInfo`

脱壳工具：    `SoftICE v4.05`,`ICEPatch v2.0`,`ICEDump 1.061`,`ProcDump 1.62`,`Imp_list`（这是一个好东东哦！URL `http://www.reversing.net/TOOLS/016/Imp_list.exe`）

目标要求：    脱壳

作者：       你看着办吧 ---- 陈小二语

日期：       2000-08-04

在《找 Entry Point 的又一方法 -----针对 Asprotect v1.0 加壳的程序》一文中我已说明了找 `Entry Point` 的方法。（补充：`Ding.Boy` 大虾写了 `BWW2000` 可以快速找到 `Entry Point`，强烈推荐，URL `http://202.103.134.11/~fsdb/bww.zip`）。这里剩下的事就不多了。我再说两句了。

### 1、首先，当然少不了用 ProcDump 来分析一下对我们有用的文件信息。

`Entry Point:     00222001`

`Size of image:    00239000`

`Image Base:      00400000`

### 1、找到 Entry Point，在入口处下指令

```
/dump 400000 239000 c:\temp\dump.exe
```

### 2、好，按 F5 键，程序运行。这时运行 Imp_list

单击 `Rebuild import`，片刻，`Imp_list` 会在工作目录下形成四个文件。分别是

`Import0.bin`

`Import1.bin`

`Import2.bin`

`Address.txt`

怎么样，简单吧。这样就 `Dump` 出了 import table。

### 3、用 ProcDump 修改 dump.exe

改各 `Section` 的 `Offset` 为 `RVA` 大小，`PSize` 为 `VSize` 大小。改 `Entry Point` 为 `1764C0`，改 `Directory` 中的 `Import Table` 的 `RVA` 为 `17D000`，大小为 `3400`。

### 4、最后别忘了把 dump.exe 中偏移 17D000 开始的字节用 Import0.bin 替换掉

运行 `OK`！

## 【后记】

这里说两句，首先在看雪的 `Crack` 教学中我知道 `cloundnumer9` 兄已经成功脱壳了（他的方法我没试过），然后 `IceWorld` 大虾在看雪论坛下一贴，既脱壳又破解了。最后才是我赶上这班车。因为开始我用手动脱壳的方法在重建 import table 上遇到了很大的困难，后来找一工具才帮我度过难关。有兴趣手动脱壳的朋友不妨一试，如果你有好方法可以重建 import table，一定要告诉我哦！`Imp_list` 这个工具可不是百试百灵的，不信你试试对 `AZPR v3.11` 和 `ARPR v1.0` 脱壳试试。

以前我总喜欢在后记中加入自已的一些问题，希望大家帮帮忙，帮我解决。可是一直没有回应。所以这次就不写什么问题了。当然如果谁对加壳、脱壳有兴趣，可以和我交个朋友。Email:`ljtt@yeah.net`。

`4dd466d0e1e0cc123afbe504da2462ca`
