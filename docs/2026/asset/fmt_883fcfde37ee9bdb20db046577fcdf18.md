
```
0049AA23  |.  7C 05         |JL SHORT MP3\_RM\_C.0049AA2A
0049AA25  |.  83F9 39       |CMP ECX,39                              ;  与ASCII码39h比较，大于就失败
0049AA28  |.  7E 1A         |JLE SHORT MP3\_RM\_C.0049AA44             ;
0049AA2A  |>  6A 00         |PUSH 0                                  ; /Arg1 = 00000000
0049AA2C  |.  66:8B0D 38AB4>|MOV CX,WORD PTR DS:[49AB38]             ; |
0049AA33  |.  B2 02         |MOV DL,2                                ; |
0049AA35  |.  B8 44AB4900   |MOV EAX,MP3\_RM\_C.0049AB44               ; |invalid register code! please retry!
0049AA3A  |.  E8 1D7CF9FF   |CALL MP3\_RM\_C.0043265C                  ; \MP3\_RM\_C.0043265C
0049AA3F  |.  E9 AB000000   |JMP MP3\_RM\_C.0049AAEF
0049AA44  |>  42            |INC EDX
0049AA45  |.  48            |DEC EAX
0049AA46  |.^ 75 D0         \JNZ SHORT MP3\_RM\_C.0049AA18
0049AA48  |>  33F6          XOR ESI,ESI                              ;  ESI清零
0049AA4A  |.  8B45 FC       MOV EAX,[LOCAL.1]                        ;  用户名放到EAX
0049AA4D  |.  E8 EA99F6FF   CALL MP3\_RM\_C.0040443C                   ;  计算用户名长度，作为下面的循环次数
0049AA52  |.  85C0          TEST EAX,EAX
0049AA54  |.  7E 13         JLE SHORT MP3\_RM\_C.0049AA69
0049AA56  |.  BB 01000000   MOV EBX,1
0049AA5B  |>  8B55 FC       /MOV EDX,[LOCAL.1]                       ;  用户名地址放到ECX
0049AA5E  |.  0FB6541A FF   |MOVZX EDX,BYTE PTR DS:[EDX+EBX-1]
0049AA63  |.  03F2          |ADD ESI,EDX                             ;  依次取用户名的每一位的ASCII码相加，和放到ESI
0049AA65  |.  43            |INC EBX
0049AA66  |.  48            |DEC EAX
0049AA67  |.^ 75 F2         \JNZ SHORT MP3\_RM\_C.0049AA5B
0049AA69  |>  69C6 26740700 IMUL EAX,ESI,77426                       ;  EAX=ESI\*77426h
0049AA6F  |.  83C0 52       ADD EAX,52                               ;  EAX+=52h
0049AA72  |.  D1F8          SAR EAX,1                                ;  EAX算术右移一位，相当于EAX=EAX/2
0049AA74  |.  79 03         JNS SHORT MP3\_RM\_C.0049AA79
0049AA76  |.  83D0 00       ADC EAX,0
0049AA79  |>  8BF0          MOV ESI,EAX                              ;  把EAX复制给ESI
0049AA7B  |.  8B45 F8       MOV EAX,[LOCAL.2]                        ;  注册码放到EAX
0049AA7E  |.  E8 DDDFF6FF   CALL MP3\_RM\_C.00408A60                   ;  关键CALL，跟进
0049AA83  |.  3BF0          CMP ESI,EAX                              ;  判断EAX是否等于ESI，不等就跳
0049AA85  |.  75 53         JNZ SHORT MP3\_RM\_C.0049AADA              ;  不能跳，一跳就失败
0049AA87  |.  6A 00         PUSH 0                                   ; /Arg1 = 00000000
0049AA89  |.  66:8B0D 38AB4>MOV CX,WORD PTR DS:[49AB38]              ; |
0049AA90  |.  B2 02         MOV DL,2                                 ; |
0049AA92  |.  B8 74AB4900   MOV EAX,MP3\_RM\_C.0049AB74                ; |congratulation! you have successfully registered!
0049AA97  |.  E8 C07BF9FF   CALL MP3\_RM\_C.0043265C                   ; \MP3\_RM\_C.0043265C
0049AA9C  |.  A1 341B4A00   MOV EAX,DWORD PTR DS:[4A1B34]
0049AAA1  |.  C600 01       MOV BYTE PTR DS:[EAX],1
0049AAA4  |.  A1 2C1C4A00   MOV EAX,DWORD PTR DS:[4A1C2C]
0049AAA9  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
0049AAAB  |.  33C9          XOR ECX,ECX
0049AAAD  |.  BA 04000000   MOV EDX,4
0049AAB2  |.  8B18          MOV EBX,DWORD PTR DS:[EAX]
0049AAB4  |.  FF53 14       CALL DWORD PTR DS:[EBX+14]
0049AAB7  |.  8B15 341B4A00 MOV EDX,DWORD PTR DS:[4A1B34]            ;  MP3\_RM\_C.004A2D1B
0049AABD  |.  A1 2C1C4A00   MOV EAX,DWORD PTR DS:[4A1C2C]
0049AAC2  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
0049AAC4  |.  B9 01000000   MOV ECX,1
0049AAC9  |.  E8 7E18F8FF   CALL MP3\_RM\_C.0041C34C
0049AACE  |.  A1 082D4A00   MOV EAX,DWORD PTR DS:[4A2D08]
0049AAD3  |.  E8 1CF8FDFF   CALL MP3\_RM\_C.0047A2F4
0049AAD8  |.  EB 15         JMP SHORT MP3\_RM\_C.0049AAEF
0049AADA  |>  6A 00         PUSH 0                                   ; /Arg1 = 00000000
0049AADC  |.  66:8B0D 38AB4>MOV CX,WORD PTR DS:[49AB38]              ; |
0049AAE3  |.  B2 02         MOV DL,2                                 ; |
0049AAE5  |.  B8 44AB4900   MOV EAX,MP3\_RM\_C.0049AB44                ; |invalid register code! please retry!
0049AAEA  |.  E8 6D7BF9FF   CALL MP3\_RM\_C.0043265C                   ; \MP3\_RM\_C.0043265C
0049AAEF  |>  33C0          XOR EAX,EAX                              ;  (初始 cpu 选择)
0049AAF1  |.  5A            POP EDX
0049AAF2  |.  59            POP ECX
0049AAF3  |.  59            POP ECX
0049AAF4  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0049AAF7  |.  68 31AB4900   PUSH MP3\_RM\_C.0049AB31
0049AAFC  |>  8D45 E8       LEA EAX,[LOCAL.6]
0049AAFF  |.  E8 7896F6FF   CALL MP3\_RM\_C.0040417C
0049AB04  |.  8D45 EC       LEA EAX,[LOCAL.5]
0049AB07  |.  E8 7096F6FF   CALL MP3\_RM\_C.0040417C
0049AB0C  |.  8D45 F0       LEA EAX,[LOCAL.4]
0049AB0F  |.  E8 6896F6FF   CALL MP3\_RM\_C.0040417C
0049AB14  |.  8D45 F4       LEA EAX,[LOCAL.3]
0049AB17  |.  E8 6096F6FF   CALL MP3\_RM\_C.0040417C
0049AB1C  |.  8D45 F8       LEA EAX,[LOCAL.2]
0049AB1F  |.  BA 02000000   MOV EDX,2
0049AB24  |.  E8 7796F6FF   CALL MP3\_RM\_C.004041A0
0049AB29  \.  C3            RETN
```

走到`0049AA7E`这一行，关键，跟进

```
00408A60  /$  53            PUSH EBX
00408A61  |.  56            PUSH ESI
00408A62  |.  83C4 F4       ADD ESP,-0C
00408A65  |.  8BD8          MOV EBX,EAX                              ;  注册码放到EBX
00408A67  |.  8BD4          MOV EDX,ESP
00408A69  |.  8BC3          MOV EAX,EBX
00408A6B  |.  E8 F0A2FFFF   CALL MP3\_RM\_C.00402D60                   ;  关键，跟进
00408A70  |.  8BF0          MOV ESI,EAX
00408A72  |.  833C24 00     CMP DWORD PTR SS:[ESP],0
00408A76  |.  74 19         JE SHORT MP3\_RM\_C.00408A91               ;  跳了
00408A78  |.  895C24 04     MOV DWORD PTR SS:[ESP+4],EBX
00408A7C  |.  C64424 08 0B  MOV BYTE PTR SS:[ESP+8],0B
00408A81  |.  8D5424 04     LEA EDX,DWORD PTR SS:[ESP+4]
00408A85  |.  A1 781A4A00   MOV EAX,DWORD PTR DS:[4A1A78]
00408A8A  |.  33C9          XOR ECX,ECX
00408A8C  |.  E8 CBF8FFFF   CALL MP3\_RM\_C.0040835C
00408A91  |>  8BC6          MOV EAX,ESI
00408A93  |.  83C4 0C       ADD ESP,0C
00408A96  |.  5E            POP ESI                                  ;  ESI出栈
00408A97  |.  5B            POP EBX
00408A98  \.  C3            RETN
```

走到 `00408A6B` 这一行，关键`CALL`，跟进

```
00402D60  /$  53            PUSH EBX
00402D61  |.  56            PUSH ESI
00402D62  |.  57            PUSH EDI
00402D63  |.  89C6          MOV ESI,EAX                              ;  注册码放到ESI
00402D65  |.  50            PUSH EAX
00402D66  |.  85C0          TEST EAX,EAX
00402D68  |.  74 6C         JE SHORT MP3\_RM\_C.00402DD6
00402D6A  |.  31C0          XOR EAX,EAX                              ;  EAX清空
00402D6C  |.  31DB          XOR EBX,EBX                              ;  EBX清空
00402D6E  |.  BF CCCCCC0C   MOV EDI,0CCCCCCC                         ;  EDI赋值0CCCCCCC
00402D73  |>  8A1E          /MOV BL,BYTE PTR DS:[ESI]
00402D75  |.  46            |INC ESI                                 ;  ESI自加
00402D76  |.  80FB 20       |CMP BL,20                               ;  注册码的第一位与20h比较
00402D79  |.^ 74 F8         \JE SHORT MP3\_RM\_C.00402D73
00402D7B  |.  B5 00         MOV CH,0
00402D7D  |.  80FB 2D       CMP BL,2D                                ;  注册码第一位与2Dh比较
00402D80  |.  74 62         JE SHORT MP3\_RM\_C.00402DE4
00402D82  |.  80FB 2B       CMP BL,2B                                ;  注册码第一位与2Bh比较
00402D85  |.  74 5F         JE SHORT MP3\_RM\_C.00402DE6
00402D87  |>  80FB 24       CMP BL,24                                ;  注册码第一位与24h比较
00402D8A  |.  74 5F         JE SHORT MP3\_RM\_C.00402DEB
00402D8C  |.  80FB 78       CMP BL,78                                ;  注册码第一位与78h比较
00402D8F  |.  74 5A         JE SHORT MP3\_RM\_C.00402DEB
```
