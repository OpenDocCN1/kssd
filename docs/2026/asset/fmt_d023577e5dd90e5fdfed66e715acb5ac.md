
## 基础知识

`2.29`后的`libc`在两个`free chunk`进行合并前多一次对`prevsize`的值检查对应的源代码如下：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 | `/* consolidate backward */` `if` `(!prev_inuse(p)) {` `prevsize = prev_size(p);` `size += prevsize;` `p = chunk_at_offset(p, -((long) prevsize));` `if` `(__glibc_unlikely(chunksize(p) != prevsize))` `malloc_printerr("corrupted size vs. prev_size while consolidating");` `unlink_chunk(av, p);` `}` |

## 调试

1. 第一步重要的是让新申请的`chunk`末两字节的位置为`\x00\x00`，方便以后的利用。

```
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x55555555d000
Size: 0x290 (with flag bits: 0x291)
Allocated chunk | PREV_INUSE
Addr: 0x55555555d290
Size: 0x20 (with flag bits: 0x21)
Allocated chunk | PREV_INUSE
Addr: 0x55555555d2b0
Size: 0x2d50 (with flag bits: 0x2d51)
Top chunk | PREV_INUSE
Addr: 0x555555560000
Size: 0x1e000 (with flag bits: 0x1e001)
```

1. 接下来申请两个`chunk`，`prev_0x510`和`victim_0x500`，以及一个`barrier_0x20`。这两个`chunk`是为了后续合并利用。然后继续申请`a_0x500`，`barrier_0x20`，`b_0x520`，`barrier_0x20`。

```
Allocated chunk | PREV_INUSE
Addr: 0x55555555d000
Size: 0x290 (with flag bits: 0x291)
Allocated chunk | PREV_INUSE
Addr: 0x55555555d290
Size: 0x20 (with flag bits: 0x21)
Allocated chunk | PREV_INUSE
Addr: 0x55555555d2b0
Size: 0x2d50 (with flag bits: 0x2d51)
// prev
Allocated chunk | PREV_INUSE
Addr: 0x555555560000
Size: 0x510 (with flag bits: 0x511)
// victim
Allocated chunk | PREV_INUSE
Addr: 0x555555560510
Size: 0x500 (with flag bits: 0x501)
Allocated chunk | PREV_INUSE
Addr: 0x555555560a10
Size: 0x20 (with flag bits: 0x21)
// a
Allocated chunk | PREV_INUSE
Addr: 0x555555560a30
Size: 0x500 (with flag bits: 0x501)
Allocated chunk | PREV_INUSE
Addr: 0x555555560f30
Size: 0x20 (with flag bits: 0x21)
// b
Allocated chunk | PREV_INUSE
Addr: 0x555555560f50
Size: 0x520 (with flag bits: 0x521)
Allocated chunk | PREV_INUSE
Addr: 0x555555561470
Size: 0x20 (with flag bits: 0x21)
Top chunk | PREV_INUSE
Addr: 0x555555561490
Size: 0x1cb70 (with flag bits: 0x1cb71)
```

1. 依次释放 `a`、`b`、`prev`，然后申请一个 `0x1010` 大小的 `chunk`，出发整理机制，将其放入 `largebin` 中。

```
pwndbg> bins
largebins	//b prev a
0x500-0x530: 0x555555560f50 —▸ 0x555555560000 —▸ 0x555555560a30 —▸ 0x7ffff7bb7010 (main_arena+1168) ◂— 0x555555560f50
// b
pwndbg> x/8gx 0x555555560f50
0x555555560f50:	0x0000000000000000	0x0000000000000521
0x555555560f60:	0x0000555555560000	0x00007ffff7bb7010
0x555555560f70:	0x0000555555560a30	0x0000555555560f50
0x555555560f80:	0x0000000000000000	0x0000000000000000
// prev
pwndbg> x/8gx 0x555555560000
0x555555560000:	0x0000000000000000	0x0000000000000511
0x555555560010:	0x0000555555560a30	0x0000555555560f50
0x555555560020:	0x0000555555560a30	0x0000555555560f50
0x555555560030:	0x0000000000000000	0x0000000000000000
// a
pwndbg> x/8gx 0x555555560a30
0x555555560a30:	0x0000000000000000	0x0000000000000501
0x555555560a40:	0x00007ffff7bb7010	0x0000555555560000
0x555555560a50:	0x0000555555560f50	0x0000555555560000
0x555555560a60:	0x0000000000000000	0x0000000000000000
```

1. 接下来将 `prev` 申请回来，将其 `bk` 位置写为 `0x501`，将`victim_prev_size`写为`0x500`，充当`fake_chunk`的`size`，其`fd_next`和`bk_next`分别指向`a`和`b`。

```
// prev&fake_chunk
pwndbg> x/8gx 0x555555560000
0x555555560000:	0x0000000000000000	0x0000000000000511
0x555555560010:	0x0000555555560a30	0x0000000000000501
0x555555560020:	0x0000555555560a30	0x0000555555560f50
0x555555560030:	0x0000000000000000	0x0000000000000000
// victim
pwndbg> x/8gx 0x555555560510
0x555555560510:	0x0000000000000500	0x0000000000000501
0x555555560520:	0x0000000000000000	0x0000000000000000
0x555555560530:	0x0000000000000000	0x0000000000000000
0x555555560540:	0x0000000000000000	0x0000000000000000
```

1. 接下来将`b`申请回来，修改其`fd`位置后两位为`\x00\x10`，让`b`的`fd`指向`fake_chunk`。再将`a`申请出来，然后依次释放`a`和`victim`，让`a`的`bk`指向`victim`。然后在修改其后两位为`\x00\x10`，让`a`的`bk`指向`fake_chunk`。

```
// b
pwndbg> x/8gx 0x555555560f50
0x555555560f50:	0x0000000000000000	0x0000000000000521
0x555555560f60:	0x0000555555560010	0x00007ffff7bb7010
0x555555560f70:	0x0000555555560a30	0x0000555555560a30
0x555555560f80:	0x0000000000000000	0x0000000000000000
// a
pwndbg> x/8gx 0x555555560a30
0x555555560a30:	0x0000000000000000	0x0000000000000501
0x555555560a40:	0x00007ffff7bb6be0	0x0000555555560010
0x555555560a50:	0x0000000000000000	0x0000000000000000
0x555555560a60:	0x0000000000000000	0x0000000000000000
```

1. 接下来要做的就简单了，将`victim`申请回来，然后利用`off-by-null`将其`size`的`prev_inuse`位清零，然后`free`掉`victim`。就完成了依次`overlap`。

```
pwndbg> x/8gx 0x555555560000
0x555555560000:	0x0000000000000000	0x0000000000000511
0x555555560010:	0x0000555555560a30	0x0000000000000a01
0x555555560020:	0x00007ffff7bb6be0	0x00007ffff7bb6be0
0x555555560030:	0x0000000000000000	0x0000000000000000
pwndbg> bins
unsortedbin
all: 0x555555560010 —▸ 0x7ffff7bb6be0 (main_arena+96) ◂— 0x555555560010
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2024-4-26 18:07

被jelasin编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)
