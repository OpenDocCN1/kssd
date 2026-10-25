
# [原创]Angr基础笔记

最近学习`angr`，整理了一下相关资料内容。

## Angr

官方文档：

<mark class="encrypted">2beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3N6J5i4K6u0r3j5h3&6Y4M7W2)9J5k6r3c8G2j5H3`.`.</mark>

<mark class="encrypted">df4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3N6J5i4K6u0W2K9h3!0Q4x3V1k6S2M7r3W2Q4x3X3c8V1L8$3y4Q4x3V1j5`.</mark>

<mark class="encrypted">67dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2j5h3&6Y4M7W2)9J5k6h3W2G2i4K6u0r3</mark>

## Angr架构简介

![](upload/attach/202001/821225_5RUKSMBU23RGSFH.jpg)

- `CLE`（CLE Loading Everything）。`angr`工具第一步就是将二进制文件加载到`cle.loader`类里，`.loader`类加载所有的对象并导出一个进程内存的抽象，生成该程序已加载和准备运行的地址空间。具体的源码分析可以<a href="elink@6ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8G2N6h3c8G2N6h3c8G2N6i4A6G2N6h3I4W2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5e0x3#2y4o6b7K6y4R3`.`.">参考这里</a>

```
>>> print b.loader.find_symbol_got_entry('__libc_start_main')
>>> print b.loader.main_bin.imports
{'__gmon_start__': <cle.elf.ELFRelocation at 0x7f9928941650>,
'__libc_start_main': <cle.elf.ELFRelocation at 0x7f9928941dd0>,
'__stack_chk_fail': <cle.elf.ELFRelocation at 0x7f9928941590>,
'fgets': <cle.elf.ELFRelocation at 0x7f9928941550>,
'getenv': <cle.elf.ELFRelocation at 0x7f9928406810>,
'printf': <cle.elf.ELFRelocation at 0x7f99284062d0>,
'ptrace': <cle.elf.ELFRelocation at 0x7f99286cca10>,
'puts': <cle.elf.ELFRelocation at 0x7f99284068d0>}
```

- `PyVEX`
  - 将指令转换成中间语言（IR）、分析 IR 并且模拟
  - i.e., 不只知道他是什么，还知道他做了什么
  - `state`、`symbolic memory`、`SimProcedure`
- `Claripy`
  - 设置符号变量以及`solver`、收集限制式（约束条件）
  - 是一个前端界面，而后端可以是各种`solver`，比如`z3`
- `angr` & `surveyors`
  - 一整个集成符号执行
  - `path`、`path_group`、`factory`、...

## 脚本示例

对于这样的题目

![](upload/attach/202001/821225_CG6XJSMQBUUD27X.jpg)

![](upload/attach/202001/821225_S8ZPFYJB92Q98JG.jpg)

我们有多种方式写：

### Surveyors

通过构建`Surveyors`的方法

```
import angr
p = angr.Project("test")
ex = p.surveyors.Explorer(find=(0x400844, ), avoid=(0x400855,))
ex.run()
print ex.found[0].state.posix.dumps(0)
```

### path_group

```
import angr
p = angr.Project("test")
initial_state = p.factory.entry_state()
pg = p.factory.path_group(initial_state)
pg.explore(find=(0x4005d1,))
print pg
# <PathGroup with 18 deadended, 4 active, 1 found>
print pg.found[0]
# <Path with 64 runs (at 0x4005d1)>
print pg.found[0].state.posix.dumps(0)
# input_string
```

### SimState

- `entry_state()`：一个`SimState`，初始化为程序在二进制入口点时的状态
- `blank_state()`：一个初始化很少的`SimState`对象

```
>>> import angr
>>> b = angr.Project('/bin/true')
>>> s = b.factory.blank_state(addr=0x08048591)
>>> s = b.factory.entry_state()
# The first 5 bytes of the binary
>>> print s.memory.load(b.loader.min_addr(), 5)
```

## 如何设置 `args`

```
import angr
import claripy
p = angr.Project("test")
args = claripy.BVS('args', 8*16)
initial_state = prog.factory.entry_state(args=["./vul", args])
pg = p.factory.path_group(initial_state)
pg.explore(find=(0x4005d1,))
print pg
# <PathGroup with 18 deadended, 4 active, 1 found>
print pg.found[0]
# <Path with 64 runs (at 0x4005d1)>
print pg.found[0].state.posix.dumps(0)
# input_string
```

## Claripy后端

```
# Create a 32-bit symbolic bitvector "x"
>>> claripy.BVS('x', 32)
# Create a 32-bit bitvectory with the value 0x12345678
>>> claripy.BVV(0x12345678, 32)
<BV32 BVV(0x12345678, 32)>
```

- 建立一个32bit的符号值容器`"x"`：
  `claripy.BVS('x',32)`
- 建立一个32bit的具体值(`0xc001b3475`)容器：
  `claripy.BVV(0xc001b3a75,32)`
- 建立一个32bit的步进值，从1000到2000能被10整除的数：
  `claripy.SI(name='x',bits=32,lower_bound=1000,upper_bound=2000,stride=10)`
- 如果两个`bitvector`的长度不同是不能共同运算的，需要将其扩展：

```
# 64-bit bitvectors with concrete values 1 and 100
>>> one = state.solver.BVV(1, 64)
>>> one
<BV64 0x1>
# create a 27-bit bitvector with concrete value 9
>>> weird_nine = state.solver.BVV(9, 27)
>>> weird_nine
<BV27 0x9>
>>> weird_nine.zero_extend(64 - 27)
<BV64 0x9>
>>> one + weird_nine.zero_extend(64 - 27)
<BV64 0xa>
```

## Memory Access

**在内存位置上放符号变量**

方便我们追踪并求解内存位置上的值

```
import angr
p = angr.Project('./vul')
s = p.factory.blank_state(addr=0x80485c8)
bvs = s.se.BVS('to_memory', 8*4)
s.se.add(bvs > 1000)
s.memory.store(0x08049b80, bvs, endness='Iend_LE')
pg = p.factory.path_group(s, immutable=False)
```

注意这里的`s.se.add(bvs > 1000)`，构造了一个符号限制表达式。类似的逻辑限制表达式如下表：

![](upload/attach/202001/821225_GN66JDPS7UY7FTB.jpg)

获取内存数据

`s.se`是`state`的一个solver engine

```
# get the integer
>>> print s.se.any_int(s.regs.rax)
# get the string
>>> print s.se.any_str(s.memory.load(0x1000, 10, endness='Iend_LE'))
# storing data
>>> s.regs.rax = aaaa
>>> s.memory.store(0x1000, aaaa, endness='Iend_LE')
>>> s.memory.store(s.regs.rax, aaaa, endness='Iend_LE')
```

## Posix

给输入的值（`stdin`）加上限制

```
p = angr.Project('./vul')
st = p.factory.full_init_state(args=['./vul'])
# Constrain the first 28 bytes to be non-null and non-newline
for _ in xrange(28):
    k = st.posix.files[0].read_from(1)
    st.se.add(k != 0)
    st.se.add(k != 10)
# Constrain the last byte to be a newline
k = st.posix.files[0].read_from(1)
st.se.add(k == 10)
# Reset the symbolic stdin's properties and set its length
st.posix.files[0].seek(0)
st.posix.files[0].length = 29
```

这里通过`st.se.add(k != 10)`限制了输入不为`'\n'`

## 优化脚本

实际用`angr`跑，会发现

- 跑了几个小时都还没有找到目标路径
- 跑着跑着就崩了
- 自动分析似乎很美好，但却隐藏着许多问题

- Environment
  - shared library
- Exploration Strategy
  - BFS
  - DFS
- Explosion
  - path explosion
  - path pruning

## Environment

情境

- 对符号执行来说，`libc`里复杂无比，一旦进入`libc` function分析可能就挂在里面了
- `Crypto function`
- 看不懂的`syscall`

### SimProcedure

```
p = angr.Project('./vul',
                 load_options={'auto_load_libs': True},
                 use_sim_procedures=True,
                 exclude_sim_procedures_func='strcmp')
```

### Hook symbol

```
class my_strcmp(simuvex.SimProcedure):
    def run(self):
        ...
        return ...
p.hook_symbol('strcmp', my_strcmp)
```

- Go into library

### Hook

```
'''
$ objdump -M intel -d ./vul | grep -A2 85d7
80485d7: e8 9f 00 00 00 call 804867b
80485dc: 89 44 24 10 mov DWORD PTR [esp+0x10],eax
80485e0: 83 7c 24 10 ff cmp DWORD PTR [esp+0x10],0xffffffff
'''
def check1(state):
    state.regs.eax = 20
p.hook(0x080485d7, check1, length=5)
```

### Unknown syscall

```
initial_state = project.factory.entry_state(
    args=[project.filename, arg1],
    add_options={'BYPASS_UNSUPPORTED_SYSCALL'})
```

## Exploration Strategy

### Exploration techniques

```
pg = p.factory.path_group(initial_state, immutable=False)
pg.use_technique(angr.exploration_techniques.DFS())
# pg.explore(find=(0x08041234, ))
pg.run(step_func=my_find_func)
```

## Explosion路径爆炸的解决方案

例如有这么一个情景

```
int counter = 0, values = 0;
for(i=0; i<100; i++){
    if(input[i] == 'B'){
        counter++;
        values += 2;
    }
}
if(counter == 75)
    bug();
```

会发现在二叉决策的时候，会发生路径爆炸的现象。

![](upload/attach/202001/821225_4HU84BHHK8TG6YE.jpg)

这个时候可以采用**`Veritesting`**的技术

![](upload/attach/202001/821225_3RRB88QMVY8JFBQ.jpg)

```
pg = p.factory.path_group(initial_state, immutable=False, veritesting=True)
```

### 情景2

`Unsatisfiable path`代表这条路不可能发生，即无法发生任何一組`input`使得`binary`可以照这条路执行

![](upload/attach/202001/821225_92PX6YJ57X67JB8.jpg)

### `LAZY_SOLVES`

自己给他取了个名字`剪枝`

- 懒得检查，意思是当路径探索完的时候才进行检查
- 预设是开启的

```
initial_state = project.factory.entry_state(args=[project.filename, arg1])
initial_state.options.discard('LAZY_SOLVES')
```

![](upload/attach/202001/821225_Y3H4Y4KGARZS3Q3.jpg)

![](upload/attach/202001/821225_5ADW8BEAFJ6C7BS.jpg)

### Dynamic path pruning

- 根据已经检查的路径，推测现在`unsatisfiable path`的比例
- 依照`unsatisfiable path`的比例调整之后路径要不要进行检查的概率

## Other Debug Options

- `REVERSE_MEMORY_NAME_MAP`
  - 保留内存地址的信息，让我们可以拿`BVS`的名字（`'file_/dev/stdin'`）来得到模拟的内存地址（`0xffff1234`）
- `TRACK_ACTION_HISTORY`
  - 方便查看之前所模拟执行过的状态的`ACTION`记录

## 小结

`AST`是语法分析树的简写。`se`是符号执行的简写。

为什么要用`Claripy`？

- Python中的integer几乎没有大小上限，无法很好的模拟C语言中例如`int`、`byte`的位数的限制，例如`int`中的`0XFFFFFFFF+1`会造成溢出，而Python中内置的数据类型则不会溢出。所以这里，我们采用`claripy.BVS`创建符号变量或者`claripy.BVV`来创建一个含有位数限制的数值。
