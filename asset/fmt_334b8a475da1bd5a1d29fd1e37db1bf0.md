
### 汇编代码段

```assembly
00477786  |ADD EAX,ESI                       ; eax=eax+esi
00477788  |MOV ESI,0DE6FD                    ; esi=0DE6FDh
0047778D  |CDQ                               ; edx 清零
0047778E  |IDIV ESI                          ; eax/esi,商送 eax，余数送 edx
00477790  |MOV ESI,EDX                       ; 余数送 esi
00477792  |INC EBX                           ; ebx++
00477793  |DEC ECX                           ; ecx--
00477794  \JNZ SHORT HNMWEBGA.00477765        ; 未取完则返回继续
00477796  MOV EAX,ESI                         ; 最后的结果送 eax
00477798  MOV ECX,0DE6FD                     ; ecx=0DE6FDh
0047779D  CDQ                                 ; edx 清零
0047779E  IDIV ECX                            ; eax/ecx
004777A0  MOV ESI,EDX                         ; 余数送 esi
004777A2  CMP ESI,2710                        ; 是否大于等于 2710h?
004777A8  JGE SHORT HNMWEBGA.004777B3         ; 是则跳
004777AA  MOV EAX,182C5                       ; eax=182C5h
004777AF  SUB EAX,ESI                         ; eax=eax-esi
004777B1  MOV ESI,EAX                         ; esi=eax
004777B3  MOV EBX,2                           ; ebx=2
004777B8  MOV EDI,HNMWEBGA.004E1698
004777BD  /MOV EAX,ESI                        ; eax=esi
004777BF  |MOV ECX,0A                         ; ecx=0Ah
004777C4  |CDQ                                ; edx 清零
004777C5  |IDIV ECX                           ; eax/ecx
004777C7  |CMP EAX,DWORD PTR DS:[EDI]         ; 商是否等于 D92Dh(第二次是 171F2h)?
004777C9  |JNZ SHORT HNMWEBGA.004777DD        ; 不等就跳
004777CB  |MOV EAX,0E5588                     ; eax=0E5588h
004777D0  |CALL HNMWEBGA.00402C08             ; 这个就是 call(2)
004777D5  |MOV ESI,EAX                        ; 返回值送 esi
004777D7  |ADD ESI,270F                       ; esi=esi+270Fh
004777DD  |ADD EDI,4                          ; edi=edi+4
004777E0  |DEC EBX                            ; ebx--
004777E1  \JNZ SHORT HNMWEBGA.004777BD        ; 不为零则返回继续
004777E3  MOV EBX,ESI                         ; 最后的返回值
004777E5  XOR EAX,EAX
004777E7  POP EDX
004777E8  POP ECX
004777E9  POP ECX
004777EA  MOV DWORD PTR FS:[EAX],EDX
004777ED  PUSH HNMWEBGA.00477802
004777F2  LEA EAX,DWORD PTR SS:[EBP-4]
004777F5  CALL HNMWEBGA.00403BD8
004777FA  RETN
004777FB  JMP HNMWEBGA.00403654
00477800  JMP SHORT HNMWEBGA.004777F2
00477802  MOV EAX,EBX                         ; 最后的返回值
00477804  POP EDI
00477805  POP ESI
00477806  POP EBX
00477807  POP ECX
00477808  POP EBP
00477809  RETN
```

### 附表

```text
004E1674  95 02 00 00 D3 00 00 00 C5 02 00 00 A3 01 00 00
004E1684  E5 03 00 00 65 00 00 00 7B 01 00 00 D3 01 00 00
004E1694  BD 02 00 00 2D D9 00 00 F2 71 01 00 5F 8D 40 00
```

### 整理信息

`Regname`: `cyclotron[BCG]`
`Regcode`: `536956-WG2-790`

### 算法总结

1. `sn1` 与 `sn2` 必须满足下列关系：
   `sn1 % 2BDh + 63h = sn2`

2. 对用户名的计算(用户名必须大于等于 5 位)：
   a. 将用户名复制两遍并连接起来，如 `cyclotroncyclotroncyclotron`
   b. 计数器 `i` 从 0 开始，根据计数器取密码表的 `dword` 值 `table[(i+1)%9]`，根据计数器取用户名串 `cyclotroncyclotroncyclotron` 每一位的 ASCII 值 `name[i]`。
      `{tmp=table[i+1]*name[i]*182C5h; tmp+=sn1; val=tmp%0xDE6FD;}`
      循环用户名长度 * 3 次。
   c. `val = val % DE6FDh`，判断 `sn1` 是否大于等于 `2710h`，是则 `val = 182C5h - val`，否则继续。

3. 最终的 `val` 必须与 `sn1` 相等。

### Turbo C 注册机

```c
#include "stdio.h"
#include "string.h"

void main()
{
    int table[9]={0x295,0xD3,0x2C5,0x1A3,0x3E5,0x65,0x17B,0x1D3,0x2BD},i,length;
    long sn1=0,tmp=1;
    char regname[30];

    printf("\t**********************************************\n");
    printf("\n\t\tKeyGen for HNM Web Gallery Creator V2.12\n\t\t\tProduced by cyclotron\n");
    printf("\n\t**********************************************\n");

    do
        printf("\n\tPlease input your Regname(>4 chars):");
    while((length=strlen(gets(regname)))<=5);

    for(i=0;i<length*3;i++)
    {
        tmp=regname[i%length]*0x182C5*table[(i+1)%9];
        tmp+=sn1;
        sn1=tmp%0xDE6FD;
    }

    sn1=sn1%0xDE6FD;
    sn1=sn1>=0x2710?sn1:0x182C5-sn1;

    printf("\n\tYour Regcode is:\t%ld-WG2-%ld\n\n\tThank you for your use!\n",sn1,sn1%0x2BD+0x63);
    getchar();
}
```

`cyclotron[BCG]`
2003.8.4
`4dd466d0e1e0cc123afbe504da2462ca`
