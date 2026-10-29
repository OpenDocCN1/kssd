
# APK 自我保护

[*]将除`META-INF`文件夹之外的文件压缩成`zip`文件，签名后生成`.apk`文件。

若是攻击者使用`apktool`重打包，运行重打包后的文件会出现如下运行时错误：

![](https://bbs.kanxue.com/upload/attach/201312/444052_ovzt8dkrel0layo.png)

**图 17 使用 Manifest Cheating 重打包后 APK 文件运行时错误**

## 调试器检测

在对 APK 逆向分析时，往往会采取动态调试技术，可以使用`netbeans+apktool`对反汇编生成的`smali`代码进行动态调试。为了防止 APK 被动态调试，可以检测是否有调试器连接。

Android 系统在`android.os.Debug`类中提供了`isDebuggerConnected()`方法，用于检测是否有调试器连接。可以在`Application`类中调用`isDebuggerConnected()`方法，判断是否有调试器连接，如果有，直接退出程序。

除了`isDebuggerConnected`方法，还可以通过在`AndroidManifest`文件的`application`节点中加入`android:debuggable="false"`使得程序不可被调试，这样如果希望调试代码，则需要修改该值为`true`，因此可以在代码中检查这个属性的值，判断程序是否被修改过，代码如下：

```
1. if(getApplicationInfo().flags &= ApplicationInfo.FLAG_DEBUGGABLE != 0){

2. System.out.println("Debug");

3. android.os.Process.killProcess(android.os.Process.myPid());

4. }
```

## 代码混淆

使用 Java 编写的代码很容易被反编译，因此可以使用代码混淆的方法增加反编译代码阅读的难度。`ProGuard`是一款免费的 Java 代码混淆工具，提供了文件压缩、优化、混淆和审核功能。在 Eclipse+ADT 开发环境下，每个 Android 应用程序项目目录下会默认生成`project.properties`和`proguard-project.txt`文件。如果需要使用`ProGuard`进行压缩以及混淆，首先需要在`project.properties`文件中去掉对如下语句的注释：

```
1. proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt
```

`ProGuard`的相关配置信息需要在`proguard-project.txt`文件中声明，在其中可以设置需要混淆和保留的类或方法。由于在某些情况下，`ProGuard`会错误地认为某些代码没有被使用，如在只在`AndroidManifest`文件中引用的类，从`JNI`中调用的方法等。对于这些情况，需要在`proguard-project.txt`文件中添加`-keep`命令，用来保留类或方法。关于`ProGuard`更加详细的配置项可以参考`ProGuard Manual`。

除了使用`ProGuard`对 Android 代码进行混淆外，还可以使用`DexGuard`。`DexGuard`是特别针对 Android 的一款代码优化混淆的收费软件，提供代码优化混淆、字符串加密、类加密、`Assets`资源加密、隐藏对敏感 API 的调用、篡改检测以及移除 Log 代码。`DexGuard`的进一步分析可以参考 JEB 上的相关 blog，可以在这里总结一下：

### 1．字符串加密

经过`DexGuard`加固过的 APK，对字符串的访问会通过调用一个解密函数来完成加密字符串的解密。如图 18 所示，红框中的字节数组是加密后的字符串，在`onCreate`函数中，调用了解密函数进行解密。字符解密函数如图 19 所示，对其进行处理后如图 20 所示，加密算法也很简单，基本思路是：当前字符由前一个字符加上加密字符数组中的字符，再减去常量 8 形成，当字符长度达到给定的长度时，会最终构成字符串并返回。

![](https://bbs.kanxue.com/upload/attach/201312/444052_ro0iyhtsb0s6ub8.png)

**图 18 DexGuard 字符串加密**

![](https://bbs.kanxue.com/upload/attach/201312/444052_24oje5lk5c5xx1o.png)

图 19 字符串解密函数

![](https://bbs.kanxue.com/upload/attach/201312/444052_i29utp5mzdmoebv.png)

图 20 处理后的解密函数

### 2. assets 加密

APK 文件的`assets`目录下包含了应用程序需要使用到的资源文件，`DexGuard`提供了对`assets`资源文件的加密功能。对于一个经过保护的`asset`资源文件，例如 `1.png`文件，使用十六进制查看器查看该文件，如图 21 所示。从图中可见，加密后的`png`文件，缺失了相应的文件头。解密则是首先通过反射调用`AssetManager.open`函数，同时对该函数的反射调用

![](https://bbs.kanxue.com/upload/attach/201312/444052_qulgmavaczmdyb0.png)

**图 21 经过加密的 png 文件**

又使用了加密处理，最后通过`Cipher`类完成`png`文件的解密。相关解密处理如所示。

![](https://bbs.kanxue.com/upload/attach/201312/444052_v6h3m6o3kg6xupr.png)

**图 22 asset 解密处理**

关于`ProGuard`和`DexGuard`还可以参考`ProGuard and DexGuard`，其中除了介绍了`ProGuard`和`DexGuard`，还提供了一些 APK 加固处理的方法。

关于代码混淆，还可以参考`Android:Game of Obfuscation`。

## NDK

Android 软件的开发主要使用 Java 语言，但是 Android 也提供了对本地语言 C、C++的支持。借助`JNI`，可以在 Java 类中使用 C 语言库中的特定函数，或在 C 语言程序中使用 Java 类库。一般来说，如果代码中对处理速度有较高要求或者为了更好地控制硬件，抑或者为了复用既有的 C/C++代码，都可以考虑通过`JNI`来实现对 Native 代码的调用。

由于逆向 Native 程序的汇编代码要比逆向 Java 汇编代码困难，因此可以考虑在关键代码部位使用 Native 代码，如注册验证，加解密操作等。一个可能的借助 Native 代码保护 APK 的方法是：将核心业务逻辑代码放入加密的`.jar`或者`.apk`文件中，在需要调用时使用 Native 代码进行解密，同时完成对解密后文件的完整性校验，不过不管是`.jar`还是`.apk`文件，解密后都会留在物理存储上，为了避免这种情况，可以使用反射技术直接调用`dalvik.system.DexFile.openDex()`方法，该方法接受`classes.dex`文件字节流返回`DexFile`对象。

关于 Native 代码的编写，可以参考 Google 官方文档的 Android NDK。

## 逆向工具对抗

在逆向分析 Android 应用程序时，一般会使用`apktool`，`baksmali/smali`，`dex2jar`，`androguard`，`jdGUI`以及`IDA Pro`等。因此可以考虑使得这些工具在反编译 APK 时出错来保护 APK，这些工具大部分都是开源的，可以通过阅读其源代码，分析其在解析 APK、dex 等文件存在的缺陷，在开发 Android 应用程序时加以利用。可以参考 Tim Strazzere 的 `Dex Education:Practicing Safe Dex`，相应的 Demo，看雪上的中文翻译，不过其中的很多技巧已经失效了。`DexLabs`的`Dalvik Bytecode Obfuscation on Android`介绍了垃圾字节码插入的技术。

使用`apktool`进行重打包时，对于后缀为`png`的文件，会按照`png`格式的文件进行打包处理，因此如果在项目开发时，有意将一个非`png`格式文件的文件名改为后缀为`png`的文件，则使用`apktool`进行重打包时会出错。可以利用这种方法来对抗重打包。可以试试对这个文件使用`apktool`进行重打包，会报很多错误，但是这种 appt 导致的错误，很多都是由于第一个错误一起的，如图 23 所示。从第一个错误描述中可知，`res/drawable-hdpi/station.png`不是

![](https://bbs.kanxue.com/upload/attach/201312/444052_8b1y6w03anjccqa.png)

**图 23 apktool 重打包错误**

一个 PNG 格式的文件，使用`file`命令，可以发现实际上是一个 Windows icon 文件，如图 24 所示。将这个文件后缀修改成`.icon`就可以重新打包了。

![](https://bbs.kanxue.com/upload/attach/201312/444052_51ce8z4oyceksi5.png)

**图 24 station.png 的真实文件类型**

## 总结

以上 APK 自我保护的技术并不能做到完全的保护作用，只是提高了逆向分析的难度，在实际运用中应该根据情况多种技术结合使用。这些技术其实很多来源于 Android 恶意代码，所以可以关注 Android 恶意代码中使用的一些技术来应用到自己开发的 Android 应用程序中。

注：本帖由看雪论坛志愿者PEstone 重新将pdf整理排版，若和原文有出入，以原作者附件为准

总结的一些关于APK自我保护的方法，当然还有很多其他的技巧，无法一一列举，现在可以使用的一些服务包括：

1.梆梆 :[d48K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2L8X3N6U0L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@a6cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2L8X3N6U0L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.)

2.爱加密 :[f52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6K9X3W2S2L8h3W2Q4x3X3g2U0L8W2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@ff4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6K9X3W2S2L8h3W2Q4x3X3g2U0L8W2)9J5c8R3`.`.)

3.APKProtect :[7b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4H3K9%4m8J5L8%4c8W2j5%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@a46K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4H3K9%4m8J5L8%4c8W2j5%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5`.)

4.Shield4J :[da8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5K9h3g2D9k6o6c8B7i4K6u0W2j5$3!0E0i4K6u0r3](https://bbs.kanxue.com/elink@33bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5K9h3g2D9k6o6c8B7i4K6u0W2j5$3!0E0i4K6u0r3)

5.DexGuard :[183K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4S2K9h3E0G2j5g2)9J5k6h3y4G2L8g2)9J5c8X3c8W2P5r3N6#2j5i4u0V1](https://bbs.kanxue.com/elink@41aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4S2K9h3E0G2j5g2)9J5k6h3y4G2L8g2)9J5c8X3c8W2P5r3N6#2j5i4u0V1)

欢迎补充！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [APK自我保护.pdf](javascript:void(0);)

  （1.37MB，1057次下载）

- [1.png](javascript:void(0);)

  （42.98kb，20次下载）

- [2.png](javascript:void(0);)

  （20.47kb，13次下载）

- [3.png](javascript:void(0);)

  （19.24kb，13次下载）

- [4.png](javascript:void(0);)

  （56.74kb，8次下载）

- [5.png](javascript:void(0);)

  （30.18kb，11次下载）

- [6.png](javascript:void(0);)

  （16.37kb，9次下载）

- [7.png](javascript:void(0);)

  （12.48kb，8次下载）

- [8.png](javascript:void(0);)

  （20.27kb，9次下载）

- [9.png](javascript:void(0);)

  （12.25kb，14次下载）

- [10.png](javascript:void(0);)

  （8.36kb，11次下载）

- [11.png](javascript:void(0);)

  （106.08kb，10次下载）

- [12.png](javascript:void(0);)

  （6.55kb，8次下载）

- [13.png](javascript:void(0);)

  （12.81kb，11次下载）

- [14.png](javascript:void(0);)

  （17.49kb，9次下载）

- [15.png](javascript:void(0);)

  （37.38kb，10次下载）

- [16.png](javascript:void(0);)

  （53.23kb，9次下载）

- [17.png](javascript:void(0);)

  （83.26kb，9次下载）

- [18.png](javascript:void(0);)

  （72.38kb，13次下载）

- [19.png](javascript:void(0);)

  （75.19kb，10次下载）

- [20.png](javascript:void(0);)

  （71.05kb，8次下载）

- [21.png](javascript:void(0);)

  （105.58kb，10次下载）

- [22.png](javascript:void(0);)

  （92.79kb，11次下载）

- [23.png](javascript:void(0);)

  （123.47kb，10次下载）

- [24.png](javascript:void(0);)

  （26.29kb，12次下载）
