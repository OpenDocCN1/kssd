
# 对不同存储区域的数组分析

> 原文：[`www.pediy.com/kssd/pediy11/123566.html`](https://www.pediy.com/kssd/pediy11/123566.html)

这算是一篇简单的学习笔记吧，主要内容是从《IDA Pro 权威指南》里看到的，写这篇的目的是为了记录自己的学习历程，另外感觉好像做一些学习笔记会对知识的理解更加有帮助。虽然学得比较慢，但也要坚持，感觉还不错。

下面对放在三个不同存储区的数组进行简要的分析

## 1. 对全局数组进行分析

代码:

```cpp
#include <cstdio>
#include <iostream>
using namespace std;

int global_array[3];

int main()
{
    int idx = 2;
    global_array[0] = 10;
    global_array[1] = 20;
    global_array[2] = 30;
    global_array[idx] = 40;
    
    return 0;
}
```

对全局数组来说，相信大家都知道是放在静态存储区的，就是说在编译时就可以获知该数组的基址，由于基址固定，编译器就可以计算出使用固定索引访问的任何数组元素的固定地址，从这一点可以看出，如果要对程序做出一些保密性措施，就可能对全局变量进行一些隐藏，具体的做法我也不知道，但尽量少定义一些比较重要的全局数组还是有好处的。

这里需要注意两点。

第一，使用常量索引访问全局数组时，在对应的反汇编代码清单中，对应的数组元素以全局变量的形式出现。换句话说，反汇编代码清单基本上不提供任何数组存在的证据。

第二，使用可变索引值将带领我们来到数组的开头，因为在计算要访问的数组元素的具体地址时，需要用到数组的基址加上相应的偏移量，此时基址即呈现出来。

从第一点来看好像对以全局数组可以进行隐藏，但在实际当中有可能恰恰相反，因为第二点可以看出对可变索引值进行访问时就能够得到数组的基址。相信大家在使用全局数组时都不会元素个数很小的，一般全局数组都是用来节省内存空间才采用的一种方法，所以说以常量索引访问全局数组几乎是不可能的。

.代码:

```assembly
text:00401030                 ; =============== S U B R O U T I N E =======================================
.text:00401030
.text:00401030                 ; Attributes: bp-based frame
.text:00401030
.text:00401030                 _main_0 proc near               ; CODE XREF: _mainj
.text:00401030
.text:00401030                 var_44= byte ptr -44h
.text:00401030                 var_4= dword ptr -4
.text:00401030
.text:00401030 55              push    ebp
.text:00401031 8B EC           mov     ebp, esp
.text:00401033 83 EC 44        sub     esp, 44h
.text:00401036 53              push    ebx
.text:00401037 56              push    esi
.text:00401038 57              push    edi
.text:00401039 8D 7D BC        lea     edi, [ebp+var_44]
.text:0040103C B9 11 00 00 00  mov     ecx, 11h
.text:00401041 B8 CC CC CC CC  mov     eax, 0CCCCCCCCh
.text:00401046 F3 AB           rep stosd
.text:00401048 C7 45 FC 02 00 00+mov     [ebp+var_4], 2  //为局部变量 idx 赋值为 2
.text:0040104F C7 05 0C 2E 43 00+mov     dword_432E0C, 0Ah  //从
.text:0040104F                ; 这里以后的三行可以看出来全局数组的地址在编译时候就能够确定，
.text:0040104F                ; 此时的基址应该是 0x00432E0C。红色标记。
.text:00401059 C7 05 10 2E 43 00+mov     dword_432E10, 14h
.text:00401063 C7 05 14 2E 43 00+mov     dword_432E14, 1Eh
.text:0040106D 8B 45 FC        mov     eax, [ebp+var_4]  //把 idx 存储到 eax 寄存器中，用来进行偏移量计算。
.text:00401070 C7 04 85 0C 2E 43+mov     dword_432E0C[eax*4], 28h 
.text:00401070                ; 基址加上偏移量就可以定位数组的元素的下标。eax * 4，4 表示数组元素的大小，
.text:00401070                ; 这里千万不能够就认为是 int 型的，具体情况具体分析。
.text:0040107B 33 C0           xor     eax, eax
.text:0040107D 5F             pop     edi
.text:0040107E 5E             pop     esi
.text:0040107F 5B             pop     ebx
.text:00401080 8B E5          mov     esp, ebp
.text:00401082 5D             pop     ebp
.text:00401083 C3             retn
.text:00401083                 _main_0 endp
.text:00401083
.text:00401083                 ; ---------------------------------------------------------------------------
```

## 2. 对栈分配的数组进行分析

代码:

```cpp
#include <cstdio>
#include <iostream>
using namespace std;

int main()
{
int stack_array[3];
int idx = 2;
stack_array[0] = 10;
stack_array[1] = 20;
stack_array[2] = 30;
stack_array[idx] = 40;
    
return 0;
}
```

从下面的代码我们可以看出，编译器几乎以完全相同的方式处理栈分配的数组和全局分配的数组。由于在编译时候并不能够知道栈分配数组的基址，所以在程序中并没有像在全局数组中一样给出第一个元素的地址，所以对于常量索引访问数组时我们还是不能够快速判断程序中是否使用了数组，因为常量索引访问数组与局部变量的使用看不出有什么区别，但这个不用过于担心，因为常量索引访问数组是很少的。如果数组元素才几个，那还不如直接定义变量进行操作可能更简单。从下面的反汇编代码可以得到一些额外的信息，根据栈中 `idx` 的位置可以推断出，以 `var_C` 开始的数组最多包含 3 个元素（否则，它将覆盖 `idx`），如果对缓冲区溢出有比较深刻的了解的话，可以很清楚的知道这里就可以使该数组溢出，并破坏其后的数据，到底需要在数组中填充多少数据，对于缓冲区溢出的判断和怎么去利用我还是菜鸟，不懂得，以后有学习之后再来补充相关的知识和应用。

代码:

```assembly
.text:00401030                 ; =============== S U B R O U T I N E =======================================
.text:00401030
.text:00401030                 ; Attributes: bp-based frame
.text:00401030
.text:00401030                 _main_0 proc near               ; CODE XREF: _main j
.text:00401030
.text:00401030                 var_50= byte ptr -50h
.text:00401030                 var_10= dword ptr -10h
.text:00401030                 var_C= dword ptr -0Ch
.text:00401030                 var_8= dword ptr -8
.text:00401030                 var_4= dword ptr -4
.text:00401030
.text:00401030 55              push    ebp
.text:00401031 8B EC           mov     ebp, esp
.text:00401033 83 EC 50        sub     esp, 50h
.text:00401036 53              push    ebx
.text:00401037 56              push    esi
.text:00401038 57              push    edi
.text:00401039 8D 7D B0        lea     edi, [ebp+var_50]
.text:0040103C B9 14 00 00 00  mov     ecx, 14h
.text:00401041 B8 CC CC CC CC  mov     eax, 0CCCCCCCCh
.text:00401046 F3 AB           rep stosd
.text:00401048 C7 45 F0 02 00 00+mov     [ebp+var_10], 2
.text:0040104F C7 45 F4 0A 00 00+mov     [ebp+var_C], 0Ah  //这里只能够用 var_C 来表示，并不像全局数组那样用地址表示。
.text:00401056 C7 45 F8 14 00 00+mov     [ebp+var_8], 14h
.text:0040105D C7 45 FC 1E 00 00+mov     [ebp+var_4], 1Eh
.text:00401064 8B 45 F0        mov     eax, [ebp+var_10]
.text:00401067 C7 44 85 F4 28 00+mov     [ebp+eax*4+var_C], 28h
.text:00401067                ; 基址加上偏移量就可以定位数组的元素的下标。eax * 4，4 表示数组元素的大小，
.text:00401067                ; 这里千万不能够就认为是 int 型的，具体情况具体分析。
.text:0040106F 33 C0           xor     eax, eax
.text:00401071 5F             pop     edi
.text:00401072 5E             pop     esi
.text:00401073 5B             pop     ebx
.text:00401074 8B E5          mov     esp, ebp
.text:00401076 5D             pop     ebp
.text:00401077 C3             retn
.text:00401077                 _main_0 endp
.text:00401077
.text:00401077                 ; ---------------------------------------------------------------------------
```

## 3. 对堆分配的数组进行分析

代码:

```cpp
#include <cstdio>
#include <iostream>
using namespace std;

int main()
{
    int *heap_array = (int *)malloc(3 * sizeof(int));
    int idx = 2;
    heap_array[0] = 10;
    heap_array[1] = 20;
    heap_array[2] = 30;
    heap_array[idx] = 40;
    return 0;
}
```

堆分配的数组是使用一个动态内存分配函数（如 C 中的 `malloc`（对应 `free`）或者 C++中的 `new`（`new` 是个运算符，并不是函数，这一点需要弄明白，对应 `delete`）。从编译器的角度讲，处理堆分配的数组的主要区别在于，它必须根据内存分配函数返回的地址值，生成对数组的所有引用。

数组的起始地址（有 `EAX` 寄存器中的 `malloc` 返回）存储在局部变量 `heap_array` 中。每一次访问数组是，首先必须读取 `heap_array` 的内容，以获得数组的基址，然后再在它上面加上一个偏移值，计算出数组中对应元素的地址。对于堆分配的数组有一个非常有用的特点。如果能够确定数组的总大小和每个元素的大小，则可以计算出该数组有多少个元素，对堆分配的数组而言，传递给内存分配函数的参数（示例中式 `0x0C`）即表示了分配给数组的字节总数，用这个除以元素大小，即可得到数组中元素的个数。

代码:



```assembly
.text:00401030               ; ================ S U B R O U T I N E =======================================
.text:00401030
.text:00401030               ; Attributes: bp-based frame
.text:00401030
.text:00401030               _main_0 proc near            ; CODE XREF: _main j
.text:00401030
.text:00401030               var_48= byte ptr -48h
.text:00401030               var_8= dword ptr -8
.text:00401030               var_4= dword ptr -4
.text:00401030
.text:00401030 55            push    ebp
.text:00401031 8B EC         mov     ebp, esp
.text:00401033 83 EC 48       sub     esp, 48h
.text:00401036 53            push    ebx
.text:00401037 56            push    esi
.text:00401038 57            push    edi
.text:00401039 8D 7D B8      lea     edi, [ebp+var_48]
.text:0040103C B9 12 00 00 00  mov     ecx, 12h
.text:00401041 B8 CC CC CC CC mov     eax, 0CCCCCCCCh
.text:00401046 F3 AB         rep stosd
.text:00401048 6A 0C         push    0Ch            ; Size
.text:0040104A E8 41 71 00 00 call    _malloc
.text:0040104F 83 C4 04      add     esp, 4
.text:00401052 89 45 FC      mov     [ebp+var_4], eax  //eax 寄存器返回数组的起始地址
.text:00401055 C7 45 F8 02 00 00+ mov     [ebp+var_8], 2
.text:0040105C 8B 45 FC      mov     eax, [ebp+var_4]
.text:0040105F C7 00 0A 00 00 00 mov     dword ptr [eax], 0Ah
.text:00401065 8B 4D FC      mov     ecx, [ebp+var_4]
.text:00401068 C7 41 04 14 00 00+ mov     dword ptr [ecx+4], 14h
.text:0040106F 8B 55 FC      mov     edx, [ebp+var_4]
.text:00401072 C7 42 08 1E 00 00+ mov     dword ptr [edx+8], 1Eh
.text:00401079 8B 45 F8      mov     eax, [ebp+var_8]
.text:0040107C 8B 4D FC      mov     ecx, [ebp+var_4]
.text:0040107F C7 04 81 28 00 00+ mov     dword ptr [ecx+eax*4], 28h
.text:00401086 33 C0         xor     eax, eax
.text:00401088 5F            pop     edi
.text:00401089 5E            pop     esi
.text:0040108A 5B            pop     ebx
.text:0040108B 83 C4 48      add     esp, 48h
.text:0040108E 3B EC         cmp     ebp, esp
.text:00401090 E8 8B 8F 00 00 call    __chkesp
.text:00401095 8B E5         mov     esp, ebp
.text:00401097 5D            pop     ebp
.text:00401098 C3            retn
.text:00401098               _main_0 endp
.text:00401098
.text:00401098               ; ---------------------------------------------------------------------------
```

## 学习总结

1.  关于数组的使用，通过上面的分析能够得到唯一的确定结论是:只有当变量被用作数组的索引时，才最容易确定数组的存在。但其实我们可以反过来想，一般我们写程序使用数组都是用变量索引来访问数组元素的，很少有常量索引来访问数组元素，所以我们要把握重点，对于具体问题具体分析。

2.  要访问数组中的元素，首先需要用索引乘以数组元素的大小，计算出相应元素的偏移量，然后将得到的偏移量与数组的基址相加，得到数组元素的访问地址。

3.  对于变量索引访问数组的我们一般可以得到数组元素的大小，这一点对于分析也有一定的帮助。
