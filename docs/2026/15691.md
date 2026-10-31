
# [原创]开源 基于C++实现的 `armv8asmjit`

## 缘由

那时候也想转 android 端逆向。也想对 arm 指令研究一番。去看过 `capstore` 源码，里面做了很多优化，不方便学习。而且 `capstore` 对 arm 和 armv8 是完全分开处理的。从 android 6.0 开始，已经开始使用 armv8 架构了。觉得 armv8 以后是趋势。所以打算干脆拿 armv8 指令练手。本想写个反汇编引擎， 但 `capstore` 写的已经特别好了，写出来自己也不会用。也就提不起什么兴趣。由于那时候对 vmp 也比较感兴趣。拜读过 xiaowei 的 `Wproject` 代码，也自己参照 `Wproject` 实现过精简版（只模拟了几条指令，主要是为了研究学习），对 vmp 也理解个大概。但是 `wp` 不支持 arm，听说作者也不打算写了。然后就想要不自己写个 arm 版本的。就算写不成也能学习到不少。说干就干，第一个难题就摆在眼前。`Wproject` 的虚拟机引擎是由开源项目 `asmjit` 生成的。但是 `asmjit` 不支持 arm 和 armv8。没办法，只能自己参照手册写一个。

## armv8 A64

### 通用寄存器文件和堆栈指针

通用寄存器文件中的31个通用寄存器被命名为 `R0-R30`，并编码在指令寄存器字段，值为0-30。编码值31的通用寄存器字段表示根据指令和操作数位置，指定当前堆栈指针或零寄存器。当在特定指令变量中使用寄存器时，它们必须有资格指示操作数数据大小，32位或64位，以及指令的数据大小。当数据大小为32位时，使用寄存器的低32位，并在读取时忽略高32位并在写入时清零。

## AArch64

### 指令解析

和 X86 有所不同，X86 根据第一个字节能确定操作码。而 AArch64 需要根据类别。

AArch64 长度是固定 4 字节大小。根据第 25-28bit 来确定属于哪一大类指令。

AArch64 将指令归为以下几大类（如下图）:

![](https://bbs.kanxue.com/upload/attach/201709/624198_441hix4qz5bkzb9.png)

拿 `ADR` 指令举例：`ADR` 指令属于 `Data Processing-immediate` 这个大类下，`PC-rel. addressing` 这个子类中。

![](https://bbs.kanxue.com/upload/attach/201709/624198_w7ynyz870xczkni.png)

![](https://bbs.kanxue.com/upload/attach/201709/624198_ns53r77hrb5kglw.png)

从上图可以看到根据第 31 位决定是 `ADR` 指令还是 `ADRP` 指令。

## `armv8asmjit`

设计上参考 `asmjit`。`asmjit` 采用指令归组的方式进行代码优化。我能想到的是将指令机器码相近的归为一组。分为如下几组（如下图）：

![](https://bbs.kanxue.com/upload/attach/201709/624198_ruk05vypng7zn37.png)

## 重定位问题

AArch64 有如下几类指令是基于 PC 寻址的。需要进行重定位处理。每一类的指令处理方式不一样。

![](https://bbs.kanxue.com/upload/attach/201709/624198_iyktoynrmpqsb6e.png)

在这里我的做法是记录当前指令信息。保存到 `vector`。待需要重定位的时候，根据基址去遍历 `vector`，计算得到正确指令。

![](https://bbs.kanxue.com/upload/attach/201709/624198_ifuxqokwbq8c7s0.png)

## 范例测试对比

![](https://bbs.kanxue.com/upload/attach/201709/624198_jh7sk3tsvhlenda.png)

AArch64指令大概有接近200条指令，自己实现了大概160条指令，还有一些不常见的指令就没有添加了。放硬盘快一年了，也没有用到，干脆放到看雪供各位看官学习。代码在附件自行下载。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`Armv8AsmJit.rar`](javascript:void(0);)

  （801.57kb，188次下载）
