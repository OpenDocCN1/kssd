
![13.png](https://bbs.kanxue.com/upload/attach/202001/394321_6K6QGKSUCEJJ6SR.jpg)

这样简单太多了，已经不记得是第几次被中文路径坑了，又想用英文系统了。

### 5

把`Util!SMBIOSHelper::IsSupportDevice`修正一番，替换原始的`utlil.dll`，直接运行软件，在华为的友商电脑中成功启动。

![14.png](https://bbs.kanxue.com/upload/attach/202001/394321_BVFAE8UF2FYNPUN.jpg)

开始尝试连接手机。

点击我的手机->扫码连接，提示需使用华为浏览器扫描。

因为我不是华为手机，所以是不是下载一个华为浏览器就OK了呢。

一番搜索，在华为市场找到了华为浏览器的app。

![15.png](https://bbs.kanxue.com/upload/attach/202001/394321_NQ44QXMMMYEQ2CE.jpg)

兴致勃勃地装到了我的小米手机上，嗯，友商之间很友好，并没有出现什么异常情况。

打开浏览器，点开二维码扫描，扫描软件管家二维码，提示需要安装华为移动服务，ok，安装。

![16.jpg](https://bbs.kanxue.com/upload/attach/202001/394321_PD7M6RNQ5AN6F2T.jpg)

安装完成之后，啥也没提示，就光秃秃一个网页页面，电脑助手。

懵逼中，再次扫描，同样地情况。

脑筋一转，把链接复制发到了电脑，浏览器打开，看到了下载按钮。

![17.png](https://bbs.kanxue.com/upload/attach/202001/394321_8QFP8B59KX2JG4U.jpg)

第三个app了，继续把华为电脑助手安卓版装入了小米，瞬间感觉`MIUI`好像被华为全家桶强X了一下ε(┬┬﹏┬┬)3。

重新扫描，这次果然不一样了，浏览器跳转到了助手页面，提示配对。

这中间又出现了n多问题

怀疑蓝牙服务不正常、二维码生成不正常、电脑管家是否对手机校验、分析是否还有`check`...

不想细说了，反正经过一番折腾，终于手机和电脑连上了。

![18.png](https://bbs.kanxue.com/upload/attach/202001/394321_7NMU5MUNG9MNWA6.jpg)

但是这也没有多屏协同啊，就一个手机助手啊。

难道我哪里又弄错了？

去搜了搜多屏协同的帖子，算是找到了问题。

![19.png](https://bbs.kanxue.com/upload/attach/202001/394321_EHTKVZY8DN8A4UW.jpg)

TMD，原来下载的软件版本低了，而我还眼瞎地没看到上面的下载列表的`10.0.2.59`版本。

真是被自己蠢哭了。

下载最新版，重新修正了`util.dll`，这次连上果然不一样了。

![20.png](https://bbs.kanxue.com/upload/attach/202001/394321_5XJB4D89CNFF4XJ.jpg)

但是，最终还是没能用上多屏协同，尽管我已经安装了3个华为的app，但就是手机不支持（`EMUI`系统功能）。

![21.png](https://bbs.kanxue.com/upload/attach/202001/394321_GYBB87KTBBK2N47.jpg)

看到这些条件，我真是感到心累，“这是对冲动最好的惩罚”。

### 6

是的，虽然折腾了这么久，最后也没用上这个牛逼的功能，但是我很充实（哭）。

因为我为其他拥有华为手机但没有华为电脑然后想体验华为多屏协同功能的大伙做出了我的贡献。

我满足了。

哎，卑微的我去咸鱼看看二手的华为手机，或者去问问华为`EMUI`适配不适配友商的手机了。

或者哪位大佬有换代的华为手机邮我一个，地址是：`XXXX`。

哎，曾经的我没钱买华为，现在的我依然没钱买华为。

申明：虽然华为手机副总裁已经说过不反对破解，但这里还是郑重申明，文章仅仅是做技术研究，如侵删，谢谢。

最后，小声地说，如果想体验的朋友，请到[d66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3S2C8k6$3N6Q4x3V1k6t1N6h3q4%4k6h3W2Q4y4h3k6b7b7@1#2S2L8X3q4Y4k6i4u0Q4y4h3k6z5b7R3`.`.](https://bbs.kanxue.com/elink@620K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3S2C8k6$3N6Q4x3V1k6t1N6h3q4%4k6h3W2Q4y4h3k6b7b7@1#2S2L8X3q4Y4k6i4u0Q4y4h3k6z5b7R3`.`.)下载试用，务必阅后即焚。

文件较大，也可以公众号后台回复：`HuaweiNB`获取网盘链接。

如果觉得内容还不错，欢迎关注公众号：[汉客儿](https://bbs.kanxue.com/elink@f5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2u0F1c8W2S2U0L8W2)9#2k6V1I4B7x3$3I4X3f1h3N6#2N6U0S2q4f1h3g2v1N6H3`.`.)。

交流群：[753894145](https://bbs.kanxue.com/elink@83eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7M7g2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3i4K6y4r3i4K6g2X3N6%4k6Q4x3@1b7I4x3o6t1%4i4K6t1$3j5h3#2H3i4K6y4n7K9#2)9K6c8o6g2%4N6K6k6@1L8p5t1`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-2-29 11:34

被`anhkgg`编辑，原因：
