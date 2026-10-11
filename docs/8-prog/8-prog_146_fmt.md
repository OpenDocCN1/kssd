
# 来个简单题目，看哪位兄弟写的最好！

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-590.htm`](https://www.pediy.com/kssd/pediy09/pediy09-590.htm)

> 很简单。一个整型数据。

> 不使用循环，不使用 `If`，将其逆序输出。

> 例如：

> 输入：`124578`

> 输出：`875421`

> 不知道能实现么，兄弟们讨论下！

> 那天看了 `shoooo` 回的一个逆序输出的题目，

> 有点启发，后来想这个题目用类似的思路作，

> 试了下，好像不行！

> 题目要求输入是整型，最大长度有限制，那就可以玩体力活了

> 负数的不要了

```c
#include <stdio.h>
#include <math.h>

void IntRev(int x, char *szOut)
{
  int n;

  n = (int)log10(x);
  szOut[0] = x%10 + 0x30;
  x /= 10;
  szOut[1] = x%10 + 0x30;
  x /= 10;
  szOut[2] = x%10 + 0x30;
  x /= 10;
  szOut[3] = x%10 + 0x30;
  x /= 10;
  szOut[4] = x%10 + 0x30;
  x /= 10;
  szOut[5] = x%10 + 0x30;
  x /= 10;
  szOut[6] = x%10 + 0x30;
  x /= 10;
  szOut[7] = x%10 + 0x30;
  x /= 10;
  szOut[8] = x%10 + 0x30;
  x /= 10;
  szOut[9] = x%10 + 0x30;
  szOut[n+1] = 0;
}

int main()
{
  char szOut[20];
  int x = 2147483647;

  IntRev(x, szOut);

  printf(szOut);

  return 0;
}
```

> `VC6`，位数不限（不过当然要求输入和输出值都小于 2 的 32 次方），从最高位开始转换

```c
#pragma optimize("",off)

unsigned int reva(unsigned int a)
{
  unsigned int f2=1000000000;
  unsigned int f1=1;
  return (((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10))&&((!(a/f2))&&(f2/=10)&&(f1*=10)))*0+((f1=f2)*0+(f2=1)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1);
}

int main(int argc, char* argv[])
{
  printf("%u %u %u\r\n",reva(1234567893),reva(12345),reva(0));
  return 0;
}
```

> 输出：

```
3987654321 54321 0
Press any key to continue
```

> 我想出一个方法:

> 整数:

> 代码:

```c
#include<stdio.h>
int revd(unsigned u)
{
  return u&&(printf("%u",u%10),revd(u/10));
}
int main()
{
  return revd(123456789);
}
```

> 字符串:

> 代码:

```c
#include<stdio.h>
int revs(char *p)
{
  return *p&&printf("%c",*p,revs(p+1));
}
int main()
{
  return revs("123456789");
}
```

> 也许还有更简单的吧...

> 最后给出一个直接返回整数的版本:

> 代码:

```c
unsigned revd(unsigned u, unsigned t=0)
{
  return (!(u&&!(t=revd(u/10,t*10+u%10))))*t;
}
int main()
{
  return printf("%u\n",revd(123456789));
}
```

> 回楼上，`push/pop` 本质上只是数据传送语句，和 `mov[esp],X;add esp,4;` 并无不同

> 用`&&`跳过执行，在 `ASM` 里仍有跳转语句，改进一下：

```c
#pragma optimize("",off)

unsigned int reva(unsigned int a)
{
  unsigned int f2=1000000000;
  unsigned int f1=1;
  return ((f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f2/=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1))+(f1*=((((a+(a-1)/(-1))/f2-1)/(-1))*9+1)))*0+((f1=f2)*0+(f2=1)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1+(f1/=10)*0+(f2*=10)*0+((a/f2)%10)*f1);
}

int main(int argc, char* argv[])
{
  printf("%u %u %u %u %u\r\n",reva(1234567893),reva(12345),reva(1),reva(4294967293),reva(0));
  return 0;
}
```

> 输出：

```
3987654321 54321 1 3927694924 0
Press any key to continue
```

> 纯算术运算，在汇编里没有跳转：

```asm
.text:00401000 sub_401000      proc near               ; CODE XREF: _main+5p
.text:00401000                                             ; _main+10p ...
.text:00401000
.text:00401000 f1              = dword ptr -8
.text:00401000 f2              = dword ptr -4
.text:00401000 a               = dword ptr  8
.text:00401000
.text:00401000                 push    ebp
.text:00401001                 mov     ebp, esp
.text:00401003                 sub     esp, 8
.text:00401006                 push    esi
.text:00401007                 mov     [ebp+f2], 3B9ACA00h
.text:0040100E                 mov     [ebp+f1], 1
.text:00401015                 mov     eax, [ebp+a]
.text:00401018                 sub     eax, 1
.text:0040101B                 xor     edx, edx
.text:0040101D                 or      ecx, 0FFFFFFFFh
.text:00401020                 div     ecx
.text:00401022                 mov     edx, [ebp+a]
.text:00401025                 add     eax, edx
.text:00401027                 xor     edx, edx
.text:00401029                 div     [ebp+f2]
.text:0040102C                 sub     eax, 1
.text:0040102F                 xor     edx, edx
.text:00401031                 or      ecx, 0FFFFFFFFh
.text:00401034                 div     ecx
.text:00401036                 mov     ecx, eax
.text:00401038                 imul    ecx, 9
.text:0040103B                 add     ecx, 1
.text:0040103E                 mov     eax, [ebp+f2]
.text:00401041                 xor     edx, edx
.text:00401043                 div     ecx
.text:00401045                 mov     [ebp+f2], eax
.text:00401048                 mov     eax, [ebp+a]
.text:0040104B                 sub     eax, 1
.text:0040104E                 xor     edx, edx
.text:00401050                 or      ecx, 0FFFFFFFFh
.text:00401053                 div     ecx
.text:00401055                 mov     edx, [ebp+a]
.text:00401058                 add     eax, edx
.text:0040105A                 xor     edx, edx
.text:0040105C                 div     [ebp+f2]
.text:0040105F                 sub     eax, 1
.text:00401062                 xor     edx, edx
.text:00401064                 or      ecx, 0FFFFFFFFh
.text:00401067                 div     ecx
.text:00401069                 imul    eax, 9
.text:0040106C                 add     eax, 1
.text:0040106F                 mov     edx, [ebp+f1]
.text:00401072                 imul    edx, eax
.text:00401075                 mov     [ebp+f1], edx
.text:00401078                 mov     eax, [ebp+a]
.text:0040107B                 sub     eax, 1
.text:0040107E                 xor     edx, edx
.text:00401080                 or      ecx, 0FFFFFFFFh
.text:00401083                 div     ecx
.text:00401085                 mov     edx, [ebp+a]
.text:00401088                 add     eax, edx
.text:0040108A                 xor     edx, edx
```



```
.text:0040108C             div     [ebp+f2]  
.text:0040108F             sub     eax, 1  
.text:00401092             xor     edx, edx  
.text:00401094             or      ecx, 0FFFFFFFFh  
.text:00401097             div     ecx  
.text:00401099             mov     ecx, eax  
.text:0040109B             imul    ecx, 9  
.text:0040109E             add     ecx, 1  
.text:004010A1             mov     eax, [ebp+f2]  
.text:004010A4             xor     edx, edx  
.text:004010A6             div     ecx  
.text:004010A8             mov     [ebp+f2], eax  
.text:004010AB             mov     eax, [ebp+a]  
.text:004010AE             sub     eax, 1  
.text:004010B1             xor     edx, edx  
.text:004010B3             or      ecx, 0FFFFFFFFh  
.text:004010B6             div     ecx  
.text:004010B8             mov     edx, [ebp+a]  
.text:004010BB             add     eax, edx  
.text:004010BD             xor     edx, edx  
.text:004010BF             div     [ebp+f2]  
.text:004010C2             sub     eax, 1  
.text:004010C5             xor     edx, edx  
.text:004010C7             or      ecx, 0FFFFFFFFh  
.text:004010CA             div     ecx  
.text:004010CC             imul    eax, 9  
.text:004010CF             add     eax, 1  
.text:004010D2             mov     edx, [ebp+f1]  
.text:004010D5             imul    edx, eax  
.text:004010D8             mov     [ebp+f1], edx  
.text:004010DB             mov     eax, [ebp+a]  
.text:004010DE             sub     eax, 1  
.text:004010E1             xor     edx, edx  
.text:004010E3             or      ecx, 0FFFFFFFFh  
.text:004010E6             div     ecx  
.text:004010E8             mov     edx, [ebp+a]  
.text:004010EB             add     eax, edx  
.text:004010ED             xor     edx, edx  
.text:004010EF             div     [ebp+f2]  
.text:004010F2             sub     eax, 1  
.text:004010F5             xor     edx, edx  
.text:004010F7             or      ecx, 0FFFFFFFFh  
.text:004010FA             div     ecx  
.text:004010FC             mov     ecx, eax  
.text:004010FE             imul    ecx, 9  
.text:00401101             add     ecx, 1  
.text:00401104             mov     eax, [ebp+f2]  
.text:00401107             xor     edx, edx  
.text:00401109             div     ecx  
.text:0040110B             mov     [ebp+f2], eax  
.text:0040110E             mov     eax, [ebp+a]  
.text:00401111             sub     eax, 1  
.text:00401114             xor     edx, edx  
.text:00401116             or      ecx, 0FFFFFFFFh  
.text:00401119             div     ecx  
.text:0040111B             mov     edx, [ebp+a]  
.text:0040111E             add     eax, edx  
.text:00401120             xor     edx, edx  
.text:00401122             div     [ebp+f2]  
.text:00401125             sub     eax, 1  
.text:00401128             xor     edx, edx  
.text:0040112A             or      ecx, 0FFFFFFFFh  
.text:0040112D             div     ecx  
.text:0040112F             imul    eax, 9  
.text:00401132             add     eax, 1  
.text:00401135             mov     edx, [ebp+f1]  
.text:00401138             imul    edx, eax  
.text:0040113B             mov     [ebp+f1], edx  
.text:0040113E             mov     eax, [ebp+a]  
.text:00401141             sub     eax, 1  
.text:00401144             xor     edx, edx  
.text:00401146             or      ecx, 0FFFFFFFFh  
.text:00401149             div     ecx  
.text:0040114B             mov     edx, [ebp+a]  
.text:0040114E             add     eax, edx  
.text:00401150             xor     edx, edx  
.text:00401152             div     [ebp+f2]  
.text:00401155             sub     eax, 1  
.text:00401158             xor     edx, edx  
.text:0040115A             or      ecx, 0FFFFFFFFh  
.text:0040115D             div     ecx  
.text:0040115F             mov     ecx, eax  
.text:00401161             imul    ecx, 9  
.text:00401164             add     ecx, 1  
.text:00401167             mov     eax, [ebp+f2]  
.text:0040116A             xor     edx, edx  
.text:0040116C             div     ecx  
.text:0040116E             mov     [ebp+f2], eax  
.text:00401171             mov     eax, [ebp+a]  
.text:00401174             sub     eax, 1  
.text:00401177             xor     edx, edx  
.text:00401179             or      ecx, 0FFFFFFFFh  
.text:0040117C             div     ecx  
.text:0040117E             mov     edx, [ebp+a]  
.text:00401181             add     eax, edx  
.text:00401183             xor     edx, edx  
.text:00401185             div     [ebp+f2]  
.text:00401188             sub     eax, 1  
.text:0040118B             xor     edx, edx  
.text:0040118D             or      ecx, 0FFFFFFFFh  
.text:00401190             div     ecx  
.text:00401192             imul    eax, 9  
.text:00401195             add     eax, 1  
.text:00401198             mov     edx, [ebp+f1]  
.text:0040119B             imul    edx, eax  
.text:0040119E             mov     [ebp+f1], edx  
.text:004011A1             mov     eax, [ebp+a]  
.text:004011A4             sub     eax, 1  
.text:004011A7             xor     edx, edx  
.text:004011A9             or      ecx, 0FFFFFFFFh  
.text:004011AC             div     ecx  
.text:004011AE             mov     edx, [ebp+a]  
.text:004011B1             add     eax, edx  
.text:004011B3             xor     edx, edx  
.text:004011B5             div     [ebp+f2]  
.text:004011B8             sub     eax, 1  
.text:004011BB             xor     edx, edx  
.text:004011BD             or      ecx, 0FFFFFFFFh  
.text:004011C0             div     ecx  
.text:004011C2             mov     ecx, eax  
.text:004011C4             imul    ecx, 9  
.text:004011C7             add     ecx, 1  
.text:004011CA             mov     eax, [ebp+f2]  
.text:004011CD             xor     edx, edx  
.text:004011CF             div     ecx  
.text:004011D1             mov     [ebp+f2], eax  
.text:004011D4             mov     eax, [ebp+a]  
.text:004011D7             sub     eax, 1  
.text:004011DA             xor     edx, edx  
.text:004011DC             or      ecx, 0FFFFFFFFh  
.text:004011DF             div     ecx  
.text:004011E1             mov     edx, [ebp+a]  
.text:004011E4             add     eax, edx  
.text:004011E6             xor     edx, edx  
.text:004011E8             div     [ebp+f2]  
.text:004011EB             sub     eax, 1  
.text:004011EE             xor     edx, edx  
.text:004011F0             or      ecx, 0FFFFFFFFh  
.text:004011F3             div     ecx  
.text:004011F5             imul    eax, 9  
.text:004011F8             add     eax, 1  
.text:004011FB             mov     edx, [ebp+f1]  
.text:004011FE             imul    edx, eax  
.text:00401201             mov     [ebp+f1], edx  
.text:00401204             mov     eax, [ebp+a]  
.text:00401207             sub     eax, 1  
.text:0040120A             xor     edx, edx  
```



```
.text:0040120C                    or      ecx, 0FFFFFFFFh
.text:0040120F                    div     ecx
.text:00401211                    mov     edx, [ebp+a]
.text:00401214                    add     eax, edx
.text:00401216                    xor     edx, edx
.text:00401218                    div     [ebp+f2]
.text:0040121B                    sub     eax, 1
.text:0040121E                    xor     edx, edx
.text:00401220                    or      ecx, 0FFFFFFFFh
.text:00401223                    div     ecx
.text:00401225                    mov     ecx, eax
.text:00401227                    imul    ecx, 9
.text:0040122A                    add     ecx, 1
.text:0040122D                    mov     eax, [ebp+f2]
.text:00401230                    xor     edx, edx
.text:00401232                    div     ecx
.text:00401234                    mov     [ebp+f2], eax
.text:00401237                    mov     eax, [ebp+a]
.text:0040123A                    sub     eax, 1
.text:0040123D                    xor     edx, edx
.text:0040123F                    or      ecx, 0FFFFFFFFh
.text:00401242                    div     ecx
.text:00401244                    mov     edx, [ebp+a]
.text:00401247                    add     eax, edx
.text:00401249                    xor     edx, edx
.text:0040124B                    div     [ebp+f2]
.text:0040124E                    sub     eax, 1
.text:00401251                    xor     edx, edx
.text:00401253                    or      ecx, 0FFFFFFFFh
.text:00401256                    div     ecx
.text:00401258                    imul    eax, 9
.text:0040125B                    add     eax, 1
.text:0040125E                    mov     edx, [ebp+f1]
.text:00401261                    imul    edx, eax
.text:00401264                    mov     [ebp+f1], edx
.text:00401267                    mov     eax, [ebp+a]
.text:0040126A                    sub     eax, 1
.text:0040126D                    xor     edx, edx
.text:0040126F                    or      ecx, 0FFFFFFFFh
.text:00401272                    div     ecx
.text:00401274                    mov     edx, [ebp+a]
.text:00401277                    add     eax, edx
.text:00401279                    xor     edx, edx
.text:0040127B                    div     [ebp+f2]
.text:0040127E                    sub     eax, 1
.text:00401281                    xor     edx, edx
.text:00401283                    or      ecx, 0FFFFFFFFh
.text:00401286                    div     ecx
.text:00401288                    mov     ecx, eax
.text:0040128A                    imul    ecx, 9
.text:0040128D                    add     ecx, 1
.text:00401290                    mov     eax, [ebp+f2]
.text:00401293                    xor     edx, edx
.text:00401295                    div     ecx
.text:00401297                    mov     [ebp+f2], eax
.text:0040129A                    mov     eax, [ebp+a]
.text:0040129D                    sub     eax, 1
.text:004012A0                    xor     edx, edx
.text:004012A2                    or      ecx, 0FFFFFFFFh
.text:004012A5                    div     ecx
.text:004012A7                    mov     edx, [ebp+a]
.text:004012AA                    add     eax, edx
.text:004012AC                    xor     edx, edx
.text:004012AE                    div     [ebp+f2]
.text:004012B1                    sub     eax, 1
.text:004012B4                    xor     edx, edx
.text:004012B6                    or      ecx, 0FFFFFFFFh
.text:004012B9                    div     ecx
.text:004012BB                    imul    eax, 9
.text:004012BE                    add     eax, 1
.text:004012C1                    mov     edx, [ebp+f1]
.text:004012C4                    imul    edx, eax
.text:004012C7                    mov     [ebp+f1], edx
.text:004012CA                    mov     eax, [ebp+a]
.text:004012CD                    sub     eax, 1
.text:004012D0                    xor     edx, edx
.text:004012D2                    or      ecx, 0FFFFFFFFh
.text:004012D5                    div     ecx
.text:004012D7                    mov     edx, [ebp+a]
.text:004012DA                    add     eax, edx
.text:004012DC                    xor     edx, edx
.text:004012DE                    div     [ebp+f2]
.text:004012E1                    sub     eax, 1
.text:004012E4                    xor     edx, edx
.text:004012E6                    or      ecx, 0FFFFFFFFh
.text:004012E9                    div     ecx
.text:004012EB                    mov     ecx, eax
.text:004012ED                    imul    ecx, 9
.text:004012F0                    add     ecx, 1
.text:004012F3                    mov     eax, [ebp+f2]
.text:004012F6                    xor     edx, edx
.text:004012F8                    div     ecx
.text:004012FA                    mov     [ebp+f2], eax
.text:004012FD                    mov     eax, [ebp+a]
.text:00401300                    sub     eax, 1
.text:00401303                    xor     edx, edx
.text:00401305                    or      ecx, 0FFFFFFFFh
.text:00401308                    div     ecx
.text:0040130A                    mov     edx, [ebp+a]
.text:0040130D                    add     eax, edx
.text:0040130F                    xor     edx, edx
.text:00401311                    div     [ebp+f2]
.text:00401314                    sub     eax, 1
.text:00401317                    xor     edx, edx
.text:00401319                    or      ecx, 0FFFFFFFFh
.text:0040131C                    div     ecx
.text:0040131E                    imul    eax, 9
.text:00401321                    add     eax, 1
.text:00401324                    mov     edx, [ebp+f1]
.text:00401327                    imul    edx, eax
.text:0040132A                    mov     [ebp+f1], edx
.text:0040132D                    mov     eax, [ebp+a]
.text:00401330                    sub     eax, 1
.text:00401333                    xor     edx, edx
.text:00401335                    or      ecx, 0FFFFFFFFh
.text:00401338                    div     ecx
.text:0040133A                    mov     edx, [ebp+a]
.text:0040133D                    add     eax, edx
.text:0040133F                    xor     edx, edx
.text:00401341                    div     [ebp+f2]
.text:00401344                    sub     eax, 1
.text:00401347                    xor     edx, edx
.text:00401349                    or      ecx, 0FFFFFFFFh
.text:0040134C                    div     ecx
.text:0040134E                    mov     ecx, eax
.text:00401350                    imul    ecx, 9
.text:00401353                    add     ecx, 1
.text:00401356                    mov     eax, [ebp+f2]
.text:00401359                    xor     edx, edx
.text:0040135B                    div     ecx
.text:0040135D                    mov     [ebp+f2], eax
.text:00401360                    mov     eax, [ebp+a]
.text:00401363                    sub     eax, 1
.text:00401366                    xor     edx, edx
.text:00401368                    or      ecx, 0FFFFFFFFh
.text:0040136B                    div     ecx
.text:0040136D                    mov     edx, [ebp+a]
.text:00401370                    add     eax, edx
.text:00401372                    xor     edx, edx
.text:00401374                    div     [ebp+f2]
.text:00401377                    sub     eax, 1
.text:0040137A                    xor     edx, edx
.text:0040137C                    or      ecx, 0FFFFFFFFh
.text:0040137F                    div     ecx
.text:00401381                    imul    eax, 9
.text:00401384                    add     eax, 1
.text:00401387                    mov     edx, [ebp+f1]
```



```asm
.text:0040138A             imul    edx, eax
.text:0040138D             mov     [ebp+f1], edx
.text:00401390             mov     eax, [ebp+a]
.text:00401393             sub     eax, 1
.text:00401396             xor     edx, edx
.text:00401398             or      ecx, 0FFFFFFFFh
.text:0040139B             div     ecx
.text:0040139D             mov     edx, [ebp+a]
.text:004013A0             add     eax, edx
.text:004013A2             xor     edx, edx
.text:004013A4             div     [ebp+f2]
.text:004013A7             sub     eax, 1
.text:004013AA             xor     edx, edx
.text:004013AC             or      ecx, 0FFFFFFFFh
.text:004013AF             div     ecx
.text:004013B1             mov     ecx, eax
.text:004013B3             imul    ecx, 9
.text:004013B6             add     ecx, 1
.text:004013B9             mov     eax, [ebp+f2]
.text:004013BC             xor     edx, edx
.text:004013BE             div     ecx
.text:004013C0             mov     [ebp+f2], eax
.text:004013C3             mov     eax, [ebp+a]
.text:004013C6             sub     eax, 1
.text:004013C9             xor     edx, edx
.text:004013CB             or      ecx, 0FFFFFFFFh
.text:004013CE             div     ecx
.text:004013D0             mov     edx, [ebp+a]
.text:004013D3             add     eax, edx
.text:004013D5             xor     edx, edx
.text:004013D7             div     [ebp+f2]
.text:004013DA             sub     eax, 1
.text:004013DD             xor     edx, edx
.text:004013DF             or      ecx, 0FFFFFFFFh
.text:004013E2             div     ecx
.text:004013E4             imul    eax, 9
.text:004013E7             add     eax, 1
.text:004013EA             mov     edx, [ebp+f1]
.text:004013ED             imul    edx, eax
.text:004013F0             mov     [ebp+f1], edx
.text:004013F3             mov     eax, [ebp+f2]
.text:004013F6             mov     [ebp+f1], eax
.text:004013F9             mov     [ebp+f2], 1
.text:00401400             mov     eax, [ebp+a]
.text:00401403             xor     edx, edx
.text:00401405             div     [ebp+f2]
.text:00401408             xor     edx, edx
.text:0040140A             mov     ecx, 0Ah
.text:0040140F             div     ecx
.text:00401411             mov     ecx, edx
.text:00401413             imul    ecx, [ebp+f1]
.text:00401417             mov     eax, [ebp+f1]
.text:0040141A             xor     edx, edx
.text:0040141C             mov     esi, 0Ah
.text:00401421             div     esi
.text:00401423             mov     [ebp+f1], eax
.text:00401426             mov     edx, [ebp+f2]
.text:00401429             imul    edx, 0Ah
.text:0040142C             mov     [ebp+f2], edx
.text:0040142F             mov     eax, [ebp+a]
.text:00401432             xor     edx, edx
.text:00401434             div     [ebp+f2]
.text:00401437             xor     edx, edx
.text:00401439             mov     esi, 0Ah
.text:0040143E             div     esi
.text:00401440             imul    edx, [ebp+f1]
.text:00401444             add     ecx, edx
.text:00401446             mov     eax, [ebp+f1]
.text:00401449             xor     edx, edx
.text:0040144B             mov     esi, 0Ah
.text:00401450             div     esi
.text:00401452             mov     [ebp+f1], eax
.text:00401455             mov     eax, [ebp+f2]
.text:00401458             imul    eax, 0Ah
.text:0040145B             mov     [ebp+f2], eax
.text:0040145E             mov     eax, [ebp+a]
.text:00401461             xor     edx, edx
.text:00401463             div     [ebp+f2]
.text:00401466             xor     edx, edx
.text:00401468             mov     esi, 0Ah
.text:0040146D             div     esi
.text:0040146F             imul    edx, [ebp+f1]
.text:00401473             add     ecx, edx
.text:00401475             mov     eax, [ebp+f1]
.text:00401478             xor     edx, edx
.text:0040147A             mov     esi, 0Ah
.text:0040147F             div     esi
.text:00401481             mov     [ebp+f1], eax
.text:00401484             mov     edx, [ebp+f2]
.text:00401487             imul    edx, 0Ah
.text:0040148A             mov     [ebp+f2], edx
.text:0040148D             mov     eax, [ebp+a]
.text:00401490             xor     edx, edx
.text:00401492             div     [ebp+f2]
.text:00401495             xor     edx, edx
.text:00401497             mov     esi, 0Ah
.text:0040149C             div     esi
.text:0040149E             imul    edx, [ebp+f1]
.text:004014A2             add     ecx, edx
.text:004014A4             mov     eax, [ebp+f1]
.text:004014A7             xor     edx, edx
.text:004014A9             mov     esi, 0Ah
.text:004014AE             div     esi
.text:004014B0             mov     [ebp+f1], eax
.text:004014B3             mov     eax, [ebp+f2]
.text:004014B6             imul    eax, 0Ah
.text:004014B9             mov     [ebp+f2], eax
.text:004014BC             mov     eax, [ebp+a]
.text:004014BF             xor     edx, edx
.text:004014C1             div     [ebp+f2]
.text:004014C4             xor     edx, edx
.text:004014C6             mov     esi, 0Ah
.text:004014CB             div     esi
.text:004014CD             imul    edx, [ebp+f1]
.text:004014D1             add     ecx, edx
.text:004014D3             mov     eax, [ebp+f1]
.text:004014D6             xor     edx, edx
.text:004014D8             mov     esi, 0Ah
.text:004014DD             div     esi
.text:004014DF             mov     [ebp+f1], eax
.text:004014E2             mov     edx, [ebp+f2]
.text:004014E5             imul    edx, 0Ah
.text:004014E8             mov     [ebp+f2], edx
.text:004014EB             mov     eax, [ebp+a]
.text:004014EE             xor     edx, edx
.text:004014F0             div     [ebp+f2]
.text:004014F3             xor     edx, edx
.text:004014F5             mov     esi, 0Ah
.text:004014FA             div     esi
.text:004014FC             imul    edx, [ebp+f1]
.text:00401500             add     ecx, edx
.text:00401502             mov     eax, [ebp+f1]
.text:00401505             xor     edx, edx
.text:00401507             mov     esi, 0Ah
.text:0040150C             div     esi
.text:0040150E             mov     [ebp+f1], eax
.text:00401511             mov     eax, [ebp+f2]
.text:00401514             imul    eax, 0Ah
.text:00401517             mov     [ebp+f2], eax
.text:0040151A             mov     eax, [ebp+a]
.text:0040151D             xor     edx, edx
.text:0040151F             div     [ebp+f2]
.text:00401522             xor     edx, edx
.text:00401524             mov     esi, 0Ah
.text:00401529             div     esi
```



```asm
.text:0040152B imul edx, [ebp+f1]
.text:0040152F add ecx, edx
.text:00401531 mov eax, [ebp+f1]
.text:00401534 xor edx, edx
.text:00401536 mov esi, 0Ah
.text:0040153B div esi
.text:0040153D mov [ebp+f1], eax
.text:00401540 mov edx, [ebp+f2]
.text:00401543 imul edx, 0Ah
.text:00401546 mov [ebp+f2], edx
.text:00401549 mov eax, [ebp+a]
.text:0040154C xor edx, edx
.text:0040154E div [ebp+f2]
.text:00401551 xor edx, edx
.text:00401553 mov esi, 0Ah
.text:00401558 div esi
.text:0040155A imul edx, [ebp+f1]
.text:0040155E add ecx, edx
.text:00401560 mov eax, [ebp+f1]
.text:00401563 xor edx, edx
.text:00401565 mov esi, 0Ah
.text:0040156A div esi
.text:0040156C mov [ebp+f1], eax
.text:0040156F mov eax, [ebp+f2]
.text:00401572 imul eax, 0Ah
.text:00401575 mov [ebp+f2], eax
.text:00401578 mov eax, [ebp+a]
.text:0040157B xor edx, edx
.text:0040157D div [ebp+f2]
.text:00401580 xor edx, edx
.text:00401582 mov esi, 0Ah
.text:00401587 div esi
.text:00401589 imul edx, [ebp+f1]
.text:0040158D add ecx, edx
.text:0040158F mov eax, [ebp+f1]
.text:00401592 xor edx, edx
.text:00401594 mov esi, 0Ah
.text:00401599 div esi
.text:0040159B mov [ebp+f1], eax
.text:0040159E mov edx, [ebp+f2]
.text:004015A1 imul edx, 0Ah
.text:004015A4 mov [ebp+f2], edx
.text:004015A7 mov eax, [ebp+a]
.text:004015AA xor edx, edx
.text:004015AC div [ebp+f2]
.text:004015AF xor edx, edx
.text:004015B1 mov esi, 0Ah
.text:004015B6 div esi
.text:004015B8 imul edx, [ebp+f1]
.text:004015BC add ecx, edx
.text:004015BE mov eax, ecx
.text:004015C0 pop esi
.text:004015C1 mov esp, ebp
.text:004015C3 pop ebp
.text:004015C4 retn
.text:004015C4 sub_401000 endp
```

不能用汇编吗，楼上的太恐怖了～～～

`eax` 是输入，`ebx` 是输出

```asm
MOV EAX,124578
XOR EBX,EBX
ROR EAX,14
MOV BL,AL
ROR EBX,4
ROL EAX,4
MOV BL,AL
ROL EAX,4
ROR EBX,4
MOV BL,AL
ROR EBX,4
ROL EAX,4
MOV BL,AL
ROL EAX,4
ROR EBX,4
MOV BL,AL
ROR EBX,4
ROL EAX,4
MOV BL,AL
ROL EAX,4
ROR EBX,4
XOR BL,BL
```

这个通用性比较强:

```asm
MOV EAX,124578
XOR EBX,EBX
ROL EAX,4
push offset DONE
push offset LOOP
ret
ret
nop
nop
nop
nop
nop
nop

LOOP:
MOV BL,AL
ROL EAX,4
ROR EBX,4
sub esp,4
dec byte ptr [esp]
ret

DONE:
AND AL,0F
AND BL,0F0
OR BL,AL
ROR EBX,4
```

用 `nop` 的长度来控制循环的次数，`nop` 的次数是转换的位数-2

Try compile the following code under VS2005, with `/W4` warning level, for x86

逆序输出 16 进制和 10 进制数，不能使用循环和 `If`，无非是更麻烦一些而已：

// reverse 0x89ABCDEF to 0xFEDCBA98

```cpp
VOID SwapIt( LPBYTE b1, LPBYTE b2 )
{
    BYTE b = *b1;
    *b1 = (*b2)<<4 + (*b2)>>4;
    *b2 = b<<4 + b>>4;
}

HRESULT ReverseInt_Hex( LPDWORD pInt, LPDWORD pdwResult )
{
    DWORD i = 0;
    LPBYTE p = NULL;
    /*
        if( NULL == pInt )               well, since we CANNOT use if, assume the parameter is valid here
            return E_INVALIDARG;
    */
    i = *pInt;
    p = (LPBYTE) &i;

    SwapIt( p[0], p[3] );
    SwapIt( p[1], p[2] );
    *pdwResult = i;
    return S_OK;
}

// since 0xFFFFffff = 4294967295, there are 10 digits at most for a decimal unsigned integer
// e.g. reverse 12345678 to 87654321

HRESULT ReverseInt_Decimal( LPDWORD pInt, LPDWORD pdwResult )
{
    DWORD i = 0;
    DWORD j = 0;
    BYTE k[10] = {0};                                      // stores the digits, from left to right
    /*
        if( NULL == pInt )              again, we trust you, man :-)
            return E_INVALIDARG;
    */
    i = *pInt;
    k[0] = i / 1000000000;    j = i - k[0] * 1000000000;
    k[1] = j /  100000000;    j-= k[1] * 100000000;
    k[2] = j /   10000000;    j-= k[2] * 10000000;
    k[3] = j /    1000000;    j-= k[3] * 1000000;
    k[4] = j /     100000;    j-= k[4] * 100000;
    k[5] = j /      10000;    j-= k[5] * 10000;
    k[6] = j /       1000;    j-= k[6] * 1000;
    k[7] = j /        100;    j-= k[7] * 100;
    k[8] = j /         10;    j-= k[8] * 10;
    k[9] = j;

    *pdwResult = k[9] * 1000000000 +
                 k[8] * 100000000  +
                 k[7] * 10000000   +
                 k[6] * 1000000    +
                 k[5] * 100000     +
                 k[4] * 10000      +
                 k[3] * 1000       +
                 k[2] * 100        +
                 k[1] * 10         +
                 k[0];
    return S_OK;
}
```

看我这个,我写的 `strrev`,表面上没有判断指令的,使用了循环,但是还是没有使用 `jmp` 而是用 `push ret`,这样至少在心里上好受点,因为没有循环的世界和没有 `if` 的世界一样不好受,除非你的内存可以无限的大,我的例子中没有使用 `if` 指令,而是使用了`%`来判断字串结束否,其实也是一种 `if`,但是很不明显而已,没有长度限制

```cpp
void  __declspec(naked) __stdcall mystrrev(const char*  p_str)
{
  __asm
  {
    mov     eax,dword ptr [esp+4]
    pushad
    pushfd
    push  eax
    call  l_kernel
    pop    eax
    popfd
    popad
    retn  4
l_kernel:
    mov     edi, dword ptr [esp+4]
    xor     eax, eax
    mov     ecx, -1
    repne   scas byte ptr es:[edi]
    neg     ecx
    dec     ecx
    dec     ecx
    mov     eax, ecx
    xor     edx, edx
    mov     ebx, 2
    div     ebx
    add     edx, eax
    mov     ebx, edx
    call    l_Loop
    retn
    dec     ebx
l_Loop:
    xor     edx, edx
    mov     eax, 1
    div     ebx
    mov     esi, dword ptr [esp+8]
    mov     al, byte ptr [ecx+esi-1]
    neg     ecx
    mov     ah, byte ptr [ecx+edi-1]
    mov     byte ptr [ecx+edi-1], al
    neg     ecx
    mov     byte ptr [ecx+esi-1], ah
    dec     ecx
    mov     esi, dword ptr [esp]
    add     esi, edx
    push    esi
    retn
  }
}

int _tmain(int argc, _TCHAR* argv[])
{
  char p_sTest[] = "123456789abcdefghijklmnopqrstuvwxyz";
  puts(p_sTest);
  mystrrev(p_sTest);
  puts(p_sTest);
  gets(p_sTest);
  return 0;
}
```

`TC 2.0 pass`

引用:

```cpp
#include <stdio.h>
void pp()
{char c;
if((c=getch())!=27)
{printf("%c",c);
```



用 28 楼 `dwng` 的方法改了上面的代码

引用:

```
#include <stdio.h>

int pp()
{
    char c;
    (
        ((c=getch())!=27)
        &&
        (printf("%c",c)|pp()|printf("%c",c))
    )
    ||
    (printf("\\n\\r"));
    return 1;
}

void main()
{
    pp();
    getch();
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
