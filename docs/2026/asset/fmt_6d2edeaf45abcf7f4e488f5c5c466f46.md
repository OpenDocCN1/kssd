
# 由 2021 ByteCTF 引出的 Intent 重定向浅析

**在此先感谢 ByteCTF 的赛前培训，感谢 summer 师傅的倾情讲解。**

## Intent 浅要概述

- Intent 是 Android 程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据。Intent 一般可被用于启动活动、启动服务以及发送广播等场景。
- Intent 是一种运行时绑定（runtime binding）机制，它能在程序运行的过程中连接两个不同的组件。通过 Intent，你的程序可以向 Android 表达某种请求或者意愿，Android 会根据意愿的内容选择适当的组件来响应。

Intent 大致可以分为两种：显式 Intent 和隐式 Intent。

- 显式 Intent：直接设置目标组件的 `ComponentName`（目的组件），用于一个应用内部的消息传递，比如启动另一个 Activity 或者一个 service。

  通过 Intent 的 `setComponent()` 和 `setClass()` 来指定目标组件的 `ComponentName`。

- 隐式 Intent：没有指定 `ComponentName`，而是通过指定一系列的 action 和 category 等信息，交由系统去分析，并找出合适的组件来响应。

同时，Intent 可以携带以下信息。

1. **component（组件）**：目的组件

   Component 属性明确指定 Intent 的目标组件的类名称。如果 component 这个属性有指定的话，将直接使用它指定的组件（显式 Intent）。指定了这个属性以后，Intent 的其他所有属性都是可选的。

2. **Action（动作）**：用来表现意图的行动

   Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 Intent Filter 可以包含多个 Action。在 AndroidManifest.xml 的 Activity 定义时，可以在其 `<intent-filter>` 节点指定一个 Action 列表，用于标识 Activity 所能接受的“动作”。

3. **category（类别）**：用来表现动作的类别

   该属性也是通过作为 `<intent-filter>` 的子元素来声明的。如果没有指定 category，将会使用默认的 `"android.intent.category.DEFAULT"`。只有当 `<action>` 和 `<category>` 中的内容同时能够匹配上 Intent 中指定的 action 和 category 时，这个活动才能响应 Intent。

   Intent 可以通过 `addCategory()` 方法指定多个 `category`，只有同时满足时，才能匹配成功。

4. **data（数据）**：表示与动作要操纵的数据

   Data 是用一个 URI 对象来表示的。`<data>` 标签可配置以下属性：

   - `android:scheme`：用于指定数据的协议部分，如上例中的 `http` 部分。
   - `android:host`：用于指定数据的主机名部分，如上例中的加密内容。
   - `android:port`：用于指定数据的端口部分，一般紧随在主机名之后。
   - `android:path`：用于指定主机名和端口之后的部分，如一段网址中跟在域名之后的内容。
   - `android:mimeType`：用于指定可以处理的数据类型，允许使用通配符的方式进行指定。

   只有 `<data>` 标签中指定的内容和 Intent 中携带的 Data 完全一致时，当前活动才能够响应该 Intent。

5. **type（数据类型）**：指定 Data 属性的数据类型

   如果 Intent 对象中既包含 URI 又包含 Type，那么，在 `<intent-filter>` 中也必须二者都包含才能通过测试。

   Type 属性用于明确指定 Data 属性的数据类型或 MIME 类型。但是通常来说，当 Intent 不指定 Data 属性时，Type 属性才会起作用；否则 Android 系统将会根据 Data 属性值来分析数据的类型，所以无需指定 Type 属性。

   data 和 type 属性一般只需要一个。通过 `setData()` 方法会把 type 属性设置为 `null`，相反，设置 `setType()` 方法会把 data 设置为 `null`。如果想要两个属性同时设置，要使用 `Intent.setDataAndType()` 方法。

6. **extras（扩展信息）**：扩展信息

   提供附加数据。使用 `putExtra()` 方法来设置额外的 key-value 数据。

7. **Flags（标志位）**：期望意图的运行模式

   用来指示系统如何启动一个 Activity（比如：这个 Activity 属于哪个 Activity 栈）和 Activity 启动后如何处理它（比如：是否把这个 Activity 归为最近的活动列表中）。

## babydroid

环境：`API 30_x86`

### 漏洞分析

#### 服务源码文件 `server.py`

```python
print_to_user(r"""
____ __ ____ __
/\ _`\ /\ \ /\ _`\ __ /\ \
\ \ \L\ \_\ \ \ \__\ __ \ \/\ \ \_ \__ \__\_ /\\_\ \_\ \
\ \ \_ <' /'__`\ \ \ '__`\ /\ \/\ \ \ \ \/\`'__\/ __`\/ \ /'_` \
\ \ \L\ \/\ \L\.\_\ \ \L\ \ \ \_\ \ \ \ \ \//\ \L\ \ \/\ \L\ \
\ \____/ \ \__/.\\_\ \ \_,_\_/\/`____ \ \____/ \_\ \____/ \_\_\__,_\
\/_____/ \/_/\/_/ \/_/ \/\___/> \/\___/ \/_/ \/\___/ \/_/\/_\_,_ /
/\___/
\/__/
""")
if not proof_of_work():
    print_to_user("Please proof of work again, exit...\n")
    exit(-1)
print_to_user("Please enter your apk url:")
url = sys.stdin.readline().strip()
EXP_FILE = download_file(url)
if not check_apk(EXP_FILE):
    print_to_user("Invalid apk file.\n")
    exit(-1)
print_to_user("Preparing android emulator. This may takes about 2 minutes...\n")
emulator = setup_emulator()
adb(["wait-for-device"])
adb_install(APK_FILE)
adb_activity(f"{VULER}/.MainActivity", wait=True)
with open(FLAG_FILE, "r") as f:
    adb_broadcast(
        f"com.bytectf.SET_FLAG",
        f"{VULER}/.FlagReceiver",
        extras={"flag": f.read()}
    )
time.sleep(3)
adb_install(EXP_FILE)
adb_activity(f"{ATTACKER}/.MainActivity")
print_to_user("Launching! Let your apk fly for a while...\n")
time.sleep(EXPLOIT_TIME_SECS)
try:
    os.killpg(os.getpgid(emulator.pid), signal.SIGTERM)
except:
    traceback.print_exc()
```

- h13～h15：进行了一个 SHA-256 的验证，必须满足 `sha256((prefix + proof).encode()).hexdigest().startswith(difficulty * "0") == True` 才能继续。其中，`prefix` 为 `random_hex(6)` 生成的随机字符；`proof` 为用户输入的字符串。可以每次爆破得到正确的结果来输入。
- h17～h22：用户输入一个 URL 地址。server 会将该文件下载下来，同时检查是否为 APK。
- h24～h26：新建一个安卓模拟器。
- h28：将含有漏洞的 APK 安装。
- h29：启动受害 APK。
- h30～h31：打开服务器中保存的 flag 文件，然后通过 adb 命令发送一个广播，将 flag 传输给 `.FlagReceiver`。其中命令如下：

```bash
shell su root am broadcast -W -a com.bytectf.SET_FLAG -n com.bytectf.babydroid/.FlagReceiver -e flag [flag]
```

- h34～h35：安装 attack APK 并运行。

#### APK 分析

##### `AndroidManifest.xml`

![](upload/attach/202110/925172_2FYGS76HGNS9XAT.png)

可以看到注册了两个 Activity、一个 Receiver，还有一个 FileProvider。其中两个 Activity 都是可导出的（可被外部组件访问）。

##### `MainActivity`

MainActivity 中没有做操作，只是载入布局来显示。

##### `Vulnerable`（易受攻击的）

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.startActivity(this.getIntent().getParcelableExtra("intent"));
}
```

该 Activity 通过 `getIntent()` 首先获得 Intent 对象，之后使用 `getParcelableExtra("intent")` 获取反序列化后名为 `"intent"` 的 extra 数据，并使用 `startActivity()` 来启动该 Intent。

##### `FlagReceiver`

```java
public void onReceive(Context context, Intent intent) {
    String flag;
    String str = "flag";
    if ((flag = intent.getStringExtra(str)) != null) {
        File file = new File(context.getFilesDir(), str);
        this.writeFile(file, flag);
        Log.e("FlagReceiver", "received flag.");
    }
    return;
}
```

该广播接收者从 Intent 中取到一个 key 为 `"flag"` 的 String 类型数据，之后将其写出到 `/data/data/[package_name]/files/flag` 文件中，对应了 `server.py` 中传入 flag 的逻辑。我们想要获取的 flag 也就是这个路径。

##### FileProvider

[FileProvider | Android Developers（google.cn）](elink@938K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1P5q4)9J5c8X3y4G2M7X3g2Q4x3V1k6U0L8$3&6@1k6h3&6@1i4K6u0r3c8X3W2D9k6g2m8J5L8%4k6A6k6r3g2J5i4K6y4r3K9r3I4Q4x3@1c8W2L8R3`.`.)

FileProvider 是 `ContentProvider` 的一个特殊子类，通过使用 `content://` URI 来代替 `file://` URI，以促进安全地共享与 App 关联的文件。

Content URI 允许临时授予该文件的读写权限。所以当创建一个包含文件 Content URI 的 Intent 时，（如果需要）可以通过 `Intent.setFlags()` 添加相应的权限。同时，权限会通过 Intent 传递给接收的 Activity（或者服务）。

在创建 FileProvider 时，需提供一个 XML 文件来指定该 Provider 提供的文件。如 `res/xml/file_paths.xml`，其中提供了相应的映射路径以及别名。同时需要在 `AndroidManifest.xml` 中的 FileProvider 对应节点内定义：

```xml
<meta-data
    android:name="android.support.FILE_PROVIDER_PATHS"
    android:resource="@xml/file_paths">
</meta-data>
```

其中 `resource` 属性内为提供的 XML 文件。

该 APK 指定的文件 XML 中定义如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <root-path name="root" path="" />
</paths>
```

- `<root-path>` 节点：代表设备的根目录。

另外的节点：

![](upload/attach/202110/925172_66S2NW7ZGJBPK62.png)

- `name` 属性：给该目录起的别名，用于隐藏路径。
- `path` 属性：临时授权访问的路径（该目录下的子目录）。

该 APK 提供的 FileProvider 提供了根目录，通过 `root/` 别名来访问。

如果我们想访问 flag 的存储路径，实际构造的部分 URI 应该为：

`root/data/data/[package_name]/files/flag`

### 利用思路

#### FlagURI

题目将 flag 存储到了 `/data/data/[package_name]/files/flag` 文件，同时提供了一个 FileProvider。因此，思路是通过 FileProvider 来将该文件读取。

首先需要构造一个 Content URI，格式为：

```uri
content://[authorities]/[name]/[file_relative_path]
```

构造的 URI 为：

```uri
content://androidx.core.content.FileProvider/root/data/data/com.bytectf.babydroid/files/flag
```

#### 获得权限

由于该 `FileProvider` 的 `exported` 属性（必须）设置为 `false`，导致我们无法在外部组件中直接使用该 Provider。

但是，该 APK 提供了一个 `Vulnerable` Activity，并且（不做任何校验地）将接收到的 Intent 中 key 为 `"intent"` 的 extra 数据当作 Intent，并使用 `startActivity()` 来启动。这样就对启动的 Activity 进行了临时授权，可以访问该应用中未导出的组件。

因此，我们可以设置一个 Intent 来启动 `Vulnerable`，同时给该 Intent 附加一个 key 为 `"intent"` 的数据，该数据包含着构造好的恶意 Intent。

当 `Vulnerable` 被启动时，就会找到 Intent 的数据。我们在 Intent 中附上我们的 attack APK 的 Activity，同时附加上 flag 的 Content URI。

之后转到我们自己的 Activity 时，就可以任意读写目标应用内部文件。

### Attack 代码实现

#### Attack APP
