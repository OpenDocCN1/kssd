
可以轻易发现，主要是调用了 `MainActivity.this.getck` 方法，通过其返回值判断成功失败

### 代码示例

```
class c implements View.OnClickListener {
  c() {
  }
  @Override // android.view.View.OnClickListener
  public void onClick(View view) {
    Context applicationContext;
    String str;
    String obj = MainActivity.this.y.getText().toString();
    String obj2 = MainActivity.this.z.getText().toString();
    if (obj.length() != 16) {
      Toast.makeText(MainActivity.this.getApplicationContext(), "用户名长度必须为16字节", 0).show();
      return;
    }
    if (MainActivity.this.getck(obj.getBytes(), com.test.pac.demo.a.a.b(obj2)) > 0) {
      applicationContext = MainActivity.this.getApplicationContext();
      str = "verify success!";
    } else {
      applicationContext = MainActivity.this.getApplicationContext();
      str = "verify failed!";
    }
    Toast.makeText(applicationContext, str, 0).show();
  }
}
```

### 分析函数 `MainActivity_getck`

可以发现主要就是在调用函数 `dword_1A50BC[0] + 4` 而这个地址其实就是在 `_init` 初始化的 `[19ca8c + image_base]+4`（函数 `sub_43280`）

```
int __cdecl Java_com_test_pac_demo_MainActivity_getck(JNIEnv *a1, int a2, void *username_, void *password_)
{
  username_len = (*a1)->GetArrayLength(a1, username_);
  username = (*a1)->GetByteArrayElements(a1, username_, 0);
  password_len = (*a1)->GetArrayLength(a1, password_);
  password = (*a1)->GetByteArrayElements(a1, password_, 0);
  v8 = (*(int (__cdecl **)(int, jbyte *, jsize, jbyte *, jsize))(*(_DWORD *)dword_1A50BC[0] + 4))(
    dword_1A50BC[0],
    username,
    username_len,
    password,
    password_len);
  (*a1)->ReleaseByteArrayElements(a1, username_, username, 0);
  (*a1)->ReleaseByteArrayElements(a1, password_, password, 0);
  return v8;
}
```

### 分析函数 `sub_43280`

这个函数是解密的核心函数，这里总结一下流程：

- 依次按照加密的流程逆向运算对 `password` 进行解密
- 调用了一个加密过程中未出现过的函数 `sub_43860` 加密 `password`
- 调用函数 `sub_424E0` 解密 `password`
- 函数 `sub_424E0` 解密失败，返回 `0`

```
int __cdecl sub_43280(BYTE *fun3, BYTE *username, int username_len, char *password, int password_len)
{
  result = 0;
  if ( username_len == 16 )
  {
    memset(password_encrypt, 0, 256);
    memset(v71, 0, 256);
    ......
    v15 = v13 + 1 - ((v13 + ((unsigned int)((v13 + 1) >> 31) >> 24) + 1) & 0xFFFFFF00);
    v16 = (unsigned __int8)v71[v15];
    v14 = (v16 + v14) % 256;
    v17 = v71[v14];
    v71[v14] = v16;
    v71[v15] = v17;
    password[v12++] ^= v71[(unsigned __int8)(v71[v14] + v17)];
    // 上面的代码在 sub_42940 是分析过的
    // 注意和 password 进行异或的值都是通过
    // fun3 初始化 v71、72在通过相同的位运算计算出来的
    // 也就是说这里就是在对 password 进行解密
    v13 = v15;
  }
  while ( password_len != v12 );
  if ( password_len >= 32 )
  {
    v19 = password;
    sub_43990((int)fun3, (int)(password + 2), password_len - 18, (unsigned int *)&password[password_len - 16]);
    // 这里的第二次调用 sub_43990 也是同样的道理
    // 也是异或相同的值对 password 进行解密
    v20 = (unsigned __int8)password[3];
    v21 = 16 * (unsigned __int8)password[2];
    v22 = v20 + v21 == 0;
    v23 = v20 + v21;
    v18 = 0;
    if ( !v22 )
    {
      v24 = v20 + 16 * (unsigned __int8)password[2];
      v25 = 0;
      if ( v24 < 0x20 )
        goto LABEL_17;
      // 动态调试得知此处跳转至 LABEL_17
      v25 = v24 & 0xFFFFFFE0;
      ......
      if ( v24 != v25 )
      {
        LABEL_17:
        v29 = v24 - v25;
        v30 = &v19[v25 + 4];
        do
        {
          *(v30 - 4) = *v30;
          // v30 就是 password，在函数 sub_42940 中也是
          // 存在这段代码的，这里还是对password进行解密
          ++v30;
          --v29;
        }
        while ( v29 );
      }
      v18 = v23;
      // v20 = (unsigned __int8)password[3];
      // v21 = 16 * (unsigned __int8)password[2];
      // v22 = v20 + v21 == 0;
      // v23 = v20 + v21;
      //
      // v18 = password[2]*16 + password[3]
    }
  }
  else
  {
    v18 = 0;
  }
  v31 = v18;
  // v31 = password[2]*16 + password[3]
  //
  // 回去翻看了一下 sub_43070 的分析过程，发现这个值是固定的 = 20
  //   src[2] = BYTE1(num_20);                       // src[2] = n的高位 = 0
  //   src[3] = num_20;                              // src[3] = 20
  memset(password_encrypt, 0, sizeof(password_encrypt));
  v32 = sub_43860(v47, password, v31, password_encrypt, COERCE_FLOAT(120));
  // 这里出现了一个在加密过程中没有出现过的函数
  // 这个函数将password加密的数据存放到 password_encrypt 中
  // 参数 v31 应该是欲加密的长度为20，返回值 v32 = v31 = 20
  result = 0;
  if ( v32 == 20 )
  {
    if ( username )
    {
      v33 = _mm_loadu_si128((const __m128i *)username);
      si128 = _mm_load_si128((const __m128i *)&xmmword_148380);
      ......
      *(__m128i *)&key[16] = _mm_unpackhi_epi8(v39, v42);
      *(__m128i *)key = _mm_unpacklo_epi8(v39, v42);
      // 这段循环之前在 sub_42940 也是分析过的
      // 显然这就是把用户名变换一下，然后当做下面求 hash 的key使用
      // key[0-15] = (username 8bit分组右移四位) ^ si128 再高64位分组交错 username
      // key[16-31] = (username 8bit分组右移四位) ^ si128 再低64位分组交错 username
    }
    HMAC_CTX_init(ctx);
    v43 = EVP_sha256();
    HMAC_Init_ex(ctx, key, 32, v43, 0);
    HMAC_Update((int)ctx, (int)&password_encrypt[2], 2);
    // 将 password_encrypt[2-3] 传入
    v49 = 0;
    HMAC_Final((int)ctx, (int)hash, (int)&v49);
    // 得到 hash
    v44 = -32;
    v45 = 0;
    do
      v45 = *(_WORD *)&fun3[2 * (key[v44++ + 32] ^ HIBYTE(v45)) + 84] ^ (v45 << 8);
    while ( v44 );
    // 这个算法之前也分析过，把 key[i]*2 * v45 当做id
    // 求得fun3数据区的hash
    v48 = _mm_shuffle_epi8(_mm_loadu_si128((const __m128i *)username), (__m128i)xmmword_1483E0);
    //
    // xmmword_1483E0 = 102030405060708090A0B0C0D0E0Fh
    // 对 username 的顺序进行变换，传入下边的 sub_424e0
    // 不过这个参数并不影响结果
    memset(v71, 0, sizeof(v71));
    result = sub_424E0(
      // EVP_DecryptFinal_ex 失败，result = 0
      v46,(int)hash,(int)username,12,(int)&v48,16,(int)&password_encrypt[4],
      16,(int)password_encrypt,2,v71);
    if ( !result || v45 != *(_WORD *)v71 )
      return 0;
    // 返回 0
  }
  return result;
}
```

#### 分析函数 `sub_43860`

没有找到这个函数对应的加密流程，应该是导致解密失败的主要原因

```
int __cdecl sub_43860(int a1, char *password, int password_len, void *dest, float a5)
{
  v5 = &v13[-((4 * password_len + 15) & 0xFFFFFFF0)];
  memcpy(v5, password, password_len);
  // v5 = password
  if ( password_len >= 4 )
  {
    *(_DWORD *)&v13[12] = -LODWORD(a5);
    // v13 中记录了两个值， v13[16] 中记录的是长度除以4
    v6 = 0;
    *(_DWORD *)&v13[16] = password_len / 4;
    if ( (unsigned int)(password_len / 4) <= 7 )
      goto LABEL_6;
    // 动态调试此处跳转 LABEL_6
    ......
    while ( *(_DWORD *)&v13[16] != v6 )
    {
      LABEL_6:
      *(_DWORD *)&v5[4 * v6] = (*(_DWORD *)&password[4 * v6] << v13[12]) | (*(_DWORD *)&password[4 * v6] >> SLOBYTE(a5));
      ++v6;
      // 通过这个公式循环修改 v5 中的数据，循环次数为长度的四分之一
    }
  }
  memcpy(dest, v5, password_len);
  // 将修改后的数据拷贝到dest
  return password_len;
}
```

#### 分析函数 `sub_424E0`

`EVP_DecryptFinal_ex` 解密失败，这里我查阅了一下 openssl 源码，结合动态调试返回的地方，对应在 `EVP_R_INVALID_OPERATION` 处返回，正好源码里有注释，翻译了一下是：防止解密时意外使用加密上下文。 应该是解密的数据不对导致的失败返回 0。
