
# Base64 编码解码算法 Cpp+sdk 源码

> 原文：[`www.pediy.com/kssd/pediy10/77538.html`](https://www.pediy.com/kssd/pediy10/77538.html)

偶是编程初学者,比较菜![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

为了学习这个 Base64 的编码解码算法,一直搞到现在凌晨 4:20 ![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")

大部位时间都花在 Debug 上而了。

不过总算是弄出来了,我把它拿出来,也给大家看看,献丑了(那些大牛们可别笑哦!![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") )...........

呵呵,希望对跟我一样还在入门的朋友有点帮助吧..![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Base64.rar`

 |

这是我在第一次看 BASE64 编码规则后花 1 小时写的

```c
/*

编码表

A B C D E F G H I J K L M N O P Q
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16

R S T U V W X Y Z a b c d e f g
17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33

i j k l m n o p q r s t u v w x y
34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50

z 0 1 2 3 4 5 6 7 8 9 + / (pad) =
51 52 53 54 55 56 57 58 59 60 61 62 63

*/

int Base64_Encode(char *Src, int SrcLength, char *OutputBuffer, int OutputBufferLength)

{

  int GroupCount;

  int GroupResidualLength;

  int EncodeResultLength;

  int i;

  unsigned char *SrcData;

  unsigned char PaddingBuffer[3];

  static char Base64_EncodeTable[64]={"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"};

  /*将原始数据分为 3 个字节一组*/

  GroupCount = SrcLength / 3;

  GroupResidualLength = SrcLength % 3;

  EncodeResultLength = GroupCount * 4;

  if(0 != GroupResidualLength)

    EncodeResultLength = EncodeResultLength + 4;

  if(EncodeResultLength > OutputBufferLength)

  {

    /*输出缓冲区不足*/

    return 0;

  }

  /*编码分组*/

  SrcData = (unsigned char *)Src;

  for(i = 0; i < GroupCount; i++)

  {

    OutputBuffer[i*4 + 0]= Base64_EncodeTable[ ( SrcData[i*3 + 0]&0xFC )>>2 ];

    OutputBuffer[i*4 + 1]= Base64_EncodeTable[ ((SrcData[i*3 + 0]&0x03)<<4) + ((SrcData[i*3 + 1]&0xF0)>>4) ];

    OutputBuffer[i*4 + 2]= Base64_EncodeTable[ ((SrcData[i*3 + 1]&0x0F)<<2) + ((SrcData[i*3+ 2]&0xC0)>>6) ];

    OutputBuffer[i*4 + 3]= Base64_EncodeTable[ SrcData[i*3 + 2]&0x3F ];

  }

  /*编码未分组的剩余字节*/

  if(0 != GroupResidualLength)

  {

    PaddingBuffer[0] = 0;

    PaddingBuffer[1] = 0;

    PaddingBuffer[2] = 0;

    for(i = 0; i < GroupResidualLength; i++)

      PaddingBuffer[i] = SrcData[GroupCount*3 + i];

    OutputBuffer[GroupCount*4 + 0]= Base64_EncodeTable[ ( PaddingBuffer[0]&0xFC )>>2 ];

    OutputBuffer[GroupCount*4 + 1]= Base64_EncodeTable[ ((PaddingBuffer[0]&0x03)<<4) + ((PaddingBuffer[1]&0xF0)>>4) ];

    OutputBuffer[GroupCount*4 + 2]= Base64_EncodeTable[ ((PaddingBuffer[1]&0x0F)<<2) + ((PaddingBuffer[2]&0xC0)>>6) ];

    OutputBuffer[GroupCount*4 + 3]= Base64_EncodeTable[ PaddingBuffer[2]&0x3F ];

  }

  return EncodeResultLength;

}

int Base64_Decode(char *Src, int SrcLength, char *OutputBuffer, int OutputBufferLength)

{

  int GroupCount;

  int GroupResidualLength;

  int EncodeResultLength;

  int i,j;

  unsigned char *SrcData;

  unsigned char PaddingBuffer[4];

  unsigned char DecodeBuffer[4];

  static unsigned char Base64_DecodeTable[256] = {

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x3E, 0x00, 0x00, 0x00, 0x3F,

    0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E,

    0x0F, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 0x20, 0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28,

    0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 0x30, 0x31, 0x32, 0x33, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00,

    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00

  };

  /*将原始数据分为 4 个字节一组*/

  GroupCount = SrcLength / 4;

  GroupResidualLength = SrcLength % 4;

  EncodeResultLength = GroupCount * 3;

  if(0 != GroupResidualLength)

    EncodeResultLength = EncodeResultLength + 3;

  /*解码分组*/

  SrcData = (unsigned char *)Src;

  for(i = 0; i < GroupCount; i++)

  {

    DecodeBuffer[0] = Base64_DecodeTable[ SrcData[i*4 + 0] ];

    DecodeBuffer[1] = Base64_DecodeTable[ SrcData[i*4 + 1] ];

    DecodeBuffer[2] = Base64_DecodeTable[ SrcData[i*4 + 2] ];

    DecodeBuffer[3] = Base64_DecodeTable[ SrcData[i*4 + 3] ];

    OutputBuffer[i*3 + 0]= (DecodeBuffer[0]<<2) | ((DecodeBuffer[1]&0x30)>>4);

    OutputBuffer[i*3 + 1]= ((DecodeBuffer[1]&0x0f)<<4) | ((DecodeBuffer[2]&0x3c)>>2);

    OutputBuffer[i*3 + 2]= ((DecodeBuffer[2]&0x03)<<6) | (DecodeBuffer[3]&0x3f);

  }

  /*解码未分组的剩余字节*/

  if(0 != GroupResidualLength)

  {

    PaddingBuffer[0] = '=';

    PaddingBuffer[1] = '=';

    PaddingBuffer[2] = '=';

    PaddingBuffer[3] = '=';

    for( i = 0; i < GroupResidualLength; i++ )

    {

      PaddingBuffer[i] = SrcData[GroupCount*4 + i];

    }

    DecodeBuffer[0] = Base64_DecodeTable[ PaddingBuffer[0] ];

    DecodeBuffer[1] = Base64_DecodeTable[ PaddingBuffer[1] ];

    DecodeBuffer[2] = Base64_DecodeTable[ PaddingBuffer[2] ];

    DecodeBuffer[3] = Base64_DecodeTable[ PaddingBuffer[3] ];

    OutputBuffer[GroupCount*3 + 0]= (DecodeBuffer[0]<<2) | ((DecodeBuffer[1]&0x30)>>4);

    OutputBuffer[GroupCount*3 + 1]= ((DecodeBuffer[1]&0x0f)<<4) | ((DecodeBuffer[2]&0x3c)>>2);

    OutputBuffer[GroupCount*3 + 2]= ((DecodeBuffer[2]&0x03)<<6) | (DecodeBuffer[3]&0x3f);

  }

  return EncodeResultLength;

}
```
