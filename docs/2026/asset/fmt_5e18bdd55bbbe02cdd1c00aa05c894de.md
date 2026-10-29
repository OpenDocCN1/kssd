
该处代码，是demo中`ActivityThread`某静态函数的hook示例，`callOrigin`的时候第一个参数必须显式传`null`

```Java
@HookName("android.app.ActivityThread")
public class ActivityThread {
private static final String TAG = ActivityThread.class.getSimpleName();
@HookName("currentActivityThread")
public static Object currentActivityThread() {
Slog.d(TAG, "proxy\_currentActivityThread called", new Exception());
return ArtEngine.callOrigin(null);
}
}
```

### 7.其他特性

#### 7.1 支持线程级别的hook状态切换

**某些场景下您可能会使用该特性**

* 在代理函数中，直接或者间接调用了某些函数。而这些函数您不确定是否被其他业务逻辑所hook，而您不希望导致不明确的问题，比如死循环等，那么您就可以调用`ArtEngine.hookTransition`接口

```Java
try {
ArtHook.hookTransition(true);
Log.d(TAG, "TextView\_ctor<init> called before+++++++");
TextView tv = new TextView(DemoApplication.getMyApplication());
Log.d(TAG, "TextView\_ctor<init> called end--------" + tv);
} finally {
ArtHook.hookTransition(false);
}
```

示例函数分为三个步骤，第二步为调用原函数。第一步以及第三步均调用了`ArtHook.hookTransition`。

所以，您可以简单的将`hookTransition`理解为`hook`状态切换, 该函数非常实用:

1) `true`：状态切换为`origin`，该线程取消`hook`.
2) `false`: 状态切换为`hook`，该线程将进行相关函数代理.

**注意**: 如果您在第二步中，有大量逻辑代码，如果该处逻辑中有直接或者间接调用某一个被`hook`的函数，那么它将不会被代理，直到您调用`hookTransition(false)`为止。所以此处使用了`finally`语句块，确保`hook`继续生效，否则当前线程`hook`功能会失效(仅当前线程)。

## 愿景

### 1.广大需求爱好者可以尽情提出想法，热烈欢迎

### 2.愿大家多提issue

### 3.欢迎提交代码

### 4.丰富完善Apolo插件，不仅仅只有artHook

## 致谢

* 注解参考/使用了其他框架的现有方式，比如`sandhook`

## 个人简介

### @WaxMoon android framework/hook爱好者，2015年入坑

### 个人QQ3403281183

群消息会及时同步最新feature。

### ApoloPlugin微信群

![](upload/attach/202205/953263\_3CV6REPNZ2NADVM.png)

### ApoloPlugin QQ群

![](upload/attach/202205/953263\_MK5YQPT6Q6VVA6R.png)

# ApoloPlugin

**ApoloPlugin** 提供了一个Android轻量级的java hook 库，它支持 arm32 和 arm64两种架构。Apolo意为阿波罗，其为艺术之神，Art翻译过来也有艺术之意，故以此命名。

## 背景

`ArtHook`的灵感，都要从接触`jvmti`技术开始。但是framework只支持debug包加载`jvmti`插件，虽然某些技术分享者过掉了debug检测，但是运行时依然功能受限。后续又通过研究`android.os.Trace`，发现了art另一机制-`instrumentation`。该机制功能非常强大，比如下述源码(均在art/runtime/instrumentation.cc中)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `void Instrumentation::InstallStubsForClass(ObjPtr<mirror::Class> klass) {`  `...`  `}`  `void Instrumentation::InstallStubsForMethod(ArtMethod``*` `method) {`  `...`  `}`    `Instrumentation::InstrumentationLevel Instrumentation::GetCurrentInstrumentationLevel() const {`  `if` `(interpreter_stubs_installed_) {`  `return` `InstrumentationLevel::kInstrumentWithInterpreter;`  `}` `else` `if` `(entry_exit_stubs_installed_) {`  `return` `InstrumentationLevel::kInstrumentWithInstrumentationStubs;`  `}` `else` `{`  `return` `InstrumentationLevel::kInstrumentNothing;`  `}`  `}` |

特别是`GetCurrentInstrumentationLevel`函数中`kInstrumentWithInterpreter`以及`kInstrumentWithInstrumentationStubs`这两个变量大大引发了我的思考，并由此开始了Apolo插件之路。

经过一个月左右的研究，此路已通。虽然还有一些问题待解决，但是经过版本迭代，我相信Apolo插件会更加完善。我个人其实也没有什么终极目标，毕竟技术一直都在变化，只愿能真正帮助到有需求的人。

## 特性

- 支持Android8.1 - Android12
- 支持armeabi-v7a以及arm64-v8a
- 支持静态函数、实例函数、构造方法
- 提供便利的接口调用原方法
- 无需将`ArtMethod`解释执行，性能损耗较小
- 兼容JIT/AOT模式
- 支持xposed api

**注意**: 暂时不支持声明为abstract、interface、native等特殊函数hook

## 即将支持

- **jni函数的hook**功能**研发中...
- **java trace功能**研发中...(逆向分析神器，无需脱壳)
- **Apolo辅助模块**讨论设计中...
- **原理架构文档**待完善

## 缺陷

- 不支持java.lang.String构造函数以及其他暂未发现的函数
- release包的情况下，某些函数可能hook不到(debug包没有该问题，后续会完善解决)

## 如何接入

### 在您module下的build.gradle中增加依赖项

`ApoloPlugin`目前发布在`maven central`, 方便接入

|  |  |
| --- | --- |
| 1  2  3 | `dependencies {`  `implementation` `"io.github.waxmoon:ApoloPlugin:0.0.1"`  `}` |

### 如果您要使用xposed api，也可以增加xposed依赖

|  |  |
| --- | --- |
| 1  2  3 | `dependencies {`  `implementation` `"io.github.waxmoon:xposed:0.0.1"`  `}` |

## 了解接口函数并上手

接口函数可以在仓库`ApoloPlugin module`找到，目前只提供了Java版本

示例代码可以在`app module`中找到

### 1.加载核心so库，初始化hook引擎

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `public` `class` `DemoApplication extends Application {`  `private static Application sApp;`  `static {`  `/``/``init ArtHook`  `ArtEngine.preLoad();`  `}`    `@Override`  `public void onCreate() {`  `super``.onCreate();`  `}`  `}` |

`hook`引擎中会绕过`hiddenApi`检测，该处参考了`RePublic`。所以调用`preLoad`之后，您无须担心`hiddenApi`问题。

### 2.使用HookClass/HookName注解声明您要hook的java method

比如`hook` Java.lang.String.equals函数

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `@HookClass``(String.``class``)`  `public` `class` `StringProxy {`    `private static final String TAG` `=` `StringProxy.``class``.getSimpleName();`  `@HookName``(``"equals"``)`  `public static boolean equals(@ThisObject String str1,` `Object` `str2) {`  `Slog.d(TAG,` `"proxy_equals called %s vs %s"``, str1, str2);`  `return` `ArtEngine.callOrigin(str1, str2);`  `}`  `}` |

**上述代码样例有四个重点**：

- 针对代理类`StringProxy`, 使用`HookClass`或者`HookName`注解，指明原函数所属`class`
- 针对代理函数,使用`HookName`注解, 指明原函数的函数信息
- 原函数若为对象函数，第一个参数必须使用`ThisObject`注解
- 使用`ArtEngine.callOrigin`接口调用原函数

#### QA:

1) 参数类型不是public class，如何声明代理函数？

|  |  |
| --- | --- |
| 1  2 | `代理函数的参数也可使用HookName注解，比如android.app.ContextImpl.createAppContext(ActivityThread mainThread, LoadedApk packageInfo), 您的代理函数参数可以这样使用:`  `(HookName(``"android.app.ActivityThread"` `Object` `mainThread, @HookName(``"android.app.LoadedApk"``)` `Object` `packageInfo))` |

2) 代理函数中能否调用其他被代理的函数，这样是否会导致死循环?

|  |  |
| --- | --- |
| 1 | `代理函数中支持调用其他被代理的函数，但是一旦调用链形成A``-``>ProxyA``-``>B``-``>ProxyB``-``>A这样一个环形结构，就会出现死循环。您可以在调用其他函数时，尝试使用ArtEngine.hookTransition规避此类问题。app样例中就有该类问题，比如hook了StringBuilder.toString函数，并且在代理函数中使用Log.d，有可能就会因为再次间接调用StringBuilder.toString, 导致死循环.` |

下图为`StringBuilder.toString`代理函数的`smali` code，我们可以看到字符串拼接被编译成了`StringBuilder.append`，最后会调用`StringBuilder.toString`。

 ![](https://bbs.kanxue.com/upload/attach/202205/953263_U9Z63VCD5HU26NV.png)
