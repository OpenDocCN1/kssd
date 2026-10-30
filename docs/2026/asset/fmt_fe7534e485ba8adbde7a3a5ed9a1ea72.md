
# Windows 2000 系统资源管理器的一个漏洞

## 0x00-前言

*这似乎又是一个0 day漏洞（？）*

这个漏洞与`pif`文件有关，是我在研究`pif`文件的时候发现的，关于`pif`文件的一些资料我整理到自己的博客里了：[cddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4k6Y4N6X3M7g2)9J5k6i4c8G2M7q4)9J5c8U0t1H3x3U0y4Q4x3V1j5H3y4#2)9J5c8U0l9^5i4K6u0r3M7$3!0E0k6g2)9J5k6r3k6S2j5%4c8Q4x3X3c8S2j5X3!0#2N6q4)9J5k6s2m8A6k6W2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@451K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4k6Y4N6X3M7g2)9J5k6i4c8G2M7q4)9J5c8U0t1H3x3U0y4Q4x3V1j5H3y4#2)9J5c8U0l9^5i4K6u0r3M7$3!0E0k6g2)9J5k6r3k6S2j5%4c8Q4x3X3c8S2j5X3!0#2N6q4)9J5k6s2m8A6k6W2)9J5c8R3`.`.`.)

后面会用到里面的一些内容。

**实验环境**

Microsoft Windows 2000 5.00.2195 Service Pack 4

**用到的软件工具**

- 010Editor
- ollydbg
- IDA pro

## 0x01-漏洞简述

首先，win 2000系统里存在一个文件`_default.pif`，它的结构如下：

![](https://bbs.kanxue.com/upload/attach/202308/968912_9B7UZD88HZ4EMVE.png)

我在这个文件的基础上添加了一个名为`WINDOWS VMM 4.0`的块，如下图：

![](https://bbs.kanxue.com/upload/attach/202308/968912_J7SFPH2TADUXEAD.png)

把这个文件放在U盘根目录下，插入win 2000系统后，浏览U盘时，资源管理器崩溃闪退。

![](https://bbs.kanxue.com/upload/attach/202308/968912_A2A89KHGAN8ZPHB.gif)

## 0x02-对崩溃的逆向跟踪，找到溢出点

我们可以确定的是，崩溃的出现是由于我们构造的`pif`文件里新的内容，而且应该出现在对文件内容的处理的过程中。

而根据`pif`文件的格式：

| 块名 | 长度 | 应用场景 |
| --- | --- | --- |
| MICROSOFT PIFEX | 0171h | The basic section, all OS |
| WINDOWS 286 3.0 | 0006h | Windows 3.X in standard mode |
| WINDOWS 386 3.0 | 0068h | Windows 3.X in enhanced mode, 95, 98, NT, 2000 |
| WINDOWS NT 3.1 | 008Eh | Windows NT, 2000 |
| WINDOWS NT 4.0 | 068Ch | Windows NT 4.0, 2000 |
| WINDOWS VMM 4.0 | 01ACh | Windows 95, 98, NT 4.0, 2000 |
| CONFIG SYS 4.0 | Variable | Windows 95, 98 |
| AUTOEXECBAT 4.0 | Variable | Windows 95, 98 |

可以看到每个数据块都有一个名字，猜测在对这些数据块处理的代码里可能包含这些名字的字符串常量，也许可以由此定位到数据块处理相关代码。

于是在`explorer.exe`和其调用的动态链接库中搜索这些字符串，最终发现只有系统`shell32.dll`中存在这些字符串：

![](https://bbs.kanxue.com/upload/attach/202308/968912_PHBZXNGG3RF36J3.png)

然后在`ollydbg`里追踪崩溃

由于windows系统只允许存在一个`explorer.exe`进程实例，我们用`ollydbg`打开新的`explorer.exe`进程进行动态调试时，系统会自动关闭该进程，从而阻止了调试的继续进行。所以需要通过`File -> Attach`菜单项将系统当前的`explorer.exe`进程附加`ollydbg`中的方式进行调试。

在调用这些字符串常量的函数里下大量断点，通过函数执行后崩溃是否发生判断溢出点的大致位置：

![调试过程中的部分断点](https://bbs.kanxue.com/upload/attach/202308/968912_EQJPWDHNWR4FQR8.png)

最后跟踪到这个函数：

![](https://bbs.kanxue.com/upload/attach/202308/968912_JEMGDHEG4S6379H.png)

也就是函数`sub_7909062B`

整个跟踪过程如下：

![](https://bbs.kanxue.com/upload/attach/202308/968912_9P98D8A38HWQ5AG.png)

动态分析函数`sub_7909062B`（上图中蓝色笔的部分），观察堆栈，发现在某次执行`lstrcpyA`后，堆栈里的返回地址和其它数据被覆盖

执行`lstrcpyA`前后堆栈里的变化：

![执行前](https://bbs.kanxue.com/upload/attach/202308/968912_N2MVD6QZEZVEG3H.png)

![执行后](https://bbs.kanxue.com/upload/attach/202308/968912_JX2SA4GJTDAHAAP.png)

## 0x03-不太成功的漏洞利用

函数的返回地址被覆盖，在Windows 2000系统上的漏洞利用应该是毫无难度的简单劳动，但是这回的不太一样。

在`sub_79057E1D`函数开始的地方将`this`指针保存在堆栈里：

![](https://bbs.kanxue.com/upload/attach/202308/968912_HSS97HGHTPFKZV2.png)

在堆栈被覆盖后的代码里又使用到`this`指针。

![](https://bbs.kanxue.com/upload/attach/202308/968912_SDXGU9H72PVZ5P6.png)

造成程序没有执行到函数返回指令的机会，所以按照常规方法通过返回地址得到程序执行权的方法在这里不适用。

但是`this`指针被覆盖也为我们提供了获得执行的机会，我们在溢出`this`指针的数据处设置为堆栈地址硬编码，在溢出数据中构造出索引虚函数地址的数据，可以**概率性**的实现计算器的启动。下面是我构造的`pif`文件：

![](https://bbs.kanxue.com/upload/attach/202308/968912_75NRZW9JZKFQHYA.png)

程序执行到`this`指针索引到的虚函数调用时，数据区里包含了我们的`shellcode`，启动的程序路径，和用来索引虚函数的链表结构。如下图：

![](https://bbs.kanxue.com/upload/attach/202308/968912_2U4EEXU96R69AVU.png)

其中包含了我写的`shellcode`：

![](https://bbs.kanxue.com/upload/attach/202308/968912_B9RAXVQ6WGRY3RT.png)

最终效果如下：

![可以看到试了几次才成功](https://bbs.kanxue.com/upload/attach/202308/968912_BY48XYE68HESDEM.gif)

**概率性**成功的原因是，`exploere.exe`进程每次启动的其栈地址都是动态变化的，溢出发生时真实的堆栈可能没在我们提前设定的地址处。我这里测试平均每5次会有一次成功启动计算器，最终的频次体验应该和电脑有关。

进一步研究，发现程序运行到调用虚函数地址时的寄存器`EAX`，`EBX`，`ESI`的值是溢出数据的某个位置的地址，所以理想的做法应该是寻找一个与`JMP EAX, JMP EBX, JMP ESI`类似的指令，假如这个指令的地址为`A`，设`this`的值为`x`，`[ ]`表示在进程地址空间里寻址后取值；我们可以将被覆盖的`this`指针设定为满足下面的公式的值：

`[[(x+4)]+0x1c] = A`

显然，找到满足上式的`x`是不容易的，我做了尝试，但失败了。不知道有没有更好的技巧来利用这种漏洞。

## 0x04-Windows XP及其他版本系统里存在该漏洞吗

进一步的，我对Windows XP的`shell32.dll`进行了逆向分析，发现同样的位置与windows 2000的代码极为相似，但是它用安全的`_StringCopyWorker`函数代替了`lstrcpy`，如下图：

![](https://bbs.kanxue.com/upload/attach/202308/968912_ZX68KU6G7Y8K3MY.png)

所以有理由相信，在后续的windows系统中微软已经发现了这个漏洞，并修补了它。
