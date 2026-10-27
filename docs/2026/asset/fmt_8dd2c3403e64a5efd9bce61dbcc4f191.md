
回到`__except_handler3`的代码，在取得当前`trylevel`之后，代码就指向了相应的`SCOPETABLE`成员并调用了`filter-expression`代码。若`filter-expression`返回`EXCEPTION_CONTINUE_SEARCH`，`__except_handler3`继续查找下一个`SCOPETABLE`成员，这个成员由`previousTryLevel`域指定。若在遍历过程中没有找到处理程序，`__except_handler3`就返回`DISPOSITION_CONTINUE_SEARCH`，这就使系统移向下一个`EXCEPTION_REGISTRATION`帧。

若`filter-expression`返回`EXCEPTION_EXECUTE_HANDLER`，则意味着异常应该由相应的`_except`块代码来处理。这就意味着所有之前的`EXCEPTION_REGISTRATION`帧都要从链表中移除而且要执行`_except`块。第一个活儿是通过调用`__global_unwind2`来完成的，之后我再介绍。在一些清理代码之后，代码的执行就离开了`__except_handler3`并进入`_except`块。奇怪的是控制流从未从`_except`块返回，尽管`__except_handler3`调用了它。如何设置当前的`trylevel`呢？这是由编译器暗自处理的，编译器以`on-the-fly`的方式完成对扩展的`EXCEPTION_REGISTRATION`结构体中的`trylevel`域的修改。如果察看使用`SEH`的函数的汇编代码就会发现函数代码的不同位置都有修改`[EBP-04]`处的当前`trylevel`的代码。`__except_handler3`如何调用的`_except`，而控制流又为何从不返回呢？因为一个`CALL`指令将返回地址压入堆栈，可以认为这就打乱了堆栈。如果察看一下为`_except`块生成的代码，就会发现它所作的第一件事就是将`EXCEPTION_REGISTRATION`结构体之后的8字节处的`DWORD`加载到`ESP`寄存器中。作为其`prologue`代码的一部分，函数将`ESP`的值保存起来，这样`_except`之后还可以将其取回。

## The ShowSEHFrames Program

此时是不是对`EXCEPTION_REGISTRATIONs`、`scopetables`、`trylevels`、`filter-expressions`和`unwinding`这些东西感到有些招架不住，我当初也是这样的。编译器级的结构化异常处理的主题对更多的学习并没有什么帮助。如果没有总体上的了解的话，其中的很多东西就没有意义。当面对一大堆理论时，我很自然地倾向于写些使用这些理论的代码。如果程序能工作，我就知道我的理解（通常是）是正确的。

`Figure 10`是`ShowSEHFrames.EXE`的源代码。它使用`_try/_except`块来建立起由几个`Visual C++ SEH`帧构成的链表。之后，显示每一帧的信息，以及`Visual C++`为每一帧建立的`scopetables`。程序并不生成任何异常。我包含了所有的`_try`块来强制`Visual C++`生成多个`EXCEPTION_REGISTRATION`帧，每一帧有多个`scopetable`成员。

`ShowSEHFrames`里重要的函数是`WalkSEHFrames`和`ShowSEHFrame`。`WalkSEHFrames`首先打印出`__except_handler3`的地址，原因一会儿再讲。接着，函数从`FS:[0]`得到一个指向异常链表表头的指针然后遍历链表中的每一个节点。每个节点都是`VC_EXCEPTION_REGISTRATION`类型的，我定义这个结构体是为了描述`Visual C++`的异常处理帧。对于链表中的每一个节点，`WalkSEHFrames`将指向节点的指针传递给`ShowSEHFrame`函数。

`ShowSEHFrame`首先打印异常帧的地址、回调函数地址、前一异常帧的地址和一个指向`scopetable`的指针。接着，对于每一个`scopetable`成员，代码打印出`previous trylevel`，`filter-expression`的地址和`_except`块的地址。我又是怎么知道`scopetable`中到底有多少个成员的呢？其实我并不知道。我假设`VC_EXCEPTION_REGISTRATION`中的当前`trylevel`比`scopetable`的成员总数少一。

`Figure 11`所示即为`ShowSEHFrames`的运行结果。首先看以“Frame:”开头的每一行。注意每个后继的实例是如何显示堆栈上高地址的异常帧的。接着，在前三个`Frame:`行里，注意`Handler`的值都是相同的（`004012A8`）。看看输出的开头就知道这个`004012A8`就是`Visual C++`运行时库的`__except_handler3`函数的地址。这就证实了我前面所说的一个成员处理多个异常。

也许有人会疑惑，因为`ShowSEHFrames`只有两个使用`SEH`的函数，而却有三个使用`__except_handler3`作为回调函数的异常帧。第三个异常帧来自于`Visual C++`的运行时库。`Visual C++`的运行时库的`CRT0.C`源代码显示对`main`或`WinMain`的调用被封装在了`_try/_except`块中。这个`_try`块的`filter-expression`代码在`WINXFLTR.C`文件中。

回到`ShowSEHFrames`，最后一帧的`Handler:`此行包含一个不同的地址，`77F3AB6C`。经过查找，就会发现这个地址是在`KERNEL32.DLL`里。这个特殊的帧是由`KERNEL32.DLL`的`BaseProcessStart`函数安装的，这个函数我在前面讲到过。

## Unwinding

在深挖`unwinding`的实现代码之前，我们先来简要总结一下`unwinding`的含义。前面我曾提到异常处理程序信息是如何保存在链表里的，又是如何由线程信息块的第一个`DWORD`（`FS:[0]`）来指向的。因为某一异常的处理程序不一定是链表的头节点，这就需要有一种有序的方法来移除此实际处理程序之前链表中的所有异常处理程序。

正如在`Visual C++`的`__except_handler3`函数中见到的，`unwinding`是由`__global_unwind2 RTL`函数完成的。此函数是对未公开的`RtlUnwind API`的非常简单的封装：

```
__global_unwind2(void * pRegistFrame)
{
     _RtlUnwind( pRegistFrame,
                 &__ret_label,
                 0, 0 );
     __ret_label:
}
```

尽管`RtlUnwind`是实现编译器级`SEH`的关键的`API`，但却没有公开。它是一个`KERNEL32`函数，`Windows NT`将`KERNEL32.DLL`调用`forward`到了`NTDLL.DLL`，在`NTDLL.DLL`里的也是一个`RtlUnwind`函数。我拼凑了这个函数的一些伪码，即`Figure 12`所示。

尽管`RtlUnwind`看起来很繁琐，但如果合理地划分一下还是不难理解的。此`API`首先从`FS:[4]`和`FS:[8]`取得线程堆栈的当前栈顶和栈底。这两个值对于后面的健壮性检查是很重要的，这里的健壮性检查就是保证所有被`unwound`的异常帧都落在堆栈的范围内。

接着，`RtlUnwind`在堆栈上建立一个`EXCEPTION_RECORD`，并将`ExceptionCode`域设为`STATUS_UNWIND`。而且`EXCEPTION_RECORD`的`ExceptionFlags`域中的`EXCEPTION_UNWINDING`标志也要置位。指向此`EXCEPTION_RECORD`结构体的指针之后会作为参数传递给每一个异常回调函数。此后，代码调用`_RtlpCaptureContext`函数来创建一个`CONTEXT`结构体，此结构体也会作为异常回调的`unwind`调用的一个参数。`RtlUnwind`后面的部分就遍历`EXCEPTION_REGISTRATION`结构体的链表。对于每一帧，代码调用`RtlpExecuteHandlerForUnwind`函数，后面会讲到此函数。正是这个函数用`EXCEPTION_UNWINDING`标志调用了异常回调函数。每次回调之后，相应的异常帧通过调用`RtlpUnlinkHandler`将其移除。当`RtlUnwind`到达第一个参数指定地址的帧时，就停止`unwinding`帧。这些代码中间还有许多用于错误检查的代码，这些代码保证了程序的正常执行。如果出现了问题，`RtlUnwind`就会引起异常来告知所遇到的问题，而且此异常的`EXCEPTION_NONCONTINUABLE`标志是置位的。当此标志置位时，是不允许进程继续执行的，因此进程必须结束。

## Unhandled Exceptions

本文前面部分我完整描述了`UnhandledExceptionFilter API`。一般不用直接调用这个`API`（尽管可以）。大多数情况下，它是由`KERNEL32`的默认异常回调的`filter-expression`代码来调用的。前面的`BaseProcessStart`伪码说明了这一点。

`Figure 13`是我给出的`UnhandledExceptionFilter`的伪码。这个`API`的开头有些奇怪（至少在我看来是这样的）。若是一个`EXCEPTION_ACCESS_VIOLATION`异常，代码就调用`_BasepCheckForReadOnlyResource`。尽管我没有提供此函数的伪码，但我这里可以大概说一下。如果是因为向`EXE`或`DLL`的`resource section`（`.rsrc`）进行写入而发生异常，`_BasepCurrentTopLevelFilter`就修改引起异常的页的属性，从而允许写操作，`UnhandledExceptionFilter`返回`EXCEPTION_CONTINUE_EXECUTION`，并重新执行引起异常的指令。

`UnhandledExceptionFilter`的下一个任务是决定进程是否要在`Win32`调试器下运行。也就是说，进程是以`DEBUG_PROCESS`或`DEBUG_ONLY_THIS_PROCESS`标志创建的。`UnhandledExceptionFilter`使用`NtQueryInformationProcess`函数来判断进程是否正在被调试。如果是，此`API`就返回`EXCEPTION_CONTINUE_SEARCH`，说明系统的其它部分会唤醒调试器进程并告知调试器被调试进程引起了异常。如果有`user-installed unhandled exception filter`，则调用它。一般都没有`user-installed`的回调，但是可以用`SetUnhandledExceptionFilter API`装一个。我提供了这个`API`的伪码。这个`API`只是用新的用户回调的地址来修改一个全局变量，然后返回旧回调的值。

做好准备工作后，`UnhandledExceptionFilter`就可以进行其主要的工作：用那个老面孔的应用程序错误对话框来通知程序的错误。有两种办法可以避免此对话框的出现。第一种就是进程调用了`SetErrorMode`并设置了`SEM_NOGPFAULTERRORBOX`标志。另一种就是将`AeDebug`注册键值下的`Auto`值设为`1`。这时，`UnhandledExceptionFilter`略过程序错误对话框并自动启动由`AeDebug`键的`Debugger`值所指定的调试器。如果对“just in time debugging”比较熟悉的话，这就是操作系统对其的支持，之后还会讨论。

大多数情况下，这两种逃避此对话框的条件都为假，`UnhandledExceptionFilter`就调用`NTDLL.DLL`函数中的`NtRaiseHardError`函数。正是这个函数唤出了程序错误对话框。这个对话框等待用户点击`OK`结束进程或`Cancel`调试进程。

若点击了`OK`，`UnhandledExceptionFilter`就返回`EXCEPTION_EXECUTE_HANDLER`。调用`UnhandledExceptionFilter`的代码通常以结束自己来回应（就像`BaseProcessStart`代码中那样的）。这就带来一个有趣的问题。多数人认为系统没有处理异常而将进程结束。实际上更准确的说法是系统作了一些工作，这样未处理的异常使进程自己将自己结束。

如果点击了程序错误对话框的`Cancel`才执行了`UnhandledExceptionFilter`真正有意思的代码，这时会调试器加载引起异常的进程。在调试器`attach`到出错进程后，代码首先调用`CreateEvent`来创建一个事件以通知调试器。事件句柄和当前进程`ID`都要传递给`sprintf`，`sprintf`格式化启动调试器的命令行。万事俱备后，`UnhandledExceptionFilter`调用`CreateProcess`来启动调试器。若`CreateProcess`成功，代码对前面创建的事件调用`NtWaitForSingleObject`。此调用一直阻塞直到调试器进程通知此事件，指示调试器已经成功地`attach`到出错进程上。`UnhandledExceptionFilter`还有其它的零星代码，但我这里只捡重要的说了说。

## Into the Inferno

到了目前这个地步，如果还保留什么就太不公平了。我已经讲了发生异常时操作系统如何调用用户定义的函数；讲了一般回调的内部运行以及编译器如何使用它们来实现`_try`和`_catch`；讲了没人处理异常时的情况以及系统对其的处理。所剩下的只有起初异常回调是从何处开始的。是的，我们来深入系统内幕来看看结构化异常处理的开始阶段。

`Figure 14`所示为我为`KiUserExceptionDispatcher`和一些相关函数写的伪码。`KiUserExceptionDispatcher`位于`NTDLL.DLL`中，它是异常发生后执行的起点。这样说也不是百分之百的准确。例如，在`Intel`体系下，异常会使控制转到一个`ring 0`（内核模式）的处理程序。此处理程序由对应此异常的中断描述符表表项所定义。我将跳过所有的内核模式代码并假设发生异常时`CPU`直接执行`KiUserExceptionDispatcher`。

`KiUserExceptionDispatcher`的关键就是对`RtlDispatchException`的调用。这个调用启动了对注册的异常处理程序的查找。如果处理程序处理了异常并继续执行，则对`RtlDispatchException`的调用不再返回。如果`RtlDispatchException`返回了，则有两种可能：要么调用了`NtContinue`使进程继续，要么就是产生了另一个异常。若是后者，异常就不能再继续了，进程必须结束。接着说`RtlDispatchExceptionCode`，这就是遍历异常帧的代码。函数获得一个指向`EXCEPTION_REGISTRATIONs`链表的指针并遍历每一个节点查找处理程序。因为堆栈可能崩溃掉，这个函数非常谨慎。在调用每个`EXCEPTION_REGISTRATION`指定的处理程序之前，代码要保证在线程堆栈中`EXCEPTION_REGISTRATION`是`DWORD`对齐的且前面的`EXCEPTION_REGISTRATION`的地址高。

`RtlDispatchException`并不直接调用`EXCEPTION_REGISTRATION`结构体中指定的地址，而是调用`RtlpExecuteHandlerForException`来做这个脏累活儿。根据`RtlpExecuteHandlerForException`内部发生的情况，`RtlDispatchException`要么继续遍历异常帧要么产生另一个异常。这个二级异常指示异常回调函数中出现问题不能继续执行。`RtlpExecuteHandlerForException`的代码和另一个函数`RtlpExecutehandlerForUnwind`紧密相关。我在前面讲`unwinding`时曾提到这个函数。这两个函数都在将控制送到`ExecuteHandler`函数之前用不同的值加载`EDX`寄存器。换种说法就是`RtlpExecuteHandlerForException`和`RtlpExecutehandlerForUnwind`是同一个`ExecuteHandler`函数的不同的前端。

`ExecuteHandler`就是`EXCEPTION_REGISTRATION`的`handler`域被取出和执行的地方。也许看上去有些奇怪，对异常回调函数的调用本身也被一个结构化异常处理程序封装了起来。在这里使用`SEH`尽管有点儿怪，但认真考虑一下还是合理的。如果异常回调引起了另一个异常，操作系统需要知道此事件。根据异常是发生在初始的回调还是`unwind`中的回调，`ExecuteHandler`返回`DISPOSITION_NESTED_EXCEPTION`或`DISPOSITION_COLLIDED_UNWIND`。这两个可都是“红色警戒！立即关闭！”级别的代号。读者也许像我一样很难让所有的函数都与`SEH`直接关联。类似地，也很难记住谁调用了谁。为了帮助我自己，我画了个图即`Figure 15`。

现在，在执行`ExecuteHandler`前设置`EDX`寄存器干什么呢？其实很简单。若调用用户的处理程序时出错，则不管`EDX`里是什么`ExecuteHandler`都会将其作为纯粹的异常处理程序。它将`EDX`寄存器压栈作为最小`EXCEPTION_REGISTRATION`结构体的`handler`域。本质上讲，`ExecuteHandler`使用的纯粹的异常处理和我在`MYSEH`和`MYSEH2`程序里使用的差不多。

## Conclusion
