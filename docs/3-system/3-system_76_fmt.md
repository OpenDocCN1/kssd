
# 对目标程序自带 DLL 输出表加密\[开源\]

> 原文：[`www.pediy.com/kssd/pediy12/129979.html`](https://www.pediy.com/kssd/pediy12/129979.html)

初学 C++用 MFC 写的第一个小程序，源码写的可能不太好看，有什么地方写的不好，请一定要指出，在此感谢帮助过我的朋友！[](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")  

程序虽然比较简单，不过第一次用 C++写程序，也花了不少时间，支持下哈！[](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")  

本程序是单向加密，如果你加密的目标程序中，有一个 `DLL` 调用另一个 `DLL`，或者有另一个应用程序调用某 `DLL`，本程序加密后必定会导致另一程序无法载入 `DLL` 函数！所以在加密前要确认，你的 `DLL` 只被该目标程序调用！  

使用方法及效果看下面的图就会了解：  

![](img/53bf8d7e233e9df4211facdb931336f5.jpg)  

![](img/77b985f74a45ecb40a56f8b4faccb143.jpg)  

![](img/fb1c6e9e52bd700e4005d232b5b5db2c.jpg)  

![](img/9c21d5aaa63456f9ac38e3ee9fb99989.jpg)  

![](img/710ad93b857f25e0c5d26328cad5cb3c.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `LocalDLLExportEncrypt.rar`  
|
