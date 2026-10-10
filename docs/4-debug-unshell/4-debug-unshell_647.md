

# EXECryptor v1.53

> 原文：[`www.pediy.com/kssd/pediy06/pediy6442.htm`](https://www.pediy.com/kssd/pediy06/pediy6442.htm)

如何用 OllyDBG 跟踪 EXECryptor v1.53？    
  
    EXECryptor v1.53 是目前利用 SEH 反跟踪最完美的一个壳，在 SEH 处理中对 DR0-DR7 等调试寄存器进行清零并修改 EIP 寄存器来实现跳转。如果对 SEH 缺乏完全的了解，跟踪起来简直是一头雾水。    
一、SEH 复习   
1、建立 SEH 的框架   
   ASSUME FS:NOTHING        ;否则 Masm 编译报错    
        push    offset SEH\_Handler   
        push    fs:\[0\]          
        mov     fs:\[0\],esp               
  
2、传递给异常处理例程的参数   
    传递给 final 型的参数,只有一个即指向 EXCEPTION\_POINTERS 结构的指针, EXCEPTION\_POINTERS 定义如下:   
       EXCEPTION\_POINTERS STRUCT   
        pExceptionRecord  DWORD      ?                  
       ContextRecord     DWORD      ?   
      EXCEPTION\_POINTERS ENDS   
       执行时堆栈结构如下:   
                 esp    -> ptEXCEPTION\_POINTERS    
                            然后执行 call \_Final\_Handler   
  
注意堆栈中的参数是指向 EXCEPTION\_POINTERS 的指针,而不是指向 pExceptionRecord 的指针   
以下是 EXCEPTION\_POINTERS 两个成员的详细结构   
              EXCEPTION\_RECORD STRUCT   
          ExceptionCode         DWORD      ?       ;异常码   
          ExceptionFlags        DWORD      ?       ;异常标志   
          PExceptionRecord     DWORD      ?      ;指向另外一个 EXCEPTION\_RECORD 的指针   
          ExceptionAddress      DWORD      ?       ;异常发生的地址   
          NumberParameters      DWORD      ?       ;下面 ExceptionInformation 所含有的 dword 数目   
          ExceptionInformation  DWORD EXCEPTION\_MAXIMUM\_PARAMETERS dup(?)   
        EXCEPTION\_RECORD ENDS                         
;EXCEPTION\_MAXIMUM\_PARAMETERS ==15   
  
具体参数解释:   
ExceptionCode 异常类型,SDK 里面有很多类型,但你最可能遇到的几种类型如下:   
              C0000005h----读写内存冲突   
              C0000094h----非法除 0   
              C00000FDh----堆栈溢出或者说越界   
              80000001h----由 Virtual Alloc 建立起来的属性页冲突   
              C0000025h----不可持续异常,程序无法恢复执行,异常处理例程不应处理这个异常   
              C0000026h----在异常处理过程中系统使用的代码,如果系统从某个例程莫名奇妙的返回,则出现此代码,例如调用 RtlUnwind 时没有 Exception Record 参数时产生的异常填入的就是这个代码   
              80000003h----调试时因代码中 int3 中断   
              80000004h----处于被单步调试状态   
              注:也可以自己定义异常代码,遵循如下规则:   
              
      位:       31~30            29~28           27~16          15~0   
    含义:     严重程度          29 位            功能代码        异常代码   
              0==成功         0==Mcrosoft     MICROSOFT 定义   用户定义   
              1==通知         1==客户   
              2==警告          28 位   
              3==错误         被保留必须为 0   
ExceptionFlags 异常标志   
              0----可修复异常   
              1----不可修复异常   
              2----正在展开,不要试图修复什么,需要的话,释放必要的资源   
pExceptionRecord 如果程序本身导致异常,指向那个异常结构   
ExceptionAddress 发生异常的 eip 地址   
ExceptionInformation 附加消息,在调用 RaiseException 可指定或者在异常号为 C0000005h 即内存异常时(ExceptionCode=C0000005h) 的含义如下,其他情况下一般没有意义   
第一个 dword 0==读冲突 1==写冲突   
第二个 dword 读写冲突地址   
CONTEXT 具体结构含义:   
  
              CONTEXT STRUCT                     ; \_                     
          ContextFlags  DWORD      ?       ;  |---------------  +00   
          iDr0          DWORD      ?       ;  |                 +04   
          iDr1          DWORD      ?       ;  |                 +08   
          iDr2          DWORD      ?       ;   >调试寄存器       +0C   
          iDr3          DWORD      ?       ;  |                 +10   
          iDr6          DWORD      ?       ;  |                 +14   
          iDr7          DWORD      ?       ; \_|                 +18   
          FloatSave     FLOATING\_SAVE\_AREA <>  ;浮点寄存器区      +1C~~+88   
          regGs         DWORD      ?       ;--|                 +8C   
          regFs         DWORD      ?       ;  |段寄存器         +90           
          regEs         DWORD      ?       ;  |/                +94   
          regDs         DWORD      ?       ;--|                 +98   
          regEdi        DWORD      ?       ;\_\_\_\_\_\_\_\_\_\_\_\_        +9C    
          regEsi        DWORD      ?       ;       |  通用      +A0      
          regEbx        DWORD      ?       ;       |   寄       +A4   
          regEdx        DWORD      ?       ;       |   存       +A8   
          regEcx        DWORD      ?       ;       |   器       +AC   
          regEax        DWORD      ?       ;\_\_\_\_\_\_\_|\_\_\_ 组\_      +B0       
          regEbp        DWORD      ?       ;++++++++++++++++    +B4   
          regEip        DWORD      ?       ;    |控制           +B8   
          regCs         DWORD      ?       ;    |寄存           +BC   
          regFlag       DWORD      ?       ;    |器组            +C0   
          regEsp        DWORD      ?       ;    |               +C4     
          regSs         DWORD      ?       ;+++++++++++++++++   +C8   
          ExtendedRegisters db MAXIMUM\_SUPPORTED\_EXTENSION dup(?)   
        CONTEXT ENDS   
二、EXECryptor v1.53 跟踪要点   
1、密切注意建立 SEH 框架结构，并随时在命令窗口用 D fs:\[0\]察看，其中 fs:\[4\]就是 SEH\_handler.然后到该地址按 F2 设断点。   
2、进入 SEH 处理后，注意修改 EIP 寄存器指令。EXECryptor v1.53 典型修改指令为 mov \[ECX+B8\],XXXXXX 或 add \[ecx+b8\],0b 等。并在所修改的 EIP 处设断点(按 F2)   
3、跟到 Lock cmpxchg8b eax（非法指令）时，修改指令为 int3(0CC),这是全程跟踪的关键点，直到没有执行到该地址指令后要恢复原来的值。这段代码主要是搜索 kernel32.dll 得到函数入口地址，这部分函数为：getmodulehandle;getprocaddress;loadlibrarya;freelibrary 等。   
4、防止进入单步 SEH 陷阱。   
5、EXECryptor v1.53 典型 SEH 结构分析   
00401000 >call EXECrypt.00401029   
00401005  mov ecx,dword ptr ss:\[esp+C\]  //ContextRecord 地址   
00401009  mov dword ptr ds:\[ecx\],10017   
0040100F  mov dword ptr ds:\[ecx+B8\],EXECrypt.004FC000 //改变 EIP 为 4FC000   
00401019  xor eax,eax   
0040101B  mov dword ptr ds:\[ecx+14\],eax  //Dr6=0   
0040101E  mov dword ptr ds:\[ecx+18\],eax  //Dr7=0   
00401021  and byte ptr ds:\[ecx+C1\],0FE   //修改标志寄存器的 T 位。   
00401028  retn   
00401029  xor eax,eax   
0040102B  push dword ptr fs:\[eax\]   
0040102E  mov dword ptr fs:\[eax\],esp  //建立 SEH 框架   
00401031  int3     //异常转移到 00401005   
00401032  retn 

举例：   
00401000 >call EXECrypt.00401029    
00401005  mov ecx,dword ptr ss:\[esp+C\]  //ContextRecord 地址    
00401009  mov dword ptr ds:\[ecx\],10017    
0040100F  mov dword ptr ds:\[ecx+B8\],EXECrypt.004FC000 //改变 EIP 为 4FC000    
00401019  xor eax,eax    
0040101B  mov dword ptr ds:\[ecx+14\],eax  //Dr6=0    
0040101E  mov dword ptr ds:\[ecx+18\],eax  //Dr7=0    
00401021  and byte ptr ds:\[ecx+C1\],0FE   //修改标志寄存器的 T 位。    
00401028  retn    
00401029  xor eax,eax    
0040102B  push dword ptr fs:\[eax\]    
0040102E  mov dword ptr fs:\[eax\],esp  //建立 SEH 框架    
00401031  int3     //异常转移到 00401005    
00401032  retn   
在 00401005 处 F2 设断点,f9 运行，断下后察看堆栈，就知道了。   
EXCEPTION\_POINTERS STRUCT    
        pExceptionRecord  DWORD      ?                   
       ContextRecord     DWORD      ?    
      EXCEPTION\_POINTERS ENDS  
    
 4dd466d0e1e0cc123afbe504da2462ca