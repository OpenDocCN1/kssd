

# 【翻译】结构异常处理之一 异常分发 SEH

> 原文：[`www.pediy.com/kssd/pediy10/64427.html`](https://www.pediy.com/kssd/pediy10/64427.html)

【文章标题】: 异常分发和调试器异常处理  
【文章作者】: kanghtta  
【作者邮箱】: kanghtta@hotmail.com  
【作者主页】: http://kanghtta.cublog.cn  
【下载地址】: 自己搜索下载  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  Exception Dispatching  
  异常分发：  
  当一个硬件或软件的异常发生时，处理机终止异常进程的执行并将控制权转交给操作系统。首先，操作系统将描述异常的信息和当前线程的机器状态保存起来。然后，系统尝试着去找一个异常处理程序来处理这个异常。  
  产生异常的线程的机器状态字被保存在一个 CONTEXT 结构中。如果这个异常被成功处理，那么 CONTEXT 结构（称作环境记录）中的信息就能让操作系统在异常产生的地方继续执行。而描述一个异常的信息（也称异常记录）被保存在 EXCEPTION RECORD 的结构中。由于将与机器相关的环境记录和与机器无关的异常信息分开存储，使得异常处理机制就能够在不同的平台上移植。  
  环境记录和异常记录的信息能够被 GetExceptionInformation 函数使用，也能被任何异常处理程序使用，作为异常处理程序执行后的结果；异常记录包含的信息如下所示：  
  1)异常码，用于确定异常的类型；  
  2) 异常标志位，标示异常事件是否是连续的。任何尝试都将在一个不可开拓的异常后引发一个新的异常。                                        说明：　如在异常标志中包含了 EXCEPTION\_NONCONTINUABLE 标志位时，如果异常处理函数返回的值为 EXCEPTION\_CONTINUE\_EXECUTION，这样，只会引发另一个异常；   
  3)指向另一个异常纪录的指针。如果异常嵌套着产生，那么将产生一异常链接表。（链表）   
  4)异常发生的地址  
  5)一个用于提供异常附加信息的３２位参数的数组。  
  typedef struct \_EXCEPTION\_RECORD { // exr   
      DWORD ExceptionCode;   
      DWORD ExceptionFlags;   
      struct \_EXCEPTION\_RECORD \*ExceptionRecord;   
      PVOID ExceptionAddress;   
      DWORD NumberParameters;   
      DWORD ExceptionInformation\[EXCEPTION\_MAXIMUM\_PARAMETERS\];   
  } EXCEPTION\_RECORD;   
    
  当在用户模式代码中有异常产生时，系统使用下面的搜索顺序来查找一个异常处理函数：  
  1.如果进程正在被调试，系统将通知调试器。更多的信息，请参考 Debugger Exception Handling  
  2.如果进程没有被调试，或者是和它关联的调试器并没有处理异常，操作系统通过搜索产生异常的线程堆栈结构（也称为栈帧）来试图定位一个基于框架的异常处理程序；操作系统首先查找当前堆栈结构，然后逆序把堆栈结构仔细的搜寻一遍；  
  3.如果没有基于框架的处理程序被发现，或者没有基于框架的处理程序处理异常，但是进程正在被调试，那么操作系统将再一次的通知调试器；  
  4.如果进程没有被调试，或者是和它关联的调试器没有处理异常，操作系统就根据异常类型提供默认处理。对于大多数而言，默认的处理是调用 ExitProcess 函数；  
  当一个异常产生在内核模式代码中时，操作系统查找内核堆栈结构试图定位一个异常处理程序。如果异常处理程序没有被定位或是没有异常处理程序处理这个异常，好像 ExitWindows 函数将被调用，操作系统被关闭；  
  Debugger Exception Handling  
  调试器异常处理  
  用户模式异常的处理方式为高级调试器提供了很好的支持。如果产生异常的进程正在被调试，操作系统就产生一调试事件。如果调试器是用 WaitForDeBugEvent 函数，调试事件将导致这个函数返回一指向 DEBUG EVENT 结构的指针。这个结构包含了进程，线程标识符和调试器能访问的环境记录（CONTEXT 结构体）。这个结构还包含一个 EXCEPTION DEBUG INFO 结构，此结构包含一个异常记录（EXCEPTION\_RECORD 结构体）的拷贝。  
  当操作系统找到一个异常处理程序时，它尝试着通知进程的调试器两次。第一次通知会提供给调试器一个处理断点或单步调试异常的机会。这被认为是首次机会通知。在这时，用户能在任何异常处理函数完成之前，通过输入调试命令来巧妙的处理线程环境。第二次通知调试器当且仅当操作系统不能找到基于框架的异常处理函数来处理这个异常时才会发生。如果调试器在最后一次收到通知时都不进行异常处理，系统就终止正在被调试的进程；  
  设置了每一个通知后，调试器使用 ContinueDebugEvent 函数来将控制权返回给系统，在控制权返回给系统之前，调试器能处理异常并修改合适的线程状态，它同样可以选择不处理异常。用 ContinueDebugEvent 函数，调试器能指出它要处理的异常，在这种情况下，机器状态字被恢复，线程继续执行。调试器同样能指出它不处理的异常，在这种情况下，系统将继续为它查找异常处理程序。  
    
  附录：CONTEXT 结构，MSDN 中提示在 winnt。H 文件中定义：  
  typedef struct \_CONTEXT {  
    
      //  
      // The flags values within this flag control the contents of  
      // a CONTEXT record.  
      //  
      // If the context record is used as an input parameter, then  
      // for each portion of the context record controlled by a flag  
      // whose value is set, it is assumed that that portion of the  
      // context record contains valid context. If the context record  
      // is being used to modify a threads context, then only that  
      // portion of the threads context will be modified.  
      //  
      // If the context record is used as an IN OUT parameter to capture  
      // the context of a thread, then only those portions of the thread's  
      // context corresponding to set flags will be returned.  
      //  
      // The context record is never used as an OUT only parameter.  
      //  
    
      DWORD ContextFlags;  
    
      //  
      // This section is specified/returned if CONTEXT\_DEBUG\_REGISTERS is  
      // set in ContextFlags.  Note that CONTEXT\_DEBUG\_REGISTERS is NOT  
      // included in CONTEXT\_FULL.  
      //  
    
      DWORD   Dr0;  
      DWORD   Dr1;  
      DWORD   Dr2;  
      DWORD   Dr3;  
      DWORD   Dr6;  
      DWORD   Dr7;  
    
      //  
      // This section is specified/returned if the  
      // ContextFlags word contians the flag CONTEXT\_FLOATING\_POINT.  
      //  
    
      FLOATING\_SAVE\_AREA FloatSave;  
    
      //  
      // This section is specified/returned if the  
      // ContextFlags word contians the flag CONTEXT\_SEGMENTS.  
      //  
    
      DWORD   SegGs;  
      DWORD   SegFs;  
      DWORD   SegEs;  
      DWORD   SegDs;  
    
      //  
      // This section is specified/returned if the  
      // ContextFlags word contians the flag CONTEXT\_INTEGER.  
      //  
    
      DWORD   Edi;  
      DWORD   Esi;  
      DWORD   Ebx;  
      DWORD   Edx;  
      DWORD   Ecx;  
      DWORD   Eax;  
    
      //  
      // This section is specified/returned if the  
      // ContextFlags word contians the flag CONTEXT\_CONTROL.  
      //  
    
      DWORD   Ebp;  
      DWORD   Eip;  
      DWORD   SegCs;              // MUST BE SANITIZED  
      DWORD   EFlags;             // MUST BE SANITIZED  
      DWORD   Esp;  
      DWORD   SegSs;  
    
      //  
      // This section is specified/returned if the ContextFlags word  
      // contains the flag CONTEXT\_EXTENDED\_REGISTERS.  
      // The format and contexts are processor specific  
      //  
    
      BYTE    ExtendedRegisters\[MAXIMUM\_SUPPORTED\_EXTENSION\];  
    
  } CONTEXT;  
    
    
    
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2008 年 05 月 07 日 PM 04:14:07