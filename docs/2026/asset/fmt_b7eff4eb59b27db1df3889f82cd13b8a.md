
# [原创]Blowfish Cipher浅析

## 0x00 Introduction

Blowfish是一种对称密钥分组密码，Bruce Schneier在1993年设计并公布。它没有专利，任何人均可自由使用。

## 0x01 Detail

- 密钥长度：32-448位
- 分组长度：64位
- 16轮循环的Feistel结构

## 0x02 Feistel Structure

在介绍Blowfish之前先来搞清楚Feistel Structure(下图来自Wikipedia):

![](upload/attach/202001/817966_RP6EHMZQENC8JMK.jpg)

1. 将原数据分成左右两部分
2. 原数据的右侧不变，直接变成下次循环的左侧
3. 将原数据的右侧与子密钥传递给轮函数F
4. 轮函数返回值与原数据左侧进行异或运算，变成下次循环的右侧

上述4个步骤是一轮循环的工作过程，Feistel Structure是进行了16轮循环方才完成一次加密。需要说明一点，在最后一轮循环中左右数据不对调。解密过程是加密过程的反向，具体可参照上图，不再赘述。

## 0x03 Source Code

> 笔者分析的源码是<a href="elink@82bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3S2F1k6h3W2W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3y4G2k6r3g2Q4x3V1k6T1k6Y4y4Z5i4K6u0V1K9$3!0U0i4K6u0W2P5X3W2H3">Paul Kocher</a>版本，并没有使用<a href="elink@ecfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3S2F1k6h3W2W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3y4G2k6r3g2Q4x3V1k6T1k6Y4y4Z5i4K6u0V1M7$3y4Z5i4K6u0W2P5X3W2H3">Bruce Schneier</a>版本。Schneier版本中的子密钥来源是`BLOWFISH.DAT`文件，而Kocher版本是直接在源文件中定义子密钥，分析起来较为直观。如需其他版本可到<a href="elink@4e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3S2F1k6h3W2W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3q4U0j5h3c8W2L8h3W2U0i4K6u0r3j5X3I4G2N6$3k6A6M7$3S2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3S2@1L8h3H3`.">此网站</a>下载

### 0x03.1 `BLOWFISH_CTX`

在`blowfish.h`头文件中有如下定义：

```
typedef struct {
unsigned long P[16 + 2];
unsigned long S[4][256];
} BLOWFISH_CTX;
```

为结构体定义别名，结构体中的两数组即P-Box与S-Box。

### 0x03.2 `ORIG_P[16 + 2]`与`ORIG_S[4][256]`

`ORIG_P`与`ORIG_S`取自<a href="elink@dfcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4#2M7r3g2J5i4K6u0V1j5$3!0E0M7s2g2@1K9h3&6Y4i4K6u0W2L8%4u0Y4i4K6u0r3M7r3W2Q4x3X3c8Z5k6i4S2S2i4K6g2X3j5%4g2J5M7X3g2F1N6q4)9J5k6h3S2@1L8h3H3`.">圆周率的小数位</a>，每4个字节赋值给其中的一个元素。

### 0x03.3 `void Blowfish_Init(BLOWFISH_CTX *ctx, unsigned char *key, int keyLen)`

该函数用来初始化S-Box与P-Box。传递参数中的`key`即密钥，`keyLen`是密钥长度。

1. 对`BLOWFISH_CTX *ctx`中S-Box进行初始化，直接将`ORIG_S`中的每个元素逐一赋值给S-Box
2. 对`BLOWFISH_CTX *ctx`中P-Box进行初始化，具体过程如下：
   a. `data=0x00000000;`
   b. 如果参数中的字符数组`key`长度不足4，则循环使用`key`中字符(当使用到`key`中最后一个字符时，下一个字符是`key`中第一个字符)与`data << 8`进行或运算
   **上述过程总结起来就是将参数中的字符数组`key`转换为ASCII码形式(e.g.:`key[3]="abc"`---->>`0x61626361`并存储于`data`中)**
   c. 将`ORIG_P`中的每个元素与`data`作异或运算后逐一赋值给P-Box
3. `datal=0x00000000;`
   `datar=0x00000000;`
4. 将上面经过变换后的`ctx`，`datal`与`datar`传递给`Blowfish_Encrypt`
5. 将加密后的`datal`与`datar`赋值给P-Box中的元素
6. 重复9次步骤4-5
7. 与步骤4类似，不过这次传递的是上面过程中已经加密后的`datal`与`datar`
8. 将加密后的`datal`与`datar`赋值给S-Box中的元素
9. 重复512次步骤7-8

> 步骤5、8中提到的赋值过程是这样的(以步骤5来举例)：
> 第一次 `P[0]=datal`，`P[1]=datar`
> 第二次 `P[2]=datal`，`P[3]=datar`
> ......

### 0x03.4 `void Blowfish_Encrypt(BLOWFISH_CTX *ctx, unsigned long *xl, unsigned long *xr)`

该函数是Blowfish的加密部分。传递参数中的`ctx`应该已经初始化过S-Box与P-Box，`xl`指向原数据的左半部分，`xr`指向原数据的右半部分。

1. 左侧数据与P-Box中第i个元素作异或运算(i=n-1,其中n是轮数)
2. 将左侧数据与`ctx`传递给轮函数F
3. 右侧数据与轮函数F返回值作异或运算
4. 交换运算后的左右两侧数据
5. 重复16次步骤1-5
6. 将最后一轮运算互换的左右两侧数据换回来
7. 右侧数据与P-Box中第16个元素作异或运算
8. 左侧数据与P-Box中第17个元素作异或运算

上述过程即一次完整的加密过程，可参照下图来理解(来自Wikipedia，其中轮函数F的工作过程见`0x03.6`)：

![](upload/attach/202001/817966_BXX34WDN9T5MPF7.jpg)

### 0x03.5 `void Blowfish_Decrypt(BLOWFISH_CTX *ctx, unsigned long *xl, unsigned long *xr)`

解密过程是加密过程的反向，如有困惑，可参照源码理解，不再赘述。

### 0x03.6 `static unsigned long F(BLOWFISH_CTX *ctx, unsigned long x)`

轮函数工作过程：
1. 将参数`x`逐字节分割，赋值给`a`,`b`,`c`,`d`四个变量(e.g.:`x=0x21564378`,则`a=0x21`,`b=0x56`,`c=0x43`,`d=0x78`)
2. `y = ((ctx->S[0][a] + ctx->S[1][b]) ^ ctx->S[2][c]) + ctx->S[3][d];`
3. 返回y的值

## 0x04 Demo

此Demo来自<a href="elink@d3eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3S2F1k6h3W2W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3y4G2k6r3g2Q4x3V1k6T1k6Y4y4Z5i4K6u0V1K9$3!0U0i4K6u0W2P5X3W2H3">Paul Kocher</a>版本根目录下的`blowfish_test.c`：

```
#include <stdio.h>
#include "blowfish.h"
void main(void) {
unsigned long L = 1, R = 2;
BLOWFISH_CTX ctx;
Blowfish_Init (&ctx, (unsigned char*)"TESTKEY", 7);
Blowfish_Encrypt(&ctx, &L, &R);
printf("%08lX %08lX\n", L, R);
if (L == 0xDF333FD2L && R == 0x30A71BB4L)
printf("Test encryption OK.\n");
else
printf("Test encryption failed.\n");
Blowfish_Decrypt(&ctx, &L, &R);
if (L == 1 && R == 2)
printf("Test decryption OK.\n");
else
printf("Test decryption failed.\n");
}
```

需要说明的一点是Paul Kocher这个版本并没有考虑到小端序的情况，它均按大端序来处理，所以如果在Linux平台运行此Demo会像下图所示：

![](upload/attach/202001/817966_2BVH84F4EXYYXV7.jpg)

可以看到加密结果并非源码中给出的结果，而在Windows平台下一切正常：

![](upload/attach/202001/817966_HQC57KGVAVU45UP.jpg)

## 0x05 参考

- <a href="elink@612K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6n7L8r3!0%4k6X3W2K6K9q4)9#2k6W2)9J5z5r3y4A6M7r3S2W2M7R3`.`.">Wikipedia</a>
- <a href="elink@eb0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4#2M7r3g2J5i4K6u0V1j5$3!0E0M7s2g2@1K9h3&6Y4i4K6u0W2L8%4u0Y4i4K6u0r3M7r3W2Q4x3X3c8Z5k6i4S2S2i4K6g2X3j5%4g2J5M7X3g2F1N6q4)9J5k6h3S2@1L8h3H3`.">圆周率小数部分</a>
