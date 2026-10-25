
# ACProtect 1.22b 脱壳手记

> 原文：[`www.pediy.com/kssd/pediy06/pediy6479.htm`](https://www.pediy.com/kssd/pediy06/pediy6479.htm)

目标：`NOTEPAD.EXE`，自己用 `ACProtect` `1.22b` 专业版加壳，`Compress Opetions` 里选 `Opetion1`，`Advanced Opetions` 的六个选项全选；加壳前 `49.7 KB`，加壳后 `143 KB`

工具：`OllyDbg Beta110b` 原版; `ImportREC 1.6 FINAL`

破解平台：`Win2000SP4 Chinese`

主要目的：跳过 `IAT` 加密，快速到达伪 `OEP`，`Replace Code` 的一些处理;

由于 `ASProtect` 的 `Stolen code` 已经被各位老大们搞的很透彻了，`ACProtect` 的 `Stolen code` 的分析和修复没多大差别，所以我就不在罗嗦了(省的敲不少字呢，呵呵:))

废话少说，开工

———————————————————————————————————

用 `OllyDbg` 加载目标后，忽略所有异常，用插件 `IsDebuggerPresent` 隐藏调试器标志，程序停在这:

```
01010000  >  60                 PUSHAD
01010001    66:8BCA             MOV CX,DX
01010004    87C8               XCHG EAX,ECX
01010006    41                 INC ECX
01010007    72 01             JB SHORT NOTEPAD\_.0101000A
01010009    43                 INC EBX
0101000A    48                 DEC EAX
0101000B    FC                 CLD
0101000C    50                 PUSH EAX
0101000D    E8 01000000       CALL NOTEPAD\_.01010013
01010012    7A 83             JPE SHORT NOTEPAD\_.0100FF97
01010014    C40458            LES EAX,FWORD PTR DS:[EAX+EBX*2]
01010017    66:8BEE           MOV BP,SI
0101001A    E8 01000000       CALL NOTEPAD\_.01010020

.                ;从开头到 01010BB6 都是壳的分段解密，可以看准

.            ;每段循环结束位置 0F85 xxFFFFFF，用 F4 直接跳到下一循环

.

.

.

01010BB6    4F               DEC EDI
01010BB7    81F3 0503C769   XOR EBX,69C70305
01010BBD    68 4F460000     PUSH 464F
01010BC2    4E               DEC ESI
01010BC3    59               POP ECX      ;ECX=464Fh,这段循环解密的 dword 数
01010BC4    8B28             MOV EBP,DWORD PTR DS:[EAX]  ;循环开始,这是一个大循环,解密了从 01010BEA
01010BC6    03EB             ADD EBP,EBX      ;开始的(464Fh-1)*4=11938h 字节
01010BC8    C1C5 12          ROL EBP,12
01010BCB    83C0 04          ADD EAX,4
01010BCE    0328             ADD EBP,DWORD PTR DS:[EAX]
01010BD0    83C0 FC          ADD EAX,-4
01010BD3    8928             MOV DWORD PTR DS:[EAX],EBP
01010BD5    81F3 B2A9C00F   XOR EBX,0FC0A9B2
01010BDB    81C0 04000000    ADD EAX,4
01010BE1    83E9 01          SUB ECX,1
01010BE4   ^ 0F85 DAFFFFFF    JNZ NOTEPAD\_.01010BC4
01010BEA    116C95 AB        ADC DWORD PTR SS:[EBP+EDX*4-55],EBP   ;在这下硬件执行断点
01010BEE    53               PUSH EBX
01010BEF   ^ 79 91           JNS SHORT NOTEPAD\_.01010B82
```

上面的循环结束后 `01010BEA` 处变为

```
01010BEA   /E9 A9180100     JMP NOTEPAD\_.01022498  ;这个 JMP 远距离跳到壳真正开始工作的地方
01010BEF   |0000             ADD BYTE PTR DS:[EAX],AL
01010BF1   |0000             ADD BYTE PTR DS:[EAX],AL
01010BF3   |0000             ADD BYTE PTR DS:[EAX],AL
01010BF5   |0000             ADD BYTE PTR DS:[EAX],AL
01010BF7   |0000             ADD BYTE PTR DS:[EAX],AL
```

在 `01010BEA` 上点右键,`Folllow Dump->Selection`,在内存窗口里搜索 `HEX:4D4D4D4D`,找到如下:

```
01010F19  4D 4D 4D 4D C5 73 FE FF D2 73 FE FF 00 00 00 00  MMMM 舠?襰?....
01010F29  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01010F39  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01010F49  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01010F59  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

`01010F19` 就是 `ACProtect` 在后面开始处理 `Stolen code` 时放置 `0006FFF0`(即 `OD` 刚加载程序时的 `EBP`,一般程序多数是 `0012FFF0`,`NOTEPAD.EXE` 有点特殊),在 `01010F19` 下硬件访问断点,目的是快速到达处理 `Stolen code` 的地方

———————————————————————————————————

### 2.跳过 IAT 加密

在 `GetProcAddress` 入口处下硬件断点,也可以在 `GetProcAddress` 入口处跳过一字节后用 `F2` 下普通断点.`F9` 运行,会在 `GetProcAddress` 上断下,堆栈显示的是 `GlobalAlloc`,这是壳自己用到的 `API`,一共 23 个,依次是

`GlobalAlloc`.`GlobalFree`.`GetCurrentProcessId`.`CreateToolhelp32Snapshot`.`Process32First`.`Process32Next`.`CloseHandle`.`CreateFileA`.`TerminateProcess`.`IsDebuggerPresent`.`OpenProcess`.`ReadFile`.`WriteFile`.`FreeLibrary`.`GetTempPathA`.`UnhandledExceptionFilter`.`GetThreadContext`.`SetThreadContext`.`GetCurrentThread`.`EnumWindows`.`GetWindowTextA`.`GetClassNameA`.`PostMessageA`

一直按 `F9` 运行,在处理过 `PostMessageA` 后,就开始处理程序的 `API`,堆栈和寄存器都会有个较大的变化,断下后,是处 `IsProcessorFeaturePresent`,`Return to msvcrt.78001EAA`,继续 `F9`,直到堆栈第一次显示 `Return to NOTEPAD.XXXXX`,这时处理的是 `FindTextW`,取消 `GetProcAddress` 的硬件执行断点,`Ctrl+F9` 运行返回到壳里

`PS`:如果被加壳的程序是用 `Delphi` 或 `BCB` 写的话,在 `PostMessageA` 后处理的下一个 `API`,堆栈显示就是返回到壳程序,但 `VC++`的程序在处理完 `PostMessageA` 后还会由 `Kenrel` 调用 `GetProcAddress` 处理一些 `msvcrt`,`COMCTL32` 等里的一些 `API`,所以必须等处理完这些 `API` 后,在壳调用 `GetProcAddress` 处理程序原来的 `API` 时才能 `Ctrl+F9` 返回到壳里

`Ctrl+F9` 运行返回后:

```
0102201B    53                PUSH EBX
0102201C    FFB5 29DE4000     PUSH DWORD PTR SS:[EBP+40DE29]    
01022022    FF95 80B04100     CALL DWORD PTR SS:[EBP+41B080]    ;Call GetProcAddress
01022028    3B9D 31DE4000     CMP EBX,DWORD PTR SS:[EBP+40DE31];Ctrl+F9 返回到这里,EBP=00C0F000
0102202E    7C 0F             JL SHORT NOTEPAD\_.0102203F         ;所以 GetProcAddress 的地址是
01022030    90                NOP             ;EBP+41B080=102A080,记下这个地址! 
01022031    90                NOP
01022032    90                NOP
01022033    90                NOP
01022034    60                PUSHAD
01022035    2BC0              SUB EAX,EAX
01022037    8803              MOV BYTE PTR DS:[EBX],AL
01022039    43                INC EBX
0102203A    3803              CMP BYTE PTR DS:[EBX],AL
0102203C   ^ 75 F9            JNZ SHORT NOTEPAD\_.01022037
0102203E    61                POPAD
0102203F    0BC0              OR EAX,EAX
01022041   ^ 0F84 21FFFFFF     JE NOTEPAD\_.01021F68    ;EAX=刚才 GetProcAddress 得到的 API 的入口
01022047    3B85 90B04100     CMP EAX,DWORD PTR SS:[EBP+41B090];EBP+41B090 指向 MessageBox
0102204D    75 0A             JNZ SHORT NOTEPAD\_.01022059   ;① 这里把得到的 API 入口与 MessageBox
0102204F    90                NOP                  ;比较,相等就用壳的 MessageBox 来代替
01022050    90                NOP                  ;程序的 MessageBox,把①处的 JNZ 改为 JMP
01022051    90                NOP                  ;即可跳过 ACProtect 对 MessageBox 作的手脚
01022052    90                NOP
01022053    8D85 D4E34000     LEA EAX,DWORD PTR SS:[EBP+40E3D4]
01022059    56                PUSH ESI
0102205A    FFB5 29DE4000     PUSH DWORD PTR SS:[EBP+40DE29]
01022060    5E                POP ESI
01022061    39B5 FA234000     CMP DWORD PTR SS:[EBP+4023FA],ESI
01022067    74 15             JE SHORT NOTEPAD\_.0102207E
01022069    90                NOP
0102206A    90                NOP
0102206B    90                NOP
0102206C    90                NOP
0102206D    39B5 FE234000     CMP DWORD PTR SS:[EBP+4023FE],ESI
01022073    74 09             JE SHORT NOTEPAD\_.0102207E
01022075    90                NOP
01022076    90                NOP
01022077    90                NOP
01022078    90                NOP
01022079    EB 63             JMP SHORT NOTEPAD\_.010220DE
0102207B    90                NOP
0102207C    90                NOP
0102207D    90                NOP
0102207E    80BD 90304100 00  CMP BYTE PTR SS:[EBP+413090],0
01022085    74 57             JE SHORT NOTEPAD\_.010220DE
01022087    90                NOP
01022088    90                NOP
01022089    90                NOP
0102208A    90                NOP
0102208B    EB 07             JMP SHORT NOTEPAD\_.01022094
0102208D    90                NOP
0102208E    90                NOP
0102208F    90                NOP
01022090    0100              ADD DWORD PTR DS:[EAX],EAX
01022092    0000              ADD BYTE PTR DS:[EAX],AL
01022094    8BB5 F6DE4000     MOV ESI,DWORD PTR SS:[EBP+40DEF6]       ;EBP+40DEF6 指向 Acpr 要转移 API 的地方
0102209A    83C6 0D            ADD ESI,0D
0102209D    81EE EA1B4000      SUB ESI,401BEA
010220A3    2BF5              SUB ESI,EBP
010220A5    83FE 00           CMP ESI,0
010220A8    7F 34             JG SHORT NOTEPAD\_.010220DE  ;② 不跳就把 API 转移到[EBP+40DEF6]
```



`010220AA` `90` `NOP` ;所以把②处的 `JG` 改为 `JMP` 即可跳过 `Acpr`   

`010220AB` `90` `NOP` ;对 `IAT` 的加密!   

`010220AC` `90` `NOP`   

`010220AD` `90` `NOP`   

`010220AE` `8BB5 F6DE4000` `MOV ESI,DWORD PTR SS:\[EBP+40DEF6\]`   

`010220B4` `53` `PUSH EBX`   

`010220B5` `50` `PUSH EAX`   

`010220B6` `E8 CABCFFFF` `CALL NOTEPAD\_.0101DD85`   

`010220BB` `8BD8` `MOV EBX,EAX`   

`010220BD` `58` `POP EAX`   

`010220BE` `33C3` `XOR EAX,EBX`   

`010220C0` `C606 68` `MOV BYTE PTR DS:\[ESI\],68`   

`010220C3` `8946 01` `MOV DWORD PTR DS:\[ESI+1\],EAX`   

`010220C6` `C746 05 81342400` `MOV DWORD PTR DS:\[ESI+5\],243481`   

`010220CD` `895E 08` `MOV DWORD PTR DS:\[ESI+8\],EBX`   

`010220D0` `C646 0C C3` `MOV BYTE PTR DS:\[ESI+C\],0C3`   

`010220D4` `5B` `POP EBX`   

`010220D5` `8BC6` `MOV EAX,ESI`   

`010220D7` `8385 F6DE4000 0D` `ADD DWORD PTR SS:\[EBP+40DEF6\],0D`   

`010220DE` `5E` `POP ESI`   

`010220DF` `8907` `MOV DWORD PTR DS:\[EDI\],EAX` ;③ 这里把 `API` 写入正确的 `IAT` 里   

`010220E1` `8385 2DDE4000 04` `ADD DWORD PTR SS:\[EBP+40DE2D\],4` ;`d EDI` 就可以看到程序原本的   

`010220E8` `^ E9 C6FEFFFF` `JMP NOTEPAD\_.01021FB3` ;`IAT` 跳转表   

`010220ED` `83C6 14` `ADD ESI,14`   

`010220F0` `8B95 31DE4000` `MOV EDX,DWORD PTR SS:\[EBP+40DE31\]`   

`010220F6` `^ E9 28FEFFFF` `JMP NOTEPAD\_.01021F23`   

`010220FB` `60` `PUSHAD` ;④ 上面的①②改掉后,用 `F4` 直接运行到这里   

`010220FC` `E8 00000000` `CALL NOTEPAD\_.01022101` ;也可下硬件断点,`F9` 运行,断下后   

`01022101` `5E` `POP ESI` ;再把①②处改会原来的代码.   

`01022102` `83EE 06` `SUB ESI,6` ;一定要改回去,不然壳会在最后   

`01022105` `B9 09020000` `MOV ECX,209` ;效验并解密伪 `OEP` 处代码时出错!   

`0102210A` `29CE` `SUB ESI,ECX`   

`0102210C` `BA BAA8FF62` `MOV EDX,62FFA8BA`   

`01022111` `C1E9 02` `SHR ECX,2`   

`01022114` `83E9 02` `SUB ECX,2`   

`01022117` `83F9 00` `CMP ECX,0`   

`0102211A` `7C 1A` `JL SHORT NOTEPAD\_.01022136`   

`0102211C` `8B048E` `MOV EAX,DWORD PTR DS:\[ESI+ECX\*4\]`   

`0102211F` `8B5C8E 04` `MOV EBX,DWORD PTR DS:\[ESI+ECX\*4+4\]`   

`01022123` `2BC3` `SUB EAX,EBX`   

`01022125` `C1C8 1F` `ROR EAX,1F`   

`01022128` `33C2` `XOR EAX,EDX`   

`0102212A` `81C2 D82567EC` `ADD EDX,EC6725D8`   

`01022130` `89048E` `MOV DWORD PTR DS:\[ESI+ECX\*4\],EAX`   

`01022133` `49` `DEC ECX`   

`01022134` `^ EB E1` `JMP SHORT NOTEPAD\_.01022117` ;这段循环是把 `010220F6` 前的代码   

`01022136` `61` `POPAD` ;恢复加密,这样的循环在 `Acpr` 里   

`01022137` `61` `POPAD` ;随处可见,每段解密后的执行代码后   

`01022138` `C3` `RETN` ;都会有这样的恢复加密的循环   

`PS:` 在③处 `d edi` 后,可以在内存窗口里随便找一个字节,把它随便改成别的值,然后再改会原来的值,这样 `OD` 就会把之后写入该内存页所有的值用红色高亮显示,在④处断下后,在内存窗口里找到最前面的用红色显示的值,它的地址就是 `IAT` 了!我这里看到的是 `01001000`,用 `ImportREC` 得到输入表时 `RAV` 里填的就是 `01001000-01000000=1000`   

———————————————————————————————————   

### 3.快速到达处理 `Stolen code` 的地方   

我们已经在 `01010F19` 下了硬件访问断点,在做完上面的处理后,按 `F9` 运行,就会直接断在开始处理 `Stolen code` 的地方,如下   

```
        .
        .
        .
        .
010236D1    61               POPAD
010236D2    56               PUSH ESI
010236D3    BE E50E0101      MOV ESI,NOTEPAD\_.01010EE5
010236D8    890E             MOV DWORD PTR DS:\[ESI\],ECX
010236DA    5E               POP ESI
010236DB    FF35 E50E0101    PUSH DWORD PTR DS:\[1010EE5\]
010236E1    893C24           MOV DWORD PTR SS:\[ESP\],EDI
010236E4    57               PUSH EDI
010236E5    BF 190F0101      MOV EDI,NOTEPAD\_.01010F19
010236EA    893D E10E0101    MOV DWORD PTR DS:\[1010EE1\],EDI
010236F0    5F               POP EDI
010236F1    FF35 E10E0101    PUSH DWORD PTR DS:\[1010EE1\]            ; NOTEPAD\_.01010F19
010236F7    8B3C24           MOV EDI,DWORD PTR SS:\[ESP\]
010236FA    8F05 DD0E0101    POP DWORD PTR DS:\[1010EDD\]            ; NOTEPAD\_.01010F19
01023700    892F             MOV DWORD PTR DS:\[EDI\],EBP    ;EDI\=01010F19
01023702    8B3C24           MOV EDI,DWORD PTR SS:\[ESP\]    ;断在这儿   
01023705    8F05 F90E0101    POP DWORD PTR DS:\[1010EF9\]
0102370B    FF35 190F0101    PUSH DWORD PTR DS:\[1010F19\]
01023711    8925 150F0101    MOV DWORD PTR DS:\[1010F15\],ESP
01023717    893D BD0E0101    MOV DWORD PTR DS:\[1010EBD\],EDI
0102371D    90               NOP
0102371E    90               NOP
0102371F    60               PUSHAD
01023720    50               PUSH EAX
01023721    E8 01000000      CALL NOTEPAD\_.01023727
        .
        .
        .
        .
```   

从这段代码起 `Acpr` 开始处理 `Stolen code`,具体的 `Stolen code` 我不是很在行,几位老大已经分析的清晰透彻了.   

`Acpr` 对 `Stolen code` 处理的代码是分段解密后再执行的,因为我们前面在这个内存页里改过数据,所以所有重新写入的代码都是用红色高亮显示了,这就给我们用 `F4` 快速执行过解密代码带来了方便,具体方法是在最后一句高亮显示的代码开始向上找 `0F85 xxFFFFFF`(`JNZ xxxxxxxx`)远程跳转语句,然后 `F4` 到它下面的一句上,有的 `0F85 xxFFFFFF` 的语句会因为它前后的代码环境而不被 `OD` 显示出来,这是可以 `Crtl+G` 到 `0F85 xxFFFFFF` 的地址,也可以找到高亮显示的最后一个 `E8 010000` 或 `EB 01`,然后 `F4` 到其上,再 `F7` 几步就可以看到 `0F85 xxFFFFFF`(`JNZ xxxxxxxx`)了   

处理 `Stolen code` 的前几段代码都是对堆栈的处理,要是不分析 `Stolen code` 的话可以不用管,但是后面几段因为会包含到程序 `OEP` 的 `Call`,如果执行这些 `Call` 的话,就有可能因为执行了 `Replace Code` 而改变程序的数据结构.   

我的处理方法是把这些 `Call` 先 `nop` 掉,在执行完 `nop` 后,在把 `Call` 的代码恢复回去,例如:下面代码执行到①处 `CALL DWORD PTR DS:\[100115C\]` 时把它先用 `nop` 填充掉,等走完 `nop` 后再把 `CALL DWORD PTR DS:\[100115C\]` 写回去,但②③会用到 `EAX` 指针,我们跳过了 `Call`,所以到②和③时会出错,于是一不做二不休,把②③的代码也按刚才的方法干掉 :)   

```
01028D7E   /E9 04000000     JMP NOTEPAD\_.01028D87
01028D83   |66:BD 27CC      MOV BP,0CC27
01028D87   61               POPAD
01028D88    FF15 5C110001   CALL DWORD PTR DS:\[100115C\]            ; msvcrt.\_\_p\_\_fmode ①
01028D8E    891D 110F0101   MOV DWORD PTR DS:\[1010F11\],EBX        
01028D94    FF35 110F0101   PUSH DWORD PTR DS:\[1010F11\]
01028D9A    C705 0D0F0101 4>MOV DWORD PTR DS:\[1010F0D\],NOTEPAD\_.010088>
01028DA4    8B1D 0D0F0101   MOV EBX,DWORD PTR DS:\[1010F0D\]         ; NOTEPAD\_.010065D0
01028DAA    8B0B            MOV ECX,DWORD PTR DS:\[EBX\]
01028DAC    8B1C24          MOV EBX,DWORD PTR SS:\[ESP\]
01028DAF    8F05 090F0101   POP DWORD PTR DS:\[1010F09\]
01028DB5    8908            MOV DWORD PTR DS:\[EAX\],ECX    ;②
01028DB7    FF15 4C110001   CALL DWORD PTR DS:\[100114C\]            ; msvcrt.\_\_p\_\_commode
01028DBD    53              PUSH EBX
01028DBE    893424          MOV DWORD PTR SS:\[ESP\],ESI
01028DC1    57              PUSH EDI
01028DC2    BF 40880001     MOV EDI,NOTEPAD\_.01008840
01028DC7    8BF7            MOV ESI,EDI
01028DC9    5F              POP EDI
01028DCA    8B16            MOV EDX,DWORD PTR DS:\[ESI\]
01028DCC    8B3424          MOV ESI,DWORD PTR SS:\[ESP\]
01028DCF    8F05 050F0101   POP DWORD PTR DS:\[1010F05\]
01028DD5    8910            MOV DWORD PTR DS:\[EAX\],EDX    ;③
01028DD7    90              NOP
01028DD8    60              PUSHAD
01028DD9    E8 01000000     CALL NOTEPAD\_.01028DDF
```   

`PS:` 每段处理 `Stolen code` 的代码前总要走好几段解密代码,鼠标滚轮费的紧啊,偶的食指也隐隐作痛了 :) 越往后走 `F4` 就越要甚用,不然一不小心就会跟飞,不放心的话最好用 `F7` 步入,最后来到下面:   

```
01029A80    E8 6245FFFF      CALL NOTEPAD\_.0101DFE7    ;EBP\=00C0F000
01029A85    8DB5 80B04100    LEA ESI,DWORD PTR SS:\[EBP+41B080\]  ;EBP+41B080=102A080,就是前面我们
01029A8B    8DBD 1EE34000    LEA EDI,DWORD PTR SS:\[EBP+40E31E\]  ;记下的 GetProcAddress 的地址!
01029A91    B9 05000000      MOV ECX,5
01029A96    F3:A5            REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]
01029A98    40               INC EAX        ;这里把包含 GetProcAddress 在内
01029A99    EB 01            JMP SHORT NOTEPAD\_.01029A9C  ;的 5 个 API 地址 Copy 到\[101D31E\]
```   

这 5 个 `API` 是壳的 `IAT` 里的,依次是 `GetProcAddress`、`GetModuleHandleA`、`LoadLibraryA`、`ExitProcess`、`MessageBoxA`



这段之后再经过一段解密代码就到了壳解密伪 OEP 处代码并跳向伪 OEP 的地方,所以如果要快速到达伪 OEP,就可以在前面我们得到`GetProcAddress`的地址时,在它上面下硬件访问断点!

经过一段解密代码后来到这里:

```
01029C3E    66:C1DF 4D       RCR DI,4D

01029C42    E8 A043FFFF      CALL NOTEPAD\_.0101DFE7

01029C47    8B85 E4AC4100    MOV EAX,DWORD PTR SS:[EBP+41ACE4]

01029C4D    0385 31DE4000    ADD EAX,DWORD PTR SS:[EBP+40DE31]

01029C53    8985 E4AC4100    MOV DWORD PTR SS:[EBP+41ACE4],EAX

01029C59    E8 8943FFFF      CALL NOTEPAD\_.0101DFE7

01029C5E    C685 A1AC4100 E8 MOV BYTE PTR SS:[EBP+41ACA1],0E8

01029C65    E8 7D43FFFF      CALL NOTEPAD\_.0101DFE7

01029C6A    C785 A2AC4100 FF>MOV DWORD PTR SS:[EBP+41ACA2],25FF

01029C74    8D85 E4AC4100    LEA EAX,DWORD PTR SS:[EBP+41ACE4]

01029C7A    8985 A4AC4100    MOV DWORD PTR SS:[EBP+41ACA4],EAX

01029C80    E8 6243FFFF      CALL NOTEPAD\_.0101DFE7

01029C85    8DBD 80AA4100    LEA EDI,DWORD PTR SS:[EBP+41AA80]

01029C8B    8D8D 96AC4100    LEA ECX,DWORD PTR SS:[EBP+41AC96]

01029C91    2BCF             SUB ECX,EDI

01029C93    C1E9 02          SHR ECX,2

01029C96    E8 EA40FFFF      CALL NOTEPAD\_.0101DD85    ;这个循环解密出伪 OEP 处的代码

01029C9B    AB               STOS DWORD PTR ES:[EDI]    ;|如果前面改掉的代码不及时恢复的

01029C9C   ^ E2 F8          LOOPD SHORT NOTEPAD\_.01029C96  ;/话,这里就会解密错误

01029C9E    61              POPAD

01029C9F    EB 01           JMP SHORT NOTEPAD\_.01029CA2

01029CA2   - FF25 E49C0201  JMP DWORD PTR DS:[1029CE4]  ; NOTEPAD\_.0100649F 飞向伪 OEP!
```

到达伪 OEP 后就可以`dump`出程序,运行`ImportREC`,`OEP`填入伪`OEP=0000649F`,`RAV`填入前面得到的`00001000`,`Size`填入`1000`,`Get Import`,即可得到完整的 API,把 Invalid 的`FThunk`全部`Delete`掉,按默认选项,选中`Add new section`,`Fix Dump`,`OK`! :)

PS:有的程序在填入伪 OEP 后,`IAT AutoSearch`会可能得不到 IAT 的`RAV`

可惜的是`NOTEPAD.EXE`本身太小,`Acpr`居然没有`Replace Code`,我是一边脱壳一边写的,程序`Dump`出后,才发现没有`Replace Code`,实在可惜.其实,在伪 OEP `Dump`出的程序已经包含了解密`Replace Code`的代码和数据表,修复 IAT,`Stolen code`和 OEP 后程序就可以正常运行.当然,要把脱壳后的程序修复的接近完美的话,也可以自己手工修复所有的`Replace Code` :)

PS:`Acpr`的`Stolen code`有点儿局限,有些由`Delphi`和`BCB`写的程序的原 OEP 处是`EB 10` `JMP xxxxxxxx`的跳转语句,例如`OllyDbg`,这时`Acpr`就不会`Stolen code`.我本来是想把`OllyDbgBeta110b`的主程序加壳当试练品,可`Acpr`没`Stolen code`,只好换`NOTEPAD.EXE`了 :P

### 后记

上次写破文是在 5 个月前,那时就怀疑自己的表达是否清楚合理,现在就更没把握了,还请各位看了不要笑话.国内软件用`Acpr`加壳的不多,不知道是因为没有破版放出,还是对`Acpr`没有信心,关于`Acpr`的破文就更少了.与之成鲜明对比的是`Aspr`,`ASProtect 1.23RC4`和`1.30`也不见有破版广泛流传(我自己就没有`1.23 RC4`的可用版,`1.30`见都没见过 555~),但用其加壳的程序比比皆是(1.30 我不敢说,但加`1.23 RC4`的确实不少),破文更是扑天盖地.现在`Acpr`的破版在一月之内接连放出,不知道局面是否会有所改变

thl0057

04.04.01

于 HD

`4dd466d0e1e0cc123afbe504da2462ca`
