

# X86-64CPU 下 开发 64 位操作系统试验----双进程试验

> 原文：[`www.pediy.com/kssd/pediy10/67679.html`](https://www.pediy.com/kssd/pediy10/67679.html)

前一阵在网上翻到一篇介绍如何初始化“长模式”的文章，（文章出处已经记不得了，在此对作者表示感谢）我对文章进行了研究，发现其中有些不对，就仔细阅读了 AMD64 的技术手册 Vol2，也就是系统编程那一卷。然后改正了文章的一些错误，并以此为基础，近一步实现了长模式下的双进程试验。前后 4 天的文档阅读，到代码修改到调试成功还真是花了不少功夫，也让我对 AMD64 有了更深入的了解。下面就把我的一些学习心得和大家分享。  
    长模式不同于兼容模式，在长模式下所有的代码和数据都是 64 位编码的，IDT，TSS，段描述符号等数据结构都做了相应的扩展，任务切换，分页的处理等都有了些变化，具体的请大家去看 AMD64 的手册，下面我就简单说一下我下面的代码。代码从实模式开始运行按如下步骤操作：  
    1.关中断，首先利用 CPUID 检查 CPU 是否支持长模式，如果不支持则打印信息，然后停机(halt)。  
    2.打开 A20 线，载入 32 位保护模式下用的 GDT 和 IDT，其中 64 位模式下的相关数据结构也包含在了此 GDT 下，包括 64 位下的 0 级代码段描述符，0 级数据段描述符，3 级代码段和数据段描述符，TSS64 描述符；32 位的 IDT 里面的中断处理程序什么也不作只是 IRETD，因为我们直到最后才会开启中断。设置好数据段和堆栈指针，利用一个长跳转进入 32 位保护模式，由于 GDT 里面的设置，我们其实是进入了 FLAT 模式，呵呵。  
    3.在 32 位模式下我们的主要工作就是初始化 4 级分页系统。这里我们采用的是 PAE－4K 每页的分页方式，内存设定为 32M，这在 bochs2.3.6 下面是可以的，配置文件里面设置了 32M 内存。其中 PAE 物理地址扩展是进入长模式所必须的，分页也是长模式所必须的。为了简单起见，分页后的虚拟地址和实际的物理地址是一一对应的。64 位模式下，页表项是 8 字节长的，这一点要注意。首先把所有的内存都设为只有 0 级可以访问。然后又特别开放了 32K 以后到 C0000 的内存给 RING3，这是因为我们的代码是加载在内存 8000 地址的，同时我们的 RING3 代码也会写屏，所以要开发到 C0000。这里还要注意，页级别保护是 AND 运算的，也就是每一级的保护位 AND 起来才是最后的保护模式。所以，我要把前三级的低 3 位都置成 1，也就是 7，而在最后一级上做限制才可以。另外页表系统在 1M 内存以上，0，1，2 级的堆栈指针都在 1M 以上，这样就防止 RING3 的代码恶意修改内核的数据了，实现了隔离。另外还设置了两个 RING3 的 RSP 指针，我们这里只演示了两个进程。  
  4. 分页完成之后，就是开启 PAE，开启分页，启动长模式，利用长跳转进入 64 位代码。此时我们已经进入了纯 64 位模式了，进入纯 64 位模式之后我们会在屏幕中间显示一个粉色的 L 表示成功，而后重新裁入 GDT64，IDT64 等数据结构，重置选择子等工作。我们还把模式寄存器的值读了出来，可以在调试时看出长模式是 ACTIVE 的，EFER = 0x00000500。我们还在屏幕中间显示了一个 0，这在后面要用到。  
  5. 下面就开始安排双进程了，我们这里又两个进程：  
    进程 1:不断的增加 rax，rax 已经预先清 0，同时做推栈和弹栈，这是测试 RING3 的堆栈可用。  
       .INE:    inc rax  
        push rax  
        pop  rbx  
        nop  
        nop  
        nop  
        nop  
       jmp .INE  
     
      进程 2:在我们前面显示 0 的地方，轮流显示 0～9 字符，同时做推栈和弹栈，这是测试 RING3 的堆栈可用。  
     .INE2:  mov al,byte \[0xb8060\] ;实验  
        inc al  
        cmp al,0x3a  
        jnz  .T1  
        mov al,'0'  
     .T1:    mov \[0xb8060\],al          
        push rax  
        pop  rcx      
        jmp .INE2     
      
   只是两个简单的进程，大家可以加入更复杂的东西。不过堆栈只预留了 4KB。下面开始讲开启 RING3 进程和进程调度。  
6. 我们一直都在 RING0 工作，现在我们要返回 RING3 了，我们采用的手段是通过构造中断返回现场来返回，我们压入进程 1 的代码指针等相关数据结构，把 RFLAG 的 IF 位放开，通过 IRETQ 来返回到 RING3.这条指令执行过后，进程 1 就开始跑了。下面来说调度。要说调度我们先来看 64 位下的中断处理函数，除了时间中断，其他的中断处理只是简单的返回。我们这里为了简单，依然用 IRQ8 来处理时间中断。每当中断来临时候，我们首先会把 EAX 的值 DUMP 到屏幕上，然后通过一个 ctrl 变量来指示应该换入哪个进程。我们已经事先在寄存器保存区设置好了进程 2 的相关量，进程切换是很容易的事情，无非就是保存 A 进程的相关寄存器，载入 B 进程的相关寄存器，然后从中断返回。这样在屏幕上大家就可以看见粉色的 0～9 在不停的翻转，同时右边红色的 EAX 值也在不停的刷新。在 BOCHS 中可以发现，EAX 的值不是连续的，显示恍惚，那是因为两个进程都用到了 RAX，而每次时间中断到来时都会 DUMP 当前进程的 RAX，所以显示会恍惚，这也正显示了进程调度的成功。大家可以在 BOCHS 中按暂停键来看右边 DUMP 的数据的变化。    
   好了所有的东西就讲完了。附件里时直接可以在 BOCHS 下面加载的磁盘镜像。下面的代码是按照 FASM 格式编写的，产生纯二进制代码，然后由我的载入程序载入到 0x8000 的。下面是我从 BOCHS 的 log 文件中截取的一段：  
0001796619i\[BIOS \] Booting from 0000:7c00  
01699216000p\[WGUI \] >>PANIC<< POWER button turned off.我关调了 BOCHS  
01699216000i\[CPU0 \] CPU is in long mode (active)  
01699216000i\[CPU0 \] CS.d\_b = 16 bit  
01699216000i\[CPU0 \] SS.d\_b = 32 bit  
01699216000i\[CPU0 \] EFER   = 0x00000500  
01699216000i\[CPU0 \] | RAX=0000000006508920  RBX=0000000006508920  
01699216000i\[CPU0 \] | RCX=00000000c0000080  RDX=0000000000000000  
01699216000i\[CPU0 \] | RSP=000000000009ffff  RBP=0000000000007caf  
01699216000i\[CPU0 \] | RSI=00000000ffff88ca  RDI=00000000001235f8  
01699216000i\[CPU0 \] |  R8=0000000000000000   R9=0000000000000000  
01699216000i\[CPU0 \] | R10=0000000000000000  R11=0000000000000000  
01699216000i\[CPU0 \] | R12=0000000000000000  R13=0000000000000000  
01699216000i\[CPU0 \] | R14=0000000000000000  R15=0000000000000000  
01699216000i\[CPU0 \] | IOPL=0 id vip vif ac vm rf nt of df IF tf sf zf AF pf cf  
01699216000i\[CPU0 \] | SEG selector     base    limit G D  
01699216000i\[CPU0 \] | SEG sltr(index|ti|rpl)     base    limit G D  
01699216000i\[CPU0 \] |  CS:003b( 0007| 0|  3) 00000000 0000ffff 0 0  
01699216000i\[CPU0 \] |  DS:0000( 0004| 0|  0) 00000000 000fffff 1 1  
01699216000i\[CPU0 \] |  SS:0043( 0008| 0|  3) 00000000 000fffff 1 1  
01699216000i\[CPU0 \] |  ES:0000( 0004| 0|  0) 00000000 000fffff 1 1  
01699216000i\[CPU0 \] |  FS:0000( 0000| 0|  0) 00000000 000fffff 1 1  
01699216000i\[CPU0 \] |  GS:0000( 0000| 0|  0) 00000000 000fffff 1 1  
01699216000i\[CPU0 \] |  MSR\_FS\_BASE:0000000000000000  
01699216000i\[CPU0 \] |  MSR\_GS\_BASE:0000000000000000  
01699216000i\[CPU0 \] | RIP=0000000000009b44 (0000000000009b44)  
01699216000i\[CPU0 \] | CR0=0x80000011 CR1=0x0 CR2=0x0000000000000000  
01699216000i\[CPU0 \] | CR3=0x00120000 CR4=0x00000020  
01699216000i\[CPU0 \] >> nop  : 90   这是关机时执行的指令  
  
  
///////////////////////////////////////////////////////////////////////////////////////  
  
  
 USE16  
    org 0x8000  
       REAL\_MODE\_STACK\_TOP EQU 0x7C00 - 1  
    CODE\_SEL EQU CODE\_DSCR - DUMMY\_DSCR   ; code segment selector  
    DATA\_SEL EQU DATA\_DSCR - DUMMY\_DSCR   ; data segment selector  
      
    LONG\_MODE\_RSP3   EQU    0xA0000 - 1  
    LONG\_MODE\_RSP31    EQU    0xA0000 - 1 - 4 \* 1024  
    LONG\_MODE\_RSP0    EQU    0x120000 - 1   
    LONG\_MODE\_RSP1    EQU    0x120000 - 1 - 4 \* 1024  
    LONG\_MODE\_RSP2   EQU    0x120000 - 1 - 8 \* 1024  
    LONG\_MODE\_PTE\_BASE    = 0x123000  
    LONG\_MODE\_PDE\_BASE    = 0x122000  
    LONG\_MODE\_PDPE\_BASE    = 0x121000  
    LONG\_MODE\_PML4E\_BASE= 0x120000  
      
jmp 0x0:CODE16    ; jump to 16 bit code, initialize base enviroment  
ALIGN 4  
GDTR32    dw GDT\_END - GDT - 1  
        dd GDT  
ALIGN 4  
IDTR32    dw IDT32\_END - IDT32\_START - 1  
        dd IDT32\_START  
          
PRINT\_16:  
    push ax  
    push bx  
    push si  
 .NEW\_CHAR:  
    lodsb  
    or  al, al  
    jz  short .EXIT\_PRINT\_16  
    mov ah, 0x0E  
    mov bx, 0x0007  
    int 0x10  
    jmp short .NEW\_CHAR  
 .EXIT\_PRINT\_16:  
    pop si  
    pop bx  
    pop ax  
    ret  
  
MSG\_NO\_LONG\_MODE db 'Current CPU dose not support LONG mode',13,10,0  
NO\_LONG\_MODE:  
    cli  
    mov si,MSG\_NO\_LONG\_MODE  
    call PRINT\_16  
    hlt  
    jmp $   ; halt here  
  
          
; 16 bit main start  
CODE16:  
    ; initialize 16 bit registers  
    cli  
    mov ax, cs  
    mov ds, ax  
    mov es, ax  
    xor ax, ax  
    mov ss, ax  
    mov sp, REAL\_MODE\_STACK\_TOP  
  
    ; check if x64 long mode is supported  
    ; AMD document for details about this check  
    ; NOTE: I suppose CPU is 386+, maybe I should  
    ;   do another check  
    mov eax, 0x80000000  
    cpuid  
    cmp eax, 0x80000000  
    jbe NO\_LONG\_MODE  
    mov eax, 0x80000001  
    cpuid  
    bt  edx, 29  
    jnc NO\_LONG\_MODE  
        
    ; prepare for protect mode  
    ; load gdtr  
    ; enable cr0.pe  
    cli  
    ; enable A20 gate  
      
    in      al,92h  
    or      al,00000010b  
    out     92h,al  
                 
    mov  ebx, IDTR32  
    lidt \[ebx\]  
    mov  ebx, GDTR32  
    lgdt \[ebx\]  
    mov  eax, cr0  
    or   eax, 1  
    mov  cr0, eax  
  
    mov bx, DATA\_SEL  
    mov ds, bx  
    mov es, bx  
    mov fs, bx  
    mov gs, bx  
    mov ss, bx  
    mov esp, 0x7C00 - 1  
      
    ; make a far jump to enter protect mode  
    db 0xEA  
    dw CODE32  
    dw CODE\_SEL  
  
;--------------- PROTECT MODE ----------  
;................  FOR protect mode  ...................      
ALIGN 16  
GDT:  
DUMMY\_DSCR    dw 0  
            dw 0  
            dw 0  
            dw 0    ; reserved by i386 CPU  
  
CODE\_DSCR    dw 0xFFFF  
            dw 0x0000  
            db 0x00  
            db 0x9A  
            db 0xCF  
            db 0x00  
  
DATA\_DSCR    dw 0xFFFF  
            dw 0x0000  
            db 0x00  
            db 0x92  
            db 0xCF  
            db 0x00  
                          
    CODE64\_SELECTOR        =    .CODE\_DESCRIPTOR - GDT  
    .CODE\_DESCRIPTOR    dw 0xFFFF  
                        dw 0x0000  
                        db 0x00  
                        db 0x9A  
                        db 0x20  
                        db 0x00  
                          
    DATA64\_SELECTOR        =    .DATA\_DESCRIPTOR - GDT  
    .DATA\_DESCRIPTOR    dw 0xFFFF  
                        dw 0x0000  
                        db 0x00  
                        db 0x92  
                        db 0xCF  
                        db 0x00  
                          
    TSS64\_SELECTOR        =    .TSS\_DESCRIPTOR - GDT  
    .TSS\_DESCRIPTOR        dw TSS64\_END - TSS64 - 1  
                        dw TSS64  
                        db 0x00  
                        db 0x89        ; available TSS  
                        db 0x80  
                        db 0x00  
                        dd 0x00000000  
                        dd 0x00000000  
   CODE64\_SELECTOR3        =    .CODE\_DESCRIPTOR3 - GDT+3  
    .CODE\_DESCRIPTOR3    dw 0xFFFF  
                        dw 0x0000  
                        db 0x00  
                        db 0xfA  ;3 级代码段  
                        db 0x20  
                        db 0x00                       
  
DATA64\_SELECTOR3        =    .DATA\_DESCRIPTOR3 - GDT+3  
    .DATA\_DESCRIPTOR3    dw 0xFFFF  
                        dw 0x0000  
                        db 0x00  
                        db 0xf2  
                        db 0xCF  
                        db 0x00                          
GDT\_END:  
  
ALIGN 16  
IDT32\_START:  
    rept 256 {  
        dw PROTECT\_MODE\_EXCEPTION\_HANDLE  
        dw CODE\_SEL  
        db 0x00  
        db 0x8E  
        dw 0x0000  
    }  
IDT32\_END:  
  
;---------- FOR LONG MODE -----------  
ALIGN 4  
GDTR64    dw GDT\_END - GDT - 1  
        dq GDT  
          
IDTR64    dw IDT64\_END - IDT64 - 1  
        dq IDT64  
  
ALIGN 16  
TSS64:  
    dd 0    ; reserved, ingore  
    dq LONG\_MODE\_RSP0  
    dq LONG\_MODE\_RSP1  
    dq LONG\_MODE\_RSP2  
    dq 0    ; reserved  
    times 7 dq LONG\_MODE\_RSP0  
    dq 0  
    dw 0  
    dw $ + 2  
    db 0xff  
TSS64\_END:  
  
ALIGN 16  
IDT64:  
    rept 8 {  
        dw LONG\_MODE\_EXCEPTION\_HANDLE1  
        dw CODE64\_SELECTOR  
        db 0x00  
        db 0x8E  
        dw 0x0000  
        dd 0x00000000  
        dd 0x00000000  
    }  
    ;时钟中断  
        dw LONG\_MODE\_EXCEPTION\_HANDLE  
        dw CODE64\_SELECTOR  
        db 0x00  
        db 0x8E  
        dw 0x0000  
        dd 0x00000000  
        dd 0x00000000  
    rept 247 {  
        dw LONG\_MODE\_EXCEPTION\_HANDLE1  
        dw CODE64\_SELECTOR  
        db 0x00  
        db 0x8E  
        dw 0x0000  
        dd 0x00000000  
        dd 0x00000000  
    }          
IDT64\_END:  
  
USE32  
  
CODE32:  
;初始化 PML4E  
    mov edi,LONG\_MODE\_PML4E\_BASE  
    mov \[edi\],dword LONG\_MODE\_PDPE\_BASE+7  
    add edi,4  
    xor ebx,ebx  
    mov \[edi\],ebx  
    add edi,4  
    mov ecx,511  
.NPML:      
    mov \[edi\],ebx  
    add edi,4  
    mov \[edi\],ebx  
    add edi,4  
    loop .NPML  
  
;初始化 PDPE      
    mov \[edi\],dword LONG\_MODE\_PDE\_BASE+7  
    add edi,4  
    mov \[edi\],ebx  
    add edi,4  
    mov ecx,511  
.NPDE:  
    mov \[edi\],ebx  
    add edi,4  
    mov \[edi\],ebx  
    add edi,4  
    loop .NPDE          
      
;初始化 PDE      
    mov eax,LONG\_MODE\_PTE\_BASE+7  
    mov \[edi\],eax  
    add edi,4  
    mov edi,ebx  
    add edi,4  
    mov ecx,15  
.NPTE:  
    add eax,0x1000  
    mov \[edi\],eax  
    add edi,4  
    mov edi,ebx  
    add edi,4     
    loop .NPTE  
    mov ecx,512-16  
.NPTE1:      
    mov \[edi\],ebx  
    add edi,4  
    mov \[edi\],ebx  
    add edi,4  
    loop .NPTE1  
   
;初始化 PTE      
   mov edi,LONG\_MODE\_PTE\_BASE  
   mov eax, 1  
   mov ecx,512\*16  
.NPP:     
   mov \[edi\],eax  
   add edi,4  
   mov \[edi\],ebx  
   add edi,4  
   add eax,0x1000  
   loop .NPP   
     
   ;保留前 32k 内存，开放 32K 以后到 C0000 之前的内存给 ring3  
   mov edi,LONG\_MODE\_PTE\_BASE+0x8\*8  
   mov eax, 0x8\*0x1000+7  
   mov ecx, 183  
.NP2:     
   mov \[edi\],dword eax  
   add eax,0x1000  
   add edi,8      
   loop .NP2   
   
      
    mov eax, cr4  
    bts eax, 5  
    mov cr4, eax  
  
    mov eax, LONG\_MODE\_PML4E\_BASE  
    mov cr3, eax  
      
    mov ecx, 0xC0000080  
    rdmsr  
    bts eax, 8  
    wrmsr  
  
    mov eax, cr0  
    and eax, 0x1FFFFFFF  
    bts eax, 31  
    mov cr0, eax  
      
    db 0xEA  
    dd CODE64  
    dw CODE64\_SELECTOR  
                     
PROTECT\_MODE\_EXCEPTION\_HANDLE:  
    iretd  
      
; Dump eax to screen  
HEX\_TABLE    db '0', '1', '2', '3', '4', '5', '6', '7'  
            db '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'  
  
      
USE64  
LONG\_MODE\_EXCEPTION\_HANDLE1:  
    iretq  
  
CODE64:  
    mov rax, DATA64\_SELECTOR  
    mov ds, ax  
    mov es, ax  
    xor ax,ax  
    mov ss, ax  
    mov fs, ax  
    mov gs, ax  
    mov rsp, LONG\_MODE\_PML4E\_BASE - 1  
    mov rbx, GDTR64  
    lgdt \[rbx\]  
    mov rbx, IDTR64  
    lidt \[rbx\]  
           
    mov ax, TSS64\_SELECTOR  
    ltr ax  
     
    mov rax, LONG\_MODE\_PML4E\_BASE  
    mov cr3, rax  
      
    mov \[0xB8050\], byte 'L'  
    mov \[0xB8051\], byte 13  
      
    mov \[0xB8060\], byte '0'  
    mov \[0xB8061\], byte 13  
      
    mov rcx, 0xC0000080  
    rdmsr    
    xor rax,rax           
    mov \[srip\],.INE2   
    push DATA64\_SELECTOR3  
    push LONG\_MODE\_RSP3  
    pushfq  
    pop rbx  
    bts rbx,9  
    push rbx  
    push CODE64\_SELECTOR3  
    push .INE  
    iretq  
      
   .INE:    inc rax  
    push rax  
    pop  rbx  
    nop  
    nop  
    nop  
    nop  
jmp .INE  
     
     
.INE2:  mov al,byte \[0xb8060\] ;实验  
        inc al  
        cmp al,0x3a  
        jnz  .T1  
        mov al,'0'  
.T1:    mov \[0xb8060\],al          
        push rax  
        pop  rcx      
        jmp .INE2     
          
SAVEREG:  
 ;第二个进程的参数  
          srax dq 0  
          srbx dq 0  
          srcx dq 0  
          srdx dq 0  
          srbp dq 0  
          srsi dq 0  
          srdi dq 0  
          srsp dq LONG\_MODE\_RSP31  
          sr8  dq 0  
          sr9  dq 0  
          sr10 dq 0  
          sr11 dq 0  
          sr12 dq 0  
          sr13 dq 0  
          sr14 dq 0  
          sr15 dq 0  
          sr\*\*\* dq 0x246    
          srip  dq 0  
;第一个进程的参数  
          srax1 dq 0  
          srbx1 dq 0  
          srcx1 dq 0  
          srdx1 dq 0  
          srbp1 dq 0  
          srsi1 dq 0  
          srdi1 dq 0  
          srsp1 dq 0  
          sr81  dq 0  
          sr91  dq 0  
          sr10\_1 dq 0  
          sr11\_1 dq 0  
          sr12\_1 dq 0  
          sr13\_1 dq 0  
          sr14\_1 dq 0  
          sr15\_1 dq 0  
          sr\*\*\*\_1 dq 0    
          srip\_1  dq 0  
 ctrl dd 0                     
LONG\_MODE\_EXCEPTION\_HANDLE:  
    call DUMP\_RAX  
    push rax  
    mov al,20h  
    out 20h,al  
    mov eax,\[ctrl\]  
    or eax, eax  
    jz .TASK2 ;应该换出入第二个进程  
.TASK1:  
    dec \[ctrl\]  
    pop \[srax\]  
    mov \[srbx\], rbx  
    mov \[srcx\], rcx  
    mov \[srdx\], rdx  
    mov \[srbp\], rbp  
    mov \[srsi\], rsi  
    mov \[srdi\], rdi   
    mov \[sr8\],  r8    
    mov \[sr9\],  r9      
    mov \[sr10\], r10  
    mov \[sr11\], r11  
    mov \[sr12\], r12  
    mov \[sr13\], r13  
    mov \[sr14\], r14  
    mov \[sr15\], r15  
    pop \[srip\]  
    pop rax  
    pop \[sr\*\*\*\]  
    pop \[srsp\]  
    push \[srsp1\]  
    push \[sr\*\*\*\_1\]  
    push rax  
    push \[srip\_1\]   
    mov rax, \[srax1\]  
    mov rbx, \[srbx1\]  
    mov rcx, \[srcx1\]  
    mov rdx, \[srdx1\]  
    mov rbp, \[srbp1\]  
    mov rsi, \[srsi1\]  
    mov rdi, \[srdi1\]  
    mov r8,  \[sr81\]    
    mov r9,  \[sr91\]      
    mov r10, \[sr10\_1\]  
    mov r11, \[sr11\_1\]  
    mov r12, \[sr12\_1\]  
    mov r13, \[sr13\_1\]  
    mov r14, \[sr14\_1\]  
    mov r15, \[sr15\_1\]               
    iretq               
.TASK2:     
    inc \[ctrl\]  
    pop \[srax1\]  
    mov \[srbx1\], rbx  
    mov \[srcx1\], rcx  
    mov \[srdx1\], rdx  
    mov \[srbp1\], rbp  
    mov \[srsi1\], rsi  
    mov \[srdi1\], rdi   
    mov \[sr81\],  r8    
    mov \[sr91\],  r9      
    mov \[sr10\_1\], r10  
    mov \[sr11\_1\], r11  
    mov \[sr12\_1\], r12  
    mov \[sr13\_1\], r13  
    mov \[sr14\_1\], r14  
    mov \[sr15\_1\], r15  
    pop \[srip\_1\]  
    pop rax  
    pop \[sr\*\*\*\_1\]  
    pop \[srsp1\]  
    push \[srsp\]  
    push \[sr\*\*\*\]  
    push rax  
    push \[srip\]       
    mov rax, \[srax\]  
    mov rbx, \[srbx\]  
    mov rcx, \[srcx\]  
    mov rdx, \[srdx\]  
    mov rbp, \[srbp\]  
    mov rsi, \[srsi\]  
    mov rdi, \[srdi\]   
    mov r8,  \[sr8\]    
    mov r9,  \[sr9\]      
    mov r10, \[sr10\]  
    mov r11, \[sr11\]  
    mov r12, \[sr12\]  
    mov r13, \[sr13\]  
    mov r14, \[sr14\]  
    mov r15, \[sr15\]           
    iretq  
DUMP\_RAX:  
    push rax  
    push rbx  
    push rcx  
    push rdx  
      
    mov ebx, 0xB8000 + 2 \* ( 0 \* 25 + 80 - 10)  
    mov dh, 0xC  
    mov dl, '0'  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
  
    mov dl, 'x'  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
      
    mov ecx, eax  
    shr ecx, 28  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
      
    mov ecx, eax  
    shr ecx, 24  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
      
    mov ecx, eax  
    shr ecx, 20  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
  
    mov ecx, eax  
    shr ecx, 16  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
  
    mov ecx, eax  
    shr ecx, 12  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
  
    mov ecx, eax  
    shr ecx, 8  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
  
    mov ecx, eax  
    shr ecx, 4  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
    inc ebx  
    inc ebx  
      
    mov ecx, eax  
    and ecx, 0x0000000F  
    mov dl, byte \[HEX\_TABLE + ecx\]  
    mov \[ebx\], dx  
      
    pop rdx  
    pop rcx  
    pop rbx  
    pop rax  
    ret  
      
  
////////////////////////////////////////////////  
欢迎大家有空到我的博客：[`blog.sina.com.cn/tigerdeng`](http://blog.sina.com.cn/tigerdeng) 来逛逛啊。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || drboot1.rar  
 |