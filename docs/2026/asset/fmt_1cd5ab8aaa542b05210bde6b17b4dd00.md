
```php
<?php

require_once('flag.php');

error_reporting(0);



if(!isset($_GET['msg'])){

    highlight_file(__FILE__);

    die();

}



$msg = $_GET['msg'];

if(file_get_contents($msg)!=="Hello Challenge!"){

    die('Wow so rude!!!!1');

}



echo "Hello Hacker! Have a look around.\n";



$k1=$_GET['key1'];

$k2=$_GET['key2'];



$cc = 1337;$bb = 42;



if(intval($k1) !== $cc || $k1 === $cc){

    die("lol no\n");

}



if(strlen($k2) == $bb){

    if(preg_match('/^\d+＄/', $k2) && !is_numeric($k2)){

        if($k2 == $cc){

            $cc = $_GET['cc'];

        }

    }

}



list($k1,$k2) = [$k2, $k1];



if(substr($cc, $bb) === sha1($cc)){

    foreach ($_GET as $lel => $hack){

        $$lel = $hack;

    }

}



$b="2";$a="b";

if($$a !== $k1){

    die("lel no\n");

}



// plz die now

assert_options(ASSERT_ACTIVE, 1);

assert_options(ASSERT_BAIL, 1);

eval("$bb=$cc");

assert("$bb=$cc");

echo "Good Job ;)\n";

//TODO

//echo $flag;  

?>

```

源码量还是不小的，让我们耐心地分析。

首先，程序执行了`require_once('flag.php');`。

[`require_once`](https://bbs.kanxue.com/elink@98eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2J5k6i4q4#2K9i4u0W2i4K6u0V1L8$3&6U0k6g2)9J5k6i4m8Z5M7l9`.`.)函数的定义是“`require_once`语句和`require`语句完全相同，唯一区别是PHP会检查该文件是否已经被包含过，如果是则不会再次包含。”。

[`require`](https://bbs.kanxue.com/elink@b91K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2J5k6i4q4#2K9i4u0W2i4K6u0W2M7r3S2H3)语句的作用是“`require`和`include`几乎完全一样，除了处理失败的方式不同之外。require在出错时产生`E_COMPILE_ERROR`级别的错误。换句话说将导致脚本中止而`include`只产生警告（`E_WARNING`），脚本会继续运行。”。

[`include`](https://bbs.kanxue.com/elink@519K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2A6L8X3y4D9N6h3c8W2i4K6u0W2M7r3S2H3)语句“包含并运行指定文件”。

那么什么是“包含并运行”呢？你可以理解为**把要“包含”的文件原封不动地替换到`require_once`、`require`、`include`函数所在的位置，并执行其中的php语句**。以`require_once('flag.php');`为例，`flag.php`这个文件中可能存在`$flag='xxx';`，也就是我们想夺取的flag的内容。`require_once`函数的作用是在php后端执行到`require_once`时立即将`flag.php`的内容替换`require_once('flag.php');`，并在我们当前审计的php后端中执行`flag.php`中的所有内容。在php开发中，适当使用`require_once`、`require`、`include`可以提高代码复用程度，减少代码冗余。而在Web类型的CTF比赛中，将flag单独写在`flag.php`中并通过`require_once`、`require`、`include`包含进来，可以避免在主程序中暴露flag的内容。

接着，程序执行了`error_reporting(0);`。[`error_reporting`](https://bbs.kanxue.com/elink@42dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2W2M7Y4u0G2M7W2)9J5k6s2u0W2M7r3!0J5N6r3W2F1k6#2)9J5k6i4m8Z5M7l9`.`.)可以“设置应该报告何种PHP错误”。而`error_reporting(0);`可以关闭所有PHP错误报告。这句话并不妨碍我们做题。

接着运行了如下代码段：

```php
if(!isset($_GET['msg'])){

    highlight_file(__FILE__);

    die();

}
```

[`highlight_file`](https://bbs.kanxue.com/elink@a98K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2Z5K9h3N6Z5L8r3W2Y4K9s2c8Q4x3X3c8X3K9h3I4W2i4K6u0W2M7r3S2H3)可以“语法高亮一个文件”，也就是将参数所指的文件语法高亮后发送到前端。这里用到的的参数是`__FILE__`，这是一个[魔术常量](https://bbs.kanxue.com/elink@b73K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3L8r3q4F1k6%4g2S2k6$3g2Q4x3X3g2U0L8$3&6K6N6r3q4F1N6s2y4Q4x3X3g2H3M7X3g2V1k6h3k6A6L8X3g2V1i4K6u0W2M7r3S2H3)，代表了本文件的完整路径和文件名。我们以前已经学过`isset`函数了。所以我们要做的是在URL中传入`msg`参数，这样才会不执行到`die();`。

继续往下看，运行了如下代码段：

```php
$msg = $_GET['msg'];

if(file_get_contents($msg)!=="Hello Challenge!"){

    die('Wow so rude!!!!1');

}



echo "Hello Hacker! Have a look around.\n";
```

如果你阅读过[第四篇教程](https://bbs.pediy.com/thread-249143.htm)，应该知道需要给`msg`参数传入`data://text/plain,Hello Challenge!`，才能避免执行到`die('Wow so rude!!!!1');`。但是当你在地址栏输入`` cc5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3V1y4t1k6h3I4D9L8H3`.`. Challenge! ``并按回车时，地址栏中的地址居然变成了`732K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3V1y4t1k6h3I4D9L8#2)9J5y4e0t1H3b7$3S2S2L8r3I4W2L8X3N6W2i4K6t1I4`，也就是Hello和Challenge之间的空格变成了`%20`，这是为什么呢？其实，这是浏览器把空格符号进行了URL编码（URL Encode）。如果不这么做，将违反HTTP协议，导致后端返回400 Bad Request错误，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201901/833800_6QX69XR99HCTZSR.png)

使用URL编码还有一个好处。以[第三篇教程](https://bbs.pediy.com/thread-249088.htm)为例，如果你想给`a`参数传入`123&456=`，而不进行URL编码的话，后端接收到的HTTP请求将是

```
GET /dmsj/level0/?a=123&456= HTTP/1.1
```

就会理解为传入了两个参数：`a`和`456`，这明显与我们原来的意图不符。所以，浏览器会自动对发送的表单的数据进行URL编码。

好了，现在我们访问`964K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3V1y4t1k6h3I4D9L8#2)9J5y4e0t1H3b7$3S2S2L8r3I4W2L8X3N6W2i4K6t1I4`，将会显示“Hello Hacker! Have a look around. lol no”，意味着`echo "Hello Hacker! Have a look around.\n";`已经成功执行。不过不要急，这里我还要教你一种让`file_get_contents`函数返回"Hello Challenge!"的方法，那就是使用`data://text/plain;base64,`。base64是另一种比较常用的编码，[这个网站](https://bbs.kanxue.com/elink@c78K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8G2L8$3I4Q4x3X3g2G2M7$3y4Z5K9h3&6S2i4K6u0W2L8X3g2@1i4K6u0r3k6h3&6U0M7Y4W2H3N6q4)9K6c8Y4c8&6M7r3g2Q4x3@1b7K6)可以转换明文和base64编码。我们可以将`Hello Challenge!`输入进去，将会转化成`SGVsbG8gQ2hhbGxlbmdlIQ==`，我们接着将其与`` 494K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3@1u0T1j5i4y4W2y4U0c8Q4x3V1x3`. ``拼接，得到`200K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3@1u0T1j5i4y4W2y4U0c8Q4x3V1y4e0c8#2k6K6j5V1M7^5k6#2p5J5K9r3S2T1c8%4S2D9j5X3#2V1L8p5W2c8i4K6y4p5i4K6y4p5`。当后端执行到`file_get_contents("data://text/plain;base64,SGVsbG8gQ2hhbGxlbmdlIQ==")`时，将会首先从`data://text/plain;base64,`判定这是base64编码过的数据，然后就会把`SGVsbG8gQ2hhbGxlbmdlIQ==`解码为`Hello Challenge!`，从而返回`Hello Challenge!`。

让我们继续往下看：

```php
$k1=$_GET['key1'];

$k2=$_GET['key2'];



$cc = 1337;$bb = 42;



if(intval($k1) !== $cc || $k1 === $cc){

    die("lol no\n");

}
```

程序接着将`key1`参数赋值给`$k1`，将`key2`参数赋值给`$k2`。接着设置`$cc`为1337，`$bb`为42。接下来，if语句中有`die("lol no\n");`，所以我们的目标是让if返回`FALSE`。而根据`||`的运算规则，必须让`intval($k1) !== $cc`和`$k1 === $cc`都返回`FALSE`才行。[`intval`](https://bbs.kanxue.com/elink@56cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7q4)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8Y4g2S2L8q4)9J5c8Y4A6Z5i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2A6L8Y4c8$3j5h3I4Q4x3X3g2H3K9s2l9`.)可以获取变量`$k1`的整数值。我们的目标是让`!==`这个比较返回`FALSE`，那么就要做到`intval($k1) === $cc`，而`$cc`为1337，那么我们只要传入`key1=1337`即可。由于`||`运算符的特性，当`intval($k1) !== $cc`返回`FALSE`时还会执行到`$k1 === $cc`。`===`被称为全等运算符，效果是当`$k1`等于`$cc`并且它们的类型也相同时，才会返回`TRUE`。那么，因为`$k1`是字符串类型，而`$cc`是整数类型，所以`$k1 === $cc`会返回`FALSE`，正是我们想要的。所以，我们当前的payload是：

`55fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4x3g2)9J5k6e0t1K6x3g2)9J5k6e0R3^5i4K6u0W2x3e0p5%4i4K6u0r3j5X3q4T1P5i4m8Z5M7q4)9J5c8W2)9K6c8X3#2K6k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3@1u0T1j5i4y4W2y4U0c8Q4x3V1y4e0c8#2k6K6j5V1M7^5k6#2p5J5K9r3S2T1c8%4S2D9j5X3#2V1L8p5W2c8i4K6y4p5i4K6y4p5`.

访问以上网址，显示“Hello Hacker! Have a look around. lel no”，注意这里显示的是“lel no”而不是“lol no”，说明我们顺利通过了这一关。

继续往下看：

```php
if(strlen($k2) == $bb){

    if(preg_match('/^\d+＄/', $k2) && !is_numeric($k2)){

        if($k2 == $cc){

            $cc = $_GET['cc'];

        }

    }

}
```
