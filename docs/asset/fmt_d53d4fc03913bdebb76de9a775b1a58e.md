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
