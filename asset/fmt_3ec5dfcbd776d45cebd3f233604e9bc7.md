
# Hash 算法工具

> 原文：[`www.pediy.com/kssd/pediy11/112273.html`](https://www.pediy.com/kssd/pediy11/112273.html)

在各位前辈面前抖胆放一款自己写的小东西--  

该程序用于计算字符串或文件的`Hash`值\[基于`Crypto++`lib\]，目前支持如下算法：  

```
/* 00 */ CRC32 // 非加密校验和  
/* 01 */ Adler32 // 非加密校验和  
/* 02 */ Tiger  
/* 03 */ Panama  
/* 04 */ Whirlpool  
/* 05 */ MD2  
/* 06 */ MD4  
/* 07 */ MD5  
/* 08 */ RIPEMD-128  
/* 09 */ RIPEMD-160  
/* 10 */ RIPEMD-256  
/* 11 */ RIPEMD-320  
/* 12 */ SHA1  
/* 13 */ SHA2-224  
/* 14 */ SHA2-256  
/* 15 */ SHA4-384  
/* 16 */ SHA4-512  
/* 17 */ HMAC-Tiger  
/* 18 */ HMAC-Panama  
/* 19 */ HMAC-Whirlpool  
/* 20 */ HMAC-MD2  
/* 21 */ HMAC-MD4  
/* 22 */ HMAC-MD5  
/* 23 */ HMAC-RIPEMD-128  
/* 24 */ HMAC-RIPEMD-160  
/* 25 */ HMAC-RIPEMD-256  
/* 26 */ HMAC-RIPEMD-320  
/* 27 */ HMAC-SHA1  
/* 28 */ HMAC-SHA2-224  
/* 29 */ HMAC-SHA2-256  
/* 30 */ HMAC-SHA4-384  
/* 31 */ HMAC-SHA4-512  
```

![](img/d836e5e13cd7a8e895bff67e5d74bef6.jpg)  

下载地址:

`Hash.rar`  

楼下补了源码  

附件中给出了该程序的源码, 用`VC++ 6.0`编译,注意,依赖于`Crypto++`, 我用的是`5.6.0`版本, 同时要注意源码目录中的`lib`目录, 将其添加到`VC`环境变量中.

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || [`Hash_[3.0.20].zip`](112273/851879/47149.zip)  
|
