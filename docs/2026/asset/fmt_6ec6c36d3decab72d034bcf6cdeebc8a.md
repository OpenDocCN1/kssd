
`_BYTE v21[256];` `// [esp+10h] [ebp-21Ch] BYREF`  `_BYTE v22[264];` `// [esp+110h] [ebp-11Ch] BYREF`  `unsigned int v23;` `// [esp+218h] [ebp-14h]`    `v23 = __readgsdword(0x14u);`  `memcpy`(dest, hash_temp, user_name_length_add_80);`//注意此时dest被赋值为hash_temp.最后一步的化简会用到`  `memset`(v22, 0, 256);`  `memset`(v21, 0, `sizeof`(v21));`  `for` `( i = 0; i != 256; ++i )`  `{`  `v21[i] = i;` `// v21 依次赋值 1 2 3 4 5 6 ...`  `v22[i] = v39[i % c_16];` `// v22 依次填充 0，因为v39全是0`  `}`  `v_dest = dest;`

### 第一次循环加密

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `v6 = dest;`  `v7 = 0;`  `v8 = -256;`  `do`  `{`  `v9 = (unsigned __int8)v22[v8];`  `v10 = v9 + v7 + (unsigned __int8)v22[v8 + 256];`  `v10 %= 256;`  `v11 = v21[v10];`  `v21[v10] = v9;`  `v22[v8++] = v11;`  `v7 = v10;`  `}`  `while` `( v8 );` |

这个循环的赋值有点混乱，利用两个相邻的数组越界赋值，所以这里我们先进行一个初步整理

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `unsigned char v21[256] = {0};`    `for` `(int i = 0; i < 256; i++)`  `v21[i] = i;`    `int v7 = 0;`  `int v8 = 0;`  `int v9 = 0;`  `int v10 = 0;`  `char v11 = 0;`    `for` `(int i = 0; i < 256; i++)`  `{`  `v9 = v21[i];`  `v10 = v9 + v7;`  `v10 %= 256;`  `v11 = v21[v10];`  `v21[v10] = v9;`  `v21[i] = v11;`  `v7 = v10;`  `}` |

这样看依然不是很清晰，所以进一步整理：

这时可以看出是 `v10` 每次加上`v21[i]`，避免`v10`作为下标时越界又与256取余，随后利用一个中间变量交换`v21[v10]`和`v21[i]`的值，实际上这一部分的结果是固定的。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `int v_tmp = 0;`  `for` `(int i = 0; i < 256; i++)`  `{`  `v10 += v21[i];`  `v10 %= 256;`  `v_tmp = v21[v10];`  `v21[v10] = v21[i];`  `v21[i] = v_tmp;`  `}` |

### 第二次循环加密

整个循环中不好分析的难点在于 `v15 = v12 + 1 - ((v12 + ((unsigned int)((v12 + 1) >> 31) >> 24) + 1) & 0xFFFFFF00);`

这里做下该语句的简要分析：

`(v12+1)>>31` 因为`v12`是有符号类型，也就是说该结果不是0就是0xffffffff, 关键在于`v12`的最高位判断

而 `v12` 的值来自于 `v15`，纵观整个循环 `v15` 又来自 `v12`，看似互相套娃但由于`v15`和`v12`的初值都为0

所以在循环量较小的时候 `v12` 的高位必然为0，所以 `((v12 + ((unsigned int)((v12 + 1) >> 31) >> 24) + 1) & 0xFFFFFF00) = 0`

那么看似复杂的第一行代码可以直接简化为 `v15 = v12 + 1`

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `v12 = 0;`  `v13 = 0;`  `v14 = user_name_length_add_80;`  `do`  `{`  `v15 = v12 + 1 - ((v12 + ((unsigned int)((v12 + 1) >> 31) >> 24) + 1) & 0xFFFFFF00);`  `v16 = (unsigned __int8)v21[v15];`  `v13 = (v16 + v13) % 256;`  `v17 = v14;`  `v18 = v21[v13];`  `v21[v13] = v16;`  `v21[v15] = v18;`  `*v_dest++ ^= v21[(unsigned __int8)(v21[v13] + v18)];`  `v14 = v17 - 1;`  `v19 = v17 == 1;`  `v12 = v15;`  `}` |

随后将代码进一步进行化简，执行流程就更为清晰

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `char tmp = 0;`  `do`  `{`  `//取出 v13 和 v15，v13 += v21[v15]，为避免越界v13作为下标越界，模256取余数`  `v15++;`  `v13 = (v21[v15] + v13) % 256;`    `// 交换 v21[13] 和 v21[15]`  `tmp = v21[v13];`  `v21[v13] = v21[v15];`  `v21[v15] = tmp;`    `// v21[13]+v21[15] 作为下标id，为避免越界，类型强转为unsigned char使其不超过255`  `// *hash_temp ^= v21[id]; hash_temp++;`  `*hash_temp++ ^= v21[(unsigned char)(v21[v13] + v21[v15])];`    `//v14是 user_name_length_add_80 = 0x60`  `v14--;`  `}` `while` `(v14!=0);` |

#### 函数 sub_46660 总结

- 生成一串随机数作为明文和key

- 调用函数 `hmac_ctx_hash` 得到随机明文的 hash

- 调用函数 `evp_encrypt` 以上一步的 hash 作为 key，得到 `user_name` 的密文

- 对用户名长度做了一些校验，跳过了一堆 SSE 指令（感谢手下留情）

- 通过 hash 第二次加密 `encrypt_data` 得到 `encrypt_data2`

- 调用函数 `encrypt_4` 第三次加密，得到最终的 dest

- dest 即为回传至屏幕的 password

# 按钮 check2

由于已经详细的分析了 generate2， 简单观察便可知check2只是其流程的逆运算，这里附上一些分析代码片段：

**代码片段1**

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `v21 = encrypt_4(pass_word, pass_word_len, (BYTE *)&v32, 16, v33);``//`  `// encrypt_4 是已经分析过的，但在解密时重新调用了一遍，这时再分析一遍该函数可以发现`  `// 核心算法是和一串固定的字节数组异或，这里第二次异或可以还原之前的值，作为解密`  `v23 = v19;`  `*(_QWORD *)v34 = *(_QWORD *)v33;` `// 这里可以看到将 encrypt_4 的结果 v33 赋值给 v34，显然这就是对应加密时的逆运算`  `*(_QWORD *)&v34[8] = *(_QWORD *)&v33[8];`  `*(_QWORD *)&v34[16] = *(_QWORD *)&v33[16];`  `*(_QWORD *)&v34[24] = *(_QWORD *)&v33[24];`  `*(_QWORD *)&v34[32] = *(_QWORD *)&v33[32];`  `*(_QWORD *)&v34[40] = *(_QWORD *)&v33[40];` |

**代码片段2**

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `do`  `{`  `v15 = v7 ^ v24[((_BYTE)v7 - 1) & 0xF];``// 这里也是，重新审视一下这个循环算法，就是异或，这里第二次调用作为解密算法`  `v5[v7 - 1] = __ROL1__(v15 ^ v33[v7 + 63], 4) - v15;`  `--v7;`  `}`  `while` `( v7 > 0 );` |

**代码片段3**

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `v21 = v6 - 80;`  `v16 = Decrypt((int)v34, (int)&v34[32], 12, (int)v31, 27, (int)&v5[v20 - 16], 16, (int)v5, v6 - 80, v24);`  `// 解密数据，解密后的数据放在v24中`  `v4 = 0;`  `if` `( v16 && v21 == user_name_len && !memcmp(v24, user_name, user_name_len) )``// 校验用户名和解密后的数据，一致返回1`  `return` `1;` |

**代码片段4**

```
int __cdecl Decrypt(int key, int a2, int a3, int a4, int a5, int a6, int a7, int a8, int a9, void *dest)
{
int ctx;
// esi
int v11;
// eax
int v12;
// edi
size_t v14;
// [esp+4h] [ebp-418h] BYREF
char src[1024];
// [esp+8h] [ebp-414h] BYREF
unsigned int v16;
// [esp+408h] [ebp-14h]

v16 = __readgsdword(0x14u);
ctx = EVP_CIPHER_CTX_new();
v11 = EVP_aes_256_gcm();
EVP_DecryptInit_ex(ctx, v11, 0, 0, 0);
// 大致浏览一下参数与和之前的基本一致，只是调用的是解密函数
EVP_CIPHER_CTX_ctrl(ctx, 9, a3, 0);
EVP_DecryptInit_ex(ctx, 0, 0, key, a2);
if ( a5 )
EVP_DecryptUpdate(ctx, 0, &v14, a4, a5);
EVP_DecryptUpdate(ctx, src, &v14, a8, a9);
memcpy(dest, src, v14);
// 这里也是一样，解密后的数据放到最后一个参数地址中
EVP_CIPHER_CTX_ctrl(ctx, 17, a7, a6);
v12 = EVP_DecryptFinal_ex(ctx, src, &v14);
EVP_CIPHER_CTX_free(ctx);
return v12;
}
```

**check2 按钮总结**

分析完 check2 按钮后，发现相比较 generate2 缺少一个流程，那就是求用户名 hash 作为key，于是回溯了一下这个 key，通过追溯发现，这个 key 已经通过两个异或加密函数加密在了 password 中，只需要将其逆运算取出来即可使用。

# 初始化函数 _init

在分析按钮 generate3 时很多数据静态分析时并不存在，所以这里还需要分析 `_init` 函数，该函数地址并没有被 IDA 分析出来，可以通过以下两个办法找到：

- 使用 readelf 工具查看

```
.\readelf.exe -d C:\Users\Administrator\Desktop\libnative-lib.so

```

- 使用 IDA 动态调试，在被写入数据的地址下内存写入断点

### 分析函数 sub_41BA0

这个函数就是 `_init` 函数，其主要分为三个函数，分别是 set_buff1、 set_buff2、 set_buff3

#### 分析函数 set_buff1

这个函数首先将 19ca9c + image_base 存到 buff1_60[0-3] 中，根据后面的分析结果这里存的是一个函数地址，之后在 buff1_60[28-43] 中填入了一些随机数，随后调用了 sub_48840，并传入了一个字符串

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `BYTE *__cdecl set_buff1(BYTE *buff1)`  `{`  `*(_DWORD *)buff1 = &off_19CA9C;` `// buff1_60[0-3] = 19ca9c + image_base`  `*((_DWORD *)buff1 + 5) = 0;` `// buff1_60[16-27] = 0`  `*((_DWORD *)buff1 + 4) = 0;`  `*((_DWORD *)buff1 + 6) = 0;`  `*((_DWORD *)buff1 + 12) = 0;` `// buff1_60[44-59] = 0`  `*((_DWORD *)buff1 + 11) = 0;`  `*((_DWORD *)buff1 + 14) = 0;`  `*((_DWORD *)buff1 + 13) = 0;`  `v1 = lrand48();` `// buff1_60[28-43] = rand()`  `buff1[28] = v1 + v1 / 255;`  `v2 = lrand48();`  `......`  `buff1[42] = v15 + v15 / 255;`  `v16 = lrand48();`  `buff1[43] = v16 + v16 / -16777216;`  `return` `sub_48840(buff1 + 16,` `"3390fd362dfdda0030d5737632d3d213"``, 32u);``//`  `// buff1[0-4] 19ca9c + image_base`  `// buff1[16-19] 存放着申请堆空间大小+1，也就是49`  `// buff1[20-23] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的长度，也就是32`  `// buff1[24-27] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的指针`  `// buff1[28-43] 存放着随机数`  `}` |

##### 分析函数 sub_48840

该函数主要是一些范围的判断，主要还是调用函数 sub_488E0，并传入了一个字符串

|  |  |

| --- | --- |
