
[10] [[PATCH] [RFC][PR102768] aarch64: Add compiler support for Shadow Call Stack](https://bbs.kanxue.com/elink@1ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8Y4g2Q4x3X3g2G2M7X3N6Q4x3V1k6H3K9i4m8W2M7X3#2S2K9h3I4Q4x3V1k6Y4j5$3y4Q4x3X3c8H3j5i4c8U0K9r3g2K6i4K6u0r3x3U0l9J5x3g2)9J5k6p5&6G2N6X3g2E0j5X3g2J5i4K6u0r3y4e0R3#2x3e0V1&6i4K6u0W2K9s2c8E0L8l9`.`.)

[11] [Stack unwinding | MaskRay](https://bbs.kanxue.com/elink@47fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5i4y4C8M7X3q4&6i4K6u0W2L8h3g2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9J5x3q4)9J5k6o6p5I4i4K6u0V1x3o6S2Q4x3X3c8K6N6r3q4U0K9#2)9J5k6s2g2F1N6$3W2F1k6r3W2F1k6H3`.`.)

[12] [探索Android平台ARM unwind技术 - 知乎](https://bbs.kanxue.com/elink@5aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3K6y4U0V1I4y4U0p5I4y4W2)9K6c8X3W2$3K9#2)9#2k6Y4y4S2i4K6y4p5x3e0l9J5y4o6x3J5x3s2f1`.)

[13] [linux 栈回溯(x86_64 ) - 知乎](https://bbs.kanxue.com/elink@77aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3H3x3U0M7J5y4U0l9^5x3R3`.`.)

[14] [Unwind 栈回溯详解：libunwind_RToax-CSDN博客_libunwind](https://bbs.kanxue.com/elink@490K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8W2u0G2L8X3N6Q4y4h3k6f1L8$3q4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4x3o6R3@1y4U0f1H3z5b7`.`.)

---

# 附:

1. 关于基于DWARF的栈回溯和IA-64 C++ABI的其他分析可参考[11-14]

2. 以下代码用于测试`libgcc`/`libunwind`中的`_Unwind_Backtrace`实现:

```
#include <stdio.h>

#include <stdlib.h>

#include <unwind.h>

 

unsigned long level = 0;

 

#ifndef CC_IS_CLANG

#define AARCH64_DWARF_V0       64

#define AARCH64_DWARF_NUMBER_V 32

#define DWARF_ALT_FRAME_RETURN_COLUMN   (AARCH64_DWARF_V0 + AARCH64_DWARF_NUMBER_V)

#define DWARF_FRAME_REGISTERS           (DWARF_ALT_FRAME_RETURN_COLUMN + 1)

 

struct dwarf_eh_bases

{

  void *tbase;

  void *dbase;

  void *func;

};

 

struct _Unwind_Context

{

  _Unwind_Word reg[DWARF_FRAME_REGISTERS+1];

  void *cfa;

  void *ra;

  void *lsda;

  struct dwarf_eh_bases bases;

  _Unwind_Word flags;

  _Unwind_Word version;

  _Unwind_Word args_size;

  char by_value[DWARF_FRAME_REGISTERS+1];

};

 

//基于libgcc的栈回溯的回调函数需要解除非IA-64 ABI判断寄存器状态才能决定是否可输出

_Unwind_Reason_Code gcc_callback(struct _Unwind_Context * context, void * args)

{

    unsigned long * plevel = args;

    //在sp(x31)未初始化时直接调用_Unwind_GetGR(context, __builtin_dwarf_sp_column()) 会导致crash

    printf("Unwind Frame(%ld): CFA:%016lx, SP:%016lx, RA:%016lx\n", *plevel, \

        _Unwind_GetCFA(context),

        context->reg[__builtin_dwarf_sp_column()]?_Unwind_GetGR(context, __builtin_dwarf_sp_column()):0,    

        _Unwind_GetIP(context));

    int i;

    for(i = 0; i < 32; i++) {

        if(i%4 == 0) {

            if(i != 0) printf("\n");

        }

            //printf("R%02d(%d):%016lx,", i, context->by_value[i], context->reg[i]);

        if(context->reg[i] && !context->by_value[i])

            printf("R%02d:%016lx,", i, _Unwind_GetGR(context, i));

        else

            printf("R%02d:%016lx,", i, context->reg[i]);

    }

    printf("\n\n");

    (*plevel)++;

 

 

    return _URC_NO_REASON;

}

#endif

 

//此函数完全使用IA-64 ABI中的接口函数,在libunwind中运行正常,但基于libgcc则会crash

_Unwind_Reason_Code std_callback(struct _Unwind_Context * context, void * args)

{

    unsigned long * plevel = args;

    printf("Unwind Frame(%ld): CFA:%016lx, SP:%016lx, RA:%016lx\n", *plevel, \

        _Unwind_GetCFA(context), _Unwind_GetGR(context, __builtin_dwarf_sp_column()), _Unwind_GetIP(context));

    int i;

    for(i = 0; i < 32; i++) {

        if(i%4 == 0) {

            if(i != 0) printf("\n");

        }

        printf("R%02d:%016lx,", i, _Unwind_GetGR(context, i));

    }

    printf("\n\n");

    (*plevel)++;

 

 

    return _URC_NO_REASON;

}

 

_Unwind_Reason_Code (*pcallback) (struct _Unwind_Context * context, void * args);

 

int main(void)

{

#ifndef CC_IS_CLANG

    pcallback = gcc_callback;

#else

    pcallback = std_callback;

#endif

    _Unwind_Backtrace(pcallback, &level);

    return 0;

}
```

编译命令:

```
.PHONY: all

CC = /mnt/disk0/disk0/toolchains/llvm/clang+llvm-12.0.1-x86_64-linux-gnu-ubuntu-16.04/bin/clang

AARCH64_GCC_TOOLCHAIN = /mnt/disk0/disk0/gcc_source_code/mk_cross_compiler/cross-gcc/

TARGET = aarch64-linux-gnu

INCLUDE := -L/mnt/disk0/disk0/toolchains/llvm/clang+llvm-12.0.1-aarch64-linux-gnu/lib/ -L/mnt/disk0/disk0/toolchains/llvm/clang+llvm-12.0.1-aarch64-linux-gnu/lib/clang/12.0.1/lib/linux

 

CLANG_CC = $(CC) --target=$(TARGET) --gcc-toolchain=$(AARCH64_GCC_TOOLCHAIN) $(INCLUDE) -DCC_IS_CLANG

 

all:

        ## gcc 基于 libgcc的编译

        aarch64-linux-gnu-gcc -static main.c -O0 -o main

        ## clang基于libunwind的编译

        $(CLANG_CC) -static main.c -o main_clang  --rtlib=compiler-rt -lunwind -stdlib=libc++ -stdlib=libstdc++  -fuse-ld=lld -lsupc++
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-2-19 15:33

被ashimida编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)
