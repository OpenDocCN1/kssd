
# [原创] AArch64中va_list/va_start/va_arg/...的实现

## 一、背景[3]

### 1. 可变参函数

可变参函数指的是一个可以接受可变个参数的函数, 调用此函数时**只有caller知道为此函数传入了多少个参数, 可变参函数callee只知道caller最少传入了多少个参数**:

- callee中可以确定的参数称为命名参数(Named arguments)
- callee中不确定的参数称为匿名参数(Anonymous arguments)

对于可变参函数(callee), 其需要:

- 在**其函数栈中预留空间存储所有可能的匿名寄存器参数**(否则由于不确定传入了多少个寄存器参数，不保存则AAPCS64标准中所有传参寄存器默认在callee中不能用)
- 在**运行时通常需要通过已知的某个命名参数来确定caller到底传入了多少个参数**(caller在调用可变参数函数时也需要显式或隐式的告知callee自己传入了多少个参数), 如:

```c
/* 常见的处理可变参的宏定义如下，后续代码中直接使用内联函数

	#define va_start(v,l)	__builtin_va_start(v,l)

	#define va_end(v)		__builtin_va_end(v)

	#define va_arg(v,l)		__builtin_va_arg(v,l)

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

   

   func1(3, 1, 2, 3);    //一般情况下调用可变参数需通过命名参数显式告知参数个数

}  
```

### 输出

```
tangyuan@ubuntu:~/tests/gcc/aarch64_stack/caller_callee_parm_test$ ./main 
main1: sp:0x40007ff860, fp:0x40007ff860
1
2
3
```

### 2. 可变参函数的参数

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

- 对caller来说, 编译期间即可确定其向可变参callee传入了多少个参数，故其直接按照AAPCS64标准布局参数(寄存器参数直接赋值给硬件寄存器，栈参数保存到outgoing栈对应位置).
- 对callee来说, 编译期间只能确定有多少个命名参数:
  - **命名寄存器参数**直接通过硬件寄存器获取
  - **命名栈参数**通过incoming栈获取
- 对callee来说, 匿名参数的个数通常需直接或间接通过某个命名参数得知:
  - **匿名栈参数**同样**继续**通过incoming区域获取
  - **匿名寄存器参数**则是从**匿名寄存器存储区域**(callee-allocated save area for register varargs)获取

## 二、va_xxx系列函数

在可变参函数中通常会使用va_xxx系列函数来获取匿名参数，va_xxx函数在gcc中会对应到其对应的一个builtin函数上:

```c
//stdarg.h

#define va_start(v,l)	__builtin_va_start(v,l)

#define va_end(v)	__builtin_va_end(v)

#define va_arg(v,l)	__builtin_va_arg(v,l)

#define va_copy(d,s)	__builtin_va_copy(d,s)
```

gcc中定义的此系列内联函数包括:

```
__builtin_va_list

__builtin_va_start

__builtin_va_end

__builtin_va_copy

__builtin_next_arg

__builtin_saveregs

__builtin_va_arg
```

这里只介绍其中部分，其余类似的函数见源码.

## 三、__builtin_va_list

`__builtin_va_list`是gcc内置的一个结构体类型, 在gcc内部定义如下:

```c
/* 关于函数栈的构成可参考[3], AAPCS64中可变参函数的说明可参考[1] */

typedef struct {

       void *__stack;    /* __stack 记录下一个匿名栈参数的存储位置, 随着va_arg的调用可能变化 */

       void *__gr_top;   /* __gr_top 记录最后一个匿名通用寄存器参数的尾地址, 其不随va_arg调用变化 */

       void *__vr_top;   /* __vr_top 记录最后一个匿名浮点寄存器参数的尾地址, 其不随va_arg调用变化 */

       int   __gr_offs;  /* __gr_offs 记录下一个匿名通用寄存器参数到__gr_top的偏移(负数),随着va_arg的调用可能变化 */

       int   __vr_offs;  /* __vr_offs 记录下一个匿名浮点寄存器参数到__vr_top的偏移(负数),随着va_arg的调用可能变化 */

} __builtin_va_list;
```

各字段在函数栈中的位置如下:

![](https://bbs.kanxue.com/upload/attach/202204/490870_Q8CRP7W3Y3F8RFT.jpg)

一个经过初始化后的`__builtin_va_list`结构体(`vl`)中:

- 可以通过`vl->__stack`获取下一匿名栈参数, [`vl->__stack`, ......] 为匿名栈参数区
- 可以通过`vl->__gr_top + vl->__gr_offs` 获取下一个匿名通用寄存器参数, [`vl->__gr_top + vl->__gr_offs` , `vl->__gr_top`]为匿名通用寄存器参数区
- 可以通过`vl->__vr_top + vl->__vr_offs` 获取下一个匿名浮点寄存器参数, [`vl->__vr_top + vl->__vr_offs` , `vl->__vr_top`]为匿名浮点寄存器参数区

## 四、__builtin_va_start(vl, x)

`__builtin_va_start(vl, x)`的作用是初始化`vl`结构体，在aarch64中这里的`x`没有实际的作用，但调用此函数时`x`必须是此函数的最后一个命名参数，否则会编译报错。`__builtin_va_list vl;` 就是一个简单的结构体局部变量定义，故在使用`vl`之前必须调用`__builtin_va_start`函数对其初始化, `__builtin_va_start`的作用为:

- 设置`vl->__stack` 指向此函数第一个匿名栈参数可能出现的位置
- 设置`vl->__gr_top` 指向最后一个匿名通用寄存器参数尾地址
- 设置`vl->__gr_offs` 为`vl->__gr_top` 到 第一个匿名通用寄存器参数之间的偏移(负数)
- 设置`vl->__vr_top`指向最后一个匿名浮点寄存器参数尾地址
- 设置`vl->__vr_offs` 为`vl->__vr_top` 到 第一个匿名浮点寄存器参数之间的偏移(负数)

需要注意的是, 编译器按照AAPCS64布局函数栈，**一个函数的第一个匿名栈参数/通用/浮点寄存器参数的位置(基于sp的偏移)只与当前函数声明的(命名)参数列表有关**，**故这些偏移在编译阶段即可确定,** `__builtin_va_start` 基于当前`sp` + offset计算此次运行时这些参数的位置。

## 五、__builtin_va_arg(vl, type)

`__builtin_va_arg(vl, type)` 的作用是返回当前尚未访问的一个类型为`type`的匿名参数的值(注意返回的是值而不是地址), 并调整`vl`中的`__stack`/`__gr_offset`/`__vr_offset`指向下一个匿名参数位置。`__builtin_va_arg`从哪个区域返回匿名参数是由当前`vl`中的信息和此次要获取的参数类型`type`共同决定的:

- `type`为浮点类型时，`__builtin_va_arg`会先尝试在匿名浮点寄存器参数区域查找，若没有则在incoming栈中查找下一个参数
- `type`为整型时, `__builtin_va_arg`会先尝试在匿名通用寄存器参数区查找，若没有则在incoming栈中查找下一个参数

以通用寄存器为例, 判断匿名通用寄存器参数区是否还有参数的标准为:

1. [`vl->__gr_top + vl->__gr_offs` , `vl->__gr_top`] 区域是否还能存下一个类型为`type`的参数
2. 若能存下则按照`type`类型大小返回 `vl->__gr_top + vl->__gr_offs` 中存储**的值**作为下一个匿名参数 (此函数总是返回**值**，无法get到地址)
3. `vl->__gr_offs += sizeof(type);` offset指向下一个匿名通用寄存器参数(未考虑对齐)

其伪代码[1]简化后如下:

```c
type __builtin_va_arg (va_list ap, type)

{

      int nreg, offs;                                    

      /* 若这是个浮点类型的参数(根据type判断) 则先尝试从VPRs 匿名浮点寄存器参数区域中获取参数 */

      if (type passed in simd/fp registers) 

      {

        offs = ap.__vr_offs;

        /* 若匿名浮点寄存器参数区域已没有参数, 则直接去incoming栈区域获取下一个参数 */

        if (offs >= 0)   

          goto on_stack; 	

        

        /* 否则看当前匿名浮点寄存器参数区域中是否能存下此类型的参数 */

        nreg = (sizeof(type) + 15) / 16;

        

        /* 如果存不下则说明此参数只能是栈参数,跳转到on_stack从incoming栈获取 */	

        ap.__vr_offs = offs + (nreg * 16);  	 	

        if (ap.__vr_offs > 0)		   	

          goto on_stack;

          

        /* 如果匿名浮点寄存器参数区域还有参数，则直接从此区域中获取并返回(返回"值"而不是地址) */

        return *(type *)(ap.__vr_top + offs);   	 

          

      } else {					/* 否则尝试从GPRs匿名通用寄存器参数区域获取参数 */

        offs = ap.__gr_offs;

        

        /* 若匿名通用寄存器参数区域为空, 则直接去incoming栈区域获取参数 */

        if (offs >= 0)          		

          goto on_stack;

          

        if (alignof(type) > 8) offs = (offs + 15) & -16; 

        

        nreg = (sizeof(type) + 7) / 8;           /* 计算此类型占用几个标准寄存器空间 */

        /* 根据类型确定匿名通用寄存器区域是否还能存下一个此类型的匿名参数 */

        ap.__gr_offs = offs + (nreg * 8);   

         /* 如果不能(offs > 0),则去incoming栈区域获取下一个参数 */     

        if (ap.__gr_offs > 0)                   

          goto on_stack;

        

        /* 若参数在GPRs匿名通用寄存器区域,则直接返回参数值(而不是地址) */

        return *(type *)(ap.__gr_top + offs);    

      }

    

/* 若此匿名参数不在匿名寄存器区域则其只能保存在incoming栈中, 这里从incoming栈中获取此匿名参数 */

on_stack:

      intptr_t arg = ap.__stack;         /* 获取当前incoming栈中尚未被解析到的匿名栈参数地址 */

    

      if (alignof(type) > 8)			 /* 按需对齐 */

              arg = (arg + 15) & -16;

              

        /* 修改ap.__stack指向下一个可能出现的匿名栈参数的位置 */

      ap.__stack = (void *)((arg + sizeof(type) + 7) & -8); 

      return *(type *)arg;			 /* 返回此匿名栈参数的值 */

}
```

举例如下:
