
# Lambda Pro v2.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue285.htm`](https://www.pediy.com/kssd/pediy04/kanxue285.htm)

```
/////////////////////////////////////////////////////////////////////

// 目标软件：Lambda Pro

// 软件版本：v2.0-MAC-Limit-DEMO-0724

// 官方网站：http://www.coretek.com.cn/

// 软件授权：商用软件

// 操作系统：Win95/98/ME、WinNT/2000

// 软件简介：

// 中国拥有自己知识产权的实时操作系统 - Delta OS 的 IDE。因为是简体中文

// 软件，个人觉得很适合于初学实时操作系统(RTOS)的人们作为学习参考。

// 软件保护：IntsallShield v6.x 序列号保护(此序列号与网卡 MAC 地址有关)

/////////////////////////////////////////////////////////////////////

// 使用工具：InstallShield Decompiler v1.00 beta 7

//               (用于反汇编 SETUP.INX 脚本文件)

//               Borland C++ v3.1

//               (用于编写序列号生成器)

//               Our Brain...:-)

/////////////////////////////////////////////////////////////////////

// 关于本文：本文主要目的在于教学，让初学者掌握 InstallShoeld 序列号分

// 析的一般方法，所以找了这个很不常见的软件...请勿将此教程

// 用于商业目的。

//               Always Your Best Friend: FiNALSErAPH

// 水平有限，难免疏漏...

// Any Question?

// Mail To: FiNALSErAPH@yahoo.com.cn

//                   2001-12-04

/////////////////////////////////////////////////////////////////////

// 第一部分：脚本分析

/////////////////////////////////////////////////////////////////////

// ISD SETUP.INX SETUP.TXT -O -E

// (将 SETUP.INX 反编译并将结果输出至 SETUP.TXT 文件)

/////////////////////////////////////////////////////////////////////

function program()
begin
/* 000055A5: 0006 */   g_str27 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

/* 000055D3: 0006 */   g_str28 = "YW9KV8IU7MT6DS5NQ4AP3LJ2EH1RG0OFZCBX";

/* 00005601: 0021 */   function_22();

                       //将安装目录下的 TEST_001.EXE 文件拷贝到 C:\并重命名

                       //为 GETMAC.EXE。执行 C:\GETMAC.EXE(获取网卡 MAC 地址)

                       //生成 C:\MACINFO.TXT(里面就是网卡的物理地址)

                       //在安装正常完成后这些文件会被删除

/* 00005607: 0021 */   function_184(12);

/* 00005612: 0021 */   function_10();

/* 00005618: 0021 */   function_8();

/* 0000561E: 0021 */   function_9();

/* 00005624: 0006 */   g_number28 = 0;

/* 00005630: 0021 */   function_1();

                       //密码验证程序入口

 .

 .

 .

end;

/////////////////////////////////////////////////////////////////////

// : Call Referenced(1):

// :  00005630,

function NUMBER function_1()
    NUMBER n0;

    NUMBER n1;

begin

// : Jump Referenced(1):

// :  00005A82,

label_13:

/* 00005A4F: 0021 */   function_23();

                       //从 C:\MACINFO.TXT 中获取网卡信息备用

// : Jump Referenced(1):

// :  00005AB8,

label_14:

/* 00005A57: 0021 */   function_11();

                       //显示欢迎画面

/* 00005A5D: 0006 */   n0 = LAST_RESULT;

/* 00005A67: 000D */   n1 = n0 == 12;

/* 00005A76: 0004 */   if(! n1) goto label_15;

/* 00005A82: 0005 */   goto label_13;

// : Jump Referenced(2):

// :  00005A76, 00005AEE,

label_15:

/* 00005A8D: 0021 */   function_12();

                       //显示版权信息

/* 00005A93: 0006 */   n0 = LAST_RESULT;

/* 00005A9D: 000D */   n1 = n0 == 12;

/* 00005AAC: 0004 */   if(! n1) goto label_16;

/* 00005AB8: 0005 */   goto label_14;

// : Jump Referenced(3):

// :  00005AAC, 00005B3F, 00005B83,

label_16:

/* 00005AC3: 0021 */   function_13();

                       //获取用户输入的序列号(赋给 g_str26)

/* 00005AC9: 0006 */   n0 = LAST_RESULT;

/* 00005AD3: 000D */   n1 = n0 == 12;

/* 00005AE2: 0004 */   if(! n1) goto label_17;

/* 00005AEE: 0005 */   goto label_15;

// : Jump Referenced(1):

// :  00005AE2,

label_17:

/* 00005AF9: 000D */   n1 = n0 == 1;

/* 00005B08: 0004 */   if(! n1) goto label_18;

/* 00005B14: 0021 */   function_24();

                       //序列号验证程序的核心部分(即算法验证部分)

/* 00005B1A: 0006 */   n1 = LAST_RESULT;

/* 00005B24: 000D */   n1 = n1 == 1;

/* 00005B33: 0004 */   if(! n1) goto label_18;

/* 00005B3F: 0005 */   goto label_16;

                       //输入序列号错误则陷入循环

 .

 .

 .

end;

/////////////////////////////////////////////////////////////////////

// : Call Referenced(1):

// :  00005B14,

function NUMBER function_24()
    STRING s0;

    STRING s1[13];

    STRING s2;

    NUMBER n0;

    NUMBER n1;

    NUMBER n2;

    NUMBER n3;

    NUMBER n4;

    NUMBER n5;

    NUMBER n6;

    NUMBER n7;

    NUMBER n8;

begin

/* 00007036: 0006 */   s2 = g_str26;

/* 00007040: 0021 */   function_455(s2);

/* 00007049: 0006 */   n7 = LAST_RESULT;

/* 00007053: 000E */   n7 = n7 != 52;

/* 00007062: 0004 */   if(! n7) goto label_82;

/* 0000706E: 0021 */   function_304("Invalid serial number!", -65535);

/* 00007092: 0027 */   // -- Start Return Code -- //

/* 00007096: 0023 */   return 1;

//判断输入的序列号是否为 52 位

// : Jump Referenced(1):

// :  00007062,

label_82:

/* 000070A1: 0006 */   n0 = 0;

// : Jump Referenced(1):

// :  00007106,

label_83:

/* 000070AF: 000B */   n7 = n0 <= 51;

/* 000070BE: 0004 */   if(! n7) goto label_84;

/* 000070CA: 001E */   n7 = s2[n0];

/* 000070D7: 0021 */   function_18(n7);

/* 000070E0: 0006 */   n7 = LAST_RESULT;

/* 000070EA: 001D */   s2[n0] = n7;

/* 000070F7: 0007 */   n0 = n0 + 1;

/* 00007106: 0005 */   goto label_83;

//将输入序列号中的小写字母全部换成大写字母

// : Jump Referenced(1):

// :  000070BE,

label_84:

/* 00007111: 001E */   n7 = s2[0];

/* 00007120: 000C */   n7 = n7 >= 48;

/* 0000712F: 001E */   n8 = s2[0];

/* 0000713E: 000B */   n8 = n8 <= 52;

/* 0000714D: 0019 */   n7 = n7 && n8;

/* 0000715A: 0004 */   if(! n7) goto label_85;

/* 00007166: 0006 */   g_str28 = "YW9KV8IU7MT6DS5NQ4AP3LJ2EH1RG0OFZCBX";

/* 00007194: 0005 */   goto label_86;

// : Jump Referenced(1):

// :  0000715A,

label_85:

/* 0000719F: 0006 */   g_str28 = "IR6UQ5GP4MO3YN2SK1AJ0THZLFX9ECW8DBV7";

// : Jump Referenced(1):

// :  00007194,

label_86:

/* 000071CF: 0006 */   n0 = 1;

//根据输入的第一个字符选择“字符替换表”

// : Jump Referenced(1):

// :  00007244,

label_87:

/* 000071DD: 000B */   n7 = n0 <= 51;

/* 000071EC: 0004 */   if(! n7) goto label_88;

/* 000071F8: 001E */   n7 = s2[n0];

/* 00007205: 0021 */   function_19(g_str28, n7);

/* 00007211: 0006 */   n1 = LAST_RESULT;

/* 0000721B: 001E */   n7 = g_str27[n1];

/* 00007228: 001D */   s2[n0] = n7;

/* 00007235: 0007 */   n0 = n0 + 1;

/* 00007244: 0005 */   goto label_87;

//根据上面选择的“字符替换表”对输入字符串进行变换(注意：[00]不变，只变换[01] - [51])

//由 g_str28 确定位置 - >对应于 g_str27 的字符

// : Jump Referenced(1):

// :  000071EC,

label_88:

/* 0000724F: 0006 */   n2 = 1;

/* 0000725B: 0006 */   n1 = 49;

// : Jump Referenced(1):

// :  000073AE,

label_89:

/* 00007269: 000B */   n7 = n1 <= 51;

/* 00007278: 0004 */   if(! n7) goto label_93;

/* 00007284: 001E */   n7 = s2[n2];

/* 00007291: 0021 */   function_20(n7);

/* 0000729A: 0006 */   n3 = LAST_RESULT;

/* 000072A4: 0007 */   n0 = n2 + 1;

/* 000072B3: 0007 */   n7 = n2 + 15;

// : Jump Referenced(1):

// :  00007319,

label_90:

/* 000072C4: 000B */   n8 = n0 <= n7;

/* 000072D1: 0004 */   if(! n8) goto label_91;
```
