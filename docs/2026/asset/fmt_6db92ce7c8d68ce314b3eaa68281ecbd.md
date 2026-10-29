
我们可以看到,最后一条指令是状态变量与`0x76579ace`进行比较,相等的情况下走`0x70688`（假分支）,这个地址就是`case 0x76579aced`对应代码区域的起始地址.

```
 423 @ 000710c0  if (x8_26.w8 == 0xdcf244e1) then 439 @ 0x71080 else 444 @ 0x710c4

```

对于这个测试, `0xdcf244e1`对应的`case`入口是`0x71080`（真分支）

用这个方式来获取`case`代码区域的起始地址是比较可靠的.

因为`llvm`在给`switch`生成代码时,据我观察,在无法使用跳转表的情况下,会转到使用二分查找实现,查找到最后一般都会有这么一条测试`case`编号的指令.

找到`case`出口也比较简单:`包含对状态变量进行更新基本块的出口就是case出口`.

```
 104 @ 000707bc  aop_init()

 105 @ 000707c4  uint64_t x8_1 = 0xc88d21cd

 106 @ 000707c8  goto 10 @ 0x704f0

```

实际上,现实代码中会存在有多个出口的情况,这个问题在修复的时候再讨论.

如果是条件跳转,我们还需要知道跳转目标与测试条件的对应关系.

```
 134 @ 00070854  bool cond:12_1 = x0_4.w0 != 0

 135 @ 00070868  if (cond:12_1) then 144 else 146

 ---

 144 @ 00070868  uint64_t x8_1 = 0x91c5439 ; 真分支

 145 @ 00070868  goto 155 @ 0x7086c ; 跳转到分发器

 ---

 146 @ 00070868  uint64_t x8_1 = 0xf291f0e9 ; 假分支

 147 @ 00070868  goto 155 @ 0x7086c ; 跳转到分发器

```

`case 0x91c5439`对应的条件可以通过查看入口边的类型得到

```
def resolve_branch_condition(state):

    il_bb = state.il_basic_block

    assert len(il_bb.incoming_edges) <= 1

    if len(il_bb.incoming_edges) == 1:

        return il_bb.incoming_edges[0].type == BranchType.TrueBranch

    else:

        return True

```

到这反混淆所需信息就完全获取到了.

以上我主要说明了思路, 省略了很多细节.

例如`cff`中下面这种`case`

```
  62 @ 00000814  bool cond:5_1 = x19_1.w19 == 0xcb0c4ceb

  63 @ 0000081c  uint64_t x7_1 = 0xd40c7864

  64 @ 00000820  uint64_t x20 = zx.q(x1.w1)

  65 @ 00000824  if (cond:5_1) then 16 @ 0x768 else 76 @ 0x828

```

这个基本块既是`case`入口,又是出口,还是分发器代码的一部分.

`vdog JNI_OnLoad`里面的一个`case`:

```
 313 @ 00070e54  bool cond:60_1 = x9.w9 != x8_22.w8

 314 @ 00070e58  uint64_t x8_1 = zx.q(x9.w9)

 315 @ 00070e5c  if (cond:60_1) then 10 @ 0x704f0 else 320 @ 0x70e64

```

`313`处, 在分发器里面,状态变量(`x9.w9`)跟另外一个变量比较(`x8_22.w8`), 这时候`x8_22.w8`的值是多少?

上面两个问题我在代码里面都有处理,有兴趣可以参考代码.

即使没有处理也没有关系,我反混淆的修复是保守的,对于无法处理的情况,仍然会走原来逻辑,只是最终反混淆效果会差一点.

下面开始修复部分内容,修复难度在我看来要比分析难. 修复`cff`这个看起来简单的样本要比`vdog`难.

# 修复

代码修复主要有两个难点:

- `patch`二进制

- 处理编译器优化移动的代码

## `patch`二进制

修复时,在代码中寻找用来`patch`代码空洞还是比较困难的.

之前考虑使用`IDA`微码进行反混淆,这是最主要的原因.

`llvm-deobfuscator`是把混淆框架代码作为空洞使用.

而我反混淆目标是为了在`IDA`中, 使用他的反编译器进行静态分析, 不需要创建一个新的可执行的二进制, 所有我是用`IDA`创建一个新的代码段来存放`patch`代码.

这种处理方法能让我保留原来的分发器,有意的保留分发器, 可以在未识别到真实后继基本块或者`case`出入口的情况下,走原来分发器流程.

如果所有状态的后继都被我处理了, 分发器就变成不可达代码, `IDA`的反编译器会自动把它优化掉.

## 处理编译器优化移动的代码

目前看到的反混淆文章好像都没有提到如何处理这个问题的.

如下函数:

```
void foo1()

{

    int state_var = 0;

    while(1) {

        switch (state_var) {

            case 0: {

                        puts("enter case 0");

                        state_var = 1;

                        break;

                    }

            case 1: {

                        puts("enter case 1 or 2");

                        puts("enter case 1");

                        state_var = 2;

                        break;

                    }

            case 2: {

                        puts("enter case 1 or 2");

                        puts("enter case 2");

                        state_var = 3;

                        break;

                    }

            case 3: {

                        puts("enter case 3");

                        state_var = 4;

                        break;

                    }

        }

        if (state_var == 4) {

            break;

        }

    }

}

```

有可能会被编译器优化成:

```
void foo2()

{

    int state_var = 0;

    while(1) {

        switch (state_var) { // 分发器入口

            case 0: {

                        puts("enter case 0");

                        state_var = 1;

                        break;

                    }

            case 1: case 2: {

                           puts("enter case 1 or 2"); // 1, 如何处理这部分代码?

                           if (state_var == 1) {

                               puts("enter case 1");  // case 1 入口

                               state_var = 2;

                           } else {

                               puts("enter case 2");  // 2, 用上文提到的方法只能识别到case 1的入口, 怎么确定这是case 2的入口?

                               state_var = 3;

                           }

                           puts("exit case 1 or 2");  // 3, 如何处理这部分代码?

                           break;

                       }

            case 3: {

                        puts("enter case 3");

                        state_var = 4;

                        break;

                    }

        }

        if (state_var == 4) {

            break;

        }

    }

}

```

我解决的办法是通过代码拷贝,把`foo2`转换成`foo1`.具体做法是

- 拷贝从分发器入口,到`case`入口的所有代码到`patch`代码区域,并在执行`case`代码前,执行这些复制过来的指令.

- 同样的,在离开`case`时, 拷贝`case`出口到分发器入口所有代码到`patch`代码区域,并在执行后继`case`代码前,执行这些复制过来的指令.

问题`2`, 属于分析阶段的问题, 我现在还没有处理.

当前反混淆方案会在进入`case 1`之后, 由于`state 2`对应`case 2`的入口没有识别到, 控制流会重新进入分发器, 从分发器进入`case 2`.

现实代码中,还会出现下面的情况

```
CMP             W9, #0

CSET            W9, GT

EOR             W8, W9, W8

TBNZ            W8, #0, to_dispatcher ; 出口1

CBZ             W0, to_dispatcher     ; 出口2

...

CMP             W8, #0x30 ; '0'

MOV             W8, #0xdeadbeef     ; next case

CSEL            W24, W24, W8, EQ

B               to_dispatcher       ; 出口3

```

`case`有多个出口情况,伪码:

```
...

while(1) {

    switch (state_var) {

    ....

        case x: {

                    state_var = next_case; // 设置跳转目标

                    // 真实后继case只有一个, 但这个case有多个出口

                    if (cond) {

                        puts("exit 1");

                        break;

                    }

                    puts("exit 2");

                    break;

                }

    ....

}

```

为了处理这种代码, 我们在修复的时候, 需要找到`case`的所有出口,在每个出口处都进行`patch`.

综上,我们需要进行如下修复操作:

- 找到`case`的所有出口

- `patch case` 的出口, 使其跳转到我们的`path`代码

- 拷贝`patch case` 出口时覆盖掉的指令到`patch`代码区域

- 拷贝`case`出口到分发器入口所有代码到`patch`代码区域

- 拷贝从分发器入口,到`case`入口的所有代码到`patch`代码区域

- 使用汇编器,生成从`patch`代码到真实目标跳转指令

修复后的代码有如下两种布局.

- 只有一个跳转目标的代码

```
;原始代码

tbnz w8,  #0, to_dispatcher ; 出口1

cont:

cbz  w0, to_dispatcher     ; 出口2

```

```
;修复出口1后的代码

b   patch_code ;  跳转到我们的patch代码

cont:

cbz w0, to_dispatcher     ; 出口2



; patch 代码区域

patch_code:

tbnz w8, #0, to_case_entry ; 准备进入真实目标

b cont ; 返回

to_case_entry:

patch时覆盖掉的指令

离开当前基本块到进入分发器的所有指令

离开分发器到目标case的所有指令

跳转到目标case

```

- 有两个跳转目标的代码

```
这个基本块位于cff 0x00000798

csel    w7, w15, w14, eq

mov     w20, w0

b       to_dispatcher

```

```
cset    w7, eq ; !!!不能在此直接跳转到patch代码, 下面的mov指令是活跃的!!!

mov     w20, w0

b       patch_code



; patch代码区域

patch_code:

cbnz    w7, true_branch (0x8)

cbz     w7, false_branch



true_branch:

patch时覆盖掉的指令

离开当前基本块到进入分发器的所有指令

离开分发器到真分支入口的所有指令

跳转到真分支





false_branch:

patch时覆盖掉的指令

离开当前基本块到进入分发器的所有指令

离开分发器到假分支的所有指令

跳转到假分支

```

下面是两个具体的修复实例.

- 修复`cff`函数入口跳转到真实目标

`cff`样本的分发器位于`0x00000768`

分发器入口代码

```
MOV             W0, W20 ; W0活跃

MOV             W19, W7

CMP             W19, W10

```

这个块内的`W0`是函数返回值,不可以删除这条指令.

从函数入口到真实后继的修复代码:

```
loc_4000000

MOV             W0, W20 ; 复制的dispatcher代码

MOV             W19, W7

CMP             W19, W10



CMP             W19, W11 

CMP             W19, W8

MOV             W7, W19

MOV             W20, W0

B               loc_7D0 ; 0x2495548b 的后继

```

可以看到,修复代码复制了分发器到`case`入口路径上的所有指令.

- 修复`vdog`

再来看下`vdog`中起始地址为`0x000704dc`的基本块

```
STR             W9, [X19,#0x1C]

B               loc_704F0 ; 跳转到分发器

```

从函数入口到真实后继的修复代码:

```
loc_4000000



STR             W9, [X19,#0x1C] ; 0x000704dc基本块指令



MOV             W9, W8 ; 分发器指令

CMP             W9, W27

CMP             W9, W23

CMP             W9, W26

CMP             W9, W20

MOV             W8, #0x76579ACD

CMP             W9, W8

...

MOV             W8, #0x667472C5

CMP             W9, W8

MOV             W8, W9

B               loc_707E0 ; 0x667472c5对应的case入口

```

`0x000704dc`基本块另外一个前驱

```
loc_704CC

LDR             X8, [X19,#0x80]

MOV             W8, #0x6FA40FC9

LDR             W9, [X19,#0x18] ; 该指令将被覆盖

; 下落到0x000704dc

STR             W9, [X19,#0x1C]

B               loc_704F0 ; 跳转到分发器

```

修复后:
