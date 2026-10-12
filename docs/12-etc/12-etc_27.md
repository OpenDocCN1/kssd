
# S60 软件分析的两个小技巧

> 原文：[`www.pediy.com/kssd/pediy10/60537.html`](https://www.pediy.com/kssd/pediy10/60537.html)

最近换了个 NOKIA 手机玩，碰到个 S60 软件需要注册，于是重拾 IDA，偶有所得，与大伙分享一下。这里不谈具体软件，只以 S60 SDK 自带的 Hello World 为例。

## (1) 库序号调用 -> 库中具体函数名

反汇编 Hello World 后。注意里面很多系统 API 都是通过链接库中序号来导入的

代码:

```
.text:00008DAC                 MOV     R4, R0
.text:00008DB0                 SUB     SP, SP, #4
.text:00008DB4                 BL      avkon_131
```

由于 S60 下软件基本只能靠静态分析，这给我们的分析带来了很大不便，如何才能看到真正的函数名呢。首先我们找到该库，在 SDK 安装目录下\`\\Symbian\\9.2\\S60_3rd_FP1\\Epoc32\\release\\armv5\\lib\\`下的 `avkon.dso` 和 `avkon.lib` 两个文件。这里 dso 文件顾名思义就是动态链接库吧。我们先来看看其中的符号信息，

代码:

```
arm-none-symbianelf-readelf.exe -a -W avkon.dso > avkon.txt
```

输出中我们可以看到有序号和函数名字的对应关系。

代码:

```
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000     4 FUNC    GLOBAL DEFAULT    1 _Z11ApacPlain12v@@avkon{000a0000}[100056c6].dll
     2: 00000004     4 FUNC    GLOBAL DEFAULT    1 _Z11ApacPlain16v@@avkon{000a0000}[100056c6].dll
```

总不能对每个函数我都手动来修改其对应函数名吧，能否将将 IDA 反汇编中的函数名字给替换掉呢？通过万能的 google，找到了下面现成的脚本

`http://www.zhongts.net/MyEssay/ida.lib.htm`，

对脚本略作修改，并将 txt 文件中无用的列去除后，执行该脚本。那讨厌的库函数序号终于都变为为函数名了，这样看起来是不是就舒服多了呢。

代码:

```
.text:00008DAC                 MOV     R4, R0
.text:00008DB0                 SUB     SP, SP, #4
.text:00008DB4                 BL      _ZN12CAknDocumentC2ER15CEikApplication  ; CAknDocument::CAknDocument(CEikApplication &)
```

## (2) 字符串 -> 字符串引用之处

如何才能快速找到注册码校验之处呢，首先我们就想到了注册码错误提示字符串调用之处。

由于多国语言支持的考虑，手机软件中所用的字符串一般都存储于 rsc 资源文件中。我们先来看看资源文件是如何被调用的。

代码:

```
.text:000086E4                 MOVL    R1, 0x277C4000
.text:000086EC                 ADD     R1, R1, #0xA
.text:000086F0                 LDR     R0, \R6,#4
.text:000086F4                 BL      _ZNK7CCoeEnv26AllocReadResourceAsDes16LCEi  ; CCoeEnv::AllocReadResourceAsDes16LC(int)
```

其对应的程序源码就是类似于

代码:

```
// Load a string from the resource file and display it
HBufC* textResource = StringLoader::LoadLC( R_HEWB_COMMAND1_TEXT );
```

这里这个 `0x277C4000` + `0xa` 就是该字符串对应的资源 ID，如何把这里这个资源 ID 和具体资源对应起来呢。由于 Symbian 未提供 rsc 文件的文件格式，我们只能自己来分析了。

首先搜索 rsc 资源件很容易就找到了该字符串，其第一个字节为该串长度。（注： 由于在 rsc 文件中字符串起始位置不一定是双字节对齐，对中文字符串需按二进制来搜索）

代码:

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F

000001B0   00 00 00 FF FF 00 00 00  00 00 00 00 0C 48 65 6C              Hel
000001C0   6C 6F 20 77 6F 72 6C 64  21 05 41 62 6F 75 74 59   lo world! AboutY
```

OK，我们在这里确实找到了出错提示串，其具体的 ID 号还要继续分析。

注意 RSC 末尾有很多两字节数，看上去很像索引数组什么的。

代码:

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F

00000240   1D 00 22 00 3E 00 8D 00  A5 00 23 01 65 01 BC 01     "  >  ??# e ?
00000250   C9 01 CF 01 29 02 3E 02                            ??) >
```

经过分析确认最后 half word3E 02，其实也就是该表的索引其实位置指针，对应于文件偏移 `0x023e`。同样的数组中其他值也是文件内部偏移位置。

0015

001d

0022

..

我们看到 `0x01bc` 在这个数组中索引为 9, 这样我们就知道了该字符串的索引为 9。但有个小问题是，每个资源文件的起始地址是不同，这个是在编译资源文件时动态生成的。

代码:

```
#define R_HEWB_COMMAND1_TEXT                     0x277c4009
#define R_ABOUT_DIALOG_TITLE                     0x277c400a
```

在浏览 rsc 文件时我们发现资源文件的第一行中有出现 C4 77 02 这几个 byte 内容，

这里我们我们大胆推断 `0x6.5` 字节偏移开始的 WORD 就是该资源文件起始 ID 的定义。

代码:

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F

00000000   6B 4A 1F 10 00 00 00 00  C4 77 02 00 CB 07 86 79   kJ        ?
00000010   01 B2 00 AA 0F 04 00 00  00 01 40 7C 27 04 48 45    ??     @|' HE
```

那么我们现在有了该资源 ID: `0x277c4000` + `0x9` = `0x277c4009`

然后就可以在 IDA 中搜索这个立即数，来找到其引用位置了。不过具体对于 hello world 这个程序，其索引值不是简单的一个 ldr 而是一条 mvn 一条 add 指令。这样就只能搜索 `0x277c4000` 了。

容易找到了这里，这里就是弹出 Hello world 对话框的代码了。

代码:

```
.text:0000876C loc_876C                                ; CODE XREF: .text:00008680j
.text:0000876C                 MOVL    R0, 0x277C4000
.text:00008774                 MOV     R1, #0
.text:00008778                 ADD     R0, R0, #9
.text:0000877C                 BL      CONE_11
.text:00008780                 MOV     R7, R0
.text:00008784                 MOV     R0, #0xBC
.text:00008788                 BL      _ZN4User7AllocZLEi  ; User::AllocZL(int)
.text:0000878C                 MOV     R5, R0
.text:00008790                 BL      _ZN19CAknInformationNoteC1Ev  ; CAknInformationNote::CAknInformationNote(void)
.text:00008794                 MOV     R1, R7
.text:00008798                 MOV     R0, R5
.text:0000879C                 BL      _ZN22CAknResourceNoteDialog9ExecuteLDERK7TDesC16  ; CAknResourceNoteDialog::ExecuteLD(TDesC16  const&)
.text:000087A0                 BL      _ZN12CleanupStack13PopAndDestroyEv  ; CleanupStack::PopAndDestroy(void)
.text:000087A4
```

另外对 S60 的保护一般都还比较简单，所以也可以通过修改程序代码后运行查看运行情况，甚至更改流程直接打印出明码比较的注册码，这也是不难办到的，这就要发挥你的想象力啦。
