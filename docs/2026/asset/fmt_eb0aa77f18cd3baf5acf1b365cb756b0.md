
```c
unsigned int __cdecl hmac_ctx_hash(BYTE *data, BYTE *key, BYTE *res)
{
    int md; // esi
    __int64 v4; // xmm1_8
    __int64 v5; // xmm0_8
    int len; // [esp+8h] [ebp-154h] BYREF
    char v8; // [esp+Eh] [ebp-14Eh] BYREF
    char v9; // [esp+Fh] [ebp-14Dh] BYREF
    BYTE hash3[32]; // [esp+10h] [ebp-14Ch] BYREF
    BYTE hash2[32]; // [esp+30h] [ebp-12Ch] BYREF
    int ctx[52]; // [esp+50h] [ebp-10Ch] BYREF
    BYTE hash1[40]; // [esp+120h] [ebp-3Ch] BYREF
    unsigned int canary; // [esp+148h] [ebp-14h]

    canary = __readgsdword(0x14u);
    memset(hash1, 0, 32);
    HMAC_CTX_init((int)ctx); // 初始化一个 HMAC_CTX
    md = EVP_sha256(); // 返回 sha256 的 EVP_MD
    HMAC_Init_ex((int)ctx, (int)key, 32, md, 0); // 初始化HAMC_CTX上下文结构，key为秘钥，len为秘钥长度，md为计算hash的函数集合
    HMAC_Update((int)ctx, (int)data, 0); // 向HMAC上下文输入字节流，加密并输出，传入长度为0
    // 经过 demo 验证，该条指令不会对上下文产生影响
    len = 0;
    HMAC_Final((int)ctx, (int)hash1, (int)&len); // 生成最终的HMAC串，成功时len更新为HMAC的长度。
    memset(hash2, 0, sizeof(hash2));
    v9 = 1;
    HMAC_CTX_init((int)ctx);
    HMAC_Init_ex((int)ctx, (int)hash1, 32, md, 0); // 把 hash1 作为 key
    HMAC_Update((int)ctx, (int)data, 0);
    HMAC_Update((int)ctx, (int)&v9, 1); // 输入字节流 1,长度为1
    len = 0;
    HMAC_Final((int)ctx, (int)hash2, (int)&len); // 得到第二次的 hash
    memset(hash3, 0, sizeof(hash3));
    v8 = 2;
    HMAC_CTX_init((int)ctx);
    HMAC_Init_ex((int)ctx, (int)hash1, 32, md, 0); // 把第一次的 hash 作为 key 再次加密
    HMAC_Update((int)ctx, (int)hash2, 32); // 输入字节流 hash2，长度为32
    HMAC_Update((int)ctx, (int)&v8, 1); // 输入字节流 2，长度为1
    len = 0;
    HMAC_Final((int)ctx, (int)hash3, (int)&len); // 得到第三次的 hash
    v4 = *(_QWORD *)&hash2[8]; // v4 = hash2[8-15]
    *(_QWORD *)res = *(_QWORD *)hash2; // res[0-7] = hash2[0-7]
    *((_QWORD *)res + 1) = v4; // res[8-15] = hash2[8-15]
    *((_QWORD *)res + 2) = *(_QWORD *)&hash2[16]; // res[16-23] = hash2[16-23]
    *((_QWORD *)res + 3) = *(_QWORD *)&hash2[24]; // res[24-31] = hash2[24-31]
    *((_QWORD *)res + 7) = *(_QWORD *)&hash3[24]; // res[56-63] = hash3[24-31]
    *((_QWORD *)res + 6) = *(_QWORD *)&hash3[16]; // res[48-55] = hash3[16-31]
    v5 = *(_QWORD *)hash3;
    *((_QWORD *)res + 5) = *(_QWORD *)&hash3[8]; // res[40-47] = hash3[8-15]
    *((_QWORD *)res + 4) = v5; // res[32-39] = hash3[0-7]
    return __readgsdword(0x14u);
}
```

### 测试demo

```c
#include "openssl/hmac.h"
#include "openssl/evp.h"
#include "openssl/aes.h"
#pragma comment(lib, "libcrypto.lib")

int main()
{
    unsigned char key[32] = { 0 };
    unsigned char data[32] = { 0 };
    unsigned char hash[32] = { 0 };
    unsigned int len = 0;
    for (int i = 0; i < 32; i++)
    {
        key[i] = i;
        data[i] = i*2;
    }

    HMAC_CTX *ctx = HMAC_CTX_new();
    const EVP_MD *md = EVP_sha256();
    HMAC_Init_ex(ctx, key, 32, md, 0);
    //HMAC_Update(ctx, data, 0); // 该条语句注释未产生任何影响
    HMAC_Final(ctx, hash, &len);
}
```

#### 分析函数 `evp_encrypt`（部分二）

这个函数的总的来说就是使用上一层传入的`hash`作为`key`，加密用户名得到`encrypt_data`，然后将`encrypt_data`分段赋值给最终的`res_encrypt_data`，分析这个函数的问题也还是在于作者使用了`EVP_EncryptUpdate`这个不熟悉的函数，并且还总是在参数中不该传0的传0，无法确定是否对上下文产生了影响，解决方法是花了一些时间在VS中搭了一下环境，测试了各种情况的结果。

```c
int __cdecl evp_encrypt(
    int hash,
    int iv,
    int iv_len,
    BYTE *const_key,
    int const_key_length,
    int byte_user_name,
    int user_name_length,
    char *res_encrypt_data)
{
    int ctx; // esi
    int cipher; // eax
    size_t v10; // eax
    __int64 v11; // xmm1_8
    int v13; // [esp+0h] [ebp-41Ch] BYREF
    size_t encrypt_data_len; // [esp+4h] [ebp-418h] BYREF
    BYTE encrypt_data[1024]; // [esp+8h] [ebp-414h] BYREF
    unsigned int v16; // [esp+408h] [ebp-14h]

    v16 = __readgsdword(0x14u);
    ctx = EVP_CIPHER_CTX_new(); // 创建加密上下文
    cipher = EVP_aes_256_gcm(); // 选择一种加密算法
    EVP_EncryptInit_ex(ctx, cipher, 0, 0, 0); // 初始化密码上下文ctx
    EVP_CIPHER_CTX_ctrl(ctx, 9, iv_len, 0); // # define EVP_CTRL_AEAD_SET_IVLEN 0x9
    // 设置向量IV的长度
    EVP_EncryptInit_ex(ctx, 0, 0, hash, iv); // 使用hash作为key初始化ctx
    EVP_EncryptUpdate(ctx, 0, (int)&encrypt_data_len, (int)const_key, const_key_length); // 加密 const_key，让人困惑的是输出地址为0，
    // 无法确定是否会对上下文产生影响，经过 demo 测试最终确定加密结果被丢弃
    EVP_EncryptUpdate(ctx, (int)encrypt_data, (int)&encrypt_data_len, byte_user_name, user_name_length); // 加密用户名
    memcpy(res_encrypt_data, encrypt_data, encrypt_data_len); // dest[0-15] = encrypt_data[0-15]
    // 将第二次EncryptUpdate的结果拷贝到dest中，而且经过动态调试得知，encrypt_data_len=16
    EVP_EncryptFinal_ex(ctx, (int)encrypt_data, (int)&v13); // 块对齐
    EVP_CIPHER_CTX_ctrl(ctx, 16, 16, (int)encrypt_data); // # define EVP_CTRL_AEAD_GET_TAG 0x10   获取标签
    v10 = encrypt_data_len;
    v11 = *(_QWORD *)&encrypt_data[8]; // v11 = encrypt_data[8-15]
    *(_QWORD *)&res_encrypt_data[encrypt_data_len] = *(_QWORD *)encrypt_data; // dest[16-23] = encrypt_data[0-7]
    *(_QWORD *)&res_encrypt_data[v10 + 8] = v11; // dest[24-31] = v11 = encrypt_data[8-15]
    EVP_CIPHER_CTX_free(ctx); // 释放上下文
    return encrypt_data_len + 16; // 返回加密结果的长度+16
}
```

### 测试demo

```c
#include "openssl/hmac.h"
#include "openssl/evp.h"
#include "openssl/aes.h"
#pragma comment(lib, "libcrypto.lib")

int main()
{
    unsigned char key[32] = { 0 };
    unsigned char iv[32] = { 0 };
    unsigned char encrypt_data[32] = { 0 };
    unsigned char user_name[16] = { 'a','b','c','d', 'a','b','c','d', 'a','b','c','d', 'a','b','c','d' };
    const unsigned char *const_key = (const unsigned char *)"123456awxzcdfqwqt2wetbwerw";
    int len = 0;
    for (int i = 0; i < 32; i++)
    {
        key[i] = i;
        iv[i] = i;
    }

    EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
    const EVP_CIPHER *cipher = EVP_aes_256_gcm();
    EVP_EncryptInit_ex(ctx, cipher, 0, key, iv);
    EVP_CIPHER_CTX_ctrl(ctx, 9, 12, 0);
    //EVP_EncryptUpdate(ctx, 0, &len, const_key, 27);// 该条语句注释未产生任何影响

    len = 0;
    EVP_EncryptUpdate(ctx, encrypt_data, &len, user_name, 16);
    len = 0;
    EVP_EncryptFinal_ex(ctx, encrypt_data, &len);
}
```

#### 二次加密 `encrypt_data`（部分三）

这一部分主要就是通过`hash`第二次加密`encrypt_data`得到`encrypt_data2`。

```c
*(__m128i *)hash_temp = _mm_load_si128((const __m128i *)&hash[48]);
// 从hash的第48个字节开始加载128bits,也就是加载16个字节
......
do
{
    v28 = (v_esi + 1) ^ hash_temp[v_esi & 0xF];
    // hash48[0-F] 分别与 1 进行异或，也就是说src每一位如果是
    // 奇数：减一
    // 偶数：不变
    encrypt_data2[v_esi] =
        v28 ^ (((unsigned __int8)(encrypt_data[v_esi] + v28) >> 4) | (16* (encrypt_data[v_esi] + v28)));
    // encrypt_data[i] + v28 看成一个整体结果记为 res
    // res转成了无符号类型所以是逻辑右移，左侧用零补齐
    // res乘以16等价于逻辑左移4位
    // 其实就是把res的高四位和低四位颠倒了一下
    // 最后再和v28异或
}
while (v8 != ++v_esi); // 循环32次
```

#### 分析函数 `encrypt_4`（部分四）

这个函数主要分为三个部分：

- 初始化数组`v21`、`v22`，需要注意的是`v21`与`v22`在内存中相邻
