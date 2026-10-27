
[93fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4T1k6h3I4Q4y4h3k6T1K9h3N6Q4y4h3k6^5N6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0p5%4z5e0t1%4y4U0M7@1](https://bbs.kanxue.com/elink@191K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4T1k6h3I4Q4y4h3k6T1K9h3N6Q4y4h3k6^5N6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0p5%4z5e0t1%4y4U0M7@1)

[ca5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3@1H3i4K6g2X3y4o6j5J5z5e0j5&6x3o6g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4y4K6x3K6y4U0f1%4y4l9`.`.](https://bbs.kanxue.com/elink@1bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3@1H3i4K6g2X3y4o6j5J5z5e0j5&6x3o6g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4y4K6x3K6y4U0f1%4y4l9`.`.)

## 概念

花指令是企图隐藏掉不想被逆向工程的代码块或其它功能的一种方法，在真实代码中插入一些垃圾代码的同时还保证原有程序的正确执行，而程序无法很好地反编译，难以理解程序内容，达到混淆视听的效果。

花指令通常用于加大静态分析的难度。

## 原理

### 反编译器的线性反编译

反编译器的工作原理是，从`exe`的入口`AddressOfEntryPoint`处开始，依序扫描字节码，并转换为汇编，比如第一个16进制字节码是`0xE8`，一般`0xE8`代表汇编里的`CALL`指令，且后面跟着的4个字节数据跟地址有关，那么反编译器就读取这一共5个字节，反编译为`CALL 0x地址`。

对应的，有些字节码只需要一个字节就可以反编译为一条指令，例如`0x55`对应的是`push ebp`，这条语句每个函数开始都会有。同样，有些字节码又需要两个、三个、四个字节来反编译为一条指令。

也就是说，如果中间只要一个地方反编译出错，例如两条汇编指令中间突然多了一个字节`0xE8`，那反编译器就会将其跟着的4个字节处理为`CALL`指令地址相关数据给反编译成一条`CALL 0x地址`指令。但实际上`0xE8`后面的四个字节是单独的字节码指令。这大概就是**线性反编译**。

### 线性扫描和递归下降

**线性扫描：**

线性扫描的特点：从入口开始，一次解析每一条指令，遇到分支指令不会递归进入分支。

**递归下降：**

当使用线性扫描时，比如遇到`call`或者`jmp`的时候，不会跳转到对应地址进行反汇编，而是反汇编`call`指令的下一条指令，这就会导致出现很多问题。

递归下降分析当遇到分支指令时，会递归进入分支进行反汇编。

### 使反汇编引擎解析错误

`X86`指令集的长度是不固定的，有一些指令很短，只有1个字节，有些指令比较长，可以达到5字节，指令长度不是固定的。如果通过巧妙的构造，引导反汇编引擎解析一条错误的指令，扰乱指令的长度，就能使反汇编引擎无法按照正常的指令长度一次解析邻接未解析的指令，最终使反汇编引擎输出错误的反汇编结果。

### 机器码

`0xE8` `CALL` 后面的四个字节是地址
`0xE9` `JMP` 后面的四个字节是偏移
`0xEB` `JMP` 后面的二个字节是偏移
`0xFF15` `CALL` 后面的四个字节是存放地址的地址
`0xFF25` `JMP` 后面的四个字节是存放地址的地址

`0x68` `PUSH` 后面的四个字节入栈
`0x6A` `PUSH` 后面的一个字节入栈

### 对于出题人

从出题人的角度来看，构造有效花指令的关键思路就是构造使源程序逻辑不受影响的内联汇编代码，同时在内联汇编代码中嵌入`jmp` `call`+`ret`之类的对应机器码指令，使反汇编软件在反汇编时错误地识别这些机器码为汇编指令，从而影响反汇编出来的程序的正常流程。

## 写花指令的原则

**保持堆栈的平衡**

## 常用指令含义

`push ebp` ----把基址指针寄存器压入堆栈

`pop ebp` ----把基址指针寄存器弹出堆栈

`push eax` ----把数据寄存器压入堆栈

`pop eax` ----把数据寄存器弹出堆栈

`nop` -----不执行

`add esp,1`-----指针寄存器加1

`sub esp,-1`-----指针寄存器加1

`add esp,-1`--------指针寄存器减1

`sub esp,1`-----指针寄存器减1

`inc ecx` -----计数器加1

`dec ecx` -----计数器减1

`sub esp,1` ----指针寄存器-1

`sub esp,-1`----指针寄存器加1

`jmp 入口地址`----跳到程序入口地址

`push 入口地址`---把入口地址压入堆栈

`retn` ------ 反回到入口地址,效果与`jmp 入口地址`一样

`mov eax,入口地址` ------把入口地址转送到数据寄存器中.

`jmp eax` ----- 跳到程序入口地址

`jb 入口地址`

`jnb 入口地址` ------效果和`jmp 入口地址`一样,直接跳到程序入口地址

`xor eax,eax` 寄存器`EAX`清0

`CALL` 空白命令的地址 无效`call`

## 栈指针平衡

![image-20230815094310227](https://bbs.kanxue.com/upload/attach/202311/957393_NZ9UPWR2RMKB6FK.jpg)

当使用`IDA`分析伪代码时，有花指令会发生

![image-20230815094348064](https://bbs.kanxue.com/upload/attach/202311/957393_YRY9Z8BECKWT9MJ.jpg)

无法查看伪代码

需要去给出的地址查看具体发生的问题

这里，我们要设置一下`IDA`，让它显示出栈指针

`（Options-General-Disassembly-"Stack pointer"）`

![image-20230815094454421](https://bbs.kanxue.com/upload/attach/202311/957393_ZNTCS95VZDJD7PR.jpg)

程序段结束后，不应发生`mov esp,ebp`的操作，因为在`pop`出栈后，`esp`和`ebp`的值相等，这一步是多余的，因为栈指针已经正确地回到了调用函数前的位置，这会引起栈指针不平衡。

这就需要修改栈指针

### 手动修改地址

> 注意：每条语句前的栈指针是这条语句未执行的栈指针。

找到函数段的开始地址

![image-20230815094801422](https://bbs.kanxue.com/upload/attach/202311/957393_9EJJ3XD66636XJ9.jpg)

计算结束地址的栈指针应为多少：

`0x21E-0x4 = 0x21A`

修改最后两句应为的栈指针：

`Alt+k`：

![image-20230815095110085](https://bbs.kanxue.com/upload/attach/202311/957393_RA5YK2TJANTM7A2.jpg)

![image-20230815095119159](https://bbs.kanxue.com/upload/attach/202311/957393_9U59QMBHW472DYW.jpg)

栈指针平衡

### 使用插件nop掉

通过前面知道，经过`pop`栈针已经平衡，所以这两句汇编代码是没有必要的

![image-20230815095303010](https://bbs.kanxue.com/upload/attach/202311/957393_KHG4BJFNGNWCGTE.jpg)

删除这两条指令的目的是在已经发生了出栈操作并且栈指针`ESP`与基址指针`EBP`相等的情况下，不再手动调整栈指针。这是因为栈指针已经回到了调用函数之前的位置，不需要再额外的指令来处理栈平衡。

![image-20230815095343701](https://bbs.kanxue.com/upload/attach/202311/957393_F3FJNWADJV5V547.jpg)

## 花指令的编写

> 此处笔者踩了个大坑，值得一提的是，`win`下的`gcc`只支持`x86`下的内联汇编。

环境：`VS2022` `x86`架构 `C++`

reference

[内联汇编官方文档](https://bbs.kanxue.com/elink@583K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6U0M7s2m8Q4x3V1k6S2M7%4y4W2L8h3u0D9k6i4u0Q4x3V1k6A6L8X3I4A6L8X3g2Q4x3V1k6S2M7$3#2Q4x3@1k6$3K9h3g2%4i4K6y4p5L8i4y4$3j5#2)9J5k6o6p5%4x3l9`.`.)

### asm

> **`__asm`** 关键字用于调用内联汇编程序，并且可在 C 或 C++ 语句合法时出现。 它不能单独显示。 它后面必须跟一个程序集指令、一组括在大括号中的指令，或者至少是一对空大括号。 此处的术语“**`__asm`** 块”指任何指令或指令组（无论是否在大括号中）。

### asm语法

```
asm-block:
    __asm assembly-instruction;opt
    __asm { assembly-instruction-list };opt

assembly-instruction-list:
    assembly-instruction;opt
    assembly-instruction;assembly-instruction-list;opt
```

### asm示例

1.括在大括号里的简单 **`__asm`** 块：

```
__asm {
    mov al, 2
    mov dx, 0xD007
    out dx, al
}
```

2. **`__asm`** 放在每个程序集指令前面：

```
__asm mov al, 2
__asm mov dx, 0xD007
__asm out dx, al
```

3.由于 **`__asm`** 关键字是语句分隔符，因此还可将程序集指令放在同一行中：

```
__asm mov al, 2   __asm mov dx, 0xD007   __asm out dx, al
```

这三个示例将生成相同的代码，但第一个样式（用大括号括起 **`__asm`** 块）具有一些优势。 大括号可清楚地将程序集代码与 C 或 C++ 代码分隔开，并避免了不必要的 **`__asm`** 关键字重复。 大括号还可防止二义性。 如果要将 C 或 C++ 语句放在与 **`__asm`** 块相同的行上，则必须将此块括在大括号中。 如果没有大括号，编译器无法判断程序集代码停止的位置以及 C 或 C++ 语句的开始位置。

### 花指令实现

reference：

[deaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6y4U0b7&6x3q4)9J5x3$3R3J5i4K6u0V1x3b7`.`.](https://bbs.kanxue.com/elink@4bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6y4U0b7&6x3q4)9J5x3$3R3J5i4K6u0V1x3b7`.`.)

1.插入字节：这里就提到汇编里一个关键指令：`_emit 立即数`

```
//C语言中使用内联汇编
__asm
{
    _emit 0xE8
}
//代表在这个位置插入一个字节数据0xE8
```

2.保证不被执行：通过构造一个永恒的跳转

```
__asm
{
    jmp Label1
    db thunkcode1;  垃圾数据
    //垃圾数据例如：_emit 0xE8
    Label1:
}
```

例如这样：

![img](https://bbs.kanxue.com/upload/attach/202311/957393_R3QXDS9EDB65NCB.jpg)

我对这反编译器对花指令的反编译稍作修改

![img](https://bbs.kanxue.com/upload/attach/202311/957393_TH7V8BEU8PJXAYT.jpg)

**去掉花指令-> nop(0x90)**

这部分是通过`IDA`手动去掉花指令，也可以在`IDA`里用`IDApython/IDC`写脚本去，或者在`OD`调试的时候去掉，原理都一样。

![img](https://bbs.kanxue.com/upload/attach/202311/957393_DGXCRPJZDNFNR48.jpg)

![img](https://bbs.kanxue.com/upload/attach/202311/957393_JMACS66YK99V5YB.jpg)

![img](https://bbs.kanxue.com/upload/attach/202311/957393_HBFXRAMKDF3326W.jpg)

![img](https://bbs.kanxue.com/upload/attach/202311/957393_YRKKZQW7R5ET7PU.jpg)

![img](https://bbs.kanxue.com/upload/attach/202311/957393_A97J6RF237XK7M6.jpg)

### 花指令分类

> ### emit指令的作用：
>
> 1. 编译器不认识的指令，拆成机器码来写。
> 2. 插入垃圾字节来反跟踪，又称花指令。
>    用`emit`就是在当前位置直接插入数据（实际上是指令），一般是用来直接插入汇编里面没有的特殊指令，多数指令可以用`asm`内嵌汇编来做，没有必要用`emit`来做，除非你不想让其它人看懂你的代码。
>    我们来看用`IDA`反汇编的效果吧。

#### 1.最简单的花指令

##### a.最简单的jmp

```
jmp Label1
  db thunkcode1;垃圾数据
Labe1:
```

不过很可惜，反编译器能直接识别这种简单花指令，遇到这种能轻松过掉并反编译。

##### b.过时的多节形式与多层乱序

这两周都是通过多次跳转，把垃圾数据和有用代码嵌套在一起，不过这种形式也比较老套了，反编译器依然能够轻松过掉并成功反汇编。

```
#多节形式
JMP Label1
  Db thunkcode1
Label1:
  …… 
  JMP Label2
  Db thunkcode2
Label2:
  …… 
JMP Label1
  Db thunkcode1
Label2:
  …… 
  JMP Label3
  Db thunkcode3
Label1:
  …… .
  JMP Label2
  Db thunkcode2
Label3:
  …… 
```

#### 2.简单花指令

##### a.互补条件代替jmp跳转

```
asm
{
  Jz Label
  Jnz Label
  Db thunkcode;垃圾数据
Label:
}
```

类似这种，无论如何都会跳转到`label1`处，还是能骗过反编译器。

##### b.跳转指令构造花指令

1.简单跳转

```
__asm {
    push ebx;
    xor ebx, ebx;
    test ebx, ebx;
    jnz LABEL7;
    jz    LABEL8;
LABEL7:
    _emit 0xC7;
LABEL8:
    pop ebx;
}
```

很明显，先对`ebx`进行`xor`之后，再进行`test`比较，`zf`标志位肯定为1，就肯定执行`jz LABEL8`，也就是说中间`0xC7`永远不会执行。
