
> **引用：*最初由 `readyu` 发布***
>
> `mov eax,[mem]` 比 `mov ecx,[mem]`
>
> `mov [mem],eax` 比 `mov [mem],ecx`
>
> 的操作码各短了一个字节。
>
> ```
> 000003C5: A1 00 00 00 00 mov eax,dword ptr \_nValidP...
> ```
>
> 我的意思是说为什么前面那么多行都完全一样，而后面的几十行本身也可以一样，但为什么它们不一样？
>
> 我分别用 `VC2003` 和 `VC2005` 的 `CL.exe` 编译，发现结果都和你说的一样。
>
> 估计原因可能是:
>
> 1. `MS` 内部编译器版本和我们用的有点不一样。
> 2. 分歧是从这一句开始的，这个宏的定义可能有些不一样。
>
> ```
> pSection = IMAGE_FIRST_SECTION(pNTHeader); // Defined in WINNT.H
> ```
>
> **引用：*最初由 `readyu` 发布***
>
> 我分别用 `VC2003` 和 `VC2005` 的 `CL.exe` 编译，发现结果都和你说的一样。
>
> 估计原因可能是:
>
> 1. `MS` 内部编译器版本和我们用的有点不一样。
> 2. 分歧是从这一句开始的，这个宏的定义可能有些不一样。
>
> ```
> pSection = IMAGE_FIRST_SECTION(pNTHeader); //...
> ```
>
> 这个问题我已经试过，我最初并不是用的这个宏，而是直接根据反汇编代码写的语句，但结果还是这样，与用不用这个宏没有关系。
>
> **引用：*最初由 `readyu` 发布***
>
> 我分别用 `VC2003` 和 `VC2005` 的 `CL.exe` 编译，发现结果都和你说的一样。
>
> 估计原因可能是:
>
> 1. `MS` 内部编译器版本和我们用的有点不一样。
> 2. 分歧是从这一句开始的，这个宏的定义可能有些不一样。
>
> ```
> pSection = IMAGE_FIRST_SECTION(pNTHeader); //...
> ```
>
> 我也怀疑是不是 `Microsoft` 使用的内部编译器与我们使用的不一样。
>
> 楼主的编译命令是从 `makefile` 里查到的嘛？
>
> 加上 `/Bd` 参数看看传给 `c1.dll` 和 `c2.dll` 的参数是什么？
>
> 或者用 `wrk` 里的 `cl` 试一下？
>
> **引用：*最初由 `Rinrin` 发布***
>
> 楼主的编译命令是从 `makefile` 里查到的嘛？
>
> 加上 `/Bd` 参数看看传给 `c1.dll` 和 `c2.dll` 的参数是什么？
>
> 或者用 `wrk` 里的 `cl` 试一下？
>
> 只有 `WIN32_LEAN_AND_MEAN` 这个宏的使用是从 `makefile` 文件中查到的，其它的是我自己实验出来的。
>
> `OpenRCE` 的 `RolfRolles` 答复如下：
>
> ```
> Nice job on the decompilation.
>
> My experience diffing patches (noticing that sometimes the only difference between two functions seemed to be register/stack allocation) makes me suspect that MSVC uses a non-deterministic register allocator. I may be wrong, and would be interested to hear the correct answer if so. Register allocation is often formulated as a graph-coloring problem, which is in NP, and therefore there can exist no algorithm to solve it optimally in polynomial time (assuming P != NP). Consequently, approximation algorithms are often used to "solve" problems such as this, and these algorithms often employ the use of random variables.
>
> Therefore, in general, there is no guarantee that two successive compilations of the same source code (or a conforming decompilation/recompilation) will see the same variables assigned to the same registers at the same program points. If the only differences between your recompilation and the original binary are register assignments / stack spills, you should consider it a job well-done; I would, anyway.
> ```
>
> `RolfRolles` 提到的寄存器分配问题通常用图着色问题来表示，关于这方面的详细信息，大家可以参考《`Advanced Compiler Design and Implementation`》（中文译名《高级编译器设计与实现》）第十六章。
>
> [`en.wikipedia.org/wiki/Register_allocation`](http://en.wikipedia.org/wiki/Register_allocation)
>
> `4dd466d0e1e0cc123afbe504da2462ca`
