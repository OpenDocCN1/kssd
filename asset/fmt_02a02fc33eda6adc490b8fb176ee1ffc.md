
```
004902CC   . |FF51 5C       CALL    DWORD PTR DS:[ECX+5C]  
004902CF   . |A1 EC2F4900   MOV     EAX, DWORD PTR DS:[492FEC]  
004902D4   . |8B00          MOV     EAX, DWORD PTR DS:[EAX]  
004902D6   . |8B80 D4020000 MOV     EAX, DWORD PTR DS:[EAX+2D4]  
004902DC   . |B2 01         MOV     DL, 1  
004902DE   . |8B08          MOV     ECX, DWORD PTR DS:[EAX]  
004902E0   . |FF51 5C       CALL    DWORD PTR DS:[ECX+5C]  
004902E3   . |A1 EC2F4900   MOV     EAX, DWORD PTR DS:[492FEC]  
004902E8   . |8B00          MOV     EAX, DWORD PTR DS:[EAX]  
004902EA   . |8B80 38030000 MOV     EAX, DWORD PTR DS:[EAX+338]  
004902F0   . |B2 01         MOV     DL, 1  
004902F2   . |8B08          MOV     ECX, DWORD PTR DS:[EAX]  
004902F4   . |FF51 5C       CALL    DWORD PTR DS:[ECX+5C]  
004902F7   . |A1 EC2F4900   MOV     EAX, DWORD PTR DS:[492FEC]  
004902FC   . |8B00          MOV     EAX, DWORD PTR DS:[EAX]  
004902FE   . |8B80 D0020000 MOV     EAX, DWORD PTR DS:[EAX+2D0]  
00490304   . |33D2          XOR     EDX, EDX  
00490306   . |E8 391BFAFF   CALL    1.00431E44  
0049030B   . |E9 25040000   JMP     1.00490735  
00490310   > \C745 D8 00010>MOV     DWORD PTR SS:[EBP-28], 100  
00490310   >  C745 D8 00010>MOV     DWORD PTR SS:[EBP-28], 100  
00490317   .  C745 D4 01000>MOV     DWORD PTR SS:[EBP-2C], 1  
0049031E   .  8D45 D8       LEA     EAX, DWORD PTR SS:[EBP-28]  
00490321   .  50            PUSH    EAX                                      ; /pBufSize = C2A0C040  
00490322   .  8D85 CFFEFFFF LEA     EAX, DWORD PTR SS:[EBP-131]             ; |  
00490328   .  50            PUSH    EAX                                      ; |Buffer = C2A0C040  
00490329   .  8D45 D4       LEA     EAX, DWORD PTR SS:[EBP-2C]              ; |  
0049032C   .  50            PUSH    EAX                                      ; |pvalueType = C2A0C040  
0049032D   .  6A 00         PUSH    0                                        ; |Reserved = NULL  
0049032F   .  68 3C084900   PUSH    1.0049083C                               ; |valueName = "ProducuID"  
00490334   .  8B45 D0       MOV     EAX, DWORD PTR SS:[EBP-30]              ; |  
00490337   .  50            PUSH    EAX                                      ; |hKey = C2A0C040  
00490338   .  E8 CB68F7FF   CALL    <JMP.&ADVAPI32.RegQueryvalueExA>        ; \RegQueryvalueExA  
0049033D   .  8D45 E8       LEA     EAX, DWORD PTR SS:[EBP-18]  
00490340   .  8D95 CFFEFFFF LEA     EDX, DWORD PTR SS:[EBP-131]  
00490346   .  B9 01010000   MOV     ECX, 101  
0049034B   .  E8 A03AF7FF   CALL    1.00403DF0  
00490350   .  C745 D8 00010>MOV     DWORD PTR SS:[EBP-28], 100  
00490357   .  C745 D4 01000>MOV     DWORD PTR SS:[EBP-2C], 1  
0049035E   .  8D45 D8       LEA     EAX, DWORD PTR SS:[EBP-28]  
00490361   .  50            PUSH    EAX                                      ; /pBufSize = C2A0C040  
00490362   .  8D85 CFFEFFFF LEA     EAX, DWORD PTR SS:[EBP-131]             ; |  
00490368   .  50            PUSH    EAX                                      ; |Buffer = C2A0C040  
00490369   .  8D45 D4       LEA     EAX, DWORD PTR SS:[EBP-2C]              ; |  
0049036C   .  50            PUSH    EAX                                      ; |pvalueType = C2A0C040  
0049036D   .  6A 00         PUSH    0                                        ; |Reserved = NULL  
0049036F   .  68 48084900   PUSH    1.00490848                               ; |valueName = "ProducuKey"  
00490374   .  8B45 D0       MOV     EAX, DWORD PTR SS:[EBP-30]              ; |  
00490377   .  50            PUSH    EAX                                      ; |hKey = C2A0C040  
00490378   .  E8 8B68F7FF   CALL    <JMP.&ADVAPI32.RegQueryvalueExA>        ; \RegQueryvalueExA  
0049037D   .  8D45 E4       LEA     EAX, DWORD PTR SS:[EBP-1C]  
00490380   .  8D95 CFFEFFFF LEA     EDX, DWORD PTR SS:[EBP-131]  
00490386   .  B9 01010000   MOV     ECX, 101  
0049038B   .  E8 603AF7FF   CALL    1.00403DF0  
00490390   .  E8 A799F7FF   CALL    1.00409D3C  
00490395   .  83C4 F8       ADD     ESP, -8  
00490398   .  DD1C24        FSTP    QWORD PTR SS:[ESP]  
0049039B   .  9B            WAIT  
0049039C   .  E8 4B8DF7FF   CALL    1.004090EC  
004903A1   .  8BD8          MOV     EBX, EAX  
004903A3   .  80BD CFFEFFFF>CMP     BYTE PTR SS:[EBP-131], 0  
004903AA   .  75 6C         JNZ     SHORT 1.00490418  
004903AC   .  8BC3          MOV     EAX, EBX  
004903AE   .  33D2          XOR     EDX, EDX  
004903B0   .  52            PUSH    EDX  
004903B1   .  50            PUSH    EAX  
004903B2   .  8D45 E8       LEA     EAX, DWORD PTR SS:[EBP-18]  
004903B5   .  E8 7E87F7FF   CALL    1.00408B38  
004903BA   .  8B45 E8       MOV     EAX, DWORD PTR SS:[EBP-18]  
004903BD   .  E8 7E3AF7FF   CALL    1.00403E40  
004903C2   .  50            PUSH    EAX  
004903C3   .  8B45 E8       MOV     EAX, DWORD PTR SS:[EBP-18]  
004903C6   .  E8 393CF7FF   CALL    1.00404004  
004903CB   .  8BD8          MOV     EBX, EAX                                 ; |  
004903CD   .  53            PUSH    EBX                                      ; |Buffer = 00000013  
004903CE   .  6A 01         PUSH    1                                        ; |valueType = REG_SZ  
004903D0   .  6A 00         PUSH    0                                        ; |Reserved = 0  
004903D2   .  68 3C084900   PUSH    1.0049083C                               ; |valueName = "ProducuID"  
004903D7   .  8B45 D0       MOV     EAX, DWORD PTR SS:[EBP-30]              ; |  
004903DA   .  50            PUSH    EAX                                      ; |hKey = C2A0C040  
004903DB   .  E8 3868F7FF   CALL    <JMP.&ADVAPI32.RegSetvalueExA>          ; \RegSetvalueExA  
004903E0   .  8B45 E8       MOV     EAX, DWORD PTR SS:[EBP-18]  
004903E3   .  E8 583AF7FF   CALL    1.00403E40  
004903E8   .  50            PUSH    EAX                                      ; /BufSize = C2A0C040 (-1029652416.)  
004903E9   .  53            PUSH    EBX                                      ; |Buffer = 00000013  
004903EA   .  6A 01         PUSH    1                                        ; |valueType = REG_SZ  
004903EC   .  6A 00         PUSH    0                                        ; |Reserved = 0  
004903EE   .  68 48084900   PUSH    1.00490848                               ; |valueName = "ProducuKey"  
004903F3   .  8B45 D0       MOV     EAX, DWORD PTR SS:[EBP-30]              ; |  
004903F6   .  50            PUSH    EAX                                      ; |hKey = C2A0C040  
004903F7   .  E8 1C68F7FF   CALL    <JMP.&ADVAPI32.RegSetvalueExA>          ; \RegSetvalueExA  
004903FC   .  A1 EC2F4900   MOV     EAX, DWORD PTR DS:[492FEC]  
00490401   .  8B00          MOV     EAX, DWORD PTR DS:[EAX]  
00490403   .  8B80 D0020000 MOV     EAX, DWORD PTR DS:[EAX+2D0]  
00490409   .  BA 5C084900   MOV     EDX, 1.0049085C                          ;  ASCII "Days(1/31) UnRegistered "  
0049040E   .  E8 311AFAFF   CALL    1.00431E44 
```

我也是看了楼哥发的每月话题的贴子后才想起用 `OD` 调试程序，第 1 次用感觉确实不错。以上疏忽之处在所难免，欢迎大家指正。

这个程序的算法不复杂，而且还有一个通用注册码`9I94-AXw2-29MU-w2R6`,程序注册成功后将注册信息倒序后保存在 `HKEY_LOCAL_MACHINE/Software/Microsoft/Windows/CurrentVersion`中的`ProducuID`和`ProducuKey`中，有兴趣的话各位可自己跟一下看看。

`-------------------------------------ShenGe-------------------------------------`

照例贴个笑话轻松一下！

### 半杯牛奶

作者:`zom`

键盘边，不知什么人留下了半杯牛奶。。。

乐观主义者说：那杯子有一半是满的。

悲观主义者说：那杯子有一半是空的。

`Pascal` 程序员说：它是整型还是浮点的？

`C` 程序员说：我要直接对着牛奶罐喝。

汇编程序员说：我要直接对着奶牛喝。

`Basicc` 程序员说：我还在哺乳期。

`Prolog` 程序员说：我喝了，别问我怎么喝。

共享游戏软件作家说：这杯免费，下杯要付钱。

安全顾问说：剩下的那半杯在那儿？

版权保护的疯子说：有人免费喝了半杯！

自由软件基金会说：那是奶牛对全人类的贡献！

`IBM` 说：从我们这里租杯子吧，我们会为你斟上我们认为最好的饮料。

`微软` 说：剩下的市场份额已经不值得 `Microsoft` 牛奶占有了！

`4dd466d0e1e0cc123afbe504da2462ca`
