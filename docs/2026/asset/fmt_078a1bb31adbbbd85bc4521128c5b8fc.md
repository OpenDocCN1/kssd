
```c
void free(void \*p)
{
if (!p) return;
struct meta \*g = get\_meta(p);// 通过chunk p 用get\_meta得到对应的meta
int idx = get\_slot\_index(p);// 得到对应chunk的 idx
size\_t stride = get\_stride(g); // 得到sizeclasses 中对应chunk类型的size
unsigned char \*start = g->mem->storage + stride\*idx;
unsigned char \*end = start + stride - IB;
//\*start = g->mem->storage(得到group中第一个chunk地址) + stride\*idx(加上对应chunk偏移);
// start 就为对应p(chunk)的起始地址
// end 对应结束地址
get\_nominal\_size(p, end);//算出真实大小
uint32\_t self = 1u<<idx, all = (2u<<g->last\_idx)-1;//设置bitmap 标志
((unsigned char \*)p)[-3] = 255;
\*(uint16\_t \*)((char \*)p-2) = 0;
if (((uintptr\_t)(start-1) ^ (uintptr\_t)end) >= 2\*PGSZ && g->last\_idx) {
unsigned char \*base = start + (-(uintptr\_t)start & (PGSZ-1));
size\_t len = (end-base) & -PGSZ;
if (len) madvise(base, len, MADV\_FREE);
}
// atomic free without locking if this is neither first or last slot
for (;;) {
uint32\_t freed = g->freed\_mask;
uint32\_t avail = g->avail\_mask;
uint32\_t mask = freed | avail; // 将释放的chunk 和 现在可用的 chunk 加起来
assert(!(mask&self));
if (!freed || mask+self==all) break;
//!freed 没有被释放的chunk，mask+self==all说明释放了当前chunk所有chunk 都将被回收
// 此group 会被弹出队列
if (!MT)
g->freed\_mask = freed+self;// 设置free\_mask 表示chunk 被释放
else if (a\_cas(&g->freed\_mask, freed, freed+self)!=freed)
continue;
return;
}
wrlock();
struct mapinfo mi = nontrivial\_free(g, idx);// 含有meta 操作 ，内有unlink 是漏洞利用的关键
unlock();
if (mi.len) munmap(mi.base, mi.len);
}
```

## free 的流程

`free`：

通过`get_meta(p)`得到`meta` (`get_meta`是通过`chunk`对应的`offset`索引到对应的`group`再索引到`meta`) 下面会详细介绍`get_meta`

通过`get_slot_index(p)`得到对应`chunk`的`idx` -> 通过`get_nominal_size(p, end)`算出真实大小

重置`idx`和`offset` `idx`被置为`0xff`标记`chunk`

修改`freed_mask`标记`chunk`被释放

最后调用`nontrivial_free`完成关于`meta`一些剩余操作 (注意进入`nontrivial_free`是在`for`循环外还未设置)

### 细节

释放`chunk`的时候，先只会修改`freed_mask`,不会修改`avail_mask`，说明`chunk`在释放后，不会立即被复用

注意进入`nontrivial_free`是在`for`循环外还未设置`freed_mask` 跳出循环的条件是`if (!freed || mask+self==all) break;`

`free`中`chunk`的起始位置可以通过`chunk`的`idx`定位

### get_meta

```c
static inline struct meta \*get\_meta(const unsigned char \*p)
{
assert(!((uintptr\_t)p & 15));
int offset = \*(const uint16\_t \*)(p - 2);// 得到chunk offset
int index = p[-3] & 31;;// 得到chunk idx
if (p[-4]) {
assert(!offset);
offset = \*(uint32\_t \*)(p - 8);
assert(offset > 0xffff);
}
const struct group \*base = (const void \*)(p - UNIT\*offset - UNIT);// 通过offset 和chunk 地址计算出group地址
const struct meta \*meta = base->meta;// 从group 得到 meta 地址
assert(meta->mem == base);// 检查meta 是否指向对应的group
assert(index <= meta->last\_idx);// 检查chunk idx 是否超过 meta 最大chunk 容量
assert(!(meta->avail\_mask & (1u<<index)));
assert(!(meta->freed\_mask & (1u<<index)));
const struct meta\_area \*area = (void \*)((uintptr\_t)meta & -4096);// 得到meta\_area 地址
assert(area->check == ctx.secret);// 检查 check 校验值
if (meta->sizeclass < 48) { // 如果属于 sizeclasses 管理的chunk 大小
assert(offset >= size\_classes[meta->sizeclass]\*index);
assert(offset < size\_classes[meta->sizeclass]\*(index+1));
} else {
assert(meta->sizeclass == 63);
}
if (meta->maplen) {
assert(offset <= meta->maplen\*4096UL/UNIT - 1);
}
return (struct meta \*)meta;
}
```

## nontrivial_free

关于`nontrivial_free()`函数很重要，这里尽量详细说明

```c
static struct mapinfo nontrivial\_free(struct meta \*g, int i)// i = idx
{
uint32\_t self = 1u<<i;
int sc = g->sizeclass;
uint32\_t mask = g->freed\_mask | g->avail\_mask;//mask=已经被free的chunk +可使用的chunk
if (mask+self == (2u<<g->last\_idx)-1 && okay\_to\_free(g))
{ /\*
如果 mask+self == (2u<<g->last\_idx)-1 代表此meta中group里的chunk 都被释放 或者 都被用了
(2u<<g->last\_idx)-1 计算出的值化成二进制，其中每位含义类似于bitmap，如果每位为1表每位要不是被free 不然就是被
okay\_to\_free 检测是否可以被释放
\*/
if (g->next)
{ // 如果队列中 有下一个meta
assert(sc < 48);// 检测 sc 是不是mmap 分配的
// 检测当前meta g 和 队列里的active[sc] meta 是否一样，一样则activate\_new赋值为1
int activate\_new = (ctx.active[sc]==g);
dequeue(&ctx.active[sc], g);// 当前meta 出队
// 在出队操作后 ,ctx.active[sc]==meta ->next 是指的刚刚出队meta 的下一个meta
if (activate\_new && ctx.active[sc])
activate\_group(ctx.active[sc]);//如果有下一个meta 直接激活 然后修改avail\_mask 标志位
}
return free\_group(g);
}
else if (!mask)
{// mask==0 group chunk 空间已被完全使用
assert(sc < 48);
// might still be active if there were no allocations
// after last available slot was taken.
if (ctx.active[sc] != g) {// 如果 g 未被加入 队列ctx.ative[sc]
queue(&ctx.active[sc], g);// 把g 加入队列
}
}
a\_or(&g->freed\_mask, self);// 修改对应 的freed\_mask 标志 ，表示着对应的chunk 已被释放
return (struct mapinfo){ 0 };
}
```

```c
static inline void dequeue(struct meta \*\*phead, struct meta \*m)
{
if (m->next != m) {
m->prev->next = m->next; // 这里存在指针互写 在 prev 所指地址上 写入next 指针
m->next->prev = m->prev; // 在next 所指地址上 写入prev 指针
if (\*phead == m) \*phead = m->next;// 队列头如果为m 那就更新为m->next
} else {
\*phead = 0;
}
m->prev = m->next = 0; // 清理m(meta)的头尾指针
}
```

## dequeue 触发条件

![image-20210920171255661](upload/attach/202208/907645_RU38UNQWAV9XQN4.jpg)

> `self` = 1 << `idx`

下面是几种简单的触发情况

1.`avail_mask`表示只有一个`chunk`被使用,`freed_mask`=0，而`free`刚好要`free`一个`chunk`

满足`okay_to_free()`条件就可以进入`dequeue`进行出队操作

> 如`add(1,0x20)`再`free(1)`就会使得`meta`被回收

2.`avail_mask`=0,`freed_mask`表示只有1个`chunk`没被释放，这时释放的`chunk`就应该是那最后一个`chunk`

如下面情况`avail_mask`==0 `free_mask`=63=00111111 `last_idx`=6

已经释放6个`chunk`还有最后一个`chunk`没被释放在释放最后一个`chunk`时会触发`dequeue`使得对应`meta`出队

![image-20210920171918336](upload/attach/202208/907645_8WSV93SP66QAQR9.jpg)

3.如果发现这个`group`中所有的`chunk`要么被`free`,要么是可用的,那么就会回收掉这个`group`，调用`dequeue`从队列中出队

## 0x02 CTF 中musl题利用

一般有如下几种利用方法，核心原理都是构造假的`chunk`索引到假的`group`从而所引导假的`meta`

或覆盖`group`中指向`meta`的指针覆盖为假的`meta`，然后使得假的`meta``dequeue`最终实现`unlink`

(构造`fake_meta`需要先泄露`secret`校验值)

### dequeue 的两种流程

**一、**

通过构造假的`meta`满足各种条件通过以下流程

`free()`->`nontrivial_free()`->`dequeue`

这里通过`free`到`dequeue`

**二、**

通过`realloc`里也带有`free`

`realloc()`->`free(old)`->`nontrivial_free()`->`dequeue`

### 伪造meta后控制程序流的方法

注意:`musl`是没有`malloc_hook`和`free_hook`这种一般的`hook`位

且`musl`程序的`IO_FILE`结构体格式和`libc`不一样没有`IO_jump_t`的`vtable`

但是存在`read`，`write`，`seek`，`close`四个函数指针

![image-20210920180511605](upload/attach/202208/907645_UX2NAK28CM6C8HR.jpg)

讲一下思路

1.伪造`meta`后满足各种条件使得其进入`dequeue`通过`unlink`,构造`prev`,`next`实现任意地址指针互写

通过任意地址互写指针，向`stdout_used`写入我们伪造的`fake_stdout`地址，通过`IO_FILE`劫持程序执行流

到我们布置好的`fake_stdout`上，可以找`IO_FILE`里的一些函数`exit``puts`在`fake_stdout`上布置`rop_chain`然后通过栈迁移的`gadget`利用`FSOP`劫持程序到布置的`fake_stdout`上

2.第二种方式更麻烦也是伪造`fake_meta`也是任意地址指针互写，先进行布局使得`fake_meta``dequeue`实现`unlink`，

在利用指针互写修改`fake_meta`中的`mem`(`mem`就是`group`区域)，把`mem`修改为我们想要的地址，

然后让`fake_meta`通过`queue`入队，可以实现任意地址分配的，然后同样是打`IO_FILE`通过修改`stdout``stdin`和`stderr`结构体劫持程序流

## 0x03 补充

前面提到过

- 第一种:如果一个`group`中所有的`chunk`都已经被使用，且没有`free`掉的`chunk`
- 第二种:`group`中的`chunk`当`free`掉最后一个`chunk`，都处于`freed`的状态

处于这两种状态的`group`对应的`meta`会被`dequeue`出队

1.第一种状态，在`malloc`时候触发`dequeue`，当前`meta`出队

```c
/\*
/src/malloc/mallocng/malloc.c
\*/
static uint32\_t try\_avail(struct meta \*\*pm)
{
struct meta \*m = \*pm;
uint32\_t first;
if (!m) return 0;
uint32\_t mask = m->avail\_mask;
if (!mask)
{ //这里如果mask==0，含义就是group中的chunk已被全部使用，没有空闲了
if (!m) return 0;
if (!m->freed\_mask)//并且没有已经释放的chunk
{
dequeue(pm, m);//直接让这个meta 出队，因为group已经没有可用空间了
m = \*pm;
if (!m) return 0;
}
else {
m = m->next;
\*pm = m;
}
....
}
.....
}
```

2.第二种状态在`free`时候，在`nontrivial_free()`中触发`dequeue`，当前`meta`出队，不被使用
