
``` 
//bp 499EDA, F9, 第一次中断

00499EDA  CALL CX3.008DE0DD

// F8后变成

00499EDA  CALL 00135496

00135496    8B43 10         MOV EAX,DWORD PTR DS:[EBX+10]

00135499    33E6            XOR ESP,ESI

0013549B    33E6            XOR ESP,ESI

0013549D    8B10            MOV EDX,DWORD PTR DS:[EAX]

0013549F    C3              RETN

// 说明第一次解密正常

// F9, 第二次中断在00499EDA, F8, 出错

// 到 00135496 一看, 刚才解密的代码不见了

// 我跟过没脱壳原程序, 第二次时代码还在

// 可能是检测不到壳, 把那块内存清零了

// 解决办法,用 UltraEdit 打开CX3.EXE, 找到 08 47 13 00, (只有一处)

// 改成 93 12 8E 00, 叫他就地解密, 嘿嘿.

// 用 OD 载入CX3.EXE还是出错, Shit, Trace into again, 这次时间要1分钟吧

// 错在 414272 CALL MessageBoxA 处

// 难道是 IAT 修复的有问题?  检查了半天, 也没有搞定.

// 再仔细看看 Trace into 的记录, 00414044 ---> 00414272

// 用OD 载入未脱壳原程序, 先到OEP 495FAD

// bp 414044, F9, 断下, F7走

// 又是一大段花指令, SMC , 循环,

00414044

004141FC    83C4 04         ADD ESP,4

004141FF    66:81F1 D343    XOR CX,43D3

00414204    F8              CLC

00414205    4E              DEC ESI

00414206  ^ 0F85 88FFFFFF   JNZ XieXieMa.00414194                    ; // 循环

0041420C    E8 01000000     CALL XieXieMa.00414212                   ; // F4 到这里

00414212    83C4 04         ADD ESP,4

00414215    E9 04000000     JMP XieXieMa.0041421E

0041421A    66:BB 1EBA      MOV BX,0BA1E

0041421E    61              POPAD

0041421F    8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]

00414222    6A 03           PUSH 3

00414224    E8 8F3FFFFF     CALL XieXieMa.004081B8                   ; // F8

00414229    C745 FC 0000000>MOV DWORD PTR SS:[EBP-4],0

00414230    8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]

00414233    6A 01           PUSH 1

00414235    E8 F640FFFF     CALL XieXieMa.00408330                   ; // F8

0041423A    8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]

0041423D    68 A8717F00     PUSH XieXieMa.007F71A8                   ; ASCII "XieXieMaster 1.0.10"

00414242    68 A0717F00     PUSH XieXieMa.007F71A0                   ; ASCII "XieXie"

00414247    E8 2441FFFF     CALL XieXieMa.00408370                   ; // F8

0041424C    6A 10           PUSH 10

0041424E    FF15 B8544B00   CALL DWORD PTR DS:[4B54B8]               ; // F8, 脱壳后是 GetAsyncKeyState

00414254    0FBFC0          MOVSX EAX,AX

00414257    85C0            TEST EAX,EAX

00414259    0F8C D5010000   JL XieXieMa.00414434

0041425F    C705 041B4C00 0>MOV DWORD PTR DS:[4C1B04],0

00414269    60              PUSHAD

0041426A    6A 02           PUSH 2

0041426C    6A 00           PUSH 0

0041426E    6A 00           PUSH 0

00414270    6A FF           PUSH -1

00414272    FF15 C4544B00   CALL DWORD PTR DS:[4B54C4]               ; XieXieMa.008DDB6D,  这里脱壳后成了 MessageBoxA,

                                                                     // 但未脱壳程序不是, F7 进去看看, 发现来到了壳代码,奇怪???

008E951E    60              PUSHAD

008E951F    F8              CLC

008E9520    7A 03           JPE SHORT XieXieMa.008E9525

008E9522    7B 01           JPO SHORT XieXieMa.008E9525

008E9524    7A D3           JPE SHORT XieXieMa.008E94F9

// 请看 Acprotect.h (附件中), 各位看官就会明白的...

// 原来这个程序使用了 ACProtect 的 EMbedded protector 技术

// 该技术具体见 Acprotect 的帮助

// 我的解决办法, 再次修改 008DD000 代码, 如下, 也去调用壳代码

008DD000 >  B8 1E958E00     MOV EAX,008E951E

008DD005    8705 C4544B00   XCHG DWORD PTR DS:[<&user32.MessageBoxA>],EAX

008DD00B  - E9 988FBBFF     JMP 00495FA8

// CX3.EXE 终于可以运行了, 不过别高兴得太早.

// 只要你一走棋, 或点注册, game over...

// 用OD 载入CX3.EXE , F9运行, 点注册"OK", 出错

008E9E5C    8BCE            MOV ECX,ESI

008E9E5E    8BB48D EC1B4000 MOV ESI,DWORD PTR SS:[EBP+ECX\*4+401BEC]

008E9E65    03B5 7BCF4000   ADD ESI,DWORD PTR SS:[EBP+40CF7B]

008E9E6B    8B948D 7C1D4000 MOV EDX,DWORD PTR SS:[EBP+ECX\*4+401D7C]

008E9E72    8BBC8D 0C1F4000 MOV EDI,DWORD PTR SS:[EBP+ECX\*4+401F0C]

008E9E79    87CA            XCHG EDX,ECX

008E9E7B    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]     ; // 内寸访问错误

008E9E7D    60              PUSHAD

// 这时 ECX=3445, ESI=004090AF, EDI=00142348

// 出错的地方从 004090AF 拷贝 3445 字节到 00142348

// 到内存镜像一看

Memory map, item 4

 Address=00130000       Size=0000B000 (45056.)        Owner=         00130000 (itself)

 Section=               Type=Priv 00021004            Access=RW     Initial access=RW

// 程序只对 130000 到 13B000-1 有读写权限, 不能访问 00142348

// 跟踪未脱壳原程序,  Size=0001C000

// 那我们就在有读写权限的地方加点空间

// 用LordPE 打开 CX3.EXE,  修改 .mackt 段的 VSize 为 10000, 原来 2000,

// .mackt 的 VA = 00903000

// 新加空间对应的 VA = 00905000

// 用 UltraEdit 打开CX3.EXE, 找到 48 23 14 00, (只有一处)

// 注意前面还有 0013BC40, 0013EF78, 不清楚用处, 一并修改

// 改成 00 50 90 00 ,00 90 90 00, 00 D0 90 00

// 呵呵, 现在再点注册, 弹出一个"Incorret Register Code"的对话框

// 跟未脱壳原程序一样了.

// 这里我还发现了一个Bug (或许就是这样设计的)

// 你一走棋, 叫你注册, 你点OK, "Incorrect Register code", 你再走棋

// 几次反复后, 程序出错, 一开始以为脱壳不对, 后来试了原程序也是这样.

// 下面开始暴破, 对 MessageBoxA  下断

// 第一个关键比较

// 注册对话框, 点注册, 断在 4A5232, 看看STACK

0012F9DC   001A0146  |hOwner = 001A0146 ('XieXieMaster 1.0.10> Unregist...',class='AfxFrameOrView42s')

0012F9E0   007FE5AC  |Text = "Incorrect registration code !"

0012F9E4   007FCE3C  |Title = "XieXie Error"

0012F9E8   00000030  Style = MB\_OK|MB\_ICONEXCLAMATION|MB\_APPLMODAL

0012F9EC   0012FB60

0012F9F0   00418FED  RETURN to cx3.00418FED from cx3.004A520A

0012F9F4   007FE5AC  ASCII "Incorrect registration code !"

0012F9F8   007FCE3C  ASCII "XieXie Error"

0012F9FC   00000030

// 418FED, 还不快去

00418FB4    50              PUSH EAX

00418FB5    68 74CD7F00     PUSH cx3.007FCD74                        ; ASCII "Registration code"

00418FBA    68 64CD7F00     PUSH cx3.007FCD64                        ; ASCII "Registration"

00418FBF    E8 C8F4FEFF     CALL cx3.0040848C

00418FC4    E8 C3AAFFFF     CALL cx3.00413A8C                        ; // 关键CALL?

00418FC9    A1 081B4C00     MOV EAX,DWORD PTR DS:[4C1B08]            ; // 关键比较 1

00418FCE    85C0            TEST EAX,EAX                             ; // EAX = 0, Over

00418FD0    75 1B           JNZ SHORT cx3.00418FED

00418FD2    A1 20F14B00     MOV EAX,DWORD PTR DS:[4BF120]

00418FD7    8B4D EC         MOV ECX,DWORD PTR SS:[EBP-14]

00418FDA    6A 30           PUSH 30

00418FDC    68 3CCE7F00     PUSH cx3.007FCE3C                        ; ASCII "XieXie Error"

00418FE1    FF3485 B0EF4B00 PUSH DWORD PTR DS:[EAX\*4+4BEFB0]

00418FE8    E8 1DC20800     CALL cx3.004A520A                        ; // 显示"Incorrect Register Code"

00418FED    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]

00418FF0    8B15 041B4C00   MOV EDX,DWORD PTR DS:[4C1B04]

// 第二个关键比较, 00417C03 处还有调用 MessageBoxA

00417BFE  |.  50            PUSH EAX                                 ; /Style => MB\_OK|MB\_APPLMODAL

00417BFF  |.  50            PUSH EAX                                 ; |Title => NULL

00417C00  |.  52            PUSH EDX                                 ; |Text

00417C01  |.  6A FF         PUSH -1                                  ; |hOwner = FFFFFFFF

00417C03  |.  FF15 C4544B00 CALL DWORD PTR DS:[<&user32.MessageBoxA>>; MessageBoxA

00417C09  |.  8A0424        MOV AL,BYTE PTR SS:[ESP]

00417C0C  |.  84C0          TEST AL,AL                               ; // 关键比较 2

00417C0E  |.  74 37         JE SHORT cx4.00417C47

00417C10  |.  8D9424 E80400>LEA EDX,DWORD PTR SS:[ESP+4E8]

00417C17  |.  8D0424        LEA EAX,DWORD PTR SS:[ESP]

00417C1A  |.  50            PUSH EAX

00417C1B  |.  FF35 84EB4B00 PUSH DWORD PTR DS:[4BEB84]               ;  cx4.007F71A8

00417C21  |.  68 E0CD7F00   PUSH cx4.007FCDE0                        ;  ASCII "%s>  Licensed to %s"

00417C26  |.  52            PUSH EDX

00417C27  |.  E8 66D90700   CALL cx4.00495592

00417C2C  |.  83C4 10       ADD ESP,10

00417C2F  |.  8D8424 E80400>LEA EAX,DWORD PTR SS:[ESP+4E8]

00417C36  |.  8BCF          MOV ECX,EDI

00417C38  |.  50            PUSH EAX

00417C39  |.  E8 E0B60800   CALL cx4.004A331E

00417C3E  |.  81C4 D0080000 ADD ESP,8D0

00417C44  |.  5E            POP ESI

00417C45  |.  5F            POP EDI

00417C46  |.  C3            RETN

00417C47  |>  8D8424 000100>LEA EAX,DWORD PTR SS:[ESP+100]
``` 
