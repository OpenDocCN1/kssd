
# [原创]梆梆加固apk的一点思考

梆梆这项产品是一款非常优秀的产品，整体的逻辑构架非常严谨。其中用到了加密，ndk，封装类等多项技术。使得经过帮帮处理的APK文件结构非常清晰，整体功能的封装又减小了整个文件被逆向的风险。

下载一个`apk`上传到梆梆云端，获取其加固包。利用加固前后的APK进行对比，发现了一个问题，其加固后的`apk`包结构非常的清晰，模块高度聚合，整个包结构中都是安卓的几大组件来完成的。而主要功能则被封装在了`java`包装类中了。

![](https://bbs.kanxue.com/upload/attach/201304/257009_tbu151l0vkfrksi.jpg)

加固后

![](https://bbs.kanxue.com/upload/attach/201304/257009_iwsn8zcf3134hhx.jpg)

加固前

前后对比发现，梆梆可以很准确的解析混淆过的`apk`文件（说明混淆对于保护文件的作用并不是那么大）。这些接收器提供者的确定则是利用`AndroidManifest`文件来确定的。

利用`apktool`解压加固包，发现和原文件中相比，多了以下三个文件

![](https://bbs.kanxue.com/upload/attach/201304/257009_x1i40waop4mmn2f.jpg)

但是`classes.jar`无法打开，判断为加密处理过。另外两个是`so`文件分别对应`arm`和`intel`平台(尚未判断出这两个`so`文件的作用)。

经过分析，发现重点在`java`封装类和`lib`库中的`so`文件中。首先观察`java`封装类，该类并没有将原`APK`作者的自定义类单纯的封装在一起，而是利用动态加载这一功能来实现的，动态的加载`classes.jar`文件然后解密来完成一些列的操作。`com.secapk.wrapper`该文件包下的三个类是其整个框架的核心。首先`ACall`类中加载`lib`库文件中的`so`文件，申明`native`函数，其次`MyClassLoader`则是用来加载解密后的`classes.jar`文件，而`Util`工具类则是完成一系列如下功能：

1. 处理器信息判断

![](https://bbs.kanxue.com/upload/attach/201304/257009_z17u6e4xl6ra7c5.jpg)

2. 拷贝`classes.jar`文件到指定位置

![](https://bbs.kanxue.com/upload/attach/201304/257009_y1ru4cbij5ufwsx.jpg)

3. 拷贝`classes.jar`到缓存中

![](https://bbs.kanxue.com/upload/attach/201304/257009_dulka7ix1i0qz8w.jpg)

4. 删除文件，主要是被更新版本的函数来调用

![](https://bbs.kanxue.com/upload/attach/201304/257009_w0aubji7qt2mcgb.jpg)

其他功能略。。。

最后是`lib`库中的`so`文件。使用`IDA`打开以后发现导出表中并没有常见的以包名开始的函数。这是因为梆梆采用了使用导出表中函数序号来实现对`native`方法的加载。该`so`文件完成了对`classes.jar`文件的解密，以及其他一些功能

![](https://bbs.kanxue.com/upload/attach/201304/257009_5q4qozpcx4q8zyn.jpg)

最后总结一下梆梆的大体构架：保留继承几大组件的类，添加一个应用类和一个`java`包(即本例中的`com.secapk.wrapper`，主要是用它完成一系列的功能操作，贯穿自定义类和`android`机制),其他自定义类封装在`classes.jar`文件中加密保存，同时使用`NDK`编程来完成对自定义类加密封装包的解密操作等。

这种方式是的被梆梆处理的文件结构非常清晰，但是整体的逻辑结构却被保护得很好，有效的保护了其处理过的文件不被抄袭。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)

（43.07kb，12次下载）

- [2.jpg](javascript:void(0);)

（21.33kb，31次下载）

- [3.jpg](javascript:void(0);)

（6.07kb，7次下载）

- [4.jpg](javascript:void(0);)

（12.13kb，9次下载）

- [5.jpg](javascript:void(0);)

（21.27kb，11次下载）

- [6.jpg](javascript:void(0);)

（31.23kb，7次下载）

- [7.jpg](javascript:void(0);)

（35.36kb，8次下载）

- [8.jpg](javascript:void(0);)

（94.38kb，10次下载）
