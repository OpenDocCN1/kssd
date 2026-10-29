
```java
public void onBannerAdLoadSuccess() {
String v0 = "MiMoAdBannerAdapter";
MLog.w(v0, "load banner ad success ,start render");
if(MiMoAdBannerAdapter.access$100(this.a) != null) {
MLog.w(v0, "start show banner ad");
MiMoAdBannerAdapter.access$100(this.a).showAd(MiMoAdBannerAdapter.access$200(this.a), MiMoAdBannerAdapter.access$300(this.a));
}
else {
MLog.w(v0, "load banner ad success,but banner ad is null");
this.a.notifyLoadError(new MMAdError(-2000));
this.a.trackDspLoad(String.valueOf(-2000), null);
}
}
```

上述调用栈关系也可以使用动态调试的方式，一步到位。我没有使用调试的方法，因为这个广告在我的手机上出现的概率太低，想调试一次太难了，大家理解一下哈。

## 基于上述分析，去掉主页上方广告位的思路：

- 源头去除：`onBannerAdLoadSuccess` 函数中，`showAd` 调用移除。缺点：可能影响了 app 原本逻辑。
- 尾部去除：设置 `view` 的 `visibility`。优点：基本不影响 app 原本逻辑。

昨天调试的时候保存了广告栏显示的代码源头。大家可以自行分析一下，看是否跟我找到的一样。

```java
//com.xiaomi.ad.mediation.mimo.MiMoAdBannerAdapter
public void loadAndShow(AdInternalConfig arg1, AdLoadAndShowListener arg2, AdLoadAndShowInteractionListener arg3) {
super.loadAndShow(arg1, arg2, arg3);
AndroidUtils.runOnMainThread(this.mMainHandler, new MiMoAdBannerAdapter$a(this, arg1));
}
```

```java
public class MiMoAdBannerAdapter$a implements Runnable {
public MiMoAdBannerAdapter$a(MiMoAdBannerAdapter arg1, AdInternalConfig arg2) {
this.b = arg1;
this.a = arg2;
super();
}
public void run() {
this.b.loadBannerAd(this.a);
}
}
```

### 直接将 `run` 函数 `return`，实测有效

```java
java -jar ~/env/baksmali-2.5.2.jar d classes2.dex
cd out/com/miui/zeus/mimo/sdk/ad/banner
vim a\$3.smali
//去掉invoke-virtual。相当于run()函数什么都没有做
```

### 还有一种方案：模拟广告位 `close` 的方法，对 app 逻辑无任何影响，大家自行尝试

![](upload/attach/202106/855618\_ECRUNMV7ESA8W7T.png)

# 点击礼物盒跳转至广告 activity，如何 disable 该功能

## 老办法，先用 `dumpsys` 看看 activity 是哪个

```shell
admin@C02D7132MD6R bin % adb shell dumpsys activity activities
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)
Display #0 (activities from top to bottom):
Stack #1:
...
...
\* Hist #1: ActivityRecord{1f34e49 u0 com.outfit7.mytalkingtomfree/com.miui.zeus.mimo.sdk.ad.reward.RewardVideoAdActivity t3}
packageName=com.outfit7.mytalkingtomfree processName=com.outfit7.mytalkingtomfree
launchedFromUid=10139 launchedFromPackage=com.outfit7.mytalkingtomfree userId=0
app=ProcessRecord{382ce36 4930:com.outfit7.mytalkingtomfree/u0a139}
Intent { cmp=com.outfit7.mytalkingtomfree/com.miui.zeus.mimo.sdk.ad.reward.RewardVideoAdActivity (has extras) }
...
...
resizeMode=RESIZE\_MODE\_RESIZEABLE
\* Hist #0: ActivityRecord{64ced92 u0 com.outfit7.mytalkingtomfree/com.jinke.Main t3}
packageName=com.outfit7.mytalkingtomfree processName=com.outfit7.mytalkingtomfree
launchedFromUid=10139 launchedFromPackage=com.outfit7.mytalkingtomfree userId=0
app=ProcessRecord{382ce36 4930:com.outfit7.mytalkingtomfree/u0a139}
Intent { flg=0x10000000 cmp=com.outfit7.mytalkingtomfree/com.jinke.Main (has extras) }
```

## `RewardVideoAdActivity` 与主页（`Main`）不一个 activity，暴力拦截 `startActivity` 可行？

先挂断点查看 `startActivity` 调用栈，我断在了 `instrumentation->execSartActivity` 方法。

![](upload/attach/202106/855618\_ZQAJK3BPHUJE4JY.png)

### 很可惜，尝试了头部去除以及尾部去除，重打包运行会有两个问题：

1. activity 确实被拦截掉了，但是页面会卡在一个 `view` 中，显示加载中...
2. 按返回键尝试将卡 `view` 的界面关掉，礼物盒没有正常开启奖励，肯定是影响了游戏逻辑。

奖励没拿到。。。。毛用都没有

## 继续硬头皮找关键点吧

### 广告 activity 显示有一个特点，`close` 按钮隐藏后显示

广告加载完之后，右上角有一个 X 按钮，会显示出来。未加载完的时候隐藏。所以就在 `Activity` 中找找有没有相关的 `close` 或者 `onClick` 关键字。

```java
//com.miui.zeus.mimo.sdk.ad.reward.RewardVideoAdActivity
private void k() {
int v0 = com.miui.zeus.mimo.sdk.utils.a.c(this.i.Y());
if(v0 == 0) {
v0 = l.a("mimo\_reward\_view\_end\_page\_landscape");
}
this.h = LayoutInflater.from(((Context)this)).inflate(v0, this.g, true);
String v0\_1 = this.i.M();
String v1 = this.i.L();
Bitmap v0\_2 = BitmapFactory.decodeFile(v0\_1, this.o);
Bitmap v1\_1 = BitmapFactory.decodeFile(v1, this.o);
this.h.findViewById(l.c("mimo\_reward\_flv\_video")).setImageBitmap(v0\_2);
this.h.findViewById(l.c("mimo\_reward\_icon")).setImageBitmap(v1\_1);
this.h.findViewById(l.c("mimo\_reward\_title")).setText(this.i.h());
this.h.findViewById(l.c("mimo\_reward\_summary")).setText(this.i.g());
this.h.findViewById(l.c("mimo\_reward\_dsp")).setText(this.i.i());
View v0\_3 = this.h.findViewById(l.c("mimo\_reward\_jump\_btn"));
((TextView)v0\_3).setText(this.i.O());
b v1\_2 = new b();
this.y = v1\_2;
((com.miui.zeus.mimo.sdk.anim.a)v1\_2).b(v0\_3).a(1200).a(-1).b(1).a(new AccelerateDecelerateInterpolator()).c();
this.h.findViewById(l.c("mimo\_reward\_close\_img")).setOnClickListener(((View$OnClickListener)this));
this.g.setOnClickListener(((View$OnClickListener)this));
}
```

`void k()` 这个函数中使用了 `LayoutInflater`，`mimo_reward_close_img`，`setOnClickListener(this)`。太明显了，毫无压力。

```java
//com.miui.zeus.mimo.sdk.ad.reward.RewardVideoAdActivity
private void p() {
int v0;
if(this.i.c()) {
this.n();
if(!TextUtils.isEmpty(this.i.u())) {
v0 = 1;
}
else {
goto label\_11;
}
}
else {
label\_11:
v0 = 0;
}
if(v0 != 0) {
this.m();
}
else {
this.f.setVisibility(8);
this.k.setVisibility(8);
this.g.setVisibility(0);
b v0\_1 = this.y;
if(v0\_1 != null) {
((com.miui.zeus.mimo.sdk.anim.a)v0\_1).c();
}
ViewFlipper v0\_2 = this.u;
if(v0\_2 != null) {
v0\_2.stopFlipping();
}
com.miui.zeus.mimo.sdk.utils.analytics.b.a(this.i.m(), this.i, "END\_PAGE\_VIEW", "load\_success", 0, "");
}
}
```

搜到了，就在 `RewardVideoAdActivity` 代码中，代码还是那么的明显，毫无压力。

`this.g.setVisibility(0)`，这里的 `0` 是一个 flag，就是将 `view` 显示出来。

### 在 `RewardVideoAdActivity.p()` 函数打断点，调试一把，看看调用栈

![](upload/attach/202106/855618\_PKWHM75Y5KWWM9B.png)

源头是通过 `onCompletion` 调过来的，`com.miui.zeus.mimo.sdk.video.TextureVideoView$3` 实现了 `MediaPlayer.OnCompletionListener` 接口。

```java
public class com.miui.zeus.mimo.sdk.video.TextureVideoView$3 implements MediaPlayer$OnCompletionListener {
public com.miui.zeus.mimo.sdk.video.TextureVideoView$3(TextureVideoView arg1) {
this.a = arg1;
super();
}
public void onCompletion(MediaPlayer arg2) {
TextureVideoView.c(this.a, 5);
TextureVideoView.d(this.a, 5);
if(TextureVideoView.e(this.a) != null) {
TextureVideoView.e(this.a).hide();
}
if(TextureVideoView.h(this.a) != null) {
TextureVideoView.h(this.a).onCompletion(TextureVideoView.d(this.a));
}
}
}
```

原来广告是通过 `MediaPlayer` 播放的，然后播放是否完成是通过 `MediaPlayer` 的回掉接口调过来的。

### 模拟 `MediaPlayer` 播放完成，模拟点击 `close` 按钮，实现插件

#### 首先要拿到 `MediaPlayer` 实例

手有点疼了。。。。

主要是通过 `jeb`，查看交叉引用关系。

```java
//com/miui/zeus/mimo/sdk/video/TextureVideoView
public Map m;
public int n;
public int o;
public Surface p;
public MediaPlayer q;
```

`TextureVideoView` 是 `VideoAdView` 的成员。

```java
//com.miui.zeus.mimo.sdk.video.VideoAdView
public TextureVideoView a;
public ImageView b;
public FrameLayout c;
public boolean d;
public static final String e = "VideoAdView";
```

`VideoAdView` 是 `RewardVideoActivity` 的成员。

```java
public class RewardVideoAdActivity extends Activity implements View$OnClickListener, a {
public static final String a = "key\_baseadinfo";
public static final String b = "RewardVideoAdActivity";
public static final String c = "key\_exposure";
public static final long d = 60000;
public EventRecordFrameLayout e;
public VideoAdView f;
```

结论：可以通过拿到 `RewardVideoActivity` 实例，然后反射一步步拿到 `MediaPlayer` 实例。

#### 如何拿到 `RewardVideoActivity` 实例

这可能是个老生常谈的问题。方式可能有好几种吧，下面是我能想到的：

1. `ActivityThread` 中保存了当前进程的 activity 列表：

   ```
   final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
   ```

2. 拦截 `ActivityThread.H`，这玩意是一个 `Handler`，第一时间通过 `system_server` 将 activity 的生命周期回掉过来。

3. 通过 `frida` 或者其他 hook 工具，在 `activity->onCreate` 或者 `onResume` 这里下手。

4. 通过注入 `Instrumentation` 的方式，管控 activity 生命周期，可以拦截到 `onCreate` 以及 `onResume` 等。

我使用了最后一种方案，用 `java` 写了点代码，最后编译为 `smali` 文件后，通过静态代码注入重打包了 `apk`。

# 贴代码

## `instrumentation` 注入：
