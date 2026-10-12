
# 超级小精灵 ver1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum612.htm`](https://www.pediy.com/kssd/pediy03/forum612.htm)

超级小精灵 `ver1.0`

下载地址：`http://www.soft999.com/download2/superspirit.exe`

今天无聊就下载了它，可一看要注册，就当练习很快就把它搞定了。由于我比较懒，过程就不写了，很简单，初学者可以用它来练手。

用户名：`SuperSpirit`

注册码：`Rq0NJA0QJaDaDi`

其实注册成功后，软件在注册表中把 `sos` 的值改为 `1` 就表示已注册成功，你正在使用的就已经是注册版了，哈哈 `sos` 搞定你没有商量。

愿意和每一位破解爱好者交流破解心得。（哈哈，我也是一个初学者，今天很高兴）

```c
#include "stdio.h"

main()

{

char string[81],s[81];

static int f[14]={0x5a,0x73,0x30,0x34,0x48,0x6d,0x33,0x31,

    0x4d,0x61,0x44,0x65,0x4f,0x6b

    };

int i,a=0,b,d,e;

 unsigned c;

loop: printf("******SuperSpirit V1.00****\\nThis keygen is maked by Pretender\\nPlease input your name        ");

gets(string);

for(i=0;(c=string[i])!='\\0';i++)

{

a=c&f[i];

s[i]=a;

}

d=i;e=i;

if(i<7) {printf("Your User name MUST more than 6 chars\\n\\n");goto loop;}

if(i>15) {printf("Your User name MUST less than 16 chars\\n\\n");goto loop;}

else for(d=i;d<=13;d++)

  {a=string[e-2]&f[d];s[d]=a;

  if(e==i) e=e+1;

    else e=e-1;

    }

    s[14]=0x0;

  printf("Your Registation number is    ");

  for(i=0;i<14;i++)

  {string[i]=s[i]-48;

  if(string[i]<0) s[i]=s[i+1]-string[i]+2;

  if(string[i]>74) s[i]=s[i+1]-string[i];

  if(string[i]>=10&&string[i]<=16) s[i]=s[i]+10;

  if(string[i]>=43&&string[i]<=48) s[i]=s[i]+20;

  printf("%c",s[i]);}

printf("\\nWelcome to WWW.CRACKNOW.COM\\n");}
```

```reg
REGEDIT4

[HKEY_LOCAL_MACHINE\\System]

"xios"="1"

[HKEY_CURRENT_USER\\Software\\Microsoft\\Windows]

"sos"="1"

"really"="0"  4dd466d0e1e0cc123afbe504da2462ca
```
