
这可能会涉及到一些安全问题，比如多线程之间的条件竞争，在修改`dataPostion`之后被其他线程使用，当然这通过`synchronized`同步块防御住了。另一个问题是，在`LazyValue`未使用之前，其所对应的`Parcel`是不能被释放的，否则就会出现类似 UAF 的内存问题。考虑到对于`Parcel`的内存分配策略而言，大多数是使用手工管理的`obtain()`/`recycle()`方式，这个问题是有可能存在的。

## `Parcel` 内存管理

由于`Parcel`本身是为了频繁的 IPC 传输而设计的，因此对于其分配和释放通常使用手工管理的方式，以避免 Java 堆分配或者 GC 带来的性能损耗。在阅读系统源码或者 AIDL 生成的模版代码时都能发现，`Parcel`使用`obtain()`进行分配，使用`recycle()`进行释放。

分配的代码实现如下:

```java
public static Parcel obtain() {
Parcel res = null;
synchronized (sPoolSync) {
if (sOwnedPool != null) {
res = sOwnedPool;
sOwnedPool = res.mPoolNext;
res.mPoolNext = null;
sOwnedPoolSize--;
}
}
if (res == null) {
res = new Parcel(0);
} else {
res.mRecycled = false;
res.mReadWriteHelper = ReadWriteHelper.DEFAULT;
}
return res;
}
```

其中`sOwnedPool`是静态属性，`mPoolNext`是成员属性，二者都是`Parcel`类型。因此内存池中的`Parcel`可以看做是一个表头为`sOwnedPool`的单链表结构。`obtain()`本质上是从链表中取出表头的数据。

对于 IPC 接收到的`Parcel`数据分配方式略有不同，因为这些`Parcel`在 C++ 层由系统创建，因此使用不同的链表，表头为`sHolderPool`静态属性。这些`Parcel`通过构造函数`Parcel(long nativePtr)`去构建，生命周期由系统管理(`mOwnsNativeParcelObject`为`false`)，因此需要区分开来。这类`Parcel`的分配通过重载的`obtain(long)`方法去创建，与上述实现大同小异。

在释放过程会将这两种情况区分开来:

```java
public final void recycle() {
if (mRecycled) {
Log.wtf(TAG, "Recycle called on unowned Parcel. (recycle twice?) Here: "
+ Log.getStackTraceString(new Throwable())
+ " Original recycle call (if DEBUG\_RECYCLE): ", mStack);
return;
}
mRecycled = true;
// We try to reset the entire object here, but in order to be
// able to print a stack when a Parcel is recycled twice, that
// is cleared in obtain instead.
mClassCookies = null;
freeBuffer();
if (mOwnsNativeParcelObject) {
synchronized (sPoolSync) {
if (sOwnedPoolSize < POOL\_SIZE) {
mPoolNext = sOwnedPool;
sOwnedPool = this;
sOwnedPoolSize++;
}
}
} else {
mNativePtr = 0;
synchronized (sPoolSync) {
if (sHolderPoolSize < POOL\_SIZE) {
mPoolNext = sHolderPool;
sHolderPool = this;
sHolderPoolSize++;
}
}
}
}
```

释放操作相当于单链表的插入，即将释放的`Parcel`放入`sOwnPool`/`sHolderPool`链表的头部。从上述代码可以看出，`Parcel`的分配和释放过程是后进先出(LIFO)的，即`Parcel.obtain()`会分配出最近一次释放的对象。

## `Parcel` UAF

通过`LazyValue`的实现以及`Parcel`内存管理的策略，我们似乎可以找到一个攻击场景: 在`Parcel`读取`LazyValue`之后，将`Parcel`进行释放，而后再读取对应的`LazyValue`，此时如果`Parcel`被分配并填充了敏感数据，那么我们的`LazyValue`就可以读取出这些敏感内容造成数据泄露。如果泄露的数据来自其他进程，且数据中包含特权的`IBinder`等结构，那么还可能造成提权或者 RCE 的危害！

为此，我们首先需要找到一个`recycle()`后再次使用`LazyValue`的场景。接下来，就是漏洞上场的时间了，出现上述问题的漏洞编号是 [CVE-2022-20452][452]，其 patch 代码为:

```diff
diff --git a/core/java/android/os/BaseBundle.java b/core/java/android/os/BaseBundle.java
index 0418a4b..b599028 100644
--- a/core/java/android/os/BaseBundle.java
+++ b/core/java/android/os/BaseBundle.java
@@ -438,8 +438,11 @@
map.ensureCapacity(count);
}
try {
+ // recycleParcel being false implies that we do not own the parcel. In this case, do
+ // not use lazy values to be safe, as the parcel could be recycled outside of our
+ // control.
recycleParcel &= parcelledData.readArrayMap(map, count, !parcelledByNative,
- /\* lazy \*/ true, mClassLoader);
+ /\* lazy \*/ recycleParcel, mClassLoader);
} catch (BadParcelableException e) {
if (sShouldDefuse) {
Log.w(TAG, "Failed to parse Bundle, but defusing quietly", e);
@@ -1845,7 +1848,6 @@
// bundle immediately; neither of which is obvious.
synchronized (this) {
initializeFromParcelLocked(parcel, /\*recycleParcel=\*/ false, isNativeBundle);
- unparcel(/\* itemwise \*/ true);
}
return;
}
```

问题还是出现在`Bundle`反序列化的过程中，之前说过`Bundle`通过`readFromParcelInner()`去进行反序列化，其实现如下:

```java
private void readFromParcelInner(Parcel parcel, int length) {
final int magic = parcel.readInt();
final boolean isJavaBundle = magic == BUNDLE\_MAGIC;
final boolean isNativeBundle = magic == BUNDLE\_MAGIC\_NATIVE;
if (!isJavaBundle && !isNativeBundle) {
throw new IllegalStateException("Bad magic number for Bundle: 0x"
+ Integer.toHexString(magic));
}
if (parcel.hasReadWriteHelper()) {
// If the parcel has a read-write helper, it's better to deserialize immediately
// otherwise the helper would have to either maintain valid state long after the bundle
// had been constructed with parcel or to make sure they trigger deserialization of the
// bundle immediately; neither of which is obvious.
synchronized (this) {
initializeFromParcelLocked(parcel, /\*recycleParcel=\*/ false, isNativeBundle);
unparcel(/\* itemwise \*/ true);
}
return;
}
// 直接使用 Parcel.appendFrom 拷贝原 Parcel 的数据
Parcel p = Parcel.obtain();
p.appendFrom(parcel, offset, length);
mParcelledData = p;
```

注意 if 分支外部，这是大部分`Bundle`反序列化的流程，即通过新分配一个`parcel`去拷贝原始数据并保存到`mParcelledData`中。因为开发者知道`parcel`参数的生命周期不由自身控制。

而在 if 内部，调用了`initializeFromParcelLocked()`，其中会使用`readArrayMap()`对`parcel`数据进行反序列化，这其中是带有`LazyValue`的。由于开发者知道`parcel`生命周期不可控，因此在后续调用了`unparcel(true)`去强制对每个`LazyValue`进行反序列化以去除对`parcel`数据的依赖。

如果`unparcel()`内部某些`LazyValue`能够被攻击者绕过解析，那么这里就存在一个 UAF 漏洞，后续读取`LazyValue`时就能泄露出复用的`Parcel`中的数据。当然，要触发这个漏洞需要有几个前提:

1. `hasReadWriteHelper()`返回 true；
2. `unparcel()`能被绕过；

对于问题 1，我们可以在 AOSP 代码中搜索符合条件的`Parcel`类，比如`RemoteViews`；

对于问题 2，我们可以尝试使其中某个`LazyValue`解析失败，比如`Parcelable`的类不存在时，代码会抛出`ClassNotFoundException`，被捕捉后再次抛出`BadParcelableException`。有趣的是这个异常会在`getValueAt()`中被捕捉，如下所示:

```java
@Nullable
final <T> T getValueAt(int i, @Nullable Class<T> clazz, @Nullable Class<?>... itemTypes) {
Object object = mMap.valueAt(i);
if (object instanceof BiFunction<?, ?, ?>) {
try {
object = ((BiFunction<Class<?>, Class<?>[], ?>) object).apply(clazz, itemTypes);
} catch (BadParcelableException e) {
if (sShouldDefuse) {
Log.w(TAG, "Failed to parse item " + mMap.keyAt(i) + ", returning null.", e);
return null;
} else {
throw e;
}
}
mMap.setValueAt(i, object);
}
return (clazz != null) ? clazz.cast(object) : (T) object;
}
```

如果`sShouldDefuse`为 true，那么异常不会被再次抛出，而只是打印异常并返回 null。对于`system\_server`而言该条件是满足的，因为其目的就是为了防止系统重要服务频繁崩溃。因此，我们可以将`Bundle`的某个值设置为另外一个容器，比如`List`，然后在容器中存储一个不存在的`Parcelable`，那么`List`中后续的`LazyValue`将不会被递归解析到，我们也就获得了一个 UAF 对象。

## 漏洞利用

该漏洞的利用过程比较曲折，由于漏洞已经修复，因此笔者对于复现的兴趣缺缺。此外原作者也公开了详细利用思路和漏洞利用代码，感兴趣的可以自行参考:

- [Exploit for CVE-2022-20452, privilege escalation on Android from installed app to system app (or another app) via `LazyValue` using `Parcel` after `recycle()`][lv]

这里提炼一下其中值得学习的点:

1. UAF 漏洞的核心利用目标是获取目标应用的`IApplicationThread`句柄，这通常是应用启动初期使用`attachApplication()`传递给`system\_server`的，主要用于让后者给应用发送四大组件的生命周期回调。通过滥用这个`Binder`句柄可以实现 RCE，参考前面的 CVE-2021-0928 (`scheduleReceiver`)；

2. 为了能从`system\_server`中泄露任意应用的`IApplicationThread`句柄，需要两个条件。首先是有一个接口可以发送`Parcelable`数据并将其取回，`AppWidgetHost`、`Notification.contentView`、`Mediasession`都可以满足，作者选用的是`MediaSession.get/setQueue`接口；

3. 其次，要使得 UAF 对象能被包含`IApplicationThread`的`Parcel`复用，需要准确的时机。但是`attachApplication()`只在应用启动时一个很小的时间窗口中被调用，因此作者寻找其中可能存在的锁去拓展这个窗口；

4. `ParceledListSlice`是一个在 IPC 间传输的特殊数据结构，在其数据量小时可以同步传输，而对于大量的数据，会将其转换为`binder`发送给对方然后进行异步传输；

5. `ActivityManager.moveTaskToFront()`调用时可以提供`ActivityOptions`参数，以`Bundle`形式进行封装，并在`system\_server`端进行反序列化。因此攻击者可以在`Bundle`中加入`ParceledListSlice`类型的数据，从而在反序列化时回调到自身进行阻塞。该函数调用时候持有`mGlobalLock`锁，因此可以阻塞`attachApplication()`的执行，从而更好地构造 UAF 风水；

6. 由于调用了`moveTaskToFront()`，种种原因导致无法使用`startActivity()`来启动目标应用，因此作者使用`ContentProvider`来启动目标应用。

7. 在我们的 UAF `LazyValue` `Parcel`被成功占位后，对应的`IApplicationThread`实际上在比较靠前的位置，但我们的`LazyValue`所在位置比较靠后因此无法读取到对应`binder`。解决方案是可以找其他占位对象，不过作者这里利用了一个新的漏洞 [CVE-2022-20474][474]，即`readLazyValue()`方法中`objectLenth`只检查了证书溢出，却没检查负数的情况，因此设置负数的`prefixLength`实际上可以读取`Parcel`前方的数据，从而绕过该限制。

其他还有亿点细节，就需要动手复现才能真正理解了。理论上该漏洞可以影响安全补丁在 2022-11 之前的 Android 13 设备。攻击的目标应用可以是任意应用，因此可以选择`Settings.apk`，其 UID=system，攻击成功后就相当于获取到了 system 权限。这在 Goole VRP 中应该是 10 万刀乐的标准。

[452]: <mark class="encrypted">8baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5c8W2)9J5b7W2)9J5c8U0q4S2j5h3f1%4x3U0l9%4y4K6u0S2z5o6k6W2x3X3c8T1y4U0R3J5k6o6u0W2z5h3g2V1y4K6M7&6x3K6M7K6x3K6c8W2y4o6M7#2k6U0y4Q4x3U0f1#2c8g2)9J5y4e0t1I4i4K6u0r3</mark>
