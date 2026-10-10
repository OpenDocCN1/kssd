
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
