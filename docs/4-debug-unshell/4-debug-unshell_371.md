
# 中国游戏中心大厅程序

> 原文：[`www.pediy.com/kssd/pediy03/forum363.htm`](https://www.pediy.com/kssd/pediy03/forum363.htm)

下载：`http://202.96.162.68/MyGames/DownLoad/Setup-Plaza.exe`

> Asprtect 已经出到 1.1 了, 怎么还在 1.0 打转, 只讲重建 `ImportTable` 的部份, 跟蹤技巧省略, 自己看教程慢慢摸索, 才能进步。这个软件有两个程式被 `Asprotect` 加壳, 一个是 `iGame.exe`, 一个是 `GameDev.dll`, 我们以 `iGame.exe` 为例, dll 的脱壳方法基本一样, 只是要點小技巧, 用 `Procdump32` 去分析 `iGame.exe`

> 你会发现没有 `.idata` section 只有 `.rdata` section, `Asprotect` 针对只有 `.rdata` section 会有另外一种编码方式, 我们不需重建 `Import Table`, 只要

> `Modify Image_Thunk_Data` 的矩阵结构即可。

> 那要如何 `Modify` 它呢？当然还是有利的工具 `Imp_list`, 这次我们用 `Imp_list V1.09`, 可在老俄那边下载, 记 `.rdata` section 的资讯 `VO=4C000, Vs=11000`

> [脱 壳]:

> (1) 用 `BW2K`, 得知 `OEP=004224D6`

> (2) 执行 `Imp_list`, 会分别得出, `Address.txt`, `Import0.bin`, `Import1.bin`, `Import2.bin`

> (3) 在无 `Soft-ICE` 下, 按 `Ctrl-N` 叫出 `Trw2K`, 下 `faults off`, 再下 `CreatefileA do "d *(esp+4)"`, 执行 `iGame.exe`, 拦住后, 按 `F12/F10`, 直到返回主程式, 下 `BD *, G 004224D6`, 下 `Suspend` 用 `Procdump32 dump(full)`, 存成 `iGdump.exe`, 改 `EP=000224D6`

> (4) 用 `Hex Workshop` 打开 `Import1.bin`, 从偏移 `1000` 处开始选方块 `728`, 将其贴至 `iGdump.exe` 的 `.rdata` section 的 `Raw Offset` 处, 即 `004B600`。

> 这样即可脱颖成功.

> C-Pen 2000\10\08 `4dd466d0e1e0cc123afbe504da2462ca`
