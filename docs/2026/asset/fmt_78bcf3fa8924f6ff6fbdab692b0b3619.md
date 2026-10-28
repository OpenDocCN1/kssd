
了解了流程之后，我们看一下`encrypt`传了什么，可以看到传了本机的用户名，以及文件的信息，到服务器端

![](https://bbs.kanxue.com/upload/attach/202011/873515_2MM7QSPZBBGSNR6.png)

![](https://bbs.kanxue.com/upload/attach/202011/873515_AB9UH4A4WA65GWG.png)

使用了`RSA`和`AES`算法上传到服务器

![](https://bbs.kanxue.com/upload/attach/202011/873515_NCZ4S84EE6GNMGA.png)

![](https://bbs.kanxue.com/upload/attach/202011/873515_DKVV3EMASPS7MSQ.png)

![](https://bbs.kanxue.com/upload/attach/202011/873515_WVNEDBPTSCB3ATN.png)

![](https://bbs.kanxue.com/upload/attach/202011/873515_4GZQP72C5DN5TQ3.png)

![](https://bbs.kanxue.com/upload/attach/202011/873515_KZYXF2UAUWRE882.png)

### `InternetReadFile`：
![](https://bbs.kanxue.com/upload/attach/202011/873515_EYM243EX6F3KRBU.png)
同样他会下载东西到本机里，这里可能`C2`服务器已经失效，文件的大小是`0`

![](https://bbs.kanxue.com/upload/attach/202011/873515_UWETKMWR79HJYXE.png)

`C2`服务器 通过硬编码的方式，和`C2`服务器进行交互：

![](https://bbs.kanxue.com/upload/attach/202011/873515_CSTDEZX4QYUAWEJ.png)

### C2 服务器地址：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `216.10`.`40.16`  `91.121`.`54.71`  `209.236`.`123.42`  `77.55`.`211.77`  `85.105`.`140.135`  `138.97`.`60.141`  `217.13`.`106.14`  `190.2`.`31.172` |

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
