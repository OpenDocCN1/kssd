
# Sentinel RMS lic 制作总结 2 模拟狗法

> 原文：`www.pediy.com/kssd/pediy11/117086.html`

作者：zhongtiany  

软件名称：Sentinel RMS  

软件类别：商业保护类  

软件介绍：Sentinel RMS lic 制作  

工具：lsdecode Sentinel 8.2.2；ollydbg1.10；SentinelLM SDK8.0.6；moZfet SentinelLM tools；TORO Sentinel SuperPro Emulator1.7；UltraEdit.  

我发现论坛里有很多人在寻找 SentinelLM 8.0 以上版本 lic 的制作方法，`wlscgen8.0` 以上的去狗版太难找了，肯定是有的，但是大家都不愿意拿出来共享。上周折腾了好几天，终于用另外一种方法模拟狗法成功制作了 lic，现总结出来与大家分享。希望大家多多发表自己的见解，分享自己的收获。  

在 Sentinel LM lic 制作总结 1 中提到：以＊号开头的 license 串则是 8.0 及以上版本的，必须用 SentinelLM SDK 8.0 以上版本的 `wlscgen.exe` 制作，本文针对 SentinelLM SDK 8.0 和 8.2 版本 lic 的制作。  

### 1、找出 VendorID，Feature name 等信息。  

（1）简便的方法：前提条件是已经有县城的 lic，根据 Demo License 用工具 lsdecode Sentinel 8.2.2（一个 SentinelLM license 文件的解码工具）推算出 Vendor ID、Vendor Info、Feature 等信息。  

用法为：将 lservrc 文件考到 lsdecode Sentinel 8.2.2 的目录下；  

用 ollydbg 加载 `lsdecode.exe`，按 Ctr+G 查找以下地址，并设置断点，然后在相关部分查找有关信息：(仅用于 7.x-8.1 Sentinel licenses)  

`vendor code`：`00422460` (`EAX`)  

`vendor info`：`00423A60` (`ECX ASCII`)  

`Feature Name`：`00423988` (`ECX ASCII`)  

`7 secrets`：`00409CF7` (`EAX ASCII`)  

`Client lock Code`：`00423906`  

`Feature version`：`004239DD`  

（2）用 ollydbg 调试方法：安装完目标软件后运行 ollydbg, 载入主程序。载入后用 ollydbg1.10 的 godup 插件（用于加载 sentinellm ida sig)取相应的 sig 文件和 `dumsig.exe` 文件（`dumsig.exe` 在 godup plug 目录中）。右键》查看》模块，点击并进入 `lsapiw32.dll` 模块；右键》查看》所有参考文本字串”：搜索 `"%s error: Illegal vendor identification"` , 该字符串位于 `computevendorcode` 子程序内，该子程序即调用 vendor id ，在其返回处设置断点。按 F9 运行程序，到断点处中止运行，察看 `eax` 内容，即为 VendorID。同时也可在堆栈中看到 Feature name。  

### 2、得到自己需要的 wlscgen.exe（即含有自己需要的 Vendor ID 信息）：  

（1）得到 SentinelLM SDK 安装序列号：在 dos 中运行 moZfet SentinelLM tools 中的 `slmkeys.EXE`，得到一个所有 VendorID 对用的 SDK SNs 的数据库 `SLMKEYS.DAT` 文件，这个过程时间比较长；然后用 moZfet SentinelLM tools 中 `SLMtoolkit.exe` brute forte the SDK SN，根据 VendorID 生成 SDK SN  

（2）用生成的 SDK SN 安装 SentinelLM SDK，即可得到自己需要的 `wlscgen.exe`。  

### 3、启用模拟狗 TORO Sentinel SuperPro Emulator1.7(见附件)：  

（1）用 UltraEdit-32 修改 sample RegFile 目录中的 `A870.TORO`，这样修改：将偏移地址为 `0x000060h` 处的 `70 0A`（默认的 VID 为 `0x0A70`）修改为你的 VID（例如: 如果你的 VID 是 ABCD，那么需要将 `A870.TORO` 的 VID 改为 CDAB ）；  

（2）复制 dll 文件夹中的两个 dll 文件和修改 VID 后的 `A870.TORO`，共三个文件到 `C:\WINDOWS\system32` 目录下；  

（3）运行 Service 目录中的 `install.bat`，安装模拟狗。  

### 4、运行 wlscgen 8.0.6 并制作 lic。  

注：此方法适用于 Sentinel sdk 7.3-8.1；  

对于 8.0.6 版，不需修改模拟狗中的 VID，wlscgen 一样也可以生成 lic，lic 中的 VID 只取决于 `wlscgen.exe` 中的 VID。  

## 关于 Sentinel sdk 8.2  

由于目前网上找不到 Sentinel sdk 8.2 及以上版本，但可以找到 `wlscgen8.2` 的版本。需要修改 `wlscgen8.2` 中的 Vendor ID，使用 TORO DSF USB EMULATOR 模拟狗，用 `pva dumper3.3` 生成`.dmp` 文件，然后用 `dmp2mkey2.3` 将 dmp 文件转化成`.reg` 文件，然后修改 write password，Vendor ID，和部分 Cell 中的数据，并按照 MultiKey 18.2.X 中的例子修改一下.reg 的格式其中要注意：  

如果用 mkey 18.2.x 模拟，需要这样修改 reg 文件： 

```
"CellType"=hex:\
01,01,03,03,03,01,03,01,\
```

如果用 mkey 18.0.x 模拟，需要这样修改 reg 文件： 

```
"CellType"=hex:\
01,01,03,03,03,03,03,03,\
```

最后用 mkey 软件进行模拟，生成 lic。  

谁有 `wlscgen8.2` 以上的版本请共享一下，sentinel lm sdk 更好！  

谢谢！

### 上传的附件

| 图片 | 文件名 |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | TORO Sentinel SuperPro Emulator 1.7.part1.rar |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | TORO Sentinel SuperPro Emulator 1.7.part2.rar |
