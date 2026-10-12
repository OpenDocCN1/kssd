
# WeaveMaker 7.55

> 原文：[`www.pediy.com/kssd/pediy04/kanxue196.htm`](https://www.pediy.com/kssd/pediy04/kanxue196.htm)

用 `trw2000` 装人，先输入 `Name:abcd,Key:1122334`。

`ctrl-n` 进入 `trw2000`，下断点 `bpx getdlgitemtexta; F5;`。

然后再输入“4”，将被拦在 `getdlgitemtexta` 模块中，按数次 `F12`；来到这里：

```
=====================================================================

:004C6798 8D852C010000            lea eax, dword ptr [ebp+0000012C] 《==光标在这里

:004C679E 50                      push eax

:004C679F 6A06                    push 00000006

:004C67A1 E8BAFFFEFF              call 004B6760

:004C67A6 83C410                  add esp, 00000010

:004C67A9 E99E000000              jmp 004C684C



:004C684C 55                      push ebp

:004C684D E84E010000              call 004C69A0 <=========进入

:004C6852 83C404                  add esp, 00000004

:004C6855 85C0                    test eax, eax

:004C6857 7427                    je 004C6880

:004C6859 8B8DEC020000            mov ecx, dword ptr [ebp+000002EC]

======================================================================

:004C69A0 81EC18020000            sub esp, 00000218

:004C69A6 53                      push ebx

:004C69A7 55                      push ebp

:004C69A8 8BAC2424020000          mov ebp, dword ptr [esp+00000224]

:004C69AF 56                      push esi

:004C69B0 57                      push edi

* Possible StringData Ref from Data Obj ->"Demo"
                                        |

:004C69B1 BFD0115300              mov edi, 005311D0

:004C69B6 8DB52C010000            lea esi, dword ptr [ebp+0000012C]

:004C69BC 8BC6                    mov eax, esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C69E0(C)
|

:004C69BE 8A10                    mov dl, byte ptr [eax]

:004C69C0 8A1F                    mov bl, byte ptr [edi]

:004C69C2 8ACA                    mov cl, dl

:004C69C4 3AD3                    cmp dl, bl

:004C69C6 751E                    jne 004C69E6

:004C69C8 84C9                    test cl, cl

:004C69CA 7416                    je 004C69E2

:004C69CC 8A5001                  mov dl, byte ptr [eax+01]

:004C69CF 8A5F01                  mov bl, byte ptr [edi+01]

:004C69D2 8ACA                    mov cl, dl

:004C69D4 3AD3                    cmp dl, bl

:004C69D6 750E                    jne 004C69E6

:004C69D8 83C002                  add eax, 00000002

:004C69DB 83C702                  add edi, 00000002

:004C69DE 84C9                    test cl, cl

:004C69E0 75DC                    jne 004C69BE
```

以上是合法性检查和判断是否为 "Demo"。`F10` 带过……来到这里：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C6A6B(U)
|

:004C6AB0 8D4C241C                lea ecx, dword ptr [esp+1C]

:004C6AB4 51                      push ecx

:004C6AB5 53                      push ebx      <==注册码

:004C6AB6 E855090000              call 004C7410      <=====注意！！！！

:004C6ABB 8D5528                  lea edx, dword ptr [ebp+28]

:004C6ABE 8DB544020000            lea esi, dword ptr [ebp+00000244]

:004C6AC4 52                      push edx

:004C6AC5 8D8594020000            lea eax, dword ptr [ebp+00000294]

:004C6ACB 56                      push esi

:004C6ACC 8D4D3C                  lea ecx, dword ptr [ebp+3C]

:004C6ACF 50                      push eax

:004C6AD0 8D9424AC000000          lea edx, dword ptr [esp+000000AC]

:004C6AD7 51                      push ecx

:004C6AD8 52                      push edx

:004C6AD9 E862050000              call 004C7040

:004C6ADE 8B85EC020000            mov eax, dword ptr [ebp+000002EC]

:004C6AE4 83C41C                  add esp, 0000001C

:004C6AE7 85C0                    test eax, eax

:004C6AE9 7410                    je 004C6AFB

:004C6AEB 8D4C2424                lea ecx, dword ptr [esp+24]

:004C6AEF 51                      push ecx

:004C6AF0 50                      push eax

:004C6AF1 E8DA450000              call 004CB0D0

:004C6AF6 83C408                  add esp, 00000008

:004C6AF9 EB13                    jmp 004C6B0E

============================================================================
```

在 `:004C6AB6` 处将我们输入的注册码转换为数字。并存入 `[ecx]` 中。

接下来的代码是将 `NAME` 变为以下形式：

`22.abcd.Never.22` (其中 `.` 代表 `ascii` 码 `0D`)

`F10` 带过……来到这里：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C6AF9(U)
|

:004C6B0E 8D842460010000          lea eax, dword ptr [esp+00000160]

:004C6B15 8D4C2410                lea ecx, dword ptr [esp+10]

:004C6B19 50                      push eax

:004C6B1A 8D542424                lea edx, dword ptr [esp+24]

:004C6B1E 51                      push ecx

:004C6B1F 52                      push edx

:004C6B20 6A00                    push 00000000

:004C6B22 8D442434                lea eax, dword ptr [esp+34]

:004C6B26 56                      push esi

:004C6B27 8D8C24AC000000          lea ecx, dword ptr [esp+000000AC]

:004C6B2E 50                      push eax

:004C6B2F 51                      push ecx

:004C6B30 E88B060000              call 004C71C0 <=========产生注册码

:004C6B35 8B44242C                mov eax, dword ptr [esp+2C]

:004C6B39 8B4C2438                mov ecx, dword ptr [esp+38]

:004C6B3D 83C41C                  add esp, 0000001C

:004C6B40 3BC8                    cmp ecx, eax <======注册码比较 eax 为真正的注册码

:004C6B42 8985F0020000            mov dword ptr [ebp+000002F0], eax

:004C6B48 898DF4020000            mov dword ptr [ebp+000002F4], ecx

:004C6B4E 7510                    jne 004C6B60

:004C6B50 5F                      pop edi

:004C6B51 5E                      pop esi

:004C6B52 5D                      pop ebp

:004C6B53 B801000000              mov eax, 00000001

:004C6B58 5B                      pop ebx

:004C6B59 81C418020000            add esp, 00000218

:004C6B5F C3                      ret

============================================================================
```

在 `:004C6B40` 处下命令：`? eax` 可得到真正的注册码：`8b4a8a5f` (`HEX`) 即 `2336918111` (`DEX`).

以下是注册算法：

```
=============================================================================

:004CBC00 56                      push esi

:004CBC01 8B74240C                mov esi, dword ptr [esp+0C]

:004CBC05 83C8FF                  or eax, FFFFFFFF

:004CBC08 85F6                    test esi, esi

:004CBC0A 7E24                    jle 004CBC30

:004CBC0C 8B4C2408                mov ecx, dword ptr [esp+08]

:004CBC10 57                      push edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CBC2D(C)
|

:004CBC11 33D2                    xor edx, edx

:004CBC13 8BF8                    mov edi, eax

:004CBC15 8A11                    mov dl, byte ptr [ecx] <== [ecx]为用户名:

:004CBC17 81E7FF000000            and edi, 000000FF          "22.abcd.Never.22"

:004CBC1D 33D7                    xor edx, edi

:004CBC1F C1E808                  shr eax, 08

:004CBC22 8B1495D4145300          mov edx, dword ptr [4*edx+005314D4] <==查表

:004CBC29 33C2                    xor eax, edx

:004CBC2B 41                      inc ecx

:004CBC2C 4E                      dec esi

:004CBC2D 75E2                    jne 004CBC11

:004CBC2F 5F                      pop edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CBC0A(C)
|

:004CBC30 F7D0                    not eax

:004CBC32 5E                      pop esi

:004CBC33 C3                      ret
```

先更正一个笔误：程序将用户名转化为 `22xxxx.Never.22` (其中“`.`”为 `ASCII` 码 `0x0d`，`xxxx` 为用户名）。

在 `004CBC22` 处设断点，中断后，下命令 `w 5314D4 l 400 c:\table.bin` `dump` 出它的加密表。（因为我懒得自己输入，呵呵……）



下面代码编译后，将生成的`exe`文件和`table.bin`放在同一目录下，即可。

```c
#include <io.h>   
#include <stdlib.h>   
#include <stdio.h>   
#include <string>   

unsigned long Table[256];   

char Never[]={0x0d,'N','e','v','e','r',0x0d,'2','2',0};   

char Name[255];   

char YourName[40];   

int main(int argc, char* argv[])   

{   

    unsigned long eax,edi,edx;   

    FILE *fp;   

    fp = fopen(".\\table.bin","rb");   

    if (fp==NULL)   

        return 0;   

    fread(Table,sizeof(unsigned long),256,fp);   

    fclose(fp);   

    printf("Wrtie By leo_cyl\n");   

    printf("input your name :");   

    scanf("%s",YourName);   

    Name[0]=Name[1]='2';   

    int len = strlen(YourName);   

    for (int i=0;i<len;i++)   

        Name[i+2]=YourName[i];   

    for(i=0;i<strlen(Never);i++)   

        Name[len+2+i]=Never[i];   

    Name[len+2+i]=0;   

    eax=0xffffffff;   

    for(i=0;i<strlen(Name);i++) {   

        edx=0;   

        edi=eax;   

        edi=edi&0xff;   

        edx = (unsigned long)Name[i];   

        edx=edx^edi;   

        eax=eax>>8;   

        edx=Table[edx];   

        eax=eax^edx;   

    }   

    eax=~eax;   

    printf("Key= %u\n",eax);   

    return 0;   

}
```

4dd466d0e1e0cc123afbe504da2462ca
