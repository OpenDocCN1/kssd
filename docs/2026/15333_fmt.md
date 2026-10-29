
# [原创]什么值得买｜ JEB Pro 3.12.0

自从 2019 年圣诞节有人放出了 JEB 3.7.0 破解版本，咸鱼了小半年的 JEB 终于开始发力更新了。。。

2020 年 1 月 10 号，JEB 发布 3.12.0，以及《JEB Android Updates – Lambda Recovery, Unreflecting Code, Generic String Decryption, and More》

ChangeLog:

```text
Important additions to the Dalvik/DEX decompiler; Added support for Java bytecode decompilation.

DEX Decompiler:
- Generic deobfuscator: attempt to perform string decryption and replace method calls by strings (see blog post)
- Optimizer: unreflect dynamic code (replace invocations-via-reflection code by static code, and clean-up)
- Optimizer: improved array inlining
- Other optimizers: additions and upgrades across the board
- Lambda reconstructor: a bug fix

Other Additions:
- Support for Java bytecode decompilation: single classfile, jar archives.
  (Java bytecode is converted to DEX and processed by DEX decompiler.)
- Upgrades related to DEX handling and rendering in the UI client.
```

JEB 最近的更新基本上都算回归正道，发力在 `Dex Decompiler` 的优化上，如今 JEB 的 DEX 反编译效果，代码可读性也比以前强了不少。再加上一些独有的优化，相信以后会打破 Dex 反编译不如 Jadx，Native 反编译不如 IDA 的局面。

这次虽然不是架构或者核心上的大更新，但是可以看到多出了一些比较实用功能。

## Lambda 的支持

JEB 虽然在 3.10.0 版本就开始支持在反编译时重建 `Lambda` 语法，不过这次是第一次公开写在 blog 里，也算是介绍新特性。 开源反编译器 `Jadx 1.1` 目前还没有跟上这个步伐, 可以看看他们俩的对比

源码:

```java
public void lTest() {
    View e = findViewById(R.id.view);
    e.setOnClickListener((View arg1) -> Toast.makeText(MainActivity.this, "lambda test", Toast.LENGTH_SHORT).show());
}
```

带 `->` 的那条语句就是 `lamdba` 表达式，作用相当于新建一个匿名类再实现对应方法，节省了好几行代码。

JADX:

![](https://bbs.kanxue.com/upload/attach/202002/715510_S7KSMCTKURZQ6Q8.png)

Jadx 直接表现为一个匿名类。不过其实 Jadx 也算还比较好看了，

不信你看 CFR:

![](https://bbs.kanxue.com/upload/attach/202002/715510_66TDAXXAC7D9KKK.png)

嗯.. 打扰了, 不过这个不能作为对比依据，因为 `CFR` 是 `Java Decompiler`, 是先要做一道 `dex2jar` 然后才做的反编译

所以看看`JEB 3.12`:

![](https://bbs.kanxue.com/upload/attach/202002/715510_P3XA2KEJNRQJTMR.png)

emm, 还不错。 值得一提的是，有人可能觉得 JEB 没有像 Jadx 一样自动替换常量的功能，导致像 `0x7F070090` 这样的资源 ID 看起来很不友好， 不过其实你按 tab 键，切换到汇编视图，也能看到 JEB 帮你做好的资源常量注释，能让你知道这个 ID 代表的资源。

![](https://bbs.kanxue.com/upload/attach/202002/715510_JNRC8K7EWFVFMDP.png)

还有 `setContentView` 的时候，也会注释一个 `layout` 的注释

![](https://bbs.kanxue.com/upload/attach/202002/715510_K852KEH9G7Q2SSN.png)

可以看到上面还有覆写的注释。还可以对资源 ID 进行交叉引用，将光标点到资源 ID，然后:

> 文件 -> 脚本 -> Registered -> `AndroidXrefResId`

这是 JEB 自带的脚本实例中的一个，结果如下：

![](https://bbs.kanxue.com/upload/attach/202002/715510_KC2EQGUUEWS2W6Q.png)

这些都是 JEB3 以来不断优化的细节。

## "反"-反射

这个功能很易懂，就是将反射调用优化成正常的调用语句或者字段访问、赋值语句。

具体原理在 2013 年 JEB 发表过的《Decompiled Java Code Manipulation using JEB API – Part 3: Defeating Reflection》中，当时还是 JEB1 的时代，现在都快 JEB4 了，JEB 才把这个功能内嵌到 JEB3 里面，事实证明，一款软件不被破解就永远不知道自己还有多少功能需要更新！(hhh, 开玩笑，请多多支持正版。)

效果是这样子的

源码:

```java
try {
    Class.forName("com.example.sedemo.MainActivity")
            .getMethod("rTest").invoke(null);
} catch (Exception ex) {
    Log.d("TAG", "on except");
}

public static void rTest() {
    Log.d("atg", "rTest: ");
}
```

效果:

![](https://bbs.kanxue.com/upload/attach/202002/715510_DT92M94KFKD8XVS.png)

也有不足的点，

1. 不会消除掉 `catch`，因为有可能造成反编译歧义。

2. 对于非 `Android SDK`、`Java API` 以及已经声明的的方法，不会进行这个优化，比如把方法名改成 `rTest1`，就会变成原来的样子。

![](https://bbs.kanxue.com/upload/attach/202002/715510_57R3X22GKDPKZFU.png)

## 通用字符串解混淆

这个东西我研究了半天也没知道是怎么触发的，好像也没有开关。大概就是依赖 `JEB` 写的 `ast.emulator` 来动态的解密字符串并且把解密 Call 替换为常量。

虽然很实用，不用以后每次都手动写解密脚本，然鹅没学会咋用，估计还没完善吧。放个官方效果图

Before:

![](https://bbs.kanxue.com/upload/attach/202002/715510_JSURRH742VNQ3VU.png)

After:

![](https://bbs.kanxue.com/upload/attach/202002/715510_RGBWF9A2TDMPGSR.png)

## Java 反编译器

JEB 之前是支持的一直都是 DEX 反编译，这次也支持 `Java` 了，也就是说可以反编译 `class` 文件。

从描述上看，是使用 `dx` 将 `java bytecode` 转成 `dex`，再反编译之，然并卵。

## PS

反编译效果还是有很多地方不如 `Jadx`，比如对 `switch` 的优化好像就不太友好，然并卵的功能很多，不如先专心做好 `Dex Decompiler`。。。

不过架不住比 IDA 便宜 N 倍，800 块钱的 `Jeb Android` 值得(?)拥有。
