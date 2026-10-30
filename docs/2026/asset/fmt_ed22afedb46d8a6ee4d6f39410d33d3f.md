
# [原创]cve-2012-0158漏洞分析与利用

## 一：漏洞概述：

`cve-2012-0158`是一个经典的office漏洞，漏洞原因时office在解析`activeX`控件时调用系统的`MSCOMCTL.OCX`库，此库中的函数在解析控件时存在栈溢出漏洞。

## 二：实验环境：

`win7(32位)`，`OD`，`WinDbg`,`IDAPro`，`office 2003`,`010editor`;

## 三：获取poc:

`Metasploit`

`d7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.

`726K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4#2M7X3W2@1P5h3k6G2j5%4g2K6i4K6u0W2j5$3!0E0`

## 四：漏洞复现：

首先启动`WORD`，然后打开`OD`，附加`WORD`；

![](https://bbs.kanxue.com/upload/attach/202003/839055_WA6YCAM29CFRN9J.png)

然后用`WORD`打开poc文件，此时会触发漏洞，观察`OD`；此时`eip`已被修改位`41414141`，观察堆栈情况，可以看出栈中的函数返回地址被修改位`41414141`；堆栈上方观察到一个来自`MSCOMCTL`模块的地址，反汇编窗口跟随查看；

![](https://bbs.kanxue.com/upload/attach/202003/839055_S6AMMQRAHGHEWUB.png)

根据反汇编窗口函数指令，找到函数起始位置下断点，然后`F8`动态调试，同时观察堆栈情况，找到触发栈溢出的指令或`Call`的调用；

![](https://bbs.kanxue.com/upload/attach/202003/839055_TSEHFUBM2NFKR4B.png)

经过`F8`动态调试分析，发现执行完`CALL MSCOMCTL.275C876D`函数后，堆栈中函数返回地址被修改，说明这个函数内部触发了栈溢出；

![](https://bbs.kanxue.com/upload/attach/202003/839055_4CMCG2VW8MJAPWK.png)

用`010Editor`打开poc文件定位到`414141`位置；并结合`OD`的堆栈情况和`IDA`对
`MSCOMCTL.275C876D`函数的分析，可以看出poc文件中的`8282`是缓冲区长度。此值设置过大导致下面调用`qmemcpy`函数发生溢出；

![](https://bbs.kanxue.com/upload/attach/202003/839055_HUHMDCVHX99C9KS.png)

![](https://bbs.kanxue.com/upload/attach/202003/839055_CEVNVJKS4XU9PHM.png)

## 五：漏洞利用

我们将上述`41414141`改为`jmp esp`指令的地址，同时调用溢出函数的函数平衡堆栈时有`ret 8`,所以`shellcode`的起始位置应如下图所示：

![](https://bbs.kanxue.com/upload/attach/202003/839055_BNY4UW9FDRGD337.png)

接下来用`WinDbg`通过指令"`!py mona.py jmp -r esp`"来搜索合适的跳板指令；如图所示，这里我们选择`0x729a0535`作为跳板指令；

![](https://bbs.kanxue.com/upload/attach/202003/839055_Q6UW6T6UEXVV6WU.png)

编写`shellcode`，弹出一个"`Hello IOTA!`"字符串,`vs`代码如下，`Release`编译后，在`OD`中提取`opcode`即可，这里不再赘述；
