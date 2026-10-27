
2. 如果文件不匹配任何以前的文件类型，它将调用<a href="elink@d1aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3q4S2x3K6M7&6z5e0t1%4i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">`CryptEnumOIDFunction`</a>.aspx)函数，传递它的功能名称为“`CryptSIPDllIsMyFileType`”和“`CryptSIPDllIsMyFileType2`”。 这些功能分别对应于以下注册表项的查找：

- `HKLM\\SOFTWARE\\[WOW6432Node]\\Microsoft\\Cryptography\\OID\\EncodingType 0\\CryptSIPDllIsMyFileType\\<All sub-GUIDs>`

- `HKLM\\SOFTWARE\\[WOW6432Node]\\Microsoft\\Cryptography\\OID\\EncodingType 0\\CryptSIPDllIsMyFileType2\\<All sub-GUIDs>`

随着`CryptEnumOIDFunction`枚举每个SIP GUID注册表子项，它将从“`FuncName`”和“`Dll`”列出的注册表键值中调用DLL导出函数。

“`CryptSIPDllIsMyFileType`”的功能原型文档<a href="elink@c0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3y4U0y4e0b7J5y4U0x3$3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">戳我</a>.aspx)，“`CryptSIPDllIsMyFileType2`”的功能原型文档<a href="elink@4abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3y4U0y4e0b7J5y4U0b7H3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">戳我</a>.aspx)。 如果有实现，`CryptSIPDllIsMyFileType`函数首先被调用，如果其中一个函数返回“TRUE”，则返回处理签名的SIP GUID。 在实践中（至少在Windows 10上），没有SIP实现`CryptSIPDllIsMyFileType`，所以随后调用`CryptSIPDllIsMyFileType2`函数来尝试解决处理SIP。 例如，PowerShell（SIP GUID：603BCC1F-4B59-4E08-B724-D2C6297EF351）将`CryptSIPDllIsMyFileType2`实现为`pwrshsip！PsIsMyFileType`。 经过反汇编、反编译及整理输出，这里是`PsIsMyFileType`函数的C语言版的原型：

```c
#define CRYPT\_SUBJTYPE\_POWERSHELL\_IMAGE \
{ 0x603BCC1F, \
0x4B59, \
0x4E08, \
{ 0xB7, 0x24, 0xD2, 0xC6, 0x29, 0x7E, 0xF3, 0x51 } \
}
BOOL WINAPI PsIsMyFileType(IN WCHAR \*pwszFileName, OUT GUID \*pgSubject) {
BOOL bResult;
WCHAR \*SupportedExtensions[7];
WCHAR \*Extension;
GUID PowerShellSIPGUID = CRYPT\_SUBJTYPE\_POWERSHELL\_IMAGE;
SupportedExtensions[0] = L"ps1";
SupportedExtensions[1] = L"ps1xml";
SupportedExtensions[2] = L"psc1";
SupportedExtensions[3] = L"psd1";
SupportedExtensions[4] = L"psm1";
SupportedExtensions[5] = L"cdxml";
SupportedExtensions[6] = L"mof";
bResult = FALSE;
if (pwszFileName && pgSubject) {
Extension = wcsrchr(pwszFileName, '.');
if (Extension) {
Extension++;
for (int i = 0; i < 7; i++) {
if (!\_wcsicmp(Extension, SupportedExtensions[i])) {
bResult = TRUE;
memcpy(pgSubject, &PowerShellSIPGUID, sizeof(GUID));
break;
}
}
}
}
else
{
SetLastError(
ERROR\_INVALID\_PARAMETER
);
}
return
bResult;
}
```

从C代码中可以看出，如果任何文件有任何上述扩展名，那么PowerShell SIP将是用作代码签名的SIP。 `CryptSIPDllIsMyFileType2`不一定要检查文件扩展名，SIP还可以选择打开文件句柄并检查文件中的魔数值，以确定正确的文件/blob SIP处理顺序。

其他支持的SIP文件类型处理函数如下（非详尽列表）：

1. 000C10F1-0000-0000-C000-000000000046

`C:\Windows\System32\MSISIP.DLL`

`MsiSIPIsMyTypeOfFile`

2. 06C9E010-38CE-11D4-A2A3-00104BD35090

`C:\Windows\System32\wshext.dll`

`IsFileSupportedName`

3. 0AC5DF4B-CE07-4DE2-B76E-23C839A09FD1

`C:\Windows\System32\AppxSip.dll`

`AppxSipIsFileSupportedName`

4. 0F5F58B3-AADE-4B9A-A434-95742D92ECEB

`C:\Windows\System32\AppxSip.dll`

`AppxBundleSipIsFileSupportedName`

5. 1629F04E-2799-4DB5-8FE5-ACE10F17EBAB

`C:\Windows\System32\wshext.dll`

`IsFileSupportedName`

6. 1A610570-38CE-11D4-A2A3-00104BD35090

`C:\Windows\System32\wshext.dll`

`IsFileSupportedName`

7. 5598CFF1-68DB-4340-B57F-1CACF88C9A51

`C:\Windows\System32\AppxSip.dll`

`P7xSipIsFileSupportedName`

8. 603BCC1F-4B59-4E08-B724-D2C6297EF351

`C:\Windows\System32\WindowsPowerShell\v1.0\pwrshsip.dll`

`PsIsMyFileType`

9. 9F3053C5-439D-4BF7-8A77-04F0450A1D9F

`C:\Windows\System32\EsdSip.dll`

`EsdSipIsMyFileType`

10. CF78C6DE-64A2-4799-B506-89ADFF5D16D6

`C:\Windows\System32\AppxSip.dll`

`EappxSipIsFileSupportedName`

11. D1D04F0C-9ABA-430D-B0E4-D7E96ACCE66C

`C:\Windows\System32\AppxSip.dll`

`EappxBundleSipIsFileSupportedName`

对于读者来说, 逆向上面的某些函数来查看Windows所支持代码签名的文件或二进制blob的类型，这将是一个很有价值的练习。

一旦需要检索签名的软件获得该 SIP 的 GUID, 那它就可以继续提取该证书。

## 文件签名检索和哈希验证

一旦负责处理特定文件/二进制Blob格式的签名的SIP通过其各自的GUID标识符被识别，`WinVerifyTrust`就会知道如何从该文件中获取数字签名并验证其计算出的哈希对嵌入在数字中的签名哈希签名。为实现这一点, `WinVerifyTrust`在注册表中调用以下函数:

SIP 签名检索功能位置：

- `HKLM\\SOFTWARE\\[WOW6432Node]\\Microsoft\\Cryptography\\OID\\EncodingType 0\\CryptSIPDllGetSignedDataMsg\\{SIP Guid}`

- `Dll`

- `FuncName`

SIP 哈希验证函数：

- `HKLM\\SOFTWARE\\[WOW6432Node]\\Microsoft\\Cryptography\\OID\\EncodingType 0\\CryptSIPDllVerifyIndirectData\\{SIP Guid}`

- `Dll`

- `FuncName`

<a href="elink@c0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3y4U0y4e0b7J5y4e0R3#2i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">`CryptSIPDllGetSignedDataMsg`</a>.aspx)和<a href="elink@c2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3y4U0y4e0b7J5y4e0V1I4i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">`CryptSIPDllVerifyIndirectData`</a>.aspx)的函数原型在MSDN有文档，同样也存在于Windows SDK中的mssip.h头文件中。

SIP签名检索功能原型：

```c
BOOL WINAPI CryptSIPGetSignedDataMsg(
IN SIP\_SUBJECTINFO \*pSubjectInfo,
OUT DWORD \*pdwEncodingType,
IN DWORD dwIndex,
IN OUT DWORD \*pcbSignedDataMsg,
OUT BYTE \*pbSignedDataMsg);
```

SIP 哈希验证函数原型：

```c
BOOL WINAPI CryptSIPVerifyIndirectData(
IN SIP\_SUBJECTINFO \*pSubjectInfo,
IN SIP\_INDIRECT\_DATA \*pIndirectData);
```

<a href="elink@dc4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3u0T1y4K6x3$3y4o6x3@1i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">`SIP\_SUBJECTINFO`</a>.aspx) <a href="elink@eb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3k6r3g2K6K9%4c8G2M7q4)9J5c8X3u0T1y4K6x3$3y4o6x3K6i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6f1`.">`SIP\_INDIRECT\_DATA`</a>.aspx)

提供给这些函数的参数由调用信任提供者负责填充（有关信任提供者架构的更多细节，请参见以下部分）。当`CryptSIPGetSignedDataMsg`被调用时，SIP将提取编码的数字签名（最常用的是`CERT\_SIGNED\_CONTENT\_INFO`结构体，ASN.1 `PKCS\_7\_ASN\_ENCODING`和`X509\_ASN\_ENCODING`编码），并通过“`pbSignedDataMsg`”参数返回。`CERT\_SIGNED\_CONTENT\_INFO`内容由签名证书（包括其发行链）、用于对文件进行哈希和签名的算法以及文件的签名散列组成。调用信任提供者然后对数字签名进行解码，提取散列算法和签名哈希值，然后将它们传递给`CryptSIPVerifyIndirectData`。在校验认证码哈希计算并与已签名哈希进行比较后，如果匹配，则`CryptSIPVerifyIndirectData`返回TRUE，否则返回FALSE，然后`WinVerifyTrust`将返回一个错误，表明哈希不匹配。

`CryptSIPVerifyIndirectData`是最重要的数字签名验证功能之一，但这将会犯错：因为攻击者可以将现有的合法数字签名应用于其恶意软件——这是一种<a href="elink@101K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5%4u0S2K9i4g2Q4x3V1k6K6N6r3q4@1N6i4y4Q4x3V1j5^5y4K6V1$3z5e0l9%4z5e0f1&6y4o6j5^5x3U0M7%4y4K6k6Q4x3@1k6D9j5h3&6Y4i4K6y4p5k6h3^5`.">在野</a>的攻击技术。

以下是一个适用于合法认证码签名的恶意软件示例的哈希失真的示例：

![在使用微软认证码签名的未签名文件上显示哈希不匹配错误的示例（注意相同的SignerCertificate指纹值）](./upload/attach/201710/570137\_jugfubuwmedttmi.png)

\*在使用微软认证码签名的未签名文件上显示哈希不匹配错误的示例（注意相同的SignerCertificate指纹值）\*

![未签名的文件在应用于签名文件的认证码签名时无法验证。微软就是这么设计的。](./upload/attach/201710/570137\_ritp933a43sapl4.png)

\*未签名的文件在应用于签名文件的认证码签名时无法验证。微软就是这么设计的。\*

## 信任提供者架构

到目前为止，已经讨论了SIP的基本架构。如现在应该理解的，SIP仅负责数字签名应用、检索和散列计算、验证。应用于文件的数字签名的存在是无意义的，除非某些标准被实际验证。这就是信任提供者发挥作用的地方——除了内置到所需的信任提供者中的标准之外，还可以根据调用者指定的参数组合对`WinVerifyTrust`进行验证。

像SIP一样，信任提供者也由GUID唯一标识。 截至到Windows 10，有以下信任提供者存在：

| GUID | 描述 |
