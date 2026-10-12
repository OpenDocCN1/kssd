
## 汇编代码分析

```
004231EE  INC      ECX                                 ; //进一位  
004231EF  MOV      EDI, EDX                            ; EDI=EDX=21  
004231F1  MOV      DL, BYTE PTR DS:[EAX+EBP]           
004231F4  MOV      BYTE PTR SS:[ESP+13], DL  
004231F8  MOV      DL, BYTE PTR DS:[EAX+EDI]           
; DL<==DS:[EAX+EDI]=C1 EAX=01260490 EDI=21  
004231FB  MOV      BYTE PTR DS:[EAX+EBP], DL           
004231FE  MOV      EAX, DWORD PTR DS:[ESI+530]           
00423204  MOV      DL, BYTE PTR SS:[ESP+13]             
00423208  MOV      BYTE PTR DS:[EDI+EAX], DL           
0042320B  MOV      EAX, DWORD PTR SS:[ESP+14]           
0042320F  ADD      EAX, ECX  
00423211  CMP      EAX, EBX                            ; //计算的次数=6  
00423213  JL       SHORT 超级拖拉.004231C8              
; //用用户名计算的值做指针把试验码重新排列
00423215   
  DEC      EBX  
00423216  TEST     EBX, EBX  
00423218  MOV      DWORD PTR SS:[ESP+18], EBX  
0042321C  JLE      SHORT 超级拖拉.00423289  
0042321E  MOV      EAX, -4F9                           ; EAX=-4F9  
00423223  LEA      EDI, DWORD PTR DS:[ESI+4F9]         
; EDI<== DS:[ESI+4F9]=00470D61,(ASCII "xyang")
00423229  SUB      EAX, ESI  
0042322B  MOV      DWORD PTR SS:[ESP+14], EAX          ; EAX=FFB8F29F  
0042322F  XOR      ECX, ECX  
00423231  XOR      EDX, EDX  
00423233  MOV      CL, BYTE PTR DS:[EDI-1]             ; CL=DS:[470D60]=66 ('f')  
00423236  MOV      EBX, 29                             ; EBX=29  
0042323B  MOV      EBP, ECX                            ; EBP=66  
0042323D  XOR      ECX, ECX  
0042323F  MOV      CL, BYTE PTR DS:[EDI]               ; CL=DS:[470D61]=78 ('x')  
00423241  MOV      EAX, EBP                            ; EAX=66  
00423243  SHL      EAX, 8                               ; EAX=00006600  
00423246  ADD      EAX, ECX                            ; EAX=6678  
00423248  DIV      EBX       ; EAX=6678 DIV EBX=29 ==>EAX=27F EDX=21  
0042324A  MOV      EAX, EBP                            ; EAX=66  
0042324C  NOT      EAX                                 ; EAX=FFFFFF99  
0042324E  SHL      EAX, 8                               ; EAX=FFFF9900  
00423251  NOT      ECX                                 ; ECX=FFFFFF87  
00423253  ADD      EAX, ECX                            ; EAX=FFFF9987  
00423255  MOV      ECX, 25                             ; ECX=25  
0042325A  MOV      EBX, EDX                            ; EBX=21  
0042325C  XOR      EDX, EDX  
0042325E  DIV      ECX                                 
; EAX=FFFF9887 DIV ECX=25 ==>EAX=6EB3B79 EDX=0A
00423260  MOV      EAX, DWORD PTR DS:[ESI+530]           
00423266  INC      EDI  
00423267  MOV      CL, BYTE PTR DS:[EAX+EBX]     
; CL=DS:[EAX+EBX]=C1 EAX=01260490 EBX=21
0042326A  MOV      EBP, EDX                            ; EBP=EDX=A  
0042326C  MOV      DL, BYTE PTR DS:[EAX+EBP]  ; DL=DS:[EAX+EBP]=31 ('1')   
0042326F  MOV      BYTE PTR DS:[EAX+EBX], DL  
00423272  MOV      EAX, DWORD PTR DS:[ESI+530]  
00423278  MOV      BYTE PTR DS:[EAX+EBP], CL          ; CL=C1  
0042327B  MOV      ECX, DWORD PTR SS:[ESP+14]  
0042327F  MOV      EAX, DWORD PTR SS:[ESP+18]  
00423283  ADD      ECX, EDI  
00423285  CMP      ECX, EAX  
00423287  JL       SHORT 超级拖拉.0042322F              
; //用用户名计算的值做指针把试验码重新排列  
|  
004232EC  MOV      DWORD PTR DS:[4719B8], EDI  
004232F2  MOV      ECX, DWORD PTR DS:[ESI+530]         ; ECX<==DS:[470D98]=01260490,(ASCII "78")  
004232F8  MOV      EDX, DWORD PTR DS:[ESI+52C]         ; EDX<==DS:[470D94]=01260450  
004232FE  MOV      EAX, DWORD PTR DS:[ECX]             ; EAX=DS:[1260490]=34003837  
00423300  MOV      DWORD PTR DS:[EDX], EAX  
00423302  MOV      EAX, DWORD PTR DS:[ECX+4]           
00423305  MOV      DWORD PTR DS:[EDX+4], EAX  
00423308  MOV      EAX, DWORD PTR DS:[ECX+8]  ; EAX=DS:[1260498]=32C17433  
0042330B  MOV      DWORD PTR DS:[EDX+8], EAX  
0042330E  MOV      EAX, DWORD PTR DS:[ECX+C]    ; EAX=DS:[ECX+C]=007134C0  
00423311  MOV      DWORD PTR DS:[EDX+C], EAX  
00423314  MOV      CX, WORD PTR DS:[ECX+10]          ; CX=DS:[ECX+10]=0033  
00423318  MOV      WORD PTR DS:[EDX+10], CX  
0042331C  MOV      EDX, DWORD PTR DS:[ESI+52C]  
00423322  MOV      BYTE PTR DS:[EDX+12], 0  
00423326  MOV      EAX, DWORD PTR DS:[ESI+530]           
0042332C  MOV      ECX, DWORD PTR DS:[ESI+528]  
00423332  ADD      EAX, 12  
00423335  MOV      EDX, DWORD PTR DS:[EAX]             
00423337  MOV      DWORD PTR DS:[ECX], EDX  
00423339  MOV      EDX, DWORD PTR DS:[EAX+4]    ; EDX=DS:[EAX+4]=3A733236  
0042333C  MOV      DWORD PTR DS:[ECX+4], EDX  
0042333F  MOV      EDX, DWORD PTR DS:[EAX+8]    ; EDX=DS:[EAX+8]=3275006B  
00423342  MOV      DWORD PTR DS:[ECX+8], EDX  
00423345  MOV      EDX, DWORD PTR DS:[EAX+C]   ; EDX=DS:[EAX+C]=3139FA76  
00423348  MOV      DWORD PTR DS:[ECX+C], EDX  
0042334B  MOV      AX, WORD PTR DS:[EAX+10]     ; AX=DS:[EAX+10]=3701  
0042334F  MOV      WORD PTR DS:[ECX+10], AX  
00423353  MOV      ECX, DWORD PTR DS:[ESI+528]  ; ECX=01260470  
00423359  XOR      EAX, EAX  
0042335B  MOV      BYTE PTR DS:[ECX+12], 0
;到这里试验码经过用用户名计算的值做指针重新排列
```

到这里试验码变成 `"12x45f7o9rFcBelqdiuk6mg8pnDstjvH3bzAhCaEyGwIJK"`

\====================================================

### 第三部分:注册码最后的计算和比较 (fly 的功劳)

如果你悔牌超过 29 次后，会提示你“未注册".就是说你必须悔牌超过 29 次后 .才会来到这里:

```
00423490  SUB      ESP, 0C  
00423493  PUSH     EBX  
00423494  MOV      EBX, ECX  
00423496  PUSH     EBP  
00423497  XOR      ECX, ECX  
00423499  PUSH     ESI  
0042349A  MOV      ESI, DWORD PTR DS:[EBX+530]  
004234A0  PUSH     EDI  
004234A1  MOV      BYTE PTR SS:[ESP+14], CL  
004234A5  MOV      BYTE PTR SS:[ESP+15], CL  
004234A9  MOV      BYTE PTR SS:[ESP+16], CL  
004234AD  MOV      BYTE PTR SS:[ESP+17], CL  
004234B1  /MOV     EAX, 38E38E39  
004234B6  |MOV     EBP, 47  
004234BB  |IMUL    ECX  
004234BD  |SAR      EDX, 1  
004234BF  |MOV      EAX, EDX  
004234C1  |SHR      EAX, 1F  
004234C4  |ADD      EDX, EAX  
004234C6  |XOR      EAX, EAX  
004234C8  |MOV      AL, BYTE PTR DS:[ESI+ECX]   
004234CB  |LEA      EDI, DWORD PTR SS:[ESP+EDX+14>  
004234CF  |CDQ  
004234D0  |IDIV     EBP  
004234D2  |MOV      AL, BYTE PTR DS:[EDI]  
004234D4  |ADD      AL, DL  
004234D6  |INC      ECX  
004234D7  |CMP      ECX, 9  
004234DA  |MOV      BYTE PTR DS:[EDI], AL       
004234DC  \\JL      SHORT 超级拖拉.004234B1
;以上把试验码重新排列后的字符串的前 9 位"12x45f7o9"的 hex 值计算后相加   
;  AL =B4
004234DE  MOV      ECX, 9  
004234E3  /MOV     EAX, 38E38E39  
004234E8  |MOV     EBP, 1D  
004234ED  |IMUL    ECX  
004234EF  |SAR      EDX, 1  
004234F1  |MOV      EAX, EDX  
004234F3  |SHR      EAX, 1F  
004234F6  |ADD      EDX, EAX  
004234F8  |XOR      EAX, EAX  
004234FA  |MOV      AL, BYTE PTR DS:[ESI+ECX]      
004234FD  |LEA      EDI, DWORD PTR SS:[ESP+EDX+14>  
00423501  |CDQ  
00423502  |IDIV     EBP  
00423504  |MOV      AL, BYTE PTR DS:[EDI]  
00423506  |ADD      AL, DL  
00423508  |INC      ECX  
00423509  |CMP      ECX, 12  
0042350C  |MOV      BYTE PTR DS:[EDI], AL          ; EAX=8B  
0042350E  \\JL      SHORT 超级拖拉.004234E3
;以上把试验码重新排列后的字符串的第 10-18 位"rFcBelqdi"的 hex 值计算后相加   
;  AL =8B
00423510  MOV      ECX, 12  
00423515  /MOV     EAX, 38E38E39  
0042351A  |MOV     EBP, 2F  
0042351F  |IMUL    ECX  
00423521  |SAR      EDX, 1  
00423523  |MOV      EAX, EDX  
00423525  |SHR      EAX, 1F  
00423528  |ADD      EDX, EAX  
0042352A  |XOR      EAX, EAX  
0042352C  |MOV      AL, BYTE PTR DS:[ESI+ECX]  
0042352F  |LEA      EDI, DWORD PTR SS:[ESP+EDX+14>  
00423533  |CDQ  
00423534  |IDIV     EBP  
00423536  |MOV      AL, BYTE PTR DS:[EDI]  
00423538  |ADD      AL, DL  
0042353A  |INC      ECX  
0042353B  |CMP      ECX, 1B  
0042353E  |MOV      BYTE PTR DS:[EDI], AL  
00423540  \\JL      SHORT 超级拖拉.00423515        ; EAX=95
;以上把试验码重新排列后的字符串的第 19-27 位"uk6mg8pnD"的 hex 值计算后相加   
;  AL =95
00423542  MOV      ECX, 1B  
```
