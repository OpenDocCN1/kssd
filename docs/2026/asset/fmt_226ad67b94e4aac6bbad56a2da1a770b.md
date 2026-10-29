
# [原创]带360壳分析umeng协议

## 0x00 序

为什么要带壳分析umeng协议啊，因为嘬呗。

## 0x01 charles抓包

直接打开`charles`手机设置好代理，过滤掉肯定无关的链接，从手机打开需要抓包的软件。发现只有2条网络请求 483K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4D9L8$3N6Q4x3X3g2#2L8h3g2F1k6#2)9J5k6h3y4G2L8g2)9J5c8X3q4H3M7q4)9#2k6X3I4G2k6%4x3.

![图片描述](upload/attach/201712/630232\_nf6kmrr28cvbfz3.png)

一看就知道是`umeng`的协议，但是参数加密了，只能反编译看看他是如何加密的。挂起`jadx`打开`apk`，发现`apk` 360加固了，没有办法只能脱壳。拿出我的脱壳神机google5儿子(自己编译的`android4.4`的系统，修改了内核代码，能过掉`proc`文件状态的反调试)

## 0x02 针对性反编译&&过掉签名验证

可是安装的时候提示

![图片描述](upload/attach/201712/630232\_zj3a6ljncau2cbk.png)，

我靠设备版本太低，没关系，反编译修改一下最低`sdk`版本。

![图片描述](upload/attach/201712/630232\_mpc4nnqs9myl1t1.png)

加上`-s`参数,只反编译资源文件，然后打开`apktool.yml`这个文件

![图片描述](upload/attach/201712/630232\_5pcjfk0rytlew3f.png)

把这里修改成19.ok。重打包提示签名被修改，没关系继续安装`xposed` `hook`签名模块（`DisableVerifySignature.apk`）。然后软件就可以打开了。

## 0x03 过掉反调试&&脱壳（不一样的脱壳）

根据吾爱 欧阳锋锋大神的帖子 d4dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7#2z5e0f1&6y4U0m8Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`. 过掉 时间差，`rtld_db_dlactivity` 这两个反调试一路走到这里

![图片描述](upload/attach/201712/630232\_7muqe8sis6i9dnv.png)

dump 出`dex`文件。再次拖入`jadx`发现还是不对啊。是真实代码，可是没有`umeng`的代码。这是怎么回事啊。和我之前脱的软件不一样，难道360又升级了。带着疑惑我又打开了我自己写的一个通用脱壳软件。先给大家介绍下我的软件：

本工具是一个`apk`软件，打开后`listview`显示手机上安装的所有用户`apk`（图标➕`apk`名字），然后可以选择需要脱壳`apk`，然后再运行需要脱壳`apk`。即可完成脱壳。

代码实现: （`java`+`jni`）

1. `xmlpull`解析选择`apk`的`apkminifest`，获取其包名，`ApplicationName`, `MainActityName`。

2. `Xposed` `hook` `android.content.ContextWrapper` `attachBaseContext`得到壳的`classloder`

3. 根据`classloder`继续`hook`真正类的`oncreate`。

4. 继续根据`classloader`获取内存中的所有`cookie`。

5. 继续调用`jni`。根据`cookie`能转换成`DexFile`,然后其实就是解析`dex`文件了。

6. 在内存其实他的指向都是完整的。只不过没有在连续的地方。`dump`下来的也就不完整。那么我就分部`dump`。1. `classdef`之前的，2. `classdef`之后的，3. `classdef`，4. `classdata`，5. `opcode`。然后修改指向组合。

## 0x04 dump后的代码修复

通过这个软件`dump`下来的`dex`文件有60个，我把每个`dex`文件都打开看看，发现`jar`包都单独是一个`dex`文件，感觉和分包一样。想了想再把他们组合再修复oncrete 肯定是需要很长时间。不修复了。只看`umeng`这个`dex`把。

`jadx`打开`dex`搜索`host`

![图片描述](upload/attach/201712/630232\_m00hsz7qb7tut00.png)

直接就找到了。

![图片描述](upload/attach/201712/630232\_n2l964zmhb9cjk1.png)

看到了参数传递了`byte[]`，继续跟踪调用的地方。

![图片描述](upload/attach/201712/630232\_b3obcd00tad832z.png)

跟到这里发现 调用了`ch`的`b`函数，里面肯定是处理了`a`变量，最后返回了 `byte[]`.但是我怎么搜索都收拾不到`a`变量在哪里调用了，继续跟踪 `b`函数，

跟了一圈发现`umeng`的代码写的真是好啊，各种继承，接口，越看越晕都不知道谁掉的谁。

![图片描述](upload/attach/201712/630232\_3b5t3ecyehfru7q.png)，

怎么和我之前 抓包的情况不一样啊。之前抓过几次都是 几个变量拼接，然后在调用 通用加密函数，虽然加密函数可能在`jni`，可能有`ollvm`混淆，但是都能看懂大概啊。这个是越看越懵。

最后才想起原来`dump`出来的指令被优化了，需要修复

![图片描述](upload/attach/201712/630232\_vu6apuq5x1s1cc4.png)

![图片描述](upload/attach/201712/630232\_fxlsa47hsb3sg9a.png)

全局搜索替换所有优化的地方，修复完看到了引用的地方

这下代码看懂了

![图片描述](upload/attach/201712/630232\_t8jbmx5ygost31d.png)

![图片描述](upload/attach/201712/630232\_lc20bu3w6c6204n.png)

![图片描述](upload/attach/201712/630232\_foxcc9wu0ufxd31.png)

他把每一个参数都单独取出来转换成`byte[]` 然后写入`OutputStream`

既然看懂了那么就看看她这几个参数都是啥直接传进去就ok了。刚好他有一个`tostring`函数

![图片描述](upload/attach/201712/630232\_2zq58b7ibnx9dr1.png)

## 0x05 带壳hook原始类

拖出来的`dex`没有修复不能 动态调试，那么我`hook`一下打印出来就好了

![图片描述](upload/attach/201712/630232\_3nfdgy2uxpfa0bi.png)

![图片描述](upload/attach/201712/630232\_mtn7fjd1eckgcdv.png)

参数是都打印出来了，简单的能看懂，看不懂的继续跟代码发现又他娘的是加密了，不过还好加密函数还是同一个，但是看了看加密的串是10个对象的100个参数。

![图片描述](upload/attach/201712/630232\_zw104lkgfx8gomd.png)

![图片描述](upload/attach/201712/630232\_t098c1o9yqyrc8m.png)

继续`xposed` `hook`打印参数把，幸好他们的类名有规律

![图片描述](upload/attach/201712/630232\_j232k9v3enxg100.png)

并且有2个类是他们的组合

![图片描述](upload/attach/201712/630232\_z1dq8amrexogsqj.png)

继续看懂参数：有的参数是设备固定值，有些参数是固定值的`md5`，有些参数是固定参数加上时间戳的`md5`.都看懂了模拟代码。ok大功告成。继续看下一条协议。

## 0x06 打印堆栈查看调用关系

第二条协议其实还是这个接口，就是参数多了一点，多的那点就是第一条协议的返回值。虽然看懂了，但是不知道她是怎么解析返回值的啊，

![图片描述](upload/attach/201712/630232\_l8hwqvwv2ufharg.png)

`install_channel` 找到这个参数 `hook` 赋值的地方，打印堆栈

![图片描述](upload/attach/201712/630232\_rlz5kz9bhqwzuul.png)

发现是这里读的

![图片描述](upload/attach/201712/630232\_zj5vv7pvze6de83.png)

## 0x07 后续

嗯。加密，解密逻辑都找到了，模拟代码把。直接拿他代码就可以，稍微修改一下，根据`hook`下来的参数，多比较几次。测试，请求参数一摸一样的还原。。。。

由于不会排版，排版是参考了别人的，

最后，感谢论坛中各大神的分享，让我学习到了很多东西。

## 0x00 序

为什么要带壳分析umeng协议啊，因为嘬呗。

## 0x01 charles抓包

直接打开`charles`手机设置好代理，过滤掉肯定无关的链接，从手机打开需要抓包的软件。发现只有2条网络请求 f1bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4D9L8$3N6Q4x3X3g2#2L8h3g2F1k6#2)9J5k6h3y4G2L8g2)9J5c8X3q4H3M7q4)9#2k6X3I4G2k6%4x3.

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_nf6kmrr28cvbfz3.png)

一看就知道是`umeng`的协议，但是参数加密了，只能反编译看看他是如何加密的。挂起`jadx`打开`apk`，发现`apk` 360加固了，没有办法只能脱壳。拿出我的脱壳神机google5儿子(自己编译的`android4.4`的系统，修改了内核代码，能过掉`proc`文件状态的反调试)

## 0x02 针对性反编译&&过掉签名验证

可是安装的时候提示

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_zj3a6ljncau2cbk.png)，

我靠设备版本太低，没关系，反编译修改一下最低`sdk`版本。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_mpc4nnqs9myl1t1.png)

加上`-s`参数,只反编译资源文件，然后打开`apktool.yml`这个文件

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_5pcjfk0rytlew3f.png)

把这里修改成19.ok。重打包提示签名被修改，没关系继续安装`xposed` `hook`签名模块（`DisableVerifySignature.apk`）。然后软件就可以打开了。

## 0x03 过掉反调试&&脱壳（不一样的脱壳）

根据吾爱 欧阳锋锋大神的帖子 f1fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7#2z5e0f1&6y4U0m8Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`. 过掉 时间差，`rtld_db_dlactivity` 这两个反调试一路走到这里

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_7muqe8sis6i9dnv.png)

dump 出`dex`文件。再次拖入`jadx`发现还是不对啊。是真实代码，可是没有`umeng`的代码。这是怎么回事啊。和我之前脱的软件不一样，难道360又升级了。带着疑惑我又打开了我自己写的一个通用脱壳软件。先给大家介绍下我的软件：

本工具是一个`apk`软件，打开后`listview`显示手机上安装的所有用户`apk`（图标➕`apk`名字），然后可以选择需要脱壳`apk`，然后再运行需要脱壳`apk`。即可完成脱壳。

代码实现: （`java`+`jni`）

1. `xmlpull`解析选择`apk`的`apkminifest`，获取其包名，`ApplicationName`, `MainActityName`。

2. `Xposed` `hook` `android.content.ContextWrapper` `attachBaseContext`得到壳的`classloder`

3. 根据`classloder`继续`hook`真正类的`oncreate`。

4. 继续根据`classloader`获取内存中的所有`cookie`。

5. 继续调用`jni`。根据`cookie`能转换成`DexFile`,然后其实就是解析`dex`文件了。

6. 在内存其实他的指向都是完整的。只不过没有在连续的地方。`dump`下来的也就不完整。那么我就分部`dump`。1. `classdef`之前的，2. `classdef`之后的，3. `classdef`，4. `classdata`，5. `opcode`。然后修改指向组合。

## 0x04 dump后的代码修复

通过这个软件`dump`下来的`dex`文件有60个，我把每个`dex`文件都打开看看，发现`jar`包都单独是一个`dex`文件，感觉和分包一样。想了想再把他们组合再修复oncrete 肯定是需要很长时间。不修复了。只看`umeng`这个`dex`把。

`jadx`打开`dex`搜索`host`

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_m00hsz7qb7tut00.png)

直接就找到了。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_n2l964zmhb9cjk1.png)

看到了参数传递了`byte[]`，继续跟踪调用的地方。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_b3obcd00tad832z.png)

跟到这里发现 调用了`ch`的`b`函数，里面肯定是处理了`a`变量，最后返回了 `byte[]`.但是我怎么搜索都收拾不到`a`变量在哪里调用了，继续跟踪 `b`函数，

跟了一圈发现`umeng`的代码写的真是好啊，各种继承，接口，越看越晕都不知道谁掉的谁。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_3b5t3ecyehfru7q.png)，

怎么和我之前 抓包的情况不一样啊。之前抓过几次都是 几个变量拼接，然后在调用 通用加密函数，虽然加密函数可能在`jni`，可能有`ollvm`混淆，但是都能看懂大概啊。这个是越看越懵。

最后才想起原来`dump`出来的指令被优化了，需要修复

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_vu6apuq5x1s1cc4.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_fxlsa47hsb3sg9a.png)

全局搜索替换所有优化的地方，修复完看到了引用的地方

这下代码看懂了

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_t8jbmx5ygost31d.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_lc20bu3w6c6204n.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_foxcc9wu0ufxd31.png)

他把每一个参数都单独取出来转换成`byte[]` 然后写入`OutputStream`

既然看懂了那么就看看她这几个参数都是啥直接传进去就ok了。刚好他有一个`tostring`函数

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_2zq58b7ibnx9dr1.png)

## 0x05 带壳hook原始类

拖出来的`dex`没有修复不能 动态调试，那么我`hook`一下打印出来就好了

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_3nfdgy2uxpfa0bi.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_mtn7fjd1eckgcdv.png)

参数是都打印出来了，简单的能看懂，看不懂的继续跟代码发现又他娘的是加密了，不过还好加密函数还是同一个，但是看了看加密的串是10个对象的100个参数。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_zw104lkgfx8gomd.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_t098c1o9yqyrc8m.png)

继续`xposed` `hook`打印参数把，幸好他们的类名有规律

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_j232k9v3enxg100.png)

并且有2个类是他们的组合

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_z1dq8amrexogsqj.png)

继续看懂参数：有的参数是设备固定值，有些参数是固定值的`md5`，有些参数是固定参数加上时间戳的`md5`.都看懂了模拟代码。ok大功告成。继续看下一条协议。

## 0x06 打印堆栈查看调用关系

第二条协议其实还是这个接口，就是参数多了一点，多的那点就是第一条协议的返回值。虽然看懂了，但是不知道她是怎么解析返回值的啊，

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_l8hwqvwv2ufharg.png)

`install_channel` 找到这个参数 `hook` 赋值的地方，打印堆栈

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_rlz5kz9bhqwzuul.png)

发现是这里读的

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_zj5vv7pvze6de83.png)
