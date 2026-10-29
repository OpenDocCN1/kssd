
之后，只需要设置`scrollViewDelegate`代理即可。因为我们是在分类中添加了`scrollView`的代理方法，如果原来Apple已经在`WKWebView`中实现了`scrollView`的代理方法？毕竟Apple不会无缘无故将`WKWebView`设置为`scrollView`的代理，它肯定是有效果要实现，我将`WebKit`拖到`Hopper`中发现，确实如此：

![](upload/attach/201902/93_DQTRGAR3SCSXC4B.jpg)

在 demo 中测试，发现我们确实可以监听`scrollView`滚动。但是因为我不知道原来的`WKWebView`的监听滚动用来实现怎么样的效果，所以无法确定原来的监听是否依然有效。当分类和原类定义一个同一个方法时，运行时只有一个方法会被调用。从逆向的角度出发，我想直接 hook `WKWebView`的滚动监听方法，然后调用原来方法的同时，实现自己的监听通知。

```
+(void)load {
Method scrollViewWillBeginDragging = class_getInstanceMethod(self, @selector(scrollViewWillBeginDragging:));
Method hook_scrollViewWillBeginDragging = class_getInstanceMethod(self, @selector(hook_scrollViewWillBeginDragging:));
Method scrollViewDidEndScrollingAnimation = class_getInstanceMethod(self, @selector(scrollViewDidEndScrollingAnimation:));
Method hook_scrollViewDidEndScrollingAnimation = class_getInstanceMethod(self, @selector(hook_scrollViewDidEndScrollingAnimation:));
method_exchangeImplementations(scrollViewWillBeginDragging, hook_scrollViewWillBeginDragging);
method_exchangeImplementations(scrollViewDidEndScrollingAnimation, hook_scrollViewDidEndScrollingAnimation);
}
```

如上使用`method_exchangeImplementations`来实现hook，苹果会检查上架 app 的符号表，我们的实现并没有涉及到私有函数或属性，我想应该不会被拒吧？因为我也不是很熟悉 Apple 的审核规则，需要有大神可以补充解答。

第一次写这么长的文章，谢谢看完，逆向过程不是单独的线索一条线，也会有连蒙带猜，乐趣无穷。

## 说明

该文章为本人原创写作于去年，简书和掘金上的文章皆为本人发表，现将其发表于看雪。

少数派是国内最大的一个分析高品质数字消费指南的平台，致力于更好地运用数字产品或科学方法,帮助用户提升工作效率和生活品质。当推出iOS版本后，我立刻进行了下载和使用，作为一个开发者，首先必须是一个数字商品的消费者。最近期的一次更新中，发现了一个比较严重的bug，于是我利用逆向知识，对其进行了分析。

### 问题描述

- 最新版 v.1.0.4 在访问文章后返回会导致crash
- 逆向分析后发现在 iOS8 以上会有这个问题。（在 iOS8 以上系统中使用了 `WebKit` 框架）

## 找到崩溃原因

直观来说闪退最主要的原因有：找不到方法的实现，坏内存访问等。平时在使用Xcode开发自己的 app 时，可以直接在Xcode中快速找到这些 crash 的原因，相信这些定位崩溃的关键字大家已经很熟悉了。那么如何在没有源码的情况下定位这些 crash？

- 可以直接使用新版Mac的控制台来查看 iPhone 的日志输出
- 直接使用 `lldb`

我们当然是直接使用lldb啦，

基本操作，

```
// iphone
$ debugsever *:1234 -a pid

// mac
$ lldb
(lldb): process connect connect://ip:1234
```

![](https://bbs.kanxue.com/upload/attach/201902/93_C5B2UJJZKYB6MXQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201902/93_48GNANS29HGWCDZ.jpg)

lldb后，使用`c`命令运行程序，操作触发崩溃后可以看到如下输出：

![](https://bbs.kanxue.com/upload/attach/201902/93_DTB9JF23NTBDVVY.jpg)

可以看到我们非常熟悉的关键字：`reason=EXE_BAD_ACCESS`。由此可以判断崩溃是由于访问坏内存导致的。

使用命令`bt`打印调用堆栈

![](https://bbs.kanxue.com/upload/attach/201902/93_ERC7GJ2NQRMC4AA.jpg)

可以看到，程序是运行到一个`WebKit`的内部方法`[WKScrollViewDelegateForwarder forwardingTargetForSelector:]`之后访问换内存导致闪退的。天哪，我不会发现了一个Apple API的bug吧，继续往下分析。

- sspai应该是使用了WebKit框架的`WKWebView`来请求浏览的网页
- 通过这个类方法名可以看到这是一个关于`delegate`的类，应该是这个类对于我们设置的delegate做了一些事情导致的。
- 可能是sspai设置了`WKWebView`的delegate。当然现在只是猜想，之后需要通过`Hopper`来看一下这个sspai的Mach-O文件。

## 由行为猜想

根据崩溃的发生位置和时间

- 时间发生在进入文章后返回，这涉及到了两个控制器，可能是两个控制器之间的`delegate`
- 因为是在返回之后才会闪退，也可能是返回后控制器`dealloc`做的一些事情导致的

## 通过逆向查找bug点

逆向后知道类名如下：

- 文章列表控制器（首页）：`HomeTableViewController`
- 文章浏览控制器：`ArticleViewController`

1. 找到进入`ArticleViewController`的方法，查看应用是如何初始化的`ArticleViewController`
2. 浏览`ArticleViewController`类，查看可以方法

因为是从`HomeTableViewController`进入的`ArticleViewController`，所以我们需要在`HomeTableViewController`的`.h`文件中查找这个转跳入口，可以在`Hopper`中看到这个`turnToArticleViewController:cell:`非常可疑，根据我们的正向开发经验，通过这个方法名`turnTo vc`转跳并用`cell`参数传递了一个数据`model`。

在`Hopper`中查看方法如下：

```
 -[HomeTableViewController turnToArticleViewController:cell:]:

sub        sp, sp, #0x90 ; Objective C Implementation defined at 0x1006d3658 (instance method), DATA XREF=0x1006d3658

stp        x24, x23, [sp, #0x50]

stp        x22, x21, [sp, #0x60]

stp        x20, x19, [sp, #0x70]

stp        x29, x30, [sp, #0x80]

add        x29, sp, #0x80

mov        x19, x3

mov        x20, x0

mov        x0, x2

bl         imp___stubs__objc_retain

mov        x21, x0

mov        x0, x19

bl         imp___stubs__objc_retain

mov        x22, x0

adrp       x8, #0x1007e0000 ; @selector(setCurTableView:)

ldr        x0, [x8, #0xab8] ; objc_cls_ref_ArticleViewController,__objc_class_ArticleViewController_class

adrp       x8, #0x1007ca000

ldr        x1, [x8, #0x498] ; "alloc",@selector(alloc)

bl         imp___stubs__objc_msgSend

adrp       x8, #0x1007ca000

ldr        x1, [x8, #0x810] ; "initWithArticle:",@selector(initWithArticle:)

mov        x2, x21

bl         imp___stubs__objc_msgSend ; articleVC = [[ArticleViewController alloc]initWithArticle: articleModel ]

mov        x19, x0

mov        x0, x21

bl         imp___stubs__objc_release

adrp       x23, #0x10065c000

ldr        x23, [x23, #0x480] ; __NSConcreteStackBlock_10065c480,__NSConcreteStackBlock

str        x23, [sp, #0x28]

movz       w24, #0xc200

stp        w24, wzr, [sp, #0x30]

adr        x8, #0x100076ae4

nop

str        x8, [sp, #0x38]

adrp       x8, #0x100662000

add        x8, x8, #0x990 ; 0x100662990

str        x8, [sp, #0x40]

mov        x0, x22

bl         imp___stubs__objc_retain

mov        x21, x0

str        x21, [sp, #0x48]

adrp       x8, #0x1007cb000 ; @selector(cancelButtonClickAction:)

ldr        x1, [x8, #0x5c8] ; "setUpdateCommentCount:",@selector(setUpdateCommentCount:)

add        x2, sp, #0x28

mov        x0, x19

bl         imp___stubs__objc_msgSend ; [articleVC setUpdateCommentCount: block]

str        x23, sp

stp        w24, wzr, [sp, #0x8]

adr        x8, #0x100076b48

nop

str        x8, [sp, #0x10]

adrp       x8, #0x100662000

add        x8, x8, #0x9c0 ; 0x1006629c0

stp        x8, x21, [sp, #0x18]

adrp       x8, #0x1007cc000 ; @selector(showAlert)

ldr        x22, [x8, #0xa00] ; "setUpdateLikeCount:",@selector(setUpdateLikeCount:)

mov        x0, x21

bl         imp___stubs__objc_retain

mov        x21, x0

mov        x2, sp

mov        x0, x19

mov        x1, x22

bl         imp___stubs__objc_msgSend

adrp       x8, #0x1007cb000 ; @selector(cancelButtonClickAction:)

ldr        x1, [x8, #0x2e8] ; "setHidesBottomBarWhenPushed:",@selector(setHidesBottomBarWhenPushed:)

orr        w2, wzr, #0x1

mov        x0, x19

bl         imp___stubs__objc_msgSend

adrp       x8, #0x1007ca000

ldr        x1, [x8, #0x6f0] ; "navigationController",@selector(navigationController)

mov        x0, x20

bl         imp___stubs__objc_msgSend

mov        x29, x29

bl         imp___stubs__objc_retainAutoreleasedReturnValue

mov        x20, x0

adrp       x8, #0x1007ca000

ldr        x1, [x8, #0x818] ; "pushViewController:animated:",@selector(pushViewController:animated:)

orr        w3, wzr, #0x1

mov        x2, x19

bl         imp___stubs__objc_msgSend

mov        x0, x20

bl         imp___stubs__objc_release

ldr        x0, [sp, #0x20]

bl         imp___stubs__objc_release

ldr        x0, [sp, #0x48]

bl         imp___stubs__objc_release

mov        x0, x21

bl         imp___stubs__objc_release

mov        x0, x19

bl         imp___stubs__objc_release

ldp        x29, x30, [sp, #0x80]

ldp        x20, x19, [sp, #0x70]

ldp        x22, x21, [sp, #0x60]

ldp        x24, x23, [sp, #0x50]

add        sp, sp, #0x90

ret
```

> 我在其中加入了一些方法调用注释，可以看到方法的实现内容很简单，即使不懂汇编，通过这些`@selector`和正向经验也可以快速推断出来。
> 主要做了以下事情：
> 1. 初始化一个`ArticleViewController`类，并传递了一个`ArticleModel`的文章数据`model`
> 2. 设置了评论数和点赞数的block回调
> 3. 控制器转跳时隐藏`BottomBar`
> 4. 然后转跳

### 顺藤摸瓜查看`ArticleViewController`的初始化

在`Hopper`中查看到方法`initWithArticle:`如下，看到其中一段如下：
