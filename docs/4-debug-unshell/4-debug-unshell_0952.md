
# RLPack 加壳的 Armadillo find protected 1.8

> 原文：[`www.pediy.com/kssd/pediy10/79939.html`](https://www.pediy.com/kssd/pediy10/79939.html)

**【文章标题】**: `RLPack` 加壳的 `Armadillo` `find` `protected` `1.8`

**【文章作者】**: `CCDebuger`

**【下载地址】**: 自己搜索下载

**【作者声明】**: 只是调试时的笔记，权当灌水。

---

**【详细过程】**

原版加了 `RLPack` `Full` `Edition` 的壳，啥版本我不大清楚。在我的 `Symantec` `AntiVirus` 企业版下会报毒，只好脱掉。

虽然 `RLPack` `Full` `Edition` 应该是个压缩壳，不过这个 `Armadillo` `Find` `Protected` `1.8` 加的壳在 `OD` 里跑的话还会出错，原来是检测到了调试器。来看看怎么把这个壳去掉吧。这玩意我还试用了 `Syser` `Debugger` 来脱的，本来也想把用 `Syser` `Debugger` 脱的过程加进去，不过嫌烦，就没写了。改天有空再写吧。

### 一、寻找 OEP

先用 `OD` 载入程序看看，因为是压缩壳，`OEP` 应该很好找，根据成对的 `PUSHAD`、`POPAD`，上下翻翻就行了：

```
引用:   00415532 >   60                  PUSHAD                                      ; 加壳程序的 EP
  00415533     E8 00000000     CALL 00415538
  00415538     8B2C24          MOV EBP,DWORD PTR SS:[ESP]
  省略代码若干...
  0041567D     A9 00000080     TEST EAX,80000000
  00415682     74 1A           JE SHORT 0041569E
  00415684     35 00000080     XOR EAX,80000000
  00415689     50              PUSH EAX
  0041568A     8985 0A140000   MOV DWORD PTR SS:[EBP+140A],EAX
  00415690     8B85 431F0000   MOV EAX,DWORD PTR SS:[EBP+1F43]
  00415696     C700 20202000   MOV DWORD PTR DS:[EAX],202020
  0041569C     EB 06           JMP SHORT 004156A4
  0041569E     FFB5 431F0000   PUSH DWORD PTR SS:[EBP+1F43]
  004156A4     FFB5 3F1F0000   PUSH DWORD PTR SS:[EBP+1F3F]
  004156AA     FF95 F6030000   CALL DWORD PTR SS:[EBP+3F6]
  004156B0     85C0            TEST EAX,EAX
  004156B2     0F84 17030000   JE 004159CF
  004156B8     E8 F20E0000     CALL 004165AF                            ; 这个 CALL 是用来加密输入表的，NOP 掉
  004156BD     C785 0A140000 0>MOV DWORD PTR SS:[EBP+140A],0
  004156C7     8907            MOV DWORD PTR DS:[EDI],EAX
  004156C9     83C7 04         ADD EDI,4
  省略代码若干...
  00415721     FF95 FE030000   CALL DWORD PTR SS:[EBP+3FE]
  00415727     E8 06050000     CALL 00415C32
  0041572C     E8 A7000000     CALL 004157D8
  00415731     61              POPAD
  00415732  -  E9 A5DDFEFF     JMP 004034DC                             ; 跳到程序的 OEP 004034DC
  00415737     83BD FE130000 0>CMP DWORD PTR SS:[EBP+13FE],0
  0041573E     74 16           JE SHORT 00415756
```

---

### 二、干掉调试器检测

程序检测到调试器时应该有个对话框，不过这个对话框你看不到，只听见一声响，就是个系统常见的应用程序错误，然后退出。现在重新载入程序，设断 `BP MessageBoxA`，`F9` 运行，会被断下，看堆栈：

```
引用:   0012FF6C   004167F0   /CALL 到 MessageBoxA 来自 ArmaFP18.004167EA
  0012FF70   00000000   |hOwner = NULL
  0012FF74   00416DE4   |Text = "Debugger detected  - please close it down and restart!
  Windows NT users: Please note that having the WinIce/SoftIce
  service installed means that you are running a debugger!"
  0012FF78   00416989   |Title = "Error:"
  0012FF7C   00000030   \Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
```

我们在汇编窗口中转到 `004167F0` 处，往上看看，很容易就知道哪里能跳过这个对话框：

```
引用:   00416794     60              PUSHAD
  00416795     83BD B3170000 0>CMP DWORD PTR SS:[EBP+17B3],1            ; 在这设个硬件执行断点，重新载入程序
  0041679C     74 09           JE SHORT 004167A7
  0041679E     83BD B7170000 0>CMP DWORD PTR SS:[EBP+17B7],1
  004167A5     75 4D           JNZ SHORT 004167F4                         ; 这里一定要跳，不必改代码，修改一下标志位，或者运行到上面那条指令时，改一下上面 [EBP+17B7] 中的值，让它跳。否则就完蛋
  004167A7     8D9D 46140000   LEA EBX,DWORD PTR SS:[EBP+1446]
  004167AD     53              PUSH EBX
  004167AE     FF95 F2030000   CALL DWORD PTR SS:[EBP+3F2]
  004167B4     8985 1A140000   MOV DWORD PTR SS:[EBP+141A],EAX
  004167BA     81BD 36140000 0>CMP DWORD PTR SS:[EBP+1436],ABBC680D
  004167C4     75 12           JNZ SHORT 004167D8
  004167C6     FFB5 36140000   PUSH DWORD PTR SS:[EBP+1436]
  004167CC     50              PUSH EAX
  004167CD     E8 07010000     CALL 004168D9
  004167D2     8985 36140000   MOV DWORD PTR SS:[EBP+1436],EAX
  004167D8     6A 30           PUSH 30
  004167DA     8D85 51140000   LEA EAX,DWORD PTR SS:[EBP+1451]
  004167E0     50              PUSH EAX
  004167E1     8D85 AC180000   LEA EAX,DWORD PTR SS:[EBP+18AC]
  004167E7     50              PUSH EAX
  004167E8     6A 00           PUSH 0
  004167EA     FF95 36140000   CALL DWORD PTR SS:[EBP+1436]             ; 这就是检测到调试器的对话框
  004167F0     61              POPAD
  004167F1     58              POP EAX
  004167F2     61              POPAD
  004167F3     C3              RETN
  004167F4     61              POPAD
  004167F5     C3              RETN
```

到这壳的调试器检测应该就解决了。

检测调试器的代码：

```
引用:   004166F8     8D85 B7170000   LEA EAX,DWORD PTR SS:[EBP+17B7]
  004166FE     50                PUSH EAX                          ; 放置调试标志位的地址
  004166FF     6A FF             PUSH -1
  00416701     FF95 A7170000   CALL DWORD PTR SS:[EBP+17A7]        ; kernel32.CheckRemoteDebuggerPresent
  00416707     8B85 A7170000   MOV EAX,DWORD PTR SS:[EBP+17A7]       ; kernel32.CheckRemoteDebuggerPresent
  0041670D     8138 8B442408   CMP DWORD PTR DS:[EAX],824448B       ; 检测 CheckRemoteDebuggerPresent 函数的开头几个字节
  00416713     75 0A             JNZ SHORT 0041671F                  ; 不跳则有调试器
  00416715     C785 B7170000 0>MOV DWORD PTR SS:[EBP+17B7],1            ; 置标志位
  0041671F     64:A1 30000000  MOV EAX,DWORD PTR FS:[30]
  00416725     83C0 68          ADD EAX,68
  00416728     8B00            MOV EAX,DWORD PTR DS:[EAX]
  0041672A     83F8 70          CMP EAX,70
  0041672D     75 0A             JNZ SHORT 00416739
  0041672F     C785 B3170000 0>MOV DWORD PTR SS:[EBP+17B3],1            ; 第二个置标志位的地方
  00416739     64:A1 18000000  MOV EAX,DWORD PTR FS:[18]
```

---

### 三、去除输入表加密

这个壳还会加密输入表，我们先去掉调试器检测，跑到 `OEP` 去看看：

```
引用:   004034DC     55                PUSH EBP                                        ; OEP
  004034DD     8BEC            MOV EBP,ESP
  004034DF     6A FF             PUSH -1
  004034E1     68 48904000     PUSH 00409048
  004034E6     68 10404000     PUSH 00404010
  004034EB     64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
  004034F1     50              PUSH EAX
  004034F2     64:8925 0000000>MOV DWORD PTR FS:[0],ESP
  004034F9     83EC 58          SUB ESP,58
  004034FC     53              PUSH EBX
  004034FD     56              PUSH ESI
  004034FE     57              PUSH EDI
  004034FF     8965 E8          MOV DWORD PTR SS:[EBP-18],ESP
  00403502     FF15 10614000   CALL DWORD PTR DS:[406110]                       ; 这里可以看出输入表被加密了
  00403508     33D2            XOR EDX,EDX
  0040350A     8AD4            MOV DL,AH
```

我们在数据窗口中转到 `00406110` 处，上下翻翻，就知道输入表是从 `00406000` 处开始的。现在用 `OD` 重新载入程序，在数据窗口中转到 `00406000`，设内存写入断点，`F9` 运行，会被断下：

```
引用:   00415858     33DB            XOR EBX,EBX
  0041585A     A4                  MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]      ; 断在这里
  0041585B     B3 02             MOV BL,2
  0041585D     E8 6D000000     CALL 004158CF
  00415862  ^  73 F6             JNB SHORT 0041585A
  00415864     33C9            XOR ECX,ECX
```

继续 `F9`，输入表会解开，跑到了前面我们设的检测调试器的那个硬件断点那。修改一下标志位，不让壳检测到调试器，继续 `F9`：

```
引用:   0041567D     A9 00000080     TEST EAX,80000000
  00415682     74 1A           JE SHORT 0041569E
  00415684     35 00000080     XOR EAX,80000000
  00415689     50              PUSH EAX
  0041568A     8985 0A140000   MOV DWORD PTR SS:[EBP+140A],EAX
  00415690     8B85 431F0000   MOV EAX,DWORD PTR SS:[EBP+1F43]
  00415696     C700 20202000   MOV DWORD PTR DS:[EAX],202020
  0041569C     EB 06           JMP SHORT 004156A4
  0041569E     FFB5 431F0000   PUSH DWORD PTR SS:[EBP+1F43]
  004156A4     FFB5 3F1F0000   PUSH DWORD PTR SS:[EBP+1F3F]
  004156AA     FF95 F6030000   CALL DWORD PTR SS:[EBP+3F6]
```



### 汇编代码分析

```asm
  004156B0    85C0            TEST EAX,EAX  
  004156B2    0F84 17030000   JE 004159CF  
  004156B8    E8 F20E0000     CALL 004165AF          ; 这个 CALL 是用来加密输入表的，NOP 掉  
  004156BD    C785 0A140000 0\>MOV DWORD PTR SS:\[EBP+140A\],0  
  004156C7    8907            MOV DWORD PTR DS:\[EDI\],EAX     ; 断在这里，这里就是加密输入表了  
  004156C9    83C7 04        ADD EDI,4  
  004156CC    8B85 431F0000  MOV EAX,DWORD PTR SS:\[EBP+1F43\] 
```

### 处理方法

这时候有两种方法：

1. 知道了加密输入表 `CALL` 的位置，重新用 `OD` 载入程序，`NOP` 掉加密的 `CALL`，就可以得到完整的输入表，然后到 `OEP` `dump`，用 `ImportREC` 重建输入表。不过这种方法我不大推荐，我推荐下面第二种方法。

2. 当我们断在 `004156C7` 处时，我们在数据窗口中看一下 `00406000` 处的内容，查看方式选“长型->地址”：

   00406000  00009590
   00406004  0000959C
   00406008  000095AA
   0040600C  000095B6
   00406010  000095C2
   00406014  000095D4
   00406018  000095E2
   0040601C  000095F6
   00406020  00009604
   00406024  0000961A
   00406028  0000962A
   0040602C  0000963E
   省略若干...

这个一看就是原始的输入表，那还等啥？直接用 `LordPE` 修正镜像大小，然后 `dump`。用 `PETools` 的编辑工具把 `dump` 程序的 `OEP` 改成 `00034DC`，再用 `010editor` 之类的 16 进制编辑工具定位到偏移 `6000` 处，根据 `IAT` 最后以 20 个 0 结束，就可以确定输入表的开始地址和大小了。往下搜索有二十个 `HEX` 值 `0`，根据找到的最近位置，很容易就确定了 `IAT` 的 `RVA` 是 `000093AC`，大小是 `50`。在 `PE` 编辑工具中填上我们上面得到的值，现在输入表就修复好了，根本不需要使用 `ImportREC`。同样发现输入表都解开后才检测调试器的，这样一来，连避开调试器检测都不需要了，到检测调试器前就可以 `dump` 来修正了。

---

### 三、脱壳后的优化

要了解一下原来没加壳的程序有几个区段的话，只要重新载入程序，`BP` `VirtualProtect`，`F9` 运行，注意看堆栈就可以了：

**1. .text 段**

```
0012FF70   004155B4  /CALL 到 VirtualProtect 来自 ArmaFP18.004155AE  
0012FF74   00401000  |Address \= ArmaFP18.00401000  
0012FF78   00004A1E  |Size = 4A1E (18974.)  
0012FF7C   00000020  |NewProtect \= PAGE\_EXECUTE\_READ  
0012FF80   00416EEA  \\pOldProtect \= ArmaFP18.00416EEA  
```

**2. .rdata 段**

```
0012FF70   004155B4  /CALL 到 VirtualProtect 来自 ArmaFP18.004155AE  
0012FF74   00406000  |Address \= ArmaFP18.00406000  
0012FF78   00003BE4  |Size = 3BE4 (15332.)  
0012FF7C   00000040  |NewProtect \= PAGE\_EXECUTE\_READWRITE  
0012FF80   00416EEA  \\pOldProtect \= ArmaFP18.00416EEA  
```

**3. .data 段**

```
0012FF70   004155B4  /CALL 到 VirtualProtect 来自 ArmaFP18.004155AE  
0012FF74   0040A000  |Address \= ArmaFP18.0040A000  
0012FF78   0000349C  |Size = 349C (13468.)  
0012FF7C   00000040  |NewProtect \= PAGE\_EXECUTE\_READWRITE  
0012FF80   00416EEA  \\pOldProtect \= ArmaFP18.00416EEA  
```

**4. .rsrc 段**

```
0012FF70   004155B4  /CALL 到 VirtualProtect 来自 ArmaFP18.004155AE  
0012FF74   0040E000  |Address \= ArmaFP18.0040E000  
0012FF78   00000EE8  |Size = EE8 (3816.)  
0012FF7C   00000020  |NewProtect \= PAGE\_EXECUTE\_READ  
0012FF80   00416EEA  \\pOldProtect \= ArmaFP18.00416EEA  
```

剩下其他的都是垃圾了，按上面显示的重建一下各个区段，按 `1000` 对齐，偏移 `F000` 后面的都可以删掉。附上脱壳后的程序，界面处理了一下，使字体显得好看点。

---

**【版权声明】**: 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

**上传的附件**

| 缩略图 | 文件名 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `ArmaFP.v1.8.Unpacked.rar` \[解压密码：pediy\] |
