
# NotePro 1.57 脱壳之简版(Asprotect 1.2se)

下载URL = http://download.21cn.com/file/utility/bianji/noteproV1.57.zip

Calculating hash of 978432 bytes file `C:\Program Files\NotePro\notepro.exe`...

MD5        : 57322E435C1E39F7BECA3585D5E05D50

CRC-32      : FEE836FF

------------------------------------------------

## 重要资料

Entry Point  : `001012`

ImageBase    : `400000`

IAT Start RVA : `1C1150`

IAT Length    : `1174`  (ImpREC)

------------------------------------------------

## 使用工具

于win98se 中使用：`TRW2000v1.23`, `SuperBPM` ,`ImpREC1.2beta2`.

## 脱壳

1) 运行 `SuperBPM`,并选择 `Erase`
2) 运行 `TRW2000`,用"`Load`" 调入 `notepro.exe`。
3) 在 `TRW` 中,下 `bpx 401012` (即上面的 EntryPoint+ImageBase) , 再按 `F5`
4) 最后会在 `401012` 处停下。
5) `bc *` ,再用 `pedump c:\notedump.exe` ,再 `F5`
6) 到此脱壳完毕。 (无需关闭 `notepro.exe`)

## 修复ImportTable

1) 打开 `ImpREC` , 调入 `notepro.exe`
2) 在左下角的 IAT Infos needed 添入资料：

    OEP  : `001012`

    RVA  : `400000`

    Size : `1C1150`

  直接按 "`GetImport`" 钮。
3) 用 `ShowInvalids` 钮及鼠标右键来修复剩余的函数名。
4) 最后还有 5 个地址不能修复，需要手工填入。它们皆属于 `KERNEL32.dll`

  它们是：`FreeResource`, `GetCommandLineA`, `GetCurrentProcessId`, `GetVersion`, `LockResource`
5) 所有函数名修复后，按 `FixDump`(使用缺省选项)来修复我们 dumped 出来的 "`notedump.exe`"
6) 完毕。将 `notedump\_.exe` 拷贝到 `notepro` 的目录中。`notedump\_.exe` 可以正常运行。
