
(8484e7d0) `nt!KiCallbackReturn` | (8484e864) `nt!Dr_kira_a`

Exact matches:

    `nt!KiCallbackReturn` = <no type information>

Index:002C, Attribute: ee, DPL=3, Type=000E(InternalType:20004), Selector:0008, Offset:8484E90C

(8484e90c) `nt!KiRaiseAssertion` | (8484e994) `nt!Dr_kui_a`

Exact matches:

    `nt!KiRaiseAssertion` = <no type information>

Index:002D, Attribute: ee, DPL=3, Type=000E(InternalType:20004), Selector:0008, Offset:8484F4A8

(8484f4a8) `nt!KiDebugService` | (8484f528) `nt!Dr_kit3_a`

Exact matches:

    `nt!KiDebugService` = <no type information>

Index:002E, Attribute: ee, DPL=3, Type=000E(InternalType:20004), Selector:0008, Offset:8484DFFE

(8484dffe) `nt!KiSystemService` | (8484e082) `nt!KiFastCallEntry2`

Exact matches:

    `nt!KiSystemService` = <no type information>

There are total 7 item(s).

====================================================================================

直接说下其它的一些结果，Task gate有3个，TSS有4个，3个对应着Task gate，1个是所有任务共用的。没有LDT，没有Call gate，没有Trap gate。

看过OS的基本数据后，想到一些不明白地方，

1. PE的文件加载时，各个节（如`.text`，`.data`）和保护模式的段是怎么对应的？从用户态进入内核态时，段（代码段，数据段等）的切换是怎么做的？`int 2e`或者`sysenter`一条指令就够了嘛？

2. 象`int 0`，`int 1`，`int 3`（对应的异常码是`#DE`，`#DB`，`#BP`）通常都认为是异常，为什么OS中却是使用中断门，而不是陷阱门来表示？

不知道答案，也不知道从哪下手。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
