
- 如果叶证书的有效期恰好延长了一定时间（Chrome 为 39 个月），某些应用程序（例如[Chrome 浏览器](https://bbs.kanxue.com/elink@4e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6b7%4y4e0M7@1y4b7`.`.)）可能会显示`NET::ERR_CERT_VALIDITY_TOO_LONG`错误。如果使用默认 Burp CA 证书，则会发生这种情况，因为 Burp Suite 颁发的叶证书与其 CA 证书具有相同的有效性。您可以通过创建自己的 CA 证书并将其导入 Burp Suite 来规避此问题，如本[博客](https://bbs.kanxue.com/elink@1a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8Y4k6A6M7$3!0Q4x3X3g2T1k6g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3g2)9J5c8U0x3I4i4K6u0r3N6i4y4A6L8X3N6Q4x3X3c8S2i4K6u0V1j5%4g2K6N6r3!0E0i4K6u0V1M7X3!0G2N6q4)9J5k6r3y4S2i4K6u0V1N6$3W2@1K9q4)9J5k6r3u0#2M7Y4m8Q4x3X3c8X3L8%4u0Q4x3X3c8A6L8Y4y4H3k6h3y4@1K9h3&6Y4i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6r3&6Q4x3X3c8@1M7X3q4X3k6X3W2U0i4K6u0r3)中所述。

完成这些步骤并启动应用程序后，请求应显示在拦截代理中。

> 可以在[secure.force.com](https://bbs.kanxue.com/elink@275K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4y4W2j5%4g2J5k6g2)9J5k6h3k6G2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6@1L8$3!0D9M7#2)9J5c8Y4N6W2j5X3q4H3M7q4)9J5c8Y4A6S2M7r3q4F1k6s2u0G2K9h3c8K6k6i4c8#2M7l9`.`.)上找到使用 Android 设备设置[OWASP ZAP](https://bbs.kanxue.com/elink@eebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5i4m8H3k6h3&6V1K9i4S2Q4x3V1j5H3P5o6l9^5i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6s2c8G2L8$3I4K6i4K6t1K6L8%4N6S2M7%4m8Q4x3X3c8*7j5i4l9`.)的视频。

其他一些差异：

- 从 Android 8.0（API 级别 26）开始，当 HTTPS 流量通过另一个连接进行隧道传输时，应用程序的网络行为会发生变化。
- 从 Android 9（API 级别 28）开始，当握手期间出现问题时，`SSLSocket` 和 `SSLEngine` 在错误处理方面的表现会有所不同。

如前所述，**从 Android 7.0（API 级别 24）开始，Android 操作系统将默认不再信任用户 CA 证书**，除非在应用程序中指定。在下一节中，我们将解释绕过此 Android 安全控制的两种方法。

### 绕过网络安全配置

从 Android 7.0（API 级别 24）开始，网络安全配置允许应用通过定义应用信任的 CA 证书来自定义其网络安全设置。

为了实现应用程序的网络安全配置，您需要创建一个名为 `.xml` 的新 xml 资源文件`network_security_config.xml`。这在[Android 网络安全配置培训](https://bbs.kanxue.com/elink@5b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1j5$3!0F1k6X3W2Y4)中有详细说明。

创建后，应用程序还必须在清单文件中包含一个条目以指向新的网络安全配置文件。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `<?xml version="1.0" encoding="utf-8"?>`  `<manifest ... >`  `<application android:networkSecurityConfig="@xml/network_security_config"`  `... >`  `...`  `<\/application>`  `<\/manifest>` |

网络安全配置使用一个 XML 文件，应用程序在该文件中指定哪些 CA 证书是受信任的。有多种方法可以绕过网络安全配置，下文将对此进行介绍。另请参阅[Android P 中的网络安全配置安全分析师指南以](https://bbs.kanxue.com/elink@0adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1L8%4N6K6k6h3y4#2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9^5i4K6u0r3x3e0g2Q4x3V1k6S2i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8S2L8X3q4D9P5i4y4@1M7#2)9J5k6r3N6#2K9h3c8W2i4K6u0V1N6r3!0Q4x3X3c8F1k6i4c8%4L8%4u0C8i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8U0L8$3&6X3K9h3N6#2M7X3q4@1K9h3!0F1i4K6u0V1K9h3&6Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1M7q4)9J5c8R3`.`.)获取更多信息。

#### 将用户证书添加到网络安全配置

网络安全配置有不同的配置可用于通过 `src` 属性[添加非系统证书颁发机构：](https://bbs.kanxue.com/elink@715K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1j5$3!0F1k6X3W2Y4i4K6t1K6b7%4g2K6N6r3!0E0g2s2u0#2M7%4b7`.)

|  |  |
| --- | --- |
| 1  2 | `<certificates src=["system" | "user" | "raw resource"]`  `overridePins=["true" | "false"] />` |

每个证书可以是以下之一：

- 指向包含 X.509 证书文件的“raw resource”ID
- `"system"` 用于预安装的系统 CA 证书
- `"user"` 用于用户添加的 CA 证书

应用信任的 CA 证书可以是系统信任的 CA，也可以是用户 CA。通常，您已经将拦截代理的证书添加为 Android 中的附加 CA。因此，我们将重点关注“用户”设置，它允许您通过以下网络安全配置强制Android 应用信任此证书：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `<network-security-config>`  `<base-config>`  `<trust-anchors>`  `<certificates src="system" />`  `<certificates src="user" />`  `<\/trust-anchors>`  `<\/base-config>`  `<\/network-security-config>` |

要实施此新设置，您必须按照以下步骤操作：

- 使用 `apktool` 之类的反编译工具反编译应用程序：

|  |  |
| --- | --- |
| 1 | `$ apktool d <filename>.apk` |

- 通过创建包含`<certificates src="user" />`如上所述的网络安全配置来使应用程序信任用户证书
- 在反编译应用程序时进入`apktool`创建的目录并使用`apktool`重新构建应用程序。新的 apk 将在`dist`目录中。

|  |  |
| --- | --- |
| 1 | `$ apktool b` |

- 您需要重新打包应用程序，如“逆向工程和篡改”一章的“[重新打包](https://bbs.kanxue.com/elink@5f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8U0m8^5x3o6g2U0i4K6u0V1M7X3g2$3k6i4u0K6k6g2)9J5k6r3g2F1k6$3W2F1k6h3g2J5K9h3&6Y4i4K6u0V1j5h3&6V1i4K6u0V1N6r3q4E0M7r3g2J5K9h3&6Y4i4K6t1K6M7X3g2H3j5h3y4C8j5h3N6A6L8X3M7`.)”部分所述。有关重新打包过程的更多详细信息，您还可以查阅[Android 开发者文档](https://bbs.kanxue.com/elink@1f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3M7s2g2T1L8r3W2K6K9q4)9J5c8X3q4H3M7q4)9J5k6s2y4A6k6$3&6A6L8X3N6Q4x3U0y4K6K9h3N6F1K9h3&6Y4i4K6u0V1L8h3q4F1N6h3q4D9L8s2V1`.)，该文档对整个过程进行了说明。

请注意，即使此方法非常简单，但它的主要缺点是您必须为要评估的每个应用程序执行此操作，这是测试的额外开销。

> 请记住，如果您正在测试的应用程序具有其他强化措施，例如验证应用程序签名，您可能无法再启动该应用程序。作为重新打包的一部分，您将使用自己的密钥对应用程序进行签名，因此签名更改触发应用程序立即终止。您需要通过在重新打包应用程序期间修补它们或通过 Frida 进行动态检测来识别和禁用此类检查。

有一个可用的 python 脚本可以自动执行上述步骤，称为[Android-CertKiller](https://bbs.kanxue.com/elink@f8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5#2x3h3Z5H3i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6p5y4W2M7Y4c8w2K9h3I4D9k6i4t1`.)。此 Python 脚本可以从已安装的 Android 应用程序中提取 APK，对其进行反编译，使其可调试，添加允许用户证书的新网络安全配置，构建和签名新 APK，并使用 SSL 绕过安装新 APK。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38 | `python main.py`  `-w`    `***********************`  `Android CertKiller (v0.1)`  `***********************`    `CertKiller Wizard Mode`  `-------------------------`  `List of devices attached`  `4200dc72f27bc44d`    `device`    `-------------------------`    `Enter Application Package Name: nsc.android.mstg.owasp.org.android_nsc`    `Package: /data/app/nsc.android.mstg.owasp.org.android_nsc-1/base.apk`    `I. Initiating APK extraction from device`  `complete`  `-------------------------`  `I. Decompiling`  `complete`  `-------------------------`  `I. Applying SSL bypass`  `complete`  `-------------------------`  `I. Building New APK`  `complete`  `-------------------------`  `I. Signing APK`  `complete`  `-------------------------`    `Would you like to install the APK on your device(y/N): y`  `-------------------------`  `Installing Unpinned APK`  `-------------------------`  `Finished` |

#### 使用 Magisk 在系统信任的 CA 中添加代理的证书

为了避免为每个应用程序配置网络安全配置的义务，我们必须强制设备接受代理的证书作为系统受信任的证书之一。
