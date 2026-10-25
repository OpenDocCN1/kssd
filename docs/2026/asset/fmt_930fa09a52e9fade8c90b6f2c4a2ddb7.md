
# 分析笔记

`0059CB0C`   `FF75E8`                 `push`    `dword ptr [ebp-$18]`

`0059CB0F`   `8D45F0`                 `lea`     `eax, [ebp-$10]`

`0059CB12`   `BA03000000`             `mov`     `edx, $00000003`

|

`0059CB17`   `E82C81E6FF`             `call`    `00404C48`

`0059CB1C`   `8B55F0`                 `mov`     `edx, [ebp-$10]`

`0059CB1F`   `8B45FC`                 `mov`     `eax, [ebp-$04]`

* Reference to: `System.@LStrCmp`;                                          比较

|

`0059CB22`   `E8A581E6FF`             `call`    `00404CCC`

`0059CB27`   `7504`                   `jnz`     `0059CB2D`

`0059CB29`   `B301`                   `mov`     `bl, $01`

`0059CB2B`   `EB02`                   `jmp`     `0059CB2F`

`0059CB2D`   `33DB`                   `xor`     `ebx, ebx`

`0059CB2F`   `33C0`                   `xor`     `eax, eax`

`0059CB31`   `5A`                     `pop`     `edx`

`0059CB32`   `59`                     `pop`     `ecx`

`0059CB33`   `59`                     `pop`     `ecx`

`0059CB34`   `648910`                 `mov`     `fs:[eax], edx`

**** FINALLY

`dede`就是好，代码的流程大概清楚了，粗略的跟踪一下知道验证过程是：

读取输入的注册信息(做某些变换?)然后通过`blowfish`(encrypt还是`decrypt`？)算法处理后

与特定的字符串：

`"'作者的名字，软件信息，网站等等'+'网卡mac'+'硬盘序列号'"`进行比较，如果一样则验证通过，注册成功。

这里用到了`blowfish`算法，要做的就是找出算法的`key`还有是`encrypt`还是`decrypt`。

粗略跟踪知道输入信息是在：

```
0059CAF1  |. FF53 58       CALL DWORD PTR DS:[EBX+58]              ;
```
进行处理的，步过此`call`后就得到了后面和特定字符串比较的数据，进入`call`：
```
0053CCE0   . 53            PUSH EBX
0053CCE1   . 56            PUSH ESI
0053CCE2   . 57            PUSH EDI
0053CCE3   . 8BD9          MOV EBX,ECX
0053CCE5   . 8BFA          MOV EDI,EDX
0053CCE7   . 8BF0          MOV ESI,EAX
0053CCE9   . 8BD3          MOV EDX,EBX
0053CCEB   . 8BC7          MOV EAX,EDI
0053CCED   . E8 16F5FFFF   CALL EsoftPro.0053C208                   对输入数据进行base64解码
0053CCF2   . 8B03          MOV EAX,DWORD PTR DS:[EBX]
0053CCF4   . E8 8F7EECFF   CALL EsoftPro.00404B88
0053CCF9   . 50            PUSH EAX
0053CCFA   . 8BC3          MOV EAX,EBX
0053CCFC   . E8 D780ECFF   CALL EsoftPro.00404DD8
0053CD01   . 50            PUSH EAX
0053CD02   . 8BC3          MOV EAX,EBX
0053CD04   . E8 CF80ECFF   CALL EsoftPro.00404DD8
0053CD09   . 8BD0          MOV EDX,EAX
0053CD0B   . 8BC6          MOV EAX,ESI
0053CD0D   . 59            POP ECX
0053CD0E   . 8B18          MOV EBX,DWORD PTR DS:[EAX]
0053CD10   . FF93 80000000 CALL DWORD PTR DS:[EBX+80]             对解码后的数据进一步的变换
0053CD16   . 5F            POP EDI
0053CD17   . 5E            POP ESI
0053CD18   . 5B            POP EBX
0053CD19   . C3            RETN
```

## base64解码

```
0053C0D4  /$ 56            PUSH ESI
0053C0D5  |. 57            PUSH EDI
0053C0D6  |. 55            PUSH EBP
0053C0D7  |. 83C4 F0       ADD ESP,-10
0053C0DA  |. 890C24        MOV DWORD PTR SS:[ESP],ECX
0053C0DD  |. 8BF8          MOV EDI,EAX
0053C0DF  |. 895424 08     MOV DWORD PTR SS:[ESP+8],EDX
0053C0E3  |. 33F6          XOR ESI,ESI
0053C0E5  |. 33ED          XOR EBP,EBP
0053C0E7  |. 33C0          XOR EAX,EAX
0053C0E9  |. 8B1424        MOV EDX,DWORD PTR SS:[ESP]
0053C0EC  |. 85D2          TEST EDX,EDX
0053C0EE  |. 79 03         JNS SHORT EsoftPro.0053C0F3
0053C0F0  |. 83C2 03       ADD EDX,3
0053C0F3  |> C1FA 02       SAR EDX,2
0053C0F6  |. 85D2          TEST EDX,EDX
0053C0F8  |. 0F8E 02010000 JLE EsoftPro.0053C200
0053C0FE  |. 895424 0C     MOV DWORD PTR SS:[ESP+C],EDX
0053C102  |> BA 04000000   /MOV EDX,4                              ;
0053C107  |. 8D4424 04     |LEA EAX,DWORD PTR SS:[ESP+4]
0053C10B  |> 33C9          |/XOR ECX,ECX                           ;
0053C10D  |. 8A0C37        ||MOV CL,BYTE PTR DS:[EDI+ESI]
0053C110  |. 83F9 3D       ||CMP ECX,3D                            ;  Switch (cases 2B..7A)
0053C113  |. 7F 14         ||JG SHORT EsoftPro.0053C129
0053C115  |. 74 52         ||JE SHORT EsoftPro.0053C169
0053C117  |. 83E9 2B       ||SUB ECX,2B
0053C11A  |. 74 43         ||JE SHORT EsoftPro.0053C15F
0053C11C  |. 83E9 04       ||SUB ECX,4
0053C11F  |. 74 43         ||JE SHORT EsoftPro.0053C164
0053C121  |. 49            ||DEC ECX
0053C122  |. 83E9 0A       ||SUB ECX,0A
0053C125  |. 72 2B         ||JB SHORT EsoftPro.0053C152
0053C127  |. EB 43         ||JMP SHORT EsoftPro.0053C16C
0053C129  |> 83C1 BF       ||ADD ECX,-41
0053C12C  |. 83E9 1A       ||SUB ECX,1A
0053C12F  |. 72 0A         ||JB SHORT EsoftPro.0053C13B
0053C131  |. 83C1 FA       ||ADD ECX,-6
0053C134  |. 83E9 1A       ||SUB ECX,1A
0053C137  |. 72 0C         ||JB SHORT EsoftPro.0053C145
0053C139  |. EB 31         ||JMP SHORT EsoftPro.0053C16C
0053C13B  |> 8A0C37        ||MOV CL,BYTE PTR DS:[EDI+ESI]          ;  Cases 41 ('A'),42 ('B'),43 ('C'),44 ('D'),45 ('E'),46 ('F'),47 ('G'),48 ('H'),49 ('I'),4A ('J'),4B ('K'),4C ('L'),4D ('M'),4E ('N'),4F ('O'),50 ('P'),51 ('Q'),52 ('R'),53 ('S'),54 ('T')... of switch 0053C110
0053C13E  |. 80E9 41       ||SUB CL,41
0053C141  |. 8808          ||MOV BYTE PTR DS:[EAX],CL
0053C143  |. EB 27         ||JMP SHORT EsoftPro.0053C16C
0053C145  |> 8A0C37        ||MOV CL,BYTE PTR DS:[EDI+ESI]          ;  Cases 61 ('a'),62 ('b'),63 ('c'),64 ('d'),65 ('e'),66 ('f'),67 ('g'),68 ('h'),69 ('i'),6A ('j'),6B ('k'),6C ('l'),6D ('m'),6E ('n'),6F ('o'),70 ('p'),71 ('q'),72 ('r'),73 ('s'),74 ('t')... of switch 0053C110
0053C148  |. 80E9 61       ||SUB CL,61
0053C14B  |. 80C1 1A       ||ADD CL,1A
0053C14E  |. 8808          ||MOV BYTE PTR DS:[EAX],CL
0053C150  |. EB 1A         ||JMP SHORT EsoftPro.0053C16C
0053C152  |> 8A0C37        ||MOV CL,BYTE PTR DS:[EDI+ESI]          ;  Cases 30 ('0'),31 ('1'),32 ('2'),33 ('3'),34 ('4'),35 ('5'),36 ('6'),37 ('7'),38 ('8'),39 ('9') of switch 0053C110
0053C155  |. 80E9 30       ||SUB CL,30
0053C158  |. 80C1 34       ||ADD CL,34
0053C15B  |. 8808          ||MOV BYTE PTR DS:[EAX],CL
0053C15D  |. EB 0D         ||JMP SHORT EsoftPro.0053C16C
0053C15F  |> C600 3E       ||MOV BYTE PTR DS:[EAX],3E              ;  Case 2B ('+') of switch 0053C110
0053C162  |. EB 08         ||JMP SHORT EsoftPro.0053C16C
0053C164  |> C600 3F       ||MOV BYTE PTR DS:[EAX],3F              ;  Case 2F ('/') of switch 0053C110
0053C167  |. EB 03         ||JMP SHORT EsoftPro.0053C16C
0053C169  |> C600 FF       ||MOV BYTE PTR DS:[EAX],0FF             ;  Case 3D ('=') of switch 0053C110
0053C16C  |> 46            ||INC ESI                               ;  Default case of switch 0053C110
0053C16D  |. 40            ||INC EAX
0053C16E  |. 4A            ||DEC EDX
0053C16F  |.^75 9A         |\JNZ SHORT EsoftPro.0053C10B
0053C171  |. 8A4424 04     |MOV AL,BYTE PTR SS:[ESP+4]
0053C175  |. C1E0 02       |SHL EAX,2
0053C178  |. 33D2          |XOR EDX,EDX
0053C17A  |. 8A5424 05     |MOV DL,BYTE PTR SS:[ESP+5]
0053C17E  |. C1EA 04       |SHR EDX,4
0053C181  |. 0AC2          |OR AL,DL
0053C183  |. 8B5424 08     |MOV EDX,DWORD PTR SS:[ESP+8]
0053C187  |. 88042A        |MOV BYTE PTR DS:[EDX+EBP],AL
0053C18A  |. 8D45 01       |LEA EAX,DWORD PTR SS:[EBP+1]
0053C18D  |. 8A5424 06     |MOV DL,BYTE PTR SS:[ESP+6]
0053C191  |. 80FA FF       |CMP DL,0FF
0053C194  |. 74 27         |JE SHORT EsoftPro.0053C1BD
0053C196  |. 807C24 07 FF  |CMP BYTE PTR SS:[ESP+7],0FF
0053C19B  |. 75 20         |JNZ SHORT EsoftPro.0053C1BD
0053C19D  |. 8A4424 05     |MOV AL,BYTE PTR SS:[ESP+5]
0053C1A1  |. C1E0 04       |SHL EAX,4
0053C1A4  |. 81E2 FF000000 |AND EDX,0FF
0053C1AA  |. C1EA 02       |SHR EDX,2
0053C1AD  |. 0AC2          |OR AL,DL
0053C1AF  |. 8B5424 08     |MOV EDX,DWORD PTR SS:[ESP+8]
0053C1B3  |. 88442A 01     |MOV BYTE PTR DS:[EDX+EBP+1],AL
0053C1B7  |. 8D45 02       |LEA EAX,DWORD PTR SS:[EBP+2]
0053C1BA  |. 45            |INC EBP
0053C1BB  |. EB 38         |JMP SHORT EsoftPro.0053C1F5
0053C1BD  |> 80FA FF       |CMP DL,0FF
0053C1C0  |. 74 33         |JE SHORT EsoftPro.0053C1F5
0053C1C2  |. 8A4424 05     |MOV AL,BYTE PTR SS:[ESP+5]
0053C1C6  |. C1E0 04       |SHL EAX,4
0053C1C9  |. 81E2 FF000000 |AND EDX,0FF
0053C1CF  |. C1EA 02       |SHR EDX,2
0053C1D2  |. 0AC2          |OR AL,DL
0053C1D4  |. 8B5424 08     |MOV EDX,DWORD PTR SS:[ESP+8]
```
