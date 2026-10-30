
`kd> dt AuthorizationData`
`Local var @ 0x332f9d0 Type PKERB_AUTHORIZATION_DATA_s`
`+0x000 next : (null)`
`+0x004 value : PKERB_AUTHORIZATION_DATA_Seq`
`kd> dx -id 0,0,89c47a68 -r1 (\*((KDCSVC!PKERB_AUTHORIZATION_DATA_Seq \*)0x332f9d4))`
`(\*((KDCSVC!PKERB_AUTHORIZATION_DATA_Seq \*)0x332f9d4)) [Type: PKERB_AUTHORIZATION_DATA_Seq]`
`[+0x000] auth_data_type : 128 [Type: long]`
`[+0x004] auth_data [Type: tagASN1octetstring_t]`
`kd> dx -id 0,0,89c47a68 -r1 (\*((KDCSVC!tagASN1octetstring_t \*)0x332f9d8))`
`(\*((KDCSVC!tagASN1octetstring_t \*)0x332f9d8)) [Type: tagASN1octetstring_t]`
`[+0x000] length : 0x260 [Type: unsigned long]`
`[+0x004] value : 0x16c828 : 0x4 [Type: unsigned char *]`
`kd> db 0x16c828 l 260`

```
0016c828 04 00 00 00 00 00 00 00-01 00 00 00 c0 01 00 00 ................
0016c838 48 00 00 00 00 00 00 00-0a 00 00 00 22 00 00 00 H..........."...
0016c848 08 02 00 00 00 00 00 00-06 00 00 00 14 00 00 00 ................
0016c858 30 02 00 00 00 00 00 00-07 00 00 00 14 00 00 00 0...............
0016c868 48 02 00 00 00 00 00 00-01 10 08 00 cc cc cc cc H...............
0016c878 b0 01 00 00 00 00 00 00-00 00 02 00 c2 dd c3 d9 ................
0016c888 0f f7 d7 01 ff ff ff ff-ff ff ff 7f ff ff ff ff ................
0016c898 ff ff ff 7f 56 b9 d8 d7-0f f7 d7 01 56 79 42 02 ....V.......VyB.
0016c8a8 d9 f7 d7 01 ff ff ff ff-ff ff ff 7f 18 00 18 00 ................
0016c8b8 04 00 02 00 00 00 00 00-08 00 02 00 00 00 00 00 ................
0016c8c8 0c 00 02 00 00 00 00 00-10 00 02 00 00 00 00 00 ................
0016c8d8 14 00 02 00 00 00 00 00-18 00 02 00 01 00 00 00 ................
0016c8e8 72 04 00 00 03 02 00 00-01 00 00 00 1c 00 02 00 r...............
0016c8f8 20 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ...............
0016c908 00 00 00 00 18 00 1a 00-20 00 02 00 16 00 18 00 ........ .......
0016c918 24 00 02 00 28 00 02 00-00 00 00 00 00 00 00 00 $...(...........
0016c928 80 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c938 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c948 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c958 00 00 00 00 0c 00 00 00-00 00 00 00 0c 00 00 00 ................
0016c968 57 00 49 00 4e 00 53 00-52 00 56 00 53 00 45 00 W.I.N.S.R.V.S.E.
0016c978 52 00 56 00 45 00 52 00-00 00 00 00 00 00 00 00 R.V.E.R.........
0016c988 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c998 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c9a8 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0016c9b8 00 00 00 00 01 00 00 00-03 02 00 00 07 00 00 00 ................
0016c9c8 0d 00 00 00 00 00 00 00-0c 00 00 00 57 00 49 00 ............W.I.
0016c9d8 4e 00 53 00 52 00 56 00-53 00 45 00 52 00 56 00 N.S.R.V.S.E.R.V.
0016c9e8 45 00 52 00 0c 00 00 00-00 00 00 00 0b 00 00 00 E.R.............
0016c9f8 52 00 45 00 4e 00 50 00-45 00 4e 00 47 00 59 00 R.E.N.P.E.N.G.Y.
0016ca08 55 00 30 00 33 00 00 00-04 00 00 00 01 04 00 00 U.0.3...........
0016ca18 00 00 00 05 15 00 00 00-db ac e2 f8 a5 b2 f3 d1 ................
0016ca28 a1 c4 3e 10 00 00 00 00-00 b4 a3 e5 0f f7 d7 01 ..>.............
0016ca38 18 00 57 00 49 00 4e 00-53 00 52 00 56 00 53 00 ..W.I.N.S.R.V.S.
0016ca48 45 00 52 00 56 00 45 00-52 00 00 00 00 00 00 00 E.R.V.E.R.......
0016ca58 76 ff ff ff c1 fc e6 ad-46 30 3f 05 5e ed 74 c0 v.......F0?.^.t.
0016ca68 20 7d c9 54 00 00 00 00-76 ff ff ff 42 e1 22 e3 }.T....v...B.".
0016ca78 3b 44 cd ee b7 d7 50 5f-2d f9 44 ab 00 00 00 00 ;D....P_-.D.....
```

之后便是将`PAC`放入`TGT`中，将其打包并使用`krbtgt`密钥进行加密，通过`AS_REP`消息传递回`Client`

### 关键代码

关键代码如下，不再展开分析

```cpp
KerbErr = BuildReply(
&ClientTicketInfo,
(Nonce != 0) ? Nonce : RequestBody->nonce,
&Ticket.server_name,
Ticket.realm,
((RequestBody->bit_mask & addresses_present) != 0) ? RequestBody->addresses : NULL,
&Ticket,
&ReplyBody
);
...
KerbErr = KerbPackTicket(
&Ticket,
ServerKey,
ServiceTicketInfo.PasswordVersion,
&Reply.ticket
);
...
KerbErr = KerbPackKdcReplyBody(
&ReplyBody,
(EncryptionKey.keyvalue.value != NULL) ? &EncryptionKey : ClientKey,
(EncryptionKey.keyvalue.value != NULL) ? KERB_NO_KEY_VERSION : ClientTicketInfo.PasswordVersion,
KERB_TGS_REP_SALT,
KERB_ENCRYPTED_AS_REPLY_PDU,
&Reply.encrypted_part
);
```

## 还原机器账户名

还原机器账户名的目的是使得域控处理`TGS_REQ`请求的时候，找不到账户从而是用自己的`Key`加密

```c#
//undo samaccountname change
SetMachineAccountAttribute(argContainer, argDistinguishedName, argDomain, argDomainController, "samaccountname", argMachineAccount, argMachineAccount, false, false, argVerbose, credential);
```

## 申请ST

`Client`向域控申请`WINSRVSERVER`的服务票据，域控在`HandleTGSRequest`函数中处理`TGS_REQ`请求。

首先通过`KerbFindPreAuthDataEntry`获取`TGS_REQ`中包含的`ApRequest`

```cpp
ApRequest = KerbFindPreAuthDataEntry(
KRB5_PADATA_TGS_REQ,
RequestMessage->KERB_KDC_REQUEST_preauth_data
);
```

之后便是解析获得的`APRequest`获得解密后的`TGT`

```cpp
//验证请求。这包括对AP请求进行解码，找到合适的密钥来解密票据，并检查票据。
KerbErr = KdcVerifyKdcRequest(
ApRequest->preauth_data.value,
ApRequest->preauth_data.length,
ClientAddress,
TRUE, // this is a kdc request
&UnmarshalledApRequest,
&UnmarshalledAuthenticator,
&SourceEncryptPart,
&ReplyKey,
&SourceTicketKey,
&ServerTicketInfo,
&UseSubKey,
pExtendedError
);
```

`KdcVerifyKdcRequest`做了以下几件事情

- `KdcVerifyKdcRequest`
- 解包`ApRequest` ---- `KerbUnpackApRequest`
- 根据其中的服务名（`kbrtgt`）获取服务的相关信息 ---- `KdcNormalize`
- 通过相关信息找到服务的`Hash` ---- `KerbGetKeyFromList`
- 解密`TGT` --- `KerbCheckTicket`
- 获得解密后的`TGT` --- `KerbVerifyTicket`
- 用`TGT`中的`Key`（`key`为`Client`与`KDC`通信所需要的`LogonSessionKey`）解密获得`Authenticator` --- `KerbUnpackAuthenticator`
- ……（校验检查之类的）

查看这个函数的结果，获得了传过来的明文`TGT`和`krbtgt`的相关服务信息
