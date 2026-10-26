
# EncryptPE

## 2003.5.18 主程序脱壳

【脱文作者】 simonzh2000[US]

【使用工具】 Peid0.92, Ollydbg1.10(反Antidbg版), ImportREC1.60, LordPE

【破解平台】 Win2000SP4 English

【软件名称】 EncryptPE 2003.5.18

【软件简介】 老王的壳 2003.5.18

【加壳方式】 自己

【作者声明】 本笔记只用于学习交流, 初学Crack，只是感兴趣技术，没有其他目的, 如有不妥之处, 请谅解.

老王同志很大方, 免费给大家使用. 我也为大家抛快砖, 引点玉出来.

用 IsDebug 插件去掉 OD的调试器标志。

忽略除了 “INT3异常” 之外的其它异常, 添加“ 忽略0EEDFADE ”异常。

```
004B7000 >  60              PUSHAD  //进入OD后停在这
004B7001    9C              PUSHFD
004B7002    64:FF35 0000000>PUSH DWORD PTR FS:[0]
004B7009    E8 79010000     CALL EncryptP.004B7187
```

F9运行，程序会中断在INT3异常处，Shift+F9通过异常, 有时会出来警告, 恭喜, 你中奖了, 重新来过. 经过几次 INT3 异常后, 程序会在 0EEDFADE 处长时间异常，所以上面忽略了这个指定异常。上个 WC , 回来后, OD 已停下,

```
7119CF57     CC                   int3                  //异常, Shift+F9 过
7119CF58     90                   nop
7119CF59     64:8F05 00000000     pop dword ptr fs:[0]
7119CF60     C3                   retn

77F9FFE4    8B0424          MOV EAX,DWORD PTR SS:[ESP]  // C0000008 异常, 二次, shift+F9 过
77F9FFE7    8BE5            MOV ESP,EBP
77F9FFE9    5D              POP EBP
77F9FFEA    C3              RETN

7119CF57     CC                   int3                  //异常, 停，看看堆栈
7119CF58     90                   nop
7119CF59     64:8F05 00000000     pop dword ptr fs:[0]
7119CF60     C3                   retn

// Stack

0012FF98   0012FFE0  Pointer to next SEH record
0012FF9C   7119CE8D  SE handler                         // 到 7119C8ED 下断, Shift+F9

7119CE8D    53              PUSH EBX
7119CE8E    52              PUSH EDX
7119CE8F    8B5C24 14       MOV EBX,DWORD PTR SS:[ESP+14]            // CONTEXT 结构的指针
7119CE93    8B93 C4000000   MOV EDX,DWORD PTR DS:[EBX+C4]
7119CE99    8B83 C0000000   MOV EAX,DWORD PTR DS:[EBX+C0]
7119CE9F    A3 38F61B71     MOV DWORD PTR DS:[711BF638],EAX
7119CEA4    E8 8F040000     CALL V1200351.7119D338                   // F8
7119CEA9    9C              PUSHFD
7119CEAA    58              POP EAX
7119CEAB    A3 38F61B71     MOV DWORD PTR DS:[711BF638],EAX
7119CEB0    E8 83040000     CALL V1200351.7119D338                   // F8
7119CEB5    8B83 B8000000   MOV EAX,DWORD PTR DS:[EBX+B8]
7119CEBB    40              INC EAX
7119CEBC    8983 B8000000   MOV DWORD PTR DS:[EBX+B8],EAX
7119CEC2    8B4424 0C       MOV EAX,DWORD PTR SS:[ESP+C]
7119CEC6    8B00            MOV EAX,DWORD PTR DS:[EAX]
7119CEC8    3D 03000080     CMP EAX,80000003
7119CECD    75 71           JNZ SHORT V1200351.7119CF40
7119CECF    803D 54F61B71 0>CMP BYTE PTR DS:[711BF654],1
7119CED6    74 4F           JE SHORT V1200351.7119CF27
7119CED8    8B42 0C         MOV EAX,DWORD PTR DS:[EDX+C]
7119CEDB    8983 9C000000   MOV DWORD PTR DS:[EBX+9C],EAX
7119CEE1    8B42 10         MOV EAX,DWORD PTR DS:[EDX+10]
7119CEE4    8983 A0000000   MOV DWORD PTR DS:[EBX+A0],EAX
7119CEEA    8B42 14         MOV EAX,DWORD PTR DS:[EDX+14]
7119CEED    8983 B4000000   MOV DWORD PTR DS:[EBX+B4],EAX
7119CEF3    8B42 1C         MOV EAX,DWORD PTR DS:[EDX+1C]
7119CEF6    8983 A4000000   MOV DWORD PTR DS:[EBX+A4],EAX
7119CEFC    8B42 20         MOV EAX,DWORD PTR DS:[EDX+20]
7119CEFF    8983 A8000000   MOV DWORD PTR DS:[EBX+A8],EAX
7119CF05    8B42 24         MOV EAX,DWORD PTR DS:[EDX+24]
7119CF08    8983 AC000000   MOV DWORD PTR DS:[EBX+AC],EAX
7119CF0E    8B42 28         MOV EAX,DWORD PTR DS:[EDX+28]
7119CF11    8983 B0000000   MOV DWORD PTR DS:[EBX+B0],EAX            // EAX = 499780, 异常处理完毕这里继续, OEP
7119CF17    8B02            MOV EAX,DWORD PTR DS:[EDX]
7119CF19    8942 24         MOV DWORD PTR DS:[EDX+24],EAX
7119CF1C    89D0            MOV EAX,EDX
7119CF1E    83C0 24         ADD EAX,24
7119CF21    8983 C4000000   MOV DWORD PTR DS:[EBX+C4],EAX
7119CF27    31C0            XOR EAX,EAX
7119CF29    8943 04         MOV DWORD PTR DS:[EBX+4],EAX
7119CF2C    8943 08         MOV DWORD PTR DS:[EBX+8],EAX
7119CF2F    8943 0C         MOV DWORD PTR DS:[EBX+C],EAX
7119CF32    8943 10         MOV DWORD PTR DS:[EBX+10],EAX
7119CF35    C743 18 5501000>MOV DWORD PTR DS:[EBX+18],155
7119CF3C    5A              POP EDX
7119CF3D    5B              POP EBX
7119CF3E    C3              RETN

00499780    55              PUSH EBP                                 // 到这里下断, F9,  下面修复 IAT
00499781    8BEC            MOV EBP,ESP
00499783    83C4 F0         ADD ESP,-10
00499786    B8 98954900     MOV EAX,EncryptP.00499598
0049978B    E8 D4D3F6FF     CALL EncryptP.00406B64

// 往下找找, 发现程序里有很多这样的东西

00406AA0    90              NOP
00406AA1  - E9 8A70A000     JMP 00E0DB30
00406AA6    8BC0            MOV EAX,EAX
00406AA8    90              NOP
00406AA9  - E9 E26FA000     JMP 00E0DA90
00406AAE    8BC0            MOV EAX,EAX
00406AB0    90              NOP
00406AB1  - E9 4670A000     JMP 00E0DAFC
00406AB6    8BC0            MOV EAX,EAX
00406AB8    90              NOP
00406AB9  - E9 1270A000     JMP 00E0DAD0
00406ABE    8BC0            MOV EAX,EAX

// API

00406AA0    90              NOP
00406AA1  - E9 EE71A000     JMP 00E0DC94

...

7119D2FD    8B4424 0C       MOV EAX,DWORD PTR SS:[ESP+C]     // 如果JMP E0DC94
7119D301    89C3            MOV EBX,EAX                      // 那么 EAX=E0DC99
7119D303    83C0 02         ADD EAX,2
7119D306    8B00            MOV EAX,DWORD PTR DS:[EAX]
7119D308    8B00            MOV EAX,DWORD PTR DS:[EAX]
7119D30A    31D8            XOR EAX,EBX
7119D30C    894424 0C       MOV DWORD PTR SS:[ESP+C],EAX     // EAX 即 API
7119D310    8B00            MOV EAX,DWORD PTR DS:[EAX]
7119D312    3C CC           CMP AL,0CC                       // 检查1
7119D314    74 14           JE SHORT V1200351.7119D32A
7119D316    80FC CC         CMP AH,0CC                       // 检查2
7119D319    74 0F           JE SHORT V1200351.7119D32A
7119D31B    C1E8 10         SHR EAX,10
7119D31E    3C CC           CMP AL,0CC                       // 检查3
7119D320    74 08           JE SHORT V1200351.7119D32A
7119D322    80FC CC         CMP AH,0CC                       // 检查4
7119D325    74 03           JE SHORT V1200351.7119D32A
7119D327    EB 08           JMP SHORT V1200351.7119D331
7119D329  - E9 C60554F6     JMP 676DD8F4
7119D32E    1B71 01         SBB ESI,DWORD PTR DS:[ECX+1]
7119D331    5B              POP EBX
7119D332    58              POP EAX
7119D333    9D              POPFD
7119D334    C3              RETN

// 上面就是壳解密 API 的过程

// 写一段恢复 API 的补丁程序, 放到 7119CF60

7119CF60    60                 PUSHAD
7119CF61    B8 50124000        MOV EAX,401250                              ; // 搜索从 401250 开始
7119CF66    BA 00000101        MOV EDX,1010000                             ; // 把 API 放到 1010000 开始的空闲区域
7119CF6B    66:8138 90E9       CMP WORD PTR DS:[EAX],0E990                 ; // 90 E9 = NOP, JMP　XXXXXXX
7119CF70    0F85 2F000000      JNZ V1200351.7119CFA5
7119CF76    8BC8               MOV ECX,EAX                                 ; // [EAX] is  90 E9
7119CF78    8B40 02            MOV EAX,DWORD PTR DS:[EAX+2]                ; // EAX = XXXXXXXX
7119CF7B    03C1               ADD EAX,ECX
7119CF7D    83C0 06            ADD EAX,6
7119CF80    3D 00000070        CMP EAX,70000000                            ; // > 7000 0000 就是 API
7119CF85    0F87 0E000000      JA V1200351.7119CF99
7119CF8B    83C0 05            ADD EAX,5
7119CF8E    8BD8               MOV EBX,EAX
7119CF90    83C0 02            ADD EAX,2
7119CF93    8B00               MOV EAX,DWORD PTR DS:[EAX]
7119CF95    8B00               MOV EAX,DWORD PTR DS:[EAX]
7119CF97    33C3               XOR EAX,EBX                                 ; // < 7000 0000 的 API
7119CF99    8902               MOV DWORD PTR DS:[EDX],EAX                  ; // 保存 API Address
7119CF9B    83C2 04            ADD EDX,4
7119CF9E    8BC1               MOV EAX,ECX
7119CFA0    90                 NOP
7119CFA1    90                 NOP
7119CFA2    90                 NOP
7119CFA3    90                 NOP
7119CFA4    90                 NOP
7119CFA5    83C0 04            ADD EAX,4
```
