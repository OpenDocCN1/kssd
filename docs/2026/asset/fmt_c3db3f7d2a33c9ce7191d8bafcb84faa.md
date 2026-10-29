
```
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

2. 在运行时向用户请求权限：查看<<mark class="encrypted">e0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6H3k6i4u0E0K9i4y4K6K9h3!0F1M7#2)9J5c8Y4u0W2M7i4g2W2M7%4c8A6L8X3N6Q4x3X3g2Z5N6r3#2D9</mark>> 了解更多细节。

3. 获得序列号：

```
String serial = android.os.Build.getSerial();
```

检索 IMEI：

1. 在你的 Android 清单中设置需要的权限：

```
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

2. 如果您使用的是 Android 6（API 等级23）或更高版本，请在运行时向用户请求权限：有关更多详细信息，请参阅<<mark class="encrypted">5e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6H3k6i4u0E0K9i4y4K6K9h3!0F1M7#2)9J5c8Y4u0W2M7i4g2W2M7%4c8A6L8X3N6Q4x3X3g2Z5N6r3#2D9</mark>>。

3. 获得 IMEI：

```
TelephonyManager tm = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
String IMEI = tm.getDeviceId();
```

### SSAID

谷歌建议不要使用这些标识符，除非应用程序处于高风险中。你可以如下检索 SSAID ：

```
String SSAID = Settings.Secure.ANDROID_ID;
```

<a href="elink@08fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5M7#2)9J5k6h3N6G2L8$3N6D9k6h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6c8Q4x3V1k6U0K9r3q4F1k6$3g2K6i4K6u0V1N6r3!0Q4x3X3c8V1k6i4k6A6j5$3g2Q4x3X3c8A6k6r3g2F1N6r3W2X3K9h3g2J5M7#2)9J5k6r3W2F1i4K6u0W2K9s2c8E0L8l9`.`.">自 Android 8.0（API 等级26）以来，SSAID 和 MAC 地址的行为已更改</a>。此外， Google SDK 文档中还提供了有关标识符的<a href="elink@0ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4g2K6k6i4u0Q4x3X3c8V1j5i4c8S2i4K6u0V1K9h3c8K6i4K6u0W2K9s2c8E0L8l9`.`.">新的建议</a>。由于这种新行为，我们建议开发人员不要仅依赖SSAID。标识符变得不稳定起来，例如恢复出厂设置后或升级到Android 8.0（API 等级26）后重新安装应用程序时，SSAID 可能会更改。有些设备具有相同的 `ANDROID_ID` 或具有的 `ANDROID_ID` 可以被覆盖。因此，最好使用从 `AndroidKeyStore` 中随机生成的密钥，用 `AES_GCM` 加密 `ANDROID_ID`。然后应将加密的 `ANDROID_ID` 存储在 `SharedPreferences` 中（私密的）。一旦应用程序签名更改，应用程序就可以检查 delta 并注册新的 `ANDROID_ID`。如果在没有新的应用程序签名密钥的情况下进行了更改，则应表明存在其他问题。

### 有效性评估

当源代码可用时，你可以寻找一些关键术语：

* 不再有效的唯一标识符：

a. 没有 `Build.getSerial` 的 `Build.SERIAL`

b. 适用于 HTC 设备的 `htc.camera.sensor.front_SN`

c. `persist.service.bdroid.bdadd`

d. 来自 `WifiManager` 的 `Settings.Secure.bluetooth_address` 或 `WifiInfo.getMacAddress`，除非清单中启用了系统权限 `LOCAL_MAC_ADDRESS`。

* `ANDROID_ID` 仅用作标识符，随着时间的流逝，这将影响旧设备的绑定质量。

* 缺少实例 ID、`Build.SERIAL` 和 IMEI。

```
TelephonyManager tm = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
String IMEI = tm.getDeviceId();
```

* 使用 `KeyPairGeneratorSpec` 或 `KeyGenParameterSpec` API 在 `AndroidKeyStore` 中创建私钥。

为了确保可以使用标识符，请检查 `AndroidManifest.xml` 中 IMEI 和 `Build.Serial` 的用法。 该文件应包含权限 `<uses-permission android：name =“ android.permission.READ_PHONE_STATE” />`。

>当 Android 8.0（API 等级26）的应用程序请求 `Build.Serial` 时，其结果将为“UNKNOWN”

有几种动态测试设备绑定的方法，见前面“使用模拟器动态分析”和“使用不同的 root 设备”。

## 引用

### OWASP MASVS

* MSTG-RESILIENCE-1：“应用程序通过提醒用户或终止应用程序来检测到已生根或越狱的设备并对其做出响应。”
* MSTG-RESILIENCE-2：“应用程序阻止调试或检测并响应所附加的调试器，必须覆盖所有可用的调试协议。”
* MSTG-RESILIENCE-3：“应用程序会检测并响应其自身沙箱中的可执行文件和关键数据的篡改。”
* MSTG-RESILIENCE-4：“应用程序可以检测并响应设备上使用广泛的逆向工程工具和框架。”
* MSTG-RESILIENCE-5：“应用程序检测并响应在模拟器中运行的情况。”
* MSTG-RESILIENCE-6：“应用程序检测并响应对其自身存储空间中的代码和数据的篡改。”
* MSTG-RESILIENCE-9：“混淆应用于编程防御，这反过来会阻碍通过动态分析去混淆。””
* MSTG-RESILIENCE-10：“应用程序通过设备的多种独特属性获取设备指纹实现了设备绑定功能。”

### SafetyNet Attestation

* Developer Guideline - <<mark class="encrypted">389K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6K6j5h3k6W2N6s2W2F1k6i4c8Q4x3V1k6S2N6s2c8W2M7%4c8S2N6r3W2G2L8W2)9J5k6h3S2@1L8h3H3`.</mark>>
* SafetyNet Attestation Checklist - <<mark class="encrypted">1a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6K6j5h3k6W2N6s2W2F1k6i4c8Q4x3V1k6S2N6s2c8W2M7%4c8S2N6r3W2G2L8W2)9J5k6r3y4Z5k6h3y4C8L8r3W2K6N6l9`.`.</mark>>
* Do's & Don'ts of SafetyNet Attestation - <<mark class="encrypted">d27K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5M7#2)9J5k6h3N6G2L8$3N6D9k6h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3e0q4Q4x3V1j5I4x3q4)9J5k6s2c8Z5K9h3&6Y4M7#2)9J5k6s2W2G2N6g2)9J5k6r3#2A6k6$3S2@1i4K6u0V1j5X3g2Q4x3X3c8V1L8$3W2F1k6#2)9J5k6s2N6J5L8$3&6Y4i4K6u0V1N6$3S2W2L8W2)9J5k6h3S2@1L8h3H3`.</mark>>
* SafetyNet Verification Samples - <<mark class="encrypted">c07K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2K6j5h3#2H3L8r3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2m8D9j5i4W2Q4x3X3c8K6j5h3k6W2N6s2W2F1k6i4c8Q4x3V1j5`.</mark>>
* SafetyNet Attestation API - Quota Request - <<mark class="encrypted">c39K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3k6$3!0G2k6$3I4W2M7r3I4S2P5g2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6U0L8$3&6@1j5h3y4@1i4K6u0r3M7$3q4X3k6i4c8&6L8X3g2@1M7i4t1`.</mark>>

### Tools

* adb - <<mark class="encrypted">909K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3j5$3!0E0L8h3q4F1k6q4)9J5k6r3I4A6L8X3g2Q4x3V1k6S2k6r3t1`.</mark>>
* Frida - <<mark class="encrypted">719K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2</mark>>
* DDMS - <<mark class="encrypted">5a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3M7s2u0G2k6X3W2D9k6g2)9J5c8X3#2G2L8X3W2@1L8%4t1`.</mark>>

原文链接：

平台概述[215K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8U0m8^5x3o6g2S2i4K6u0V1M7r3I4S2N6r3k6G2M7X3#2Q4x3X3c8G2N6X3g2J5N6X3W2W2N6H3`.`.](https://bbs.kanxue.com/elink@e9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8U0m8^5x3o6g2S2i4K6u0V1M7r3I4S2N6r3k6G2M7X3#2Q4x3X3c8G2N6X3g2J5N6X3W2W2N6H3`.`.)

Android 基础安全测试[dc7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8U0m8^5x3o6g2T1i4K6u0V1j5X3q4K6K9h3y4Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9#2k6Y4c8W2M7%4c8A6L8X3M7`.](https://bbs.kanxue.com/elink@9deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3N6A6N6r3u0G2L8$3E0Q4x3X3g2A6L8#2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6r3g2K6N6r3W2F1k6#2)9J5k6r3N6#2K9h3c8W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6s2c8W2M7%4c8A6L8X3N6Q4x3X3c8Y4N6h3W2V1k6g2)9J5c8U0m8^5x3o6g2T1i4K6u0V1j5X3q4K6K9h3y4Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9#2k6Y4c8W2M7%4c8A6L8X3M7`.)

