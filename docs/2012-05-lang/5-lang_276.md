
# VB PCODE 程序破解纲要

> 原文：[`www.pediy.com/kssd/pediy04/kanxue315.htm`](https://www.pediy.com/kssd/pediy04/kanxue315.htm)

1. 不管 VB 程序是采用何种方法编译——自然编译或 pcode 编译——其根本还是建立在 VB 那几个 dll 文件上，这主要包括 `msvbvm60.dll` 与 `oleaut32.dll` 两个文件。
   所以你应该努力学习上述 2 个文件中的函数及其应用。学习的方法有：
   (1) 利用 `wdasm` 反汇编 `msvbvm60.dll` 与 `oleaut32.dll`，分析其反汇编结果
   (2) 利用 `smartcheck` 记录,作调试信息
   (3) 利用 `trw2000` 或 `softice` 动态跟踪
   直到你能了解大部分的 vb 函数。
   此时你具备了 50%的能力。

2. 利用 `wkt vb pcode debugger` 进行动态跟踪， 从中进一步了解 `msvbvm60.dll` 与 `oleaut32.dll` 函数的应用；并通过 `exdec`、`wkt vb pcode debugger` 等相关工具加深自己对以上函数独特命名的理解。
   这使你获得 35%的能力。

3. 关于 vb pcode 程序断点设置的技巧
   包括如何在 `wkt vb pcode debugger` 中设断；
   如何在 `trw2000` 中设断；
   如何利用 `softice` 设断(记得载入 `msvbvm60.nms` 或 `oleaut32.nms`)。
   这部分占 15%。

充分学习上述纲要，破解 vb pcode 程序并不困难。
以上所述即为《VB PCODE 研究》一文之纲要。特此字之。
感谢 Aming 兄指正！ 4dd466d0e1e0cc123afbe504da2462ca
