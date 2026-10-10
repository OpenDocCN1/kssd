

# Driver.Suite.3.0.1.的 license 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6777.htm`](https://www.pediy.com/kssd/pediy06/pediy6777.htm)

制作 Compuware.SoftIce.Driver.Suite.3.0.1.StinkyD 的 license  
  
  
软件名称: Compuware.SoftICE.Driver.Suite.3.0.1  
下载地址：fttp://wxw.exetools.com/incoming/Compuware.SoftICE.Driver.Suite.3.0.1.StinkyD/  
  
工具及注册机下载：[`www.pediy.com/practise/bbs6/cws3xw32.rar`](http://www.pediy.com/practise/bbs6/cws3xw32.rar)  
  
保护方式: FlexLm  
使用工具：OllyDbg, IDA, HexWorkshop  
  
文章作者：tulipfan\[CCG\]  
写作时间: 2003-9-3  
  
通过这篇文章我们将向你介绍如何制作 FlexLm 的 license  
  
  
第一部分: 废话  
  
这个软件前些日子就 Down 了,不过没有安装,因为我现在比较喜欢使用 OllyDbg,一般的软件用它完全足够了  
StinkyD 版本身没有带破解,不过可以使用 ROR 的 keygen 和 ds3lic.dat.  
  
关于 ROR  
  
从进半年来看,ROR 组织是目前发布作品最多,质量最好的组织之一,特别是他的注册机,做的很专业!  
  
而且听说是国人自己的破解组织!自豪中...  
  
  
第二部分: 转入正题  
  
破解 FlexLm 最主要的是找到 4 样东西.  
  
1. 确定版本号  
2. 找 vendor\_id  
3. 计算 ENCRYPTION\_SEED  
4. 找 FEATURE  
  
有了这几个,就可以使用 FlexLm SDK 做出注册机了.  
  
1. 确定版本号  
  
确定版本号可以通过 lmutil.exe 来确定  
  
在命令行运行 lmutil.exe lmver cws3xw32.dll  
  
lmutil - Copyright (C) 1989-2001 Globetrotter Software, Inc.  
FLEXlm v7.2i (liblmgr.a), Copyright (C) 1988-2001 Globetrotter Software, Inc.  
  
还有一个办法就是用二进制编辑器,推荐使用 HexWorkshop,它的查找功能很强,特别是 Find All Instances 更是我最最常用的.  
在查找对话框 Type 中选择类型 Text String,Value 中输入"@(#) FLEXlm v",查找,"@(#) FLEXlm v"后面的就是版本号.  
  
最后确定版本号是 7.2i,这一步的目的有两个,一是在 lmkg.exe 中生成 Vendor 信息时用,二是在写注册机 lmcrypt.exe 时使用  
  
  
2. 找 vendor\_id  
  
要找 vendor\_id 就需要找到 l\_sg 这个函数了(至于为什么要找这个函数建议看一些更基础的文章),在这儿我主要是介绍一下如何  
定位 l\_sg 函数,这个是 FlexLm 的一个内部使用的函数. lc\_init,lc\_checkout 都会调用它  
  
下面是它们的调用关系,括号里面的数字是调用的次数,这个是我从 7.2i 版的 demo.exe 上跟踪得到的结果,其他版本可能略有不同.  
  
a. lc\_init -> l\_init  
b. l\_init -> l\_sg  
  
a. lc\_checkout -> l\_checkout  
b. l\_checkout -> lm\_start\_real(2)  
c. lm\_start\_real -> l\_good\_lic\_key(3)  
d. l\_good\_lic\_key -> l\_sg(2)  
  
从上面可以看到它们的调用关系.   
  
我们可以通过 l\_sg 确定 vendor\_id 和 ENCRYPTION\_SEED,通过 lc\_checkout 可以确定 FEATURE  
  
IDA 通过 FlexLm 的 sig 文件可以确定上面的大部分函数,再通过他们的调用关系,很容易找到其他函数.  
  
制作 FlexLm 的批处理文件和工具包含在附件中.  
  
  
最后定位到 l\_sg 函数  
  
.text:10023C6F C7 45 F0 8E B9 48+                mov     \[ebp+var\_10\], 7648B98Eh 这个数在确定 l\_sg 函数上  
.text:10023C76 C7 45 EC 03 00 00+                mov     \[ebp+var\_14\], 3 也很有用  
.text:10023C7D 8B 45 08                          mov     eax, \[ebp+arg\_0\]  
.text:10023C80 8B 48 6C                          mov     ecx, \[eax+6Ch\]  
.text:10023C83 8B 91 D4 01 00 00                 mov     edx, \[ecx+1D4h\]  
.text:10023C89 81 E2 00 80 00 00                 and     edx, 8000h  
.text:10023C8F 85 D2                             test    edx, edx  
.text:10023C91 74 23                             jz      short loc\_10023CB6  
.text:10023C93 83 3D B0 95 10 10+                cmp     dword\_101095B0, 0  
.text:10023C9A 74 1A                             jz      short loc\_10023CB6  
.text:10023C9C 8B 45 10                          mov     eax, \[ebp+arg\_8\]  
.text:10023C9F 50                                push    eax  
.text:10023CA0 8B 4D 0C                          mov     ecx, \[ebp+arg\_4\]  
.text:10023CA3 51                                push    ecx  
.text:10023CA4 8B 55 08                          mov     edx, \[ebp+arg\_0\]  
.text:10023CA7 52                                push    edx  
.text:10023CA8 FF 15 B0 95 10 10                 call    dword\_101095B0 在这个地方下断  
  
到达断点后按 F8,  
  
在 cmdbar 上输入 dd \[esp+4\]  
  
00B6DE84  003D2084  ASCII "compuware" vendor\_id  
  
输入 dd \[esp+8\]  
  
00B6DFD0  00000004  
00B6DFD4  6F0BDE5B data\[0\]  
00B6DFD8  FE5E8B42 data\[1\]  
00B6DFDC  1B3BD819  
00B6DFE0  F25DBA4E  
00B6DFE4  E6C879BF  
00B6DFE8  54483E7D  
  
输入 dd \[esp\]  
  
003D1F38  00000066  
003D1F3C  00FB009A  
003D1F40  BC87EB00 job+08  
003D1F44  1B1678DF job+0c  
003D1F48  970A62BC job+10  
  
  
3. 计算 ENCRYPTION\_SEED  
  
使用附件中的工具 calcseed.exe,输入上述信息,计算  
  
#define ENCRYPTION\_SEED1 0x15263748  
#define ENCRYPTION\_SEED2 0xf0436413  
  
  
4. 找 Feathure  
  
在 lc\_checkout 处下断 bpx 10015929,按 F8  
在 cmdbar 上输入 dd esp  
  
00B6E538  003D1F38  
00B6E53C  003D5F60  ASCII "DriverStudio" Feathure  
00B6E540  003D5FD8  ASCII "003.000" Version  
  
这个地方可能会中断好几次,这样就可以得到越多的 Feathure,不过我现在只是需要里面的 SoftICE,上面一个完全够用了.  
  
至此,分析已经基本完成,可以使用 SDK 写注册机了  
  
  
  
第三部分: 写注册机  
  
使用附件中的 lmkg.exe,选择 version 7.0,在 vendor\_id 输入"compuware"得到  
  
/\* Version 7 keys \*/  
#define VENDOR\_KEY1 0x9b3aca39  
#define VENDOR\_KEY2 0x1ebd887f  
#define VENDOR\_KEY3 0x3a876975  
#define VENDOR\_KEY4 0x05360fdb  
#define VENDOR\_KEY5 0x0550b591  
#define CRO\_KEY1 0x9f1ac831  
#define CRO\_KEY2 0x589feaf9  
  
#define VENDOR\_NAME "compuware"  
  
编辑 ENCRYPTION\_SEED1,ENCRYPTION\_SEED2,其中 3,4 是 SDK 中的默认值,不需要改动(好像暂时没有什么用)  
  
#define ENCRYPTION\_SEED1 0x15263748  
#define ENCRYPTION\_SEED2 0xf0436413  
#define ENCRYPTION\_SEED3 0x22222222  
#define ENCRYPTION\_SEED4 0x32323232  
  
按照上面的信息,编辑 lm\_code.h,编译 lmcrypt.c 就大功告成了!  
  
  
下面就是使用注册机生成自己注册码了.  
  
先手工编辑一个 license.dat 如下  
  
FEATURE DriverStudio compuware 003.000 permanent uncounted \\  
0 HOSTID=ANY ISSUER=tulipfan ISSUED=2-Sep-2003 \\  
ck=0  
  
在命令行运行 lmcrypt.exe license.dat  
  
恭喜你,一个没有时间次数限制属于你自己的 license 生成了!!!  
  
  
  
第四部分: 感谢  
  
CrackZ  
Nolan Blender  
Oleh Yuschuk  
和所有 CCG 和看学论坛上的朋友们!  

 4dd466d0e1e0cc123afbe504da2462ca