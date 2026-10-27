
`1640` `(r11+0)` `rax`  `1648` `(r11+8)` `rbx`  `1650` `(r11+10)` `rcx`  `1658` `(r11+18)` `rdx`  `1660` `(r11+20)` `rsp`  `1668` `(r11+28)` `rbp`  `1670` `(r11+30)` `rsi`  `1678` `(r11+38)` `rdi`  `1680` `(r11+40)` `r8`  `1688` `(r11+48)` `r9`  `1690` `(r11+50)` `r10`  `1698` `(r11+58)` `r11`  `16a0` `(r11+60)` `r12`  `16a8` `(r11+68)` `r13`  `16b0` `(r11+70)` `r14`  `16b8` `(r11+78)` `r15`  `16c0` `(r11+80)` `rflag`

接下来保存 虚拟机指令起始点和虚拟机寄存器的起始指针

```
1400d30b1 : lea     r11, byte_14006024B+1             ,R11 <--   14006024c    //这个是V_RIP (即虚拟指令起始点)
1400d30b8 : lea     r10, [rbp+88h]                    ,R10 <--   1638        //这个是 V_REG_P
1400d30bf : lea     rsp, [rsp-8]                      ,RSP <--   15a8
1400d30c4 : mov     [rsp], r10
1400d30c8 : lea     rsp, [rsp-8]                      ,RSP <--   15a0
1400d30cd : mov     [rsp], r11
```

结构如下：

```
[15a0]    V_RIP                            虚拟指令起始点
[15a8]    V_REG_P 1638（虚拟机寄存器）
```

`14006024c`这是当前函数的虚拟指令起始点，可以先记住，后面就知道为什么我这么说了。

![](https://bbs.kanxue.com/upload/attach/202305/914360_KVBPGYJREF9G62H.png)

接下来进入

```
1400d30d1 : call    sub_1400D5B02                     ,RSP <--   1598
```

进call，return地址入栈：

```
[1598] = return add（call    sub_1400D5B02 下一行的地址 是int3）
```

这个函数，它也会保存一下寄存器状态，但是没啥用：

```
1400d5b02 : lea     rsp, [rsp-8]                      ,RSP <--   1590
1400d5b07 : mov     [rsp+8+var_8], rbx
1400d5b0b : lea     rsp, [rsp-8]                      ,RSP <--   1588
1400d5b10 : mov     [rsp+10h+var_10], rsi
1400d5b14 : lea     rsp, [rsp-8]                      ,RSP <--   1580
1400d5b19 : mov     [rsp+18h+var_18], rdi
1400d5b1d : lea     rsp, [rsp-8]                      ,RSP <--   1578
1400d5b22 : mov     [rsp+20h+var_20], rbp
1400d5b26 : lea     rsp, [rsp-8]                      ,RSP <--   1570
1400d5b2b : mov     [rsp+28h+var_28], r15
```

保存的状态如下：

```
[1570]  r15                                rbp = 15b0
[1578]  rbp
[1580]  rdi
[1588]  rsi
[1590]  rbx
```

然后会把V_RIP换个位置;

```
1400d5b2f : sub     rsp, 68h                          ,RSP <--   1508
,RF <--   12
1400d5b33 : mov     rbp, rsp                          ,RBP <--   1508
1400d5b38 : mov     r11, [rbp+98h]                    ,R11 <--   V_RIP
1400d5b42 : jmp     loc_1400D9146
1400d914f : mov     r10, [rbp+0A0h]                   ,R10 <--   V_REG_p
1400d9156 : jmp     loc_1400D7560
1400d756a : call    loc_1400DA21F                     ,RSP <--   1500
1400da21f : lea     rsp, [rsp+8]                      ,RSP <--   1508
//进call又rsp+8，假装是jmp
1400da224 : lea     r9, loc_1400E47B0                 ,R9 <--   1400e47b0
//这是int3 指令的地址
1400da22e : jmp     loc_1400D8689
1400d868e : mov     [rbp+0], r9
1400d8695 : jmp     loc_1400DB455
1400db45f : mov     [rbp+8], r11                      V_RIP 放到 [1510]
```

接下来就开始处理虚拟指令了。先把栈空间的格式整理一下：（这个地方非常重要）

```
虚拟机内RBP = RSP = 1508 ,R10 = V_REG_P
[1508] int3指令指针
[1510] V_RIP
//当前执行到的位置
.....
[1570]  r15                                rbp = 15b0
[1578]  rbp
[1580]  rdi
[1588]  rsi
[1590]  rbx
[1598] = return add（sub_1400D5B02 下一行的地址 是int3）   
[15a0]    V_RIP
//虚拟指令起始点,这是不会变的
[15a8]    V_REG_P
//1638（虚拟机寄存器）,这是不会变的
[15b0](rbp+0)    rflag
//这里以下是进入虚拟机前的寄存器状态
[15b8](rbp+8)    rax
[15c0](rbp+10)    rbp        (原来的栈底)
[15c8](rbp+18)    rbx
[15d0](rbp+20)    rcx
[15d8](rbp+28)    rdi
[15e0](rbp+30)    rdx
[15e8](rbp+38)    rsi
[15f0](rbp+40)    rsp     (原来的栈顶)
[15f8](rbp+48)    r8
[1600](rbp+50)    r9
[1608](rbp+58)    r10
[1610](rbp+60)    r11
[1618](rbp+68)    r12
[1620](rbp+70)    r13
[1628](rbp+78)    r14
[1630](rbp+80)    r15
[1638]    UnKnow                V_REG_P = r10 = 1638
//虚拟机的虚拟寄存器从 1638 开始往下都是
[1640](r11+0)    rax
//这里以下是进入虚拟机前的寄存器状态
[1648](r11+8)    rbx
[1650](r11+10)    rcx
[1658](r11+18)    rdx
[1660](r11+20)    rsp
[1668](r11+28)    rbp
[1670](r11+30)    rsi
[1678](r11+38)    rdi
[1680](r11+40)    r8
[1688](r11+48)    r9
[1690](r11+50)    r10
[1698](r11+58)    r11
[16a0](r11+60)    r12
[16a8](r11+68)    r13
[16b0](r11+70)    r14
[16b8](r11+78)    r15
[16c0](r11+80)    rflag
```

**其实只需要记住 R10放着V_REG_P 和 [rbp+8] 放着 V_RIP 。**

## handle分发

这是一段 tvmhandle的分发(tvmopcode的处理方式)

```
1400d7234 : mov     r9, [rbp+8]                       ,R9 <--   14006024c
1400d7240 : jmp     loc_1400D99D9
1400d99db : mov     r8b, [r9]                         ,R8 <--   e8
1400d99de : xor     r8b, 5Dh                          ,R8 <--   b5  ,RF <--   82
1400d99e2 : mov     rdx, 25E9ECA9BDE22AEAh            ,RDX <--   25e9eca9bde22aea
1400d99ec : not     rdx                               ,RDX <--   da161356421dd515
1400d99ef : lea     rdx, [r9+rdx]                     ,RDX <--   da1613578223d761
1400d99f3 : jmp     loc_1400D86A6
1400d86a8 : mov     r9, 0DA161356421DD513h            ,R9 <--   da161356421dd513
1400d86b2 : not     r9                                ,R9 <--   25e9eca9bde22aec
1400d86b5 : lea     r9, [rdx+r9]                      ,R9 <--   14006024d
1400d86bf : mov     [rbp+8], r9
1400d86c9 : movzx   r8, r8b
1400d86cd : sub     r8, 1                             ,R8 <--   b4  ,RF <--   6
1400d86d1 : jmp     loc_1400D7E10
1400d7e11 : cmp     r8, 0C8h                          ,RF <--   93
1400d7e18 : jnb     loc_1400D9954
1400d7e1e : lea     r9, word_1400DB5AA                ,R9 <--   1400db5aa
1400d7e25 : mov     r8, [r9+r8*8]                     ,R8 <--   d60f2
1400d7e29 : lea     r9, cs:140000000h                 ,R9 <--   140000000
1400d7e30 : jmp     loc_1400D6558
1400d6559 : add     r8, r9                            ,R8 <--   1400d60f2
,RF <--   2
1400d655c : jmp     r8
//进入handle
```

[rbp + 8] 是 V_RIP，放入r9，然后从[R9] 取出 tvmopcode 放入r8b 然后异或 0x5D，然后 r9 +1(上面是混淆过的，实际效果就是 + 1 )。

R8b - 1 如果大于0xC8，就跳转到`1400D9954`，说明这是未知的tvmopcode，出错。`1400D9954`是int3指令。

然后 handleTable 放入 R9，取表项 [R9 + R8 *8]，即为这个handle的偏移，加上基址0x140000000，即为这个handle 的实际处理地址：1400d60f2，通过 jmp r8 跳转过去。
