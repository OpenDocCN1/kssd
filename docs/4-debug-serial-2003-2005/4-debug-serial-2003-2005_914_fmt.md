
# 数据转换器(通用版)2003.03---分析过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-271.htm`](https://www.pediy.com/kssd/pediy07/pediy7-271.htm)

**【破文标题】** 数据转换器(通用版)2003.03---分析过程  
**【破文作者】** `fcrjzmd`  
**【作者主页】** 未申请  
**【作者邮箱】** 未申请  
**【所属组织】** 无名无派，希望能找到好归宿!  
**【软件名称】** 数据转换器  
**【下载地址】** 不记得，在网上乱找到的!  
**【破解工具】** `OD`、`PEID`  
**【保护方式】** 注册码+`ASPack` `2.1` -> `Alexey` `Solodovnikov`  
**【软件限制】** `30` 天  
**【破解难度】** 简单  

\-----------------------------------------------------------------  

**【破解声明】**  

我是一只小菜鸟我要加倍努力，我的一点菜鸟心得愿与大家分享，破文内容必定会有破漏，我要学的还很多恳请前辈们指教!  

\-----------------------------------------------------------------  

**【破解分析】**  

用 `PEID` 查到 `ASPack` `2.1` -> `Alexey` `Solodovnikov`，这个简单手脱搞定了这个不用多说了言归正传了。用 `OD` 载入在 `007212E7` 下断吧!

```
00720EA0  /.  55             PUSH EBP
00720EA1  |.  8BEC           MOV EBP,ESP
00720EA3  |.  33C9           XOR ECX,ECX
00720EA5  |.  51             PUSH ECX
00720EA6  |.  51             PUSH ECX
00720EA7  |.  51             PUSH ECX
00720EA8  |.  51             PUSH ECX
00720EA9  |.  51             PUSH ECX
00720EAA  |.  51             PUSH ECX
00720EAB  |.  53             PUSH EBX
00720EAC  |.  8BD8           MOV EBX,EAX
00720EAE  |.  33C0           XOR EAX,EAX
00720EB0  |.  55             PUSH EBP
00720EB1  |.  68 BE0F7200    PUSH dumped\_.00720FBE
00720EB6  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
00720EB9  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
00720EBC  |.  8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
00720EBF  |.  8B83 04030000  MOV EAX,DWORD PTR DS:[EBX+304]
00720EC5  |.  E8 CED9D6FF    CALL dumped\_.0048E898                     ;  这个 CALL 检测注册版本的字符长度!
00720ECA  |.  837D FC 00     CMP DWORD PTR SS:[EBP-4],0                ;  和 0 做比较!
00720ECE  |.  75 0F          JNZ SHORT dumped\_.00720EDF               ;  不相等跳走!
00720ED0  |.  B8 D40F7200    MOV EAX,dumped\_.00720FD4                 ;  提示：请选择注册版本！
00720ED5  |.  E8 5EC10000    CALL dumped\_.0072D038
00720EDA  |.  E9 C4000000    JMP dumped\_.00720FA3
00720EDF  |>  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
00720EE2  |.  8B83 0C030000  MOV EAX,DWORD PTR DS:[EBX+30C]
00720EE8  |.  E8 ABD9D6FF    CALL dumped\_.0048E898                     ;  获取用户名及字符长度!
00720EED  |.  837D F8 00     CMP DWORD PTR SS:[EBP-8],0                ;  用户名长度和 0 做比较!
00720EF1  |.  75 1D          JNZ SHORT dumped\_.00720F10               ;  不为 0 跳走继续执行，反之失败!
00720EF3  |.  B8 F00F7200    MOV EAX,dumped\_.00720FF0                 ;  提示：用户名不能为空！
00720EF8  |.  E8 3BC10000    CALL dumped\_.0072D038
00720EFD  |.  8B83 0C030000  MOV EAX,DWORD PTR DS:[EBX+30C]
00720F03  |.  8B10           MOV EDX,DWORD PTR DS:[EAX]
00720F05  |.  FF92 C4000000  CALL DWORD PTR DS:[EDX+C4]
00720F0B  |.  E9 93000000    JMP dumped\_.00720FA3
00720F10  |>  8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
00720F13  |.  8B83 14030000  MOV EAX,DWORD PTR DS:[EBX+314]
00720F19  |.  E8 7AD9D6FF    CALL dumped\_.0048E898                     ;  获取假码及字符长度!
00720F1E  |.  837D F4 00     CMP DWORD PTR SS:[EBP-C],0                ;  假码长度和 0 做比较!
00720F22  |.  75 1A          JNZ SHORT dumped\_.00720F3E               ;  不为 0 跳走继续执行，反之失败!
00720F24  |.  B8 0C107200    MOV EAX,dumped\_.0072100C                 ;  提示：注册码不能为空！
00720F29  |.  E8 0AC10000    CALL dumped\_.0072D038
00720F2E  |.  8B83 14030000  MOV EAX,DWORD PTR DS:[EBX+314]
00720F34  |.  8B10           MOV EDX,DWORD PTR DS:[EAX]
00720F36  |.  FF92 C4000000  CALL DWORD PTR DS:[EDX+C4]
00720F3C  |.  EB 65          JMP SHORT dumped\_.00720FA3
00720F3E  |>  8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]
00720F41  |.  8B83 0C030000  MOV EAX,DWORD PTR DS:[EBX+30C]
00720F47  |.  E8 4CD9D6FF    CALL dumped\_.0048E898                     ;  获取用户名
00720F4C  |.  8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]              ;  EAX=fcrjzmd
00720F4F  |.  50             PUSH EAX                                   ;  压入用户名!
00720F50  |.  8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]
00720F53  |.  8B83 14030000  MOV EAX,DWORD PTR DS:[EBX+314]
00720F59  |.  E8 3AD9D6FF    CALL dumped\_.0048E898                     ;  获取假码!
00720F5E  |.  8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]              ;  EAX=假码
00720F61  |.  50             PUSH EAX                                   ;  压入假码!
00720F62  |.  8D55 E8        LEA EDX,DWORD PTR SS:[EBP-18]
00720F65  |.  8B83 FC020000  MOV EAX,DWORD PTR DS:[EBX+2FC]
00720F6B  |.  E8 28D9D6FF    CALL dumped\_.0048E898                     ;  获取机器码!
00720F70  |.  8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]              ;  EAX=机器码!
00720F73  |.  50             PUSH EAX                                   ;  压入机器码!
00720F74  |.  8B83 04030000  MOV EAX,DWORD PTR DS:[EBX+304]
00720F7A  |.  8B10           MOV EDX,DWORD PTR DS:[EAX]
00720F7C  |.  FF92 CC000000  CALL DWORD PTR DS:[EDX+CC]
00720F82  |.  8BD0           MOV EDX,EAX
00720F84  |.  8BC3           MOV EAX,EBX
00720F86  |.  59             POP ECX
00720F87  |.  E8 54040000    CALL dumped\_.007213E0                     ;  跟进*****关键 CALL
00720F8C  |.  84C0           TEST AL,AL
00720F8E  |.  74 09          JE SHORT dumped\_.00720F99
00720F90  |.  8BC3           MOV EAX,EBX
00720F92  |.  E8 0D050000    CALL dumped\_.007214A4
00720F97  |.  EB 0A          JMP SHORT dumped\_.00720FA3
00720F99  |>  B8 28107200    MOV EAX,dumped\_.00721028                  ;  注册信息不正确，请仔细检查！

\------------------------------------------------------------------------------------------------------
```

跟进 `00720F87`

```
007213E0  /$  55             PUSH EBP
007213E1  |.  8BEC           MOV EBP,ESP
007213E3  |.  6A 00          PUSH 0
007213E5  |.  6A 00          PUSH 0
007213E7  |.  6A 00          PUSH 0
007213E9  |.  53             PUSH EBX
007213EA  |.  56             PUSH ESI
007213EB  |.  57             PUSH EDI
007213EC  |.  894D FC        MOV DWORD PTR SS:[EBP-4],ECX
007213EF  |.  8BFA           MOV EDI,EDX
007213F1  |.  8BF0           MOV ESI,EAX
007213F3  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
007213F6  |.  E8 0139CEFF    CALL dumped\_.00404CFC                     ;  获取机器码!
007213FB  |.  8B45 0C        MOV EAX,DWORD PTR SS:[EBP+C]               ;  EAX=机器码
007213FE  |.  E8 F938CEFF    CALL dumped\_.00404CFC                     ;  获取用户名!
00721403  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]               ;  EAX=用户名!
00721406  |.  E8 F138CEFF    CALL dumped\_.00404CFC                     ;  获取假码!
0072140B  |.  33C0           XOR EAX,EAX                                ;  EXA=0
0072140D  |.  55             PUSH EBP
0072140E  |.  68 92147200    PUSH dumped\_.00721492
00721413  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
00721416  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
00721419  |.  33DB           XOR EBX,EBX                                ;  Switch (cases 0..1)
0072141B  |.  83EF 01        SUB EDI,1
0072141E  |.  72 04          JB SHORT dumped\_.00721424                 ;  小于跳转!
00721420  |.  74 26          JE SHORT dumped\_.00721448                 ;  相等跳转!
00721422  |.  EB 46          JMP SHORT dumped\_.0072146A
00721424  |>  6A 00          PUSH 0                                     ;  Case 0 of switch 00721419
00721426  |.  8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
00721429  |.  50             PUSH EAX
0072142A  |.  8B4D 0C        MOV ECX,DWORD PTR SS:[EBP+C]
0072142D  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
00721430  |.  8BC6           MOV EAX,ESI
00721432  |.  E8 11FCFFFF    CALL dumped\_.00721048
00721437  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0072143A  |.  8B55 08        MOV EDX,DWORD PTR SS:[EBP+8]
0072143D  |.  E8 1638CEFF    CALL dumped\_.00404C58
00721442  |.  75 26          JNZ SHORT dumped\_.0072146A
00721444  |.  B3 01          MOV BL,1
00721446  |.  EB 22          JMP SHORT dumped\_.0072146A
00721448  |>  6A 01          PUSH 1                                     ;  Case 1 of switch 00721419
```



```
0072144A  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
0072144D  |.  50            PUSH EAX
0072144E  |.  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]
00721451  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00721454  |.  8BC6          MOV EAX,ESI
00721456  |.  E8 EDFBFFFF   CALL dumped\_.00721048            ;  算法关键 CALL!跟进。。
0072145B  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]     ;  真码出现!
0072145E  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]     ;  假码出现!
00721461  |.  E8 F237CEFF   CALL dumped\_.00404C58            ;  对比 CALL
00721466  |.  75 02         JNZ SHORT dumped\_.0072146A       ;  传说中经典的对比语句!^\_^~!!
```

\-------------------------------------------------------------------------------------------------------

跟进 `00721456`

```
00721048  /$  55            PUSH EBP
00721049  |.  8BEC          MOV EBP,ESP
0072104B  |.  51            PUSH ECX
0072104C  |.  B9 11000000   MOV ECX,11                        ;  ECX=11
00721051  |>  6A 00         /PUSH 0
00721053  |.  6A 00         |PUSH 0
00721055  |.  49            |DEC ECX
00721056  |.^ 75 F9         \\JNZ SHORT dumped\_.00721051
00721058  |.  51            PUSH ECX
00721059  |.  874D FC       XCHG DWORD PTR SS:[EBP-4],ECX     ;  0012EFBC 和 ECX 内容交换
0072105C  |.  53            PUSH EBX
0072105D  |.  56            PUSH ESI
0072105E  |.  894D F8       MOV DWORD PTR SS:[EBP-8],ECX      ;  将 ECX=fcrjzmd(用户名)保存到 0012EFB8 里!
00721061  |.  8955 FC       MOV DWORD PTR SS:[EBP-4],EDX      ;  将 EDX=MRG254KBDBG5BP(机器码)保存到 0012EFBC 里!
00721064  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
00721067  |.  E8 903CCEFF   CALL dumped\_.00404CFC            ;  获取 fcrjzmd 用户名
0072106C  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
0072106F  |.  E8 883CCEFF   CALL dumped\_.00404CFC
00721074  |.  33C0          XOR EAX,EAX                        ;  EAX=0
00721076  |.  55            PUSH EBP
00721077  |.  68 83137200   PUSH dumped\_.00721383
0072107C  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
0072107F  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
00721082  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
00721085  |.  E8 823ACEFF   CALL dumped\_.00404B0C            ;  获取机器码的字符长度!
0072108A  |.  8BF0          MOV ESI,EAX                        ;  ESI=EAX=E(我机器码字符长度)
0072108C  |.  85F6          TEST ESI,ESI                        ;  测试 ESI
0072108E  |.  7E 24         JLE SHORT dumped\_.007210B4       ;  小于或等于则跳转!
00721090  |.  BB 01000000   MOV EBX,1                          ;  EBX=1
00721095  |>  8D55 E8       /LEA EDX,DWORD PTR SS:[EBP-18]     ;  将 0012EFA8 送入 EDX
00721098  |.  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
0072109B  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出机器码的 ASCII 码。
007210A0  |.  E8 178BCEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210A5  |.  8B55 E8       |MOV EDX,DWORD PTR SS:[EBP-18]    ;  转换后的 10 进制字符保存到 EDX
007210A8  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]    ;  将 0012EFB0 送入 EAX
007210AB  |.  E8 643ACEFF   |CALL dumped\_.00404B14           ;  这个 CALL 作用就是把转换好的 10 进制字符串合并起来，并保存在 0012EFB0 里!
007210B0  |.  43            |INC EBX                           ;  EBX+1=计数器
007210B1  |.  4E            |DEC ESI                           ;  ESI-1
007210B2  |.^ 75 E1         \\JNZ SHORT dumped\_.00721095     ;  取完为止!
007210B4  |>  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
007210B7  |.  E8 503ACEFF   CALL dumped\_.00404B0C            ;  获取用户名字符长度!
007210BC  |.  8BF0          MOV ESI,EAX                        ;  ESI=EAX=7(用户名长度)
007210BE  |.  85F6          TEST ESI,ESI                        ;  测试 ESI
007210C0  |.  7E 3F         JLE SHORT dumped\_.00721101       ;  小于或等于则跳转!
007210C2  |.  BB 01000000   MOV EBX,1                          ;  EBX=1
007210C7  |>  8D55 E4       /LEA EDX,DWORD PTR SS:[EBP-1C]     ;  将 0012EFA4 送入 EDX
007210CA  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
007210CD  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出用户名的 ASCII 码!
007210D2  |.  E8 E58ACEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210D7  |.  8B55 E4       |MOV EDX,DWORD PTR SS:[EBP-1C]    ;  转换后的 10 进制字符保存到 EDX
007210DA  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]    ;  将 0012EFB0 送入 EAX
007210DD  |.  E8 323ACEFF   |CALL dumped\_.00404B14           ;  这个 CALL 作用就是把上面机器码转换好的 10 进制字符串合并起来，并保存在 0012EFB0 里!
007210E2  |.  8D55 E0       |LEA EDX,DWORD PTR SS:[EBP-20]    ;  将 0012EFA0 送入 EDX
007210E5  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]     ;  EAX=fcrjzmd(用户名)
007210E8  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出用户名的 ASCII 码!
007210ED  |.  E8 CA8ACEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210F2  |.  8B55 E0       |MOV EDX,DWORD PTR SS:[EBP-20]    ;  转换后的 10 进制字符保存到 EDX
007210F5  |.  8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]    ;  将 0012EFAC 送入 EAX
007210F8  |.  E8 173ACEFF   |CALL dumped\_.00404B14           ;  ***这个 CALL 作用就是把用户名转换好的 10 进制字符串合并起来，并保存在
007210FD  |.  43            |INC EBX                           ;  EBX+1=计数器
007210FE  |.  4E            |DEC ESI                           ;  ESI-1
007210FF  |.^ 75 C6         \\JNZ SHORT dumped\_.007210C7     ;  取完为止!
00721101  |>  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]      ;  将 0012EFB4 送入 EAX
00721104  |.  50            PUSH EAX
00721105  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]     ;  EAX=778271505352756668667153668010299114106122109100(
00721108  |.  E8 FF39CEFF   CALL dumped\_.00404B0C            ;  获取常数 30
0072110D  |.  8BD0          MOV EDX,EAX                        ;  EDX=EAX=30
0072110F  |.  83EA 0A       SUB EDX,0A                          ;  EDX=EDX-0A=26
00721112  |.  B9 0A000000   MOV ECX,0A                          ;  ECX=0A
00721117  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]     ;  EAX=778271505352756668667153668010299114106122109100(
0072111A  |.  E8 4D3CCEFF   CALL dumped\_.00404D6C            ;  取机器码和用户名 10 进制合并字符串的 0612210910
0072111F  |.  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]      ;  EAX=1
00721122  |.  83E8 01       SUB EAX,1                          ;  EAX=EAX-1;  Switch (cases 0..1)
00721125  |.  72 0B         JB SHORT dumped\_.00721132        ;  小于跳转!
00721127  |.  0F84 D0000000 JE dumped\_.007211FD             ;  相等则跳!
0072112D  |.  E9 33020000   JMP dumped\_.00721365
00721132  |>  68 9C137200   PUSH dumped\_.0072139C            ;  ASCII "JLH-";  Case 0 of switch 00721122
00721137  |.  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
0072113A  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0072113D  |.  8A52 01       MOV DL,BYTE PTR DS:[EDX+1]
00721140  |.  E8 EF38CEFF   CALL dumped\_.00404A34
00721145  |.  FF75 DC       PUSH DWORD PTR SS:[EBP-24]
00721148  |.  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]
0072114B  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0072114E  |.  8A52 03       MOV DL,BYTE PTR DS:[EDX+3]
00721151  |.  E8 DE38CEFF   CALL dumped\_.00404A34
```



```asm
00721156  |.  FF75 D8       PUSH DWORD PTR SS:[EBP-28]  
00721159  |.  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]  
0072115C  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]  
0072115F  |.  8A52 05       MOV DL,BYTE PTR DS:[EDX+5]  
00721162  |.  E8 CD38CEFF   CALL dumped\_.00404A34  
00721167  |.  FF75 D4       PUSH DWORD PTR SS:[EBP-2C]  
0072116A  |.  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]  
0072116D  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]  
00721170  |.  8A52 07       MOV DL,BYTE PTR DS:[EDX+7]  
00721173  |.  E8 BC38CEFF   CALL dumped\_.00404A34  
00721178  |.  FF75 D0       PUSH DWORD PTR SS:[EBP-30]  
0072117B  |.  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  
0072117E  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]  
00721181  |.  8A52 09       MOV DL,BYTE PTR DS:[EDX+9]  
00721184  |.  E8 AB38CEFF   CALL dumped\_.00404A34  
00721189  |.  FF75 CC       PUSH DWORD PTR SS:[EBP-34]  
0072118C  |.  68 AC137200   PUSH dumped\_.007213AC                 ;  ASCII "-SOFT-"  
00721191  |.  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]  
00721194  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]  
00721197  |.  8A52 01       MOV DL,BYTE PTR DS:[EDX+1]  
0072119A  |.  E8 9538CEFF   CALL dumped\_.00404A34  
0072119F  |.  FF75 C8       PUSH DWORD PTR SS:[EBP-38]  
007211A2  |.  8D45 C4       LEA EAX,DWORD PTR SS:[EBP-3C]  
007211A5  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]  
007211A8  |.  8A52 03       MOV DL,BYTE PTR DS:[EDX+3]  
007211AB  |.  E8 8438CEFF   CALL dumped\_.00404A34  
007211B0  |.  FF75 C4       PUSH DWORD PTR SS:[EBP-3C]  
007211B3  |.  8D45 C0       LEA EAX,DWORD PTR SS:[EBP-40]  
007211B6  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]  
007211B9  |.  8A52 05       MOV DL,BYTE PTR DS:[EDX+5]  
007211BC  |.  E8 7338CEFF   CALL dumped\_.00404A34  
007211C1  |.  FF75 C0       PUSH DWORD PTR SS:[EBP-40]  
007211C4  |.  8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]  
007211C7  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]  
007211CA  |.  8A52 07       MOV DL,BYTE PTR DS:[EDX+7]  
007211CD  |.  E8 6238CEFF   CALL dumped\_.00404A34  
007211D2  |.  FF75 BC       PUSH DWORD PTR SS:[EBP-44]  
007211D5  |.  8D45 B8       LEA EAX,DWORD PTR SS:[EBP-48]  
007211D8  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]  
007211DB  |.  8A52 09       MOV DL,BYTE PTR DS:[EDX+9]  
007211DE  |.  E8 5138CEFF   CALL dumped\_.00404A34  
007211E3  |.  FF75 B8       PUSH DWORD PTR SS:[EBP-48]  
007211E6  |.  68 BC137200   PUSH dumped\_.007213BC                 ;  ASCII "-WARE0"  
007211EB  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]  
007211EE  |.  BA 0D000000   MOV EDX,0D  
007211F3  |.  E8 D439CEFF   CALL dumped\_.00404BCC  
007211F8  |.  E9 68010000   JMP dumped\_.00721365  
007211FD  |>  68 9C137200   PUSH dumped\_.0072139C                 ;  压入 ASCII "JLH-"; Case 1 of switch 00721122  
00721202  |.  8D45 B4       LEA EAX,DWORD PTR SS:[EBP-4C]  
00721205  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]          ;  EDX=778271505352756668667153668010299114106122109100(  
机器码和用户名合并起来 10 进制字符串)  
00721208  |.  8A12          MOV DL,BYTE PTR DS:[EDX]               ;  取第一位，DL=7  
0072120A  |.  E8 2538CEFF   CALL dumped\_.00404A34  
0072120F  |.  FF75 B4       PUSH DWORD PTR SS:[EBP-4C]  
00721212  |.  8D45 B0       LEA EAX,DWORD PTR SS:[EBP-50]  
00721215  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]          ;  EDX=778271505352756668667153668010299114106122109100(  
机器码和用户名合并起来 10 进制字符串)  
00721218  |.  8A52 02       MOV DL,BYTE PTR DS:[EDX+2]             ;  取第三位，DL=8  
0072121B  |.  E8 1438CEFF   CALL dumped\_.00404A34  
00721220  |.  FF75 B0       PUSH DWORD PTR SS:[EBP-50]  
00721223  |.  8D45 AC       LEA EAX,DWORD PTR SS:[EBP-54]  
00721226  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]          ;  EDX=778271505352756668667153668010299114106122109100(  
机器码和用户名合并起来 10 进制字符串)  
00721229  |.  8A52 04       MOV DL,BYTE PTR DS:[EDX+4]             ;  取第五位，DL=7  
0072122C  |.  E8 0338CEFF   CALL dumped\_.00404A34  
00721231  |.  FF75 AC       PUSH DWORD PTR SS:[EBP-54]  
00721234  |.  8D45 A8       LEA EAX,DWORD PTR SS:[EBP-58]  
00721237  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]          ;  EDX=778271505352756668667153668010299114106122109100(  
机器码和用户名合并起来 10 进制字符串)  
0072123A  |.  8A52 06       MOV DL,BYTE PTR DS:[EDX+6]             ;  取第七位，DL=5  
0072123D  |.  E8 F237CEFF   CALL dumped\_.00404A34  
00721242  |.  FF75 A8       PUSH DWORD PTR SS:[EBP-58]  
00721245  |.  8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]  
00721248  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]          ;  EDX=778271505352756668667153668010299114106122109100(  
机器码和用户名合并起来 10 进制字符串)  
0072124B  |.  8A52 08       MOV DL,BYTE PTR DS:[EDX+8]             ;  取第九位，DL=5  
0072124E  |.  E8 E137CEFF   CALL dumped\_.00404A34  
00721253  |.  FF75 A4       PUSH DWORD PTR SS:[EBP-5C]  
00721256  |.  68 AC137200   PUSH dumped\_.007213AC                 ;  压入 ASCII "-SOFT-"  
0072125B  |.  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]  
0072125E  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]           ;  取 10 进制字符串送入 EDX=0612210910  
00721261  |.  8A12          MOV DL,BYTE PTR DS:[EDX]               ;  取第一位，DL=0  
00721263  |.  E8 CC37CEFF   CALL dumped\_.00404A34  
00721268  |.  FF75 A0       PUSH DWORD PTR SS:[EBP-60]  
0072126B  |.  8D45 9C       LEA EAX,DWORD PTR SS:[EBP-64]  
0072126E  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]           ;  取 10 进制字符串送入 EDX=0612210910  
00721271  |.  8A52 02       MOV DL,BYTE PTR DS:[EDX+2]             ;  取第三位，DL=1  
00721274  |.  E8 BB37CEFF   CALL dumped\_.00404A34  
00721279  |.  FF75 9C       PUSH DWORD PTR SS:[EBP-64]  
0072127C  |.  8D45 98       LEA EAX,DWORD PTR SS:[EBP-68]  
0072127F  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]           ;  取 10 进制字符串送入 EDX=0612210910  
00721282  |.  8A52 04       MOV DL,BYTE PTR DS:[EDX+4]             ;  取第五位，DL=2  
00721285  |.  E8 AA37CEFF   CALL dumped\_.00404A34  
0072128A  |.  FF75 98       PUSH DWORD PTR SS:[EBP-68]  
0072128D  |.  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  
00721290  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]           ;  取 10 进制字符串送入 EDX=0612210910  
00721293  |.  8A52 06       MOV DL,BYTE PTR DS:[EDX+6]             ;  取第七位，DL=0  
00721296  |.  E8 9937CEFF   CALL dumped\_.00404A34  
0072129B  |.  FF75 94       PUSH DWORD PTR SS:[EBP-6C]  
0072129E  |.  8D45 90       LEA EAX,DWORD PTR SS:[EBP-70]  
007212A1  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]           ;  取 10 进制字符串送入 EDX=0612210910  
007212A4  |.  8A52 08       MOV DL,BYTE PTR DS:[EDX+8]             ;  取第九位，DL=1  
007212A7  |.  E8 8837CEFF   CALL dumped\_.00404A34  
007212AC  |.  FF75 90       PUSH DWORD PTR SS:[EBP-70]  
007212AF  |.  68 CC137200   PUSH dumped\_.007213CC                 ;  压入 ASCII "-WARE1-"  
007212B4  |.  8D45 8C       LEA EAX,DWORD PTR SS:[EBP-74]  
007212B7  |.  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]          ;  
取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  
007212BA  |.  8A52 07       MOV DL,BYTE PTR DS:[EDX+7]             ;  取第八位，DL=4  
007212BD  |.  E8 7237CEFF   CALL dumped\_.00404A34  
007212C2  |.  FF75 8C       PUSH DWORD PTR SS:[EBP-74]  
007212C5  |.  8D45 88       LEA EAX,DWORD PTR SS:[EBP-78]  
007212C8  |.  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]          ;  
取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  
007212CB  |.  8A52 04       MOV DL,BYTE PTR DS:[EDX+4]             ;  取第五位，DL=9  
007212CE  |.  E8 6137CEFF   CALL dumped\_.00404A34  
007212D3  |.  FF75 88       PUSH DWORD PTR SS:[EBP-78]  
007212D6  |.  8D45 84       LEA EAX,DWORD PTR SS:[EBP-7C]  
```



```
007212D9  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

007212DC  |.   8A52 06       MOV DL,BYTE PTR DS:[EDX+6]              ;  取第七位，DL=1  

007212DF  |.   E8 5037CEFF   CALL dumped\_.00404A34  

007212E4  |.   FF75 84       PUSH DWORD PTR SS:[EBP-7C]  

007212E7  |.   8D45 80       LEA EAX,DWORD PTR SS:[EBP-80]  

007212EA  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

007212ED  |.   8A52 03       MOV DL,BYTE PTR DS:[EDX+3]              ;  取第四位，DL=9  

007212F0  |.   E8 3F37CEFF   CALL dumped\_.00404A34  

007212F5  |.   FF75 80       PUSH DWORD PTR SS:[EBP-80]  

007212F8  |.   8D85 7CFFFFFF LEA EAX,DWORD PTR SS:[EBP-84]  

007212FE  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

00721301  |.   8A52 01       MOV DL,BYTE PTR DS:[EDX+1]              ;  取第二位，DL=0  

00721304  |.   E8 2B37CEFF   CALL dumped\_.00404A34  

00721309  |.   FFB5 7CFFFFFF PUSH DWORD PTR SS:[EBP-84]  

0072130F  |.   8D85 78FFFFFF LEA EAX,DWORD PTR SS:[EBP-88]  

00721315  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

00721318  |.   8A52 02       MOV DL,BYTE PTR DS:[EDX+2]              ;  取第三位，DL=2  

0072131B  |.   E8 1437CEFF   CALL dumped\_.00404A34  

00721320  |.   FFB5 78FFFFFF PUSH DWORD PTR SS:[EBP-88]  

00721326  |.   8D85 74FFFFFF LEA EAX,DWORD PTR SS:[EBP-8C]  

0072132C  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

0072132F  |.   8A12          MOV DL,BYTE PTR DS:[EDX]                ;  取第一位，DL=1  

00721331  |.   E8 FE36CEFF   CALL dumped\_.00404A34  

00721336  |.   FFB5 74FFFFFF PUSH DWORD PTR SS:[EBP-8C]  

0072133C  |.   8D85 70FFFFFF LEA EAX,DWORD PTR SS:[EBP-90]  

00721342  |.   8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]            ;   

取用户名 20 位的 10 进制字符串送入 EDX=10299114106122109100  

00721345  |.   8A52 01       MOV DL,BYTE PTR DS:[EDX+1]              ;  取第二位，DL=0  

00721348  |.   E8 E736CEFF   CALL dumped\_.00404A34  

0072134D  |.   FFB5 70FFFFFF PUSH DWORD PTR SS:[EBP-90]  

00721353  |.   68 DC137200   PUSH dumped\_.007213DC  

00721358  |.   8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]  

0072135B  |.   BA 16000000   MOV EDX,16                             ;  EDX=16  

00721360  |.   E8 6738CEFF   CALL dumped\_.00404BCC                 ;  合并注册码!  

00721365  |>   33C0          XOR EAX,EAX                            ;  Default case of switch 00721122  

00721367  |.   5A            POP EDX  

00721368  |.   59            POP ECX  

00721369  |.   59            POP ECX  

0072136A  |.   64:8910       MOV DWORD PTR FS:[EAX],EDX  

0072136D  |.   68 8A137200   PUSH dumped\_.0072138A  

00721372  |>   8D85 70FFFFFF LEA EAX,DWORD PTR SS:[EBP-90]  

00721378  |.   BA 24000000   MOV EDX,24  

0072137D  |.   E8 EE34CEFF   CALL dumped\_.00404870  

00721382  |\.   C3            RETN  
```

---

## 总结

1. 将机器码和用户名转换成 10 进制字符串合并在一起，记做 `A`
2. 取机器码和用户名 10 进制字符串的后 11 位，去掉最后一位，记做 `B`。
3. 将用户名转换成 10 进制字符串，记做 `C`
4. 取 `A` 的第一、三、五、七、九位！
5. 取 `B` 的第一、三、五、七、九位！
6. 取 `C` 的第八、五、七、四、二、三、一、二位！
7. 注册码格式：`JHL-XXXXX-SOFT-XXXXX-WAER1-XXXXXXXXX-SL`

---

本文章写于 `2005-6-4 16:22:52`

`4dd466d0e1e0cc123afbe504da2462ca`
