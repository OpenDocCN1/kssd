
除非另有说明，我们假定在这里是 root 设备。从 <a href="elink@e18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0r3M7X3g2D9k6h3q4K6k6i4x3`.">Frida 发布页</a>下载`frida-server`二进制文件。确保为你的 Android 设备或模拟器的架构下载了正确的`frida-server`二进制文件：`x86`、`x86_64`、`arm` 或 `arm64`。确保服务器版本(至少是主版本号)与本地 Frida 安装版本匹配。PyPI通常安装最新版本的 Frida。如果不确定安装的版本，可以使用 Frida 命令行工具检查：

```
$ frida --version
```

或者你可以运行以下命令来自动检测 Frida 版本并下载正确的`frida-server`二进制文件：

```
$ wget https://github.com/frida/frida/releases/download/$(frida --version)/frida-server-$(frida --version)-android-arm.xz
```

复制`frida-server`到设备上并运行它：

```
$ adb push frida-server /data/local/tmp/
$ adb shell "chmod 755 /data/local/tmp/frida-server"
$ adb shell "su -c /data/local/tmp/frida-server &"
```

## 在 Android 上使用 Frida

随着`frida-server`的运行，你现在应该可以使用以下命令获得正在运行的进程的列表（用 `-U` 选项指示 Frida 使用一个已连接的 USB 设备或模拟器）：

```
$ frida-ps -U
PID Name
----- --------------------------------------------------------------
276 adbd
956 android.process.media
198 bridgemgrd
30692 com.android.chrome
30774 com.android.chrome:privileged_process0
30747 com.android.chrome:sandboxed
30834 com.android.chrome:sandboxed
3059 com.android.nfc
1526 com.android.phone
17104 com.android.settings
1302 com.android.systemui
(...)
```

或使用 `-Uai` 参数组合限制列表，以获取连接的 USB 设备（`-U`）上当前安装的（`-i`）所有应用程序（`-a`）:

```
$ frida-ps -Uai
PID Name Identifier
----- ---------------------------------------- ------------------------------
766 Android System android
30692 Chrome com.android.chrome
3520 Contacts Storage com.android.providers.contacts
- Uncrackable1 sg.vantagepoint.uncrackable1
- drozer Agent com.mwr.dz
```

这将显示所有应用程序的名称和标识符，如果目前正在运行，还将显示它们的 PID。在列表中搜索你的应用程序，并注意 PID 或其名称/标识符，从现在起你将使用其中一个来引用你的应用程序。建议使用这些应用的标识符，因为在每次运行应用程序时 PID 都会改变。例如，以 `com.android.chrome` 为例，你现在可以在所有 Frida 工具上使用此字符串，如在 Frida CLI，`frida-trace` 或 Python 脚本上。

### 使用 frida-trace 跟踪原生库

要跟踪特定的（底层）库调用，你可以使用 `frida-trace` 命令行工具：

```
$ frida-trace -U com.android.chrome -i "open"
```

这会在`__handlers__/libc.so/open.js`中生成一些 JavaScript 代码，Frida 将其注入到进程中。该脚本将跟踪对 `libc.so` 中的 `open` 函数的所有调用。你可以使用 Frida <a href="elink@b65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1j5`.">JavaScript API</a> 根据需要修改生成的脚本。

不幸的是，尚不支持跟踪 Java 类的高级方法（但<a href="elink@c4fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0V1M7s2W2@1K9r3!0F1i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3y4K6l9`.">将来</a>可能会）。

### Frida CLI 和 Java API

使用 Frida CLI 工具（`Frida`）与 Frida 交互工作。它挂接到一个进程，并为你提供 Frida API 的命令行界面。

```
$ frida -U com.android.chrome
```

通过 `-l` 选项，你可以使用 Frida CLI 加载脚本，例如，加载 `myscript.js`：

```
$ frida -U -l myscript.js com.android.chrome
```

Frida 还提供了一个 <a href="elink@761K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4B7j5i4k6S2">Java API</a>，这对处理 Android 应用程序特别有帮助，它允许你直接使用 Java 类和对象。下面是一个脚本，用于重写`Activity`类的`onResume`函数：

```
Java.perform(function () {
var Activity = Java.use("android.app.Activity");
Activity.onResume.implementation = function () {
console.log("[*] onResume() got called!");
this.onResume();
};
});
```

上面的脚本调用 `Java.perform` 以确保您的代码在 Java VM 的环境中执行。它通过 `Java.use` 实例化一个 `android.app.Activity` 类的包装器，并重写 `onResume` 函数。新的 `onResume` 函数实现打印通知到控制台，并在活动每次处于恢复状态时通过调用 `this.onResume` 来调用原来的 `onResume` 方法。

Frida 还允许你搜索并使用堆上的实例化对象。以下脚本搜索 `android.view.View` 对象的实例，并调用其 `toString` 方法，结果打印到控制台：

```
setImmediate(function() {
console.log("[*] Starting script");
Java.perform(function () {
Java.choose("android.view.View", {
"onMatch":function(instance){
console.log("[*] Instance found: " + instance.toString());
},
"onComplete":function() {
console.log("[*] Finished heap search")
}
});
});
});
```

输出会像这样：

```
[*] Starting script
[*] Instance found: android.view.View{7ccea78 G.ED..... ......ID 0,0-0,0 #7f0c01fc app:id/action_bar_black_background}
[*] Instance found: android.view.View{2809551 V.ED..... ........ 0,1731-0,1731 #7f0c01ff app:id/menu_anchor_stub}
[*] Instance found: android.view.View{be471b6 G.ED..... ......I. 0,0-0,0 #7f0c01f5 app:id/location_bar_verbose_status_separator}
[*] Instance found: android.view.View{3ae0eb7 V.ED..... ........ 0,0-1080,63 #102002f android:id/statusBarBackground}
[*] Finished heap search
```

你还可以使用 Java 的反射功能。要列出 `android.view.View` 类的公共方法，你可以在 Frida 中为此类创建一个包装器，并从该包装器的 `class` 属性调用 `getMethods`：

```
Java.perform(function () {
var view = Java.use("android.view.View");
var methods = view.class.getMethods();
for(var i = 0; i < methods.length; i++) {
console.log(methods[i].toString());
}
});
```

这将在终端打印很长的方法列表：

```
public boolean android.view.View.canResolveLayoutDirection()
public boolean android.view.View.canResolveTextAlignment()
public boolean android.view.View.canResolveTextDirection()
public boolean android.view.View.canScrollHorizontally(int)
public boolean android.view.View.canScrollVertically(int)
public final void android.view.View.cancelDragAndDrop()
public void android.view.View.cancelLongPress()
public final void android.view.View.cancelPendingInputEvents()
...
```

### Frida Binding

为了扩展脚本编写体验，Frida 提供了与Python，C，NodeJS 和 Swift 等编程语言的绑定。

以 Python 为例，首先要注意的是不需要更多的安装步骤。使用 `import frida` 启动 Python 脚本，就可以开始了。请参阅下面的脚本，该脚本仅运行先前的 JavaScript 代码段：

```
# frida_python.py
import frida
session = frida.get_usb_device().attach('com.android.chrome')
source = """
Java.perform(function () {
var view = Java.use("android.view.View");
var methods = view.class.getMethods();
for(var i = 0; i < methods.length; i++) {
console.log(methods[i].toString());
}
});
"""
script = session.create_script(source)
script.load()
session.detach()
```

在这种情况下，运行 Python 脚本（`python3 frida_python.py`）与上一个示例具有相同的结果：它会打印 `android.view.View` 类的所有方法到终端。但是，你可能希望使用 Python 中的数据。使用 `send` 代替 `console.log` 会将数据以 JSON 格式从 JavaScript 发送到 Python 上。请阅读以下示例中的注释：

```
# python3 frida_python_send.py
import frida
session = frida.get_usb_device().attach('com.android.chrome')
# 1. we want to store method names inside a list
android_view_methods = []
source = """
Java.perform(function () {
var view = Java.use("android.view.View");
var methods = view.class.getMethods();
for(var i = 0; i < methods.length; i++) {
send(methods[i].toString());
}
});
"""
script = session.create_script(source)
# 2. this is a callback function, only method names containing "Text" will be appended to the list
def on_message(message, data):
if "Text" in message['payload']:
android_view_methods.append(message['payload'])
# 3. we tell the script to run our callback each time a message is received
script.on('message', on_message)
script.load()
# 4. we do something with the collected data, in this case we just print it
for method in android_view_methods:
print(method)
session.detach()
```

这有效地过滤了方法，只打印包含字符串“Text”的方法：

```
$ python3 frida_python_send.py
public boolean android.view.View.canResolveTextAlignment()
public boolean android.view.View.canResolveTextDirection()
public void android.view.View.setTextAlignment(int)
public void android.view.View.setTextDirection(int)
public void android.view.View.setTooltipText(java.lang.CharSequence)
...
```
