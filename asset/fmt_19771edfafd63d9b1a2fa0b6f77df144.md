
:005B4B04 BE01000000  mov esi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:005B4B02(C)

|

:005B4B09 43  inc ebx

:005B4B0A 4F  dec edi

:005B4B0B 75B7 jne 005B4AC4

==============================================================================================

它的算法已经明确，就做个注册机吧。

TC 2.0 下调试通过

```c
#include "stdio.h"

int key[48]={'B','i','X','w','D','I','d','i','5','1','6','8','q','c','I','E','F','M','E','W','D','S','V','d','k','v','m','e','m','i','f','P','C','i','E','s','e','f','G','F','2','1','Q','C','L','M','W','E'};

main()
{
char S[10],*p=S;
int *p1=key;
unsigned long a,b,c,d,e;

printf("ChengYuSuCha v3.0 keygen by HMILY[BCG]\n");
printf("My e-mail : gyyxll@21cn.com\n");
printf("*******************HMILY[BCG]*******************\n");
printf("please enter your serial : ");
gets(S);
printf(" Your register code is   : ");
while(*p!='\0')
  {
    a=*p&0x0F;
    b=*p1&0x0F;
    c=a^b;
    d=*p&0xF0;
    e=c+d;
    p++;p1++;
    printf("%c",e);
  }
}
```

4dd466d0e1e0cc123afbe504da2462ca
