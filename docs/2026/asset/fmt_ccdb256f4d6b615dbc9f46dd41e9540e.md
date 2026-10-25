
```
0040653D   |.  8D85 FCFEFFFF     lea eax,dword ptr ss:[ebp-104]

00406543   |.  68 60AB4100       push AXMAN.0041AB60                  ; /src = "My Love"

00406548   |.  50                push eax                             ; |dest

00406549   |.  E8 D8B90000       call <jmp.&MSVCRT.strcat>            ; \strcat

0040654E   |.  83C4 18           add esp,18

00406551   |.  8D45 E8           lea eax,dword ptr ss:[ebp-18]        ; (ASCII "jackilyhttp://estudy.ys168.comMy Love")

00406554   |.  8BCF              mov ecx,edi                          ；以上为 把name和company字符串所有字符小写并把“My Love”连接在一起

00406556   |.  50                push eax

00406557   |.  8D85 FCFEFFFF     lea eax,dword ptr ss:[ebp-104]

0040655D   |.  50                push eax

0040655E   |.  E8 8D010000       call AXMAN.004066F0                   ；关键之关键，算法，跟进

00406563   |.  FF75 E8           push dword ptr ss:[ebp-18]           ; /<%u> = 95AAA45(注册码的HEX值,十进制为156936773)

00406566   |.  8D45 D0           lea eax,dword ptr ss:[ebp-30]        ; |

00406569   |.  68 40A04100       push AXMAN.0041A040                  ; |format = "%u" (c语言中代表无符号十进制值）

0040656E   |.  50                push eax                             ; |s

0040656F   |.  FF15 30464100     call dword ptr ds:[<&MSVCRT.sprintf>>; \sprintf

00406575   |.  83C4 0C           add esp,0C

00406578   |.  8D45 D0           lea eax,dword ptr ss:[ebp-30]

0040657B   |.  8BCF              mov ecx,edi

0040657D   |.  6A 01             push 1

0040657F   |.  6A 09             push 9

00406581   |.  6A 30             push 30

00406583   |.  50                push eax

00406584   |.  E8 73000000       call AXMAN.004065FC

00406589   |.  8B75 10           mov esi,dword ptr ss:[ebp+10]        ; 以下是将156936773每三个数字间加“-”

0040658C   |.  8D45 D0           lea eax,dword ptr ss:[ebp-30]        ; ASII "156936773"

0040658F   |.  6A 03             push 3                               ; /maxlen = 3 复制前3个字符

00406591   |.  50                push eax                             ; |src = "156936773"

00406592   |.  56                push esi                             ; |dest

00406593   |.  FF15 D4454100     call dword ptr ds:[<&MSVCRT.strncpy>>; \strncpy

00406599   |.  8066 03 00        and byte ptr ds:[esi+3],0

0040659D   |.  BB 5CAB4100       mov ebx,AXMAN.0041AB5C

004065A2   |.  53                push ebx                             ; /src => "-"

004065A3   |.  56                push esi                             ; |dest = "156"

004065A4   |.  E8 7DB90000       call <jmp.&MSVCRT.strcat>            ; \strcat

004065A9   |.  8B3D D8454100     mov edi,dword ptr ds:[<&MSVCRT.strnc>;  MSVCRT.strncat

004065AF   |.  8D45 D3           lea eax,dword ptr ss:[ebp-2D]

004065B2   |.  6A 03             push 3                               ; /maxlen = 3

004065B4   |.  50                push eax                             ; |src = "936773"

004065B5   |.  56                push esi                             ; |dest = "156-"

004065B6   |.  FFD7              call edi                             ; \strncat

004065B8   |.  8066 07 00        and byte ptr ds:[esi+7],0

004065BD   |.  56                push esi                             ; |dest = "156-936"

004065BD   |.  56                push esi                             ; |dest

004065BE   |.  E8 63B90000       call <jmp.&MSVCRT.strcat>            ; \strcat

004065C3   |.  8D45 D6           lea eax,dword ptr ss:[ebp-2A]

        .

        .                                                         ;004065C6至004065EB非关键部分省略

        .

        .

004065EE   |.  5F                pop edi

004065EF   |.  5E                pop esi

004065F0   |.  5B                pop ebx

004065F1   |.  64:890D 00000000  mov dword ptr fs:[0],ecx

004065F8   |.  C9                leave

004065F9   \.  C2 0C00           retn 0C
```

---

由`0040655E`跟进，以下为此次重点，关键算法，到这里了，有一种自豪感吧。

```
004066F4   |.  33D2              xor edx,edx                  ; edx清0

004066F6   |.  8A01              mov al,byte ptr ds:[ecx]     ；第一个字符“j”，HEX值为61，放入al

004066F8   |.  84C0              test al,al

004066FA   |.  74 25             je short AXMAN.00406721      ;是0的话跳走

004066FC   |.  56                push esi                     ；开始计算

004066FD   |>  0FBEC0            /movsx eax,al                ；eax=al

00406700   |.  C1E2 04           |shl edx,4                   ; edx 左移0x4

00406703   |.  03D0              |add edx,eax                 ；edx=edx+eax

00406705   |.  41                |inc ecx                     ; ecx加1，也就是上面指向字符串地址向后移一位，取下一个字符

00406706   |.  8BC2              |mov eax,edx                 ；把edx值给eax

00406708   |.  25 000000F0       |and eax,F0000000            ；与0xF0000000

0040670D   |.  74 07             |je short AXMAN.00406716     ；为0的话，至00406716

0040670F   |.  8BF0              |mov esi,eax

00406711   |.  C1EE 18           |shr esi,18

00406714   |.  33D6              |xor edx,esi                  ；异或

00406716   |>  F7D0              |not eax                     ；取反

00406718   |.  23D0              |and edx,eax                 ；

0040671A   |.  8A01              |mov al,byte ptr ds:[ecx]    ； 取下一个字符放入al里

0040671C   |.  84C0              |test al,al                  ；检查是否为0，也就是是否取完字符串中的字符

0040671E   |.^ 75 DD             \jnz short AXMAN.004066FD    ；满足条件的话，继续计算

00406720   |.  5E                pop esi

00406721   |>  8B4424 08         mov eax,dword ptr ss:[esp+8]

00406725   |.  8910              mov dword ptr ds:[eax],edx   ;最终结果放EAX指向的地址里

00406727   \.  C2 0800           retn 8
```

---

## 破解总结

由此向上追溯到`00406480`处，为比较真假注册码。此注册代码部分有三个关键`call`,分别是`00406464`、`00406491`和`004066F0`。个人认为关键算法倒不是很难，难点在于要理解“数据处理”，也就是把`name`和`company`字符串和`"My Love"` `strcat`在一起的部分，和将注册码每三个数字间加“-”这两个部分运算。好在ollydbg调试结构原理让我们能清晰、直观地分析这些调用。该注册机在本人文件空间 <http://estudy.ys168.com> 可以下载。

## 用户名、密码

提供一组注册码：

`name`: jackily
`company`: http://estudy.ys168.com
`code`: 156-936-773

---

## 算法注册机

```
/* axman 3.12rGB c语言注册机 */
/* 在Turboc 2.0 下调试通过  */
#include "stdio.h"
#include "string.h"
#include "stdlib.h"
main()
{ char name[100]={0},company[50]={0},code[50]={0};
  int i;
  unsigned long l=0,k,m;
  printf("Axman 3.12rGB KeyGen by jackily 2004-12-15\n");
  printf("Email:jackily_zhang@msn.com or jackily_zhang@yahoo.com.cn\n");
  printf("please input name:");
  scanf("%s",name);
  printf("\nplease input company:");
  scanf("%s",company);
  strcat(name,company);
  strlwr(name);
  strcat(name,"My Love");
  for (i=0;i<strlen(name);i++)
     { k=name[i];l<<=0x4;l+=name[i];k=l;
     if ((k&=0xf0000000)!=0) { m=k; m>>=0x18; l^=m;}
     k=~k; /*0xffffffff-k;*/  /*not eax*/
     l&=k;
     }
  ltoa(l,code,10);  /* change "l" into string */
  printf("\nserial number is:");
  for (i=0;i<3;i++) printf("%c",code[i]);
  printf("-");
  for (i=3;i<6;i++) printf("%c",code[i]);
  printf("-");
  for (i=6;i<strlen(code);i++)
  printf("%c",code[i]);
  if (9-strlen(code)==1) printf("0");  /* 注册码长度小于9时，补0 */
  if (9-strlen(code)==2) printf("00");
  }
```

---

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

2004-11-15
