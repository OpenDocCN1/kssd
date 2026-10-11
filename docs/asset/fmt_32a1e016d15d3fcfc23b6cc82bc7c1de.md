
这个函数代码不长。主要分为两个大分支，一个分支处理异常，一个分支展开（参考地址`f87203dc`处的`test`指令）。

处理异常的代码负责遍历`scopetable`，依次调用`scopetable_entry::lpfnFilter`（参考`f8720419`处代码)，并针对不同的返回值做出不同的处理：

1.  返回`EXCEPTION_CONTINUE_EXECUTION`，则说明异常已经被刚刚调用的`lpfnFilter`修复。返回`ExceptionContinueExecution`。
2.  返回`EXCEPTION_CONTINUE_SEARCH`，则继续遍历下一个`scopetable_entry`。
3.  返回`EXCEPTION_EXECUTE_HANDLER`，则说明当前`scopetable_entry::lpfnHandler`负责处理该异常。于是调用它。

对于展开的代码，则直接开始局部展开，即对`scopetable`进行展开。

更具体的信息，请参考上面反汇编代码中我附的注释。

有几个小点需要说一下：

1.  该函数多处使用`scopetable`中坑内的数据进行安全检查。这些操作对理解该函数流程没有帮助，可以忽略。如果觉得上面代码不纯净，可以参考我后续的附录 1《Ntfs!_except_handler3 的反汇编代码》，这个函数流程更清晰简单。
2.  一旦有某`scopetable_entry::lpfnFilter`返回`EXCEPTION_EXECUTE_HANDLER`，就会进行全局展开和局部展开。展开结束后会调用该`scopetable_entry::lpfnHandler`，该函数即为`_except`处理域，该函数形式是一个函数，实际上只是一段不返回的代码。即这段代码中没有`ret`指令。执行完整个`_except`处理域后，会接着执行其后的指令，并不会返回`_except_handler4`。
3.  该函数既启动展开，又负责展开，于是会出现类似于“重入”的现象。理解的过程中容易扰乱思路。

`PassThrough!_except_handler4`在执行过程中可能会调用这几个函数：

`PassThrough!_EH4_CallFilterFunc`、

`PassThrough!_EH4_TransferToHandler`、

`PassThrough!_EH4_GlobalUnwind`、

`PassThrough!_EH4_LocalUnwind`

这几个函数名很明白的说明了它们的功能：

`PassThrough!_EH4_CallFilterFunc` 负责调用 `scopetable_entry::lpfnFilter`；

`PassThrough!_EH4_TransferToHandler` 负责调用 `scopetable_entry::lpfnFilter`；

`PassThrough!_EH4_GlobalUnwind` 负责全局展开；

`PassThrough!_EH4_LocalUnwind` 负责局部展开。

来看看`_EH4_CallFilterFunc`和`_EH4_TransferToHandler`的反汇编代码，都很短。剩余两个展开相关的函数稍后咱们再来分析。

```
    kd> uf PassThrough!_EH4_CallFilterFunc
        PassThrough!_EH4_CallFilterFunc [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 408]:
          f87205d1  push    ebp
          f87205d2  push    esi
          f87205d3  push    edi
          f87205d4  push    ebx
          f87205d5  mov     ebp,edx
          f87205d7  xor     eax,eax
          f87205d9  xor     ebx,ebx
          f87205db  xor     edx,edx
          f87205dd  xor     esi,esi
          f87205df  xor     edi,edi
          f87205e1  call    ecx ; lpfnFilter();
          f87205e3  pop     ebx
          f87205e4  pop     edi
          f87205e5  pop     esi
          f87205e6  pop     ebp
          f87205e7  ret

    kd> uf PassThrough!_EH4_TransferToHandler
        PassThrough!_EH4_TransferToHandler [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 450]:
          f87205e8  mov     ebp,edx
          f87205ea  mov     esi,ecx ; esi = lpfnHandler
          f87205ec  mov     eax,ecx
          f87205ee  xor     eax,eax
          f87205f0  xor     ebx,ebx
          f87205f2  xor     ecx,ecx
          f87205f4  xor     edx,edx
          f87205f6  xor     edi,edi
          f87205f8  jmp     esi     ; jmp lpfnHandler
```

到这里，咱们就以`PassThrough!_except_handler4`为例分析完了`MSC`提供的`EXCEPTION_REGISTRATION::handler`函数。这个过程中多次接触到一个名为“展开”的概念，这就是咱们要讲的第三个部分。

### 三、展开（unwind）

为了说明这个概念，需要先回顾下异常发生后的处理流程。

我们假设一系列使用`SEH`的函数调用流程：

`func1` -> `func2` -> `func3`。在`func3`执行的过程中触发了异常。

看看分发异常流程`RtlRaiseException` -> `RtlDispatchException` -> `RtlpExecuteHandlerForException`

`RtlDispatchException`会遍历异常链表，对每个`EXCEPTION_REGISTRATION`都调用`RtlpExecuteHandlerForException`。

`RtlpExecuteHandlerForException`会调用`EXCEPTION_REGISTRATION::handler`，也就是`PassThrough!_except_handler4`。如咱们上面分析，该函数内部遍历`EXCEPTION_REGISTRATION::scopetable`，如果遇到有`scopetable_entry::lpfnFilter`返回`EXCEPTION_EXECUTE_HANDLER`，那么`scopetable_entry::lpfnHandler`就会被调用，来处理该异常。

因为`lpfnHandler`不会返回到`PassThrough!_except_handler4`，于是执行完`lpfnHandler`后，就会从`lpfnHandler`之后的代码继续执行下去。也就是说，假设`func3`中触发了一个异常，该异常被`func1`中的`__except`处理块处理了，那`__except`处理块执行完毕后，就从其后的指令继续执行下去，即异常处理完毕后，接着执行的就是`func1`的代码。不会再回到`func2`或者`func3`，这样就有个问题，`func2`和`func3`中占用的资源怎么办？这些资源比如申请的内存是不会自动释放的，岂不是会有资源泄漏问题？

这就需要用到“展开”了。

说白了，所谓“展开”就是进行清理。（注：这里的清理主要包含动态分配的资源的清理，栈空间是由`func1`的“`mov esp,ebp`”这类操作顺手清理的。当时我被“谁来清理栈空间”这个问题困扰了很久……）

那这个展开工作由谁来完成呢？由`func1`来完成肯定不合适，毕竟`func2`和`func3`有没有申请资源、申请了哪些资源，`func1`无从得知。于是这个展开工作还得要交给`func2`和`func3`自己来完成。

展开分为两种：“全局展开”和“局部展开”。

全局展开是指针对异常链表中的某一段，局部展开针对指定`EXCEPTION_REGISTRATION`。用上面的例子来讲，局部展开就是针对`func3`或`func2`（某一个函数）内部进行清理，全局展开就是`func2`和`func3`的局部清理的总和。再归纳一下，局部展开是指具体某一函数内部的清理，而全局展开是指，从异常触发点（`func3`）到异常处理点（`func1`）之间所有函数（包含异常触发点`func3`）的局部清理的总和。

来看反汇编代码：

```
    kd> uf PassThrough!_EH4_GlobalUnwind
        PassThrough!_EH4_GlobalUnwind [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 485]:
          f87205fa   push    ebp
          f87205fb   mov     ebp,esp
          f87205fd   push    ebx
          f87205fe   push    esi
          f87205ff   push    edi
          f8720600   push    0   ; pReturnValue
          f8720602   push    0   ; pExceptionRecord
          f8720604   push    offset PassThrough!_EH4_GlobalUnwind+0x15 (f872060f) ; pReturnEip
          f8720609   push    ecx ; pExceptionRegistration
          f872060a   call    PassThrough!RtlUnwind (f8720678)
          f872060f   pop     edi
          f8720610   pop     esi
          f8720611   pop     ebx
          f8720612   pop     ebp
          f8720613   ret
```

`RtlUnwind`的原型：

```
        VOID RtlUnwind(
            PEXCEPTION_REGISTRATION pExceptionRegistration
            PVOID pReturnEip
            PEXCEPTION_RECORD pExceptionRecord,
            PVOID pReturnValue
        );
```
