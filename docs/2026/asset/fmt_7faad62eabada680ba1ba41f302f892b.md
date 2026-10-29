
`.``class` `public abstract Lcom``/``happy``/``learnsmali``/``BaseActivity;` `/``/` `.``class` `表示类路径 包名``+``类名`  `.``super` `Landroidx``/``appcompat``/``app``/``AppCompatActivity;` `/``/` `.``super` `表示父类的路径`  `.source` `"BaseActivity.kt"`                                   `/``/` `表示源码文件名`    `# interfaces`  `.implements Lcom``/``happy``/``learnsmali``/``action``/``ActivityAction;`  `.implements Lcom``/``happy``/``learnsmali``/``action``/``ClickAction;`  `.implements Lcom``/``happy``/``learnsmali``/``action``/``HandlerAction;`  `.implements Lcom``/``happy``/``learnsmali``/``action``/``BundleAction;`  `.implements Lcom``/``happy``/``learnsmali``/``action``/``KeyboardAction;`      `# annotations`  `.annotation system Ldalvik``/``annotation``/``MemberClasses;`  `value` `=` `{`  `Lcom``/``happy``/``learnsmali``/``BaseActivity$Companion;,`  `Lcom``/``happy``/``learnsmali``/``BaseActivity$OnActivityCallback;`  `}`  `.end annotation`

**1-3行定义基本信息**：表示有源文件 `BaseActivity.kt` 反编译得到的 smali 文件（第三行），文件路径位于 `com/happy/learnsmali/`（第二行），继承于 `androidx/appcompat/app/AppCompatActivity`（第三行）。

**5-9行定义接口信息**：表示 `BaseActivity` 类实现的接口类有：

- `com/happy/learnsmali/action/ActivityAction`
- `com/happy/learnsmali/action/ClickAction`
- `com/happy/learnsmali/action/HandlerAction`
- `com/happy/learnsmali/action/BundleAction`
- `com/happy/learnsmali/action/KeyboardAction`

**11-16行定义内部类**：表示 `BaseActivity` 类有两个内部类 -- `Companion` 和 `OnActivityCallback`。

分析完 smali 开头的文件信息，我们可以据此可以构造出 java 代码：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `class` `BaseActivity extends AppCompatActivity`  `implements ActivityAction, ClickAction, HandlerAction, BundleAction, KeyboardAction {`    `class` `Companion {`  `/``/` `...`  `}`    `class` `OnActivityCallback {`  `/``/` `...`  `}`  `}` |

**其他方法**

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `# virtual methods   //Representation is a virtual method`  `.method protected onCreate(Landroid``/``os``/``Bundle;)V`  `.``locals` `1`  `.param p1,` `"savedInstanceState"`    `# Landroid/os/Bundle;`    `.line` `10`  `invoke``-``super` `{p0, p1}, Landroid``/``app``/``Activity;``-``>onCreate(Landroid``/``os``/``Bundle;)V`    `.line` `11`  `const``/``high16 v0,` `0x7f050000`    `invoke``-``virtual {p0, v0}, Lcom``/``justart``/``samlidemo``/``MainActivity;``-``>setContentView(I)V`    `.line` `12`  `return``-``void`  `.end method` |

- 方法以 `.method` 开始，以 `.end method` 结束；
- 位于第一行的最后 `V` 表示返回类型为 void；
- 方法参数 `Landroid/os/Bundle;` 表示方法 `onCreate()` 的参数为 `Bundle` 类型；
- `. param` 表示方法的参数名称为 `savedInstanceState`;
- 最后 `return-void` 表示返回的值类型为 void；

### 数据类型

- byte：`B`
- char：`C`
- double：`D`
- float：`F`
- int：`I`
- long：`J`
- short：`S`
- void：`V`
- boolean：`Z`
- array：`[XXX`
- Object：`Lxxx/yyy`

相信有 JNI 基础会对上面的数据类型好明白，这里解析上面的最后两项：

**array：`[XXX`**

在基础类型前加 `[` 表示数组类型，例如 int 数组和 byte 数组为 `[I`、`[B`。

**Object：`Lxxx/yyy`**

以 `L` 开头的类型表示为对象，如 `String` 对象对应表示为 `Ljava/lang/String;` (对象类型需要后面跟分号)，其中 `java/lang` 表示 `java.lang` 包，`String` 表示该包路径下的一个对象。

这里可能会有童鞋有疑惑，如果类是使用 `Ljava/lang/String;` 来表示，那么内部类又应该在 smali 中如何定义呢？可能使用过 Java 反射的童鞋脑海里面闪过 `$` 符号。是的，在 smali 语法中同样是使用 `Ljava/lang/String$xxx;` 来表示 `xxx` 是 `String` 类的内部类。

### 寄存器

Dalvik VM 与 JVM 最大的区别之一就是 Dalvik VM 是基于寄存器的。基于寄存器是什么意思呢？个人理解的是有点类似于汇编语言，通过寄存器来存储数据、传递数据。在 smali 中本地寄存器用 `v` 开头的字母 + 数字来表示，如 `v0`、`v1`、`v2` 、...，而参数寄存器则使用 `p` 开头 + 数字来表示，如 `p1`、`p2`、`p3` 、...。特别注意的是，`p0` 参数寄存器不一定是表示第一个参数，在非 static 函数中，`p0` 表示 `this`，`p1` 则表示第一个参数，`p2` 表示函数中的第二个参数。而在 static 函数中 `p0` 则才对应第一个参数（**因为 Java 的 static 的方法没有对象的概念**）。本地寄存器没有限制，理论上是可以任意使用的。

### 成员变量

下面继续介绍有关成员变量的内容：

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `# static field`  `.field private static final PREFS_INSTALLATION_ID:Ljava``/``lang``/``String;` `=` `"installationId"`  `/``/``...`    `# instance field`  `.field private _activityPackageName:Ljava``/``lang``/``String;` |

上面定义的 static field 和 instance field 均为成员变量，格式是：

**.field pubilc/private [static] [final] varName:<类型>**

static field 和 instance field 虽然均为成员变量，但它们还是存在区别的。当然最明显的区别就是是否与对象相关，static field 是类层面的概念，而 instance field 是对象层面的概念。

出现成员变量，那就意味着有变量的赋值与取值。在 smali 语法中，取值指令有：`iget`、`sget`、`iget-boolean`、`sget-boolean`、`iget-object`、`sget-object` 等，而赋值指令有：`iput`、`sput`、`iput-boolean`、`sput-boolean`、`iput-object`、`sput-object` 等。

> `iget` / `iput` 分别表示 instance field 成员变量的取值和赋值；
>
> `sget` / `sput` 分别表示 static field 成员变量的取值和赋值；
>
> **是否为 instance field 还是 static field 成员的取值和赋值指令，根据指令前缀判断即可**。带 `-object` 后缀表示操作的是成员变量是对象类型，而不带该后缀则表示操作的是基本数据类型。特别地，boolean 基本数据类型使用带 `-boolean` 后缀。

下面有个例子：

|  |  |

| --- | --- |

| 1  2 | `const``/``4` `v0,` `0x0`  `iput``-``boolean v0, p0, Lcom``/``disney``/``xx``/``XxActivity;``-``>isRunning:Z` |

在上面的例子中，使用了 `v0` 本地寄存器，并且把 `0x0` 传递到 `v0` 本地寄存器，然后第二句使用 `iput-boolean` 指令把 `v0` 寄存器中的值传递到 `com.disney.xx.XxActivity` 的成员变量 `isRunning`。即相当于：`this.isRunning = false;`（上面提到，在非 static 函数中 `p0` 表示为 `this` ，在这里则表示为 `com.disney.xx.XxActivity` 的对象实例）。

**static field 成员变量**

|  |  |

| --- | --- |

| 1 | `sget``-``object` `v0, Lcom``/``disney``/``xx``/``XxActivity;``-``>PREFS_INSTALLATION_ID:Ljava``/``lang``/``String;` |

操作指令 `sget-object` 是用来获取静态成员变量并保存在紧接的本地参数列表中。在这里，把位于 `com.disney.xx.XxActivity` 类中的静态成员 `PREFS_INSTALLATION_ID` 的值传递给本地寄存器 `v0`。

**instance field 成员变量**

|  |  |

| --- | --- |

| 1 | `iget``-``object` `v0, p0, Lcom``/``disney``/``xx``/``XxActivity;``-``>_view:Lcom``/``disney``/``common``/``WMWView;` |

操作指令 `iget-object` 也是用来获取类成员变量并保存在紧接的本地参数列表中。这里把 `com.disney.xx.XxActivity` 类中的对象成员 `_view` 赋值给本地寄存器 `v0` 中。

通过观察上面的 **static field 静态成员变量** 和 **instance field 类成员变量**，可以总结出以下的格式：

**<opcode> <本地寄存器>, [<参数寄存器>], <变量所属的类变量> ->varName:<变量类型>**

put 指令和上面提到的 get 指令格式是类似的，这里可以直接通过看下面的例子：

|  |  |

| --- | --- |

| 1  2 | `const``/``4` `v3,` `0x0`  `sput``-``object` `v3, p0, Lcom``/``disney``/``xx``/``XxActivity;``-``>globalIapHandler:Lcom``/``disney``/``config``/``GlobalPurchaseHandler;` |

Java 代码表示： `this.globalIapHandler = null;` (null = `0x0`)

|  |  |

| --- | --- |

| 1  2  3 | `.local v0, wait:Landroid``/``os``/``Message;`  `const``/``4` `v1,` `0x2`  `iput v1, v0, Landroid``/``os``/``Message;``-``>what:I` |

Java 代码表示： `wait.what = 0x2;`（wait 是 `Message` 的实例）

### 函数调用

函数定义的格式：

**function (type1type2type3...)RetValue**

需要注意的是函数的参数类型需要定义为 smali 语法中的类型，同时参数之间不可以有其他的分隔符，例子如下：

**helloSmali ()V**

表示 `void helloSmali()`

**helloSmali ([BI)Z**

表示 `boolean helloSmali(byte[], int)`

**helloSmali (ZLjava/lang/String;[I[I)V**

表示 `void helloSmali(boolean, String, int[], int[])`

在 smali 中函数和成员变量一样也分为两种类型，但不同于成员变量中的 **static field 静态成员变量** 和 **instance field 类成员变量**，函数中的是 **direct method** 和 **virtual method**。那么函数的 **direct method** 和 **virtual method** 有什么区别呢？简单来说，**direct method** 就是 private 函数，而 **virtual method** 则是 public 和 protect 函数。

所以在调用函数的时候，有 `invoke-direct`、`invoke-virtual`，另外还有 `invoke-static`、`invoke-super` 以及 `invoke-interface` 等几种不同的指令。同时还存在着 `invoke-XXX/range` 指令，这是参数传参个数大于 4 个的时候调用的指令。

**invoke-static**

|  |  |

| --- | --- |

| 1 | `invoke``-``static {}, Lcom``/``disney``/``xx``/``UnlockHelper;``-``>unlockCrankypack()Z` |

`invoke-static` 表示调用的是类静态函数。Java 代码表示为：`UnlockHelper.unlockCrankypack()`，这里注意到 **invoke-static 后紧接着 `{}`，表示的是调用该方法的实例 + 参数列表**，由于这个方法既不需要参数，也是类静态方法，所以 `{}` 内为空，再看一个例子：

|  |  |

| --- | --- |

| 1  2 | `const``-``string v0,` `"fmodex"`  `invoke``-``static {v0}, Ljava``/``lang``/``System;``-``>loadLibrary(Ljava``/``lang``/``String;)V` |

这里调用的是 `static void System.loadLibrary(String)` 来加载 so 库，而 `v0` 则表示传参 `fmodex`。

**invoke-super**

表示调用父类方法用的指令，在重载的方法都可以看到。

**invoke-direct**

表示调用 private 函数的方法，如：

|  |  |

| --- | --- |

| 1 | `invoke``-``direct {p0}, Lcom``/``disney``/``xx``/``XxActivity;``-``>getGlobalIapHandler()Lcom``/``disney``/``config``/``GlobalPurchaseHandler;` |

这里的 `GlobalPurchaseHandler getGlobalIapHandler()` 表示 `getGlobalIapHandler()` 是定义在 `XxActivity` 类中，权限为 private 的方法。

**invoke-virtual**

表示调用的是 protected 或 public 函数。

|  |  |

| --- | --- |
