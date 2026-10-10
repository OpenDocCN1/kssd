

# 去 FLEXLM/NET ECC 标志，强行 12 位 SIGN 验证

> 原文：[`www.pediy.com/kssd/pediy10/82230.html`](https://www.pediy.com/kssd/pediy10/82230.html)

#define LM\_PUBKEYS    3  
#define LM\_MAXPUBKEYSIZ    40  
#define LM\_MAXSIGNS    4   /\* SIGN=, SIGN2=, SIGN3=, SIGN4= \*/  
#define LM\_MAX\_BEH\_VER    4   
  
typedef struct \_pubkeyinfo   
  {  
    int pubkeysize\[LM\_PUBKEYS\];  
    unsigned char pubkey\[LM\_PUBKEYS\]\[LM\_MAXPUBKEYSIZ\];  
    int (\*pubkey\_fptr)();  
    int strength;  
    int sign\_level;  
  } LM\_VENDORCODE\_PUBKEYINFO;  
  
typedef struct \_vendorcode   
  {  
    short type;       
    unsigned long data\[2\];  
    unsigned long keys\[4\];  
    short flexlm\_version;  
    short flexlm\_revision;  
    char flexlm\_patch\[2\];  
    char behavior\_ver\[LM\_MAX\_BEH\_VER + 1\];  
    unsigned long trlkeys\[2\];  
    int signs;  
    int strength;  
    int sign\_level;  
    LM\_VENDORCODE\_PUBKEYINFO pubkeyinfo\[LM\_MAXSIGNS\];  
  } VENDORCODE;  
  
....................  
  
static unsigned int l\_2078index = 16; //0x10, 静态存储  
  
....................  
  
Static int l\_n36\_buf  
  (  
    char \*buf;  
    VENDORCODE \*v;  
    unsigned int l\_buf\_13;  
    unsigned char \*l\_ctr\_17;  
    unsigned int l\_8indexes;  
    unsigned int \*l\_18index;  
    char \*buf2  
  );  
{  
   ......................  
  
  //第一处 Flag 的位置  
  if (l\_18index) \*l\_18index = 1;   
  
   .............................  
  
  //第二处 Flag 的位置  
  if (l\_4counters == 0) v->pubkeyinfo\[0\].pubkeysize\[0\] += (l\_2078index << 0);   
  
   ................................  
  
}  
  
对应如下近似代码(每个程序有差别)：  
  
第一处 Flag 的位置  
cmp     \[ebp+arg\_14\], 0  
jz      short loc\_xxxx  
mov     eax, \[ebp+arg\_14\]  
**mov     dword ptr \[eax\], 1** //有的程序为 2  
  
第二处 Flag 的位置  
cmp     ds:?l\_4counters@?1??l\_counters\_1@@9@9, 0 ; \`l\_counters\_1'::\`2'::l\_4counters  
jnz     short loc\_xxxx  
mov     eax, \[ebp+Dst\]  
mov     ecx, \[eax+3Ch\]  
**add     ecx, \_l\_2078index //找到静态存储位置：0x10-->0x00**  
mov     edx, \[ebp+Dst\]  
mov     \[edx+3Ch\], ecx