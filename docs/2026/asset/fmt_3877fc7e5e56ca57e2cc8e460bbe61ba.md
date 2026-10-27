
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
| 1  2  3  4  5  6  7  8  9 | `140059dc2` `: v_mov_iregb_b                      ( iregb  :[ r10` `+` `0x90` `] ,b      :``0x6`             `);`  `140059dc6` `: v_mov_iregll_ll                    ( iregll :[ r10` `+` `0x98` `] ,ll     :``0x1`             `);`  `140059dd1` `: v_mov_iregll_ll                    ( iregll :[ r10` `+` `0xa0` `] ,ll     :``0x1`             `);`  `140059ddc` `: v_mov_iregll_ll                    ( iregll :[ r10` `+` `0xa8` `] ,ll     :``0x600`           `);`  `140059de7` `: v_mov_iregll_ll                    ( iregll :[ r10` `+` `0x0` `]  ,ll     :``0x140001250`     `);`  `140059df2` `: v_mov_iregll_iregll                ( iregll :[ r10` `+` `0xa8` `] ,iregll :PO_rsp          );`  `140059df7` `: v_mov_iregll_ll                    ( iregll :[ r10` `+` `0xa0` `] ,ll     :``0x28`            `);`  `140059e02` `: v_not_oregll_iregll                ( oregll :[ r10` `+` `0xb0` `] ,iregll :[ r10` `+` `0xa8` `]  );`  `140059e07` `: v_add_oregll_iregll_iregll_oregl   ( oregll :[ r10` `+` `0xb8` `] ,iregll :[ r10` `+` `0xb0` `] ,iregll :[ r10` `+` `0xa0` `] ,oregl  :PO_rf           );` |
```

我就可以得到这么一张表：

![](https://bbs.kanxue.com/upload/attach/202305/914360_9RTWEG7Q3QPHFAY.png)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `traceTaskRegTable`为总赋值表，`traceTaskRegList`为单个虚拟机寄存器的赋值栈，`traceTaskReg`为入栈的单个记录。相关定义参考：`idapython`/`deTvm.py`的三个类，命名同上 |

当需要检索这一句`traceCode`的相关`traceCode`时：

|  |  |

| --- | --- |

| 1 | `140059e02` `: v_not_oregll_iregll                ( oregll :[ r10` `+` `0xb0` `] ,iregll :[ r10` `+` `0xa8` `]  );` |

就可以直接查表，先找到`[ r10 + 0xa8 ]`的赋值记录栈`traceTaskRegList`，然后通过地址找到最近的一次赋值`traceTaskReg`，然后`traceTaskReg`记录了这一句`traceCode`，就可以找到了。

于是就找到了相关`traceCode`:

|  |  |

| --- | --- |

| 1 | `140059df2` `: v_mov_iregll_iregll                ( iregll :[ r10` `+` `0xa8` `] ,iregll :PO_rsp          );` |

因为是将`PO_reg`赋值给它，所以到此检索完毕，如果不是，则按照相同的方法继续往上找。

相关代码参考：`idapython/deTvm.py . traceTask.VRegRecord()`

###### 一些特殊的例子

例如有两个连续的` tvmToAsm`，导出的相关`traceCode`如下：

![](https://bbs.kanxue.com/upload/attach/202305/914360_AAET6ANRNGUYDQG.png)

我们可以发现，有两段相关`traceCode`是相同的，这就出现问题了：

第一个`tvmToAsm`可以翻译成 `xor edi,edi`，那么第二个`tvmToAsm`能翻译成什么呢？

如果我们人为识别，就可以将其翻译成 `mov ecx,edi`，因为 `[r10 + 450]`在前面已经放入了 `PO_rdi`，下面又取它放入 `PO_rcx`。

于是我们可以进行优化，如果 有一句 `v_mov_iregll_iregll(PO_reg , xxxx)`那么我们就可以将 `xxxx`的上一次赋值标记为`PO_reg`，

还是以上面的那一段代码为例：

|  |  |

| --- | --- |

| 1 | `v_mov_iregll_iregll                ( iregll:PO_rdi         ,iregll:[ r10` `+` `0x450` `] ); <``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-` |

`[r10 + 0x450]`上一次赋值语句为：

|  |  |

| --- | --- |

| 1 | `v_mov_iregll_iregl                 ( iregll:[ r10` `+` `0x450` `],iregl :[ r10` `+` `0x98` `]  );` |

我们可以将其标记为 `PO_rdi`，那么当其他的`workingTraceCode`向上进行查找赋值表的时候，就可以找到`PO_rdi`，于是就优化为了：

![](https://bbs.kanxue.com/upload/attach/202305/914360_VMJTMXB86TSMKB6.png)

实现代码参考：`idapython/deTvm.py . traceTask.VRegRecord()`

###### 关于push和pop

push:

![](https://bbs.kanxue.com/upload/attach/202305/914360_7XBA4A9MAWV7257.png)

这是两个连续的 `tvmToAsm`，

如果按照一般的分析方式，那么这两句可以翻译为：

|  |  |

| --- | --- |

| 1  2 | `mov qword ptr[rsp` `-` `0x8``],r13`  `lea rsp,[rsp` `-` `8``]` |

乍一看没什么问题，就是将 `push r13`分开成两句执行，可是如果是`pop`，那么情况就有点不同了：

pop:

![](https://bbs.kanxue.com/upload/attach/202305/914360_SCWY9KKZFATHRYK.png)

如果按照一般的分析方式，那么这两句可以翻译为:

|  |  |

| --- | --- |

| 1  2 | `lea rsp,[rsp` `+` `8``]`  `mov r13,qword ptr[rsp]` |

这就出问题了，这两句并不等于 `pop r13` 指令，那么问题出在哪呢，我们取消掉变量传播优化再看看：

![](https://bbs.kanxue.com/upload/attach/202305/914360_RUVGQR328RQB7TC.png)

问题就出在，这两句 `workingTraceCode`在进行变量溯源时，都找到了 `0x14004dca7` 这一句，

将 `PO_rsp` 放入虚拟机寄存器 `[r10 + 0xa8]`，并且在第一句`workingTraceCode`中，又更改了 `PO_rsp`的值，所以导致出错。

幸运的是这种情况只会出现在 `push` 和 `pop` 中（参考`idapython/deTvm.py . tvmToAsmAll.outerror()`），

所以我们要对 `push` 和 `pop` 特殊处理，请参考：`idapython/deTvm.py . tvmToAsmAll.findPushAndPop()`

特殊处理，优化后的 `push` 和 `pop`：

push:

![](https://bbs.kanxue.com/upload/attach/202305/914360_35FFDX6YFFWRJWZ.png)

pop:

![](https://bbs.kanxue.com/upload/attach/202305/914360_KHYEFGNDYY4BC6A.png)

这就清晰很多了。

### 关于 jcc

前置知识：

各个标志位的位置：

![](https://bbs.kanxue.com/upload/attach/202305/914360_QCBUPNR8GTC69H4.png)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `JO                jmp` `if` `OF` `=` `1`  `JNO                jmp` `if` `OF` `=` `0`  `JB JC JNAE        jmp` `if` `CF` `=` `1`  `JNB JNC JAE        jmp` `if` `CF` `=` `0`  `JZ JE            jmp` `if` `ZF` `=` `1`  `JNZ JNZ            jmp` `if` `ZF` `=` `0`  `JBE JNA            jmp` `if` `CF` `=` `1` `or`  `ZF` `=` `1`  `JNBE JA            jmp` `if` `CF` `=` `0` `and` `ZF` `=` `0`  `JS                jmp` `if` `SF` `=` `1`  `JNS                jmp` `if` `SF` `=` `0`  `JP JPE            jmp` `if` `PF` `=` `1`  `JNP JPO            jmp` `if` `PF` `=` `0`  `JL JNGE            jmp` `if` `SF !``=` `OF`  `JNL JGE            jmp` `if` `SF` `=` `OF`  `JLE JNG            jmp` `if` `ZF` `=` `1` `or` `SF !``=` `OF`  `JNLE JG            jmp` `if` `ZF` `=` `0` `and` `SF` `=` `OF` |

`tvm`巧妙的利用了`and sub setz je`这四种指令模拟一个`jcc`，例如：

###### JE:

![](https://bbs.kanxue.com/upload/attach/202305/914360_3UZFARQJZRWHQ49.png)

(`v_je` 的跳转是通过 加减 `V_RIP` 实现的，我这里直接优化成 绝对地址，省了我们去计算)

上面的代码，先保留 `rflag` 的`zf`位，然后再减去`zf`位，如果结果为0，那么 `V_RIP` 就变成 `v_je` 的第二个操作数 `0x14003c274`否则 `V_RIP`变为`0x14003c1fe`

于是，上面这段`tvmasm`可以翻译为：

|  |  |

| --- | --- |

| 1  2 | `je xxx(V_RIP` `=` `0x14003c274``)`  `jmp xxx(V_RIP` `=` `0x14003c1fe``)` |

###### JG

![](https://bbs.kanxue.com/upload/attach/202305/914360_KGZ7HDP8NK79DP4.png)

第一个`tvmToAsm`：

如果 `zf = sf = of = 0`，则`V_RIP =` `0x1400369ec` ，否则 `V_RIP =` `0x140036ac6` (其实就是下面那一段，因为优化了所以地址对不上)

第二个`tvmToAsm`：

如果 `zf = 0` 且 `sf = of = 1`，则`V_RIP =` `0x1400369ec`，否则 `VRIP =` `0x1400368ef`

于是，上面这段`tvmasm`可以翻译为：

|  |  |

| --- | --- |

| 1  2 | `jg xxx(V_RIP` `=` `0x1400369ec``)`  `jmp xxx(V_RIP` `=` `0x1400368ef``)` |

###### 提取特征

`JE` 的特征 `0x40 0x40` （看上图你就知道是什么特征了）

`JG` 的特征 `0x8c0 0x0 0x8c0 0x880`（看上图你就知道是什么特征了）

全部的`jcc`特征：（代码参考`idapython/deTvm.py . tvmToAsm.vjcc_handle()`）

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52 | `JBE JNA`  `0x41` `0x1` `0x41` `0x40` `（这其实是错误的，tvm的bug？）`    `JGE JNL`  `0x880` `0x0` `0x880` `0x880`    `JL JNGE`  `0x880` `0x800` `0x880` `0x80`    `JG JNLE`  `0x8C0` `0x0` `0x8C0` `0x880`    `JZ JE`  `0x40` `0x40`    `JNZ JNE`  `0x40` `0x0`    `JS`  `0x80` `0x80`    `JNS`  `0x80` `0x0`    `JC JB JNAE`  `0x1` `0x1`    `JNC JNB JAE`  `0x1` `0x0`    `JA JNBE`  `0x41` `0x0`    `JP JPE`  `0x4` `0x4` `0x0` `0x0`    `JNP JPO`  `0x4` `0x0`    `JO`  `0x800` `0x800`    `JNO`  `0x800` `0x0`    `JLE JNG`  `0x8C0` `0x40`      `zf`  `0x8C0` `0x800`     `of`  `0x8C0` `0x80`         `sf`  `0x8C0` `0xC0`         `zf``+``sf`  `0x8C0` `0x840`     `zf``+``of`  `0x8C0` `0x8C0`     `zf``+``of``+``sf` |

上面我说`JBE JNA` 的特征是`0x41 0x1 0x41 0x40`这其实是错误的：

因为 这只是 `jmp if ZF != CF`，真正的 `JBE JNA` 是 `jmp if CF = 1 or ZF = 1`，

对应的特征应该是 `0x41 0x1 0x41 0x40 0x41 0x41`，在这个版本的 `tvm` 中 ，它将 `JBE JNA`错误处理成了`jmp if ZF != CF`

我逆了较新版本的 `tvm` ，`JBE JNA`这里的`bug`就被修复了，就是`0x41 0x1 0x41 0x40 0x41 0x41`。

(看来`ACE`部门用的`tvm`版本不够新啊)

### 脚本还原ASM

参考`idapython/deTvm.py . tvmToAsmAll.AllTvmAsmToAsm()`

##### 第一步，先识别所有的AsmOpcode:

`push` 和 `pop` 在上文已经识别出来了，

`jcc` 在上文也识别出来了，

`tvm`没有模拟的`Asm`，也可以通过跟踪 `v_jmp_ll` 得到，上文也说了。

一些明显的 `tvmAsm`也可以直接识别原本的`AsmOpcode`:

|  |  |

| --- | --- |

| 1  2  3  4  5 | `asmOpcode` `=` `[`  `"sar"``,``"or"``,``"ror"``,``"xor"``,``"shr"``,``"shl"``,``"movsxd"``,`  `"movsx"``,``"movzx"``,``"dec"``,``"inc"``,``"test"``,`  `"cmp"``,``"rep stosb"``,``"sbb"``,``"int3"`  `]` |

如果`tvmToAsm`中的`traceCode`的`tvmAsm`含有以上的字符串，

那么可以直接将这个`tvmtoAsm`的`AsmOpcode`设置为对应项，例如：

![](https://bbs.kanxue.com/upload/attach/202305/914360_UDC7T9C4XVGQJEE.png)

可以直接将`Asm`的`Opcode` 设置成 `xor`，如果要翻译成`Asm`的话，就翻译成了 `xor edx,edx`

|  |  |

| --- | --- |
