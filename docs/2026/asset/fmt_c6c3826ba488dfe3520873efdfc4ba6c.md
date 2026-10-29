
在使用该直播软件的过程中, 惊讶地发现里面有很部分用户竟还是在校**学生**, 包括用户和主播. 通过某种手段添加其中一个主播聊天得知, SM行业“水很深”, 除了直播间收礼物外, 主播一般还会将高质量用户引流到其他社交平台(刷“飞机”加微信). 然后私底下会视情况进行现实调教. 用该主播的话说, ”一天2个或者3个, 元旦那天最多, 一天有8个”, 这每一“个”的收费自然也是不低的, 内容大概有XX、XX和XX, 重口味一点的有XX、XX等...

对于该“圈子”的是非且不去谈, 这个公司游走在法律边缘, 还致未成年人的身心健康于无物, 老子动画片(进击的巨人)都看不了, 这种直播竟没有相关部门管束? 对于成年的观众, 大家都有辨别是非的能力, 就不多说了吧. 至于主播, 很多人就是作为工作或者兼职, 赚的也是辛苦钱, 也不多说了吧. 毕竟, 言多必失, 你懂的.

---

![contact\_me](upload/attach/202003/844554\_CHCU6V8WDDNDRPU.jpg)

\[qn\]: <mark class="encrypted">a64K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3&6F1P5X3S2Q4x3V1k6I4L8X3y4D9K9b7`.`.</mark>
\[baidu\]: upload/attach/202003/844554\_RHB5GK4UFACVUBH.png
\[code\]: upload/attach/202003/844554\_6VCDG9AGHCFJDUW.png
\[shell\]: upload/attach/202003/844554\_JARU947QPQ2DB3Y.png
\[shot\]: upload/attach/202003/844554\_RX3NWJ5XRK8PVYS.png
\[price\]: upload/attach/202003/844554\_SWCVAC8KVSWW9VH.png
\[user\]: upload/attach/202003/844554\_TTHK55DZ4BSWF4R.png
\[ui\]: upload/attach/202003/844554\_VV2UPWBSUQK9W6C.png
\[unpack\]: upload/attach/202003/844554\_EFPM4YZWABH3GMC.png
\[webpage\]: upload/attach/202003/844554\_9G85Q9HDENGWNYU.png

> 注：本文首发于我的个人博客 50dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0

> 需要看雪账号，因此特来混积分:)

当你凝视深渊的时候, 深渊也在凝视你 —— 尼采.

从大约13年开始, 各大直播平台就如雨后春笋般冒出来, 犹记得当初还经常在午休时看DOTA的直播下饭. 后来不知道什么时候开始不再只是游戏直播, 而是一度刮起了女主播打擦边球的歪风邪气, 再后来严厉整顿, 各个平台也才逐渐收敛野蛮的发展, 逐渐步入正轨. 然而, 在阳光照不到的地方, 依然暗流涌动...

# 前言

发现这一迹象的契机, 是在总部位于卢森堡的某联盟网站发现开始有大量中国人拍摄的视频, 这在国际网站中是比较少见的. 而这些视频有个共同特征, 都打有某某直播的水印. 当时并没有在意, 加上年底也比较忙没空深究. 直到今天闲下来, 才想起来一探究竟.

首先按照水印的logo去搜索, 发现一个网站:

![webpage](https://bbs.kanxue.com/upload/attach/202003/844554_9G85Q9HDENGWNYU.png)

主页倒是中规中矩, 一副文艺社交的样子, 但是看标题就知道不一般啊. SM是什么东西, 熟悉二次元的朋友应该都不会陌生. 尽管我也有所了解, 但随着深入探究还是被震惊到, 所以建议未成年的小兄弟三思而百度.

说到百度, 来看看百度对这个站的收录是如何的:

![baidu](https://bbs.kanxue.com/upload/attach/202003/844554_RHB5GK4UFACVUBH.png)

竟然没有收录! 果然藏得很深. 什么叫暗网? 不是洋葱才叫暗网, 这也是一种暗网. 回过头来, 这个网站显然只是一个下载站, 也没有web访问界面, 因此我们要深入的话需要下载其手机APP. 网站提供了安卓和iOS版本, 我们先选择前者来进行分析.

分析APP的第一步当然是使用了. 以防万一, 这种APP还是先放在沙盒里运行, 保不准有什么顽固木马. 意料之中的是不支持x86的模拟器, 换到arm环境中正常启动. 贵圈的画风是怎么样的的? 截了一张图大家感受下:

![shot](https://bbs.kanxue.com/upload/attach/202003/844554_RX3NWJ5XRK8PVYS.png)

里面大致分为直播和回放, 当然天下没有免费的午餐, 每个视频或者直播都只能免费播放8秒, 8秒之后自动弹出购买窗口, 平均购买价格为288-988金币不等, 金币价格如下, 支持微信支付哦:)

![price](https://bbs.kanxue.com/upload/attach/202003/844554_SWCVAC8KVSWW9VH.png)

# 探索

大致玩了一遍这个APP, 基本功能都了解了, 接下来就从技术角度去探索下里面的实现. 一般来说逆向都要事先定下目标目标, 但是我们这次没有很明确的目标, 只是随便逛逛, 顺便复习下逆向技能.

## 脱壳

首先在刚才的首页下载安卓apk, 用`JADX`看看大致的结构:

![shell](https://bbs.kanxue.com/upload/attach/202003/844554_JARU947QPQ2DB3Y.png)

看样子是使用了360的安全加固, 正好之前写了个脱壳脚本, 一键脱掉:

![unpack](https://bbs.kanxue.com/upload/attach/202003/844554_EFPM4YZWABH3GMC.png)

之后每个`dex`就可以正常使用`jadx-gui`来打开看了, 不过这里分享一个我比较喜欢的操作, 即用`jadx`命令行分别反编译各个`dex`, 然后用`rsync`合并到一处.

```
mkdir t380.jadx

for i in {1..4};do

    JAVA_OPTS="-Xmx8G" jadx -j 1 -r -d t380_${i}.jadx unpacked_classes${i}.dex

    rsync -a t380_${i}.jadx t380.jadx

done

```

为什么要这么做? 因为使用`jadx-gui`分别打开每个`dex`时候不论是查找函数还是查找引用都不太方便, 合并到一处可以直接用`find`/`grep`来查找. 这对于上百兆的巨型APK逆向还是很实用的.

## 逆向

虽然这个APK只有50多M, 但代码量也不小, 直接看有点无从下手, 就从收费的地方开始吧! 毕竟这是整个环境最核心的部分. 我们打开一个直播间, 等8秒后弹出付费窗口, 定位这个地方:

![ui](https://bbs.kanxue.com/upload/attach/202003/844554_VV2UPWBSUQK9W6C.png)

当然, 还有个比较直接的方法, 直接使用`adb`来查看当前置顶的activity:

```
$ adb shell dumpsys window windows | grep Focus

  mCurrentFocus=Window{be2e8cd u0 com.zhuoyigou.dese/com.zhuoyigou.dese.ui.live.activity.PlayerLiveActivtiy}

  mFocusedApp=AppWindowToken{73e5eb8 token=Token{bfded1b ActivityRecord{2d0fb2a u0 com.zhuoyigou.dese/.ui.live.activity.PlayerLiveActivtiy t50}}}

```

这个直播软件有个独特的地方是里面除了直播, 还有大部分是视频回放, 观看视频回放同样是需要收费. 定位方法也是同样, 回放的activity为`com.zhuoyigou.dese.ui.live.activity.PlayBackActivity`.

这里再分享个小技巧, 逆向过程中经常会需要用编辑器打开某个文件, 定义个shell函数可以加速该流程:

```
function jopen() {

    p=`echo -n $1 | tr . /`

    file=$p.java

    gvim -R $file

}

```

这里以`PlayBackActivity`为例, 在`jadx`代码目录下, 可以直接这样打开文件:

```
jopen com.zhuoyigou.dese.ui.live.activity.PlayBackActivity

```

哈, 扯远了, 来看看弹窗付费的地方是如何实现的吧, 该类的部分代码风格如下:

![code](https://bbs.kanxue.com/upload/attach/202003/844554_6VCDG9AGHCFJDUW.png)

做了一点对抗, 但是却没有完全混淆, 看来是不太了解木桶原理啊. 根据里面的方法名称, 发现一个比较有趣的函数:

```
private void showTimeOutDialog() {

    if (isMainActivityTop(GiftActivity.class)) {

        reflashGiftActivity("1", null);

    }

    if (!isFinishing()) {

        this.mLiveTimeOutDialog = new LiveTimeOutDialog(this, 2131493319);

        this.mLiveTimeOutDialog.show();

        this.mLiveTimeOutDialog.setGold(this.mPlayBackBean.getLiveRoom().getPayMoney());

        this.mLiveTimeOutDialog.setTitle("replay");

        this.mLiveTimeOutDialog.setCancelable(false);

        this.mLiveTimeOutDialog.setBuyOnClick(new 12(this));

        this.mLiveTimeOutDialog.setNoBuyOnClick(new 13(this));

    }

}

```

直接把这个dialog禁掉试试. 测试可以用一些hook框架如`xposed`或者`frida`, 这里选择后者, 因为可能会需要做native的分析(有没有发现这个`Activity`的`onCreate()`函数是native的?). `frida`更新速度飞快, 而且很多用法和特性都没有写到文档里, 多看看别人写的脚本也许会有很多灵感.

劫持函数是常规操作了:

```
const PlayBackActivity = Java.use("com.zhuoyigou.dese.ui.live.activity.PlayBackActivity");

PlayBackActivity.showTimeOutDialog.implementation = function() {

    log("skip showTimeOutDialog()");

}

```

重新附加进程, 打开直播间, 发现确实不弹窗了, 但是过十几秒后视频窗口仍然会自己返回. 了解安卓开发的应该都知道, `Activity`返回一般是调用`finish()`函数, 不过搜了一圈发现调用的地方太多了, 懒癌发作不想一个个检查, 于是祭出`frida`大法, 在`finish()`函数中查看堆栈:

```
[+] skip showTimeOutDialog()

[+] skip finish()

[+] java.lang.Exception: printStackTrace here:

    at android.app.Activity.finish(Native Method)

    at com.zhuoyigou.dese.ui.live.activity.PlayBackActivity$11.handleMessage(PlayBackActivity.java:561)

    at android.os.Handler.dispatchMessage(Handler.java:102)

    at android.os.Looper.loop(Looper.java:148)

    at android.app.ActivityThread.main(ActivityThread.java:5539)

    at java.lang.reflect.Method.invoke(Native Method)

    at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:726)

    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)

    at de.robv.android.xposed.XposedBridge.main(XposedBridge.java:107)

```

根据`finish()`的堆栈查看`com.zhuoyigou.dese.ui.live.activity.PlayBackActivity$11`类中的`handleMessage()`函数来进行验证, 确实是有这个逻辑:

```
void handleMessage(Message msg) {

    switch(msg.what) {

        // ...

        case 1002:

            this.this$0.mPointProgressBar.setVisibility(8);

            PlayBackActivity.access$1400(this.this$0).onPause();

            if (PlayBackActivity.access$400(this.this$0)) {

                PlayBackActivity.access$1500(this.this$0);

            } else {

                PlayBackActivity.access$1600(this.this$0);

            }

            PlayBackActivity.access$600(this.this$0).sendEmptyMessageDelayed(AidConstants.EVENT_NETWORK_ERROR, 11000);

            return;

        case AidConstants.EVENT_NETWORK_ERROR /*1003*/:

            if (PlayBackActivity.access$400(this.this$0) && PlayBackActivity.access$900(this.this$0).isShowing()) {

                PlayBackActivity.access$900(this.this$0).dismiss();

            }

            this.this$0.finish();

            return;

        // ...

    }

}

```

在11秒后触发事件, 直接退出窗口. 在8秒的试看后还给你3秒选择是否购买哦:) 作为验证, 我们直接对`finish()`函数下手, 发现禁用`dialog`的同时禁用`finish()`函数, 就可以绕过金币购买的限制了, 想看多久看多久.

## 云存储

虽然现在可以无限制看片, 但正事还是要做的. 下一步看看是哪家对象存储提供了服务, 说不定还能找到硬编码的`key`. 在代码中瞎逛, 又(?)发现一个有趣的函数:
