
# 仙剑 5 简单内存修改器，附 C++ 源码

> 原文：[`www.pediy.com/kssd/pediy12/137270.html`](https://www.pediy.com/kssd/pediy12/137270.html)

昨天刚把仙剑 5 通关，总共花了大概 25 个小时，因为可能是我没练级，后面几个 BOSS 打不过。。。所以中途自己找了些数据，今天用这些数据写了一个简单的修改器，有需要的就拿走吧。。

数据：`base:930580`，`i` 为人物索引

```
[[base]+i*4]+500   //最大精
[[base]+i*4]+504   //当前精
[[base]+i*4]+508   //最大神
[[base]+i*4]+50C   //当前神
[[base]+i*4]+510   //当前气
```

金钱：`9305d8`

`5E1FA7`     //NOP 掉三个字节实现锁精

`5E46CA`     //NOP 掉三个字节实现锁气

![](img/5fb0ca841cbe04d88b799f7e7c57dfff.jpg)

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `Pal5 Cheat Tool.rar`
|
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `Pal5.rar`
|
