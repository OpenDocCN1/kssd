

# 长度反汇编引擎的打造

> 原文：[`www.pediy.com/kssd/pediy10/87915.html`](https://www.pediy.com/kssd/pediy10/87915.html)

长度反汇编引擎(Length-Disassembler Engine)的打造  
  
  
  忙，好忙。利用晚上下班的时间给大家赶出篇文章来。貌似好长时间没有睡过好觉了。这个星期还要回趟家乡去.... \*\*\*\* 总是时间太少........  
  
   这个星期先给大家来一篇 Virus 编写中经常要用到的长度反汇编引擎 Length-Disassembler Engine 的编写思路文章。因为它在 Virus 编写中也是十分重要。我们病毒变形、感染很多时候都要用到它。   
  
  
  长度反汇编引擎(Length-Disassembler Engine)是用于获取目标地址所在代码的长度。举个例子 push ebp | mov ebp, esp 。我想大家对这两句代码并不陌生了，一般被用于我们的子程序中建  
  
立堆栈框架。那么这两句指令的长度是不同的，有时候我们要替换某内存地址中的指令的话，如果我们茫然的替换，只能把稳定性给大大的抛弃了。所以这个时候就需要我们的长度反汇编引擎来获取我  
  
们要替换地址所在指令的长度，如果小于或者等于替换指令的长度，我们则可以进行替换。假设我们通过 Length-Disassembler Engine 获取 push ebp 所在的地址的代码长度，那么返回的长度肯定是 1。  
那么目前我所见到过非常短小精炼的长度反汇编引擎就是 rgblde，300 bytes+  啊？有点扯远了我们继续主题。   
  
比如把  
  
Oep:  
push ebp  ;1 bytes  $55  
mov ebp, esp   ;2 bytes  $8B, $EC  
  
换成  
Oep:  
push ebp  ;1 bytes  $55  
push esi  ;1 bytes  $56  
mov  ebp, esp  ;2 bytes  $8B, $EC  
  
假如我们在替换指令的时候, 我们的程序按照之前的替换方式。从 mov ebp, esp 地址处开始替换。因为此时这个地址是我们的指令 push esi，(假设替换指令是 jmp shor Oep(2 bytes) - $EB $FD)。那  
  
么替换后形成  
Oep:  
push ebp  ;1 bytes  $55  
jmp  short Oep  ;2 bytes  $EB $FD  
    ;1 bytes  $EC  
  
那么此时如果替换后的话，并执行的话肯定会出现异常的。还有平常我们的 inline Hook 例如那常用的 5 字节 替换(jmp long xxxx)，如果我们不通过 Length-Disassembler Engine 长度取下要替换目标地  
  
址的指令长度是否等于 5 字节就茫然替换的话这个程序的稳定性就太欠缺了。以上就是我们 Length-Disassembler Engine 的用处。它在病毒中也常常要用，例如用到 EPO，例如搜索代码段的 5 字节指令进行  
  
替换，或者用到变形等。  
  
  
  
OK。了解了用处我们就来实现吧。  
  
  
首先要编写长度反汇编引擎  
    
  
                     intel 指令格式  
  
+-------------+--------+----------+---------+--------------+------------+  
; instruction ; opcode ;  ModR/M  ;   SIB   ; Displacement ; Immediate  ;  
;   prefixe   ;        ;          ;         ;              ;            ;  
+-------------+--------+----------+---------+--------------+------------+  
    前缀(可选)   操作码  （可选）    （可选）  地址偏移(可选) 立即数(可选)  
  
  
这里 opcode 成员是必需的，其他 5 个成员是可选的。但是千万切记它们的顺序是不能颠倒的。sib 我们可以认为是 mod/rm 的扩展。只要有 sib 成员那么 mod/rm 也是必需的。  
  
在这里我不想太多的去介绍 Intel 指令格式，因为要介绍它我估计得另立一个专题的讲。幸好论坛也有朋友做过此类的系列课程。所以与其我给大家简要讲解，大家还不如去先学习下这些相关的专题。  
  
如我们论坛   
  
egogg 的打造自己的反汇编引擎 Intel 指令编码学习报告专题。  
  
  
我这里主要将部分的重点内容给大家标记下。  
  
前缀：  
   1. 前缀是唯一的一个可能出现在 Code 之前的域   
  
   2. 所有的 Prefixes 都只有一个字节   
  
   3. 在一个 opcode 中可能会有多个 Prefixes   
  
   4. 如果 Prefixes 不能对随它之后的 opcode 起作用,那么处理器将忽略它。  
  
   5. 一条指令可能只有一个 CODE 域,一个 mod r/m 域,或者一个 offset 域等,但是可以有多个 Prefixes.    
  
  
Prefixes 被分为:   
1.  切换默认操作数大小(66h)   
  
2.  切换默认地址大小(67h)   
  
3.  重复(Rep)(F3h,F2h)   
  
4.  切换默认段(2eh,36h,3eh,26h,64h,65h)   
  
5.  总线锁定(Buslock)(F0)   
  
  
ModR/M  
涉及内存操作数的指令都有一个紧挨着主操作码的寻址格式说明字节也就是 ModR/M。你应该知道 ModR/m 什么时候该用，什么时候不该用了吧。  
  
  
SIB  
ModR/M 字节编码需要第二寻址字节(SIB).基址+索引或者比例+索引形式的 32 位寻址则需要 SIB 字节成员。  
  
  
Displacement  Immediate  
这两个成员我就不说了吧。  
  
  
OK。了解了结构，我们来说说如何编写。  
  
不知道大家是否用过 opcode 表。  
我们每一个汇编助记符就相当于 opcode 表的索引。举个例子  
pushad - 060h  
  
那么我们假设我们将如 pushad, dec reg 等单字节指令自己定义一个数值，例如 1 来表示它是一个单字节指令。  
  
  那么我们的反汇编引擎读取目标地址的机器码后，通过 opcode 表偏移地址+机器码来索引我们机器码在表中的对应成员，例如我们的 pushad 如果取出的对应成员值是 1 的话，我们则知道我们的指令是一  
  
个单字节指令。那么我们就可以直接将 size +1，然后返回过程，这样我们的长度反汇编过程返回的就是 1.  
  
  这就是我们长度反汇编引擎的过程。  
  
  当然如果都是单字节指令的话我们也就不用去学上面的 intel 格式了。  
  
  其实清楚了上面的 pushad 的长度反汇编引擎的工作原理，那么接下来的解码工作也是很简单。  
  
  例如通过 opcode 表偏移地址+机器码来索引我们机器码在表中的对应成员，然后通过对成员进行判断。如果是前缀的话，则跳转到我们的前缀处理过程处去执行。如果是存在 ModR/m 的指令则跳转到我们  
  
的 ModR/m 过程去执行，其他的亦然。由于我们的指令成员字节数是可以确定，所以只要通过对指令结构成员进行解码确定我们的目标地址代码到底存在哪些成员，就可以确定目标地址代码的长度。  
  
其实说到底还是要求你对 intel 指令格式的掌握。所以看不懂的朋友还是尽量先把基础补补。  
  
  
**我这里主要说说引擎的大小优化。**  
  
 其实之前看到很多作者在设计表的时候居然用 dword 类型来初始化，这大大增加我们 opcode 表的字节大小。后期有的人采用了 byte 来初始化。NONO，这都不是我们想要的。我们可以采用 4bit 位的方式来  
定义，这样大大简化我们的 opcode 表字节大小。恩，没错。这个方式最早被 29a 的 sars 所使用。我们今天也是学习的 29a - sars 的 Catchy32 代码思路。那么我们在通过 4bit 位来定义成员，我们还是要讲究  
  
点技巧，这样后面为我们提供方便。  
  
  
1h 2h 4h 8h 我们通过每次\*2 的形式来定义(还是否记得逻辑左移指令)。没错也就是每次 shl 1 位。为什么要这样？  
我们这样定义的话，我们就省略了 cmp 判断了。直接通过 btr 来测试我们二进制位并影响 CF 标志位，然后通过 jc 分支跳转 来完成我们的判断，是不是很 GOOD。  
  
完成后的 Opcode 表如下。  
 ;--------------------Opcode Table--------------------    
  
 ;++   
 ;Description:  
 ;Size of table element is 4 bits.  
 ;0h-one byte instruction  
 ;1h-ModRM byte  
 ;2h-imm8,rel8 etc  
 ;4h-ptr16 etc  
 ;8h-imm16/32,rel16/32 etc  
 ;0Fh-prefix  
 ;0Eh-unsupported opcodes  
 ;--  
  
pref66h equ 1  
pref67h equ 2  
Table:  
;    01  23    45   67   89   AB   CD   EF  
db 011h,011h,028h,000h,011h,011h,028h,000h;0Fh  
db 011h,011h,028h,000h,011h,011h,028h,000h;1Fh  
db 011h,011h,028h,0F0h,011h,011h,028h,0F0h;2Fh  
db 011h,011h,028h,0F0h,011h,011h,028h,0F0h;3Fh  
db 000h,000h,000h,000h,000h,000h,000h,000h;4Fh  
db 000h,000h,000h,000h,000h,000h,000h,000h;5Fh  
db 000h,011h,0FFh,0FFh,089h,023h,000h,000h;6Fh  
db 022h,022h,022h,022h,022h,022h,022h,022h;7Fh  
db 039h,033h,011h,011h,011h,011h,011h,011h;8Fh  
db 000h,000h,000h,000h,000h,0C0h,000h,000h;9Fh  
db 088h,088h,000h,000h,028h,000h,000h,000h;AFh  
db 022h,022h,022h,022h,088h,088h,088h,088h;BFh  
db 033h,040h,011h,039h,060h,040h,002h,000h;CFh  
db 011h,011h,022h,000h,011h,011h,011h,011h;DFh  
db 022h,022h,022h,022h,088h,0C2h,000h,000h;EFh  
db 0F0h,0FFh,000h,011h,000h,000h,000h,011h;FFh  
;==============================================  
Lentable equ $-Table  
;===============EXTENDED OPCODES===============  
TableExt:  
;    01  23    45   67   89   AB   CD   EF  
db 011h,011h,0E0h,000h,000h,0EEh,0E1h,003h;0Fh  
db 011h,011h,011h,011h,01Eh,0EEh,0EEh,0EEh;1Fh  
db 011h,011h,01Eh,01Eh,011h,011h,011h,011h;2Fh  
db 000h,000h,000h,0EEh,0EEh,0EEh,0EEh,0EEh;3Fh  
db 011h,011h,011h,011h,011h,011h,011h,011h;4Fh  
db 011h,011h,011h,011h,011h,011h,011h,011h;5Fh  
db 011h,011h,011h,011h,011h,011h,011h,011h;6Fh  
db 033h,033h,011h,010h,011h,011h,011h,011h;7Fh  
db 088h,088h,088h,088h,088h,088h,088h,088h;8Fh  
db 011h,011h,011h,011h,011h,011h,011h,011h;9Fh  
db 000h,001h,031h,011h,000h,001h,031h,011h;AFh  
db 011h,011h,011h,011h,0EEh,031h,011h,011h;BFh  
db 011h,031h,033h,031h,000h,000h,000h,000h;CFh  
db 0E1h,011h,011h,011h,011h,011h,011h,011h;DFh  
db 011h,011h,011h,011h,011h,011h,011h,011h;EFh  
db 0E1h,011h,011h,011h,011h,011h,011h,01Eh;FFh  
;==============================================  
  
OK，这里给大家贴一段我参考 29a - sars 的 Catchy32 代码思路写的一段长度反汇编引擎,我去掉了些不必要的代码, 并且以上我已经说了这个符号表以及这个引擎的判断方式，相信理解了以上，你看代码应该没有问题了。。它处理完重定位 大概应该在 500 byte  - 600 byte 之间。抱歉原谅我不喜欢静态库的方式来移植到高级语言编译器中，我还是喜欢 shellcode 方式。。  
  
  

代码:

```


  format PE GUI 4.0
  include 'win32ax.inc'
  
.text
  
 entry  $
 
   mov  esi, Lde32
   push  esi
   call  Lde32
   add  esi, eax
   jmp  $-13
   ret
  
  

 ;++
 ;
 ; Int
 ;   Lde32(
 ;   IN byte \*pDestAddress   
 ;   )
 ;
 ; Routine Description:
 ;
 ;    获取目标地址代码的长度
 ;
 ; Thanks sars .
 ;
 ; Arguments:
 ;
 ;    (esp)            - return address
 ;
 ;    Data  (esp+4\*8+4) - pDestAddress
 ;
 ; Return Value:
 ;
 ;    eax = TRUE, initialization succeeds . = OpCode Length
 ;    eax = -1,   Faild
 ;--
 
 include 'optable.inc'
 Lde32:
  pushad
  xor  ecx, ecx
  mov  esi, \[esp+4\*8+4\]
  
 Lde\_ExtFlags:
   xor  eax, eax
   xor  ebx, ebx
   cdq
  lodsb
  mov  cl, al
  cmp  al, 0fh
  je  Lde\_ExtdTable
   jmp  Lde\_NormTable
   
 Lde\_ExtdTable:        ;Load flags from extended table
  lodsb
  inc   ah      ;EAX=al+100h (100h/2 - lenght first table)

 Lde\_NormTable:        ;Load flags from normal table
  shr  eax, 1      ;Div 2
  mov  al, byte \[Table+eax\]
  
  jc  Lde\_CheckC1    ;如果是奇数则取低 4 位
  shr  eax, 4      ;Get Hight high 4-bits

 Lde\_CheckC1:        
   and  eax, 0Fh    ;...low
   xchg  eax, ebx
   
 ;--------------Opcode type checking---------------
 Lde\_CheckFlags:
   cmp  bl, 0Eh      ;unsupported opcodes
   je  Lde\_Error
   cmp  bl, 0Fh      ;test prefix
   je  Lde\_Prefix
   or  ebx, ebx    ;Test One byte command   
   jz  Lde\_GetLen
   btr  ebx, 0      ;Test ModRM byte
   jc  Lde\_ModRM    
   btr  ebx, 1      ;Test imm8,rel8 etc
   jc  Lde\_incr1
   btr  ebx, 2      ;Test ptr16 etc
   jc  Lde\_incr2
 ;-----imm16/32,rel16/32, etc types processing-----
 Lde\_16\_32:
   and   bl, 11110111b      ;Reset 16/32 sign 
   
   cmp   cl, 0A0h    ;Processing group 0A0h-0A3h
  jb  Lde\_Check66h
  cmp  cl, 0A3h
  ja  Lde\_Check66h
  test  ch, pref67h    
  jnz  Lde\_incr2
  jmp  Lde\_incr4
    
 Lde\_Check66h:        ;Processing other groups
  test   ch, pref66h      ;pref66h                    
  jz   Lde\_incr4                            
  jmp   Lde\_incr2                            
 
 ;---------------Prefixes processing---------------
 Lde\_Prefix:
   cmp  cl, 66h
   je  Lde\_SetFlag66h
   cmp  cl, 67h
   jne  Lde\_ExtFlags
   
  Lde\_SetFlag67h:
    or  ch, pref67h  
    jmp  Lde\_ExtFlags
    
  Lde\_SetFlag66h:
   or  ch, pref66h
   jmp  Lde\_ExtFlags
   
 ;--------------ModR/M byte processing-------------
 Lde\_ModRM:
   lodsb
   
   cmp  cl, 0F7h
   je  Lde\_GroupF6F7
   cmp  cl, 0F6h
   jnz  Lde\_Modxx
   
 Lde\_GroupF6F7:
   test   al, 00111000b  
  jnz   Lde\_Modxx
   test   cl, 00000001b
  jz  Lde\_incbt1      
  test  ch, pref66h
  jnz  Lde\_incbt2  
  inc   esi
  inc   esi
 Lde\_incbt2:  inc   esi
 Lde\_incbt1:  inc   esi
   
 Lde\_Modxx:
   mov   edx, eax
  and   al, 00000111b    ;al <- only R/M bits
  test  dl, 11000000b    ;Check MOD bits
  jz    Lde\_Mod00
  jp    Lde\_CheckFlags    ;Or Lde\_Mod11
  js    Lde\_Mod10
 
 Lde\_Mod01:
  test   ch, pref67h
  jnz   Lde\_incr1     ;16-bit addressing
  cmp   al, 4      ;Check SIB
  je   Lde\_incr2
  jmp   Lde\_incr1

 Lde\_Mod00:
  test   ch, pref67h    ;pref67h
  jz   Lde\_Mod00\_32    ;32-bit addressing
  cmp   al, 6
  je   Lde\_incr2
  jmp   Lde\_CheckFlags  
 Lde\_Mod00\_32:
  cmp   al, 4      ;Check SIB
  jne   Lde\_disp32

 Lde\_SIB:        ;Processing SIB byte
  lodsb
  and   al, 00000111b
  cmp   al, 5
  je   Lde\_incr4
  jmp   Lde\_CheckFlags  

 Lde\_disp32:  
  cmp   al, 5
  je   Lde\_incr4
  jmp   Lde\_CheckFlags 

 Lde\_Mod10:
  test   ch, pref67h
  jnz   Lde\_incr2    ;16-bit addressing
  cmp   al, 4      ;Check SIB
  je   Lde\_incr5
  jmp   Lde\_incr4

 Lde\_incr5:  inc   esi
 Lde\_incr4:  inc   esi
  inc   esi
 Lde\_incr2:  inc   esi
 Lde\_incr1:  inc   esi
  jmp   Lde\_CheckFlags  
 
 ;-----------Command length calculation------------  
 Lde\_GetLen:
   sub  esi, \[esp+4\*8+4\]
   mov  \[esp+pushad\_eax\], esi
   jmp  Lde\_Ret
   
   
 ;-----------Setting the error-----------------------
 Lde\_Error:
   xor  eax, eax
   dec  eax
   mov  \[esp+pushad\_eax\], eax
    
 Lde\_Ret:
   popad
   retn 4\*1

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Lde.rar  
 |