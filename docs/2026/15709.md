
# [原创] 申请 Google Map API Key 方法

## 1. 生成 SHA1 密钥

首先找到 debug keystore 位置：

打开 Eclipse → Windows → Preferences → Android → Build。

一般是这样的路径：`C:\Documents and Settings\Administrator\.android\debug.keystore`

## 2. 在 CMD 中执行

```cmd
c:\jdk\bin\>keytool -v -list -alias androiddebugkey -keystore "C:\Documents and Settings\Administrator\.android\debug.keystore"
```

## 3. 申请 Android Map 的 API Key

打开 Google APIs Console：

`db4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6M7#2)9J5c8X3y4G2L8Y4y4G2L8r3g2Q4x3V1k6Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6y4n7`

1. 创建项目并命名。
2. 在 `services` 中找到并启用 Google Map API v2。
3. 在 `api access` 中点击 `create new android key`。
4. 输入 SHA1 密钥和工程包名称，用分号隔开。

例如：

```text
SHA1;my.package.name
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
