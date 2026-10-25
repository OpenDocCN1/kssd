
# 一个奇怪的带壳 crackme 的简单分析（Anti、Validate）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-667.htm`](https://www.pediy.com/kssd/pediy08/pediy8-667.htm)

【文章标题】: 一个奇怪的带壳 crackme 的简单分析  
【文章作者】: hawking  
【作者邮箱】: rich_hawking@hotmail.com  
【软件名称】: Bustme4  
【软件大小】: 21.5k  
【下载地址】: bm4.zip  
【加壳方式】: 不详  
【保护方式】: 未知  
【使用工具】: PEiD OLlyICE  
【操作平台】: win2k  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

首先声明，本人菜菜，分析水平有限，本文只是自己破解过程的一个记录，供和我一样的菜鸟参考，也备自己日后遇到类似问题时查找。侠之大者可以直接漂过。如果肯指点一二，感激不尽。

近来坛子里加壳与脱壳版里一个奇怪的带壳 crackme 讨论的人很多，不少大牛都信手给出了 key，成了创意比拼。我等小菜，载入 OD 里还没开始跑就 GameOver 了。实在是心有不甘。

发扬一下 cracker 的风格，自己动手搞定它。感谢 skylly，我一开始茫无头绪时，是你的提示给了我前进的方向。

### （一）Anti （Validate 部分参见 11 楼）

首先用 PDiD 查一下壳，显示 Nothing found * 点击 EP Section 后面的按钮可以看出有 6 个段，段名都是$ 看来好像是加壳了。

OD 载入

```
00409508 >- E9 00500000     jmp     0040E50D  
0040950D    0000            add     byte ptr \[eax\], al  
0040950F    0000            add     byte ptr \[eax\], al  
```

`F8`,来到 `0040E50D`,OD 提示不知如何单步，晕，才第一条指令啊。

1 tlscallback+CC 保护入口

我们再次在 PEiD 中打开，点一下 Subsystem 后面的按钮，看看 PE Details

可以看到

Directory **Information**

RVA SIZE

TLSTable: 00009000 00000024

点一下后面的按钮打开 TLS Viewer 可以看到

CallbackTableVA: 00409060

在数据窗口里可以看到 `00409060` 处的数值是 `00409040`。

具体 TLS 是什么我也不清楚，如果有大侠路过，请指点一下。只是大概了解这里的 Callback Function 会在系统加载程序的时候执行（程序第一条代码执行之前），我们可以看到这里的 Callback Function 地址是 `00409040`，我们在这里下断。然后将 OD 调试选项中事件选项卡设置第一次暂停于系统断点（默认情况下会暂停于 WinMain）。

Ctrl+F2 重新开始。

```
77F813B2    C3              retn  
77F813B3    33C9            xor     ecx, ecx  
77F813B5    E9 A5BE0000     jmp     77F8D25F  
```

`F9`

```
00409040    803D 08954000 C>cmp     byte ptr \[<模块入口点>\], 0CC     ;看看模块入口点是否下断 是则很可能被调试  
00409047    75 0A           jnz     short 00409053                   ;如果没下断则直接去模块入口点  
00409049    C705 09954000 0>mov     dword ptr \[409509\], 5000         ;如果下断了则修改模块入口点代码，使 jmp 跳向一个没有内容的地址，引发异常  
00409053    C2 0C00         retn    0C  
```

原来 OD 会自动在模块入口点下 Int3 断点，如果我们 Alt+B 打开断点窗口就会看到

地址 模块 激活 反汇编

```
00409508 Bustme4 仅一次  jmp 00409180  
```

我们修改 `jnz` 为 `jmp`，始终让它跳就可以避免这里的检测了。

-----------------------------------------------------------------------------------------

CC 反调试

注意后面通过的 `00750000` 段代码也存在 Int3 反调试，有些地址下 F2 断点将会导致程序异常。

```
0075001E    8B0424          mov     eax, dword ptr \[esp\]  
00750021    8038 CC         cmp     byte ptr \[eax\], 0CC  
00750024  - 0F84 A5C5FFFF   je      0074C5CF  
0075002A    C3              retn  
```

另外 `00401000` 代码段也存在 Int3 反调试。

```
00401081    A1 F2324000     mov     eax, dword ptr \[<&USER32.GetDlgItemTextA>\]  
00401086    8038 CC         cmp     byte ptr \[eax\], 0CC  
00401089  - 0F84 2AE1FFFF   je      003FF1B9  
0040108F    A1 F6324000     mov     eax, dword ptr \[<&USER32.MessageBoxA>\]  
00401094    8038 CC         cmp     byte ptr \[eax\], 0CC  
00401097  - 0F84 2AE1FFFF   je      003FF1C7  
```

如果想在上述地址下断点的话，就要修改一下 `je` 处的代码，直接 `nop` 掉就可以了。

-----------------------------------------------------------------------------------------

由于程序老是出现意外，我们不得不频繁重新开始，这种重复性的工作太没有意思了。怎么办，交给脚本来处理好了，这里我们用 OllyScript 写脚本通过检测。

脚本其实很简单的，脚本命令总共也就那么几条，看一下 OllyScript 附带的说明文档就搞定了，也可以参考 hnhuqiong 大侠的 ODbgScript 入门系列。

Script:
```
asm 00409047,"jmp 00409053"
esto
```

脚本很简单，由于水平不高，地址都是硬编码的，有些地址可能会因为机器的不同而存在差异，请自行修改。

2 进程入口参数校验

来到模块入口点

```
00409508 >^\\E9 73FCFFFF     jmp     00409180  

00409180    803D E1944000 0>cmp     byte ptr \[4094E1\], 1            ;比较 4094E1 处标记，是 1 的话则 Call ExitProcess Game Over!  
00409187    0F84 73030000   je      00409500  
0040918D    C605 E1944000 0>mov     byte ptr \[4094E1\], 1            ;设 4094E1 处标记为 1，如果程序被 Dump 的话，则执行 Dump 后的程序会不正常  
00409194    FF15 1E314000   call    dword ptr \[<&KERNEL32.GetCommand>; KERNEL32.GetCommandLineA 取命令行参数  
0040919A    83F8 00         cmp     eax, 0  
0040919D    74 16           je      short 004091B5  
0040919F    50              push    eax  
004091A0    68 52944000     push    00409452                         ; ASCII "Teeth\_In\_The\_Grass"  
004091A5    FF15 C6304000   call    dword ptr \[<&KERNEL32.lstrcmp>\]  ; KERNEL32.lstrcmpA  比较参数与上面的字符串  
004091AB    83F8 00         cmp     eax, 0  
004091AE    75 05           jnz     short 004091B5                   ;如果命令行参数不等于上面的字符串则跳  
004091B0  - E9 F7FAFFFF     jmp     00408CAC  
004091B5    68 58020000     push    258  
004091BA    68 F6914000     push    004091F6  
004091BF    6A 00           push    0  
004091C1    FF15 1A314000   call    dword ptr \[<&KERNEL32.GetModuleF>; KERNEL32.GetModuleFileNameA  
004091C7    68 80944000     push    00409480  
004091CC    68 A0944000     push    004094A0  
004091D1    6A 00           push    0  
004091D3    6A 00           push    0  
004091D5    6A 00           push    0  
004091D7    6A 00           push    0  
004091D9    6A 00           push    0  
004091DB    6A 00           push    0  
004091DD    68 52944000     push    00409452                         ; ASCII "Teeth\_In\_The\_Grass"  
004091E2    68 F6914000     push    004091F6  
004091E7    FF15 16314000   call    dword ptr \[<&KERNEL32.CreateProc>; KERNEL32.CreateProcessA   将 CommandLine 设为上面的参数然后创建进程  
004091ED    6A 00           push    0  
004091EF    FF15 C2304000   call    dword ptr \[<&KERNEL32.ExitProces>; KERNEL32.ExitProcess      Game Over!  
004091F5    C3              retn  

004033C7    FF20            jmp     dword ptr \[eax\]                   ;这里 jmp 到 SetUnHandledExceptionFilter  
```

这里我们一路 `F8`，可以看到由于程序没有带命令行参数启动，导致上面的字符串比较没能通过，程序将"`Teeth_In_The_Grass`"作为 CommandLine 的值，重新创建了一个新的进程，然后就退出了。

要通过这里的检测，我们必须将"`Teeth_In_The_Grass`"作为参数创建进程并调试。

Script:
```
asm 0040918D,"mov byte ptr[4094E1],0"
go 40919a
mov eax,00409452
go 4091B0
```

3 `SetUnhandledExceptionFilter`

然后就是一路 `jmp` 来 `jmp` 去，来到

```
0006FFBC   FFFFFFFF  /CALL 到 **SetUnhandledExceptionFilter**  
0006FFC0   00405325  \\pTopLevelFilter = Bustme4.00405325  
0006FFC4   77E71AF6  返回到 KERNEL32.77E71AF6  
```

看看堆栈

```
004053AF    FF15 DE304000   call    dword ptr \[<&KERNEL32.VirtualAll>; KERNEL32.VirtualAlloc  
```

观察堆栈可以知道，如果程序出现了 UnhandledException，正常运行时将会由 `00405325` 处的代码进行处理。由于我们正在调试程序，这时如果 `F9`，OD 会提示程序不知如何继续。

程序从 `004091B0` 处开始就一直跳来跳去，也没干什么正事。要想过这里的检测，我们改一下 `004091B0` 处的代码

Script:
```
asm 004091B0,"jmp 00405325"
```

4 CreateThead 线程保护

........

```
00405441    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
```

........

```
0040589B    FF15 F2304000   call    dword ptr \[<&KERNEL32.GetCurrentProcessId>\]    ; KERNEL32.GetCurrentProcessId  
```

........

```
0040590D    FF15 D2304000   call    dword ptr \[<&KERNEL32.OpenProcess>\]            ; KERNEL32.OpenProcess  
```

........

```
00405953    6A 02           push    2                                              ;传 Options 参数  
```

........

```
00405A03    FF15 EE304000   call    dword ptr \[<&KERNEL32.DuplicateHandle>\]        ; KERNEL32.DuplicateHandle  
```

........

```
0006FF74   00000040  |hSourceProcess = 00000040 (window)  
0006FF78   FFFFFFFE  |hSource = FFFFFFFE  
0006FF7C   00000040  |hTargetProcess = 00000040 (window)  
0006FF80   004052F5  |phTarget = Bustme4.004052F5  
0006FF84   00000000  |Access = 0  
0006FF88   00000001  |Inheritable = TRUE  
0006FF8C   00000002  \\Options = DUPLICATE\_SAME\_ACCESS  
```

堆栈：

```
00405A66    FF15 D6304000   call    dword ptr \[<&KERNEL32.CloseHandle>\]            ; KERNEL32.CloseHandle  
```

这儿程序复制了一个句柄，源句柄为 `FFFFFFFFFE(-2)`,这里按 `F8` 将会出现引用的内存不能为"written"的错误。

SourceProcess 应该是不存在 `FFFFFFFFFE` 的句柄的，这里不知道有没有什么好的方法处理一下。我们采用最笨的方法，直接跳过它，注意堆栈平衡就可以了。

Script:
```
asm 00405953,"jmp 00405A09"
```

........

```
00405AF1    FF15 02314000   call    dword ptr \[<&KERNEL32.CreateThread>\]           ; KERNEL32.CreateThread  
```

........

```
0006FF80   00000000  |pSecurity = NULL  
0006FF84   00000000  |StackSize = 0  
0006FF88   00406727  |ThreadFunction = Bustme4.00406727  
0006FF8C   00000000  |pThreadParm = NULL  
0006FF90   00000000  |CreationFlags = 0  
0006FF94   0040501C  \\pThreadId = Bustme4.0040501C  
```

堆栈：

```
00405B9D    FF15 0A314000   call    dword ptr \[<&KERNEL32.GetExitCodeThread>\]      ; KERNEL32.GetExitCodeThread  
```

这里程序创建了一个新线程，线程函数入口地址 `00406727`，这个线程除了 `SuspendThread` `VirtualQuery` `ResumeThread` 外没做什么特别的事情。其实并没有像有些大侠说的那样是什么线程保护。

........

```
00405BB8    813D F5524000 0>cmp     dword ptr \[4052F5\], 103                        ;比较线程退出代码  
```

........

```
00405BD9  ^\\0F84 5EFFFFFF   je      00405B3D  
00405BDF    66:87CB         xchg    bx, cx  
```

........

```
00405C46    FF15 12314000   call    dword ptr \[<&KERNEL32.GetTickCount>\]           ; KERNEL32.GetTickCount  
```

直接在 `00405BDF` 处下断 `F9` 就可以通过以上代码了。

5 `GetTickCount` 时间校验

........

```
00405C63    89C6            mov     esi, eax  
```

........

```
00405D6E    FF15 12314000   call    dword ptr \[<&KERNEL32.GetTickCount>\]           ; KERNEL32.GetTickCount  
```

........

```
00405D8B    29F0            sub     eax, esi  
```

........

```
00405DB6    3D D0070000     cmp     eax, 7D0  
```

........

```
00405DCD  ^\\0F8F 8BF7FFFF   jg      0040555E                                      ;如果单步调试的话需要将这一句 nop 掉，如果直接 F4 过来没有在两个 GetTickCount 中间的代码停顿过的话可不作处理  
```

........

```
00405E55    C705 F9524000 3>mov     dword ptr \[4052F9\], 30                          ;后面 RaiseException 会循环 0x30 次  
```

6 `RaiseException`

........

```
00405FB4    FF15 EA304000   call    dword ptr \[<&KERNEL32.RaiseException>\]         ; KERNEL32.RaiseException  
```

........

```
0006FF40   40010007  |ExceptionCode = 40010007  
0006FF44   00000000  |ExceptionFlags = EXCEPTION\_CONTINUABLE  
0006FF48   00000002  |nArguments = 2  
0006FF4C   00401000  \\pArguments = Bustme4.00401000  
0006FF50   0006FFE0  指向下一个 SEH 记录的指针  
0006FF54   004060BC  SE 处理程序  
```

堆栈：

```
00406227    FF15 12314000   call    dword ptr \[<&KERNEL32.GetTickCount>\]           ; KERNEL32.GetTickCount  
```

很明显程序在这里人为地制造了一处错误。如果要想程序正常运行，必须要跳到 SE 处理程序入口处 `004060BC`。

我们在 `004060BC` 处下断，不断地 `Shift+F9`,可最终程序也没有停在我们想断下来的 `004060BC` 处，而是弹出一个窗口告诉我们某某地址内存不能为"read"，然后就 Game Over 了。

这里不知道有没有什么好的办法处理通过，我用的是将 ExceptionCode 改为 `C0123333`，然后通过。

Script:
```
asm 00405E55,"mov dword ptr[4052F9],1"
asm 00405F9B,"push 0C0123333"
```

........

```
00406286    C705 F9524000 3>mov     dword ptr \[4052F9\], 30  
```

........

```
0040640B    FF15 EA304000   call    dword ptr \[<&KERNEL32.RaiseException>\]         ; KERNEL32.RaiseException  
```

........

```
0006FF40   40010007  |ExceptionCode = 40010007  
0006FF44   00000000  |ExceptionFlags = EXCEPTION\_CONTINUABLE  
0006FF48   00000002  |nArguments = 2  
0006FF4C   00401000  \\pArguments = Bustme4.00401000  
0006FF50   0006FFE0  指向下一个 SEH 记录的指针  
0006FF54   00406508  SE 处理程序  
```

堆栈：

```
0040667D    FF15 12314000   call    dword ptr \[<&KERNEL32.GetTickCount>\]           ; KERNEL32.GetTickCount  

004066B2    3D 88130000     cmp     eax, 1388  

004066D5   /0F8F 84050000   jg      00406C5F                                        ;同上处理  
```

按上面相同的方法处理通过

Script:
```
asm 00406286,"mov dword ptr [4052F9],1"
asm 004063E7,"push 0C0123333"

00750000    58              pop     eax  
00750001    3D 5FAF2F9F     cmp     eax, 9F2FAF5F  
00750006  - 75 FE           jnz     short 00750006  
00750008    E8 11000000     call    0075001E  

007513D2    FF10            call    dword ptr \[eax\]                                ; USER32.GetForegroundWindow  

0075145D    FF10            call    dword ptr \[eax\]                                ; USER32.GetWindowThreadProcessId  
```

7 `GetWindowThreadProcessId` 检测调试器

.......

```
0006FFB4   0075145F  /CALL 到 **GetWindowThreadProcessId** 来自 0075145D  
0006FFB8   003C0690  |hWnd = 003C0690 ('OllyICE - Bustme4.exe',class='pediy06',wndproc=03503168)  
0006FFBC   004052ED  \\pProcessID = Bustme4.004052ED  
```

........

```
0075151D    FF10            call    dword ptr \[eax\]                                ; KERNEL32.OpenProcess  

0075162F    FF10            call    dword ptr \[eax\]                                ; KERNEL32.ReadProcessMemory  

007516B5    FF10            call    dword ptr \[eax\]                                ; KERNEL32.CloseHandle  

00751728    FF15 C6304000   call    dword ptr \[<&KERNEL32.lstrcmp>\]                ; KERNEL32.lstrcmpA  

00751760    83F8 00         cmp     eax, 0  

00751778   /75 2F           jnz     short 007517A9                                  ;这里必须跳，否则紧随其后的代码会将 004052EC 的值减 1   
```

........

```
00751851    FF15 0E314000   call    dword ptr \[<&KERNEL32.OutputDebugStringA>\]     ; KERNEL32.OutputDebugStringA  
```

堆栈：

```
00751287    FF15 E2304000   call    dword ptr \[<&KERNEL32.VirtualPro>; KERNEL32.VirtualProtect  
```

这里取得创建当前窗口的进程 ID，并保存在 `004052ED` 处。由于我们正在用 OD 调试程序，所以这里取得是 OD 的进程 ID。

```
0006FF6C   00405000  |Address = Bustme4.00405000  
0006FF70   00003A53  |Size = 3A53 (14931.)  
0006FF74   00000001  |NewProtect = PAGE\_NOACCESS  
0006FF78   0006FF7C  \\pOldProtect = 0006FF7C  

00401014    8B0424          mov     eax, dword ptr \[esp\]             //这里放的是某个标志，在先前的代码里赋值为 0x1956DA59  
00401017    3D 59DA5619     cmp     eax, 1956DA59                    //比较标志，如果不相等则跳去执行 ExitProcess，Game Over!  
0040101C    75 0A           jnz     short 00401028  
0040101E    8F4424 FC       pop     dword ptr \[esp\-4\]  
00401022    E8 D9EF3500     call    00760000                         ; DialogBoxParamA  这里的 00760000 段代码模拟了 DialogBoxParamA 函数，调用这里进行窗口初始化等工作  
00401027    90              nop  
00401028    6A 00           push    0  
0040102A    E8 D1EF3600     call    00770000                         ; ExitProcess  如果标志比较没有通过就跳到这里自杀  
0040102F    90              nop  
00401030    90              nop  

00401040    55              push    ebp                               //消息循环处理例程入口  
00401041    89E5            mov     ebp, esp  
00401043    53              push    ebx  
00401044    56              push    esi  
00401045    57              push    edi  
00401046    817D 0C 1001000>cmp     dword ptr \[ebp+C\], 110            //窗口初始化消息  
0040104D    74 13           je      short 00401062  

0040104F    817D 0C 1101000>cmp     dword ptr \[ebp+C\], 111            //WM\_COMMAND  
00401056    74 14           je      short 0040106C  

00401058    837D 0C 10      cmp     dword ptr \[ebp+C\], 10             //关闭窗口消息  
0040105C    74 6C           je      short 004010CA  

0040105E    31C0            xor     eax, eax                          //Default 消息  
00401060    EB 75           jmp     short 004010D7  

00401062    8B45 08         mov     eax, dword ptr \[ebp+8\]             //窗口初始化，这里\[ebp+8\]保存的是窗口句柄  
00401065    A3 70214000     mov     dword ptr \[402170\], eax            //将窗口句柄保存到\[402170\]  
0040106A    EB 66           jmp     short 004010D2  

0040106C    837D 10 02      cmp     dword ptr \[ebp+10\], 2              //Exit 按钮被按下了  
00401070    74 58           je      short 004010CA  

00401072    837D 10 01      cmp     dword ptr \[ebp+10\], 1              //Validate 按钮被点击  
00401076    75 5A           jnz     short 004010D2  

00401078    B8 00204000     mov     eax, 00402000                    ; #include <windows.h>\\n\\nint main(int argc, char \*argv\[\])\\n\\n{\\n\\n        hwnd phstart;\\n\\n        point ptwnd = {15, getsystemmetrics(sm\_cyscreen)-15 };\\n\\n        phstart = windowfrompoint(ptwnd);\\n\\n        setwindowtext(phstart,"fuck");\\n\\nreturn 0;\\n\\n  
0040107D    D1E0            shl     eax, 1  
0040107F    31C0            xor     eax, eax  
00401081    A1 F2324000     mov     eax, dword ptr \[<&USER32.GetDlgI>   //检测 GetDlgItemTextA 函数是否被下断  
00401086    8038 CC         cmp     byte ptr \[eax\], 0CC  
00401089  - 0F84 2AE1FFFF   je      003FF1B9                            //Game Over  
0040108F    A1 F6324000     mov     eax, dword ptr \[<&USER32.Message>   //检测 MessageBoxA 函数是否被下断  
00401094    8038 CC         cmp     byte ptr \[eax\], 0CC  
00401097  - 0F84 2AE1FFFF   je      003FF1C7                            //Game Over  

0040109D    E8 3E000000     call    004010E0                            //这里对输入数据进行验证  
004010A2    83F8 00         cmp     eax, 0                              //如果结果大于等于 0 则弹出成功对话框  
004010A5    7D 02           jge     short 004010A9  
004010A7    EB 1F           jmp     short 004010C8  

004010A9    6A 40           push    40  
004010AB    68 51214000     push    00402151                         ; solved  
004010B0    68 58214000     push    00402158                         ; you solved the crackme.  
004010B5    FF35 70214000   push    dword ptr \[402170\]  
004010BB    E8 E6000000     call    004011A6                         ; MessageBoxA  
004010C0    6A 00           push    0  
004010C2    E8 39EF3700     call    00780000                         ; ExitProcess 显示成功信息后程序正常退出  
004010C7    90              nop  
004010C8    EB 08           jmp     short 004010D2  
004010CA    6A 00           push    0                                //如果是 WM\_CLOSE 或者是 Exit 按钮被按下则在这里处理  
004010CC    E8 2FEF3800     call    00790000                         ; ExitProcess       Game Over!  
004010D1    90              nop  
004010D2    B8 01000000     mov     eax, 1  
004010D7    5F              pop     edi  
004010D8    5E              pop     esi  
004010D9    5B              pop     ebx  
004010DA    C9              leave  
004010DB    C2 1000         retn    10  

004010DE    90              nop  
004010DF    90              nop  
004010E0    B8 F5204000     mov     eax, 004020F5                    ; jesus loves you, but only in a purely platonic way. i'm sure.  
004010E5    C8 000100       enter   100, 0                           //这里分配了一段大小为 100 的栈空间  
004010E9    89E7            mov     edi, esp  
004010EB    68 00030000     push    300                              //Count  
004010F0    68 74214000     push    00402174                         //Buffer  
004010F5    68 EA000000     push    0EA                              //ControlID 输入框 ID  
004010FA    FF35 70214000   push    dword ptr \[402170\]               //hWnd    
00401100    E8 9B000000     call    004011A0                         ; GetDlgItemTextA 得到输入字符串 Skey  
00401105    83F8 00         cmp     eax, 0                             
00401108    74 75           je      short 0040117F                   //如果 Skey 长度为 0 则 return -64  
0040110A    31C9            xor     ecx, ecx                         //ecx\=0  
0040110C    31DB            xor     ebx, ebx                         //ebx\=0  
0040110E    0FB791 74214000 movzx   edx, word ptr \[ecx+402174\]       //取 Skey 的前两位字符  
00401115    80FA 39         cmp     dl, 39                           //0x39 是 9 的 ASCII 码  
00401118    7E 03           jle     short 0040111D  
0040111A    80EA 07         sub     dl, 7                            //key\[ecx\] - 7  
0040111D    80FE 39         cmp     dh, 39  
00401120    7E 03           jle     short 00401125  
00401122    80EE 07         sub     dh, 7                            //key\[ecx+1\] - 7  
00401125    80EA 30         sub     dl, 30                           //key\[ecx\] - 30                             
00401128    80EE 30         sub     dh, 30                           //key\[ecx+1\] - 30  
0040112B    C0E2 04         shl     dl, 4  
0040112E    00D6            add     dh, dl  
00401130    86F2            xchg    dl, dh                            //这里的运算就是将用户输入的字符每两位一组转换成相应的 hex 数值，例如输入的是"B4",计算结果将是 0xB4  
00401132    8893 74214000   mov     byte ptr \[ebx+402174\], dl         //将运算的结果 hkey 写回去  
00401138    43              inc     ebx                               //ebx + 1  
00401139    83C1 02         add     ecx, 2                            //ecx + 2  
0040113C    83E8 02         sub     eax, 2                             
0040113F    83F8 00         cmp     eax, 0                           //每两位 1 组循环处理 Skey 字符直到字符串结束  
00401142  ^ 7F CA           jg      short 0040110E  
00401144    C683 74214000 0>mov     byte ptr \[ebx+402174\], 0  
0040114B    68 74214000     push    00402174  
00401150    57              push    edi                              //前面分配的栈地址  
00401151    E8 AAEE3900     call    007A0000                         ; lstrcpy   将刚刚运算得到的结果 hkey 复制到栈内去  
00401156    90              nop  
00401157    57              push    edi  
00401158    E8 A3EE3A00     call    007B0000                         ; lstrlen 计算栈内的 hkey 长度  
0040115D    90              nop  
0040115E    83F8 05         cmp     eax, 5                           //如果长度小于 5 则 return -64  
00401161    7C 1C           jl      short 0040117F  
00401163    83F8 0A         cmp     eax, 0A                          //如果长度大于 A 则 return -64  
00401166    7F 17           jg      short 0040117F  
00401168    68 FC0D4000     push    00400DFC  
0040116D    810424 37130000 add     dword ptr \[esp\], 1337  
00401174    57              push    edi  
00401175    E8 86EE3B00     call    007C0000                         ; lstrcmp  比较 hkey 与"XRB3-GHTP-XXCD-DD54-OPBJ-TYP1" 由于通过这里的 hkey 长度在 5 与 A 之间，而比较的字符串长度是 1C，所以这两个字符串不可能相同  
0040117A    90              nop  
0040117B    89C1            mov     ecx, eax  
0040117D    E3 07           jecxz   short 00401186                   //上面的两个字符串不相等的话则 return -64  
0040117F    B8 9CFFFFFF     mov     eax, -64  
00401184    EB 05           jmp     short 0040118B  
00401186    B8 02000000     mov     eax, 2                           //上面的字符串相等的话则 return 2，这样 validate 就成功了。  
0040118B    C9              leave  
0040118C    C3              retn  
0040118D    90              nop  
0040118E    90              nop  

0040119E    90              nop  
0040119F    90              nop  
004011A0  - E9 5BEE3C00     jmp     007D0000                         ; GetDlgItemTextA  
004011A5    90              nop  
004011A6  - E9 55EE3D00     jmp     007E0000                         ; MessageBoxA  
004011AB    90              nop  
004011AC    C3              retn  

E930EFFFFFE92BEFFFFF9090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909090909079214000  
```

这里程序设置了一个暗桩，如果检测到被调试，则最终 `004052EC` 值为 1，否则 `004052EC` 值为 2。检测后并不立即异常，而是在后面有对 `004052EC` 寄存的值检测的代码，如果检测到是 1 则会使程序运行不正常而退出。

要跳过这里程序的检测，可以按照通用的方法在调用 `GetWindowThreadProcessID` 后立即将 `004052ED` 处的值改为 Explorer.exe 的进程 ID,

也可以修改上面的 `jnz` 代码为 `jmp`。

Script:
```
asm 00751778,"jmp 007517A9"
```

8 OutputDebugStringA 检测调试器

```
00402179  - E9 2BEFFFFF     jmp     004010A9                          //只是针对这个 key 会来到这里  
```

这里还有一处检测，由于我用的是 OllyICE，己经针对 OutputDebugStringA 作了修改了，此处顺利通过。

9 Anti_Dump

```
004010E5    C8 000100       enter   100, 0                            //★平时没见过这条指令  
004010E9    89E7            mov     edi, esp                          //将堆栈的指针传给了 edi  
004010EB    68 00030000     push    300                               //★这里设置了 Count=300，表示最长可以取到 300 长度的输入字符串  
004010F0    68 74214000     push    00402174  
004010F5    68 EA000000     push    0EA  
004010FA    FF35 70214000   push    dword ptr \[402170\]  
00401100    E8 9B000000     call    004011A0                         ; GetDlgItemTextA  

0040114B    68 74214000     push    00402174  
00401150    57              push    edi                                //★ 这里的指向了当前栈空间  
00401151    E8 AAEE3900     call    007A0000                         ; lstrcpy  复制字符串，由于我们可取的字符串最长可能为 300，而刚才我们分配的栈空间大小才 100，如果 Skey 长度大于 100 的时候会发生什么？  

0040117D   /E3 07           jecxz   short 00401186  
0040117F   |B8 9CFFFFFF     mov     eax, -64  
00401184   |EB 05           jmp     short 0040118B  
00401186   \\B8 02000000     mov     eax, 2  
0040118B    C9              leave                                       //★见过，没好好理解过  
0040118C    C3              retn  
```

堆栈:

```
0006FC9C   004010A2  返回到 Bustme4.004010A2 来自 Bustme4.004010E0  
0006FCA0   00000111  
0006FCA4   002D0454  
0006FCA8   0046AAD0  
```

这里将 `00405000` 段代码保护方式设置成了 `PAGE_NOACCESS`，这样在您 DUMP 程序的时候就会出错。如果您想 Dump 程序，就要想办法跳过这段代码或者将 NewProtect 的值设置成 `PAGE_READWRITE` 之类的。

之后程序就没有什么 Anti 了，程序又 `VirtualAlloc` 若干段内存，并对 `00401000` 段进行解码，解码完之后会来到 `00401014`，然后进入 `0076000` 调用 `DialogBoxParamA` 载入程序界面，并设定 DlgProc 地址在 `00401040`。

给个跑到 `00401014` 处的完整 Script：

```
asm 00409047,"jmp 00409053"
esto
asm 0040918D,"mov byte ptr[4094E1],0"
go 40919a
mov eax,00409452
asm 004091B0,"jmp 00405325"
asm 00405953,"jmp 00405A09"
go 00405A09
asm 00405E55,"mov dword ptr[4052F9],1"
asm 00405F9B,"push 0C0123333"
esto
asm 00406286,"mov dword ptr [4052F9],1"
asm 004063E7,"push 0C0123333"
esto
asm 00751778,"jmp 007517A9"
go 00401014
ret
```

注意您有可能需要修改脚本中的相应地址，并且在运行脚本之前请清除所有的断点，调试选项中选择忽略所有异常。

--------------------------------------------------------------------------------

## 【经验总结】

这个壳用

1 tlscallback+CC 保护入口

2 进程入口参数校验

3 `SetUnhandledExceptionFilter`

4 CreateThead 线程保护 （个人感觉好像这里 CreateThread 没什么 Anti 的作用，不知道分析的对不对，请高手指正 ）

5 `GetTickCount` 时间校验

6 `RaiseException` （程序正常运行的时候不知道它是怎么通过这里的，请高手说说）

7 `GetWindowThreadProcessId` 检测调试器

8 OutputDebugStringA 检测调试器

9 Anti_Dump 设置某些代码段为 PAGE_NOACCESS 防止 Dump

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 12 月 24 日 11:28:51

### （二）Validate

抛了块砖砖，咋就不见大侠们的玉来呢？![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg) ![](img/45777b5dbd6b901752ab7df42fce6892.jpg)

再接再励，不信大牛们能做到的事我们菜菜努力之后会做不到。再说了有大侠们的讨论在前，多少我们也会得到一些启发吧。

用上面的脚本跑到 `00401014` 处，我们再用一个脚本去掉一些花花，这样看来会舒服一些。

Script:
```
fill 00401027,1,90
fill 0040102F,1,90
fill 004010C7,1,90
fill 004010D1,1,90
fill 00401156,1,90
fill 0040115D,1,90
fill 0040117A,1,90
fill 004011A5,1,90
fill 004011AB,1,90
ret
```

其实这里的代码少得可怜，正好适合我等菜菜分析。![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) 看见大段大段的代码就有些胆怯。

```
0006FB98   0000003C  
0006FB9C   00000001  
0006FBA0   01100077  
0006FBA4   00000004  
0006FBA8   00000005  
0006FBAC   00000001  
0006FBB0   0000000E  
```

........

```
0006FB98   1B0DACCB  
0006FB9C   000000B0  
0006FBA0   01100077  
0006FBA4   00000004  
0006FBA8   00000005  

0006FC9C   004010A2  返回到 Bustme4.004010A2 来自 Bustme4.004010E0  
0006FCA0   00000111  
0006FCA4   002D0454  
0006FCA8   0046AAD0  

> 代码:
> 
> * * *
> 
> EBFFF08B5C2404EB01E985DB0F444C241081C102893540EB02FF158D3408C1C61FEB01E833DB0FCB871C24FF3424EB03FFE8E9BB337812235381342422781223EB01E96AFFFF0C24B8FA3821C7EB01E8C1E8188BD4CD2EEB02FF15B9933497638D7E010FAFC1353729382968FACD98E5C12C2418588BD4CD2EEB02FF150F455C2404EB02FF15B9D96A93E5C1E91C3E8D04CD785634128D84C896A9CBED8BD4CD2EEB05E9EB05EBFCEBFCE95BEB06E9EB0659EBFBEBFBE9EB05E9EB05EBFCEBFCE95EEB2090E9EB14EB03E8FF15EBF7E9E9EB21E9FF15EB0F90EBEDE9EBF8E94BEEFFFFE9EB05E9EBE8FF15EBECFF15EB01E9EB08EB03E9FF15EBF4E95AD1E8EBDDE9FF1574214000
> 
> * * *
> 
> 
> 
> 代码:
> 
> * * *
> 
> EB02EB4DE8F9FFFFFF608B6C24248B453C8B7C057801EF8B4F188B5F2001EB498B348B01EE31C099AC84C07407C1CA0D01C2EBF43B54242875E58B5F2401EB668B0C4B8B5F1C01EB032C8B896C241C61C331DB648B43308B400C8B701CAD8B40085E688E4E0EEC5068AAFC0D7C50FFD68BF85A5AFFD666536A3266686C33687368656C54FFD0536A416863757465686C457865685368656C5450FFD753686F70656E53682E636F6D68656469796862732E70683A2F2F6268687474706A0353538D54240C528D5424285253FFD083C448B811AB1040C1E80866C74001602266B8732280700190E943EEFFFF90B1EBBAB7B5C4C8CBC9FAB2BBD0E8D2AABDE2CACD9090909074214000
> 
> * * *
> 
> 
> 
4dd466d0e1e0cc123afbe504da2462ca
```

[PRE58]

[PRE59]

[PRE60]

[PRE61]

[PRE62]

我们在 `0040109D` 处下断，然后在输入框内输入试炼码 hawking，点 Validate 按钮被断下来，`F7` 来到

[PRE63]

........

[PRE64]

通过我们的分析，程序会将我们输入的字符串两两一组转换成 1 个字节，但是不管我们输入的字符串长度是多少，都不能同时满足 `len(hkey)>=5` and `len(hkey) <=A` and `len(hkey) = 1C` 这个条件的，看来这个 CrackMe 是不可能 Validate 成功的。除非我们改变程序的执行流程。

而我们执行的情况也确实如此，不管我们输入什么样的字符串，`0040118C` retn 到 `004010A2` 之后 `eax` 值总是-64,没戏。

但是大侠们给出的 key 确实是能弹出成功的对话框框出来的，而且有的弹出的对话框还很彪捍，而我们又没有修改程序的执行代码呀，这是怎么回事呢？

输入 aalloverred 大侠给出的 key

[PRE65]

运行并跟踪看一下。没什么出奇，程序和我们先前看到的流程都是一样的，比较到长度是否大于 A 时就 return -64 了。看来是不会成功的了，但是 `F8` 几下之后竟然弹出了成功的对话框，狂晕！

再仔细跟着看了看，原来这里 retn 并没有 retn 到我们先前的 `004010A2` 处进行 `eax` 值是否大于等于 0 的比较，而是来到了

[PRE66]

然后程序就直接跳过了返回值的检验，成功了。



我们来看看关键代码![](img/f5d9879e4703d3dcc963d0743af268d2.jpg) 秘密就在这里了。

```
[PRE67]
[PRE68]
[PRE69]
```

我们看看 `enter` 到底何方神圣

`ENTER`—Make Stack Frame for Procedure Parameters

Creates a stack frame for a procedure. The first operand (`size` operand) specifies the size of the stack frame (that is, the number of bytes of dynamic storage allocated on the stack for the procedure). The second operand (`nesting level` operand) gives the lexical nesting level (0 to 31) of the procedure.

这里 `enter 100 , 0` 在栈里动态划分一块 100 个字节的空间

之前堆栈：

```
[PRE70]
```

寄存器：

```
ESP 0006FC9C
EBP 0006FCAC
```

之后堆栈：

```
[PRE71]
```

寄存器：

```
ESP 0006FB98
EBP 0006FC98
```

如果我们输入的字符串长度大于 100 的话，`lstrcpy` 操作将会超出 `enter` 分配的 100 大小的空间，copy 到程序正常的堆栈空间里来。这就是所谓的溢出吧。

再看看 `LEAVE`—High Level Procedure Exit

Releases the stack frame set up by an earlier `ENTER` instruction.

```
ESP ← EBP;
EBP ← Pop();
```

之前堆栈：

```
[PRE72]
```

寄存器：

```
ESP 0006FB98
EBP 0006FC98
```

之后堆栈：

```
[PRE73]
```

寄存器：

```
ESP 0006FC9C
EBP 0006FCAC
```

然后的 `retn` 语句就会返回到 `ESP` 所指向的 004010A2。到这里已经很明显了，只要 `lstrcpy` 的时候能够覆盖掉 006FC9C 处的值，那么程序执行到后来的 `retn` 时，就会返回到我们修改后想让它到达的地方。来个实验，随便输入 0x208 位字符，然后再加上 A9104000 应该也能验证成功。

由于程序可以 `retn` 到你想要 `retn` 到的任意地址，因此您完全可以随心所欲地作许多操作，这就要看您的创意和编程功底了。

shoooo 有个 CrackMe 和这个有着异曲同工之妙。有意者可以在 `CrackMe & ReverseMe` 版块搜索下载试玩之。

搞了半天，终于站在各位大侠的肩膀上将这个 CrackMe 分析完了。如果没有大侠们的 key 作参考，我们菜菜还真的难以想到这种通过堆栈溢出动态改变程序执行流程的方法（好像也叫什么 ShellCode 吧），改天好好看看《The Shellcoder's handbook》，学习学习。

看别的大侠写文章往往举重若轻，信手拈来。自己写来却是别有一番滋味在心头。很多东东知其然，不知其所以然，平时不写的时候感觉不到，真正想表达，特别是想清晳地阐述的时候往往失语，可能这就是高手与菜菜的区别吧。![](img/70b1b1b49da6fb968b36803b1ec24864.jpg) ![](img/70b1b1b49da6fb968b36803b1ec24864.jpg) ![](img/70b1b1b49da6fb968b36803b1ec24864.jpg)

贴一组可用的 key

> ```
> [PRE74]
> ```
>

大家都来贴

看谁的有创意

> ```
> [PRE75]
> ```
>

```
[PRE76]
```
