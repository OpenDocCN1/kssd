

# 关于异常处理

> 原文：[`www.pediy.com/kssd/pediy11/122867.html`](https://www.pediy.com/kssd/pediy11/122867.html)

关于异常处理， Exception Handling，写了点总结。  
  
VC6 begin ------------  
  
如果 VC6 不带编译选项 /GX, 代码中便不支持 try catch 否则会报错：  
  warning C4530: C++ exception handler used, but unwind semantics are not enabled. Specify -GX   
  
在 VC6 中，编译选项 /GX 可以通过   
  Project Settings | C/C++ | C++ Language | Enable exception handling  
来控制是否使用。  
  
编译选项　/EHsc 与 /GX 等价。  
  
VC6 新建工程默认是带 /GX 的。但新建 ATL COM AppWizard 工程默认是不带 /GX 的。  
  
检查带 try catch 的 C++ 代码生成的汇编，发现它是用以下框架实现的：  
                push    -1  
                push    offset \_\_ehhandler$?test@@YAXXZ  
                mov     eax, large fs:0  
                push    eax  
                mov     large fs:0, esp  
                ....  
\_\_ehhandler$?test@@YAXXZ:  
                mov     eax, offset stru\_426490  ; stru\_426490 is struct \_msExcept  
                jmp     \_\_\_CxxFrameHandler  
  
\_msExcept       struc ; (sizeof=0x20, variable size)  
Magic           dd ?                    ; base 16  
Count           dd ?                    ; base 10  
InfoPtr         dd ?                    ; offset  
CountDtr        dd ?                    ; base 10  
DtrPtr          dd ?                    ; offset  
\_unk            dd 3 dup(?)  
Info            \_msExcInfo 0 dup(?)  
\_msExcept       ends  
  
VC6 也可以用 \_\_try \_\_except 它是这样实现的：  
                push    -1  
                push    offset \_\_sehtable$?test@@YAXXZ  
                push    offset \_\_except\_handler3  
                mov     eax, dword ptr fs:0  
                push    eax  
                mov     dword ptr fs:0, esp  
                ...  
  
\_\_sehtable$?test@@YAXXZ \_msEH <0FFFFFFFFh, offset $L600, offset $L601>  
  
\_msEH           struc ; (sizeof=0xC)  
\_unk            dd ?                    ; base 16  
FilterProc      dd ?                    ; offset  
ExitProc        dd ?                    ; offset  
\_msEH           ends  
  
这种\_\_try \_\_except 不需要 /GX 可以直接支持  
  
总之，无论是 try catch 还是 \_\_try \_\_except 都是通过 X86 CPU 提供的 FS:0 来实现的，可以认识，它实现上是一回事。就是说，在汇编一级，它们是同理的。不过是 FS:0 对应的实现函数有不同的写法。如果发现 \_\_except\_handler3 那是 \_\_try \_\_except.如果发现  
\_\_\_CxxFrameHandler, 那是 try catch.  
VC6 end ------------  
  
VS 2008 begin -----------  
换 Visual Studio 2008 测试，  
  Visual Studio 2008 Professional Edition  
  Version 9.0.21022.8 RTM  
新建一个 Win32 Console 工程，测试可以直接使用 try catch 在编译选项中已经包含了 /EHsc  
  
打开工程属性，  
  C/C++ | Code Generation | Enable C++ Exceptions  
对应的选择  
  No  
  Yes /EHsc  
  Yes with SEH Exceptions /EHa  
  
网上搜一下，有说法：/EHa 的意思是 asynchronous model, /EHsc 的意思是 synchronous  
  
如果选 No 代码中便不支持 try catch 否则会报错：  
  warning C4530: C++ exception handler used, but unwind semantics are not enabled. Specify /EHsc   
  
测试 try catch 用 release 编译　/EHsc 或 /EHa 选项，结果类似 VC6 ，用到了   
  jmp \_\_CxxFrameHandler3  ; export from MSVCR90.dll  
  
  
  
VS 2008 end -----------  
  
最后总结  
  
使用 try catch 技术，叫做 C++ exception handling  
使用 \_\_try \_\_except 技术，叫做 structured exception handling(简称 SEH)  
另外还有 MFC exceptions 我想应该是 MFC 所做的扩展。  
  
try throw catch 是 C++标准关键字。而 \_\_try \_\_exception \_\_finally 是 Microsoft 扩展。  
  
  
无论是 C++ exception handling 还是 structured exception handling ，都是通过 X86 CPU 提供的 FS:0 来实现的，可以认为，它们实现上是一回事。可以用 \_set\_se\_handler 来实现自己的处理程序。  
  
网上找到一句话：  
As stated by MSDN, for "C++ programs, C++ exception handling should be used instead of structured exception handling"   
  
就是说，Microsoft 推荐我们用 try catch 尽量不要用 \_\_try \_\_except.  
  
Microsoft 很 BT 地推荐我们用 C 而不是 C++ 写驱动，所以我们在驱动代码中大量看到 \_\_try \_\_except 而看不到 try catch. 实现上，用 C++ 写驱动是可以的，在驱动中用 try catch 也是可以的。  
  
C 也支持 try catch ，这时应该叫　C exception handling 了。  
  
C++ exception handling 使用简单说明  
  
C++ exception handling 可以包含 try throw catch，可以有多个 catch，每个 catch 处理不同的数据类型，用 catch(...) 处理所有类型。  
  
在函数定义后面可以加 throw 来表示这个函数可能会抛出异常。  
  void funcname(double) throw(string, double, bool);  
  
如果 #include <exception> 可以使用 class std::exception.  
  
structured exception handling(简称 SEH)　使用简单说明  
SEH 可以包含   
  \_\_try  
  \_\_except  
  \_\_finally  
  \_\_leave  
提供了  
  EXCEPTION\_CONTINUE\_EXECUTION (1)   
  EXCEPTION\_CONTINUE\_SEARCH (0)  
  EXCEPTION\_EXECUTE\_HANDLER (1)   
甚至可以从出问题的地方继续执行。  
  
提供了  
  GetExceptionCode()  
  GetExceptionInformation()  
函数。  
  
\_\_try \_\_leave \_\_finally 实际上是 Microsft 利用 fs:0 技术所做的一个 C++扩展，它与异常无关，方便编程。