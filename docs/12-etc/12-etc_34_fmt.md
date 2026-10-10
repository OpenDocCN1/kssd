
# Symbian S60 3rd Reverse CrAcKiNg Tutorial

> 原文：[`www.pediy.com/kssd/pediy10/87583.html`](https://www.pediy.com/kssd/pediy10/87583.html)

记得去年 `Cater` 曾经在某群里说过要写六个手机平台的解密教程  
只是 `Cater` 真的好忙 好辛苦的说  
`Symbian` 是我 06 年久接触了，那个时候 `S60` 2nd 版本的手机有碰过  
感觉 `ARM` 的汇编指令也不是很难，也过了几个软件  
时隔今日，塞班 `S60` 三版的(`OS 9.x`)的系统也总结总结些个教程  
满足下 某人 总是威逼 小 `Cater` 的野兽般的欲望吧。

--------------------------------------------------------------------------------

Symbian S60 3rd  
Reverse CrAcKiNg Tutorial

By_CaterQiu

Mail: Cater.Qiu\[At\]Gmail.Com

MyBlog: Http://www.caterqiu.cn

May 1th,2009

--------------------------------------------------------------------------------

## 基础知识

### Symbain OS

`Symbian OS`(中文译音“塞班系统”)由诺基亚、索尼爱立信、摩托罗拉、西门子等几家大型移动通讯设备商共同出资组建的一个合资公司，专门研发智能手机操作系统，现在已经被诺基亚收购。

`Symbian OS` 硬件 `CPU` 采用的是 `ARM` 系列，使用的是 `ARM 32` 位指令而非 `THUMB`。

### EPCO

`Symbian` 操作系统的前身是 `EPOC`，"EPOC”这个词起源于世界将会进入“a new epoch of personal convenience”。`EPOC` 是一个开放的操作系统，一开始的时候 `EPOC` 就加上了无线通信和一个外加应用程序的体系，因此在无线通信方面与其他操作系统相比具备先天的优势。

### Symbian S60

Nokia `S60` 系列手机是市面上智能机里流行最广的手机。

`Symbian Series 60` 是 `Symbian S60` 的全称

`S60` 可以细化分为第一版、第二版、第三版、第四版、第五版

第一版和第二版差别不大，笼统概括为 `S60 2nd`、`S60` 二版。包含 `OS6`、`OS7`、`OS8` 三个系列

第三版第四版第五版核心是 `OS9` 系列的，笼统概括为 `S60 3rd`、`S60` 三版

### 2nd And 3rd

`S60 3rd` 和 `S60 2nd` 从操作系统上看有很大的区别：

1、三版引入了权限签名这一安全机制，可通过对程序安装包进行签名授权。

2、`S60 2nd` 是 `EPOC6`，`S60 3rd` 是 `EPOC9`，这也意味着其 `ELF` `(Executable and Linking Format)` 是可执行连接格式也发生了改变

在 `S60 2nd` 版本中，可执行体是 `EPOC6` 格式的 `.APP` 文件和 `.DLL` 文件

在 `S60 3rd` 版本中，可执行体是 `EPOC9` 格式的 `.EXE` 文件和 `.DLL` 文件

3、三版和二版的程序安装包，格式是不一样的，尽管都是 `.sis` 结尾的文件

`EPOC9` 格式的可执行文件是可以压缩的，三版签名后的安装包一般是 `.sisx`

### ARM CPU

`ARM` 公司是专门从事基于 `RISC`（reduced instruction set computer，精简指令集计算机) 微处理芯片制造的企业。设计出的产品性能高、成本低和能耗省的特点，适用于多种领域，例如嵌入式系统、`DSP`、和手机。

`ARM` 处理器本身是 32 位设计，但也配备 `Thumb 16` 位指令集

--------------------------------------------------------------------------------

## 准备工作

### IDA Pro 5.2

鬼斧神工逆向分析利器

下载地址：[`bbs.pediy.com/showthread.php?t=55801`](http://bbs.pediy.com/showthread.php?t=55801)

### IDS files for EPOC6 and EPOC9

`Symbian EPOC` 格式标示符文件，配合 `IDA5.2` 使用，解压置 `IDA` 相关目录即可

下载地址：

[`arteam.biz.hr/downloads/Symbi...ro_by_argv.rar`](http://arteam.biz.hr/downloads/Symbian_EPOC6_EPOC9_Unleashed_IDS_Files_for_IDA_Pro_by_argv.rar)

### WinHex

强大的磁盘编辑工具，也是非常棒的十六进制编辑器。

### SisContents

`S60` 三版程序(`Sis,Sisx`) 打包、解包、签名工具。

[`symbiandev.cdtools.net`](http://symbiandev.cdtools.net)

### CeleASM

用于查看 `ARM OPCODE`

宇宙杰出青年 `Yonsm(郭春杨)` 之作

主页：[WWW.Yonsm.NET](http://WWW.Yonsm.NET)

### Symbian_OS_9.x-ELF_Toolz

用于解压和压缩 3rd `EXE` 和 `DLL` 工具

当然我不是宇宙杰出青年，博客还是有的

Blog： [WwW.CaterQiu.Cn](http://WwW.CaterQiu.Cn)

### RESEdit.exe

`S60` 三版程序 `RSC` 资源修改工具，主要用作汉化

这四款小工具我的博客有文件打包，详情参阅

[`WwW.CaterQiu.Cn/Article/Symbi..._CaterQiu.html`](http://WwW.CaterQiu.Cn/Article/Symbian_S60_3rd_Reverse_CrAcKiNg_Tutorial_By_CaterQiu.html)

--------------------------------------------------------------------------------

## 实战操作

`Example：DVDPlayer 1.26.SISx` （http://www.viking.tm）

### Setp 1：Unpack Sis/Sisx File

![](img/474ad4198592823420c249e05659ee10.jpg)

Use_SisContents_Unpack_S60_3rd_Target_Sis_File

### Setp 2：UnComPress ELF File

![](img/7177829282ee95743883e8d49b3eef0e.jpg)

Use_Symbian_OS_9.x_ELF_Toolz_UnComPress_Target_ELF_File

Compare File Size Between Original(BAK_dvdplayer.exe) And NoCompress(dvdplayer.exe),

You Have Found dvdplayer.exe Was Unpacked.

### Setp 3：Analysis Program Flow

![](img/20b2cf97773bbff0a56812e584f45710.jpg)

把脱壳后的程序拖入 `IDA` 中

![](img/57779e0ecd5b52ee6779a74c787750bd.jpg)

勾上 堆栈指针和机器码字节数为 8

![](img/da35368f8a5fa24f0ececb897af88e1e.jpg)

`Shift+F12` 设置下字符串类型，`Unicode` 要的

![](img/c52ec237abc253109fc3f13ec6639ebe.jpg)

字符串窗口中 `357062008960014`

类似 `IMEI` 的串号，双击过来

![](img/8d90076f31c16e3f647e988e933e60e9.jpg)

继续向上回溯字符串被调用的地方

![](img/2fed4c0c552430d9dfb7ae4503db863b.jpg)

回溯到到 `sub_8A2C`

注意：

`MOV R12,SP`

`STMFD SP!,{R4,R5,R7,R11,R12,LR,PC}`

这两句类似 `Win32` 程序汇编代码中

`PUSH EBP`

`MOV EBP，ESP`

`ADD ESP，XXX`

意味着什么呢？意味着这个是子功能函数的函数入口

![](img/a83633dd2e44c66767d048638edaaa30.jpg)

既然这个已经是子函数的入口了

我大致猜测刚才显示的那个串号是作者自己用来做的手机串号

我估计写入这个串号是为了检测

当串号为 `357062008960014`

就不需要启动软件注册功能

暂时这么猜测吧

按下 `X` 按键看看有哪些地方调用

还好只有一处

![](img/fe2a8b0633e2c5bcfa7508e4060d3035.jpg)

返回到这里

分析代码详细过程我就不在这里细说了

`Loc _8280` 这里与解密有关的部分，我们进入分析

![](img/fc2c11913aef12c972ef2fe283885071.jpg)

进来后，再进入 `sub_9114` 分析看看

![](img/9971d198c16a38814604d3d3ae156c79.jpg)

很显然 `sub_9114` 也是一个子函数

这里这号是图形化的，看看整个分支流程

![](img/cdf4cc91673c291295767c058fa3db6a.jpg)

CompareF 比较函数

再猜测下咯

刚才是作者手机串号的代码

这里的比较

很大何能是判断是不是作者的手机

`R0` 寄存器和 `8x86 EAX` 寄存器一样常用语函数返回值

没查 `SDK` 了，和 `Windows` 比较函数一样

一般返回 0 说明两参数比较相等

ATTACH\]25409\[/ATTACH\]

加个注解解释下吧

`LDR R0, [R6,#0x98]` ; 相当于 `MOV R0,[R6+0x98]`

`MOV R1, R10` ; 相当于 `MOV R1,R10`

`BL TDesC16::CompareF(TDesC16 const&)` ; 相当于 `Comparf(R0,R1)`

`CMP R0, #0` ; `R0` 为函数的返回值，为 0，说明参数 `R0` 和参数 `R1` 相等

`MOVEQ R8, #1` ; `IF R0=0 Then R8=1`

`STREQ R8, [R6,#0x90]` ; `IF R0=0 Then Storage R8 To R6+0x90 Memory`

`BEQ loc_922C` ; `IF R0=0 Then Call Loc_922C Function`

很明显要改变程序流程

需要 `patch` 下代码

修改

`CMP R0,#0`

变成

`CMP R0,R0`

可以一直作用下列的流程

自然而然

我们的补丁代码也就是这个了

### Setp 4：Make ARM OPCODE

![](img/5e16feb46f6b27dcce40b76d1bf740e1.jpg)

Use_CeleASM_Make_OPCODE

生成的 `OPCODE` 是

```
5 00000004 e3500000 CMP R0,#0
6 00000008 e1500000 CMP R0,R0
```

![](img/e6a822e9666dc5c1224a153cbac3c34c.jpg)

看看 `CMP R0,#0` 的 `OPCODE` 和我们途中的 `OPCODE` 是否相同

![](img/f1db96469181f54c80e821b56c0efb3c.jpg)

不难看出

`5 00000004 e3500000 CMP R0,#0`

是对应 `Little-Endianl` 数据存储类型的 `ARM CPU`

也就是低位字节排在内存较低地址

`Patch_OPCODE: 00 00 50 E1 (CMP R0,R0)`

### Setp 5：Patch Data

![](img/e6a822e9666dc5c1224a153cbac3c34c.jpg)

先从 `IDA` 中查看需要修改代码对应的文件偏移

`0x000012AC`

![](img/55d622c7e59d0a96542df19a0e6fb5f7.jpg)

原来的

![](img/4560ff54c0feb04532d8280b6c88938a.jpg)

修改后就这样了

最后存盘即可。

### Setp 6：Compress ELF File

![](img/1c225a5ec52d1e507ec2df7b65668347.jpg)

Use_Symbian_OS_9.x_ELF_Toolz_ComPress_Target_ELF_File

### Setp 7：Pack S60 3rd File

1、把之前破解好，加压缩的 `dvdplayer.exe` 复制一份到别处

2、删除之前解包出来的文件夹

`D:\Symbian_Reverse_Toolz\Toolz\SisContents121\DVDPlayer 1.26`

3、再次用 `SisContents` 打开原版的 `.sis` 文件，点击 `Extract Files` 图标

4、拷贝之前破解好，加压缩的 `dvdplayer.exe`，复制到

`D:\Symbian_Reverse_Toolz\Toolz\SisContents121\DVDPlayer 1.26\sys\bin`

替换掉解包出来的

5、删除原来的签名：回到 `SisContents` 中， `Tools->Delete Signatures`

6、`File->Save As->另存文件即可。`

### Setp 8：Run Patched .Sis File With Mobile

在手机上测试我们修改后的程序，破解成功。

说明那个串号是一个作者留下的小路吧，当然如果替换作者原来测试机的串号 `357062008960014` 成自己的串号，那么程序也是一路绿灯通行的。

## 学习小结

`ARM` 指令集需要掌握

遇到 `B` 当 `CALL` 指令用即可

`BL` 是调用系统函数

`Bxx` 是条件调用类似条件跳转 `Jxx` 指令

`R0-R3` 可以用作参数传递

`R0` 和 `Win32` 汇编中 `EAX` 作用一样常用语存放函数返回值

解密思路和 `Windows` 上的解密思路是一样的

可以通过字符串来定位关键代码，

也可以通过相关 `API` 找关键代码

### Symbian 解密注意事项

解包后记得先解压 `EXE` 或者 `DLL` 文件

修改后的 `EXE`、`DLL` 程序不可以直接替换到手机中使用，需要做成安装包

打包前一定要给 `EXE`、`DLL` 压缩下

打包时要注意签名，最好先删除所有的签名

[`WwW.CaterQiu.Cn/Article/Symbi..._CaterQiu.html`](http://WwW.CaterQiu.Cn/Article/Symbian_S60_3rd_Reverse_CrAcKiNg_Tutorial_By_CaterQiu.html)

Symbian S60 3rd Reverse CrAcKiNg_Tutorial

By CaterQiu

Blog [www.caterqiu.cn](http://www.caterqiu.cn)

Mail Cater.Qiu@Gmail.com

2009-5-1

--------------------------------------------------------------------------------

文中提到的附件我已打包

`Symbian_Reverse_Toolz.rar` [附件请到原帖下载：[`bbs.pediy.com/showthread.php?t=87583`](http://bbs.pediy.com/showthread.php?t=87583) ]

相关资料.rar

实话的说，本来教程想拿英文写的，顺带练习练习的

只是越写越困 越不通顺~所以后面 就用中文混淆一下吧~

顺带说下，明天是某人生日，某人很寂寞，先睡觉了。

感谢：水水、老 k、RoBinhood、我的群主
