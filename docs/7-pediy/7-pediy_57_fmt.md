
# keymaker 原理：INT 3 的插入

> 原文：[`www.pediy.com/kssd/pediy05/pediy50268.htm`](https://www.pediy.com/kssd/pediy05/pediy50268.htm)

记得上次问树袋熊大哥，大哥回答得挺详细，但我一直想试一试 `INT 3` 断点是如何插入的。经过两天的折腾，终于弄明白了，不敢独享，大哥见笑啦。下面是分析：

目标：英语会话精灵 2.0(`topbar.exe`)注册机

使用工具：`IDA4.15`, `Softice`

程序使用 `upx` 加壳，有改动！部分资源经过 `XOR` 加密（文件尾部）。废话少说，切入正题。

为节省篇幅，只分析其原理：（假设已经知道我们要插入断点的地址--即已经跟踪出注册码地址）

### 1. 使用 `CreateProcessA` 加载要调试的程序 `topbar.exe`，获得句柄 `hThread`

循环使用 `ReadProcessMemory` 读取 `topbar.exe` 进程地址 `addr1`（`47a057`），如果此处指令为我们预设指令，则使用子程序（`sub1`）修改此处指令为 `INT 3(CCH)`，并保存原指令（`5byte`）后等待中断发生。

**`sub1` 流程**

```asm
dwLength＝1Ch

lpBuffer－－>PMEMORY_BASIC_INFORMATION(dwLength=1C byte)
lpAddress-->addr1
hProcess-->

invoke VirtualQueryEx, hProcess, lpAddress, lpBuffer, dwLength (Kernel32.lib)

lpflOldProtect-->
flNewProtect=4;
dwSize-->1000h
lpAddress-->47a000
hProcess-->

invoke VirtualProtectEx, hProcess, lpAddress, dwSize, flNewProtect, lpflOldProtect

lpNumberOfBytesWritten=0
nSize=5
lpBuffer-->读取缓冲
lpBaseAddress-->addr1
hProcess-->

invoke ReadProcessMemory, hProcess, lpBaseAddress, lpBuffer, nSize, lpNumberOfBytesWritten

lpflOldProtect-->
flNewProtect=flOldProtect;
dwSize-->1000h
lpAddress-->47a000
hProcess-->

invoke VirtualProtectEx, hProcess, lpAddress, dwSize, flNewProtect, lpflOldProtect
```

### 2. 中断处理

中断发生后，由调试程序（注册机）接管，使用 `GetThreadContext` 取得 `topbar.exe` 的 `CONTEXT` 上下文。

读取 `CONTEXT` 结构的 `offset B8h` 即 `regEip` 的值，比较是否程序在此中断，此时 `regEip` 的值应为 `addr1 + 1`，将 `regEip` 减 1（即恢复 IP 指针），使用 `SetThreadContext` 保存修改了的 `CONTEXT` 结构。

指针是恢复了但被修改的指令并没有恢复，因此下一步就是恢复原来的指令：调用 `sub2`。

**`sub2` 流程**

```asm
dwLength＝1Ch

lpBuffer－－>PMEMORY_BASIC_INFORMATION(dwLength=1C byte)
lpAddress-->addr1
hProcess-->

invoke VirtualQueryEx, hProcess, lpAddress, lpBuffer, dwLength (Kernel32.lib)

lpflOldProtect-->
flNewProtect=4;
dwSize-->1000h
lpAddress-->47a000
hProcess-->

invoke VirtualProtectEx, hProcess, lpAddress, dwSize, flNewProtect, lpflOldProtect

lpNumberOfBytesWritten=0
nSize=5
lpBuffer-->写缓冲指向保存好的指令
lpBaseAddress-->addr1
hProcess-->

invoke WriteProcessMemory, hProcess, lpBaseAddress, lpBuffer, nSize, lpNumberOfBytesWritten

lpflOldProtect-->
flNewProtect=flOldProtect;
dwSize-->1000h
lpAddress-->47a000
hProcess-->

invoke VirtualProtectEx, hProcess, lpAddress, dwSize, flNewProtect, lpflOldProtect
```

### 3. 读取注册码

现在指令也恢复了，被调试程序可以正常运行了，完了吗？不对，忘了我们要干什么啦。好的，下一步就是读取注册码（假设我们已经知道注册码的位置，在此程序中我们修改的指令的上一条指令 `POP EAX` 的 `EAX` 中存放有注册码，见附源程序片段）：

使用 `GetThreadContext` 取得 `topbar.exe` 的 `CONTEXT` 结构上下文：

```asm
lpContext->CONTEXT 结构
ContextFlags＝10007h ; CONTEXT_FULL＝CONTEXT_CONTROL OR CONTEXT_INTEGER OR CONTEXT_SEGMENTS
invoke GetThreadContext, hThread, lpContext
```

此时 `CONTEXT` `->` `B0h` 即 `regEax` 即为注册码的地址，将其存入 `CONTEXT` `->` `B8h`，然后使用 `ReadProcessMemory` 读出注册码。

**`CONTEXT` STRUCT 定义**

```asm
CONTEXT STRUCT
    ContextFlags      DWORD       ?
    iDr0              DWORD       ?
    iDr1              DWORD       ?
    iDr2              DWORD       ?
    ...
    ...
    regEdx            DWORD       ?
    regEcx            DWORD       ?
    regEax            DWORD       ?       ;offset B0
    regEbp            DWORD       ?
    regEip            DWORD       ?       ;offset B8
    regCs             DWORD       ?
    regFlag           DWORD       ?
    regEsp            DWORD       ?
    regSs             DWORD       ?
    ExtendedRegisters db MAXIMUM_SUPPORTED_EXTENSION dup(?)
CONTEXT ENDS
```

### 附：`topbar.exe` 源程序片段

```asm
:0047A048 8B45F4                mov eax, dword ptr [ebp-0C]
:0047A04B 8D55F8                lea edx, dword ptr [ebp-08]
:0047A04E E89DE7F8FF            call 004087F0
:0047A053 8B55F8                mov edx, dword ptr [ebp-08]
:0047A056 58                    pop eax－－－》》》注册码
:0047A057 E8489EF8FF            call 00403EA4
:0047A05C 0F853D010000          jne 0047A19F
:0047A062 6840000400            push 00040040
:0047A067 B908A24700            mov ecx, 0047A208

* Possible StringData Ref from Code Obj ->"successs!"
                        |
:0047A06C BA18A24700            mov edx, 0047A218
```

结论：通过上面的分析，现在我就可以写出自己的注册机，实际上我已经写了。使用的就是看雪学院下载的 `win32asm` 教程第 28-29 篇的 `debug` 篇的一个小程序（只需稍加改动，加入我的代码即可）。当然，要想向树袋熊大哥一样写出那么漂亮的界面，那得下一翻大功夫。

如有想试牛刀者，Email to me `softdim@vip.sina.com`。英语会话精灵 2.0(`topbar.exe`)及其注册机文件都不大，是一个很好的学习例子。
