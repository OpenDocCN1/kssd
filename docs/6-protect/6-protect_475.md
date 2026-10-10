

# Anti-Anti-Dump and nonintrusive tracers（上）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-755.htm`](https://www.pediy.com/kssd/pediy08/pediy8-755.htm)

【文章标题】: 反--反转存和非侵入性跟踪器（上）  
【文章作者】: deroko/ARTeam；译者：kkbing/看雪论坛  
【作者声明】: 此文时看雪老大从ＡＲＥＴｅａｍ移入，见其有些价值拿来翻译。其中存在很多不足甚至是错误的地方，欢迎批评指正。特别感谢 aalloverred 的指正  
\--------------------------------------------------------------------------------  
【详细过程】  
  译文：         
  1. 摘要  
  2..必须的知识  
  2.1.自定位代码  
  2.2．找回 kernel32.dll 基址和 APIs  
  2.3. 内存补丁注入  
  2.4 混合钩子的方法  
  2.5 接下来呢？  
  3．内存管理  
  3.1．扩展程序空间  
  3.2．用 VirtualAlloc 和 VirtualFree 管理内存  
  3.3．Delphi 代码的问题  
  3.4．内存管理的结论  
  关键词：编码，钩子，反－转存，非入侵，内存管理  
  1．  摘要  
  现在有许多壳都是把代码分割并重定位到新的缓冲区来阻止转存。这样的缓冲区有时很难修复，从而许多人在这里放弃了。我可以列举几例，运用得就是这种方法，如：Armadillo, ASProtect SKE, krypton 等。到目前我们是运用 od 脚本或外面的工具去修复被分割的代码或被除去的输入地址表。然而我们并不知道他的思想方法，最终也学不到新东西。  
  我在这里展示一些思想方法，它已经被运用到解决以上的问题了，但是要理解它是有些困难的。首先你应该熟悉汇编，PE 格式和钩子。最好有些基础，因为在这篇文章里会涉及他们中的一些重要部分。  
  我想这些技巧还没有运用于 RCE，至少我还没见到。无论如何我会涉及到 RCE 的一些有趣的方面。  
  2．  必须的知识  
  如果你已知道所有的这方面材料。就可以进入第三部分了，因为我将覆盖自定位代码和向目标进程注入的一些基础知识。  
  2.1．自定位代码  
  自定位代码是一种可以在任何特定的内存区域中执行的代码。这样的代码不仅用在病毒和 shell 中，而且一些壳也用到。自定位代码的主要问题是访问数据的能力。因此，病毒作者用“delta”偏移去访问自身的变量。在这一点上我将走的很快，并直接展示代码以及如何在自定位代码中参考变量。  
                      call delta  
  delta:              pop ebp  
                      sub ebp, offset delta  
    
                      mov eax, \[ebp+kernel32\]  
    
                      call \[ebp+GetModuleHandleA\]  
  kernel32             dd     ?  
  GetModuleHandleA     dd     ?  
  可见，自定位代码中熟练操作数据并不困难。这里有一些规则：  
  ——ebp 经常用作 delta, esi, edi 和 ebx 也可能被用作 delta ,因为在调用 API 时，他们的值并不会改变。我一直用的是 ebp,因为 esi/edi 经常联合应用于数据的拷贝，而 ebx 被用来当作重要数据的指针（如：在病毒感染期间，可以用它指向任何我们所需的 PE 文件。）（译者注：call/pop/sub 三个指令的组合是经典的解决代码重定位的方法，几乎所有的病毒开始就是这三句）  
  以下是由 Super/29a 提出，稍后由 Benny/29a \[1\]描述的一个运用 delta 的一个技巧，可以使你编译的代码更小。  
                      call delta  
  delta:              pop ebp  
                      mov eax, \[ebp+kernel32-delta\]  
                      call \[ebp+GetModuleHandleA-delta\]  
  kernel32            dd   ?  
  GetModuleHandleA    dd   ?  
  事实上，这是一个非常好的技巧，但是我们现在不太多关心代码的大小.我们可以运用以前的方法，因为它在调试时的可读性强，并且在编写自定位代码时键入的更少。好了，我要告诉你们的关于自定位代码的就是这些了。不要只学习它的定义（你会找到许多关于它的），更重要的是它的原理。  
  2.2 重新得到 kernel32.dll 基址和 APIs  
  对于自定位代码来说，下一个重要的问题就是在 Win32 环境下调用 APIs。为了能这样做，我们需要定位 kernel32.dll 的基址。以下是一些我们可以完成它的方法。  
  -扫描 SEH  
  -由 Ratter/29a 提供的 PEB 技巧  
  -获得硬编码。（译者注：比如 Win2k 下一般是 77e60000h，WinXP SP1  
            是 77e40000h，SP2 是 7c800000h 等。但是这么做不具有通用性）  
  扫描 SEH---首先我们需要知道描述的 SEH(结构化异常处理)的数据结构表  
  kd> dt nt!\_EXCEPTION\_REGISTRATION\_RECORD  
  +0x000 Next       : 前一个 \_EXCEPTION\_REGISTRATION\_RECORD 结构  
  +0x004 Handler     : 异常处理回调函数地址  
  kd>  
  （译者注：这是 windbg 中的命令，用来获取结构的信息,很实用）  
    
  任何 SEH 链将会赋予一个指向 kernel32.dll 中某个值的句柄,当然,如果这是最后一个 EXCEPTION\_REGISTRATION\_RECORD，那么\_EXCEPTION\_REGISTRATION\_RECORD 结构的第一个参数名将被置为-1，这样我们就知道何时在 kernel32.dll 中获取地址了  
    
  getkernelbase:  
                  pushad  
                  xor edx, edx  
                  mov esi, dword ptr FS:\[edx\]  
  \_\_seh:         lodsd  
                  cmp eax, 0FFFFFFFFh  
                  je \_\_kernel  
                  mov esi, eax  
                  jmp \_\_seh  
  \_\_kernel:       mov edi, dword ptr\[esi + 4\]  
                  and edi, 0FFFF0000h  
  \_\_spin:         cmp word ptr\[edi\], 'ZM'  
                  jz \_\_test\_pe  
                  sub edi, 10000h  
                  jmp \_\_spin  
  \_\_test\_pe:      mov ebx, edi  
                  add ebx, \[ebx.MZ\_lfanew\]  
                  cmp word ptr\[ebx\],'EP'  
                  je \_\_exit\_k32  
                  sub edi, 10000h  
                  jmp \_\_spin  
  \_\_exit\_k32:     mov \[esp.Pushad\_eax\], edi  
                  popad  
                  ret  
    
  这个代码并不是最优化的，但是它展示了逻辑。扫描 SHE 直到这里等于-1，简单的获得句柄的地址和搜索 MZ 与 PE 符号。一旦我们找到了他们，我们就获了 kernel32.dll 的地址。  
  PEB 方法---这个方法是由 Ratter/29a 发现和提出的，我会给出例子，但是要想获得更多的解释请回到\2\  
    
  mov eax, dword ptr FS:\[30h\]  
  mov eax, dword ptr\[eax+0ch\]  
  mov eax, dword ptr\[eax+1ch\]  
  mov eax, dword ptr\[eax\]  
  mov eax, \[eax+8\]  
    
  首先我们找回 PEB 的值，接着找到 PEB\_LDR\_DATA，随后我们就进入到 InInitializationOrderModuleList，头一个 LIST\_ENTRY 指向 ntdll.dll ,再进入下一个表的入口，瞧，kernel32.dll 的基址我们就接收到了。  
    
  获得装入时硬编码的值----这个非常简单，并不需要太多的知识，我们用 GetModuleHandleA 去重新获得 kernel32.dll 的值并储存到偏移值中。  
  例码：  
             pushs <"kernel32.dll">  
             call GetModuleHandleA  
             mov kernel32, eax  
  loader:  
  ...  
  kernel32   dd   ?  
    
  重新获取 API 的地址也很简单，一旦我们获得了 kernel32.dll,我们就可以写出我们自己的 GetProcAddress，在 kerlnel32.dll 的输出表中查找我们所需的 APIs  
  输出表被装载在离 PE 头偏移 78h 处，它的结构如下：  
  typedef struct \_IMAGE\_EXPORT\_DIRECTORY {  
        DWORD Characteristics;  
        DWORD TimeDateStamp;  
        WORD MajorVersion;  
        WORD MinorVersion;  
        DWORD Name;  
        DWORD Base;  
        DWORD NumberOfFunctions;  
        DWORD NumberOfNames;  
        DWORD AddressOfFunctions; // 指向导出函数地址表的ＲＶＡ  
        DWORD AddressOfNames; // 指向函数名地址表的ＲＶＡ  
        DWORD AddressOfNameOrdinals; // 指向函数名序号表的ＲＶＡ  
  } IMAGE\_EXPORT\_DIRECTORY, \*PIMAGE\_EXPORT\_DIRECTORY;  
    
  在这个结构中有三个重要的成员：  
  DWORD AddressOfFunctions; //指向导出函数地址表的ＲＶＡ  
  DWORD AddressOfNames; //指向函数名地址表的ＲＶＡ  
  DWORD AddressOfNameOrdinals; //指向函数名序号表的ＲＶＡ  
  以下的表格将会表示的更加清晰。  
    
  array of name RVA          oridnals           array of functions RVA  
  +---------------+     +---------------+ 　　　　+---------------+  
  | RVA1 name | 　<----->  | ordinal1 |    <----->| RVA of API1 |  
  +---------------+ +   ---------------+ 　　　　　+---------------+  
  | RVA2 name | 　<----->  | ordinal2 |    <----->| RVA of API2 |  
  +---------------+ +-  --------------+ 　　　　　+---------------+  
  | RVA3 name | 　<----->  | ordinal3 |    <----->| RVA of API3 |  
  +---------------+ +-  --------------+ 　　　　　　+---------------+  
  | RVA4 name | 　<----->  | ordinal4 |    <----->| RVA of API4 |  
  +---------------+ +--  -------------+ 　　　　　　+---------------+  
  | RVA5 name | 　<----->  | ordinal5 |     <----->| RVA of API5 |  
  +---------------+ 　+--  -------------+ 　　　　　　+---------------+  
  我们先在 AddressOfNames 中搜寻我们所要 API 的名字会得到他的索引，然后再依此索引在 AddressOfFunctions 中得到 oridnal 的索引，接着就可以获得本程序中 API 的 RVA(相对虚拟地址)，最后加上 dll 的基址就是我们所需 API 的地址。就是这么简单（译者注：可以参见罗云彬的书，讲的比较详细）  
  为了缩减病毒和后来的 shellcodes 的大小，并逃脱扫描检测特殊字符串，病毒作者运用 hash 或 crc32 校验来寻找 APIs。最简单，迄今最快的 hashing algo 由 z0mbie 介绍，由 rol/xor 组成。（译者注：参见 29A-4.227）  
  \_\_1:   rol eax, 7 ;hash algo (x) by z0mbie  
            xor al, byte ptr \[edx\]  
            inc edx  
            cmp byte ptr \[edx\], 0  
            jnz \_\_1  
  2.3.内存补丁注入  
  内存补丁注入事实上是简单的装入，它将注入把我们的自定位代码注入目标进程，这样的注入为我们干一些卑鄙的工作（如：钩子），也许你会奇怪我为什么不用 dll 注入的方法。简单的说我不喜欢在一个文件夹下有两个源文件，而且作为病毒程序，除了独立偏移外我不喜欢任何其它的事情。记住，自定位代码是非常好的，而 dll 注入在 C 程序中很普遍。  
  好了，让我们来看看如何把自定位代码注入到目标中。首先我们用 CreateProcessA 创建一个挂起的进程,接着用 VirtualAllocEx 和 WriteProcessMemory 在目标中写我们的自定位代码。接着在目标的入口处储存钩子直到钩子到达。一旦我们成功装好钩子，则自定位代码就承担起钩子的任务，做所有卑鄙的事，储存原始字节并返回目标进程的入口点。  
                push offset pinfo  
                push offset sinfo  
                push 0  
                push 0  
                push CREATE\_SUSPENDED  
                push 0  
                push 0  
                push 0  
                push offset progy  
                push 0  
                callW CreateProcessA  
                push PAGE\_EXECUTE\_READWRITE  
                push MEM\_COMMIT  
                push 2000h  
                push 0  
                push pinfo.pi\_hProcess  
                callW VirtualAllocEx ;allocate big enough block  
                mov mhandle, eax  
                push 0  
                push 2  
                push offset infinite  
                push 401000h  
                push pinfo.pi\_hProcess  
                callW WriteProcessMemory ;store jmp $ at entry point  
                push pinfo.pi\_hThread  
                callW ResumeThread  
                mov ctx.context\_ContextFlags, CONTEXT\_FULL  
  \_\_cycle\_ep:  
                push 100h  
                callW Sleep  
                push offset ctx  
                push pinfo.pi\_hThread  
                callW GetThreadContext  
                cmp ctx.context\_eip, 401000h  
                jne \_\_cycle\_ep  
                push pinfo.pi\_hThread  
                callW SuspendThread  
                push 0  
                push size\_loader ;size of loader  
                push offset loader ;loader code  
                push mhandle ;allocated mem block  
                push pinfo.pi\_hProcess  
                callW WriteProcessMemory  
                push mhandle  
                pop ctx.context\_eip ;eip == my code  
                push offset ctx  
                push pinfo.pi\_hThread  
                callW SetThreadContext;set context  
                push pinfo.pi\_hThread  
                callW ResumeThread ;resume thread　  
  　　　　　　  
  ２.４混合 hooking 的途径  
  我们有两种下钩子的方法，但如果要对付壳的话，就只有一种选择了。下钩子的两种方法分别是 IAT hooking 和 APIs hooking.前者不是我们的选择，因为壳可以自动找到或运用 GetProcAddress 去找到所有的 APIs，所以是不实际的。更好的选择是第二种方法，它由在 API 入口处或结束处储存钩子组成，所以能控制它的输出。  
  让我们看一个 k32 中的 API：  
  .text:7C809A81 VirtualAlloc proc near  
  .text:7C809A81  
  .text:7C809A81            mov edi, edi  
  .text:7C809A83     push ebp  
  .text:7C809A84     mov ebp, esp  
  .text:7C809A86     push \[ebp+arg\_10\] ; flProtect  
  .text:7C809A89     push \[ebp+flProtect\] ; flAllocationType  
  .text:7C809A8C     push \[ebp+flAllocationType\] ; dwSize  
  .text:7C809A8F     push \[ebp+dwSize\] ; lpAddress  
  .text:7C809A92     push 0FFFFFFFFh ; hProcess  
  .text:7C809A94     call VirtualAllocEx  
  .text:7C809A99     pop ebp  
  .text:7C809A9A     retn 10h  
  .text:7C809A9A VirtualAlloc endp  
  .text:7C809A9D     db 90h  
  .text:7C809A9E     db 90h  
  我们就用这种方法去钩住 VirtualAlloc，这样一来它就只能调用我们的代码，并分配和释放内存。不同程序在 dll 中的输出都最终在 k32.dll 和 ntdll.dll 中结束并去调用 native APIs。所以，如果我们知道将要 hooking 的是什么，那么也可以调用随后的 API.。看一看 VirtualAlloc，它将调用 VirtualAllocEx，我们可以模仿这些，但没有必要储存和执行原始指令，这会使钩子太冗长。这也会节约很多时间，因为我们没有用 Length Disassemble Engine 去决定指令的长度。注意，运用 Length Disassemble Engine 和储存旧的字节并不难办到，但是对于这篇文章没有必要。当我们在 ret/retn 处 hooking 时，我们必须用 Length Disassemble Engine 去定位 ret/retn.我 hooking kernel32.dll 的方法是运行ＬＤＥ去寻找 ret/rent，并查看是否填充 nops。（译者注：此方法译者不很明白，所以比较生硬，建议参看原文和以下原代码）  
  让我们看一些代码：  
                      ;ebx - where to redirect  
                      ;edi - pointer to api  
                      ;esi - CMD\_RETN - hook api at ret/retn  
                      ; - CMD\_ENTRY - hook api at entry  
  CMD\_RETN     equ 1  
  CMD\_ENTRY   equ 2  
  hook\_    api: pusha  
                      lea ecx, \[ebp+dummy\]  
                      push ecx  
                      push PAGE\_EXECUTE\_READWRITE  
  　　　　　　　　　　 push 1000h  
                      push edi  
                      call \[ebp+VirtualProtect\]  
                      test esi, CMD\_ENTRY  
                      jz \_\_hook\_at\_ret  
                      mov ecx, edi  
                      mov al, 0e9h  
                      stosb  
                      add ecx, 5  
                      sub ebx, ecx  
                      mov \[edi\], ebx  
                      jmp \_\_exit\_hook  
                      ;most APIs are paded with nop, if no nop, fail!!!  
  \_\_hook\_at\_ret:   push edi  
                      call ldex86  
                      add edi, eax  
                      cmp byte ptr\[edi\], 0c3h  
                      je \_\_check\_ret  
                      cmp byte ptr\[edi\], 0c2h  
                      jne \_\_hook\_at\_ret  
                      cmp word ptr\[edi+3\], 9090h  
                      jne \_\_exit\_hook ;failed  
                      jmp \_\_hook\_api  
  \_\_check\_ret:   cmp dword ptr\[edi+1\], 90909090h  
                      jne \_\_exit\_hook  
  \_\_hook\_api:  mov ecx, edi  
                      mov al, 0e9h  
                      stosb  
                      add ecx, 5  
                      sub ebx, ecx  
                      mov \[edi\], ebx  
  \_\_exit\_hook:   popa  
                      Retn  
  就如同你在上面代码中所见，这就是混合的 hooking 方法，也是这篇文章在这方面所需要的一切。你将会理解，我什么一旦涉及内存管理和被保护程序区段重定位时就用这种方法。  
    
  2.5．下面是什么呢？   
   我将给你展示怎样在 ring3 上写内存管理以及如何为你的目标写非入侵式跟踪。  
    
  3.内存管理  
  也许你会问在反-反转存中为什么会用到内存管理。其实非常简单，我会用最简单的方式来讲解的。当你用一些保护性壳（如：aspr,armadillo）时，它会分配许多缓冲区，而原来此处的代码已经被转移了。由于这个事实，有时去转存是根本不可能的，在目标程序被解压释放时，这些大量的缓冲区完全是由它自己去分配和释放的。一个简单的方法是去钩住 VirtualAlloc,往后是 VirtualAllocEx，甚至是 ntdll.dll 中的 NtAllocateVirtualMemory，就可以返回去程序缓冲区，随后也就容易转存了。像这样的保护会重分配所有的数据在我们可能转存的范围内，这种情况下可以用 LordPE 的一个插件，尽管是我以前写的，但在这种情况下运用是很有魅力的。为了避免太多内存的消耗和对内存的分配和释放的跟踪，我将用一个相似的概念，在 Intel CPU 中被用来把虚拟内存转换成物理内存。我也可以用堆链表来组织内存管理，但这需要更多的代码。  
    
  3.1 扩张程序空间  
  假如说程序的大小是 9000h 字节，我们不能保证任何节区在重定位一些缓冲区时被重写。在这种况下，为了后来转存没有问题，我们需要在目标程序中分配缓冲区。我们将用内存补丁去增加程序的大小。  
  - 用 CreateProcessA 创建挂起状态的进程  
  - 用 ReadProcessMemory 读整个程序的内存  
  - 用 NtUnmapViewOfSection 释放目标程序中被用的内存（译者注：可参见 http://undocumented.ntinternals.net/UserMode/Undocumented%20Functions/NT%20Objects/Section/NtUnmapViewOfSection.html）  
  - 用 VirtualAllocEx 在目标程序中相同的基址处分配更多的缓冲区  
  - 用 WriteProcessMemory 写回原来的程序  
    
  样品代码：  
                      push offset pinfo  
                      push offset sinfo  
                      push 0  
                      push 0  
                      push CREATE\_SUSPENDED  
                      push 0  
                      push 0  
                      push 0  
                      push offset progy  
                      push 0  
                      callW CreateProcessA  
                      push PAGE\_READWRITE  
                      push MEM\_COMMIT  
                      push c\_size  
                      push 0  
                      push -1  
                      callW VirtualAllocEx  
                      mov esi, eax  
                      push 0  
                      push c\_size  
                      push esi  
                      push c\_start  
                      push pinfo.pi\_hProcess  
                      callW ReadProcessMemory  
                      push c\_start  
                      push pinfo.pi\_hProcess  
                      callW NtUnmapViewOfSection  
                      mov eax, c\_size  
                      add eax, NEW\_MEM\_RANGE  
                      push PAGE\_EXECUTE\_READWRITE  
                      push MEM\_COMMIT or MEM\_RESERVE  
                      push eax  
                      push c\_start  
                      push pinfo.pi\_hProcess  
                      callW VirtualAllocEx  
                      push 0  
                      push c\_size  
                      push esi  
                      push eax  
                      push pinfo.pi\_hProcess  
                      callW WriteProcessMemory  
  c\_start - base of progy  
  c\_size - size of progy  
  NEW\_MEM\_RANGE - increased size of our target  
    
  在这以后我们把自定位代码注入到了目标中，它将 hook VirtualAlloc 和 VirtualFree 并返回内存范围而不是新分配程序范围负责。但是真正的问题是如何写这样的内存管理。（译者注：作者是在程序尾添加区段，也可以在程序各区段之间，如果每个区段之间的空间都不够，还可以把自定位代码分开分别加入其中，CIH 就是这种方法）  
    
  3.2 用 VirtualAlloc and VirtualFree 管理内存  
  我们是否知道新的缓冲区被定位了呢？答案是肯定的，它就在我们原程序的结尾处。为了描述我们缓冲区每一页的状态，我也将用 4 字节大小的表来描述每页的状态（我将称它为 PTE）我们不得不时刻跟踪被 VirtualAlloc 分配的任何区域，因为一旦调用了 VirtualFree 我们可以在这页上做还未使用的记号，使得再次调用 VirtualAlloc 时可以返回。如果没有这些，我们甚至可能益出缓冲区，并导致页失败.首先用一个内存管理结构来描述被分配缓冲区的开始，范围和类型,与堆相似，但后来我会指出这样做太慢而且会导致内存泄漏。这时我们记得 Intel CPU 是如何把虚拟内存转变为物理内存的。在 PDE/PTE 中虚拟地址被用作索引，并包括物理结构和每页的状态，所以为什么不采用这样的技术和在 ring 3 级上做相同的事情和内存管理呢？它的速度很快，用页索引去访问存有每页状态的数据。  
  短暂的思考后，我就得到了我想要的，页的入口：（译者注：作者在 ring3 级上模仿虚拟内存转变为物理内存的方法非常有效，巧妙，请仔细阅读）  
  31     　　　　             2 　1 　0  
  +---------------------------+---+---+  
  | FIRST PAGE INDEX 　　　　  | R | P |  
  +---------------------------+---+---+  
  Ｐ－当前的位显示，页是否在用或空闲。  
  Ｒ－保存位，仅仅在 VirtualAlloc 以 MEM\_RESERVE 被调用。  
  FIRST PAGE INDEX 又称为ＦＰＩ－拥有第一页的索引，用来决定块大小。  
    
  缓冲区的格局如下：:  
  +--------------+----------------------------------+--------------+  
  | Progy 　　　 | VirtualAlloc/Free buffer 　　　　 | 　　　PTE   |  
  +--------------+----------------------------------+--------------+  
  你的 PTE 大小也许是=（buffer/1000h）\*4,在我这里，我分配了 1000 页并用 4 页去描述每页的状态。  
  为了获得 PTE，你应该获得每页的索引，而获得它是非常简单。假如我们的程序开始于 400000h，结束于 500000h，则程序的结尾就是我们缓冲区的开始，PTE 是被定位在我们缓冲区的最后 4 页  
           mov esi, \[edi+memstart\] ;esi=500000h  
           add esi, NEW\_MEM\_RANGE ;esi+1004000h  
           sub esi, 4000h ;structs for memory manager(PTEs)  
  memstart       = end of progy  
  NEW\_MEM\_RANGE = 1004000h ;1000\*1000 pages + 4000h for PTE  
  现在我们简单的来获取索引，假定 eax 是虚拟地址。  
            mov edx, eax  
            sub edx, \[ebp+memstart\] ;-500000h  
            shr edx, 0ch ;index into edx  
  瞧，用简单的\[esi+edx\*4\]就可以看见是否这页被分配了，被保留或可用。当然，这是我的执行，在你的执行中组织可以是不同的 PTE, 应该给 PTEs 分配足量的空间以来满足我们的要求。基本上我们用 4000h 来描述缓冲区 1000000h 字节的状态,难道这不美好？你不得不去喜欢 Intel 和他们把物理内存转换成虚拟内存的思想。当然，你可以分配更小的缓冲区和更小的 PTE,那将完全取决于你。  
  现在我将告诉你关于 FIRST PAGE INDEX 以及它为什么这么重要。我随后将给你展示如何在这样的缓冲区中运用 nonintrusive tracers，还是先告诉你 FPI.FPI 用来查看被分配缓冲区的大小和调用 VirtualFree 时释放缓冲区的大小。FPI 含有第一页的索引，并被置于任何一个 PTE 描述的某个范围。如果 FPI 是 1 in 3 PTEs ,这就意味着这个缓冲区开始于 PTE 的 INDEX 1 ，并且所有的拥有 FPI 1 的页都是相同缓冲区的一部分。这将稍后帮助我们编写 nonintrusive tracer 代码和释放内存，因为有时 VirtualFree 被作为 VirtualFree(page\_base, 0, MEM\_DECOMMIT)来调用，并且如果不知道缓冲区的大小会导致内存泄露。也许你奇怪我为什么不储存第一个 PTE 的大小并在接着的 PTEs 上做记号，而是去储存它们每个的 FPI。简单的说，FPI 将使我们知道 nonintrusive tracer 中必须改变的内存缓冲区的大小.如果异常在第三页发生，你仅仅只用在第三页改变保护，但是我们想在调用 VirtualAlloc 时改变整个范围的保护是，这就更有意义了。  
    
  看一段代码，我想你会理解  
  allocatememory      proc  
                      arg virtualbase  
                      arg range  
                      arg flags  
                      arg memprotection  
                      local numofpages:dword  
                      local dummy\_var:dword  
                      local virtualaddress:dword  
                      call deltaalloc  
  deltaalloc:         pop edi  
                      sub edi, offset deltaalloc  
                      mov esi, \[edi+memstart\]  
                      add esi, NEW\_MEM\_RANGE  
                      sub esi, 4000h ;structs for memory manager(PTEs)  
                      mov eax, range  
                      mov edx, eax  
                      shr eax, 0ch  
                      and edx, 0FFFh  
                      test edx, edx  
                      jz \_\_mm0  
                      inc eax  
  \_\_mm0:              mov numofpages, eax  
                      cmp virtualbase, 0  
                      jne \_\_commitpage ;commit reserved pages???? yep  
  ;find free block big enough and commit pages  
  ;starting from index 1  
                      mov ecx, 1  
  \_\_cycle\_empty:      test dword ptr\[esi+ecx\*4\], 1 ;committed?  
                      jnz \_\_next\_pte  
                      test dword ptr\[esi+ecx\*4\], 2 ;reserved?  
                      jz \_\_check\_size  
  \_\_next\_pte:         inc ecx  
                      cmp ecx, 1000h  
                      jne \_\_cycle\_empty  
  \_\_check\_size:       mov eax, numofpages  
                      add eax, ecx  
  \_\_cycle\_size:       dec eax  
                      test dword ptr\[esi+eax\*4\], 1  
                      jnz \_\_next\_pte  
                      test dword ptr\[esi+eax\*4\], 2  
                      jnz \_\_next\_pte  
                      cmp eax, ecx  
                      jne \_\_cycle\_size  
                      ;at this point we have found PTEs large enough to  
                      ;describe needed memory buffer  
                      mov eax, numofpages ;ecx is index used to get page  
                      add eax, ecx  
                      mov edx, ecx  
                      shl edx, 2 ;FPI  
                      mov ebx, flags ;1 for P or 2 for R  
  \_\_add\_pages:        dec eax  
                      mov dword ptr\[esi+eax\*4\], 0 ;set PTE to 0  
                      or dword ptr\[esi+eax\*4\], edx;set FPI  
                      or dword ptr\[esi+eax\*4\], ebx;set flags  
                      cmp eax, ecx  
                      jne \_\_add\_pages  
  \_\_done:             shl ecx, 0ch  
                      add ecx, \[edi+memstart\]  
                      mov virtualaddress, ecx  
                      jmp \_\_exitalloc  
  \_\_commitpage:       push virtualbase  
                      pop virtualaddress  
                      mov eax, virtualbase  
                      sub eax, \[edi+memstart\]  
                      shr eax, 0ch  
                      mov ecx, numofpages  
                      mov edx, eax  
                      shl edx, 2  
  \_\_commit\_em:        mov dword ptr\[esi+eax\*4\], 0 ;clear pte  
                      or dword ptr\[esi+eax\*4\], 3 ;flags  
                      or dword ptr\[esi+eax\*4\], edx ;fpi  
                      inc eax  
                      loop \_\_commit\_em  
  \_\_exitalloc:        mov eax, virtualaddress  
                      leave  
                      retn 10h  
                      endp  
  注意，如何从ＰＴＥ的１索引开始。  
  ; 从索引１开始  
            mov ecx, 1  
  这非常重要，空的 PTE 被我的 deallocatememory 设置为 0，随后被分配和释放的区域将会把 FPI 置零，在这种情况下，寻找 FPI 是 0 的内存范围将返回更大的范围。当然，我们可以检验 PTE 的 R 和 P 位，但是这会扩大代码并且使代码的可读性降低。现在，如果你仔细读这个源代码，你将理解在 ring 3 级上用 PTEs 写一个漂亮的内存管理代码是多么容易，VirtualFree 写起来也很简单。  
    
  deallocatememory:  
                   mov esi, \[ebp+memstart\]  
                   add esi, NEW\_MEM\_RANGE  
                   sub esi, 4000h ;pointer to PTE  
                   ;freeing using index and FPI to find all pages  
                   mov edx, eax  
                   sub edx, \[ebp+memstart\]  
                   shr edx, 0ch ;index into eax  
                   mov ecx, edx ;FPI into ecx  
                   mov eax, edx  
                   cmp eax, 1000h  
                   jnb \_\_exit\_free  
  \_\_freemem:       mov edx, \[esi+eax\*4\]  
                   shr edx, 2  
                   cmp edx, ecx ;FPI...  
                   jne \_\_exit\_free  
                   mov dword ptr\[esi+eax\*4\], 0 ;clear PTE  
                   inc eax  
                   jmp \_\_freemem  
  \_\_exit\_free:     retn  
  以上就是我要告诉你关于内存管理的一切．  
    
  3.3 用 Deiphi 编码的问题  
  在这种情况下，理解 Delphi 是非常重要的。ASProtect SKE 的 virtual.dll 是用 Delphi 写的，这是一个大的问题。我在玩弄了一会儿 ASProtect 和其他 Delphi apps 后，我的代码都失败了，甚至我用一切正确的方式去模仿。两个小时的跟踪后我就能够确定这样的问题了，这些就在下面。  
  - @System@@FreeMem  
  - @System@SysFreeMem  
  为了让这个引擎工作，程序必须成功返回 0，否则 Delphi app 会退出 。甚至我模仿任何事情，他都无故失败，所以唯一的方法是用以下两句程序去修补 aspr virtual.dll 或其他 Delphi 程序。  
  ----------  
  mov eax, 0  
  retn  
  接下来的问题是如何定位这两句程序？此时运用签名的方法在我脑海中闪过。立刻调用 VirtualAlloc，它将为 aspr virtual.dll 分配地址，所以可以储存地址，也可以插入非入侵调试，一旦我们下 int3h 断点时就可以获得。我们也就知道了扫描 signatures 的最佳时机。  
  更简的方法是在 virtual.dll 的入口处转存和在 IDA 中扫描这两句程序的地址。那将非常的方便和简单  
  .dumped:00496564 ; \_\_fastcall System::\_\_linkproc\_\_ FreeMem(void)  
  .dumped:00496564 @System@@FreeMem$qqrv proc near  
  .dumped:00496564  
  .dumped:00496564     test eax, eax  
  .dumped:00496566     jz short locret\_496572  
  .dumped:00496568     call ds:off\_4CE01C  
  .dumped:0049656E     or eax, eax  
  .dumped:00496570     jnz short loc\_496573  
  .dumped:00496572  
  .dumped:00496572 locret\_496572:  
  .dumped:00496572     retn  
  .dumped:00496573  
  .dumped:00496573 loc\_496573:  
  .dumped:00496573     mov al, 2  
  .dumped:00496575     jmp sub\_4965CC  
  .dumped:00496575 @System@@FreeMem$qqrv endp  
  And also:  
  .dumped:00497114 ; \_\_fastcall System::SysFreeMem(void \*)  
  .dumped:00497114 @System@SysFreeMem$qqrpv proc  
  .dumped:00497114  
  .dumped:00497114     var\_4 = dword ptr -4  
  .dumped:00497114  
  .dumped:00497114     push ebp  
  .dumped:00497115     mov ebp, esp  
  .dumped:00497117     push ecx  
  .dumped:00497118     push ebx  
  .dumped:00497119     push esi  
  .dumped:0049711A     push edi  
  .dumped:0049711B     mov ebx, eax  
  .dumped:0049711D     xor eax, eax  
  .dumped:0049711F     mov ds:dword\_4D042C, eax  
  .dumped:00497124     cmp ds:byte\_4D0428, 0  
  .dumped:0049712B     jnz short loc\_49714C  
  .dumped:0049712D     call @System@\_16436 ; System::\_16436  
  .dumped:00497132     test al, al  
  .dumped:00497134     jnz short loc\_49714C  
  .dumped:00497136     mov ds:dword\_4D042C, 8  
  .dumped:00497140     mov \[ebp+var\_4\], 8  
  .dumped:00497147     jmp loc\_4972AD  
  .dumped:0049714C  
    
  3.4 内存管理总结  
  如果你已理解上面的一切，那么试图反转存将会失败。你可以在任何一个转存范围内得到所有的东西。当然，为了能在 LordPE 中看见整个程序范围，你必须确定增加了 PEB 的大小。  
                               mov eax, dword ptr fs:\[30h\]  
                               mov eax, \[eax+0ch\]  
                               mov eax, \[eax+14h\]  
                               add dword ptr\[eax+18h\], NEW\_MEM\_RANGE  
  这就是所有在 ring3 上关于内存管理的知识 。我希望你获得其中的思想，并且能自己写出在你的目标中的内存管理。祝你好运。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 05 月 18 日 2:22:02  

\[翻译\]Anti-Anti-Dump\_and\_Nonintrusive\_tracers(下)  
  
\[出处及相关\]  
kanxue 的 ARTeam 团队优秀文章汇总帖:  
http://bbs1.pediy.com:8081/showthread.php?s=&threadid=24875  
原文地址:  
http://bbs1.pediy.com:8081/attachment.php?s=&attachmentid=1236  
上半部分地址(由 kkbing 翻译):  
http://bbs1.pediy.com:8081/showthread.php?s=&threadid=25001  
\[重要的废话\]  
实在不好意思贴出来,因为这篇文章着实的超出了我的能力,我只是一只小小小小鸟![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)  
因此大家就当帮忙找错误看好了,谢谢!只希望千万不要误人子弟就好了:)  
  
最后的附件是做成了 pdf 格式的译文，因为我是按照原文的格式翻译的，贴出来的可能看着不舒服.  
\[译者\]aalloverred  
  
\[译文\]  
  
  
4. 用于内存管理器的非侵入式跟踪器  
一旦我们利用代码强制性的将所有的东西存到了一个可以转存的地方，我们就可以准备访问这些地方了。一些保护壳会在分配的缓存中存储多态的 oep，所以我们还要将 eip 访问权交给这些缓存。有很多方法实现这一点，我这里要使用的是  
PAGE\_GUARD, 和我在文献\[6\]中的 oepfinder X.Y.Z 中介绍的方法一样。  
4.1. 编写非侵入式跟踪器  
有关非侵入式跟踪器，有很多文章中都有所涉及，而在文献\[5,6\]中讲解的尤其得多。但我还是要提一些比较重要的东西；不久以后，也就是你读完了这一部分以后，你就会体会到理解这些东西是多么的重要以及它们是多么的有用。我在\[5,6\]中已经讲述过非侵入式跟踪器和 loaders 了，所以这里我将只是简要的提及它们。  
非侵入式跟踪器的相关概念包括 KiUserExceptionDispatcher 的 hooking，及其我们自己处理所有的异常。如果我们处理异常，我们只是简单的调用 NtContinue ；如果我们不处理异常，我们就返回 KiUserExceptionDispatcher。我们先来看一个非侵入式跟踪器样板示例：  
nonintrusive: mov ecx, \[esp+4\]  
mov ebx, \[esp\]  
pushad  
call deltakiuser  
deltakiuser:  pop ebp  
sub ebp, offset deltakiuser  
...  
retkiuser0:   popad  
mov \[ecx.context\_dr0\], 0  
retkiuser:    push 0deadc0deh  
ret  
不要因为看到了 mov \[ecx.context\_dr0\], 0 这一句而困惑；当讲述道第 4.3 节的时候你自然就会理解它。  
现在我们先看一看 KiUserExceptionDispatcher 这个函数：  
.text:7C90EAEC mov ecx, \[esp+arg\_0\]  
.text:7C90EAF0 mov ebx, \[esp+0\]  
.text:7C90EAF3 push ecx  
.text:7C90EAF4 push ebx  
.text:7C90EAF5 call \_RtlDispatchException@8  
.text:7C90EAFA or al, al  
.text:7C90EAFC jz short loc\_7C90EB0A  
.text:7C90EAFE pop ebx  
.text:7C90EAFF pop ecx  
.text:7C90EB00 push 0  
.text:7C90EB02 push ecx  
.text:7C90EB03 call \_ZwContinue@8  
.text:7C90EB08 jmp short loc\_7C90EB15  
.text:7C90EB0A  
.text:7C90EB0A loc\_7C90EB0A:  
.text:7C90EB0A pop ebx  
.text:7C90EB0B pop ecx  
.text:7C90EB0C push 0  
.text:7C90EB0E push ecx  
.text:7C90EB0F push ebx  
.text:7C90EB10 call \_ZwRaiseException@12  
你可能注意到了这与我在\[6\]中所讲述的东西不是一样的么？别急，接着往下走，你会看到为什么会这样。我们将要 hook KiUserExceptionDispatcher 的前两条指令，注意如果我们要得到 stealth code 我们可以 hook \_RtlDispatchException 并将我们的跟踪器插入到那里。有无数种可能。我们 hook KiUserExceptionDispatcher 时必须模仿被覆盖的字节它们是：  
.text:7C90EAEC mov ecx, \[esp+4\]  
.text:7C90EAF0 mov ebx, \[esp\]  
ecx = 指向 CONTEXT  
ebx = 指向 EXCEPTION\_CODE  
我们可以通过检查的 ebx 的值轻松的决定我们是要处理这个异常还是要将控制权返回给 KiUserExceptionDispatcher：  
nonintrusive: mov ecx, \[esp+4\]  
mov ebx, \[esp\]  
pushad  
call deltakiuser  
deltakiuser:  pop ebp  
sub ebp, offset deltakiuser  
cmp dword ptr\[ebx\], EXCEPTION\_BREAKPOINT  
je \_\_bp\_conditions  
cmp dword ptr\[ebx\], EXCEPTION\_GUARD\_PAGE  
jne retkiuser0  
...  
retkiuser0:   popad  
mov \[ecx.context\_dr0\], 0  
retkiuser:     push 0deadc0deh <--- 在 hook 引擎中改变此值使其指向正确的值  
ret             
因为 KiUserExceptionDispatcher 是 ntdll.dll 的输出函数，我们可以简单的使用 GetProcAddress 定位它并且找到我们要 hook 的地址。同样注意，我一直使用的都是偏移地址，因为所有的代码都是在我注入的代码中执行的：  
mov eax, \[ebp+KiUserExceptionDispatcher\]  
lea ebx, \[ebp+nonintrusive\]  
mov byte ptr\[eax\], 0e9h  
mov ecx, eax  
add ecx, 5  
sub ebx, ecx  
mov dword ptr\[eax+1\], ebx  
add eax, 7  
mov dword ptr\[ebp+retkiuser+1\], eax  
有了这个钩子，你也许就可以肯定了，所有的异常都会经过你的钩子，你可以处理它们。当然非侵入式跟踪器也有弊端，因为这些理论不适用于两个进程比如当我们使用调试器/被调试程序的时候，因为这些异常可能会被调试器处理。另一方面，如果所有的异常都传递为 DBG\_EXCEPTION\_NOT\_HANDLED 我们的跟踪器就会毫无问题的工作。只是一个想法，但是为什么不 hook WaitForDebugEvent 并且将所有的异常传递为 DBG\_EXCEPTION\_NOT\_HANDLED 呢。只是设想，思考一下，读一下 5.3 章。   
  
非侵入式跟踪的第二个也是最大的一个缺陷就是处理堆栈时大量的垃圾代码。如果我们仔细的看看 Exception 异常的处理过程我们会看到 CONTEXT 和 Exception 代码由 ring0 拷贝到了用户堆栈。如果某些多态指令混淆了堆栈，拷贝就不会被执行，程序也就随之崩溃了。这是一种特别的情况，当我们单步跟踪代码的时候我们必须检查下一条指令是不是要改变堆栈，这样我们就可以阻止单步跟踪或者在我们的跟踪器中模仿这样的指令。这一点，我要推荐一下 z0mbie 的 XDE engine\[7\]。  
  
好了，希望你了解了基本思路。  
4.2. 非侵入式跟踪器中使用 PAGE\_GUARD  
PAGE\_GUARD 被用来提供简单好用的访问警告，但是也要注意到一旦异常发生了，PAGE\_GUARD 不会被移除。调试器 (也就是我们的非侵入是跟踪器) 应该在跳转到异常发生的地方执行前应该先改变页的的保护属性。我们也可以使用 PAGE\_NOACCESS，我们要使用这样或者那样的方法改变错误页的保护属性。但是使用 PAGE\_GUARD 的优点源于这样一个事实：我们同样知道了错误代码传给了我们的跟踪器，这是我们使用的 PAGE\_GUARD 唯一原因。同样也可以在保护器中使用对地址 0x00000000h 处的内存访问，通常代码是下面这个样子：  
xor eax, eax  
push offset sehhandle  
push dword ptr fs:\[eax\]  
mov dword ptr fs:\[eax\], esp  
mov \[eax\], eax <-- 异常  
我们必须手动确定异常是由于寄存器的错误使用而产生的，还是由于我们的页被设置成了 page\_noaccess（不可访问属性）而产生的。在 4.3 我还会涉及到如何区分寄存器的错误使用和读/写我们的内存区域。要设置 PAGE\_GUARD 我们得回到内存管理器部分并且将这个属性设置到一定的内存范围，使用下面的方法：  
\_\_setpageguard:   or memprotection, PAGE\_GUARD  
lea eax, dummy\_var  
          push eax  
          push memprotection  
          push range  
          push virtualaddress  
          call \[edi+VirtualProtect\]  
太简单了是吧? 这部分要存储在我们每个内存管理器的后面。很简单，现在你应该也明白了为什么所有的分配的内存缓冲区的都是以页为边界的。因为不管你怎么做都 VirtualProtect 将可访问的起始地址设为：  
Address to set access = address and 0FFFFF000h  
访问错误的页简单的使用 FPI，多亏有了 FPI，我们可以容易的找到 VirtualAddress 以及任何给定区域的大小：  
getfpi: push esi  
 mov esi, \[ebp+memstart\]  
     add esi, NEW\_MEM\_RANGE-4000h  
     sub eax, \[ebp+memstart\]  
     shr eax, 0ch ;获得当前页的索引  
     mov eax, \[esi+eax\*4\]  
     shr eax, 2 ;获得 FPI  
     pop esi  
     retn  
;eax=FPI？,返回页的基址  
getvafromfpi:   
     shl eax, 0ch  
     add eax, \[ebp+memstart\]  
     retn  
;eax FPI， 由 getfpi 中返回。本函数返回大小  
getsizefromfpi:  
     push esi ecx edx ebx  
     mov esi, \[ebp+memstart\]  
     add esi, NEW\_MEM\_RANGE-4000h  
     mov ecx, eax  
     xor ebx, ebx  
\_\_cycle\_fpi:   
     mov edx, \[esi+ecx\*4\]  
     shr edx, 2  
     cmp edx, eax ;比较各 FPI  
     jne \_\_gotsizefromfpi  
     inc ebx  
     inc ecx  
     jmp \_\_cycle\_fpi  
\_\_gotsizefromfpi:   
     shl ebx, 0ch  
     mov eax, ebx  
     pop ebx edx ecx esi  
     retn  
现在，要是你不喜欢我的方法，如何实现不同的内存管理也可以取决于你，完全取决于你，本文的主要目的只是想让你理解这个方法的思路。如果你正在像我一样思考，你应该会问了：怎样才能知道是由于对受保护页的执行引发了 EXCEPTION\_GUARD\_PAGE 还是对受保护页的读写引发了异常呢？不使用 ring0 这几乎是不可能的。我说几乎是因为如果肯定了是对受保护页的读写引发了异常，你就必须在你的内存管理器中跟踪受保护页的信息。   
4.3.特别情况下的 PAGE\_GUARD  
就像我在编写自己的小程序的时候一样。我意识到由 KiUserExceptionDispatcher 传回的数据已经不足以满足我的需要了。我的处境很可能是一个程序员能够碰到的最糟糕的处境了。我想做的是迫使使用了 Advanced OEP protection（高级 OEP 保护手段）的 ASProtect SKE 2.2 将分配的所有缓存都存储到一个大缓存以方便以后转储。为了找到 ASPR 混淆后的多态变形的 oep，我计划在每个新分配的缓存上使用 PAGE\_GUARD，这样我就可以确定什么时候 EIP 到达缓存并在这个时候登入，但此时一个大问题来了。  
  
因为所有的缓存都位于一个大内存缓冲区中，而我的代码用作了内存管理器（释放/分配内存页），我无法确定什么时候 eip 到达了某个可能的范围中，因为在缓冲内存中这样的范围实在是太多了。实际上我所需要的不是作为信息传递给 KiUserExceptionDispatcher 的东西，对，我需要的是 cr2 寄存器的内容，这样就能得到错误地址，这样如果 EIP 和 cr2 的内容相匹配，就说明我们到达了自己的受保护页。  
Ring3 中这一点的解决方案就是保存每个内存页的信息，这样一旦发生了 PAGE\_GUARD 异常，我们就可以确定这个 PAGE\_GUARD 异常发生时，EIP 是在一个没有 PAGE\_GUARD 属性的页中还是在一个标记了 PAGE\_GUARD 的内存页中。是个不错的主意，但是需要编写更多的代码，还需要重新组织我的 ring3 的内存管理器，增加对每个内存页的保护功能。因为我懒得重新编写，所以我就又为 KiUserExceptionDispatcher 添加了一小段代码。  
  
当然也可能 hook KiTrap0E 并且在 context 结构中返回 cr2 寄存器的内容。嗯，不错的想法。cr2 寄存器将会保存有错误发生的 va（虚拟地址），因为对受保护页的访问只不过是触发异常，我们除了得到 cr2 寄存器的值还要确定异常是由于执行产生的还是由于读写产生的。当然，KiUserExceptionDispatcher 是不会替我们返回出错地址的，就是因为糟糕的这一点我们必须改进它，使它能返回 cr2 寄存器的值：  
cr2 = faulting\_address0 //aal 注：出错地址  
eip = faulting\_address0 //aal 注：出错地址  
exceptioncode = EXCEPTION\_GUARD\_PAGE//错误代码 = EXCEPTION\_GUARD\_PAGE  
爽，登入它，去除这一页的保护，然后等待下次访问。但是如何得到 cr2 寄存器的内容呢？？？是啊，这的确是个头痛的问题，但是实际上确实是可以做到的。来看看 KiTrap0E 是怎么做的：  
.text:804DAF25 \_KiTrap0E:  
.text:804DAF25 mov word ptr \[esp+2\], 0  
.text:804DAF2C push ebp  
.text:804DAF2D push ebx  
.text:804DAF2E push esi  
...  
.text:804DB0ED mov ecx, 3  
.text:804DB0F2 mov edi, eax  
.text:804DB0F4 mov eax, 0C0000006h  
.text:804DB0F9 call CommonDispatchException  
  
到这里所有的 KiTrap 调用将会调用 CommonDispatchException，这个函数会将 CONTEXT 和 ERROR\_CODE 的内容保存到堆栈中并且将 EIP 重新引导到 KiUserExceptionDispatcher。继续跟踪 CommonDispatchException，我们来到这里：  
.text:804D8A8D CommonDispatchException proc near  
.text:804D8A8D  
.text:804D8A8D  
.text:804D8A8D  
.text:804D8A8D sub esp, 50h  
.text:804D8A90 mov \[esp+50h+var\_50\], eax  
.text:804D8A93 xor eax, eax  
.text:804D8A95 mov \[esp+50h+var\_4C\], eax  
.text:804D8A99 mov \[esp+50h+var\_48\], eax  
.text:804D8A9D mov \[esp+50h+var\_44\], ebx  
.text:804D8AA1 mov \[esp+50h+var\_40\], ecx  
.text:804D8AA5 cmp ecx, 0  
.text:804D8AA8 jz short loc\_804D8AB6  
.text:804D8AAA lea ebx, \[esp+50h+var\_3C\]  
.text:804D8AAE mov \[ebx\], edx  
.text:804D8AB0 mov \[ebx+4\], esi  
.text:804D8AB3 mov \[ebx+8\], edi  
.text:804D8AB6  
.text:804D8AB6 loc\_804D8AB6:  
.text:804D8AB6 mov ecx, esp  
.text:804D8AB8 test dword ptr \[ebp+70h\], 20000h  
.text:804D8ABF jz short loc\_804D8AC8  
.text:804D8AC1 mov eax, 0FFFFh  
.text:804D8AC6 jmp short loc\_804D8ACB  
.text:804D8AC8  
\---------------------------------------------------------------------  
.text:804D8AC8  
.text:804D8AC8 loc\_804D8AC8:  
.text:804D8AC8 mov eax, \[ebp+6Ch\]  
.text:804D8ACB  
.text:804D8ACB loc\_804D8ACB:  
.text:804D8ACB and eax, 1  
.text:804D8ACE push 1  
.text:804D8AD0 push eax  
.text:804D8AD1 push ebp  
.text:804D8AD2 push 0  
.text:804D8AD4 push ecx  
.text:804D8AD5 call \_KiDispatchException@20  
.text:804D8ADA mov esp, ebp  
.text:804D8ADC jmp Kei386EoiHelper@0  
.text:804D8ADC CommonDispatchException endp  
好了，因为知道早晚都会调用\_KiDispatchException 所以我们就直接跟到这里了：  
.text:804F318D ; \_\_stdcall KiDispatchException(x,x,x,x,x)  
.text:804F318D \_KiDispatchException@20 proc near  
.text:804F318D  
.text:804F318D push 390h  
.text:804F3192 push offset dword\_804F3278  
.text:804F3197 call \_\_SEH\_prolog  
.text:804F319C mov eax, ds:\_\_\_security\_cookie  
.text:804F31A1 mov \[ebp-1Ch\], eax  
.text:804F31A4 mov esi, \[ebp+8\]  
.text:804F31A7 mov \[ebp-2ECh\], esi  
  
.text:804F31AD mov ecx, \[ebp+0Ch\]  
.text:804F31B0 mov \[ebp-2F0h\], ecx  
.text:804F31B6 mov ebx, \[ebp+10h\]  
.text:804F31B9 mov \[ebp-2F8h\], ebx  
.text:804F31BF db 3Eh  
.text:804F31BF mov eax, ds:0FFDFF020h  
.text:804F31C5 inc dword ptr \[eax+504h\]  
.text:804F31CB mov dword ptr \[ebp-2E8h\], 10017h  
.text:804F31D5 cmp byte ptr \[ebp+14h\], 1  
.text:804F31D9 jz loc\_804F5A76  
.text:804F31DF cmp ds:\_KdDebuggerEnabled, 0  
.text:804F31E6 jnz loc\_804F5A76  
.text:804F31EC  
.text:804F31EC loc\_804F31EC:  
.text:804F31EC  
.text:804F31EC lea eax, \[ebp-2E8h\]  
.text:804F31F2 push eax  
.text:804F31F3 push ecx  
.text:804F31F4 push ebx  
.text:804F31F5 call \_KeContextFromKframes@12  
.text:804F31FA mov eax, \[esi\]  
.text:804F31FC cmp eax, 80000003h  
.text:804F3201 jnz loc\_804F5A20  
.text:804F3207 dec dword ptr \[ebp-230h\]  
.text:804F320D  
.text:804F320D loc\_804F320D:  
.text:804F320D  
.text:804F320D xor edi, edi  
.text:804F320F  
.text:804F320F loc\_804F320F:  
.text:804F320F cmp byte ptr \[ebp+14h\], 0  
.text:804F3213 jnz loc\_804F58C3  
.text:804F3219 cmp byte ptr \[ebp+18h\], 1  
.text:804F321D jnz loc\_80516D98  
.text:804F3223 mov eax, ds:\_KiDebugRoutine  
.text:804F3228 cmp eax, edi  
.text:804F322A jz loc\_80505721  
.text:804F3230 push edi  
.text:804F3231 push edi  
.text:804F3232 lea ecx, \[ebp-2E8h\]  
.text:804F3238 push ecx  
.text:804F3239 push esi  
.text:804F323A push dword ptr \[ebp-2F0h\]  
.text:804F3240 push ebx  
.text:804F3241 call eax  
.text:804F3243 test al, al  
.text:804F3245 jz loc\_80505721  
.text:804F324B  
.text:804F324B loc\_804F324B:  
.text:804F324B push dword ptr \[ebp+14h\]  
.text:804F324E push dword ptr \[ebp-2E8h\]  
.text:804F3254 lea eax, \[ebp-2E8h\]  
.text:804F325A push eax  
.text:804F325B push dword ptr \[ebp-2F0h\]  
.text:804F3261 push ebx  
.text:804F3262 call \_KeContextToKframes@20  
.text:804F3267  
.text:804F3267 loc\_804F3267:  
.text:804F3267  
.text:804F3267 mov ecx, \[ebp-1Ch\]  
.text:804F326A call @xHalReferenceHandler@4  
  
.text:804F326F call \_\_SEH\_epilog  
.text:804F3274 retn 14h  
.text:804F3274 \_KiDispatchException@20 endp ; sp = -14h  
.text:804F3274  
如果不实际的跟踪一下，就看不出这些代码有什么意义，所以我们可以使用 Softice 开始跟踪，很快我们就会发现我们想要的字节：  
.text:804F5959 call \_ProbeForWrite@12  
.text:804F595E mov ecx, 0B3h  
.text:804F5963 lea esi, \[ebp+var\_2E8\]  
.text:804F5969 rep movsd  
804F5969 处的指令负责将 CONTEXT 拷贝到用户堆栈，当然是为 KiUserExceptionDispatcher 做这个工作。如果我们能够 hook 那条指令，我们当然也就能够保存 cr2 寄存器的内容，并将它的值放到 CONTEXT 结构的某个域中传回 ring3。这些域就是 dr0/4 regs。如果我们不处理它，我们的非侵入式跟踪器应该清除 cr2 的值。  
我们将要 hook "lea esi, \[ebp+var\_2E8\]" 处，因为它的大小足够保存 hook-也就是 push/ret 组合，注意千万不要 hook rep movsd 处，否则驱动卸载的时候有可能发生 BSOD（aal 注：Blue Screen of Death，蓝屏死机）的错误。只是有可能发生，正像 Mark Russinovich 所描述的，当驱动的地址空间中的代码在执行时，如果打断了某个线程，而我们在这个时候卸载驱动，当被中断的线程打算再次执行时就会返回到空内存而你会得到一个内存页错误（PAGE\_FAULT），看到的就是 BSOD。我还没有遇到过这种情况，但是这的确是有可能发生的。只要你遭遇到 BSOD 却一点也不知道是什么原因造成的时候，脑子中想到这一点就行。  
同样我们必须标识我们的进程，完成这一点，我使用的是 cr3 技巧，因为 cr3 中存储了 PDE 的物理地址，因为每个进程都有自己的内存空间，那样我们就能毫无问题的标识进程了 \[11\]。注意我使用的 KeStackAttachProcess 是不必要的， 我们可以直接在 KPROCESS 结构(EPROCESS 的一部分)中得到 cr3 的值：  
kd> dt nt!\_KPROCESS  
+0x000 Header : \_DISPATCHER\_HEADER  
+0x010 ProfileListHead : \_LIST\_ENTRY  
+0x018 DirectoryTableBase : \[2\] Uint4B <-- 就是这里了   
+0x020 LdtDescriptor : \_KGDTENTRY  
ntoskrnl.exe 基址的偏移地址 25963h 处，我们现在就可以准备完成这个技巧了。开始干吧！！！！你必须在你自己的 ntoskrnl.exe 中看看这个偏移地址，因为这个偏移地址可能会有所不同。  
另外还有一种条件非常非常非常重要，这就是 cr2 的保存!!!  
cr2 的值没有被保存，如果其它的内存页错误，cr2 就会保存那个错误地址。 这对我们来说使非常危险的，因为内存页错误的处理程序同样也用来将内存页存储在页面文件中。有一种存储 cr2 的值的方法就是 hook int 0eh，它能够在当且仅当 cr2 的值属于我们的进程时存储它。接下来的代码中，我们的 hook 将会得到已经存储了的 cr2 的值，并且将它的值放到 context.context\_dr0 中传给我们的代码。  
hookint0eh label dword  
push eax  
mov eax, cr3  
cmp eax, c\_cr3  
jne \_\_exit\_int0e  
mov eax, cr2  
mov c\_cr2, eax  
\_\_exit\_int0e:   
pop eax  
jmp cs:\[oldint0eh\]  
;将 cr2 的内容保存在 context.context\_dr0 中  
hookmycode label dword  
push eax  
mov eax, cr3  
cmp eax, c\_cr3  
jne \_\_exithook  
lea esi, \[ebp-2e8h\]  
mov eax, c\_cr2  
mov \[esi.context\_dr0\], eax  
\_\_exithook:   
pop eax  
lea esi, \[ebp-2e8h\]  
retaddr:   
push 0deac0deh  
ret  
现在在你的非侵入式跟踪器中你只要测试是否 EIP = DR0，如果相等的话我们就登入访问，而如果不是我们就去除 PAGE\_GUARD 然后在错误指令后设置 int 3h。我们有改进后的 KiUserExceptionDispatcher 替我们作我们想做的事。   
同样这也将帮助我们分别访问我们的内存页时产生的 ACCESS\_VIOLATION 时我们是不是在使用 PAGE\_NOACCESS 而不是在使用 PAGE\_GUARD。  
4.4. 进入内存区域并访问  
一旦我们知道了如何解决问题，我们就得进入我们的内存页面。这可以通过向文件进行写入操作来实现，或者更好一点的方法是使用 www.sysinternals.com 的 OutputDebugStringA 和 DbgView 得到输出：  
示例代码：  
\_\_log: lea ecx, \[ebp+format5\]  
    lea ebx, \[ebp+buffer\]  
    push eax  
    push ecx  
    push ebx  
    call \[ebp+wsprintfA\]  
    add esp, 0ch  
    push ebx  
    call \[ebp+OutputDebugStringA\] ;登入 访问  
    ...  
    format5 db "eip log : 0x%.08X", 13, 10, 0  
现在，你应该在 DbgView 中得到了许多数据，保存下日志数据，研究一下地址，排出掉重复的部分，并且通过再次运行跟踪器缩减日志文件的大小。  
4.5. 从跟踪器中触发驱动  
这个实际上用来通知驱动要关心什么样 PID，此时驱动已经安装好了，但是我们还得告诉它开始跟踪我们：  
push 0  
push 0  
push OPEN\_EXISTING  
push 0  
push 0  
push GENERIC\_READ or GENERIC\_WRITE  
push offset driver  
callW CreateFileW  
mov dhandle, eax  
call DeviceIoControl, eax, 20h, o pid, 4, o pid, 4, o dwbytes, 0  
push dhandle  
callW CloseHandle  
...  
driver: unis <\\\\.\\ring0>  
pid = pid of process we are tracing//我们所跟踪程序的 pid  
我还使用了文章 Loader from ring0 \[11\]中讲述的相同的方法来标识进程。  
4.6. 制作 stealth 非侵入式跟踪器  
好，这里假定我们对付的是个新壳，它会在 KiUserExceptionDispatcher 中寻找我们的 hook，我们要做的就是打败它。怎样才能做到这一点呢？的确不是简单的任务，多亏有了 yates 展示的技巧\[8\]我们能过继续下去。  
KiUserExceptionDispatcher 是一个永远永远都不会返回的过程，它将要调用 NtContinue 或者是 NtRaiseException。我们看看都发生了什么：  
\- 异常发生  
\- ntoskrnl.exe 通过 KiTrapXX 接手控制权  
\- KiTraps 实际上是 IDT 的入口(?)，并且根据不同的异常在 KiTrapXX 的入口处有两种可能的堆栈布局：  
+---------------+    +---------------+  
|     EFLAGS      |    |      EFLAGS     |  
+---------------+    +---------------+  
|       CS         |   |         CS       |  
+---------------+   +---------------+  
|       EIP        |   |        EIP     |  
+---------------+   +---------------+  
            |   Error Code   |  
            +---------------+  
  
因为一些异常并不引发错误，同时也为了从 ring0 中退出来时更容易些，不管什么异常发生了一些 KiTrapXX 都将 0 压入堆栈模仿代码，比如 KiTrap01 和 KiTrap03：  
\_KiTrap01  
0008:804D8D7C PUSH 00 <--- dummy Error Code  
0008:804D8D7E MOV WORD PTR \[ESP+02\],0000  
0008:804D8D85 PUSH EBP  
0008:804D8D86 PUSH EBX  
0008:804D8D87 PUSH ESI  
0008:804D8D88 PUSH EDI  
0008:804D8D89 PUSH FS  
\_KiTrap03  
0008:804D915B PUSH 00 <--- dummy Error Code  
0008:804D915D MOV WORD PTR \[ESP+02\],0000  
0008:804D9164 PUSH EBP  
0008:804D9165 PUSH EBX  
0008:804D9166 PUSH ESI  
0008:804D9167 PUSH EDI  
0008:804D9168 PUSH FS  
但是 KiTrap0E (内存页错误处理程序) 并没有将 0 压入堆栈因为错误代码存在了堆栈中。  
\_KiTrap0E  
0008:804DAF25 MOV WORD PTR \[ESP+02\],0000  
0008:804DAF2C PUSH EBP  
0008:804DAF2D PUSH EBX  
0008:804DAF2E PUSH ESI  
0008:804DAF2F PUSH EDI  
0008:804DAF30 PUSH FS  
0008:804DAF32 MOV EBX,00000030  
从中断中返回是由一个简单的 IRETD 指令完成的，它与 ret 指令相近，也是跳转到堆栈中所保存的 EIP。异常处理完毕之后，ring0 确定要调用 KiUserExceptionDispatcher 时它就会将 KiUserExceptionDispatcher 的地址存储在堆栈中，所以 IRETD 只是简单的返回了 KiUserExceptionDispatcher ：  
0008:804F5A0F MOV EAX,\[\_KeUserExceptionDispatcher\]  
0008:804F5A14 MOV \[EBX+68\],EAX  
:dd ebx+68  
0010:EEC21DCC 7C90EAEC 0000001B 00000246 0013FCD0 ìê |....F.......  
0010:EEC21DDC 00000023 00000000 00000000 00000000 #...............  
正如你所看到的，EIP 被 KiUserExceptionDispatcher 的地址覆盖了以及堆栈中保存的 CS，Eflags，esp 和 SS。因为我们要做的是 hook 这些指令，所以他会指向 ntdll.dll 中其他的代码，就是我们使用 yates 展示的方法所存储的那些代码。 同样，也有更好的方法应该尽量不要扫描磁盘上的 ntdll.dll，而是使用内存中已经载入的文件直接重新引导至 UserSharedData，在用户模式下被设置成了只读：  
kd> ? SharedUserData  
Evaluate expression: 2147352576 = 7ffe0000  
kd>  
但是在 ring0 它被映射到了：  
#define KI\_USER\_SHARED\_DATA 0xffdf0000  
所以我们可以在 ring0 中向那里写入，并且重新引导我们的异常(Exceptions)到达负责跳转到 KiUserExceptionDispatcher 的地方，或者只是简单的调用我们存储在被调试程序内存空间某处的非侵入式跟踪器。见\[ring0stealthtracer\]文件夹。在你运行之前确定你理解了这些代码将要做什么。我这次使用的是 PID 标识进程。我们首先定位 ntoskrnl.exe 的基址：   
iMOV esi, ZwCreateFile  
and esi, 0FFFFF000h  
\_\_find\_base: cmp word ptr\[esi\],'ZM'  
je \_\_ntoskrnlbase  
sub esi, 1000h  
jmp \_\_find\_base  
\_\_ntoskrnlbase: mov ntoskrnlbase, esi  
然后我们必须定位\_KeUserExceptionDispatcherVariable（变量），它没有被输出。对我们来说，幸运的是它被存储在一个以双字（dword）对齐的边界处，而且在 ntoskrnl.exe 中 KiUserExceptionDispatcher 只出现了一次，所以我们可以使用 KiUserExceptionDispacther 的地址搜索它：  
mov edi, esi  
mov ebx, esi  
add ebx, dword ptr\[ebx+3ch\]  
mov ecx, \[ebx.NT\_OptionalHeader.OH\_SizeOfImage\]  
shr ecx, 2  
cld  
mov eax, kiuser  
repnz scasd  
sub edi, 4  
一旦我们定位在未输出的的\_KeUserExceptionDispatcher 我们就可以将我们的代码保存在 UserSharedData 然后将\_KeUserExceptionDispatcher 覆盖成我们代码的地址：  
push edi  
mov KiUserExceptionDispatcher, kiuser  
mov edi, kiusershareddata+100h  
mov esi, offset shareddatahook  
mov ecx, shareddatahooksize  
cld  
rep movsb  
pop edi  
mov dword ptr\[edi\], kiusershareddataring3+100h  
...  
kiusershareddata equ 0ffdf0000h  
kiusershareddataring3 equ 07ffe0000h  
  
完工！  
让我们看看如果用 softICE 跟踪的话是什么样子：  
\_KiTrap03  
0008:804D915B PUSH 00  
0008:804D915D MOV WORD PTR \[ESP+02\],0000  
0008:804D9164 PUSH EBP  
0008:804D9165 PUSH EBX  
0008:804D9166 PUSH ESI  
0008:804D9167 PUSH EDI  
0008:804D9168 PUSH FS  
...  
0008:804F5A0F MOV EAX,\[\_KeUserExceptionDispatcher\]  
0008:804F5A14 MOV \[EBX+68\],EAX  
0008:804F5A17 OR DWORD PTR \[EBP-04\],-01  
… :  
dd \_KeUserExceptionDispatcher  
0023:80552AF0 7FFE0100 7C90EAD0 7C90EAC0 00002626 ... .ê |.ê |&&..  
这里是 iretd 之前的堆栈:  
0023:F0E9DDCC 7FFE0100 0000001B 00000246 0013FCC8 ... ....F.......  
0023:F0E9DDDC 00000023 00000000 00000000 00000000 #...............  
然后我们就到了我们存在 UserSharedData 中的代码：  
001B:7FFE0100 CALL 7FFE0105  
001B:7FFE0105 POP EDX  
001B:7FFE0106 SUB EDX,F7129409  
001B:7FFE010C MOV ECX,FS:\[0020\] <--从 TEB 中得到 PID  
001B:7FFE0112 CMP DWORD PTR \[EDX+F712943B\],-01 <-- 不跟踪  
001B:7FFE0119 JZ ntdll!KiUserExceptionDispatcher   到 KiUser 跳转  
001B:7FFE011B CMP \[EDX+F712943B\],ECX <-- 被跟踪的程序的 PID 么?  
001B:7FFE0121 JNZ ntdll!KiUserExceptionDispatcher 不是，我们调用 KiUser  
001B:7FFE0123 JMP \[EDX+F7129437\] <-- 我们调到 nonintrusive  
001B:7FFE0129 JMP \[EDX+ntdll!KiUserExceptionDispatcher\] <-- 回到 KiUser  
不是被跟踪的程序，所以我们继续对 KiUserExceptionDispatcher 的访问：  
ntdll!KiUserExceptionDispatcher  
001B:7C90EAEC MOV ECX,\[ESP+04\]  
001B:7C90EAF0 MOV EBX,\[ESP\]  
001B:7C90EAF3 PUSH ECX  
001B:7C90EAF4 PUSH EBX  
希望你理解了 stealth 技术? 祝未来的保护器好运!!  
注意这个技巧是 Barnaby Jack 在他关于 ring0shell code 著名的文章 \[9\] 。.  
  
4.7. 非侵入式跟踪器结论  
到这里，希望你已经明白了基本思路，我提供的所有的代码和示例都是以在 ntoskrnl.exe 中放置 hook 为目的的，所以你需要将我的驱动源代码中的偏移地址修改为你自己的所有的驱动代码都只是未经修正过的，如果没有提前修改而因为错误的偏移地址导致了蓝屏死机，系统崩溃，数据丢失等均由使用者本人负责。  
  
  
5. 装载器的装载器  
装载器的装载器实际上是处理双进程保护的装载器。一些软件使用装载器装载真正的游戏，我已经做了一个处理这种软件的装载器。因为这样就存在了两个装载器所以我把它称作装载器的装载器。网上很可能已经有我写的关于这方面的文章，但是在这里我还是想描述一下，主要是因为我们所对付的就有一些双进程的保护壳。我们必须获得第二个进程的控制权，为了达到这个目的我们将要 hook 的是 CreateProcessA，这样等它完成了自己的任务时，我们就可以得到一个 PROCESS\_INFORMATION 结构，这个结构中包含了 ProcessID 和 ThreadID。  
\- ProcessId 有了它我们就可以打开进程然后在它的缓存地址空间写东西等等…  
\- ThreadId 有了它我们才能控制某些东西，Get/SetThreadContex，Resume/SuspendThread。  
注意：如果我们是从注入的偏移地址无关代码中 hook 我们就不需要了，因为我们已经有了保存在 PROCESS\_INFORMATION 结构中的 ProcessHandle 和 ThreadHandle。  
我将讲述 3 种方法，其中一种有可能还没有在 RCE 领域中被讨论过。  
5.1. 使用注入代码实现的装载器的装载器  
这种方法的思路是将我们的内存管理器注入到第二个进程中，整个过程涉及到在返回(ret)时 hook CreateProcessA，此时我们会得到一个错误代码（如果进程的执行没有问题的话）。我们还会得到一个填充好了的 PROCESS\_INFORMATION 结构，其中就包含了进程和主线程的句柄。从根本上讲，就会有两套偏移无关代码 。示例代码在\[lflinjected\]文件夹中。  
我们来看一眼我们的例子，在第二个装载器的装载器中我还要用到这个测试(test)程序，所以脑子里记住这里我们是怎么处理的：  
push offset mutexname  
push 0  
push MUTEX\_ALL\_ACCESS  
callW OpenMutexA  
test eax, eax  
jnz \_\_2ndprocess  
push offset mutexname  
push 0  
push 0  
callW CreateMutexA  
push offset pinfo  
push offset sinfo  
push 0  
push 0  
push 0  
push 0  
push 0  
  
push 0  
push offset progy  
push 0  
callW CreateProcessA  
push -1  
push pinfo.pi\_hProcess  
callW WaitForSingleObject  
jmp \_\_exit  
\_\_2ndprocess: push 40h  
push offset mtitle  
push offset mtext  
push 0  
callW MessageBoxA  
\_\_exit: push 0  
callW ExitProcess  
看到了吧，这个小程序是要生成一个互斥对象(mutex)，然后根据互斥对象的情况来决定是输出信息还是产生一个新的进程实例。为了实现装载器的装载器，我们用注入代码的第一部分 hook 函数 CreateProcessA，第二部分用来注入。从根本上讲这里出现了两个偏移无关代码，索性的是获得 kernel 的基址(getkernelbase)和从 kernel32.dll 搜索输出函数的工作，我们只需要进行一次，因为 kernel32.dll 偏移地址是不会因为不同的进程而有所不同的。  
思路：  
\- 使用 CreateProcess 生成挂起状态的进程  
\- 注入偏移无关代码  
\- 使用偏移无关代码 hook 我们目标进程中的 CreateProcessA  
\- 一旦运行到了 CreateProcessA 返回(retn)处的 hook 就得到了进程和线程句柄  
\- 到这里我们的偏移无关代码就要使用 WriteProcessMemory 和 VirtualAllocEx 将一个新的装载器注入到第二个进程中，此处从根本上讲我们是在重复我们的原始的装载器中注入偏移无关代码的工作  
由于第一个装载器的大小的问题，源代码就不在这里贴出了，你可以在\[lflinjected\]文件夹中找到已经经过注释的源代码文件。  
5.2. 不使用注入代码实现的装载器的装载器  
这是更为简单的一个解决方案，也仅需要较少的偏移无关代码的编写和理解。但是如果进程不是使用 CREATE\_SUSPENDED 创建的，这种方法就不太好了，因为我们必须手动的加入 CREATE\_SUSPENDED 标志，这样甚至反倒需要更多的代码。  
思路：  
\- 使用 CREATE\_SUSPENDED 标志载入第一个程序  
\- 在 CreateProcessA 中插入 jmp $  
\- 运行到我们的 hook 时，从堆栈中读取 PROCESS\_INFORMATION 的地址，dwCreationFlags 和返回地址  
\- 将 dwCreationFlags 与 CREATE\_SUSPENDED 进行或操作并且将它存回堆栈，清除前面的 hook 并在堆栈的所指向的返回地址插入新的指令  
\- 运行到我们的第二个 hook 时，读取 PROCESS\_INFORMATION 我们会得到 pid 和 tid  
\- 使用 OpenProcess 和 OpenThread 处理新进程  
在\[lflwoinjected\]文件夹中查看完整的实现代码，已注释。  
5.3.用于被调试进程的非侵入式跟踪器   
我这里用的是一个使用 Armadillo 4.3 的 Standard Protection + Debug Blocker 加壳的 crackme 来注入我的跟踪器。当然这对内存管理器也适用，但为了简单明了的说明问题，我只把非侵入式的跟踪器注入到其中并且中断在 OEP 处。. Lol, guess what you got DebugBlocker  
armadillo oep finder with this tutorial.  
其实实践起来并不难，你所需要的只是一点点想象力，以及一些如何使用 Windows Debug APIs \[10\]实现 ring3 级的调试的知识，你可以参考一些调试装载器的教程。要实现这一点我们要 hook WaitForDebugEvent 以便我们第一时间得到它的输出值然后才能够检查 DEBUG\_EVENT 结构的内容，我们只对传给我们的代码的 EXCEPTION\_DEBUG\_EVENT 感兴趣，而所有其他的事件都直接传回给程序。我使用带 DBG\_EXCEPTION\_NOT\_HANDLED 参数的 ContinueDebugEvent，它会在我们的被跟踪程序中调用 KiUserExceptionDispatcher。为了达到目的，我们还得在 armadillo 中 CreateProcessA 的返回处(retn)hook，这样才能将我们的非侵入式跟踪器注入。  
  
代码和目标程序都在文件夹\[armadillo\_oep\]中，既然到了这里，还要感谢一下 Teddy Rogers 和他收集 unpackme 的网站 http://www.tuts4you.com/unpackme/ ，因为我们将要使用的 unpackme 就来自这里  
我们来看看调用了 CreateProcessW 后 armadillo 又干了什么：  
004949EC . 52 PUSH EDX  
004949ED . 6A 02 PUSH 2  
004949EF . 68 A4B44C00 PUSH armadill.004CB4A4  
004949F4 . 8B45 10 MOV EAX,DWORD PTR SS:\[EBP+10\]  
004949F7 . 50 PUSH EAX  
004949F8 . 8B4D 08 MOV ECX,DWORD PTR SS:\[EBP+8\]  
004949FB . 8B11 MOV EDX,DWORD PTR DS:\[ECX\]  
004949FD . 52 PUSH EDX  
004949FE . FF15 D0304C00 CALL DWORD PTR DS:\[<&KERNEL32.ReadProcessMem>  
这里，Armadillo 从入口处读取了 2 个字节，这样它就能够在这里插入 jmp $ hook 然后再使用 DebugActiveProces 附加进程了，当然，armadillo 也使用 Sleep/GetThreadContext 来确定程序何时到达入口点，在无限循环中：  
0048FD8D > \\A1 A0B54C00 MOV EAX,DWORD PTR DS:\[4CB5A0\]  
0048FD92 . 8B48 08 MOV ECX,DWORD PTR DS:\[EAX+8\]  
0048FD95 . 51 PUSH ECX  
0048FD96 . FF15 C4314C00 CALL DWORD PTR DS:\[<&KERNEL32.DebugActivePro>  
  
从这里开始我们就进入了一个使用 WaitForDebugEvent 和 ContinueDebugEvent 制造的调试循环(Debug Loop)。 “Hybrid” hooking of WaitForDebugEvent 在这里是不行的，因为这里的 ret 附近没有填充的 nop 指令，所以我们要使用更巧妙一些的 hook 方法。  
这个方法就是我们覆盖堆栈中所存储的返回地址，这样运行 WaitForDebugEvent，当它试图返回时就会跳转到我们的代码了。  
  
  
好，还没跑吧？或许一些图例将会帮助理解得更清楚些  
+---------------+---------------+---------------+  
|     返回地址     |     调试事件     |      毫秒      |  
+-------+-------+---------------+---------------+  
     |  
  被钩子覆盖  
     |  
+-------+-------+---------------+---------------+  
|      我的代码    |     调试事件     |      毫秒      |  
+---------------+---------------+---------------+  
现在一旦 WaitForDebugEvent 打算返回到它的调用者是就会到达我的代码，而我们就可以随心所欲的处理异常了：   
hooking 代码示例：  
; ;  
首先我们需要几个字节插入我们的 hook  
;  
mov esi, \[edi+WaitForDebugEvent\]  
xor ecx, ecx  
push esi  
\_\_get5bytes: push esi  
call ldex86  
add ecx, eax  
add esi, eax  
cmp ecx, 5  
jb \_\_get5bytes  
pop esi  
push edi  
lea edi, \[edi+rippedbytes\]  
cld  
rep movsb  
pop edi  
mov dword ptr\[edi+goback+1\], esi  
;;  
将字节拷贝到可以安全运行的缓存区域  
;  
mov esi, \[edi+WaitForDebugEvent\]  
mov ecx, esi  
lea ebx, \[edi+hookwaitfordebugevent\]  
mov byte ptr\[esi\], 0e9h  
add ecx, 5  
sub ebx, ecx  
mov dword ptr\[esi+1\], ebx  
  
被覆盖部分的字节保存在了 rippedbytes 变量中：  
hookwaitfordebugevent label dword  
pusha  
call deltahook  
deltahook: pop ebp  
sub ebp, offset deltahook  
mov eax, \[esp+8\*4\]  
;保存原来的返回地址  
mov \[ebp+waitfordebugeventretaddress\], eax  
mov dword ptr\[ebp+retorig+1\], eax  
lea ebp, \[ebp+mywaitfordebugevent\]  
mov \[esp+8\*4\], ebp  
popa  
rippedbytes db 30 dup(90h)  
goback: push 0deac0deh <- WaitForDebugEvent + ripped bytes  
ret  
经过我们的工作，WaitForDebugEvent 完成后我们就会到达这里了： imywaitfordebugevent proc  
mov ecx, \[esp-8\] ;这里是 debugevent  
pusha  
同样要注意偏移地址是负值，因为 ret 和 leave 指令已经将堆栈对齐了，因为 windows API 使用的是 stdcall 约定(除了几个函数如 wsprintfA，DbgPrint，还有其他几个???)。  
这种情况下我访问变量的时候我们必须计算出负的偏移地址，同样还有小心，此时不要将任何值压入堆栈，因为每次入栈都将会破环存在堆栈中的数据，这些数据对我们使用的这种方法是至关重要的。  
现在我们关心的就是异常的处理了，只处理 EXCEPTION\_DEBUG\_EVENT：  
mywaitfordebugevent proc  
mov ecx, \[esp-8\] ;这里是 debugevent  
pusha  
call deltamydebug  
deltamydebug: pop edi  
sub edi, offset deltamydebug  
mov ebx, ecx  
cmp \[ebx.de\_code\], EXCEPTION\_DEBUG\_EVENT  
jne \_\_return\_to\_original  
cmp \[ebx.de\_u.ER\_ExceptionCode\], EXCEPTION\_BREAKPOINT  
jne \_\_passexception  
cmp \[edi+firstint3h\], 1  
je \_\_passexception  
;在远进程中设置页面保护  
pushv <dd ?>  
push PAGE\_EXECUTE\_READWRITE or PAGE\_GUARD  
push \[edi+c\_range\]  
push \[edi+c\_start\]  
push \[edi+phandle\]  
call \[edi+VirtualProtectEx\]  
push DBG\_CONTINUE  
push \[ebx.de\_ThreadId\]  
push \[ebx.de\_ProcessId\]  
call \[edi+ContinueDebugEvent\]  
mov \[edi+firstint3h\], 1  
jmp \_\_l33t  
\_\_passexception: push DBG\_EXCEPTION\_NOT\_HANDLED  
push \[ebx.de\_ThreadId\]  
push \[ebx.de\_ProcessId\]  
call \[edi+ContinueDebugEvent\]  
;;现在的问题是我们返回哪里呢？我们可以把修改过的错误代码保存在 debug\_event.code  
;中，然后继续执行，因为我们知道再没有什么会在破坏我们的代码了。   
;   
\_\_l33t: mov \[  
ebx.de\_code\], 0deadc0deh ;l33t  
\_\_return\_to\_original: popa  
retorig: push 0deadc0deh ;改成返回地址  
ret  
endp  
我们只对第一个 int 3h 感兴趣，它产生于 DebugBreak，它应该以 ContinueDebugEvent(DBG\_CONTINUE)的形式传递，其他的情况我们都使用 ContinueDebugEvent(DBG\_EXCEPTION\_NOT\_HANDLED)的形式将它传回被调试程序，让我们的非侵入式的跟踪器来处理被跟踪进程的内存区，同样，为了避免破坏我们的代码，我们将 debug\_event.code 设置成 0deadc0deh，这样它就不会再处理 debug\_event 了， 或者我们也可以将 ThreadId 或 ProcessId 的值设成毫无意义的数值，这样 ContinueDebugEvent 就对我们程序什么都不做了。  
现在一切都交给我们存在被调试程序的非侵入式的跟踪器了，等待知道你得到一个信息告诉你 oep 找到了。另外 MessageBoxA 的“Ok”按钮被点击后，我还在非侵入式跟踪器中插入了一条 jmp $，所以现在你要做的就是转出进程并且，当然得从任务管理器或者是 Mark Russinovish 的进程管理器中结束它了。  
  
祝你好运   
代码位于 \[armadillo\_oep\]文件夹中。  
  
  
  
  
6. 调试注入的代码 – 几条建议  
调试这种代码的要点就是能够看到第二个进程里的代码，尤其是我们面对的是 loader 的 loader 或者跟踪被调试程序的时候。  
有几个技巧需要提一下：  
\- 在你的代码中使用 int 3h 或者在 SoftICE 中使用 bpint 3 或者 i3here on 来中断在代码的可疑部分  
\- 对于装载器的装载器的情况，使用 ADDR 来看第二个进程中到底发生了什么  
\- 编写非侵入式跟踪器时使用 jmp $和 ctrl+d 中断在 softice，因为此时 bpint 3 和 i3here on 会因为非侵入式跟踪器的关系中断很多次，或者可以使用 drX 寄存器跳过错误指令然后代码中的 int 3h 就会起作用了  
  
这些都是针对调试类似的装载器的几点建议，祝你好运当然这些技巧对调试注入的 DLL 时也适用。  
  
  
  
7. 结论  
这里我展示我所能想到的一些技巧以及我编写的一些代码。希望这能够帮到一些人，特别希望能让那些入门级的逆向者明白在 RCE 的世界里编程的能力是多么重要。我的意见可能有人赞成，有人反对，有人喜欢，也有人唾弃，或者是破口大骂说我照你的代码怎么编译不成功？不管怎么说，我留下了许多东西都需要思考，要记住熟能生巧。如果你开始没有成功，那就再试一次，答案肯定就在附近的某个地方。为了搞清楚内存管理器和 Delphi 代码的关系，我用了两天的时间，而编译成功 armadillo\_oep 的代码只花了 4 天。  
  
  
我在这里展示的所有技巧的相关代码都同本文附在了一起，为了更好的理解文章，我建议你读以下源代码。另外，Tasm32 DDK 也包含在了其中。  
  
  
  
随同这篇教程的所有代码都可以随意公开使用，但使用时请在致谢里提及作者以及 ARTeam。请不要使用文中的理论进行非法行为，文中的所有信息都是以学习以及帮助更好的理解程序代码的安全技术为目的的。  
  
  
8. 参考文献  
\[1\] Optimization of 32 bit code, Benny/29a, http://vx.netlux.org/29a/29a-4/29a-4.215  
\[2\] Gaining important datas from PEB under NT boxes, Ratter/29a,  
http://vx.netlux.org/29a/29a-6/29a-6.224  
\[3\] Billy Belcebu Virus Writing Guide 1.0 for Win32, Billy Belcebu,  
http://vx.netlux.org/29a/29a-4/29a-4.202  
\[4\] Retrieving API’s addresses. LethalMind, http://vx.netlux.org/29a/29a-4/29a-4.227  
\[5\] Solution to The Amazing Picture downloader, deroko,  
http://www.crackmes.de/users/warrantyvoider/the\_amazing\_picture\_downloader/  
\[6\] Unpacking and Dumping ExeCryptor and coding loader for it, deroko,  
http://tutorials.accessroot.com  
\[7\] eXtended (XDE) disassembler engine, z0mbie, http://vx.netlux.org/29a/magazines/29a-8.rar  
\[8\] Anti-Anti-Bpm, yates, http://www.yates2k.net/syscode/bpm.rar  
\[9\] Remote Windows Kernel Exploatation - Step into ring0, Barnaby Jack,  
http://www.eeye.com/~data/publish/whitepapers/research/OT20050205.FILE.pdf  
\[10\] Win32 Debug API Part 1/2/3, Iczelion, http://win32asm.cjb.net/  
\[11\] Loader from ring0, deroko, ARTeam eZine #1, http://ezine.accessroot.com  
Some useful tutorials to learn about loaders theory:  
\[12\] Shub-Niggurath and ThunderPwr coding loader series, http://tutorials.accessroot.com  
\[13\] Createing Loaders & Dumpers – Crackers Guide to Program Flow Control, yates,  
http://www.yates2k.net/lad.txt  
\[14\] Using Memory Breakpoints, Shub-Niggurath, http://tutorials.accessroot.com  
一些很有用的文章:  
\[15\] Dll Injection Tutorial, Darawk, http://www.edgeofnowhere.cc/viewtopic.php?p=2441382  
\[16\] Three Ways to Inject Your Code into Another Process, Robert Kuster, The Code Project  
http://www.codeproject.com/threads/winspy.asp  
\[17\] InjLib – A Library that implements remote code injection for all Windows versions,  
Antonio Feijao, The Code Project, http://www.codeproject.com/library/InjLib.asp  
有用的工具：  
\[18\] OllyAdvanced plug-in, MaRKuS TH-DJM,  
http://www.tuts4you.com/forum/index.php?showtopic=7092  
http://omega.intechhosting.com/~access/forums/index.php?showtopic=2542  
\[19\] LordPE plug-in, deroko, http://deroko.phearless.org/dumpdll/  
\[20\] Ice-Ext, Stan, http://stenri.pisem.net/  
  
  
9. 致谢  
这里我要感谢所有的 ARTeam 的成员将他们的知识无私奉献，感谢 29a 病毒团队出品的最好的电子杂志，感谢我 phearless 电子杂志的朋友，感谢我在 Reversing Labs 论坛上的所有朋友，感谢所有伟大的程序员… …当然，还有感谢阅读本文的你。  
  
   
http://cracking.accessroot.com  
© \[ARTeam\] 2006

 4dd466d0e1e0cc123afbe504da2462ca