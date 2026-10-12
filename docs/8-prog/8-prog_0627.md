
# 使用 VC 创建 Windows NT 下的内核 DLL 和 LIB

> 原文：[`www.pediy.com/kssd/pediy10/102426.html`](https://www.pediy.com/kssd/pediy10/102426.html)

看到 hopy 大大写的用 `gcc` 创建 `NT` 下的 `dll`，想到好多人不习惯用 Linux 下的那东西，今天偶就仍块砖头。用 `vc` 和 `EasySYS` 创建 `windows NT` 下的内核 `DLL` 和 `LIB` 库。(偶的宗旨：简单的才是最好的，能不花钱解决的才是最牛的！)

## （一）创建内核 dll

用 `vc` 创建 `NT` 下的内核 `dll` 注意以下几点:

用 `EasySYS` 生成工程后在工程目录下会有一个文件名为 `sources` 的文件，用记事本打开修改一下

，内容如下：

```
TARGETNAME=NTOS

TARGETTYPE=EXPORT\_DRIVER

TARGETPATH=bin

DLLDEF=NTOS.def

INCLUDES=.\\

SOURCES = NTOS.c\\
```

主要改动是把 `TARGETTYPE` 改为 `EXPORT\_DRIVER`，然后自己创建一个\*.def 文件，文件中写要导出的函数的名字，如下：

```
EXPORTS

  DllInitialize PRIVATE

  DllUnload PRIVATE

 \_NtReadVirtualMemory

 \_NtWriteVirtualMemory
```

注意，`DllInitialize` `DllUnload` 后面必须加 `PRIVATE`，这两个函数，在 `dll` 加载和卸载的时候会分别调用。

## （二）创建内核 lib

`lib` 很有用，把你常用的函数做成 `lib` 以后可以少很多工作。生成 `lib` 其实很简单.

主要改动是把 `TARGETTYPE` 改为 `DRIVER\_LIBRARY`

很简单吧！下面附上例子源码

注意，工程路径不能有空格，如果编译出错请添加如下环境变量：`WNETBASE` = DDK 目录。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `NT\_DLL.rar`  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `NT\_LIB.rar`  

 |
