
```
0049AC2A  .|  DD1C24       |FSTP QWORD PTR SS:[ESP]                  ; |Arg1 (8-byte)  

0049AC2D  .|  9B           |WAIT                                     ; |  

0049AC2E  .|  8D55 C0      |LEA EDX,DWORD PTR SS:[EBP-40]            ; |  

0049AC31  .|  B8 B4B14900  |MOV EAX,Esend.0049B1B4                  ; |ASCII "hhnnss"  

0049AC36  .|  E8 3DFFF6FF  |CALL Esend.0040AB78                     ; \\Esend.0040AB78  

0049AC3B  .|  8B45 C0      |MOV EAX,DWORD PTR SS:[EBP-40]            ;  取当前时间 AX,DW  

0049AC3E  .|  E8 35E2F6FF  |CALL Esend.00408E78                     ;  时间转换为十六进制，

eax=0002F911  

0049AC43  .|  2BC7         |SUB EAX,EDI                              ;  eax 与 edi 相减，即

eax=0002F911-0002F775=19c    I  

0049AC45  .|  83F8 0A      |CMP EAX,0A                               ;  eax 与 0a 是否相等  

0049AC48      0F8F 19050000 JG Esend.0049B167                      ;  如果 eax 大于 0a 则跳走  

0049AC4E  .|  EB 1F        |JMP SHORT Esend.0049AC6F  

0049AC50  |>  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]  

0049AC53  .|  E8 C092F6FF  |CALL Esend.00403F18  

0049AC58  .|  3BD8         |CMP EBX,EAX  

0049AC5A  .|  7D 13        |JGE SHORT Esend.0049AC6F  

0049AC5C  .|  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]  

0049AC5F  .|  50           |PUSH EAX  

0049AC60  .|  B9 01000000  |MOV ECX,1  

0049AC65  .|  8BD3         |MOV EDX,EBX  

0049AC67  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]  

0049AC6A  .|  E8 B194F6FF  |CALL Esend.00404120  

0049AC6F  |>  43           |INC EBX  

0049AC70  .|  81FB F5010000 |CMP EBX,1F5  

0049AC76  |.^ 75 99        \\JNZ SHORT Esend.0049AC11               ;  以上为取得时间，如两时间

相差 0a，则跳走，所以应在下面断                I?  

0049AC78  .|  BB 01000000  MOV EBX,1  

0049AC7D  |>  8D45 BC      /LEA EAX,DWORD PTR SS:[EBP-44]  

0049AC80  .|  50           |PUSH EAX  

0049AC81  .|  B9 01000000  |MOV ECX,1  

0049AC86  .|  8BD3         |MOV EDX,EBX  

0049AC88  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]           ;  注册码入 eax  

0049AC8B  .|  E8 9094F6FF  |CALL Esend.00404120                     ;  依次取注册码第一位、第二

位...  

0049AC90  .|  8B45 BC      |MOV EAX,DWORD PTR SS:[EBP-44]  

0049AC93  .|  BA C4B14900  |MOV EDX,Esend.0049B1C4                  ;  Z 入 edxD  

0049AC98  .|  E8 8B93F6FF  |CALL Esend.00404028                     ;  注册码各位与 Z 比较是否相等

404028  

0049AC9D  .|  0F87 C4040000 |JA Esend.0049B167  

0049ACA3  .|  8D45 B8      |LEA EAX,DWORD PTR SS:[EBP-48]  

0049ACA6  .|  50           |PUSH EAX  

0049ACA7  .|  B9 01000000  |MOV ECX,1  

0049ACAC  .|  8BD3         |MOV EDX,EBX  

0049ACAE  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]           ;  注册码入 eax  

0049ACB1  .|  E8 6A94F6FF  |CALL Esend.00404120                     ;  依次取注册码第一位、第二

位...  

0049ACB6  .|  8B45 B8      |MOV EAX,DWORD PTR SS:[EBP-48]  

0049ACB9  .|  BA D0B14900  |MOV EDX,Esend.0049B1D0                  ;  A 入 edx  

0049ACBE  .|  E8 6593F6FF  |CALL Esend.00404028                     ;  比较 1 与 A 是否相等 d.00404  

0049ACC3  .|  0F82 9E040000 |JB Esend.0049B167  

0049ACC9  .|  43           |INC EBX  

0049ACCA  .|  83FB 15      |CMP EBX,15                               ;  以上循环 21 次，检查注册码

是否是 A-Z 间的字母。                 I?  

0049ACCD  |.^ 75 AE        \\JNZ SHORT Esend.0049AC7D  

0049ACCF  .|  8D45 E8      LEA EAX,DWORD PTR SS:[EBP-18]  

0049ACD2  .|  8B55 FC      MOV EDX,DWORD PTR SS:[EBP-4]              ;  注册码 edx  

0049ACD5  .|  E8 5690F6FF  CALL Esend.00403D30  

0049ACDA  .|  33F6         XOR ESI,ESI  

0049ACDC  .|  BB 01000000  MOV EBX,1  

0049ACE1  |>  8D45 E4      /LEA EAX,DWORD PTR SS:[EBP-1C]  

0049ACE4  .|  50           |PUSH EAX  

0049ACE5  .|  B9 01000000  |MOV ECX,1  

0049ACEA  .|  8BD3         |MOV EDX,EBX  

0049ACEC  .|  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]           ;  注册码入 eax  

0049ACEF  .|  E8 2C94F6FF  |CALL Esend.00404120                     ;  依次取 1 位注册码  

0049ACF4  .|  BA DCB14900  |MOV EDX,Esend.0049B1DC                  ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 edx  

0049ACF9  .|  8B45 E4      |MOV EAX,DWORD PTR SS:[EBP-1C]           ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 eax  

0049ACFC  .|  E8 0395F6FF  |CALL Esend.00404204                     ;  检查注册码第 N 位在上面串的

位置，位置数入 eax                    I?  

0049AD01  .|  8BF8         |MOV EDI,EAX                              ;  eax 入 edi,  

0049AD03  .|  4F           |DEC EDI                                  ;  edi 减 1D  

0049AD04  .|  0FAFFB       |IMUL EDI,EBX                             ;  edi 乘注册码第 N 位 BX  

0049AD07  .|  03F7         |ADD ESI,EDI                              ;  把结果相加 SI,ED  

0049AD09  .|  43           |INC EBX  

0049AD0A  .|  83FB 14      |CMP EBX,14                               ;  循环 20 次 BX,  

0049AD0D  |.^ 75 D2        \\JNZ SHORT Esend.0049ACE1                ;  (第 N 位注册码在密码串中的

位置-1)乘注册码第 N 位，各位相加，并入 esi    I?  

0049AD0F  .|  8BC6         MOV EAX,ESI                                ;  把 esi 入 eaxSI  

0049AD11  .|  B9 1A000000  MOV ECX,1A                                 ;  1A 入 ecx,  

0049AD16  .|  99           CDQ  

0049AD17  .|  F7F9         IDIV ECX                                   ;  ESI 除 1A,余数入 edx  

0049AD19  .|  42           INC EDX                                    ;  余数 edx 增 1  

0049AD1A  .|  8BF2         MOV ESI,EDX                                ;  edx 入 esiE  

0049AD1C  .|  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD1F  .|  50           PUSH EAX  

0049AD20  .|  B8 DCB14900  MOV EAX,Esend.0049B1DC                     ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 eax  

0049AD25  .|  B9 01000000  MOV ECX,1  

0049AD2A  .|  8BD6         MOV EDX,ESI                                ;  上面计算的模数减 1 的值入

edx  

0049AD2C  .|  E8 EF93F6FF  CALL Esend.00404120                       ;  取第 edx 位在密码串中的值

04120  

0049AD31  .|  8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]  

0049AD34  .|  50           PUSH EAX  

0049AD35  .|  B9 01000000  MOV ECX,1  

0049AD3A  .|  BA 14000000  MOV EDX,14  

0049AD3F  .|  8B45 E8      MOV EAX,DWORD PTR SS:[EBP-18]             ;  注册码入 eaxDWOR  

0049AD42  .|  E8 D993F6FF  CALL Esend.00404120                       ;  取注册码最后一位 d.004041  

0049AD47  .|  8B55 B4      MOV EDX,DWORD PTR SS:[EBP-4C]             ;  注册码最后一位入 edxD PTR 

SS  

0049AD4A  .|  8B45 E4      MOV EAX,DWORD PTR SS:[EBP-1C]             ;  上面在密码表中取得的值入

eaxR SS:[EBP-1C  

0049AD4D  .|  E8 D692F6FF  CALL Esend.00404028                       ;  比较是否相等 end.00  

0049AD52      0F85 0F040000 JNZ Esend.0049B167  

0049AD58  .|  33F6         XOR ESI,ESI  

0049AD5A  .|  BB 01000000  MOV EBX,1  

0049AD5F  |>  8D45 E4      /LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD62  .|  50           |PUSH EAX  

0049AD63  .|  B9 01000000  |MOV ECX,1  

0049AD68  .|  8BD3         |MOV EDX,EBX  

0049AD6A  .|  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]           ;  注册码入 eax  

0049AD6D  .|  E8 AE93F6FF  |CALL Esend.00404120  

0049AD72  .|  BA DCB14900  |MOV EDX,Esend.0049B1DC                  ;  ASCII 

"GFEDCBANMLKJIHTSRQPOZYXWVU"  

0049AD77  .|  8B45 E4      |MOV EAX,DWORD PTR SS:[EBP-1C]  

0049AD7A  .|  E8 8594F6FF  |CALL Esend.00404204  

0049AD7F  .|  8BF8         |MOV EDI,EAX  

0049AD81  .|  4F           |DEC EDI  

0049AD82  .|  03F7         |ADD ESI,EDI  

0049AD84  .|  43           |INC EBX  

0049AD85  .|  83FB 13      |CMP EBX,13  

0049AD88  |.^ 75 D5        \\JNZ SHORT Esend.0049AD5F                ;  注册码前面 19 位按上面生成

一值  

0049AD8A  .|  8BC6         MOV EAX,ESI  

0049AD8C  .|  B9 1A000000  MOV ECX,1A  

0049AD91  .|  99           CDQ  

0049AD92  .|  F7F9         IDIV ECX  

0049AD94  .|  42           INC EDX  

0049AD95  .|  8BF2         MOV ESI,EDX  

0049AD97  .|  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD9A  .|  50           PUSH EAX  

0049AD9B  .|  B8 DCB14900  MOV EAX,Esend.0049B1DC                    ;  ASCII 

"GFEDCBANMLKJIHTSRQPOZYXWVU"  
```
