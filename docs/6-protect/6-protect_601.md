
# Vmprotect1.2x 诡计收集

> **原文**：[`www.pediy.com/kssd/pediy09/pediy09-569.htm`](https://www.pediy.com/kssd/pediy09/pediy09-569.htm)

> 昨天跟了一个伪代码块，发现几个有趣的伪代码组合：

> ### 1. 组合示例 1
>
> 第 000030 条:`0x92 01       |0044|VM_PUSH_IMMB   ;push 0x01`
> 第 000031 条:`0x0a            |0048|VM_ADD      ;`
> 第 000032 条:`0x92 ff          |0044|VM_PUSH_IMMB   ;PUSH -0x01`
> 第 000033 条:`0x0a            |0048|VM_ADD      ;  `
>
> 这个组合相当于 NOP 吧。对 `[esp]` 里的 DWORD 数，先加个 1，再减个 1。

> ### 2. 组合示例 2
>
> 第 000047 条:`0xc3           |0040|VM_PUSH_ESP     `
> 第 000048 条:`0x14           |0040|VM_MOV_SSA_TO_B`
>
> 这个组合相当于是把 `push [esp]` 吧，把 `[esp]` 再 push 一次。

> ### 3. 组合示例 3
>
> 第 000047 条:`0xc3           |0040|VM_PUSH_ESP     `
> 第 000048 条:`0x14           |0040|VM_MOV_SSA_TO_B `
> 第 000049 条:`0x57           |0044|VM_NOR      `
>
> 这个组合是把第 2 个组合之后加了个 `VM_NOR`。相当于是 `VM_NOT`。

> ---
> 大家碰到什么有趣的组合没?希望拿来分享。

> **第 1,3 个组合**：
>
> *Mutation. The executed file is modified on the level of processor commands (existing commands are modified, all kinds of garbage commands are added, etc.).*
>
> *This compilation type poorly protects the code it processes against hacking and analyzing and mainly prevents functions being processed from determining with signature analyzers (PEiD+KANAL, IDA+FLIRT, etc.). As a rule, there is no need to protect library functions against hacking and analyzing and it will be enough just to change their signatures for the hacker to be*

> **第 2 个组合**
>
> *The bug causing an error during the mutation of the PUSH ESP has been fixed*

> 大家说的都高深莫测，比汇编还难看懂。似乎每句都有虚拟机、变形、花指令、扰码、偷代码。。处处玄机。
> 听不大懂，只好自己自言自语了。将上面几组定义为变形组合，命名为 `VMX_???`

> **代码**:

***

> `1、VMX_INC   0xe6 01     |0040|VM_PUSH_IMMB   01h           0xf4           |0044|VM_ADD     FFB9CF5Bh  `

***

> **代码**:

***

> `2、VMX_DEC   0x92 ff          |0044|VM_PUSH_IMMB   ;PUSH -0x01   0x0a            |0048|VM_ADD      ; `

***

> **代码**:

***

> `3、VMX_NOP=VMX_INC,VMX_DEC   0x92 01        |0044|VM_PUSH_IMMB   ;push 0x01   0x0a            |0048|VM_ADD      ;   0x92 ff          |0044|VM_PUSH_IMMB   ;PUSH -0x01   0x0a            |0048|VM_ADD      ;  `

***

> **代码**:

***

> `4、VMX_PUSH_[ESP]暂定   0xc3          |0040|VM_PUSH_ESP      0x14          |0040|VM_MOV_SSA_TO_B`

***

> **代码**:

***

> `5、VMX_NOT=VMX_PUSH_[ESP]，VM_NOR   0xc3          |0040|VM_PUSH_ESP      0x14          |0040|VM_MOV_SSA_TO_B  0x57          |0044|VM_NOR      `

***

> **代码**:

***

> `6、VMX_NOP=VMX_NOT,VMX_NOT   0xa1          |0040|VM_PUSH_ESP     12FE00h        0xa9          |0040|VM_MOV_SSA_TO_B     FFF9CF5Bh  0xc1          |0044|VM_NOR                     4630A5h   0xa1          |0040|VM_PUSH_ESP     12FE00h        0xa9          |0040|VM_MOV_SSA_TO_B      4630A5h   0xc1          |0044|VM_NOR       FFF9CF5Bh`

***

> **代码**:

***

> `7、VMX_NOP=VMX_NOT,VMX_INC,VM_NOT,VM_INC   0xa1          |0040|VM_PUSH_ESP     12FE00h          0xa9          |0040|VM_MOV_SSA_TO_B      4630A5h     0xc1           |0044|VM_NOR       FFF9CF5Bh    0xe6 01     |0040|VM_PUSH_IMMB      01h          0xf4           |0044|VM_ADD        FFB9CF5Bh    0xa1          |0040|VM_PUSH_ESP                      12FE00h          0xa9          |0040|VM_MOV_SSA_TO_B     FFB9CF5Bh    0x5e          |0044|VM_NOR       4630A4h      0xe6 01     |0040|VM_PUSH_IMMB      01h          0xf5           |0044|VM_ADD       4630A5h`

***

> 贴一些常用的变形代码（摘自月中人翻译的《变形实践》，`vmprotect` 里用到不少），给愿意一起研究的看一下
> BTW:是我的马甲

> ### 1. 常用变形代码 1
>
> **代码**:

***

> `XOR Reg,-1 --> NOT Reg   XOR Mem,-1 --> NOT Mem   MOV Reg,Reg --> NOP   SUB Reg,Imm --> ADD Reg,-Imm   SUB Mem,Imm --> ADD Mem,-Imm   XOR Reg,0 --> MOV Reg,0   XOR Mem,0 --> MOV Mem,0   ADD Reg,0 --> NOP   ADD Mem,0 --> NOP   OR Reg,0 --> NOP   OR Mem,0 --> NOP   AND Reg,-1 --> NOP   AND Mem,-1 --> NOP   AND Reg,0 --> MOV Reg,0   AND Mem,0 --> MOV Mem,0   XOR Reg,Reg --> MOV Reg,0   SUB Reg,Reg --> MOV Reg,0   OR Reg,Reg --> CMP Reg,0   AND Reg,Reg --> CMP Reg,0   TEST Reg,Reg --> CMP Reg,0   LEA Reg,[Imm] --> MOV Reg,Imm   LEA Reg,[Reg+Imm] --> ADD Reg,Imm   LEA Reg,[Reg2] --> MOV Reg,Reg2   LEA Reg,[Reg+Reg2] --> ADD Reg,Reg2   LEA Reg,[Reg2+Reg2+xxx]--> LEA Reg,[2*Reg2+xxx]   MOV Reg,Reg --> NOP   MOV Mem,Mem--> NOP `

***

> ### 2. 常用变形代码 2
>
> **代码**:

***

> `PUSH Imm / POP Reg --> MOV Reg,Imm   PUSH Imm / POP Mem --> MOV Mem,Imm   PUSH Reg / POP Reg2 --> MOV Reg2,Reg   PUSH Reg / POP Mem --> MOV Mem,Reg   PUSH Mem / POP Reg --> MOV Reg,Mem   PUSH Mem / POP Mem2--> MOV Mem2,MemMOV Mem,Reg/PUSH Mem --> PUSH Reg   POP Mem / MOV Reg,Mem --> POP Reg   POP Mem2 / MOV Mem,Mem2 --> POP Mem   MOV Mem,Reg / MOV Reg2,Mem --> MOV Reg2,Reg   MOV Mem,Imm / PUSH Mem --> PUSH Imm   MOV Mem,Imm / OP Reg,Mem --> OP Reg,Imm   MOV Reg,Imm / ADD Reg,Reg2 --> LEA Reg,[Reg2+Imm]   MOV Reg,Reg2 / ADD Reg,Imm --> LEA Reg,[Reg2+Imm]   MOV Reg,Reg2 / ADD Reg,Reg3 --> LEA Reg,[Reg2+Reg3]   ADD Reg,Imm / ADD Reg,Reg2 --> LEA Reg,[Reg+Reg2+Imm]   ADD Reg,Reg2 / ADD Reg,Imm --> LEA Reg,[Reg+Reg2+Imm]   OP Reg,Imm / OP Reg,Imm2 --> OP Reg,(Imm OP Imm2)   OP Mem,Imm / OP Mem,Imm2 --> OP Mem,(Imm OP Imm2)   LEA Reg,[Reg2+Imm] / ADD Reg,Reg3 --> LEA Reg,[Reg2+Reg3+Imm]   LEA Reg,[(RegX+)Reg2+Imm] / ADD Reg,Reg2--> LEA Reg,[(RegX+)2*Reg2+Imm]   POP Mem / PUSH Mem --> NOP   MOV Mem2,Mem / MOV Mem3,Mem2 --> MOV Mem3,Mem   MOV Mem2,Mem / OP Reg,Mem2 --> OP Reg,Mem   MOV Mem2,Mem / MOV Mem2,xxx --> MOV Mem2,xxx   MOV Mem,Reg / CALL Mem --> CALL Reg   MOV Mem,Reg / JMP Mem --> JMP Reg   MOV Mem2,Mem / CALL Mem2 --> CALL Mem   MOV Mem2,Mem / JMP Mem2 --> JMP Mem   MOV Mem,Reg / MOV Mem2,Mem --> MOV Mem2,Reg   OP Reg,xxx / MOV Reg,yyy --> MOV Reg,yyy   Jcc @xxx / !Jcc @xxx--> JMP @xxx (this applies to (Jcc & 0FEh) with(Jcc | 1)   NOT Reg / NEG Reg --> ADD Reg,1   NOT Reg / ADD Reg,1 --> NEG Reg   NOT Mem / NEG Mem --> ADD Mem,1   NOT Mem / ADD Mem,1 --> NEG Mem   NEG Reg / NOT Reg --> ADD Reg,-1   NEG Reg / ADD Reg,-1 --> NOT Reg   NEG Mem / NOT Mem --> ADD Mem,-1   NEG Mem / ADD Mem,-1 --> NOT Mem   CMP X,Y / != Jcc (CMP without Jcc) --> NOP   TEST X,Y / != Jcc --> NOP   POP Mem / JMP Mem --> RET   PUSH Reg / RET --> JMP Reg   CALL Mem / MOV Mem2,EAX --> CALL Mem / APICALL_STORE Mem2   MOV Reg,Mem / CALL Reg --> CALL Mem   XOR Reg,Reg / MOV Reg8,[Mem] --> MOVZX Reg,byte ptr [Mem]   MOV Reg,[Mem] / AND Reg,0FFh --> MOVZX Reg,byte ptr [Mem]`

***

> ### 3. 常用变形代码 3
>
> **代码**:

***

> `MOV Mem,Reg   OP Mem,Reg2   MOV Reg,Mem   --> OP Reg,Reg2      MOV Mem,Reg   OP Mem,Imm   MOV Reg,Mem   --> OP Reg,Imm      MOV Mem,Imm   OP Mem,Reg   MOV Reg,Mem   --> OP Reg,Imm (it can't be SUB)      MOV Mem2,Mem   OP Mem2,Reg   MOV Mem,Mem2   --> OP Mem,Reg      MOV Mem2,Mem   OP Mem2,Imm   MOV Mem,Mem2   --> OP Mem,Imm      CMP Reg,Reg   JO/JB/JNZ/JA/JS/JNP/JL/JG @xxx   != Jcc   --> NOP      CMP Reg,Reg   JNO/JAE/JZ/JBE/JNS/JP/JGE/JLE @xxx   != Jcc   --> JMP @xxx      MOV Mem,Imm   CMP/TEST Reg,Mem   Jcc @xxx   --> CMP/TEST Reg,Imm   Jcc @xxx      MOV Mem,Reg   SUB/CMP Mem,Reg2   Jcc @xxx   --> CMP Reg,Reg2   Jcc @xxx      MOV Mem,Reg   AND/TEST Mem,Reg2   Jcc @xxx   --> TEST Reg,Reg2   Jcc @xxx      MOV Mem,Reg   SUB/CMP Mem,Imm   Jcc @xxx   --> CMP Reg,Imm   Jcc @xxx      MOV Mem,Reg   AND/TEST Mem,Imm   Jcc @xxx   --> TEST Reg,Imm   Jcc @xxx      MOV Mem2,Mem   CMP/TEST Reg,Mem2   Jcc @xxx   --> CMP/TEST Reg,Mem   Jcc @xxx      MOV Mem2,Mem   AND/TEST Mem2,Reg   Jcc @xxx   --> TEST Mem,Reg   Jcc @xxx      MOV Mem2,Mem   SUB/CMP Mem2,Reg   Jcc @xxx   --> CMP Mem,Reg   Jcc @xxx      MOV Mem2,Mem   AND/TEST Mem2,Imm   Jcc @xxx   --> TEST Mem,Imm   Jcc @xxx      MOV Mem2,Mem   SUB/CMP Mem2,Imm   Jcc @xxx   --> CMP Mem,Imm   Jcc @xxx      PUSH EAX   PUSH ECX   PUSH EDX   --> APICALL_BEGIN      POP EDX   POP ECX   POP EAX   --> APICALL_END`

***



# 转一段刘涛涛写的扭曲变换

有点`vmprotect`的感觉。不知是否用得到。贴过来做个备忘。

## 采用的代码扭曲方法：

* 用`JMP`把代码打乱。这已经不是什么新鲜的招数了，但它依然有效。
* 用`JMP`把多个函数缠绕在一起。这样可以让分析者找不到函数从什么地方开始，到什么地方结束。
* 把`call`改掉。破解者对`call`是极敏感的，此举可以让他找不到一个`call`。比如，我可以把：
    ```
    call sub1
    ```
  改为：
    ```
    mov eax, offset sub1 + 3
    push offset @1
    sub eax, 3
    jmp eax

    @1:
    ```
* 把`ret`改掉。破解者对`ret`是极敏感的，此举可以让他找不到一个`ret`。比如，我可以把`ret`写作：
    ```
    push ecx
    mov ecx, [esp+4]
    add esp, 8
    jmp ecx
    ```
* 改条件跳。条件跳也是极敏感的指令，比如我们可以把：
    ```
    cmp reg1, reg2
    jge L_DST

    L_NEXT:
    ```
  写作：
    ```
    push eax
    mov eax, reg1
    sub eax, reg2
    shr eax, 1fh
    neg eax
    and eax, L2 - L1
    add eax, L1
    jmp eax

    L1:
        pop eax
        jmp L_DST

    L2:
        pop eax

    L_NEXT:
    ```
  再看这个，你能看懂是什么意思吗：
    ```
    push offset @@L - offset L_3 + 23h
    jmp L_1

    L_2:
        jz L_3
        ret 4

    L_3:
        add dword ptr [esp+4], offset L_3 - 23h
        add esp, 4
        ret

    L_1:
    call L_2
        ...
    ```
  这里出现了`call`和`ret`，但又不是一般所期望的那种。这里的`call`不代表你发现了一个函数调用。这里的`ret`不代表是一个函数的结束。
* 使用堆栈代替寄存器。比如：
    ```
    MOV     EAX, DWORD PTR [ECX+0AD8h]
    PUSH    EAX
    MOV     ECX, DWORD PTR [EAX]
    ```
  可以写作：
    ```
    PUSH    EAX
    PUSH    ECX
    MOV     EAX, DWORD PTR [ESP]
    ADD     EAX, 0AD8h
    MOV     EAX, DWORD PTR [EAX]
    MOV     DWORD PTR [ESP+04h], EAX
    PUSH    DWORD PTR [ESP+04h]
    MOV     EAX, DWORD PTR [ESP]
    MOV     DWORD PTR [ESP+08h], EAX
    MOV     EAX, DWORD PTR [ESP]
    MOV     EAX, DWORD PTR [EAX]
    MOV     DWORD PTR [ESP+04h], EAX
    MOV     EAX, DWORD PTR [ESP]
    MOV     ECX, DWORD PTR [ESP+04h]
    ADD     ESP, 08h
    ```
  你能看懂吗？很明显，这个变换是不可逆变换。因为它本来使用了哪个寄存器，已经没有办法知道了。
* ……还可以想出很多扭曲变换的方法。化繁为简只有一种方法，化简为繁可以有无穷多种方法。

4dd466d0e1e0cc123afbe504da2462ca
