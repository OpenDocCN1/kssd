
# [原创]cve-2018-8453分析及利用EXP编写

在2018年11月份的时候，请论坛里各位请教了这个漏洞

https://bbs.pediy.com/thread-247444.htm

现在这个漏洞已经彻底搞完啦！来这里跟大家说一下，算是一个收尾。我把分析发我们部门公众号了：

`f05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3!0Y4d9@1y4G2i4K6u0V1d9Y4l9^5N6X3x3%4L8%4c8j5P5i4f1$3k6W2c8A6k6#2)9J5y4X3&6T1M7%4m8Q4x3@1t1`. 主要是该漏洞的成因分析

`da1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3c8U0j5W2g2W2k6h3N6y4x3p5u0I4c8i4u0@1c8s2g2X3e0#2S2X3L8#2p5`. 主要是该漏洞的利用EXP编写！

再次感谢~

。。。依照要求我整体复制过来。。。

---

## [上篇]从补丁diff到EXP--CVE-2018-8453漏洞分析与利用

下载该漏洞的单独补丁用`bindiff`与历史补丁进行查看，可发现主要变化如下（新旧变化主要用上一个补丁日的`win32k.sys`来比较，下同）：

![](https://bbs.kanxue.com/upload/attach/201901/370883_4R6K6MY2QUH7G6C.jpg)

在底部，看到颜色差异比较大的就是一个叫`NtUserSetWindowFNID`的函数，比较一下：

![](https://bbs.kanxue.com/upload/attach/201901/370883_9WPHR9QPKT4R2AC.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/370883_C7G946FBCRHRCP9.jpg)

可见判断流程中，多了一个`IsWindowBeingDestroyed`函数调用：

![](https://bbs.kanxue.com/upload/attach/201901/370883_G2ZMXEHNYF4A8JF.jpg)

也就是说，主要在设置改变窗口的一个成员时，多了一个检查。那么就意味着，本漏洞的原因是设置成员时，没有判断某成员造成，从名字上看，这个成员为`FNID`。

那么，问题在于，`FNID`这个成员没有检查又会造成什么影响呢？我们查看一下这个成员的作用，在win2000的部分源代码中，我们可以搜索`FNID`来探明`FNID`是什么意义。

![](https://bbs.kanxue.com/upload/attach/201901/370883_5HTZ8YCAVVDEMTW.jpg)

这个`FNID`成员是用来标识本窗口是一个什么样的窗口，比如是一个按钮还是一个编辑框，这一点从文章里也可以印证。而从补丁修改后新加的函数名`IsWindowBeingDestroyed`来看，这里是要判断本窗口是否已经准备删除了。从文章中说的查看ReactOS代码，可知道准备删除标记就是添加上`FNID_FREE`（`0x8000`）的标记。关键在于，不检查`FREE`之后的窗口是如何触发漏洞呢？

通过卡巴的文章，我们整理出来大致利用思路：代码先 HOOK `KernelCallbackTable` -> 产生一个主窗口 -> 在`USER32!__fnINLPCREATESTRUCT`回调中去查找并取消掉`sysShadow`窗口 -> 以主窗口作为父窗口产生一个滚动条窗口`SrollBar` -> 发送`WM_LBUTTONDOWN`消息 -> 系统处理消息时会发生`USER32!__fnDWORD`回调，在`USER32!__fnDWORD`回调中销毁主窗口 -> 这将导致主窗口销毁从而产生`USER32!__fnNCDESTROY`回调 -> `USER32!__fnNCDESTROY`回调中调用`NtUserSetWindowFNID`更改掉`FNID` -> 至此文章中开始语焉不详，文章中说重用了`sysShadow`，但我们根本理解不了如何发生得重用。所以需要我们自己来动手实现。

首先我们来实现漏洞函数得调用，仔细观察：

![](https://bbs.kanxue.com/upload/attach/201901/370883_7CCCTRR29829PEJ.jpg)

可以看到，要想成功更改`FNID`，需要满足几个条件，我们不可能只设置为`0x4000`（这个只是打个标记，不产生实际作用）。至于新`FNID`得值，我们可以按照文章中说的直接设置为`0x2a1`即可。而对于后面的条件，要求我们要设置的窗口原来不能有`FNID`（除`0x4000`和`0x8000`外，但这两个标记我们打了没用）。这里经过多次测试，发现三种情况会时`FNID`为空：一种是在任意类型窗口刚建立时，这时系统在用户态主动调用`NtUserSetWindowFNID`来设置`FNID`（`user32.dll`中自动实现），而此时，如果没有设置完`FNID`，则窗口还没有设置消息处理函数，也就没有处理消息的能力。而文章中提到了`WM_LBUTTONDOWN`消息，则可以肯定是在`Scrollbar`窗口完全创建之后。故此种情况不行。二种是用户注册的窗口类所产生的窗口，此窗口一直到销毁，都没有设置`FNID`。第三种就是文章中所说的`sysShadow`窗口，此窗口的作用只是产生阴影效果，但是确实`FNID`为空。也正是由于这个特性，本人被文章误导很长时间。后来请教leeqwind才知道，根本不是重用的`sysShadow`，而是`SBTrack`结构。另外也可以看文章截图：

![](https://bbs.kanxue.com/upload/attach/201901/370883_YWJEU58QVMCUTZH.jpg)

由于本人注意力全放在了文章触发中，还未关注利用，没注意后面的内容，其实这里已经泄露了真相（深刻检讨反思！）。从截图的红框中可看到，标记是`Usst`，分配者又是`win32k!xxxSBTrackInit`。所以很明显可知要被重用的是`SBTrack`。

文章中说明了需要在`FNID`设置为`0x8000`之后，再调用漏洞函数更改`FNID`。我们知道，一个窗口销毁的用户态接受到的最后消息是`WM_NCDESTORY`，在`win32k`中，这是在`xxxFreeWindow`函数中发送给窗口的：

![](https://bbs.kanxue.com/upload/attach/201901/370883_USM6SHZ89C5VW4Q.jpg)

可以看到在106行发送了`0x82`（`WM_NCDESTORY`）号消息，所以我们需要在106行之后想办法回到用户态。但同时有另外一个问题，就是注意第134行，这行把`FNID`打上`0x4000`的标记，而文章中完全是`0x8000`直接变成了`0x82a1`，没有`0x4000`的标记，所以我们如果再`WM_NCDESTORY`消息中去更改`FNID`，那么确实可以马上更改掉`FNID`，但是这时窗口还并没有打上`0x8000`的标记（到136行中才被标记），这与文章明显不符。所以文中所说的在`USER32!__fnNCDESTROY`中去调用`NtUserSetWindowFNID`更改`FNID`的做法为故意错误。

![](https://bbs.kanxue.com/upload/attach/201901/370883_VABRPYK9BEE6SPP.jpg)

经过本人用`pykd`动态测试发现，窗口在426行的调用后，窗口句柄将不存在，`NtUserSetWindowFNID`函数的`ValidateHwnd`函数将返回`0`从而直接跳过`FNID`设置。也就是我们想要实现文章中直接将`0x8000`设置成`0x82a1`的效果，需要我们在134行到426行之间，找到一个回到用户态的调用。

这里插播一点题外话。本人一开始是在WINDOWS7的`win32k.sys`做分析，结果搜索很长时间未能成功找到，直到某天twitter上有人提到该漏洞在win8.1之后可利用，在看了win10的`win32kfull.sys`后恍然大悟，教训深刻！

回到正题，之后我们可以看到这里：

![](https://bbs.kanxue.com/upload/attach/201901/370883_7DSFTFZNEUEHH45.jpg)

上面这张截图为win10下的`win32kfull.sys`的IDA分析结果。我们可以看到，在256行改为了`0x8000`后，在266行有一个`xxxClientFreeWindowClassExtraBytes`函数调用：

![](https://bbs.kanxue.com/upload/attach/201901/370883_TWCADHGERXWPD33.jpg)

该函数中很直接的调用了`KeUserModeCallback`！毫无条件的直接回到了用户态。所以我们只要符合进入到`xxxClientFreeWindowClassExtraBytes`函数的条件即可。仔细查看代码，发现258行的判断，主要是判断窗口是否具有扩展字节（正如`xxxClientFreeWindowClassExtraBytes`函数名字所暗示的那样），有的话则调用`xxxClientFreeWindowClassExtraBytes`函数释放掉，由于扩展字节是分配在用户空间中的，所以该函数返回到用户态让用户态代码去释放掉（至少要通知）。所以只要我们在注册窗口类的时候，`cbWndExtra`成员不为`0`即可。在窗口销毁时，就会在设置了`0x8000`之后，又回到了用户态。当窗口以`0x8000`回到用户态后，我们更改`FNID`为`0x82a1`，返回到内核态后，`xxxFreeWindow`继续往后执行。

回到`xxxFreeWindow`函数：

![](https://bbs.kanxue.com/upload/attach/201901/370883_EKTYFC863E5U7MK.jpg)

其中这里，可以看到代码判断了`FNID`的值，从而决定要不要调用`USER32!__fnDWORD`。我们知道，当一个窗口被多个其他窗口、结构引用时，即时这个窗口已经被用户调用`DestoryWindow`销毁掉了，窗口对象也要在内存中继续存在，以等待所有引用它的地方不再引用它才真正释放本对象内存。那么，如果我们在销毁了一个窗口后，它的最后一个引用也释放的时候，调用`xxxFreeWindow`时，我们就可以用`FNID`来控制流程是否要回到用户态的`USER32!__fnDWORD`调用。所以攻击链也就此完整。

结合上面提到的，文章中提到了使用`xxxSBTrackInit`。该函数主要用来实现滚动条按钮的跟随鼠标滚动，当用户在一个滚动条上按下左键，表示用户想要拖动滚动条，此时需要开始处理鼠标的移动，让滚动条也跟着相应动起来，在系统中，产生`SBTrack`结构来标记用户鼠标的当前位置，最后当用户放开鼠标左键时，表示用户已经拖动完成，需要释放相应`SBTrack`结构。

在windows 2000的源代码中，`xxxSBTrackInit`部分代码如下：

![](https://bbs.kanxue.com/upload/attach/201901/370883_BCPZ78QJBZJXNBH.jpg)

大致流程就是在调用`UserAllocPoolWithQuota`申请了内存后，初始化`SBtrack`，会将滚动条窗口以及通知窗口的指针放在本结构中，然后在2425行将当前窗口设置为捕获窗口。之后就调用`xxxSBTrackLoop`开始循环来处理用户的鼠标消息：

![](https://bbs.kanxue.com/upload/attach/201901/370883_V96HBJUJD47Q25S.jpg)

可以看到，`xxxSBTrackLoop`循环获取消息、判断消息、分发消息。当用户放开鼠标时，应当停止跟踪处理消息，退出`xxxSBTrackLoop`后回到`xxxSBTrackInit`之后，释放`SBTrack`占用的内存：

![](https://bbs.kanxue.com/upload/attach/201901/370883_ZJ9Z49EYPF2NZUX.jpg)

而往上两行，可以看到在释放`SBTrack`之前，会解除一次`spwndSBNotify`窗口的引用。结合上面的分析，我们可以让这次解除引用时，回到用户态。如果在用户态释放掉`SBTrack`，则流程再次回到内核时，紧接着后面的`UserFreePool`即造成重复释放的问题。

那么我们在用户态如何释放`SBTrack`呢？分析发现，导致释放`SBTrack`一种是用户正常放开了鼠标左键，还有一种就是`xxxEndScroll`函数：

```c
void xxxEndScroll(
    PWND pwnd,
    BOOL fCancel)
{
    UINT oldcmd;
    PSBTRACK pSBTrack;

    CheckLock(pwnd);
    UserAssert(!IsWinEventNotifyDeferred());

    pSBTrack = PWNDTOPSBTRACK(pwnd);
    if (pSBTrack && PtiCurrent()->pq->spwndCapture == pwnd && pSBTrack->xxxpfnSB != NULL) {
        ……..省略部分代码…….

        pSBTrack->xxxpfnSB = NULL;

        /*
         * Unlock structure members so they are no longer holding down windows.
         */
        Unlock(&pSBTrack->spwndSB);
        Unlock(&pSBTrack->spwndSBNotify);
        Unlock(&pSBTrack->spwndTrack);
        UserFreePool(pSBTrack);
        PWNDTOPSBTRACK(pwnd) = NULL;
    }
}
```

`xxxEndScroll`函数判断了主要根据窗口的线程信息中存放的`SBTrack`和`pq->sqpwndCapture`。

![](https://bbs.kanxue.com/upload/attach/201901/370883_BRAA8WPCK9UR45W.jpg)

而我们的程序是单线程，由于每个线程信息是属于线程的，所以线程创建的所有窗口也都指向同一线程信息结构。所以，即使`SBTrack`所属于的`Scrollbar`窗口已经释放了，只要还是同一线程创建的新窗口，`pSBTrack`也还是原来的。而`qp->spwndCapture==pwnd`如何绕过呢？我们如果创建新的窗口，给这个新窗口发送的消息和操作，`pwnd`则为新窗口，这显然不会等于在`xxxSBTrackInit`中设置的捕获窗口----旧窗口。

通过测试发现，这个Capture窗口的设置，只要简单的在用户态调用`SetCapture` API即可直接设置。所以我们只要直接调用API即可让`xxxEndScroll`中的判断完全通过。

在搜索之后，发现可以通过如下路径调用`xxxEndScroll`函数：

![](https://bbs.kanxue.com/upload/attach/201901/370883_SDGDAP6N2JBNBU9.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/370883_8DHK6Q6SXB9SWSR.jpg)

向一个窗口发送`WM_CANCELMODE`->`xxxDefWindowProc`判断消息->调用`xxxDWP_DoCancelMode`->`xxxDWP_DoCancelMode`判断当前线程信息中`pSBTrack`->`xxxEndScroll`。而上面我们知道，所有的窗口都在同一线程中创建，所以这里的判断也可以通过！

整理一下流程：

HOOK `KernelCallbackTable`->注册窗口类，`WNDCLASSEXW.cbWndExtra`设置为`4`->产生主窗口->以主窗口作为父窗口产生一个滚动条窗口`SrollBar`->发送`WM_LBUTTONDOWN`消息->系统处理消息初始化`SBTrack`结构并开始循环->发生`fnDWORD`回调，`fnDWORD`回调中销毁主窗口->销毁主窗口，释放扩展字节`xxxClientFreeWindowClassExtraBytes`->`xxxClientFreeWindowClassExtraBytes`系统调用回调`fnClientFreeWindowClassExtraBytesCallBack`->`fnClientFreeWindowClassExtraBytesCallBack` HOOK中调用`NtUserSetWindowFNID`更改掉窗口`FNID`->创建新窗口并调用`SetCapture`设置新窗口为捕获窗口->`xxxSBLoop`返回后解除主窗口引用->由于这是主窗口唯一的一个引用，这次解除导致彻底释放主窗口对象，`xxxFreeWindow`函数执行->由于主窗口对象的`FNID`已经被更改，`xxxFreeWindow`函数执行过程中将再一次回到用户态->用户态向新窗口发送`WM_CANCELMODE`消息->系统处理`WM_CANCELMODE`消息，释放了`SBTrack`->流程返回到内核继续执行`xxxSBTrackInit`函数最后的释放`SBTrack`->重复释放`SBTrack`！

值得说明的一点是：在上面这个流程中，完全跟`sysShadow`窗口没有关系，自然也跟本不需要HOOK `__fnINLPCREATESTRUCT`回调。

下面看一下具体代码实现：

首先，我们设置一下回调HOOK，这里就直接用`fs`来获取`PEB`了：
