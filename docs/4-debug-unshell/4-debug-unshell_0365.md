
# 神奇挂挂 3.7

> 原文：[`www.pediy.com/kssd/pediy06/pediy6475.htm`](https://www.pediy.com/kssd/pediy06/pediy6475.htm)

【脱文标题】 `ASProtect` 1.1b Registered SDK 之神奇挂挂 3.7 版脱壳 + 去暗桩

【脱文作者】 `weiyi75[Dfcg]`

【作者邮箱】 `weiyi75@sohu.com`

【作者主页】 Dfcg 官方大本营

【使用工具】 `Peid`, `Ollydbg1.10b`, `ImportREC1.42`, `Loadpe`

【脱壳平台】 `Win2000/XP`

【软件名称】 神奇挂挂 3.7 版

【下载地址】 [点击下载](http://www.tongtian.net/pediybbs/files/aspr_packed.rar)

【软件简介】 某外挂程序

【软件大小】 218K

【加壳方式】 `ASProtect` 1.1b Registered `[SAC]` -> `Alexey` `Solodovnikov`

【保护方式】 `ASProtect` 1.1b Registered `[SAC]` + SDK 防脱壳暗桩。

【脱壳声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：

--------------------------------------------------------------------------------

【脱壳内容】

首先 `Peid` 查壳，为 `ASProtect` 1.1b Registered `[SAC]` -> `Alexey` `Solodovnikov`, 脱壳后发现程序是 SDK 保护的，但通过二哥的教学 `AVI` `MPEG` `RM` `WMV` `Joiner4.61` 脱壳去暗桩 + 完美爆破.chm 和 `ASF-AVI-RM-WMV` `Repair` `V1.41` 脱壳去暗桩 + 汉化完美爆破.chm 两篇文章的脱壳经验，脱这个 SDK 壳易如反掌。脱壳难度中上等。

`ASProtect` 1.1b Registered 很少弄过，与现在的 `Asprotect1.2X` `SEH` 不同，不过也很容易。`SEH` 异常全部是由 13 个精心设计的非法指令 `SEH` 组成的，这样就无法用模拟跟踪找 `Oep` 了。二哥脱壳没有什么耐心，喜欢快。先大概了解了一下程序开始脱壳。

`OD` 载入程序，除了错误或有特权的指令外异常全部忽略, 1.1b 不检测 `OD`，根本无效隐藏。

```
00472001 >   60              PUSHAD  //外壳入口
00472002    E9 FD050000     JMP GoToMu.00472604
00472007    55              PUSH EBP
00472008    6967 B0 6DF7C43>IMUL ESP,DWORD PTR DS:[EDI-50],31C4F76D
0047200F    5A              POP EDX
00472010    B0 1D          MOV AL,1D
00472012    F66D 22         IMUL BYTE PTR SS:[EBP+22]
00472015    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00472016    66:B2 67        MOV DL,67
00472019    6A 8D          PUSH -73
0047201B    98              CWDE
0047201C    04 E9          ADD AL,0E9
0047201E    6E              OUTS DX,BYTE PTR ES:[EDI]                ; I/O 命令
0047201F    B2 67          MOV DL,67
00472021    EA 6DE524AD B06>JMP FAR 6DB0:AD24E56D                   ; 远距跳转
.........................................................
```

`F9` 运行。

```
011BFF94    8DC0            LEA EAX,EAX                              ; 非法使用寄存器
011BFF96    EB 01          JMP SHORT 011BFF99
011BFF98    68 648F0500     PUSH 58F64
011BFF9D    0000            ADD BYTE PTR DS:[EAX],AL
011BFF9F    00EB            ADD BL,CH
011BFFA1    02E8            ADD CH,AL
011BFFA3    0158 68        ADD DWORD PTR DS:[EAX+68],EBX
011BFFA6    98              CWDE
011BFFA7    E5 1B          IN EAX,1B                                ; I/O 命令
011BFFA9    0168 D0        ADD DWORD PTR DS:[EAX-30],EBP
011BFFAC    FF1B            CALL FAR FWORD PTR DS:[EBX]             ; 远距呼叫
011BFFAE    0168 7C        ADD DWORD PTR DS:[EAX+7C],EBP
011BFFB1    F5              CMC
011BFFB2    1B01            SBB EAX,DWORD PTR DS:[ECX]
011BFFB4    68 14F21B01     PUSH 11BF214
011BFFB9    68 B4E91B01     PUSH 11BE9B4
011BFFBE    68 70E71B01     PUSH 11BE770
011BFFC3    68 C8FB1B01     PUSH 11BFBC8
011BFFC8    C3              RETN
.........................................................
```

继续 `Shift+F9` 12 次到最后一次 `SEH` 异常。

`Alt+M` 打开内存镜像

内存镜像

| 地址 | 大小 | Owner | Section | Contains | 类型 | 访问 | 初始访问 | 映射为 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `00400000` | `00001000` | `GoToMu` | `PE header` | `Imag` | `R` | `RWE` | | |
| `00401000` | `0002F000` | `GoToMu` | `code` | `Imag` | `R` | `RWE` | | | // 当然对 Code 段下内存访问断点。

```
0042BA4F    55              PUSH EBP  //Shift+F9 中断在 Oep 入口,Loadpe 脱壳。
0042BA50    8BEC            MOV EBP,ESP
0042BA52    6A FF          PUSH -1
0042BA54    68 58404300     PUSH GoToMu.00434058
0042BA59    68 40B84200     PUSH GoToMu.0042B840                   ; JMP to
0042BA5E    MSVCRT._except_handler3; SE handler installation
0042BA5E    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
0042BA64    50              PUSH EAX
0042BA65    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
0042BA6C    83EC 68         SUB ESP,68
0042BA6F    53              PUSH EBX
0042BA70    56              PUSH ESI
0042BA71    57              PUSH EDI
0042BA72    8965 E8         MOV DWORD PTR SS:[EBP-18],ESP
0042BA75    33DB            XOR EBX,EBX
0042BA77    895D FC         MOV DWORD PTR SS:[EBP-4],EBX
0042BA7A    6A 02          PUSH 2
0042BA7C    FF15 C0054300   CALL DWORD PTR DS:[4305C0]             ; MSVCRT.__set_app_type
.......................................................
```

IAT 修复

运行 `ImportREC`，OEP 填入 `2BA4F`，自动搜索，获得输入信息，先用跟踪等级 1 修复，然后用 `Asprotect1.2X` 插件修复剩下的 2 个指针，还提示有 1 个指针无法修复，看看它的位置，在所有 `Dll` 的尾部，一般来说 `Asprotect` 插件和等级 1 几乎没有修复不了的指针，有一个已知的 Api 函数 `FindResourceA`。在尾部修复不了的更不可能。故确认其是垃圾指针，剪刀剪或删除这个指针，留下也不影响程序运行, 我是用剪刀剪的。

去除 SDK 暗桩。

`F8` 单步运行到这里

```
0042BB14  |.  E8 AD000000   CALL <JMP.&msvcrt._initterm>  //出错，Crlr+F2，重启 OD,F7 跟进。
00407700   .  E8 0B000000   CALL dumped_.00407710 //出错，F7 跟进。
0040748D   .  FF15 64B64300 CALL DWORD PTR DS:[43B664]
```

看信息框。

`DS:[0043B664]=011BC504` //壳中地址。

我们跟踪原程序看看这句是如何处理的。

```
0040748D    FF15 64B64300   CALL DWORD PTR DS:[43B664]  //跟进去发现其作用是从壳中解压代码，脱壳的人当然没有这部分代码，就看到空地址错误提示。
00407493    E9 09010000     JMP GoToMu.004075A1  //注意下面部分代码，经过上面的 Call 发生很大变化。
00407498    5C              POP ESP
00407499    21B7 5D32B612   AND DWORD PTR DS:[EDI+12B6325D],ESI
0040749F    AB              STOS DWORD PTR ES:[EDI]
004074A0    0A99 F1925FE2   OR BL,BYTE PTR DS:[ECX+E25F92F1]
004074A6    A1 840D8AD2     MOV EAX,DWORD PTR DS:[D28A0D84]
004074AB    DD              ???                               ; 未知命令
004074AC    A9 6E56158F     TEST EAX,8F15566E
004074B1   ^ 73 B0         JNB SHORT GoToMu.00407463
004074B3   ^ EB CD         JMP SHORT GoToMu.00407482
004074B5    A9 FE2551F5     TEST EAX,F55125FE
004074BA    B3 F5          MOV BL,0F5
004074BC    DBFB            FSTP EBX                          ; 非法使用寄存器
.....................................................................
```

```
011BC504    55              PUSH EBP  //壳领空代码，下面这些数据对我们不重要，它们的作用是解压程序需要的代码，就是验证壳还存在。于是我们 Ctrl+F9 返回程序，看它做了些什么。
011BC554    C2 0800         RETN 8  //我们看到它用一个 RETN 8 返回程序领空。
```

第一步要做的就是模拟解码，思路，随便在程序里找一个 `RETN 8` 返回程序模拟解码，然后复制原程序解码内容到脱壳程序里面，操作要小心。

```
011BC505    8BEC            MOV EBP,ESP
011BC507    53              PUSH EBX
011BC508    56              PUSH ESI
011BC509    8B5D 0C         MOV EBX,DWORD PTR SS:[EBP+C]
011BC50C    8B75 08         MOV ESI,DWORD PTR SS:[EBP+8]
011BC50F    8BC3            MOV EAX,EBX
011BC511    E8 5260FFFF     CALL 011B2568
011BC516    A3 CC251C01     MOV DWORD PTR DS:[11C25CC],EAX
011BC51B    8B15 CC251C01   MOV EDX,DWORD PTR DS:[11C25CC]  //解码过程。
011BC521    8BC6            MOV EAX,ESI
011BC523    8BCB            MOV ECX,EBX
011BC525    E8 2E61FFFF     CALL 011B2658
011BC52A    891D D0251C01   MOV DWORD PTR DS:[11C25D0],EBX
011BC530    53              PUSH EBX
011BC531    8BCE            MOV ECX,ESI
011BC533    8B15 C8251C01   MOV EDX,DWORD PTR DS:[11C25C8]
011BC539    A1 C4251C01     MOV EAX,DWORD PTR DS:[11C25C4]
011BC53E    E8 C9FBFFFF     CALL 011BC10C
011BC543    833D 9C251C01 0>CMP DWORD PTR DS:[11C259C],0
011BC54A    75 05          JNZ SHORT 011BC551
011BC54C    E8 5BFFFFFF     CALL 011BC4AC
..............................................
011BC554    C2 0800         RETN 8   //返回程序。
..............................................
```

于是我们随便在脱壳程序里面找个 `RETN 8`

```
004118AA  |.  C2 0800       RETN 8  //我找的这里。
0040748D   .  FF15 64B64300 CALL DWORD PTR DS:[43B664]
```

`DS:[0043B664]=011BC504` //壳中地址。

先修改第一处指向壳中的指针。

命令行 `DD 0043B664`

右键修改原 16 进制地址 `011BC504` 为 `004118AA`

修改 1 完成, 保存到文件中。

第二步，复制第一处解压代码。

```
0040748D    FF15 64B64300   CALL DWORD PTR DS:[43B664]             ; Crack.004118AA
```



# 壳代码分析与修复

## 1. 初始代码片段

```
00407493    E9 09010000     JMP Crack.004075A1   //继续 F8 步过壳中的上一句,看到连同这句代码发生很大变化。    

00407498    5C              POP ESP   

00407499    21B7 5D32B612   AND DWORD PTR DS:[EDI+12B6325D],ESI   

0040749F    AB              STOS DWORD PTR ES:[EDI]                   //这段是未解码时的代码    

004074A0    0A99 F1925FE2   OR BL,BYTE PTR DS:[ECX+E25F92F1]   

004074A6    A1 840D8AD2     MOV EAX,DWORD PTR DS:[D28A0D84]   

004074AB    DD              ???                                       ; 未知命令   

004074AC    A9 6E56158F     TEST EAX,8F15566E   

004074B1  ^ 73 B0           JNB SHORT Crack.00407463   

004074B3  ^ EB CD           JMP SHORT Crack.00407482   

004074B5    A9 FE2551F5     TEST EAX,F55125FE   

004074BA    B3 F5           MOV BL,0F5   

004074BC    DBFB            FSTP EBX                                  ; 非法使用寄存器   

................................................................................   
```

## 2. 解码后代码分析

```
00407493   /E9 01000000     JMP GoToMu.00407499   //解码后，这里有个花指令,对我们二进制复制代码修复程序有影响，于是右键分析-分析代码。    

00407498   |9A 6A10E81A 3F0>CALL FAR 023F:1AE8106A                               ; 远距呼叫   

0040749F    0083 C4048945   ADD BYTE PTR DS:[EBX+458904C4],AL   

004074A5    EC              IN AL,DX                                  ; I/O 命令   

004074A6    85C0            TEST EAX,EAX   

004074A8    C645 FC 01      MOV BYTE PTR SS:[EBP-4],1   

004074AC    74 09           JE SHORT GoToMu.004074B7               

004074AE    8BC8            MOV ECX,EAX   

004074B0    E8 CB3B0200     CALL GoToMu.0042B080   

004074B5    EB 02           JMP SHORT GoToMu.004074B9   

004074B7    33C0            XOR EAX,EAX   

004074B9    C645 FC 00      MOV BYTE PTR SS:[EBP-4],0   

004074BD    A3 68B64300     MOV DWORD PTR DS:[43B668],EAX   

004074C2    EB 04           JMP SHORT GoToMu.004074C8   

004074C4    EB 05           JMP SHORT GoToMu.004074CB   

................................................................................   
```

现在的问题是我们如何确定这段解码到什么地方结束。

## 3. 确定解码结束位置

很容易，切换到程序中

```
00407493   /E9 09010000     JMP Crack.004075A1   //错误的跳转。    

004075A1    8B0D 68B64300   MOV ECX,DWORD PTR DS:[43B668]  //到这里。    

004075A7    E8 24000000     CALL Crack.004075D0   

004075AC    EB 04           JMP SHORT Crack.004075B2   

004075AE    EB 05           JMP SHORT Crack.004075B5   

004075B0    99              CDQ   

004075B1    018B 4DF48B45   ADD DWORD PTR DS:[EBX+458BF44D],ECX   

004075B7    F0:5F           LOCK POP EDI                                ; 锁定前缀是不允许的   

004075B9    5E              POP ESI   

004075BA    64:890D 0000000>MOV DWORD PTR FS:[0],ECX   

004075C1    8BE5            MOV ESP,EBP   

004075C3    5D              POP EBP   

004075C4    C3              RETN  //到这里结束。    
```

一般来说，复制到 `RETN` 处的代码位置比较好，不容易漏掉代码。

## 4. 二进制复制代码

好于是我们切换到壳程序里 `Ctrl+G` `00407493`   

选定下面第一部分解压代码。右键-二进制-二进制复制。

```
00407493   . /E9 01000000   JMP GoToMu.00407499   

00407498     |9A            DB 9A   

00407499   > 6A 10          PUSH 10   

0040749B   .  E8 1A3F0200   CALL GoToMu.0042B3BA                          ;  JMP to mfc42.#823   

004074A0   .  83C4 04       ADD ESP,4   

004074A3   .  8945 EC       MOV DWORD PTR SS:[EBP-14],EAX   

004074A6   .  85C0          TEST EAX,EAX   

004074A8   .  C645 FC 01    MOV BYTE PTR SS:[EBP-4],1   

004074AC   .  74 09         JE SHORT GoToMu.004074B7   

004074AE   .  8BC8          MOV ECX,EAX   

004074B0   .  E8 CB3B0200   CALL GoToMu.0042B080   

004074B5   .  EB 02         JMP SHORT GoToMu.004074B9   

004074B7   >  33C0          XOR EAX,EAX   

004074B9   >  C645 FC 00    MOV BYTE PTR SS:[EBP-4],0   

004074BD   .  A3 68B64300   MOV DWORD PTR DS:[43B668],EAX   

004074C2   .  EB 04         JMP SHORT GoToMu.004074C8   

004074C4      EB            DB EB   

004074C5      05            DB 05   

004074C6   .  8901          MOV DWORD PTR DS:[ECX],EAX   

004074C8   >  8B35 A8004300 MOV ESI,DWORD PTR DS:[4300A8]   

004074CE   .  6A 00         PUSH 0   

004074D0   .  6A 20         PUSH 20   

004074D2   .  6A 03         PUSH 3   

004074D4   .  6A 00         PUSH 0   

004074D6   .  6A 01         PUSH 1   

004074D8   .  68 00000080   PUSH 80000000   

004074DD   .  68 60A74300   PUSH GoToMu.0043A760                          ;  ASCII "\".SICE"   

004074E2   .  FFD6          CALL ESI   

004074E4   .  8B3D C8064300 MOV EDI,DWORD PTR DS:[4306C8]   

004074EA   .  83F8 FF       CMP EAX,-1   

004074ED   .  74 08         JE SHORT GoToMu.004074F7   

004074EF   .  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]   

004074F2   .  50            PUSH EAX   

004074F3   .  6A 02         PUSH 2   

004074F5   .  FFD7          CALL EDI   

004074F7   >  70 03         JO SHORT GoToMu.004074FC   

004074F9   .  71 01         JNO SHORT GoToMu.004074FC   

004074FB      01            DB 01   

004074FC   .  6A 00         PUSH 0   

004074FE   .  6A 20         PUSH 20   

00407500   .  6A 03         PUSH 3   

00407502   .  6A 00         PUSH 0   

00407504   .  6A 01         PUSH 1   

00407506   .  68 00000080   PUSH 80000000   

0040750B   .  68 50A74300   PUSH GoToMu.0043A750                          ;  ASCII "\".NTiceD052"   

00407510   .  FFD6          CALL ESI   

00407512   .  83F8 FF       CMP EAX,-1   

00407515   .  74 08         JE SHORT GoToMu.0040751F   

00407517   .  8B4D EC       MOV ECX,DWORD PTR SS:[EBP-14]   

0040751A   .  51            PUSH ECX   

0040751B   .  6A 02         PUSH 2   

0040751D   .  FFD7          CALL EDI   

0040751F   >  75 05         JNZ SHORT GoToMu.00407526   

00407521   .  74 03         JE SHORT GoToMu.00407526   

00407523   .  0001          ADD BYTE PTR DS:[ECX],AL   

00407525      01            DB 01   

00407526   .  6A 00         PUSH 0   

00407528   .  6A 20         PUSH 20   

0040752A   .  6A 03         PUSH 3   

0040752C   .  6A 00         PUSH 0   

0040752E   .  6A 01         PUSH 1   

00407530   .  68 00000080   PUSH 80000000   

00407535   .  68 44A74300   PUSH GoToMu.0043A744                          ;  ASCII "\".NTICE"   

0040753A   .  FFD6          CALL ESI   

0040753C   .  83F8 FF       CMP EAX,-1   

0040753F   .  74 08         JE SHORT GoToMu.00407549   

00407541   .  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]   

00407544   .  52            PUSH EDX   

00407545   .  6A 02         PUSH 2   

00407547   .  FFD7          CALL EDI   

00407549   >  7A 05         JPE SHORT GoToMu.00407550   

0040754B   .  7B 03         JPO SHORT GoToMu.00407550   

0040754D   .  0001          ADD BYTE PTR DS:[ECX],AL   

0040754F      01            DB 01   

00407550   .  6A 00         PUSH 0   

00407552   .  6A 20         PUSH 20   

00407554   .  6A 03         PUSH 3   

00407556   .  6A 00         PUSH 0   

00407558   .  6A 01         PUSH 1   

0040755A   .  68 00000080   PUSH 80000000   

0040755F   .  68 38A74300   PUSH GoToMu.0043A738                          ;  ASCII "\".SIWVID"   

00407564   .  FFD6          CALL ESI   

00407566   .  83F8 FF       CMP EAX,-1   

00407569   .  74 08         JE SHORT GoToMu.00407573   

0040756B   .  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]   

0040756E   .  50            PUSH EAX   

0040756F   .  6A 02         PUSH 2   

00407571   .  FFD7          CALL EDI   

00407573   >  75 03         JNZ SHORT GoToMu.00407578   

00407575   .  74 01         JE SHORT GoToMu.00407578   

00407577      00            DB 00   

00407578   .  6A 00         PUSH 0   

0040757A   .  6A 20         PUSH 20   

0040757C   .  6A 03         PUSH 3   

0040757E   .  6A 00         PUSH 0   

00407580   .  6A 01         PUSH 1   

00407582   .  68 00000080   PUSH 80000000   

00407587   .  68 28A74300   PUSH GoToMu.0043A728                          ;  ASCII "\".SIWDEBUG"   

0040758C   .  FFD6          CALL ESI   

0040758E   .  83F8 FF       CMP EAX,-1   

00407591   .  74 08         JE SHORT GoToMu.0040759B   

00407593   .  8B4D EC       MOV ECX,DWORD PTR SS:[EBP-14]   

00407596   .  51            PUSH ECX   

00407597   .  6A 02         PUSH 2   

00407599   .  FFD7          CALL EDI   
```



```
0040759B   >  EB 04          JMP SHORT GoToMu.004075A1   

0040759D      7B           DB 7B                               ;  CHAR '{'   

0040759E      D7           DB D7   

0040759F      25           DB 25                               ;  CHAR '%'   

004075A0   .  92           XCHG EAX,EDX   

004075A1   >  8B0D 68B64300 MOV ECX,DWORD PTR DS:\[43B668\]   

004075A7   .  E8 24000000  CALL GoToMu.004075D0   

004075AC   .  EB 04        JMP SHORT GoToMu.004075B2   

004075AE   .  EB 05        JMP SHORT GoToMu.004075B5   

004075B0      99           DB 99   

004075B1      01           DB 01   

004075B2   >  8B4D F4      MOV ECX,DWORD PTR SS:\[EBP\-C\]   

004075B5   >  8B45 F0      MOV EAX,DWORD PTR SS:\[EBP\-10\]   

004075B8   .  5F           POP EDI   

004075B9   .  5E           POP ESI   

004075BA   .  64:890D 00000>MOV DWORD PTR FS:\[0\],ECX   

004075C1   .  8BE5         MOV ESP,EBP   

004075C3   .  5D           POP EBP   

004075C4   .  C3           RETN   

.................................................................................. 
```

可恶，这些东西都是检验调试器用的代码。

又切换到脱壳程序中，`Ctrl + G 00407493`

选定从`00407493`到`004075C4`处的全部代码，右键－二进制－二进制粘贴。

检测是否添加了许多红色代码，确认操作成功，复制所有修改到程序中，保存覆盖脱壳文件，关闭`OD`,重新载入,修改 2 完成。

这时`OD`载入程序运行提示无法处理异常，这个是修改 2 完成的标志，继续查找暗桩。

```
0042BB77  |.  FF15 84004300 CALL DWORD PTR DS:\[<&kernel32.GetModuleH>; GetModuleHandleA //F8 一直
```

顺利执行到这里，慢慢步过。

```
0042BB7D  |.  50            PUSH EAX   
0042BB7E  |.  E8 7D000000   CALL Crack.0042BC00  //出错,F7 跟进去。
```

这里的第二处要`F7`慢慢过，对比脱壳程序同步比较。

```
6BC4B4F0     FF50 58         CALL DWORD PTR DS:\[EAX+58\] //当同步跟踪到这里，脱壳程序崩溃，加壳程序运行了，Ctrl+F2 重启加脱壳程序，这里经常要重启动 OD。
```

脱壳程序`CALL 0042BC00`

```
00407740     55              PUSH EBP   
00407741     8BEC            MOV EBP,ESP   
00407743     64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   
00407749     6A FF           PUSH -1   
0040774B     68 62CA4200     PUSH Crack.0042CA62   
00407750     50              PUSH EAX   
00407751     B8 14120000     MOV EAX,1214   
00407756     64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   
0040775D     E8 BE420200     CALL Crack.0042BA20   
00407762     53              PUSH EBX   
00407763     56              PUSH ESI   
00407764     57              PUSH EDI   
00407765     8BF9            MOV EDI,ECX   
00407767     E9 CD700000     JMP Crack.0040E839   
0040776C     FA              CLI   
0040776D     EA D0AF60CD 4DA>JMP FAR AD4D:CD60AFD0               ; 远距跳转   
00407774     8957 AD         MOV DWORD PTR DS:\[EDI\-53\],EDX   
00407777     C54F 94         LDS ECX,FWORD PTR DS:\[EDI\-6C\]        ; 修正的段位寄存器   
0040777A     EE              OUT DX,AL                          ; I/O 命令   
0040777B     EC              IN AL,DX                           ; I/O 命令   
0040777C     67:AC           LODS BYTE PTR DS:\[SI\]   
0040777E     62DE            BOUND EBX,ESI                      ; 非法使用寄存器   

.............................................................
```

原程序代码

```
00407740     55              PUSH EBP   
00407741     8BEC            MOV EBP,ESP   
00407743     64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   
00407749     6A FF           PUSH -1   
0040774B     68 62CA4200     PUSH GoToMu.0042CA62   
00407750     50              PUSH EAX   
00407751     B8 14120000     MOV EAX,1214   
00407756     64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   
0040775D     E8 BE420200     CALL GoToMu.0042BA20   
00407762     53              PUSH EBX   
00407763     56              PUSH ESI   
00407764     57              PUSH EDI   
00407765     8BF9            MOV EDI,ECX   
00407767     E9 01000000     JMP GoToMu.0040776D   
0040776C     24 33           AND AL,33   
0040776E     F656 E8         NOT BYTE PTR DS:\[ESI\-18\]   
00407771     D7              XLAT BYTE PTR DS:\[EBX+AL\]   
00407772     3D 020085C0     CMP EAX,C0850002
```

为了确定解压代码的尾部地址，不断`Ctrl+F9`返回，直到

```
0041442B    FF15 58064300  CALL DWORD PTR DS:\[<&olepro32.OleLoadPic>; olepro32.OleLoadPicture   
00414431    3D 02400080    CMP EAX,80004002  //这里为止。
```

同理，我们选定这段代码，右键-二进制-二进制复制

替换脱壳程序 右键-二进制-二进制粘贴

从`00407740`到`00414431`之间的代码

代码数量相当多，一定不要遗漏了代码，耐心操作，最后一处暗桩，复制所有修改到程序中，覆盖脱壳的程序，正常运行。

【脱壳总结】 这个程序一个壳中指针，两处解码验证壳是否存在,熟练的`Od`操作能让你 15 分钟内脱掉这个壳。

-------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
