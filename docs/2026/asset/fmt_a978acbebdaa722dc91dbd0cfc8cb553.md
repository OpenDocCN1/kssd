
继续查看 `backup_method` 表示其为 Hook 方法的原方法，然后为备份的 `ArtMethod` 创建对应的 `Method` 对象。

![](https://bbs.kanxue.com/upload/attach/202002/819934_2AECXP6TPGGR3UM.jpg)

## EnableXposedHook②

（
### fast_jin 模式科普：

下文参考资料：《深入理解ART虚拟机》：
安卓 函数执行 分为两条线 第一种是 Java层，第二种JNI层 也就是 so层
当函数调用Java层进入到JNI层的是时候，虚拟机会将执行线程的状态从Runnable转换为Native。
如果JNI层又调用Java层相关函数的时候，执行线程的状态又得从Native层转换为Runnable。
线程的切换需要浪费时间，所以，对于某个特别强调执行速度的JNI函数可以设置成 fast jni模式
这种模式下执行这个native函数 将不会进行 状态切换，即执行线程的状态 始终为Runnable。
当然，这种模式的使用对GC有一些影响，所哟最好在那些本身函数执行时间段的，又不会阻塞的情况下使用。
另外，这种模式目前在art虚拟机内部 很多java native都有使用
为了和其他Native函数 进行区分，当使用fast jni模式的函数的签名信息 必须以 “！”开头
）

![](https://bbs.kanxue.com/upload/attach/202002/819934_GP7TUYDZC4344RE.jpg)

## EnableXposedHook③

把 `Method` 对象，方法额外信息和原始方法保存至 `XposedHookInfo` 结构体中，并调用 `SetEntryPointFromJni()` 把这个结构体变量的内存地址保存在 `ArtMethod` 对象中。这个方法原本是用来保存 native 方法的入口地址的，既然使用了这个位置，那么就必须把对应的标志位清除，代码实现的最后调用 `SetAccessFlags((GetAccessFlags() & ~kAccNative & ~kAccSynchronized) | kAccXposedHookedMethod)` 来完成标志位的清除（设置 Fast_jni 模式），此时这个 `ArtMethod` 对象对应是 Hook 后的方法，这个方法的实现不是 native 的。

![](https://bbs.kanxue.com/upload/attach/202002/819934_F39X982MZX87H9G.jpg)

## EnableXposedHook④

这么一来完成了整体 Hook 流程

### 总结：

#### 执行流程：

`XposedBridge.hookMethod` -> `XposedBridge.hookMethodNative` -> `EnableXposedHook`

核心代码 So 层里面的 `hookMethodNative` 和 `XposedBridge_hookMethodNative` 里面

1, `hookMethodNative` 先将 java 层传入的 被 Hook 的信息转换成 `ArtMethod`，方便调用方法进行 Hook，调用 `EnableXposedHook` 方法。
2, 在 `EnableXposedHook` 进行简单的判断 是否是被 Hook 的方法，以及是否已经被 Hook 过
准备一个备份的 `ArtMethod` 存放 原方法的信息，将备份的 `ArtMethod` 设置信息，所属类，告诉虚拟机这个方法不需要 JIT 编译，并将其设置成 Native，准备一个简单的结构体 `XposedHookInfo` 保存，保存被 Hook 方法的信息，包括原方法的信息，地址，最后将 入口设置成 `XposedHookInfo`，设置机械码执行的首地址，将原方法的 `CodeItem` 偏移设置 0。

### 参考：

32aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1W2F1L8X3!0K6N6q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0l9@1y4U0p5%4z5o6x3`.

7cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5Y4y4Q4x3X3g2E0k6h3W2*7N6g2)9J5k6h3y4F1i4K6u0r3N6r3S2J5k6h3q4V1i4K6u0V1z5o6x3J5z5o6t1@1y4g2)9J5k6o6q4Q4x3X3b7I4i4K6u0W2K9s2c8E0L8l9`.

邓凡平--- 《深入理解安卓虚拟机Art》

15eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8j5h3&6U0L8r3!0#2k6q4)9J5k6h3y4F1i4K6u0r3j5h3I4W2P5q4)9#2k6Y4N6K6j5#2)9J5c8X3q4F1k6s2u0G2K9h3c8K6i4K6u0r3y4o6M7K6y4U0t1I4

643K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3p5K6x3e0b7I4x3K6p5H3y4K6m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3I4x3o6V1J5y4e0t1$3

9b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6B7P5o6R3K6z5e0f1J5y4o6V1H3y4W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6p5H3y4o6j5^5y4o6b7`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-2-26 18:04

被珍惜Any编辑

，原因：
