
# UPX

> 原文：[`www.pediy.com/kssd/pediy04/kanxue275.htm`](https://www.pediy.com/kssd/pediy04/kanxue275.htm)

脱！脱！脱——当然是脱壳，脑筋别打坏主意！用 `UPXPR` 处理的 `EXE` 和 `DLL` 程序无法脱壳的问题想必伤了许多人的心，我的头亦发烧！随手抓了个宝贝，一招使出辟邪剑法，杀的 `UPX` 喝洗脚水！

## 打斗剧情

- 宝贝：`dREAMtHEATER` 写的 `UPX Shell 0.9`
- 示例：`SSREADER354` 的 `SSREADER.EXE`，`Pdg2.dll`

## 辟邪剑法

用 `HEX EDIT` 打开 `SSREADER.EXE` 和 `Pdg2.dll`，在程序开始下一点找文本 `"rsrc"`，`"rsrc"` 的上一点有文本 `"１"` 和 `"０"`，改成 `"upx1"` 和 `"upx0"`，再找文本 `"１.２０"`，其后有文本 `"!"`，改成 `"upx!"` 保存。

为什么这么罗嗦？每个文件的地址不同嘛！`upx=555058(hex)`，用 `upx shell 0.9` 解压缩，运行 `ssreader` 一切 `OK`! 赶快去改 `pdg2.dll` 的 `Bug`!

非常感谢 `dREAMtHEATER`！

`KINGSUN`

`2001-11-29`

版权所有 `4dd466d0e1e0cc123afbe504da2462ca`
