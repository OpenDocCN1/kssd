
# [分享][原创]FTPServer的Fuzzing

ftpserver的fuzzing测试

## 1. 格式解析

### 1.1 FTP协议简介

FTP采用双连接来进行文件的传输：控制连接用于服务器和客户端之间命令的交互以及响应，而数据连接则用于文件数据的传输。服务端打开用于FTP控制连接的TCP端口21（标准），等待FTP客户端的连接。当客户端需要登陆FTP服务器时，主动打开本地TCP端口21，与服务端建立连接（控制连接）。该连接始终等待客户和服务器之间的通信，将命令从客户端传递到服务器，并传回服务器的应答。每当需要文件传输时，就创建一个数据连接。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_XEEXX7S8RXFGBSD.png)

命令和应答在客户和服务器的控制连接上以ASCII码形式传送。以下为常用的FTP命令：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_RJPSZZQT5XJDGSJ.png)

FTP应答是ASCII码的3位数字，并跟有报文选项：

- 125 数据连接已经打开;传输开始。
- 200 就绪命令。
- 214 帮助报文(面向用户)。
- 331 用户名就绪,要求输入口令。
- 425 不能打开数据连接。
- 452 错写文件。
- 500 语法错误(未认可的命令)。
- 501 语法错误(无效参数)。
- 502 未实现的MODE(方式命令)类型。

### 1.2 `socketsniff`抓包分析

启动`ftpserver.exe`后开启`socketsnifff`，并选定`ftpserver`进程，开启监听：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_NJSZ6X67TVWP5VN.png)

在主机上使用命令`ftp 192.168.18.144`连接`ftpserver`，在连接时，`ftpserver`会收到两个数据包，这个信息可用于构建连接数据包：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_Q4K33J9396T2UC2.png)

接下来发送命令，因为fuzzing操作时会反复发送命令，所以选取的命令不应对服务器或本机造成改动，`pwd`或`dir`属于较好命令。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_JSMMXJ6S6VV5BKK.png)

发现`dir`命令发送后，服务端打开了新的数据连接端口用于传输列表内容，所以选取`pwd`最为方便，且在数据包中直接使用`XPWD`作为命令行。

## 2 Pit编写

### 2.1 `DataModel`

需要编写“发送用户名——发送密码——发送执行命令”，而且`ftp`是交互式的，但对于服务端的回答，本测试中无需考虑。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_N8R2N57K9ESJVCN.png)

### 2.2 `StateModel`

FTP是一个连贯的交互过程，所以是`output`和`input`的交替。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_YVX4FGEEU5NB26S.png)

### 2.3 `Agent`

同样规定好监视器路径和命令行即可。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_DNXTHBZP2G4XFM8.png)

### 2.4 `Test`

最后是测试环节，这里有一个不同，为了使我们的`xml`具有通用性，这里的`Host`和`Port`并不是固定的，而是以参数形式提供。同时变异策略采用`Sequential`，及`peach`将按顺序对每个元素使用其所有可用的`mutators`进行变异。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_SYVPGVCZ5XS6BEN.png)

## 3 运行测试`ftpserver.exe`

使用命令`peach -DHOST=192.168.18.144 -DPORT=21 ./samples/ftp1.xml Test`

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_YDNFRKEBD5K4V7S.png)

可以在`log`文件夹下找到测试出的较多触发漏洞情形：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_WPPWPRNU7BC26VJ.png)

选取其中一个打开，查看记录，发现`eip`被覆盖了，确实有漏洞

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_PYQ2XJNJD776V4Q.png)

再去查看此时对应的输入

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_4KYGMGM4DG6JJGY.png)

可以猜测是输入过长导致覆盖了`EIP`，导致的漏洞点。

## 4 编写POC

利用`pwntools`编写脚本尝试攻击

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_2J5PE4Y7NSEBSH4.png)

在win7中打开`ftpserver.exe`，在kali中攻击

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_AA2YWN9CXQY48BW.png)

在win7中显示`ftpserver.exe`停止工作，确认确实触发了漏洞。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_JE2XCKAYK8V2WSG.png)

用`ollydbg`分析也可以发现确实有漏洞点

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_WQCJCDVHZRAUNVU.png)

## 5 漏洞点分析

进行调试：

单步调试时，在这一句之前`EIP`显示正常，而此句命令为`ADD ESP,100`，然后`RETN`

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_5SJTRZPH6S99P6Y.png)

但显然此时栈里全是A，`ESP+100`后到达AAAA区域

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_WVWFEZRRXYVUU2P.png)

此时再`retn`，会返回到AAAA中，报错

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_SMPR8R36EP3WP9B.png)

在这个大函数外面是`00402DE0`函数（因为到这里都没有报错）

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_C3E9AARPBVC8NGB.png)

用`ida`查看更清楚

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_EPKS66BVJ24Q2U4.png)

`V8`只有`0xfc`即252，测试一下这个填充长度是否正确

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_XRG7UBH8JFP93T3.png)

再进行一步即到`abcd`处，可知长度正确。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_KMSEAB5R4NHWN7E.png)

## 6 编写EXP

可以看到他会覆盖后续栈帧，如果可以让函数执行栈顶，则可以让他执行我们的函数，所以找到`JMP ESP`的位置，用这个命令位置作为跳板，在`ftpserver.exe`板块中没有此操作，可以使用操作ALT+E来搜索其余板块，最终在`ntdll`模块中找到适配指令，用作`jmpesp`，地址是`77A23B61`，如果使用`pwntool`的`p32`可以自动调整大小端，如果直接输入，要小端输入。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_HHGEJXV2NHGNPMB.png)

单步执行调试，发现确实可以让其跳转到`JMP`上：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_8Y557466EKMBFDB.png)

跳到了`JMP`上：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_934768A9H36CMZF.png)

只要让栈顶是我的指令即可，以防万一，使用一些`NOP`雪橇，下方红框为指令区，使用的是弹框指令来测试漏洞效果。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_T232VBE3RZFKXBG.png)

要找到这个弹框函数，可以右击“查找->所有模块中名称”，就可以找到弹框函数`MessageBoxA`的地址，用来作为`shellcode`的编写地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_MRMQC977CFXCUQC.png)

弹框内容作为参数压栈：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_8Z7WUWD32M9WBAX.png)

所以EXP如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_RAG89FBF8EFVXW6.png)

测试效果如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_YCP6CXW2YJ3R5ER.png)

如果要看细节弹框位置，可以单步调试如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/916710_PMDPCSRFFGPVV49.png)

参考网站：

<mark class="encrypted">814K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1%4y4U0x3%4z5e0c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1K6x3U0V1I4y4K6R3^5</mark>

<mark class="encrypted">e67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2B7K9h3q4Z5j5h3!0Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3@1k6A6k6q4)9K6c8o6p5$3y4U0b7K6z5o6R3%4y4o6l9&6y4e0V1$3y4K6M7#2z5e0S2Q4x3U0k6%4k6Y4u0Q4x3@1c8K6M7r3W2V1k6i4u0Q4x3U0k6X3L8%4u0Q4x3@1c8H3j5H3`.`.</mark>
