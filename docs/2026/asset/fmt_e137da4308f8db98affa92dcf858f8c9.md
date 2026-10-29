
有关更多信息，请访问 Android 开发者博客（[f75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5M7#2)9J5k6h3N6G2L8$3N6D9k6h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6N6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1K9h3&6Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1L8#2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@7d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5M7#2)9J5k6h3N6G2L8$3N6D9k6h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6N6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1K9h3&6Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1L8#2)9J5k6h3S2@1L8h3H3`.)）。

## Android 上的应用程序

### 与操作系统通信

Android 应用程序通过与提供高级 Java API 的抽象层，即 Android 框架层，与系统服务进行交互。这些服务大多数是通过常规的 Java 方法来调用的，并转换为对在后台运行的系统服务的 IPC 调用。系统服务的实例有：

- 连接性（Wi-Fi、蓝牙、NFC 等）
- 文件
- 照相机
- 地理位置（GPS）
- 麦克风

框架层还提供了常见的安全功能，例如加密。

每个新的 Android 版本都会更改 API 规范。关键的漏洞修复和安全补丁通常也适用于早期的版本。在撰写本文时，支持的最旧 Android 版本是 Android 8.1（API 等级 27），当前的 Android 版本是 Android 10（API 等级 29）。

值得注意的 API 版本：

- Android 4.2（API 等级 16），2012 年 11 月发布（引入 SELinux）
- Android 4.3（API 等级 18），2013 年 7 月发布（默认启用 SELinux）
- Android 4.4（API 等级 19），2013 年 10 月发布（引入了几个新的 API 和 ART）
- Android 5.0（API 等级 21），2014 年 11 月发布（默认使用 ART，并添加了许多其他功能）
- Android 6.0（API 等级 23），2015 年 10 月发布（许多新功能和改进，包括在运行时授予详细的权限设置，而不是在安装期间授予全部权限或不提供任何权限）
- Android 7.0（API 等级 24–25），2016 年 8 月发布（ART 上的新 JIT 编译器）
- Android 8.0（API 等级 26–27），2017 年 8 月发布（安全性方面做了很多改进）
- Android 9（API 等级 28），2018 年 8 月发布（限制麦克风或摄像头的后台使用，引入锁定模式，所有应用程序默认使用 HTTPS）
- Android 10（API 等级 29），2019 年 9 月发布（通知气泡、Project Mainline）

### 普通应用程序的 Linux UID / GID

Android 利用 Linux 用户管理来隔离应用程序。但这种方法不同于传统 Linux 环境下多个应用程序常由同一用户运行的用户管理用法。Android 为每个 Android 应用程序创建一个唯一的 UID，并在单独的进程中运行该应用程序。因此，每个应用程序只能访问自己的资源，此保护由 Linux 内核强制执行。

通常，为应用程序分配的 UID 在 10000 和 99999 之间。Android 应用程序会根据其 UID 收到一个用户名。例如，UID 为 10188 的应用程序收到用户名 `u0_a188`。如果授予了应用程序请求的权限，则会将相应的组 ID 添加到该应用程序的进程中。例如，下面的应用程序的用户 ID 是 10188，它属于组 ID 3003（inet），该组与 `android.permission.INTERNET` 权限相关。`id` 命令的输出如下所示：

```text
$ id

uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet),
9997(everybody),50188(all_a188) context=u:r:untrusted_app:s0:c512,c768
```

组 ID 和权限之间的关系在下面的文件中定义：

`frameworks/base/data/etc/platform.xml`（[48eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4#2)9J5k6e0q4Q4x3X3f1I4i4K6g2X3M7U0k6Q4x3V1k6^5M7X3g2X3i4K6u0r3k6Y4u0S2L8h3g2%4L8%4u0C8M7#2)9J5c8X3u0S2M7$3g2Q4x3V1k6V1j5i4c8S2i4K6u0r3k6i4c8U0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3X3g2^5L8h3H3`.](https://bbs.kanxue.com/elink@926K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4#2)9J5k6e0q4Q4x3X3f1I4i4K6g2X3M7U0k6Q4x3V1k6^5M7X3g2X3i4K6u0r3k6Y4u0S2L8h3g2%4L8%4u0C8M7#2)9J5c8X3u0S2M7$3g2Q4x3V1k6V1j5i4c8S2i4K6u0r3k6i4c8U0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3X3g2^5L8h3H3`.)）。

```xml
<permission name="android.permission.INTERNET">
    <group gid="inet" />
</permission>

<permission name="android.permission.READ_LOGS">
    <group gid="log" />
</permission>

<permission name="android.permission.WRITE_MEDIA_STORAGE">
    <group gid="media_rw" />
    <group gid="sdcard_rw" />
</permission>
```

## 应用沙盒

应用程序在 Android 的应用沙盒中执行，应用沙盒可以将应用程序的数据和代码执行与设备上的其他应用程序隔离，这种隔离增加了一个安全层。

新应用程序安装后会创建一个根据该应用程序包命名的新目录，路径为 `/data/data/[package-name]`。此目录保存了应用程序的数据。Linux 目录权限会被设置，以便只能使用应用程序的唯一 UID 来读取和写入目录。

![Sandbox](https://bbs.kanxue.com/upload/attach/202007/802108_CGKA9JQ2SVT643C.png)

我们可以通过查看 `/data/data` 文件夹中的文件系统权限来确认这一点。例如，我们可以看到 Google Chrome 和 Calendar 都分配了一个目录，并在不同的用户帐户下运行：

```text
drwx------  4 u0_a97   u0_a97   4096 2017-01-18 14:27 com.android.calendar
drwx------  6 u0_a120  u0_a120  4096 2017-01-19 12:54 com.android.chrome
```

希望自己的应用程序都使用一个共享沙盒的开发人员可以避开沙盒。当两个应用使用相同的证书进行签名并显式共享相同的用户 ID（在它们的 `AndroidManifest.xml` 文件中包含 `sharedUserId`）时，每个应用都可以访问对方的数据目录。请参阅以下示例，在 NFC 应用中实现此目的：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.nfc"
    android:sharedUserId="android.uid.nfc">
```

### Zygote

Zygote 进程在 Android 初始化期间启动。Zygote 是用于启动应用程序的系统服务。Zygote 进程是一个包含应用程序需要的所有核心库的“基本”进程。启动后，Zygote 打开套接字 `/dev/socket/zygote` 并监听来自本地客户端的连接。接收到连接后，它将派生一个新进程，然后该进程将加载并执行应用专用代码。

### App 生命周期

在 Android 中，应用程序进程的生存期由操作系统控制。当启动应用程序组件并且该应用程序尚未运行任何其他组件时，一个新的 Linux 进程会被创建。当不再需要进程或需要回收内存以运行更重要的应用程序时，Android 可能杀死该进程。进程是否被杀死主要和用户与进程交互的状态有关。通常，进程可能处于四种状态之一：

- 前台进程（例如，在屏幕顶部运行的活动或正在运行的广播接收器）。
- 可见进程是用户意识到的进程，因此终止该进程将对用户体验产生明显的负面影响。一个例子是运行一个用户在屏幕上可见但在前台不可见的活动。
- 服务进程是托管使用 `startService` 方法启动的服务的进程。尽管这些进程对用户来说不是直接看到的，但是它们通常是用户关心的事情（例如，后台网络数据的上传或下载），因此，除非没有足够的内存来保留所有前台和可见进程，系统将始终保持这些进程运行。
- 缓存进程是当前不需要的进程，因此系统可以在需要内存时杀死它。

应用程序必须实现对多个事件做出反应的回调方法。例如，在首次创建应用进程时会调用 `onCreate` 处理程序，其他回调方法包括 `onLowMemory`、`onTrimMemory` 和 `onConfigurationChanged`。

### App Bundles

Android 应用程序能以两种形式发布：

- Android Package Kit（APK）文件
- Android App Bundle（`.aab`）

Android App Bundle 提供了应用程序需要的所有资源，但推迟了 APK 的生成及向 Google Play 的签名。

App Bundles 是经过签名的二进制文件，包含应用程序的代码在几个模块中。基本模块包含应用程序的核心，可以通过各种包含应用程序新功能的模块进行扩展，这在 app bundle 开发者文档（[315K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6S2M7s2m8Q4x3X3c8T1N6h3&6V1L8r3f1`.](https://bbs.kanxue.com/elink@666K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6S2M7s2m8Q4x3X3c8T1N6h3&6V1L8r3f1`.)）中有进一步的说明。

如果您有一个 Android App Bundle，最好使用 Google 的 `bundletool`（[e7aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3j5$3!0E0L8h3q4F1k6q4)9J5k6r3I4A6L8X3g2Q4x3V1k6T1N6h3&6V1L8r3g2@1L8$3!0D9](https://bbs.kanxue.com/elink@b85K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3j5$3!0E0L8h3q4F1k6q4)9J5k6r3I4A6L8X3g2Q4x3V1k6T1N6h3&6V1L8r3g2@1L8$3!0D9)`) 命令行工具来构建未签名的 APK，以便使用 APK 上的现有工具。

你可以通过运行以下命令从 AAB 文件创建 APK：

```bash
$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

如果你想创建已签名的 APK，准备部署到测试设备，请使用：

```bash
$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
--ks=/MyApp/keystore.jks
--ks-pass=file:/MyApp/keystore.pwd
--ks-key-alias=MyKeyAlias
--key-pass=file:/MyApp/key.pwd
```

我们建议你同时测试带有和不带有附加模块的 APK，以便清楚附加模块是否引入和（或）修复了基本模块的安全问题。

### Android Manifest

每个应用程序都有一个 Android Manifest 文件，该文件以二进制 XML 格式嵌入内容。这个文件的标准名称是 `AndroidManifest.xml`，它位于应用程序的 Android Package Kit（APK）文件的根目录中。

Manifest 文件描述了应用程序的结构、组件（活动、服务、内容提供者和 intent 接收器）和请求的权限。它还包含通用的应用程序元数据，比如应用程序的图标、版本号和主题。该文件可能会列出其他信息，比如兼容的 API（最小、目标和最大的 SDK 版本）和可以安装的存储类型（外部或内部）（[516K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6j5%4y4Q4x3V1k6V1j5i4c8S2i4K6u0r3K9h3&6K6N6r3q4D9L8q4)9J5k6r3I4G2j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@bd7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6j5%4y4Q4x3V1k6V1j5i4c8S2i4K6u0r3K9h3&6K6N6r3q4D9L8q4)9J5k6r3I4G2j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8l9`.`.)）。

下面是一个清单文件的示例，包括包名（惯例是反向的 URL，但任何字符串都可以接受）。它还列出了应用程序版本、相关 SDK、需要的权限，给出了内容提供者，使用 intent 过滤器的广播接收器和应用程序及其活动的描述：
