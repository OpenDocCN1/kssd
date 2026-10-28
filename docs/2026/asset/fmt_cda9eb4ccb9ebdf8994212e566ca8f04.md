
# [原创]对 SM-P200 平板 的 root 记录

> 这段时间学习安卓逆向的过程中,急需一台 root 机,一开始想把自己的手机 root 了,但是考虑到 root 的过程中会清除所有数据,所以就想着换一台备用机去 root,正好过去高中的时候被学校强制要求买了一台三星的 `SM-P200` 平板,高中过后这平板也闲置了下来,正好今天有需要,就把他拿出来,花了两天时间终于将平板 root 好了,所以记录一下平板 root 的过程
>
> 对于三星系列手机平板的 root 我的建议是参考此处

## 0x01 下载三星官方原版刷机包

这是最重要的一步,你需要去网上下载官方原版的刷机包,格式类似于这种 `AP`,`BL`,`CSC`

![image-20230402174733809](upload/attach/202304/963320_PXG9AVS4FNDAAZS.jpg)

如果有兴趣了解这些不同名称的刷机包的作用,可以看看这篇文章

如果是 `SM-P200` 型号的三星平板的话,可以直接下载我先前已经购买的官方原版刷机包(花了五块大洋嘞)

下载地址：e8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4P5V1H3H3d9K6q4@1M7$3E0W2d9%4c8H3M7s2q4q4f1e0k6Q4x3X3c8d9h3i4k6Y4 提取码：467m

下载的是一个 `.zip` 压缩包,解压之后就有上图所示的文件了

如果型号不一样的话, 那只能去网上找找啦

## 0x02 解锁 BL 锁 Bootloader

> Q: 为什么需要解锁 BL 锁?
>
> A: BL 锁的作用是阻止你使用非官网的刷机包进行刷机,不解锁 BL 锁,那么 root 将会失败的

**注意:解锁 BL 锁的过程将会清除平板的所有数据,请及时做好数据的备份!**

* 首先需要打开平板的开发者选项,然后开启 `OEM解锁` 选项
* 重启手机,对于三星手机来说,你需要在重启的过程中同时按住 `音量+` 和 `音量-` 这两个键,然后就会出现一个提示界面,一直 YES 点下去就可以啦
* 手机重启之后再次进入 `设置-开发者选项`,如果 `OEM解锁` 这个选项变成灰色的,那么就说明 BL 锁已经解锁成功了

如果还是对解锁过程不大清楚的话,可以去 b 站上看看这个视频

## 0x03 使用 Magisk 将刷机包中的 AP 文件刷入 root 权限

Magisk 下载地址: d92K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8%4m8B7L8$3S2F1N6%4g2Q4x3V1k6y4j5h3N6A6M7$3E0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.

下载完 apk 文件后,将平板用数据线和电脑连接起来,然后用 `adb install Magisk.apk` 命令进行安装,之后将刷机包中 `AP` 开头的文件复制到平板的根目录中

如果不清楚如何复制文件到根目录,具体方法是用数据线连接电脑后,你将在 `此电脑` 处看到

![image-20230402180832664](upload/attach/202304/963320_MG2N3G3TFWAE4BX.jpg)

双击点进去,进入这个目录中![image-20230402180907241](upload/attach/202304/963320_UHH3Q6YVCG2WSDY.jpg)

然后把先前下载的 `AP` 文件复制粘贴过去就行了

之后打开 `Magisk`,依次点击 `安装->安装到Recovery->下一步->选择并修补一个文件`,然后选择先前复制过去的 `AP` 开头的文件,点击开始后坐等 Magisk 提示 ALL done 就可以了

随后再次在电脑上进入 `此电脑\Galaxy Tab A with S Pen\Tablet\Download` 目录,你应该可以在这里找到 Magisk 最新修补好的文件,将其复制到电脑上

## 0x04 使用 Odin3 对三星平板进行刷机

**Odin3-v3.14.4 下载连接：**

35cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6k6r3k6A6L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9h3I4W2i4K6u0r3L8$3c8A6L8W2)9J5c8V1!0V1K9h3^5K6i4K6g2X3N6U0y4Q4x3X3f1I4y4q4)9J5k6e0c8Q4x3X3g2Z5N6r3#2D9

当用 USB 成功连接电脑后,这个地方应该显示为这种浅绿色,表示可以刷机

![image-20230402181717938](upload/attach/202304/963320_AGWHDV8STNE3SMW.jpg)

之后在 windows 中打开命令提示符,输入如下命令 `adb reboot bootloader` 进入 Download 界面

如果这种方法不行的话,你也可以手动进入,具体方法是先将平板关机,然后长按 `开机` 和 `音量+` 这两个键,之后会选择 `reboot to bootloader` 就可以了

之后将 `BL`,`AP`,`CSC` 文件都填进去如下图所示,点击 Start,然后坐等刷机完成就可以了

**注意: 这里的 `AP` 文件是由 Magisk 修补好的文件!**

![image-20230402181816368](upload/attach/202304/963320_XU4FUAJWNXC67A9.jpg)

重新开机后,我们用 `adb shell su` 测试一下

![image-20230402182513299](upload/attach/202304/963320_3PYP7BAZFV7XPRU.jpg)

ok 成功 root
