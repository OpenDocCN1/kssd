
# 4. 服务层——Services layer

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_KF4MBKD5XDB9QTK.png)

## 研究方法

通过获取到了某个使用CodeSys V3 PLC固件，使用IDA进行分析，分析出协议的处理流程和功能码含义。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_TM3DSCZWY5DE9FY.png)

CodeSys V3通过`ServerRegisterServiceHandler`注册对应`service_group`处理的handler（相当于传统意义上上的功能码）：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_TSM4B5BTUNYFRTZ.png)

所有注册的服务如下图所示（图示为本人编写的LUA解析插件的内容），可以看到CodeSys协议包含相当多的功能，包括用户管理，日志管理，文件传输等功能。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_JGZA5KVSE7EXCEP.png)

## CodeSys V3

为了帮助理解和研究CodeSys协议，本人不得不实现一个Wireshark解析插件来解析数据报文，下面简要介绍每个layer来分析V3数据包格式。

### 块驱动层

块驱动层的组件的主要任务是创建通过物理或软件接口进行通信的能力。任何块驱动程序组件都是一个“输入点”，用于接收数据包和传输数据包的点。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_5P2QG92NXKVCNY2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_8JYHU3MCVXDM5JM.png)

### 数据报层

该层的主要目的是路由数据包，检测 CODESYS 网络中的节点，并将数据传输到下一层。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_85E55V363T8XT9D.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_ZXT5YM2EUJVF7F6.png)

简单介绍上面关键字段的含义：

`Magic`字段：

通过 CODESYS协议生成的数据包的魔数。该字段的大小为一个字节。

`Hop count`字段：

表示在网络上接收到的数据包所经过的CodeSys节点数。每当一个 CODESYS 网络节点收到一个数据包并将其重定向到另一个 CODESYS 网络节点时，它就会递减`hop_count`的值。如果一个节点收到了一个数据包但不是它的最终接收者并且`hop_count`字段的值等于 0，则节点将丢弃此数据包。本质上，该字段让 CODESYS 节点的网络免于无休止地转发数据包。

`packet_info`字段：

表示数据包相关设置。

`priority`字段：

指定处理数据包的优先级。以下数值用于指定优先级：0 – 低，1 – 正常，2 – 高，3 – 紧急

`length_data_block`字段：

表示接收者可以接受的最大数据大小。

`Service id`字段：

服务ID，指示特定服务器必须处理接收到的数据。

### 通道层

通道是 CODESYS 网络节点之间的一种通信机制，它保证通信的同步、传输数据完整性的验证、消息传递的通知以及大量数据的传输。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_TR8R46TR367A5Y3.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_M29F7EQY6SJ45WB.png)

`Packet_type`字段：

`BLK`（`0x1`）包类型，表示数据传输。

`flags`字段：

`BLK`数据包类型的数据包标志。

`channel id`字段：

用于数据传输的通道 ID。

`Blk_id`字段：

当前 `BLK` 消息的 ID。此 ID 每次由通过信道发起通信开始的一方递增。

`Ack_id`字段：

最后一条 `ACK` 消息的 ID。该 ID 每次都由响应方更改。在应用层接收到最后一个数据传输到服务的数据包后，响应端将该ID进行更改。

`Remaining_data_size`字段：

`remaining_data` 所包含的预期数据的大小。

`Checksum`字段：

该字段是对`remaining_data`中包含的数据进行校验，`CRC32`算法用于计算校验和。

### 服务层

该层的主要任务是查询请求的服务并传输其操作设置。服务层的任务包括对在该层传输的数据进行编码、解码、加密和解密。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_7WX8CWG9W39YEFF.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_QRJN85D4W7H87H4.png)

`protocol_id`字段：

使用的协议的ID。此ID指示哪个协议处理程序修改了数据以及应使用哪个协议将数据传输到服务。

`Header size`字段:

`protocol_header`的大小。该字段的值不包含先前字段和当前字段的大小。

`Service group`字段:

被查询服务的ID。如果在ID中设置了最高有效位，则意味着该消息是来自服务的响应。

`Service ID`字段：

命令的ID，这个ID决定了服务执行的功能。

`session_id`字段：

会话的ID，包含接收到的会话或空会话的值。

`data size`字段：

`protocol_data`字段中数据的大小。

# 授权流程

CodeSys 在新版V3通讯协议中已经默认开启授权机制，用户必须设置账号密码才可以进一步使用。在设置密码成功后，用户必须登录后才能对PLC进行控制管理以及编程组态等操作。但是具体的算法之前从未有人披露过，下面我将简单介绍其流程和相关算法。

## 密码哈希存储

新版CodeSys V3通讯协议账号相关信息默认存放于`UserDatabase`文件中，旧版本的CodeSys V3的密码是`MD5`加密的，新版做了改进，使用了`scrypt`算法来进行存放。该算法在`CryptoDeriveKey`中实现，如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_HYMK6UBM8VC33KV.png)

`UserDatabase`文件内容如下，其中`admin`为用户名，后面接着的是密码哈希信息。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_AK9PHRK745NP75D.png)

- `d819a24df7bdc985` 指的是 `scrypt` 的盐salt。
- `02f6……5cfd` 指的是 `scrypt` 算出的密码哈希。
- `512，8，1` 指的是 `scrypt` 的 `N,r,p` 参数。

使用`scrypt`原因主要为该算法不仅计算所需时间长，而且占用的内存也多，使得并行计算多个摘要异常困难，因此利用彩虹表进行暴力攻击更加困难。

当然即使算法设计的再健全，也挡不住用户使用弱口令。在最新版的组态软件并没有强制要求密码强度，使得用户也可以设置诸如`123456`等弱口令密码。这就导致安全意识不强的用户会将密码设置成弱口令。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_QY9DYZPBHBUU23P.png)

通过编写`py`脚本将`scrypt`哈希信息转换为标准的`hashcat`的`scrypt hash`格式，之后可以利用`hashcat`对该哈希进行爆破攻击，当然前提是获取到哈希文件（可结合一个任意文件下载漏洞）

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_XHY7DW8K6H5MRR8.png)

生成得到的哈希如下所示：

|   |   |
|---|---|
| 1 | `SCRYPT:512:8:1:2BmiTfe9yYU=:Avat90uq9ScoWI3p+46HI8solorDp225o6Qi68wud34=` |

运行`hashcat.cat`命令：`hashcat.exe -m 8900 -a 0 scrypt.hash dict.txt`，爆破成功率依赖于字典以及密码的复杂度，可以使用高性能的GPU，加快爆破速度。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_JUXGXPD6QX587AC.png)

## 授权协议流程

客户端先通过`CmpDevice`的2号服务ID请求PLC获取公钥和`challenge`：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_EPTPBJEXHYT62T8.png)

PLC将会随机生成一个密钥对和一个32字节的`challenge`，响应包将包含公钥和`challenge`。

公钥信息如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_V8R92HV3EPBNSF9.png)

32 字节`challenge`信息如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_WE74AHHFEF5D38X.png)

客户端在接收以上信息后，使用`CryptoAsymmetricEncrypt`函数加密密码，该函数如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_XQ2MDNQJF2NTM76.png)

使用`python`描述`password_info`生成算法：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_EVPDW6AJYGHEQXC.png)

- `RSA`加密使用的是`PKCS1_OAEP`，其中的哈希算法为`SHA256`。
- `XOR`算法是简单的逐位异位算法。

最后，客户端再次通过`CmpDevice`的2号服务ID下发到PLC中：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_HFYSDA9JGPF2VCV.png)

如果密码正确的话将会返回`app key`，每次进行操作都需要携带该授权`key`。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_55G2QRUT9XNV7BC.png)

## 风险分析

从流程可以知道，如果没有加密的情况下，攻击者可以通过中间人的方法替换公钥并解密出原始密码。为了保证编程软件和PLC的通信安全，建议启用安全通信（基于`TLS`）。

# About RCE

密码保护是保护CodeSys PLC的重要防线，一旦攻击者突破这层防线，就可以利用注入代码的方法插入任意shellcode执行RCE。这是由于CodeSys为了实现可编程功能，会将用户编写的代码统一编译成机器码，并且由RunTime调度执行，执行的过程没有在VM或者sandbox中，这就导致攻击者可以插入任意的机器码（例如LINUX平台上的反弹shell）来完全接管PLC。

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_6V9WV5CF9UVHA6H.png)

如下所示，使用该方法在PLC开启了Vxworks上的Telnet服务（原先是关闭的）：

![图片描述](https://bbs.kanxue.com/upload/attach/202302/651413_ANT2PRUJDGMNVBC.png)

# 总结

本文详细介绍了CodeSys V3的协议格式和授权流程，并分析了密码保护机制的安全性问题，然而设计是美好的，但是工业现场实践过程中往往存在很多问题。

1. 由于很多厂商购买的CodeSys 需要每年花费高额的费用来获取更新支持，绝大多数厂商在购买源码后不在进行更新，这导致目前很多PLC使用的CodeSys版本仍然停留在非常老的版本，仍然受N-day的影响。
2. 电气工程师安全意识缺乏，很多部署在现场的PLC根本没设置密码。
3. 在厂家都已经停止维护的情况下，CodeSys V2 版本仍然被大量使用，该版本的RCE漏洞更多（N-day以及0-day）。

## 参考链接

```
fdfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6j5%4y4Q4x3X3c8U0k6i4u0@1i4K6u0W2K9$3q4K6M7r3g2J5M7$3E0&6i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2H3L8%4u0@1M7#2)9J5c8U0t1H3x3e0W2Q4x3V1j5H3z5g2)9J5c8U0p5^5i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8J5k6i4y4W2j5i4u0U0K9q4)9J5k6r3y4G2k6r3g2K6P5i4y4Q4x3X3c8J5N6h3&6@1K9h3#2W2i4K6u0V1j5g2)9J5k6s2m8D9j5#2)9J5k6r3y4G2L8Y4c8J5L8$3I4Q4x3X3c8X3M7X3q4E0k6i4N6G2M7X3E0Q4x3X3c8H3j5i4u0@1i4K6u0V1x3W2)9J5c8R3`.`.
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#工控设备](https://bbs.kanxue.com/forum-128-1-174.htm)
