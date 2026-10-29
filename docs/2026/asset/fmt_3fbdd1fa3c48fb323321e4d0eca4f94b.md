
# [原创]安卓加密壳（`dexcrypt`），防止 `apktool`、`dex2jar` 等工具逆向你的 apk，附上下载地址

工具支持 apk 一键自动加密，`dex2jar` 与 `apktool` 都不能正常解密出代码，但是还是无法防御 IDA 的反汇编，不过已经足够了，压缩包超过 1M，使用百度网盘进行下载比较快些。

## 测试过的程序

拉卡拉 3.0.2

龙斗士 2.2.2

...

## 软件界面

软件界面如下：

![](https://bbs.kanxue.com/upload/attach/201309/596068_hjkcsan7rqr18cz.png)

![](https://bbs.kanxue.com/upload/attach/201310/596068_gik8s7jrusxugsf.png)

## 更新日志

- **2013-10-11**：国庆了，来个安卓版本吧
- **2013-9-17**：增加 Mac OS 平台版本，无须安装，双击解压 zip 文件即可。
- **2013-9-17**：增加 Linux 版本的拖放 apk 文件功能，增加状态栏加密提示。
- **2013-9-16**：增加 Linux 版本，双击 deb 文件即可安装，删除方法是在 terminal 运行 `sudo apt-get remove dexcrypt`。
- **2013-9-13**：升级小版本 1.2.1，修复 x64 位平台安装 JRE/JDK 64 位版本时程序无法自动加密的问题（感谢 gudboy 使用报告）。
- **2013-9-11**：升级 1.2 版本，启用增强型加密方法，自动判断并使用原程序中的类进行加密，如果不勾选增强型加密，则使用的是旧版本的加密模式。
- **2013-9-4**：增加修复 Java 类的相应的文件名错误功能，如 `con`、`com1`、`lpt1`、`aux` 之类的系统保留名字，这些名字将导致 `baksmali` 无法工作；此版本更新将在使用 `baksmali.jar` 工具逆向前修复这些错误的 Java 类名后再进行逆向，以增加加壳程序的兼容性。
- **2013-8-23**：支持 64 位系统问题修正（已测试）。使用原 apk 包中相应文件的压缩算法来压缩加密后的 apk 包，使生成的包大小不至于比原包大太多，一般增大几十 K。
- **2013-8-22**：正式命名为 `dexcrypt` 加密壳。使用 `baksmali.jar`、`smali.jar` 工具代替 `apktool.jar`，并增加证书自动签名功能。

## 下载地址

### Windows（2000 以后）下载地址（2013-9-13）

[点此下载](<https://bbs.kanxue.com/elink@563K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6p5I4y4K6R3&6x3K6M7I4z5o6S2Q4x3U0k6#2K9#2)9K6c8o6x3I4y4e0M7K6z5e0R3%4z5e0x3>).

### Linux（Ubuntu 9 以后）下载地址（2013-9-17）

[点此下载](<https://bbs.kanxue.com/elink@712K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6p5$3x3K6b7^5y4e0f1@1z5e0N6Q4x3U0k6#2K9#2)9K6c8o6x3I4y4e0M7K6z5e0R3%4z5e0x3>).

### Mac OS X（10.5 以后）下载地址（2013-9-17）

[点此下载](<https://bbs.kanxue.com/elink@b33K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4U0k6h3k6u0N6R3>).

### 安卓版（2.3 - 4.x，2.x 不支持签名）下载地址（2013-10-11）

[点此下载](<https://bbs.kanxue.com/elink@393K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4p5h3i4k6b7g2l9>).

## 相关链接

[\[招生\]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- `dexcrypt.png`（38.71 KB，67 次下载）
- `Screenshot_2013-10-11-17-12-05.png`（49.94 KB，45 次下载）
