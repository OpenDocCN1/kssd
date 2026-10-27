
# [原创]GxxeRpcs中的内存hash算法

输入的hash起始地址和内存大小需要加密解密，算法如下：

![](https://bbs.kanxue.com/upload/attach/201812/99882_BYMWM9NWRU9C358.jpg)

`hash`的时候，4字节一轮，每轮用到上次的`hash`值。第一次的`hash`值是加密的地址和长度的异或。

算法如下：

![](https://bbs.kanxue.com/upload/attach/201812/99882_EXNBDXGBVGZMBN9.jpg)

做个测试：计算`MessageBoxA`为起始地址，长度为`0x2000`的`hash`值：

![](https://bbs.kanxue.com/upload/attach/201812/99882_7NH4BUEDUHR38U7.jpg)

这个是我的结果：

![](https://bbs.kanxue.com/upload/attach/201812/99882_4YY2UNNCR3K4ANA.jpg)

这个是tp的结果：

![](https://bbs.kanxue.com/upload/attach/201812/99882_RVRHAXF6PUE38NP.jpg)

下面贴代码：

```
#include <iostream>
#include <windows.h>

// Some convenience macros to make partial accesses nicer
#define LAST_IND(x,part_type)    (sizeof(x)/sizeof(part_type) - 1)
#if defined(__BYTE_ORDER) && __BYTE_ORDER == __BIG_ENDIAN
#  define LOW_IND(x,part_type)   LAST_IND(x,part_type)
#  define HIGH_IND(x,part_type)  0
#else
#  define HIGH_IND(x,part_type)  LAST_IND(x,part_type)
#  define LOW_IND(x,part_type)   0
#endif

typedef          char   int8;
typedef   signed char   sint8;
typedef unsigned char   uint8;
typedef          short  int16;
typedef   signed short  sint16;
typedef unsigned short  uint16;
typedef          int    int32;
typedef   signed int    sint32;
typedef unsigned int    uint32;

#define _BYTE  uint8
#define _WORD  uint16
#define _DWORD uint32
#define _QWORD uint64
#define BYTEn(x, n)   (*((_BYTE*)&(x)+n))
#define WORDn(x, n)   (*((_WORD*)&(x)+n))
#define DWORDn(x, n)  (*((_DWORD*)&(x)+n))

#define LOBYTE(x)  BYTEn(x,LOW_IND(x,_BYTE))
#define LOWORD(x)  WORDn(x,LOW_IND(x,_WORD))
#define LODWORD(x) DWORDn(x,LOW_IND(x,_DWORD))
#define HIBYTE(x)  BYTEn(x,HIGH_IND(x,_BYTE))
#define HIWORD(x)  WORDn(x,HIGH_IND(x,_WORD))
#define HIDWORD(x) DWORDn(x,HIGH_IND(x,_DWORD))
#define BYTE1(x)   BYTEn(x,  1)         // byte 1 (counting from 0)
#define BYTE2(x)   BYTEn(x,  2)
#define BYTE3(x)   BYTEn(x,  3)
#define BYTE4(x)   BYTEn(x,  4)
#define BYTE5(x)   BYTEn(x,  5)
#define BYTE6(x)   BYTEn(x,  6)
#define BYTE7(x)   BYTEn(x,  7)
#define BYTE8(x)   BYTEn(x,  8)
#define BYTE9(x)   BYTEn(x,  9)
#define BYTE10(x)  BYTEn(x, 10)
#define BYTE11(x)  BYTEn(x, 11)
#define BYTE12(x)  BYTEn(x, 12)
#define BYTE13(x)  BYTEn(x, 13)
#define BYTE14(x)  BYTEn(x, 14)
#define BYTE15(x)  BYTEn(x, 15)
#define WORD1(x)   WORDn(x,  1)
#define WORD2(x)   WORDn(x,  2)         // third word of the object, unsigned
#define WORD3(x)   WORDn(x,  3)
#define WORD4(x)   WORDn(x,  4)
#define WORD5(x)   WORDn(x,  5)
#define WORD6(x)   WORDn(x,  6)
#define WORD7(x)   WORDn(x,  7)

DWORD __fastcall EncryParam( DWORD input)
{
  unsigned int result; 
  DWORD xor_val = input ^ 0x95148367;
  result = xor_val;
  char val = xor_val;
  LOBYTE(xor_val) = xor_val ^ BYTE1(result) ^ BYTE2(xor_val);
  HIBYTE(result) ^= xor_val;
  BYTE1(result) ^= val;
  BYTE2(result) = xor_val;
  if (val != (_BYTE)xor_val)
  {
    char v4 = xor_val ^ val;
    LOBYTE(xor_val) = v4 ^ xor_val;
    BYTE2(result) = xor_val;
    LOBYTE(result) = xor_val ^ v4;
  }
  return result;
}

DWORD  __fastcall DecryParam(DWORD *input)
{
  unsigned int result;
  unsigned int  val2 = *input;
  char check_val = *input;
  result = val2;
  LOBYTE(val2) = BYTE2(val2);
  if (check_val != BYTE2(val2))
  {
    char val = BYTE2(val2) ^ check_val;
    LOBYTE(val2) = val ^ BYTE2(val2);
    check_val = val ^ BYTE2(val2) ^ val;
    LOBYTE(result) = check_val;
  }
  HIBYTE(result) ^= val2;
  BYTE2(result) = BYTE1(result) ^ val2;
  BYTE1(result) ^= check_val;
  return result ^ 0x95148367;
}

DWORD GetMemoryHash(DWORD addr,DWORD len) {
  addr = EncryParam(addr);
  len = EncryParam(len);
  auto hash = addr ^ len;
  const auto decry_addr = DecryParam(&addr);
  const auto decry_len = DecryParam(&len);
  auto * buff = (DWORD *)decry_addr;
  for(int i = 0; i < decry_len / 4;i++) {
    hash = buff[i] ^ (DWORD)&buff[i] ^ hash;
  }
  return hash;
}

int main() {
  LoadLibraryA("user32.dll");
  auto messagebox_addr = (DWORD)GetProcAddress(GetModuleHandleA("user32.dll"), "MessageBoxA");
  auto result = GetMemoryHash(messagebox_addr, 0x2000);
  printf("addr:%x\n", EncryParam(messagebox_addr));
  printf("len:%x\n", EncryParam(0x2000));
  printf("hash:%x\n", result);
  return 1;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
