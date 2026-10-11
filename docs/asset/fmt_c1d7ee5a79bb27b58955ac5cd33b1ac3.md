
```
008E9378   .  0AC0                     OR AL,AL
008E937A    75 28                     JNZ SHORT X2.008E93A4              ; // 已经计算过了
008E937C    90                        NOP
008E937D    90                        NOP
008E937E    90                        NOP
008E937F    90                        NOP
008E9380    8B85 26D04000             MOV EAX,DWORD PTR SS:[EBP+40D026]  ; // 400000
008E9386    8B70 3C                  MOV ESI,DWORD PTR DS:[EAX+3C]      ; // PE 头 RVA
008E9389    03B5 26D04000            ADD ESI,DWORD PTR SS:[EBP+40D026]  ; // PE 头 VA
008E938F    83C6 28                   ADD ESI,28                         ; // OEP  VA
008E9392    AD                        LODS DWORD PTR DS:[ESI]           ; // OEP
008E9393    8AD8                      MOV BL,AL
008E9395    02DC                      ADD BL,AH
008E9397    C1E8 10                   SHR EAX,10
008E939A    02D8                      ADD BL,AL
008E939C    02DC                      ADD BL,AH
008E939E    889D 78214000             MOV BYTE PTR SS:[EBP+402178],BL    ; // OEP 四字节之和解密用
008E93A4    59                        POP ECX
008E93A5    5E                        POP ESI
008E93A6    60                        PUSHAD
008E93A7    B8 02000000               MOV EAX,2
008E93AC    E8 B5BB0000               CALL X2.008F4F66
008E93B1    0BC0                      OR EAX,EAX
008E93B3    75 24                     JNZ SHORT X2.008E93D9              ; // 为 0 则解密到堆中
008E93B5    90                        NOP
008E93B6    90                        NOP
008E93B7    90                        NOP
008E93B8    90                        NOP
008E93B9    61                        POPAD
008E93BA    8BBD 2AD04000             MOV EDI,DWORD PTR SS:[EBP+40D02A]   ; // 8F402A 放 1346F8, 指向解密后的代码
008E93C0    B8 0A000000               MOV EAX,0A
008E93C5    F7E1                      MUL ECX
008E93C7    03F8                      ADD EDI,EAX
008E93C9    B9 0A000000               MOV ECX,0A
008E93CE    8A9D 78214000             MOV BL,BYTE PTR SS:[EBP+402178]
008E93D4    EB 11                     JMP SHORT X2.008E93E7
008E93D6    90                        NOP
008E93D7    90                        NOP
008E93D8    90                        NOP
008E93D9    61                        POPAD
008E93DA    8BFE                      MOV EDI,ESI                        
008E93DC    B9 0A000000               MOV ECX,0A
008E93E1    8A9D 78214000             MOV BL,BYTE PTR SS:[EBP+402178]    ; // 取出解密字节
008E93E7    AC                        LODS BYTE PTR DS:[ESI]             
008E93E8    32C3                      XOR AL,BL
008E93EA    AA                        STOS BYTE PTR ES:[EDI]
008E93EB   ^ E2 FA                   LOOPD SHORT X2.008E93E7            ; // 解密
008E93ED    83EF 0A                   SUB EDI,0A
008E93F0    57                        PUSH EDI
008E93F1    8B7424 24                MOV ESI,DWORD PTR SS:[ESP+24]      ; // 取出返回地址
008E93F5    83EE 04                   SUB ESI,4                          ; // CALL 08E9188
008E93F8    AD                        LODS DWORD PTR DS:[ESI]          
008E93F9    81EF 88214000             SUB EDI,X2.00402188
008E93FF    2BFD                      SUB EDI,EBP
008E9401    03C7                      ADD EAX,EDI                        ; // 地址转换成偏移量
008E9403    8946 FC                   MOV DWORD PTR DS:[ESI-4],EAX       ; // 变成 CALL 1346F8 + ECX * A
008E9406    5F                        POP EDI
008E9407    57                        PUSH EDI
008E9408    33C9                      XOR ECX,ECX
008E940A    83F9 08                   CMP ECX,8
008E940D    74 0E                     JE SHORT X2.008E941D
008E940F    90                        NOP
008E9410    90                        NOP
008E9411    90                        NOP
008E9412    90                        NOP
008E9413    8B448C 04                MOV EAX,DWORD PTR SS:[ESP+ECX*4+4]
008E9417    89048C                    MOV DWORD PTR SS:[ESP+ECX*4],EAX
008E941A    41                        INC ECX
008E941B   ^ EB ED                   JMP SHORT X2.008E940A
008E941D   . 893C8C                   MOV DWORD PTR SS:[ESP+ECX*4],EDI
008E9420   . 60                       PUSHAD
008E9421   . E8 00000000              CALL X2.008E9426
008E9426  /$ 5E                       POP ESI
008E9427  |. 83EE 06                  SUB ESI,6
008E942A  |. B9 ED000000              MOV ECX,0ED
008E942F  |. 29CE                     SUB ESI,ECX
008E9431  |. BA F0316F46              MOV EDX,466F31F0
008E9436  |. C1E9 02                  SHR ECX,2
008E9439  |. 83E9 02                  SUB ECX,2
008E943C  |>  83F9 00                 /CMP ECX,0
008E943F  |.  7C 1A                  |JL SHORT X2.008E945B
008E9441  |.  8B048E                 |MOV EAX,DWORD PTR DS:[ESI+ECX*4]
008E9444  |.  8B5C8E 04              |MOV EBX,DWORD PTR DS:[ESI+ECX*4+4]
008E9448  |.  2BC3                   |SUB EAX,EBX
008E944A  |.  C1C0 02                |ROL EAX,2
008E944D  |.  33C2                   |XOR EAX,EDX
008E944F  |.  81C2 B006C8EC          |ADD EDX,ECC806B0
008E9455  |.  89048E                 |MOV DWORD PTR DS:[ESI+ECX*4],EAX
008E9458  |.  49                     |DEC ECX
008E9459  |.^ EB E1                  JMP SHORT X2.008E943C
008E945B  |>  61                     POPAD
008E945C  |.  61                     POPAD
008E945D  .  C3                     RETN
```

从上面我们可以看到：

1. 恢复 `OEP` 为 `8E8000`, 解密代码才正确
2. 就地解密, 用 `LordPE` 修改 `VA=8F402A`, 原来 `1346F8`, 改成 `8EC33E`

改好后再次运行，一走棋就出错，`8F4F26`：

```
008F4F26   .  F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
```

寄存器状态：

* `ECX` `00003445`
* `ESI` `004090AB` `X2.004090AB`
* `EDI` `001455E8`
* `EIP` `008F4F26` `X2.008F4F26`

检查后知是程序试图向堆中不存在的地址作串传输。

到内存镜像一看，`130000` 段是从堆里动态分配的，大小 `9000`。

用未脱壳程序运行，大小是 `1F000`。

`1F000-9000=16000`, 是在壳代码中分配的。

参考上面 `8F71B3` 中分配内存的函数。

在 `8E8000` 补上下面的语句即可：

```
mov  eax, 8F45C9
mov  [4BB4C4],eax
push 16000
push 40
call [4BB2B0]       //kerner32.GlobalAlloc (7C58DA27)
jmp  49BF0A
```

`OK`, 现在可以走棋，但一思考就出错，跟 `RSA_Lock_Code` 和 `Embedded_Protect` 有关。

=====================================================================================================================

## `MessageBox` `SDK` 说明

引用 `ACProtect` 文档：

```
#define GetRegistrationName(szRegistrationName)  ::MessageBox (HWND(-1), szRegistrationName, NULL, 0);

#define GetTrialUsageTimes(lpUsageTime)          ::MessageBox (HWND(-1), lpUsageTime     , NULL, 1);  
```

以下分析一个用了 `Dynamic_encrypt`, `Embedded_Protect`, `RSA_Lock_Code` 的 `VC6` 程序：

```cpp
#include "windows.h"

#include "stdlib.h"

#include "stdio.h"

#include "ACProtect.h"

int main(int argc, char* argv[])
{
  //No 1 dynamic encrypt
```
