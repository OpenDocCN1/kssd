
```python
{'type': 'error', 'description': 'Error: java.lang.ClassNotFoundException: Didn\'t find class "com.iqiyi.plugin.widget.dkplayer.controller.VideoController" on path: DexPathList[[dex file "InMemoryDexFile[cookie=[0, 3983850208]]", zip file "/system/framework/org.apache.http.legacy.boot.jar", zip file "/data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/base.apk"],nativeLibraryDirectories=[/data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/lib/arm, /data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/base.apk!/lib/armeabi-v7a, /system/lib]]', 'stack': 'Error: java.lang.ClassNotFoundException: Didn\'t find class "com.iqiyi.plugin.widget.dkplayer.controller.VideoController" on path: DexPathList[[dex file "InMemoryDexFile[cookie=[0, 3983850208]]", zip file "/system/framework/org.apache.http.legacy.boot.jar", zip file "/data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/base.apk"],nativeLibraryDirectories=[/data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/lib/arm, /data/app/com.iqiyi.vider-U7T4aZIQOQyXS5iLBgsNGw==/base.apk!/lib/armeabi-v7a, /system/lib]]\n    at frida/node_modules/frida-java-bridge/lib/env.js:124\n    at frida/node_modules/frida-java-bridge/lib/class-factory.js:400\n    at frida/node_modules/frida-java-bridge/lib/class-factory.js:781\n    at frida/node_modules/frida-java-bridge/lib/class-factory.js:90\n    at frida/node_modules/frida-java-bridge/lib/class-factory.js:44\n    at /script1.js:23\n    at frida/node_modules/frida-java-bridge/lib/vm.js:11\n    at frida/node_modules/frida-java-bridge/index.js:368\n    at frida/node_modules/frida-java-bridge/index.js:318', 'fileName': 'frida/node_modules/frida-java-bridge/lib/env.js', 'lineNumber': 124, 'columnNumber': 1}
```

从运行结果来看，出现`ClassNotFoundException`错误，说明没有找到我们要`hook`的类。

### Frida 枚举 `ClassLoader`

由于是插件化 APK，类加载是在插件化框架自定义的，所以`ClassLoader`不能使用默认的。我们可以使用`Java.enumerateClassLoaders(callbacks)`来打印出所有的加载器。

```javascript
Java.perform(function () {
    Java.enumerateClassLoaders({
        "onMatch": function (loader) {
            console.log(loader);
        },
        "onComplete": function () {
            console.log("success");
        }
    });
});
```

运行结果：

![](https://bbs.kanxue.com/upload/attach/202004/403246_KXQEZKFZMH32CRX.png)

由上面分析可知，真正逻辑代码是在`plugin-shadow-apk-debug.apk`中，那该 APK 对应的`ClassLoader`是`com.tencent.shadow.core.loader.classloaders.PluginClassLoader`。

### Frida 指定 `ClassLoader`

来看下`Java.ClassFactory`中`loader`的介绍："read-only property providing a wrapper for the class loader currently being used."，`loader`是当前`ClassLoader`的 wrapper，我们修改`ClassLoader`可以通过修改该字段。`Java.classFactory`是默认的 class factory，所以我们需要修改的是`Java.classFactory.loader`。

```javascript
Java.perform(function () {
    Java.enumerateClassLoaders({
        "onMatch": function (loader) {
            if (loader.toString().startsWith("com.tencent.shadow.core.loader.classloaders.PluginClassLoader")) {
                Java.classFactory.loader = loader; // 将当前 class factory 中的 loader 指定为我们需要的
            }
        },
        "onComplete": function () {
            console.log("success");
        }
    });
});
```

### 最终脚本

```javascript
Java.perform(function () {
    Java.enumerateClassLoaders({
        "onMatch": function (loader) {
            if (loader.toString().startsWith("com.tencent.shadow.core.loader.classloaders.PluginClassLoader")) {
                Java.classFactory.loader = loader; // 将当前 class factory 中的 loader 指定为我们需要的
            }
        },
        "onComplete": function () {
            console.log("success");
        }
    });

    // 此处需要使用 Java.classFactory.use
    var videoController = Java.classFactory.use("com.iqiyi.plugin.widget.dkplayer.controller.VideoController");

    videoController.setVip.implementation = function () {
        console.log("hook setVip");
        this.setVip(true);
    };
});
```

运行结果：

![](https://bbs.kanxue.com/upload/attach/202004/403246_4MJWNEF5GNQQ3MQ.png)

可以看到，我们已经成功`hook`，并且视频上已经没有显示剩余时间。

## Frida Hook Enum

直播和小说的 VIP 判断和视频是不一致的，是通过`enum`中 VIP 字段值和`1`进行对比来判断，具体定位过程和上面类似。

![](https://bbs.kanxue.com/upload/attach/202004/403246_HVPCGF5EH2B5JZU.png)

判断代码为：

> `if (TextUtils.equals("1", PluginEnum.VIP.getValue())) {...}`

### Enum 测试

我们的目的是为了`hook VIP`，但是对`enum`的这种用法不是很熟，于是写了个测试程序，来进一步了解。

```java
public enum TestEnum {
    A("a"),
    B("b"),
    C("c");

    private String value;

    private TestEnum(String value) {
        this.value = value;
    }

    public String getValue() {
        return this.value;
    }
}
```

使用`javap`打开对应的 class 文件：

```text
Compiled from "TestEnum.java"

public final class TestEnum extends java.lang.Enum<TestEnum> {
  public static final TestEnum A;
  public static final TestEnum B;
  public static final TestEnum C;
  public static TestEnum[] values();
  public static TestEnum valueOf(java.lang.String);
  public java.lang.String getValue();
  static {};
}
```

从这里可以很明显看到，A、B 和 C 都属于`TestEnum`中的静态成员变量。来看下调用的`smali`代码：

```text
sget-object v3, Lcom/iqiyi/plugin/base/PluginEnum;->VIP:Lcom/iqiyi/plugin/base/PluginEnum;

invoke-virtual {v3}, Lcom/iqiyi/plugin/base/PluginEnum;->getValue()Ljava/lang/String;
```

从`smali`上也能看出来类似的逻辑，VIP 是`com/iqiyi/plugin/base/PluginEnum`的静态成员，然后在调用`getValue()`方法。所以我们`hook com/iqiyi/plugin/base/PluginEnum`类的`getValue`方法，然后判断调用者是否为 VIP。

### 最终脚本

```javascript
Java.perform(function () {
    var pluginEnum = Java.classFactory.use("com.iqiyi.plugin.base.PluginEnum");
    var String = Java.use("java.lang.String");

    pluginEnum.getValue.implementation = function () {
        var value = this.getValue();

        if (this == "VIP") { // 此时 this 或者 this.getString() 返回的是静态成员名
            var vip = String.$new("1");
            this.setValue(vip); // 调用 setValue 修改 VIP 值
            return vip;
        } else {
            return value;
        }
    };
});
```

## 整体脚本

```javascript
Java.perform(function () {
    Java.enumerateClassLoaders({
        "onMatch": function (loader) {
            if (loader.toString().startsWith("com.tencent.shadow.core.loader.classloaders.PluginClassLoader")) {
                Java.classFactory.loader = loader;
            }
        },
        "onComplete": function () {
            console.log("success");
        }
    });

    var videoController = Java.classFactory.use("com.iqiyi.plugin.widget.dkplayer.controller.VideoController");

    videoController.setVip.implementation = function () {
        console.log("hook setVip");
        this.setVip(true);
    };

    var pluginEnum = Java.classFactory.use("com.iqiyi.plugin.base.PluginEnum");
    var String = Java.use("java.lang.String");

    pluginEnum.getValue.implementation = function () {
        var value = this.getValue();

        if (this == "VIP") {
            var vip = String.$new("1");
            this.setValue(vip);
            return vip;
        } else {
            return value;
        }
    };
});
```

## 总结

通过对该样本的分析，逆向找寻关键代码相对简单，但是在使用 Frida `hook` 时相对难点，特别是对于 Frida 和插件化不熟的情况下。本文涉及到的有：

1. Frida 枚举所有加载的类；
2. Frida 枚举`ClassLoader`；
3. Frida 对`enum`类型的`hook`。

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
