
```
VOIDNTAPI

KiInitializeUserApc(IN PKEXCEPTION_FRAME ExceptionFrame,                    IN PKTRAP_FRAME TrapFrame,                    IN PKNORMAL_ROUTINE NormalRoutine,                    IN PVOID NormalContext,                    IN PVOID SystemArgument1,                    IN PVOID SystemArgument2)

{    //V86模式下，不投递



     /* Save the full context */

    Context.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS;

    KeTrapFrameToContext(TrapFrame, ExceptionFrame, &Context);    

    //检查不是KernleMode

    ASSERT((TrapFrame->SegCs & MODE_MASK) != KernelMode);    

    ...

    

    /* Get the aligned size */

    AlignedEsp = Context.Esp & ~3;//来自于TrapFrame.HardwareEsp或TempEsp

    //Context和4个参数的长度

    ContextLength = CONTEXT_ALIGNED_SIZE + (4 * sizeof(ULONG_PTR));    //将原始堆栈扩展ContextLength，用来保存Context和参数

    Stack = ((AlignedEsp - 8) & ~3) - ContextLength;    /* Probe the stack */

    ProbeForWrite((PVOID)Stack, AlignedEsp - Stack, 1);

    ASSERT(!(Stack & 3));    /* Copy data into it */

    //(4 * sizeof(ULONG_PTR)))是后面4个参数的位置，然后接着拷贝Context，将老的TrapFrame内容拷贝到用户太堆栈中

    RtlCopyMemory((PVOID)(Stack + (4 * sizeof(ULONG_PTR))),                  &Context,

                  sizeof(CONTEXT));    /* Run at APC dispatcher */

    TrapFrame->Eip = (ULONG)KeUserApcDispatcher; //KeUserApcDispatcher保存的其实就是KiUserApcDispatcher，是用户空间函数

    TrapFrame->HardwareEsp = Stack;//栈顶



    /* Setup Ring 3 state */

    TrapFrame->SegCs = Ke386SanitizeSeg(KGDT_R3_CODE, UserMode);

    TrapFrame->HardwareSegSs = Ke386SanitizeSeg(KGDT_R3_DATA, UserMode);

    TrapFrame->SegDs = Ke386SanitizeSeg(KGDT_R3_DATA, UserMode);

    TrapFrame->SegEs = Ke386SanitizeSeg(KGDT_R3_DATA, UserMode);

    TrapFrame->SegFs = Ke386SanitizeSeg(KGDT_R3_TEB, UserMode);

    TrapFrame->SegGs = 0;

    TrapFrame->ErrCode = 0;    /* Sanitize EFLAGS */

    TrapFrame->EFlags = Ke386SanitizeFlags(Context.EFlags, UserMode);    /* Check if thread has IOPL and force it enabled if so */

    if (KeGetCurrentThread()->Iopl) TrapFrame->EFlags |= 0x3000;    /* Setup the stack */

    *(PULONG_PTR)(Stack + 0 * sizeof(ULONG_PTR)) = (ULONG_PTR)NormalRoutine;

    *(PULONG_PTR)(Stack + 1 * sizeof(ULONG_PTR)) = (ULONG_PTR)NormalContext;

    *(PULONG_PTR)(Stack + 2 * sizeof(ULONG_PTR)) = (ULONG_PTR)SystemArgument1;

    *(PULONG_PTR)(Stack + 3 * sizeof(ULONG_PTR)) = (ULONG_PTR)SystemArgument2;    ...

    }
```

执行流程根据注释应该很清楚了，这里要解释一下`TrapFrame`。

> CPU进入内核之后，内核堆栈就会有个`TrapFrame`，保存的是用户空间的线程（因进入内核原因不同，可能是自陷、中断、异常框架，都是一样的结构）。CPU返回用户空间时会使用这个`TrapFrame`，才能正确返回原来的断点，并回复寄存器的状态 这里为了让Apc返回到用户空间执行，就会修改这个`TrapFrame`，原来的`TrapFrame`就需要保存，这里保存在了用户空间堆栈中（`CONTEXT`) 执行完Apc函数之后，执行一个`NtContinue`，将这个`CONTEXT`作为参数，这样保存的`TrapFrame`就会还原到原来的状态，然后CPU又能正常回之前的用户空间了。

`KiDeliverApc`完了之后，回到`_KiServiceExit`，会使用被修改过的`TrapFrame`回到用户空间，执行指定的`KiUserApcDispatcher`(`ntdll`提供)

```
//根据这个执行KiUserApcDispatcher

TrapFrame->Eip = (ULONG)KeUserApcDispatcher; //其实就是KiUserApcDispatcher，是用户空间函数

TrapFrame->HardwareEsp = Stack;//栈顶



.func KiUserApcDispatcher@16.globl _KiUserApcDispatcher@16_KiUserApcDispatcher@16:



    /* Setup SEH stack */    lea eax, [esp+CONTEXT_ALIGNED_SIZE+16];原始堆栈的位置，SEH

    mov ecx, fs:[TEB_EXCEPTION_LIST]    mov edx, offset _KiUserApcExceptionHandler    mov [eax], ecx

    mov [eax+4], edx



    /* Enable SEH */    mov fs:[TEB_EXCEPTION_LIST], eax



    /* Put the Context in EDI */    pop eax;弹出第一个参数

    lea edi, [esp+12];context的位置



    /* Call the APC Routine */    call eax //调用IntCallUserApc



    /* Restore exception list */    mov ecx, [edi+CONTEXT_ALIGNED_SIZE]    mov fs:[TEB_EXCEPTION_LIST], ecx



    /* Switch back to the context */    push 1

    push edi;Context

    call _ZwContinue@8 //正常是不会返回的



    /* Save callback return value */    mov esi, eax



    /* Raise status */StatusRaiseApc:

    push esi

    call _RtlRaiseStatus@4 //如果ZwContinue失败了，这里处理    jmp StatusRaiseApc    ret 16.endfunc
```

`KiUserApcDispatcher`其实挺简单的，通过esp弹出APc函数，然后调用，就进入了`IntCallUserApc`，

## 恢复`TrapFrame`

执行完成后，调用`_ZwContinue`(`Context`, 1)，回到内核回复之前修改`TrapFrame`，也会重新检查是否有Apc需要投递，有则继续投递，重复上面的步骤，直到没有了则可以回到之前被中断的用户态的断点处。

```
.func NtContinue@8_NtContinue@8:



    /* NOTE: We -must- be called by Zw* to have the right frame! */

    /* Push the stack frame */    push ebp ; 指向本次调用的自陷框架，记为T1



    /* Get the current thread and restore its trap frame */    mov ebx, PCR[KPCR_CURRENT_THREAD]    mov edx, [ebp+KTRAP_FRAME_EDX]    mov [ebx+KTHREAD_TRAP_FRAME], edx;thread->TrapFrame = edx



    /* Set up stack frame */    mov ebp, esp ; ESP指向新的框架（函数调用框架）



    /* Save the parameters */    mov eax, [ebp+0] ; 原来的EBP，就是自陷框架指针，就是T1

    mov ecx, [ebp+8] ; Context



    /* Call KiContinue */    push eax ;TrapFrame

    push 0 ;ExceptionFrame

    push ecx ;Context

    call _KiContinue@12 ; 将Context恢复到T1中



    /* Check if we failed (bad context record) */    or eax, eax

    jnz Error



    /* Check if test alert was requested */    cmp dword ptr [ebp+12], 0

    je DontTest



    /* Test alert for the thread */    mov al, [ebx+KTHREAD_PREVIOUS_MODE]    push eax

    call _KeTestAlertThread@4 ; 检查用户模式APC队列是否为空，不空将UserApcPending置为TRUEDontTest:

    /* Return to previous context */    pop ebp

    mov esp, ebp

    jmp _KiServiceExit2 ; 本质和_KiServiceExit相同，如果还有用户APC，会继续投递，直到投递完，才会回到用户被中断的点Error:

    pop ebp

    mov esp, ebp

    jmp _KiServiceExit.endfunc
```

下面将`_KiServiceExit`到`IntCallUserApc`的流程总结一下： ![img]()

[![](https://bbs.kanxue.com/upload/attach/202002/394321_ab8rd3405u61o5i.png)](https://bbs.kanxue.com/elink@31aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1K9r3E0Y4k6#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6$3W2F1i4K6u0V1j5i4m8U0i4K6u0V1j5h3&6S2L8s2W2*7k6e0q4Q4x3V1j5`.)

到这里，终于执行到了用户的Apc函数。

## 结账走人

到这，APC流程基本弄清楚了。

下一篇将结合APC机制分析一下最近比较新的AtomBombing注入技术的详细实现和各个细节。

## 参考

1. Reactos内核情景源码分析

2. [线程的Alertable与User APC](http://bbs.pediy.com/thread-209377.htm)

如果大家觉得还不错，欢迎关注我的博客：[dbcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1K9r3E0Y4k6#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6$3W2F1i4K6u0V1j5i4m8U0i4K6u0V1j5h3&6S2L8s2W2\*7k6e0q4Q4x3V1j5`.](https://bbs.kanxue.com/elink@a3eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1K9r3E0Y4k6#2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3N6$3W2F1i4K6u0V1j5i4m8U0i4K6u0V1j5h3&6S2L8s2W2*7k6e0q4Q4x3V1j5`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
