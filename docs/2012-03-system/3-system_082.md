
# 自己写个重定位表重建工具

> 原文：[`www.pediy.com/kssd/pediy11/127251.html`](https://www.pediy.com/kssd/pediy11/127251.html)

菜鸟最近在努力学习《加密》，

在 `DLL` 脱壳的重建重定位表环节遇到个坎。。。。。。

为了突破者坎，

跟着 13 章的思路自己写了个重定位工具，

其中也参考了《加密》的 `PE` 分析工具源码 。

下面写下大概的思路。。。

这个工具需要输入带有重定位 `RVA` 的 `BIN` 文件，每个 `RVA` 以 `DWORD` 对齐，最后以一个为 0 的 `DWORD` 结束

程序循环读取每个重定位 `RVA`，计算得到 `IMAGE_BASE_RELOCATION` 中的 `VirtualAddress`、`TypeOffset` 和 `SizeOfBlock`；

```
typedef struct _IMAGE_BASE_RELOCATION {

    DWORD   VirtualAddress;

    DWORD   SizeOfBlock;

//  WORD    TypeOffset[1];

} IMAGE_BASE_RELOCATION;
```

其中,

`VirtualAddress`=`重定位 RVA`/`0x1000`*`0x1000`;

`TypeOffset`=`重定位 RVA`%`0x1000`+`0x3000`;     `0x3000` 表示高 4 位的标志

`SizeOfBlock` 为整个重定位块的大小

以链表的形式储存以上数据

再写入到文件的指定 `RVA` 处

最后修改下 `OPTIONAL_HEADER` 中的重定位数据目录的地址和偏移；

总之，菜鸟表示很努力学习，求鼓励，求邀请码。。。。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `relocREB.rar` |
|---|---|
