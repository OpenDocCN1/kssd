

# intel 指令格式与长度反汇编引擎 ADE32 分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-318.htm`](https://www.pediy.com/kssd/pediy09/pediy09-318.htm)

> 我翻译的 29A#7 的一篇文章：  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*\*                          \*\*\*  
>    \*\*  INTEL INSTRUCTION FORMAT  \*\*  
>    \*\*\*                          \*\*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>      
>    intel 指令具有特有的格式：  
>   
>   
>                          intel 指令格式  
>   
> +-------------+--------+----------+---------+--------------+------------+  
> ; instruction ; opcode ;  ModR/M  ;   SIB   ; Displacement ; Immediate  ;  
> ;   prefixe   ;        ;          ;         ;              ;            ;  
> +-------------+--------+----------+---------+--------------+------------+  
>     前缀         操作码  （可选）    （可选）   地址偏移(可选) 立即数(可选)  
>   
>                1 or 2    1 字节      1 字节     1,2,4 字节      1,2,4 字节  
>                字节                            或者没有       或者没有  
>                               /         \\      
>                              /           \\     
>                             /             \\  
>                            /               \\  
>                           /                 \\  
>                          /                   \\  
>              7   6 5      3 2   0         7     6 5     3 2    0  
>             +-----+--------+-----+       +-------+-------+------+  
>             ; Mod ;  Reg/  ; R/M ;       ; scale ; index ; base ;  
>             ;     ; opcode ;     ;       ;       ;       ;      ;  
>             +-----+--------+-----+       +-------+-------+------+  
>   
>   
>    Intel 指令在长度上可以不同, 但是它们都同样具有以上的 6 组  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*\*                     \*\*\*  
>    \*\*       指令前缀        \*\*  
>    \*\*\*                     \*\*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>   
>    指令可以有（或者没有) 0,1,2,3 or 4 个前缀，每个前缀占用一个字节  
>    包含以下五种类型:  
>   
>    - 段寄存器(segment)前缀:指定如下段  2E 36 3E 26 64 65  
>   
>    - 操作数长度(Operand-Size)前缀 : 可以改变操作数长度 66  
>   
>    - 地址长度(Address-Size)前缀 : 可以改变地址长度 67  
>   
>    - 重复(REP/REPNE)前缀       : F3 F2  
>   
>    - 总线加锁(LOCK)前缀     : 控制处理器总线  F0  
>    +----------------+  
>    段寄存器前缀  
>    +----------------+  
>   
>    段寄存器前缀改变指令的默认段，默认段为 DS：  
>    2EH : CS  segment override prefix.  
>    36H : SS  segment override prefix.  
>    3EH : DS  segment override prefix.  
>    26H : ES  segment override prefix.  
>    64H : FS  segment override prefix.  
>    65H : GS  segment override prefix.  
>   
>   
>    expl:     8B00    MOV EAX,DWORD PTR DS:\[EAX\]   (none prefix)  
>           2E:8B00    MOV EAX,DWORD PTR CS:\[EAX\]  
>              8B00    MOV EAX,DWORD PTR DS:\[EAX\]   (none prefix)  
>           36:8B00    MOV EAX,DWORD PTR SS:\[EAX\]  
>   
>    +---------------------+  
>    操作数长度前缀  
>    +---------------------+  
>   
> 该前缀允许一个程序在 16 /32 位 操作数长度之间转换(默认为 32 位）  
> 如果指定前缀 66H，则转换为 16 位  
>    Quick example(in a win32 environement):  
>   
>       89 C0   MOV EAX,EAX   (none prefix)  
>    66 89 C0   MOV AX, AX    (prefix=66h)  
>   
>    +---------------------+  
>    ; 地址长度前缀 ;  
>    +---------------------+  
>   
> 同上，WIN32 编程，该前缀对我们没有用处  
>   
>    expl:     8B00   MOV EAX,DWORD PTR DS:\[EAX\]     32bits 寻址模式  
>           67:8B00   MOV EAX,DWORD PTR DS:\[BX+SI\]   16bits 寻址模式  
>   
>    +--------------------+  
>    ; 重复(REP/REPNE)前缀;  
>    +--------------------+  
>   
>          AD         LODS DWORD PTR DS:\[ESI\]  
>       F3 AD   REP   LODS DWORD PTR DS:\[ESI\]  
>       F2 AD   REPNE LODS DWORD PTR DS:\[ESI\]  
>    3E F2 AD   REPNE LODS DWORD PTR DS:\[ESI\]  
>   
>    +-----------------+  
>    ; Bus LOCK Prefix ;  
>    +-----------------+  
>   
>    No use for us...Execpt if you want to know all the mystery of intel instruction  
>    format in order to disasm the host, find a good place for the virus inside  
>    the disassembled host code and reassembly all the infected file...  
>   
>   
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*                       \*\*  
>    \*\*  操作码(OP CODE)        \*\*  
>    \*\*                       \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>    +------------------+  
>    ; 1 字节操作码 ;  
>    +------------------+  
>   
>    在 intel 文档你能找到这样的描述  
>    PUSH REG 指令:  
>      
>    PUSH REG <------> 01010reg   (where 'reg' are 3 bits)  
>    PUSH EAX  --->  50h  --->  01010000   ---> 01010 000  
>                                                 /      \\  
>                                                /        \\  
>                                               /          \\  
>                                     bits   7 6 5 4 3   2 1 0  
>                                          +-----------+--------+  
>                                          ;  OPCODE   ;REGISTER;  
>                                          +-----------+--------+  
>                                          ; 0 1 0 1 0 ; 0 0 0  ;  
>                                             push         eax  
>                                          +-----------+--------+  
>   
>         Little Opcode Table:              Register Table:  
>    
>                                          REG 8bit 16bit 32bit  
>         01000reg : INC  REG              000 : AL : AX : EAX  
>         01001reg : DEC  REG              001 : CL : CX : ECX  
>         01010reg : PUSH REG              010 : DL : DX : EDX  
>         01011reg : POP  REG              011 : BL : BX : EBX  
>         10010reg : XCHG EAX,REG          100 : AH : SP : ESP  
>         10111reg : MOV  REG,IMM32        101 : CH : BP : EBP  
>                                          110 : DH : SI : ESI  
>                                          111 : BH : DI : EDI  
>   
>    One another (fun) example:  
>   
>    90h --->  10010 000 ---> XCHG EAX,EAX  (NOP)  
>   
>    只有指令 inc reg, dec reg, push reg, pop reg, xchg eax,reg, mov reg,imm32  
>    可以这样编码!  
>   
>    其他方式编码:  
>   
>    89C1h ---> 1000100111000001b  ---> mov ecx,eax  
>   
>    仍旧为一字节操作码: C1h 为 \[ModR/M\] 字段!  
>   
>                89h      C1h  
>             10001001 11000001   
>              \[code\]  \[ModR/M\]  
>               /          \\  
>              /            \\  
>             /              \\  
>    +----------------+   +------------------+  
>    ; instr  ; d ; w ;   ; Mod; REG1 ; REG2 ;  
>    +--------+---+---;   +----+------+------+  
>    ; 100010 ; 0 ; 1 ;   ; 11 ; 000  ; 001  ;  
>    +--------+---+---+   +----+------+------+  
>   
>  instr  : 操作码指令  
>   
>  (d)bit: 如果 (d)=0 顺序为 REG2-REG1  
>          如果 (d)=1 顺序为 REG1-REG2  
>   
>  (w)bit: if (w)=0 we are in 8  bits mode in 32bits environement(Win32)  
>          if (w)=1 we are in 32 bits mode in 32bits environement(Win32)  
>   
>          if (w)=0 we are in 8  bits mode in 16bits environement  
>          if (w)=1 we are in 16 bits mode in 16bits environement  
>   
>   Mod  : 以后再讲  
>   
>                             ############################  
>                             # example for the (d) bit: #  
>                             ############################  
>                               (in 32bits environement)  
>   
>        \[code\]             \[ModR/M\]            \[code\]             \[ModR/M\]  
> +----------------;;------------------+  +----------------;;------------------+  
> ; instr  ; d ; w ;; Mod; REG1 ; REG2 ;  ; instr  ; d ; w ;; Mod; REG1 ; REG2 ;  
> +--------+---+---;;----+------+------+  +--------+---+---;;----+------+------+  
> ; 100010 ; 0 ; 1 ;; 11 ; 000  ; 001  ;  ; 100010 ; 1 ; 1 ;; 11 ; 000  ; 001  ;  
> +--------+---+---;;----+------+------+  +--------+---+---;;----+------+------+  
>     !      !   \\          EAX    ECX        !      !   \\          EAX    ECX  
>     !      !    !       or AX  or CX        !      !    !       or AX  or CX  
>    \\!/     !   \\!/      or AL  or CL       \\!/     !   \\!/      or AL  or CL  
>     '      !    '                           '      !    '  
>    MOV     !   32bits                      MOV     !   32bits  
>           \\!/                                     \\!/  
>            '  
>         the order                               the order  
>       is REG2-REG1                             is REG1-REG2  
>  \\\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_/      \\\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_/  
>                  \\/                                        \\/  
>              MOV ECX,EAX                              MOV EAX,ECX  
>   
>                             ############################  
>                             # example for the (w) bit: #  
>                             ############################  
>   
>   
>        \[code\]             \[ModR/M\]             \[code\]             \[ModR/M\]  
> +----------------;;------------------+  +----------------;;------------------+  
> ; instr  ; d ; w ;; Mod; REG1 ; REG2 ;  ; instr  ; d ; w ;; Mod; REG1 ; REG2 ;  
> +--------+---+---;;----+------+------+  +--------+---+---;;----+------+------+  
> ; 100010 ; 0 ; 1 ;; 11 ; 000  ; 001  ;  ; 100010 ; 0 ; 0 ;; 11 ; 000  ; 001  ;  
> +--------+---+---;;----+------+------+  +--------+---+---;;----+------+------+  
>     !      !   \\          EAX    ECX        !      !   \\          EAX    ECX  
>     !      !    !       or AX  or CX        !      !    !       or AX  or CX  
>    \\!/     !   \\!/      or AL  or CL       \\!/     !   \\!/      or AL  or CL  
>     '      !    '                           '      !    '  
>    MOV     !   32bits                      MOV     !   8bits  
>           \\!/                                     \\!/  
>            '  
>         the order                               the order  
>       is REG2-REG1                             is REG2-REG1  
>  \\\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_/      \\\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_  \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_/  
>                  \\/                                        \\/  
>              MOV ECX,EAX                              MOV CL,AL  
>   
>   
>                 examples with 66h prefix:  
>   
>   ;-------+-----------+----------+   ;-------+-----------+----------+  
>   ; Pref. ;   \[code\]  ; \[ModR/M\] ;   ; Pref. ;   \[code\]  ; \[ModR/M\] ;  
>   +-------+-----------+----------+   +-------+-----------+----------+  
>   ; 66h   ;   89h     ;  C1h     ;   ;       ;   88h     ;  C1h     ;  
>   +-------+-----------+----------+   +-------+-----------+----------+  
>   ; 66h   ; 10001001  ; 11000001 ;   ;       ; 10001000  ; 11000001 ;   
>   ;       ;   (w)=1   ;          ;   ;       ;    (w)=0  ;          ;  
>   +-------+-----------+----------+   +-------+-----------+----------+  
>                MOV CX,AX                          MOV CL,AL  
>   
>   
>    Little Instruction Opcode table:  
>   
>        BINARY       OPCODE  
>       -----------------------  
>       000010dw    OR  REG,REG  
>       001000dw    AND REG,REG  
>       001010dw    SUB REG,REG  
>       001100dw    XOR REG,REG  
>       001110dw    CMP REG,REG  
>       100000dw    ADD REG,REG  
>       100010dw    MOV REG,REG  
>    +-------------------+  
>    ; 双字节操作码 ;  
>    +-------------------+  
>    expl: 0f22 c0    mov cr0,eax  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*              \*\*  
>    \*\*    ModR/M    \*\*  
>    \*\*              \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>   
>    \[ModR/M\] 告诉处理器哪一个寄存器或者内存地址被使用  
>    \[ModRM\] 8 位被分为如下 3 组  
>    groups (2![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif")3).  
>   
>                          7   6 5      3 2   0        
>                         +-----+--------+-----+  
>                         ; Mod ;  Reg/  ; R/M ;  
>                         ;     ; opcode ;     ;  
>                         +-----+--------+-----+  
>   
>       \* \[Mod\]:  
>    
>         00  : 内存地址(memory address)                   expl: eax, \[eax\]        
>         01  : memory address+1 字节地址偏移量             expl: \[eax+00\]    
>         10  : memory address+一个双字地址偏移量          expl: \[eax+00000000\]  
>         11  : 两个操作数都为内存                         expl: eax,eax  
>   
>   
>       \* \[Reg/opcode\]:  
>   
>         这个字段被认为是代码扩展字段或者作为寄存器字段，处理器知道哪一个右侧编码相对这个字段  
>   
>          -如果为代码扩展字段(Code extension):  
>   
>           有的指令需要一个操作数，而有的需要两个操作数  
>           ADD: instruction requires 2-Operands     (add eax, eax)  
>           MUL: instruction requires only 1-Operand (mul eax)  
>   
>           如果我们使用单操作数  
>           中间的 3-Bits \[Reg/opcode\]field 就是代码扩展字段.  
>             
>           example:  
>    
>           F7D0 : 11110111 11 010 000   not eax  
>           F7E0 : 11110111 11 100 000   mul eax  
>           F7F0 : 11110111 11 110 000   div eax  
>   
>           they all have 0xF7 for the \[code\] byte, only \[Reg/opcode\] is different.  
>           (000 is for the reg eax!)  
>             
>          -如果为寄存器字段:  
>   
>           example with the hex instruction:    
>   
>          \[code\] \[MorR/M\]  
>            39    d0  
>            CMP     \\\_  
>             \\        \\  
>              \\        \\  
>               \\          7 6   5 4 3    2 1 0   bits  
>                \\        +-----+--------+-------+  
>                 \\       ; Mod ;  Reg/  ; R/M   ;  
>                  \\      ;     ; opcode ;       ;  
>                   |     +-----+--------+-------+  
>                    \\    ; 1 1 ; 0 1 0  ; 0 0 0 ;  
>                     \\   +-----+--------+-------+  
>                       CMP        EDX      EAX  
>   
>        So 39d0h is CMP EAX,EDX  (because of the (d) bits in \[code\] field)  
>   
>   
>        \* \[R/M\]  
>   
>         依赖于(Mode) Bits in the ModRM 字节:   
>   
>   
>   - If \[Mod\]=00 and \[R/M\]=101 : 没有寄存器用于计算地址，取而代之为一个双字在\[ModR/M\]字段后面  
>                                 expl: 3305 563412   xor eax,\[12345678h\]  
>   
>   - If \[Mod\]=00 and \[R/M\]=100 : \[ModR/M\]后面含有一个 SIB 字节  
>   
>   - If \[Mod\]=01 and \[R/M\]=100 : \[ModR/M\]后面含有一个 SIB 字节  
>   
>   - If \[Mod\]=10 and \[R/M\]=100 : \[ModR/M\]后面含有一个 SIB 字节  
>   - If \[Mod\]=11               : exmpl: 89:C0 =  mov eax, eax   
>    
>    (P.S.: I've not verify this part, if \[Mod\]=xx then it means....)  
>    (Let's make your own mind about that)  
>   
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*              \*\*  
>    \*\*     SIB      \*\*  
>    \*\*              \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>    SIB  代表(Scale : Index :Base)全称.  
>    一般格式为( Scale \* Index + Base ).   
>   
>                           7 6   5 4 3    2 1 0   bits  
>                         +-----+--------+-------+  
>                         ;Scale;  Index ; Base  ;  
>                         +-----+--------+-------+  
>                          Scale \* Index + Base   
>   
>    \* Scale : 被认为是乘数(of the index register)  
>   
>    00:xxx:xxx = 2⁰ = 1  (\*1)  
>    01:xxx:xxx = 2¹ = 2  (\*2)  
>    10:xxx:xxx = 2² = 4  (\*4)  
>    11:xxx:xxx = 2³ = 8  (\*8)  
>   
>    \* Index : 是一个寄存器(除了 esp)  
>    \* Base  : 基础寄存器（Base Register）？  
>   
>    expl:  
>   
>         \[SIB\]      |  
>    ----------------+----------------------------  
>    00 : 000 : 001  |  mov reg, \[1 \* eax + ecx\]  
>    01 : 001 : 010  |  mov reg, \[2 \* ecx + edx\]  
>    01 : 110 : 111  |  mov reg, \[2 \* edi + esi\]  
>    10 : 010 : 011  |  mov reg, \[4 \* edx + ebx\]  
>    11 : 011 : 000  |  mov reg, \[8 \* eax + ebx\]  
>   
>   
>    - 如果索引寄存器为 ESP，则 index 被忽略，在这个例子里 scale 也被忽略，只有 base register 被使用计算地址  
>    - 如果我们需要如下编码(add reg, \[esp\]) 它可以使用 SIB 字节，但是它被编码成这样  
>      (add reg, \[esp + DISPLACEMENT\])  
>   
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*              \*\*  
>    \*\* DISPLACEMENT \*\*  
>    \*\*              \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>    Just a quik example with the instruction 8b bd 78563412 :  
>   
>                          8b      bd    78563412  
>                        \[code\] \[MorR/M\]  \[???\]  
>                         /         \\  
>                        /           \\  
>                       /             \\  
>                      /               \\  
>    +----------------;               +-----+--------+-------+  
>    ; instr  ; d ; w ;               ; Mod ;  Reg/  ; R/M   ;  
>    +--------+---+---;               ;     ; opcode ;       ;  
>    ; 100010 ; 1 ; 1 ;               +-----+--------+-------+  
>    +--------+---+---;               ; 1 0 ; 1 1 1  ; 1 0 1 ;  
>    (d)=0 : the order is             +-----+--------+-------+  
>            REG1-REG2                         EDI      EBP  
>    (w)=1 : 32 bits mode                    or code  
>    instr : MOV r32,r/m32                  extension  
>   
>   
> \[code\]       = 8b        : opcode  
>                           ---> MOV r32,r/m32  
>   
> \[Mod\]        = 10        : memory address with 1 dword displacement   
>                           ---> MOV REG,\[REG+DWORD\]          
>   
> \[Reg/opcode\] = 111       : this instruction don't need an code extension  
>                            so it is a register  
>                           ---> MOV EDI,\[EBP+DWORD\]  
>   
> \[R/M\]        = 101       : there is no \[SIB\] after the byte\[ModR/M\]  
>    
> \[78563412\]   = 12345678h : it is the deplacement  
>                           ---> MOV EDI,\[EBP+12345678h\]  
>   
>   
> So the instruction 8b bd 78563412 is mov edi,\[ebp+12345678h\]  
>   
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*             \*\*  
>    \*\*  IMMEDIATE  \*\*  
>    \*\*             \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>   expl:   05h          is the opcode for ADD EAX,imm32  
>           05h 00000010 is the instruction for add eax, 10000000  
>   
>   
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>    \*\*                        \*\*  
>    \*\*  ALGO OF DISASSEMBLY   \*\*  
>    \*\*                        \*\*  
>    \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>        !  
>       \\!/  
>  +-----'-------+  
>  ; Read 1 byte ; <---------  
>  +-------------+           \\  
>        !                    \\               +------------------+  
>       \\!/                    --------<----- ; Convert to ASCII ;  
>  +-----'------------+                       +-------.----------+  
>  ; Check for prefix ;                              /!\\  
>  +------------------+                               !  
>        !                                            !  
>       \\!/                                   +-----------------+  
>  ;-----'----------------+                   ; \[IMMEDIATE\]     ;  
>  ; Decode \[code\] byte(s);                   +-----------------+  
>  +----------------------+                   ; \[DISPLACEMENT\]  ;  
>        !                                    +-----------------+  
>       \\!/                                   ; decode \[SIB\]    ;  
>  +-----'-----------------+                  +-----------------+  
>  ; JMP or CALL opcodes ? ; -------->------> ; decode \[ModR/M\] ;  
>  +-----------------------+                  +-----------------+  
> 下面是我注释的 ADE32 的代码，很多语义表示不清楚，大家凑合看吧，偶水平不行啊![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
>   
> // ADE32 2.03c -- advanced 16/32-bit opcode assembler/disassembler engine  
>   
> // this stuff is used to get instruction length and parse it into  
> // prefix, opcode, modregr/m, address, immediate, etc.  
> //指令特征  
> #define C\_ERROR   0xFFFFFFFF  
> #define C\_ADDR1   0x00000001 //操作码中地址大小的位字段(字节)  
> #define C\_ADDR2   0x00000002  
> #define C\_ADDR4   0x00000004 //(双字)  
> #define C\_LOCK    0x00000008 //加锁前缀  
> #define C\_67      0x00000010 //地址大小修饰前缀(16/32 位)  
> #define C\_66      0x00000020 //操作数大小修饰前缀(16/32 位)  
> #define C\_REP     0x00000040 //重复前缀  
> #define C\_SEG     0x00000080 //段寄存器前缀  
> #define C\_ANYPREFIX  (C\_66+C\_67+C\_LOCK+C\_REP+C\_SEG)  
> #define C\_DATA1   0x00000100 //操作码中数据大小的位字段  
> #define C\_DATA2   0x00000200  
> #define C\_DATA4   0x00000400  
> #define C\_SIB     0x00000800 //SIB 字节  
> #define C\_ADDR67  0x00001000 //地址字节数为 disasm\_defaddr  
> #define C\_DATA66  0x00002000 //数据字节数为 disasm\_defdata  
> #define C\_MODRM   0x00004000 //MODRM 字节  
> #define C\_BAD     0x00008000  
> #define C\_OPCODE2 0x00010000 //操作码第二个字节  
> #define C\_REL     0x00020000 // 这是跳转指令 jxx 或者 call  
> #define C\_STOP    0x00040000 // 这是回跳指令，ret 或者 jmp  
> //指令信息  
> struct disasm\_struct  
> {  
>   BYTE  disasm\_defaddr;         // 00  
>   BYTE  disasm\_defdata;         // 01  
>   DWORD disasm\_len;             // 02 03 04 05  
>   DWORD disasm\_flag;            // 06 07 08 09  
>   DWORD disasm\_addrsize;        // 0A 0B 0C 0D  
>   DWORD disasm\_datasize;        // 0E 0F 10 11  
>   BYTE  disasm\_rep;             // 12  
>   BYTE  disasm\_seg;             // 13  
>   BYTE  disasm\_opcode;          // 14  
>   BYTE  disasm\_opcode2;         // 15  
>   BYTE  disasm\_modrm;           // 16  
>   BYTE  disasm\_sib;             // 17  
>   //地址  
>   union  
>   {  
>   BYTE  disasm\_addr\_b\[8\];       // 18 19 1A 1B  1C 1D 1E 1F  
>   WORD  disasm\_addr\_w\[4\];  
>   DWORD disasm\_addr\_d\[2\];  
>   char  disasm\_addr\_c\[8\];  
>   short disasm\_addr\_s\[4\];  
>   long  disasm\_addr\_l\[2\];  
>   };  
>   //数据  
>   union  
>   {  
>   BYTE  disasm\_data\_b\[8\];       // 20 21 22 23  24 25 26 27  
>   WORD  disasm\_data\_w\[4\];  
>   DWORD disasm\_data\_d\[2\];  
>   char  disasm\_data\_c\[8\];  
>   short disasm\_data\_s\[4\];  
>   long  disasm\_data\_l\[2\];  
>   };  
> }; // disasm\_struct  
> //按操作码从 0x00 开始排列，每一行代表一个操作  
> //码对应的指令特征的集合  
> DWORD ade32\_table\[512\] = {  
> /\* 00 \*/  C\_MODRM,  
> /\* 01 \*/  C\_MODRM,  
> /\* 02 \*/  C\_MODRM,  
> /\* 03 \*/  C\_MODRM,  
> /\* 04 \*/  C\_DATA1,  
> /\* 05 \*/  C\_DATA66,  
> /\* 06 \*/  C\_BAD,  
> /\* 07 \*/  C\_BAD,  
> /\* 08 \*/  C\_MODRM,  
> /\* 09 \*/  C\_MODRM,  
> /\* 0A \*/  C\_MODRM,  
> /\* 0B \*/  C\_MODRM,  
> /\* 0C \*/  C\_DATA1,  
> /\* 0D \*/  C\_DATA66,  
> /\* 0E \*/  C\_BAD,  
> /\* 0F \*/  C\_OPCODE2,  
> /\* 10 \*/  C\_MODRM+C\_BAD,  
> /\* 11 \*/  C\_MODRM,  
> /\* 12 \*/  C\_MODRM+C\_BAD,  
> /\* 13 \*/  C\_MODRM,  
> /\* 14 \*/  C\_DATA1+C\_BAD,  
> /\* 15 \*/  C\_DATA66+C\_BAD,  
> /\* 16 \*/  C\_BAD,  
> /\* 17 \*/  C\_BAD,  
> /\* 18 \*/  C\_MODRM+C\_BAD,  
> /\* 19 \*/  C\_MODRM,  
> /\* 1A \*/  C\_MODRM,  
> /\* 1B \*/  C\_MODRM,  
> /\* 1C \*/  C\_DATA1+C\_BAD,  
> /\* 1D \*/  C\_DATA66+C\_BAD,  
> /\* 1E \*/  C\_BAD,  
> /\* 1F \*/  C\_BAD,  
> /\* 20 \*/  C\_MODRM,  
> /\* 21 \*/  C\_MODRM,  
> /\* 22 \*/  C\_MODRM,  
> /\* 23 \*/  C\_MODRM,  
> /\* 24 \*/  C\_DATA1,  
> /\* 25 \*/  C\_DATA66,  
> /\* 26 \*/  C\_SEG+C\_BAD,  
> /\* 27 \*/  C\_BAD,  
> /\* 28 \*/  C\_MODRM,  
> /\* 29 \*/  C\_MODRM,  
> /\* 2A \*/  C\_MODRM,  
> /\* 2B \*/  C\_MODRM,  
> /\* 2C \*/  C\_DATA1,  
> /\* 2D \*/  C\_DATA66,  
> /\* 2E \*/  C\_SEG+C\_BAD,  
> /\* 2F \*/  C\_BAD,  
> /\* 30 \*/  C\_MODRM,  
> /\* 31 \*/  C\_MODRM,  
> /\* 32 \*/  C\_MODRM,  
> /\* 33 \*/  C\_MODRM,  
> /\* 34 \*/  C\_DATA1,  
> /\* 35 \*/  C\_DATA66,  
> /\* 36 \*/  C\_SEG+C\_BAD,  
> /\* 37 \*/  C\_BAD,  
> /\* 38 \*/  C\_MODRM,  
> /\* 39 \*/  C\_MODRM,  
> /\* 3A \*/  C\_MODRM,  
> /\* 3B \*/  C\_MODRM,  
> /\* 3C \*/  C\_DATA1,  
> /\* 3D \*/  C\_DATA66,  
> /\* 3E \*/  C\_SEG+C\_BAD,  
> /\* 3F \*/  C\_BAD,  
> /\* 40 \*/  0,  
> /\* 41 \*/  0,  
> /\* 42 \*/  0,  
> /\* 43 \*/  0,  
> /\* 44 \*/  C\_BAD,  
> /\* 45 \*/  0,  
> /\* 46 \*/  0,  
> /\* 47 \*/  0,  
> /\* 48 \*/  0,  
> /\* 49 \*/  0,  
> /\* 4A \*/  0,  
> /\* 4B \*/  0,  
> /\* 4C \*/  C\_BAD,  
> /\* 4D \*/  0,  
> /\* 4E \*/  0,  
> /\* 4F \*/  0,  
> /\* 50 \*/  0,  
> /\* 51 \*/  0,  
> /\* 52 \*/  0,  
> /\* 53 \*/  0,  
> /\* 54 \*/  0,  
> /\* 55 \*/  0,  
> /\* 56 \*/  0,  
> /\* 57 \*/  0,  
> /\* 58 \*/  0,  
> /\* 59 \*/  0,  
> /\* 5A \*/  0,  
> /\* 5B \*/  0,  
> /\* 5C \*/  C\_BAD,  
> /\* 5D \*/  0,  
> /\* 5E \*/  0,  
> /\* 5F \*/  0,  
> /\* 60 \*/  C\_BAD,  
> /\* 61 \*/  C\_BAD,  
> /\* 62 \*/  C\_MODRM+C\_BAD,  
> /\* 63 \*/  C\_MODRM+C\_BAD,  
> /\* 64 \*/  C\_SEG,  
> /\* 65 \*/  C\_SEG+C\_BAD,  
> /\* 66 \*/  C\_66,  
> /\* 67 \*/  C\_67,  
> /\* 68 \*/  C\_DATA66,  
> /\* 69 \*/  C\_MODRM+C\_DATA66,  
> /\* 6A \*/  C\_DATA1,  
> /\* 6B \*/  C\_MODRM+C\_DATA1,  
> /\* 6C \*/  C\_BAD,  
> /\* 6D \*/  C\_BAD,  
> /\* 6E \*/  C\_BAD,  
> /\* 6F \*/  C\_BAD,  
> /\* 70 \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* 71 \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* 72 \*/  C\_DATA1+C\_REL,  
> /\* 73 \*/  C\_DATA1+C\_REL,  
> /\* 74 \*/  C\_DATA1+C\_REL,  
> /\* 75 \*/  C\_DATA1+C\_REL,  
> /\* 76 \*/  C\_DATA1+C\_REL,  
> /\* 77 \*/  C\_DATA1+C\_REL,  
> /\* 78 \*/  C\_DATA1+C\_REL,  
> /\* 79 \*/  C\_DATA1+C\_REL,  
> /\* 7A \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* 7B \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* 7C \*/  C\_DATA1+C\_REL,  
> /\* 7D \*/  C\_DATA1+C\_REL,  
> /\* 7E \*/  C\_DATA1+C\_REL,  
> /\* 7F \*/  C\_DATA1+C\_REL,  
> /\* 80 \*/  C\_MODRM+C\_DATA1,  
> /\* 81 \*/  C\_MODRM+C\_DATA66,  
> /\* 82 \*/  C\_MODRM+C\_DATA1+C\_BAD,  
> /\* 83 \*/  C\_MODRM+C\_DATA1,  
> /\* 84 \*/  C\_MODRM,  
> /\* 85 \*/  C\_MODRM,  
> /\* 86 \*/  C\_MODRM,  
> /\* 87 \*/  C\_MODRM,  
> /\* 88 \*/  C\_MODRM,  
> /\* 89 \*/  C\_MODRM,  
> /\* 8A \*/  C\_MODRM,  
> /\* 8B \*/  C\_MODRM,  
> /\* 8C \*/  C\_MODRM+C\_BAD,  
> /\* 8D \*/  C\_MODRM,  
> /\* 8E \*/  C\_MODRM+C\_BAD,  
> /\* 8F \*/  C\_MODRM,  
> /\* 90 \*/  0,  
> /\* 91 \*/  0,  
> /\* 92 \*/  0,  
> /\* 93 \*/  C\_BAD,  
> /\* 94 \*/  C\_BAD,  
> /\* 95 \*/  C\_BAD,  
> /\* 96 \*/  C\_BAD,  
> /\* 97 \*/  C\_BAD,  
> /\* 98 \*/  C\_BAD,  
> /\* 99 \*/  0,  
> /\* 9A \*/  C\_DATA66+C\_DATA2+C\_BAD,  
> /\* 9B \*/  0,  
> /\* 9C \*/  C\_BAD,  
> /\* 9D \*/  C\_BAD,  
> /\* 9E \*/  C\_BAD,  
> /\* 9F \*/  C\_BAD,  
> /\* A0 \*/  C\_ADDR67,  
> /\* A1 \*/  C\_ADDR67,  
> /\* A2 \*/  C\_ADDR67,  
> /\* A3 \*/  C\_ADDR67,  
> /\* A4 \*/  0,  
> /\* A5 \*/  0,  
> /\* A6 \*/  0,  
> /\* A7 \*/  0,  
> /\* A8 \*/  C\_DATA1,  
> /\* A9 \*/  C\_DATA66,  
> /\* AA \*/  0,  
> /\* AB \*/  0,  
> /\* AC \*/  0,  
> /\* AD \*/  C\_BAD,  
> /\* AE \*/  0,  
> /\* AF \*/  C\_BAD,  
> /\* B0 \*/  C\_DATA1,  
> /\* B1 \*/  C\_DATA1,  
> /\* B2 \*/  C\_DATA1,  
> /\* B3 \*/  C\_DATA1,  
> /\* B4 \*/  C\_DATA1,  
> /\* B5 \*/  C\_DATA1,  
> /\* B6 \*/  C\_DATA1+C\_BAD,  
> /\* B7 \*/  C\_DATA1+C\_BAD,  
> /\* B8 \*/  C\_DATA66,  
> /\* B9 \*/  C\_DATA66,  
> /\* BA \*/  C\_DATA66,  
> /\* BB \*/  C\_DATA66,  
> /\* BC \*/  C\_DATA66+C\_BAD,  
> /\* BD \*/  C\_DATA66,  
> /\* BE \*/  C\_DATA66,  
> /\* BF \*/  C\_DATA66,  
> /\* C0 \*/  C\_MODRM+C\_DATA1,  
> /\* C1 \*/  C\_MODRM+C\_DATA1,  
> /\* C2 \*/  C\_DATA2+C\_STOP,  
> /\* C3 \*/  C\_STOP,  
> /\* C4 \*/  C\_MODRM+C\_BAD,  
> /\* C5 \*/  C\_MODRM+C\_BAD,  
> /\* C6 \*/  C\_MODRM+C\_DATA1,  
> /\* C7 \*/  C\_MODRM+C\_DATA66,  
> /\* C8 \*/  C\_DATA2+C\_DATA1,  
> /\* C9 \*/  0,  
> /\* CA \*/  C\_DATA2+C\_STOP+C\_BAD,  
> /\* CB \*/  C\_STOP+C\_BAD,  
> /\* CC \*/  C\_BAD,  
> /\* CD \*/  C\_DATA1,  
> /\* CE \*/  C\_BAD,  
> /\* CF \*/  C\_STOP+C\_BAD,  
> /\* D0 \*/  C\_MODRM,  
> /\* D1 \*/  C\_MODRM,  
> /\* D2 \*/  C\_MODRM,  
> /\* D3 \*/  C\_MODRM,  
> /\* D4 \*/  C\_DATA1+C\_BAD,  
> /\* D5 \*/  C\_DATA1+C\_BAD,  
> /\* D6 \*/  C\_BAD,  
> /\* D7 \*/  C\_BAD,  
> /\* D8 \*/  C\_MODRM,  
> /\* D9 \*/  C\_MODRM,  
> /\* DA \*/  C\_MODRM,  
> /\* DB \*/  C\_MODRM,  
> /\* DC \*/  C\_MODRM,  
> /\* DD \*/  C\_MODRM,  
> /\* DE \*/  C\_MODRM,  
> /\* DF \*/  C\_MODRM,  
> /\* E0 \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* E1 \*/  C\_DATA1+C\_REL+C\_BAD,  
> /\* E2 \*/  C\_DATA1+C\_REL,  
> /\* E3 \*/  C\_DATA1+C\_REL,  
> /\* E4 \*/  C\_DATA1+C\_BAD,  
> /\* E5 \*/  C\_DATA1+C\_BAD,  
> /\* E6 \*/  C\_DATA1+C\_BAD,  
> /\* E7 \*/  C\_DATA1+C\_BAD,  
> /\* E8 \*/  C\_DATA66+C\_REL,  
> /\* E9 \*/  C\_DATA66+C\_REL+C\_STOP,  
> /\* EA \*/  C\_DATA66+C\_DATA2+C\_BAD,  
> /\* EB \*/  C\_DATA1+C\_REL+C\_STOP,  
> /\* EC \*/  C\_BAD,  
> /\* ED \*/  C\_BAD,  
> /\* EE \*/  C\_BAD,  
> /\* EF \*/  C\_BAD,  
> /\* F0 \*/  C\_LOCK+C\_BAD,  
> /\* F1 \*/  C\_BAD,  
> /\* F2 \*/  C\_REP,  
> /\* F3 \*/  C\_REP,  
> /\* F4 \*/  C\_BAD,  
> /\* F5 \*/  C\_BAD,  
> /\* F6 \*/  C\_MODRM,  
> /\* F7 \*/  C\_MODRM,  
> /\* F8 \*/  0,  
> /\* F9 \*/  0,  
> /\* FA \*/  C\_BAD,  
> /\* FB \*/  C\_BAD,  
> /\* FC \*/  0,  
> /\* FD \*/  0,  
> /\* FE \*/  C\_MODRM,  
> /\* FF \*/  C\_MODRM,  
> /\* 00 \*/  C\_MODRM,  
> /\* 01 \*/  C\_MODRM,  
> /\* 02 \*/  C\_MODRM,  
> /\* 03 \*/  C\_MODRM,  
> /\* 04 \*/  C\_ERROR,  
> /\* 05 \*/  C\_ERROR,  
> /\* 06 \*/  0,  
> /\* 07 \*/  C\_ERROR,  
> /\* 08 \*/  0,  
> /\* 09 \*/  0,  
> /\* 0A \*/  0,  
> /\* 0B \*/  0,  
> /\* 0C \*/  C\_ERROR,  
> /\* 0D \*/  C\_ERROR,  
> /\* 0E \*/  C\_ERROR,  
> /\* 0F \*/  C\_ERROR,  
> /\* 10 \*/  C\_ERROR,  
> /\* 11 \*/  C\_ERROR,  
> /\* 12 \*/  C\_ERROR,  
> /\* 13 \*/  C\_ERROR,  
> /\* 14 \*/  C\_ERROR,  
> /\* 15 \*/  C\_ERROR,  
> /\* 16 \*/  C\_ERROR,  
> /\* 17 \*/  C\_ERROR,  
> /\* 18 \*/  C\_ERROR,  
> /\* 19 \*/  C\_ERROR,  
> /\* 1A \*/  C\_ERROR,  
> /\* 1B \*/  C\_ERROR,  
> /\* 1C \*/  C\_ERROR,  
> /\* 1D \*/  C\_ERROR,  
> /\* 1E \*/  C\_ERROR,  
> /\* 1F \*/  C\_ERROR,  
> /\* 20 \*/  C\_ERROR,  
> /\* 21 \*/  C\_ERROR,  
> /\* 22 \*/  C\_ERROR,  
> /\* 23 \*/  C\_ERROR,  
> /\* 24 \*/  C\_ERROR,  
> /\* 25 \*/  C\_ERROR,  
> /\* 26 \*/  C\_ERROR,  
> /\* 27 \*/  C\_ERROR,  
> /\* 28 \*/  C\_ERROR,  
> /\* 29 \*/  C\_ERROR,  
> /\* 2A \*/  C\_ERROR,  
> /\* 2B \*/  C\_ERROR,  
> /\* 2C \*/  C\_ERROR,  
> /\* 2D \*/  C\_ERROR,  
> /\* 2E \*/  C\_ERROR,  
> /\* 2F \*/  C\_ERROR,  
> /\* 30 \*/  C\_ERROR,  
> /\* 31 \*/  C\_ERROR,  
> /\* 32 \*/  C\_ERROR,  
> /\* 33 \*/  C\_ERROR,  
> /\* 34 \*/  C\_ERROR,  
> /\* 35 \*/  C\_ERROR,  
> /\* 36 \*/  C\_ERROR,  
> /\* 37 \*/  C\_ERROR,  
> /\* 38 \*/  C\_ERROR,  
> /\* 39 \*/  C\_ERROR,  
> /\* 3A \*/  C\_ERROR,  
> /\* 3B \*/  C\_ERROR,  
> /\* 3C \*/  C\_ERROR,  
> /\* 3D \*/  C\_ERROR,  
> /\* 3E \*/  C\_ERROR,  
> /\* 3F \*/  C\_ERROR,  
> /\* 40 \*/  C\_MODRM,  
> /\* 41 \*/  C\_MODRM,  
> /\* 42 \*/  C\_MODRM,  
> /\* 43 \*/  C\_MODRM,  
> /\* 44 \*/  C\_MODRM,  
> /\* 45 \*/  C\_MODRM,  
> /\* 46 \*/  C\_MODRM,  
> /\* 47 \*/  C\_MODRM,  
> /\* 48 \*/  C\_MODRM,  
> /\* 49 \*/  C\_MODRM,  
> /\* 4A \*/  C\_MODRM,  
> /\* 4B \*/  C\_MODRM,  
> /\* 4C \*/  C\_MODRM,  
> /\* 4D \*/  C\_MODRM,  
> /\* 4E \*/  C\_MODRM,  
> /\* 4F \*/  C\_MODRM,  
> /\* 50 \*/  C\_ERROR,  
> /\* 51 \*/  C\_ERROR,  
> /\* 52 \*/  C\_ERROR,  
> /\* 53 \*/  C\_ERROR,  
> /\* 54 \*/  C\_ERROR,  
> /\* 55 \*/  C\_ERROR,  
> /\* 56 \*/  C\_ERROR,  
> /\* 57 \*/  C\_ERROR,  
> /\* 58 \*/  C\_ERROR,  
> /\* 59 \*/  C\_ERROR,  
> /\* 5A \*/  C\_ERROR,  
> /\* 5B \*/  C\_ERROR,  
> /\* 5C \*/  C\_ERROR,  
> /\* 5D \*/  C\_ERROR,  
> /\* 5E \*/  C\_ERROR,  
> /\* 5F \*/  C\_ERROR,  
> /\* 60 \*/  C\_ERROR,  
> /\* 61 \*/  C\_ERROR,  
> /\* 62 \*/  C\_ERROR,  
> /\* 63 \*/  C\_ERROR,  
> /\* 64 \*/  C\_ERROR,  
> /\* 65 \*/  C\_ERROR,  
> /\* 66 \*/  C\_ERROR,  
> /\* 67 \*/  C\_ERROR,  
> /\* 68 \*/  C\_ERROR,  
> /\* 69 \*/  C\_ERROR,  
> /\* 6A \*/  C\_ERROR,  
> /\* 6B \*/  C\_ERROR,  
> /\* 6C \*/  C\_ERROR,  
> /\* 6D \*/  C\_ERROR,  
> /\* 6E \*/  C\_ERROR,  
> /\* 6F \*/  C\_ERROR,  
> /\* 70 \*/  C\_ERROR,  
> /\* 71 \*/  C\_ERROR,  
> /\* 72 \*/  C\_ERROR,  
> /\* 73 \*/  C\_ERROR,  
> /\* 74 \*/  C\_ERROR,  
> /\* 75 \*/  C\_ERROR,  
> /\* 76 \*/  C\_ERROR,  
> /\* 77 \*/  C\_ERROR,  
> /\* 78 \*/  C\_ERROR,  
> /\* 79 \*/  C\_ERROR,  
> /\* 7A \*/  C\_ERROR,  
> /\* 7B \*/  C\_ERROR,  
> /\* 7C \*/  C\_ERROR,  
> /\* 7D \*/  C\_ERROR,  
> /\* 7E \*/  C\_ERROR,  
> /\* 7F \*/  C\_ERROR,  
> /\* 80 \*/  C\_DATA66+C\_REL,  
> /\* 81 \*/  C\_DATA66+C\_REL,  
> /\* 82 \*/  C\_DATA66+C\_REL,  
> /\* 83 \*/  C\_DATA66+C\_REL,  
> /\* 84 \*/  C\_DATA66+C\_REL,  
> /\* 85 \*/  C\_DATA66+C\_REL,  
> /\* 86 \*/  C\_DATA66+C\_REL,  
> /\* 87 \*/  C\_DATA66+C\_REL,  
> /\* 88 \*/  C\_DATA66+C\_REL,  
> /\* 89 \*/  C\_DATA66+C\_REL,  
> /\* 8A \*/  C\_DATA66+C\_REL,  
> /\* 8B \*/  C\_DATA66+C\_REL,  
> /\* 8C \*/  C\_DATA66+C\_REL,  
> /\* 8D \*/  C\_DATA66+C\_REL,  
> /\* 8E \*/  C\_DATA66+C\_REL,  
> /\* 8F \*/  C\_DATA66+C\_REL,  
> /\* 90 \*/  C\_MODRM,  
> /\* 91 \*/  C\_MODRM,  
> /\* 92 \*/  C\_MODRM,  
> /\* 93 \*/  C\_MODRM,  
> /\* 94 \*/  C\_MODRM,  
> /\* 95 \*/  C\_MODRM,  
> /\* 96 \*/  C\_MODRM,  
> /\* 97 \*/  C\_MODRM,  
> /\* 98 \*/  C\_MODRM,  
> /\* 99 \*/  C\_MODRM,  
> /\* 9A \*/  C\_MODRM,  
> /\* 9B \*/  C\_MODRM,  
> /\* 9C \*/  C\_MODRM,  
> /\* 9D \*/  C\_MODRM,  
> /\* 9E \*/  C\_MODRM,  
> /\* 9F \*/  C\_MODRM,  
> /\* A0 \*/  0,  
> /\* A1 \*/  0,  
> /\* A2 \*/  0,  
> /\* A3 \*/  C\_MODRM,  
> /\* A4 \*/  C\_MODRM+C\_DATA1,  
> /\* A5 \*/  C\_MODRM,  
> /\* A6 \*/  C\_ERROR,  
> /\* A7 \*/  C\_ERROR,  
> /\* A8 \*/  0,  
> /\* A9 \*/  0,  
> /\* AA \*/  0,  
> /\* AB \*/  C\_MODRM,  
> /\* AC \*/  C\_MODRM+C\_DATA1,  
> /\* AD \*/  C\_MODRM,  
> /\* AE \*/  C\_ERROR,  
> /\* AF \*/  C\_MODRM,  
> /\* B0 \*/  C\_MODRM,  
> /\* B1 \*/  C\_MODRM,  
> /\* B2 \*/  C\_MODRM,  
> /\* B3 \*/  C\_MODRM,  
> /\* B4 \*/  C\_MODRM,  
> /\* B5 \*/  C\_MODRM,  
> /\* B6 \*/  C\_MODRM,  
> /\* B7 \*/  C\_MODRM,  
> /\* B8 \*/  C\_ERROR,  
> /\* B9 \*/  C\_ERROR,  
> /\* BA \*/  C\_MODRM+C\_DATA1,  
> /\* BB \*/  C\_MODRM,  
> /\* BC \*/  C\_MODRM,  
> /\* BD \*/  C\_MODRM,  
> /\* BE \*/  C\_MODRM,  
> /\* BF \*/  C\_MODRM,  
> /\* C0 \*/  C\_MODRM,  
> /\* C1 \*/  C\_MODRM,  
> /\* C2 \*/  C\_ERROR,  
> /\* C3 \*/  C\_ERROR,  
> /\* C4 \*/  C\_ERROR,  
> /\* C5 \*/  C\_ERROR,  
> /\* C6 \*/  C\_ERROR,  
> /\* C7 \*/  C\_ERROR,  
> /\* C8 \*/  0,  
> /\* C9 \*/  0,  
> /\* CA \*/  0,  
> /\* CB \*/  0,  
> /\* CC \*/  0,  
> /\* CD \*/  C\_DATA1,  
> /\* CE \*/  0,  
> /\* CF \*/  0,  
> /\* D0 \*/  C\_ERROR,  
> /\* D1 \*/  C\_ERROR,  
> /\* D2 \*/  C\_ERROR,  
> /\* D3 \*/  C\_ERROR,  
> /\* D4 \*/  C\_ERROR,  
> /\* D5 \*/  C\_ERROR,  
> /\* D6 \*/  C\_ERROR,  
> /\* D7 \*/  C\_ERROR,  
> /\* D8 \*/  C\_ERROR,  
> /\* D9 \*/  C\_ERROR,  
> /\* DA \*/  C\_ERROR,  
> /\* DB \*/  C\_ERROR,  
> /\* DC \*/  C\_ERROR,  
> /\* DD \*/  C\_ERROR,  
> /\* DE \*/  C\_ERROR,  
> /\* DF \*/  C\_ERROR,  
> /\* E0 \*/  C\_ERROR,  
> /\* E1 \*/  C\_ERROR,  
> /\* E2 \*/  C\_ERROR,  
> /\* E3 \*/  C\_ERROR,  
> /\* E4 \*/  C\_ERROR,  
> /\* E5 \*/  C\_ERROR,  
> /\* E6 \*/  C\_ERROR,  
> /\* E7 \*/  C\_ERROR,  
> /\* E8 \*/  C\_ERROR,  
> /\* E9 \*/  C\_ERROR,  
> /\* EA \*/  C\_ERROR,  
> /\* EB \*/  C\_ERROR,  
> /\* EC \*/  C\_ERROR,  
> /\* ED \*/  C\_ERROR,  
> /\* EE \*/  C\_ERROR,  
> /\* EF \*/  C\_ERROR,  
> /\* F0 \*/  C\_ERROR,  
> /\* F1 \*/  C\_ERROR,  
> /\* F2 \*/  C\_ERROR,  
> /\* F3 \*/  C\_ERROR,  
> /\* F4 \*/  C\_ERROR,  
> /\* F5 \*/  C\_ERROR,  
> /\* F6 \*/  C\_ERROR,  
> /\* F7 \*/  C\_ERROR,  
> /\* F8 \*/  C\_ERROR,  
> /\* F9 \*/  C\_ERROR,  
> /\* FA \*/  C\_ERROR,  
> /\* FB \*/  C\_ERROR,  
> /\* FC \*/  C\_ERROR,  
> /\* FD \*/  C\_ERROR,  
> /\* FE \*/  C\_ERROR,  
> /\* FF \*/  C\_ERROR  
> }; // ade32\_table\[\]  
>   
> // ade32\_disasm() -- returns opcode length or 0  
>   
> int ade32\_disasm(IN BYTE\* opcode0, IN OUT disasm\_struct\* diza)  
> {  
>   BYTE\* opcode = opcode0;  
>   
>   disasm\_struct temp\_diza;                   // comment if NULL is never passed  
>   if (diza == NULL) diza = &temp\_diza;       //  
>   
>   memset(diza, 0x00, sizeof(disasm\_struct)); // comment these lines,  
>   diza->disasm\_defdata = 4;                  // and fill structure before call  
>   diza->disasm\_defaddr = 4;                  // -- to 允许 16/32-bit disasm  
>   
>   if (\*(WORD\*)opcode == 0x0000) return 0;  
>   if (\*(WORD\*)opcode == 0xFFFF) return 0;  
>   
>   DWORD flag = 0;  
>     
> repeat\_prefix:  
>   //按字节取出操作码  
>   BYTE c = \*opcode++;  
>   //查表,取出特征码  
>   DWORD t = ade32\_table\[ c \];  
>   //是否是已知前缀  
>   if (t & C\_ANYPREFIX)  
>   {  
>     //是否为两次相同前缀  
>     if (flag & t) return 0;    // twice LOCK,SEG,REP,66,67  
>     //标志  
>     flag |= t;  
>     //如果含有 C\_67，需要 16 位地址  
>     if (t & C\_67)  
>     {   
>       //???  
>       diza->disasm\_defaddr ^= 2⁴;  
>     }  
>     //如果含有 C\_66，使用 16 位操作数  
>     else  
>     if (t & C\_66)  
>     {  
>       //???  
>       diza->disasm\_defdata ^= 2⁴;  
>     }  
>     else  
>     //含有段标志  
>     if (t & C\_SEG)  
>     {  
>       //保存段前缀  
>       diza->disasm\_seg = c;  
>     }  
>     else  
>     if (t & C\_REP)  
>     {  
>       //原理同上  
>       diza->disasm\_rep = c;  
>     }  
>     // LOCK  
>   
>     goto repeat\_prefix;  
>   
>   } // C\_ANYPREFIX  
>   //保存标志  
>   flag |= t;  
>   //保存操作码  
>   diza->disasm\_opcode = c;  
>   //操作码是否含有第二个字节  
>   if (c == 0x0F)  
>   {  
>     c = \*opcode++;  
>     //取出第二个字节  
>     diza->disasm\_opcode2 = c;  
>     //根据第二字节取得指令特征  
>     flag |= ade32\_table\[ 256 + c \]; // 2nd flagtable half  
>   
>     if (flag == C\_ERROR) return 0;  
>   }  
>   //操作码大概为 F7xx  
>   else  
>   if (c == 0xF7)  
>   {  
>     //opcode 已经自加 1  
>     if (((\*opcode) & 0x38)==0)  
>       //xx 为数据（立即数？？)  
>       flag |= C\_DATA66;  
>   }  
>   //不详，同上  
>   else  
>   if (c == 0xF6)  
>   {  
>     if (((\*opcode) & 0x38)==0)  
>       flag |= C\_DATA1;  
>   }  
>   //如果设置 C\_MODRM 标志  
>   if (flag & C\_MODRM)  
>   {  
>     c = \*opcode++;  
>     //保存值  
>     diza->disasm\_modrm = c;  
>     //在 MorR/M 右边含有一个 SIB 字节  
>     if ((c & 0x38) == 0x20)  
>         if (diza->disasm\_opcode == 0xFF)  
>               flag |= C\_STOP;  
>     //MorR/M 的高 2 位  
>     BYTE mod = c & 0xC0;  
>     //MorR/M 的低 3 位  
>     BYTE rm  = c & 0x07;  
>     //如果高 2 位不为 11  
>     if (mod != 0xC0)  
>     {  
>       if (diza->disasm\_defaddr == 4)  
>       {  
>         //寄存器参与地址计算  
>         if (rm == 4)  
>         {  
>           ////在 MorR/M 右边含有一个 SIB 字节  
>           flag |= C\_SIB;  
>           //取出 SIB  
>           c = \*opcode++;  
>           //保存  
>           diza->disasm\_sib = c;  
>           //取出 SIB 的低 3 位，为 base 寄存器  
>           rm = c & 0x07;  
>         }  
>         //操作数为内存地址+字节偏移量，且地址在寄存器中  
>         if (mod == 0x40)  
>         {  
>           flag |= C\_ADDR1;  
>         }  
>         //操作数为内存地址+双字偏移量，同上  
>         else  
>         if (mod == 0x80)  
>         {  
>           flag |= C\_ADDR4;  
>         }  
>         //mod 高 2 位为 00  
>         else  
>         {  
>           //且 mod 低 3 位为 101，则不使用寄存器计算地址  
>           if (rm == 5)  
>             flag |= C\_ADDR4;  
>         }  
>       }  
>       else // MODRM 16-bit  
>       {  
>   
>         if (mod == 0x40)  
>         {  
>           flag |= C\_ADDR1;  
>         }  
>         else  
>         if (mod == 0x80)  
>         {  
>           flag |= C\_ADDR2;  
>         }  
>         else  
>         {  
>           if (rm == 6)  
>             flag |= C\_ADDR2;  
>         }  
>       }  
>     }  
>   } // C\_MODRM  
>    //保存标志  
>   diza->disasm\_flag = flag;  
>   //取出标志  
>   DWORD a =  flag & (C\_ADDR1 | C\_ADDR2 | C\_ADDR4);  
>   //#define C\_DATA1   0x00000100   
>   //#define C\_DATA2   0x00000200  
>   //#define C\_DATA4   0x00000400  
>   //移动到低四位  
>   DWORD d = (flag & (C\_DATA1 | C\_DATA2 | C\_DATA4)) >> 8;  
>   //全部地址长度  
>   if (flag & C\_ADDR67) a += diza->disasm\_defaddr;  
>   //全都立即数长度  
>   if (flag & C\_DATA66) d += diza->disasm\_defdata;  
>   //???  
>   diza->disasm\_addrsize = a;  
>   diza->disasm\_datasize = d;  
>   //取出偏移量  
>   DWORD i;  
>   for(i=0; i<a; i++)  
>     diza->disasm\_addr\_b\[i\] = \*opcode++;  
>   //取出立即数  
>   for(i=0; i<d; i++)  
>     diza->disasm\_data\_b\[i\] = \*opcode++;  
>   //长度  
>   diza->disasm\_len = opcode - opcode0;  
>     
>   return diza->disasm\_len;  
>   
> } // ade32\_disasm()  
>   
> // ade32\_asm() -- returns assembled opcode length  
>   
> int ade32\_asm(OUT BYTE\* opcode, IN OUT disasm\_struct\* s)  
> {  
>   BYTE\* opcode0 = opcode;  
>   
>   if (s->disasm\_flag & C\_SEG)     \*opcode++ = s->disasm\_seg;  
>   if (s->disasm\_flag & C\_LOCK)    \*opcode++ = 0xF0;  
>   if (s->disasm\_flag & C\_REP)     \*opcode++ = s->disasm\_rep;  
>   if (s->disasm\_flag & C\_67)      \*opcode++ = 0x67;  
>   if (s->disasm\_flag & C\_66)      \*opcode++ = 0x66;  
>   \*opcode++ = s->disasm\_opcode;  
>   if (s->disasm\_flag & C\_OPCODE2) \*opcode++ = s->disasm\_opcode2;  
>   if (s->disasm\_flag & C\_MODRM)   \*opcode++ = s->disasm\_modrm;  
>   if (s->disasm\_flag & C\_SIB)     \*opcode++ = s->disasm\_sib;  
>   for (DWORD i=0; i<s->disasm\_addrsize; i++)  
>     \*opcode++ = s->disasm\_addr\_b\[i\];  
>   for (DWORD i=0; i<s->disasm\_datasize; i++)  
>     \*opcode++ = s->disasm\_data\_b\[i\];  
>   
>   return opcode - opcode0;  
> } // ade32\_asm 

 4dd466d0e1e0cc123afbe504da2462ca