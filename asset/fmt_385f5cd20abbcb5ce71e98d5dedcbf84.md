
### 汇编代码分析

```
004060D6  |. 8BD7              MOV EDX,EDI                       ; [ EDX = EDI = 0000000A ]
004060D8  |. 6A 03             PUSH 3
004060DA  |. 03D1              ADD EDX,ECX                       ; [ EDX = EDX +ECX ]
004060DC  |. B9 19000000       MOV ECX,19                        ; [ ECX = 19h ]
004060E1  |. 03C2              ADD EAX,EDX                       ; [ EAX = EAX + EDX ]
004060E3  |. C64424 1B 00      MOV BYTE PTR SS:[ESP+1B],0
004060E8  |. 99                CDQ                               ; [ EAX 符号扩展为 64 位 EDX:EAX ]
                                                  ; [ 此处 CDQ 的作用实际就是将 EDX 清零 ]
                                                  ; ( 不知道我的理解对不对，望高手指点。)
004060E9  |. F7F9              IDIV ECX                          ; [ EAX 除以 ECX 商存入 EAX ，余数存入 EDX ]
004060EB  |. 0FBE46 01        MOVSX EAX,BYTE PTR DS:[ESI+1]      ; [ 将第一部分的第二个数字的 ASCII 码给 EAX ]
004060EF  |. 8BCF              MOV ECX,EDI                       ; [ ECX = EDI ]
004060F1  |. 80C2 41           ADD DL,41                         ; [ DL = DL +41h ]
004060F4  |. 885424 18         MOV BYTE PTR SS:[ESP+18],DL       ; [ 将 DL 值存入 SS:[ESP+18] ]
                                                  ; [ 这就是注册码第二部分的第一位 ]
                                                  ; [ 接下来是计算注册码第二部分的第二、三位]
                                                  ; [ 算法与此相同，不再赘述 ]
004060F8  |. 0FBE56 04        MOVSX EDX,BYTE PTR DS:[ESI+4]
004060FC  |. 03CA              ADD ECX,EDX
004060FE  |. 03C1              ADD EAX,ECX
00406100  |. B9 19000000       MOV ECX,19
00406105  |. 99                CDQ
00406106  |. F7F9              IDIV ECX
00406108  |. 0FBE46 02        MOVSX EAX,BYTE PTR DS:[ESI+2]
0040610C  |. 8BCF              MOV ECX,EDI
0040610E  |. 80C2 41           ADD DL,41
00406111  |. 885424 19         MOV BYTE PTR SS:[ESP+19],DL
00406115  |. 0FBE56 05        MOVSX EDX,BYTE PTR DS:[ESI+5]
00406119  |. 03CA              ADD ECX,EDX
0040611B  |. 03C1              ADD EAX,ECX
0040611D  |. B9 19000000       MOV ECX,19
00406122  |. 99                CDQ
00406123  |. F7F9              IDIV ECX
00406125  |. 80C2 41           ADD DL,41
00406128  |. 885424 1A         MOV BYTE PTR SS:[ESP+1A],DL

; --------------- 计算第二部分的前三个字母结束 -------------

0040612C  |. 8D5424 18         LEA EDX,DWORD PTR SS:[ESP+18]
00406130  |. 52                PUSH EDX
00406131  |. 53                PUSH EBX
00406132  |. 8B1D 8C964100     MOV EBX,DWORD PTR DS:[<&MSVCRT.strncpy>]
00406138  |. FFD3              CALL EBX                          ; [ 将这三个字母接在 0012F8A0 中部分注册码的后面 ]

; --------------- 计算第二部分的中间三个字母开始 --------------

0040613A  |. 0FBE46 03        MOVSX EAX,BYTE PTR DS:[ESI+3]      ; [ 将第一部分的第四个数字的 ASCII 码给 EAX ]
0040613E  |. 0FBE0E           MOVSX ECX,BYTE PTR DS:[ESI]        ; [ 将第一部分的第一个数字的 ASCII 码给 ECX ]
00406141  |. 2BC8              SUB ECX,EAX                       ; [ ECX = ECX - EAX ]
00406143  |. 6A 03             PUSH 3
00406145  |. 8D4439 20         LEA EAX,DWORD PTR DS:[ECX+EDI+20] ; [ EAX = ECX + EDI + 20h ]
00406149  |. B9 19000000       MOV ECX,19                        ; [ ECX = 19h ]
0040614E  |. 99                CDQ                               ; [ 将 EDX 清零 ]
0040614F  |. F7F9              IDIV ECX                          ; [ EAX 除以 ECX 商存入 EAX ，余数存入 EDX ]
00406151  |. 0FBE46 04        MOVSX EAX,BYTE PTR DS:[ESI+4]      ; [ 将第一部分的第五个数字的 ASCII 码给 EAX ]
00406155  |. 80C2 41           ADD DL,41                         ; [ DL = DL + 41h ]
00406158  |. 885424 24         MOV BYTE PTR SS:[ESP+24],DL       ; [ 将 DL 值存入 DS:[ESI+24] ]
                                                  ; [ 这就是注册码第二部分的第四位 ]
                                                  ; [ 接下来是计算注册码第二部分的第五、六位]
                                                  ; [ 算法与此相同，不再赘述 ]
0040615C  |. 0FBE56 01        MOVSX EDX,BYTE PTR DS:[ESI+1]
00406160  |. 2BD0              SUB EDX,EAX
00406162  |. 8D443A 20         LEA EAX,DWORD PTR DS:[EDX+EDI+20]
00406166  |. 99                CDQ
00406167  |. F7F9              IDIV ECX
00406169  |. 0FBE46 05        MOVSX EAX,BYTE PTR DS:[ESI+5]
0040616D  |. 80C2 41           ADD DL,41
00406170  |. 885424 25         MOV BYTE PTR SS:[ESP+25],DL
00406174  |. 0FBE56 02        MOVSX EDX,BYTE PTR DS:[ESI+2]
00406178  |. 2BD0              SUB EDX,EAX
0040617A  |. 8D443A 20         LEA EAX,DWORD PTR DS:[EDX+EDI+20]
0040617E  |. 99                CDQ
0040617F  |. F7F9              IDIV ECX
00406181  |. 8D45 0A           LEA EAX,DWORD PTR SS:[EBP+A]
00406184  |. 80C2 41           ADD DL,41
00406187  |. 885424 26         MOV BYTE PTR SS:[ESP+26],DL

; --------------- 计算第二部分的中间三个字母结束 -----------------

0040618B  |. 8D5424 24         LEA EDX,DWORD PTR SS:[ESP+24]
0040618F  |. 52                PUSH EDX
00406190  |. 50                PUSH EAX
00406191  |. FFD3              CALL EBX                          ; [ 将这三个字母接在 0012F8A0 中部分注册码的后面 ]

; --------------- 计算第二部分的后三个字母开始 -------------------

00406193  |. 0FBE46 03        MOVSX EAX,BYTE PTR DS:[ESI+3]      ; [ 将第一部分的第四个数字的 ASCII 码给 EAX ]
00406197  |. 0FBE0E           MOVSX ECX,BYTE PTR DS:[ESI]        ; [ 将第一部分的第一个数字的 ASCII 码给 ECX ]
0040619A  |. 03C7              ADD EAX,EDI                       ; [ EAX = EAX + EDI ]
0040619C  |. 33C1              XOR EAX,ECX                       ; [ EAX = EAX XOR ECX ]
0040619E  |. B9 19000000       MOV ECX,19                        ; [ ECX = 19h ]
004061A3  |. 99                CDQ                               ; [ 将 EDX 清零 ]
004061A4  |. F7F9              IDIV ECX                          ; [ EAX 除以 ECX 商存入 EAX ，余数存入 EDX ]
004061A6  |. 0FBE46 04        MOVSX EAX,BYTE PTR DS:[ESI+4]      ; [ 将第一部分的第五个数字的 ASCII 码给 EAX ]
004061AA  |. 80C2 41           ADD DL,41                         ; [ DL = DL + 41h ]
004061AD  |. 03C7              ADD EAX,EDI                       ; [ EAX = EAX + EDI ]
004061AF  |. 885424 2C         MOV BYTE PTR SS:[ESP+2C],DL       ; [ DL 存入 SS:[ESP+2C] ]
                                                  ; [ 这就是注册码第二部分的第七位 ]
                                                  ; [ 接下来是计算注册码第二部分的第八、九位]
                                                  ; [ 算法与此相同，不再赘述 ]
004061B3  |. 0FBE56 01        MOVSX EDX,BYTE PTR DS:[ESI+1]
004061B7  |. 33C2              XOR EAX,EDX
004061B9  |. 6A 03             PUSH 3
004061BB  |. 99                CDQ
004061BC  |. F7F9              IDIV ECX
004061BE  |. 0FBE46 05        MOVSX EAX,BYTE PTR DS:[ESI+5]
004061C2  |. 03C7              ADD EAX,EDI
004061C4  |. 80C2 41           ADD DL,41
004061C7  |. 885424 31         MOV BYTE PTR SS:[ESP+31],DL
004061CB  |. 0FBE56 02        MOVSX EDX,BYTE PTR DS:[ESI+2]
004061CF  |. 33C2              XOR EAX,EDX
004061D1  |. 99                CDQ
004061D2  |. F7F9              IDIV ECX
```
