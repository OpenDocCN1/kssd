
# 一种 Windows CE 平台 AutoRun 程序的实现方法

> 原文：[`www.pediy.com/kssd/pediy10/79599.html`](https://www.pediy.com/kssd/pediy10/79599.html)

一种 Windows CE 平台 AutoRun 程序的实现方法

在 PC 平台有一类通过移动存储介质传播的自动运行病毒。当移动存储介质插入 PC 之后，如果用户双击设备就会自动运行病毒。

在 Windows CE 平台也有类似的自动运行程序。而且根据实验发现 Windows CE 平台的 AutoRun 程序甚至比 PC 平台更危险。只要用户插入染毒 SD 卡就会启动病毒不需要用户点击！

## 操作方法

01、在用户 SD 卡根目录下建立一个名为 `2577` 的目录。(`2577` 同当前系统 CPU 类型相关，如果是 `ARM CPU` 就是 `2577`。)

02、将希望自动运行的程序拷贝到 `2577` 目录下改名为 `AutoRun.exe`

将这个 SD 卡插入 Windows CE 系统时，`AutoRun.exe` 程序就会运行。这个实验可以在模拟器和真机上做成功。

## 本人声明

这项技术可以用于编写手机病毒。本人发布这项技术的研究成果只是提高大家的开发兴趣，同大家交流进行技术交流并不是鼓励编写病毒做坏事。

## 实验效果图及示例程序

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || AutoRun 程序实验图及样本程序.rar |
