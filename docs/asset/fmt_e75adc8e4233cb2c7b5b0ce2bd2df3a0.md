
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
