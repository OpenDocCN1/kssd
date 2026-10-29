
# 玩转Android10源码开发定制(十)增加获取当前运行最顶层的Activity命令

## 一、获取当前运行的顶层 Activity 的几种方式

在 Android 开发中，由于某些需求常常需要获取当前顶层的 Activity 信息。比如 App 中获取顶层 Activity 界面信息来判断某一个 app 是否在前台运行、统计某一个 app 的使用时长、更有恶意程序通过监听界面伪造 app 进行盗号以及欺诈、自动化开发中通过顶层 Activity 进行页面元素定位点击(比如基于辅助功自动化、uiautomator 自动化)等等操作。 在逆向工程中，获取当前运行 app 运行顶层 activity 也比较常用。通过顶层 Activity 可以快速定位界面中的功能在哪一个页面。

在 Android app 开发中，获取顶层 Activity 有以下方式:

1. 低于 Android 5.0,情况下使用 `getRunningTasks`

首先需要在 Androidmanifest 中添加权限

```
<uses-permission android:name = "android.permission.GET_TASKS"/>
```

代码如下：

```
ActivityManager mAm = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
String activity_name = mAm.getRunningTasks(1).get(0).topActivity.getClassName();
```

2. 高于安卓5.0使用`UsageStatsManager`获取到应用程序的运行情况。

在AndroidManifest文件中添加权限:

```
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
```

启动授权页面，需要用户授权app获取应用使用情况统计权限。跳转授权页面参考:

```
Intent intent = new Intent(Settings.ACTION_USAGE_ACCESS_SETTINGS);
context.startActivity(intent);
```

获取顶层 activity 参考代码:

```
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
UsageEvents usageEvents = sUsageStatsManager.queryEvents(beginTime, endTime);
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

除了 app 中使用代码获取顶层 Activity，也可以通过 adb 命令获取当前顶层的 Activity 信息，比如 windows 系统下 Android 10 手机可以用如下 adb 命令:

```
C:\Users\Qiang>adb shell dumpsys activity |findstr "mResume"
mResumedActivity: ActivityRecord{12f93fd u0 com.sohu.inputmethod.sogou/.SogouIMEHomeActivity t50}
```

由于 `ActivityManager->getRunningTasks` 获取顶层 Activity 比较准确一些，以下将采用 `ActivityManager->getRunningTasks` 方式来进行研究实现。

## 二、源码追踪 ActivityManager getRunningTasks 调用

`ActivityManager`源码路径位于:

> frameworks/base/core/java/android/app/ActivityManager.java

`ActivityManager` 中 `getRunningTask` 调用如下:

```
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

`ActivityTaskManager` 源码路径:

> frameworks/base/core/java/android/app/ActivityTaskManager.java

`ActivityTaskManager` 中 `getService` 实现如下:

```
//getService 实现代码
public static IActivityTaskManager getService() {
return IActivityTaskManagerSingleton.get();
}
//IActivityTaskManagerSingleton 初始化代码
private static final Singleton<IActivityTaskManager> IActivityTaskManagerSingleton =new Singleton<IActivityTaskManager>() {
@Override
protected IActivityTaskManager create() {
final IBinder b = ServiceManager.getService(Context.ACTIVITY_TASK_SERVICE);
return IActivityTaskManager.Stub.asInterface(b);
}
};
```

`ActivityTaskManager.getService().getTasks` 调用最终变成了 `IActivityTaskManager.getTasks`。在安卓源码中没有找到 `IActivityTaskManager.java` 文件，只找到 `IActivityTaskManager.aidl` 文件，说明 `IActivityTaskManager.getTasks` 调用最终转化了 binder 的进程间调用。安卓中 aidl 是用于 binder 通信定义服务器和客户端通信接口的一种描述语言，源码编译的时候会将 aidl 文件转化为 java 文件。

`IActivityTaskManager.aidl` 文件路径如下:

>frameworks/base/core/java/android/app/IActivityTaskManager.aidl

在源码中搜索关键字“extends IActivityTaskManager"找到 `ActivityTaskManagerService` 文件使用了。那么说明 `ActivityTaskManagerService` 最终实现了 `getTasks` 的函数功能。以下追踪 `ActivityTaskManagerService` 中的调用情况:

`ActivityTaskManagerService` 代码路径:

> frameworks/base/services/core/java/com/android/server/wm/ActivityTaskManagerService.java

`ActivityTaskManagerService` 中 `getTasks` 代码如下:

```
//getTasks 实现代码，调用了 getFilteredTasks
@Override
public List<ActivityManager.RunningTaskInfo> getTasks(int maxNum) {
return getFilteredTasks(maxNum, ACTIVITY_TYPE_UNDEFINED, WINDOWING_MODE_UNDEFINED);
}
// getFilteredTasks 实现代码
@Override
public List<ActivityManager.RunningTaskInfo> getFilteredTasks(int maxNum,@WindowConfiguration.ActivityType int ignoreActivityType,@WindowConfiguration.WindowingMode int ignoreWindowingMode) {
final int callingUid = Binder.getCallingUid();
final int callingPid = Binder.getCallingPid();
final boolean crossUser = isCrossUserAllowed(callingPid, callingUid);
final int[] profileIds = getUserManager().getProfileIds(UserHandle.getUserId(callingUid), true);
ArraySet<Integer> callingProfileIds = new ArraySet<>();
for (int i = 0; i < profileIds.length; i++) {
callingProfileIds.add(profileIds[i]);
}
ArrayList<ActivityManager.RunningTaskInfo> list = new ArrayList<>();
synchronized (mGlobalLock) {
if (DEBUG_ALL) Slog.v(TAG, "getTasks: max=" + maxNum);
final boolean allowed = isGetTasksAllowed("getTasks", callingPid, callingUid);
mRootActivityContainer.getRunningTasks(maxNum, list, ignoreActivityType,
ignoreWindowingMode, callingUid, allowed, crossUser, callingProfileIds);
}
return list;
}
```

通过以上分析,已经知道系统如何实现 `getRunningTask` 函数功能的流程。借鉴 `getRunningTask` 实现，要增加一个获取顶层 Activity 的接口，只需要在 `IActivityTaskManager.aidl` 中增加一个接口，仿照 `ActivityTaskManagerService` 中 `getTasks` 的实现方式即可。

`ActivityManager getRunningTasks`大概的一个调用流程总结如下,图中省略了许多binder交互的细节:

![](<mark class="encrypted">8d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6C8M7U0u0Q4x3X3g2U0L8W2)9J5k6r3u0B7i4K6u0W2N6h3k6A6L8r3g2G2M7#2)9J5k6h3y4G2L8g2)9J5c8U0q4S2x3e0R3&6j5K6g2T1i4K6u0V1x3K6R3#2y4W2)9J5k6o6b7#2k6o6c8Q4x3X3b7&6x3$3u0U0i4K6u0V1x3K6f1%4x3U0N6T1j5e0N6W2k6U0y4V1i4K6u0W2M7r3&6Y4i4K6y4r3g2f1y4D9L8%4g2V1f1s2g2T1L8r3W2U0d9$3g2&6i4K6y4p5g2p5!0w2c8f1&6Q4y4h3j5^5k6o6S2T1y4K6u0T1k6g2)9J5k6o6f1%4z5h3q4Q4x3X3b7@1k6e0R3K6i4K6u0V1j5X3k6V1x3q4)9J5k6o6g2X3y4X3y4W2x3e0f1@1y4X3j5I4x3#2)9J5y4W2y4A6k6$3&6S2N6s2g2J5k6g2)9K6c8o6l9I4c8Y4c8$3h3r3S2n7P5h3N6A6x3f1!0U0d9X3N6a6g2X3b7^5N6i4m8K9L8o6p5H3k6#2)9J5y4e0t1#2x3@1c8Q4x3U0k6q4P5s2m8A6M7X3g2K6i4K6y4p5x3e0j5H3z5e0R3&6z5o6b7&6y4g2)9J5z5b7`.`.</mark>

## 三、为系统增加 getTopActivity 接口

1. 在`IActivityTaskManager.aidl`文件中添加`getTopActivity`。添加内容如下:

```
///ADD START
String getTopActivity();
///ADD END
```

2. 在`ActivityTaskManagerService`中重写`getTopActivity`方法，完成真正的功能，由于我们提供的接口是adb shell调用，所以此处没有处理权限检测，默认情况下adb shell有权限访问的。代码如下:

```
///ADD START
@Override
public String getTopActivity()
{
List <ActivityManager.RunningTaskInfo> appTasks = getTasks(1);
if (null != appTasks && !appTasks.isEmpty()) {
return appTasks.get(0).topActivity.getPackageName()+"/"+appTasks.get(0).topActivity.getClassName();
}
return "";
}
///ADD END
```

以上工作做好之后，我们就可以通过如下代码调用获取顶层 activity 了。代码如下：

```
String topString=ActivityTaskManager.getService().getTopActivity();
```

## 四、系统增加 gettopactivity 命令

1. 创建目录`frameworks/base/cmds/gettopactivity`,用来存放命令源码

```
> qiang@ubuntu:~/lineageOs$ mkdir -p frameworks/base/cmds/gettopactivity
```

2. 在`gettopactivity`目录创建调用新增`getTopActivity`方法的源码文件`src/com/android/commands/atm/Atm.java`，在`Atm.java`中添加如下代码:
