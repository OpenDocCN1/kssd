
# [原创] Android 分析之路（一）

之前一段时间看了一下 Android 开发，发现 Android 还是挺有意思的。最近看了些 Android 分析的资料，决定自己动手试试。本人比较菜，此贴纯属学习总结贴，请各位指教！![](https://bbs.kanxue.com/view/img/face/1.gif)

找了几个带有恶意代码的样本用于学习和分析。

首先，解压 APK。我使用了`apktool`将 APK 文件解压成`smali`文件，另外还用`zip`解压工具解压了一份。一份用于看`smali`，一份再用`dex2jar`和`xjad`转成 Java 文件，看 Java 代码。

![](https://bbs.kanxue.com/upload/attach/201307/516280_q6cgvqmrq9z5rby.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_6qr9982lbv4aa4i.png)

文件夹`classes_dex2jar`中存放的就是软件的 Java 文件，将这些 Java 文件加入`sourceInsight`中，这样方便查看。

第二，先从整体上看看这个项目，首先看的就是`AndroidManifest.xml`文件。其中主要注意一些敏感权限就行，比如发送短信、拨打电话、连接网络，但是这些也是作为一个参考。在文件中有两句话应该要提高警惕：

![](https://bbs.kanxue.com/upload/attach/201307/516280_lph1him3rzrgkwo.png)

一个是在收到短信的时候有事件响应，另一个是去电时候有事件响应。

第三，开始 Java 文件的分析，这里需要使用刚刚导入`sourceInsight`的那些文件了。

此时可以直接全文搜索关键字，如`sms`、`Internet`、`url`等敏感字眼，但是，有时可能没有用。这里作为学习，还是从入口函数开始：

![](https://bbs.kanxue.com/upload/attach/201307/516280_daanz1p43h6ud2y.png)

可以看出在创建的时候就触发了很多自己实现的函数，接下来可以继续跟进函数，首先是`load()`：

![](https://bbs.kanxue.com/upload/attach/201307/516280_kklijllvp4hnvac.png)

可以看出`load()`方法就已经侵犯了用户的隐私了。跟入上面花圈的函数，就会发现它读取了用户的 SIM 卡的信息，获取用户的手机号码等，并将这些信息存入`Constants`中。这里有个`Constants.data`是一个加密的字段，对于这种已经侵犯了隐私的软件，这种加密字段就非常可疑了，所以我是想通过反编译出来的函数来解密这个字段：

![](https://bbs.kanxue.com/upload/attach/201307/516280_m7dxl37bze0l1t1.png)

事实证明，反编译器不是万能的，有些东西还是出不来的。这个时候我又想到了`smali`文件，但是要将大量的`smali`转成 Java 工作量太大，不合适，而且我还没有学习`smali`语法，所以……

不过静态调试不行，又想到了动态调试，如果可以下断点，或是输出就好了。这里我使用了`smali`注入的方法，直接将解密后的`data`值打印，找到`Constants.smali`文件，添加`Log.d`对应的语句：

![](https://bbs.kanxue.com/upload/attach/201307/516280_vgd9hto6e2ngwik.png)

添加完成之后，不着急打包，先继续分析一下。回到`load`方法，发现这个加密字段被作为 JSON 存放了起来，这里就提示我们要小心后面软件调用到的 JSON。

回到入口，继续查看：

![](https://bbs.kanxue.com/upload/attach/201307/516280_062cfd8v3n0izkq.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_lwir2xcctt3ttcv.png)

这里开始开启服务操作，之后启动`MainService`：

![](https://bbs.kanxue.com/upload/attach/201307/516280_rkxswh1zmr7ni5c.png)

对于画线的方法，其实就是调用了`ThreadOperationRun`，这个在`ThreadOperation.java`中定义了，而这个`ThreadOperationRun()`没有做别的事情，就是在尝试发送短信。它调用了`startRepeatTimer()`、`startSmsTimer`、`sendSms`：

![](https://bbs.kanxue.com/upload/attach/201307/516280_raj794q619i8u68.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_bt8zvg6oy9quhw9.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_8pmdioyz0kwur0y.png)

对于最后一个没有什么好说的了，就是发送短信。前面两个的话都有同一条语句：

![](https://bbs.kanxue.com/upload/attach/201307/516280_vtfw3bi9hyheq8o.png)

而在`MainReceiver`中的`OnReceiver()`，我们可以轻易地找到收到短信触发事件的语句：

![](https://bbs.kanxue.com/upload/attach/201307/516280_xthrn0j75x8l6zo.png)

之后就是获取电话号码、发送短信、删除短信的业务了：

![](https://bbs.kanxue.com/upload/attach/201307/516280_b7fjc45l29tbdbx.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_020njbwp44nt8fw.png)

注意`startRepeatWaitTimer`，它有一次启动了`MainReceiver`。现在就可以打包 APK 并签名安装了，用`logcat`可以看见刚刚加入的输出信息（我还在一些敏感函数前加了一些 log，方便分析流程）：

![](https://bbs.kanxue.com/upload/attach/201307/516280_4v8tl9m8fi9g8z6.png)

![](https://bbs.kanxue.com/upload/attach/201307/516280_s289ddpvzhf1hmr.png)

至此这个样本初步分析完成。样本中还有一些打电话和打开网页下载的地方，由于本人初学，`smali`语法还没有看，Java 文件又被破坏，所以暂时分析不了，等这几天看完后再分析之！![](https://bbs.kanxue.com/view/img/face/12.gif)![](https://bbs.kanxue.com/view/img/face/12.gif)

小弟初入 Android 门庭，如有不对的地方，请各位大牛指正！![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

再次声明，这个就是学习笔记，不要喷哦~![](https://bbs.kanxue.com/view/img/face/28.gif)![](https://bbs.kanxue.com/view/img/face/28.gif)

附件：`326K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6x3$3x3U0f1J5y4K6M7%4x3K6W2Q4x3U0k6#2K9#2)9K6c8o6x3I4x3U0p5$3z5o6b7^5x3U0f1`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [图片1.png](javascript:void(0);)（9.72kb，12次下载）
- [图片2.png](javascript:void(0);)（12.10kb，9次下载）
- [图片3.png](javascript:void(0);)（8.58kb，20次下载）
- [图片4.png](javascript:void(0);)（7.30kb，4次下载）
- [图片5.png](javascript:void(0);)（10.37kb，114次下载）
- [图片6.png](javascript:void(0);)（3.35kb，4次下载）
- [图片7.png](javascript:void(0);)（9.47kb，19次下载）
- [图片8.png](javascript:void(0);)（4.85kb，4次下载）
- [图片9.png](javascript:void(0);)（4.54kb，4次下载）
- [图片10.png](javascript:void(0);)（4.97kb，5次下载）
- [图片11.png](javascript:void(0);)（2.86kb，4次下载）
- [图片12.png](javascript:void(0);)（3.28kb，4次下载）
- [图片13.png](javascript:void(0);)（1.74kb，4次下载）
- [图片14.png](javascript:void(0);)（0.87kb，4次下载）
- [图片15.png](javascript:void(0);)（1.74kb，4次下载）
- [图片16.png](javascript:void(0);)（11.30kb，9次下载）
- [图片17.png](javascript:void(0);)（8.98kb，9次下载）
- [图片18.png](javascript:void(0);)（43.20kb，36次下载）
- [图片19.png](javascript:void(0);)（55.35kb，26次下载）
