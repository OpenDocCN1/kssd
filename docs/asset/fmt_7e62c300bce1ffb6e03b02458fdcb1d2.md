
# [转载] RDTSC 指令的使用以及相关问题

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-510.htm`](https://www.pediy.com/kssd/pediy07/pediy7-510.htm)

RDTSC 指令的使用以及相关问题  
Author：Lenus  
From: `www.popbase.net`  
E-mail：`Lenus_M@163.com`

---

## 1. 前言

最近碰到的 `RDTSC` 指令的问题，感觉挺烦人的，今天下午搞明白了一些东西就先放上来了。

---

## 2. 正文

### 1. 使用 RDTSC 来 anti-debug

以下内如引用 intel 指令手册。

//引用开始

`RDTSC—Read Time-Stamp Counter`

`Opcode Instruction Description`

`0F 31 RDTSC Read time-stamp counter into EDX:EAX`

`Description`

Loads the current value of the processor’s time-stamp counter into the `EDX:EAX` registers. The time-stamp counter is contained in a 64-bit `MSR`. The high-order 32 bits of the `MSR` are loaded into the `EDX` register, and the low-order 32 bits are loaded into the `EAX` register. The processor increments the time-stamp counter `MSR` every clock cycle and resets it to `0` whenever the processor is reset.

The time stamp disable (`TSD`) flag in register `CR4` restricts the use of the `RDTSC` instruction. When the `TSD` flag is clear, the `RDTSC` instruction can be executed at any privilege level; when the flag is set, the instruction can only be executed at privilege level `0`. The time-stamp counter can also be read with the `RDMSR` instruction, when executing at privilege level `0`.

The `RDTSC` instruction is not a serializing instruction. Thus, it does not necessarily wait until all previous instructions have been executed before reading the counter. Similarly, subsequent instructions may begin execution before the read operation is performed.

This instruction was introduced into the `IA-32` Architecture in the `Pentium` processor.

`Operation`

```
IF (CR4.TSD ← 0) OR ((CR4.TSD ← 1) AND (CPL=0))
    THEN
       EDX:EAX ← TimeStampCounter;
    ELSE (* CR4 is 1 and CPL is 1, 2, or 3 *)
       #GP(0)
FI;
```

`Flags Affected`

`None.`

`Protected Mode Exceptions`

`#GP(0) If the TSD flag in register CR4 is set and the CPL is greater than 0.`

`Real-Address Mode Exceptions`

`#GP If the TSD flag in register CR4 is set.`

`Virtual-8086 Mode Exceptions`

`#GP(0) If the TSD flag in register CR4 is set.`

//引用结束

英文太多，我也看不懂多少，大概的意思是这样，将计算机启动以来的 `CPU` 运行周期数放到 `EDX：EAX` 里面，`EDX` 是高位，`EAX` 是低位。

`CPU` 运行周期数指的是 `CPU` 的一个时钟触发吧，就是应该是一个上升沿或者一个下降沿表示一个周期。

有一点你应该明白

周期数 / `CPU` 主频 = `CPU` 通电以来的执行秒数，也就是 `GetTickcount` 干的事情。

或者这样说使用两次 `RDTSC`，把两次的结果相减，得到“间隔周期数”，间隔周期数 / `CPU` 主频 = `CPU` 执行这两条指令间的秒数

这样我们就能使用太来做一些 `anti` 了

基本的思想是：

```
RDTSC
mov temp_1,eax           ;edx 估计不会改变
RDTSC
sub eax,temp_1
cmp eax,isDebug_number   ;isDebug_number 是一个自定义的小数目作为判断临界值
jg  @F
   ;如果比 isDebug_number 小，说明没有 Debug
@@:
   ;如果比 isDebug_number 大，说明有 Debug
```

因为在使用 `Debug` 的单步跟踪的时候，我们就算怎么迅速的 press `F7`（`F8`）需要的时间也大概要 0.0 几秒吧，如果让 `CPU` 来执行这些指令那么就可能只用 0.0000000 几秒....

所以说这个 `isDebug_number` 临界值还是很好选取的！

好了基本上可以来总结一下了：

`RDTSC` 的 `anti` 就是象 `GetTickcount` 一样，使用判断两个指令间执行的周期数（时间）来判断是否有 `debug`，因为我们确信 `cpu` 执行的周期数（时间）会远远低于某个值，解除他的办法也很简单，比如说你可以找到 `cmp eax,isDebug_number` 这一句，然后就不用我说什么了。

btw：

对于现在论坛上讨论得沸沸扬扬的 `[MSLRH]0.31` 壳，我没有仔细看他是怎么使用这个指令的，但是我找到有关键的比较处，我想应该差不多吧。

附：

`GetTickcount` 返回的是本次 `Windows` 启动以来的 `ms` 数，得到的时间数值直接在 `eax` 中返回，由于这是一个 32 位的整数，可以表示的范围是 `1～ffffffffh ms`，所以当 `Windows` 连续运行 49.7 天以后，计数器会清零并重新开始。

### 2. 深入研究 RDTSC 带来的问题

到这里基本上 `RDTSC` 用来 `anti` 的办法就是这样，下面的内容是对其他的一些问题的讨论，没有兴趣的朋友可以不用向下看了。

两个实验的程序下载：  
点击浏览该文件

#### 实验一：

下面是一个测试实例：

```
.data
szCaption       db  'this is a rdtsc test....',0
szMsg           db  '两个指令间经过的周期是：%08X',0
szBuffer        dd  128 dup(?)
Temp            dd ?
.code

   start:

                 rdtsc 

   push eax              ；将第一次的结果保存起来
   rdtsc
   sub eax,[esp]         ；向减得到“间隔周期数”
   mov Temp,eax
   invoke  wsprintf,addr szBuffer,addr szMsg,Temp         ；转化一下格式
   invoke MessageBox,0,addr szBuffer,addr szCaption,MB_OK
   invoke ExitProcess,0     

end  start    
```

结果：  
1. 如果直接执行我的机器上的结果是：`00000035`  
2. 如果是用 `OD` 的 `F9` 执行完上面的结果是：`0085B662`  
3. 如果用 `OD` 的 `F8`（`F7`）执行上面的结果是：`13DCCEA5`  
大家可以看到基本上不是几个数量及的了吧......

#### 实验二：

对于上面的两个 `rdtsc` 指令之间只有一个指令就是 `push eax`，而我们知道，之所以上面的 `anti` 会导致有大数量及的周期数是因为，在 `debug` 等待我们处理的时候（停在 `push eax` 的时候）`CPU` 采用多任务处理的机制，将这个线程挂起，把 `CPU` 的时间片分给了其他线程，因为实在是太快了导致就象多个线程同时运行一样，而停在 `push eax` 的时候虽然时间很短 0.0 几秒，但是 `CPU` 的时间片却已经轮回了很多次。

好了，现在问题也就来了：会不会这样，在两个 `rdtsc` 指令之间有很多的指令，导致 `CPU` 在还没执行到第二个 `rdtsc` 的时候，时间片已经用完了，这时等到时间片再回到这个线程的时候，`CPU` 的“间隔周期数”已经是很大的值了。

下面我们就这个问题来做个实验

核心代码：

```
invoke  CreateFile,addr lpFileName,GENERIC_WRITE,\   ;先打开文件
                    NULL,NULL,OPEN_ALWAYS,\
                    FILE_ATTRIBUTE_NORMAL,NULL
;省略掉若干指令
        rdtsc
        mov ecx,10000h       ；修改这个数可以看到不同的结果
;做个无用的循环            
Do_nothing:     nop
                 loop Do_nothing
        push eax
        rdtsc
;省略掉若干指令
        invoke  WriteFile,hFile,addr szInput,0ah,\        ;把间隔数写到文件中
                 addr NumberOfBytesWritten,NULL
```

让我们看看我们得到的 `txt` 文件

```
000C432D    <-------线程被打断
0004004E
00043FFC
0004001D
00041FE4
0004001B
00043663
0004001B
00041FCC
0004001B
00041FBC
002EF50D    <-------线程被打断
00042245
00042448
0004207C
0004001B
00041FC8
000608AF    <-------线程被打断
0004002F
00042181
0004001B
0004368F
0004001B
00041FE4
0004001B
00041FCC
0004329C
0004205A
00041FA0
0004001B
000452BF
000A4780    <-------线程被打断
```

可以看到基本上“间隔周期数”维持在 `00040000` 这个数量及，但是如果线程的时间片被打断了以后数字会突然增加，但是基本上不会到达 `01000000` 这个级别，所以我的结论是线程被打断的问题，对于执行了上万行（`mov ecx,10000h`）的指令来说差别也不是很大，那么我们一般来说可以不考虑他，当然这个“间隔周期数”还与系统中运行的线程数有关系，但是应该不是很大的问题。这个问题的研究就留给感兴趣的朋友了，我就到此为止了^-^

---

## 3. 总结

对于使用 `rdtsc` 来做 `anti` 实在不是一个明智只举，我认为他与 `GetTickcount` 一样让人有一点不舒服的感觉。所以个人来说知道他的原理就可以大家不要用烂他就好了，好的 `anti-debug` 还有很多。

---

## 4. 后话

在此要特别感谢小 love，感谢他教会我这么多的东西，祝他新的学期能够学习进步。

`4dd466d0e1e0cc123afbe504da2462ca`
