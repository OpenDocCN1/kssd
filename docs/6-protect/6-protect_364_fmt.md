
# Windows 反调试参考

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-275.htm`](https://www.pediy.com/kssd/pediy09/pediy09-275.htm)
> 英文区的 Windows Anti-Debug Reference 我花了整整一下午终于完工了.发上来![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")
> 不对的地方请多多指正
> Windows 反调试参考
> Nicolas Falliere 2007-09-12
> 链接: [`www.securityfocus.com/infocus/1893`](http://www.securityfocus.com/infocus/1893)
> 看雪论坛 Angeljyt 翻译于 2007-09-15

## 1. 说明

这篇文章分类并提供了几种在基于 WINDOWS NT 的操作系统中使用的反调试技术.反调试技术是程序用来检测自身是否运行于调试器之下的各种方式之一.它们被用于商业执行保护,加壳以及恶意软件, 为了阻止或减慢逆向工程过程.我们假定程序是在 ring3 级调试器,例如 Windows 平台的 `OllyDbg` 下分析.本文的目标是逆向工程人员和恶意软件分析人员.注意有一点我们只单纯的涉及一些基本的反调试和反跟踪技术.特殊的调试器检测,例如窗口或进程枚举,注册表扫描等等. 在此我将不再多说.

## 2. 反调试和反跟踪技术

### 探索内存差异

#### (1) `kernel32!IsDebuggerPresent`

如果进程正在被调试 `IsDebuggerPresent` 返回值为 1, 否则为 0. 这个 API 简单的读取 `PEB!BeingDebugged` 字节标志(在 `PEB` 结构中偏移量为 2).

设置 `PEB!BeingDebugged` 为 0 就可以轻松绕过它.

例子:

```
call IsDebuggerPresent
test eax, eax
jne @DebuggerDetected
...
```

#### (2) `PEB!IsDebugged`

这个字段指向进程环境块中的第二个字节.当进程被调试时由系统设置.这个字节可以重置为 0 而对程序的执行没有任何影响(这仅是一个通知标志).

例子:

```
mov eax, fs:[30h]
mov eax, byte [eax+2]
test eax, eax
jne @DebuggerDetected
...
```

#### (3) `PEB!NtGlobalFlags`

当进程创建的时候,系统设置一些标志,它们将定义各种 API 在该程序中的行为.这些标志能从 `PEB` 中偏移量为 `0x68`(请查看本文最后参考)的 `DWORD` 类型字段读取到. 不同标志的默认值设置依赖于进程是否被调试.如果进程正被调试, 在 `ntdll` 中一些控制堆处理流程的标志将被设置:

`FLG_HEAP_ENABLE_TAIL_CHECK`, `FLG_HEAP_ENABLE_FREE_CHECK` 和 `FLG_HEAP_VALIDATE_PARAMETERS`.

这个反调试能被重置 `NtGlobalFlags` 字段通过

例子:

```
mov eax, fs:[30h]
mov eax, [eax+68h]
and eax, 0x70
test eax, eax
jne @DebuggerDetected
...
```

#### (4) Heap flags

如先前所解释的 `NtGlobalFlags` 通知尤其是堆流程怎么表现 . 尽管很容易修改 `PEB` 字段,但是如果堆表现的行为和进程未被调试时不一致将会是个大问题. 这是个强有力的反调试,因为进程的堆有很多并且它们的块能单独被 `FLG_HEAP_*` 标志(例如块尾)影响. 堆头也一样能被影响.例如,检查 在堆头的 `ForceFlags` (偏移量为 `0x10`) 标志能发现是否有调试器.

这有两种方式可以轻松的绕过这个反调试:

- 创建一个非调试进程, 然后再附加调试器(一种容易的解决方案就是创建一个挂起进程, 运行到入口点, 补丁到一个无限循环中, 恢复进程, 附加调试器, 还原入口点).
- 强制被调试进程的 `NtGlobalFlags` 标志,通过注册表键 `"HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"`, 创建一个键名为进程名没有值的子键,在此子键下面把字符串 `GlobalFlags` 设为空值.

例子:

```
mov eax, fs:[30h]
mov eax, [eax+18h] ;process heap
mov eax, [eax+10h] ;heap flags
test eax, eax
jne @DebuggerDetected
...
```

#### (5) Vista 反调试 (没有名字)

这个反调试特定于 Windows Vista ,我通过比较在有和没有调试器时程序的内存转储发现的.我不能确定它的可靠性, 但是值得一提(测试于 Windows Vista 32 位, SP0, 英文版本).

当进程被调试, 它的主线程 `TEB`, 在偏移量 `0xBFC`, 包括一个在系统 dll 中引用 的 unicode 字符串的指针. 而且, 字符串就跟在指针后面 (因此位于 `TEB` 中偏移量 `0xC00`). 如果进程未被调试, 指针为 `NULL` 并且字符串没有出现.

例子:

```
call GetVersion
cmp al, 6
jne @NotVista
push offset _seh
push dword fs:[0]
mov fs:[0], esp
mov eax, fs:[18h] ; teb
add eax, 0BFCh
mov ebx, [eax] ; pointer to a unicode string
test ebx, ebx ; (ntdll.dll, gdi32.dll,...)
je @DebuggerNotFound
sub ebx, eax ; the unicode string follows the
sub ebx, 4 ; pointer
jne @DebuggerNotFound
;debugger detected if it reaches this point
;...
```

### 探索系统差异

#### (1) `NtQueryInformationProcess`

`ntdll!NtQueryInformationProcess` 是对 `ZwQueryInformationProcess` 进行包装的 syscall. 函数原型如下:

```
NTSYSAPI NTSTATUS NTAPI NtQueryInformationProcess(
IN HANDLE ProcessHandle,
IN PROCESS_INFORMATION_CLASS ProcessInformationClass,
OUT PVOID ProcessInformation,
IN ULONG ProcessInformationLength,
OUT PULONG ReturnLength
);
```

当把 `ProcessInformationClass` 设为 7 (`ProcessDebugPort` 常量)时调用, 如果进程被调试系统将会设置 `ProcessInformation` 为-1.

这也是个强有力的反调试, 并且没有容易的办法绕过它. 但是, 如果程序被跟踪, 当 syscall 返回的时候 `ProcessInformation` 能够被修改.

另外一个解决方案就是使用系统级驱动用钩子钩住 `ZwNtQueryInformationProcess` syscall.

绕过 `NtQueryInformationProcess` 将会使许多反调试技术 (例如 `CheckRemoteDebuggerPresent` 或者 `UnhandledExceptionFilter`) 也失效.

例子:

```
push 0
push 4
push offset isdebugged
push 7 ;ProcessDebugPort
push -1
call NtQueryInformationProcess
test eax, eax
jne @ExitError
cmp isdebugged, 0
jne @DebuggerDetected
...
```

#### (2) `kernel32!CheckRemoteDebuggerPresent`

这个 API 接受两个参数: 一个是进程句柄, 另一个是 `DWORD` 指针. 调用成功如果进程正在被调试则 `DWORD` 指针值设置为 1. ,在系统内部这个 API 通过设置参数 `ProcessDebugPort` (7)调用 `ntdll!NtQueryInformationProcess` `ProcessInformationClass`

例子:

```
push offset isdebugged
push -1
call CheckRemoteDebuggerPresent
test eax, eax
jne @DebuggerDetected
...
```

#### (3) `UnhandledExceptionFilter`

当发生异常时 Windows XP SP>=2, Windows 2003, and Windows Vista 等操作系统处理异常的方式如下:

- 如果有向量化异常则传递控制给每个进程的向量化异常处理器.
- 如果异常未被处理, 传递控制给每个线程的 `SEH` 处理器, 在产生异常的进程中指向 `FS:[0]`. `SEH` 是链起来的,如果前面的未处理则后面的将被轮流调用.
- 如果异常没有被任何处理器处理, 最终 `SEH` 处理器 (由系统设置) 将会调用 `kernel32!UnhandledExceptionFilter`. 这个函数将会怎么做取决于进程是否被调试
- 如果未被调试, 它会调用用户级过滤器 (通过 `kernel32!SetUnhandledExceptionFilter` 设置).
- 如果被调试, 程序将被终止.

在 `UnhandledExceptionFilter` 中是使用 `ntdll!NtQueryInformationProcess` 检测调试器的.

例子:

```
push @not_debugged
call SetUnhandledExceptionFilter
xor eax, eax
mov eax, dword [eax] ; trigger exception
;program terminated if debugged
;...

@not_debugged:
;process the exception
;continue the execution
;...
```

#### (4) `NtSetInformationThread`

`ntdll!NtSetInformationThread` 是对 `ZwSetInformationThread` syscall 包装. 函数原型如下:

```
NTSYSAPI NTSTATUS NTAPI NtSetInformationThread(
IN HANDLE ThreadHandle,
IN THREAD_INFORMATION_CLASS ThreadInformationClass,
IN PVOID ThreadInformation,
IN ULONG ThreadInformationLength
);
```

当把 `ThreadInformationClass` 设置为 `0x11` (`ThreadHideFromDebugger` 常量)调用, 线程将会从调试器在分离出去.

类似于 `ZwQueryInformationProcess`, 绕过这个反调试要么修改 `ZwSetInformationThread` 参数在它被调用之前, 要么直接使用内核驱动钩子钩住这个 syscall.

例子:

```
push 0
push 0
push 11h ;ThreadHideFromDebugger
push -2
call NtSetInformationThread
;thread detached if debugged
;...
```

#### (5) `kernel32!CloseHandle` 和 `NtClose`

APIs 让 `ZwClose` syscall (例如 `CloseHandle`, 非直接地) 的使用者能用于检测调试器. 当进程被调试, 使用一个无效的句柄调用 `ZwClose` 将会产生一个 `STATUS_INVALID_HANDLE` (`0xC0000008`) 异常.

As with all anti-debugs that rely on information made directly available from the kernel (therefore involving a syscall), 唯一合适绕过"CloseHandle"这个反调试的办法就是要么在它被调用之前从 ring3 修改 syscall 数据 , 要么安装内核钩子.

这个反调试尽管非常的强悍,但还没见广泛用于恶意软件.

例子:

```
push offset @not_debugged
push dword fs:[0]
mov fs:[0], esp
push 1234h ;invalid handle
call CloseHandle
; if fall here, process is debugged
;...

@not_debugged:
;...
```

#### (6) Self-debugging(自我调试)

进程能通过调试自身检测出是否被调试. 例如创建一个新进程,然后在父进程中调用 `kernel32!DebugActiveProcess(pid)`.

这个 API 轮流调用 `ntdll!DbgUiDebugActiveProcess` ,而 `ntdll!DbgUiDebugActiveProcess` 将调用 syscall `ZwDebugActiveProcess`. 如果进程正在被调试则 syscall 失败. 注意检索父进程 ID 能通过 `toolhelp32` APIs (在父进程 `PROCESSENTRY32` 结构中的 `th32ParentProcessID` 字段).

#### (7) Kernel-mode timers(内核模式定时器)

`kernel32!QueryPerformanceCounter` 是一个有效的反调试. 这个 API 调用 `ntdll!NtQueryPerformanceCounter` ,而 `ntdll!NtQueryPerformanceCounter` 是对 `ZwQueryPerformanceCounter` 包装的 syscall.

再一次, 没有容易的办法绕过这个反调试.

#### (8) User-mode timers(用户模式定时器)

`kernel32!GetTickCount` 这个 API 返回自从系统启动以来的毫秒数. 非常有意思的就是它并没有使用内核相关的服务来完成任务. 每个用户模式进程拥有映射到自己地址空间的计数器. 对于 8Gb 用户模式空间, 返回值如下:

```
d[0x7FFE0000] * d[0x7FFE0004] / (2²⁴)
```

#### (9) `kernel32!OutputDebugStringA`

这个反调试非常的原始, 我只在用 `ReCrypt` v0.80 加壳的文件中遇到过一次. 这个伎俩包括使用一个有效的 `ASCII` 字符串调用 `OutputDebugStringA`. 如果程序正处于被调试, 返回值将会是作为参数传递进去的字符串的地址. 在正常条件下, 返回值为 1.

例子:

```
xor eax, eax
push offset szHello
call OutputDebugStringA
cmp eax, 1
jne @DebuggerDetected
...
```

#### (10) `Ctrl-C`

当控制台程序被调试, `Ctrl-C` 信号将会抛出 `EXCEPTION_CTL_C` 异常, 反之信号处理器将会被直接调用如果未被调试.

例子:

```
push offset exhandler
push 1
call RtlAddVectoredExceptionHandler
push 1
push sighandler
```



> 调用 `SetConsoleCtrlHandler`  
> 推送 `0`  
> 推送 `CTRL_C_EVENT`  
> 调用 `GenerateConsoleCtrlEvent`  
> 推送 `10000`  
> 调用 `Sleep`  
> 推送 `0`  
> 调用 `ExitProcess`  
> `exhandler:`  
> `;check if EXCEPTION_CTL_C, if it is,`  
> `;debugger detected, should exit process`  
> `;...`  
> `sighandler:`  
> `;continue`  
> `;...`

> - CPU anti-debug

> **(1) Rogue `Int3` (流氓的 `Int3`)**

> 愚弄脆弱的调试器这可是个经典的反调试。它包含插入 `INT3` 机器码在有效的指令序列中间。当 `INT3` 被执行到，如果程序未被调试，控制将会传递给受保护的异常处理器程序执行继续。
>
> 由于 `INT3` 指令被调试器用于设置软件断点，插入 `INT3` 机器码能够愚弄调试器使其认为这是一个自己的断点。因此控制不会传递给异常处理器，当然程序的结果将会改变。调试器应该跟踪在哪里自己设置了软件断点来避免它。
>
> 类似地，注意 `INT3` 可以编码为 `0xCD`, `0x03`。
>
> 例子：
>
> ```
> push offset @handler
> push dword fs:[0]
> mov fs:[0], esp
> ;...
> db 0CCh
> ;if fall here, debugged
> ;...
> @handler:
> ;continue execution
> ;...
> ```

> **(2) "Ice" Breakpoint**

> 这个所谓的 "Ice breakpoint" 是 `Intel` 未公开的指令之一，机器码 `0xF1`。它被用于检查跟踪程序。
>
> 执行这个指令将产生单步异常。因此，如果程序已经被跟踪，调试器将会以为它是通过设置标志寄存器中的单步标志位生成的正常异常。相关的异常处理器将不会被执行到，自然程序也就不会按预期的执行了。
>
> 绕过这个伎俩很容易：运行跳过这个指令，而不是单步到它上面。异常将会生成，既然程序未被跟踪，调试器应该知道传递控制给异常处理器。
>
> 例子：
>
> ```
> push offset @handler
> push dword fs:[0]
> mov fs:[0], esp
> ;...
> db 0F1h
> ;if fall here, traced
> ;...
> @handler:
> ;continue execution
> ;...
> ```

> **(3) Interrupt `2Dh` (`2Dh` 中断)**

> 如果程序未被调试这个中断将会生产一个断点异常。被调试并且未使用跟踪标志执行这个指令，将不会有异常产生程序正常执行。如果被调试并且指令被跟踪，尾随的字节将被跳过并且执行继续。因此，使用 `INT 2Dh` 能作为一个强有力的反调试和反跟踪机制。
>
> 例子：
>
> ```
> push offset @handler
> push dword fs:[0]
> mov fs:[0], esp
> ;...
> db 02Dh
> mov eax, 1 ;anti-tracing
> ;...
> @handler:
> ;continue execution
> ;...
> ```

> **(4) Timestamp counters (时间戳计数器)**

> 存储自系统启动以来 `CPU` 时钟周期数的高精度计数，能使用 `RDTSC` 指令查询。经典的反调试包括在关键点测量时间差值，常在异常处理器周围。如果差值太大，这就意味着程序正处于调试器控制之下（在调试器中处理异常并且把控制返回给被调试者这是个很耗时的任务）。
>
> 例子：
>
> ```
> push offset handler
> push dword ptr fs:[0]
> mov fs:[0],esp
> rdtsc
> push eax
> xor eax, eax
> div eax ;trigger exception
> rdtsc
> sub eax, [esp] ;ticks delta
> add esp, 4
> pop fs:[0]
> add esp, 4
> cmp eax, 10000h ;threshold
> jb @not_debugged
> @debugged:
> ...
> @not_debugged:
> ...
> handler:
> mov ecx, [esp+0Ch]
> add dword ptr [ecx+0B8h], 2 ;skip div
> xor eax, eax
> ret
> ```

> **(5) `Popf` and the trap flag (`Popf` 和陷阱标志)**

> 位于标志寄存器中的陷阱标志，控制程序的跟踪。如果标志被设置，执行一个指令将会生产一个单步异常。陷阱标志能被处理为了阻碍跟踪者。例如，下面这的指令序列将会设置陷阱标志：
>
> ```
> pushf
> mov dword [esp], 0x100
> popf
> ```
>
> 如果程序被跟踪，将不会没有实际效果在标志寄存器上，因此调试器将会处理异常，认为这是来自合法的跟踪。异常处理器不会被执行到。绕过这个反跟踪伎俩只要简单的跳过 `pushf` 指令

> **(6) Stack Segment register (堆栈段寄存器)**

> 这是个非常原始的反跟踪。我在一个叫 `MarCrypt` 的壳中遇到。我相信这并不广为人知，更不用说使用了。
>
> 它包括跟踪过下面的指令序列：
>
> ```
> push ss
> pop ss
> pushf
> nop
> ```
>
> 当跟踪到 `pop ss` 上面，下一个指令将会被执行并且调试器不会中断于其上，因此停止在下面一个指令上（在本例中是 `NOP`）。
>
> `Marcrypt` 以下面的方式使用这个反调试：
>
> ```
> push ss
> ; junk
> pop ss
> pushf
> ; junk
> pop eax
> and eax, 0x100
> or eax, eax
> jnz @debugged
> ; carry on normal execution
> ```
>
> 这个伎俩就是这样的，如果调试器跟踪过这些指令序列，`popf` 将会被隐式执行，并且调试器不能够设置刚进堆栈的陷阱标志位。保护程序检查陷阱标志如果发现则终止程序。
>
> 一个简单绕过这个反跟踪的办法就是断点在 `popf`（我不明白这里上面的指令序列中并没有 `popf`，是不是作者写错了应该是 `pushf`）然后运行程序（为了避免使用陷阱标志）。

> **(7) Debug registers manipulation (调试寄存器处理)**

> 调试寄存器 (`DR0` 到 `DR7`) 被用于设置硬件断点。保护程序能处理它们来检测到硬件断点被设置（也就是说正在被调试），重置它们或者设置为特殊的值来执行以后的代码检查。一些壳例如 `tElock` 使用调试寄存器来阻止逆向者使用它们。
>
> 从用户模式视图来看，调试寄存器不能使用有特权的 `mov drx, ...` 指令来设置。其它的方式存在：
>
> - 一个异常产生，线程上下文改变（它包括在异常发生之时的 `CPU` 寄存器），然后恢复到使用新的上下文正常执行。
> - 其它的方式就是使用 `NtGetContextThread` 和 `NtSetContextThread` syscalls（在 `kernel32` 中使用 `GetThreadContext` 和 `SetThreadContext` 有效）。
>
> 大多数的保护者使用第一种，非正式方式。
>
> 例子：
>
> ```
> push offset handler
> push dword ptr fs:[0]
> mov fs:[0],esp
> xor eax, eax
> div eax ;generate exception
> pop fs:[0]
> add esp, 4
> ;continue execution
> ;...
> handler:
> mov ecx, [esp+0Ch] ;skip div
> add dword ptr [ecx+0B8h], 2 ;skip div
> mov dword ptr [ecx+04h], 0 ;clean dr0
> mov dword ptr [ecx+08h], 0 ;clean dr1
> mov dword ptr [ecx+0Ch], 0 ;clean dr2
> mov dword ptr [ecx+10h], 0 ;clean dr3
> mov dword ptr [ecx+14h], 0 ;clean dr6
> mov dword ptr [ecx+18h], 0 ;clean dr7
> xor eax, eax
> ret
> ```

> **(8) Context modification (上下文改变)**

> 由于调试寄存器使用，上下文也能够以一种不方便的方式改变程序的执行流程。调试器很容易被混淆！
>
> 注意另一个 syscall，`NtContinue` 能用于在当前线程中加载一个新的上下文（例如，这个 syscall 被用于异常处理器的管理）。

> **- 未归类的反调试**

> **(1) TLS-callback (线程本地存储回调)**

> 这个反调试在几年前还并不为人所知。它包括指示 `PE` 装载器程序的第一个入口点位于线程本地存储入口（在 `PE` 可选头部中的第 10 个目录入口数），通过这样做，程序的入口点将不会首先被执行。`TLS` 入口能以一种隐蔽的方式执行反调试检查。
>
> 注意实际上，这个技术并没有广泛使用。
>
> 即使稍老一点的的调试器（包括 `OllyDbg` 在内）也并没有注意到 `TLS`，对策是非常容易采纳的，通过自定义插件的补丁工具。

> **(2) CC scanning (CC 扫描)**

> 壳最常见的一种保护方案就是循环对 `CC`（这是 `INT3` 的机器码）扫描，目的在于检测调试器设置的软件断点。如果你想要避免这个麻烦，可以使用硬件断点或者自定义一种软件断点。`CLI` (`0xFA`)（汇编语言中是清除中断指令）就是替换 `INT3` 机器码很好的候选者。This instruction does have the requirements for the job: 在 `ring3` 级程序中执行它会产生一个特权指令异常，并且只占用一字节空间。

> **(3) EntryPoint RVA set to 0 (入口点 `RVA` 偏移量设置为 0)**

> 一些加壳的文件把入口点 `RVA` 设置为 0，这意味着它们将会开始执行 `'MZ...'` 对应于 `'dec ebx / pop edx ...'`。
>
> 这本身并不是一个反调试技术，但是如果使用软件断点中断在入口点就会变得让人苦恼。
>
> 如果创建一个挂起的进程，然后设置 `INT3` 于 `RVA 0`，将会擦除 `MZ` 值（`'M'`）。这个魔术标志在进程创建时被检查，但是当进程恢复（希望到达入口点）时它会再次在 `ntdll` 中被检查。哪样的话，一个 `INVALID_IMAGE_FORMAT` 异常将会产生。
>
> 如果你创建自己的跟踪或者调试工具，可以使用硬件断点避免这个问题。

> **[3] 总结**

> 知道恶意软件或者保护壳使用的反调试和反跟踪技术对于逆向工程者是非常有用的知识。程序总是有办法检测到在调试器中运行--- 同样适用于虚拟或模拟环境，但是既然 `ring3` 级调试器是已知最常见使用的分析工具，了解常见的反调试技巧，以及怎么样通过它们事实证明是很有用的。

> **[4] 链接**

> `MSDN`
>
> `Portable Executable Tutorial`, `Matt Pietrek`
>
> `Syscall Reference`, `The Metasploit Project`
>
> `Undocumented Functions for MS Windows NT/2K`
>
> `Intel Manuals`
>
> - Common exception codes - `Microsoft Windows SDK`, `ntdll.h`
> - Status codes list (including common exception codes) - `Microsoft Windows DDK`, `ntstatus.h`
> - Context Structures documentation - `Microsoft Windows SDK`, `ntdll.h`

> **[5] Data reference**

> - `CONTEXT` structure for `IA32` processors
>
> ```
> struct CONTEXT_IA32 {
> // ContextFlags must be set to the appropriate CONTEXT_* flag
> // before calling (Set|Get)ThreadContext
> DWORD ContextFlags;
> // CONTEXT_DEBUG_REGISTERS (not included in CONTEXT_FULL)
> DWORD Dr0; // 04h
> DWORD Dr1; // 08h
> DWORD Dr2; // 0Ch
> DWORD Dr3; // 10h
> DWORD Dr6; // 14h
> DWORD Dr7; // 18h
> // CONTEXT_FLOATING_POINT
> FLOATING_SAVE_AREA FloatSave;
> // CONTEXT_SEGMENTS
> DWORD SegGs; // 88h
> DWORD SegFs; // 90h
> DWORD SegEs; // 94h
> DWORD SegDs; // 98h
> // CONTEXT_INTEGER
> DWORD Edi; // 9Ch
> DWORD Esi; // A0h
> DWORD Ebx; // A4h
> DWORD Edx; // A8h
> DWORD Ecx; // ACh
> DWORD Eax; // B0h
> // CONTEXT_CONTROL
> DWORD Ebp; // B4h
> DWORD Eip; // B8h
> DWORD SegCs; // BCh (must be sanitized)
> DWORD EFlags; // C0h
> DWORD Esp; // C4h
> DWORD SegSs; // C8h
> // CONTEXT_EXTENDED_REGISTERS (processor-specific)
> BYTE ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];
> };
> ```

> - Process Environment Block structure (from The Wine Project)
>
> ```
> struct PEB {
> BOOLEAN InheritedAddressSpace; // 00
> BOOLEAN ReadImageFileExecOptions; // 01
> BOOLEAN BeingDebugged; // 02
> BOOLEAN SpareBool; // 03
> HANDLE Mutant; // 04
> HMODULE ImageBaseAddress; // 08
> PPEB_LDR_DATA LdrData; // 0c
> RTL_UPROCESS_PARAMETERS *ProcessParameters; // 10
> PVOID SubSystemData; // 14
> HANDLE ProcessHeap; // 18
> PRTL_CRITICAL_SECTION FastPebLock; // 1c
> PVOID /*PPEBLOCKROUTI*/ FastPebLockRoutine; // 20
> PVOID /*PPEBLOCKROUTI*/ FastPebUnlockRoutine; // 24
> ULONG EnvironmentUpdateCount; // 28
> PVOID KernelCallbackTable; // 2c
> PVOID EventLogSection; // 30
> PVOID EventLog; // 34
> PVOID /*PPEB_FREE_BLO*/ FreeList; // 38
> ```



# PEB Structure

```c
ULONG TlsExpansionCounter; // 3c
PRTL_BITMAP TlsBitmap; // 40
ULONG TlsBitmapBits[2]; // 44
PVOID ReadOnlySharedMemoryBase; // 4c
PVOID ReadOnlySharedMemoryHeap; // 50
PVOID *ReadOnlyStaticServerData; // 54
PVOID AnsiCodePageData; // 58
PVOID OemCodePageData; // 5c
PVOID UnicodeCaseTableData; // 60
ULONG NumberOfProcessors; // 64
ULONG NtGlobalFlag; // 68
BYTE Spare2[4]; // 6c
LARGE_INTEGER CriticalSectionTimeout; // 70
ULONG HeapSegmentReserve; // 78
ULONG HeapSegmentCommit; // 7c
ULONG HeapDeCommitTotalFreeTh; // 80
ULONG HeapDeCommitFreeBlockTh; // 84
ULONG NumberOfHeaps; // 88
ULONG MaximumNumberOfHeaps; // 8c
PVOID *ProcessHeaps; // 90
PVOID GdiSharedHandleTable; // 94
PVOID ProcessStarterHelper; // 98
PVOID GdiDCAttributeList; // 9c
PVOID LoaderLock; // a0
ULONG OSMajorVersion; // a4
ULONG OSMinorVersion; // a8
ULONG OSBuildNumber; // ac
ULONG OSPlatformId; // b0
ULONG ImageSubSystem; // b4
ULONG ImageSubSystemMajorVersion; // b8
ULONG ImageSubSystemMinorVersion; // bc
ULONG ImageProcessAffinityMask; // c0
ULONG GdiHandleBuffer[34]; // c4
ULONG PostProcessInitRoutine; // 14c
PRTL_BITMAP TlsExpansionBitmap; // 150
ULONG TlsExpansionBitmapBits[32]; // 154
ULONG SessionId; // 1d4
};
```

# TEB Structure

Thread Environment Block structure (from The Wine Project)

```c
struct TEB
{
    NT_TIB Tib; // 000 Info block
    PVOID EnvironmentPointer; // 01c
    CLIENT_ID ClientId; // 020 PID,TID
    PVOID ActiveRpcHandle; // 028
    PVOID ThreadLocalStoragePointer; // 02c
    PEB *Peb; // 030
    DWORD LastErrorValue; // 034
    ULONG CountOfOwnedCriticalSections; // 038
    PVOID CsrClientThread; // 03c
    PVOID Win32ThreadInfo; // 040
    ULONG Win32ClientInfo[0x1f]; // 044
    PVOID WOW32Reserved; // 0c0
    ULONG CurrentLocale; // 0c4
    ULONG FpSoftwareStatusRegister; // 0c8
    PVOID SystemReserved1[54]; // 0cc
    PVOID Spare1; // 1a4
    LONG ExceptionCode; // 1a8
    BYTE SpareBytes1[40]; // 1ac
    PVOID SystemReserved2[10]; // 1d4
    DWORD num_async_io; // 1fc
    ULONG_PTR dpmi_vif; // 200
    DWORD vm86_pending; // 204
    DWORD pad6[309]; // 208
    ULONG gdiRgn; // 6dc
    ULONG gdiPen; // 6e0
    ULONG gdiBrush; // 6e4
    CLIENT_ID RealClientId; // 6e8
    HANDLE GdiCachedProcessHandle; // 6f0
    ULONG GdiClientPID; // 6f4
    ULONG GdiClientTID; // 6f8
    PVOID GdiThreadLocaleInfo; // 6fc
    PVOID UserReserved[5]; // 700
    PVOID glDispachTable[280]; // 714
    ULONG glReserved1[26]; // b74
    PVOID glReserved2; // bdc
    PVOID glSectionInfo; // be0
    PVOID glSection; // be4
    PVOID glTable; // be8
    PVOID glCurrentRC; // bec
    PVOID glContext; // bf0
    ULONG LastStatusValue; // bf4
    UNICODE_STRING StaticUnicodeString; // bf8
    WCHAR StaticUnicodeBuffer[261]; // c00
    PVOID DeallocationStack; // e0c
    PVOID TlsSlots[64]; // e10
    LIST_ENTRY TlsLinks; // f10
    PVOID Vdm; // f18
    PVOID ReservedForNtRpc; // f1c
    PVOID DbgSsReserved[2]; // f20
    ULONG HardErrorDisabled; // f28
    PVOID Instrumentation[16]; // f2c
    PVOID WinSockData; // f6c
    ULONG GdiBatchCount; // f70
    ULONG Spare2; // f74
    ULONG Spare3; // f78
    ULONG Spare4; // f7c
    PVOID ReservedForOle; // f80
    ULONG WaitingOnLoaderLock; // f84
    PVOID Reserved5[3]; // f88
    PVOID *TlsExpansionSlots; // f94
};
```

# NtGlobalFlags

```
FLG_STOP_ON_EXCEPTION 0x00000001
FLG_SHOW_LDR_SNAPS 0x00000002
FLG_DEBUG_INITIAL_COMMAND 0x00000004
FLG_STOP_ON_HUNG_GUI 0x00000008
FLG_HEAP_ENABLE_TAIL_CHECK 0x00000010
FLG_HEAP_ENABLE_FREE_CHECK 0x00000020
FLG_HEAP_VALIDATE_PARAMETERS 0x00000040
FLG_HEAP_VALIDATE_ALL 0x00000080
FLG_POOL_ENABLE_TAIL_CHECK 0x00000100
FLG_POOL_ENABLE_FREE_CHECK 0x00000200
FLG_POOL_ENABLE_TAGGING 0x00000400
FLG_HEAP_ENABLE_TAGGING 0x00000800
FLG_USER_STACK_TRACE_DB 0x00001000
FLG_KERNEL_STACK_TRACE_DB 0x00002000
FLG_MAINTAIN_OBJECT_TYPELIST 0x00004000
FLG_HEAP_ENABLE_TAG_BY_DLL 0x00008000
FLG_IGNORE_DEBUG_PRIV 0x00010000
FLG_ENABLE_CSRDEBUG 0x00020000
FLG_ENABLE_KDEBUG_SYMBOL_LOAD 0x00040000
FLG_DISABLE_PAGE_KERNEL_STACKS 0x00080000
FLG_HEAP_ENABLE_CALL_TRACING 0x00100000
FLG_HEAP_DISABLE_COALESCING 0x00200000
FLG_VALID_BITS 0x003FFFFF
FLG_ENABLE_CLOSE_EXCEPTION 0x00400000
FLG_ENABLE_EXCEPTION_LOGGING 0x00800000
FLG_ENABLE_HANDLE_TYPE_TAGGING 0x01000000
FLG_HEAP_PAGE_ALLOCS 0x02000000
FLG_DEBUG_WINLOGON 0x04000000
FLG_ENABLE_DBGPRINT_BUFFERING 0x08000000
FLG_EARLY_CRITICAL_SECTION_EVT 0x10000000
FLG_DISABLE_DLL_VERIFICATION 0x80000000
```
