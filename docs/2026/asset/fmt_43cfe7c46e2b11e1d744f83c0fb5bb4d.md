
```cpp
int \_\_cdecl sub\_46660(BYTE \*byte\_user\_name, unsigned int user\_name\_length, char \*dest)
{
......
canary = \_\_readgsdword(0x14u); // 开启canary保护
memset(&key\_data[32], 0, 32);
v3 = -32;
do
{
v4 = lrand48(); // 产生一个正的长整型随机数
key\_data[v3++ + 64] = v4 + v4 / 255; // 为 key\_data[32-63] 位赋值随机数
}
while ( v3 );
memset(key\_data, 0, 32);
v5 = -32;
do
{
v\_eax = lrand48();
key\_data[v5++ + 32] = v\_eax + v\_eax / 255; // 为 key\_data[0-31] 位赋值随机数
}
while ( v5 );
memset(hash, 0, sizeof(hash));
hmac\_ctx\_hash(&key\_data[32], key\_data, hash); // 参数1:data 参数2:key 参数3:dest
strcpy((char \*)const\_key, "123456awxzcdfqwqt2wetbwerw");
memset(encrypt\_data, 0, sizeof(encrypt\_data));
evp\_encrypt((int)hash, (int)&hash[32], 12, const\_key, 27, (int)byte\_user\_name, user\_name\_length, encrypt\_data);// 加密用户名
memset(encrypt\_data2, 0, sizeof(encrypt\_data2));
\*(\_\_m128i \*)hash\_temp = \_mm\_load\_si128((const \_\_m128i \*)&hash[48]);// 从hash的第48个字节开始加载128bits,也就是加载16个字节
\_\_memcpy\_chk((int)encrypt\_data2, (int)encrypt\_data, user\_name\_length + 16, 0x2800);
if ( !(((int)(user\_name\_length + 16) < 0) ^ \_\_OFADD\_\_(16, user\_name\_length) | (user\_name\_length == -16)) )// 用户名长度为16，所以 1 ^ 0 | 0 = 1
// \_\_OFADD\_\_ 测试两数相加后是否溢出
{
v\_esi = 0;
v8 = user\_name\_length + 16; // v8 = 32
if ( user\_name\_length >= 4294967280 ) // //检查用户名长度是否大于4294967280
goto LABEL\_15;
if ( user\_name\_length + 15 > 15 ) // 检查用户名长度是否大于 0，此处跳转至LABEL15，感谢手下留情
goto LABEL\_15;
v\_esi = v8 & 0xFFFFFFF0;
si128 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11535] + 1));
v10 = \_mm\_load\_si128((const \_\_m128i \*)(&off\_1A25DC - 92275));
v11 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11534] + 1));
v12 = \_mm\_load\_si128((const \_\_m128i \*)(&off\_1A25DC - 92267));
v13 = 0;
v34 = \*(\_\_m128i \*)hash\_temp;
v31 = \*(\_\_m128i \*)(&(&off\_1A25DC)[-11533] + 1);
v33 = \*(\_\_m128i \*)(&off\_1A25DC - 92259);
v32 = \_mm\_load\_si128((const \_\_m128i \*)(&(&off\_1A25DC)[-11532] + 1));
......
while ( v\_esi != v13 );
if ( v8 != v\_esi )
{
LABEL\_15:
do
{
v28 = (v\_esi + 1) ^ hash\_temp[v\_esi & 0xF];// hash48[0-F] 分别与 1 进行异或，也就是说src每一位如果是
// 奇数：减一
// 偶数：不变
encrypt\_data2[v\_esi] = v28 ^ (((unsigned \_\_int8)(encrypt\_data[v\_esi] + v28) >> 4) | (16
\* (encrypt\_data[v\_esi] + v28)));//
// encrypt\_data[i] + v28 看成一个整体结果记为 res
// res转成了无符号类型所以是逻辑右移，左侧用零补齐
// res乘以16等价于逻辑左移4位
// 其实就是把res的高四位和低四位颠倒了一下
// 最后再和v28异或
}
while ( v8 != ++v\_esi ); // 循环32次
}
}
memset(s, 0, 960u);
\*(\_OWORD \*)hash\_temp = \*(\_OWORD \*)hash;
\*(\_OWORD \*)&hash\_temp[16] = \*(\_OWORD \*)&hash[16];
\*(\_OWORD \*)&hash\_temp[32] = \*(\_OWORD \*)&hash[32];
\*(\_OWORD \*)&hash\_temp[48] = \*(\_OWORD \*)&hash[48];// hash\_temp[0-63] = hash[0-63]
\_\_memcpy\_chk((int)s, (int)encrypt\_data2, user\_name\_length + 16, 960);//这里把密文进行了一次拷贝，但看起来后续没有用到
\*(\_OWORD \*)v39 = 0LL;
return encrypt\_4(hash\_temp, user\_name\_length + 80, (int)v39, 16, dest);// 第三次加密得到最终的dest
}
```

### 分析函数 `hmac_ctx_hash`（部分一）

这个函数总的来讲就是不断的将求得的`hash`作为`key`再次求`hash`，在分析这个函数时遇到的问题主要在于作者使用了 `HMAC_Update` 并且还总是在参数中不该传0的传0，无法确定是否对上下文产生了影响，解决方法是花了一些时间在 VS 中搭了一下环境，写了一个简单的 demo 进行简单的测试。

```cpp
unsigned int \_\_cdecl hmac\_ctx\_hash(BYTE \*data, BYTE \*key, BYTE \*res)
{
int md; // esi
\_\_int64 v4; // xmm1\_8
\_\_int64 v5; // xmm0\_8
int len; // [esp+8h] [ebp-154h] BYREF
char v8; // [esp+Eh] [ebp-14Eh] BYREF
char v9; // [esp+Fh] [ebp-14Dh] BYREF
BYTE hash3[32]; // [esp+10h] [ebp-14Ch] BYREF
BYTE hash2[32]; // [esp+30h] [ebp-12Ch] BYREF
int ctx[52]; // [esp+50h] [ebp-10Ch] BYREF
BYTE hash1[40]; // [esp+120h] [ebp-3Ch] BYREF
unsigned int canary; // [esp+148h] [ebp-14h]
canary = \_\_readgsdword(0x14u);
memset(hash1, 0, 32);
HMAC\_CTX\_init((int)ctx); // 初始化一个 HMAC\_CTX
md = EVP\_sha256(); // 返回 sha256 的 EVP\_MD
HMAC\_Init\_ex((int)ctx, (int)key, 32, md, 0); // 初始化HAMC\_CTX上下文结构，key为秘钥，len为秘钥长度，md为计算hash的函数集合
HMAC\_Update((int)ctx, (int)data, 0); // 向HMAC上下文输入字节流，加密并输出，传入长度为0
// 经过 demo 验证，该条指令不会对上下文产生影响
len = 0;
HMAC\_Final((int)ctx, (int)hash1, (int)&len); // 生成最终的HMAC串，成功时len更新为HMAC的长度。
memset(hash2, 0, sizeof(hash2));
v9 = 1;
HMAC\_CTX\_init((int)ctx);
HMAC\_Init\_ex((int)ctx, (int)hash1, 32, md, 0);// 把 hash1 作为 key
HMAC\_Update((int)ctx, (int)data, 0);
HMAC\_Update((int)ctx, (int)&v9, 1); // 输入字节流 1,长度为1
len = 0;
HMAC\_Final((int)ctx, (int)hash2, (int)&len); // 得到第二次的 hash
memset(hash3, 0, sizeof(hash3));
v8 = 2;
HMAC\_CTX\_init((int)ctx);
HMAC\_Init\_ex((int)ctx, (int)hash1, 32, md, 0);// 把第一次的 hash 作为 key 再次加密
HMAC\_Update((int)ctx, (int)hash2, 32); // 输入字节流 hash2，长度为32
HMAC\_Update((int)ctx, (int)&v8, 1); // 输入字节流 2，长度为1
len = 0;
HMAC\_Final((int)ctx, (int)hash3, (int)&len); // 得到第三次的 hash
v4 = \*(\_QWORD \*)&hash2[8]; // v4 = hash2[8-15]
\*(\_QWORD \*)res = \*(\_QWORD \*)hash2; // res[0-7] = hash2[0-7]
\*((\_QWORD \*)res + 1) = v4; // res[8-15] = hash2[8-15]
\*((\_QWORD \*)res + 2) = \*(\_QWORD \*)&hash2[16]; // res[16-23] = hash2[16-23]
\*((\_QWORD \*)res + 3) = \*(\_QWORD \*)&hash2[24]; // res[24-31] = hash2[24-31]
\*((\_QWORD \*)res + 7) = \*(\_QWORD \*)&hash3[24]; // res[56-63] = hash3[24-31]
\*((\_QWORD \*)res + 6) = \*(\_QWORD \*)&hash3[16]; // res[48-55] = hash3[16-31]
v5 = \*(\_QWORD \*)hash3;
\*((\_QWORD \*)res + 5) = \*(\_QWORD \*)&hash3[8]; // res[40-47] = hash3[8-15]
\*((\_QWORD \*)res + 4) = v5; // res[32-39] = hash3[0-7]
return \_\_readgsdword(0x14u);
}
```

**测试demo**

```cpp
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

### **分析函数 `evp_encrypt`（部分二）**

这个函数的总的来说就是使用上一层传入的 `hash` 作为 `key`，加密用户名得到 `encrypt_data`，然后将 `encrypt_data`分段赋值给最终的 `res_encrypt_data`，分析这个函数的问题也还是在于作者使用了 `EVP_EncryptUpdate` 这个不熟悉的函数，并且还总是在参数中不该传0的传0，无法确定是否对上下文产生了影响，解决方法是花了一些时间在 VS 中搭了一下环境，测试了各种情况的结果。

```cpp
int \_\_cdecl evp\_encrypt(
int hash,
int iv,
int iv\_len,
BYTE \*const\_key,
int const\_key\_length,
int byte\_user\_name,
int user\_name\_length,
char \*res\_encrypt\_data)
{
int ctx; // esi
int cipher; // eax
size\_t v10; // eax
\_\_int64 v11; // xmm1\_8
int v13; // [esp+0h] [ebp-41Ch] BYREF
size\_t encrypt\_data\_len; // [esp+4h] [ebp-418h] BYREF
BYTE encrypt\_data[1024]; // [esp+8h] [ebp-414h] BYREF
unsigned int v16; // [esp+408h] [ebp-14h]
v16 = \_\_readgsdword(0x14u);
ctx = EVP\_CIPHER\_CTX\_new(); // 创建加密上下文
cipher = EVP\_aes\_256\_gcm(); // 选择一种加密算法
EVP\_EncryptInit\_ex(ctx, cipher, 0, 0, 0); // 初始化密码上下文ctx
EVP\_CIPHER\_CTX\_ctrl(ctx, 9, iv\_len, 0); // # define EVP\_CTRL\_AEAD\_SET\_IVLEN 0x9
// 设置向量IV的长度
EVP\_EncryptInit\_ex(ctx, 0, 0, hash, iv); // 使用hash作为key初始化ctx
EVP\_EncryptUpdate(ctx, 0, (int)&encrypt\_data\_len, (int)const\_key, const\_key\_length);// 加密 const\_key，让人困惑的是输出地址为0，
// 无法确定是否会对上下文产生影响，经过 demo 测试最终确定加密结果被丢弃
EVP\_EncryptUpdate(ctx, (int)encrypt\_data, (int)&encrypt\_data\_len, byte\_user\_name, user\_name\_length);// 加密用户名
memcpy(res\_encrypt\_data, encrypt\_data, encrypt\_data\_len);// dest[0-15] = encrypt\_data[0-15]
// 将第二次EncryptUpdate的结果拷贝到dest中，而且经过动态调试得知，encrypt\_data\_len=16
EVP\_EncryptFinal\_ex(ctx, (int)encrypt\_data, (int)&v13);// 块对齐
EVP\_CIPHER\_CTX\_ctrl(ctx, 16, 16, (int)encrypt\_data);// # define EVP\_CTRL\_AEAD\_GET\_TAG 0x10 获取标签
v10 = encrypt\_data\_len;
v11 = \*(\_QWORD \*)&encrypt\_data[8]; // v11 = encrypt\_data[8-15]
\*(\_QWORD \*)&res\_encrypt\_data[encrypt\_data\_len] = \*(\_QWORD \*)encrypt\_data;// dest[16-23] = encrypt\_data[0-7]
\*(\_QWORD \*)&res\_encrypt\_data[v10 + 8] = v11; // dest[24-31] = v11 = encrypt\_data[8-15]
EVP\_CIPHER\_CTX\_free(ctx); // 释放上下文
return encrypt\_data\_len + 16; // 返回加密结果的长度+16
}
```

**测试demo**
