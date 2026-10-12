
# softworm 发布了 VM 解码器

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-539.htm`](https://www.pediy.com/kssd/pediy08/pediy8-539.htm)

`http://forum.exetools.com/attachment.php?attachmentid=4189`

附件下载：`decode.rar`

这个程序是拿来分析 `dumped\_.exe` 的，`pcode` 数据是用 `ReadProcessMemory` 读的。只是个 `console` 程序，需要改 `ProcessID` 和需要读的内存地址。程序不完整，只是够得到当前的结果。

`dumped\_.exe` 不是放出的那个，是没有删除 dump 出来那一大堆 `section`(vm 代码)的。

问题是 `themida` 加壳的时候加了些随机性，`pcode` 数据内各 `field` 的偏移量是变化的，解释程序的处理顺序也有变化。虽然套路差不多，但不能直接用到另一个程序上。我想 `themida` 加壳时显示的“生成特定 vm”可能指这个。

这个东西我耗了差不多 4 个月，只是笨工夫而已，没什么可夸耀的。鸡蛋壳不妨介绍一下怎么去掉 `nag`。
