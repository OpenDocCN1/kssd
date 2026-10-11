
# The Cryptography API, or How to Keep a Secret

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-764.htm`](https://www.pediy.com/kssd/pediy08/pediy8-764.htm)

`CRYPT API` 是微软提供给我们的一个很好的加密解密的工具。在 MSDN 中有这么一篇介绍 API 的文章，我挺感兴趣的，又结合了网上原有的一些文章，将 msdn 中的文章翻译出来，希望大家多提宝贵意见。原来的文章后面还有一个相关例子的介绍，由于各种原因，没传上去，下次有时间再传吧。

`The Cryptography API, or How to Keep a Secret`

译注：因本文发布时间较早，文中所讲述的某些内容已变化，发生变化处译者已标出。

`Robert Coleridge`

`Microsoft Developer Network Technology Group`

`August 19, 1996`

## 摘要

本文描述在新的 `Windows NT 4.0` 版本和即将在 `Windows 95` 之后版本中发布的微软加密应用程序接口 (`Microsoft Cryptography application programming interface`)。 本文检验了为安装并使用这些新的 API 需要具备的条件。 为了完成例程的编译，你需要 `Microsoft Visual C++ 4.2` 或以后版本及 `Windows NT 4.0` 或以后版本。

## 简介

加密 API 在企业计算模型（`Enterprise Computing Model`）中有着重要的应用。企业计算化程度意味着人的接触更全球化，例如国际贸易，州际库存管理等等。在这些领域中经常要通过不安全渠道传输重要信息—例如传真，通过电子邮件收发订单，以及各种各样的情况。使用 `Cryptography API`，你就能够确保信息的安全性。

本文使用 `CRYPTOAPI` 例程来演示如何加密、解密数据（`decrypt or encrypt data`），对文件签名、验证签名（`sign and verify files`），及增加、删除用户（`add and remove users`）。

## 加密 API 总览

### 加密服务提供者模块(`Cryptography Service Provider――CSP`)

\[编者注：文章中部分内容直接取材于平台 SDK（`Platform SDK`）的 `CryptoAPI` 文章。\]

当对用户的秘密数据提供保护时，加密 API 中的函数允许应用程序以一种灵活的方式来加密或者进行电子签名。所有加密操作都由独立的模块来实施，这些模块叫作加密服务提供者（`cryptographic service provider`）。操作系统中已经包含了一个 CSP，名叫“`Microsoft RSA Base Provider`”。

每个 CSP 都对加密 API 层提供一个不同的实现。一些提供强加密算法，而另一些会包含如智能卡（`smartcards`，智能卡是一张存有用户安全信息芯片的塑料卡片）之类的硬件部件。另外，一些 CSP 可能会直接与用户交互，如使用用户的签名私鈅进行数据签名时。

应用程序不需要直接使用依赖于特定 CSP 的属性。例如，“`Microsoft RSA Base Provider`”目前使用 40 位的会话密鈅（`session keys`）和 512 位的公鈅（`public keys`） (参阅 "Symmetric Versus Public-Key Encryption," `MSDN Library`, `Platform`, `SDK`, `DDK` 文档)。当程序维护这些的时候，要小心的不要假定需要使用多大的内存去存储它们。另外，当用户在系统中安装一个不同的 CSP 后，程序很可能出错。你要努力使写尽可能完善的程序。

### 密鈅库

每个 CSP 都有一个密鈅库（`key database`），里面存储着由 CSP 保存的算法密鈅。每个密鈅库都包含一个或多个密鈅容器（`key container`）， 每个容器都包含所有属于特定用户（或使用加密 API 的客户端程序）的密鈅对。每个密鈅容器都被赋与一个唯一的名字，这个名字是程序要获得此容器句柄时传给函数 `CryptAcquireContext` 的参数。图 1 图解了密鈅容器中的内容：

![](img/4716c5d63d162c1a94e489044b7dd797.jpg)

图 1 密鈅容器的内容

CSP 在两次会话之间保存所有密鈅容器，包括所有属于它的公/私鈅对（`public/private key pairs`）。但会话密鈅在两次会话之间（`session keys`）不被保存。

尽管在计算机中可以找出这些密鈅来，但这些密鈅是以一种加密的安全格式保存的。

通常，会为每个用户创建一个默认的密鈅容器。这个密鈅容器是以用户的登录名来命名的，之后任何程序都可以使用它。应用程序自己也可以创建自己的密鈅容器和密鈅对，应用可以自己起名。

### 密鈅

#### 会话密鈅

会话密鈅被用在加密和解密数据时。应用使用 `CryptGenKey` 或者 `CryptDeriveKey` 函数来创建它。CSP 将密鈅安全的保存在内部。

与密鈅对不同，会话密鈅是可变的。应用可以将密鈅保存下来以备后用或者用 `CryptExportKey` 函数以加密的密鈅二进制大对象或密鈅 `blob`（`key binary large object or key blob`）形式将密鈅导出到应用中，然后传输给其它人。

#### 公鈅、私鈅对

每个用户通常都有两对公鈅/私鈅对。一对用于加密会话密鈅，另一对用于创建数字签名。它们分别叫做密鈅交换（ `key exchange `）密鈅对与签名（`signature`）密鈅对。

注意，尽管大多数 CSP 创建的密鈅容器会包含两对密鈅对，但这不是必须的。一些 CSP 根本不存储密鈅对，而另一些会存储额外的。

### 加密

当进行数据加密时，明文消息（`plain-text message`）在被编码后会看起来象完全随机的二进制数据，以至于没有密鈅就很难将其转化为原来的消息。本文中使用如下定义：

* 消息（`Message`）：指任何数据块。消息可以是 `ASCII` 文本，一个数据库文件或者任何你要安全存储或者传输的数据。

* 明文（`Plain text`）：指没有被加密的数据。

* 密文（`Cipher text`）：指被加密过的数据。

一旦消息被加密，它就可以存储在非安全的介质上或者通过非安全的网络传输而仍然保持安全。之后，消息可以解密回原来的格式。图 2 图解了这个过程：

![](img/181907f478e4de610ce8ebad789d839f.jpg)

图 2 . 加密、解密消息

当加密消息时，要使用加密密鈅（`encryption key`）。 这就类型于用一把钥匙去锁一把锁一样。当解密这个消息时，必须用相应的解密密鈅（`decryption key`）。对解密密鈅的严格限制访问非常重要，因为任何拿到它的人可以解开用相应的加密密鈅加密的所有消息。

可能有点不可思议，但数据加密/解密就是这么简单。真正的难点是安全的保存密鈅和安全的将密鈅传输到其它人那。这部分已经超出了本文的范围，我建议读者去读文章 "Exchanging Cryptographic Keys" 的相应部分，文章在 `Win32 Cryptography API` 文档中(`MSDN Library`, `Platform`, `SDK`, `DDK` 文档)。

有两种主要的加密算法：对称算法（`symmetric algorithms`）与公鈅算法（`public-key algorithms `）\[也叫做非对称算法（`asymmetric algorithms`）\]。 使用对称算法的系统有时归入传统型（ `conventional`）中。

### 算法

对称算法是最普遍的加密算法类型。它们叫“对称”是因为使用相同的密鈅进行加密与解密。与使用公鈅算法的密鈅不同，对称密鈅是经常变化的。因为这个原因，此处将它们归于会话密鈅。与公鈅算法相比，对称算法非常快，因此最适于加密大量数据的情况。一些最常用的对称加密算法是 `RC2`， `RC4`，与数据加密标准（`Data Encryption Standard`，`DES`）。（译注：单重 `DES` 因密鈅长度已不适应当前的加密环境，最好不用。可以使用三重 `DES`，或者 `AES`）

公鈅(非对称)算法使用一对不同的密鈅：一个公鈅和一个私鈅。私鈅由密鈅对的所有者自己保存，公鈅可以自由分发给所有要求得到的人。如果用一个密鈅加密一个消息，必须用另一个密鈅解密此消息。公鈅算法很慢， 要比对称算法慢数千倍。因此它们一般仅用来加密会话密鈅。它们也用来对消息进行数字签名（`digitally sign`），下一部分会讨论这个内容。最常用的公鈅加密算法之一是 `RSA Public-Key Cipher`。

![](img/8b42bf1d36ad19dd5a278f653a1585a8.jpg)

### 文件签名

数字签名（`Digital signatures`）用在你要以明文形式分发一个消息，并且你想要让接收者能够验证这个消息自从离开你手后没有被篡改过。对消息签名并不会改变消息，它仅生成一个可附着在消息上或者单独传输的数字签名。

数字签名使用公鈅算法生成。使用私鈅来生成，并且使用相对应的公鈅来验证。图 3 图解了这一过程：

## 几个加密 API 函数

\[编者注：下面缩进部分的文件引用自 `MSDN Library`, `Platform`, `SDK`, 及 `DDK` 文档。\]

### 初始化 CSP：`CryptAcquireContext`, `CryptReleaseContext`

函数 `CryptAcquireContext` 用来获得 CSP 中一个特定密鈅容器的句柄。返回的句柄然后就可以对选择的 CSP 进行调用。

函数 `CryptReleaseContext` 用于释放函数 `CryptAcquireContext` 返回的句柄。 `CryptReleaseContext` 不会删除任何 `Cryptography API` 对象，它仅仅释放对象的句柄。

函数 `CryptAcquireContext` 执行两个操作。 首先试着查找变量中指定的 CSP， 如果找到，函数试着查找 CSP 中匹配指定密鈅容器名的密鈅容器。此函数也可以用于建立、删除密鈅容器，这取决于函数中的参数值。

取得默认 CSP 中默认密鈅容器的代码如下所示：（译注：原文章代码中注释为英文，此处译为中文，下面情况同）

```c
#include <wincrypt.h>      // 对 CryptoAPI 的定义

/*

对于非 C/C++用户，此处用到的常量如下：

#define MS_DEF_PROV       "Microsoft Base Cryptographic Provider v1.0"

#define PROV_RSA_FULL             1

*/

BOOL bResult;

HCRYPTPROC hProv;

// 试图取得转为密鈅容器的句柄

bResult = CryptAcquireContext(
            &hProv,            // 保存返回句柄的变量
            NULL,              // 默认密鈅容器
            MS_DEF_PROV,       // 默认 CSP
            PROV_RSA_FULL,     // 要取得的 CSP 类型
            0);                // 未指定动作

.

.

.

// 在此处执行操作

.

.

.

// 释放容器句柄

CryptReleaseContext(hProv);
```

如果 `CryptAcquireContext` 调用成功，返回值非零，变量 `hProv` 即为要取得的密鈅容器句柄。

要在默认 CSP 中添加或者创建一个密鈅容器，要写的代码如下：

```c
#include <wincrypt.h>      // 对 CryptoAPI 的定义

/*

对于非 C/C++用户，此处用到的常量如下：

#define MS_DEF_PROV       "Microsoft Base Cryptographic Provider v1.0"

#define PROV_RSA_FULL             1

#define CRYPT_NEWKEYSET         0x8

*/

BOOL bResult;

HCRYPTPROC hProv;

// 试图添加一个新的密鈅容器

BResult = CryptAcquireContext(
            &hProv,               // 保存返回句柄的变量
            NULL,                 // 默认密鈅容器
            MS_DEF_PROV,          // 默认 CSP
            PROV_RSA_FULL,        // 要取得的 CSP 类型
            CRYPT_NEWKEYSET);     // 创建一个新密鈅容器

.

.

.

// 在此处执行操作

.

.

.

// 释放容器句柄

CryptReleaseContext(hProv);
```

如果 `CryptAcquireContext` 调用成功，返回值非零，变量 `hProv` 即为新的密鈅容器句柄。

要从默认 CSP 中删除一个存在的密鈅容器，要写的代码如下：

```c
#include <wincrypt.h>      // 对 CryptoAPI 的定义

/*

对于非 C/C++用户，此处用到的常量如下：

#define MS_DEF_PROV       "Microsoft Base Cryptographic Provider v1.0"

#define PROV_RSA_FULL             1

#define CRYPT_DELETEKEYSET       0x10

*/

BOOL bResult;

HCRYPTPROC hProv;

// 试图删除密鈅容器

BResult = CryptAcquireContext(
            &hProv,                // 保存返回句柄的变量
            NULL,                  // 默认密鈅容器
            MS_DEF_PROV,           // 默认 CSP
            PROV_RSA_FULL,         // 要取得的 CSP 类型
            CRYPT_DELETEKEYSET);   // 删除存在的密鈅容器
```

如果 `CryptAcquireContext` 调用成功，返回值非零，变量 `hProv` 指向的密鈅容器已经删除，此密鈅容器不再有效。

### 散列数据：`CryptCreateHash`, `CryptHashData`, `CryptGetHashParam`, `CryptDestroyHash`

当我说“散列法”或“散列”（ "hashing" or "hash " ）时，是指从一块数据中派生出一个数值的方法或算法。这可能是简单的将所有数据位相加，或复杂到要对数据进行傅立叶变换。（译注：散列也被称为哈希，杂凑）

上面列出的四个函数是用于创建或者维护从提供的数据生成的散列值的，一般一起使用：

函数 `CryptCreateHash` 用于散列数据时初始化。它返回 CSP 散列对象的句柄，此句柄会在后续 `CryptHashData` 函数散列数据时使用。

下一步是调用 `CryptGetHashParam` 函数取得散列值。

函数 `CryptDestroyHash` 释放函数 `CryptCreateHash` 返回的句柄。`CryptDestroyHash` 不会删除任何加密 API 对象，它仅仅释放散列对象的句柄。

`CryptHashData` 函数用来从提供的数据中计算密码散列。为计算一个大数据块或者数据块的几个部分时，此函数可被调用多次。例如，我们要对缓冲区 `pBuffer` 中 `dwBufferLen` 字节长的数据进行散列。在此例子中我仅使用 `CALG_MD5` 散列算法来实现此目的。 加密 API SDK 文档中还提供对许多其它算法详细的描述。 本例子假定只散列一块数据。一旦调用 `CryptGetHashParam` 函数取得了散列值，此散列实例对象便不能再散列其它数据了。

```c
#include <wincrypt.h>      // 对 CryptoAPI 的定义

/*

对于非 C/C++用户，此处用到的常量如下：

#define ALG_CLASS_HASH              (4 << 13)

#define ALG_TYPE_ANY                (0)

#define ALG_SID_MD5                 3

#define CALG_MD5        (ALG_CLASS_HASH | ALG_TYPE_ANY | ALG_SID_MD5)

#define HP_HASHVAL          0x0002  // 散列值

#define HP_HASHSIZE         0x0004  // 散列值长度

*/

BOOL bResult;

HCRYPTHASH hHash;

DWORD dwBufferSize;

DWORD dwValue;

PBYTE pBuffer;

// 获得散列对象句柄

bResult = CryptCreateHash(
```



### 获取散列对象与计算散列值

下面代码展示了如何获取散列对象句柄并处理散列数据：

```c
    hProv,            // 之前获得的 CSP 句柄
    CALG_MD5,         // 散列算法
    0,                // 非密鈅散列
    0,                // 置 0
    &hHash);          // 保存散列对象句柄的变量

// 散列数据
bResult = CryptHashData(
    hHash,            // 散列对象句柄
    pBuffer,          // 数据缓冲区指针
    dwBufferlen,     // 数据长度
    0);              // 未指定值

// 得到散列值尺寸
dwBufferSize = sizeof(DWORD);

bResult = CryptGetHashParam(
    hHash,            // 散列对象句柄
    HP_HASHSIZE,      // 得到散列值尺寸
    &dwValue,         // 保存散列值长度缓冲区
    &dwBufferSize,    // 缓冲区长度
    0);              // 必须置 0

// 创建保存散列值的缓冲区
pBuffer = new char [dwBufferSize];

// Get hash value.
bResult = CryptGetHashParam(
    hHash,            // 散列对象句柄
    HP_HASHVAL,      // 得到散列值
    pBuffer,          // 保存散列值长度缓冲区
    &dwBufferSize,    // 缓冲区长度
    0);              // 必须置 0

// 释放散列对象
CryptDestroyHash(hHash);
```

上面例子为 `pBuffer` 指向的数据生成一个散列值。如果还要散列其它数据，用这个数据调用 `CryptHashData`，产生的散列值仍会是原来的值。已警告过—使用 `HP_HASHVALUE` 参数调用 `CryptGetHashParam` 会阻止使用此对象继续进行散列。

### 生成密鈅：`CryptDeriveKey`, `CryptGenKey`, `CryptDestroyKey`

这三个函数用来产生密鈅句柄：

* `CryptDeriveKey` 函数从一个指定的密码（`password`）产生密鈅。
* `CryptGenKey` 函数从一个随机产生的数值产生密鈅。
* `CryptDestroyKey` 函数释放密鈅对象。

使用 `CryptGenKey` 函数时，建议使用 `CRYPT_EXPORTABLE` 参数以创建一个可导出的会话密鈅。这会建立一个可从一台机器移到另一台机器的值。不提供此参数，返回值仅在此机器/会话中有效。

下面是如何使用 `CryptDeriveKey` 函数的例子，假定 `pPassword` 指向一个用户指定的密码，`dwPasswordLength` 为密码长度。

```c
#include <wincrypt.h>      // 对 CryptoAPI 的定义

/*
对于非 C/C++用户，此处用到的常量如下：

#define ALG_CLASS_HASH               (4 << 13)
#define ALG_TYPE_ANY                 (0)
#define ALG_SID_MD5                  3
#define CALG_MD5         (ALG_CLASS_HASH | ALG_TYPE_ANY | ALG_SID_MD5)
#define CRYPT_EXPORTABLE             0x00000001
#define ALG_CLASS_DATA_ENCRYPT       (3 << 13)
#define ALG_TYPE_STREAM              (4 << 9)
#define ALG_SID_RC2                  2
#define CALG_RC4         (ALG_CLASS_DATA_ENCRYPT|ALG_TYPE_STREAM|ALG_SID_RC4)
*/

BOOL bResult;

HCRYPTHASH hHash;

HCRYPTKEY hKey;

// 获得散列对象句柄
bResult = CryptCreateHash(
    hProv,            // 之前获得的 CSP 句柄
    CALG_MD5,         // 散列算法
    0,                // 非密鈅散列
    0,                // 置 0
    &hHash);          // 保存散列对象句柄的变量

// 散列数据
bResult = CryptHashData(
    hHash,            // 散列对象句柄
    pPassword,        // 指向密码的指针
    dwPasswordLength, // 数据长度
    0);              // 未指定值

// 从指定的密码产生密鈅
bResult = CryptDeriveKey(
    hProv,            // 之前获得的 CSP 句柄
    CALG_RC4,         // 流加密
    hHash,            // 密码散列后对象句柄
    CRYPT_EXPORTABLE, // 密鈅可导出
    &hKey);           // 保存密鈅对象句柄的变量

.
.
.

// 用密鈅进行操作
.
.
.

// 释放散列对象
CryptDestroyHash(hHash);

// 释放密鈅对象
CryptDestroyKey(hKey);
```

### 加密与解密数据：`CryptEncrypt`, `CryptDecrypt`

简单来说，尽管不全对，加密 API 处理数据是围绕两个函数—加密(`CryptEncrypt`) 与解密(`CryptDecrypt`)。

这两个函数非常易用，但需要对其参数进行一下说明：

* 每个函数的头六个参数是相同的
* 头两个参数仅是密鈅句柄和一个可选的散列对象
* 第三个参数是一个布尔值，此值在最后一块数据块之前保持为 `FALSE`，为让函数对最后一块数据进行特殊处理，在最后一块数据时置为 `TRUE`
* 第四与第五个参数是标志值和一个指向加密或解密数据的指针
* 第六个参数是缓冲区中待加密字符的数量
* 第七个参数通常与第六个参数相同，它指出数据块长度。这是因为对于许多算法来说，加密数据尺寸与解密数据尺寸是相同的。然而，某些算法增加加密数据的长度。在这种情况下，第五个参数中的缓冲区必须大到足以容纳额外的数据。

缓冲区长度的问题可以在加密前通过调用 `CryptEncrypt` 函数返回需要缓冲区的尺寸来解决。下面的例子代码演示了这种技术。在这个例子中，某些值已假定之前已获得，我们仅要加密 `pData` 指向的缓冲区中 `dwDataLen` 字节长的数据。

```c
BOOL bResult;

PBYTE pBuffer;

DWORD dwSize;

// 将缓冲中数据长度赋给变量
dwSize = dwDataLen;

// 让 API 返回给我们需要的缓冲长度
bResult = CryptEncrypt(
    hKey,            // 之前获得的密鈅对象
    0,               // 不散列数据
    TRUE,            // 最后的还是缓冲的数据
    0,               // 必须置 0
    NULL,            // 无数据，简单的返回尺寸
    &dwSize,         // 数据的尺寸
    dwSize);         // 数据块尺寸

// 现在得到了输出缓冲区尺寸，创建此缓冲区
pBuffer = new char[dwSize];

// 加密数据
bResult = CryptEncrypt(
    hKey,            // 之前获得的密鈅对象
    0,               // 不散列数据
    TRUE,            // 最后的还是缓冲的数据
    0,               // 必须置 0
    pBuffer,         // 数据缓冲区
    &dwSize,         // 数据尺寸
    dwSize);         // 数据块尺寸
```

### 同时进行加密与解密

当使用同一个密鈅进行加密或解密两个数据流时，必须采取一些措施。同一个物理会话密鈅不得被用于同一个操作，因为每个会话密鈅容器的内部状态信息在同时进行一个操作时会混乱。对此问题的简单解决办法是制作一份会话密鈅的拷贝。这样，原始密鈅进行一个操作，拷贝密鈅进行另一个操作。

制作一个会话密鈅的拷贝可以通过调 `CryptExportKey` 导出密鈅，然后调 `CryptImportKey` 将它导进来。密鈅导入后，CSP 会给这个“新”的密鈅分配自己的内部内存区域，就好象它跟原来的密鈅完全没有关联一样。
