
# ASProtect 1.23 RC4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6365.htm`](https://www.pediy.com/kssd/pediy06/pediy6365.htm)

### 前言
哈哈,二哥叫我帮忙脱的,看幻影差点就忘记了!现在才看,对不起哦!

脱掉 `ASProtect 1.23 RC4 - 1.3.08.24` -> `Alexey Solodovnikov`

### 软件信息
- **软件名称**：`Christmas 3D Screensaver`
- **下载页面**：`http://www.3planesoft.com`
- **软件大小**：有点大!
- **应用平台**：`WIN9X/WINNT/WIN2K/WINXP`
- **软件简介**：好像是一个圣诞节有关的东西,谁要是破了就发个注册机出来给偶下载回家过圣诞!谢谢
- **软件限制**：`ASProtect 1.23 RC4 - 1.3.08.24` -> `Alexey Solodovnikov`
- **文章作者**：辉仔 Yock

### 作者声明
本人发表这篇文章只是为了学习和研究!!!请不用于商业用途或是将本文方法制作的注册机或是补丁文件任意传播,读者看了文章后所做的事情与我无关,我也不会负责,请读者看了文章后三思而后行!最后希望大家在经济基础好的时候,支持共享软件!(在这里最此软件的作者以万二分的歉意鞠躬...)

### 破解工具
`OD` `lordpe` `impor`

---

### 过程
读者看之前一定要紧记住 `ASProtect` 可的地址每次都不同的,下面的代码在最后那一小段我是分开两次复制的,所以不要看代码地址,寻呼要学会理解我所做的!

`OD` 载入后隐藏 `OD`!忽略所有异常(除内存)

`F9` 运行,会异常,用 `Shift+F9` 通过异常 `25` 次到下面这里!

```assembly
00AE39EC    3100            XOR     [EAX],EAX
//这里异常!

00AE39EE    64:8F05 0000000>POP     DWORD PTR FS:[0]               ; 0012FFE0
00AE39F5    58              POP     EAX                             ; 0012FFE0
00AE39F6    833D B07EAE00 0>CMP     DWORD PTR [AE7EB0],0
00AE39FD    74 14           JE      SHORT 00AE3A13
00AE39FF    6A 0C           PUSH    0C
00AE3A01    B9 B07EAE00     MOV     ECX,0AE7EB0
00AE3A06    8D45 F8         LEA     EAX,[EBP-8]
00AE3A09    BA 04000000     MOV     EDX,4
00AE3A0E    E8 2DD1FFFF     CALL    00AE0B40
00AE3A13    FF75 FC         PUSH    DWORD PTR [EBP-4]
00AE3A16    FF75 F8         PUSH    DWORD PTR [EBP-8]
00AE3A19    8B45 F4         MOV     EAX,[EBP-C]                     ; Christma.004707D2
00AE3A1C    8338 00         CMP     DWORD PTR [EAX],0
00AE3A1F    74 02           JE      SHORT 00AE3A23
00AE3A21    FF30            PUSH    DWORD PTR [EAX]
00AE3A23    FF75 F0         PUSH    DWORD PTR [EBP-10]              ; Christma.00400000
00AE3A26    FF75 EC         PUSH    DWORD PTR [EBP-14]
00AE3A29    C3              RETN

//在这里 F2 下断,再按 Shift+F9 就停在这里了!

//取消断点后按 F7 返回

00AF4F28    8AF3            MOV     DH,BL
//返回到这里!
//下面就要按 F7 慢慢跟咯!

00AF4F2A    66:81C0 A665    ADD     AX,65A6
00AF4F2F    66:81C9 3236    OR      CX,3632
00AF4F34    E8 08000000     CALL    00AF4F41
00AF4F39    0039            ADD     [ECX],BH
00AF4F3B  ^ 7E DF           JLE     SHORT 00AF4F1C
00AF4F3D    2C F5           SUB     AL,0F5
00AF4F3F    8AFB            MOV     BH,BL
00AF4F41    0F84 02000000   JE      00AF4F49
00AF4F47    8BC3            MOV     EAX,EBX
00AF4F49    5F              POP     EDI                             ; Christma.00400000
00AF4F4A    0FBFD0          MOVSX   EDX,AX
00AF4F4D    81C7 15160000   ADD     EDI,1615
00AF4F53    B9 1D4FD976     MOV     ECX,76D94F1D
00AF4F58    66:B8 19C0      MOV     AX,0C019
00AF4F5C    66:BA 8C55      MOV     DX,558C
00AF4F60    2BDB            SUB     EBX,EBX
00AF4F62    E9 0D000000     JMP     00AF4F74
00AF4F67    51              PUSH    ECX
00AF4F68    B6 B7           MOV     DH,0B7
00AF4F6A    24 8D           AND     AL,8D
00AF4F6C    42              INC     EDX
00AF4F6D    53              PUSH    EBX
00AF4F6E    90              NOP
00AF4F6F    898E AFBC45B9   MOV     [ESI+B945BCAF],ECX
00AF4F75    C138 5A         SAR     DWORD PTR [EAX],5A              ; 移动常数超出 1..31 的范围
00AF4F78    67:0F87 0600000>JA      00AF4F85                          ; 多余的前缀
00AF4F7F    66:8BD6         MOV     DX,SI
00AF4F82    80F1 EC         XOR     CL,0EC
00AF4F85    FF343B          PUSH    DWORD PTR [EBX+EDI]
00AF4F88    81E0 842A501B   AND     EAX,1B502A84
00AF4F8E    0FB7CF          MOVZX   ECX,DI
00AF4F91    5E              POP     ESI                             ; Christma.00400000
00AF4F92    0F83 05000000   JNB     00AF4F9D
00AF4F98    80C9 C6         OR      CL,0C6
00AF4F9B    B1 43           MOV     CL,43
00AF4F9D    81C6 2A7E6325   ADD     ESI,25637E2A
00AF4FA3    E8 06000000     CALL    00AF4FAE
00AF4FA8    76 77           JBE     SHORT 00AF5021
00AF4FAA    E4 4D           IN      AL,4D                           ; I/O 命令
00AF4FAC    0213            ADD     DL,[EBX]
00AF4FAE    66:B8 0507      MOV     AX,705
00AF4FB2    B4 0E           MOV     AH,0E
00AF4FB4    5A              POP     EDX                             ; Christma.00400000
00AF4FB5    53              PUSH    EBX
00AF4FB6    B9 B90CD669     MOV     ECX,69D60CB9
00AF4FBB    0FBFC1          MOVSX   EAX,CX
00AF4FBE    5A              POP     EDX                             ; Christma.00400000
00AF4FBF    66:81E2 5734    AND     DX,3457
00AF4FC4    81F6 1BB4497A   XOR     ESI,7A49B41B
00AF4FCA    80E5 B0         AND     CH,0B0
00AF4FCD    8AF4            MOV     DH,AH
00AF4FCF    81EE B819242E   SUB     ESI,2E2419B8
00AF4FD5    81DA 47E12F2A   SBB     EDX,2A2FE147
00AF4FDB    E8 12000000     CALL    00AF4FF2
00AF4FE0    12E3            ADC     AH,BL
00AF4FE2  ^ E0 99           LOOPDNE SHORT 00AF4F7D
00AF4FE4    5E              POP     ESI                             ; Christma.00400000
00AF4FE5    3F              AAS
00AF4FE6    0C 55           OR      AL,55
00AF4FE8    6A 5B           PUSH    5B
00AF4FEA    F8              CLC
00AF4FEB    D136            SAL     DWORD PTR [ESI],1
00AF4FED    37              AAA
00AF4FEE    A4              MOVS    BYTE PTR ES:[EDI],BYTE PTR [ESI]
00AF4FEF    0D C2D3E913     OR      EAX,13E9D3C2
00AF4FF4    0000            ADD     [EAX],AL
00AF4FF6    002F            ADD     [EDI],CH
00AF4FF8    3C C5           CMP     AL,0C5
00AF4FFA    1A4B 28         SBB     CL,[EBX+28]
00AF4FFD    41              INC     ECX
00AF4FFE    E6 27           OUT     27,AL                           ; I/O 命令
00AF5000    D4 7D           AAM     7D
00AF5002  ^ 72 C3           JB      SHORT 00AF4FC7
00AF5004    40              INC     EAX                             ; Christma.004707D2
00AF5005  ^ 79 BE           JNS     SHORT 00AF4FC5
00AF5007    1F              POP     DS                              ; 修正的段位寄存器
00AF5008    6C              INS     BYTE PTR ES:[EDI],DX            ; I/O 命令
00AF5009    35 B2A25856     XOR     EAX,5658A2B2
00AF500E    81DA E9138344   SBB     EDX,448313E9
00AF5014    0F8B 08000000   JPO     00AF5022
00AF501A    66:8BD0         MOV     DX,AX
00AF501D    BA 4657B770     MOV     EDX,70B75746
00AF5022    E8 09000000     CALL    00AF5030
00AF5027    5D              POP     EBP                             ; Christma.00400000
00AF5028    D2A3 A0591EFF   SHL     BYTE PTR [EBX+FF1E59A0],CL
00AF502E    CC              INT3
00AF502F    15 66B8914A     ADC     EAX,4A91B866
00AF5034    B9 CDFE696D     MOV     ECX,6D69FECD
00AF5039    80D9 FC         SBB     CL,0FC
00AF503C    58              POP     EAX                             ; Christma.00400000
00AF503D    8F041F          POP     DWORD PTR [EDI+EBX]             ; Christma.00400000
00AF5040    8BCA            MOV     ECX,EDX
00AF5042    66:8BD3         MOV     DX,BX
00AF5045    66:BA 2C34      MOV     DX,342C
00AF5049    0FB7C3          MOVZX   EAX,BX
00AF504C    83EB 04         SUB     EBX,4
00AF504F    8AD1            MOV     DL,CL
00AF5051    8BD6            MOV     EDX,ESI
00AF5053    E8 14000000     CALL    00AF506C
00AF5058    5C              POP     ESP                             ; Christma.00400000
00AF5059    65:3AEB         CMP     CH,BL                           ; 多余的前缀
00AF505C    48              DEC     EAX                             ; Christma.004707D2
00AF505D    E1 06           LOOPDE  SHORT 00AF5065
```
