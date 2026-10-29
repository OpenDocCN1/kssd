
# Activity_main.xml文件

```
<WebView
    android:id="@+id/eeeewebview"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
/>
```

# MainActivity.java文件

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // WebView
    WebView webView = (WebView) findViewById(R.id.eeeewebview);
    webView.loadUrl("4fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8b7.");
    webView.setWebViewClient(new WebViewClient(){
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            //使用WebView加载显示url
            view.loadUrl(url);
            //返回true
            return true;
        }
    });
}
```

写完之后运行，发现报错，无法打开网页（`net::ERR_CLEARTEXT_NOT_PERMITTED`），经过搜索在 `manifest` 内设置 `usesCleartextTraffic` 为 `true` 即可。

![图片1](https://bbs.kanxue.com/upload/attach/202212/922338_6REPJMN6FE73QN3.png)

![图片2](https://bbs.kanxue.com/upload/attach/202212/922338_93QCBZ36ZUZ4DWJ.png)

可以看到百度已经被打开了，啊～因为这个 app 是用来测试其他东西的，所以会看到三个奇奇怪怪的按钮。

## 2. URI

`Uri` 代表要操作的数据，Android 上可用的每种资源（图像、视频片段、网页等）都可以用 `Uri` 来表示。从概念上来讲，`URI` 包括 `URL`。

`Uri` 的基本结构是：

```
大致为[scheme:]scheme-specific-part[#fragment]
细分为[scheme:][//authority][path][?query][#fragment]
```

`path` 可以存在多个，以 `/` 连接：`scheme://authority/path1/path2/path3?query#fragment`

`query` 可以带参数的返回值也可不带：`scheme://authority/path1/path2/path3?id=1#fragment`

举例如下：

```
http://b12K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2S2i4K6u0W2j5$3^5./about?id=1
```

`scheme` 是在 `:` 之前，所以他匹配的是 `http`。

`authority` 是在 `//` 之后，所以 `a3dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2W2k6h3g2S2i4K6u0W2j5$3&6Q4c8e0c8Q4b7U0S2Q4z5p5g2Q4c8e0g2Q4z5o6g2Q4b7U0k6Q4c8e0g2Q4b7f1k6Q4b7U0W2Q4c8e0g2Q4b7V1q4Q4z5e0b7` 是 `authority`。

`path` 自然对应的就是 `about` 这个页面。

`query` 对应的是 `id=1`。

在安卓内，除了 `authority` 和 `scheme` 必须存在，其他的可以选择性的要或者不要。

将一个 `url` 解析成 `uri` 对象的操作是 `Uri.parse("6a7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4@1f1J5i4K6R3H3i4K6W2p5i4K6t1&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1#2i4@1t1H3i4K6R3$3i4@1f1%4i4K6V1&6i4@1u0q4i4@1f1#2i4@1u0m8i4@1p5$3i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1#2i4K6W2p5i4K6R3H3i4@1f1^5i4@1p5%4i4@1p5K6i4@1f1$3i4K6W2q4i4K6V1H3i4@1f1$3i4K6R3^5i4K6V1H3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1t1^5i4@1q4m8N6i4u0A6i4@1f1#2i4@1q4r3i4@1t1&6i4@1f1^5i4@1t1I4i4@1p5I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1#2i4@1q4r3i4@1t1&6i4@1f1#2i4K6R3#2i4@1t1$3i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1#2i4K6R3#2i4@1t1$3i4@1f1@1i4@1u0n7i4K6V1$3i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6V1H3i4K6R3@1i4@1f1%4i4@1p5%4i4K6S2p5i4@1f1$3i4K6V1K6i4K6S2p5i4@1f1@1i4@1u0p5i4K6W2o6i4@1f1@1i4@1u0m8i4K6R3$3")`。

## 3. Intent

### 1. Intent 功能

`Intent` 是各大组件之间通信的桥梁，Android 有四个组件，分别是 `Activity`、`Service`、`Broadcast Receiver`、`Content Provider`；组件之间可以进行通信，互相调用，从而形成一个 app。

每个应用程序都有若干个 `Activity` 组成，每一个 `Activity` 都是一个应用程序与用户进行交互的窗口，呈现不同的交互界面。因为每一个 `Activity` 的任务不一样，所以经常在各个 `Activity` 之间进行跳转，在 Android 中这个动作是靠 `Intent` 来完成的。通过 `startActivity()` 方法发送一个 `Intent` 给系统，系统会根据这个 `Intent` 帮助你找到对应的 `Activity`，即使这个 `Activity` 在其他的应用中，也可以用这种方法启动它。

![图片3](https://bbs.kanxue.com/upload/attach/202212/922338_GYPNEGZWYVMGUNJ.png)

### 2. 显式 Intent 和隐式 Intent

`Intent` 包括两种，一是显式另一个是隐式。显式 `Intent` 通常是已经知道要启动 `Activity` 的包名，多发于同一个 app 内；隐式 `Intent` 只知道要执行的动作是什么，比如拍照、录像、打开一个网站。

那么隐式的 `Intent` 如何启动一个组件呢？如果没有约束的话可能会造成一些后果，所以在 `Manifest` 文件内定义了 `intent-filter` 标签，如果组件中的 `intent-filter` 和 `intent` 中的 `intent-filter` 匹配，系统就会启动该组件，并把 `intent` 传给它；若有多个组件都符合，系统便会弹出一个窗口，任我们选择启动该 `intent` 的应用（app）。

在 `intent-filter` 标签中，我们可以选择三个 `intent` 的属性进行设置，包括 `action`、`category`、`data`。

![图片4](https://bbs.kanxue.com/upload/attach/202212/922338_ECQX57FKCN5SNS6.png)

上图 `intent-filter` 定义的 `action` 为 `MAIN`，代表 app 以这个 `activity` 开始。

### 3. Intent 属性

#### 1. Component

该属性是显式 `Intent` 特有的，表明要启动的类的全称，包括包名和类名。有它就意味着只有 `Component name` 匹配上的那个组件才能接收你发送出来的显式 `Intent`。

下面代码可以启动另一个 app 的主页面：

```java
Intent intent = new Intent(Intent.ACTION_MAIN);
intent.addCategory(Intent.CATEGORY_LAUNCHER);
ComponentName cn = new ComponentName(packageName, className);
intent.setComponent(cn);
startActivity(intent);
```

一个 `activity` 是否能被其他 app 的组件启动取决于 `android:exported`，`true` 能，`false` 不能。如果是 `false`，这个 `activity` 只能被相同 app 的组件启动，或者是相同 user ID 的 app 的组件启动。

![图片5](https://bbs.kanxue.com/upload/attach/202212/922338_MB36CQUTMEWD5QM.png)

**如果显式设置 `exported` 属性，不管这个 `activity` 有没有设置 `intent-filter`，那么 `exported` 的值就是显式设置的值。**

**如果没有设置 `exported` 属性，那么 `exported` 属性取决于这个 `activity` 是否有 `intent-filter`。**

**如有 `intent-filter`，那么 `exported` 的值就是 `true`。**

**如没有 `intent-filter`，那么 `exported` 的值就是 `false`。**

#### 2. Action

一个字符串变量，用来指定 `Intent` 要执行的动作类别（比如：`view` or `pick`）。你可以在你的应用程序中自定义 `action`，但是大部分的时候你只使用在 `Intent` 中定义的 `action`，你可以通过 `Intent` 的 `setAction()` 方法设置 `action`。

#### 3. Data

一个 `Uri` 对象，对应着一个数据。只设置数据的 `URI` 可以调用 `setData()` 方法，只设置 `MIME` 类型可以调用 `setType()` 方法，如果要同时设置这两个可以调用 `setDataAndType()`。

#### 4. Category

一个包含 `Intent` 额外信息的字符串，表示哪种类型的组件来处理这个 `Intent`。任何数量的 `Category` 描述都可以添加到 `Intent` 中，你可以通过调用 `addCategory()` 方法来设置 `category`。

#### 5. Extras

`Intent` 可以携带的额外 `key-value` 数据，你可以通过调用 `putExtra()` 方法设置数据，每一个 `key` 对应一个 `value` 数据。你也可以通过创建 `Bundle` 对象来存储所有数据，然后通过调用 `putExtras()` 方法来设置数据。

#### 6. Flags

用来指示系统如何启动一个 `Activity`（比如：这个 `Activity` 属于哪个 `Activity` 栈）和 `Activity` 启动后如何处理它（比如：是否把这个 `Activity` 归为最近的活动列表中）。

# 二：题目环境布置

### 1. Docker 存在问题

运行 `run.sh`，我自己启动了一遍 docker 环境，修改了一些部分，最终发现是在 `server.py` 文件的 `setup_emulator()` 函数中没有模拟出来手机，只是创建了一个 `AVD` 环境，并没有 `emulator` 成功。

由于自己能力有限，实在不知道如何修好这个 docker 环境，便就此搁置，导致后面 Silver Droid 利用也不完全；如若后续进步，必定再战一次。

![图片6](https://bbs.kanxue.com/upload/attach/202212/922338_6PMVZVGGV2QJGD8.png)

![图片7](https://bbs.kanxue.com/upload/attach/202212/922338_64ZNFCZ53KJMDXH.png)

### 2. server.py 脚本内函数

#### 1. adb_broadcast

`adb broadcast` 便是将服务器上的 `flag` 传给 apk 的 `FlagReceiver`，通过 `adb shell` 进入手机，可以查看到 `flag` 被存到了 `files/flag` 内。

之前有一个疑问，便是 `manifest` 文件将 `FlagReceiver` 设置为 `exported` 为 `false` 和设置了 `intent-filter`，防止外界 app 进行干扰，那么是怎么将 `flag` 传递给 `FlagReceiver` 呢？

![图片8](https://bbs.kanxue.com/upload/attach/202212/922338_S8JUT9W7K642AX4.png)

由于 `root` 的情况下，是忽略掉 `exported` 的，所以可以对其进行广播：

```
am broadcast -W -a "com.wuhengctf.SET_FLAG" -n "com.bytectf.silverdroid/.FlagReceiver" -e 'flag' 'flag{eeeeeeee}'
```

![图片9](https://bbs.kanxue.com/upload/attach/202212/922338_QAK5DTYD3AZD54V.png)

#### 2. adb_activity

通过 `intent` 传递 `url` 数据，下面可以通过 `-d` 选项来指定 `Intent data URI`：

```
am start -a android.intent.action.VIEW -d https://f5cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0
```

![图片10](https://bbs.kanxue.com/upload/attach/202212/922338_KP5PV7KXWB2EVCR.png)
![图片11](https://bbs.kanxue.com/upload/attach/202212/922338_47WZ2F2J979JJQA.png)

下面的题目介绍，都是以 `Pixel 4` 为环境打的，因为 docker 我这边模拟不起来。

同时记得自己写的 apk 要在 `AndroidManifest.xml` 内加两句话，可以让其有网络访问的权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>

<application
    android:usesCleartextTraffic="true">
```

# 三：Silver Droid

## 1. server.py 分析

主要由攻击者提供一个 `url`，在 `url` 内布置好 `exp`，从而进行达到利用的目的，具体见代码块内分析。
