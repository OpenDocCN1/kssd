
# [原创]破解“贪吃蛇大作战”的签名信息服务端验证机制

## 0x00 序

第一次尝试做一些简单的逆向分析，内容比较简单，高手们莫见笑。

“贪吃蛇大作战”这个游戏最近玩的人挺多，我也在玩。5分钟限时版，最好成绩也就3000多。

我分析的版本是v2.0.1：

![](https://bbs.kanxue.com/upload/attach/201612/738921_172m8slseour3e6.png)

经过修改，玩了一把5分钟限时赛：长度69224，击杀1456。

## 0x01 反调试和签名校验

将原包重新签名，安装到手机上，一直提示网络无法连接，原包没有问题。这里很明显是将签名信息上传到了服务器端，在服务器端进行了签名校验，校验失败则断开与此客户端的连接。

写一个小程序进行注入（利用`ptrace`），对一些关键函数进行hook，比如`libc.so`的`fopen()`函数。

![](https://bbs.kanxue.com/upload/attach/201612/738921_pfm1o2mpr0op54h.png)

在`hook_entry`中，对`libc.so`的`fopen()`作inline hook，监测一下程序都打开了哪些文件。

我本意是想看看，它是否会在运行时直接去读取apk包，自己解析其中的与签名信息相关的文件。结果是没有，但发现它一直在读`cmdline`文件，猜测可能是在作反调试（未去证实，因为后面的分析和修改并未借助动态调试）。

这里说一下，假设它是自己打开apk文件，从中读取与签名相关的文件，提取签名信息。那么我们可以在某个位置放一个原包，然后hook关键函数，将其读取的文件路径修改为原包位置，即可绕过这种签名校验。

举一反三，这种方式也可以绕过大部分反调试措施。比如常见的检测`traceid`是否非0的反调方式，我们可以hook `fopen()`/`open()`，然后在它要读取该文件之前先读取该文件，并将其`traceid`重新修改为0，并将其写到sd卡某个目录下，再将打开文件的位置重定向到该文件，那它就检测不到`ptrace`了。

还有一些anti-hook机制，大概思路是校验本地文件的数据和加载到内存中的数据是否一致。通过类似方式也可以轻松绕过，一句话，因为我们可以先注入，先完成hook，先做各种Anti-anti。

## 0x02 破解签名校验

因为它没有在运行时直接`fopen()`/`open()` apk文件，所以考虑应该还是通过调用系统api读取的签名信息。

将原包解压，发现只有两个so，其中`libweibosdkcore.so`看起来是微博sdk。将另一个`libJustATest.so`拖到IDA中看一下，没加壳，并且只看到`xxx_getATestString`这么一个有用的导出函数，从名字上看，很可能就是获取上传到服务器端的校验字符串。

![](https://bbs.kanxue.com/upload/attach/201612/738921_pndgvd33jq273cq.png)

跳转到该方法，f5，进行一些简单的参数名和参数类型以及函数调用的修正。发现它里面进行了一大通的各种字符串的拼接，最后将该字符串返回（根据之前的猜测，该字符串可能就是发送给服务端的校验字符串）。

![](https://bbs.kanxue.com/upload/attach/201612/738921_ggs8tw9ysmbm5nk.png)

发现里面调用了java层`com.wepie.snake.helper.update.QiniuEtagUtil`类的`getSignString`函数。用AndroidKiller反编译一下APK包（该APK没有作任何防反编译的措施，dex也没加壳），找到`getSignString`函数。

![](https://bbs.kanxue.com/upload/attach/201612/738921_adlhf6063z72hwe.png)

没错，就是在这里调用系统API获取的签名（其实，我们可以一开始就全局搜索某些关键API，来定位获取签名的位置）。

借助Xposed hook `getSignString`方法，将正确的签名字符串通过日志打印出来。

![](https://bbs.kanxue.com/upload/attach/201612/738921_aoxgw7cp7lrwele.png)

正确的签名字符串是（作了MD5计算后的结果）：678a930b9829b54a44f92a840916f7d1

剩下的工作就简单了，修改smali，将`getSignString`的返回结果固定为上面的这个正确的签名字符串。

![](https://bbs.kanxue.com/upload/attach/201612/738921_jjycs88m3a9090b.png)

重新编译、打包、签名、安装，发现用新签名的APK包已经可以正常使用了。

## 0x03 后序

其实破解签名校验之后，基本上是想改什么改什么了，因为原包没有做任何的加壳和混淆的工作。比如，看看下面这个类，应该知道怎么下手了吧（修改的时候注意一下，它里面好像有一些简单的数据合理性校验之类的东西，我没细看）。

![](https://bbs.kanxue.com/upload/attach/201612/738921_1nh4dy5481086zl.png)

最后，大家学习就好，别做什么破坏，也别释放出什么破解版之类的东西。初次尝试一点简单的逆向分析，大牛们绕过吧。

附：我认为现在so端最有用的加固措施是llvm混淆，因为普通加解密壳从机制上来说比较容易脱掉。dex端已经出现了解释器壳（伪vmp），纯粹的类抽取的话，通过自定义rom（定制dalvik或art，遍历`class_def`加载并初始化，然后dump…）也可以脱掉大部分的。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [图片 1.png](javascript:void(0);)

  （81.65kb，6次下载）

- [图片 3.png](javascript:void(0);)

  （158.50kb，11次下载）

- [图片 4.png](javascript:void(0);)

  （187.42kb，6次下载）

- [图片 5.png](javascript:void(0);)

  （246.21kb，8次下载）

- [图片 6.png](javascript:void(0);)

  （167.45kb，10次下载）

- [图片 7.png](javascript:void(0);)

  （147.04kb，6次下载）

- [图片 8.png](javascript:void(0);)

  （91.47kb，4次下载）

- [图片 9.png](javascript:void(0);)

  （228.30kb，5次下载）

- [doc.zip](javascript:void(0);)

  （562.59kb，37次下载）
