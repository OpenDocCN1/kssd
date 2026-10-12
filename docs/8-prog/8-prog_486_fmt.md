
# 质因数分解 C 版

> 原文：[`www.pediy.com/kssd/pediy10/92780.html`](https://www.pediy.com/kssd/pediy10/92780.html)

【原创】质因数分解 C 版

最大数支持 `4294967294`，也就是 `scanf("%lu")` 最大的数。

因为分解 `4294967294` 只需要 `65535` 的质数表。所以速度很快。

理论上 `2.5×10¹⁷` 的分解都可以在很短的时间内分出来。

`2.5×10¹⁷` 需要 `500000000` 的质数表。这里的算法我机器用了 `12.015 s`。

质数表部分算法参考了 `arab` 和 `erex` 的算法。并做了一些优化。

`http://bbs.pediy.com/showthread.php?t=89497`

但是只用到 `30` 因为 `30` 就可以节省 `3/4` 的空间用到 `2310` 也就节省 `4/5`。对速度影响较重。

`arab` 的算法算 `500000000` 需要 `71s`。

没做空间优化的算法因为会用到虚拟内存速度影响很大。

代码:

```c
/**
 *   N*30+1, N*30+7, N*30+11, N*30+13, N*30+17, N*30+19, N*30+23, N*30+29
 **/

#include "stdio.h"
#include "math.h"

long unsigned NUM;

char* real;;

long unsigned k;


inline void wtable(register unsigned long a)
{
    register unsigned long b;
    b=a/30;
    a=a%30;
    switch(a)
    {
      case 1:real[b]=real[b]&&!1;break;
      case 7:real[b]=real[b]&&!(1<<1);break;
      case 11:real[b]=real[b]&&!(1<<2);break;
      case 13:real[b]=real[b]&&!(1<<3);break;
      case 17:real[b]=real[b]&&!(1<<4);break;
      case 19:real[b]=real[b]&&!(1<<5);break;
      case 23:real[b]=real[b]&&!(1<<6);break;
      case 29:real[b]=real[b]&&!(1<<7)
      default:;
    }
}

inline int rtable(register unsigned long a)
{
  register unsigned long b;
  if(a>5)
  {
    b=a/30;
    if(real[b]==0)return 0;
    a=a%30;
    switch(a)
    {
      case 1:return real[b]&&1;
      case 7:return real[b]&&(1<<1);
      case 11:return real[b]&&(1<<2);
      case 13:return real[b]&&(1<<3);
      case 17:return real[b]&&(1<<4);
      case 19:return real[b]&&(1<<5);
      case 23:return real[b]&&(1<<6);
      case 29:return real[b]&&(1<<7);
      default:return 0;
    }
  }
  else
  {
    switch(a)
    {
      case 2:return 1;
      case 3:return 1;
      case 5:return 1;
      default:return 0;
    }
  }
}

void suShu()
{


  register unsigned long i=3,j,step;

  for(j=0; j<NUM/30+1; j++)
  {
    real[j] = 0xFF;//初始化数组
  }

  while(i<=k)
  {
  if(rtable(i)==0)
    {
      step=i<<1;
      for(j=i*i;j<=NUM;j+=step)
      wtable(j);
    }
    ++i;
    ++i;
  }

}


int main()
{
  unsigned long in;
  register unsigned long i;
  int f=0;
  printf("please inpout the No.:\n");
  scanf("%lu",&in);
  NUM=(unsigned long)sqrt((long double)in);
  real = new char[NUM/30+1];
  k = (long)sqrt((double)NUM);
  suShu();
  printf("%lu=",in);
  for (i=2;i<=NUM;i++)
  {
    if(rtable(i)==0)
      continue;
    else
    {
      if(in%i!=0)
        continue;
      else
      {
        in=in/i;
        i--;
        printf("%ld*",i+1);
        f=1;
      }
    }

  }
  if(in==1&f==1)printf("\b");
  else if(f==1)printf("%ld",in);
  else if(in<2)printf("\b can not factorization !");
  else printf("\b is prime number !");

  return 0;
}
```

ps：`2.5×10¹⁷` 基本上是 `56-58` 位 2 进制。而真正的 RSA 一般要求 `512` 位密匙。所以 RSA 还是相当安全的。
