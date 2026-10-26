
# [原创]Windows按键精灵分析

## 一、概述

### 基本信息

![图片描述](upload/attach/202106/861115\_BE27WHDAFYKPPA3.png)

样本无壳。

![图片描述](upload/attach/202106/861115\_BXFFJVU2RD85PEV.png)

数字签名是吊销签名。

![图片描述](upload/attach/202106/861115\_MPDD7T4F566KS9Q.png)

### 外挂简介

引用徐胜的一段话"外挂是一种特定的辅助软件。外挂与木马程序的区别在于，外挂是用户主动使用的，而木马则是在用户的终端上偷偷运行的。根据外挂是否有模块进行游戏客户端，把外挂大致分成两大类，即内存挂和非内存挂。“很显然这个样本非内存挂。

### 流程图

![图片描述](upload/attach/202106/861115\_HTGUJJRDUCTJ5B6.png)

## 二、技术细节详细分析

### 动态分析

#### 文件行为

![图片描述](upload/attach/202106/861115\_VKVF4A2Y8KMYC7W.png)

释放的压缩包存在`C:\Users\<username>\AppData\Roaming\MyMacro`启用。

#### 网络行为

![图片描述](upload/attach/202106/861115\_YJVRJWFRPPCUARF.png)

![图片描述](upload/attach/202106/861115\_KV7V4QWSAFNHKZV.png)

通过端口8888和51019向117.27.139.134传输信息。

#### 注册表行为

![图片描述](upload/attach/202106/861115\_7AHGBUZ7H85JA7Y.png)

创建回调键值去启用注册的`CLSID`。

![图片描述](upload/attach/202106/861115\_R5HQJQ8QF8DDP7K.png)

增添键值，追踪原文件。

### 调试分析

初始化获取系统时间，跳入函数起始函数，检查文件和所在路径。

![图片描述](upload/attach/202106/861115\_YX3H8J5FBA9MQ65.png)

进入到`main`函数后，对异常，以及线程对临界区数据的读取做了进一步限制。

![图片描述](upload/attach/202106/861115\_TAT2EA95AT2WWTJ.png)

屏蔽系统错误窗口。

![图片描述](upload/attach/202106/861115\_9ZNU2PVPNNPKFB8.png)

生成相应的配置信息。

![图片描述](upload/attach/202106/861115\_QMB2M8MN8MVATTM.png)

根据参数的信息，可以直到这是监视对话框、消息框、菜单或滚动条中的输入事件而生成的消息。

![图片描述](upload/attach/202106/861115\_3658Q52MCYXA46W.png)

如上图，这个函数是`profuncaddreess`，调试他。

![图片描述](upload/attach/202106/861115\_YCQ2AERCHNNQCQB.png)

如果没有`hook`成功，将会直接跳转。设置成功，返回原函数。

![图片描述](upload/attach/202106/861115\_434QFK7UYBQURUT.png)

经过多跳，跳入设置设置注册表的函数。

![图片描述](upload/attach/202106/861115\_JEKSG7BXY47YKGC.png)

设置消息过滤，并向OLE注册`IMessageFilter`接口，根据每个线程只能注册一个消息过滤器。多线程单元中的线程不能有消息过滤器。

![图片描述](upload/attach/202106/861115\_QCXB85WY43FAE5T.png)

![图片描述](upload/attach/202106/861115\_4WU5WNYMH82N57R.png)

初始化com组件，初始化控件，初始化`gdip`函数库。

![图片描述](upload/attach/202106/861115\_Y4Q4NKWJBTW62DT.png)

创建`guid`，重启管理器。

![图片描述](upload/attach/202106/861115\_6STZHFSXFFGJ5Z2.png)

比对特殊字符。

![图片描述](upload/attach/202106/861115\_TFWYB6Z33VWQM5X.png)

通过下面的网址返回相应的信息。

![图片描述](upload/attach/202106/861115\_X93HJSB83FF9CH2.png)

![图片描述](upload/attach/202106/861115\_3XQTHXVYEJQAE6U.png)

设置事件跟踪。

![图片描述](upload/attach/202106/861115\_GH92WF3S3ZV3XKF.png)

监控鼠标的移动，点击，和时间间隔。

![图片描述](upload/attach/202106/861115\_S8M654SNRC8TY7W.png)

函数最后取消注册，取消互锁，取消钩子。

用`dbgview`看一下谁与客户端进行交互。

![图片描述](upload/attach/202106/861115\_CCK35AW8DXBEQQA.png)

QQbox客户端进行进程间管道通信。

`runner.exe`对`Visible`、`Enabled`、`Left`、`Top`、`Width`、`Height`、`ZOrder`、`Text`、`TextColor`、`BackColor`、`Hwnd`、`NormalColor`、`OverColor`、`FocusColor`、`Appearance`、`ColorScheme`、`Password`这些参数进行判断。

![图片描述](upload/attach/202106/861115\_NAT2TF483RNWKJU.png)

3908为原外挂程序访问网址。

![图片描述](upload/attach/202106/861115\_DUY2QA2YBJB4TR2.png)

![图片描述](upload/attach/202106/861115\_6UDZRGQZEHBRRW6.png)

![图片描述](upload/attach/202106/861115\_J6TGEKZ9TFPZEX4.png)

外挂可执行程序释放的`runner.exe`执行脚本。

![图片描述](upload/attach/202106/861115\_QXE57NMYQ2TRK7J.png)

`runner`加了所谓的强壳。

![图片描述](upload/attach/202106/861115\_XWU3JEXXU99RCER.png)

建议用1.4脚本去调试，也可以硬调。使用1.4脚本脱壳，可以参照我的od配置。

![图片描述](upload/attach/202106/861115\_KD3838U9KU2CQP5.png)

由于限制附件的大小，这是我OD的所有配置，运行起来没有异常。

![图片描述](upload/attach/202106/861115\_2DF64SYPK8VUF4C.png)

经过调试两种方法都可行。

![图片描述](upload/attach/202106/861115\_KKS6XVXUP2KKQKD.png)

通过`loadMenu`和`GetSubMenu`这种方式去执行脚本内容。
