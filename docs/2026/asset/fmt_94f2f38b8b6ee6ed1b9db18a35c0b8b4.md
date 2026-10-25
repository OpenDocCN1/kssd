
### X64

> 参数1、参数2、参数3分别保存在`RDX`、`R8D`、`R9D`中，`this`指针存放`RCX`中，剩下的参数从右往左依次入栈，被调用者实现栈平衡，返回值存放在 `RAX` 中。

```
# 该代码是 msvc 2017 x64 生成的汇编代码

    int sum = calc.thiscall_add(1, 2, 3, 4, 5, 6, 7);

00007FF602E6190F  mov         dword ptr [rsp+38h],7  
00007FF602E61917  mov         dword ptr [rsp+30h],6  
00007FF602E6191F  mov         dword ptr [rsp+28h],5  
00007FF602E61927  mov         dword ptr [rsp+20h],4  
00007FF602E6192F  mov         r9d,3  
00007FF602E61935  mov         r8d,2  
00007FF602E6193B  mov         edx,1  
00007FF602E61940  lea         rcx,[calc]              # this指针
00007FF602E61944  call        Calc::thiscall_add (07FF602E610A0h)  
00007FF602E61949  mov         dword ptr [sum],eax    # 返回值

int Calc::thiscall_add(int a, int b, int c, int d, int e, int f, int g)

{

00007FF602E61770  mov         dword ptr [rsp+20h],r9d  
00007FF602E61775  mov         dword ptr [rsp+18h],r8d  
00007FF602E6177A  mov         dword ptr [rsp+10h],edx  
00007FF602E6177E  mov         qword ptr [rsp+8],rcx  
00007FF602E61783  push        rbp  
00007FF602E61784  push        rdi  
00007FF602E61785  sub         rsp,0E8h  
00007FF602E6178C  mov         rbp,rsp  
00007FF602E6178F  mov         rdi,rsp  
00007FF602E61792  mov         ecx,3Ah  
00007FF602E61797  mov         eax,0CCCCCCCCh  
00007FF602E6179C  rep stos    dword ptr [rdi]  
00007FF602E6179E  mov         rcx,qword ptr [rsp+108h]  
    int sum = a + b + c + d + e + f + g;
00007FF602E617A6  mov         eax,dword ptr [b]  
00007FF602E617AC  mov         ecx,dword ptr [a]  
00007FF602E617B2  add         ecx,eax  
00007FF602E617B4  mov         eax,ecx  
00007FF602E617B6  add         eax,dword ptr [c]  
00007FF602E617BC  add         eax,dword ptr [d]  
00007FF602E617C2  add         eax,dword ptr [e]  
00007FF602E617C8  add         eax,dword ptr [f]  
00007FF602E617CE  add         eax,dword ptr [g]  
00007FF602E617D4  mov         dword ptr [sum],eax  
    return sum;
00007FF602E617D7  mov         eax,dword ptr [sum]  # 存放返回值
}
00007FF602E617DA  lea         rsp,[rbp+0E8h]  
00007FF602E617E1  pop         rdi  
00007FF602E617E2  pop         rbp  
00007FF602E617E3  ret                                 # 没做栈平衡
```

### ARM

> 参数1、参数2、参数3分别保存在`R1`、`R2`、`R3`中，`this`指针存放`R0`中，剩下的参数从右往左依次入栈，被调用者实现栈平衡，返回值存放在 `R0` 中。

```
; 该代码是 arm-linux-androideabi-gcc + IDA PRO 生成的反汇编代码
.text:000085BC                 MOV             R3, #4
.text:000085C0                 STR             R3, [SP] ; int
.text:000085C4                 MOV             R3, #5
.text:000085C8                 STR             R3, [SP,#4] ; int
.text:000085CC                 MOV             R3, #6
.text:000085D0                 STR             R3, [SP,#8] ; int
.text:000085D4                 MOV             R3, #7
.text:000085D8                 STR             R3, [SP,#12] ; int
.text:000085DC                 MOV             R3, #8
.text:000085E0                 STR             R3, [SP,#16] ; int
.text:000085E4                 MOV             R3, #9
.text:000085E8                 STR             R3, [SP,#20] ; int
.text:000085EC                 MOV             R3, #10
.text:000085F0                 STR             R3, [SP,#24] ; int
.text:000085F4                 MOV             R0, R2  ; this
.text:000085F8                 MOV             R1, #1  ; int
.text:000085FC                 MOV             R2, #2  ; int
.text:00008600                 MOV             R3, #3  ; int
.text:00008604                 BL              _ZN4Calc12thiscall_addEiiiiiiiiii ; Calc::thiscall_add(int,int,int,int,int,int,int,int,int,int)
.text:00008608                 MOV             R3, R0

.text:00008544                 EXPORT _ZN4Calc12thiscall_addEiiiiiiiiii
.text:00008544
.text:00008544                 STR             R11, [SP,#-4]!
.text:00008548                 ADD             R11, SP, #0
.text:0000854C                 SUB             SP, SP, #0x1C
.text:00008550                 STR             R0, [R11,#-16]
.text:00008554                 STR             R1, [R11,#-20]
.text:00008558                 STR             R2, [R11,#-24]
.text:0000855C                 STR             R3, [R11,#-28]
.text:00008560                 LDR             R2, [R11,#-20]
.text:00008564                 LDR             R3, [R11,#-24]
.text:00008568                 ADD             R2, R2, R3
.text:0000856C                 LDR             R3, [R11,#-28]
.text:00008570                 ADD             R2, R2, R3
.text:00008574                 LDR             R3, [R11,#4]
.text:00008578                 ADD             R2, R2, R3
.text:0000857C                 LDR             R3, [R11,#8]
.text:00008580                 ADD             R2, R2, R3
.text:00008584                 LDR             R3, [R11,#12]
.text:00008588                 ADD             R2, R2, R3
.text:0000858C                 LDR             R3, [R11,#16]
.text:00008590                 ADD             R3, R2, R3
.text:00008594                 STR             R3, [R11,#-8]
.text:00008598                 LDR             R3, [R11,#-8]
.text:0000859C                 MOV             R0, R3            # 返回值
.text:000085A0                 SUB             SP, R11, #0
.text:000085A4                 LDR             R11, [SP],#4
.text:000085A8                 BX              LR
```

### ARM64

> 参数1~参数7 分别保存到 `X1`~`X7` 寄存器中，`this`指针存放`X0`中，剩下的参数从右往左依次入栈，被调用者实现栈平衡，返回值存放在 `X0` 中。
