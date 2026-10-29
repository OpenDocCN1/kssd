
> 以下的一些检测方法在 Berdhard Mueller 的文章《多种特征检测 Frida》中被提出，请参阅它以获得更多细节和示例代码片段。

| 方法 | 描述 | 讨论 |
| --- | --- | --- |
| 检查应用程序签名 | 为了将 `frida-gadget` 嵌入到 APK 中，需要重新打包并签名。你可以在应用程序启动时检查 APK 的签名（例如 `GET_SIGNING_CERTIFICATES`，从 API 等级 28 开始），并将其与你在 APK 中固定的签名进行比较。 | 可以很简单地绕过，比如修改 APK 或者 hook 系统调用。 |
| 在环境中检测相关工件 | 工件可以是包文件、二进制文件、库、进程和临时文件。对于 Frida，这可能是运行在目标的 root 系统中的 `frida-server`（负责通过 TCP 公开 Frida 的守护进程）。检查正在运行的服务（`getRunningServices`）和搜索名为“frida-server”的进程（`ps`），你还可以浏览已加载的库列表并检查可疑的库（例如，那些名字中包含“frida”的库）。 | 从 Android 7.0（API 等级 24），检查运行的服务和进程不会显示像 `frida-server` 这样的守护进程，因为不是由应用本身启动的。即使有可能，只要重命名相应的 Frida 工具（`Frida-server`、`Frida-gadget`、`Frida-agent`），绕过检测也很容易。 |
| 检查打开的 TCP 端口 | 默认情况下，`frida-server` 进程绑定到 TCP 端口 `27042`，检查此端口是否打开是检测守护进程的另一种方法。 | 这个方法在默认模式下会检测到 `frida-server`，但是监听端口可以通过命令行参数更改，所以绕过很简单了。 |
| 检查响应 D-Bus Auth 的端口 | `frida-server` 使用 D-Bus 协议进行通信，因此可以期望它响应 D-Bus AUTH。向每一个开放的端口发送 D-Bus AUTH 消息并检查回复，`frida-server` 会把自己显示出来。 | 这是一种相当健壮的检测 `frida-server` 的方法，但是 Frida 提供了不需要 `frida-server` 的其他操作模式。 |
| 扫描进程内存查找相关工件 | 扫描内存以查找 Frida 库中的工件，例如所有版本的 `frida-gadget` 和 `frida-agent` 中的字符串“LIBFRIDA”。例如，使用 `Runtime.getRuntime().exec` 遍历 `/proc/self/maps` 或 `/proc/<pid>/maps`（取决于 Android 版本）中列出的内存映射来搜索字符串。 | 这种方法更有效一些，而且仅用 Frida 是很难绕过的，特别是在添加了一些混淆和多个工件被扫描的情况下。然而，所选择的工件可能会在 Frida 二进制文件中被修改，在 Berdhard Mueller 的 GitHub 上可以找到源代码。 |

请记住，这个表格远非详尽无遗。我们可以开始讨论命名管道（`frida-server` 用于外部通信）、检测跳板（在函数开头处插入的间接跳转向量），这将有助于检测 Substrate 或 Frida 的拦截器，但是也可能无效，例如对 Frida 的 Stalker 以及许多其他或多或少有效的检测方法。它们中的每一个都将取决于你是否正在使用 root 设备、特定版本的 root 方法或工具本身的版本。最后，这是猫捉老鼠游戏的一部分，保护在不受信任的环境（运行在用户设备中的应用程序）上处理的数据。

> 需要注意的是，这些控制只会增加逆向工程过程的复杂性。如果使用，最好的方法是巧妙地组合这些控制，而不是单独使用它们。然而，它们都不能保证 100% 的有效性，因为逆向工程师总是能够完全访问设备，因此总是会赢！您还必须考虑到，将一些控制集成到应用程序中可能会增加应用程序的复杂性，甚至会影响其性能。

### 有效性评估

在测试设备中安装各种逆向工程工具和框架，然后启动应用程序。至少包括：`Frida`、`Xposed`、`Substrate for Android`、`Drozer`、`RootCloak`、`Android SSL Trust Killer`。

应用程序应该以某种方式响应这些工具的存在。例如：

- 提醒用户并要求承担责任。
- 优雅地停止应用程序执行。
- 安全清除存储在设备上的任何敏感数据。
- 向后端服务器报告，例如进行欺诈的检测。

接下来，绕过逆向工程工具的检测，回答以下问题：

- 这些机制可以被简单地绕过吗（例如，通过 hook 单个 API 函数）？
- 通过静态和动态分析识别反逆向工程代码有多难？
- 您是否需要编写自定义代码来禁用防御？你需要多少时间？
- 你认为绕过这些机制的难度如何？

以下步骤将指导你绕过逆向工程工具检测：

1. 修改反逆向工程功能，通过使用 NOP 指令覆盖相关的字节码或原生代码来禁用不希望的行为。
2. 使用 Frida 或 Xposed 在 Java 和原生层上 hook 文件系统 API，返回原始文件的句柄，而不是修改后的文件。
3. 使用内核模块拦截与文件相关的系统调用，当进程尝试打开修改后的文件时，返回文件未修改版本的文件描述符。

关于修改、代码注入和内核模块的例子，请参考“Android 上的篡改和逆向工程”一章。

## 模拟器检测测试（MSTG-RESILIENCE-5）

### 概述

在防逆向方面，模拟器检测的目的是增加在模拟器设备上运行应用程序的难度，这阻碍了逆向工程师使用某些喜欢的工具和技术。这种难度的增加强制逆向工程师去对抗模拟器检测或使用真实的物理设备，从而限制了大规模设备分析所需的访问。

### 模拟器检测示例

有几个指标可以表明设备是模拟器。尽管所有这些 API 调用都可以 hook，但这些指标提供了第一道防线。

第一组指标在 `build.prop` 文件中。

```text
API Method Value Meaning
Build.ABI armeabi possibly emulator
BUILD.ABI2 unknown possibly emulator
Build.BOARD unknown emulator
Build.Brand generic emulator
Build.DEVICE generic emulator
Build.FINGERPRINT generic emulator
Build.Hardware goldfish emulator
Build.Host android-test possibly emulator
Build.ID FRF91 emulator
Build.MANUFACTURER unknown emulator
Build.MODEL sdk emulator
Build.PRODUCT sdk emulator
Build.RADIO unknown possibly emulator
Build.SERIAL null emulator
Build.USER android-build emulator
```

你可以在 root 的 Android 设备上编辑文件 `build.prop`，也可以在从源代码编译 AOSP 时对其进行修改，两种技术都可以让你绕过上面的静态字符串检查。

下一组静态指标利用 `Telephony manage`。所有 Android 模拟器都有固定的值，这个 API 可以查询。

```text
API Value Meaning
TelephonyManager.getDeviceId() 0's emulator
TelephonyManager.getLine1 Number() 155552155 emulator
TelephonyManager.getNetworkCountryIso() us possibly emulator
TelephonyManager.getNetworkType() 3 possibly emulator
TelephonyManager.getNetworkOperator().substring(0,3) 310 possibly emulator
TelephonyManager.getNetworkOperator().substring(3) 260 possibly emulator
TelephonyManager.getPhoneType() 1 possibly emulator
TelephonyManager.getSimCountryIso() us possibly emulator
TelephonyManager.getSimSerial Number() 89014103211118510720 emulator
TelephonyManager.getSubscriberId() 310260000000000 emulator
TelephonyManager.getVoiceMailNumber() 15552175049 emulator
```

请记住 hook 框架，如 `Xposed` 和 `Frida`，都能 hook 这个 API 提供虚假的数据。

### 绕过模拟器检测

1. 修改模拟器检测功能，通过使用 NOP 指令覆盖相关的字节码或原生代码来禁用不希望的行为。
2. 使用 Frida 或 Xposed API 在 Java 层和原生层 hook 文件 API，返回看起来正常的值（最好取自真实设备），而不是模拟器的值。例如，你可以重写 `TelephonyManager.getDeviceID` 方法返回一个 IMEI 值。
3. 关于修改、代码注入和内核模块的例子，请参考“Android 上的篡改和逆向工程”一章。

### 有效性评估

在模拟器中安装并运行应用程序。应用程序应该检测到它正在模拟器中执行，并终止或拒绝执行本应受到保护的功能。

努力绕过防御，并回答以下问题：

- 通过静态和动态分析识别模拟器检测代码有多难？
- 检测机制可以被简单地绕过吗（例如，通过 hook 单一的 API 函数）？
- 你是否需要编写自定义代码来禁用反模拟器功能？你需要多少时间？
- 你认为绕过这些机制的难度如何？

## 运行时完整性检测测试（MSTG-RESILIENCE-6）

### 概述

此类别中的控制会验证应用程序内存空间的完整性，以保护应用程序不受在运行时的内存补丁的影响。这些补丁包括对二进制代码、字节码、函数指针表和重要数据结构不希望的修改，以及加载到进程内存中的恶意代码。完整性可通过以下方式验证：

1. 将内存或内存校验与正确值进行比较。
2. 在内存中搜索不希望被修改的签名。

这与“检测逆向工程工具和框架”有一些重叠。事实上，我们在那一章中展示了基于签名的方法，当时我们展示了如何搜索进程内存寻找 Frida 相关的字符串。下面是各种完整性监控的更多示例。

### 运行时完整性检测示例

#### Java 运行时检测篡改

下面的检测代码来自 dead && end 的博客。

```java
try {
    throw new Exception();
}
catch (Exception e) {
    int zygoteInitCallCount = 0;
    for (StackTraceElement stackTraceElement : e.getStackTrace()) {
        if (stackTraceElement.getClassName().equals("com.android.internal.os.ZygoteInit")) {
            zygoteInitCallCount++;
            if (zygoteInitCallCount == 2) {
                Log.wtf("HookDetection", "Substrate is active on the device.");
            }
        }
        if (stackTraceElement.getClassName().equals("com.saurik.substrate.MS$2") &&
            stackTraceElement.getMethodName().equals("invoked")) {
            Log.wtf("HookDetection", "A method on the stack trace has been hooked using Substrate.");
        }
        if (stackTraceElement.getClassName().equals("de.robv.android.xposed.XposedBridge") &&
            stackTraceElement.getMethodName().equals("main")) {
            Log.wtf("HookDetection", "Xposed is active on the device.");
        }
        if (stackTraceElement.getClassName().equals("de.robv.android.xposed.XposedBridge") &&
            stackTraceElement.getMethodName().equals("handleHookedMethod")) {
            Log.wtf("HookDetection", "A method on the stack trace has been hooked using Xposed.");
        }
    }
}
```

#### 检测 Native Hook

通过使用 ELF 二进制文件，本地函数 hook 可以通过重写内存中的函数指针（例如，Global Offset Table 或 PLT hook）或修改函数代码本身的一部分（内联 hook）来安装。检查各自内存区域的完整性是检测这种 hook 的一种方法。

Global Offset Table（GOT）用于解析库函数。在运行时，动态链接器会使用全局符号的绝对地址对该表进行修改。GOT hook 会重写存储的函数地址，并将合法的函数调用重定向到逆向者的代码。可以通过枚举进程内存映射并验证每个 GOT 条目指向合法加载的库来检测这种类型的钩子。

与之相反，GNU `ld` 只在第一次需要符号地址时才解析符号地址（惰性绑定），Android 链接器解析所有外部函数，并在加载库之后立即写入相应的 GOT 条目（立即绑定）。因此，您可以期望所有 GOT 条目在运行时都指向它们各自库的代码部分中的有效内存位置。GOT hook 检测方法通常会遍历 GOT 并进行验证。

内联 hook 的工作方式是在函数代码的开头或结尾覆盖一些指令。在运行时，这个所谓的跳板将重定向执行注入的代码。你可以通过检查库函数的开头和结尾是否有可疑指令来检测内联 hook，例如可疑的跳转，会跳到库外位置。

### 绕过和有效性评估

确保禁用了所有基于文件的逆向工程工具的检测。然后，使用 Xposed、Frida 和 Substrate 注入代码，并尝试安装原生 hook 和 Java 方法 hook。应用程序应该能够检测到内存中的“恶意”代码，并做出相应的反应。

使用以下技术绕过检查：

1. 修改完整性检查，通过使用 NOP 指令覆盖相应的字节码或原生代码来禁用不希望的行为。
2. 使用 Frida 或 Xposed 来 hook 用于检测的 API，然后返回虚假值。

关于修改、代码注入和内核模块的例子，请参考“Android 上的篡改和逆向工程”一章。

## 混淆测试（MSTG-RESILIENCE-9）

### 概述

混淆是将代码和数据转换为更难理解的过程，它是每个软件保护方案中不可分割的一部分。需要理解的重要一点是，混淆不是可以简单打开或关闭的东西，程序代码可能以多种方式、不同程度变得完全或部分不可理解。
