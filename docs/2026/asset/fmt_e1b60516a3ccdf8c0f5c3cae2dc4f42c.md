
# [原创]第一章:1.6、switch-case识别技巧提高

通过上一节我们一起学习了switch-case分支最基本的两种模式，但是很显然这并不是全部，但是即便是复杂，一个switch-case分支又能复杂到哪里去呢？期望我们的读者都能这种想法！复杂switch-case分支仅仅是应用了一点数据结构方面的知识而已……

在撰写本章之前，笔者曾经多次演练怎样用一篇文章来教会初学者例如平衡二叉树这种数据结构，但是最后的结果都证明这是一个不可能完成的任务。因此，本小节在涉及到相关数据结构方便的知识时并不会为各位介绍太多，相关的细节内容还要自己参考其他相关资料慢慢摸索。

## 1.6.1、switch-case分支结构与稀疏矩阵

我们前面为各位读者分别介绍了转成if-esle与利用跳转表两种优化模式，但是在最后我隐含着提出了一个问题，既如果我们的switch-case分支两个数之差大于50甚至更多的时候，那么我们此时是否仍需要利用跳转表来解决问题呢？很显然我们不能这样做，假如我们遇到如下这段代码：

```c
int _tmain(int argc, _TCHAR* argv[])
{
    switch (argc)
    {
    case 0:
        printf("argc=0",argc);
        break;
    case 1:
        printf("argc=%d",argc);
        break;
    case 6:
        printf("argc=%d",argc);
        break;
    case 7:
        printf("argc=%d",argc);
        break;
    case 199:                      // 注意这里！
        printf("argc=%d",argc);
        break;
    default:
        printf("nNum=%d,error!",argc);
    }
    return 0;
}
```

我们通过上面这个稍显极端的例子可以发现，如果此时编译器仍以跳转表的方式来优化的话，那么会出现什么情况？这会使得编译出来的代码具有多达788字节的冗余数据，至少会使其体积变为用Debug模式生成代码体积的2.7倍以上！

通过与编译器打这么长市间的交道，我们猜也能猜得出编译器肯定不会使用这么笨的方法，由此便引出的“稀疏矩阵”。

“稀疏矩阵”这名字起的很好，正可谓阅名知意，通过名字我们就可以猜到这应该是一个用来表达稀疏数据的矩阵，正好可以用于我们刚刚所面对的这种情况。

那么我们的switch-case分支结构生么时候才会用到稀疏矩阵，而稀疏矩阵又是怎么回事呢？

下面就由笔者一一为各位解答……

### （1、）什么时候应用稀疏矩阵

由于每个编译器所使用的策略不一样，因此其“体积-效率比”权值的设定也不尽相同，笔者在这里只能告诉大家，如果使用跳转表所生成代码的体积大于使用稀疏矩阵的体积，那么一般情况下编译器就会选择使用稀疏矩阵来优化此段代码。

### （2、）什么是稀疏矩阵

单从数学上讲，假设我们有一个`m`行乘以`n`列的二维阵列（既二维数组），那么如果此矩阵中非零值数量`N`小于等于`m*n`的话，那么我们就将这个矩阵称之为稀疏矩阵。

当然稀疏矩阵的具体定义与其它特点还有很多，这里我们不再一一讨论，现在我们着重讲解一下编译器优化时所使用的稀疏矩阵是个什么情况。

其实当我们深入的接触这种优化方式之后可以发现，编译器所用的优化方案仅仅是思路上借鉴了稀疏矩阵，但是其实际使用中并不符合稀疏矩阵的相关定义，因此笔者认为将其称之为“间接表”更为合适一些，现在就让我们共同看一看switch-case分支结构的间接表是怎么回事。

在VC系列编译器中，其针对switch-case分支结构的间接表都是用一字节表示的，因此其最小索引值与最大索引值之差不得大于256，否则此优化方法便不再适用。

其次，这个拥有256个`byte`型元素的数组（间接表）需要与跳转表相呼应最终才能保证程序流程执行到正确的地方上去。下面我就带领各位读者深入的了解一下间接表是怎样被体现出来的。

鉴于以后知识的复杂性，从现在开始，我们将不再使用OllyDbg，因此也请各位读者跟随本文一起转变到IDA这个最专业的逆向工作平台上去（有关于IDA的使用请参考相关文章）。

我们下面以一个例子来证明，以前面那个源码为蓝本，看看IDA生成的反汇编代码是不是更已读易懂：

```asm
.text:00401000 ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:00401000 _main proc near                         ; CODE XREF: __tmainCRTStartup+10Ap
.text:00401000
.text:00401000 argc= dword ptr  4
.text:00401000 argv= dword ptr  8
.text:00401000 envp= dword ptr  0Ch
.text:00401000
.text:00401000     mov eax, [esp+argc]
.text:00401004     cmp eax, 0C7h                       ; switch 200 cases
.text:00401009     ja  short loc_40107B                ; default
.text:00401009                                         ; jumptable 00401012 cases 2-5,8-198
.text:0040100B     movzx ecx, ds:byte_4010A8[eax]
.text:00401012     jmp ds:off_401090[ecx*4]            ; switch jump
.text:00401019
.text:00401019 $LN6:                                   ; DATA XREF: _main:off_401090o
.text:00401019     push 0                              ; jumptable 00401012 case 0
.text:0040101B     push offset Format                  ; "argc=0"
.text:00401020     call ds:__imp__printf
.text:00401026     add esp, 8
.text:00401029     xor eax, eax
.text:0040102B     retn
.text:0040102C ; ---------------------------------------------------------------------------
.text:0040102C
.text:0040102C $LN5:                                   ; CODE XREF: _main+12j
.text:0040102C                                         ; DATA XREF: _main:off_401090o
.text:0040102C     push 1                              ; jumptable 00401012 case 1
.text:0040102E     push offset aArgcD                  ; "argc=%d"
.text:00401033     call ds:__imp__printf
.text:00401039     add esp, 8
.text:0040103C     xor eax, eax
.text:0040103E     retn
.text:0040103F ; ---------------------------------------------------------------------------
.text:0040103F
.text:0040103F $LN4:                                   ; CODE XREF: _main+12j
.text:0040103F                                         ; DATA XREF: _main:off_401090o
.text:0040103F     push 6                              ; jumptable 00401012 case 6
.text:00401041     push offset aArgcD                  ; "argc=%d"
.text:00401046     call ds:__imp__printf
.text:0040104C     add esp, 8
.text:0040104F     xor eax, eax
.text:00401051     retn
.text:00401052 ; ---------------------------------------------------------------------------
.text:00401052
.text:00401052 $LN3:                                   ; CODE XREF: _main+12j
.text:00401052                                         ; DATA XREF: _main:off_401090o
.text:00401052     push 7                              ; jumptable 00401012 case 7
.text:00401054     push offset aArgcD                  ; "argc=%d"
.text:00401059     call ds:__imp__printf
.text:0040105F     add esp, 8
.text:00401062     xor eax, eax
.text:00401064     retn
.text:00401065 ; ---------------------------------------------------------------------------
.text:00401065
.text:00401065 $LN2:                                   ; CODE XREF: _main+12j
.text:00401065                                         ; DATA XREF: _main:off_401090o
.text:00401065     push 0C7h                           ; jumptable 00401012 case 199
.text:0040106A     push offset aArgcD                  ; "argc=%d"
.text:0040106F     call ds:__imp__printf
.text:00401075     add esp, 8
.text:00401078     xor eax, eax
.text:0040107A     retn
.text:0040107B ; ---------------------------------------------------------------------------
.text:0040107B
.text:0040107B loc_40107B:                             ; CODE XREF: _main+9j
.text:0040107B                                         ; _main+12j
.text:0040107B                                         ; DATA XREF: ...
.text:0040107B     push eax                            ; default
.text:0040107B                                         ; jumptable 00401012 cases 2-5,8-198
.text:0040107C     push offset aNnumDError             ; "nNum=%d,error!"
.text:00401081     call ds:__imp__printf
.text:00401087     add esp, 8
.text:0040108A     xor eax, eax
.text:0040108C     retn
.text:0040108C ; ---------------------------------------------------------------------------
```

我想通过上面的反汇编代码已经足以能表明IDA的强大之处了，很明显它已经自动识别出了这就是一个switch-case语句，并为我们生成了清晰明了的注释，看看这是多么惬意呀！

但是笔者在这里要提醒各位读者注意，IDA也并不是每次都能灵验的（在实战时大多数情况都是如此），因此在学习逆向时一定要注意学会忽略IDA的注释，否则总有后悔的那一天的。

通过上面的代码我们可以分析出主要是以下两句代码在控制其流程：

```asm
.text:0040100B     movzx ecx, ds:byte_4010A8[eax]
.text:00401012     jmp ds:off_401090[ecx*4]            ; switch jump
```

为了更好的理解第一句汇编指令的意思，我们需要看看`4010A8`处保存了些什么信息：

```asm
.text:004010A8 byte_4010A8     db      0               ; DATA XREF: _wmain+Br
.text:004010A8                                         ; indirect table for switch statement
.text:004010A9                 db    1
.text:004010AA                 db    5
…………
.text:004010AD                 db    5
.text:004010AE                 db    2
.text:004010AF                 db    3
.text:004010B0                 db    5
…………
.text:0040116E                 db    5
.text:0040116F                 db    4
```

由于我们的switch-case分支结构拥有6个分支，因此间接表里保存的内容都是在0-5之间的，然后便根据此索引来确定调转到第几个分支上去。下面我们来人工模拟一下，假如此时switch-case分支结构接收到的判断变量为166，那么首先会通过执行`mov eax, [esp+argc]`将值传递给`eax`，进行简单的对比检查之后，已确定其值未超过switch-case分支的最大值，然后就通过执行`movzx ecx, ds:byte_4010A8[eax]`指令，以`eax`为索引到地址为`4010A8h`的间接表中寻取相对应的值为跳转表索引，并将此索引保存在`ecx`里，最后在以`ecx`为索引执行`jmp ds:off_401090[ecx*4]`指令跳转到正确定分支上去，整体流程如下图：

```text
argc = 166 (假设传入的数值为166)

     ┃
     ┃
     ▽
mov eax, [esp+argc]
    eax = 166
     ┃
     ┃
     ┃
     ▽                               byte_4010A8
movzx ecx, ds:byte_4010A8[eax]           *---*
    ecx = 5                           000| 0 |
     ┃                               001| 1 |
     ┃                               002| 2 |
     ┃                               ...| . |
     ┃                               165| 5 |
     ┃                        eax--> 166| 5 |
     ┃                               ...| . |
```
