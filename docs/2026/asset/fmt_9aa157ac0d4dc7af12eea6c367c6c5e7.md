
而在`_IO_FILE_plus`结构体中，`_chain`的偏移为`0x68`，而`top chunk`之后为`0x8`单位的`last_remainder`，接下来为`unsorted bin`的`fd`与`bk`指针，共`0x10`大小，再之后为`small bin`中的指针（每个`small bin`有`fd`与`bk`指针，共`0x10`个单位），剩下`0x50`的单位，从`smallbin[0]`正好分配到`smallbin[4]`（准确说为其`fd`字段），大小就是从`0x20`到`0x60`，而`smallbin[4]`的`fd`字段中的内容为该链表中最靠近表头的`small bin`的地址 (`chunk header`)，因此`0x60`的`small bin`的地址即为`fake struct`的`_chain`中的内容，只需要控制该`0x60`的`small bin`（以及其下面某些堆块）中的部分内容，即可进行`FSOP`。

### IO_FILE attack 之 利用_fileno字段

`_fileno`的值就是**文件描述符**，位于`stdin`文件结构开头`0x70`偏移处，如：`stdin`的`fileno`为`0`，`stdout`的`fileno`为`1`，`stderr`的`fileno`为`2`。

在漏洞利用中，可以通过修改`stdin`的`_fileno`值来**重定位**需要读取的文件，本来为`0`的话，表示从标准输入中读取，修改为`3`则表示为从文件描述符为`3`的文件（已经`open`的文件）中读取，该利用在某些情况下可直接读取`flag`。

### IO_FILE attack 之 任意读写

1.利用`stdin`进行**任意写**

`scanf`，`fread`，`gets`等读入走`IO`指针（`read`不走）。

**大体流程**为：若`_IO_buf_base`为空，则调用`_IO_doallocbuf`去初始化输入缓冲区，然后判断输入缓冲区是否存在剩余数据，如果输入缓冲区有剩余数据（`_IO_read_end > _IO_read_ptr`）则将其**直接拷贝至目标地址**（不会对此时输入的数据进行读入），如果没有或不够，则调用`__underflow`函数**执行系统调用读取数据**（`SYS_read`）到输入缓冲区（从`_IO_buf_base`到`_IO_buf_end`，默认`0x400`，即将数据读到`_IO_buf_base`，读取`0x400`个字节），此时若实际读入了`n`个字节的数据，则`_IO_read_end = _IO_buf_base + n`（即`_IO_read_end`指向实际读入的最后一个字节的数据），之后再将输入缓冲区中的数据拷贝到目标地址。

这里需要注意的是，若输入缓冲区中没有剩余的数据，则每次读入数据进输入缓冲区，仅和`_IO_buf_base`与`_IO_buf_end`有关。

在将数据从输入缓冲区拷贝到目标地址的过程中，**需要满足所调用的读入函数的自身的限制条件**，例如：使用`scanf("%d",&a)`读入整数，则当在输入缓冲区中遇到了字符（或`scanf`的一些截断符）等不符合的情况，就会停止这个拷贝的过程。最终，`_IO_read_ptr`指向成功拷贝到目的地址中的最后一个字节数据在输入缓冲区中的地址。因此，若是遇到了不符合限制条件的情况而终止拷贝，则最终会使得`_IO_read_end > _IO_read_ptr`，即再下一次读入之前会被认定为输入缓冲区中仍有剩余数据，在此情况下，**很有可能不会进行此次读入**，或将输入缓冲区中剩余的数据拷贝到此次读入的目标地址，从而**导致读入的错误**。

`getchar()`和`IO_getc()`的作用是刷新`_IO_read_ptr`，每次调用，会从输入缓冲区读一个字节数据，即将`_IO_read_ptr++`。

相关源码：

```c
_IO_size_t _IO_file_xsgetn (_IO_FILE *fp, void *data, _IO_size_t n)
{
...
if (fp->_IO_buf_base == NULL)
{
...
//输入缓冲区为空则初始化输入缓冲区
}
while (want > 0)
{
have = fp->_IO_read_end - fp->_IO_read_ptr;
if (have > 0)
{
...
//memcpy
}
if (fp->_IO_buf_base
&& want < (size_t) (fp->_IO_buf_end - fp->_IO_buf_base))
{
if (__underflow (fp) == EOF) // 调用__underflow读入数据
...
}
...
return n - want;
}
```

```c
int _IO_new_file_underflow (_IO_FILE *fp)
{
_IO_ssize_t count;
...
// 会检查_flags是否包含_IO_NO_READS标志，包含则直接返回。
// 标志的定义是#define _IO_NO_READS 4，因此_flags不能包含4。
if (fp->_flags & _IO_NO_READS)
{
fp->_flags |= _IO_ERR_SEEN;
__set_errno (EBADF);
return EOF;
}
// 如果输入缓冲区里存在数据，则直接返回
if (fp->_IO_read_ptr < fp->_IO_read_end)
return *(unsigned char *) fp->_IO_read_ptr;
...
// 调用_IO_SYSREAD函数最终执行系统调用读取数据
count = _IO_SYSREAD (fp, fp->_IO_buf_base,
fp->_IO_buf_end - fp->_IO_buf_base);
...
}
libc_hidden_ver (_IO_new_file_underflow, _IO_file_underflow)
```

综上，为了做到**任意写**，满足如下条件，即可进行利用：

(1) 设置`_IO_read_end`等于`_IO_read_ptr`（使得输入缓冲区内没有剩余数据，从而可以从用户读入数据）。

(2) 设置`_flag &~ _IO_NO_READS`即`_flag &~ 0x4`（一般不用特意设置）。

(3) 设置`_fileno`为`0`（一般不用特意设置）。

(4) 设置`_IO_buf_base`为`write_start`，`_IO_buf_end`为`write_end`（我们目标写的起始地址是`write_start`，写结束地址为`write_end`），且使得`_IO_buf_end-_IO_buf_base`大于要写入的数据长度。

2.利用`stdout`进行**任意读/写**

`printf`，`fwrite`，`puts`等输出走`IO`指针（`write`不走）。

在`_IO_2_1_stdout_`中，`_IO_buf_base`和`_IO_buf_end`为输出缓冲区起始位置（默认大小为`0x400`），在输出的过程中，会先将需要输出的数据从目标地址拷贝到输出缓冲区，再从输出缓冲区输出给用户。

缓冲区建立函数`_IO_doallocbuf`会建立输出缓冲区，并把基地址保存在`_IO_buf_base`中，结束地址保存在`_IO_buf_end`中。在建立里输出缓冲区后，会将基址址给`_IO_write_base`，若是设置的是全缓冲模式`_IO_FULL_BUF`，则会将结束地址给`_IO_write_end`，若是设置的是行缓冲模式`_IO_LINE_BUF`，则`_IO_write_end`中存的是`_IO_buf_base`，此外，`_IO_write_ptr`表示输出缓冲区中已经使用到的地址。即`_IO_write_base`到`_IO_write_ptr`之间的空间是已经使用的缓冲区，`_IO_write_ptr`到`_IO_write_end`之间为剩余的输出缓冲区。

最终实际调用了`_IO_2_1_stdout_`的`vtable`中的`_xsputn`，也就是`_IO_new_file_xsputn`函数，源码如下：

```c
_IO_size_t _IO_new_file_xsputn (_IO_FILE *f, const void *data, _IO_size_t n)
{
const char *s = (const char *) data;
_IO_size_t to_do = n;
int must_flush = 0;
_IO_size_t count = 0;
if (n <= 0)
return 0;
if ((f->_flags & _IO_LINE_BUF) && (f->_flags & _IO_CURRENTLY_PUTTING))
{ //如果是行缓冲模式...
count = f->_IO_buf_end - f->_IO_write_ptr; //判断输出缓冲区还有多少空间
if (count >= n)
{
const char *p;
for (p = s + n; p > s; )
{
if (*--p == '\n') //最后一个换行符\n为截断符，且需要刷新输出缓冲区
{
count = p - s + 1;
must_flush = 1; //标志为真：需要刷新输出缓冲区
break;
}
}
}
}
else if (f->_IO_write_end > f->_IO_write_ptr) //判断输出缓冲区还有多少空间（全缓冲模式）
count = f->_IO_write_end - f->_IO_write_ptr;
if (count > 0)
{
//如果输出缓冲区有空间，则先把数据拷贝至输出缓冲区
if (count > to_do)
count = to_do;
f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);
s += count;
to_do -= count;
}
if (to_do + must_flush > 0) //此处关键，见下文详细讨论
{
_IO_size_t block_size, do_write;
if (_IO_OVERFLOW (f, EOF) == EOF) //调用_IO_OVERFLOW
return to_do == 0 ? EOF : n - to_do;
block_size = f->_IO_buf_end - f->_IO_buf_base;
do_write = to_do - (block_size >= 128 ? to_do % block_size : 0);
if (do_write)
{
count = new_do_write (f, s, do_write);
to_do -= count;
if (count < do_write)
return n - to_do;
}
if (to_do)
to_do -= _IO_default_xsputn (f, s+do_write, to_do);
}
return n - to_do;
}
libc_hidden_ver (_IO_new_file_xsputn, _IO_file_xsputn)
```

（1）**任意写**

可以看到，在行缓冲模式下，判断输出缓冲区还有多少空间，用的是`count = f->_IO_buf_end - f->_IO_write_ptr`，而在全缓冲模式下，用的是`count = f->_IO_write_end - f->_IO_write_ptr`，若是还有空间剩余，则会将要输出的数据复制到输出缓冲区中（此时由`_IO_write_ptr`控制，向`_IO_write_ptr`拷贝`count`长度的数据），因此可通过这一点来实现任意地址写的功能。

**利用方式**：以全缓冲模式为例，只需将`_IO_write_ptr`指向`write_start`，`_IO_write_end`指向`write_end`即可。

这里需要注意的是，有宏定义`#define _IO_LINE_BUF 0x0200`，此处`flag & _IO_LINE_BUF`为真，则表示`flag`中包含了`_IO_LINE_BUF`标识，即开启了行缓冲模式（可用`setvbuf(stdout,0,_IOLBF,1024)`开启），若要构造`flag`包含`_IO_LINE_BUF`标识，则`flag |= 0x200`即可。

（2）**任意读**

先讨论`_IO_new_file_xsputn`源代码中`if (to_do + must_flush > 0)`有哪些情况会执行该分支中的内容：

(a) 首先要明确的是`to_do`一定是非负数，因此若`must_flush`为`1`的时候就会执行该分支中的内容，而再往上看，当需要输出的内容中有`\n`换行符的时候就会需要刷新输出缓冲区，即将`must_flush`设为`1`，故当输出内容中有`\n`的时候就会执行该分支的内容，如用`puts`函数输出就一定会执行。

(b) 若`to_do`大于`0`，也会执行该分支中的内容，因此，当 输出缓冲区未建立 或者 输出缓冲区没有剩余空间 或者 输出缓冲区剩余的空间不够一次性将目标地址中的数据完全拷贝过来 的时候，也会执行该`if`分支中的内容。

而该`if`分支中主要调用了`_IO_OVERFLOW()`来刷新输出缓冲区，而在此过程中会调用`_IO_do_write()`输出我们想要的数据。

相关源码：

```c
int _IO_new_file_overflow (_IO_FILE *f, int ch)
{
// 判断标志位是否包含_IO_NO_WRITES => _flags需要不包含_IO_NO_WRITES
if (f->_flags & _IO_NO_WRITES)
{
f->_flags |= _IO_ERR_SEEN;
__set_errno (EBADF);
return EOF;
}
// 判断输出缓冲区是否为空 以及 是否不包含_IO_CURRENTLY_PUTTING标志位
// 为了不执行该if分支以免出错，最好定义 _flags 包含 _IO_CURRENTLY_PUTTING
if ((f->_flags & _IO_CURRENTLY_PUTTING) == 0 || f->_IO_write_base == NULL)
{
...
}
// 调用_IO_do_write 输出 输出缓冲区
// 从_IO_write_base开始，输出(_IO_write_ptr - f->_IO_write_base)个字节的数据
if (ch == EOF)
return _IO_do_write (f, f->_IO_write_base,
f->_IO_write_ptr - f->_IO_write_base);
return (unsigned char) ch;
}
libc_hidden_ver (_IO_new_file_overflow, _IO_file_overflow)
```

```c
static _IO_size_t new_do_write (_IO_FILE *fp, const char *data, _IO_size_t to_do)
{
...
_IO_size_t count;
// 为了不执行else if分支中的内容以产生错误，可构造_flags包含_IO_IS_APPENDING 或 设置_IO_read_end等于_IO_write_base
if (fp->_flags & _IO_IS_APPENDING)
fp->_offset = _IO_pos_BAD;
else if (fp->_IO_read_end != fp->_IO_write_base)
{
_IO_off64_t new_pos
= _IO_SYSSEEK (fp, fp->_IO_write_base - fp->_IO_read_end, 1);
if (new_pos == _IO_pos_BAD)
return 0;
fp->_offset = new_pos;
}
// 调用函数输出输出缓冲区
count = _IO_SYSWRITE (fp, data, to_do);
...
return count;
}
```

综上，为了做到**任意读**，满足如下条件，即可进行利用：

(1) 设置`_flag &~ _IO_NO_WRITES`，即`_flag &~ 0x8`；

(2) 设置`_flag & _IO_CURRENTLY_PUTTING`，即`_flag | 0x800`；

(3) 设置`_fileno`为`1`；

(4) 设置`_IO_write_base`指向想要泄露的地方，`_IO_write_ptr`指向泄露结束的地址；
