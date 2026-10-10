
# 基于 IA-64 的 CPU 指令集的学习

> 原文：[`www.pediy.com/kssd/pediy10/100884.html`](https://www.pediy.com/kssd/pediy10/100884.html)

## 一 寄存器(Registers)

### 1. 通用目的寄存器

`r0` 到 `r127` 的寄存器,一共 128 个,前 32 个 `r0` 到 `r31` 是静态的,`r1` 保存全局指针(`gp`),`r12` 保存栈指针(`sp`),每个寄存器是 64 位的宽度,`r32` 到 `r127` 是动态的,一共 96 个;

这里所谓的静态是指 `r0` 到 `r31` 总是指向固定的 CPU 物理寄存器,所谓的动态是指给定寄存器并非总是引用准确相同的 CPU 物理寄存器

寄存器改名是 CPU 达到提升新能目标的基本因素

其实动态寄存器和 CPU 的物理寄存器之间的对应在函数内不会发生变化的，寄存器改名仅仅是控制流从一个函数到另一个函数的时候才发生

IA-64 的每一件事情的唯一的目标就是速度,所以在 64 位系统中参数是通过这些寄存器传递给被调用函数的,比起 X86 从一个较慢的主内存中读参数要快很多，因为后者需要花费 12 个时钟周期，而前者仅仅需要 1 个时钟周期

动态寄存器的存在使得函数能有拥有自己高达 96 个寄存器集使用,`r32` 到 `r127` 是保留给函数内使用的,函数的局部变量和参数都保存在这 96 个寄存器中,但并非每次都用完这么多寄存器

这里就有了非常奇怪的事情了,每个函数都维持这个 96 个寄存器,那么子函数被调用的时候会怎么样呢,事实是,父函数并没有保存当前的 96 个寄存器的内容,不是吧，这样也行,但是确实是真的,在 IA-64 中有一个比较奇妙的东西---寄存器栈引擎(`Register Stack Engine`),这是 IA-64 的一个非常了不起的特性,其中的原理还不太清楚,在这里重要的是记住 `r32` 到 `r127` 对于每个函数都是相互对立的,这就像 win32 时候出现的进程有各自相互独立的地址空间

### 2. 浮点数寄存器

`f0` 到 `f127`,总计 128 个,浮点数寄存器保存就像 C++那样的长双精度值,与整型的寄存器一样,某些浮点数寄存器也有预定义的方式,`f0` 总是设置为 `0.0`,`f1` 总是设置为 `1.0`

### 3. 分支寄存器

`b0` 到 `b7`,一共 8 个, 这些寄存器保存了 CPU 将控制传递到某个位置代码的地址,在 IA-64 中，所有的控制传输都是以分支寄存器的形式存在的.

在 IA-64 中,`br.call` 指令相对于 X86 的 `CALL`,`br.ret` 相对于 X86 的 `RET`,`br` 就是 X86 的中 `JMP`

这些指令的操作数不能像 X86 那样是任意的内存地址,必须首先将地址加载进分支寄存器中,若不是这样,这些操作数也是相对于指令指针的地址

## 二 过程帧,参数传递,寄存器栈引擎(Resgister Stack Engine)

### 1. 过程帧

96 个动态寄存器被划分指向不同的区域,Input,Local,Output

正如图中所示的那样,参数通过 `Input registers` 传递的,局部变量和编译器临时变量是通过 `Local registers` 传递的,`Output registers` 保存了调用下一个提前准备好的函数的参数,编译器决定了对于一个特别的函数需要多少 Input,local,output 寄存器,在函数的预处理部分,编译器放置一个 `alloc` 指令分配所需收入,本地,输出的寄存器数量,对于输出区域的大小,编译器使用主函数调用子函数的最大参数数量,实际的输出寄存器的大小是八个寄存器或者更少

`br.call` 调用之后 96 个动态寄存器将被改名,而且 `CFM` 也会被更新的,在调用函数的时候 outputyou 有不同的名字,在 `br.call` 调用之前输出寄存器是 `r41` 到 `r46`,调用之后将改名为 `r32` 到 `r37`

那么当子函数返回以后会还原之前的寄存器吗，这里就有了 `ar.pfs`,作为 `call` 指令的一部分,CPU 将当前的 `CFM` 寄存器的值复制到 `ar.pfs` 中,在函数调用之后,新的函数是没有输入和局部寄存器的,而且输出寄存器中保存了从父函数传递过来的参数,`alloc` 在这里做一些篡改行为,之前函数的输寄存器分成了新函数的输入寄存器,然后 `alloc` 为新的函数建立新的局部和输出寄存器,最后 `alloc` 保存 `ar.pfs` 到一个通用目的寄存器里面,`alloc` 指令使用的例子如下:

`alloc r36 = ar.pfs, 0x7, 0x0, 0x2, 0x0`

这条指令建立了一个帧,使用了七个输入输入寄存器作为输入区域,零个局部寄存器,两个输出寄存器作为输出区域,最后那个零是可以忽视的,`ar.fps` 被保存到了 `r36` 中,以后建立新的输入，局部和输出寄存时候引用

### 2. 参数传递

参数可能通过通用目的寄存器,浮点数寄存器和线程栈中传递,这里有一个参数槽的概念,它的长度是八字节宽,但是实际的参数的长度是由参数本身的类型决定的,浮点数参数一般使用 `f8` 到 `f15` 的浮点数寄存器进行传递的,任何参数超过八个参数槽将被保存到栈中,如果输入的参数即有整型数,又有浮点数，有些事情变的更加的复杂,在这里参数槽和通用目的输出寄存器有着一一对应的关系,如果浮点数被映射到特别的参数槽中,那么输出寄存器将要被映射到没有被使用的参数槽,例子如下：

```
SomeOtherFunction(void)
{
    int x = 1, y = 2;
    float fpv1 = 1.2, fpv2 = 3.4;
    BarFunction( x, fpv1, fpv2, y );
}
void BarFunction( int a, float b, float c, int d )
{
   
}
```

在 `SomeOtherFunction` 中输出寄存器使用了 `r40` 到 `r43`,寄存器为调用 `BarFunction` 提前建立如下:

```
r40     = x
r41     = ?    // unused
r42     = ?    // unused
r43     = y
f8     = fpv1
f9     = fpv2
```

超过第一个八字节槽的参数将被保存到调用函数的栈中,首先的八字节参数槽在栈指针之上的 `0x10` 字节处,随后进来的参数槽都是大于八字节参数槽的,值得注意的是,`br.call` 并不改名栈指针,这个和 X86 栈操作不同,对于编译器来说会尽量避免使用栈传递参数，因为内存访问是很慢的,对于比八字节更大的参数,一部分保存到寄存器中,剩下保存到栈中,编译器知道它被分成了两个部分

### 3. 函数返回

IA-64 对于任意的符合八字节或者更小的字节的整型数保存到 `r8` 寄存器中, `r8` 就相当于 X86 中的 `EAX`,对于大于八字节的整型返回值可以使用 `r8` 到 `r11` 的寄存器,浮点型的返回值一般通过 `f8` 返回,如果返回值超过了单个 `r8` 能表示的范围,它将可以使用 `f8` 到 `f11` 的浮点数寄存器;在 X86 中返回地址在调用子函数的时候已经提前被保存到了栈中,但是在 IA64 中,栈中更本没有保存返回地址,而是在分支寄存器里,考虑如下指令:

`br.call.dptk.many b0 = b6`

`qptk.mang` 是 `br.call` 的一部分，在这里并不是重要的，`b6` 保存了 `call` 的下一条指令的地址,总之,返回地址一般被保存在 `b0` 中,但是在进入被调用函数内,`b0` 的值要被保存通用目的寄存器中,因为被调用函数还调用其他的子函数，如果 `b0` 不临时的保存到其他的寄存器中的话,它的值有可能被下一个子函数的返回地址覆盖,在子函数通过 `br.ret` 返回到调用函数的时候,除了要设置返回地址寄存器外,还要改变过程帧和寄存器名字到调用函数的状态,这个时候隐含的一个操作是 `ar.pfs` 移动到 `CFM` 寄存器

### 4. 寄存器栈引擎(The Register Stack Engine)

`RSE` 是 IA-64 中非常重要的一个特性,应用程序更本不知道它的存在以及它是如何工作的，实际上,它更像一个后台硬件线程,这个线程将读取和写入动态的寄存器到外部的内存中，这个内存是为以后恢复而使用的,并且操作系统负责为它分配内存,这里有一个分配原则,嵌套越深的函数的寄存器,最新越少使用的被写入内存,这样可以释放掉这些不常用的寄存器为新的调用使用,这里有一对寄存器 `ar.bsp` 和 `ar.bspstore` 保存了之前被溢出到内存的寄存器的轨迹,当函数返回的时候,`RSE` 知道重新加载之前保存在内存中的寄存器,这里要搞清楚一个一一对应的关系,在函数帧中的寄存器和保存在内存中已被日后恢复的寄存器

## 三 全局指针(Global Pointer)

### 1. 全局指针

是提前指定的访问一个加载模块内数据的值,这个指针在 X86 中是没有必要的,因为对于 32 位(`4GB`)的指令足以容纳 32 位的地址成为编码的一部分,例如:

`MOV ECX,\[0x12345678\]`

这个指令一共占了六个字节,`MOV ECX` 占两个字节,剩下的四个字节是双字的地址----24 位

在 IA-64 中,每个指令的长度是 41 位,那么保存 64 位的地址是不可能的,所以所有的内存访问必须通过加载通用目的寄存器的指针值才可以,对于每一个基于 IA-64 的加载模块都一个全局指针值,这个值指向加载模块静态数据区域附近,这些数据包括,全局变量,字符串,COM 虚函数表,其他各种项目,对于当前的加载模块全局指针的值是被加载到 `r1` 中的,在反汇编工具和调试器中,为了记忆的方便,`r1` 通常被另一个名字引用的,`GP` 寄存器

加载模块的数据区域被限制为 `4MB`,因为在 IA-64 的一个条单指令能被增加到寄存器的最大立即数是 22 位,计算一下你会发现 22 位引用 4MB 的内存大小

几乎所有的加载模块的数据是相对于 `GP` 编址的,所以指令中没有包含任何硬编码的地址,不像 X86 经常需要那样做,代码是不依赖于位置的,每次在内存加载一个地址是不需要修改为正确的值,基本消除了 X86 系统上的基址重地位机制

那么超过 4MB 的数据该如何处理呢,这就和编译器有关了,在较小的数据区域存储了一个指向大内存区域的指针,编译知道什么时候间接的通过一个指针得到大内存区域的地址,早在 1992 年 win32 就包含了这个指针的定义,在 `WINNT.H` 里可以查到`#define IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR`

全局指针的设置是在调用 `CALL` 的时候发生的,也就是父函数负责设置正确的全局指针值,这个值保存在一个 Intel 提前定义的数据库结构中,是经过微软同意为了实现 64 位系统

### 2. 介绍 PLEBAL\_DESCRIPTOR

在微软的 Visual Studio 6.0 的 `WINNT.H` 可以找到对这个结构的定义:

```
 
typedef struct \_PLABEL\_DESCRIPTOR { 
 
ULONGLONG EntryPoint; 
 
ULONGLONG GlobalPointer; 
 
} PLABEL\_DESCRIPTOR, \*PPLABEL\_DESCRIPTOR; 
```

在每个加载模块的函数中都有一个这样的结构,是编译器自动生成的,`EntryPoint` 是指向函数地址,`GlobalPointer` 是函数相关的 `GP` 的值,正常情况下,所有的 `PLABEL\_DESCRIPTOR` 指在加载模块内包含相同的全局指针值,当加载模块的时候操作系统初始化这些 `PLABEL\_DESCRIPTOR` 的值,当生成 `call` 调用的时候,首先从 `GlobalPointer` 得到 `GP` 的指针,虽然从 `EntryPoint` 得到跳转地址

PE 头的加载模块的入口点 `rva`,导出函数的 `rva`,导入地址表的 `rva`,COM vtables 的入口点 `rva` 在 IA-64 中都指向 `PLABLE\_DESCRIPTOR`,而且 `GetProcAddress` 返回的地址也是一个 `PLABEL\_DESCRIPTOR` 结构,编译器假象所有的函数指针都指向 `PLABEL\_DESCRIPTOR` 并且生成合适的代码,这里举一个 C++的例子：

```
void foo(void)
{
}
void main()
{
    void \* p = &foo;
}
```

`&foo` 解析的是 `foo` 的 `PLABEL\_DESCRIPTOR`,而不是 `foo` 的地址,如果想得到函数的地址,就映射函数的地址到 `PLABEL\_DESCRIPTOR`,然后读这个接口第一域 `EntryPoint`,它的值就是真正的函数地址

这里贴上微软 MATT 的例子:

```
#define WIN32\_LEAN\_AND\_MEAN
#include <windows.h>
#include <stdio.h>
#ifndef \_M\_IA64
#error "This program only runs on an IA64 system\\n" );
#endif
PVOID GetGlobalPointerFromHMODULE( HMODULE hModule );
int main()
{
// Get the "address" of function main().  This address is really
// a PLABEL\_DESCRIPTOR, defined in WINNT.H
PLABEL\_DESCRIPTOR \* pLabelDesc = (PLABEL\_DESCRIPTOR \*)&main;
// Print out the PLABEL\_DESCRIPTOR fields
printf( "EntryPoint:    %p\\n", pLabelDesc->EntryPoint );
printf( "GlobalPointer: %p\\n", pLabelDesc->GlobalPointer );
// Just for fun, find out and display the GlobalPointer of this EXE,
// using the RVA stored in the PE header.  This GlobalPointer should
// match the GlobalPointer from above.
HMODULE hModule = GetModuleHandle( 0 );    // Get HMODULE of this 
// program
PVOID globalPointer = GetGlobalPointerFromHMODULE( hModule );
printf( "GlobalPointer from PE header: %p\\n", globalPointer );
return 0;    
}
// MakePtr is A handy macro function for creating pointers of the 
// appropriate type.  It adds the two values as integral types, and cast 
// the result to the desired pointer type
#define MakePtr(cast, ptr, addValue) \\
(cast)((DWORD\_PTR)(ptr) + (DWORD\_PTR)(addValue))
PVOID GetGlobalPointerFromHMODULE( HMODULE hModule )
{
// Point to the "DOS" header so that we can locate the "PE" header
PIMAGE\_DOS\_HEADER pDosHdr = (PIMAGE\_DOS\_HEADER)hModule;
// Make a pointer to the PE32+ header   
PIMAGE\_NT\_HEADERS pNTHdr = MakePtr( PIMAGE\_NT\_HEADERS,
hModule,
pDosHdr->e\_lfanew );
// Get the RVA of the GlobalPointer from the PE's DataDirectory
DWORD gpRVA = pNTHdr->OptionalHeader.DataDirectory
\[IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR\].VirtualAddress;
// Add the RVA to the module load address to form the GlobalPointer 
// address
return MakePtr( PVOID, hModule, gpRVA );
}
```

IA-64 指令集的东西不止这些，INTEL 的官方手册上介绍的很详细，也很头疼,以上介绍的对于在调试器中调试程序应该没什么问题了

参考:

[`www.intel.com/design/`](http://www.intel.com/design/)

[`msdn.microsoft.com/zh-cn/maga...11(en-us).aspx`](http://msdn.microsoft.com/zh-cn/magazine/cc301711\(en-us\).aspx)



[`msdn.microsoft.com/zh-cn/maga...17(en-us).aspx`](http://msdn.microsoft.com/zh-cn/magazine/bb985017\(en-us\).aspx)
