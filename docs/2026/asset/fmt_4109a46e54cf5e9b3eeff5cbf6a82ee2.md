
```
parameters.Add("psp_tt", "0");
parameters.Add("username", userName);
parameters.Add("password", password);
parameters.Add("mem_pass", "1");
HttpWebResponse response = HttpWebResponseUtility.CreatePostHttpResponse(loginUrl, parameters, null, null, encoding, null);
string cookieString = response.Headers["Set-Cookie"];
```

```
string loginUrl = "7a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4K6M7r3!0J5N6q4)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3i4K6y4r3L8r3!0Y4K9h3^5`.";
string userName = "userName";
string password = "password";
string tagUrl = "dd3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4S2L8X3N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`."+userName+"/tags";
Encoding encoding = Encoding.GetEncoding("gb2312");
IDictionary<string, string> parameters = new Dictionary<string, string>();
parameters.Add("tpl", "fa");
parameters.Add("tpl_reg", "fa");
parameters.Add("u", tagUrl);
parameters.Add("psp_tt", "0");
parameters.Add("username", userName);
parameters.Add("password", password);
parameters.Add("mem_pass", "1");
HttpWebResponse response = HttpWebResponseUtility.CreatePostHttpResponse(loginUrl, parameters, null, null, encoding, null);
string cookieString = response.Headers["Set-Cookie"];
```

### （2）发送GET请求到HTTP站点

在`cookieString`中包含了服务器端返回的会话信息数据，从中提取了之后可以设置Cookie下次登录时带上这个Cookie就可以以认证用户的信息，假设我们已经登录成功并且获取了Cookie，那么发送GET请求的代码如下：

```
string userName = "userName";
string tagUrl = "553K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4S2L8X3N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`."+userName+"/tags";
CookieCollection cookies = new CookieCollection();//如何从response.Headers["Set-Cookie"];中获取并设置CookieCollection的代码略
response = HttpWebResponseUtility.CreateGetHttpResponse(tagUrl, null, null, cookies);
```

```
string userName = "userName";
string tagUrl = "c0aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4S2L8X3N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`."+userName+"/tags";
CookieCollection cookies = new CookieCollection();//如何从response.Headers["Set-Cookie"];中获取并设置CookieCollection的代码略
response = HttpWebResponseUtility.CreateGetHttpResponse(tagUrl, null, null, cookies);
```

### （3）发送POST请求到HTTP站点

以登录51CTO为例

```
string loginUrl = "2efK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2G2L8h3g2Q4x3X3f1#2x3h3y4@1L8#2)9J5k6h3y4G2L8g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6y4r3M7#2)9K6c8q4)9J5c8V1W2F1k6r3g2^5i4K6u0r3k6r3!0x3L8$3N6A6L8R3`.`.";
string userName = "userName";
string password = "password";
IDictionary<string, string> parameters = new Dictionary<string, string>();
parameters.Add("email", userName);
parameters.Add("passwd", password);
HttpWebResponse response = HttpWebResponseUtility.CreatePostHttpResponse(loginUrl, parameters, null, null, Encoding.UTF8, null);
```

本文只是讲解了在C#中发送请求到HTTP和HTTPS的用法，分GET/POST两种方式，为减少一些繁琐和机械的编码，将其封装为一个类，发送数据之后返回`HttpWebResponse`对象实例，利用这个实例我们可以获得服务器端返回的Cookie以便用认证用户的身份继续发送请求，或者读取服务器端响应的内容，不过在读取响应内容时要注意响应格式和编码.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
