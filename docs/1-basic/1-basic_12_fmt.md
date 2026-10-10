
# 汇编优化提示

> 原文：[`www.pediy.com/kssd/pediy11/119761.html`](https://www.pediy.com/kssd/pediy11/119761.html)

Assembly Optimization Tips  

汇编优化提示  

【作者】：Mark Larson  

【翻译】：Dhuta  

暑假瞄了一些汇编优化的文章，感觉这篇有点意思。尽管英文水平不咋地，还是倔起牛劲翻译了下。肯定有不好的地方，大家海涵~英文原文附件给出~如果有什么错误还望批评指正~另外，如果 admin 感觉可以加精的话就麻烦下了![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")~  

一段时间过去了，增加一点内容。《怎样优化 Pentium 系列处理器的代码》[`www.codingnow.com/2000/download/pentiumopt.htm`](http://www.codingnow.com/2000/download/pentiumopt.htm)这篇文章很不错，而且和本文有许多联系。还有，要强烈推荐一本书，昨天才在图书馆看到的，对理解底层东西很有帮助----周明德教授的《64 位微处理器系统编程和应用编程》[`baike.baidu.com/view/3100612.htm`](http://baike.baidu.com/view/3100612.htm)。该书系统地介绍了 CPU 体系结构及一些汇编指令，让我们的视野更开阔！Oh！Yeah！![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")  

## 正文

你需记住的最重要的事情就是代码花费的时间！不同的方法可以加速你的代码或者不能，所以你要多多尝试。因而计算代码花费的时间来看看你尝试的每个方法是否可以提速是件很重要的事情。

## 初级

### <1>释放所有 8-CPU 寄存器，以使它们用于你自己的代码中
```
push ebx  
push esi  
push edi  
push ebp                    ; 必须在改变 ESP 之前完成
    ; 装载 ESI、EDI 和其他传递栈中值的寄存器，这必须在释放 ESP 之前完成。
    movd mm0,esp                ;  没有入栈/出栈操作超过此处，
    xor ebx,ebx                 ;  所以你怎样保存呢?一个变量嘛！
    mov esp,5  
inner_loop:  
    mov   [eax_save],eax        ; eax_save 是一个全局变量不可能是局
                ; 部的，因为那要求 EBP 来指向它。
    add   ebx,eax     
    mov   eax,[eax_save]        ;  存储 eax
    movd esp,mm0                ;  必须在做 POPs 之前完成
    pop   ebp  
    pop   edi  
    pop   esi  
    pop   ebx  
    ret
```

### <2>寄存器的最大化使用

多数高级点的编译器会产生非常多的到内存变量的访问。我通常避免那样做，因为我尽力把这些变量放在寄存器中。所以要使 8-CPU 寄存器自由使用，以备燃眉之急~

### <3>复杂指令

避免复杂指令(`lods`, `stos`, `movs`, `cmps`, `scas`, `loop`, `xadd`, `enter`, `leave`)。复杂指令就是那些可以做许多事情的指令。例如，`stosb` 向内存写 1 字节，同时增加 `EDI`。 这些复杂指令阻止早期的 Pentium 快速运行，因为处理器的发展趋势是力使自己更像精简指令集计算机(RISC)。使用 `rep` 和字符串指令依旧很快----而这是唯一的例外。

### <4>不要再 P4 上使用 INC/DEC

在 P4 上用 `ADD`/`SUB` 代替 `INC`/`DEC`。通常前者更快些。`ADD`/`SUB` 耗费 0.5 `cycle` 而 `INC`/`DEC` 耗费 1 `cycle`.

### <5>循环移位(rotating)

避免用寄存器作循环移位计数器；立即数作计数器时，除了 1，不要用其他值。

### <6>消除不必要的比较指令

通过合理运用条件跳转指令来消除不必要的比较指令，但是，这些条件跳转指令时基于标志位的。而这些标志位已经被前面的算术指令设置了。

```
dec     ecx  
cmp     ecx,0  
jnz     loop_again
```

优化为：

```
dec     ecx  
jnz     loop_again
```

### <7>lea 指令的使用

`lea` 非常酷，除了在 P4 上它表现得慢一点。你可以在这一条指令中执行许多数学运算，并且它不会影响标志寄存器。所以你可以把它放在一个正在被修改的寄存器和一个涉及标志位的条件跳转指令中间。

```
top_of_loop:  
    dec   eax  
    lea   edx,[edx*4+3]     ; 乘 4 加 3。不影响标志位
    jnz   top_of_loop       ; 所以 jnz 判断的是 dec eax 执行后的状态字
```

### <8>ADC 和 SBB 的使用

许多编译器都不充分利用 `ADC` 和 `SBB`。你可以用它们很好地提速，如：把两个 64-bit 的数加在一起或者两个大数相加。记牢：`ADC` 和 `SBB` 在 P4 上慢。当你手头有类似工作时，你可以使用 `addq` 和用 `MMX` 来完成。所以第二个优化建议是：用 `MMX` 做这样的加减法。但你的处理器必须支持 `MMX`。

```
add    eax,[fred]  
adc    edx,[fred+4]
    ; 下面的 3 条指令完成同样的功能
movd  mm0,[fred]       ; 得到 MM0 中的 32-bit 的值
movd  mm1,[fred+4]     ; 得到 MM1 中的 32-bit 的值
paddq mm0,mm1          这是一种未优化的方法，实际上，你可以在前面的循环中预取 MM0 和 MM1。我这样做是为了好理解。
```

### <9>ROL, ROR, RCL, RCR 和 BSWAP

使用 `BSWAP` 指令把 Big Endian 数据转换成 Little Endian 格式是很酷的一种方法。你也可以使用它临时存储寄存器的高位的的值(16-bit 或 8-bit)。类似地，你可以使用 `ROL`/`ROR` 来存储 8-bit 或 16-bit 值。这也是获得更多“寄存器”的一种方法。如果你处理的都是 16-bit 的值，你可以把你的 8 个 32-bit 的寄存器转换成 16 个 16-bit 的寄存器。这样你就有更多寄存器可以使用了。`RCL` 和 `RCR` 可以很容易地被使用在计算寄存器中比特位(0 or 1)的多少。牢记：P4 上 `ROL`, `ROR`, `RCL`, `RCR` 和 `BSWAP` 速度慢。循环移位指令大约比 `BSWAP` 快 2 倍。所以，如果你必须在 P4 机上使用上述指令，还是选循环移位指令吧。

```
xor   edx,edx           ; 设置两个 16-bit 的寄存器为 0
mov   dx,234            ; 设置低位寄存器为 234
bswap edx               ; 高低位寄存器互换
mov   dx,345            ; 设置当前低位寄存器为 345
bswap edx               ; 当前高低位互换
add   dx,5              ; 当前低位寄存器加 5
bswap edx               ; 高低位互换
add   dx,7              ; 当前低位寄存器加 7
```

### <10>字符串指令

许多编译器没有充分利用字符串指令( `scas`, `cmps`, `stos`, `movs` 和 `lods`)。所以检测这些指令写成的函数是否比一些库函数速度快是非常有意义的。例如：当我在看 VC++的 `strlen()`函数时，我真的很惊讶。在 radix40 代码中，处理一个 100 字节的字符串，它竟跑了 416 `cycles`！！！我认为这慢地荒诞！！！

### <11>以乘代除

If you have a full 32-bit number and you need to divide, you can simply do a multiply and take the top 32-bit half as the result. This is faster because multiplication is faster than division. ( thanks to pdixon for the tip).

如果你有一个满 32-bit(`full 32-bit`)的数要做除法，你可以简单地做乘法，然后取高 32-bit 部分作为结果。这样会快些，因为乘法比除法快！(感谢 `pdixon` 提供了这个 tip)(译者注：由于水平有限，此 tip 翻译尚待商讨，而且不能给出一个例子。还仰仗各位帮忙。)

### <12>被常数除

这儿有一些很好的信息----怎样被常数除(在 Agner Fog 的 `pentopt.pdf` 中)。我(Mark Larson)写了一个小程序：可以根据你输入的除数自动产生汇编代码序列。我会继续探究探究，然后贴出来。这是 Agner 的文档的链接。Agner's Pentopt PDF (http://www.agner.org/assem/)

### <13>展开(Unrolling)

这是一个指导方针。在一般优化策略里，“展开”往往被忽略，因此我想为它加个注脚。我总是用数值总数相等的宏来建立我的“展开”结构。那样的话，你可以尝试不同的值来看看哪种是最容易的。你希望这个展开“安身”于 L1 代码缓存(或追踪缓存,trace cache)。使用等价(equate)语句可以很容易地尝试不同的展开值来达到最快的速度。

```
UNROLL_AMT       equ   16   ; # 展开循环的次数
UNROLL_NUM_BYTES equ    4   ; # 一次循环处理的字节数
mov     ecx,1024
looper:
offset2 = 0
REPEAT UNROLL_AMT
    add     eax,[edi+offset2]
offset2 = offset2 + UNROLL_NUM_BYTES
    add     edi,UNROLL_AMT * UNROLL_NUM_BYTES   ; 按 16*4 字节来处理
    sub     ecx,UNROLL_AMT             ; 从循环计数器中减去展开循环的次数
    jnz     looper
```

### <14>MOVZX 指令

使用 `MOVZX` 来避免偏爱的寄存器受阻。我使用 `MOVZX` 非常多。许多人首先将满 32-bit 寄存器(`full 32-bit register`)XOR 一下。但是，`MOVZX` 无需这个多余的 `XOR` 指令而可以完成同样的事情。而且你必须提前做 `XOR` 运算，之后才能使用寄存器，并且这个操作是花费一定时间的。但有 `MOVZX` 在，你就无需担心啦。

### <15>用 MOVZX 来避免 SHIFT 和 AND 指令的使用

我用汇编来对 C 代码中的位运算提速。`the_array` 是一个 `dword` 的数组。代码功能是得到数组中一个 `dword` 的任意字节。`Pass` 是一个值为 0-3 的变量。因此这个功能有如下 C 代码：

```
unsigned char c = ((the_array[i])>>(Pass<<3)) & 0xFF；
； 我为了避免 Pass 变量的使用，展开了循环 4 次。所以我得到了如下更多的代码：
unsigned char c = (the_array[i])>>0) & 0xFF；
unsigned char c = (the_array[i])>>8) & 0xFF；
unsigned char c = (the_array[i])>>16) & 0xFF；
unsigned char c = (the_array[i])>>24) & 0xFF；
```

在汇编中，我摒弃 `SHIFT` 和 `AND` 会发生什么呢？节约了我 2 个指令！更不用提 P4 上 `SHIFT` 指令非常慢(4 `cycles`!!!)的事实。所以如果可能，尽量避免使用 `SHIFT` 指令。我们以第三行为例，所以只需右移 16 位即可：

```
mov     eax,[esi]  ; esi 指向 the_array 数组
shr     eax,16
and     eax,0FFh  
； 那么我们怎样避免 SHR 和 AND 的使用呢？我们举例对 dword 中的第三个字节做 MOVZX 运算。
movzx   eax,byte ptr [esi+2]   ; unsigned char c = (the_array[i])>>16) & 0xFF；
```

### <16>align 指令

该伪指令非常重要，它可以对齐你的代码和数据来很好地提速。对代码，我通常按 4 字节边界对齐。对于数据呢，2 字节数据按 2 字节边界对齐，4 字节数据按 4 字节边界对齐，8 字节数据按 8 字节边界对齐，16 字节数据按 16 字节边界对齐。一般情况，如果不以一个 16-bit 边界对齐 SSE 或 SSE2 数据的话，将会产生一个异常。如果有 VC++开发包，你可以在 VC++环境中对齐你的数据。VC++加入了对静态数据和动态内存的支持。对于静态数据，你可以使用 `__declspec(align(4))`来按 4 字节边界对齐。

### <17>用于 2 的幂的 BSR 指令

你可以把 `BSR` 命令用于变量的 2 的最高幂的计数。

### <18>用 XOR 置寄存器为 0

这是非常陈旧的常识了，但是我依然要重提一下。它也有一个侧面好处----消除对寄存器的依赖性。这就是人们使用寄存器之前用 `XOR` 置零成风的原因。但我宁愿使用 `MOVZX`，因为 `XOR` 很狡猾~(看看我前面对 `MOVZX` 的讨论)在 P4 上，也增加了 `PXOR` 支持，以消除对 `XOR` 的依赖性。我认为 P3 做了同样的事情。

### <19>使用 XOR 和 DIV

如果你确定你的数据在做除法时是无符号数，使用 `XOR EDX, EDX`，然后 `DIV`。它比 `CDQ` 和 `IDIV` 快。

### <20>尽量避免明显的依赖关系

如果你修改一个寄存器，然后在紧跟的下一行中让它和某个值进行比较，相反地，你最好在两者之间插入其他的寄存器的修改指令。依赖就是任何时间你修改一个寄存器，然后紧跟着对它读或写。(译者注：其实就是 AGI 延迟； AGI: Address Generation Interlock)

```
inc edi  
inc eax  
cmp eax,1    ; 这行依赖于上一行，所以产生延迟
jz fred
； 移动周围的指令到这儿可以打破这种依赖
inc eax  
inc edi  
cmp eax,1  
jz fred
```

### <21>P4 机上避免使用的指令

在 P4 机上避免使用如下指令： `adc`, `sbb`, `rotate` 指令, `shift` 指令, `inc`, `dec`, `lea`, 和任何耗费多于 4 `uops`(微指令)的指令。你怎么知道当前运行代码的处理器是 P4？`CPUID` 命令！

### <22>使用查找表

在 P4 机上，你可以通过建立查找表，来规避长时间延迟的指令(前面已列出来)。幸而 P4 机的内存速度很快，所以如果建立的查找表不在 cache 里，它也并不会很大地影响性能。

### <23>使用指针而不是计算下标(索引)

许多时候，C 语言中的循环会出现两个非幂数的乘法。你可以很容易地用加法来完成。下面是一个使用结构体的例子：

```
typedef struct fred  
{  
    int fred；  
    char bif；  
}freddy_type；
freddy_type charmin[80]；
```

`freddy_type` 结构的大小是 5 字节。如果你想在循环中访问它们，编译器会产生此种代码----对每个数组元素的访问都乘以 5！！！Ewwwwwwwwwwwww(译者注：语气词)！！！那么我们应该怎样做呢？

```
for ( int t = 0； t < 80； t++)  
{  
  charmin[t].fred = rand()； // 编译器为了得到偏移量，竟乘以了 5，EWWWWWWWW！
  charmin[t].bif = (char)(rand() % 256)；  
}
```

在汇编中，我们以偏移量 0 开始，这指向了数据第一个元素。然后我们每次把循环迭代器加 5 来避免 `MUL` 指令的使用。

```
mov   esi,offset charmin  
mov   ecx,80  
fred_loop:  
   ；...freddy_type 结构中 FRED 和 BIF 元素的处理命令
   add   esi,5        ; 指向下一个结构的入口地址
   dec   ecx  
   jnz   fred_loop
```



MUL 指令的规避也应用在循环中。我曾见过人们在循环中以乘来实现变量增加或者终止条件。相反，你应尽量用加法。

## <24>遵从默认分支预测

尽量设计你的代码使向后的条件跳转经常发生，并且向前的条件跳转几乎从不发生。这显然与分支预测有关。CPU 中的静态分支预测器(`static branch predictor`)只使用简单的规则来猜测一个条件跳转是否发生。所以应使向后跳转的循环分支靠近结束。然后让同一循环的特殊退出条件(`exit condition`)执行一个向前的跳转(这个跳转只在此跳转不经常发生的这个特定的条件下退出)。

## <25>消除分支

如果可能，消除分支！这是显然的，但我见过许多人在他们的汇编代码中使用了太多的分支。保持程序的简单。使用尽可能少的分支。

## <26>使用 CMOVcc 来消除分支

我曾见到过 `CMOVcc` 指令确实比条件跳转指令快。所以我建议使用 `CMOVcc` 而非条件跳转指令。当在你的跳转不容易被分支预测逻辑猜到的情况下，它可能会更快。如果你遇到那种情况，设定基准点(`benchmark`)看看！

## <27>局部变量 vs 全局变量

在一个过程模块(子函数)中使用局部变量而不是全局变量。如果你使用局部变量，你会得到更少的缓存未命中(`Cache miss`)。

## <28>地址计算

在你需要某地址之前计算它。不得不说你为了得到某一特定地址必须计算一些令人讨厌的东西。例如地址乘 20。你可以在需要该地址的代码之前预算它。

## <29>小点儿的寄存器

有些时候，使用小点儿的寄存器可以提升速度。我在 `radix40` 代码中验证了它。如果你使用 `EDX` 来修改下面的代码，它跑得会慢些。
```
movzx           edx,byte ptr [esi]     ;从 ascii 数组取数据
test            dl,ILLEGAL              ;位测试
jnz             skip_illegal
```

## <30>指令长度

尽量使你的指令大小保持在 8 字节以下。

## <31>使用寄存器传递参数

如果可能，尝试用寄存器传递参数而不是栈。如果你有 3 个变量要压栈作为参数，至少有 6 个内存读操作和 3 个内存写操作。你不得不把每个变量由内存读入 CPU 寄存器然后把它们压入栈。这是 3 个内存读操作。然后压向栈顶产生 3 个写操作。然后为什么你会压你从不使用的参数呢？所以，又出现了最少 3 个读栈操作。(译者注：两内存变量不能直接传递数据)

## <32>不要向栈传递大数据

不要向栈传递 `64-bit` 或 `128-bit` 的数据(或者更大)。相反，你应该传递指向该数据的指针。

；=========================中级=========================

## <33>加法方向

寄存器加向内存比内存加向寄存器速度更快。这与指令耗费的微指令(`micro-ops`)的多少有关。所以，你知道该怎么做了。
```
add    eax,[edi]       ;如果可能，不要这样做
add    [edi],eax       ;这才是首选
```

## <34>指令选择

尽量选取产生最少微指令和最少延迟的指令。

## <35>未对齐字节数据流的双字(dword)对齐处理

对于没有 4 字节边界对齐的缓冲区，一次分解出一个 `dword` 会使性能湮没。(译者注：因为地址 32-bit 对齐时，处理速度最快)你可以通过处理开始的 X(0-3)字节直到遇到一个 4 字节对齐的边界处的方法来规避这种情况。

## <36>使用 CMOVcc 来复位一个无限循环指针

如果你想多次传递一个数组，并当达到数组末端的时候复位到开始处，你可以使用 `CMOVcc` 指令。
```
dec ecx             ; 减少数组索引
cmovz ecx,MAX_COUNT ; 如果在开始处，复位索引为 MAX_COUNT(结尾处)。
```

## <37>以减法来代替乘以 0.5 的乘法

这可能不会对所有情况奏效除了 `real4` 变量乘以 0.5，或者被 2 除(`real4` 变量是浮点数)，你只需把指数减 1 即可。但对 0.0 不奏效。对 `real8` 变量，要减去 `00100000h`。(这个 tip 是 `donkey` 贴出来的,donkey posted this)

```
.data
  somefp  real4  2.5
.code
  sub dword ptr [somefp],00800000h    ;用 2 除 real4
```

## <38>自修改代码

P4 优化手册建议避免自修改代码的使用。我曾经见到过几次自修改代码可以跑得更快的情况。但是每次使用前你都要明确它在当前情况下会更快。(译者注：自修改代码即 INC/DEC)

## <39>MMX, SSE, SSE2 指令

许多编译器对 `MMX,SSE` 和 `SSE2` 不会产生很好的代码。`GCC` 和 `Intel` 编译器对此做了更多工作，所以较其他产品好一些。但是你自己的"大脑+手"编译器在这项工作中的应用仍然是很大的成功。

## <40>使用 EMMS

`EMMS` 在 `Intel` 处理器上倾向为很慢的指令。在 `AMD` 上它比较快。通常我不会在例行的基础程序中使用它，因为它慢。我很少在有许多 `MMX` 指令的程序中再使用很多浮点指令。反之依然(`vice versa`)。所以我经常在做任何浮点运算之前等着执行 `EMMS`。如果你有许多浮点和很少的 `MMX`，那么应在你调用的所有的 `MMX` 子程序(如果有的话)执行完再执行 `EMMS`。然而，在每个执行 `MMX` 的子程序中加入 `EMMS` 会使代码变慢。

## <41>转换到 MMX,SSE,SSE2

你的代码能转换到 `MMX, SSE, 或者 SSE2` 吗？如果能，你可以并行处理来极大的提升速度。

## <42>预取数据

这往往未被充分利用。如果你处理一个非常大的数组(`256KB` 或更大)，在 P3 及以后处理器上使用 `PREFETCXH` 指令可以使你的代码无论在什么地方都可以提速 10-30%。但事实上，如果你没有合理使用的话，代码性能还可能降级。在这方面，“展开”表现良好，因为我把它展开为用此指令预取的字节的整数倍。在 P3 上，此倍数为 32，而在 P4 上，它是 128。这就意味着你可以在 P4 机上很容易地展开循环来一次处理 128 字节，你可以从预取和展开中得到好处。但并不是在每种情况下展开为 128 字节都会有最好的提速。你可以尝试不同的字节数。
```
UNROLL_NUM_BYTES equ    4                       ; 一次循环要处理的字节数
UNROLL_AMT       equ    128/UNROLL_NUM_BYTES    ;我们想展开循环以使它每次处理 128 字节
    mov     ecx,1024
looper:
  offset2 = 0
REPEAT UNROLL_AMT
  prefetchnta [edi+offset2+128]         ; 在我们需要之前预取 128 字节到 L1 缓存
  add     eax,[edi+offset2]
  offset2 = offset2 + UNROLL_NUM_BYTES
  add     edi,UNROLL_AMT * UNROLL_NUM_BYTES   ; 我们处理 16*4 字节
  sub     ecx,UNROLL_AMT                ; 调整迭代器到下一循环
  jnz     looper
```

## <43>缓存模块化(Cache Blocking)

不得不说，你必须对内存中的大数组调用许多过程(子函数)。你最好把数组分成块儿并装入缓存(`Cache`)来减少缓存未命中(`cache miss`)。例如：你正在执行 3D 代码，第一个过程可能传递坐标，第二个过程可能是规模大小，第三个可能是变换方式。所以与其在整个的的大数组里翻箱倒柜似地找，你应该把数据大块(`chunk`)分成适于缓存(`Cache`)的小块。然后调用此 3 个过程。然后进行下面数据大块的类似操作。

## <44>TLB 启动(TLB Priming)

`TLB` 就是旁路转换缓冲，或称页表缓冲(`Translation Lookaside Buffer`)，是虚拟内存地址到物理内存地址的转换部件。它通过对页表入口点的快速访问来提升性能。未在 `TLB` 缓存的代码或数据会促使缓存未命中，这就降低了代码速度。解决方法就是：在你必须读某页之前预读该页的一个数据字节。我会在后面的 tips 中提供一个例子。

## <45>混入代码来消除依赖

在 C 代码中，C 编译器把不同的代码块分别对待。当进入汇编语言级别时，你可以混入(`intermix`)它们来消除依赖。

## <46>并行处理

许多编译器没有充分利用 CPU 有 2 个 ALU 流水线的事实，而 ALU 是人们使用的大部分。在 P4 机上你会更得意----如果操作得当，你可以在一个指令周期执行 4 个 ALU 运算指令。如果你把任务分配，并行处理之，这也会消除依赖。真是一箭双雕！在循环中采取这个策略。
```
looper:
        mov     eax,[esi]
        xor     eax,0E5h        ;依赖上一行
        add     [edi],eax       ;依赖上一行
        add     esi,4
        add     edi,4
        dec     ecx
        jnz     looper
  ;那么我们如何使它并行化并且减少依赖呢？
looper:
        mov     eax,[esi]
        mov     ebx,[esi+4]
        xor     eax,0E5
        xor     ebx,0E5
        add     [edi],eax
        add     [edi+4],ebx
        add     esi,8
        add     edi,8
        sub     ecx,2
        jnz     looper
```

## <47>避免内存访问

重新构建代码来避免内存访问(或者其他 I/O 操作)。一种方法就是在向内存写一个值的时候，先在一个寄存器中累加它。下面给出一个例子。在这个例子里，假设每次循环我们从源数组向目的数组(元素为 `dword` 大小)连续相加 3 个字节的值。目的数组已经置 0。
```
mov     ecx,AMT_TO_LOOP
looper:
        movzx byte ptr eax,[esi]
        add     [edi],eax
        movzx byte ptr eax,[esi+1]
        add     [edi],eax
        movzx byte ptr eax,[esi+3]
        add     [edi],eax
        add     edi,4
        add     esi,3
        dec     ecx
        jnz     looper
  ;我们可以在寄存器中累加结果，然后只需向内存写一下即可。
        mov     ecx,AMT_TO_LOOP
looper:
        xor     edx,edx                 ;置 0 以存储结果
        movzx byte ptr eax,[esi]
        add     edx,eax
        movzx byte ptr eax,[esi+1]
        add     edx,eax
        movzx byte ptr eax,[esi+3]
        add     edx,eax
        add     esi,3
        mov     [edi],edx
        add     edi,4
        dec     ecx
        jnz     looper
```

## <48>何时转换 call 为 jump

如果子程序的最后一个语句是一个 `call`，考虑把它转换为一个 `jump` 来减少一个 `call/ret`。

## <49>使用数组作为数据结构

(这个 tip 不是只针对汇编的，但汇编表现更优异)你可以使用一个数组来实现数据的结构(例如树和链表)。通过使用数组，内存会无缝连接，代码会因更少的缓存未命中而提速。

；=========================高级=========================

## <50>避免前缀

尽量避免使用前缀。段超越(`segment overrides`),分支标志(`branch hints`),操作数大小强制转换(`operand-size override`),地址大小强制转换(`address-size override`),封锁数据指令(`LOCKs`),重复前缀(`REPs`)等都会产生前缀。前缀会增加指令的长度，执行时间也有所延长。

## <51>将代码中的读/写操作分组

如果 `bus` 总线上有许多交互的读命令和写命令，考虑分组。同一时间处理更多的读和写命令。下面的是我们要避免的：
```
mov eax,[esi]
mov [edi],eax
mov eax,[esi+4]
mov [edi+4],eax
mov eax,[esi+8]
mov [edi+8],eax
mov eax,[esi+12]
mov [edi+12],eax
  ;将读和写指令分组
mov eax,[esi]
mov ebx,[esi+4]
mov ecx,[esi+8]
mov edx,[esi+12]
mov [edi],eax
mov [edi+4],ebx
mov [edi+8],ecx
mov [edi+12],edx
```

## <52>充分利用 CPU 执行单元(EU,execution units)来加速代码

选择在不同处理单元执行的命令。如果你能合理地这样做的话，执行代码的时间会等于吞吐时间(`throughput time`)而没有延迟时间(`latency time`)。对许多指令来说，吞吐时间是较少的。

## <53>交错 2 个循环以同步执行

你可以展开一个循环 2 次，而不是一条接另一条的运行命令，你可以同步运行它们。这为什么很有用？有 2 个原因。第一，有时你要执行必须使用某个寄存器的指令并且这些指令有很长的延迟。例如 `MUL/DIV`，两个连在一起的 `MUL` 指令会产生对 `EDX:EAX` 的依赖和争用。第二，有时一些指令本身就有很长的延迟。所以，你自然想尝试在在一个循环后面放置一些来自另一个循环的指令来减少延迟直到它返回结果。P4 机上的许多 `MMX, SSE` 和 `SSE2` 指令都采取这个策略。这儿有一个例子循环。
```
loop:
A1  ; instruction 1 loop 1
D2  ; instruction 4 loop 2
B1  ; instruction 2 loop 1
A2  ; instruction 1 loop 2
C1  ; instruction 3 loop 1
B2  ; instruction 2 loop 2
D1  ; instruction 4 loop 1
C2  ; instruction 3 loop 2
```

## <54>使用 MMX/SSE/SSE2 时比较指令设置的标志

当与 `MMX/SSE/SSE2` 打交道时，比较指令会产生对标志位的设置。在某些情况下，当你搜索一个文件中的模式(例如换行符)时，这会很有用。所以你可以使用它来搜索模式，而不仅仅来做数学运算。你可以使用 `MMX/SSE/SSE2` 中比较指令产生的标志来控制部分 `MMX` 或 `SSE` 寄存器上的数学运算。例如下面的代码片段：如果有 5，把 9 加到它 `MMX` 寄存器的 `dword` 部分。
```
  ; if (fredvariable == 5)
  ;       fredvariable += 9;
  ;-------------------------------
    movq  mm5,[two_fives]      ;mm5 有两个 DWORD 5 在里面
    movq  mm6,[two_nines]          ;mm6 有两个 DWORD 9 在里面
    movq  mm0,[array_in_memory]   ;取值
    movq  mm1,mm0                  ;回写
    pcmpeqd  mm1,mm5                ;mm1 现在在每个 DWORD 位置都为 FFFFFFFF
                  ;在 MM1 有一个 5，其他所有位置都为 0  
    pand  mm1,mm6                ;把 MM6 中不为 5 的位置置 0
    paddd  mm0,mm1                 ;只向 MM0 中值为 5 的位置加 9
```

## <55>PSHUFD 和 PUSHFW 指令



在 P4 的 MMX,SSE 和 SSE2 中，移动指令(`MOV`系列)速度慢。你可以在 SSE 和 SSE2 中使用"`pushfd`"，MMX 中使用"`pushfw`"来避免如上情况。它快 2 指令周期呢。但有一个警告：它是与微指令被分配加载到哪个流水线有关的。而没有掌握更多技术的时候，有时使用慢点的"`MOVDQA`"会比替代它的"`PUSHFD`"快。所以你要对你的代码精打细算。

```asm
pushfd xmm0,[edi],0E4h     ；拷贝 EDI 指向位置的 16 字节到 XMM0。0E4h 会直接拷贝。
pushfw mm0,[edi],0E4h      ；拷贝 EDI 指向位置的 8 字节到 MM0。0E4h 会直接拷贝。
```

## <56>直接写内存---绕开缓存(cache)

这是另一个优化内存处理的策略。如果你必须向许多内存空间(256KB 及以上)进行写操作，绕开缓存直接向内存写更快！如果你的 CPU 是 P3，你可以使用"`movntq`"或"`movntps`"指令。前者执行 8 字节的写操作，而后者是 16 字节。16-byte 写需要 16 字节对齐。在 P4 上，你还可以使用"`movntdq`"，它也可以用于 16 字节，但必须 16 字节对齐。这个方法在内存填充和内存拷贝中均适用，二者都做写操作。这里有一些样本代码。我必须自己动手并行使用 8 个 XMM 寄存器来帮助消除 P4 机 `MOVDQA` 指令的一些延迟。然而，为了帮助理解，我没那么做。

```asm
mov     ecx,16384           ；写 16384 个 16-bit 值，16384*16 = 256KB
                            ；所以我们正在拷贝一个 256KB 的数组
mov     esi,offset src_arr   ；指向必须以 16-bit 对齐的源数组的指针，否则会产生异常
mov     edi,offset dst_arr   ；指向必须以 16-bit 对齐的目的数组的指针，否则会产生异常
looper:
movdqa  xmm0,[esi]          ；工作在 P3 及以上
movntps [edi],xmm0          ；工作在 P3 及以上
add     esi,16
add     edi,16
dec     ecx
jnz     looper
```

## <57>使用 MMX/SSE/SSE2 时每个循环处理 2 个事件

在 P4 上，MMS/SSE/SSE2 指令的延迟那么长以至于我总是每个循环处理 2 个事件或者提前读取一个循环。如果你有足够的寄存器，可以多于 2 个事件。所有的各种各样的 MOVE(包括 `MOVD`)指令在 P4 上的速度都慢。所以 2 个 32-bit 的数字数组相加运算在 P4 上比 P3 上还慢。一个快点儿的方法可能就是每个循环(这个循环在 `FRED` 标号之前预读循环初始值 `MM0` 和 `MM1`)处理两个事件。你必须做的只是在数组元素个数为奇时进行特殊的处理；在最后检查一下，如果为奇数，加一个额外的 dword。这儿有个并没有提前读取值的代码段。我想，把它改为提前读取值是很容易的，所以我没有两个都贴出。下面的代码可以：在 P4 机上避免 `ADC` 这个速度慢的指令来把两个数组相加。

```asm
pxor    mm7,mm7     ； the previous loops carry stays in here
fred:
movd    mm0,[esi]   ； esi points to src1
movd    mm1,[edi]   ； edi points to src2, also to be used as the destination
paddq   mm0,mm1     ； add both values together
paddq   mm0,mm7     ； add in remainder from last add
movd    [edi],mm0   ； save value to memory
movq    mm7,mm0
psrlq   mm7,32      ； shift the carry over to bit 0
add     esi,8
add     edi,8
sub     ecx,1
jnz     fred
movd    [edi],mm7   ； save carry
```

## <58>预读 MMX 或 XMM 寄存器来规避长时间的延迟

在需要之前预读一个 SSE2 寄存器会提升速度。这是因为 `MOVDQA` 指令在 P4 上花费 6 cycles。这确实慢。鉴于它有如此长之延迟，我想在确定不会产生阻碍的地方提前读取它。这里有一个例子。

```asm
movdqa xmm1,[edi+16]  ；在我们需要之前读取入 XMM1，P4 上花费 6 cycles，不包括从缓存取的时间。
por xmm5,xmm0         ；做 OR 运算，XMM0 已预读。P4 上花费 2 cycles。
pand xmm6,xmm0        ；做 AND 运算，XMM0 已预读。P4 上花费 2 cycles。
movdqa xmm2,[edi+32]  ；在我们需要之前预读入 XMM2，P4 上花费 6 cycles，不包括从缓存取的时间。
por xmm5,xmm1         ；做 OR 运算，XMM1 已预读。P4 上花费 2 cycles。
pand xmm6,xmm1        ；做 AND 运算，XMM1 已预读。P4 上花费 2 cycles。
```

## <59>在一个或多个寄存器中累加一个结果来避免执行慢的指令

在一个或多个寄存器中累加一个结果来避免执行慢的指令。我用这个策略加速用 SSE2 写的比较/读循环。比较慢的指令是 `PMOVMSKB`。所以，我累加结果在一个寄存器中而不是每次循环都执行这个指令。对每个 4KB 的内存读操作，我会用 `PMOVMSKB`，它会很大地提速。下面我们通过分析一个使用 `PREFETCH` 和 `TLB` 启动的例子来证明。下面的代码有 2 个循环。内层循环被展开来处理 128 字节(P4 机上 `PREFETCH` 指令的预取字节数)。另一个循环被展开为 4KB。所以我可以使用 `TLB` 启动。如果你使用的系统没有使用 4KB 页大小，你不得不适当地修改你的代码。在拥有最大 6.4 GB/s 内存带宽的戴尔服务器(`Dell Server`)系统上，我测试了这段代码。我能够以 5.55 GB/s 做读和比较操作(在没有 Windows 环境下。在 Windows 环境下会运行地慢点)。我遗漏标号"`compare_failed`"的代码有 2 个原因：1)剪切/粘贴的代码已经够多了；2)它没有论证任何我要展现的技术。"`compare_failed`"的代码只是简单地(在 `PCMPEQD` 找到失败地址所属的最近的 4KB 内存块后)做一个 `REP SCASD` 来找到失败的地址。这个例子有非常巨大的代码量，所以我把它放在最后以免你读它的时候睡着；)(译者注：感觉下面的代码注释翻译出来有点别扭，而且原文也不难理解。故略。)

```asm
read_compare_pattern_sse2 proc near
        mov         edi,[start_addr]        ；Starting Address
        mov         ecx,[stop_addr]         ；Last addr to NOT test.
        mov         ebx,0FFFFFFFFh          ；AND mask
        movd        xmm6,ebx                ；AND mask
        pshufd      xmm6,xmm6,00000000b     ；AND mask
        movdqa      xmm0,[edi]              ；Get first 16 bytes
        mov         eax,[pattern]           ；EAX holds pattern
        pxor        xmm5,xmm5               ；OR mask
        movd        xmm7,eax                ；Copy EAX to XMM7
        pshufd      xmm7,xmm7,00000000b     ；Blast to all DWORDS
outer_loop:
        mov         ebx,32                  ；128 32 byte blocks
        mov         esi,edi                 ；save start of block
if DO_TLB_PRIMING
        mov         eax,[edi+4096]          ；TLB priming
endif                                 ；if DO_TLB_PRIMING
fred_loop:
        movdqa      xmm1,[edi+16]           ；read 16 bytes
        por         xmm5,xmm0               ；OR into mask
        pand        xmm6,xmm0               ；AND into mask
        movdqa      xmm2,[edi+32]           ；read 16 bytes
        por         xmm5,xmm1               ；OR into mask
        pand        xmm6,xmm1               ；AND into mask
        movdqa      xmm3,[edi+48]           ；read 16 bytes
        por         xmm5,xmm2               ；OR into mask
        pand        xmm6,xmm2               ；AND into mask
        movdqa      xmm0,[edi+64]           ；read 16 bytes
        por         xmm5,xmm3               ；OR into mask
        pand        xmm6,xmm3               ；AND into mask
        movdqa      xmm1,[edi+80]           ；read 16 bytes
        por         xmm5,xmm0               ；OR into mask
        pand        xmm6,xmm0               ；AND into mask
        movdqa      xmm2,[edi+96]           ；read 16 bytes
        por         xmm5,xmm1               ；OR into mask
        pand        xmm6,xmm1               ；AND into mask
        movdqa      xmm3,[edi+112]          ；read 16 bytes
        por         xmm5,xmm2               ；OR into mask
        pand        xmm6,xmm2               ；AND into mask
        por         xmm5,xmm3               ；OR into mask
        prefetchnta [edi+928]               ；Prefetch 928 ahead
        pand        xmm6,xmm3               ；AND into mask
        add         edi,128                 ；Go next 128byteblock
        cmp         edi,ecx                 ；At end?
        jae         do_compare              ；No, jump
        movdqa      xmm0,[edi]              ；read 16 bytes
        sub         ebx,1                   ；Incr for inner loop
        jnz         fred_loop
do_compare:
        pcmpeqd     xmm5,xmm7               ；Equal?
        pmovmskb    eax,xmm5                ；Grab high bits in EAX
        cmp         eax,0FFFFh              ；all set?
        jne         compare_failed          ；No, exit failure
        mov         edx,0FFFFFFFFh          ；AND mask
        pxor        xmm5,xmm5
        pcmpeqd     xmm6,xmm7               ；Equal?
        pmovmskb    eax,xmm6                ；Grab high bits in EAX
        cmp         eax,0FFFFh              ；All Set?
        jne         compare_failed          ；No, exit failure
        movd        xmm6,edx                ；AND mask
        pshufd      xmm6,xmm6,00000000b     ；AND mask
        cmp         edi,ecx                 ；We at end of range
        jb          outer_loop              ；No, loop back up
        jmp         compare_passed          ；Done!!! Success!!!
```

## <60>在循环内预取距离和位置



你会注意到，在上面的例子中，我之前预取了 928 字节而不是 128 字节（128 是 `P4` 机上的预取字节数）。为什么？`Intel` 建议在循环开始前预取 128 字节（2 `cache lines`）。但两种不同取法（在循环开始处或提前预取 128 字节）都会出错。我既没有在循环开始时预取也没之前预取 128 字节。为什么？当我研究这段代码时，我发现把 `PREFETCH` 指令放到循环周围并且改变它预取的偏移量可以使它运行得更快。所以反常得是，我写代码来尝试所有的循环内预取指令的位置和开始预取的偏移量的组合情况。这段代码写成一个汇编文件，而且把周围的 `PREFETCH` 指令移到循环内，同时修改开始预取的偏移量。然后一个 `bat` 文件编译这个修改的代码并且运行一个基准点（`benchmark`）。我运行了这个基准点几个小时来尝试不同的组合情况（我在预取距离为 32 时开始，逐步增加距离直到距离达到 1024）。在此系统上，我写的基于 928 字节而不是 128 字节的代码执行地更快。并且，几乎在循环结束处预取是最快的（在 `do_compare` 标号之前，`PREFETCHNTA` 指令大约 8 条 `line`）。

THE END.

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 汇编优化.rar |
