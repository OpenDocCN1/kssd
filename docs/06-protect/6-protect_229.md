
# 去 FLEXLM/NET ECC 标志，强行 12 位 SIGN 验证

> 原文：[`www.pediy.com/kssd/pediy10/82230.html`](https://www.pediy.com/kssd/pediy10/82230.html)

```c
#define LM_PUBKEYS    3
#define LM_MAXPUBKEYSIZ    40
#define LM_MAXSIGNS    4  /* SIGN=, SIGN2=, SIGN3=, SIGN4= */
#define LM_MAX_BEH_VER    4

typedef struct _pubkeyinfo
  {
    int pubkeysize[LM_PUBKEYS];
    unsigned char pubkey[LM_PUBKEYS][LM_MAXPUBKEYSIZ];
    int (*pubkey_fptr)();
    int strength;
    int sign_level;
  } LM_VENDORCODE_PUBKEYINFO;

typedef struct _vendorcode
  {
    short type;
    unsigned long data[2];
    unsigned long keys[4];
    short flexlm_version;
    short flexlm_revision;
    char flexlm_patch[2];
    char behavior_ver[LM_MAX_BEH_VER + 1];
    unsigned long trlkeys[2];
    int signs;
    int strength;
    int sign_level;
    LM_VENDORCODE_PUBKEYINFO pubkeyinfo[LM_MAXSIGNS];
  } VENDORCODE;

....................

static unsigned int l_2078index = 16; //0x10, 静态存储

....................

Static int l_n36_buf
  (
    char *buf;
    VENDORCODE *v;
    unsigned int l_buf_13;
    unsigned char *l_ctr_17;
    unsigned int l_8indexes;
    unsigned int *l_18index;
    char *buf2
  );
{
   ......................
  //第一处 Flag 的位置
  if (l_18index) *l_18index = 1;
   .............................
  //第二处 Flag 的位置
  if (l_4counters == 0) v->pubkeyinfo[0].pubkeysize[0] += (l_2078index << 0);
   ................................
}
```

对应如下近似代码(每个程序有差别)：

第一处 Flag 的位置

```asm
cmp     [ebp+arg_14], 0
jz      short loc_xxxx
mov     eax, [ebp+arg_14]
mov     dword ptr [eax], 1 //有的程序为 2
```

第二处 Flag 的位置

```asm
cmp     ds:?l_4counters@?1??l_counters_1@@9@9, 0 ; `l_counters_1'::`2'::l_4counters
jnz     short loc_xxxx
mov     eax, [ebp+Dst]
mov     ecx, [eax+3Ch]
add     ecx, _l_2078index //找到静态存储位置：0x10-->0x00
mov     edx, [ebp+Dst]
mov     [edx+3Ch], ecx
```
