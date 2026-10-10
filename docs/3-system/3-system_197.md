

# SEH IN ASM 研究(二) 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue370.htm`](https://www.pediy.com/kssd/pediy04/kanxue370.htm)

SEH IN ASM 研究(二)   
                                        ---提高篇   
                                      By Hume\[AfO\]/冷雨飘心   
  
                        part 4  关于异常处理的嵌套和堆栈展开   
  
    在实际程序设计过程中,不可能只有一个异常处理例程,这就产生了异常处理程序嵌套的问题,可能很多   
处理例程分别监视若干子程序并处理其中某种异常,另外一个监视所有子程序可能产生的共性异常,这作   
起来实际很容易,也方便调试.你只要依次建立异常处理框架就可以了.   
    关于 VC++异常处理可以嵌套很多人可能比较熟悉,用起来更容易不过实现比这里也就复杂得多,在 VC++   
中一个程序所有异常只指向一个相同的处理句例程,然后在这个处理例程里再实现对各个子异常处理例程的   
调用,他的大致方法是建立一个子异常处理例程入口的数组表,然后根据指针来调用子处理例程,过程比较烦   
琐,原来打算大致写一点,现在发现自己对 C/C++了解实在太少,各位有兴趣还是自己   
参考 MSDN Matt Pietrek 1996 年写的一篇文章<Crash Course on the Depths of Win32? Stru >  
ctured Exception Handling>>,里面有非常详细的说明,对于系统的实现细节也有所讨论,不过相信很多   
人都没有兴趣.hmmm...:)实际上 Kernel 的异常处理过程和 VC++的很相似.   
  
    有异常嵌套就涉及到异常展开的问题,也许你注意到了如果按照我前面的例子包括 final 都不处理异常的话,   
最后系统在终结程序之前会来一次展开,在试验之后发现,展开不会调用 final 只是对 per\_thread 例程展开   
(right?).什么是堆栈展开?为什么要进行堆栈展开?如何进行堆栈展开?   
      
    我曾经为堆栈展开迷惑过,原因是各种资料的描述很不一致,Matt Pietrek 说展开后前面的 ERR 结构被释放,   
并且好像链后面如果决定处理必须展开,很多 C/C++讲述异常处理的书也如斯说这使人很迷惑,我们再来看看 Jeremy   
Gordon 的描述,堆栈展开是处理异常的例程自愿进行的.呵呵,究竟事实如何?   
      
    在迷惑好久之后我终于找到了答案:Matt Pietrek 讲的没有错,那是 VC++以及系统 kernel 的处理方法,Jeremy   
Gordon 说的也是正确的,那是我门 asm Fans 的自由!   
  
    好了现在来说堆栈展开,堆栈展开是异常处理例程在决定处理某个异常的时候给前面不处理这个异常的处理   
例程的一个清洗的机会,前面拒绝处理这个异常的例程可以释放必要的句柄对象或者释放堆栈或者干点别的工作...   
那完全是你的自由,叫 stack unwind 似乎有点牵强.堆栈展开有一个重要的标志就是   
EXCEPTION\_RECORD.ExceptionFlag 为 2,表示正在展开,你可以进行相应的处理工作,但实际上经常用的是 6 这是   
因为还有一个 UNWIND\_EXIT 什么的,具体含义我也没有搞明白,不过对我们的工作好像没有什么影响.   
  
  注意在自己的异常处理例程中,unwind 不是自动的,必须你自己自觉地引发,如果所有例程都不处理系统最后的   
展开是注定的,当然如果没有必要你也可以不展开.   
  win32 提供了一个 api RtlUnwind 来引发展开,如果你想展开一下,就调用这个 api 吧,少候讲述自己代码如何展开   
  
RtlUnwind 调用描述如下:   
        PUSH Return value        ;返回值,一般不用   
        PUSH pExceptionRecord    ;指向 EXCEPTION\_RECORD 的指针   
        PUSH OFFSET CodeLabel    ;展开后从哪里执行   
        PUSH LastStackFrame      ;展开到哪个处理例程终止返回,通常是处理异常的 Err 结构   
        CALL RtlUnwind   
调用这个 api 之前要注意保护 ebx,esi 和 edi,否则...嘿嘿   
  
MASM 格式如下:          
        invoke RtlUnwind,pFrame,OFFSET return\_code\_Address,pExceptionRecord,Return\_value   
  
这样在展开的时候,就以 pExceptionRecord.flag=2 依次调用前面的异常处理例程,到决定异常的处理例程停止,   
Jeremy Gordon 手动展开代码和我下面的例子有所不同.他描述最后决定处理异常的 ERR 结构的 prev 成员为-1,好像   
我的结果和他的有所差异,因此采用了另外的方法,具体看下面的例子.   
      
  最后一点要注意在嵌套异常处理程序的时候要注意保存寄存器,否则你经常会得到系统异常代码为 C00000027h   
的异常调用,然后就是被终结.   
  
  一下给出一点垃圾代码演示可能有助于理解,注意 link 的时候要加入 /section:.text,RWE 否则例子里面的   
代码段不能写,SMC 功能会产生异常以致整个程序不能进行.   
  
注意:2K/XP 下非法指令异常的代码不一致,另外用下面的方法 SMC 代码段也不可以!不知如何解决?   
只用于 9X,为了在 2k/Xp 下也能运行我加了点代码,有兴趣看看,另外帮我解决一下 2K/Xp 下 SMC 的问题?thx!   
  
    下面例子很烂,不过 MASM 格式写起来容易一点,也便于理解.   
;-----------------------------------------   
;Ex4,演示堆栈展开和异常嵌套处理 by Hume,2002   
;humewen@21cn.com   
;hume.longcity.net   
;-----------------------------------------   
.586   
.model flat, stdcall   
option casemap :none  ; case sensitive   
include hd.h   
include mac.h   
  
;;--------------   
per\_xHandler1        proto C :DWORD,:DWORD,:DWORD,:DWORD   
per\_xHandler2        proto C :DWORD,:DWORD,:DWORD,:DWORD   
per\_xHandler3        proto C :DWORD,:DWORD,:DWORD,:DWORD   
;-----------------------------------------   
  
.data   
sztit  db "except Mess,by hume\[AfO\]",0   
count  dd 0,0   
Expt1\_frm  dd 0                    ;ERR 结构指针,用于堆栈展开手动代码   
Expt2\_frm  dd 0   
Expt3\_frm  dd 0   
          
;;-----------------------------------------   
    .CODE   
\_Start:   
        assume  fs:nothing   
        push    offset per\_xHandler3   
        push    fs:\[0\]   
        mov    fs:\[0\],esp   
        mov    Expt3\_frm,esp   
  
        push    offset per\_xHandler2   
        push    fs:\[0\]   
        mov    fs:\[0\],esp   
        mov    Expt2\_frm,esp   
  
        push    offset per\_xHandler1   
        push    fs:\[0\]   
        mov    fs:\[0\],esp   
        mov    Expt1\_frm,esp   
        ;--------------------------   
        ;install xhnadler   
        ;-----------------------------------------   
          
        xor    ebx,ebx   
        mov    eax,200   
        cdq   
        div    ebx                  ;除法错误   
  
        invoke    MessageBox,0,ddd("Good,divide overflow was solved!"),addr sztit,40h   
  
        sub    eax,eax   
        mov    \[eax\],ebx            ;内存写错误   
  
succ:   
        invoke    MessageBox,0,ddd("Good,memory write violation solved!"),addr sztit,40h   
          
        db      0F0h,0Fh,0C7h,0C8h  ;什么 cmpchg8b 指令的非法形式?我从来没有成功过!!   
                                    ;演示程序中使用 seh 实现 SMC 技术,加密??...   
        invoke    MessageBox,0,ddd("illeagal instruction was solved!"),addr sztit,20h   
        ;--------------------------   
        ;uninstall xhnadler   
        ;-----------------------------------------   
  
        pop    fs:\[0\]              
        add    esp,4   
        pop    fs:\[0\]              
        add    esp,4   
      ;或者 add esp,10h   
        
        pop    fs:\[0\]              
        add    esp,4   
  
    invoke    ExitProcess,0   
;-----------------------------------------          
;异常处理句柄 1,处理除法异常错误   
per\_xHandler1 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   
        pushad   
        MOV    ESI,pExcept   
ASSUME  ESI:PTR EXCEPTION\_RECORD   
        TEST    \[ESI\].ExceptionFlags,1   
        JNZ    @cantdo1   
        TEST    \[ESI\].ExceptionFlags,6   
        JNZ    @unwind1   
        CMP    \[ESI\].ExceptionCode,0C0000094h   
        JNZ    @cantdo1   
        MOV    EDI,pContext   
  
ASSUME  EDI:PTR CONTEXT   
        m2m    \[edi\].regEbx,20            ;将 ebx 置 20,修复除法错误,继续执行   
        popad   
    MOV  EAX, ExceptionContinueExecution   
        RET   
  
@unwind1:   
        invoke    MessageBox,0,CTEXT("state: unwinding in xhandler1..."),addr sztit,0   
@cantdo1:   
        popad   
        MOV    EAX,ExceptionContinueSearch   
    RET   
per\_xHandler1 ENDP   
;-----------------------------------------   
;异常处理句柄 2,处理内存写错误,扩展可以有其他的例子如自动扩充堆栈   
per\_xHandler2 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   
          
        pushad   
    MOV    ESI,pExcept   
ASSUME  ESI:PTR EXCEPTION\_RECORD   
        MOV    EDI,pContext   
ASSUME  EDI:PTR CONTEXT   
  
        call    Dispcont                            ;显示一点 lame 的消息,自己调试用   
  
        TEST    \[ESI\].ExceptionFlags,1   
        JNZ    @cantdo2   
        TEST    \[ESI\].ExceptionFlags,6   
        JNZ    @unwind2   
        CMP    \[ESI\].ExceptionCode,0C0000005h   
        JNZ    @cantdo2   
        .data                                        ;ASM 的数据定义灵活性,如果需要这是可以的   
        validAddress dd 0   
        .code   
          
        m2m    \[EDI\].regEax,<offset  validAddress>    ;置 eax 为有效地址        
        popad   
    MOV  EAX, ExceptionContinueExecution   
        RET   
  
@unwind2:   
        invoke    MessageBox,0,CTEXT("hmmm... unwinding in xhandler2..."),addr sztit,40h   
@cantdo2:   
        popad   
        MOV    EAX,ExceptionContinueSearch   
    RET   
per\_xHandler2 ENDP   
;-----------------------------------------   
  
per\_xHandler3 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   
    pushad   
        MOV    ESI,pExcept   
ASSUME  ESI:PTR EXCEPTION\_RECORD   
        MOV    EDI,pContext   
ASSUME  EDI:PTR CONTEXT        
  
        TEST    \[ESI\].ExceptionFlags,1   
        JNZ    @cantdo3   
        TEST    \[ESI\].ExceptionFlags,6   
        JNZ    @unwind3   
        ;-----------------------------------------   
                                                      
        push    ecx   
        mov    ecx,cs   
        xor    cl,cl   
        jecxz  win2k\_Xp          
win9X:   
        pop    ecx   
        CMP    \[ESI\].ExceptionCode,0C000001DH      ;非法指令异常,与 2K/XP 下的不一致   
        JNZ    @cantdo3   
        jmp    ok\_here   
win2k\_Xp:   
        pop    ecx                                  ;注意，只有在 9X 下才可以   
        CMP    \[ESI\].ExceptionCode,0C000001EH      ;非法指令异常->2K/XP   
        JNZ    @cantdo3                            ;sMc 不成   
  
        mov    \[edi\].regEip,offset safereturn   
        popad   
        mov    eax,0   
        ret   
          
          
          
        push    ebx   
        push    esi   
        push    edi        
comment $ 调用 RtlUnwind 展开堆栈          
        lea    ebx,unwindback   
        invoke    RtlUnwind,Expt3\_frm,ebx,esi,0   
        $   
        mov    dword ptr \[esi+4\],2                    ;置展开标志,准备展开,这里是   
                                                      ;手动代码   
        mov    ebx,fs:\[0\]   
          
  
selfun:   
        ;mov    eax,Expt2\_frm                    ;这里显示了 ASM 手动展开的灵活性   
        mov    eax,Expt3\_frm                        
        cmp    ebx,eax                            ;按照 Jeremy Gordon 的好像不大对头   
        ;cmp    dword ptr \[ebx\],-1                ;这样好像有问题,只好如上,请教答案            
        jz      unwindback   
        push    ebx   
        push    esi                                ; 压入 Err 和 Exeption\_registration 结构   
        call    dword ptr\[ebx+4\]   
        add    esp,8   
        mov    ebx,\[ebx\]   
        jmp    selfun   
  
unwindback:   
        invoke    MessageBox,0,CTEXT("I am Back!"),addr sztit,40h   
        pop    edi   
        pop    esi   
        pop    ebx                                  ;一定要保存这三个寄存器！   
  
        MOV    EAX,\[EDI\].regEip   
        MOV    DWORD PTR\[EAX\],90909090H            ;改为 nop 指令...SMC 呵呵这次不神秘了吧   
                                                    ;SMC 注意连接选项   
        popad   
    MOV  EAX, ExceptionContinueExecution   
        RET   
  
@unwind3:   
        invoke    MessageBox,0,CTEXT("Note... unwinding in xhandler3..."),addr sztit,40h   
@cantdo3:   
        popad   
        MOV    EAX,ExceptionContinueSearch   
    RET   
per\_xHandler3 ENDP   
;-----------------------------------------   
;lame routine for debug   
Dispcont    proc   
                inc    count   
                call    dispMsg   
                ret   
Dispcont    endp   
  
dispMsg    proc   
        local szbuf\[200\]:byte   
        pushad   
        mov    eax,dword ptr\[esi\]   
        mov    ebx,dword ptr\[esi+4\]   
        mov    ecx,dword ptr\[edi+0b8h\]   
        mov    edx,dword ptr\[edi+0a4h\]   
        .data   
        fmt    db "Context eip--> %8X  ebx--> %8X ",0dh,0ah   
                db "Flags  Ex.c-> %8x  flg--> %8X",0dh,0ah   
                db "it's the %d times xhandler was called!",0   
        .code   
        invoke    wsprintf,addr szbuf,addr fmt,ecx,edx,eax,ebx,count   
        invoke    MessageBox,0,addr szbuf,CTEXT("related Mess of context"),0   
        popad   
    ret   
dispMsg    endp   
  
;;------------------------------------------------   
END    \_Start   
;---------------------------------下面是上面用到的宏,我的 mac.h 比较长,就不贴了-----   
    ddd    MACRO Text                        ;define data in .data section   
        local name                  ;This and other can be used as: ddd("My god!")   
        .data                  ;isn't cool?   
            name    db Text,0   
        .code   
        EXITM <addr name>    
    ENDM   
  
  CTEXT MACRO y:VARARG                    ;This is a good macro   
        LOCAL sym   
    CONST segment   
        IFIDNI <y>,<>   
            sym db 0   
        ELSE   
            sym db y,0   
        ENDIF   
    CONST ends   
        EXITM <OFFSET sym>   
    ENDM   
  
    m2m MACRO M1, M2                          ;mov is too boring sometimes!   
      push M2   
      pop  M1   
    ENDM      
;-----------------------------------------   
  最后更正一点前面介绍的传送给 final 型的参数是指向 EXCEPTION\_POINTERS 的指针,压栈前的堆栈   
是如下的,不好意思,原来写的时候我也没深入研究,可能模糊了一点,如有错误,请大家指正   
push    ptEXCEPTION\_POINTERS   
call    xHandler   
下面补充一个 final 参数获得的一个例子   
;--------------------------------------------   
; Ex5,演示 final 处理句柄的参数获取,更正前面   
; 模糊的介绍   
;--------------------------------------------   
.586   
.model flat, stdcall   
option casemap :none  ; case sensitive   
include hd.h   
include mac.h   
  
;;--------------   
.data   
sztit  db "exceptION MeSs,by hume\[AfO\]",0   
fmt    db "Context eip--> %8X  ebx--> %8X ",0dh,0ah   
        db "Flags  Ex.c-> %8x  flg--> %8X",0   
szbuf  db 200 dup(0)   
;;-----------------------------------------   
    .CODE   
\_Start:   
        assume  fs:nothing   
        push    offset \_final\_xHandler0   
        call    SetUnhandledExceptionFilter   
        xor    ebx,ebx   
        mov    eax,200   
        cdq   
        div    ebx   
        invoke    MessageBox,0,ddd("Good,divide overflow was solved!"),addr sztit,40h   
        xor    eax,eax   
        mov    \[eax\],ebx   
          
    invoke    ExitProcess,0      
  
;-----------------------------------------   
\_final\_xHandler0:   
        push    ebp   
        mov    ebp,esp   
          
        mov    eax,\[ebp+8\]      ;the pointer to EXCEPTION\_POINTERS   
        mov    esi,\[eax\]        ;pointer to \_EXCEPTION\_RECORD   
        mov    edi,\[eax+4\]      ;pointer to \_CONTEXT   
        test    dword ptr\[esi+4\],1    
        jnz    @\_final\_cnotdo   
        test    dword ptr\[esi+4\],6    
        jnz    @\_final\_unwind   
  
        ;call    dispMsg   
          
  
        cmp    dword ptr\[esi\],0c0000094h   
        jnz    @\_final\_cnotdo   
  
        mov    dword ptr \[edi+0a4h\],10   
        call    dispMsg   
        
        mov    eax,EXCEPTION\_CONTINUE\_EXECUTION      ;GO ON   
        jmp    @f   
  
@\_final\_unwind:   
        invoke    MessageBox,0,CTEXT("state:In final unwind..."),addr sztit,0   
                                          ;好像不论处理不处理异常,系统展开的时候   
                                          ;都不会被调用,right?   
@\_final\_cnotdo:                            ;请教是真的吗?还是我写的有问题   
        mov    eax,EXCEPTION\_CONTINUE\_SEARCH   
        jmp    @f          
@@:        
        mov    esp,ebp   
        pop    ebp   
        ret   
;-----------------------------------------   
dispMsg    proc   
        pushad   
        mov    eax,\[esi\]   
        mov    ebx,\[esi+4\]   
        mov    ecx,\[edi+0b8h\]   
        mov    edx,\[edi+0a4h\]   
        invoke    wsprintf,addr szbuf,addr fmt,ecx,edx,eax,ebx   
        invoke    MessageBox,0,addr szbuf,CTEXT("related Mess of context"),0   
        popad   
    ret   
dispMsg    endp   
;;------------------------------------------------   
          
END    \_Start   
;====================================================================================   
  
BTW:够长了吧,基本内容介绍完毕,更多内容下一部分介绍一点利用 Seh 的 tricks,哪位大侠有什么好的想法   
或者有什么错误,请不吝指正,毕竟我是菜鸟吗...   
  
\=====================================================================================   
一点闲话:请 CUT--------------------------   
    最近很郁闷,生活上的工作上的,就这样懒懒懒散散地目无光彩地苟活于世,最近看了一点 C++,有的地方   
头大,于是拿起老家伙 asm 写了点以前许诺过的东西,感觉还是 ASM 最有助于理解基本原理~~~不过 C/C++给了我   
们另外的工具,虽然目标代码不够紧凑(理想状态),毕竟不需要我们每个人写内核,C/C++可以提高生产能力.   
    crack 也一样,如果一些基本概念都不懂明白还谈什么 crack?昨天看精华 III 一些高手的文章,真是惭愧啊!   
  
    我的专业不是计算机或者以后永远也不会搞计算机,这些当也许只是业余爱好,永远不会放弃的业余爱好.   
    以后也许很长一段时间里面我会离开大家,毕竟,面临的还有生活.   
    感谢 CCG,BCG 的各位高手,AfO 的成员们,以及那些曾无私指导过以及给我生活动力的朋友们!   
    我要奋斗!   
    我以此激励自己也同样希望能够激励大家!   
                                                        2002.1.28 深夜~~沉思中   
\----------------------------------------cut------------------------------------  4dd466d0e1e0cc123afbe504da2462ca