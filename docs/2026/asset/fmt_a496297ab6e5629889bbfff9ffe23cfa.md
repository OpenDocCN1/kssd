
# Hying旧版壳的简单脱壳

- 【目标】：天堂xxxx（因为是外挂，所以不写全名）
- 【工具】：`Olydbg1.1(diy版)`、`LORDPE`、`ImportREC1.6F`
- 【任务】：简单的脱一下目标的壳
- 【操作平台】：`WinXP sp2`
- 【作者】：`loveboom[DFCG][FCG][US]`
- 【相关链接】：自己去上网搜搜

## 简要说明

这个一个很旧的Hying的壳了，现在脱的话就非常简单了，不过看过新版的朋友就知道，新版BT很多，不过也不是那么神秘的:-)。介于关于他的壳的信息比较少，所以写一个大概的。

## 详细过程

用`peid`直接看的话显示是的`yc`的壳，不过我之前接触过几个`hying`的壳，因此我用扩展方式可以看到是`PELOCK ->Hying \*`，知道是什么壳就好办了，之前在大牛hexer的指导（不明之处看的他以前那篇文章）下跟了一个新版的壳，所以这个旧版相对来说很简单的。好了入主题先。

设置忽略全部异常，去除调试标志。

载入在`ep`处。

```asm
005D4DA8 >  60              PUSHAD                                   ; ep入口是伪yoda crypt的
005D4DA9    E8 00000000     CALL 005D4DAE
005D4DAE    5D              POP EBP
```

载入后直接在`code`段下`f2`断点。

```text
Memory map, item 14
Address=00401000
Size=001AC000 (1753088.)
Owner=L2Walker 00400000
Section=
Contains=code
Type=Imag 01001002
Access=R
Initial access=RWE
```

下断后，`f9`运行。这样中断在这里：

```asm
00373670    AC              LODS BYTE PTR DS:[ESI]                   ; 中断在这里
00373671    D2C8            ROR AL,CL
00373673    32C1            XOR AL,CL
00373675    04 66           ADD AL,66
00373677    32C5            XOR AL,CH
00373679    02C6            ADD AL,DH
0037367B    2AC2            SUB AL,DL
0037367D    02C1            ADD AL,CL
0037367F    2AC5            SUB AL,CH
00373681    32C2            XOR AL,DL
00373683    04 23           ADD AL,23
00373685    32C6            XOR AL,DH
00373687    F6D0            NOT AL
00373689    D2C8            ROR AL,CL
0037368B    D3CA            ROR EDX,CL
0037368D    AA              STOS BYTE PTR ES:[EDI]
0037368E    49              DEC ECX
0037368F    0BC9            OR ECX,ECX
00373691  ^ 75 DD           JNZ SHORT 00373670
00373693    53              PUSH EBX
00373694    6A 04           PUSH 4
00373696    68 00100000     PUSH 1000
0037369B    FF342B          PUSH DWORD PTR DS:[EBX+EBP]
0037369E    6A 00           PUSH 0
003736A0    8D85 001D4000   LEA EAX,DWORD PTR SS:[EBP+401D00]
003736A6    50              PUSH EAX
003736A7    8B85 33374000   MOV EAX,DWORD PTR SS:[EBP+403733]
003736AD    E9 010E0000     JMP 003744B3
003736B2  - E9 897C24EC     JMP EC5BB340
003736B7    897424 E8       MOV DWORD PTR SS:[ESP-18],ESI
003736BB    FF85 47374000   INC DWORD PTR SS:[EBP+403747]
003736C1    8B9D 6B374000   MOV EBX,DWORD PTR SS:[EBP+40376B]
003736C7    83FB 01         CMP EBX,1
003736CA    75 0E           JNZ SHORT 003736DA
003736CC    61              POPAD
003736CD    8B4424 CC       MOV EAX,DWORD PTR SS:[ESP-34]
003736D1    8D78 02         LEA EDI,DWORD PTR DS:[EAX+2]
003736D4    55              PUSH EBP
003736D5    8BEC            MOV EBP,ESP
003736D7    50              PUSH EAX
003736D8    EB 20           JMP SHORT 003736FA
003736DA    83FB 02         CMP EBX,2
003736DD    75 15           JNZ SHORT 003736F4
003736DF    61              POPAD
003736E0    8B4424 C8       MOV EAX,DWORD PTR SS:[ESP-38]
003736E4    FFB0 6F374000   PUSH DWORD PTR DS:[EAX+40376F]
003736EA    8B4424 D0       MOV EAX,DWORD PTR SS:[ESP-30]
003736EE    50              PUSH EAX
003736EF    8D78 02         LEA EDI,DWORD PTR DS:[EAX+2]
003736F2    EB 06           JMP SHORT 003736FA
003736F4    61              POPAD
003736F5    8B4424 CC       MOV EAX,DWORD PTR SS:[ESP-34]
003736F9    50              PUSH EAX                                 ; 这里跳去oep
003736FA    C3              RETN
```

断下后经过几个跟踪发`003736f9`处就是 `push OEP`。继续往下找，找到这里：

```asm
003737D3    87E6            XCHG ESI,ESP
003737D5    B9 7F0B0000     MOV ECX,0B7F
003737DA    58              POP EAX                                  ; 这里循环解压下面的代码
003737DB    F6D0            NOT AL
003737DD    50              PUSH EAX
003737DE    44              INC ESP
003737DF  ^ E2 F9           LOOPD SHORT 003737DA
003737E1    87E6            XCHG ESI,ESP                             ; 所以我们直接在这里下f4
003737E3    6A 04           PUSH 4
003737E5    68 00100000     PUSH 1000
003737EA    68 00200000     PUSH 2000
003737EF    6A 00           PUSH 0
003737F1    FF95 33374000   CALL DWORD PTR SS:[EBP+403733]           ; VirtualAlloc
003737F7    8985 3C3D4000   MOV DWORD PTR SS:[EBP+403D3C],EAX
003737FD    C785 403D4000 0>MOV DWORD PTR SS:[EBP+403D40],0
00373807    74 7A           JE SHORT 00373883
00373809    9C              PUSHFD
0037380A    C8 BFFFF4       ENTER 0FFBF,0F4
0037380E    3F              AAS
0037380F    F0:7A 42        LOCK JPE SHORT 00373854                  ; LOCK prefix is not allowed
00373812    FFFF            ???                                      ; Unknown command
```

直接`F4`到`3737E1`处后看看代码已经解压了，现在好看很多。

下面的部分跟过`hying`的壳的朋友就知道壳是通过取出 API 函数先判断代码是否要抽 API 代码，如果要的话就抽部分代码到壳申请的指定地址里实现 `push api`、`ret` 的方式，如果没要抽代码的就直接 `push ret` 的方式。

所以我现在要找到跳去 API 的代码，这个很容易，看看代码里有很多这样的代码：

```asm
0037383B    8B85 2B374000   MOV EAX,DWORD PTR SS:[EBP+40372B]
00373841    E9 6D0C0000     JMP 003744B3                             ; 这里一个
00373846    81340B C0751E56 XOR DWORD PTR DS:[EBX+ECX],561E75C0
0037384D    8D85 661E4000   LEA EAX,DWORD PTR SS:[EBP+401E66]
00373853    50              PUSH EAX
00373854    8B85 2F374000   MOV EAX,DWORD PTR SS:[EBP+40372F]
0037385A    E9 540C0000     JMP 003744B3                             ; 这里又一个
```

所以可以判断`003744b3`处是处理 API 的。进去看看：

```asm
003744B3    50              PUSH EAX
003744B4    8B85 383D4000   MOV EAX,DWORD PTR SS:[EBP+403D38]
003744BA    50              PUSH EAX
003744BB    E8 08000000     CALL 003744C8
003744C0    8B85 383D4000   MOV EAX,DWORD PTR SS:[EBP+403D38]
003744C6    FFE0            JMP EAX                                  ; 跳去push api ret的地址
003744C8    60              PUSHAD                                   ; 这里开始处理API
003744C9    8B7C24 24       MOV EDI,DWORD PTR SS:[ESP+24]
003744CD    8B7424 28       MOV ESI,DWORD PTR SS:[ESP+28]
003744D1    66:8B06         MOV AX,WORD PTR DS:[ESI]
003744D4    3C 50           CMP AL,50
003744D6    72 0A           JB SHORT 003744E2                        ; ＊＊＊＊＊＊
003744D8    3C 57           CMP AL,57                                ; 开始判断要不要特别"照顾"
003744DA    77 06           JA SHORT 003744E2
003744DC    8807            MOV BYTE PTR DS:[EDI],AL
003744DE    46              INC ESI
003744DF    47              INC EDI
……
003745F2   /E9 AD120000     JMP 003758A4
003745F7   |66:3D CD03      CMP AX,3CD
003745FB   |75 05           JNZ SHORT 00374602
003745FD   |E9 A2120000     JMP 003758A4
00374602   |C607 68         MOV BYTE PTR DS:[EDI],68                 ; 这里变成push api ret的方式
00374605   |8977 01         MOV DWORD PTR DS:[EDI+1],ESI
00374608   |C647 05 C3      MOV BYTE PTR DS:[EDI+5],0C3
0037460C   |83C7 06         ADD EDI,6
0037460F   |897C24 FC       MOV DWORD PTR SS:[ESP-4],EDI
00374613   |61              POPAD
00374614   |8B4424 DC       MOV EAX,DWORD PTR SS:[ESP-24]
00374618   |C2 0800         RETN 8
```

分析完代码后我们就要动"手术"了，在`003744d6`处改成：

```asm
003744D6   /E9 27010000     JMP 00374602            ;这里跳过就直接成了
push api
retn
```

的方式防止壳抽代码。

`F4`到了`003737E1`后，几个`F8`就到了处理 IAT 处了：

```asm
003738D8    0395 7F374000   ADD EDX,DWORD PTR SS:[EBP+40377F]
003738DE    8B3A            MOV EDI,DWORD PTR DS:[EDX]
003738E0    0BFF            OR EDI,EDI                               ; 判断有没有处理完iat
003738E2    75 05           JNZ SHORT 003738E9
003738E4    E9 53050000     JMP 00373E3C                             ; 处理完iat就跳
003738E9    03BD 37374000   ADD EDI,DWORD PTR SS:[EBP+403737]
003738EF    83C2 05         ADD EDX,5
003738F2    8BF2            MOV ESI,EDX
003738F4    56              PUSH ESI
003738F5    8D85 0E1F4000   LEA EAX,DWORD PTR SS:[EBP+401F0E]
003738FB    50              PUSH EAX
003738FC    8B85 2B374000   MOV EAX,DWORD PTR SS:[EBP+40372B]        ; GetModuleHandleA
00373902    E9 AC0B0000     JMP 003744B3
00373907    90              NOP
00373908    90              NOP
00373909    0BC0            OR EAX,EAX
0037390B    75 1E           JNZ SHORT 0037392B                       ; 判断有没有载入DLL如果没有就先LoadLibrary载入DLL
0037390D    56              PUSH ESI
0037390E    8D85 271F4000   LEA EAX,DWORD PTR SS:[EBP+401F27]
```
