
# Hide Files and Folders V2.5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-152.htm`](https://www.pediy.com/kssd/pediy08/pediy8-152.htm)

【破解日期】 2006 年 5 月 16 日  

【破解作者】 Ryosuke  

【作者邮箱】 没有  

【作者主页】 没有  

【使用工具】 `OD`  

【破解平台】 Windows 95/98/ME/2000/XP  

【软件名称】 Hide Files and Folders V2.5  

【下载地址】 http://www.skycn.com/soft/13171.html  

【软件简介】 Hide Files And Folders 是一个非常容易使用、可以工作在 Windows 内核级别的文件和文件夹加密工具,支持禁止用户访问互联网,可以为任意程序加上密码保护,可以对任意的文件或者文件夹进行加密隐藏保护,非常容易使用!  

【软件大小】 737 KB  

【加壳方式】 无壳  

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

-----------------------------------------------------------------------------

【破解内容】

运行，输入默认密码"admin"，按注册按钮，随便输入注册码，出现错误信息。查找"The registration code is incorrect."，来到

```
004A10F0  /$  55            push    ebp  
004A10F1  |.  8BEC          mov     ebp, esp  
004A10F3  |.  6A 00         push    0  
004A10F5  |.  53            push    ebx  
004A10F6  |.  56            push    esi  
004A10F7  |.  8BF2          mov     esi, edx  
004A10F9  |.  8BD8          mov     ebx, eax  
004A10FB  |.  33C0          xor     eax, eax  
004A10FD  |.  55            push    ebp  
004A10FE  |.  68 6F114A00   push    004A116F  
004A1103  |.  64:FF30       push    dword ptr fs:[eax]  
004A1106  |.  64:8920       mov     fs:[eax], esp  
004A1109  |.  8D45 FC       lea     eax, [ebp-4]  
004A110C  |.  8BCE          mov     ecx, esi  
004A110E  |.  8BD3          mov     edx, ebx  
004A1110  |.  E8 C3FEFFFF   call    004A0FD8      //关键 CALL  
004A1115  |.  84C0          test    al, al  
004A1117  |.  74 40         je      short 004A1159  
004A1119  |.  8B45 FC       mov     eax, [ebp-4]  
004A111C  |.  E8 0BF5FFFF   call    004A062C  
004A1121  |.  84C0          test    al, al  
004A1123  |.  74 1F         je      short 004A1144  
004A1125  |.  8B45 FC       mov     eax, [ebp-4]  
004A1128  |.  E8 6BF8FFFF   call    004A0998  
004A112D  |.  6A 00         push    0                           ; /Arg1 = 00000000  
004A112F  |.  66:8B0D 7C114>mov     cx, [4A117C]               ; |  
004A1136  |.  B2 02         mov     dl, 2                       ; |  
004A1138  |.  B8 88114A00   mov     eax, 004A1188               ; |ASCII "The registration code is correct.",CR," Thank you."  
004A113D  |.  E8 8E10F9FF   call    004321D0                    ; \HFF.004321D0  
004A1142  |.  EB 15         jmp     short 004A1159  
004A1144  |>  6A 00         push    0                           ; /Arg1 = 00000000  
004A1146  |.  66:8B0D 7C114>mov     cx, [4A117C]               ; |  
004A114D  |.  B2 01         mov     dl, 1                       ; |  
004A114F  |.  B8 C0114A00   mov     eax, 004A11C0               ; |ASCII "The registration code is incorrect."  
004A1154  |.  E8 7710F9FF   call    004321D0                    ; \HFF.004321D0  
004A1159  |>  33C0          xor     eax, eax  
004A115B  |.  5A            pop     edx  
004A115C  |.  59            pop     ecx  
004A115D  |.  59            pop     ecx  
004A115E  |.  64:8910       mov     fs:[eax], edx  
004A1161  |.  68 76114A00   push    004A1176  
004A1166  |>  8D45 FC       lea     eax, [ebp-4]  
004A1169  |.  E8 2A35F6FF   call    00404698  
004A116E  \\.  C3           retn  
```

跟进关键的算法 `CALL`

```
004A062C  /$  55            push    ebp  
004A062D  |.  8BEC          mov     ebp, esp  
004A062F  |.  83C4 F0       add     esp, -10  
004A0632  |.  53            push    ebx  
004A0633  |.  56            push    esi  
004A0634  |.  57            push    edi  
004A0635  |.  33D2          xor     edx, edx  
004A0637  |.  8955 F4       mov     [ebp-C], edx  
004A063A  |.  8955 F0       mov     [ebp-10], edx  
004A063D  |.  8945 FC       mov     [ebp-4], eax  
004A0640  |.  8B45 FC       mov     eax, [ebp-4]  
004A0643  |.  E8 0045F6FF   call    00404B48  
004A0648  |.  33C0          xor     eax, eax  
004A064A  |.  55            push    ebp  
004A064B  |.  68 35074A00   push    004A0735  
004A0650  |.  64:FF30       push    dword ptr fs:[eax]  
004A0653  |.  64:8920       mov     fs:[eax], esp  
004A0656  |.  33DB          xor     ebx, ebx  
004A0658  |.  E8 87F8FFFF   call    0049FEE4  
004A065D  |.  8B45 FC       mov     eax, [ebp-4]   
004A0660  |.  E8 F342F6FF   call    00404958  
004A0665  |.  83F8 10       cmp     eax, 10         //注册码长度必须是 16 位  
004A0668  |.  0F85 A4000000 jnz     004A0712  
004A066E  |.  8D45 F4       lea     eax, [ebp-C]  
004A0671  |.  50            push    eax  
004A0672  |.  B9 08000000   mov     ecx, 8  
004A0677  |.  BA 01000000   mov     edx, 1  
004A067C  |.  8B45 FC       mov     eax, [ebp-4]  
004A067F  |.  E8 3445F6FF   call    00404BB8        ;  取注册码前 8 位  
004A0684  |.  8D45 F0       lea     eax, [ebp-10]  
004A0687  |.  50            push    eax  
004A0688  |.  B9 08000000   mov     ecx, 8  
004A068D  |.  BA 09000000   mov     edx, 9  
004A0692  |.  8B45 FC       mov     eax, [ebp-4]  
004A0695  |.  E8 1E45F6FF   call    00404BB8        ;  取注册码后 8 位  
004A069A  |.  8D55 F8       lea     edx, [ebp-8]  
004A069D  |.  8B45 F4       mov     eax, [ebp-C]    ;  前 8 位  
004A06A0  |.  E8 BB2BF6FF   call    00403260  
004A06A5  |.  837D F8 00    cmp     dword ptr [ebp-8], 0  
004A06A9  |.  75 67         jnz     short 004A0712  
004A06AB  |.  8D55 F8       lea     edx, [ebp-8]  
004A06AE  |.  8B45 F0       mov     eax, [ebp-10]  
004A06B1  |.  E8 AA2BF6FF   call    00403260  
004A06B6  |.  837D F8 00    cmp     dword ptr [ebp-8], 0  
004A06BA  |.  75 56         jnz     short 004A0712  
004A06BC  |.  BE 30000000   mov     esi, 30  
004A06C1  |>  33C0          /xor    eax, eax  
004A06C3  |.  8945 F8       |mov    [ebp-8], eax  
004A06C6  |.  8B45 FC       |mov    eax, [ebp-4]  
004A06C9  |.  E8 8A42F6FF   |call   00404958        ;  注册码长度 0x10  
004A06CE  |.  85C0          |test   eax, eax  
004A06D0  |.  7E 17        |jle    short 004A06E9  
004A06D2  |.  BA 01000000  |mov    edx, 1  
004A06D7  |>  8BCE         /mov    ecx, esi  
004A06D9  |.  8B7D FC      ||mov   edi, [ebp-4]  
004A06DC  |.  3A4C17 FF    ||cmp   cl, [edi+edx-1]  
004A06E0  |.  75 03        ||jnz   short 004A06E5  
004A06E2  |.  FF45 F8      ||inc   dword ptr [ebp-8]  
004A06E5  |>  42           ||inc   edx  
004A06E6  |.  48           ||dec   eax  
004A06E7  |.^ 75 EE        |\jnz   short 004A06D7  
004A06E9  |>  837D F8 05   |cmp    dword ptr [ebp-8], 5    //注册码中相同码个数的最大值不能超过 5，  
004A06ED  |.  7E 09        |jle    short 004A06F8  
004A06EF  |.  C745 F8 FFFFF>mov    dword ptr [ebp-8], -1  
004A06F6  |.  EB 06        |jmp    short 004A06FE  
004A06F8  |>  46           |inc    esi  
004A06F9  |.  83FE 3A      |cmp    esi, 3A  
004A06FC  |.^ 75 C3        \|jnz   short 004A06C1  
//上面一段是计算注册码中相同码个数的最大值，这个值不能超过 5，超过了注册码就无效了  
004A06FE  |>  837D F8 00   cmp     dword ptr [ebp-8], 0  
004A0702  |.  7C 0E         jl      short 004A0712  
004A0704  |.  8B45 FC       mov     eax, [ebp-4]  
004A0707  |.  E8 B0FDFFFF   call    004A04BC    //注册码验证核心，跟进  
004A070C  |.  84C0          test    al, al  
004A070E  |.  74 02         je      short 004A0712  
004A0710  |.  B3 01         mov     bl, 1  
004A0712  |>  33C0          xor     eax, eax  
004A0714  |.  5A            pop     edx  
004A0715  |.  59            pop     ecx  
004A0716  |.  59            pop     ecx  
004A0717  |.  64:8910       mov     fs:[eax], edx  
004A071A  |.  68 3C074A00   push    004A073C  
004A071F  |>  8D45 F0       lea     eax, [ebp-10]  
004A0722  |.  BA 02000000   mov     edx, 2  
004A0727  |.  E8 903FF6FF   call    004046BC  
004A072C  |.  8D45 FC       lea     eax, [ebp-4]  
```



## 004A072F . | E8 643FF6FF call 00404698

004A0734 \. C3 retn

### 跟进验证核心 CALL

```
004A04BC  /$  55           push    ebp
004A04BD  |.  8BEC         mov     ebp, esp
004A04BF  |.  B9 06000000  mov     ecx, 6
004A04C4  |>  6A 00        /push    0
004A04C6  |.  6A 00        |push    0
004A04C8  |.  49           |dec     ecx
004A04C9  |.^ 75 F9        \jnz     short 004A04C4
004A04CB  |.  53           push    ebx
004A04CC  |.  56           push    esi
004A04CD  |.  8BD8         mov     ebx, eax
004A04CF  |.  33C0         xor     eax, eax
004A04D1  |.  55           push    ebp
004A04D2  |.  68 1A064A00  push    004A061A
004A04D7  |.  64:FF30      push    dword ptr fs:[eax]
004A04DA  |.  64:8920      mov     fs:[eax], esp
004A04DD  |.  8D45 FC      lea     eax, [ebp-4]
004A04E0  |.  8A13         mov     dl, [ebx]             ;  注册码
004A04E2  |.  E8 9943F6FF  call    00404880        //数字字符转成对应值
004A04E7  |.  8B45 FC      mov     eax, [ebp-4]
004A04EA  |.  E8 BD89F6FF  call    00408EAC
004A04EF  |.  8BF0         mov     esi, eax
004A04F1  |.  8D45 F8      lea     eax, [ebp-8]
004A04F4  |.  8A53 01      mov     dl, [ebx+1]
004A04F7  |.  E8 8443F6FF  call    00404880
004A04FC  |.  8B45 F8      mov     eax, [ebp-8]
004A04FF  |.  E8 A889F6FF  call    00408EAC
004A0504  |.  03F0         add     esi, eax
004A0506  |.  8D45 F4      lea     eax, [ebp-C]
004A0509  |.  8A53 02      mov     dl, [ebx+2]
004A050C  |.  E8 6F43F6FF  call    00404880
004A0511  |.  8B45 F4      mov     eax, [ebp-C]
004A0514  |.  E8 9389F6FF  call    00408EAC
004A0519  |.  03F0         add     esi, eax
004A051B  |.  8D45 F0      lea     eax, [ebp-10]
004A051E  |.  8A53 03      mov     dl, [ebx+3]
004A0521  |.  E8 5A43F6FF  call    00404880
004A0526  |.  8B45 F0      mov     eax, [ebp-10]
004A0529  |.  E8 7E89F6FF  call    00408EAC
004A052E  |.  03F0         add     esi, eax
004A0530  |.  A1 9C164B00  mov     eax, [4B169C]
004A0535  |.  3B30         cmp     esi, [eax]             ;  [eax]=0x16,注册码前 4 位和为 0x16
004A0537  |.  0F85 BA000000 jnz     004A05F7
004A053D  |.  8D45 EC      lea     eax, [ebp-14]
004A0540  |.  8A53 04      mov     dl, [ebx+4]
004A0543  |.  E8 3843F6FF  call    00404880
004A0548  |.  8B45 EC      mov     eax, [ebp-14]
004A054B  |.  E8 5C89F6FF  call    00408EAC
004A0550  |.  8BF0         mov     esi, eax
004A0552  |.  8D45 E8      lea     eax, [ebp-18]
004A0555  |.  8A53 07      mov     dl, [ebx+7]
004A0558  |.  E8 2343F6FF  call    00404880
004A055D  |.  8B45 E8      mov     eax, [ebp-18]
004A0560  |.  E8 4789F6FF  call    00408EAC
004A0565  |.  03F0         add     esi, eax
004A0567  |.  8D45 E4      lea     eax, [ebp-1C]
004A056A  |.  8A53 0A      mov     dl, [ebx+A]
004A056D  |.  E8 0E43F6FF  call    00404880
004A0572  |.  8B45 E4      mov     eax, [ebp-1C]
004A0575  |.  E8 3289F6FF  call    00408EAC
004A057A  |.  03F0         add     esi, eax
004A057C  |.  8D45 E0      lea     eax, [ebp-20]
004A057F  |.  8A53 0D      mov     dl, [ebx+D]
004A0582  |.  E8 F942F6FF  call    00404880
004A0587  |.  8B45 E0      mov     eax, [ebp-20]
004A058A  |.  E8 1D89F6FF  call    00408EAC
004A058F  |.  03F0         add     esi, eax
004A0591  |.  A1 D4164B00  mov     eax, [4B16D4]
004A0596  |.  3B30         cmp     esi, [eax]             ;  [eax]=0x1e,注册码 5,8,11,14 位的和必须 0x1E
004A0598  |.  75 5D        jnz     short 004A05F7
004A059A  |.  8D45 DC      lea     eax, [ebp-24]
004A059D  |.  8A53 06      mov     dl, [ebx+6]
004A05A0  |.  E8 DB42F6FF  call    00404880
004A05A5  |.  8B45 DC      mov     eax, [ebp-24]
004A05A8  |.  E8 FF88F6FF  call    00408EAC
004A05AD  |.  8BF0         mov     esi, eax
004A05AF  |.  8D45 D8      lea     eax, [ebp-28]
004A05B2  |.  8A53 09      mov     dl, [ebx+9]
004A05B5  |.  E8 C642F6FF  call    00404880
004A05BA  |.  8B45 D8      mov     eax, [ebp-28]
004A05BD  |.  E8 EA88F6FF  call    00408EAC
004A05C2  |.  03F0         add     esi, eax
004A05C4  |.  8D45 D4      lea     eax, [ebp-2C]
004A05C7  |.  8A53 0C      mov     dl, [ebx+C]
004A05CA  |.  E8 B142F6FF  call    00404880
004A05CF  |.  8B45 D4      mov     eax, [ebp-2C]
004A05D2  |.  E8 D588F6FF  call    00408EAC
004A05D7  |.  03F0         add     esi, eax
004A05D9  |.  8D45 D0      lea     eax, [ebp-30]
004A05DC  |.  8A53 0F      mov     dl, [ebx+F]
004A05DF  |.  E8 9C42F6FF  call    00404880
004A05E4  |.  8B45 D0      mov     eax, [ebp-30]
004A05E7  |.  E8 C088F6FF  call    00408EAC
004A05EC  |.  03F0         add     esi, eax
004A05EE  |.  A1 B4134B00  mov     eax, [4B13B4]
004A05F3  |.  3B30         cmp     esi, [eax]             ;  [eax]=0x08,注册码 7,10,13,16 位的和必须 0x8
004A05F5  |.  74 04        je      short 004A05FB
004A05F7  |>  33C0         xor     eax, eax
004A05F9  |.  EB 02        jmp     short 004A05FD
004A05FB  |>  B0 01        mov     al, 1        ;设置成功标志
004A05FD  |>  8BD8         mov     ebx, eax
004A05FF  |.  33C0         xor     eax, eax
004A0601  |.  5A           pop     edx
004A0602  |.  59           pop     ecx
004A0603  |.  59           pop     ecx
004A0604  |.  64:8910      mov     fs:[eax], edx
004A0607  |.  68 21064A00  push    004A0621
004A060C  |>  8D45 D0      lea     eax, [ebp-30]
004A060F  |.  BA 0C000000  mov     edx, 0C
004A0614  |.  E8 A340F6FF  call    004046BC
004A0619  \.  C3           retn
```

验证程序没有验证 `6`，`9`，`12`，`15` 位，这些位任意。

///////////////////////////////////////////////////////////////////////////////////////////

### 注册机代码

#### 生成所有的注册码

```c
// KeyGen.cpp : Defines the entry point for the console application.

//Code By NightFox 2006.05.16

#include "stdafx.h"

#include <STDIO.H>

#include <WINDOWS.H>

int main(int argc, char* argv[])
{
  BYTE serial[16];
  char serialstr[17];
  int a,b,c,d;
  int i;
  int count,maxcount;
  for(a=0;a<=9;a++)
    for(b=0;b<=9;b++)
      for(c=0;c<=9;c++)
        for(d=0;d<=9;d++)
        {
          if((a+b+c+d)==0x16)
          {
            serial[0]=a;
            serial[1]=b;
            serial[2]=c;
            serial[3]=d;
          }
          if((a+b+c+d)==0x1e)
          {
            serial[0x4]=a;
            serial[0x7]=b;
            serial[0xA]=c;
            serial[0xD]=d;
            break;
          }
          if((a+b+c+d)==0x8)
          {
            serial[0x6]=a;
            serial[0x9]=b;
            serial[0xC]=c;
            serial[0xF]=d;
          }
          serial[0x5]=a;
          serial[0x8]=b;
          serial[0xB]=c;
          serial[0xE]=d;
          if((serial[0x0]+serial[0x1]+serial[0x2]+serial[0x3])==0x16
            &&(serial[0x4]+serial[0x7]+serial[0xA]+serial[0xD])==0x1e
            &&(serial[0x6]+serial[0x9]+serial[0xC]+serial[0xF])==0x08)
          {       
            maxcount=0;
            for(i=0;i<=9;i++)
            {
              count=0;
              for(int j=0;j<16;j++)
              {
                if(serial[j]==i)
                  count++;
              }
              maxcount=count>maxcount?count:maxcount;
            }
            if(maxcount<=5)
            {
              sprintf(serialstr,"%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d%01d",
```



`serial[0]`, `serial[1]`, `serial[2]`, `serial[3]`, `serial[4]`, `serial[5]`, `serial[6]`, `serial[7]`,  
`serial[8]`, `serial[9]`, `serial[10]`, `serial[11]`, `serial[12]`, `serial[13]`, `serial[14]`, `serial[15]`);  
`printf("%s\n", serialstr);`  
`//system("pause");`  
`}`  
`}`  
`}`  
`system("pause");`  
`return 0;`  
`}`

算法很简单，就不多说了。

`/`

`-`

**【破解总结】**

谢谢你能看到这里。

`-`

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

分析得不错，但若真要穷举，似乎要 16 层循环![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg) ![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg) ![](img/dd27b7eb3e47f4f9a8b69ac3c32485f5.jpg) 

```
代码:

* * *

void main()
{int serial[16];
 int a,b,c,d,a1,a2,a3,b1,b2,b3,c1,c2,c3,d1,d2,d3;
 for(a=0;a<=8;a++) //sum=8（9 排除）
 for(b=0;b<=8;b++)
 for(c=0;c<=8;c++)
 for(d=0;d<=8;d++)
 if((a+b+c+d)==8)
 {serial[6]=a,serial[9]=b,serial[12]=c,serial[15]=d;
  for(a1=3;a1<=9;a1++) //sum=30（0,1,2 排除）
  for(b1=3;b1<=9;b1++)
  for(c1=3;c1<=9;c1++)
  for(d1=3;d1<=9;d1++)
  if((a1+b1+c1+d1)==30)
  {serial[4]=a1,serial[7]=b1,serial[10]=c1,serial[13]=d1;
   for(a2=0;a2<=9;a2++)
   for(b2=0;b2<=9;b2++)
   for(c2=0;c2<=9;c2++)
   for(d2=0;d2<=9;d2++)
   if((a2+b2+c2+d2)==22)
   {serial[0]=a2,serial[1]=b2,serial[2]=c2,serial[3]=d2;
    for(a3=0;a3<=9;a3++)
    for(b3=0;b3<=9;b3++)
    for(c3=0;c3<=9;c3++)
    for(d3=0;d3<=9;d3++)
    {serial[5]=a3,serial[8]=b3,serial[11]=c3,serial[14]=d3;
     int maxcount=0;
     for(int m=0;m<=9;m++)
     {int count=0;
      for(int n=0;n<16;n++)
      {if(serial[n]==m)
       count++;
      }
      maxcount=count>maxcount?count:maxcount;
     }
     if(maxcount<=5)
       {for(int i=0;i<16;i++) printf("%1d",serial[i]);
        printf("\n");
        //system("pause");
       }
     }
    }
   }
  }
 }

* * *
```
