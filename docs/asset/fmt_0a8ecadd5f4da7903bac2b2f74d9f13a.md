
往上下翻，可以得到 `IAT` 表的起始地址和大小。起始是 `53C100`，结束是 `53CED4`。长度一减得到 `DD4`。

现在启动 `ImpRec`，选这个进程，输入其 `OEP` `1000`，`IAT` 起始地址 `13C100`（已经减去了 `ImageBase` `400000`），长度 `DD4`，点 `GetImports`，得到一堆 `Valid NO` 的。

`Show Invalid` 后把十几个无效指针挨个 `CUT` 掉，再 `Fix dump` 刚才 `dump` 出的文件。保存后双击，正常运行。脱壳成功。
