
# ASF-AVI-RM-WMV Repair V1.41

> 原文：`www.pediy.com/kssd/pediy06/pediy6433.htm`

## 脱文信息

- 【脱文标题】 ASF-AVI-RM-WMV Repair V1.41 脱壳去暗桩+完美爆破
- 【脱文作者】 weiyi75[Dfcg]
- 【作者邮箱】 weiyi75@sohu.com
- 【作者主页】 Dfcg 官方大本营
- 【使用工具】 `Ollydbg1.10b`, `ImportREC1.42`, `Loadpe`
- 【破解平台】 `Win2000/XP`
- 【软件名称】 `ASF-AVI-RM-WMV Repair V 1.41`
- 【下载地址】 `www.fixdown.com/soft/9924.htm`
- 【软件简介】 ASF-AVI-RM-WMV Repair 是一个可以帮助你快速的修复被意外损坏的多媒体视频文件格式的修复工具,程序支持包括 ASF,AVI,RM,RMVB,WMV,WMADIVX,XVID,MPEG-4,等常用的视频文件格式
- 【软件大小】 639KB
- 【加壳方式】 `ASProtect 1.23 RC4 Registered -> Alexey Solodovnikov`
- 【保护方式】 没有使用 Asprotcet 的 Rsa 算法保护，采用自行研究的加密算法，启动时校验注册码，10 天试用期限制，SDK 防脱壳暗桩。
- 【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：

---

## 第一部分：脱去 Asprotect1.23 外壳保护，补上 Stolen Code,修复 IAT

首先 `Peid` 查壳，为 `ASProtect 1.23 RC4 Registered -> Alexey Solodovnikov`。

`Ollydbg` 载入程序,隐藏 `OD`,`OD` 异常设置忽略除内存异常外的全部异常。

```asm
00401000  >  68 01504A00     PUSH RepairVi.004A5001   //加壳程序入口点
00401005    E8 01000000     CALL RepairVi.0040100B
0040100A    C3             RETN
0040100B    C3             RETN
0040100C    F9             STC
0040100D    9C             PUSHFD
0040100E    31DD          XOR EBP,EBX
00401010    F69A F48AA45C  NEG BYTE PTR DS:[EDX+5CA48AF4]
00401016    AE             SCAS BYTE PTR ES:[EDI]
00401017    25 C4945BC8   AND EAX,C85B94C4
0040101C    D3C4          ROL ESP,CL
0040101E    DC26          FSUB QWORD PTR DS:[ESI]
00401020    9D            POPFD
00401021    94            XCHG EAX,ESP
00401022    B9 E2D941BE   MOV ECX,BE41D9E2
00401027    91            XCHG EAX,ECX
00401028    B4 70        MOV AH,70
0040102A    CA 9058      RETF 5890                      ; 远距返回
...........................................................................
```

这次用点新东西，用 `OllyScript` 的 `Asprotect1.23 rc4` 脚本到达最后一次异常。

```asm
011E3D03    3100          XOR DWORD PTR DS:[EAX],EAX //确定脚本执行完毕按钮到达地球人都知道的位置。
011E3D05    64:8F05 0000000>POP DWORD PTR FS:[0]
011E3D0C    58            POP EAX
011E3D0D    833D BC7E1E01 0>CMP DWORD PTR DS:[11E7EBC],0
011E3D14    74 14        JE SHORT 011E3D2A
011E3D16    6A 0C        PUSH 0C
011E3D18    B9 BC7E1E01  MOV ECX,11E7EBC
011E3D1D    8D45 F8      LEA EAX,DWORD PTR SS:[EBP-8]
011E3D20    BA 04000000  MOV EDX,4
011E3D25    E8 E6D2FFFF  CALL 011E1010
011E3D2A    FF75 FC      PUSH DWORD PTR SS:[EBP-4]
011E3D2D    FF75 F8      PUSH DWORD PTR SS:[EBP-8]
011E3D30    8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]
011E3D33    8338 00      CMP DWORD PTR DS:[EAX],0
011E3D36    74 02        JE SHORT 011E3D3A
011E3D38    FF30         PUSH DWORD PTR DS:[EAX]
011E3D3A    FF75 F0      PUSH DWORD PTR SS:[EBP-10]
011E3D3D    FF75 EC      PUSH DWORD PTR SS:[EBP-14]
011E3D40    C3           RETN  //这里下断点，Shift+F9 到这里后取消断点。
...........................................................................
```

虽然多种办法可以到达 Stolen Code，复习一下手动找 Stolen Code 位置，也很快的。

如无特别说明，一律 `F7` 步进，`F4` 跳出循环或到达指定位置。

```asm
011F60D8    80F0 44       XOR AL,44
011F60DB    0F84 09000000 JE 011F60EA
011F60E1    81D1 DC1DA872 ADC ECX,72A81DDC
011F60E7    0FB7FF       MOVZX EDI,DI
011F60EA    E8 0B000000  CALL 011F60FA
011F60FA    50           PUSH EAX                 ; RepairVi.004A595D
011F60FB    81C7 0DA0B27D ADD EDI,7DB2A00D
011F6101    81E9 1A917F18 SUB ECX,187F911A
011F6107    B5 7A        MOV CH,7A
011F6109    5F           POP EDI
011F610A    66:81C7 792D ADD DI,2D79
011F610F    5B           POP EBX
011F6110    68 6C2A8437  PUSH 37842A6C
011F6115    BF 5881BF7D  MOV EDI,7DBF8158
011F611A    59           POP ECX
011F611B    0F83 22000000 JNB 011F6143
011F6143    81E9 CECCD86D SUB ECX,6DD8CCCE      //这里停一下。
011F6149    81C3 0E160000 ADD EBX,160E
011F614F    BF E852A012  MOV EDI,12A052E8
011F6154    2BD2         SUB EDX,EDX
011F6156    E8 09000000  CALL 011F6164
011F615B    3D 32830039  CMP EAX,39008332
011F6160   ^ 7E DF      JLE SHORT 011F6141   //中间的循环过程等没必要执行。
011F6162    2C F5        SUB AL,0F5
011F6164    66:B8 7158   MOV AX,5871
011F6168    58           POP EAX
011F6169    0F8B 11000000 JPO 011F6180
011F616F    52           PUSH EDX
011F6170    68 2ECA6E55  PUSH 556ECA2E
011F6175    59           POP ECX
011F6176    5F           POP EDI
011F6177    66:8BC2      MOV AX,DX
011F617A    81F1 9217F440 XOR ECX,40F41792
011F6180    E9 05000000  JMP 011F618A         //F4 直接下来。
011F618A    8B3413       MOV ESI,DWORD PTR DS:[EBX+EDX]
011F618D    66:8BFA      MOV DI,DX
011F6190    81C6 B27ADE78 ADD ESI,78DE7AB2
011F6196    51           PUSH ECX
011F6197    8ACA         MOV CL,DL
011F6199    0FBFF9       MOVSX EDI,CX
011F619C    59           POP ECX
011F619D    81EE 03AAC857 SUB ESI,57C8AA03
011F61A3    0F8E 0C000000 JLE 011F61B5
011F61B5    E9 07000000  JMP 011F61C1
011F61C1    81EE 807D8F45 SUB ESI,458F7D80
011F61C7    8BFE         MOV EDI,ESI
011F61C9    893413       MOV DWORD PTR DS:[EBX+EDX],ESI
011F61CC    8BCB         MOV ECX,EBX
011F61CE    66:8BC1      MOV AX,CX
011F61D1    66:8BC7      MOV AX,DI
011F61D4    E8 0B000000  CALL 011F61E4
011F61D9   ^ 77 E4      JA SHORT 011F61BF
011F61DB    4D           DEC EBP
011F61DC    0213         ADD DL,BYTE PTR DS:[EBX]
011F61DE    50           PUSH EAX
011F61DF    49           DEC ECX
011F61E0    4E           DEC ESI
011F61E1    6F           OUTS DX,DWORD PTR ES:[EDI]       ; I/O 命令
011F61E2    7C 05        JL SHORT 011F61E9
011F61E4    B5 18        MOV CH,18
011F61E6    B8 03C9DE24  MOV EAX,24DEC903
011F61EB    0FBFFE       MOVSX EDI,SI
011F61EE    58           POP EAX
011F61EF    81EA D6EBD358 SUB EDX,58D3EBD6
011F61F5    E9 12000000  JMP 011F620C         //同理直接下来，看都不用看。
011F620C    81C2 D2EBD358 ADD EDX,58D3EBD2
011F6212    66:81C8 0C99 OR AX,990C
011F6217    B4 6E        MOV AH,6E
011F6219    81FA 60EBFFFF CMP EDX,-14A0
011F621F    0F85 1C000000 JNZ 011F6241
011F6225    8BFE         MOV EDI,ESI
011F6227    66:B8 2F83   MOV AX,832F
011F622B    E9 31000000  JMP 011F6261   //F4 直接下来。
011F6261    5B           POP EBX                ; RepairVi.00400000
011F6262    58           POP EAX
011F6263    05 2662FEDE  ADD EAX,DEFE6226
011F6268    5C           POP ESP
011F6269    EB 3F        JMP SHORT 011F62AA
011F62AA    03C3         ADD EAX,EBX                ; RepairVi.00400000
011F62AC    BB F2000000  MOV EBX,0F2
011F62B1    0BDB         OR EBX,EBX
011F62B3    75 02        JNZ SHORT 011F62B7
011F62B5    50           PUSH EAX
011F62B6    C3           RETN
011F62B7    E8 00000000  CALL 011F62BC
011F62BC    5D           POP EBP
011F62BD    81ED 23014B00 SUB EBP,4B0123
011F62C3    8D85 D2004B00 LEA EAX,DWORD PTR SS:[EBP+4B00D2]
011F62C9    8D8D 69014B00 LEA ECX,DWORD PTR SS:[EBP+4B0169]
011F62CF    03CB         ADD ECX,EBX
011F62D1    8941 01      MOV DWORD PTR DS:[ECX+1],EAX
011F62D4    8D85 11014B00 LEA EAX,DWORD PTR SS:[EBP+4B0111]
011F62DA    8D8D D6004B00 LEA ECX,DWORD PTR SS:[EBP+4B00D6]
011F62E0    8901         MOV DWORD PTR DS:[ECX],EAX
```



```assembly
011F62E2    B8 58140000     MOV EAX,1458   
011F62E7    8D8D DB004B00   LEA ECX,DWORD PTR SS:\[EBP+4B00DB\]   
011F62ED    8901            MOV DWORD PTR DS:\[ECX\],EAX   
011F62EF    8D8D 69014B00   LEA ECX,DWORD PTR SS:\[EBP+4B0169\]   
011F62F5    8D85 69134B00   LEA EAX,DWORD PTR SS:\[EBP+4B1369\]   
011F62FB    51              PUSH ECX   
011F62FC    50              PUSH EAX   
011F62FD    E8 7BFFFFFF     CALL 011F627D   
011F6302    64:EB 02        JMP SHORT 011F6307          //F4 直接下来。    
011F6307   /2E:EB 01        JMP SHORT 011F630B                          ; 多余的前缀   
011F630B    83EC 1A         SUB ESP,1A   
011F630E    83C4 16         ADD ESP,16   
011F6311    66:8135 1A631F0>XOR WORD PTR DS:\[11F631A\],0C6CB   
011F7E0A    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E0E    50              PUSH EAX   
011F7E0F    2E:EB 02        JMP SHORT 011F7E14                          ; 多余的前缀   
011F7E14    8F4424 00       POP DWORD PTR SS:\[ESP\]             ; 011F7DF2   
011F7E18    65:EB 02        JMP SHORT 011F7E1D                          ; 多余的前缀   
011F7E1D    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E21    66:0FBCC7       BSF AX,DI   
011F7E25    66:812D 2E7E1F0>SUB WORD PTR DS:\[11F7E2E\],6B2A   
011F7E2E    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E33    83E0 67         AND EAX,67   
011F7E36    66:812D 3F7E1F0>SUB WORD PTR DS:\[11F7E3F\],0B47A   
011F7E3F   /EB 01           JMP SHORT 011F7E42   
011F7E42   /EB 02           JMP SHORT 011F7E46   
011F7E46    8D83 3A184A79   LEA EAX,DWORD PTR DS:\[EBX+794A183A\]   
011F7E4C    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E51    66:812D 5B7E1F0>SUB WORD PTR DS:\[11F7E5B\],98C5   
011F7E5A   /3E:EB 02        JMP SHORT 011F7E5F                          ; 多余的前缀   
011F7E5F    06              PUSH ES   
011F7E60    EB 02           JMP SHORT 011F7E64   
011F7E64    C74424 00 477E1>MOV DWORD PTR SS:\[ESP\],11F7E47   
011F7E6C    58              POP EAX   
011F7E6D    FF50 2B         CALL DWORD PTR DS:\[EAX+2B\]   
011F7E7B    58              POP EAX                       ; 011F7E70   
011F7E7C    EB 01           JMP SHORT 011F7E7F   
011F7E7F   /EB 02           JMP SHORT 011F7E83   
011F7E83    8D4458 6F       LEA EAX,DWORD PTR DS:\[EAX+EBX\*2+6F\]   
011F7E87    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E8B    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E90    8D81 4CDFC21F   LEA EAX,DWORD PTR DS:\[ECX+1FC2DF4C\]   
011F7E96    58              POP EAX   
011F7E97    64:EB 02        JMP SHORT 011F7E9C                          ; 多余的前缀   
011F7E9C    55              PUSH EBP      \*     //这里看到 ESP 的值是 0012FFA4，难怪 12ffc0 的硬件写入断点没断下来，EBP 的值也不是 12fff0 模拟跟踪也失败，所以这个程序用 0012FFA0 的硬件写入断点一步就可以断下来，不过手动跟踪也非常快。    
011F7E9D    8BEC            MOV EBP,ESP   \*    //执行的结果是 esp=ebp=12ffa0    
011F7E9F    6A FF           PUSH -1       \*        带\*的内容是 Stolen Code   
011F7EA1    68 30284700     PUSH 472830   \*   
011F7EA6    68 E4234400     PUSH 4423E4   \*   
011F7EAB    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   \*   
011F7EB1    64:EB 02        JMP SHORT 011F7EB6                          ; 多余的前缀   
011F7EB6    50              PUSH EAX      \*   
011F7EB7    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   \*   
011F7EBE    83EC 58         SUB ESP,58           \*   
011F7EC1    64:EB 02        JMP SHORT 011F7EC6                          ; 多余的前缀   
011F7EC6    53              PUSH EBX    \*   
011F7EC7    64:EB 02        JMP SHORT 011F7ECC                          ; 多余的前缀   
011F7ECC    56              PUSH ESI    \*   
011F7ECD    64:EB 02        JMP SHORT 011F7ED2                          ; 多余的前缀   
011F7ED2    57              PUSH EDI    \*   
011F7ED3    8965 E8         MOV DWORD PTR SS:\[EBP\-18\],ESP  \*   
011F7ED6    64:EB 02        JMP SHORT 011F7EDB                          ; 多余的前缀   
011F7EDB   /64:EB 01        JMP SHORT 011F7EDF                          ; 多余的前缀   
011F7EDF    68 EA3E4400     PUSH 443EEA   
011F7EE4    68 5B7D1F01     PUSH 11F7D5B   
011F7EE9    C3              RETN   
011F7D5B    60              PUSHAD   
011F7D5C    9C              PUSHFD   
011F7D5D    FC              CLD   
011F7D5E    BF 9A7D1F01     MOV EDI,11F7D9A   
011F7D63    B9 58140000     MOV ECX,1458   
011F7D68    F3:AA           REP STOS BYTE PTR ES:\[EDI\]   //F7 可以看着它循环执行 Stolen Code     
011F7D6A    9D              POPFD   
011F7D6B    61              POPAD   
011F7D6C    C3              RETN   
```

整理一下 Stolen Code 

```assembly
PUSH EBP   
MOV EBP,ESP   
PUSH -1   
PUSH 00472830   
PUSH 004423E4               
MOV EAX,DWORD PTR FS:\[0\]   
PUSH EAX   
MOV DWORD PTR FS:\[0\],ESP   
SUB ESP,58   
PUSH EBX   
PUSH ESI   
PUSH EDI   
MOV DWORD PTR SS:\[EBP\-18\],ESP   
```

```assembly
00443EC4    0000            ADD BYTE PTR DS:\[EAX\],AL        //真 Oep,OD 直接补上整理的 Stolen Code     
00443EC6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EC8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECA    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECC    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECE    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED0    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED2    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED4    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDA    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDC    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDE    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE0    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE2    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE4    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EEA    FF15 C4834600   CALL DWORD PTR DS:\[4683C4\]      伪 OEP,滚动条往上面看。  
00443EF0    33D2            XOR EDX,EDX   
00443EF2    8AD4            MOV DL,AH   
00443EF4    8915 C0F24800   MOV DWORD PTR DS:\[48F2C0\],EDX   
00443EFA    8BC8            MOV ECX,EAX   
00443EFC    81E1 FF000000   AND ECX,0FF   
00443F02    890D BCF24800   MOV DWORD PTR DS:\[48F2BC\],ECX   
00443F08    C1E1 08         SHL ECX,8   
00443F0B    03CA            ADD ECX,EDX   
00443F0D    890D B8F24800   MOV DWORD PTR DS:\[48F2B8\],ECX   
00443F13    C1E8 10         SHR EAX,10   
00443F16    A3 B4F24800     MOV DWORD PTR DS:\[48F2B4\],EAX   
00443F1B    6A 01           PUSH 1   
00443F1D    E8 84330000     CALL RepairVi.004472A6   
```

```assembly
00443EC4    55              PUSH EBP        //OD 用插件脱壳，修正入口为 43Ec4 重建输入表选方式 2    
00443EC5    8BEC            MOV EBP,ESP   
00443EC7    6A FF           PUSH -1   
00443EC9    68 30284700     PUSH RepairVi.00472830   
00443ECE    68 E4234400     PUSH RepairVi.004423E4   
00443ED3    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   
00443ED9    50              PUSH EAX   
00443EDA    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   
00443EE1    83EC 58         SUB ESP,58   
00443EE4    53              PUSH EBX   
00443EE5    56              PUSH ESI   
00443EE6    57              PUSH EDI   
00443EE7    8965 E8         MOV DWORD PTR SS:\[EBP\-18\],ESP   
00443EEA    FF15 C4834600   CALL DWORD PTR DS:\[4683C4\]   
00443EF0    33D2            XOR EDX,EDX   
00443EF2    8AD4            MOV DL,AH   
00443EF4    8915 C0F24800   MOV DWORD PTR DS:\[48F2C0\],EDX   
00443EFA    8BC8            MOV ECX,EAX   
00443EFC    81E1 FF000000   AND ECX,0FF   
```



### 汇编代码片段
```
00443F02    890D BCF24800   MOV DWORD PTR DS:[48F2BC],ECX

00443F08    C1E1 08         SHL ECX,8

00443F0B    03CA           ADD ECX,EDX

00443F0D    890D B8F24800   MOV DWORD PTR DS:[48F2B8],ECX

00443F13    C1E8 10        SHR EAX,10

00443F16    A3 B4F24800     MOV DWORD PTR DS:[48F2B4],EAX

00443F1B    6A 01          PUSH 1

00443F1D    E8 84330000     CALL RepairVi.004472A6
```

........................................................................

### IAT 修复
运行 `ImportREC`，`OEP` 填入 `00043EC4`，自动搜索，获得输入信息，先用跟踪等级 1 修复，然后用 `Asprotect1.3` 插件修复剩下的几个指针，修复先脱壳的 `Unpack.exe`。

### 第二部分：去除暗桩
`OD` 载入修复的文件，`F9` 运行。

`OD` 提示访问违反。

看堆栈友好提示：

```
0012BEB0   0040DF22  返回到 Unpack\_.0040DF22 来自 011E14A4 //错误来自 0040DF22
0012BEB4   00469440  Unpack\_.00469440
0012BEB8   00000004
0012BEBC   FFFFFFFF
0012BEC0   004872C8  Unpack\_.004872C8
0012BEC4   FFFFFFFF
0012BEC8   0012C3C8
0012BECC   0012FF08  指针到下一个 SEH 记录
```

那么 `Ctrl+G` 去 `0040DF22` 看看。

```
0040DF1C    FF15 08FF4700   CALL DWORD PTR DS:[47FF08] //ds:[47FF08]指向壳中,跟踪原程序发现其作用是调用解压程序。

0040DF22    FF15 00FF4700   CALL DWORD PTR DS:[47FF00] //这里出错,这个 CALL 是加密刚才解压的程序。继续执行，脱壳后要 NOP 掉。

0040DF28    E9 27000000     JMP Unpack\_.0040DF54

0040DF2D    D5 24          AAD 24       //点清除分析可看到和我一样的代码。

0040DF2F    2D 81F42C54     SUB EAX,542CF481

0040DF34    3D 463DA974     CMP EAX,74A93D46

0040DF39    3938           CMP DWORD PTR DS:[EAX],EDI

0040DF3B    EC            IN AL,DX                 ; I/O 命令

0040DF3C    9B            WAIT

0040DF3D    872A          XCHG DWORD PTR DS:[EDX],EBP

0040DF3F    C2 44D6       RETN 0D644

0040DF42    92            XCHG EAX,EDX

0040DF43    08DF          OR BH,BL

0040DF45    A9 1EF9B175   TEST EAX,75B1F91E

0040DF4A    45            INC EBP

0040DF4B    D9CA          FXCH ST(2)

0040DF4D    05 44530606   ADD EAX,6065344

0040DF52    5E            POP ESI

0040DF53    F1            INT1

0040DF54    FF15 04FF4700 CALL DWORD PTR DS:[47FF04]

0040DF5A    8D86 F43A0000 LEA EAX,DWORD PTR DS:[ESI+3AF4]

0040DF60    50            PUSH EAX

0040DF61    FF15 5C834600 CALL DWORD PTR DS:[<&kernel32.Initialize>; KERNEL32.InitializeCriticalSection

0040DF67    8B4C24 10     MOV ECX,DWORD PTR SS:[ESP+10]

0040DF6B    8BC6          MOV EAX,ESI

0040DF6D    5F            POP EDI

0040DF6E    5E            POP ESI

0040DF6F    5B            POP EBX

0040DF70    64:890D 0000000>MOV DWORD PTR FS:[0],ECX

0040DF77    83C4 10       ADD ESP,10

0040DF7A    C2 0400       RETN 4
```

...........................................................................

看 `DS:[47FF08]` 里面的内容是 `011E14A4`，`ds:[48D4E0]` 里面的内容是 `011E1440`。两个地方都是指向壳中，SDK!

`Od` 载入原程序。

跟进 `0040DF1C    FF15 08FF4700   CALL DWORD PTR DS:[47FF08]`

```
011E14A4    55            PUSH EBP   //跟进，晕，好多代码啊，壳是不可能搬回的,它进壳中的过程我们没必有知道，但它从壳中取出代码动态修改程序领空代码我们脱壳后是需要的。

011E14A5    8BEC          MOV EBP,ESP

011E14A7    53            PUSH EBX

011E14A8    56            PUSH ESI

011E14A9    8B5D 0C       MOV EBX,DWORD PTR SS:[EBP+C]

011E14AC    8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]

011E14AF    8BC3          MOV EAX,EBX

011E14B1    E8 8E10FFFF   CALL 011D2544

011E14B6    A3 1C631E01   MOV DWORD PTR DS:[11E631C],EAX

011E14BB    8B15 1C631E01 MOV EDX,DWORD PTR DS:[11E631C]

011E14C1    8BC6          MOV EAX,ESI

011E14C3    8BCB          MOV ECX,EBX

011E14C5    E8 8611FFFF   CALL 011D2650

011E14CA    891D 007D1E01 MOV DWORD PTR DS:[11E7D00],EBX

011E14D0    53            PUSH EBX

011E14D1    8BCE          MOV ECX,ESI

011E14D3    8B15 FC7C1E01 MOV EDX,DWORD PTR DS:[11E7CFC]

011E14D9    A1 F87C1E01   MOV EAX,DWORD PTR DS:[11E7CF8]

011E14DE    E8 2DFBFFFF   CALL 011E1010

011E14E3    833D B0791E01 0>CMP DWORD PTR DS:[11E79B0],0

011E14EA    75 05        JNZ SHORT 011E14F1

011E14EC    E8 4FFFFFFF   CALL 011E1440

011E14F1    5E            POP ESI

011E14F2    5B            POP EBX

011E14F3    5D            POP EBP

011E14F4    C2 0800       RETN 8   //返回 40DF22
```

.........................................................

```
0040DF1C    FF15 08FF4700 CALL DWORD PTR DS:[47FF08]   //通过 011E14F4    C2 0800     RETN 8 返回 40DF22,这里知道了随便给它一个 retn 8 就可以过关。

0040AF15    FF15 D8D44800 CALL DWORD PTR DS:[48D4D8]   //到这里,下面和我们脱壳后看到的一样。但是 F8 通过这句话，下面的代码发生大变化。

0040DF28   /E9 01000000    JMP RepairVi.0040DF2E

0040DF2D   |ED            IN EAX,DX                    ; I/O 命令

0040DF2E   391D 4CDA4700  CMP DWORD PTR DS:[47DA4C],EBX

0040DF34    7F 05        JG SHORT RepairVi.0040DF3B

0040DF36    E8 05EDFFFF   CALL RepairVi.0040CC40

0040DF3B    E8 70ECFFFF   CALL RepairVi.0040CBB0

0040DF40    85C0          TEST EAX,EAX

0040DF42    77 0A        JA SHORT RepairVi.0040DF4E

0040DF44    C786 E0380000 0>MOV DWORD PTR DS:[ESI+38E0],19720203

0040DF4E    EB 04        JMP SHORT RepairVi.0040DF54

0040DF50    EA 54F978FF 150>JMP FAR 0415:FF78F954       ; 远距跳转

0040DF57    FF47 00       INC DWORD PTR DS:[EDI]

0040DF5A    8D86 F43A0000 LEA EAX,DWORD PTR DS:[ESI+3AF4]

0040DF60    50            PUSH EAX

0040DF61    FF15 5C834600 CALL DWORD PTR DS:[46835C]

0040DF67    8B4C24 10     MOV ECX,DWORD PTR SS:[ESP+10]

0040DF6B    8BC6          MOV EAX,ESI

0040DF6D    5F            POP EDI

0040DF6E    5E            POP ESI

0040DF6F    5B            POP EBX

0040DF70    64:890D 0000000>MOV DWORD PTR FS:[0],ECX

0040DF77    83C4 10       ADD ESP,10

0040DF7A    C2 0400       RETN 4
```

................................................................

### 原先地址内容,未执行 Call 之前。
```
0040DF28   /E9 27000000    JMP Unpack\_.0040DF54

0040DF2D   |D5 24         AAD 24

0040DF2F   |2D 81F42C54   SUB EAX,542CF481

0040DF34   |3D 463DA974   CMP EAX,74A93D46

0040DF39   |3938          CMP DWORD PTR DS:[EAX],EDI

0040DF3B   |EC           IN AL,DX                 ; I/O 命令

0040DF3C   |9B           WAIT

0040DF3D   |872A         XCHG DWORD PTR DS:[EDX],EBP

0040DF3F   |C2 44D6      RETN 0D644

0040DF42   |92           XCHG EAX,EDX

0040DF43   |08DF         OR BH,BL

0040DF45   |A9 1EF9B175  TEST EAX,75B1F91E

0040DF4A   |45           INC EBP

0040DF4B   |D9CA         FXCH ST(2)

0040DF4D   |05 44530606  ADD EAX,6065344

0040DF52   |5E           POP ESI

0040DF53   |F1           INT1

0040DF54   FF15 04FF4700 CALL DWORD PTR DS:[47FF04]

0040DF5A    8D86 F43A0000 LEA EAX,DWORD PTR DS:[ESI+3AF4]

0040DF60    50            PUSH EAX

0040DF61    FF15 5C834600 CALL DWORD PTR DS:[<&kernel32.Initialize>; KERNEL32.InitializeCriticalSection

0040DF67    8B4C24 10     MOV ECX,DWORD PTR SS:[ESP+10]

0040DF6B    8BC6          MOV EAX,ESI

0040DF6D    5F            POP EDI

0040DF6E    5E            POP ESI

0040DF6F    5B            POP EBX

0040DF70    64:890D 0000000>MOV DWORD PTR FS:[0],ECX

0040DF77    83C4 10       ADD ESP,10

0040DF7A    C2 0400       RETN 4
```

....................................................................

看到这些，你说说我们脱壳的程序能正常运行吗?

为了验证下面两个语句作用。

```
0040DF1C    FF15 08FF4700 CALL DWORD PTR DS:[47FF08]     //不用说，是从壳中解压需要的数据。

0040DF22    FF15 00FF4700 CALL DWORD PTR DS:[47FF00]     F2 在这里下断点，F9 运行。

0040DF28   /E9 27000000    JMP RepairVi.0040DF54      //呵呵，程序又变成这种样子，所以 0040DF22 处的 CALL 是加密 0040DF1C 解压出来的数据，反脱壳用的，我们脱壳时已经执行了这句,中了招。

0040DF2D   |D5 24         AAD 24

0040DF2F   |2D 81F42C54   SUB EAX,542CF481
```



### 0040DF34 - 0040DF7A

```
0040DF34   |3D 463DA974     CMP EAX,74A93D46   

0040DF39   |3938            CMP DWORD PTR DS:\[EAX\],EDI   

0040DF3B   |EC              IN AL,DX                           ; I/O 命令   

0040DF3C   |9B              WAIT   

0040DF3D   |872A            XCHG DWORD PTR DS:\[EDX\],EBP   

0040DF3F   |C2 44D6         RETN 0D644   

0040DF42   |92              XCHG EAX,EDX   

0040DF43   |08DF            OR BH,BL   

0040DF45   |A9 1EF9B175     TEST EAX,75B1F91E   

0040DF4A   |45              INC EBP   

0040DF4B   |D9CA            FXCH ST(2)   

0040DF4D   |05 44530606     ADD EAX,6065344   

0040DF52   |5E              POP ESI   

0040DF53   |F1              INT1   

0040DF54   FF15 04FF4700   CALL DWORD PTR DS:\[47FF04\]   

0040DF5A    8D86 F43A0000   LEA EAX,DWORD PTR DS:\[ESI+3AF4\]   

0040DF60    50              PUSH EAX   

0040DF61    FF15 5C834600   CALL DWORD PTR DS:\[46835C\]   

0040DF67    8B4C24 10       MOV ECX,DWORD PTR SS:\[ESP+10\]   

0040DF6B    8BC6            MOV EAX,ESI   

0040DF6D    5F              POP EDI   

0040DF6E    5E              POP ESI   

0040DF6F    5B              POP EBX   

0040DF70    64:890D 0000000>MOV DWORD PTR FS:\[0\],ECX   

0040DF77    83C4 10         ADD ESP,10   

0040DF7A    C2 0400         RETN 4   
```

............................................................   

#### 修复第一处暗桩。   

```
0040DF1C    FF15 08FF4700   CALL DWORD PTR DS:\[47FF08\]     //不用说，是从壳中解压需要的数据。数据我们在后面补上，看到它最后用 retn 8 返回。    
```

`Ctrl+F` 随便在程序中找个返回地址给它吧。 

```
0040EB41    C2 0800         RETN 8   //我选的这里。    
```

命令行 `dd 47FF08`   

修改壳中数据 `00B41C08` 为 `0040EB41`   

```
0040DF22    FF15 00FF4700   CALL DWORD PTR DS:\[47FF00\]    //它是个加密 Call,当然要 NOP 掉，直接用 OD 修改,注意使用 NOP 填充复选框要打上，这是基本的汇编知识。    
```

#### 第二步，用正确的解码替换错误的代码

`OD` 直接二进制复制下面的全部内容,这里 `OD` 一个工具足够了，`Winhex`   

不要嫉妒啊。先分析一下下面的内容有没有花指令，指向壳中的代码。   

```
0040DF28   /E9 01000000     JMP RepairVi.0040DF2E   

0040DF2D   |ED              IN EAX,DX                            ; I/O 命令   

0040DF2E   391D 4CDA4700   CMP DWORD PTR DS:\[47DA4C\],EBX   

0040DF34    7F 05          JG SHORT RepairVi.0040DF3B   

0040DF36    E8 05EDFFFF     CALL RepairVi.0040CC40   

0040DF3B    E8 70ECFFFF     CALL RepairVi.0040CBB0   

0040DF40    85C0           TEST EAX,EAX   

0040DF42    77 0A          JA SHORT RepairVi.0040DF4E   

0040DF44    C786 E0380000 0>MOV DWORD PTR DS:\[ESI+38E0\],19720203   

0040DF4E    EB 04          JMP SHORT RepairVi.0040DF54   //注意这里。    

0040DF50    EA 54F978FF 150>JMP FAR 0415:FF78F954                    ; 远距跳转   

0040DF57    FF47 00        INC DWORD PTR DS:\[EDI\]   

0040DF54    FF15 04FF4700   CALL DWORD PTR DS:\[47FF04\]  //这也是加密 Call,NOP 掉。    

0040DF54   > 90           NOP   

0040DF55   . 90           NOP   

0040DF56   . 90           NOP   

0040DF57   . 90           NOP   

0040DF58   . 90           NOP   

0040DF59   . 90           NOP   

0040DF5A    8D86 F43A0000   LEA EAX,DWORD PTR DS:\[ESI+3AF4\]   

0040DF60    50              PUSH EAX   

0040DF61    FF15 5C834600   CALL DWORD PTR DS:\[46835C\]   

0040DF67    8B4C24 10       MOV ECX,DWORD PTR SS:\[ESP+10\]   

0040DF6B    8BC6            MOV EAX,ESI   

0040DF6D    5F              POP EDI   

0040DF6E    5E              POP ESI   

0040DF6F    5B              POP EBX   

0040DF70    64:890D 0000000>MOV DWORD PTR FS:\[0\],ECX   

0040DF77    83C4 10         ADD ESP,10   

0040DF7A    C2 0400         RETN 4   
```

..........................................................   

将上面解码的内容二进制粘贴进入脱壳的程序，注意位置小心操作。   

#### 补上代码后。   

```
0040DF1C    FF15 08FF4700   CALL DWORD PTR DS:\[47FF08\]          //注意 DS:\[48D4E0\]指针要修改指向程序中。     


0040DF22    90              NOP   

0040DF23    90              NOP   

0040DF24    90              NOP   

0040DF25    90              NOP   

0040DF26    90              NOP   

0040DF27    90              NOP   

0040DF28    E9 01000000     JMP fsfsfsf.0040DF2E   

0040DF2D    ED              IN EAX,DX                            ; I/O 命令   

0040DF2E    391D 4CDA4700   CMP DWORD PTR DS:\[47DA4C\],EBX   

0040DF34    7F 05          JG SHORT fsfsfsf.0040DF3B   

0040DF36    E8 05EDFFFF     CALL fsfsfsf.0040CC40   

0040DF3B    E8 70ECFFFF     CALL fsfsfsf.0040CBB0   

0040DF40    85C0           TEST EAX,EAX   

0040DF42    77 0A          JA SHORT fsfsfsf.0040DF4E   

0040DF44    C786 E0380000 0>MOV DWORD PTR DS:\[ESI+38E0\],19720203   

0040DF4E    EB 04          JMP SHORT fsfsfsf.0040DF54   

0040DF50    EA 54F97890 909>JMP FAR 9090:9078F954                   ; 远距跳转   

0040DF57    90              NOP   

0040DF58    90              NOP   

0040DF59    90              NOP   

0040DF5A    8D86 F43A0000   LEA EAX,DWORD PTR DS:\[ESI+3AF4\]   

0040DF60    50              PUSH EAX   

0040DF61    FF15 5C834600   CALL DWORD PTR DS:\[<&kernel32.Initialize>; KERNEL32.InitializeCriticalSection   

0040DF67    8B4C24 10       MOV ECX,DWORD PTR SS:\[ESP+10\]   

0040DF6B    8BC6            MOV EAX,ESI   

0040DF6D    5F              POP EDI   

0040DF6E    5E              POP ESI   

0040DF6F    5B              POP EBX   

0040DF70    64:890D 0000000>MOV DWORD PTR FS:\[0\],ECX   

0040DF77    83C4 10         ADD ESP,10   

0040DF7A    C2 0400         RETN 4   
```

.................................................................   

将所有修改内容复制到程序中，保存为一个文件。   

这个程序校验处较多，处理方式相同，我就不一一解释了。   

### 修复第二处暗桩。   

`OD` 载入修复后的程序,`F9` 运行,程序界面出来了。别高兴，最关键的修复功能还没有试，肯定还有暗桩。   

执行修复文件功能。 

`OD` 提示访问违反。 

看堆栈友好提示 

```
0012B8C4   0040FB65  返回到 Unpack1.0040FB65 来自 011E1440   //错误来自 0040FB65    

0012B8C8   0012C3C8   

0012B8CC   0012C3C8   

0012B8D0   00000001   

0012B8D4   0012C3C8   

0012B8D8   0012C3C8   

0012B8DC   0012B8F8  指针到下一个 SEH 记录   

0012B8E0   004654AA  SE 句柄   
```

.................................................. 

`Ctrl+G` `0040FB65` 

```
0040FB5F    FF15 00FF4700   CALL DWORD PTR DS:\[47FF00\]  //加密解压代码,NOP 掉。    

0040FB65    E9 DA000000     JMP Unpack1.0040FC44      //一眼看出这是错误的代码，必须修正。    

0040FB6A    06              PUSH ES   

0040FB6B    96              XCHG EAX,ESI   

0040FB6C    B6 1D          MOV DH,1D   

0040FB6E    16              PUSH SS   

0040FB6F    6B54C9 3B D1    IMUL EDX,DWORD PTR DS:\[ECX+ECX*8+3B\],-2F   

0040FB74    27              DAA   

0040FB75    D6              SALC   

0040FB76    E2 4C          LOOPD SHORT Unpack1.0040FBC4   

0040FB78    53              PUSH EBX   

0040FB79    CB              RETF                              ; 远距返回   

0040FB7A    3A87 D5FEDB1A   CMP AL,BYTE PTR DS:\[EDI+1ADBFED5\]   

0040FB80    FD              STD   

0040FB81    9B              WAIT   

0040FB82  ^ EB F6          JMP SHORT Unpack1.0040FB7A   

0040FB84    44              INC ESP   

0040FB85    71 00          JNO SHORT Unpack1.0040FB87   

0040FB87    4F              DEC EDI   

0040FB88    7A F4          JPE SHORT Unpack1.0040FB7E   

.........................................................................   
```

切换到加壳程序中，`Ctrl+G` `0040FB5F` 

```
0040FB5F    FF15 00FF4700   CALL DWORD PTR DS:\[47FF00\] //下断点,执行文件修复时中断，取消断点。    

0040FB65    E9 DA000000     JMP RepairVi.0040FC44  F8 运行到这句程序动态解码。   

0040FB6A    06              PUSH ES   

0040FB6B    96              XCHG EAX,ESI   

0040FB6C    B6 1D          MOV DH,1D   

0040FB6E    16              PUSH SS   

0040FB6F    6B54C9 3B D1    IMUL EDX,DWORD PTR DS:\[ECX+ECX*8+3B\],-2F   

0040FB74    27              DAA   

0040FB75    D6              SALC   

0040FB76    E2 4C          LOOPD SHORT RepairVi.0040FBC4   

0040FB78    53              PUSH EBX   

0040FB79    CB              RETF                              ; 远距返回   

................................................................   
```

```
0040FB5F    FF15 00FF4700   CALL DWORD PTR DS:\[47FF00\]     //解码后，二进制复制下面全部内容。这个加密 Call 当然要 NOP 掉。    

0040FB65   /E9 01000000     JMP RepairVi.0040FB6B   

0040FB6A   |3E:8B86 D038000>MOV EAX,DWORD PTR DS:\[ESI+38D0\]   

0040FB71    8B78 04         MOV EDI,DWORD PTR DS:\[EAX+4\]   
```



0040FB74    8B4F 24         MOV ECX,DWORD PTR DS:[EDI+24]   

0040FB77    8B57 20         MOV EDX,DWORD PTR DS:[EDI+20]   

0040FB7A    51              PUSH ECX   

0040FB7B    52              PUSH EDX   

0040FB7C    E8 3FCDFFFF     CALL RepairVi.0040C8C0   

0040FB81    83C4 08         ADD ESP,8   

0040FB84    85C0            TEST EAX,EAX   

0040FB86    75 09           JNZ SHORT RepairVi.0040FB91   

0040FB88    A1 4CDA4700     MOV EAX,DWORD PTR DS:[47DA4C]   

0040FB8D    85C0            TEST EAX,EAX   

0040FB8F    7E 05           JLE SHORT RepairVi.0040FB96   

0040FB91    BB 01000000     MOV EBX,1   

0040FB96    57              PUSH EDI   

0040FB97    E8 94CDFFFF     CALL RepairVi.0040C930   

0040FB9C    83C4 04         ADD ESP,4   

0040FB9F    8986 E8380000   MOV DWORD PTR DS:[ESI+38E8],EAX   

0040FBA5    85DB            TEST EBX,EBX   

0040FBA7    8996 EC380000   MOV DWORD PTR DS:[ESI+38EC],EDX   

0040FBAD    0F84 8B000000   JE RepairVi.0040FC3E   

0040FBB3    8D4424 0C       LEA EAX,DWORD PTR SS:[ESP+C]   

0040FBB7    50              PUSH EAX   

0040FBB8    6A 04           PUSH 4   

0040FBBA    56              PUSH ESI   

0040FBBB    68 F0F74000     PUSH RepairVi.0040F7F0   

0040FBC0    6A 00           PUSH 0   

0040FBC2    6A 00           PUSH 0   

0040FBC4    FF15 48834600   CALL DWORD PTR DS:[468348]   

0040FBCA    85C0            TEST EAX,EAX   

0040FBCC    8986 303B0000   MOV DWORD PTR DS:[ESI+3B30],EAX   

0040FBD2    75 12           JNZ SHORT RepairVi.0040FBE6   

0040FBD4    5F              POP EDI   

0040FBD5    5E              POP ESI   

0040FBD6    5B              POP EBX   

0040FBD7    8B4C24 08       MOV ECX,DWORD PTR SS:[ESP+8]   

0040FBDB    64:890D 0000000>MOV DWORD PTR FS:[0],ECX   

0040FBE2    83C4 14         ADD ESP,14   

0040FBE5    C3              RETN   

0040FBE6    6A 00           PUSH 0   

0040FBE8    8D8E 380F0000   LEA ECX,DWORD PTR DS:[ESI+F38]   

0040FBEE    C786 2C3B0000 0>MOV DWORD PTR DS:[ESI+3B2C],0   

0040FBF8    E8 387A0400     CALL RepairVi.00457635   

0040FBFD    6A 00           PUSH 0   

0040FBFF    8D8E 20190000   LEA ECX,DWORD PTR DS:[ESI+1920]   

0040FC05    E8 2B7A0400     CALL RepairVi.00457635   

0040FC0A    6A 00           PUSH 0   

0040FC0C    8D8E 141E0000   LEA ECX,DWORD PTR DS:[ESI+1E14]   

0040FC12    E8 1E7A0400     CALL RepairVi.00457635   

0040FC17    6A 00           PUSH 0   

0040FC19    8D8E 08230000   LEA ECX,DWORD PTR DS:[ESI+2308]   

0040FC1F    E8 117A0400     CALL RepairVi.00457635   

0040FC24    6A 01           PUSH 1   

0040FC26    8D8E E8310000   LEA ECX,DWORD PTR DS:[ESI+31E8]   

0040FC2C    E8 047A0400     CALL RepairVi.00457635   

0040FC31    8B8E 303B0000   MOV ECX,DWORD PTR DS:[ESI+3B30]   

0040FC37    51              PUSH ECX   

0040FC38    FF15 4C834600   CALL DWORD PTR DS:[46834C]   

0040FC3E    EB 04           JMP SHORT RepairVi.0040FC44  //这里还藏有一句花指令。    

0040FC40    E5 87           IN EAX,87                        ; I/O 命令   

0040FC42    2E:72 FF        JB SHORT RepairVi.0040FC44       ; 多余的前缀   

0040FC3E  |>  EB 04         JMP SHORT Crackme.0040FC44   

0040FC44  |>   FF15 04FF4700 CALL DWORD PTR DS:[47FF04]  //加密 Call,脱壳后必须 NOP 掉，因为它指向壳中。    

0040FC45    15 04FF4700     ADC EAX,RepairVi.0047FF04   

0040FC4A    85DB            TEST EBX,EBX   

0040FC4C    75 43           JNZ SHORT RepairVi.0040FC91   

0040FC4E    6A 6C           PUSH 6C   

0040FC50    E8 29880400     CALL RepairVi.0045847E   

0040FC55    83C4 04         ADD ESP,4   

0040FC58    894424 10       MOV DWORD PTR SS:[ESP+10],EAX   

0040FC5C    85C0            TEST EAX,EAX   

0040FC5E    895C24 1C       MOV DWORD PTR SS:[ESP+1C],EBX   

0040FC62    74 0C           JE SHORT RepairVi.0040FC70   

0040FC64    53              PUSH EBX   

0040FC65    8BC8            MOV ECX,EAX   

0040FC67    E8 14D3FFFF     CALL RepairVi.0040CF80   

0040FC6C    8BF0            MOV ESI,EAX   

0040FC6E    EB 02           JMP SHORT RepairVi.0040FC72   

0040FC70    33F6            XOR ESI,ESI   

0040FC72    85F6            TEST ESI,ESI   

0040FC74    C74424 1C FFFFF>MOV DWORD PTR SS:[ESP+1C],-1   

0040FC7C    74 13           JE SHORT RepairVi.0040FC91   

0040FC7E    8B16            MOV EDX,DWORD PTR DS:[ESI]   

0040FC80    8BCE            MOV ECX,ESI   

0040FC82    FF92 B8000000   CALL DWORD PTR DS:[EDX+B8]   

0040FC88    8B06            MOV EAX,DWORD PTR DS:[ESI]   

0040FC8A    6A 01           PUSH 1   

0040FC8C    8BCE            MOV ECX,ESI   

0040FC8E    FF50 04         CALL DWORD PTR DS:[EAX+4]   

0040FC91    8B4C24 14       MOV ECX,DWORD PTR SS:[ESP+14]   

0040FC95    5F              POP EDI   

0040FC96    5E              POP ESI   

0040FC97    B8 01000000     MOV EAX,1   

0040FC9C    5B              POP EBX   

0040FC9D    64:890D 0000000>MOV DWORD PTR FS:[0],ECX   

0040FCA4    83C4 14         ADD ESP,14   

0040FCA7    C3              RETN   

.....................................................................   

保存所有的修改，两处暗桩去掉后，程序正常运行并可以修复文件，另外 10 天试用期随着壳的脱去化为乌有。   

最后爆破它，用别人的 KEY 注册感觉不爽。   

破解分析参考我的  AVI MPEG RM WMV Joiner4.11 脱壳+破解,过程查不多。   

关键跳转分析如下。   

0040D73F   .  E8 FCEEFFFF   CALL Crack.0040C640   

0040D744   .  83C4 0C       ADD ESP,0C   

0040D747   .  85C0          TEST EAX,EAX   

0040D749   .  74 0A         JE SHORT Crack.0040D755      //爆破点，jne 40D755    

0040D74B   .  C786 C0000000>MOV DWORD PTR DS:[ESI+C0],1  //全局标志位赋值。 

破解后任意名字注册。   

\--------------------------------------------------------------------------------   

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  

 4dd466d0e1e0cc123afbe504da2462ca
