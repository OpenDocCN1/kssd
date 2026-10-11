
# 金蝶财务软件行政事业版 V6.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50192.htm`](https://www.pediy.com/kssd/pediy05/pediy50192.htm)

软件名：金蝶财务软件行政事业版 V6.2   
作者:qdbenben   
转载:大老   
原贴:http://dalao2002.yeah.net   
欢迎转载!但要保留完整谢谢!   
加密狗类型：彩虹圣天诺狗   
加密方法：API 方式。   
解密方法：带狗解狗。   
使用工具：`TRW2000`，`W32DASM`，`Delphi5.0`

相当长一段时间之前做过金蝶标准版 7.0 的解密，当时因为时间紧，只好对软件进行暴力破解，由于暴破时对软件主文件进行了多处更改，破解后又没有时间去进行验证，不知道对软件功能有没有损害。此次破解行政事业版，时间上没有要求，所以笨笨便回避了上次的作法，转而从狗和软件之间去寻找突破口。于是便有了以下的破解思路及过程。

### 破解基本思路

采用 API 方式加密的圣天诺狗需要使用一个名为 `sx32w.dll` 的动态链接库，金蝶软件也不例外，只是行政事业版的开发者将其改名为 `kdsx32w.dll`,文件存放于`\\Windows\\system` 目录下。

`Sx32w.dll` 中包含了软件读取圣天诺狗所使用的所有过程和函数，所以可以这样讲，`sx32w.dll` 是软件与狗之间的一座桥，软件就是通过这座桥与狗进行的数据交换。

笨笨这次就是要从这座桥上做做文章，所以本次破解的目标也就是——`kdsx32w.dll`。

### 破解过程

1、使用 `TRW2000` 载入软件的主文件 `kdwin.exe`。

下断点 `bpx deviceiocontrol`，很快就会发现已经进入 `kdsx32w.dll` 的空间，并且，软件在 `kdwin.exe` 和 `kdsx32w.dll` 之间有一个多次的反复判断、执行过程。最后，软件启动登录界面或出现错误提示信息。

2、用 `W32DASM` 反汇编 `kdsx32w.dll`,查看其内部的过程并区分各过程。

3、再次用 `TRW` 载入 `kdwin.exe`，跟踪至 `kdsx32w.dll` 入仔细查看，发现 `kdsx32w.dll` 中过程的调用次序如下：

a) `RNBOsproFORMatPacket` 出错时-->`Error`

b) `RNBOsproInitialize` &nbsp; 出错时-->`Error`

c) `RNBOsproGetVersion` &nbsp; 出错时-->提示：驱动版本低-->`Error`

e) `RNBOsproFindFirstUnit` 出错时-->`Error`

f) `RNBOsproRead`

出错后转向：`RNBOsproFindNextUnit` -->`Error`

正确时：反复执行多次 `RNBOsproRead`，然后转向 g)

g) `RNBOsproQuery`

出错后转向：`RNBOsproFindNextUnit` -->`Error`

正确时：

`Pass!!`

分析以上流程发现，软件开发者将密码的查询串写入了狗中，先通过 f)处 `Read` 过程依字节读出，然后再将读出的字节合并后进入 g)处 `Query` 过程进行变换，再取出变换值与密码的验证串进行比较，一致后便通过。

4）仔细阅读 `kdsx32w.dll` 中的函数汇编代码，明确其使用的参量及返回值后，用 `delphi` 重新构造了一个 `kdsx32w.dll`,调试通过后用其替换原来的 `kdsx32w.dll`。运行软件，通过。 4dd466d0e1e0cc123afbe504da2462ca
