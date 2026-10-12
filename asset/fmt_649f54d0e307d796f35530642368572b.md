
# Sentinel LM lic 的制作总结：用去狗 `wlscgen.exe` 法

> 原文：[`www.pediy.com/kssd/pediy11/115811.html`](https://www.pediy.com/kssd/pediy11/115811.html)

**作者**: `zhongtiany`

**软件名称**: `Sentinel LM`

**软件类别**: 商业保护类

**软件介绍**: `Sentinel SLM lic` 制作

**破解工具**: `SLMDec1.03`, `ollydbg 1.10` (带 `godup` 插件), `moZfet SentinelLM tools` (含 `slmkeys.EXE` 和 `SLMtoolkit.exe`), `Sentinel SLM SDK`, `CodeFusion 3.0`, 去狗 `WlscGen.exe`。

## Sentinel LM lic 的制作总结

这两天一直在折腾 `Sentinel LM lic` 的制作，在阅读了前辈们的一些建议和总结后，终于搞定。现将自己做的总结与像我这样的初学者分享，高手就不要看了，免得浪费时间。

### 1、找出 `VendorID`，`Feature name` 等信息

**(1) 简便的方法**

前提条件是已经有现成的 `lic`，根据 `Demo License` 用工具 `SLMDec1.03`（一个 `SentinelLM license` 文件的解码工具）推算出 `Vendor ID`、`Vendor Info`、`Feature` 等信息。

用法为：在 dos 下运行：`slmdec103 XXXXXXXXXXXX#`（`Demo License`）。

显示如下：

```
C:\slmdec103>slmdec103 XXXXXXXXXXXXXXXXXXXXXXXXXX#

SentinelLM [8.0.x /7.x.x] license decode utility, v1.03 public (C)2007 by souz

------------------------------------------------------------------------------

SentinelLM 6.0.x detected

Version tag: [XX]

VendorID     = XXXX

Vendor  info = XXXX

Feature name = XXXX

Feature vers = XXXX
```

**(2) 如果没有 `lic`，可以用 `ollydbg` 调试方法**

安装完目标软件后运行 `ollydbg`, 载入主程序。载入后用 `ollydbg1.10` 的 `godup` 插件（用于加载 `sentinellm ida sig`）取相应的 `sig` 文件和 `dumsig.exe` 文件（`dumsig.exe` 在 `godup plug` 目录中）。

右键 > 查看 > 模块，点击并进入 `lsapiw32.dll` 模块；右键 > 查看 > 所有参考文本字串：搜索 `"%s error: Illegal vendor identification"`，该字符串位于 `computevendorcode` 子程序内，该子程序即调用 `vendor id`，在其返回处设置断点。按 `F9` 运行程序，到断点处中止运行，察看 `eax` 内容，即为 `VendorID`。同时也可在堆栈窗口中看到 `Feature name`。

### 2、得到 `SentinelLM SDK` 安装序列号

在 dos 中运行 `moZfet SentinelLM tools` 中的 `slmkeys.EXE`，得到一个所有 `VendorID` 对用的 `SDK SNs` 的数据库 `SLMKEYS.DAT` 文件，这个过程时间比较长；

然后用 `moZfet SentinelLM tools` 中 `SLMtoolkit.exe` brute forte the `SDK SN`，根据 `VendorID` 生成 `SDK SN` 以及无狗的 `WlscGen.exe`（注意：`SLMtoolkit.exe` 只能生成 7.10 版本，其他版本的无狗 `WlscGen.exe` 得自己找）。

（注意：如果以＊号开头的 license 串则是 8.0 的，必须用 `SentinelLM SDK8.0` 以上版本的 `wlscgen.exe` 制作）

### 3、用生成的 `SDK SN` 安装 `SentinelLM SDK`

### 4、制作去狗 `WlscGen.exe`

用 `CodeFusion 3.0` 比较安装 `SentinelLM SDK` 目录中的 `WlscGen.exe` 与同版本的无狗 `WlscGen.exe` 比较，比较差异中删掉 `VendorID` 部分（7.10 版的是 `0015c8d0-0015c8df` 部分），然后生成补丁程序，再利用生成的补丁程序修改 `SentinelLM SDK` 目录中的 `WlscGen.exe`，这样就去掉狗了。

备注：也可利用网上找到的同版本的去狗 `WlscGen.exe` 来制作补丁，数据中哪部分是 `VendorID` 的数据就得自己去判断了。

### 5、制作 `lic`

`SentinelLM SDK` 目录中的 `echoid.dat`，`lmlicgen.usr`，`lsapiw32.dll`，`echoid.exe`，去狗 `WlscGen.exe` 文件拷贝出来放到新建的文件中；

运行去狗 `WlscGen.exe`，第一次运行密码为空，直接点进去就是了。然后填入 `lic` 的 `Feature name`，`Feature vers`，`Vendor info` 等信息，生成 `lic`。

### 6、检验制作的 `lic`

方法同步骤 1，主要是检查 `Vendor ID` 对不对。最后配置 `lic`，并运行目标软件，如果能运行就 `OK` 了。

感谢看雪，`Crakz`，和 `finetopix.com` 网站上对 `SentinelLM` 早有研究的前辈们！

![](img/cda69129882b40377a87144c782680e1.jpg "微笑")

顺便向大家求一个去狗的 8.0.6 版的 `WlscGen.exe`！

## 补充资料

以下是网上能下载的去狗 `wlscgen`：

1、`WlscGen 7.3 patched`

下载链接：http://bbs.pediy.com/showthread.php?t=104568&highlight=wlscgen

2、`WlscGen 8.2.1.5 patched`

下载链接：http://rapidshare.com/files/393285789/wlscgen2A0A.zip

（来至：`rengteam.blogspot.com/2010/05/wlscgen-821500-for-your-needs.html`）

这个版本被别人加了壳，很难脱，希望哪位高手能试一下脱壳。

其他去狗版本请大家上传共享！
