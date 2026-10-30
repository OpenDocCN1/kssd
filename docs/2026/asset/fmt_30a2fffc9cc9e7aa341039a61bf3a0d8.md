
## 代码片段

```
if ((victim = first (bin)) != bin && (unsigned long)(victim->size) >= (unsigned long)(nb)) // 判断链表的第一个结点，即最大的chunk是否大于要申请的size
{
victim = victim->bk_nextsize;
while (((unsigned long)(size = chunksize (victim)) < (unsigned long)(nb))) // 从最小的chunk开始，反向遍历 chunk size链表，直到找到第一个大于等于所需chunk大小的chunk退出循环
victim = victim->bk_nextsize;
// 漏洞点，伪造bk_nextsize
if (victim != last (bin) && victim->size == victim->fd->size) // 申请的chunk对应的chunk存在多个结点，则申请相应堆头的下个结点，不申请堆头。
victim = victim->fd;

remainder_size = size - nb;
unlink (av, victim, bck, fwd);
// largebin unlink 操作
...
return p;
}
```

## 利用原理

当利用`bk_nextsize`反向遍历双链表时，如果我们可以伪造堆头节点的`bk_nextsize`，让其指向一个`evil`内存地址，然后我们在`evil_addr`上构造了相应的数据以通过`unlink`，会将该空间返回申请到非预期的内存。

绕过`unlink`最简单的是`fd`与`bk`按照`smallbin`的`unlink`方式设置，然后两个`nextsize`指针都为`null`。

下面贴上大佬的解释：

*问题出现在通过`bk_nextsize`反向遍历双链表的过程，如果能够伪造某个堆头结点中的`bk_nextsize`，将其指向非预期的内存地址，构造好数据使得非预期内存地址在通过`unlink`的检查之后，会将该空间返回给用户，最终使得可以申请出非预期的内存。最常见的就是用它来构造`overlap chunk`。*

*至于绕过`unlink`的检查，我认为最好的方式就是伪造的内存空间将`fd`与`bk`按照`smallbin` `unlink`的利用方式设置，而将`bk_nextsize`和`fd_nextsize`设置成0，这样就不会对这两个字段进行操作了。*

*典型的应用场景为：存在四个堆`ABCD`，`largebin`中存在链表`A->B`，其中`A`为`0x420`，`B`为`0x400`，`C`为`0x410`，`C`未释放。将`B`的`bk_nextsize`伪造指向`C`，同时将`C`的`fd`与`bk`构造好，将`C`的`fd_nextsize`与`bk_nextsize`赋值为0，当再次申请`0x410`大小的内存`E`时，遍历`B->bk_nextsize`会指向`C`，且`C`的大小满足需求，因此会调用`unlink`将`C`从双链表取下，因此申请出来的堆块`E`的地址会为`C`的地址，即`E`和`C`为同一内存块，实现`overlap chunk`的构造。*

## 伪造 `largebin` 的 `bk_nextsize` 以及 `bk`（插入 `largebin` 时）

|  |  |

| --- | --- |
