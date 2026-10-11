
# ReGet Junior 2.0(三）

> 原文：[`www.pediy.com/kssd/pediy04/kanxue426.htm`](https://www.pediy.com/kssd/pediy04/kanxue426.htm)

ReGet Junior 2.0 破解手记(三）

作者：囚童\[FCG\]\[BCG\]

课题：（三）脱除 ReGet Junior 2.0 主程序文件 `regetjr.exe` 的壳

背景：PACKED WITH ASProtect v1.2

下载：`http://www.cn778.net/download/c_bexpress.zip`

工具：
* CASPR v1.012
* EditPlus 2
* language2000 中文版 v4.51
* FileInfo2.40
* TYP
* BTW2000

接 ReGet Junior 2.0 破解手记(一）、(二）

在破解手记(一）、(二）中，我们讲了如何解除 ReGet Junior 2.0 广告动画画面的方法三种，解除显示软件名称和版本号的 BANNER 的方法，并制作了内存补丁。

这里，我们介绍如何给 ReGet Junior 2.0 主程序文件 `regetjr.exe` 脱壳。

`regetjr.exe` 原有的尺寸为 497K。

用 language2000 中文版 v4.51、FileInfo2.40 和 TYP 都没能查出 `regetjr.exe` 是否被压缩过，但用 BTW2000 和 TRW2000 观察到的程序入口分别是 `440A2A` 和 `400040`，据此可以判断：`regetjr.exe` 是被压缩过的，怀疑是 ASProtect v1.2。

① 用 EditPlus 建立 `CASPR.ini` 文件如下：

```ini
[Options]
Import=0x00000001
Export=0x00000001
TLS=0x00000001
Resource=0x00000001
Relocation=0x00000000
Overlay=0x00000001

[RVAs]
;IMAGE_IMPORT_DESCRIPTOR
IID=0x00000000
;IMAGE_IMPORT_OriginalFirstThunk_DATAs
IIOFTD=0x00000000
;IMAGE_IMPORT_BY_NAMEs
IIBN=0x00000000
;IMAGE_IMPORT_DESCRIPTOR.DLLNAME
IIDDN=0x00000000
;IMAGE_EXPORT_DIRECTORY
IED=0x00000000
;IMAGE_TLS_DIRECTORY
ITLSD=0x00000000
;IMAGE_RESOURCE_DIRECTORY
IRD=0x00000000
;IMAGE_RESOURCE_DATA
IRData=0x00000000
;IMAGE_RELOCATION
IR=0x00000000
```

② 用 EditPlus 建立批文件 `Un-pack.BAT` 如下：

```bat
CASPR -L regetjr.exe -P Output
```

③ 建立一个工作目录 `Un-pack`。将下列文件拷入这个目录：
* `Caspr.exe`
* `Caspr.ini`
* `Caspr1.dll`
* `Caspr2.dll`
* `regetjr.exe`
* `Un-pack.BAT`

④ 在 `Un-pack` 目录下再建一个子目录 `Output`。

⑤ 运行 `Un-pack.BAT`

`Caspr` 在 `Output` 目录中生成一个新文件 `regetjr.ex_`。

没有异常提示出现。

⑥ 将新文件 `regetjr.ex_` 以后缀 `EXE` 更名后，覆盖原有的 `regetjr.exe`。

⑦ 用 BTW2000 复查程序入口点为 `440A2A`。

⑧ 试运行新的 `regetjr.exe`，没有异常提示出现，检验各项功能齐全。

新的 `regetjr.exe` 尺寸为 982K。

脱壳成功。

相关文章：ReGet Junior 2.0 破解手记(一） ReGet Junior 2.0 破解手记(二) ReGet Junior 2.0 破解手记(三）
