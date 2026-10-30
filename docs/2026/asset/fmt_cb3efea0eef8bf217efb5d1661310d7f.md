
之后会获取请求的相关信息

- 在`REQUEST_BODY`中获得`ServerName`
- 在`TGT`中获得`cname`和`crealm`

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `KerbErr` `=` `KerbConvertPrincipalNameToKdcName(`  `&ServerName,`  `&RequestBody``-``>KERB_KDC_REQUEST_BODY_server_name`  `);`    `KerbErr` `=` `KerbConvertPrincipalNameToKdcName(`  `&SourceClientName,`  `&SourceEncryptPart``-``>client_name`  `);`    `KerbErr` `=` `KerbConvertRealmToUnicodeString(`  `&SourceClientRealm,`  `&SourceEncryptPart``-``>client_realm`  `);` |

内容分别如下

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `kd> dt ServerName`  `Local var @` `0x327fdd8` `Type` `_KERB_INTERNAL_NAME``*`  `0x00117610`  `+``0x000` `NameType         :` `0n1`  `+``0x002` `NameCount        :` `1`  `+``0x004` `Names            : [``1``] _UNICODE_STRING` `"WINSRVSERVER"`    `kd> dt SourceClientName`  `Local var @` `0x327fdd4` `Type` `_KERB_INTERNAL_NAME``*`  `0x0017f3e8`  `+``0x000` `NameType         :` `0n1`  `+``0x002` `NameCount        :` `1`  `+``0x004` `Names            : [``1``] _UNICODE_STRING` `"WINSRVSERVER"`    `kd> dt SourceClientRealm`  `Local var @` `0x327fdc4` `Type` `_UNICODE_STRING`  `"WINTESTYU03.COM"`  `+``0x000` `Length           :` `0x1e`  `+``0x002` `MaximumLength    :` `0x20`  `+``0x004` `Buffer`           `:` `0x00153578`  `"WINTESTYU03.COM"` |

之后会调用`KdcFindS4UClientAndRealm`来获取`PA_DATA_FOR_USER`这个结构中的内容

`KdcFindS4UClientAndRealm`函数会解析`PaList`并将其转换成`KERB_PA_FOR_USER`结构，目前需要注意的便是其中的`userName`是我们要请求的高权限用户的用户名`Administrator`

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `kd> dt S4URequest`  `Local var @` `0x327f9b0` `Type` `KERB_PA_FOR_USER``*`  `0x0012aaa8`  `+``0x000` `bit_mask         :` `0`  `+``0x000` `o                : [``1``]  ""`  `+``0x004` `userName         : KERB_PRINCIPAL_NAME`  `+``0x00c` `userRealm        :` `0x0012abf0`  `"WINTESTYU03.COM"`  `+``0x010` `cksum            : KERB_CHECKSUM`  `+``0x01c` `authentication_package :` `0x000fca30`  `"Kerberos"`  `+``0x020` `authorization_data : tagASN1octetstring_t`  `kd> dx` `-``id` `0``,``0``,``89de1678` `-``r1 (``*``((KDCSVC!KERB_PRINCIPAL_NAME` `*``)``0x12aaac``))`  `(``*``((KDCSVC!KERB_PRINCIPAL_NAME` `*``)``0x12aaac``))                 [``Type``: KERB_PRINCIPAL_NAME]`  `[``+``0x000``] name_type        :` `10` `[``Type``:` `long``]`  `[``+``0x004``] name_string      :` `0x82c98` `[``Type``: KERB_PRINCIPAL_NAME_name_string_s` `*``]`  `kd> dx` `-``id` `0``,``0``,``89de1678` `-``r1 ((KDCSVC!KERB_PRINCIPAL_NAME_name_string_s` `*``)``0x82c98``)`  `((KDCSVC!KERB_PRINCIPAL_NAME_name_string_s` `*``)``0x82c98``)                 :` `0x82c98` `[``Type``: KERB_PRINCIPAL_NAME_name_string_s` `*``]`  `[``+``0x000``]` `next`             `:` `0x0` `[``Type``: KERB_PRINCIPAL_NAME_name_string_s` `*``]`  `[``+``0x004``] value            :` `0x159c88` `:` `"wintestServer"` `[``Type``: char` `*``]` |

之后会通过`KdcNormalize`获取我们自身`WINSRVSERVER`的相关信息

其中的关键调用如下：

- `KdcNormalize`
  - `KdcGetTicketInfo`
    - `SamIGetUserLogonInformation2` (`WINSRVSERVER`)
    - `SamIGetUserLogonInformation2` (`WINSRVSERVER$`)

对于漏洞的利用便发生在这个函数中，并且利用了两次。

第一次实现了将申请的用户转换为域控上的`Administrator`

第二次实现了将申请的服务转换成`WINSRVSERVER$`

下面将详细分析漏洞点。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `KERBERR`  `KdcNormalize(`  `IN PKERB_INTERNAL_NAME PrincipalName,`  `IN OPTIONAL PUNICODE_STRING PrincipalRealm,`  `IN OPTIONAL PUNICODE_STRING RequestRealm,`  `IN OPTIONAL PUNICODE_STRING  TgtClientRealm,`  `IN ULONG NameFlags,`  `IN BOOLEAN bRestrictUserAccounts,`  `OUT PBOOLEAN Referral,`  `OUT PUNICODE_STRING RealmName,`  `OUT PKDC_TICKET_INFO TicketInfo,`  `OUT PKERB_EXT_ERROR  pExtendedError,`  `OUT OPTIONAL SAMPR_HANDLE` `*` `UserHandle,`  `IN OPTIONAL ULONG WhichFields,`  `IN OPTIONAL ULONG ExtendedFields,`  `OUT OPTIONAL PUSER_INTERNAL6_INFORMATION` `*` `UserInfo,`  `OUT OPTIONAL PSID_AND_ATTRIBUTES_LIST GroupMembership`  `)` |

调用`KdcNormalize`时的相关参数中最重要的就是`SourceCName`

因为我们是在利用`S4U2Self`协议请求自身的ST，所以`SourceCName`也就是自身的名字`WINSRVSERVER`

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `kd> dt SourceCName`  `Local var @` `0x327f9e0` `Type` `_KERB_INTERNAL_NAME``*`  `0x0016e920`  `+``0x000` `NameType         :` `0n1`  `+``0x002` `NameCount        :` `1`  `+``0x004` `Names            : [``1``] _UNICODE_STRING` `"WINSRVSERVER"` |

之后在`CheckSam`条件中会调用到`KdcGetTicketInfo`来获取用户`WINSRVSERVER`的相关信息

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `KerbErr` `=` `KdcGetTicketInfo(`  `&OutputPrincipal,`  `0``,` `/``/` `no lookup flags means sam name`  `bRestrictUserAccounts,`  `NULL,` `/``/` `no principal name`  `NULL,` `/``/` `no realm name,`  `TicketInfo,`  `pExtendedError,`  `UserHandle,`  `WhichFields,`  `ExtendedFields,`  `UserInfo,`  `GroupMembership`  `);` |

此时`OutputPrincipal`的值为`WINSRVSERVER`，即我们自己的机器名`DC`，目前仍一切正常

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `kd> dt OutputPrincipal`  `Local var @` `0x327f928` `Type` `_UNICODE_STRING`  `"WINSRVSERVER"`  `+``0x000` `Length           :` `0x18`  `+``0x002` `MaximumLength    :` `0x1a`  `+``0x004` `Buffer`           `:` `0x0016e92c`  `"WINSRVSERVER"` |

之后会调用`SamIGetUserLogonInformation2`在SAM中查找对应的账户信息，但由于此时已经将创建的机器账号还原，所以并不能找到对应的账号，该函数会返回错误

但是系统并不会直接提示找不到账号，而是会在其后面添加`'$'`符号，将其作为机器账号再次查找

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50 | `Status` `=` `SamIGetUserLogonInformation2(`  `GlobalAccountDomainHandle,`  `LookupFlags,`  `UserName,`  `WhichFields,`  `ExtendedFields,`  `&UserInfo,`  `&LocalMembership,`  `&LocalUserHandle`  `);`    `/``/`  `/``/` `WASBUG: For now,` `if` `we couldn't find the account` `try` `again`  `/``/` `with a` `'$'` `at the end (``if` `there wasn't one already)`  `/``/`    `if` `(((Status` `=``=` `STATUS_NOT_FOUND) ||`  `(Status` `=``=` `STATUS_NO_SUCH_USER)) &&`  `(!IsValidGuid) &&`  `((LookupFlags & ~SAM_NO_MEMBERSHIPS)` `=``=` `0``) &&`  `(UserName``-``>Length >``=` `sizeof(WCHAR)) &&`  `(UserName``-``>``Buffer``[UserName``-``>Length``/``sizeof(WCHAR)``-``1``] !``=` `L``'$'``))`  `{`  `Status` `=` `KerbDuplicateString(`  `&TempString,`  `UserName`  `);`  `if` `(!NT_SUCCESS(Status))`  `{`  `KerbErr` `=` `KRB_ERR_GENERIC;`  `goto Cleanup;`  `}`  `DsysAssert(TempString.MaximumLength >``=` `TempString.Length` `+` `sizeof(WCHAR));`  `TempString.``Buffer``[TempString.Length``/``sizeof(WCHAR)]` `=` `L``'$'``;`  `TempString.Length` `+``=` `sizeof(WCHAR);`    `D_DebugLog((DEB_TRACE,` `"Account not found ,trying machine account %wZ\n"``,`  `&TempString ));`    `Status` `=` `SamIGetUserLogonInformation2(`  `GlobalAccountDomainHandle,`  `LookupFlags,`  `&TempString,`  `WhichFields,`  `ExtendedFields,`  `&UserInfo,`  `&LocalMembership,`  `&LocalUserHandle`  `);`  `}` |

通过调试信息可以清晰的看到查找到的用户信息不再是`WINSRVSERVER`而是变成了`WINSRVSERVER$`也就是域控对应的机器账号`UserId = 0x3ed`

至此便完成了对于域控的欺骗，之后就是颁发ST的过程

|  |  |
| --- | --- |
