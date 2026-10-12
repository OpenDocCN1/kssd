
在 `5E78E2` 第 1 次异常时，处理 `user32`, `GDI32`, `ADVAPI32`, `RPCRT4`, `IMM32`, `LPK` 共 6 个 dll。`KERNEL32` 的 api 已经在更早的时候处理过了。此时上面已经填入了约 60 项 `kernel32` 的 api 地址。

相关代码与 `SDP` 以前的版本相似:  

![img/3b25cd1f2bd6ecb285d9b5afccff1e89.jpg](img/3b25cd1f2bd6ecb285d9b5afccff1e89.jpg)

需要注意的 dll 仍然是: `kernel32`, `user32`, `advapi32`, `shell32`, `gdi32`, 以及 `sdprotector.dll`。

在决定以子进程运行前，壳代码处理了上述几个 dll，获得的 API 地址保存了 2 份，分别在壳空间和堆中。

现在试试跟从按子进程运行后，这段代码的处理。试跟了 2 个，并非 `5E78E2` 异常都与 IAT 相关。用脚本试试:  

![文本框: 005E78E2   Single step event at MP3\_Audi.005E78E2counter\_child = 00000013](img/e8357663c11b0ad269031c6f5d79e2f2.jpg)

到这里 IAT 中已经填入了一些重定向值了。从上一个 `int 3` 异常开始跟。

![img/df75690fa1b4fa620716abf8f86d95e7.jpg](img/df75690fa1b4fa620716abf8f86d95e7.jpg)

这里开始了。注意这 2 个函数:  

![img/c487270ff5e41b40c7255bf57297519d.jpg](img/c487270ff5e41b40c7255bf57297519d.jpg)

进去看看:  

![img/ed57555c5707868d90f30d5b3d0ed13b.jpg](img/ed57555c5707868d90f30d5b3d0ed13b.jpg)

连续 6 个这样的函数。`5EEB3D` 进去有些计算，象 `CRC32`(有个初始化 256 byte 向量的动作 J)。不过这已经不重要了，只要稍微想一想，就明白这是在干什么。6 个特殊对待的 dll，显然这些函数用于判断 J。

IAT 中的数据很规范，基本是这样的格式。  

![img/93d030c8e47fea387e42f5551fdbb6a4.jpg](img/93d030c8e47fea387e42f5551fdbb6a4.jpg)

什么时候变成指向 `14xxxx` 或 `5Exxxx` 就是被处理过了 J。

`第 0x12 次 int 3 -> 处理 kernel32.dll`

![img/e2cf6ea55dbe2e93bb9925b54060feed.jpg](img/e2cf6ea55dbe2e93bb9925b54060feed.jpg)

(这个也许不跳也行?我没有试 J)

![img/2f649bfc1f72098023529575656ac654.jpg](img/2f649bfc1f72098023529575656ac654.jpg)

`第 0x13 次 Single step event -> 处理 user32.dll`

![img/dc4c23f75fb9c8226d97eec824b35365.jpg](img/dc4c23f75fb9c8226d97eec824b35365.jpg)

![img/d805c1bf86899c1f8810c8a50a004647.jpg](img/d805c1bf86899c1f8810c8a50a004647.jpg)

`第 0x14 次 Single step event -> 处理 advapi32.dll`

![img/84cd19553e68e511771c12b5b631f5dc.jpg](img/84cd19553e68e511771c12b5b631f5dc.jpg)

依此类推，直到 IAT 全部填充完毕。

![img/ecbabb36952c0d48bbde020770e900f1.jpg](img/ecbabb36952c0d48bbde020770e900f1.jpg)

全部有效。`Fix dump`, 运行:  

![img/2c5628e4df4ee4a8e7d062a65f2c746f.jpg](img/2c5628e4df4ee4a8e7d062a65f2c746f.jpg)

只脱掉壳，并未破解。
