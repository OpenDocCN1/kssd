
```
__int64 __fastcall JNI_OnLoad(__int64 a1)

{

  __int64 result; // rax

  __int64 v2; // rdi

  __int64 v3; // [rsp+10h] [rbp-10h] BYREF

  int v4; // [rsp+18h] [rbp-8h]

  int v5; // [rsp+1Ch] [rbp-4h]



  v3 = 0LL;

  v5 = -1;

  v4 = 0;

  memset(cmd_id_list, 0, sizeof(cmd_id_list));

  memset(jni_func_list, 0, sizeof(jni_func_list));

  if ( (*(*a1 + 060LL))(a1, &v3, 65540LL) )

    return 0xFFFFFFFFLL;

  v2 = v3;

  if ( !registerNatives(v3) )

    return 0xFFFFFFFFLL;

  v4 = _s201603301100(v2);

  if ( v4 )

    result = 0xFFFFFFFFLL;

  else

    result = 65540LL;

  return result;

}
```

```
_QWORD *__fastcall register_command_processor(int a1, __int64 a2)

{

  _QWORD *result; // rax

  int i; // [rsp+1Ch] [rbp-4h]



  for ( i = 0; i <= 511; ++i )

  {

    result = jni_func_list[i];

    if ( !result )

    {

      cmd_id_list[i] = a1;

      result = jni_func_list;

      jni_func_list[i] = a2;

      return result;

    }

  }

  return result;

}
```

注册jni函数

```
__int64 registerNatives()

{

  register_command_processor(7854, randomSM2KeyPairs);

  register_command_processor(7853, sm2Encrypt);

  register_command_processor(7852, sm2Decrypt);

  register_command_processor(7851, sm2DoSign);

  register_command_processor(7850, sm2VerifySign);

  register_command_processor(7849, sm3HashWithData);

  register_command_processor(7848, sm3HashWithFilePath);

  register_command_processor(7847, randomSM4IV);

  register_command_processor(7846, randomSM4Key);

  register_command_processor(7845, sm4EncryptData);

  register_command_processor(7844, sm4DecryptData);

  register_command_processor(7843, sm4EncryptFile);

  register_command_processor(7842, sm4DecryptFile);

  register_command_processor(7841, randomAESGCM128Key);

  register_command_processor(7840, aesGcm128EncryptData);

  register_command_processor(7871, aesGcm128DecryptData);

  register_command_processor(7870, randomRSAKeyPairs);

  register_command_processor(7869, rsaEncrypt);

  register_command_processor(7868, rsaDecrypt);

  register_command_processor(7867, rsaDoSign);

  register_command_processor(7866, rsaVerifySign);

  register_command_processor(7865, md5Data);

  register_command_processor(7864, sha1Data);

  register_command_processor(7863, sha224Data);

  register_command_processor(7862, sha256Data);

  register_command_processor(7861, hmacMD5Data);

  register_command_processor(7860, hmacSHA1Data);

  register_command_processor(7859, hmacSHA256Data);

  register_command_processor(7858, aes256Encrypt);

  register_command_processor(7857, aes256Decrypt);

  register_command_processor(7856, sha512Data);

  register_command_processor(7823, hmacSHA512Data);

  register_command_processor(7822, hmacSM3Data);

  register_command_processor(7839, collaboSM2ClientGenKeypairs);

  register_command_processor(7838, collaboSM2ServerGenKeypairs);

  register_command_processor(7837, collaboSM2ClientFristDoSign);

  register_command_processor(7836, collaboSM2ServerDoSign);

  register_command_processor(7835, collaboSM2ClientSecondDoSign);

  register_command_processor(7834, SSEcollaboSM2ServerProcessGenKeypairs);

  register_command_processor(7833, SSEcollaboSM2ServerProcessDoSign);

  register_command_processor(7761, xxxx);

  register_command_processor(7760, xxxx);

  register_command_processor(7359, xxxx);

  register_command_processor(7358, xxxx);

  register_command_processor(7342, xxxx);

  register_command_processor(7341, xxxx);

  register_command_processor(7340, xxxx);

  register_command_processor(7339, xxxx);

  return 1LL;

}
```

通过注册函数和对应类解密函数：

![](https://bbs.kanxue.com/upload/attach/202205/748627_7MQNXGJ6GF3GCMG.jpg)

jni函数表

![](https://bbs.kanxue.com/upload/attach/202205/748627_J9JZKY8VRJTCQJZ.jpg)

之后解密后进数据库（使用了`#{}`来防SQL注入）：

![](https://bbs.kanxue.com/upload/attach/202205/748627_M3H7FXED8CJ5Y6V.jpg)

根据逻辑返回相对应的`status`给C端，以上就是第一步身份验证的流程。

身份校验通过后进入第二流程，得到控制器IP和端口开始登陆。

请求数据包如下：

![](https://bbs.kanxue.com/upload/attach/202205/748627_8ANBQ6W4NFGMP5B.jpg)

调试器附加进程：内容依旧带上了上面的用户名密码验证码信息外加了一个`json` `ext`字段包含了两个内容：`codetoken`、`pubkey`。

看看S端的处理逻辑，这里简单说一下流程，不贴代码了。

S端底层获取到数据后发送到相应的注册服务，服务函数里逻辑为：解密数据包、检测用户是否存在、检测`codetoken`是否为空、检查手机号、检查`pubkey`是否为空（`key`客户端生成，服务端不会去校验它的真实性），一顿的逻辑检查通过后【`uuid`、`userid`、时间、设备类型、`pubkey`、`clientype`】入库，最后获取隧道的相关配置信息返还给C端。

隧道配置内容：

```
{"allowedips": , "dns": "", "portaluri": "", "riskuri": "" }
```

生成一个配置文件后创建隧道服务：

```
00000000007E6A27  | 48 89 54 24 50                 | mov qword ptr ss:[rsp+50],rdx           |

00000000007E6A2C  | 48 89 04 24                    | mov qword ptr ss:[rsp],rax              | [rsp]:"WireGuardTunnel$controller"

00000000007E6A30  | 48 89 4C 24 08                 | mov qword ptr ss:[rsp+8],rcx            |

00000000007E6A35  | E8 66 C8 00 00                 | call client.7F32A0                   |
```

最后就是每次连接就走指定的某隧道就行了，基本的流程摸透了，最后写一下发现的安全问题。

# 0x3. 漏洞

刚开始基本测了一周左右，发现了几个安全问题，比较有价值的就是一个拒绝服务还有加密协议的加密密钥来重新实现加密传输就可以暴力猜账号密码，这两天也挖了服务端一个远程命令注入。

首先产品说明文档里带了一个产品部署的架构图，这里脱下敏，自己照猫画虎画了一个差不多的图。

![](https://bbs.kanxue.com/upload/attach/202205/748627_FQXJ95DFAPGMNBR.jpg)

从完全黑盒测试的角度就只有一个客户端，对于服务端的所有信息除了IP地址外，没有任何其他有用的信息。

## 1. 认证机制绕过

C到S传输协议UDP，并经过某种加密进行传输，过程中没有协商之类的，看一眼抓包数据：

![](https://bbs.kanxue.com/upload/attach/202205/748627_GTEMKAQCE6RQCS7.jpg)

可以实现一个S端不管发送什么数据只给他一个成功的状态码就能欺骗到登录成功的界面，这里需要满足两个条件：

1、需要一个成功的状态码数据包或`information`。

2、没有二次校验的机制。

写了一个S端进行了测试，成功绕过了第一层校验到达登陆页面的时候才发现有二次校验的机制，之后二次校验的包来回分析发现了在第一步的校验包中会有一个设备ID的hash码要存入S的数据库中，在二次校验中会对设备HASH码查询来校验第一步的认证，此路是行不通。

POC:
