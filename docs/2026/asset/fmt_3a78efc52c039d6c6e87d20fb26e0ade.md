
`$ adb shell pm list packages -3 -f` `package:/data/app/sg.vantagepoint.helloworldjni-1/base.apk=sg.vantagepoint.helloworldjni` `package:/data/app/eu.chainfire.supersu-1/base.apk=eu.chainfire.supersu` `package:/data/app/org.teamsik.apps.hackingchallenge.easy-1/base.apk=org.teamsik.apps.hackingchallenge.easy` `package:/data/app/org.teamsik.apps.hackingchallenge.hard-1/base.apk=org.teamsik.apps.hackingchallenge.hard` `package:/data/app/sg.vp.owasp_mobile.omtg_android-kR0ovWl9eoU_yh0jPJ9caQ==/base.apk=sg.vp.owasp_mobile.omtg_android`

这与在应用程序包 ID 上运行 `adb shell pm path <app_package_id>` 相同：

```
$ adb shell pm path sg.vp.owasp_mobile.omtg_android
package:/data/app/sg.vp.owasp_mobile.omtg_android-kR0ovWl9eoU_yh0jPJ9caQ==/base.apk
```

用于 `frida-ps -Uai` 获取连接的 USB 设备 (`-U`) 上当前安装 (`-i`) 的所有应用 (`-a`)：

```
$ frida-ps -Uai
PID  Name                                      Identifier
-----   --------------------------------------   --------------------------------------
766  Android System                            android
21228  Attack me if u can                      sg.vp.owasp_mobile.omtg_android
4281  Termux                                    com.termux
-     Uncrackable1                              sg.vantagepoint.uncrackable1
```

请注意，这还显示了当前正在运行的应用程序的 PID。记下“标识符”和 PID（如果有的话），因为以后您将需要它们。

#### 探索应用程序包

一旦您收集了您想要定位的应用程序的包名称，您就需要开始收集有关它的信息。首先，按照“基本测试操作 - 获取和提取应用程序”中的说明检索 APK。

APK 文件实际上是可以使用标准解压缩器解压缩的 ZIP 文件：

```
$ unzip base.apk
$ ls -lah
-rw-r--r--  1 sven  staff  11K Dec 5 14:45 AndroidManifest.xml
drwxr-xr-x  5 sven  staff  170B Dec 5 16:18 META-INF
drwxr-xr-x  6 sven  staff  204B Dec 5 16:17 assets
-rw-r--r--  1 sven  staff  3.5M Dec 5 14:41 classes.dex
drwxr-xr-x  3 sven  staff  102B Dec 5 16:18 lib
drwxr-xr-x 27 sven  staff  918B Dec 5 16:17 res
-rw-r--r--  1 sven  staff  241K Dec 5 14:45 resources.arsc
```

以下是 apk 解压出来的文件：

- `AndroidManifest.xml`：包含应用的包名、目标和最低 API 级别、`AndroidManifest.xml`、应用配置、应用组件、权限等的定义。
- `META-INF`：包含应用的元数据
  - `MANIFEST.MF`：存储应用资源的哈希值
  - `CERT.RSA`：应用程序的证书
  - `CERT.SF`：资源列表和 `MANIFEST.MF` 文件中相应行的 SHA-1 摘要
- `assets`：包含应用资源的目录（Android 应用中使用的文件，例如 XML 文件、JavaScript 文件和图片），`AssetManager` 可以检索该目录
- `classes.dex`：编译成 DEX 文件格式的类，Dalvik 虚拟机 / ART 虚拟机可以处理。DEX 是 Dalvik 虚拟机的字节码。它针对小型设备进行了优化
- `lib`：包含 APK 一部分第三方库的目录。
- `res`：包含尚未编译到 `resources.arsc` 中的资源的目录
- `resources.arsc`：包含预编译资源的文件，例如布局的 XML 文件

由于使用标准 `unzip` 实用程序解压缩会留下一些文件，例如 `AndroidManifest.xml` 无法读取的文件，因此您最好使用 apktool 解压缩 APK，如“推荐工具 - apktool”中所述。解包结果为：

```
$ ls -alh
total 32
drwxr-xr-x  9 sven  staff  306B Dec 5 16:29 .
drwxr-xr-x  5 sven  staff  170B Dec 5 16:29 ..
-rw-r--r--  1 sven  staff  10K Dec 5 16:29 AndroidManifest.xml
-rw-r--r--  1 sven  staff  401B Dec 5 16:29 apktool.yml
drwxr-xr-x  6 sven  staff  204B Dec 5 16:29 assets
drwxr-xr-x  3 sven  staff  102B Dec 5 16:29 lib
drwxr-xr-x  4 sven  staff  136B Dec 5 16:29 original
drwxr-xr-x 131 sven  staff  4.3K Dec 5 16:29 res
drwxr-xr-x  9 sven  staff  306B Dec 5 16:29 smali
```

**The Android Manifest**

Android Manifest 是主要的信息来源，它包含了很多有趣的信息，如包名、权限、应用组件等。

以下是一些信息和相应关键字的非详尽列表，您可以通过查看文件或使用 `grep -i <keyword> AndroidManifest.xml` 在 Android Manifest 中轻松搜索：

- 应用权限：`permission`（参见“Android 平台 API”）
- 备份限额：`android:allowBackup`（请参阅“Android 上的数据存储”）
- 应用组件：`activity`, `service`, `provider`, `receiver`（参见“Android 平台 API”和“Android 上的数据存储”）
- 可调试标志：`debuggable`（参见“Android 应用程序的代码质量和构建设置”）

请参阅上述章节以了解有关如何测试每个点的更多信息。

**应用程序二进制**

如上面“探索应用程序包”中所见，应用程序二进制文件 (`classes.dex`) 可以在应用程序包的根目录中找到。它是一个所谓的 DEX（Dalvik Executable）文件，其中包含已编译的 Java 代码。由于其性质，在使用一些转换工具后，您将能够使用反编译器生成 Java 代码。我们还看到了运行 apktool 后获得的文件夹 `smali`。这包含了反汇编 Dalvik 字节码，也就是被称为 smali 的中间语言，这是 Dalvik 可执行文件的可读表示。

有关如何对 DEX 文件进行逆向工程的更多信息，请参阅“Android 上的篡改和逆向工程”一章中的“查看反编译的 Java 代码”部分。

**Native库**

您可以在 APK 中检索 `lib` 文件夹：

```
$ ls -1 lib/armeabi/
libdatabase_sqlcipher.so
libnative.so
libsqlcipher_android.so
libstlport_shared.so
```

或来自有 objection 的设备：

```
...g.vp.owasp_mobile.omtg_android on (google: 8.1.0) [usb]
# ls lib
Type    ...  Name
-----   ...
File    ...  libnative.so
File    ...  libdatabase_sqlcipher.so
File    ...  libstlport_shared.so
File    ...  libsqlcipher_android.so
```
