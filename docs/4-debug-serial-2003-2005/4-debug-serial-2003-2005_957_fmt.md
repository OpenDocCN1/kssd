
# 易用仓库管理软件 2.50 单机版--算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-63.htm`](https://www.pediy.com/kssd/pediy07/pediy7-63.htm)

==================================================================================  

【工程作者】深海游侠【OCN】【CZG】  

【作者邮箱】`shenhaiyouxia@163.com`  

==================================================================================  

【软件名称】易用仓库管理软件 2.50 单机版  

【下载地址】`http://www.newhua.com/soft/30450.htm`  

【所受限制】45 天时间限制  

【加壳保护】无壳  

【软件介绍】  

易用仓库管理软件是一款通用性极强的仓库及货物管理软件，软件支持多仓库管理.软件界面直观、操作简单，支持全键盘操作；软件支持入库、出库、退库、调库、借出、归还、盘点等多项货物操作流程，各种数据可以自动转换为 `EXCEL` 数据格式。软件提供了完备的帐务系统，可以随时查询或打印月记帐、日记帐。能自动对库存超限的商品报警，多种报表功输出功能，让你的仓库管理工作轻松自如。软件可以多台电脑联网使用，数据共享，同时操作各项软件功能。  

==================================================================================  

【工程平台】`Win 9X`  

【调试工具】`TRW2000`, `OD1.10B(复制贴代码)`, `Peid0.93`  

==================================================================================  

【破解过程】  

话不多说.首先查壳.`Borland Delphi 6.0 - 7.0` 少见.直接 `TRW2000` 载入,下万能断点.顺利断下,按 N 次 `F12` 跳过 `Borland Delphi 6.0 - 7.0` 熟悉的开头代码,我们来到这里:(软件也有注册错误提示.可有多种选择下断点,适合新手练手)  

```
00614692      8BC0           MOV EAX,EAX  
00614694   /. 55             PUSH EBP  
00614695   |. 8BEC          MOV EBP,ESP  
00614697   |. 6A 00         PUSH 0  
00614699   |. 6A 00         PUSH 0  
0061469B   |. 53            PUSH EBX  
0061469C   |. 8BD8         MOV EBX,EAX  
0061469E   |. 33C0         XOR EAX,EAX  
006146A0   |. 55            PUSH EBP  
006146A1   |. 68 54476100   PUSH DEPOT.00614754  
006146A6   |. 64:FF30      PUSH DWORD PTR FS:[EAX]  
006146A9   |. 64:8920      MOV DWORD PTR FS:[EAX],ESP  
006146AC   |. 8D55 FC      LEA EDX,DWORD PTR SS:[EBP-4]  
006146AF   |. 8B83 FC020000 MOV EAX,DWORD PTR DS:[EBX+2FC]  
006146B5   |. E8 02BFE3FF  CALL DEPOT.004505BC  
006146BA   |. 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
006146BD   |. E8 CE07DFFF  CALL DEPOT.00404E90  
006146C2   |. 85C0         TEST EAX,EAX  
006146C4   |. 75 29        JNZ SHORT DEPOT.006146EF             //是否输入用户名  
006146C6   |. 6A 40        PUSH 40  
006146C8   |. 68 60476100  PUSH DEPOT.00614760  
006146CD   |. 68 68476100  PUSH DEPOT.00614768  
006146D2   |. 8BC3         MOV EAX,EBX  
006146D4   |. E8 0B28E4FF  CALL DEPOT.00456EE4  
006146D9   |. 50            PUSH EAX  
006146DA   |. E8 3D3CDFFF  CALL <JMP.&user32.MessageBoxA>        //没有输入用户名  
006146DF   |. 8B83 FC020000 MOV EAX,DWORD PTR DS:[EBX+2FC]  
006146E5   |. 8B10         MOV EDX,DWORD PTR DS:[EAX]  
006146E7   |. FF92 C0000000 CALL DWORD PTR DS:[EDX+C0]  
006146ED   |. EB 4A        JMP SHORT DEPOT.00614739  
006146EF   |> 8D55 F8      LEA EDX,DWORD PTR SS:[EBP-8]  
006146F2   |. 8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]  
006146F8   |. E8 BFBEE3FF  CALL DEPOT.004505BC  
006146FD   |. 8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]  
00614700   |. E8 8B07DFFF  CALL DEPOT.00404E90  
00614705   |. 85C0         TEST EAX,EAX  
00614707   |. 75 29        JNZ SHORT DEPOT.00614732             //是否输入注册码  
00614709   |. 6A 40        PUSH 40  
0061470B   |. 68 60476100  PUSH DEPOT.00614760  
00614710   |. 68 78476100  PUSH DEPOT.00614778  
00614715   |. 8BC3         MOV EAX,EBX  
00614717   |. E8 C827E4FF  CALL DEPOT.00456EE4  
0061471C   |. 50            PUSH EAX  
0061471D   |. E8 FA3BDFFF  CALL <JMP.&user32.MessageBoxA>          //没有输入注册码  
00614722   |. 8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]  
00614728   |. 8B10         MOV EDX,DWORD PTR DS:[EAX]  
0061472A   |. FF92 C0000000 CALL DWORD PTR DS:[EDX+C0]  
00614730   |. EB 07        JMP SHORT DEPOT.00614739  
00614732   |> 8BC3         MOV EAX,EBX                   //EBX=字符 l>a 可疑  
00614734   |. E8 5FF9FFFF  CALL DEPOT.00614098            //第一次跟大批这里就直接出错,没错,跟进这个 CALL 吧!  
00614739   |> 33C0         XOR EAX,EAX  
0061473B   |. 5A            POP EDX  
0061473C   |. 59            POP ECX  
0061473D   |. 59            POP ECX  
0061473E   |. 64:8910      MOV DWORD PTR FS:[EAX],EDX  
00614741   |. 68 5B476100  PUSH DEPOT.0061475B  
00614746   |> 8D45 F8      LEA EAX,DWORD PTR SS:[EBP-8]  
00614749   |. BA 02000000  MOV EDX,2  
0061474E   |. E8 A904DFFF  CALL DEPOT.00404BFC  
00614753   |\. C3            RETN  
00614734   |. E8 5FF9FFFF  CALL DEPOT.00614098  

00614098   $ 55             PUSH EBP  
00614099   . 8BEC          MOV EBP,ESP  
0061409B   . 33C9         XOR ECX,ECX  
0061409D   . 51            PUSH ECX  
0061409E   . 51            PUSH ECX  
0061409F   . 51            PUSH ECX  
006140A0   . 51            PUSH ECX  
006140A1   . 51            PUSH ECX  
006140A2   . 53            PUSH EBX  
006140A3   . 56            PUSH ESI  
006140A4   . 57            PUSH EDI  
006140A5   . 8945 FC      MOV DWORD PTR SS:[EBP-4],EAX  
006140A8   . 33C0         XOR EAX,EAX  
006140AA   . 55            PUSH EBP  
006140AB   . 68 E9416100  PUSH DEPOT.006141E9  
006140B0   . 64:FF30      PUSH DWORD PTR FS:[EAX]  
006140B3   . 64:8920      MOV DWORD PTR FS:[EAX],ESP  
006140B6   . 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
006140B9   . E8 AE020000  CALL DEPOT.0061436C            //不用说了,关键 CALL,跟进!  
006140BE   . 84C0         TEST AL,AL  
006140C0   . 0F84 DB000000 JE DEPOT.006141A1             //熟悉的关键跳.  
006140C6   . 33C0         XOR EAX,EAX  
006140C8   . 55            PUSH EBP  
006140C9   . 68 85416100  PUSH DEPOT.00614185  
006140CE   . 64:FF30      PUSH DWORD PTR FS:[EAX]  
006140D1   . 64:8920      MOV DWORD PTR FS:[EAX],ESP  
006140D4   . B2 01        MOV DL,1  
006140D6   . A1 30854700  MOV EAX,DWORD PTR DS:[478530]  
006140DB   . E8 BC45E6FF  CALL DEPOT.0047869C  
006140E0   . 8BD8         MOV EBX,EAX  
006140E2   . BA 02000080  MOV EDX,80000002  
006140E7   . 8BC3         MOV EAX,EBX  
006140E9   . E8 8A46E6FF  CALL DEPOT.00478778  
006140EE   . B1 01        MOV CL,1  
006140F0   . BA 00426100  MOV EDX,DEPOT.00614200             ;  ASCII "Software\\zy\\Depot" //注册表保存注册信息的位置  
006140F5   . 8BC3         MOV EAX,EBX  
006140F7   . E8 C047E6FF  CALL DEPOT.004788BC  
006140FC   . 8D55 F4      LEA EDX,DWORD PTR SS:[EBP-C]  
006140FF   . 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
00614102   . 8B80 FC020000 MOV EAX,DWORD PTR DS:[EAX+2FC]  
00614108   . E8 AFC4E3FF  CALL DEPOT.004505BC  
0061410D   . 8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]  
00614110   . 8D55 F8      LEA EDX,DWORD PTR SS:[EBP-8]  
00614113   . E8 985ADFFF  CALL DEPOT.00409BB0  
00614118   . 8B4D F8      MOV ECX,DWORD PTR SS:[EBP-8]  
0061411B   . BA 1C426100  MOV EDX,DEPOT.0061421C             ;  ASCII "Name"     //用户名键值  
00614120   . 8BC3         MOV EAX,EBX  
00614122   . E8 3149E6FF  CALL DEPOT.00478A58  
00614127   . 8D55 EC      LEA EDX,DWORD PTR SS:[EBP-14]  
0061412A   . 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
0061412D   . 8B80 00030000 MOV EAX,DWORD PTR DS:[EAX+300]  
00614133   . E8 84C4E3FF  CALL DEPOT.004505BC  
00614138   . 8B45 EC      MOV EAX,DWORD PTR SS:[EBP-14]  
0061413B   . 8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]  
0061413E   . E8 6D5ADFFF  CALL DEPOT.00409BB0  
00614143   . 8B4D F0      MOV ECX,DWORD PTR SS:[EBP-10]  
00614146   . BA 2C426100  MOV EDX,DEPOT.0061422C             ;  ASCII "Pass"     //注册码键值  
0061414B   . 8BC3         MOV EAX,EBX  
0061414D   . E8 0649E6FF  CALL DEPOT.00478A58  
00614152   . 8BC3         MOV EAX,EBX  
00614154   . E8 6BFBDEFF  CALL DEPOT.00403CC4  
00614159   . 6A 40        PUSH 40  
0061415B   . 68 34426100  PUSH DEPOT.00614234  
00614160   . 68 40426100  PUSH DEPOT.00614240  
00614165   . 8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]  
00614168   . E8 772DE4FF  CALL DEPOT.00456EE4
```



```asm
0061416D   .   50             PUSH EAX                         ; |hOwner  
0061416E   .   E8 A941DFFF    CALL <JMP.&user32.MessageBoxA>   ; \MessageBoxA       //注册成功:)  
00614173   .   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
00614176   .   E8 15A6E5FF    CALL DEPOT.0046E790  
0061417B   .   33C0           XOR EAX,EAX  
0061417D   .   5A             POP EDX  
0061417E   .   59             POP ECX  
0061417F   .   59             POP ECX  
00614180   .   64:8910        MOV DWORD PTR FS:[EAX],EDX  
00614183   .   EB 36          JMP SHORT DEPOT.006141BB  
00614185   .^  E9 1A00DFFF    JMP DEPOT.004041A4  
0061418A   .   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
0061418D   .   E8 FEA5E5FF    CALL DEPOT.0046E790  
00614192   .   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
00614195   .   E8 1A010000    CALL DEPOT.006142B4  
0061419A   .   E8 3104DFFF    CALL DEPOT.004045D0  
0061419F   .   EB 1A          JMP SHORT DEPOT.006141BB  
006141A1   >   6A 40          PUSH 40  
006141A3   .   68 34426100    PUSH DEPOT.00614234  
006141A8   .   68 90426100    PUSH DEPOT.00614290  
006141AD   .   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
006141B0   .   E8 2F2DE4FF    CALL DEPOT.00456EE4  
006141B5   .   50             PUSH EAX                         ; |hOwner  
006141B6   .   E8 6141DFFF    CALL <JMP.&user32.MessageBoxA>   ; \MessageBoxA        //注册失败:(  
006141BB   >   33C0           XOR EAX,EAX  
006141BD   .   5A             POP EDX  
006141BE   .   59             POP ECX  
006141BF   .   59             POP ECX  
006141C0   .   64:8910        MOV DWORD PTR FS:[EAX],EDX  
006141C3   .   68 F0416100    PUSH DEPOT.006141F0  
006141C8   >   8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]  
006141CB   .   E8 080ADFFF    CALL DEPOT.00404BD8  
006141D0   .   8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]  
006141D3   .   E8 000ADFFF    CALL DEPOT.00404BD8  
006141D8   .   8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]  
006141DB   .   E8 F809DFFF    CALL DEPOT.00404BD8  
006141E0   .   8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]  
006141E3   .   E8 F009DFFF    CALL DEPOT.00404BD8  
006141E8   .   C3             RETN  
006141E9   .^ E9 6A02DFFF     JMP DEPOT.00404458  
006141EE   .^ EB D8           JMP SHORT DEPOT.006141C8  
006141F0   .   5F             POP EDI  
006141F1   .   5E             POP ESI  
006141F2   .   5B             POP EBX  
006141F3   .   8BE5           MOV ESP,EBP  
006141F5   .   5D             POP EBP  
006141F6   .   C3             RETN
```

下面部分为了省略篇幅,我只说主要算法!

```asm
006140B9   .   E8 AE020000    CALL DEPOT.0061436C  

|  
0061436C  /$   55             PUSH EBP  
0061436D  |.   8BEC           MOV EBP,ESP  
0061436F  |.   33C9           XOR ECX,ECX  
00614371  |.   51             PUSH ECX  
00614372  |.   51             PUSH ECX  
00614373  |.   51             PUSH ECX  
00614374  |.   51             PUSH ECX  
00614375  |.   51             PUSH ECX  
00614376  |.   51             PUSH ECX  
00614377  |.   51             PUSH ECX  
00614378  |.   53             PUSH EBX  
00614379  |.   56             PUSH ESI  
0061437A  |.   8BF0           MOV ESI,EAX  
0061437C  |.   33C0           XOR EAX,EAX  
0061437E  |.   55             PUSH EBP  
0061437F  |.   68 4C446100    PUSH DEPOT.0061444C  
00614384  |.   64:FF30        PUSH DWORD PTR FS:[EAX]  
00614387  |.   64:8920        MOV DWORD PTR FS:[EAX],ESP  
0061438A  |.   8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]  
0061438D  |.   8B86 00030000  MOV EAX,DWORD PTR DS:[ESI+300]  
00614393  |.   E8 24C2E3FF    CALL DEPOT.004505BC  
00614398  |.   8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]         //假码出现!  
0061439B  |.   8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]  
0061439E  |.   E8 0D58DFFF    CALL DEPOT.00409BB0  
006143A3  |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
006143A6  |.   50             PUSH EAX  
006143A7  |.   8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]  
006143AA  |.   8B86 FC020000  MOV EAX,DWORD PTR DS:[ESI+2FC]  
006143B0  |.   E8 07C2E3FF    CALL DEPOT.004505BC  
006143B5  |.   8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]        //假用户名出现!  
006143B8  |.   8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]  
006143BB  |.   E8 F057DFFF    CALL DEPOT.00409BB0  
006143C0  |.   8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]  
006143C3  |.   8D4D F4        LEA ECX,DWORD PTR SS:[EBP-C]  
006143C6  |.   8BC6           MOV EAX,ESI  
006143C8  |.   E8 CF000000    CALL DEPOT.0061449C                  //算法 CALL,再跟.(不要怕麻烦)  
006143CD  |.   8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]         //那这个一定是真码了,           ★ 内存注册机  
006143D0  |.   58             POP EAX                               //猜都能猜到一定是假码出栈比较  
006143D1  |.   E8 FE0BDFFF    CALL DEPOT.00404FD4                  //熟悉的比较跳转格式  
006143D6  |.   75 3A          JNZ SHORT DEPOT.00614412  
006143D8  |.   B3 01          MOV BL,1  
006143DA  |.   8D55 E8        LEA EDX,DWORD PTR SS:[EBP-18]  
006143DD  |.   8B86 FC020000  MOV EAX,DWORD PTR DS:[ESI+2FC]  
006143E3  |.   E8 D4C1E3FF    CALL DEPOT.004505BC  
006143E8  |.   8B55 E8        MOV EDX,DWORD PTR SS:[EBP-18]  
006143EB  |.   B8 5C9E6500    MOV EAX,DEPOT.00659E5C  
006143F0  |.   E8 3708DFFF    CALL DEPOT.00404C2C  
006143F5  |.   8D55 E4        LEA EDX,DWORD PTR SS:[EBP-1C]  
006143F8  |.   8B86 00030000  MOV EAX,DWORD PTR DS:[ESI+300]  
006143FE  |.   E8 B9C1E3FF    CALL DEPOT.004505BC  
00614403  |.   8B55 E4        MOV EDX,DWORD PTR SS:[EBP-1C]  
00614406  |.   B8 609E6500    MOV EAX,DEPOT.00659E60  
0061440B  |.   E8 1C08DFFF    CALL DEPOT.00404C2C  
00614410  |.   EB 02          JMP SHORT DEPOT.00614414  
00614412  |>   33DB           XOR EBX,EBX  
00614414  |>   33C0           XOR EAX,EAX  
00614416  |.   5A             POP EDX  
00614417  |.   59             POP ECX  
00614418  |.   59             POP ECX  
00614419  |.   64:8910        MOV DWORD PTR FS:[EAX],EDX  
0061441C  |.   68 53446100    PUSH DEPOT.00614453  
00614421  |>   8D45 E4        LEA EAX,DWORD PTR SS:[EBP-1C]  
00614424  |.   BA 03000000    MOV EDX,3  
00614429  |.   E8 CE07DFFF    CALL DEPOT.00404BFC  
0061442E  |.   8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]  
00614431  |.   BA 02000000    MOV EDX,2  
00614436  |.   E8 C107DFFF    CALL DEPOT.00404BFC  
0061443B  |.   8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]  
0061443E  |.   E8 9507DFFF    CALL DEPOT.00404BD8  
00614443  |.   8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]  
00614446  |.   E8 8D07DFFF    CALL DEPOT.00404BD8  
0061444B  \\.  C3             RETN  
0061444C   .^ E9 0700DFFF     JMP DEPOT.00404458  
00614451   .^ EB CE           JMP SHORT DEPOT.00614421  
00614453   .   8BC3           MOV EAX,EBX  
00614455   .   5E             POP ESI  
00614456   .   5B             POP EBX  
00614457   .   8BE5           MOV ESP,EBP  
00614459   .   5D             POP EBP  
0061445A   .   C3             RETN  

0061449C  /$   55             PUSH EBP  
0061449D  |.   8BEC           MOV EBP,ESP  
0061449F  |.   51             PUSH ECX  
006144A0  |.   B9 04000000    MOV ECX,4  
006144A5  |>   6A 00          /PUSH 0  
006144A7  |.   6A 00          |PUSH 0  
006144A9  |.   49             |DEC ECX  
006144AA  |.^  75 F9          \JNZ SHORT DEPOT.006144A5  
006144AC  |.   51             PUSH ECX  
006144AD  |.   874D FC        XCHG DWORD PTR SS:[EBP-4],ECX  
006144B0  |.   53             PUSH EBX  
006144B1  |.   56             PUSH ESI  
006144B2  |.   57             PUSH EDI  
006144B3  |.   8BF9           MOV EDI,ECX  
006144B5  |.   8955 FC        MOV DWORD PTR SS:[EBP-4],EDX  
006144B8  |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]         //用户名出现  
006144BB  |.   E8 B80BDFFF    CALL DEPOT.00405078  
006144C0  |.   33C0           XOR EAX,EAX  
006144C2  |.   55             PUSH EBP  
006144C3  |.   68 5D466100    PUSH DEPOT.0061465D  
006144C8  |.   64:FF30        PUSH DWORD PTR FS:[EAX]  
006144CB  |.   64:8920        MOV DWORD PTR FS:[EAX],ESP  
006144CE  |.   8BC7           MOV EAX,EDI  
006144D0  |.   E8 0307DFFF    CALL DEPOT.00404BD8  
006144D5  |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
```



```asm
006144D8  |.  E8 B309DFFF   CALL DEPOT.00404E90

006144DD  |.  8BF0         MOV ESI,EAX

006144DF  |.  85F6         TEST ESI,ESI

006144E1  |.  7E 26        JLE SHORT DEPOT.00614509             //是否有用户名?过去这里下面开始计算

006144E3  |.  BB 01000000  MOV EBX,1                            //EBX=1 计数器

006144E8  |>  8D4D EC      /LEA ECX,DWORD PTR SS:[EBP-14]

006144EB  |.  8B45 FC      |MOV EAX,DWORD PTR SS:[EBP-4]        //EAX=用户名

006144EE  |.  0FB64418 FF  |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]   //取其中一位的 ASCII 码(关键看计数器)

006144F3  |.  33D2         |XOR EDX,EDX                         //清 EDX,等下用来保存

006144F5  |.  E8 EE5BDFFF  |CALL DEPOT.0040A0E8                 //由刚才的 ASCII 码转为字符.(数字字符,也就是说刚才你那一位 ASCII 码是 31,那么这里就转为纯字符的 31!)

006144FA  |.  8B55 EC      |MOV EDX,DWORD PTR SS:[EBP-14]

006144FD  |.  8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]        //EAX=转换结果

00614500  |.  E8 9309DFFF  |CALL DEPOT.00404E98                 //排列起来,先来后到!

00614505  |.  43           |INC EBX                             //计数器+1

00614506  |.  4E           |DEC ESI

00614507  |.^ 75 DF         \JNZ SHORT DEPOT.006144E8           //循环

00614509  |>  \8B45 F8     MOV EAX,DWORD PTR SS:[EBP-8]         //排列后的结果.我输入的用户名是 12345678,到这里结果就是 3132333435363738

0061450C  |.  E8 7F09DFFF  CALL DEPOT.00404E90                  //取位数

00614511  |.  8BF0         MOV ESI,EAX

00614513  |.  85F6         TEST ESI,ESI

00614515  |.  7E 2C        JLE SHORT DEPOT.00614543             //是否有?下面进行第二轮转换!

00614517  |.  BB 01000000  MOV EBX,1                            //熟悉的计数器

0061451C  |>  8B45 F8      /MOV EAX,DWORD PTR SS:[EBP-8]        //EAX=上边转换的结果

0061451F  |.  E8 6C09DFFF  |CALL DEPOT.00404E90                 //取位数

00614524  |.  2BC3         |SUB EAX,EBX                         //看清楚,位数-计数器,可以知道这次是反着取!

00614526  |.  8B55 F8      |MOV EDX,DWORD PTR SS:[EBP-8]        //EDX=上边转换的结果

00614529  |.  8A1402       |MOV DL,BYTE PTR DS:[EDX+EAX]        //反着取其中一位 ASCII 码

0061452C  |.  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]

0061452F  |.  E8 8408DFFF  |CALL DEPOT.00404DB8                 //取 ASCII 码反方向

00614534  |.  8B55 E8      |MOV EDX,DWORD PTR SS:[EBP-18]

00614537  |.  8D45 F4      |LEA EAX,DWORD PTR SS:[EBP-C]

0061453A  |.  E8 5909DFFF  |CALL DEPOT.00404E98                 //反着写,比如说我刚才的是 3132333435363738,经过这里循环以后就变成 8373635343332313

0061453F  |.  43           |INC EBX                             //计数器+1

00614540  |.  4E           |DEC ESI

00614541  |.^ 75 D9         \JNZ SHORT DEPOT.0061451C           //循环

00614543  |>  8D45 F8      LEA EAX,DWORD PTR SS:[EBP-8]         //EAX=3132333435363738  第一次循环结果

00614546  |.  50           PUSH EAX

00614547  |.  B9 04000000  MOV ECX,4

0061454C  |.  BA 01000000  MOV EDX,1

00614551  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //EAX=8373635343332313  第二次循环结果

00614554  |.  E8 8F0BDFFF  CALL DEPOT.004050E8                  //截取前 4 位

00614559  |.  8D45 F4      LEA EAX,DWORD PTR SS:[EBP-C]

0061455C  |.  50           PUSH EAX

0061455D  |.  B9 04000000  MOV ECX,4

00614562  |.  BA 05000000  MOV EDX,5

00614567  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //EAX=8373635343332313  第二次循环结果

0061456A  |.  E8 790BDFFF  CALL DEPOT.004050E8                  //再截取 4 位

0061456F  |.  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]         //EAX=8373

00614572  |.  E8 1909DFFF  CALL DEPOT.00404E90                  //取位数

00614577  |.  83F8 04      CMP EAX,4

0061457A  |.  7D 2F        JGE SHORT DEPOT.006145AB             //大于等于 4 跳,我的就是等于~不用经过下面计算~(也就是说,当你输入的用户名是 2 位以下的时候,要经过下面第三个循环的计算!)

0061457C  |.  8B45 F8      MOV EAX,DWORD PTR SS:[EBP-8]

0061457F  |.  E8 0C09DFFF  CALL DEPOT.00404E90

00614584  |.  8BD8         MOV EBX,EAX

选择部分(1)

00614586  |.  83FB 03      CMP EBX,3                            //这里开始如果用户名小于三位，进行下面补位运算

00614589  |.  7F 20        JG SHORT DEPOT.006145AB

0061458B  |>  8D4D E4      /LEA ECX,DWORD PTR SS:[EBP-1C]

★   这里无非就一种结果:

1>就是用户名只有是 1 位的时候,转成 ASCII 码就是 2 位,EAX=2 计算,结果固定!

0061458E  |.  8BC3         |MOV EAX,EBX                         //此时取你用户名 ASCII 码位数

00614590  |.  C1E0 02      |SHL EAX,2                           //逻辑左移 2 位(当乘法用)

00614593  |.  33D2         |XOR EDX,EDX

00614595  |.  E8 4E5BDFFF  |CALL DEPOT.0040A0E8                 //结果换成字符

0061459A  |.  8B55 E4      |MOV EDX,DWORD PTR SS:[EBP-1C]

0061459D  |.  8D45 F8      |LEA EAX,DWORD PTR SS:[EBP-8]

006145A0  |.  E8 F308DFFF  |CALL DEPOT.00404E98

006145A5  |.  43           |INC EBX

006145A6  |.  83FB 04      |CMP EBX,4

006145A9  |.^ 75 E0         \JNZ SHORT DEPOT.0061458B           //循环

ˉ

006145AB  |>  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]         //上边第二次截取的结果 EAX=6353

006145AE  |.  E8 DD08DFFF  CALL DEPOT.00404E90                  //再取

006145B3  |.  83F8 04      CMP EAX,4

006145B6  |.  7D 2F        JGE SHORT DEPOT.006145E7             //大于等于 4 跳,我还是等于~不用经过下面计算~(也就是说,当你输入用户名时候大于 2 位又小于 4 位的时候,要经过下面第三个循环的计算!这里不明白的朋友可以综合上边的解释看一下,我就不占篇幅了!)

006145B8  |.  8B45 F4      MOV EAX,DWORD PTR SS:[EBP-C]

006145BB  |.  E8 D008DFFF  CALL DEPOT.00404E90

006145C0  |.  8BD8         MOV EBX,EAX

006145C2  |.  83FB 03      CMP EBX,3

006145C5  |.  7F 20        JG SHORT DEPOT.006145E7

选择部分(2)

006145C7  |>  8D4D E0      /LEA ECX,DWORD PTR SS:[EBP-20]       //此时取你经过上边运算剩余用户名 ASCII 码位数

★   这里无非两种结果:

1>是用户名是 1,2 位,到这里已经没有剩余,此时 EAX=0 去计算.得到结果一样

2>是用户名是 3 位,到这里还有 1 位用户名剩余,转为 ASCII 码就是 2 位剩余.此时 EAX=2.结果相应改变!

006145CA  |.  8BC3         |MOV EAX,EBX

006145CC  |.  C1E0 02      |SHL EAX,2                           //逻辑左移 2 位(当乘法用)

006145CF  |.  33D2         |XOR EDX,EDX

006145D1  |.  E8 125BDFFF  |CALL DEPOT.0040A0E8                 //结果转换为字符!

006145D6  |.  8B55 E0      |MOV EDX,DWORD PTR SS:[EBP-20]

006145D9  |.  8D45 F4      |LEA EAX,DWORD PTR SS:[EBP-C]

006145DC  |.  E8 B708DFFF  |CALL DEPOT.00404E98

006145E1  |.  43           |INC EBX

006145E2  |.  83FB 04      |CMP EBX,4

006145E5  |.^ 75 E0         \JNZ SHORT DEPOT.006145C7

ˉ

006145E7  |>  8D45 F0      LEA EAX,DWORD PTR SS:[EBP-10]

006145EA  |.  BA 74466100  MOV EDX,DEPOT.00614674                ;  ASCII "DEPw245d54k"  //出现字符串!       ★  注册码中的字符

006145EF  |.  E8 7C06DFFF  CALL DEPOT.00404C70

006145F4  |.  8D45 DC      LEA EAX,DWORD PTR SS:[EBP-24]

006145F7  |.  50           PUSH EAX

006145F8  |.  B9 04000000  MOV ECX,4

006145FD  |.  BA 01000000  MOV EDX,1

00614602  |.  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]        //EAX=DEPw245d54k

00614605  |.  E8 DE0ADFFF  CALL DEPOT.004050E8                    //再取前 4 位

0061460A  |.  FF75 DC      PUSH DWORD PTR SS:[EBP-24]             //结果压栈,下 D 命令可看到 DEPw

0061460D  |.  68 88466100  PUSH DEPOT.00614688                    //下 D 可看到-字符(看来注册码有它连接)

00614612  |.  FF75 F8      PUSH DWORD PTR SS:[EBP-8]              //下 D 可看到 8373,是什么我不用多说了吧?不明白的看上边!

00614615  |.  8D45 D8      LEA EAX,DWORD PTR SS:[EBP-28]

00614618  |.  50           PUSH EAX

00614619  |.  B9 05000000  MOV ECX,5                              //看到这里我想不难猜到这两句话的意思了,EDX 5 就是取的位数,而 ECX 5 就是步长,相当于从字符串的第几位取~

0061461E  |.  BA 05000000  MOV EDX,5

00614623  |.  8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]        //EAX=DEPw245d54k(看了上边两句,也就是说这次从这串字符的第 5 位开始,一共取 5 位)

00614626  |.  E8 BD0ADFFF  CALL DEPOT.004050E8                    //实现

0061462B  |.  FF75 D8      PUSH DWORD PTR SS:[EBP-28]             //下 D 可到看结果 245d5

0061462E  |.  68 88466100  PUSH DEPOT.00614688                    //字符-

00614633  |.  FF75 F4      PUSH DWORD PTR SS:[EBP-C]              //下 D 可看到 6353

00614636  |.  8BC7         MOV EAX,EDI

00614638  |.  BA 06000000  MOV EDX,6
```



```asm
0061463D  |.  E8 0E09DFFF    CALL DEPOT.00404F50                 //不同的方法连起来.大概上边已经分配过.
00614642  |.  33C0           XOR EAX,EAX                          //到这里就可以看到结果了.    DEPw-8373245d5-6353
00614644  |.  5A             POP EDX
00614645  |.  59             POP ECX
00614646  |.  59             POP ECX
00614647  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX
0061464A  |.  68 64466100    PUSH DEPOT.00614664
0061464F  |>  8D45 D8        LEA EAX,DWORD PTR SS:[EBP-28]
00614652  |.  BA 0A000000    MOV EDX,0A
00614657  |.  E8 A005DFFF    CALL DEPOT.00404BFC
0061465C  \.  C3             RETN
0061465D   .^ E9 F6FDDEFF    JMP DEPOT.00404458
00614662   .^ EB EB          JMP SHORT DEPOT.0061464F
00614664   .  5F             POP EDI
00614665   .  5E             POP ESI
00614666   .  5B             POP EBX
00614667   .  8BE5          MOV ESP,EBP
00614669   .  5D             POP EBP
0061466A   .  C3             RETN
```

## 破解总结

软件大概有三种注册码，当然前两种也是在第三种的大体内做的小变形！

**1>** 用户名如果是 `1` 位。那么就要经过上边代码中的选择`(1)`和选择`(2)`。但总的形式都是这样：

`DEPw-**8C245d5-048C`

其中 `**` 是你输入用户名的 `16` 进制 `ASCII` 码的反写。

比如说用户名是 `1` 那么注册码就是：`DEPw-138C245d5-048C`

**2>** 用户名如果是 `2`, `3` 位，那么只要经过上边代码中选择`(2)`。

**1)** 用户名 `2` 位的形式是这样：`DEPw-****245d5-048C`

其中 `****` 是你输入用户名的 `16` 进制 `ASCII` 码的反写。注意：用户名字符也要反写！

比如说用户名是 `12` 那么注册码就是：`DEPw-2313245d5-048C`

**2)** 用户名 `3` 位的形式是这样：`DEPw-*****45d5-**8C`

其中 `*****` 是你输入用户名的 `16` 进制 `ASCII` 码的反写。注意：用户名字符也要反写！

比如说用户名是 `123` 那么注册码就是：`DEPw-3323245d5-138C`

**3>** 用户名在 `4` 位以上，不用经过选择代码，只用经过基本算法流程即可！

用户名 `4` 位以上的基本形式：`DEPw-****245d5-****`

其中 `****` 是你输入用户的"后四位"`16` 进制 `ASCII` 码的反写。注意：用户名字符也要反写！

比如说用户名是 `123456789` 那么注册码就是：`DEPw-9383245d5-7363`

注：具体转换请看代码部分！

## 内存注册机

*   中断地址：`6143D0`
*   中断次数：`1`
*   第一字节：`58`
*   字节长度：`1`
*   寄存器：`EDX`（内存方式）

## 注册保存信息

`HKEY_LOCAL_MACHINE\SOFTWARE\zy\Depot\Name`: `"chenxing"`

`HKEY_LOCAL_MACHINE\SOFTWARE\zy\Depot\Pass`: `"DEPw-76E6245d5-9687"`

## 破后感评

花了 `2` 个半小时，终于写完了这篇文章，我开始写的时候正好是刚过大年初一 `0` 点，算是个好兆头吧:)

这个软件防护措施很少，确实很适合像我这样的新手们学习。同时也希望其他新手们能从中学到些什么。只要自己多动手，就没有什么好怕的:)

如果你在我的文章中能学到点什么，那么我的目的也就达到了。

同时借这个机会，希望以前 `OCN` 的兄弟都能回来，现在 `OCN` 已经再次开放了。希望大家都能多去那里交流！

论坛地址：`http://ocn.e5v.com/`

最后祝所有的朋友们新年快乐，技术更上一层楼！

收工睡觉...

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;------深海游侠【OCN】【CZG】

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;------`2005.2.9` 午夜

================================================================

**【工程声明】** 本过程只供内部学习之用！如要转载请保持过程完整！

================================================================

`4dd466d0e1e0cc123afbe504da2462ca`
