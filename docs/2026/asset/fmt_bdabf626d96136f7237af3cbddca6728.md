
```java
package com.dodonew.online.util;

import android.util.Base64;

import java.security.MessageDigest;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;

public class DesSecurity {

    Cipher deCipher;

    Cipher enCipher;

    public DesSecurity(String key, String iv) throws Exception {
        if (key == null) {
            throw new NullPointerException("Parameter is null!");
        }

        this.InitCipher(key.getBytes(), iv.getBytes());
    }

    private void InitCipher(byte[] secKey, byte[] secIv) throws Exception {
        MessageDigest messageDigest0 = MessageDigest.getInstance("MD5");
        messageDigest0.update(secKey);

        DESKeySpec dsk = new DESKeySpec(messageDigest0.digest());
        SecretKey secretKey0 =
                SecretKeyFactory.getInstance("DES").generateSecret(dsk);

        IvParameterSpec paramSpec = new IvParameterSpec(secIv);

        this.enCipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
        this.deCipher = Cipher.getInstance("DES/CBC/PKCS5Padding");

        this.enCipher.init(1, secretKey0, paramSpec);
        this.deCipher.init(2, secretKey0, paramSpec);
    }

    public byte[] decrypt64(String data) throws Exception {
        return this.deCipher.doFinal(Base64.decode(data, 0));
    }

    public String encrypt64(byte[] data) throws Exception {
        return Base64.encodeToString(this.enCipher.doFinal(data), 0);
    }
}
```

我们可以看到，这段代码用于进行 DES 加密算法的加密和解密操作。首先，`DesSecurity` 类包含两个 `Cipher` 对象：`deCipher` 和 `enCipher`，用于解密和加密操作。

`DesSecurity` 类有一个构造函数，接受 `key` 和 `iv` 两个参数，用于初始化 `DesSecurity` 对象。在构造函数中，会调用 `InitCipher` 方法进行初始化。前面 `encodeDesMap` 方法在 `new` 这个类的对象时，就会调用这个类的构造函数进行初始化。

`InitCipher` 方法接受 `secKey` 和 `secIv` 两个参数，分别代表 DES 算法的密钥和初始化向量。在该方法中，首先使用 MD5 算法对密钥进行摘要处理，然后将 MD5 算法处理后得到的 DES 密钥材料通过 `DESKeySpec` 类进行实例化。实例化 DES 密钥材料后，使用 `SecretKeyFactory.getInstance("DES")` 代码实例化密钥工厂，随后使用密钥工厂通过 `generateSecret` 方法生成最终的 DES 密钥。

随后创建一个 `IvParameterSpec` 对象 `paramSpec`，用于表示初始化向量（Initialization Vector，IV）。前面讲过，初始化向量是在使用 CBC（Cipher Block Chaining）模式进行加密时需要用到的参数，它与密钥一起影响着加密算法的输出结果。

接着使用 `Cipher` 类的 `getInstance` 方法创建加解密操作的 `Cipher` 对象 `enCipher` 和 `deCipher`，并指定使用 DES 算法和 CBC 模式，同时使用 `PKCS5Padding` 填充方式进行填充。

`InitCipher` 方法的最后调用了 `enCipher` 对象的 `init` 方法，用于初始化加密操作。参数 `1` 表示加密模式，`secretKey0` 是之前生成的 DES 密钥，`paramSpec` 是之前创建的初始化向量对象。

类似地，调用 `deCipher` 对象的 `init` 方法，用于初始化解密操作。参数 `2` 表示解密模式，`secretKey0` 是之前生成的 DES 密钥，`paramSpec` 是之前创建的初始化向量对象。

调用这个类的构造函数完成初始化后，因为 `encodeDesMap` 类调用了 `DesSecurity` 对象的 `encrypt64` 方法，所以初始化完成后会执行 `encrypt64` 方法。`encrypt64` 方法将之前得到的、需要进行 DES 加密的值作为参数，最后使用 `enCipher` 对象的 `doFinal` 方法进行加密操作。加密后，将结果进行 Base64 编码，并返回编码后的字符串。

而这个编码后的加密数据经过两次 `return` 后，回到了 `addRequestMap` 方法中。前面看过该方法的代码，返回回来的编码后的加密数据存放在变量 `s1` 中，随后进行了以下代码的处理：

```java
JSONObject obj = new JSONObject();

try {
    obj.put("Encrypt", s1);
    this.mRequestBody = obj + "";
} catch (JSONException e) {
    e.printStackTrace();
}
```

可以看到，这段代码创建了一个 `JSONObject` 对象 `obj`，并向其中放入了一个名为 `"Encrypt"` 的键和对应的 `s1` 变量中的值，然后将 `obj` 转换为字符串并赋值给 `mRequestBody`。这样一来，数据就以 JSON 格式的字符串形式存在了。

如果这段代码中的 `catch` 块捕获了 `JSONException` 类型的异常，就会在捕获到异常时调用 `e.printStackTrace()` 方法打印异常堆栈信息。

到此为止，这个软件的加密流程就被我们搞清楚了。

不知道各位听懂了没有？如果各位想要复现算法、模拟请求过程，可以自己去试试，这里就不演示了。这篇文章也临近尾声，但主要是以动态调试为基础，而动态调试除了 JEB 之外，还有一个常用的软件，那就是 Android Studio。

下面讲一下 Android Studio 怎么进行配置和使用。Android Studio 虽然用起来会比 JEB 麻烦一点，但是功能还是很强大的。至于怎么配置，这里就不多赘述了。如果没有配置好，可以看看这篇文章：

[(32条消息) android studio动态调试apk最详细教程_NarClo的博客-CSDN博客](https://bbs.kanxue.com/elink@065K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3E0W2L8X3u0G2i4K6g2X3x3U0f1%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3U0t1%4x3U0j5I4x3U0R3`.)

唯一美中不足的是，这篇文章配置的并非是 debug 模式，而且有的朋友可能没有安装 AndroidKiller。在给 `AndroidManifest.xml` 文件中添加可调试权限的代码时，和上面一样使用 MT 管理器即可。

## 对动态调试流程的补充

首先，当你知道要动态调试的 APP 的包名，而又不想特意去查找并复制粘贴 APP 的包名时，可以使用以下命令查看 PID：

| 序号 | 命令 |
| --- | --- |
| 1 | `adb shell ps` |

这个命令用于列出当前设备上运行的进程信息。它会显示以下信息：

- `USER`：运行该进程的用户。
- `PID`：进程 ID。
- `PPID`：父进程 ID。
- `VSIZE（VSZ）`：进程占用的虚拟内存大小。
- `RSS`：进程占用的物理内存大小。
- `WCHAN`：进程正在等待的事件。
- `ADDR`：进程所在内存的地址。
- `S`：进程的状态（`R`：运行，`S`：睡眠，`Z`：僵尸等）。
- `NAME`：进程的名称。

例如：

![](https://bbs.kanxue.com/upload/attach/202312/926486_A7SVA7JD5RBU3UE.webp)

其中的 `NAME` 列就是该进程的包名。我们需要观察包名，找到对应的 PID。执行这个命令前，一定要记得把要动态调试的 APP 在模拟器或真机上打开。

我们通过观察 APP 的包名，很快就找到了这个 APP 的 PID：

![](https://bbs.kanxue.com/upload/attach/202312/926486_KFA24NM73XN2RUY.webp)

这两种方法各有优劣，需要根据实际情况来判断哪个更省事，然后选择使用哪个。

下一步需要进行端口转发。文章作者使用的是官方文档给出的端口转发命令，也就是这种格式的命令：

| 序号 | 命令 |
| --- | --- |
| 1 | `adb forward tcp:主机端口 tcp:设备端口` |

执行完端口转发后，再执行启动调试的命令：

| 序号 | 命令 |
| --- | --- |
| 1 | `adb -s 通过命令 adb devices 查出来的设备名 shell am start -D -n 包名/第一个启动的 activity 的类名` |

执行这个命令时，一定要记得将要动态调试的 APP 关闭，否则会报警告。执行完这个命令后，正常情况下模拟器还是会出现这种情况：

![](https://bbs.kanxue.com/upload/attach/202312/926486_Q3AUN5CAXUQ9H99.webp)

接下来只需要去 Android Studio 里面进行附加即可：

![](https://bbs.kanxue.com/upload/attach/202312/926486_VX2S8B8SVZMQ3ZA.webp)

点击红线划出的按钮，就和之前一样可以附加了。附加以后，只要出现 `Connected to the target VM…`，就说明已经附加成功了。

可能有的人想问，debug 模式不就是多执行一行命令吗？需要写得这么麻烦吗？我只是想在这个操作的前后讲一讲，至少看完那篇文章后，再通过 debug 模式启动动态调试时不会看得那么迷糊。

## 安卓逆向常规手段总结

那么文章的最后，我们来对安卓逆向的常规手段进行总结。

首当其冲的自然是静态分析，而静态分析就是搜索。可以根据 APP 反馈情况搜索字符串或者 Android 自带的方法来进行快速定位；在抓包的情况下，可以搜索链接、参数以及算法的关键字。

如果想要获取某个方法中的参数或者返回值，那么有两种常规方法，分别是动态调试和 Log 插桩。

关于 Log 插桩，之前我们讲了如何在对应的 `smali` 文件里添加相应的 `smali` 代码，将我们要获取的信息通过日志的形式输出。但是，这里使用的是仅在 `smali` 文件中插入 `smali` 代码来实现 Log 插桩。

除此之外，还有一种经常见到的 Log 插桩方式，那就是插入 `smali` 文件。我们可以编写一段代码，将这段写好的代码单独编译成 `Smali`，也就是编译成 `dex` 文件后，直接通过 MT 管理器等工具将编译好的 `dex` 文件放入原本的 APK 文件中。

因为 `dex` 有统一的命名规则，所以需要将其命名为 `classes(n+1).dex`，其中 `n` 为 APK 文件中 `classes` 的数量。例如，某 APK 文件的 `classes` 文件数量为 2，那么添加的 `dex` 文件就应该命名为 `classes3.dex`。添加好后，再在需要插入的地方进行调用即可。

插入的 `smali` 代码在编译前可以这样写：

```java
package com.example.myapplication;

import android.util.Log;

public class GenericLogger {
    // 泛型方法，可以接受任何类型的参数，并将其转换为字符串打印
    public static <T> void log(T message) {
        Log.d("CustomTag", String.valueOf(message));
    }

    // 记录当前线程的调用堆栈信息到 Android 的日志中
    public static void logStackTrace() {
        StackTraceElement[] stackTrace =
                Thread.currentThread().getStackTrace();

        for (StackTraceElement stackTraceElement : stackTrace) {
            Log.d(
                    "StackTraceLogger",
                    "at " + stackTraceElement.toString()
            );
        }
    }
}
```

以上这段代码定义了一个名为 `GenericLogger` 的类，它包含两个静态方法：`log()` 和 `logStackTrace()`。

`log()` 方法是一个泛型方法，可以接受任何类型的参数，并将其转换为字符串打印。该方法使用 Android 的 `Log.d()` 方法将日志记录到自定义标签 `CustomTag` 中。

`logStackTrace()` 方法记录当前线程的调用堆栈信息到 Android 的日志中。该方法使用 `Thread.currentThread().getStackTrace()` 方法获取当前线程的堆栈跟踪信息，并使用 Android 的 `Log.d()` 方法将其记录到自定义标签 `StackTraceLogger` 中。

这些方法都是静态方法，可以直接通过类名调用，不需要创建 `GenericLogger` 类的实例。这使得它们可以在整个应用程序中被方便地重复使用。

写好需要编译成 `dex` 文件的 Android Java 代码后，下一步需要通过工具将文件转换成 `Smali` 指令文件，然后直接通过 MT 管理器等工具将编译好的 `dex` 文件放入原本的 APK 文件中并修改包名。

可以进行全局搜索，将全部包名替换成需要进行 Log 插桩的 APK 文件的包名，最后调用编译好的 `smali` 代码即可完成 Log 插桩。

后面的编译成 `dex` 文件并插入的步骤大家可以自行尝试。我使用 Android Studio 试过，编译前的 Java 代码是可以使用的。

其次，还有之前讲过的 APP 资源 ID。例如，可以通过工具获取资源 ID，到资源文件夹寻找相关资源调用；也可以通过搜索字符串获取字符串对应的 `name` 字段，再通过 `name` 字段到 `public.xml` 中找到该字符串的资源 ID，最后通过资源 ID 来寻找相关资源调用。

还有通过消息框、按钮等样式的调用代码来找到关键位置，以及使用 DDMS 的方法分析来寻找突破口等方法。

到此为止，我们过了一遍安卓逆向的常规手段。但安卓逆向的手段远不止如此，还有很多手段需要大家去探索。正所谓：“路漫漫其修远兮，吾将上下而求索。”愿大家在自己的路上越走越远。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-4-24 14:27

被黎明与黄昏编辑，原因：附件失效，重新上传附件。

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

上传的附件：

- [嘟嘟牛在线.apk](javascript:void(0);)（7.55MB，17 次下载）
