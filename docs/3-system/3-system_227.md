

# Windows XP 中的向量化异常处理

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-233.htm`](https://www.pediy.com/kssd/pediy09/pediy09-233.htm)

> 本文是 Matt Pietrek 在 2001 年 9 月的 MSDN 杂志上发表的关于 Windows XP 中新增的向量化异常处理（Vectored Exception Handling）方面的文章。作者深入剖析了向量化异常处理机制以及它的用途，并给出一个非常实用的样例程序。译者将其内容更新至当前最流行的 Windows XP SP2。希望它能对大家有所帮助。译者尽了最大努力来保证其正确性，有疏漏之处还请读者能够指正。  
>   
> 注：附件中是本文的 PDF 格式，可读性好  
>   
> Windows XP 中的新型向量化异常处理  
> 原名：New Vectored Exception Handling in Windows XP  
> 作者：Matt Pietrek  
>   
> 到现在为止我已在 Win32教ㄉ瞎ぷ靼四暧杏啵谡馄诩淅镂一哿艘恍┪宜不妒褂玫腤 in32 功能（从 API 层面上来说）。它们可以让我的编程生活更轻松，同时也让我更容易写出比较有用的工具。当我安装完 Windows XP Beta（以前代号为“Whistler”）时，并没有指望能够看到许多新的 API，结果却惊喜地发现我错了！在本月的专栏中，我就要讲述这些新增功能其中之一——向量化异常处理（Vectored Exception Handling）。  
>   
> 当运行我在 1997 年 11 月 MSJ 杂志 Under The Hood 专栏中介绍的 PEDIFF 程序时，我发现了向量化异常处理。你告诉 PEDIFF 两个不同版本 DLL 的路径，它就会返回这两个 DLL 导出的不同函数。这一次我比较了 Windows 2000 中的 KERNEL32.DLL 和 Windows XP 中的 KERNEL32.DLL，结果发现了向量化异常处理。Windows XP 中的 KERNEL32.DLL 中新增了许多 API，但我一眼就注意到了 AddVectoredExceptionHandler 这个 API。在最新的 MSDN?nbsp;Library 中有这个 API 的文档，因此我也不需要挖掘这方面的信息了。  
>   
> 由于 Beta 2 版本中的 WINBASE.H 有一个问题，因此你需要安装 Platform SDK RC1 发行版才能编译本文中所讲的代码。  
>   
> 快速回顾结构化异常处理  
> 到底什么是向量化异常处理，为什么要关注它呢？首先让我们来快速回顾一下通常的异常处理机制，这样你就会明白向量化异常处理与它之间的区别了。如果你使用的是像 C++那样的支持异常处理的语言，你很可能已经知道 Win32 结构化异常处理（Structured Exception Handling，SEH）了。在 C++中是使用 try/catch 语句或者是 Microsoft C/C++编译器扩展的\_\_try/\_\_except 语句来实现结构化异常处理的。关于 SEH 的内部工作机制，可以参考我在 1997 年 1 月的 MSJ 杂志上发表的文章“深入探索 Win32 结构化异常处理（A Crash Course on the Depths of Win32 Structured Exception Handling）”。  
> 简单地说，结构化异常处理使用了基于堆栈的异常结点。当你使用 try 块时，有关异常处理程序的信息被保存在当前过程（函数）的堆栈帧上。在 x86 平台上，Microsoft 使用了保存在 FS:\[0\]处的指针来指向当前的异常处理程序帧。这个帧中包含了异常发生时需要被调用的代码的地址。  
> 如果你在一个 try 块中调用另一个函数，这个新函数可能设置它自己的异常处理程序。此时在堆栈上就创建了一个新的异常处理程序帧并且有一个指针指向前一个异常处理程序帧，如图 1 所示。实际上，所有的 SEH 帧形成了一个链表，FS:\[0\]指向这个链表的头部。这里要注意的关键地方是：链表中每个后继结点必须处于线程堆栈上更高的位置。操作系统强制实行这个特别的规定，这就意味着你不能随意地将自己的异常处理程序插入到这个链表中。  
>    
> 图 1 堆栈上的异常处理程序  
>   
> 所有的异常处理帧在堆栈上以链表的形式存在并不是 SEH 中的一个小细节，它对 SEH 的正常工作是至关重要的。当异常发生时，操作系统从这个链表的头部开始，用类似于“现在发生了一个异常，你想处理它吗？”这样的代码来调用这个异常处理程序。异常处理程序可以修复出错的问题并返回 EXCEPTION\_CONTINUE\_EXECUTION 来表示它处理了这个异常。  
> 异常处理程序也可以通过返回 EXCEPTION\_CONTINUE\_SEARCH 来表示它拒绝处理异常。当发生这种情况时，操作系统移向链表中的下一个结点，并询问同样的问题。这个过程一直进行下去，直至某个异常处理程序选择处理这个异常或者到链表末尾。我在这里极大地简化了 SEH 的细节，但这对于我们来说已经足够了。  
> SEH 如此设计导致的后果是什么呢？一个很重要的地方是一个异常处理程序可以自由选择某个异常发生时所采取的动作而不用顾及在它前面安装的异常处理程序（在链表中位于后面）想采取的动作。但是，有时候这可能是个要关心的主要问题。下面这个例子解释了其中的缘由。  
>   
> 假设你写出了世界上最棒的异常处理程序。当异常发生时，你的异常处理程序可以诊断这个问题，记录相关的细节并解决这个问题。并且你把它放到了你的 main（或 WinMain）函数中，以便它可以保护整个程序。  
>   
> 现在，假设在你的程序的某个地方调用了一个外部组件，这个组件是不受你控制的。它也安装了一个异常处理程序，但这个异常处理程序写的不好。它遇到一个异常时就让进程退出了。这样，你的异常处理程序根本就没有得到机会执行，因为在异常处理程序链表中一个出现在它前面的异常处理程序处理了这个异常。总之，SEH 这个极好的设计让一个事实将其威力减弱了，那就是，只有在调用链中更深处的代码不安装它们的异常处理程序时当前的异常处理程序才能发挥作用。  
>   
> 请允许我在开始讲述向量化异常处理之前再提一些与 SEH 有关的内容。当一个正在被调试的程序中产生异常时，异常处理过程会增加一些步骤。首先，调试器被给予第一次机会来处理这个异常，或者让其子进程（即被调试程序）处理。如果子进程处理这个异常，那接下来就是前面讲的步骤。如果子进程中没有相应的异常处理程序来处理这个异常，调试器就会接到第二次处理机会的通知。（通常调试器弹出一个“未处理异常”对话框就是在这个时候。）此时，只有结束进程了。  
> 向量化异常处理  
>     简单地说，向量化异常处理类似于正常的 SEH，但有以下三点关键区别：  
> "  异常处理程序既不针对于特定的函数，也不针对于堆栈帧。  
> "  编译器中并不存在将新的异常处理程序添加到异常处理程序链表中的关键字（例如 try 或 catch）。  
> "  向量化异常处理程序是由你在自己的代码中明确添加的，而不是伴随 try/catch 之类的语句而产生的。  
> 新增的 AddVectoredExceptionHandler 这个 API 将一个指向函数的指针作为参数，把这个函数的地址添加到已注册的异常处理程序链表中。由于系统使用一个链表来保存向量化异常处理程序，因此一个程序想安装多少个向量化异常处理程序都可以。  
>   
> 向量化异常处理与结构化异常处理是如何共存的呢？在 Windows XP 中，当一个进程中产生异常时，向量化异常处理程序链表先于正常的 SEH 链表被处理。这可以很好地解决了现存代码的兼容性问题。如果向量化异常处理程序链表在 SEH 链表之后被处理，SEH 异常处理程序可能处理那个异常，这样向量化异常处理程序就没有机会处理它了。  
>   
> 在调试方面，向量化异常处理与结构化异常处理类似。也就是说，当一个程序被调试时，调试器仍然能够在目标进程（也就是被调试程序）处理异常之前得到第一次通知。只有当调试器选择将异常传递给子进程时（通常是这样），向量化异常处理程序才被调用。  
>   
> AddVectoredExceptionHandler 函数被声明在 WINBASE.H 文件中：  
> WINBASEAPI PVOID WINAPI AddVectoredExceptionHandler(  
>             ULONG FirstHandler,  
>           PVECTORED\_EXCEPTION\_HANDLER VectoredHandler );  
> (在 Windows Server!" 2003 和 Windows Vista!" SDK 的头文件中此函数的参数名称分别为 First 和 Handler，但相应的文档却仍是上面这个样子)  
>   
> 此函数的第一个参数告诉系统这个异常处理程序应该放在异常处理程序链表的开头还是结尾。异常处理程序链表并不针对于任何线程，它对进程来说是全局的。因此虽然你可以请求系统将它放在链表的开头，但却无法保证它会被首先调用。因为如果其它代码在你调用此函数之后才调用此函数并也请求被放在链表开头，那你就不会被首先调用。无论 AddVectoredExceptionHandler 何时被调用，新安装的异常处理程序总是要么被放在链表开头，要么被放在链表末尾。  
>   
> 第二个参数是一个异常处理函数的地址。它的原型如下：   
> LONG NTAPI VectoredExceptionHandler(PEXCEPTION\_POINTERS ExceptionInfo);  
> ExceptionInfo 参数是一个指针，它指向一个结构，这个结构中包含了这个函数可以获取的关于异常的所有信息，例如异常类型、地址以及异常产生时寄存器的值。这个函数或者返回 EXCEPTION\_CONTINUE\_SEARCH，或者返回 EXCEPTION\_CONTINUE\_EXECUTION。  
>   
> 当返回 EXCEPTION\_CONTINUE\_EXECUTION 时，系统会尝试继续执行出错进程。如果是这样的话，出现在这个处理程序后面的向量化异常处理程序以及所有的结构化异常处理程序都不会被调用。当返回 EXCEPTION\_CONTINUE\_SEARCH 时，系统移向下一个向量化异常处理程序。在所有的向量化异常处理程序都被调用之后，系统开始处理结构化异常处理程序链表。  
>   
> 与 AddVectoredExceptionHandler 这个 API 相对的还有一个 RemoveVectoredExceptionHandler，它从链表中移去前面安装的处理程序。这个函数并没有什么特别的地方，为了完整起见，我才在这里提起它。  
>   
> 抢占正常的 SEH 处理过程是许多系统级的程序员长期以来梦寐以求的。然而直到使用向量化异常处理才使它完全变成了现实。向量化异常处理程序可以返回 EXCEPTION\_CONTINUE\_EXECUTION，从而导致链表中后续的处理程序都不被调用。但是某一部分代码可能希望处理某些异常，这样如果你不把异常传递过去可能会引入错误。Microsoft 已经引进了向量化异常处理这个极强的新功能，因此你不能粗心地认为你的向量化异常处理程序是惟一注册的异常处理程序而将它搞得一团糟。  
>   
> 全面展示向量化异常处理  
>   
>       对于那些开发跟踪和诊断工具的人来说，在需要时获取控制权的经典方法是使用断点。不幸的是，使用断点就意味着要处理异常，具体来说就是断点异常和单步异常。使用结构化异常处理机制来处理这些异常并不可行，因为你并不能保证你的异常处理程序总能捕获这些异常。  
>   
> 一些工具（例如 Mutek 的 BugTrapper）通过覆盖 NTDLL 中部分用户模式下的异常处理代码来解决这个问题。一个可能覆盖的地方就是 NTDLL 中的 KiUserExceptionDispatcher 函数，我在前面提到的 MSJ 中那篇有关结构化异常处理方面的文章中已经讲过这个函数。虽然覆盖 KiUserExceptionDispatcher 可以达到目的，但这种方案比较脆弱。一旦出现新版本的 NTDLL，它很可能就失效了。  
>   
> 使用向量化异常处理就不需要做这些复杂的工作。如果所有的异常处理程序都按照我前面所说的那样做的话，向量化异常处理就是一个整洁的、容易扩展的捕获这些异常的方法。为了演示向量化异常处理，我创建了一个使用断点来监视对 LoadLibrary 调用情况的小程序。在这个程序中，只要 LoadLibrary 被调用，它就可以打印出将要加载的 DLL 的名称。  
>   
> 一些高级用户可能想到为导入地址表（Import Address Table，IAT）打“补丁”来解决这个问题。它当然可以和我这里使用的基于断点的方法达到同样效果。虽然在这种情况下你可以使用为 IAT 打“补丁”的方法来解决问题，但是它需要编写许多代码。你必须为所有用到的 DLL 都打“补丁”，包括那些通过调用 LoadLibrary 来动态加载的 DLL。请相信我，这些工作实际做起来比看起来困难的多。使用断点是所有这些方法中比较简单的。  
>   
> 为 IAT 打“补丁”遇到的第二个问题就是，它只适用于导出的函数。但是断点技术却适用于任何代码，而不仅仅是导出的函数。因此它对于像拦截静态链接到运行时库（相对于使用 MSVCRT.DLL）的代码对 malloc 的调用之类的事情是非常有用的。  
>   
> 图 2 是使用向量化异常处理来监视对 LoadLibrary 调用情况的那个 DLL 的代码。每当 LoadLibrary 被调用时，VectoredExcBP 就把要加载的 DLL 的名称写到标准输出中。这个 DLL 是自包含的，它并不需要任何特别的外部初始化。你只需要链接到它惟一的导出函数上就可以拿它做实验了。  
>   
> 我也写了一个调用 LoadLibrary 来加载几个常见 DLL 的演示程序，它叫做 TestVE（图 3）。TestVE 链接到 VectoredExcBP.DLL 导出的一个空函数上，这使得这个 DLL 在程序初始化时会被加载到内存当中。  
>   
> 当 VectoredExcBP 被加载时，它的 DllMain 函数调用我的 SetupLoadLibraryExWCallback 函数。这个函数使用 AddVectoredExceptionHandler 这个 API 来注册一个异常处理程序。另外，它查找 KERNEL32.DLL 中的 LoadLibraryExW 函数的地址，在这个函数的第一条指令上设置一个断点。  
>   
> VectoredExcBP 中的主要部分是 LoadLibraryBreakpointHandler 函数。传递给 AddVectoredExceptionHandler 函数的就是它的地址。每当异常发生时，这个函数就获取控制权。它只处理两个特别的异常。对于任何不感兴趣的异常，它都返回 EXCEPTION\_CONTINUE\_SEARCH，以便让其它异常处理程序来处理它。  
>   
> 我在这里并不过多地描述调试器方面的理论知识，只简要地描述一下当断点被触发时和程序恢复执行时的事件顺序。当 CPU 执行断点指令时，首先生成一个 STATUS\_BREAKPOINT 异常。当它发生时，目标函数中的任何代码都尚未执行。因此这个时候是检查函数参数和进行其它操作的最佳时机。  
>   
> 由于断点已经覆盖了原来的指令，因此接下来就是恢复原来的指令以便它可以正常执行。一般来说，这并不是什么大问题。但是这里有一个问题。如果你恢复了原来的指令并让它继续执行，那么你的断点就不存在了，这样你以后就不能再中断在这个地方了。  
>   
> 解决办法（至少是在 x86 处理器上）是让 CPU 每次执行一条指令（单步），然后将控制权返回给你，这样你就可以重新插入断点。在 x86 处理器上要想单步执行需要设置 CPU 的 EFlags 寄存器中的跟踪标志（TF，有时也称为自陷标志或陷阱标志，值为 0x100）。当跟踪标志置位时，CPU 每次仅执行一条指令，然后就生成一个 STATUS\_SINGLE\_STEP 异常。在接收到 STATUS\_SINGLE\_STEP 异常之后，将 TF 标志清除，以便恢复程序的正常执行。  
>   
> 当你仔细查看 LoadLibraryBreakpointHandler 函数代码时，会发现它与上面讲的算法完全吻合。这段代码在执行时相当谨慎，它检查异常发生的地址是否是它所要处理的。该讲的我在前面已经讲过了，勿庸赘述。这段代码包含了许多注释，很容易理解。  
>   
> 在对 STATUS\_BREAKPOINT 的处理代码中，LoadLibraryBreakpointHandler 调用了一个我定义的名为 BreakpointCallback 的函数。这个函数使用异常发生时堆栈指针中的值来寻找参数的值。对于 LoadLibrary 函数来说，它只有一个参数，那就是指向它要加载的 DLL 的名称字符串的指针。BreakpointCallback 函数获取这个指针的值并输出相应的字符串。（如果你想将这个 DLL 用于非控制台模式的应用程序的话，你当然也可以将 printf 改成类似 OutputDebugString 之类的函数。）  
>   
> 你可能想知道我为什么会选择监视 LoadLibraryExW 函数。有一个很好的理由可以解释其中的缘由。因为 LoadLibrary 函数带一个字符串参数，它同时有 ANSI 和 UNICODE 这两个版本。LoadLibrary 最常用的形式是 LoadLibraryA。但实际上，LoadLibraryA 只是对 LoadLibraryExA 进行了简单的封装，而 LoadLibraryExA 只是对 LoadLibraryExW 进行了简单的封装。同样，LoadLibraryW 也只是对 LoadLibraryExW 进行了简单的封装。这样，所有调用 DLL 的方法最终都归结为调用 LoadLibraryExW 函数。只需要在这个 API 上设置一个断点，我就可以捕获所有对 LoadLibrary 函数及其变种的调用。  
>   
> 如果想实验一下 VectoredExcBP，确保你使用的是 Windows XP Beta 2 或者更新版本的操作系统，然后运行 TestVE 程序。TestVE 自身只是通过调用 LoadLibrary 来加载两个 DLL（MFC42.DLL 和 WININET.DLL）。但是这两个 DLL 内部也要加载其它 DLL，因此你会看到其它一些附加的对 LoadLibrary 的调用。如果一切正常，你会看到类似下面的输出结果：  
>   
> LoadLibrary called on: MFC42  
> LoadLibrary called on: MSVCRT.DLL  
> LoadLibrary called on: G:\\WINDOWS\\System32\\MFC42LOC.DLL  
> LoadLibrary called on: WININET  
> LoadLibrary called on: kernel32.dll  
> LoadLibrary called on: advapi32.dll  
> LoadLibrary called on: kernel32.dll  
>   
> 向量化异常处理的实现  
> Windows XP Beta 2 中的向量化异常处理在实现上相当简单。虽然表面上 AddVectoredExceptionHandler 这个 API 是在 KERNEL32.DLL 中，但是它实际上转发到了 NTDLL 中的 RtlAddVectoredExceptionHandler 函数上。图 4 是我为这个函数写的伪代码。  
> 向量化异常处理程序链表是一个循环链表（译者注）。每个已注册的异常处理程序用一个 12 字节的结点表示，这个结点所占用的内存在进程堆上分配。同时用一个临界区对象来保护将异常处理程序结点插入到链表的头部或尾部的这部分代码。如果 FirstHandler 参数不为 0，则新的异常处理程序结点被插入到链表的头部；否则它被插入到链表的尾部。就这么简单！它并没有检查这是不是以前曾经注册过的某个结点进行的重复注册，这样某个异常处理程序可以注册（因此也会被调用）多次。  
> 在向量化异常处理的实现中另一个值得注意的地方是这些向量化异常处理程序被调用的方式。正如在我那篇关于 SEH 的文章中讲到的那样，KiUserExceptionDispatcher（在 NTDLL 中）调用了 RtlDispatchException。图 5 说明了有关向量化异常处理的代码是如何被添加到 NTDLL 中的 RtlDispatchException 函数中的。如果你把它和我以前的那篇文章中的伪代码比较一下，会发现它仅在 RtlDispatchException 函数开头添加了一个函数调用（调用 RtlCallVectorExceptionHandlers 函数）。这充分证明了向量化异常处理程序是在结构化异常处理程序之前被调用的。  
> 图 6 是 RtlCallVectorExceptionHandlers 函数的伪代码。同样，这段代码也十分简单。它使用一个临界区对象来保护一个 while 循环。在这个循环中，它遍历异常处理程序链表并调用相应的异常处理程序。如果异常处理程序返回 EXCEPTION\_CONTINUE\_EXECUTION，这个循环就直接退出而不再调用后续的处理程序。它负责返回一个值来指示 RtlDispatchException 是否应该去调用结构化异常处理程序。  
> 正如你所料，我认为向量化异常处理是 Windows XP 中添加的非常重要的功能。我已经期盼这个功能被添加到 Win32 中很长时间了。在本文中，我已经向你演示了使用向量化异常处理的极大优点，希望这个功能在以后会有新的应用。  
>   
> 附录  
>   
> 图 2 VectoredExcBP   
> //===========================================================================  
> // VectoredExcBP   Matt Pietrek 2001  
> // MSDN Magazine, September 2001  
> //  
> // !^!^!^! 警告 警告 警告 !^!^!^!  
> //  这段代码只能运行在 Windows XP 及其后继操作系统上。  
> //  要想正确地编译它，你必须做到：  
> //      A) 拥有一个定义了 AddVectoredExceptionHandler 函数的 WINBASE.H 文件  
> //      B) 确保编译器最先找到的 WINBASE.H 文件是前面提到的那个而不是其它的。  
> //         如果你不能确定如何做，请参考编译器文档。   
> //      C) 定义\_WIN32\_WINNT=0x0500（或更高）  
> //  
> // 这段代码在 Windows XP Beta 2 下用 Visual C++ 6.0 可以通过编译并正常运行。  
> // 在写这段代码时，Windows XP 还处于 beta 测试阶段，因此某些内容可能改变，  
> // 其中包括 API 的行为等等。不能保证它可以在将来的 Windows 系统上正常运行。  
> //===========================================================================  
> #include “stdafx.h”  
>   
> #ifndef \_M\_IX86  
> #error “This code only runs on an x86 architecture CPU”  
> #endif  
>   
> LONG NTAPI LoadLibraryBreakpointHandler(PEXCEPTION\_POINTERS pExceptionInfo );  
> void BreakpointCallback( PVOID pCodeAddr, PVOID pStackAddr );  
> void SetupLoadLibraryExWCallback(void);  
> BYTE SetBreakpoint( PVOID pAddr );  
> void RemoveBreakpoint( PVOID pAddr, BYTE bOriginalOpcode );  
>   
> // 全局变量  
> PVOID g\_pfnLoadLibraryAddress = 0;  
> BYTE g\_originalCodeByte;  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
>   
> BOOL APIENTRY DllMain( HANDLE hModule,   
>                        DWORD  ul\_reason\_for\_call,   
>                        LPVOID lpReserved  
>                      )  
> {  
> // 我们不需要接收有关线程启动/停止时的通知，因此禁用它们  
> DisableThreadLibraryCalls( (HINSTANCE)hModule );  
>   
> // 在进程启动时设置断点，进程退出时移除断点  
> if ( DLL\_PROCESS\_ATTACH == ul\_reason\_for\_call )  
>         SetupLoadLibraryExWCallback();  
> else if ( DLL\_PROCESS\_DETACH == ul\_reason\_for\_call )  
>         RemoveBreakpoint( g\_pfnLoadLibraryAddress, g\_originalCodeByte );  
>   
> return TRUE;  
> }  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
>   
> void SetupLoadLibraryExWCallback(void)  
> {  
> // 获取 LoadLibraryExW 函数的地址。  
> // 所有对 LoadLibraryA/W/ExA 的调用最终都转到了 LoadLibraryExW  
> g\_pfnLoadLibraryAddress=(PVOID)GetProcAddress(GetModuleHandle(“KERNEL32”),  
>                                                     “LoadLibraryExW” );  
>   
> // 为我们的断点添加一个向量化异常处理程序  
> AddVectoredExceptionHandler( 1, LoadLibraryBreakpointHandler );  
>   
> // 在 LoadLibraryExW 上设置断点  
> g\_originalCodeByte = SetBreakpoint( g\_pfnLoadLibraryAddress );  
> }  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> // 这是 LoadLibraryExW 断点的处理程序。当断点被触发时，调用相应的回调  
> // 函数（BreakpointCallback）。然后单步执行原来的指令并继续执行下去。  
> // 实际上这需要处理两个异常，正如代码中所处理的那样。  
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
>   
> LONG NTAPI LoadLibraryBreakpointHandler(PEXCEPTION\_POINTERS   
>                                         pExceptionInfo )  
> {  
> // printf( “In LoadLibraryBreakpointHandler: EIP=%p\\n”,  
> //          pExceptionInfo->ExceptionRecord->ExceptionAddress );  
>   
> LONG exceptionCode = pExceptionInfo->ExceptionRecord->ExceptionCode;  
>   
> if ( exceptionCode == STATUS\_BREAKPOINT)  
> {  
>         // 检查它是否是我们设置的断点。如果不是，将它传递给其它处理程序  
>         if ( pExceptionInfo->ExceptionRecord->ExceptionAddress  
>             != g\_pfnLoadLibraryAddress )  
>         {  
>             return EXCEPTION\_CONTINUE\_SEARCH;  
>         }  
>   
>         // 我们需要单步执行原来的指令，因此必须临时移去断点  
>         RemoveBreakpoint( g\_pfnLoadLibraryAddress, g\_originalCodeByte );  
>   
>         // 调用我们的代码以执行所需的任务  
>         BreakpointCallback( pExceptionInfo->ExceptionRecord->ExceptionAddress,  
>                             (PVOID)pExceptionInfo->ContextRecord->Esp );  
>   
>         // 设置 EFlags 寄存器中的跟踪标志，以便我们在捕获   
>         // STATUS\_SINGLE\_STEP 异常之前只执行一条指令（看下面）  
>         pExceptionInfo->ContextRecord->EFlags |= 0x00000100;  
>   
>         return EXCEPTION\_CONTINUE\_EXECUTION;    // 重新执行出错指令  
> }  
> else if ( exceptionCode == STATUS\_SINGLE\_STEP )  
> {  
>         // 检查异常地址是否是上面产生的那个异常的地址的下一个地址  
>         // 如果不是，将它传递给其它异常处理程序  
>   // 译者注：由于 Windows XP SP2 增强了系统的安全性，实际上许多  
>   // 系统 API 的 prolog 代码已经发生了变化。通常情况下系统 API 的第  
>   // 一条指令为 PUSH EBP，它的机器码为 0x55，占一个字节，因此作者  
>   // 才说要检查异常地址是否是上面产生的那个异常的地址的下一个地址  
>   //（+1）。而在 Windows XP SP2 中，LoadLibraryExW 的第一条指令变成  
>   // 了 PUSH 34h，它的机器码为 0x6A34，占两个字节，因此我们需要将  
>   // 其改为+2 才能正确执行  
>         if ( pExceptionInfo->ExceptionRecord->ExceptionAddress  
>             != (PVOID)((DWORD\_PTR)g\_pfnLoadLibraryAddress+2) )  
>         {  
>             return EXCEPTION\_CONTINUE\_SEARCH;  
>         }  
>   
>         // printf( “In STATUS\_SINGLE\_STEP handler\\n” );  
>   
>         // 我们已经单步执行完了原来的指令，因此重新设置断点  
>         SetBreakpoint( g\_pfnLoadLibraryAddress );  
>   
>         // 清除前面设置的跟踪标志  
>         pExceptionInfo->ContextRecord->EFlags &= ~0x00000100;  
>   
>         return EXCEPTION\_CONTINUE\_EXECUTION;    // 继续运行！  
> }  
> else    // 不是断点异常或单步异常。它不是我们处理的目标！  
> {  
>         return EXCEPTION\_CONTINUE\_SEARCH;  
> }  
> }  
>   
> /\*++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> // 当 LoadLibraryExW 被调用时就调用此函数。它的参数为断点的地址和堆栈指针。  
> // 堆栈指针可以用来获取堆栈上的参数值。此时我们想获取的是表示将要加载的  
> // DLL 的名称的字符串（UNICODE 格式）。  
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> void BreakpointCallback( PVOID pCodeAddr, PVOID pStackAddr )  
> {  
> DWORD nBytes;  
>   
> LPWSTR pwszDllName;  
>   
> // pStackAddr+0 == 返回地址  
> // pStackAddr+4 == 第一个参数  
> ReadProcessMemory(  GetCurrentProcess(),  
>                         (PVOID)((DWORD\_PTR)pStackAddr+4),  
>                         &pwszDllName, sizeof(pwszDllName),  
>                         &nBytes );  
>       
> printf( “LoadLibrary called on: %ls\\n”, pwszDllName );  
> }  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> // 在指定地址处设置断点，返回这个地址处原来的内容。  
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> BYTE SetBreakpoint( PVOID pAddr )  
> {  
> DWORD nBytes;  
> BYTE bOriginalOpcode;  
>   
> // 读取指定地址处的内容  
> ReadProcessMemory( GetCurrentProcess(), pAddr,  
>                         &bOriginalOpcode, sizeof(bOriginalOpcode),  
>                         &nBytes);  
>   
> // 设置断点  
> BYTE bpOpcode = 0xCC;  
> WriteProcessMemory( GetCurrentProcess(), pAddr,  
>                         &bpOpcode, sizeof(bpOpcode),  
>                         &nBytes );  
>   
> return bOriginalOpcode;   
> }  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> // 将断点处原来的内容写回到那个地址  
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> void RemoveBreakpoint( PVOID pAddr, BYTE bOriginalOpcode )  
> {  
> DWORD nBytes;  
>   
> WriteProcessMemory( GetCurrentProcess(), pAddr,  
>                         &bOriginalOpcode, sizeof(bOriginalOpcode),  
>                         &nBytes );  
> }  
>   
> /\*+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++\*/  
> extern “C” void \_\_declspec(dllexport) VectoredExcBP\_ExportedAPI(void)  
> {  
> // 这个函数什么也不做。导出它以便其它 EXE 可以链接到这个 DLL 上。  
> }  
> 返回  
>   
> 图 3 TestVE   
> #define WIN32\_LEAN\_AND\_MEAN  
> #include <windows.h>  
>   
> // 由 VectoredExcBP.DLL 导出的函数的原型声明  
> Extern “C” void VectoredExcBP\_ExportedAPI(void);  
>   
> int main()  
> {  
> // 加载两个 DLL，在它们内部会加载其它 DLL。  
> LoadLibrary( “MFC42” );  
>   
> LoadLibrary( “WININET” );  
>   
> // 调用 VectoredExcBP.DLL 导出的函数。  
> // 这条语句可以强制加载这个 DLL  
> VectoredExcBP\_ExportedAPI();  
>   
> return 0;  
> }  
> 返回  
>   
> 图 4 RtlAddVectoredExceptionHandler 函数伪代码  
>   
> struct \_VECTORED\_EXCEPTION\_NODE  
> {  
> DWORD   m\_pNextNode;  
> DWORD   m\_pPreviousNode;  
> PVOID   m\_pfnVectoredHandler;  
> }  
>   
> CRITICAL\_SECTION RtlpCalloutEntryLock;  
> \_VECTORED\_EXCEPTION\_NODE \*RtlpCalloutEntryList;  
>   
> RtlAddVectoredExceptionHandler( ULONG FirstHandler,  
>         PVECTORED\_EXCEPTION\_HANDLER VectoredHandler )  
> {  
> // 为新结点分配空间  
> PVOID pExcptNode = HeapAlloc( GetProcessHeap(), 0, 0xC );  
> if ( !pExcptNode )  
>         return 0;  
>   
> pExcptNode->m\_pfnVectoredHandler = VectoredHandler;  
>       
> RtlEnterCriticalSection( &RtlpCalloutEntryLock );  
>   
> if ( FirstHandler )  
> {  
>         pExcptNode->m\_pNextNode = RtlpCalloutEntryList->m\_pNextNode;  
>         pExcptNode->m\_pPreviousNode = &RtlpCalloutEntryList;  
>         RtlpCalloutEntryList->m\_pNextNode->m\_pPreviousNode = pExcptNode;  
>         RtlpCalloutEntryList->m\_pNextNode = pExcptNode;  
> }  
> else  
> {  
>         pExcptNode->m\_pNextNode = &RtlpCalloutEntryList;  
>         RtlpCalloutEntryList->m\_pPreviousNode->m\_pNextNode = pExcptNode;   
>         pExcptNode->m\_pPreviousNode = RtlpCalloutEntryList->m\_pPreviousNode;  
>         RtlpCalloutEntryList->m\_pPreviousNode = pExcptNode;  
> }  
>   
> RtlLeaveCriticalSection( &RtlpCalloutEntryLock );  
>   
> return pExcptNode;  
> }  
> 返回  
>   
> 图 5 RtlDispatchException 函数的伪代码  
>   
> RtlDispatchException( PEXCEPTION\_RECORD pExcptRec, CONTEXT \* pContext )  
> {  
> DWORD    stackUserBase;  
> DWORD    stackUserTop;      
> PEXCEPTION\_REGISTRATION pRegistrationFrame;  
> DWORD hLog;  
>   
> // 这一句是新增加的代码  
> RtlCallVectoredExceptionHandlers( pExcptRec, pContext );  
>   
> // 从 FS:\[4\]和 FS:\[8\]处获取堆栈界限  
> RtlpGetStackLimits( &stackUserBase, &stackUserTop );  
>   
> pRegistrationFrame = RtlpGetRegistrationHead();  
>   
> // 其余部分省略……  
> 返回  
>   
> 图 6 RtlCallVectoredExceptionHandlers 函数的伪代码  
>   
> // 此函数由 RtlDispatchException 调用  
> RtlCallVectoredExceptionHandlers( PEXCEPTION\_RECORD pExcptRec,CONTEXT \* pContext )  
> {  
> bool bContinueExecution = FALSE;  
>   
> // 用临界区保护异常回调  
> RtlEnterCriticalSection( &RtlpCalloutEntryLock );  
>   
> // 获取链表头部  
> pCurrentNode = RtlpCalloutEntryList;  
>   
> // 当我们还没有处理这个结点时……  
> while ( pCurrentNode != RtlpCalloutEntryList )  
> {  
>         // 调用异常处理函数  
>         EXCEPTION\_POINTERS pExceptionPointers  
>         LONG disposition = pCurrentNode->m\_pfnVectoredHandler( &pExcptRec );  
>   
>         // 如果处理程序允许恢复执行，中断我们的循环  
>         if ( disposition == EXCEPTION\_CONTINUE\_EXECUTION )  
>         {  
>             bContinueExecution = TRUE;  
>             break;  
>         }  
>   
>         // 移向下一个结点  
>         pCurrentNode = pCurrentNode->m\_pNextNode;  
> }  
>   
> RtlLeaveCriticalSection( &RtlpCalloutEntryLock );  
>   
> return bContinueExecution;  
> }  
> 返回  
>   
> 译者注：  
>   
> 在 Windows XP SP2 中，译者发现向量化异常处理程序链表并不是循环链表，而是一个双向链表，这个链表的头结点是一个全局变量。如果你对 Windows 中的链表处理比较熟悉的话，你会知道，在 Windows 中遇到链表时一般会使用 SINGLE\_LIST\_ENTRY 和 LIST\_ENTRY 这两个结构并结合系统自身提供的 API（例如 InsertHeadList 和 RemoveHeadList 等，实际上它们都是内联函数或宏， NTDDK.H 和 WDM.H 中都有它们的源代码）来处理，而且系统要求所处理的链表都必须有头结点。下面是我根据作者的伪代码为 Windows XP SP2 中的这些函数写的伪代码。  
>   
> VEH.H 文件  
>   
> typedef struct \_VECTORED\_EXCEPTION\_NODE {  
>   LIST\_ENTRY ListEntry;  
>   PVECTORED\_EXCEPTION\_HANDLER  pfnHandler;  // 该指针出于安全目的已经被加密  
> } VECTORED\_EXCEPTION\_NODE, \*PVECTORED\_EXCEPTION\_NODE;  
>   
> // 下面两个全局变量由 Windows 加载器进行初始化  
> // 实际进行初始化的是 LdrpInitializeProcess 函数  
>   
> // 此变量负责保护链表的插入与删除操作，由 Windows 加载器调用  
> // RtlInitializeCriticalSection 对其进行初始化  
> RTL\_CRITICAL\_SECTION RtlpCalloutEntryLock;  
>   
> // 此变量是向量化异常处理链表的头结点，由 Windows 加载器调用  
> // InitializeListHead 对其进行初始化  
> // 关于 Windows 中链表操作方面的详细信息请参考 DDK 文档中  
> // "Singly- and Doubly-Linked Lists"一节  
> LIST\_ENTRY RtlpCalloutEntryList;  
>   
> VEH.C 文件  
>   
>   
> #include <windows.h>  
> #include "veh.h"  
>   
> // 此原型来自于 Windows Server 2003 R2 SDK 中的 WINBASE.H 文件  
> PVOID WINAPI RtlAddVectoredExceptionHandler(  
>           ULONG First,  
>           PVECTORED\_EXCEPTION\_HANDLER Handler)  
> {  
>   PVECTORED\_EXCEPTION\_NODE pCurrentNode = (PVECTORED\_EXCEPTION\_NODE)  
>     RtlAllocateHeap( GetProcessHeap(), 0, sizeof(VECTORED\_EXCEPTION\_NODE) );  
>   
>   if (!pCurrentNode)  
>   {  
>     return 0;  
>   }  
>   
>   // EncodePointer 函数实际转发到了 NTDLL.DLL 中的 RtlEncodePointer 函数上  
>   pCurrentNode->pfnHandler = RtlEncodePointer(Handler);  //出于安全目的加密  
>   
>   // EnterCriticalSection 函数实际转发到了 NTDLL.DLL 中的  
> // RtlEnterCriticalSection 函数上  
>   RtlEnterCriticalSection(&RtlpCalloutEntryLock);  
>   
>   if (First)  
>   {  
>     InsertHeadList(&RtlpCalloutEntryList, &pCurrentNode->ListEntry);  
>   }  
>   else  
>   {  
>     InsertTailList(&RtlpCalloutEntryList, &pCurrentNode->ListEntry);  
>   }  
>   
>   RtlLeaveCriticalSection(&RtlpCalloutEntryLock);  
>   
>   return pCurrentNode;  
> }  
>   
>   
> ULONG WINAPI RemoveVectoredExceptionHandler(PVOID Handler)  
> {  
>   PVECTORED\_EXCEPTION\_NODE pCurrentNode, pRemovedNode;  
>   BOOL bHandlerExist = FALSE;  
>   
>   RtlEnterCriticalSection(&RtlpCalloutEntryLock);  
>   
>   for ((PLIST\_ENTRY)pCurrentNode  = RtlpCalloutEntryList.Flink;  
>         (PLIST\_ENTRY)pCurrentNode != &RtlpCalloutEntryList;  
>                    (PLIST\_ENTRY)pCurrentNode=pCurrentNode->ListEntry.Flink)  
>   {  
>     if (pCurrentNode == (PVECTORED\_EXCEPTION\_NODE)Handler)  
>     {  
>       pRemovedNode = pCurrentNode;  
>       RemoveEntryList(&pCurrentNode->ListEntry);  
>   
>       bHandlerExist = TRUE;  
>       break;  
>     }  
>   }  
>   
>   RtlLeaveCriticalSection(&RtlpCalloutEntryLock);  
>   
>   if (bHandlerExist)  
>   {  
>     RtlFreeHeap(GetProcessHeap(), 0, (LPVOID)pRemovedNode);  
>   
>     return 1;  
>   }  
>   
>   return 0;  
> }  
>   
>   
> int WINAPI RtlCallVectoredExceptionHanlers(  
>         PEXCEPTION\_RECORD pExcptRec,  
>         PCONTEXT pContext)  
> {  
>   EXCEPTION\_POINTERS ExceptionPointers;  
>   
>   // 检查链表是否为空  
>   if (RtlpCalloutEntryList.Flink == &RtlpCalloutEntryList)  
>   {  
>     return 0;  
>   }  
>   else  
>   {  
>     PVECTORED\_EXCEPTION\_NODE pCurrentNode;  
>     BYTE retValue = 0;  
>   
>     ExceptionPointers.ExceptionRecord = pExcptRec;  
>     ExceptionPointers.ContextRecord   = pContext;  
>   
>     RtlEnterCriticalSection(&RtlpCalloutEntryLock);  
>   
>     for((PLIST\_ENTRY)pCurrentNode=RtlpCalloutEntryList.Flink;  
>          (PLIST\_ENTRY) pCurrentNode != &RtlpCalloutEntryList;  
>          (PLIST\_ENTRY)pCurrentNode=pCurrentNode->ListEntry.Flink)  
>     {  
> LONG disposition = ((PVECTORED\_EXCEPTION\_HANDLER)  
> (RtlDecodePointer(pCurrentNode->pfnHandler)))  
> (&ExceptionPointers);  
>       if (disposition == EXCEPTION\_CONTINUE\_EXECUTION)  
>       {  
>         retValue = 1;  
>         break;  
>       }  
>     }  
>   
>     RtlLeaveCriticalSection(&RtlpCalloutEntryLock);  
>   
>     return retValue;  
>   }  
> }  
>   
> RtlDispatchException.c 文件  
>   
> RtlDispatchException( PEXCEPTION\_RECORD pExcptRec, CONTEXT \* pContext )  
> {  
>     DWORD    stackUserBase;  
>     DWORD    stackUserTop;      
>     PEXCEPTION\_REGISTRATION pRegistrationFrame;  
>     DWORD hLog;  
>     BYTE retValue = 0;  
>   
>     // 这一句是新增加的代码  
>     if (RtlCallVectoredExceptionHandlers( pExcptRec, pContext ))  
>     {  
>     retValue = 1;  
>     }  
>    else  
>    {  
>   // 从 FS:\[4\]和 FS:\[8\]处获取堆栈界限  
>     RtlpGetStackLimits( &stackUserBase, &stackUserTop );  
>   
>     pRegistrationFrame = RtlpGetRegistrationHead();  
>     // 其余部分省略……  
>    }  
>   
>    return retValue;  
> }  
>   
> 对于上面的伪代码，我要说的有以下几点：  
> 一、  对全局变量 RtlpCalloutEntryList 类型的识别。这个变量的名称是公开的，当我用   SoftICE 定位到这个变量的地址时看到以下内容：  
>    
> 很明显，这个变量占 8 个字节，然后结合 Windows 处理链表时对链表的要求以及  
> RtlAddVectoredExceptionHandler 的反汇编代码可以得出它是一个 LIST\_ENTRY 结构。  
> 二、  这两个全局变量的初始化。在这两个变量位置上设置内存写断点很容易找出是哪段代码对其进行初始化的，并且是如何初始化的。前面注释中已经给出，不再赘述。  
> 三、  作者给出的伪代码中的循环是用的 while 语句，根据这些函数的反汇编代码，其中进入循环时的语句形式为  
> MOV ……（一条或多条 MOV 指令）  
> JMP ……  
> 这明显是编译器为 for 语句生成的指令序列，故我在伪代码中使用了 for 循环语句。  
>   
> [（Microsoft System Journal 2001 年 9 月 Under The Hood 专栏）](http://msdn.microsoft.com/msdnmag/issues/01/09/hood/default.aspx)  
> 译者：SmartTech    电子信箱：zhzhtst@163.com 

 4dd466d0e1e0cc123afbe504da2462ca