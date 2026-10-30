
```c
#if USE\_TCACHE
/\* While we're here, if we see other chunks of the same size,
stash them in the tcache. \*/
size\_t tc\_idx = csize2tidx (nb);
if (tcache && tc\_idx < mp\_.tcache\_bins)
{
mchunkptr tc\_victim;
/\* While bin not empty and tcache not full, copy chunks over. \*/
while (tcache->counts[tc\_idx] < mp\_.tcache\_count
&& (tc\_victim = last (bin)) != bin)
{
if (tc\_victim != 0)
{
bck = tc\_victim->bk;
set\_inuse\_bit\_at\_offset (tc\_victim, nb);
if (av != &main\_arena)
set\_non\_main\_arena (tc\_victim);
bin->bk = bck;
bck->fd = bin;
tcache\_put (tc\_victim, tc\_idx);
}
}
}
```

![](upload/attach/202204/882497\_99KVE2ZM7TE523R.png)

这里注意下，取出 `small bin` `chunk` 放入 `tcache bin` 中的时候，判断方法是 `tc\_victim = last (bin)) != bin`，也就是每次都从 `small bin` 链表尾取一个 `chunk` 判断是不是等于表头，然后再使用头插法存入 `tcache bin`，直到放满 `tcache bin`。

这样的话，`chunks` 在放入 `tcache bin` 后的排序与他们在 `small bins` 中的排序是一样的。这点可以与 `fastbin\_reverse\_into\_tcache` 这个例子作比较。

在 `fastbin\_reverse\_into\_tcache` 中，如果 `tcache bin` 为空，那么从 `fastbin` 取出一个 `chunk` 后也会将 `fast bin` 中剩余 `chunk` 放入 `tcache bin`。但是这里是从 `fast bin` 的头结点开始取 `chunk`，使用头插法插入 `tcache bin`。这样放入 `tcache bin` 后的 `chunk` 排序就会与他们在 `fast bin` 相反，所以叫 `reverse`。

这时候 `bk` 链现状：

注意是 `bk`，`bk` 指向前一个 `chunk`。

```
取出victim前的small bin：
fake\_chunks[6] --> fake\_chunks[5] --> fake\_chunks[4] --> fake\_chunks[3] --> fake\_chunks[2] --> fake\_chunks[1] --> fake\_chunks[0] --> stack2 --> stack1 --> victim
取出victim后的small bin 与 tcache bin：
fake\_chunks[6] --> fake\_chunks[5]
fake\_chunks[4] --> fake\_chunks[3] --> fake\_chunks[2] --> fake\_chunks[1] --> fake\_chunks[0] --> stack2 --> stack1
```

再次申请的时候，就是从 `tcache bin` 中取 `chunk`。`tcache bin` 中取 `chunk` 原则是先从头结点取，先进后出原则，取出 `fake\_chunks[4]`。

## house\_of\_einherjar\_glibc2.34

### 原理

`house of einherjar` 和 `poison null byte` 类似。通过溢出覆盖物理相邻高地址的 `prev\_size` 和 `prev\_inuse` 位，那么 `free` 的时候就会触发合并与 `unlink`，而合并的 `size` 是由我们自己控制的。也就是说如果我们可以同时控制一个 `chunk` `prev\_size` 与 `PREV\_INUSE` 字段，那么我们就可以将新的 `chunk` 指向几乎任何位置。

### POC

`how2heap` 源码：
