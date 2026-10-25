
`sub ecx, edx`

```
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

每个循环都不再依赖于其他的循环了，在现在的CPU指令里不能这样做，原因只是没有相应的CPU指令。这段代码在P-II上执行的话，我们拥有6个ALU单元和1个负载缓冲（`load microop`?），这使得8 bytes只需要3时钟就可以比较完。在一个AMD处理器上，有7个risc指令单元（`riscops`??? 不确定!），在Athlon上它也可以3时钟处理8字节，在K6上则是4时钟8字节的速度。

Athlon上的性能测试机制显示（忽略掉分支误预测的影响），对于一个超长字符串而言，这个每次可以处理64 bits的版本对比32版本的代码提高了55%的速度（比WATCOM clib里的`strlen`还快22%）。物理如何，对于超短字符串（少于8字节），64bits版本的对比32bit版本的就要慢上3倍，但和WATCOM的clib中的`strlen`在相同短字符串的情况下还是一样快的。但需要关心一下的是，如果你的字符串够短，分支误预测（我的测试里没计算它的影响）将相对其他情况里的占用多很多的执行时间。当然任何情况下，如果你想用这些`strlen`了，你应该做一下具体情况里的性能测试，以确定哪个适合你。

## Update 补充

Norbert Juffa给我写了消息，教给我一个古老的技巧，比起我的还可以有极大的改善（爽罢，rock rock起来，我们继续看 `/:^)` ）：

> ---------quote------------------
>
> After a few minutes of research it turns out that the original null byte detection by Alan Mycroft is superior to the formula given on your website as it has shorter dependency chains. On a PIII, strcpy() performance jumped almost 20% from substituting one for the other. Mycroft's macro is
>
> 结果几分钟的研究，发现最原始的`null`字节检测（Alan Mycroft写的）比你的网站上的公式要强大很多，它有短得多的运算步骤。在P-III上，`strcpy()`的性能提高了接近20%。Mycroft的宏是：
>
> ```c
> #define has_nullbyte_(x) ((x - 0x01010101) & ~x & 0x80808080)
> ```
>
> I rendered this as:
>
> 我把它翻译成汇编了：
>
> ```asm
> mov edx, eax
> not eax
> sub edx, 0x01010101
> and eax, 0x80808080
> and eax, edx
> ; cycle 1
> ; cycle 2
> ; cycle 3
> ```
>
> BTW, one of Mycroft's original posts about this macro can be found at Deja. The post is dated 1987-04-27!
>
> 另外，Mycroft的一个最早的关于这个宏的帖子可以在deja上找到，发表日期是1987-04-27!
>
> -----------------------------------

我去做了一点搜索工作，然后非常的没错，你可以在google groups上找到一个很老的帖子，其内容如下（已做脱敏处理）：

```text
7a2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6J5L8%4g2H3M7#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3N6J5L8%4g2H3M7#2)9K6c8X3S2D9i4K6y4p5k6h3&6Q4x3U0k6D9M7W2)9K6c8q4)9J5y4X3W2W2i4K6y4p5g2g2c8r3i4K6u0V1z5q4)9J5y4X3!0W2i4K6y4p5g2g2c8r3i4K6u0V1z5q4)9J5y4Y4y4S2k6X3g2Q4x3@1c8G2k6X3k6Q4x3U0k6X3M7X3q4E0k6g2)9K6c8s2u0A6k6$3S2@1i4K6t1$3M7X3&6#2L8g2)9K6c8o6b7I4i4K6t1$3N6r3S2D9i4K6y4p5x3e0V1&6z5e0f1I4y4K6V1@1x3g2)9J5b7K6p5&6z5e0V1#2x3e0M7%4z5e0m8Q4x3V1x3I4z5e0V1&6y4e0p5%4z5o6j5K6i4K6u0o6x3e0V1&6z5e0f1I4y4o6b7@1y4g2)9J5b7K6p5&6z5e0V1#2x3e0t1@1x3e0q4Q4x3V1x3I4z5e0V1&6y4e0p5I4z5e0R3^5i4K6u0o6x3e0V1&6z5e0f1I4x3o6j5J5y4q4)9J5b7K6p5&6z5e0V1#2x3e0l9I4x3o6c8Q4x3V1x3I4z5e0V1&6y4e0l9&6x3o6t1K6i4K6u0o6x3e0V1&6z5e0f1I4y4e0M7H3y4g2)9J5b7K6p5&6z5e0V1#2x3e0f1I4x3e0u0Q4x3V1x3H3i4K6t1$3M7$3g2W2K9$3#2Q4x3@1b7@1y4U0l9#2i4K6t1#2y4o6m8#2N6r3y4K6M7X3W2Q4x3X3g2g2g2f1y4b7i4K6t1K6L8r3W2F1K9K6b7K6i4K6t1&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1#2i4K6R3@1i4@1u0r3i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1$3i4K6W2o6i4K6R3H3i4@1f1$3i4K6V1$3i4@1t1H3M7%4c8J5L8r3g2F1
```

其对应的`C`语言实现：

```c
#define hasNulByte(x) ((x - 0x01010101) & ~x & 0x80808080)
#define SW (sizeof (int) / sizeof (char))

int xstrlen (const char *s) {
    const char *p;
    int d;

    p = s - 1;
    do {
        p++;
        if ((((int) p) & (SW - 1)) == 0) {
            do {
                d  = *((int *) p);
                p += SW;
            } while (!hasNulByte (d));
            p -= SW;
        }
    } while (*p != 0);

    return p - s;
}
```

和其他的`strlen`方法比起来，在Athlon上这个完全是求32字节字符串长度最快的方法了，字节更长时MMX的方法有优势。这个函数的性能比起绝大多数库里的`strlen`，都是占有绝对优势的。

## 6. Sprite的数据复制

一个程序员（游戏程序员）常见的需求是，渲染sprite图像时怎么写出来一个高性能的sprite图像块传输函数（`blitter`，BLIT BLock Image TransfER）。最近一个在`rec.games.programmer`的讨论里，提问者的需要是把一个数组的像素一直复制到目标位置直到碰到像素值为0的情况。一开始时候的代码看起来类似于：

```c
char * scrptr;
char * destptr;

/* ... */

/* copy scanline except where transparent. */
for(x=x0;x<xl;x++) {
    if( srcptr[x]!=0 ) destptr[x]=srcptr[x];
}
```

当然，一开始的这个代码有好几个性能问题在它里面：

1. 它有一个在实际中总要判断出错的分支预测；
2. 读取内存的方式是每次读取一个内存，这样是对内存带宽极大的浪费，同样也增加了循环的次数。

也能换个角度看，如果说要写入目标内存是`Write Combine`（合并写入）和`write mask`类型的（写入mask，作用于rgb和xyzw，参考`e11K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3u0T1x3U0p5&6z5o6M7H3i4K6t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6g2Q4x3U0W2Q4x3X3g2S2M7%4m8^5i4K6t1&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1&6i4K6R3J5i4@1p5K6i4@1f1@1i4@1u0n7i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1$3i4K6R3&6i4@1p5%4i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4K6V1K6i4@1t1$3i4@1f1&6i4@1p5J5i4K6R3^5i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1#2i4K6R3#2i4@1p5#2i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1^5i4K6R3H3i4K6S2o6i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1#2i4K6R3^5i4K6R3$3i4@1f1$3i4K6V1@1i4@1q4r3i4@1f1^5i4@1q4r3i4@1q4r3i4@1f1&6i4@1p5J5i4K6R3@1i4@1f1$3i4@1t1#2i4K6S2n7i4@1f1#2i4K6V1J5i4K6S2o6i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1#2i4@1q4p5i4K6V1^5i4@1f1^5i4@1q4r3i4@1u0n7i4@1f1#2i4K6S2r3i4K6V1$3i4@1f1@1i4@1t1^5i4K6S2m8i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1$3i4@1p5H3i4@1t1%4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1@1i4@1u0n7i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1%4i4@1p5%4i4K6S2p5i4@1f1#2i4@1p5#2i4@1u0p5i4@1f1@1i4@1u0m8i4K6S2n7i4K6t1^5i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1$3i4@1q4p5i4@1p5#2i4K6t1&6i4@1f1K6i4K6R3H3i4K6R3J5i4K6t1^5i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1%4i4@1p5%4i4K6S2p5i4@1f1$3i4K6R3K6i4K6R3#2i4@1f1#2i4K6R3$3i4@1t1#2i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6V1@1i4K6W2r3i4@1f1%4i4K6W2m8i4K6R3@1i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1u0p5i4K6V1K6i4@1f1^5i4@1p5$3i4K6R3I4i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1#2i4K6R3#2i4@1p5#2i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1#2i4@1q4p5i4K6V1^5i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1$3i4K6V1^5i4@1u0q4i4@1f1#2i4@1q4p5i4K6V1^5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6R3^5i4K6V1$3i4@1f1^5i4K6R3H3i4K6R3#2b7#2m8g2i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1^5i4@1p5J5i4@1q4n7i4@1f1^5i4@1t1$3i4K6R3#2i4@1f1&6i4@1p5J5i4K6V1I4i4@1f1@1i4@1u0m8i4K6R3$3i4@1f1%4i4K6W2m8i4K6R3@1i4K6t1&6`）。

无论如何，只要说读取到的内存不是性能的瓶颈（系统内存），那每次读4字节，然后用上`mask`的是否为0判断方式就不是一个坏方案了，这样提供了一个你不需要写跳转的代码方案了。（不优化跳转的话，也就没什么特别大的优化方向了）。回复者里Russ Williams直接拿出来一个使用了`carry flag`（`CF` eflag）方案了（在奔腾CPU上有优化效果）：

```asm
; by Russ Williams
pushad
mov esi,[pbSrc]
mov ecx,[w]
shr ecx,2
mov edi,[pbDest]
sub edi,esi

looptop:
xor ebx,ebx
mov eax,[esi]
add esi,4
mov edx,eax
ror edx,16

add al,255              ; cf =  1 if al!=0 or 0 if al==0
sbb bl,0                ; bl = -1 if al!=0 or 0 if al==0
add ah,255
sbb bh,0
mov eax,edx
shl ebx,16
add al,255
sbb bl,0
add ah,255
```
