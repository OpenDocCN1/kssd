
根据后台优化（<<mark class="encrypted">24cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0H3K9h3y4Q4x3V1k6H3k6i4u0X3L8%4u0E0j5h3&6U0k6g2)9J5c8X3u0S2j5$3E0Y4M7X3!0#2L8X3c8Q4x3X3c8G2M7s2c8A6L8h3W2\*7j5i4c8A6L8$3^5`.</mark>>），针对 Android 7.0（API 等级24）或更高的应用不再接收 `CONNECTIVITY_ACTION` 广播，除非使用 `Context.registerReceiver()` 注册广播接收器。系统也不再发送 `ACTION_NEW_PICTURE` 和 `ACTION_NEW_VIDEO` 广播。

根据后台执行限制（<<mark class="encrypted">59bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3j5h3u0G2N6i4c8Q4x3V1k6$3k6i4u0K6K9h3!0F1M7#2)9J5c8X3!0J5k6h3!0Q4x3V1k6T1j5h3y4C8k6%4u0G2N6h3&6V1i4K6u0W2K9s2c8E0L8q4)9J5x3$3u0J5L8$3q4V1j5$3q4K6N6s2x3`.</mark>>），针对Android 8.0（API 等级26）及以上的应用，在隐式广播例外情况中列出的除外（<<mark class="encrypted">b80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6U0L8$3#2H3L8$3&6W2L8Y4c8K6i4K6u0r3j5Y4u0G2j5h3c8U0j5i4y4@1i4K6u0V1k6i4S2U0k6i4m8@1K9h3!0F1M7H3`.`.</mark>>），不能在其清单中注册隐式广播接收器。在运行时通过调用 `Context.registerReceiver` 创建广播接收器不受此限制的影响。

根据系统广播发生的更改，从Android 9（API 等级28）开始，`NETWORK_STATE_CHANGED_ACTION` 广播不再接收用户的位置信息或个人身份信息。

### 内容提供者

Android 使用 SQLite 来持续化存储数据：与 Linux 一样，数据存储在文件中。SQLite 是一种轻量级、高效的开源关系数据存储技术，不需要太多的处理能力，这使得它非常适合手机使用。SQLite 有包含特定类（`Cursor`, `ContentValues`, `SQLiteOpenHelper`, `ContentProvider`, `ContentResolver` 等等）的完整可用 API 。SQLite 不作为单独的进程运行的，它是应用程序的一部分。默认情况下，属于给定应用程序的数据库只能由该应用程序访问。但是，内容提供者提供了一种很棒的来抽象数据源机制（包括数据库和平面文件），它们还提供了一种标准而有效的机制来在应用（包括本地应用）之间共享数据。要让其他应用程序访问内容提供者，需要在共享它的应用程序的清单文件中显式地声明内容提供者。只要没有声明内容提供者，它们就不会被导出，只能由创建它们的应用程序调用。

内容提供者通过 URI 寻址方案实现的：它们都使用了 `content://` 模型。无论源的类型是什么（SQLite数据库、平面文件等），寻址方案都是相同的，因此对源进行了抽象，并为开发人员提供了唯一的方案。内容提供者提供了所有常规的数据库操作：创建、读取、更新、删除。这意味着任何在清单文件中拥有适当权限的应用程序都可以操作其他应用程序的数据。

### 服务

服务是 Android OS 的组件（基于 `Service` 类），它们在后台执行任务（数据处理、启动 intent 和通知等），而不提供用户界面。服务是用来长期运行进程的，它们的系统优先级低于活动应用程序，而高于不活动应用程序。因此，当系统需要资源时，它们不太可能被杀死，并且可以将它们配置为在有足够的可用资源时自动重启，这使得服务非常适合运行后台任务。请注意，服务和活动一样，都是在主应用程序线程中执行的。服务不会创建自己的线程，也不会在单独的进程中运行，除非另行指定。

### 权限

由于 Android 应用程序是安装在沙箱中，最初无法访问用户信息和系统组件（比如摄像头和麦克风），Android 为应用程序可以请求的特定任务提供了一套预定义的权限。例如，如果你想让你的应用程序使用手机的摄像头，你必须请求 `android.permission.CAMERA` 权限。在Android 6.0（API 等级23）之前，应用程序请求的所有权限都在安装时授予。从 API 等级23开始，用户只能在应用程序执行期间同意一些权限请求。

### 保护等级

Android 的权限是根据其提供的保护级别进行排序的，分为四种不同的类别：

* Normal：较低的保护级别。 它使应用程序可以访问隔离的应用程序级功能，而对其他应用程序、用户或系统的风险最小。它是在应用安装过程中授予的，并且是默认的保护级别。

例如：`android.permission.INTERNET`

* Dangerous：此权限允许应用程序执行可能影响用户隐私或用户设备正常运行的操作。安装期间可能不会授予此级别的权限，用户必须决定应用程序是否应该有这个权限。

例如：`android.permission.RECORD_AUDIO`

* Signature：只有在请求应用程序与声明该权限的应用程序使用相同的证书签名时，才会授予此权限。如果签名匹配，则自动授予权限。

例如：`android.permission.ACCESS_MOCK_LOCATION`

* SystemOrSignature：此权限仅授予系统映像中嵌入的应用程序或使用与声明该权限的应用相同的证书进行签名的应用程序。

例如：`android.permission.ACCESS_DOWNLOAD_MANAGER`

### 请求权限

应用程序可以通过在其清单中包含 `<uses-permission />` 标签来请求正常、危险和签名保护级别的权限。下面的例子是一个请求读取短信权限的 AndroidManifest.xml 示例：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.permissions.sample" ...>
<uses-permission android:name="android.permission.RECEIVE_SMS" />
<application>...</application>
</manifest>
```

### 声明权限

应用程序可以向系统上安装的其他应用程序公开功能和内容。为了限制对自己组件的访问，可以使用任何 Android 的预定义权限（<<mark class="encrypted">340K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8V1#2S2L8X3W2X3k6i4y4@1i4K6u0W2M7r3g2J5L8h3W2K6M7$3W2G2L8W2)9J5k6h3S2@1L8h3H3`.</mark>>）或定义自己的权限，使用元素 `<permission>` 声明一个新的权限。下面的例子展示了一个应用程序的权限声明：

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.permissions.sample" ...>
<permission
android:name="com.permissions.sample.ACCESS_USER_INFO"
android:protectionLevel="signature" />
<application>...</application>
</manifest>
```

上面的代码定义了一个新的权限 `com.permissions.sample.ACCESS_USER_INFO`，其保护级别为 `Signature`。任何受此权限保护的组件只能由相同开发者证书签名的应用程序访问。

### Android组件的强制权限

Android 组件可以使用权限机制来保护它们的接口。可以通过在 AndroidManifest.xml 中的相应组件标签中添加属性 `android:permission` 来在活动、服务和广播接收器使用强制权限。

```
<receiver
android:name="com.permissions.sample.AnalyticsReceiver"
android:enabled="true"
android:permission="com.permissions.sample.ACCESS_USER_INFO">
...
</receiver>
```

内容提供者有一些不同，它们支持一组单独的权限，使用内容 URI 来读取，写入和访问内容提供者。

* `android:writePermission`，`android:readPermission`：开发人员可以设置单独的读取或写入权限。

* `android:permission`：控制对内容提供者读写的一般权限。

* `android:grantUriPermissions`：如果可以使用内容URI 访问内容提供者，则为“ true”（访问暂时绕过其他权限的限制），否则为“ false”。

## 签名和发布流程

一旦应用程序成功开发，下一步就是发布分享给其他人。然而，应用程序不能简单地添加到商店并共享，它们必须首先签名，该加密签名作为应用程序开发者放置的可验证标记。它识别应用程序的作者，并确保应用程序自最初发布以来没有被修改过。

### 签名过程

在开发过程中，应用程序使用自动生成的证书进行签名，此证书本质上是不安全的并且仅用于调试。大多数商店不接受这种证书进行发布，因此，必须创建具有更安全的特性的证书。当一个应用程序安装在 Android 设备上时，包管理器会确保它是用包含在相应 APK 中的证书进行的签名。如果证书的公钥与用于设备上任何其他 APK 签名的密钥相匹配，那么新的 APK 可能与已存在的 APK 共享一个 UID。这促进了来自单个供应商的应用程序之间的交互，或者，指定的安全权限 对 Signature 保护级别来说是可能的，这将限制对使用相同密钥签名的应用程序的访问。

### APK 签名方案

Android 支持三种应用签名方案。从 Android 9（API level 28）开始，APK 可以通过 APK 签名方案 v3 、APK 签名方案 v2 或 JAR 签名（v1 方案）进行验证。对于 Android 7.0（API 等级24）及以上版本，可以通过 APK 签名方案 v2 或 JAR 签名（v1方案）对 APK 进行验证。为了向后兼容，APK 可以使用多种签名方案进行签名，以使应用程序在较新的和较旧的 SDK 版本上都运行。旧的平台会忽略 v2 签名，只验证 v1 签名（<<mark class="encrypted">71cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6S2M7r3E0K6K9h3N6F1K9h3&6Y4i4K6u0r3</mark>>）。

### JAR 签名（v1 方案）

应用程序签名的原始版本将签名的 APK 实现为标准的签名 JAR，它必须包含 `META-INF/MANIFEST.MF` 中的所有条目，所有文件都必须使用一个公共证书签名，这个方案不保护 APK 的某些部分，比如 ZIP 元数据。该方案的缺点是 APK 验证者在应用签名之前需要对不可信的数据结构进行处理，并且验证者会丢弃数据结构不包含的数据。此外，APK 验证者必须解压所有压缩文件，这需要大量的时间和内存。

### APK 签名方案（v2方案）

在 APK 签名方案中，对完整的 APK 进行哈希和签名，然后创建一个 APK 签名分块并插入到 APK 中。在验证期间，v2 方案检查整个 APK 文件的签名。这种形式的 APK 验证速度更快，并且提供了更全面的防篡改保护。你可以在下面看到 v2 方案的 APK 签名验证过程（<<mark class="encrypted">60fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6S2M7r3E0K6K9h3N6F1K9h3&6Y4i4K6u0r3N6U0u0Q4x3U0y4$3k6i4u0A6k6X3W2U0j5i4c8A6L8$3^5`.</mark>>）。

![](upload/attach/202007/802108_V277CNMNN8M8NGR.png)

apk 签名验证过程 v2 方案

### APK 签名方案（v3 方案）

v3 APK 签名块格式与 v2 相同。v3 将有关支持的 SDK 版本和 proof-of-rotation 结构的信息添加到 APK 签名分块中。在 Android 9（API 等级28）及以上版本中，可以根据APK签名方案 v3、v2 或 v1 方案对 APK 进行验证。旧的平台会忽略 v3 签名尝试验证 v2 签名，然后验证v1签名。

签名数据部分中的 proof-of-rotation 属性包含一个单链表，其中每个节点都包含用于为之前版本的应用签名的签名证书。为了实现向后兼容性，系统会让每个节点中的证书为列表中的下一个证书签名，从而为每个新密钥提供证据来证明它应该像旧密钥一样可信。单独对 APK 签名已经不可能了，因为 proof-of-rotation 结构必须使用旧的签名证书来对新的证书集签名，而不是一个一个地对它们签名。你可以在下面看到APK签名 v3 方案验证过程。（<<mark class="encrypted">a4fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6S2M7r3E0K6K9h3N6F1K9h3&6Y4i4K6u0r3N6U0x3`.</mark>>）

![](upload/attach/202007/802108_YTT6R25CT9KBWTB.png)

apk 签名验证过程 v3 方案

### 创建证书

Android 使用公共/私有证书来签名 Android 应用程序（`.apk`文件）。证书包含了大量的信息，就安全性而言，密钥是是最重要的信息类型。公共证书包含用户的公钥，私有证书包含用户的私钥。公共证书和私有证书是链接的。证书是唯一的不能重新生成，请注意，如果一个证书丢失了它就无法恢复，因此更新任何使用该证书签名的应用程序将变得不可能。应用程序创建者可以重用可用密钥存储库中现有的私有/公共密钥对，也可以生成新的密钥对。在 Android SDK 中，使用 `keytool` 命令生成一个新的密钥对。下面的命令创建一个密钥长度为2048位、过期时间为7300天即20年的 RSA 密钥对。生成的密钥对存储在当前目录中的文件 `myKeyStore.jks` 中）：

```
$ keytool -genkey -alias myDomain -keyalg RSA -keysize 2048 -validity 7300 -keystore myKeyStore.jks -storepass myStrongPassword
```

安全地存储你的密钥并确保它在整个生命周期中保持机密性是极其重要的。任何获得密钥访问权限的人都可以将你无法控制的内容（从而添加不安全的特性或使用基于签名的权限访问共享内容）更新到你的应用程序中。用户对应用程序及其开发者的信任完全基于这些证书，因此，证书保护和安全管理对于声誉和客户去留至关重要，密钥永远不能与其他个人共享。密钥存储在可以用密码保护的二进制文件中，这样的文件被称为密钥存储库。密钥存储库的密码应该是强壮的，并且只有密钥创建者知道。因此，密钥通常存储在开发人员对其访问受限的专用构建机器上。Android 证书的有效期必须超过相关应用程序（包括更新版本的应用程序），例如，Google Play 要求证书至少在2033年10月22日之前保持有效。

### 签名一个应用程序

签名过程的目标是将应用程序文件（`.apk`）与开发人员的公钥关联起来。为了实现这一点，开发人员计算 APK 文件的哈希，并用他们自己的私钥加密它。第三方可以使用作者的公钥解密加密的哈希来验证应用程序的真实性（例如，应用程序确实来自声称是发起者的用户），验证它与 APK 文件的真实哈希匹配。
