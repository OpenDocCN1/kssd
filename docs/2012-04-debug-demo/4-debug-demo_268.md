
# Private Pictures 2.0 破解分析

> 原文：[`www.pediy.com/kssd/pediy03/forum1121.htm`](https://www.pediy.com/kssd/pediy03/forum1121.htm)

## 基本信息

- **软件名称**：Private Pictures 2.0
- **作者**：PaulYoung
- **破解时间**：2001.06.30
- **软件简介**：图像加密工具。可以用密码保护你的图像文件，例如保护你的个人图片收集。可以使用程序自带的图片浏览器查看受密码保护的图像，而无需运行其他的图象浏览器；还可以将受保护的文件转换为其他的图象格式。图象加密支持 `JPG`, `PNG`, `BMP`, `GIF`, `TGA`, `TIFF`, `PCX` 和保护 `PPC` 文件。该软件是共享软件，当前版本提供 30 天的试用期。过期后完全不能使用，只有注册提示框，要网上注册。
- **下载链接**：`ftp://ftp02.softhouse.com.cn/download1/14700ppt2.zip`
- **破解工具**：`FileInfo`, `ProcDump` 1.5, `Hiew`, `W32DASM`, `SOFT-ICE`

## 破解步骤

1. 用 `FI` 侦察，得知其用 `Petite` 1.3 加的壳，可用 `ProcDump` 1.5 脱壳（1.62 反倒不支持了）。

2. `Ctrl+D` 激活 `SOFT-ICE`，下 `bpx getsystemtime`（因为该软件通过读取系统时间来判断是否过期的）。

3. 运行 `Private Pictures`，中断后下 `bd *` 中断所有断点，按 `F12` 6 下出现注册提示框，`be` 重来，按 5 下，`F10` 单步跟踪，直到……

   ```asm
   * Referenced by a (U)nconditional or (C)onditional Jump at Address:
   |:00426A15(C)
   |
   :00426A63 8D8C24C4010000 lea ecx, dword ptr [esp+000001C4]
   :00426A6A E8991B0800 call 004A8608 //弹出注册提示框，9090909090 NOP 掉它
   :00426A6F 83BC24F40400001E cmp dword ptr [esp+000004F4], 0000001E //比较是否大于 30 天
   :00426A77 7E40 jle 00426AB9 //改为 JMP 即 7E40 改为 EB40
   :00426A79 8D4C242C lea ecx, dword ptr [esp+2C]
   ```

运行一下，爽不爽，没有了注册提示框，也没有了时间限制。没办法，这类要网上注册的软件只能暴破！ `4dd466d0e1e0cc123afbe504da2462ca`
