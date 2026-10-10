

# SEH in ASM 研究

> 原文：[`www.pediy.com/kssd/pediy04/kanxue310.htm`](https://www.pediy.com/kssd/pediy04/kanxue310.htm)

SEH  in ASM 研究(一)   
                                            By Hume/冷雨飘心   
  
为什么老调重弹:   
    SEH 出现已绝非一日,但很多人可能还不彻底了解 Seh 的运行机制;有关 seh 的知识资料不是很多,asm 级的详细资料就   
更少!seh 不仅可以简化程序错误处理,使你的程序更加健壮,还被广泛应用于反跟踪以及加解密中,因此,了解 seh 非常必要,   
但遗憾的是关于 seh 详细介绍的中文资料非常少,在实践的基础上,把自己学习的一点笔记奉献给大家,希望对喜欢 ASM 的朋   
友有所帮助.如有错误,请高手不吝指正.   
  
                                    PART  I  简单接触   
  
一、SEH 背景知识   
SEH("Structured Exception Handling"),即结构化异常处理.是操作系统提供给程序设计者的强有力的处理程序错   
误或异常的武器.在 VISUAL C++中你或许已经熟悉了\_try{} \_finally{} 和\_try{} \_except {} 结构,这些并不是   
编译程序本身所固有的,本质上只不过是对 windows 内在提供的结构化异常处理的包装,不用这些高级语言编译器所提供   
的包装 ,照样可以利用系统提供的强大 seh 处理功能,在后面你将可以看到,用系统本身提供 seh 结构和规则以及 ASM 语言,   
我们将对 SEH 的机制以及实现有一个彻底的了解.   
  
发生异常时系统的处理顺序(by Jeremy Gordon):   
  
    1.系统首先判断异常是否应发送给目标程序的异常处理例程,如果决定应该发送,并且目标程序正在被调试,则系统   
    挂起程序并向调试器发送 EXCEPTION\_DEBUG\_EVENT 消息.呵呵,这不是正好可以用来探测调试器的存在吗?   
  
    2.如果你的程序没有被调试或者调试器未能处理异常,系统就会继续查找你是否安装了线程相关的异常处理例程,如果   
    你安装了线程相关的异常处理例程,系统就把异常发送给你的程序 seh 处理例程,交由其处理.   
  
    3.每个线程相关的异常处理例程可以处理或者不处理这个异常,如果他不处理并且安装了多个线程相关的异常处理例程,   
        可交由链起来的其他例程处理.   
  
    4.如果这些例程均选择不处理异常,如果程序处于被调试状态,操作系统仍会再次挂起程序通知 debugger.   
  
    5.如果程序未处于被调试状态或者 debugger 没有能够处理,并且你调用 SetUnhandledExceptionFilter 安装了最后异   
    常处理例程的话,系统转向对它的调用.   
  
    6.如果你没有安装最后异常处理例程或者他没有处理这个异常,系统会调用默认的系统处理程序,通常显示一个对话框,   
    你可以选择关闭或者最后将其附加到调试器上的调试按钮.如果没有调试器能被附加于其上或者调试器也处理不了,系统   
    就调用 ExitProcess 终结程序.   
  
    7.不过在终结之前,系统仍然对发生异常的线程异常处理句柄来一次展开,这是线程异常处理例程最后清理的机会.   
  
如果你看了上面的步骤一头雾水的话,别着急,化点时间慢慢理解或者进入下一部分实例操作.   
  
二.初步实战演习:   
  
  安装异常处理句柄.   
    
  有两种类型的异常处理句柄,一种是 final 型的,这是在你的异常未能得到线程相关处理例程处理操作系统在即将关闭程序之前会   
  回调的例程,这个例程是进程相关的而不是线程相关的,因此无论是哪个线程发生异常未能被处理,都会调用这个例程.   
    I. 见下面的例子 1:   
;//================================例子 1---final 型的异常处理=================   
;// ex. 1,by Hume,2001,just copy make your own hd.h and compile&link   
;//========================================================================   
.386   
.model flat, stdcall   
option casemap :none  ; case sensitive   
include hd.h          ;//相关的头文件，你自己维护一个吧   
;//============================   
.data   
szCap    db "By Hume\[AfO\],2001...",0   
szMsgOK db "OK,the exceptoin was handled by final handler!",0   
szMsgERR1 db "It would never Get here!",0   
buff    db 200 dup(0)   
  
.code   
\_start:   
;//========prog begin====================   
    lea    eax,Final\_Handler   
    invoke    SetUnhandledExceptionFilter,eax  ;//调用 SetUnhandledExceptionFilter 来安装 final SEH   
                                                  ;//原型很简单 SetUnhandledExceptionFilter proto   
                                                  ;//pTopLevelExceptionFilter:DWORD   
        xor    ecx,ecx   
        mov    eax,200       
        cdq   
    div    ecx   
                                                  ;//以下永远不会被执行   
        invoke    MessageBox,NULL,addr szMsgERR1,addr szCap,MB\_OK+MB\_ICONEXCLAMATION   
        invoke    ExitProcess,NULL   
          
  
;//============================   
Final\_Handler:   
    invoke    MessageBox,NULL,addr szMsgOK,addr szCap,MB\_OK+MB\_ICONEXCLAMATION   
    mov    eax,EXCEPTION\_EXECUTE\_HANDLER        ;//==1 这时不出现非法操作的讨厌对话框   
    ;mov    eax,EXCEPTION\_CONTINUE\_SEARCH    ;//==0 出现,这时是调用系统默认的异常处理过程,程序被终结了   
    ;mov    eax,EXCEPTION\_CONTINUE\_EXECUTION  ;//==-1 不断出现对话框,你将陷入死循环,可别怪我   
    ret                                      ;因为我们并没有修复 ecx,所以不断产生异常,然后不断调用这个例程   
  
;//=============================Prog Ends==============   
end \_start   
COMMENT $   
  简单来解释几句,windows 根据你的异常处理程序的返回值来决定如何进一步处理   
        EXCEPTION\_EXECUTE\_HANDLER            equ 1    表示我已经处理了异常,可以优雅地结束了   
        EXCEPTION\_CONTINUE\_SEARCH            equ 0    表示我不处理,其他人来吧,于是 windows 调用默认的处理程序   
                                                      显示一个错误框,并结束   
        EXCEPTION\_CONTINUE\_EXECUTION        equ -1  表示错误已经被修复,请从异常发生处继续执行   
        你可以试着让程序返回 0 和-1 然后编译程序,就会理解我所有苍白无力的语言...   
$   
  
;//========================================================================   
  
    II.另一种是 per\_Thread Exception Handler->线程相关的异常处理,通常每个线程初始化准备好运行时 fs 指向一个 TIB 结构   
    (THREAD INFORMATION BLOCK),这个结构的第一个元素 fs:\[0\]指向一个\_EXCEPTION\_REGISTRATION 结构   
    后面\_EXCEPTION\_REGISTRATION 为了简化,用 ERR 来代替这个结构...不要说没见过啊...   
    fs:\[0\]->   
    \_EXCEPTION\_REGISTRATION struc   
    prev dd ?                      ;前一个\_EXCEPTION\_REGISTRATION 结构   
    handler dd ?                  ;异常处理例程入口....呵呵,现在明白该怎么作了吧   
    \_EXCEPTION\_REGISTRATION ends   
    我们可以建立一个 ERR 结构然后将 fs:\[0\]换成指向他的指针,当然最常用的是堆栈,如果你用静态内存区也可以,没有人阻止你   
    在 asm 世界,放心地干吧,除了多 S 几次之外,通常不会有更大的损失   
    把 handler 域换成你的程序入口,就可以在发生异常时调用你的代码了,好马上实践一下,见例子 2   
;//========================================================================   
;// ex. 2,by Hume,2001  线程相关的异常处理   
;//========================================================================   
.386   
.model flat, stdcall   
option casemap :none  ; case sensitive   
include hd.h          ;//相关的头文件，你自己维护一个吧   
;//============================   
.data   
szCap    db "By Hume\[AfO\],2001...",0   
szMsgOK db "It's now in the Per\_Thread handler!",0   
szMsgERR1 db "It would never Get here!",0   
buff    db 200 dup(0)   
  
.code   
\_start:   
;//========prog begin====================   
  ASSUME FS:NOTHING   
        push    offset perThread\_Handler   
    push    fs:\[0\]        
        mov    fs:\[0\],esp                      ;//建立 SEH 的基本 ERR 结构,如果不明白,就仔细研究一下吧   
        xor    ecx,ecx                            
        mov    eax,200       
        cdq   
    div    ecx   
                                                  ;//以下永远不会被执行   
        invoke    MessageBox,NULL,addr szMsgERR1,addr szCap,MB\_OK+MB\_ICONINFORMATION   
        pop    fs:\[0\]   
        add    esp,4   
        invoke    ExitProcess,NULL          
  
;//============================   
perThread\_Handler:   
        invoke    MessageBox,NULL,addr szMsgOK,addr szCap,MB\_OK+MB\_ICONINFORMATION   
        mov    eax,1          ;//ExceptionContinueSearch,不处理,由其他例程或系统处理   
        ;mov    eax,0          ;//ExceptionContinueExecution,表示已经修复 CONTEXT,可从异常发生处继续执行   
    ret                        ;//这里如果返回 0,你会陷入死循环,不断跳出对话框....   
  
;//=============================Prog Ends==============   
end \_start   
COMMENT $   
  嘿嘿,这个简单吧,我们由于没有足够的资料,暂时还不能修复 ecx 的值使之从异常发生处继续执行,只是简单显示一个 MSG,然后   
  让系统处理,自然跳出讨厌的对话框了....   
  注意和 final 返回值的含义不同...   
$   
;//==================================================================================================   
  
好像到此为止,我们并没有从异常处理中得到任何好处,除了在异常发生后可以执行一点我们微不足道的代码,事实上 SEH 可以修复这些   
异常或者干我们想干的事情然后从希望的地方继续执行,嘿嘿,很爽吧,可惜我们没有足够的信息,那里找到我们所需要的信息?   
欲知后事如何,且看下回分解...   
  
                        
  
                            PARTII    继续深入   
                            ....(待续,睡个觉先) 

PART II 继续深入   
  
三、传递给异常处理句柄的参数   
  
      要想明白 seh 处理例程如何得到感兴趣的信息,首先要明白 SEH 的作用机制.事实上,当异常   
发生时,系统给了我们一个处理异常的机会,他首先会调用我们自定义的 seh 处理例程,当然也包括   
了相关信息,在调用之前,系统把包含这些信息结构的指针压入 stack,供我们的异常处理例程调用,   
传递给例程的参数通常是四个,其中只有三个有明确意义,另一个到现在为止还没有发现有什么作用,   
这些参数是:pExcept:DWORD,pErr:DWORD,pContext:DWORD,pDispatch 意义如下:   
pExcept:  ---  EXCEPTION\_RECORD 结构的指针   
pErr:    ---  前面 ERR 结构的指针   
pContext: --- CONTEXT 结构的指针   
pDispatch:---没有发现有啥意义   
  
    ERR 结构是前面介绍的\_EXCEPTION\_REGISTRATION 结构,往前翻翻,Dispatch 省略,下面介绍   
EXCEPTION\_RECORD 和 CONTEXT 结构的定义:   
    
;//================================以下是两个成员的详细结构========================================   
  
        EXCEPTION\_RECORD STRUCT   
          ExceptionCode        DWORD      ?      ;//异常码   
          ExceptionFlags        DWORD      ?      ;//异常标志   
          pExceptionRecord      DWORD      ?      ;//指向另外一个 EXCEPTION\_RECORD 的指针   
          ExceptionAddress      DWORD      ?      ;//异常发生的地址   
          NumberParameters      DWORD      ?      ;//下面 ExceptionInformation 所含有的 dword 数目   
          ExceptionInformation  DWORD EXCEPTION\_MAXIMUM\_PARAMETERS dup(?)   
        EXCEPTION\_RECORD ENDS                      ;//EXCEPTION\_MAXIMUM\_PARAMETERS ==15   
  
;//================================具体解释========================================   
  
ExceptionCode 异常类型,SDK 里面有很多类型,你可以在 windows.inc 里查找 STATUS\_ 来找到更多   
的异常类型,下面只给出 hex 值,具体标识定义请查阅 windows.inc,你最可能遇到的几种类型如下:   
  
              C0000005h----读写内存冲突   
              C0000094h----非法除 0   
              C00000FDh----堆栈溢出或者说越界   
              80000001h----由 Virtual Alloc 建立起来的属性页冲突   
              C0000025h----不可持续异常,程序无法恢复执行,异常处理例程不应处理这个异常   
              C0000026h----在异常处理过程中系统使用的代码,如果系统从某个例程莫名奇妙的返回,则出现此代码,   
                          如果 RtlUnwind 时没有 Exception Record 参数也同样会填入这个代码   
              80000003h----调试时因代码中 int3 中断   
              80000004h----处于被单步调试状态   
  
              注:也可以自己定义异常代码,遵循如下规则:   
              \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_+   
  
              位:      31~30            29~28          27~16          15~0   
              \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_+   
              含义:    严重程度          29 位            功能代码        异常代码   
                        0==成功        0==Mcrosoft    MICROSOFT 定义  用户定义   
                        1==通知        1==客户   
                        2==警告          28 位   
                        3==错误        被保留必须为 0   
ExceptionFlags 异常标志   
              0----可修复异常   
              1----不可修复异常   
              2----正在展开,不要试图修复什么,需要的话,释放必要的资源   
pExceptionRecord 如果程序本身导致异常,指向那个异常结构   
ExceptionAddress 发生异常的 eip 地址   
ExceptionInformation 附加消息,在调用 RaiseException 可指定或者在异常号为 C0000005h 即内存异常时含义如下   
              第一个 dword 0==读冲突 1==写冲突   
              第二个 dword 读写冲突地址   
;//================================解释结束========================================   
                                                          off.   
        CONTEXT STRUCT                    ; \_   
          ContextFlags  DWORD      ?      ;  |            +0   
          iDr0          DWORD      ?      ;  |            +4   
          iDr1          DWORD      ?      ;  |            +8   
          iDr2          DWORD      ?      ;  >调试寄存器  +C   
          iDr3          DWORD      ?      ;  |            +10   
          iDr6          DWORD      ?      ;  |            +14   
          iDr7          DWORD      ?      ; \_|            +18   
          FloatSave    FLOATING\_SAVE\_AREA <>  ;浮点寄存器区 +1C~~~88h   
          regGs        DWORD      ?      ;--|            +8C   
          regFs        DWORD      ?      ;  |\\段寄存器    +90    
          regEs        DWORD      ?      ;  |/            +94              
          regDs        DWORD      ?      ;--|            +98   
          regEdi        DWORD      ?      ;\_\_\_\_\_\_\_\_\_\_\_\_    +9C   
          regEsi        DWORD      ?      ;      |  通用  +A0   
          regEbx        DWORD      ?      ;      |  寄    +A4   
          regEdx        DWORD      ?      ;      |  存    +A8   
          regEcx        DWORD      ?      ;      |  器    +AC   
          regEax        DWORD      ?      ;\_\_\_\_\_\_\_|\_\_\_ 组\_  +B0        
          regEbp        DWORD      ?      ;++++++++++++++++ +B4   
          regEip        DWORD      ?      ;    |控制        +B8   
          regCs        DWORD      ?      ;    |寄存        +BC   
          regFlag      DWORD      ?      ;    |器组        +C0   
          regEsp        DWORD      ?      ;    |            +C4   
          regSs        DWORD      ?      ;++++++++++++++++ +C8   
          ExtendedRegisters db MAXIMUM\_SUPPORTED\_EXTENSION dup(?)   
        CONTEXT ENDS   
;//================================以上是两个成员的详细结构========================================   
  
      
        I、传递给 final 句柄的参数,只有两个可描述为 EXCEPTION\_POINTERS 结构,定义如下:   
            EXCEPTION\_POINTERS STRUCT   
              pExceptionRecord  DWORD      ?                
              ContextRecord    DWORD      ?   
            EXCEPTION\_POINTERS ENDS   
  
            在 call xHandler 之前,堆栈结构如下:   
            esp    -> \*EXCEPTION\_RECORD   
            esp+4  -> \*CONTEXT record  ;//具体结构见下面   
  
                然后执行 call \_Final\_Handler,这样在程序里要调用什么不轻而易举了吗?   
  
  
      
    II、 传递给 per\_thread 句柄的参数,如下:   
    在 call xHandler 之前,在堆栈中形成如下结构   
            esp    -> \*EXCEPTION\_RECORD   
            esp+4  -> \*ERR                    ;//注意这也就是 fs:\[0\]的指向   
            esp    -> \*CONTEXT record          ;//point to registers   
            esp    -> \*Param                    ;//呵呵,没有啥意义   
  
          然后执行 call \_Per\_Thread\_Handler   
  
调用 handler 的原型是这样   
        invoke HANDLER,\*EXCEPTION\_RECORD,\*\_EXCEPTION\_REGISTRATION,\*CONTEXT record,\*Param   
        即编译代码如下:   
        PUSH \*Param                    ;//通常不重要,没有什么意义   
        push \*CONTEXT record            ;//上面的结构   
        push \*ERR                      ;//the struc above   
        push \*EXCEPTION\_RECORD          ;//see above   
        CALL HANDLER   
        ADD ESP,10h   
  
  
好现在你明白了应该如何访问具体有关系统信息的细节了吧,下一部分,让我们来看看如何应用...(待续) 

利用 Apihook 到进程的入口，建立自己的 SEH 链表，在 SEH 的异常处理程序中建立 Debuger 功能，\\（也就是修改 Context.Eflags 的 TF 标志，然后单步异常时重新修改该标志\\），有了这个粗糙的 Debuger，我们可以在让其在单步中断到 SIDT 指令时修改 SIDT 指令的结果，让其取的是自己建立的一个假的 IDTR 植，同时自己建立一个假的 IDT 表，这样程序修改的将是自己的假 IDT 表，如果程序想 int 3 或 int 1 来反跟踪的话，就会发生异常，我们又可以处理以下了。。。。，然后如果只是为了脱壳的话，就 HOOK 到某个 API 或则修改程序的某处代码为 INT3，然后在异常处理程序中处理开始脱壳。特殊一点如果你想让利用修改 IDT 进入 Ring0 进行反跟踪的加壳的程序在 NT 下执行的话，也可以模拟一个它需要的环境给他吧:-)当然这中方法有一定的局限性.   
  
SEH 不仅仅是加壳者的宝刀，也是脱壳者的利箭。  4dd466d0e1e0cc123afbe504da2462ca