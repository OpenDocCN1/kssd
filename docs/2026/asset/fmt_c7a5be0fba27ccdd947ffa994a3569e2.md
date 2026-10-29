
# [原创]短视频某手sig3算法调用

> 本人菜鸟，初入逆向，前段时间把sig算法通过ida阅读伪代码和arm汇编进行了还原，花了4天时间，殊不知已有大神已经发有分析帖子了
>
> link:https://bbs.pediy.com/thread-254328.htm

## 一、前言

> 目标APP的sig3算法前面已经做足了功夫，这里只陈述一下我是如何找到sig3算法的，并没有把算法还原，因为so层onload有大量ollvm，目前还处于学习阶段，还没有触及，日后学成在继续发帖分享下学习笔记。

## 二、定位入口

> 本次目标APP版本是7.2.4.13202，这里使用Charles抓包知道了sig3的全称
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_4YK5G23RW97GWUA.jpg)
>
> sig3全称是 \_NS\_sig3 ，这里将app拖入jadx-gui进行分析，使用字符串搜索大法进行初步查找：
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_R9M9R6QJ7BM335C.jpg)
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_VX2WJ3EJZMWBU2M.jpg)
>
> 通过框选区域发现，`a3`是来自`a2.a`方法，且`str2`通过上方代码得知是sig签名结果，`b2`是一个url，但是执行了`b()`可能进行了一些处理，这里暂时先不管`b2`数据是什么，先最终到最终`a`调用的地方。

## 三、算法最终定位

> `String a3 = a2.a(b2 + str2);` 跟进`a`方法的声明
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_4QBT83QGPRQ9TX6.jpg)
> 通过查看`a`的方法声明，发现是一个`interface`，这里直接右键查看调用例
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_QBQNSNZR7K6QZXN.jpg)
>
> 发现只有一处地方使用了`implements`，进去查看具体代码实现
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_KQ8FFJ2VVWE7R5D.jpg)
>
> 继续跟进`getSigWrapper`方法实现
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_H7U4K2RFGNNZWKR.jpg)
>
> 继续跟进`j.c(str)`方法实现
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_9575TRSVWBSDVUF.jpg)
>
> 发现还在继续深入调用 `String atlasSign = KSecurity.atlasSign(str);`，继续跟进方法声明。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_ZF5ZAFE7BYKPVPD.jpg)
>
> 继续阅读下代码，我们可以直接双击`str`变量，查看他引用地方：
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_YFKVVX32B42DN43.jpg)
>
> 发现是执行 `d().a(str)` ，继续跟进查看实现方法：
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_FZURNHS37B8VVN4.jpg)
>
> 这里传了个参数2，false，继续跟进。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_PWQ9XBEU5KQCH5K.jpg)
>
> 这里我们看到，他`new`了一个`i()`，并且一直在里面赋值，通过简单的课文阅读理解，我们去掉无用的代码：
>
> ```
> i iVar = new i();
> iVar.a(KSecurity.getkSecurityParameterContext().getAppkey());
> iVar.a((Map<String, String>) null);
> iVar.b(0);
> iVar.a(z2);
> iVar.b(str.getBytes("UTF-8"));
> if (i.a(this.a).b().a(iVar, "0335") && iVar.j() != null) {
> ```
>
> 最终发现是调用了`i.a(this.a).b().a(iVar, "0335")` 实现，继续跟进查看
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_MHZWS69FF3KRZ74.jpg)
>
> 发现是`interface`，继续右键查看用例。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_T5E793SZP3537XK.jpg)
>
> 找到引用的地方，双击找到具体实现 `a(i,string)`的方法
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_8PPKVKXF37WJR58.jpg)
>
> 我们继续进行简单的语文阅读理解
>
> ```
> String trim = new String(iVar.i()).trim();
> String str2 = (String) this.a.getRouter().a(10405, new String[]{trim}, KSecurity.getkSecurityParameterContext().getAppkey(), -1, false, KSecurity.getkSecurityParameterContext().getContext(), null, Boolean.valueOf(iVar.b()));
> iVar.c(str2.getBytes("UTF-8"));
> ```
>
> 我们是否还记得，他需要算法的参数都放在了`iVar`，这里他使用 `iVar.i()` 将要签名的信息拿出来，紧接着调用 `this.a.getRouter().a` 获得结果，发现`a`这里传了很多东西，估计是最终算法，我们先继续跟一下`a`方法。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_E92WYKYKJHFN5WS.jpg)
>
> 好家伙，又是`interface`，继续右键，我们稳住。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_Z3SAVAFPN2QFFZ8.jpg)
>
> 继续跟进查看实现
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_5V5Q5XS78PP4AAM.jpg)
>
> `return JNICLibrary.doCommandNative(i, objArr);` 我们发现，最终是传给了navite进行了计算。
>
> 那么我们继续hook下他都传了什么信息，这里我选择的hook点是
>
> ```
> com.kuaishou.android.security.a.a.a.a(com.kuaishou.android.security.kfree.c.i,java.lang.String)
> ```
>
> 因为最后面这2个传的东西太多，而且很多都是固定值，懒得码这么多代码。
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_DWUVXED3FXRMSYW.jpg)
>
> 编写好的frida代码，先将手机的frida运行起来，具体frida的搭建文章我们可以阅读：
>
> https://zhuanlan.kanxue.com/article-350.htm
>
> c41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3c8S2k6r3q4U0k6o6l9J5k6X3g2X3k6l9`.
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_8BS4KDGYDNAZ8HS.jpg)
>
> 我们看道传的参数是域名的路径和sig（我们第一次看到他是str+str2的合并，原来str是域名路径）
>
> 这里我们回到`a`方法，查看他最终传给navite的参数:
>
> 参数一：10405
>
> 参数二： new String[]{trim}
>
> 参数三: `KSecurity.getkSecurityParameterContext().getAppkey()`
>
> 参数四： -1
>
> 参数五：false
>
> 参数六： `KSecurity.getkSecurityParameterContext().getContext()`
>
> 参数七：null
>
> 参数八： `Boolean.valueOf(iVar.b())`
>
> 参数二就是hook打印的内容。
>
> 参数三是一个固定的appkey，通过查看appkey的调用例找到了
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_7UX3Z5CJZG4UYHR.jpg)
>
> 这里因为文章长度问题，就不具体展示细节。
>
> 参数八：就是一开始传的false 逻辑变量，这里我们可以回到一开始传的时候查看他的赋值就知道了，这里的`b()`只是获取当时赋值的结果
>
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_5UX8RZN5B3HZ3J4.jpg)
>
> 参数全部找到了，由于so层看到了大量ollvm，我就没有进一步还原，只能通过hook方式主动调用了。

## 四、so文件定位

> 因为看到这个so的加载挺难找的，这里也记录下我的查找过程。
> 我们进入`JNICLibrary.doCommandNative(i, objArr);`的定义发现并没有找到加载so文件。
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_A6YVPGCP33Q9JZG.jpg)
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_E9YR88X5XUMJ4CG.jpg)
> 这里我选择了右键查看`b`的调用例，发现有一处`new`的地方
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_KJQJUESVJY3Q4NE.jpg)
> 我们跟进去看看。
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_HYXAASQ5YFWYZYP.jpg)
> 通过种种代码的文字提示，发现他是在这里调用 `boolean a = f.a(context);` 进行加载，貌似这个是在提示加载消耗的时间。继续跟进`f.a()`查看
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_2SGN27B5HC9NGSG.jpg)
> 发现看道了一个名字叫： `kwsgmain`，加上底部提示的 retry load so ok，怀疑这个就是so文件名字，查看lib发现确实存在该文件。
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_EUPYVQ4QVK2XG2P.jpg)
> 通过后续的继续跟进这个字符串的调用，最终发现确实是这个文件
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_RSH23ZS5W826JUE.jpg)
> 但是拖进ida看到onload那一刻起，我目前先放弃了。。先继续啃课本。
> ![](https://bbs.kanxue.com/upload/attach/202004/885524_B855KEDY9YSXBBJ.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
