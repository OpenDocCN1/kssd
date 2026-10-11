
```
0032F98D    8B1B             MOV EBX,DWORD PTR DS:[EBX]

0032F98F    C601 68          MOV BYTE PTR DS:[ECX],68

0032F992    8959 01          MOV DWORD PTR DS:[ECX+1],EBX

0032F995    C641 05 C3       MOV BYTE PTR DS:[ECX+5],0C3

0032F999    83C1 06          ADD ECX,6

0032F99C    66:8139 FF25     CMP WORD PTR DS:[ECX],25FF

0032F9A1    75 04            JNZ SHORT 0032F9A7

0032F9A3    3BCF             CMP ECX,EDI

0032F9A5   ^ 72 CE            JB SHORT 0032F975

0032F9A7    83C1 02          ADD ECX,2

0032F9AA    66:8139 FF25     CMP WORD PTR DS:[ECX],25FF

0032F9AF   ^ 74 D9            JE SHORT 0032F98A

0032F9B1    83C1 24          ADD ECX,24

0032F9B4    66:8139 FF25     CMP WORD PTR DS:[ECX],25FF

0032F9B9   ^ 74 CF            JE SHORT 0032F98A

0032F9BB    61               POPAD

0032F9BC    E8 0A84FFFF      CALL 00327DCB

...

00327DD2    5D               POP EBP

00327DD3    81ED 2E514000    SUB EBP,40512E

00327DD9    83BD F3D24000 0>CMP DWORD PTR SS:[EBP+40D2F3],0

00327DE0    74 33            JE SHORT 00327E15                   ; 强行跳过驱动

00327DE2    60               PUSHAD

00327DE3    CD20 8BC48B64    VxDJump 648BC48B

00327DE9    24 08            AND AL,8

00327DEB    50               PUSH EAX

00327DEC    60               PUSHAD

00327DED    56               PUSH ESI

00327DEE    0F014C24 FE      SIDT FWORD PTR SS:[ESP-2]

00327DF3    5E               POP ESI

00327DF4    66:8B85 0E51400>MOV AX,WORD PTR SS:[EBP+40510E]

00327DFB    66:8B9D 1051400>MOV BX,WORD PTR SS:[EBP+405110]

00327E02    66:8946 18       MOV WORD PTR DS:[ESI+18],AX

00327E06    66:895E 1E       MOV WORD PTR DS:[ESI+1E],BX

00327E0A    61               POPAD

00327E0B    5C               POP ESP

00327E0C    8D85 71514000    LEA EAX,DWORD PTR SS:[EBP+405171]

00327E12    50               PUSH EAX

00327E13    CF               IRETD

00327E14    61               POPAD

00327E15    61               POPAD

00327E16    C3               RETN

...

0032FB10    60               PUSHAD

0032FB11    8B85 BD890000    MOV EAX,DWORD PTR SS:[EBP+89BD]

0032FB17    0385 C5890000    ADD EAX,DWORD PTR SS:[EBP+89C5]

0032FB1D    E8 00000000      CALL 0032FB22

0032FDFB    50               PUSH EAX                           ; 17x.00403A3F

0032FDFC    8D78 02          LEA EDI,DWORD PTR DS:[EAX+2]

0032FDFF    EB 02            JMP SHORT 0032FE03

0032FE01    61               POPAD

0032FE02    50               PUSH EAX

0032FE03    C3               RETN                               ; to OEP

00403A3F    FFD7             CALL EDI                           ; 17x.00403A41

00403A41    58               POP EAX

00403A42    90               NOP

00403A43    90               NOP

00403A44    E8 FD980000      CALL 17x.0040D346                 ; JMP to comdlg32.GetOpenFileNameA

00403A49    85C0             TEST EAX,EAX
```

前 5 个字节是 Stolen Code, 看一下 STACK,

```
003275D0   0040F034  17x.0040F034
003275D4   004327F9  RETURN to 17x.004327F9
           ;
00403A3F                  PUSH 40F034           ;OK
```

再把`40E170-40E1A7`的数据复制到`40E000-40E037`, Dump

IAT 只要用 `LordPE` 改一下 `RVA = E120, Size=50`

> 引用:
>
> * * *
>
> *最初由 liuyilin 发布*
>
> **请教：armadillo，copy-II 能用这种思路脱吗？？ **
>
> * * *

调试壳, 运行后有两个进程, 这类壳的大致流程如下:

1. 父进程 `CreateProcess` 创建子进程, 再为子进程做个被调试标志
2. 子进程在父进程调试下开始运行, 由于标志, 走不同的路线
3. 父进程 用 `WaitForDebugEvent` 接受子进程产生的调试事件, 采取响应的措施(比如 `ARM` 的 `CC`)
4. 父进程处理好后, 调用 `ContinueDebugEvent` 继续子进程的运行, 重复 3, 直到结束

要脱这类壳, 关键是用 `OD` 做父进程, 也就是把双进程转换成单进程

1. 被调试标志是如何生成的.
   常见的有 `Mutex`, `FileMapping`, 标志文件, 命令行参数 等

2. 父进程接受子进程产生的调试事件后, 是如何处理的.
   常见的有 `SetThreadContext`, `WriteProcessMemory`, `VirtualProtect` 等

`4dd466d0e1e0cc123afbe504da2462ca`
