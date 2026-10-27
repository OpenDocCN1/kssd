
```
004A5EA2    59              POP ECX

004A5EA3    81E9 9E7E4000   SUB ECX,00407E9E

004A5EA9    BB 0B174000     MOV EBX,0040170B

004A5EAE    03D9            ADD EBX,ECX                              ; 计算出壳的EP

004A5EB0    B9 93670000     MOV ECX,6793

004A5EB5    E8 1C98FFFF     CALL 0049F6D6                            ; 这里进去计算crc值

004A5EBA    8985 2AC24000   MOV DWORD PTR SS:[EBP+40C22A],EAX        ; CRC值保存到[4AA22A处

……

004A5F1C    6A 00              PUSH 0                                   ; /phModule = 0

004A5F1E    FF95 C0BA4000      CALL DWORD PTR SS:[EBP+40BAC0]           ; \GetModuleHandleA

004A5F24    8985 DEBD4000      MOV DWORD PTR SS:[EBP+40BDDE],EAX        ; phModule保存到[004A9DDE]处

004A5F2A    8B58 3C            MOV EBX,DWORD PTR DS:[EAX+3C]            ; 定位pe头

004A5F2D    039D DEBD4000      ADD EBX,DWORD PTR SS:[EBP+40BDDE]

004A5F33    899D E2BD4000      MOV DWORD PTR SS:[EBP+40BDE2],EBX

004A5F39    8B85 E2BD4000      MOV EAX,DWORD PTR SS:[EBP+40BDE2]

004A5F3F    05 F8000000        ADD EAX,0F8

004A5F44    8985 E6BD4000      MOV DWORD PTR SS:[EBP+40BDE6],EAX        ; 定位Section name

004A5F4A    8B58 0C            MOV EBX,DWORD PTR DS:[EAX+C]             ; Section Voffset=1000

004A5F4D    039D DEBD4000      ADD EBX,DWORD PTR SS:[EBP+40BDDE]

004A5F53    899D EABD4000      MOV DWORD PTR SS:[EBP+40BDEA],EBX

004A5F59    8B58 08            MOV EBX,DWORD PTR DS:[EAX+8]             ; Vsize=73000

004A5F5C    899D EEBD4000      MOV DWORD PTR SS:[EBP+40BDEE],EBX

004A5F62    8B58 10            MOV EBX,DWORD PTR DS:[EAX+10]            ; Rsize=31c00

004A5F65    899D F2BD4000      MOV DWORD PTR SS:[EBP+40BDF2],EBX        ; 下面准备分配空间

004A5F6B    FFB5 EEBD4000      PUSH DWORD PTR SS:[EBP+40BDEE]           ; /MemSize = 73000 (Push Visze)

004A5F71    6A 40              PUSH 40                                  ; |Flags = GPTR

004A5F73    FF95 F4BA4000      CALL DWORD PTR SS:[EBP+40BAF4]           ; \GlobalAlloc

004A5F79    0BC0               OR EAX,EAX

004A5F7B    75 05              JNZ SHORT 004A5F82                       ; 如果分配成功则跳

004A5F7D    E9 2C3B0000        JMP 004A9AAE

004A5F82    8BF8               MOV EDI,EAX                              ; mov edi,hmem

……

004A5FC3    8B85 EABD4000       MOV EAX,DWORD PTR SS:[EBP+40BDEA]

004A5FC9    57                  PUSH EDI                                 ; /hmem =00142AB8

004A5FCA    50                  PUSH EAX                                 ; |Uzip address = 401000

004A5FCB    E8 353A0000         CALL 004A9A05                            ; \ApLib\_Unpack

004A5FD0    58                  POP EAX

004A5FD1    5F                  POP EDI

……

004A6032    FC                  CLD

004A6033    8B8D EEBD4000       MOV ECX,DWORD PTR SS:[EBP+40BDEE]

004A6039    8BF7                MOV ESI,EDI

004A603B    8BBD EABD4000       MOV EDI,DWORD PTR SS:[EBP+40BDEA]

004A6041    F3:A4               REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>; 还原代码

……

004A60F4    FFB5 F6BD4000       PUSH DWORD PTR SS:[EBP+40BDF6]           ; 重定位值9E000

004A60FA    FFB5 DEBD4000       PUSH DWORD PTR SS:[EBP+40BDDE]           ; imagebase

004A6100    E8 EF92FFFF         CALL 0049F3F4                            ; 这里进去就是输入表还原

0049F3F4    55                  PUSH EBP

0049F3F5    8BEC                MOV EBP,ESP

0049F3F7    83C4 F8             ADD ESP,-8

0049F3FA    60                  PUSHAD

0049F3FB    8B45 08             MOV EAX,DWORD PTR SS:[EBP+8]

0049F3FE    8B50 3C             MOV EDX,DWORD PTR DS:[EAX+3C]            ; 定位pe头

0049F401    03C2                ADD EAX,EDX

0049F403    8B90 80000000       MOV EDX,DWORD PTR DS:[EAX+80]            ; 定位输入表

0049F409    0355 08             ADD EDX,DWORD PTR SS:[EBP+8]

0049F40C    8BFA                MOV EDI,EDX

0049F40E    8B5F 0C             MOV EBX,DWORD PTR DS:[EDI+C]

0049F411    83FB 00             CMP EBX,0

0049F414    74 3C               JE SHORT 0049F452                        ; 如果输入表处理完就跳去结束处

0049F416    035D 08             ADD EBX,DWORD PTR SS:[EBP+8]

0049F419    895D FC             MOV DWORD PTR SS:[EBP-4],EBX

0049F41C    8B1F                MOV EBX,DWORD PTR DS:[EDI]

0049F41E    035D 08             ADD EBX,DWORD PTR SS:[EBP+8]

0049F421    895D F8             MOV DWORD PTR SS:[EBP-8],EBX

0049F424    33C9                XOR ECX,ECX

0049F426    8B048B              MOV EAX,DWORD PTR DS:[EBX+ECX\*4]

0049F429    8B5D F8             MOV EBX,DWORD PTR SS:[EBP-8]

0049F42C    3B5D 08             CMP EBX,DWORD PTR SS:[EBP+8]

0049F42F    74 03               JE SHORT 0049F434

0049F431    89048B              MOV DWORD PTR DS:[EBX+ECX\*4],EAX

0049F434    8B5D FC             MOV EBX,DWORD PTR SS:[EBP-4]

0049F437    8B45 0C             MOV EAX,DWORD PTR SS:[EBP+C]

0049F43A    05 BA124000         ADD EAX,004012BA

0049F43F    89048B              MOV DWORD PTR DS:[EBX+ECX\*4],EAX

0049F442    41                  INC ECX

0049F443    8B048B              MOV EAX,DWORD PTR DS:[EBX+ECX\*4]

0049F446    83F8 00             CMP EAX,0

0049F449    74 02               JE SHORT 0049F44D                        ; 如果当前DLL处理完则跳去处理下一个DLL的相关API

0049F44B  ^ EB D9               JMP SHORT 0049F426                       ; 循环回去还原输入表

0049F44D    83C7 14             ADD EDI,14

0049F450  ^ EB BC               JMP SHORT 0049F40E

0049F452    61                  POPAD

0049F453    C9                  LEAVE

0049F454    C2 0800             RETN 8

……

004A61B9    FFB5 F6BD4000       PUSH DWORD PTR SS:[EBP+40BDF6]           ; 重定位值

004A61BF    FFB5 DEBD4000       PUSH DWORD PTR SS:[EBP+40BDDE]           ; hmodule

004A61C5    E8 9594FFFF         CALL 0049F65F                            ; 这里进去就是Load All Library

0049F65F    55                  PUSH EBP

0049F660    8BEC                MOV EBP,ESP

0049F662    60                  PUSHAD

0049F663    8B45 08             MOV EAX,DWORD PTR SS:[EBP+8]

0049F666    8B50 3C             MOV EDX,DWORD PTR DS:[EAX+3C]            ; 定位PE头

0049F669    03C2                ADD EAX,EDX

0049F66B    8B90 80000000       MOV EDX,DWORD PTR DS:[EAX+80]            ; 定位输入表

0049F671    0355 08             ADD EDX,DWORD PTR SS:[EBP+8]

0049F674    8BFA                MOV EDI,EDX

0049F676    8B5F 0C             MOV EBX,DWORD PTR DS:[EDI+C]

0049F679    83FB 00             CMP EBX,0

0049F67C    74 1A               JE SHORT 0049F698                        ; 如果全部DLL加载完则跳去结束处

0049F67E    035D 08             ADD EBX,DWORD PTR SS:[EBP+8]

0049F681    8B5F 10             MOV EBX,DWORD PTR DS:[EDI+10]

0049F684    035D 08             ADD EBX,DWORD PTR SS:[EBP+8]             ; 获取DLL的名称

0049F687    8BF3                MOV ESI,EBX

0049F689    56                  PUSH ESI                                 ; /push FileName

0049F68A    8B5D 0C             MOV EBX,DWORD PTR SS:[EBP+C]             ; |

0049F68D    FF93 D1BA4000       CALL DWORD PTR DS:[EBX+40BAD1]           ; \LoadLibraryA

0049F693    83C7 14             ADD EDI,14

0049F696  ^ EB DE               JMP SHORT 0049F676                       ; 跳去取下一个dll名

0049F698    61                  POPAD

0049F699    C9                  LEAVE

0049F69A    C2 0800             RETN 8

……

004A7780    CC                  INT3                                     ; 再下来这里就异常了,记得上面我们保存下来的代码吗?

是了，把从4A7781处到4A9A04处的代码还原正确的回去。

004A7781    33C9                XOR ECX,ECX

004A7783    8B9D FEC14000       MOV EBX,DWORD PTR SS:[EBP+40C1FE]        ; 还原API的地址[4AA1FE]=004AAA00

004A7789    EB 15               JMP SHORT 004A77A0

004A778B    FF348B              PUSH DWORD PTR DS:[EBX+ECX\*4]            ; 471cec

004A778E    FFB5 F6BD4000       PUSH DWORD PTR SS:[EBP+40BDF6]           ; relo =9e000

004A7794    FFB5 DEBD4000       PUSH DWORD PTR SS:[EBP+40BDDE]           ; pe header =400000

004A779A    E8 DE78FFFF         CALL 0049F07D                            ; Fill Api Function

0049F07D    55                  PUSH EBP

0049F07E    8BEC                MOV EBP,ESP

0049F080    83C4 EC             ADD ESP,-14
```
