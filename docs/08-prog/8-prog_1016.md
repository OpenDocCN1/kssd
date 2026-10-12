
# 创建流氓的快捷方式[带源码]

> 原文：[`www.pediy.com/kssd/pediy11/104552.html`](https://www.pediy.com/kssd/pediy11/104552.html)

最近大牛们貌似都喜欢玩快捷方式。都玩出花来了，这种流氓的创建方法经常被流氓软件用。当然现在已经成为一种常规方法了，大家见怪不怪了。

效果图如下(上面的 IE 是系统的正宗快捷方式，下面是我山寨出来的。区别看右键菜单，如果恶意定义了首页。嘿嘿~~)：

![](img/ed11f7d91f197d81ca241f2767cc386e.jpg)

![](img/9c6d762e2156cc8dce7e2e2cd415c6b4.jpg)

![](img/46b8b3e574b67349a320670a93bffdfe.jpg)

原理就是在注册表新建 `guid`，利用 Windows 系统的特性。

主要操作了注册表里的三个地方:

`HKEY_CLASSES_ROOT\CLSID`

`HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID`

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace`

具体大家看我写的代码。很有意思的是 `Attributes` 的值。当为 `0` 时，所创建的快捷方式就是无敌的，没有删除选项。要右键时出现删除选项则要改为 `0x24`。如果为 `0x89`，则可以复制这个快捷方式。为 `0xf0400174` 时，右键菜单就类似于点我的电脑时的样子。

如果大家还有这个参数的详细资料，希望能共享学习。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

参考资料:

引用: `http://www.cnblogs.com/iinsnian/archive/2008/06/17/1224132.html`

`http://www.bomi.gov.cn/Article/ShowArticle.asp?ArticleID=3011`

`http://bbs.ikaka.com/showtopic-8682001.aspx`

附件是我的演示程序和源码

使用方法: `CreateLnk.exe "运行我的 QQ(&R)" QQ2009 "D:\Program Files\Tencent\QQ2009\Bin\QQ.exe"`

效果如下:

![](img/5c73231694587ca27ec57bfc61f39b99.jpg)

说在最后:

如果大家不幸中招了。可以先去 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace` 下找到快捷方式的 `guid`，然后再利用附件提供的程序进行删除(或者利用其它第三方程序也行，亦或可以手工查找删除)。

例:`CreateLnk.exe guid`

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || [`guid_CreateLnk[Src & Bin].zip`](104552/739934/37054.zip) |
|---|---|
