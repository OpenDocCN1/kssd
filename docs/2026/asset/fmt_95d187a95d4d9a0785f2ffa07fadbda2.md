
.cfi系列指令的表达能力是很有限的，它只能使用`REG + OFFSET`的形式恢复`CFA`，只能使用`[CFA + OFFSET]`与`REG1 = REG2`的形式恢复其它寄存器。为了解决这个问题，`DWARF 3`标准引入了`DWARF Expression`。它是一个支持一系列操作的基于栈的虚拟机，关于它所支持的操作可以参考`DWARF`标准[1]的2.5.1节（General Operations）

`DWARF Expression`支持的操作可以分为编码（入栈立即数），寄存器寻址（入栈 `REG + OFFSET`），栈操作（`SWAP`，`POP`等操作），算术运算，流程转移。我简单介绍一些常用的操作。

### 编码（入栈立即数）

对于一条`push imm32`指令而言，立即数`imm32`的编码可以使用两种方式：1. 标准补码编码；2. `U/SLEB128`编码[3]。`LEB128`编码包含`ULEB128`与`SLEB128`，分别编码无符号数与有符号数，它的编码与解码过程在WIKI百科上的描述很详细，我就不在此赘述了。

虚拟机中单位元素的长度与当前机器上地址的长度相等，比如在`AMD64`架构下，地址的长度是`int64`，那么虚拟机中单位元素就是`int64`。

- `DW_OP_const1(2, 4, 8)u OP1(u_int8, u_int16, u_int32, u_int64)`

  这4条指令都包含操作数`OP1`，`OP1`使用补码编码，语义都是将`OP1`压入栈中。如果`OP1`的长度小于单位元素长度，使用0补齐高位。

- `DW_OP_const1(2, 4, 8)s OP1(int8, int16, int32, int64)`

  与上一条指令基本相同，区别是这条指令压入的是有符号数，而上一条指令压入的是无符号数。

- `DW_OP_constu OP1(ULEB128)`

  这条指令包含操作数`OP1`，`OP1`使用`ULEB128`编码，它向栈中压入`OP1`。

- `DW_OP_consts OP1(SLEB128)`

  与上一条指令基本相同，区别是`OP1`使用`SLEB128`编码，压入的是有符号数。

### 寄存器寻址

- `DW_OP_breg***n*** OP1(SLEB128)`

  `n`的取值可以是0-31，代表着寄存器的编号。`AMD64`环境中寄存器编号如下图所示。这条指令向栈中压入 `REG + OP1`，`REG`是由`n`指定的。注意压入的仅仅是地址，而不存在解引用的过程，解引用操作需要使用`DW_OP_deref`指令。

  ![图片描述](https://bbs.kanxue.com/upload/attach/202203/783210_5FGC9AVZAEA7EBR.png)

- `DW_OP_bregx OP1(LEB128), OP2(SLEB128)`

  这条指令与上一条基本相同，不同的是`REG`使用操作数`OP1`指定了，`OP1`是寄存器编号。这条指令向栈中压入 `REG + OP2`，`REG`由`OP1`指定。

### 栈操作

- `DW_OP_drop`

  从栈中弹出栈顶元素

- `DW_OP_pick OP1(u_int8)`

  复制一份栈中第`OP1`个元素压入栈顶。栈中元素的编号从0开始，0是栈顶。

- `DW_OP_swap`

  交换栈顶两个元素

- `DW_OP_deref`

  弹出栈顶元素作为地址，解引用这个地址，值压入栈顶。

- `DW_OP_deref_size OP1(u_int8)`

  与上一条指令基本相同，不同的是上一条指令无法控制读取的长度，只能是栈的单位元素的长度，这条指令可以控制读取长度。操作数`OP1`指示读取长度，是字节数。如果小于栈单位元素长度，用0补齐高位；如果大于栈单位元素长度则会报错。

### 算数运算指令

- `DW_OP_plus`

  弹出栈顶两个元素，相加，值压入栈顶。

- `DW_OP_neg`

  弹出栈顶元素，取负，值压入栈顶。注意虚拟机中没有`sub`操作，因此使用`DW_OP_neg`, `DW_OP_plus`来表示减法操作，即弹出栈顶两个元素，用栈顶第二个元素减去第一个元素，值压入栈顶。

- `DW_OP_mul`

  乘法

- `DW_OP_mod`

  求模

- `DW_OP_or, and, not, xor`

  与，或，非，异或

- `DW_OP_shr, shl；ashr`

  逻辑右移，左移；算数右移

### 控制流转移指令

- `DW_OP_le, ge, eq, lt, gt, ne`

  弹出栈顶两个元素，栈顶第二个元素记为`O2`，栈顶元素记为`O1`

  比较两个操作数，`O2 le, ge, eq, ... O1`

  如果该表达式成立，把1压入栈顶，否则把0压入栈顶。

  比较是有符号形式的，`le(less equal, <=)`, `gt(great than, >)`...

  可以将无符号数的比较转为有符号数的比较，如比较无符号数`U1`，`U2`大小，可以转换为 `U1 - U2` 与 `0` 的有符号大小比较。

- `DW_OP_bra OP1(int16_t)`

  操作数`OP1`使用补码编码，并且是2字节大小。弹出栈顶元素，如果它非0，则跳转到`OP1`处执行，`OP1`表示偏移地址，它是从`OP1`后面开始算的。比如 `DW_OP_bra 10` 的字节码是 `0x28 0x0A 0x00` ，假设 `0x28` 是第0个字节，`0x00`是第2个字节，那么跳转的目标就是第12个字节

- `DW_OP_skip OP1(int16_t)`

  与上一条指令基本相同，区别是这条指令是无条件跳转。

在我们编写完一个`DWARF Expression`后，还需要通过`DW_CFA_val_expression`指令将这个`DWARF Expression`计算的值赋值给特定寄存器。

- `DW_CFA_val_expression REG(ULEB128), LENGTH_OF_EXPRESSION(ULEB128), EXPRESSION_CODE...`

  `REG`是寄存器编号，`LENGTH_OF_EXPRESSION`是编写的`Expression`的字节码的长度，后面跟着的就是`Expression`的字节码。

## DWARF Expression 存在的局限性

- 不能在虚拟机内写入程序运行内存，而只能读取程序运行内存。也就是说，如果我们要写入内存，只能在退出虚拟机后写入。

- `DWARF Expression`并不能”恢复“所有寄存器。严格来说并不是不能”恢复“所有寄存器，而是恢复之后又会调用一些`C++`标准库函数导致恢复的值又被破坏掉了，最终我们在`catch`块中没办法得到恢复的值。`RBX`，`R12-R15`寄存器是`AMD64 ABI`定义的由被调用者保护的寄存器，因此它们的值一定可以在`catch`块中被接收到。我个人建议使用这5个寄存器作为退出虚拟机时的输出，退出后可以将它们的值写入内存或者其它寄存器值。

# 方法

## 将汇编代码转换为DWARF Expression

注意`DWARF Expression`的局限性，它不能写入内存，并且不能“写入”保护寄存器以外的寄存器。我实现了一个初步的Demo，它可以将一个不包含内存访问指令与`CALL`指令的基本块从汇编代码转换为`DWARF Expression`。

由于保护寄存器以外的寄存器都是易失的，我的想法是被混淆的代码片段的输入（引用先于赋值）寄存器和输出寄存器都应该是保护寄存器的子集。为了让编译器生成这样的代码，我使用`LLVM PASS`实现了一种插桩方式，如下所示。

|  |  |

| --- | --- |

| 1  2  3  4  5 | `handle1()`    `需要混淆的代码片段`    `handle2()` |

由于代码片段的前面有对`handle1`函数的调用，因此编译器会将代码片段要引用的寄存器都转存在保护寄存器中；由于代码片段的后面有对`handl2`函数的调用，因此编译器也会将代码片段的输出全部放在保护寄存器中。这就生成了符合我们要求的代码。

而后，我的转换方案包含3步：

1. 将汇编代码转换为二叉表达式树

2. 后序遍历二叉树，得到后缀表达式

3. 按顺序翻译后缀表达式就可以得到`DWARF Expression`

### 转换为二叉表达式树

二叉树的节点如下所示：

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `class` `ASTNode:`  `def` `__init__(``self``, value, valtype, left, right):`  `self``.Value` `=` `value` `#数值`  `self``.``Type` `=` `valtype` `#数值类型，可以是寄存器，常量，操作符`  `self``.Left` `=` `left`  `self``.Right` `=` `right` |

算法主要步骤如下：

1. 初始化

   初始化寄存器环境`regs`字典，将所有寄存器的初始值都设置为`ASTNode(reg, VAL_REG, None, None)`

2. 解析每一条指令

   如`addq %reg1, %reg2`指令可以解析为：`regs[1] = ASTNode("+", VAL_OP, regs[op[0]], regs[op[1]])`；`addq $const, %reg1`指令可以解析为：`regs[1] = ASTNode("+", VAL_OP, regs[op[1]], ASTNode(int(op[0]), VAL_CONST, None, None))`

需要注意的是对于`addl, imul`等指令的语义的设置。对于`addl %eax, %ebx`而言，`ebx = (rax + rbx) & 0xFFFFFFFF`，对于`imull %eax,%ebx,%ecx`而言，`ecx = (rax * rbx) & 0xFFFFFFFF`。可以只在获取最终结果时进行截断操作，而无需在计算前截断参与运算的寄存器。

```
def Block2AST(block):
    #DO NOT touch the shit of memory access, it's really a nightmare
    if not isHandleable(block):
        return None
        # Because we add call in front of block and after block, so, the block should not use regs
        # not included in RBX, R12-R15 before set them

    regs = {}
    for reg in REGISTER_NUMBER.keys():
        regs[reg] = ASTNode(REGISTER_NUMBER[reg], VAL_REG, None, None)

    for line in block:
        mm = line.split("\t")[0]
        op = line.split("\t")[1]
        if op.find("#") != -1:
            op = op[:op.find("#")]
        op = op.split(",")
        for i in range(len(op)):
            op[i] = op[i].strip()
            op[i] = op[i].replace("%", "")
            op[i] = op[i].replace("$", "")

        if mm == 'imulq':
            if len(op) == 3:
                if isREG(op[0]):
                    regs[op[2]] = ASTNode("*", VAL_OP, regs[op[0]], regs[op[1]])
                else:
                    regs[op[2]] = ASTNode("*", VAL_OP, ASTNode(int(op[0]), VAL_CONST, None, None), regs[op[1]])
            elif len(op) == 2:
                if isREG(op[0]):
                    regs[op[1]] = ASTNode("*", VAL_OP, regs[op[0]], regs[op[1]])
                else:
                    regs[op[1]] = ASTNode("*", VAL_OP, ASTNode(int(op[0]), VAL_CONST, None, None), regs[op[1]])
            elif len(op) == 1:
                print("ERROR: IMUL WITH ONLY 1 OP")
                exit(0)
        elif mm == 'movabsq':
            regs[op[1]] = ASTNode(int(op[0]), VAL_CONST, None, None)
        elif mm == 'addq':
            if isREG(op[0]):
                regs[op[1]] = ASTNode("+", VAL_OP, regs[op[0]], regs[op[1]])
            #......省略，模拟指令语义都是一些很dirty的工作
            else:
                regs[op[1]] = ASTNode("+", VAL_OP, ASTNode(int(op[0]), VAL_CONST, None, None), regs[op[1]])
        elif mm == 'movq':
            regs[op[1]] = regs[op[0]]
        elif mm == 'shrq':
            if isREG(op[0]):
                print("ERROR: SHRQ WITH REG OP2")
                exit(0)
            else:
                regs[op[1]] = ASTNode(">>", VAL_OP, regs[op[1]], ASTNode(int(op[0]), VAL_CONST, None, None))
        else:
            print("ERROR: " + mm + " UNSUPPORTED")
        return regs
```

### 获取后缀表达式

只是一个简单的后序遍历。

```
def getPostFix(root):
    if root.Left == None and root.Right == None:
        return val2Str(root)

    s1 = getPostFix(root.Left)
    s2 = getPostFix(root.Right)
    return s1 + " " + s2 + " " + val2Str(root)
```

### 后缀表达式转为DWARF Expression
