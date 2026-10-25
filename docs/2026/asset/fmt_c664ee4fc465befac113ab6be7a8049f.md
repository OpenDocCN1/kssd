
- 在**其函数栈中预留空间存储所有可能的匿名寄存器参数**(否则由于不确定传入了多少个寄存器参数，AAPCS64标准中的所有传参寄存器在callee中均不可直接使用)

- 在**运行时通常需要通过已知的某个命名参数来确定caller到底传入了多少个参数**(caller在调用可变参数函数时通常需要显式或隐式的告知callee自己传入了多少个参数), 如:

```c
/* 常见的处理可变参的宏定义如下，后续代码中直接使用内联函数

	#define va_start(v,l)	__builtin_va_start(v,l)

	#define va_end(v)	__builtin_va_end(v)

	#define va_arg(v,l)	__builtin_va_arg(v,l)

	#define va_copy(d,s)	__builtin_va_copy(d,s)

*/



//int printf (const char * format, ... );

#define weak __attribute__((weak))

weak int func1(int x, ...)

{

        int i = x;

    

        __builtin_va_list vl;

        __builtin_va_start (vl, x);

        while(i > 0)

        {

          printf("%d\n", va_arg(vl, int)); // callee根据caller传入的参数个数逐个获取参数

          i --;

        }

        __builtin_va_end (vl);



        return 0;

}



int main(void)

{

   register void * sp asm("sp");

   printf("main1: sp:%p, fp:%p\n", sp, __builtin_frame_address(0)); //printf通过参数R0隐式确定参数个数

   

   func1(3, 1, 2, 3);	//一般情况下调用可变参数需通过命名参数显式告知参数个数

}  
```

## 输出

```text
tangyuan@ubuntu:~/tests/gcc/aarch64_stack/caller_callee_parm_test$ ./main 

main1: sp:0x40007ff860, fp:0x40007ff860

1

2

3
```

## 2. 可变参函数的参数

可变参函数的参数分为四类, 即**寄存器命名参数, 寄存器匿名参数, 栈命名参数, 栈匿名参数**, 举例如下:

```c
#define REP7(X) X,X,X,X,X,X,X

#define REP8(X) X,X,X,X,X,X,X,X

#define weak __attribute__((weak))

/*

   func1的:

   * 第1个参数(x0,必须指定)为命名寄存器参数

   * 第2-8个参数(若有)为匿名寄存器参数

   * 第9-n个参数(若有)为匿名栈参数

*/

weak int func1(int x0, ...) { ... }

/*

   func2的:

   * 前8个参数(x0-x7,必须指定)为命名寄存器参数

   * 第9个参数(x8,必须指定)为命名栈参数

   * 第10-n个参数(若有)为匿名栈参数

*/

weak int func2(int x0, int x1, int x2, int x3, int x4, int x5, int x6, int x7, int x8, ...) { ... }
```

可变参函数的参数传递同样满足AAPCS64:

* 对caller来说, 编译期间即可确定其向可变参callee传入了多少个参数，故其直接按照AAPCS64标准布局参数(寄存器参数直接赋值给硬件寄存器，栈参数保存到outgoing栈对应位置).
* 对callee来说, 编译期间只能确定有多少个命名参数:
  - **命名寄存器参数**直接通过硬件寄存器获取
  - **命名栈参数**通过incoming栈获取
* 对callee来说, 匿名参数的个数需直接或间接通过某个命名参数得知:
  - **匿名栈参数**同样**继续**通过incoming区域获取
  - **匿名寄存器参数**则是从**匿名寄存器存储区域**(callee-allocated save area for register varargs)获取

举例如下:

```c
#include <stdio.h>

#include <stdarg.h>



struct __va_list			/* 自定义的__va_list结构体 */

{

        void *__stack;

        void *__gr_top;

        void *__vr_top;

        int   __gr_offs;

        int   __vr_offs;

};



int func(int p0, int p1, int p2, ...)

{

        register unsigned long sp asm("sp");

        unsigned long  var0 = (unsigned long)&var0 - sp;	/* 第一个局部变量到callee入口sp的offset */

        unsigned long  var1 = (unsigned long)&var1 - sp;	/* 第二个局部变量到callee入口sp的offset */

        __builtin_va_list vl;



        p0 = (unsigned long)&p0 - sp;				/* 第一个命名寄存器参数的存储位置到callee入口sp的offset

                                                                 * (解引用会导致callee在局部变量中为命名寄存器参数分配存储空间,

                                                                 *  这里顺便解释局部变量的内存分配)

                                                                 */

        p1 = (unsigned long)&p1 - sp;				/* 第二个命名寄存器参数的存储位置到callee入口sp的offset */

        printf("[+] sp:%p\n", sp);

        printf("[+] var0 in addr %p (offset %p)\n", &var0, var0);

        printf("[+] var1 in addr %p (offset %p)\n", &var1, var1);

        printf("[+] vl in addr %p, size:%d\n", &vl, sizeof(vl));	/* 局部变量vl的地址及大小 */



        printf("[+] p0 in addr %p (offset %p)\n", &p0, p0);

        printf("[+] p1 in addr %p (offset %p)\n", &p1, p1);



        var0 = 0;

        __builtin_va_start (vl, p2);	/* p2记录此函数匿名寄存器个数 */

        while(var0 < p2) {

                var0 ++;

                if( var0 < 6) {

                        /* 输出剩余匿名通用寄存器参数的存储首地址和offset */

                        var1 = ((struct __va_list *)&vl)->__gr_top + ((struct __va_list *)&vl)->__gr_offs;

                } else {

                        /* 输出下一个匿名栈参数的存储位置 */

                        var1 = ((struct __va_list *)&vl)->__stack;

                }

                printf("[+] p%d in addr %p (offset %p) with value %d\n", var0 + 2, var1, var1 - sp, va_arg(vl, int));

        }

}



int main(void)

{

        func(0, 0, 7, 1, 2, 3, 4, 5, 6, 7);

}
```

输出结果:

```text
##aarch64-linux-gnu-gcc main.c -O2 -o main## 输出结果
tangyuan@ubuntu:~/tests/gcc/aarch64_stack/caller_callee_parm_test$ ./main

[+] sp:0x40007ffdf0

[+] var0 in addr 0x40007ffe38 (offset 0x48)

[+] var1 in addr 0x40007ffe30 (offset 0x40)

[+] vl in addr 0x40007ffe10, size:32

[+] p0 in addr 0x40007ffe0c (offset 0x1c)

[+] p1 in addr 0x40007ffe08 (offset 0x18)

[+] p3 in addr 0x40007ffec8 (offset 0xd8) with value 1

[+] p4 in addr 0x40007ffed0 (offset 0xe0) with value 2

[+] p5 in addr 0x40007ffed8 (offset 0xe8) with value 3

[+] p6 in addr 0x40007ffee0 (offset 0xf0) with value 4

[+] p7 in addr 0x40007ffee8 (offset 0xf8) with value 5

[+] p8 in addr 0x40007ffef0 (offset 0x100) with value 6

[+] p9 in addr 0x40007ffef8 (offset 0x108) with value 7
```

其函数栈分布如图:

![](https://bbs.kanxue.com/upload/attach/202204/490870_ZJ72CK3TQCMTX27.jpg)![]( "点击并拖拽以移动")

由此函数可以看到:

- 匿名寄存器参数(varargs)分为两部分，由高地址=>低地址分别是GPRs/FPRs.

- GPRs/FPRs内部的元素增长方向都是低地址=>高地址(如 `p3` 地址 < `p4` 地址)

- 以GPRs为例, 编译期间会根据当前函数可能出现的匿名寄存器个数确定此区域大小，如此函数中只需在匿名寄存器区域为`[p3-p7]`预留空间即可.

- 匿名栈参数和命名栈参数一样，均按照顺序存储在incoming区域(对于caller没有匿名参数一说，所有栈参数按照顺序存储)

- 若`func`有大于8个整型, 8个浮点型参数，则其函数栈中不再需要为匿名寄存器参数预留空间,如:

```c
## 此函数虽然是可变参数，但不需要分配匿名寄存器存储区域

int func(int p0, int p1, int p2, int p3, int p4, int p5, int p6, int p7, 

        double d0, double d1, double d2, double d3, double d4, double d5, double d6, double d7, 

        ...);
```

- 整个outgoing区域 + varargs区域是可变参函数的参数存储区(这里只介绍栈布局，关于可变参函数`va_xxx`系列函数的分析可参考[5])

# 五、局部变量区

在不开启优化时, 局部变量通常是按照其定义顺序依次分配到局部变量区域的(高地址=>低地址), 但开启优化时局部变量的分配可能经过延迟展开,重排等优化, 此时源码中定义顺序与分配顺序可能不同。

## 1. 定参函数的局部变量区域

局部变量区的尾地址(即第一个局部变量的尾地址):

- 对于定参函数来说，总是此函数入口时的`sp`.

- 对于不定参函数来说，通常有个匿名寄存器区的偏移.

故对于定参函数，通常会看到如下布局:

```c
#include <stdio.h>

int main(void)
{
        int x;
        int y;
        printf("%p, %p\n", &x, &y);
}
```

```text
## aarch64-linux-gnu-gcc main.c -O0 -S -o main.s

# +--------------+ <------ sp		## callee入口时sp

# |  x           |

# +--------------+ <------ sp - 4	## 局部变量 x

# |  y           |

# +--------------+ <------ sp - 8	## 局部变量 y

# |......(align) |

# +--------------+ <------ sp' + 16

# |  x30         |

# +--------------+ <------ sp' + 8

# |  x29         |

# +--------------+ <------ sp'

# |  ......      |

# +--------------+



main:

        stp     x29, x30, [sp, -32]!            ## 函数入口 sp' = sp - 32;          		                                                                                                                                                    

        mov     x29, sp

        add     x1, sp, 24

        add     x0, sp, 28

        mov     x2, x1							## 变量y的地址为 sp' + 24, 即 sp - 8

        mov     x1, x0							## 变量x的 地址为 sp' + 28, 即 sp - 4

        adrp    x0, .LC0

        add     x0, x0, :lo12:.LC0

        bl      printf

        mov     w0, 0

        ldp     x29, x30, [sp], 32

        ret
```

## 2. 局部变量区的存储内容

局部变量区除了用来保存源码中定义的局部变量外，还用来存储`gcc`编译过程中生成的临时变量以及硬件寄存器参数的原始值(若需要), 以**三.2**中代码为例,由其局部变量区可见:

![](https://bbs.kanxue.com/upload/attach/202204/490870_C4SU64FWSCGBERZ.jpg)![]( "点击并拖拽以移动")

1. 局部变量区域的起始地址为变量`var0`的尾地址

2. 源码中的局部变量`var0`/`var1`/`vl`按照高地址=>低地址的顺序依次分布(但并非总是如此，编译优化可能改变分配顺序)

3. 源码中存在对参数`p0`/`p1`/`p2`的解引用, 在局部变量区依次为其分配存储空间(参数的存储位置总是在源码中局部变量之后),实际上:

    - 在开启`-O0`优化时，在局部变量栈中无条件为所有寄存器参数预留空间(如`p0`/`p1`/`p2`)

    - 在开启`-O2`优化时，则默认不为寄存器参数预留空间(除非需要).

# 六、callee-saved registers

callee-saved regs区域用来保存 callee-saved register的, 按照AAPCS64标准 [`r19, r28`] 是callee-saved register，这些寄存器在子函数返回时要保持不变，也就是说如果子函数中使用了这些寄存器，则子函数需要保存并在返回前恢复其原始值。 函数栈中callee-saved register区域就是用来保存这些寄存器原始值的。callee-saved regs区域内部也是由低地址=>高地址增长的, 但区别在于**此区域按照寄存器的编号顺序保存，而不是其在源码中被使用的顺序**,如:
