
# [原创]虚幻4(ue4)引擎加密pak解包教程（初学者向x64源码逆向）

## 0. 前言：

遇到了喜欢的游戏，想提取壁纸，于是找quickbms等虚幻4引擎解包工具。无奈游戏加密，无法解包。不知密钥情况下，也没在网上找到现成的加密pak解包工具。想到Unreal4已经开源，就尝试着自己找一下密钥，也算是对照源码的x64逆向分析调试。以前接触的OllyDbg都是32位程序，64位稍有不同，这篇教程也算是我的初次接触x64dbg笔记了，包括了我对源码和编译器优化特征的理解，以及如何定位源码和反汇编代码的想法。

本教程仅适用于非魔改版引擎和未加壳源码对照分析，以ue 4.23为例。

注意，此教程仅用于逆向学习分析，请勿用于各种侵权行为。

完全初学者应该也能很容易看明白，欢迎大家讨论交流，如有错误请多指教~

本文内容与我在隔壁发的完全一样。

## 1. 观察源码，寻找切入点

我用的vscode，搜索关键字`decrypt`。我们发现`FAES::Key`结构，就是加密key，去epic官网查[5]发现是aes256加密。注意游戏版本要与源码版本一致，否则源码有可能修改对不上，还有此处不讨论魔改版ue4和程序加壳等问题。

![](https://bbs.kanxue.com/upload/attach/201911/617776_9UJXA7JCP7Y7ZP3.png)

![](https://bbs.kanxue.com/upload/attach/201911/617776_4XN4C7BV7AJ8ZEX.png)

看见`void DecryptData(uint8* InData, uint32 InDataSize, FGuid InEncryptionKeyGuid)`函数，右键`findallreference`找到相关信息，寻找切入点，我们看到文件`IPlatformFilePak.cpp`文件中`void FPakFile::LoadIndex(FArchive* Reader)`函数有fatal error的log，这个可以成为我们的关键点（因为是fatal，一般都会保留log，其他的log可能就没有编译到程序中[1]）

![](https://bbs.kanxue.com/upload/attach/201911/617776_PT97WE9K5DY3A93.png)

## 2. x64dbg寻找切入点

（为了减少break，在setting中只勾选entry和attach breakpoint，但是不知为什么还是在`raiseerror`中断？）右键搜索all module，reference string `"Corrupted index offset in pak file."`，可以定位`void FPakFile::LoadIndex(FArchive* Reader)`函数了。

![](https://bbs.kanxue.com/upload/attach/201911/617776_GH2S5XBP4EYHAZ8.png)

## 3. 分析函数与找到key

这部分最复杂也最麻烦，主要因为编译器的优化将很多函数变成了`inline`，

函数前面不一定非得有`push rbp`，`mov rbp,rsp`这种典型的语句，而且并不一定函数一定会有`call`，但是多个嵌套一般最多有嵌套个数个`call`。还要记住x64与x86反汇编的不同，基本上都是`__fastcall`，前四个参数为`rcx`，`rdx`，`r8`，`r9`（c++ this指针为`rcx` [2]），剩下的从右向左`push`，被调用者清理堆栈 [3]。

我们的目的是定位`void DecryptData(uint8* InData, uint32 InDataSize, FGuid InEncryptionKeyGuid)`，`FAES::DecryptData(InData,InDataSize, Key)`，函数找到`KEY`这个数据结构。不需要弄清楚加密原理（其实也没什么必要，应该是调用了标准的aes256加密算法），因此对照源码关注Jump类指令即可，一般`JGE`为`if(a<b)`, `JNE`为`if(!a)`。

有时也可以通过观察源码和反汇编看到跳过大段内容来定位。具体本例的切入点：

### 3.1

观察源码`bool bFisrtPass=True`，同时看x64dbg中的反汇编`mov r12b,1`，猜测`r12b`可能代表了`bFisrtPass`（编译器优化局部变量不一定调用堆栈，有些直接就用寄存器了)

![](https://bbs.kanxue.com/upload/attach/201911/617776_3PU5DKA4MV7H5N3.png)

![](https://bbs.kanxue.com/upload/attach/201911/617776_KNUQQKM6X5H264H.png)

### 3.2

一直往下拉吧，观察`r12b`相关的，看见`test r12b,r12b`，这个就是检查`r12b`是否为0，因此推断出下一句为`if (!bFirstPass)`，因为`jne`跳转后直接`cmd`顺腾摸瓜同时定位了`if (Info.bEncryptedIndex)`

![](https://bbs.kanxue.com/upload/attach/201911/617776_KAUURKS3XDC53JD.png)

### 3.3

对照源码`DecryptData`输入3个参数和反汇编`mov edx, lea r8, mov rcx`也是3个参数，因此推测光标处的`call`即为`DecryptData`函数。`Decrypt`条件语句中只有一个`call`，因为`GetData`直接取值了。

![](https://bbs.kanxue.com/upload/attach/201911/617776_SJ4TWDKP5AY3XX4.png)

![](https://bbs.kanxue.com/upload/attach/201911/617776_CS8RA8PZX3CTEMH.png)

### 3.4

`DecryptData`函数在反汇编中就比较混乱了，因为出现了大量的`inline`函数，好多小的跳转，我们先找大的跳转来定位。可以在graph视图下粗鲁看，然后在cpu中标记。

![](https://bbs.kanxue.com/upload/attach/201911/617776_JR7RTPMDY9PE3V2.png)

![](https://bbs.kanxue.com/upload/attach/201911/617776_MYCWMAZ35C6ET75.png)

![](https://bbs.kanxue.com/upload/attach/201911/617776_UJYGAQV6U3BB8R2.png)

### 3.5

最后找key就要耐心调试一下了，关键是注意`FPakPlatformFile::GetPakEncryptionKey(Key,InEncryptionKeyGuid)`，`FAES::DecryptData(InData,InDataSize, Key)`，这两步和key相关，调试的时候要关注`call`之前`ecx`（c函数，`rcx`为第一个参数），`r8`（第三个参数）存储地址指向的值（即`[rcx]`,`[r8]`），多次循环到此步骤是否相等来判断key。

还有key特点一般是256bit（即memory dump中的两列）看起来比较随机，一般不会出现特别整齐的。经调试并且对照三个参数的意义，发现倒数第二个`call`即为`FAES::DecryptData(InData, InDataSize,Key)` !)函数。到此在`[rcx]`的32字节即为所求的key。

![](https://bbs.kanxue.com/upload/attach/201911/617776_PPSTMF5PQWDTB9J.png)

## 4. 得到key后解包

用base 64得到asci字符串[6]，并且参考[1]的`crypto.json`文件用

```
[Unreal EngineInstall]/Engine/Binaries/Win64/UnrealPak.exe -Test [PAK FILE PATH]-cryptokeys=[LOCATION OF crypto.json]
```

指令即可解开pak文件。然后再用umodel来查看即可，一些文件可以转换为png等。

## Refer:

(1)<https://blog.jamie.holdings/2019/03/23/reverse-engineering-aes-keys-from-unreal-engine-4-projects/>

(2)<https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=vs-2019>

(3)<https://softwareengineering.stackexchange.com/questions/245668/where-does-this-go-in-a-x64-thiscall>

(4)<https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/x64-architecture>

(5)<https://docs.unrealengine.com/en-US/API/Runtime/Core/Misc/FAES/FAESKey/index.html>

(6)<https://base64.guru/converter/encode/hex>
