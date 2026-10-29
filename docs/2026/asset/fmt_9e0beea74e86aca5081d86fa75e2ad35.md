
```
1. 先确定最终输出位置

2. 从最终 packet 往前倒推

3. 每一段只关心输入、输出和实际内存读写变化

4. 写最小 Python lift

5. 和同一次 native run 做 byte-for-byte 对比

6. 再把小块组合成完整 pipeline
```

在 VM 保护场景里，不要迷信静态反编译。

更稳的方式是：

```
用静态分析找大致范围

用动态 trace 确认真实数据流

用 Python lift 固化结论

用 native 对照防止自嗨
```

最终产物不是一份“看起来像”的伪代码，而是可以跑出同样结果的实现。

## 当前边界

这次完整还原的是 `X-Medusa` 主路径，不等于整个 so 或所有 header 都已经完全还原。

当前边界：

```
1. X-Medusa 主路径可以纯 Python 复现

2. X-Argus / X-Ladon / X-Gorgon 等其它头不在本文范围内

3. VM 解释器本身没有完整转成 C

4. 三个 rand、pid、时间、f24 seed 仍属于运行时输入

5. 要跨运行输出完全一致，必须固定或捕获这些动态值
```

如果后续继续做，可以有两个方向：

```
1. 把 VM interpreter 的所有 handler 系统性转成 C/Python

2. 继续还原其它 header 的完整算法链
```

对 `X-Medusa` 来说，目前更直接有效的方式是：不追求完整解释器，而是 lift 实际执行路径和关键 native helper。

## 结语

这次 `X-Medusa` 的逆向过程，从最初只知道 VM 入口 `0x445b8`，到最终得到纯 Python 复现，大致经历了：

```
定位 header 插入点

确定 VM 执行边界

分析 packet 外层结构

倒推 second_buffer

还原 reverse-xor

还原 d71bc byte-state-machine

确认 SM3 key material

解析 src_a protobuf

还原 f23/f24 动态字段

还原 bit-slice patch

还原 AES-like transform

组合端到端 pipeline

native 对照验证
```

最终结果：

```
src_a_match=True

x_medusa_match=True
```

这也是本文最核心的判断标准：不是“猜到了算法像什么”，而是“同一次 native 输入下，Python 输出完全一致”。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2026-5-5 23:11 被scxc编辑 ，原因： 修改名词让可读性更高

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)

[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

上传的附件：

- [`libmetasec_ml.fully_deobf.so`](javascript:void(0);)

  （2.32MB，219次下载）

- [`dy_sign_send.py`](javascript:void(0);)

  （24.44kb，253次下载）

- [`medusa_lift_blocks.py`](javascript:void(0);)

  （63.96kb，236次下载）

- [`medusa_pure_x_medusa.py`](javascript:void(0);)

  （4.64kb，248次下载）

- [`medusa_reverse_walkthrough.md`](javascript:void(0);)

  （20.00kb，227次下载）

- [`medusa_re_notes.md`](javascript:void(0);)

  （35.12kb，237次下载）

- [`中间生成文件.zip`](javascript:void(0);)

  （133.92kb，225次下载）
