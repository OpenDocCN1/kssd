
# 手工快速识别+破解 FLEXLM_ECC 关键函数

> 原文：[`www.pediy.com/kssd/pediy10/81843.html`](https://www.pediy.com/kssd/pediy10/81843.html)

## 1、 爆破 l_pubkey_verify

### （1） 在 IDA 中查找字符串 “lmseed.dat” 和 “Can't open lmseed.dat”的引用
对应的源代码如下，再往上很容易确定出函数 `sb_end`。

代码:

```
  ret = sb_end(global_data);
  if (ret != SB_SUCCESS) 
  {
    fprintf(stderr, "Error 9: ");
    goto exit_seed;
  }
/*此处是关键的识别点，往上可找到 sb_end*/
  if (!(fp = fopen("lmseed.dat", "w")))
    fprintf(stderr, "Can't open lmseed.dat");
  else
  {
    fprintf(fp, 
  "\\
Once set, the values for LM_SEED1-3 must be kept secret \\n\\
and *never* change.\\n\\n\\
#define LM_SEED1 0x%08x\\n\\
#define LM_SEED2 0x%08x\\n\\
#define LM_SEED3 0x%08x\\n", seed1, seed2, seed3);
    fclose(fp);
  }
```

### （2） 查找对 sb_end 的调用
其中有一个应为 `mov [addr], sb_end`，对应的源代码如下，此处所在的函数即为 `l_pubkey_verify`。

代码:

```
if (!job->L_VERIFY_MEM)
  {
    typedef void (*cleanup)(char *);
    m = (VERIFY_MEM *)(job->L_VERIFY_MEM = 
      (char *)l_malloc(job, sizeof(*m)));
    m->cleanup = (cleanup)sb_end;
    m->strength = strength;
    m->sign_level = sign_level;
```

### （3） 在找到的 mov [addr], sb_end 继续往上
找到选择分支结构，对应的源代码如下，此处的爆破方法很多，可以直接将 `strength` 置为 0。

代码:

```
  switch(strength)
  {
  case LM_STRENGTH_LICENSE_KEY: return 0;
  case LM_STRENGTH_113BIT: ellipticCurve = 
                  &LM_PUBKEY_CURVE113BIT; break;
  case LM_STRENGTH_163BIT: ellipticCurve = 
&LM_PUBKEY_CURVE163BIT; break;
  case LM_STRENGTH_239BIT: ellipticCurve = 
&LM_PUBKEY_CURVE239BIT; break;
  default:
    {
      LM_SET_ERRNO(job, LM_BADPARAM, 531, 0);
      ret = LM_BADPARAM;
      goto exit_verify;
    }
  }
```

## 2、 在 daemon 中寻找 seed 明文

### （1） 寻找 289BEB8A 的引用
应是赋值语句 `mov [addr], 289BEB8A`，对应的源代码如下。

代码:

```
  if ((keylen < 12) || (keylen > MAX_CRYPT_LEN)) // MAX_CRYPT_LEN=20
    return 0;
  if (keylen != 12)
        len = L_SECLEN_LONG; // L_SECLEN_LONG=0x289BEB8A
```

再往前应有 “ok” “demo” “%02X”。

注意在 `289BEB8A` 的引用附近没有 `66D8B337`，根据这一条可以直接确定出关键函数 `l_string_key`。

### （2） 往下找 3D4DA1D6
找到 `mov [var_xx], 3D4DA1D6`，再沿此往上找到类似 `mov [var_xx], eax`。进一步往前是 `call sub_xxxx`，这里的 `sub_xxxx` 就是产生 SEED 明文的关键函数，返回值 `eax` 中就是 SEED 的明文。

### （3） 在 l_string_key 中找到 sub_xxxx 的两个调用
在函数调用后下断点即可。
