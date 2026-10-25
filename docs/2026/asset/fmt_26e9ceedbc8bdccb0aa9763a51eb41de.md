
# [翻译]在IDA中使用Python Z3库来简化函数中的算术运算

我在很早的时候就想接触基于SMT的程序分析，近日我开始了解这个方向。这个方面有许多的框架，文章和教程，我会一点一点深入了解。

尽管我仍然在学习这部分内容，但无论如何我并不觉得这很先进或者很高大上，相反，它是非常基础的知识，Z3（来自Microsoft Research的一个定理证明的库）对于大部分初学者来说是十分简易的。到目前为止，我所知道的除了来自Eric的[Z3Py入门教材](https://bbs.kanxue.com/elink@8cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2M7X3W2U0M7r3!0F1P5g2)9J5c8Y4Z5K6M7s2W2Q4x3X3c8@1N6i4c8G2M7X3W2S2L8l9`.`.)之外，还阅读[Dennis Yurichev](https://bbs.kanxue.com/elink@4caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8b7`.`.)的文章"[Quick introduction into SAT/SMT solvers and symbolic execution](https://bbs.kanxue.com/elink@536K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8Y4N6J5K9i4c8A6L8X3N6K6i4K6u0r3f1@1q4f1i4K6g2X3f1@1#2f1i4K6g2X3k6s2u0S2k6Y4c8Q4x3X3c8q4e0W2)9J5k6i4m8V1k6R3`.`.)"。

在上周的发表的博客中，我演示了在不运行的情况下如何编写基本模拟器来评估函数的返回值。在本文中，我将展示如何将数千个算术运算从晦涩的x86汇编代码转换为月简易的Z3表达式，本文主要分为三个部分：

- 简介
- Z3库的快速上手
- 将汇编转成Z3表达式

## 简介

本文的测试程序和上周的测试程序很类似，由主函数调用的12个挑战函数。每个挑战函数都是随机生成的，因此它包含随`eax`、`ebx`、`ecx`、`edx`寄存器（以及立即数）一起工作的`add`、`sub`、`dec`、`inc`指令的随机序列。示例程序在附件中可以下载。下面是第一个挑战函数的汇编代码：

```c
//-------------------------------------------------------------------------

uint32_t challenge_1(uint32_t c1, uint32_t c2, uint32_t c3, uint32_t c4) // 1953 operation(s)

{

    uint32_t result;

    __asm

    {

        pushad

        mov eax, [c1]

        mov edx, [c2]

        mov ecx, [c3]

        mov ebx, [c4]

        sub eax, ebx

        inc ebx

        sub ebx, ecx

        sub ecx, 0x852e4867

        add ebx, ebx

        inc ecx

        add eax, edx

        add ecx, ebx

        sub ecx, ebx

        inc ecx

        sub ebx, edx

        add eax, 0x7a20f9e6

        add ebx, 0xaa5a1584

        add edx, edx

        sub ebx, 0x1ca0a567

        sub eax, 0xf94f7d8c

        inc ecx

        inc eax

        add edx, eax

        sub ebx, edx

        inc ebx

        sub edx, 0xd68a9fa7

        inc ebx

        inc eax

        inc eax

        .

        .

        ...1000+ instructions later...

        .

        sub ebx, edx

        inc eax

        sub ebx, edx

        sub ecx, eax

        add eax, ebx

        add ecx, 0xd2cb013d

        add ecx, 0xda9d6a2e

        add edx, eax

        sub edx, 0x25ebd85d

        add ebx, ebx

        add ebx, 0x936e2259

        inc eax

        add eax, ecx

        add ebx, 0xc0c1aa

        inc ebx

        add edx, 0x921ee6d5

        add edx, edx

        add ecx, eax

        add ecx, eax

        inc ebx

        sub ebx, edx

        add ebx, eax

        inc ebx

        sub eax, 0xd9d2f9c2

        add edx, eax

        inc ecx

        add ecx, 0xad2e6bb0

        add ecx, eax

        sub ecx, ebx

        add ebx, eax

        sub ecx, 0xe2786d0c

        add eax, ebx

        add eax, ecx

        add eax, edx

        mov dword ptr [result], eax

        popad

    }

    return result;

}
```

上面函数的汇编代码看起来很相似，分析不出什么东西，所以这里就不去分析了，直接使用F5大法查看伪代码：
