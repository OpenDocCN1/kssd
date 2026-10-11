
# 【己解决】IDA 5.2 如何从本地导入符号文件

> 原文：[`www.pediy.com/kssd/pediy10/92644.html`](https://www.pediy.com/kssd/pediy10/92644.html)

`IDA 5.2` 不能从服务器上下载符号文件，将 `5.1` 的 `pdb` 文件替换后只能下载 `514` 个符号文件。

我自己从微软下载了符号文件，大概 `100` 多 `M`，放在 `windows\symbols` 目录下，请问如何手动导入这些符号文件到 `IDA` 中？谢谢了。

已经建立了 `symsrv.yes` 空文件了。

问题已经解决。分享一下，主要是两个问题：

1. 目标文件的路径上不能有中文
2. 目标文件和 `pdb` 文件的版本不合适

我用 `windbg` 自带的 `SymChk.exe` 去找目标文件的符号文件，很快就解决了。

假设目标文件为 `a.exe`，则语法为：`SymChk.exe a.exe`

默认符号文件想要存放的路径，为 `windows\symbols` 下
