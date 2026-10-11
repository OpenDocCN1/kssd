
![](img/99a1bd6afb5ff11e1b2cce39c7db2571.jpg)

这里出现了另外 2 张表。`7220B5` 的表中数据为 `size`。`Dumped_.exe` 中有：

![](img/7483ffb33e3d11e7238ee26f11f64aa4.jpg)

问题出在第 3 张表：

![](img/edb3d7ecad6cb8b1254c6a06ab57e870.jpg)

`dump` 出的数据为 `0`。这段代码要把主程序中的一段数据 copy 到这张表中数据所指的地址。在 `loader` 中执行时，这里填入了指向动态分配内存的指针。

![](img/baee70caf45dfb5beff4b83cec070d46.jpg)

显然不能直接复制这些值。有个简单的办法可以骗过 `loader`。从那张 `size` 表中可以看到，最大的数据 `FD5D`。用 `LoadPE` 再次增加 1 个 `section`，`size` 为 `FFFFFF` 即可。

![](img/0da98cb0cc72fb2f89f28a6c7ea17d73.jpg)

修改 `dumped_.exe`，设置 21 项数据，使其全部指向该地址。

![](img/4f9b71cd49a77aa98195b9e722904969.jpg)

在 `W2K` 下运行，显示窗口，但不能响应输入。在 `WinXP` 下运行什么也不显示。

下面该与主程序交手了，这需要把板凳坐穿的耐心 L。
