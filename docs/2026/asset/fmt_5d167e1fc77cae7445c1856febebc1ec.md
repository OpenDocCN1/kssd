
如果未使用 `--password` 选项，`authpass` 将提示输入密码。`realm` 定义了一组密码，并通过 `AuthType` 指令设置，通常设置为公司域或产品名称。

注：身份验证文件必须存储在 `Documents` 目录或任何提供内容的目录之外。

#### 2. 认证协议

认证协议定义了如何从用户处捕获用户凭据并提供给 Appweb，Appweb 提供了不同的认证协议：

- [Application Authentication](https://bbs.kanxue.com/elink@81cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8%4k6h3u0Q4x3V1k6V1L8$3y4Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4#2N6r3S2W2L8Y4c8A6j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8q4)9J5x3$3q4H3M7l9`.`.)

  应用程序身份验证使用特定于应用程序的方式来捕获用户名和密码。用户应用程序负责实现控制逻辑以捕获用户凭据并通过登录和注销过程重定向客户端。此协议最适用于使用服务器端 Web 框架（如 ESP）的应用程序。

- [Web Form Authentication](https://bbs.kanxue.com/elink@159K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8%4k6h3u0Q4x3V1k6V1L8$3y4Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4#2N6r3S2W2L8Y4c8A6j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8q4)9J5x3$3k6G2M7X3@1`.)

  表单身份验证方案使用 HTML Web 表单让用户输入用户名和密码凭据，并使用 HTTP `Post` 请求将凭据提交给服务器进行验证。使用此协议，可以通过 `AuthType` 指令定义特定的登录和注销页面。Appweb 管理登录/注销过程，如果身份验证成功，则会创建登录会话并将 cookie 返回到客户端的浏览器，包含 cookie 的后续请求将被自动验证并提供服务。

  要配置表单身份验证，`AuthType` 指令需要额外的参数来管理登录序列，包括指定登录网页、登录服务 URL、注销服务 URL、验证后显示的目标页面和注销后显示的目标页面。格式为：

```
AuthType form realm Login-Page Login-Service Logout-Service Logged-In-Destination Logged-Out-Destination
```

  该指令定义在登录序列期间使用的 URL。该指令根据这些 URL 的要求创建请求路由，允许未经身份验证的用户访问登录页面和登录服务。这些 `AuthType` 参数中的每一个都是可选的，可以指定为空字符串 `""` 以省略。

  例如：

```
<Route ^/>
  AuthType form example.com
  /public/login.html
  /login
  /logout
  //public/login.html
</Route>

<Route ^/public/>
  Profix /public
  Document public
  AuthType none
</Route>
```

  此示例为所有请求启用表单身份验证，并将客户端浏览器重定向到 `/public/login.html`，用户可以输入用户名和密码。登录网页应将用户名和密码提交给绑定到 `/login` URL 的登录服务。当需要注销时，客户端应向绑定到 `/logout` URL 的注销服务提交 HTTP POST 请求。`AuthType` 指令中的最后两个字段是客户端浏览器在登录和注销后将重定向到的目标 URL。第二个 `/public` 路由无需身份验证即可访问“public”目录下的文档。

  `Login-Service` 是绑定到内部服务的 URL，用于接收用户名和密码并对用户进行身份验证。此服务期望使用输入字段“用户名”和“密码”通过 POST 数据提交用户名/密码。可以通过在 AuthType 指令中为 `Login-Service` 指定空字符串 `""` 来提供自定义的登录和注销服务。如果使用自定义的登录服务，则应该调用 `httpLogin` 以根据配置的密码存储验证用户。

  **Web Form**

  这是一个最小示例登录页面：

```
<html>
<head><title>login.html</title></head>
<body>
<p>Please log in</p>
<form name="details" method="post" action="/auth/login">
username <input type="text" name="username" value=''><br/>
password <input type="password" name="password" value=''><br/>
<input type="submit" name="submit" value="OK">
</form>
</body>
</html>
```

  提交的两个字段必须命名为 `username` 和 `password` 以供“表单”身份验证方案使用。

  如果登录尝试成功，客户端将收到包含会话 cookie 的响应，并将被重定向到目标 URL。如果目标 URL 包含一个 `referrer:` 前缀并且登录请求在 HTTP 标头中包含一个引用 URL，那么该引用 URL 将用作目标而不是硬连接目标。

  注：“表单”身份验证机制以纯文本形式提交用户密码。为确保通信安全，应在使用 TLS/SSL 的安全连接上使用“表单”身份验证方案。

- [Basic Authentication](https://bbs.kanxue.com/elink@a48K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8%4k6h3u0Q4x3V1k6V1L8$3y4Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4#2N6r3S2W2L8Y4c8A6j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8q4)9J5x3$3u0S2M7$3W2U0)

- [Digest Authentication](https://bbs.kanxue.com/elink@decK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8%4k6h3u0Q4x3V1k6V1L8$3y4Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4#2N6r3S2W2L8Y4c8A6j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8q4)9J5x3$3c8A6k6$3g2K6N6l9`.`.)

  Basic 和 Digest 身份验证是 HTTP/1.1 RFC2616 规范定义的 HTTP 协议机制。因为它们在 HTTP 协议级别运行，所以功能简单，且灵活性差。当客户端尝试访问受保护的内容时，客户端的浏览器会显示一个通用弹出对话框以提示用户输入凭据。

应该只将基本和摘要式身份验证用作最后的手段。Basic 和 Digest 身份验证标准使用弱密码，通过网络重复发送凭据，并且不够安全。基本身份验证在每个请求中以明文形式传输密码。摘要式身份验证使用弱 MD5 密码，并且两者都要求对所有请求使用 SSL 以确保最低限度的安全。此外，Basic 和 Digest 都不提供可靠的注销机制。注销适用于某些浏览器，但不适用于其他浏览器甚至同一浏览器的不同版本。

Appweb 身份验证框架十分全面，使开发人员不必将各个部分进行拼凑组合成一个认证方案。更为详细的内容可以参考 [Authentication](https://bbs.kanxue.com/elink@67dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8%4k6h3u0Q4x3V1k6V1L8$3y4Q4x3V1k6#2M7$3g2J5M7#2)9J5c8X3q4#2N6r3S2W2L8Y4c8A6j5$3q4@1K9h3!0F1i4K6u0W2K9s2c8E0L8l9`.`.)。

#### 3. web框架

如果使用 ESP、PHP 或其他 Web 框架，则不应将扩展的 `AuthType form` 指令与 URL 一起使用。这是因为这些 web 框架集成了登录工具，在 web 框架中使用起来更自然。扩展的 `AuthType form` 指令适合使用静态网页的网站，因为它可以在登录期间无缝管理浏览器页面转换。使用 ESP Web 框架时，请选择使用 `AuthType app` 身份验证协议。

#### 4. SSL加密

在考虑对发送或接收来自应用程序的敏感信息时，有两种基本的安全策略：

- 保证整个应用程序的安全

- 只保证包含登录页和登录服务在内的敏感信息的安全

Appweb 通过一种简单的方式保证整个应用的安全 -- 使用 `Redirect` 指令。这适用于 SSL 编码开销最小的更快的系统（在 PC 类系统上小于 5%）。但是对于更普通的嵌入式 CPU，SSL 加密开销可能会大得惊人，尤其是在使用大密钥时。

```
Redirect secure
```

这将自动通过 SSL 重定向所有请求，并且可能用于整个服务器或仅用于特定请求路由。

另外一种方法是仅保护登录页和登录/注销服务，通过在 `AuthType` URLs 的前面加上 `https://` 前缀即可：

```
AuthType form example.com https:///public/login.html https:///login /logout http:///
```

在登录时，会使用 SSL 加密，登录成功后会切换会 http。

如果要保护应用程序的其他部分，可以定义重定向的路由：

```
<Route ^/sensitive/>
Prefix /sensitive
Documents top-secret
Redirect secure
</Route>
```

### 4. Embedded Server Pages

ESP web Framework 可以说是 Appweb 中最优秀的部分，它是一个典型的 MVC 框架，可以容易地创建快速、动态的 web 应用。ESP 是一个基于 C 的 web 框架，但不像一般的 C 编码，ESP 框架的 web 页面支持嵌入式 C 编码。甚至，它支持网页修改时动态透明的重新编译和重新加载。它还检测并自动重新编译修改后的 ESP 控制器（用 C 编写）。这使得 ESP 的行为就像一个脚本化的 Web 框架。

为了解决 C 语言常见的内存分配和内存泄漏问题，ESP 使用了垃圾回收器在请求处理完成后来自动释放内存。

ESP 框架提供了一个应用生成器、web 页面模板引擎、MVC 框架、 HTML 控制库和 API 扩展来创建 web 应用程序。

![esp](https://bbs.kanxue.com/upload/attach/202106/779730_668RRY4C3GKWBW3.jpg)

ESP 应用程序通常定义在一组目录中：

- `cache` — 缓存预编译的 ESP 控制器和页面

- `client` — 客户端 web 页内容、图片、样式等

- `controllers` — ESP 控制器方法

- `layouts` — 主页面布局

- `db` — 数据库和数据库迁移

可以使用以下命令来快速创建一个新的 ESP 应用程序：

```
mkdir blog
cd blog
esp install esp-html-mvc
```

关于 ESP 框架的内容十分丰富，可以单独拿出来详细说明，给出官网文档 [ESP Docs](https://bbs.kanxue.com/elink@1dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8h3u0W2k6s2c8Z5K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2M7%4m8Q4x3V1k6V1L8$3y4Q4x3V1j5`.)。

## 五、安全性

web server 通常通过数量众多的安全测试来保证安全性，但是作用甚微，构建一个设计安全的 web server 远比通过测试保证安全的方式更有效。对于嵌入式 web server 更难保证安全性，因为要在低内存占用和不降低性能的前提下进行。

Appweb 通过使用一个安全的 Portable Runtime (MPR)， 从一开始就重点保证安全性。MPR 是一个跨平台层，它使得 Appweb 的超过97%的代码都是可移植的。它包括许多帮助创建安全应用的机制，比如它包括一个安全的字符串和缓冲区处理模块，以帮助消除困扰许多产品的缓冲区溢出问题。

## 六、沙箱

Appweb 引入沙箱来严格控制对系统资源的占用。这意味着在严格控制的范围内运行 Web 服务器，以便请求错误不会影响系统操作。Appweb 还针对几种常见的拒绝服务攻击进行了加固。

Appweb 根据不同需求可做如下配置：

- 限制内存使用并且不允许超过内存限制的预定义的数值

- 拒绝过大的请求

- 拒绝过长的 URL

- 由指定的用户帐户或用户组运行

基于以上基础，Appweb 提供了 Secure Sockets Layer 和摘要认证以及防御策略。

## 参考

大部分内容来自 Appweb 官方文档

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2021-6-4 10:29

被有毒编辑

，原因：
