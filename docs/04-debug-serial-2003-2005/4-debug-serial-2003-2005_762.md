
# My Notes Keeper 1.1.5 注册码分析

> 原文：`www.pediy.com/kssd/pediy07/pediy7-302.htm`

【破文标题】My Notes Keeper 1.1.5 注册码分析

【破文作者】sailor

【软件名称】My Notes Keeper 1.1.5

【下载地址】`http://gz.onlinedown.net/soft/40432.htm`

【破解工具】OD、PEiD、AspackDie

【破解难度】中等

----------------------------------------------------

### 软件介绍

软件大小：`2937KB`

软件语言：英文

软件类别：国外软件/共享版/记事管理

运行环境：`Win9x`/`Me`/`NT`/`2000`/`XP`

是一款功能强大、简单易用的树状标签结构个人数据库管理软件，能进行个人信息管理和文字、表格处理，有密码保护功能。你可以通过它管理你的通讯簿、网址收藏和安排日程表等，甚至可以用它来制作电子书。软件的操作方式跟 `Word` 几乎没有多大区别。

----------------------------------------------------

### 破解声明

----------------------------------------------------

### 破解分析

先查壳、脱壳

#### 1、找有用的提示

```
006ED7BB  |.  68 1AD96E00        PUSH unpacked.006ED91A
006ED7C0  |.  64:FF30            PUSH DWORD PTR FS:[EAX]
006ED7C3  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP
006ED7C6  |.  8D55 FC            LEA EDX,DWORD PTR SS:[EBP-4]
006ED7C9  |.  8B83 00030000      MOV EAX,DWORD PTR DS:[EBX+300]
006ED7CF  |.  E8 5CD4D8FF        CALL unpacked.0047AC30
006ED7D4  |.  837D FC 00         CMP DWORD PTR SS:[EBP-4],0               ;  没有输入用户名，则 bye
006ED7D8  |.  74 14              JE SHORT unpacked.006ED7EE
006ED7DA  |.  8D55 F8            LEA EDX,DWORD PTR SS:[EBP-8]
006ED7DD  |.  8B83 04030000      MOV EAX,DWORD PTR DS:[EBX+304]
006ED7E3  |.  E8 48D4D8FF        CALL unpacked.0047AC30
006ED7E8  |.  837D F8 00         CMP DWORD PTR SS:[EBP-8],0
006ED7EC  |.  75 5B              JNZ SHORT unpacked.006ED849              ;  没有输入试炼码，则 bye
006ED7EE  |>  6A 10              PUSH 10
006ED7F0  |.  68 28D96E00        PUSH unpacked.006ED928                   ;  ASCII "My Notes Keeper"
006ED7F5  |.  8D45 F4            LEA EAX,DWORD PTR SS:[EBP-C]
006ED7F8  |.  50                 PUSH EAX
006ED7F9  |.  8D55 F0            LEA EDX,DWORD PTR SS:[EBP-10]
006ED7FC  |.  A1 087E7500        MOV EAX,DWORD PTR DS:[757E08]
006ED801  |.  E8 EA9AD1FF        CALL unpacked.004072F0
006ED806  |.  8B55 F0            MOV EDX,DWORD PTR SS:[EBP-10]
006ED809  |.  B9 40D96E00        MOV ECX,unpacked.006ED940                ;  ASCII "Reg"
006ED80E  |.  B8 4CD96E00        MOV EAX,unpacked.006ED94C                ;  ASCII "strBothNameCode"
006ED813  |.  E8 E07BF2FF        CALL unpacked.006153F8
006ED818  |.  8B45 F4            MOV EAX,DWORD PTR SS:[EBP-C]
006ED81B  |.  E8 847AD1FF        CALL unpacked.004052A4
006ED820  |.  50                 PUSH EAX
006ED821  |.  8BC3               MOV EAX,EBX
006ED823  |.  E8 283CD9FF        CALL unpacked.00481450
006ED828  |.  50                 PUSH EAX                                 ; |hOwner
006ED829  |.  E8 CAABD1FF        CALL <JMP.&user32.MessageBoxA>           ; \温馨提示，可要输入用户名和注册码哦
006ED82E  |.  8B83 00030000      MOV EAX,DWORD PTR DS:[EBX+300]
006ED834  |.  8B10               MOV EDX,DWORD PTR DS:[EAX]
006ED836  |.  FF92 C4000000      CALL DWORD PTR DS:[EDX+C4]
006ED83C  |.  33C0               XOR EAX,EAX
006ED83E  |.  8983 4C020000      MOV DWORD PTR DS:[EBX+24C],EAX
006ED844  |.  E9 8F000000        JMP unpacked.006ED8D8
006ED849  |>  8D55 EC            LEA EDX,DWORD PTR SS:[EBP-14]
006ED84C  |.  8B83 04030000      MOV EAX,DWORD PTR DS:[EBX+304]
006ED852  |.  E8 D9D3D8FF        CALL unpacked.0047AC30
006ED857  |.  8B45 EC            MOV EAX,DWORD PTR SS:[EBP-14]            ;  eax＝试炼码
006ED85A  |.  50                 PUSH EAX
006ED85B  |.  8D55 E8            LEA EDX,DWORD PTR SS:[EBP-18]
006ED85E  |.  8B83 00030000      MOV EAX,DWORD PTR DS:[EBX+300]
006ED864  |.  E8 C7D3D8FF        CALL unpacked.0047AC30
006ED869  |.  8B45 E8            MOV EAX,DWORD PTR SS:[EBP-18]            ;  eax=用户名
006ED86C  |.  5A                 POP EDX
006ED86D  |.  E8 6A82F2FF        CALL unpacked.00615ADC                   ;  关键 CALL，进入
006ED872  |.  84C0               TEST AL,AL
006ED874  |.  74 0C              JE SHORT unpacked.006ED882
006ED876  |.  C783 4C020000 0100>MOV DWORD PTR DS:[EBX+24C],1
006ED880  |.  EB 56              JMP SHORT unpacked.006ED8D8
006ED882  |>  6A 10              PUSH 10
006ED884  |.  68 28D96E00        PUSH unpacked.006ED928                   ;  ASCII "My Notes Keeper"
006ED889  |.  8D45 E4            LEA EAX,DWORD PTR SS:[EBP-1C]
006ED88C  |.  50                 PUSH EAX
006ED88D  |.  8D55 E0            LEA EDX,DWORD PTR SS:[EBP-20]
006ED890  |.  A1 A0827500        MOV EAX,DWORD PTR DS:[7582A0]
006ED895  |.  E8 569AD1FF        CALL unpacked.004072F0
006ED89A  |.  8B55 E0            MOV EDX,DWORD PTR SS:[EBP-20]
006ED89D  |.  B9 40D96E00        MOV ECX,unpacked.006ED940                ;  ASCII "Reg"
006ED8A2  |.  B8 64D96E00        MOV EAX,unpacked.006ED964                ;  ASCII "strInvalidNameCode"
006ED8A7  |.  E8 4C7BF2FF        CALL unpacked.006153F8
006ED8AC  |.  8B45 E4            MOV EAX,DWORD PTR SS:[EBP-1C]
006ED8AF  |.  E8 F079D1FF        CALL unpacked.004052A4
006ED8B4  |.  50                 PUSH EAX
006ED8B5  |.  8BC3               MOV EAX,EBX
006ED8B7  |.  E8 943BD9FF        CALL unpacked.00481450
006ED8BC  |.  50                 PUSH EAX                                 ; |hOwner
006ED8BD  |.  E8 36ABD1FF        CALL <JMP.&user32.MessageBoxA>           ; \注册失败对话框
006ED8C2  |.  8B83 00030000      MOV EAX,DWORD PTR DS:[EBX+300]
006ED8C8  |.  8B10               MOV EDX,DWORD PTR DS:[EAX]
006ED8CA  |.  FF92 C4000000      CALL DWORD PTR DS:[EDX+C4]
006ED8D0  |.  33C0               XOR EAX,EAX
```

#### 2、

```
00615ADC  /$  55                 PUSH EBP
00615ADD  |.  8BEC               MOV EBP,ESP
00615ADF  |.  83C4 F4            ADD ESP,-0C
00615AE2  |.  53                 PUSH EBX
00615AE3  |.  56                 PUSH ESI
00615AE4  |.  33C9               XOR ECX,ECX
00615AE6  |.  894D FC            MOV DWORD PTR SS:[EBP-4],ECX
00615AE9  |.  8BF2               MOV ESI,EDX
00615AEB  |.  8BD8               MOV EBX,EAX
00615AED  |.  33C0               XOR EAX,EAX
00615AEF  |.  55                 PUSH EBP
00615AF0  |.  68 665B6100        PUSH unpacked.00615B66
00615AF5  |.  64:FF30            PUSH DWORD PTR FS:[EAX]
00615AF8  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP
00615AFB  |.  66:B9 1F00         MOV CX,1F
00615AFF  |.  66:BA 0C00         MOV DX,0C
00615B03  |.  66:B8 0F27         MOV AX,270F
00615B07  |.  E8 506FDFFF        CALL unpacked.0040CA5C
00615B0C  |.  83C4 F8            ADD ESP,-8
00615B0F  |.  DD1C24             FSTP QWORD PTR SS:[ESP]
00615B12  |.  9B                 WAIT
00615B13  |.  8D4D F4            LEA ECX,DWORD PTR SS:[EBP-C]
00615B16  |.  B8 8C807400        MOV EAX,unpacked.0074808C
00615B1B  |.  8BD3               MOV EDX,EBX                        ;  edx=用户名
00615B1D  |.  E8 F643F3FF        CALL unpacked.00549F18               ;  计算注册码关键 CALL，跟进
00615B22  |.  8D4D FC            LEA ECX,DWORD PTR SS:[EBP-4]
00615B25  |.  8D45 F4            LEA EAX,DWORD PTR SS:[EBP-C]
00615B28  |.  BA 08000000        MOV EDX,8
00615B2D  |.  E8 7A25F3FF        CALL unpacked.005480AC
00615B32  |.  8BD6               MOV EDX,ESI                        ;  edx=试炼码
00615B34  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]         ;  eax=注册码
00615B37  |.  E8 8440DFFF        CALL unpacked.00409BC0
00615B3C  |.  85C0               TEST EAX,EAX
00615B3E  |.  75 08              JNZ SHORT unpacked.00615B48
00615B40  |.  85DB               TEST EBX,EBX
00615B42  |.  74 04              JE SHORT unpacked.00615B48
00615B44  |.  85F6               TEST ESI,ESI
00615B46  |.  75 04              JNZ SHORT unpacked.00615B4C
```



# 3、进入计算注册码关键 CALL

## 00615B48

```
00615B48  |>  33C0               XOR EAX,EAX                          ;  注册失败标志

00615B4A  |.  EB 02              JMP SHORT unpacked.00615B4E

00615B4C  |>  B0 01             MOV AL,1                               ;  注册成功标志

00615B4E  |>  8BD8               MOV EBX,EAX

00615B50  |.  33C0               XOR EAX,EAX

00615B52  |.  5A                 POP EDX

00615B53  |.  59                 POP ECX

00615B54  |.  59                 POP ECX

00615B55  |.  64:8910            MOV DWORD PTR FS:[EAX],EDX

00615B58  |.  68 6D5B6100        PUSH unpacked.00615B6D

00615B5D  |>  8D45 FC            LEA EAX,DWORD PTR SS:[EBP-4]

00615B60  |.  E8 7FF2DEFF        CALL unpacked.00404DE4

00615B65  \.  C3                 RETN
```

## 3.1 进入 CALL 00549570

```
00549F18  /$  55                 PUSH EBP

00549F19  |.  8BEC               MOV EBP,ESP

00549F1B  |.  6A 00              PUSH 0

00549F1D  |.  6A 00              PUSH 0

00549F1F  |.  53                 PUSH EBX

00549F20  |.  56                 PUSH ESI

00549F21  |.  57                 PUSH EDI

00549F22  |.  8BF1               MOV ESI,ECX

00549F24  |.  8BDA               MOV EBX,EDX

00549F26  |.  8BF8               MOV EDI,EAX

00549F28  |.  33C0               XOR EAX,EAX

00549F2A  |.  55                 PUSH EBP

00549F2B  |.  68 BE9F5400        PUSH unpacked.00549FBE

00549F30  |.  64:FF30            PUSH DWORD PTR FS:[EAX]

00549F33  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP

00549F36  |.  66:C706 F6D9       MOV WORD PTR DS:[ESI],0D9F6

00549F3B  |.  FF75 0C            PUSH DWORD PTR SS:[EBP+C]                   ;  /Arg2

00549F3E  |.  FF75 08            PUSH DWORD PTR SS:[EBP+8]                   ;  |Arg1

00549F41  |.  E8 32FDFFFF        CALL unpacked.00549C78                      ;  \unpacked.00549C78

00549F46  |.  66:8946 02         MOV WORD PTR DS:[ESI+2],AX

00549F4A  |.  8D45 FC            LEA EAX,DWORD PTR SS:[EBP-4]

00549F4D  |.  8BD3               MOV EDX,EBX

00549F4F  |.  E8 28AFEBFF        CALL unpacked.00404E7C

00549F54  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]               ;  eax=用户名

00549F57  |.  E8 48B1EBFF        CALL unpacked.004050A4                      ;  用户名字符数

00549F5C  |.  8BD8               MOV EBX,EAX

00549F5E  |.  83FB 01            CMP EBX,1                                   ;  如果没有输入用户名

00549F63  |> |8B45 FC            /MOV EAX,DWORD PTR SS:[EBP-4]

00549F66  |. |8A4418 FF          |MOV AL,BYTE PTR DS:[EAX+EBX-1]

00549F6A  |. |3C 7F              |CMP AL,7F                                  ;  如果字符 ASCII 大于 7F

00549F6C  |. |76 0F              |JBE SHORT unpacked.00549F7D                ;  从用户名中剔除掉

00549F6E  |. |8D45 FC            |LEA EAX,DWORD PTR SS:[EBP-4]

00549F71  |. |B9 01000000        |MOV ECX,1

00549F76  |. |8BD3               |MOV EDX,EBX

00549F78  |. |E8 C7B3EBFF        |CALL unpacked.00405344

00549F7D  |> |4B                 |DEC EBX

00549F7E  |. |85DB               |TEST EBX,EBX

00549F80  |.^|75 E1              \JNZ SHORT unpacked.00549F63

00549F82  |> \8D55 F8            LEA EDX,DWORD PTR SS:[EBP-8]

00549F85  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]

00549F88  |.  E8 ABFCEBFF        CALL unpacked.00409C38                      ;  用户名字符转大写　ABCD

00549F8D  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]

00549F90  |.  E8 DBF5FFFF        CALL unpacked.00549570                      ;  进去看看(3.1)

00549F95  |.  8946 04            MOV DWORD PTR DS:[ESI+4],EAX                ;  ?eax=45674

00549F98  |.  8BD6               MOV EDX,ESI                                 ;  45674＝（（（41*10）+42）*10+43）*10+44

00549F9A  |.  8BC7               MOV EAX,EDI

00549F9C  |.  B1 01              MOV CL,1

00549F9E  |.  E8 A5F4FFFF        CALL unpacked.00549448                      ;  关键的查表计算处(3.2)

00549FA3  |.  33C0               XOR EAX,EAX

00549FA5  |.  5A                 POP EDX

00549FA6  |.  59                 POP ECX

00549FA7  |.  59                 POP ECX

00549FA8  |.  64:8910            MOV DWORD PTR FS:[EAX],EDX

00549FAB  |.  68 C59F5400        PUSH unpacked.00549FC5

00549FB0  |>  8D45 F8            LEA EAX,DWORD PTR SS:[EBP-8]

00549FB3  |.  BA 02000000        MOV EDX,2

00549FB8  |.  E8 4BAEEBFF        CALL unpacked.00404E08

00549FBD  \.  C3                 RETN
```

### 00549570

```
00549570  /$  53                 PUSH EBX

00549571  |.  8BD8               MOV EBX,EAX

00549573  |.  8BC3               MOV EAX,EBX

00549575  |.  E8 2ABBEBFF        CALL unpacked.004050A4

0054957A  |.  50                 PUSH EAX

0054957B  |.  8BC3               MOV EAX,EBX

0054957D  |.  E8 22BDEBFF        CALL unpacked.004052A4

00549582  |.  5A                 POP EDX

00549583  |.  E8 B0FFFFFF        CALL unpacked.00549538                      ;  再进入

00549588  |.  5B                 POP EBX

00549589  \.  C3                 RETN

00549538  /$  53                 PUSH EBX

00549539  |.  56                 PUSH ESI

0054953A  |.  33C9               XOR ECX,ECX

0054953C  |.  8BDA               MOV EBX,EDX

0054953E  |.  4B                 DEC EBX

0054953F  |.  85DB               TEST EBX,EBX

00549541  |.  7C 25              JL SHORT unpacked.00549568

00549543  |.  43                 INC EBX

00549544  |>  C1E1 04            /SHL ECX,4                                  ;  ecx*10

00549547  |.  33D2               |XOR EDX,EDX

00549549  |.  8A10               |MOV DL,BYTE PTR DS:[EAX]                   ;  dl=41

0054954B  |.  03CA               |ADD ECX,EDX                                 ;  41*10+42

0054954D  |.  8BD1               |MOV EDX,ECX

0054954F  |.  81E2 000000F0      |AND EDX,F0000000

00549555  |.  85D2               |TEST EDX,EDX

00549557  |.  74 07              |JE SHORT unpacked.00549560                 ;  如果和大于 10000000，则处理一下

00549559  |.  8BF2               |MOV ESI,EDX

0054955B  |.  C1EE 18            |SHR ESI,18

0054955E  |.  33CE               |XOR ECX,ESI

00549560  |>  F7D2               |NOT EDX

00549562  |.  23CA               |AND ECX,EDX

00549564  |.  40                 |INC EAX

00549565  |.  4B                 |DEC EBX

00549566  |.^ 75 DC              \JNZ SHORT unpacked.00549544

00549568  |>  8BC1               MOV EAX,ECX                                 ;  以上循环完成(((A*10)+B)*10)+C)*10)+D

0054956A  |.  5E                 POP ESI                                      ;  即(((41*10)+42)*10)+43)*10)+44

0054956B  |.  5B                 POP EBX

0054956C  \.  C3                 RETN
```

## 3.2

```
00549448  /$  53                 PUSH EBX

00549449  |.  56                 PUSH ESI

0054944A  |.  57                 PUSH EDI

0054944B  |.  83C4 E8            ADD ESP,-18

0054944E  |.  884C24 08          MOV BYTE PTR SS:[ESP+8],CL

00549452  |.  895424 04          MOV DWORD PTR SS:[ESP+4],EDX

00549456  |.  890424            MOV DWORD PTR SS:[ESP],EAX

00549459  |.  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]

0054945D  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]

0054945F  |.  894424 0C          MOV DWORD PTR SS:[ESP+C],EAX

00549463  |.  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]

00549467  |.  8B40 04            MOV EAX,DWORD PTR DS:[EAX+4]                 ;  eax=45674

0054946A  |.  894424 10          MOV DWORD PTR SS:[ESP+10],EAX

0054946E  |.  C74424 14 04000000 MOV DWORD PTR SS:[ESP+14],4                  ;  进行 4 次循环查表计算

00549476  |.  BE 00D07300        MOV ESI,unpacked.0073D000                   ;  下面开始计算注册码了

0054947B  |>  8B5424 0C          /MOV EDX,DWORD PTR SS:[ESP+C]               ;  ?edx==D9F6（好像是固定值）

0054947F  |.  33C0               |XOR EAX,EAX

00549481  |.  8A4424 08          |MOV AL,BYTE PTR SS:[ESP+8]                 ;  ss:[esp+8]==1

00549485  |.  8BD8               |MOV EBX,EAX
```



```asm
00549487  .|  03DB               |ADD EBX,EBX  

00549489  .|  8D1C5B             |LEA EBX,DWORD PTR DS:[EBX+EBX*2]  

0054948C  .|  8B04DE             |MOV EAX,DWORD PTR DS:[ESI+EBX*8]        ;  查表 1  

0054948F  .|  8B0C24             |MOV ECX,DWORD PTR SS:[ESP]  

00549492  .|  8B0C81             |MOV ECX,DWORD PTR DS:[ECX+EAX*4]        ;  查表 2  

00549495  .|  8B44DE 04          |MOV EAX,DWORD PTR DS:[ESI+EBX*8+4]      ;  查表 1  

00549499  .|  8B3C24             |MOV EDI,DWORD PTR SS:[ESP]  

0054949C  .|  8B0487             |MOV EAX,DWORD PTR DS:[EDI+EAX*4]        ;  查表 2  

0054949F  .|  8B5CDE 08          |MOV EBX,DWORD PTR DS:[ESI+EBX*8+8]      ;  查表 1  

005494A3  .|  8B3C24             |MOV EDI,DWORD PTR SS:[ESP]  

005494A6  .|  8B1C9F             |MOV EBX,DWORD PTR DS:[EDI+EBX*4]        ;  查表 2  

005494A9  .|  03D3               |ADD EDX,EBX                             ;  这下面一大堆，要看懂得花不少功夫  

005494AB  .|  03DA               |ADD EBX,EDX                             ;  我不懂编程，也只知道几句汇编语句  

005494AD  .|  8BFA               |MOV EDI,EDX                             ;  但是不妨碍我们用 win32 汇编来写注册机  

005494AF  .|  C1EF 07            |SHR EDI,7                               ;  拷贝，粘贴就 OK 了  

005494B2  .|  33D7               |XOR EDX,EDI                             ;  不求甚解，会有提高嘛？！  

005494B4  .|  03CA               |ADD ECX,EDX                             ;  嗳:(  

005494B6  .|  03D1               |ADD EDX,ECX  

005494B8  .|  8BF9               |MOV EDI,ECX  

005494BA  .|  C1E7 0D            |SHL EDI,0D  

005494BD  .|  33CF               |XOR ECX,EDI  

005494BF  .|  03C1               |ADD EAX,ECX  

005494C1  .|  03C8               |ADD ECX,EAX  

005494C3  .|  8BF8               |MOV EDI,EAX  

005494C5  .|  C1EF 11            |SHR EDI,11  

005494C8  .|  33C7               |XOR EAX,EDI  

005494CA  .|  03D8               |ADD EBX,EAX  

005494CC  .|  03C3               |ADD EAX,EBX  

005494CE  .|  8BFB               |MOV EDI,EBX  

005494D0  .|  C1E7 09            |SHL EDI,9  

005494D3  .|  33DF               |XOR EBX,EDI  

005494D5  .|  03D3               |ADD EDX,EBX  

005494D7  .|  03DA               |ADD EBX,EDX  

005494D9  .|  8BFA               |MOV EDI,EDX  

005494DB  .|  C1EF 03            |SHR EDI,3  

005494DE  .|  33D7               |XOR EDX,EDI  

005494E0  .|  03CA               |ADD ECX,EDX  

005494E2  .|  8BD1               |MOV EDX,ECX  

005494E4  .|  C1E2 07            |SHL EDX,7  

005494E7  .|  33CA               |XOR ECX,EDX  

005494E9  .|  03C1               |ADD EAX,ECX  

005494EB  .|  8BD3               |MOV EDX,EBX  

005494ED  .|  C1EA 0F            |SHR EDX,0F  

005494F0  .|  33C2               |XOR EAX,EDX  

005494F2  .|  03D8               |ADD EBX,EAX  

005494F4  .|  8BC3               |MOV EAX,EBX  

005494F6  .|  C1E0 0B            |SHL EAX,0B  

005494F9  .|  33D8               |XOR EBX,EAX                             ;  第一次的 ebx 总是一样的 45E675B9  

005494FB  .|  8B4424 10          |MOV EAX,DWORD PTR SS:[ESP+10]           ;  第一次的[esp+10]=45674  

005494FF  .|  33C3               |XOR EAX,EBX  

00549501  .|  8B5424 0C          |MOV EDX,DWORD PTR SS:[ESP+C]            ;  第一次的[esp+c]=D9F6  

00549505  .|  895424 10          |MOV DWORD PTR SS:[ESP+10],EDX  

00549509  .|  894424 0C          |MOV DWORD PTR SS:[ESP+C],EAX  

0054950D  .|  83C6 0C            |ADD ESI,0C  

00549510  .|  FF4C24 14          |DEC DWORD PTR SS:[ESP+14]  

00549514  .^  0F85 61FFFFFF      \JNZ unpacked.0054947B  

0054951A  .|  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]             ;  [esp+4]存放计算出来的注册码  

0054951E  .|  8B5424 10          MOV EDX,DWORD PTR SS:[ESP+10]            ;  [esp+10]存放的是计算出的注册码第一部分  

00549522  .|  8910               MOV DWORD PTR DS:[EAX],EDX  

00549524  .|  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]  

00549528  .|  8B5424 0C          MOV EDX,DWORD PTR SS:[ESP+C]             ;  [esp+C]存放的是计算出的注册码第二部分  

0054952C  .|  8950 04            MOV DWORD PTR DS:[EAX+4],EDX  

0054952F  .|  83C4 18            ADD ESP,18  

00549532  .|  5F                 POP EDI  

00549533  .|  5E                 POP ESI  

00549534  .|  5B                 POP EBX  

00549535  \.  C3                 RETN  
```

### 表 1 数据

```text
0073D000  00 00 00 00 03 00 00 00 01 00 00 00 02 00 00 00  .............  

0073D010  01 00 00 00 03 00 00 00 01 00 00 00 00 00 00 00  .............  

0073D020  02 00 00 00 03 00 00 00 02 00 00 00 00 00 00 00  .............  

0073D030  03 00 00 00 02 00 00 00 00 00 00 00 01 00 00 00  .............  

0073D040  00 00 00 00 02 00 00 00 02 00 00 00 01 00 00 00  .............  

0073D050  03 00 00 00 00 00 00 00 03 00 00 00 01 00 00 00  .............  

0073D060  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  

0073D070  01 00 00 00 FF FF FF FF 60 22 E8 00 F4 FF FF FF  ...??\`"??  

0073D080  0C 00 00 00 E8 FF FF FF 18 00 00 00 C4 FF FF FF  ....?...?...  

0073D090  3C 00 00 00 C4 FF FF FF 3C 00 00 00 00 00 00 00  <...??.<.......  
```

### 表 2 数据

```text
0074808C  26 76 14 55 07 F1 0B 8D 40 2A 49 F9 4A 51 74 28  &vU?岪*I 鵍 Qt(  

0074809C  01 00 00 00 04 00 00 00 05 00 00 00 09 00 00 00  .............  

007480AC  0A 00 00 00 28 00 00 00 32 00 00 00 5A 00 00 00  ....(...2...Z...  
```

---

### 总结

---

### 版权信息

```asm
stringone db 3,2,0,1,0,2,2,1,3,0,3,1,0  

stringtwo db 26h,76h,14h,55h,07h,0F1h,0Bh,8Dh,40h,2Ah,49h,0F9h,4Ah,51h,74h,28h,0  

        lea eax,hInput1  

;以下代码完成删除大于 7F 的 ASCII 字符，新字符仍然存放于 hInput1 中  

        lea ebx, namelength  

        mov ebx,[ebx]  

abc1:    mov ecx,[eax+ebx-1]  

        cmp cl,7Fh  

        jbe abc2  

        push ebx  

        push eax  

    ;将前后两个字符串合并  

        xor ecx,ecx  

        mov [eax+ebx-1],cl  

        add ebx,eax  

        invoke lstrcat,eax,ebx  

        invoke lstrlen,eax  

        lea edx,namelength  

        mov [edx],eax  

        pop eax  

        pop ebx  

abc2:    dec ebx  

        test ebx,ebx  

        jnz abc1  

;以上代码完成删除大于 7F 的 ASCII 字符  

        lea ebx,namelength  

        mov ebx,[ebx]  

        push ebx  

        invoke CharUpperBuff,eax,ebx;字符串转化成大写 

;以下代码完成((41*10+42)*10+43)*10+44 功能，完全照搬软件中的代码  

        pop ebx  

        lea eax,hInput1  

        xor ecx,ecx  

        dec ebx  

        test ebx,ebx  

        jl abc5  

        inc ebx  

abc3:    SHL ECX,4  

        XOR EDX,EDX  

        MOV DL,[EAX]  

        ADD ECX,EDX  

        MOV EDX,ECX  

        AND EDX,0F0000000h  

        TEST EDX,EDX  

        JE abc4  

        MOV ESI,EDX  

        SHR ESI,18h  

        XOR ECX,ESI  

abc4:    NOT EDX  

        AND ECX,EDX  

        INC EAX  

        DEC EBX  

        JNZ abc3  

abc5:  

;以上代码完成((41*10+42)*10+43)*10+44 功能  

        push ebp  

        mov eax,ecx  

        MOV DWORD PTR SS:[ESP+10h],EAX  

        MOV DWORD PTR SS:[ESP+14h],4  

        MOV DWORD PTR SS:[ESP+0Ch],0D9F6h  

        lea esi,stringone  

        lea ebp,stringtwo  

abc6:    MOV EDX,DWORD PTR SS:[ESP+0Ch]  

        mov al,[esi]  

        and eax,0FFh  

        xor ecx,ecx  

        add ecx,eax  

        LEA ECX,DWORD PTR DS:[ECX*4]  

        add ecx,ebp  

        mov ecx,[ecx]  

        inc esi  

        mov al,[esi]  

        and eax,0FFh  

        xor edi,edi  

        add edi,eax  

        LEA EDI,DWORD PTR DS:[EDI*4]  

        add edi,ebp  

        mov eax,[edi]  

        inc esi  

        mov bl,[esi]  

        and ebx,0FFh  

        LEA EBX,DWORD PTR DS:[EBX*4]  

        add ebx,ebp  

        mov ebx,[ebx]  

;上面代码完成查表功能，并将查到的结果放到相应的 ecx,eax,ebx 中  

        ADD EDX,EBX  

        ADD EBX,EDX  

        MOV EDI,EDX  

        SHR EDI,7  

        XOR EDX,EDI  

        ADD ECX,EDX  

        ADD EDX,ECX  

        MOV EDI,ECX  

        SHL EDI,0Dh
```



```assembly
XOR ECX,EDI
ADD EAX,ECX
ADD ECX,EAX
MOV EDI,EAX
SHR EDI,11h
XOR EAX,EDI
ADD EBX,EAX
ADD EAX,EBX
MOV EDI,EBX
SHL EDI,9
XOR EBX,EDI
ADD EDX,EBX
ADD EBX,EDX
MOV EDI,EDX
SHR EDI,3
XOR EDX,EDI
ADD ECX,EDX
MOV EDX,ECX
SHL EDX,7
XOR ECX,EDX
ADD EAX,ECX
MOV EDX,EBX
SHR EDX,0Fh
XOR EAX,EDX
ADD EBX,EAX
MOV EAX,EBX
SHL EAX,0Bh
XOR EBX,EAX
MOV EAX,DWORD PTR SS:[ESP+10h]
xor eax,ebx
MOV EDX,DWORD PTR SS:[ESP+0Ch]
MOV DWORD PTR SS:[ESP+10h],EDX
MOV DWORD PTR SS:[ESP+0Ch],EAX
inc esi
DEC DWORD PTR SS:[ESP+14h]
JNZ abc6
MOV EDX,DWORD PTR SS:[ESP+10h]
lea eax,strsntemp
MOV DWORD PTR DS:[EAX],EDX
MOV EDX,DWORD PTR SS:[ESP+0Ch]
MOV DWORD PTR DS:[EAX+4],EDX
lea eax,strsntemp
lea edi,strsn
abc7: mov bl,[eax+ecx-1]
and ebx,0Fh
.if bl>9
add bl,37h
.else
add bl,30h
.endif
mov [edi+ecx*2-1],bl
mov bl,[eax+ecx-1]
and ebx,0F0h
shr ebx,4
.if bl>9
add bl,37h
.else
add bl,30h
.endif
mov [edi+ecx*2-2],bl
dec ecx
test ecx,ecx
jnz abc7
pop ebp
```
