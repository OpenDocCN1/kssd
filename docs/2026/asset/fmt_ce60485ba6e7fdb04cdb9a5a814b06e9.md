
- [ReparcelBug - PoC for CVE-2017-0806](https://bbs.kanxue.com/elink@23aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6d9k6i4m8S2M7X3y4W2L8p5u0#2k6H3`.`.)

- [CVE-2017-0806 原理分析](https://bbs.kanxue.com/elink@b99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6u0L8Y4c8W2L8Y4c8K6e0r3q4T1i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3x3W2)9J5x3$3W2K6M7%4g2W2j5$3!0E0L8h3g2F1N6q4)9J5k6o6x3@1y4o6x3$3y4e0b7^5x3R3`.`.)

- [Bundle风水——Android序列化与反序列化不匹配漏洞详解](https://bbs.kanxue.com/elink@6b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5x3K6j5@1)

## 漏洞修复

上述漏洞的修复似乎很直观，只需要把`Vulnerable`类中不匹配的读写修复就行了。但实际上这类漏洞并不是个例，历史上由于代码编写人员的粗心大意，曾经出现过许多因为读写不匹配导致的提权漏洞，包括但不限于：

- CVE-2017-0806 `GateKeeperResponse`
- CVE-2017-0664 `AccessibilityNodelnfo`
- CVE-2017-13288 `PeriodicAdvertisingReport`
- CVE-2017-13289 `ParcelableRttResults`
- CVE-2017-13286 `OutputConfiguration`
- CVE-2017-13287 `VerifyCredentialResponse`
- CVE-2017-13310 `ViewPager`'s `SavedState`
- CVE-2017-13315 `DcParamObject`
- CVE-2017-13312 `ParcelableCasData`
- CVE-2017-13311 `ProcessStats`
- CVE-2018-9431 `OSUInfo`
- CVE-2018-9471 `NanoAppFilter`
- CVE-2018-9474 `MediaPlayerTrackInfo`
- CVE-2018-9522 `StatsLogEventWrapper`
- CVE-2018-9523 `Parcel.wnteMapInternal0`
- CVE-2021-0748 `ParsingPackagelmpl`
- CVE-2021-0928 `OutputConfiguration`
- CVE-2021-0685 `ParsedIntentInfol`
- CVE-2021-0921 `ParsingPackagelmpl`
- CVE-2021-0970 `GpsNavigationMessage`
- CVE-2021-39676 `AndroidFuture`
- CVE-2022-20135 `GateKeeperResponse`
- ...

另一个修复思路是修复 TOCTOU 漏洞本身，即确保检查和使用的反序列化对象是相同的，但这种修复方案也是治标不治本，同样可能会被攻击者找到其他的攻击路径并绕过。

因此，为了彻底解决这类层出不穷的问题，Google 提出了一种简单粗暴的缓释方案，即直接从`Bundle`类中下手。虽然`Bundle`本身是`ArrayMap`结构，但在反序列化时候即便只需要获取其中一个 key，也需要把整个`Bundle`反序列化一遍。这其中的主要原因在于序列化数据中每个元素的大小是不固定的，且由元素的类型决定，如果不解析完前面的所有数据，就不知道目标元素在什么地方。

为此在 21 年左右，AOSP 中针对`Bundle`提交了一个称为[LazyBundle(9ca6a5)](https://bbs.kanxue.com/elink@138K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3i4K6g2X3i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8Y4m8D9j5i4c8X3L8%4u0E0i4K6u0r3k6Y4u0S2L8h3g2%4L8%4u0C8M7#2)9J5c8X3u0S2M7$3g2Q4x3V1k6Q4x3V1u0Q4x3V1j5&6j5$3p5$3j5e0g2W2x3U0q4S2x3e0V1^5y4$3k6V1x3K6R3H3x3r3p5^5z5e0W2U0x3e0x3^5y4r3t1J5x3X3b7J5x3$3t1$3k6r3g2W2)的 patch。其主要思想为针对一些长度不固定的自定义类型，比如`Parcelable`、`Serializable`、`List`等结构或容器，会在序列化时将对应数据的大小添加到头部。这样在反序列化时遇到这些类型的数据，可以仅通过检查头部去选择性跳过这些元素的解析，而此时 sMap 中对应元素的值会设置为`LazyValue`，在实际用到这些值的时候再去对特定数据进行反序列化。

这个 patch 可以在一定程度上缓释针对 Bundle 风水的攻击，而且在提升系统健壮性也有所助益，因为即便对于损坏的 Parcel 数据，如果接收方没有使用到对应的字段，就可以避免异常的发生。对于之前的 Bundle 解析策略，哪怕只调用了`size`方法，也会触发所有元素的解析从而导致异常。在这个 patch 中`unparcel`还增加了一个 boolean 参数`itemwise`，如果为 true 则按照传统方式解析每个元素，否则就会跳过`LazyValue`的解析。

# CVE-2021-0928

在前面反序列化的示例中，漏洞主要出在一个自定义的`Vulnerable`类中，即手工编写的`readFromParcel`/`writeToParcel`不匹配问题。在现实中，这种出现问题的类通常只在进程间使用而几乎不用于跨进程，否则在正常 IPC 调用时候就会出现明显的数据错误。但也还有一种情况，比如在`readFromParcel`方法调用的过程中出现异常，但是上级进行了“优雅”的`try-catch`，从而程序并不会报错，但异常发生后`Parcel`中还有剩余数据并未消费，因此会被后续的解析错误使用，比如在 AIDL 调用时候就会被后续的参数用到。

`CVE-2021-0928`就是这么一个经典的漏洞。漏洞本身只存在于 Andorid 12 beta3 的短暂时间窗口中，因此我们并不需要太过于关注细节，此外漏洞的作者也已经在 Github 上公开了该漏洞的详细介绍以及 POC 代码。作为历史漏洞研究，笔者这里只做简单的介绍，并重点关注一些值得学习的漏洞利用思路和漏洞模式。完整的漏洞细节分析和 POC 可以参考原作者的分享：

- [CVE-2021-0928, writeToParcel/createFromParcel serialization mismatch in android.hardware.camera2.params.OutputConfiguration](https://bbs.kanxue.com/elink@e0cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6d9k6i4m8S2M7X3y4W2L8p5u0#2k6K6t1`.)

## 漏洞介绍

主要出现漏洞的类为`android.hardware.camera2.params.OutputConfiguration`，其代码片段如下所示：

```java
package android.hardware.camera2.params;

public final class OutputConfiguration implements Parcelable {
    private OutputConfiguration(@NonNull Parcel source) {
        int rotation = source.readInt();
        int surfaceSetId = source.readInt();
        // ...
        boolean isMultiResolution = source.readInt() == 1; // New in Android 12
        ArrayList<Integer> sensorPixelModesUsed = new ArrayList<Integer>(); // New in Android 12
        source.readList(sensorPixelModesUsed, Integer.class.getClassLoader()); // New in Android 12
        // ...
    }

    public static final @android.annotation.NonNull Parcelable.Creator<OutputConfiguration> CREATOR =
            new Parcelable.Creator<OutputConfiguration>() {
        @Override
        public OutputConfiguration createFromParcel(Parcel source) {
            try {
                OutputConfiguration outputConfiguration = new OutputConfiguration(source);
                return outputConfiguration;
            } catch (Exception e) {
                Log.e(TAG, "Exception creating OutputConfiguration from parcel", e);
                return null;
            }
        }
    };
}
```

其中在`createFromParcel`中对构造函数进行了异常捕捉，但是对于异常仅仅是打印并返回，这导致 Parcel 数据残留从而影响后续的数据解析。

该漏洞本身并不复杂，是个很常见的 Parcel 序列化/反序列化不匹配问题，关键是这个漏洞如何利用？由于漏洞的本质是序列化数据残留，因此我们可以主要关注一些 AIDL 调用场景的参数，这样解析下一个参数时其内容也就被我们所污染的数据控制。其中一个漏洞触发链路就是`sendBroadcast`。

## Android 广播

关于 Android 广播的发送和接收流程应该已经有很多其他博客介绍过了，这里将其简化为下面的阶段，假设广播由应用 A 发送给应用 B：

1. A 调用`sendBroadcast`，这实际上是个 AIDL 接口，接收方即实际实现为`system_server`；
2. `system_server`接收到 Intent 后进行一系列处理，判断合法的接收方，然后调用`IApplicationThread.scheduleReceiver`发送广播数据；这同样是一个 AIDL 接口，由各个应用在启动之初通过`IActivityManager.attachApplication()`传递给`system_server`；因此这个调用实际上会进入到应用 B 中；
3. B 触发`ActivityThread.handleMessage`，进而调用`handleReceiver`；
4. `handleReceiver`中通过传来的数据去实例化应用上下文，获取对应的 receiver，最终调用开发者注册的`onReceive`回调。

第 2 步中`scheduleReceiver`的定义如下：

```java
public final void scheduleReceiver(
        Intent intent,
        ActivityInfo info,
        CompatibilityInfo compatInfo,
        int resultCode,
        String data,
        Bundle extras,
        boolean sync,
        int sendingUser,
        int processState);
```

其中第一个`intent`是 A 应用指定的`sendBroadcast`的参数，第二个参数则是`system_server`传递给应用 B 的。漏洞利用思路就是通过`intent`参数的反序列化数据残留，间接地修改`info`参数，因为应用 B 会使用`ActivityInfo`中的数据去实例化代码，具体来说就是：

```java
private void handleReceiver(ReceiverData data) {
    String component = data.intent.getComponent().getClassName();
    LoadedApk packageInfo =
            getPackageInfoNoCheck(data.info.applicationInfo, data.compatInfo);
    app = packageInfo.makeApplicationInner(false, mInstrumentation);
    // ...
}
```

应用会通过传入的`ActivityInfo`构造`LoadedApk`，其中会使用内部的`zipPaths`创建 ClassLoader，攻击者如果可以控制这个字段，就能实现针对应用 B 的任意代码执行。

要实现这个目的，还有亿点细节需要解决，首先我们先来看一个 Java 语言的特性。

## Java 类型擦除

Java 类型擦除也称为[Type Erasure](https://bbs.kanxue.com/elink@0abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1j5h3g2D9k6s2g2F1k6#2)9J5k6h3y4G2L8g2)9J5c8X3A6S2N6X3q4Q4x3X3c8@1P5i4m8W2i4K6u0V1k6i4u0S2M7%4g2J5k6b7`.`.)，主要产生在 Java 的泛型代码中。Java 虚拟机中并没有泛型类型的概念，因此在编译时实际类型会被替换为其限制对象或者原始对象`Object`类型，这个过程就叫做类型擦除。

例如，下述代码：

```java
public static <E> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

会在编译时变成：

```java
public static void printArray(Object[] array) {
    for (Object element : array) {
        System.out.printf("%s ", element);
    }
}
```

即模版类型 E 变成了原始类型 Object。对于有限制类型的模版，比如：

```java
public static <E extends Comparable<E>> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

会在编译时替换成其限制类型：

```java
public static void printArray(Comparable[] array) {
    for (Comparable element : array) {
        System.out.printf("%s ", element);
    }
}
```

那么，这个特性对于漏洞的利用有什么帮助呢？回顾`OutputConfiguration`类，其中有个序列化的字段`sensorPixelModesUsed`是`ArrayList<Integer>`类型，在反序列化该字段时，使用的是`Parcel.readList`方法，其调用链路是：

- `Parcel.readList`
