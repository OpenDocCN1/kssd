
## 2. 访问原来的应用中受保护的数据，比如用户身份证书信息和隐私信息。冒充合法应用的身份还可以绕过杀软等安全解决方案。

## 3. 下载其他恶意软件的代码

- 所用工具

  `GetApkInfo.jar`

- 检测方法：

  使用命令`java -jar GetApkInfo.jar *.apk`查看APK信息及签名版本，检测是否启用`signature scheme V1 + V2`签名机制。

- 检测记录：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_9S8N7V36HHGZGWX.png)

  已采用`signature scheme V1`签名机制，未采用`V2`签名机制，存在漏洞

- 修复建议

  使用`signature scheme V1 + V2`签名机制

- 安全使用

  使用`signature scheme V1 + V2`签名机制

## WebView

`WebView`（网络视图）能加载显示网页，可以将其视为一个浏览器。它在Kitkat（android 4.4）以前使用WebKit渲染引擎加载显示网页，在Kitkat之后使用Google自家内核chromium，是Android系统中的一个View控件。对于Android操作系统来说`WebView`本身只是一个View控件，它存在的一些漏洞对于整体应用来说应该是影响不大的。但是，由于Android的发展越来越好，Android应用程序存在这升级难、更新难等客户端的通病，使得目前市面上WebApp越来越流行起来。这就使得`WebView`的漏洞影响面得到了扩大。

- 问题描述：

  1.远程代码执行漏洞

  2.UXSS漏洞

  3.密码明文存储漏洞

  4.域控制不严格漏洞

- 所用工具：

  `ApkIDE`

- 检测方法：

  分析是否使用

  `WebView` 中 `addJavascriptInterface()` 接口

  Android 4.4之前的WebKi

  `WebView` 内置导出的 `searchBoxJavaBridge_`对象

  `WebView` 内置导出的 `accessibility` 和 `accessibilityTraversalObject` 对象

- 检测记录：

  使用`ApkIDE`加载测试APP，打开一个`.smali`文件，打开java源码，查询是否使用`addjavascriptInterface`、`searchBoxJavaBridge_`接口函数，查询`WebView`默认开启密码保存功能

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_YF8RZPDMCESG6Q8.png)

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_P8PD4H78Y27ANJE.png)

- 修复建议：

  采用`@JavascriptInterface`代替`addjavascriptInterface`

- 安全使用

  1.在Android 4.2版本之前采用**拦截`prompt()`**进行漏洞修复。

  在Android 4.2版本之后采用`@JavascriptInterface`代替`addjavascriptInterface`

  2.使用`onPageStarted (WebView view, String url, Bitmap favicon)`方法在跳转前进行跨域判断

  3.使用最新的Webkit内核，但APK的size会变大，并且后续需要跟进Google Webkit官方进行更新。

  4.客户端对iframe object标签属性进行过滤

  5.删除`searchBoxJavaBridge_`接口

  6.关闭密码保存提醒：`WebSettings.setSavePassword(false)`

# 敏感信息

## 本地存储数据安全分析

存储安全分析主要就是对Android应用程序运行中所产生的文件，如数据库文件、私有文件、证书文件等，进行安全验证分析，特别是对明文存储敏感数据，导致直接被攻击者复制或篡改。

- 问题描述：

  将隐私数据明文保存在外部存储。

  将系统数据明文保存在外部存储。

  将软件运行时依赖的数据保存在外部存储。

  将软件安装包或者二进制代码保存在外部存储。

  全局可读写的内部文件存储。

- 所用工具：

  `MT管理器`

- 检测方法：

  运行apk进行一些数据操作(例如输入账号密码点击登录)，打开软件安装目录，查询是否有敏感数据。

- 检测记录：

  `MT管理器`打开根目录，打开`data`文件夹，在打开`data`文件夹内的`data`文件夹，在此文件夹内查询是否存在敏感信息

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_RY368Z5758M5NP8.png)

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_5FCE7PKTWVK57QQ.png)

  检测到敏感数据以明文形式存储在本地，存在漏洞

- 修复建议：

  加密储存敏感信息

- 安全使用

  若无必要，敏感数据禁止存储在本地，若需存储时加密存储数据，避免信息泄露。

## 操作记录检测

- 问题描述：

  当进行敏感信息操作时，若以明文显示输入数据并未检测是否被录屏、截屏，会导致敏感信息泄露

- 检测方法

  当进行敏感信息操作时，是否明文显示输入数据并检测是否被录屏、截屏，避免敏感信息泄露

- 检测记录

  检测到输入账号密码时，无法录屏、截屏，并以特殊符号显示所输入数据。

- 修复建议

  当进行敏感信息操作时，掩码显示输入数据并检测是否被录屏、截屏，避免敏感信息泄露

## 边信道信息泄漏

当应用处理用户或其它数据源输入的数据时，可能会把数据放在不安全的位置，而这些数据能够被同一设备上其它的恶意应用读取，这就造成了风险,也就是俗称的

边信道信息泄漏。

- 问题描述：

  敏感信息泄露

- 检测方法

  使用`adb logcat` 命令在终端中打印出所有的日志，查询是否有敏感信息泄露

- 检测记录

  `adb logcat >日志.txt`

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_HFPNUZSDW2JYRPQ.png)

  泄露操作信息，存在漏洞。

- 修复建议：

  对传输数据进行加密

## 传输过程中的数据被解密

- 问题描述：

  App在进行网络传输时，如果使用的是HTTPS SSL加密协议，是否对链路中的SSL加密Cert证书做了合法性的校验`checkClientTrusted`/`checkServerTrusted`等函数，以防止传输过程中的数据被解密，从而协议被逆向破解。

- 所用工具：

  `Burp suite`

- 检测方法：

  在传输数据时，进行抓包，分析传输数据是否可以解密

- 检测记录：

  设置代理，抓取数据包的结果如下：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_NHH6S9KBZE2M5DX.png)

  `https`协议传输，可抓取`https`协议传数据包

  反编译发现存在证书本地明文存储的情况，如下：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_FA9XT7JZPRRP6RW.png)

  在反编译源码中搜索证书名字，可定位获取到证书密钥，从而导致使用双向校验被绕过，如下：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_YQ5Y7VEJ3KV24DF.png)

- 修复建议：

  加密传输数据

- 安全使用

  对链路中的SSL加密Cert证书做了合法性的校验`checkClientTrusted`/`checkServerTrusted`等函数，以防止传输过程中的数据被解密，从而协议被逆向破解。

## 明文传输检测

- 问题描述：

  在与服务器端进行数据交互时，传输数据为加密，导致传输数据被拦截时泄露，敏感信息

- 所用工具：

  `Burp suite`

  检测方法：

  在与服务器端进行数据交互时，进行抓包，查看是否有敏感信息

- 检测记录：

  手机连接`Burp`代理,用`Burp`进行抓包，APP登录操作： 检测到账号、登录凭证不安全传输。

  密码虽然是MD5散列后的数值，但是传输使用的是`http`协议，存在被嗅探窃取的可能性，一旦泄漏，黑客能够直接通过该用户名和Password登录。

- 修复建议：

  加密传输数据

## 鉴权机制

### 验证码可绕过漏洞

- 问题描述：

  程序的验证码机制没有验证验证码的正确性，导致验证码名存实亡，起不到保护作用。

- 所用工具：

  `Burp suite`

- 检测方法：

  登录时，修改请求或响应信息，查询是否存在绕过问题

- 检测记录：

  经测试，App修改手机号需要验证码进行验证，如下：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_V3DTVQBJUA3EQJA.png)

  通过以下类接口，可直接对手机号进行修改，绕过验证码的验证：`6d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1k6u0e0#2c8Q4y4h3k6e0b7W2y4Q4x3V1k6S2i4K6u0r3L8i4W2Q4x3V1k6#2M7$3g2J5K9h3&6X3L8#2)9J5c8Y4m8Z5L8$3&6W2e0h3!0V1K9h3k6&6i4K6u0W2k6r3)9`.

  当前手机号：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_5JDSGRRBK2S68VQ.png)

  使用此接口直接进行修改，如下：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_FHN8AT55Y5Y3HJ9.png)

  修改成功，如下所示：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_D7BBH4SZJX73JUC.png)

- 修复建议：

  对短信验证码正确性的进行前后端验证

### 未使用有效的token机制，导致可以绕过鉴权

- 问题描述：

  如果被测应用没有使用有效的token机制，对登陆响应中的服务器返回的鉴权信息进行修改，即可绕过服务器鉴权，直接访问系统内部信息。

- 所用工具：

  `Burp suite`

- 检测方法

  从代理抓包分析，APP登录是会将一个`deviceid`和用户凭证一起提交，用户登录成功后，该`deviceid`将作为后续接口调用的凭证，即Token。

  经查，该`deviceid`为设备中某些公开信息（IMEI、设备型号、操作系统版本等）的Base64编码结果，不具备隐私性，黑客可以通过获取这些公开信息伪造用户身份，如下图所示：

  账号登录时进行抓包

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_QUMX6H4XM6KCJZ9.png)

  拦截响应信息并进行修改

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_MG62N7RRV6D2Q27.png)

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_YWW4HHUXWK9ZMKW.png)

  进行修改

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_92AAS2A237C7ZWF.png)

  查询是否登录成功，与可进行一些工程操作

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_U5FB6N8SVYY6MY5.png)

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_YTWNME73K2ANN44.png)

- 修复建议：

  使用有效果的token机制，并在`deviceid`中加入随机数据

- 安全使用

  使用有效果的token机制，并在`deviceid`中加入随机数据。

### 登录设计缺陷，存在被暴力破解风险

- 问题描述：

  用户登录过程中，未对同一用户的登录失败次数做限制，导致存在被暴力破解的风险。

- 所用工具：

  `Burp suite`

- 检测方法

  对登录接口进行暴力破解，错误50次后，输入正确用户名和密码仍然成功登录，表明没有对登录接口做暴力破解防护。

- 检测记录

  登录时，输入错误的密码，进行抓包

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_XK6FDWAGA6HZWAQ.png)

  将其发送到`Intruder`模块，进行重发包操作，

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_22UNVQWWX85PJHR.png)

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_DKKH3K5C95HV8T8.png)

  输入正确的账号密码检测是否可以正常登录

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_XAZF73QJ4EJMFTB.png)

- 修复建议：

  限制账号登录错误次数，超过阀值时需要提供验证码机制

- 安全使用

  限制账号登录错误次数，超过阀值时需要提供验证码机制。

### 利用业务逻辑缺陷制作短信炸弹

- 问题描述：

  短信发送功能模块存在逻辑问题，导致可对指定手机号码随意发送短信，造成短信炸弹攻击，可能造成用户投诉或恶意软件传播等。

- 所用工具：

  `Burp suite`

- 检测方法

  在使用短信发送功能时，进行抓包，修改请求信息，并进行多次重发包测试

- 检测记录

  用户选择注册或忘记密码功能时，会触发短信验证码发送，多次调用验证码发送接口，每分钟能收到多条短消息，且没有发现总量限制。

- 修复建议

  如果已经限制了每个用户每分钟的短信条数（如果没有限制，请做限制处理），建议进一步限制每个用户每天的短信条数。

- 安全使用

  验证短信发送人是否为指定用户，并限制一段时间段内，可发送的短信数量

### 钓鱼劫持

登录界面可被钓鱼劫持

- 问题描述：

  应用存在钓鱼劫持风险。应用程序没有做防钓鱼劫持措施，通过劫持应用程序的登录界面，可以获取用户的账号和密码，可能导致用户账号信息的泄露。

- 钓鱼原理

  需要理解，Android启动一个Activity时，是这样设计的，给Activity加入一个标志位`FLAG_ACTIVITY_NEW_TASK`，就能使它置于栈顶并立马呈现给用户。但是这样的设计却有一个缺陷。如果这个Activity是用于盗号的伪装Activity呢？这种现象在XcodeGhost事件中，已经被证实是可以实现的。

  在Android系统当中，程序可以枚举当前运行的进程而不需要声明其他权限，这样的话，就可以编写一个程序，启动一个后台的服务，这个服务不断地扫描当前运行的进程，当发现目标进程启动时，就启动一个伪装的Activity。如果这个Activity是登录界面，那么就可以从中获取用户的账号密码，具体的过程如下图：

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_4GRYT3R4AZEVCHG.png)所用工具

  `hijackactivity.apk`

- 检测记录：

  打开App劫持工具，选择需要劫持的App对应的包名，打开app检测是否被劫持

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_F2GEG7XGZ659V8R.png)

  无法被劫持不存在漏洞

  ![](https://bbs.kanxue.com/upload/attach/202009/767964_XYY3RC3EX67UNEH.png)

  修复建议：

  应用程序自身通过获取栈顶activity，判断系统当前运行的程序，一旦发现应用切换（可能被劫持），给予用户提示以防范钓鱼程序的欺诈。

- 安全使用

  建议在登录Activity的`onpause`方法中实现钓鱼劫持防护功能

  使用HTML5架构或android+HTML5混合开发，实现登陆、支付等关键页面，降低被劫持的风险。

## 服务器安全测试

对App服务器进行安全测试，检测是否存在漏洞。

# App安全评估手册-Android到此告一段落

介绍了评估环境搭建与常见评估方法，并进行了模拟实战

# 参考资料

- 《全国移动App风险监测评估报告》

- 《OWASP Mobile Top 10》

- 《移动APP渗透测试方案》

- 《APP安全测评》

- [冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-11-16 14:00

被梦幻的彼岸编辑

，原因：
