
# Study on ECC patch by replacing pubkeys in flexlm

> 原文：[`www.pediy.com/kssd/pediy10/73239.html`](https://www.pediy.com/kssd/pediy10/73239.html)

首先要说明的一点是，本文只是为了研究用替换公钥的方法来探讨去除 ECC 的可能性，因为去除 ECC 有更简单的方法，通常 `PATCH` 几个字节就可以了，具体在论坛上已经有很多人讨论过了。

ECC 破解的最困难之处在于，在没有找到 ECC 加密算法的漏洞之前，必须找到私钥才能产生正确的 `LIC`，因为采用 ECC 时，根本不存在 `signature` 的比较过程，也就是说你无法知道他正确的 `signature` 是什么。加密和解密采用了不同密钥，加密是用私钥，我们无法知道，而解密要用到公钥，这个公钥必须存在于软件之中。那么，如果我们自己产生一对私钥/公钥，用公钥替换软件中的公钥，那么我们用自己的私钥产生的 `LIC` 就能通过 ECC 校验。

`flexlm` 中进行 ECC 校验的函数是 `l_pubkey_verify`，其定义如下

```c
int l_pubkey_verify (LM_HANDLE *job,  

    unsigned char * input,  

    int inputlen,  

    char *signature,  

    int siglen,  

    int *pubkeysize,  

    unsigned char pubkey[LM_PUBKEYS][LM_MAXPUBKEYSIZ],  

    int strength,  

    int sign_level)  

{  

......  

}  
```

`pubkeysize` 指向公钥块的大小，`pubkey` 就是公钥，例如某软件的公钥数据结构如下：

```
debug002:FFFF6C6C  dd 10h      pubkeysize[0]  

debug002:FFFF6C70  dd 16h      pubkeysize[1]  

debug002:FFFF6C74  dd 1Fh      pubkeysize[2]  

debug002:FFFF6C78  db  66h     pubkey[0]  

debug002:FFFF6C79  db  5Ch   

debug002:FFFF6C7A  db 0C4h   

debug002:FFFF6C7B  db  94h   

debug002:FFFF6C7C  db  68h   

debug002:FFFF6C7D  db 0E5h   

debug002:FFFF6C7E  db  11h   

debug002:FFFF6C7F  db  41h   

debug002:FFFF6C80  db 0EBh   

debug002:FFFF6C81  db 0CCh   

debug002:FFFF6C82  db    8   

debug002:FFFF6C83  db 0CAh   

debug002:FFFF6C84  db 0F6h   

debug002:FFFF6C85  db 0D6h   

debug002:FFFF6C86  db 0F4h   

debug002:FFFF6C87  db 0ABh   

......  

debug002:FFFF6CA0  db  65h    pubkey[1]  

debug002:FFFF6CA1  db 0A1h   

debug002:FFFF6CA2  db    9   

debug002:FFFF6CA3  db  15h   

debug002:FFFF6CA4  db 0B5h   

debug002:FFFF6CA5  db  5Ch   

debug002:FFFF6CA6  db  91h   

debug002:FFFF6CA7  db 0E6h   

debug002:FFFF6CA8  db 0E1h   

debug002:FFFF6CA9  db    0   

debug002:FFFF6CAA  db 0A6h   

debug002:FFFF6CAB  db 0E4h   

debug002:FFFF6CAC  db 0E0h   

debug002:FFFF6CAD  db  95h   

debug002:FFFF6CAE  db  25h   

debug002:FFFF6CAF  db 0D5h   

debug002:FFFF6CB0  db  9Dh   

debug002:FFFF6CB1  db  41h   

debug002:FFFF6CB2  db  43h   

debug002:FFFF6CB3  db 0C7h   

debug002:FFFF6CB4  db  83h   

debug002:FFFF6CB5  db 0F9h   

...  

debug002:FFFF6CC8  db  66h    pubkey[2]  

debug002:FFFF6CC9  db 0E4h 

debug002:FFFF6CCA  db  24h 

debug002:FFFF6CCB  db  7Fh 

debug002:FFFF6CCC  db 0A6h 

debug002:FFFF6CCD  db  36h 

debug002:FFFF6CCE  db  36h 

debug002:FFFF6CCF  db  27h 

debug002:FFFF6CD0  db  25h 

debug002:FFFF6CD1  db  15h

debug002:FFFF6CD2  db 0C0h 

debug002:FFFF6CD3  db 0E0h 

debug002:FFFF6CD4  db  99h 

debug002:FFFF6CD5  db  8Fh 

debug002:FFFF6CD6  db  0Dh

debug002:FFFF6CD7  db  61h 

debug002:FFFF6CD8  db 0B7h 

debug002:FFFF6CD9  db 0C4h 

debug002:FFFF6CDA  db 0E5h 

debug002:FFFF6CDB  db 0E6h 

debug002:FFFF6CDC  db  6Eh 

debug002:FFFF6CDD  db  66h 

debug002:FFFF6CDE  db 0A0h 

debug002:FFFF6CDF  db 0A4h 

debug002:FFFF6CE0  db  88h 

debug002:FFFF6CE1  db  27h 

debug002:FFFF6CE2  db  44h 

debug002:FFFF6CE3  db 0FDh 

debug002:FFFF6CE4  db 0CDh 

debug002:FFFF6CE5  db 0F4h 

debug002:FFFF6CE6  db 0DAh 

...     
```

利用 `flexlm` SDK 产生一对私钥/公钥，内容在 `lmpubkey.h` 中

```c
#include "lmclient.h"     

#define LM_PUBLIC_KEY     

#define LM_KEY_CALLBACK l_pubkey_verify     

lm_extern int l_pubkey_verify();     

static int l_pubseedcnt = 2;     

static unsigned char lm_pubkey[2][3][40] = {{{0x65, 0x9c, 0x83, 0x38, 0x9, 0xdf, 0x5f, 0x24, 0xd7, 0xc2, 0xba, 0xcc, 0xef, 0x3b, 0x3, 0x59},  

  {0x66, 0x9f, 0x68, 0x83, 0x39, 0x6e, 0xda, 0x32, 0xd2, 0x8c, 0xd8, 0x4b, 0x2b, 0xf0, 0xff, 0x90, 0xdc, 0x5b, 0x3e, 0x1f, 0x26, 0x0},  

  {0x65, 0x13, 0x12, 0x82, 0xa3, 0x62, 0x86, 0xb0, 0x71, 0xfb, 0x81, 0x77, 0x48, 0x42, 0x36, 0x27, 0x62, 0x51, 0x8b, 0x10, 0x11, 0xa1, 0x58, 0x2, 0x14, 0x37, 0xee, 0xa6, 0x16, 0x6f, 0x4b}}  

,  

  {{0x65, 0x9d, 0xbd, 0x7a, 0x36, 0xab, 0xb0, 0xf4, 0x35, 0x62, 0xe3, 0xd, 0xaa, 0x53, 0x1c, 0x31},  

  {0x65, 0x9f, 0x61, 0x54, 0x36, 0xe9, 0x1d, 0x30, 0xa5, 0x8f, 0x68, 0xd, 0x19, 0xaf, 0xba, 0xbd, 0xb6, 0x1d, 0x89, 0xe1, 0x99, 0xec},  

  {0x66, 0xc7, 0x69, 0x61, 0xc0, 0x8c, 0x74, 0x96, 0xfc, 0xc0, 0xc4, 0xac, 0x10, 0x2c, 0x38, 0xe9, 0xe6, 0xd6, 0x9b, 0x4b, 0xeb, 0xd5, 0xc7, 0x37, 0x58, 0x75, 0xb5, 0xa3, 0xce, 0xdd, 0xe6}}  

};  

static unsigned int lm_pubsize[2][3] = {{0x10, 0x16, 0x1f}  

,  

  {0x10, 0x16, 0x1f}  

};
```

用上述内容替换内存中 `pubkey`，那么你自己产生的 `LIC` 将成功通过 ECC 校验。

以上仅仅是在内存中进行 `pubkey` 替换，实际上由于 `flexlm` 的 `obfuscate` 措施，`pubkey` 并没有在软件的磁盘文件中连续存放，是在 `l_n36_buf` 中完成重新组合，有时间再来研究这个。
