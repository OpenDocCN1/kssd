
# [原创]漏洞原理的梳理（不断更新中）

> **今天是5月5日，劳动节即将结束，考虑到自己对 Web 安全虽然懂一些，但是思路依然不够清晰，并且面试的时候，有些不知道怎么表达。故决定有时间就写一写，希望可以和大家一起学习，共同进步。**
>
> **也希望大家可以积极留言，有什么好的想法，一起探讨。因为这方面涉及的内容较多，需要不断添加，欢迎大家发表意见。**
>
> **[CTF 工具下载资源库](<https://bbs.kanxue.com/elink@c64K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4@1k6Y4c8G2L8$3I4K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8W2)9J5c8R3`.`.)`**

## 1. Redis 未授权访问漏洞

### 漏洞原理

`redis`，俗称中间件，用于数据库、缓存和消息中间件，默认情况下，**会绑定在 `6379` 端口**。如果没有对其进行相关的策略，就会将 `redis` 服务暴露在公网上。**如果在没有设置密码认证的情况下**，会导致任意用户在可以访问目标服务器的情况下未授权访问 `redis` 以及读取 `redis` 的数据。同理，可以通过 `redis` 提供的 `config` 命令，对文件进行写操作，将自己的公钥写入到目标服务器的 `/root/.ssh` 文件夹的 `authotrised_keys` 文件中，从而使用对应的私钥直接使用 `ssh` 服务登录目标服务器。（黑体部分，是 `redis` 漏洞成立的必须条件）

[相关漏洞解析网站](<https://bbs.kanxue.com/elink@0c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3#2B7L8$3E0W2M7W2)9J5c8Y4m8Q4x3V1j5&6y4e0b7^5z5e0j5J5i4K6u0W2K9s2c8E0L8l9`.`.)`)

附一张图片，证明可行：（左图是 `redis` 连接，并写入 shell；右图是菜刀连接）

![](https://bbs.kanxue.com/upload/attach/202105/910385_75QS4T6QKT3PR7G.jpg)![](https://bbs.kanxue.com/upload/attach/202105/910385_P8XMPQVW7CXZRVN.jpg)

### 预防措施

1. 默认只对本地开放，绑定 IP
2. 添加登录密码
3. 修改对外开放的端口
4. 用低权限来运行 `redis` 服务
5. 采用防火墙 `iptables` 来配合，限制 IP

## 2. 宽字节注入漏洞

### 漏洞原理

宽字节注入源于程序员设置 MySQL 连接时错误配置为：`set character_set_client=gbk`。正常的情况下，当 `GPC` 开启或使用 `addslashes` 函数过滤 `GET` 和 `POST` 提交的参数时，会将 `'` 转义成 `\'`；但是如果存在宽字节注入，我们输入 `%df%27` 时，首先经过上面提到的单引号转义为 `%df%5c%27`，然后在数据库查询前由于使用了 `GBK` 多字节编码，即在汉字编码范围内两个字节会被编码为一个汉字。然后 MySQL 服务器会对查询语句进行 `GBK` 编码，即 `%df%5c` 转换成了汉字“運”，而单引号逃逸了出来，从而造成了注入漏洞。

### 防御措施

1. 使用 `mysql_set_charset(utf8)` 指定字符集。
2. 使用 `mysql_real_escape_string` 进行转义。

附一张图片，证明可行：

![](https://bbs.kanxue.com/upload/attach/202105/910385_QTETZ4AJB87ARBP.jpg)

此外关于数据库注入，有一些其他疑问可以分享给大家：（其实是我之前遇到的一些面试题，当时突然遇到，自己没有讲明白）

- 如何判断自己入侵的数据库类型呢？
- 一般的 SQL 注入的流程是什么，以 MySQL 为例。
- 当我们的后台遭遇大规模的 SQL 报警，该怎么办？

（1）SQL 注入是程序开发人员在开发过程中，编程不规范，没有过滤掉一些敏感字眼，从而导致用户通过一些不合法输入，欺骗服务器执行恶意的 SQL 命令。具体的 SQL 注入，可以分为这几种：布尔型注入、报错注入、延时注入、联合查询注入等。确定数据库类型，可以大大提升我们的办事效率。常遇到的数据库有：Oracle、MySQL、SQL Server、Access、MongoDB 等。

判断方法：使用辅助的符号来判断，如注释符号 `/*` 和 `--+` 等；使用特定的函数来判断，比如 `len` 和 `length`、`version()` 和 `@@version`；利用错误信息；数据库服务器的系统变量不同。

[服务器类型分析](<https://bbs.kanxue.com/elink@935K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0l9#2y4U0p5K6y4q4)9K6c8X3k6J5L8$3#2Q4x3@1c8A6L8X3k6G2M7X3#2S2N6r3W2G2L8W2)9J5k6h3c8W2N6r3q4A6L8q4)9J5k6i4y4I4L8q4)9J5y4f1f1$3i4K6t1#2b7U0y4Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2z5o6g2Q4x3U0g2m8y4g2)9J5y4f1f1#2i4K6t1#2z5o6S2Q4x3U0g2m8y4q4)9J5y4f1f1$3i4K6t1#2z5e0k6Q4x3U0g2m8c8q4)9J5y4f1f1$3i4K6t1#2z5e0g2Q4x3U0g2n7x3q4)9J5y4f1f1$3i4K6t1#2z5p5c8Q4x3U0g2m8c8g2)9J5y4f1f1#2i4K6t1#2b7V1q4Q4x3U0f1&6x3#2)9J5y4f1f1%4i4K6t1#2b7U0q4Q4x3U0g2n7b7W2)9J5y4f1f1#2i4K6t1#2z5f1g2Q4x3U0f1^5b7R3`.`.)`)

（2）一般的 SQL 注入流程是：

- 信息收集（`nmap -T4 -A -v ip`），收集当前开放的端口；
- 找到注入页面，`and 1=1` 和 `and 1=2` 验证存在 SQL 注入；
- `order by 4`，判断字段；`id=-1 union select 1,2,3,4` 判断显错位；
- `union select 1,database(),3,4` 判断数据库类型；
- `union select 1,table_name,3,4 from information_schema.tables where table_schema='数据库' --+` 获取数据库的表名；
- `union select 1,group_concat(column_name),3,4 from information_schema.columns where table_name='表名'`，获取字段；
- `union select 1,group_concat(user),3,4 from '表名'`，获取字段信息；

也可以利用 `sqlmap` 来实现上面的自动化注入。

[sql注入的总结](<https://bbs.kanxue.com/elink@094K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7@1x3U0x3J5y4U0R3%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0l9I4z5o6R3%4x3K6k6Q4x3@1k6K6M7r3#2Q4x3@1b7I4x3o6l9I4i4K6u0W2x3U0l9I4y4q4)9J5k6e0x3H3x3o6q4Q4x3X3f1#2y4e0l9I4)`)

（3）利用后渗透的思维，当服务器发起大量的 SQL 注入的报警时，应该怎么做呢？

首先需要保证正常的业务，可以关闭一些不必要的应用，腾出资源，保证业务的正常进行。一般情况下，CPU 都有一个阈值，需要查看后台的数据包接受情况，确定是否存在丢包现象。查看日志和流量数据包，检查短时间内，数据包发送方及其发送次数，还有数据包的大概内容。对于可疑的 IP，立即封禁，并在适当情况下，对其溯源。检查防火墙以及查看攻击方常攻击的端口和服务，对于不必要的，建议关掉；然后将一些默认端口，改成其他的，并严格控制不能存在弱密码。对于一些关键核心业务，建议绑定 IP，并降低权限运行，避免攻击方获取最高权限。

## 3. JBoss 反序列化漏洞

### 基础知识

序列化就是把对象转换成字节流，便于保存在内存、文件和数据库中；反序列化，就是逆过程，由字节流还原成对象。Java 中的 `ObjectOutputStream` 类的 `writeObject()` 方法可以实现序列化，`ObjectInputStream` 类的 `readObject()` 方法用于反序列化。

### 漏洞原理

形成漏洞的根源在于类 `ObjectInputStream` 在反序列化时，**没有对生成的对象的类型做限制**。

**CVE-2017-12149**：漏洞影响 5.x 和 6.x 版本的 JBoss AS。该漏洞位于 JBoss 的 `HttpInvoker` 组件中的 `ReadOnlyAccessFilter` 过滤器中，其 `doFilter` 方法在没有进行任何安全检查和限制的情况下尝试将来自客户端的序列化数据流进行反序列化，导致攻击者可以通过精心设计的序列化数据来执行任意代码。

**CVE-2017-7504**：JBoss AS 4.x 及之前版本中，`JbossMQ` 实现过程的 JMS over HTTP Invocation Layer 的 `HTTPServerILServlet.java` 文件存在反序列化漏洞，远程攻击者可借助特制的序列化数据利用该漏洞执行任意代码。

[这方面很好的资料](<https://bbs.kanxue.com/elink@436K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7#2y4e0f1$3y4e0x3$3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6V1$3y4o6b7&6x3K6M7`.)

下图为 CVE-2017-12149 的，（左边是拼接目录，证明存在；右边是使用反序列化工具，直接得到的结果，工具见附件）

![](https://bbs.kanxue.com/upload/attach/202105/910385_GV2U9V4AJ3NPX2T.jpg)![](https://bbs.kanxue.com/upload/attach/202105/910385_T88YZW3E7NNSK2Q.jpg)

## 4. XML 外部实体注入漏洞

### 基础知识

XML 作为一个定义数据的工具或方法，使得两种不同的技术系统可以通过 XML 进行通信和交换数据。而有些 XML 文档包含 `SYSTEM` 标识符定义的“实体”，这些 XML 文档会在 `DOCTYPE` 头部标签中呈现。这些定义的“实体”能够访问本地或者远程的内容。

（左图为一般的 XML，右图是包含 `SYSTEM` 标识符的 XML）

![](https://bbs.kanxue.com/upload/attach/202105/910385_KSTTV3YBJX8ZSJB.jpg)![](https://bbs.kanxue.com/upload/attach/202105/910385_QT3FDAN3N9SUBM2.jpg)

在右图中，`entityex` 将会被 `file:///etc/passwd` 的内容替换。

### 漏洞原理

在 XML 实体中，关键字 `SYSTEM` 会令 XML 解析器从 URL 中读取内容，并允许在 XML 文档中被替换。故攻击者可以通过实体将自定义的内容发送给应用程序。XML 外部实体注入攻击利用标准可以扩展外部实体的特性，外部实体包括本地文件和远程地址，再加上实体声明中可以引用实体声明的参数。

[XML详解](<https://bbs.kanxue.com/elink@bfbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3q4U0K9$3I4A6L8$3&6Q4x3V1k6H3i4K6u0r3z5e0x3H3x3U0f1J5z5q4)9J5k6h3S2@1L8h3H3`.)

（没有找到相应的靶场，暂没有实现）

## 5. XSS 绕过新姿势

### 漏洞原理

XSS，又名跨站脚本攻击，指的是恶意攻击者往 Web 页面中插入恶意的 HTML 代码，当用户浏览该页面时，嵌入其中 Web 里面的 HTML 代码会被执行，从而达到恶意的特殊目的。XSS 属于被动式的攻击。

### XSS 的分类

反射型 XSS（把用户输入的数据发射给浏览器，非持久的）、存储型 XSS（把 XSS 脚本写入 Web 应用程序的数据库中，当用户浏览网页时，由 Web 后端程序触发 XSS 脚本）、DOM 型 XSS（通过前端 JS 将 XSS 脚本写入 DOM 节点中，从而触发 XSS）。

### 一般防御措施

对输入和 URL 参数进行过滤（黑白名单结合），对输出进行编码。简言之，就是将常见的 `<`、`>`、`"`、`;`、`alert`、`script` 等常用的 XSS 语句给过滤掉，然后输出也进行编码。

[XSS防御总结](<https://bbs.kanxue.com/elink@2b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3W2Y4k6r3g2W2M7q4)9J5c8Y4m8Q4x3V1j5@1y4U0V1#2x3K6b7^5i4K6u0W2K9s2c8E0L8l9`.`.)`)

绕过新姿势，先看图效果（远不止这样简单）：

![](https://bbs.kanxue.com/upload/attach/202106/910385_T8FVS77FE9NGDRP.jpg)

先解释一下情况，因为工作的需要，我需要对目标网站进行代码审计，然后拿到 `flag`；经过一系列的分析，发现目标网站存在 XSS 漏洞。具体的代码在工作的电脑上。经过了大量的尝试，发现这里过滤掉了几乎所有我可以想到的 XSS 脚本，也包括很多网上流行的方法。至此我决定另辟蹊径。

首先这里限制了长度，然后对 `<` 进行了转义，对 `alert()` 和其他的触发函数 `onerror` 等进行了限制，其中包括通过 `src` 进行传参的方法也不行。我决定换一种脚本，就是不使用 `alert()`，不通过弹窗来实现，构造为 `<marquee><h1>xss</h1></marquee>`。通过篡改了前端的脚本，绕过了长度的限制，进一步对脚本进行加密，尝试了多种，发现使用 HTML characters 的转码比较合适，然后在转码前使用 `”` 和 `>>`，使它原来的语句产生闭合。最后生成的语句是 `&#8221;&#62;&#62;&#60;&#109;&#97;&#114;&#113;&#117;&#101;&#101;&#62;&#60;&#104;&#49;&#62;&#120;&#115;&#115;&#60;&#47;&#104;&#49;&#62;&#60;&#47;&#109;&#97;&#114;&#113;&#117;&#101;&#101;&#62;`

有心人可以对 `”>><marquee><h1>xss</h1></marquee>` 进行尝试，然后会发现，页面会对自动解析还原并成功绕过。最后成功将 XSS 写入数据库中。

这里主要使用了 HackBar 工具的 XSS 功能，鉴于目前 HackBar 也开始收费，在附件中给大家提供了免费的 HackBar 工具 [`{4c98c9c7-fc13-4622-b08a-a18923469c1c}.xpi`](<https://bbs.pediy.com/upload/attach/202106/910385_KKQJMWSANPF593N._xpi>) ，具体的安装过程有疑问请留言，也可自行百度。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-6-9 23:31 被奋进的小杨编辑，原因：

上传的附件：

- `jboss-_CVE-2017-12149-master.zip`（2.22MB，10次下载）
- `{4c98c9c7-fc13-4622-b08a-a18923469c1c}.xpi`（107.18kb，7次下载）
