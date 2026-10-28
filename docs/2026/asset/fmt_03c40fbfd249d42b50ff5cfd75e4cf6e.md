
myblog：<a href="elink@06fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2Q4y4h3k6A6N6r3q4U0K9o6q4Q4x3X3g2Y4K9i4c8W2k6g2)9J5k6h3W2G2i4K6u0r3">The\_Itach1</a>

然后附件给一下病毒和dump出的MRB部分吧。

# MEMZ病毒分析

第一次分析病毒，拿彩虹猫来练了练手，先自己分析过程，遇到不懂的在看其他师傅的分析，学到很多，本文章将一步步分析彩虹猫病毒的实现过程，并且还原部分代码。

## 病毒分析的准备工作

环境：我采用的是吾爱破解的xp系统，里面有很多现成的工具，ida，xdbg都有，只不过ida版本有点低，但是对于本病毒的分析也没太大影响，百度网盘链接：787K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4h3i4k6a6x3U0j5@1d9W2)9J5k6r3q4@1N6W2u0g2h3X3g2%4z5p5q4D9f1K6W2Y4 提取码：abcd。（如果有大佬知道怎么在xp系统中使用更高版本的ida，或者有更好的分析病毒的虚拟机，可以在评论区给点资源，谢谢啦）

分析步骤，一般的病毒分析步骤，如下

- 查壳，常见的有upx，ASpack，等等，可以用来实现简单免杀。
- 行为分析，是否修改注册表，联网，等等，可以在虚拟机中跑一遍看看效果（但是不要直接连网），或者用[vt](<https://bbs.kanxue.com/elink@4efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6i4K6u0r3>)直接跑跑看，有个大致的判断。
- 导入的dll以及API，使用StudyPE+去查看。
- 关键字符串，ida中shift+f12就可以直接查看。
- 放到ida静态分析，或者动调。

但是，对于彩虹猫病毒这个样本，里面的函数和逻辑都比较清晰，先在虚拟机里面跑几遍，看看行为，就直接开始分析了。

## 分析过程

分析过程大致分为3部分，启动部分，检测部分，彩虹部分，MBR部分。

### 启动部分

首先判断该进程的传参，类似于标准 C 运行时`argv`和`argc`值。从而走不同的流程。

![](https://bbs.kanxue.com/upload/attach/202109/926755_2YJNDSET3XC5PNT.jpg)

然后看下面干了什么。

![](https://bbs.kanxue.com/upload/attach/202109/926755_P48CUAQFMNDFSX2.jpg)

大概就是以不同参数开了6个`MEMZ.exe`进程。对于里面出现的函数可以到MSDN官方文档去查，举个例子，对于`SetPriorityClass()`，推荐在谷歌用[必应](<https://bbs.kanxue.com/elink@781K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8W2)9J5k6h3u0A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1j5>).上直接搜索该函数，然后进官方文档看函数参数意义，比如说，我要看`0x80`代表什么。

![](https://bbs.kanxue.com/upload/attach/202109/926755_76PUJ3EB9C6S4BA.jpg)

### 检测部分

这一部分就是用来判断用户是否主动关机，是否主动关闭`MEMZ.exe`，从而来用函数进行相应措施。

最外层。

![](https://bbs.kanxue.com/upload/attach/202109/926755_9PN7G4S3DGAQN3J.jpg)

然后来看看创建的线程函数`check`干了什么。

![](https://bbs.kanxue.com/upload/attach/202109/926755_TJW9PYJCZ246HYQ.jpg)

可以看出`check`函数用来判断是否主动关闭`MEMZ.exe`。

`shutdown()`函数中的`StartAddress`无法直接反编译，先用一位师傅[谁的狗哥](https://bbs.pediy.com/thread-262873.htm)处理一下函数，然后来看看`shutdown()`函数。

![](https://bbs.kanxue.com/upload/attach/202109/926755_8XWYWNCYZ3XPVVC.jpg)

然后看看创建的20个线程，就是用`SetWindowsHookExW()`实现了20个随机位置的messagebox弹窗，要理解这一部分，去查阅官方文档的函数。

![](https://bbs.kanxue.com/upload/attach/202109/926755_C4ZNG92QMU43SWD.jpg)

然后我们跳出来，回到最外层，来看看消息处理函数干了什么。

![](https://bbs.kanxue.com/upload/attach/202109/926755_XM2TSCX3RPDTDFB.jpg)

`WM_QUERYENDSESSION`，[7b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3A6W2M7%4y4A6j5$3q4A6N6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6x3I4x3o6M7&6x3o6x3](<https://bbs.kanxue.com/elink@59eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3A6W2M7%4y4A6j5$3q4A6N6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6x3I4x3o6M7&6x3o6x3>).所以这个就是用来防用户关机的。

到这里检测部分就分析完了。

### 彩虹部分

这部分是参数为`/main`的`MEZE.exe`进程执行的。主要表现在电脑桌面的花里胡哨。

![](https://bbs.kanxue.com/upload/attach/202109/926755_AWEUTB8R4WSF5T8.jpg)

这里`v9`类似于下面的结构

```
typedef struct {
    int time;
    int (*func)();
} functable;
functable a[10];
```

下面来分析这10个函数，里面很多api就不多讲了。

`func1()`，实现乱打开程序和网站。

![](https://bbs.kanxue.com/upload/attach/202109/926755_E836PRSVQTZ8SF6.jpg)

`func2()`，实现鼠标不受控制。

![](https://bbs.kanxue.com/upload/attach/202109/926755_CDZAWH5KUJ9FQQZ.jpg)

`func3()`，键盘消息

![](https://bbs.kanxue.com/upload/attach/202109/926755_TBKZJMH2CDWWG56.jpg)

`func4()`，发出声音

![](https://bbs.kanxue.com/upload/attach/202109/926755_JYZ97FDPXNDRAQ8.jpg)

`func5()`，桌面变色。

![](https://bbs.kanxue.com/upload/attach/202109/926755_UM4W6MJ69FR53DZ.jpg)

`func6()`，随机弹窗

![](https://bbs.kanxue.com/upload/attach/202109/926755_256NZSARTC9G4AD.jpg)

`func7()`，图标显示

![](https://bbs.kanxue.com/upload/attach/202109/926755_6XTT9WEF68PTFF4.jpg)

`func8()`，不知道效果，迭代子窗口。

![](https://bbs.kanxue.com/upload/attach/202109/926755_XKQS6D3UKQNB94Q.jpg)

`func9()`，桌面窗口复制显现。

![](https://bbs.kanxue.com/upload/attach/202109/926755_V2MJ6KE3MP8VP7J.jpg)

`fun10()`，随机复制某块区域。

![](https://bbs.kanxue.com/upload/attach/202109/926755_M5BY2ZGWJZNFF5T.jpg)

一些函数的执行次数数是有限的，一些是随机的，可以看线程函数的代码。

### MBR部分

这部分也是在以`/main`为参数的`MEMZ.exe`实现的。该部分才是真正十分有危害的部分。

MRB介绍：[aa3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Q4x3X3g2T1K9h3q4F1j5$3S2W2L8X3N6Q4x3X3g2F1k6i4c8Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9I4y4g2)9J5k6h3S2@1L8h3H3](<https://bbs.kanxue.com/elink@ceeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Q4x3X3g2T1K9h3q4F1j5$3S2W2L8X3N6Q4x3X3g2F1k6i4c8Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9I4y4g2)9J5k6h3S2@1L8h3H3>)，可以很形象的从文章最后一张图片看出MRB就是用来启动操作系统的。而`MEMZ.exe`的MBR部分就是修改了MRB。

![](https://bbs.kanxue.com/upload/attach/202109/926755_CQQADMP4QZ2AQVY.jpg)

开始我是直接动调，dump出了这段数据，然后拖到ida去分析，但是根本看不了。后面又看了狗哥的方法，就去试了试，还真可以，下面来写写详细过程。

环境搭建，首先我们需要在虚拟机里面下载Bochs，然后双击`bximage.exe`，然后选择1，选择fd，然后选大小，然后写名称。

![](https://bbs.kanxue.com/upload/attach/202109/926755_NFDB93RTMH458GG.jpg)

将得到的img和`bochsrc-sample.txt`都复制到新建的文件夹里面，路径桌面都行，但不要有中文。

打开`bochsrc-sample.txt`的该处修改`ata0-master`:

![](https://bbs.kanxue.com/upload/attach/202109/926755_QS554KTVXJ3R5CJ.jpg)

然后修改`.txt`为`.bxrc`，双击，然后选择continue，ok。

![](https://bbs.kanxue.com/upload/attach/202109/926755_FAZRFA8P2BJY7PB.jpg)

然后将`bochsrc-sample..bxrc`拖到ida分析，就可以动调了。动调还是一样的，只不过是16位汇编，分析起确实恼火，我也只能通过看反应，和看寄存器来分析个大概。

动调分析。

解密第二部分数据

![](https://bbs.kanxue.com/upload/attach/202109/926755_G9V8SBBUZN2688C.jpg)

打印字符串

![](https://bbs.kanxue.com/upload/attach/202109/926755_NXSF3G3TQF2SCE7.jpg)

播放视频

![](https://bbs.kanxue.com/upload/attach/202109/926755_Y7G9FRF97SQF8J7.jpg)

到这里该病毒就差不多分析完了。

## 部分代码实现

几乎除了MRB部分都写了，但是防关机功能并未成功，如果知道问题的大佬可以评论一波。
