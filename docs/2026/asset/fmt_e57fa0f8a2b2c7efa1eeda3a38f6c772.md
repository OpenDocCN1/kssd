
这就出问题了，这两句并不等于 `pop r13` 指令，那么问题出在哪呢，我们取消掉变量传播优化再看看：

![](upload/attach/202305/914360\_RUVGQR328RQB7TC.png)

问题就出在，这两句 `workingTraceCode` 在进行变量溯源时，都找到了 `0x14004dca7` 这一句，

将 `PO_rsp` 放入虚拟机寄存器 `[r10 + 0xa8]`，并且在第一句 `workingTraceCode` 中，又更改了 `PO_rsp` 的值，所以导致出错。

幸运的是这种情况只会出现在 `push` 和 `pop` 中（参考`idapython/deTvm.py . tvmToAsmAll.outerror()`），

所以我们要对 `push` 和 `pop` 特殊处理，请参考：`idapython/deTvm.py . tvmToAsmAll.findPushAndPop()`

特殊处理，优化后的 `push` 和 `pop`：

`push`:

![](upload/attach/202305/914360\_35FFDX6YFFWRJWZ.png)

`pop`:

![](upload/attach/202305/914360\_KHYEFGNDYY4BC6A.png)

这就清晰很多了。

## 关于 jcc

前置知识：

各个标志位的位置：

![](upload/attach/202305/914360\_QCBUPNR8GTC69H4.png)

```
JO jmp if OF = 1
JNO jmp if OF = 0
JB JC JNAE jmp if CF = 1
JNB JNC JAE jmp if CF = 0
JZ JE jmp if ZF = 1
JNZ JNZ jmp if ZF = 0
JBE JNA jmp if CF = 1 or ZF = 1
JNBE JA jmp if CF = 0 and ZF = 0
JS jmp if SF = 1
JNS jmp if SF = 0
JP JPE jmp if PF = 1
JNP JPO jmp if PF = 0
JL JNGE jmp if SF != OF
JNL JGE jmp if SF = OF
JLE JNG jmp if ZF = 1 or SF != OF
JNLE JG jmp if ZF = 0 and SF = OF
```

tvm巧妙的利用了 `and` `sub` `setz` `je` 这四种指令模拟一个 `jcc`，例如：

### JE:

![](upload/attach/202305/914360\_3UZFARQJZRWHQ49.png)

(`v_je` 的跳转是通过 加减 `V_RIP` 实现的，我这里直接优化成 绝对地址，省了我们去计算)

上面的代码，先保留 `rflag` 的 `zf` 位，然后再减去 `zf` 位，如果结果为 0，那么 `V_RIP` 就变成 `v_je` 的第二个操作数 `0x14003c274`，否则 `V_RIP` 变为 `0x14003c1fe`

于是，上面这段 `tvmasm` 可以翻译为：

```
je xxx(V\_RIP = 0x14003c274)
jmp xxx(V\_RIP = 0x14003c1fe)
```

### JG

![](upload/attach/202305/914360\_KGZ7HDP8NK79DP4.png)

第一个 `tvmToAsm`：

如果 `zf = sf = of = 0`，则 `V_RIP = 0x1400369ec` ，否则 `V_RIP = 0x140036ac6` (其实就是下面那一段，因为优化了所以地址对不上)

第二个 `tvmToAsm`：

如果 `zf = 0` 且 `sf = of = 1`，则 `V_RIP = 0x1400369ec`，否则 `VRIP = 0x1400368ef`

于是，上面这段 `tvmasm` 可以翻译为：

```
jg xxx(V\_RIP = 0x1400369ec)
jmp xxx(V\_RIP = 0x1400368ef)
```

### 提取特征

`JE` 的特征 `0x40 0x40` （看上图你就知道是什么特征了）

`JG` 的特征 `0x8c0 0x0 0x8c0 0x880`（看上图你就知道是什么特征了）

全部的 `jcc` 特征：（代码参考`idapython/deTvm.py . tvmToAsm.vjcc_handle()`）

```
JBE JNA
0x41 0x1 0x41 0x40 （这其实是错误的，tvm的bug？）
JGE JNL
0x880 0x0 0x880 0x880
JL JNGE
0x880 0x800 0x880 0x80
JG JNLE
0x8C0 0x0 0x8C0 0x880
JZ JE
0x40 0x40
JNZ JNE
0x40 0x0
JS
0x80 0x80
JNS
0x80 0x0
JC JB JNAE
0x1 0x1
JNC JNB JAE
0x1 0x0
JA JNBE
0x41 0x0
JP JPE
0x4 0x4 0x0 0x0
JNP JPO
0x4 0x0
JO
0x800 0x800
JNO
0x800 0x0
JLE JNG
0x8C0 0x40 zf
0x8C0 0x800 of
0x8C0 0x80 sf
0x8C0 0xC0 zf+sf
0x8C0 0x840 zf+of
0x8C0 0x8C0 zf+of+sf
```

上面我说 `JBE JNA` 的特征是 `0x41 0x1 0x41 0x40` 这其实是错误的：

因为 这只是 `jmp if ZF != CF`，真正的 `JBE JNA` 是 `jmp if CF = 1 or ZF = 1`，

对应的特征应该是 `0x41 0x1 0x41 0x40 0x41 0x41`，在这个版本的 `tvm` 中 ，它将 `JBE JNA` 错误处理成了 `jmp if ZF != CF`

我逆了较新版本的 `tvm` ，`JBE JNA`这里的 bug 就被修复了，就是 `0x41 0x1 0x41 0x40 0x41 0x41`。

(看来ACE部门用的`tvm`版本不够新啊)

## 脚本还原ASM

参考`idapython/deTvm.py . tvmToAsmAll.AllTvmAsmToAsm()`

### 第一步，先识别所有的AsmOpcode:

`push` 和 `pop` 在上文已经识别出来了，

`jcc` 在上文也识别出来了，

`tvm` 没有模拟的 `Asm`，也可以通过跟踪 `v_jmp_ll` 得到，上文也说了。

一些明显的 `tvmAsm` 也可以直接识别原本的 `AsmOpcode`:

```
asmOpcode = [
"sar","or","ror","xor","shr","shl","movsxd",
"movsx","movzx","dec","inc","test",
"cmp","rep stosb","sbb","int3"
]
```

如果 `tvmToAsm` 中的 `traceCode` 的 `tvmAsm` 含有以上的字符串，

那么可以直接将这个 `tvmtoAsm` 的 `AsmOpcode` 设置为对应项，例如：

![](upload/attach/202305/914360\_UDC7T9C4XVGQJEE.png)

可以直接将 `Asm` 的 `Opcode` 设置成 `xor`，如果要翻译成 `Asm` 的话，就翻译成了 `xor edx,edx`

- 如果 `tvmToAsm` 只出现了 `and`，没有出现 `not` `add` 那么可以识别成 `and`
- 如果 `tvmToAsm` 只出现了 `not`，没有出现 `and` `add` 那么可以识别成 `not`

以上部分，参考代码：`idapython/deTvm.py . tvmToAsm.setASMOpcode_1()`

识别 `lea` `mov` `add` `sub` ，这部分比较复杂，如果是人为识别就简单。

我们先对每一个 `tvmToAsm` 内的 `traceCode` 再进行一次变量分析，列出一个赋值表，类似于上文赋值表的结构。

除此之外，我们还要对其进行标记设置，例如：

#### sub:

![](upload/attach/202305/914360\_JS8NQFDCTJMMQRN.png)

**标记的结构：[`handle`,`tvmPara`,`IsUseRflag`]，依次是处理手段、虚拟机寄存器、是否使用(输出)标志位**

```
14004a4c3 : 将 [r10 + 0xb0] 标记为 [not,PO_rsp,False]
14004a4c8 : 将 [r10 + 0xb8] 标记为 [not,PO_rsp,False],[add,0x80,True] //会拷贝前一个的标记
14004a4e4 : 将 [r10 + 0xb8] 标记为 [not,PO_rsp,False],[add,0x80,True],[not,None,False]
-优化-> [None,PO_rsp,False],[sub,0x80,True] //会拷贝前一个的标记
14004a4e9 : 将 [r10 + 0x98] 标记为 [None,PO_rsp,False],[sub,0x80,True] //会拷贝前一个的标记
```

有了这些标记，再加上 `14004a507` 这一句指令，我们就可以识别这个 `tvmToAsm` 的 `AsmOpcode` 了：

```
出现 sub ，且影响标志位，所以为 sub
```

#### add:

![](upload/attach/202305/914360\_5U6HKF9ZGB37NP5.png)

```
1400627da : [r10 + 0xa8] 标记 [None,PO_r9,False],[add,0x9,False]
1400627e1 : 同上
1400627e6 : [r10 + 0x91] 标记 [None,PO_r9,False],[add,0x9,False],[mem,None,False]
1400627eb : [r10 + 0x90] 标记 [None,PO_r9,False],[add,0x9,False],[mem,None,False],[add,PO_rax,True]
1400627ff : [r10 + 0xa8] 标记 [None,PO_r9,False],[add,0x9,False]
140062806 : 同上
```

有了这些标记，再加上 `14006280b` 这一句指令，我们就可以识别这个 `tvmToAsm` 的 `AsmOpcode` 了：

```
出现了 add 标记，并且影响标志位，只能是 add
```

#### lea:

![](upload/attach/202305/914360\_RCXDKMH68J2Z8RJ.png)

```
14004a825 : [r10 + 0xa0] 标记 [None,PO_rsp,False],[add,0x50,False]
```

有了这些标记，再加上 `14004a82c` 这一句指令，我们就可以识别这个 `tvmToAsm` 的 `AsmOpcode` 了：

```
标记中出现了add,且不影响标志位，就只能是 lea
```

#### mov:

![](upload/attach/202305/914360\_PR528USD34N263N.png)

```
140062826 : [r10 + 0x98] 标记 [None,PO_r9,False],[add,0x9,False]
14006282d : 同上
140062832 : [r10 + 0x90] 标记 [None,PO_r9,False],[add,0x9,False],[mem,None,False]
```

有了这些标记，再加上 `140062837` 这一句指令，我们就可以识别这个 `tvmToAsm` 的 `AsmOpcode` 了：

```
标记中出现了 add,但不影响标志位，并且出现了 mem ,那么只能是 mov
```

以上部分都只是一些简单例子，关于更加严格的 `sub` `add` `lea` `mov` 分类，参考代码：

`idapython/deTvm.py . tvmToAsm.record_tage()` 负责跟踪标记

`idapython/deTvm.py . tvmToAsm.setASMOpcode_2()`负责分类 `sub` `add` `lea` `mov`

### 第二步，操作数识别：

到这里的时候，全部 `tvmToAsm` 的 `AsmOpcode` 都已经全部识别。

首先介绍一个工具函数 ：`GetAsmPara` 可以将标记转换成 `Asm` 操作数的格式。

#### mov：

例子和上图是一样的。转换为标记如下：

```
140062826 : [r10 + 0x98] 标记 [None,PO_r9,False],[add,0x9,False]
14006282d : 同上
140062832 : [r10 + 0x90] 标记 [None,PO_r9,False],[add,0x9,False],[mem,None,False]
```

看到 `140062837` 这一条 `workingTraceCode`：

`v_mov_iregb_iregb ( iregb :PO_rax ,iregb :[ r10 + 0x90 ] );`

看到第二个参数的类型为 `iregb`,是 8bit 宽的操作数。
`GetAsmPara( PO_rax )` 返回 `"al"`
`GetAsmPara( [r10 + 0x90])`：
通过标记 `[None,PO_r9,False],[add,0x9,False],[mem,None,False]`，返回 `"[r9 + 0x9]"`
我们知道这个 `tvmToAsm` 的 `AsmOpcode` 为 `mov`，且 `workingTraceCode` 的第二个参数类型为 `iregb`，
那么我们就可以知道：第一个操作数为 `al`，第二个操作数为 `byte ptr[r9 + 0x9]`
于是这个 `tvmToAsm` 就可以翻译为：

```
mov al,byte ptr[r9 + 0x9]
```

代码参考：`idapython/deTvm.py . tvmToAsm.mov_handle()`

#### lea:

例子和上图是一样的。转换为标记如下：

```
14004a825 : [r10 + 0xa0] 标记 [None,PO_rsp,False],[add,0x50,False]
```

看到 `14004a82c` 这一条 `workingTraceCode`：

`v_mov_iregll_iregll ( iregll :PO_r8 ,iregll :[ r10 + 0xa0 ] );`

`GetAsmPara( PO_r8 )` 返回 `"r8"`
`GetAsmPara( [r10 + 0xa0 ])`：
通过标记 `[None,PO_rsp,False],[add,0x50,False]`，返回 `"rsp + 0x50"` //没有中括号，标记中出现`mem`才会有中括号，`lea`要后面自己加中括号
我们知道这个 `tvmToAsm` 的 `AsmOpcode` 为 `lea`,且 `workingTraceCode` 的第二个参数类型为 `iregll`,
那么我们就可以知道：第一个操作数为 `r8`，第二个操作数为 `[rsp + 0x50]`
于是这个 `tvmToAsm` 就可以翻译为：

```
lea r8,[rsp + 0x50]
```

代码参考：`idapython/deTvm.py . tvmToAsm.lea_handle()`

#### add sub:

用上面 `add` 的例子，但是进行 `GetAsmPara` 的是红框框起来的这两个：（对于 `sub` 的处理是一样的）

![](upload/attach/202305/914360\_9YFZ3J95YHKF3DS.png)

```
1400627ff : [r10 + 0xa8] 标记 [None,PO_r9,False],[add,0x9,False]
140062806 : 同上
```

对红框的两个参数进行 `GetAsmPara`：

```
看类型 iregb ,8bit宽度
GetAsmPara( PO_rax ) 返回 "al"
GetAsmPara( [r10 + 0xa8] ):
标记为：[None,PO_r9,False],[add,0x9,False]
因为 [r10 + 0xa8]类型为 ipreg，且8bit宽度
所以返回 "byte ptr[r9 + 0x9]"
于是这个tvmToAsm就可以翻译为：
add byte ptr[r9 + 0x9],al
```

代码参考：`idapython/deTvm.py . tvmToAsm.add_sub_handle()`

**以下都是 对 红框框框起来的 `tvmPara` 进行 `GetAsmPara`**

#### push:

![](upload/attach/202305/914360\_EKZC9TCHM5D2EKJ.png)

代码参考：`idapython/deTvm.py . tvmToAsm.push_handle()`

#### pop:

![](upload/attach/202305/914360\_D7A6W5B5AYVES77.png)

代码参考：`idapython/deTvm.py . tvmToAsm.pop_handle()`

#### cmp test:

`cmp`:

![](upload/attach/202305/914360\_4JP6M8XWG6TUEQW.png)

`test`:

![](upload/attach/202305/914360\_WPDCXR6G2M46HDA.png)

代码参考：`idapython/deTvm.py . tvmToAsm.cmp_test_handle()`

#### movzx movsx movsxd

举例如果 `opcode` 是 `movzx` ：

![](upload/attach/202305/914360\_ZMB3WR4T6QTXGW6.png)

其余的一样，取 `opcode` 对应的第二个参数，然后 `workingTraceCode` 的第一个参数进行 `GetAsmPara`

代码参考：`idapython/deTvm.py . tvmToAsm.movzx_movsx_movsxd_handle()`

#### not
