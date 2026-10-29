
因为 fragment 有它们自己的生命周期，所以 `Fragment` 类中包含的事件管理器可以重定义和重写。这些事件管理器包括：`onAttach`、`onCreate`、`onStart`、`onDestroy`和`onDetach`。还有其他的，读者可以查看[Android Fragment 详解](https://bbs.kanxue.com/elink@dedK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3q4H3M7q4)9J5c8V1k6J5j5h3N6E0k6h3&6@1i4K6u0W2K9s2c8E0L8l9`.`.)进行了解。

Fragment 可以简单地通过继承 Android 提供的 `Fragment` 类来实现：

```
public class myFragment extends Fragment {

    ...

}
```

Fragment 不需要在 manifest 文件中声明，因为它依赖于 activity。

activity 可以使用 Fragment 管理器（`FragmentManager` 类）管理自身的 fragment。有了这个类，activity 可以很容易地找到，添加，移除和替换相关 fragment。

fragment 管理器可以使用如下方法创建：

```
FragmentManager fm = getFragmentManager();
```

fragment 不需要有用户接口。使用它们可以很方便有效的管理与用户交互有关的后台操作。fragment 可以永久声明，所以操作系统可以保留它的状态，即使它的 activity 已经被销毁了。

### 进程间通信 IPC

正如前面所说，每一个 Android 进程都有其自身的沙箱地址空间。IPC 设备确保 app 可以安全的交换信号和数据。Android 的 IPC 是基于 Binder 的（它是在 `OpenBinder` 上的定制实现），并非依赖于默认的 Linux IPC 设备。大多数 Android 系统服务和高级的 IPC 服务都依赖于于 Binder。

Binder 一词代表很多含义，包括：
- Binder 驱动：内核驱动
- Binder 协议：用于与 binder 驱动通信的低级的基于 ioctl 的协议
- `IBinder` 接口：一个定义好的，用 Binder 对象实现的行为
- Binder 对象：对 `IBinder` 接口的实现
- Binder 服务：对 Binder 对象的实现；例如，位置服务，和传感器服务
- Binder 客户：使用到 Binder 服务的对象

Binder 框架包含了一个客户端-服务器通信模型。要使用 IPC，app 需在代理对象中调用 IPC 函数。代理对象透明的将参数打包并向 Binder 服务器发送一个 transaction，Binder 服务器以字符驱动的形式实现（`/dev/binder`）。服务器拥有一个进程池用来处理接收到的请求和发送信息到目标对象。从客户端 app 来看，一切都好像只是一次常规的函数调用——所有的活都已经由 Binder 框架做完了。

![](https://bbs.kanxue.com/upload/attach/201807/743775_XCVQ67EZTXMHEDG.png)

Binder 概述。图片来源：*[Android Binder by Thorsten Schreiber](https://bbs.kanxue.com/elink@6ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1k6s2y4Q4x3X3g2J5N6h3u0Q4x3X3g2V1k6g2)9J5c8X3#2W2k6r3W2S2i4K6u0r3j5i4c8@1j5h3y4Z5L8h3g2F1N6s2y4Q4x3V1k6X3K9h3I4W2M7#2)9J5c8U0t1H3x3e0q4Q4x3V1j5I4x3q4)9J5c8X3#2S2K9h3&6Q4x3X3g2H3k6r3j5`.)*

允许其他应用程序绑定它们的服务称为 bind service。这些服务需给客户端提供一个 `IBinder` 接口。开发者用 AIDL（Android Interface Descriptor Language，Android 接口定义语言）来为远程服务定义接口。

`Servicemanger` 是一个用来管理系统服务注册和查找的系统守护进程。它包含了所有已注册服务的 name/Binder 对。用 `android.os.ServiceManager` 中的 `addService` 添加服务，以 name 为参数，用 `getService` 检索服务：

```
public static IBinder getService(String name)
```

你可以用 `service list` 命令查看系统服务：

```
$ adb shell service list
Found 99 services:
0 carrier_config: [com.android.internal.telephony.ICarrierConfigLoader]
1 phone: [com.android.internal.telephony.ITelephony]
2 isms: [com.android.internal.telephony.ISms]
3 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
```

### Intent

`Intent` 信息是在 Binder 之上建立的一种异步通信框架。这个框架支持点对点通信和发布-订阅模式的通信。一个 `Intent` 就是一个信息对象，可用于请求来自另一个 app 组件的某一个动作。`intent` 可以以多种方式促进 IPC，但最主要的方式是如下三种：

- 启动 activity
  - 一个 activity 代表了 app 中的一个屏幕。你可以通过发送一个 `startActivity` intent 启动一个 activity。这个 intent 需包括该 activity 和需要的数据。

- 启动服务
  - 服务是在后台完成操作的一个组件，没有用户接口。在 Android 5.0（API 21）及以后，你可以用 `JobScheduler` 启动一个服务。

- 发送广播
  - 广播就是任何 app 都可以接收的信息。系统为系统事件发送广播，包括系统启动和要求初始化。你可以用 `sendBroadcast` 和 `sendOrderBroadcast` 给其他 app 发送广播。

`Intent` 是用于在 app 间和组件间发送信息的组件。app 可以用它们来给自身的组件发送信息（例如，在 app 内启动一个 activity），也可以给其他 app，操作系统发送信息。`Intent` 可用于启动 activity 和 service，执行给定数据的操作，给整个系统广播信息。

有两种类型的 `Intent`。显式 intent ，显示地给出了要启动的组件的名字（完整的类名）。例如：

```
Intent intent = new Intent(this, myActivity.myClass);
```

隐式 `Intent` ，是发送给操作系统，让它根据给定的数据（"[a20K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3@1`.](https://bbs.kanxue.com/elink@e0cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.)" ，在下面的例子中）执行给定操作。由系统来决定哪个 app 或类来执行相应的服务。例如：

```
Intent intent = new Intent(Intent.MY_ACTION, Uri.parse("d8bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3@1`.")); 
```

`Intent filer` 意图过滤器，是在 manifest 中的一个表达式，用来指定组件想要接收的 intent 的类型。例如，通过在 activity 中声明一个 intent filter，其他 app 就有可能使用特定类型的 intent 来启动你的 activity。否则，你的 activity 就只能由一个显式的 intent 来启动了。

Android 用 intent 给 app 广播信息（例如来电或来信），重要的电池信息（电量不足，比方说），还有网络状态改变（信号不足，例如）。也可以添加 extra 数据（通过 `putExtra/getExtra`）

这里是几个由操作系统发送的 intent 列表。所有的常量都已经在 `Intent` 类中定义，完整的列表可以查看 Android 官方文档：

- `ACTION_CAMERA_BUTTON`
- `ACTION_MEDIA_EJECT`
- `ACTION_NEW_OUTGOING_CALL`
- `ACTION_TIMEZONE_CHANGED`

为提高安全性和隐私，使用本地广播管理器在 app 内发送和接收 intent，而不会将它们发送到其他地方。这对于确保敏感和隐私信息（例如，地理位置信息）不流到 app 之外很有用。

### 广播接收器

广播接收器是让 app 接收来自其他 app 或系统的信息的组件。有了它，app 就可以对各种事件（内部的，被其他 app 初始化，或被操作系统初始化）做出反应。最普遍的用法是更新用户接口，启动服务，更新内容，和创建用户通知。

广播接收器需要在 app 的 manifest 文件中声明。manifest 要明确指定广播接收器和 intent filter 之间的关联，以便指明接收器要监听的动作。如果没有声明广播接收器，app 不会监听广播信息。但是，也不用运行 app 去接收 intent，当有相关的 intent 的时候，系统会自动启动该 app.

下面是一个在 manifest 中声明广播接收器的例子：

```
<receiver android:name=".myReceiver" >
        <intent-filter>
            <action android:name="com.owasp.myapplication.MY_ACTION" />
        </intent-filter>
    </receiver>
```

在接收到一个隐式 intent 后，Android 会列出所有在它们的过滤器中注册了给定 action 的 app。若有多于一个的 app 注册了相同的 action，系统就会让用户从可选列表中选择一个。

广播接收器的一个比较好玩的特点是，它们有优先级。通过这种方式，一个 intent 可以根据优先级发送给所有满足条件的接收者。

本地广播接收器可以用于确保 intent 仅在 app 内部被收到，任何来自其他 app 的 intent 都会被丢弃。这对于提高安全性很有用。

### 内容提供器

Android 用 SQLite 永久存储数据：和 Linux 一样，将数据存储在文件中。SQLite 是一款轻量的，高效的，开源的关系型数据存储技术，不需要过多的处理能力，这使得它很适合移动应用。有操作 SQLite 的 API，里面有很多专门用于处理此类问题的类（`Cursor`, `ContentValues`, `SQLiteOpenHelper`, `ContentProvider`, `ContentResolver`, 等等）。SQLite 并不以独立的进程存在，它是 app 的一部分。属于给定 app 的数据库默认只能在这个 app 内访问。但是，内容提供器为提取数据资源（包括数据库和 flat file）提供了一种很好的机制，它们也为 app 间，包括原生 app 间，的数据共享提供了一种标准的、高效的机制。要访问其他 app，内容提供器需要在 manifest 文件中显式声明该 app 可以共享它。如若不声明内容提供器，外界将无法访问它们，只能被创建它们的 app 访问。

内容提供器通过 URI 寻址策略来实现：它们都使用 `content://` 模式。不管资源的类型是什么（SQLite 数据库，flat file 等等），寻址策略都一样，因此提取资源并提供给开发者一个唯一的方案。内容提供器有所有常规的数据库操作：增、删、改、查。这就意味着，在 manifest 文件声明的有合法权限的任意 app 都可以从外面操纵这些数据。

### 服务

服务是在后台执行任务（数据处理，启动 intent，通知，等等），并且没有用户界面的 Android OS 组件（基于 `Service` 类）。服务意味着长期执行进程。它们的系统优先级低于所有的活跃 app，高于所有的不活跃 app。因此，在系统需要资源时，它们不太可能会被杀掉，而且资源再次足够的适合，它们会自动重启。Activity 是在主 app 进程中执行的。它们是执行异步任务的最佳候选人。

### 权限

因为 Android app 是安装在沙箱中的，因此初始状态下无法访问用户信息和系统组件（例如，相机和麦克风），Android 为系统提供了一个为特定任务预定义的一系列权限，app 在需要的时候可以申请。例如，如果你希望你的 app 能够使用手机相机，你需要申请 `android.permission.CAMERA` 权限。在 Marshmallow（API 23）之前，所有的权限在安装时全部授予。自 Marshmallow 以后，用户需要在 app 运行时同意 app 申请的一些权限。

#### 保护级别

Android 权限根据它们的保护级别进行排序，并分为四类：

- *Normal*: 较低的保护水平。让 app 以最小的风险访问应用程序层的特征，不会给系统、用户或其他系统造成危害。在 app 安装时授予，并且是默认的保护级别。例如：`android.permission.INTERNET`

- *Dangerous*: 这个级别的权限权限让 app 可以执行有可能有损用户隐私的操作，或者对用户设备的正常操作。这个级别的权限不会在安装的时候授予，用户需决定 app 能否拥有该权限。例如：`android.permission.RECORD_AUDIO`

- *Signature*： 这个级别的权限只在申请这个权限的 app 和声明了这个权限的 app 有相同的数字签名的时候才会授予。如果数字签名匹配，将自动授权。例如：`android.permission.ACCESS_MOCK_LOCATION`

- *SystemOrSignature*: 这个级别的权限只有在该 app 已经内嵌到系统镜像或者和声明了该权限的 app 有相同的数字签名的时候才会授予。例如：`android.permission.ACCESS_DOWNLOAD_MANAGER`

#### 申请权限

app 可以通过在 manifest 文件中使用 `<uses-permission/>` 申请 Normal 、Dangerous、Signature 级别的权限。下面的例子是在 `AndroidManifest.xml` 中申请读取 SMS 信息的权限：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    <application>...</application>
</manifest>
```

#### 声明权限

app 可以向系统内已安装的其他 app 暴露自己的特征和内容。为限制对自身组件的访问，它可以使用 Android 官方定义的任意一个权限，或者自己定义一个。元素中定义了一个新的权限。下面的例子是如何在 app 中声明一个权限：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.permissions.sample" ...>

    <permission
    android:name="com.permissions.sample.ACCESS_USER_INFO"
    android:protectionLevel="signature" />
    <application>...</application>
</manifest>
```

上面的代码中定义了一个新的权限，叫做：`com.permissions.sample.ACCESS_USER_INFO`, 保护级别是 `Signature`。任何由这个权限保护的组件都只能被与本 app 一样的开发者证书签名的 app 所访问。

#### 为组件指定权限

也可以用权限保护 Android 组件。活动、服务、内容提供器、广播接收器——所有这些都可以用权限机制保护它们的接口。通过在 `AndroidManifest.xml` 文件中添加 `android:permission` 志相应的组件标签，可以为活动，服务，和广播接收器增加权限。

```
<receiver
    android:name="com.permissions.sample.AnalyticsReceiver"
    android:enabled="true"
    android:permission="com.permissions.sample.ACCESS_USER_INFO">
    ...
</receiver>
```

内容提供器有一点不同。Android 提供了一些通过 URI 来读、写和访问内容提供器的权限。

- `android.writePermission`，`android.readPermission`：开发者可以分别授予读权限和写权限。
- `android:permission`: 控制对内容提供器的读写的基本权限
- `android.grantUriPermissoin` : 如果内容提供器可以访问给定的 URI 就为 true （这个访问暂时绕过了其他权限的限制），否则为 false。

### 签名与发布过程
