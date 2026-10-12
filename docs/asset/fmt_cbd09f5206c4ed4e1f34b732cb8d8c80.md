
# 读写端口的方法总结 1

> 原文：[`www.pediy.com/kssd/pediy12/136270.html`](https://www.pediy.com/kssd/pediy12/136270.html)

//以下都是我的个人理解 有什么不妥之处欢迎跟帖指正

## 1 什么是 I/O 端口? CPU 使用什么指令与外设进行数据交换?

CPU 与 I/O 设备通过硬件接口或控制器相连接，这些接口或控制器都有数量不等的端口，这些端口有统一的地址编码，CPU 通过这些端口使用输入输出指令 `IN`、`OUT` 与外设进行数据交换。

## 2 读写端口有什么意义

其实读写端口就是在读写硬件了。在这几个小例子中，我仅仅是向 `CMOS`（注意与 `BIOS` 的区别：`BIOS` 存放在 `ROM`，`CMOS` 存放在 `RAM` 中，断电信息就会丢失）写入垃圾数据达到清除密码的目的。实际上端口是 CPU 与硬件交互的桥梁。举个具体的例子，比方说软件还原卡，分为两种吧：一种是磁盘过滤型的，比方说无盘系统常用的顺网；还有一种就是 `MBR` 型的，还原精灵。

其实都可以用读写硬盘的端口来绕过这些还原卡的保护。

大家都知道有一个软件叫 `MHDD`，很神奇，没有 `BIOS` 依然可以对硬盘进行各种操作。`BIOS` 提供了 `INT13H` 以及扩展 `INT13H` 标准（只支持到 128GB 的硬盘，如果没记错的话），现在用的应该是扩展的标准。很明显了，通过直接读写硬盘的端口来读取修改数据是可以绕过 `INT13h` 的限制的。采用端口读写硬盘这种技术的话，足以让很多基于接管 `INT13H` 的还原卡下岗了。

## 3 读写端口的方法

### @1 使用六个宏定义

`READ_PORT_UCHAR` `READ_PORT_USHORT` `READ_PORT_ULONG` `WRITE_PORT_UCHAR` `WRITE_PORT_USHORT` `WRITE_PORT_ULONG`

（其实 `VC` 中也有 `Inp`、`Outp` 这样的宏，汇编代码都是用 `in` `out` 实现的）

### @2 `VC` 中内嵌汇编代码

和 @1 中的原理是一样的。

### @3

`BOOLEAN Ke386QueryIoAccessMap(ULONG MapNumber, PKIO_ACCESS_MAP IoAccessMap);`
`BOOLEAN Ke386SetIoAccessMap(ULONG MapNumber, PKIO_ACCESS_MAP IoAccessMap);`
`BOOLEAN Ke386IoSetAccessProcess(PKPROCESS Process, ULONG MapNumber);`

未文档化函数的使用操纵 `IOPM`。

### @4 `NtSystemDebugControl` `ntsetinformationprocess` 等 windows 系统内功能繁杂的函数的使用

可以实现无驱读写端口。

### @5 如果要实现较为稳定和兼容性好点的读写端口可以用第三方库

非常好用。`WinIo` 目前可以支持到 `Win7` `X64`。

先把我总结的 @1 @2 的方法发上来，代码在 `XP` `SP3` 虚拟机上运行通过。

证明读写端口成功的标志就是 `CMOS` 的密码被清空了，所以提前先在给虚拟机的 `CMOS` 设置上密码。

## 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 我的帖子.rar |
