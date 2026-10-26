
# 时代人事管理系统 V(算法分析简单)高手莫入

## 基本信息

【破解作者】 `layper`

【作者邮箱】 `layper2002@yahoo.com.cn`

【使用工具】 `peid`,`od`,`AspackDie`

【破解平台】 `Win9x/NT/2000/XP`

【软件名称】 时代人事管理系统 V

【下载地址】 `1b5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4G2k6Y4c8J5k6h3N6Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4y4Z5j5i4u0W2N6$3q4J5k6g2)9#2k6Y4k6A6k6i4N6Q4x3X3g2S2M7%4m8Q4x3@1k6A6k6q4)9K6c8q4)9J5c8V1u0r3b7U0j5^5x3o6t1%4i4K6u0V1y4o6l9J5y4W2)9J5k6o6c8p5c8U0c8Q4x3X3b7&6b7@1j5H3i4K6u0V1`

`76E49180E544/`

【软件大小】 `2.84MB`

【加壳方式】 `ASPack 2.12 -> Alexey Solodovnikov`

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

用`PEID`查壳`ASPack 2.12 -> Alexey Solodovnikov`，用`AspackDie`直接脱壳成功后再一次用`PEID`查询，用`Borland Delphi 6.0 - 7.0`写的，直接用`OD`载入-右键搜索字符串，找到“注册码不正确,无法注册!”，双击回到反汇编处，下断在`0061AA18`处，`F9`运行，在注册对话框处输入用户名“`layper`”,注册码“`1357924680`”，点确定后下断在此

```asm
0061AA18   .  53            PUSH EBX
0061AA19   .  8BD8          MOV EBX,EAX
0061AA1B   .  8BC3          MOV EAX,EBX
0061AA1D   .  E8 EAFEFFFF   CALL Unpacked.0061A90C                   ;  算法CALL，跟进
0061AA22   .  84C0          TEST AL,AL
0061AA24   .  74 09         JE SHORT Unpacked.0061AA2F               ；跳到错误对话框
0061AA26   .  8BC3          MOV EAX,EBX
0061AA28   .  E8 87FCFFFF   CALL Unpacked.0061A6B4                   ;  跳到成功的CALL
0061AA2D   .  EB 18         JMP SHORT Unpacked.0061AA47
0061AA2F   >  6A 10         PUSH 10
0061AA31   .  B9 64AA6100   MOV ECX,Unpacked.0061AA64
0061AA36   .  BA 6CAA6100   MOV EDX,Unpacked.0061AA6C                ;  注册码不正确,无法注册!
0061AA3B   .  A1 70146500   MOV EAX,DWORD PTR DS:[651470]
0061AA40   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
0061AA42   .  E8 5965E8FF   CALL Unpacked.004A0FA0
0061AA47   >  33D2          XOR EDX,EDX
0061AA49   .  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
0061AA4F   .  E8 484EE6FF   CALL Unpacked.0047F89C
0061AA54   .  33D2          XOR EDX,EDX
0061AA56   .  8B83 04030000 MOV EAX,DWORD PTR DS:[EBX+304]
0061AA5C   .  E8 3B4EE6FF   CALL Unpacked.0047F89C
0061AA61   .  5B            POP EBX
0061AA62   .  C3            RETN
```

跟进`0061AA1D`来到

```asm
0061A90C  /$  55            PUSH EBP
0061A90D  |.  8BEC          MOV EBP,ESP
0061A90F  |.  33C9          XOR ECX,ECX
0061A911  |.  51            PUSH ECX
0061A912  |.  51            PUSH ECX
0061A913  |.  51            PUSH ECX
0061A914  |.  51            PUSH ECX
0061A915  |.  51            PUSH ECX
0061A916  |.  51            PUSH ECX
0061A917  |.  51            PUSH ECX
0061A918  |.  53            PUSH EBX
0061A919  |.  56            PUSH ESI
0061A91A  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
0061A91D  |.  33C0          XOR EAX,EAX
0061A91F  |.  55            PUSH EBP
0061A920  |.  68 08AA6100   PUSH Unpacked.0061AA08
0061A925  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
0061A928  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
0061A92B  |.  33C0          XOR EAX,EAX
0061A92D  |.  8945 F0       MOV DWORD PTR SS:[EBP-10],EAX
0061A930  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
0061A933  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0061A936  |.  8B80 00030000 MOV EAX,DWORD PTR DS:[EAX+300]
0061A93C  |.  E8 2B4FE6FF   CALL Unpacked.0047F86C                  ；这个CALL把用户名的长度放入EAX
0061A941  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]            ；[EAX]=layper
0061A944  |.  E8 6BA6DEFF   CALL Unpacked.00404FB4                  ;又是用户名长度
0061A949  |.  8BD8          MOV EBX,EAX                             ；放入EBX
0061A94B  |.  85DB          TEST EBX,EBX                            ；比较用户名是否为空
0061A94D  |.  7E 3A         JLE SHORT Unpacked.0061A989
0061A94F  |.  BE 01000000   MOV ESI,1                               ；ESI=1，ESI为计数器，为下面用户名计算作准备
0061A954  |>  8D45 EC       /LEA EAX,DWORD PTR SS:[EBP-14]
0061A957  |.  50            |PUSH EAX
0061A958  |.  B9 01000000   |MOV ECX,1
0061A95D  |.  8BD6          |MOV EDX,ESI
0061A95F  |.  8B45 F4       |MOV EAX,DWORD PTR SS:[EBP-C]          ；[EAX]=layper
0061A962  |.  E8 ADA8DEFF   |CALL Unpacked.00405214
0061A967  |.  8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]
0061A96A  |.  E8 45A8DEFF   |CALL Unpacked.004051B4
0061A96F  |.  8945 F8       |MOV DWORD PTR SS:[EBP-8],EAX
0061A972  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]
0061A975  |.  8A00          |MOV AL,BYTE PTR DS:[EAX]             ;用户名逐位放入AL
0061A977  |.  25 FF000000   |AND EAX,0FF
0061A97C  |.  69C0 A3FE2600 |IMUL EAX,EAX,26FEA3                  ；乘于26FEA3
0061A982  |.  0145 F0       |ADD DWORD PTR SS:[EBP-10],EAX        ；与上一次循环的值相加，[ebp-10]里既是用户名运算结果
0061A985  |.  46            |INC ESI                              ；计数器加1
0061A986  |.  4B            |DEC EBX                              ；EBX也是计数器，减1
0061A987  |.^ 75 CB         \JNZ SHORT Unpacked.0061A954          ；没算完继续循环，算完往下
0061A989  |>  33DB          XOR EBX,EBX
0061A98B  |.  8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
0061A98E  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0061A991  |.  8B80 04030000 MOV EAX,DWORD PTR DS:[EAX+304]
0061A997  |.  E8 D04EE6FF   CALL Unpacked.0047F86C                  ；注册码长度
0061A99C  |.  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
0061A99F  |.  33D2          XOR EDX,EDX
0061A9A1  |.  E8 DAF2DEFF   CALL Unpacked.00409C80                   ;  注册码算法CALL，跟进
0061A9A6  |.  3B45 F0       CMP EAX,DWORD PTR SS:[EBP-10]            ；比较用户名运算结果与注册码运算结果
0061A9A9  |.  75 32         JNZ SHORT Unpacked.0061A9DD
0061A9AB  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
0061A9AE  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0061A9B1  |.  8B80 00030000 MOV EAX,DWORD PTR DS:[EAX+300]
0061A9B7  |.  E8 B04EE6FF   CALL Unpacked.0047F86C
0061A9BC  |.  837D E4 00    CMP DWORD PTR SS:[EBP-1C],0
0061A9C0  |.  75 04         JNZ SHORT Unpacked.0061A9C6
0061A9C2  |.  33DB          XOR EBX,EBX
0061A9C4  |.  EB 02         JMP SHORT Unpacked.0061A9C8
0061A9C6  |>  B3 01         MOV BL,1
0061A9C8  |>  B8 5C476500   MOV EAX,Unpacked.0065475C
0061A9CD  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]
0061A9D0  |.  E8 57A3DEFF   CALL Unpacked.00404D2C
0061A9D5  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
0061A9D8  |.  A3 60476500   MOV DWORD PTR DS:[654760],EAX
0061A9DD  |>  33C0          XOR EAX,EAX
0061A9DF  |.  5A            POP EDX
0061A9E0  |.  59            POP ECX
0061A9E1  |.  59            POP ECX
0061A9E2  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0061A9E5  |.  68 0FAA6100   PUSH Unpacked.0061AA0F
0061A9EA  |>  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
0061A9ED  |.  BA 02000000   MOV EDX,2
0061A9F2  |.  E8 05A3DEFF   CALL Unpacked.00404CFC
0061A9F7  |.  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
0061A9FA  |.  E8 D9A2DEFF   CALL Unpacked.00404CD8
0061A9FF  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
0061AA02  |.  E8 D1A2DEFF   CALL Unpacked.00404CD8
0061AA07  \.  C3            RETN
```

跟进`0061A9A1`处的算法`CALL`来到这里

```asm
00409C80  /$  53            PUSH EBX
00409C81  |.  51            PUSH ECX
00409C82  |.  8BDA          MOV EBX,EDX
00409C84  |.  8BD4          MOV EDX,ESP
00409C86  |.  E8 959BFFFF   CALL Unpacked.00403820             ；这里还要跟进，真正的算法CALL
00409C8B  |.  833C24 00     CMP DWORD PTR SS:[ESP],0
00409C8F  |.  74 02         JE SHORT Unpacked.00409C93
00409C91  |.  8BC3          MOV EAX,EBX
00409C93  |>  5A            POP EDX
00409C94  |.  5B            POP EBX
00409C95  \.  C3            RETN
```

跟进`00409C86`来到

```asm
00403820  /$  53            PUSH EBX
00403821  |.  56            PUSH ESI
00403822  |.  57            PUSH EDI
00403823  |.  89C6          MOV ESI,EAX
00403825  |.  50            PUSH EAX
00403826  |.  85C0          TEST EAX,EAX
00403828  |.  74 6C         JE SHORT Unpacked.00403896
0040382A  |.  31C0          XOR EAX,EAX
0040382C  |.  31DB          XOR EBX,EBX
0040382E  |.  BF CCCCCC0C   MOV EDI,0CCCCCCC                    ；EDI=0CCCCCCC
00403833  |>  8A1E          /MOV BL,BYTE PTR DS:[ESI]
00403835  |.  46            |INC ESI
00403836  |.  80FB 20       |CMP BL,20                         ；注册码是否为空
```
