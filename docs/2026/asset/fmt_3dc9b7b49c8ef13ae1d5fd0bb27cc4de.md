
`TextView`的id为`Text_Changed`，设置了宽度为`match_parent`，高度为`wrap_content`，文本内容为`@string/changed_no`，字体大小为`18sp`，居中显示。

整体布局结构为线性布局，上方是一个相对布局，下方是一个文本视图。

这个布局的外观为一个按钮位于屏幕右上角，下方是一个居中显示的文本视图。

前面说过启动`ReceiverActivity`，显示飞行模式状态，而`ReceiverActivity`的主要作用自然是显示飞行模式状态了，但是我为这个`Activity`设置了和广告界面一样的显示方式，以下是`ReceiverActivity`类的代码实现：

```
package com.example.fourmodule;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;
public class ReceiverActivity extends AppCompatActivity {
private Handler handler;
private Runnable closeRunnable;

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_receiver);

TextView textView = findViewById(R.id.Text_Changed);

Intent intent = getIntent();
if (intent != null && intent.hasExtra("isAirplaneModeOn")) {
boolean isAirplaneModeOn = intent.getBooleanExtra("isAirplaneModeOn", false);
if (isAirplaneModeOn) {
// 修改文本样式为飞行模式开启时的样式
textView.setText(R.string.changed_yes);
}
else {
// 修改文本样式为飞行模式关闭时的样式
textView.setText(R.string.changed_no);
}
}

// 初始化Handler和Runnable
handler = new Handler();
closeRunnable = new Runnable() {
@Override
public void run() {
closeActivity();
}
};

// 启动计时器，延迟五秒后关闭界面
handler.postDelayed(closeRunnable, 5000);

// 关闭按钮点击事件
Button closeButton = findViewById(R.id.receiverButton);
closeButton.setOnClickListener(new View.OnClickListener() {
@Override
public void onClick(View v) {
// 取消计时器并关闭界面
handler.removeCallbacks(closeRunnable);
closeActivity();
}
});
}

private void closeActivity() {
// 关闭界面
finish();
// 进入新的界面
startActivity(new Intent(ReceiverActivity.this, NewActivity.class));
}
}
```

我们可以从以上的`ReceiverActivity`类的代码中看出，在`onCreate`方法中，首先调用了父类的`onCreate`方法，并设置了布局文件为`activity_receiver`。然后通过`findViewById`方法获取了一个名为`Text_Changed`的`TextView`，并获取了从意图中传递过来的额外信息`isAirplaneModeOn`，根据这个信息来更新`TextView`的文本内容。

接着，初始化了一个`Handler`和一个`Runnable`对象。`Handler`用于管理线程和消息队列，而`Runnable`是一个接口，用于定义要在新线程中执行的代码块。

然后，通过`handler.postDelayed`方法启动了一个计时器，延迟五秒后调用`closeRunnable`中的`run`方法，从而关闭当前界面并启动一个名为`NewActivity`的新界面。

最后，设置了一个名为`receiverButton`的按钮的点击事件监听器，当按钮被点击时，会取消计时器并立即关闭当前界面，并启动名为`NewActivity`的新界面。

编写好代码的`Activity`自然要注册这个`Activity`了，随即我在应用清单文件中对这个`Activity`进行了注册：

```
<activity android:name=".ReceiverActivity" />
```

让我们来看看这个`Activity`长啥样：

![](https://bbs.kanxue.com/upload/attach/202311/926486_KBCRSYFM6P6NCCG.webp)

这个是接收到打开飞行模式的广播后所显示的界面，而接收到关闭飞行模式的广播后所显示的界面自然是显示飞行模式现处于关闭状态啦！

最后说一下这个简单的`APP`布局文件所需字符串全部存放在`res/values/strings.xml`文件中：

```
<resources>
<string name="app_name">FourModule</string>
<string name="button_text_one">关闭</string>
<string name="text_one">走过路过不要错过，过了这个村就没这家店了！</string>
<string name="text_two">这个是正文！</string>
<string name="changed_yes">飞行模式现处于打开状态</string>
<string name="changed_no">飞行模式现处于关闭状态</string>
<string name="stop_button">关闭服务</string>
<string name="content_text_one">检索用户字典的数据</string>
</resources>
```

至此，这个模拟点开`APP`就是广告的`APP`实现就完成了！这个简单的`APP`可能会有很多不专业的地方，但我并非专门进行安卓开发的，只是秉承着“知己知彼，百战不殆”的态度进行的简单涉猎所编写的`APP`。

### 上帝视角破解广告

接下来就是我们开上帝视角来解决掉这烦人的广告了，在这之前还需将`APK`进行打包才可以将其安装在手机或者模拟器上，打包好`APK`后我们就开始上帝视角破解之路。

首先我们要的是跳过广告界面，可以想到最直接的两种方法，一是将广告界面的显示时间改为`0`秒，二是修改应用清单文件中的`APP`启动界面的`Activity`。但是第二种方法一般不推荐使用，容易出现一些奇怪的问题，因为一般在启动`Activity`的时候都会预先加载一些数据，而第二种方法可能会导致这些需要预先加载的数据没有加载上，而导致一些奇奇怪怪的问题。

所以我们就使用第一种方法来解决这个广告界面：

按照正常流程第一步我们应该进行查壳工作，但是这是我们自己编写的简单程序，自己知道没有加壳自然就不进行查壳工作了，如果是进行逆向工作时第一步是一定要进行查壳工作的。

第二步使用`MT`管理器的`Activity`记录功能获取到我们想要得到的`Activity`类信息，获取到`com.example.fourmodule.MainActivity`这条`Activity`类信息后，然后在反编译后的`Dex`文件中筛选搜索类型为类名，然后就对`com.example.fourmodule.MainActivity`这么一搜，我们就可以看到以下信息：

![](https://bbs.kanxue.com/upload/attach/202311/926486_QFEDWDKN39SAKD3.webp)

看到这个搜索结果有人可能会有些好奇，之前在开发时什么时候写过`MainActivity1`、`MainActivity1`、`MainActivity1`、`MainActivity2`、`MainActivity$3`这几个类。这只需要我们一点进去就能明白是怎么回事了：

```
package com.example.fourmodule;

class MainActivity$1 implements Runnable {
final MainActivity this$0;

MainActivity$1(MainActivity mainActivity) {
this.this$0 = mainActivity;
}

@Override
public void run() {
MainActivity.access$000(this.this$0);
}
}
```

看起来是不是既熟悉又陌生，来，我们再来看一个段代码你就知道是怎么回事了：

```
closeRunnable = new Runnable() {
@Override
public void run() {
closeActivity();
}
};
```

这一段初始化`Runnable`的代码和`MainActivity1`是不是有相似之处，没错，`MainActivity1`是不是有相似之处，没错，`MainActivity1`是不是有相似之处，没错，`MainActivity1`就是`MainActivity`中的一个匿名内部类，而`XXX1`、`XXX1`、`XXX1`、`XXX2`、`XXX$3`什么的都是`XXX`中定义的内部类。

这是一个小插曲，我们继续，进入到`MainActivity`后，我们先来看一下反编译后的代码转成`Java`是个什么样子：

```
// Decompiled by Jadx - 479ms
package com.example.fourmodule;

import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
private Runnable closeRunnable;
private Handler handler;

protected void onCreate(Bundle bundle) {
super.onCreate(bundle);
setContentView(R.layout.activity_main);
this.handler = new Handler();
Runnable runnable = new MainActivity$1(this);
this.closeRunnable = runnable;
this.handler.postDelayed(runnable, 5000L);
((Button) findViewById(R.id.closeButton)).setOnClickListener(new MainActivity$2(this));
findViewById(R.id.closeText).setOnClickListener(new MainActivity$3(this));
}

/*
JADX WARN: Multi-variable type inference failed
*/
public void closeActivity() {
finish();
startActivity(new Intent((Context) this, (Class<?>) NewActivity.class));
}
}
```

我们是打算将广告界面的显示时间改为`0`秒，那可以看到`this.handler.postDelayed(runnable, 5000L);`这行代码，就是这行代码启动计时器，延迟五秒后关闭界面，那么我们需要将其中的`5000`改为`0`，因为我们无法修改`Java`代码，只能修改`smali`代码，所以我们需要找到`postDelayed`方法的调用位置，因为要调用某个有参数的方法，那在调用之前必须先准备好参数。欸！就这么一搜，就找到了目标：

![](https://bbs.kanxue.com/upload/attach/202311/926486_56DFX7NTF9KMJ3W.webp)

看到这里，想必大家都知道只要将`0x1388`改为`0x0`就完成了将广告界面的显示时间改为`0`秒的工作，我们保存、回编译、重新签名、重新安装一条龙服务完成后就可以看到结果了。

完成将广告界面的显示时间改为`0`秒的工作后，服务是不会开启了，因为`MainActivity`只存在`0`秒，也没法正常开启服务了，但是这个`APP`一旦接收到飞行模式改变状态的广播后依旧会弹出`ReceiverActivity`界面，我们下一步就是不让它在飞行模式状态改变时弹出`ReceiverActivity`界面。

首先我们我们可以想到以下两种方式：

第一：依旧是修改`ReceiverActivity`界面的显示时间，将其改为`0`。

第二：不让`MyReceiver`广播接收器在接收到飞行模式状态改变的广播后启动`ReceiverActivity`。

第一种方式我们之前已经讲过了，就不重复讲了，这里我们讲第二种方式来解决这个问题。

正常情况下我们是通过`MT`管理器的`Activity`记录功能获取到我们想要得到的`Activity`类信息，然后找到调用这个`Activity`的位置并处理掉调用这个`Activity`的代码。我们知道接收到飞行模式状态改变的广播后弹出来的`Activity`是`ReceiverActivity`，那么我们直接搜索它：

![](https://bbs.kanxue.com/upload/attach/202311/926486_3QMVMKZRCEPF9PT.webp)

长按`ReceiverActivity`后选择复制，我们可以看到一条信息`Lcom/example/fourmodule/ReceiverActivity;`，这条信息可以帮助我们找到是谁调用了`ReceiverActivity`类，我们拿着这一条信息按照搜索类型为代码就这么一搜索，可以看到：

![](https://bbs.kanxue.com/upload/attach/202311/926486_KJZM2AVTG93PAK6.webp)

我们直接排除`ReceiverActivity`类自己的调用，可以看到`MyReceiver`中有`Lcom/example/fourmodule/ReceiverActivity;`这条信息，我们点进去可以看到这么一段代码：
