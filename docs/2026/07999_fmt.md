
# SEH in ASM 研究(一) (7千字)

## SEH in ASM 研究(一)

By Hume/冷雨飘心

为什么老调重弹:

SEH出现已绝非一日,但很多人可能还不彻底了解Seh的运行机制;有关seh的知识资料不是很多,asm级的详细资料就更少!seh不仅可以简化程序错误处理,使你的程序更加健壮,还被广泛应用于反跟踪以及加解密中,因此,了解seh非常必要,但遗憾的是关于seh详细介绍的中文资料非常少,在实践的基础上,把自己学习的一点笔记奉献给大家,希望对喜欢ASM的朋友有所帮助.如有错误,请高手不吝指正.

PART  I  简单接触

## 一、SEH背景知识

SEH("Structured Exception Handling"),即结构化异常处理.是操作系统提供给程序设计者的强有力的处理程序错误或异常的武器.在VISUAL C++中你或许已经熟悉了`_try{}` `_finally{}` 和`_try{}` `_except {}` 结构,这些并不是编译程序本身所固有的,本质上只不过是对windows内在提供的结构化异常处理的包装,不用这些高级语言编译器所提供的包装 ,照样可以利用系统提供的强大seh处理功能,在后面你将可以看到,用系统本身提供seh结构和规则以及ASM语言,我们将对SEH的机制以及实现有一个彻底的了解.

发生异常时系统的处理顺序(by Jeremy Gordon):

    1.系统首先判断异常是否应发送给目标程序的异常处理例程,如果决定应该发送,并且目标程序正在被调试,则系统挂起程序并向调试器发送`EXCEPTION_DEBUG_EVENT`消息.呵呵,这不是正好可以用来探测调试器的存在吗?
    2.如果你的程序没有被调试或者调试器未能处理异常,系统就会继续查找你是否安装了线程相关的异常处理例程,如果你安装了线程相关的异常处理例程,系统就把异常发送给你的程序seh处理例程,交由其处理.
    3.每个线程相关的异常处理例程可以处理或者不处理这个异常,如果他不处理并且安装了多个线程相关的异常处理例程,可交由链起来的其他例程处理.
    4.如果这些例程均选择不处理异常,如果程序处于被调试状态,操作系统仍会再次挂起程序通知debugger.
    5.如果程序未处于被调试状态或者debugger没有能够处理,并且你调用`SetUnhandledExceptionFilter`安装了最后异常处理例程的话,系统转向对它的调用.
    6.如果你没有安装最后异常处理例程或者他没有处理这个异常,系统会调用默认的系统处理程序,通常显示一个对话框,你可以选择关闭或者最后将其附加到调试器上的调试按钮.如果没有调试器能被附加于其上或者调试器也处理不了,系统就调用`ExitProcess`终结程序.
    7.不过在终结之前,系统仍然对发生异常的线程异常处理句柄来一次展开,这是线程异常处理例程最后清理的机会.

如果你看了上面的步骤一头雾水的话,别着急,化点时间慢慢理解或者进入下一部分实例操作.

## 二.初步实战演习:

安装异常处理句柄.

有两种类型的异常处理句柄,一种是final型的,这是在你的异常未能得到线程相关处理例程处理操作系统在即将关闭程序之前会回调的例程,这个例程是进程相关的而不是线程相关的,因此无论是哪个线程发生异常未能被处理,都会调用这个例程.

### I. 见下面的例子1:

```
;//================================例子1---final型的异常处理=================
;// ex. 1,by Hume,2001,just copy make your own hd.h and compile&link
;//========================================================================
.386
.model flat, stdcall
option casemap :none  ; case sensitive
include hd.h          ;//相关的头文件，你自己维护一个吧
;//============================
.data
szCap    db "By Hume[AfO],2001...",0
szMsgOK db "OK,the exceptoin was handled by final handler!",0
szMsgERR1 db "It would never Get here!",0
buff    db 200 dup(0)
.code
_start:
;//========prog begin====================
    lea    eax,Final_Handler
    invoke    SetUnhandledExceptionFilter,eax
;//调用SetUnhandledExceptionFilter来安装final SEH
      ;//原型很简单SetUnhandledExceptionFilter proto
      ;//pTopLevelExceptionFilter:DWORD
        xor    ecx,ecx
        mov    eax,200
        cdq
    div    ecx
      ;//以下永远不会被执行
        invoke    MessageBox,NULL,addr szMsgERR1,addr
szCap,MB_OK+MB_ICONEXCLAMATION
        invoke    ExitProcess,NULL
;//============================
Final_Handler:
    invoke    MessageBox,NULL,addr szMsgOK,addr
szCap,MB_OK+MB_ICONEXCLAMATION
    mov    eax,EXCEPTION_EXECUTE_HANDLER
      ;//==1 这时不出现非法操作的讨厌对话框
    ;mov    eax,EXCEPTION_CONTINUE_SEARCH
  ;//==0 出现,这时是调用系统默认的异常处理过程,程序被终结了
    ;mov    eax,EXCEPTION_CONTINUE_EXECUTION  ;//==-1
不断出现对话框,你将陷入死循环,可别怪我
    ret
;因为我们并没有修复ecx,所以不断产生异常,然后不断调用这个例程
;//=============================Prog Ends==============
end _start
```

简单来解释几句,windows根据你的异常处理程序的返回值来决定如何进一步处理

`EXCEPTION_EXECUTE_HANDLER` equ 1 表示我已经处理了异常,可以优雅地结束了

`EXCEPTION_CONTINUE_SEARCH` equ 0 表示我不处理,其他人来吧,于是windows调用默认的处理程序显示一个错误框,并结束

`EXCEPTION_CONTINUE_EXECUTION` equ -1 表示错误已经被修复,请从异常发生处继续执行

你可以试着让程序返回0和-1然后编译程序,就会理解我所有苍白无力的语言...

### II.另一种是per_Thread Exception Handler->线程相关的异常处理,通常每个线程初始化准备好运行时fs指向一个TIB结构
(THREAD INFORMATION BLOCK),这个结构的第一个元素`fs:[0]`指向一个`_EXCEPTION_REGISTRATION`结构后面`_EXCEPTION_REGISTRATION`为了简化,用ERR来代替这个结构...不要说没见过啊...

`fs:[0]->`

```
_EXCEPTION_REGISTRATION struc
prev dd ?
      ;前一个_EXCEPTION_REGISTRATION结构
handler dd ?
  ;异常处理例程入口....呵呵,现在明白该怎么作了吧
_EXCEPTION_REGISTRATION ends
```

我们可以建立一个ERR结构然后将`fs:[0]`换成指向他的指针,当然最常用的是堆栈,如果你用静态内存区也可以,没有人阻止你在asm世界,放心地干吧,除了多S几次之外,通常不会有更大的损失

把`handler`域换成你的程序入口,就可以在发生异常时调用你的代码了,好马上实践一下,见例子2

```
;//========================================================================
;// ex. 2,by Hume,2001  线程相关的异常处理
;//========================================================================
.386
.model flat, stdcall
option casemap :none  ; case sensitive
include hd.h          ;//相关的头文件，你自己维护一个吧
;//============================
.data
szCap    db "By Hume[AfO],2001...",0
szMsgOK db "It's now in the Per_Thread handler!",0
szMsgERR1 db "It would never Get here!",0
buff    db 200 dup(0)
.code
_start:
;//========prog begin====================
  ASSUME FS:NOTHING
        push    offset perThread_Handler
    push    fs:[0]
        mov    fs:[0],esp
                ;//建立SEH的基本ERR结构,如果不明白,就仔细研究一下吧
        xor    ecx,ecx
        mov    eax,200
        cdq
    div    ecx
      ;//以下永远不会被执行
        invoke    MessageBox,NULL,addr szMsgERR1,addr
szCap,MB_OK+MB_ICONINFORMATION
        pop    fs:[0]
        add    esp,4
        invoke    ExitProcess,NULL
;//============================
perThread_Handler:
        invoke    MessageBox,NULL,addr szMsgOK,addr
szCap,MB_OK+MB_ICONINFORMATION
        mov    eax,1
  ;//ExceptionContinueSearch,不处理,由其他例程或系统处理
        ;mov    eax,0
  ;//ExceptionContinueExecution,表示已经修复CONTEXT,可从异常发生处继续执行
    ret
        ;//这里如果返回0,你会陷入死循环,不断跳出对话框....
;//=============================Prog Ends==============
end _start
```

嘿嘿,这个简单吧,我们由于没有足够的资料,暂时还不能修复ecx的值使之从异常发生处继续执行,只是简单显示一个MSG,然后让系统处理,自然跳出讨厌的对话框了....

注意和final返回值的含义不同...

好像到此为止,我们并没有从异常处理中得到任何好处,除了在异常发生后可以执行一点我们微不足道的代码,事实上SEH可以修复这些异常或者干我们想干的事情然后从希望的地方继续执行,嘿嘿,很爽吧,可惜我们没有足够的信息,那里找到我们所需要的信息?欲知后事如何,且看下回分解...

PARTII    继续深入

....(待续,睡个觉先)
