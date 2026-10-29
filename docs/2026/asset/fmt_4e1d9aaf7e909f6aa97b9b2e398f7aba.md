
# [原创]某 App 头像泄露凭证漏洞

懒了很久没写文章，最近对某 App 进行渗透测试。该漏洞修复后，将详情共享一下，具体是什么 App 也就不说了。

某 App 注册后需要初始化用户信息，这里可以上传头像。上传头像有个蜜汁操作：服务器返回头像地址给 App，App 再将头像地址返回给服务器，从而设置用户的头像地址（黑人问题）。这就导致我们可以设置任意地址，这里用百度地址试试能不能成功。

![](https://bbs.kanxue.com/upload/attach/201911/741401_BDJA4Q7AZ26PW2B.jpg)

服务器返回包：

![](https://bbs.kanxue.com/upload/attach/201911/741401_B9ZWTVFDG7ERVQE.jpg)

没有报错，那就成功了。那就将我们的服务器地址设置上去，别人访问到我们的头像时就会向我们服务器进行请求。随便弄了一个 PNG 图片，可以看到请求时带了一个 `uauth` 参数。

![](https://bbs.kanxue.com/upload/attach/201911/741401_HBPR6D5V2NCZ28E.jpg)

我方服务器收到数据如下：

![](https://bbs.kanxue.com/upload/attach/201911/741401_NAQBGEUTG7DM6CU.jpg)

这里的 `uauth` 参数是 JWT（JSON Web Token）。JWT 主要由三个部分组成：头部（HEADER）、载荷（PAYLOAD）、签证（SIGNATURE），使用“.”划分。这里可以直接对中间部分进行 Base64 解码，得到 `session`，明文的 `sid` 就是用户的 `session`。

![](https://bbs.kanxue.com/upload/attach/201911/741401_BPTSDJD9TTXJB3F.jpg)

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)
