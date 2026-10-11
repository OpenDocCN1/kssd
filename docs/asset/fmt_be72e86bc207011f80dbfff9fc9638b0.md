
# 对程序流程分析和优化熟悉的 TX 进来谈谈，如何对付 switch 做的混淆

> 原文：[`www.pediy.com/kssd/pediy10/75503.html`](https://www.pediy.com/kssd/pediy10/75503.html)

`siwtch` 混淆就是指利用 `switch` 指令将程序流程打乱（MSIL 中的 `switch` 可以理解为 C 中的 `switch..case`），然后利用一个 `int32` 常数变量，给该变量不同的值代表不同的执行顺序，最终保证原程序的正确执行。但这种混淆后的 .net 程序是无法反编译为高级语言的。

有没有熟悉这方面的朋友，谈谈反混淆有什么好的方法，或是好的算法。

示例代码如下：

```
L_0000: br.s L_004f

L_0002: ldloc num2 //num2 就是 switch 的循环变量

L_0006: switch (L_010e, L_0137, L_0165, L_0193, L_0089, ....//等等)

L_004f: nop

L_0050: ldc.i4 9 //num2=9

L_0055: stloc num2

L_0059: br.s L_0002 //跳回 switch

L_005b: ldsfld bool io::g

L_0060: brtrue.s L_00d7

L_0062: ldc.i4 14 //num2=14

L_0067: stloc num2

L_006b: br.s L_0002 //跳回 switch
```

这种混淆对流程的影响可以在 IDA 的流程图中看出来：

![](img/05c45e3d2c39abaf0019a59f9f00232d.jpg)

只是个 DFA 吧，直接根据每个 `stloc num2` 得到流程不行吗

代码:

```python
import re, string
s = '''
L_0000: br.s L_004f
L_0002: ldloc num2
L_0006: switch (L_010e, L_0137, L_0165, L_0193, L_0089, L_case5, L_case6, L_case7, L_case8, L_case9, L_case10, L_case11, L_case12, L_case13, L_case14)
L_004f: nop 
L_0050: ldc.i4 9
L_0055: stloc num2
L_0059: br.s L_0002
L_005b: ldsfld bool io::g
L_0060: brtrue.s L_00d7
L_0062: ldc.i4 14
L_0067: stloc num2
L_006b: br.s L_0002
'''
nl = '\n\*L\_\[0-9a-f\]+:\s+'
xl = '\n\*L_([0-9a-f]+):\s+'
path = re.findall( xl + 'ldc.i4 (\d+)'
                 + nl + 'stloc num2'
                 + nl + 'br.s', s)
t, = re.findall(nl + 'switch \(([^)\]+)\)', s)
handlers = map(string.strip, t.split(','))
for i, j in path:
    print 'L_%s: br.s %s' % (i, handlers[int(j)])
#OUTPUT
#L_0050: br.s L_case9
#L_0062: br.s L_case14
```

贴的代码太少, 不知道具体是啥样, 按这个结果重写一下 `br.s` 就变成标准的流程混乱了

我在一楼贴的代码，相对还有顺序可遵循，forgot 给出的方法应该可行。但如果要通用，还得考虑更多情况。最基本的，要分辨原程序中真的 `switch` 和混淆软件后添加的 `switch`，甚至有的 `switch` 中的跳转表、部分是真的，部分是后加上去的，这时就不能简单的把 `switch` 删除。

有时还有其它情况，下面再给个代码

```
L_0001: ldc.i4 0x11 //这里是首次跳进 switch 时 num2 的值

L_0006:br L_007e  //如何确定这种首次跳转 switch 的指令？有时 br 会跳至 switch 之前，有时跳至 switch 之后。

...

L_007e: stloc num2

L_0082: ldloc num2

L_0086: switch (L_0137, L_0035, L_02a6, L_02e3, L_01ea, L_034d, L_0286, //等等)

L_0113: ldc.i4 9

L_0118: stloc num2

L_011c: call bool fw1EVUdHk05sAm1BhHQ.1E3tqPdlbHrwXtZ1nbw::m2f40ZKjNxjvXPpu1CDQ()

L_0121: brtrue L_0082 //存在不定的跳转情况

L_0126: ldc.i4.0 

L_0127: stloc.1 

L_0128: ldc.i4 30

L_012d: stloc num2

L_0131: ldc.i4.1 

L_0132: brtrue L_0082 // if(1==true) br，这种情况倒是好处理

...

L_026b: ldc.i4 14

L_0270: br L_0086 //不经过 num2，直接跳转

...

L_037f: ldc.i4 11

L_0384:br L_007E //不跳到 L_0082 了，先跳再给 num2 赋值
```

先列这么多，forgot 给出出主意。

复杂的情况需要生成一个树，结点类似 `{ next, branches, code }`，

相连的代码从一侧可以遍历，找到顺序出现的

`ldc XX`

`stloc num2`

`ldloc num2`

`switch`

可以判断是回归到 `switch`。`fake jcc` 可以通过遍历树发现后调整为 `next`。

关于条件跳转，不知道情况多复杂，如果只是个 `call` 人肉加个模式就行，否则就非常麻烦要分析堆栈。

即使真的 `switch` 也可能是个状态机，也能按这个整理。
