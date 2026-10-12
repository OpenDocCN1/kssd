
# 天音怒放的注册机

> 原文：[`www.pediy.com/kssd/pediy03/forum795.htm`](https://www.pediy.com/kssd/pediy03/forum795.htm)

```c
#include "stdio.h"

main()
{
char string[81];

long pw[12]={0x185a5411,0x2afc4281,0x28a912b5,0x0f732f09,
     0x119fe5f2,0x0bb026fd,0x120abb66,0x0ac01ff5,
     0x29a9f2f0,0x3541c352,0x353a745c,0x288a8742};

long d,e=0;

int i,c;

printf("******TYPLAYER 2.13B******\nThis keygen is maked by Pretender\nPlease input your name : ");

gets(string);

for(i=0;(c=string[i])!='\0';i++)
{if(c<0) c+=256;
  if(c>96&&c<123) c-=32;
  d=pw[(i+1)%12]*c;
  e+=d;
}

printf("Your name is          : ");

puts(string);

e=(0x7fffffff-e)*0xd3;

d=e=e>0?e:-e;

/*if(d<100000000) d=d*10+8;
if(d<10000000) d=d*100+45;
if(d<1000000) d=d*1000+738;
if(d<100000) d=d*10000+7338;
if(d<10000) d=d*100000+83690;
if(d<1000) d=d*1000000+309678;
if(d<100) d=d*10000000+2467309;
if(d<10) d=d*100000000+13247685;*/

for(i=0;i<9;i++)
{string[i]=d%10;
  d=d/10;}

printf("Your Register code is  : ");

for(i=0;i<9;i++)
{printf("%d",string[8-i]);
  if(i==2||i==6) printf("-");}

printf("\nWelcome to WWW.CRACKNOW.COM\n");}
```

有点问题，就是第一步算出的数是否大于 9 位，如果小于这个注册机不合适

但我试了 10 几个用户名都没有出现小于 10 位的~所以~~~嘿嘿~~  4dd466d0e1e0cc123afbe504da2462ca
