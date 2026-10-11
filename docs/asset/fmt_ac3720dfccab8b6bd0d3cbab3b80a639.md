
当异常发生时，系统遍历 `EXCEPTION_REGISTRATION` 结构体链表直至找到处理此异常的处理程序。一旦找到了处理程序，系统再一次遍历此链表，直到处理异常的节点。在第二次遍历中，系统对所有的异常处理函数进行第二次调用。关键的区别就是在第二次调用中，异常标志被设为值 2。这个值对应着 `EH_UNWINDING`（`EH_UNWINDING` 的定义在 Visual C++ 运行时库源代码的 `EXCEPT.INC` 里，但 Win32 SDK 里并没有等价的定义）。

`EH_UNWINDING` 是什么意思呢？当异常回调被第二次调用时（带有 `EH_UNWINDING` 标志），操作系统就给处理函数一次做所需清理的机会。什么样的清理呢？一个很好的例子就是 C++ 类的析构函数。当函数的异常处理函数拒绝处理异常时，控制流一般并不会以正常的方式从函数中退出。现在考虑一个函数，此函数声明了一个局部的 C++ 类。C++ 规范指出析构函数是必须被调用的。第二次标志为 `EH_UNWINDING` 的异常处理回调就是为做调用析构函数和 `_finally` 块此类的清理工作提供机会。在异常被处理并且所有之前的 exception frames 都被调用以进行 unwind 之后，程序从回调函数选择的地方继续。但是记住，这并不等于将指令指针设为所要的代码地址并继续执行。继续执行出的代码要求堆栈和帧指针（Intel CPU 的 `ESP` 和 `EBP` 寄存器）都设为在处理异常的堆栈帧中相应的值。因此，接受某一异常的处理程序负责将堆栈指针和堆栈帧指针设为包含处理异常的 SEH 代码的堆栈帧中的值。

Figure 6 Unwinding from an Exception

更一般地说，从异常中的 unwinding 使得位于处理帧的堆栈区域之下的所有的东西都被移除，几乎相当于从未调用过那些函数。unwinding 的另一个效果就是链表中位于处理异常的 `EXCEPTION_REGISTRATION` 之前的所有 `EXCEPTION_REGISTRATIONs` 都被从链表中移除。这是有意义的，因为这些 `EXCEPTION_REGISTRATION` 一般都是在堆栈上构建的。在异常被处理后，堆栈指针和堆栈帧指针在内存中的地址要比从链表中移除的那些 `EXCEPTION_REGISTRATIONs` 高。Figure 6 所示即为所述。

### Help! Nobody Handled It!

到目前为止我都是假设操作系统总能在 `EXCEPTION_REGISTRATION` 链表中找到处理程序。要是没有相应的处理程序怎么办？这种情况几乎不会发生。原因是操作系统私地下为每个线程都准备了一个默认的异常处理程序。这个默认的异常处理程序总是链表的最后一个节点并总被选来处理异常。它的行为与一般的异常回调函数有些不同，我之后会说明。

我们来看一下系统在那里安插这个默认的、最终的异常处理程序。这显然要在线程执行的前期进行，要在任何用户代码执行之前。Figure 7 为我为 `BaseProcessStart` 写的伪码，`BaseProcessStart` 是 Windows NT 的 `KERNEL32.DLL` 的一个内部函数。`BaseProcessStart` 需要一个参数，即线程入口的地址。`BaseProcessStart` 运行在新进程的上下文中并调用入口点来启动进程第一个线程。

注意在伪码中，对 `lpfnEntryPoint` 的调用被封装在了一对 `_try` 和 `_except` 中。这个 `_try` 块就是用来在异常处理链表中安装那个默认的最终异常处理程序的。所有之后注册的异常处理程序都会插在链表中这个处理程序的前面。若 `lpfnEntryPoint` 函数返回，线程就运行至完成而不引起异常。若是这样，`BaseProcessStart` 调用 `ExitThread` 来结束线程。

要是另一种情况，即线程发生了异常却再也没有异常处理程序了怎么办？在这种情况下，控制流流进 `_except` 关键字后的大括号里。在 `BaseProcessStart` 里，这段代码叫 `UnhandledExceptionFilter` API，我在后面还会回来介绍它。现在的关键是 `UnhandledExceptionFilter` API 包含着默认的异常处理函数。

若 `UnhandledExceptionFilter` 返回的是 `EXCEPTION_EXECUTE_HANDLER`，`BaseProcessStart` 的 `_except` 块就执行。`_except` 块代码所作的就是调用 `ExitProcess` 来结束当前进程。仔细考虑一下，这样做还是有意义的；一个常识就是，如果程序引起了异常又没有处理程序能处理此异常，系统就结束该进程。伪码中所展示的正是这种情况。

还要最后补充一点。如果引发异常的线程是作为服务运行的且是用于一个基于线程的服务，则 `_except` 块并不会调用 `ExitProcess` 而是调用 `ExitThread`。没有人会因为一个服务出错而结束整个服务进程。

`UnhandledExceptionFilter` 中的默认异常处理程序又作了些什么呢？当我在讨论班上提出这个问题时，没几个人能猜出未处理的异常发生时操作系统的默认行为。通过对默认处理程序行为的演示，答案一点即明，人们就都明白了。我只是运行了一个主动引起异常的程序，并指出其结果（见 Figure 8）。

Figure 8 Unhandled Exception Dialog

`UnhandledExceptionFilter` 显示了一个对话框，告诉你发生了一个异常。此时，要么可以结束进程，要么就调试引发异常的进程。在这幕后还有相当多的操作，我在本文结束前再来讲这些东西。正如我所提到的，当异常发生时，用户编写的代码可以得到执行（通常是这样的）。类似地，在 unwind 操作过程中，用户编写的代码也可以得到执行。用户的代码可能仍有问题并引起另一个异常。因此，异常回调函数还可以返回另外两个值：`ExceptionNestedException` 和 `ExceptionCollidedUnwind`。显然这些内容就很深了，我并不想在这里介绍。其对于理解基本事实来说太难了。

## Compiler-level SEH

尽管我偶尔会使用 `_try` 和 `_except`，但目前我所讲到的都是由操作系统实现的。然而，看看我那两个使用纯操作系统 SEH 的程序的变态样子，编译器对此的封装实在是必要的。我们来看一下 Visual C++ 是如何在操作系统级的 SEH 支持之上构建其结构化异常处理的。

在继续进行之前要记住一件重要的事，那就是另一种编译器可能会与纯操作系统级的 SEH 的做法完全不同。没有人说过必须要实现 Win32 SDK 文档所描述的 `_try/_except` 模型。例如，Visual Basic 5.0 在其运行时代码里使用了结构化异常处理，但其数据结构与算法与我这里所讲的完全不同。若读一下 Win32 SDK 文档关于结构化异常处理的描述，就会找到所谓的“frame-based”的异常处理程序的语义，其形式如下：

```
try {
    // guarded body of code
}
except (filter-expression) {
    // exception-handler block
}
```

简单讲，try 中的所有的代码都被一个构建在函数堆栈帧上的 `EXCEPTION_REGISTRATION` 保护起来。在函数的入口，新的 `EXCEPTION_REGISTRATION` 被放入异常处理链表的表头。在 `_try` 块的结尾处，其 `EXCEPTION_REGISTRATION` 被从链表头移除。如前所述，异常处理链的表头保存在 `FS:[0]`。因此，若在调试器中的汇编代码中单步执行，就会看到以下的指令：

`MOV DWORD PTR FS:[00000000],ESP`

或是

`MOV DWORD PTR FS:[00000000],ECX`

可以十分确信代码正在建立或撤除一个 `_try/_except` 块。现在知道了一个 `_try` 块对应着堆栈上的一个 `EXCEPTION_REGISTRATION` 结构体，那 `EXCEPTION_REGISTRATION` 里的回调函数呢？使用 Win32 的术语，异常回调函数对应着 filter-expression 代号。filter-expression 就是关键字 `_except` 后括号中的代码。正是这个 filter-expression 代号决定了是否执行后面 {} 块中的代码。

因为 filter-expression 是程序员写的，程序员可以决定代码中某处发生的异常是否在该处处理。filter-expression 代码可以简单到只有一个“`EXCEPTION_EXECUTE_HANDLER`”，也可以调用一个函数把 p 算到两千万再返回一个代号告诉系统下一步做什么，这是程序员的选择。关键一点是：filter-expression 的代号正对应我前面提到的异常回调函数。

我刚才所讲的都十分简单，但是只是理想中的美好的情形。残酷的现实是事情要复杂的多。对于初学者，filter-expression 并不是由操作系统直接调用的。实际的情形是每个 `EXCEPTION_REGISTRATION` 的异常处理程序域都指向同一个函数。这个函数在 Visual C++ 的运行时库中，叫做 `__except_handler3`。是

另外一点就是并不是每次进入或退出 `_try` 块都要建立或撤除 `EXCEPTION_REGISTRATION`。对于使用 SEH 的每个函数，只创建一个 `EXCEPTION_REGISTRATION`。换句话说，在一个函数里可以使用多个 `_try/_except` 组合，但只在堆栈上建立一个 `EXCEPTION_REGISTRATION`。类似地，可以在一个函数的 `_try` 块中嵌套另一个 `_try` 块，Visual C++ 仍然只创建一个 `EXCEPTION_REGISTRATION`。如果对于整个 EXE 或 DLL 来说一个异常处理程序就足够了以及如果用一个 `EXCEPTION_REGISTRATION` 就可以处理多个 `_try` 块，那显然还要有比所见到的更多的机制。这是通过一个一般情况下看不到的表中的数据来完成的。然而，既然本文的目的就是要解剖结构化异常处理，我们就来看一下这些数据结构。

## The Extended Exception Handling Frame

Visual C++ 的 SEH 实现并没有使用纯粹的 `EXCEPTION_REGISTRATION` 结构，而是在结构体的末尾加入了额外的数据域。这个额外的数据的关键之处在于它允许一个函数（`__except_handler3`）来处理所有的异常并将控制流转向相应的 filter-expressions 和代码中的 `_except` 块。关于这个 Visual C++ 扩展的 `EXCEPTION_REGISTRATION` 的一点信息可以从 Visual C++ 的运行时库源代码中的 `EXSUP.INC` 文件里找到。在这个文件里，可以找到以下定义：

```
;struct _EXCEPTION_REGISTRATION{
;     struct _EXCEPTION_REGISTRATION *prev;
;     void (*handler)(PEXCEPTION_RECORD,
;                     PEXCEPTION_REGISTRATION,
;                     PCONTEXT,
;                     PEXCEPTION_RECORD);
;     struct scopetable_entry *scopetable;
;     int trylevel;
;     int _ebp;
;     PEXCEPTION_POINTERS xpointers;
;};
```

前两个域前面已经见过了，`prev` 和 `handler`。他们组成了最基本的 `EXCEPTION_REGISTRATION` 结构体。新加的是最后的三个域：`scopetable`、`trylevel` 和 `_ebp`。`scopetable` 域指向一个 `scopetable_entries` 类型结构体数组，而 `trylevel` 是这个数组的索引。最后一个域，`_ebp`，是创建 `EXCEPTION_REGISTRATION` 之前的堆栈帧指针（`EBP`）的值。

`_ebp` 域成为扩展的 `EXCEPTION_REGISTRATION` 结构体的一部分不是偶然的。结构体包含它是因为大多数函数都以一个 `PUSH EBP` 开始。这就使得所有其它的 `EXCEPTION_REGISTRATION` 域可以通过帧指针的负偏移来访问。例如，`trylevel` 在 `[EBP-04]`，`scopetable` 指针在 `[EBP-08]` 等等。

在扩展的 `EXCEPTION_REGISTRATION` 结构体后面，Visual C++ 压入了两个额外的值。第一个 `DWORD` 为一个指向 `EXCEPTION_POINTERS` 结构体（一个标准的 Win32 结构体）的指针保留空间。这个指针就是调用 `GetExceptionInformation` API 返回的指针。尽管 SDK 文档隐含提到 `GetExceptionInformation` 是一个标准的 Win32 API，但事实上 `GetExceptionInformation` 是一个编译器相关的函数。当调用此函数时，Visual C++ 生成下面的代码：

`MOV EAX,DWORD PTR [EBP-14]`

与 `GetExceptionInformation` 相同，`GetExceptionCode` 也依赖于编译器。`GetExceptionCode` 返回的值是 `GetExceptionInformation` 返回的数据结构中一个域的值。Visual C++ 会生成以下的代码，这些代码的作用留给读者作为练习。

`MOV EAX,DWORD PTR [EBP-14]`

`MOV EAX,DWORD PTR [EAX]`

`MOV EAX,DWORD PTR [EAX]`

回到扩展的 `EXCEPTION_REGISTRATION` 结构体，在结构体起始处之前的 8 个字节处，Visual C++ 保留了一个 `DWORD` 来保存所有 prologue 代码执行后最终的堆栈指针（`ESP`）。这个 `DWORD` 就是函数执行时一个普通的 `ESP` 寄存器的值（当然参数压栈是为了准备调用另外函数的情况除外）。

看起来我好像一股脑儿倒出了一大堆东西，确实是。在向下继续之前，我们先暂停一会儿，复习一下 Visual C++ 为用到结构化异常处理的函数生成的标准异常帧：

`EBP-00 _ebp`

`EBP-04 trylevel`

`EBP-08 scopetable pointer`

`EBP-0C handler function address`

`EBP-10 previous EXCEPTION_REGISTRATION`

`EBP-14 GetExceptionPointers`

`EBP-18 Standard ESP in frame`

从操作系统的观点来看，构成纯 `EXCEPTION_REGISTRATION` 的域仅有两个：`[EBP-10]` 处的 `prev` 指针和 `[EBP-0Ch]` 处的处理函数指针。帧中其它的东西都是依赖于 Visual C++ 实现的。记住这些后，我们来看包含着编译器级 SEH 的 Visual C++ 的运行时库函数，`__except_handler3`。

## __except_handler3 and the scopetable

尽管我非常想将 Visual C++ 的运行时库源代码指点出来并让读者自己去研究 `__except_handler3` 函数，但是我不能，因为此函数的代码并未提供。这里只好用我仓促拼凑出的 `__except_handler3` 的伪码来应付一下了（见 Figure 9）。

尽管 `__except_handler3` 看上去是成堆的代码，但要记着它只是一个异常回调函数，就像我在文章开头介绍的那样。和 `MYSEH.EXE` 和 `MYSEH2.EXE` 中的 homegrown 异常回调函数一样，此函数也需要四个参数。在最高一级上，`__except_handler3` 被一个 `if` 语句分为了两部分。这是因为函数可以被调用两次，一次是正常调用，一次是在 unwind 过程中。大部分的代码都用在了 non-unwinding 的回调中。

这段代码的开头首先在堆栈上创建一个 `EXCEPTION_POINTERS` 结构体，并用 `__except_handler3` 的两个参数将其初始化。此结构体的地址，即伪码中的 `exceptPtrs`，被放在 `[EBP-14]`。这就初始化了 `GetExceptionInformation` 和 `GetExceptionCode` 函数用到的指针。接着，`__except_handler3` 从 `EXCEPTION_REGISTRATION` 帧（`[EBP-04]`）取得当前的 `trylevel`。`trylevel` 变量用作 `scopetable` 数组的索引，使得一个 `EXCEPTION_REGISTRATION` 可以用于一个函数中的多个 `_try` 块和嵌套的 `_try` 块。每一个 `scopetable` 的成员定义如下：

```
typedef struct _SCOPETABLE
{
     DWORD       previousTryLevel;
     DWORD       lpfnFilter
     DWORD       lpfnHandler
} SCOPETABLE, *PSCOPETABLE;
```
