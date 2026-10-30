
博客有我的联系方式,欢迎大家来玩,地址:<mark class="encrypted">078K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1H3P5o6u0D9i4K6u0W2j5$3^5`.</mark>

最近要出一趟远门,回来之后也就要实习了,估摸着是没时间再写了,后面看情况吧,毕竟主要研究平台还是Windows.

## 何为Teache

从`Glibc` 2.26版本开始,新引入了一个`tcache`的技术,旨在提升内存分配的速度,但是提升速度的时候舍弃了很多检查,因此导致了很多新的安全问题.接下来以2.27版本的源码稍微分析一下.

### 相关结构体

```
#if USE_TCACHE



/* We overlay this structure on the user-data portion of a chunk when

   the chunk is stored in the per-thread cache.  */

typedef struct tcache_entry

{

  struct tcache_entry *next;

} tcache_entry;

```

`tcache_entry`用单向链表来链接`Tcache`结构,`next`指针指向下一个相同大小的`chunk`.与`fast bin`不同的是,`next`指向`user data`,而不是指向`chunk`的开头.

```
# define TCACHE_MAX_BINS        64



typedef struct tcache_perthread_struct

{

  char counts[TCACHE_MAX_BINS];

  tcache_entry *entries[TCACHE_MAX_BINS];

} tcache_perthread_struct;

```

`tcache_perthread_struct`是整个`Tcache`的管理结构.

`counts`用来记录`entries`中每一项当前链入的`chunk`数目,最多可以有七个`chunk`.

`entries`保存了64项`tcache_entry`,也就是说一个管理结构最多可以拥有64个`tcache_entry`.

### 基本操作

```
/* Caller must ensure that we know tc_idx is valid and there's room

   for more chunks.  */

static __always_inline void

tcache_put (mchunkptr chunk, size_t tc_idx)

{

  tcache_entry *e = (tcache_entry *) chunk2mem (chunk);

  assert (tc_idx < TCACHE_MAX_BINS);

  e->next = tcache->entries[tc_idx];

  tcache->entries[tc_idx] = e;

  ++(tcache->counts[tc_idx]);

}
```

`tcache_put`函数首先把`chunk`的`user data`部分转化为一个`tcache_entry *`类型,然后插入到`tcache->entries[tc_idx]`的头部,然后把`tcache->counts[tc_idx]`加一,用来表示新增了一个`chuank`到结构体中.

```
/* Caller must ensure that we know tc_idx is valid and there's

   available chunks to remove.  */

static __always_inline void *

tcache_get (size_t tc_idx)

{

  tcache_entry *e = tcache->entries[tc_idx];

  assert (tc_idx < TCACHE_MAX_BINS);

  assert (tcache->entries[tc_idx] > 0);

  tcache->entries[tc_idx] = e->next;

  --(tcache->counts[tc_idx]);

  return (void *) e;

}
```

`tcache_get`函数从结构体中取出一个`tcache`,将他以`void *`的形式返回,并且把`tcache->counts[tc_idx]`的值减一,用来表示结构体中少了一个`chunk`.

### Tcache的初始化

```
void *

__libc_malloc (size_t bytes)

{

  // 省略部分无关代码

#if USE_TCACHE

  /* int_free also calls request2size, be careful to not pad twice.  */

  size_t tbytes;

  checked_request2size (bytes, tbytes);

  size_t tc_idx = csize2tidx (tbytes);



  MAYBE_INIT_TCACHE ();



  DIAG_PUSH_NEEDS_COMMENT;

  if (tc_idx < mp_.tcache_bins

      /*&& tc_idx < TCACHE_MAX_BINS*/ /* to appease gcc */

      && tcache

      && tcache->entries[tc_idx] != NULL)

    {

      return tcache_get (tc_idx);

    }

  DIAG_POP_NEEDS_COMMENT;

#endif



  if (SINGLE_THREAD_P)

    {

      victim = _int_malloc (&main_arena, bytes);

      assert (!victim || chunk_is_mmapped (mem2chunk (victim)) ||

          &main_arena == arena_for_chunk (mem2chunk (victim)));

      return victim;

    }

  arena_get (ar_ptr, bytes);



  victim = _int_malloc (ar_ptr, bytes);

```

首先根据传入的参数`bytes`计算所需`chuank`的实际大小,然后计算`Tcache`对应的下标`tc_idx`.

如果是第一次`malloc`的话,会调用`MAYBE_INIT_TCACHE()`来对`Tcache`进行初始化.初始化之后,如果下标`tc_idx`没有越界,并且`tcache->entries[tc_idx]`不为空的话,就会调用`tcache_get(tc_idx)`来获取`Tcache`中的一块`chunk`.

上述条件不成立的话,程序才会进入常规的内存分配中,这里就可以看出`Tcache`的优先级非常高,比`fast bin`还要更早被分配.

```
static void

tcache_init(void)

{

  mstate ar_ptr;

  void *victim = 0;

  const size_t bytes = sizeof (tcache_perthread_struct);



  if (tcache_shutting_down)

    return;



  // 找到可用的 arena

  arena_get (ar_ptr, bytes);

  // 申请一个 sizeof(tcache_prethread_struct) 大小的 chunk

  victim = _int_malloc (ar_ptr, bytes);

  if (!victim && ar_ptr != NULL)

    {

      ar_ptr = arena_get_retry (ar_ptr, bytes);

      victim = _int_malloc (ar_ptr, bytes);

    }



  if (ar_ptr != NULL)

    __libc_lock_unlock (ar_ptr->mutex);



  /* In a low memory situation, we may not be able to allocate memory

     - in which case, we just keep trying later.  However, we

     typically do this very early, so either there is sufficient

     memory, or there isn't enough memory to do non-trivial

     allocations anyway.  */

  // 初始化Tcache

  if (victim)

    {

      tcache = (tcache_perthread_struct *) victim;

      memset (tcache, 0, sizeof (tcache_perthread_struct));

    }

}
```

`tcache_init()`成功执行之后,`tcache_perthread_struct`就被成功建立了.

### 申请内存

```
static void *

_int_malloc (mstate av, size_t bytes)

{

    if ((unsigned long) (nb) <= (unsigned long) (get_max_fast ()))

    {

        idx = fastbin_index (nb);

        mfastbinptr *fb = &fastbin (av, idx);

        mchunkptr pp;

        victim = *fb;



        if (victim != NULL)

        {

            if (SINGLE_THREAD_P)

                *fb = victim->fd;

            else

                REMOVE_FB (fb, pp, victim);

            if (__glibc_likely (victim != NULL))

            {

                size_t victim_idx = fastbin_index (chunksize (victim));

                if (__builtin_expect (victim_idx != idx, 0))

                    malloc_printerr ("malloc(): memory corruption (fast)");

                check_remalloced_chunk (av, victim, nb);

                #if USE_TCACHE

                /* While we're here, if we see other chunks of the same size,

                stash them in the tcache.  */

                size_t tc_idx = csize2tidx (nb);

                if (tcache && tc_idx < mp_.tcache_bins)

                {

                    mchunkptr tc_victim;



                    /* While bin not empty and tcache not full, copy chunks.  */

                    while (tcache->counts[tc_idx] < mp_.tcache_count

                    && (tc_victim = *fb) != NULL)

                    {

                        if (SINGLE_THREAD_P)

                            *fb = tc_victim->fd;

                        else

                        {

                            REMOVE_FB (fb, pp, tc_victim);

                            if (__glibc_unlikely (tc_victim == NULL))

                                break;

                        }

                        tcache_put (tc_victim, tc_idx);

                    }

                }

                #endif

                void *p = chunk2mem (victim);

                alloc_perturb (p, bytes);

                return p;

            }

        }

    }

```

在相应大小的`fast bin`找到`chunk`之后,就把`chunk`从`fast bin`中拿出来.然后把相应大小的`fast bin`中剩下的`chunk`全部加入`Tcache`,直到放满为止(也就是满七个为止).最后返回最开始那个满足条件的`chunk`.

如果`fast bin`不满足分配的话,接着进入`small bin`的分配:

```
if (in_smallbin_range (nb))

    {

      idx = smallbin_index (nb);

      bin = bin_at (av, idx);



      if ((victim = last (bin)) != bin)

        {

          bck = victim->bk;

      if (__glibc_unlikely (bck->fd != victim))

        malloc_printerr ("malloc(): smallbin double linked list corrupted");

          set_inuse_bit_at_offset (victim, nb);

          bin->bk = bck;

          bck->fd = bin;



          if (av != &main_arena)

        set_non_main_arena (victim);

          check_malloced_chunk (av, victim, nb);

#if USE_TCACHE

      /* While we're here, if we see other chunks of the same size,

         stash them in the tcache.  */

      size_t tc_idx = csize2tidx (nb);

      if (tcache && tc_idx < mp_.tcache_bins)

        {

          mchunkptr tc_victim;



          /* While bin not empty and tcache not full, copy chunks over.  */

          while (tcache->counts[tc_idx] < mp_.tcache_count

             && (tc_victim = last (bin)) != bin)

        {

          if (tc_victim != 0)

            {

              bck = tc_victim->bk;

              set_inuse_bit_at_offset (tc_victim, nb);

              if (av != &main_arena)

            set_non_main_arena (tc_victim);

              bin->bk = bck;

              bck->fd = bin;



              tcache_put (tc_victim, tc_idx);

                }

        }

        }

#endif

          void *p = chunk2mem (victim);

          alloc_perturb (p, bytes);

          return p;

        }

    }

```

和`fast bin`基本一样.在相应大小的`small bin`找到`chunk`之后,就把`chunk`从`small bin`中拿出来.然后把相应大小的`small bin`中剩下的`chunk`全部加入`Tcache`,直到放满为止(也就是满七个为止).最后返回最开始那个满足条件的`chunk`.

如果`small bin`不满足分配的话,接着进入`unsorted bin`的分配:
