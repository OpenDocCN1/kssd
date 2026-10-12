
# Easy Karaoke Player 3.0.177

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-244.htm`](https://www.pediy.com/kssd/pediy08/pediy8-244.htm)

【破文作者】网游难民

【作者主页】`http://bbs.126sohu.com`

【破解工具】`PEiD`, `OD`

【破解平台】`Windows XP`

【软件名称】`Easy Karaoke Player 3.0.177`

【软件大小】`511KB`

【原版下载】`http://www.onlinedown.net/soft/46535.htm`

【保护方式】注册码，重启验证

【软件简介】`Easy Karaoke Player` 可以播放卡拉 ok 或录制卡拉 ok 歌曲到 `wav` 檔。它可以利用其内建的强大的多媒体播放器来播放所有的多媒体格式档案。如 `MP3`, `MIDI`, `KAR`, `WAV`, `WMA`, `AC3`, `OGG`, `MPA`, `MP2`, `AU`, `AIF`, `SND`，等等。并且支持下列影像格式 `VCD`, `MPEG`, `AVI`, `WMV`, `ASF`, `OGM`, `DIVX`, `DAT`, `SVCD`

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 详细过程

### 一、查壳
用 `PEiD` 查壳为 `Microsoft Visual C++ 7.0`

### 二、逆向分析
运行软件，输入试练码，点确定弹出对话框"Registration failed!"，退出软件

再用 `OD` 截入脱壳后的程序，右键超级字串查找，找到"Registration failed!"，双击进入程序代码段，往上查找。

### 三、汇编代码
```
00410CC0   .  83EC 0C       SUB ESP,0C------这里下断
00410CC3   .  55            PUSH EBP
00410CC4   .  56            PUSH ESI
00410CC5   .  57            PUSH EDI
00410CC6   .  BF 01000000   MOV EDI,1
00410CCB   .  57            PUSH EDI
00410CCC   .  8BF1          MOV ESI,ECX
00410CCE   .  E8 F4CA0100   CALL karaoke.0042D7C7
00410CD3   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]            ;   用户名地址
00410CD6   .  8B68 F4       MOV EBP,DWORD PTR DS:[EAX-C]             ;   用户名位数
00410CD9   .  83FD 02       CMP EBP,2                                ;  判断用户名是否大于两位
00410CDC   .  7D 15         JGE SHORT karaoke.00410CF3               ;  大于就跳
00410CDE   .  6A 00         PUSH 0
00410CE0   .  6A 00         PUSH 0
00410CE2   .  68 C4194400   PUSH karaoke.004419C4                    ;  please input correct user name!
00410CE7   .  E8 E72C0200   CALL karaoke.004339D3
00410CEC   .  5F            POP EDI
00410CED   .  5E            POP ESI
00410CEE   .  5D            POP EBP
00410CEF   .  83C4 0C       ADD ESP,0C
00410CF2   .  C3            RETN
00410CF3   >  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]            ;  试练码地址
00410CF6   .  8379 F4 08    CMP DWORD PTR DS:[ECX-C],8               ;  试练码是否大于 8 位
00410CFA   .  7D 15         JGE SHORT karaoke.00410D11               ;  大于就跳
00410CFC   .  6A 00         PUSH 0
00410CFE   .  6A 00         PUSH 0
00410D00   .  68 9C194400   PUSH karaoke.0044199C                    ;  please input correct registration code!
00410D05   .  E8 C92C0200   CALL karaoke.004339D3
00410D0A   .  5F            POP EDI
00410D0B   .  5E            POP ESI
00410D0C   .  5D            POP EBP
00410D0D   .  83C4 0C       ADD ESP,0C
00410D10   .  C3            RETN
00410D11   >  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]            ;  用户名地址
00410D14   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]             ;  用户名位数
00410D17   .  85C9          TEST ECX,ECX
00410D19   .  7D 0A         JGE SHORT karaoke.00410D25
00410D1B   .  68 57000780   PUSH 80070057
00410D20   .  E8 EB07FFFF   CALL karaoke.00401510
00410D25   >  8A10          MOV DL,BYTE PTR DS:[EAX]                 ;  用户名第一位 ascii 送 dl
00410D27   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00410D2A   .  3978 F4       CMP DWORD PTR DS:[EAX-C],EDI
00410D2D   .  7D 0A         JGE SHORT karaoke.00410D39
00410D2F   .  68 57000780   PUSH 80070057
00410D34   .  E8 D707FFFF   CALL karaoke.00401510
00410D39   >  8A40 01       MOV AL,BYTE PTR DS:[EAX+1]               ;  取用户名第 2 位
00410D3C   .  884424 0E     MOV BYTE PTR SS:[ESP+E],AL               ;  用户名第 2 位 ascii 放到 ss:[esp+E]
00410D40   .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]            ;  用户名地址
00410D43   .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-C]             ;  用户名位数
00410D46   .  85C9          TEST ECX,ECX
00410D48   .  7D 0A         JGE SHORT karaoke.00410D54
00410D4A   .  68 57000780   PUSH 80070057
00410D4F   .  E8 BC07FFFF   CALL karaoke.00401510
00410D54   >  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+70]            ;  用户名地址
00410D57   .  53            PUSH EBX
00410D58   .  8A18          MOV BL,BYTE PTR DS:[EAX]                 ;  用户名第 1 位 ascii 送 bl,见 410d25
00410D5A   .  3979 F4       CMP DWORD PTR DS:[ECX-C],EDI
00410D5D   .  7D 0A         JGE SHORT karaoke.00410D69
00410D5F   .  68 57000780   PUSH 80070057
00410D64   .  E8 A707FFFF   CALL karaoke.00401510
00410D69   >  0FB6C2        MOVZX EAX,DL                             ;  用户名第 1 位 ascii 送 eax,见 410d25
00410D6C   .  83C8 4B       OR EAX,4B                                 ;  用户名第 1 位 ascii 码与 4bh 逻辑或运算,结果放入 eax
00410D6F   .  99            CDQ                                       ;  把 EAX 中的字的符号扩展到 EDX 中去
00410D70   .  BF 0A000000   MOV EDI,0A                                ;  把 EDI 的值设置为 Ah
00410D75   .  F7FF          IDIV EDI                                  ;  第 1 位用户名的 ascii 与 4bh 逻辑或运算的值除以 A，余数放入 dl 中
00410D77   .  0FB64424 12   MOVZX EAX,BYTE PTR SS:[ESP+12]           ;  用户名的第 2 位 ascii 码的值送到 eax
00410D7C   .  83C8 41       OR EAX,41                                 ;  用户名第 2 位 ascii 码与 41h 逻辑或运算，结果放入 eax
00410D7F   .  885424 16     MOV BYTE PTR SS:[ESP+16],DL              ;  第 1 位用户名的 ascii 与 4bh 逻辑或运算的值除以 A 的余数放入 SS:[ESP+16]中
00410D83   .  99            CDQ                                       ;  把 EAX 中的字的符号扩展到 EDX 中去
00410D84   .  F7FF          IDIV EDI                                  ;  用户名第 2 位的 ascii 与 41h 逻辑或运算的值除以 A，余数放入 dl 中
00410D86   .  0FB6C3        MOVZX EAX,BL                              ;  用户名第 1 位 ascii 送 eax,bl 被赋值见 410d25
00410D89   .  83C8 52       OR EAX,52                                 ;  用户名第 1 位 ascii 码与 52h 逻辑或运算,结果放入 eax
00410D8C   .  885424 12     MOV BYTE PTR SS:[ESP+12],DL              ;  用户名第 2 位的 ascii 与 41h 逻辑或运算的值除以 A 的余数放入 PTR SS:[ESP+12]中
00410D90   .  99            CDQ                                       ;  把 EAX 中的字的符号扩展到 EDX 中去
00410D91   .  F7FF          IDIV EDI                                  ;  第 1 位用户名的 ascii 与 52h 逻辑或运算的值除以 A，余数放入 dl 中
00410D93   .  0FB641 01     MOVZX EAX,BYTE PTR DS:[ECX+1]            ;  用户名第 2 位 ascii 送 eax
00410D97   .  83C8 4B       OR EAX,4B                                 ;  用户名第 2 位 ascii 码与 4bh 逻辑或运算,结果放入 eax
00410D9A   .  8BCF          MOV ECX,EDI                               ;  把 edi 的值赋予 ecx，即：edi=ecx=Ah
00410D9C   .  885424 17     MOV BYTE PTR SS:[ESP+17],DL              ;  第 1 位用户名的 ascii 与 52h 逻辑或运算的值除以 A 的余数放入 SS:[ESP+17]中
00410DA0   .  99            CDQ                                       ;  把 EAX 中的字的符号扩展到 EDX 中去
00410DA1   .  F7F9          IDIV ECX                                  ;  eax/ecx,即：用户名第 2 位的 ascii 与 4bh 逻辑或运算的值除以 A，余数放入 dl 中
00410DA3   .  33C0          XOR EAX,EAX                               ;  eax 清 0
00410DA5   .  33C9          XOR ECX,ECX                               ;  ecx 清 0
00410DA7   .  85ED          TEST EBP,EBP
00410DA9   .  885424 18     MOV BYTE PTR SS:[ESP+18],DL              ;  把用户名第 2 位的 ascii 与 4bh 逻辑或运算的值除以 A 的余数放入 SS:[ESP+18]中
00410DAD   .  7E 20         JLE SHORT karaoke.00410DCF
00410DAF   .  90            NOP
00410DB0   >  85C9          TEST ECX,ECX
00410DB2   .  0F8C D2000000 JL karaoke.00410E8A
00410DB8   .  8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]            ;  用户名地址
00410DBB   .  3B4F F4       CMP ECX,DWORD PTR DS:[EDI-C]             ;  用户名位数
00410DBE   .  0F8F C6000000 JG karaoke.00410E8A
00410DC4   .  0FB6140F      MOVZX EDX,BYTE PTR DS:[EDI+ECX]
00410DC8   .  03C2          ADD EAX,EDX
00410DCA   .  41            INC ECX
00410DCB   .  3BCD          CMP ECX,EBP
00410DCD   .^ 7C E1         JL SHORT karaoke.00410DB0                ;  一个回旋，把用户名逐个转换成 ASCII 码然后相加，结果保存到寄存器 EAX 中
```



```
00410DCF   >   8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]            ;  试练码地址  

00410DD2   .   8B51 F4       MOV EDX,DWORD PTR DS:[ECX-C]             ;  试练码位数  

00410DD5   .   85D2          TEST EDX,EDX  

00410DD7   .   7D 0A         JGE SHORT karaoke.00410DE3  

00410DD9   .   68 57000780   PUSH 80070057  

00410DDE   .   E8 2D07FFFF   CALL karaoke.00401510  

00410DE3   >   8A11          MOV DL,BYTE PTR DS:[ECX]                 ;  试练码第 1 位的 ASCII 码放入 dl 中  

00410DE5   .   8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]  

00410DE8   .   8379 F4 01    CMP DWORD PTR DS:[ECX-C],1  

00410DEC   .   885424 19     MOV BYTE PTR SS:[ESP+19],DL             ;  试练码第 1 位的 ASCII 码放入 SS:[ESP+19]中  

00410DF0   .   7D 0A         JGE SHORT karaoke.00410DFC  

00410DF2   .   68 57000780   PUSH 80070057  

00410DF7   .   E8 1407FFFF   CALL karaoke.00401510  

00410DFC   >   8A49 01       MOV CL,BYTE PTR DS:[ECX+1]               ;  试练码第 2 位的 ASCII 码放入 cl 中  

00410DFF   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E02   .   884C24 13     MOV BYTE PTR SS:[ESP+13],CL             ;  试练码第 2 位的 ASCII 码放入 SS:[ESP+13]中  

00410E06   .   837F F4 02    CMP DWORD PTR DS:[EDI-C],2  

00410E0A   .   7D 0A         JGE SHORT karaoke.00410E16  

00410E0C   .   68 57000780   PUSH 80070057  

00410E11   .   E8 FA06FFFF   CALL karaoke.00401510  

00410E16   >   8A4F 02       MOV CL,BYTE PTR DS:[EDI+2]               ;  试练码第 3 位的 ASCII 码放入 cl 中  

00410E19   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E1C   .   884C24 14     MOV BYTE PTR SS:[ESP+14],CL             ;  试练码第 3 位的 ASCII 码放入 PTR SS:[ESP+14]中  

00410E20   .   837F F4 03    CMP DWORD PTR DS:[EDI-C],3  

00410E24   .   7D 0A         JGE SHORT karaoke.00410E30  

00410E26   .   68 57000780   PUSH 80070057  

00410E2B   .   E8 E006FFFF   CALL karaoke.00401510  

00410E30   >   8A4F 03       MOV CL,BYTE PTR DS:[EDI+3]               ;  试练码第 4 位的 ASCII 码放入 cl 中  

00410E33   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E36   .   884C24 15     MOV BYTE PTR SS:[ESP+15],CL             ;  试练码第 4 位的 ASCII 码放入 SS:[ESP+15]中  

00410E3A   .   837F F4 04    CMP DWORD PTR DS:[EDI-C],4  

00410E3E   .   7D 0A         JGE SHORT karaoke.00410E4A  

00410E40   .   68 57000780   PUSH 80070057  

00410E45   .   E8 C606FFFF   CALL karaoke.00401510  

00410E4A   >   8A5F 04       MOV BL,BYTE PTR DS:[EDI+4]               ;  试练码第 5 位的 ASCII 码放入 bl 中  

00410E4D   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E50   .   837F F4 05    CMP DWORD PTR DS:[EDI-C],5  

00410E54   .   7D 0A         JGE SHORT karaoke.00410E60  

00410E56   .   68 57000780   PUSH 80070057  

00410E5B   .   E8 B006FFFF   CALL karaoke.00401510  

00410E60   >   8A4F 05       MOV CL,BYTE PTR DS:[EDI+5]               ;  试练码第 6 位的 ASCII 码放入 cl 中  

00410E63   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E66   .   884C24 1A     MOV BYTE PTR SS:[ESP+1A],CL             ;  试练码第 6 位的 ASCII 码放入 SS:[ESP+1A]中  

00410E6A   .   837F F4 06    CMP DWORD PTR DS:[EDI-C],6  

00410E6E   .   7D 0A         JGE SHORT karaoke.00410E7A  

00410E70   .   68 57000780   PUSH 80070057  

00410E75   .   E8 9606FFFF   CALL karaoke.00401510  

00410E7A   >   8A4F 06       MOV CL,BYTE PTR DS:[EDI+6]               ;  试练码第 7 位的 ASCII 码放入 cl 中  

00410E7D   .   8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410E80   .   884C24 1B     MOV BYTE PTR SS:[ESP+1B],CL             ;  试练码第 7 位的 ASCII 码放入 SS:[ESP+1B]中  

00410E84   .   837F F4 07    CMP DWORD PTR DS:[EDI-C],7  

00410E88   .   7D 0A         JGE SHORT karaoke.00410E94  

00410E8A   >   68 57000780   PUSH 80070057  

00410E8F   .   E8 7C06FFFF   CALL karaoke.00401510  

00410E94   >   8A4F 07       MOV CL,BYTE PTR DS:[EDI+7]               ;  试练码第 8 位的 ASCII 码放入 cl 中  

00410E97   .   0FB67C24 16   MOVZX EDI,BYTE PTR SS:[ESP+16]          ;  把第 1 位用户名的 ascii 与 4bh 逻辑或运算的值除以 A 的余数放入 edi(410d7f)  

00410E9C   .   0FB6D2        MOVZX EDX,DL                             ;  试练码第 1 位的 ASCII 码放入 edx 中(410de3)  

00410E9F   .   83EA 30       SUB EDX,30                               ;  试练码第 1 位的 ASCII 码减去 30，即变为第 1 位试练码放入 edx  

00410EA2   .   3BFA          CMP EDI,EDX                              ;  比较上面两个值  

00410EA4      75 48         JNZ SHORT karaoke.00410EEE               ;  不等则跳向下面的比较  

00410EA6   .   0FB65424 13   MOVZX EDX,BYTE PTR SS:[ESP+13]          ;  试练码第 2 位的 ASCII 码放入 edx 中  

00410EAB   .   0FB67C24 12   MOVZX EDI,BYTE PTR SS:[ESP+12]          ;  用户名第 2 位的 ascii 与 41h 逻辑或运算的值除以 A 的余数放入 edi 中  

00410EB0   .   83EA 30       SUB EDX,30                               ;  试练码第 2 位的 ASCII 码减去 30，即变为第 2 位试练码放入 edx  

00410EB3   .   3BFA          CMP EDI,EDX                              ;  比较上面两个值  

00410EB5   .   75 37         JNZ SHORT karaoke.00410EEE               ;  不等则跳向下面的比较  

00410EB7   .   0FB65424 14   MOVZX EDX,BYTE PTR SS:[ESP+14]          ;  试练码第 3 位的 ASCII 码放入 edx 中  

00410EBC   .   0FB67C24 17   MOVZX EDI,BYTE PTR SS:[ESP+17]          ;  第 1 位用户名的 ascii 与 52h 逻辑或运算的值除以 A 的余数放入 edi 中  

00410EC1   .   83EA 30       SUB EDX,30                               ;  试练码第 3 位的 ASCII 码减去 30，即变为第 3 位试练码放入 edx  

00410EC4   .   3BFA          CMP EDI,EDX                              ;  比较上面两个值  

00410EC6   .   75 26         JNZ SHORT karaoke.00410EEE               ;  不等则跳向下面的比较  

00410EC8   .   0FB65424 15   MOVZX EDX,BYTE PTR SS:[ESP+15]          ;  试练码第 4 位的 ASCII 码放入 edx 中  

00410ECD   .   0FB67C24 18   MOVZX EDI,BYTE PTR SS:[ESP+18]          ;  把用户名第 2 位的 ascii 与 4bh 逻辑或运算的值除以 A 的余数放入 edi 中  

00410ED2   .   83EA 30       SUB EDX,30                               ;  试练码第 4 位的 ASCII 码减去 30，即变为第 4 位试练码放入 edx  

00410ED5   .   3BFA          CMP EDI,EDX                              ;  比较上面两个值  

00410ED7   .   75 15         JNZ SHORT karaoke.00410EEE               ;  不等则跳向下面的比较  

00410ED9   .   99            CDQ                                      ;  把 EAX 中的字的符号扩展到 EDX 中去  

00410EDA   .   BF 0A000000   MOV EDI,0A                               ;  给 edi 赋值 Ah  

00410EDF   .   F7FF          IDIV EDI                                 ;  eax（用户名各位 ascii 累加值）除以 A，把余数放入 dl  

00410EE1   .   0FB6C2        MOVZX EAX,DL                             ;  用户名各位 ascii 累加值除以 A 的余数放入 eax  

00410EE4   .   0FB6D3        MOVZX EDX,BL                             ;  试练码第 5 位的 ASCII 码放入 edx 中  

00410EE7   .   83EA 30       SUB EDX,30                               ;  试练码第 5 位的 ASCII 码减去 30，即变为第 5 位试练码放入 edx  

00410EEA   .   3BC2          CMP EAX,EDX                              ;  比较上面两个值  

00410EEC      74 39         JE SHORT karaoke.00410F27                ;  不等则继续下面比较，相等则跳向成功-----关键跳  

00410EEE   >   8A4424 19     MOV AL,BYTE PTR SS:[ESP+19]             ;  试练码第 1 位的 ASCII 码放入 al 中  

00410EF2   .   B2 36         MOV DL,36                                ;  给 dl 赋值 36  

00410EF4   .   3AC2          CMP AL,DL                                ;  比较上面两个值  

00410EF6      0F85 81000000 JNZ karaoke.00410F7D                     ;  不相等则跳向注册失败处  

00410EFC   .   B0 33         MOV AL,33                                ;  给 al 赋值 33  

00410EFE   .   384424 13     CMP BYTE PTR SS:[ESP+13],AL             ;  试练码第 2 位的 ASCII 码和 33 比较  

00410F02      75 79         JNZ SHORT karaoke.00410F7D               ;  不相等则跳向注册失败处  

00410F04   .   384424 14     CMP BYTE PTR SS:[ESP+14],AL             ;  试练码第 3 位的 ASCII 码和 33 比较  

00410F08      75 73         JNZ SHORT karaoke.00410F7D               ;  不相等则跳向注册失败处  

00410F0A   .   385424 15     CMP BYTE PTR SS:[ESP+15],DL             ;  试练码第 4 位的 ASCII 码和 36 比较  

00410F0E      75 6D         JNZ SHORT karaoke.00410F7D               ;  不相等则跳向注册失败处  

00410F10   .   80FB 32       CMP BL,32                                ;  试练码第 5 位的 ASCII 码和 32 比较  

00410F13      75 68         JNZ SHORT karaoke.00410F7D               ;  不相等则跳向注册失败处  

00410F15   .   807C24 1A 31  CMP BYTE PTR SS:[ESP+1A],31             ;  试练码第 6 位的 ASCII 码和 31 比较  
```



```assembly
00410F1A      75 61         JNZ SHORT karaoke.00410F7D                 ;  不相等则跳向注册失败处  

00410F1C   .  807C24 1B 34  CMP BYTE PTR SS:[ESP+1B],34              ;  试练码第 7 位的 ASCII 码和 34 比较  

00410F21      75 5A         JNZ SHORT karaoke.00410F7D                 ;  不相等则跳向注册失败处  

00410F23   .  3ACA         CMP CL,DL                                  ;  试练码第 8 位的 ASCII 码和 36（410ef2）比较  

00410F25      75 56         JNZ SHORT karaoke.00410F7D                 ;  不等则跳向注册失败处，相等则跳向成功-----关键跳  

00410F27   >  6A 00         PUSH 0  

00410F29   .  6A 00         PUSH 0  

00410F2B   .  68 80194400   PUSH karaoke.00441980                      ;  registration has succeeded!  

00410F30   .  E8 9E2A0200   CALL karaoke.004339D3  

00410F35   .  8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]  

00410F38   .  E8 14900200   CALL karaoke.00439F51  

00410F3D   .  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]  

00410F40   .  57            PUSH EDI                                   ;  /Arg3  

00410F41   .  68 60004400   PUSH karaoke.00440060                      ;  |username  

00410F46   .  68 9CFD4300   PUSH karaoke.0043FD9C                      ;  |option  

00410F4B   .  8BC8          MOV ECX,EAX                                ;  |  

00410F4D   .  E8 982B0200   CALL karaoke.00433AEA                      ;  \\karaoke.00433AEA  

00410F52   .  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00410F55   .  E8 F78F0200   CALL karaoke.00439F51  

00410F5A   .  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]  

00410F5D   .  57            PUSH EDI                                   ;  /Arg3  

00410F5E   .  68 4C004400   PUSH karaoke.0044004C                      ;  |registration_code  

00410F63   .  68 9CFD4300   PUSH karaoke.0043FD9C                      ;  |option  

00410F68   .  8BC8          MOV ECX,EAX                                ;  |  

00410F6A   .  E8 7B2B0200   CALL karaoke.00433AEA                      ;  \\karaoke.00433AEA  

00410F6F   .  5B            POP EBX  

00410F70   .  5F            POP EDI  

00410F71   .  8BCE          MOV ECX,ESI  

00410F73   .  5E            POP ESI  

00410F74   .  5D            POP EBP  

00410F75   .  83C4 0C       ADD ESP,0C  

00410F78   .  E9 57B00100   JMP karaoke.0042BFD4  

00410F7D   >  6A 00         PUSH 0  

00410F7F   .  6A 00         PUSH 0  

00410F81   .  68 68194400   PUSH karaoke.00441968                      ;  registration failed!  

00410F86   .  E8 482A0200   CALL karaoke.004339D3  

00410F8B   .  5B            POP EBX  

00410F8C   .  5F            POP EDI  

00410F8D   .  5E            POP ESI  

00410F8E   .  5D            POP EBP  

00410F8F   .  83C4 0C       ADD ESP,0C  

00410F92   .  C3            RETN  
```

## 算法总结

第 1 位注册码：用户名第 1 位的 `ascii` 与 `4bh` 逻辑或运算的值除以 `A` 的余数。

第 2 位注册码：用户名第 2 位的 `ascii` 与 `41h` 逻辑或运算的值除以 `A` 的余数。

第 3 位注册码：用户名第 1 位的 `ascii` 与 `52h` 逻辑或运算的值除以 `A` 的余数。

第 4 位注册码：用户名第 2 位的 `ascii` 与 `4bh` 逻辑或运算的值除以 `A` 的余数。

第 5 位注册码：用户名各位 `ascii` 累加值除以 `A` 的余数。

其它位数字随意填写。

另外还有一组通用注册码：

前 8 位：`63362146`

其它位数字随意填写。

## 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
