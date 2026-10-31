
```cpp
void
internal_function
_dl_fini (void)
{
/* Lots of fun ahead. We have to call the destructors for all still
   loaded objects, in all namespaces. The problem is that the ELF
   specification now demands that dependencies between the modules
   are taken into account. I.e., the destructor for a module is
   called before the ones for any of its dependencies.
   To make things more complicated, we cannot simply use the reverse
   order of the constructors. Since the user might have loaded objects
   using `dlopen' there are possibly several other modules with its
   dependencies to be taken into account. Therefore we have to start
   determining the order of the modules once again from the beginning. */
/* We run the destructors of the main namespaces last. As for the
   other namespaces, we pick run the destructors in them in reverse
   order of the namespace ID. */
#ifdef SHARED
  int do_audit = 0;
again:
#endif
  for (Lmid_t ns = GL(dl_nns) - 1; ns >= 0; --ns)
    {
      /* Protect against concurrent loads and unloads.  */
      __rtld_lock_lock_recursive (GL(dl_load_lock));
      unsigned int nloaded = GL(dl_ns)[ns]._ns_nloaded;
      /* No need to do anything for empty namespaces or those used for
         auditing DSOs.  */
      if (nloaded == 0
#ifdef SHARED
          || GL(dl_ns)[ns]._ns_loaded->l_auditing != do_audit
#endif
          )
        __rtld_lock_unlock_recursive (GL(dl_load_lock));
      else
        {
          /* Now we can allocate an array to hold all the pointers and
             copy the pointers in.  */
          struct link_map *maps[nloaded];
          unsigned int i;
          struct link_map *l;
          assert (nloaded != 0 || GL(dl_ns)[ns]._ns_loaded == NULL);
          for (l = GL(dl_ns)[ns]._ns_loaded, i = 0; l != NULL; l = l->l_next)
            /* Do not handle ld.so in secondary namespaces.  */
            if (l == l->l_real)
              {
                assert (i < nloaded);
                maps[i] = l;
                l->l_idx = i;
                ++i;
                /* Bump l_direct_opencount of all objects so that they
                   are not dlclose()ed from underneath us.  */
                ++l->l_direct_opencount;
              }
          assert (ns != LM_ID_BASE || i == nloaded);
          assert (ns == LM_ID_BASE || i == nloaded || i == nloaded - 1);
          unsigned int nmaps = i;
          /* Now we have to do the sorting.  */
          _dl_sort_fini (maps, nmaps, NULL, ns);
          /* We do not rely on the linked list of loaded object anymore
             from this point on.  We have our own list here (maps).  The
             various members of this list cannot vanish since the open
             count is too high and will be decremented in this loop.  So
             we release the lock so that some code which might be called
             from a destructor can directly or indirectly access the
             lock.  */
          __rtld_lock_unlock_recursive (GL(dl_load_lock));
          /* 'maps' now contains the objects in the right order.  Now
             call the destructors.  We have to process this array from
             the front.  */
          for (i = 0; i < nmaps; ++i)
            {
              struct link_map *l = maps[i];
              if (l->l_init_called)
                {
                  /* Make sure nothing happens if we are called twice.  */
                  l->l_init_called = 0;
                  /* Is there a destructor function?  */
                  if (l->l_info[DT_FINI_ARRAY] != NULL
                      || l->l_info[DT_FINI] != NULL)
                    {
                      /* When debugging print a message first.  */
                      if (__builtin_expect (GLRO(dl_debug_mask)
                                            & DL_DEBUG_IMPCALLS, 0))
                        _dl_debug_printf ("\ncalling fini: %s [%lu]\n\n",
                                          DSO_FILENAME (l->l_name),
                                          ns);
                      /* First see whether an array is given.  */
                      if (l->l_info[DT_FINI_ARRAY] != NULL)
                        {
                          ElfW(Addr) *array =
                            (ElfW(Addr) *) (l->l_addr
                                            + l->l_info[DT_FINI_ARRAY]->d_un.d_ptr);
                          unsigned int i = (l->l_info[DT_FINI_ARRAYSZ]->d_un.d_val
                                            / sizeof (ElfW(Addr)));
                          while (i-- > 0)
                            ((fini_t) array[i]) ();
                        }
                      /* Next try the old-style destructor.  */
                      if (l->l_info[DT_FINI] != NULL)
                        DL_CALL_DT_FINI
                          (l, l->l_addr + l->l_info[DT_FINI]->d_un.d_ptr);
                    }
#ifdef SHARED
                  /* Auditing checkpoint: another object closed.  */
                  if (!do_audit && __builtin_expect (GLRO(dl_naudit) > 0, 0))
                    {
                      struct audit_ifaces *afct = GLRO(dl_audit);
                      for (unsigned int cnt = 0; cnt < GLRO(dl_naudit); ++cnt)
                        {
                          if (afct->objclose != NULL)
                            /* Return value is ignored.  */
                            (void) afct->objclose (&l->l_audit[cnt].cookie);
                          afct = afct->next;
                        }
                    }
#endif
                }
              /* Correct the previous increment.  */
              --l->l_direct_opencount;
            }
        }
    }
#ifdef SHARED
  if (! do_audit && GLRO(dl_naudit) > 0)
    {
      do_audit = 1;
      goto again;
    }
  if (__glibc_unlikely (GLRO(dl_debug_mask) & DL_DEBUG_STATISTICS))
    _dl_debug_printf ("\nruntime linker statistics:\n"
                      "  final number of relocations: %lu\n"
                      "final number of relocations from cache: %lu\n",
                      GL(dl_num_relocations),
                      GL(dl_num_cache_relocations));
#endif
}
```

只要伪造 `rtld_global` 结构体就可以使得 `array` 指向我们可控的数据区，从而伪造好一系列函数，进而劫持程序的流。可以触发 `call` 的有两个点，第一个点可以 `call` 到很多指针，是一个数组；另一个点就只有一个函数。剩下的工作就是根据代码绕过检测，调用到调用点，需要注意的是，有时候远程的 `rtld_global` 的偏移与本地不一样，需要爆破。`house of banana`便是利用`large bin attack`往 `rtld_global` 写入堆的地址，并事先在堆里伪造好`rtld_global`结构体，这样程序`exit`或者正常退出 `main` 函数时，便会执行到伪造的函数，此时若我们将函数伪造成`one_gadget`或者`system` 则可以 `get shell`。

### 利用思路

- 直接伪造 `_rtld_global` 的 `_ns_loaded`，布局好其他内容，使其调用到 `fini_array`
- 伪造 `link_map` 的 `next` 指针，布局好其他内容，使其调用到 `fini_array`
- 修改 `link_map->l_addr`，根据偏移使其调用到指定区域的函数

### 试用场景

- 程序能够显式的执行 `exit` 函数
- 程序通过 `libc_start_main` 启动的主函数，且主函数能够结束

### 需绕过的点

劫持 `&(_rtld_global._dl_ns._ns_loaded->l_next->l_next->l_next) = fake`。

- `check0`：`fake+0x28 = fake`

`maps` 必须要有四个元素，所以我劫持的是第三个节点的 `next` 指针这样不会破环长度从而绕过下面的两个断言。劫持时只需在 `_rtld_global._dl_ns._ns_loaded->l_next->l_next->l_next` 处写入 `fake` 就行，这时可以使用 `large bin attack`。

```cpp
assert (ns != LM_ID_BASE || i == nloaded);
assert (ns == LM_ID_BASE || i == nloaded || i == nloaded - 1);
```

为了能写入 `maps[i] = l;`，需要绕过 `if (l == l->l_real)`，所以 `fake+0x28 (offset is 0x28)` 处要写入 `fake` 自己的地址。

```c
struct link_map
{
  /* These first few members are part of the protocol with the debugger.
     This is the same format used in SVR4.  */
  ElfW(Addr) l_addr;  /* Difference between the address in the ELF
                         file and the addresses in memory.  */
  char *l_name;       /* Absolute file name object was found in.  */
  ElfW(Dyn) *l_ld;    /* Dynamic section of the shared object.  */
  struct link_map *l_next, *l_prev; /* Chain of loaded objects.  */
  /* All following members are internal to the dynamic linker.
     They may change without notice.  */
  /* This is an element which is only ever different from a pointer to
     the very same copy of this type for ld.so when it is used in more
     than one namespace.  */
  struct link_map *l_real;
  ......
}
if (l == l->l_real)
{
  assert (i < nloaded);
  maps[i] = l;
  l->l_idx = i;
  ++i;
  /* Bump l_direct_opencount of all objects so that they
     are not dlclose()ed from underneath us.  */
  ++l->l_direct_opencount;
}
```

- `check1`：`glibc_2.23==>fake+0x314 = 0x1c || glibc_2.31==>fake+0x31c = 0x1c`

是个枚举体中成员 `l_init_called`，由于各版本有所差异，所以还是现查现用。

```c
struct link_map
{
  ...
  unsigned int l_relocated:1;  /* Nonzero if object's relocations done.  */
  unsigned int l_init_called:1; /* Nonzero if DT_INIT function called.  */
  unsigned int l_global:1;     /* Nonzero if object in _dl_global_scope.  */
  unsigned int l_reserved:2;   /* Reserved for internal use.  */
  ...
}
```

```c
void
_dl_fini (void)
{
[...]
  /* 'maps' now contains the objects in the right order.  Now
     call the destructors.  We have to process this array from
     the front.  */
  for (i = 0; i < nmaps; ++i)
    {
      struct link_map *l = maps[i];
      if (l->l_init_called)
        {
          /* Make sure nothing happens if we are called twice.  */
          l->l_init_called = 0;
          ...
        }
}
```

- `check2&check3`：`l->l_info[DT_FINI_ARRAY] != NULL`
