
```c
mark\_bin (av, victim\_index);
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

### smallbin的fwd bck赋值

- 如果属于`small bin`则进行`fwd`和`bck`的赋值

> `small bin` 的链表表头赋值给 `bck`:`bck = bin_at (av, victim_index)`;

> 首个`chunk`赋值给`fwd` :`fwd = bck->fd`;

```c
if (in\_smallbin\_range (size)){
victim\_index = smallbin\_index (size);
bck = bin\_at (av, victim\_index);
fwd = bck->fd;
}
```

### largebin 入bin链和chunk size链

- 如果属于`large_bins`同理进行赋值 然后判断该插入什么合适的位置

> 因为`largebin`是按照大小进行的排序 由大到小 所以最小的在链表最后

```c
victim\_index = largebin\_index (size);
bck = bin\_at (av, victim\_index);
fwd = bck->fd;
```

- 判断`large`是否有空闲`chunk`:

```c
if (fwd != bck)
```

- 如果当前`chunk`比最后一位`chunk`还小则直接加入链表末尾

> `bck`是头 `bck->bk`应该就是最后一位

> 然后要加入`fwd`和`bck`之间 我们应该先调整`fwd`和`bck` 所以`bck`改为链表最后一位 `fwd`改为链表头

> `bck`<----->`chunk`<----->`fwd`

```c
if ((unsigned long) (size)< (unsigned long) chunksize\_nomask (bck->bk)){
fwd = bck;
bck = bck->bk;
victim->fd\_nextsize = fwd->fd;
victim->bk\_nextsize = fwd->fd->bk\_nextsize;
fwd->fd->bk\_nextsize = victim->bk\_nextsize->fd\_nextsize = victim;
}
```

- 否则进行遍历判断 匹配第一个小于等于 当前`chunk`的

```c
while ((unsigned long) size < chunksize\_nomask (fwd)){
fwd = fwd->fd\_nextsize;
assert (chunk\_main\_arena (fwd));
}
```

- 如果该`chunk`与当前`chunk`相同则让`chunk`插入`fwd`之后 所以

> 因为`large bin`是按照大小进行的排序 所以我们为了不额外修改`chunk size`链表 直接将`chunk`链接到`fwd`后面

```c
if ((unsigned long) size== (unsigned long) chunksize\_nomask (fwd))
fwd = fwd->fd;
```

- 当我们需求的`chunk size`大于`large`中所有的`chunk size`的情况 执行`largebin`的入`chunk_size`链操作:<span id = "attack"></span>

> 这里我理解的是`largebin`存在两条链 也就是`chunk size`的链 和`fd bk`构成的`bins`链 这里先是入的`chunk size`的链

```c
victim->fd\_nextsize = fwd;
victim->bk\_nextsize = fwd->bk\_nextsize;
if (\_\_glibc\_unlikely (fwd->bk\_nextsize->fd\_nextsize != fwd))
malloc\_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
fwd->bk\_nextsize = victim;
victim->bk\_nextsize->fd\_nextsize = victim;
```

- 这里就是重点了 也就是`large bin`的入链操作

- 首先这是初始状态

![status01](upload/attach/202403/994584\_98TSV6DGHK6H4BU.jpg) ![status1](upload/attach/202403/994584\_DPSDMQ3PZN9K955.jpg)![status1](upload/attach/202403/994584\_U34AZN3Y7CTQKNB.jpg)![status1](upload/attach/202403/994584\_9TQ69NTSAAX3N2M.jpg)

- 让`bck`等于`fwd->bk` 也就是把`bck`提到`fwd`前方 并且进行安全检查

```c
bck = fwd->bk;
if (bck->fd != fwd)
malloc\_printerr ("malloc(): largebin double linked list corrupted (bk)");
```

- 最后就是执行入链操作了

> 在一开始的时候提过

```c
mark\_bin (av, victim\_index);
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

### 从largebin中获取chunk

- `largebin`情况

```c
if (!in\_smallbin\_range (nb))
```

### chunk脱链 remainder chunk入链

> 首先是判断情况 我们只处理这一种情况:`largebin`中有`chunk` 然后`largebin`中最大的`chunk`大于我们的需求

> 接下来的代码都是从`largebin`中获取`chunk`

```c
if ((victim = first (bin)) != bin && (unsigned long) chunksize\_nomask (victim)>= (unsigned long) (nb))
```

- 取最小的`chunk` 反方向循环 找到刚好大于等于我们所需`chunk size`的 `chunk`

> 如果一个大小的`chunk`链表中有多个`chunk` 优先取第二个 不轻易改变`chunk size`链表的值

```c
// 取largebin的最后一个chunk 也就是最小的那个chunk
victim = victim->bk\_nextsize;
// 取首个大于所需的chunk size的large chunk
while (((unsigned long) (size = chunksize (victim)) < (unsigned long) (nb)))
victim = victim->bk\_nextsize;
/\* Avoid removing the first entry for a size so that the skip
list does not have to be rerouted. \*/
// 这里避免删除chunk size链中的首个chunk 避免我们修改chunk size链表 所以我们取第二个
if (victim != last (bin) && chunksize\_nomask (victim) == chunksize\_nomask (victim->fd))
victim = victim->fd;
```

- `chunk` 通过`unlink_chunk`脱链 `remainder chunk`入`unsortedbin`链

> 安全检查 是否切割后的`chunk`大于`minsize` 与安全检查 `largebin`第一个`chunk`和头的互锁状态

```c
// 算剩余的remainder\_size
remainder\_size = size - nb;
// 对 我们large bin中的chunk 进行unlink操作
unlink\_chunk (av, victim);
/\* Exhaust \*/
// 安全检查 如果切割的chunk 小于MINSIZE 则 设置下一个chunk p为0
if (remainder\_size < MINSIZE){
set\_inuse\_bit\_at\_offset (victim, size);
if (av != &main\_arena)
set\_non\_main\_arena (victim);
}else{
remainder = chunk\_at\_offset (victim, nb);
/\* We cannot assume the unsorted list is empty and therefore
have to perform a complete insert here. \*/
// 根据注释大概知道是进行完整的插入操作
// 取得unsorted\_chunk bin链表的的头
bck = unsorted\_chunks(av);
// 取 第一个chunk
fwd = bck->fd;
// 安全检查:检查第一个chunk的bk是否为unsorted bin的头
if (\_\_glibc\_unlikely (fwd->bk != bck)){
malloc\_printerr ("malloc(): corrupted unsorted chunks");
}
// remainder 入unsortedbin
remainder->bk = bck;
remainder->fd = fwd;
bck->fd = remainder;
fwd->bk = remainder;
// 如果是remiander 则将fd\_nextsize bk\_nextsize 设置为null
if (!in\_smallbin\_range (remainder\_size)){
remainder->fd\_nextsize = NULL;
remainder->bk\_nextsize = NULL;
}
// 这里应该是设置head的一系列操作
set\_head (victim, nb | PREV\_INUSE |
(av != &main\_arena ? NON\_MAIN\_ARENA : 0));
set\_head (remainder, remainder\_size | PREV\_INUSE);
// foot就是下一个chunk的prev\_size部分
set\_foot (remainder, remainder\_size);
}
```

### 返回被切割后的chunk

```c
check\_malloced\_chunk (av, victim, nb);
void \*p = chunk2mem (victim);
alloc\_perturb (p, bytes);
return p;
```

### 从topchunk中获取chunk

> 我是大概浏览的 大概意思是去剩下的chunk中寻找 如果没找到就去topchunk分配 如果topchunk不够就去系统申请

## 漏洞利用

> 我们主要是利用:[largechunk中最大的chunk还是小于我们所需求的chunk大小](#attack)这种情况 我们来详细分析一下这个流程中究竟干了什么

```c
victim->fd\_nextsize = fwd;
victim->bk\_nextsize = fwd->bk\_nextsize;
if (\_\_glibc\_unlikely (fwd->bk\_nextsize->fd\_nextsize != fwd))
malloc\_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
fwd->bk\_nextsize = victim;
victim->bk\_nextsize->fd\_nextsize = victim;
// 以及
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

- 我们可以发现 这里的代码 危险的地方在于 如果现在我们能够修改`largebin`中`fwd`位置的`chunk` 我们就能够泄露`victim`的地址

> 我们主要利用这两行代码

```
victim->bk\_nextsize->fd\_nextsize = victim;
bck->fd = victim;
```

- 如何实现？比如

- 我们修改`largebin`中的`chunk` 也就是`fwd`的`bk`为我们想要泄露到的`目标地址-0x10`时

- 所以`fwd->bk->fd`也就是`目标地址`

- 阅读前后逻辑我们知道这段代码中`bck=fwd->bk`

- `bck->fd` 最后被赋值`victim`

- 所以也就是`fwd->bk->fd`被赋值`victim` 也就是目标地址赋值`victim`

- 我们修改`fwd`的`bk_nextsize`为`目标地址-0x20`

- 所以`fwd->bk_nextsize->fd_nextsize`等于目标地址

- 然后也因为`victim->bk_nextsize = fwd->bk_nextsize;` 和`victim->bk_nextsize->fd_nextsize = victim`所以等价替换

- `fwd->bk_nextsize->fd_nextsize=victim`也就是目标地址等于`victim`

# large Bin Attack学习（\_int\_malloc源码细读 ）

> 参考文章:

> wiki:[Large Bin Attack - CTF Wiki (ctf-wiki.org)](https://bbs.kanxue.com/elink@7d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6D9j5i4u0Y4k6g2)9J5k6r3u0A6L8W2)9J5k6r3q4@1N6r3q4U0K9#2)9J5c8R3`.`.)

> 源码级调试glibc:[源码级调试glibc\_glibc cannot be compiled without optimization-CSDN博客](https://bbs.kanxue.com/elink@2d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4K6N6s2u0G2N6s2W2U0L8$3!0F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2x3U0j5$3x3U0j5^5y4b7`.`.)

> 源码分析:[glibc 2.31 malloc与free 源码分析（持续更新） - PwnKi - 博客园 (cnblogs.com)](https://bbs.kanxue.com/elink@16aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2g2G2L8r3g2I4K9g2)9J5c8Y4m8Q4x3V1j5I4y4e0f1J5x3o6j5J5x3g2)9J5k6h3S2@1L8h3H3`.)+[glibc malloc源码分析 - PwnKi - 博客园 (cnblogs.com)](https://bbs.kanxue.com/elink@90aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2g2G2L8r3g2I4K9g2)9J5c8Y4m8Q4x3V1j5I4x3U0M7K6x3e0R3%4y4g2)9J5k6h3S2@1L8h3I4Q4x3U0y4Q4y4h3k6A6L8Y4c8Q4y4h3k6E0j5h3I4D9L8$3x3`.)

> 详细拆分了\_int\_malloc的流程 并且按照功能分了标题 想要了解对应部分就直接点击标题跳转即可

> 第一次阅读glibc的源码然后进行分析 有错误的地方请大佬指正

## 源码分析(largebin malloc)

> 每次去看别人文章分析总结的 总感觉比较难记住 每个libc版本的区别 然后也没彻底理解一些操作 所以进行阅读源码

> 然后重点是检查机制部分 如果只想看重点就直接跳转到[largebin入链操作](https://bbs.kanxue.com/#largebin)

- 然后在正式阅读源码之前 我们先理清楚`largebin`的结构（去除了头部的`fd_nextsize/bk_nextsize` 为了图片干净一点）

  ![largebin_struct](https://bbs.kanxue.com/upload/attach/202403/994584_2QBMNU43EECPWAJ.jpg)

  - 我们可以简化一下 去除尾链的`fd`和头链的`bk`方便我们理清逻辑

    ![large_struct](https://bbs.kanxue.com/upload/attach/202403/994584_FRH86PQD27AKCJB.jpg)
