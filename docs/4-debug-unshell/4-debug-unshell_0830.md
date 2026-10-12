
# 阿达连连看 2005 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-643.htm`](https://www.pediy.com/kssd/pediy07/pediy7-643.htm)

最近工作忙, 好久没脱壳了, 5.1 节总算有点空了, 来论坛学习学习.

脱了一下阿达连连看 2005 5.05, 祝大家节日快乐.

【脱文作者】 simonzh2000

【使用工具】 Ollydbg1.10, LordPE, ImportRec

【脱壳平台】 Win2000 Pro SP4 English

【作者声明】 本笔记只用于学习交流, 初学 Crack，只是感兴趣技术，没有其他目的, 如有不妥之处, 希望作者谅解.

感谢 window, hexer, loveboom, fly, forgot, cyclotron 等大侠.

对比 window 的脱文, 发现 `"CMP EAX,CD000094"` 已经找不到了, 这段代码是使用前才解密出来的, 那我们只好一步一步来了.

## 一. 初始工作

```
004A013D  > /E9 00000000     JMP adalinks.004A0142

004A0142   \60               PUSHAD
004A0143    E8 14000000      CALL adalinks.004A015C
004A015C    58               POP EAX                  ; adalinks.004A0148
004A015D    61               POPAD
004A015E  ^ E9 6EFFFEFF     JMP adalinks.004900D1

004900D1    60               PUSHAD
004900D2    E8 00000000      CALL adalinks.004900D7
004900D7    5D               POP EBP
004900D8    81ED D7000000    SUB EBP,0D7
004900DE    8DB5 EE000000    LEA ESI,DWORD PTR SS:[EBP+EE]
004900E4    55               PUSH EBP
004900E5    56               PUSH ESI
004900E6    81C5 FD010000    ADD EBP,1FD
004900EC    55               PUSH EBP
004900ED    C3               RETN

004901FD    81D1 DE7CDF28    ADC ECX,28DF7CDE
00490203    8DA8 B4D76E52    LEA EBP,DWORD PTR DS:[EAX+526ED7B4]

............................// F7 走, SMC 壳代码

00490D5B    8DA8 83CBFC48    LEA EBP,DWORD PTR DS:[EAX+48FCCB83]
00490D61    83C3 04          ADD EBX,4
00490D64    FF0424           INC DWORD PTR SS:[ESP]
00490D67    C3               RETN

004900EF    5D               POP EBP                  ; adalinks.00490000
004900F0    8B45 00          MOV EAX,DWORD PTR SS:[EBP]
004900F3    0BC0             OR EAX,EAX
004900F5    74 04            JE SHORT adalinks.004900FB
004900F7    55               PUSH EBP
004900F8    FF65 0C          JMP DWORD PTR SS:[EBP+C]
004900FB    FF45 00          INC DWORD PTR SS:[EBP]
004900FE    8B4424 24        MOV EAX,DWORD PTR SS:[ESP+24]
00490102    8945 04          MOV DWORD PTR SS:[EBP+4],EAX
00490105    8DB5 84000000    LEA ESI,DWORD PTR SS:[EBP+84]
0049010B    56               PUSH ESI
0049010C    FF55 78          CALL DWORD PTR SS:[EBP+78]           ; KERNEL32.GetModuleHandleA
0049010F    8D75 1C          LEA ESI,DWORD PTR SS:[EBP+1C]
00490112    56               PUSH ESI
00490113    50               PUSH EAX
00490114    FF55 74          CALL DWORD PTR SS:[EBP+74]           ; KERNEL32.GetProcAddress
00490117    8945 2C          MOV DWORD PTR SS:[EBP+2C],EAX
0049011A    6A 04            PUSH 4
0049011C    68 00100000      PUSH 1000
00490121    FF75 10          PUSH DWORD PTR SS:[EBP+10]
00490124    6A 00            PUSH 0
00490126    FF55 2C          CALL DWORD PTR SS:[EBP+2C]           ; KERNEL32.VirtualAlloc
00490129    50               PUSH EAX
0049012A    8945 0C          MOV DWORD PTR SS:[EBP+C],EAX
0049012D    8B5D 08         MOV EBX,DWORD PTR SS:[EBP+8]
00490130    03DD             ADD EBX,EBP
00490132    50               PUSH EAX
00490133    53               PUSH EBX
00490134    E8 18000000      CALL adalinks.00490151               ; F8 过, 解压壳代码到 340000
00490139    5A               POP EDX
0049013A    52               PUSH EDX
0049013B    55               PUSH EBP
0049013C    8D85 DE000000    LEA EAX,DWORD PTR SS:[EBP+DE]
00490142    C600 EB          MOV BYTE PTR DS:[EAX],0EB
00490145    C640 01 10       MOV BYTE PTR DS:[EAX+1],10
00490149    8B45 30          MOV EAX,DWORD PTR SS:[EBP+30]
0049014C    8945 74          MOV DWORD PTR SS:[EBP+74],EAX
0049014F    FFE2             JMP EDX                    ; 340000,   以后可以直接 F4 到这里

00340000    B9 04000000      MOV ECX,4                  ; 花指令很多
00340005    E8 1F000000      CALL 00340029

... 第一种 单步异常 反跟踪, 如下

0034002C    E8 24000000      CALL 00340055              ; 单步陷阱
00340031                                          ; SEH=340031
00340055    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00340070    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
0034005F    9C               PUSHFD
00340065    810C24 00010000  OR DWORD PTR SS:[ESP],100         ; Trap Flag = 1
0034006C    9D               POPFD
0034006D    90               NOP                  ; 异常

00340031    8B4424 04        MOV EAX,DWORD PTR SS:[ESP+4]      ; pEXCEPTION_RECORD
00340035    8B00             MOV EAX,DWORD PTR DS:[EAX]        ; ExceptionCode
00340037    3D 04000080      CMP EAX,80000004          
0034003C    75 08            JNZ SHORT 00340046            ; 不能跳          
0034003E    8B6424 08        MOV ESP,DWORD PTR SS:[ESP+8]      ; 恢复 Stack
00340042    EB 04            JMP SHORT 00340048
00340046    0C E9            OR AL,0E9
00340048    64:8F05 0000000>POP DWORD PTR FS:[0]          ; 恢复 SEH
0034004F   ^ 74 F3           JE SHORT 00340044
00340051   ^ 75 F1           JNZ SHORT 00340044
00340044    58               POP EAX
00340045    EB 0C            JMP SHORT 00340053

... 下面还有一种 INT3 异常,   EIP+2 就可

这两种反复出现,   只是用来反跟踪, 烦人. OD 忽略 这两种异常, F9, 停下, 看 LOG
```

**Log data**

| Address | Message |
| :--- | :--- |
| `0034012C` | `INT3 command at 0034012C` |
| `003403D4` | `INT3 command at 003403D4` |
| `003404F2` | `INT3 command at 003404F2` |
| `003406F6` | `INT3 command at 003406F6` |
| `00340806` | `INT3 command at 00340806` |
| `0034085E` | `INT3 command at 0034085E` |
| `0034091C` | `INT3 command at 0034091C` |
| `00340974` | `INT3 command at 00340974` |
| `003409DF` | `INT3 command at 003409DF` |
| `00340AB5` | `INT3 command at 00340AB5` |
| `00340BF4` | `INT3 command at 00340BF4` |
| `00340CC5` | `INT3 command at 00340CC5` |
| `00340D57` | `INT3 command at 00340D57` |
| `00341120` | `INT3 command at 00341120` |
| `00341311` | `INT3 command at 00341311` |
| `003413FA` | `INT3 command at 003413FA` |
| `00341538` | `INT3 command at 00341538` |
| `003415A3` | `INT3 command at 003415A3` |
| `0034171C` | `INT3 command at 0034171C` |
| `0034183A` | `INT3 command at 0034183A` |
| `0034193F` | `INT3 command at 0034193F` |
| `00341A10` | `INT3 command at 00341A10` |
| `00341A55` | `INT3 command at 00341A55` |
| `00341A9A` | `INT3 command at 00341A9A` |
| `00341B0B` | `INT3 command at 00341B0B` |
| `00341B50` | `INT3 command at 00341B50` |
| `00341C80` | `INT3 command at 00341C80` |
| `00341EA5` | `INT3 command at 00341EA5` |
| `00342114` | `INT3 command at 00342114` |
| `003425FB` | `INT3 command at 003425FB` |
| `003426B9` | `INT3 command at 003426B9` |
| `0034285D` | `INT3 command at 0034285D` |
| `003428EF` | `INT3 command at 003428EF` |
| `00342A63` | `INT3 command at 00342A63` |
| `00342ABB` | `INT3 command at 00342ABB` |
| `00342BAD` | `INT3 command at 00342BAD` |
| `00342E6F` | `INT3 command at 00342E6F` |
| `00342FF3` | `INT3 command at 00342FF3` ; 最后一个在这里 |
| `00343EE9` | `Illegal instruction` |

;=======================================================================================================================================================================

## 二. SEH = 3434BA 的处理

重新来过

```
00342FEF    66:BF 4D4A       MOV DI,4A4D   ; 下普通断点, F9, F2, 取消忽略异常
00342FF3    CC               INT3          
00342FBE    8B4C24 0C        MOV ECX,DWORD PTR SS:[ESP+C]    ; SEH
00342FC2    FF81 B8000000    INC DWORD PTR DS:[ECX+B8]       ; EIP+1
00342FC8    FF81 B8000000    INC DWORD PTR DS:[ECX+B8]       ; EIP+1
00342FCE    33C0             XOR EAX,EAX
00342FD0    C3               RETN

00342FF5    90               NOP
00342FF6    64:8F05 0000000>POP DWORD PTR FS:[0]
00342FFD    58               POP EAX

...

; 复制 4 个数到 35096E,  后 3 个是 GetMoudleHandleA, LoadLibraryA, VirtualAlloc

0034302E    B9 03000000      MOV ECX,3
00343037    8D75 74          LEA ESI,DWORD PTR SS:[EBP+74]     ; 490074
```



00343051 8DBA 26F44000 LEA EDI,DWORD PTR DS:[EDX+40F426] ; 35096E
0034306B 8B06 MOV EAX,DWORD PTR DS:[ESI]
00343074 8907 MOV DWORD PTR DS:[EDI],EAX
00343081 83C6 04 ADD ESI,4
00343089 83C7 04 ADD EDI,4
00343090 ^E2 D9 LOOPD SHORT 0034306B

; 后面有一些反跟踪手段, 如 压一些数到 Stack, 再跳到 Stack 执行
...
00343306 6A 00 PUSH 0
00343308 FF95 2AF44000 CALL DWORD PTR SS:[EBP+40F42A] ; KERNEL32.GetModuleHandleA
00343329 8985 36F44000 MOV DWORD PTR SS:[EBP+40F436],EAX ; adalinks.00400000-> [35097E]
00343346 6A 04 PUSH 4
00343348 68 00100000 PUSH 1000
0034334D 68 00100000 PUSH 1000
00343352 6A 00 PUSH 0
00343354 FF95 32F44000 CALL DWORD PTR SS:[EBP+40F432] ; KERNEL32.VirtualAlloc
0034335A 8985 1AFD4000 MOV DWORD PTR SS:[EBP+40FD1A],EAX ; 360000-> [351262]
; 解密 2a2h 个字节代码
00343D73 8DB5 C31F4000 LEA ESI,DWORD PTR SS:[EBP+401FC3] ; 34350B
00343EC8 8BFE MOV EDI,ESI
00343ECA B9 A2020000 MOV ECX,2A2
00343ECF AC LODS BYTE PTR DS:[ESI]
00343ED0 32C1 XOR AL,CL
00343ED2 C0C0 04 ROL AL,4
00343ED5 AA STOS BYTE PTR ES:[EDI]
00343ED6 ^ E2 F7 LOOPD SHORT 00343ECF
00343ED8 5E POP ESI
00343ED9 33C0 XOR EAX,EAX
00343EDB 64:FF35 0000000>PUSH DWORD PTR FS:[0]
00343EE2 64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00343EE9 0F0B UD2 ; Illegal instruction, SEH = 3434BA
00343EEB 33 ; 会变成 CC(INT3)
00343EEC 90 NOP
; 关键的地方出现了, 大致是这样的,
; 利用非法指令进入异常处理
; regEIP + 2, not [regEIP+2] 后正好是 CC, 退出非法指令异常处理
; Int3 异常, 再次进入异常处理
; regEIP + 1, 设置 Dr0, Dr1, Dr2, Dr3, 退出 Int3 异常
; 碰到 Dr0, Dr1, Dr2, Dr3 指定的地方, 单步中断
; 单步中断中分别解密代码
; 解密出除零异常处理代码( 这好象是最新改的，针对Ｗindow 的脱文）
; 除零异常
; 再次设置 Dr0, Dr1, Dr2, Dr3
; 结束这一段程序
00343EED 56 PUSH ESI
00343EEE 8DB5 6C224000 LEA ESI,DWORD PTR SS:[EBP+40226C] ; 3437B4
00344043 B9 BD040000 MOV ECX,4BD
00344048 AC LODS BYTE PTR DS:[ESI]
00344049 32C1 XOR AL,CL
0034404B 04 4D ADD AL,4D
0034404D C0C0 03 ROL AL,3
00344050 AA STOS BYTE PTR ES:[EDI]
00344051 ^ E2 F5 LOOPD SHORT 00344048 ; 解密 4BD 字节代码
00344053 5E POP ESI
00344054 8D8D C91A4000 LEA ECX,DWORD PTR SS:[EBP+401AC9]
0034405A 2BCE SUB ECX,ESI ; 3011
003441B0 AC LODS BYTE PTR DS:[ESI] ; 340000
003441B1 03D8 ADD EBX,EAX
003441B3 ^ E2 FB LOOPD SHORT 003441B0
003441B5 8BC3 MOV EAX,EBX ; 340000- 343011 的字节累加和 (11EE9B)
003441B7 F8 CLC ; Dr0 中断, 我们做 3 件事( 见 3434BA 分析)
; 1. [3434BF]= 1,
; 2. not eax (结果 FFEE1164)
; 3. EIP + 1
003441B8 90 NOP
003441B9 8DB5 352E4000 LEA ESI,DWORD PTR SS:[EBP+402E35] ; 34437D
003441EC B9 EDC50000 MOV ECX,0C5ED
0012FF8C F7E1 MUL ECX ; 在 stack 解密 C5ED 字节代码
0012FF8E D3C8 ROR EAX,CL
0012FF90 3006 XOR BYTE PTR DS:[ESI],AL
0012FF92 46 INC ESI
0012FF94 FFC0 INC EAX
0012FF96 D40A AAM
0012FF98 ^ E2 F2 LOOPD SHORT 0012FF8C
00344285 8B1C24 MOV EBX,DWORD PTR SS:[ESP] ; 11EE9B (上次计算的字节和)
0034437D B9 A9040000 MOV ECX,4A9
00344399 8DB5 C91A4000 LEA ESI,DWORD PTR SS:[EBP+401AC9] ; 343011
0034439F 33C0 XOR EAX,EAX
003443A1 3206 XOR AL,BYTE PTR DS:[ESI]
003443A3 C1C8 08 ROR EAX,8
003443A6 46 INC ESI
003443A7 ^ E2 F8 LOOPD SHORT 003443A1 ; 再算一个校验和 EAX = D60B4944
003443A9 FC CLD ; Dr1 中断, 我们做 3 件事( 见 3434BA 分析)
; 1. [3434BF]= 2
; 2. rol eax, 13 (结果 4A26B05A)
; 3. EIP + 1
003443AA 90 NOP
003443AB B9 5AC40000 MOV ECX,0C45A
003444FF 8DB5 C82F4000 LEA ESI,DWORD PTR SS:[EBP+402FC8] ; 344510
00344505 8D4481 43 LEA EAX,DWORD PTR DS:[ECX+EAX*4+43]
00344509 3006 XOR BYTE PTR DS:[ESI],AL
0034450B D40A AAM
0034450D 46 INC ESI
0034450E ^ E2 F5 LOOPD SHORT 00344505 ; 再解密 C45A 代码
00344510 B9 E0070000 MOV ECX,7E0
00344515 C1E9 02 SHR ECX,2 ; ECX = 1F8
00344545 8DB5 2A284000 LEA ESI,DWORD PTR SS:[EBP+40282A] ; 343D72
0034454B 33DB XOR EBX,EBX
0034454D AD LODS DWORD PTR DS:[ESI]
0034454E 33D8 XOR EBX,EAX
00344550 ^ E2 FB LOOPD SHORT 0034454D ; 再算一个校验和 EAX = FBE2D833
EBX = DAB2196B
00344552 F9 STC ; Dr2 中断, 我们做 4 件事( 见 3434BA 分析)
; 1. [3434BF]= 3
; 2. EAX = 009526C4
; 3. EBX = DAB20D59
; 4. EIP + 1
00344553 90 NOP
00344554 B9 DCC30000 MOV ECX,0C3DC
00344559 C1E9 02 SHR ECX,2 ; ECX = 30F7
00344573 8DB5 46304000 LEA ESI,DWORD PTR SS:[EBP+403046] ; 34458E
00344579 33D2 XOR EDX,EDX
0034457B F7E3 MUL EBX
0034457D 81C2 2635B204 ADD EDX,4B23526
00344583 3116 XOR DWORD PTR DS:[ESI],EDX
00344585 8BC3 MOV EAX,EBX
00344587 8BDA MOV EBX,EDX
00344589 83C6 04 ADD ESI,4
0034458C ^ E2 EB LOOPD SHORT 00344579 ; 解密一段代码
0034458E 8DB5 AF314000 LEA ESI,DWORD PTR SS:[EBP+4031AF] ; 3446F7
003446E3 B9 2E230000 MOV ECX,232E
003446E8 C1E9 02 SHR ECX,2 ; 8CB
003446EB 90 NOP ; Dr3 中断, 我们做 3 件事( 见 3434BA 分析)
; 1. [3434BF] ++
; 2. xor byte ptr [ESI], 55
; 3. EIP+1
003446EC 90 NOP
003446ED 802E 13 SUB BYTE PTR DS:[ESI],13
003446F0 F616 NOT BYTE PTR DS:[ESI]
003446F2 83C6 04 ADD ESI,4
003446F5 ^ E2 F4 LOOPD SHORT 003446EB ; 反复 Dr3 中断



# 代码处理与异常处理分析

## 1. 代码解密部分

; 自己写一段代码处理一下了, 否则累死你  
; 注意最后 `[3434BF] = 8CE`  

处理好后继续  

```asm
003446F7    8DB5 30274000   LEA ESI,DWORD PTR SS:[EBP+402730]        ; 343C78

0034484C    8BFE            MOV EDI,ESI

0034484E    B9 F6000000     MOV ECX,0F6

00344853    AC              LODS BYTE PTR DS:[ESI]

00344854    32C1            XOR AL,CL

00344856    04 63           ADD AL,63

00344858    AA              STOS BYTE PTR ES:[EDI]

00344859    ^ E2 F8         LOOPD SHORT 00344853                      ; 对 除零异常处理代码的解密

0034485B    B8 00010000     MOV EAX,100

0034488D    33D2            XOR EDX,EDX

0034488F    33DB            XOR EBX,EBX

00344891    F7F3            DIV EBX                                     ; 除零异常, 重新设置 Dr0, 1, 2, 3

00344893    90              NOP

00344894    64:8F05 0000000>POP DWORD PTR FS:[0]                        ; 拆除异常

0034489B    58              POP EAX                                     ; 取消硬件断点, 进入第三部分
```

## 2. SEH 处理逻辑

; SEH

; 4 类情况

```asm
003434BA    E8 04000000     CALL 003434C3

003434BF    0000            ADD BYTE PTR DS:[EAX],AL                    ; 这里是一个单步中断的记数器

003434C1    0000            ADD BYTE PTR DS:[EAX],AL

003434C3    5A              POP EDX

003434C4    8B4424 04       MOV EAX,DWORD PTR SS:[ESP+4]                ; pEXCEPTION_RECORD

003434C8    8B00            MOV EAX,DWORD PTR DS:[EAX]                  ; Exception_Code

003434CA    8B4C24 0C       MOV ECX,DWORD PTR SS:[ESP+C]                ; pContext

003434CE    C701 17000100   MOV DWORD PTR DS:[ECX],10017                ; ContextFlags

00343501    FF81 B8000000   INC DWORD PTR DS:[ECX+B8]                   ; regEIP + 1

00343507    3D 03000080     CMP EAX,80000003                            ; int3 的处理

0034350C    /0F85 05010000  JNZ 00343617

00343512    8B81 B4000000   MOV EAX,DWORD PTR DS:[ECX+B4]               ; regEBP

00343518    8D80 6F2C4000   LEA EAX,DWORD PTR DS:[EAX+402C6F]           ; 3441B7

0034354B    8941 04         MOV DWORD PTR DS:[ECX+4],EAX                ; Dr0 = 3441B7, OD 加入硬件执行断点

0034354E    8B81 B4000000   MOV EAX,DWORD PTR DS:[ECX+B4]

00343554    8D80 612E4000   LEA EAX,DWORD PTR DS:[EAX+402E61]           ; 3443A9

00343587    8941 08         MOV DWORD PTR DS:[ECX+8],EAX                ; Dr1 = 3443A9, OD 加入硬件执行断点

0034358A    8B81 B4000000   MOV EAX,DWORD PTR DS:[ECX+B4]

00343590    8D80 0A304000   LEA EAX,DWORD PTR DS:[EAX+40300A]           ; 344552

003435C3    8941 0C         MOV DWORD PTR DS:[ECX+C],EAX                ; Dr2 = 344552, OD 加入硬件执行断点

003435C6    8B81 B4000000   MOV EAX,DWORD PTR DS:[ECX+B4]

003435CC    8D80 A3314000   LEA EAX,DWORD PTR DS:[EAX+4031A3]           ; 3446EB

003435FF    8941 10         MOV DWORD PTR DS:[ECX+10],EAX               ; Dr3 = 3446EB, OD 加入硬件执行断点

00343602    33C0            XOR EAX,EAX

00343604    8161 14 F00FFFF>AND DWORD PTR DS:[ECX+14],FFFF0FF0          ; Dr6

0034360B    C741 18 5501000>MOV DWORD PTR DS:[ECX+18],155               ; Dr7

00343612    E9 5A070000     JMP 00343D71                                ; 结束异常处理

00343617    3D 1D0000C0     CMP EAX,C000001D                            ; 非法指令的处理

0034361C    0F85 8E010000   JNZ 003437B0

00343622    8D81 B8000000   LEA EAX,DWORD PTR DS:[ECX+B8]               
```

```asm
0012FBF4    FF00            INC DWORD PTR DS:[EAX]                      ; regEIP+1

0012FBF6    8B00            MOV EAX,DWORD PTR DS:[EAX]

0012FBF8    F610            NOT BYTE PTR DS:[EAX]                        ; 把这里的 代码变成 CC(int3)

0012FBFA    C3              RETN                                        ; 返回后马上退出异常处理
```

```asm
003437B0    3D 04000080     CMP EAX,80000004                            ; Dr0, Dr1, Dr2, Dr3 引起的单步中断在这里处理

003437B5    0F85 B9040000   JNZ 00343C74

003437BB    FF02            INC DWORD PTR DS:[EDX]                      ; 3434BF 记数器 + 1

003437BD    8B02            MOV EAX,DWORD PTR DS:[EDX]

003437BF    83F8 01         CMP EAX,1                                   ; 第一次

003437C2    /75 0B          JNZ SHORT 003437CF

003437C4    |F791 B0000000  NOT DWORD PTR DS:[ECX+B0]                   ; regEAX 取反 (取反后 EAX = FFEE1164)

003437CA    |E9 9E040000    JMP 00343C6D

003437CF    83F8 02         CMP EAX,2                                   ; 第二次

003437D2    0F85 60010000   JNZ 00343938

003437D8    8B81 B0000000   MOV EAX,DWORD PTR DS:[ECX+B0]               ; regEAX

0012FBEC    C1C0 13         ROL EAX,13                                  ; 移位后 regEAX = 4A26B05A

0034392D    8981 B0000000   MOV DWORD PTR DS:[ECX+B0],EAX

00343938    83F8 03         CMP EAX,3                                   ; 第三次

0034393B    0F85 D7010000   JNZ 00343B18

00343941    53              PUSH EBX

00343942    8181 B0000000 2>ADD DWORD PTR DS:[ECX+B0],4B23526           ; regEAX + 4B23526 = 00950D59

00343979    8B81 B0000000   MOV EAX,DWORD PTR DS:[ECX+B0]               ; regEAX

0034397F    8B99 A4000000   MOV EBX,DWORD PTR DS:[ECX+A4]               ; regEBX

0012FBE4    66:93           XCHG AX,BX                                  ; regEBX = DAB20D59

0012FBE6    66:01D8         ADD AX,BX                                   ; regEAX = 009526C4

00343B06    8981 B0000000   MOV DWORD PTR DS:[ECX+B0],EAX

00343B0C    8999 A4000000   MOV DWORD PTR DS:[ECX+A4],EBX

00343B18    8B81 A0000000   MOV EAX,DWORD PTR DS:[ECX+A0]               ; 第四次以上吗, regESI

0012FBEC    8230 55         XOR BYTE PTR DS:[EAX],55                    ; xor byte ptr [regESI], 55

00343C74    3D 940000C0     CMP EAX,C0000094                            ; 除零错, (这部分代码是刚解密出来的)

00343C79    0F85 EF000000   JNZ 00343D6E

00343C7F    C702 00000000   MOV DWORD PTR DS:[EDX],0                    ; [3434BF]=0

00343C85    FF81 B8000000   INC DWORD PTR DS:[ECX+B8]                   ; regEIP + 1

00343C8D    C741 04 2301FF0>MOV DWORD PTR DS:[ECX+4],0FFF0123          ; Dr0 = 0FFF0123

00343CC1    C741 08 6745FF0>MOV DWORD PTR DS:[ECX+8],0FFF4567          ; Dr1 = 0FFF4567

00343CDF    C741 0C AB89FF0>MOV DWORD PTR DS:[ECX+C],0FFF89AB          ; Dr2 = 0FFF89AB

00343D13    C741 10 EFCDFF0>MOV DWORD PTR DS:[ECX+10],0FFFCDEF         ; Dr3 = 0FFFCDEF

00343D31    8161 14 F00FFFF>AND DWORD PTR DS:[ECX+14],FFFF0FF0         ; Dr6 = 0

00343D65    C741 18 5501000>MOV DWORD PTR DS:[ECX+18],155               ; Dr7 = 155
```

## 3. SEH = 3448D7 的处理

```asm
0034489C    8BFC            MOV EDI,ESP                                 ; 12FFA4

0034489E    8DA5 21F44000   LEA ESP,DWORD PTR SS:[EBP+40F421]           ; 350969

003448A4    B9 9AC00000     MOV ECX,0C09A

003448A9    8B85 A6F44000   MOV EAX,DWORD PTR SS:[EBP+40F4A6]           ; CEE46188

003448AF    BB BDD89800     MOV EBX,98D8BD

003448B4    BE D5260000     MOV ESI,26D5

003448B9    33D2            XOR EDX,EDX

003448BB    F7E6            MUL ESI

003448BD    05 78563412     ADD EAX,12345678

003448C2    83D2 00         ADC EDX,0

003448C5    F7F3            DIV EBX

003448C7    58              POP EAX

003448C8    32C2            XOR AL,DL

003448CA    50              PUSH EAX

003448CB    4C              DEC ESP

003448CC    8BC2            MOV EAX,EDX

003448CE    ^ E2 E9         LOOPD SHORT 003448B9                        ; 解密下面这段代码 (主要是 SEH 3448D7)

003448D0    8BE7            MOV ESP,EDI                                 ; 恢复 Stack

003448D2    E8 06010000     CALL 003449DD

003448D7                                       ; SEH

003449DD    64:FF35 0000000>PUSH DWORD PTR FS:[0]

003449E4    64:8925 0000000>MOV DWORD PTR FS:[0],ESP

003449EB    8DB5 66394000   LEA ESI,DWORD PTR SS:[EBP+403966]           ; 344EAE
```



### 异常处理循环

```asm
003449F1    B9 A7040000     MOV ECX,4A7                   ; size = 4A7, 345355  

003449F6    8A0431          MOV AL,BYTE PTR DS:[ECX+ESI]  

003449F9    CC              INT3                          ; Int3, 进入下面的异常处理  

003449FA    90              NOP  

003449FB    880431          MOV BYTE PTR DS:[ECX+ESI],AL  

003449FE  ^ E2 F6           LOOPD SHORT 003449F6          ; 要反复异常 4A7 次, 厉害  

00344A00    64:8F05 0000000> POP DWORD PTR FS:[0]         ; 拆除异常, 进入第四部分  

00344A07    58              POP EAX  

;SEH  

; 第一次进入 SEH 后先设好 Drx  

dr0 0012FCD4   0FFF0123  

dr1 0012FCD8   0FFF4567  

dr2 0012FCDC   0FFF89AB  

dr3 0012FCE0   0FFFCDEF  

003448D7    8B4C24 0C       MOV ECX,DWORD PTR SS:[ESP+C]    ; pContext=12FCD0  

003448DB    8B81 B0000000   MOV EAX,DWORD PTR DS:[ECX+B0]   ; regEAX  

003448E1    8B51 04         MOV EDX,DWORD PTR DS:[ECX+4]    ; Dr0  

00344911    F6D0            NOT AL  

00344913    32C2            XOR AL,DL  

00344915    66:25 FF00      AND AX,0FF  

00344930    66:03D0         ADD DX,AX  

00344933    66:C1CA 03      ROR DX,3  

00344937    66:8951 04      MOV WORD PTR DS:[ECX+4],DX      ; Dr0  

00344952    66:3151 08      XOR WORD PTR DS:[ECX+8],DX      ; Dr1  

00344956    66:8B51 08      MOV DX,WORD PTR DS:[ECX+8]  

0034495A    66:C1CA 02      ROR DX,2  

0034498B    66:0151 0C      ADD WORD PTR DS:[ECX+C],DX      ; Dr2  

0034498F    66:8B51 0C      MOV DX,WORD PTR DS:[ECX+C]      

00344993    66:F7D2         NOT DX  

00344996    66:2B51 10      SUB DX,WORD PTR DS:[ECX+10]     ; Dr3  

0034499A    66:D1CA         ROR DX,1  

003449CA    66:3151 04      XOR WORD PTR DS:[ECX+4],DX      ; Dr1  

003449CE    8981 B0000000   MOV DWORD PTR DS:[ECX+B0],EAX   ; regEAX  

003449D4    FF81 B8000000   INC DWORD PTR DS:[ECX+B8]       ; regEIP+1  

003449DA    33C0            XOR EAX,EAX  

003449DC    C3              RETN                            ; 第一次到这里后如下操做  

在 344A07 设软件断点, 忽略 Int3 异常,  F9, 停下后, 去 Stack 把 最终的 Drx 找回来, 第十一部分还要检查  

0012FCD4   0FFF4A3D  

0012FCD8   0FFF23B9  

0012FCDC   0FFF9002  

0012FCE0   0FFFCDEF  

0012FCE4   0  

0012FCE8   155  

进入第四部分  

;============================================================================================================================================================================  

### 四. 处理壳要用的 API  

00344B57    8D05 A9F24000   LEA EAX,DWORD PTR DS:[40F2A9]  

00344B5D    03C5            ADD EAX,EBP                   ; 3507F1  

00344B5F    8985 22F44000   MOV DWORD PTR SS:[EBP+40F422],EAX  

00344B65    8DB5 3BF54000   LEA ESI,DWORD PTR SS:[EBP+40F53B]    ; 350A83 "kernel32.dll"  

00344B6B    56              PUSH ESI  

00344B6C    8D85 37364000   LEA EAX,DWORD PTR SS:[EBP+403637]    ; 返回地址 344B7F  

00344B72    50              PUSH EAX  

00344B73    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]    ; KERNEL32.GetModuleHandleA  

00344B79    E9 3EB70000     JMP 003502BC                  ; 3502BC 相当于 Call EAX, 返回地址前面给定  

00344B8E  

00344B7F    8BF0            MOV ESI,EAX                   ; KERNEL32.7C570000  

00344B81    8985 3BF54000   MOV DWORD PTR SS:[EBP+40F53B],EAX    ; 350A83, 覆盖 字符  

00344B87    8D9D 49F54000   LEA EBX,DWORD PTR SS:[EBP+40F549]    ; 350A91  

00344CDC    B9 21000000     MOV ECX,21                    ; 21 个 API(kernel32.dll)  

00344CE1    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]    ; 350F44  

00344CE7    53              PUSH EBX  

00344CE8    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344CEA   /EB 06           JMP SHORT 00344CF2  

00344CEC   |F6D0            NOT AL                        ; 只是简单的 Not 解密字符  

00344CEE   |AA              STOS BYTE PTR ES:[EDI]  

00344CEF   |43              INC EBX  

00344CF0   |8A03            MOV AL,BYTE PTR DS:[EBX]  

00344CF2   \\0AC0           OR AL,AL  

00344CF4  ^ 75 F6           JNZ SHORT 00344CEC  

00344CF6    AA              STOS BYTE PTR ES:[EDI]  

00344CF7    5B              POP EBX                         ; 00350A91  

00344CF8    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]   ; 350F44 ( API 明文)  

00344CFE    51              PUSH ECX  

00344CFF    57              PUSH EDI  

00344D00    56              PUSH ESI  

00344D01    8D85 CC374000   LEA EAX,DWORD PTR SS:[EBP+4037CC]  ; 返回地址 344D14  

00344D07    50              PUSH EAX  

00344D08    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]  ; 3507F1, 相当于 GetProcAddress  

00344D0E    E9 A9B50000     JMP 003502BC                    

00344D13  

00344D14    0FB64B FF       MOVZX ECX,BYTE PTR DS:[EBX-1]  

00344D18    8903            MOV DWORD PTR DS:[EBX],EAX      ; [350A91] 放 API  

00344D1A    03D9            ADD EBX,ECX  

00344D1C    43              INC EBX  

00344D1D    59              POP ECX  

00344D1E  ^ E2 C1           LOOPD SHORT 00344CE1  

有 `CloseHandle`, `CreateFileA`, `CreateFileMappingA`, `CreateThread`, `DeleteFileA`, `DeviceIoControl`,  

`ExitProcess`, `FindResourceA`, `GetCommandLineA`, `GetFileSize`, `GetCurrentProcess`, `GetCurrentProcessId`,  

`GetCurrentThread`, `GetModuleHandleA`, `GetModuleFileNameA`, `GetTempPathA`, `GetVersion`, `lstrcmp`,  

`LoadResource`, `MapViewOfFile`, `ReadProcessMemory`, `ResetEvent`, `SetEvent`, `SetLastError`, `SetThreadPriority`,  

`TerminateThread`, `UnmapViewOfFile`, `VirtualAllocEx`, `VirtualFree`, `VirtualProtect`, `WaitForSingleObject`,  

`WriteProcessMemory`, `WriteFile`  

00344D20    8DB5 AEF44000   LEA ESI,DWORD PTR SS:[EBP+40F4AE]  ; 3509F6 ("USER32.DLL")  

00344E75    56              PUSH ESI  

00344E76    8D85 41394000   LEA EAX,DWORD PTR SS:[EBP+403941]  ; 返回地址 344E89  

00344E7C    50              PUSH EAX  

00344E7D    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]  ; KERNEL32.GetModuleHandleA  

00344E83   /E9 34B40000     JMP 003502BC                     

00344E88  

00344E89    0BC0            OR EAX,EAX                      ; 找不到  

00344E8B    75 15           JNZ SHORT 00344EA2  

00344E8D    56              PUSH ESI  

00344E8E    8D85 5A394000   LEA EAX,DWORD PTR SS:[EBP+40395A]  ; 返回地址 344EA2     

00344E94    50              PUSH EAX  

00344E95    8B85 2EF44000   MOV EAX,DWORD PTR SS:[EBP+40F42E]  ; KERNEL32.LoadLibraryA  

00344E9B   /E9 1CB40000     JMP 003502BC  

00344EA0  

00344EA1  

00344EA2    8BF0            MOV ESI,EAX                   ; USER32.77E10000  

00344EA4    8D9D BAF44000   LEA EBX,DWORD PTR SS:[EBP+40F4BA]  ; 350A02  

00344EAA    B9 08000000     MOV ECX,8  

00344EAF    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  

00344EB5    53              PUSH EBX  

00344EAF    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  ; 350F44  

00344EB5    53              PUSH EBX  

00344EE3    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344EE5    EB 06           JMP SHORT 00344EED  

00344EE7    F6D0            NOT AL  

00344EE9    AA              STOS BYTE PTR ES:[EDI]  

00344EEA    43              INC EBX  

00344EEB    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344EED    0AC0            OR AL,AL  

00344EEF  ^ 75 F6           JNZ SHORT 00344EE7  

00344EF1    AA              STOS BYTE PTR ES:[EDI]          ; 解密字符  

00344EF2    5B              POP EBX  

00344EF3    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  ; 350F44  

00344EF9    51              PUSH ECX  

00344EFA    57              PUSH EDI  

00344EFB    56              PUSH ESI  

00344EFC    8D85 C8394000   LEA EAX,DWORD PTR SS:[EBP+4039C8]  ; 返回地址, 344F10  

00344F02    50              PUSH EAX  

00344F03    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]  ; 3507F1, 相当于 GetProcAddress  

00344F09    E9 AEB30000     JMP 003502BC  
```



# 汇编代码分析

## 第一部分：指令列表

```
00344F10    0FB64B FF       MOVZX ECX,BYTE PTR DS:[EBX-1]
00344F14    8903            MOV DWORD PTR DS:[EBX],EAX
00344F16    03D9            ADD EBX,ECX
00344F18    43              INC EBX
00344F19    59              POP ECX
00344F1A  ^ E2 93           LOOPD SHORT 00344EAF
```

涉及函数：`CreateDialogIndirectParamA`, `DialogBoxIndirectParamA`, `EnumWindows`, `GetWindowTextA`, `MessageBoxA`, `SendMessageA`, `SetTimer`, `wsprintfA`

```
00344F1C    8DB5 4AF74000   LEA ESI,DWORD PTR SS:[EBP+40F74A]    ; 00350C92 "WS2_32.DLL"
00345071    56              PUSH ESI
00345072    8D85 3D3B4000   LEA EAX,DWORD PTR SS:[EBP+403B3D]    ; 返回地址, 345085
00345078    50              PUSH EAX
00345079    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]    ; KERNEL32.GetModuleHandleA
0034507F    E9 38B20000     JMP 003502BC
00345085    0BC0            OR EAX,EAX
00345087    75 15           JNZ SHORT 0034509E
00345089    56              PUSH ESI
0034508A    8D85 563B4000   LEA EAX,DWORD PTR SS:[EBP+403B56]    ; 返回地址, 34509E
00345090    50              PUSH EAX
00345091    8B85 2EF44000   MOV EAX,DWORD PTR SS:[EBP+40F42E]    ; KERNEL32.LoadLibraryA
00345097    E9 20B20000     JMP 003502BC
0034509E    8BF0            MOV ESI,EAX                             ; WS2_32.#390
003450A0    8D9D 56F74000   LEA EBX,DWORD PTR SS:[EBP+40F756]     ; 350C9E
003450A6    B9 04000000     MOV ECX,4
003450AB    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]     ; 350F44
003450B1    53              PUSH EBX
003450B2    8A03            MOV AL,BYTE PTR DS:[EBX]
003450B4    EB 06           JMP SHORT 003450BC
003450B6    F6D0            NOT AL
003450B8    AA              STOS BYTE PTR ES:[EDI]
003450B9    43              INC EBX
003450BA    8A03            MOV AL,BYTE PTR DS:[EBX]
003450BC    0AC0            OR AL,AL
003450BE  ^ 75 F6           JNZ SHORT 003450B6
003450C0    AA              STOS BYTE PTR ES:[EDI]
003450C1    5B              POP EBX
003450C2    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]     ; 350F44
003450C8    51              PUSH ECX
003450C9    57              PUSH EDI
003450CA    56              PUSH ESI
003450CB    8D85 973B4000   LEA EAX,DWORD PTR SS:[EBP+403B97]     ; 返回地址, 3450DF
003450D1    50              PUSH EAX
003450D2    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]     ; 3507F1, 相当于 GetProcAddress
003450D8    E9 DFB10000     JMP 003502BC
003450DF    0FB64B FF       MOVZX ECX,BYTE PTR DS:[EBX-1]
003450E3    8903            MOV DWORD PTR DS:[EBX],EAX            ; 350C9E
003450E5    03D9            ADD EBX,ECX
003450E7    43              INC EBX
003450E8    59              POP ECX
003450E9  ^ E2 C0           LOOPD SHORT 003450AB
```

涉及函数：`WSASend`, `WSARecv`, `send`, `recv`

```
003450EB    8DB5 73F74000   LEA ESI,DWORD PTR SS:[EBP+40F773]     ; 350cbb, ("advapi32.dll")
00345240    56              PUSH ESI
00345241    8D85 0C3D4000   LEA EAX,DWORD PTR SS:[EBP+403D0C]     ; 返回地址, 345254
00345247    50              PUSH EAX
00345248    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]     ; KERNEL32.GetModuleHandleA
0034524E    E9 69B00000     JMP 003502BC
00345254    0BC0            OR EAX,EAX                              ; ADVAPI32.7C2D0000
00345256    75 15           JNZ SHORT 0034526D
0034526D    8BF0            MOV ESI,EAX                             ; ADVAPI32.7C2D0000
0034526F    8D9D 81F74000   LEA EBX,DWORD PTR SS:[EBP+40F781]     ; 350CC9
00345275    B9 08000000     MOV ECX,8
0034527A    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]     ; 350FF4
00345280    53              PUSH EBX
00345281    8A03            MOV AL,BYTE PTR DS:[EBX]
00345283    EB 06           JMP SHORT 0034528B
00345285    F6D0            NOT AL
00345287    AA              STOS BYTE PTR ES:[EDI]
00345288    43              INC EBX
00345289    8A03            MOV AL,BYTE PTR DS:[EBX]
0034528B    0AC0            OR AL,AL
0034528D  ^ 75 F6           JNZ SHORT 00345285
0034528F    AA              STOS BYTE PTR ES:[EDI]
00345290    5B              POP EBX
00345291    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]    ; 350F44
00345297    51              PUSH ECX
00345298    57              PUSH EDI
00345299    56              PUSH ESI
0034529A    8D85 663D4000   LEA EAX,DWORD PTR SS:[EBP+403D66]    ; 返回地址, 3452AE
003452A0    50              PUSH EAX
003452A1    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]    ; 3507F1, 相当于 GetProcAddress
003452A7    E9 10B00000     JMP 003502BC
003452AE    0FB64B FF       MOVZX ECX,BYTE PTR DS:[EBX-1]
003452B2    8903            MOV DWORD PTR DS:[EBX],EAX
003452B4    03D9            ADD EBX,ECX
003452B6    43              INC EBX
003452B7    59              POP ECX
003452B8  ^ E2 C0           LOOPD SHORT 0034527A
```

涉及函数：`CloseServiceHandle`, `ControlService`, `CreateServiceA`, `DeleteService`, `OpenSCManagerA`, `OpenServiceA`, `QueryServiceStatus`, `StartServiceA`

备注：想用驱动？

## 第二部分：API 寻址结束，进入第五部分

```
; API 寻址结束, 进入第五部分
;======================================================================================================================================================================
```

## 第三部分：检查调试器 (RDTSC, CreateFile, ZwSetInformationThread)

```
003452BA    8D85 853D4000   LEA EAX,DWORD PTR SS:[EBP+403D85]    ; 返回地址, 3452CD
003452C0    50              PUSH EAX
003452C1    8B85 05F64000   MOV EAX,DWORD PTR SS:[EBP+40F605]    ; KERNEL32.GetCurrentThread
003452C7    E9 F0AF0000     JMP 003502BC
003452CD    6A 02           PUSH 2                                ; THREAD_PRIORITY_HIGHEST
003452CF    50              PUSH EAX
003452D0    8D85 9B3D4000   LEA EAX,DWORD PTR SS:[EBP+403D9B]    ; 返回地址, 3452E3
003452D6    50              PUSH EAX
003452D7    8B85 B5F64000   MOV EAX,DWORD PTR SS:[EBP+40F6B5]    ; KERNEL32.SetThreadPriority
003452DD   /E9 DAAF0000     JMP 003502BC                          ; 提高优先级, 保证下面时间校验通过( 没调试情况下 )
003452E3    50              PUSH EAX
003452E4    52              PUSH EDX
003452E5    51              PUSH ECX
003452E9    0F31            RDTSC                                  ; CB8:B1F630AA
003452EB    8BC8            MOV ECX,EAX
```

... 一些垃圾代码, 拖延时间

```
0034531E    0F31            RDTSC
00345320    83C4 04         ADD ESP,4
00345323    2BC1            SUB EAX,ECX
00345325    3D 00000200     CMP EAX,20000
0034532A    76 04           JBE SHORT 00345330                     ; 一定要跳 **************************
0034532C    83C4 0C         ADD ESP,0C
0034532F    C3              RETN
00345330    59              POP ECX
00345331    5A              POP EDX
00345332    58              POP EAX
```

## 第四部分：内存校验与文件检查

```
00345365    8B85 36F44000   MOV EAX,DWORD PTR SS:[EBP+40F436]        ; adalinks.00400000
0034536B    0340 3C         ADD EAX,DWORD PTR DS:[EAX+3C]            ; PE 头
003454BD    8B40 50         MOV EAX,DWORD PTR DS:[EAX+50]            ; SizeOfImage
003454C0    3385 4AF44000   XOR EAX,DWORD PTR SS:[EBP+40F44A]        ; [350992] 后变成 507E3E
00345615    8B8D 26F44000   MOV ECX,DWORD PTR SS:[EBP+40F426]        ; [35096E] = 507E3E
0034561B    3BC1            CMP EAX,ECX
0034561D    8DB5 1FF94000   LEA ESI,DWORD PTR SS:[EBP+40F91F]        ; 350E67
00345623    46              INC ESI
00345773    B9 09000000     MOV ECX,9                                 ; 9 次调用 CreateFileA, 检查
00345778    E9 89010000     JMP 00345906
0034577D    51              PUSH ECX                                   ; 解密 字符
0034577E    56              PUSH ESI
0034577F    AC              LODS BYTE PTR DS:[ESI]
```



00345780    EB 06               JMP SHORT `00345788`

00345782    F6D0                NOT AL

00345784    8846 FF             MOV BYTE PTR `DS:[ESI-1]`,AL

00345787    AC                  LODS BYTE PTR `DS:[ESI]`

00345788    0AC0                OR AL,AL

0034578A  ^ 75 F6               JNZ SHORT `00345782`

0034578C    5E                  POP ESI

0034578D    6A 00               PUSH 0

0034578F    68 80000000         PUSH 80

00345794    6A 03               PUSH 3

00345796    6A 00               PUSH 0

00345798    6A 03               PUSH 3

0034579A    68 000000C0         PUSH C0000000

003457CC    56                  PUSH ESI

003457CD    8D85 98424000       LEA EAX,DWORD PTR `SS:[EBP+404298]`       ; 返回地址 3457E0

003457D3    50                  PUSH EAX

003457D4    8B85 56F54000       MOV EAX,DWORD PTR `SS:[EBP+40F556]`       ; `KERNEL32.CreateFileA`

003457DA    E9 DDAA0000         JMP `003502BC`

003457E0    50                  PUSH EAX

003457E1    52                  PUSH EDX

003457E2    51                  PUSH ECX

003457E6    0F31                RDTSC

003457E8    8BC8                MOV ECX,EAX

... 垃圾代码

0034581B    0F31                RDTSC

0034581D    83C4 04             ADD ESP,4

00345820    2BC1                SUB EAX,ECX

00345822    3D 00000200         CMP EAX,20000

00345827    76 04               JBE SHORT `0034582D`                     ;  一定要跳 `****************************************************************`

00345829    83C4 0C             ADD ESP,0C

0034582C    C3                  RETN

0034582D    59                  POP ECX

0034582E    5A                  POP EDX

0034582F    58                  POP EAX

00345862    83F8 FF             CMP EAX,-1

00345865    74 05               JE SHORT `0034586C`                     ;  一定要跳 `****************************************************************`

00345867    /E9 3CBD0000        JMP `003515A8`

0034586C    56                  PUSH ESI                                 ; 加密 字符

0034586D    AC                  LODS BYTE PTR `DS:[ESI]`

0034586E    EB 06               JMP SHORT `00345876`

00345870    F6D0                NOT AL

00345872    8846 FF             MOV BYTE PTR `DS:[ESI-1]`,AL

00345875    AC                  LODS BYTE PTR `DS:[ESI]`

00345876    0AC0                OR AL,AL

00345878  ^ 75 F6               JNZ SHORT `00345870`

0034587A    5E                  POP ESI

0034587B    59                  POP ECX

0034587C    0FB646 FF           MOVZX EAX,BYTE PTR `DS:[ESI-1]`

00345880    03F0                ADD ESI,EAX

00345882    46                  INC ESI

00345883    49                  DEC ECX

00345884    50                  PUSH EAX

00345885    52                  PUSH EDX

00345886    51                  PUSH ECX

0034588A    0F31                RDTSC

0034588C    8BC8                MOV ECX,EAX

... 垃圾代码

003458BF    0F31                RDTSC

003458C1    83C4 04             ADD ESP,4

003458C4    2BC1                SUB EAX,ECX

003458C6    3D 00000200         CMP EAX,20000

003458CB    76 04               JBE SHORT `003458D1`                   ;  一定要跳 `****************************************************************`

003458CD    83C4 0C             ADD ESP,0C

003458D0    C3                  RETN

003458D1    59                  POP ECX

003458D2    5A                  POP EDX

003458D3    58                  POP EAX

003458D4    EB 30               JMP SHORT `00345906`

00345906    0BC9                OR ECX,ECX

00345908  ^ 0F85 6FFEFFFF       JNZ `0034577D`

`\\.NTICE`

`\\.SICE`

`\\.TWX2002`

`\\.filemon`

`\\.regmon`

`\\.FILEVXD`

`\\.REGVXD`

`\\.ICEDUMP`

`\\.BW2K`

0034590E    8CC9                MOV CX,CS

00345910    32C9                XOR CL,CL

00345912    0BC9                OR ECX,ECX

00345914    0F84 B3020000       JE `00345BCD`

00345BCD    50                  PUSH EAX

00345BCE    52                  PUSH EDX

00345BCF    51                  PUSH ECX

00345BD3    0F31                RDTSC

00345BD5    8BC8                MOV ECX,EAX

... 垃圾代码

00345C08    0F31                RDTSC

00345C0A    83C4 04             ADD ESP,4

00345C0D    2BC1                SUB EAX,ECX

00345C0F    3D 00000200         CMP EAX,20000

00345C14    76 04               JBE SHORT `00345C1A`                   ;  一定要跳 `****************************************************************`

00345C16    83C4 0C             ADD ESP,0C

00345C19    C3                  RETN

00345C1A    59                  POP ECX

00345C1B    5A                  POP EDX

00345C1C    58                  POP EAX

00345C5E    8D85 78484000       LEA EAX,DWORD PTR `SS:[EBP+404878]`   ; 返回地址 345DC0

00345DB3    50                  PUSH EAX

00345DB4    8B85 2AF44000       MOV EAX,DWORD PTR `SS:[EBP+40F42A]`   ; `KERNEL32.GetModuleHandleA`

00345DBA    E9 FDA40000         JMP `003502BC`                         ; `ntdll.dll`

00345DC0    E8 17000000         CALL `00345DDC`

00345DDC    50                  PUSH EAX                               ; `ntdll.77F80000`

00345DDD    8D85 F7494000       LEA EAX,DWORD PTR `SS:[EBP+4049F7]`    ; 返回地址 345F3F

00345DE3    50                  PUSH EAX

00345F33    8B85 22F44000       MOV EAX,DWORD PTR `SS:[EBP+40F422]`    ; 3507F1, 相当于 `GetProcAddress`

00345F39    E9 7EA30000         JMP `003502BC`

00345F3F    8BF8                MOV EDI,EAX                            ; `ntdll.ZwSetInformationThread`

00345F41    8D85 0C4A4000       LEA EAX,DWORD PTR `SS:[EBP+404A0C]`    ; 返回地址 345F54

00345F47    50                  PUSH EAX

00345F48    8B85 05F64000       MOV EAX,DWORD PTR `SS:[EBP+40F605]`    ; `KERNEL32.GetCurrentThread`

00345F54    6A 00               PUSH 0                                 ; 4 个参数

00345F56    6A 00               PUSH 0

00345F58    6A 11               PUSH 11

003460A9    50                  PUSH EAX

003460AA    50                  PUSH EAX

003460AB    52                  PUSH EDX

003460AC    51                  PUSH ECX

003460B0    0F31                RDTSC

003460B2    8BC8                MOV ECX,EAX

... 垃圾代码

003460E5    0F31                RDTSC

003460E7    83C4 04             ADD ESP,4

003460EA    2BC1                SUB EAX,ECX

003460EC    3D 00000200         CMP EAX,20000

003460F1    76 04               JBE SHORT `003460F7`                  ;  一定要跳 `****************************************************************`

003460F3    83C4 0C             ADD ESP,0C

003460F6    C3                  RETN

003460F7    59                  POP ECX

003460F8    5A                  POP EDX

003460F9    58                  POP EAX

003460FA    EB 30               JMP SHORT `0034612C`

0034612C    8D85 F34B4000       LEA EAX,DWORD PTR `SS:[EBP+404BF3]`   ; 返回地址 34613B

00346132    50                  PUSH EAX

00346133    8BC7                MOV EAX,EDI                           ; `ntdll.ZwSetInformationThread`

00346135    E9 82A10000         JMP `003502BC`                         ; 这里不能跳 `********************************************************************`

                                                     ; 直接把 EIP 改到 34613B, 5 次 POP DWORD, 参数出栈

`;===========================================================================================================================================================================`

## 六. 用户代码解压

0034613B    8D1D 9AFC4000       LEA EBX,DWORD PTR `DS:[40FC9A]`       ; 40FC9A

00346141    833C2B 00           CMP DWORD PTR `DS:[EBX+EBP]`,0        ; `[3511E2]=8B000`

00346145    0F84 51070000       JE `0034689C`                         ; 解压结束跳到 第七部分

0034615A    8D042B              LEA EAX,DWORD PTR `DS:[EBX+EBP]`      ; 3511E2

00346167    8B48 08             MOV ECX,DWORD PTR `DS:[EAX+8]`        ; 26000

00346174    8B70 04             MOV ESI,DWORD PTR `DS:[EAX+4]`        ; 1000

00346181    03B5 36F44000       ADD ESI,DWORD PTR `SS:[EBP+40F436]`   ; 400000

00346191    8BFE                MOV EDI,ESI



```
00346193    53                 PUSH EBX
00346194    E8 5D000000        CALL 003461F6
0012FF78    52                 PUSH EDX                                ; 对压缩的数据先做一变换
0012FF79    BA 2635B204        MOV EDX, 4B23526
0012FF7E    AC                 LODS BYTE PTR DS:[ESI]
0012FF7F    D2C8               ROR AL, CL
0012FF81    30C8               XOR AL, CL
0012FF83    04 65              ADD AL, 65
0012FF85    30E8               XOR AL, CH
0012FF87    00F0               ADD AL, DH
0012FF89    28D0               SUB AL, DL
0012FF8B    00C8               ADD AL, CL
0012FF8D    28E8               SUB AL, CH
0012FF8F    30D0               XOR AL, DL
0012FF91    04 23              ADD AL, 23
0012FF93    30F0               XOR AL, DH
0012FF95    F6D0               NOT AL
0012FF97    D2C8               ROR AL, CL
0012FF99    D3CA               ROR EDX, CL
0012FF9B    8846 FF            MOV BYTE PTR DS:[ESI-1], AL
0012FF9E    49                 DEC ECX
0012FF9F   ^ 75 DD             JNZ SHORT 0012FF7E
0012FFA1    5A                 POP EDX
0012FFA2    C3                 RETN

003462DD    53                 PUSH EBX                                  ; adalinks.0040FC9A
003462DE    6A 04              PUSH 4
003462E0    68 00100000        PUSH 1000
003462E5    FF342B             PUSH DWORD PTR DS:[EBX+EBP]               ; 8B000
003462E8    6A 00              PUSH 0
003462EA    8D85 96514000      LEA EAX, DWORD PTR SS:[EBP+405196]          ; 返回地址 3466DE (很远啊)
003462F0    50                 PUSH EAX
003462F1    8B85 32F44000      MOV EAX, DWORD PTR SS:[EBP+40F432]          ; KERNEL32.VirtualAlloc
003462F7    E9 C09F0000        JMP 003502BC
003466DE    5B                 POP EBX                                   ; adalinks.0040FC9A
0034682E    8BF0               MOV ESI, EAX                              ; 930000( 分配的空间)
00346830    8BC3               MOV EAX, EBX
00346849    03C5               ADD EAX, EBP                              ; 3511E2
0034684B    8B78 04            MOV EDI, DWORD PTR DS:[EAX+4]             ; 1000
0034684E    03BD 36F44000      ADD EDI, DWORD PTR SS:[EBP+40F436]        ; 40000
00346854    56                 PUSH ESI
00346855    57                 PUSH EDI
00346856    8D85 1E534000      LEA EAX, DWORD PTR SS:[EBP+40531E]        ; 返回地址 346866
0034685C    50                 PUSH EAX
0034685D    8B85 A2F44000      MOV EAX, DWORD PTR SS:[EBP+40F4A2]        ; 490151 解压用户代码到 930000
00346863    FFE0               JMP EAX                                   ; call 的变形
00346866    8B0C2B             MOV ECX, DWORD PTR DS:[EBX+EBP]           ; 8B000
00346869    56                 PUSH ESI
0034686A    51                 PUSH ECX
0034686B    C1E9 02            SHR ECX, 2
0034686E    F3:A5              REP MOVS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI]  ; 930000->401000
00346870    59                 POP ECX
00346871    83E1 03            AND ECX, 3
00346874    F3:A4              REP MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]    ; 代码解压完成了
00346876    5E                 POP ESI
00346877    53                 PUSH EBX                                  ; 40FC9A
00346878    68 00800000        PUSH 8000
0034687D    6A 00              PUSH 0
0034687F    56                 PUSH ESI                                  ; 930000
00346880    8D85 4B534000      LEA EAX, DWORD PTR SS:[EBP+40534B]        ; 返回地址 346893
00346886    50                 PUSH EAX
00346887    8B85 FAF64000      MOV EAX, DWORD PTR SS:[EBP+40F6FA]        ; KERNEL32.VirtualFree
0034688D    E9 2A9A0000        JMP 003502BC
00346893    5B                 POP EBX                                   ; adalinks.0040FC9A
00346894    83C3 0C            ADD EBX, 0C
00346897   ^ E9 A5F8FFFF      JMP 00346141                              ; 一段代码解压完成了, 循环下一段
```

### 七. 处理 IAT

```
;   003515D7   FF FF FF FF 0C 4D 53 56 42 56 4D 36 30 2E 44 4C   .MSVBVM60.DLL
;   003515E7   4C 00 C5 00 00 80 18 D5 F3 D5 F2 B3 10 93 52 F2
;   003515F7   92 10 15 D1 BF 52 B5 9F 12 F1 F2 51 DE D1 9F 00
;   00351607   0D 10 10 FF 71 51 F3 51 7F B3 9F BF 15 BF 00
```

到这一步，加密后的 IAT 在内存里如下存放：

```
; 1. FF FF FF FF    ----------- 00 00 00 00 表示 所有 DLL 结束
; 2. xx   ----- DLL Name　长度(不算 null)
; 3. DLL 名字, null 结尾 ( 明文 )
; 4. 80 yy yy yy    ---------- yy yy yy 表示　API 个数 ,   80 表示需要重定向
; 5. zz             ---------- ZZ<>0 表示 API Name　长度(不算 null),   ZZ==0, 后 4 byte 函数序号, 1 byte NULL
; 6. API Name, null 结尾 ( 密文, 解密代码见 12FF68 )
; 7. 重复 5,  6 结束 一个 DLL
; 重复 1,..,7 处理 所有 DLL
```

经过壳的 IAT 处理，形成了下面的一个调用过程，（引用 Window 的表示）：

```
;
; iat 中地址 --> `Hook_proc`:
; `Hook_proc`:
;             |PUSH DWORD PTR DS:[`Hook_proc`+1C]
;             |XOR DWORD PTR SS:[`ESP`], `key`
;             |ret; -> |`Stub_proc`:
;                       |`api_start_code`
;                       |`api_some_code`
;                       |push `api_next_code_addr`
;                       |ret
;
; `Hook_proc`+1C:
;   `Stub_proc_xor_key`
```

经过对壳代码 `348A32`、`34A2DC` 的修改，可以变成下面的形式：

```
;
; iat 中地址 --> `Hook_proc`:
; `Hook_proc`:
;             |PUSH DWORD PTR DS:[`Hook_proc`+1C]
;             |XOR DWORD PTR SS:[`ESP`], `key`
;             |ret; ->   `APIaddress`
;
; `Hook_proc`+1C:
;   `APIaddress_xor_key`
```

```
0034689C    8DB5 DD544000    LEA ESI, DWORD PTR SS:[EBP+4054DD]          ; 346A25
003469F1    87E6             XCHG ESI, ESP
003469F3    B9 43780000      MOV ECX, 7843
003469F8    58               POP EAX
003469F9    F6D0             NOT AL
003469FB    50               PUSH EAX
003469FC    44               INC ESP
003469FD   ^ E2 F9           LOOPD SHORT 003469F8                        ; 在 Stack 执行, 解密 346A25 开始的代码
003469FF    87E6             XCHG ESI, ESP                               ; 恢复 Stack
00346A01    6A 04            PUSH 4
00346A03    68 00100000      PUSH 1000
00346A08    68 00200000      PUSH 2000
00346A0D    6A 00            PUSH 0
00346A0F    FF95 32F44000    CALL DWORD PTR SS:[EBP+40F432]              ; KERNEL32.VirtualAlloc, 这一段空间是把 API 的开头部分搬到这里
00346A15    8985 22FD4000    MOV DWORD PTR SS:[EBP+40FD22], EAX          ; [35126A]=930000
00346A1B    C785 26FD4000 0> MOV DWORD PTR SS:[EBP+40FD26], 0
00346A25    8B85 66F44000    MOV EAX, DWORD PTR SS:[EBP+40F466]          ; [3509AE]=1 , IAT 被加密的标志
00346A58    0BC0             OR EAX, EAX
00346A5A    0F85 96090000    JNZ 003473F6
003473F6    8D95 C91A4000    LEA EDX, DWORD PTR SS:[EBP+401AC9]         ; 343011
00347429    0395 86F44000    ADD EDX, DWORD PTR SS:[EBP+40F486]         ; E5C6
0034742F    8B3A             MOV EDI, DWORD PTR DS:[EDX]                ; [3515D7]= FF FF FF FF, IAT 开始的地方
00347580    0BFF             OR EDI, EDI
00347582    75 05            JNZ SHORT 00347589
00347584    E9 6C340000      JMP 0034A9F5                                ; 所有 DLL 处理完了, 到这里
00347589    03BD 36F44000    ADD EDI, DWORD PTR SS:[EBP+40F436]       ; adalinks.00400000
003476DE    83C2 05          ADD EDX, 5                                ; EDX 指向 DLL 的名字
0034770E    8BF2             MOV ESI, EDX                              ; ESI 指向 DLL 的名字
00347710    56               PUSH ESI
00347711    8D85 0A624000    LEA EAX, DWORD PTR SS:[EBP+40620A]       ; 返回地址 347752
00347744    50               PUSH EAX
00347745    8B85 2AF44000    MOV EAX, DWORD PTR SS:[EBP+40F42A]       ; KERNEL32.GetModuleHandleA
0034774B    E9 6C8B0000      JMP 003502BC
00347752    0BC0             OR EAX, EAX
00347754    75 1E            JNZ SHORT 00347774                       ; DLL 已加载
00347756    56               PUSH ESI
00347757    8D85 23624000    LEA EAX, DWORD PTR SS:[EBP+406223]       ; 返回地址 34776B
0034775D    50               PUSH EAX
```



0034775E `8B85 2EF44000` `MOV EAX,DWORD PTR SS:\[EBP+40F42E\]` `; KERNEL32.LoadLibraryA`

00347764 `E9 538B0000` `JMP 003502BC`

0034776B `0BC0` `OR EAX,EAX` `; DLL base Memory`

0034776D `75 05` `JNZ SHORT 00347774` `; LoadLibrary OK, 跳, 花指令后到 12FF90`

0012FF90 `0FB64E FF` `MOVZX ECX,BYTE PTR DS:\[ESI-1\]` `; DLL 名字字符长度, 跳过`

0012FF94 `01CE` `ADD ESI,ECX`

0012FF96 `89F2` `MOV EDX,ESI`

0012FF99 `FFC2` `INC EDX` `; Null 结尾`

0012FF9B `8B0A` `MOV ECX,DWORD PTR DS:\[EDX\]` `; 800000C5`

0012FF9D `81E1 00000080` `AND ECX,80000000`

00347990 `8BF0` `MOV ESI,EAX` `; MSVBVM60`

00347992 `0BC9` `OR ECX,ECX`

00347994 `0F85 62070000` `JNZ 003480FC` `; 8X XX XX XX 是重定向的标志, 两种字符加密方式也不一样`

003480FC `8B0A` `MOV ECX,DWORD PTR DS:\[EDX\]`

003480FE `81E1 FFFFFFFF7F` `AND ECX,7FFFFFFF`

00348104 `51` `PUSH ECX` `; DLL 对应的 API 个数`

00348105 `52` `PUSH EDX`

00348255 `C1E1 05` `SHL ECX,5` `; 每个 API HOOK 32 字节空间`

00348258 `6A 04` `PUSH 4`

0034825A `68 00100000` `PUSH 1000`

0034825F `51` `PUSH ECX`

00348260 `6A 00` `PUSH 0`

00348262 `8D85 2D6D4000` `LEA EAX,DWORD PTR SS:\[EBP+406D2D\]` `; 返回地址 348275`

00348268 `50` `PUSH EAX`

00348269 `8B85 32F44000` `MOV EAX,DWORD PTR SS:\[EBP+40F432\]` `; KERNEL32.VirtualAlloc, 这一段空间是 IAT 的 HOOK`

0034826F `E9 48800000` `JMP 003502BC`

00348275 `8985 82F44000` `MOV DWORD PTR SS:\[EBP+40F482\],EAX` `; \[3509CA\]`

0034827B `5A` `POP EDX`

0034827C `59` `POP ECX`

003483CE `2BBD 36F44000` `SUB EDI,DWORD PTR SS:\[EBP+40F436\]` `; adalinks.00400000`

003483D4 `83FF FF` `CMP EDI,-1`

003483D7 `/74 15` `JE SHORT 003483EE`

003483D9 `|03BD 36F44000` `ADD EDI,DWORD PTR SS:\[EBP+40F436\]`

003483DF `|EB 09` `JMP SHORT 003483EA`

003483E1 `|8907` `MOV DWORD PTR DS:\[EDI\],EAX`

003483E3 `|83C0 20` `ADD EAX,20`

003483E6 `|83C7 04` `ADD EDI,4`

003483E9 `|49` `DEC ECX`

003483EA `|0BC9` `OR ECX,ECX`

003483EC `^|75 F3` `JNZ SHORT 003483E1`

003483EE `\\59` `POP ECX`

0034841D `8BF8` `MOV EDI,EAX` `; VirtualAlloc 分配的空间`

0034841F `57` `PUSH EDI`

00348420 `51` `PUSH ECX`

00348421 `E9 8B040000` `JMP 003488B1` `; 开始循环 为每个 API 搞一个 Hook, 如下`

`; PUSH DWORD PTR DS:\[11C001C\]`

`; XOR DWORD PTR SS:\[ESP\],C8021001`

`; RETN`

00348426 `8D47 1C` `LEA EAX,DWORD PTR DS:\[EDI+1C\]`

00348456 `66:C707 FF35` `MOV WORD PTR DS:\[EDI\],35FF`

003485AA `C747 06 8134240>` `MOV DWORD PTR DS:\[EDI+6\],243481`

00348700 `8947 02` `MOV DWORD PTR DS:\[EDI+2\],EAX`

00348730 `C647 0D C3` `MOV BYTE PTR DS:\[EDI+D\],0C3`

00348734 `52` `PUSH EDX`

00348735 `0F31` `RDTSC` `; 随机数`

00348737 `32E0` `XOR AH,AL`

00348739 `C1C8 08` `ROR EAX,8`

0034873C `02E0` `ADD AH,AL`

00348755 `C1C8 08` `ROR EAX,8`

00348758 `32E0` `XOR AH,AL`

003488A9 `8947 09` `MOV DWORD PTR DS:\[EDI+9\],EAX`

003488AC `5A` `POP EDX`

003488AD `83C7 20` `ADD EDI,20` `; 20h 字节`

003488B0 `49` `DEC ECX`

003488B1 `0BC9` `OR ECX,ECX`

003488B3 `^ 0F85 6DFBFFFF` `JNZ 00348426` `; 下一个`

003488B9 `59` `POP ECX`

003488BA `5F` `POP EDI`

003488BB `83C2 04` `ADD EDX,4`

003488BE `51` `PUSH ECX` `; 开始处理每一个 API`

003488BF `0FB602` `MOVZX EAX,BYTE PTR DS:\[EDX\]` `; 不等于 0 表示 API Name 长度, 到 34927E`

003488C2 `0BC0` `OR EAX,EAX` `; 等于 0, 表示后 4 byte 是函数序号, 到 3488CA`

003488C4 `0F85 B4090000` `JNZ 0034927E`

003488CA `42` `INC EDX` `; 函数序号的处理`

00348A1A `52` `PUSH EDX`

00348A1B `8B02` `MOV EAX,DWORD PTR DS:\[EDX\]` `; 函数序号`

00348A1D `50` `PUSH EAX`

00348A1E `56` `PUSH ESI` `; DLL MemoryBase`

00348A1F `8D85 EA744000` `LEA EAX,DWORD PTR SS:\[EBP+4074EA\]` `; 返回地址 348A32`

00348A25 `50` `PUSH EAX`

00348A26 `8B85 22F44000` `MOV EAX,DWORD PTR SS:\[EBP+40F422\]` `; 3507F1 GetProcAddress`

00348A2C `E9 8B780000` `JMP 003502BC`

`; 00348A32 8B 9D 22 FD 40 00 03 9D 26 FD 40 00 53 6A 00 50`

00348A32 `8B9D 22FD4000` `MOV EBX,DWORD PTR SS:\[EBP+40FD22\]` `; 930000 改成 JMP 348FD1`

00348A38 `039D 26FD4000` `ADD EBX,DWORD PTR SS:\[EBP+40FD26\]` `; 已用掉空间`

00348A3E `53` `PUSH EBX`

00348A3F `6A 00` `PUSH 0`

00348A41 `50` `PUSH EAX` `; API address`

00348B91 `53` `PUSH EBX` `; 壳的内存`

00348B92 `E8 8B780000` `CALL 00350422` `; 把 API 开头部分搬到自己的内存, 前面已经见过, F8`

00348B97 `2B85 22FD4000` `SUB EAX,DWORD PTR SS:\[EBP+40FD22\]`

00348CEC `8985 26FD4000` `MOV DWORD PTR SS:\[EBP+40FD26\],EAX`

00348CF2 `60` `PUSHAD` `; 空间还够不够`

00348CF3 `3D C01F0000` `CMP EAX,1FC0`

00348CF8 `0F86 80010000` `JBE 00348E7E`

00348CFE `6A 04` `PUSH 4`

00348D00 `68 00100000` `PUSH 1000`

00348D05 `68 00200000` `PUSH 2000`

00348D0A `6A 00` `PUSH 0`

00348D0C `8D85 D7774000` `LEA EAX,DWORD PTR SS:\[EBP+4077D7\]` `; 返回地址 348D1F`

00348D12 `50` `PUSH EAX`

00348D13 `8B85 32F44000` `MOV EAX,DWORD PTR SS:\[EBP+40F432\]` `; KERNEL32.VirtualAlloc`

00348D19 `E9 9E750000` `JMP 003502BC`

00348D1F `8985 22FD4000` `MOV DWORD PTR SS:\[EBP+40FD22\],EAX` `; 保存新空间`

00348E74 `C785 26FD4000 0>` `MOV DWORD PTR SS:\[EBP+40FD26\],0`

00348E7E `61` `POPAD`

00348E7F `5B` `POP EBX`

00348E80 `8BC3` `MOV EAX,EBX`

00348FD1 `3347 09` `XOR EAX,DWORD PTR DS:\[EDI+9\]`

00349123 `8947 1C` `MOV DWORD PTR DS:\[EDI+1C\],EAX`

00349126 `5A` `POP EDX` `; 00351885`

00349276 `83C2 04` `ADD EDX,4`

00349279 `/E9 17160000` `JMP 0034A895` `; 下一个 API`

0034927E `42` `INC EDX` `; 函数名字的处理`

0034927F `52` `PUSH EDX`

00349286 `8BF2` `MOV ESI,EDX`

0034928D `8DBD FCF94000` `LEA EDI,DWORD PTR SS:\[EBP+40F9FC\]` `; buffer 350F44`

003493E7 `33C0` `XOR EAX,EAX` `; 解密 API Name`

003493EE `0FB64E FF` `MOVZX ECX,BYTE PTR DS:\[ESI-1\]`

0012FF68 `50` `PUSH EAX`

0012FF69 `AC` `LODS BYTE PTR DS:\[ESI\]`

0012FF6A `34 79` `XOR AL,79`

0012FF6C `2C 55` `SUB AL,55`

0012FF6E `C0C0 03` `ROL AL,3`

0012FF71 `F6D0` `NOT AL`

0012FF73 `AA` `STOS BYTE PTR ES:\[EDI\]`

0012FF74 `31C0` `XOR EAX,EAX`



## 汇编代码段与 API 列表

```
0012FF76    49                  DEC ECX  
0012FF77    ^ 75 F0             JNZ SHORT 0012FF69  
0012FF79    AA                  STOS BYTE PTR ES:[EDI]  
0012FF7A    58                  POP EAX  
0012FF7B    C3                  RETN  

00349542    8D95 FCF94000       LEA EDX,DWORD PTR SS:[EBP+40F9FC]     ; 350F44 ( Name 解密完毕)  

; 需要比较是否下面所列的特殊 API  

00350D81                43 72 65 61 74 65 44 69 61 6C 6F 67      CreateDialog  
00350D91  50 61 72 61 6D 41 00 44 69 61 6C 6F 67 42 6F 78  ParamA.DialogBox  
00350DA1  50 61 72 61 6D 41 00 45 78 69 74 50 72 6F 63 65  ParamA.ExitProce  
00350DB1  73 73 00 46 72 65 65 52 65 73 6F 75 72 63 65 00  ss.FreeResource.  
00350DC1  47 65 74 50 72 6F 63 41 64 64 72 65 73 73 00 47  GetProcAddress.G  
00350DD1  65 74 56 65 72 73 69 6F 6E 00 47 65 74 4D 6F 64  etVersion.GetMod  
00350DE1  75 6C 65 48 61 6E 64 6C 65 41 00 47 65 74 43 75  uleHandleA.GetCu  
00350DF1  72 72 65 6E 74 50 72 6F 63 65 73 73 00 47 65 74  rrentProcess.Get  
00350E01  43 75 72 72 65 6E 74 50 72 6F 63 65 73 73 49 64  CurrentProcessId  
00350E11  00 47 65 74 43 6F 6D 6D 61 6E 64 4C 69 6E 65 41  .GetCommandLineA  
00350E21  00 4C 6F 61 64 4C 69 62 72 61 72 79 41 00 4C 6F  .LoadLibraryA.Lo  
00350E31  63 6B 52 65 73 6F 75 72 63 65 00 53 65 6E 64 4D  ckResource.SendM  
00350E41  65 73 73 61 67 65 41 00 73 65 6E 64 00 72 65 63  essageA.send.rec  
00350E51  76 00                                              v.  
```

```
00349698    52                  PUSH EDX  
00349699    8D85 DAF84000       LEA EAX,DWORD PTR SS:[EBP+40F8DA]     ; 350E22, "LoadLibraryA"  
0034969F    50                  PUSH EAX  
003496A0    8D85 BA824000       LEA EAX,DWORD PTR SS:[EBP+4082BA]     ; 返回地址 349802  
003497F5    50                  PUSH EAX  
003497F6    8B85 57F64000       MOV EAX,DWORD PTR SS:[EBP+40F657]     ; KERNEL32.lstrcmpA  
003497FC    E9 BB6A0000         JMP 003502BC  
00349802    5A                  POP EDX                               ; 00350F44  
00349803    85C0                TEST EAX,EAX  
00349805    75 0B               JNZ SHORT 00349812  
00349807    8D85 26E34000       LEA EAX,DWORD PTR SS:[EBP+40E326]  
0034980D    E9 D80D0000         JMP 0034A5EA                          ; 需要特殊处理的 API  
00349812    52                  PUSH EDX  
00349962    52                  PUSH EDX  
00349963    8D85 79F84000       LEA EAX,DWORD PTR SS:[EBP+40F879]     ; 350DC1 "GetProcAddress"  
00349969    50                  PUSH EAX  
0034996A    8D85 84854000       LEA EAX,DWORD PTR SS:[EBP+408584]     ; 返回地址 349ACC  
00349AC0    8B85 57F64000       MOV EAX,DWORD PTR SS:[EBP+40F657]     ; KERNEL32.lstrcmpA  
00349AC6    /E9 F1670000        JMP 003502BC  
00349ACC    5A                  POP EDX                               ; 00350F44  
00349ACD    85C0                TEST EAX,EAX  
00349ACF    75 0B               JNZ SHORT 00349ADC  
00349AD1    8D85 37E34000       LEA EAX,DWORD PTR SS:[EBP+40E337]  
00349AD7    E9 0E0B0000         JMP 0034A5EA                          ; 需要特殊处理的 API  
00349ADC    52                  PUSH EDX  
00349ADD    52                  PUSH EDX  
00349ADE    8D85 88F84000       LEA EAX,DWORD PTR SS:[EBP+40F888]     "GetVersion"  
00349AE4    50                  PUSH EAX  
00349AE5    8D85 B0854000       LEA EAX,DWORD PTR SS:[EBP+4085B0]     ; 返回地址 349AF8  
00349AEB    50                  PUSH EAX  
00349AEC    8B85 57F64000       MOV EAX,DWORD PTR SS:[EBP+40F657]     ; KERNEL32.lstrcmpA  
00349AF2    E9 C5670000         JMP 003502BC  
00349AF8    5A                  POP EDX                               ; 下面的比较没有花指令了  
00349AF9    85C0                TEST EAX,EAX  
00349AFB    75 0B               JNZ SHORT 00349B08  
00349AFD    8D85 4CE34000       LEA EAX,DWORD PTR SS:[EBP+40E34C]  
00349B03    E9 E20A0000         JMP 0034A5EA  
00349B08    52                  PUSH EDX  
00349B09    52                  PUSH EDX  
00349B0A    8D85 93F84000       LEA EAX,DWORD PTR SS:[EBP+40F893]  
00349B10    50                  PUSH EAX  
00349B11    8D85 DC854000       LEA EAX,DWORD PTR SS:[EBP+4085DC]  
00349B17    50                  PUSH EAX  
00349B18    8B85 57F64000       MOV EAX,DWORD PTR SS:[EBP+40F657]  
00349B1E    E9 99670000         JMP 003502BC  
....... 15 次  
0034A2B7    5A                  POP EDX  
0034A2B8    85C0                TEST EAX,EAX  
0034A2BA    75 0B               JNZ SHORT 0034A2C7  
0034A2BC    8D85 E8E44000       LEA EAX,DWORD PTR SS:[EBP+40E4E8]  
0034A2C2    E9 23030000         JMP 0034A5EA                          ; 需要特殊处理的 API  
0034A2C7    52                  PUSH EDX                              ; 比较结束, 都不是到这里  
0034A2C8    56                  PUSH ESI  
0034A2C9    8D85 948D4000       LEA EAX,DWORD PTR SS:[EBP+408D94]     ; 返回地址 34A2DC  
0034A2CF    50                  PUSH EAX  
0034A2D0    8B85 22F44000       MOV EAX,DWORD PTR SS:[EBP+40F422]     ; 3507F1 , GetProcAddress  
0034A2D6    E9 E15F0000         JMP 003502BC  
                      ; 0034A2DC 8B 9D 22 FD 40 00 03 9D 26 FD 40 00 53 6A 00 50  
0034A2DC    8B9D 22FD4000       MOV EBX,DWORD PTR SS:[EBP+40FD22]     ; 930000 改成 JMP 34A5EA ******
0034A2E2    039D 26FD4000       ADD EBX,DWORD PTR SS:[EBP+40FD26]     ; 已用掉空间  
0034A2E8    53                  PUSH EBX  
0034A2E9    6A 00               PUSH 0  
0034A2EB    50                  PUSH EAX                              ; API address  
0034A2EC    53                  PUSH EBX                              ; 自己的内存  
0034A2ED    E8 30610000         CALL 00350422                         ; 把 API 开头部分搬到自己的内存, 前面已经见过, F8
0034A2F2    2B85 22FD4000       SUB EAX,DWORD PTR SS:[EBP+40FD22]     ; 这个 API 占用的空间 (包括花指令)  
0034A2F8    8985 26FD4000       MOV DWORD PTR SS:[EBP+40FD26],EAX  
0034A2FE    60                  PUSHAD  
0034A2FF    3D C01F0000         CMP EAX,1FC0  
0034A304    0F86 8D010000       JBE 0034A497                          ; 空间还够不够  
0034A30A    6A 04               PUSH 4  
0034A30C    68 00100000         PUSH 1000  
0034A311    68 00200000         PUSH 2000  
0034A316    6A 00               PUSH 0  
0034A318    8D85 3F8F4000       LEA EAX,DWORD PTR SS:[EBP+408F3F]     ; 返回地址 34A33B  
0034A31E    50                  PUSH EAX  
0034A31F    8B85 32F44000       MOV EAX,DWORD PTR SS:[EBP+40F432]     ; VirtualAlloc  
0034A325    E9 925F0000         JMP 003502BC  
0034A33B    8985 22FD4000       MOV DWORD PTR SS:[EBP+40FD22],EAX     ; 保存新空间  
0034A341    C785 26FD4000 0>MOV DWORD PTR SS:[EBP+40FD26],0
0034A497    61                  POPAD  
0034A5E7    5B                  POP EBX                               ; 00930000  
0034A5E8    8BC3                MOV EAX,EBX  
0034A5EA    3347 09             XOR EAX,DWORD PTR DS:[EDI+9]  
0034A73C    8947 1C             MOV DWORD PTR DS:[EDI+1C],EAX  
0034A73F    5A                  POP EDX                               ; 003515EE  
0034A88F    0FB642 FF           MOVZX EAX,BYTE PTR DS:[EDX-1]         ; API name 长度  
0034A893    03D0                ADD EDX,EAX                           ; 下一个 API  
0034A895    42                  INC EDX                               ; 不管是 API 名字还是 API 序号, 后面都带 NULL  
0034A896    83C7 20             ADD EDI,20  
0034A9E8    59                  POP ECX  
0034A9E9    49                  DEC ECX  
0034A9EA    ^ 0F85 CEDEFFFF     JNZ 003488BE                          ; 下一个 API  
0034A9F0    ^\E9 3ACAFFFF       JMP 0034742F                          ; 下一个 DLL  
DLL 结束, 还原两处修改, VB 程序只有一个 DLL, 也没有特殊函数, 简单一点  
;========================================================================================================================================================================  
```

## 八. `ZwQueryInformationProcess` 检查调试器, 文件 `CRC` 校验 等

```
0034A9F5    E8 0A000000         CALL 0034AA04                         ; IAT 处理完了  
0034AA04    8D85 1E964000       LEA EAX,DWORD PTR SS:[EBP+40961E]      ; 返回地址 34AB66  
0034AA0A    50                  PUSH EAX  
0034AB5A    8B85 2AF44000       MOV EAX,DWORD PTR SS:[EBP+40F42A]       ; KERNEL32.GetModuleHandleA  
```



```
0034AB60    E9 57570000     JMP 003502BC  

0034AB66    0BC0            OR EAX,EAX                        ; ntdll.77F80000  

0034AB68    0F84 A3090000   JE 0034B511  

0034AB8D    50              PUSH EAX  

0034AB8E    8D85 A8974000   LEA EAX,DWORD PTR SS:[EBP+4097A8]        ; 返回地址 34ACF0  

0034ACE3    50              PUSH EAX  

0034ACE4    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]        ; 3507F1 GetProcAddress  

0034ACEA    E9 CD550000     JMP 003502BC  

0034ACF0    0BC0            OR EAX,EAX                        ; ntdll.ZwQueryInformationProcess  

0034ACF2    0F84 19080000   JE 0034B511  

0034ACF8    8BF0            MOV ESI,EAX  

0034ACFA    8D85 14994000   LEA EAX,DWORD PTR SS:[EBP+409914]        ; 返回地址 34AE5C  

0034AD00    50              PUSH EAX  

0034AE50    8B85 DDF54000   MOV EAX,DWORD PTR SS:[EBP+40F5DD]        ; KERNEL32.GetCurrentProcess  

0034AE56    E9 61540000     JMP 003502BC  

0034AE5C    8BF8            MOV EDI,EAX  

0034AE5E    50              PUSH EAX  

0034AE5F    8BC4            MOV EAX,ESP                     ; 12FFA0 存放结果  

0034AFB0    6A 00           PUSH 0  

0034AFB2    6A 04           PUSH 4  

0034B103    50              PUSH EAX  

0034B104    6A 07           PUSH 7  

0034B255    57              PUSH EDI  

0034B256    FFD6            CALL ESI                        ; ntdll.ZwQueryInformationProcess  

0034B3A7    58              POP EAX  

0034B3A8    0BC0            OR EAX,EAX  

0034B3AA    0F84 61010000   JE 0034B511                     ; 必须跳 ************************************************************  

0034B511    B9 00010000     MOV ECX,100  

0034B516    2BE1            SUB ESP,ECX                     ; 使用 Stack 空间放文件名  

0034B667    8BF4            MOV ESI,ESP  

0034B669    8BFC            MOV EDI,ESP  

0034B7BA    C1E9 02         SHR ECX,2  

0034B7BD    33C0            XOR EAX,EAX  

0034B90E    F3:AB           REP STOS DWORD PTR ES:[EDI]  

0034B910    68 00010000     PUSH 100  

0034BA64    56              PUSH ESI  

0034BA65    8B85 36F44000   MOV EAX,DWORD PTR SS:[EBP+40F436]        ; adalinks.00400000  

0034BBBA    50              PUSH EAX                        ; adalinks.00400000  

0034BBBB    8D85 D5A74000   LEA EAX,DWORD PTR SS:[EBP+40A7D5]        ; 返回地址 34BD1D  

0034BD10    50              PUSH EAX  

0034BD11    8B85 29F64000   MOV EAX,DWORD PTR SS:[EBP+40F629]        ; KERNEL32.GetModuleFileNameA  

0034BD17   /E9 A0450000     JMP 003502BC  

0034BD1D    6A 00           PUSH 0  

0034BD1F    68 80000000     PUSH 80  

0034BD24    6A 03           PUSH 3  

0034BD26    6A 00           PUSH 0  

0034BD28    6A 03           PUSH 3  

0034BD2A    68 00000080     PUSH 80000000  

0034BD2F    56              PUSH ESI                        ; "adalinks.exe"  

0034BD30    8D85 03A84000   LEA EAX,DWORD PTR SS:[EBP+40A803]       ; 返回地址 34BD4D  

0034BD36    50              PUSH EAX  

0034BD37    8B85 56F54000   MOV EAX,DWORD PTR SS:[EBP+40F556]       ; KERNEL32.CreateFileA  

0034BD3D    E9 7A450000     JMP 003502BC  

0034BD4B    8BD8            MOV EBX,EAX                   ; hFile  

0034BD4D    81C4 00010000   ADD ESP,100                

0034BD53    6A 00           PUSH 0  

0034BD55    53              PUSH EBX  

0034BD56    8D85 21A84000   LEA EAX,DWORD PTR SS:[EBP+40A821]       ; 返回地址 34BD69   

0034BD5C    50              PUSH EAX  

0034BD5D    8B85 D0F54000   MOV EAX,DWORD PTR SS:[EBP+40F5D0]        ; KERNEL32.GetFileSize  

0034BD63    E9 54450000     JMP 003502BC  

0034BD69    8985 3AF44000   MOV DWORD PTR SS:[EBP+40F43A],EAX        ;FileSize  

0034BD6F    6A 00           PUSH 0  

0034BD71    FFB5 3AF44000   PUSH DWORD PTR SS:[EBP+40F43A]  

0034BD77    6A 00           PUSH 0  

0034BD79    6A 02           PUSH 2  

0034BD7B    6A 00           PUSH 0  

0034BD7D    53              PUSH EBX  

0034BD7E    8D85 49A84000   LEA EAX,DWORD PTR SS:[EBP+40A849]       ; 返回地址 34BD91  

0034BD84    50              PUSH EAX  

0034BD85    8B85 63F54000   MOV EAX,DWORD PTR SS:[EBP+40F563]        ; KERNEL32.CreateFileMappingA  

0034BD8B    E9 2C450000     JMP 003502BC  

0034BD91    8985 3EF44000   MOV DWORD PTR SS:[EBP+40F43E],EAX        ; hFileMap  

0034BD97    6A 00           PUSH 0  

0034BD99    6A 00           PUSH 0  

0034BD9B    6A 00           PUSH 0  

0034BD9D    6A 04           PUSH 4  

0034BD9F    FFB5 3EF44000   PUSH DWORD PTR SS:[EBP+40F43E]          

0034BDA5    8D85 77A84000   LEA EAX,DWORD PTR SS:[EBP+40A877]        ; 返回地址 34BDBF  

0034BDAB    50              PUSH EAX  

0034BDAC    8B85 6FF64000   MOV EAX,DWORD PTR SS:[EBP+40F66F]        ; KERNEL32.MapViewOfFile  

0034BDB2    E9 05450000     JMP 003502BC  

0034BDBF    8985 42F44000   MOV DWORD PTR SS:[EBP+40F442],EAX        ; hFileMem  

0034BDC5    53              PUSH EBX                        ; hFile  

0034BDC6    8B40 3C         MOV EAX,DWORD PTR DS:[EAX+3C]            ; Dos 头  

0034BDC9    8B8D 3AF44000   MOV ECX,DWORD PTR SS:[EBP+40F43A]        ; FileSize  

0034BDCF    2BC8            SUB ECX,EAX                     ; 大小不包括  Dos 头  

0034BDD1    8BB5 42F44000   MOV ESI,DWORD PTR SS:[EBP+40F442]        ; hFileMem  

0034BDD7    03F0            ADD ESI,EAX                     ; 从 PE 头开始  

0034BDD9    E8 19350000     CALL 0034F2F7                     ; F8, CRC 校验 = 014D2896  

0034BDDE    5B              POP EBX  

0034BDDF    3385 4AF44000   XOR EAX,DWORD PTR SS:[EBP+40F44A]        ; [350992]  

0034BDE5    C1C8 03         ROR EAX,3  

0034BDE8    8BF0            MOV ESI,EAX                     ; 2022EBCC  

0034BDEA    8B85 42F44000   MOV EAX,DWORD PTR SS:[EBP+40F442]        ; hFileMem  

0034BF3F    0340 3C         ADD EAX,DWORD PTR DS:[EAX+3C]  

0034BF42    8B78 FC         MOV EDI,DWORD PTR DS:[EAX-4]         ; 2022EBCC 校验和就放在 DOS 头最后 4 字节, ha  

0034BF45    FFB5 42F44000   PUSH DWORD PTR SS:[EBP+40F442]           ; hFileMem  

0034BF4B    8D85 16AA4000   LEA EAX,DWORD PTR SS:[EBP+40AA16]        ; 返回地址 34BF5E  

0034BF51    50              PUSH EAX  

0034BF52    8B85 D9F64000   MOV EAX,DWORD PTR SS:[EBP+40F6D9]        ; KERNEL32.UnmapViewOfFile  

0034BF58    E9 5F430000     JMP 003502BC  

0034BF5E    FFB5 3EF44000   PUSH DWORD PTR SS:[EBP+40F43E]           ; hFileMap  

0034BF64    8D85 2FAA4000   LEA EAX,DWORD PTR SS:[EBP+40AA2F]        ; 返回地址 34BF77  

0034BF6A    50              PUSH EAX  

0034BF6B    8B85 49F54000   MOV EAX,DWORD PTR SS:[EBP+40F549]        ; KERNEL32.CloseHandle  

0034BF71    E9 46430000     JMP 003502BC  

0034BF77    53              PUSH EBX                        ; hFile  

0034BF78    8D85 43AA4000   LEA EAX,DWORD PTR SS:[EBP+40AA43]        ; 返回地址 34BF8B     

0034BF7E    50              PUSH EAX  

0034BF7F    8B85 49F54000   MOV EAX,DWORD PTR SS:[EBP+40F549]        ; KERNEL32.CloseHandle  

0034BF85    E9 32430000     JMP 003502BC  

0034BF8B    8B85 6EF44000   MOV EAX,DWORD PTR SS:[EBP+40F46E]        ; [3509B6]=1 校验文件的选项  

0034C0E0    83F8 01         CMP EAX,1  

0034C0E3    75 08           JNZ SHORT 0034C0ED  

0034C0E5    3BF7            CMP ESI,EDI  

0034C0E7    0F85 BB540000   JNZ 003515A8                     ; 不能跳 ************************************************************  

;=========================================================================================================================================================================  

九. 完成 IAT 的处理 , 和 15 个特殊 API 搬到壳里  

;代码段里 CALL IAT, JMP IAT 　等语句中 跳转距离需要修改  

0034C0ED    8BB5 12FC4000   MOV ESI,DWORD PTR SS:[EBP+40FC12]        ; [35115A] = 1000  
```



0034C0F3 03B5 36F44000 ADD ESI,DWORD PTR SS:\[EBP+40F436\] ; adalinks.00400000

0034C248 8B8D 16FC4000 MOV ECX,DWORD PTR SS:\[EBP+40FC16\] ; \[3511AE\] = 81000

0034C39D 83E9 05 SUB ECX,5

0034C3A0 E9 A8050000 JMP 0034C94D

0034C3A5 66:8B06 MOV AX,WORD PTR DS:\[ESI\]

0034C3A8 3C E8 CMP AL,0E8 ; Call XXXXX

0034C3AA 0F85 BA020000 JNZ 0034C66A

0034C66A 3C E9 CMP AL,0E9 ; JMP XXXXX

0034C66C 0F85 B7020000 JNZ 0034C929

0034C929 3C 0F CMP AL,0F

0034C92B 75 1E JNZ SHORT 0034C94B

0034C92D 80FC 7F CMP AH,7F

0034C930 76 19 JBE SHORT 0034C94B

0034C932 80FC 90 CMP AH,90

0034C935 73 14 JNB SHORT 0034C94B

0034C937 8BC6 MOV EAX,ESI ; 条件转移

0034C939 2B85 36F44000 SUB EAX,DWORD PTR SS:\[EBP+40F436\]

0034C93F 83C0 06 ADD EAX,6

0034C942 2946 02 SUB DWORD PTR DS:\[ESI+2\],EAX

0034C945 83C6 05 ADD ESI,5

0034C948 83E9 05 SUB ECX,5

0034C94B 46 INC ESI

0034C94D 81F9 00000080 CMP ECX,80000000

0034C953 0F82 4CFAFFFF JB 0034C3A5

; 15 个特殊 API 全部搬到壳里

0034C959 8D85 7CB54000 LEA EAX,DWORD PTR SS:\[EBP+40B57C\] ; 返回地址 34CAC4

0034C95F 50 PUSH EAX

0034CAAF 8B85 4BF64000 MOV EAX,DWORD PTR SS:\[EBP+40F64B\] ; KERNEL32.GetVersion

0034CAB5 E9 02380000 JMP 003502BC

0034CAC4 8985 0BF94000 MOV DWORD PTR SS:\[EBP+40F90B\],EAX

0034CACA 8D85 E4B64000 LEA EAX,DWORD PTR SS:\[EBP+40B6E4\] ; 返回地址 34CC2C

0034CAD0 50 PUSH EAX

0034CC20 8B85 DDF54000 MOV EAX,DWORD PTR SS:\[EBP+40F5DD\] ; KERNEL32.GetCurrentProcess

0034CC26 E9 91360000 JMP 003502BC

0034CC2C 8985 13F94000 MOV DWORD PTR SS:\[EBP+40F913\],EAX

0034CC32 8D85 FDB64000 LEA EAX,DWORD PTR SS:\[EBP+40B6FD\] ; 返回地址 34CC45

0034CC38 50 PUSH EAX

0034CC39 8B85 F0F54000 MOV EAX,DWORD PTR SS:\[EBP+40F5F0\] ; KERNEL32.GetCurrentProcessId

0034CC3F E9 78360000 JMP 003502BC

0034CC45 8985 17F94000 MOV DWORD PTR SS:\[EBP+40F917\],EAX

0034CC4B 8D85 70B84000 LEA EAX,DWORD PTR SS:\[EBP+40B870\] ; 返回地址 34CDB8

0034CC51 50 PUSH EAX

0034CDA1 8B85 BFF54000 MOV EAX,DWORD PTR SS:\[EBP+40F5BF\] ; KERNEL32.GetCommandLineA

0034CDA7 E9 10350000 JMP 003502BC

0034CDB8 8985 1BF94000 MOV DWORD PTR SS:\[EBP+40F91B\],EAX

0034CDBE 6A 00 PUSH 0

0034CDC0 8D85 DAB94000 LEA EAX,DWORD PTR SS:\[EBP+40B9DA\] ; 返回地址 34CF22

0034CDC6 50 PUSH EAX

0034CF16 8B85 17F64000 MOV EAX,DWORD PTR SS:\[EBP+40F617\] ; KERNEL32.GetModuleHandleA

0034CF1C E9 9B330000 JMP 003502BC

0034CF22 8985 0FF94000 MOV DWORD PTR SS:\[EBP+40F90F\],EAX ; adalinks.00400000

0034CF28 6A 00 PUSH 0

0034CF2A FFB5 56F74000 PUSH DWORD PTR SS:\[EBP+40F756\] ; WS2\_32.WSASend 地址

0034CF30 8D85 1AFC4000 LEA EAX,DWORD PTR SS:\[EBP+40FC1A\] ; 351162

0034CF36 50 PUSH EAX

0034D086 E8 97330000 CALL 00350422 ; F8, 把 这个 API 搬到 351162

0034D08B 6A 00 PUSH 0

0034D08D FFB5 5FF74000 PUSH DWORD PTR SS:\[EBP+40F75F\] ; WS2\_32.WSARecv

0034D093 8D85 5AFC4000 LEA EAX,DWORD PTR SS:\[EBP+40FC5A\]

0034D099 50 PUSH EAX ; 3511A2

0034D09A E8 83330000 CALL 00350422 ; F8, 把 这个 API 搬到 3511A2

; ShellTmpMap ???

0034D09F 8D85 34FA4000 LEA EAX,DWORD PTR SS:\[EBP+40FA34\] ; 350F7C "ShellTmpMap"

0034D0A5 50 PUSH EAX

0034D0A6 68 00010000 PUSH 100

0034D0AB 6A 00 PUSH 0

0034D0AD 6A 04 PUSH 4

0034D0AF 6A 00 PUSH 0

0034D0B1 6A FF PUSH -1

0034D0B3 8D85 7EBB4000 LEA EAX,DWORD PTR SS:\[EBP+40BB7E\] ; 返回地址 34D0C6

0034D0B9 50 PUSH EAX

0034D0BA 8B85 63F54000 MOV EAX,DWORD PTR SS:\[EBP+40F563\] ; KERNEL32.CreateFileMappingA

0034D0C0 E9 F7310000 JMP 003502BC

0034D0C6 83F8 00 CMP EAX,0

0034D0C9 0F84 D9440000 JE 003515A8

0034D0CF 8985 40FA4000 MOV DWORD PTR SS:\[EBP+40FA40\],EAX ; hFileMap

0034D0D5 68 00010000 PUSH 100

0034D0DA 6A 00 PUSH 0

0034D0DC 6A 00 PUSH 0

0034D0DE 6A 06 PUSH 6

0034D0E0 50 PUSH EAX

0034D0E1 8D85 ACBB4000 LEA EAX,DWORD PTR SS:\[EBP+40BBAC\] ; 返回地址 34D0F4

0034D0E7 50 PUSH EAX

0034D0E8 8B85 6FF64000 MOV EAX,DWORD PTR SS:\[EBP+40F66F\] ; KERNEL32.MapViewOfFile

0034D0EE E9 C9310000 JMP 003502BC

0034D0F4 8985 44FA4000 MOV DWORD PTR SS:\[EBP+40FA44\],EAX ; hFileMem

0034D0FA 8BF8 MOV EDI,EAX

0034D0FC 8DB5 48FA4000 LEA ESI,DWORD PTR SS:\[EBP+40FA48\] ; 350F90 "ShellMap"

0034D102 B9 0A000000 MOV ECX,0A

0034D107 F3:A4 REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]

0034D109 8B85 0BF94000 MOV EAX,DWORD PTR SS:\[EBP+40F90B\] ; OSVersion

0034D10F 3D 00000080 CMP EAX,80000000

0034D114 73 19 JNB SHORT 0034D12F

0034D116 64:FF35 3000000 PUSH DWORD PTR FS:\[30\]

0034D11D 58 POP EAX

0034D11E 0FB658 02 MOVZX EBX,BYTE PTR DS:\[EAX+2\] ; IsDebugPresent

0034D122 0ADB OR BL,BL

0034D124 0F85 7E440000 JNZ 003515A8 ; 不能跳 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

0034D12A E9 77010000 JMP 0034D2A6

0034D2A6 8BB5 5EF44000 MOV ESI,DWORD PTR SS:\[EBP+40F45E\] ; \[3509A6\] 这是什么加密选项?

0034D2AC 0BF6 OR ESI,ESI

0034D2AE 0F84 EA020000 JE 0034D59E

0034D59E 8CC9 MOV CX,CS ; 跟 OSVersion 有关的东东?

0034D5A0 32C9 XOR CL,CL

0034D5A2 0BC9 OR ECX,ECX

0034D5A4 0F84 7F010000 JE 0034D729 ; 2K 下跳了

0034D729 E8 0E000000 CALL 0034D73C ; 一个简单的 SEH, 检查 SoftICE?

0034D72E 8B4C24 0C MOV ECX,DWORD PTR SS:\[ESP+C\]

0034D732 8381 B800000 0 ADD DWORD PTR DS:\[ECX+B8\],2 ; EIP+2

0034D739 33C0 XOR EAX,EAX

0034D73B C3 RETN

0034D73C 64:FF35 0000000 PUSH DWORD PTR FS:\[0\]

0034D743 64:8925 0000000 MOV DWORD PTR FS:\[0\],ESP

0034D74A 33C0 XOR EAX,EAX

0034D74C CD 01 INT 1 ; 内存访问异常

0034D74E 40 INC EAX

0034D74F 40 INC EAX

0034D750 0BC0 OR EAX,EAX

0034D752 64:8F05 0000000 POP DWORD PTR FS:\[0\]

0034D759 58 POP EAX

0034D75A 0F84 483E0000 JE 003515A8 ; 不能跳 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

; 修正 JMP IAT 到 HOOK table

0034D760 8BB5 7EF44000 MOV ESI,DWORD PTR SS:\[EBP+40F47E\] ; \[3509C6\]=2D50

0034D766 0BF6 OR ESI,ESI

0034D768 0F84 7B010000 JE 0034D8E9

0034D76E 03B5 36F44000 ADD ESI,DWORD PTR SS:\[EBP+40F436\] ; adalinks.00400000

0034D774 E9 65010000 JMP 0034D8DE



### 十. AntiDump

```
0034D8E9    8B85 92F44000   MOV EAX,DWORD PTR SS:[EBP+40F492]       ; [3509DA]  这是什么加密选项?  
0034D8EF    0BC0            OR EAX,EAX  
0034D8F1    0F84 2A040000   JE 0034DD21  
0034DD21    6A 04           PUSH 4  
0034DD23    68 00100000     PUSH 1000  
0034DD28    68 00100000     PUSH 1000  
0034DD2D    6A 00           PUSH 0  
0034DD2F    8D85 FAC74000   LEA EAX,DWORD PTR SS:[EBP+40C7FA]      ; 返回地址 34DD42     
0034DD35    50             PUSH EAX  
0034DD36    8B85 32F44000   MOV EAX,DWORD PTR SS:[EBP+40F432]      ; KERNEL32.VirtualAlloc  
0034DD3C    E9 7B250000     JMP 003502BC  
0034DD42    8985 AAF44000   MOV DWORD PTR SS:[EBP+40F4AA],EAX  
0034DD48    8185 AAF44000 0>ADD DWORD PTR SS:[EBP+40F4AA],1000  
0034DD52    64:FF35 3000000>PUSH DWORD PTR FS:[30]  
0034DD59    58             POP EAX  
0034DEA9    85C0          TEST EAX,EAX 不能                      ; AntiDump  
0034DEAB    78 12         JS SHORT 0034DEBF  
0034DEAD    8B40 0C       MOV EAX,DWORD PTR DS:[EAX+C]  
0034DEB0    8B40 0C       MOV EAX,DWORD PTR DS:[EAX+C]  
0034DEB3    C740 20 0010000>MOV DWORD PTR DS:[EAX+20],1000        ; PEB 里 VirtualSize,  不允许改, 跳过 ************  
0034DEBA    E9 90010000   JMP 0034E04F  
0034E04F    50            PUSH EAX                     ; 改文件头的属性  
0034E050    8BC4          MOV EAX,ESP  
0034E052    50            PUSH EAX  
0034E053    6A 04         PUSH 4  
0034E055    68 00100000   PUSH 1000  
0034E05A    FFB5 36F44000  PUSH DWORD PTR SS:[EBP+40F436]         ; adalinks.00400000  
0034E060    8D85 2BCB4000  LEA EAX,DWORD PTR SS:[EBP+40CB2B]     ; 返回地址 34E073  
0034E066    50            PUSH EAX  
0034E067    8B85 07F74000  MOV EAX,DWORD PTR SS:[EBP+40F707]     ; KERNEL32.VirtualProtect  
0034E06D    E9 4A220000   JMP 003502BC  
0034E073    83C4 04       ADD ESP,4  
0034E076    0BC0          OR EAX,EAX  
0034E078    0F84 5E010000  JE 0034E1DC                   ; 这里直接跳, 跳过下面的修改  ************  
0034E07E    8B95 36F44000  MOV EDX,DWORD PTR SS:[EBP+40F436]     ; adalinks.00400000  
0034E084    0352 3C      ADD EDX,DWORD PTR DS:[EDX+3C]  
0034E087    8B42 30      MOV EAX,DWORD PTR DS:[EDX+30]     ; BaseofData  
0034E1D9    8942 2C      MOV DWORD PTR DS:[EDX+2C],EAX     ; BaseofCode 被修改了, 不允许**********  
0034E1DC    8DB5 8AF94000  LEA ESI,DWORD PTR SS:[EBP+40F98A]     ; 350ED2  
0034E1E2    8BFE         MOV EDI,ESI               ; 350ED2  
0034E1E4    B9 4F000000  MOV ECX,4F  
0034E1E9    EB 05        JMP SHORT 0034E1F0  
0034E1EB    AC           LODS BYTE PTR DS:[ESI]  
0034E1EC    2C 80       SUB AL,80  
0034E1EE    AA           STOS BYTE PTR ES:[EDI]  
0034E1EF    49          DEC ECX  
0034E1F0    0BC9        OR ECX,ECX  
0034E1F2   ^ 75 F7       JNZ SHORT 0034E1EB           ; 解密出 "I am ...."  
0034E1F4    8DB5 8AF94000  LEA ESI,DWORD PTR SS:[EBP+40F98A]  
0034E1FA    8BFE         MOV EDI,ESI  
0034E1FC    B9 4F000000  MOV ECX,4F  
0034E201    EB 05        JMP SHORT 0034E208  
0034E203    AC           LODS BYTE PTR DS:[ESI]  
0034E204    04 80       ADD AL,80  
0034E206    AA           STOS BYTE PTR ES:[EDI]  
0034E207    49          DEC ECX  
0034E208    0BC9        OR ECX,ECX  
0034E20A   ^ 75 F7       JNZ SHORT 0034E203           ; 怎么又加密回去了?  
0034E20C    8B85 76F44000  MOV EAX,DWORD PTR SS:[EBP+40F476]     ; [3509BE] ?  
0034E212    83F8 01     CMP EAX,1  
0034E215    75 19       JNZ SHORT 0034E230  
0034E230    83BD 96F44000 0>CMP DWORD PTR SS:[EBP+40F496],0       ; [3509DE]=0EFAB  
0034E237    74 2F       JE SHORT 0034E268  
0034E239    8B8D 36F44000  MOV ECX,DWORD PTR SS:[EBP+40F436]     ; adalinks.00400000  
0034E23F    2B8D 5AF44000  SUB ECX,DWORD PTR SS:[EBP+40F45A]     ; [3509A2] = 40000000  
0034E245    8DBD C91A4000  LEA EDI,DWORD PTR SS:[EBP+401AC9]  
0034E24B    03BD 96F44000  ADD EDI,DWORD PTR SS:[EBP+40F496]     ; 351FBC  
0034E251    8DB5 C91A4000  LEA ESI,DWORD PTR SS:[EBP+401AC9]  
0034E257    03B5 9AF44000  ADD ESI,DWORD PTR SS:[EBP+40F49A]     ; 351FE5  
0034E25D    AD          LODS DWORD PTR DS:[ESI]  
0034E25E    EB 04       JMP SHORT 0034E264  
0034E260    010C38      ADD DWORD PTR DS:[EAX+EDI],ECX  
0034E263    AD          LODS DWORD PTR DS:[ESI]  
0034E264    0BC0        OR EAX,EAX  
0034E266   ^ 75 F8       JNZ SHORT 0034E260           ; 什么东西需要重定位?  
```

### 十一. 校验内存, Drx, 8 个连环 SEH

```
0034E268    8B85 96F44000  MOV EAX,DWORD PTR SS:[EBP+40F496]      ; 0EFAB  
0034E3BD    0385 36F44000  ADD EAX,DWORD PTR SS:[EBP+40F436]      ; adalinks.00400000  
0034E512    894424 EC      MOV DWORD PTR SS:[ESP-14],EAX          ; adalinks.0040EFAB  
0034E543    896C24 E8      MOV DWORD PTR SS:[ESP-18],EBP  
0034E574    C785 7AF44000 0>MOV DWORD PTR SS:[EBP+40F47A],0       ; [3509c2], 要校验壳内存代码  
0034E595    33C0          XOR EAX,EAX                
0034E5C4    8DB5 C91A4000  LEA ESI,DWORD PTR SS:[EBP+401AC9]     ; 343011  
0034E5E1    B9 59D90000   MOV ECX,0D959                 ; size  
0034E613    C1E9 02      SHR ECX,2  
0034E616    EB 08        JMP SHORT 0034E620  
0034E618    AD          LODS DWORD PTR DS:[ESI]  
0034E619    3185 7AF44000  XOR DWORD PTR SS:[EBP+40F47A],EAX     ; [3509C2]  
0034E61F    49          DEC ECX  
0034E620    0BC9        OR ECX,ECX  
0034E622   ^ 75 F4       JNZ SHORT 0034E618  
```

F4 到 0034E624, 结果是

```
003509C2  36 40 A6 B8  
```

```
0034E624    8B4424 EC      MOV EAX,DWORD PTR SS:[ESP-14]          ; adalinks.0040EFAB  
0034E628    2B85 36F44000  SUB EAX,DWORD PTR SS:[EBP+40F436]      ; adalinks.00400000  
0034E62E    8985 96F44000  MOV DWORD PTR SS:[EBP+40F496],EAX     ; [3509DE]  
0034E634    8B6C24 E8      MOV EBP,DWORD PTR SS:[ESP-18]  
0034E638    8B85 52F44000  MOV EAX,DWORD PTR SS:[EBP+40F452]     ; [35099A]  
0034E63E    0BC0          OR EAX,EAX  
0034E640    74 0F        JE SHORT 0034E651  
0034E651    8B85 7AF44000  MOV EAX,DWORD PTR SS:[EBP+40F47A]    ; [3509C2]= B8A64036 内存校验和  
0034E657    E8 48000000   CALL 0034E6A4              ; 异常校验 Drx  

;SEH  
0012FCD4   0FFF4A3D                    ; 进入异常处理代码后, 先把 Drx 按第三部分设好************  
0012FCD8   0FFF23B9  
0012FCDC   0FFF9002  
0012FCE0   0FFFCDEF  
0012FCE4   0  
0012FCE8   155  
0034E65C    8B4C24 0C      MOV ECX,DWORD PTR SS:[ESP+C]          ; pContext  
```



## 汇编代码段

```
0034E660    FF81 B8000000   INC DWORD PTR DS:[ECX+B8]               ; regEip+1  

0034E666    33C0            XOR EAX,EAX  

0034E668    3341 04         XOR EAX,DWORD PTR DS:[ECX+4]            ; dr0  

0034E66B    0341 08         ADD EAX,DWORD PTR DS:[ECX+8]            ; dr1  

0034E66E    3341 0C         XOR EAX,DWORD PTR DS:[ECX+C]            ; dr2  

0034E671    0341 10         ADD EAX,DWORD PTR DS:[ECX+10]          ; dr3  

0034E674    0181 B0000000   ADD DWORD PTR DS:[ECX+B0],EAX          ; regEAX = D8A80C19  

0034E67A    60              PUSHAD  

0034E67B    8D71 04         LEA ESI,DWORD PTR DS:[ECX+4]            ; ESI->Dr0  

0034E67E    8BA9 B4000000   MOV EBP,DWORD PTR DS:[ECX+B4]          ; regEBP  

0034E684    8DBD 3AFD4000   LEA EDI,DWORD PTR SS:[EBP+40FD3A]      ; 351282  

0034E68A    81C7 08010000   ADD EDI,108                             ; 35138A  

0034E690    B9 06000000     MOV ECX,6  

0034E695    83BD 46F44000 0>CMP DWORD PTR SS:[EBP+40F446],0        ; [35098E]  

0034E69C    75 02           JNZ SHORT 0034E6A0  

0034E69E    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI] ; 把 Dr0-Dr7 都复制出来  

0034E6A0    61              POPAD  

0034E6A1    33C0            XOR EAX,EAX  

0034E6A3    C3              RETN  

0034E6A4    64:FF35 0000000>PUSH DWORD PTR FS:[0]  

0034E6AB    64:8925 0000000>MOV DWORD PTR FS:[0],ESP  

0034E6B2    CC              INT3                                      ; Int3 异常  

0034E6B3    90              NOP  

0034E6B4    64:8F05 0000000>POP DWORD PTR FS:[0]  

0034E6BB    83C4 04         ADD ESP,4  

0034E6BE    8985 7AF44000   MOV DWORD PTR SS:[EBP+40F47A],EAX     ; [3509c2] = D8A80C19  

0034E6C4    8B85 46F44000   MOV EAX,DWORD PTR SS:[EBP+40F446]     ; [35098E]  

0034E819    0BC0            OR EAX,EAX  

0034E81B    0F85 4B040000   JNZ 0034EC6C  

0034E821    8D85 C91A4000   LEA EAX,DWORD PTR SS:[EBP+401AC9]     ; 343011  

0034E827    0185 8AF44000   ADD DWORD PTR SS:[EBP+40F48A],EAX     ; [3509d2] = 0  

0034E82D    33C0            XOR EAX,EAX  

0034E82F    8B8D 6AF44000   MOV ECX,DWORD PTR SS:[EBP+40F46A]     ; [3509B2] = 0  

0034E984    83F9 01         CMP ECX,1  

0034E987    0F85 DF020000   JNZ 0034EC6C  

0034EC6C    8B85 46F44000   MOV EAX,DWORD PTR SS:[EBP+40F446]     ; [35098E]  

0034EC72    0BC0            OR EAX,EAX  

0034EC74    0F85 68010000   JNZ 0034EDE2  

0034EC7A    8B85 FEBF4000   MOV EAX,DWORD PTR SS:[EBP+40FBFE]     ; [351146]  

0034EC80    0BC0            OR EAX,EAX  

0034EC82    0F84 5A010000   JE 0034EDE2  

0034EDE2    8BB5 12FC4000   MOV ESI,DWORD PTR SS:[EBP+40FC12]     ; [35115A]=1000  

0034EDE8    03B5 36F44000   ADD ESI,DWORD PTR SS:[EBP+40F436]     ; adalinks.00400000  

0034EDEE    8B8D 16FC4000   MOV ECX,DWORD PTR SS:[EBP+40FC16]     ; 81000  

0034EDF4    E8 FE040000     CALL 0034F2F7                           ; F8, 计算用户代码校验和 D3981C17  

0034EDF9    8985 4EF44000   MOV DWORD PTR SS:[EBP+40F44E],EAX     ; [350996]  

0034EDFF    8BC5            MOV EAX,EBP                             ; FFF41548  

0034EE01    8DB5 3AFD4000   LEA ESI,DWORD PTR SS:[EBP+40FD3A]      ; 351282  

0034EE07    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE0A    0146 08         ADD DWORD PTR DS:[ESI+8],EAX  

0034EE0D    83C6 20         ADD ESI,20  

0034EE10    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE13    83C6 20         ADD ESI,20  

0034EE16    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE19    0146 08         ADD DWORD PTR DS:[ESI+8],EAX  

0034EE1C    83C6 20         ADD ESI,20  

0034EE1F    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE22    83C6 20         ADD ESI,20  

0034EE25    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE28    83C6 20         ADD ESI,20  

0034EE2B    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE2E    83C6 20         ADD ESI,20  

0034EE31    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE34    83C6 20         ADD ESI,20  

0034EE37    0146 04         ADD DWORD PTR DS:[ESI+4],EAX  

0034EE3A    8DB5 36FD4000   LEA ESI,DWORD PTR SS:[EBP+40FD36]    ; 35127E  

0034EE40    0106            ADD DWORD PTR DS:[ESI],EAX  

0034EE42    8D85 5AFE4000   LEA EAX,DWORD PTR SS:[EBP+40FE5A]    ; 3513A2  

0034EE48    50              PUSH EAX  

0034EF98    64:FF35 0000000>PUSH DWORD PTR FS:[0]  

0034EF9F    64:8925 0000000>MOV DWORD PTR FS:[0],ESP  

0034EFA6    33C0            XOR EAX,EAX  

0034EFA8    8B00            MOV EAX,DWORD PTR DS:[EAX]           ; 内存访问异常, SEH = 3513A2  

0034EFAA    90              NOP  

0034EFAB    90              NOP                                   ; 单步  

0034EFAC    CC              INT3                                   ; INT3  

0034EFAD    E8 FB          JMP SHORT 0034EFAA  

0034CDAD    9C              PUSHFD  

0034CDAE    9C              PUSHFD                                ; 单步  

0034CDAF    58              POP EAX  

0034CDB0    80CC 01         OR AH,1                               ; 设置单步标志  

0034CDB3    50              PUSH EAX  

0034CDB4    9D              POPFD  

0034CDB5    9D              POPFD  

0034CABB    33C0            XOR EAX,EAX  

0034CABD    F7F0            DIV EAX                                ; x/0  

0034BDB9    40              INC EAX  

0034BDBA    D1C8            ROR EAX,1  

0034BDBC    CE              INTO                                   ; 整数溢出  

0034BD43    6285 2AFD4000   BOUND EAX,QWORD PTR SS:[EBP+40FD2A] ; 数组超限  

; SEH  

003513A2    8BD4            MOV EDX,ESP  

003513A4    60              PUSHAD  

003513A5    8B7A 0C         MOV EDI,DWORD PTR DS:[EDX+C]         ; pContext  

003513A8    8BAF B4000000   MOV EBP,DWORD PTR DS:[EDI+B4]        ; regEBP  

003513AE    8BB5 36FD4000   MOV ESI,DWORD PTR SS:[EBP+40FD36]    ; [35127e]  

003513B4    8B5A 04         MOV EBX,DWORD PTR DS:[EDX+4]         ; pEXCEPTION_RECORD  

003513B7    AD              LODS DWORD PTR DS:[ESI]             ; EXCEPTION_CODE  

003513B8    3B03            CMP EAX,DWORD PTR DS:[EBX]          ; [EBX] 必须等于 EXCEPTION_CODE  
```

## 异常处理结果

下面是结果，这将是我们要经过的 8 个异常。

| ExceptionCode | regEIP | Dr0 | Dr1 |
| :--- | :--- | :--- | :--- |
| Dr2 | Dr3 | Dr6 | Dr7 |

```
00351282  05 00 00 C0 AA EF 34 00 AB EF 34 00 00 00 00 00   
00351292  00 00 00 00 00 00 00 00 00 00 00 00 01 01 00 00   
003512A2  04 00 00 80 AB EF 34 00 00 00 00 00 00 00 00 00   
003512B2  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
003512C2  03 00 00 80 AD CD 34 00 AE CD 34 00 00 00 00 00  
003512D2  00 00 00 00 00 00 00 00 00 00 00 00 01 01 00 00   
003512E2  04 00 00 80 AE CD 34 00 00 00 00 00 00 00 00 00  
003512F2  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
00351302  04 00 00 80 BB CA 34 00 00 00 00 00 00 00 00 00  
00351312  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
00351322  94 00 00 C0 B9 BD 34 00 00 00 00 00 00 00 00 00   
00351332  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
00351342  95 00 00 C0 43 BD 34 00 00 00 00 00 00 00 00 00  
00351352  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
00351362  8C 00 00 C0 2B A3 34 00 00 00 00 00 00 00 00 00   
00351372  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
```

### 异常流程

1. `34EFA8` 内存访问异常，`regEIP` = `34EFAA`，`dr0`= `34EFAB`，结束，`F9`
2. `34EFAB` 单步异常(由于 `Dr0`)，`regEIP` = `34EFAB`，`drx`= `0`，结束，`F9`
3. `34EFAC` `INT3`，`regEIP` = `34CDAD`，`dr0`= `34CDAE`，结束，`F9`
4. `34CDAE` 单步异常(由于 `Dr0`)，`regEIP` = `34CDAE`，`drx`= `0`，结束，`F9`
5. `34CDB5` 单步异常(由于 `34CDB0`)，`regEIP` = `34CABB`，`drx`= `0`，结束，`F9`
6. `34CABD` `DIV0`，`regEIP` = `34BDB9`，`drx`= `0`，结束，`F9`
7. `34BDBC` `INT0`，`regEIP` = `34BD43`，`drx`= `0`，结束，`F9`
8. `34BD43` 数组超限，`regEIP` = `34A32B`，`drx`= `0`，OK，不能用 `F9` 了，`SHIFT+F9` 到 `34A32B`




十二. 清理现场, 走向胜利

```
003513BA    0F85 70010000   JNZ 00351530                  ; 不能跳 ************************************************************
003513C0    C707 17000100   MOV DWORD PTR DS:[EDI],10017        ; ContextFlags
003513C6    AD              LODS DWORD PTR DS:[ESI]
003513C7    8987 B8000000   MOV DWORD PTR DS:[EDI+B8],EAX       ; regEIP
003513CD    8D7F 04         LEA EDI,DWORD PTR DS:[EDI+4]        ; EDI -> Dr0
0012FBCC    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr0
0012FBCD    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr1
0012FBCE    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr2
0012FBCF    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr3
0012FBD0    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr6
0012FBD1    A5              MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  ; Dr7
00351524    8385 36FD4000 2>ADD DWORD PTR SS:[EBP+40FD36],20    ; [35127E]+20
0035152B    61              POPAD
0035152C    33C0           XOR EAX,EAX
00351534    C3              RETN
```

8 个异常都处理完毕, 这里继续

```
0034A32B    64:8F05 0000000>POP DWORD PTR FS:[0]               ; 0012FFE0
0034A332    58              POP EAX
0034A482    ^\E9 D1F7FFFF     JMP 00349C58
```

```
00349C58    8BC5           MOV EAX,EBP
00349C5A    50              PUSH EAX
00349C5B    52              PUSH EDX
00349C5C    51              PUSH ECX
00349C60    0F31           RDTSC
00349C62    8BC8           MOV ECX,EAX
..................
00349C95    0F31           RDTSC
00349C97    83C4 04        ADD ESP,4
00349C9A    2BC1           SUB EAX,ECX
00349C9C    3D 00000200    CMP EAX,20000
00349CA1    76 04        JBE SHORT 00349CA7              ; 必须跳 ************************************************************
00349CA3    83C4 0C        ADD ESP,0C
00349CA6    C3              RETN
00349CA7    59              POP ECX
00349CA8    5A              POP EDX
00349CA9    58              POP EAX
00349CAA    EB 30          JMP SHORT 00349CDC
; 把 351282 的内容再加密起来
00349CDC    8DB5 3AFD4000   LEA ESI,DWORD PTR SS:[EBP+40FD3A]     ; 还搞了一个 SEH 死循环, 但没用, 马上撤消了???
00349CE2    2946 04        SUB DWORD PTR DS:[ESI+4],EAX
00349CE5    50              PUSH EAX
00349CE6    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349CED    83EC 08        SUB ESP,8
00349CF0    B8 1FF44000     MOV EAX,40F41F
00349CF5    50              PUSH EAX
00349CF6    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349CFD    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349D04    83C4 10        ADD ESP,10
00349D07    B8 1FF44000     MOV EAX,40F41F
00349D0C    50              PUSH EAX
00349D0D    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349D14    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349D1B    B8 1FF44000     MOV EAX,40F41F
00349D20    50              PUSH EAX
00349D21    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349D28    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349D2F    33C0           XOR EAX,EAX
00349D31    40              INC EAX
00349D32    83C4 10        ADD ESP,10
00349D35    64:8F05 0000000>POP DWORD PTR FS:[0]
00349D3C    58              POP EAX
00349D3D    2946 08        SUB DWORD PTR DS:[ESI+8],EAX
00349D40    50              PUSH EAX
00349D41    52              PUSH EDX
00349D42    51              PUSH ECX
00349D46    0F31           RDTSC
00349D48    8BC8           MOV ECX,EAX
00349D7B    0F31           RDTSC
00349D7D    83C4 04        ADD ESP,4
00349D80    2BC1           SUB EAX,ECX
00349D82    3D 00000200    CMP EAX,20000
00349D87    76 04        JBE SHORT 00349D8D               ; 必须跳 ************************************************************
00349D89    83C4 0C        ADD ESP,0C
00349D8C    C3              RETN
00349D8D    59              POP ECX
00349D8E    5A              POP EDX
00349D8F    58              POP EAX
00349D90    EB 30          JMP SHORT 00349DC2
00349DC2    83C6 20        ADD ESI,20
00349DC5    2946 04        SUB DWORD PTR DS:[ESI+4],EAX
00349DC8    50              PUSH EAX
00349DC9    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349DD0    83EC 08        SUB ESP,8
00349DD3    B8 1FF44000     MOV EAX,40F41F
00349DD8    50              PUSH EAX
00349DD9    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349DE0    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349DE7    83C4 10        ADD ESP,10
00349DEA    B8 1FF44000     MOV EAX,40F41F
00349DEF    50              PUSH EAX
00349DF0    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349DF7    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349DFE    B8 1FF44000     MOV EAX,40F41F
00349E03    50              PUSH EAX
00349E04    64:FF35 0000000>PUSH DWORD PTR FS:[0]
00349E0B    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00349E12    33C0           XOR EAX,EAX
00349E14    40              INC EAX
00349E15    83C4 10        ADD ESP,10
00349E18    64:8F05 0000000>POP DWORD PTR FS:[0]
00349E1F    58              POP EAX
00349E20    83C6 20        ADD ESI,20
00349E23    2946 04        SUB DWORD PTR DS:[ESI+4],EAX
00349E26    50              PUSH EAX
00349E27    52              PUSH EDX
00349E28    51              PUSH ECX
00349E61    0F31           RDTSC
00349E63    83C4 04        ADD ESP,4
00349E66    2BC1           SUB EAX,ECX
00349E68    3D 00000200    CMP EAX,20000
00349E6D    76 04        JBE SHORT 00349E73       ; 必须跳 ************************************************************
00349F4A    0F31           RDTSC
00349F4C    83C4 04        ADD ESP,4
00349F4F    2BC1           SUB EAX,ECX
00349F51    3D 00000200    CMP EAX,20000
00349F56    76 04        JBE SHORT 00349F5C        ; 必须跳 ************************************************************
0034A033    0F31           RDTSC
0034A035    83C4 04        ADD ESP,4
0034A038    2BC1           SUB EAX,ECX
0034A03A    3D 00000200    CMP EAX,20000
0034A03F    76 04        JBE SHORT 0034A045        ; 必须跳 ************************************************************
0034A07A    2946 04        SUB DWORD PTR DS:[ESI+4],EAX
0034A07D    8DB5 36FD4000   LEA ESI,DWORD PTR SS:[EBP+40FD36]  ; 35127E
0034A083    B8 3AFD4000     MOV EAX,40FD3A
0034A088    8906           MOV DWORD PTR DS:[ESI],EAX
0034A08A    8D85 558B4000   LEA EAX,DWORD PTR SS:[EBP+408B55]   ;  返回地址 34A09D
0034A090    50              PUSH EAX
0034A091    8B85 05F64000   MOV EAX,DWORD PTR SS:[EBP+40F605]   ; KERNEL32.GetCurrentThread
0034A097    E9 20620000     JMP 003502BC
0034A09D    6A 00          PUSH 0
0034A09F    50              PUSH EAX
0034A0A0    8D85 BA8C4000   LEA EAX,DWORD PTR SS:[EBP+408CBA]   ;  返回地址 34A202
0034A0A6    50              PUSH EAX
0034A1F6    8B85 B5F64000   MOV EAX,DWORD PTR SS:[EBP+40F6B5]   ; KERNEL32.SetThreadPriority
0034A1FC    E9 BB600000     JMP 003502BC
0034A202    ^\E9 F6C0FFFF     JMP 003462FD
003462FD    50              PUSH EAX
003462FE    64:FF35 0000000>PUSH DWORD PTR FS:[0]
```



```asm
00346305    83EC 08             SUB ESP,8
00346308    B8 1FF44000         MOV EAX,40F41F
0034630D    50                PUSH EAX
0034630E    64:FF35 00000000   PUSH DWORD PTR FS:[0]
00346315    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
0034631C    83C4 10           ADD ESP,10
0034631F    B8 1FF44000        MOV EAX,40F41F
00346324    50                PUSH EAX
00346325    64:FF35 00000000   PUSH DWORD PTR FS:[0]
0034632C    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
00346333    B8 1FF44000        MOV EAX,40F41F
00346338    50                PUSH EAX
00346339    64:FF35 00000000   PUSH DWORD PTR FS:[0]
00346340    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
00346347    33C0              XOR EAX,EAX
00346349    40                INC EAX
0034634A    83C4 10           ADD ESP,10
0034634D    64:8F05 00000000   POP DWORD PTR FS:[0]
00346354    58                POP EAX
00346355    FF85 46F44000      INC DWORD PTR SS:[EBP+40F446]
0034635B    50                PUSH EAX
0034635C    52                PUSH EDX
0034635D    51                PUSH ECX
00346396    0F31              RDTSC
00346398    83C4 04           ADD ESP,4
0034639B    2BC1              SUB EAX,ECX
0034639D    3D 00000400       CMP EAX,40000
003463A2    76 04             JBE SHORT 003463A8    ; 必须跳 *****************************************************
003463A4    83C4 0C           ADD ESP,0C
003463A7    C3                RETN
003463A8    59                POP ECX
003463A9    5A                POP EDX
003463AA    58                POP EAX
003463AB    EB 33             JMP SHORT 003463E0

003463E7    83BD 96F44000 00  CMP DWORD PTR SS:[EBP+40F496],0 ; [3509DE]=EFAB
003463EE   /0F84 28010000     JE 0034651C
003463F5    64:FF35 00000000   PUSH DWORD PTR FS:[0]
003463FC    83EC 08           SUB ESP,8
003463FF    B8 1FF44000        MOV EAX,40F41F
00346404    50                PUSH EAX
00346405    64:FF35 00000000   PUSH DWORD PTR FS:[0]
0034640C    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
00346413    83C4 10           ADD ESP,10
00346416    B8 1FF44000        MOV EAX,40F41F
0034641B    50                PUSH EAX
0034641C    64:FF35 00000000   PUSH DWORD PTR FS:[0]
00346423    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
0034642A    B8 1FF44000        MOV EAX,40F41F
0034642F    50                PUSH EAX
00346430    64:FF35 00000000   PUSH DWORD PTR FS:[0]
00346437    64:8925 00000000   MOV DWORD PTR FS:[0],ESP
0034643E    33C0              XOR EAX,EAX
00346440    40                INC EAX
00346441    83C4 10           ADD ESP,10
00346444    64:8F05 00000000   POP DWORD PTR FS:[0]
0034644B    58                POP EAX
0034644C    8D85 C91A4000      LEA EAX,DWORD PTR SS:[EBP+401AC9] ; 343011
00346452    EB 02             JMP SHORT 00346456
00346456    0385 96F44000      ADD EAX,DWORD PTR SS:[EBP+40F496] ; [3509DE]
0034645C    E8 03000000        CALL 00346464
00346467    894424 EC          MOV DWORD PTR SS:[ESP-14],EAX    ; 351FBC
00346470    61                POPAD ; 快到 OEP 了
00346471    50                PUSH EAX
00346472    52                PUSH EDX
00346473    51                PUSH ECX
00346477    0F31              RDTSC
00346479    8BC8              MOV ECX,EAX
003464AC    0F31              RDTSC
003464AE    83C4 04           ADD ESP,4
003464B1    2BC1              SUB EAX,ECX
003464B3    3D 00000400       CMP EAX,40000 ; 必须跳 *****************************************************
003464B8    76 04             JBE SHORT 003464BE
003464BA    83C4 0C           ADD ESP,0C
003464BD    C3                RETN
003464BE    59                POP ECX
003464BF    5A                POP EDX
003464C0    58                POP EAX
003464C1    EB 33             JMP SHORT 003464F6

00351FBC    68 BC354000       PUSH 4035BC ; Stolen Code
00351FC1    EB 01             JMP SHORT 00351FC4
00351FC9   /EB 11            JMP SHORT 00351FDC
00351FCB   |50               PUSH EAX
00351FCC   |E8 04000000      CALL 00351FD5
00351FD1   |E8 31400058      CALL 58356007
00351FD6   |8B00             MOV EAX,DWORD PTR DS:[EAX] ; [351FD1]=4031E8
00351FD8   |870424          XCHG DWORD PTR SS:[ESP],EAX
00351FDB   |C3              RETN ; 4031E8 OEP ************************************************
00351FDC   \EB 01            JMP SHORT 00351FDF
00351FDE    27              DAA
00351FDF    68 FA314000      PUSH 4031FA ; ***********
00351FE4    C3              RETN

004031F0    68 BC354000       PUSH adalinks.004035BC ; 修改 OEP 如下
004031F5    E8 EEFFFFFF      CALL adalinks.004031E8
004031FA    0000

00402D50    90              NOP ; 共 C5 个 API
00402D51  - E9 4ADBDB00      JMP 011C08A0
00402D56    90              NOP
00402D57  - E9 E4DFDB00      JMP 011C0D40
....
004031E2    90              NOP
004031E3  - E9 D8DCDB00      JMP 011C0EC0
004031E8    90              NOP
004031E9  - E9 92E1DB00      JMP 011C1380
```

写一段代码修复一下 `IAT`

```asm
00480A30    60                PUSHAD
00480A31    BF 00104000       MOV EDI,adalinks.00401000 ; 401000 开始放 THUNK
00480A36    BE 502D4000       MOV ESI,adalinks.00402D50
00480A3B    B9 C5000000       MOV ECX,0C5
00480A40    8B46 02           MOV EAX,DWORD PTR DS:[ESI+2]
00480A43    03C6              ADD EAX,ESI
00480A45    83C0 06           ADD EAX,6
00480A48    8B58 09           MOV EBX,DWORD PTR DS:[EAX+9]
00480A4B    3358 1C           XOR EBX,DWORD PTR DS:[EAX+1C]
00480A4E    891F              MOV DWORD PTR DS:[EDI],EBX ; [401000] 变成 API address
00480A50    66:C706 FF25      MOV WORD PTR DS:[ESI],25FF
00480A55    83C6 02           ADD ESI,2
00480A58    893E              MOV DWORD PTR DS:[ESI],EDI ; 402D50 变成 JMP [401000]
00480A5A    83C6 04           ADD ESI,4
00480A5D    83C7 04           ADD EDI,4
00480A60  ^ E2 DE             LOOPD SHORT adalinks.00480A40
00480A62    61                POPAD
```

`60 BF 00 10 40 00 BE 50 2D 40 00 B9 C5 00 00 00 8B 46 02 03 C6 83 C0 06 8B 58 09 33 58 1C 89 1F 66 C7 06 FF 25 83 C6 02 89 3E 83 C6 04 83 C7 04 E2 DE 61`

再写一段代码，修复 `LEA EDI, [XXXX]` 等语句为 `LEA EDI, [XXXX-1]`

比如 `LEA EDI, [402D51]` 要改为 `LEA EDI, [402D50]`

修好后 `DUMP`，`ImportREC` 修复

```asm
0048BD00      60              PUSHAD
0048BD01      BE 00104000     MOV ESI,401000
0048BD06      AC              LODS BYTE PTR DS:[ESI]
0048BD07      3C 8D           CMP AL,8D
0048BD09      75 2F           JNZ SHORT fix.0048BD3A
0048BD0B      8A06            MOV AL,BYTE PTR DS:[ESI]
0048BD0D      3C 1D           CMP AL,1D ; 8D 1D
0048BD0F      74 12           JE SHORT fix.0048BD23
0048BD11      3C 3D           CMP AL,3D ; 8D 3D
0048BD13      74 0E           JE SHORT fix.0048BD23
0048BD15      3C 15           CMP AL,15 ; 8D 15
0048BD17      74 0A           JE SHORT fix.0048BD23
0048BD19      3C 35           CMP AL,35 ; 8D 35
0048BD1B      74 06           JE SHORT fix.0048BD23
0048BD1D      EB 1B           JMP SHORT fix.0048BD3A
0048BD1F      90              NOP
0048BD20      90              NOP
0048BD21      90              NOP
0048BD22      90              NOP
0048BD23      8B46 01         MOV EAX,DWORD PTR DS:[ESI+1]
0048BD26      3D 502D4000     CMP EAX,402D50 ; 开始的地方
0048BD2B      72 0D           JB SHORT fix.0048BD3A
0048BD2D      3D EE314000     CMP EAX,fix.004031EE ; 结束的地方
```



```asm
0048BD32     77 06          JA  SHORT fix.0048BD3A  

0048BD34     46             INC ESI                             ; 需要修正  

0048BD35     FF0E          DEC DWORD PTR DS:[ESI]  

0048BD37     83C6 04       ADD ESI,4  

0048BD3A     81FE E0BC4800 CMP ESI,fix.0048BCE0  

0048BD40     ^ 72 C4       JB  SHORT fix.0048BD06  

0048BD42     61            POPAD  

60 BE 00 10 40 00 AC 3C 8D 75 2F 8A 06 3C 1D 74 12 3C 3D 74 0E 3C 15 74 0A 3C 35 74 06 EB 1B 90  

90 90 90 8B 46 01 3D 50 2D 40 00 72 0D 3D EE 31 40 00 77 06 46 FF 0E 83 C6 04 81 FE E0 BC 48 00  

72 C4 61  
```

运行程序，用任何一种模式玩游戏都出错，看来有暗桩，下断 `rtcFileLen`，点击学习模式

```asm
00437963   .  E8 D2B4FCFF             CALL <JMP.&msvbvm60.rtcFileLen>       
  
00437968   .  90                      NOP                                   ; EAX = A1000 (脱壳后大小)  

00437969   .  33C9                    XOR ECX,ECX  

0043796B   .  8A0C1E                  MOV CL,BYTE PTR DS:[ESI+EBX]  

0043796E   .  6BC9 64                 IMUL ECX,ECX,64  

00437971   .  0F80 79030000           JO  fix.00437CF0  

00437977   .  33D2                    XOR EDX,EDX  

00437979   .  8A143E                  MOV DL,BYTE PTR DS:[ESI+EDI]  

0043797C   .  03CA                    ADD ECX,EDX  

0043797E   .  0F80 6C030000           JO  fix.00437CF0  

00437984   .  69C9 00010000           IMUL ECX,ECX,100                      ; ECX = 3A600 (未脱壳大小)  

0043798A   .  0F80 60030000           JO  fix.00437CF0  

00437990   .  33D2                    XOR EDX,EDX                           ; EDX = 0 (标志)  

00437992   .  3BC1                    CMP EAX,ECX                           ; 比较大小  

00437994   .  0F95C2                  SETNE DL                              ; NOP  

004379A5   .  F7DA                    NEG EDX  

004379A7   .  8BF2                    MOV ESI,EDX  

004379CA   .  66:85F6                TEST SI,SI  

004379CD   .  74 2D                   JE  SHORT fix.004379FC                ; 必须跳  
```

还有文件字节的校验，并用校验和生成 `Unzip32` 的解压密码，有兴趣各位研究一下.

```asm
004549C3   .  50            PUSH EAX  

004549C4   .  68 301A4100   PUSH fix.00411A30            ; UNICODE ".exe"  这个是文件校验的  

004549C9   .  FFD3          CALL EBX                     ; vbaStrCat     

004549CB   .  8BD0          MOV EDX,EAX  

004549CD   .  8D4D 98       LEA ECX,DWORD PTR SS:[EBP-68]  

004549D0   .  FFD6          CALL ESI  

004549D2   .  8B4D AC       MOV ECX,DWORD PTR SS:[EBP-54]  

004549D5   .  50            PUSH EAX  

004549D6   .  6A 01         PUSH 1  

004549D8   .  51            PUSH ECX  

004549D9   .  68 CCE94000   PUSH fix.0040E9CC  

004549DE   .  FFD3          CALL EBX  

004549E0   .  8BD0          MOV EDX,EAX  

004549E2   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

004549E5   .  FFD6          CALL ESI  

004549E7   .  8B55 A8       MOV EDX,DWORD PTR SS:[EBP-58]  

004549EA   .  50            PUSH EAX  

004549EB   .  52            PUSH EDX  

004549EC   .  FFD3          CALL EBX  

004549EE   .  8BD0          MOV EDX,EAX  

004549F0   .  8D4D A0       LEA ECX,DWORD PTR SS:[EBP-60]  

004549F3   .  FFD6          CALL ESI  

004549F5   .  50            PUSH EAX  

004549F6   .  68 301A4100   PUSH fix.00411A30            ;  UNICODE ".exe"   这个是启动子进程的  

004549FB   .  FFD3          CALL EBX                     ;  vbaStrCat  

004549FD   .  8BD0          MOV EDX,EAX  

004549FF   .  8D4D 9C       LEA ECX,DWORD PTR SS:[EBP-64]  

00454A02   .  FFD6          CALL ESI  

00454A04   .  50            PUSH EAX  

00454A05   .  E8 0EE5FAFF   CALL <JMP.&msvbvm60.__vbaLenBstr>  

00454A0A   .  90            NOP  

00454A0B   .  8BC8          MOV ECX,EAX  

00454A0D   .  E8 78E5FAFF   CALL <JMP.&msvbvm60.__vbaI2I4>  

00454A12   .  90            NOP  

00454A13   .  50            PUSH EAX  

00454A14   .  6A 20         PUSH 20  

00454A16   .  E8 A9E4FAFF   CALL <JMP.&msvbvm60.__vbaFileOpen>  
```

今天才发现这篇……简单看了一下，除了表达对你非人的毅力之钦佩，还有几点要说：

1. 好多 `rdtsc` 是花，直接枪毙。
2. `ShellTmpMap` 是空的。
3. 无限 `SEH` 也是花，如果旧版调试器就跟不完，新版能放慢速度。
4. “代码段里 `CALL` `IAT`, `JMP` `IAT` 等语句中 跳转距离需要修改”

   这是不正确的，因为提高了冗余度，要把代码还原回来，跟 `IAT` 没关系。
5. “跟 `OSVersion` 有关的东东?”

   `NT` 系列下检查 `INT` `1` 的 `DPL`，`9x` 下我忘了。
6. 这个版本好像不是很新哦

`4dd466d0e1e0cc123afbe504da2462ca`
