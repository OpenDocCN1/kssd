
# Font Creator Program

> 原文：[`www.pediy.com/kssd/pediy02/forum146.htm`](https://www.pediy.com/kssd/pediy02/forum146.htm)

谁有好像国际音标一样韦氏音标的字体或者知道它的下载？

如果有的请 E-mail: `peter@nie-go.com`

如果知道它下载请知之，本人用了许多搜器都没有找。

如果真的没有只好自己做字体，这里有作字体要破解：

## 软件信息

- **软件名称**：`Font Creator Program`
- **整理日期**：`2000.7.14`
- **最新版本**：`2.2`
- **文件大小**：`1165KB`
- **软件授权**：共享软件
- **使用平台**：`Win95/98/NT`
- **发布公司**：`home page`

### 软件简介

可用来制作字体(`TTF`)的程序，除可自行制作外也能够直接编辑修改视窗操作系统上的任何 `TrueType` 字体，制作好的字体或修改好的字体能够让你储存和使用。

### 本地下载

- `http://www.newhua.com.cn/down/fcp2.exe`
- `http://newhua.xingtai.net/down/fcp2.exe`

### 汉化补丁

本地下载：`2.20 build 227` 汉化补丁 `46KB` `YC-周` 汉化

- `http://newhua.21f.net/down/HB_fcp220(227)_YC_ZHOU.zip`
- `http://www.newhua.com.cn/down/HB_fcp220(227)_YC_ZHOU.zip`

## 破解说明

最好把一个完整的过程写出来，最好把密码也写出来。

这个软件本人不会用的，如果有大客用过也发表一下好吗？

- `Name:` `dr0`
- `Company:` （不要填任何东西）
- `Code:` `H0D220828026`

用 `hmemcpy`，中断后按 `12` 次 `F12` 就到了：

```
015F:004B74A7  CALL       00434534             //hmemcpy
015F:004B74AC  PUSH       DWORD PTR [EBP-10]
015F:004B74AF  LEA        EDX,[EBP-14]
015F:004B74B2  MOV        EAX,[EBP-04]
015F:004B74B5  MOV        EAX,[EAX+00000304]
015F:004B74BB  CALL       00434534             //hmemcpy
015F:004B74C0  PUSH       DWORD PTR [EBP-14]
015F:004B74C3  LEA        EDX,[EBP-18]
015F:004B74C6  MOV        EAX,[EBP-04]
015F:004B74C9  MOV        EAX,[EAX+0000030C]
015F:004B74CF  CALL       00434534             //hmemcpy
015F:004B74D4  PUSH       DWORD PTR [EBP-18]
015F:004B74D7  LEA        EDX,[EBP-1C]
015F:004B74DA  MOV        EAX,[EBP-04]
015F:004B74DD  MOV        EAX,[EAX+00000314]
015F:004B74E3  CALL       00434534             //hmemcpy
015F:004B74E8  PUSH       DWORD PTR [EBP-1C]
015F:004B74EB  LEA        EAX,[EBP-08]
015F:004B74EE  MOV        EDX,00000004
015F:004B74F3  CALL       004040A4
015F:004B74F8  LEA        EDX,[EBP-24]
015F:004B74FB  MOV        EAX,[EBP-04]
015F:004B74FE  MOV        EAX,[EAX+0000030C]
015F:004B7504  CALL       00434534             //hmemcpy
015F:004B7509  MOV        EAX,[EBP-24]
015F:004B750C  MOV        EDX,00000003
015F:004B7511  DEC        EDX
015F:004B7512  CMP        EDX,[EAX-04]
015F:004B7515  JB         004B751C
015F:004B7517  CALL       00402F34
015F:004B751C  INC        EDX
015F:004B751D  MOV        AL,[EDX+EAX-01]
015F:004B7521  PUSH       EAX
015F:004B7522  LEA        EDX,[EBP-28]
015F:004B7525  MOV        EAX,[EBP-04]
015F:004B7528  MOV        EAX,[EAX+000002E0]
015F:004B752E  CALL       00434534
015F:004B7533  MOV        EAX,[EBP-28]
015F:004B7536  LEA        ECX,[EBP-20]
015F:004B7539  POP        EDX
015F:004B753A  CALL       004B496C
015F:004B753F  MOV        EAX,[EBP-20]
015F:004B7542  PUSH       EAX
015F:004B7543  LEA        EDX,[EBP-2C]
015F:004B7546  MOV        EAX,[EBP-08]
015F:004B7549  CALL       004089F4
015F:004B754E  MOV        EDX,[EBP-2C]       //真注册码
015F:004B7551  POP        EAX                 //假注册码
015F:004B7552  CALL       004040F4           //比较注册码
015F:004B7557  JNZ        004B75AF
```
