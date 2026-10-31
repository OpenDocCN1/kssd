
# [原创]实现脱早期 AJM 的 APP 壳的流程

# 实现 AJM 早期 APP 脱壳的流程

## 1. 使用工具查看 APP 是否已加固

经过工具查看，此 APP 是经过 `AJM` 进行了加固处理。

![](https://bbs.kanxue.com/upload/attach/202004/732177_ZZK4T9JVKXTZGB7.png)

## 2. 使用 JADX 查看 APP 结构

### 2.1 查看 APP 使用的 Application 类

![](https://bbs.kanxue.com/upload/attach/202004/732177_U3Y29MAWDCQ76UU.png)

### 2.2 分析 SuperApplication

在这可以看到 `attachBaseContext` 和 `onCreate` 函数分别调用了 `NativeApplication` 的 `load` 和 `run` 函数。根据 Android Application 的生命周期来说，`attachBaseContext` 比 `onCreate` 早。

![](https://bbs.kanxue.com/upload/attach/202004/732177_GXPZZ97H3SV4PKZ.png)

### 2.3 分析 NativeApplication

从这边可以看到 `load` 和 `run` 均为 `native` 函数，并且该类中还有一个 `runAll` 的 `native` 函数。同时，可以看到里面静态加载了 `exec.so` 与 `execmain.so`。

![](https://bbs.kanxue.com/upload/attach/202004/732177_XG4WH73T78EE8WG.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_HG9GFH399M8M36K.png)

## 3. 使用 IDA 静态分析 libexec.so

### 3.1 在导出表中查看 JNI_OnLoad、load、run 等信息

暂无发现。

![](https://bbs.kanxue.com/upload/attach/202004/732177_SY6KXD69BVK6H9N.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_S5SDBTBNF9Q766M.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_VJ6REWYG27F4KHY.png)

### 3.2 在 String 表中浏览一下是否有重要信息

在 String 表中发现了有个 `classes.dex` 的可疑字符串。

![](https://bbs.kanxue.com/upload/attach/202004/732177_F3DHV9U8PJ8VXRP.png)

双击点击进入并查看交叉引用。

![](https://bbs.kanxue.com/upload/attach/202004/732177_J2G385VK2JZVT4Z.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_ZZX97CZZH2Y9GMM.png)

来到引用的位置，并且按 `F5` 切换查看伪 C 代码。

![](https://bbs.kanxue.com/upload/attach/202004/732177_7MTW9RZQ3G5VN3E.png)

### 3.3 分析引用了 classes.dex 字符串所在函数的伪 C 代码

在之前的 `sub_1A8E8` 函数中也有传入 `"assets/ijiami.dat"`，而后再传入了 `"classes.dex"` 字符串信息。

那么，目前有理由怀疑此处为加载解密后的 DEX 数据。

![](https://bbs.kanxue.com/upload/attach/202004/732177_MYMCPZAQ94D8ESM.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_E9EWYHHUY837D6W.png)

## 4. 使用 IDA 动态调试 dump DEX 文件

### 4.1 启动 android_server

![](https://bbs.kanxue.com/upload/attach/202004/732177_UB7FKWMBAVCEG5B.png)

### 4.2 端口转发

![](https://bbs.kanxue.com/upload/attach/202004/732177_J7YC97VN58FP89A.png)

### 4.3 启动调试程序

![](https://bbs.kanxue.com/upload/attach/202004/732177_M9AXXHNKEUA5ZHE.png)

### 4.4 开始附加调试进程

![](https://bbs.kanxue.com/upload/attach/202004/732177_AQMD8KAHTU7BWXJ.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_5NGXVWF34HVA68W.png)

### 4.5 进程附加完毕后勾选三项

![](https://bbs.kanxue.com/upload/attach/202004/732177_3NE5YA8FVKENVUF.png)

### 4.6 使用 DDMS 查看调试进程

![](https://bbs.kanxue.com/upload/attach/202004/732177_6KV7TEC32K2JCME.png)

### 4.7 使用 jdb 开启调试

![](https://bbs.kanxue.com/upload/attach/202004/732177_TZYTZFXRNU3KD2N.png)

### 4.8 在 IDA 中按 F9 开始加载 so，并定位关键函数位置后开始正式调试

加载 `libexec.so` 后，可使用基址加偏移的方式定位到引用 `classes.dex` 字符串的函数位置。

当前 `libexec.so` 的基址为 `0xB38DD000`。

![](https://bbs.kanxue.com/upload/attach/202004/732177_MR9TPM9RJK9M88D.png)

偏移地址为：`0xEDBC`。

![](https://bbs.kanxue.com/upload/attach/202004/732177_AEZQJAN9D9GZB36.png)

计算绝对地址：`0xB38DD000 + 0xEDBC = 0xB38EBDBC`。

此时注意，在消息输出窗口看到 `libexec.so` 加载后，再使用 `G` 键进行跳转，否则过早跳转过去后按 `P` 键，发现汇编指令与静态分析时对不上。

![](https://bbs.kanxue.com/upload/attach/202004/732177_RXYT9ZR796WQ4BX.png)

过早跳转后按 `P` 键转换的汇编指令：

![](https://bbs.kanxue.com/upload/attach/202004/732177_VW63BCNBG4AZFTK.png)

OK，该次调试可以结束了。那么重新开启调试，从头走一遍 IDA 动态调试的流程。待发现第一次加载了 `libexec.so` 之后，可以在该模块的 `deinit` 函数中先下断，然后一直按 `F9`，直到运行到此函数为止。

![](https://bbs.kanxue.com/upload/attach/202004/732177_WKBUCER99G6S7CG.png)

OK，目前已经运行到此处了，那么开始重新计算绝对地址。

![](https://bbs.kanxue.com/upload/attach/202004/732177_VVGPQQ9DW9X8KZB.png)

![](https://bbs.kanxue.com/upload/attach/202004/732177_ZTXE33N8XEB7WQT.png)

绝对地址计算：`0xB38DD000 + 0xEDBC = 0xB38EBDBC`。

按 `G` 键跳转到此地址位置。

![](https://bbs.kanxue.com/upload/attach/202004/732177_XBYSTWB885C9DVZ.png)

这次可以清晰看到该函数的逻辑了，那么可以在这下断，并按 `F9` 运行到此处。

![](https://bbs.kanxue.com/upload/attach/202004/732177_M73RSUY35KQUHVB.png)

运行到此地址后开始按 `F8`，同时 Hex View 窗口可同步查看 `R0` 寄存器。

![](https://bbs.kanxue.com/upload/attach/202004/732177_PT62ZG2QYC6UNR5.png)

经过一阵子的 `F8` 之后，到了 `0xB38EBE3E` 这个地址，可以看到 `R0` 寄存器此时的数据开头为 `dex.035`。这就是此次需要 dump 的 DEX 数据了。

小框中的 4 个字节为 DEX 数据的大小。由于 Android 的文件采用的是小端编码，所以需要倒过来读，此时的数据大小为：`0x0FAEB4`。而开始位置为 `R0` 寄存器当前指向的位置 `0xB36C0000`。

![](https://bbs.kanxue.com/upload/attach/202004/732177_GQZ5ST43SZ4UGB3.png)

在 IDA 中使用脚本 dump DEX。

![](https://bbs.kanxue.com/upload/attach/202004/732177_WH66MK53V6GNAGZ.png)

## 5. 使用 JADX 工具验证从 IDA 中 dump 的 DEX 文件

毫无疑问，此次脱壳圆满成功。

![](https://bbs.kanxue.com/upload/attach/202004/732177_3DT7DEA2USTZ38V.jpg)

[[招生]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)
