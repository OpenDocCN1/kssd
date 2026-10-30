
Netlogon session初始化由client完成，client和sever会交换随机的8字节随机数（称为`challenges`），然后他们都通过使用密钥派生函数将两个`challenges`与client的账户密码hash混合在一起来计算`session key`。然后，client使用计算得到的`session key`计算得到一个`client credential`。

server重新计算相同的credential，如果与client计算的结果相同，则判定client拥有正确的用户密码以及`session key`，验证通过。其过程大致如下图所示：

![](https://bbs.kanxue.com/upload/attach/202009/779730_7D2EN24C235FN5T.png)

在认证握手阶段，client和server都可以协商是否对subsequent messages进行加密和密码学认证(称为`seal and sign`)，其主要目的是抵御网络层面的攻击者。如果不使用加密，一些执行重要action的Netlogon call仍然会包含一个`authticator`值，该值也是使用`session key`计算获得。

### 3. 详细分析

客户端和服务器用于生成凭据值的加密原语由`ComputeNetlogonCredential()`函数实现，该函数接收一个8字节长度的输入，然后使用`session key`对其进行转换，然后输出8字节长度的数据。这里保证安全性的基本原则是，不知道`session key`的攻击者将无法计算或猜测与某个输入匹配的正确输出，那么也就无法获取可以成功实现认证的credential。

#### 1. 2种credential的算法

在计算credential时可使用的算法有2种：`AES-CFB8`和`2DES`。

##### 2DES

在使用`2DES`进行计算时，计算`session key`的逻辑如下：

```c
InitLMKey(KeyIn, KeyOut)
KeyOut[0] = KeyIn[0] >> 0x01;
KeyOut[1] = ((KeyIn[0]&0x01)<<6) | (KeyIn[1]>>2);
KeyOut[2] = ((KeyIn[1]&0x03)<<5) | (KeyIn[2]>>3);
KeyOut[3] = ((KeyIn[2]&0x07)<<4) | (KeyIn[3]>>4);
KeyOut[4] = ((KeyIn[3]&0x0F)<<3) | (KeyIn[4]>>5);
KeyOut[5] = ((KeyIn[4]&0x1F)<<2) | (KeyIn[5]>>6);
KeyOut[6] = ((KeyIn[5]&0x3F)<<1) | (KeyIn[6]>>7);
KeyOut[7] = KeyIn[6] & 0x7F;
for (int i=0; i<8; i++){
    KeyOut[i] = (KeyOut[i] << 1) & 0xfe;
}
```

假设`bytes(s,e,l)`为从字节数组`l`返回，从`s`到`e`。`session key`计算完成后，credential按照如下逻辑进行计算：

```text
ComputeNetlogonCredential(Input, Sk, Output)
    SET k1 to bytes(0, 6, Sk)
    CALL InitLMKey(k1, k3)
    SET k2 to bytes(7, 13, Sk)
    CALL InitLMKey(k2, k4)
    CALL DES_ECB(Input, k3, &output1)
    CALL DES_ECB(output1, k4, &output2)
    SET Output to output2
```

##### AES-CFB8

如果client和server之间协商了AES支持，则使用AES-128加密算法使用0初始化矢量在8位CFB模式下计算Netlogon凭据。其计算过程大致如下：

```text
ComputeNetlogonCredential(Input, Sk, Output)
    SET IV = 0
    CALL AesEncrypt(Input, Sk, IV, Output)
```

`AesEncrypt`是8位CFB模式下的AES-128加密算法，初始化矢量为0。

具体使用哪种算法，是在进行认证时由client进行相关`flags`的设置。但是，由于现代版本的Windows Server的默认设置都会拒绝使用`2DES`加密方案，所以大多数的domains都使用AES加密方案。而本次漏洞就产生于这种新发展的加密方案，`2DES`的旧版加密方案反而没有该漏洞的存在。

#### 2. 存在漏洞的AES-CFB8

基本的AES块密码操作需要16个字节的输入并将其置换为大小相等的输出，为了加密更大或更小的input，需要选择一种`mode of operation`。在`ComputerNetlogonCredential()`函数种，仅需要转换8字节数据，选择了`CFB8`模式。因为该模式比一般的AES的任何一种模式都要慢大约16倍，所以利用范围很有限，大部分情况下都不会选择使用该模式。

`AES-CFB8`算法的计算过程大致如下：

- 首先，在明文plaintext前面加上16字节长度的`Initialistation Vector`
- 对修改后的`IV+plaintext`进行AES运算，获取其结果的第一个字节
- 使用获取的第1和字节和plaintext的下一个字节进行异或操作

上述过程示意图如下：

![](https://bbs.kanxue.com/upload/attach/202009/779730_NQTG9SCXB9WDCBT.png)

从上面的过程可以看出，为了完成对message的各个字节数据的加密，需要指定一个`IV`来引导整个过程。`IV`必须具备唯一性，必须为随机产生，这样对于同一个plaintext才可能产生不同的加密结果。但是，在`ComputeNetlogonCredential()`函数中，该值被设置为了**固定长度、值全为0的16字节数据**。但是，使用全0的`IV`，就一定会出问题吗？概率很低，但不代表没有概率：在256个密钥中总是可以找到1个，对全0的plaintext应用`AESCFB8`加密将导致全0的ciphertext，其过程如下：

![](https://bbs.kanxue.com/upload/attach/202009/779730_ZAF92ZDNPNCWX4U.png)

也就是说，如果`IV`和plaintext均为0，那么有256次机会可以找到一个`key`，使得最后的计算结果也全为0。实际上，此属性比较通用：当`IV`仅由0组成时，将存在一个0≤X≤255的整数，该整数表示以n字节开头的X值的明文最终被转换成n个值为0的字节开头的密文。X取决于加密密钥，并且是随机分布的。

到此为止，漏洞内容已经明确：一个全0的输入会导致一个全0的输出。

### 4. 利用思路

#### 1. 欺骗client credential

在通过`NetrServerReqChallenge()`函数完成`challenges`交换后，client会调用`NetrServerAuthenticate3()`函数来进行自身验证，函数原型如下：

```c
NTSTATUS NetrServerAuthenticate3(
    [in, unique, string] LOGONSRV_HANDLE PrimaryName,
    [in, string] wchar_t* AccountName,
    [in] NETLOGON_SECURE_CHANNEL_TYPE SecureChannelType,
    [in, string] wchar_t* ComputerName,
    [in] PNETLOGON_CREDENTIAL ClientCredential,
    [out] PNETLOGON_CREDENTIAL ServerCredential,
    [in, out] ULONG* NegotiateFlags,
    [out] ULONG* AccountRid
);
```

该函数的`ClientCrendential`参数为一个指向`NETLOGON_CREDENTIAL`结构的指针，该结构中包含client的credential：

```c
typedef struct _NETLOGON_CREDENTIAL {
    CHAR data[8];
} NETLOGON_CREDENTIAL,
 *PNETLOGON_CREDENTIAL;
```

credential的计算由`ComputeNetlogonCredential()`完成，该函数利用`NetrServerReqChallenge()`函数传来的`challenge`进行计算，而`challenge`是完全可以由我们控制的，因此可以设置为8个0。这也就意味着，对于一个正确有效的`ClientCredential`来说，可以在256次机会中计算得到一个全0的`ClientCredential`。

因为server在每次认证请求中也会产生一个`server challenge`，该值也是`session key`计算的一个参数，也就是说，对于每次认证请求，`session key`都是不同的。但是因为计算机并没有做防爆破的机制，登录认证失败后可以一直尝试登录，所以可以进行256次爆破，直到找到正确的`key`。

#### 2. 绕过signing 和 sealing

虽然通过第一步可以实现身份验证绕过，但是对于Netlogon的传输加密机制（`RPC signing and sealing`）来说，我们不知道`session key`到底是多少。但是，singing和sealing是可选项，可以通过在`NetrServerAuthenticate3()`函数中取消设置对应的标志位来关闭这2个选项。在现代的client中，默认是拒绝接受未设置这2个`flag`的server的连接请求的，但是server并不拒绝client的连接请求。所以，我们可以在进行攻击时手动关闭这2个`flag`。

#### 3. 欺骗调用

即使禁用了加密，每个执行某些敏感操作的函数调用也必须包含一个`Authenticator`值。该值的计算通过使用`ComputeNetlogonCredential(带有会话密钥)`计算`ClientStoredCredential + Timestamp`获得：

```text
SET TimeNow = current time;
SET ClientAuthenticator.Timestamp = TimeNow;
SET ClientStoredCredential = ClientStoredCredential + TimeNow;
CALL ComputeNetlogonCredential(ClientStoredCredential, Session-Key, ClientAuthenticator.Credential);
```

`ClientStoredCredential`是client维护的一个增量值。执行握手时，它会初始化为与我们提供的`ClientCredential`相同的值。因为client credential全0，因此对于身份验证后执行的第一个调用，`ClientStoredCredential`的值为0。

`TimeStamp`为`NETLOGON_AUTHENTICATOR`结构中的`TimeStamp`字段，包含当前的Posix时间。但是事实上，server并未对该值设置多少限制，因此可以简单地设置为1970年1月1日，即值设为0。

经过第一步后，我们知道`ComputeNetlogonCredential(0)=0`。因此，可以通过简单地提供全零`authenticator`和全零`TimeStamp`来验证第一个调用。

#### 4. 修改计算机的AD域密码

经过前3步后，我们已经可以向任何计算机发起Netlogon的请求。接下来，一种思路是调用`NetrServerPasswordGet()`函数来获取计算机密码的NTLM哈希。但是，哈希利用了另外一种机制使用`session key`进行加密，所以该函数无法利用。退而使用`NetrServerPasswordSet2()`函数，其主要作用为client设置新的计算机密码，函数原型如下：

```c
NTSTATUS NetrServerPasswordSet2(
    [in, unique, string] LOGONSRV_HANDLE PrimaryName,
    [in, string] wchar_t* AccountName,
    [in] NETLOGON_SECURE_CHANNEL_TYPE SecureChannelType,
    [in, string] wchar_t* ComputerName,
    [in] PNETLOGON_AUTHENTICATOR Authenticator,
    [out] PNETLOGON_AUTHENTICATOR ReturnAuthenticator,
    [in] PNL_TRUST_PASSWORD ClearNewPassword
);
```

这种密码没有hash，但是使用`session key`进行了加密。可以再次使用`CFB8`和全0的`IV`进行解决。

plaintext密码结构包含516个字节，最后的4个字节指明了密码长度（字节为单位）。结构中所有不属于密码功能的字节都被视为填充，并且可以具有任意值。如果我们提供516个0，那么加密结果就是516个0。实际上，计算机并不禁止设置空密码，所以这里使用516个0可以登录域上的任意一台计算机。完成后，可以代表此计算机建立新的Netlogon连接。这次计算机的密码是空的，因此可以正常使用该协议。如果需要，现在还可以设置任何其他非空密码。其过程大致如下：

![](https://bbs.kanxue.com/upload/attach/202009/779730_HKW9BDHJCU4VSSK.png)

以这种方式更改计算机密码时，只能在AD中更改。目标系统本身仍将在本地存储其原始密码。然后，该计算机将无法再向域进行身份验证，并且只能通过手动操作来重新同步。因此，可以将任何设备锁定在域之外。同样，只要计算机帐户在域中具有特殊特权，现在就可以滥用这些特权。

#### 5. 从改密到域控

**后续可能跟渗透关系更大，个人对渗透了解不多，可能理解有误。**

到此为止，可以更改的计算机的密码是DC本身的密码，AD中存储的DC密码与存储在本地注册表（`HKLM\SECURITY\Policy\Secrets\$machine.ACC`）中的并不相同。这可能会造成DC发生一些不可预知的行为。但是依旧可以登录。只需要运行Impacket的`secretsdump`脚本就可以让DC的密码生效，该脚本会利用复制DRS协议通过域的从域中提取所有用户HASH，包括域管理员HASH（包括`krbtgt`，可以用来制作金票），然后用来登录DC更新电脑密码注册表，就可以令攻击者成为域管理员。

### 5. PoC分析

针对该漏洞的poc目前有2种，1种是检测脚本，1种是完整的漏洞利用脚本。

检测脚本的代码分析如下：

|  |  |
| --- | --- |
