
# 神奇注册表

> 原文：`www.pediy.com/kssd/pediy03/forum576.htm`

```c
#include "stdio.h"   

main()   

{   

char string[81];   

static int f[32]={0x48,0xee,0x76,0x1d,0x67,0x69,0xa1,0x1b,   

    0x7a,0x8c,0x47,0xf8,0x54,0x95,0x97,0x5f,   

    0x78,0xd9,0xda,0x6c,0x59,0xd7,0x6b,0x35,   

    0xc5,0x77,0x85,0x18,0x2a,0x0e,0x52,0xff};   

int i,a=0;   

  unsigned c;   

printf("This keygen is maked by Pretender\nPlease input your name       ");   

gets(string);   

printf("Your Registation number is   ");   

for(i=0;(c=string[i])!='\0';i++)   

{if(c>127) c=-c;   

a=c^f[i];if(a>127) a=a-256;   

while(a>57) {a-=3;continue;}   

while(a<48) {a+=3;continue;}   

printf("%d",a-48);   

}   

printf("\nWelcome to WWW.CRACKNOW.COM");}
```

目前只能以 32 个以下字符数（16 个以下汉字也可以）作用户名~~~~（我懒得抄`F[]`那张表~只抄了 32 个~~呵呵~~~~~不过我想够用了！）

没天理！为什么我每次都要贴 3 遍才能成功？！！？！？！？  4dd466d0e1e0cc123afbe504da2462ca
