
# [原创]A64dbg尝鲜——实战某加固so CrackMe

[toc]

## 前言

### Q&A

Q：为什么写这篇文章？

A：因为一方面，目前还没有使用A64dbg工具进行实战并附有详细分析和使用的文章，此文算是新品开箱的尝鲜文章，分享给各位看雪用户。另一方面，作为安卓逆向爱好者，拿到新的逆向工具还是比较激动的，在进行了不少尝试后，还是希望分享给大家一些有用的东西的。

Q：文章所涉及的功能，是A64dbg不同于其他逆向工具的全部了吗？

A：当然不是，涉及的只是我个人最感兴趣的功能——即A64dbg的UnicornVM调试模式下的ADCpp脚本系统。

Q：看完文章后去下载A64dbg可以复现这篇文章吗？

A：可能不行，我是拿到了A64dbg的测试授权，在授权情况下拥有完整功能后才进行的测试，未授权的话可能会有些限制，但我并没有具体地测试此文使用到的A64dbg功能被限制到了什么程度，如果感兴趣的话可自行尝试。

Q：文中写的内容好像用unidbg也可以做到，有什么区别吗？

A：unidbg很强大，但A64dbg的UnicornVM模式并不同于unidbg，怎么简洁明了地区别理解它们呢，那就是unidbg需要模拟linux系统和java环境，而A64dbg的UnicornVM模式并没有模拟这些，是在设备机中启动了`lidadbg-server`然后客户端调试器进行对接，也就是说是运行在真实的环境中的。

### 关于A64dbg的ADCpp脚本系统

最近上手了A64dbg的unicornVM模式的下调试，分析样本为[so文件动态加解密的CrackMe](https://bbs.pediy.com/thread-266546.htm#msg_header_h3_1)一文中进行了360加固的crackme。

先要说明的是，在了解A64dbg之后，首先吸引到我的是A64dbg的ADCpp脚本系统，先看一下介绍。

![](upload/attach/202104/802108_6Z54PEKFUHPXSE4.jpg)

> IDA有IDC这样的类C脚本系统，Frida嫁接了一层JavaScript脚本系统。虽然用起来还行，但始终不能让人满意。因为与Native打交道，就应该像C/C++那样直白，毕竟操作void *才是Native的精华。受限于C/C++是编译型的静态语言，想要实现像Python/JavaScript那样的便捷性着实不易。
>
> LLDB的表达式倒是可以使用C/C++语句，但是高度依赖类型系统，否则难以写出有效的C/C++表达式。即便如此，还是太弱了，不能写出复杂的完整C/C++函数。
>
> 但是，这一切即将成为过去时，我们将在A64Dbg v1.6专业版中引入UnicornVM的时候同时引入ADCpp这个C/C++脚本系统，它是把C/C++定义为了编译型动态语言，使用解释执行的方式运行代码。C/C++最终编译为机器相关的`arch.adc`字节码，由UnicornVM直接加载执行。不需要手动指定头文件、不需要手动指定链接库文件、不需要手动加载动态库，你只需要code，剩下的就是交给ADCpp了。
>
> 你还可以在A64Dbg端定义数据接收的Python3函数，然后在C/C++端发送给它，类似于Frida JavaScript与Python的交互，提供ADCpp接口可以主动调用内存模块中的函数，或者是只要有一个指针即可。
>
> ——[A64Dbg-ADCpp脚本系统简介](<elink@cb3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2y4Z5g2U0g2Q4x3X3c8h3y4q4S2p5K9i4W2T1y4r3#2X3N6W2k6C8K9f1q4t1b7b7`.`.)

就是说通过ADCpp脚本系统你可以实现使用C/C++代码来完成对内存中模块函数的注入hook，或者是进行主动调用，甚至只要有一个指针地址就可以实现，并且不管是系统空间还是用户空间。

基于对此功能的好奇，我进行了简单的测试，尝试使用A64dbg主动调用socrackme的解密函数，随后完成dump解密的so文件，以测试A64dbg在安卓逆向中内存操作的实战效果，这里将测试结果分享给大家。

## 360旧版加固的vm与反调试

在开始之前，先来说下[so文件动态加解密的CrackMe](https://bbs.pediy.com/thread-266546.htm#msg_header_h3_1)一文所做的分析，作者是看雪用户genliese 。genliese在文章开头说的很清楚，因为360加固的反调试所以so文件采用全静态分析，这是全文的基调。

关于反调试，笔者简单分析了下此版本的360加固，发现是典型的旧360壳，判断标志是`libjiagu.so`中存在`__fun_a_18`函数，而反调试就在里面的case当中，在论坛上几年前就已经有相关的分析文章了，最早的应该是[某数字公司VMP脱壳简记](https://bbs.pediy.com/thread-223528.htm)。

`__fun_a_18`函数的流程图看上去像是被加了ollvm控制流平坦化，但其实是一个基于堆栈的vm，找了很多资料，国内应该没有详细分析此vm的文章，但是论坛里有一篇翻译老外分析360加固的文章（[老外挑战360加固--实战分析（很详细）](https://bbs.pediy.com/thread-225561.htm)），里面有详细说这个vm，并且在github有相关还原代码。

![image-20210406141916800](upload/attach/202104/802108_M5UVWNWH8HFWZ76.png)

众所周知，旧版360加固的学习价值很高，是入门者练手分析很好的素材，这里我整理了些相关文章，列出来方便大家学习：

* [360加固保分析](https://bbs.pediy.com/thread-260049.htm)
* [某数字公司VMP脱壳简记](https://bbs.pediy.com/thread-223528.htm)
* [根据”so劫持”过360加固详细分析](https://bbs.pediy.com/thread-223796.htm)
* [老外挑战360加固--实战分析（很详细）](https://bbs.pediy.com/thread-225561.htm)
* [`vm_emulator.py`](<elink@947K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3N6Y4u0E0j5i4c8@1k6h3!0Q4x3V1k6p5e0f1&6b7i4K6u0r3j5X3I4G2j5W2)9J5c8U0g2U0j5K6g2S2y4r3c8W2j5e0q4W2z5r3g2U0k6e0t1@1k6U0p5#2k6r3b7@1x3K6M7H3x3e0x3@1k6e0p5&6j5e0b7#2j5$3u0W2k6e0u0Q4x3V1k6$3L8g2)9#2k6X3g2E0N6h3I4S2N6r3!0J5i4K6u0W2M7s2W2Q4x3U0y4x3x3K6f1H3>)
* [360加固之onCreate函数还原并重打包](https://bbs.pediy.com/thread-223223.htm)
* [某壳分析学习过程](https://bbs.pediy.com/thread-224708.htm)
* [某vmp壳原理分析笔记](https://bbs.pediy.com/thread-225798.htm)

现在回过头来说下反调试，旧版360加固主要有时间反调试、`rtld_db_dlactivity`反调试、`traceid`反调试和IDA端口反调试。

其中`rtld_db_dlactivity`反调试和`traceid`反调试是针对`ptrace`方式调试的反调试，在使用A64dbg的UnicornVM模式调试的测试过程中，笔者并没有发现触发反调试，说明A64dbg-UnicornVM模式的调试机制并不依赖于`ptrace`，也就是能直接无视掉此壳的反调试。

如果大家想尝试使用IDA进行调试分析，过反调试的话可以参考这篇文章[360加固保分析](https://bbs.pediy.com/thread-260049.htm)。

## so文件分析

文章中genliese使用FART定制ROM进行了脱壳，笔者进了测试，这个版本的360壳并没有进行指令抽取及vmp，使用dex整体dump即可脱壳。

dex的代码逻辑很简单，输入传进native层的`test`函数进行验证。

![image-20210406160919922](upload/attach/202104/802108_DGMWW9A3APQAGQX.png)

首先`libnative-lib.so`的`.init_array`中的函数进行了字符串解密，

![image-20210406162008183](upload/attach/202104/802108_G8WA6FDHYBA86X3.png)

我们跳过，直接看下`JNI_OnLoad`函数，可以看到字符串还没有被解密出来，以及java层的`test`方法应该是关联到了native层的`ooxx`函数。

![image-20210406162103022](upload/attach/202104/802108_6RQVHCM23EXUUWZ.png)

我们来看`ooxx`函数。

```text
.text:00008DC4 EXPORT ooxx
.text:00008DC4 ooxx ; CODE XREF: j_ooxx+8↑j
.text:00008DC4 ; DATA XREF: LOAD:000007C0↑o ...
.text:00008DC4
.text:00008DC4 var_2C = -0x2C
.text:00008DC4 var_28 = -0x28
.text:00008DC4 var_24 = -0x24
.text:00008DC4 var_18 = -0x18
.text:00008DC4 var_14 = -0x14
.text:00008DC4 var_10 = -0x10
.text:00008DC4
.text:00008DC4 ; __unwind {
.text:00008DC4 PUSH {R4-R7,LR}
.text:00008DC6 ADD R7, SP, #0xC
.text:00008DC8 SUB SP, SP, #0x24
.text:00008DCA MOV R3, R2
.text:00008DCC MOV R12, R1
.text:00008DCE MOV LR, R0
.text:00008DD0 STR R0, [SP,#0x30+var_10]
.text:00008DD2 STR R1, [SP,#0x30+var_14]
.text:00008DD4 STR R2, [SP,#0x30+var_18]
.text:00008DD6 STR R3, [SP,#0x30+var_24]
.text:00008DD8 STR.W R12, [SP,#0x30+var_28]
.text:00008DDC STR.W LR, [SP,#0x30+var_2C]
.text:00008DE0 BL sub_8930
.text:00008DE4 MOV R0, R0
.text:00008DE6 MOV R0, R0
.text:00008DE8 MOV R0, R0
.text:00008DEA MOV R0, R0
.text:00008DEC MOV R0, R0
.text:00008DEE MOV R0, R0
.text:00008DF0 MOV R0, R0
.text:00008DF2 MOV R0, R0
.text:00008DF4 MOV R0, R0
.text:00008DF6 MOV R0, R0
.text:00008DF8 MOV R0, R0
.text:00008DFA MOV R0, R0
.text:00008DFC MOV R0, R0
.text:00008DFE MOV R0, R0
.text:00008E00 BX R0
.text:00008E00 ; End of function ooxx
.text:00008E00
.text:00008E00 ; ---------------------------------------------------------------------------
.text:00008E02 DCW 0x4502
.text:00008E04 DCD 0x41064304, 0x4D0A4F08, 0x490E4B0C, 0x55125710, 0x51165314
.text:00008E04 DCD 0x5D1A5F18, 0x591E5B1C, 0x65226720, 0x61266324, 0x6D2A6F28
.text:00008E2C DCD 0x692E6B2C
.text:00008E30 DCD 0x75327730
.text:00008E34 DCD 0x71367334
.text:00008E38 DCD 0x7D3A7F38, 0x793E7B3C
.text:00008E40 DCD 0x5420740
.text:00008E44 DCD 0x1460344
.text:00008E48 DCD 0xD4A0F48
```

可以看到初始化后跳到了`sub_8930`，而`sub_8930`函数中，我们看到了十分明显的内存指令解密标志`mprotect`以及`cacheflush`函数。

至于如何具体进行的内存指令解密操作，[so文件动态加解密的CrackMe](https://bbs.pediy.com/thread-266546.htm#msg_header_h3_1)一文中十分详细的讲解了，这里我就不多说了，此crackme是一个很好的指令解密学习素材，大家感兴趣的话可以动手分析下。可以猜到，`sub_8930`进行内存指令解密后，便是开始执行真正的代码了，就是对输入进行校验。

![image-20210406162527526](upload/attach/202104/802108_W7D8H6SGP3VHQBT.png)

到这一步可以知道，想要拿到`flag`就需要知道被加密的代码是什么。genliese采取的方式是使用fridad动态dump内存中的加密指令的解密密钥，然后再使用密钥静态解密文件中的指令加密部分，然后IDA即可反编译出解密后的`ooxx`函数，以及不得不称赞的是genliese的代码写的很漂亮，让人看的很舒服。

### A64dbg-ADCpp测试

好了，下面就开始进入此文的关键的部分，对A64dbg的ADCpp脚本系统的内存操作功能进行测试，这里测试的主要是ADCpp脚本系统的主动调用和内存dump特性，我们的测试思路如下所述。

尽管我们可以在指令解密完后设下断点，然后输入`flag`点击触发断点随后dump内存。但是我们并不这样做，而是选择让程序保持运行，使用ADCpp脚本系统主动调用指令解密函数`ooxx`，然后让断点停在指令解密完后，随后进行dump so文件。

也就是说程序一方面正在运行等待我们的输入，光标还在闪动，而另一方面，我们已经悄无声息地开辟了一条新的战线，执行了内存里的函数，并停在了我们设下的断点，然后进行了dump操作。

至于为什么在解密完的时刻设下断点，让调试器在这里停在这里，因为`ooxx`函数解密出来的指令在对输入进行校验后，还会将`ooxx`再加密回去。

#### 启动lidadbg-server

我们在设备机里面启动`lidadbg-server`后，然后打开A64dbg，Options->Preferences里客户端进行设置。

填好Remote Android栏下的ip和port，接着填好`adb`的路径，以及最重要的是Default Platform选择Remote UnicornVM Android，这样才能进行UnicornVM模式下的调试。

![image-20210406211011362](upload/attach/202104/802108_6ZNSGFVT9VDGHKR.png)

然后运行Apk，点击File->attach，然后搜索包名attach即可。

![image-20210406211409609](upload/attach/202104/802108_F7ZH5DDH9QSTEZ2.png)

#### 设置断点

第一次加载会比较慢，等待加载好后，点击`Symbols`窗口，左下角搜索模块名称`libnative-lib.so`，右下角搜索函数`ooxx`，双击。

![image-20210406165127510](upload/attach/202104/802108_2EKGQ49ARP86STT.png)

进入汇编窗口，可以看到跳到了`sub_8930`函数，双击。

![image-20210406165559637](upload/attach/202104/802108_7N6UBMP5RE2Y6F7.png)

然后找到指令后解密完后，调用`cacheflush`的地方，设下断点。

![image-20210406170036462](upload/attach/202104/802108_3PNA3XZBHVFBJHE.png)

#### ADCpp脚本系统主动调用解密函数ooxx

现在断点已经设置好了，按照我们的计划，要开始进行主动调用了。

ADCpp脚本运行有两种方式，一种是直接载入脚本，另一种则是通过最下方的命令行窗口，这里我们都尝试一下。

然后主动调用函数也有两种方式，一种适用于有符号的函数，我们通过`extern "C" void ooxx();`就可以引用A64dbg解析好的有符号的函数，可以直接调用；另一种则是知道函数在内存中的地址后，我们将绝对地址转化为函数指针然后进行调用，这时候要注意汇编是否在thumb模式下。

##### 方式一

方式一，从解析出来的模块符号表中找到`ooxx`函数，然后进行调用。
