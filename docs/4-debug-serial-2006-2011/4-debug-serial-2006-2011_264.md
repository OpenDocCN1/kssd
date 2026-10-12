
# 破解 AudioVideo To Wav Converter 1.0.3.0305

> **原文**：[`www.pediy.com/kssd/pediy09/pediy09-244.htm`](https://www.pediy.com/kssd/pediy09/pediy09-244.htm)

> **【文章标题】**：破解 AudioVideo To Wav Converter 1.0.3.0305
> **【文章作者】**：ioukenzo/de 神
> **【作者邮箱】**：`sos_ftp@yahoo.com.cn`
> **【作者主页】**：暂时还没有
> **【作者 QQ 号】**：`250771765`
> **【软件名称】**：AudioVideo To Wav Converter
> **【下载地址】**：`http://www.newhua.com/soft/32395.htm`
> **【加壳方式】**：壳
> **【使用工具】**：`Peid`, `OD`
> **【操作平台】**：`WinXp SP2`
> **【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

### 【详细过程】

#### 一：行程序就要求注册了。那我们就便注一下吧。看有有提示！

#### 二：用 `PEID` 查一下壳。

#### 三：用 `OD` 载入，追出真码。

================================================================

#### （1）载入 `OD` 后搜索一下注册失败时的错误指示－－> `Registration failed!` 找到后双击来到以下地方：

================================================================

```
00404A84   .  5E            POP ESI
00404A85   .  5D            POP EBP
00404A86   .  83C4 0C       ADD ESP,0C
00404A89   .  E9 82E80100   JMP AV2Wav.00423310
00404A8E   >  6A 00         PUSH 0
00404A90   .  6A 00         PUSH 0
00404A92   .  68 E8E24200   PUSH AV2Wav.0042E2E8      ;  Registration failed! ;双击错误提示之后 OD 停在这里。
00404A97   .  E8 401D0200   CALL AV2Wav.004267DC
00404A9C   .  5B            POP EBX
00404A9D   .  5F            POP EDI
00404A9E   .  5E            POP ESI
00404A9F   .  5D            POP EBP
00404AA0   .  83C4 0C       ADD ESP,0C
00404AA3   .  C3            RETN
```

================================================================

#### (2)下断慢慢分析。

================================================================

```
004047D3   .  55            PUSH EBP                 ;来到这里按 F2 下断。
004047D4   .  56            PUSH ESI
004047D5   .  57            PUSH EDI
004047D6   .  BF 01000000   MOV EDI,1
004047DB   .  57            PUSH EDI
004047DC   .  8BF1          MOV ESI,ECX
004047DE   .  E8 C3AA0100   CALL AV2Wav.0041F2A6
004047E3   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]           ;  把用名 EAX
004047E6   .  8B68 F4       MOV EBP,DWORD PTR DS:[EAX-C]           ;  把用名的度 EBP
004047E9   .  83FD 02       CMP EBP,2                          ;  用名的度和 2 比
004047EC   .  7D 15         JGE SHORT AV2Wav.00404803
004047EE   .  6A 00         PUSH 0
004047F0   .  6A 00         PUSH 0
004047F2   .  68 44E34200   PUSH AV2Wav.0042E344             ;  Please input correct User Name!
004047F7   .  E8 E01F0200   CALL AV2Wav.004267DC
004047FC   .  5F            POP EDI
004047FD   .  5E            POP ESI
004047FE   .  5D            POP EBP
004047FF   .  83C4 0C       ADD ESP,0C
00404802   .  C3            RETN
00404803   >  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]           ;  把假 ECX
00404806   .  8379 F4 08    CMP DWORD PTR DS:[ECX-C],8             ;  假的度和 8 比
0040480A   .  7D 15         JGE SHORT AV2Wav.00404821
0040480C   .  6A 00         PUSH 0
0040480E   .  6A 00         PUSH 0
00404810   .  68 1CE34200   PUSH AV2Wav.0042E31C         ;  Please input correct Registration Code!
00404815   .  E8 C21F0200   CALL AV2Wav.004267DC
0040481A   .  5F            POP EDI
0040481B   .  5E            POP ESI
0040481C   .  5D            POP EBP
0040481D   .  83C4 0C       ADD ESP,0C
00404820   .  C3            RETN
00404821   >  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]           ;  把用名 EAX
00404824   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]           ;  把用名的度 ECX
00404827   .  85C9          TEST ECX,ECX
00404829   .  7D 0A         JGE SHORT AV2Wav.00404835
0040482B   .  68 57000780   PUSH 80070057
00404830   .  E8 1BC9FFFF   CALL AV2Wav.00401150
00404835   >  8A10          MOV DL,BYTE PTR DS:[EAX]               ;  取用名的第一位字符的 ASCII 值 DL
00404837   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]           ;  把用名 EAX
0040483A   .  3978 F4       CMP DWORD PTR DS:[EAX-C],EDI             ;  用名的度和 EDI 的值比
0040483D   .  7D 0A         JGE SHORT AV2Wav.00404849
0040483F   .  68 57000780   PUSH 80070057
00404844   .  E8 07C9FFFF   CALL AV2Wav.00401150
00404849   >  8A40 01       MOV AL,BYTE PTR DS:[EAX+1]             ;  取用名第二位字符的 ASCII 值 AL
0040484C   .  884424 0E     MOV BYTE PTR SS:[ESP+E],AL             ;  [ESP+E]=AL=用名第二位字符的 ASCII 值
00404850   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]           ;  把用名 EAX
00404853   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]           ;  把用名的度 ECX
00404856   .  85C9          TEST ECX,ECX
00404858   .  7D 0A         JGE SHORT AV2Wav.00404864
0040485A   .  68 57000780   PUSH 80070057
0040485F   .  E8 ECC8FFFF   CALL AV2Wav.00401150
00404864   >  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+70]           ;  把用名 ECX
00404867   .  53            PUSH EBX
00404868   .  8A18          MOV BL,BYTE PTR DS:[EAX]               ;  取用名第一位字符的 ASCII 值 BL
0040486A   .  3979 F4       CMP DWORD PTR DS:[ECX-C],EDI             ;  用名的度和 EDI 的值比
0040486D   .  7D 0A         JGE SHORT AV2Wav.00404879
0040486F   .  68 57000780   PUSH 80070057
00404874   .  E8 D7C8FFFF   CALL AV2Wav.00401150
00404879   >  0FB6C2        MOVZX EAX,DL                       ;  逐位取用名的 ASCII 值 EAX
0040487C   .  83C8 46       OR EAX,46                        ;  EAX 和 46 域
0040487F   .  99            CDQ
00404880   .  BF 0A000000   MOV EDI,0A                     ;  把 0AEDI
00404885   .  F7FF          IDIV EDI
00404887   .  0FB64424 12   MOVZX EAX,BYTE PTR SS:[ESP+12]     ;  取用名第二位字符的 ASCII 值 EAX
0040488C   .  83C8 42       OR EAX,42                        ;  EAX 和 42 域
0040488F   .  885424 16     MOV BYTE PTR SS:[ESP+16],DL         ;  [ESP+16]=DL=01
00404893   .  99            CDQ
00404894   .  F7FF          IDIV EDI
00404896   .  0FB6C3        MOVZX EAX,BL                       ;  取用名第一位字符的 ASCII 值 EAX
00404899   .  83C8 43       OR EAX,43                        ;  EAX 和 43 域
0040489C   .  885424 12     MOV BYTE PTR SS:[ESP+12],DL         ;  [ESP+12]=DL=01
004048A0   .  99            CDQ
004048A1   .  F7FF          IDIV EDI
004048A3   .  0FB641 01     MOVZX EAX,BYTE PTR DS:[ECX+1]         ;  取用名第二位字符的 ASCII 值 EAX
004048A7   .  83C8 44       OR EAX,44                        ;  EAX 和 44 域
004048AA   .  8BCF          MOV ECX,EDI                     ;  ECX=EDI=0A
004048AC   .  885424 17     MOV BYTE PTR SS:[ESP+17],DL         ;  [ESP+17]=DL=07
004048B0   .  99            CDQ
004048B1   .  F7F9          IDIV ECX
004048B3   .  33C0          XOR EAX,EAX                     ;  EAX 清零
004048B5   .  33C9          XOR ECX,ECX                     ;  ECX 清零
004048B7   .  85ED          TEST EBP,EBP
004048B9   .  885424 18     MOV BYTE PTR SS:[ESP+18],DL         ;  [ESP+18]=DL=01
004048BD   .  7E 20         JLE SHORT AV2Wav.004048DF
004048BF   .  90            NOP
004048C0   >  85C9          TEST ECX,ECX
004048C2   .  0F8C D2000000 JL AV2Wav.0040499A
004048C8   .  8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]           ;  把用名 EDI
004048CB   .  3B4F F4       CMP ECX,DWORD PTR DS:[EDI-C]
004048CE   .  0F8F C6000000 JG AV2Wav.0040499A
004048D4   .  0FB6140F      MOVZX EDX,BYTE PTR DS:[EDI+ECX]      ;  逐位取用名的 ASCII 值 EDX
004048D8   .  03C2          ADD EAX,EDX                     ;  EAX=EAX+EDX
```



```
004048DA   . 41            INC ECX                           ;  ECX 自加一
004048DB   . 3BCD          CMP ECX,EBP                       ;  ECX 和 EBP 比
004048DD   . ^ 7C E1       JL SHORT AV2Wav.004048C0          ;  循环
004048DF   > 8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]     ;  把假 ECX
004048E2   . 8B51 F4       MOV EDX,DWORD PTR DS:[ECX-C]      ;  把假的度 EDX
004048E5   . 85D2          TEST EDX,EDX
004048E7   . 7D 0A         JGE SHORT AV2Wav.004048F3
004048E9   . 68 57000780   PUSH 80070057
004048EE   . E8 5DC8FFFF   CALL AV2Wav.00401150
004048F3   > 8A11          MOV DL,BYTE PTR DS:[ECX]          ;  取假第一位的 ASCII 值 DL
004048F5   . 8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]     ;  把假 ECX
004048F8   . 8379 F4 01    CMP DWORD PTR DS:[ECX-C],1        ;  假的度和 1 比
004048FC   . 885424 19     MOV BYTE PTR SS:[ESP+19],DL      ;  [ESP+19]＝DL＝假第一位的 ASCII 值
00404900   . 7D 0A         JGE SHORT AV2Wav.0040490C
00404902   . 68 57000780   PUSH 80070057
00404907   . E8 44C8FFFF   CALL AV2Wav.00401150
0040490C   > 8A49 01       MOV CL,BYTE PTR DS:[ECX+1]        ;  取假第二位的 ASCII 值 CL
0040490F   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
00404912   . 884C24 13     MOV BYTE PTR SS:[ESP+13],CL      ;  [ESP+13]=CL=假第二位的 ASCII 值
00404916   . 837F F4 02    CMP DWORD PTR DS:[EDI-C],2        ;  假的度和 2 比
0040491A   . 7D 0A         JGE SHORT AV2Wav.00404926
0040491C   . 68 57000780   PUSH 80070057
00404921   . E8 2AC8FFFF   CALL AV2Wav.00401150
00404926   > 8A4F 02       MOV CL,BYTE PTR DS:[EDI+2]        ;  取假第三位的 ASCII 值 CL
00404929   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
0040492C   . 884C24 14     MOV BYTE PTR SS:[ESP+14],CL      ;  [ESP+14]=CL=假第三位的 ASCII 值
00404930   . 837F F4 03    CMP DWORD PTR DS:[EDI-C],3        ;  假的度和 3 比
00404934   . 7D 0A         JGE SHORT AV2Wav.00404940
00404936   . 68 57000780   PUSH 80070057
0040493B   . E8 10C8FFFF   CALL AV2Wav.00401150
00404940   > 8A4F 03       MOV CL,BYTE PTR DS:[EDI+3]        ;  取假第四位的 ASCII 值 CL
00404943   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
00404946   . 884C24 15     MOV BYTE PTR SS:[ESP+15],CL      ;  [ESP+15]=CL=假第四位的 ASCII 值
0040494A   . 837F F4 04    CMP DWORD PTR DS:[EDI-C],4        ;  假的度和 4 比
0040494E   . 7D 0A         JGE SHORT AV2Wav.0040495A
00404950   . 68 57000780   PUSH 80070057
00404955   . E8 F6C7FFFF   CALL AV2Wav.00401150
0040495A   > 8A4F 04       MOV CL,BYTE PTR DS:[EDI+4]        ;  取假第五位的 ASCII 值 CL
0040495D   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
00404960   . 837F F4 05    CMP DWORD PTR DS:[EDI-C],5        ;  假的度和 5 比
00404964   . 7D 0A         JGE SHORT AV2Wav.00404970
00404966   . 68 57000780   PUSH 80070057
0040496B   . E8 E0C7FFFF   CALL AV2Wav.00401150
00404970   > 8A5F 05       MOV BL,BYTE PTR DS:[EDI+5]        ;  取假第六位的 ASCII 值 CL
00404973   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
00404976   . 885C24 1A     MOV BYTE PTR SS:[ESP+1A],BL      ;  [ESP+1A]=BL=假第六位的 ASCII 值
0040497A   . 837F F4 06    CMP DWORD PTR DS:[EDI-C],6        ;  假的度和 6 比
0040497E   . 7D 0A         JGE SHORT AV2Wav.0040498A
00404980   . 68 57000780   PUSH 80070057
00404985   . E8 C6C7FFFF   CALL AV2Wav.00401150
0040498A   > 8A5F 06       MOV BL,BYTE PTR DS:[EDI+6]        ;  取假第七位的 ASCII 值 BL
0040498D   . 8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]     ;  把假 EDI
00404990   . 885C24 1B     MOV BYTE PTR SS:[ESP+1B],BL      ;  [ESP+1A]=BL=假第七位的 ASCII 值
00404994   . 837F F4 07    CMP DWORD PTR DS:[EDI-C],7        ;  假的度和 7 比
00404998   . 7D 0A         JGE SHORT AV2Wav.004049A4
0040499A   > 68 57000780   PUSH 80070057
0040499F   . E8 ACC7FFFF   CALL AV2Wav.00401150
004049A4   > 8A5F 07       MOV BL,BYTE PTR DS:[EDI+7]        ;  取假第八位的 ASCII 值 BL
004049A7   . 0FB67C24 16   MOVZX EDI,BYTE PTR SS:[ESP+16]   ;  EDI=[ESP+16]=01
004049AC   . 0FB6D2        MOVZX EDX,DL                      ;  取假第一位的 ASCII 值 EDX
004049AF   . 83EA 30       SUB EDX,30                        ;  EDX=EDX-30
004049B2   . 3BFA          CMP EDI,EDX                       ;  EDI 和 EDX 比
004049B4   . 75 48         JNZ SHORT AV2Wav.004049FE
004049B6   . 0FB65424 13   MOVZX EDX,BYTE PTR SS:[ESP+13]
004049BB   . 0FB67C24 12   MOVZX EDI,BYTE PTR SS:[ESP+12]
004049C0   . 83EA 30       SUB EDX,30
004049C3   . 3BFA          CMP EDI,EDX
004049C5   . 75 37         JNZ SHORT AV2Wav.004049FE
004049C7   . 0FB65424 14   MOVZX EDX,BYTE PTR SS:[ESP+14]
004049CC   . 0FB67C24 17   MOVZX EDI,BYTE PTR SS:[ESP+17]
004049D1   . 83EA 30       SUB EDX,30
004049D4   . 3BFA          CMP EDI,EDX
004049D6   . 75 26         JNZ SHORT AV2Wav.004049FE
004049D8   . 0FB65424 15   MOVZX EDX,BYTE PTR SS:[ESP+15]
004049DD   . 0FB67C24 18   MOVZX EDI,BYTE PTR SS:[ESP+18]
004049E2   . 83EA 30       SUB EDX,30
004049E5   . 3BFA          CMP EDI,EDX
004049E7   . 75 15         JNZ SHORT AV2Wav.004049FE
004049E9   . 99            CDQ
004049EA   . BF 0A000000   MOV EDI,0A
004049EF   . F7FF          IDIV EDI
004049F1   . 0FB6C2        MOVZX EAX,DL
004049F4   . 0FB6D1        MOVZX EDX,CL
004049F7   . 83EA 30       SUB EDX,30
004049FA   . 3BC2          CMP EAX,EDX
004049FC   . 74 3A         JE SHORT AV2Wav.00404A38
004049FE   > 807C24 19 38  CMP BYTE PTR SS:[ESP+19],38      ;  假第一位的 ASCII 值和 38 比
00404A03   . 0F85 85000000 JNZ AV2Wav.00404A8E
00404A09   . 807C24 13 33  CMP BYTE PTR SS:[ESP+13],33      ;  假第二位的 ASCII 值和 33 比
00404A0E   . 75 7E         JNZ SHORT AV2Wav.00404A8E
00404A10   . 807C24 14 39  CMP BYTE PTR SS:[ESP+14],39      ;  假第三位的 ASCII 值和 39 比
00404A15   . 75 77         JNZ SHORT AV2Wav.00404A8E
00404A17   . 8A5424 15     MOV DL,BYTE PTR SS:[ESP+15]      ;  把假第四位的 ASCII 值 DL
00404A1B   . B0 31         MOV AL,31                         ;  AL=31
00404A1D   . 3AD0          CMP DL,AL                         ;  假第四位的 ASCII 值和 AL 的值比
00404A1F   . 75 6D         JNZ SHORT AV2Wav.00404A8E
00404A21   . 80F9 36       CMP CL,36                         ;  假第五位的 ASCII 值和 36 比
00404A24   . 75 68         JNZ SHORT AV2Wav.00404A8E
00404A26   . 384424 1A     CMP BYTE PTR SS:[ESP+1A],AL      ;  假第六位的 ASCII 值和 AL 的值比
00404A2A   . 75 62         JNZ SHORT AV2Wav.00404A8E
00404A2C   . 807C24 1B 34  CMP BYTE PTR SS:[ESP+1B],34      ;  假第七位的 ASCII 值和 34 比
00404A31   . 75 5B         JNZ SHORT AV2Wav.00404A8E
00404A33   . 80FB 36       CMP BL,36                         ;  假第八位的 ASCII 值和 36 比
00404A36   . 75 56         JNZ SHORT AV2Wav.00404A8E
00404A38   > 6A 00         PUSH 0
00404A3A   . 6A 00         PUSH 0
00404A3C   . 68 00E34200   PUSH AV2Wav.0042E300              ;  Registration has succeeded!
00404A41   . E8 961D0200   CALL AV2Wav.004267DC
00404A46   . 8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]
00404A49   . E8 96340200   CALL AV2Wav.00427EE4
00404A4E   . 8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]
```



```
00404A51   .  57            PUSH EDI                             ; /Arg3  
00404A52   .  68 80E04200   PUSH AV2Wav.0042E080                 ; |username  
00404A57   .  68 FCDB4200   PUSH AV2Wav.0042DBFC                 ; |Option  
00404A5C   .  8BC8          MOV ECX,EAX                          ; |  
00404A5E   .  E8 901E0200   CALL AV2Wav.004268F3                 ; \AV2Wav.004268F3  
00404A63   .  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  
00404A66   .  E8 79340200   CALL AV2Wav.00427EE4  
00404A6B   .  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]  
00404A6E   .  57            PUSH EDI                               ; /Arg3  
00404A6F   .  68 6CE04200   PUSH AV2Wav.0042E06C                 ; |registration_code  
00404A74   .  68 FCDB4200   PUSH AV2Wav.0042DBFC                 ; |Option  
00404A79   .  8BC8          MOV ECX,EAX                          ; |  
00404A7B   .  E8 731E0200   CALL AV2Wav.004268F3                 ; \AV2Wav.004268F3  
00404A80   .  5B            POP EBX  
00404A81   .  5F            POP EDI  
00404A82   .  8BCE          MOV ECX,ESI  
00404A84   .  5E            POP ESI  
00404A85   .  5D            POP EBP  
00404A86   .  83C4 0C       ADD ESP,0C  
00404A89   .  E9 82E80100   JMP AV2Wav.00423310  
00404A8E   >  6A 00         PUSH 0  
00404A90   .  6A 00         PUSH 0  
00404A92   .  68 E8E24200   PUSH AV2Wav.0042E2E8       ;  Registration failed!  
00404A97   .  E8 401D0200   CALL AV2Wav.004267DC  
00404A9C   .  5B            POP EBX  
00404A9D   .  5F            POP EDI  
00404A9E   .  5E            POP ESI  
00404A9F   .  5D            POP EBP  
00404AA0   .  83C4 0C       ADD ESP,0C  
00404AA3   .  C3            RETN   
```

---

**【经验总结】**

**四：总结**

这款软件算是明码比较的。用户名没有参与运算。但要大于 2 位。注册码是通用的。要大于或者等于 8 位，而且前 8 位是固定的。

真码是：`83916146`

---

**【版权声明】**: 本文原创于 `ioukenzo/de` 神, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
