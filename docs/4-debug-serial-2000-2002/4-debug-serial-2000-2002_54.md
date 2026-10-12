
# FlashFXP 1.2.602

> 原文：[`www.pediy.com/kssd/pediy01/forum091.htm`](https://www.pediy.com/kssd/pediy01/forum091.htm)

`FlashFXP` V1.2 Build 602

下载地址：

`http://software.wx88.net/ftpfiles/internet/int-ftpm/ffxp1.2.602.zip`

或：`http://www.flashfxp.com/preview/ffxp1.2.602.zip`

这个程序的原来似乎是用 `UPX` 加壳的，这次却换用 `PECompact` 加壳了。手动脱壳不难。程序入口是一对 `PUSHFD/PUSHAD`，在第一个 `CALL` 时 F8 进入，F10 往下走一阵。找到 `POPAD/POPFD` 这对指令，在其上设断。拦下，F10 到 `RET` 行时便可用 `Procdump` 脱壳了。`EOP=518024`。

注册信息以加密过的形式存放在 `FlashFxp.ini` 中，程序启动时读入并解码还原。因此，设断 `bpx getprivateprofilestringa do "dd \*(ss:(esp+8))"`。运行程序，第 2 次拦截，数据窗中见到 `Serial` 的字样。F12 回到程序，F10 往下两行，`EDX` 中便是密文形式的序号。设断 `bpr`。可以见到程序将序号转放到内存的另一处，同样对这一处设断 `bpr`。拦下后跟踪，来到

```
.0047DFFE: 8B4508                mov     eax,[ebp+00008]
.0047E001: 8B55F0                mov     edx,[ebp-0010]  /在此行设断
.0047E004: E83F5CF8FF            call    .000403C48
.0047E009: 33C0                  xor     eax,eax
```

可以见到 `EDX` 中先后出现解码后的序号和姓名（即我们输入的注册信息）。然后 F10 来到

```
.00509EB4: E8739FEFFF            call    .000403E2C      /(1)
.00509EB9: 83F802                cmp     eax,002         /注册码长度大于姓名长度的 2 倍?
.00509EBC: 0F8EBA000000          jle     .000509F7C      /不是则跳开(这里应使其不跳)
.00509EC2: 8B45F8                mov     eax,[ebp-0008]
.00509EC5: 8B55FC                mov     edx,[ebp-0004]
.00509EC8: E86FA0EFFF            call    .000403F3C      /(2)
.00509ECD: 0F85A9000000          jne     .000509F7C      /这里使其不跳
.00509ED3: 8B45FC                mov     eax,[ebp-0004]
.00509ED6: E8718CFAFF            call    .0004B2B4C      /(3)
.00509EDB: 84C0                  test    al,al
.00509EDD: 750B                  jne     .000509EEA      /这里使其不跳
.00509EDF: 55                    push    ebp
.00509EE0: E81FFBFFFF            call    .000509A04      /(4)
.00509EE5: 59                    pop     ecx
.00509EE6: 84C0                  test    al,al
.00509EE8: 752D                  jne     .000509F17      /这里使其跳
.00509EEA: C605582F520001        mov     b,[000522F58],001
.00509EF1: 6A00                  push    000
.00509EF3: 6A00                  push    000
.00509EF5: 6875040000            push    000000475
.00509EFA: A1482F5200            mov     eax,[000522F48]
.00509EFF: E8E003F3FF            call    .00043A2E4
```

附带说一下，跟踪时可见到程序中一个不小的黑名单。另外，如果只 Patch (1)(2) 两处，你可以试试会发生什么。:) 不脱壳直接对程序进行 Patch 也行，具体做法可参考“侠客之家”IceBird 的教程。

原来它对名字还有些限制，昨天没仔细看，还以为是有假注册码，被它骗了。

1、它要求名字中必须有空格，即名字由 `First Name` 和 `Last Name` 构成，且以空格格开；

2、名字的第一个字母必须大写，最好把 `First Name` 和 `Last Name` 的第一个字母都大写；

3、名字的长度要大于等于 4；

4、如冰毒所说的，名字的前几个字母不要和黑名单中的重合了，即如果黑名单中的名字是你的名字的子串，就 begger off。

它用同一个子程序来判断注册码。

第一次调用这个子程序把 `INI` 文件中的注册码密文变成明文；

第二次调用这个子程序把 `INI` 文件中的名字的密文变成明文；

第三次则调用该子程序对注册码的明文进行变换，如果变换的结果等于你的名字，则 OK，否则 game over。

该段子程序如下，用 `SoftICE` 跟一下就清楚了。

```
0167:0047DF3B LEA     EAX,[EBP-2C]
0167:0047DF3E PUSH    EAX
0167:0047DF3F MOV     ECX,00000002
0167:0047DF44 MOV     EDX,00000001
0167:0047DF49 MOV     EAX,[EBP-04]
0167:0047DF4C CALL    00404030
0167:0047DF51 MOV     ECX,[EBP-2C]
0167:0047DF54 LEA     EAX,[EBP-28]
0167:0047DF57 MOV     EDX,0047E064
0167:0047DF5C CALL    00403E78
0167:0047DF61 MOV     EAX,[EBP-28]
0167:0047DF64 CALL    00408D84
0167:0047DF69 MOV     EDI,EAX             //两个字符转换成一个数 a
0167:0047DF6B MOV     DWORD PTR [EBP-14],00000003
0167:0047DF72 LEA     EAX,[EBP-2C]
0167:0047DF75 PUSH    EAX
0167:0047DF76 MOV     ECX,00000002
0167:0047DF7B MOV     EDX,[EBP-14]
0167:0047DF7E MOV     EAX,[EBP-04]
0167:0047DF81 CALL    00404030
0167:0047DF86 MOV     ECX,[EBP-2C]
0167:0047DF89 LEA     EAX,[EBP-28]
0167:0047DF8C MOV     EDX,0047E064
0167:0047DF91 CALL    00403E78
0167:0047DF96 MOV     EAX,[EBP-28]
0167:0047DF99 CALL    00408D84
0167:0047DF9E MOV     EBX,EAX             //两个字符转换成一个数 b
0167:0047DFA0 CMP     ESI,[EBP-0C]
0167:0047DFA3 JGE     0047DFA8
0167:0047DFA5 INC     ESI
0167:0047DFA6 JMP     0047DFAD
0167:0047DFA8 MOV     ESI,00000001
0167:0047DFAD MOV     EAX,[EBP-08]
0167:0047DFB0 MOVZX   EAX,BYTE PTR [ESI+EAX-01]  //取出一个常数
0167:0047DFB5 XOR     EAX,EBX             //异或
0167:0047DFB7 MOV     [EBP-18],EAX
0167:0047DFBA CMP     EDI,[EBP-18]       //根据大小做不同处理
0167:0047DFBD JL      0047DFCE
0167:0047DFBF MOV     EAX,[EBP-18]
0167:0047DFC2 ADD     EAX,000000FF
0167:0047DFC7 SUB     EAX,EDI
0167:0047DFC9 MOV     [EBP-18],EAX
0167:0047DFCC JMP     0047DFD1
0167:0047DFCE SUB     [EBP-18],EDI
0167:0047DFD1 LEA     EAX,[EBP-28]
0167:0047DFD4 MOV     EDX,[EBP-18]
0167:0047DFD7 CALL    00403D54
0167:0047DFDC MOV     EDX,[EBP-28]
0167:0047DFDF LEA     EAX,[EBP-10]
0167:0047DFE2 CALL    00403E34
0167:0047DFE7 MOV     EDI,EBX
0167:0047DFE9 ADD     DWORD PTR [EBP-14],02
0167:0047DFED MOV     EAX,[EBP-04]
0167:0047DFF0 CALL    00403E2C
0167:0047DFF5 CMP     EAX,[EBP-14]
0167:0047DFF8 JG      0047DF72            //循环
0167:0047DFFE MOV     EAX,[EBP+08]
0167:0047E001 MOV     EDX,[EBP-10]       //变换后的串
0167:0047E004 CALL    00403C48
```

日期、名字、注册码在注册表中和 `INI` 文件中都记录了。

如下是注册机。一个可用的注册码：

`Name: China Cracker`

`Code: 832E09019806CDA317F3D468CEA9`

```c
#include <stdio.h>

#include <string.h>

#include <windows.h>

void main(void)

{

    char table[] = { 0xE8, 0x9F, 0x73, 0xF7, 0xFF, 0xEB, 0xB2,
                     0x01, 0x8B, 0x83, 0x28, 0x03, 0xE8, 0x16,
                     0x26, 0xFA};
    unsigned char name[120];
    signed long code[128];
    int k;
    int len;
    signed long temp, code1, code2;
    printf("Input your name: ");
    gets(name);
    len = strlen(name);
    //用 GetTickCount( )模拟随机数
    code[0] = GetTickCount() & 0x000000FFL;
    for (k = 0; k < len; k++)
    {
        temp = code[k] + (long)name[k];
        if (temp > 0xFF)
        {
            code[k+1] = (temp - 0x000000FFL) ^ ((long)table[k] & 0x000000FFL);
        }
        else
        {
            code[k+1] = temp ^ ((long)table[k] & 0x000000FFL);
        }
    }
    printf("Your code is: ");
    for (k = 0; k < (len+1); k++)
    {
        printf("%02X", code[k] & 0xFF);
    }
    printf("\n");
}
```
