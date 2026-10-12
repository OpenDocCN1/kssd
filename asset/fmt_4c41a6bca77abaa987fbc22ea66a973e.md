
# armadillo3.50a

> 原文：[`www.pediy.com/kssd/pediy06/pediy6837.htm`](https://www.pediy.com/kssd/pediy06/pediy6837.htm)

过年在家无事可做，幸好有本看雪老大编写的《加密与解密》和从网上下的《软件破解工具新年大礼包 2004》，就把自己关在屋里玩起了脱壳破解。在众位高手文章的指点下，从简单的 `upx`，`Aspack` 到复杂的 `Telock98`、`Krypton`、`Asprotect`，都亲手脱了一下，给我的感觉就是 `Ollydbg` 简直就是为了脱壳解密而开发的，用他可轻松对付这些加密壳。更主要的是可以边破解，边听歌，遇到不理解的地方可以把他放一边来看老大们的文章和翻阅 `MSDN`。但是在用 `Ollydbg` 脱 `armadillo` 时却进行不下去了。关于 `armadillo` 壳已经有很多文章在介绍他的原理了，大家都知道 `ollydbg` 是调用 `windows` 系统的调试 `api` 工作在 `r3` 上的，`windows` 系统只允许一个进程被一个调试器来调试，而 `armadillo` 自己生成 2 个进程，父进程做调试器，子进程做被调试者，父进程负责对子进程的代码进行解码，而对脱壳来说很重要的 `IAT` 部分却是子进程负责进行解密并 `hook`。`Ollydbg` 现在却没办法 `attach` 上子进程。记得有篇文章说用 `DebugActiveProcessStop` 这个函数可以使调试器和子进程脱离，但这个函数在 `xp` 和 `.net server` 的系统上才得到支持，我这台老旧的 `k6/2 300` 的 `cpu` 运行个 `win2000` 已经显得有些吃力了，根本没奢望用 `xp` 系统。在接下来的时间里我不断的翻阅资料和调试 `arm` 壳，终于找到一个用 `ollydbg` 修复 `IAT` 的方法，虽然这个方法并不完美，但我却用这个方法轻松的脱掉了好几个 `arm` 壳。好了，罗嗦了这么多，现在我们开始实战吧。

目标程序：就用 `armadillo3.50a(public builder)` 自己吧

工具：`OLLYDBG 1.09d`、`lordpe`、`import REC v1.4.2+`、`armadillo dumper 1.0`（一个小工具）

以上工具可以在 2004 新年大礼包和《加密与解密》的附带光盘里找到。

## 第一步：查找 OEP

查找 `OEP` 可以说和别的加密壳比起来，`armadillo` 的 `OEP` 查找可以说是最简单的了。用 `lordpe` 查看我们的目标程序，可以看到他的 `.text`、`.rdata`、`.data` 段的 `Roffset` 和 `Rsize` 都为零，这些段都是未加壳时原程序的运行空间，现在这里的代码被外壳加密起来放别的地方并把这里清零了。外壳执行完自己的任务跳到 `OEP` 执行前，把这些需要写入解密代码地址空间设置了 `PAGE_GUARD`(在 `win2000` 里)属性。可以想象跳转到 `OEP` 执行时肯定先引发一个异常，外壳（父进程）捕获这个异常，然后调用 `VirtualProtectEx` 修改 `0x1000` 内存段属性，并用 `WriteProcessMemory` 写入这个地址空间 `0x1000` 大小已解密的数据。然后调用 `ContinueDebugEvent`，使用参数 `DBG_CONTINUE` 让子程序在出错的地方重新运行。当子程序运行到 `GUARD` 属性的地址时就按上面的方法进行解密。解密一定的块后，父进程会根据一些算法每解密一个块后再破坏另外一个块的数据，并把它重新设置为 `GUARD` 属性，也就是说任何时候内存里也没有完整的原程序影像，这可防止 `lordpe` 之类的工具把他 `dump` 出来。下面是具体的查找 `OEP` 的步骤。

用 `OLLYDBG` 载入 `armadillo.exe`，会停在入口 `49A000` 处，记下入口开始二字节的内容（`60,E8`），会在后面的 `dump` 步骤里面用到。在命令窗口里下断点：`bp WaitForDebugEvent`。`Armadillo` 会检查是否存在调试器，所以我们必须隐藏我们的 `ollydbg`（用插件或下 `bp IsDebuggerPresent` 命令）。在调试设置里忽略掉单步中断，然后 `F9` 运行，用 `SHIFT+F9` 跳过 2 次异常，会断在 `WaitForDebugEvent` 处，这时查看堆栈窗口的参数说明：

```
0012DAAC   004777F0  /CALL 到 WaitForDebugEvent 来自 Armadill.004777EA
0012DAB0   0012EB60  |pDebugEvent = 0012EB60
0012DAB4   000003E8  \Timeout = 1000. ms
```

`0012EB60` 处就是发生调试事件时具体的内容存放地址，每次调用 `WaitForDebugEvent` 时都会使用这个地址。我们在数据窗口打开这个地址，取消 `WaitForDebugEvent` 断点，再下一个新的断点 `bp WriteProcessMemory`,按 `F9` 继续。等程序断在刚下的断点时，象我前面解释过的那样，是因为子程序执行到了 `OEP` 处发生异常，父进程调用 `WriteProcessMemory` 在向子程序 `OEP` 所在的块写解密过的数据。查看堆栈窗口：

```
0012D94C   0047B509  /CALL 到 WriteProcessMemory 来自 Armadill.0047B503
0012D950   00000050  |hProcess = 00000050 (window)
0012D954   0043B000  |Address = 43B000  OEP 肯定在 43B000—43C000 内
0012D958   00B83250  |Buffer = 00B83250
0012D95C   00001000  |BytesToWrite = 1000 (4096.)
0012D960   0012DA68  \pBytesWritten = 0012DA68
```

我们查看数据窗口，也就是存放具体调试信息的 `12EB60` 处：

```
0012EB60  01 00 00 00    DebugEventCode 01 表示 EXCEPTION_DEBUG_EVENT
0012EB64  D8 05 00 00    ProcessId 发生调试事件的进程 id
0012EB68  DC 05 00 00   ThreadId  发生调试时间的线程 id
0012EB6C  01 00 00 80    因为 12eb60 处为 1，所以这里代表的意思是 ExceptionCode
0012EB70  00 00 00 00
0012EB74  00 00 00 00  
0012EB78  B2 BE 43 00   这里就是我们要找的 OEP！！！！
```

现在我解释一下 `12eb60` 处数据的意思，整个数据块以偏移 `0` 处的数据决定整块所代表的事件。为 `1` 时是发生了异常，位 `2` 代表创建新线程事件，为 `3` 代表创建新进程事件。。。。。。当为 `1` 时，说明发生了异常，那么偏移 `0x0c` 处（`12eb6c`）代表的就是具体的异常代码，在 `winnt.h` 里我看到了他的定义：

```
#define STATUS_GUARD_PAGE_VIOLATION     ((DWORD   )0x80000001L)
```

可以看到发生异常是因为存取 `GUARD` 属性的内存页，偏移 `0x18`(`12eb78`)是发生异常时的指令地址，他就是我们要找的 `OEP` 啦，现在知道我为什么 `armadillo` 的 `OEP` 最好查找的原因了吧！如果你想深入的了解调试 `api` 和用到的数据结构的话，建议你看一下 `Platform SDK Documentation`，他里面介绍的比较详细。

## 第二步：抓取内存映像文件

最好在 `OEP` 处抓取内存映像文件，如果让程序运行起来再抓取的话，程序的全局变量好多已经被程序自己修改了，这样的话即使脱壳成功，程序也不见得能良好的运行。那我们就在子程序的 `OEP` 处写个 `JMP EIP` 指令。前面找 `OEP` 时我们知道子程序已经被挂起来了，现在父程序正准备调用写 `WriteProcessMemory` 向子程序 `OEP` 所在的块写数据，缓冲区地址是 `00B83250`，计算 `OEP` 所在字的地址：`00B83250+(0043BEB2-0043B000)=00B84102`。来到 `00B84102` 处看到入下数据

```
00B84102  55 8B EC 6A FF 68 C8 55 44 00 68 14 BC 43 00 64  U 嬱 j h 萓 D.h 糃.d
```

`jmp eip` 对应的二进制为 `EB FE`,拿笔记下 `55 8B`（这个数据在我们 `DUMP` 出映像时还要把他写入 `OEP` 处）。把他修改为 `EB FE`。

```
00B84102  EB FE EC 6A FF 68 C8 55 44 00 68 14 BC 43 00 64 膻靔 h 萓 D.h 糃.d
```

现在按 `F9` 运行，这时 `CPU` 占用率马上到了 `100%`，因为我们的子程序反复的在 `OEP` 处运行 `JMP EIP` 这条指令引起的。如果你感觉机器被拖的太慢的话可以降低他的优先级。子程序是被挂起来了，但是我们却不能抓取内存映像。只有 `OEP` 所在的块已经解密。现在要做的就是运行一段代码扫描一下子进程 `GUARD` 属性的地址空间，以触发调试器对其解码。我用的是一个小工具，是在 2004 春节大礼包里找到的。名字叫 `armadillo dumper 1.0`。如果你会编程并且分析甚至于编写过外挂程序的话，这样的小东东可以自己很容易的编写一个哦 J。

现在打开 `lordpe`，在我们的子进程上点右键，选择 `Dump Region` ，可以看到在地址 `00401000` 处有一块大小为 `3A000` 的块标记为 `GUARD` 属性。现在运行 `DUMPER` 小程序，`PID` 里面输入子进程的 `PROCESSID`。点 `dump` 会弹出另外一个对话框，在 `dump start` 里输入地址里输入 `0x401000`, `pags count` 里输入页数目 `0x3a000/1000=0x3a`,点 `dump` 按钮，`ollydbg` 马上中断在 `WriteProcessMemory` 处，我们一边按 `F9` 一边看堆栈窗口调用 `WriteProcessMemory` 的参数，会发现 `Address` 参数以 `0x1000` 的增量不断增加。

```
0012D94C   0047B509  /CALL 到 WriteProcessMemory 来自 Armadill.0047B503
0012D950   00000050  |hProcess = 00000050 (window)
0012D954   00401000  |Address = 401000    这个参数不断变化
0012D958   00B83250  |Buffer = 00B83250
0012D95C   00001000  |BytesToWrite = 1000 (4096.)
0012D960   0012DA68  \pBytesWritten = 0012DA68
```

按 `n` 下 `F9` 后，我们发现 `Address` 突然跳回了 `401000` 处，以后随着按 `F9` `Address` 参数不断的在高地址和低地址空间来回跳。这就是 `Armadillo` 在把解密过的数据重新加密起来，并且在这个地址空间重新赋予 `GUARD` 属性。看来只有把这个加密函数找出来，禁止 `Armadillo` 加密已经解密的数据。

我们再下一个新的断点: `bp VirtualProtectEx`，继续按 `F9`，当发现参数为下面时停止

```
0012D94C   0047B5C4  /CALL 到 VirtualProtectEx 来自 Armadill.0047B5BE
0012D950   00000050  |hProcess = 00000050 (window)
0012D954   00406000  |Address = Armadill.00406000
0012D958   00001000  |Size = 1000 (4096.)
0012D95C   00000120  |NewProtect = PAGE_EXECUTE_READ|PAGE_GUARD
0012D960   0012DA58  \pOldProtect = 0012DA58
```

上面的参数说明外壳已经完成了对数据的加密，并对地址 `406000`，大小为 `0x1000` 的块并重新 `GUARD` 起来。按 `ALT+F9` 回到调用 `VirtualProtectEx` 处：

```
0047B5C4    85C0          TEST EAX,EAX
0047B5C6    75 0F         JNZ SHORT Armadill.0047B5D7
0047B5C8    70 07         JO SHORT Armadill.0047B5D1
0047B5CA    7C 03         JL SHORT Armadill.0047B5CF
0047B5CC    EB 05         JMP SHORT Armadill.0047B5D3
```

按 `F8` 跟踪，下面是一段花指令，多按几次 `F8` 会到下面的代码：

```
0047B5FC    61            POPAD
0047B5FD    B0 01         MOV AL,1
0047B5FF    5F            POP EDI
0047B600    5E            POP ESI
0047B601    5B            POP EBX
0047B602    8BE5          MOV ESP,EBP
0047B604    5D            POP EBP
0047B605    C3            RETN
```

执行 `RETN` 就会到调用这个函数的代码处，而上面这个 `CALL` 很可能也就是我们要找的加密函数。

```
0047A42D    8B15 64044B00   MOV EDX,DWORD PTR DS:[4B0464]
0047A433    8D04B2          LEA EAX,DWORD PTR DS:[EDX+ESI*4]
0047A436    50              PUSH EAX
0047A437    8B0D 78044B00   MOV ECX,DWORD PTR DS:[4B0478]
0047A43D    8B15 7C044B00   MOV EDX,DWORD PTR DS:[4B047C]
0047A443    8B048A          MOV EAX,DWORD PTR DS:[EDX+ECX*4]
0047A446    50              PUSH EAX
0047A447    E8 73000000     CALL Armadill.0047A4BF
0047A44C    83C4 0C         ADD ESP,0C   执行 RETN 我们到这里
0047A44F    50              PUSH EAX
0047A450    F7D0            NOT EAX
0047A452    0FC8            BSWAP EAX
0047A454    58              POP EAX
0047A455    73 00           JNB SHORT Armadill.0047A457
0047A457    9C              PUSHFD
0047A458    60              PUSHAD
0047A459    EB 2B           JMP SHORT Armadill.0047A486
```

地址 `47a447` 处应该就是我们找的调用加密函数的地方。现在我们来修改他一下。这个是使用 `c` 调用方式，所以我们不用管堆栈平衡。可能要检查返回值，所以我修改成下面的样子：

```
0047A43D    8B15 7C044B00   MOV EDX,DWORD PTR DS:[4B047C]
0047A443    8B048A          MOV EAX,DWORD PTR DS:[EDX+ECX*4]
0047A446    50              PUSH EAX
0047A447    B8 01000000     MOV EAX,1   使返回值为 1，欺骗下面的代码
0047A44C    83C4 0C         ADD ESP,0C
0047A44F    50              PUSH EAX
0047A450    F7D0            NOT EAX
```

关闭 `OLLYDBG`，重新用前面的方法到 `OEP` 处，用 `DUMPER` 扫描 `GUARD` 地址空间，当第一次断在 `WriteProcessMemory` 时修改地址 `47A447` 的代码为 `MOV EAX,1`。取消断点运行程序，用 `DUMPER` 把所有的 `GUARD` 空间都扫描一下，这时用 `lordpe` 就可以抓取映像文件了。到这里我们已经打败了 `copymem-II` 抓取了映像文件。

注：最好用 `lordpe`，因为 `armadillo` 象 `TELOCK98` 那样修改了内存中的文件头，而 `lordpe` 可以根据磁盘上的文件头抓取映像。

## 第三步：重建输入表
