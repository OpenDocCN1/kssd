
# Themida 的 VM 分析－－变形代码清理

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-649.htm`](https://www.pediy.com/kssd/pediy08/pediy8-649.htm)

这是我的部分笔记，没有完成,供参考。可能最近几个月没时间搞了。

## 1. 变形代码长啥样?

变形代码在壳里用得越来越多,这种源于病毒的技巧被用来掩盖代码的真实目的,以及对 `cracker` 进行身心折磨:-(。变形代码不象普通的花指令那样,用简单的字符串匹配即可清理(其生成过程也不同,花指令一般以在编程时插入宏的方式创建)。

下面的实例来自 `Themida` 虚拟机代码:

```assembly
01596EEF  push    edx      
01596EF0  add     ebx, ecx *     
01596EF2  mov     edx, 0     
01596EF7  mov     bh, 35h  *   
01596EF9  add     edx, edi         
01596EFB  shr     al, 3  *           
01596EFE  push    ecx          
01596EFF  sub     eax, ebx  *     
01596F01  jmp     loc_14F562A      
014F562A  mov     ecx, 7445h       
014F562F  neg     ecx          
014F5631  not     ecx          
014F5633  not     ecx          
014F5635  or      ecx, 629Ah       
014F563B  jmp     loc_14FA94D      
014FA94D  sub     al, dl  *         
014FA94F  or      ecx, 205Ch       
014FA955  xor     ecx, 0FFFFEBFFh 
014FA95B  mov     bl, 8Ah  *   
014FA95D  add     ecx, edx         
014FA95F  add     ah, dh     *   
014FA961  jmp     loc_1594A3C      
01594A3C  mov     esi, [ecx]       
01594A3E  mov     bl, ch  *   
01594A40  pop     ecx          
01594A41  mov     ebx, 7570h  *   
01594A46  mov     edx, [esp]    
01594A49  add     esp, 4   
```

这段代码非常典型,我们可以尝试手工清理。其中的 3 个 `jmp` 是所谓的置换 `jmp`,即人为将代码分块,打乱其物理存放位置,再用 `JMP` 指令链接。

除了 `JMP` 指令,其中还夹杂了一些垃圾指令(带`*`号的行,从代码的逻辑可以识别),拿掉这些 `JMP` 和垃圾指令后剩下:

```assembly
01596EEF  push    edx      
01596EF2  mov     edx, 0     
01596EF9  add     edx, edi         
01596EFE  push    ecx          
014F562A  mov     ecx, 7445h       
014F562F  neg     ecx          
014F5631  not     ecx          
014F5633  not     ecx          
014F5635  or      ecx, 629Ah       
014FA94F  or      ecx, 205Ch       
014FA955  xor     ecx, 0FFFFEBFFh 
014FA95D  add     ecx, edx         
01594A3C  mov     esi, [ecx]       
01594A40  pop     ecx          
01594A46  mov     edx, [esp]    
01594A49  add     esp, 4
```

`014F562A` - `014FA955` 对 `ecx` 的赋值及计算,其结果为 0,可以压缩为 1 句,现在很容易看出代码的真正含义了:

```text
01596EEF  push    edx  ----------------------\     
01596EF2  mov     edx, 0                     \
01596EF9  add     edx, edi                    \
01596EFE  push    ecx  -------\               \
014F562A  mov     ecx, 0     \       \
014FA95D  add     ecx, edx     mov esi,[edx]   mov esi,[edi]
01594A3C  mov     esi, [ecx]  /               /
01594A40  pop     ecx  -------/              /
01594A46  mov     edx, [esp]---  pop edx  ----/
01594A49  add     esp, 4  -----/
```

这里实际使用了 2 种代码模式:

`MOV Reg,[ESP]`/`ADD ESP,4` -> `POP Reg`

`PUSH Reg1`/`MOV Reg1,Imm`/`ADD Reg1,Reg2`/`Operate Reg3,Mem`/`POP Reg1` -> `Operate Reg3,[Reg2+Imm]`

(这里 `Reg1` 用来生成内存地址)

## 2. 怎样生成变形代码?

置换 `JMP` 的生成过程如下,将代码随机划分为片断,例如每段 1-10 行,用数据结构(如 `List`)管理代码块信息,然后混洗,以下为伪码:

```text
ESI = Initial address of instructions
EDI = Address of last instruction

; 用第 1-60h 行代码创建若干代码片段,每段 8-16 行
Given ESI = 00000000h, 
    EDI = 00000060h
while(ESI < EDI)
   Store ESI
   ESI += Random(8)+8
   Store ESI
   if((ESI+0F > EDI)
      Store ESI,EDI
      break;
   end if
end while

Result (for example):
   DD    00000000h,0000000Ah  <- 1 (第 0-10 行)
   DD    0000000Ah,00000017h  <- 2  
   DD    00000017h,00000023h  <- 3
   DD    00000023h,00000032h  <- 4
   DD    00000032h,0000003Dh  <- 5
   DD    0000003Dh,00000049h  <- 6
   DD    00000049h,00000052h  <- 7
   DD    00000052h,00000060h  <- 8
```

洗完后的结果:

```text
   DD    00000032h,0000003Dh  <- 5 
   DD    00000023h,00000032h  <- 4
   DD    0000000Ah,00000017h  <- 2  
   DD    00000000h,0000000Ah  <- 1  
   DD    00000017h,00000023h  <- 3  
   DD    00000052h,00000060h  <- 8  
   DD    0000003Dh,00000049h  <- 6  
   DD    00000049h,00000052h  <- 7  
```

按洗完的顺序物理摆放代码,每块代码在末尾用 `jmp` 跳到逻辑上的下一块,保持代码原来的执行顺序,由于代码物理位置的变化,有些指令需要修正(`call`,`jmp`,`jcc` 等)。

程序执行时,`eip` 会随着执行自然落到物理位置的第 1 块(但这并不是程序逻辑的第 1 块,具体说是第 5 块,第 1 块现在排在第 4),所以在混洗后的第 1 个物理块前,应该有 1 句:

`JMP EntryPoint` (处理前代码的首句,在第 1 块内)

变形代码可按预定义的模式生成。随机选择合适的模式对代码进行扩展,这是前面压缩过程的逆反。这个过程是递归的,需要定义停止条件(如递归深度)。如:

```text
push eax -> sub esp,4     -> push ebp          -> ...
         mov [esp],eax    mov  ebp,esp
                         add  ebp,4
                         sub  ebp,4
                         xchg ebp,[esp]
                         pop  esp
                         mov [esp],eax
```

代码的物理膨胀同样需要修正指令。显然,变形代码的生成过程必须有反汇编引擎的支持。

仅仅有这些是不够的。`cracker` 同样可以借助反汇编引擎重建代码,只要识别出匹配的代码序列即可对扩展后的代码进行压缩。然而,如果在有效的代码中夹杂入垃圾代码,重建难度就直线上升了。例如:

```assembly
sub esp,4
shl ebp,2 <- 垃圾指令
mov [esp],eax
```

中间的 1 句是垃圾指令,这用肉眼很容易识别,但用程序来做就没这么简单了。这意味着在检测代码序列,与预定义模式进行比对时,允许对不连续指令进行匹配,而这样做误判几乎是无法避免的。

或许更好的解决方式是使用 `emulator`,从代码的算法(而不是具体的指令)来分析,类似启发式反病毒。

注意,在上面的例子中,加入 `shl ebp,2`(或者别的什么指令,如 `add ebp,1234`),前提是当前 `ebp` 是空闲寄存器,可随意使用。简单的处理可以是对空闲寄存器进行约定,比如约定某段代码不使用 `ebp`,但这样做强度不够,更好的办法是能判断出当前位置可用的空闲寄存器。

`z0mbie` 的 `XDE` 反汇编引擎就带有这样的功能(实际上这大概是其最重要的设计目的之一)。`XDE` 中最重要的概念是对象集(`object set`),即指令读写的 `registers`,`memory` 等,表示为 1 个 `DWORD` 中的对应位。被反汇编的指令,结果数据结构中包含了该指令读取的源对象集(`src_set`)与写入的目的对象集(`obj_set`)。具体细节可参考 `XDE` 源码及 `z0mbie` 的`Permutation conditions`。下面的代码是 `XDE v1.02` 带的例子,演示了获取空闲对象集(英文注释是原代码带的)。

```c
int AnalyzeRegs(/* IN/OUT */ CMistfall* M)
{
  HOOY* h;   // current element of the file, instruction or anything else
  // step 1/3 -- mark all registers as "used"
  // 对 List 内的每条指令,初始化为所有对象都被占用
  ForEachInList(M->HooyList, HOOY, h)
  {
     h->regused = XSET_UNDEF;  // XSET_UNDEF=0xFFFFFFFF
  }
  // step 2/3 -- for each instruction, clear dst_set&~src_set
  ForEachInList(M->HooyList, HOOY, h)
  {
     if(h->flags & FL_OPCODE)
     if((h->flags & FL_ERROR) == 0)
     {
       xde_instr instr;
       xde_disasm(h->dataptr, &instr);
       // update h->regused
       // 对链表上的每个指令节点,计算出对应的空闲对象集。仔细看看这个位运算:
       //  
       // 1. ~instr.src_set: 
       //    不作为当前指令 src_set 使用的 obj_set
       //  
       // 2. instr.dst_set & (~instr.src_set): 
       //    取 1 的结果与当前指令目的对象集的重叠。1 取到的 onj_set 只是当前指令未用,但后续的
       //    指令可能会使用,并不能判断为空闲。与的结果是,未作为当前指令 src 使用,同时被用作
       //    当前指令的 dst,这个结果就是从当前指令可以判断出的空闲 obj_set
       //  
       //    这种策略很保守,取到的结果可靠,但能取到的 obj_set 很有限,大部分时候为空。
       //    如 add eax,ebx:
       //     
       //    ~instr.src_set = {ecx|edx|esp|ebp|esi|edi|memory} 
       //    dst_set = {eax|flags} 
       //    与的结果为 0,即仅凭这条指令,取不到任何空闲 obj_set 数据。
       //  
       //    那么什么指令能取到空闲 obj_set? 可以立刻想到 1 个;-)
       //    mov eax,12345678
       //    此时 eax 为空闲寄存器,即在这条指令之前,可以插入指令,随意使用 eax
       //    (add eax,ebx/shl eax,cl/...) 
       //  
       // 3.  ~(instr.dst_set & (~instr.src_set)) 
       //     取反的结果对应不能判断为空闲的 obj_set,这个结果可能比指令真正使用的 obj_set 大。
       h->regused &= ~(instr.dst_set & (~instr.src_set));
       //  
       // incorrect, need to be replaced with recursive subroutine analysis.
       // however, the following means: 
       //   "CALL'ed subroutines doesnt use FLAGS as a source_object"
       //  
       if (XDE_CMD(instr.flag) == C_CMD_CALL)
          h->regused &= ~XSET_FL;
      }
  }
  // step 3/3 -- propagate zero bits within freeset, until possible;
  // works similar to "wave" algo
  // 从 2 得到的空闲 obj_set 是很少的,这里使用"波浪"算法传递 free set
```



```
for(;;)  
{  
    int modified = 0;  
    ForEachInList(M->HooyList, HOOY, h)  
    {  
        if (h->flags & FL_OPCODE)  // 当前节点为代码
        if ((h->flags & FL_ERROR) == 0)  
        if (h->next)      // 存在后续节点,也为代码
        if (h->next->flags & FL_OPCODE)  
        if ((h->next->flags & (FL_LABEL|FL_ERROR)) == 0)  
        {  
            xde_instr instr;  
            xde_disasm(h->dataptr, &instr);  
            if (((instr.src_set|instr.dst_set) & (~h->next->regused)) == 0)  
            {  
                // 1. instr.src_set|instr.dst_set 为当前指令读写的 obj_set
                //
                // 2. ~h->next->regused 为下一句指令的空闲 obj_set(来自第 2 步)
                //
                // 这个条件为 true 的含义是: 当前指令使用的 obj_set(包括 src 和 dst)
                // 与下一句指令的空闲 obj_set 不存在交叉,即下一条指令的空闲 obj_set
                // 当前指令没有使用
                //   
                // 条件满足,会执行 h->regused &= h->next->regused
                // 即 h->next->regused 内对应为 0 的位(空闲 obj_set)被传递到当前指令
                //
                // ForEachInList 循环遍历所有指令,每次只影响了相邻的 2 条指令
                //
                // 由 for(;;)和 ForEachInList 构成的 2 重循环的目的,在于尽可能地传递
                // 空闲 obj_set 信息,在每条指令位置提取出尽可能多的空闲 obj_set
                if ((h->regused & h->next->regused) != h->regused)  
                {  
                    // h->regused 的某位为 1,h->next->regused 的对应位为 0 时条件为
                    // true,即某个 obj,对下一条指令确定为空闲,对当前指令不能判断
                    // 为空闲。注意这个检查是在修改当前指令的 regused(即下面的
                    // h->regused &= h->next->regused)之前做的。
                    // 有了外面的 if 作保证,这个原来对当前指令"可能使用"的 obj 可明
                    // 确判断为空闲。
                    // 注意对 1 条指令,src_set,dst_set 是通过反汇编得到的确切数据,
                    // 即当前指令肯定要使用的 obj_set。这和 regused 是不同的概念。
                    // regused 中为 0 的位代表空闲 obj,为 1 的位代表不能确切判定为空
                    // 闲 obj(当前指令不一定使用)。regused 涵盖的范围>= src_set|dst_set。
                    // regused 预置为 0xFFFFFFFF(全部占用),通过第 2 步的分析及前后
                    // 指令空闲 obj_set 的传递来剔除空闲 obj_set。
                    // 当条件为 true,增加 modified,表示相邻指令间出现了空闲 obj_set
                    // 传递。结束 ForEachInList 后进入下一轮 for(;;)再次遍历指令链表
                    modified++;  
                }  
                h->regused &= h->next->regused;  
            }  
        }  
        if (!modified) break;  
    }  
    return 1;  
} // AnalyzeRegs    
```

用这段代码测试了几个 `kernel32` 中的 API，对空闲 `obj_set` 的识别效果似乎还不够好，不过这种做法是可行的。另外，这里的代码虽然很短，读起来并不容易，但愿我没有理解错;-)。

`MetaPHOR` 还包括寄存器变换（随机选择不同的寄存器），变量选择，用不同的 `opcode` 编码实现同样的功能等等.

下面是一个整理出来的 handler，变形代码没有清理干净。但 `Themida` 的 `VM` 基本就这模样（请别问我要程序，毛病很多，没有完成，我怀疑按代码模式匹配的办法根本得不到理想的结果）。

```
_14A0000:014A0000 l_014A0000:                        ; DATA XREF: _1480000:01480008o
_14A0000:014A0000                    mov     esi, [edi]      ; Moved from 014A1FB5
_14A0000:014A0002                    mov     al, [esi+1]
_14A0000:014A0008                    mov     dl, [esi]
_14A0000:014A000A                    and     dl, 80h
_14A0000:014A000D                    or      dl, dl
_14A0000:014A000F                    jz      loc_14A001F
_14A0000:014A0015                    mov     dword ptr [edi+20h], 0
_14A0000:014A001F
_14A0000:014A001F loc_14A001F:                        ; CODE XREF: _14A0000:014A000Fj
_14A0000:014A001F                    and     al, 3
_14A0000:014A0022                    mov     dl, [esi]
_14A0000:014A0024                    or      al, al
_14A0000:014A0026                    jnz     loc_14A0312
_14A0000:014A002C                    xor     byte ptr [edi+20h], 1Fh
_14A0000:014A0033
_14A0000:014A0033 loc_14A0033:                        ; CODE XREF: _14A0000:014A0322j
_14A0000:014A0033                                        ; _14A0000:014A066Ej
_14A0000:014A0033                                        ; _14A0000:014A07FEj
_14A0000:014A0033                    nop
_14A0000:014A0034                    mov     ecx, [esi]
_14A0000:014A0036                    and     ecx, 80h
_14A0000:014A003C                    or      ecx, ecx
_14A0000:014A003E                    jz      loc_14A004E
_14A0000:014A0044                    mov     dword ptr [edi+8], 0
_14A0000:014A004E
_14A0000:014A004E loc_14A004E:                        ; CODE XREF: _14A0000:014A003Ej
_14A0000:014A004E                    nop
_14A0000:014A004F                    mov     ebx, [esi+7]
_14A0000:014A0055                    and     ebx, 0FFh
_14A0000:014A005B                    mov     ecx, ebx
_14A0000:014A005D                    xor     bl, [edi+8]
_14A0000:014A0063                    xor     bl, 20h
_14A0000:014A0066                    sub     bl, 10h
_14A0000:014A0069                    xor     [edi+8], ecx
_14A0000:014A006F                    test    ebx, 80h
_14A0000:014A0075                    jz      loc_14A0327
_14A0000:014A007B                    and     ebx, 0FFFFFF7Fh
_14A0000:014A0081                    imul    ebx, 0Eh
_14A0000:014A0084                    sub     [edi], ebx
_14A0000:014A0086
_14A0000:014A0086 loc_14A0086:                        ; CODE XREF: _14A0000:014A032Cj
_14A0000:014A0086                    pushf
_14A0000:014A0087                    add     ebx, 0FFFFFFFFh
_14A0000:014A008D                    popf
_14A0000:014A008E                    mov     ebp, [esi+0Ch]
_14A0000:014A0094                    and     ebp, 0FFh
_14A0000:014A009A                    mov     ebx, [esi]
_14A0000:014A009C                    and     ebx, 80h
_14A0000:014A00A2                    or      ebx, ebx
_14A0000:014A00A4                    jz      loc_14A00B4
_14A0000:014A00AA                    mov     dword ptr [edi+10h], 0
_14A0000:014A00B4
_14A0000:014A00B4 loc_14A00B4:                        ; CODE XREF: _14A0000:014A00A4j
_14A0000:014A00B4                    mov     ebx, ebp
_14A0000:014A00B6                    xor     ebp, [edi+10h]
_14A0000:014A00BC                    xor     ebp, 92h
_14A0000:014A00C2                    sub     ebp, 5Dh
_14A0000:014A00C8                    and     ebx, 0FFh
_14A0000:014A00CE                    xor     [edi+10h], ebx
_14A0000:014A00D4                    and     ebp, 0FFh
_14A0000:014A00DA                    mov     [edi+0Ch], ebp
_14A0000:014A00E0                    mov     eax, [esi+1]
_14A0000:014A00E6                    and     eax, 1Ch
_14A0000:014A00EC                    shr     eax, 2
_14A0000:014A00EF                    shl     eax, 3
_14A0000:014A00F2                    or      eax, eax
_14A0000:014A00F4                    jz      loc_14A018D
_14A0000:014A00FA                    mov     ebx, [edi+38h]
_14A0000:014A0100                    add     [edi+38h], eax
_14A0000:014A0106                    cmp     dword ptr [edi+38h], 0F0000070h
_14A0000:014A0110                    jb      loc_14A013F
_14A0000:014A0116                    mov     eax, [edi+38h]
_14A0000:014A011C                    sub     eax, 0F0000070h
_14A0000:014A0122                    add     eax, 0F0000000h
_14A0000:014A0128                    mov     [edi+38h], eax
_14A0000:014A012E                    sub     ebx, 0F0000070h
_14A0000:014A0134                    neg     ebx
_14A0000:014A0136                    mov     eax, ebx
_14A0000:014A0138                    mov     ebx, 0F0000000h
_14A0000:014A013D                    sub     ebx, eax
_14A0000:014A013F
_14A0000:014A013F loc_14A013F:                        ; CODE XREF: _14A0000:014A0110j
_14A0000:014A013F                    mov     dx, si
_14A0000:014A0142                    push    esi
_14A0000:014A0143                    mov     esi, [edi+3Ch]
_14A0000:014A0149                    mov     eax, edi
_14A0000:014A014B                    mov     edx, [edi+38h]
_14A0000:014A0151                    sub     edx, ebx
_14A0000:014A0153                    lea     edi, [edi+144h]
_14A0000:014A0159                    add     edi, edx
_14A0000:014A015B                    lea     edx, [eax+144h]
_14A0000:014A0161                    add     edx, 70h
_14A0000:014A0167                    mov     ecx, 0Eh
_14A0000:014A016C
```



```asm
_14A0000:014A016C loc_14A016C:                          ; CODE XREF: _14A0000:014A0347j
_14A0000:014A016C                or      ecx, ecx
_14A0000:014A016E                jnz     loc_14A0331
_14A0000:014A0174                mov     edi, [eax+3Ch]
_14A0000:014A017A                lea     esi, [eax+144h]
_14A0000:014A0180                mov     ecx, 0Eh
_14A0000:014A0185                imul    ecx, 8
_14A0000:014A0188                rep movsb
_14A0000:014A018A                pop     esi
_14A0000:014A018B                mov     edi, eax
_14A0000:014A018D
_14A0000:014A018D loc_14A018D:                          ; CODE XREF: _14A0000:014A00F4j
_14A0000:014A018D                mov     eax, esi
_14A0000:014A018F                mov     eax, [esi+0Ah]
_14A0000:014A0195                and     eax, 0FFh
_14A0000:014A019B                mov     ecx, [esi]
_14A0000:014A019D                and     ecx, 80h
_14A0000:014A01A3                or      ecx, ecx
_14A0000:014A01A5                jz      loc_14A01B5
_14A0000:014A01AB                mov     dword ptr [edi+18h], 0
_14A0000:014A01B5
_14A0000:014A01B5 loc_14A01B5:                          ; CODE XREF: _14A0000:014A01A5j
_14A0000:014A01B5                mov     ecx, eax
_14A0000:014A01B7                xor     eax, [edi+18h]
_14A0000:014A01BD                xor     eax, 63h
_14A0000:014A01C3                sub     eax, 0F2h
_14A0000:014A01C9                and     ecx, 0FFh
_14A0000:014A01CF                xor     [edi+18h], ecx
_14A0000:014A01D5                and     eax, 0FFh
_14A0000:014A01DB                mov     [edi+14h], eax
_14A0000:014A01E1                mov     ebx, [esi+2]
_14A0000:014A01E7                mov     dl, [edi+20h]
_14A0000:014A01ED                mov     eax, 4
_14A0000:014A01F2                add     ecx, ebp
_14A0000:014A01F4                xor     ebp, ebp
_14A0000:014A01F6                sub     ebp, 0FFFFFFE3h
_14A0000:014A01FC                or      ch, 63h
_14A0000:014A01FF
_14A0000:014A01FF loc_14A01FF:                          ; CODE XREF: _14A0000:014A0360j
_14A0000:014A01FF                or      eax, eax
_14A0000:014A0201                jnz     loc_14A034C
_14A0000:014A0207                push    ebx
_14A0000:014A0208                pushf
_14A0000:014A0209                add     ebx, eax
_14A0000:014A020B                popf
_14A0000:014A020C                mov     dl, [esi+0Bh]
_14A0000:014A0212                xor     dl, [edi+20h]
_14A0000:014A0218                add     dl, 5Bh
_14A0000:014A021B                add     dl, [edi+20h]
_14A0000:014A0221                test    dl, 80h
_14A0000:014A0224                jz      loc_14A0365
_14A0000:014A022A                or      dword ptr [edi+24h], 10h
_14A0000:014A0234
_14A0000:014A0234 loc_14A0234:                          ; CODE XREF: _14A0000:014A036Fj
_14A0000:014A0234                shl     edx, 8
_14A0000:014A0237                mov     dl, [esi+8]
_14A0000:014A023D                xor     dl, [edi+20h]
_14A0000:014A0243                add     dl, 0F0h
_14A0000:014A0246                add     dl, [edi+20h]
_14A0000:014A024C                shl     edx, 8
_14A0000:014A024F                mov     dl, [esi+6]
_14A0000:014A0255                xor     dl, [edi+20h]
_14A0000:014A025B                add     dl, 50h
_14A0000:014A025E                add     dl, [edi+20h]
_14A0000:014A0264                shl     edx, 8
_14A0000:014A0267                mov     dl, [esi+9]
_14A0000:014A026D                xor     dl, [edi+20h]
_14A0000:014A0273                add     dl, 66h
_14A0000:014A0276                add     dl, [edi+20h]
_14A0000:014A027C                and     dl, 7Fh
_14A0000:014A027F                mov     ebx, edx
_14A0000:014A0281                and     edx, 40h
_14A0000:014A0287                or      edx, edx
_14A0000:014A0289                jz      loc_14A0374
_14A0000:014A028F                or      dword ptr [edi+24h], 20h
_14A0000:014A0299
_14A0000:014A0299 loc_14A0299:                          ; CODE XREF: _14A0000:014A037Ej
_14A0000:014A0299                mov     edx, ebx
_14A0000:014A029B                and     edx, 7
_14A0000:014A02A1                or      dl, dl
_14A0000:014A02A3                jnz     loc_14A0383
_14A0000:014A02A9                mov     edx, [esp+8]
_14A0000:014A02B0                push    edx
_14A0000:014A02B1
_14A0000:014A02B1 loc_14A02B1:                          ; CODE XREF: _14A0000:014A0390j
_14A0000:014A02B1                                            ; _14A0000:014A0696j
_14A0000:014A02B1                                            ; _14A0000:014A0806j
_14A0000:014A02B1                                            ; _14A0000:014A0813j
_14A0000:014A02B1                                            ; _14A0000:014A081Aj
_14A0000:014A02B1                push    esi
_14A0000:014A02B2                mov     dword ptr [edi+48h], 1
_14A0000:014A02BC                mov     edx, ebx
_14A0000:014A02BE                and     edx, 0FFFFFF00h
_14A0000:014A02C4                or      edx, edx
_14A0000:014A02C6                jnz     loc_14A0395
_14A0000:014A02CC
_14A0000:014A02CC loc_14A02CC:                          ; CODE XREF: _14A0000:014A05F2j
_14A0000:014A02CC                                            ; _14A0000:014A06A0j
_14A0000:014A02CC                                            ; _14A0000:014A06B0j
_14A0000:014A02CC                pop     esi
_14A0000:014A02CD                cmp     dword ptr [edi+48h], 0
_14A0000:014A02D7                jz      loc_14A05F7
_14A0000:014A02DD                mov     eax, [esp]
_14A0000:014A02E0                cmp     eax, 80000000h
_14A0000:014A02E6                jb      loc_14A0639
_14A0000:014A02EC                cmp     eax, 800445C0h
_14A0000:014A02F2                jnb     loc_14A0639
_14A0000:014A02F8                sub     eax, 80000000h
_14A0000:014A02FE                add     eax, [edi+4Ch]
_14A0000:014A0304                mov     [edi], eax
_14A0000:014A0306                add     esp, 8
_14A0000:014A030C                jmp     dword ptr [edi+58h]
_14A0000:014A0312 ; ---------------------------------------------------------------------------
_14A0000:014A0312
_14A0000:014A0312 loc_14A0312:                          ; CODE XREF: _14A0000:014A0026j
_14A0000:014A0312                cmp     al, 1
_14A0000:014A0315                jnz     loc_14A065F
_14A0000:014A031B                add     byte ptr [edi+20h], 76h
_14A0000:014A0322                jmp     loc_14A0033
_14A0000:014A0327 ; ---------------------------------------------------------------------------
_14A0000:014A0327
_14A0000:014A0327 loc_14A0327:                          ; CODE XREF: _14A0000:014A0075j
_14A0000:014A0327                imul    ebx, 0Eh
_14A0000:014A032A                add     [edi], ebx
_14A0000:014A032C                jmp     loc_14A0086
_14A0000:014A0331 ; ---------------------------------------------------------------------------
_14A0000:014A0331
_14A0000:014A0331 loc_14A0331:                          ; CODE XREF: _14A0000:014A016Ej
_14A0000:014A0331                cmp     edi, edx
_14A0000:014A0333                jb      loc_14A033F
_14A0000:014A0339                lea     edi, [eax+144h]
_14A0000:014A033F
```



```
_14A0000:014A033F loc_14A033F:                     ; CODE XREF: _14A0000:014A0333j  
_14A0000:014A033F                     movsd  
_14A0000:014A0340                     movsd  
_14A0000:014A0341                     sub     ecx, 1  
_14A0000:014A0347                     jmp     loc_14A016C  
_14A0000:014A034C ; -------------------------------------------------------------  
_14A0000:014A034C  
_14A0000:014A034C loc_14A034C:                     ; CODE XREF: _14A0000:014A0201j  
_14A0000:014A034C                     ror     ebx, 4  
_14A0000:014A034F                     cmp     dl, 80h  
_14A0000:014A0352                     jbe     loc_14A0673  
_14A0000:014A0358                     xor     bl, dl  
_14A0000:014A035A  
_14A0000:014A035A loc_14A035A:                     ; CODE XREF: _14A0000:014A0675j  
_14A0000:014A035A                     sub     eax, 1  
_14A0000:014A0360                     jmp     loc_14A01FF  
_14A0000:014A0365 ; -------------------------------------------------------------  
_14A0000:014A0365  
_14A0000:014A0365 loc_14A0365:                     ; CODE XREF: _14A0000:014A0224j  
_14A0000:014A0365                     and     dword ptr [edi+24h], 0FFFFFFEFh  
_14A0000:014A036F                     jmp     loc_14A0234  
_14A0000:014A0374 ; -------------------------------------------------------------  
_14A0000:014A0374  
_14A0000:014A0374 loc_14A0374:                     ; CODE XREF: _14A0000:014A0289j  
_14A0000:014A0374                     and     dword ptr [edi+24h], 0FFFFFFDFh  
_14A0000:014A037E                     jmp     loc_14A0299  
_14A0000:014A0383 ; -------------------------------------------------------------  
_14A0000:014A0383  
_14A0000:014A0383 loc_14A0383:                     ; CODE XREF: _14A0000:014A02A3j  
_14A0000:014A0383                     cmp     dl, 1  
_14A0000:014A0386                     jnz     loc_14A067A  
_14A0000:014A038C                     mov     edx, [esp]  
_14A0000:014A038F                     push    edx  
_14A0000:014A0390                     jmp     loc_14A02B1  
_14A0000:014A0395 ; -------------------------------------------------------------  
_14A0000:014A0395  
_14A0000:014A0395 loc_14A0395:                     ; CODE XREF: _14A0000:014A02C6j  
_14A0000:014A0395                     mov     eax, 0  
_14A0000:014A039A                     mov     esi, 0  
_14A0000:014A039F                     mov     edx, ebx  
_14A0000:014A03A1                     and     edx, 200h  
_14A0000:014A03A7                     mov     ecx, [edi+70h]  
_14A0000:014A03AD                     and     ecx, 1  
_14A0000:014A03B3                     shr     ecx, 0  
_14A0000:014A03B6                     or      edx, edx  
_14A0000:014A03B8                     jz      loc_14A03DE  
_14A0000:014A03BE                     mov     edx, ebx  
_14A0000:014A03C0                     and     edx, 100h  
_14A0000:014A03C6                     shr     edx, 8  
_14A0000:014A03C9                     xor     edx, ecx  
_14A0000:014A03CB                     not     edx  
_14A0000:014A03CD                     and     edx, 1  
_14A0000:014A03D3                     or      eax, edx  
_14A0000:014A03D5                     shl     eax, 1  
_14A0000:014A03D8                     add     esi, 1  
_14A0000:014A03DE  
_14A0000:014A03DE loc_14A03DE:                     ; CODE XREF: _14A0000:014A03B8j  
_14A0000:014A03DE                     mov     edx, ebx  
_14A0000:014A03E0                     and     edx, 800h  
_14A0000:014A03E6                     or      edx, edx  
_14A0000:014A03E8                     jz      loc_14A041D  
_14A0000:014A03EE                     mov     ecx, [edi+70h]  
_14A0000:014A03F4                     and     ecx, 40h  
_14A0000:014A03FA                     shr     ecx, 6  
_14A0000:014A03FD                     mov     edx, ebx  
_14A0000:014A03FF                     and     edx, 400h  
_14A0000:014A0405                     shr     edx, 0Ah  
_14A0000:014A0408                     xor     edx, ecx  
_14A0000:014A040A                     not     edx  
_14A0000:014A040C                     and     edx, 1  
_14A0000:014A0412                     or      eax, edx  
_14A0000:014A0414                     shl     eax, 1  
_14A0000:014A0417                     add     esi, 1  
_14A0000:014A041D  
_14A0000:014A041D loc_14A041D:                     ; CODE XREF: _14A0000:014A03E8j  
_14A0000:014A041D                     mov     edx, ebx  
_14A0000:014A041F                     and     edx, 2000000h  
_14A0000:014A0425                     or      edx, edx  
_14A0000:014A0427                     jz      loc_14A045C  
_14A0000:014A042D                     mov     ecx, [edi+70h]  
_14A0000:014A0433                     and     ecx, 4  
_14A0000:014A0439                     shr     ecx, 0Ah  
_14A0000:014A043C                     mov     edx, ebx  
_14A0000:014A043E                     and     edx, 1000000h  
_14A0000:014A0444                     shr     edx, 18h  
_14A0000:014A0447                     xor     edx, ecx  
_14A0000:014A0449                     not     edx  
_14A0000:014A044B                     and     edx, 1  
_14A0000:014A0451                     or      eax, edx  
_14A0000:014A0453                     shl     eax, 1  
_14A0000:014A0456                     add     esi, 1  
_14A0000:014A045C  
_14A0000:014A045C loc_14A045C:                     ; CODE XREF: _14A0000:014A0427j  
_14A0000:014A045C                     mov     edx, ebx  
_14A0000:014A045E                     and     edx, 2000h  
_14A0000:014A0464                     or      edx, edx  
_14A0000:014A0466                     jz      loc_14A049B  
_14A0000:014A046C                     mov     ecx, [edi+70h]  
_14A0000:014A0472                     and     ecx, 80h  
_14A0000:014A0478                     shr     ecx, 7  
_14A0000:014A047B                     mov     edx, ebx  
_14A0000:014A047D                     and     edx, 1000h  
_14A0000:014A0483                     shr     edx, 0Ch  
_14A0000:014A0486                     xor     edx, ecx  
_14A0000:014A0488                     not     edx  
_14A0000:014A048A                     and     edx, 1  
_14A0000:014A0490                     or      eax, edx  
_14A0000:014A0492                     shl     eax, 1  
_14A0000:014A0495                     add     esi, 1  
_14A0000:014A049B  
_14A0000:014A049B loc_14A049B:                     ; CODE XREF: _14A0000:014A0466j  
_14A0000:014A049B                     mov     edx, ebx  
_14A0000:014A049D                     and     edx, 8000h  
_14A0000:014A04A3                     or      edx, edx  
_14A0000:014A04A5                     jz      loc_14A04DA  
_14A0000:014A04AB                     mov     ecx, [edi+70h]  
_14A0000:014A04B1                     and     ecx, 800h  
_14A0000:014A04B7                     shr     ecx, 0Bh  
_14A0000:014A04BA                     mov     edx, ebx  
_14A0000:014A04BC                     and     edx, 4000h  
_14A0000:014A04C2                     shr     edx, 0Eh  
_14A0000:014A04C5                     xor     edx, ecx  
_14A0000:014A04C7                     not     edx  
_14A0000:014A04C9                     and     edx, 1  
_14A0000:014A04CF                     or      eax, edx  
_14A0000:014A04D1                     shl     eax, 1  
_14A0000:014A04D4                     add     esi, 1  
_14A0000:014A04DA  
_14A0000:014A04DA loc_14A04DA:                     ; CODE XREF: _14A0000:014A04A5j  
_14A0000:014A04DA                     mov     edx, ebx  
_14A0000:014A04DC                     and     edx, 20000h  
_14A0000:014A04E2                     or      edx, edx  
_14A0000:014A04E4                     jz      loc_14A0519  
_14A0000:014A04EA                     mov     ecx, [edi+70h]  
_14A0000:014A04F0                     and     ecx, 4  
_14A0000:014A04F6                     shr     ecx, 2  
_14A0000:014A04F9                     mov     edx, ebx  
```



```
\_14A0000:014A04FB                 and     edx, 10000h  

\_14A0000:014A0501                 shr     edx, 10h  

\_14A0000:014A0504                 xor     edx, ecx  

\_14A0000:014A0506                 not     edx  

\_14A0000:014A0508                 and     edx, 1  

\_14A0000:014A050E                 or      eax, edx  

\_14A0000:014A0510                 shl     eax, 1  

\_14A0000:014A0513                 add     esi, 1  

\_14A0000:014A0519  

\_14A0000:014A0519 loc\_14A0519:                              ; CODE XREF: \_14A0000:014A04E4j  

\_14A0000:014A0519                 mov     edx, ebx  

\_14A0000:014A051B                 and     edx, 80000h  

\_14A0000:014A0521                 or      edx, edx  

\_14A0000:014A0523                 jz      loc\_14A0569  

\_14A0000:014A0529                 mov     ecx, [edi+70h]  

\_14A0000:014A052F                 and     ecx, 80h  

\_14A0000:014A0535                 shr     ecx, 7  

\_14A0000:014A0538                 mov     edx, [edi+70h]  

\_14A0000:014A053E                 and     edx, 800h  

\_14A0000:014A0544                 shr     edx, 0Bh  

\_14A0000:014A0547                 xor     ecx, edx  

\_14A0000:014A0549                 mov     edx, ebx  

\_14A0000:014A054B                 and     edx, 40000h  

\_14A0000:014A0551                 shr     edx, 12h  

\_14A0000:014A0554                 xor     edx, ecx  

\_14A0000:014A0556                 not     edx  

\_14A0000:014A0558                 and     edx, 1  

\_14A0000:014A055E                 or      eax, edx  

\_14A0000:014A0560                 shl     eax, 1  

\_14A0000:014A0563                 add     esi, 1  

\_14A0000:014A0569  

\_14A0000:014A0569 loc\_14A0569:                              ; CODE XREF: \_14A0000:014A0523j  

\_14A0000:014A0569                 mov     edx, ebx  

\_14A0000:014A056B                 and     edx, 200000h  

\_14A0000:014A0571                 or      edx, edx  

\_14A0000:014A0573                 jz      loc\_14A059F  

\_14A0000:014A0579                 push    84h  

\_14A0000:014A057E                 call    dword ptr [edi+5Ch]  

\_14A0000:014A0584                 mov     edx, [edi+30h]  

\_14A0000:014A058A                 mov     edx, [edx]  

\_14A0000:014A058C                 and     edx, 0FFFFh  

\_14A0000:014A0592                 or      edx, edx  

\_14A0000:014A0594                 jnz     loc\_14A059F  

\_14A0000:014A059A                 mov     eax, 1  

\_14A0000:014A059F  

\_14A0000:014A059F loc\_14A059F:                              ; CODE XREF: \_14A0000:014A0573j  

\_14A0000:014A059F                                     ; \_14A0000:014A0594j  

\_14A0000:014A059F                 mov     edx, ebx  

\_14A0000:014A05A1                 and     edx, 800000h  

\_14A0000:014A05A7                 or      edx, edx  

\_14A0000:014A05A9                 jz      loc\_14A05CF  

\_14A0000:014A05AF                 push    84h  

\_14A0000:014A05B4                 call    dword ptr [edi+5Ch]  

\_14A0000:014A05BA                 mov     edx, [edi+30h]  

\_14A0000:014A05C0                 mov     edx, [edx]  

\_14A0000:014A05C2                 or      edx, edx  

\_14A0000:014A05C4                 jnz     loc\_14A05CF  

\_14A0000:014A05CA                 mov     eax, 1  

\_14A0000:014A05CF  

\_14A0000:014A05CF loc\_14A05CF:                              ; CODE XREF: \_14A0000:014A05A9j  

\_14A0000:014A05CF                                     ; \_14A0000:014A05C4j  

\_14A0000:014A05CF                 mov     ecx, esi  

\_14A0000:014A05D1                 mov     esi, 1  

\_14A0000:014A05D6                 shl     esi, cl  

\_14A0000:014A05D8                 sub     esi, 1  

\_14A0000:014A05DE                 and     ebx, 10h  

\_14A0000:014A05E4                 or      ebx, ebx  

\_14A0000:014A05E6                 jnz     loc\_14A069B  

\_14A0000:014A05EC                 mov     [edi+48h], eax  

\_14A0000:014A05F2                 jmp     loc\_14A02CC  

\_14A0000:014A05F7 ; ---------------------------------------------------------------------------  

\_14A0000:014A05F7  

\_14A0000:014A05F7 loc\_14A05F7:                              ; CODE XREF: \_14A0000:014A02D7j  

\_14A0000:014A05F7                 nop  

\_14A0000:014A05F8                 add     esp, 8  

\_14A0000:014A05FE                 mov     ebx, [edi+0Ch]  

\_14A0000:014A0604                 shl     ebx, 2  

\_14A0000:014A0607                 mov     eax, [edi+14h]  

\_14A0000:014A060D                 shl     eax, 2  

\_14A0000:014A0610                 mov     esi, [edi+40h]  

\_14A0000:014A0616                 add     esi, ebx  

\_14A0000:014A0618                 mov     esi, [esi]  

\_14A0000:014A061A                 add     esi, eax  

\_14A0000:014A061C                 push    ecx  

\_14A0000:014A061D                 push    esi  

\_14A0000:014A061E                 push    ebx  

\_14A0000:014A061F                 mov     ebx, 0  

\_14A0000:014A0624                 add     ebp, ebx  

\_14A0000:014A0626                 mov     esi, ebx  

\_14A0000:014A0628                 pop     ebx  

\_14A0000:014A0629                 mov     ecx, esi  

\_14A0000:014A062B                 add     ax, dx  

\_14A0000:014A062E                 pop     esi  

\_14A0000:014A062F                 add     ecx, esi  

\_14A0000:014A0631                 mov     esi, [ecx]  

\_14A0000:014A0633                 shr     dh, 1  

\_14A0000:014A0636                 pop     ecx  

\_14A0000:014A0637                 jmp     esi  

\_14A0000:014A0639 ; ---------------------------------------------------------------------------  

\_14A0000:014A0639  

\_14A0000:014A0639 loc\_14A0639:                              ; CODE XREF: \_14A0000:014A02E6j  

\_14A0000:014A0639                                     ; \_14A0000:014A02F2j  

\_14A0000:014A0639                 cmp     eax, [edi+4Ch]  

\_14A0000:014A063F                 jb      loc\_14A06B5  

\_14A0000:014A0645                 cmp     eax, [edi+50h]  

\_14A0000:014A064B                 ja      loc\_14A06B5  

\_14A0000:014A0651                 mov     [edi], eax  

\_14A0000:014A0653                 add     esp, 8  

\_14A0000:014A0659                 jmp     dword ptr [edi+58h]  

\_14A0000:014A065F ; ---------------------------------------------------------------------------  

\_14A0000:014A065F  

\_14A0000:014A065F loc\_14A065F:                              ; CODE XREF: \_14A0000:014A0315j  

\_14A0000:014A065F                 cmp     al, 2  

\_14A0000:014A0662                 jnz     loc\_14A07F7  

\_14A0000:014A0668                 xor     [edi+20h], dl  

\_14A0000:014A066E                 jmp     loc\_14A0033  

\_14A0000:014A0673 ; ---------------------------------------------------------------------------  

\_14A0000:014A0673  

\_14A0000:014A0673 loc\_14A0673:                              ; CODE XREF: \_14A0000:014A0352j  

\_14A0000:014A0673                 add     bl, dl  

\_14A0000:014A0675                 jmp     loc\_14A035A  

\_14A0000:014A067A ; ---------------------------------------------------------------------------  

\_14A0000:014A067A  

\_14A0000:014A067A loc\_14A067A:                              ; CODE XREF: \_14A0000:014A0386j  

\_14A0000:014A067A                 cmp     dl, 2  

\_14A0000:014A067D                 jnz     loc\_14A0803  

\_14A0000:014A0683                 mov     edx, [edi+34h]  

\_14A0000:014A0689                 cmp     edx, 80000000h  

\_14A0000:014A068F                 jbe     loc\_14A0818  

\_14A0000:014A0695                 push    edx  

\_14A0000:014A0696                 jmp     loc\_14A02B1  

\_14A0000:014A069B ; ---------------------------------------------------------------------------  

\_14A0000:014A069B  

\_14A0000:014A069B loc\_14A069B:                              ; CODE XREF: \_14A0000:014A05E6j  

\_14A0000:014A069B                 shr     eax, 1  
```



```
\_14A0000:014A069E                 cmp     esi, eax  
\_14A0000:014A06A0                 jz      loc\_14A02CC  
\_14A0000:014A06A6                 mov     dword ptr \[edi+48h\], 0  
\_14A0000:014A06B0                 jmp     loc\_14A02CC  
\_14A0000:014A06B5 ; -------------------------------------------------------------------------  
\_14A0000:014A06B5  
\_14A0000:014A06B5 loc\_14A06B5:                                  ; CODE XREF: \_14A0000:014A063Fj  
\_14A0000:014A06B5                                   ; \_14A0000:014A064Bj  
\_14A0000:014A06B5                 cmp     dword ptr \[edi+60h\], 0  
\_14A0000:014A06BF                 jz      loc\_14A06E1  
\_14A0000:014A06C5                 test    dword ptr \[edi+24h\], 10h  
\_14A0000:014A06CF                 jz      loc\_14A06E1  
\_14A0000:014A06D5                 sub     eax, \[edi+60h\]  
\_14A0000:014A06DB                 add     eax, \[edi+64h\]  
\_14A0000:014A06E1  
\_14A0000:014A06E1 loc\_14A06E1:                                  ; CODE XREF: \_14A0000:014A06BFj  
\_14A0000:014A06E1                                   ; \_14A0000:014A06CFj  
\_14A0000:014A06E1                 push    eax  
\_14A0000:014A06E2                 mov     ebx, \[edi+24h\]  
\_14A0000:014A06E8                 and     ebx, 20h  
\_14A0000:014A06EE                 or      ebx, ebx  
\_14A0000:014A06F0                 jz      loc\_14A0739  
\_14A0000:014A06F6                 mov     esi, \[edi+68h\]  
\_14A0000:014A06FC                 add     esi, 8  
\_14A0000:014A0702                 mov     eax, \[edi\]  
\_14A0000:014A0704                 mov     ebx, \[esi+10h\]  
\_14A0000:014A070A  
\_14A0000:014A070A loc\_14A070A:                                  ; CODE XREF: \_14A0000:014A0869j  
\_14A0000:014A070A                 cmp     dword ptr \[esi\], 0FFFFFFFFh  
\_14A0000:014A0710                 jnz     loc\_14A081F  
\_14A0000:014A0716  
\_14A0000:014A0716 loc\_14A0716:                                  ; CODE XREF: \_14A0000:014A0841j  
\_14A0000:014A0716                 mov     ebx, \[esi+14h\]  
\_14A0000:014A071C                 add     ebx, \[edi+54h\]  
\_14A0000:014A0722                 push    0ACh  
\_14A0000:014A0727                 call    dword ptr \[edi+5Ch\]  
\_14A0000:014A072D                 mov     eax, \[edi+30h\]  
\_14A0000:014A0733                 add     dword ptr \[eax\], 4  
\_14A0000:014A0739  
\_14A0000:014A0739 loc\_14A0739:                                  ; CODE XREF: \_14A0000:014A06F0j  
\_14A0000:014A0739                 pop     eax  
\_14A0000:014A073A                 mov     esi, eax  
\_14A0000:014A073C                 push    0ACh  
\_14A0000:014A0741                 call    dword ptr \[edi+5Ch\]  
\_14A0000:014A0747                 mov     eax, \[edi+30h\]  
\_14A0000:014A074D                 mov     esp, \[eax\]  
\_14A0000:014A074F                 or      ebx, ebx  
\_14A0000:014A0751                 jz      loc\_14A0758  
\_14A0000:014A0757                 push    ebx  
\_14A0000:014A0758  
\_14A0000:014A0758 loc\_14A0758:                                  ; CODE XREF: \_14A0000:014A0751j  
\_14A0000:014A0758                 push    esi  
\_14A0000:014A0759                 mov     esi, \[edi+38h\]  
\_14A0000:014A075F                 pushf  
\_14A0000:014A0760                 sub     esi, 0F0000000h  
\_14A0000:014A0766                 popf  
\_14A0000:014A0767                 add     esi, \[edi+3Ch\]  
\_14A0000:014A076D                 mov     eax, \[edi+3Ch\]  
\_14A0000:014A0773                 add     eax, 70h  
\_14A0000:014A0779                 mov     ecx, 0Eh  
\_14A0000:014A077E                 push    edi  
\_14A0000:014A077F                 mov     ebp, \[edi+3Ch\]  
\_14A0000:014A0785                 lea     edi, \[edi+144h\]  
\_14A0000:014A078B  
\_14A0000:014A078B loc\_14A078B:                                  ; CODE XREF: \_14A0000:014A0858j  
\_14A0000:014A078B                 or      ecx, ecx  
\_14A0000:014A078D                 jnz     loc\_14A0846  
\_14A0000:014A0793                 mov     eax, \[esp\]  
\_14A0000:014A0796                 lea     esi, \[eax+144h\]  
\_14A0000:014A079C                 mov     ecx, 1Ch  
\_14A0000:014A07A1                 mov     edi, \[eax+3Ch\]  
\_14A0000:014A07A7                 rep movsd  
\_14A0000:014A07A9                 pop     edi  
\_14A0000:014A07AA                 mov     dword ptr \[edi+38h\], 0F0000000h  
\_14A0000:014A07B4                 push    dword ptr \[edi+70h\]  
\_14A0000:014A07BA                 push    dword ptr \[edi+74h\]  
\_14A0000:014A07C0                 push    dword ptr \[edi+84h\]  
\_14A0000:014A07C6                 push    dword ptr \[edi+8Ch\]  
\_14A0000:014A07CC                 push    dword ptr \[edi+7Ch\]  
\_14A0000:014A07D2                 push    dword ptr \[edi+0ACh\]  
\_14A0000:014A07D8                 push    dword ptr \[edi+0A4h\]  
\_14A0000:014A07DE                 push    dword ptr \[edi+94h\]  
\_14A0000:014A07E4                 push    dword ptr \[edi+9Ch\]  
\_14A0000:014A07EA                 mov     dword ptr \[edi+28h\], 0  
\_14A0000:014A07F4                 popa  
\_14A0000:014A07F5                 popf  
\_14A0000:014A07F6                 retn  
\_14A0000:014A07F7 ; -------------------------------------------------------------------------  
\_14A0000:014A07F7  
\_14A0000:014A07F7 loc\_14A07F7:                                  ; CODE XREF: \_14A0000:014A0662j  
\_14A0000:014A07F7                 nop  
\_14A0000:014A07F8                 add     \[edi+20h\], dl  
\_14A0000:014A07FE                 jmp     loc\_14A0033  
\_14A0000:014A0803 ; -------------------------------------------------------------------------  
\_14A0000:014A0803  
\_14A0000:014A0803 loc\_14A0803:                                  ; CODE XREF: \_14A0000:014A067Dj  
\_14A0000:014A0803                 cmp     dl, 3  
\_14A0000:014A0806                 jnz     loc\_14A02B1  
\_14A0000:014A080C                 mov     edx, \[edi+34h\]  
\_14A0000:014A0812                 push    edx  
\_14A0000:014A0813                 jmp     loc\_14A02B1  
\_14A0000:014A0818 ; -------------------------------------------------------------------------  
\_14A0000:014A0818  
\_14A0000:014A0818 loc\_14A0818:                                  ; CODE XREF: \_14A0000:014A068Fj  
\_14A0000:014A0818                 push    dword ptr \[edx\]  
\_14A0000:014A081A                 jmp     loc\_14A02B1  
\_14A0000:014A081F ; -------------------------------------------------------------------------  
\_14A0000:014A081F  
\_14A0000:014A081F loc\_14A081F:                                  ; CODE XREF: \_14A0000:014A0710j  
\_14A0000:014A081F                 add     ebx, \[edi+54h\]  
\_14A0000:014A0825                 mov     ecx, \[esi+4\]  
\_14A0000:014A082B                 add     ecx, \[edi+54h\]  
\_14A0000:014A0831                 cmp     ecx, eax  
\_14A0000:014A0833                 jnb     loc\_14A085D  
\_14A0000:014A0839                 cmp     eax, ebx  
\_14A0000:014A083B                 ja      loc\_14A085D  
\_14A0000:014A0841                 jmp     loc\_14A0716  
\_14A0000:014A0846 ; -------------------------------------------------------------------------  
\_14A0000:014A0846  
\_14A0000:014A0846 loc\_14A0846:                                  ; CODE XREF: \_14A0000:014A078Dj  
\_14A0000:014A0846                 cmp     esi, eax  
\_14A0000:014A0848                 jb      loc\_14A0850  
\_14A0000:014A084E                 mov     esi, ebp  
\_14A0000:014A0850  
\_14A0000:014A0850 loc\_14A0850:                                  ; CODE XREF: \_14A0000:014A0848j  
\_14A0000:014A0850                 movsd  
\_14A0000:014A0851                 movsd  
\_14A0000:014A0852                 sub     ecx, 1  
\_14A0000:014A0858                 jmp     loc\_14A078B  
```



```
_14A0000:014A085D ; ---------------------------------------------------------------------------
_14A0000:014A085D
_14A0000:014A085D loc_14A085D:                            ; CODE XREF: _14A0000:014A0833j
_14A0000:014A085D                                         ; _14A0000:014A083Bj
_14A0000:014A085D            add     esi, 0Ch
_14A0000:014A0863            mov     ebx, [esi+10h]
_14A0000:014A0869            jmp     loc_14A070A
```
