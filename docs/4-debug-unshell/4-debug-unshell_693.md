
# 用 forgot 的 pack v0.1.6 加壳的简单脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-487.htm`](https://www.pediy.com/kssd/pediy07/pediy7-487.htm)

用 `forgot` 的 `pack v0.1.6` 加壳的简单脱壳  

日期：2005 年 2 月 5 日  
脱壳人：`csjwaman[DFCG]`  

【软件名称】：`forgot` 的 `pack Me v0.1.6` 加壳的 `win98` 记事本  

【下载地址】：[`www.pediy.com/bbs/pediy7/pediy7-487.rar`](http://www.pediy.com/bbs/pediy7/pediy7-487.rar)  

【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！  

【操作系统】：`winxp`  

【脱壳工具】：`OD` 等传统工具  

## 【脱壳过程】

`forgot` 的 `pack v0.1.6` 加壳和 `015` 版加壳的区别就是多了入口代码的修改。但修改字节并不多，如果是 `delphi` 或 `vc/masm` 语言的只改了前 3 个字节，`VB` 的也只改了前一句 5 个字节。下面我用 `forgot` 的 `pack v0.1.6` 加壳的 `win98` 记事本(见附件)为例进行脱壳。  

如果看过我脱 `015` 版的文章，那么可以省略相关的章节。

### 第一步，找 d-process 处：

用 `OD` 载入程序，忽略所有异常，`bp WriteProcessMemory`，`F9` 运行程序，断下：

```
77E41A94  >  55                 PUSH EBP////断下。
77E41A95    8BEC               MOV EBP,ESP
77E41A97    51                 PUSH ECX
77E41A98    51                 PUSH ECX
77E41A99    8B45 0C            MOV EAX,DWORD PTR SS:[EBP+C]
77E41A9C    53                 PUSH EBX
```

观察堆栈数据：

```
00A18580   00A1BD98   /CALL 到 WriteProcessMemory 来自 00A1BD93
00A18584   00000034   |hProcess = 00000034
00A18588   00A1D492   |Address = A1D492////重要!记下备用。
00A1858C   00A1CE01   |Buffer = 00A1CE01
00A18590   0000029A   |BytesToWrite = 29A (666.)
00A18594   00000000   \pBytesWritten = NULL
```

好，`F9` 直接运行程序，等出现界面时，运行 `LordPE` 选中映像文件小的那个进程，然后部分 `DUMP`，把从 `A1D492` 处开始大小为 `29A` 的数据 `DUMP` 下来，命名为 `A1D492.bin`。

### 第二步，变双进程为单进程：

重新载入程序，`bp CreateProcessA`，运行程序，断下：

```
77E41BBC  >  55                 PUSH EBP////断在这儿。
77E41BBD    8BEC               MOV EBP,ESP
77E41BBF    6A 00              PUSH 0
77E41BC1    FF75 2C            PUSH DWORD PTR SS:[EBP+2C]
77E41BC4    FF75 28            PUSH DWORD PTR SS:[EBP+28]
77E41BC7    FF75 24            PUSH DWORD PTR SS:[EBP+24]
```

观察堆栈数据：

```
00A1856C   00A1AE1E   /CALL 到 CreateProcessA 来自 00A1AE19
00A18570   00A19B8B   |ModuleFileName = "C:\Documents and Settings\csjwaman\桌面\forgot-unpackme\VC++1.EXE"
00A18574   00A1AE0F   |CommandLine = "X"////调试标志。
00A18578   00000000   |pProcessSecurity = NULL
00A1857C   00000000   |pThreadSecurity = NULL
00A18580   00000000   |InheritHandles = FALSE
00A18584   00000003   |CreationFlags = DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS
00A18588   00000000   |pEnvironment = NULL
00A1858C   00000000   |CurrentDir = NULL
00A18590   00A188F8   |pStartupInfo = 00A188F8
00A18594   00A188E8   \pProcessInfo = 00A188E8
```

重新载入程序，`bp GetCommandLineA`，运行程序，断下：

```
77E5E358  >  A1 1476EB77      MOV EAX,DWORD PTR DS:[77EB7614]////断在这里。取消断点。
77E5E35D    C3               RETN////返回。
```

返回到：

```
00A18F78    68 9F6F56B6      PUSH B6566F9F
00A18F7D    50               PUSH EAX
00A18F7E    E8 5D000000      CALL 00A18FE0
00A18F83    EB FF          JMP SHORT 00A18F84
00A18F85    71 78          JNO SHORT 00A18FFF
00A18F87    C2 5000        RETN 50
00A18F8A   ^ EB D3          JMP SHORT 00A18F5F
00A18F8C    5B               POP EBX
```

搜索二进制字符“`803e58`”，找到：

```
00A19217  ^\ EB 80           JMP SHORT 00A19199/////找到这里。这里有个花指令。
00A19219    3E:58           POP EAX                              ; 多余的前缀
00A1921B    0F84 CA3F0000  JE 00A1D1EB
```

`NOP` 掉花指令后：

```
00A19217    90               NOP
00A19218    803E 58          CMP BYTE PTR DS:[ESI],58////58 为调试标志。用于判断是否为子进程。
00A1921B    0F84 CA3F0000  JE 00A1D1EB////此处改为 JMP!
00A19221    68 9F6F56B6      PUSH B6566F9F
00A19226    50               PUSH EAX
00A19227    E8 5D000000      CALL 00A19289
```

### 第三步，修补程序：

修改上面这个跳转后，用十六进制工具把 `A1D492.bin` 的数据复盖掉 `OD` 的 `DUMP` 区从 `0A1D492` 处开始的 `29A` 个字节数据。

### 第四步，查找入口：

做完上述工作后，在 `OD` 的 `CPU` 窗口，`Ctrl+G`,输入 `0A1D492`，点确定后来到：

```
00A1D48D  ^\ EB F0           JMP SHORT 00A1D47F////NOP 掉。
00A1D48F    0FC7C8           CMPXCHG8B EAX ////NOP 掉。
00A1D492    FFC3             INC EBX/////来到这里。
00A1D494    FFC8             DEC EAX
00A1D496    F7D8             NEG EAX
00A1D498    F7C1 2DF16825   TEST ECX,2568F12D
```

搜索二进制字符“`8D7802`”，找到：

```
00A1FC9A  ^\ EB 8D           JMP SHORT 00A1FC29////找到这里。有花指令。
00A1FC9C    78 02          JS SHORT 00A1FCA0
00A1FC9E    55               PUSH EBP
00A1FC9F    8BEC             MOV EBP,ESP
00A1FCA1    50               PUSH EAX
```

清除花指令后：

```
00A1FC9A    90               NOP
00A1FC9B    8D78 02          LEA EDI,DWORD PTR DS:[EAX+2]////注意这里!F2 下断。F9 运行程序后，断下时 EAX=4010CC。呵呵，光明就在眼前!
00A1FC9E    55               PUSH EBP////看看这句好象很熟悉噢：)
00A1FC9F    8BEC             MOV EBP,ESP////还有这句也是。
00A1FCA1    50               PUSH EAX/////注意这里!
00A1FCA2    E9 67010000      JMP 00A1FE0E////F8 到这里。
00A1FCA7    83FB 02          CMP EBX,2
00A1FCAA    0F85 5C010000   JNZ 00A1FE0C
```

跳到：

```
00A1FE0C    61               POPAD
00A1FE0D    50               PUSH EAX
00A1FE0E    C3               RETN////直接跳到这里，返回!
00A1FE0F    0100             ADD DWORD PTR DS:[EAX],EAX
00A1FE11    0000             ADD BYTE PTR DS:[EAX],AL
00A1FE13    0000             ADD BYTE PTR DS:[EAX],AL
00A1FE15    0000             ADD BYTE PTR DS:[EAX],AL
```

返回后：

```
004010CC    FFD7             CALL NEAR EDI////返回到这里，EDI=004010CE
004010CE    58               POP EAX////这句和 00A1FCA1 处的 PUSH EAX 相对应!
004010CF    83EC 44          SUB ESP,44
004010D2    56               PUSH ESI
004010D3    FF15 E4634000    CALL NEAR DWORD PTR DS:[4063E4]            ; kernel32.GetCommandLineA
004010D9    8BF0             MOV ESI,EAX
004010DB    8A00             MOV AL,BYTE PTR DS:[EAX]
004010DD    3C 22            CMP AL,22
004010DF    75 1B            JNZ SHORT VC++.004010FC
004010E1    56               PUSH ESI
004010E2    FF15 F4644000    CALL NEAR DWORD PTR DS:[4064F4]            ; USER32.CharNextA
004010E8    8BF0             MOV ESI,EAX
```

返回到 `004010CC` 时，我们可以确定入口到了，但入口代码被修改了。和原来的不同了。那么原来的代码呢？  

到 `00A1FC9E` 处看看吧:)好，把代码补回去。补回代码后：

```
004010CC    55               PUSH EBP////看起来顺眼了吧:)DUMP 吧。
004010CD    8BEC             MOV EBP,ESP
004010CF    83EC 44          SUB ESP,44
004010D2    56               PUSH ESI
004010D3    FF15 E4634000    CALL NEAR DWORD PTR DS:[4063E4]            ; kernel32.GetCommandLineA
004010D9    8BF0             MOV ESI,EAX
004010DB    8A00             MOV AL,BYTE PTR DS:[EAX]
```

`DUMP` 要注意纠正映像大小，然后删除 `SHIT` 区段，再用 `ImportREC v142+` 修复 `IAT`，优化一下，`388K--->56K` `OK` 完工!  

这里再补充说明一下 `VB` 程序代码的补回：  

到达入口后：

```
0040116C    FFD7             CALL NEAR EDI////显然代码被修改了。
0040116E    58               POP EAX
0040116F    90               NOP
00401170    90               NOP
00401171    E8 F0FFFFFF      CALL VB2.00401166         ; JMP to MSVBVM60.ThunRTMain
00401176    0000             ADD BYTE PTR DS:[EAX],AL
00401178    0000             ADD BYTE PTR DS:[EAX],AL
0040117A    0000             ADD BYTE PTR DS:[EAX],AL
0040117C    3000             XOR BYTE PTR DS:[EAX],AL
0040117E    0000             ADD BYTE PTR DS:[EAX],AL
00401180    3800             CMP BYTE PTR DS:[EAX],AL
```

看堆栈：

```
00A08964   00407C14   VB2.00407C14////这就是被修改掉的代码。
00A08968   00426828   返回到 VB2.00426828
```

补回后：

```
0040116C    68 147C4000      PUSH VB2.00407C14/////代码长度正好!
00401171    E8 F0FFFFFF      CALL VB2.00401166         ; JMP to MSVBVM60.ThunRTMain
00401176    0000             ADD BYTE PTR DS:[EAX],AL
00401178    0000             ADD BYTE PTR DS:[EAX],AL
0040117A    0000             ADD BYTE PTR DS:[EAX],AL
0040117C    3000             XOR BYTE PTR DS:[EAX],AL
0040117E    0000             ADD BYTE PTR DS:[EAX],AL
```

`4dd466d0e1e0cc123afbe504da2462ca`
