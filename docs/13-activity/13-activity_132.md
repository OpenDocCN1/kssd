

# 第二阶段第一题简单分析,加入如何得到 51 字节文件的方法,计算出的，不是穷举

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-257.htm`](https://www.pediy.com/kssd/pediy09/pediy09-257.htm)

> 关键是前面的计算出 0x0D 的部分  
> 这部分其实就是  
> ((a\*0x78CC02A869948F1B mod 2⁶⁴) mod 0x5BE6FF82A5164785) mod 2³² =0x0D  
> 求解的算法 zmworm 以前有过一篇帖子介绍的很详细  
> 我再贴一下详细的求解过程吧  
> 为了简化求解过程,我把最后的 mod 2³² 去掉，原始方程变为  
> (a\*0x78CC02A869948F1B mod 2⁶⁴) mod 0x5BE6FF82A5164785=0x0D  
> 由于前面的部分肯定小于 2⁶⁴,0x5BE6FF82A5164785\*3 则大于 2⁶⁴  
> 所以前面的部分(a\*0x78CC02A869948F1B mod 2⁶⁴)只能有 3 种情况,0x0D,0x5BE6FF82A5164785+0x0D,0x5BE6FF82A5164785\*2+0x0D  
> 题目化简为  
> a\*0x78CC02A869948F1B mod 2⁶⁴=d  
> a                      b                          c      d  
> 根据不同的 d 会有 3 个方程  
> 求解方法为  
> b=0x78CC02A869948F1B  
> 由于 gcd(b,c)=1 所以这个方程比较好解  
> 求乘法逆元  
> 求解 b\*b' mod 2⁶⁴=1  
> 使用扩展欧几里德算法可以求出  
> b'=90B22DD80D53313  
> a=d\*b' mod 2⁶⁴  
> 针对不同的 d 求解 a  
> (1)  
> a=0x7590C53F8AD397F7  
> xor 0x1C 后的指令  
> 00B40000   /EB 7C           jmp     short 00B4007E  
> 00B40002   |CF              iretd  
> 00B40003   |45              inc     ebp  
> 00B40004   |23E6            and     esp, esi  
> 00B40006   |8CF9            mov     cx, seg?                         ; 未定义的段寄存器  
> 由于要跳到 7E 处执行，马上跳回来也要 128 字节(0x80)  
> (2)  
> a=0x704DA9F23D6365D6  
> 00B40000    CA AF7F         retf    7FAF  
> 00B40003    42              inc     edx  
> 00B40004    EE              out     dx, al  
> 00B40005    47              inc     edi  
> 00B40006    51              push    ecx  
> xor 0x1C 后的指令  
> 这个肯定没戏了  
> (3)  
> a=0x6B0A8EA4EFF333B5  
> 00B40000    A9 9AEF00B8     test    eax, B800EF9A  
> 00B40005    36:16           push    ss  
> 00B40007    7D 6B           jge     short 00B40074  
> 这个其实还有希望，之前没发现  
> 选择(1)的计算结果的好处是后面 xor 0x1C 后,前两个字节会变成 EB 7C  
> 作题的时候被算法搞晕了，没仔细研究(3)的代码，确实是应该选(3)  
> 当时看了(1)的跳转和(2)肯定不行，就没有多想(3)，结果最小代码长度竟然就出在(3)上  
> 加入当时忽略的 MOD 2³² 条件  
> 由于此条件并不影响计算结果的低 32 位，所以对(1)和(2)并没有什么影响  
> 下面针对不同的高 32 位计算新的 k  
> d=0x0000000100000000+0x5BE6FF82A5164785\*2+0x0D 时  
> k=0xEBDFC1B7 EFF333B5  
> xor 0x1C 后代码为  
> 00B40000    A9 9AEF00AB     test    eax, AB00EF9A  
> 00B40005    6A C3           push    -3D  
> 00B40007    28                ??  
> d=0x0000000200000000+0x5BE6FF82A5164785\*2+0x0D 时  
> k=0x6CB4F4CA EFF333B5  
> xor 0x1C 后代码为  
> 00B40000    A9 9AEF00D6     test    eax, D600EF9A  
> 00B40005    22A8 C4XXXXXX   and     ch, byte ptr \[eax+XXXXXXC4\]  
> d=0x0000000300000000+0x5BE6FF82A5164785\*2+0x0D 时  
> k=0xED8A27DD EFF333B5  
> xor 0x1C 后代码为  
> 00B40000    A9 9AEF00C1     test    eax, C100EF9A  
> 00B40005    E6 96           out     96, al  
> 00B40007    7B                jpo  
> d=0x0000000400000000+0x5BE6FF82A5164785\*2+0x0D 时  
> k=0x6E5F5AF0 EFF333B5  
> xor 0x1C 后代码为  
> 00B40000    A9 9AEF00EC     test    eax, EC00EF9A  
> 00B40005    B6 43           mov     dh, 43  
> 00B40007    2D 00000000     sub     eax, 0  
> d=0x0000000500000000+0x5BE6FF82A5164785\*2+0x0D 时  
> k=0xEF348E03 EFF333B5  
> xor 0x1C 后代码为  
> 00B40000    A9 9AEF001F     test    eax, 1F00EF9A  
> 00B40005    91              xchg    eax, ecx  
> 00B40006    28C7            sub     bh, al  
> ...........  
> 在里面选一个不改变原来程序流程(不会跳走，也不会异常)的，然后根据前面 8 个字节的代码补上其余的修改字符的指令，平衡堆栈，就可以达到 51 个字节的极限值 

> 90B22DD80D53313\*(5be6ff82a5164785\*2+xxxxxxxxxxxx000d)  
> 保证 5be6ff82a5164785\*2+xxxxxxxxxxxx000d<10000000000000000  
> xxxxxxxxxxxx 可以任意取值(当然也可能取错，不过大多数都好使) 

> 算法的最后是一个一元二次方程  
> y = -x²+26x+169  
> 可以知道在 Y 得到最大值的时候,x 为 13 也就是 0xD,这个数值正好将 ret 的地址覆盖,而多一分不让,少一点也不行!  
>   
> 然后穷举就 ok 了.... 

 4dd466d0e1e0cc123afbe504da2462ca