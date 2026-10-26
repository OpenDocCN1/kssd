
# [原创]Windows逆向学习笔记——破解Source Insight 4

[TOC]

## 前言

业精于勤，学习Win32逆向课程已有月余，为增功力，故研究po jie此SourceInsight4程序。论坛中有多位大佬写过SI4 po jie文章，此文自不可与之相较，只求叙述清楚几个po jie的关键点。

学必有师，笔者师从科锐钱老师。师者，技艺精湛自不再话下，诲人不倦耐心教导着实难能可贵，在此拜谢。

本文po jie的是当前最新版本Source Insight 4.0.118，总结po jie过程有以下三个部分：

1. 逆向Serial Number序列号验证算法，构造可用序列号
2. 逆向分析验证license签名过程，自签名license文件，并使用自己的公钥替换程序中原始公钥
3. 程序运行起来后，会单独开启一个线程在线检查Serial Number是否有效，需要修改二进制文件过掉此检查

除此之外，笔者也看了一些大佬的文章，提到有黑名单检查，因为没遇到这个逻辑，本着能用就行的原则，所以这里的po jie中没有涉及，后面如果遇到再研究也不迟。

文中用到的所有代码可点击右侧超链接，在 <a href="elink@013K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6h3&6Y4P5U0N6Q4x3V1k6x3k6h3q4J5L8W2)9J5k6q4N6A6L8X3c8G2N6%4y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3e0r3g2S2M7X3&6o6M7X3q4U0K9#2)9J5c8V1y4J5j5h3y4C8f1@1V1@1">Github</a>上查看

## 环境准备

`IDA6.8`、`x32dbg`

<a href="elink@935K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8%4g2J5j5$3g2A6L8Y4y4A6k6$3S2@1i4K6u0W2j5$3!0E0i4K6u0r3">官网</a> 下载`sourceinsight40118-setup.exe`

## 0、简单po jie：修改两字节

因为本文主要是研究自签名license并替换程序中公钥的po jie方法，比较麻烦，纯粹是为了学习目的。如果对这种方法不感兴趣，这部分内容就提供另一种简单的po jie方法：修改 `sourceinsight4.exe` 二进制文件中的两个字节完成po jie

- 第一处是 `*VA=005141A8*`，`*FA=1135A8*`，将 `74` 改为 `EB`，原因可参考 `*第 2.2 节*`

- 第二处是 `*VA=00513470*`，`*FA=112870*`，将 `83` 改为 `C3`，原因可参考 `*第 3 节*`

最后，复制本文 `*2.3节*` 中 `*si4.lic*` 的内容，并保存为 `*C:\ProgramData\Source Insight\4.0\si4.lic*` 文件即可po jie程序

![在这里插入图片描述](upload/attach/202008/897338\_S4VYU6JWDX4ADAJ.jpg)

## 1、验证序列号的过程——逆向分析

这部分内容分4个小节：

1.1 节 讲述如何定位到验证序列号的代码

1.2 节 中逆向还原验证序列号格式的函数，并按照格式构造出正式版序列号

1.3 节 中逆向还原本地校验序列号的函数，构造出可通过本地校验的序列号

1.4 节 本地验证序列号

### 1.1 定位“Serial Number错误”弹窗代码

安装程序后，打开`sourceinsight4.exe`程序会看到选择license文件的界面，如下图：

![在这里插入图片描述](upload/attach/202008/897338\_4AMMCFTAWQAVAHN.jpg)

选择第一项“输入serial number”，点击“Next >”继续，显示输入Serial Number的对话框，并且提示 Serial Number 的格式“S4XX-XXXX-XXXX-XXXX”。按照格式输入一个，点击"Next>"，会弹出“无效的Serial Number”警告，如下图：

![在这里插入图片描述](upload/attach/202008/897338\_NNR9G3QYVUA7BQP.jpg)

在IDA中搜索弹窗警告中的文本“The serial number ...”，找到引用此字符串的地址是 `*.text:00513A69*`。

分析`*.text:00513A69*`此位置上下文代码，可以看出 `*.text:00513A45 call sub_510B50*` 是Serial Number的验证函数，若验证失败（返回结果0），则会再判断是否是3.x版本的Serail Number，不是则弹出上图中的警告；若验证成功则跳转到 `*loc_513ACB*` 继续判断验证结果。代码如下所示：

```c
.text:00513A25 mov eax, dword_673488
.text:00513A2A push 1 ; int
.text:00513A2C lea ecx, [eax+604h]
.text:00513A32 push ecx ; int
.text:00513A33 lea edx, [eax+60Ch]
.text:00513A39 push edx ; int
.text:00513A3A add eax, 608h
.text:00513A3F push eax ; int
.text:00513A40 lea eax, [esp+158h+MultiByteStr]
.text:00513A44 push eax ; char *
.text:00513A45 call sub_510B50 ; 验证Serial Number是否正确
.text:00513A4A add esp, 34h
.text:00513A4D test eax, eax ; eax = 0, 弹出错误窗口并返回
.text:00513A4D ; eax!= 0, 继续验证 Serial Number格式
.text:00513A4F jnz short loc_513ACB
.text:00513A51 lea ecx, [esp+128h+MultiByteStr]
.text:00513A55 push ecx
.text:00513A56 call sub_561CB0
.text:00513A5B add esp, 4
.text:00513A5E test eax, eax
.text:00513A60 jz short loc_513A69
.text:00513A62 push offset aTheSerialNumbe ; "The serial number you entered is for ve"...
.text:00513A67 jmp short loc_513A6E
.text:00513A69 ; ---------------------------------------------------------------------------
.text:00513A69
.text:00513A69 loc_513A69: ; CODE XREF: sub_5139C0+A0j
.text:00513A69 push offset aTheSerialNum_0 ; "The serial number you entered is not co"...
.text:00513A6E
.text:00513A6E loc_513A6E: ; CODE XREF: sub_5139C0+A7j
.text:00513A6E call sub_40AC20
.text:00513A73 add esp, 4
... ...
.text:00513ACA retn
.text:00513ACB ; ---------------------------------------------------------------------------
.text:00513ACB
.text:00513ACB loc_513ACB: ; CODE XREF: sub_5139C0+8Fj
... ...
.text:00513AE1 cmp [eax+604h], edx
.text:00513AE7 jz short loc_513B0D
.text:00513AE9 push offset aTheSerialNum_1 ; "The serial number you entered is for a "...
.text:00513AEE call sub_40AC20
... ...
.text:00513B0C retn
```

### 1.2 Serial Number验证函数 分析

详细分析上面步骤中提到的 `*sub_510B50*` 处的序列号验证函数，总结出Serial Number的验证规则如下：

SerailNumber字符串长度必须为19（16个字符加上3个分隔符）

SerailNumber[0]必须等'S'

SerailNumber[1]是'0'-'9'

SerailNumber[2]等于'T'表示Trial license试用许可，等于'B'表示Beta license测试许可，等于'S'表示Standard license标准许可，等于'U'表示Upgrade license升级许可

SerailNumber[3]等于 'G'、'V'、'R' 其中之一

SerailNumber[6]等于 'R'、'G'、'D'、'F' 其中之一

SerialNumber的前12个字符经过函数 `*.text:00510C6B call sub_510320*` 转换后得到4个字符，与SerialNumber的最后4个字符必须相同

代码如下所示：
