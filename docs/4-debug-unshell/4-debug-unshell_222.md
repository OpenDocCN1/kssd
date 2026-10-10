

# armadillo3.50a

> 原文：[`www.pediy.com/kssd/pediy06/pediy6837.htm`](https://www.pediy.com/kssd/pediy06/pediy6837.htm)

过年在家无事可做，幸好有本看雪老大编写的《加密与解密》和从网上下的《软件破解工具新年大礼包 2004》，就把自己关在屋里玩起了脱壳破解。在众位高手文章的指点下，从简单的 upx，Aspack 到复杂的 Telock98、Krypton、Asprotect，都亲手脱了一下，给我的感觉就是 Ollydbg 简直就是为了脱壳解密而开发的，用他可轻松对付这些加密壳。更主要的是可以边破解，边听歌，遇到不理解的地方可以把他放一边来看老大们的文章和翻阅 MSDN。但是在用 Ollydbg 脱 armadillo 时却进行不下去了。关于 armadillo 壳已经有很多文章在介绍他的原理了，大家都知道 ollydbg 是调用 windows 系统的调试 api 工作在 r3 上的，windows 系统只允许一个进程被一个调试器来调试，而 armadillo 自己生成 2 个进程，父进程做调试器，子进程做被调试者，父进程负责对子进程的代码进行解码，而对脱壳来说很重要的 IAT 部分却是子进程负责进行解密并 hook。Ollydbg 现在却没办法 attach 上子进程。记得有篇文章说用 DebugActiveProcessStop 这个函数可以使调试器和子进程脱离，但这个函数在 xp 和.net server 的系统上才得到支持，我这台老旧的 k6/2 300 的 cpu 运行个 win2000 已经显得有些吃力了，根本没奢望用 xp 系统。在接下来的时间里我不断的翻阅资料和调试 arm 壳，终于找到一个用 ollydbg 修复 IAT 的方法，虽然这个方法并不完美，但我却用这个方法轻松的脱掉了好几个 arm 壳。好了，罗嗦了这么多，现在我们开始实战吧。  
  
目标程序：就用 armadillo3.50a(public builder)自己吧  
工具：OLLYDBG 1.09d、lordpe、import REC v1.4.2+、armadillo dumper 1.0（一个小工具）  
以上工具可以在 2004 新年大礼包和《加密与解密》的附带光盘里找到。  
  
  
  
第一步：查找 OEP  
查找 OEP 可以说和别的加密壳比起来，armadillo 的 OEP 查找可以说是最简单的了。用 lordpe 查看我们的目标程序，可以看到他的.text、.rdata、.data 段的 Roffset 和 Rsize 都为零，这些段都是未加壳时原程序的运行空间，现在这里的代码被外壳加密起来放别的地方并把这里清零了。外壳执行完自己的任务跳到 OEP 执行前，把这些需要写入解密代码地址空间设置了 PAGE\_GUARD(在 win2000 里)属性。可以想象跳转到 OEP 执行时肯定先引发一个异常，外壳（父进程）捕获这个异常，然后调用 VirtualProtectEx 修改 0x1000 内存段属性，并用 WriteProcessMemory 写入这个地址空间 0x1000 大小已解密的数据。然后调用 ContinueDebugEvent，使用参数 DBG\_CONTINUE 让子程序在出错的地方重新运行。当子程序运行到 GUARD 属性的地址时就按上面的方法进行解密。解密一定的块后，父进程会根据一些算法每解密一个块后再破坏另外一个块的数据，并把它重新设置为 GUARD 属性，也就是说任何时候内存里也没有完整的原程序影像，这可防止 lordpe 之类的工具把他 dump 出来。下面是具体的查找 OEP 的步骤。  
   用 OLLYDBG 载入 armadillo.exe，会停在入口 49A000 处，记下入口开始二字节的内容（60,E8），会在后面的 dump 步骤里面用到。在命令窗口里下断点：bp WaitForDebugEvent。Armadillo 会检查是否存在调试器，所以我们必须隐藏我们的 ollydbg（用插件或下 bp IsDebuggerPresent 命令）。在调试设置里忽略掉单步中断，然后 F9 运行，用 SHIFT+F9 跳过 2 次异常，会断在 WaitForDebugEvent 处，这时查看堆栈窗口的参数说明：  
0012DAAC   004777F0  /CALL 到 WaitForDebugEvent 来自 Armadill.004777EA  
0012DAB0   0012EB60  |pDebugEvent = 0012EB60  
0012DAB4   000003E8  \\Timeout = 1000. ms  
  
0012EB60 处就是发生调试事件时具体的内容存放地址，每次调用 WaitForDebugEvent 时都会使用这个地址。我们在数据窗口打开这个地址，取消 WaitForDebugEvent 断点，再下一个新的断点 bp WriteProcessMemory,按 F9 继续。等程序断在刚下的断点时，象我前面解释过的那样，是因为子程序执行到了 OEP 处发生异常，父进程调用 WriteProcessMemory 在向子程序 OEP 所在的块写解密过的数据。查看堆栈窗口：  
0012D94C   0047B509  /CALL 到 WriteProcessMemory 来自 Armadill.0047B503  
0012D950   00000050  |hProcess = 00000050 (window)  
0012D954   0043B000  |Address = 43B000  OEP 肯定在 43B000—43C000 内  
0012D958   00B83250  |Buffer = 00B83250  
0012D95C   00001000  |BytesToWrite = 1000 (4096.)  
0012D960   0012DA68  \\pBytesWritten = 0012DA68  
我们查看数据窗口，也就是存放具体调试信息的 12EB60 处：  
0012EB60  01 00 00 00    DebugEventCode 01 表示 EXCEPTION\_DEBUG\_EVENT  
0012EB64  D8 05 00 00    ProcessId 发生调试事件的进程 id  
0012EB68  DC 05 00 00   ThreadId  发生调试时间的线程 id  
0012EB6C  01 00 00 80    因为 12eb60 处为 1，所以这里代表的意思是 ExceptionCode  
0012EB70  00 00 00 00  
0012EB74  00 00 00 00   
0012EB78  B2 BE 43 00   这里就是我们要找的 OEP！！！！  
。。。。。。。。。。。。  
现在我解释一下 12eb60 处数据的意思，整个数据块以偏移 0 处的数据决定整块所代表的事件。为 1 时是发生了异常，位 2 代表创建新线程事件，为 3 代表创建新进程事件。。。。。。当为 1 时，说明发生了异常，那么偏移 0x0c 处（12eb6c）代表的就是具体的异常代码，在 winnt.h 里我看到了他的定义：  
#define STATUS\_GUARD\_PAGE\_VIOLATION      ((DWORD   )0x80000001L)  
可以看到发生异常是因为存取 GUARD 属性的内存页，偏移 0x18(12eb78)是发生异常时的指令地址，他就是我们要找的 OEP 啦，现在知道我为什么 armadillo 的 OEP 最好查找的原因了吧！如果你想深入的了解调试 api 和用到的数据结构的话，建议你看一下 Platform SDK Documentation，他里面介绍的比较详细。  
  
  
  
第二步：抓取内存映像文件  
最好在 OEP 处抓取内存映像文件，如果让程序运行起来再抓取的话，程序的全局变量好多已经被程序自己修改了，这样的话即使脱壳成功，程序也不见得能良好的运行。那我们就在子程序的 OEP 处写个 JMP EIP 指令。前面找 OEP 时我们知道子程序已经被挂起来了，现在父程序正准备调用写 WriteProcessMemory 向子程序 OEP 所在的块写数据，缓冲区地址是 00B83250，计算 OEP 所在字的地址：00B83250+(0043BEB2-0043B000)=00B84102。来到 00B84102 处看到入下数据  
00B84102  55 8B EC 6A FF 68 C8 55 44 00 68 14 BC 43 00 64  U 嬱 j�h 萓 D.h 糃.d  
jmp eip 对应的二进制为 EB FE,拿笔记下 55 8B（这个数据在我们 DUMP 出映像时还要把他写入 OEP 处）。把他修改为 EB FE。  
00B84102  EB FE EC 6A FF 68 C8 55 44 00 68 14 BC 43 00 64  膻靔�h 萓 D.h 糃.d  
  
现在按 F9 运行，这时 CPU 占用率马上到了 100%，因为我们的子程序反复的在 OEP 处运行 JMP EIP 这条指令引起的。如果你感觉机器被拖的太慢的话可以降低他的优先级。子程序是被挂起来了，但是我们却不能抓取内存映像。只有 OEP 所在的块已经解密。现在要做的就是运行一段代码扫描一下子进程 GUARD 属性的地址空间，以触发调试器对其解码。我用的是一个小工具，是在 2004 春节大礼包里找到的。名字叫 armadillo dumper 1.0。如果你会编程并且分析甚至于编写过外挂程序的话，这样的小东东可以自己很容易的编写一个哦 J。  
现在打开 lordpe，在我们的子进程上点右键，选择 Dump Region ，可以看到在地址 00401000 处有一块大小为 3A000 的块标记为 GUARD 属性。现在运行 DUMPER 小程序，PID 里面输入子进程的 PROCESSID。点 dump 会弹出另外一个对话框，在 dump start 里输入地址里输入 0x401000,pags count 里输入页数目 0x3a000/1000=0x3a,点 dump 按钮，ollydbg 马上中断在 WriteProcessMemory 处，我们一边按 F9 一边看堆栈窗口调用 WriteProcessMemory 的参数，会发现 Address 参数以 0x1000 的增量不断增加。  
0012D94C   0047B509  /CALL 到 WriteProcessMemory 来自 Armadill.0047B503  
0012D950   00000050  |hProcess = 00000050 (window)  
0012D954   00401000  |Address = 401000    这个参数不断变化  
0012D958   00B83250  |Buffer = 00B83250  
0012D95C   00001000  |BytesToWrite = 1000 (4096.)  
0012D960   0012DA68  \\pBytesWritten = 0012DA68  
按 n 下 F9 后，我们发现 Address 突然跳回了 401000 处，以后随着按 F9 Address 参数不断的在高地址和低地址空间来回跳。这就是 Armadillo 在把解密过的数据重新加密起来，并且在这个地址空间重新赋予 GUARD 属性。看来只有把这个加密函数找出来，禁止 Armadillo 加密已经解密的数据。  
我们再下一个新的断点: bp VirtualProtectEx，继续按 F9，当发现参数为下面时停止  
0012D94C   0047B5C4  /CALL 到 VirtualProtectEx 来自 Armadill.0047B5BE  
0012D950   00000050  |hProcess = 00000050 (window)  
0012D954   00406000  |Address = Armadill.00406000  
0012D958   00001000  |Size = 1000 (4096.)  
0012D95C   00000120  |NewProtect = PAGE\_EXECUTE\_READ|PAGE\_GUARD  
0012D960   0012DA58  \\pOldProtect = 0012DA58  
上面的参数说明外壳已经完成了对数据的加密，并对地址 406000，大小为 0x1000 的块并重新 GUARD 起来。按 ALT+F9 回到调用 VirtualProtectEx 处：  
0047B5C4    85C0            TEST EAX,EAX  
0047B5C6    75 0F           JNZ SHORT Armadill.0047B5D7  
0047B5C8    70 07           JO SHORT Armadill.0047B5D1  
0047B5CA    7C 03           JL SHORT Armadill.0047B5CF  
0047B5CC    EB 05           JMP SHORT Armadill.0047B5D3  
按 F8 跟踪，下面是一段花指令，多按几次 F8 会到下面的代码：  
0047B5FC    61              POPAD  
0047B5FD    B0 01           MOV AL,1  
0047B5FF    5F              POP EDI  
0047B600    5E              POP ESI  
0047B601    5B              POP EBX  
0047B602    8BE5            MOV ESP,EBP  
0047B604    5D              POP EBP  
0047B605    C3              RETN  
执行 RETN 就会到调用这个函数的代码处，而上面这个 CALL 很可能也就是我们要找的加密函数。  
0047A42D    8B15 64044B00   MOV EDX,DWORD PTR DS:\[4B0464\]  
0047A433    8D04B2          LEA EAX,DWORD PTR DS:\[EDX+ESI\*4\]  
0047A436    50              PUSH EAX  
0047A437    8B0D 78044B00   MOV ECX,DWORD PTR DS:\[4B0478\]  
0047A43D    8B15 7C044B00   MOV EDX,DWORD PTR DS:\[4B047C\]  
0047A443    8B048A          MOV EAX,DWORD PTR DS:\[EDX+ECX\*4\]  
0047A446    50              PUSH EAX  
0047A447    E8 73000000     CALL Armadill.0047A4BF  
0047A44C    83C4 0C         ADD ESP,0C   执行 RETN 我们到这里  
0047A44F    50              PUSH EAX  
0047A450    F7D0            NOT EAX  
0047A452    0FC8            BSWAP EAX  
0047A454    58              POP EAX  
0047A455    73 00           JNB SHORT Armadill.0047A457  
0047A457    9C              PUSHFD  
0047A458    60              PUSHAD  
0047A459    EB 2B           JMP SHORT Armadill.0047A486  
地址 47a447 处应该就是我们找的调用加密函数的地方。现在我们来修改他一下。这个是使用 c 调用方式，所以我们不用管堆栈平衡。可能要检查返回值，所以我修改成下面的样子：  
0047A43D    8B15 7C044B00   MOV EDX,DWORD PTR DS:\[4B047C\]  
0047A443    8B048A          MOV EAX,DWORD PTR DS:\[EDX+ECX\*4\]  
0047A446    50              PUSH EAX  
0047A447    B8 01000000     MOV EAX,1   使返回值为 1，欺骗下面的代码  
0047A44C    83C4 0C         ADD ESP,0C  
0047A44F    50              PUSH EAX  
0047A450    F7D0            NOT EAX  
关闭 OLLYDBG，重新用前面的方法到 OEP 处，用 DUMPER 扫描 GUARD 地址空间，当第一次断在 WriteProcessMemory 时修改地址 47A447 的代码为 MOV EAX,1。取消断点运行程序，用 DUMPER 把所有的 GUARD 空间都扫描一下，这时用 lordpe 就可以抓取映像文件了。到这里我们已经打败了 copymem-II 抓取了映像文件。  
注：最好用 lordpe，因为 armadillo 象 TELOCK98 那样修改了内存中的文件头，而 lordpe 可以根据磁盘上的文件头抓取映像。  
  
  
  
第三步：重建输入表  
用 OLLYDBG 修 IAT 部分确实把我给难住了，因为 IAT 是在子程序中被解密的，而我们的 OLLYDBG 却没办法调试他。没办法，只好慢慢分析这个壳工作过程了。我发现父进程在开始调试子进程到子进程运行到 OEP 处，父进程根本没干扰过子进程的运行。象 WriteProcessMemory,SetThreadContext,SendMessage,等函数根本没调用过。对于调试事件通过调用 ContinueDebugEvent,参数用 DBG\_EXCEPTION\_NOT\_HANDLED 或 DBG\_CONTINUE 让子进程自己处理异常情况。我就产生这样的想法：可不可以让 OLLYDBG 加载的进程认为已经有父进程存在，自己应该做子进程？我们不需要这个进程多完美，只要他能给我们解出没加密的 IAT 就可以了。现在就开始分析他们父子之间的关系了。这一分析才发现自己的知识真的很匮乏，真应了那句，书到用时方恨少。又搬起了《WINDOWS 核心编程》这部武学秘籍边学习边研究。把这部书又翻了一遍才发现以前虽然看过这不书，而里面的好多精髓却没消化掉。现在对照这个 arm 壳，终于对里面所讲的父子进程之间，线程之间的关系、通信，内核对象，句柄的继承性等有了深刻的体会。呵呵，又说了这么多废话。现在我们继续说 arm 壳。分析发现，判断是否有父进程存在是根据一个互斥对象。Arm 壳会先尝试打开这个对象，如果成功，那么就自己作为子程序来运行。而且还不仅如此，父进程换创建了一个可继承的共享内存块句柄，并以此来和子程序通信。  
我们用 ollydbg 载入 armadillo.exe,下断点: bp CreateFileMappingA,bp SetEnvironmentVariableA,bp CreateProcessW, bp WriteProcessMemory,bp ResumeThread, bp DebugActiveProcess 运行程序断下时如下：  
0012DA9C   00476803  /CALL 到 CreateFileMappingA 来自 Armadill.004767FD  
0012DAA0   FFFFFFFF  |hFile = FFFFFFFF  
0012DAA4   0012F270  |pSecurity = 0012F270  
0012DAA8   00000004  |Protection = PAGE\_READWRITE  
0012DAAC   00000000  |MaximumSizeHigh = 0  
0012DAB0   00001000  |MaximumSizeLow = 1000  
0012DAB4   00000000  \\MapName = NULL  
再看 pSecurity 地址：  
0012F270  0C 00 00 00 00 00 00 00 01 00 00 00 D8 36 13 00  ........ ...? .  
第 3 个参数为 TRUE，说明 CreateFileMappingA 返回的句柄具有继承性。  
继续运行会断在 CreateEnvironmentVariableA:  
0012DAAC   00476C9E  /CALL 到 SetEnvironmentVariableA 来自 Armadill.00476C98  
0012DAB0   004AAB8C  |VarName = "\_RS"  
0012DAB4   0012F27C  \\Value = "72"  
父进程创建了一个环境变量"\_RS"，他的值就是刚才返回的句柄的十进制的表示”72”。  
0012DA8C   0047757C  /CALL 到 CreateProcessW 来自 Armadill.00477576  
0012DA90   0012EE10  |ModuleFileName = "C:\\Program Files\\Armadillo\\Armadillo.exe"  
0012DA94   00020998  |CommandLine = ""C:\\Program Files\\Armadillo\\Armadillo.exe""  
0012DA98   00000000  |pProcessSecurity = NULL  
0012DA9C   00000000  |pThreadSecurity = NULL  
0012DAA0   00000001  |InheritHandles = TRUE  //注意这里  
0012DAA4   00000004  |CreationFlags = CREATE\_SUSPENDED //这里使子进程挂起  
0012DAA8   00000000  |pEnvironment = NULL  
0012DAAC   00000000  |CurrentDir = NULL  
0012DAB0   0012EDCC  |pStartupInfo = 0012EDCC  
0012DAB4   0012F48C  \\pProcessInfo = 0012F48C  
当调用 CreateProcess 时，InheritHandles 使用了 TRUE，这表明父进程里的可继承句柄子进程都会拥有一个。  
0012D7AC   0047BA37  /CALL 到 WriteProcessMemory 来自 Armadill.0047BA31  
0012D7B0   00000050  |hProcess = 00000050 (window)  
0012D7B4   0049A000  |Address = 49A000   这不就是程序的入口地址吗？  
0012D7B8   0012DA9C  |Buffer = 0012DA9C  去看看他在入口写了什么东东  
0012D7BC   00000002  |BytesToWrite = 2  
0012D7C0   0012DAA0  \\pBytesWritten = 0012DAA0  
看下面的数据可知道在程序的入口写下了 jmp eip 指令  
0012DA9C  EB FE 00 00                                      膻..   
这时再让子进程开始运行，子进程也只能在入口处打转，却跑不到那里去了。  
0012D7D4   0047BAD9  /CALL 到 ResumeThread 来自 Armadill.0047BAD3  
0012D7D8   00000054  \\hThread = 00000054 (window)  
开始大模大样的调试子进程了。  
0012DAB0   00477664  /CALL 到 DebugActiveProcess 来自 Armadill.0047765E  
0012DAB4   000007C4  \\ProcessId = 7C4  
修复入口处字  
0012D7AC   0047BA5F  /CALL 到 WriteProcessMemory 来自 Armadill.0047BA59  
0012D7B0   00000050  |hProcess = 00000050 (window)  
0012D7B4   0049A000  |Address = 49A000  
0012D7B8   004B0440  |Buffer = Armadill.004B0440  
0012D7BC   00000002  |BytesToWrite = 2  
0012D7C0   0012DAA0  \\pBytesWritten = 0012DAA0  
到这里你是否想到了什么？嘿嘿，如果我们让父进程断在 12DAB0 处，我们再打开一个 ollydbg 不就可以附加进子进程了吗？而且从这里切入的话可以让我们省下创建共享内存，初始化内存，设置新的环境变量等琐碎事情。  
OK，让我们重新开始，这次只下一个断点 bp DebugActiveProcess，断下时堆栈窗口是：  
0012DAB0   00477664  /CALL 到 DebugActiveProcess 来自 Armadill.0047765E  
0012DAB4   000002B4  \\ProcessId = 2B4  
说明子进程 id 为 0x2b4,打开一个 ollydbg 程序，附加进这个进程，OK，成功了。  
按 ALT+F9 会来到入口处：  
0049A000 >- EB FE           JMP SHORT Armadill.  
0049A002    0000            ADD BYTE PTR DS:\[EAX\],AL  
0049A004    0000            ADD BYTE PTR DS:\[EAX\],AL  
子程序还在入口处打转呢。我们前面记过这个值，现在把他改回来。  
0049A000 >  60              PUSHAD  
0049A001    E8 00000000     CALL Armadill.0049A006  
0049A006    5D              POP EBP  
0049A007    50              PUSH EAX  
0049A008    51              PUSH ECX  
0049A009    EB 0F           JMP SHORT Armadill.0049A01A  
此时父进程还没生成哪个要命的互斥对象，所以在这里我们任由他运行下去的他还会再产生一个子进程。所以我们必须下一个断点: bp OpenMutexA。这时候不需要隐藏我们的调试器了，子程序不会检测用户模式调试器是否存在，因为他就工作在被调试的环境里。  
0012F574   0046F043  /CALL 到 OpenMutexA 来自 Armadill.0046F03D  
0012F578   001F0001  |Access = 1F0001  
0012F57C   00000000  |Inheritable = FALSE  
0012F580   0012FBB4  \\MutexName = "2B4::DABB778916"  
看到了吗？他尝试打开一个名为 2B4::DABB778916 的互斥对象。看到这个对象名有什么特别的地方了吗？2B4 不就正是这个子进程的 id 吗？他这样做就可以保证一个事例的多个实现而不互相干扰。现在我们必须建这个对象，我选择了在 401000 处输入如下代码：  
00401000    60              PUSHAD  
00401001    9C              PUSHFD  
00401002    68 B4FB1200     PUSH 12FBB4         ASCII "2B4::DABB778916"  
00401007    33C0            XOR EAX,EAX  
00401009    50              PUSH EAX  
0040100A    50              PUSH EAX  
0040100B    E8 E694A677     CALL KERNEL32.CreateMutexA  
00401010    9D              POPFD  
00401011    61              POPAD  
00401012  - E9 8F9FA777     JMP KERNEL32.OpenMutexA  
在 401000 处点右键选择新建起源，然后 F9 运行，又断在 OpenMutexA 处。ALT+F9 返回到调用处，这时查看 CPU 窗口：  
ERROR\_SUCCESS (00000000)  
说明我们刚才的代码起了作用。现在最好把刚才的代码清零，我认为这是个好习惯:)  
现在他已经乖乖的把自己当成子程序来运行了。至于他怎么解密 IAT 当然也就逃不过 ollydbg 眼睛了。  
在数据窗口里按 CTRL+G,输入 IAT 的地址 444000,继续运行程序，大约在第 15 次异常后我们发现 IAT 被写入了数据。  
00444000  4A 78 04 00 5E 78 04 00 70 78 04 00 82 78 04 00  Jx .^x .px .倄 .  
00444010  B6 78 04 00 A4 78 04 00 94 78 04 00 3C 78 04 00  秞 . .攛 .br> 00444020  00 00 00 00 74 6E 04 00 5C 6E 04 00 8C 6E 04 00  ....tn .\\n .宯 .  
00444030  11 00 00 80 00 00 00 00 56 77 04 00 62 77 04 00   ..€....Vw .bw .  
00444040  6E 77 04 00 42 77 04 00 8E 77 04 00 A0 77 04 00  nw .Bw .巜 .\_w .  
。。。。。。  
是不是很眼熟？armadillo 虽然还原了 IAT 表，但是他们指向的地址却已经被清零。所以这些数据也都是也垃圾~~~。这些数据给我们的作用只是能精确的定位 IAT 的地址和大小而已。  
现在在 444000 下内存写断点。以找到修改 IAT 表的指令地址，再经过几次异常，中断在这里：  
00EDFB31  8908         MOV DWORD PTR DS:\[EAX\],ECX  
00EDFB33  8B85 24E9FFFF MOV EAX,DWORD PTR SS:\[EBP\-16DC\]   Armadill.00444000  
00EDFB39    83C0 04         ADD EAX,4  
按几次 F8，会来到这里：  
00EDF7BF    6A 01           PUSH 1  
00EDF7C1    58              POP EAX         Armadill.00444004  
00EDF7C2    85C0            TEST EAX,EAX  
00EDF7C4    0F84 7D030000   JE 00EDFB47  
00EDF7CA    66:83A5 08E7FFF>AND WORD PTR SS:\[EBP\-18F8\],0  
00EDF7D2    83A5 00E7FFFF 0>AND DWORD PTR SS:\[EBP\-1900\],0  
00EDF7D9    83A5 04E7FFFF 0>AND DWORD PTR SS:\[EBP\-18FC\],0  
00EDF7E0    8B85 88ECFFFF   MOV EAX,DWORD PTR SS:\[EBP\-1378\]  
00EDF7E6    0FBE00          MOVSX EAX,BYTE PTR DS:\[EAX\]  
00EDF7E9    85C0            TEST EAX,EAX  
00EDF7EB    0F85 0C010000   JNZ 00EDF8FD  
00EDF7F1    C785 B0E6FFFF D>MOV DWORD PTR SS:\[EBP\-1950\],0EC78DF  
00EDF7FB    C785 B4E6FFFF 9>MOV DWORD PTR SS:\[EBP\-194C\],0EC7994  
00EDF805    C785 B8E6FFFF 0>MOV DWORD PTR SS:\[EBP\-1948\],0EC7906  
00EDF80F    C785 BCE6FFFF 1>MOV DWORD PTR SS:\[EBP\-1944\],0EC7919  
00EDF819    C785 C0E6FFFF 5>MOV DWORD PTR SS:\[EBP\-1940\],0EC7957  
00EDF823    C785 C4E6FFFF 5>MOV DWORD PTR SS:\[EBP\-193C\],0EC795C  
00EDF82D    C785 C8E6FFFF 6>MOV DWORD PTR SS:\[EBP\-1938\],0EC7961  
00EDF837    C785 CCE6FFFF 6>MOV DWORD PTR SS:\[EBP\-1934\],0EC7966  
00EDF841    C785 D0E6FFFF C>MOV DWORD PTR SS:\[EBP\-1930\],0EC79C6  
00EDF84B    C785 D4E6FFFF 8>MOV DWORD PTR SS:\[EBP\-192C\],0EC798D  
00EDF855    C785 D8E6FFFF D>MOV DWORD PTR SS:\[EBP\-1928\],0EC78DF  
00EDF85F    C785 DCE6FFFF 9>MOV DWORD PTR SS:\[EBP\-1924\],0EC7994  
00EDF869    C785 E0E6FFFF 0>MOV DWORD PTR SS:\[EBP\-1920\],0EC7906  
00EDF873    C785 E4E6FFFF 1>MOV DWORD PTR SS:\[EBP\-191C\],0EC7919  
00EDF87D    C785 E8E6FFFF B>MOV DWORD PTR SS:\[EBP\-1918\],0EC79B7  
00EDF887    C785 ECE6FFFF B>MOV DWORD PTR SS:\[EBP\-1914\],0EC79BC  
00EDF891    C785 F0E6FFFF C>MOV DWORD PTR SS:\[EBP\-1910\],0EC79C1  
00EDF89B    C785 F4E6FFFF 6>MOV DWORD PTR SS:\[EBP\-190C\],0EC7966  
00EDF8A5    C785 F8E6FFFF C>MOV DWORD PTR SS:\[EBP\-1908\],0EC79C6  
。。。。。。。。。。。。。。。。。。。  
这一段代码就是解密 IAT 并 HOOK 的程序段。  
要防止被 IAT 被破坏和加密的指令地址是：  
00EDF8DA    8908            MOV DWORD PTR DS:\[EAX\],ECX  
跳过上面的地址可以防止垃圾数据写入 IAT 表，  
00EDFA23   /75 11           JNZ SHORT 00EDFA36  
上面的地址改为 JMP 可以防止 api 被 hook。  
看他们的地址我们就可以知道这些代码是动态生成的，为了抢在执行这些代码前控制他们，我们重新运行程序，在第 15 次异常后在 00EDFA23 处下内存写断点。并修改 JNZ 为 JMP，NOP 掉 00EDF8DA 的指令，这样我们就可以得到一个完整的，未加密的 IAT 表了。运行 importREC，直接添入 IAT 的位置和大小就可以用他修复我们的前面 DUMP 出的文件了。用 fi 查一下，是用 vc 编写的。  
我用这个方法脱了几个 armadillo3.50a 壳时，都很成功。但是在脱加壳的记事本等 windows 系统所带文件时，却得不到正确的 IAT :)。这说明我这个方法不完美，或许我忽略了什么东西，甚至于这个方法根本就是错误的。之所以成功只不过是运气好，瞎猫碰着个死耗子。如果那位高人知道我错在什么地方，欢迎来信告诉我啊，我的信箱是：mysqladm@yahoo.com，本人感激不尽。解 IAT 部分用上面的方法不行的话就只好用 SOFTICE、TRW 等系统级的调试工具了。  
最后说明一下，上面脱出的文件还不能运行，跟踪一下会发现，他还调用了外壳提供的接口。这种情况对我这个初通次道的菜鸟就不知道怎么办了，希望某位大侠出面写个这方面的文章让我们开开眼界。  
我很久没写过东西了，这个帖子花了我好长时间才完成，而且感觉自己很多想要说的东西都没表达出来。脱这个壳给我最大的好处就是更加了解 windows 系统，这也正是我学脱壳的原因 ^\_^。关于 armadillo 壳的文章很多，我也记不清这些高人的名字了，在此衷心感谢他们的指点。  
  
Mysqladm   
     2004.3.3  

 4dd466d0e1e0cc123afbe504da2462ca