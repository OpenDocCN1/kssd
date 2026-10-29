
# 代码乱序

```
1. .method public test()V

2. .locals 4

3.

4. .prologue

5. .line 24

6. const/4 v0, 0x1

7.

8. .line 25

9. .local v0, a:I

10. const/4 v1, 0x2

11.

12. .line 26

13. .local v1, b:I

14. add-int v2, v0, v1

15.

16. .line 27

17. .local v2, c:I

18. sget-object v3, Ljava/lang/System;->out:Ljava/io/PrintStream;

19.

20. invoke-virtual {v3, v2}, Ljava/io/PrintStream;->println(I)V

21.

22. .line 28

23. return-void
```

我们可以根据上述提到的代码乱序原理，将 `test` 这个函数乱序成如下代码所示(删除了`.line`)：

```
1. .method public test()V

2. .locals 4

3.

4. .local v2, c:I

5. goto :lab1

6. :lab3

7. sget-object v3, Ljava/lang/System;->out:Ljava/io/PrintStream;

8. invoke-virtual {v3, v2}, Ljava/io/PrintStream;->println(I)V

9. goto :end

10.

11. .local v1, b:I

12. :lab2

13. add-int v2, v0, v1

14. goto :lab3

15.

16. .local v0, a:I

17. :lab1

18. const/4 v0, 0x1

19. const/4 v1, 0x2

20. goto :lab2

21.

22. :end

23. return-void

24. .end method
```

最后使用 `apktool` 重新打包发布。进行代码乱序可以在一定程度上增加逆向分析的难度，例如可以使用 `dex2jar`+`jd-GUI` 工具来分析上述乱序前后的代码。乱序前代码如图 7 所示：

![](https://bbs.kanxue.com/upload/attach/201312/444052_hwq60pgsc383jq2.png)

**图 7 使用 jd-GUI 分析乱序前代码**

乱序后的代码如图 8 所示。从乱序前后的代码可以看出，使用代码乱序技术能够在一定程度上增加逆向分析的难度，当然这是因为 `dex2jar` 工具在进行代码解析时的问题，如果能够针对性的处理这种代码乱序的情况，那么这种反编译的情况应该会有所好转。

关于代码乱序的技术，可以参考 `ADAM:An automatic and extensible platform to stress test android anti-virus systems`、`DroidChameleon:Evaluating Android Anti-malware against Transformation Attacks`。

![](https://bbs.kanxue.com/upload/attach/201312/444052_2jwow42qw69egww.png)

**图 8 使用 jd-GUI 分析乱序后代码**

# 模拟器检测

一般在分析 APK 的过程中会借助于 Android 模拟器，比如分析网络行为，动态调试等。因此从 APK 自我保护的角度出发，可以增加对 APK 当前运行环境的检测，判断是否运行在模拟器中，如果运行在模拟器中可以选择退出整个应用程序的执行或者跳到其他分支。模拟器检测的手段有很多，下面逐一分析。

## 1.属性检测

Android 属性系统类似于 Windows 的注册表机制，所有的进程可以共享系统设置值。关于 Android 属性系统的详细原理，可以参考我对于 Android 属性系统的分析文章 `http://www.kanxue.com/bbs/showthread.php?t=182901`。一些属性值在 Android 模拟器和真机上是不同的，例如对于 Nexus4 和 SDK 为 4.1.2 的模拟器来说，`Build.BRAND` 和 `Build.DEVICE` 属性值分别如图 9 和图 10 所示。根据这些属性值在真实机器和模拟器上的差别可以比较容易的

![](https://bbs.kanxue.com/upload/attach/201312/444052_48f4jd37j5tohn6.png)

**图 9 Nexus4 的 BRAND 和 DEVICE 值**

![](https://bbs.kanxue.com/upload/attach/201312/444052_cp9gph9y0m73q60.png)

**图 10 Android 模拟器的 BRAND 和 DEVICE 值**

检测 Android 应用程序是否运行在模拟器中。不过对于这种检测方式，绕过也是比较容易的，我现在想到的有 3 种方式绕过：

- 在源码中修改相应的属性值，重新编译生成内核等镜像文件，再使用这些重新生成的镜像文件加载模拟器(对于 `BRAND` 属性值可以修改`/build/target/product/generic.mk`文件中的 `PRODUCT_BRAND` 值，重新编译过程没测试，是不是只需要修改这个值就能搞定不一定正确，可以参考下 `build.prop` 生成过程分析)；
- 修改 `boot.img` 文件；
- 使用 `Xposed` 框架，可以 hook `SystemProperties.get` 函数，在 `before` 函数中检查需要获取的属性，根据情况修改对应的值，然后返回；

另外还可以通过检测 IMEI，IMSI 等值来判断是否是模拟器，在模拟器中，这两个值默认分别是 `000000000000000` 和 `310260000000000`。通过以下代码可以获取 IMSI 值：

```
1. TelephonyManager manager = (TelephonyManager)getSystemService(TELEPHONY_SERVICE);

2. String imsi = manager.getSubscriberId();
```

`TELEPHONY_SERVICE` 需要申请 `android.permission.READ_PHONE_STATE` 权限。同样我们可以有相应的绕过方式，一个相对简单的方法是直接修改 Android SDK 下`/tools/emulator-arm.exe` 文件(Windows 版本)。使用 `010Editor` 打开 `emulator-arm.exe` 文件，搜索 `CIMI`，如图 11，”`CIMI.`”后面的 15 位数字值是 IMSI，”`CGSN.`”后面的 15 位数字为 IMEI，修改这两个值(确保没有运行模拟器)，然后保存。

![](https://bbs.kanxue.com/upload/attach/201312/444052_t4yf4wbkslaff0y.png)

**图 11 修改 IMSI 和 IMEI**

修改后再运行模拟器，此时查看 IMSI 值如所示，IMEI 值如所示，可见可以成功修改这两个值。

![](https://bbs.kanxue.com/upload/attach/201312/444052_m4rw3ynkp51hmi6.png)

**图 12 修改后的 IMSI 值**

![](https://bbs.kanxue.com/upload/attach/201312/444052_moqx6ifmcdwspeg.png)

**图 13 修改后的 IMEI 值**

还存在其他的修改方式，可以参考 `Hide the Emulator`以及 `Android emulator patch for configurable IMEI, IMSI and SIM card serial number`。

当然还可以检查一些其他的值，如电池的电池状态、电池电量，`Secure.ANDROID_ID`，`DeviceId`，手机号码等。

## 2.虚拟机文件检测

相对于真实设备，Android 模拟器中存在一些特殊的文件或者目录，如`/system/bin/qemu-props`，该可执行文件可以用来在模拟器中设置系统属性。另外还有`/system/lib/libc_malloc_debug_qemu.so` 文件以及`/sys/qemu_trace` 目录。我们可以通过检测这些特殊文件或者目录是否存在来判断 Android 应用程序是否运行在模拟器中，关键代码如下：

```
1. private static String[] known_files = {

2. "/system/lib/libc_malloc_debug_qemu.so",

3. "/sys/qemu_trace",

4. "/system/bin/qemu-props"

5. };

6.

7. public static boolean hasQEmuFiles() {

8. for(String pipe : known_files) {

9. File qemu_file = new File(pipe);

10. if (qemu_file.exists())

11. return true;

12. }

13. return false;

14. }
```

更完整的代码可以参考 Tim Strazzere 的 Github 中 `anti-emulator`，该项目中还列举了其他一些模拟器检测的方法，如检测 socket 文件`/dec/socket/qemud`。

## 3.基于 Cache 行为的模拟器检测方法

BlueBox 关于 Android 模拟器检测的方法

62bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9N6h3g2T1L8%4S2Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0H3L8%4u0S2N6r3g2Q4x3X3c8T1L8r3!0Y4i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6r3g2E0N6h3I4S2N6r3!0J5i4K6u0V1k6r3g2@1k6h3y4@1K9h3!0F1i4K6u0r3

## 4.基于代码指令执行的模拟器检测方法

DexLabs 关于 Android 模拟器检测的方法

04cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2P5r3I4S2j5Y4y4Q4x3X3g2G2M7X3N6Q4x3V1k6T1L8r3!0Y4i4K6u0r3j5Y4c8V1k6i4c8W2j5%4b7`.

## 5.其他方法

其他一些检测方法，可以参考如下文献：

- `DISSECTING THE ANDROID BOUNCER`
- `逃离安卓动态检测`
- `Guns and Smoke to Defeat Mobile Malware`
- `DEX EDUCATION 201 ANTI-EMULATION`
- `INSECURE MAGZINE 34 – Introduction to Android malware analysis`

# APK 伪加密

APK 实际上是 Zip 压缩文件，但是 Android 系统在解析 APK 文件时，和传统的解压缩软件在解析 Zip 文件时还是有所差异的，利用这种差异可以实现给 APK 文件加密的功能。Zip 文件格式可以参考 MasterKey 漏洞分析的一篇文章。在 Central Directory 部分的 File Header 头文件中，有一个 2 字节长的名为 General purpose bit flags 的字段，这个字段中每一位的作用可以参考 Zip 文件格式规范的 4.4.4 部分，其中如果第 0 位置 1，则表示 Zip 文件的该 Central Directory 是加密的，如果使用传统的解压缩软件打开这个 Zip 文件，在解压该部分 Central Directory 文件时，是需要输入密码的，如图 14 所示。但是 Android 系统在解析 Zip 文件时并没有使用这一位，也就是说这一位是否置位对 APK 文件在 Android 系统的运行没有任何影响。

一般在逆向 APK 文件时，会首先使用 `apktool` 来完成资源文件的解析，`dex` 文件的反汇编工作，但如果将 Zip 文件中 Central Directory 的 General purpose bit flags 第 0 位置 1 的话，`apktool(version:1.5.2)` 将无法完成正常的解析工作，如图 15 所示，但是又不会影响到 APK 在 Android 系统上的正常运行，如图 16 所示。

![](https://bbs.kanxue.com/upload/attach/201312/444052_eroxze63wpfhfsg.png)

**图 14 传统解压缩软件需要输入密码进行解压缩**

![](https://bbs.kanxue.com/upload/attach/201312/444052_inicqbb310mknb5.png)

**图 15 apktool 解析伪加密的 APK 文件失败**

对 APK 文件进行伪加密可以使用这个脚本，在 Python 的 `zipfile` 模块中，`ZipInfo` 类中记录了 Zip 文件中相应的 Central Directory 的相关信息，包括 General purpose bit flags，在 `ZipInfo` 类中属性为 `flag_bits`，因此上述脚本中将需加密的 APK 文件的每个 `ZipInfo` 的 `flag_bits` 和 1 做或操作，实现在 General purpose bit flags 的第 0 位置 1.

而需要去除这些伪加密的标志的话，可以使用这个脚本。相关内容可以参考 BlueBox 之前提出的一个 Android Security Analysis Chanllenge.。

![](https://bbs.kanxue.com/upload/attach/201312/444052_ixo9bg74dgeumc8.png)

**图 16 伪加密的 APK 可以正常运行**

```
1. unsigned int gpbf = get2LE(lfhBuf + kLFHGPBFlags);

2. if ((gpbf & kGPFUnsupportedMask) != 0) {

3. ALOGW("Invalid General Purpose Bit Flag: %d", gpbf);

4. return false;

5. }
```

# Manifest Cheating

`AndroidManifest.xml` 是 Android 应用程序的配置文件，包含了包名、应用程序名称、申请的权限信息以及组件信息等。在 Android 应用程序开发，生成 APK 时，`aapt` 会负责完成资源的打包，打包会将文本格式的 XML 资源文件编译成二进制格式的 XML 资源文件。将文本格式的 XML 文件转换成二进制格式，一方面通过字符串资源池的统一管理，减少文件体积；另一方面二进制格式的 XML 文件解析速度也会更快。在 Android 开发过程中，生成的 `R.java` 文件中包含了相应的资源类型、名称以及对应的 id 值。资源 id 是 32bit 的整型值，格式为:0xPPTTNNNN。其中 PP 表示使用该资源的包，TT 代表该资源的类型，而 NNNN 是该类型中资源的名称。对于应用程序资源，PP 值固定为 7f，而对于被引用的系统资源包，其 PP 值为 01。TT 和 NNNN 一般是 `aapt` 按照资源出现的顺序生成的。更多分析可以参考罗升阳的 Android 应用程序资源的编译和打包过程分析。

`Manifest Cheating` 的基本原理是，在 AndroidManifest 的`<application>`节点中插入一个未知 id(如 `0x0`)，名称为 `name` 的属性，其值可以是一个从未定义实现的 Java 类文件名。而对 AndroidManifest 的修改需要在二进制格式下进行，这样才能不会破坏之前 `aapt` 对资源文件的处理。由于是未知的资源 id，在应用程序运行过程中，Android 会忽略此属性。但是在使用 `apktool` 进行重打包时，首先会将 `AndroidManifest.xml` 转换为明文，进而会包含名称为 `name` 的属性，而相应的 id 信息会丢失，`apktool` 重打包会重新进行资源打包处理，由于该 `name` 属性值是一个未实现的 Java 类，重打包后的应用程序在运行过程中，由于 `application` 节点中定义的类是先于所有其他组件运行的，若系统找不到对应的类，会出现运行时错误，Dalvik 虚拟机会直接关闭。另外，也可以实现 `name` 属性值对应的 Java 类，若此类被调用，则表明被重打包了，可以采取进一步的措施。这样就可以起到保护自身 APK 的作用，防止被重打包。但是这种方法也很容易被绕过，只需要在经过 `apktool` 解码的 `AndroidManifest` 文件中，去掉在 `application` 节点中添加的 `name` 属性即可。整个过程如下：

- 将 APK 解压缩，提取其中的 `AndroidManifest.xml` 文件；
- 使用 `axml` 工具，修改二进制的 `AndroidManifest.xml` 文件，在 `application` 节点下插入 id 未知(如 `0x0`)，名为 `name` 的属性(值可以任意，只要不对应到项目中的类文件名即可，如 `some.class`)；
