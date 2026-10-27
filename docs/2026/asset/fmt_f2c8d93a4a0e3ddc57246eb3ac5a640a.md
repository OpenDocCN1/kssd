
目前我的探索发现到此。以下领域尚未覆盖：

- 非字符串常量如何存储
- 函数原型的形式为何

NFI涵括了这些以及其它未提及的东西，但是是5.1版本的，所以你得参照`lundump.c`和`LAT`。

# 全面窥探字节码

伴随此文件而来的还有两个模块文件：`vm52.lua`和`bytecode.lua`。这阶段一开始，我们仅使用`numberAt`。这个函数将4字节子字符串转成32位数字，使用宿主机器的字节端序。我们从上面了解到dump后的Lua5.2函数在28是一个可变长函数参数，在29是寄存器数量，在30是指令数量，随后是许多的4字节指令。

所以让我们来看一下。

```
$ lua -l vm52

Lua 5.2.1  Copyright (C) 1994-2012 Lua.org, PUC-Rio

   func = load 'local  hello = "Hello" print (hello.." World!")'

   chunk = string.dump(func)

   function hasvarargs(chunk) return chunk:sub(28,28):byte()>0 end

   function nstack(chunk) return chunk:sub(29,29):byte() end

   function ninstr(chunk) return vm52.numberAt(chunk,30) end

   function instr(chunk,i) return vm52.numberAt(chunk:sub(30+4*i,33+4*i)) end

   ns = nstack(chunk)

   print ("This function" ..

     (hasvarargs(chunk) and " has a variable number of arguments and" or "") ..

     " uses "..ns.." register"..(ns~=1 and "s" or ""))

   for i=1,ninstr(chunk) do

      print(string.format("%08X",instr(chunk,i)))

   end 

00000001

00404046

00000080

000080C1

0100C096

0100405D

0080001F
```

你能看到字节码文件的字节内容，倒转是因为我的机器是小端字节序。

模块文件`vm52.lua`不仅有`numberAt`：它有上面代码定义的四个函数，更重要的，它还有`assemble`和`disassemble`。后者做的跟`lua -l`一样处理dump后的字节码。

继续之前的Lua会话，

```
   for i=1,vm52.ninstr(code) do

      print(vm52.disassemble(vm52.instr(code,i)))

   end 

LOADK 0 -1

GETTABUP 1 0 -2

MOVE 2 0

LOADK 3 -3

CONCAT 2 2 3

CALL 1 2 1

RETURN 0 1
```

# 指令解剖

以上16进制列表与字节码文件相对应，不过字节是倒转的。这是因为我的系统是小端序的：小端字节序的32位数字真是难以阅读，所以普遍使用的是大端字节序。

所以在分解和显示它们之前，我们把指令字节倒转过来。`numberAt`函数会做这个事情，如果需要的话：

就像我们看过的，指令是：

```
00 00 00 01   LOADK 0 -1

00 40 40 46   GETTABUP 1 0 -2

00 00 00 80   MOVE 2 0

00 00 80 c1   LOADK 3 -3

01 00 c0 96   CONCAT 2 2 3

01 00 40 5d   CALL 1 2 1

00 80 00 1f   RETURN 0 1
```

我们现在将字节展开成二进制并且分组。如果`lopcodes.c`指明指令是`iABC`模式的，我们用（9，9，8，6）分组，定义值为`B`，`C`，`A`，`OP`；如果指明是`iABx`或者`iAsBx`，用（18，8，6）分组，定义值为`Bx`，`A`，`OP`或者`sBx`，`A`，`OP`；如果是`iAx`，分为（26，6），定义值为`Ax`，`OP`。

```
    B    Bx   C         A      OP

000000000000000000  00000000 000001   LOADK 0 -1       (A,Bx)

000000000 100000001 00000001 000110   GETTABUP 1 0 -2  (A,B,C)

000000000 000000000 00000010 000000   MOVE 2 0         (A,B) 

000000000000000010  00000011 000001   LOADK 3 -3       (A,Bx)

000000010 000000011 00000010 010110   CONCAT 2 2 3     (A,B,C)

000000010 000000001 00000001 011101   CALL 1 2 1       (A,B,C)

000000001 000000000 00000000 011111   RETURN 0 1       (A,B)
```

上面的表格里面，每一位的编号从左到右是31，30，…，0。记号“0+6”意味着位置0 的低6位（表格最右边）。如果指令作为一个Lua数字存储，这几位用`bit32:extract(0,6)`表示。

1、     `OP`是操作码（opcode），在0+6（看下面）。

2、     `A`是所有指令的第一操作数(operand)，在6:8，视为无符号数字。最大值是255。

3、     `B`是模式`iABC`的第二操作数，在23+9，视为有符号数字。实际的指令未必用到`B`。

第一位是符号位。如果是1，其它8位为`B`的绝对值减1。因此没有-0，`B`的最小值是-256，最大值255。

4、     `C`是`iABC`的第三操作数，在14+9，视为像`B`一样的有符号数字。实际指令未必用到`C`。

5、     `Bx`是`iABx`的第二操作数，在14+18，像`B`一样视为19位有符号数字，但是不需存储符号位因为`Bx`总是负的。

6、     `sBx`意思是“`Bx`视为18为有符号数字”。

7、     `Ax`是模式`iAx`的唯一操作数，在6+26，视为`Bx`（27位有符号数字，负数，符号位不存储）。

指令的操作码及其功能描述在`lopcodes.h`的一个`enum`中。我们以其相关功能进行分组。描述（从`lopcodes.h`中逐字取出）中使用以下记号，`B`，`Bx`等不是指位编码的数字，而是指`luac`反编译后能看到的数字。

```
R(A), R(B), R(C)

    寄存器A,B或者C的数值。

Kst(Bx)

    常数|Bx|。

KPROTO[Bx]

    函数原型|Bx|。

RK(B), RK(C)

    寄存器B或者C的值；如果B或者C是负数，常数|B|或者|C|。

FPF

    每次刷新的Fields。阅读NFI的解释。

UpValue

    外部局部变量B的值

Pc

    程序计数器：下一条被执行的指令的编号

closure

    创建一个闭包（函数加上外部局部变量）。
```

## 加载参数

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 1 | `LOADK(A,Bx)` | `R(A) := Kst(Bx)` |
| 2 | `LOADKX(A)` | `R(A) := Kst(extra arg)` |
| 39 | `EXTRAARG(Ax)` | `extra (larger) argument for previous opcode` |

## 一元函数

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 0 | `MOVE(A,B)` | `R(A) := R(B)` |
| 19 | `UNM(A,B)` | `R(A) := -R(B)` |
| 20 | `NOT(A,B)` | `R(A) := not R(B)` |
| 21 | `LEN(A,B)` | `R(A) := length of R(B)` |

## 二元函数

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 13 | `ADD(A,B,C)` | `R(A) := RK(B) + RK(C)` |
| 14 | `SUB(A,B,C)` | `R(A) := RK(B) - RK(C)` |
| 15 | `MUL(A,B,C)` | `R(A) := RK(B) * RK(C)` |
| 16 | `DIV(A,B,C)` | `R(A) := RK(B) / RK(C)` |
| 17 | `MOD(A,B,C)` | `R(A) := RK(B) % RK(C)` |
| 18 | `POW(A,B,C)` | `R(A) := RK(B) ^ RK(C)` |

## 表格处理

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 7 | `GETTABLE(A,B,C)` | `R(A) := R(B)[RK(C)]` |
| 10 | `SETTABLE(A,B,C)` | `R(A)[RK(B)] := RK(C)` |
| 11 | `NEWTABLE(A,B,C)` | `R(A) := {} (size = B,C)` |
| 12 | `SELF(A,B,C)` | `R(A+1) := R(B); R(A) := R(B)[RK(C)]` |
| 36 | `SETLIST(A,B,C)` | `R(A)[(C-1)*FPF+i] := R(A+i), 1 <= i <= B)` |

## 元组处理

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 4 | `LOADNIL(A,B)` | `R(A), R(A+1), ..., R(A+B) := nil` |
| 22 | `CONCAT(A,B,C)` | `R(A) := R(B).. ... ..R(C)` |
| 29 | `CALL(A,B,C)` | `R(A), ... ,R(A+C-2) := R(A)(R(A+1), ... ,R(A+B-1))` |
| 31 | `RETURN(A,B)` | `return R(A), ... ,R(A+B-2) (see note)` |
| 38 | `VARARG(A,B)` | `R(A), R(A+1), ..., R(A+B-2) = vararg` |
| 30 | `TAILCALL(A,B,C)` | `return R(A)(R(A+1), ... ,R(A+B-1))` |
| 34 | `TFORCALL(A,C)` | `R(A+3), ... ,R(A+2+C) := R(A)(R(A+1), R(A+2));` |

## 与外部局部变量的交互

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 5 | `GETUPVAL(A,B)` | `R(A) := UpValue` |
| 9 | `SETUPVAL(A,B)` | `UpValue := R(A)` |
| 6 | `GETTABUP(A,B,C)` | `R(A) := UpValue[RK(C)]` |
| 8 | `SETTABUP(A,B,C)` | `UpValue[A][RK(B)] := RK(C)` |

## 逻辑函数

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 3 | `LOADBOOL(A,B,C)` | `R(A) := (Bool)B; if (C) pc++` |
| 24 | `EQ(A,B,C)` | `if ((RK(B) == RK(C)) ~= A) then pc++` |
| 25 | `LT(A,B,C)` | `if ((RK(B) < RK(C)) ~= A) then pc++` |
| 26 | `LE(A,B,C)` | `if ((RK(B) <= RK(C)) ~= A) then pc++` |
| 27 | `TEST(A,C)` | `if not (R(A) <=> C) then pc++` |
| 28 | `TESTSET(A,B,C)` | `if (R(B) <=> C) then R(A) := R(B) else pc++` |

## 分支、循环和闭包

|  |  |  |
| --- | --- | --- |
| 操作码 | 名称和参数 | 描述 |
| 23 | `JMP(A,sBx)` | `pc+=sBx; if (A) close all upvalues >= R(A) + 1` |
| 32 | `FORLOOP(A,sBx)` | `R(A)+=R(A+2); if R(A) <?= R(A+1) then { pc+=sBx; R(A+3)=R(A) }` |
| 33 | `FORPREP(A,sBx)` | `R(A)-=R(A+2); pc+=sBx` |
| 35 | `TFORLOOP(A,sBx)` | `if R(A+1) ~= nil then { R(A)=R(A+1); pc += sBx }` |
| 37 | `CLOSURE(A,Bx)` | `R(A) := closure(KPROTO[Bx])` |

# 札记

以下几乎逐字抄录自`lopcodes.h`

- 在`CALL`里，如果（`B==0`）则`B=top`.如果（`C==0`），那么`top`设置成`last_result+1`，这样下一个开放指令（`CALL`，`RETURN`，`SETLIST`）可以使用`top`.

- 在`VARARG`里，如果（`B==0`）则使用`varargs`的真实值作为`top`.（如同`C==0`的`CALL`）

- 在`RETURN`里，如果（`B==0`）则返回直到`top`。

- 在`SETLIST`里，如果（`B==0`）则`B=top`;如果（`C==0`）则吓一跳'指令'是`EXTRAARG`（真正的`C`）。

- 在`LOADKX`里，下一条'指令'总是`EXTRAARG`.

- 在比较的指令里，`A`指定条件测试必须满足的条件（true或者false）

- 所有的'跳跃'(skips)假设下一指令是一个跳转（jump）。

# VM5.2概要评论

> ### 一知半解危害不浅。彼埃利亚的泉水如不痛饮就别只尝一点。

这句亚历山大·蒲柏的名言是个很好的忠告。

你绝对要先读过NFI，然后把这部分仅仅当成文档其它必须理解的部分的要点总结。实际上，它仅包含了一些帮助我自己读懂上面操作指令的东西，所以继续之前重读一下前面是个好主意。

我发现反编译后的Lua代码片段很有帮助，可以弄清楚为什么虚拟机指令可以达到预期的效果。例如：

```
$ lua -l vm52

   src = "local x, y = ...; return x+y, ..."

   chunk = string.dump(load(src))   

   for i=1,vm52.ninstr(chunk) do

      print(vm52.disassemble(vm52.instr(chunk,i)))

   end

VARARG 0 3

ADD 2 0 1

VARARG 3 0

RETURN 2 0

RETURN 0 1 
```

```
VARARG 0 3

    从vararg加载前（3-1）个参数到寄存器0开始的连续寄存器。

ADD 2 0 1

    将寄存器0和1的数相加，结果保存至寄存器2。

VARARG 3 0

    加载vararg的所有参数列表到从寄存器3开始的连续寄存器。

RETURN 2 0

    返回寄存器2到栈顶的所有东西

RETURN 0 1

    一般情况下，Lua不能确定没有条件分支语句跳过前面的return语句，作为一个好的处理措施，它总是插入另一个return语句。
```

Lua的C API在一个虚拟堆栈上做一切的事情。从某个角度看，虚拟机仅仅只是一个通向API的接口，提供了一个Lua源码和API间的抽象层。然而，VM会访问`lua_State`的一个私有域，而C API完全通过文档描述的方式工作。

虚拟机本身并不是基于栈的。它允许独占使用叫做帧（frame）的栈的一部分，但是在帧的内部，VM的寄存器数量是固定的，没有pushing和poping。一个重要的使用上的区别是负索引跟C API的含义不同。它们指向别处存储的常量，不在栈帧里面。

VM可以访问其它活跃栈帧上的外部局部变量。

局部变量是寄存器的别名。`luac`列表从调试信息中抓取他们的名字，但是指令反编译器做不到这点。创建它们不涉及任何开销。

# 现在就是这样

当我有时间和兴趣的时候，我会添加更多的东西。欢迎各位指正和评论。如果我还活跃的话，你能在`lua-l`上找到我。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`lua5.2字节码和虚拟机.docx`](javascript:void(0);)

  （42.06kb，199次下载）
