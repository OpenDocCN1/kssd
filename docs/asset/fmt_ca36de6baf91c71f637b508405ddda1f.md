
### 汇编代码段

```
0167:73475782  JNZ       73475790   
0167:73475784  FSTP      ST(0)   
0167:73475786  FLD       REAL8 PTR \[733C0718\]   
0167:7347578C  POP       EBP   
0167:7347578D  RET       0010   
0167:73475790  FLD       REAL8 PTR \[EBP+08\]   
0167:73475793  FCOMP     ST(1)   
0167:73475795  FSTSW   AX   
0167:73475797  SAHF   
0167:73475798  JNZ       734757A4   
0167:7347579A  FLD       REAL8 PTR \[EBP+10\]   
0167:7347579D  FCOMP     ST(1)   
0167:7347579F  FSTSW   AX   
0167:734757A1  SAHF   
0167:734757A2  JB        734757CE   
0167:734757A4  FLD       REAL8 PTR \[EBP+08\]   
0167:734757A7  FCOMP     ST(1)   
0167:734757A9  FSTSW   AX   
0167:734757AB  SAHF   
0167:734757AC  FSTP      ST(0)   
0167:734757AE  JAE       734757D2   
0167:734757B0  FLD       REAL8 PTR \[EBP+10\]   
0167:734757B3  PUSH      ECX   
0167:734757B4  PUSH      ECX   
0167:734757B5  FSTP      REAL8 PTR \[ESP\]   
0167:734757B8  CALL      73480307   
0167:734757BD  FCOMP     REAL8 PTR \[EBP+10\]   
0167:734757C0  POP       ECX   
0167:734757C1  POP       ECX   
0167:734757C2  FSTSW   AX   
0167:734757C4  SAHF   
0167:734757C5  JZ        734757D2   
0167:734757C7  PUSH      05   
0167:734757C9  CALL      733B5359   
0167:734757CE  FSTP      ST(0)   
0167:734757D0  JMP       734757C7   
0167:734757D2  CALL      733A68B8   
0167:734757D7  FLD       REAL8 PTR \[EBP+08\]   
0167:734757DA  FLD       REAL8 PTR \[EBP+10\]   
0167:734757DD  AND       DWORD PTR \[EAX\],00   
0167:734757E0  CALL      73480400   <--上面的一堆都没什么用,不过是把上面的 EDX 结果放入 ST0 中,但带过这个 CALL 后发现那个大数出现了,所以要进入这个 CALL   
0167:734757E5  FST       REAL8 PTR \[EBP+10\]   
0167:734757E8  PUSH      ECX   
0167:734757E9  PUSH      ECX   
```

### 进入 `734757E0` 处的 `CALL`

```
0167:73480400  SUB       ESP,10   
0167:73480403  FXCH      ST(1)   
0167:73480405  FSTP      REAL8 PTR \[ESP\]   
0167:73480408  FST       REAL8 PTR \[ESP+08\]   
0167:7348040C  MOV       EAX,\[ESP+0C\]   
0167:73480410  CALL      73480419  <--进入这个 CALL   
0167:73480415  ADD       ESP,10   
0167:73480418  RET   
```

### 进入 `73480419` 处的 `CALL`

```
0167:73480419  MOV       ECX,EAX   
0167:7348041B  PUSH      EAX   
0167:7348041C  WAIT   
0167:7348041D  FSTCW     WORD PTR \[ESP\]   
0167:73480420  CMP       WORD PTR \[ESP\],027F   
0167:73480426  JZ        7348042D   
0167:73480428  CALL      73483C17   
0167:7348042D  AND       ECX,7FF00000   
0167:73480433  LEA       EDX,\[ESP+08\]   
0167:73480437  CMP       ECX,7FF00000   
0167:7348043D  JZ        7348058D   
0167:73480443  CALL      73483C47   
0167:73480448  JZ        73480586   
0167:7348044E  TEST      EAX,7FF00000   
0167:73480453  JZ        734805AC   
0167:73480459  MOV       CL,\[ESP+0F\]   
0167:7348045D  AND       CL,80   
0167:73480460  JNZ       73480622   
0167:73480466  FYL2X <--前面 ST0=R(上面的结果),ST1\=3(好像是常数)   
                      <--该句是 ST0<-ST1\*(log2(ST0))   
0167:73480468  CALL      73483C02  <--进入这个 CALL   
0167:7348046D  CMP       CL,01   
0167:73480470  JNZ       73480474   
```

```
0167:73483C02  FLD       ST(0)   
0167:73483C04  FRNDINT           <--将 ST0 取整   
0167:73483C06  FSUBR     ST(1),ST  <--相减求出小数部分   
0167:73483C08  FXCH      ST(1)     <--ST0 与 ST1 交换   
0167:73483C0A  FCHS <--ST0 变为正数.此时 ST0=上面结果的小数部分,ST1\=整数部分   
0167:73483C0C  F2XM1               <-- ST0<-(2^ST0)+1   
0167:73483C0E  FLD1                <-- ST0<-ST0\-1   
0167:73483C10  FADDP     ST(1),ST  <-- ST0<-ST0+ST1   
0167:73483C12  FSCALE  <-- ST0<-ST0\*(2^ST1) 到这里终于计算完了   
0167:73483C14  FSTP      ST(1)   
0167:73483C16  RET   
```

本来想写个注册机，在汇编里挺清楚的语句，到了高级语言里面一堆数据类型全乱了。请各位高手写个注册机贴出来学习一下，我的编程实在太烂了...

### 后记

因为从来没有见到过浮点计算的指令，上面的许多运算都是在看雪老师的`《Crack Tutorial》`中浮点运算一节中查到的，但里面对 `FSCALE` 命令的解释有误，说是求 2 的 `ST(0)` 次方，即 `ST(0) <- 2^ST(0)`。而我却怎么也算不对，在网上查了一下，发现应该是 `ST(0) <- ST(0) * (2^ST(1))`。下面是我找到的资料，请各位看一下对不对。

> **引用:**
>
> `FSCALE`          Scale by a power of 2               Exceptions: I O U
>                                     C3 C2 C1 C0: ? ? * ?
>
> `FSCALE`
>
>       Logic   ST . ST \* 2\*\*ST(1)
>
>   `FSCALE` interprets the contents of `ST(1)` as an integer, and uses it
>
>   as an exponent of 2 with which to multiply the contents of `ST.` Thus,
>
>   `FSCALE` provides a quick way to multiply or divide by a power of 2.
>
>   The number in `ST(1)` must be -2\*\*15 <= `ST(1)` < 2\*\*15. It is truncated
>
>   before the operation is performed. If 0 < `ST(1)` < 1, the result is
>
>   undefined (and no exception is generated).
>
>   Opcode      Format
>
>   D9 FD       FSCALE
>
>   Timing
>
>   Operand       8087         287        387      486     Pentium
>
>   -             32-38        32-38      67-86    30-32    20-31   NP

`4dd466d0e1e0cc123afbe504da2462ca`
