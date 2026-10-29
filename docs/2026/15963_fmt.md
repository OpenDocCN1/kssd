
# [原创] 定制 Xposed 框架（干货）

由于很多 APP 开始针对 XP 做对应的检测，如微信、支付宝、钉钉等，所以使用定制化的方式隐藏框架。

## 常见检测方法

1. 由于 Xposed 的 Hook 是通过修改被 Hook 方法的 `so`，使其变为 `native` 来实现的，所以检测方也可以通过检测方法是否变成了 `native` 来达到检测的目的。例如，`Throwable.class.getDeclaredMethod("getStackTrace")` 是否为 Native 方法。

2. 通过 `StackTraceElement.getClassName()` 检测是否存在以下类名：

   - `de.robv.android.xposed`
   - `com.android.internal.os.ZygoteInit`
   - `com.saurik.substrate`

   通过 `getMethodName()` 检测以下方法：

   - `handleHookedMethod`
   - `main`
   - `invoked`

3. 检查 `XposedHelper` 的成员：

   - `fieldCache`
   - `methodCache`
   - `constructorCache`

4. 检查 Xposed 的文件。

5. 检查安装包。

6. 检查 `/system/lib` 是否有 Xposed 文件：

   - `/system/lib/libxposed_art.so`
   - `/system/lib64/libxposed_art.so`

7. 检查 `/system/framework/XposedBridge.jar`。

8. 通过代码抛出一个异常，在堆栈中检查是否含有 `de.robv.android.xposed.XposedBridge` 之类的内容。

   检查 `com.android.internal.os.ZygoteInit` 是否出现了两次。

9. 检查环境变量 `CLASSPATH=/system/framework/XposedBridge.jar`。

10. 检查 `/proc/self/maps`。通过读取 Shell 命令 `/proc/pid（应用进程 ID）/maps`，可以拿到当前上下文的 `so` 和 `jar` 列表，查找 Xposed 相关内容。

11. 通过 `loadClass` 检测：

    - `de.robv.android.xposed.XposedHelpers`
    - `de.robv.android.xposed.XposedBridge`

## 几种隐藏方案

1. **Java 层检测 XP 特征值**

   可以根据应用获取 XP 特征值的方法，定制 Hook 进行修改隐藏，或者通过通用性 Hook 代码解决。但这只限于 Java 层，在 `so` 中读取 XP 文件时就无效了。

2. **`so` 层检测**

   如果能逆向代码，在 Java 层中找到上述方法，也可以通过 Hook 形式隐藏。缺点是每个 APP 都得找，而且检测代码一变就得改，极其不灵活。

3. **定制化 XP 框架**

   修改所有特征值，也可以达到隐藏的目的。

定制 XP 是最好的解决办法。网络上有一些相关资料，但都没有干货或者不全。网上暴露出来的基本上只教了 `XposedInstaller`、`XposedBridge` 的修改，往下到 `native` 层以及 `XposedTools` 的修改资料几乎没有，或者说跟没有差不多，所以有了这篇文章。

在开始之前，你必须去按照这篇博客 [点这里](https://bbs.kanxue.com/elink@dc7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5@1y4K6q4T1j5h3t1@1z5h3y4T1x3b7`.`.)，先去编译 Android 源码之后编译下官方的 XP（坑很多，要配的环境非常多，要有足够的耐心）。

完成上一步后就可以开始进行对应的修改。

## 1. XposedInstaller.apk

负责安装 XP 环境，其实就是下载与手机 CPU 架构、SDK 对应的 ZIP，然后刷入手机和管理模块。

环境：Android Studio，网络对应的资料都有，我就随便写写。

- 包名：随便改一个，AS 直接改就行。
- 名称：随便改一个。

![](https://bbs.kanxue.com/upload/attach/201911/735880_JXBQXMDP26NZRGV.jpg)

## 2. XposedBridge-art.jar

给开发者提供对应的 API 支持，底层实现 JNI 调用 `libxposed-art.so` 的 `native` 方法。

环境：Android Studio，网络上也有资料，随便写写。

- 原始包名：`de.robv.android.xposed`，随便改一个。
- 原始名称：`XposedBridge.jar`，随便改一个。

![](https://bbs.kanxue.com/upload/attach/201911/735880_8BCW2C2KMXDX886.jpg)

## 3. Xposed

给 `XpBridge` 提供 API 支持，调用了很多 Android ART 的方法，以及最重要的替换 Android Zygote（`app_process`）。

![](https://bbs.kanxue.com/upload/attach/201911/735880_DXZ8N7H2WUU94T2.jpg)

![](https://bbs.kanxue.com/upload/attach/201911/735880_2VR78ZH6EHQ5QKE.jpg)

![](https://bbs.kanxue.com/upload/attach/201911/735880_KZ4BYQTGGCXSU2V.jpg)

## 4. Xposed-Tools

XP 框架的编译器，打包出可以刷机的 ZIP。以下两个是打包脚本和刷机包脚本，文件内容改成你对应的就好了。这点网上根本没有资料，全靠自己一个文件一个文件看，巨坑。

如果有源码不准备刷机安装 XP，单独改 `build.pl` 就好了。

`flash-script.sh` 不改，刷机就会报错，缺少 `xposed.prop`。

![](https://bbs.kanxue.com/upload/attach/201911/735880_JKAQPRJGYY3UA25.jpg)

![](https://bbs.kanxue.com/upload/attach/201911/735880_R2KTEV6BXKTFXUE.jpg)

![](https://bbs.kanxue.com/upload/attach/201911/735880_U84Y7VHE3RDTSYV.jpg)

### 思维导图

![](https://bbs.kanxue.com/upload/attach/201911/735880_F6CC89J7APXSPXH.jpg)

## 5. 编译与运行

把 Xposed 项目复制到 Android 源码的 `frameworks/base/cmds` 下。

再去下载与你编译的 Android 源码对应的 Xposed 版 ART，进行替换。

`xposedtools` 开始编译，静静等待就好了。我这里编译了 Android 5.1、Android 6.1，生成了 ARM、ARM64、x86 手机可用的刷机包。

![](https://bbs.kanxue.com/upload/attach/201911/735880_SK3Q6MJAXQQFFXR.jpg)

数据线连接手机，把对应的刷机包推送到手机里：

```bash
adb reboot recovery
```

选择刷机包安装即可，手机重启后即可看到。

注意，要使用定制版本的 XP，项目里的 `jar` 包一定得是你定制后的，其他不用动。

![](https://bbs.kanxue.com/upload/attach/201911/735880_PVQ8A3FE9GDJUNQ.jpg)

![](https://bbs.kanxue.com/upload/attach/201911/735880_EMFATQZ9R5ABZPH.jpg)

## 总结

得亏在改之前认真复习了一遍 Zygote 和 Xposed 框架的源码。由于网上资料过少，经历了不少坑，很多都是自己摸索出来的。我觉得应该有不少人有这种需求，所以写了这篇文章，希望能够帮助到你。有任何问题可以私信我。

在 CSDN 写的，原文可能看起来舒服点：[传送门](https://bbs.kanxue.com/elink@a7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8U0j5@1y4K6q4T1j5h3t1@1z5h3y4T1x3b7`.`.)

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
