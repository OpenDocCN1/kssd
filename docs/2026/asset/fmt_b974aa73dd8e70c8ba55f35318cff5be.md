
```
//Source: <9e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0r3K9$3g2&6M7%4c8G2M7X3g2Q4x3V1k6w2k6i4W2s2k6h3&6b7j5i4u0S2L8h3g2@1k6i4u0e0M7r3g2U0i4K6u0W2K9s2c8E0L8q4)9J5y4X3N6@1i4K6y4n7

KeyGenerator keyGenerator = KeyGenerator.getInstance(

     KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");

keyGenerator.init(

     new KeyGenParameterSpec.Builder("key2",

             KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)

             .setBlockModes(KeyProperties.BLOCK_MODE_GCM)

             .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)

             .build());

SecretKey key = keyGenerator.generateKey();

// The key can also be obtained from the Android Keystore any time as follows:

KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");

keyStore.load(null);

key = (SecretKey) keyStore.getKey("key2", null);
```

c. 通过 AES-GCM 加密，使用密钥对认证数据和应用程序存储的其他敏感数据进行加密，并使用实例 ID 等设备特定参数作为关联数据：

```
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

final byte[] nonce = new byte[GCM_NONCE_LENGTH];

random.nextBytes(nonce);

GCMParameterSpec spec = new GCMParameterSpec(GCM_TAG_LENGTH * 8, nonce);

cipher.init(Cipher.ENCRYPT_MODE, key, spec);

byte[] aad = "<deviceidentifierhere>".getBytes();;

cipher.updateAAD(aad);

cipher.init(Cipher.ENCRYPT_MODE, key);

//use the cipher to encrypt the authentication data see 0x50e for more details.
```

d. 使用存储在 Android 密钥存储库中的公钥加密密钥，并将加密的密钥存储在应用程序的私有存储中。

e. 当需要访问令牌等身份验证数据或其他敏感数据时，使用存储在 Android 密钥存储库中的私钥对密钥进行解密，然后使用解密后的私钥对密文进行解密。

1. 使用基于令牌的设备身份验证（实例 ID）确保使用了相同的应用程序实例。

### 静态分析

过去，Android 开发者经常依赖于 `Settings.Secure.ANDROID_ID` （SSAID）和MAC 地址。[随着 Android 8.0（API 等级26）的发布，这种情况有所改变](https://bbs.kanxue.com/elink@c5cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5M7#2)9J5k6h3N6G2L8$3N6D9k6h3u0D9L8$3N6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6c8Q4x3V1k6U0K9r3q4F1k6$3g2K6i4K6u0V1N6r3!0Q4x3X3c8V1k6i4k6A6j5$3g2Q4x3X3c8A6k6r3g2F1N6r3W2X3K9h3g2J5M7#2)9J5k6r3W2F1i4K6u0W2K9s2c8E0L8l9`.`.)。由于 MAC 地址在没有连接到接入点时通常是随机的，因此 SSAID 不再是一个绑定设备的 ID，而是一个绑定到用户、设备和请求 SSAID 的应用程序的签名密钥的值。此外，在谷歌的 SDK 文档中有[关于标识符的新建议](https://bbs.kanxue.com/elink@020K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4g2K6k6i4u0Q4x3X3c8V1j5i4c8S2i4K6u0V1K9h3c8K6i4K6u0W2K9s2c8E0L8l9`.`.)。基本上，谷歌建议：

- 涉及广告时，请使用 Advertising ID（`AdvertisingIdClient.Info`），以便用户可以选择拒绝。

- 使用Instance ID (`FirebaseInstanceId`）进行设备标识。

- 仅将 SSAID 用于欺诈检测以及由同一开发人员签名的应用之间的共享状态。

请注意，Instance ID 和 Advertising ID 在设备升级和设备重置之间并不稳定，但是 Instance ID 至少允许识别设备上当前安装的软件。

### 动态分析

有几种方法可以测试应用程序绑定。

#### 使用模拟器动态分析

1. 在模拟器上运行应用程序。

2. 确保你可以在应用程序实例中提高信任（例如在应用程序中进行身份验证）。

3. 按照以下步骤从模拟器中检索数据：

   a. 通过 `ADB shell` 以 SSH 方式进入模拟器。

   b. 运行 `run-as <your app-id>`。 你的 `app-id` 是 `Android Manifest.xml` 中描述的包。

   c. `chmod 777` 缓存和 `shared-preferences` 中的内容。

   d. 从 `app-id` 中退出当前用户。

   e. 将 `/data/data/<your appid>/cache` 的内容和 `shared-preferences` 复制到 SD 卡。

   f. 使用 `ADB` 或 `DDMS` 提取内容。

4. 在另一个模拟器上安装应用程序。

5. 在应用程序的数据文件夹中，覆盖步骤3中的数据。

   a. 将第3步中的数据复制到第二个模拟器的SD卡中。

   b. 通过 `ADB shell` 以 SSH 方式进入模拟器。

   c. 运行 `run-as <your app-id>`。 你的 `app-id` 是 `Android Manifest.xml` 中描述的包。

   d. `chmod 777` 缓存和 `shared-preferences` 中的内容。

   e. 从 `app-id` 中退出当前用户。

   f. 将 `/data/data/<your appid>/cache` 的内容和 `shared-preferences` 复制到 SD 卡。

6. 你可以继续通过身份验证吗？ 如果是这样，则绑定可能无法正常工作。

#### 使用两个不同的 root 设备

7. 在你的 root 设备上运行应用程序。

8. 确保你可以在应用程序实例中提高信任（例如在应用程序中进行身份验证）。

9. 从第一个带根的设备检索数据。

10. 在第二个 root 设备上安装应用程序。

11. 在应用程序的数据文件夹中，用步骤3中的数据覆盖。

12. 你可以继续通过身份验证吗？ 如果是这样，则绑定可能无法正常工作。

### Google Instance ID

[谷歌 Instance ID](https://bbs.kanxue.com/elink@46dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9h3&6K6N6r3q4F1j5$3g2Q4x3X3c8A6k6q4)9J5c8R3`.`.) 使用令牌对正在运行的应用程序实例进行身份验证，当应用程序被重置、卸载等时候，实例 ID 被重置，这意味着你将拥有该应用程序的新“实例”。 通过以下步骤获取 Instance ID：

1. 在 Google Developer Console 中为给定应用程序配置 Instance ID，这包括管理 `PROJECT_ID`。

2. 设置 Google Play 服务，在文件 `build.gradle` 中，添加

```
apply plugin: 'com.android.application'

  ...

  

  dependencies {

      compile 'com.google.android.gms:play-services-gcm:10.2.4'

  }
```

3. 获得一个 Instance ID

```
String iid = Instance ID.getInstance(context).getId();

//now submit this iid to your server.
```

4. 生成令牌。

```
String authorizedEntity = PROJECT_ID; // Project id from Google Developer Console
String scope = "GCM"; // e.g. communicating using GCM, but you can use any
                  // URL-safe characters up to a maximum of 1000, or
                  // you can also leave it blank.
String token = Instance ID.getInstance(context).getToken(authorizedEntity,scope);
//now submit this token to the server.
```

5. 确保在无效的设备信息，安全性问题等情况下，可以处理来自 Instance ID 的回调，这需要扩展 `Instance IDListenerService` 并在那里处理回调：

```
public class MyInstance IDService extends Instance IDListenerService {

public void onTokenRefresh() {
  refreshAllTokens();
}

private void refreshAllTokens() {
  // assuming you have defined TokenList as
  // some generalized store for your tokens for the different scopes.
  // Please note that for application validation having just one token with one scopes can be enough.
  ArrayList<TokenList> tokenList = TokensList.get();
  Instance ID iid = Instance ID.getInstance(this);
  for(tokenItem : tokenList) {
    tokenItem.token =
      iid.getToken(tokenItem.authorizedEntity,tokenItem.scope,tokenItem.options);
    // send this tokenItem.token to your server
  }
}

};
```

6. 在你的 Android清单中注册服务：

```
<service android:name=".MyInstance IDService" android:exported="false">
<intent-filter>
      <action android:name="com.google.android.gms.iid.Instance ID" />
</intent-filter>
</service>
```

当你向服务器提交 Instance ID（`iid`）和令牌时，你可以通过 Instance ID 云服务来使用服务器验证令牌和 `iid`。当 `iid` 或令牌无效时，你可以触发一个保护程序（例如，通知服务器可能的复制或安全问题，或者从应用程序中删除数据并要求重新注册）。

请注意，[Firebase 也支持 Instance ID](https://bbs.kanxue.com/elink@7a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0W2j5X3q4K6k6g2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4y4Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0r3j5$3!0E0i4K6u0r3k6$3!0G2k6$3I4W2i4K6u0r3k6X3W2J5k6h3u0S2M7$3g2Q4x3V1k6A6K9h3c8Q4x3V1k6r3K9i4u0W2j5X3q4K6k6f1W2F1M7%4c8S2L8X3y4W2d9h3b7`.)。

### IMEI & Serial

谷歌建议不要使用这些标识符，除非应用程序处于高风险中。

在 Android 8.0（API 等级26）之前的 Android 设备，可以请求如下序列号：

```
   String serial = android.os.Build.SERIAL;
```

对于运行 Android 版本 O 及更高版本的设备，可以通过下面方式请求设备的序列号：

1. 在你的 Android 清单中设置权限：

```
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

2. 在运行时向用户请求权限：查看[dbfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6H3k6i4u0E0K9i4y4K6K9h3!0F1M7#2)9J5c8Y4u0W2M7i4g2W2M7%4c8A6L8X3N6Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@926K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6H3k6i4u0E0K9i4y4K6K9h3!0F1M7#2)9J5c8Y4u0W2M7i4g2W2M7%4c8A6L8X3N6Q4x3X3g2Z5N6r3#2D9) 了解更多细节。

3. 获得序列号：

```
String serial = android.os.Build.getSerial();
```

检索 IMEI：

4. 在你的 Android 清单中设置需要的权限：

```
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```
