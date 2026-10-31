
```c
/* The tag name of this struct is _IO_FILE to preserve historic
   C++ mangled names for functions taking FILE* arguments.
   That name should not be used in new code.  */
struct _IO_FILE
{
  int _flags; /* High-order word is _IO_MAGIC; rest is flags. */

  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr; /* Current read pointer */
  char *_IO_read_end; /* End of get area. */
  char *_IO_read_base; /* Start of putback+get area. */
  char *_IO_write_base; /* Start of put area. */
  char *_IO_write_ptr; /* Current put pointer. */
  char *_IO_write_end; /* End of put area. */
  char *_IO_buf_base; /* Start of reserve area. */
  char *_IO_buf_end; /* End of reserve area. */

  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base; /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */

  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};

struct _IO_FILE_complete
{
  struct _IO_FILE _file;
#endif
  __off64_t _offset;
  /* Wide character stream stuff.  */
  struct _IO_codecvt *_codecvt;
  struct _IO_wide_data *_wide_data;
  struct _IO_FILE *_freeres_list;
  void *_freeres_buf;
  size_t __pad5;
  int _mode;
  /* Make sure we don't get into trouble again.  */
  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
};
```

`_IO_FILE_complete`可以理解为`_IO_FILE`的豪华加长版，其中的`_wide_data`需要关注一下，有些手法会用到。

## `_IO_jump_t`

`vtable`是一个指向`_IO_jump_t`结构体的指针。当打开一个文件时，相应的 `_IO_FILE` 结构体会被创建，并将其 `vtable` 字段指向对应文件类型的 `_IO_jump_t` 结构体的地址。这样，当需要执行文件操作时，可以通过 `_IO_FILE` 结构体中的 `vtable` 字段获取到相应的 `_IO_jump_t` 结构体，并通过其中的函数指针调用相应的文件操作函数。

通常，一个`_IO_jump_t` 结构体包含以下函数指针

```c
struct _IO_jump_t
{
  JUMP_FIELD(size_t, __dummy);
  JUMP_FIELD(size_t, __dummy2);
  JUMP_FIELD(_IO_finish_t, __finish);
  JUMP_FIELD(_IO_overflow_t, __overflow);
  JUMP_FIELD(_IO_underflow_t, __underflow);
  JUMP_FIELD(_IO_underflow_t, __uflow);
  JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
  /* showmany */
  JUMP_FIELD(_IO_xsputn_t, __xsputn);
  JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
  JUMP_FIELD(_IO_seekoff_t, __seekoff);
  JUMP_FIELD(_IO_seekpos_t, __seekpos);
  JUMP_FIELD(_IO_setbuf_t, __setbuf);
  JUMP_FIELD(_IO_sync_t, __sync);
  JUMP_FIELD(_IO_doallocate_t, __doallocate);
  JUMP_FIELD(_IO_read_t, __read);
  JUMP_FIELD(_IO_write_t, __write);
  JUMP_FIELD(_IO_seek_t, __seek);
  JUMP_FIELD(_IO_close_t, __close);
  JUMP_FIELD(_IO_stat_t, __stat);
  JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
  JUMP_FIELD(_IO_imbue_t, __imbue);
#if 0
  get_column;
  set_column;
#endif
};
```

注意，程序中存在很多`_IO_jump_t`结构体，不同的`_IO_FILE_plus`可能采用不同的`_IO_jump_t`，例如`stdin`/`stdout`/`stderr`使用`_IO_file_jumps`，可以通过`gdb`查看

![](https://bbs.kanxue.com/upload/attach/202309/903664_CHAGT8D4CX5P3X5.png)

# FSOP基础

**FSOP** (File Stream Oriented Programming) 是针对`_IO_FILE`结构体与IO操作的攻击方式

通常可以分为两个步骤：**劫持**并**伪造**IO结构体、**触发**IO流

因为触发IO流的方式影响程序进入的IO结构体，所以我们先讲如何触发IO流

## 触发IO流

触发IO流的方式主要分为两种：**FSOP**的经典方式和**house of kiwi**的方式

### FSOP - `_IO_flush_all_lockp()`

该函数的调用有以下三种触发方式：

1. `libc`检测到**内存错误**，从而执行`abort`函数时（在`glibc-2.26`删除）

2. 程序执行`exit`函数

3. 程序从`main`函数返回（同样是执行`exit`）

可以看到，高版本下，基本的触发方式都是通过`exit`函数，其栈回溯为：

```text
_IO_flush_all_lockp (do_lock=do_lock@entry=0x0)
_IO_cleanup ()
__run_exit_handlers (status=0x0, listp=<optimized out>, run_list_atexit=run_list_atexit@entry=0x1)
__GI_exit (status=<optimized out>)
```

就能调用到`_IO_flush_all_lockp()`函数

该函数的代码片段如下：

```c
int _IO_flush_all_lockp (int do_lock)
{
  int result = 0;
  struct _IO_FILE *fp;
  int last_stamp;

  fp = (_IO_FILE *) _IO_list_all;
  while (fp != NULL)
  {
    ...
    if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
    || (_IO_vtable_offset (fp) == 0
        && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
        > fp->_wide_data->_IO_write_base))
#endif
    )
    && _IO_OVERFLOW (fp, EOF) == EOF)
    result = EOF;

    fp = fp->_chain; //遍历链表
  }
  [...]
}
```

下面讲如何达成调用`_IO_OVERFLOW`函数的条件：

```c
if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
|| (_IO_vtable_offset (fp) == 0
&& fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
> fp->_wide_data->_IO_write_base))
#endif
)
&& _IO_OVERFLOW (fp, EOF) == EOF)
```

根据短路原则，执行`_IO_OVERFLOW`需要满足前面的条件为真，这里有一个或语句，故有两种达成条件，满足任一即可：

1. 

   ```
   fp->_mode = 0
   fp->_IO_write_ptr > fp->_IO_write_base
   ```

   这是较为常用的达成条件，只需伪造`_IO_FILE`中的`_IO_write_ptr`、`_IO_write_base`和`_mode`即可达成

2. 

   ```
   _IO_vtable_offset (fp) == 0
   fp->_mode > 0
   fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base
   ```

   该条件可以结合**house of cat**使用。其中`_IO_vtable_offset (fp) == 0` 即 `_vtable_offset`为0，另外需要关注`_wide_data`结构体，该手法会在house of cat中讲解

条件满足后，会从`_IO_list_all`开始沿着`fp -> chain`遍历`FILE`结构体，执行对应虚表中的`_IO_OVERFLOW`，即`<_IO_file_jumps+24>`

### house of kiwi - `__malloc_assert`

该手法用于在程序无`exit`时触发IO，且能控制`rdx`

以下为`__malloc_assert`的代码片段

