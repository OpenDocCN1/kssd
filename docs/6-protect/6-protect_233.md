

# FLEXlm9.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6939.htm`](https://www.pediy.com/kssd/pediy06/pediy6939.htm)

翻译第一篇 On Software Reverse Engineering - 1  
            Reverse Engineering, FLEXlm, IMSL  
    本文讨论了软件反向工程方法和 FLEXLM 系统的学习实例。  
问题描述  
    我下载了 IMSL\_CNL5.5(C 数字库)，但它由 FLEXLM 保护。对于不同的 OS 和编译器有不同的二进制下载，但通过平台可采用通用的破解技术（在我们的例子中，FLEXLM 许可证文件是当然的独立平台）。该产品主要包含了数学和统计库（静态和动态的）子程序，FLEXLM 的 FEATURE 名称为“CMATH”和"CSTAT"。X86 的发布版还有一个 CMATH 的优化版本叫做“CMPERF"，它通过捆绑 Intel MKL6.1 以实现更有效的运行。对于 CMPERF 没有额外的许可证。因为 CMATH 和 CSTAT 使用了相同的许可证机制，从现在开始，我们将聚焦在 CMATH 上。CSTAT 的（破解）过程是完全类似的。  
    出于有效的目的，该步提供了一个简单的程序”cmath.c"，将它和目标文件“cmath\_s.lib"链接以得到可执行文件。它调用 imsl\_f\_lin\_sol\_gen()并检查许可证文件，如果许可证文件不正确，将报告错误。从网上，我们找到 IMS\_CNL5.0 的如下许可证文件：  
SERVER hostname hostid 27000   
DAEMON VNI "<vni\_dir>\\license\\bin\\bin.i386nt\\vni.exe"  
FEATURE CMATH VNI 5.0 permanent uncounted 3F23BE3056E4 HOSTID=ANY  
FEATURE CSTAT VNI 5.0 permanent uncounted 2C60CD4570B0 HOSTID=ANY  
   我们试用它并如预料一样的得到“Version incorrect(版本不正确）”的错误；用 5.5 替换 5.0，得到“incorrect softkey code（不正确的软件加密代码）”错误，因此很明显的该方法不会工作。事实上，它采取了相当复杂的工作以应对 FLEXLM。软件破解有不同的级别，其相应的复杂范围从相对简单到相当困难－我们将在后面看到。现在，我们在下面列出了我们的任务和使用的工具。  
   对象:Visual Numerics IMSL CNL 5.5  
   保护:Macrovision FLEXlm 9.2  
   工具:Microsoft Visual Studio 7.1 (CL, NMAKE, DUMPBIN, LIB, …)  
        RedHat Cygwin 1.3.5  
        IDM UltraEdit 9.0  
        Datarescue IDA Pro 4.3 (FLAIR, …)  
        URSoft W32Dasm 8.9  
        Sysinternals File Monitor 6.0  
   资源:Macrovision FLEXlm 9.2 SDK source code  
        Macrovision FLEXlm 8.1 SDK binary release  
初步试探  
    所有的破解起步于信息收集。例如，通过在 UltraEdit 中搜索 cmath\_s.lib 很容易发现目标程序采用了 FLEXLM9.2 进行许可证管理。阅读相关的文献也是非常有帮助的。在\[4\]中有一些优秀的论文，讲述了以前攻击早期的 FLEXLM 版本，它们包含了许多珍贵的知识。  
    在实践中，我加载 cmath.exe 到 W32Dasm 调试器中并很快开始了漂亮的追踪。通过反汇编代码集追踪需要惊人的经验，可能你花费了数小时进行来回的跳转而没有得到程序实际运行的任何线索。但我试图找出一些（有用的）东西（在 File Monitor 的帮助下）：   
004133DE E89D2F0100 call 00426380 ;读 license.dat  
... ...  
00413411 – 0041345B ... ... ;循环读取“CMATH”  
... ...  
004138CD E83E670000 call 0041A010 ;必须返回 0 以通过检查  
004138D2 83C41C add esp, 0000001C  
004138D5 8945F0 mov dword ptr \[ebp-10\], eax ;返回值存储在 EAX 中  
004138D8 837DF000 cmp dword ptr \[ebp-10\], 00000000  
004138DC 0F848C070000 je 0041406E ;进行 imsl\_f\_lin\_sol\_gen()  
... ...  
00413907 8B8D18F7FFFF mov ecx, dword ptr \[ebp+FFFFF718\]  
0041390D 33C0 xor eax, eax  
0041390F 8A81A7444100 mov al, byte ptr \[ecx+004144A7\]  
00413915 FF24857F444100 jmp dword ptr \[4\*eax+0041447F\] ;跳到错误信息  
    很明显，子程序 0041A010 是关键。步入里面以揭示更多复杂的结构－在它里面有一些复杂的调用链。在实践中，该过程返回 FFFFFFF8 (or –8)，它将是错误代码。因此，我们在 004138D5 处设置一个断点，使用 W32Dasm"Modify Data(修改数据)"按键修改 EAX 寄存器的值为 0，并让它继续运行。哇噻!程序被迷惑了，并且输出了正确的结果就好像我们已有了正确的许可证数据。  
    现在，事情变得很明朗了，我们可以在代码段 004138D2 – 004138DC 上打补丁以设置返回值总是为 0。为了达到该目的，我们可以参照\[5\]中关于 MOV 和 JMP 的详细 X86 指令格式，剩下的事情就是修改代码了。注意，添加 NOP 用于补缀更改留下的特别的字节以使修改的可执行文件和原文件的长度相同。事实上，我们改变的仅有三行不同：  
 004138CD E83E670000 call 0041A010 ;返回必须为 0 以通过检查  
004138D2 83C41C add esp, 0000001C ;堆栈指针调整  
004138D5 C745F000000000 mov \[ebp-10\], 00000000 ;假装返回值为 O  
004138DC 90 nop ;补缀特别的字节以保持代码排列  
004138DD E98C070000 jmp 0041406E ;无条件跳转到 imsl\_f\_lin\_sol\_gen()  
    当然补丁 cmath.exe 仅是概念的证据，实际事情是补丁库自身。cmath.c per se 是非常短的，事实上 cmath.exe 的所有内容来自 cmath\_s.lib，包括 IMSL 函数和 FLEXLM 代码。为了定位上面的代码，我们在 UltraEdit\[2\]中搜索二进制串 837DF0000F848C070000 (关于 CMP 和 JE 行的代码)，它将引导我们到在 cmath\_s.lib 中代码放置的唯一位置(文件偏移 00106F70 – 00106F80)。照着更改字节，我们将得到一个补丁库。通过链接程序到新的补丁库进行测试，除了许可证文件无效外，所有功能都运行良好。  
     因此，我们已经取得了第一个成功。接着，我们对库的 DLL 版本做相同的事情，并且理想的开发一个工具自动进行而不是手工进行，但现在我们要省略它。在这点上，补丁通常是最容易的也是破解的第一步，它仅要求对保护方案有很少的洞察。此外，补丁仅针对一个特定的二进制目标工作，补丁工具对不同的版本或在不同的平台上无效。尽管补丁是强有力和有效的，但它距完全的反向工程还很远。  
进一步分析  
    我们采用通过一个假的"OK"来迷惑 FLEXLM，但是我们仍然不知道真的许可证代码。许多软件采用这种保护方式：基于用户信息（名称，组织，购买的特征等）和某种算法，程序计算一些 hash/checksum/license code(许可证代码)/signature(签名)，并将它和一个用户提供的（数据）进行比较。如果它们匹配，用户将被授权。对于我们所见的所有软件，该机制几乎是通用的，FLEXLM 也不例外。在我们的许可证文件中，代码 3F23BE3056E4 是"SIGN="签名。  
    关于 FLELXELM 更多的一点：之所以叫 FLEXLM 是因为它声称对商业软件许可证管理提供了一个灵活的解决方案。它作了许多努力以适应各种情况－counted/uncounted, feature/incremental, server/local, borrowing, trial, mobile, …（保持原文） –但是我们对这些并不感兴趣。我们所要的是正确的签名以使我们能在任何时间、任何地点运行具体的目标程序。  
因为我们已经有了 FLEXLM SDK9.2 的源代码，接下来的事情就是阅读它。FLEXLM SDK 是 Macrovision 提供给客户的所谓的" 守护神"，并帮助他们封装他们自己的"守护神软件“给”最终用户“。在我们的例子中，Visual Numerics（可视数字）是 Vendor(守护神)，IMSL CNL 5.5 是守护神软件，我们是最终用户。每个 Vendor 有一个唯一的名字或 Vendor ID。正如在许可证中所见，这里是"VNI"。通常，Vendors 仅能得到二进制版本或 FLEXLM SDK 的部分源（代码），但我们很幸运的得到了关于 FLEXLM 的完整的源代码。  
    但是，事实证明所有的 C 源代码（非 C++)并非易读的。 该工程演变超过了十年（1988 的第一版）;新旧函数叠加/缠绕在一起，常常伴有不必要的多余；它的注释很少并且一些旧类型代码习惯很差；宏的和处理指令的过多使用令人非常烦恼。最初是在 UNIX 平台上开发，后来通过 NMAKE 工具过渡到 Windows 环境。为有效的建立和调试如此大的一个应用程序，我们需要一个好的 IDE，但在 Windows 下没有如此的 IDE 能直接容纳 Makefile。在 Windows 上，Visual Studio 可能是最好的 IDE 并具有“Makefile Project"能力。我们为 FLEXLM SDK 着手创建一个 VS7 工程。它花了我一些时间完成－需要修改在 makefiles（制作文件）中的一些错误－但当它能用时，确实很方便。  
    FLEXLM 的核心组件是 lmgr.lib(或 lmgr9a.dll)，所有的其他组件都取决于它。Vendors 和最终用户更熟悉 lmgrd.exe,lmtools.exe,lmnewgen.exe, makekey.exe 等。在成功的建立后，我们试图产生 VNI 许可证文件但失败了，因为我们并没有其 vendors 和 seeds。通过\[2\], \[3\], \[4\]，每个 Vendor 从 Macrovision 接收到了 5 个 vendor 密码（VENDOR\_KEY1,… VENDOR\_KEY5），并且他们自身选择 3 个随机 seeds (LM\_SEED1, LM\_SEED2, LM\_SEED3)。这 8 个数放置在 lm\_code.h 中并被加密、模糊，最后嵌入到目标程序和产生许可证的工具中。我们的工作当然是回复这些数，但现在？  
    或许我们可以尝试容易一点的方法：因为目标程序将计算真正的签名并和从许可证中读取的进行对比，我们可以在对比发生时找到真正的签名而无需知道 Vendor 密码和 seeds。这并不是如看起来的那么简单，它要求我们在正确的时间、正确的位置设置断点。在数小时的追踪后，我们确定的仅是沿着指令流是一个死结，它无助于定位对比代码，除非 FLEXLM 使用一些标准的 API 如 strncmp()或 Win32CompareString()(实际上 FLEXLM 在 l\_privat.h 中定义它自己的宏 STRNCMP)。  
    这使我们产生了一个关于反向工程的基本问题：我们怎样能理解混乱的、高熵（？）的反汇编代码呢？这里没有完整的答案，我们可以参考\[7\]中关于一些严肃的理论讨论。但是这里我们想聚焦于一个特殊的技术：由 IDR Pro 介绍的 FLAIR(a.k.a. FLIRT, c.f. \[6\])。  
    我们都知道，如果应用程序由调试版本建立且符号文件(.PDB, .DBG 文件) 可获得时，调试将非常容易。符号是关于程序包括标识（变量、函数）名称和内存偏移、源代码行号等的信息。在一个调试建立编译器/链接器中，保存这些到应用程序二进制或单独的符号文件中。它们可以给调试者展现用户代码，将更接近于源代码（甚至更好，就象在 VS 中的源代码调试）。很显然，在我们的例子中，提交给最终用户的软件发布版本中的符号也被剥离出来了。  
但这并没有结束。即使没有符号，我们仍然可以从二进制文件、特别是库（文件）中得到一些有用的东西。在 Windows 上，EXE 和 DLL 是 PE 格式，而 OBJ 和 LIB 是 COFF 格式（LIB 不仅仅是一堆 OBJ 堆放在一起）；在两种格式库中的调用通过函数名称和参数进行，它们必须公开可视\[3\]。为达到该目的，PE 有输入和输入段，COFF 有符号表列。VS 提供了一组命令找寻它们。  
F:\\>dumpbin /disasm %vni\_dir%\\cnl55\\cmath.exe  
F:\\>dumpbin /rawdata %vni\_dir%\\cnl55\\cmath.exe  
F:\\>dumpbin /imports %vni\_dir%\\cnl55\\cmath.exe  
F:\\>dumpbin /exports %vni\_dir%\\cnl55\\bin\\cmath.dll  
F:\\>dumpbin /imports %vni\_dir%\\cnl55\\bin\\cmath.dll  
F:\\>dumpbin /exports %vni\_dir%\\cnl55\\lib\\cmath.lib  
F:\\>dumpbin /symbols %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
F:\\>dumpbin /linkermember %vni\_dir%\\cnl55\\lib\\cmath.lib  
F:\\>dumpbin /linkermember %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
F:\\>dumpbin /archivemembers %vni\_dir%\\cnl55\\lib\\cmath.lib  
F:\\>dumpbin /archivemembers %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
F:\\>lib /list %vni\_dir%\\cnl55\\lib\\cmath.lib  
F:\\>lib /list %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
F:\\>lib /extract:vc++\\flexlm.obj %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
F:\\>dumpbin /symbols /disasm flexlm.obj  
    在此，我们必须缩小 LIB 和 DLL 之间的差异，它仅仅是静态链接 VS 动态链接的差异。开发一个程序通常需要三步：  
初步:Source File (C, H, C++…), ASCII format/源文件 ASII 格式  
中间:Object File (OBJ, LIB…), COFF format /目标文件 COFF 格式  
最后:Image File (EXE, DLL, SYS…), PE format /映像文件 PE 格式  
    编译器处理源文件为目标文件，链接器将目标文件链接到输出的映像文件，加载器将映像文件从磁盘加载到内存。注意 DLL 已经被链接器处理了（在链接后），转换标识为内存地址或“被替换的数字字符”。相反的，目标文件是“链接前（的文件）”并保留了原始的符号；否则链接器将不能解析它们。因此 DLL 比 LIB 文件更接近于 EXE，哪怕(LIB)的名称为“库”。实际上，“动态链接”部分是在运行时由系统加载器进行的（一些修正/重定位），而不通过链接器。你可以说这是微软用了误导术语、迷惑用户并隐藏其技术要点的花招，但他们确实擅长于这些。  
    从一个破解者（原为黑客，我认为破解者更好）的视点看，这意味这越到后面，熵更高，信息更少。特别的，LIB 比 DLL 提供了更多的线索。DLL 仅输出了公开的 API 并隐藏了专用的（如在 cmath.dll 中仅 IMSL API 是输出的，而 FLEXLM 函数内部保留），但是 LIB 符号包含了两者。事实上，为了建立一个程序以调用 DLL API，我们需要传递其输入库到链接器。输入库是一个 COFF LIB 文件，它作为一个符号参考进行服务（通过指向 DLL），并不包含函数体。  
现在，我们可以发现在库文件中的函数，找出我们关心的函数，甚至可以提取相应的目标文件（仅针对 LIB）。下面是一个 imsl\_f\_lin\_sol\_gen()的例子。在我们讨论之前，驻留在 PE 调试段或单独的符号文件中的调试信息最好靠近于源代码（调试信息位于第一和第二步之间）。此外，我们在此获得的东西在反向工程中仍然是非常重要的（调试－建立符号是 COFF 符号的一个扩展集）：  
F:\\>dumpbin /archivemembers /symbols %vni\_dir%\\cnl55\\lib\\cmath\_s.lib|egrep "member|imsl\_f\_lin\_sol\_gen”  
... ...  
Archive member name at 1C8CEE: vc++\\gmres.obj/  
00B 00000000 SECT3 notype () External | \_imsl\_f\_lin\_sol\_gen\_min\_residual  
... ...  
Archive member name at 1DAAAA: vc++\\fspgen.obj/  
00B 00000000 SECT3 notype () External | \_imsl\_f\_lin\_sol\_gen\_coordinate  
05A 00000000 UNDEF notype () External | \_imsl\_f\_lin\_sol\_gen  
... ...  
Archive member name at 1FC10E: /3128 vc++\\fdmbndg.obj  
007 00000000 SECT2 notype () External | \_imsl\_f\_lin\_sol\_gen\_band  
... ...  
Archive member name at 58AC3E: /6132 vc++\\flinslg.obj  
007 00000000 SECT2 notype () External | \_imsl\_f\_lin\_sol\_gen  
... ...  
F:\\>dumpbin /exports %vni\_dir%\\cnl55\\bin\\cmath.dll | grep -i imsl\_f\_lin\_sol\_gen  
438 1B5 000432C0 imsl\_f\_lin\_sol\_gen  
439 1B6 0023B130 imsl\_f\_lin\_sol\_gen\_band  
440 1B7 0024AE30 imsl\_f\_lin\_sol\_gen\_coordinate  
441 1B8 00259C30 imsl\_f\_lin\_sol\_gen\_min\_residual  
 F:\\>lib /extract:vc++\\flinslg.obj %vni\_dir%\\cnl55\\lib\\cmath\_s.lib  
    IDA FLAIR 将带入更深入的一步。虽然在源代码中 API 调用看起来好像“x = imsl\_f\_lin\_sol\_gen(n, a, b, 0);”，在反汇编中它显示为“call 004033B0” (静态链接) or “call \[00402054\]” (动态链接)。沿着其内存地址的恰当的函数名称标记将使汇编分析容易得多，而这恰恰是 FLAIR 所能做到的。许多调试器有类似的注解功能，但通常限于输出的 API。IDA 试图扩展到包含尽可能多的函数。  
    FLAIR 的思想是为每个可标识的库函数创建一个“签名”，以使 IDA 在分析汇编代码时能认知和标记它。在名称指示时，它基本上就是一个认知的问题。再次，由于内容的不同，它仅为 LIB 工作而不针对发布的 DLL 版本。我们必须看到 DLL 有其优势诸如代码共享和主程序简化。例如，cmath.exe 的静态链接大小约 700KB，但其动态链接小于 4KB。但破解所关心的是，LIB 是比 DLL 更好的方式（当追踪 DLL 链接的应用程序时，我们的大多数时间都是在 10000000+ 或 80000000+ 区而不是熟悉的 00400000+ 区; 在 DLL 版本的 cmath.exe 中，指令“102D12BA: call 102D7980” 返回 FFFFFFF8）。  
    IDA FLAIR 并非完美的－它不能操作 DLL，一些函数不能被识别，可能发生错误的认知等。但它仍然是切实可行的。它最初的目的是分离样板 API(诸如 Win32,MFC,ATL 等\[6\])，因此我们可以聚焦在主程序算法上而不是标准的库函数上。在我们的实例中，我们更感兴趣的是得到这些重点的 FLEXLM 函数，因此我们不必步入每个调用以得到全部代码迷惑的一个大（结构）图。在实际中，我们创建了 cmath\_s.lib 和 lmgr.lib 的签名文件，在 IDA 中将它们应用到 cmath.exe，发现 FLAIR 做得很好－它认知了大多数 FLEXLM 函数。作为一个优秀的静态分析工具，IDA 也提供了一个 WinGraph32 特征称作“显示流程图表”。我发现它在对照源代码时特别有助于代码的理解。

 4dd466d0e1e0cc123afbe504da2462ca