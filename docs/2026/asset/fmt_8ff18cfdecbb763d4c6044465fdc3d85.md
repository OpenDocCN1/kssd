
# [原创]绝不杀后台之变态保活（卸载了APP还能运行）

第一篇：[[原创]利用ROOT权限或Xposed随意对apk进行保资料降级](https://bbs.pediy.com/thread-265277.htm)

第二篇：[[原创]Xposed对手机App进行安装卸载或升级限制](https://bbs.pediy.com/thread-265278.htm)

记得在读高中的时候，那回刚刚安卓系统刚刚火起来，那回手机性能还不够，划开后台列表下面并不是理解后台运行而是写着最近任务四个小字，但是有时候，它又能真的后台运行，去uc浏览器看看网页，回到QQ是不用重新打开的，但很多时候，又需要重新打开，真是个谜。

## 基础：

每个进程对应一个进程`pid`,

在`linux`系统中，在操作系统内核，都会维护一个值`oom_adj`的值，

这个值会根据系统资源情况，用户当前使用情况进行动态调整。

这个值越低，越不容易被系统杀死，如果足够低，在某种程度可做到与系统进程同生共死，即我要是运行不了，就死机给你看。

安卓基于`linux`, 也有这样一个值, 但是也有自己的特色，就是由`ActivityManagerService`（`AMS`）去维护管理这个值。

听起来，`AMS`是活动管理器，但是`AMS`是管理着应用的生杀大权的，我们在手机桌面点击一个图标，如果应用未启动，是会由`AMS`去启动进程的，这样想起来，`AMS`管理`oom_adj`是理所应当的。

怎样查看自己应用的`oom_adj`呢？我们以咸鱼为例

```
ps -A | grep com.taobao.idlefish
cat /proc/pid/oom_adj
```

我们可以观察发现，

当闲鱼APP在前台的时候，看到`oom_adj`为0

当我们退到后台，随便打开某其它应用吧，`oom_adj`一下子掉到6了

我们看下桌面进程`com.miui.home`吧，发现它是2

所以说，系统进程虽然也在后台，但是它优先级还是挺高的。

再看下更重要的`zygote`进程，它的`oom_adj`是-17

假如我们有`root`权限，能不能直接把值写入内核空间呢，答案是可以的。

```
echo -1 > /proc/pid/oom_adj
```

但是，它只能生效那么一会，前面说了，`oom_adj`是由活动大管家动态调整的，你退一下到后台，大管家会重新给你刷新一下，手机电量不够了，内存不够了，大管家都会给我们的应用重新贴标签。你是重要的，你是不太重要的。请拿着这个`oom_adj`，内存不够了，先把高的给斩首示众了，告诉广大应用们，手机各项资源是稀缺的，谁不听话老占内存，老占网络又耗电什么又不做点"产出高的事情"或”不可或缺“就该死。

## 实践

1. 在`AMS`调整`oom_adj`的过程中，拦截它，改它结果。

2. 不断地刷新`oom_adj`的值，防止被覆盖。

3. 研究发现，`oom_adj`存放在`processRecord`实例中。

```
val processRecords = mutableSetOf<Any>()
// 这里我们把闲鱼和加入到保活白名单
val lockPackageList = setOf("com.taobao.idlefish", "com.netease.cloudmusic")
hookAllMethods("com.android.server.am.ActivityManagerService.newProcessRecordLocked") {
after { m ->
try {
changeAppInfoForLockPackage(m.result, -12) // 弄成-12
}catch (e:Throwable){
Log.e(Tag, "newProcessRecordLocked" + e.stackTraceString)
}
}
}

private fun changeAppInfoForLockPackage(processRecord: Any, oomAdjValue: Int) {
if (processRecord.toString().contains("system")) return
Log.d(Tag, "!@@ >>>> $processRecord")
val info = processRecord.get<ApplicationInfo>("info")!!
val processName = processRecord.get<String>("processName")!!
// 注意，只保活主进程就够了，其它子进程的优先级自然会被拉起来
lockPackageList.firstOrNull { info.packageName == it && processName == it }?.let {
setAdj(processRecord, oomAdjValue)
setPersistent(processRecord)
processRecords.add(processRecord)
}
if (!thread.isAlive) thread.start()
}

// 这个设置小了就能逍遥自在，无论怎么吃内存吃电也不会被大管家斩首示众
fun setAdj(processRecord: Any, value: Int) {
processRecord.set<Int>("maxAdj", value)
}

// 设置了这个就能得永生，不小心死了也能被大管家复活
fun setPersistent(processRecord: Any) {
if (Build.VERSION.SDK_INT < 29) {
processRecord.set<Boolean>("persistent", true)
}

if (Build.VERSION.SDK_INT >= 29) {
processRecord.call("setPersistent", true)
}
}
```

## 效果

1. 退到后台打开各种其它app, 尽情消耗内存，或者直接后台点停止运行，目标应用依然纹丝不动，打开后仍是上次的界面。

2. 卸载网易云音乐，歌曲仍在播放，为什么卸载了还能运行，因为卸载前已经把有用的东东放内存了。一般路人甲应用卸载了，大管家会认为你不需要它了，会给它结束掉，但是要是被为重要应用，是不会被杀的。

3. `kill -9` 才能杀死，因为不属于`AMS`大管家管理

除了直接修改`oom_adj`的指，还有方法可以间接修改噢。

就是找一个应用程序开放的服务，在系统进程使用`bindService`把应用绑住，应用也能拿上免死卡。

原理写得不多，网上也有，我只给最关键的中间过程，大家有啥想聊的我再补充上。互相学习。

第一篇：[[原创]利用ROOT权限或Xposed随意对apk进行保资料降级](https://bbs.pediy.com/thread-265277.htm)

第二篇：[[原创]Xposed对手机App进行安装卸载或升级限制](https://bbs.pediy.com/thread-265278.htm)

记得在读高中的时候，那回刚刚安卓系统刚刚火起来，那回手机性能还不够，划开后台列表下面并不是理解后台运行而是写着最近任务四个小字，但是有时候，它又能真的后台运行，去uc浏览器看看网页，回到QQ是不用重新打开的，但很多时候，又需要重新打开，真是个谜。

## 基础：

每个进程对应一个进程`pid`,

在`linux`系统中，在操作系统内核，都会维护一个值`oom_adj`的值，

这个值会根据系统资源情况，用户当前使用情况进行动态调整。

这个值越低，越不容易被系统杀死，如果足够低，在某种程度可做到与系统进程同生共死，即我要是运行不了，就死机给你看。

安卓基于`linux`, 也有这样一个值, 但是也有自己的特色，就是由`ActivityManagerService`（`AMS`）去维护管理这个值。

听起来，`AMS`是活动管理器，但是`AMS`是管理着应用的生杀大权的，我们在手机桌面点击一个图标，如果应用未启动，是会由`AMS`去启动进程的，这样想起来，`AMS`管理`oom_adj`是理所应当的。

怎样查看自己应用的`oom_adj`呢？我们以咸鱼为例

```
ps -A | grep com.taobao.idlefish
cat /proc/pid/oom_adj
```

我们可以观察发现，

当闲鱼APP在前台的时候，看到`oom_adj`为0

当我们退到后台，随便打开某其它应用吧，`oom_adj`一下子掉到6了

我们看下桌面进程`com.miui.home`吧，发现它是2

所以说，系统进程虽然也在后台，但是它优先级还是挺高的。

再看下更重要的`zygote`进程，它的`oom_adj`是-17

假如我们有`root`权限，能不能直接把值写入内核空间呢，答案是可以的。

```
echo -1 > /proc/pid/oom_adj
```

但是，它只能生效那么一会，前面说了，`oom_adj`是由活动大管家动态调整的，你退一下到后台，大管家会重新给你刷新一下，手机电量不够了，内存不够了，大管家都会给我们的应用重新贴标签。你是重要的，你是不太重要的。请拿着这个`oom_adj`，内存不够了，先把高的给斩首示众了，告诉广大应用们，手机各项资源是稀缺的，谁不听话老占内存，老占网络又耗电什么又不做点"产出高的事情"或”不可或缺“就该死。

## 实践

1. 在`AMS`调整`oom_adj`的过程中，拦截它，改它结果。

2. 不断地刷新`oom_adj`的值，防止被覆盖。

3. 研究发现，`oom_adj`存放在`processRecord`实例中。

```
val processRecords = mutableSetOf<Any>()
// 这里我们把闲鱼和加入到保活白名单
val lockPackageList = setOf("com.taobao.idlefish", "com.netease.cloudmusic")
hookAllMethods("com.android.server.am.ActivityManagerService.newProcessRecordLocked") {
after { m ->
try {
changeAppInfoForLockPackage(m.result, -12) // 弄成-12
}catch (e:Throwable){
Log.e(Tag, "newProcessRecordLocked" + e.stackTraceString)
}
}
}

private fun changeAppInfoForLockPackage(processRecord: Any, oomAdjValue: Int) {
if (processRecord.toString().contains("system")) return
Log.d(Tag, "!@@ >>>> $processRecord")
val info = processRecord.get<ApplicationInfo>("info")!!
val processName = processRecord.get<String>("processName")!!
// 注意，只保活主进程就够了，其它子进程的优先级自然会被拉起来
lockPackageList.firstOrNull { info.packageName == it && processName == it }?.let {
setAdj(processRecord, oomAdjValue)
setPersistent(processRecord)
processRecords.add(processRecord)
}
if (!thread.isAlive) thread.start()
}

// 这个设置小了就能逍遥自在，无论怎么吃内存吃电也不会被大管家斩首示众
fun setAdj(processRecord: Any, value: Int) {
processRecord.set<Int>("maxAdj", value)
}

// 设置了这个就能得永生，不小心死了也能被大管家复活
fun setPersistent(processRecord: Any) {
if (Build.VERSION.SDK_INT < 29) {
processRecord.set<Boolean>("persistent", true)
}

if (Build.VERSION.SDK_INT >= 29) {
processRecord.call("setPersistent", true)
}
}
```

## 效果

1. 退到后台打开各种其它app, 尽情消耗内存，或者直接后台点停止运行，目标应用依然纹丝不动，打开后仍是上次的界面。

2. 卸载网易云音乐，歌曲仍在播放，为什么卸载了还能运行，因为卸载前已经把有用的东东放内存了。一般路人甲应用卸载了，大管家会认为你不需要它了，会给它结束掉，但是要是被为重要应用，是不会被杀的。

3. `kill -9` 才能杀死，因为不属于`AMS`大管家管理

除了直接修改`oom_adj`的指，还有方法可以间接修改噢。

就是找一个应用程序开放的服务，在系统进程使用`bindService`把应用绑住，应用也能拿上免死卡。

原理写得不多，网上也有，我只给最关键的中间过程，大家有啥想聊的我再补充上。互相学习。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
