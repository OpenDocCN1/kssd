
## 1. 浮点数与整数转换

### 浮点数转换成整数

首先我们还是借助了一个64位的局部变量，先把浮点数装入浮点寄存器，然后用转换指令变成整数再存入一个局部变量就行了。

### 整数转换成浮点数

```
fild f2
fstp f
```

先把整数用转换指令装入浮点寄存器，然后把浮点数存到一个局部变量就可以了。

我在后面会详细说明浮点数的运算和浮点寄存器的。

## 2. 字符与字符串常量

怎么定义它们？它们是以什么形式存在计算机中？

首先我们怎么在汇编中定义它们呢？先看看例子`ch2-5`：

```
.386
.model flat,stdcall
option casemap:none
includelib        msvcrt.lib
printf        proto C :VARARG
.data
str1        db        'this is a string test',0ah,0dh,0
str2        db        "this is a string test",0ah,0dh,0
.code
start:
        call        main
        ret
main proc
        mov        eax,        offset str1
        push        offset str2
        call        printf
        add        esp,4
        push        offset str1
        call        printf
        add        esp,4
        ret
        main        endp
end        start
```

程序就是在屏幕上打印两行消息，下面是运行结果：

然后我们用`cdb`调试器看看定义的那两个字符串变量在内存中到底是怎么样的：

嗯，它们是以 ASCII 码的形式存在的。

其他的数据类型我会在下面的各个章节中随着编程的算法和调试一起讲解。

## 3. 顺序程序设计

汇编语言的顺序编程比较好理解，就是在编程的时候没有跳转，没有循环，看看例子`ch3-1`：

### 例 ch3-1：输入三角形的边长，求三角形的面积

我假设输入的三边长都是能构成三角形的，求三角形面积的公式为`area = sqrt(s(s-a)(s-b)(s-c))`。

`s = (a + b + c) / 2`。

这里要用到浮点指令，那就先回顾下浮点指令的用法：这里要用加减乘除和平方根五种指令。由于Intel的浮点数据寄存器是一种堆栈结构，我们要记住这一点。

#### 数据传送指令：`fld`、`fild`、`fst`、`fstp`

`fld 源操作数`，源操作数可以是浮点寄存器和内存。这个指令主要是把源操作数压入浮点寄存器堆栈（其实就是`st0`）。如果源操作数是整数，那就用`fild`。

`fst`和`fstp`是把`st(0)`浮点寄存器中的数弹出到目的操作数中，目的操作数可以为浮点寄存器和内存。

#### 加减法指令：`fadd`、`faddp`、`fsub`、`fsubp`

第一种形式：`fadd 目的操作数，源操作数`。其中目的操作数、源操作数可以为浮点寄存器和内存。

第二种形式：`fadd 源操作数`，我本人比较喜欢这种，它不会把我脑海里的浮点寄存器的顺序弄乱，这种形式的源操作数只能是内存。减法指令同加法指令，就不多说了。

#### 乘除法指令：`fmul`、`fdiv`

浮点的乘除法是不区分有符号和无符号数的，它们也有两种形式：

`fmul 目的操作数，源操作数`和`fmul 源操作数`这两种形式。第一种目的操作数和源操作数可以为浮点寄存器和内存，但第二种源操作数只能是内存。

#### 平方根指令：`fsqrt`

这个指令就一种形式，就是`fsqrt`，就是把第0个浮点寄存器`st0`的值变成平方根值然后存在`st0`中。

再回到例题中，我们应该先算`s`值，然后再算平方根下面的值然后求平方根就行了。

`s`值是三边长除以2，转换成浮点指令就是：

```
fld  a        ; 先把 a 值放到浮点寄存器 st0 中
fadd b        ; 这个就是 a+b，结果存在 st0 中
fadd c        ; 同上，结果存在 st0 中
fdiv two      ; 现在算出了三边长的和，再除以 2，结果在 st0 中
fstp s        ; 最后把 st0 里面的值存到 s 中，就把 s 值算出来了
```

再来算根号下面的值，这里有乘法和减法，我们先算减法：

```
fld  s
fsub a        ; 这个就是 s-a，结果在 st0 里

fld  s
fsub b        ; 这个结果还是在 st0 里，但是上面那个 s-a 已经被推到 st(1) 这个浮点堆栈里了

fld  s
fsub c        ; 这个结果还是在 st0 里
```

那么`s-a`被推到`st(2)`了，`s-b`被推到`st(1)`了，`s-c`的值在`st(0)`中，这里千万不能记乱了。下一步算乘法，先算`s*(s-c)`：

```
fmul s            ; 结果在 st0 里
fmul st(0), st(1) ; 这一步是算 s*(s-a)*(s-b)
fmul st(0), st(2) ; 最后算 s*(s-a)*(s-b)*(s-c)
```

然后把`s*(s-a)*(s-b)*(s-c)`的结果再求平方根：

```
fsqrt
```

再把这个平方根的值弹出到`area`中。

这是按照上面的想法写出的程序：

```
.386
.model flat,stdcall
option casemap:none
includelib        msvcrt.lib
printf        proto C :VARARG
scanf        proto C :VARARG
.data
msg1        db        "%f,%f,%f",0
msg2        db        "area=%7.2f",0ah,0dh,0
msg3        db        "please input three floating numbers",0ah,0dh,0
two        real4        2.0
.code
start:
        call        main
        ret
main proc
        local        a:real4
        local        b:real4
        local        c1:real4
        local        s:real4
        local        area:real8
        push        offset msg3
        call        printf
        add        esp,4
        lea        eax,c1
        push        eax
        lea        eax,b
        push        eax
        lea        eax,a
        push        eax
        push        offset msg1
        call        scanf
        add        esp,16
        fld        a
        fadd        b
        fadd        c1
        fdiv        two
        fstp        s
        fsub        a
        fsub        b
        fsub        c1
        fmul        s
        fmul        st,st(1)
        fmul        st,st(2)
        fsqrt
        fstp        area
        push        dword ptr area+8
        push        dword ptr area+4
        push        offset msg2
        call        printf
        add        esp,12
        ret
        main        endp
end        start
```

编译运行：下面是结果：

但是出乎意料哦，错了，怎么办呢？你发现什么问题了没？那只有调试看看了，用`cdb -2 C:\w\b\ch3\ch3-1\ch3-1`打开程序然后逐步调试看看，当跟踪到下面这句时就发现有问题了：

就是`fstp s`，也就是`00401041 d95df0          fstp    dword ptr [ebp-10h]  ss:0023:0013ffac=a9e45d08`，然后发现`st`中的值被弹走了，但紧接着又用`st(0)`用于下一步进行计算了。先退出调试，改动下再运行看看对不对，但结果还是错的：

那还得继续调试了，发现我对`fsub`指令理解错了，原来这个指令执行完之后它不把结果推到下一个浮点堆栈：

那只能用`fsubp`了，改动如下：

```
fld        a
fsubp        st(1),st

fld        s
fld        b
fsubp        st(1),st

fld        s
fld        c1
fsubp        st(1),st        ;然后改成这种了
```

但结果还是错的，一看到这句

```
push        dword ptr area+8
push        dword ptr area+4
```

想一下"高低原则"，改成

```
push        dword ptr area+4
push        dword ptr area+8
```

编译运行，哈哈，OK了：

最后正确的代码：

```
.386
.model flat,stdcall
option casemap:none
includelib        msvcrt.lib
printf        proto C :VARARG
scanf        proto C :VARARG
.data
msg1        db        "%f,%f,%f",0
msg2        db        "area=%7.2f",0ah,0dh,0
msg3        db        "please input three floating numbers",0ah,0dh,0
two        real4        2.0
.code
start:
        call        main
        ret
main proc
        local        a:real4
        local        b:real4
        local        c1:real4
        local        s:real4
        local        area:real8        ;这里是变量定义
        push        offset msg3
        call        printf
        add        esp,4                ;这里是提示输入三个边长
        lea        eax,c1
        push        eax
        lea        eax,b
        push        eax
        lea        eax,a
        push        eax
        push        offset msg1
        call        scanf
        add        esp,16                ;这里是调用库函数scanf
        fld        a
        fadd        b
        fadd        c1
        fdiv        two
        ;fstp        s                这一句其实是有问题的，你可以调试看看
        fst        s                ;改成这句
        ;fsub        a
        ;fsub        b
        ;fsub        c1                ;经调试发现这三句都是错的
        fld        a
        fsubp        st(1),st
        fld        s
        fld        b
        fsubp        st(1),st
        fld        s
        fld        c1
        fsubp        st(1),st        ;然后改成这种了
        fmul        s
        fmul        st,st(1)
        fmul        st,st(2)
        fsqrt
        fstp        qword ptr area                ;这一段是计算area值,我的文档上有详细解释
        push        dword ptr area+4
        push        dword ptr area+8
        push        offset msg2
        call        printf
        add        esp,12                ;打印结果
        ret
        main        endp
end        start
```

呵呵，定义的时候用了`c1`，因为`c`是汇编里面的关键字，所以我就改了下。你可能会说，你怎么不优化一下呢？呵呵，先把程序写对了，把指令理解透了再去优化也不晚。

### 练习 ch2-2

求下面`x1`和`x2`的值：

`x1 = (b + sqrt(b*b - 4ac)) / 2a`

`x2 = (b - sqrt(b*b - 4ac)) / 2a`

`a`、`b`、`c`由键盘输入。

提示下正确运行结果：

`a = 1`，`b = 3`，`c = 2`

`x1 = -1.00`

`x2 = -2.00`

## 4. 选择分支结构程序设计

前面我们看到的程序都是从上到下这样执行的，程序的执行流程没有跳转和循环。CPU为我们实现程序的跳转提供了硬件环境和一些指令。首先CPU它有个处理器状态寄存器，跳转指令就是无条件跳转`jmp`和有条件跳转指令`jcc`（里面包含了很多种形式的）。总的来说就是你在你写的程序里面用跳转指令让CPU去改变程序的执行流程，CPU会根据那个状态寄存器的一个或多个值的改变来执行程序。我们先来看个例子：

### 例 4.1：输入3个整数`a`、`b`、`c`，请输出其中最大的数

我们只要依次把这三个数比较一次，在每次比较中，把那个大值继续和下一个数来做比较，然后再输出那个最大值就行了。代码如下：

```
.386
.model flat,stdcall
option casemap:NONE
includelib        msvcrt.lib
printf        proto   c:VARARG
scanf        proto   c:VARARG
.data
msg1        db        "please input three int numbers",0ah,0dh,0
```

