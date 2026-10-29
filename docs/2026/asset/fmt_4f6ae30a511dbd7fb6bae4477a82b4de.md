
```
<activity android:allowEmbedded=["true" | "false"]
android:allowTaskReparenting=["true" | "false"]
android:alwaysRetainTaskState=["true" | "false"]
android:autoRemoveFromRecents=["true" | "false"]
android:banner="drawable resource"
android:clearTaskOnLaunch=["true" | "false"]
android:colorMode=[ "hdr" | "wideColorGamut"]
android:configChanges=["mcc", "mnc", "locale",
"touchscreen", "keyboard", "keyboardHidden",
"navigation", "screenLayout", "fontScale",
"uiMode", "orientation", "density",
"screenSize", "smallestScreenSize"]
android:directBootAware=["true" | "false"]
android:documentLaunchMode=["intoExisting" | "always" |
"none" | "never"]
android:enabled=["true" | "false"]
android:excludeFromRecents=["true" | "false"]
android:exported=["true" | "false"]
android:finishOnTaskLaunch=["true" | "false"]
android:hardwareAccelerated=["true" | "false"]
android:icon="drawable resource"
android:immersive=["true" | "false"]
android:label="string resource"
android:launchMode=["standard" | "singleTop" |
"singleTask" | "singleInstance" | "singleInstancePerTask"]
android:lockTaskMode=["normal" | "never" |
"if\_whitelisted" | "always"]
android:maxRecents="integer"
android:maxAspectRatio="float"
android:multiprocess=["true" | "false"]
android:name="string"
android:noHistory=["true" | "false"]
android:parentActivityName="string"
android:persistableMode=["persistRootOnly" |
"persistAcrossReboots" | "persistNever"]
android:permission="string"
android:process="string"
android:relinquishTaskIdentity=["true" | "false"]
android:resizeableActivity=["true" | "false"]
android:screenOrientation=["unspecified" | "behind" |
"landscape" | "portrait" |
"reverseLandscape" | "reversePortrait" |
"sensorLandscape" | "sensorPortrait" |
"userLandscape" | "userPortrait" |
"sensor" | "fullSensor" | "nosensor" |
"user" | "fullUser" | "locked"]
android:showForAllUsers=["true" | "false"]
android:stateNotNeeded=["true" | "false"]
android:supportsPictureInPicture=["true" | "false"]
android:taskAffinity="string"
android:theme="resource or theme"
android:uiOptions=["none" | "splitActionBarWhenNarrow"]
android:windowSoftInputMode=["stateUnspecified",
"stateUnchanged", "stateHidden",
"stateAlwaysHidden", "stateVisible",
"stateAlwaysVisible", "adjustUnspecified",
"adjustResize", "adjustPan"] >
...
</activity>
```

这些没必要记，只需知道`<activity>`元素唯一的必要属性是`android:name`，`<activity>`元素需要在`<application>`元素当中作为子元素进行声明，例如：

```
<manifest ... >
<application ... >
<activity android:name=".ExampleActivity" />
...
</application ... >
...
</manifest >
```

而其他属性用到了再去查就好了。如果有兴趣详细了解`<activity>`元素的语法结构的请参阅`<activity>`元素参考文档：<a href="elink@0daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6Y4N6h3W2V1k6g2)9J5c8Y4c8G2M7r3W2U0M7#2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6u0r3j5h3y4@1K9i4k6A6N6s2W2Q4x3X3c8W2L8r3g2E0k6h3&6@1i4K6y4r3K9r3I4Q4x3@1c8\*7K9q4)9J5k6r3y4F1">Android 开发者 | Android Developers (google.cn)</a>。

### 声明 intent 过滤器

前面在讲应用清单文件时已经讲解了intent过滤器，intent过滤器是什么这里就不重复说了，这里主要来看看怎么声明intent过滤器。

这里我们直接来看看官方文档中是怎么说声明intent过滤器的：

要使用此功能，您需要在`<activity>`元素中声明 `<intent-filter>`属性。此元素的定义包括`<action>`元素，以及可选的`<category>`元素和`<data>`元素。这些元素组合在一起，可以指定 `Activity` 能够响应的 intent 类型。例如，以下代码段展示了如何配置一个发送文本数据并接收其他 `Activity` 的文本数据发送请求的 `Activity`：

```
<activity android:name=".ExampleActivity" android:icon="@drawable/app\_icon">
<intent-filter>
<action android:name="android.intent.action.SEND" />
<category android:name="android.intent.category.DEFAULT" />
<data android:mimeType="text/plain" />
</intent-filter>
</activity>
```

在此示例中，`<action>`元素指定该 `Activity` 会发送数据。将`<category>`元素声明为 `DEFAULT` 可使 `Activity` 能够接收启动请求。`<data>`元素指定此 `Activity` 可以发送的数据类型。以下代码段展示了如何调用上述 `Activity`：

```
val sendIntent = Intent().apply {
action = Intent.ACTION\_SEND
type = "text/plain"
putExtra(Intent.EXTRA\_TEXT, textMessage)
}
startActivity(sendIntent)
```

如果您打算构建一个独立的应用，不允许其他应用激活其 `Activity`，则不需要任何其他 intent 过滤器。您不想让其他应用访问的 `Activity` 不应包含 intent 过滤器，您可以自己使用显式 intent 启动它们。

### 声明权限

开发者可以使用`<activity>`标记来控制哪些应用可以启动某个`Activity`。如果一个`Activity`是另一个`Activity`的子`Activity`，那么父`Activity`必须在其清单文件中声明相同的权限，才能启动子`Activity`。如果父`Activity`声明了`<uses-permission>`元素，那么每个子`Activity`都必须具有相匹配的`<uses-permission>`元素。

例如，假设调用方应用程序想要调用一个名为 SocialApp 的应用在社交媒体上分享文章，则 SocialApp 本身必须定义调用它的应用所需具备的权限：

```
<manifest>
<activity android:name="...."
android:permission=”com.google.socialapp.permission.SHARE\_POST”
/>
```

然后，为了能够调用 SocialApp，调用方应用程序必须匹配 SocialApp 清单中设置的权限：

```
<manifest>
<uses-permission android:name="com.google.socialapp.permission.SHARE\_POST" />
</manifest>
```

### `Activity`的生命周期

`Activity`的生命周期是指`Activity`从创建到销毁的过程，当用户浏览、退出和返回到某个应用时，应用中的`Activity`实例会在其生命周期的不同状态之间进行转换。`Activity`类提供了一系列回调方法，这些回调方法可以让`Activity`知道某个状态已经改变，例如系统正在创建、停止或恢复某个 `Activity`，或者正在销毁该 `Activity` 所在的进程。而这些状态具体包括：

1. `onCreate()`：当`Activity`被创建时调用，通常用于初始化操作。
2. `onStart()`：当`Activity`可见但还未获取焦点时调用。
3. `onResume()`：当`Activity`获取焦点并开始与用户交互时调用。
4. `onPause()`：当`Activity`失去焦点但仍然可见时调用，通常用于保存数据或释放资源。
5. `onStop()`：当`Activity`完全不可见时调用，通常用于释放资源。
6. `onDestroy()`：当`Activity`被销毁时调用，通常用于清理工作。

为了在 `Activity` 生命周期的各个阶段之间导航转换，`Activity` 类提供六个核心回调：`onCreate()`、`onStart()`、`onResume()`、`onPause()`、`onStop()` 和 `onDestroy()`。当 `Activity` 进入新状态时，系统会调用其中每个回调。

![](upload/attach/202311/926486\_PZVVV6T8GPTJRYX.webp)

这张图全是英文看不懂？没关系下面我们来慢慢讲解。

#### `onCreate()`

`onCreate()`为必须实现的回调方法，它会在系统创建应用程序的 `Activity` 时触发，通常会用于初始化 `Activity` 的基本组件：例如，应用程序应该在此处创建视图并将数据绑定到列表。最重要的是，开发者必须在此处调用 `setContentView()` 来定义 `Activity` 界面的布局。

`onCreate()` 完成后，下一个回调将是 `onStart()`。

#### `onStart()`

当 `Activity` 进入“已开始”状态时，系统会调用此回调。`onStart()` 调用使 `Activity` 对用户可见，因为应用会为 `Activity` 进入前台并支持互动做准备。例如，应用通过此方法来初始化维护界面的代码。

当 `Activity` 进入"已开始"状态时，与 `Activity` 生命周期相关联的所有生命周期感知型组件都将收到 `ON\_START`事件(`onStart` 事件的常量)，你可以理解为当 `Activity` 进入"已开始"状态时，会告诉所有生命周期感知型组件：“‘我’进入已开始状态啦！”。

`onStart()` 方法会非常快速地完成，并且与“已创建”状态一样，`Activity` 不会一直处于“已开始”状态。一旦此回调结束，`Activity` 便会进入“已恢复”状态，系统将调用 `onResume()` 方法。

#### `onResume()`

`Activity` 会在进入“已恢复”状态时来到前台，然后系统调用 `onResume()` 回调。这是应用与用户互动的状态。应用会一直保持这种状态，直到某些事件发生，让焦点远离应用。此类事件包括接到来电、用户导航到另一个 `Activity`，或设备屏幕关闭。

当 `Activity` 进入"已恢复"状态时，与 `Activity` 生命周期相关联的所有生命周期感知型组件都将收到 `ON\_RESUME` 事件(`onResume`事件的常量)，你可以理解为当 `Activity` 进入"已恢复"状态时，会告诉所有生命周期感知型组件：“‘我’进入已恢复状态啦！”。这时，生命周期组件可以启用在组件可见且位于前台时需要运行的任何功能，例如启动相机预览。

当发生中断事件时，`Activity` 进入“已暂停”状态，系统调用 `onPause()` 回调。

如果 `Activity` 从“已暂停”状态返回“已恢复”状态，系统将再次调用 `onResume()` 方法。因此，需要实现 `onResume()`，以初始化在 `onPause()`期间释放的组件，并执行每次 `Activity` 进入“已恢复”状态时必须完成的任何其他初始化操作。

#### `onPause()`

系统将此方法视为用户将要离开应用的 `Activity` 的第一个标志（尽管这并不总是意味着 `Activity` 会被销毁）；此方法表示 `Activity` 不再位于前台（尽管在用户处于多窗口模式时 `Activity` 仍然可见）。使用 `onPause()` 方法暂停或调整当 `Activity`处于“已暂停”状态时不应继续（或应有节制地继续）的操作，以及开发者希望很快恢复的操作。`Activity` 进入此状态的原因有很多。例如：

- 如 `onResume()` 部分所述，某个事件会中断应用执行。这是最常见的情况。
- 在 Android 7.0（API 级别 24）或更高版本中，有多个应用在多窗口模式下运行。无论何时，都只有一个应用（窗口）可以拥有焦点，因此系统会暂停所有其他应用。
- 有新的半透明 `Activity`（例如对话框）处于开启状态。只要 `Activity` 仍然部分可见但并未处于焦点之中，它便会一直暂停。

当 `Activity` 进入已暂停状态时，与 `Activity` 生命周期相关联的所有生命周期感知型组件都将收到 `ON\_PAUSE`事件(`onPause`事件的常量)，你可以理解为当 `Activity` 进入"已暂停"状态时，会告诉所有生命周期感知型组件：“‘我’进入已暂停状态啦！”。这时，生命周期组件可以停止在组件未位于前台时无需运行的任何功能，例如停止相机预览。

`onPause()` 方法的完成并不意味着 `Activity` 离开“已暂停”状态。相反，`Activity` 会保持此状态，直到其恢复或变成对用户完全不可见。如果 `Activity` 恢复，系统将再次调用 `onResume()` 回调。如果 `Activity` 从“已暂停”状态返回“已恢复”状态，系统会让 `Activity` 实例继续驻留在内存中，并会在系统调用 `onResume()` 时重新调用该实例。在这种情况下，无需重新初始化在任何回调方法导致 `Activity` 进入“已恢复”状态期间创建的组件。如果 `Activity` 变为完全不可见，系统会调用 `onStop()`。

#### `onStop()`

如果应用程序的 `Activity` 不再对用户可见，说明其已进入“已停止”状态，因此系统将调用 `onStop()` 回调。例如，当新启动的 `Activity` 覆盖整个屏幕时，可能会发生这种情况。如果 `Activity` 已结束运行并即将终止，系统还可以调用 `onStop()`。

当 `Activity` 进入"已停止"状态时，与 `Activity` 生命周期相关联的所有生命周期感知型组件都将收到 `ON\_STOP` 事件(`onStop`事件的常量)，你可以理解为当 `Activity` 进入"已停止"状态时，会告诉所有生命周期感知型组件：“‘我’进入已停止状态啦！”。这时，生命周期组件可以停止在组件未显示在屏幕上时无需运行的任何功能。

在 `onStop()` 方法中，应用应释放或调整在应用对用户不可见时的无用资源。例如，应用可以暂停动画效果，或从精确位置更新切换到粗略位置更新。使用 `onStop()` 而非 `onPause()` 可确保与界面相关的工作继续进行，即使用户在多窗口模式下查看应用程序的 `Activity` 也能如此。
