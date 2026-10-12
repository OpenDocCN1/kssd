
# 汇编语言学习笔记

> 原文：[`www.pediy.com/kssd/pediy11/119420.html`](https://www.pediy.com/kssd/pediy11/119420.html)

这个文档还没写完，我会继续写的，由于我这里的网速慢，有些图片没传上来，我会传个附件上来，但不知能成功，第一次发帖，见谅哈^\_^.

这里是附件:

汇编语言学习笔记

\---傻瓜学汇编

## 前言

当我在学汇编的时候发现一到了实际编程就发现学过的那些指令串不起来，什么浮点数啊整数啊，怎么跳转啊，怎么循环啊，脑袋立马变成浆糊。下面的文档是我的学习经历，希望对初学者在学习加密解密，软件调试，单片机编程有点帮助。

## 目录

1. 编程环境的搭建
2. 深入理解汇编语言的数据
3. 顺序程序设计
4. 分支结构程序设计
5. 循环
6. 数组及指针
7. 函数
8. 结构
9. 综合运用
10. 参考文献

# 一：编程环境的搭建

首先装好 `masm32v10` 和 `windbg`，和 `editplus`，然后在 `editplus` 中输入下面的程序，具体的请参考罗云彬的那本书，里面有详尽的说明，编译运行看看：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
msg db "hello,this is the first test program!",0dh ,0ah,0
.code
start:
 call main
 ret
main proc
 push offset msg
 call printf
 add esp,4
 push offset msg
 call printf
 add esp,4
 ret
 main endp
end start
```

下面是运行结果：

这里输出两行消息主要是我在写这个最简单的程序的时候发现他不换行，于是我在数据定义后面我加了“0ah，0dh”，呵呵，就是回车换行的十六进制表示，你也可以用其他方法试试，

程序就不多解释了，后面会有更多的解释，不过你一定要走到这以步，才能进行下一章。

## 2.深入理解汇编语言的数据

整数常量及变量，先看一段很简单的汇编程序：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
PRICE EQU 30
msg1 db "total=%d",0dh,0ah,0
.code
start:
 call main
 ret
main proc
 local num:dword
 local total:dword
 mov num,10
 mov eax,num
 imul eax,eax,PRICE
 mov total,eax
 push offset msg1
 call printf
 add esp,4
 ret
 main endp
end start
```

程序的意思很简单就是在屏幕上打印出某个东西的价格，如过要你拿笔和纸算，拿你肯定很快就能算出来，但你让电脑怎么算呢？当电脑执行到第一个语句的时候，也就是 `num=10`，

它就把 10 放到某个地方并且记住这个值，寄存器或者内存，呵呵，它也就这两个地方，为什么要这么做呢？因为后面要用它来计算啊，为了算出这个值，电脑好的办法就是放在它的内存里，为什么不是寄存器？因为寄存器太少了，就那么几个，呵呵，所以了它就把 10 存在一个叫 `num` 的内存里，注意了哦，`num` 是程序里的变量名，是存中里的一个位置的名称，它的值是 10，你可能会问，不起名不行么？行，等下在调试器中你看到的就是没名的。来看看它在调试器中的样子：

`num` 变成了`[ebp-4]`了，现在你想象有个几千行的程序如果都用`[ebp-4]`这样的名字的话，那我们会疯的，所以汇编程序就让我们给程序里面的变量起个直观的名字，而不是用具体的数字去让你去记住变量内存的位置。程序中 `imul eax,eax,1eh` 中的 `1eh` 就是个整形常量，也就是 30.现在你应该对常量和变量有点感觉了吧。

再看个例子：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
a db 12h
b dw 1234h
c1 dd 12345678h
msg1 db "the number is=%xh",0dh,0ah,0
.code
start:
 call main
 ret
main proc
 mov al,a
 cbw
 cwde
 push eax
 push offset msg1
 call printf
 add esp,8
 mov ax,word ptr a
 cwde
 push eax
 push offset msg1
 call printf
 add esp,8
 mov eax,dword ptr a
 push eax
 push offset msg1
 call printf
 add sp,8
 ret
 main endp
end start
```

首先，你得想 `a`，`b`，`c1` 三个变量在程序中到底是怎么存的，是 12 12 34 12 34 56 78，还是 78 56 34 12 34 12 12 呢？呵呵，用调试器载入程序看看就知道了：

哈哈，看到了没，正确的是这个：00403000： 12 34 12 78 56 34 12 74-68，这是为什么？

还有就是这个程序打印的三个结果又是什么呢？是 12h 和 0012h 和 00000012h 吗？如果是，那你就错了哦，应该是：the number is=12h the number is=3412h the number is=78123412h

呵呵，首先，你得明白这三个你定义的数据在内存是怎么存的，一个原则就是你定义的数据的高位存在内存中的高字节地址，你定义的第二个数据：1234h，高位字节是 12 吧，低位字节是 34 吧，所以编译器它先存 34 字节存在内存的低地址，然后再把 12 存在高地址，当然如果是你只定义了一个字节那顺序就没反了，就像你定义的第一个字节数据 12 好一样，同样第三个双自数据 12345678h，编译器它就先存 78 好字节了，然后是 56 好字节，34h 字节，12h 字节。下面我们来看看程序：

`mov al，a`，就是是把 12h 放到 `al` 中，`movzx ax，al` 0 扩展指令，将 `al` 中的字节扩展到 `ax` 中，不足的位用 0 填充，不改变 `al` 的值，`al` 里面是什么值，扩展后 `ax` 的值还是等于 `al` 中的值。 `movzx eax,ax` ;0 扩展指令，将 `ax` 中的字节扩展到 `eax` 中，不足的位用 0 填充，不改变 `ax` 的值，`al` 里面是什么值，扩展后 `eax` 的值还是等于 `ax` 中的值。然后 `push eax`，和 `push offset msg1`，`call printf` 就是调用 `c` 语言库函数 `printf` 打印消息，

就相当于 `c` 语言里面的:`printf("the number is=%xh\n",a);`下面的和这段一样，我就不写废话了。如果面对的是有符号数，那就得用 `movsx` 了，当然还有其他指令，后面再介绍。

### 浮点数：

在计算机内部，浮点数是以二进制表示的，所以，要先转换为二进制浮点数，转换分两部，整数部分的装换，采用“除 2 取余法”，小数部分的装换，采用“乘 2 取整法”，例如 19.2，

先将 19 转换成二进制：10011，然后将 0.2 转换成二进制：00110011……0011，它是个无穷循环小数，然后就是规格化，分三种情况：如果定义的数据类型是 `dword` 或者是 `real4`，那么符号位占一位，阶码占 8 位，位数占 23 位，总共是 32 位，如果定义的类型是 `qword` 或 `real8`，那么符号位占一位，阶码占 11 位，位数占 52 位，总共 64 位，如果定义的类型是 `real10` 或者是 `tword`，那么阶码占 15 位，位数占 64 位，符号位占一位，总共 80 位。怎么算阶码呢？如果是 32 位，就将阶码加上 127，然后转换成二进制，如果是 64 位，就加上 1023，如果是 80 位，就加上 16383。我们看看怎么将 19.2 转换成 32 位的二进制浮点数：

首先将 19 转换成二进制：10011，然后将 0.2 转换成二进制：00110011……0011，整理成 32 位就是：10011，001100110011001100110011001。然后规格化为：1，0011001100110011001100110011001x2 的 4 次方，阶码为 127 加 4 等于 131：10000011。所以当浮点数 19.2 表示为三种不同的数据类型为：

32 位（`dword`，`real4`）：0，10000011，0011001100110011001101

64 位（`qword`，`real8`）：0，100000000011，0011001100110011001100110011001100110011001100110011

80 位：0，100000000000011，001100110011001100110011001100110011001100110011001100110011001100110011。

转换成 16 进制就是 4199999Ah，40333333 33333333 h，403999999999999999ah。

然后我们 `yong` 程序来验证一下对不对。例子如下：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
f1 real4 19.2
f2 real8 19.2
f3 real10 19.2
msg1 db "the floating number is=%g",0dh,0ah,0
.code
start:
 call main
 ret
main proc
 local f:real8
 fld dword ptr f1
 fstp f
 push dword ptr f+4
 push dword ptr f+8
 push offset msg1
 call printf
 add esp,12
 push dword ptr f2+4
 push dword ptr f2
 push dword ptr offset msg1
 call printf
 add esp,12
 fld f3
 fstp qword ptr f
 push dword ptr f+4
 push dword ptr f+8
 push offset msg1
 call printf
 add esp,12
 ret
 main endp
end start
```

程序很简单，就是分别在屏幕上打印三个浮点值，如下图：

在这里我要说明下，我只有把 32 位和 80 位的转换为 64 位的，才能打印成功，这可能是库函数 `printf` 的原因，怎么转换呢？

32 位浮点转换 64 位浮点：首先得借助一个 64 位的浮点局部变量：

```
local f:real8
 fld dword ptr f1
 fstp f
```

第一句定义了 `f` 位一个 64 位的浮点局部变量，第二句就是把 32 位浮点数转换为 80 位的，然后第三句就是把 80 位的转换位 64 位的。

80 位浮点转换位 64 位浮点数：同样借助一个 64 位的浮点局部变量：

```
fld f3
fstp qword ptr f
```

第二句就是把 80 位的浮点转换位 60 位的。

但是这两句怎么解释呢：`push dword ptr f2+4`

`push dword ptr f2`

为什么要先把 `f2` 的高 4 位字节入栈呢？

好，我们先来看看这个数转换成 64 位的 16 进制为：40333333 33333333h，前面我说了高低对应原则，那么这个 64 位的 16 进制在内存中高 4 字节地址应该存 40333333h，也就是存它的高 4 字节，然后是 33333333h，但是，呵呵，在堆栈中的地址是从高往低增长的，所以我们应该先把这个数的高四字节入栈，也就是 40333333h，怎么在内存中得到这高 4 字节呢？就是从 `f2+4` 处压入 4 字节就可以了，然后就是低 4 字节入栈。如果还没理解，用 `cdb` 调试一下就清楚了。

### 浮点与整数之间的转换：

先看例子成 `ch2-4`：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
f1 real8 19.2
f2 dword 20
msg1 db "the floating to int number is=%d",0dh,0ah,0
msg2 db "the int to floating number is=%f",0dh,0ah,0
.code
start:
 call main
 ret
main proc
 local f:real8
 fld f1
 fistp dword ptr f
 push dword ptr f
 push offset msg1
 call printf
 add esp,8
 fild f2
 fstp f
 push dword ptr f+4
 push dword ptr f+8
 push offset msg2
 call printf
 add esp,12
 ret
 main endp
end start
```

运行结果为：

浮点数转换成整数：

```
fld f1
 fistp word ptr f
```

首先我们还是借助了一个 64 位的局部变量，先把浮点数装入浮点寄存器，然后用装换整行的指令变成整数再存入一个局部变量就行了。

整数转换成浮点数：

```
fild f2
fstp f
```

先把整数用装换指令装入浮点寄存器，然后把浮点数存到一个局部变量就可以了。

我在后面会详细说名浮点数的运算和浮点寄存器的。

### 字符与字符串常量：

怎么定义他们？他们是以什么形式存在计算机中？

首先我们怎么在汇编中定义他们呢：先看看例子 `ch2-5`：

```
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
.data
str1 db 'this is a string test',0ah,0dh,0
str2 db "this is a string test",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 mov eax, offset str1
 push offset str2
 call printf
 add esp,4
 push offset str1
 call printf
 add esp,4
 ret
 main endp
end start
```

程序就是在屏幕上打印两行消息，下面是运行结果：

然后我们用 `cdb` 调试器看看定义的那两个字符串变量在内存中到底是怎么样的：

恩，它们是以 `asii` 码的形式存在的。

其他的数据类型我会在下面的各个章节会随着编程的算法和调试一起讲解。

# 3：顺序程序设计

汇编语言的顺序编程比较好理解，就是在编程的时候没有跳转，没有循环，看看例子 `ch3-1`：

例 `ch3-1`：输入三角形的边长，求三角形的面积。

我假设输入的三边长都是能构成三角形的，求三角形面积的公式为 `area=s(s-a)（s-b（s-c）`。

`s=（a+b+c）/2.`



这里要用到浮点指令，那就先回顾下浮点指令的用法：这里要加减乘除和平方根五种指令,由于 Intel 的浮点数据寄存器是种堆栈结构，我们要记住这一点。

### 先看看数据传送指令
`fld` 和 `fild`，`fst`，`fstp`：

`fld` 源操作数，源操作数可以是浮点寄存器和内存，这个指令主要是把源操作数压入浮点寄存器堆栈（其实就是 `st0`），如果源操作数是整数，那就用 `fild`。

`Fst` 和 `fstp` 是把 `st(0)` 浮点寄存器中的数弹出到目的操作数中，目的操作数可以为浮点寄存器和内存。

### 加减法指令
`fadd`，`faddp`，`fsub`，`fsubp`

第一种形式：`fadd` 目的操作数，源操作数。其中目的操作数，源操作数可以为浮点寄存器和内存。

第二种形式：`fadd` 源操作数，我本人比较喜欢这种，它不会把我脑海里的浮点寄存器的顺序弄乱，这种形式的源操作数只能是内存。减法指令同加法指令，就不多说了。

### 乘除法指令
`fmul`，`fdiv`

浮点的乘除法是不区分有符号和无符号数的，他们也有两种形式：

`fmul` 目的操作数，源操作数 和 `fmul` 源操作数这两种形式，第一种 操作数和源操作数可以为浮点寄存器和内存，但第二种 源操作数 值能为内存。

### 平方根指令
`fsqrt`

这个指令就一种形式就是 `fsqrt`，就是把第 0 个浮点寄存器 `st0` 的值变成平方根值然后存在 `st0` 中。

### 例题计算过程

再回到例题中，我们应该先算 `s` 值，然后再算平方根下面的值然后求平方根就行了。

`S` 值是三边长除以 2，转换成浮点指令就是：`fld` `a`；先把 `a` 值放到浮点寄存器 `st0` 中

再就是 `fadd` `b`；这个就是 `a+b`，结果存在 `st0` 中

然后 `fadd` `c`；同上，结果存在 `st0` 中

现在算出了三边长的和，在除以 2 就 ok 了 `fdiv` `two`；结果在 `st0` 中，

最后把 `st0` 里面的值用 `fstp` `s` 存到 `s` 中就把 `s` 值算出来了。

再来算根号下面的值，这里有乘法和减法，我们先算减法：`fld` `s`

`fsub`，`a`；这个就是 `s-a`，结果在 `st0` 里

再 `fld` `s`

`fsub` `b`；这个结果还是在 `st0` 里，但是上面那个 `s-a` 已经被推到 `st(1)` 这个浮点堆栈了啊，记住了。

再 `fld` `s`

`fsub` `c`；这个结果还是在 `st0` 里，

那么 `s-a` 被推到 `st(2)` 了，`s-b` 被推到 `st(1)` 了，`s-c` 的值在 `st(0)` 中，这里千万不能及乱了啊，下一步算乘法，先算 `s*(s-c)`， `mul` `s`；果在 `st0`li

`fmul` `st(0)`，`st(1)` 这一步是算 `s*(s-a)*(s-b)`

最后算 `s*(s-a)*(s-b)*(s-a)` `fmul` `st(0)`，`st(2)`，

然后把 `s*(s-a)*(s-b)*(s-a)` 的结果再求平方根 `fsqrt`

再把这个平方根的值弹出到 `area` 中。

这是按照上面的想法写出的程序：

```asm
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
scanf proto C :VARARG
.data
msg1 db "%f,%f,%f",0
msg2 db "area=%7.2f",0ah,0dh,0
msg3 db "please input three floating numbers",0ah,0dh,0
two real4 2.0
.code
start:
 call main
 ret
main proc
 local a:real4
 local b:real4
 local c1:real4
 local s:real4
 local area:real8
 push offset msg3
 call printf
 add esp,4
 lea eax,c1
 push eax
 lea eax,b
 push eax
 lea eax,a
 push eax
 push offset msg1
 call scanf
 add esp,16
 fld a
 fadd b
 fadd c1
 fdiv two
 fstp s
 fsub a
 fsub b
 fsub c1
 fmul s
 fmul st,st(1)
 fmul st,st(2)
 fsqrt
 fstp area
 push dword ptr area+8
 push dword ptr area+4
 push offset msg2
 call printf
 add esp,12
 ret
 main endp
end start
```

编译运行：下面是结果：

但是出乎意料哦，错了，怎么办呢？你发现什么问题了没？那只有调试看看了，用

`cdb -2 C:\\w\\b\\ch3\\ch3-1\\ch3-1` 打开程序然后逐步调试看看，当跟踪到下面这句是就发现有问题了：

就是 `fstp`，`s`，也就是 `00401041 d95df0 fstp dword ptr [ebp-10h] ss:0023:0013ffac=a9e45d08`，然后发现 `st` 中的值被弹走了，但紧接着又用 `st(0)` 用与下一步进行计算了，先推出调试，改动下再运行看看对不对，但结果还是错的：

那还得继续调试了，发现我对 `fsub` 指令理解错了，原来这个指令执行完之后它不把结果推到下一个浮点堆栈：

那只能用 `fsubp` 了，改动如下：

```asm
fld a
fsubp st(1),st
fld s
fld b
fsubp st(1),st
fld s
fld c1
fsubp st(1),st ;然后改成这种了
```

但结果还是错的，一看到这句 `push dword ptr area+8`
`push dword ptr area+4`
想一下“高低原则”，改成 `push dword ptr area+4`
`push dword ptr area+8`

编译运行，哈哈，ok 了：

### 最后正确的代码

```asm
.386
.model flat,stdcall
option casemap:none
includelib msvcrt.lib
printf proto C :VARARG
scanf proto C :VARARG
.data
msg1 db "%f,%f,%f",0
msg2 db "area=%7.2f",0ah,0dh,0
msg3 db "please input three floating numbers",0ah,0dh,0
two real4 2.0
.code
start:
 call main
 ret
main proc
 local a:real4
 local b:real4
 local c1:real4
 local s:real4
 local area:real8 ;这里是变量定义
 push offset msg3
 call printf
 add esp,4 ;这里是提示输入三个边长
 lea eax,c1
 push eax
 lea eax,b
 push eax
 lea eax,a
 push eax
 push offset msg1
 call scanf
 add esp,16 ;这里是调用库函数 scanf
 fld a
 fadd b
 fadd c1
 fdiv two
 ;fstp s 这一句其实是有问题的，你可以调试看看
 fst s ;改成这句
 ;fsub a
 ;fsub b
 ;fsub c1 ;经调试发现这三句都是错的
 fld a
 fsubp st(1),st
 fld s
 fld b
 fsubp st(1),st
 fld s
 fld c1
 fsubp st(1),st ;然后改成这种了
 fmul s
 fmul st,st(1)
 fmul st,st(2)
 fsqrt
 fstp qword ptr area ;这一段是计算 area 值,我的文档上有详细解释
 push dword ptr area+4
 push dword ptr area+8
 push offset msg2
 call printf
 add esp,12 ;打印结果
 ret
 main endp
end start
```

呵呵，定义的时候用了 `c1`，因为 `c` 是汇编里面的关键字，所以我就改了下，你可能会说，你怎么不优化一下呢？呵呵，先把程序写对了，把指令理解透了再去优化也不晚。

### 练习 ch2-2

求下面 `x1` 和 `x2` 的值：

`x1=`（`b+b*b-4ac`）`/2a`, `x2=`（`b- b*b-4ac`）`/2a`, `a,b,c` 由键盘输入。

提示下真确运行结果：

`a=1`，`b=3`，`c=2`

`x1=-1.00`

`x2=-2.00`

## 4.选择分支结构程序设计

前面我们看到的程序都是从上到下这样执行的，程序的执行流程没有跳转和循环，cpu 为我们实现程序的跳转提供了硬件环境和一些指令，首先 cpu 它有个处理器状态寄存器，跳转指令就是无条件跳转 `jmp` 和有条件跳转指令 `jcc`（里面包含了很多种形式的），总的来说就是你在你写的程序里面用跳转指令让 cpu 取改变程序的执行流程，cpu 会根据那个状态寄存器的一个或多个值的改变来执行程序，我们先来看个例子：

### 例 4.1：输入 3 个整数 a，b，c，请输出其中最大的数。

我们只要一次把这三个数比较一次，在每次比较中，把那个大值继续和下一个数来做比较，然后再输出那个最大值就行了。代码如下：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db "please input three int numbers",0ah,0dh,0
msg2 db "The Max of three numbers is =%d",0ah,0dh,0
msg3 db "%d,%d,%d",0
.code
start:
 call main
 ret
main proc
 local a:dword
 local b:dword
 local c1:dword
 push offset msg1
 call printf
 add esp,4
 lea eax,c1
 push eax
 lea eax,b
 push eax
 lea eax,a
 push eax
 push offset msg3
 call scanf
 add esp,16 ;提示用户输入三个整型数
 mov eax,a
 cmp eax,b
 jg a_compare_c ;跳转到将 a 与 c 比较
 mov eax,b
 cmp eax,c1
 jg print_max ;到这里三个数已经全部比较完，跳转到打印最最大值
 @@:
 mov eax,c1 ;到这里三个数已经全部比较完，跳转到打印最最大值
 jmp print_max
 a_compare_c:
 cmp eax,c1
 jg print_max ;到这里三个数已经全部比较完，跳转到打印最最大值
 jmp @b
 print_max:
 push eax
 push offset msg2
 call printf
 add esp,8
 ret
 main endp
end start
```

运行结果：

无论是整数还是负数，结果都是正确的。这里的 `cmp` 指令相当于减法，只是它不保存运算结果，而是改变前面说的状态寄存器，具体的就是状态寄存器的符号位，零标志位，溢出标志位和进位位，然后那个条件跳转指令 `jg` 就会根据上面那些状态位来确定怎么跳转，这里的 `jg` 即使指令就是根据符号位和零标志位来确定跳转的，你可以用 `windbg` 验证一下。在分支程序设计中，在前面有比较，比较完之后看当前情况满足哪种情况，然后用跳转指令跳取即可，前面这个判断中是个最简单的判断，下面我们来看看比较复杂的情况，先看例子 ch4-2：

### 例 ch4-2 输入一个年份，判断是否闰年。

在判断是否是闰年时，我们可以用一个复杂的判断表达式来包含所有的闰年条件，也就是当输入的年份（`year%4==0&&year%100!==0`）||`year%400==0` 则输出这个年份是闰年，否则输出不是，现在的问题就是怎么把上面的那个复杂的判断用汇编语言来正确的表达，下面是代码：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db "please input a year",0ah,0dh,0
msg2 db "%d is not a leap year",0ah,0dh,0
msg3 db "%d is a leap yaer" ,0ah,0dh,0
msg4 db "%d"
.code
start:
 call main
 ret
main proc
 local year:dword
 local leap:dword
 push offset msg1
 call printf
 add esp,4 ;到这里是输出提示信息
 lea eax,year
 push eax
 push offset msg4
 call scanf
 add esp,8 ;这几句是调用输入函数把输入的年份存到 year 变量里
 mov eax,year
 xor edx,edx
 mov ebx,400
 div ebx
 test edx,edx
 jz is_a_leap_year ;这里先测试年份除以 400，如果满足就直接跳转到打印是闰年的代码处
 mov eax,year
 xor edx,edx
 mov ebx,4
 div ebx
 test edx,edx ;到这里就是判断闰年的第二种情况了，如果除以 4，不能整除就直接跳到打印不是闰年的代码处
 jnz is_not_a_year ;如果能整除就继续判断
 mov eax,year
 xor edx,edx
 mov ebx,100
 div ebx
 test edx,edx
 jz is_not_a_year ;这里是继续判断这个年份能不能整除 100，如果你能整除就跳到打印不是闰年代码处
is_a_leap_year:
 push year
 push offset msg3
 call printf
 add esp,8 ;打印是闰年的信息
 jmp out_this_program ;然后跳转到退出本程序
is_not_a_year:
 push year
 push offset msg2
 call printf
 add esp,8 ;打印不是闰年消息
out_this_program:
 ret ;退出本程序的地方
 main endp
end start
```

下面是运行结果：

`C:\\w\\b\\ch4\\ch4-2>ch4-2`



`please input a year`

2008

2008 is a leap yaer

`C:\\w\\b\\ch4\\ch4-2>ch4-2`

`please input a year`

2000

2000 is a leap yaer

`C:\\w\\b\\ch4\\ch4-2>ch4-2`

`please input a year`

1989

1989 is not a leap year

这里我们先判断那个比较简单的测试点就是看年份能不能能整除 400，如果为真，就直接跳转到打印是闰年的代码处，然后再判断看能不能整除 4，如果为假，就直接跳转到打印不是闰年的代码处，然后下面的就不用再判断了，呵呵，因为这里是 `&&` 逻辑与哦，就是著名的“短路表达式”，程序本身比较简单，就不多解释了。下面来看看判断语句的嵌套。

### 判断语句的嵌套

判断语句的嵌套就是判断里面又有判断，呵呵，先看例子：

#### 例 ch4-3：有一函数

从题意可以看出，当输入一个 x 值，程序就得先判断是不是小与 0 啊，然后紧接着判断是不是等于 0 啊，再判断是不是大于 0 啊，然后才算判断完全，下面是代码：

``` 
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db "please input a int number",0ah,0dh,0
msg2 db "the Y value is %d",0ah,0dh,0
msg4 db "%d"
.code
start:
 call main
 ret
main proc
 local x:dword
 push offset msg1
 call printf
 add esp,4 ;输出提示信息
 lea eax,x
 push eax
 push offset msg4
 call scanf
 add esp,8 ;调用 scanf 将输入的一个值存在局部变量 x 里面
 ;mov eax,x
 ;cmp eax,-1
 ;je Y_equal_minus_one
 ;test eax,eax
 ;jz Y_equal_zero
 ;cmp eax,1
 ;je Y-equal_one 未优化的嵌套判断语句
 ;下面是有点优化的嵌套判断语句
 mov eax,x
 inc eax ;x 如果是-1 加 1 就是 0 啦，呵呵
 jle Y_equal_minus_one ;如果是 0 就跳转到打印 y 值的代码处
 mov eax,x
 test eax,eax ;用 test 来判断寄存器是否为 0，感觉要比 cmp eax,0 要好很多，你可以用 cdb 看看
 jz Y_equal_zero
 mov eax,x
 dec eax ;x 如果是 1 减 1 就是 0 啦，呵呵
 jae Y_equal_one ;如果是 0 就跳转到打印 y 值的代码处
Y_equal_minus_one:
 push -1
 push offset msg2
 call printf
 add esp,8
 jmp out_this_program ;这里如果不跳转的话就会执行到 Y_equal_zero 了，这明显是不行的，所要跳转
Y_equal_zero:
 xor eax,eax
 push eax
 push offset msg2
 call printf
 add esp,8
 jmp out_this_program ;这里如果不跳转的话就会执行到 Y_equal_zero 了，这明显是不行的，所要跳转
Y_equal_one:
 push 1
 push offset msg2
 call printf
 add esp,8 ;这里就不要再跳转了，直接执行到下面就行了
out_this_program:
 ret
 main endp
end start
```

下面是运行结果：

`C:\\w\\b\\ch4\\ch4-3>ch4-3`

`please input a int number`

0

`the Y value is 0`

`C:\\w\\b\\ch4\\ch4-3>ch4-3`

`please input a int number`

\-56

`the Y value is -1`

`C:\\w\\b\\ch4\\ch4-3>ch4-3`

`please input a int number`

56

`the Y value is 1`

像这种判断嵌套的地方，就是不停的比较，然后根据判断结果跳转到要执行的地方，下面我们来看看浮点数的判断,在比较完两个浮点数之后，我们紧接着得用 `fstsw` 将 FPU 的状态寄存器里面的比较结果存到一个 16 位的内存位置或者是个 16 位的寄存器，但最好是存在 `ax` 中，因为紧接着就得用 `sahf` 指令把浮点数比较的结果存到标志寄存器 EFLAGS 中，然后我们就可用 JCC 条件跳转指令来实现跳转了，具体的化来看看例子：

#### 例子 ch4-4

这个一元二次方程怎么求解，大家应该很熟悉了吧，就不多说了，下面来看看怎么实现浮点数的比较，代码如下：

``` 
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
zero dq 0.0000001
four dw 4
msg1 db "please input three numbers",0ah,0dh,0
msg2 db "%f,%f,%f",0
msg3 db "The equation is not a quadratic" ,0ah,0dh,0
msg4 db "The equation has two cpmplex roots: %8.4f+%8.4fi",0ah,0dh,0
msg5 db "The equation has two cpmplex roots: %8.4f-%8.4fi",0ah,0dh,0
msg6 db "The equation has two equal roots: %8.4f",0ah,0dh,0
msg7 db "The equation has two real roots: %8.4f,%8.4f",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local a:dword
 local b:dword
 local c1:dword
 local d:dword
 local disc:dword
 local x1:qword
 local x2:qword
 local realpart:qword
 local imagpart:qword
 local x:qword ;一些局部变量定义
 push offset msg1
 call printf
 add esp,4 ;打印提示消息
 lea eax,c1
 push eax
 lea eax,b
 push eax
 lea eax,a
 push eax
 push offset msg2
 call scanf
 add esp,12 ;调用 scanf 函数得到方程的三个系数
 fld a
 fabs
 fcom zero ;这里就是浮点数的比较和判断了，如果 a 位零
 fnstsw ax
 sahf
 jb is_not_quadratic ;就跳转到打印消息的代码处然后退出本程序
 fld b
 fmul b
 fild four
 fmul a
 fmul c1
 fsubp st(1),st
 fst disc ;到这里就是计算 b*b-4ac
 fabs
 fcom zero ;将 b*b-4ac 与零比较，
 fnstsw ax
 sahf
 jbe two_equal_roots ;等于零，就跳转到计算有两个相等的实根处
 fld disc
 fcom zero
 fnstsw ax
 sahf
 ja two_roots ;大于零，就跳转到计算有两个不相等的实根处
 shr four,1
 fild four
 fmul a
 fld b
 fchs
 fdivp st(1),st
 fstp realpart
 fild four
 fmul a
 fld disc
 fchs
 fsqrt
 fdivp st(1),st
 fstp imagpart ;这部分人是计算有两个复数根的地方
 push dword ptr imagpart+4
 push dword ptr imagpart+8
 push dword ptr realpart+4
 push dword ptr realpart+8
 push offset msg4
 call printf
 add esp,20
 push dword ptr imagpart+4
 push dword ptr imagpart+8
 push dword ptr realpart+4
 push dword ptr realpart+8
 push offset msg5
 call printf
 add esp,20 ;到这里是打印两个复数根的代码处
 jmp out_this_program
is_not_quadratic:
 push offset msg3
 call printf
 add esp,4
 jmp out_this_program ;这一段代码是打印不是二次方程的消息
two_equal_roots:
 shr four,1
 fild four
 fmul a
 fld b
 fchs
 fdivp st(1),st
 fstp x
 push dword ptr x+4
 push dword ptr x+8
 push offset msg6
 call printf
 add esp,12
 jmp out_this_program ;这一段是计算和打印有两个相等的实数根
two_roots:
 shr four,1
 fild four
 fmul a
 fld disc
 fsqrt
 fsub b
 fdiv st,st(1)
 fstp x1
 fild four
 fmul a
 fld disc
 fsqrt
 fchs
 fsub b
 fdiv st,st(1)
 fstp x2
 push dword ptr x2+4
 push dword ptr x2+8
 push dword ptr x1+4
 push dword ptr x1+8
 push offset msg7
 call printf
 add esp,20 ;这一段是计算和打印两个不相等的实数根
out_this_program:
 ret
 main endp
end start
```

下面是运行结果：

`C:\\w\\b\\ch4\\ch4-4>ch4-4`

`please input three numbers`

1,2,1

`The equation has two equal roots: -1.0000`

`C:\\w\\b\\ch4\\ch4-4>ch4-4`

`please input three numbers`

2,6,1

`The equation has two real roots: -0.1771, -2.8229`

`C:\\w\\b\\ch4\\ch4-4>ch4-4`

`please input three numbers`

1,2,2

`The equation has two cpmplex roots: -1.0000+ 1.0000i`

`The equation has two cpmplex roots: -1.0000- 1.0000i`

从这里你可以看见浮点数的比较和整数的比较有很大的不同之处了吧？

下面我们来看看直接跳转表

直接跳转表类似于 c 语言里面的 `case` 语句，实现了程序的多分支跳转

先看例子，例子 `ch4-5`：给出一百分成绩，要求输出成绩等级‘A’，’B‘，C，D，E，90 分以上就输出’A‘，80~89 分为’B‘，70~79 为’C’,60~69 为’D‘，60 以下为’E‘。

首先我们把打印成绩等级的代码写成直接定址表，方便程序的判断和跳转，代码如下：

``` 
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
onehunderd dword 100.0
one dword 1.0
msg1 db "please input a score",0ah,0dh,0
msg2 db "the score leveal is %c",0ah,0dh,0
msg3 db "%d",0
.code
start:
 call main
 ret
main proc
 local score:dword
 local s:dword
 local p:dword
 local w:dword
 local d:dword
 local f:qword
 push dword ptr offset msg1
 call printf
 add esp,4 ;输出提示消息
 lea eax,score
 push eax
 push offset msg3
 call scanf
 add esp,8 ;得到输入的的成绩
 mov eax,score
 cmp eax,60
 jb less_60
 cmp eax,70
 jb less_70
 cmp eax,80
 jb less_80
 cmp eax,90
 jb less_90
 mov ebx,0
 jmp print_score
 less_60:
 mov ebx,16
 jmp print_score
 less_70:
 mov ebx,12
 jmp print_score
 less_80:
 mov ebx,8
 jmp print_score
 less_90:
 mov ebx,4 ;这一段根据输入的成绩来生成跳转表的序数
 print_score:
 jmp short set
 table dword print_a,print_b,print_c,print_d,print_e
 set :
 call dword ptr table[ebx] ;这一段就是跳转表，根据前面的比较结果来跳转到相应的代码处
 print_a:
 push 'A'
 push offset msg2
 call printf
 add esp,8
 ret
 print_b:
 push 'B'
 push offset msg2
 call printf
 add esp,8
```



`ret`

`print_c:`

`push` `'C'`

`push` `offset` `msg2`

`call` `printf`

`add` `esp,8`

`ret`

`print_d:`

`push` `'D'`

`push` `offset` `msg2`

`call` `printf`

`add` `esp,8`

`ret`

`print_e:`

`push` `'E'`

`push` `offset` `msg2`

`call` `printf`

`add` `esp,8`

`ret`

`out1:`

`ret`

`main` `endp`

`end` `start`

下面是运行结果：

`C:\\w\\b\\ch4\\ch4-5>ch4-5`

`please input a score`

99

`the score leveal is A`

`C:\\w\\b\\ch4\\ch4-5>ch4-5`

`please input a score`

88

`the score leveal is B`

`C:\\w\\b\\ch4\\ch4-5>ch4-5`

`please input a score`

77

`the score leveal is C`

`C:\\w\\b\\ch4\\ch4-5>ch4-5`

`please input a score`

66

`the score leveal is D`

`C:\\w\\b\\ch4\\ch4-5>ch4-5`

`please input a score`

55

`the score leveal is E`

`C:\\w\\b\\ch4\\ch4-5>`

5：循环

先看看简单的一重循环，例子 `ch5-2`：

例 `ch5-2`：求 `1+2+3+4`。。。。。。+100 的和

如果不用循环来算的话，就是先算 `1+2`，然后把结果再加到 `3`，直到加到 `100`，这里面计算部分都是重复的，既然是重复的，那我们就用循环来实现它，具体代码如下：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "The sum of 1+2+3...+100 is:%d",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local sum:dword
 local i:dword
 mov i,1
 mov sum,0 ;这部分是初始化循环变量和存和的变量
again: ;循环开始处
 cmp i,100 ;比较循环变量，不满足就跳出循环
 ja out_again
 mov eax,i
 add eax,sum
 mov sum,eax
 inc i
 jmp again ;这部分就是计算 1+2+3+。。。100
 out_again: ;这部分是循环结束的地方，打印结果
 push sum
 push offset msg1
 call printf
 add esp,8
 ret
 main endp
end start
```

运行结果：

`C:\\w\\b\\ch5\\ch5-2>ch5-2`

`The sum of 1+2+3...+100 is:5050`

总的来说，循环它有个循环初始值和结束值，先判断满足循环条件么，满足就进入循环体，不满足就退出循环。

例子 `ch5-1`：

首先我们看看怎么算某个数的阶乘，用数学公式表达就是 `t=t*1`，`t=t*2`…..`t=t*n`,

然后用个循环把这些乘积加起来就行了，我们用 `n` 作为循环控制的变量，它从 `1` 递增到 `20`，然后用个临时变量 `temp` 保存着每个数的阶乘值，然后把 `temp` 值加起来存在 `sum` 中就行了，代码如下：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db "The sum of 1!+2!+3!...+20! is:%e",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local sum:qword
 local temp:dword
 local n:dword
 mov n,1
 fldz
 fstp sum
 fld1
 fstp temp
 again:
 cmp n,20
 ja outagain
 fimul n
 fadd sum
 fst sum
 inc n
 jmp again
outagain:
 fstp sum
 push dword ptr sum+4
 push dword ptr sum+8
 push offset msg1
 call printf
 add esp,12
 ret
 main endp
end start
```

运行结果如下：

`C:\\w\\b\\ch5\\ch5-1>ch5-1`

`The sum of 1!+2!+3!...+20! is:-1.#QNAN0e+000`

呵呵，错的不行，唉，仔看看程序，实在没看出哪里错了，只好调试了，用 `cdb -2 C:\\w\\b\\ch5\\ch5-1\\ch5-1` 打开程序，然后用 `rm f` 打开我们想要看的寄存器，然后用 `g @$exentry` 跳到我们的这个程序，然后单步跟踪程序发现 `ch5_1!main+0x1d`:

`00401023 da4df0 fimul dword ptr [ebp-10h] ss:0023:0013ffac=00000001` 这句时，发现 `st0` 是个不定值，`st0= 0.183338657867652323710e+0834`，所以乘出来当然是错的哦，然后就不停的改，调试，最后终于正确了，结果如下：

`C:\\w\\b\\ch5\\ch5-1>ch5-1`

`The sum of 1!+2!+3!...+20! is:2.561326e+018`

最后正确的代码：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db "The sum of 1!+2!+3!...+20! is:%e",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local sum:qword
 local temp:dword
 local n:dword
 mov n,1 ;初始化循环变量 n
 fldz
 fstp sum ;初始化存和的变量
 fld1
 fstp temp ;初始化计算阶乘的变量
 again:
 cmp n,20
 ja outagain ;不满足循环条件就退出循环
 fld temp
 fild n
 fmulp st(1),st
 fst temp
 fld sum
 faddp st(1),st
 fstp sum
 inc n
 jmp again ;这一段就是计算 temp=temp*n，sum=sum+temp，就相当于算出了 1！+2！+3！。。。20！
outagain:
 push dword ptr sum+4
 push dword ptr sum+8
 push offset msg1
 call printf
 add esp,12
 ret ;打印结果
 main endp
end start
```

写浮点运算的时候，一定得弄清楚浮点寄存器的变化哦，要不然都不直到程序是错在哪里了，只有多调试，在调试的时候看每个浮点指令对浮点寄存器的影响，然后慢慢的就有感觉了的哦，就很清楚要用哪些浮点指令，脑袋就很清楚了，不再是浆糊啦。

下面我们再来看看多重循环：

多重循环就是循环里面有循环，先看个例子：

呵呵，我们先看这个图形的上半部分，一共有四行，每行的※号是呈 `1,3,5,7` 递增的，所以我们先用 `3` 个循环，外循环控制总共打印多少行，第二 个循环控制每行打印多少个空格 第三个循环控制打印多少个\*号，然后同理再打印出下半部分就可以了，代码如下：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
scanf proto c:VARARG
.data
msg1 db '*',0
msg2 db ' ',0
msg3 db 0ah,0dh
.code
start:
 call main
 ret
main proc
 local i:dword
 local j:dword
 local k:dword
 mov i,0
for1:
 cmp i,3
 ja outfor1 ;第一层循环判断，不满足就退出
 mov j,0
print_space:
 mov eax,2
 sub eax,i
 cmp j,eax
 jg outfor2
 push offset msg2 ;空格数是 2-i
 call printf
 add esp,4
 inc j
 jmp print_space ;这一段是打印空格
outfor2:
 mov k,0
print_x:
 mov eax,i
 imul eax,eax,2
 cmp k,eax
 ja print_crlf
 push offset msg1 ;\*号数是 2*i
 call printf
 add esp,4
 inc k
 jmp print_x ;这一段是打印\*号
print_crlf:
 push offset msg3
 call printf
 add esp,4
 inc i ;这一段是打印回车换行
 jmp for1 ;到这里就跳转到第一层循环
outfor1:
 mov i,0
for11:
 cmp i,2
 ja outfor11
 mov j,0
print_spcae1:
 mov eax,i
 cmp j,eax
 jg outfor22
 push offset msg2
 call printf
 add esp,4
 inc j ;空格数是 i
 jmp print_spcae1
outfor22:
 mov k,0
printf_x1:
 mov eax,i
 imul eax,eax,2
 mov ebx,4
 sub ebx,eax
 cmp k,ebx
 ja ptint_crlf1
 push offset msg1
 call printf
 add esp,4
 inc k ;※号数是 4-2*i
 jmp printf_x1
ptint_crlf1:
 push offset msg3
 call printf
 add esp,4
 inc i
 jmp for11
outfor11:
 ret
 main endp
end start
```

运行结果：

`C:\\w\\b\\ch5\\ch5-3>ch5-3`

` *`
` ***`
` *****`
` *******`
` *****`
` ***`
` *`

这里主要弄清，需要及个循环，及每个循环要打印东西的数目，理清了这些就比较容易写程序了。

6.数组和指针

数组其实就是一片连续的存放着同类型数据的内存，指针就是存放着某种数据类型的内存地址，它们之间其实在汇编层是差不多的，看看例子：

例 `ch6-1`：用下标法输出数组的全部元素

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "%d",0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword ;定义了一个循环变量，就是数组的下标
 local a[10]:byte ;定义了一个 10 个字节的数组
 local b[10]:dword ;定义了一个 10 个双字的数组
 local p :byte ;定义了一个字节变量，这里可以把它看成是字节指针
 local p1 :dword ;定义了一个双字变量，这里可以把它看成是双字指针
 mov i,0
 mov ebx,i
again:
 cmp i,9
 mov eax,i
 ja out_again
 mov byte ptr a[ebx],al ;这句就是用下标访问数组了，和下面的比较下，想想为什么
 inc i
 jmp again ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9
out_again:
 mov i,0
 mov ebx,i
 again1:
 cmp i,9
 mov eax,i
 ja print_arry
 mov b[ebx*4],eax ;根据下标 i 访问数组，和上面那句比较下看看
 inc i
 jmp out_again
print_arry:
 mov i,0
 mov ebx,i
loop_print_arry:
 cmp i,9
 ja out_print_arry
 mov al,byte ptr a[ebx]
 movzx eax,al
 push eax
 push offset msg1
 call printf
 add esp,8
 jmp loop_print_arry ;循环打印字节数组
out_print_arry:
 mov i,0
 mov ebx,i
loop_print_arry1:
 cmp i,9
 ja out_this_program
 mov eax,dword ptr b[ebx*4]
 push eax
 push offset msg1
 call printf
 add esp,8
 jmp loop_print_arry1 ;循环打印双字节数组
out_this_program:
 ret
 main endp
end start
```

编译运行一看，程序没能运行正常，再看看自己写的那些代码，又看不出什么问题，那又只好在调试器里面找问题了，经调试发现这句 `mov ebx,i`

应该放到循环语句的里面，然后再把这句 `out_ again:`

`mov i,0`

里面的 `mov i,0` 放到 `out_ again:`的前面，然后在打印每个数组元素的语句后面加上 `inc i`，然后编译运行，终于正确了，下面是运行结果：

`C:\\w\\b\\ch6\\ch6-1>ch6-1`

0123456789

0123456789

第-行输出是字节数组的，第二行输出是双字节数组的

最后经调试发现错误后改正的代码：

```asm
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
```



`printf` `proto` `c:VARARG`

.data

`msg1` `db` `"%d",0`

`msg2` `db` `0ah,0dh,0`

.code

`start:`  

  `call` `main`  
  `ret`  

`main` `proc`  

  `local` `i:dword` ;定义了一个循环变量，就是数组的下标  
  `local` `a[10]:byte` ;定义了一个 10 个字节的数组  
  `local` `b[10]:dword` ;定义了一个 10 个双字的数组  
  `local` `p` `:byte` ;定义了一个字节变量，这里可以把它看成是字节指针  
  `local` `p1` `:dword` ;定义了一个双字变量，这里可以把它看成是双字指针  
  `mov` `i,0`  
`again:`   
  `cmp` `i,9`  
  `mov` `ebx,i`  
  `mov` `eax,i`  
  `ja` `out_again`  
  `mov` `byte ptr a[ebx],al` ;这句就是用下标访问数组了，和下面的比较下，想想为什么  
  `inc` `i`  
  `jmp` `again` ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9  
`out_again:`  
  `mov` `i,0`  
`again1:`  
  `cmp` `i,9`  
  `mov` `eax,i`  
  `mov` `ebx,i`  
  `ja` `print_arry`  
  `mov` `b[ebx*4],eax` ;根据下标 i 访问数组，和上面那句比较下看看  
  `inc` `i`  
  `jmp` `again1`  
`print_arry:`  
  `mov` `i,0`  
`loop_print_arry:`  
  `cmp` `i,9`  
  `ja` `out_print_arry`  
  `mov` `ebx,i`  
  `mov` `al,byte ptr a[ebx]`  
  `movzx` `eax,al`  
  `push` `eax`  
  `push` `offset msg1`  
  `call` `printf`  
  `add` `esp,8`  
  `inc` `i`  
  `jmp` `loop_print_arry` ;循环打印字节数组  
`out_print_arry:`  
  `push` `offset msg2`  
  `call` `printf`  
  `add` `esp,4` ;这里打印回车换行  
  `mov` `i,0`  
`loop_print_arry1:`  
  `cmp` `i,9`  
  `mov` `ebx,i`  
  `ja` `out_this_program`  
  `mov` `eax,dword ptr b[ebx*4]`  
  `push` `eax`  
  `push` `offset msg1`  
  `call` `printf`  
  `add` `esp,8`  
  `inc` `i`  
  `jmp` `loop_print_arry1` ;循环打印双字节数组  
`out_this_program:`  
  `push` `offset msg2`  
  `call` `printf`  
  `add` `esp,4` ;这里打印回车换行  
  `ret`  
  `main` `endp`  

`end` `start`  

呵呵，你可以和前面的代码比较看看，如果你看不出第一个程序的问题，最好的是你把第一个代码编译运行，自己调试着看看，保你收获多多。

下面看看用指针试试：看例子 `ch6-2`：

### 例 ch6-2：用指针输出数组的全部元素

代码如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "%d",0
msg2 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword ;定义了一个循环变量，就是数组的下标
 local a[10]:byte ;定义了一个 10 个字节的数组
 local b[10]:dword ;定义了一个 10 个双字的数组
 local p :dword ;定义了一个双字变量，这里可以把它看成是字节指针,至于怎么会这样，请看下面代码
 local p1 :dword ;定义了一个双字变量，这里可以把它看成是双字指针
 mov i,0
again:
 cmp i,9
 mov ebx,i
 mov eax,i
 ja out_again
 mov byte ptr a[ebx],al ;这句就是用下标访问数组了，和下面的比较下，想想为什么
 inc i
 jmp again ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9
out_again:
 mov i,0
again1:
 cmp i,9
 mov eax,i
 mov ebx,i
 ja print_arry
 mov b[ebx*4],eax ;根据下标 i 访问数组，和上面那句比较下看看
 inc i
 jmp again1
print_arry:
 lea eax,a
 mov p,eax
 add eax,10
loop_print_arry:
 cmp p,eax
 ja out_print_arry
 mov bl,byte ptr p ;这里就是把双字节指针当成字节用
 movzx ebx,bl
 push ebx
 push offset msg1
 call printf
 add esp,8
 inc p
 jmp loop_print_arry ;循环打印字节数组
out_print_arry:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 lea eax,b
 mov p1,eax
 add eax,40
loop_print_arry1:
 cmp p1,eax
 ja out_this_program
 mov ebx,dword ptr p
 push ebx
 push offset msg1
 call printf
 add esp,8
 add p,4
 jmp loop_print_arry1 ;循环打印双字节数组
out_this_program:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 ret
 main endp
end start
```

但是我们得到了很奇怪的结果，没办法啦，只有调试咯，经调试发现 `mov bl,byte ptr p` 这句访问数组元素根本访问不到，因为 `ch6_2!main+0x3f`:

```
00401045 8d45f2 lea eax,[ebp-0Eh]
0:000>
eax=0013ffae ebx=0000000a ecx=0013ffb0 edx=7c90e514 esi=00000000 edi=00000000
eip=00401048 esp=0013ff7c ebp=0013ffbc iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
ch6_2!main+0x42:
00401048 8945c4 mov dword ptr [ebp-3Ch],eax ss:0023:0013ff80=852c3fe0
0:000>
eax=0013ffae ebx=0000000a ecx=0013ffb0 edx=7c90e514 esi=00000000 edi=00000000
eip=0040104b esp=0013ff7c ebp=0013ffbc iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000202
```

因为 `a` 的首地址是 `0013ffae`，当执行这句 `mov bl,byte ptr p` 时，你可以看到调试器是这样的：

```
00401048 8945c4 mov dword ptr [ebp-3Ch],eax ss:0023:0013ff80=852c3fe0
```

哈哈，它访问了 `0013ff80`里面的元素了，唉，怎么办呢？

我想只能用第一个那种寄存器间接寻址方式了，呵呵，又回到第一种访问方式了。

发现第二个错误就是这个程序调用了 `printf` 之后，就改变了 `eax`，`ecx`，`ebx` 的值，反正就是我们这里用到的寄存器都被它改了。然后修改后的程序如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
msg1 db "%d",0
msg2 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword ;定义了一个循环变量，就是数组的下标
 local a[10]:byte ;定义了一个 10 个字节的数组
 local b[10]:dword ;定义了一个 10 个双字的数组
 local p :dword ;定义了一个双字变量，这里可以把它看成是字节指针,至于怎么会这样，请看下面代码
 local p1 :dword ;定义了一个双字变量，这里可以把它看成是双字指针
 mov i,0
again:
 cmp i,9
 mov ebx,i
 mov eax,i
 ja out_again
 mov byte ptr a[ebx],al ;这句就是用下标访问数组了，和下面的比较下，想想为什么
 inc i
 jmp again ;这一段代码主要是往 a 数组里面依次存入 0，1，2，。。。9
out_again:
 mov i,0
again1:
 cmp i,9
 mov eax,i
 mov ebx,i
 ja print_arry
 mov b[ebx*4],eax ;根据下标 i 访问数组，和上面那句比较下看看
 inc i
 jmp again1
print_arry:
 lea eax,a
 mov p,eax
 add eax,10
loop_print_arry:
 push eax
 cmp p,eax
 jae out_print_arry
 mov ecx,p
 mov bl,byte ptr [ecx] ;这里就是把双字节指针当成字节用
 movzx ebx,bl
 push ebx
 push offset msg1
 call printf
 add esp,8
 inc p
 pop eax
 jmp loop_print_arry ;循环打印字节数组
out_print_arry:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 lea eax,b
 mov p1,eax
 add eax,40
loop_print_arry1:
 push eax ;由于下面的 ptintf 会改变我们这里要用到的寄存器，我用他们的时候就先保存
 cmp p1,eax
 jae out_this_program
 mov ecx,p1
 mov ebx,dword ptr [ecx]
 push ebx
 push offset msg1
 call printf
 add esp,8
 add p1,4
 pop eax ;弹出保存的 eax 值
 jmp loop_print_arry1 ;循环打印双字节数组
out_this_program:
 push offset msg2
 call printf
 add esp,4 ;这里打印回车换行
 ret
 main endp
end start
```

下面是运行结果：

```
C:\w\b\ch6\ch6-2>ch6-2
0123456789
0123456789
```

呵呵，终于是对的了。

下面再来看看多重数组：

先看个最简单的例子 `ch6-3`：输出一个二维数组，例如：

然后运行完程序在屏幕上打印如下：

然后写好的程序如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
a dd 1,2,3
 dd 4,5,6 ;定义了一个两行三列的整数数组
msg1 db "array a :",0ah,0dh,0
msg2 db "%8d",0
msg3 db 0ah,0dh,0
.code
start:
 call main
 ret
main proc
 local i:dword
 local j:dword ;定义了两个循环变量
 push offset msg1
 call printf
 add esp,4 ;打印消息
 mov i,0
 mov j,0 ;初始化两个循环变量
loop_1:
 cmp i,1
 ja out_loop_1
loop_2:
 cmp j,2
 ja out_loop_2
 mov eax,3 ;首先这个数组每行有三个元素，所以要用当前的行数乘以 3
 mul i
 add eax,j ;然后再加上当前的列数，由于这个数组数四字节的，所以还要
 push a[eax*4] ;乘以 4，你才能正确的访问每个数组元素
 push offset msg2
 call printf
 add esp,8 ;内循环就是依次打印每行的数组元素
 inc j
 jmp loop_2
out_loop_2:
 inc i
 push offset msg3
 call printf
 add esp,4
 jmp loop_1 ;这里是跳转到外循环，外循环是控制打印多少行
out_loop_1:
 ret
 main endp
end start
```

但是结果是这样的：

```
C:\w\b\ch6\ch6-3>ch6-3
array a :
       1       2       3
```

很明显是错的哦，你能在上面的那段程序里看出哪里错了么？呵呵，我反正没看出什么名堂来，经过调试之后发现，再进入第二次循环之前忘记把循环变量赋初值了！！！那我就在这里改了下：

`ja out_loop_1`

`mov j,0`

再调试运行，结果如下：

```
C:\w\b\ch6\ch6-3>ch6-3
array a :
       1       2       3
       4       5       6
```

呵呵，是正确的，最后正确的代码如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib msvcrt.lib
printf proto c:VARARG
.data
a dd 1,2,3
 dd 4,5,6 ;定义了一个两行三列的整数数组
```



## 示例代码

```asm
msg1  db  "array a :",0ah,0dh,0

msg2  db      "%8d",0

msg3  db  0ah,0dh,0

.code

start:

  call  main

  ret

main  proc

  local  i:dword

  local  j:dword      ;定义了两个循环变量

  push  offset msg1

  call  printf

  add  esp,4      ;打印消息

  mov  i,0

          ;初始化两个循环变量

loop_1:

  cmp  i,1

  ja  out_loop_1

  mov  j,0      ;呵呵，就是错在这里了，再进入第二次循环之前忘记把循环变量赋初值了！！！

loop_2:

  cmp  j,2

  ja  out_loop_2

  mov  eax,3      ;首先这个数组每行有三个元素，所以要用当前的行数乘以 3

  mul  i

  add  eax,j      ;然后再加上当前的列数，由于这个数组数四字节的，所以还要

  push  a[eax*4]    ;乘以 4，你才能正确的访问每个数组元素

  push  offset msg2

  call  printf

  add  esp,8      ;内循环就是依次打印每行的数组元素

  inc  j

  jmp  loop_2

out_loop_2:

  inc  i

  push  offset msg3

  call  printf

  add  esp,4

  jmp  loop_1      ;这里是跳转到外循环，外循环是控制打印多少行

out_loop_1:

  ret

  main  endp

end  start
```

这里再回顾下怎么访问数组中的元素：

### 怎么访问一维数组中的元素

这个简单，就是用序数乘以元素的字节大小再加上数组基地址就可以了，比如在例子 `ch6-2` 里面的这句 `mov b[ebx*4],eax`，就把 `eax` 里面的值存到了以 `ebx` 为序数（就是在数组中第几个中的几）乘以 `4`，因为这个数组的每个元素是 `4` 字节的，再加上数组基地址 `b`，就可以了。

### 怎么访问二维数组中的元素

这个复杂点，就是先用每行数组元素的个数（是个常量）乘以当前行的序数再加上当前列的序数，再把这个值乘以元素的字节大小就可以了，比如在 `ch6-3` 里面的这一段代码：

```asm
mov eax,3      ;首先这个数组每行有三个元素，所以要用当前的行数乘以 3
mul i
add eax,j      ;然后再加上当前的列数，由于这个数组数四字节的，所以还要
push  a[eax*4]
```

到了最后一句就是把数组 `a` 里面的第 `i` 行的第 `j` 个元素入栈，乘以 `3` 是因为这个二维数组是每行 `3` 个元素，好，下面再看个例子

## 例子 ch6-4

为了求出其中最大的那个元素的值，我们的用循环遍历整个数组，然后就可以找出来了，代码如下：

```asm
.386

.model flat,stdcall

option casemap:NONE

includelib  msvcrt.lib

printf  proto   c:VARARG

.data

a  dd  1,2,3,4

  dd  9,8,7,6    ;定义了一个 3*4 的二维数组

  dd  -10,10,-5,2

msg1  db  "the max value of array a is: %d,it in row %d column %d of array a",0ah,0dh,0

.code

start:

  call  main

  ret

main  proc

  local  i:  dword

  local  j:  dword

  local  max:  dword

  local  row:  dword

  local  column: dword  ;局部变量定义

  mov  eax,a[0]

  mov  max,eax    ;这里相当于 max=a【0，0】

  mov  i,0

loop_1:

  cmp  i,2

  ja  out_loop_1

  mov  j,0

loop_2:

  cmp  j,3

  ja  out_loop_2

  mov  eax,4

  mul  i

  add  eax,j

  mov  ebx,a[eax*4]  ;这里就是 ebx 等于当前 a【i，j】

  cmp  ebx,max    ;这里是用 a【i，j】和 max 做比较

  jg  change_max  ;如果 a【i，j】大于 max 就将他们两个交换

  inc  j

  jmp  loop_2

change_max:

  mov  max,ebx

  mov  eax,i

  mov  row,eax

  mov  eax,j

  mov  column,eax

  inc  j

  jmp  loop_2

out_loop_2:

  inc  i

  jmp  loop_1

out_loop_1:

  push  column

  push  row

  push  max

  push  offset msg1

  call  printf

  add  esp,12    ;在两个循环外打印出题目的要求

  ret

  main  endp

end  start
```

下面是运行结果：

```
C:\w\b\ch6\ch6-4>ch6-4
the max value of array a is: 10,it in row 2 column 1 of array a
```

上传的附件

| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || 汇编语言学习笔记.doc |
