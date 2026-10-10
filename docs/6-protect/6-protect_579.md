

# 虚拟机保护技术浅谈

> 原文：[`www.pediy.com/kssd/pediy12/139586.html`](https://www.pediy.com/kssd/pediy12/139586.html)

对加密与解密的内容进行了一些总结、重新归纳整理，加入了自己的理解；希望对新手有所帮助。  
  
                                  《加密与解密》第三版  第 471 页  
虚拟机保护技术  
  虚拟机概览  
     所谓虚拟机保护技术，是指将代码翻译为机器和人都无法识别的一串伪代码字节流；在具体执行时再对这些伪代码进行一一翻译解释，逐步还原为原始代码并执行。  
     这段用于翻译伪代码并负责具体执行的子程序就叫作虚拟机 VM（好似一个抽象的 CPU）。它以一个函数的形式存在，函数的参数就是字节码的内存地址。  
     将虚拟机应用到商业中的保护壳现有三款:Vmprotect，themida 和 execrypt。  
虚拟机架构  
     我们知道，代码中的指令多种多样，组织形式也千变万化；虚拟机不可能针对每一种具体情况都进行翻译处理。必须对所有可能遇到的指令先进行抽象归类，然后分解为若干简单的小指令，再交由各个专门的子程序（handler）去处理。  
     学过编译原理的同学应该都知道三元式代码吧，也叫做 3 地址代码(three adress code)。即不论多么复杂的赋值公式，都可以分解为数个 3 地址代码式序列。  
    （什么是 3 地址代码，1 段 3 地址代码只完成 1 次运算，譬如 1 次二目运算、1 次比较，或者 1 次分支跳转运算。）  
     与此类似，不论多么复杂的指令，都可以分解为一串不可再分割的原子指令序列。  
  
     虚拟机（CPU）的体系架构可分为 3 种，基于堆栈的(Stack based)，基于寄存器的(Register based)和 3 地址机器。我们只讲述基于堆栈的虚拟机架构(Stack based)；这种架构的虚拟机需要频繁操作堆栈，其使用的虚拟寄存器(虚拟的 eax、ebx 等等)保存在堆栈中；每个原子指令的 handler 都需要 push、pop。  
     现在的 CPU 都有大量的寄存器，堆栈一般只是在函数传递参数时使用（譬如 PC 机用的 x86 系列 CPU）。但也有一些 CPU 只操作内存，没有堆栈，也没有寄存器。使用这种 CPU 的机器称之为 3 地址机器。  
     基于堆栈的 CPU 或虚拟机没有临时变量、寄存器的概念，所有的东西都放入堆栈。由于指令中不需要指定操作数，所以其指令相对基于寄存器的要短。也因此相对简单，在嵌入式系统中运用较多。用于保护代码，我们也选择这种。  
     举例，譬如指令 add,基于堆栈的 CPU 首先从堆栈里 Pop 两个数，然后将两数相加，再把和 Push 到堆栈。Add 指令只占用 1 个字节。而基于寄存器的 CPU 对应指令为 add Reg1,Reg2，需要 3 个字节。请仔细想象一下没有寄存器的 CPU，它的指令是怎样的，该会是多么简洁。当然，指令简洁带来的缺点就是效率低下。  
     我们这里谈的虚拟机保护技术，就是把基于寄存器的 CPU 代码，改造成基于堆栈的 CPU 的伪代码。然后再由基于堆栈的虚拟机(CPU)对伪代码解释执行。  
指令系统  
     关键在于设计一个虚拟的基于堆栈的虚拟机(CPU)的指令系统。这个指令系统越简洁，复用性越高越好。  
     还是以 add 指令为例。X86 系列 CPU 的 add 指令有许多格式，譬如：add reg,imm 、add reg,reg、add reg,mem、add mem,reg 等等。而基于堆栈的虚拟机 CPU 则没有这么多花样，就一个单单的 add 指令，参数和返回都是在堆栈里。  
     我们需要为我们的虚拟机 CPU 模拟实现这样的 add 命令：  
Vadd:                            ;virtual add  
        Mov eax,\[esp+4\]          ;取源操作数  
        Mov ebx,\[esp\]            ;取目的操作数  
        Add ebx,eax              ;  
        Add esp,8                ;把参数从堆栈中删掉，平衡堆栈  
        Push ebx                 ;把结果压入堆栈  
      而原有的 add 命令的参数，我们需要翻译为 push 命令 。根据 push 的对象不同，需要不同的实现：  
vPushReg32:                      ;寄存器入栈 。esi 指向字节码的内存地址  
      Mov eax,dword ptr\[esi\]     ;从伪代码(字节码)中获得寄存器在 VMcontext 结构中的偏移地址  
      Add esi,4                  ;VMcontext 结构保存了各个寄存器的值。该结构保存在堆栈内。  
      Mov eax,dowrd ptr \[edi +eax\]   ;得到寄存器的值。edi 指向 VMcontext 结构的基址  
      Push eax                    ;压入堆栈  
      Jmp VMDispatcher            ;任务完成,跳回任务分派点  
vPushImm32:                       ;立即数入栈  
      Mov eax,dword ptr\[esi\]      ;字节码，不用翻译就是了  
      Add esi,4  
      Push eax                    ;立即数入栈  
      Jmp VMDispatcher  
     有 Push 指令了，也得有 Pop 指令：  
vPopReg32:  
      Mov eax,dword,ptr\[esi\]     ;从伪代码(字节码)中获得寄存器在 VMcontext 结构中的偏移地址  
      Add esi,4  
      Pop dword ptr \[edi+eax\]    ;弹回寄存器  
      Jmp VMDispatcher  
  基于堆栈的虚拟机指令系统就是这样简单的：单字节的动作指令（譬如 add、dec），以及各式各样的 push、pop 等堆栈操作指令。没有复杂的寄存器与内存操作。我们需要把 x86 的 CPU 指令翻译成虚拟机 CPU 的指令，譬如：  
     Add esi,eax  
  转换为虚拟机的指令如下:  
     vPushReg32 eax\_index  
     vPushReg32 esi\_index  
     Vadd  
     vPopReg32 esi\_index      ;不弹 eax\_index，它作为返回结果保存在堆栈里  
  涉及跳转的指令：(譬如 Jmp、Call、Ret)  
  Esi 指向当前伪代码的地址，就好比我们虚拟机 CPU 的 IP 寄存器，指向当前指令的地址。  
  可以通过修改 Esi 来改变程序执行的流程。譬如：  
Vjmp:  
     Mov esi,dword ptr \[esp\]   ;jmp 的参数在堆栈中  
     Add esp,4                 ;平衡堆栈  
     Jmp VMDispatcher  
  Call 指令相对麻烦一点，因为 Call 的函数未必是虚拟机的伪代码了。所以碰到 Call 指令，就要退出虚拟机，交由真实的 CPU 去处理了。代码类似下面：  
Vcall:  
     Push all vreg            ;所有的虚拟的寄存器 (维护在堆栈中)  
     Pop  all reg             ;弹出到真实的寄存器中(保存虚拟机的运行结果)  
     Push 返回地址            ;可以让 Call 调用完成后，把控制权归还给虚拟机。详细见原书第 480 页。  
     Push 要调用的函数地址  
     Retn  
    以下是 vRetn:  
vRetn:  
     Xor eax,eax  
     Mov ax,word ptr \[esi\]     ;retn 的操作数是 WORD 类型的  
     Add esi,2                 ;  
     Mov ebx,dword ptr\[ebp\]    ;得到要返回的地址   
     Add ebp,4                 ;释放空间  
     Add ebp,eax               ;如果有操作数，同样释放  
     Push ebx                  ;压入返回地址  
     Push ebp                  ;压入堆栈指针  
     Push \[edi + 0x1c\]         ;逐个压入虚拟寄存器的值  
     Push \[edi + 0x18\]  
     Push \[edi + 0x14\]  
     Push \[edi + 0x10\]  
     Push \[edi + 0x0c\]  
     Push \[edi + 0x08\]  
     Push \[edi + 0x04\]  
     Push \[edi\]  
     pop eax                   ;弹出至寄存器  
     pop ebx  
     pop ecx  
     pop edx  
     pop esi  
     pop edi  
     pop ebp  
     Popfd  
     Pop esp                   ;还原堆栈指针  
     retn  
  
           
  其余，要注意标志位的处理、不可模拟指令，以及指令的优化。还有异常处理，这里就不展开了。  
     
  
     VStartVM 是虚拟机的入口，负责保存运行环境(各个寄存器的值)、以及初始化堆栈(虚拟机使用的变量全部在堆栈中)。  
     Bytecode 是伪代码；VMDispatcher 对伪代码逐个阅读处理，然后分发给下面的各个子程序(Handler)。  
     加壳程序先把已知的 X86 指令解释成了字节码，放在 PE 文件中，然后将原处代码删掉，  
   改成类似的代码进入虚拟机执行循环。  
     push bytecode         ;伪代码地址 ，作为参数  
     jmp   VstartVM  
     VstartVM 就是进入虚拟机的函数，它的代码大概类似这样的：  
VStartVM:  
   push eax   ;将寄存器压入堆栈,后面会由伪指令取出并存放到 VMContext 中;此处也可以加入一些随机特征用于迷惑  
   push ebx  
   push ecx  
   push edx  
   push esi  
   push edi  
   push ebp  
   pushfd  
   mov esi ,\[esp+0x20\]   ;esp+0x20 指向 VStartVM 的参数,即伪代码的内存地址  
   mov ebp,esp           ;ebp 指向当前堆栈  
   Sub esp,0x200         ;在堆栈中开辟 0x200 字节,存放 VMcontext  
   mov edi,esp           ;edi 指向 VMcontext  
   Sub esp,0x40          ;到这里，才到达 VM 真正使用的堆栈，不一定非要 0x40 字节  
VMDispatcher:  
  Mov eax,byte ptr\[esi\]  ;获得伪代码 bytecode  
  Lea esi,\[esi+1\]           
  Jmp dword ptr \[eax\*4+JumpAddr\];跳到 Handler 执行处，由加壳引擎填充  
                                ;每读一个 byte 就跳到函数表模拟执行代码。(堆栈型 CPU 的指令短，1 字节足够)  
                                  ;JUMPADDR 就是一张函数表（有点类似 VTBL 或者 switch-case 表），  
VM\_END                            ;VM 结束标记  
VStartVM 初始化后，堆栈情形如下：  
  
  堆栈低地址...  
  ... --------  
  ...    |  
  ... 0x40 字节的空间，VM 使用的堆栈空间（不一定非要 0x40 大小）  
  ...    |  
  ... --------                               edi 指向这里(VMcontext)  
  ...    |  
  ...  0x200 字节的空间，用于存放 VMcontext  
  ...    |  
  ... --------                                
  flags  保存的标志寄存器                     ebp 指向这里  
  ebp --------  
  edi     |  
  esi     |  
  edx    保存的寄存器  
  ecx     |  
  ebx     |  
  eax --------  
  伪代码开始地址(虚拟机参数)                  esi 指向这里(伪代码地址)  
  ...  
  堆栈高地址...  
    
  edi 指向 VMcontext;esi 指向伪代码的地址；ebp 指向真实堆栈的栈顶; 这三个寄存器在 VM 内不要再改了。  
  VMContext 是虚拟机 VM 使用的虚拟环境结构：  
     Struct VMContext  
       {  
            DWORD v\_eax;  
            DWORD v\_ebx;  
            DWORD v\_ecx;  
            DWORD v\_edx;  
            DWORD v\_esi;  
            DWORD v\_edi;  
            DWORD v\_ebp;  
            DWORD v\_efl;  
        }  
    VM 之所以使用堆栈保存自己的寄存器结构，是考虑到多线程程序的兼容。  
    大家都知道脱壳时的堆栈平衡原理吧。同样的，虚拟机在执行翻译后的程序代码时，也不可以随便变动原来的堆栈地址。还需经常检查在堆栈中的 VMcontext 结构不被冲掉。  
  
VCheckESP:  
  Lea eax,dword ptr \[edi+0x100\]  
  Cmp edi,ebp                     ;比较 ebp 与 edi(VMContext)之间的距离  
  Jl VMDispatcher                 ;如果小于就继续执行  
  Mov edx,edi  
  Mov ecx,esp  
  Sub ecx,edx                     ;ecx = esp - edi  
  Push esi                        ;保存 esi  
  Mov esi,esp  
  Sub esp,0x60  
  Mov edi,esp                     ;edi = esp - 0x60   
  Push edi                        ;保存新的 edi 地址  
  Sub esp,0x40  
  Cld   
  Rep movsb                        ;复制  
  Pop edi  
  Pop esi  
    Jmp VMDispatcher