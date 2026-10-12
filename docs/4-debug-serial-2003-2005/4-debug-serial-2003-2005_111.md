
# ElectrObalz

> 原文：`www.pediy.com/kssd/pediy05/pediy50621.htm`

ElectrObalz 一个游戏

软件大小：1335 KB  
软件语言：英语  
软件类别：国外软件 / 共享版 / 游戏  
应用平台：`Win9x/NT/2000/XP`  
下载页面: `www.isotope244.com/games.html`

### 软件说明

Smash through over 50 levels in this awesome action game. The goal is simple, destroy all of the bricks in each level to proceed to the next level. Along the way you will receive many different power items to help you.

The main element in ElectrObalz is Ether. Ether comes in two forms, red and blue, both of which are needed to electrically form an electrOball! Your space pad initiates an electrOball and then you can reflect the ball about the level to destroy many different types of bricks. After completing many levels and encountering many items and bricks, and only then will you have the experience to finish the epic adventure of ElectrObalz!

### 【作者声明】
本人只是对 Crack 感兴趣，没有其它目的。

### 【破解工具】
`Ollydbg1.09` 中文版

---

### 【过程】
smallrice 兄弟用 `WINHEX` 破解了这个软件，说明是明码比较可能简单。正好用来练习学习算法。运行游戏打开注册窗口，输入用户名: `fxyang` E_mail: `fxyang@163.com` 试验码: `789456123012345678` 注册码要 18 位。再运行 `Ollydbg` 用附加功能加载程序，然后按 `ALT+E` 选择注册主程序，会车，来到程序领空。`CTRL+N` 选择 `GetDlgtemTextA` 下中断，运行程序，被 `OD` 中断后来到这里：

```
00411CF0   SUB      ESP, 20  
00411CF3   OR       ECX, FFFFFFFF  
00411CF6   XOR      EAX, EAX  
00411CF8   PUSH     ESI  
00411CF9   MOV      ESI, DWORD PTR SS:[ESP+34]   
; ESI<--ASCII "789456123012345678"(试验码)
00411CFD   PUSH     EDI  
00411CFE   MOV      EDI, ESI  
00411D00   REPNE    SCAS BYTE PTR ES:[EDI]  
00411D02   NOT      ECX  
00411D04   DEC      ECX  
00411D05   CMP      ECX, 12                
; <--长度 12(H)=18 位
00411D08   JE       SHORT Electrob.00411D10  
00411D0A   POP      EDI  
00411D0B   POP      ESI  
00411D0C   ADD      ESP, 20  
00411D0F   RETN  
00411D10   MOV      ECX, DWORD PTR SS:[ESP+34]      
; ECX<--0012F4EC,(ASCII "fxyang@163.com")(E_mail)
00411D14   MOV      EDX, DWORD PTR SS:[ESP+30]       
; EDX<--0012F52C,(ASCII "fxyang")(用户名)
00411D18   LEA      EAX, DWORD PTR SS:[ESP+8]  
00411D1C   PUSH     EAX  
00411D1D   MOV      EAX, DWORD PTR SS:[ESP+30]       
; EAX<--0042A540  ASCII "ElectrObalz"(固定参数)
00411D21   PUSH     ECX  
00411D22   PUSH     EDX  
00411D23   PUSH     EAX  
00411D24   CALL     Electrob.00411D60  <--计算注册码的地方 F7  
00411D29   ADD      ESP, 10  
00411D2C   TEST     EAX, EAX  
00411D2E   JNZ      SHORT Electrob.00411D36  
00411D30   POP      EDI  
00411D31   POP      ESI  
00411D32   ADD      ESP, 20  
00411D35   RETN  
00411D36   ADD      ESI, 9  
00411D39   PUSH     8  
00411D3B   LEA      ECX, DWORD PTR SS:[ESP+15]       
; ECX<--0012F4B9,(ASCII "910876135") <--通过计算得到的正确后八位
00411D3F   PUSH     ESI                        
; ESI=0012F575,(ASCII "012345678")<--试验码的后八位
00411D40   PUSH     ECX  
00411D41   CALL     Electrob.00418070  
00411D46   ADD      ESP, 0C  
00411D49   NEG      EAX  
00411D4B   SBB      EAX, EAX  
00411D4D   POP      EDI  
00411D4E   INC      EAX               
; EAX=0<--失败的标志  
; EAX=1<--成功的标志  
00411D4F   POP      ESI  
00411D50   ADD      ESP, 20  
00411D53   RETN

CALL     Electrob.00411D60          <--计算注册码的地方 F7

00411D63   PUSH     EBX  
00411D64   PUSH     EBP  
00411D65   PUSH     ESI  
00411D66   MOV      ESI, DWORD PTR SS:[ESP+18]       
; ESI<--0042A540  ASCII "ElectrObalz"
00411D6A   TEST     ESI, ESI  
00411D6C   PUSH     EDI  
00411D6D   JE       Electrob.00411F77  
00411D73   MOV      EBP, DWORD PTR SS:[ESP+20]       
; EBP<--0012F52C,(ASCII "fxyang")
00411D77   TEST     EBP, EBP  
00411D79   JE       Electrob.00411F77  
00411D7F   MOV      EDX, DWORD PTR SS:[ESP+24]       
; EDX<--0012F4EC,(ASCII "fxyang@163.com")
00411D83   TEST     EDX, EDX  
00411D85   JE       Electrob.00411F77  
00411D8B   MOV      EAX, DWORD PTR SS:[ESP+28]  
00411D8F   TEST     EAX, EAX  
00411D91   JE       Electrob.00411F77  
00411D97   MOV      EDI, ESI                 
; EDI<--0042A540  ASCII "ElectrObalz"
00411D99   OR       ECX, FFFFFFFF  
00411D9C   XOR      EAX, EAX  
00411D9E   MOV      BYTE PTR SS:[ESP+10], 0  
00411DA3   REPNE    SCAS BYTE PTR ES:[EDI]  
00411DA5   NOT      ECX  
00411DA7   DEC      ECX                ; ECX=B  
00411DA8   MOV      EDI, EBP           ; EBP<--0012F52C,(ASCII "fxyang")  
00411DAA   MOV      EBX, ECX  
00411DAC   OR       ECX, FFFFFFFF  
00411DAF   REPNE    SCAS BYTE PTR ES:[EDI]  
00411DB1   NOT      ECX  
00411DB3   DEC      ECX                 ; ECX=6  
00411DB4   MOV      EDI, EDX            ; EDX<--0012F4EC,(ASCII "fxyang@163.com")  
00411DB6   ADD      EBX, ECX  
00411DB8   OR       ECX, FFFFFFFF  
00411DBB   REPNE    SCAS BYTE PTR ES:[EDI]  
00411DBD   NOT      ECX  
00411DBF   DEC      ECX                 ; ECX=E  
00411DC0   ADD      EBX, ECX            ; EBX=1F<--总长度  
00411DC2   LEA      EDI, DWORD PTR DS:[EBX+1] ; EDI=1F+1=20  
00411DC5   PUSH     EDI  
00411DC6   CALL     Electrob.00417478  
00411DCB   MOV      EBP, EAX                
00411DCD   ADD      ESP, 4  
00411DD0   TEST     EBP, EBP  
00411DD2   JE       Electrob.00411F77  
00411DD8   MOV      ECX, EDI               
00411DDA   XOR      EAX, EAX  
00411DDC   MOV      EDX, ECX               
00411DDE   MOV      EDI, EBP  
00411DE0   SHR      ECX, 2                
00411DE3   REP      STOS DWORD PTR ES:[EDI]  
00411DE5   MOV      ECX, EDX  
00411DE7   AND      ECX, 3  
00411DEA   REP      STOS BYTE PTR ES:[EDI]  
00411DEC   OR       ECX, FFFFFFFF  
00411DEF   MOV      EDI, ESI                 
00411DF1   XOR      EAX, EAX  
00411DF3   REPNE    SCAS BYTE PTR ES:[EDI]  
00411DF5   NOT      ECX  
00411DF7   SUB      EDI, ECX  
00411DF9   MOV      EAX, ECX               
00411DFB   MOV      ESI, EDI                 
00411DFD   MOV      EDI, EBP  
00411DFF   SHR      ECX, 2  
00411E02   REP      MOVS DWORD PTR ES:[EDI], DWORD P>  
00411E04   MOV      ECX, EAX               
00411E06   XOR      EAX, EAX  
00411E08   AND      ECX, 3  
00411E0B   REP      MOVS BYTE PTR ES:[EDI], BYTE PTR>  
00411E0D   MOV      EDI, DWORD PTR SS:[ESP+20]       
; EDI<--0012F52C,(ASCII "fxyang")
00411E11   OR       ECX, FFFFFFFF  
00411E14   REPNE    SCAS BYTE PTR ES:[EDI]  
00411E16   NOT      ECX  
00411E18   SUB      EDI, ECX  
00411E1A   MOV      ESI, EDI               
00411E1C   MOV      EDX, ECX               
00411E1E   MOV      EDI, EBP                 
; EDI<--0042A540  ASCII "ElectrObalz"
00411E20   OR       ECX, FFFFFFFF  
00411E23   REPNE    SCAS BYTE PTR ES:[EDI]  
00411E25   MOV      ECX, EDX  
00411E27   DEC      EDI  
00411E28   SHR      ECX, 2  
00411E2B   REP      MOVS DWORD PTR ES:[EDI], DWORD P>  
00411E2D   MOV      ECX, EDX  
00411E2F   AND      ECX, 3  
00411E32   REP      MOVS BYTE PTR ES:[EDI], BYTE PTR>  
00411E34   MOV      EDI, DWORD PTR SS:[ESP+24]       
; EDI<--0012F4EC,(ASCII "fxyang@163.com")
00411E38   OR       ECX, FFFFFFFF  
00411E3B   REPNE    SCAS BYTE PTR ES:[EDI]  
00411E3D   NOT      ECX  
00411E3F   SUB      EDI, ECX  
00411E41   MOV      ESI, EDI  
00411E43   MOV      EDX, ECX  
00411E45   MOV      EDI, EBP  
00411E47   OR       ECX, FFFFFFFF  
00411E4A   SCAS     BYTE PTR ES:[EDI]  
00411E4C   ECX, EDX  
00411E4E   EDI  
00411E4F   ECX, 2  
00411E52   REP      MOVS DWORD PTR ES:[EDI], DWORD P>  
00411E54   MOV      ECX, EDX  
00411E56   AND      ECX, 3  
00411E59   REP      MOVS BYTE PTR ES:[EDI], BYTE PTR>  
00411E5B   LEA      ESI, DWORD PTR DS:[EBX-1]  
00411E5E   TEST     ESI, ESI  
00411E60   JBE      SHORT Electrob.00411E7B  
00411E62   OR       EDI, FFFFFFFF  
00411E65   LEA      EAX, DWORD PTR SS:[EBP+1]          
00411E68   SUB      EDI, EBP               
```



; `EAX`<--`00F2E1E0` ASCII `" ElectrObalzfxyangfxyang@163.com"`  

; 以上把三个参数连接起来

```asm
00411E6A   MOV     CL, BYTE PTR DS:[EAX-1]  ; CL=DS:[EAX-1]=45 ('E')  
00411E6D   MOV     DL, BYTE PTR DS:[EAX]    ; DL=DS:[EAX]=6C ('l')  
00411E6F   ADD     DL, CL  
00411E71   MOV     BYTE PTR DS:[EAX], DL    ; DL=B1==>DS:[EAX]  
00411E73   INC     EAX  
00411E74   LEA     EDX, DWORD PTR DS:[EDI+EAX] ; EDX=DS:[EDI+EAX]=01  
00411E77   CMP     EDX, ESI  
00411E79   JB      SHORT Electrob.00411E6A  
00411E7B   ADD     EBX, -2                  
00411E7E   TEST    EBX, EBX  
00411E80   JBE     SHORT Electrob.00411E91
```

; 把连接后的字符串的 hex 值从前向后相加取低位,放在后一位的位置上  

; 完成后内存中的字符串是:

```
00EC23D8  45 B1 16 79 ED 5F AE 10  E?y 韄?  
00EC23E0  71 DD 57 BD 35 AE 0F 7D  q 軼??}  
00EC23E8  E4 4A C2 3B 9C 0A 71 B1  鋭??q?
00EC23F0  E2 18 4B 79 DC 4B B8 00  ?Ky 躃?
```

---

|

```asm
00411E82   MOV     AL, BYTE PTR DS:[EBX+EBP+1]      ; AL=B8  
00411E86   MOV     CL, BYTE PTR DS:[EBX+EBP]        ; CL=4B  
00411E89   ADD     CL, AL  
00411E8B   MOV     BYTE PTR DS:[EBX+EBP], CL        ; CL=03  
00411E8E   DEC     EBX  
00411E8F   JNZ     SHORT Electrob.00411E82 
```

; 把连接后的字符串的 hex 值从后向前相加取低位   

; 完成后内存中的字符串是:

```
00EC23D8  45 AB FA E4 6B 7E 1F 71  E鋕~q  
00EC23E0  61 F0 13 BC FF CA 1C 0D  a???.  
00EC23E8  90 AC 62 A0 65 C9 BF 4E  惉 b 爀煽 N  
00EC23F0  9D BB A3 58 DF 03 B8 00  澔??
```

---

|      


```asm
00411E91   MOV     ESI, DWORD PTR SS:[ESP+1C]  
00411E95   OR      ECX, FFFFFFFF  
00411E98   MOV     EDI, ESI                 
00411E9A   XOR     EAX, EAX  
00411E9C   XOR     EDX, EDX  
00411E9E   REPNE   SCAS BYTE PTR ES:[EDI]  
00411EA0   NOT     ECX  
00411EA2   DEC     ECX  
00411EA3   JE      SHORT Electrob.00411EC3  
00411EA5   MOV     CL, BYTE PTR DS:[EDX+ESI]   ; CL=DS:[EDX+ESI]=45 ('E')  
00411EA8   MOV     BL, BYTE PTR SS:[ESP+10]     ; BL=SS:[0012F48C]=00  
00411EAC   ADD     BL, CL  
00411EAE   MOV     EDI, ESI                 ; EDI<--0042A540  ASCII "ElectrObalz"  
00411EB0   OR      ECX, FFFFFFFF  
00411EB3   XOR     EAX, EAX  
00411EB5   INC     EDX  
00411EB6   MOV     BYTE PTR SS:[ESP+10], BL     ; BL=45  
00411EBA   REPNE   SCAS BYTE PTR ES:[EDI]  
00411EBC   NOT     ECX  
00411EBE   DEC     ECX  
00411EBF   CMP     EDX, ECX  
00411EC1   JB      SHORT Electrob.00411EA5           
```

; 把字符 `"ElectrObalz"` 各位的 hex 值相加取低位=`57`==>`SS:[0012F48C]`

```asm
00411EC3   LEA     EDX, DWORD PTR SS:[ESP+14]  
00411EC7   PUSH    EDX  
00411EC8   CALL    Electrob.00418208            ; EAX=3EA101EF  
00411ECD   LEA     EAX, DWORD PTR SS:[ESP+18]  
00411ED1   PUSH    EAX  
00411ED2   CALL    Electrob.004180A8  
00411ED7   MOV     ESI, DWORD PTR DS:[EAX+14]  
00411EDA   MOV     EDI, DWORD PTR DS:[EAX+1C]  
00411EDD   ADD     ESP, 8  
00411EE0   SUB     ESI, 64  
00411EE3   CMP     EDI, 3E7  
00411EE9   JLE     SHORT Electrob.00411EF0  
00411EEB   MOV     EDI, 3E7  
00411EF0   CMP     ESI, 64  
00411EF3   JLE     SHORT Electrob.00411EFA  
00411EF5   MOV     ESI, 63  
00411EFA   TEST    EDI, EDI  
00411EFC   JGE     SHORT Electrob.00411F00  
00411EFE   XOR     EDI, EDI  
00411F00   TEST    ESI, ESI  
00411F02   JGE     SHORT Electrob.00411F06  
00411F04   XOR     ESI, ESI  
00411F06   CALL    Electrob.00417FDA  
00411F0B   CDQ  
00411F0C   MOV     ECX, 0A  
00411F11   IDIV    ECX  
00411F13   MOV     EAX, DWORD PTR SS:[ESP+10]  
00411F17   AND     EAX, 0FF  
00411F1C   PUSH    EDX  
00411F1D   PUSH    ESI  
00411F1E   CDQ  
00411F1F   IDIV    ECX  
00411F21   PUSH    EDI  
00411F22   PUSH    EDX  
00411F23   PUSH    0  
00411F25   CALL    Electrob.00417FDA  
00411F2A   CDQ  
00411F2B   MOV     ECX, 0A  
00411F30   MOV     ESI, DWORD PTR SS:[ESP+3C]  
00411F34   IDIV    ECX  
00411F36   PUSH    EDX  
00411F37   PUSH    Electrob.0042A54C                
```

; ASCII `"%01d%01d%02d%03d%02dXXXXXXXX%01d"` <--字符串的格式

```asm
00411F3C   PUSH    ESI  
00411F3D   CALL    Electrob.00417EFE                
```

; <--计算出来常数串 `0012F4B0` ASCII `"400710803XXXXXXXX6"`  

; 这个字符串是随机的格式如上

```asm
00411F42   ADD     ESP, 20  
00411F45   XOR     ECX, ECX  
00411F47   XOR     EAX, EAX  
00411F49   MOV     EDI, 0A  
00411F4E   MOV     AL, BYTE PTR DS:[ECX+EBP]  //开始真正的注册码计算      
```

; `AL`=DS:[ECX+EBP]=45 ('E') <--上面计算的串的第一位

```asm
00411F51   CDQ  
00411F52   IDIV    EDI                 
```

; `EAX`=45 IDIV EDI=A ==>`EAX`= 6 `EDX`=9

```asm
00411F54   ADD     DL, 30                 
```

; `DL`=9+30=39  

; 变成数字

```asm
00411F57   MOV     BYTE PTR DS:[ESI+ECX+9], DL  ; DS:[ESI+ECX+9]<--DL=39  
00411F5B   INC     ECX  
00411F5C   CMP     ECX, 8              ; <--计算 8 位  
00411F5F   JB      SHORT Electrob.00411F47      
00411F61   PUSH    EBP                
```

; 计算 8 位填上面的 x 位  

; `0012F4B0` ASCII `"400710803910876135"` <--完成计算后的字符串  

; 说明以上计算实际上就是把每个字节的 16 进制值转变成 10 进制,然后取个位做注册码  

; 取值的长度是八位

```asm
00411F62   CALL    Electrob.00417486  
00411F67   ADD     ESP, 4  
00411F6A   MOV     EAX, 1  
00411F6F   POP     EDI  
00411F70   POP     ESI  
00411F71   POP     EBP  
00411F72   POP     EBX  
00411F73   ADD     ESP, 8  
00411F76   RETN
```

==========================================================================

到这里算法跟踪已完成,现在总结一下  

   条件--注册码的长度是 18 位

   算法总结:

       1.用字符串`"ElectrObalz"`和用户名,`E_mail`连接成一个新字符串  

       2.把新字符串的前一位 hex 值和后一位的 hex 值相加,值放在后一位.然后继续  

       3.把计算后的串从最后一位开始的 hex 值加前一位的 hex 值,值放在前一位上.  

       然后继续.  

       4.最后取前八个字节,把它的 16 进制值转变成 10 进制,然后取个位做注册码  

       把这个值替换在第 10-17 位的位置上,就是 18 位的注册码.也就是说,注册  

       码的 1-9 位和第 18 位是任何数字即可.另外注册码应该与 `E_mail` 无关,因为  

       计算注册码时只取前 8 字节.  

                                       by  `fxyang[OCN]`

                                       2003.4.19

   `4dd466d0e1e0cc123afbe504da2462ca`
