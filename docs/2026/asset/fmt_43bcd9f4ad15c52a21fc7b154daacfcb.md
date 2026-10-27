
# SEH IN ASM 研究(二)---提高篇:关于异常处理的嵌套和堆栈展开 (15千字)

## SEH IN ASM 研究(二)

---提高篇

By Hume[AfO]/冷雨飘心

### part 4 关于异常处理的嵌套和堆栈展开

在实际程序设计过程中,不可能只有一个异常处理例程,这就产生了异常处理程序嵌套的问题,可能很多处理例程分别监视若干子程序并处理其中某种异常,另外一个监视所有子程序可能产生的共性异常,这作起来实际很容易,也方便调试.你只要依次建立异常处理框架就可以了.

关于VC++异常处理可以嵌套很多人可能比较熟悉,用起来更容易不过实现比这里也就复杂得多,在VC++中一个程序所有异常只指向一个相同的处理句例程,然后在这个处理例程里再实现对各个子异常处理例程的调用,他的大致方法是建立一个子异常处理例程入口的数组表,然后根据指针来调用子处理例程,过程比较烦琐,原来打算大致写一点,现在发现自己对C/C++了解实在太少,各位有兴趣还是自己参考MSDN Matt Pietrek 1996年写的一篇文章<Crash Course on the Depths of Win32? Stru

ctured Exception Handling>>,里面有非常详细的说明,对于系统的实现细节也有所讨论,不过相信很多人都没有兴趣.hmmm...:)实际上Kernel的异常处理过程和VC++的很相似.

有异常嵌套就涉及到异常展开的问题,也许你注意到了如果按照我前面的例子包括final都不处理异常的话,最后系统在终结程序之前会来一次展开,在试验之后发现,展开不会调用final只是对per\_thread例程展开(right?).什么是堆栈展开?为什么要进行堆栈展开?如何进行堆栈展开?

我曾经为堆栈展开迷惑过,原因是各种资料的描述很不一致,Matt Pietrek说展开后前面的ERR结构被释放,并且好像链后面如果决定处理必须展开,很多C/C++讲述异常处理的书也如斯说这使人很迷惑,我们再来看看Jeremy Gordon的描述,堆栈展开是处理异常的例程自愿进行的.呵呵,究竟事实如何?

在迷惑好久之后我终于找到了答案:Matt Pietrek讲的没有错,那是VC++以及系统kernel的处理方法,Jeremy Gordon说的也是正确的,那是我门asm Fans的自由!

好了现在来说堆栈展开,堆栈展开是异常处理例程在决定处理某个异常的时候给前面不处理这个异常的处理例程的一个清洗的机会,前面拒绝处理这个异常的例程可以释放必要的句柄对象或者释放堆栈或者干点别的工作...那完全是你的自由,叫stack unwind似乎有点牵强.堆栈展开有一个重要的标志就是`EXCEPTION_RECORD.ExceptionFlag`为2,表示正在展开,你可以进行相应的处理工作,但实际上经常用的是6这是因为还有一个UNWIND\_EXIT什么的,具体含义我也没有搞明白,不过对我们的工作好像没有什么影响.

注意在自己的异常处理例程中,unwind不是自动的,必须你自己自觉地引发,如果所有例程都不处理系统最后的展开是注定的,当然如果没有必要你也可以不展开.

win32提供了一个api `RtlUnwind`来引发展开,如果你想展开一下,就调用这个api吧,少候讲述自己代码如何展开`RtlUnwind`调用描述如下:

        PUSH Return value        ;返回值,一般不用
        PUSH pExceptionRecord    ;指向`EXCEPTION_RECORD`的指针
        PUSH OFFSET CodeLabel    ;展开后从哪里执行
        PUSH LastStackFrame      ;展开到哪个处理例程终止返回,通常是处理异常的Err结构
        CALL RtlUnwind

调用这个api之前要注意保护`ebx`,`esi`和`edi`,否则...嘿嘿

MASM格式如下:

        `invoke RtlUnwind,pFrame,OFFSET return_code_Address,pExceptionRecord,Return_value`

这样在展开的时候,就以`pExceptionRecord.flag=2` 依次调用前面的异常处理例程,到决定异常的处理例程停止,Jeremy Gordon手动展开代码和我下面的例子有所不同.他描述最后决定处理异常的ERR结构的prev成员为-1,好像我的结果和他的有所差异,因此采用了另外的方法,具体看下面的例子.

最后一点要注意在嵌套异常处理程序的时候要注意保存寄存器,否则你经常会得到系统异常代码为`C00000027h`的异常调用,然后就是被终结.

一下给出一点垃圾代码演示可能有助于理解,注意link的时候要加入 `/section:.text,RWE` 否则例子里面的代码段不能写,SMC功能会产生异常以致整个程序不能进行.

注意:2K/XP下非法指令异常的代码不一致,另外用下面的方法SMC代码段也不可以!不知如何解决?
只用于9X,为了在2k/Xp下也能运行我加了点代码,有兴趣看看,另外帮我解决一下2K/Xp下SMC的问题?thx!

下面例子很烂,不过MASM格式写起来容易一点,也便于理解.

```asm
;-----------------------------------------
;Ex4,演示堆栈展开和异常嵌套处理 by Hume,2002
;humewen@21cn.com
;hume.longcity.net
;-----------------------------------------

.586
.model flat, stdcall
option casemap :none  ; case sensitive
include hd.h
include mac.h
;;--------------

per_xHandler1        proto C :DWORD,:DWORD,:DWORD,:DWORD
per_xHandler2        proto C :DWORD,:DWORD,:DWORD,:DWORD
per_xHandler3        proto C :DWORD,:DWORD,:DWORD,:DWORD
;-----------------------------------------

.data

sztit  db "except Mess,by hume[AfO]",0
count  dd 0,0
Expt1_frm  dd 0
    ;ERR结构指针,用于堆栈展开手动代码
Expt2_frm  dd 0
Expt3_frm  dd 0
;;-----------------------------------------

    .CODE
_Start:
        assume  fs:nothing
        push    offset per_xHandler3
        push    fs:[0]
        mov    fs:[0],esp
        mov    Expt3_frm,esp
        push    offset per_xHandler2
        push    fs:[0]
        mov    fs:[0],esp
        mov    Expt2_frm,esp
        push    offset per_xHandler1
        push    fs:[0]
        mov    fs:[0],esp
        mov    Expt1_frm,esp
        ;--------------------------
        ;install xhnadler
        ;-----------------------------------------
        xor    ebx,ebx
        mov    eax,200
        cdq
        div    ebx
          ;除法错误
        invoke    MessageBox,0,ddd("Good,divide
overflow was solved!"),addr sztit,40h
        sub    eax,eax
        mov    [eax],ebx
    ;内存写错误
succ:
        invoke    MessageBox,0,ddd("Good,memory
write violation solved!"),addr sztit,40h
        db      0F0h,0Fh,0C7h,0C8h
;什么cmpchg8b指令的非法形式?我从来没有成功过!!
              ;演示程序中使用seh实现SMC技术,加密??...
        invoke    MessageBox,0,ddd("illeagal
instruction was solved!"),addr sztit,20h
        ;--------------------------
        ;uninstall xhnadler
        ;-----------------------------------------
        pop    fs:[0]
        add    esp,4
        pop    fs:[0]
        add    esp,4
      ;或者add esp,10h
        pop    fs:[0]
        add    esp,4
    invoke    ExitProcess,0
;-----------------------------------------
;异常处理句柄1,处理除法异常错误
per_xHandler1 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD
        pushad
        MOV    ESI,pExcept
ASSUME  ESI:PTR EXCEPTION_RECORD
        TEST    [ESI].ExceptionFlags,1
        JNZ    @cantdo1
        TEST    [ESI].ExceptionFlags,6
        JNZ    @unwind1
        CMP    [ESI].ExceptionCode,0C0000094h
        JNZ    @cantdo1
        MOV    EDI,pContext
ASSUME  EDI:PTR CONTEXT
        m2m    [edi].regEbx,20
      ;将ebx置20,修复除法错误,继续执行
        popad
    MOV  EAX, ExceptionContinueExecution
        RET
@unwind1:
        invoke    MessageBox,0,CTEXT("state:
unwinding in xhandler1..."),addr sztit,0
@cantdo1:
        popad
        MOV    EAX,ExceptionContinueSearch
    RET
per_xHandler1 ENDP
;-----------------------------------------
;异常处理句柄2,处理内存写错误,扩展可以有其他的例子如自动扩充堆栈
per_xHandler2 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD
        pushad
    MOV    ESI,pExcept
ASSUME  ESI:PTR EXCEPTION_RECORD
        MOV    EDI,pContext
ASSUME  EDI:PTR CONTEXT
        call    Dispcont
                    ;显示一点lame的消息,自己调试用
        TEST    [ESI].ExceptionFlags,1
        JNZ    @cantdo2
        TEST    [ESI].ExceptionFlags,6
        JNZ    @unwind2
        CMP    [ESI].ExceptionCode,0C0000005h
        JNZ    @cantdo2
        .data
    ;ASM的数据定义灵活性,如果需要这是可以的
        validAddress dd 0
        .code
        m2m    [EDI].regEax,<offset  validAddress>
  ;置eax为有效地址
        popad
    MOV  EAX, ExceptionContinueExecution
        RET
@unwind2:
        invoke    MessageBox,0,CTEXT("hmmm...
unwinding in xhandler2..."),addr sztit,40h
@cantdo2:
        popad
        MOV    EAX,ExceptionContinueSearch
    RET
per_xHandler2 ENDP
;-----------------------------------------
per_xHandler3 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD
    pushad
        MOV    ESI,pExcept
ASSUME  ESI:PTR EXCEPTION_RECORD
        MOV    EDI,pContext
ASSUME  EDI:PTR CONTEXT
        TEST    [ESI].ExceptionFlags,1
        JNZ    @cantdo3
        TEST    [ESI].ExceptionFlags,6
        JNZ    @unwind3
        ;-----------------------------------------
        push    ecx
        mov    ecx,cs
        xor    cl,cl
        jecxz  win2k_Xp
win9X:
        pop    ecx
        CMP    [ESI].ExceptionCode,0C000001DH
    ;非法指令异常,与2K/XP下的不一致
        JNZ    @cantdo3
        jmp    ok_here
win2k_Xp:
        pop    ecx
    ;注意，只有在9X下才可以
        CMP    [ESI].ExceptionCode,0C000001EH
    ;非法指令异常->2K/XP
        JNZ    @cantdo3
                    ;sMc不成
        mov    [edi].regEip,offset safereturn
        popad
        mov    eax,0
        ret
        push    ebx
        push    esi
        push    edi
comment $ 调用RtlUnwind展开堆栈
        lea    ebx,unwindback
        invoke    RtlUnwind,Expt3_frm,ebx,esi,0
        $
        mov    dword ptr [esi+4],2
                ;置展开标志,准备展开,这里是
          ;手动代码
        mov    ebx,fs:[0]
selfun:
        ;mov    eax,Expt2_frm
              ;这里显示了ASM手动展开的灵活性
        mov    eax,Expt3_frm
        cmp    ebx,eax
                    ;按照Jeremy
Gordon的好像不大对头
        ;cmp    dword ptr [ebx],-1
            ;这样好像有问题,只好如上,请教答案
        jz      unwindback
        push    ebx
        push    esi
  ; 压入Err和Exeption_registration结构
        call    dword ptr[ebx+4]
        add    esp,8
        mov    ebx,[ebx]
        jmp    selfun
unwindback:
        invoke    MessageBox,0,CTEXT("I am
Back!"),addr sztit,40h
        pop    edi
```
