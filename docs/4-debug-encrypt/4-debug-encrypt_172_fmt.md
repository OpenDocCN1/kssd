
# 常见密码算法总结--(1)分组对称密码

> 原文：[`www.pediy.com/kssd/pediy11/113921.html`](https://www.pediy.com/kssd/pediy11/113921.html)

常见密码算法总结--(1)分组对称密码

`http://bbs.pediy.com/showthread.php?t=113921`

`NJZhuJinhua@csdn` May.27, 2010

`http://blog.csdn.net/njzhujinhua`

转载请注明出处。

《常见密码算法总结--(2)分组密码加密模式》见

[`bbs.pediy.com/showthread.php?t=114169`](http://bbs.pediy.com/showthread.php?t=114169)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635313.aspx`](http://blog.csdn.net/NJZhuJinhua/arc...0/5635313.aspx)

《常见密码算法总结--(3)加密模式的 `openssl` 代码分析》见

[`bbs.pediy.com/showthread.php?t=114170`](http://bbs.pediy.com/showthread.php?t=114170)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635343.aspx`](http://blog.csdn.net/NJZhuJinhua/arc...0/5635343.aspx)

## 目录

- （一）分组对称密码
- （二）非对称密码
- （三）杂凑 `Hash` 函数

### （一）分组对称密码

`DES`：

第一个得到广泛应用的密码算法，属于对称，分组密码系列，输入明文 64 位，密钥 56 位，密文 64 位。`DES` 密钥太短，已经远远不能适应保密需要。另外 `DES` 设计为用硬件实现，软件实现时效率很低，`3DES` 更加低效。

`IDEA`：

属于对称，分组密码，明文 64 位，密钥 128 位，密文 64 位。由来学嘉和 `James Massey` 提出，是一种专利算法，在欧洲使用较广。

`RC` 系列：

是 `Ron Rivest` 为 `RSA` 设计的密码算法，

`RC4`：变长密钥，`Rivest` 在 1987 年设计

`RC5`：分组长，密钥长，及轮数均可变的对称，分组密码。`Rivest` 在 1994 年设计。

`AES`:

`NIST` 发起高级加密标准的评选，要求实现更快，安全性至少要达到 `3DES` 水平，应该使用 128 位分组，支持 256 位密钥，128 与 192 位密钥也必须支持。

进入最后一轮的有 `Rijndael`, `Serpent`, `Twofish`, `RC6` and `MARS`。其中 `Rijndael` 最后胜出，成为了 `AES`。`Rijndael` 将替换 `DES-3DES`。

`Serpent` 达到了 `Rijndael` 的安全性但是运行较慢，排在第二位；运行最快的是 `RC6` 但是安全性稍逊于 `Rijndael`。所有参选的 `AES` 其密钥为 128-256 位，`Twofish` 在密钥小于 256 时暂时超过 `Rijndael`，排在第三。`Blowfish` 的安全性也很高，未见对其的有效攻击，但因其分组只有 64 位，在加密大量数据时的低效而没有入选 `AES` 的短名单。

`Rijndael` 是分组迭代密码，分组长可谓 128,192,256；密钥长可谓 128,192,256。为满足 `AES`，`Rijndael` 的分组长主要使用 128 位。`Rijndael` 的轮数为 10,12,14 轮。

`camellia`

继美国 2000 年发布 `AES` 后，2003 年 2 月欧洲最新一代的安全标准 `NESSIE`(`New European Schemes for Signatures、Integrity and Encryption`)发布，其中的两个 128 位分组密码算法是 `camellia` 和 `AES`。

`camellia` 算法支持 128 位分组，密钥可以为 128,192,256 位，接口与 `AES` 一致。对此算法尚无了解，待后续补充。

《常见密码算法总结--(1)分组对称密码》见

[`bbs.pediy.com/showthread.php?t=113921`](http://bbs.pediy.com/showthread.php?t=113921)

[`blog.csdn.net/NJZhuJinhua/arc...7/5629455.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/27/5629455.aspx)

《常见密码算法总结--(3)加密模式的 `openssl` 代码分析》见

[`bbs.pediy.com/showthread.php?t=114170`](http://bbs.pediy.com/showthread.php?t=114170)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635343.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/30/5635343.aspx)

`NJZhuJinhua@csdn` May.30, 2010

[`bbs.pediy.com/showthread.php?t=114169`](http://bbs.pediy.com/showthread.php?t=114169)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635313.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/30/5635313.aspx)

转载请注明出处。

### (二)分组密码加密模式

本来计划第二节写非对称密码的，但发在 `pediy` 看雪后读者说第一写的太粗，特决定把 `openssl` 的实现一起拉进来结合其代码分析学习。

本节讲分组密码的工作模式。

为了对长度超过密码算法分组大小的明文进行加密，设计到分组密码的工作模式的问题，简单说就是分组块进行加密时的链接关系。也可以理解为密码算法如 `DES`，`AES` 等解决的是一个分组长度的明文加密成密文的过程，而对于任意长度的明文的加密过程则以加密算法为基础，并在某种工作模式下来完成加密过程。因而单独说此数据采用什么算法加密的是没意义的，同样是 `aes` 加密，模式不一样时，密文不一样。有了密文，有了密钥，不知道加密模式的话一样无法解密。

分组密码的工作模式主要有

1. 电子密码本模式 `ECB`
2. 密码分组链接模式 `CBC`
3. 密码反馈模式 `CFB`
4. 输出反馈模式 `OFB`
5. 计数器模式 `CTR`
6. 密文挪用模式 `CTS`

#### (1)电子密码本模式 `ECB`

将明文的各个分组独立的使用相同的密钥进行加密，这种方式加密时各分组的加密独立进行互不干涉，因而可并行进行。同样因为各分组独立加密的缘故，相同的明文分组加密之后具有相同的密文。该模式容易暴露明文分组的统计规律和结构特征。不能防范替换攻击。

其实照实现来看，`ECB` 的过程只是把明文进行分组，然后分别加密，最后串在一起的过程。当消息长度超过一个分组时，不建议使用该模式。在每个分组中增加随机位(如 128 位分组中 96 位为有效明文，32 位的随机数)则可稍微提高其安全性,但这样无疑造成了加密过程中数据的扩张。

#### (2)密码分组链接模式 `CBC`

该模式将当前分组的明文与前一个分组的密文的异或作为加密算法的输入。加密后的密文继续参与下一个分组的加密过程。在第一个分组的加密时需要一个初始向量 `IV`，起到虚拟的第 0 个分组的密文的作用。

记 `c_0=IV` 的话

第 `i` 个分组 `m_i` 的加密过程可以表示为 `c_i=E_k(m_i \oplus c_{i-1})`

(这里及以后的下标等表示采用 `TEX` 语法。)

解密过程也是从第一个分组开始，第 `i` 个密文分组 `c_i` 的解密为 `m_i=E_k^{-1}(c_i) \oplus c_{i-1}`。

在 `CBC` 模式下，相同的明文经相同密钥及 `IV` 下，将产生相同的密文。改变任一个，密文则不同。这一性质说明 `CBC` 模式可以用于消息认证，即用来产生消息认证吗 `MAC`。将 `MAC` 附着在明文分组的后面，使接收消息的人可以确认消息的来源正确且中途没有被篡改过。

`CBC` 模式的自同步：由于 `CBC` 链接属性，使密文 `c_i` 依赖于 `m_i` 以及 `c_{i-1}`，而 `c_{i-1}` 又依赖于 `i-1` 明文及 `i-2` 的密文，因而 `c_i` 相当于依赖于 `m_i` 及前面所有的明文分组。因而过程中明文一位的改变影响后面所有密文的值。而解密过程中密文中一位的改变仅影响本分组以及下一个分组的解密，对于再下一个密文分组的解密则无影响。`CBC` 模式是自同步或密文自动密钥，对于位错误或丢失整个分组情况下，很快能恢复。但是，对于位丢失这样的分组边界错位则没办法了。

#### (3)密码反馈模式 `CFB`

`CFB` 模式时产生一个密钥流并与分组中的一部分尽心运算。可以将一个大分组分成很多小部分，如 128 位分组分成 16 份，每份 8 位进行一次运算，可以即时输出 8 位的密文。甚至可以分成 128 份，每次一位的输出，但如此依赖循环次数大大增加，效率极低。

加密过程可以描述为

设 `c_0=IV`

第 `i` 部分的密文为 `c_i=m_i \oplus Z_i` 其中

`Z_i=E_k(c_{i-1})`。

`CFB` 模式的加密过程与解密过程完全相同，不能用于公钥算法

#### (4)输出反馈模式 `OFB`

与 `CFB` 相同的是都是利用分组密码产生流密码的密钥流，不同的是密钥流与密文无关了，而仅与 `IV` 及密钥 `K` 有关。

加密过程可以描述为

设 `Z_0=IV`

第 `i` 部分的密文为 `c_i=m_i \oplus Z_i` 其中

`Z_i=E_k(Z_{i-1})`。

解密时直接生成 `Z=Z_1|Z_2|Z_3|Z_4|Z_5...;`

然后 `M=m_1|m_2|m_3...=c_1 \oplus Z_1 | c_2 \oplus Z_2 |...`

#### (5)计数器模式 `CTR`

计数器模式类似于 `OFB`，差别是构造密钥流的方式不一致：计数器模式的密钥流是通过使用密钥 `K` 加密系列计数器产生的。

选择一个计数器 `ctr`，是一个长度与明文分组长度相同的比特串，首先构造一系列长度为分组长度的比特串，记为:

`Z=Z_1|Z_2|Z_3|Z_4|Z_5...`

`Z_i=(ctr+i-1)mod 2^n` 这里 `n` 为分组长。

则

加密过程表示为 `c_i=m_i \oplus Z_i`

解密过程表示为 `m_i=c_i \oplus Z_i`

#### (6)`CTS` 密文挪用模式

用于处理任意长度明文数据，并产生相同长度的密文数据。除最后两个明文分组外，对前面的所有块 `CTS` 模式与 `CBC` 模式效果一样。

《常见密码算法总结--(1)分组对称密码》见

[`bbs.pediy.com/showthread.php?t=113921`](http://bbs.pediy.com/showthread.php?t=113921)

[`blog.csdn.net/NJZhuJinhua/arc...7/5629455.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/27/5629455.aspx)

《常见密码算法总结--(2)分组密码加密模式》见

[`bbs.pediy.com/showthread.php?t=114169`](http://bbs.pediy.com/showthread.php?t=114169)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635313.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/30/5635313.aspx)

`NJZhuJinhua@csdn` May.30, 2010

[`bbs.pediy.com/showthread.php?t=114170`](http://bbs.pediy.com/showthread.php?t=114170)

[`blog.csdn.net/NJZhuJinhua/arc...0/5635343.aspx`](http://blog.csdn.net/NJZhuJinhua/archive/2010/05/30/5635343.aspx)

转载请注明出处。

### （三）加密模式的 `openssl` 代码分析

`openssl`（本文所用 `openssl` 代码为 1.0.0 版本，2010.03.29 发布。`http://www.openssl.org` 本文所用的内容最近几年应该未有更新，因而版本似乎也没多少关系）代码中 `crypto\modes` 目录下提供了 `cbc128`,`cfb128`,`ctr128`,`cts128`,`ofb128` 模式的实现。`modes.h` 提供接口声明，其余诸文件分别提供了各种模式。

代码:



```c
typedef void (*block128_f)(const unsigned char in[16],
                            unsigned char out[16],
                            const void *key);

typedef void (*cbc128_f)(const unsigned char *in, unsigned char *out,
                         size_t len, const void *key,
                         unsigned char ivec[16], int enc);

// cbc128 的加密与解密
void CRYPTO_cbc128_encrypt(const unsigned char *in, unsigned char *out,
                          size_t len, const void *key,
                          unsigned char ivec[16], block128_f block);
void CRYPTO_cbc128_decrypt(const unsigned char *in, unsigned char *out,
                          size_t len, const void *key,
                          unsigned char ivec[16], block128_f block);

// ctr128 的加密与解密
void CRYPTO_ctr128_encrypt(const unsigned char *in, unsigned char *out,
                          size_t len, const void *key,
                          unsigned char ivec[16], unsigned char ecount_buf[16],
                          unsigned int *num, block128_f block);

// ofb128 的加密与解密
void CRYPTO_ofb128_encrypt(const unsigned char *in, unsigned char *out,
                          size_t len, const void *key,
                          unsigned char ivec[16], int *num,
                          block128_f block);

// cfb128 的加密与解密
void CRYPTO_cfb128_encrypt(const unsigned char *in, unsigned char *out,
                          size_t len, const void *key,
                          unsigned char ivec[16], int *num,
                          int enc, block128_f block);
void CRYPTO_cfb128_8_encrypt(const unsigned char *in, unsigned char *out,
                            size_t length, const void *key,
                            unsigned char ivec[16], int *num,
                            int enc, block128_f block);
void CRYPTO_cfb128_1_encrypt(const unsigned char *in, unsigned char *out,
                            size_t bits, const void *key,
                            unsigned char ivec[16], int *num,
                            int enc, block128_f block);

// cts128 模式
size_t CRYPTO_cts128_encrypt_block(const unsigned char *in, unsigned char *out,
                                   size_t len, const void *key,
                                   unsigned char ivec[16], block128_f block);
size_t CRYPTO_cts128_encrypt(const unsigned char *in, unsigned char *out,
                            size_t len, const void *key,
                            unsigned char ivec[16], cbc128_f cbc);
size_t CRYPTO_cts128_decrypt_block(const unsigned char *in, unsigned char *out,
                                  size_t len, const void *key,
                                  unsigned char ivec[16], block128_f block);
size_t CRYPTO_cts128_decrypt(const unsigned char *in, unsigned char *out,
                            size_t len, const void *key,
                            unsigned char ivec[16], cbc128_f cbc);
```

我们按照 `cbc`、`cfb`、`ofb` 的顺序进行。

(1) `CRYPTO_cbc128_encrypt`、`CRYPTO_cbc128_decrypt`

该函数在 `libeay32` 的实现中被以下代码调用：
* `seed_cbc.c` (`SEED_cbc_encrypt()`)
* `cts128.c` (`CRYPTO_cts128_encrypt_block()`)
* `cmll_cbc.c` (`Camellia_cbc_encrypt()`)
* `aes_cbc.c` (`AES_cbc_encrypt()`)

例如 `aes_cbc.c` 中:

```c
void AES_cbc_encrypt(const unsigned char *in, unsigned char *out,
                    size_t len, const AES_KEY *key,
                    unsigned char *ivec, const int enc) {

  if (enc)
    CRYPTO_cbc128_encrypt(in, out, len, key, ivec, (block128_f)AES_encrypt);
  else
    CRYPTO_cbc128_decrypt(in, out, len, key, ivec, (block128_f)AES_decrypt);
}
```

`AES` 的加解密公用一个函数，具体是加密还是解密通过参数 `enc` 指示。`AES_cbc_encrypt` 通过将参数原封不动传递给 `CRYPTO_cbc128_encrypt` 或者 `CRYPTO_cbc128_decrypt` 完成加解密。最后一个回调函数 `AES_encrypt` 和 `AES_decrypt` 即使用该模式的加密算法，用于加密一个分组。

其他模式也基本同这例子，后面的模式不再详细说明其怎么跟加密算法的结合。

代码:



```c
void CRYPTO_cbc128_encrypt(const unsigned char *in, unsigned char *out,
                 size_t len, const void *key,
                 unsigned char ivec[16], block128_f block)
{
    size_t n;
    const unsigned char *iv = ivec;

    //确保明文，密文，密钥及初始向量指针的有效性
    assert(in && out && key && ivec);

#if !defined(OPENSSL_SMALL_FOOTPRINT)
    if (STRICT_ALIGNMENT &&
        ((size_t)in|(size_t)out|(size_t)ivec)%sizeof(size_t) != 0) 
    {
        // cpu 对其，但 in out 及 ivec 中有一个不是 sizeof(size_t)字节对齐的，如地址为 0x08000001，sizeof(size_t)==4 情况下
        while (len>=16) 
        {
            for(n=0; n<16; ++n)
                out[n] = in[n] ^ iv[n];【A】
            // 第一次循环时 iv 为入参给的 ivec，之后的循环已经在【C】处被设为新产生的密文了

            (*block)(out, out, key);【B】
            // 调用具体的回调函数，第一个参数为【A】处算出的明文分组与上一个密文分组的异或分组。第二个参数 out 为出参，意思是加密之后的值还写到 out 所在的内存。key 为提供的密钥。
            iv = out;【C】
            // 更新密文分组，用于下一分组的明文进行异或。
            len -= 16;
            in  += 16;
            out += 16;
            // 移动指针等，准备处理下一分组。当剩余的 len 小于 16 时，或压根输入就小于 16 时进行后面【D】的流程
        }
    } 
    else 
    {
        // 这个与上一个目的一样，不过对已经对齐的情形进行了优化，一次处理了 sizeof(size_t)个字节的异或运算而已。
        while (len>=16) 
        {
            for(n=0; n<16; n+=sizeof(size_t))
                *(size_t*)(out+n) =
                *(size_t*)(in+n) ^ *(size_t*)(iv+n);
            (*block)(out, out, key);
            iv = out;
            len -= 16;
            in  += 16;
            out += 16;
        }
    }
#endif    
    while (len) 【D】
    // 如果定义了 OPENSSL_SMALL_FOOTPRINT 则上述并未执行，len 仍是入参的 len。否则这里的 len 经上面处理后应该是小于 16 的了。下面处理总体来说逻辑跟上面差不多，多了 len<16 的情形的处理。
    {
        for(n=0; n<16 && n<len; ++n)
            out[n] = in[n] ^ iv[n];【E】
        // 如果实际剩余长度 len 小于 16 则对 len 字节进行异或运算，否则对分组长 16 字节进行运算。
        for(; n<16; ++n)
            out[n] = iv[n];【F】
        // 如果上述 len 小于 16 的情形，需要执行【F】，及最后一个分组的明文的最后字节用上一个分组的密文对应字节填充(在明文总长度就小于 16 时则是用初始向量填充了)
        (*block)(out, out, key);
        // 对分组加密
        iv = out;
        if (len<=16) break;
        // 退出
        len -= 16;
        in  += 16;
        out += 16;
    }
    memcpy(ivec,iv,16);
}

// cbc128 模式的解密函数
void CRYPTO_cbc128_decrypt(const unsigned char *in, unsigned char *out,
                 size_t len, const void *key,
                 unsigned char ivec[16], block128_f block)
{
    size_t n;
    union { size_t align; unsigned char c[16]; } tmp;

    assert(in && out && key && ivec);

#if !defined(OPENSSL_SMALL_FOOTPRINT)
    if (in != out) // 解密后的明文是否放在密文的内存中
    {
        const unsigned char *iv = ivec;

        if (STRICT_ALIGNMENT &&
            ((size_t)in|(size_t)out|(size_t)ivec)%sizeof(size_t) != 0) 
        {
            while (len>=16) 
            {
                (*block)(in, out, key); // 解密一个分组
                for(n=0; n<16; ++n)  // 到这里 out 为该分组对应的明文分组与前一个密文分组的异或。
                    out[n] ^= iv[n];
                iv = in; // 到这里后 out 为明文分组了
                len -= 16;
                in  += 16;
                out += 16;
            }
        }
        else 
        {
            while (len>=16) 
            {
                (*block)(in, out, key); // 与上同理
                for(n=0; n<16; n+=sizeof(size_t))
                    *(size_t *)(out+n) ^= *(size_t *)(iv+n);
                iv = in;
                len -= 16;
                in  += 16;
                out += 16;
            }
        }
        memcpy(ivec,iv,16);
    } 
    else 
    {
        // 如果解密后的明文放到原密文的地方，则执行此部分。除了使用临时变量 tmp 中转一下外与上一样，略
        if (STRICT_ALIGNMENT &&
            ((size_t)in|(size_t)out|(size_t)ivec)%sizeof(size_t) != 0) 
        {
            unsigned char c;
            while (len>=16) 
            {
                (*block)(in, tmp.c, key);
                for(n=0; n<16; ++n) 
                {
                    c = in[n];
                    out[n] = tmp.c[n] ^ ivec[n];
                    ivec[n] = c;
                }
                len -= 16;
                in  += 16;
                out += 16;
            }
        }
        else 
        {
            // 字节对齐时的优化操作
            size_t c;
            while (len>=16) {
                (*block)(in, tmp.c, key);
                for(n=0; n<16; n+=sizeof(size_t)) {
                    c = *(size_t *)(in+n);
                    *(size_t *)(out+n) =
                    *(size_t *)(tmp.c+n) ^ *(size_t *)(ivec+n);
                    *(size_t *)(ivec+n) = c;
                }
                len -= 16;
                in  += 16;
                out += 16;
            }
        }
    }
#endif     
    while (len) 
    {
        // 解密最后一个分组，或 openssl 定义了 OPENSSL_SMALL_FOOTPRINT 的话这里解密所有分组。与加密时基本一样，略
        unsigned char c;
        (*block)(in, tmp.c, key);
        for(n=0; n<16 && n<len; ++n) {
            c = in[n];
            out[n] = tmp.c[n] ^ ivec[n];
            ivec[n] = c;
        }
        if (len<=16) {
            for (; n<16; ++n)
                ivec[n] = in[n];
            break;
        }
        len -= 16;
        in  += 16;
        out += 16;
    }
}
```

(2) `CRYPTO_cfb128_encrypt` `CRYPTO_cfb128_1_encrypt` `CRYPTO_cfb128_8_encrypt`

这三个实现了 cfb 模式的三种制式。其函数声明见上文描述。

这三种均被 `seed_cfb.c` `cmll_cfb.c` `aes_cfb.c` 引用。

在 `aes_cfb.c` 内使用情况如下：

```c
void AES_cfb128_encrypt(const unsigned char *in, unsigned char *out,
                    size_t length, const AES_KEY *key,
                    unsigned char *ivec, int *num, const int enc) {
    
    CRYPTO_cfb128_encrypt(in,out,length,key,ivec,num,enc,(block128_f)AES_encrypt);
}

/* N.B. This expects the input to be packed, MS bit first */
void AES_cfb1_encrypt(const unsigned char *in, unsigned char *out,
                    size_t length, const AES_KEY *key,
                    unsigned char *ivec, int *num, const int enc)
{
    CRYPTO_cfb128_1_encrypt(in,out,length,key,ivec,num,enc,(block128_f)AES_encrypt);
}

void AES_cfb8_encrypt(const unsigned char *in, unsigned char *out,
                    size_t length, const AES_KEY *key,
                    unsigned char *ivec, int *num, const int enc)
{
    CRYPTO_cfb128_8_encrypt(in,out,length,key,ivec,num,enc,(block128_f)AES_encrypt);
}
```
