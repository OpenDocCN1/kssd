

# Armadillo Strategic Code Splicing 手修经验小结

> 原文：[`www.pediy.com/kssd/pediy10/58221.html`](https://www.pediy.com/kssd/pediy10/58221.html)

Armadillo Strategic Code Splicing 手修经验小结  
  
（本文以 PassDiskProtect\_C.exe 为例）  
  
一、为什么  
  
    首先，我想说，如果你只是为了脱壳。那你不需要继续看下去。  
    那为什么要修复这个呢？当你用 IDA 分析某个带有这种保护的脱壳文件的时候你就会非常想这样做，尤其是你选择使用 HexRays 功能的时候，当我们想对 Main 函数查看源代码的时候，我们就会得到一个错误，而无法查看。或者对代码段第一个过程查看源代码的时候，显示的结果会让你非常的失望。而修复了之后，一切都变得很完美。  
  
二、我们需要什么  
  
    首先，你需要痛恨 Arm 的这项保护。因为像这种 300K 不到的程序，Arm 改写了 3000 多处，意味着手动修复将花费你大量的时间。我现在大约可以 300 处/小时的样子，希望有编程高人据此文写出辅助工具，造福我们。  
    其次，你的文件必须起始于 OEP 处，不管他此时是在内存中还是在脱壳后的文件中。  
    再次，调整你的 OD，将寄存器框缩小，堆栈框缩小。将内存框和代码框放到相似的大小。代码框定位到代码段的起始，内存框定位到被移走的代码区的起始，并调整为反汇编模式。  
  
三、我们开始吧  
  
    具体过程没什么可说的，就是在内存框中找出每一句正确的代码，然后 Shift-C，在代码框中正确的位置 Shift-V。我们这里主要说下，怎么区分垃圾代码。  
  
    首先，可以确定的是，一切正确的代码都没有被变形。我猜测这是由于 Arm 采取的是客户端程序加密，由于目标程序的不可确定性，所以很多地方他不敢做的太过分。  
    第二，一切有关内存数据的读写操作语句都是正确的。  
    第三，一切 mov reg1,reg2 和 xchg reg1,reg2 语句在 reg1=reg2 的时候，都是垃圾语句。  
    第四，一切前面有对应的 push reg 语句的 pop reg 连同之间的语句全部是垃圾语句。没有对应的 push reg/pop reg 都是正确的。  
    第五，成对的语句及他们之间的语句都是垃圾语句，包括 not reg；xchg reg1,reg2；bswap reg。  
    第六，一切的调转语句，除了调向程序代码段的，都是垃圾语句。  
    第七，Arm 不会偷 Call 语句，相反，Call 语句的上下合每个过程的开头和结束地点是最容易被偷的。  
    第八，Arm 在偷过代码的地方使用 Jmp Long 原句调转语句，再加上垃圾代码块，补足被偷的代码长度。所以，偷走的代码最少有 5 个字节，所以如果你看见一个单一的 Jmp，再对应的块中发现了 Push Imm32(Imm32>0x7f)，mov reg,Imm32 则被偷的必然是这一句，当然也可以是 1\*5，2+3，1+4 之类的。还有一种情况是 Jmp+nop 的，这样则必然是 6 字节，因为 Jmp 占据 5 字节，剩余的字节用 Nop 无疑是最安全的。  
    第九、垃圾代码以块的方式存在，意味着一块的垃圾代码中不会有任何的正确代码。同样也意味着垃圾代码后面的远距调转可以跳向代码段中 Jmp 之后到正常代码之间的任何垃圾块的起始。  
    第十，被偷走的代码块中，JmpLong 之前的语句貌似都是正确的（这个不敢确定，因为如果以后 Arm 在这边也插一点垃圾并不会影响程序的正确运行……）  
  
    文字就这么多了，我们来看些代码吧。  
  
00521000    6A FF           push    -1    正确  
00521002    87F3            xchg    ebx, esi  对应  
00521004    87F1            xchg    ecx, esi  
00521006    66:93           xchg    ax, bx  
00521008    67:E3 00        jcxz    short 0052100B  
0052100B    66:93           xchg    ax, bx  
0052100D    87F1            xchg    ecx, esi  
0052100F    7C 00           jl      short 00521011  
00521011    87F3            xchg    ebx, esi  对应  
00521013    68 78824200     push    00428278  正确  
00521018  - E9 E8FFEDFF     jmp     00401005  
  
0052101D    50              push    eax        正确  
0052101E    8D4424 08       lea     eax, dword ptr \[esp+8\]  正确  
00521022    64:A3 00000000  mov     dword ptr fs:\[0\], eax  正确  
00521028    66:93           xchg    ax, bx      对应  
0052102A    50              push    eax  
0052102B    75 00           jnz     short 0052102D  
0052102D    58              pop     eax  
0052102E    8BF6            mov     esi, esi  
00521030    66:93           xchg    ax, bx      对应  
00521032    C74424 10 00000>mov     dword ptr \[esp+10\], 0  正确  
0052103A  - E9 DCFFEDFF     jmp     0040101B  
  
00521159    6A 00           push    0        正确  
0052115B    6A 03           push    3        正确  
0052115D    6A 00           push    0        正确  
0052115F    6A 02           push    2        正确  
00521161    68 00000080     push    80000000      正确  
00521166    87CB            xchg    ebx, ecx      对应  
00521168    79 02           jns     short 0052116C  
0052116A    79 03           jns     short 0052116F  
0052116C    77 00           ja      short 0052116E  
0052116E    87CB            xchg    ebx, ecx  
00521170    50              push    eax  
00521171  - E9 8AFFEDFF     jmp     00401100  
00521176    93              xchg    eax, ebx  
00521177    F7D2            not     edx  
00521179    66:87CB         xchg    bx, cx  
0052117C    66:87CB         xchg    bx, cx  
0052117F    F7D2            not     edx  
00521181    8BDB            mov     ebx, ebx  
00521183    93              xchg    eax, ebx      对应  
00521184    A3 1C6F4300     mov     dword ptr \[436F1C\], eax  正确  
00521189    0F95C0          setne   al        正确  
0052118C  - E9 86FFEDFF     jmp     00401117  
  
00521328    97              xchg    eax, edi      对应  
00521329    78 00           js      short 0052132B  
0052132B    66:87D3         xchg    bx, dx  
0052132E    91              xchg    eax, ecx  
0052132F    87F1            xchg    ecx, esi  
00521331    0FCB            bswap   ebx  
00521333    87C9            xchg    ecx, ecx  
00521335    0FCB            bswap   ebx  
00521337    76 00           jbe     short 00521339  
00521339    87F1            xchg    ecx, esi  
0052133B    91              xchg    eax, ecx  
0052133C    66:87D3         xchg    bx, dx  
0052133F    97              xchg    eax, edi      对应  
00521340    68 81874200     push    00428781      正确  
00521345  - E9 ADFFEDFF     jmp     004012F7  
  
00521374    50              push    eax        对应  
00521375    97              xchg    eax, edi  
00521376    66:87F2         xchg    dx, si  
00521379    97              xchg    eax, edi  
0052137A    66:87DB         xchg    bx, bx  
0052137D    97              xchg    eax, edi  
0052137E    66:87F2         xchg    dx, si  
00521381    97              xchg    eax, edi  
00521382    58              pop     eax        对应  
00521383    899C24 E4000000 mov     dword ptr \[esp+E4\], ebx  正确  
0052138A  - E9 A6FFEDFF     jmp     00401335  
  
00522C1A    66:92           xchg    ax, dx      对应  
00522C1C    0FCE            bswap   esi  
00522C1E    0FCE            bswap   esi  
00522C20    66:92           xchg    ax, dx      对应  
00522C22    8BDB            mov     ebx, ebx      注意！这句是垃圾语句  
00522C24    68 B4CD4200     push    0042CDB4      正确  
00522C29    8D4C24 24       lea     ecx, dword ptr \[esp+24\]  正确  
  
005260E8    FF75 0C         push    dword ptr \[ebp+C\]    正确  
005260EB    8D45 F0         lea     eax, dword ptr \[ebp-10\]  正确  
005260EE    8BDB            mov     ebx, ebx      注意！这句是垃圾语句  
005260F0    50              push    eax        正确  
005260F1  - E9 F752EEFF     jmp     0040B3ED  
  
例子就举这么多吧，差不多都能涵盖到了。