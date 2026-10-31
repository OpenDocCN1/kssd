
# glibc malloc 内存管理分析

## 宏定义与 bin 索引

```
#define in\_smallbin\_range(sz) \
((unsigned long) (sz) < (unsigned long) MIN\_LARGE\_SIZE)
#define bin\_index(sz) \
((in\_smallbin\_range (sz)) ? smallbin\_index (sz) : largebin\_index (sz))
```

上面展示的宏中有一个名为`SMALLBIN\_CORRECTION`的宏，该宏出现在`MIN\_LARGE\_SIZE`宏以及`smallbin\_index`宏中并影响着它们，至于`SMALLBIN\_CORRECTION`宏在不在其他宏中起实际作用，取决于`MALLOC\_ALIGNMENT`是否大于`CHUNK\_HDR\_SZ`。

GLibC会接收程序申请的内存大小，然后根据改申请的大小判断它在`bins`数组中的下标，其中`small bin`通过`smallbin\_index`获取下标，`large bin`通过`largebin\_index\_64`获取下标。

```
#define smallbin\_index(sz) (((unsigned) (sz)) >> 4) + SMALLBIN\_CORRECTION
#define largebin\_index\_64(sz) \
(((((unsigned long) (sz)) >> 6) <= 48) ? 48 + (((unsigned long) (sz)) >> 6) :\
......)
#define largebin\_index(sz) largebin\_index\_64
```

## bins 数组结构与 bin_at

结构体`struct malloc\_state`中的`bins[NBINS \* 2 - 2]`数组元素的数量是比较奇怪的，按照道理来讲，一个`NBINS`就足够了，为什么要`NBINS \* 2 - 2`呢？

首先内存有最小值的限制`MALLOC\_ALIGNMENT`，所以`sz >> 4`永远都不会是0，所以`bins`数组元素的数量就变成了`NBINS - 1`，又因为`bins`数组中的链表都是双向双向链表，存储着`fd`和`bk`两个成员，所以`bins`数组就扩成了`2\*(NBINS - 1)`。

由于奇怪的`bins`数组，所以GLibC提供`bin\_at`接口索引数组，但不幸的是，`bin\_at`接口的索引方式更加奇怪。

首先`bins[(i-1)\*2]`中的`(i-1)\*2`指的是`bins`数组中第`i`对`fd`和`bk`成员。

```
#define bin\_at(m, i) \
(mbinptr) (((char \*) &((m)->bins[((i) - 1) \* 2])) - offsetof (struct malloc\_chunk, fd))
```

找到第`i`对元素后会取出元素在数组中的内存地址，最后再减去`fd`的偏移值，这是一个很奇怪但又不奇怪的操作。

在`bins`数组存放数据的角度中，数组中的每个元素放置的都是由`stuct malloc\_chunk`描述的空闲chunk信息。

但在`bins`数组的设计角度来看，每`bins[i \* 2]`和`bins[i \* 2 - 1]`构成了第`i`对空闲chunk链表，`bin\_at`用于检索空闲chunk链表，而不是某个空闲chunk。

\*--------bins----------\* \*--------------new struct---------------\*
| ... | chunk\_$i | ... | -> | fd\_$(i-1) | bk\_$(i-1) | fd\_$i | bk\_$i |
\*--------bins----------\* \*--------------new struct---------------\*
addr = bin\_at(M, i)
addr->fd = fd\_i
addr->bk = bk\_i

空闲chunk进入链表时，会先从`bins`中取出第`i - 1`对空闲chunk链表，`bck->fd`可以获取第`i`对空闲chunk链表中（`\*(bck + offsetof(fd))`），然后会更新`victim`的`bk`和`fd`，其中`bk`为`bck`，`fd`为第`i`对链表，最后将第`i`对链表的`fd`（`bck->fd`）更新成`victim`，以及`fwd->bk`更新成`victim`。

空闲chunk离开链表时，会先检查链表中是不是只有一个元素，如果不是就会开始移除空闲chunk，它会先更新第`idx`对链表头的`bk`，`bk`对应`victim->bk`，再更新`bck`的`fd`为第`idx - 1`对链表，最后返回被移除链表的`victim`指针。

```
#define last(b) ((b)->bk)
#define chunk2mem(p) ((void\*)((char\*)(p) + CHUNK\_HDR\_SZ))
enter ->
bck = bin\_at (av, victim\_index); // bck = &(bins[(i-2)\*2])
fwd = bck->fd; // fwd = bins[(i-1)\*2] = fd\_i
victim->bk = bck; // set new chunk bk to default link
victim->fd = fwd; // let new chunk link to old chunk
fwd->bk = victim; // let old chunk link to new chunk
bck->fd = victim; // update fwd to new chunk
leave ->
bin = bin\_at (av, idx); // bin = &(bins[(i-2)\*2])
if ((victim = last (bin)) != bin) { // victim = bins[(i-1)\*2 + 1] = bk\_i
bck = victim->bk; // get current bk\_i linked chunk\_a
bin->bk = bck; // update current bk\_i to chunk\_a
bck->fd = bin; // update current fd\_i to default
void \*p = chunk2mem (victim);
}
```

`bins`数组中的`fd`和`bk`它们的分工并不相同，`fd`指向的永远是最晚入链的空闲chunk，而`bk`则会指向最早入链的空闲chunk，出链时都是让`bk`指向的空闲chunk出去，显然这是一种FIFO的管理方式，它与LIFO不同，越先进入队列的数据也会越先出列。

(i-1) / next <-----------|
| fd\_i | bk\_i | |
| | |
| ---------------|------------|
|--> victim -> bk --| |
^ -> fd -------------| |
| | |
|--< bk <- old chunk <--| |
(i-1) / prev <-- fd <- ^ |
|---------|

当`arena`完成初始化后，你可能会发现`bins`数组中不管是哪对链表，其`fd`和`bk`指向的都是第`i-1`对链表，这是因为`arena`初始化时，`bins`数组会被挨个初始化。

`bins`数组可以看作由两大部分组成，一是按照大小区分的链表，二是链表中`fd`和`bk`组成的双向链表，而通过`bin\_at`接口可以直接对特定链表中的`fd`和`bk`进行管理。

```
for (i = 1; i < NBINS; ++i)
{
bin = bin\_at (av, i);
bin->fd = bin->bk = bin;
}
```

### unsorted bin

上面提到过，因为chunk需要跟`MALLOC\_ALIGNMENT`对齐，所以`bins[0]`和`bins[1]`会空出来，这个空出来的`bin\_at[1]`并不会闲置，而是会给`unsorted bin`使用。

不过`unsorted bin`又是个什么东西呢？

`unsorted bin`可以看作是一个缓冲区，空闲chunk进入`small bin`和`large bin`之前会先进入`unsorted bin`链表中，分配内存时（`small\_bin`范围内），如果其余类型的链表中找不到空闲chunk，那就会进入`bin\_at[1]`中查找，`bin\_at[1]`中如果存在合适的chunk就会把它提供给申请者。

```
if (in\_smallbin\_range (nb))
......
if (in\_smallbin\_range (nb) &&
bck == unsorted\_chunks (av) &&
victim == av->last\_remainder &&
(unsigned long) (size) > (unsigned long) (nb + MINSIZE))
```

这么做主要是为了增强局部性，申请者有更大的机会使用到仍位于CPU缓存上的内存地址。

### 空闲 chunk 的重新赋值

chunk被释放变成空闲chunk时，其内存中存放的还是原来的数据吗？

答案当然不是，准确来讲是未必是。

之所以将未必是，是因为空闲chunk被赋新值需要在特定的场景下才会被激活。

第一种情况是看chunk会不会进入`tcache bins`中（），`tcache`是GLibC推出的一种提高堆管理性能的机制，在GLibC的`tcache`机制被启用的情况下，被释放的chunk会优先进入`tcache bins`中。

如果chunk进入`tcache bins`，那么会通过`tcache\_put`接口完成进入操作，该函数会将`e->next`对应的chunk地址放入`tcache->entries`中，离开时，GLibC会通过`tcache\_get`接口取出chunk，`tcache\_put`存放`e->next`之前，`e->next`中保存的数据会变成`tcache->entries[tc\_idx]`中的原数值（头插法），如果GLibC的版本比较新，那么新数值还会被`PROTECT\_PTR`随机化。

第二种情况是看chunk属不属于`fastbin`，如果属于`fastbin`，那么向链表插入新chunk时，数据也会被更新成上一个链表头的地址。

\_int\_free
-> tcache\_put
-> e->next = PROTECT\_PTR (&e->next, tcache->entries[tc\_idx])
-> tcache->entries[tc\_idx] = e
-> if ((unsigned long)(size) <= (unsigned long)(get\_max\_fast ()))
-> p->fd = PROTECT\_PTR (&p->fd, old);
(gdb) x /s buf
0x555d5892ead0: "test"
tcache\_put (chunk=0x555d5892eac0, tc\_idx=9)
(gdb) p /x \*(tcache\_entry\*)(0x555d5892eac0+0x10)
$1 = {next = 0x55580d47630e, key = 0x1962346efba31014}

## 子线程堆的复杂情况

对于主线程的`arena`来讲，不断的申请堆内存会导致`arena`中的`top`不断向上扩展。首先当申请的内存大小`nd`加上最小值仍在`top chunk`的范围内时，GLibC会从`top chunk`中取出内存交给主程序，如果发现申请内存大小`nd`加`MINSIZE`超出`top chunk`的范围，就会通过`sysmalloc`扩展`top chunk`。

```
victim = av->top;
size = chunksize (victim);
if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
remainder = chunk\_at\_offset (victim, nb);
av->top = remainder;
```

`sysmalloc`会通过`brk`扩充内存，并更新`arena`中的`system\_mem`，完成内存空间的扩充后，`sysmalloc`会更新`top`，并将返回`top`中的可用内存地址给主程序使用。

\_int\_malloc
......
else
{
void \*p = sysmalloc (nb, av);
if (p != NULL)
alloc\_perturb (p, bytes);
return p;
}
sysmalloc
-> av->system\_mem += size;
-> p = av->top;
-> av->top = remainder;

上面是主线程的`arena`针对堆内存申请时的扩展操作，子线程是不是也一样呢？

尽管主线程的`arena`和子线程的`arena`都通过`struct malloc\_state`描述，但是它们之间的差别还是挺大的。

首先主线程的`arena`和子线程的`arena`有一个非常明显的差别，它就是分配出来的内存地址形式，`main arena`的地址是跟主程序地址相似的，而`non main arena`的地址则是跟动态库的地址相似。

00401000-00402000 r-xp 00001000 08:08 7471258 ${program\_path}
00405000-0042e000 rw-p 00000000 00:00 0 [heap]
7ffff0000000-7ffff00b9000 rw-p 00000000 00:00 0

其次就是`non main arena`对待申请堆内存的态度并不与`main arena`一致。

在分配内存的数量不够多不够大时，仍会在`top chunk`中获取可用的内存区域，但是当申请的堆内存足够多时，`non main arena`就会分成多个子堆，子堆之间通过`heap\_info`结构体进行管理。

`ar\_ptr`记录着子堆所属的`arena`地址，`prev`记录着上一个子堆的`heap\_info`地址（单向链表，主子堆的`heap\_info`地址是起始位置，结束位置的`prev`数值为0），`size`记录当前内存的大小，`mprotect\_size`记录这被`PROT\_READ|PROT\_WRITE`标志保护的内存大小，`pagesize`代表当前内存页大小，至于`pad`，它是为了结构体大小和0x10或0x8对齐才存在的，没有什么特别含义。

non main\_arena address -> 0x7fffe0000030
non main\_arena top address ->
(gdb) p /x ((struct malloc\_state \*)(0x7fffe0000030))->top
$6 = 0x7ffe7a986020
heap\_info address ->
(gdb) p /x (0x7ffe7a986020 & ~(0x4000000 - 1))
$3 = 0x7ffe78000000
heap\_info ->
(gdb) p /x \*(heap\_info\*)0x7ffe78000000
$4 = {ar\_ptr = 0x7fffe0000030, prev = 0x7ffe80000000, size = 0x2987000, mprotect\_size = 0x2987000, pagesize = 0x1000, pad = {0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}}

知道有`heap\_info`结构体这么个东西后，又要面临一个问题，`heap\_info`结构体的地址是怎么产生的呢？`arena`根据什么信息可以获取子堆呢？

首先`non main arena`的`heap\_info`地址是根据`arena`的`top`地址产生的，GLibC会在`top`地址的下方选择`heap\_info`地址，并要求`heap\_info`地址与`HEAP\_MAX\_SIZE`对其，由此可以得到`heap\_info`地址的计算公式为`top & ~(HEAP\_MAX\_SIZE - 1)`。

GLibC提供`heap\_for\_ptr`接口，快速找到`arena`对应的`heap\_info`地址。

至于其他的子堆，则需要通过主子堆`heap\_info`中的`prev`成员进行遍历。

heap\_for\_ptr
-> max\_size = heap\_max\_size(); = HEAP\_MAX\_SIZE = 0x4000000
-> return PTR\_ALIGN\_DOWN (ptr, max\_size);

`heap\_info`的上方（`heap\_info\_addr + sizeof(heap\_info)`）就是数据区。

| top chunk |
| data | | data |
| sub heap - heap\_info | -> | sub heap | -> ......
| arena - struct malloc\_state |

\_int\_malloc分配内存时，发现`top chunk`空间不够用时，会进入`sysmalloc`函数的内部，该函数会根据`arena`地址进行判断，主线程`arena`是一个待遇，即通过`brk`机制扩展可用内存区域，子线程`arena`则是另一种处理方法。

sysmalloc
if (av != &main\_arena)
......
else
......

`sysmalloc`对子线程的子堆主要有两种处理方法。
