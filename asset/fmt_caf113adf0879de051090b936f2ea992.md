
# `tElock` 0.90

> 原文：[`www.pediy.com/kssd/pediy04/kanxue483.htm`](https://www.pediy.com/kssd/pediy04/kanxue483.htm)

先看看旧帖子：

`http://www.chat001.com/forum/crackforum/40418.html`

`http://www.chat001.com/forum/crackforum/23607.html`

`http://www.chat001.com/forum/crackforum/24002.html`

....

1、找 OEP：

仍然是 `BPX VirtualProtectEx`，中断下来后再手动跳过几个 SEH 就和旧版本的一样，直接到达 OEP。参看精华区中关于旧版本的脱壳文章。

2、从内存中 dump IT：

先找准 IT 的起始位置，一般就是`.idata`段的起始处。找 IT 的其它方法参看精华区。假定找到的位置为 `XXXXXXXX`，然后“`bpx VirtualAlloc do "dd XXXXXXXX"`，每次中断后注意看数据区中的内容，看看 `telock` 什么时候把解压出来的完整的 IT 表放在那个位置。一旦发现 IT/IAT 已经完整解压出来，小心地在它破坏 IT 之前将其 dump 出来即可。那段破坏代码和旧版本的一样，很容易找到。

之后的修复就不用说了。

注意：

1、`telock` 用调试器寄存器来做 CRC 检查，所以不要设 `BPM` 断点

2、设 `BPX` 断点时最好不要把断点设在 API 函数的第一条指令上，比如“`BPX VirtualProtectEx`”可以设为“`BPX VirtualProtectEx+1`”等，因为 `telock` 会检查某些 API 函数入口处有无 `0xCC`（`int 3`）。

`4dd466d0e1e0cc123afbe504da2462ca`
