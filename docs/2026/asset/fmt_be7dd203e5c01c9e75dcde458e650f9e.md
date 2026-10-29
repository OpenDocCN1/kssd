
# [原创]格式化字符串漏洞

本文是关于格式化字符串漏洞的介绍，论坛中有很多类似优秀的文章，我也对格式化字符串漏洞做一个总结，大家互相学习进步。

`fmtstr` github地址(包含代码和pdf文档):

[96cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6&6L8r3y4S2L8X3N6W2L8q4)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3k6E0N6s2y4@1M7R3`.`.](https://bbs.kanxue.com/elink@5beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6&6L8r3y4S2L8X3N6W2L8q4)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3k6E0N6s2y4@1M7R3`.`.)

该文章仅用于学习目的，不能用于商用，谢谢！由于本人能力有限，文章中可能会存在纰漏，欢迎大家指出，我及时斧正。

图片上传会变的模糊，这个我实在没办法，如果没办法看，请去github下载pdf。

## 测试平台

系统：CentOS release 6.10 (Final)、32 位

内核版本：`Linux 2.6.32-754.10.1.el6.i686`  i686 i386 GNU/Linux

`gcc` 版本： 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC)

`gdb` 版本：GNU gdb (GDB) Red Hat Enterprise Linux (7.2-92.el6)

`libc` 版本：`libc-2.12.so`

## 漏洞原理

程序中提供了参数可控（该格式化字符串参数来自外部输入）的`printf`族和`scanf`族函数或错误的参数类型或格式化字符串参数和传入参数个数不一致等情况，这导致我们可以控制程序行为或泄露一些信息。例如：

1、 可控的参数

![](https://bbs.kanxue.com/upload/attach/201911/665739_N5AEVQA7E7V864A.jpg)

2、 参数类型错误

![](https://bbs.kanxue.com/upload/attach/201911/665739_G2JZ8FQXRG5ZZDJ.jpg)

3、 格式化字符串参数和传入参数不符

![](https://bbs.kanxue.com/upload/attach/201911/665739_5BSSKKW27VH6ZN3.jpg)

第一种危害最大，我们完全可以控制程序行为，第二、三种不能控制程序行为，但可以对信息泄露提供一些帮助。

## 通用利用方式

格式化漏洞有两种利用方式：一种是实现任何地址读（可用于信息泄露），一种是实现任意地址写（可用于覆盖返回地址、`got`表、函数虚表等）。

## 格式化字符串介绍

我们以`printf`函数为例来说明几个主要的格式化字符串的参数：

`%d` - 十进制 - 输出十进制整数

`%s` - 字符串 - 从内存中读取字符串

`%x` - 十六进制 - 输出十六进制数

`%c` - 字符 - 输出字符

`%p` - 指针 - 指针地址

`%n` – 把前面打印的字符长度输出到指定地址

`%N$` - 第N个参数

![](https://bbs.kanxue.com/upload/attach/201911/665739_QA2G3QA7KVY9UW6.jpg)

程序执行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_RTBFTCCDTDW8YZF.jpg)

这里面前5个格式化字符串都用于输出，第6个用于输入，第七个用于指定参数的位置，第几个参数。

### 参数不一致

现在我们来看看参数个数不一致会发生什么情况。参数不一致指的是格式化字符串参数个数和实际输入参数不一致。

1、没有输入参数

![](https://bbs.kanxue.com/upload/attach/201911/665739_9DQKUXWMFGRYUST.jpg)

然我们看看程序运行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_3UR5EJYC7GHCB49.jpg)

从图中可以看到，程序把`sp+4`地址对应内存值作为第一个参数（靠近栈顶第二个）。

2、带有输入参数：

![](https://bbs.kanxue.com/upload/attach/201911/665739_V7P9D58AUKBCRW2.jpg)

然我们看看程序运行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_R4Y3DUU286A3NV3.jpg)

程序中包含7个格式化字符串参数，但实际传入参数只有两个。我们看到这次程序把`sp+12`地址处的内存值作为程序第三个格式化参数，以此类推（我们设定的格式化参数都是4字节输出）。在看一个例子：

![](https://bbs.kanxue.com/upload/attach/201911/665739_FHEFUG2579U8DQZ.jpg)

程序运行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_4AECK7QFAJEY5P2.jpg)

运行结果和第二个例子唯一的不同的是第三个格式化参数是`%s`，输出也变成了`sp+12`地址处内存中对应的字符串“|=x”（该地址不一定是字符串）。

通过这三个例子，我们了解到`printf`处理格式化参数的原理是如果输入的格式化参数个数多于实际输入参数，它将会把`sp + N*4`的内存值作为格式化参数对应的类型输出（`N`代表第N个参数，也可以理解为到达栈顶的步长）。在我测试的机器上第一个参数的地址是当前`sp+4`。以上面三个例子说明：

第一个例子：依据`sp + N*4`，此时`N`为1，所以对应第一个参数地址为`sp + 1*4`即为`sp + 4`。第二个例子：依据`sp + N*4`，此时`N`为3（有两个参数），所以对应格式化参数的参数地址为`sp + 3*4`，即`sp + 12`，第三个同理。

### 指定参数位置

通过`%N$`我们可以指定格式化参数对应的实际参数位置，如`%2$d`代表把第二个输入参数以十进制方式输出。

1、 栈上没有定义参数

![](https://bbs.kanxue.com/upload/attach/201911/665739_JWC6UZRQQ8WKWYQ.jpg)

运行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_SQXKYFKPA5VQ6K6.jpg)

从运行结果我们可以看到指定输出参数位置和不指定的运行结果是一致的，原理同上一节讲的。

2、栈上存在定义的参数

![](https://bbs.kanxue.com/upload/attach/201911/665739_7NFM66VFPVFKN9K.jpg)

运行结果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_W77DA7HPSYGKSS2.jpg)

从运行结果中可以看出，如果存在实际参数`%N$`中的`N`就代表第几个参数（它永远指向真是的参数，而不是`sp + N*4`），否则它指向`sp + N*4`（可以理解为距离栈顶的步长）。

## 漏洞利用

### 泄露栈内存

泄露内存的核心都是利用`%N$`，可以延伸出两种形式：泄露栈内存和泄露任意内存。泄露栈内存有以下两种方式：

1、`printf("%N$#x", {arg1...argn})` - 打印第`N`个参数的二级制值，`%N$`表示第`N`个参数, `N`可以大于`n`，`arg`代表实际输入参数（`N` >= 1）。

2、`printf("%N$#x")` - 打印当前栈顶距离`N`的内存值（`N` >= 1）。

`%#x`根据实际需要可以被替代为`%s`、`%p`。

前面已经举例了，这里就不在单独举例了。

### 泄露任意内存

`printf("[addr] %N$s")` - leak mem，`addr`是我们要读取内存的地址，`N`为格式化字符串漏洞的`stackpop`步长（视漏洞实际情况定），这里用到了`%s`，它会打印从`addr`到`NUL`字节之间所有内存。原理如下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_CSYT2CDQMF2QDKN.jpg)

我们用一个测试程序来演示一下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_D5C7RQFD2PS4RCF.jpg)

首先我们需要确定步长`N`，它可以通过使用`AAAA-%P-%P-%P……`这样的字符串进行测试：

![](https://bbs.kanxue.com/upload/attach/201911/665739_DERW6N9SURF3EAJ.jpg)

通过上图我们可以看到从`AAAA`到`0x41414141`相隔是7，所以这里的`N`为7。我们知道了步长为7，现在只要把`addr`替换成我们想要的地址就可以泄露任意内存了。我们用`pwn`实现一个`exp`用来泄露`scanf`函数地址，`exp`如下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_BBVWEWT9MHZQ7WH.jpg)

先调试看看`scanf`函数地址：

![](https://bbs.kanxue.com/upload/attach/201911/665739_28Z7WRQPSPBG3AG.jpg)

再运行`exp`看看打印的`scanf`函数地址是否正确：

![](https://bbs.kanxue.com/upload/attach/201911/665739_ENZ5NZGDCRBPQEH.jpg)

从图上可以看到打印的值是正确的。

### 覆盖任意内存

任意地址写核心是依赖格式化字符串`%n`，覆盖任意内存有以下两种方式：

1、 `printf("%Nd%n%[M-N]d%n",  a<dummy arg>, addr {a1，addr1…… an, addrn})`，此种情况用于`addr`地址已知（`addr{n}`已经被作为`printf`的实际输入参数），`N`为`addr`内存要被写入的值，`a`可以为任意值，`{}`中的参数可有可无，依实际情况而定，`M`为`addr1`要被设定的值（这里是`M-N`的原因是，前面已经打印了`N`个字符，`%n`是前面打印的字符数）。下面以两个测试程序说明：

1)    覆盖栈内存

![](https://bbs.kanxue.com/upload/attach/201911/665739_QQW8GA4C75V9CC6.jpg)

运行结果如下所示：

从图中我们可以轻易看到变化前`a=11`，`b=12`，`c=13`，改变后`a=100`，`b=200`，`c=300`。

![](https://bbs.kanxue.com/upload/attach/201911/665739_RHWAQK54VSP9THF.jpg)

2)    覆盖其他内存

在这个测试程序中，我们实现了堆内存和`puts` `got`表项的覆盖，前提是堆内存地址和`got`表项地址被当作参数传入了`printf`函数。

![](https://bbs.kanxue.com/upload/attach/201911/665739_2AJ5X2WXC29GWXP.jpg)

地址`0x80497dc`对应`puts`函数的`got`表项地址。堆内存覆盖后`a`的值由11变成120，`puts`函数的`got`表项覆盖后，调用`puts`函数会变成调用`system`函数。

![](https://bbs.kanxue.com/upload/attach/201911/665739_M5MR6WG8FRVTN87.jpg)

运行结果如下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_WM9EXNFX5BVJHJN.jpg)

2、 `printf("[addr]%Dd %N$n")`，此种情况可用于任何情况情况，原理同任意内存读，`D`代表`addr`要被覆盖的值，`N`代表步长（依实际漏洞情况而定）。以一个简单例子说明：

![](https://bbs.kanxue.com/upload/attach/201911/665739_UHHXH2D4BWBXUU3.jpg)

借助`pwn`编写`exp`如下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_U4VEV7YVEBCQK3A.jpg)

运行结果如下：

从下图可以看到`a`已经由10被覆盖为100了。

![](https://bbs.kanxue.com/upload/attach/201911/665739_56PN5QN7N6JRQVV.jpg)

覆盖其他地址同这个例子，这里就不在单独写程序演示了。

## 额外技巧

### 地址后移

`printf("[addr]%Dd %N$n")`这种格式输出`addr`就占4个字节，执行覆盖时最小值是4，可不可以小于4？

当然可以，见之前介绍的原理图，`addr`后移步长后移。`addr`被放到后面变成`printf("xx%N$n[**][addr]")`形式，`xx`、`**`可以为任意字符；`xx`代表要打印的小于4的字符串（赋值几就几个字符，如打印2就变成`11%N$n……`）；`**`代表要对齐的字符（我在测试过程中发现，需要按4字节对齐，否则程序是崩溃，如果打印2，格式如：`11%N$nxx……`补齐两个字符）；`N`代表步长，这里步长需要在原有基础+2（`xx**`代表一个参数，`%N$n`代表一个参数，`addr`后移了两个参数）。

我们同样以`demo13`为例来说明这种情况，依据刚刚描述借助`pwn`实现`exp`如下：

![](https://bbs.kanxue.com/upload/attach/201911/665739_AQYVU32ZH2PA29N.jpg)

执行效果：

![](https://bbs.kanxue.com/upload/attach/201911/665739_42P2WSXTSWHSXQX.jpg)

### 控制任意字节

如果把一个内存的值覆盖成`0xffffffff`通过使用上面描述的方法那岂不是要打印`0xffffffff`个字符，即耗时又不美观，怎么办？如果把某个地址4字节中的一个字节覆盖怎么办（上面提到的都是按指针长度覆盖）如果覆盖一片内存区域，超过4字节长度怎么办？

![](https://bbs.kanxue.com/upload/attach/201911/665739_THB8YD8HQJKJAZA.jpg)

通过这个表，我们就可以把`%N$n`转变成如下形式：

`%N$hh[X]`，操作一个字节，读操作`X=x|p|s`，写操作`X=n`

`%N$h[X]`，操作两个字节，读操作`X=x|p|s`，写操作`X=n`

`%N$l[X]`等价于`%N$n`，操作4个字节，读操作`X=x|p|s`，写操作`X=n`

`%N$ll[X]`，操作8个字节，读操作`X=x|p|s`，写操作`X=n`

这样写特别大的数时我们就可以按照单字节处理，不用打印那么多字符，同理你可以任意处理几个字节，少于4个多于四个无所谓。

举例我们想把地址为`0x0804A028`处覆盖为`0x12345678`，我们可以按照如下方式覆盖：

`0x0804A028` `\x78`

`0x0804A029` `\x56`

`0x0804A02a` `\x34`

`0x0804A02b` `\x12`

假设步长为6，则payload的样子如下：

```
p32(0x0804A028)+p32(0x0804A029)+p32(0x0804A02a)+p32(0x0804A02b)+pad1+'%6$hhn'+pad2+'%7$hhn'+pad3+'%8$hhn'+pad4+'%9$hhn'
```

这样这里的`pad`，后面的`pad`需要减去前面打印的字符。

我们可以直接借助`pwn`提供的格式化字符工具`pwnlib.fmtstr.fmtstr_payload(offset, writes, numbwritten=0, write_size='byte') → str`

`offset`表示偏移步长

`writes`表示目标地址`{addr: value, addr2: value2}`

`numbwritten`表示`printf`已经写入的字节数

`write_size`表示写入的字宽， 只能是`byte`，`short`，`int`(`hhn`、`hn` 或者`n`)

上面把地址为`0x0804A028`处覆盖为`0x12345678`可以用`pwn`表示如下：

```
fmtstr_payload(6, {0x804A028:0x12345678})
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-11-7 10:53

被angelToms编辑

，原因：
