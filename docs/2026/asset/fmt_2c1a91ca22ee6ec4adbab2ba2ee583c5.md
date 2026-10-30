
# [原创]第5题解题过程

## 反调试分析

程序运行后输入回车验证，没有任何提示，于是 OD 附加进程，但是发现无论是硬件断点还是内存断点，只要程序一运行到下断的位置就会直接崩溃，各种版本的 OD 过反调试都失效：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_3m18d74cytu29ga.png)](https://bbs.kanxue.com/upload/attach/201706/752211_3m18d74cytu29ga.png)

后来发现这是一个神奇的反调试，核心代码在驱动层，并且实现方法也很简单，主要就是在底层遍历找到`crackme`进程将调试端口置为 0：

![](https://bbs.kanxue.com/upload/attach/201706/752211_4j5rinam4lbu28d.png)

而这个检测触发的条件需要应用层程序发送一个控制码来触发：

![](https://bbs.kanxue.com/upload/attach/201706/752211_r50rpca40i7mukn.png)

分析一下应用层程序对应的控制码部分，一共有 2 个函数里发送了该控制码，以下是一个线程函数里专门每隔 3 秒钟发送一次该控制码进行反调试检测：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_wlvxb4418z4cjk4.png)](https://bbs.kanxue.com/upload/attach/201706/752211_wlvxb4418z4cjk4.png)

另外一处是在验证输入`key`时，在把`key`传给驱动程序前做了一次检测：

![](https://bbs.kanxue.com/upload/attach/201706/752211_zcq4hk27okfsfis.png)

这样一来，要过反调试有两个选择，一种是`patch`应用层程序，把这两处发送的控制码的地方简单`patch`，但是有个弊端就是程序需至少发送一次该控制码给驱动层才能将`bCheck`值改成 1，后面才会去正常验证`key`返回结果。这样的话就比较麻烦，所以用另外一种选择，修改驱动，在处理该控制码的位置将调用反调试函数的代码`patch`掉，然后修正一下校验和，在包含该驱动的`exe`文件对应处修改`patch`后的数据：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_i1nokmfc4w6t6n9.png)](https://bbs.kanxue.com/upload/attach/201706/752211_i1nokmfc4w6t6n9.png)

这样卸载驱动程序后重新运行程序，便可以高枕无忧地进行动态调试了。下面开始分析验证算法。

## 验证算法分析

此时可以正常下断，`GetWindowText*`可以跟踪到输入串，发现检查了输入串的长度为 6 才进行校验：

![](https://bbs.kanxue.com/upload/attach/201706/752211_m03gvqvf49xv9lx.png)

然后把输入串翻转一下直接发送给驱动程序：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_9xdfmee4qlabidn.png)](https://bbs.kanxue.com/upload/attach/201706/752211_9xdfmee4qlabidn.png)

驱动层计算完结果直接返回，看着像 MD5 串：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_i3pmekvp6c35v5s.png)](https://bbs.kanxue.com/upload/attach/201706/752211_i3pmekvp6c35v5s.png)

接着把该 16 进制串计算一下 MD5：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_0g74aveyvb2b0oy.png)](https://bbs.kanxue.com/upload/attach/201706/752211_0g74aveyvb2b0oy.png)

最后取其子串进行比较，一致的话则成功：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_bsj5e3nah9z8ral.png)](https://bbs.kanxue.com/upload/attach/201706/752211_bsj5e3nah9z8ral.png)

可见问题的关键在于把输入串传递给驱动进行计算的过程了，由于输入串长度为 6 比较小，比较容易进行暴力枚举进行破解，然后又有两个选择，一个是直接当驱动是个黑盒，在枚举过程中将字符串传给驱动去计算再返回校验，另一种就是去分析驱动的算法来提高暴力枚举的“效率”。本人尝试了两种方法，发现确实速度提供了不少，毕竟减少了通信的读写过程。上面提到，驱动返回的结果有点像 MD5 串，一开始以为是修改后的 MD5 算法，但是仔细过一遍应用层和驱动层的 MD5 算法能够发现，其实算法是一致的，也就是都用的是 MD5 算法，但是驱动层又另外对输入串进行了一次处理，导致返回的 MD5 值并不是原输入串的 MD5：

![](https://bbs.kanxue.com/upload/attach/201706/752211_fa1qbatsu6okt2w.png)

这样的话就清晰很多了，都是 MD5 算法，枚举算法好办了，省去了和驱动通信的额外开销，可以专心地跑脚本了……贴个比较暴力的 6 层循环枚举：

[![](https://bbs.kanxue.com/upload/attach/201706/752211_xoxu3srwz5w81g2.png)](https://bbs.kanxue.com/upload/attach/201706/752211_xoxu3srwz5w81g2.png)

就这样吧，看装备看人品了哈。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [crack.zip](javascript:void(0);)

  （1.55MB，14 次下载）
