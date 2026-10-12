
这里并不会像 `Ollydbg` 那样统计次数。一般这个指令用于单步步入，类似的还有 `P`，单步步过。较有特色的命令是 `PC` 和 `TC`，到 `CALL` 就自动暂停。当然 `WinDbg` 也有能统计的命令。

`wt [WatchOptions] [= StartAddress] [EndAddress]`

跟踪，并显示统计结果。

`WatchOptions` 可缺省，可选参数如下

`-l num` 跟踪深度限制,例如限制跟踪深度为 10 则 `wt -l10 00402312`

`-nc` 不显示单独 `CALL` 的信息

`-ns` 不显示累计信息

`-nw` 不显示在跟踪过程中的警告信息

`EndAddress` 终点地址。当 `wt` 在模块或者函数入口点，此项可缺省。

现在我们来看看它的显示效果（从帮助文件中复制过来的:P）

```
0:000> wt                       函数的开始，现在使用"wt"

Tracing MyModule!myFunction to return address 00401137

 141 [   0]   MyModule!myFunction
  20 [   1]    MyModule!anotherFunction
   5 [   2]      MyModule!deeperFunction
  10 [   1]    MyModule!anotherFunction
   3 [   2]      MyModule!deeperFunction
  30 [   1]    MyModule!anotherFunction
   4 [   0]   MyModule!myFunction

147 Instructions were executed 146(0 from other threads) traces 147 sums

Function Name             Invocations  MinInstr  MaxInstr  AvgInstr

MyModule!deeperFunction         2           3           5           4
MyModule!anotherFunction        1          68          68         68
MyModule!myFunction             1         213         213        213

0 system calls were executed

System Call:
```

### 3. 调试示例

首先要说明的是 `WinDbg` 中的某些指令是区分大小写的。最后我将以实际调试过程来说明 `Ollydbg` 与 `WinDbg` 的差别。先介绍一下目标软件，`black light beta` 版。自动查 `ROOTKIT`，使用修改文件名并重新启动系统的方法清除 `ROOTKIT`。国庆节到期，爆破时间限制不难，设断改跳转就可以了，如果写这个，就没什么好写了。用 `PEID` 查了一下，没有壳，有 `TLS` 表。这个才是这次调试的主角，我希望通过调试来增强对 `TLSCALLBACK` 的了解。

先使用 `Ollydbg` 载入目标程序，如果你的设置是首次暂停在 `WINMAIN`，并且没有使用任何关于 `TLS` 的增强插件，那么你会看到窗口出来了，但是 `Ollydbg` 却提示“进程已经终止，退出代码 0”。

现在用 `WinDbg` 载入程序，选中“debug child processes also”。程序一开始停在系统断点。输入 `g $exentry`。没断成功，落在子进程的系统断点。再次输入 `g $exentry`，产生了一个非法访问的错误。`gN $exentry`，终于能断在 `EP` 了。好了，如果你喜欢用这个软件，你可以爆破它的时间限制了。

`Ollydbg` 与 `WinDbg` 的第一次较量，在没有任何插件的情况，`WinDbg` 由于支持子进程调试，使得它在这个特例里要简单一些。

现在我们可以知道，程序在入口点之前就已经被运行，很可能是 `TLSCALLBACK`，并在建立新进程后退出。

#### 关于多进程调试

```
0:000> |                     ;$$查询当前进程数,注意 0:000 表示当前调试的进程的识别号是 0

.   0  id: 46c  create  name: beta.exe
#   1  id: 410  child   name: beta.exe

0:000> | 1 s               ;$$切换到子进程 1。S 是切换参数，当前状态是默认显示的。

eax=00000000 ebx=7ffdf000 ecx=00010101 edx=ffffffff esi=00000000 edi=00000000
eip=0041e3ec esp=0012ffc4 ebp=0012fff0 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246

beta+0x1e3ec:
0041e3ec 6a60              push    60h

1:001>                    ;$$切换之后命令提示符变成 1:001 了。
```

#### 回到正题

现在我们需要调试 `TLS` 处的代码。回到 `Ollydbg`，我用的插件是 `shoooo` 大大的 `WMOS`，能显示 `TLSCALLBACK` 的首址。为了方便每次都停在 `TLSCALLBACK`，我在入口处设了硬断，重新运行之后，我们可以开始调试了。

```
0040C7C0    8B4424 08       mov eax,dword ptr ss:[esp+8]        一开始便从初试栈中读取数据
0040C7C4    83EC 54         sub esp,54
0040C7C7    56              push esi                初始栈-58
0040C7C8    BE 01000000     mov esi,1
0040C7CD    3BC6            cmp eax,esi           比较初始栈+8 是否为 1
0040C7CF    0F85 E4010000   jnz blbeta.0040C9B9             没有跳转
0040C7D5    53              push ebx                初始栈-5c
0040C7D6    8D4424 64       lea eax,dword ptr ss:[esp+64]         EAX=初始栈+8
0040C7DA    50              push eax                此处是 0040c7e2 的参数
0040C7DB    FF15 ACA34300   call dword ptr ds:[<&KERNEL32.GetCommandLineW>;
0040C7E1    50              push eax
0040C7E2    FF15 60A44300   call dword ptr ds:[<&SHELL32.CommandLineToArg>;
0040C7E8    8BD8            mov ebx,eax                 EBX=参数首址
0040C7EA    397424 64       cmp dword ptr ss:[esp+64],esi      参数长度与 1 比较
0040C7EE    76 19           jbe short blbeta.0040C809          跳了
```

#### 如果没有跳会怎么样？

```
0040C7F0    8B4B 04         mov ecx,dword ptr ds:[ebx+4]
0040C7F3    68 74364400     push blbeta.00443674                  ; UNICODE "/q"
0040C7F8    51              push ecx
0040C7F9    E8 B1290100     call blbeta.0041F1AF    从传入参数看，此 CALL 判断运行参数是否为/q
0040C7FE    83C4 08         add esp,8
0040C801    85C0            test eax,eax
0040C803    0F84 AF010000   je blbeta.0040C9B8      可能是参数/q 对应的功能模块
```

`Ollydbg` 的参考字符功能，基本可以猜到该程序能以参数启动，格式为“`/ 参数`”，长度不等于 1

#### 回到正题

虽然跳过了一大堆 `junk code` 不过我猜肯定不止那么少。

```
0040C8A8    8B0B            mov ecx,dword ptr ds:[ebx]
0040C8AA    51              push ecx
0040C8AB    E8 21200100     call blbeta.0041E8D1
0040C8B0    68 74364400     push blbeta.00443674                  ; UNICODE "/q"
0040C8B5    8BF0            mov esi,eax
0040C8B7    E8 15200100     call blbeta.0041E8D1  前面已经有处理/q 的例程了，这里还处理？忽略
0040C8BC    83C4 08         add esp,8
0040C8BF    6A 04           push 4
0040C8C1    8D6C06 04       lea ebp,dword ptr ds:[esi+eax+4]
0040C8C5    68 00100000     push 1000
0040C8CA    8D7C2D 00       lea edi,dword ptr ss:[ebp+ebp]
0040C8CE    57              push edi
0040C8CF    6A 00           push 0
0040C8D1    FF15 B0A34300   call dword ptr ds:[<&KERNEL32.VirtualAlloc>]  ; KERNEL32.VirtualAlloc
0040C8D7    8BCF            mov ecx,edi       这条指令跟下一条指令相当于 mov edx,edi
0040C8D9    8BD1            mov edx,ecx       到这里还有 junk code
0040C8DB    8BF0            mov esi,eax       又是一个 junk code
0040C8DD    C1E9 02         shr ecx,2
0040C8E0    33C0            xor eax,eax
0040C8E2    8BFE            mov edi,esi        相当于 mov edi,eax
0040C8E4    F3:AB           rep stos dword ptr es:[edi]
```

代码分析=体力活，本文重点介绍工具使用，而不是代码还原。现在忽略细节，直接看它用了什么 API。

`CALL KERNEL32.VirtualAlloc`

`Call KERNEL32.GetModuleFileNameW`

`call KERNEL32.CreateEventW`

`call KERNEL32.CreateProcessW` 其中|CommandLine = `""D:\blbeta.exe" /q"`

`call KERNEL32.WaitForSingleObject`

`call KERNEL32.CloseHandle`

`call KERNEL32.VirtualFree`

`call KERNEL32.ExitProcess`

`ExitProcess` 之后又来到一开始的地方了。

```
0040C7C0    8B4424 08       mov eax,dword ptr ss:[esp+8]
```
