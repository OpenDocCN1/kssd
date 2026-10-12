
# CCProxy 4.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1176.htm`](https://www.pediy.com/kssd/pediy03/forum1176.htm)

```
:00402C1E 8B442434             mov eax, dword ptr [esp+34]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402C1C(U)

:00402C22 0FBE0408             movsx eax, byte ptr [eax+ecx]  ;指向字符串 microserver0,12 位，参与注册码的计算
:00402C26 0FBE140B             movsx edx, byte ptr [ebx+ecx]  ;指向字符串 ccproxy20010,12 位，参与注册码的计算
:00402C2A 03C2                 add eax, edx ；加
:00402C2C BD3E000000           mov ebp, 0000003E ；将来算注册码时作除数用的
:00402C31 0FBE140F             movsx edx, byte ptr [edi+ecx]  ;指向“序列号”,最多 12 位，不足 12 位的，后面补两个 0x0，余下的位数用特征码的相应位补充，参与注册码的计算
:00402C35 03C2                 add eax, edx ；加
:00402C37 0FBE11               movsx edx, byte ptr [ecx]  ;指向“特征码”,12 位，参与注册码的计算
:00402C3A 03C2                 add eax, edx ；加

至此为止加完四个字符串的相应位

:00402C3C 99                   cdq
:00402C3D F7FD                 idiv ebp ；除以 0x3e，得余数有用
:00402C3F 83FA1A               cmp edx, 0000001A  ；余数>0x1a,则转向下一判断
:00402C42 7D05                 jge 00402C49
:00402C44 80C261               add dl, 61       ；小于 0x1a 则加上 0x61
:00402C47 EB0D                 jmp 00402C56

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402C42(C)

:00402C49 83FA34               cmp edx, 00000034 ；若大于 0x1a 再看是否大于 0x34
:00402C4C 7D05                 jge 00402C53      ；不小于则转向 402c53，减去 4
:00402C4E 80C227               add dl, 27       ；小于则加上 0x27
:00402C51 EB03                 jmp 00402C56      ；每位注册码都这样算。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00402C4C(C)

:00402C53 80EA04               sub dl, 04

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00402C47(U), :00402C51(U)

:00402C56 8B442438             mov eax, dword ptr [esp+38]
:00402C5A 88140E               mov byte ptr [esi+ecx], dl  ；保存注册码
:00402C5D 41                   inc ecx
:00402C5E 48                   dec eax                    ；计数剩余的注册码位数，没算完则继续
:00402C5F 89442438             mov dword ptr [esp+38], eax
:00402C63 75B9                 jne 00402C1E
```

注册机：

代理服务器软件 `CCProxy 4.0` 的注册机。

下载地址：[`software.wx88.net/down/ccproxy40.zip`](http://software.wx88.net/down/ccproxy40.zip)

注册机：(`tc2.0` 编译通过：`tcc -mt -lt crk-ccp`)

```c
#include "stdio.h"

/* Key Generator for CCProxy Ver 4.0 */
/* tcc -mt -lt crk-ccp */

main()
{
    char *str1="microserver0";
    char *str2="ccproxy20010";
    char setup[13];
    char rn[13];
    char sn[13];
    int i=0;
    int temp;

L1:
    puts("请输入您的特征码(共 12 位)：");
    gets(setup);
    if(strlen(setup)-12)goto L1;

L2:
    puts("请输入您的序列号(共 12 位)：");
    gets(rn);
    if(strlen(rn)-12)goto L2;

    for(i=0;i<12;i++)
    {
        temp=(str1[i]+str2[i]+setup[i]+rn[i])%0x3e;
        sn[i]=temp>=0x1a?(temp>=0x34?temp-4:temp+0x27):(temp+0x61);
    }
    sn[i]=0;

    printf("\n\n 多谢使用西风瘦马的第一个注册机！\n 您的注册码是: %s",sn);
    getch();
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
