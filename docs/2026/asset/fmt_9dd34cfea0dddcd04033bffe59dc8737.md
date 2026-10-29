
```
_IO_size_t

_IO_sgetn (_IO_FILE *fp, void *data, _IO_size_t n)

{

  /* FIXME handle putback buffer here! */

  return _IO_XSGETN (fp, data, n);

}
```

- `_IO_XSGETN` 的跳转定义为 `JUMP2(FUNC,THIS,X1,X2)` ， `this` 传入的是 `FUNC`，在代码中是 `fp` 的地址，所以劫持后的第一个参数的位置在 `fp` 的位置处

```c
#在跳转的时候函数定义为JUMP2(FUNC,THIS,X1,X2)

#define _IO_XSGETN(FP, DATA, N) JUMP2 (__xsgetn, FP, DATA, N)

#define JUMP2(FUNC, THIS, X1, X2) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1, X2)
```

- 劫持分两类讨论

- 第一种，对于 `stderr`,`stdout`,`stdin` 的劫持，只能 `overwrite` `_IO_FILE_plus` 结构体中的 `vtable` 指针，使其指向我们所伪造的跳转表，达到劫持目的，该方法也普适于其他的文件流。

- 第二种，通过 `overwrite` 跳转表中的函数指针来达到覆写的目的

- 要实现以上的任意读写，前提需要已知 `libc` 的基地址，并且程序漏洞可以实现任意地址写

```
libc_base_address -> _IO_list_all -> 任意结构体地址 -> 偏移计算vtable -> vtable表地址

libc_base_address -> _std_in/_std_out/_std_err地址 -> 偏移计算vtable -> vtable表地址
```

### 针对vtable的劫持缓解机制

- 在 `libc-2.24` 版本上，增加了对 `vtable` 的劫持检测机制，在 `Libc-2.23` 版本一下的跳转是如下实现的

```
//libc-2.23

# define _IO_JUMPS_FUNC(THIS) _IO_JUMPS_FILE_plus (THIS)

//libc-2.24

# define _IO_JUMPS_FUNC(THIS) (IO_validate_vtable (_IO_JUMPS_FILE_plus (THIS)))
```

- 在 `libc-2.24` 版本上，增加的检查机制 `IO_validate_vtable` 定义如下,如果 `offset` 大于 `section_length`,则进入 `_IO_vtable_check()` 来检查这个 `vtable` 是否合法。

```
static inline const struct _IO_jump_t *

IO_validate_vtable (const struct _IO_jump_t *vtable)

{

  /* Fast path: The vtable pointer is within the __libc_IO_vtables

     section.  */

  uintptr_t section_length = __stop___libc_IO_vtables - __start___libc_IO_vtables;

  const char *ptr = (const char *) vtable;

  uintptr_t offset = ptr - __start___libc_IO_vtables;

  if (__glibc_unlikely (offset >= section_length))

    /* The vtable pointer is not in the expected section.  Use the

       slow path, which will terminate the process if necessary.  */

    _IO_vtable_check ();

  return vtable;

}
```

### 新的利用方式

- 在 `libc-2.24` 版本以后(大于等于)针对 `IO_FILE` 文件的 `vtable` 劫持增加了检测机制使得无法再使用 `vtable` 来劫持程序流

- 但是 `IO_FILE` 的结构体中的 `_IO_buf_base`，`_IO_buf_end` 仍可以用来读取、写入数据

- 在 `libc` 中不仅仅只有 `_IO_file_jumps` 这么一个 `vtable`,还有一个叫做 `_IO_str_jmps` 的，这个 `vtable` 不在 `check` 的范围内(引自 `ctf-wiki`)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-1-5 18:03

被 `seana` 编辑

，原因：
