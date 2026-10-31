
# [转帖][记笔记]DroidSSLUnpinning

[原帖]：`cc8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4L8$3!0&6N6h3&6p5L8%4c8S2i4K6u0r3c8s2u0G2K9h3c8e0f1@1I4g2L8Y4m8A6L8X3&6A6L8X3M7`
[原帖]：`69aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4L8$3!0&6N6h3&6p5L8%4c8S2i4K6u0r3c8s2u0G2K9h3c8e0f1@1I4g2L8Y4m8A6L8X3&6A6L8X3M7`

## 安卓证书锁定解除的工具

**经常有朋友问我，手机安装代理证书后这个app的https流量依然抓不到明文包该如何操作，这种情况基本是遇到证书锁定了，分享下我的操作。**

- [x] 目录`JustTrustMePlus`加了些`JusTrustMe`没覆盖到的锁定场景。（基于`Xposed`模块[`justTrustMe`](<https://bbs.kanxue.com/elink@fc3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6A6L8$3^5J5y4q4)9J5c8V1A6#2M7%4c8f1M7Y4g2K6N6p5#2W2>)稍作修改）
  - 使用方法1：安装激活`Xposed`后，安装目录下提供的`apk`，勾选`jusTrustMe`模块激活重启即可。
- [ ] 目录`ObjectionUnpinningPlus`加了些`ObjectionUnpinning`没覆盖到的锁定场景。（基于`Frida`模块[`objection hook pinning`](<https://bbs.kanxue.com/elink@4e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3V1k6G2j5X3A6W2j5%4c8A6L8$3^5>)稍作修改）
  - 使用方法1 attach：`frida -U com.example.mennomorsink.webviewtest2 --no-pause -l hooks.js`
  - 使用方法2 spawn：`python application.py com.example.mennomorsink.webviewtest2`
  - 更为详细使用方法：ToBeDone 参考我的文章[`Frida.Android.Practice`](<https://bbs.kanxue.com/elink@cd5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4L8$3!0&6N6h3&6p5L8%4c8S2i4K6u0r3c8s2u0G2K9h3c8p5M7X3!0H3M7#2)9J5c8U0t1H3x3e0S2Q4x3V1j5>) 实战ssl pinning bypass章节。
- [x] `ObjectionUnpinningPlus` hook list：
  - `SSLcontext`（ART only）
  - `okhttp`
  - `webview`
  - `XUtils`（ART only）
  - `httpclientandroidlib`
  - `JSSE`
- [x] 若有没有覆盖到的场景可以联系我微博`832K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3W2T1L8#2)9J5k6h3y4G2L8g2)9J5c8X3I4#2L8$3c8A6L8X3M7I4z5e0V1I4i4K6u0W2`
- [ ] 如遇双向锁定即客户端锁定后服务端也对客户端证书验证`checkClientTrusted`，还需将证书文件导入代理软件，可能会有密码但必然会存在客户端中。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-4-28 09:38 被roysue编辑，原因：
