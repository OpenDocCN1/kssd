
```c
_stprintf(tt, L"BB %02X%02X%02X%02X\tMOV EBX, %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

break;

}

case 0xBC:

{

//取出MOV后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"BC %02X%02X%02X%02X\tMOV ESP, %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

break;

}

case 0xBD:

{

//取出MOV后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"BD %02X%02X%02X%02X\tMOV EBP, %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

break;

}

case 0xBE:

{

//取出MOV后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"BE %02X%02X%02X%02X\tMOV ESI, %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

break;

}

case 0xBF:

{

//取出MOV后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"BF %02X%02X%02X%02X\tMOV EDI, %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

break;

}

}

if (codeList[index]<0xB8)

{

index += 2;

}

else {

index += 5;

}

}

/*

C2的解析

*/

void Translate_C2() {

//取出RETN后面的2字节立即数

DWORD im = *((WORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

_stprintf(tt, L"C2 %02X%02X\t\tRETN %04X\r\n", b1, b2, im);

append(tt);

free(tt);

index += 3;

}

/*

C3的解析

*/

void Translate_C3() {

append(L"C3\t\tRETN\r\n");

index ++;

}

/*

CA的解析

*/

void Translate_CA() {

//取出RETF后面的2字节立即数

DWORD im = *((WORD*)(codeList + index + 1));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

_stprintf(tt, L"CA %02X%02X\t\tRETF %04X\r\n", b1, b2, im);

append(tt);

free(tt);

index += 3;

}

/*

CB的解析

*/

void Translate_CB() {

append(L"CB\t\tRETF\r\n");

index ++;

}

/*

E0 E1 E2 E3的解析

*/

void Translate_E0_E3() {

switch (codeList[index])

{

case 0xE0:

{

//取出EX后面的1字节立即数

char im = *((char*)(codeList + index + 1));

DWORD ii = im + baseAddr + index + 2;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

_stprintf(tt, L"E0 %02X\t\tLOOPNE SHORT %08X\r\n", (BYTE)im, ii);

append(tt);

free(tt);

break;

}

case 0xE1:

{

//取出EX后面的1字节立即数

char im = *((char*)(codeList + index + 1));

DWORD ii = im + baseAddr + index + 2;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

_stprintf(tt, L"E1 %02X\t\tLOOPE SHORT %08X\r\n", (BYTE)im, ii);

append(tt);

free(tt);

break;

}

case 0xE2:

{

//取出EX后面的1字节立即数

char im = *((char*)(codeList + index + 1));

DWORD ii = im + baseAddr + index + 2;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

_stprintf(tt, L"E2 %02X\t\tLOOP SHORT %08X\r\n", (BYTE)im, ii);

append(tt);

free(tt);

break;

}

case 0xE3:

{

//取出EX后面的1字节立即数

char im = *((char*)(codeList + index + 1));

DWORD ii = im + baseAddr + index + 2;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

_stprintf(tt, L"E3 %02X\t\tJECXZ SHORT %08X\r\n", (BYTE)im, ii);

append(tt);

free(tt);

break;

}

}

index += 2;

}

/*

E8的解析

*/

void Translate_E8() {

//取出E8后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

im = im + 5 + baseAddr + index;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"E8 %02X%02X%02X%02X\tCALL %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

index += 5;

}

/*

E9的解析

*/

void Translate_E9() {

//取出E9后面的4字节立即数

DWORD im = *((DWORD*)(codeList + index + 1));

im = im + 5 + baseAddr + index;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

_stprintf(tt, L"E9 %02X%02X%02X%02X\tJMP %08X\r\n", b1, b2, b3, b4, im);

append(tt);

free(tt);

index += 5;

}

/*

EB的解析

*/

void Translate_EB() {

//取出EB后面的1字节立即数

char im = *((char*)(codeList + index + 1));

DWORD ii = im + 2 + baseAddr + index;

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

_stprintf(tt, L"EB %02X\t\tJMP SHORT %08X\r\n", b1, ii);

append(tt);

free(tt);

index += 2;

}

/*

EA的解析

*/

void Translate_EA() {

//取出EA后面的4字节立即数

DWORD im1 = *((DWORD*)(codeList + index + 1));

//取出EA后面的2字节跨段数据

WORD im2 = *((WORD*)(codeList + index + 5));

TCHAR* tt = (TCHAR*)malloc(256);

memset(tt, 0, 256);

BYTE b1 = *((BYTE*)(codeList + index + 1));

BYTE b2 = *((BYTE*)(codeList + index + 2));

BYTE b3 = *((BYTE*)(codeList + index + 3));

BYTE b4 = *((BYTE*)(codeList + index + 4));

BYTE b5 = *((BYTE*)(codeList + index + 5));

BYTE b6 = *((BYTE*)(codeList + index + 6));

_stprintf(tt, L"EA %02X%02X%02X%02X:%02X%02X JMP FAR %04X:%08X\r\n", b1,b2,b3,b4,b5,b6, im2,im1);

append(tt);

free(tt);

index += 7;

}

};
```

# 验证

由于直接拿exe去验证的话,会有很多硬编码我们还没解析到,因此我这里将读取出来的`codeList`直接写死一串字节

```c++
unsigned char t[50] = {
0x50 ,0x5F ,0x40 ,0x4F ,0x0F ,0x84 ,0x50 ,0x5F ,0x14 ,0x7F ,
0x78 ,0x0f ,0x90 ,0x95 ,0xB0 ,0x90 ,0xB8 ,0x0F ,0x80 ,0xC2 ,0xC3 ,0xC3 ,0xC2 ,
0xC2 ,0xC3 ,0xE0 ,0x90 ,0xE8 ,0x12 ,0x34 ,0x56 ,0x78 ,0xE9 ,0x90 ,0x0F ,0x80 ,
0x12 ,0xEB,0x90 ,0xEA ,0xE8 ,0xE9 ,0x00 ,0x00 ,0x0F ,0x80
};
```

这串字节刚好包含了我们解析的字节,并且很多字节既可以是偏移也可以是OpCode

OD打开经典的飞鸽,将这段编码写到`401000`处

![1](upload/attach/202010/867232\_W9TMNPGCWN235U4.png)

可以看到这是这段编码从0x50处开始解析的结果

运行我们的程序进行对比

![1](upload/attach/202010/867232\_MCQY39P68HMXES8.png)

进行对比后结果是没有什么区别的,至此我们的引擎中对于`定长指令`的解析已经全部结束

下章开始将会进行最复杂的`变长指令`的解析

# 结尾

定长指令的解析主要是锻炼思路方法,因此很多冗余代码其实可以进行函数封装的

我这里图房间就没进行封装,直接CV了,如果看着不方便的话,附加给各位上传了项目源码

感谢各位的阅读

# 目录

[Step.1\_引擎大纲](https://bbs.pediy.com/thread-262477.htm)

[Step.2\_框架搭建](https://bbs.pediy.com/thread-262477.htm)

Step.3\_经典定长指令的解析

[Step.4\_变长指令复习1](https://bbs.pediy.com/thread-262503.htm)

[Step.5\_变长指令复习2 - 指令前缀](https://bbs.pediy.com/thread-262532.htm)

[Step.6\_引擎核心->函数封装](https://bbs.pediy.com/thread-262633.htm)

# 内容目录

目录

- [目录](https://bbs.kanxue.com/#目录)

- [内容目录](https://bbs.kanxue.com/#内容目录)

- [前言](https://bbs.kanxue.com/#前言)

- 白皮书A-2 A-3部分截图

- [功能说明](https://bbs.kanxue.com/#功能说明)

- [逻辑构思](https://bbs.kanxue.com/#逻辑构思)

- [代码实现](https://bbs.kanxue.com/#代码实现)

- [验证](https://bbs.kanxue.com/#验证)

- [结尾](https://bbs.kanxue.com/#结尾)

# 前言

由于硬编码种类繁多,按顺序分析不合理,所以我们将根据Intel白皮书(Step.1中已提供下载) 图A-2 与 A-3中提供的数据,先从定长指令开始入手,本节将会先解析 `50~5F 40~4F 0F80~0F8F 70~7F 90~97 B0~BF C2 C3 CA CB E0 E1 E2 E3 E8 E9 EA EB`这些经典定长指令.将解析的逻辑通过这些简单的指令梳理清楚

# 白皮书A-2 A-3部分截图

![1](https://bbs.kanxue.com/upload/attach/202010/867232_WGXKDGPDTPKGR68.png)

![1](https://bbs.kanxue.com/upload/attach/202010/867232_WCM9GWKGFKNJZMG.png)

![1](https://bbs.kanxue.com/upload/attach/202010/867232_BEPJZMFSXEH73VF.png)
