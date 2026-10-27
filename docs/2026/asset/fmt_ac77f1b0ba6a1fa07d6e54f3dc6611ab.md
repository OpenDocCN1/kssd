
```python
def get_phi_vars(ircfg,loc_db,mdis):
res = []
blks = list(ircfg.blocks)
irblock = (ircfg.blocks[blks[-1]])
if irblock_has_phi(irblock):
for dst, sources in viewitems(irblock[0]):
phi_vars = sources.args
parent_blks = get_phi_sources_parent_block(
ircfg,
irblock.loc_key,
phi_vars
)
for var, loc in parent_blks.items():
irblock = ircfg.get_block(list(loc)[0])
for asg in irblock:
dst, src = asg.items()[0]
if dst == var and isinstance(src,ExprInt):
res += [int(src)]
elif dst==var and isinstance(src,ExprOp):
reachings = ReachingDefinitions(ircfg)
digraph = DiGraphDefUse(reachings)
# the state var always a leaf
for head in digraph.heads():
if head.var == src.args[0]:
head_offset=loc_db.get_location_offset(head.label)
if isinstance(mdis.dis_instr(head_offset).args[1],ExprInt):
res+=[int(mdis.dis_instr(head_offset).args[1])]
return res
```

## 0x03 参阅链接

- <a href="elink@6c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7Y4b7@1L8Y4c8J5y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3e0f1!0p5k6h3k6D9j5i4c8@1k6h3&6W2M7W2)9J5c8R3`.`.">MODeflattener - Miasm's OLLVM Deflattener</a>

- <a href="elink@887K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7i4g2S2M7X3E0K6L8r3q4T1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2G2j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5k6s2u0W2j5$3!0$3k6i4u0A6L8X3N6Q4x3X3c8S2L8W2)9J5k6r3!0D9L8s2k6E0i4K6u0V1M7s2u0G2N6r3g2U0N6r3g2V1i4K6u0V1M7s2u0G2k6%4u0S2L8g2)9J5k6h3S2@1L8h3H3`.">Deobfuscation: recovering an OLLVM-protected program—Quasklab</a>

- <a href="elink@94eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5I4x3e0t1`.">利用符号执行去除控制流平坦化—腾讯SRC</a>

- <a href="elink@085K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5h3&6@1K9r3g2K6K9i4y4Q4x3X3g2@1L8#2)9J5c8U0t1H3x3U0q4Q4x3V1j5H3x3#2)9J5c8U0l9K6i4K6u0r3k6X3I4S2N6s2c8W2L8X3W2F1k6#2)9#2k6X3c8W2N6r3g2U0N6r3W2G2L8W2)9J5k6h3S2@1L8h3H3`.">Automated Detection of Control-flow Flattening</a>

- <a href="elink@1d3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6h3q4Q4x3X3c8K6k6h3y4Q4x3V1k6E0K9h3q4K6L8b7`.`.">miasm—Github</a>

- <a href="elink@1d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9h3q4K6L8g2)9J5k6i4u0W2i4K6u0r3L8h3W2S2M7$3#2Q4y4h3k6V1L8%4S2&6k6$3g2F1i4K6u0r3">miasm Documentation</a>

- <a href="elink@de7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3i4K6u0r3L8r3W2T1i4K6u0r3g2s2u0S2L8Y4y4X3L8%4u0E0M7#2)9J5c8V1!0T1k6Y4g2K6j5$3q4@1K9h3!0F1i4K6u0r3c8X3I4S2N6s2c8W2L8X3W2F1k6#2)9J5k6h3y4H3M7l9`.`.">OLLVM Flattening—Github</a>

- [基于LLVM Pass实现控制流平坦化—看雪](https://bbs.pediy.com/thread-266082.htm)

文章首发于安全客——[链接](https://bbs.kanxue.com/elink@f9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1@1z5o6p5#2y4l9`.`.)

## 0x00 前言

笔者于五月份时遇到几个经控制流平坦化的样本，由于之前没有接触过这方面知识，未深入分析。

![](https://bbs.kanxue.com/upload/attach/202107/817966_8FVQAXZUH93PD2T.jpg)

七月初看到一篇博客——[MODeflattener - Miasm's OLLVM Deflattener](https://bbs.kanxue.com/elink@4f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7Y4b7@1L8Y4c8J5y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3e0f1!0p5k6h3k6D9j5i4c8@1k6h3&6W2M7W2)9J5c8R3`.`.)，作者同时在Github上公开其源码，故笔者决定对其一探究竟。该工具使用`miasm`框架，Quarklab于2014年的博客——[Deobfuscation: recovering an OLLVM-protected program](https://bbs.kanxue.com/elink@8a7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7i4g2S2M7X3E0K6L8r3q4T1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2G2j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5k6s2u0W2j5$3!0$3k6i4u0A6L8X3N6Q4x3X3c8S2L8W2)9J5k6r3!0D9L8s2k6E0i4K6u0V1M7s2u0G2N6r3g2U0N6r3g2V1i4K6u0V1M7s2u0G2k6%4u0S2L8g2)9J5k6h3S2@1L8h3H3`.)中已提到此框架，但其采用符号执行以遍历代码，计算每个Relevant Block目标终点，恢复执行流程。腾讯SRC于2017年的博客——[利用符号执行去除控制流平坦化](https://bbs.kanxue.com/elink@c88K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5I4x3e0t1`.)中亦是采用符号执行(基于`angr`框架)，而`MODeflattener`采用静态分析方式来恢复执行流程。笔者于下文首先结合具体样本分析其源码及各函数功能，之后指出该工具不足之处并提出改进方案。关于涉及到的`miasm`框架中类，函数，属性及方法可参阅[`miasm Documentation`](https://bbs.kanxue.com/elink@005K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9h3q4K6L8g2)9J5k6i4u0W2i4K6u0r3L8h3W2S2M7$3#2Q4y4h3k6V1L8%4S2&6k6$3g2F1i4K6u0r3)与[`miasm`源码](https://bbs.kanxue.com/elink@99aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6h3q4Q4x3X3c8K6k6h3y4Q4x3V1k6E0K9h3q4K6L8b7`.`.)。LLVM，OLLVM，LLVM Pass及控制流平坦化不再赘述，具体可参阅[基于LLVM Pass实现控制流平坦化](https://bbs.pediy.com/thread-266082.htm)与[OLLVM Flattening源码](https://bbs.kanxue.com/elink@c87K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3i4K6u0r3L8r3W2T1i4K6u0r3g2s2u0S2L8Y4y4X3L8%4u0E0M7#2)9J5c8V1!0T1k6Y4g2K6j5$3q4@1K9h3!0F1i4K6u0r3c8X3I4S2N6s2c8W2L8X3W2F1k6#2)9J5k6h3y4H3M7l9`.`.)。

## 0x01 源码分析

### 0x01.1 检测平坦化

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `def` `calc_flattening_score(asm_graph):`  `score` `=` `0.0`  `for` `head` `in` `asm_graph.heads_iter():`  `dominator_tree` `=` `asm_graph.compute_dominator_tree(head)`  `for` `block` `in` `asm_graph.blocks:`  `block_key` `=` `asm_graph.loc_db.get_offset_location(block.lines[``0``].offset)`  `dominated` `=` `set``(`  `[block_key]` `+` `[b` `for` `b` `in` `dominator_tree.walk_depth_first_forward(block_key)])`  `if` `not` `any``([b` `in` `dominated` `for` `b` `in` `asm_graph.predecessors(block_key)]):`  `continue`  `score` `=` `max``(score,` `len``(dominated)``/``len``(asm_graph.nodes()))`  `return` `score` |

计算每个块支配块数量，除以图中所有块，取最高分——若不低于`0.9`，证明该函数经过控制流平坦化。如下图：

![](https://bbs.kanxue.com/upload/attach/202107/817966_ANVVJ34N2MWWP3H.jpg)

块`b`支配除块`a`外其余所有块(包含自身)，那么其得分为`9/10=0.9`即最高分。读者可自行查阅Dominator Tree及Predecessors相关数据结构知识，此外不作展开。

### 0x01.2 获取平坦化相关信息

![](https://bbs.kanxue.com/upload/attach/202107/817966_M9VGFWFKSC8HN5Q.jpg)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `def` `get_cff_info(asmcfg):`  `preds` `=` `{}`  `for` `blk` `in` `asmcfg.blocks:`  `offset` `=` `asmcfg.loc_db.get_location_offset(blk.loc_key)`  `preds[offset]` `=` `asmcfg.predecessors(blk.loc_key)`  `# pre-dispatcher is the one with max predecessors`  `pre_dispatcher` `=` `sorted``(preds, key``=``lambda` `key:` `len``(preds[key]), reverse``=``True``)[``0``]`  `# dispatcher is the one which suceeds pre-dispatcher`  `dispatcher` `=` `asmcfg.successors(asmcfg.loc_db.get_offset_location(pre_dispatcher))[``0``]`  `dispatcher` `=` `asmcfg.loc_db.get_location_offset(dispatcher)`    `# relevant blocks are those which preceed the pre-dispatcher`  `relevant_blocks` `=` `[]`  `for` `loc` `in` `preds[pre_dispatcher]:`  `offset` `=` `asmcfg.loc_db.get_location_offset(loc)`  `relevant_blocks.append(get_block_father(asmcfg, offset))`    `return` `relevant_blocks, dispatcher, pre_dispatcher` |

首先拥有最大数量Predecessor者即为`Pre-Dispatcher`，而`Pre-Dispatcher`后继为`Dispatcher`，`Pre-Dispatcher`所有前趋均为`Relevant Block`。

|  |  |
| --- | --- |
| 1  2  3 | `dispatcher_blk` `=` `main_asmcfg.getby_offset(dispatcher)`  `dispatcher_first_instr` `=` `dispatcher_blk.lines[``0``]`  `state_var` `=` `dispatcher_first_instr.get_args_expr()[``1``]` |

`Dispatcher`块第一条指令为状态变量：

![](https://bbs.kanxue.com/upload/attach/202107/817966_F8PXFJ2TN39876E.jpg)

### 0x01.3 去平坦化

|  |  |
| --- | --- |
