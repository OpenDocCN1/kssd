
所以为什么要比较`0x39`和`0xf0`?我们知道这里的比较是加密后比较的，加密前，`0x39` ^ `0x26` = `0x1f`, `0xf0` ^ `0x26` = `0xd6`。

所以他本质是检查原始的值是不是`0x1f` 和 `0xd6`。为什么检查这两个值？

我们看看`arm64`的指令手册，对于`BR`（寄存器跳转）指令：

![](upload/attach/202307/872365_NDJBM83FMXUNP76.jpg)

昭然若揭！

`br`指令的高16位是`0xd61f`！！！

所以该加固其实是在检查`_exit`函数的第二条指令是不是`br`指令。通常`inline hook`第一条指令是`mov`常数到寄存器，然后第二条是一个`br`寄存器指令。检查第二条指令高16位是不是`0xd61f`,就可以判断目标函数是否被`inline hook`了！

由于`frida`自己`hook`了`exit`和`_exit`函数，虽然断点指令检查通过了，但是`hook`检查没有通过，程序造`crash`退出了。不过该加固在检查前是先加密，然后比较加密后的结果，有点骚。

知道这个之后也好绕过了，只需要把比较的`0x39`和`0xf0`（加密后的）换成其他的值就可以绕过了。然后重新压缩，加密，回填到原`so`中即可，不在赘述。

至此，该加固的加固方式（多映射内存，然后解密解压填充），反调试（`Tracerid` + 断点指令），反`hook`（`br`指令检查）全部绕过了，可以随意调试，`hook`该游戏了。

## 4. 后记

这次研究的是日本的游戏和加固，所以写的比较露骨。游戏名字叫 ブルーロック 咱也不知道是啥意思hh。看到了`apk`里面的`nhnent appguard`,不知道有没有朋友知道是什么。不过本质是一个加密的压缩壳，这在手游保护里不太常见。反调试的断点指令检测手法和反`hook`的`br`指令检测手法也很有新意，不是常规的路数。

希望这篇文章能对大家学习检测手法，反调试与绕过有一些帮助，也希望给国内手游保护厂商提供一些外面人的思路。

## 0. 前言

前两天水友群里老哥遇到了一个游戏样本：

![](https://bbs.kanxue.com/upload/attach/202307/872365_GAXUXKWZFP996FP.jpg)

我点开一看，符号都没去卧槽？？！什么逆天保护：

![](https://bbs.kanxue.com/upload/attach/202307/872365_8SUT3699DGB9T37.png)

所谓“浅藏诲盗，冶容诲淫”，保护做成这样不就是自找破解吗？于是我想当然的说“这玩意适合新手入门！”

![](https://bbs.kanxue.com/upload/attach/202307/872365_N64F4GGN62H2B7C.jpg)

深入研究了之后，发现比想象中的要复杂，而且检测手段都挺别致，不常见，并且是一款叫做`nhnent appguard`的日韩商业保护，挺有意思，于是出一篇文章总结以下。

## 1. 加固分析

打开最核心的`libloader.so`,映入眼帘的是`ida`的一堆`warning`，提示`section header`不合法。无所谓，直接强行进去：

![](https://bbs.kanxue.com/upload/attach/202307/872365_3HKXJ5QMZSKYBMZ.png)

看到代码都是灰色的，一些无意义的二进制，应该是被加密了，看看`init array`：

![](https://bbs.kanxue.com/upload/attach/202307/872365_32K2ZS4GAUVFQTN.png)

第二个是红色的什么鬼？试图`jump`过去看看，但是`ida`显示：`Command "JumpAsk" failed`

说明这个地址目前还不在可见范围？？

`010editor`打开看看他的段信息：

![](https://bbs.kanxue.com/upload/attach/202307/872365_AJ3CF9J9ACB29F7.png)

代码段的文件长度是`0x25c848`,内存长度也是`0x25c848`,padding是`0x10000`,没什么问题。

数据段的文件偏移是`0x25ea50`,但是内存中的偏移却是`0x35ea50`,这中间活生生多出来了`0x100000`字节（100K）,根据代码段的`padding 0x10000`,数据段的内存偏移应该是`0x26ea50`才对，这显然不正常。

`init array` 的第二个函数地址是`0x29c1ec`,既没有在代码段里，也没有在数据段里，而在中间空出来的`padding`里？？！神奇！

所以第一个`init` 函数肯定要把这部分处理好，否则`linker`会报错，我们先看第一个`init`函数：

![](https://bbs.kanxue.com/upload/attach/202307/872365_7KAAEHWEZP26JKM.png)

通过`fopen`打开`/proc/self/maps`文件，然后通过`strtok`和`strtoul`找到链接过程中多映射的`100k`的内存空间，获取首地址

![](https://bbs.kanxue.com/upload/attach/202307/872365_FXJGVNXD26QHNWQ.png)

然后通过`mmap`将这些地址先设置为`rwx`权限，再通过`memcpy`把藏在数据段的代码复制过去，然后使用`mprotect`把这块内存恢复为`rx`权限：

![](https://bbs.kanxue.com/upload/attach/202307/872365_72HHJGBYNT3FGQR.png)

其中`memcpy`分别根据源地址，目标地址，长度，分5次`copy`

![](https://bbs.kanxue.com/upload/attach/202307/872365_53FHYE9HEG2QV4G.jpg)

复制完后，第二个`init`函数就已经在内存中准备好了，可以开始执行。

第二个`init`函数就是作解密用的：

![](https://bbs.kanxue.com/upload/attach/202307/872365_XDE642FAHE58WDB.png)

可以看到先解密了字符串，简单了异或了几个字符串就结束了，然后解密`section`。

字符串解密比较简单，我们看看如何解密`section`：

给`linker`的`callarray`函数下断点，在执行到第二个`init`函数时跟进去：

![](https://bbs.kanxue.com/upload/attach/202307/872365_CB9GHPYJ44RPXG2.png)

首先根据`X0`定位到`so`里面的加密数据，起始地址是：`0xab710`

![](https://bbs.kanxue.com/upload/attach/202307/872365_9KPKFPZ76YMSCGV.png)

刚好是`.text`的开始

然后将`.text`开始偏移`0x4`和`0xc`的字节拿来异或，得到的结果是加密数据的长度：

![](https://bbs.kanxue.com/upload/attach/202307/872365_XQPWJG39U5RR2QP.png)

然后根据`X1`(`解密后数据大小长度`) `malloc`一段内存，再将真正的加密数据内容（`.text`偏移`0x10`开始），按照计算的长度`x22`, `memcpy`过来

![](https://bbs.kanxue.com/upload/attach/202307/872365_GBY4TW6VSYSHGTX.png)

然后将`.text`偏移`0x8`和`0xc`的数据异或，计算出第一个`key`（`w11`），用于后面解密：

![](https://bbs.kanxue.com/upload/attach/202307/872365_AVQRBVXG7BWY3B5.png)

第二个`key` 在加密数据的末尾，通过`x22`找到加密数据的末尾，下一个字节即是第二个秘钥(`w25`)

![](https://bbs.kanxue.com/upload/attach/202307/872365_779UV4MCPPTG2YX.png)

然后开始解密工作，每字节异或一次，同时根据当前的偏移，奇数字节用`key1`，偶数字节用`key2`：

![](https://bbs.kanxue.com/upload/attach/202307/872365_F4ZR6Y5J5MPHK67.png)

解密完后看到数据的开头是`0x78`,`0x9c`，显然是`zlib`的压缩数据：

![](https://bbs.kanxue.com/upload/attach/202307/872365_9G76HKV7523C3BH.png)

接下来初始化`zlib`为`1.2.7`版本：

![](https://bbs.kanxue.com/upload/attach/202307/872365_BQNQFWQBMYAM2GW.png)

然后循环解压，再将解压后的代码复制回`.text`段：

![](https://bbs.kanxue.com/upload/attach/202307/872365_MN7SYK28B3K2UBV.png)

解压复制完代码也就解密完成了：

![](https://bbs.kanxue.com/upload/attach/202307/872365_2XAR2T3X62RZ5M9.png)

所以我们可以根据他的解密方式，自己写一套加密方式，把解压后的代码修改后压缩，加密回去，这样就可以对原代码进行修改了。加密代码如下：

```
import zlib
def encode(inName,outName):
with open(inName,"rb") as f:
data = f.read()
#压缩
o1 = list(zlib.compress(data))
#加密
index = 0
step = 0
key1 = 0xEAB54A3C
key0 = 0x5AB9C6D5
lenraw = len(o1)
out = []
while index < lenraw:
if lenraw - index >= 4:
raw = getInt32(o1,index)
if step == 0:
addInt32(out,0,raw ^ key0)
step = 1
else:
addInt32(out,0,raw ^ key1)
step = 0
index = index + 4
else:
while index<lenraw:
out.append(data[index])
index = index + 1
addInt32(out,0,key0)
with open(outName,"wb") as f:
f.write(bytes(out))
```

注意由于修改后的代码经过压缩长度可能会变化，所以需要针对性的修改`.text`开始的`0x4`,`0x8`和`0xc`处的字节，以符合解密算法。同时需要在加密数据的末尾按照小端法重新添上`key2`

至此，该加固的解密方式已经全部清楚，并且我们也可以定制化的修改代码，加密回去了。

## 2. 反调试

### 2.1 Tracerid反调试

加固研究完了，我们直接`F9`起飞，但是`ida`崩了。显然后反调试存在，但是`so`中没搜索到什么有用的字符串信息，应该是被加密了。

研究各种安全保护的第一步应该是：找到字符串解密函数。找到这个之后对方做的各种骚操作就一目了然了。

寻找字符串解密函数通常可以给`fopen`下断点，然后回溯。以反调试为例，通常先打开`/proc/self/status`文件，然后读`tracerid`。而打开这个文件会用到`fopen`。`so`里没有明文`/proc/self/status`，那肯定是解密后传给`fopen`的，即解密函数离`fopen`不远。

通过简单的调试观察发现，`sub_C4A50`是字符串解密函数：

![](https://bbs.kanxue.com/upload/attach/202307/872365_K7XNHQEWD3YXVZW.png)

可以看到也是简单的异或。

我们给`sub_C4A50`下断点，看看能发现什么：

发现解密出了很多常用的`libc`函数，感觉不妙

![](https://bbs.kanxue.com/upload/attach/202307/872365_BVMWKAU3RGKGU3Z.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_NFNDSPTR5Y29M76.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_MCP3V6B5K98DG8Z.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_ATG43JRMFH3ASDK.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_GKAMT5Y7PHFHTE3.png)

然后出现了熟悉的 `/proc/self/status`

![](https://bbs.kanxue.com/upload/attach/202307/872365_PGDB35U2VQMV8EV.png)

单步跟下去，发现又加密了`Tracerid`,这下图穷匕首现了：

![](https://bbs.kanxue.com/upload/attach/202307/872365_3DW4WNB3MWNWF34.png)

接下来就是常规的操作：

`fopen`打开`/proc/self/status`:

![](https://bbs.kanxue.com/upload/attach/202307/872365_QSRD22PWR9AE9B6.png)

`fgets`循环读取，然后`strstr`检查是否读到了`tracerid`:

![](https://bbs.kanxue.com/upload/attach/202307/872365_BFNWC5JYJQHAXC2.png)

如果读到了通过`atoi`获取`Tracerid`,如果不是`0`就进入退出流程：

![](https://bbs.kanxue.com/upload/attach/202307/872365_3MTSSUNWAP2YJF8.png)

退出流程比较复杂:

![](https://bbs.kanxue.com/upload/attach/202307/872365_VVRTN4XQPAQZ3FW.jpg)

最终会根据传入的参数不同，定制一种退出方式，有`exit`,有造`crash`,也有`svc exitgroup`,不再赘述。

我们只需要将`atoi`的返回结果改成`0`，就可以绕过了。

但是，要真的这么简单也就不会有这篇文章了。

### 2.2 断点指令检测反调试

一开始并不知道`atoi`后走的是什么退出流程，所以给`exit`下了断点：

![](https://bbs.kanxue.com/upload/attach/202307/872365_UG8AGBN37PMKXD6.png)

但是发现下断点之后，还没走到检查`Tracerid`一步，`exit`就被调用了，说明有别的检查。

通过反复的调试发现，只要给`exit`下断点，目标进程还没到检查`Tracerid`就会提前走退出流程。如果不给`exit`下断点，就会去检查`Tracerid`，然后根据`atoi`的结果决定是否走退出流程。

我们记得在调试中看到了解密出了`exit`,`_exit`等字符串，跟下去看看：

![](https://bbs.kanxue.com/upload/attach/202307/872365_DE8ATVVPSE354Y5.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_KE8UA48TRUH9E5V.png)

发现不光解密了`exit`,`_exit`，还解密了`__exit`,`kill`,`tgkill`

![](https://bbs.kanxue.com/upload/attach/202307/872365_UBN6R9KW5BWAM4T.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_3RZD7MTNHK69TA3.png)
![](https://bbs.kanxue.com/upload/attach/202307/872365_FPD6RCUYZ49K2QC.png)

而且每解密一个函数名，都通过`dlsym`获取函数的地址：

![](https://bbs.kanxue.com/upload/attach/202307/872365_VH3NYQ2CKA7WM3Q.png)

神奇。

接着跟下去，发现调用`dlclose`关闭了之前`dlopen`打开的`libc`

![](https://bbs.kanxue.com/upload/attach/202307/872365_EEV46ZR7ZSK3KET.png)

接着进入了另一个`BLR X8`。

跟到某一步发现把`exit`的地址当做参数，调用了一个函数：

![](https://bbs.kanxue.com/upload/attach/202307/872365_DW4W3VE6UDUQZMR.png)

发现将`exit`函数的前8个字节复制给了另一个地址：

![](https://bbs.kanxue.com/upload/attach/202307/872365_2VQ8H8V9XKKKEQP.png)

可以看到，复制的结果正好是`exit`的前八个字节：

![](https://bbs.kanxue.com/upload/attach/202307/872365_D4G2UXKDQYZWYYK.png)

`exit`函数：
