
# HTTP POST 实现自动登录网站

> 原文：[`www.pediy.com/kssd/pediy12/115147.html`](https://www.pediy.com/kssd/pediy12/115147.html)

作者：ssLong

使用 HTTP POST 提交表单实现登录网站功能

首先使用抓包工具得到网站登录时提交的 POST 数据，例如抓到的登录校内网 `http://ipgw.neu.edu.cn/` 的数据如下：

```
POST /ipgw/ipgw.ipgw HTTP/1.1

Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/x-shockwave-flash, application/msword, application/vnd.ms-excel, application/vnd.ms-powerpoint, application/x-silverlight, application/x-ms-application, application/x-ms-xbap, application/vnd.ms-xpsdocument, application/xaml+xml, */*

Referer: http://ipgw.neu.edu.cn/

Accept-Language: zh-cn

Content-Type: application/x-www-form-urlencoded

Accept-Encoding: gzip, deflate

User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; 360SE)

Host: ipgw.neu.edu.cn

Content-Length: 70

Connection: Keep-Alive

Cache-Control: no-cache

Cookie: JServSessionIdipgw=ikac52afz1

uid=你的用户名&password=你的密码&range=2&operation=disconnect&timeout=1
```

可以看到用户名和密码。

因此可以模拟一个登录的程序如下：

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Net;
using System.IO;
using System.Web;

namespace CheckWeb
{
    class CUsrPassword
    {
        /// <summary>
        /// 使用用户名和密码登录 strUrl
        /// </summary>
        /// <param name="strUrl">post 的网址</param>
        /// <param name="strUser">用户名</param>
        /// <param name="strPassword">密码</param>
        /// <returns>true 登录成功</returns>
        public static bool PostUserPassword(String strUrl, String strUser, String strPassword)
        {
            try
            {
                strUrl = "http://ipgw.neu.edu.cn/ipgw/ipgw.ipgw";  // 必须为提交的路径
                HttpWebRequest req = (HttpWebRequest)WebRequest.Create(strUrl);
                String sendData = "uid=你的用户名&password=你的密码&range=2&operation=connect&timeout=1";  // 待发送的数据
                ASCIIEncoding encoding = new ASCIIEncoding();
                byte[] requestBytes = encoding.GetBytes(sendData);// System.Text.Encoding.Default.GetBytes(sendData);
                req.Method = "POST";
                req.Referer = "http://ipgw.neu.edu.cn/";
                req.ContentType = "application/x-www-form-urlencoded";
                req.ContentLength = requestBytes.Length;
                req.UserAgent = "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; SV1; .NET CLR 2.0.1124)";
                Stream requestStream = req.GetRequestStream(); // 获得输出流
                if (requestStream == null)
                {
                    String str = "failed";
                }
                requestStream.Write(requestBytes, 0, requestBytes.Length); // 向目标地址写入编码后的数据
                requestStream.Close(); // 关闭流
                // 接收返回参数到 string backstr
                HttpWebResponse res = (HttpWebResponse)req.GetResponse();
                StreamReader sr = new StreamReader(res.GetResponseStream(), System.Text.Encoding.Default);
                String backstr = sr.ReadToEnd();
                sr.Close();
                res.Close();
            }
            catch (Exception exp)
            {
                String str = exp.Message;
            }
            return true;
        }
    }
}
```
