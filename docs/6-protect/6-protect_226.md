

# 手工快速识别+破解 FLEXLM\_ECC 关键函数

> 原文：[`www.pediy.com/kssd/pediy10/81843.html`](https://www.pediy.com/kssd/pediy10/81843.html)

1、  爆破 l\_pubkey\_verify  
（1） 在 IDA 中查找字符串 “lmseed.dat” 和 “Can't open lmseed.dat”的引用，对应的源代码如，再往上很容易确定出函数 sb\_end。  

代码:

```
  ret = sb\_end(global\_data);
  if (ret != SB\_SUCCESS) 
  {
    fprintf(stderr, "Error 9: ");
    goto exit\_seed;
  }
/\*此处是关键的识别点，往上可找到 sb\_end\*/
  if (!(fp = fopen("lmseed.dat", "w")))
    fprintf(stderr, "Can't open lmseed.dat");
  else
  {
    fprintf(fp, 
  "\\
Once set, the values for LM\_SEED1-3 must be kept secret \\n\\
and \*never\* change.\\n\\n\\
#define LM\_SEED1 0x%08x\\n\\
#define LM\_SEED2 0x%08x\\n\\
#define LM\_SEED3 0x%08x\\n", seed1, seed2, seed3);
    fclose(fp);
  }

```

（2）查找对 sb\_end 的调用，其中有一个应为 mov \[addr\], sb\_end，对面应的源代码如下，此处所在的函数即为 l\_pubkey\_verify  
代码:

```

if (!job->L\_VERIFY\_MEM)
  {
    typedef void (\*cleanup)(char \*);
    m = (VERIFY\_MEM \*)(job->L\_VERIFY\_MEM = 
      (char \*)l\_malloc(job, sizeof(\*m)));
    m->cleanup = (cleanup)sb\_end;
    m->strength = strength;
    m->sign\_level = sign\_level;

```

（3）在找到的 mov \[addr\], sb\_end 继续往上，找到选择分支结构，对应的源代码如下，此处的爆破方法很多，可以直接将 strength 置为 0  
代码:

```

  switch(strength)
  {
  case LM\_STRENGTH\_LICENSE\_KEY: return 0;
  case LM\_STRENGTH\_113BIT: ellipticCurve = 
                &LM\_PUBKEY\_CURVE113BIT; break;
  case LM\_STRENGTH\_163BIT: ellipticCurve = 
&LM\_PUBKEY\_CURVE163BIT; break;
  case LM\_STRENGTH\_239BIT: ellipticCurve = 
&LM\_PUBKEY\_CURVE239BIT; break;
  default:
    {
      LM\_SET\_ERRNO(job, LM\_BADPARAM, 531, 0);
      ret = LM\_BADPARAM;
      goto exit\_verify;
    }
  }

```

2、  在 daemon 中寻找 seed 明文  
（1）寻找 289BEB8A 的引用，应是赋值语句 mov \[addr\], 289BEB8A，对应的源代码如下  
代码:

```

  if ((keylen < 12) || (keylen > MAX\_CRYPT\_LEN)) // MAX\_CRYPT\_LEN=20
    return 0;
  if (keylen != 12)
        len = L\_SECLEN\_LONG; // L\_SECLEN\_LONG=0x289BEB8A

```

再往前应有 “ok” “demo” “%02X”。  
注意在 289BEB8A 的引用附近没有 66D8B337，根据这一条可以直接确定出关键函数 l\_string\_key。  
（2）往下找 3D4DA1D6，找到 mov \[var\_xx\], 3D4DA1D6，再沿此往上找到类似 mov \[var\_xx\]，eax。 进一步往前是 call sub\_xxxx，这里的 sub\_xxxx 就是产生 SEED 明文的关键函数，返回值 eax 中就是 SEED 的明文。  
（3）在 l\_string\_key 中找到 sub\_xxxx 的两个调用，在函数调用后下断点即可。 

1、  爆破 l\_pubkey\_verify  
（1） 在 IDA 中查找字符串 “lmseed.dat” 和 “Can't open lmseed.dat”的引用，对应的源代码如，再往上很容易确定出函数 sb\_end。  

代码:

```
  ret = sb\_end(global\_data);
  if (ret != SB\_SUCCESS) 
  {
    fprintf(stderr, "Error 9: ");
    goto exit\_seed;
  }
/\*此处是关键的识别点，往上可找到 sb\_end\*/
  if (!(fp = fopen("lmseed.dat", "w")))
    fprintf(stderr, "Can't open lmseed.dat");
  else
  {
    fprintf(fp, 
  "\\
Once set, the values for LM\_SEED1-3 must be kept secret \\n\\
and \*never\* change.\\n\\n\\
#define LM\_SEED1 0x%08x\\n\\
#define LM\_SEED2 0x%08x\\n\\
#define LM\_SEED3 0x%08x\\n", seed1, seed2, seed3);
    fclose(fp);
  }

```

（2）查找对 sb\_end 的调用，其中有一个应为 mov \[addr\], sb\_end，对面应的源代码如下，此处所在的函数即为 l\_pubkey\_verify  
代码:

```

if (!job->L\_VERIFY\_MEM)
  {
    typedef void (\*cleanup)(char \*);
    m = (VERIFY\_MEM \*)(job->L\_VERIFY\_MEM = 
      (char \*)l\_malloc(job, sizeof(\*m)));
    m->cleanup = (cleanup)sb\_end;
    m->strength = strength;
    m->sign\_level = sign\_level;

```

（3）在找到的 mov \[addr\], sb\_end 继续往上，找到选择分支结构，对应的源代码如下，此处的爆破方法很多，可以直接将 strength 置为 0  
代码:

```

  switch(strength)
  {
  case LM\_STRENGTH\_LICENSE\_KEY: return 0;
  case LM\_STRENGTH\_113BIT: ellipticCurve = 
                &LM\_PUBKEY\_CURVE113BIT; break;
  case LM\_STRENGTH\_163BIT: ellipticCurve = 
&LM\_PUBKEY\_CURVE163BIT; break;
  case LM\_STRENGTH\_239BIT: ellipticCurve = 
&LM\_PUBKEY\_CURVE239BIT; break;
  default:
    {
      LM\_SET\_ERRNO(job, LM\_BADPARAM, 531, 0);
      ret = LM\_BADPARAM;
      goto exit\_verify;
    }
  }

```

2、  在 daemon 中寻找 seed 明文  
（1）寻找 289BEB8A 的引用，应是赋值语句 mov \[addr\], 289BEB8A，对应的源代码如下  
代码:

```

  if ((keylen < 12) || (keylen > MAX\_CRYPT\_LEN)) // MAX\_CRYPT\_LEN=20
    return 0;
  if (keylen != 12)
        len = L\_SECLEN\_LONG; // L\_SECLEN\_LONG=0x289BEB8A

```

再往前应有 “ok” “demo” “%02X”。  
注意在 289BEB8A 的引用附近没有 66D8B337，根据这一条可以直接确定出关键函数 l\_string\_key。  
（2）往下找 3D4DA1D6，找到 mov \[var\_xx\], 3D4DA1D6，再沿此往上找到类似 mov \[var\_xx\]，eax。 进一步往前是 call sub\_xxxx，这里的 sub\_xxxx 就是产生 SEED 明文的关键函数，返回值 eax 中就是 SEED 的明文。  
（3）在 l\_string\_key 中找到 sub\_xxxx 的两个调用，在函数调用后下断点即可。