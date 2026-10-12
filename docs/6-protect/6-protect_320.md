
# 换个角度看多态 - Polymorphic Games

> 原文：[`www.pediy.com/kssd/pediy10/75760.html`](https://www.pediy.com/kssd/pediy10/75760.html)

对多态技术感兴趣朋友，可能多是钟情于代码混淆这块，无意中发现了这篇文章，可能大侠们早已看过，但偶觉得写的比较有趣，他从另一个角度揭示了一种代码变换技术。于是做了一个简单的翻译（水平有限，大家凑合看喽），原文链接：

`http://mirror.sweon.net/z0mbie/pgames.txt`

可能我们写多态时，更多的是考虑等效指令替换，指令乱序，随机指令生成，随机寄存器等这些因素，这篇文章倒是提了另外的思路，所以可以学习一下了，虽然不一定马上加入自己写的东东里，但留个思路日后琢磨吗，不废话了，下面就是。

## Polymorphic Games

### part 1 -- conditional execution w/o jxx

让我们考虑一些 `C` to `asm` 的转换示例吧。

我们设置 `a` 是 `eax`，`b` 是 `ebx`，`c` 是 `ecx`，`d` 是 `edx`，“condition” 是一个结果或者是些二进制的对比值，例如一个位，`0` 或者 `1`

#### 例子 1

首先，我们要知道下面的语句如何反汇编:

```
c = condition ? -1 : 0;
```

这个看起来像:

```
; CF <-- condition
sbb ecx, ecx
```

或者

```
; ecx.high_bit <-- condition
sar ecx, 31
```

#### 例子 2

让我们看一个更真实的情况:

```
c = a < b ? -1 : 0;
```

或者从这个例子:

```
if (a < b) c = -1; else c = 0;
```

因此，我们必选这样:

```
cmp eax, ebx
sbb ecx, ecx
```

或者

```
mov ecx, eax
sub ecx, ebx
sar ecx, 31
```

#### 例子 3

一些更复杂点的代码, 例如 `a=MIN(a,b)` function:

```
if (a > b) a = b;
```

这相当于:

```
a = a > b ? b : a;
```

也相当于:

```
a = a + ((a > b ? -1 : 0) & (b - a))
```

也相当于:

```
a += ((b - a) < 0 ? -1 : 0) & (b - a)
```

其中（例子 1 和例子 2）也可以用如下的方式来表达：

```
sub     ebx, eax
sbb     ecx, ecx
and     ecx, ebx
add     eax, ecx
```

#### 例子 4

绝对值, `abs()` function:

```
a = abs(a)
if (a < 0) a = -a;
a = a < 0 ? -a : a;
```

但是，有了 `NEG` 操作，它如同 `NOT` + `INC` (我考虑典型的 `x86 asm`)

我可以通过如下的方式做：

```
a = (a ^ (a<0?-1:0)) - (a<0?-1:0)
```

反汇编看起来是这样:

```
mov     edx, eax
sar     edx, 31
xor     eax, edx
sub     eax, edx
```

#### example 5

一些 `C` 的代码：

```
if (a != 0)
  a = b;
else
  a = c;

a = a ? b : c;

a = ((a != 0 ? -1 : 0) & b) | ((a != 0 ? 0 : -1) & c);
```

因此，它看起来像这样：

```
cmp     eax, 1
sbb     eax, eax
and     ecx, eax
xor     eax, -1
and     eax, ebx
or      eax, ecx
```

正如你所看到的，一些基本的操作码都可以被重新编码，除了条件跳转语句，

即便我们有些条件语句，那其实也可以避免使用 `jxx` 指令来达到目的.

例如：

```
if (c) a >>= b
```

可以被转成

```
a >>= (c ? b : 0),
```

这相当于例 5 + `shr`

等等.

但是，如果我们想调用子程序呢? okey，这有解决的办法：

```
func_addr = condition ? real_func : fake_func;
func_addr(arg1, arg2, ...);
int fake_func() { return 0; }
```

但是，如果我想初始化一些变量呢？让我们先这样做：

```
var_addr = condition ? &real_var : &fake_var;
*var_addr = ...;
```

因此，一下情况：

```
if (condition)
{
  statement1;
  statement2;
  statementN;
}
```

你可以转换成：

```
if (condition) statement1;
if (condition) statement2;
if (condition) statementN;
```

每一行都可以单独编码，使用的技术，可以看上面的例子.

我们唯一不能改变，是直接跳转，和循环处理。

像 `while()` or `for(;;)`.

然而，我们可以扩展到有条件的执行整个程序，

像这样所示：

```
while(1)
{
  if (c) line1;
  if (c) line2;
  if (c) c = ...;
  if (c) line3;
  if (c) line4;
}
```

然后，我们仅需要一个 `jmp` 跳转这样的每一个程序上.

正如你所看到的，可以有很多条件，

`if (c) if (d) if (e) lineN`

\- 3 个嵌套循环,

或者，我们可以写成一个状态机的形式，例如

```
if (c==1) line1;
if (c>=2&&c<=3) line2;
if (c==4) line3;
```

等等，所有的程序操作都将可以编码成 `w/o jmps`

对于一个程序，编写者尽量最少的使用 `jxx`,理由如下:

1. 难以理解其逻辑
2. 更不容易理解其指令的排列顺序

ok, 可以想象，用相同的 `C` 的源代码，来产生不同的一些不同的独特的程序.

你可以写一些模板，通过脚本来处理这些模板到 `C` 的转换。

在这些模板中，你可以替换一些像下面的宏的一些定义，这些都是可以随机选择的.

```
#define H0(x)       (((signed)(x)) >> (sizeof((signed)(x))*8-1))
#define H1(a,b)     H0((a)-(b))
#define MIN1(a,b)   ((a)+(H1(b,a) & ((b)-(a))))
#define MIN2(a,b)   ((a)-(H1(b,a) & ((a)-(b))))
#define MIN3(a,b)   ((b)-(H1(a,b) & ((b)-(a))))
#define MIN4(a,b)   ((b)+(H1(a,b) & ((a)-(b))))
//#define MIN5(a,b)   ((a)<(b)?(a):(b))
//#define MIN6(a,b)   ((a)>(b)?(b):(a))
//#define MIN7(a,b)   ((b)>(a)?(a):(b))
//#define MIN8(a,b)   ((b)<(a)?(b):(a))

#define MAX1(a,b)   ((a)+(H1(a,b) & ((b)-(a))))
#define MAX2(a,b)   ((a)-(H1(a,b) & ((a)-(b))))
#define MAX3(a,b)   ((b)-(H1(b,a) & ((b)-(a))))
#define MAX4(a,b)   ((b)+(H1(b,a) & ((a)-(b))))
//#define MAX5(a,b)   ((a)<(b)?(b):(a))
//#define MAX6(a,b)   ((a)>(b)?(a):(b))
//#define MAX7(a,b)   ((b)>(a)?(b):(a))
//#define MAX8(a,b)   ((b)<(a)?(a):(b))

#define ABS1(a)     (((a)^H0(a))-H0(a))
//#define ABS2(a)     ((a)>0?(a):-(a))
//#define ABS3(a)     ((a)>=0?(a):-(a))
//#define ABS4(a)     ((a)<0?-(a):(a))
//#define ABS5(a)     ((a)<=0?-(a):(a))
```

所有的这些宏可以产生不同的代码，生成 `w/o jmps` 类型的代码.

### part 2 -- generating code

在某些情况下，复杂的多态解密器是可以被 `set-of-instructions` 技术检测出来的.

这一技术可以被定义为以下方面：

```
if (一些代码已经包含了一些已知的指令)
{
    return true
  or
    仿真这些指令.
}
```

因此，分析算法时当遇到非定义的指令，那将直接 `returns false`.

可能对抗这种算法检测的方法是，用 `mistfall-like decryptor` 技术注入到

程序代码中（入口模糊），使多态部分的解码器分多个小的代码片断，这些部分都是

间接的联系（例如，分散不同的代码节位置，并利用预先设计的程序进行这些空间的计算）

当你再产生代码时，就可以用下面的方法来改变他们了：

#### 例子 6

```
; at this step we know result of (r1 ? r2)
          cmp     r1, r2
          jxx     l2
l1:
; never executed really, but can be emulated
          {random part of random .code segment}
l2:
; can be either executed or emulated
          {part of our code}
```

#### 例子 7

```
; at this step we know that (r1 % (N+1)) == 2
          and     r1, N
          <jmp|call> dword ptr [offset table1 + r1 * 4]
          ...
table1:   dd l0
          dd l1
          dd l2
          ...
          dd lN
l0:
          ; never executed really, but can be emulated
          {random part of random .code segment}
l1:
          ; never executed really, but can be emulated
          {random part of random .code segment}
l2:
          ; can be either executed or emulated
          {part of our code}
lN:
          ...
```

正如您所看到的，例 6/7 所构造的方法将允许您产生这样的代码.

1. 总体的指令效果是非常接近标志的指令（ps：猜想他的大意应该是说，构造的指令在效果上同真实的、人为写的程序很接近，也就是很

好的迷惑了扫描器）

2. 充分和正确的仿真是需要一个真正确定的指令指标（ps:应该就是指对 `set-of-instructions` 定义的那些指令）

给出的定理:如果一些算法检测将要分析每个有条件指令的跳转，如果仅选择一个条件的指令流（其中只以一些指令是用来定义的）进行分析，那么这将导致误报。

然而，在这里要应该指出：统计法或更先进的

检测算法被用来使用时，是所有其它的简单方法都失效时才会这样做。

那么唯一的办法是写在 `near-to-undetectable` 的代码是不变的，检测算法修正所有导致检测失败的错误。
