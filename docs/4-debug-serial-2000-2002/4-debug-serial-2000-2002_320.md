
# fileaxe v1.00

> 原文：[`www.pediy.com/kssd/pediy03/forum1004.htm`](https://www.pediy.com/kssd/pediy03/forum1004.htm)

文件斧头 `fileaxe.exe` v1.00

crack by 6767

工具：只用 WDASM 就能搞定

说明：文件切割器，是否好用不详，刚下载的

地址：`ftp://211.100.8.153/download/99/10261_axefb.zip`

比较恶，未注册版本更改浏览器默认页。

先脱壳，反编译，找出错信息：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00466AD3(C), :00466AE4(C)
* Possible StringData Ref from Code Obj ->"错误的序列号！"
:00466D48 B8E86D4600        mov eax, 00466DE8
:00466D4D E80255FeFF        call 0044C254
```

向上找出错的跳转：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00466A8F(C)
:00466ACF 837DF800          cmp dword ptr [ebp-08], 00000000    <-这里判断注册码是否为空
:00466AD3 0F846F020000      je 00466D48
:00466AD9 8B45F8            mov eax, dword ptr [ebp-08]
:00466ADC 8B55F0            mov edx, dword ptr [ebp-10]    <-正确注册码的位置
:00466ADF E8A0D2F9FF        call 00403D84
:00466AE4 0F855E020000      jne 00466D48
:00466AEA 8D9504FEFFFF      lea edx, dword ptr [ebp+FFFFFE04]
:00466AF0 33C0              xor eax, eax
:00466AF2 E8CDBDF9FF        call 004028C4
:00466AF7 8B8504FEFFFF      mov eax, dword ptr [ebp+FFFFFE04]
:00466AFD 8D55EC            lea edx, dword ptr [ebp-14]
:00466B00 E8AB16FAFF        call 004081B0
:00466B05 8D8500FEFFFF      lea eax, dword ptr [ebp+FFFFFE00]
* Possible StringData Ref from Code Obj ->"fileaxe.ini"
:00466B0B B9C46D4600        mov ecx, 00466DC4
```

你的注册码可以用 `SI` 跟出来，下网后再跟吧。

## 文件加密工具 fedt V1.28

crack by 6767

工具：只用 WDASM 就能搞定

说明：文件加密工具，效果未知

地址：`ftp://211.100.8.153/download/142/10284_fedtw128.exe`

先用 `upx` 解压，反编译，找出错信息：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:0040955F(C)
:00409636 A154634900        mov eax, dword ptr [00496354]
:0040963B 6A00              push 00000000
* Possible StringData Ref from Data Obj ->"错误信息！"
:0040963D B910E74800        mov ecx, 0048E710
* Possible StringData Ref from Data Obj ->"非法用户名或注册码！"
:00409642 BAFBE64800        mov edx, 0048E6FB
:00409647 8B00              mov eax, dword ptr [eax]
:00409649 E866D20600        call 004768B4
```

向上找出错的跳转：

```
:00409540 8D55EC            lea edx, dword ptr [ebp-14]    <-
:00409543 8D45F8            lea eax, dword ptr [ebp-08]    <-
:00409546 E8C5D40600        call 00476A10
:0040954B 50                push eax    <-出口值进栈
:0040954C FF4DD8            dec [ebp-28]
:0040954F 8D45EC            lea eax, dword ptr [ebp-14]
:00409552 BA02000000        mov edx, 00000002
:00409557 E8E4D30600        call 00476940
:0040955C 59                pop ecx    <-弹出 FLAG
:0040955D 84C9              test cl, cl
:0040955F 0F84D1000000      je 00409636    <-为 0 则 GAMEOVER
```

由于它根据你的机器生成一个与注册注册有关的 `ID` 号，正确的处理方法是暴掉 `call 00476A10` 的出口 `EAX` 值，使其等于 1。

```
* Referenced by a CALL at Addresses:
:0040485B  , :00409546
:00476A10 55                push ebp
:00476A11 8BEC              mov ebp, esp
:00476A13 53                push ebx
:00476A14 8B00              mov eax, dword ptr [eax]
:00476A16 8B12              mov edx, dword ptr [edx]
:00476A18 E89F1EFFFF        call 004688BC
:00476A1D 0F94C0            sete al    <-共 6 字节空间，怎么改由你
:00476A20 83E001            and eax, 00000001    <-
:00476A23 5B                pop ebx
:00476A24 5D                pop ebp
:00476A25 C3                ret
```

可以作 LOADER 或内存补丁，解决大家面临的问题。

2 分钟搞定，下面要怎样做取决于你自己，当然你也可以去注册:-）

```c
#include "stdio.h"

main()
{
char string[28],string1[8];

int i,d,e,j,f;

printf(" \*\*\*\*\*\*\*fedt 1.27\*\*\*\*\*\*\*\*\\nThis keygen is made by Pretender\\nPlease input your name  : ");

gets(string);

printf("Please input your ID    : ");

gets(string1);

printf("Your Register code is  : ");

strcat(string,string1);

d=strlen(string);

string[d]=0;

for(i=0;i<8;i++)
  {for(j=0;;j++)
  {e=string[i];e+=i;e=e^i;
    f=string[d-i];f+=i;f+=(i*i);f-=(j*j);f-=d;f=f&0xff;
    e=e^f;
    if(e>=48&&e<=57) break;}
    printf("%d",e-48);}

printf("\\n \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      ---    ---    ---\\n \*Welcome to WWW.CRACKNOW.COM\*      /  4dd466d0e1e0cc123afbe504da2462ca
```
