
![](upload/attach/202002/811277_ETDGZN8Z8DYG5E8.png)

## 总结

上篇我们主要讲了 `robust` 的原理，并采用第一种方法——`Robust` 自带的 `EnhancedRobustUtils` 工具类来 hook，得到我们想要的答案。从做题的角度来说，这是一种很好很快的办法；但是从学习的角度，可能这道题用 hook `DexClassLoader` 的方式更有趣味和意义。下篇我会详细介绍怎么 hook `DexClassLoader` 动态加载的类及方法，来获得最终答案：D

前段时间看到有朋友在问怎么使用 `frida` 去 hook 动态加载的 dex 之类的问题。确实，关于如何 hook 动态加载的 dex，网上的资料很少，更不用说怎么使用 `frida` 去 hook 动态加载的 dex 了。（`frida` 的官方文档已经无力吐槽……）后来偶然的情况下发现了一道 CTF 题目可以作为很好的案例，所以花了很多心思将文章写下来分享给大家。涉及的内容比较多，我打算用上下篇将 `frida` hook 动态加载的 dex 的方法讲清楚，上篇主要是引导及一些前置的知识。

## 目录

- [文章涉及内容及使用到的工具](https://bbs.kanxue.com/#文章涉及内容及使用到的工具)
  - 0x00 使用到的工具
  - 0x01 涉及知识点
- [apk的安装和分析](https://bbs.kanxue.com/#apk的安装和分析)
  - 0x00 apk安装及使用
  - 0x01 静态代码分析
  - 0x02 Robust热修复框架原理
- [JavaScript代码构造](https://bbs.kanxue.com/#javascript代码构造)
  - 0x00 hook点分析
  - 0x01 hook代码构造
  - 0x02 执行py脚本获取结果
- [总结](https://bbs.kanxue.com/#总结)

---

博客同步：[访问](https://bbs.kanxue.com/elink@823K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6G2N6i4W2#2j5h3&6^5j5#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6l9&6x3o6l9^5x3o6R3`.)（一些 web 安全的研究会直接发布到这上面）

## 文章涉及内容及使用到的工具

### 0x00 使用到的工具

- ADT（Android Developer Tools）
- Jadx-gui
- JEB
- `frida`
- `apktool`
- 010 editor
- 天天模拟器（genymotion，实体机等亦可）

### 0x01 涉及知识点

- Robust 热修复框架原理
- Java 反射
- Robust 类 hook
- `frida` 基础 hook

## apk的安装和分析

> 文章使用的是 DDCTF2018 的 Android 逆向第二题 Hello Baby Dex。

示例地址：[下载](https://bbs.kanxue.com/elink@27fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9r3!0K6N6r3#2S2P5X3g2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1f1X3g2$3k6i4u0K6k6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6t1k6h3I4D9L8#2)9J5y4e0t1H3b7X3q4T1P5g2)9J5y4e0t1H3c8r3g2^5i4K6u0r3j5i4m8H3i4K6u0V1M7X3g2D9k6h3q4K6k6g2)9J5k6h3q4H3K9H3`.`.)

### 0x00 apk安装及使用

程序功能很简单，就是输入密码并验证，错误会 Toast 出一些信息。按这个惯性，可能得输入某些正确的值才能获取 flag 吧。废话不多说，直接反编译看看。

![](https://bbs.kanxue.com/upload/attach/202002/811277_Z2BHGRU6XFQY9G8.jpg)

### 0x01 静态代码分析

一般情况下，我是直接扔到 `jadx` 中去看反编译后的代码，可是这次 `jadx` 反编译出来的结果有很多错误，这里我就不贴图了，大家可以尝试一下，看看是什么错误。后面放到 `jeb` 工具中，便没有报错。所以查看反编译的效果时，大家可以多尝试几个反编译器对比效果。查看 `AndroidManifest.xml`，找到了程序的入口 `MainActivity`。

```xml
<activity android:name="cn.chaitin.geektan.crackme.MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

在此同时，在 `cmd` 中通过 `apktool d [apk]`，再将 apk 文件反编译出来。

![](https://bbs.kanxue.com/upload/attach/202002/811277_EX93PP8AVARBDQ4.png)

接下来直接定位到 `MainActivity` 的 `onCreate()` 方法，可以看到代码是混淆过的，阅读上稍微有点小困难。但是在 `onCreate()` 方法中，我们还是可以发现一些可疑的方法及变量，比如 `PatchProxy`、`changeQuickRedirect` 等。

![](https://bbs.kanxue.com/upload/attach/202002/811277_Y3CG9MMGCEZU5Q9.jpg)

继续搜索有用的信息，我们可以发现在 `onCreate()` 方法的 `else` 逻辑中调用了 `this.runRobust()`，并且我们发现在类中每一个方法里面都会存在一些 `changeQuickRedirect` 变量，以及 `isSupport()`、`accessDispatch()` 方法。

![](https://bbs.kanxue.com/upload/attach/202002/811277_8CQCKZNY9MCR8TJ.png)

上面的先放一边，我们来看看 `runRobust()` 方法中写了什么。在 `else` 条件语句中可以看到它实例化了一些对象。

```java
private void runRobust() {
    // 固定格式的 changeQuickRedirect、isSupport、accessDispatch
    int v4 = 4;

    Object[] v0 = new Object[0];

    ChangeQuickRedirect v2 = MainActivity.changeQuickRedirect;

    Class[] v5 = new Class[0];

    Class v6 = Void.TYPE;

    MainActivity v1 = this;

    boolean v0_1 = PatchProxy.isSupport(v0, v1, v2, false, v4, v5, v6);

    if (v0_1) {
        v0 = new Object[0];

        v2 = MainActivity.changeQuickRedirect;

        v5 = new Class[0];

        v6 = Void.TYPE;

        v1 = this;

        PatchProxy.accessDispatch(v0, v1, v2, false, v4, v5, v6);
    } else {
        Context v1_1 = this.getApplicationContext();

        // 实例化 PatchManipulateImp 类
        PatchManipulateImp v2_1 = new PatchManipulateImp();

        // 以及实例化 PatchExecutor 类
        PatchExecutor v0_2 = new PatchExecutor(
            v1_1,
            ((PatchManipulate) v2_1),
            new GeekTanCallBack()
        );

        v0_2.start();
    }
}
```

跟进 `PatchManipulateImp` 类，可以发现在 `fetchPatchList` 方法中，它调用了 `getAssets().open("GeekTan.BMP");`，从资源文件夹中加载了一个 BMP 图片文件，并将这个 BMP 文件内容写入 `GeekTan.jar` 中。

![](https://bbs.kanxue.com/upload/attach/202002/811277_YWNM8HTZJ76ES85.png)

具体代码如下：

```java
try {
    v10 = arg17.getAssets().open("GeekTan.BMP");

    v8 = new File(
        arg17.getCacheDir() + File.separator + "GeekTan" + File.separator + "GeekTan.jar"
    );

    if (!v8.getParentFile().exists()) {
        v8.getParentFile().mkdirs();
    }
} catch (Exception v9) {
    goto label_171;
}

// 将 v8 通过 FileOutputStream 方法赋值 v12
try {
    v12 = new FileOutputStream(v8);
} catch (Throwable v0_3) {
    goto label_17;
}

int v0_4 = 1024;

try {
    byte[] v7 = new byte[v0_4];

    while (true) {
        // v10 是 GeekTan.BMP 赋值 v11
        int v11 = v10.read(v7);

        if (v11 <= 0) {
            break;
        }

        // 最终写入到 v12 中，而 v12 是 new FileOutputStream(v8)
        ((OutputStream) v12).write(v7, 0, v11);
    }
} catch (Throwable v0_3) {
    goto label_88;
}
```

显然这个 BMP 文件很可疑，我们放到 010 editor 中看看二进制的内容，发现它真正的文件格式是一个压缩包，并且可以直观地看到在压缩包中存在一个 dex 文件。

![](https://bbs.kanxue.com/upload/attach/202002/811277_8E5BMZE4KX3HTY3.png)

同时我们还可以发现，`fetchPatchList()` 方法中实例化了一个 `Patch` 对象。

```java
Patch v13 = new Patch();
```

并在 `fetchPatchList()` 方法的最后，调用：

```java
v0_6 = "cn.chaitin.geektan.crackme.PatchesInfoImpl";

v13.setPatchesInfoImplClassFullName(v0_6);
```

回到 `runRobust()`，接下来实例化了 `PatchExecutor` 类，可以看到第二个参数即为 `PatchManipulateImp` 类的实例。

```java
Context v1_1 = this.getApplicationContext();

PatchManipulateImp v2_1 = new PatchManipulateImp();

PatchExecutor v0_2 = new PatchExecutor(
    v1_1,
    ((PatchManipulate) v2_1),
    new GeekTanCallBack()
);
```

这个 `PatchExecutor` 类是在 `com.meituan.robust` 包下的，这是一个第三方的包，目前官方已经将其开源。因为直接看混淆的代码还是比较费时的，在此**暂停**上面的分析，简单学习一下 Robust。

### 0x02 Robust热修复框架原理

我们先简单了解一下 Robust 是什么。Robust 是美团出的一款热修复框架，可以在 GitHub 上面[下载](https://bbs.kanxue.com/elink@d43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4k6h3W2@1N6h3q4F1i4K6u0V1c8r3W2S2L8Y4m8A6L8X3N6Q4x3V1k6d9L8$3u0#2M7%4b7`.) 它的最新源码。

Robust 的基本原理，我们主要了解这 4 个步骤就能清晰明白了。

1. 将 apk 代码中每个函数都在编译打包阶段自动地插入一段代码。

例如，原函数：

```java
public long getIndex() {
    return 100;
}
```

它会被处理成这样：

```java
// 在该类中声明一个接口变量 changeQuickRedirect
public static ChangeQuickRedirect changeQuickRedirect;

// 在要修复的方法中添加以下逻辑代码
public long getIndex() {
    if (changeQuickRedirect != null) {
        // PatchProxy 中封装了获取当前 className 和 methodName 的逻辑，
        // 并在其内部最终调用了 changeQuickRedirect 的对应函数
        if (PatchProxy.isSupport(
                new Object[0],
                this,
                changeQuickRedirect,
                false
        )) {
            return ((Long) PatchProxy.accessDispatch(
                new Object[0],
                this,
                changeQuickRedirect,
                false
            )).longValue();
        }
    }

    return 100L;
}
```

可以看到，Robust 为每个 class 增加了一个类型为 `ChangeQuickRedirect` 的静态成员，而在每个方法前都插入了使用 `changeQuickRedirect` 相关的逻辑。当 `changeQuickRedirect` 不为 null 时，会执行到 `accessDispatch` 方法，从而替换掉之前老的逻辑，达到修复的目的。

2. 生成需要修复的类及方法的类文件并打包成 dex。

接下来你可能已经将需要修复的类及方法写好了，这个时候调用 Robust 的 autopatch 文件夹中的类及方法会生成如下主要文件：`PatchesInfoImpl.java`、`xxxPatchControl.java`（其中 xxx 为原类的名字）。

`PatchesInfoImpl.java` 的内容是由 `PatchesInfoFactory` 类的 `createPatchesInfoClass` 生成的，这是它生成 `PatchesInfoImpl` 逻辑。可以看到，这个类其实是用拼接得到的。
