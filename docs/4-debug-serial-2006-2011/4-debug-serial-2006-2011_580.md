
# HA_4UWMAMP3Converter_v592 算法分析

> 原文：[`www.pediy.com/kssd/pediy12/129910.html`](https://www.pediy.com/kssd/pediy12/129910.html)

## 前言

这个分析我打算是在 `PYG` 发的，我只是想证明我不是哪种把别人成果当成自己的成果的人。

精装友情通讯录算法分析这个算法分析我也看到别的爆破，一时冲动想看看这软件的算法是怎样的！

1. 我是用万能断点找的。
2. 因为算法分析，因此我也不具体讲怎么找到个这的。
3. 不会自己找找一些破解教程，好的我也不多说看我分析吧！

## 入口点代码分析

```assembly
0048DA60  /$  55             PUSH EBP
0048DA61  |.  8BEC           MOV EBP,ESP
0048DA63  |.  6A 00          PUSH 0
0048DA65  |.  6A 00          PUSH 0
0048DA67  |.  6A 00          PUSH 0
0048DA69  |.  6A 00          PUSH 0
0048DA6B  |.  6A 00          PUSH 0
0048DA6D  |.  53             PUSH EBX
0048DA6E  |.  56             PUSH ESI
0048DA6F  |.  894D F8        MOV DWORD PTR SS:[EBP-8],ECX
0048DA72  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
0048DA75  |.  8BF0           MOV ESI,EAX
0048DA77  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DA7A  |.  E8 0171F7FF    CALL WMAMP3Co.00404B80
0048DA7F  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0048DA82  |.  E8 F970F7FF    CALL WMAMP3Co.00404B80
0048DA87  |.  33C0           XOR EAX,EAX
0048DA89  |.  55             PUSH EBP
0048DA8A  |.  68 57DB4800    PUSH WMAMP3Co.0048DB57
0048DA8F  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
0048DA92  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
0048DA95  |.  33DB           XOR EBX,EBX
0048DA97  |.  33D2           XOR EDX,EDX
0048DA99  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DA9C  |.  E8 3372F7FF    CALL WMAMP3Co.00404CD4
0048DAA1  |.  85C0           TEST EAX,EAX
0048DAA3  |.  7E 0B          JLE SHORT WMAMP3Co.0048DAB0
0048DAA5  |.  8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
0048DAA8  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
0048DAAB  |.  E8 C86CF7FF    CALL WMAMP3Co.00404778
0048DAB0  |>  8D4D F4        LEA ECX,DWORD PTR SS:[EBP-C]
0048DAB3  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
0048DAB6  |.  8BC6           MOV EAX,ESI
0048DAB8  |.  E8 2F010000    CALL WMAMP3Co.0048DBEC             // 算法 CALL 跟进去看看
```

来到 `CALL WMAMP3Co.0048DBEC` 就可以看下面这些代码，经过分析就可以得到以下的结果。

## 关键函数 0048DBEC 分析

```assembly
0048DBEC  /$  55             PUSH EBP
0048DBED  |.  8BEC           MOV EBP,ESP
0048DBEF  |.  6A 00          PUSH 0
0048DBF1  |.  6A 00          PUSH 0
0048DBF3  |.  6A 00          PUSH 0
0048DBF5  |.  6A 00          PUSH 0
0048DBF7  |.  6A 00          PUSH 0
0048DBF9  |.  6A 00          PUSH 0
0048DBFB  |.  6A 00          PUSH 0
0048DBFD  |.  6A 00          PUSH 0
0048DBFF  |.  53             PUSH EBX
0048DC00  |.  56             PUSH ESI
0048DC01  |.  57             PUSH EDI
0048DC02  |.  8BD9           MOV EBX,ECX
0048DC04  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
0048DC07  |.  8BF8           MOV EDI,EAX
0048DC09  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC0C  |.  E8 6F6FF7FF    CALL WMAMP3Co.00404B80
0048DC11  |.  33C0           XOR EAX,EAX
0048DC13  |.  55             PUSH EBP
0048DC14  |.  68 47DD4800    PUSH WMAMP3Co.0048DD47
0048DC19  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
0048DC1C  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
0048DC1F  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
0048DC22  |.  BA 60DD4800    MOV EDX,WMAMP3Co.0048DD60         ;  ASCII "Jt^S0Mvx5C1"
0048DC27  |.  E8 746DF7FF    CALL WMAMP3Co.004049A0           ;  把用户名的固定字符串连起来我叫这个字符为 字符 1
0048DC2C  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC2F  |.  E8 646DF7FF    CALL WMAMP3Co.00404998           ;  字符串 1 的位数传给 EAX
0048DC34  |.  8BF0           MOV ESI,EAX                        ;  在给 ESI
0048DC36  |.  D1FE           SAR ESI,1                           ;  算术右移 1
0048DC38  |.  79 03          JNS SHORT WMAMP3Co.0048DC3D
0048DC3A  |.  83D6 00        ADC ESI,0
0048DC3D  |>  8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]
0048DC40  |.  50             PUSH EAX
0048DC41  |.  8BCE           MOV ECX,ESI                         ;  把 ESI 的值给 ECX 做取字串的参数
0048DC43  |.  BA 01000000    MOV EDX,1
0048DC48  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC4B  |.  E8 A06FF7FF    CALL WMAMP3Co.00404BF0             ;  在取字符串 1 的前 ECX 位做为字符串 2
0048DC50  |.  8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]
0048DC53  |.  50             PUSH EAX
0048DC54  |.  8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
0048DC57  |.  50             PUSH EAX
0048DC58  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC5B  |.  E8 386DF7FF    CALL WMAMP3Co.00404998
0048DC60  |.  8BC8           MOV ECX,EAX                         ;  把字符 1 位数做取字符串 3 最后一位的参数
0048DC62  |.  8D56 01        LEA EDX,DWORD PTR DS:[ESI+1]       ;  把取字符串 3 第一位的参数给 EDX
0048DC65  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC68  |.  E8 836FF7FF    CALL WMAMP3Co.00404BF0
0048DC6D  |.  8B55 EC        MOV EDX,DWORD PTR SS:[EBP-14]
0048DC70  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
0048DC73  |.  59             POP ECX
0048DC74  |.  E8 6B6DF7FF    CALL WMAMP3Co.004049E4             ;  把取出的字符串 3 和在取字符串 2 连起来
0048DC79  |.  8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
0048DC7C  |.  50             PUSH EAX
0048DC7D  |.  B9 0A000000    MOV ECX,0A
0048DC82  |.  BA 01000000    MOV EDX,1
0048DC87  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC8A  |.  E8 616FF7FF    CALL WMAMP3Co.00404BF0             ;  取字串的前 10 位为字符串 4
0048DC8F  |.  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
0048DC92  |.  50             PUSH EAX
0048DC93  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DC96  |.  E8 FD6CF7FF    CALL WMAMP3Co.00404998
0048DC9B  |.  8BC8           MOV ECX,EAX
0048DC9D  |.  BA 06000000    MOV EDX,6
0048DCA2  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048DCA5  |.  E8 466FF7FF    CALL WMAMP3Co.00404BF0             ;  从字符串的第 6 位开始取一直到最后一位为字符串 5
0048DCAA  |.  837D F4 00     CMP DWORD PTR SS:[EBP-C],0
0048DCAE  |.  75 10          JNZ SHORT WMAMP3Co.0048DCC0
0048DCB0  |.  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
0048DCB3  |.  BA 60DD4800    MOV EDX,WMAMP3Co.0048DD60          ;  ASCII "Jt^S0Mvx5C1"
0048DCB8  |.  8B4D F8        MOV ECX,DWORD PTR SS:[EBP-8]
0048DCBB  |.  E8 246DF7FF    CALL WMAMP3Co.004049E4
0048DCC0  |>  53             PUSH EBX
0048DCC1  |.  8B4D F4        MOV ECX,DWORD PTR SS:[EBP-C]
0048DCC4  |.  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
0048DCC7  |.  8BC7           MOV EAX,EDI
0048DCC9  |.  E8 92F0FFFF     CALL WMAMP3Co.0048CD60             ;  这是关键的算法 CALL 要进看看哪就来到
```

`WMAMP3Co.0048CD60` 是关键算法，需要注意分析。

## 核心算法函数 0048CD60 分析

```assembly
0048CD60  /$  55             PUSH EBP
0048CD61  |.  8BEC           MOV EBP,ESP
0048CD63  |.  83C4 E0        ADD ESP,-20
0048CD66  |.  53             PUSH EBX
0048CD67  |.  56             PUSH ESI
0048CD68  |.  57             PUSH EDI
0048CD69  |.  33DB           XOR EBX,EBX
0048CD6B  |.  895D E0        MOV DWORD PTR SS:[EBP-20],EBX
0048CD6E  |.  895D F0        MOV DWORD PTR SS:[EBP-10],EBX
0048CD71  |.  894D F8        MOV DWORD PTR SS:[EBP-8],ECX
0048CD74  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
0048CD77  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
0048CD7A  |.  E8 017EF7FF    CALL WMAMP3Co.00404B80
0048CD7F  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0048CD82  |.  E8 F97DF7FF    CALL WMAMP3Co.00404B80
0048CD87  |.  33C0           XOR EAX,EAX
0048CD89  |.  55             PUSH EBP
0048CD8A  |.  68 7CCE4800    PUSH WMAMP3Co.0048CE7C
0048CD8F  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
0048CD92  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
0048CD95  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0048CD98  |.  E8 FB7BF7FF    CALL WMAMP3Co.00404998
0048CD9D  |.  8945 F4        MOV DWORD PTR SS:[EBP-C],EAX
```



# 汇编代码段 1

```
0048CDA0  |.  837D F4 00    CMP DWORD PTR SS:[EBP-C],0
0048CDA4  |.  75 0D         JNZ SHORT WMAMP3Co.0048CDB3
0048CDA6  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
0048CDA9  |.  BA 94CE4800   MOV EDX,WMAMP3Co.0048CE94               ;  ASCII "Think Space"
0048CDAE  |.  E8 C579F7FF   CALL WMAMP3Co.00404778
0048CDB3  |>  33F6          XOR ESI,ESI                             ;  ESI 清 0
0048CDB5  |.  BB 00010000   MOV EBX,100                             ;  EBX 置 16 进制的 100
0048CDBA  |.  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
0048CDBD  |.  50            PUSH EAX
0048CDBE  |.  C745 E4 00010>MOV DWORD PTR SS:[EBP-1C],100
0048CDC5  |.  C645 E8 00    MOV BYTE PTR SS:[EBP-18],0
0048CDC9  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
0048CDCC  |.  33C9          XOR ECX,ECX
0048CDCE  |.  B8 A8CE4800   MOV EAX,WMAMP3Co.0048CEA8               ;  ASCII "%1.2x"
0048CDD3  |.  E8 30CFF7FF   CALL WMAMP3Co.00409D08
0048CDD8  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]             ;  这是前 10 位的字符串给 EAX
0048CDDB  |.  E8 B87BF7FF   CALL WMAMP3Co.00404998                  ;  算出字串的位数给 EAX
0048CDE0  |.  8BF8          MOV EDI,EAX                             ;  把 EAX 的值给 EDI 做下面的循环的参数用
0048CDE2  |.  85FF          TEST EDI,EDI
0048CDE4  |.  7E 60         JLE SHORT WMAMP3Co.0048CE46
0048CDE6  |.  C745 EC 01000>MOV DWORD PTR SS:[EBP-14],1             ;  SS;[EBP-14]置 1
0048CDED  |>  8B45 FC       /MOV EAX,DWORD PTR SS:[EBP-4]            ;  把字符串 4 给 EAX
0048CDF0  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]           ;  SS；[EBP-14]传给 EDX 做取字符串的参数
0048CDF3  |.  0FB64410 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EDX-1]       ;  循环取出字符串 4 的字符给 EAX
0048CDF8  |.  03C3          |ADD EAX,EBX                             ;  EAX 加 16 进制的 100
0048CDFA  |.  B9 FF000000   |MOV ECX,0FF                             ;  ECX 置 0FF
0048CDFF  |.  99            |CDQ                                     ;  EDX 清 0
0048CE00  |.  F7F9          |IDIV ECX                                ;  EAX 带符号数除 ECX
0048CE02  |.  8BDA          |MOV EBX,EDX                             ;  把佘数给 EBX
0048CE04  |.  3B75 F4       |CMP ESI,DWORD PTR SS:[EBP-C]
0048CE07  |.  7D 03         |JGE SHORT WMAMP3Co.0048CE0C
0048CE09  |.  46            |INC ESI                                 ;  ESI 自动加 1
0048CE0A  |.  EB 05         |JMP SHORT WMAMP3Co.0048CE11
0048CE0C  |>  BE 01000000   |MOV ESI,1
0048CE11  |>  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]            ;  把字符 5 给 EAX
0048CE14  |.  0FB64430 FF   |MOVZX EAX,BYTE PTR DS:[EAX+ESI-1]       ;  循环取出字符串 5 的字符给 EAX
0048CE19  |.  33D8          |XOR EBX,EAX                             ;  EBX 和 EAX 逻辑异或运算
0048CE1B  |.  8D45 E0       |LEA EAX,DWORD PTR SS:[EBP-20]
0048CE1E  |.  50            |PUSH EAX
0048CE1F  |.  895D E4       |MOV DWORD PTR SS:[EBP-1C],EBX           ;  把运算结果给 SS；[EBP-1C]
0048CE22  |.  C645 E8 00    |MOV BYTE PTR SS:[EBP-18],0
0048CE26  |.  8D55 E4       |LEA EDX,DWORD PTR SS:[EBP-1C]
0048CE29  |.  33C9          |XOR ECX,ECX                             ;  ECX 清 0
0048CE2B  |.  B8 A8CE4800   |MOV EAX,WMAMP3Co.0048CEA8              ;  ASCII "%1.2x"
0048CE30  |.  E8 D3CEF7FF   |CALL WMAMP3Co.00409D08
0048CE35  |.  8B55 E0       |MOV EDX,DWORD PTR SS:[EBP-20]
0048CE38  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]
0048CE3B  |.  E8 607BF7FF   |CALL WMAMP3Co.004049A0                 ;  100 加上运出字符串 6
0048CE40  |.  FF45 EC       |INC DWORD PTR SS:[EBP-14]
0048CE43  |.  4F            |DEC EDI
0048CE44  |.^ 75 A7         \JNZ SHORT WMAMP3Co.0048CDED
0048CE46  |>  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]
0048CE49  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0048CE4C  |.  E8 E378F7FF   CALL WMAMP3Co.00404734
0048CE51  |.  33C0          XOR EAX,EAX
0048CE53  |.  5A            POP EDX
0048CE54  |.  59            POP ECX
0048CE55  |.  59            POP ECX
0048CE56  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0048CE59  |.  68 83CE4800   PUSH WMAMP3Co.0048CE83
0048CE5E  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
0048CE61  |.  E8 7A78F7FF   CALL WMAMP3Co.004046E0
0048CE66  |.  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
0048CE69  |.  E8 7278F7FF   CALL WMAMP3Co.004046E0
0048CE6E  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
0048CE71  |.  BA 02000000   MOV EDX,2
0048CE76  |.  E8 8978F7FF   CALL WMAMP3Co.00404704
0048CE7B  \\. C3            RETN
0048CE7C  .^ E9 E371F7FF   JMP WMAMP3Co.00404064
0048CE81  .^ EB DB         JMP SHORT WMAMP3Co.0048CE5E
0048CE83  .  5F            POP EDI
0048CE84  .  5E            POP ESI
0048CE85  .  5B            POP EBX
0048CE86  .  8BE5          MOV ESP,EBP
0048CE88  .  5D            POP EBP
0048CE89  .  C2 0400       RETN 4
```

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

# 汇编代码段 2

```
0048DCCE  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
0048DCD1  |.  50            PUSH EAX
0048DCD2  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]                ;  把字符 7 放入 EAX
0048DCD4  |.  B9 05000000   MOV ECX,5                                 ;  把参数 5 放到 ECX 这个参数控制取字位数
0048DCD9  |.  BA 01000000   MOV EDX,1                                 ;  把参数 1 放到 EDX 从第几开始取
0048DCDE  |.  E8 0D6FF7FF   CALL WMAMP3Co.00404BF0                   ;  取字符串 7 的 1-5 位为字符串 8
0048DCE3  |.  FF75 E8       PUSH DWORD PTR SS:[EBP-18]
0048DCE6  |.  68 74DD4800   PUSH WMAMP3Co.0048DD74
0048DCEB  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
0048DCEE  |.  50            PUSH EAX
0048DCEF  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]
0048DCF1  |.  B9 05000000   MOV ECX,5                                 ;  又是取 5 位
0048DCF6  |.  BA 06000000   MOV EDX,6                                 ;  从第 6 位取起
0048DCFB  |.  E8 F06EF7FF   CALL WMAMP3Co.00404BF0                   ;  取出字符串 7 中的 5-10 为字符串 9
0048DD00  |.  FF75 E4       PUSH DWORD PTR SS:[EBP-1C]
0048DD03  |.  68 74DD4800   PUSH WMAMP3Co.0048DD74
0048DD08  |.  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
0048DD0B  |.  50            PUSH EAX
0048DD0C  |.  8B03          MOV EAX,DWORD PTR DS:[EBX]
0048DD0E  |.  B9 05000000   MOV ECX,5                                 ;  又是取 5 位
0048DD13  |.  BA 0B000000   MOV EDX,0B                                ;  从 11 位开始取起
0048DD18  |.  E8 D36EF7FF   CALL WMAMP3Co.00404BF0                   ;  取字符串 7 中的 11-16 为字符串 10
0048DD1D  |.  FF75 E0       PUSH DWORD PTR SS:[EBP-20]
0048DD20  |.  8BC3          MOV EAX,EBX
0048DD22  |.  BA 05000000   MOV EDX,5
0048DD27  |.  E8 2C6DF7FF   CALL WMAMP3Co.00404A58                   ;  把字符串 8。字符串 9 和字符串 10 用-串联起来组成注册码
0048DD2C  |.  33C0          XOR EAX,EAX
0048DD2E  |.  5A            POP EDX
0048DD2F  |.  59            POP ECX
0048DD30  |.  59            POP ECX
0048DD31  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0048DD34  |.  68 4EDD4800   PUSH WMAMP3Co.0048DD4E
0048DD39  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
0048DD3C  |.  BA 08000000   MOV EDX,8
0048DD41  |.  E8 BE69F7FF   CALL WMAMP3Co.00404704
0048DD46  \\. C3            RETN
0048DD47  .^ E9 1863F7FF   JMP WMAMP3Co.00404064
0048DD4C  .^ EB EB         JMP SHORT WMAMP3Co.0048DD39
0048DD4E  .  5F            POP EDI
0048DD4F  .  5E            POP ESI
0048DD50  .  5B            POP EBX
0048DD51  .  8BE5          MOV ESP,EBP
0048DD53  .  5D            POP EBP
0048DD54  .  C3            RETN
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

# 汇编代码段 3

```
0048DABD  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]              ;  真码
0048DAC0  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
```



```
0048DAC3  |.  E8 DCAFF7FF    CALL WMAMP3Co.00408AA4                 //关键 CALL  

0048DAC8  |.  85C0          TEST EAX,EAX                           //  关键跳转  

0048DACA  |.  75 41         JNZ SHORT WMAMP3Co.0048DB0D  

0048DACC  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]  

0048DACF  |.  8BC6          MOV EAX,ESI  

0048DAD1  |.  E8 DAF3FFFF   CALL WMAMP3Co.0048CEB0  

0048DAD6  |.  84C0          TEST AL,AL  

0048DAD8  |.  74 62         JE SHORT WMAMP3Co.0048DB3C  

0048DADA  |.  B3 01         MOV BL,1  

0048DADC  |.  6A 40         PUSH 40  

0048DADE  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]  

0048DAE1  |.  A1 ECEF4B00   MOV EAX,DWORD PTR DS:[4BEFEC]  

0048DAE6  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]  

0048DAE8  |.  E8 0B97FDFF   CALL WMAMP3Co.004671F8  

0048DAED  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]  

0048DAF0  |.  E8 9B70F7FF   CALL WMAMP3Co.00404B90  

0048DAF5  |.  50            PUSH EAX                                ; |Title  

0048DAF6  |.  68 68DB4800   PUSH WMAMP3Co.0048DB68                  ; |Text = "Registered successfully, Thanks for your registration."  

0048DAFB  |.  A1 ECEF4B00   MOV EAX,DWORD PTR DS:[4BEFEC]           ; |  

0048DB00  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]              ; |  

0048DB02  |.  8B40 30       MOV EAX,DWORD PTR DS:[EAX+30]           ; |  

0048DB05  |.  50            PUSH EAX                                ; |hOwner  

0048DB06  |.  E8 4D9BF7FF   CALL <JMP.&user32.MessageBoxA>          ; \MessageBoxA  

0048DB0B  |.  EB 2F         JMP SHORT WMAMP3Co.0048DB3C  

0048DB0D  |>   6A 10        PUSH 10  

0048DB0F  |.  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]  

0048DB12  |.  A1 ECEF4B00   MOV EAX,DWORD PTR DS:[4BEFEC]  

0048DB17  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]  

0048DB19  |.  E8 DA96FDFF   CALL WMAMP3Co.004671F8  

0048DB1E  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]  

0048DB21  |.  E8 6A70F7FF   CALL WMAMP3Co.00404B90  

0048DB26  |.  50            PUSH EAX                                ; |Title  

0048DB27  |.  68 A0DB4800   PUSH WMAMP3Co.0048DBA0                  ; |Text = "Invalid Registration Code! \r\nPlease enter an available Registration Code."  

0048DB2C  |.  A1 ECEF4B00   MOV EAX,DWORD PTR DS:[4BEFEC]           ; |  

0048DB31  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]              ; |  

0048DB33  |.  8B40 30       MOV EAX,DWORD PTR DS:[EAX+30]           ; |  

0048DB36  |.  50            PUSH EAX                                ; |hOwner  

0048DB37  |.  E8 1C9BF7FF   CALL <JMP.&user32.MessageBoxA>          ; \MessageBoxA  

0048DB3C  |>   33C0          XOR EAX,EAX  

0048DB3E  |.  5A            POP EDX  

0048DB3F  |.  59            POP ECX  

0048DB40  |.  59            POP ECX  

0048DB41  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX  

0048DB44  |.  68 5EDB4800   PUSH WMAMP3Co.0048DB5E  

0048DB49  |>   8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]  

0048DB4C  |.  BA 05000000   MOV EDX,5  

0048DB51  |.  E8 AE6BF7FF   CALL WMAMP3Co.00404704  

0048DB56  \.  C3            RETN  

0048DB57    .^ E9 0865F7FF   JMP WMAMP3Co.00404064  

0048DB5C    .^ EB EB        JMP SHORT WMAMP3Co.0048DB49  

0048DB5E    .  8BC3          MOV EAX,EBX  

0048DB60    .  5E            POP ESI  

0048DB61    .  5B            POP EBX  

0048DB62    .  8BE5          MOV ESP,EBP  

0048DB64    .  5D            POP EBP  

0048DB65    .  C3            RETN
```

我的用户名是：`bbs.chinapyg.com`

我输的假码是：`123456789`

算的注册码是：`1003A-FA58E-F1208`
