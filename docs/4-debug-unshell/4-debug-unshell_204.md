

# Wealth-Lab Developer 2.0 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50071.htm`](https://www.pediy.com/kssd/pediy05/pediy50071.htm)

目标软件：Wealth-Lab Developer 2.0   
目标文件：WealthLab.exe   
加壳方式：Armadillo 1.8x-2.x   
使用工具：WinDbg 或 trw2000, peditor, WinHex 10.2 SR-2，m$的 win32 sdk 文档   
        
URL:      http://www.silicmdr.com/downloads/WealthLabSetup.exe   
本文作者：leo\_cyl   
  
    论题中关于 Armadillo 外壳的讨论很少，以前 hying 曾经写过一些，但很少人能理解，现在就   
  
以 Wealth-Lab Developer 2.0 为例子初步探讨对付 Armadillo 外壳的方法。因为我在 xp 下没装 s-ice   
  
，只有 WinDbg 可用，所以本文兼做 WinDbg 的简单教学吧，同时为照顾使用 trw2000 的朋友，我会列出   
  
等价的 trw 命令。   
  
    Armadillo 外壳反跟踪和反 dump 挺强的，（至少用 procdump 不行）。当我们运行   
  
WealthLab.exe 后，在 Program Files\\Wealth-Lab Developer 2.0 目录下产生 WEALTHLAB.TMP0 文件。   
  
这是个不能运行的脱壳文件，也是 Armadillo 外壳的至命弱点。   
  
（一）对 anti-debug 的分析   
  
      首先 WealthLab.exe 通过 CreateFileA 产生 ArmXXXX.tmp 文件（在 window 的 temp 目录）并作为   
  
dll 载入，Armadillo 的一些重要功能在 ArmXXXX.tmp 中，它的一个主要引出函数是 GetProgramInfo，   
  
作用是 un-packing 和 un-encrypting ，还有就是检查系统中有无调试器。   
      用 WinDbg 载入 WealthLab.exe，因为 Armadillo 采用 seh 技术改变程序的流程，所以使用以下命   
  
令“sxd \*”就是告诉 WinDbg 捕获异常后不要处理，有原程序处理，对应 trw2000 的命令是“faults   
  
off”，另外 Armadillo 调用 IsDebuggerPresent 来判断系统中有无调试器，如果用 trw2000，当然检   
  
查不到，但用 WinDbg 时要下断点“bp IsDebuggerPresent”，然后 f5 执行……中断后来到这里：   
  
kernel32!IsDebuggerPresent:   
77e52e92 64a118000000    mov    eax,fs:\[00000018\]   
77e52e98 8b4030          mov    eax,\[eax+0x30\]   
77e52e9b 0fb64002        movzx  eax,byte ptr \[eax+0x2\]   
77e52e9f c3              ret    《==== eax\=1 表示系统有调试器，所以将 eax 改为 0   
                                    （如果用 trw2000，忽略这里）   
  
然后 f5 执行……中断后来到这里：   
04064be 33854cfbffff    xor    eax,\[ebp\-0x4b4\]   
004064c4 8be8            mov    ebp,eax   
004064c6 b804000000      mov    eax,0x4   
004064cb cc              int    3  〈==== seh 技术！   
004064cc 8b0dbc574100    mov    ecx,\[image00400000+0x157bc (004157bc)\]   
  
看一下 seh 链“dd fs:0”最后查到地址 40a944。下命令“gn 40a944”（trw2000 中要用 INT3HERE   
  
OFF 命令，并下断点 bpx 40a944）来到这里：   
0040a944 55              push    ebp   
0040a945 8bec            mov    ebp,esp   
0040a947 83ec08          sub    esp,0x8   
0040a94a 53              push    ebx   
0040a94b 56              push    esi   
0040a94c 57              push    edi   
0040a94d 55              push    ebp   
0040a94e fc              cld   
……   
……   
  
以上代码产生 ArmXXXX.tmp，并获得引出函数地址（在此忽略），执行到 00406807  时将调用   
  
GetProgramInfo   
  
004067f6  mov dword ptr \[ebp\-0x134\],0x4157dc   
00406800  lea    ecx,\[ebp\-0x154\]   
00406806  push    ecx   
00406807  call dword ptr \[ebp\-0x490\]{ARM1!GetProgramInfo (1000b0c0)} 〈=进入   
0040680d  add    esp,0x4   
00406810  and    eax,0xff   
00406815  test    eax,eax   
00406817  jnz    image00400000+0x6823 (00406823)   
  
进入 ARM1!GetProgramInfo (1000b0c0)后来到这……   
  
1000913c 8065d800        and    byte ptr \[ebp\-0x28\],0x0   
10009140 8365fc00        and    dword ptr \[ebp\-0x4\],0x0   
10009144 0f018dbcfdffff  sidt    \[ebp\-0x244\]      〈==\[ebp\-0x244\]中防入 idt 表   
1000914b 8b85befdffff    mov    eax,\[ebp\-0x242\]   
10009151 83c008          add    eax,0x8      〈=== int2 handle 的地址   
10009154 8b18            mov    ebx,\[eax\]   
10009156 83c010          add    eax,0x10      〈===int3 handle 的地址   
10009159 8b00            mov    eax,\[eax\]   
1000915b 25ffff0000      and    eax,0xffff   
10009160 81e3ffff0000    and    ebx,0xffff   
10009166 2bc3            sub    eax,ebx      〈== int2,3 handle 的地址相减   
10009168 83f81e          cmp    eax,0x1e   
1000916b 7547            jnz    ARM1!ReleaseHook+0x3c5b (100091b4)   
1000916d 8b8548fcffff    mov    eax,\[ebp\-0x3b8\]   
10009173 050c010000      add    eax,0x10c   
  
这是第二个 anti-debug 代码，判断 int2，3 handle 地址的距离，正常的话小于 0x1e。如果安装了   
  
int3 handle（某些调试器会）的话，大于 0x1e。奇怪的是 WinDbg 不能走完这段代码，在 1000914b   
  
处异常（trw2000 就可以）。所以在 10009144 处，我把 eip 改到 10009168，并改 eax 小于 0x1e。   
  
f5 继续执行，其间在 kernel32!IsDebuggerPresent 又会中断一次，将返回值改为 0；来到这里：   
  
1000b82e push    ebx   
1000b82f push    0x3   
1000b831 push    ebx   
1000b832 push    eax   
1000b833 call    ARM1!GetProgramInfo+0x8c8 (1000b988)   
1000b838 pop    ecx   
1000b839 push    eax   
1000b83a call    dword ptr \[10013048\]{kernel32!CreateFileA (77e5a837)}   
1000b840 cmp    eax,0xffffffff   
1000b843 jz      ARM1!GetProgramInfo+0x78e (1000b84e)   
1000b845 push    eax   
1000b846 call    dword ptr \[ARM1!NukeNow+0x6da1 (10013044)\]   
1000b84c jmp    ARM1!GetProgramInfo+0x799 (1000b859)   
1000b84e call    dword ptr \[10013060\]{ntdll!RtlGetLastWin32Error}   
1000b854 cmp    eax,0x2   
1000b857 jz      ARM1!GetProgramInfo+0x79d (1000b85d)   
1000b859 mov    byte ptr \[ebp\-0x1\],0x1   
  
eax 为"\\\\.\\SICE", "\\\\.\\NTICE", 和 "\\\\.\\SIWDEBUG"  ，哈哈，检测 s-ice！如果它们存在或   
  
RtlGetLastWin32Error（win98 是 GetLastError ）不等于 2，将设立 DEBUG 标志。注意这里重复 3 次   
  
。   
  
以上是 anti-debug 的分析，对付它的方法大家都知道了吧！  :)   
  
（二）脱壳   
  
      WealthLab.exe 运行后，在工作目录下产生 WEALTHLAB.TMP0 文件。这是个不能运行的脱壳文件   
  
。把改名为 WEAL-NUPACK.exe。用 peditor 查看，OEP 是 4086fc。用 WinHex 10.2 打开，发现从 0x400 到   
  
0x40400 共 256k 的内容被添入 0x58，即 pop eax 的机器码。只要找回这部分代码就是完整的脱壳文件   
  
了。但不能用 procdump，系统会崩溃，幸好 WinHex 提供了类似功能。先用 peditor 的 FLC 计算器计算   
  
0x400 和 0x40400 的虚拟地址，分别是 0x401000,0x441000。   
    WealthLab.exe 运行后，在运行 WinHex ，按 ALT+F9 开始内存编辑，选 WEALTHLAB.TMP0 进程，   
  
来到偏移 0x401000 初，选 0x401000 到 0x441000（共 256k）的一块内存，复制下来，并把它粘贴到   
  
WEAL-NUPACK.exe 对应的地方（0x400 到 0x40400 共 256k）。   
    到此已经把加壳文件还原了，但还是不能运行！why？呵呵……因为 OEP 不对嘛！   
  
（三）找 OEP   
    前面两步还挺容易，找 OEP 却把我难住了，足足花了 3 个小时才找到。因为外壳和原程序在两个   
  
不同的进程中，地址空间不一样，debug 起来有困难，幸好我们知道是外壳产生了原程序的进程，所   
  
以断点 CreateProcessA 是关键！   
    从新载入 WealthLab.exe，躲过 anti-debug，继续从第一步描述的地址开始，下断点“bp   
  
CreateProcessA”，来到这里：   
  
:00408908 6820594100              push 00415920   
:0040890D 8D45B8                  lea eax, dword ptr \[ebp\-48\]   
:00408910 50                      push eax   
:00408911 6A00                    push 00000000   
:00408913 6A00                    push 00000000   
:00408915 6A04                    push 00000004   
:00408917 6A00                    push 00000000   
:00408919 6A00                    push 00000000   
:0040891B 6A00                    push 00000000   
  
\* Reference To: KERNEL32.GetCommandLineA, Ord:00CAh   
                                  |   
:0040891D FF15D4004100            Call dword ptr \[004100D4\]   
:00408923 50                      push eax   
:00408924 8B4D08                  mov ecx, dword ptr \[ebp+08\]   
:00408927 51                      push ecx   
  
\* Reference To: KERNEL32.CreateProcessA, Ord:0044h   
                                  |   
:00408928 FF1544004100            Call dword ptr \[00410044\]   
:0040892E 85C0                    test eax, eax   
:00408930 7527                    jne 00408959   
:00408932 C705E057410009000000    mov dword ptr \[004157E0\], 00000009   
  
外壳在这里产生 WEALTHLAB.TMP0 进程，在此前已经把环境变量和命令行参数设定好了。f10 执行 10 多   
  
行来到这里：   
  
  
\* Reference To: KERNEL32.ResumeThread, Ord:022Ch   
                                  |   
:00406DC1 FF152C004100            Call dword ptr \[0041002C\]   
  
……   
……   
……   
  
\* Reference To: KERNEL32.GetExitCodeProcess, Ord:010Bh   
                                  |   
:00406DE2 FF1540004100            Call dword ptr \[00410040\]   
:00406DE8 81BD4CF2FFFF03010000    cmp dword ptr \[ebp+FFFFF24C\], 00000103   
:00406DF2 7423                    je 00406E17   
:00406DF4 C705E05741000A000000    mov dword ptr \[004157E0\], 0000000A   
  
……   
……   
……   
  
\* Reference To: KERNEL32.Sleep, Ord:0296h   
                                  |   
:00406E19 FF1558004100            Call dword ptr \[00410058\]   
:00406E1F 8B1524594100            mov edx, dword ptr \[00415924\]   
:00406E25 52                      push edx   
  
\* Reference To: KERNEL32.SuspendThread, Ord:0298h   
                                  |   
:00406E26 FF1554014100            Call dword ptr \[00410154\]   
:00406E2C 83F8FF                  cmp eax, FFFFFFFF   
:00406E2F 7530                    jne 00406E61   
  
……   
……   
……   
  
\* Reference To: KERNEL32.GetThreadContext, Ord:0167h   
                                  |   
:00406E6F FF1538004100            Call dword ptr \[00410038\]   
:00406E75 85C0                    test eax, eax   
:00406E77 7530                    jne 00406EA9   
:00406E79 C705E05741000A000000    mov dword ptr \[004157E0\], 0000000A   
:00406E83 A124594100              mov eax, dword ptr \[00415924\]   
  
注意，实际上外壳自身作为调试器来启动 WEALTHLAB.TMP0 进程，把 WEALTHLAB.TMP0 进程挂起后调用   
  
GetThreadContext 获得线程 Context 即“线程上下文”，如果不懂的话，请看看有关操作系统原理的   
  
书吧。获得线程 Context 有什么用呢？等一下有说明……，继续执行：   
  
\* Reference To: KERNEL32.ResumeThread, Ord:022Ch   
                                  |   
:00406ED2 FF152C004100            Call dword ptr \[0041002C\]   
:00406ED8 E9EAFEFFFF              jmp 00406DC7   
  
……   
……   
……   
  
\* Reference To: KERNEL32.VirtualProtectEx, Ord:02C4h   
                                  |   
:00406F08 FF1558014100            Call dword ptr \[00410158\]   
:00406F0E 85C0                    test eax, eax   
:00406F10 7523                    jne 00406F35   
:00406F12 C705E05741000A000000    mov dword ptr \[004157E0\], 0000000A   
  
……   
……   
……   
  
:00406F35 8D8554F2FFFF            lea eax, dword ptr \[ebp+FFFFF254\]   
:00406F3B 50                      push eax   
:00406F3C 6A02                    push 00000002   
:00406F3E 6894554100              push 00415594   
:00406F43 8B0DCC224100            mov ecx, dword ptr \[004122CC\]   
:00406F49 330DD8224100            xor ecx, dword ptr \[004122D8\]   
:00406F4F 330DE0224100            xor ecx, dword ptr \[004122E0\]   
:00406F55 330DF0224100            xor ecx, dword ptr \[004122F0\]   
:00406F5B 51                      push ecx   
:00406F5C 8B1520594100            mov edx, dword ptr \[00415920\]   
:00406F62 52                      push edx   
  
\* Reference To: KERNEL32.WriteProcessMemory, Ord:02E9h   
                                  |   
:00406F63 FF1534004100            Call dword ptr \[00410034\]   
:00406F69 85C0                    test eax, eax   
:00406F6B 7523                    jne 00406F90   
  
看看上面代码，呵呵…… 原来外壳将 WEALTHLAB.TMP0 进程的部分段（即内容是 0x58 的那部分）设为   
  
类似不可读写或执行的属性，来触发自己解码。前面说过外壳将作为调试器而 WEALTHLAB.TMP0 正好   
  
是被调试器！具体可看看 hying 以前关于 Armadillo 外壳的帖子，精华里有。   
继续执行：   
  
:00407035 6A64                    push 00000064   
  
\* Reference To: KERNEL32.Sleep, Ord:0296h   
                                  |   
:00407037 FF1558004100            Call dword ptr \[00410058\]   
:0040703D 8B0DD4224100            mov ecx, dword ptr \[004122D4\]   
:00407043 330DF8224100            xor ecx, dword ptr \[004122F8\]   
:00407049 330DE0224100            xor ecx, dword ptr \[004122E0\]   
:0040704F 330DF0224100            xor ecx, dword ptr \[004122F0\]   
:00407055 898D18F3FFFF            mov dword ptr \[ebp+FFFFF318\], ecx   
:0040705B 8D9560F2FFFF            lea edx, dword ptr \[ebp+FFFFF260\]   
:00407061 52                      push edx        
:00407062 A124594100              mov eax, dword ptr \[00415924\]   
:00407067 50                      push eax   
  
\* Reference To: KERNEL32.SetThreadContext, Ord:0283h   
                                  |   
:00407068 FF158C004100            Call dword ptr \[0041008C\] 〈==关键啊！！！   
:0040706E 85C0                    test eax, eax   
:00407070 7523                    jne 00407095   
  
外壳调用 SetThreadContext 来改变 WEALTHLAB.TMP0 进程的 eip 来达到改变 EOP 的目的！！！   
关于 SetThreadContext 的说明请看 MSDN 文档。其中 edx 是 Context 指针，edx+b8 是 eip 的值；   
下命令“dd edx+b8"将看到 OEP 的地址是 006a19c4。（有关 Context 结构请看附录），用 peditor 将   
  
WEAL-NUPACK.exe 的 OEP 改为 006a19c4，到此脱壳完成，脱壳文件完美运行！！   
  
（四）总结   
    外壳作为调试器来调试加壳程序，并利用类似页异常的处理方式来动态解码，利用调试函数   
  
SetThreadContext 来动态改变加壳程序的 OEP，达到隐藏 OEP 的目的。   
    其实相对来说，Armadillo 比 aspr 要容易多了，（如果你对 window 的调试原理有了解的话）如   
  
果再加上类似 aspr 的花指令和反反复复的 seh，来加强反静态分析，应该是个保护很强的外壳。   
  
附录：   
  Context 结构的声明（来自 winnt.h 注释是我加的）   
typedef struct \_CONTEXT86 {   
    DWORD ContextFlags;   
  
    DWORD  Dr0;   
    DWORD  Dr1;   
    DWORD  Dr2;   
    DWORD  Dr3;   
    DWORD  Dr6;   
    DWORD  Dr7;   
  
    FLOATING\_SAVE\_AREA FloatSave;  //FLOATING\_SAVE\_AREA 的大小是 112byte   
    DWORD  SegGs;   
    DWORD  SegFs;   
    DWORD  SegEs;   
    DWORD  SegDs;   
  
    DWORD  Edi;   
    DWORD  Esi;   
    DWORD  Ebx;   
    DWORD  Edx;   
    DWORD  Ecx;   
    DWORD  Eax;   
  
    DWORD  Ebp;              // 偏移是 0xb8   
    DWORD  Eip;   
    DWORD  SegCs;              // MUST BE SANITIZED   
    DWORD  EFlags;            // MUST BE SANITIZED   
    DWORD  Esp;   
    DWORD  SegSs;   
  
} CONTEXT86;  4dd466d0e1e0cc123afbe504da2462ca