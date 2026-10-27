
# Authenticode签名对未签名代码的应用

众所周知，攻击者将合法的数字证书应用于其恶意软件，这大概是为了躲避基本的签名验证程序。`Petya勒索病毒`便是这种情况。作为一名逆向工程师或`red team`的开发者，了解哪些合法的签名可以被应用到其他的未签名代码以及攻击者提供的代码的方法。这篇博客将会给出代码签名机制、二进制数字签名，以及对数字证书应用于未签名的PE文件的相关技术背景的介绍。不久，你将会在我下个月发布的研究中看到相关技术的联系。

## 背景

对PE文件（如exe、dll、sys等文件）进行签名是什么意思呢？一个比较简明的答案就是选择相应的PE文件，右键单击之后，查看文件属性，如果存在“数字签名”选项卡，则表示该文件已经被签名。当你看到文件属性中的这个“数字签名”选项卡的时候，实际上意味着该PE文件是`Authenticode签名`的，在这个文件中有一个二进制的数据包，其中包括一个证书和这个文件的签名哈希值（更具体点说，`Authenticode哈希值`就是在散列计算中去除PE头的那部分数据的哈希值）。存储`Authenticode签名`的格式在`PE代码签名规范`【见附件1】中有详细说明。

![text](https://bbs.kanxue.com/upload/attach/202002/740325_AY4448MM7PPGEMR.jpg "图 1")

然而，很多本应该被签名的文件（以`notepad.exe`为例），事实上却并没有我们所说的`“数字签名”`选项卡。那这是否意味着该文件没有签名，并且微软也运行了这些未签名的代码呢？这个要视情况而定。虽然`notepad.exe`本身没有嵌入`Authenticode签名`，但实际上它通过目录签名这种方式进行过签名。Windows包含由许多目录文件组成的目录存储库，基本上只是`Authenticode`的散列表。然后，每个目录文件都被签名，以证明任何具有匹配哈希的文件源自目录文件的签名者（基本上都是`Microsoft`的程序）。因此，当`Explorer UI`不尝试查找目录签名时，几乎其他的所有签名验证工具都将执行目录查找。在`PowerShell`和`Sysinternals Sigcheck`中可以获取`Authenticode签名`。

> 注：目录文件存储在`%windir%\System32\CatRoot{F750E6C3-38EE-11D1-85E5-00C04FC295EE}`

![text](https://bbs.kanxue.com/upload/attach/202002/740325_EXCTBCFKFSRU822.jpg "图 2")

在上述屏幕截图中，`SignatureType`属性指示`notepad.exe`是目录签名（`Catalog`）。还值得注意的是`IsOSBinary`属性。虽然具体的实现没有文档化，但如果签名链接到几个已知的，散列的`Microsoft`根证书之一，则将显示`True`。有兴趣了解更多有关如何工作的人可以逆向`CertVerifyCertificateChainPolicy()`函数【[cdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3q4S2x3K6M7%4x3e0j5K6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5](https://bbs.kanxue.com/elink@585K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3q4S2x3K6M7%4x3e0j5K6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5)】。

`Sigcheck`工具使用`-i`参数，执行目录证书验证，并显示包含匹配的`Authenticode哈希`的目录文件路径。`-h`参数还将计算并显示PE文件的`SHA1`和`SHA256`形式的`Authenticode散列`（分别为`PESHA1`和`PE256`）。

现在你已经了解了数字签名的二进制格式和存储位置的基本概念，接下来可以开始将现有签名应用于未签名的代码。

## 嵌入式签名认证的应用

将嵌入的`Authenticode签名`从签名文件应用到未签名的PE文件是非常简单的。虽然这个过程可以自动化完成，这里我将用十六进制编辑器和`CFF Explorer`来讲解如何手动实现。

**步骤1：** 识别要窃取的`Authenticode签名`。在实例中，我将使用`kernel32.dll`中的一个签名作为演示。

**步骤2：** 识别`security directory`中的`WIN_CERTIFICATE`结构的偏移量和大小。

![图 9](upload/attach/202002/740325_NU7Y7FQERDBG5T3.jpg "图 9")

可以看到上述截图中的文件偏移量为`0x000A9600`，大小为`0x00003A68`。

**步骤3：** 在一款十六进制编辑器中打开`kernel32.dll`，从偏移量`0xA9600`处选择`0x3A68`字节的数据，然后复制这些数据。

![图 10](upload/attach/202002/740325_95ZC7GWSTNV2S9N.jpg "图 10")

**步骤4：** 在十六进制编辑器中打开你的未签名PE（在此示例中为`HelloWorld.exe`），滚动到最后，粘贴从`kernel32.dll`复制的`0x3A68`字节的数据。注意签名开头的文件偏移量（在我的机器环境下为`0x00000E00`）。粘贴完数据之后记得保存文件。

![图 11](upload/attach/202002/740325_34QKJRWPA6UCJ5G.jpg "图 11")

**步骤5：** 在`CFF Explorer`中打开`HelloWorld.exe`，并更新安全目录以指向应用的数字签名：`offset - 0x00000E00`，`size - 0x00003A68`。进行修改后保存文件。忽略“无效”警告，这是因为`CFF Explorer`不将安全目录视为文件偏移量，并在尝试引用数据所在的位置时发出警告。

![图 12](upload/attach/202002/740325_UDK95BCDAZFKQYQ.jpg "图 12")

这就完成了所有操作。现在，签名验证实用程序将正确解析和显示签名。唯一需要注意的是，它们将报告签名无效，因为所计算的文件的`Authenticode`与证书中存储的签名哈希的`Authenticode`不匹配。

![图 13](upload/attach/202002/740325_K9QVSE2U4PJYYSG.jpg "图 13")

现在，如果你想知道为什么`SignerCertificate`指纹符号不匹配，那么你是一个精明的读者。考虑到我们应用了相同的签名，为什么证书指纹不符合？这是因为`Get-AuthenticodeSignature`首先尝试对`kernel32.dll`进行目录文件查找。在这种情况下，它找到了`kernel32.dll`的目录条目，并显示目录文件的签名者的签名信息。`kernel32.dll`也是`Authenticode`签名的。所以要验证`Authenticode`散列的指纹值是否相同，请临时停止`CryptSvc`服务，该服务负责执行目录哈希查询。如此，你将看到指纹值匹配。这表示目录哈希是使用不同的代码签名证书用于签署`kernel32.dll`本身的证书。

![图 14](upload/attach/202002/740325_W933N5CZK57VY95.jpg "图 14")

## 目录签名对PE文件的应用

实际上，`CryptSvc`一直在运行，并且将会执行目录查找。假设你想要注意OPSEC，并匹配用于签署目标二进制文件的相同证书。事实证明，您可以通过在`WIN_CERTIFICATE`结构中交换`bCertificate`的内容并相应地更新`dwLength`来将目录文件的内容实际应用于嵌入式PE签名。可以跟着后续的实例一起操作。请注意，我们的目标（在本实例中）是将`Authenticode签名`应用于我们的无符号二进制文件，与用于对包含的目录文件进行签名的二进制文件相同：本实例中，证书指纹为：`AFDD80C4EBF2F61D3943F18BB566D6AA6F6E5033`。

**步骤1：** 识别包含目标二进制文件的`Authenticode`哈希的目录文件——`kernel32.dll`。如果一个文件是`Authenticode`签名的，`Sigcheck`工具将无法解析目录文件。然而，`Signtool`（包括在`Windows SDK`中）可以做到。

![图 15](upload/attach/202002/740325_33UABTA9KVZKANP.jpg "图 15")

**步骤2：** 在十六进制编辑器中打开目录文件，并获取文件大小为`0x000137C7`。

![图 16](upload/attach/202002/740325_TTBXNNF4C2T5ZXA.jpg "图 16")

**步骤3：** 我们将在十六进制编辑器中手动构造一个`WIN_CERTIFICATE`结构。我们来看看我们提供的每个成员：

+ `dwLength`：这是`WIN_CERTIFICATE`结构的总长度，即`bCertificate`字节加上其他字段的大小 = `4`（`DWORD`大小）+ `2`（`WORD`大小）+ `2`（`WORD`大小）+ `0x000137C7`（`bCertificate`——`.cat`文件的文件大小）= `0x000137CF`。

+ `wRevision`：这将是`0x0200`来表示`WIN_CERT_REVISION_2_0`。

+ `wCertificateType`：这将是`0x0002`来表示`WIN_CERT_TYPE_PKCS_SIGNED_DATA`。

+ `bCertificate`：由目录文件的原始字节组成。

在十六进制编辑器中制作数据时，请注意数据以小端格式存储。

![图 17](upload/attach/202002/740325_SXQHUEZKNK6P7TW.jpg "图 17")

**步骤4：** 复制`WIN_CERTIFICATE`结构的所有字节，附加到未签名的PE的末尾，并相应地更新安全目录的偏移量和大小。

![](upload/attach/201710/740325_igpnwlt8mdivzai.bmp)

现在，假设你的计算和对齐是正确的，看看与目录文件的指纹匹配情况！

![](upload/attach/201710/740325_o4a6es3hqh79wh2.bmp)

![](upload/attach/201710/740325_nl2nvq0t1zsjhsk.bmp)

这篇博文中提出的技术，希望可以引起一些人考虑如何检测数字签名的滥用。虽然我没有彻底研究清楚签名的启发，但我只是提出一系列问题，以期待能够激励他人开始调查和撰写检测潜在的签名异常：

- 对于合法签名的Microsoft PE文件中PE时间戳和证书有效期之间是否存在任何相关性？攻击者提供的代码的PE时间戳偏离上述相关性吗？

- 阅读本文后，你对一个具有哈希不匹配的“签名”文件的信任级别是什么级别呢？

- 你将如何检测具有由目录文件组成的嵌入式`Authenticode`签名的PE文件？提示：前面提到的特定OID可能是有用的。

- 如何在不同的系统上验证目录签名文件的签名？

- 停止/禁用的`CryptSvc`服务对执行本地签名验证的安全产品有什么影响？如果发生这种情况，那么所有意图和目的的大多数系统文件将不再被签名。

- 我看到的每一个合法的PE填充在一个`0x10`字节的边界。我在示例中将目录内容应用于`Authenticode`签名的示例不是`0x10`字节对齐。

- 如何区分合法的`Microsoft`数字签名和所有证书属性应用于自签名证书的签名？

- 如果在数字签名之后附加数据，该怎么办？这在过去一直被滥用。

- 建议英特尔的专家应该在使用不同的证书调查相同的代码时，找到`Authenticode`哈希成为一个有趣的实验方向。`VirusTotal`将其提供为`Authentihash`值：即使用`sigcheck -h`计算的哈希值。如果我正在调查在`VirusTotal`中的一个`Authentihash`上有多次命中的样本的变体，我会发现这非常有趣。
