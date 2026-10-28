
# 代码角度看SSL握手过程

## 以 `ECDHE_RSA_WITH_AES_128_GCM_SHA256` 密码套件为例

**QQ**：`181S3493S7`

### 名词解释
密码套件 -- SSL握手过程联合使用了一系列加密&摘要算法，这样一组算法的集合，称为密码套件。

### 调试信息输出代码

`fa3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6f1c8X3i4K6u0r3j5i4c8D9M7H3`.` .` -- windows

fork自

`558K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0M7Y4m8J5k6g2)9J5c8X3q4@1L8s2x3`.`.

### 测试方法
编译运行运行`atls`;
下载安装`openssl`，`openssl s_client -connect 127.0.0.1:44444`;
用`wireshark`抓`loopback`包，过滤条件是`tls`;

### SSL服务器视角
`a_tls_handshake[1]`: `a_tls_get_client_hello()`
`a_tls_handshake[2]`: `a_tls_send_srv_hello()`
`a_tls_handshake[8]`: `a_tls_send_srv_cert()`
`a_tls_handshake[3]`: `a_tls_send_srv_ke()` Key Exchange
`a_tls_handshake[4]`: `a_tls_send_srv_hello_done()`
`a_tls_handshake[12]`: `a_tls_get_client_ke()` Key Exchange
`a_tls_handshake[11]`: `a_tls_get_client_ccs()` Change Cipher Spec
`a_tls_handshake[14]`: `a_tls_get_client_finished()`
`a_tls_handshake[6]`: `a_tls_send_srv_ticket()`
`a_tls_handshake[5]`: `a_tls_send_srv_ccs()` Change Cipher Spec
`a_tls_handshake[10]`: `a_tls_send_srv_finished()`

#### `a_tls_get_client_hello()`
这里有个两个重要信息，和一些扩展信息
1. 重要的是 `Client random` `32 Bytes`
2. 重要的是支持的加密套件 `list`

扩展信息里给出了
1. 说明了 `client` 支持的椭圆曲线 `list`
2. 支持的 `hash` 与 `sign` 算法的 `list`；`sigalg_pair_t g_sigalg_pair[A_TLS_MAX_SIG_ALG]`

#### `a_tls_send_srv_hello()`
1 重要的是 `Server random` `32 Bytes`
2 服务器检查 `client` 支持的密码套件组，如果服务器支持，则从中选则一个返回给 `client`,
本文以 `ECDHE_RSA_WITH_AES_128_GCM_SHA256` 为例

`ECDH` - `Elliptic Curve Diffie–Hellman key exchange` - 椭圆曲线迪菲-赫尔曼密钥交换  
`ECDHE` - `(ECDH Ephemeral)` 临时 `ECDH` - 保证前向安全

#### `a_tls_send_srv_cert()`
服务器把自己的证书（链）发给 `client`

#### `a_tls_send_srv_ke()`
服务器用 `ECDHE` 算法，选择一个 `server` 支持的椭圆曲线，本例为已命名的椭圆曲线，`secp256r1`，
注意 `client hello` 的扩展里可能说明了 `client` 支持的椭圆曲线 `list`

用该椭圆曲线，生成一个 `65bytes` 的公钥；

这里选择用 `client_hello` 里说明的 `hash` 与 `sign` 算法，`rsa_pss_rsae_sha256` : `0x0804`;
`hash` 算法为 `sha256`;
`client random` `32bytes` + `server random` `32bytes` + `1` + `2` + `1` + `ECDH` 公钥 `65bytes`，
上面共 `0x85` 字节过一次 `hash`;

接着是对上面 `hash` 的结果做 `rsa_pss_padding`;
有一个等于当前 `RSA` 密钥长度的 `buf`（`0x100` 字节），内容清 `0`，密钥长度是证书决定的；
生成一个 `hashlen`(`0x20` 字节)的随机数作为 `salt`;
`buf - 1 - 0x20*2 - 1` 处一字节存一个 `"1"`，后面 `20` 字节存这个 `salt`;
`0x00 * 8`（`8` 字节） + 上次 `hash` 结果（`0x20` 字节） + `salt`（`0x20` 字节），再来一次 `hash`;
这个 `hash` 挨着上面的 `salt` 存。

然后进入 `mgf1` 算法（掩码生成函数），
```
用上面最后的hash值（0x20字节），加一个count(4字节 大端序)，count从0开始计数；
循环（count++）给这0x20+count4字节做hash，用这个hash，与buf[i]处异或填充buf[i]，i从0到HashLen.
也就是每做一次hash填32个字节（HashLen == 0x20字节）；
填充范围是上面的buf从头到(buf+0x100-HashLen-1)处；
最后不到20字节的，按实际剩余字节数填充。
buf最后一个字节置0xbc,首字节out[0] &= 0xFF >> 1;
```
`mgf1` 算法结束，拼出 `buf`(`0x100` 字节：`RSA` 密钥长度）；

然后 `RSA_private_encrypt(0x100, buf（0x100字节）, out, rsa, RSA_NO_PADDING)` 生成的 `out` 也是 `0x100` 字节；
这 `0x100` 个字节就是 `RSA sign` 的内容；

#### `a_tls_send_srv_hello_done()`
简单（略）

#### `a_tls_get_client_ke()`
收到 `client` 的 `ECDH` 的公钥 `65` 字节，算出 `pms` `0x20` 字节，
用 `"master secret"` `13` 字节，+ `client_random` `0x20` 字节，+ `srv_random` `0x20` 字节，共 `0x4d` 字节；
与 `pms` `0x20` 字节一起，经过 `phash` 算法(内部调用 `HMAC` 算法(参数调用 `sha256`，一种标准的 `hash` 加盐算法))，算出 `48` 字节的 `master_Key`.

#### `a_tls_get_client_ccs()`
用 `""key expansion"` `13` 字节，+ `srv_random` `0x20` 字节，+ `client_random` `0x20` 字节，共 `0x4d` 字节；
与 `master_Key` `0x30` 字节一起，经过 `phash` 算法(内部调用 `HMAC` 算法(参数调用 `sha256`，一种标准的 `hash` 加盐算法))，算出 `0x68` 字节的 `Key_Block`;
`Key_Block` 包含读与写的 `cipher_key` `0x10` 字节，`IV` `4` 字节。

`a_tls_init_cipher` 用 `Key_Block` 各部分（读的 `cipher_key` `0x10` 字节，`IV` `4` 字节）初始化对称解密算法（`AES_128_GCM`）;
这里初始化的是读取 `client` 消息的对称解密算法。

#### `a_tls_get_client_finished()`
第一次收到被对称加密的数据，具体解密方式（`AES_128_GCM`）为以前写的 `demo` 的解密算法，不再叙述；
解出来的值为，当前所有接收和发送的握手（数据类型为 `HANDHSHAKE = 0x16`）数据（除当前的 `finished` 消息外），经过一次 `sha256` 得到一个 `hash` 值,
`"client finished"` `15` 个字节， + 上面 `hash` 值 `0x20` 个字节，与 `MASTER_KEY`（`48` 字节）一起；
经过 `phash` 算法(内部调用 `HMAC` 算法(参数调用 `sha256`，一种标准的 `hash` 加盐算法))，得到 `12` 字节的值，
这个值就是 `client` 发来的值，不一致的话说明被篡改；

#### `a_tls_send_srv_ticket()`
`atls` 这里只是简单实现，不是标准；

#### `a_tls_send_srv_ccs()` Change Cipher Spec
`a_tls_init_cipher` 用 `Key_Block` 各部分（写的 `cipher_key` `0x10` 字节，`IV` `4` 字节）初始化对称解密算法（`AES_128_GCM`）;
这里初始化的是 `server` 发送消息的对称加密算法。

#### `a_tls_send_srv_finished()`
第一次发送被对称加密的数据；
当前所有接收和发送的握手（数据类型为 `HANDHSHAKE = 0x16`）数据（除当前的 `finished` 消息外），经过一次 `sha256` 得到一个 `hash` 值,
`"server finished"` `15` 个字节， + 上面 `hash` 值 `0x20` 个字节，与 `MASTER_KEY`（`48` 字节）一起；
经过 `phash` 算法(内部调用 `HMAC` 算法(参数调用 `sha256`，一种标准的 `hash` 加盐算法))，得到 `12` 字节的值，
这个值就是发给 `client` 的值，`client` 验证不一致的话说明被篡改；

到这里握手完毕，到 `client` 向服务器发 `GET` 了。

注意 `GCM` 加密模式，这里开始读/写都算第二次加密了，因为读/写 `finished` 消息就是第一次加密。

- END -
