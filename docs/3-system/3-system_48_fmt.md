
# 为程序增加区段

> 原文：[`www.pediy.com/kssd/pediy10/88243.html`](https://www.pediy.com/kssd/pediy10/88243.html)

给和我一样的菜鸟![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

不修改原来的文章了

在这里总结一下,步骤如下

1. 构造新的区段头

2. 增加实际空间

3. 修正 `NumberOfSections` 和 `SizeOfImage`

`NumberOfSections`(此值位于 PE 头开始的偏移 `6H` 处, `WORD`)你添加几个节就在原来的基础上加几

`SizeOfImage`(此值位于 PE 头开始的 `50h` 处, `DWORD`)则根据增加的 `Virtual Size` 做相应修改

手工增加后的 `test.exe` 已经上传

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `test.rar` |
| :--- | :--- | :--- |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `为程序增加区段.rar` |
| :--- | :--- | :--- |
