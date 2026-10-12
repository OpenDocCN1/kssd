
# Audio To Video Mixer 3.0.145

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-239.htm`](https://www.pediy.com/kssd/pediy08/pediy8-239.htm)

【软件名称】`Audio To Video Mixer 3.0.145`

【软件类别】国外软件/共享版/视频工具

【运行环境】`Win9x/Me/NT/2000/XP`

【保护方式】用户名、注册码

【作者声明】初学 `Crack`，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。

【下载地址】`http://www.onlinedown.net/soft/39614.htm`

【软件信息】视频编辑工具，用于将音频文件混合入视频中。能将你家庭视频的原音消除，加入背景音乐，解说或更多的音轨。

## 一、侦察敌情

`PEiD` 检查：`Microsoft Visual C++ 7.0`

输入用户名：`wzwgp`，注册码：`12345678`，提示“`Registration failed!`”

## 二、追码

`od` 载入，超级字串参考：地址=`00416CB2` 文本字串=`registration failed!`

根据提示信息来到 `004169F3` 处下断，

```
004169F3   .  55             PUSH EBP               ; F2
004169F4   .  56             PUSH ESI
004169F5   .  57             PUSH EDI
004169F6   .  BF 01000000    MOV EDI,1
004169FB   .  57             PUSH EDI
004169FC   .  8BF1           MOV ESI,ECX
004169FE   .  E8 BDB40100    CALL a2vmixer.00431EC0
00416A03   .  8B46 70        MOV EAX,DWORD PTR DS:[ESI+70]    ; 用户名地址入 EAX
00416A06   .  8B68 F4        MOV EBP,DWORD PTR DS:[EAX-C]     ; [EAX-C]=用户名位数
00416A09   .  83FD 02       CMP EBP,2                   ; 用户名位数要大于 2
00416A0C   .  7D 15         JGE SHORT a2vmixer.00416A23
00416A0E   .  6A 00         PUSH 0
00416A10   .  6A 00         PUSH 0
00416A12   .  68 DC874400   PUSH a2vmixer.004487DC          ; please input correct user name!
00416A17   .  E8 A4510200   CALL a2vmixer.0043BBC0
00416A1C   .  5F           POP EDI
00416A1D   .  5E           POP ESI
00416A1E   .  5D           POP EBP
00416A1F   .  83C4 0C       ADD ESP,0C
00416A22   .  C3           RETN

00416A23   >  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]    ; 注册码地址入 ECX
00416A26   .  8379 F4 08    CMP DWORD PTR DS:[ECX-C],8        ; 注册码位数要大于 8
00416A2A   .  7D 15         JGE SHORT a2vmixer.00416A41
00416A2C   .  6A 00         PUSH 0
00416A2E   .  6A 00         PUSH 0
00416A30   .  68 B4874400   PUSH a2vmixer.004487B4          ; please input correct registration code!please input
00416A35   .  E8 86510200   CALL a2vmixer.0043BBC0
00416A3A   .  5F           POP EDI
00416A3B   .  5E           POP ESI
00416A3C   .  5D           POP EBP
00416A3D   .  83C4 0C       ADD ESP,0C
00416A40   .  C3           RETN

00416A41   >  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00416A44   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]
00416A47   .  85C9          TEST ECX,ECX
00416A49   .  7D 0A         JGE SHORT a2vmixer.00416A55
00416A4B   .  68 57000780   PUSH 80070057
00416A50   .  E8 ABA8FEFF   CALL a2vmixer.00401300

00416A55   >  8A10          MOV DL,BYTE PTR DS:[EAX]         ; 取第一位用户名
00416A57   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00416A5A   .  3978 F4       CMP DWORD PTR DS:[EAX-C],EDI
00416A5D   .  7D 0A         JGE SHORT a2vmixer.00416A69
00416A5F   .  68 57000780   PUSH 80070057
00416A64   .  E8 97A8FEFF   CALL a2vmixer.00401300

00416A69   >  8A40 01       MOV AL,BYTE PTR DS:[EAX+1]       ; 取第二位用户名
00416A6C   .  884424 0E     MOV BYTE PTR SS:[ESP+E],AL
00416A70   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00416A73   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]
00416A76   .  85C9          TEST ECX,ECX
00416A78   .  7D 0A         JGE SHORT a2vmixer.00416A84
00416A7A   .  68 57000780   PUSH 80070057
00416A7F   .  E8 7CA8FEFF   CALL a2vmixer.00401300

00416A84   >  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+70]
00416A87   .  53           PUSH EBX
00416A88   .  8A18          MOV BL,BYTE PTR DS:[EAX]         ; 取第一位用户名
00416A8A   .  3979 F4       CMP DWORD PTR DS:[ECX-C],EDI
00416A8D   .  7D 0A         JGE SHORT a2vmixer.00416A99
00416A8F   .  68 57000780   PUSH 80070057
00416A94   .  E8 67A8FEFF   CALL a2vmixer.00401300

00416A99   >  0FB6C2        MOVZX EAX,DL                     ; DL=77
00416A9C   .  83C8 41       OR EAX,41                        ; EAX=77 or 41=77
00416A9F   .  99           CDQ
00416AA0   .  BF 0A000000   MOV EDI,0A
00416AA5   .  F7FF          IDIV EDI                         ; EAX=77 idiv 0A=B,9
00416AA7   .  0FB64424 12   MOVZX EAX,BYTE PTR SS:[ESP+12]   ; 取第二位用户名(7A)
00416AAC   .  83C8 32       OR EAX,32                        ; EAX=7A or 32 =7A
00416AAF   .  885424 16     MOV BYTE PTR SS:[ESP+16],DL      ; 保存余数
00416AB3   .  99           CDQ
00416AB4   .  F7FF          IDIV EDI                         ; EAX=7A idiv A=C,2
00416AB6   .  0FB6C3        MOVZX EAX,BL                     ; 取第一位用户名
00416AB9   .  83C8 56       OR EAX,56                        ; EAX=77 or 56=77
00416ABC   .  885424 12     MOV BYTE PTR SS:[ESP+12],DL
00416AC0   .  99           CDQ
00416AC1   .  F7FF          IDIV EDI                         ; EAX=77 idiv A=B,9
00416AC3   .  0FB641 01     MOVZX EAX,BYTE PTR DS:[ECX+1]    ; 取第二位用户名
00416AC7   .  83C8 4D       OR EAX,4D                        ; EAX=7A or 4D =7F
00416ACA   .  8BCF          MOV ECX,EDI
00416ACC   .  885424 17     MOV BYTE PTR SS:[ESP+17],DL
00416AD0   .  99           CDQ
00416AD1   .  F7F9          IDIV ECX                         ; EAX=7F idiv A=C,7
00416AD3   .  33C0          XOR EAX,EAX
00416AD5   .  33C9          XOR ECX,ECX
00416AD7   .  85ED          TEST EBP,EBP
00416AD9   .  885424 18     MOV BYTE PTR SS:[ESP+18],DL
00416ADD   .  7E 20         JLE SHORT a2vmixer.00416AFF
00416ADF   .  90           NOP

00416AE0   >  85C9          TEST ECX,ECX
00416AE2   .  0F8C D2000000 JL a2vmixer.00416BBA
00416AE8   .  8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]
00416AEB   .  3B4F F4       CMP ECX,DWORD PTR DS:[EDI-C]
00416AEE   .  0F8F C6000000 JG a2vmixer.00416BBA
00416AF4   .  0FB6140F      MOVZX EDX,BYTE PTR DS:[EDI+ECX]
00416AF8   .  03C2          ADD EAX,EDX                      ; 用户名逐位累加＝23F
00416AFA   .  41           INC ECX                          ; ECX=计数器
00416AFB   .  3BCD          CMP ECX,EBP                      ; 取完没有？
00416AFD   .  7C E1         JL SHORT a2vmixer.00416AE0

00416AFF   >  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]    ; 注册码入 ECX
00416B02   .  8B51 F4       MOV EDX,DWORD PTR DS:[ECX-C]     ; 注册码位数入 EDX
00416B05   .  85D2          TEST EDX,EDX
00416B07   .  7D 0A         JGE SHORT a2vmixer.00416B13
00416B09   .  68 57000780   PUSH 80070057
00416B0E   .  E8 EDA7FEFF   CALL a2vmixer.00401300

00416B13   >  8A11          MOV DL,BYTE PTR DS:[ECX]         ; 取第一位注册码
00416B15   .  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]
00416B18   .  8379 F4 01    CMP DWORD PTR DS:[ECX-C],1
00416B1C   .  885424 19     MOV BYTE PTR SS:[ESP+19],DL
00416B20   .  7D 0A         JGE SHORT a2vmixer.00416B2C
00416B22   .  68 57000780   PUSH 80070057
00416B27   .  E8 D4A7FEFF   CALL a2vmixer.00401300

00416B2C   >  8A49 01       MOV CL,BYTE PTR DS:[ECX+1]       ; 取第二位注册码
00416B2F   .  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]
00416B32   .  884C24 13     MOV BYTE PTR SS:[ESP+13],CL
00416B36   .  837F F4 02    CMP DWORD PTR DS:[EDI-C],2
00416B3A   .  7D 0A         JGE SHORT a2vmixer.00416B46
00416B3C   .  68 57000780   PUSH 80070057
00416B41   .  E8 BAA7FEFF   CALL a2vmixer.00401300

00416B46   >  8A4F 02       MOV CL,BYTE PTR DS:[EDI+2]       ; 取第三位注册码
00416B49   .  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]
00416B4C   .  884C24 14     MOV BYTE PTR SS:[ESP+14],CL
00416B50   .  837F F4 03    CMP DWORD PTR DS:[EDI-C],3
00416B54   .  7D 0A         JGE SHORT a2vmixer.00416B60
00416B56   .  68 57000780   PUSH 80070057
00416B5B   .  E8 A0A7FEFF   CALL a2vmixer.00401300

00416B60   >  8A4F 03       MOV CL,BYTE PTR DS:[EDI+3]       ; 取第四位注册码
00416B63   .  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]
00416B66   .  884C24 15     MOV BYTE PTR SS:[ESP+15],CL
```



## 三、算法小结

1. 用户名 2 位以上，注册码要求 8 位以上。
2. 只取用户名的前两位 16 进制数进行运算二次，得到注册码前 4 位。
3. 用户名的 16 进制数累加和进行运算，得到注册码的第 5 位。
4. 只验证注册码前 5 位，后 3 位可以任意数。
5. 或是固定注册码：`12807835`

## 四、算法验证

1. 用户名：`wzwgp`(0x777A776770)
2. `77 or 41 = 77` `idiv A = B,9`
   `7A or 32 = 7A` `idiv A = C,2`
   `77 or 56 = 77` `idiv A = B,9`
   `7A or 4D = 7F` `idiv A = C,7`
   注册码前 4 位：`9 2 9 7`
3. 用户名的 16 进制数累加和: `77+7A+77+67+70=23F`
   `23F idiv A = 39,5`
   注册码第 5 位：`5`
4. 用户名：`wzwgp`
   注册码：`92975***` 或 `12807835`

注册信息保存在：`HKEY_CURRENT_USER\Software\DigitByteStudio\a2vmixer\Option`

4dd466d0e1e0cc123afbe504da2462ca

```assembly
00416B6A   .  837F F4 04     CMP DWORD PTR DS:[EDI-C],4
00416B6E   .  7D 0A          JGE SHORT a2vmixer.00416B7A
00416B70   .  68 57000780    PUSH 80070057
00416B75   .  E8 86A7FEFF    CALL a2vmixer.00401300
00416B7A   >  8A4F 04        MOV CL,BYTE PTR DS:[EDI+4]       ;  取第五位注册码
00416B7D   .  8B7E 74        MOV EDI,DWORD PTR DS:[ESI+74]
00416B80   .  837F F4 05     CMP DWORD PTR DS:[EDI-C],5
00416B84   .  7D 0A          JGE SHORT a2vmixer.00416B90
00416B86   .  68 57000780    PUSH 80070057
00416B8B   .  E8 70A7FEFF    CALL a2vmixer.00401300
00416B90   >  8A5F 05        MOV BL,BYTE PTR DS:[EDI+5]       ;  取第六位注册码
00416B93   .  8B7E 74        MOV EDI,DWORD PTR DS:[ESI+74]
00416B96   .  885C24 1A      MOV BYTE PTR SS:[ESP+1A],BL
00416B9A   .  837F F4 06     CMP DWORD PTR DS:[EDI-C],6
00416B9E   .  7D 0A          JGE SHORT a2vmixer.00416BAA
00416BA0   .  68 57000780    PUSH 80070057
00416BA5   .  E8 56A7FEFF    CALL a2vmixer.00401300
00416BAA   >  8A5F 06        MOV BL,BYTE PTR DS:[EDI+6]       ;  取第七位注册码
00416BAD   .  8B7E 74        MOV EDI,DWORD PTR DS:[ESI+74]
00416BB0   .  885C24 1B      MOV BYTE PTR SS:[ESP+1B],BL
00416BB4   .  837F F4 07     CMP DWORD PTR DS:[EDI-C],7
00416BB8   .  7D 0A          JGE SHORT a2vmixer.00416BC4
00416BBA   >  68 57000780    PUSH 80070057
00416BBF   .  E8 3CA7FEFF    CALL a2vmixer.00401300
00416BC4   >  8A5F 07        MOV BL,BYTE PTR DS:[EDI+7]       ;  取第八位注册码
00416BC7   .  0FB67C24 16    MOVZX EDI,BYTE PTR SS:[ESP+16]
00416BCC   .  0FB6D2         MOVZX EDX,DL
00416BCF   .  83EA 30        SUB EDX,30
00416BD2   .  3BFA           CMP EDI,EDX                      ;  判断第一位注册码
00416BD4   .  75 48          JNZ SHORT a2vmixer.00416C1E
00416BD6   .  0FB65424 13    MOVZX EDX,BYTE PTR SS:[ESP+13]
00416BDB   .  0FB67C24 12    MOVZX EDI,BYTE PTR SS:[ESP+12]
00416BE0   .  83EA 30        SUB EDX,30
00416BE3   .  3BFA           CMP EDI,EDX                      ;  判断第二位注册码
00416BE5   .  75 37          JNZ SHORT a2vmixer.00416C1E
00416BE7   .  0FB65424 14    MOVZX EDX,BYTE PTR SS:[ESP+14]
00416BEC   .  0FB67C24 17    MOVZX EDI,BYTE PTR SS:[ESP+17]
00416BF1   .  83EA 30        SUB EDX,30
00416BF4   .  3BFA           CMP EDI,EDX                      ;  判断第三位注册码
00416BF6   .  75 26          JNZ SHORT a2vmixer.00416C1E
00416BF8   .  0FB65424 15    MOVZX EDX,BYTE PTR SS:[ESP+15]
00416BFD   .  0FB67C24 18    MOVZX EDI,BYTE PTR SS:[ESP+18]
00416C02   .  83EA 30        SUB EDX,30
00416C05   .  3BFA           CMP EDI,EDX                      ;  判断第四位注册码
00416C07   .  75 15          JNZ SHORT a2vmixer.00416C1E
00416C09   .  99             CDQ
00416C0A   .  BF 0A000000    MOV EDI,0A
00416C0F   .  F7FF           IDIV EDI                         ;  EAX=23F idiv A=39,5
00416C11   .  0FB6C2         MOVZX EAX,DL                     ;  DL=5
00416C14   .  0FB6D1         MOVZX EDX,CL
00416C17   .  83EA 30        SUB EDX,30
00416C1A   .  3BC2           CMP EAX,EDX                      ;  判断第五位注册码
00416C1C   .  74 3A          JE SHORT a2vmixer.00416C58
00416C1E   >  807C24 19 31   CMP BYTE PTR SS:[ESP+19],31      ;  1
00416C23   .  0F85 85000000  JNZ a2vmixer.00416CAE
00416C29   .  807C24 13 32   CMP BYTE PTR SS:[ESP+13],32      ;  2
00416C2E   .  75 7E          JNZ SHORT a2vmixer.00416CAE
00416C30   .  8A5424 14      MOV DL,BYTE PTR SS:[ESP+14]
00416C34   .  B0 38          MOV AL,38
00416C36   .  3AD0           CMP DL,AL                        ;  8
00416C38   .  75 74          JNZ SHORT a2vmixer.00416CAE
00416C3A   .  807C24 15 30   CMP BYTE PTR SS:[ESP+15],30      ;  0
00416C3F   .  75 6D          JNZ SHORT a2vmixer.00416CAE
00416C41   .  80F9 37        CMP CL,37                        ;  7
00416C44   .  75 68          JNZ SHORT a2vmixer.00416CAE
00416C46   .  384424 1A      CMP BYTE PTR SS:[ESP+1A],AL      ;  8
00416C4A   .  75 62          JNZ SHORT a2vmixer.00416CAE
00416C4C   .  807C24 1B 33   CMP BYTE PTR SS:[ESP+1B],33      ;  3
00416C51   .  75 5B          JNZ SHORT a2vmixer.00416CAE
00416C53   .  80FB 35        CMP BL,35                        ;  5
00416C56   .  75 56          JNZ SHORT a2vmixer.00416CAE
00416C58   >  6A 00          PUSH 0
00416C5A   .  6A 00          PUSH 0
00416C5C   .  68 98874400    PUSH a2vmixer.00448798           ;  registration has succeeded!please input correct registration code!please input correct user name!
00416C61   .  E8 5A4F0200    CALL a2vmixer.0043BBC0
00416C66   .  8B7E 70        MOV EDI,DWORD PTR DS:[ESI+70]
00416C69   .  E8 3B8E0200    CALL a2vmixer.0043FAA9
00416C6E   .  8B40 04        MOV EAX,DWORD PTR DS:[EAX+4]
00416C71   .  57             PUSH EDI                         ; /Arg3
00416C72   .  68 547C4400    PUSH a2vmixer.00447C54           ; |username
00416C77   .  68 4C7C4400    PUSH a2vmixer.00447C4C           ; |option
00416C7C   .  8BC8           MOV ECX,EAX                      ; |
00416C7E   .  E8 43500200    CALL a2vmixer.0043BCC6           ; \\a2vmixer.0043BCC6
00416C83   .  8B7E 74        MOV EDI,DWORD PTR DS:[ESI+74]
00416C86   .  E8 1E8E0200    CALL a2vmixer.0043FAA9
00416C8B   .  8B40 04        MOV EAX,DWORD PTR DS:[EAX+4]
00416C8E   .  57             PUSH EDI                         ; /Arg3
00416C8F   .  68 387C4400    PUSH a2vmixer.00447C38           ; |registration_code
00416C94   .  68 4C7C4400    PUSH a2vmixer.00447C4C           ; |option
00416C99   .  8BC8           MOV ECX,EAX                      ; |
00416C9B   .  E8 26500200    CALL a2vmixer.0043BCC6           ; \\a2vmixer.0043BCC6
00416CA0   .  5B             POP EBX
00416CA1   .  5F             POP EDI
00416CA2   .  8BCE           MOV ECX,ESI
00416CA4   .  5E             POP ESI
00416CA5   .  5D             POP EBP
00416CA6   .  83C4 0C        ADD ESP,0C
00416CA9   .  E9 A4F10100    JMP a2vmixer.00435E52
00416CAE   >  6A 00          PUSH 0
00416CB0   .  6A 00          PUSH 0
00416CB2   .  68 80874400    PUSH a2vmixer.00448780           ;  registration failed!
00416CB7   .  E8 044F0200    CALL a2vmixer.0043BBC0
00416CBC   .  5B             POP EBX
00416CBD   .  5F             POP EDI
00416CBE   .  5E             POP ESI
00416CBF   .  5D             POP EBP
00416CC0   .  83C4 0C        ADD ESP,0C
00416CC3   .  C3             RETN
```
