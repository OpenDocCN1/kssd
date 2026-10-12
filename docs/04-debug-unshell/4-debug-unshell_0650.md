
# ExeCryptor 2.2.X 的 Anti Ollydbg 小结

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-673.htm`](https://www.pediy.com/kssd/pediy07/pediy7-673.htm)

## 简介

附件：`testdebugger.rar`

本来想等主程序的分析结束了再一起贴出这篇文章，但看到了 Softworm 兄弟的虚拟机分析，佩服得很。

美中不足的是兄弟没有描述壳对 `OD` 的 `Anti`，我这篇文章算是一个补充吧，希望能对大家有点用。

本文为了突出主题，用 `ExeCryptor 2.2.5` 加壳时用了 `Aggresive Anti Debug`，`Compress Code/Data`，`Virtualization 0%`，

好象生成的线程比主程序还要多几个。

没有使用 `Anti Trace`（多了一些异常），`Protect Entry Point`，`Delay Dll Loading`（`IAT` 加密），

也没有使用 `Kill RegMon/FileMon`（反正我也没用），`Active Watch`（保留一些线程不停检测调试器）。

### ExeCryptor 对 OD 的 Anti 列表

1.  `OEP` 处的 `CC` 检查
2.  `AntiDump`
3.  `IsDebuggerPresent`
4.  `[Heap+10h]`
5.  `PEB.NtGlobalFlag`
6.  `GetTickCount`
7.  `FindWindow`
8.  `OutputDebugString`
9.  `ReadProcessMemory(004B1C86, 004C91A0)`
10. 对 `API` 的检查很严格，几乎每一句都不能下 `CC` 断点，不能修改
11. 对 `Softice` 也有一些检查
12. 利用 `OpenProcess("CSRSS.EXE")` 来检测 `OD`

由于加密的选项关系，没有出现异常，所以你可以使用硬件断点。

如果是主程序，有很多异常，还会检查硬件断点，并清除，难度会大一些。

而且主程序中还有一个 `Anti`，好象和线程同步有关，我正分析中。

如果你知道的话，请提示一下。

## 一. TLS CallBack 中的飞刀

关于 `TLS CallBack` 的定义我就不详细描述了，如果你不太清楚，请参考 `MSDN` 和我以前的一篇文章。

先设置 `OD` 中断在系统断点，`F4` 到 `TLS CallBack Entry`。

```asm
0042CC4A    E8 EBFEFFFF     CALL TestDebu.0042CB3A        ; TLS Callback 入口, 先解密一段代码, F8 过
0042CC4F    05 7B070000     ADD EAX,77B
0042CC54    FFE0            JMP EAX
```

内存代码校验，`OD` 启动时会在 `OEP 0042CC3E` 处自动加一个一次性断点，`E8->CC`，

`Execryptor` 计算 `42CB40` 开始的 `116h` 字节校验和，第一把飞刀。

处理方法很简单，在系统断点断下后，去掉 `OEP` 处断点。

```asm
00429E1D    9D              POPFD
00429E1E    8B09            MOV ECX,DWORD PTR DS:[ECX]    ; [0042AD7C]=116h
00429E20    9C              PUSHFD
00428600    81C2 9D1DF602   ADD EDX,2F61D9D               ; EDX = F1F2F4F8 初始值
00428606    9D              POPFD
00428607    31C0            XOR EAX,EAX
0042A12E    AC              LODS BYTE PTR DS:[ESI]        ; 0042CB40
0042C614    01C2            ADD EDX,EAX
0042C616    C1C2 03         ROL EDX,3
0042C619    31C2            XOR EDX,EAX
0042C61B    49              DEC ECX
0042C61C  ^ 0F85 15F8FFFF   JNZ TestDebu.0042BE37          ; 注意这里不是循环出口.
0042BAF1    AC              LODS BYTE PTR DS:[ESI]        ; 在这里下个条件断点 ESI==0042CC3E, 可以看的很清楚
0042C516    01C2            ADD EDX,EAX
0042C518    C1C2 03         ROL EDX,3
00427949    31C2            XOR EDX,EAX
0042BC26    49              DEC ECX                       ; 116h->0
0042BC27    0F85 0A020000  JNZ TestDebu.0042BE37
0042BC2D    68 43C14200     PUSH TestDebu.0042C143        ; 循环出口, EDX = 227029ED, 计算的结果
0042B22F    8B00            MOV EAX,DWORD PTR DS:[EAX]    ; [0042AD88]=227029ED, 预先保留的结果
0042B231    29D0            SUB EAX,EDX
0042B26B   /0F84 8E0C0000   JE TestDebu.0042BEFF          ; 必须跳, 否则 Over
```

### AntiDump，第二把飞刀

```asm
0042C2BF    8CC8            MOV AX,CS
0042C2C1    30C0            XOR AL,AL
0042C2C3    09C0            OR EAX,EAX
0042C2C5  ^ 0F84 6BD6FFFF   JE TestDebu.00429936          ; 2K 下跳, 我的系统是 2KSP4, 98 下注意 FS:[20h]
0042B6A5    64:8B05 3000000>MOV EAX,DWORD PTR FS:[30]     ; PEB
...EAX+0Ch
00429216    8B00            MOV EAX,DWORD PTR DS:[EAX]
...EAX+0Ch
0042A00D    8B00            MOV EAX,DWORD PTR DS:[EAX]    ; 遍历用户模块列表
0042A010    68 F424D116     PUSH 16D124F4
0042A015    873C24          XCHG DWORD PTR SS:[ESP],EDI
0042A018    8BD7            MOV EDX,EDI
0042A01A    5F              POP EDI
0042A01B    C1C2 08         ROL EDX,8
00427744    81F2 CC3F1351   XOR EDX,51133FCC
0042AEB6    03D0            ADD EDX,EAX
0042AEB8    81C2 4634C87F   ADD EDX,7FC83446              ; 71EE0+20=71F00 (SizeOfImage)
0042AA4F    C702 00100000   MOV DWORD PTR DS:[EDX],1000   ; 花招
0042A296    64:FF35 0000000>PUSH DWORD PTR FS:[0]
0042AAA0    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
```

### IsDebuggerPresent，第三把飞刀

```asm
00429A1F    64:8B05 3000000>MOV EAX,DWORD PTR FS:[30]
004276D4    03C7            ADD EAX,EDI                   ; EDI=2
0042B767    8B00            MOV EAX,DWORD PTR DS:[EAX]
00429FF9    08C0            OR AL,AL
00429FFB  ^ 0F84 83F3FFFF   JE TestDebu.00429384          ; 必须跳, 否则 Over
```

利用 `ProcessHeap` 检测调试器，如果被调试 `[Heap+10h]` 处不为 0。`Hying` 的壳是 `[Heap+Ch]`，不知道是谁学谁啊？

```asm
00429592    64:8B05 1800000>MOV EAX,DWORD PTR FS:[18]     ; TEB
...EAX+30h
0042AB28    8B00            MOV EAX,DWORD PTR DS:[EAX]    ; PEB
...EAX+18h
004281FA    8B00            MOV EAX,DWORD PTR DS:[EAX]    ; ProcessHeap
00429527    8378 10 00      CMP DWORD PTR DS:[EAX+10],0   ; 必须为 0
0042952B    0F84 6C210000   JE TestDebu.0042B69D          ; 必须跳, 否则 Over
```

利用 `PEB.NtGlobalFlag` 检测调试器，如果被 `OD` 调试则 `NtGlobalFlag=70`，第五把飞刀没见过。

我用 `WinHex` 看了一下，应该是 0。

```asm
00428235    64:8B05 3000000>MOV EAX,DWORD PTR FS:[30]     ; PEB
0042823C    81C0 06DCBBA1   ADD EAX,A1BBDC06
00428242    81C0 6224445E   ADD EAX,5E442462              ; EAX+68h
00428248    8B00            MOV EAX,DWORD PTR DS:[EAX]    ; PEB.NtGlobalFlag
00428FF7    59              POP ECX
00428FF8    81C1 70CEFEFF   ADD ECX,FFFECE70              ; ECX=70h(被 OD 调试就是 70h)
00428FFE    9D              POPFD
00428FFF    85C1            TEST ECX,EAX
00427896   /0F84 57160000   JE TestDebu.00428EF3          ; 必须跳, 否则 Over

0042984D    68 409CB45F     PUSH 5FB49C40
00429852    59              POP ECX
00429853    C1C1 06         ROL ECX,6
0042B0D2    81F1 151027ED   XOR ECX,ED271015              ; ECX=2(表示什么?)
0042B0D8    85C1            TEST ECX,EAX
0042B0DB  ^\0F84 59EAFFFF   JE TestDebu.00429B3A          ; 必须跳, 否则 Over

00429B3A    64:8F05 0000000>POP DWORD PTR FS:[0]
00429B41    83C4 04         ADD ESP,4
```

## 循环解压代码

```asm
0042B37B    8B10            MOV EDX,DWORD PTR DS:[EAX]    ; [00417490]=00401000
00428659    09D2            OR EDX,EDX                    ; 目的地址
0042865B    0F84 38180000  JE TestDebu.00429E99          ; 循环出口 ************************************
00428661    52              PUSH EDX
00429FCD    57              PUSH EDI                      ; TestDebu.00400000
0042C26A    68 940D6E0E     PUSH 0E6E0D94
0042C26F    5F              POP EDI
0042C270    81CF A5C8A799   OR EDI,99A7C8A5
0042C276    81EF 03657F85   SUB EDI,857F6503
0042C27C    81E7 E1611956   AND EDI,561961E1
0042C282    C1C7 15         ROL EDI,15
0042C285    81C7 F8BDFDEB   ADD EDI,EBFDBDF8              ; EDI=4
00428282    03C7            ADD EAX,EDI                   ; 00417494
00428284    5F              POP EDI
...
00428139    871C24          XCHG DWORD PTR SS:[ESP],EBX
0042813C    C3              RETN                          ; 004274E3, 第一次, 第二次调用解压子程序在这里
0042C437    8B10            MOV EDX,DWORD PTR DS:[EAX]
...
0042C432    871424          XCHG DWORD PTR SS:[ESP],EDX
0042C435    C3              RETN                          ; 004274E3, 第三次调用解压子程序在这里
0042C437    8B10            MOV EDX,DWORD PTR DS:[EAX]
...
00429E99    68 40C34200     PUSH TestDebu.0042C340        ; 解压结束到这里
```

## 处理 IAT

```asm
0042C340
0042C345    90              NOP
0042C346    E8 19060000     CALL TestDebu.0042C964        ; 处理 IAT 的代码, F7 跟进, 由于我们没有加密 IAT, 所以 IAT 在这里处理
0042C34B
...
00429BD9    870424          XCHG DWORD PTR SS:[ESP],EAX
00429BDC    58              POP EAX
00429BDD    873C24          XCHG DWORD PTR SS:[ESP],EDI
00429BE0    FFD0            CALL EAX                      ; 004120E0, 跳到刚解开的最后一段
```

这一段的垃圾代码简直太恐怖了，大概就是所谓的虚拟技术了。

主要是创建了 4 个线程

...

```asm
0040573D    C1E0 02         SHL EAX,2
00405740    99              CDQ
00405741    030424          ADD EAX,DWORD PTR SS:[ESP]
00405744    135424 04       ADC EDX,DWORD PTR SS:[ESP+4]
00407DAC   /0F85 9A230000   JNZ TestDebu.0040A14C         ; 对 API Name 做 HASH 变换
```


` 块。

[content]
#### 代码片段分析

以下汇编代码片段展示了地址计算、API 查找及线程创建等过程。

```
00407DB2   |8B45 F8          MOV EAX,DWORD PTR SS:[EBP-8]  

00407DB5   |C1E0 03          SHL EAX,3  

0040C31D    0145 F8          ADD DWORD PTR SS:[EBP-8],EAX  

0040C320    8B45 F8          MOV EAX,DWORD PTR SS:[EBP-8]  

0040C323    C1E8 0B          SHR EAX,0B  

0040C326    3145 F8          XOR DWORD PTR SS:[EBP-8],EAX  

0040C329    E8 39F0FFFF      CALL TestDebu.0040B367  

004088AB   /0F85 0ABB0000    JNZ TestDebu.004143BB            ; 判断是否要找的 API             
004088B1   \\E9 24C3FFFF     JMP TestDebu.00404BDA            ; 找到了  
0040B7BE    8B36             MOV ESI,DWORD PTR DS:[ESI]  
0040B7C0    03C6             ADD EAX,ESI                       ; 计算 API 地址  

                       依次是 GetModuleHandleA, DebugBreak, ExitProcess,   可以下硬件断点 GetModuleHandleA 来到这里  

00403995    3B45 F8          CMP EAX,DWORD PTR SS:[EBP-8]  

00403998    0F85 3DDC0000    JNZ TestDebu.004115DB  

0040758D    81C6 B0A6C376    ADD ESI,76C3A6B0  

00407593    8B36             MOV ESI,DWORD PTR DS:[ESI]  

00407595    9D               POPFD  

00407596    03C6             ADD EAX,ESI  

00407598    E9 E29D0000      JMP TestDebu.0041137F  

004074A8    0345 FC          ADD EAX,DWORD PTR SS:[EBP-4]  

004074AB    8945 F4          MOV DWORD PTR SS:[EBP-C],EAX     ; 这里也计算 API 地址  

00405422    64:FF35 0000000>PUSH DWORD PTR FS:[0]             ; 计算出地址后, 先检查是否有 CC  
00405429    64:8925 0000000>MOV DWORD PTR FS:[0],ESP  

00405430    8B00             MOV EAX,DWORD PTR DS:[EAX]  

00405432    B8 01000000      MOV EAX,1  

00404A76    64:8F05 0000000>POP DWORD PTR FS:[0]  

00404A7D    83C4 04          ADD ESP,4  

0040DB90    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]     ; API Address  
0040DB93    8A00             MOV AL,BYTE PTR DS:[EAX]  

0040DB95    2C 99            SUB AL,99                         ; 检查断点 CC  
0040DB97    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]  

0040B2B2    F62A             IMUL BYTE PTR DS:[EDX]  

0040B2B4    3C A4            CMP AL,0A4                        ; (CC-99)* CC=28A4  
0040B2B6   ^ 0F85 A2F0FFFF    JNZ TestDebu.0040A35E            ; 必须跳 ******************************  

0040A367    E8 29030000      CALL TestDebu.0040A695  

                      依次是 InitializeCriticalSection, EnterCriticalSection, GlobalAlloc, GlobalLock, LeaveCriticalSection  

                       IsDebuggerPresent, CreateThread,  当计算出 CreateThread 后就开始创建线程了, 对 CreateThread 下一硬件断点  

一共创建了 4 个线程,  有 2 个地方调用 CreateThread  

0040CA38    8905 883C4000    MOV DWORD PTR DS:[403C88],EAX  

0040CA3E    C3               RETN                              ; Return to CreateThread(第一次)  
0040DB8E    5B               POP EBX  
0040DB8F    C3               RETN                              ; Return to 00414936  
00414936    C3               RETN                              ; Return to CreateThread(后三次)  
```

`CreateThread` 被断下后, `Stack`

```
    0006F854   0040AC52  /CALL to CreateThread from TestDebu.0040AC4D  
    0006F858   00000000  |pSecurity = NULL  
    0006F85C   00000000  |StackSize = 0  
    0006F860   00415374  |ThreadFunction = TestDebu.00415374  
    0006F864   00412D75  |pThreadParm = TestDebu.0040CE04            //00412D75, 0040F516, 00409175 后面还有三次, 每次只有 Parm 不一样  
    0006F868   00000000  |CreationFlags = 0                          // 先把这里改成 4 (suspend)  
    0006F86C   0006F870  \pThreadId = 0006F870  
```

第一次 `CreateThread` 后会回到 `004074AB` 处计算出 `CloseHandle` 地址,  对 `CloseHandle` 也下一硬件断点

执行 `CloseHandle(hThread)`  后返回下面

```
0041056F     870C24          XCHG DWORD PTR SS:[ESP],ECX  
00410572     8BC1            MOV EAX,ECX  
00410574     59              POP ECX  
00410575     31C0            XOR EAX,EAX  
00410577     C3              RETN  
```

...

第四次从 `CloseHandle` 返回后, `TLS CallBack` 就快结束了, `F7` 走一会就来到下面

```
00428700     8B00             MOV EAX,DWORD PTR DS:[EAX]  
00428702     59               POP ECX  
00428703     871C24           XCHG DWORD PTR SS:[ESP],EBX  
00428706     8BEB             MOV EBP,EBX  
00428708     5B               POP EBX  
00428709     C2 0C00          RETN 0C                             ; 进入系统领空, TLS CallBack 结束.  
```

#### 二. 在 壳的 `OEP` `0042CC3E` 处下一断点,  `F9`

这里的代码还会创建十一个线程,  用进程管理器可以看到一共有 16 个线程.

```
0042CC3E >   E8 F7FEFFFF      CALL TestDebu.0042CB3A           ; 壳的 OEP  
0042CC43     05 10150000      ADD EAX,1510  
0042CC48     FFE0             JMP EAX                           ; 00428A67  
```

...

```
004067EA     3B05 CCDC4000    CMP EAX,DWORD PTR DS:[40DCCC]    ; kernel32.ExitProcess  
```

...

```
00414936     C3               RETN                              ; return to CreateThread  
      0006FF9C   0040AC52  /CALL to CreateThread from TestDebu.0040AC4D  
      0006FFA0   00000000  |pSecurity = NULL  
      0006FFA4   00000000  |StackSize = 0   
      0006FFA8   00415374  |ThreadFunction = TestDebu.00415374  
      0006FFAC   00407916  |pThreadParm = TestDebu.00407916, 0040FF8A, 0040F180, 00403B5B, 00411DA9, 00413A8E, 0040F5B8, 00403AE0, 00415208, 0040330E, 0040D675  
      0006FFB0   00000000  |CreationFlags = 0  
      0006FFB4   0006FFB8  \pThreadId = 0006FFB8  
```

执行 `CloseHandle(hThread)` 返回下面

```
0041056F     870C24           XCHG DWORD PTR SS:[ESP],ECX  
00410572     8BC1             MOV EAX,ECX  
00410574     59               POP ECX  
00410575     31C0             XOR EAX,EAX  
00410577     C3               RETN                              ; return to 0040D675  
```

第三次后会回到 `004074AB` 处计算出 `IsDebuggerPresent` 地址, 并调用

第十一次后,

```
0040D675     84C0             TEST AL,AL  
0040D677   ^ 0F84 83C6FFFF    JE TestDebu.00409D00  
00409D00     C3               RETN                              ; return tp 00410a38  
00410A38   - E9 C305FFFF      JMP TestDebu.00401000             ; 这不到了 OEP 吗?    *************************   
00401000     FF15 58204000    CALL DWORD PTR DS:[402058]        ; 真正的 OEP, Dump, 修复 IAT 就可     
00401006     83F8 01          CMP EAX,1  
00401009     74 36            JE SHORT TestDebu.00401041  
0040100B     6A 00            PUSH 0  
0040100D     E8 05000000      CALL TestDebu.00401017  
```

一直到现在, 从线程都不需运行,  难道 `ExeCryptor` 的从线程除了 `Anti`, 不起其他作用?

#### 三. `ExeCryptor` 的从线程

当主程序的对话筐出来时, 从线程才会运行, 预先在 `00415374` 下断, 看看他还有那些飞刀

每生成一个从线程都 `Suspend`, 回到主线程,  等到所有从线程都生成完毕, 再一个一个激活.

`ThreadParm=0040CE04` 的从线程 `id= 3c8`,  这是其中最复杂的一个, 里面还会生成新线程

```
00415374     55               PUSH EBP  
00415375     8BEC             MOV EBP,ESP  
00415377     53               PUSH EBX  
```

...

```
004074A8     0345 FC          ADD EAX,DWORD PTR SS:[EBP-4]  
004074AB     8945 F4          MOV DWORD PTR SS:[EBP-C],EAX    ; 这里下个断点  
```

计算出 `GetCurrentThread`,  `SetThreadPriority`

```
    0076FFA8   00404BF3  /CALL to GetCurrentThread from TestDebu.00404BEE  
    0076FFA4   0040AB84  /CALL to SetThreadPriority                 ; 设置自己为 IDLE, 看来真的没有其他作用  
    0076FFA8   FFFFFFFE  |hThread = FFFFFFFE  
    0076FFAC   FFFFFFFF  \Priority = THREAD_PRIORITY_IDLE  
```

`0040AB84`

```
0040AB84     8B55 08          MOV EDX,DWORD PTR SS:[EBP+8]           ; TestDebu.0040CE04  
0040AB87     B0 01            MOV AL,1  
0040AB89     FFD2             CALL EDX                                 ; TestDebu.0040CE04,  原来 ThreadParm 是线程真正的入口, 前面大家都是一样的  
```

...

```
004133C3     870C24           XCHG DWORD PTR SS:[ESP],ECX  
004133C6     C3               RETN                                      ; Return to EnterCriticalSection  
00403794     C3               RETN                                      ; Return to GetModuleHandleA  
```

...

`0040A905`

```
0040A905     A3 B45A4100      MOV DWORD PTR DS:[415AB4],EAX            ; user32.77E10000  
```

接着计算出

`FindWindowA`, `GetTickCount`, `GetCurrentProcessId`, `EnumWindows`, `GetWindowThreadProcessId`, `OpenProcess`,  `ReadProcessMemory`,

```
0076FF78   004047B6  /CALL to FindWindowA  
0076FF7C   0076FF84  |ClassName = "OLLYDBG"  
0076FF80   00000000  \Title = NULL  
```



```
00403122    837D F8 00       CMP DWORD PTR SS:[EBP-8],0  

00403126    0F84 11000100   JE TestDebu.0041313D                   ; 必须跳  

0076FF78   0040F7A8  /CALL to FindWindowA  

0076FF7C   0040F848  |Class = "WispWindowClass"                    ; 什么东东?  

0076FF80   00000000  \Title = NULL  

0076FF74   0040F9F6  /CALL to GetTickCount from TestDebu.0040F9F1  

004091B5    8901           MOV DWORD PTR DS:[ECX],EAX              ; [406444]  

004039C6    FF45 FC        INC DWORD PTR SS:[EBP-4]  

004039C9    817D FC 0001000>CMP DWORD PTR SS:[EBP-4],100           ; 拖延一段时间  

004039D0    0F85 E7570000  JNZ TestDebu.004091BD  

0076FF74   00407136  /CALL to GetTickCount from TestDebu.00407131  

00407136    2B05 44644000  SUB EAX,DWORD PTR DS:[406444]  

0040713C    C1E8 11        SHR EAX,11  

0040713F    0F84 54CB0000  JE TestDebu.00413C99                   ; 必须跳  

0076FF80   0040BBA1  /CALL to GetCurrentProcessId from TestDebu.0040BB9C  

004098F1    8907           MOV DWORD PTR DS:[EDI],EAX  
```

用 `GetCurrentProcessID` 得到自己的 PID

接着 `EnumWindows(WndEnumProc=004099E0)`

```
    0076FF78   00413FD6  /CALL to EnumWindows from TestDebu.00413FD1  

    0076FF7C   004099E0  |Callback = TestDebu.004099E0  

    0076FF80   00000000  \lParam = 0  
```

`WndEnumProc`

```
004099E0    55           PUSH EBP  

004099E1    E9 F66F0000  JMP TestDebu.004109DC  
```

接着计算出

`GetWindowThreadProcessId`, `OpenProcess`, `ReadProcessMemory`,

调用 `GetWindowThreadProcessId`

```
      0076FC64   004158A2  /CALL to GetWindowThreadProcessId from TestDebu.0041589D  

      0076FC68   000303BC  |hWnd = 000303BC (class='tooltips_class32',parent=000802AE)  

      0076FC6C   0076FF2C  \pProcessID = 0076FF2C  

004158A2    8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]            ; EAX 是 这次 Enum 到的 PID  

004158A5    3B05 B4DC4000 CMP EAX,DWORD PTR DS:[40DCB4]           ; [0040DCB4] 放着 程序自己的 PID  

004158AB  ^ 0F84 CA57FFFF  JE TestDebu.0040B07B                   ; 是自己吗?  

004033FC    3B05 C8DC4000 CMP EAX,DWORD PTR DS:[40DCC8]           ; 是上次 Enum 到的 PID 吗?  

00403402    0F84 737C0000  JE TestDebu.0040B07B  

0040BB65    A3 C8DC4000    MOV DWORD PTR DS:[40DCC8],EAX          ; 保留这次 Enum 到的 PID, 下次用  
```

调用 `OpenProcess( Enum 到的 PID)`

```
0076FC60   0040A5BE  /CALL to OpenProcess from TestDebu.0040A5B9  

0076FC64   00000010  |Access = VM_READ  

0076FC68   00000000  |Inheritable = FALSE  

0076FC6C   000003BC  \ProcessId = 3BC  
```

调用 `ReadProcessMemory`

```
0076FC58   00403C0F  /CALL to ReadProcessMemory from TestDebu.00403C0A  

0076FC5C   00000054  |hProcess = 00000054 (window)  

0076FC60   00400000  |pBaseAddress = 400000                      ; 读文件头?  

0076FC64   0076FED4  |Buffer = 0076FED4  

0076FC68   00000040  |BytesToRead = 40 (64.)  

0076FC6C   0076FF14  \pBytesRead = 0076FF14  

00403C0F    85C0        TEST EAX,EAX                              ; 读不成功, 直接跳过  

00403C11    0F84 98090000 JE TestDebu.004045AF  

00403C17    E9 AC100100  JMP TestDebu.00414CC8  

0076FC58   00410523  /CALL to ReadProcessMemory from TestDebu.0041051E  

0076FC5C   00000054  |hProcess = 00000054 (window)  

0076FC60   004B1C86  |pBaseAddress = 4B1C86                      ; ( OD 4B1C86 是 "EBX+", 改成 "ebx+" 再看看?)  

0076FC64   0076FD70  |Buffer = 0076FD70  

0076FC68   00000004  |BytesToRead = 4  

0076FC6C   0076FF14  \pBytesRead = 0076FF14  

00410523    85C0        TEST EAX,EAX  

00410525    0F84 365A0000 JE TestDebu.00415F61                  ; 读不成功, 直接跳过  

0041052B  ^\E9 9EB0FFFF    JMP TestDebu.0040B5CE  

00412E16    81BD 38FEFFFF 4>CMP DWORD PTR SS:[EBP-1C8],2B584245   ; 读到的是 "EBX+" ?  

00412E20   /0F85 3B310000 JNZ TestDebu.00415F61                 ; 必须跳 *******  

00412E26  ^|E9 0136FFFF    JMP TestDebu.0040642C  

0076FC58   00414B31  /CALL to ReadProcessMemory from TestDebu.00414B2C  

0076FC5C   00000054  |hProcess = 00000054 (window)  

0076FC60   004C91A0  |pBaseAddress = 4C91A0                      ; ( OD 4C91A0 是什么, 各位自己去看吧, 把这里改了, OD 的插件就不能用了)  

0076FC64   0076FC70  |Buffer = 0076FC70                          

0076FC68   00000100  |BytesToRead = 100 (256.)  

0076FC6C   0076FF14  \pBytesRead = 0076FF14  

00407E43    85C0        TEST EAX,EAX  

00407E45  ^ 0F84 6EB4FFFF  JE TestDebu.004032B9  

00407E4B  ^ E9 0AC3FFFF    JMP TestDebu.0040415A  

00409069    8138 54444247  CMP DWORD PTR DS:[EAX],47424454       ; "ODBG"  

0040906F   /0F84 817A0000  JE TestDebu.00410AF6                 ; 不能跳 ********  

00409075   |E9 C61B0000   JMP TestDebu.0040AC40  
```

接着调用 `CloseHandle(hProcess)`

```
0040B07B    C745 F8 0100000>MOV DWORD PTR SS:[EBP-8],1  

0040B082  ^ E9 1CDAFFFF     JMP TestDebu.00408AA3  
```

如果检测出 OD 则马上计算 `SendMessageA`, 关掉 OD, 自己也会退出.

```
0076FC5C   0040D5BA  /CALL to SendMessageA  

0076FC60   0007016A  |hWnd = 7016A                               ; OD 的主窗口  

0076FC64   00000010  |Message = WM_CLOSE  

0076FC68   00000000  |wParam = 0  

0076FC6C   00000000  \lParam = 0  
```

比较结束后, 退出 `EnumWindow`, 回到下面

```
00413FD6    E8 D3E3FFFF    CALL TestDebu.004123AE  

00413FDB    84C0        TEST AL,AL  

00413FDD  ^ E9 41F8FEFF    JMP TestDebu.00403823  
```

还会继续 `CreateThread` 两次

```
0076FF60   0040AC52  R.@.  /CALL to CreateThread from TestDebu.0040AC4D  

0076FF64   00000000  ....  |pSecurity = NULL  

0076FF68   00000000  ....  |StackSize = 0  

0076FF6C   00415374  tSA.  |ThreadFunction = TestDebu.00415374  

0076FF70   00413FDB  .?A.  |pThreadParm = TestDebu.00413FDB, 0040611A  

0076FF74   00000000  ....  |CreationFlags = 0  

0076FF78   0076FF7C  .v.  \pThreadId = 0076FF7C  
```

创建好后线程自己就结束了.

`CloseHandle` 后...

```
0040977A    8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

0040977D    59         POP ECX  

0040977E    5D         POP EBP  

0040977F    C2 0400    RETN 4  
```

其中 `00413FDB` 的线程比较简单

```
004074AB    8945 F4       MOV DWORD PTR SS:[EBP-C],EAX            ; kernel32.OutputDebugStringA  

0090FF2C   0040F53D  /CALL to OutputDebugStringA from TestDebu.0040F53A  

0090FF30   0090FF34  \String = "%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s"  

0040F53D    E8 73660000    CALL TestDebu.00415BB5  

0040977A    8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

0040977D    59         POP ECX  

0040977E    5D         POP EBP  

0040977F    C2 0400    RETN 4                                    ; 线程就结束了.  
```

`0040611A` 的线程 `id = 404` 要复杂

```
0040611A    84C0        TEST AL,AL  

0040611C    0F84 DCDD0000  JE TestDebu.00413EFE  

00414A54    8CC8        MOV AX,CS  

00414A56    30C0        XOR AL,AL  

00414A58    09C0        OR EAX,EAX  

00414A5A    E9 2C130000  JMP TestDebu.00415D8B  

004074AB    8945 F4       MOV DWORD PTR SS:[EBP-C],EAX            

; kernel32.CreateToolhelp32Snapshot, Process32First, Process32Next  

008EFE50   00405140  @Q@.  /CALL to CreateToolhelp32Snapshot from TestDebu.0040513D  

008EFE54   00000002  ....  |Flags = TH32CS_SNAPPROCESS  

008EFE58   00000000  ....  \ProcessID = 0  

00405140    8945 EC       MOV DWORD PTR SS:[EBP-14],EAX       ; handle=58h  

00405143    C785 BCFEFFFF 2>MOV DWORD PTR SS:[EBP-144],128  

0040514D    68 D0E84000    PUSH TestDebu.0040E8D0  

008EFE50   0040F111  ..@.  /CALL to Process32First from TestDebu.0040F10E  

008EFE54   00000058  X...  |hSnapshot = 00000058 (window)  

008EFE58   008EFE5C  \...  \pProcessentry = 008EFE5C  

0040F111    F7D8        NEG EAX  

0040F113    68 57304000  PUSH TestDebu.00403057  

0040364B    8B00        MOV EAX,DWORD PTR DS:[EAX]  

0040364D    8038 63      CMP BYTE PTR DS:[EAX],63            ; 进程名的第一字母 "c"  

0040BB3A    8B00        MOV EAX,DWORD PTR DS:[EAX]  
```



# 0040BB3C

`0040BB3C    8038 43           CMP BYTE PTR DS:[EAX],43            ; "C"`

`00411602    C3               RETN`

`008EFE50   00415D77  w]A.   /CALL to Process32Next`
`008EFE54   00000058  X...   \hSnapshot = 00000058 (window)`
`008EFE58   008EFE5C  \\...  \\pProcessentry = 008EFE5C`

`0040390D    FF45 E4         INC DWORD PTR SS:[EBP-1C]            ; 找到了, 再去比较下一字符`
`00408421    8038 73         CMP BYTE PTR DS:[EAX],73             ; "s"`
`00408424    0F84 2BA70000   JE TestDebu.00412B55`
`00403AEF    8038 53         CMP BYTE PTR DS:[EAX],53             ; "S", 到这里我们就明白了, 是 "CSRSS.EXE"`

...

找到了就跳出循环, 关闭 `Handle`

`008EFE54   0041270A  .'A.   /CALL to CloseHandle from TestDebu.00412705`
`008EFE58   00000058  X...   \hObject = 00000058 (window)`

`0041270A  ^\\E9 F80EFFFF     JMP TestDebu.00403607`

`0040D19F    64:FF35 0000000>PUSH DWORD PTR FS:[0]`
`0040D1A6    64:8925 0000000>MOV DWORD PTR FS:[0],ESP`
`0040D1AD    64:8B05 3000000>MOV EAX,DWORD PTR FS:[30]`
`0040D1B4    8B40 68         MOV EAX,DWORD PTR DS:[EAX+68]       ; PEB.NTGlobalFlag`
`0041417C    85C3            TEST EBX,EAX                         ; EBX = 30000, 表示什么? 正常情况下 NTGlobalFlag=0`
`0041417F  ^\\0F84 6594FFFF   JE TestDebu.0040D5EA                 ; 跳`
`00414185  ^ E9 3B86FFFF     JMP TestDebu.0040C7C5`

`0040C7C5  ... 不跳的话直接到这里, 安全过关`

`0040D5E2    31C0            XOR EAX,EAX`
`0040D5E4    8905 643C4000   MOV DWORD PTR DS:[403C64],EAX`
`0040D5EA    64:8F05 0000000>POP DWORD PTR FS:[0]`
`0040D5F1    83C4 04         ADD ESP,4`
`0040D5F4    8BE5            MOV ESP,EBP`
`0040D5F6    E9 85290000     JMP TestDebu.0040FF80`

`0040977A    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]        ; 结束线程`
`0040977D    59              POP ECX`
`0040977E    5D              POP EBP`
`0040977F    C2 0400         RETN 4`

==========================================================================================================================================

跳的话

`0040D5EA    64:8F05 0000000>POP DWORD PTR FS:[0]                 ; 008EFFDC`
`0040D5F1    83C4 04         ADD ESP,4`

`008EFF98   00408912  ..@.   /CALL to OpenProcess from TestDebu.0040890D`
`008EFF9C   00000C3A  :...   |Access = CREATE_THREAD|VM_OPERATION|VM_READ|VM_WRITE|QUERY_INFORMATION|800`
`008EFFA0   00000000  ....   |Inheritable = FALSE`
`008EFFA4   000000FC  ....   \\ProcessId = FC`

`00408912    85C0            TEST EAX,EAX`
`00408914    0F84 D4B70000   JE TestDebu.004140EE 这里必须跳, 也能过关.`

如果不跳, 又开始创建线程了, 先看看他创建的线程, 死定了.

`id = 11C`

`0076FF60   0040AC52  R.@.   /CALL to CreateThread from TestDebu.0040AC4D`
`0076FF64   00000000  ....   |pSecurity = NULL`
`0076FF68   00000000  ....   |StackSize = 0`
`0076FF6C   00415374  tSA.   |ThreadFunction = TestDebu.00415374`
`0076FF70   00413FDB  .?A.   |pThreadParm = TestDebu.0040DAB6`
`0076FF74   00000000  ....   |CreationFlags = 0`
`0076FF78   0076FF7C  |.v.   \\pThreadId = 0076FF7C`

`0040DAB6    84C0            TEST AL,AL`
`0040DAB8  ^ 0F84 06D0FFFF   JE TestDebu.0040AAC4`

`008EFF98   7C59A25A  /CALL to SleepEx from kernel32.7C59A255,`
`008EFF9C   00007530  |Timeout = 30000. ms  ( 暂停 30S)`
`008EFFA0   00000000  \\Alertable = FALSE`
`008EFFA4   0040AABD  TestDebu.0040AABD`
`008EFFA8   00007530`

暂停结束后

`0040AABD    6A 00           PUSH 0`
`0040AABF    E8 59790000     CALL TestDebu.0041241D`
`0040AAC4    C3              RETN`

直接就走向了 `ExitProcess.`

回到 `404` 线程, 又创建了一个

`id = 378, handle = 54`

`008DFF64   00409B61  R.@.   /CALL to CreateThread`
`008DFF68   00000000  ....   |pSecurity = NULL`
`008DFF6C   00000000  ....   |StackSize = 0`
`008DFF70   0041493C  tSA.   |ThreadFunction = TestDebu.0041493c`
`008DFF74   008DFF84  .?A.   |pThreadParm = TestDebu.8DFF84`
`008DFF78   00000000  ....   |CreationFlags = 0`
`008DFF7C   008DFF80  |.v.   \\pThreadId = 008DFF80`

```
008DFF80  D8 03 00 00
           01 01 00 00
           1A 61 40 00
           4C FC 7E 00
008DFF90  40 C8 40 00
           A0 FF 8D 00
           A1 3C 41 00
           10 01 00 00
```

先到 `378` 这个线程去看看

`0041493C    56              PUSH ESI`
`0041493D    8BF5            MOV ESI,EBP`
`0041493F    873424          XCHG DWORD PTR SS:[ESP],ESI`
`00414942    8BEC            MOV EBP,ESP`
`004105FF    8B45 08         MOV EAX,DWORD PTR SS:[EBP+8]`
`00410602    8B08            MOV ECX,DWORD PTR DS:[EAX]            ; ECX = 101h`
`00410604    83C0 04         ADD EAX,4`
`00410607    8B18            MOV EBX,DWORD PTR DS:[EAX]            ; EBX = 0040611A`
`00411339    83C0 04         ADD EAX,4`
`0041133C    8B38            MOV EDI,DWORD PTR DS:[EAX]            ; EDI = 007EFC4C`
`00410F7B    03C2            ADD EAX,EDX                           ; + 4`
`00410F7E    8B30            MOV ESI,DWORD PTR DS:[EAX]            ; ESI = 0040C840`

`EAX + 4`

`0040FFF8    8B28            MOV EBP,DWORD PTR DS:[EAX]            ; EBP = 008DFFA0`
`0040FFFA    83C0 04         ADD EAX,4`
`0040FFFD    8B10            MOV EDX,DWORD PTR DS:[EAX]            ; EDX = 00413CA1`
`0040FFFF    B0 01           MOV AL,1`
`00410001    FFD2            CALL EDX                              ; TestDebu.00413CA1`
`0040911D    8B00            MOV EAX,DWORD PTR DS:[EAX]            ; EAX = 110`
`0040381B    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]`
`0040381E    59              POP ECX`
`0040381F    5D              POP EBP`
`00403820    C2 0400         RETN 4                                ; 线程结束, 干了什么?`

回到 `id=404` 这个线程

`004074AB    8945 F4         MOV DWORD PTR SS:[EBP-C],EAX          ; kernel32.WaitForSingleObject, 等待 378 线程的结束`
`008EFF74   0040CB55  U.@.   /CALL to WaitForSingleObject`
`008EFF78   0000005C  \\...   \hObject = 00000054 (window)`
`008EFF7C   FFFFFFFF  ....   \Timeout = INFINITE`
`0040CB55    E8 E1D7FFFF     CALL TestDebu.0040A33B                ; 这里下个断点`
`008EFF7C   0040CB5A  Z.@.   /CALL to CloseHandle from TestDebu.0040CB55`
`008EFF80   00000378  x...   \hObject = 00000378                   ;  378 是 ThreadID, 不是句柄,  产生异常, 按 F9 还是 Shift+F9 都是死`
`0040CB5A    83C4 10         ADD ESP,10`
`0040CB5D    68 28A34000     PUSH TestDebu.0040A328`

下面就完蛋了.

`00414148    84C0            TEST AL,AL`
`0041414A  ^ 0F84 4D8BFFFF   JE TestDebu.0040CC9D`
`00414150  ^ E9 9FBBFFFF     JMP TestDebu.0040FCF4`
`008EFF94   0040BF2C  ,.@.   /CALL to ExitProcess from TestDebu.0040BF27`
`008EFF98   00000110  ....   \ExitCode = 110`

====================================================================================================================================================================

# 解压子程序

`004274E3    56              PUSH ESI`
`004274E4    57              PUSH EDI`
`004274E5    53              PUSH EBX`
`004274E6    31DB            XOR EBX,EBX`
`004274E8    89C6            MOV ESI,EAX`
`004274EA    89D7            MOV EDI,EDX`
`004274EC    0FB606          MOVZX EAX,BYTE PTR DS:[ESI]`
`004274EF    89C2            MOV EDX,EAX`
`004274F1    83E0 1F         AND EAX,1F`
`004274F4    C1EA 05         SHR EDX,5`
`004274F7    74 2D           JE SHORT TestDebu.00427526`
`004274F9    4A              DEC EDX`
`004274FA    74 15           JE SHORT TestDebu.00427511`
`004274FC    8D5C13 02       LEA EBX,DWORD PTR DS:[EBX+EDX+2]`
`00427500    46              INC ESI`
`00427501    C1E0 08         SHL EAX,8`
`00427504    89FA            MOV EDX,EDI`
`00427506    0FB60E          MOVZX ECX,BYTE PTR DS:[ESI]`
`00427509    46              INC ESI`
`0042750A    29CA            SUB EDX,ECX`
`0042750C    4A              DEC EDX`
`0042750D    29C2            SUB EDX,EAX`
`0042750F    EB 32           JMP SHORT TestDebu.00427543`
`00427511    C1E3 05         SHL EBX,5`
`00427514    8D5C03 04       LEA EBX,DWORD PTR DS:[EBX+EAX+4]`
`00427518    46              INC ESI`
`00427519    89FA            MOV EDX,EDI`
`0042751B    0FB70E          MOVZX ECX,WORD PTR DS:[ESI]`



### 处理 IAT 的子程序

```
0042C964    55             PUSH EBP
0042C965    8BEC           MOV EBP,ESP
0042C967    83C4 F4        ADD ESP,-0C
0042C96A    56             PUSH ESI
0042C96B    57             PUSH EDI
0042C96C    53             PUSH EBX
0042C96D    BE 00204000    MOV ESI,TestDebu.00402000
0042C972    B8 00004000    MOV EAX,TestDebu.00400000
0042C977    8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
0042C97A    89C2           MOV EDX,EAX
0042C97C    8B46 0C        MOV EAX,DWORD PTR DS:[ESI+C]
0042C97F    09C0           OR EAX,EAX
0042C981    0F84 8E000000  JE TestDebu.0042CA15
0042C987    01D0           ADD EAX,EDX
0042C989    89C3           MOV EBX,EAX
0042C98B    50             PUSH EAX
0042C98C    FF15 B4704100  CALL DWORD PTR DS:[<&kernel32.GetModuleH>; kernel32.GetModuleHandleA
0042C992    09C0           OR EAX,EAX
0042C994    0F85 0F000000  JNZ TestDebu.0042C9A9
0042C99A    53             PUSH EBX
0042C99B    FF15 B8704100  CALL DWORD PTR DS:[<&kernel32.LoadLibrar>; kernel32.LoadLibraryA
0042C9A1    09C0           OR EAX,EAX
0042C9A3    0F84 64000000  JE TestDebu.0042CA0D
0042C9A9    8945 F8        MOV DWORD PTR SS:[EBP-8],EAX          ; kernel32.7C570000
0042C9AC    6A 00          PUSH 0
0042C9AE    8F45 F4        POP DWORD PTR SS:[EBP-C]
0042C9B1    8B06           MOV EAX,DWORD PTR DS:[ESI]
0042C9B3    09C0           OR EAX,EAX
0042C9B5    8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
0042C9B8    0F85 03000000  JNZ TestDebu.0042C9C1
0042C9BE    8B46 10        MOV EAX,DWORD PTR DS:[ESI+10]
0042C9C1    01D0           ADD EAX,EDX
0042C9C3    0345 F4        ADD EAX,DWORD PTR SS:[EBP-C]
0042C9C6    8B18           MOV EBX,DWORD PTR DS:[EAX]
0042C9C8    8B7E 10        MOV EDI,DWORD PTR DS:[ESI+10]
0042C9CB    01D7           ADD EDI,EDX
0042C9CD    037D F4        ADD EDI,DWORD PTR SS:[EBP-C]
0042C9D0    09DB           OR EBX,EBX
0042C9D2    0F84 E4000000  JE TestDebu.0042CABC
0042C9D8    F7C3 00000080  TEST EBX,80000000
0042C9DE    0F85 04000000  JNZ TestDebu.0042C9E8
0042C9E4    8D5C13 02      LEA EBX,DWORD PTR DS:[EBX+EDX+2]
0042C9E8    81E3 FFFFFFFF  AND EBX,7FFFFFFF
0042C9EE    53             PUSH EBX
0042C9EF    FF75 F8        PUSH DWORD PTR SS:[EBP-8]
0042C9F2    FF15 BC704100  CALL DWORD PTR DS:[<&kernel32.GetProcAdd>; kernel32.GetProcAddress
0042C9F8    09C0           OR EAX,EAX
0042C9FA    0F84 0D000000  JE TestDebu.0042CA0D
0042CA00    8907           MOV DWORD PTR DS:[EDI],EAX
0042CA02    8345 F4 04     ADD DWORD PTR SS:[EBP-C],4
0042CA06    ^ E9 A6FFFFFF   JMP TestDebu.0042C9B1
0042CA0B    0000           ADD BYTE PTR DS:[EAX],AL
0042CA0D    53             PUSH EBX
0042CA0E    89D8           MOV EAX,EBX
0042CA10    E8 09000000    CALL TestDebu.0042CA1E            ; ExitProcess, 不能到这里
0042CA15    5B             POP EBX
0042CA16    5F             POP EDI
0042CA17    5E             POP ESI
0042CA18    8BE5           MOV ESP,EBP
0042CA1A    5D             POP EBP
0042CA1B    C3             RETN
```

### 调试心得

太好了！下回去好好学习！

我的 `OD` 改得千疮百孔，`ring3`、`ring0` 都使上了，忍受着时常蓝屏的痛苦，仍然不能抵挡检测，很是郁闷。我没有跟 `thread`，是用 `SoftIce` 和 `OD` 一起上的（因为不习惯用 `SoftIce` 脱壳）。

开始写了个程序注入代码，发现干掉所有的线程程序仍可正常运行。

在 `SofiIce` 下个断点：

```
:bpmb CreateRemoteThread x if(*(esp+10)<700000) do " r eip xxxxxxxx;x"
```

`xxxxxxxx` 是 `CreateRemoteThread` 尾巴的地址。这样搞在 `OD` 内要处理一下 `Closehandle` 无效 `handle` 的问题。

这种偷懒取巧的办法，没好意思在文章里写出来。佩服 `simonzh`，有时间把 `OD` 再补一补！[](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg) 。

### 补充说明

补充一下，壳用 `OpenProcess("CSRSS.EXE")` 来检测 `OD`。

如果 不成功说明 没有 `OD`

如果 成功说明 有 `OD`，则 `Over`。

`4dd466d0e1e0cc123afbe504da2462ca`
