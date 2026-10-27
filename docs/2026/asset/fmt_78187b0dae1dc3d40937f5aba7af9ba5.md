
安全目录表的二进制数据是`WIN_CERTIFICATE`结构【[4aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3c8F1y4e0R3J5x3o6f1&6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5](https://bbs.kanxue.com/elink@a46K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3c8F1y4e0R3J5x3o6f1&6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5) 】的。下面是`kernel32.dll`在`010Editor`中解析的结果【[9c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3q4@1N6r3W2X3k6i4y4@1j5i4c8A6L8$3&6Q4x3V1k6V1x3e0m8X3z5e0p5^5y4e0W2T1k6o6m8V1k6X3k6V1y4r3p5#2x3K6V1&6y4o6g2S2k6e0l9J5k6h3y4U0j5R3`.`.，](https://bbs.kanxue.com/elink@bd6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3q4@1N6r3W2X3k6i4y4@1j5i4c8A6L8$3&6Q4x3V1k6V1x3e0m8X3z5e0p5^5y4e0W2T1k6o6m8V1k6X3k6V1y4r3p5#2x3K6V1&6y4o6g2S2k6e0l9J5k6h3y4U0j5W2!0q4c8W2!0n7b7#2)9^5b7H3`.`.) 相应文件见附件2 】（文件偏移量是`0x000A9600`）：

![图 7](https://bbs.kanxue.com/upload/attach/202002/740325_6W9TD4QXE27G4GW.jpg "图 7")

PE文件的`Authenticode签名`应始终具有`WIN_CERT_TYPE_PKCS_SIGNED_DATA`的`wRevision`。后续的字节数组是与目录文件内容相同的`PKCS＃7`以及`ASN.1`编码签名数据。唯一的区别是你应该找不到`1.3.6.1.4.1.311.12.1.1`这样的OID，因为这表明存在的目录散列签名。

在网上在线的`ASN.1`解码器解析出的原始`bCertificate`数据证实我们正在处理正确的`PKCS＃7`的数据：

![图 8](https://bbs.kanxue.com/upload/attach/202002/740325_TN4KN3DFMAMN2ZH.jpg "图 8")

## 数字签名对未签名PE文件的应用

现在你已经了解了数字签名的二进制格式和存储位置的基本概念，接下来可以开始将现有签名应用于未签名的代码。

### 嵌入式签名认证的应用

将嵌入的`Authenticode签名`从签名文件应用到未签名的PE文件是非常简单的。虽然这个过程可以自动化完成，这里我将用十六进制编辑器和`CFF Explorer`来讲解如何手动实现。

**步骤1:** 识别要窃取的`Authenticode签名`。在实例中，我将使用`kernel32.dll`中的一个签名作为演示。

**步骤2:** 识别`security directory`中的`WIN_CERTIFICATE`结构的偏移量和大小。

![图 9](https://bbs.kanxue.com/upload/attach/202002/740325_NU7Y7FQERDBG5T3.jpg "图 9")

可以看到上述截图中的文件偏移量为`0x000A9600`，大小为`0x00003A68`。

**步骤3:** 在一款十六进制编辑器中打开`kernel32.dll`，从偏移量`0xA9600`处选择`0x3A68`字节的数据，然后复制这些数据。

![图 10](https://bbs.kanxue.com/upload/attach/202002/740325_95ZC7GWSTNV2S9N.jpg "图 10")

**步骤4:** 在十六进制编辑器中打开你的未签名PE（在此示例中为`HelloWorld.exe`），滚动到最后，粘贴从`kernel32.dll`复制的`3A68`字节的数据。 注意签名开头的文件偏移量（在我的机器环境下为`0x00000E00`）。粘贴完数据之后记得保存文件。

![图 11](https://bbs.kanxue.com/upload/attach/202002/740325_34QKJRWPA6UCJ5G.jpg "图 11")

**步骤5:** 在`CFF Explorer`中打开`HelloWorld.exe`，并更新安全目录以指向应用的数字签名：`offset - 0x00000E00`，`size - 0x00003A68`。 进行修改后保存文件。 忽略“无效”警告，这是因为`CFF Explorer`不将安全目录视为文件偏移量，并在尝试引用数据所在的位置时发出警告。

![图 12](https://bbs.kanxue.com/upload/attach/202002/740325_UDK95BCDAZFKQYQ.jpg "图 12")

这就完成了所有操作。现在，签名验证实用程序将正确解析和显示签名。 唯一需要注意的是，它们将报告签名无效，因为所计算的文件的Authenticode与证书中存储的签名哈希的Authenticode不匹配。

![图 13](https://bbs.kanxue.com/upload/attach/202002/740325_K9QVSE2U4PJYYSG.jpg "图 13")

现在，如果你想知道为什么`SignerCertificate`指纹符号不匹配，那么你是一个精明的读者。考虑到我们应用了相同的签名，为什么证书指纹不符合？这是因为`Get-AuthenticodeSignature`首先尝试对`kernel32.dll`进行目录文件查找。在这种情况下，它找到了`kernel32.dll`的目录条目，并显示目录文件的签名者的签名信息。`kernel32.dll`也是Authenticode签名的。 所以要验证Authenticode散列的指纹值是否相同，请临时停止`CryptSvc`服务，该服务负责执行目录哈希查询的服务。如此，你将看到指纹值匹配。 这表示目录哈希是使用不同的代码签名证书用于签署`kernel32.dll`本身的证书。

![图 14](https://bbs.kanxue.com/upload/attach/202002/740325_W933N5CZK57VY95.jpg "图 14")

### 目录签名对PE文件的应用

实际上，`CryptSvc`一直在运行，并且将会执行目录查找。假设你想要注意OPSEC，并匹配用于签署目标二进制文件的相同证书。事实证明，您可以通过在`WIN_CERTIFICATE`结构中交换`bCertificate`的内容并相应地更新`dwLength`来将目录文件的内容实际应用于嵌入式PE签名。 可以跟着后续的实例一起操作。请注意，我们的目标（在本实例中）是将`Authenticode签名`应用于我们的无符号二进制文件，与用于对包含的目录文件进行签名的二进制文件相同：本实例中，证书指纹为： `AFDD80C4EBF2F61D3943F18BB566D6AA6F6E5033` 。

**步骤1:** 识别包含目标二进制文件的`Authenticode`哈希的目录文件——`kernel32.dll`。 如果一个文件是`Authenticode`签名的，`Sigcheck`工具将无法解析目录文件。然而，`Signtool`（包括在`Windows SDK`中）可以做到。

![图 15](https://bbs.kanxue.com/upload/attach/202002/740325_33UABTA9KVZKANP.jpg "图 15")

**步骤2:** 在十六进制编辑器中打开目录文件，并获取文件大小为`0x000137C7`

![图 16](https://bbs.kanxue.com/upload/attach/202002/740325_TTBXNNF4C2T5ZXA.jpg "图 16")

**步骤3:** 我们将在十六进制编辑器中手动构造一个`WIN_CERTIFICATE`结构。我们来看看我们提供的每个成员:

- `dwLength`：这是`WIN_CERTIFICATE`结构的总长度，即`bCertificate`字节加上其他字段的大小= `4（DWORD大小）+ 2（WORD大小）+ 2（WORD大小）+ 0x000137C7`（`bCertificate` —— `.cat文件的文件大小`）= `0x000137CF`。

- `wRevision`：这将是`0x0200`来表示`WIN_CERT_REVISION_2_0`。

- `wCertificateType`：这将是`0x0002`来表示`WIN_CERT_TYPE_PKCS_SIGNED_DATA`。

- `bCertificate`:由目录文件的原始字节组成。

在十六进制编辑器中制作数据时，请注意数据以小端格式存储。

![图 17](https://bbs.kanxue.com/upload/attach/202002/740325_SXQHUEZKNK6P7TW.jpg "图 17")

步骤4：复制所有`WIN_CERTIFICATE`的所有字节，附加到未签名的PE的末尾，并相应地更新安全目录的偏移量和大小。

![](https://bbs.kanxue.com/upload/attach/201710/740325_igpnwlt8mdivzai.bmp)

现在，假设你的计算和对齐是正确的，看看与目录文件的指纹匹配情况！

![](https://bbs.kanxue.com/upload/attach/201710/740325_o4a6es3hqh79wh2.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_nl2nvq0t1zsjhsk.bmp)

这篇博文中提出的技术，希望可以引起一些人考虑如何检测数字签名的滥用。虽然我没有彻底研究清楚签名的启发，但我只是提出一系列问题，以期待能够激励他人开始调查和撰写检测潜在的签名异常：

- 对于合法签名的Microsoft PE文件中PE时间戳和证书有效期之间是否存在任何相关性？攻击者提供的代码的PE时间戳偏离上述相关性吗？

- 阅读本文后，你对一个一个具有哈希不匹配的“签名”文件的信任级别是什么级别呢？

- 你将如何检测具有由目录文件组成的嵌入式`Authenticode`签名的PE文件？提示：前面提到的特定OID可能是有用的。

- 如何在不同的系统上验证目录签名文件的签名？

- 停止/禁用的`CryptSvc`服务对执行本地签名验证的安全产品有什么影响？如果发生这种情况，那么所有意图和目的的大多数系统文件将不再被签名。

- 我看到的每一个合法的PE填充在一个`0x10`字节的边界。我在示例中将目录内容应用于`Authenticode`签名的示例不是`0x10`字节对齐。

- 如何区分合法的`Microsoft`数字签名和所有证书属性应用于自签名证书的签名？

- 如果在数字签名之后附加数据，该怎么办？这在过去一直被滥用。

- 建议英特尔的专家应该在使用不同的证书调查相同的代码时，找到`Authenticode`哈希成为一个有趣的实验方向。`VirusTotal`将其提供为`Authentihash`值：即使用`sigcheck -h`计算的哈希值。如果我正在调查在`VirusTotal`中的一个`Authentihash`上有多次命中的样本的变体，我会发现这非常有趣。

![](https://bbs.kanxue.com/upload/attach/201710/740325_4272azd3q5nr1s7.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_85magj4trszc0oh.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_amtx50r565bysy3.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_80qi4cgll0nluyz.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_6fhh7gaqz6v9kt4.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_0r4l3rh6an1080t.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_a63q8ia60cr6se1.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_oc9xbfe7zh6u1ql.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_hucohsgn1eb7msn.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_9lf3uk7nvriq1a0.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_ccaw5xpvyyypjir.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_cy1wavu0f6jrvoe.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_6kom1ywxjj30ely.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_oa9v4v0q5c23o2w.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_7froqy4asnkick1.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_wk62e6q8i9sclsg.bmp)

![](https://bbs.kanxue.com/upload/attach/201710/740325_yp5fcoohwxtuv5p.bmp)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.bmp](javascript:void(0);)

  （136.54kb，8次下载）

- [2.bmp](javascript:void(0);)

  （211.93kb，7次下载）

- [3.bmp](javascript:void(0);)

  （207.70kb，6次下载）

- [4.bmp](javascript:void(0);)

  （145.90kb，6次下载）

- [6.bmp](javascript:void(0);)

  （348.91kb，6次下载）

- [5.bmp](javascript:void(0);)

  （244.93kb，6次下载）

- [7.bmp](javascript:void(0);)

  （101.35kb，6次下载）

- [8.bmp](javascript:void(0);)

  （211.82kb，6次下载）

- [9.bmp](javascript:void(0);)

  （147.26kb，6次下载）

- [10.bmp](javascript:void(0);)

  （72.45kb，6次下载）

- [11.bmp](javascript:void(0);)

  （61.41kb，6次下载）

- [12.bmp](javascript:void(0);)

  （168.80kb，7次下载）

- [13.bmp](javascript:void(0);)

  （134.24kb，6次下载）

- [14.bmp](javascript:void(0);)

  （150.84kb，7次下载）

- [15.bmp](javascript:void(0);)

  （253.97kb，7次下载）

- [16.bmp](javascript:void(0);)

  （604.74kb，6次下载）

- [17.bmp](javascript:void(0);)

  （1.22MB，6次下载）

- [附件1-Authenticode\_PE.docx](javascript:void(0);)

  （116.22kb，28次下载）

- [附件2-BouncyCastle.Crypto.dll](javascript:void(0);)

  （2.13MB，14次下载）
