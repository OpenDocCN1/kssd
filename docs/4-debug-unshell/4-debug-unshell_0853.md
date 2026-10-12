
# 脱 PECompact 2.5 Retail 加的 DLL 壳

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-478.htm`](https://www.pediy.com/kssd/pediy09/pediy09-478.htm)

> 【文章标题】: 脱 `PECompact` 2.5 Retail 加的 DLL 壳

> 【文章作者】: `hbqjxhw`

> 【软件名称】: `qtintf.dll`

> 【软件大小】: 1.13M

> 【下载地址】: [http://free5.ys168.com/?hbqjxhw](http://free5.ys168.com/?hbqjxhw)

> 【加壳方式】: `PECompact` 2.5 Retail -> `Jeremy Collake`

> 【保护方式】: 加壳

> 【编写语言】: `Borland C++ DLL Method 2`

> 【使用工具】: `OllyDBG`、`WinHex`、`PEiD`、`LordPE`、`PEditor`、`ImportREC`、`PEComAngela`、`Dll_LoadEx`

> 【操作平台】: `WinXP SP2`

> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

【详细过程】

根据好多人求助没有 `PECompact` 2.0 以上加的 DLL 的文章，写下此篇以解燃眉之急。

又深入地学习了 FLY 大侠写的《用 `Ollydbg` 手脱 `PECompact` 加壳的 DLL 》之后，才有下面的文章。

根据 FLY 大侠说的“很多兄弟都不喜欢脱 DLL 的壳，我也是这样。DLL 比 `EXE` 多了个重定位表需要修复，况且单独一个 DLL 连脱壳后的测试都难以彻底进行。”

下面就重点讲一讲这壳在什么地方处理重定位表吧，其它的都非常简单了。

少说废话了，下面就开始

### 第一步：`DUMP`

```
00E037A8   B8 F0051901    MOV EAX,qtintf.011905F0   ; 进入 OD 后停在这（与 2.0 以下的版本不同了）
00E037AD   50             PUSH EAX
00E037AE   64:FF35 0000000>PUSH DWORD PTR FS:[0]
00E037B5   64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00E037BC   33C0          XOR EAX,EAX-----------------------》4 下之后到这里。
00E037BE   8908          MOV DWORD PTR DS:[EAX],ECX
00E037C0   50            PUSH EAX
00E037C1   45            INC EBP
00E037C2   43            INC EBX
```

按 `F8`，4 下之后看堆栈如下：

```
0006F8A0   0006F9BC   指向下一个 SEH 记录的指针
0006F8A4   011905F0   SE 处理器
0006F8A8   7C9211A7   返回到 ntdll.7C9211A7
0006F8AC   00D90000   ASCII "MZP"
0006F8B0   00000001
0006F8B4   00000000
0006F8B8   00E037A8   qtintf.<模块入口点>
```

其实这两处的 `011905F0` 我们要找的值

`00E037A8 > B8 F0051901 MOV EAX,qtintf.011905F0------》`这一处的

```
0006F8A4   011905F0   SE 处理器-----》还有这里的
```

`Ctrl+G` 输入 `011905F0`，来到了如下

```
011905F0   B8 75F318F1     MOV EAX,F118F375                 ; 来到了这里，在此处下 F2 断点。F9 运行到此处，去掉 F2 断点。
011905F5   8D88 9E120010   LEA ECX,DWORD PTR DS:[EAX+100012>
011905FB   8941 01         MOV DWORD PTR DS:[ECX+1],EAX
011905FE   8B5424 04       MOV EDX,DWORD PTR SS:[ESP+4]
01190602   8B52 0C         MOV EDX,DWORD PTR DS:[EDX+C]
01190605   C602 E9         MOV BYTE PTR DS:[EDX],0E9
01190608   83C2 05         ADD EDX,5
0119060B   2BCA           SUB ECX,EDX
0119060D   894A FC         MOV DWORD PTR DS:[EDX-4],ECX
01190610   33C0           XOR EAX,EAX
01190612   C3             RETN
```

向下找到

```
0119068B   894E 14        MOV DWORD PTR DS:[ESI+14],ECX
0119068E   FFD7           CALL EDI
01190690   8985 3F130010   MOV DWORD PTR SS:[EBP+1000133F],>
01190696   8BF0           MOV ESI,EAX
01190698   8B4B 14        MOV ECX,DWORD PTR DS:[EBX+14]
0119069B   5A             POP EDX
0119069C   EB 0C          JMP SHORT qtintf.011906AA
0119069E   03CA           ADD ECX,EDX
011906A0   68 00800000    PUSH 8000
011906A5   6A 00          PUSH 0
011906A7   57             PUSH EDI
011906A8   FF11           CALL DWORD PTR DS:[ECX]
011906AA   8BC6           MOV EAX,ESI
011906AC   5A             POP EDX
011906AD   5E             POP ESI
011906AE   5F             POP EDI
011906AF   59             POP ECX
011906B0   5B             POP EBX
011906B1   5D             POP EBP
011906B2   FFE0           JMP EAX-------》找到这里，在此下硬件执行断点，F9 到达此处，F8 一下就到了 OEP。
```

------------------------------`JMP EAX`-----------------------------------------

```
00E037A8   /EB 10         JMP SHORT qtintf.00E037BA-------》这里就是要找的 OEP 了。
00E037AA   |66:623A        BOUND DI,DWORD PTR DS:[EDX]
00E037AD   |43             INC EBX
00E037AE   |2B2B           SUB EBP,DWORD PTR DS:[EBX]
00E037B0   |48             DEC EAX
00E037B1   |4F             DEC EDI
00E037B2   |4F             DEC EDI
00E037B3   |4B             DEC EBX
00E037B4   |90             NOP
00E037B5   -|E9 80DD0301    JMP 01E4153A
00E037BA   \A1 07DD0301    MOV EAX,DWORD PTR DS:[103DD07]
00E037BF    C1E0 02        SHL EAX,2
00E037C2    A3 0BDD0301    MOV DWORD PTR DS:[103DD0B],EAX
00E037C7    8B4424 08      MOV EAX,DWORD PTR SS:[ESP+8]
00E037CB    A3 79DD0301    MOV DWORD PTR DS:[103DD79],EAX
00E037D0    FF1485 69DD0301 CALL DWORD PTR DS:[EAX*4+103DD69>
00E037D7    833D 79DD0301 0>CMP DWORD PTR DS:[103DD79],1
00E037DE    75 5E          JNZ SHORT qtintf.00E0383E
00E037E0    803D 13DD0301 0>CMP BYTE PTR DS:[103DD13],0
```

大家看到没有

`00E037A8 > B8 F0051901 MOV EAX,qtintf.011905F0---》`程序载入的入口

`00E037A8 > /EB 10 JMP SHORT qtintf.00E037BA----》`OEP 的入口

这两个是相同的。

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `qtintf.dll`，然后完整脱壳，得到 `dumped.dll`。

### 第二步：`Import Table`

在 OEP 附进随便找个 `CALL` 进入，找到一个函数调用

```
00E037E9   E8 A4FA2000    CALL qtintf.01013292      ; JMP 到 kernel32.GetVersion
```

就这个吧，进入之后就看到了 IAT

```
011480C8  77DA6BF0  ADVAPI32.RegCloseKey----》这是 IAT_Start
011480CC  77DA761B  ADVAPI32.RegOpenKeyExA
011480D0  77DA7883  ADVAPI32.RegQueryValueExA
011480D4  00000000

011480D8  77DA6BF0  ADVAPI32.RegCloseKey
011480DC  77DA761B  ADVAPI32.RegOpenKeyExA
011480E0  77DA7883  ADVAPI32.RegQueryValueExA
```

。。。

```
01148A94  769E2A2F  OLE32.RevokeDragDrop
01148A98  00000000
01148A9C  769ADCF8  OLE32.CoGetMalloc
01148AA0  769E3D23  OLE32.CoLockObjectExternal
01148AA4  76A803F1  OLE32.DoDragDrop
01148AA8  769AF6DA  OLE32.OleInitialize
01148AAC  769E3373  OLE32.OleUninitialize
01148AB0  769AF61A  OLE32.RegisterDragDrop
01148AB4  769C4BED  OLE32.ReleaseStgMedium
01148AB8  769E2A2F  OLE32.RevokeDragDrop----》这是 IAT_End
```

`IAT_Start=011480C8`

`IAT_End =01148AB8`

运行 `ImportREC`，选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取 DLL”，选择 `qtintf.dll`，填入 `RVA=003B80C8`、`大小=9F4`、`OEP=000737A8`，点“Get Import”。用 `PEditor` 纠正 `dumped.dll` 的 `DumpFixer`，修正区块。`FixDump！`

### 第三步：`Relocation Table`

重载入 DLL 程序

来到第一步的这里：

`Ctrl+G` 输入 `011905F0`，来到了如下

```
011905F0   B8 75F318F1     MOV EAX,F118F375                 ; 来到了这里，在此处下 F2 断点。F9 运行到此处，去掉 F2 断点。
011905F5   8D88 9E120010   LEA ECX,DWORD PTR DS:[EAX+100012>
011905FB   8941 01         MOV DWORD PTR DS:[ECX+1],EAX
011905FE   8B5424 04       MOV EDX,DWORD PTR SS:[ESP+4]
01190602   8B52 0C         MOV EDX,DWORD PTR DS:[EDX+C]
01190605   C602 E9         MOV BYTE PTR DS:[EDX],0E9
01190608   83C2 05         ADD EDX,5
0119060B   2BCA           SUB ECX,EDX
0119060D   894A FC         MOV DWORD PTR DS:[EDX-4],ECX
01190610   33C0           XOR EAX,EAX
01190612   C3             RETN
```

可以向下找到这个：

```
0119068E   FFD7           CALL EDI----------》在这里下 F2 断点，进入看一看。
01190690   8985 3F130010   MOV DWORD PTR SS:[EBP+1000133F],>
01190696   8BF0           MOV ESI,EAX
01190698   8B4B 14        MOV ECX,DWORD PTR DS:[EBX+14]
0119069B   5A             POP EDX
0119069C   EB 0C          JMP SHORT qtintf.011906AA
0119069E   03CA           ADD ECX,EDX
011906A0   68 00800000    PUSH 8000
011906A5   6A 00          PUSH 0
011906A7   57             PUSH EDI
011906A8   FF11           CALL DWORD PTR DS:[ECX]
011906AA   8BC6           MOV EAX,ESI
```



### 代码段 1：函数返回与跳转

```
011906AC    5A                POP EDX
011906AD    5E                POP ESI
011906AE    5F                POP EDI
011906AF    59                POP ECX
011906B0    5B                POP EBX
011906B1    5D                POP EBP
011906B2    FFE0              JMP EAX
```

### CALL EDI 分析

```
00880AA0    53                PUSH EBX              ; 进入到这里了。
00880AA1    57                PUSH EDI
00880AA2    56                PUSH ESI
00880AA3    55                PUSH EBP
00880AA4    E8 00000000       CALL 00880AA9
00880AA9    5D                POP EBP
00880AAA    81ED 4C130010     SUB EBP,1000134C
00880AB0    8DB5 43130010     LEA ESI,DWORD PTR SS:[EBP+100013>
00880AB6    8B46 FC           MOV EAX,DWORD PTR DS:[ESI-4]
```

（中间省略部分）

```
00880B44   /0F85 94000000    JNZ 00880BDE
00880B4A   |56               PUSH ESI
00880B4B   |E8 40030000      CALL 00880E90
00880B50   |56               PUSH ESI
00880B51   |E8 55020000      CALL 00880DAB
00880B56   |56               PUSH ESI
00880B57   |E8 53010000      CALL 00880CAF--------》这个 CALL 就是我们要找的了
00880B5C   |90               NOP
```

### CALL 00880CAF 分析

```
00880CAF    55                PUSH EBP-----》 进入到这里了。
00880CB0    8BEC              MOV EBP,ESP
00880CB2    83C4 F8           ADD ESP,-8
00880CB5    53                PUSH EBX
00880CB6    57                PUSH EDI
00880CB7    56                PUSH ESI
00880CB8    8B7D 08           MOV EDI,DWORD PTR SS:[EBP+8]
00880CBB    8B47 04           MOV EAX,DWORD PTR DS:[EDI+4]----》EAX=DS:[00880954]=00D90000 (qtintf.00D90000)
00880CBE    8B5F 08           MOV EBX,DWORD PTR DS:[EDI+8]----》EBX=DS:[00880958]=00D90000 (qtintf.00D90000)
00880CC1    3BC3              CMP EAX,EBX---------------------》比较映像基址是否相等，相等则不处理。
00880CC3    74 59             JE SHORT 00880D1E---------------》还是使用 FLY 大侠的（改标志位 Z=0，使这里不跳转）
00880CC5    8B77 38           MOV ESI,DWORD PTR DS:[EDI+38]---》ESI=DS:[00880988]=003D7000,重定位的 RVA
00880CC8    85F6              TEST ESI,ESI
00880CCA    74 52             JE SHORT 00880D1E
00880CCC    03F3              ADD ESI,EBX---------------------》ESI=00D90000+003D7000=1167000
00880CCE    8BD3              MOV EDX,EBX
00880CD0    2BD8              SUB EBX,EAX
00880CD2    895D F8           MOV DWORD PTR SS:[EBP-8],EBX
00880CD5    AD                LODS DWORD PTR DS:[ESI]
00880CD6    8BD8              MOV EBX,EAX
00880CD8    03DA              ADD EBX,EDX
00880CDA    AD                LODS DWORD PTR DS:[ESI]
00880CDB    85C0              TEST EAX,EAX
00880CDD    74 3F             JE SHORT 00880D1E
00880CDF    8BC8              MOV ECX,EAX
00880CE1    83E9 08           SUB ECX,8
00880CE4    85C9              TEST ECX,ECX
00880CE6   ^ 74 ED             JE SHORT 00880CD5
00880CE8    66:C745 FE FFFFF   MOV WORD PTR SS:[EBP-2],0FFFF
00880CEE    66:AD             LODS WORD PTR DS:[ESI]
00880CF0    66:837D FE FF     CMP WORD PTR SS:[EBP-2],0FFFF
00880CF5    74 04             JE SHORT 00880CFB
00880CF7    66:0345 FE        ADD AX,WORD PTR SS:[EBP-2]
00880CFB    66:8945 FE        MOV WORD PTR SS:[EBP-2],AX
00880CFF    8BF8              MOV EDI,EAX
00880D01    81E7 FF0F0000     AND EDI,0FFF
00880D07    03FB              ADD EDI,EBX
00880D09    66:C1E8 0C        SHR AX,0C
00880D0D    66:83F8 03        CMP AX,3
00880D11    75 05             JNZ SHORT 00880D18
00880D13    8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]
00880D16    0107              ADD DWORD PTR DS:[EDI],EAX
00880D18    49                DEC ECX
00880D19    49                DEC ECX
00880D1A   ^ 75 D2             JNZ SHORT 00880CEE
00880D1C   ^ EB B7             JMP SHORT 00880CD5
00880D1E    5E                POP ESI-------------------------------》在此下 F4 运行到此处，这时 ESI 的值为 ESI=0118E4E0
00880D1F    5F                POP EDI
00880D20    5B                POP EBX
00880D21    C9                LEAVE
00880D22    C2 0400           RETN 4
```

重定位表的大小是：`0118E4E0-8-1167000=274D8`（其实我们知道了 `003D7000` 之后，用 `WinHex` 打开 `dumped\_.dll`，到达此处之后，向下翻到为 0 处就是重定位表的大小了）。

使用 `WinHex` 打开 `dumped\_.dll`，复制 `003D7000－003FE4D8` 之间的 16 进制数值，另存为 `1.bin`。

运行看雪老师写的辅助修复 `PECompact` 加壳 DLL 重定位表的工具 `PEComAngela.exe`，打开 `1.bin`，创建 `pediy.bin` 文件失败了！

[http://bbs.pediy.com/showthread.php?s=&threadid=40785](http://bbs.pediy.com/showthread.php?s=&threadid=40785) 这里有讨论。

后来使用了 `Relox V1.0a` 也不能修复。

这里我也感觉很郁闷，为什么不能创建成功呢？好了我就进入程序内看一看吧。经过仔细考虑之后，进入此 `CALL 00880CAF` 修改了如下代码：

### 修改后的 CALL 00880CAF 代码

```
00880CAF    55                PUSH EBP
00880CB0    8BEC              MOV EBP,ESP
00880CB2    83C4 F8           ADD ESP,-8
00880CB5    53                PUSH EBX
00880CB6    57                PUSH EDI
00880CB7    56                PUSH ESI
00880CB8    8B7D 08           MOV EDI,DWORD PTR SS:[EBP+8]
00880CBB    8B47 04           MOV EAX,DWORD PTR DS:[EDI+4]
00880CBE    8B5F 08           MOV EBX,DWORD PTR DS:[EDI+8]
00880CC1    3BC3              CMP EAX,EBX
00880CC3    74 59             JE SHORT 00880D1E-----------------------》注册意要改标志位 Z=0，使这里不跳转哟
00880CC5    8B77 38           MOV ESI,DWORD PTR DS:[EDI+38]
00880CC8    85F6              TEST ESI,ESI
00880CCA    74 52             JE SHORT 00880D1E
00880CCC    03F3              ADD ESI,EBX
00880CCE    8BD3              MOV EDX,EBX
00880CD0    2BD8              SUB EBX,EAX
00880CD2    895D F8           MOV DWORD PTR SS:[EBP-8],EBX
00880CD5    AD                LODS DWORD PTR DS:[ESI]
00880CD6    8BD8              MOV EBX,EAX
00880CD8    03DA              ADD EBX,EDX
00880CDA    AD                LODS DWORD PTR DS:[ESI]
00880CDB    85C0              TEST EAX,EAX
00880CDD    74 3F             JE SHORT 00880D1E
00880CDF    8BC8              MOV ECX,EAX
00880CE1    83E9 08           SUB ECX,8
00880CE4    85C9              TEST ECX,ECX
00880CE6   ^ 74 ED             JE SHORT 00880CD5
00880CE8    66:C745 FE FFFFF   MOV WORD PTR SS:[EBP-2],0FFFF
00880CEE    66:AD             LODS WORD PTR DS:[ESI]
00880CF0    66:837D FE FF     CMP WORD PTR SS:[EBP-2],0FFFF
00880CF5    74 54             JE SHORT 00880D4B
00880CF7    66:0345 FE        ADD AX,WORD PTR SS:[EBP-2]
00880CFB    EB 4E             JMP SHORT 00880D4B
00880CFD    90                NOP--------------------------------
00880CFE    90                NOP                           |
00880CFF    8BF8              MOV EDI,EAX
00880D01    81E7 FF0F0000     AND EDI,0FFF                      |
00880D07    03FB              ADD EDI,EBX
00880D09    66:C1E8 0C        SHR AX,0C                     |
00880D0D    66:83F8 03        CMP AX,3                         ---> 这里的代码相当与废了。
00880D11    75 05             JNZ SHORT 00880D18                |
00880D13    8B45 F8           MOV EAX,DWORD PTR SS:[EBP-8]
00880D16    0107              ADD DWORD PTR DS:[EDI],EAX        |
00880D18    49                DEC ECX
00880D19    49                DEC ECX
00880D1A   ^ 75 D2             JNZ SHORT 00880CEE                |
00880D1C   ^ EB B7             JMP SHORT 00880CD5-----------------
00880D1E    5E                POP ESI
00880D1F    5F                POP EDI
00880D20    5B                POP EBX
00880D21    C9                LEAVE
00880D22    C2 0400           RETN 4------------------》此处返回。
00880D25    55                PUSH EBP
00880D26    8BEC              MOV EBP,ESP
00880D28    53                PUSH EBX
```



> `00880D29` `57` PUSH EDI
> `00880D2A` `56` PUSH ESI
> `00880D2B` `8B75 08` MOV ESI,`DWORD PTR SS:[EBP+8]`
> `00880D2E` `66:8B4D 10` MOV CX,`WORD PTR SS:[EBP+10]`
> `00880D32` `8B55 0C` MOV EDX,`DWORD PTR SS:[EBP+C]`
> `00880D35` `AD` LODS `DWORD PTR DS:[ESI]`
> `00880D36` `85C0` TEST EAX,EAX
> `00880D38` `74 07` JE SHORT `00880D41`
> `00880D3A` `03C2` ADD EAX,EDX
> `00880D3C` `66:8908` MOV `WORD PTR DS:[EAX]`,CX
> `00880D3F` `EB F4` JMP SHORT `00880D35`
> `00880D41` `5E` POP ESI
> `00880D42` `5F` POP EDI
> `00880D43` `5B` POP EBX
> `00880D44` `C9` LEAVE
> `00880D45` `C2 0C00` RETN `0C`
> `00880D48` `90` NOP
> `00880D49` `90` NOP
> `00880D4A` `90` NOP
> `00880D4B` `66:8945 FE` MOV `WORD PTR SS:[EBP-2]`,AX
> `00880D4F` `66:36:8946 FE` MOV `WORD PTR SS:[ESI-2]`,AX--------》还原重定位表数据就是这里了。
> `00880D54` `8BF8` MOV EDI,EAX
> `00880D56` `90` NOP
> `00880D57` `90` NOP
> `00880D58` `90` NOP
> `00880D59` `81E7 FF0F0000` AND EDI,`0FFF`
> `00880D5F` `03FB` ADD EDI,EBX
> `00880D61` `66:C1E8 0C` SHR AX,`0C`
> `00880D65` `66:83F8 03` CMP AX,`3`
> `00880D69` `75 05` JNZ SHORT `00880D70`
> `00880D6B` `8B45 F8` MOV EAX,`DWORD PTR SS:[EBP-8]`
> `00880D6E` `0107` ADD `DWORD PTR DS:[EDI]`,EAX
> `00880D70` `49` DEC ECX
> `00880D71` `49` DEC ECX
> `00880D72` `0F85 76FFFFFF` JNZ `00880CEE`
> `00880D78` `E9 58FFFFFF` JMP `00880CD5`

> 下面是二进数据，可以直接复制用：

> ```
> 55 8B EC 83 C4 F8 53 57 56 8B 7D 08 8B 47 04 8B 5F 08 3B C3 74 59 8B 77 38 85 F6 74 52 03 F3 8B
> D3 2B D8 89 5D F8 AD 8B D8 03 DA AD 85 C0 74 3F 8B C8 83 E9 08 85 C9 74 ED 66 C7 45 FE FF FF 66
> AD 66 83 7D FE FF 74 54 66 03 45 FE EB 4E 90 90 8B F8 81 E7 FF 0F 00 00 03 FB 66 C1 E8 0C 66 83
> F8 03 75 05 8B 45 F8 01 07 49 49 75 D2 EB B7 5E 5F 5B C9 C2 04 00 55 8B EC 53 57 56 8B 75 08 66
> 8B 4D 10 8B 55 0C AD 85 C0 74 07 03 C2 66 89 08 EB F4 5E 5F 5B C9 C2 0C 00 90 90 90 66 89 45 FE
> 66 36 89 46 FE 8B F8 90 90 90 81 E7 FF 0F 00 00 03 FB 66 C1 E8 0C 66 83 F8 03 75 05 8B 45 F8 01
> 07 49 49 0F 85 76 FF FF FF E9 58 FF FF FF
> ```

> 这里返回之后。

> 并运行到第一步提到的 `OEP` 处之后，请重新用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `qtintf.dll`，然后完整脱壳，得到 `unqtintf.dll`。

> 再执行第二步：运行 `ImportREC`，选中 `Ollydbg` 的 `loaddll.exe` 的进程，然后点“选取 DLL”，选择 `qtintf.dll`，填入 `RVA=003B80C8`、`大小=9F4`、`OEP=000737A8`，点“Get Import”。用 `PEditor` 纠正 `unqtintf.dll` 的 `DumpFixer`，修正区块。`FixDump`！

> 最后用 `LordPE` 修正 `unqtintf_.dll` 的重定位表 `RVA=003D7000`、`大小=000274D8`，保存之。自己优化一下吧！

> 用 `Dll_LoadEx.exe` 工具测试可以载入。

--------------------------------------------------------------------------------
### 【经验总结】

没什么经验只能说玩一玩还可以吧！

2007 年 03 月 09 日 21:00:00

`4dd466d0e1e0cc123afbe504da2462ca`
