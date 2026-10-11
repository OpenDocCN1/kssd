
# flexlm v9.2 的 `l_n36_buf` 及 `l_n36_buff` 说明

> 原文：[`www.pediy.com/kssd/pediy10/91334.html`](https://www.pediy.com/kssd/pediy10/91334.html)

在 flexlm v9.2 中，程序首先调用 `lc_new_job--l_n36_buf--lc_init`，`l_n36_buf` 实际上是获取 `vendorid` 以及 `SEED0 SEED1 KEY0--KEY3` 以及其他的初始化数据。当然 `vendorid` 是明码，`seed` 和 `key0--key3` 是加过密的。将 `code` 放入 `l_xorname` 执行后 `key0--key3` 就恢复成原始状态，然后在 `l_sg` 的函数里边把 `code` 放入 `l_n36_buff` 去执行。

`l_n36_buff` 使用 `_time` 函数生成 `job->mem_ptr2_bytes[12]` 数据，`l_n36_buff` 恢复 `key5_order[0]--key5_order[3]`、`key5_uniqx`、`sig[0]--sig[3]`，然后对 `l_n36_buf` 提取出的 `SEED0`、`SEED1` 进行 `xor` 运算。所以为什么在 `l_string_key` 程序中看到的 `seed0` 和 `seed1` 都是变化的，原因就在于 `mem_ptr2_bytes[12]` 是用 `_time` 函数生成的。如果用 `idapro` 手工恢复 `seed`，只需要把 `mem_ptr2_bytes[12]` 全部设为 0，或者直接用 `lmnewgen.c` 中的 `key5()` 程序恢复就可以了。我相信 `CrackZ` 没有完全看这 2 个函数，所以他发表的文章对这 2 个函数没有详细的说明。

`l_getattr.c`

```c
void l_xorname(name, vc)
char *name;  //一般是 vendorid
VENDORCODE *vc;
{
#define VENDORMAGIC_V7 0x08BC0EF8
  int i;
  char buf[MAX_VENDOR_NAME + 1];
  /* make sure vendor name is all zeros after the first zero */
  memset(buf, 0, sizeof(buf));
  strcpy(buf, name);
  for (i = 0;i < 4; i++)
    vc->keys[i] &= 0xffffffff; /* 64-bit fix */
  vc->keys[0] ^= buf[0] ^
    (buf[1] << 8) ^
    (buf[2] << 16) ^
    (buf[3] << 24)  ^ VENDORMAGIC_V7;
  vc->keys[1] ^= buf[2] ^
    (buf[5] << 8) ^
    (buf[7] << 16) ^
    (buf[4] << 24)  ^ VENDORMAGIC_V7;
  vc->keys[2] ^= buf[4] ^
    (buf[6] << 8) ^
    (buf[1] << 16) ^
    (buf[6] << 24)  ^ VENDORMAGIC_V7;
  vc->keys[3] ^= buf[5] ^
    (buf[0] << 8) ^
    (buf[2] << 16) ^
    (buf[3] << 24)  ^ VENDORMAGIC_V7;
}

#define MAX_DAEMON_NAME 10    /* Max length of DAEMON string */
#define MAX_VENDOR_NAME MAX_DAEMON_NAME  /* Synomym for MAX_DAEMON_NAME */
```

`lmnewgen.c`

```c
static
void key5(k)
VENDORCODE *k;
{
  unsigned long *keys;
  unsigned long signature;
#define SIGSIZE 4
  char sig[SIGSIZE];
  int i = SIGSIZE-1;
  int len = strlen(vendor_name);
//len 直接把 vedorname 的长度写进去就好，比如 vendor=KHJZXe,那么 len=6
  sig[0] = sig[1] = sig[2] = sig[3] = 0;
  for (i = 0; i < 10; i++)
  {
    if (sig[i%SIGSIZE] != vname[i%len])
      sig[i%SIGSIZE] ^= vname[i % len];
  }
  k->data[0] ^=
    (((((long)sig[0] << key5_order[0])|
        ((long)sig[1] << key5_order[1]) |
        ((long)sig[2] << key5_order[2]) |
        ((long)sig[3] << key5_order[3]))
    ^ key5_uniqx
    ^ k->keys[1]
    ^ k->keys[0]) & 0xffffffff);
  k->data[1] ^=
    (((((long)sig[0] << key5_order[0])|
        ((long)sig[1] << key5_order[1]) |
        ((long)sig[2] << key5_order[2]) |
        ((long)sig[3] << key5_order[3]))
    ^ key5_uniqx
    ^ k->keys[1]
    ^ k->keys[0]) & 0xffffffff);
}
```

我说的是为什么每回调用 `l_sg()` 函数，`seed` 都会变化的根本原理。
