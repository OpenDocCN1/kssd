
# 天音怒放 2.15

> 原文：[`www.pediy.com/kssd/pediy03/forum939.htm`](https://www.pediy.com/kssd/pediy03/forum939.htm)

## 注册码跟踪过程

**姓名：** `CrackerABC[BCG]`

**注册码：** `612-4972-13`

下面是过程：

首先随便输入任意注册码：`12121212`

启动 `TRW2000`，然后使用搜索法

```
s 30:0 l ffffffff '12121212'
```

找到一个地址 `******`

然后下 `BPM 30:****`

`F5` 返回程序空间，点注册被拦截

下 `BD *`

`PMODULE`

被拦截到这里：

```
0167:004382BE  CALL    `USER32!CallWindowProcA`
0167:004382C3  MOV     [EBX+0C],EAX
0167:004382C6  MOV     EAX,[EBX]
0167:004382C8  CMP     EAX,BYTE +0C
0167:004382CB  JNZ     004382E8
```

然后一直小心的 `F10` 到下面位置：

```
0167:004AA146  LEA     EDX,[EBP-08]
0167:004AA149  MOV     EAX,[EBX+02E8]
0167:004AA14F  CALL    0043451C
0167:004AA154  MOV     EAX,[EBP-08]
0167:004AA157  POP     EDX
0167:004AA158  CALL    004BA774  //----->典型比对，F8 跟进去
0167:004AA15D  TEST    AL,AL
0167:004AA15F  JZ      004AA16B
0167:004AA161  MOV     DWORD [EBX+0234],01
0167:004BA7AD  MOV     DL,01
0167:004BA7AF  MOV     EAX,[004B9A9C]
0167:004BA7B4  CALL    004BF380
0167:004BA7B9  MOV     EBX,EAX
0167:004BA7BB  CMP     BYTE [EBX+10],00
0167:004BA7BF  JZ      004BA83B
```

`F8` 跟进来以后 `F10` 到下面位置：

```
0167:004BA7AA  MOV     ECX,[EBP-04]
0167:004BA7AD  MOV     DL,01
0167:004BA7AF  MOV     EAX,[004B9A9C]
0167:004BA7B4  CALL    004BF380
0167:004BA7B9  MOV     EBX,EAX //--------> D EAX 得到注册码，是后面的那个。注意格式是 ***-****-**
0167:004BA7BB  CMP     BYTE [EBX+10],00
0167:004BA7BF  JZ      004BA83B
0167:004BA7C1  MOV     EAX,[004C3360]
0167:004BA7C6  MOV     EAX,[EAX]
0167:004BA7C8  MOV     EDX,[EAX+049C]
0167:004BA7CE  LEA     EAX,[EBP-10]
0167:004BA7D1  MOV     ECX,004BA88C
0167:004BA7D6  CALL    00404004
0167:004BA7DB  MOV     ECX,[EBP-10]
0167:004BA7DE  MOV     DL,01
0167:004BA7E0  MOV     EAX,[00472E94]
0167:004BA7E5  CALL    00472F3C
0167:004BA7EA  MOV     ESI,EAX
0167:004BA7EC  MOV     EAX,[EBP-04]
0167:004BA7EF  PUSH    EAX
0167:004BA7F0  MOV     ECX,004BA89C
0167:004BA7F5  MOV     EDX,004BA8B0
0167:004BA7FA  MOV     EAX,ESI
0167:004BA7FC  MOV     EDI,[EAX]
0167:004BA7FE  CALL    NEAR [EDI+04]
0167:004BA801  MOV     EAX,[EBP-08]
0167:004BA804  PUSH    EAX
0167:004BA805  MOV     ECX,004BA8C0
0167:004BA80A  MOV     EDX,004BA8B0
0167:004BA80F  MOV     EAX,ESI
0167:004BA811  MOV     EDI,[EAX]
0167:004BA813  CALL    NEAR [EDI+04]
0167:004BA816  MOV     EAX,ESI
0167:004BA818  CALL    00402FD0
0167:004BA81D  MOV     EAX,004BA8D4
0167:004BA822  CALL    0045A750
0167:004BA827  MOV     EAX,[004C3360]
0167:004BA82C  MOV     EAX,[EAX]
0167:004BA82E  MOV     BYTE [EAX+049A],01
0167:004BA835  MOV     BYTE [EBP-09],01
0167:004BA839  JMP     SHORT 004BA849
0167:004BA83B  MOV     BYTE [EBP-09],00
0167:004BA83F  MOV     EAX,004BA8F4
0167:004BA844  CALL    0045A750 //------>出错了，注册不正确。
0167:004BA849  MOV     EAX,EBX
0167:004BA84B  CALL    00402FD0
```

`********************************************************************`

**主页：** `http://crackerabc.longcity.net` （`[BCG]`大本营）

软件破解，软件注册，破解教学，破解申请，注册机码。

请保留我的信息，谢谢！

`********************************************************************`

## 注册机源码

```c
#include "stdio.h"

main()
{
char string[81];

long pw[12]={0x185a5411,0x2afc4281,0x28a912b5,0x0f732f09,
         0x119fe5f2,0x0bb026fd,0x120abb66,0x0ac01ff5,
         0x29a9f2f0,0x3541c352,0x353a745c,0x388a8742/*原来的注册机就错在这！原来是 0x288a8742 这是因为我不小心看错了，呵呵~~）*/};

long d,e=0;

int i,c;

printf("******TYPLAYER 2.15********\nThis keygen is made by Pretender\nPlease input your name : ");

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

if(e>0)
{d=e;d/=10;
if(d<100000000) d=d*10+8;
if(d<10000000) d=d*100+45;
if(d<1000000) d=d*1000+738;
if(d<100000) d=d*10000+7338;
if(d<10000) d=d*100000+83690;
if(d<1000) d=d*1000000+309678;
if(d<100) d=d*10000000+2467309;
if(d<10) d=d*100000000+13247685;}

if(e<0)
{d=-e;
 if(d<100000000) d=d*10+8;
if(d<10000000) d=d*100+45;
if(d<1000000) d=d*1000+738;
if(d<100000) d=d*10000+7338;
if(d<10000) d=d*100000+83690;
if(d<1000) d=d*1000000+309678;
if(d<100) d=d*10000000+2467309;
if(d<10) d=d*100000000+13247685;}

for(i=0;i<9;i++)
{string[i]=d%10;
 d=d/10;}

printf("Your Register code is  : ");

for(i=0;i<9;i++)
{printf("%d",string[8-i]);
 if(i==2||i==6) printf("-");}

printf("\n ********************************************************************     ---    ---    ---\n *Welcome to WWW.CRACKNOW.COM*     /      /      / --\n ********************************************************************     ---    ---    --/\n");}
```

`4dd466d0e1e0cc123afbe504da2462ca`
