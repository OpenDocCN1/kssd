
# 互动视音频点播系统 1.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue042.htm`](https://www.pediy.com/kssd/pediy04/kanxue042.htm)

**目标软件：**`IActVOD`(互动视音频点播系统) 1.3

**工具：**`trw2000`+`w32dasm`

**目的：**加入 `BCG`

**软件简介：**`IActVOD` 是一运用于局域网上，进行互动视音频点播系统。

支持格式：
1. `mpg`, `dat`
2. `mp3`

**下载地址：**`http://www.csdn.net/soft/openfile.asp?kind=1&id=12892`

**在服务端注册。**

**过程：**

`BCG` 的加入方法马上就要改变了，是要个什么 `CARCKME`！！倒！！！！我可不行，赶紧写一篇吧，希望 `CrackerABC` 和 `XY2000` 两位前辈高抬贵手，放小弟一马！！（真不要脸，一听变难了就吓成这样。＾＿＾）

不多说了，过程如下：

本想用 `TRW2000` 直接追出注册码，可是试了试又是没什么结果！！！（这孩子真笨，怎么也教不会＾O＾）

咦！！想起来了，风飘雪教程中有一篇什么“S 搜索大法”，我来试试！

运行程序

填入注册信息

咦？有序列号，看来只能我一个人用了，呵呵，不管它，先搞定再说！！

打入 `87654321`

`ctrl+n` 调出 `trw2000`

`bpx hmemcpy`

`g`

按注册 立刻被拦

`bc *`

`pmodule`

一直按 `F12`，直到错误提示出现

按“确定”回到 `trw2000` 下

`s 0,ffffffff '87654321'`

按 `alt+pagedown` 往下几行就会看到真正的注册码：`873288108`

**结果：**

填写注册信息 `873288108` 嘻嘻，当然成功了！

本想收笔，可一想一个人使多没意思呀，听说 `crackcode2000` 这个软件做注册机挺简单的，唉，硬起头皮，试试吧!

运行 `W32DASM`

调入 `VODSRV.EXE`(没有加壳 ^\_^ 爽!)

在串式参考中查找到“注册成功”

双击后来到：

```
:004014C6 55                   push ebp
:004014C7 8BEC                 mov ebp, esp
:004014C9 6AFF                 push FFFFFFFF
:004014CB 68AE004300           push 004300AE
:004014D0 64A100000000         mov eax, dword ptr fs:[00000000]
:004014D6 50                   push eax
:004014D7 64892500000000       mov dword ptr fs:[00000000], esp
:004014DE 81ECA4010000         sub esp, 000001A4
:004014E4 898D58FEFFFF         mov dword ptr [ebp+FFFFFE58], ecx
:004014EA 6A01                 push 00000001
:004014EC 8B8D58FEFFFF         mov ecx, dword ptr [ebp+FFFFFE58]
:004014F2 E833590200           call 00426E2A
:004014F7 8D45F0               lea eax, dword ptr [ebp-10]
:004014FA 50                   push eax
:004014FB E8E6720000           call 004087E6
:00401500 83C404               add esp, 00000004
:00401503 8B8D58FEFFFF         mov ecx, dword ptr [ebp+FFFFFE58]
:00401509 81C19C000000         add ecx, 0000009C
:0040150F E87C020000           call 00401790
:00401514 50                   push eax
:00401515 E8A74B0100           call 004160C1
:0040151A 83C404               add esp, 00000004
:0040151D 8945EC               mov dword ptr [ebp-14], eax
:00401520 8B4DF0               mov ecx, dword ptr [ebp-10]
:00401523 51                   push ecx
:00401524 E8D3740000           call 004089FC
:00401529 83C404               add esp, 00000004
:0040152C 3945EC               cmp dword ptr [ebp-14], eax   在 EAX 里放着真正的注册码!
:0040152F 0F8505010000         jne 0040163A
:00401535 6A00                 push 00000000
* Possible Reference to Dialog:
                              |
:00401537 6854D14300           push 0043D154
* Possible StringData Ref from Data Obj ->"注册成功！"  -----------往上看
                              |
:0040153C 685CD14300           push 0043D15C
:00401541 8B8D58FEFFFF         mov ecx, dword ptr [ebp+FFFFFE58]
:00401547 E87D510200           call 004266C9
:0040154C 6804010000           push 00000104
:00401551 8D95E4FEFFFF         lea edx, dword ptr [ebp+FFFFFEE4]
:00401557 52                   push edx
:00401558 6A00                 push 00000000
```

OK!填写 `crackcode.ini` 信息

```
[Options]
CommandLine=VODSrv.exe
Mode=1
First_Break_Address=40152C
First_Break_Address_Code=39
First_Break_Address_Code_Lenth=3
Save_Code_Address=EAX
```

**总结：**

把 `crackcode.exe` 和 `crackcode.ini` 拷贝到程序目录下，运行 `crackcode.exe` 呼出主程序，在注册信息里随便填写，按注册，`carckcode` 便把真正的注册码显示出来!太爽了!!!

打完，收工!

lllufh

2001-09-18 1:53 `4dd466d0e1e0cc123afbe504da2462ca`
