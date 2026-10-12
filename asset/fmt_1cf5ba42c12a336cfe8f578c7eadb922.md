
4) 保存 DLL 中的所有输出函数的原始入口点，可以在 `IMAGE_EXPORT_DIRECTORY.ED_AddressOfFunctions.` 里找到它们。

5) `N` = 由 `kernel32` 输出的 API 的总数，位于 `IMAGE_EXPORT_DIRECTORY.ED_NumberOfFunctions.`。

6) 将 `kernel32` 的每一个 API 转移到我们的缓冲区。

7) 存储对 DLL 的权限（一些壳用 DLL 激活异常）。

我们来详细看看如何将所有的 APIs 重定向到缓冲区，注意要在壳运行前完成。

我们称我们使用的缓冲区为 `DivBuffer`。该缓冲区大小为 `N*M`，`M` 是所有缓冲区中的最大值。我们要做的如下：

```
for (i=0; i<N; ++i){

    Change the entry point
    of the i-th API to DivBuffer[i*M].

    Generate random garbage
    and write it to DivBuffer[i*M].

    Write some instructions, after the
    garbage, to save the value of i.

    Write a jump, after the previous
    instructions, to our hooker procedure.
}
```

将 `i-th` 的 API 的入口点改变到 `DivBuffer[i*M]`。

产生随机垃圾并把它们写到 `DivBuffer[i*M]`。

在垃圾代码后写一些指令保存 `i` 的值。

在上面的指令后面写一跳转，转到我们的钩子程序。

算法中，`M` 只是垃圾代码大小的上界（对目前的壳而言，`M=30` 是个好的数值）。我们创建的所有的垃圾缓冲区会来到（lead to）钩子程序（hooker procedure），它负责记录调用和原料。

我们概述一下钩子，它是相当简单的程序：它保存寄存器，把一切写入日志文件，然后恢复寄存器，跳转到 API 原始入口点（我们已经保存）。钩子看起来像下面的：

```
hooker PROC
    mov eax, esp                  ; save esp and ebp
    mov ecx, ebp
    pushad
    ...
    ; 写日志文件
    ; (壳调用的 API，以用参数,…)
    ...
    ; 计算壳欲调用的 API 的实际入口点，用它覆盖下面的 dword
    ...
    ; 恢复寄存器
    popad
    ; 跳到 API 的实际入口点(是 push/ret)
    db 68h
    ; first opcode of push XXXXXXXXh
@ActualEP: db 0,0,0,0
    ret
hooker ENDP
```

可以观察到，在最开始，我们失去 `eax` 和 `ecx` 的值。没关系，对 Windows 而言，`eax` 和 `ecx` 都是“垃圾”寄存器，也就是说它们并不被 APIs 使用。因此，在内部使用它们是安全的（剩余的寄存器需要保留），这为我们节省了一些头痛事，因为我们能保存那里的 `esp` 和 `ebp`。不用说，垃圾代码需要保留（preserve）除 `eax` 和 `ecx` 之外的所有寄存器。

### C. 当壳调用某个 API 时，发生了什么？

壳认为它要做如下的事：

1) 定位 API 的映像基址
2) 以名字查找它，通常将该名字的固定编码（hardcoded）hash 值（hash value）与每个 API 的 hash 值相比较，直到匹配为止。
3) 模拟 API 的前 `K` 个指令（`K` 每次可以随机选取）
4) 跳转到第(`K+1`)-th 个 API 指令

实际上，它做：

1) 定位 API 的基址
2) 以名字查找，通常比较……
3) 模拟第 `i-th` 个缓冲区（假设它调用第 `i-th` API）的前 `K` 个指令
4) 跳转到第(`K+1`)-th 垃圾指令

最后，我们就能在我们的程序“记录器”的入口点挂钩 API 调用。

壳看起来并没有任何不同之处，但我们要让调用看起来它好像没有受到保护一样。（The packer doesn’t see any difference but we have kept the call as if it was done without the protection.）

### D. 操作提示

下列建议可能会帮助你轻松上手，编写自己的记录器。

- `Kernel32`、`user32` 和 `advapi32` 加载于运行在同一操作系统上的所有进程中的相同的映像基址上。因此，记录器可以从载体程序上继承（实际上是全部的）APIs。
- 要测试自己的程序，不必从注入代码到另一进程开始。在此之前，推荐在载体内分配一个缓冲区，将记录器拷贝到那里，进行测试。这样，你就能做个最小化测试（minimum test）。
- 当最终注入代码到另一进程，你就能在记录器的开始处设置断点。这样一来，当你用 `CreateRemoteThread` 运行它时，它就会蹦溃（crash），你有机会附载（attach）你的调试器。总的来说，你可以在你想检查的记录器的任意一点设一个 `int3` 断点。
- 写到日志文件的步骤：
  - 写入目标调用的 APIs 的名字
  - 做一个从十六进制到可打印字符串的小步骤（记住偏移量是以 endian order 形式给出的）
    （Do a small procedure passing from hexadecimal to a printable string (remember that offsets are given in endian order).）
  - 使用映射你的日志文件的共享文件，以便你不用从内存中抓取（dumping）就可以随时看到并保存它。从而使你控制可执行程序。
  - 记住文件映射不能在内存中扩大，用个大点的（`1Mb`）。
- 我们推荐（作为无版权目标程序）Yoda 的加密器作实验。它重定向 API 调用，有少数你必须绕过的异常或小把戏。

### 五、记录日志和 IAT 重建

看了壳（或目标程序）是如何完成挂钩所有的调用后，我们现在转到如何对这些调用采取行动。我们要解决下面这些主题：

1) 记录 API 调用到日志
2) 改变它们的参数或返回值
3) 重新激活调用，以获取（plaintext）API 地址

我们用于此次实验的壳是市面上最强壳之一。当然，我们不会泄露有助于破掉它的任何信息（所有的偏移量和原料都已修改）。

让我们一步一步回顾在现实中如何去做：

#### A. 记录 `kernel32` 日志

首先，一直要做的是记录所有指向 `kernel32` 的调用，仅仅少数从其他 DLLs 调用 APIs（不包括后面的 `ADVAPI32`）。

在日志里，你要有 API 和它从何处调用（返回值是在 `dword ptr [esp]`）。注意你的 API 可以来自同一（或其他的）DLL 的其他 APIs 里面调用，所有你就要以某种方式过滤掉这个调用。

我们按如下的做：

```asm
mov ebx, dword ptr [esp]        ; take return address
shr ebx, 28                     ; keep only the most
                                  ; significative byte
test ebx, ebx
jnz Dont_Log_Me
```

这样做是因为在 WinNT 里，`kernel32` 总是加载于 `77E40000h`。有很多更好的方法过滤它们，其中查看位于 `PEB`（译者：是什么？）的已加载模块是否匹配可能是最好的方法，但这很可靠又十分简单。我们看看来自被保护的记事本的日志：

```
Logger started for DLL KERNEL32.DLL,
target = Notepad.exe

VirtualAlloc From: 00B10024
Param: Buffer size: 00000200 API return: 00A70000

VirtualAlloc From: 00B20101
Param: Buffer size: 00001000 API return: 00A80000

LoadLibraryA From: 00A20BFE
Param: ADVAPI32.DLL <=== interesting!

VirtualFree From: 00A2310B ...
GetLocalTime From: 00A45150 <=== interesting!
...

VirtualFree From: 00D01711 VirtualFree From:00D02224
```

现在，你必须阅读了它调用的 APIs 并注意最相关的 APIs。我们单独隔离出 `kernel32.GetLocalTime` 是因为它典型的用于 30 天试用期或之类的。注意到壳加载了 `ADVAPI32`，这也很有趣，因此，你现在应记录来自 `ADVAPI32` 的所有调用。

只要去阅读一下日志，你就会精确地明白它是如何工作的。

#### B. 记录某些选定 APIs 参数日志

开始编写大量汇编代码去对成百上千的 APIs 的参数进行解码，将是毫无意义的。相反，我们只需选取调用的 APIs 中一小部分进行处理。

可以在 `esp+4`，`esp+8`，…找到参数，因此，你仅需把它们写到记录器里（小心，因为它们是指针，`NULL` 会毁掉你的记录器）。

#### C. 修改 API 调用的结果

本例中，我们选择的 API 是 `kernel32.GetLocalTime`。我们想钩住它，并改变它的返回值为一固定日期（于是我们就一直注册了）。要这样做，我们需要在调用前知道参数 `lpSystemTime` 的值，参见 `[win32hlp]`，并在调用后立刻修改它指向的数据结构。结果，除写入日志外，钩子还需保存 `lpSystemTime`。下面的代码演示如何进行完整的过程：

```asm
;--------------------------------------------------------; hooking the return address from the call
;--------------------------------------------------------
; first, we save the bytes at the return address because we
; are going to overwrite them with a jump to our code
cld                             ; clear direction flag
mov esi, dword ptr [esp]        ; take return
mov edi, offset your_buffer     ;
mov ecx, 6                      ; the size of a push/ret
repnz stosb
                                ;
; next we overwrite them with a push/ret leading to our code
mov edi, dword ptr [esp]        ; take return address
mov al, 68h                     ; write the push
stosb
lea eax, [My_GetLocalTime]      ; write the address of my
                                ; procedure
stosd
mov al, 0C3h                    ; write the ret
stosb
; we also need to keep track of lpSystemTime
mov eax, dword ptr [esp+4]   ; store the parameter lpSystemTime
mov dword ptr [ebp+lpSystemTime], eax
```

现在，我们让目标程序完成对 `kernel32.GetLocalTime` 的调用，因为它已经钩住我们的代码了。注意，目标程序将对它所有的区块具有读/写权限，所以你对此不必担心。现在，我们要改变返回值：

```asm
;--------------------------------------------------------
; Changing the return to our fake value
;--------------------------------------------------------
; The target jumps here when hooked:
My_GetLocalTime PROC
    pushad
    pushf             ; not needed in this case but you might have
                      ; to add it too
    ; compute the delta handle in ebp
    call @@1
@@1:
    pop ebp
    sub ebp, @@1
    ; modify the returned structure
    mov eax, dword ptr [ebp+lpSystemTime]     ; point to the
                                              ; SYSTEMTIME structure
    mov [eax.wYear], 2004
    mov [eax.wMonth], 4
    mov [eax.wDayOfWeek], 1
    mov [eax.wDay], 8

    ; write back the bytes at the return instruction
    cld
    lea esi, [ebp+your_buffer]
    lea edi, [ebp+API_ReturnAddress]    ; the value we had at esp
                                          ; at the hooker
    mov ecx, 6
    repnz stosb

    ; restore registers and flags and return
    popf
    popad
    ret
My_GetLocalTime ENDP
```

这使壳相信我们仍旧是在 2004 年 4 月 8 号。当然，记录 API 返回值日志同存储 `eax` 和 API 目前使用的所有结构一样简单。

#### D. 处理剩余的 DLLs

本例中，我们知道壳加载了 `ADVAPI32.DLL`（对 `kernel32` 的一次考查就带给了我们所有加载的 DLLs）。`ADVAPI32` 是个相当重要的 DLL，它将我们所需的 APIs 包含到注册表，那是壳用来保存它们的注册信息的地方。在两种可能性：

1) 总结自己的算法：挂钩 `LoadLibraryA`、在每次调用时，查看哪个 DLL 被加载，同时挂钩它（译者：DLL）所有的 APIs。
2) 简单地对新 DLL 运用我们现有的算法

第一种方法没有很益处，事实上，在实验中从未用过它。

#### E. IAT 单一地址嗅探

观察下列由壳完成的调用：

```
LoadLibraryA From: 00BA08BC Param: COMDLG32.DLL
```

壳从不使用来自 `COMDLG32.DLL` 的 APIs，因此，要重建目标的 IAT，就必须完成此调用。让我们挂钩来自 `COMDLG32` 所有对 APIs 的调用，然后看看会发生什么：

```
Logger started for DLL   COMDLG32
End of log file
```

一个空的日志文件？为什么？我们只是简单的运行目标程序，但我们仍不得不强迫它从 `COMDLG32` 调用一些 API。现在，我们运行目标，但还要从菜单选择“选择字体”（你要用用目标程序，直到你在日志里看见一些有趣的东西，这也是我们为什么推荐创建一个共享文件作为日志映像文件）。

```
Logger started for DLL COMDLG32
CommDlgExtendedError Return Address: 01002E39
End of log file
```

好的，有了。我们想想当前链接三个最常见的可能，它们通过下面链接到 API：

- `call dword ptr [IAT_ENTRY]`: where `IAT_ENTRY` is an absolute address inside the IAT.
- `call RELATIVE_IAT_ENTRY`: Here, the return address + `RELATIVE_IAT_ENTRY` is inside the IAT (adjust for negative references).

在本例，`NotePad.exe` 中，这些是：
