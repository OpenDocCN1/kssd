
# 随机字符串生成函数

> 原文：[`www.pediy.com/kssd/pediy10/89204.html`](https://www.pediy.com/kssd/pediy10/89204.html)

写着玩的，见笑![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

```c
#include "stdafx.h"

#include <stdio.h>

#include "windows.h"

#pragma comment(lib,"winmm.lib")

int main(int argc, char* argv[])
{
  int RanNum;
  int n1,n2;
  int count;
  char *buffer;
  char ch;
  //产生种子
  srand((unsigned(timeGetTime())));
  RanNum=rand();
  n1=RanNum%10;
  printf("随机数字%d\n",n1);
  n1++;
  buffer=(char *)VirtualAlloc(NULL,2*n1,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
  memset((void *)buffer,0,2*n1);
  for (count=0;count<(2*n1-1);count++)
  {
    RanNum=rand();
    n2=RanNum%52;
    printf("随机数字%d\n",n2);
    if (n2>=26)
    {
      ch=n2&0x0F;
      ch+=0x47;  //'a'------'z'
    }
    else
    {
      ch=n2&0x0F;
      ch+=0x41;  //'A'-----'Z'
    }
    buffer[count]=ch;
  }
  printf("生成随机字符串:%s\n",buffer);
  VirtualFree(buffer,2*n1,MEM_DECOMMIT);
  return 0;
}
```

感觉 `LZ` 的代码比较复杂啊

```c
#include <stdio.h>

#include <stdlib.h>

#include <memory.h>

char* GetRndStr(unsigned int nLen)
{
  char* psz=NULL;
  long lrdtsc;
  psz=(char*)malloc(nLen+1);
  if (psz)
  {
    psz[nLen]=0;
    __asm RDTSC;
    __asm mov lrdtsc,eax;
    while(nLen--)
    {
      psz[nLen]=((lrdtsc>>(8*(nLen%4)))&0xff)%26+'a';
    }
  }
  return psz;
}

int main(int argc, char* argv[])
{
  char* psz=NULL;
  psz=GetRndStr(10);
  if (psz)
  {
    printf("生成随机字符串:%s\n",psz);
    free(psz);
  }
  return 0;
}
```
