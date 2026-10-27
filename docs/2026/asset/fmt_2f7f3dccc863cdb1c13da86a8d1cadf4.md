
![](https://bbs.kanxue.com/upload/attach/202001/817966_BXX34WDN9T5MPF7.jpg)

#### 0x03.5 `void Blowfish_Decrypt(BLOWFISH_CTX *ctx, unsigned long *xl, unsigned long *xr)`

解密过程是加密过程的反向，如有困惑，可参照源码理解，不再赘述。

#### 0x03.6 `static unsigned long F(BLOWFISH_CTX *ctx, unsigned long x)`

轮函数工作过程：

1. 将参数`x`逐字节分割，赋值给`a`,`b`,`c`,`d`四个变量(e.g.:`x=0x21564378`,则`a=0x21`,`b=0x56`,`c=0x43`,`d=0x78`)

2. `y = ((ctx->S[0][a] + ctx->S[1][b]) ^ ctx->S[2][c]) + ctx->S[3][d];`

3. 返回`y`的值

## 0x04 Demo

此Demo来自[Paul Kocher](https://bbs.kanxue.com/elink@c00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6j5$3S2F1k6h3W2W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3y4G2k6r3g2Q4x3V1k6T1k6Y4y4Z5i4K6u0V1K9$3!0U0i4K6u0W2P5X3W2H3)版本根目录下的`blowfish_test.c`：

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

![](https://bbs.kanxue.com/upload/attach/202001/817966_2BVH84F4EXYYXV7.jpg)

可以看到加密结果并非源码中给出的结果，而在Windows平台下一切正常：

![](https://bbs.kanxue.com/upload/attach/202001/817966_HQC57KGVAVU45UP.jpg)

## 0x05 参考

- [Wikipedia](https://bbs.kanxue.com/elink@702K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6n7L8r3!0%4k6X3W2K6K9q4)9#2k6W2)9J5z5r3y4A6M7r3S2W2M7R3`.`.)

- [圆周率小数部分](https://bbs.kanxue.com/elink@f2eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4#2M7r3g2J5i4K6u0V1j5$3!0E0M7s2g2@1K9h3&6Y4i4K6u0W2L8%4u0Y4i4K6u0r3M7r3W2Q4x3X3c8Z5k6i4S2S2i4K6g2X3j5%4g2J5M7X3g2F1N6q4)9J5k6h3S2@1L8h3H3`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
