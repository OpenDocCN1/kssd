

# 为 VB 编译的程序增加自己的 DLL 调用【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-280.htm`](https://www.pediy.com/kssd/pediy08/pediy8-280.htm)

奇迹智能英语记忆系统 3.0 真人发音，比原来的 TTS 可是好了 N 多。但其中一些单词还是只有 TTS 发音，而且句子全是 TTS，而且用的是 MS 的 TTS 发音，真是惨不忍听。如果能采用 NEOSPEECH 发音，就和真人差不多了。  
一开始，做了个可以用 NEOSPEECH 发音的 ttd.exe 程序，然后，修改奇迹智能原代码，将 TTS 发音的调用改为 SHELL 调用：形式为 ttd.exe string，string 为传入的参数（就是要读出的单词或句子），由 ttd.exe 再调用 sapi 发音。但这种 SHELL 方式每发音一次就要调用一次 ttd.exe，速度比较慢。因此，想做成 DLL 调用。可以先做一个 DLL，然后在奇迹智能中调用就行。DLL 好做，但如何在奇迹智能中调用呢？  
  
查了论坛上的资料，增加 DLL 调用的方法有两种，一种是 PE 工具增加 improt，一种是在程序中直接调用。但是找不到 VB 程序中增加 DLL 调用的方法。  
于是自己在程序中找找了，看看有没有相关的代码。  
正好程序中有一个打开网站的 SHELL 调用，就先跟踪它吧！  
用 dream 写的 VB 事件断点插件对全部事件下断，（为全部事件下断的方法：VB 程序分析方法及修改例 （“万能分析断点”、启动 form 修改，form 隐藏））很快就跟踪到 SHELL 调用：相关代码如下：  
  
  
0044FD08   .  73 68 65 6C 6C 33 32 2E 64 6C 6C 00         ascii   "shell32.dll",0　　要调用的 DLL 的 ASCII　　　　　  
0044FD18   .  53 68 65 6C 6C 45 78 65 63 75 74 65 41 00   ascii   "ShellExecuteA",0　要调用的函数的 ASCII  
0044FD26      00          db      00  
0044FD27      00          db      00  
0044FD28      08FD4400    dd      luosoft\_.0044FD08     ;  ASCII "shell32.dll"　　　 指向 DLL 的地址  
0044FD2C      18FD4400    dd      luosoft\_.0044FD18     ;  ASCII "ShellExecuteA"     指向函数的地址  
0044FD30      00      　　db      00  
0044FD31      00          db      00  
0044FD32      00          db      00　　　　　　　　　　　　　　　　　　　　　　　　 有四个 00  
0044FD33      00          db      00  
0044FD34      D46B6100    dd      luosoft\_.00616BD4　　　　　　　　　　　　　　　　　地址指针，在 00616BD4+8 中存放函数的调用地址  
0044FD38      00          db      00  
0044FD39      00          db      00  
0044FD3A      00          db      00  
0044FD3B      00          db      00  
0044FD3C      00          db      00  
0044FD3D      00          db      00  
0044FD3E      00          db      00  
0044FD3F      00          db      00  
0044FD40   $  A1 DC6B6100 mov     eax, \[616BDC\]　　　　　　将 00616BD4+C 中的函数调用地址放 EAX  
0044FD45   .  0BC0        or      eax, eax  
0044FD47   .  74 02       je      short 0044FD4B　　　　　 函数调用地址是否为空  
0044FD49   .  FFE0        jmp     eax　　　　　　　　　　　不为空，直接调用函数  
0044FD4B   >  68 28FD4400 push    0044FD28　　　　 　　　　函数调用地址为空时，压入 DLL 指针  
0044FD50   .  B8 D02C4000 mov     eax, <jmp.&MSVBVM60.DllFunctionCall>　  
0044FD55   .  FFD0        call    eax　　　　　　　　　　　调用 VB 库得到函数调用指针，指针放入 00616BD4+8=616BDC  
0044FD57   .  FFE0        jmp     eax　　　　　　　　　　　调用函数  
  
要注意的是，  
0044FD28      08FD4400    dd      luosoft\_.0044FD08     ;  ASCII "shell32.dll"　　　 指向 DLL 的地址  
0044FD2C      18FD4400    dd      luosoft\_.0044FD18     ;  ASCII "ShellExecuteA"     指向函数的地址  
044FD30    　 00      　　db      00  
0044FD31      00          db      00  
0044FD32      00          db      00　　　　　　　　　　　　　　　　　　　　　　　　 有四个 00  
0044FD33      00          db      00  
0044FD34      D46B6100    dd      luosoft\_.00616BD4　  
这一段数据很重要，一是指向 DLL 及函数的指针，还有就是四个 00 及其后的函数地址存放地址。开始时，我没有指定函数存放地址，结果调用就出错。  
  
需要注意的是 0044FD50   .  B8 D02C4000 mov     eax, <jmp.&MSVBVM60.DllFunctionCall>　  
<jmp.&MSVBVM60.DllFunctionCall>究竟是什么呢？  
我们来看看 VB 程序开头，可以找到如下一连串 jmp，仔细看看，就能找到<jmp.&MSVBVM60.DllFunctionCall>的地址了：00402CD0  
  
00402CB8   .- FF25 F0104000    jmp     \[<&MSVBVM60.\_CIsin>\]                      ;  MSVBVM60.\_CIsin  
00402CBE   .- FF25 5C114000    jmp     \[<&MSVBVM60.\_CIsqrt>\]                     ;  MSVBVM60.\_CIsqrt  
00402CC4   .- FF25 94124000    jmp     \[<&MSVBVM60.\_CItan>\]                      ;  MSVBVM60.\_CItan  
00402CCA   .- FF25 84124000    jmp     \[<&MSVBVM60.\_allmul>\]                     ;  MSVBVM60.\_allmul  
00402CD0   .- FF25 30114000    jmp     \[<&MSVBVM60.DllFunctionCall>\]             ;  MSVBVM60.DllFunctionCall  
00402CD6   .- FF25 4C114000    jmp     \[<&MSVBVM60.\_\_vbaRedim>\]                  ;  MSVBVM60.\_\_vbaRedim  
00402CDC   .- FF25 90104000    jmp     \[<&MSVBVM60.\_\_vbaVarXor>\]                 ;  MSVBVM60.\_\_vbaVarXor  
00402CE2   .- FF25 38104000    jmp     \[<&MSVBVM60.\_\_vbaVarIdiv>\]                ;  MSVBVM60.\_\_vbaVarIdiv  
00402CE8   .- FF25 E8114000    jmp     \[<&MSVBVM60.\_\_vbaVarInt>\]                 ;  MSVBVM60.\_\_vbaVarInt  
  
  
现在就可以增加自己的 DLL 调用了。按以上代码形式，找个空地，将 DLL 和函数改为自己的。然后在调用 DLL 的程序中加上调用就行了：  
push 参数　　　　　这个参数是你的 DLL 函数要用到的参数，我的这个程序就是 string 的地址，只有一个参数，所以用一个 PUSH 就行了。  
call 0044FD40  
  
  
附：回顾一下一般程序中增加 DLL 调用的方法： http://bbs.pediy.com/showthread.php?s=&threadid=17365&highlight=%BC%C7%CA%C2%B1%BE  
0100878B      push    0100873C                         ; /pModule = "USER32.dll"  
01008790      call    \[<&KERNEL32.GetModuleHandleA>\]   ; \\GetModuleHandleA  
01008796      push    01001344                         ; /ProcNameOrOrdinal = "AppendMenuA"  
0100879B      push    eax                              ; |hModule  
0100879C      call    \[<&KERNEL32.GetProcAddress>\]     ; \\GetProcAddress  
010087A2      call    eax  
  
原来是没看懂，现在明白了：  
这是先 push 要调用的 DLL 的地址，然后调用 GetModuleHandleA，而 call    \[<&KERNEL32.GetModuleHandleA>\]的代码也是应从程序中找到。  
call    \[<&KERNEL32.GetProcAddress>\] 也是一样的。  
  
后来发现，采用\[<&KERNEL32.GetModuleHandleA>\]调用的 DLL，是在程序启动时已加载了的。  
如果要调用启动时未加载的 DLL，必须用 call    <jmp.&kernel32.LoadLibraryA>   
即：  
0100878B      push    0100873C                         ; /pModule = "USER32.dll"  
01008790      call    \[<&KERNEL32.LoadLibraryA>\]   ; \\LoadLibraryA  
01008796      push    01001344                         ; /ProcNameOrOrdinal = "AppendMenuA"  
0100879B      push    eax                              ; |hModule  
0100879C      call    \[<&KERNEL32.GetProcAddress>\]     ; \\GetProcAddress  
010087A2      call    eax  
  
MSVBVM60.DllFunctionCall 中用的就是 LoadLibraryA 

 4dd466d0e1e0cc123afbe504da2462ca