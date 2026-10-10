
# 第二阶段第三题

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-765.htm`](https://www.pediy.com/kssd/pediy09/pediy09-765.htm)
>
> 脱壳机：`unpacker.rar`
>
> 说明：
>
> `un test.exe`
>
> 脱壳成功后同目录下会生成 `test_un.exe`
>
> 代码：
>
> ***



> `以 petgui.exe 本身为例,ollydbg 载入停在入口点:`
```
004E3046  >  B8 00304E00      MOV EAX,petgui.004E3000    ;指向壳区段,用来读取一些加壳数据
004E304B   68 E3644100      PUSH petgui.004164E3    ;SEH 处理入口
004E3050   64:FF35 00000000  PUSH DWORD PTR FS:[0]
004E3057   64:8925 00000000  MOV DWORD PTR FS:[0],ESP
004E305E   66:9C            PUSHFW
004E3060   60               PUSHAD
004E3061   50               PUSH EAX
004E3062   8BD8             MOV EBX,EAX
004E3064   0300             ADD EAX,DWORD PTR DS:[EAX]
004E3066   68 A4A50000      PUSH 0A5A4
004E306B   6A 00            PUSH 0
004E306D   FF50 1C          CALL DWORD PTR DS:[EAX+1C]          ;kernel32.GlobalAlloc
004E3070   8943 08          MOV DWORD PTR DS:[EBX+8],EAX  ;解压缩用的临时缓冲区
004E3073   68 00004000      PUSH petgui.00400000
004E3078   8B3C24           MOV EDI,DWORD PTR SS:[ESP]
004E307B   8B33             MOV ESI,DWORD PTR DS:[EBX]
004E307D   66:81C7 8007     ADD DI,780
004E3082   8D741E 08        LEA ESI,DWORD PTR DS:[ESI+EBX+8]
004E3086   893B             MOV DWORD PTR DS:[EBX],EDI
004E3088   53               PUSH EBX
004E3089   8B5E 10          MOV EBX,DWORD PTR DS:[ESI+10]
004E308C   B8 80080000      MOV EAX,880
004E3091   56               PUSH ESI
004E3092   6A 02            PUSH 2
004E3094   50               PUSH EAX
004E3095   57               PUSH EDI
004E3096   6A 15            PUSH 15
004E3098   6A 0A            PUSH 0A
004E309A   56               PUSH ESI
004E309B   6A 04            PUSH 4
004E309D   50               PUSH EAX
004E309E   57               PUSH EDI
004E309F   FFD3             CALL EBX                            ; kernel32.VirtualProtect
004E30A1   83EE 08          SUB ESI,8
004E30A4   59               POP ECX
004E30A5   F3:A5            REP MOVSD
004E30A7   59               POP ECX
004E30A8   66:83C7 58       ADD DI,58
004E30AC   81C6 02010000    ADD ESI,102
004E30B2   F3:A5            REP MOVSD        ;把输入表用到 dll 名字复制到 PE 头内某区域
004E30B4   FFD3             CALL EBX
004E30B6   58               POP EAX
004E30B7   8D90 A0010000    LEA EDX,DWORD PTR DS:[EAX+1A0]  ;获取压缩数据表
004E30BD   8B0A             MOV ECX,DWORD PTR DS:[EDX]    ;压缩数据 RVA
004E30BF   83C2 14          ADD EDX,14
004E30C2   8B5A F0          MOV EBX,DWORD PTR DS:[EDX-10]  ;解压后的长度
004E30C5   85DB             TEST EBX,EBX
004E30C7  74 F4            JE SHORT petgui.004E30BD
004E30C9   8B0424           MOV EAX,DWORD PTR SS:[ESP]    ;IMAGEBASE
004E30CC   8D3401           LEA ESI,DWORD PTR DS:[ECX+EAX]  ;压缩数据 VA
004E30CF   8B6C24 04        MOV EBP,DWORD PTR SS:[ESP+4]
004E30D3   8B6D 08          MOV EBP,DWORD PTR SS:[EBP+8]  ;临时缓冲区
004E30D6   8B4A FC          MOV ECX,DWORD PTR DS:[EDX-4]  ;压缩数据长度*4
004E30D9   8BFD             MOV EDI,EBP
004E30DB   52               PUSH EDX
004E30DC   F3:A5            REP MOVSD        ;把压缩数据拷贝到临时缓冲区
004E30DE   8BF5             MOV ESI,EBP
004E30E0   8B7A F4          MOV EDI,DWORD PTR DS:[EDX-C]  ;解压后的目的 RVA
004E30E3   03F8             ADD EDI,EAX        ;解压后的目的 VA
004E30E5   EB 28            JMP SHORT petgui.004E310F    ;跳转到解压缩函数
```
解压缩函数:
```
004E310F   81FB 50C30000    CMP EBX,0C350
004E3115   73 0E            JNB SHORT petgui.004E3125
004E3117   6A 05            PUSH 5
004E3119   68 60C0FFFF      PUSH -3FA0
004E311E   68 60FCFFFF      PUSH -3A0
004E3123   EB 0C            JMP SHORT petgui.004E3131
004E3125   6A 08            PUSH 8
004E3127   68 0083FFFF      PUSH FFFF8300
004E312C   68 00FBFFFF      PUSH -500
004E3131   6A FF            PUSH -1
004E3133   33D2             XOR EDX,EDX
004E3135   33C9             XOR ECX,ECX
004E3137   A4               MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
004E3138   305F FF          XOR BYTE PTR DS:[EDI-1],BL
004E313B   4B               DEC EBX
004E313C  7E A9            JLE SHORT petgui.004E30E7
...
004E30E7   58               POP EAX
004E30E8   58               POP EAX
004E30E9   58               POP EAX
004E30EA   58               POP EAX
004E30EB   5A               POP EDX
004E30EC  74 CF            JE SHORT petgui.004E30BD    ;解压缩成功,则去继续解压缩下一块数据
004E30EE  ^E9 19FFFFFF      JMP petgui.004E300C      ;解压缩失败报告错误
```
我们发现上面的解压缩部分是个死循环,没有正常的出口,是直到解压异常后,进入到前面设置的 SEH 处理入口的:
```
004164E3   E8 4F000000      CALL petgui.00416537
...
00416537   33C0             XOR EAX,EAX
00416539   5E               POP ESI
0041653A   64:8B18          MOV EBX,DWORD PTR FS:[EAX]
0041653D   8B1B             MOV EBX,DWORD PTR DS:[EBX]
0041653F   8D63 D6          LEA ESP,DWORD PTR DS:[EBX-2A]
00416542   5D               POP EBP
00416543   8D8E CB020000    LEA ECX,DWORD PTR DS:[ESI+2CB]  ;004167B3
00416549   894B 04          MOV DWORD PTR DS:[EBX+4],ECX  ;设置新的 SEH 处理入口 004167B3
0041654C   64:891D 00000000 MOV DWORD PTR FS:[0],EBX
00416553   8B3C24           MOV EDI,DWORD PTR SS:[ESP]
00416556   FF77 08          PUSH DWORD PTR DS:[EDI+8]    ;临时缓冲区地址
00416559   FF95 A0070000    CALL DWORD PTR SS:[EBP+7A0]         ;kernel32.GlobalFree,释放临时缓冲区
0041655F   81C7 3D000000    ADD EDI,3D
00416565   6A 0E            PUSH 0E
00416567   59               POP ECX
00416568   F3:A4            REP MOVSB        ;拷贝 0Eh 字节加密的数据,和以后的 oep 相关
0041656A   FF33             PUSH DWORD PTR DS:[EBX]
0041656C   56               PUSH ESI
0041656D   57               PUSH EDI
0041656E   8DB7 55010000    LEA ESI,DWORD PTR DS:[EDI+155]
00416574   8BCE             MOV ECX,ESI
00416576   2BCF             SUB ECX,EDI
00416578   F3:AA            REP STOS BYTE PTR ES:[EDI]    ;清除壳区段一些不用的数据
0041657A   60               PUSHAD
0041657B   FFE0             JMP EAX        ;跳到 0 地址去运行,会产生异常
```
上面的代码异常后来到这里,主要完成解压缩后的数据修复还原:
```
004167B3   33C0             XOR EAX,EAX
004167B5   64:8B18          MOV EBX,DWORD PTR FS:[EAX]
004167B8   8B1B             MOV EBX,DWORD PTR DS:[EBX]
004167BA   8D63 AE          LEA ESP,DWORD PTR DS:[EBX-52]
004167BD   61               POPAD
004167BE   833E 00          CMP DWORD PTR DS:[ESI],0    ;指向压缩数据表
004167C1  0F84 B6FDFFFF    JE petgui.0041657D      ;处理结束跳到
004167C7   8B7E 08          MOV EDI,DWORD PTR DS:[ESI+8]  ;解压缩后的目的 RVA
004167CA   03FD             ADD EDI,EBP        ;解压缩后的目的 RVA
004167CC   8B4E 0C          MOV ECX,DWORD PTR DS:[ESI+C]  ;需要补充的 0 的个数
004167CF   D1F9             SAR ECX,1
004167D1   51               PUSH ECX
004167D2   72 15            JB SHORT petgui.004167E9    ;如果是代码段,要先还原优化代码
004167D4   037E 04          ADD EDI,DWORD PTR DS:[ESI+4]  ;解压缩后的长度
004167D7   C1F9 02          SAR ECX,2
004167DA   33C0             XOR EAX,EAX
004167DC   F3:AB            REP STOS DWORD PTR ES:[EDI]    ;补充 0
004167DE   59               POP ECX
004167DF   83E1 03          AND ECX,3
004167E2   F3:AA            REP STOS BYTE PTR ES:[EDI]
004167E4   83C6 14          ADD ESI,14
004167E7  ^EB D5            JMP SHORT petgui.004167BE
004167E9   8B5E 04          MOV EBX,DWORD PTR DS:[ESI+4]  ;开始还原压缩优化代码
004167EC   83EB 06          SUB EBX,6
004167EF   33D2             XOR EDX,EDX
004167F1   3BD3             CMP EDX,EBX
004167F3  7D DF            JGE SHORT petgui.004167D4
004167F5   8A043A           MOV AL,BYTE PTR DS:[EDX+EDI]
004167F8   42               INC EDX
004167F9   3C E8            CMP AL,0E8        ;E8 类型 call
004167FB   74 12            JE SHORT petgui.0041680F
004167FD   3C E9            CMP AL,0E9        ;E9 类型 jmp
004167FF   74 0E            JE SHORT petgui.0041680F
00416801   3C 0F            CMP AL,0F        ;0F 类型 jcc
00416803  75 EC            JNZ SHORT petgui.004167F1
00416805   8A043A           MOV AL,BYTE PTR DS:[EDX+EDI]
00416808   24 F0            AND AL,0F0
0041680A   3C 80            CMP AL,80
0041680C  75 E3            JNZ SHORT petgui.004167F1
0041680E   42               INC EDX
0041680F   8B043A           MOV EAX,DWORD PTR DS:[EDX+EDI]
00416812   3C 0A            CMP AL,0A        ;标志字节
00416814  75 DB            JNZ SHORT petgui.004167F1
00416816   66:C1E8 08       SHR AX,8        ;对前面几种带相对 DWORD 偏移的代码进行偏移修复
0041681A   C1C0 10          ROL EAX,10
0041681D   86C4             XCHG AH,AL
0041681F   83C2 04          ADD EDX,4
00416822   2BC2             SUB EAX,EDX
00416824   89443A FC        MOV DWORD PTR DS:[EDX+EDI-4],EAX  ;写回修复值
00416828  ^EB C7            JMP SHORT petgui.004167F1
```
修复结束来到这里:
```
0041657D   5B               POP EBX
0041657E   5A               POP EDX
0041657F   64:8F05 00000000 POP DWORD PTR FS:[0]
00416586   58               POP EAX
00416587   6A 03            PUSH 3        ;初始输入表加密常数
00416589   53               PUSH EBX
0041658A   33DB             XOR EBX,EBX
0041658C   68 3E030000      PUSH 33E        ;校验长度
00416591   8B0C24           MOV ECX,DWORD PTR SS:[ESP]
00416594   0FBAE3 00        BT EBX,0
00416598   72 16            JB SHORT petgui.004165B0
0041659A   64:8B35 1C000000 MOV ESI,DWORD PTR FS:[1C]
004165A1   0FBAF6 00        BTR ESI,0
004165A5   64:0335 22000000 ADD ESI,DWORD PTR FS:[22]
004165AC   46               INC ESI
004165AD   66:33DE          XOR BX,SI
004165B0   321C11           XOR BL,BYTE PTR DS:[ECX+EDX]  ;计算一段壳处理代码的校验值
004165B3   C1C3 0F          ROL EBX,0F
004165B6   49               DEC ECX
004165B7  7D DB            JGE SHORT petgui.00416594
004165B9   8D48 3B          LEA ECX,DWORD PTR DS:[EAX+3B]  ;用前面校验值去解码 4 个 DWORD
004165BC   3119             XOR DWORD PTR DS:[ECX],EBX
004165BE   3159 04          XOR DWORD PTR DS:[ECX+4],EBX
004165C1   3159 08          XOR DWORD PTR DS:[ECX+8],EBX
004165C4   3159 0C          XOR DWORD PTR DS:[ECX+C],EBX
004165C7   59               POP ECX
004165C8   315C11 01        XOR DWORD PTR DS:[ECX+EDX+1],EBX
004165CC   33DB             XOR EBX,EBX
004165CE   8BF2             MOV ESI,EDX
004165D0   81BA 769BFEFF 46>CMP DWORD [EDX+FFFE9B76],0E3046  ;Virus detection 之检查程序的入口点
004165DA   75 21            JNZ SHORT petgui.004165FD
004165DC   81EE B2640100    SUB ESI,164B2
004165E2   0FB64E 06        MOVZX ECX,BYTE PTR DS:[ESI+6]
004165E6   6BC9 0A          IMUL ECX,ECX,0A
004165E9   66:81C1 3E00     ADD CX,3E
004165EE   331E             XOR EBX,DWORD PTR DS:[ESI]
004165F0   D3C3             ROL EBX,CL
004165F2   83C6 04          ADD ESI,4
004165F5   49               DEC ECX
004165F6  75 F6            JNZ SHORT petgui.004165EE
004165F8   3958 04          CMP DWORD PTR DS:[EAX+4],EBX  ;Virus detection 之检查 PE 头
004165FB   74 08            JE SHORT petgui.00416605
004165FD   83C4 2A          ADD ESP,2A
00416600  -E9 14CA0C00      JMP petgui.004E3019
00416605   8DB5 CC590100    LEA ESI,DWORD PTR SS:[EBP+159CC]  ;开始处理输入表
0041660B   8D8D 00080000    LEA ECX,DWORD PTR SS:[EBP+800]  ;输入表用的 dll 名字
00416611   8BD8             MOV EBX,EAX
00416613   833E 00          CMP DWORD PTR DS:[ESI],0
00416616   0F84 0E020000    JE petgui.0041682A      ;输入表处理结束则跳到 0041682A
0041661C   51               PUSH ECX
0041661D   51               PUSH ECX
0041661E   FF95 A4070000    CALL DWORD PTR SS:[EBP+7A4]    ;GetModuleHandleA
00416624   85C0             TEST EAX,EAX
00416626   75 11            JNZ SHORT petgui.00416639
00416628   83EC 04          SUB ESP,4
0041662B   FF95 90070000    CALL DWORD PTR SS:[EBP+790]    ;LoadLibraryA
00416631   85C0             TEST EAX,EAX
00416633   0F84 DF000000    JE petgui.00416718
00416639   8BF8             MOV EDI,EAX        ;模块基地址
0041663B   0340 3C          ADD EAX,DWORD PTR DS:[EAX+3C]  ;PE
0041663E   8B40 78          MOV EAX,DWORD PTR DS:[EAX+78]  ;export rva
00416641   FF7438 18        PUSH DWORD PTR DS:[EAX+EDI+18]  ;export 数目
00416645   8B4C38 24        MOV ECX,DWORD PTR DS:[EAX+EDI+24]
00416649   03CF             ADD ECX,EDI
0041664B   51               PUSH ECX
0041664C   8B4C38 20        MOV ECX,DWORD PTR DS:[EAX+EDI+20]
00416650   03CF             ADD ECX,EDI
00416652   51               PUSH ECX
00416653   FF7438 10        PUSH DWORD PTR DS:[EAX+EDI+10]  ;export ord 最小值
00416657   FF7438 14        PUSH DWORD PTR DS:[EAX+EDI+14]  ;export ord 最大值
0041665B   8B4438 1C        MOV EAX,DWORD PTR DS:[EAX+EDI+1C]  ;export 函数表
0041665F   03C7             ADD EAX,EDI
00416661   50               PUSH EAX
00416662   56               PUSH ESI
00416663   8B36             MOV ESI,DWORD PTR DS:[ESI]
00416665   03F5             ADD ESI,EBP
00416667   8B06             MOV EAX,DWORD PTR DS:[ESI]
00416669   85C0             TEST EAX,EAX
0041666B   0F84 81000000    JE petgui.004166F2      ;本模块结束则跳到 004166F2
00416671   79 2F            JNS SHORT petgui.004166A2    ;是否为序号方式
00416673   0FBAE0 1E        BT EAX,1E
00416677   72 29            JB SHORT petgui.004166A2
00416679   0FB7C0           MOVZX EAX,AX      ;通过序号直接去函数地址
0041667C   2B4424 0C        SUB EAX,DWORD PTR SS:[ESP+C]
00416680   0F82 AB000000    JB petgui.00416731
00416686   3B4424 08        CMP EAX,DWORD PTR SS:[ESP+8]
0041668A   0F83 A1000000    JNB petgui.00416731
00416690   C1E0 02          SHL EAX,2
00416693   034424 04        ADD EAX,DWORD PTR SS:[ESP+4]
00416697   8B00             MOV EAX,DWORD PTR DS:[EAX]
00416699   03C7             ADD EAX,EDI
0041669B   8906             MOV DWORD PTR DS:[ESI],EAX    ;取得的函数地址写到输入表
0041669D   83C6 04          ADD ESI,4
004166A0  ^EB C5            JMP SHORT petgui.00416667
004166A2   03C6             ADD EAX,ESI        ;Name 方式取函数地址
004166A4   50               PUSH EAX
004166A5   50               PUSH EAX
004166A6   57               PUSH EDI
004166A7   FF95 94070000    CALL DWORD PTR SS:[EBP+794]    ;GetProcAddress
004166AD   85C0             TEST EAX,EAX
004166AF   74 7F            JE SHORT petgui.00416730
004166B1   FF4C24 28        DEC DWORD PTR SS:[ESP+28]    ;下面一段是输入表加密选择处理部分
004166B5   7D 1F            JGE SHORT petgui.004166D6
004166B7   8B5424 24        MOV EDX,DWORD PTR SS:[ESP+24]  ;满足条件进行加密
004166BB   C602 E9          MOV BYTE PTR DS:[EDX],0E9    ;写 JMP 指令
004166BE   2BC2             SUB EAX,EDX
004166C0   83E8 05          SUB EAX,5
004166C3   8942 01          MOV DWORD PTR DS:[EDX+1],EAX  ;写 JMP 到 API 的相对偏移
004166C6   8BC2             MOV EAX,EDX
004166C8   83C2 05          ADD EDX,5
004166CB   895424 24        MOV DWORD PTR SS:[ESP+24],EDX
004166CF   83E2 07          AND EDX,7
004166D2   895424 28        MOV DWORD PTR SS:[ESP+28],EDX  ;调整输入表加密常数
004166D6   8906             MOV DWORD PTR DS:[ESI],EAX    ;函数地址写到输入表
004166D8   873C24           XCHG DWORD PTR SS:[ESP],EDI
004166DB   83C9 FF          OR ECX,FFFFFFFF
004166DE   33C0             XOR EAX,EAX
004166E0   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
004166E2   FD               STD
004166E3   F7D1             NOT ECX
004166E5   4F               DEC EDI
004166E6   F3:AA            REP STOS BYTE PTR ES:[EDI]    ;销毁函数名
004166E8   5F               POP EDI
004166E9   FC               CLD
004166EA   83C6 04          ADD ESI,4
004166ED  ^E9 75FFFFFF      JMP petgui.00416667      ;去处理下一个函数
004166F2   5E               POP ESI        ;一个模块处理结束
004166F3   83C4 18          ADD ESP,18
004166F6   8B16             MOV EDX,DWORD PTR DS:[ESI]
004166F8   03D5             ADD EDX,EBP
004166FA   8D43 47          LEA EAX,DWORD PTR DS:[EBX+47]
004166FD   8B4C24 04        MOV ECX,DWORD PTR SS:[ESP+4]
00416701   833A 00          CMP DWORD PTR DS:[EDX],0    ;遍历本模块生成的函数地址
00416704   74 12            JE SHORT petgui.00416718    ;本模块处理完成则跳去处理下一模块
00416706   3B1A             CMP EBX,DWORD PTR DS:[EDX]    ;下面一段代码根据函数加密情况去解码 OEP
00416708   8318 00          SBB DWORD PTR DS:[EAX],0    ;没加密会减 1,加密会减 2,最后 ror 3 位
0041670B   390A             CMP DWORD PTR DS:[EDX],ECX
0041670D   8318 00          SBB DWORD PTR DS:[EAX],0
00416710   83C2 04          ADD EDX,4
00416713   C108 03          ROR DWORD PTR DS:[EAX],3
00416716  ^EB E9            JMP SHORT petgui.00416701
00416718   C706 00000000    MOV DWORD PTR DS:[ESI],0
0041671E   5F               POP EDI
0041671F   83C9 FF          OR ECX,FFFFFFFF
00416722   33C0             XOR EAX,EAX
00416724   F2:AE            REPNE SCAS BYTE PTR ES:[EDI]  ;指向下一模块名字
00416726   8BCF             MOV ECX,EDI
00416728   83C6 04          ADD ESI,4
0041672B  ^E9 E3FEFFFF      JMP petgui.00416613      ;跳去处理下一模块
```
所有输入表处理完成以后来到这里:
```
0041682A   59               POP ECX
0041682B   5E               POP ESI
0041682C   FD               STD
0041682D   33C0             XOR EAX,EAX
0041682F   B9 57030000      MOV ECX,357
00416834   E8 04C80C00      CALL petgui.004E303D
```
再来到这里:
```
004E303D   5F               POP EDI
004E303E   F3:AA            REP STOS BYTE PTR ES:[EDI]    ;销毁壳处理代码
004E3040   61               POPAD
004E3041   66:9D            POPFW
004E3043   83C4 08          ADD ESP,8
004E3046  -E9 8CA0F2FF      JMP petgui.0040D0D7      ;跳到 OEP
```
前面是 level1~9 的 exe 的情况,level0 的入口稍有不同:
```
1000E10F  >  B8 00E00010      MOV EAX,b0.1000E000
1000E114   6A 00            PUSH 0        ;level 这里多 push 一个 0
1000E116   68 F67B0010      PUSH b0.10007BF6
```
level0 的解压算法也是特殊的:
```
1000E1A4   50               PUSH EAX
1000E1A5   800424 0C        ADD BYTE PTR SS:[ESP],0C    ;解压参数 8
1000E1A9   50               PUSH EAX
1000E1AA   800424 46        ADD BYTE PTR SS:[ESP],46    ;解压参数 7
1000E1AE   50               PUSH EAX
1000E1AF   800424 65        ADD BYTE PTR SS:[ESP],65    ;解压参数 6
1000E1B3   50               PUSH EAX
1000E1B4   800424 A1        ADD BYTE PTR SS:[ESP],0A1    ;解压参数 5
1000E1B8   50               PUSH EAX
1000E1B9   800424 BF        ADD BYTE PTR SS:[ESP],0BF    ;解压参数 4
1000E1BD   8B0B             MOV ECX,DWORD PTR DS:[EBX]
1000E1BF   83C3 14          ADD EBX,14
1000E1C2   8B53 F0          MOV EDX,DWORD PTR DS:[EBX-10]
1000E1C5   85D2             TEST EDX,EDX
1000E1C7  74 F4            JE SHORT b0.1000E1BD
1000E1C9   8B4424 18        MOV EAX,DWORD PTR SS:[ESP+18]
1000E1CD   8D3401           LEA ESI,DWORD PTR DS:[ECX+EAX]
1000E1D0   8B6C24 1C        MOV EBP,DWORD PTR SS:[ESP+1C]
1000E1D4   8B6D 08          MOV EBP,DWORD PTR SS:[EBP+8]
1000E1D7   8B4B FC          MOV ECX,DWORD PTR DS:[EBX-4]
1000E1DA   8BFD             MOV EDI,EBP
1000E1DC   F3:A5            REP MOVSD
1000E1DE   8BF5             MOV ESI,EBP
1000E1E0   8B7B F4          MOV EDI,DWORD PTR DS:[EBX-C]
1000E1E3   03F8             ADD EDI,EAX
1000E1E5   53               PUSH EBX        ;解压参数 3
1000E1E6   52               PUSH EDX        ;解压参数 2
1000E1E7   57               PUSH EDI        ;解压参数 1
1000E1E8   55               PUSH EBP        ;解压参数 0
1000E1E9   E8 2D000000      CALL b0.1000E21B      ;解压函数,函数比较长,不贴代码了
1000E1EE   85C0             TEST EAX,EAX
1000E1F0  0F84 DCFEFFFF    JE b0.1000E0D2
1000E1F6   5A               POP EDX
1000E1F7   59               POP ECX
1000E1F8   59               POP ECX
1000E1F9   5B               POP EBX
1000E1FA  ^EB C1            JMP SHORT b0.1000E1BD
```
对于 dll 在输入表处理结束后,会有一段重定位表的处理代码:
```
10007F4C   8D8D 00D00000    LEA ECX,DWORD PTR SS:[EBP+D000]  ;指向加密的 reloc 表
10007F52   8BF9             MOV EDI,ECX
10007F54   8BD5             MOV EDX,EBP
10007F56   8B01             MOV EAX,DWORD PTR DS:[ECX]
10007F58   85C0             TEST EAX,EAX
10007F5A   74 1A            JE SHORT b0.10007F76    ;0 结束标志
10007F5C   3C FF            CMP AL,0FF
10007F5E   75 0A            JNZ SHORT b0.10007F6A
10007F60   C1E8 08          SHR EAX,8        ;如果是 0FFh 表示长地址跨越
10007F63   03D0             ADD EDX,EAX        ;
10007F65   83C1 04          ADD ECX,4        ;包括前面的 0FFh 一共占用 4 字节
10007F68  ^EB EC            JMP SHORT b0.10007F56
10007F6A   41               INC ECX        ;不是 0FF 的,怎只用单字节表示一项
10007F6B   25 FF000000      AND EAX,0FF        ;表示和前一项重定位地址的差
10007F70   03D0             ADD EDX,EAX        ;累加差值得到地址
10007F72   012A             ADD DWORD PTR DS:[EDX],EBP    ;修复重定位
10007F74  ^EB E0            JMP SHORT b0.10007F56
```
如果加壳选项没选 Mangle imports 则输入表加密部分的处理代码会是一些 nop 字节



# 脱壳机实现原理

1. 以 `SEC_IMAGE` 方式 `CreateFileMapping`, 得到展开的 `PE` 映像, 并复制到一块新分配的内存
2. 通过入口点简单判断, 取得第一处的 `SEH` 处理地址和压缩数据表
3. 根据后面的一些代码特征判断一下压缩算法是哪种
4. 用相对应的解压算法根据压缩数据表完成解压缩
5. 以前面的 `SEH` 处理地址为入口, 从已解压的壳处理代码中得到一些关键数据和加密常量
6. 修复 `E8`, `E9`, 和 `0F 8?` 三种类型代码的偏移值
7. 修复输入表, 并同时完成 `OEP` 的解码
8. 如果有重定位表则修复重定位表
9. 修复 `PE` 头, 做些简单优化, 以减小体积
10. 把内存中的数据逐块写到输出文件, 完成脱壳

`4dd466d0e1e0cc123afbe504da2462ca`
