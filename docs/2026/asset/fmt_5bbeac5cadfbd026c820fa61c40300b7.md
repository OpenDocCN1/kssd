
## Malware

```
function SuperDecrypt

{

    param($script)

    $bytes = [Convert]::FromBase64String($script)

    ## XOR “encryption”

    $xorKey = 0x42

    for($counter = 0; $counter -lt $bytes.Length; $counter++)

    {

        $bytes[$counter] = $bytes[$counter] -bxor $xorKey

    }

    [System.Text.Encoding]::Unicode.GetString($bytes)

}

$decrypted = SuperDecrypt "FUIwQitCNkInQm9CCkItQjFCNkJiQmVCEkI1QixCJkJlQg=="

Invoke-Expression $decrypted
```

运行此命令将生成以下日志条目：

```
Compiling Scriptblock text (1 of 1):

        function SuperDecrypt

        {

            param($script)

            $bytes = [Convert]::FromBase64String($script)

            ## XOR "encryption"

            $xorKey = 0x42

            for($counter = 0; $counter -lt $bytes.Length; $counter++)

            {

                $bytes[$counter] = $bytes[$counter] -bxor $xorKey

            }

            [System.Text.Encoding]::Unicode.GetString($bytes)

        }

        ScriptBlock ID: ad8ae740-1f33-42aa-8dfc-1314411877e3

        Compiling Scriptblock text (1 of 1):

        $decrypted = SuperDecrypt "FUIwQitCNkInQm9CCkItQjFCNkJiQmVCEkI1QixCJkJlQg=="

        ScriptBlock ID: ba11c155-d34c-4004-88e3-6502ecb50f52

        Compiling Scriptblock text (1 of 1):

        Invoke-Expression $decrypted

        ScriptBlock ID: 856c01ca-85d7-4989-b47f-e6a09ee4eeb3

        Compiling Scriptblock text (1 of 1):

        Write-Host 'Pwnd'

        ScriptBlock ID: 5e618414-4e77-48e3-8f65-9a863f54b4c8
```

如果脚本块的长度超过了单个事件中ETW可以容纳的长度，则Windows PowerShell会将脚本分为多个部分。 这是从脚本的日志消息重新组合脚本的示例代码：

```
$created = Get-WinEvent -FilterHashtable @{
    ProviderName="Microsoft-Windows-PowerShell"; Id = 4104 } | Where-Object { <Criteria> }

$sortedScripts = $created | sort { $_.Properties[0].Value }

$mergedScript = -join ($sortedScripts | % { $_.Properties[2].Value })
```

与所有保留缓冲区有限的日志系统（即ETW日志）一样，针对此基础架构的一种攻击是使用虚假事件来充斥日志以隐藏早期证据。 为了保护自己免受这种攻击，请确保您设置了某种形式的事件日志收集。

## Cryptographic Message Syntax (CMS) encryption and decryption cmdlets（加密消息语法（CMS）加密和解密cmdlets）

PowerShell版本5和KB 3000850引入了对使用加密消息语法（CMS）格式的内容保护的支持。 这些cmdlet支持使用IETF标准格式对内容进行加密和解密，以加密保护信息，如RFC5652所述。

```
Get-CmsMessage [-Content] <string>

Get-CmsMessage [-Path] <string>

Get-CmsMessage [-LiteralPath] <string>

Protect-CmsMessage [-To] <CmsMessageRecipient[]> [-Content] <string> [[-OutFile] <string>]

Protect-CmsMessage [-To] <CmsMessageRecipient[]> [-Path] <string> [[-OutFile] <string>]

Protect-CmsMessage [-To] <CmsMessageRecipient[]> [-LiteralPath] <string> [[-OutFile] <string>]

Unprotect-CmsMessage [-EventLogRecord] <EventLogRecord> [[-To] <CmsMessageRecipient[]>] [-IncludeContext]

Unprotect-CmsMessage [-Content] <string> [[-To] <CmsMessageRecipient[]>] [-IncludeContext]

Unprotect-CmsMessage [-Path] <string> [[-To] <CmsMessageRecipient[]>] [-IncludeContext]

Unprotect-CmsMessage [-LiteralPath] <string> [[-To] <CmsMessageRecipient[]>] [-IncludeContext]
```

CMS加密标准实现了公共密钥加密，其中用于加密内容的密钥（公共密钥）和用于解密内容的密钥（私有密钥）不同。

您的公钥可以广泛共享，并且不是敏感数据。 如果使用此公共密钥加密了任何内容，则只有您的私有密钥可以解密它。 有关公钥密码术的更多信息，请参见：b9aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2F1i4K6u0W2N6$3W2C8K9i4m8W2k6r3W2S2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3W2C8K9g2)9J5c8W2m8#2j5X3I4A6j5#2)9J5k6r3E0W2P5g2)9#2k6X3y4J5P5i4m8@1L8$3N6J5j5i4m8Z5P5g2!0q4x3#2)9^5x3q4)9^5x3R3`.

要在Windows PowerShell中被识别，加密证书需要一个唯一密钥标识符（EKU）才能将它们标识为数据加密证书（例如“代码签名”，“加密邮件”的标识符）。

这是创建适用于文档加密的证书的示例：

（将“主题”中的文本更改为您的姓名，电子邮件或其他标识符），然后放入文件中（即：DocumentEncryption.inf）：

```
[Version]

Signature = "$Windows NT$"

[Strings]

szOID_ENHANCED_KEY_USAGE = "2.5.29.37"

szOID_DOCUMENT_ENCRYPTION = "1.3.6.1.4.1.311.80.1"

[NewRequest]

Subject = "cn=me@somewhere.com"

MachineKeySet = false

KeyLength = 2048

KeySpec = AT_KEYEXCHANGE

HashAlgorithm = Sha1

Exportable = true

RequestType = Cert

KeyUsage = "CERT_KEY_ENCIPHERMENT_KEY_USAGE | CERT_DATA_ENCIPHERMENT_KEY_USAGE"

ValidityPeriod = "Years"

ValidityPeriodUnits = "1000"

[Extensions]

%szOID_ENHANCED_KEY_USAGE% = "{text}%szOID_DOCUMENT_ENCRYPTION%"
```

执行：

```
certreq -new DocumentEncryption.inf DocumentEncryption.cer
```

然后你会看到加密内容：

```
106 [C:\temp]
>> $protected = "Hello World" | Protect-CmsMessage -To "*me@somewhere.com*"

107 [C:\temp]
>> $protected

-----BEGIN CMS-----
MIIBqAYJKoZIhvcNAQcDoIIBmTCCAZUCAQAxggFQMIIBTAIBADA0MCAxHjAcBgNVBAMMFWxlZWhv
bG1AbWljcm9zb2Z0LmNvbQIQQYHsbcXnjIJCtH+OhGmc1DANBgkqhkiG9w0BAQcwAASCAQAnkFHM
proJnFy4geFGfyNmxH3yeoPvwEYzdnsoVqqDPAd8D3wao77z7OhJEXwz9GeFLnxD6djKV/tF4PxR
E27aduKSLbnxfpf/sepZ4fUkuGibnwWFrxGE3B1G26MCenHWjYQiqv+Nq32Gc97qEAERrhLv6S4R
G+2dJEnesW8A+z9QPo+DwYU5FzD0Td0ExrkswVckpLNR6j17Yaags3ltNVmbdEXekhi6Psf2MLMP
TSO79lv2L0KeXFGuPOrdzPAwCkV0vNEqTEBeDnZGrjv/5766bM3GW34FXApod9u+VSFpBnqVOCBA
DVDraA6k+xwBt66cV84OHLkh0kT02SIHMDwGCSqGSIb3DQEHATAdBglghkgBZQMEASoEEJbJaiRl
KMnBoD1dkb/FzSWAEBaL8xkFwCu0e1ZtDj7nSJc=
-----END CMS-----

108 [C:\temp]
>> $protected | Unprotect-CmsMessage
Hello World
```

CMSMessageRecipient类型的任何参数都支持以下格式的标识符：

•实际证书（从证书提供者处检索）

•包含证书的文件的路径

•包含证书的目录的路径

•证书的指纹（用于在证书存储中查找）

•证书的主题名称（用于在证书存储中查找）

要在证书提供者中查看文档加密证书，可以将-DocumentEncryptionCert动态参数用于Get-ChildItem（dir）：

58 [Cert:\currentuser\my]
 >> dir -DocumentEncryptionCert

### Interoperability of CMS Content（CMS内容的互操作性）

由于CMS格式是IETF标准，因此PowerShell支持对其他符合标准的工具生成的内容进行解密，并且它生成的内容可以由其他符合标准的工具解密。

支持CMS消息格式的较流行的实现之一是OpenSSL库和命令行工具链。 与OpenSSL库交换数据时的主要挑战来自OpenSSL假设，即内容以P7M格式包含在电子邮件正文中。 幸运的是，这些基于文本的标题相对容易添加和删除。

以下PowerShell命令演示了如何使用OpenSSL和PowerShell加密和解密其他应用程序生成的内容。

```
## Install the OpenSSL package

Install-Package OpenSSL.Light

## OpenSSL requires certificates in the PEM format. To create this,
## export the Windows certificate in PFX format, and ensure that
## the PFX is protected by a password (rather than account) as
## OpenSSL doesn't support group-protected PFX files

& "C:\Program Files\OpenSSL\bin\openssl.exe" pkcs12 -in C:\temp\cert.pfx -out c:\temp\cert.pem -nodes

## 1) Encrypt with PowerShell, decrypt with OpenSSL.
## First, protect some content in PowerShell.

Get-Process | Protect-CmsMessage -To "*myRecipient*" | Set-Content encrypted.txt

<#
PowerShell uses BEGIN CMS / END CMS sigils to signify encrypted content.
OpenSSL requires an email-header:

    MIME-Version: 1.0
    Content-Disposition: attachment; filename="smime.p7m"
    Content-Type: application/pkcs7-mime; smime-type=enveloped-data; name="smime.p7m"
    Content-Transfer-Encoding: base64

So, tweak the data.
#>

$p7mHeader = @'

MIME-Version: 1.0

Content-Disposition: attachment; filename="smime.p7m"

Content-Type: application/pkcs7-mime; smime-type=enveloped-data; name="smime.p7m"

Content-Transfer-Encoding: base64

'@

$unixContent = Get-Content encrypted.txt | Select-String -notmatch "----"

$p7mHeader,"`r`n",$unixContent | Set-Content encrypted_unix.txt -Encoding ASCII

## Finally, decrypt with OpenSSL. Ensure that the content is encoded as ASCII.

& "C:\Program Files\OpenSSL\bin\openssl.exe" cms -decrypt -in encrypted_unix.txt -recip .\cert.pem

## 2) Encrypt with OpenSSL, decrypt with PowerShell
## First, protect some content with OpenSSL

$encrypted = Get-Process | & "C:\Program Files\OpenSSL\bin\openssl.exe" cms -encrypt -recip .\cert.pem

## Change the OpenSSL mail header to the standard CMS header

"-----BEGIN CMS-----",$($encrypted -notmatch ":"),"-----END CMS-----" > encrypted.cms

## Finally, decrypt with PowerShell

Unprotect-CmsMessage -Path .\encrypted.cms
```

## Secure code generation APIs（安全代码生成API）

当你编写的代码的输入能够被攻击者控制的时候，代码注入漏洞是所有漏洞中危害最大的。一个很好的例子就是你在受约束的PowerShell运行空间中为使用者提供的函数。如果攻击能利用其中一个函数进行代码注入，那么他们可以像执行代码本身那样执行恶意代码，不受运行空间的限制。

几乎每种语言都可能受到代码注入漏洞的攻击。 在SQL中，这称为“ SQL注入”。 在网站中，这称为“跨站点脚本”。 在CGI应用程序中，shell脚本或调用系统命令的工具-称为“命令注入”。

在PowerShell中，代码注入漏洞的最常见来源来自在提交给Invoke-Expression命令的字符串中包括攻击者控制的输入。 例如：
