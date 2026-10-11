
```
        CMP    [ESI].ExceptionCode,0C000001EH      ;非法指令异常->2K/XP   

        JNZ    @cantdo3                              ;sMc 不成   

        mov    [edi].regEip,offset safereturn   

        popad   

        mov    eax,0   

        ret   

        push    ebx   

        push    esi   

        push    edi         

comment $ 调用 RtlUnwind 展开堆栈       

        lea    ebx,unwindback   

        invoke    RtlUnwind,Expt3_frm,ebx,esi,0   

        $   

        mov    dword ptr [esi+4],2                   ;置展开标志,准备展开,这里是   

                                                    ;手动代码   

        mov    ebx,fs:[0]   

selfun:   

        ;mov    eax,Expt2_frm                   ;这里显示了 ASM 手动展开的灵活性   

        mov    eax,Expt3_frm                      

        cmp    ebx,eax                              ;按照 Jeremy Gordon 的好像不大对头   

        ;cmp    dword ptr [ebx],-1                 ;这样好像有问题,只好如上,请教答案         

        jz     unwindback   

        push    ebx   

        push    esi                                  ; 压入 Err 和 Exeption_registration 结构   

        call    dword ptr[ebx+4]   

        add    esp,8   

        mov    ebx,[ebx]   

        jmp    selfun   

unwindback:   

        invoke    MessageBox,0,CTEXT("I am Back!"),addr sztit,40h   

        pop    edi   

        pop    esi   

        pop    ebx                                   ;一定要保存这三个寄存器！   

        MOV    EAX,[EDI].regEip   

        MOV    DWORD PTR[EAX],90909090H              ;改为 nop 指令...SMC 呵呵这次不神秘了吧   

                                                    ;SMC 注意连接选项   

        popad   

    MOV  EAX, ExceptionContinueExecution   

        RET   

@unwind3:   

        invoke    MessageBox,0,CTEXT("Note... unwinding in xhandler3..."),addr sztit,40h   

@cantdo3:   

        popad   

        MOV    EAX,ExceptionContinueSearch   

    RET   

per_xHandler3 ENDP   

;-----------------------------------------   

;lame routine for debug   

Dispcont    proc   

            inc    count   

            call    dispMsg   

            ret   

Dispcont    endp   

dispMsg    proc   

        local szbuf[200]:byte   

        pushad   

        mov    eax,dword ptr[esi]   

        mov    ebx,dword ptr[esi+4]   

        mov    ecx,dword ptr[edi+0b8h]   

        mov    edx,dword ptr[edi+0a4h]   

        .data   

        fmt    db "Context eip--> %8X  ebx--> %8X ",0dh,0ah   

            db "Flags  Ex.c-> %8x  flg--> %8X",0dh,0ah   

            db "it's the %d times xhandler was called!",0   

        .code   

        invoke    wsprintf,addr szbuf,addr fmt,ecx,edx,eax,ebx,count   

        invoke    MessageBox,0,addr szbuf,CTEXT("related Mess of context"),0   

        popad   

    ret   

dispMsg    endp   

;;------------------------------------------------   

END    _Start
```

;---------------------------------下面是上面用到的宏,我的 `mac.h` 比较长,就不贴了-----

```
    ddd    MACRO Text                     ;define data in .data section   

        local name                        ;This and other can be used as: ddd("My god!")   

        .data                            ;isn't cool?   

            name    db Text,0   

        .code   

        EXITM <addr name>   

    ENDM   

    CTEXT MACRO y:VARARG                     ;This is a good macro   

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

    m2m MACRO M1, M2                           ;mov is too boring sometimes!   

        push M2   

        pop  M1   

    ENDM   
```

;-----------------------------------------

最后更正一点前面介绍的传送给 `final` 型的参数是指向 `EXCEPTION_POINTERS` 的指针,压栈前的堆栈是如下的,不好意思,原来写的时候我也没深入研究,可能模糊了一点,如有错误,请大家指正

```
push    ptEXCEPTION_POINTERS   

call    xHandler
```

下面补充一个 `final` 参数获得的一个例子

```
;--------------------------------------------   

; Ex5,演示 final 处理句柄的参数获取,更正前面   

; 模糊的介绍   

;--------------------------------------------   

.586   

.model flat, stdcall   

option casemap :none  ; case sensitive   

include hd.h   

include mac.h   

;;--------------   

.data   

sztit  db "exceptION MeSs,by hume[AfO]",0   

fmt    db "Context eip--> %8X  ebx--> %8X ",0dh,0ah   

        db "Flags  Ex.c-> %8x  flg--> %8X",0   

szbuf  db 200 dup(0)   

;;-----------------------------------------   

    .CODE   

_Start:   

        assume  fs:nothing   

        push    offset _final_xHandler0   

        call    SetUnhandledExceptionFilter   

        xor    ebx,ebx   

        mov    eax,200   

        cdq   

        div    ebx   

        invoke    MessageBox,0,ddd("Good,divide overflow was solved!"),addr sztit,40h   

        xor    eax,eax   

        mov    [eax],ebx   

    invoke    ExitProcess,0     

;-----------------------------------------   

_final_xHandler0:   

        push    ebp   

        mov    ebp,esp   

        mov    eax,[ebp+8]     ;the pointer to EXCEPTION_POINTERS   

        mov    esi,[eax]       ;pointer to _EXCEPTION_RECORD   

        mov    edi,[eax+4]     ;pointer to _CONTEXT   

        test    dword ptr[esi+4],1   

        jnz    @_final_cnotdo   

        test    dword ptr[esi+4],6   

        jnz    @_final_unwind   

        ;call    dispMsg   

        cmp    dword ptr[esi],0c0000094h   

        jnz    @_final_cnotdo   

        mov    dword ptr [edi+0a4h],10   

        call    dispMsg   

        mov    eax,EXCEPTION_CONTINUE_EXECUTION     ;GO ON   

        jmp    @f   

@_final_unwind:   

        invoke    MessageBox,0,CTEXT("state:In final unwind..."),addr sztit,0   

                                                    ;好像不论处理不处理异常,系统展开的时候   

                                                    ;都不会被调用,right?   

@_final_cnotdo:                       ;请教是真的吗?还是我写的有问题   

        mov    eax,EXCEPTION_CONTINUE_SEARCH   

        jmp    @f         

@@:         

        mov    esp,ebp   

        pop    ebp   

        ret   

;-----------------------------------------   

dispMsg    proc   

        pushad   

        mov    eax,[esi]   

        mov    ebx,[esi+4]   

        mov    ecx,[edi+0b8h]   

        mov    edx,[edi+0a4h]   

        invoke    wsprintf,addr szbuf,addr fmt,ecx,edx,eax,ebx   

        invoke    MessageBox,0,addr szbuf,CTEXT("related Mess of context"),0   

        popad   

    ret   

dispMsg    endp   

;;------------------------------------------------   

END    _Start
```

`4dd466d0e1e0cc123afbe504da2462ca`

`BTW`:够长了吧,基本内容介绍完毕,更多内容下一部分介绍一点利用 `Seh` 的 `tricks`,哪位大侠有什么好的想法或者有什么错误,请不吝指正,毕竟我是菜鸟吗...

====================================================================================

一点闲话:请 CUT--------------------------

最近很郁闷,生活上的工作上的,就这样懒懒懒散散地目无光彩地苟活于世,最近看了一点 `C++`,有的地方头大,于是拿起老家伙 `asm` 写了点以前许诺过的东西,感觉还是 `ASM` 最有助于理解基本原理~~~不过 `C/C++` 给了我们另外的工具,虽然目标代码不够紧凑(理想状态),毕竟不需要我们每个人写内核,`C/C++` 可以提高生产能力.

`crack` 也一样,如果一些基本概念都不懂明白还谈什么 `crack`?昨天看精华 III 一些高手的文章,真是惭愧啊!

我的专业不是计算机或者以后永远也不会搞计算机,这些当也许只是业余爱好,永远不会放弃的业余爱好.

以后也许很长一段时间里面我会离开大家,毕竟,面临的还有生活.

感谢 `CCG`,`BCG` 的各位高手,`AfO` 的成员们,以及那些曾无私指导过以及给我生活动力的朋友们!

我要奋斗!

我以此激励自己也同样希望能够激励大家!

2002.1.28 深夜~~沉思中

----------------------------------------cut------------------------------------
