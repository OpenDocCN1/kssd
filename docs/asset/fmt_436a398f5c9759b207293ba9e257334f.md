
# 汇编写驱动光速入门

> 原文：[`www.pediy.com/kssd/pediy12/131002.html`](https://www.pediy.com/kssd/pediy12/131002.html)

用汇编写 windows 驱动非常简单，只要有汇编基础和用 C 写驱动的能力即可。有了这两样基础，我们距离用汇编写驱动，只剩下一层窗户纸，一戳就破。

## 一、准备软件：Radasm

为了快速入门，好的 IDE 是降低入门难度的有效手段。Radasm 是我最为钟爱的一款软件，第一次使用过它，我就果断地将 `masm32` 删除了。

RadASM 是一款著名的 WIN32 汇编编辑器，精通 win32 汇编的人对它应该不会陌生。支持 MASM、TASM 等多种汇编编译器，提供优秀的 IDE，自带一个资源编辑器和一个调试器（OD）。拥有较强的工程管理功能，加之众多插件的支持，使得它用汇编语言编写 Windows 软件变得得心应手。

## 二、认识 KmdKit

之所以能用 Radasm 编写驱动，就是因为它整合了 KmdKit，它是由俄国人开发的。KmdKit 的全称是 Kernel Mode Driver development Kit for assembly language programmers，即内核模式驱动程序汇编开发包。KmdKit 中包括了用宏汇编编译器开发驱动程序需要用到的所有东西。你也可以从 `http://www.wasm.ru/` 或者 `http://www.freewebs.com/four-f/` 下载到最新版本的 KmdKit。

## 三、汇编写驱动的基本框架

```
.386

.model flat, stdcall

option casemap:none

.data

.code

;;;DriverEntry 这个入口函数还是老样子![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

DriverEntry proc pDriverObject:PDRIVER_OBJECT,pusRegistryPath:PUNICODE_STRING

        ret

DriverEntry endp

end DriverEntry  ;;;这里和用户模式有点不同![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")
```

## 四、可供使用的系统调用库

KmdKit 开发包提供的 inc 和 lib 库文件都分别保存在“w2k”“wnet”和“wxp”三个的文件夹里，其大部分内容都在 w2K 里。对应的，Radasm 也在 `masm32` 文件夹中的 `Include` 和 `Lib` 文件夹里，也分别保存了这三个文件夹。

另外，有一个特别值得注意的问题，当 Radasm 引用以上三个文件夹里 lib 文件时，要使用绝对路径，为什么必须要这样我也不清楚，如果有明白这个问题的朋友，也请你告诉我。

## 五、示例

```
.386

.model flat, stdcall

option casemap:none

include w2k\ntddk.inc

include w2k\ntoskrnl.inc

includelib D:\RadASM\masm32\lib\w2k\ntoskrnl.lib ;;请填写自己的绝对路径

.data

Hello_Str db 'Hello,world',0

Unload_Str db "Driver has been unloaded",0

.code

DriverEntry proc pDriverObject:PDRIVER_OBJECT,pusRegistryPath:PUNICODE_STRING

    push offset Hello_Str

    call DbgPrint

    add  esp,4

    mov  eax,pDriverObject

    mov [eax+34H],offset DriverUnloadProc ;;xp 里 pDriverObject.DriverUnload

    mov eax,STATUS_SUCCESS

    ret

DriverEntry endp

DriverUnloadProc proc  DriverObject:PDRIVER_OBJECT

    push offset Unload_Str

    call DbgPrint

    add  esp,4

    ret

DriverUnloadProc endp

end DriverEntry
```

一些收集的汇编驱动文档~希望有用

罗云彬翻译的前 7 章(貌似从 aogo 站上收集回来的)

附件中的 zip 文件。

第八章共享 section 通讯

`bbs.pediy.com/upload/2005/10/files/kmdtut_8.rar`

KMDTut 第九章共享内存,

`bbs.pediy.com/upload/2005/10/files/kmdtut_09.rar`

KMDTut 第十章注册表:

`bbs.pediy.com/upload/2005/10/files/kmdtut_10.rar`

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `kmdtutcn.zip`

 |
