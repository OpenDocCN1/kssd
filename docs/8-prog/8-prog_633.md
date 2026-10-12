
# 学习小结：协议驱动的安装

> 原文：[`www.pediy.com/kssd/pediy10/97724.html`](https://www.pediy.com/kssd/pediy10/97724.html)

比较简单的东西，自己总结一下，发出来，方便日后复习。关于协议驱动的例子，`DDK2000` 中有 `Packet`，`DDK3790` 中有 `ndisprot`，我用的是 `DDK3790`。

编译后将 `ndisprot.sys` 和 `ndisprot.inf` 放到同一目录下。

## 一：手动安装

右键单击“网上邻居”，选择属性菜单，在打开的“网络连接”界面中选择一个“本地连接”，右键单击选择属性弹出属性对话框。然后依次点击“安装”-》“协议”-》“添加”-》“从磁盘安装”-》“浏览”，找到 `ndisprot.inf` 文件，点“打开”。最后一路点击“确定”和“关闭”完成协议驱动的安装。

## 二：自动安装

上述手动过程完全可以用程序自动化完成，感兴趣的朋友可以参看附件的 `InstallProt.rar`。代码主要参考 `bindview`，使用时，自己修改一下协议驱动的 `.inf` 文件路径，编译后运行，程序会自动安装协议驱动，安装成功完成后，会弹出安装成功的提示对话框，点确定后会又自动卸载。

## 三：一种邪恶的方法

在 `DriverEntry` 里面调用 `NdisOpenAdapter` 直接打开网卡，这样可以将协议驱动当作普通驱动一样加载了，附件 `BasicProt.rar` 是示例代码，主要参考《Rootkits: Windows 内核的安全防护》。

### 上传的附件

| 预览 | 文件 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `BasicProt.rar` |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `InstallProt.rar` |
