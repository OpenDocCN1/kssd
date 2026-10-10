
# DIY 新浪微博 Android 手机客户端（一）（二）（三）完

> 原文：[`www.pediy.com/kssd/pediy12/131213.html`](https://www.pediy.com/kssd/pediy12/131213.html)

作者：地球仪  

原文：[`hi.baidu.com/aqtata817/blog/i...87b8947bb.html`](http://hi.baidu.com/aqtata817/blog/item/776246021f81b6d87b8947bb.html)

玩过新浪微博的人都知道，在每条微博的下方都会显示出该条微博来自于什么样的客户端

比如网页的、手机版的、Android 的、iPhone 的、iPad 的、Symbian 的、Kjava 的以及很多第三方应用等等

![](img/b2cb2daa9b8a7d7826480e3d2dd86509.jpg)

本次教程的最终目的是“偷龙转凤”，通过修改 Android 客户端让其发表的微博显示“来自 iPhone 客户端”

iPhone 多火呀！用 iPhone 多有面子呀！呵呵

第一件事还是准备好所需要的工具。工欲善其事，必先利其器！

【手机一部】

当然必须是 Android 系统的

【新浪微博 Android 客户端】

有了手机当然也要有客户端安装文件了，下载地址：http://t.sina.com.cn/mobile/android.php

【JDK】

后面要用到一些 Java 程序，所以必须安装 Java 环境

我这里装的 1.6 版，下载地址：http://www.java.net/download/jdk6/6u10/promoted/b32/binaries/jdk-6u10-rc2-bin-b32-windows-i586-p-12_sep_2008.exe

【ADB】

这个安装了 Android 的 SDK 都会有的，或着单独下载提取出来的：http://mobile.yesky.com/imagelist/2009/301/byk87b49p6gr.rar

这里面带有 USB 驱动，可以不用理会

【Tcpdump】

Linux 下的网络监视工具，下载：http://www.uuroid.com/upload/tcpdump.zip

【dex2jar】

Java 程序，可以把 dex 文件反编译成 jar 文件，下载：http://laichao.googlecode.com/files/dex2jar-0.0.7-SNAPSHOT.zip

【JD-GUI】

Java 反编译程序，可以查看 jar 文件源码，下载：http://www.duote.com/soft/7793.html

【baksmali 和 smali】

解包打包 java 源码的工具，下载：http://code.google.com/p/smali/downloads/list

【IDA】

强大的反汇编工具，不用解释了吧，我这用的 5.5 版

【91 手机助手】

主要用它来传输文件，当然你用豌豆荚等工具也是可以的

【EFFind】

文本搜索工具，这个不是必须的，只是有它的话能更快地从大量的源码中找到自己想要的数据

新浪微博开放了 API 接口，官方网站：http://www.weibo.cn/wiki/index.php

这里只需要简单地看看“授权机制说明”那部分，有两种授权方式

其中的 Basic Authentication 方式需要一个 Appkey 值，而新浪就是通过这个 key 来判断你是什么客户端

比如“优酷连接分享”的 Appkey 是 2684493555，这个值在优酷网点击转贴到新浪微博后，在 URL 地址栏都能看到

![](img/28a8b14c8d388e41cbe29f5d10b127e7.jpg)

再看浏览器的地址栏

![](img/74cc6fc132076282d45ddc11ec02105b.jpg)

也就是说，我们只要在客户端上修改对应的 Appkey 即可达到欺骗服务器的目的

这也是本教程的最终目的

不过后来分析发现，新浪官方的客户端是没有 Appkey 的，官方的客户端并不访问这个公开的 API 接口

而是有自己内部的访问接口，当然我们也不必去关心内部接口，因为我们不是要去做客户端

第一篇就暂时到这，做好准备工作，下篇实战

作者：地球仪

原文：[`hi.baidu.com/aqtata817/blog/i...ad9f9fd1c.html`](http://hi.baidu.com/aqtata817/blog/item/670d79fd975f061ad9f9fd1c.html)

上次把准备工作做好了，这次直就接“开刀”

为了方便后面的操作，先把 `smali.jar`、`baksmali.jar`、`dex2jar` 放到 `C:\123` 目录下

## 1.寻找 Appkey

将安装程序扩展名改为 rar 或 zip 都可以，只要能用工具解压就行，将其中的 `classes.dex` 文件拖出来放到 `C:\123` 目录下

进入 CMD 控制台，输入命令：`dex2jar.bat classes.dex`

![](img/045b4ffcba97ad6f22eec13a1024d73d.jpg)

成功后会在该目录下生成一个 `classes.dex.dex2jar.jar` 文件

再用 JD-GUI 打开它，选择“File - Save All Sources”将源码全部提取出来

提取出的文件是一个 zip 压缩包，将其解压到 `C:\123\yuanma`

微博公开的 API 接口文档显示，如果使用 Basic Authentication 授权方式的话

必须在所有访问接口参数里带有“`source=appkey`”

我们这里打开 EF Find 搜索刚才解压出的所有源码文件，搜索字符为“`source=`”

很可惜，什么都没找到，再继续尝试“`appkey`”等字符，依然无所获

![](img/49326a52e1a964474603539f8cc58ce8.jpg)

现在基本可以验证之前所说的，官方的客户端是访问的未公开的接口，不需要 Appkey

为了找到接口地址和相关参数，我们需要在手机上抓包

## 2.寻找未公开接口及参数

这里要请出 ADB 和 Tcpdump 了，注意先把 ADB 目录添加到环境变量中去，省去了在控制台下输入路径的麻烦

打开 91 手机助手，把 Tcpdump 放到手机的 `/data/local` 目录下

![](img/f03a8b914b3ce735a48e14b76534a765.jpg)

但是它还没有权限运行，需要修改文件权限，我们要进入到 `adb shell`

命令：`adb shell chmod 777 /data/local/tcpdump`

![](img/93b7e79720c8494eec062f6384cd162d.jpg)

查看一下属性，已经有了所有权限

![](img/2b855065e21c79cd614fc527f621aa73.jpg)

再进去手机的 shell，进入刚才的目录下

```
adb shell
cd data
cd local
```

现在准备抓包了，在启动前先进入手机微博的登录窗口，随便输入帐号密码，不要登录

再回到控制台启动 tcpdump，命令：`./tcpdump -w dump.txt -s 0 -v port 80`

![](img/a3437ebaca14ff75ec7dfd67637f7f9d.jpg)

tcpdump 已经处于监听状态，回到手机里了，点“登录”

会发现在控制台窗口提示“Got 12” 表示抓到了 12 个包

好了，让它停止工作，按下 Ctrl + C 键即可

用 91 手机助手刷新目录，可以看到多了个 `dump.txt` 文件

![](img/57679130ba44bb8f5e77b91607da09ad.jpg)

直接将其拖出来，这里说一下，这个文件应该是用 Wireshark 来查看的

之所以不用是因为这里并不需要很详细的信息，只需要看个大概就可以，用记事本打开

可以看到是以 post 方式提交数据，而且也确实如之前所说，不是访问的公开接口

![](img/d497b04ac8ac84ef0e15b21e7081b799.jpg)

![](img/6a2f3ee377c7aeef0446049995c5c6d5.jpg)

`u=`帐号，`p=`密码，`c` 应该是客户端标识符，`s` 不详，`ua` 应该只是统计数据用的

如果你会编程，这里最好做个程序测试下参数，我这里就直接说测试结果了

`u`，`p`，`c`，`s` 参数是必须的，否则就登录不成功

尝试把 `c` 参数改为“`iphone`”，但是也无法登录成功，所以 `s` 参数应该是通过标识计算出来的

## 3.解密 S 参数

我们这里再回到第一步，从源码中找线索，打开 EF Find 搜索“`login.php`”，果然，找到一个

仔细看看代码，发现将用户名+密码作为参数传递给 `calculateS` 函数了

![](img/0d9cb18b7a13f66f2bca224ee555c2ed.jpg)

跟踪 `calculateS` 函数，可以看出没有复杂的加密，只是 MD5 加密了下，然后固定的取出几个字符作为 S 参数

而 S 参数也要和 C 参数对应才行，服务器会效验

![](img/669df901366cee53e41f8163fd3f1e67.jpg)

到这里，关于客户端标识加密的方法知道了，我们还要将其改为 iPhone 的

下篇再讨论如何获取 iPhone 的 Key 及如何修改 Android 客户端

作者：地球仪

原文：[`hi.baidu.com/aqtata817/blog/i...44a90a713.html`](http://hi.baidu.com/aqtata817/blog/item/2fb58c12c890e1144a90a713.html)

## 1.寻找 iPhone 的 Key

iPhone 的安装包为 ipa 格式，官方并不提供 ipa 文件下载

我这里找到的一个地址：http://3g.sina.com.cn/tv/soft/weibo/weibo_10243010.ipa

用 IDA 打开 ipa 文件，有很多文件，我们点击 Size 排序一下，选择最大的那个 Weibo

![](img/996a604fe44002bbe39609b0a250cd5a.jpg)

按下 Shift+F12 调出字符串窗口，设置过滤字符类型，细心找下就能发现一个类似 Key 的字符

![](img/afdc8c080b946df94511f5262eb99cda.jpg)

经过测试后，确定该字符串正是 iPhone 的 Key

## 2.重建 Android 客户端

我们现在要做的就是将 Android 客户端中的 Key 替换掉，再将“`android`”字符替换为“`iphone`”

还记得第二篇里提取出的 `classes.dex` 文件吗？这次再用 `baksmali.jar` 将其反编译

进入控制台，输入：`java -jar baksmali.jar -o C:\123\out classes.dex`

![](img/094575b91674d668a2f2fc7df5943159.jpg)

这时 `C:\123` 目录下又多出了个 `out` 目录，其中就是反编译出的源码

我们用 EF Find 搜索这个目录，搜索 Android 的 Key 和替换为 iPhone 的 Key

搜索标识字符“`\"android\"`”替换为“`\"iphone\"`”，注意有引号，这样才表示是一个字符串

修改好后再次进入控制台，输入命令：`java -jar smali.jar C:\123\out -o C:\123\_classes.dex`

![](img/02c7bde3b5ff14dc297016a495881de7.jpg)

成功后，会多出一个 `\_classes.dex` 文件，改名为 `classes.dex` 后覆盖到客户端安装包里并删除 META-INF 文件夹

但是这个安装包并不能成功安装，因为文件已经被修改，需要重新进行签名

我们把这个安装包改名为 `jj.apk` 放到 Auto-sign 目录下，双击 `Signapk.bat` 即可

稍等一会，目录下会生成签名后的 apk 安装包，这就是最终的修改后的 Android 客户端了

（当然这里并不一定要用 Auto-sign，别的 apk 签名工具都可以）

最后，我们先将手机上的客户端卸载掉，再安装这个修改后的客户端并测试

![](img/05c680a2e60bc3ba83d1c88200f57679.jpg)

看看下方显示的“来自 iPhone 客户端”，成功啦！

![](img/a2139b3f7794073a45debdb62cf3dc55.jpg)

最后提一下 iPad，这个并没有访问内部接口，而是访问的公开接口，Appkey 是 2849184197

好了，教程到这里就全部结束了，谢谢！
