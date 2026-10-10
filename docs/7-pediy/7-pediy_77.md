

# SMC 例子

> 原文：[`www.pediy.com/kssd/pediy06/pediy6952.htm`](https://www.pediy.com/kssd/pediy06/pediy6952.htm)

; 从 MASM32 的例子里找到的，我写了中文的注释，希望对广大菜鸟同胞理解 SMC 有所帮助。   
;   
; #########################################################################   
;   
;   这是一个读写 PE 文件代码段的例程。在这个例子里有两个过程，第一个先被调用，   
;   第二个被读取并写入第一个过程里面，然后第一个过程被再次调用，前后的代码   
;   已经改变了。   
;   链接时应把参数设为"/section:.text,RWE",也就是说把代码段设为可读可写可执   
;   行。（非常重要，不然您就等着"非法操作"吧。）   
;   
; #########################################################################   
  
      .386   
      .model flat, stdcall   
      option casemap :none   ; case sensitive   
  
; #########################################################################   
  
      include masm32includewindows.inc   
      include masm32includeuser32.inc   
      include masm32includekernel32.inc   
  
      includelib masm32libuser32.lib   
      includelib masm32libkernel32.lib   
  
      CalledProc PROTO         ;过程 1   
      ReplaceMentProc  PROTO   ;过程 2   
  
; #########################################################################   
  
    .data   
        lnth1       dd 0   
        szDlgTitle  db "SMC example",0   
        Phony       db "This is 1st call of proc",0   
        Replc       db "This is 2nd call of proc",0   
        ttl1        db "Original Code",0   
        ttl2        db "Replacement Code",0   
  
; #########################################################################   
       
    .code   
  
start:   
  
    invoke CalledProc   ; 调用第一个过程，这时的代码还是原来的，因此显示 This is 1st call of proc   
       
    lea eax, rpEnd      ; 找找 rpEnd 在哪里，原来是在第二个过程末尾   
    lea edx, rpStart    ; 第二个过程的开始处   
    sub eax, edx        ; 这两个 LABEL 的偏移量相减是什么呢   
    mov lnth1, eax      ; 得到了第二个过程的代码长度，放在 lnth1   
  
    lea esi, rpStart    ; rpStart，这是第二个过程的开始处 ->ESI   
    lea edi, ppStart    ; ppStart，这是第一个过程的开始处 ->EDI   
    mov ecx, lnth1      ; 把长度放在 ECX   
    rep movsb           ; 这个指令的作用是把 ECX 个字节从 EDI 处搬到 ESI 处（大概就是这意思），详情请查手册。   
                        ; 执行完后会怎么样呢，你一定猜到了，这时过程 1 实际上已经是过程 2 的内容了   
  
    invoke CalledProc   ; 再调用第一个过程，结果呢，显示出来的是 This is 2st call of proc   
  
    invoke ExitProcess,0; 退出   
  
; #########################################################################   
  
CalledProc proc   
  
    ppStart::   ;两个冒号表示这个 LABEL 是全局的，因为默认时 LABEL 只能在本过程内使用   
  
    invoke MessageBox,0,ADDR Phony,ADDR ttl1,MB\_OK  ;对话框显示 This is 1st call of proc   
  
    ppEnd::   
  
  ; ------------------------------------------------------   
  ; 下面的 NOP 是为了给复制过来的过程 2 留下地方，每个 NOP 可以占一个字节   
  ; ------------------------------------------------------   
  
    nop   
    nop   
    nop   
    nop   
    nop   
    nop   
    nop   
    nop   
  
    ret   
  
CalledProc endp   
  
; #########################################################################   
  
ReplaceMentProc proc   
  
    ; -----------------------------------   
    ; 这个过程 2 永远不会被执行，但它的代码   
    ; 却被复制了过去   
    ; -----------------------------------   
  
    rpStart::   
       
    push MB\_OK or MB\_ICONEXCLAMATION   
    lea eax, ttl2   
    push eax   
    lea eax, Replc   
    push eax   
    push 0   
    lea eax, MessageBox   
    call eax              ;对话框显示 This is 2st call of proc   
  
    rpEnd::   
  
    ret   
  
ReplaceMentProc endp   
  
; #########################################################################   
  
end start 

  
 4dd466d0e1e0cc123afbe504da2462ca