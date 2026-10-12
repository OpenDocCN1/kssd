
# Touch Typing version 98 3.3

> 原文：[`www.pediy.com/kssd/pediy01/forum080.htm`](https://www.pediy.com/kssd/pediy01/forum080.htm)

`www.shetef.com`

这个软件的主页上还有个注册表监视工具和文件读写监视工具。`Touch Typing` 是用 `VB5` 写的。破解的关键在于 `__vbaStrComp()` 和 `rtcR8ValFromBStr()`。

```c
#include <stdio.h>
#include <string.h>

void main(void)
{
    char SerialStr[64]="Touch-Typing_000";
    char PasswordStr[64]="PA00";
    char buf[64];
    long serial;
    long password;

    printf("KeyGen for Touch Typing version 98 3.3.\n");
    printf("Input a magic number: ");
    scanf("%lu", &password);
    sprintf(buf,"%lu", password);
    buf[6] = '\0';
    sscanf(buf, "%lu", &password);
    strcat(PasswordStr, buf);
    serial = password * 142L + 48935L;
    sprintf(buf, "%lu", serial);
    strcat(SerialStr, buf);
    printf("Your serial:  ");
    puts(SerialStr);
    printf("Your password: ");
    puts(PasswordStr);
}
```

另外，在 `SoftICE` 中敲 `wf` 命令可以把 `x86` 的浮点寄存器栈窗口打开，便于观察浮点运算的过程。`rtcR8ValFromBStr()` 负责把一个字符串转换成一个浮点数，在 `rtcR8ValFromBStr()` 的函数体的尾部会把转换得到的浮点数用 `FLD` 指令加载到浮点寄存器中，如下所示，在执行完指令 `FLD REAL8 PTR [EBP-08]` 后敲“`dl ebp-8`”命令可以看见局部变量 `[ebp-8]` 中存放的是转换后的浮点数，且浮点寄存器 `ST0` 中也存放该数。然后跟踪，看它如何处理 `ST0` 中的这个数即可。

```asm
MSVBVM50!rtcR8ValFromBstr

0167:7998D1FE  PUSH      EBP
0167:7998D1FF  MOV       EBP,ESP
0167:7998D201  SUB       ESP,10
0167:7998D204  PUSH      ESI
0167:7998D205  PUSH      EDI
0167:7998D206  MOV       EDI,[EBP+08]
0167:7998D209  TEST      EDI,EDI
0167:7998D20B  JNZ       7998D21B
0167:7998D20D  FLD       REAL8 PTR [7999C918]
0167:7998D213  POP       EDI
0167:7998D214  POP       ESI
0167:7998D215  MOV       ESP,EBP
0167:7998D217  POP       EBP
0167:7998D218  RET       0004
0167:7998D21B  PUSH      EDI
0167:7998D21C  CALL      [798B19E4]
0167:7998D222  MOV       ECX,EAX
0167:7998D224  MOV       ESI,EAX
0167:7998D226  MOV       AX,[EDI]
0167:7998D229  TEST      AX,AX
0167:7998D22C  JZ        7998D25D
0167:7998D22E  CMP       AX,0009
0167:7998D232  JZ        7998D252
0167:7998D234  CMP       AX,000A
0167:7998D238  JZ        7998D252
0167:7998D23A  CMP       AX,000D
0167:7998D23E  JZ        7998D252
0167:7998D240  CMP       AX,0020
0167:7998D244  JZ        7998D252
0167:7998D246  CMP       AX,3000
0167:7998D24A  JZ        7998D252
0167:7998D24C  MOV       [ECX],AX
0167:7998D24F  ADD       ECX,02
0167:7998D252  ADD       EDI,02
0167:7998D255  MOV       AX,[EDI]
0167:7998D258  TEST      AX,AX
0167:7998D25B  JNZ       7998D22E
0167:7998D25D  MOV       [ECX],AX
0167:7998D260  PUSH      05
0167:7998D262  LEA       EAX,[EBP-10]
0167:7998D265  MOV       [EBP-08],ESI
0167:7998D268  MOV       WORD PTR [EBP-10],0008
0167:7998D26E  PUSH      EAX
0167:7998D26F  CALL      7998D657
0167:7998D274  MOV       EDI,EAX
0167:7998D276  PUSH      ESI
0167:7998D277  CALL      [798B1988]
0167:7998D27D  TEST      EDI,EDI
0167:7998D27F  JGE       7998D289
0167:7998D281  PUSH      00
0167:7998D283  PUSH      EDI
0167:7998D284  CALL      798BB385
0167:7998D289  FLD       REAL8 PTR [EBP-08]  //这里是在取转换结果
0167:7998D28C  JMP       7998D213
```
