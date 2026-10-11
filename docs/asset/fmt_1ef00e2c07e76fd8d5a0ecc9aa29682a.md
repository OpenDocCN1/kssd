
# Speed Video Converter 3.1.12 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/73289.html`](https://www.pediy.com/kssd/pediy10/73289.html)

## 元信息

- **破文标题**：Speed Video Converter 3.1.12 算法分析
- **破文作者**：tianxj
- **作者邮箱**：`tianxj_2007@126.com`
- **作者主页**：WwW.ChiNaPYG.CoM
- **破解工具**：`PEiD`, `OD`
- **破解平台**：Windows XP
- **软件名称**：Speed Video Converter 3.1.12
- **软件大小**：2592KB
- **软件类别**：国外软件/视频转换
- **软件语言**：英文
- **更新时间**：2008-9-8
- **原版下载**：自己找一下
- **保护方式**：注册码

## 软件简介

Speed Video Converter 是一款小型视频转换工具。快速的转换速度和易用的用户界面是其卖点。Speed Video Converter 支持各种视频格式，例如 AVI(Divx, xDiv), MPEG-4, mpeg(vcd, svcd, dvd 兼容), wmv, asf, QuickTime, VOB, DAT。它支持批量文件转换可以一次点击转换多个文件。

## 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

### 一、运行程序，进行注册

输入错误的注册信息进行检测，有提示信息：

> "Invalid username or registration code"

### 二、查壳

用 `PEiD` 对 `Speed Video Converter.exe` 查壳，为 Microsoft Visual C++ 6.0。

### 三、动态分析

运行 `OD`，打开 `Speed Video Converter.exe`，右键超级字串参考查找 ASCII。发现 "Invalid username or registration code"。

```
代码:
```
