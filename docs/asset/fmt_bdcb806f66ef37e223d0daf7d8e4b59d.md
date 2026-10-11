
```asm
004011CE   .  A3 F4DB4000   MOV DWORD PTR DS:[40DBF4],EAX             ; |

004011D3   .  0FB746 14     MOVZX EAX,WORD PTR DS:[ESI+14]            ; |

004011D7   .  51            PUSH ECX                                  ; |Size

004011D8   .  6A 00         PUSH 0                                     ; |Address = NULL

004011DA   .  8D7C30 18     LEA EDI,DWORD PTR DS:[EAX+ESI+18]         ; |EDI 指向 HookStubt 字符串 // 该资源 PE 特殊的一个区段名称

004011DE   .  FF15 28A04000 CALL DWORD PTR DS:[<&KERNEL32.VirtualAll>; \\VirtualAlloc

004011E4   .  8B96 80000000 MOV EDX,DWORD PTR DS:[ESI+80]             ;  输入表地址

004011EA   .  8B8E 84000000 MOV ECX,DWORD PTR DS:[ESI+84]             ;  输入表长度

004011F0   .  A3 3CDC4000   MOV DWORD PTR DS:[40DC3C],EAX             ;  刚申请的 size = 2000 指针

004011F5   .  8B47 0C       MOV EAX,DWORD PTR DS:[EDI+C]              ;  HookStubt 区段的 VA 地址

004011F8   .  2BD0          SUB EDX,EAX

004011FA   .  8915 0CDC4000 MOV DWORD PTR DS:[40DC0C],EDX

00401200   .  8B56 38       MOV EDX,DWORD PTR DS:[ESI+38]             ;  内存节对齐粒度 = 200  // 这个 PE 格式比较特殊 O(∩\_∩)O~

00401203   .  890D DCDA4000 MOV DWORD PTR DS:[40DADC],ECX             ;  输入表长度送[40DADC]

00401209   .  8B4F 08       MOV ECX,DWORD PTR DS:[EDI+8]              ;  HookStubt 区段的 Size

0040120C   .  8D6F 08       LEA EBP,DWORD PTR DS:[EDI+8]

0040120F   .  895424 18     MOV DWORD PTR SS:[ESP+18],EDX            ;  存放一下 内存对齐粒度

00401213   .  8B56 28       MOV EDX,DWORD PTR DS:[ESI+28]            ;  OEP

00401216   .  83E9 14       SUB ECX,14

00401219   .  2BD0          SUB EDX,EAX

0040121B   .  A3 2CDC4000   MOV DWORD PTR DS:[40DC2C],EAX             ;  HookStub 的 VA

00401220   .  890D 04DC4000 MOV DWORD PTR DS:[40DC04],ECX             ;  74-14=60

00401226   .  8915 40DC4000 MOV DWORD PTR DS:[40DC40],EDX             ;  AFE = EFE- 400

0040122C   .  C74424 10 000>MOV DWORD PTR SS:[ESP+10],0

00401234   .  C74424 14 040>MOV DWORD PTR SS:[ESP+14],4                ;  该资源 PE 的区段数

0040123C   .  8D6424 00     LEA ESP,DWORD PTR SS:[ESP]

00401240   >  8B45 00       MOV EAX,DWORD PTR SS:[EBP]

00401243   .  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]               ;  该区段 在文件的偏移

00401246   .  8B15 3CDC4000 MOV EDX,DWORD PTR DS:[40DC3C]              ;  为之申请的 空间

0040124C   .  50            PUSH EAX                                    ;  SIZE

0040124D   .  8B4424 14     MOV EAX,DWORD PTR SS:[ESP+14]               ;  该资源 PE 的区段数

00401251   .  03CB          ADD ECX,EBX

00401253   .  51            PUSH ECX                                    ;  源地址

00401254   .  03D0          ADD EDX,EAX

00401256   .  52            PUSH EDX                                    ;  循环 COPY 函数

00401257   .  E8 84550000   CALL Fengyue'.004067E0                     ;  memcpy 函数

0040125C   .  8B55 00       MOV EDX,DWORD PTR SS:[EBP]

0040125F   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+24]               ;  对齐粒度

00401263   .  8D440A FF     LEA EAX,DWORD PTR DS:[EDX+ECX-1]

00401267   .  33D2          XOR EDX,EDX

00401269   .  F7F1          DIV ECX

0040126B   .  83C4 0C       ADD ESP,0C

0040126E   .  83C5 28       ADD EBP,28                                   ;  下一个区段的 size

00401271   .  0FAFC1        IMUL EAX,ECX

00401274   .  014424 10     ADD DWORD PTR SS:[ESP+10],EAX               ;  累加器

00401278   .  836C24 14 01  SUB DWORD PTR SS:[ESP+14],1                 ;  该资源 PE 的区段数

0040127D   .^. 75 C1         JNZ SHORT Fengyue'.00401240

0040127F   .  8B4424 10     MOV EAX,DWORD PTR SS:[ESP+10]

00401283   .  6A 04         PUSH 4                                       ; /Protect = PAGE\_READWRITE

00401285   .  A3 F4DB4000   MOV DWORD PTR DS:[40DBF4],EAX               ; |

0040128A   .  8B86 A4000000 MOV EAX,DWORD PTR DS:[ESI+A4]               ; |

00401290   .  68 00100000   PUSH 1000                                    ; |AllocationType = MEM\_COMMIT

00401295   .  50            PUSH EAX                                     ; |Size = 64

00401296   .  6A 00         PUSH 0                                       ; |Address = NULL

00401298   .  A3 1CDC4000   MOV DWORD PTR DS:[40DC1C],EAX               ; |

0040129D   .  FF15 28A04000 CALL DWORD PTR DS:[<&KERNEL32.VirtualAll>; \\VirtualAlloc

004012A3   .  8B0D 1CDC4000 MOV ECX,DWORD PTR DS:[40DC1C]

004012A9   .  8B96 A0000000 MOV EDX,DWORD PTR DS:[ESI+A0]               ;  Relocation Table address 重定位表  1C00

004012AF   .  51            PUSH ECX

004012B0   .  8BE8          MOV EBP,EAX

004012B2   .  52            PUSH EDX

004012B3   .  8BCB          MOV ECX,EBX

004012B5   .  892D FCDB4000 MOV DWORD PTR DS:[40DBFC],EBP

004012BB   .  E8 50080000   CALL Fengyue'.00401B10                     ; 该函数校对各节表的数据

004012C0   .  83C4 04       ADD ESP,4

004012C3   .  03C3         ADD EAX,EBX

004012C5   .  50            PUSH EAX

004012C6   .  55            PUSH EBP

004012C7   .  E8 14550000   CALL Fengyue'.004067E0                     ;  memcpy 函数

004012CC   .  8BCD          MOV ECX,EBP

004012CE   .  83C4 0C       ADD ESP,0C

004012D1   .  8379 04 00    CMP DWORD PTR DS:[ECX+4],0                  ;  长度

004012D5   .  74 7F         JE SHORT Fengyue'.00401356

004012D7   .  EB 07         JMP SHORT Fengyue'.004012E0

004012D9   .  8DA424 000000>LEA ESP,DWORD PTR SS:[ESP]

004012E0   >  8B47 0C       MOV EAX,DWORD PTR DS:[EDI+C]                ;  第 1 个区段的 VA

004012E3   .  8B59 04       MOV EBX,DWORD PTR DS:[ECX+4]                ;  size

004012E6   .  25 00F0FFFF   AND EAX,FFFFF000

004012EB   .  2901          SUB DWORD PTR DS:[ECX],EAX

004012ED   .  83EB 08       SUB EBX,8                                    ;  -8

004012F0   .  D1EB          SHR EBX,1                                    ;  / 2  word 型的去处理

004012F2   .  BA 00000000   MOV EDX,0

004012F7   .  74 54         JE SHORT Fengyue'.0040134D

004012F9   .  8DA424 000000>LEA ESP,DWORD PTR SS:[ESP]

00401300   >  0FB74451 08   MOVZX EAX,WORD PTR DS:[ECX+EDX*2+8]         ;  按 WORD 型循环处理 数据

00401305   .  8BE8          MOV EBP,EAX

00401307   .  81E5 00F00000 AND EBP,0F000

0040130D   .  81FD 00300000 CMP EBP,3000

00401313   .  75 31         JNZ SHORT Fengyue'.00401346

00401315   .  66:8B6F 0C    MOV BP,WORD PTR DS:[EDI+C]                  ;  VA

00401319   .  25 FF0F0000   AND EAX,0FFF                                 ;  保留低 4*3 位

0040131E   .  66:81E5 FF0F  AND BP,0FFF

00401323   .  66:2BC5       SUB AX,BP

00401326   .  8B2D 3CDC4000 MOV EBP,DWORD PTR DS:[40DC3C]               ;  空间 基址

0040132C   .  66:0D 0030    OR AX,3000                                  ;  将数据处理后的结果

00401330   .  66:894451 08  MOV WORD PTR DS:[ECX+EDX*2+8],AX            ;  解码后的数据替换原数据 进行保存

00401335   .  25 FF0F0000   AND EAX,0FFF

0040133A   .  0301          ADD EAX,DWORD PTR DS:[ECX]

0040133C   .  03C5          ADD EAX,EBP                                  ;  计算出 空间中地址 进行重定位处理

0040133E   .  8B6E 34       MOV EBP,DWORD PTR DS:[ESI+34]               ;  ImageBase

00401341   .  036F 0C       ADD EBP,DWORD PTR DS:[EDI+C]                 ;  VA

00401344   .  2928          SUB DWORD PTR DS:[EAX],EBP                   ;  进行重定位调整

00401346   >  83C2 01       ADD EDX,1

00401349   .  3BD3          CMP EDX,EBX

0040134B   .^. 72 B3         JB SHORT Fengyue'.00401300

0040134D   >  0349 04       ADD ECX,DWORD PTR DS:[ECX+4]

00401350   .  8379 04 00    CMP DWORD PTR DS:[ECX+4],0

00401354   .^. 75 8A         JNZ SHORT Fengyue'.004012E0

00401356   >  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+1C]

0040135A   .  51            PUSH ECX                                      ; /hResource

0040135B   .  FF15 10A04000 CALL DWORD PTR DS:[<&KERNEL32.FreeResour>; \\FreeResource

00401361   .  5F            POP EDI

00401362   .  5E            POP ESI                                      ;  Fengyue'.0040F8A0

00401363   .  5D            POP EBP
```
