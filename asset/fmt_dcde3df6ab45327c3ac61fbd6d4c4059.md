
# ARMdillo 3.x COPYMEM II 标准壳之传奇外挂

> 原文：`www.pediy.com/kssd/pediy07/pediy7-584.htm`

**目标**：传奇外挂 1.07  
**工具**：`Olydbg1.1`(diy 版)、`LORDPE`、`ImportREC1.6F`  
**任务**：`ARM` 3.6 或以下版本的 `COPYMEM II` 标准壳  
**操作平台**：`WinXP` sp2  
**作者**：`LOVEBOOM[DFCG][FCG][US]`  
**相关链接**：……

**简要说明**：这是一个 `ARM` 3.6 或以下版本的 `COPYMEM II` 壳，壳没有乱序，也没有抽壳代码。所以比较容易。

**详细过程**：

`OD` 设置：忽略全部异常，用修改过的 `OD` (必须 PATCH 过 `OutputDebugStringA` 漏洞)，隐藏 `OD`，忽略 `LOCKX` (`C000001E`)。

脱壳过程分为四部分：找 `OEP`、`DUMP` 程序、修复 `IAT`，修复优化主程序。

## 第一部分：找 OEP

用 `OD` 载入目标程序：

```
0056E379  >/$  55            PUSH EBP                             ;  壳入口
0056E37A  |.   8BEC          MOV EBP,ESP
……
```

因为 3.7 以下版本的 `ARM` 不会检测全部 API 的 `CC` 断点。所以可以直接下断 `BP`。先下 `BP WaitForDebugEvnet` 断点，然后 `F9` 运行，到这里停下：

```
7C85A268  >   8BFF            MOV EDI,EDI                          ; 第一次停这里,停下后注意观察堆栈.
7C85A26A    55             PUSH EBP
7C85A26B    8BEC            MOV EBP,ESP
……
```

停下后去除 `CC` 断点，看看堆栈的信息：

![](img/a0b6805d14ec2123177ef68d47be9c28.jpg)

注意 `ESP+4` 处的地址，在 `dump` 窗口中显示该地址信息：

![](img/d375380918f269ff2ea1e3cbbb4de598.jpg)

再下 `bp WaiteProcessMemory` 断点并运行，断下后看看我们刚才地 `DUMP` 窗口中就可以找到程序有 `OEP` 了。

![](img/3e5c097e0d8d9676419e8188bf5df456.jpg)

`OEP` 在 `1C` 处，这个程序的 `OEP` 也就是 `4014AC`，记下先。到这里第一部分就算完成了。

## 第二部分：DUMP 程序

知道 `OEP` 后，如果你现在直接去 `follow 4014AC` 的话，你将什么都得不到。看看刚才断下地方的堆栈信息，我们可以得到我们要用的东西：

![](img/3348a5f699ba62d20b381159e4b463e9.jpg)

先算出真正代码所在的位置，计算方法为：`OEP` 减去堆栈 `ESP+8` 处的 `address 401000` 加上 `ESP+0C` 处的 `Buffer 003c64b8`。（至于为什么，可以去参考相关的文章。）所以这个程序的 `OEP` 代码为：`4014AC-401000+003C64B8=003C6964`。

```
003C6964   /EB 10           JMP SHORT 003C6976                     ; 这里就是程序原 OEP 代码
003C6966   |66:623A         BOUND DI,DWORD PTR DS:[EDX]
003C6969   |43              INC EBX
003C696A   |2B2B            SUB EBP,DWORD PTR DS:[EBX]
003C696C   |48              DEC EAX
```

以前有关文章到这里后要再处理下再 `DUMP`，不过这个程序不用处理的。附以前文章里说的处理地方，你也可以直接跳过这一步到 `NEXT1` 处。

下面以这个为例说说处理的情况。先看堆栈：

```
0012D98C   0056C421  /CALL to WriteProcessMemory from MLEx.0056C41B
0012D990   0000004C  |hProcess = 0000004C (window)
0012D994   00401000  |Address = 401000
0012D998   003C64B8  |Buffer = 003C64B8
0012D99C   00001000  |BytesToWrite = 1000 (4096.)
0012D9A0   0012DAA8  \pBytesWritten = 0012DAA8
0012D9A4   00000002
0012D9A8   00000002
0012D9AC   0012F5A4
0012D9B0   00000000
0012D9B4   00000000
0012D9B8   00000000
0012D9BC   00000000
0012D9C0   00000000
0012D9C4   00000000
0012D9C8   00000000
0012D9CC   00000000
0012D9D0   00000000
0012D9D4   00000000
0012D9D8   00000000
0012D9DC   00000000
0012D9E0   00000000
0012D9E4   00000000
0012D9E8   00000000
0012D9EC   00000000
0012D9F0   00000000
0012D9F4   00000000
0012D9F8   00000000
0012D9FC   00000000
0012DA00   00000000
0012DA04   0012BDA4
0012DA08   00000000
0012DA0C   7C9305C8  RETURN to ntdll.7C9305C8 from ntdll.7C922F2F
0012DA10   005EC790  MLEx.005EC790
0012DA14   0012BE70
0012DA18   7C930551  RETURN to ntdll.7C930551 from ntdll.7C922F2F
0012DA1C   00561378  MLEx.00561378
0012DA20   7C93056D  RETURN to ntdll.7C93056D from ntdll.7C92EE02
0012DA24   0012BEC4
0012DA28   00000000
0012DA2C   00000000
0012DA30   00000000
0012DA34   00000000
0012DA38   00000020
0012DA3C   0012C368
0012DA40   0012BDEC
0012DA44   004A6C1C  MLEx.004A6C1C
0012DA48   0012C7FC
0012DA4C   0012BE20
0012DA50   00000039
0012DA54   004C1AB5  MLEx.004C1AB5
0012DA58   0012BE20
0012DA5C   0012BE08
0012DA60   004A6E3B  MLEx.004A6E3B
0012DA64   0012BE20
0012DA68   00000039
0012DA6C   0012C368
0012DA70   00000001
0012DA74   00000000
0012DA78   0012C340
0012DA7C   004A77D8  MLEx.004A77D8
0012DA80   0012BE20
0012DA84   828BF055
0012DA88   003C74B8
0012DA8C   003C74B8
0012DA90   5C3A4431
0012DA94   75636F44
0012DA98   00000020
0012DA9C   00401000  MLEx.00401000
0012DAA0   00000020
0012DAA4   003C74B8
0012DAA8   00001000
0012DAAC   003C74B8
0012DAB0  /0012DAE4
0012DAB4  |0056B12E  RETURN to MLEx.0056B12E from MLEx.0056B475
0012DAB8  |00000000
```

在 `0012DAB4` 处右键跟随：

![](img/5ee94b0bfac197705846b454c8df84f3.jpg)

在代码窗口中到这里：

```
0056B129   .  E8 47030000   CALL 0056B475                          ;  这个 CALL 处理解码
0056B12E   >  83C4 0C       ADD ESP,0C                               ;  跟随到这里
……
```

到这里后 `CTRL+F` 查找命令：`CALL 0056B475` 共会找到两处（如果 `ARM` 3.7 或以上版本，会有好几个的），第二个地方为：

```
0056B3E4   .  E8 8C000000   CALL 0056B475                          ;  第二个地方
0056B3E9   .  83C4 0C       ADD ESP,0C
0056B3EC   .  9C            PUSHFD
0056B3ED   .  60            PUSHAD
……
```

第二地方 `NOP` 掉就行了：

```
0056B3E4     90             NOP
0056B3E5     90             NOP
0056B3E6     90             NOP
0056B3E7     90             NOP
0056B3E8     90             NOP
0056B3E9   .  83C4 0C       ADD ESP,0C
```

处理完毕。

### NEXT1:

到这里后，修改一下代码就要 `DUMP` 了，让程序挂起来才能 `DUMP` 出来，用和 `SICE` 的方法一样改 `EIP` 为 `JMP EIP` 就行了。

修改 `003C6964`：

```
003C6964   /EB 10           JMP SHORT 003C6976
003C6966   |66:623A         BOUND DI,DWORD PTR DS:[EDX]
003C6969   |43              INC EBX
003C696A   |2B2B            SUB EBP,DWORD PTR DS:[EBX]
```

修改为：

```
003C6964  - EB FE           JMP SHORT 003C6964
003C6966    66:623A         BOUND DI,DWORD PTR DS:[EDX]
003C6969    43              INC EBX
003C696A    2B2B            SUB EBP,DWORD PTR DS:[EBX]
003C696C    48              DEC EAX
```

这然后让程序“挂起来”，打开 `LORDPE` 选择第二个进程，然后选择 `ARMDUMP` 的插件。选择好后就可以 `dump full` 程序。第二部分完毕。

## 第三部分：修复 IAT

代码已经 `DUMP` 下来了，我们现在要修复 `IAT`。关闭前面打开的 `OD`，然后用 `OD` 载入刚才 `DUMP` 的程序，在 `OEP` 附件找到一下 `IAT` 的地起位置。比较简单的找到的地址 `533000` (起始位置前一点不用担心搞错)。记得地址后，用 `OD` 再次载入目标，下断：`bp DebugActiveProcess`，运行后断下，就可以看到 `PROCESS ID` 了。到这里后，再开一个 `OD` 来附加这个 `PROCESS`。附加后中断在系统中，按 `F9` 运行，运行中按 `F12` 停止就可以到壳代码处，因为那里的代码是死循环的：－）

```
0056E379 >- EB FE           JMP SHORT <ModuleEntryPoint>     停在这里
0056E37B    EC              IN AL,DX                         ; I/O command
```

暂停后把 `EP` 代码改回去，改成 `558B` (`push ebp, mov ebp, esp`)，其实到了这里壳就像是一个双进程标准壳了，按双进程标准壳的方法就可以获取到 `IAT`。具体方法：

改回代码后直接下断 `BP OpenMutexA`，`F9` 运行后中断然后清除断点记下 `ESP+C` 的内容。

![](img/3b8bf1b7ce13abad50c91d88ef989831.jpg)

在一空白处写入以下代码：

```
00401000    60             PUSHAD
00401001    9C             PUSHFD
00401002    68 F0FB1200    PUSH 12FBF0                        ; ASCII "168::DAC4C6C5E3"
00401007    33C0           XOR EAX,EAX
00401009    50             PUSH EAX
0040100A    50             PUSH EAX
0040100B    E8 2FDB407C    CALL kernel32.CreateMutexA
00401010    9D             POPFD
00401011    61             POPAD
00401012  - E9 04DC407C    JMP kernel32.OpenMutexA
```

把 `EIP` 定位到代码处（`Ctrl+*`），写完代码后更改 `OD` 异常设置：打开内存访问错误异常。然后 `F9` 运行程序，内存异常断下后，忽略全部的异常并下断 `Bp GetModuleHandleA+5`，`Shift+F9` 忽略异常继续运行，`API` 断下后，取消断点，`ALT+F9` 执行返回到用户代码：

```
00CA532B    FF15 A450CC00   CALL DWORD PTR DS:[CC50A4]        ; kernel32.GetModuleHandleA
00CA5331    8B0D 60D8CC00   MOV ECX,DWORD PTR DS:[CCD860]     ; 返回到这里
00CA5337    89040E          MOV DWORD PTR DS:[ESI+ECX],EAX
00CA533A    A1 60D8CC00     MOV EAX,DWORD PTR DS:[CCD860]
00CA533F    393C06          CMP DWORD PTR DS:[ESI+EAX],EDI
00CA5342    75 16           JNZ SHORT 00CA535A
00CA5344    8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:[EBP-14C]
00CA534A    50             PUSH EAX
00CA534B    FF15 B850CC00   CALL DWORD PTR DS:[CC50B8]        ; kernel32.LoadLibraryA
00CA5351    8B0D 60D8CC00   MOV ECX,DWORD PTR DS:[CCD860]
00CA5357    89040E          MOV DWORD PTR DS:[ESI+ECX],EAX
00CA535A    A1 60D8CC00     MOV EAX,DWORD PTR DS:[CCD860]
00CA535F    393C06          CMP DWORD PTR DS:[ESI+EAX],EDI
00CA5362    0F84 AD000000   JE 00CA5415                        ; 这里 Magic jmp 改为 JMP
;改为 JMP 00CA5415
00CA5368    33C9            XOR ECX,ECX
```
