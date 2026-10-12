
# 使用 AndBug 调试 Android Java Bytecode

> 原文：[`www.pediy.com/kssd/pediy12/141995.html`](https://www.pediy.com/kssd/pediy12/141995.html)

我们在开发 android 应用时，一般会使用 eclipse 来调试 java 代码。在有源代码的情况下，使用 eclipse 调试时很方便的。

但是当我们做 android 逆向时，显然不会有源代码，这时候如果想要调试目标 java 代码呢？

一种做法是：我们可以用反编译工具将 apk 编译成源代码，然后再编译回去，在某些情况下这种做法是有效的，但是并不是所有的 apk 都能成功编译成源代码的，即使编译成了源代码，再编译回去时也可能会产生很多错误。

这里向大家介绍一个有用的工具：AndBug，它可以在没有源代码的情况下，调试 android 上的 java 程序，并支持断点、call stack 查看、查看 class、method 信息等。最重要的是，我们可以添加自己的脚本来加入我们所需要的功能，扩展这个工具。

AndBug 的主页是：https://github.com/swdunlop/AndBug

## 安装 AndBug

安装很简单，只要两步

1. 下载源代码

```
git clone https://github.com/swdunlop/AndBug.git
```

2. 编译

```
make
```

如果提示没有头文件，先安装 `python-dev`

## 使用 AndBug

假设有这样一个例子：

我们有一个 apk，名字叫做 `HttpRequest.apk`，它的界面包括一个按钮和一个文本框，当我们单击按钮的时候，程序会去访问一个固定的网页，并将页面的代码显示在文本框中，而这个网页的 URL 我们是不知道的，如下图所示，

![](img/62674f0fe868f10e9f0a77825313548d.jpg)

现在我们的任务是要找出这个 URL 是什么(当然，对于这个简单的例子，我们通过抓包或者静态反编译都能解决这个问题）。

我们将通过 AndBug 来完成这个工作。

1. 首先在模拟器中启动 apk，在命令行下输入 `adb shell ps`，找到相应的进程 `pid`，在这个例子中 `pid` 是 `243`

![](img/91f210461be3abb6fb7bd36c722acfae.jpg)

2. 进入 AndBug 目录，输入 `PYTHONPATH=lib ./andbug shell –p 243`，不出意外的话就能 attach 上目标进程了

![](img/c142255dc967973604fc93e4ddfa21fb.jpg)

3. 我们可以在命令行中输入各种命令，输入 `help` 可以看到所有可用的命令，常用的命令有:

`classes`: 查看加载的 class

`Break`: 下断点

`suspend`: 暂停进程

`resume`: 恢复进程运行

4. 我们应该在哪下断点呢？不要急，我们先看看这个 apk 用到了那些网络相关的类吧，输入:

```
classes java.net
```

![](img/e349f46bff927b89bd585092b64c82f8.jpg)

出现了一堆相关 class，我们看到里面有 `java.net.URL`，一般我们访问网络资源时，通常都会新建一个 URL 类，在这个类上下断点应该是合理的![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

5. 在 `java.net.URL` 类上下断点，输入

```
break java.net.URL
```

![](img/a76a57f271a706f9c0a4314fc09942d5.jpg)

这条命令会对 URL 类中的所有方法下断点，如果只想断其中某一个方法，可以在参数后面加上方法名

6. 下面就是触发断点了，回到 app 界面，单击 `get url` 按钮

![](img/6f18c4d6ee629614923cbabd57d1c3f1.jpg)

呵呵，断下来了! 看看堆栈，应该是在 URL 的 init 函数处断了下来

然后启动 navi server，输入 `navi`，可以看到 server 地址 `http://localhost:8080`

![](img/cb0b531ee95e9fe7bdb716b982140851.jpg)

7. 使用浏览器访问 `http://localhost:8080`，神奇的一幕发生了，我们可以观察到当前线程的状态以及类成员变量，当然也包括我们关心的 URL!

![](img/93c9812fda95005fe366513541a4a489.jpg)

以上只是一个简单的功能，我们完全可以通过添加自己的脚本，将 AndBug 打造成一个 android 逆向的利器! ![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")

文中用到的 apk 在附件中

`HttpRequest.apk`
