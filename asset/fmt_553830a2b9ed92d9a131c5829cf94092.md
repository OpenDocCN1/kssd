
# ChinaRen 校友录伴侣 v1.01

> 原文：[`www.pediy.com/kssd/pediy03/forum1207.htm`](https://www.pediy.com/kssd/pediy03/forum1207.htm)

## 软件信息

- **软件名称**：ChinaRen 校友录伴侣
- **版本**：`v1.01` build `10717`
- **下载地址**：`http://alumni.chinaren.com/`
- **工具**：`PW32GOLD`，`TRW1.23`

## 逆向过程

1. 先用 `PW32` 反汇编，查找“你输入的用户注册码不正确”。
2. 来到如下地址：

```
:004B5026 8B83E4020000             mov eax, dword ptr [ebx+000002E4]
:004B502C E847EDF7FF               call 00433D78
:004B5031 8B55EC                   mov edx, dword ptr [ebp-14]
:004B5034 58                       pop eax
:004B5035 E8E2F0F4FF               call 0040411C ------>比较
:004B503A 7416                     je 004B5052   ------>不跳即死

* Possible StringData Ref from Code Obj ->"您输入的用户注册码不正确。"

:004B503C B8B8524B00               mov eax, 004B52B8
:004B5041 E85A46FAFF               call 004596A0
:004B5046 8BC3                     mov eax, ebx
:004B5048 E87B99F9FF               call 0044E9C8
:004B504D E9DE010000               jmp 004B5230

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B503A(C)

:004B5052 8D55E8                   lea edx, dword ptr [ebp-18]
.....................................................................................
```

3. 进比较 `CALL`：

```
:0040411C 53                       push ebx
:0040411D 56                       push esi
:0040411E 57                       push edi
:0040411F 89C6                     mov esi, eax --->真的注册码（D EAX）
:00404121 89D7                     mov edi, edx --->假的注册码（D EDX）
:00404123 39D0                     cmp eax, edx --->比较
:00404125 0F848F000000             je 004041BA
......................................................................................
```

4. 运行程序，打开注册框，注册名填 `crackboy`，注册码填 `123456`。
5. 先不要按确定，运行 `TRW`，`BPX 0040411F`，`F5` 回到程序，按确定被拦下。
6. 这时下 `D EAX` 可看到真的注册码 `023690261367041454`。回到程序用真注册码注册成功！

## 整理注册码

- **注册名**：`crackboy`
- **注册码**：`023690261367041454`

如转载请保持文章完整性。

## Keygen 源码

```c
#include "stdio.h"

main()
{
    unsigned char string[20];

    int i,d,x,y,z,m,n;

    unsigned long c,e=0,f=0,o=0,p=0;

    start:printf("***********CRAP 1.0***********\nThis keygen is made by Pretender\nPlease input your name : ");

    gets(string);

    d=strlen(string);

    for(i=0;i<d;i++)
        if(string[i]>='A'&&string[i]<='Z') string[i]+=32;

    x=string[0]&1;y=string[d-1]&1;

    m=x;n=y;

    if(x==0&&y==0) {x=0x3b;y=0x17;z=0x7;}

    if(x==0&&y==1) {x=0x3b;y=0x7;z=0x17;}

    if(x==1&&y==0) {x=0x17;y=0x3b;z=0x7;}

    if(x==1&&y==1) {x=0x17;y=0x7;z=0x3b;}

    for(i=0;i<d;i++)
    {c=string[i];

    e+=c;e+=x;

    f+=c*y;

    o=c/z;o+=p;o+=(i+1)*z;p=o;}

    e*=x;f*=y;o*=x;o*=y;o*=z;

    e%=1000000;f%=1000000;o%=1000000;

    printf("Your Register code is  : ");

    if(m==0&&n==0) printf("%06lu%06lu%06lu",f,o,e);

    if(m==0&&n==1) printf("%06lu%06lu%06lu",o,f,e);

    if(m==1&&n==0) printf("%06lu%06lu%06lu",o,e,f);

    if(m==1&&n==1) printf("%06lu%06lu%06lu",e,o,f);

    printf("\n ***********************     ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*     /       /       / --\n ***********************     ---     ---     --/\n");
}
```
`4dd466d0e1e0cc123afbe504da2462ca`
