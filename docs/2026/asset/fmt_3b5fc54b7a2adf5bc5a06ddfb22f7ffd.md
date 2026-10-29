
```java
package com.android.commands.atm;
import android.app.ActivityTaskManager;
import android.app.IActivityTaskManager;
import android.os.RemoteException;
import android.os.ResultReceiver;
import android.os.SELinux;
import android.os.ServiceManager;
import android.util.AndroidException;
import java.io.File;
import java.io.FileDescriptor;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.PrintStream;
public class Atm {
private IActivityTaskManager mAtm;
public static void main(String[] args) {
(new Atm()).run();
}
public void run() throws Exception {
//获取ActivityTaskManager对象
mAtm = ActivityTaskManager.getService();
if (mAtm == null) {
System.err.println(NO\_SYSTEM\_ERROR\_CODE);
throw new AndroidException("Can't connect to activity task manager; is the system running?");
}
try {
//调用getTopActivity方法，获取顶层运行activity信息
String topString=mAtm.getTopActivity();
System.out.println("TopActivity:"+topString);
} catch (RemoteException e) {
System.err.println(NO\_SYSTEM\_ERROR\_CODE);
throw new AndroidException("Can't call activity task manager; is the system running?");
}
}
}
```

3. 在`gettopactivity`目录下增加`gettopactivity` shell脚本，作为`adb shell`调用的命令。脚本内容如下:

```sh
#!/system/bin/sh
base=/system
export CLASSPATH=$base/framework/gettopactivity.jar
exec app\_process $base/bin com.android.commands.atm.Atm "$@"
```

4. 在`gettopactivity`目录下添加预编译可执行程序`Android.mk`脚本，内容如下:

```makefile
#///ADD START
#///ADD END
LOCAL\_PATH:= $(call my-dir)
include $(CLEAR\_VARS)
LOCAL\_SRC\_FILES := \
$(call all-java-files-under, src)
LOCAL\_MODULE := gettopactivity
include $(BUILD\_JAVA\_LIBRARY)
include $(CLEAR\_VARS)
LOCAL\_MODULE := gettopactivity
LOCAL\_SRC\_FILES := gettopactivity
LOCAL\_MODULE\_CLASS := EXECUTABLES
LOCAL\_MODULE\_TAGS := optional
include $(BUILD\_PREBUILT)
```

5. 将`gettopactivity`模块加入到编译文件链接中。在之前的文章中已经讨论过了预编译二进制模块加入编译链放入文件路径:`make/target/product/base_system.mk`中。加入之后的部分内容如下:

```makefile
#///ADD START
# add frida server to system
#///ADD END
# Base modules and settings for the system partition.
PRODUCT\_PACKAGES += \
myfridaserverarm64 \
gettopactivity \
...(省略)
```

## 五、编译系统测试效果

源码根目录执行编译：

```sh
source build/envsetup.sh
breakfast oneplus3
brunch oneplus3
```

刷机测试效果:

```text
C:\Users\Qiang>adb shell gettopactivity
TopActivity:com.starbucks.cn/com.starbucks.cn.ui.StarbucksLaunchActivity
```

## 一、获取当前运行的顶层 Activity 的几种方式

在 Android 开发中，由于某些需求常常需要获取当前顶层的 Activity 信息。比如 App 中获取顶层 Activity 界面信息来判断某一个 app 是否在前台运行、统计某一个 app 的使用时长、更有恶意程序通过监听界面伪造 app 进行盗号以及欺诈、自动化开发中通过顶层 Activity 进行页面元素定位点击(比如基于辅助功自动化、uiautomator 自动化)等等操作。 在逆向工程中，获取当前运行 app 运行顶层 activity 也比较常用。通过顶层 Activity 可以快速定位界面中的功能在哪一个页面。

在 Android app 开发中，获取顶层 Activity 有以下方式:

1. 低于 Android 5.0,情况下使用`getRunningTasks`

首先需要在`Androidmanifest`中添加权限

```xml
<uses-permission android:name="android.permission.GET_TASKS" />
```

代码如下：

```java
ActivityManager mAm = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
String activity_name = mAm.getRunningTasks(1).get(0).topActivity.getClassName();
```

2. 高于安卓5.0使用`UsageStatsManager`获取到应用程序的运行情况。

在`AndroidManifest`文件中添加权限:

```xml
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
```

启动授权页面，需要用户授权app获取应用使用情况统计权限。跳转授权页面参考:

```java
Intent intent = new Intent(Settings.ACTION_USAGE_ACCESS_SETTINGS);
context.startActivity(intent);
```

获取顶层 activity 参考代码:

```java
public static getTopActivity()
{
long endTime = System.currentTimeMillis();
long beginTime = endTime - 10000;
UsageStatsManager sUsageStatsManager =null;
if (sUsageStatsManager == null) {
sUsageStatsManager = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
}
String result = "";
UsageEvents.Event event = new UsageEvents.Event();
UsageEvents usageEvents = sUsageStatsManager.queryEvents(beginT ime, endTime);
while (usageEvents.hasNextEvent()) {
usageEvents.getNextEvent(event);
if (event.getEventType() == UsageEvents.Event.MOVE_TO_FOREGROUND) {
result = event.getPackageName()+"/"+event.getClassName();
}
}
if (!android.text.TextUtils.isEmpty(result)) {
return result;
}
}
return "";
}
```

除了 app 中使用代码获取顶层 Activity，也可以通过 `adb` 命令获取当前顶层的 Activity 信息，比如 windows 系统下 Android 10 手机可以用如下 `adb` 命令:

```text
C:\Users\Qiang>adb shell dumpsys activity |findstr "mResume"
mResumedActivity: ActivityRecord{12f93fd u0 com.sohu.inputmethod.sogou/.SogouIMEHomeActivity t50}
```

由于 `ActivityManager->getRunningTasks` 获取顶层 Activity 比较准确一些，以下将采用 `ActivityManager->getRunningTasks` 方式来进行研究实现。

## 二、源码追踪 ActivityManager getRunningTasks 调用

`ActivityManager`源码路径位于: `frameworks/base/core/java/android/app/ActivityManager.java`

`ActivityManager` 中 `getRunningTask` 调用如下:

```java
// getRunningTasks 方法实现
// 最终调用 getTaskService().getTasks
public List<RunningTaskInfo> getRunningTasks(int maxNum) throws SecurityException {
try {
return getTaskService().getTasks(maxNum);
} catch (RemoteException e) {
throw e.rethrowFromSystemServer();
}
}
# getTaskService 方法实现
private static IActivityTaskManager getTaskService() {
return ActivityTaskManager.getService();
}
```

由以上代码分析可知，最终调用的是 `ActivityTaskManager.getService().getTasks`。接下来分析 `ActivityTaskManager` 中的调用情况。

`ActivityTaskManager` 源码路径: `frameworks/base/core/java/android/app/ActivityTaskManager.java`

`ActivityTaskManager` 中 `getService` 实现如下:

```java
//getService 实现代码
public static IActivityTaskManager getService() {
return IActivityTaskManagerSingleton.get();
}
//IActivityTaskManagerSingleton 初始化代码
private static final Singleton<IActivityTaskManager> IActivityTaskManagerSingleton = new Singleton<IActivityTaskManager>() {
@Override
protected IActivityTaskManager create() {
final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
return IActivityTaskManager.Stub.asInterface(b);
}
};
```

`ActivityTaskManager.getService().getTasks` 调用最终变成了 `IActivityTaskManager.getTasks`。在安卓源码中没有找到 `IActivityTaskManager.java` 文件，只找到 `IActivityTaskManager.aidl` 文件，说明 `IActivityTaskManager.getTasks` 调用最终转化了 `binder` 的进程间调用。安卓中 `aidl` 是用于 `binder` 通信定义服务器和客户端通信接口的一种描述语言，源码编译的时候会将 `aidl` 文件转化为 `java` 文件。

`IActivityTaskManager.aidl` 文件路径如下: `frameworks/base/core/java/android/app/IActivityTaskManager.aidl`

在源码中搜索关键字 `extends IActivityTaskManager` 找到 `ActivityTaskManagerService` 文件使用了。那么说明 `ActivityTaskManagerService` 最终实现了 `getTasks` 的函数功能。以下追踪 `ActivityTaskManagerService` 中的调用情况:

`ActivityTaskManagerService` 代码路径: `frameworks/base/services/core/java/com/android/server/wm/ActivityTaskManagerService.java`

`ActivityTaskManagerService` 中 `getTasks` 代码如下:
