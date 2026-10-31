
```c
struct _IO_str_fields {
  /* These members are preserved for ABI compatibility.  The glibc
     implementation always calls malloc/free for user buffers if
     _IO_USER_BUF or _IO_FLAGS2_USER_WBUF are not set.  */
  _IO_alloc_type _allocate_buffer_unused;
  _IO_free_type _free_buffer_unused;
};

/* This is needed for the Irix6 N32 ABI, which has a 64 bit off_t type,
   but a 32 bit pointer type.  In this case, we get 4 bytes of padding
   after the vtable pointer.  Putting them in a structure together solves
   this problem.  */

struct _IO_streambuf {
  FILE _f;
  const struct _IO_jump_t *vtable;
};

typedef struct _IO_strfile_ {
  struct _IO_streambuf _sbf;
  struct _IO_str_fields _s;
} _IO_strfile;
```

```c
#define _IO_MAGIC 0xFBAD0000 /* Magic number */
#define _OLD_STDIO_MAGIC 0xFABC0000 /* Emulate old stdio. */
#define _IO_MAGIC_MASK 0xFFFF0000
#define _IO_USER_BUF 1 /* User owns buffer; don't delete it on close. */
#define _IO_UNBUFFERED 2
#define _IO_NO_READS 4 /* Reading not allowed */
#define _IO_NO_WRITES 8 /* Writing not allowd */
#define _IO_EOF_SEEN 0x10
#define _IO_ERR_SEEN 0x20
#define _IO_DELETE_DONT_CLOSE 0x40 /* Don't call close(_fileno) on cleanup. */
#define _IO_LINKED 0x80 /* Set if linked (using _chain) to streambuf::_list_all.*/
#define _IO_IN_BACKUP 0x100
#define _IO_LINE_BUF 0x200
#define _IO_TIED_PUT_GET 0x400 /* Set if put and get pointer logicly tied. */
#define _IO_CURRENTLY_PUTTING 0x800
#define _IO_IS_APPENDING 0x1000
#define _IO_IS_FILEBUF 0x2000
#define _IO_BAD_SEEN 0x4000
#define _IO_USER_LOCK 0x8000
```

## 图解

![图片描述](https://bbs.kanxue.com/upload/attach/202310/958172_NGSM9YERY4WZ923.webp)

## IO相关函数概述

- `fopen` 未调用 `vtable` 中的函数，`fopen` 对应的函数 `__fopen_internal` 内部会调用 `malloc` 函数，分配 `FILE` 结构的空间。因此我们可以获知 `FILE` 结构是存储在堆上的。
- 流程
    - 使用 `malloc` 分配 `FILE` 结构
    - 设置 `FILE` 结构的 `vtable`
    - 初始化分配的 `FILE` 结构
    - 将初始化的 `FILE` 结构链入 `FILE` 结构链表头部
    - 调用系统调用打开文件

```c
// fopen() 函数
// libio/iofopen.c
_IO_FILE *
__fopen_internal (const char *filename, const char *mode, int is32)
{
  struct locked_FILE
  {
    struct _IO_FILE_plus fp;
#ifdef _IO_MTSAFE_IO
    _IO_lock_t lock;
#endif
    struct _IO_wide_data wd;
  } *new_f = (struct locked_FILE *) malloc (sizeof (struct locked_FILE)); // 为 FILE 结构分配空间
  if (new_f == NULL)
    return NULL;
#ifdef _IO_MTSAFE_IO
  new_f->fp.file._lock = &new_f->lock;
#endif
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
  _IO_no_init (&new_f->fp.file, 0, 0, &new_f->wd, &_IO_wfile_jumps);
#else
  _IO_no_init (&new_f->fp.file, 1, 0, NULL, NULL);
#endif
  _IO_JUMPS (&new_f->fp) = &_IO_file_jumps; // 设置 vtable = &_IO_file_jumps
  _IO_file_init (&new_f->fp); // 调用 _IO_file_init 函数进行初始化
#if !_IO_UNIFIED_JUMPTABLES
  new_f->fp.vtable = NULL;
#endif
  if (_IO_file_fopen ((_IO_FILE *) new_f, filename, mode, is32) != NULL) // 打开目标文件
    return __fopen_maybe_mmap (&new_f->fp.file);
  _IO_un_link (&new_f->fp);
  free (new_f);
  return NULL;
}

_IO_FILE *
_IO_new_fopen (const char *filename, const char *mode)
{
  return __fopen_internal (filename, mode, 1);
}

// libio/fileops.c
# define _IO_new_file_init _IO_file_init
void
_IO_new_file_init (struct _IO_FILE_plus *fp)
{
  /* POSIX.1 allows another file handle to be used to change the position
     of our file descriptor.  Hence we actually don't know the actual
     position before we do the first fseek (and until a following fflush). */
  fp->file._offset = _IO_pos_BAD;
  fp->file._IO_file_flags |= CLOSED_FILEBUF_FLAGS;
  _IO_link_in (fp); // 调用 _IO_link_in 函数将 fp 放进链表
  fp->file._fileno = -1;
}

// libio/genops.c
void
_IO_link_in (struct _IO_FILE_plus *fp)
{
  if ((fp->file._flags & _IO_LINKED) == 0)
    {
      fp->file._flags |= _IO_LINKED;
#ifdef _IO_MTSAFE_IO
      _IO_cleanup_region_start_noarg (flush_cleanup);
      _IO_lock_lock (list_all_lock);
      run_fp = (_IO_FILE *) fp;
      _IO_flockfile ((_IO_FILE *) fp);
#endif
      fp->file._chain = (_IO_FILE *) _IO_list_all; // fp 放到链表头部
      _IO_list_all = fp; // 链表头 _IO_list_all 指向 fp
      ++_IO_list_all_stamp;
#ifdef _IO_MTSAFE_IO
      _IO_funlockfile ((_IO_FILE *) fp);
      run_fp = NULL;
      _IO_lock_unlock (list_all_lock);
      _IO_cleanup_region_end (0);
#endif
    }
}
```

- `fread()` 最终会调用 `_IO_fiel_xsgetn`
    - 流程大致为 `_IO_fread->_IO_sgetn->_IO_XSGETN->_IO_file_xsgetn`。

```c
// fread
// libio/iofread.c
/*
* buf: 存放读取数据的缓冲区。
* size: 指定每个记录的长度。
* count: 指定记录的个数。
* stream: 目标文件流。
* 返回值: 返回读取到数据缓冲区中的记录个数。
*/
_IO_size_t
_IO_fread (void *buf, _IO_size_t size, _IO_size_t count, _IO_FILE *fp)
{
  _IO_size_t bytes_requested = size * count;
  _IO_size_t bytes_read;
  CHECK_FILE (fp, 0);
  if (bytes_requested == 0)
    return 0;
  _IO_acquire_lock (fp);
  bytes_read = _IO_sgetn (fp, (char *) buf, bytes_requested); // 调用 _IO_sgetn 函数
  _IO_release_lock (fp);
  return bytes_requested == bytes_read ? count : bytes_read / size;
}

// libio/genops.c
_IO_size_t
_IO_sgetn (_IO_FILE *fp, void *data, _IO_size_t n)
{
  /* FIXME handle putback buffer here! */
  return _IO_XSGETN (fp, data, n); // 调用宏 _IO_XSGETN
}

// libio/libioP.h
#define _IO_JUMPS_FILE_plus(THIS) \
  _IO_CAST_FIELD_ACCESS ((THIS), struct _IO_FILE_plus, vtable)
#if _IO_JUMPS_OFFSET
#   define _IO_JUMPS_FUNC(THIS) \
  (*(struct _IO_jump_t **) ((void *) &_IO_JUMPS_FILE_plus (THIS) \
    + (THIS)->_vtable_offset))
#   define _IO_vtable_offset(THIS) (THIS)->_vtable_offset
#else
#   define _IO_JUMPS_FUNC(THIS) _IO_JUMPS_FILE_plus (THIS)
#   define _IO_vtable_offset(THIS) 0
#endif
#define JUMP2(FUNC, THIS, X1, X2) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1, X2)
#define _IO_XSGETN(FP, DATA, N) JUMP2 (__xsgetn, FP, DATA, N)
```

- `fwrite` 最终会调用 `_IO_file_xsputn`
    - 流程 `_IO_fwrite->_IO_XSPUTN->_IO_new_file_xsputn->_IO_OVERFLOW->_IO_new_file_overflow`
