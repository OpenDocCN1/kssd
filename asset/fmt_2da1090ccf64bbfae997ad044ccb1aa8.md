
# OllyDBG 分析报告系列(6)---OD 异常处理机制

> 原文：[`www.pediy.com/kssd/pediy10/65686.html`](https://www.pediy.com/kssd/pediy10/65686.html)

这是分析报告的第六部分，请多多指教。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

`Ollydbg`（以下均简称为 `OD`）中对各类断点、各类热键以及菜单消息的处理都是通过一个不停循环的消息线程处理的，当然对于调试程序异常的处理也包括在里面。

在 `OD` 中该循环位于 `WinMain` 函数的中间部分，先说一下其流程，然后具体分析代码：

1.  得到开机到现在的时间，根据时间间隔更新日志文件，流程如下：
    *   a) 检查是否创建了记录 `OD` 调试信息日志文件，若有创建继续检查距离上次更新是否超过 30 秒，若超过则将调试信息新增部分写入到文件中；
    *   b) 检查是否创建了运行跟踪结果的日志文件，若有创建继续检查距离上次更新是否超过 30 秒，若超过则将运行跟踪结果新增部分写入到文件中；
    *   c) 保存最后一次更新时间；
2.  检查是否需要刷新反汇编窗口、主窗口，若需要则检查刷新间隔是否超过 0.5 秒，超过则刷新，并记录该次刷新的时间，否则就不刷新；
3.  接受线程的消息队列，并将消息信息放入 `MSG` 结构体中，并检查消息是否接受成功，若失败跳过第 4 步对消息的处理；
4.  转换子窗体中的快捷键消息，若转换成功则跳过对其他消息的处理；
5.  ……一般消息的判断以及消息收发，这里跳过，没有分析；
6.  对 `EXE`、`DLL` 做相应的检查以及设置（添加临时断点、添加插件栏）；
7.  使用 `WaitForDebugEvent` 函数接收异常，若没有接收到则发送 `NULL` 消息，跳转到第 20 步；
8.  先让插件处理异常，然后检查接收到的异常信息中进程 `Id` 是否为调试进程的 `Id`，异常码是否为 `EXCEPTION_DEBUG_EVENT`，若不是则调用 `ContinueDebugEvent` 继续调试程序，其 `ContinueStatus` 参数设置为 `DBG_CONTINUE`，跳转到第 1 步继续循环；这一步比较关键，当异常为调试异常时并没有调用 `ContinueDebugEvent` 函数让函数继续执行，这样调试程序就被断下了；
9.  修改调试程序的状态标识为被调试状态，调用函数检查 `int3` 断点，若是运行状态则设置断点；
10. 检查异常、线程的信息结构体，设置标识，调用一个函数处理所有的异常；
11. 检查更新线程信息，检查是否开了运行跟踪文件，若开了则更新文件，将调试程序状态设置回去；
12. 检查异常信息以及平台情况，做相应的错误处理；
13. 下面开始为对调试功能的处理，一共有 9 种情况（`0-No animation`、`1-Animate into`、`Animate over`、`Execute till RET`、`Skip RET instruction`、`Execute till user code`、`Run trace in`、`Run trace over`、`Gracefully stop animation`），有一个全局变量标识其状态；
14. 对 `Execute till RET`（执行到返回）进行处理，检查调试寄存器信息，检查第一个机器码是否为 `C2`（`retn…`）`C3`（`retn`）`CA`（`retf…`）`CB`（`retf`）或者 `CF`（`iretd`），若不是则跳过对其处理的部分；若是则开始处理：
    *   a) 调用`_GO` 函数检查线程信息，设置断点，检查其合法性，根据检查结果设置提示信息；
    *   b) 调整优先级，对调试程序结构体信息、`OD` 界面显示信息以及插件信息的更新
    *   c) 发送广播到子窗体，要求更新窗口内容，设置窗口获得焦点，最后返回到第 1 步
15. 对 `Execute till user code`（执行到用户代码）进行处理，检查 `eip` 所在的模块是否为系统模块，若不是则调整其优先级，若是则跳过设置优先级；根据 `ebx` 判断是否调用函数设置所有断点；
16. 对 `Gracefully stop animation`（停止自动跟踪）处理，检查线程信息，检查完毕后设置 `OD` 的界面，发送子窗体更新广播，更新插件；
17. 对于其他操作调用`_GO` 函数进行处理，除了线程 `ID`，其他参数都设 0 值；
18. 对 `No animation`（无自动跟踪）进行处理，调用 `Set_all_Bpoint` 函数处理，然后跳转到第 20 步，函数中调用 `ContinueDebugEvent` 函数继续调试程序，其 `ContinueStatus` 参数设置为 `DBG_EXCEPTION_NOT_HANDLED`；
19. ……部分省略，是一些对 `OD` 操作的处理，跟上面雷同，一般都是调用`_GO` 函数执行代码；检查更新标识；用`_Broadcast` 函数发送更新广播；检查是否有自动跟踪文件，对其进行更新等等；
20. 两个对系统时间全局变量的判断以及设置，更新窗口，跳转到循环体开始处

得到开机到现在的时间，检查是否创建了记录 `OD` 调试信息的日志文件以及运行跟踪结果的日志文件，根据结果进行相应的跳转：

```
00439077   >  E8 566>call    <jmp.&KERNEL32.GetTickCount>
0043907C   . |8945 C>mov     dword ptr [ebp-34], eax
0043907F   . |833D E>cmp     dword ptr [4D55E8], 0  ; 检查调试信息日志文件指针是否存在
00439086   . |75 09  jnz     short 00439091       ; 存在则跳转到下面检查暂停时间
00439088   . |833D 4>cmp    dword ptr [4D9E40], 0    ; 检查运行跟踪的文件指针是否存在
0043908F   . |74 43  je      short 004390D4       ; 存在则跳转到下面检查暂停时间
```

这里为检查时间处，根据时间更新文件：

```
00439091   > |A1 283>mov     eax, dword ptr [4E3B28]
00439096   . |05 307>add     eax, 7530      ; 上次更新时间增加 30 秒
0043909B   . |3B45 C>cmp     eax, dword ptr [ebp-34]  ;与当前时间比较，检查是否间隔超过 30 秒
0043909E   . |73 34  jnb     short 004390D4  ; 超过则跳过文件的更新
004390A0   . |833D E>cmp     dword ptr [4D55E8], 0  ; 检查调试信息日志文件指针是否存在
004390A7   . |74 0D  je      short 004390B6  ; 不存在跳过更新被调试的文件
```

更新调试信息日志文件：

```
004390A9   . |8B15 E>mov     edx, dword ptr [4D55E8]
004390AF   . |52     push    edx
004390B0   . |E8 B3B>call    004A4F68  ; _fflush 更新调试信息日志文件
004390B5   . |59     pop     ecx
```

检查运行跟踪文件指针是否存在，不存在则跳过更新运行跟踪文件：

```
004390B6   > |833D 4>cmp    dword ptr [4D9E40], 0
004390BD   . |74 0D  je      short 004390CC  ; 不存在则跳过更新运行跟踪文件
004390BF   . |8B0D 4>mov    ecx, dword ptr [4D9E40]
004390C5   . |51     push    ecx
004390C6   . |E8 9DB>call   004A4F68     ; _fflush 更新运行跟踪文件
004390CB   . |59     pop     ecx
```

将开机到当前的时间赋值给一个全局变量，保存文件最后一次更新时间：

```
004390CC   > |8B45 C>mov     eax, dword ptr [ebp-34]
004390CF   . |A3 283>mov     dword ptr [4E3B28], eax
```

检查是否需要刷新反汇编窗口，若不需要则跳过刷新：

```
004390D4   > |833D 3>cmp    dword ptr [4E3B34], 0  ; [4E3B34] 反汇编窗口刷新标识
004390DB   . |74 26  je      short 00439103
```

检查刷新窗口的间隔时间是否超过 0.5 秒，若超过则刷新所有窗口，并设置刷新时间以及刷新标识：

```
004390DD   . |8B55 C>mov     edx, dword ptr [ebp-34]
004390E0   . |2B15 3>sub     edx, dword ptr [4E3B38]    ; 得到两次刷新窗口的时间间隔
004390E6   . |81FA F>cmp     edx, 1F4      ; 比较时间间隔是否低于等于 0.5 秒
004390EC   . |76 15  jbe     short 00439103      ; 若低于 0.5 秒则跳过刷新
004390EE   . |E8 D95>call    _Redrawdisassembler    ; 刷新反汇编窗口
004390F3   . |8B4D C>mov     ecx, dword ptr [ebp-34]
004390F6   . |33C0   xor     eax, eax
004390F8   . |890D 3>mov     dword ptr [4E3B38], ecx    ; 记录刷新最后时间
004390FE   . |A3 343>mov     dword ptr [4E3B34], eax    ; 将刷新标识置零
```

检查主窗口更新标识，若不需要更新则将当前时间设置到记录主窗口更新时间的全局变量，然后跳过对主窗口的刷新：

```
00439103   > |833D C>cmp     dword ptr [4D57C0], 0  ; 检查主窗口更新标识
0043910A   . |74 09  je      short 00439115
0043910C   . |833D 3>cmp     dword ptr [4E3B30], 0  ; 检查 nNumber 是否为 0
00439113   . |75 0B  jnz     short 00439120
00439115   > |8B55 C>mov     edx, dword ptr [ebp-34]  ; 记录更新时间
00439118   . |8915 2>mov     dword ptr [4E3B2C], edx
0043911E   . |EB 55  jmp     short 00439175    ; 跳过对主窗口的更新
```

检查距离上次刷新的时间间隔是否超过 0.5 秒，小于则跳过刷新：

```
00439120   > |8B0D 2>mov     ecx, dword ptr [4E3B2C]
00439126   . |81C1 E>add     ecx, 3E8
0043912C   . |3B4D C>cmp     ecx, dword ptr [ebp-34]
0043912F   . |73 44  jnb     short 00439175
```

检查 `nNumber` 是否超过 50，小于则跳过更新：

```
00439131   . |833D 3>cmp     dword ptr [4E3B30], 32
00439138   . |7C 2B  jl      short 00439165
```

间隔时间*1000/`edx` （`edx` > 50），最后用计算的值更新：

```
0043913A   . |8B45 C>mov     eax, dword ptr [ebp-34]
0043913D   . |8B15 3>mov     edx, dword ptr [4E3B30]
00439143   . |2B05 2>sub     eax, dword ptr [4E3B2C]
00439149   . |50     push    eax                   ; /Divisor
0043914A   . |68 E80>push   3E8                   ; |Multiplier = 3E8 (1000.)
0043914F   . |52     push    edx                   ; |Multiplicand => 0
00439150   . |E8 FB5>call   <jmp.&KERNEL32.MulDiv>        ; \MulDiv
00439155   . |50     push    eax                   ; /Arg2
00439156   . |8D8E 6>lea     ecx, dword ptr [esi+3162]         ; |
0043915C   . |51     push    ecx                   ; |Arg1
0043915D   . |E8 CA8>call   _Flash                 ; \_Flash
00439162   . |83C4 0>add     esp, 8
00439165   > |33C0   xor     eax, eax
00439167   . |8B55 C>mov     edx, dword ptr [ebp-34]
0043916A   . |A3 303>mov     dword ptr [4E3B30], eax  ; 将除数置零
0043916F   . |8915 2>mov     dword ptr [4E3B2C], edx    ; 将系统时间记录
```

接受主线程的消息队列，并将消息信息放入 `MSG` 结构体中，并检查消息是否接受成功，若失败跳过下面对消息的处理：

```
00439175   > |6A 01  push    1                   ; /RemoveMsg = PM_REMOVE
00439177   . |6A 00  push    0                   ; |MsgFilterMax = WM_NULL
00439179   . |6A 00  push    0                   ; |MsgFilterMin = WM_NULL
0043917B   . |6A 00  push    0                   ; |hWnd = NULL
0043917D   . |8D8D 3>lea     ecx, dword ptr [ebp-9C8]         ; |
00439183   . |51     push    ecx                   ; |pMsg
00439184   . |E8 A56>call   <jmp.&USER32.PeekMessageA>     ; \PeekMessageA
00439189   . |85C0   test    eax, eax
0043918B   . |0F84 C>je      00439454
```

转换子窗口消息中的快捷键消息：

```
00439191   . |8D85 3>lea     eax, dword ptr [ebp-9C8]
00439197   . |50     push    eax                   ; /pMsg
00439198   . |8B15 8>mov     edx, dword ptr [4D3B80]      ; |
0043919E   . |52     push    edx    ; |hClient => 00020762 (class='MDIClient',parent=000607B6)
0043919F   . |E8 3E6>call   <jmp.&USER32.TranslateMDISysA>; \TranslateMDISysAccel
004391A4   . |85C0   test    eax, eax
004391A6   . |0F85 9>jnz     00439442
```

若子窗口有操作消息，则跳过对主窗体的消息的判断，对子窗口消息进行处理；下面是对一般消息的判断，处理还是在下面，这里跳过：

……

这里开始为对所有消息的处理：

```
00439454   > |833D 8>cmp     dword ptr [4D5780], 0
```
