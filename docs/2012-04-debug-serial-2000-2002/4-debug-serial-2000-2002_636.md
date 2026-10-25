
# 极速火龙 CD 压缩器

> 原文：[`www.pediy.com/kssd/pediy03/forum1271.htm`](https://www.pediy.com/kssd/pediy03/forum1271.htm)

我所说的是极速火龙 CD 压缩器的注册机，它的注册码生成过程是这样的：

```asm
0167:00476907  MOV       EAX,[EBP-08]//读取用户名前面的 4 位字符   
0167:0047690A  MOV       AL,[EAX]//获取 ASCII 码   
0167:0047690C  MOV       [EBP-04],AL   
0167:0047690F  MOV       EAX,[EBP-08]   
0167:00476912  MOV       AL,[EAX+01]   
0167:00476915  MOV       [EBP-03],AL   
0167:00476918  MOV       EAX,[EBP-08]   
0167:0047691B  MOV       AL,[EAX+02]   
0167:0047691E  MOV       [EBP-02],AL   
0167:00476921  MOV       EAX,[EBP-08]   
0167:00476924  MOV       AL,[EAX+03]   
0167:00476927  MOV       [EBP-01],AL   
0167:0047692A  LEA       EDX,[EBP-0C]   
0167:0047692D  MOV       EAX,[EBX+02EC]   
0167:00476933  CALL      0042D598   
0167:00476938  MOV       EAX,[EBP-0C]   
0167:0047693B  CALL      00408704   
0167:00476940  MOV       EBX,EAX   
0167:00476942  XOR       EAX,EAX   
0167:00476944  POP       EDX   
0167:00476945  POP       ECX   
0167:00476946  POP       ECX   
0167:00476947  MOV       [FS:EAX],EDX   
0167:0047694A  JMP       SHORT 00476958   
0167:0047694C  JMP       004032B0   
0167:00476951  XOR       EBX,EBX   
0167:00476953  CALL      0040360C   
0167:00476958  MOVSX     EAX,BYTE [EBP-04]//将 4 个 ASCII 码相加   
0167:0047695C  MOVSX     EDX,BYTE [EBP-03]   
0167:00476960  ADD       EAX,EDX   
0167:00476962  MOVSX     EDX,BYTE [EBP-02]   
0167:00476966  ADD       EAX,EDX   
0167:00476968  MOVSX     EDX,BYTE [EBP-01]   
0167:0047696C  ADD       EAX,EDX   
0167:0047696E  IMUL      EAX,EAX,00BC614E//将加后的和转换成 16 制数字，并且乘以 BC614E,获取 16 制的数字    
0167:00476974  MOV       ECX,BB//将 ECX 赋值 10 制数字 187   
0167:00476979  CDQ//将 EAX*BC614E 的 16 制的数字转换成 10 制   
0167:0047697A  IDIV      ECX//然后除以 187   
0167:0047697C  CMP       EBX,EAX//EAX 就是正确的注册码。
```

上面的注册码生成过程，在您的帮助下，我还是没有办法用 `VB` 写出注册机，也许是我的水平太差了吧。上面的过程您可以写出注册机，无论是 `VB` 或者是 `VC`，请告诉我正确的注册机该怎样写，谢谢了。

```c
#include "stdio.h"

main()
{
static char string[20];

int i;

long c,e=0;

start:printf("******SpeedDragon 1.2******\nThis keygen is made by Pretender\nPlease input your name : ");

gets(string);

for(i=0;i<4;i++)

{c=string[i];e+=c;}

e*=0xbc614e;

e/=0xbb;

if(e>1000) printf("Your Register code is  : %ld",e);

else {printf("You must change your username!\n");goto start;}

printf("\n ******************************       ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*       /       /       / --\n ******************************       ---     ---     --/\n");

}  4dd466d0e1e0cc123afbe504da2462ca
```
