
他们之间的唯一的不同在于`bit1`，`Opcode`中的这个位一般被称为`d`位，`d`位可以在有着相同操作和操作对象的两条指令之间区分操作对象的顺序。（前面和一些朋友讨论过，Intel汇编格式 `Instruction arg1, arg2`中，`arg1`代表着目的操作数，而`arg2`代表着源操作数。例如，`mov arg1, arg2`就是要把`arg2`存放到`arg1`中去，这个规定是无法改变的，而这里的`d`位只是用于在两条指令之间进行区分，也就是你读了`Opcode`的这一位，不用看指令说明书，就知道`ModR/M`编码的两个操作对象，哪个是源操作数，哪个是目的操作数。）`d`位只是在机器码识别中很有用的一个标志而已，一般一条指令`r/m`和`reg`的位置都是在指令说明书中明确规定了的。

`d`位值的涵义是很明显的，当`d = 1`时，源操作数为`R/M`目的操作数为`Reg/Opcode`；当`d = 0`时，源操作数为`Reg/Opcode`而目的操作数为`R/M`。可以用`d`位来进行区分的指令范围也很明显——指令的操作对象中必须有`ModR/M`编码的两个操作对象。`d`位在指令识别过程中的作用跟第四部分中提到过的“规律”没有任何的区别，在`switch...case`式的`Opcode`识别方式中，`d`位是必须要考虑的。在用翻译表的方式设计的实现方式中，由于操作对象的顺序可以预先存储在表格中，所以`d`位的解析一般是没有什么意义的（当然，如果把`d`位信息也编码到翻译表中，减少翻译表的信息冗余也是可以的）。

只要是用`ModR/M`编码的指令，大家可以在`od`中看看，`d`位是不是能像上面说的那样，区分`R/M`和`Reg/Opcode`的顺序。

### 5.3 ModR/M的解析的代码实现

看了上面的总结，能手工翻译在机器码和汇编指令之间进行翻译，写出一个解析`ModR/M`的函数是一点也不困难的。要想加深对这部分的理解，自己动手写出这样一个函数是很有必要的。这里我把我理解的实现解析`ModR/M`代码的过程及其要注意的问题列在下边：（也可以在后面看完`SIB`的总结之后再动手写）

1. 提取`Opcode`中的`w`位，结合指令大小改写指令以及`CPU`当前的工作模式（16位？32位？），决定`ModR/M`编码的操作对象的大小；根据“地址大小改写”指令前缀是否出现以及`CPU`当前的工作模式决定寻址模式是16位还是32位。
2. 根据`d`位，或者指令说明，确定`ModR/M`中编码的两个操作对象哪个为源操作数，那个为目的操作数。
3. 读取`ModR/M`字节，解析成`Mod:Reg/Opcode:R/M`的形式。
4. 根据`Reg/Opcode`部分生成相应的寄存器字符串，存放在第2步中决定的（源/目的）操作数字符串中。
5. 根据寻址模式（16/32）以及`Mod`的值分别将`R/M`的编码解析成相应的字符串。（如果有`displacement`出现，而且是8位的，那么记得要对其进行符号为扩展，翻译给人看得话，就是把符号显示出来；碰到`SIB`的情况，对`SIB`进行解析；对一些特殊情况要进行特殊处理，比如32位模式下`Mod = 00, R/M = 101, 100`等情况，16位模式下`Mod = 00, R/M = 110`的情况等等）。

我的实现方式，对寄存器的编码采用字符串数组的形式，把解析到的`Reg, R/M`等的值作为索引：

```c
const char *Register32[] = {"eax", "ecx", "edx", "ebx", "esp", "ebp", "esi", "edi"};

const char *Register16[] = {"ax", "cx", "dx", "bx", "sp", "bp", "si", "di"};

const char *Register8[] = {"al", "cl", "dl", "bl", "ah", "ch", "dh", "bh"};

const char *Address16[] = {"bx+si", "bx+di", "bp+si", "bp+di", "si", "di", "bp", "bx"};
```

这里还是采用各个部分分别解析然后再合成的策略：

```c
sprintf(OperandRM, "%s %s[%s%s]", SizeStr, SegmentPrefixStr, RMStr, DisplacementStr);
```

先只考虑各个部分的解析，然后再按照一定格式合成起来。

这部分的代码（`SIB`的解析也附加上）下载（默认了`CPU`工作在32位模式下）：**[modrmsib.rar](https://bbs.kanxue.com/attach-download-19796.htm)**

运行效果：

![](https://bbs.kanxue.com/upload/attach/200810/177482_uphnwzqjjogd93i.gif)

对照`od`：

![](https://bbs.kanxue.com/upload/attach/200810/177482_p4ufgrqsky6oyyh.gif)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [modrm.gif](javascript:void(0);)
  （2.49kb，1233次下载）
- [16-bitrm2.GIF](javascript:void(0);)
  （80.82kb，1264次下载）
- [32-bitrm.GIF](javascript:void(0);)
  （76.97kb，1335次下载）
- [result.GIF](javascript:void(0);)
  （8.67kb，1235次下载）
- [odresult.GIF](javascript:void(0);)
  （12.76kb，1234次下载）
- [modrmsib.rar](javascript:void(0);)
  （17.50kb，242次下载）
