
# 一个简单病毒的分析

> 原文：[`www.pediy.com/kssd/pediy11/118848.html`](https://www.pediy.com/kssd/pediy11/118848.html)

生平在看雪的第一篇帖子，本人小菜，就拿一简单病毒入手，大牛们见笑....

## 病毒信息

- **病毒类型**： 后门
- **文件 MD5**： `2A1AEF106795864CA9DB643A116807DC`
- **加壳类型**： `UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`

## 病毒行为

### 1. 释放文件

`c:\WINDOWS\Fonts\wuauclt.exe`

### 2. 提升自身权限

![](img/5051ae6b478961592ff4a691714a4976.jpg)

### 3. 加载 urlmon.dll 并下载病毒文件

加载 `urlmon.dll`，得到 `URLDownloadToFile` 函数的地址。调用函数下载病毒文件 `http://360.1s.fr/ps.jpg` 到 `c:\windwos\fonts\gern.fon` 目录下。

![](img/1cf68ffaa0726bc20d2403d69c631938.jpg)
![](img/4ea2700ed1521f48bbb15f8acf3388c4.jpg)
![](img/a5db47a714b130322ff0f5aca2285a10.jpg)

### 4. 伪装与传播

在 `c` 盘根目录下生成 `sa.exe` 文件，设置文件为文件夹样式并设置为隐藏属性。运行 `sa.exe`，然后调用 `CMD` 命令结束 `wuault`(系统自动更新)进程，然后将病毒本身拷贝到 `system` 下的 `font` 目录中并重新命名为 `wuault.exe`。病毒运行 `font` 目录下的 `wuault` 程序后，调用 `cmd` 命令删除自体。

![](img/e13e368b695542a5fdc0010327d3ee4d.jpg)

```
0012FBFC   0040247E  ~$@.  /CALL 到 WinExec 来自 dumped\_.0040247C

0012FC00   0012FF20   .  |CmdLine = "cmd /c del "C:\Documents and Settings\commander\桌面\dumped\_.exe""

0012FC04   00000000  ....  \ShowState = SW_HIDE

0012FC08   7C930738  8   ntdll.7C930738
```

### 5. 设置 IE 首页

将病毒信息保存在 `c:\windows\font\gern.fon` 文件中，从这个文件中读取信息并设置为 `IE` 首页。

![](img/39c941eb9383233761d185c5869d7d4f.jpg)

### 6. 开启端口

开启 `1083` 端口等待网络连接..

## 附件

样本文件及加壳后文件。。因为是病毒所以加了密码，解压密码为：`xiaoju`

![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")

### 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `样本.rar` |
|---|---|
