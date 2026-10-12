
# C#下用 `HttpWebRequest` 发送 `GET`/`HTTP`/`HTTPS` 请求

> 原文：[`www.pediy.com/kssd/pediy12/133921.html`](https://www.pediy.com/kssd/pediy12/133921.html)

这个来自于我最近的一个项目，在项目中我需要将一些自己发表的和收藏整理的网文集中到一个地方存放，如果全部采用手工操作工作量大而且繁琐，因此决定利用 C#来实现。在很多地方都需要验证用户身份才可以进行下一步操作，这就免不了 `POST` 请求来登录，在实际过程中发现有些网站登录是 `HTTPS` 形式的，在解决过程中遇到了一些小问题，现在跟大家分享。

## 通用辅助类

下面是我编写的一个辅助类，在这个类中采用了 `HttpWebRequest` 中发送 `GET`/`HTTP`/`HTTPS` 请求，因为有的时候需要获取认证信息（如 `Cookie`），所以返回的是 `HttpWebResponse` 对象，有了返回的 `HttpWebResponse` 实例，可以获取登录过程中返回的会话信息，也可以获取响应流。

代码如下：

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Net.Security;
using System.Security.Cryptography.X509Certificates;
using System.DirectoryServices.Protocols;
using System.ServiceModel.Security;
using System.Net;
using System.IO;
using System.IO.Compression;
using System.Text.RegularExpressions;

namespace BaiduCang
{
    /// <summary>
    /// 有关 HTTP 请求的辅助类
    /// </summary>
    public class HttpWebResponseUtility
    {
        private static readonly string DefaultUserAgent = "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.2; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)";

        /// <summary>
        /// 创建 GET 方式的 HTTP 请求
        /// </summary>
        /// <param name="url">请求的 URL</param>
        /// <param name="timeout">请求的超时时间</param>
        /// <param name="userAgent">请求的客户端浏览器信息，可以为空</param>
        /// <param name="cookies">随同 HTTP 请求发送的 Cookie 信息，如果不需要身份验证可以为空</param>
        /// <returns></returns>
        public static HttpWebResponse CreateGetHttpResponse(string url, int? timeout, string userAgent, CookieCollection cookies)
        {
            if (string.IsNullOrEmpty(url))
            {
                throw new ArgumentNullException("url");
            }
            HttpWebRequest request = WebRequest.Create(url) as HttpWebRequest;
            request.Method = "GET";
            request.UserAgent = DefaultUserAgent;
            if (!string.IsNullOrEmpty(userAgent))
            {
                request.UserAgent = userAgent;
            }
            if (timeout.HasValue)
            {
                request.Timeout = timeout.Value;
            }
            if (cookies != null)
            {
                request.CookieContainer = new CookieContainer();
                request.CookieContainer.Add(cookies);
            }
            return request.GetResponse() as HttpWebResponse;
        }

        /// <summary>
        /// 创建 POST 方式的 HTTP 请求
        /// </summary>
        /// <param name="url">请求的 URL</param>
        /// <param name="parameters">随同请求 POST 的参数名称及参数值字典</param>
        /// <param name="timeout">请求的超时时间</param>
        /// <param name="userAgent">请求的客户端浏览器信息，可以为空</param>
        /// <param name="requestEncoding">发送 HTTP 请求时所用的编码</param>
        /// <param name="cookies">随同 HTTP 请求发送的 Cookie 信息，如果不需要身份验证可以为空</param>
        /// <returns></returns>
        public static HttpWebResponse CreatePostHttpResponse(string url, IDictionary<string, string> parameters, int? timeout, string userAgent, Encoding requestEncoding, CookieCollection cookies)
        {
            if (string.IsNullOrEmpty(url))
            {
                throw new ArgumentNullException("url");
            }
            if (requestEncoding == null)
            {
                throw new ArgumentNullException("requestEncoding");
            }
            HttpWebRequest request = null;
            //如果是发送 HTTPS 请求
            if (url.StartsWith("https", StringComparison.OrdinalIgnoreCase))
            {
                ServicePointManager.ServerCertificateValidationCallback = new RemoteCertificateValidationCallback(CheckValidationResult);
                request = WebRequest.Create(url) as HttpWebRequest;
                request.ProtocolVersion = HttpVersion.Version10;
            }
            else
            {
                request = WebRequest.Create(url) as HttpWebRequest;
            }
            request.Method = "POST";
            request.ContentType = "application/x-www-form-urlencoded";
            if (!string.IsNullOrEmpty(userAgent))
            {
                request.UserAgent = userAgent;
            }
            else
            {
                request.UserAgent = DefaultUserAgent;
            }
            if (timeout.HasValue)
            {
                request.Timeout = timeout.Value;
            }
            if (cookies != null)
            {
                request.CookieContainer = new CookieContainer();
                request.CookieContainer.Add(cookies);
            }
            //如果需要 POST 数据
            if (!(parameters == null || parameters.Count == 0))
            {
                StringBuilder buffer = new StringBuilder();
                int i = 0;
                foreach (string key in parameters.Keys)
                {
                    if (i > 0)
                    {
                        buffer.AppendFormat("&{0}={1}", key, parameters[key]);
                    }
                    else
                    {
                        buffer.AppendFormat("{0}={1}", key, parameters[key]);
                    }
                    i++;
                }
                byte[] data = requestEncoding.GetBytes(buffer.ToString());
                using (Stream stream = request.GetRequestStream())
                {
                    stream.Write(data, 0, data.Length);
                }
            }
            return request.GetResponse() as HttpWebResponse;
        }

        private static bool CheckValidationResult(object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors errors)
        {
            return true; //总是接受
        }
    }
}
```

从上面的代码中可以看出 `POST` 数据到 `HTTP` 和 `HTTPS` 站点不同，`POST` 数据到 `HTTPS` 站点的时候需要设置 `ServicePointManager` 类的 `ServerCertificateValidationCallback` 属性，并且在 `POST` 到 `https://passport.baidu.com/?login` 时还需要将 `HttpWebResquest` 实例的 `ProtocolVersion` 属性设置为 `HttpVersion.Version10`（这个未验证是否所有的 `HTTPS` 站点都需要设置），否则在调用 `GetResponse()` 方法时会抛出“基础连接已经关闭: 连接被意外关闭。”的异常。

## 用法举例

这个类用起来也很简单：

（1）`POST` 数据到 `HTTPS` 站点，用它来登录百度：

```csharp
string loginUrl = "https://passport.baidu.com/?login";
string userName = "userName";
string password = "password";
string tagUrl = "http://cang.baidu.com/" + userName + "/tags";
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



### （2）发送 GET 请求到 HTTP 站点

在`cookieString`中包含了服务器端返回的会话信息数据，从中提取了之后可以设置Cookie，下次登录时带上这个Cookie就可以以认证用户的信息。假设我们已经登录成功并且获取了Cookie，那么发送GET请求的代码如下：

```csharp
string userName = "userName";
string tagUrl = "http://cang.baidu.com/"+userName+"/tags";

CookieCollection cookies = new CookieCollection(); // 如何从 response.Headers["Set-Cookie"];中获取并设置 CookieCollection 的代码略

response = HttpWebResponseUtility.CreateGetHttpResponse(tagUrl, null, null, cookies);
```

### （3）发送 POST 请求到 HTTP 站点

以登录`51CTO`为例：

```csharp
string loginUrl = "http://home.51cto.com/index.php?s=/Index/doLogin";
string userName = "userName";
string password = "password";

IDictionary<string, string> parameters = new Dictionary<string, string>();
parameters.Add("email", userName);
parameters.Add("passwd", password);

HttpWebResponse response = HttpWebResponseUtility.CreatePostHttpResponse(loginUrl, parameters, null, null, Encoding.UTF8, null);
```

### 总结

本文只是讲解了在`C#`中发送请求到HTTP和HTTPS的用法，分GET/POST两种方式，为减少一些繁琐和机械的编码，将其封装为一个类，发送数据之后返回`HttpWebResponse`对象实例，利用这个实例我们可以获得服务器端返回的Cookie以便用认证用户的身份继续发送请求，或者读取服务器端响应的内容，不过在读取响应内容时要注意响应格式和编码。

### 附：登录参数设置示例

以下是在构建登录请求时设置参数的代码示例：

```csharp
parameters.Add("tpl_reg", "fa");
parameters.Add("u", tagUrl);
parameters.Add("psp_tt", "0");
parameters.Add("username", userName);
parameters.Add("password", password);
parameters.Add("mem_pass", "1");

HttpWebResponse response = HttpWebResponseUtility.CreatePostHttpResponse(loginUrl, parameters, null, null, encoding, null);

string cookieString = response.Headers["Set-Cookie"];
```
