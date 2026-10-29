
```c
const struct \_IO\_jump\_t \_IO\_wfile\_jumps libio\_vtable =
{
JUMP\_INIT\_DUMMY,
JUMP\_INIT(finish, \_IO\_new\_file\_finish),
JUMP\_INIT(overflow, (\_IO\_overflow\_t) \_IO\_wfile\_overflow),
JUMP\_INIT(underflow, (\_IO\_underflow\_t) \_IO\_wfile\_underflow),
JUMP\_INIT(uflow, (\_IO\_underflow\_t) \_IO\_wdefault\_uflow),
JUMP\_INIT(pbackfail, (\_IO\_pbackfail\_t) \_IO\_wdefault\_pbackfail),
JUMP\_INIT(xsputn, \_IO\_wfile\_xsputn),
JUMP\_INIT(xsgetn, \_IO\_file\_xsgetn),
JUMP\_INIT(seekoff, \_IO\_wfile\_seekoff),
JUMP\_INIT(seekpos, \_IO\_default\_seekpos),
JUMP\_INIT(setbuf, \_IO\_new\_file\_setbuf),
JUMP\_INIT(sync, (\_IO\_sync\_t) \_IO\_wfile\_sync),
JUMP\_INIT(doallocate, \_IO\_wfile\_doallocate),
JUMP\_INIT(read, \_IO\_file\_read),
JUMP\_INIT(write, \_IO\_new\_file\_write),
JUMP\_INIT(seek, \_IO\_file\_seek),
JUMP\_INIT(close, \_IO\_file\_close),
JUMP\_INIT(stat, \_IO\_file\_stat),
JUMP\_INIT(showmanyc, \_IO\_default\_showmanyc),
JUMP\_INIT(imbue, \_IO\_default\_imbue)
};
libc\_hidden\_data\_def (\_IO\_wfile\_jumps)
```

其中，带`default`的都是共用的函数，大都在`genops.c`里面；`new\_file`和`file`的大都在`fileops.c`里面；`wdefault`是宽字符共用的函数，大都在`wgenops.c`里面；只有`wfile`的才是自己单独定义的函数，在`wfileops.c`里面。从上面可以看出`wfile`单独定义的操作只有5个。

### `IO\_FILE`结构体

通过源码可以看出，`\_IO\_FILE`结构体经过了很多次的完善。

```c
struct \_IO\_FILE
{
int \_flags; /\* High-order word is \_IO\_MAGIC; rest is flags. \*/
/\* The following pointers correspond to the C++ streambuf protocol. \*/
char \*\_IO\_read\_ptr; /\* Current read pointer \*/
char \*\_IO\_read\_end; /\* End of get area. \*/
char \*\_IO\_read\_base; /\* Start of putback+get area. \*/
char \*\_IO\_write\_base; /\* Start of put area. \*/
char \*\_IO\_write\_ptr; /\* Current put pointer. \*/
char \*\_IO\_write\_end; /\* End of put area. \*/
char \*\_IO\_buf\_base; /\* Start of reserve area. \*/
char \*\_IO\_buf\_end; /\* End of reserve area. \*/
/\* The following fields are used to support backing up and undo. \*/
char \*\_IO\_save\_base; /\* Pointer to start of non-current get area. \*/
char \*\_IO\_backup\_base; /\* Pointer to first valid character of backup area \*/
char \*\_IO\_save\_end; /\* Pointer to end of non-current get area. \*/
struct \_IO\_marker \*\_markers;
struct \_IO\_FILE \*\_chain;
int \_fileno;
int \_flags2;
\_\_off\_t \_old\_offset; /\* This used to be \_offset but it's too small. \*/
/\* 1+column number of pbase(); 0 is unknown. \*/
unsigned short \_cur\_column;
signed char \_vtable\_offset;
char \_shortbuf[1];
\_IO\_lock\_t \*\_lock;
#ifdef \_IO\_USE\_OLD\_IO\_FILE // 可以看出如果使用旧的 \_IO\_FILE ,那我们经常说的IO就是 \_IO\_FILE\_complete
};
struct \_IO\_FILE\_complete
{
struct \_IO\_FILE \_file;
#endif
\_\_off64\_t \_offset;
/\* Wide character stream stuff. \*/
struct \_IO\_codecvt \*\_codecvt;
struct \_IO\_wide\_data \*\_wide\_data;
struct \_IO\_FILE \*\_freeres\_list;
void \*\_freeres\_buf;
size\_t \_\_pad5;
int \_mode;
/\* Make sure we don't get into trouble again. \*/
char \_unused2[15 \* sizeof (int) - 4 \* sizeof (void \*) - sizeof (size\_t)];
};
struct \_IO\_FILE\_complete\_plus
{
struct \_IO\_FILE\_complete file;
const struct \_IO\_jump\_t \*vtable;
};
```

在调试中可以看到全部信息。

```c
pwndbg> ptype stdout
type = struct \_IO\_FILE {
int \_flags;
char \*\_IO\_read\_ptr;
char \*\_IO\_read\_end;
char \*\_IO\_read\_base;
char \*\_IO\_write\_base; # 本质上是通过修改这个结构题泄露
char \*\_IO\_write\_ptr; # 这两个指针地址之间的内容
char \*\_IO\_write\_end; # 输出内容的结尾
char \*\_IO\_buf\_base; # 缓冲区的基地址
char \*\_IO\_buf\_end; # 缓冲区的结束地址
char \*\_IO\_save\_base;
char \*\_IO\_backup\_base;
char \*\_IO\_save\_end;
struct \_IO\_marker \*\_markers;
struct \_IO\_FILE \*\_chain;
int \_fileno;
int \_flags2;
\_\_off\_t \_old\_offset;
unsigned short \_cur\_column;
signed char \_vtable\_offset;
char \_shortbuf[1];
\_IO\_lock\_t \*\_lock;
\_\_off64\_t \_offset;
struct \_IO\_codecvt \*\_codecvt;
struct \_IO\_wide\_data \*\_wide\_data;
struct \_IO\_FILE \*\_freeres\_list;
void \*\_freeres\_buf;
size\_t \_\_pad5;
int \_mode;
char \_unused2[20];
} \*
```

### 全部清空函数（`fflush`）

```c
# define fflush(s) \_IO\_fflush (s) // /assert/assert.c
// /libio/iofflush.c
int \_IO\_fflush (FILE \*fp)
{
if (fp == NULL)
return \_IO\_flush\_all ();
else
{
int result;
CHECK\_FILE (fp, EOF);
\_IO\_acquire\_lock (fp);
result = \_IO\_SYNC (fp) ? EOF : 0;
\_IO\_release\_lock (fp);
return result;
}
}
libc\_hidden\_def (\_IO\_fflush)
```

**可以看出 `fflush`函数在参数为空时，清空(`\_IO\_flush\_all\_lockp => \_IO\_OVERFLOW`)全部文件；不为空时，同步(`sync`)指定文件，两种情况执行步骤不同。**

### FSOP

FSOP执行是靠`\_IO\_flush\_all\_lockp`，该函数的功能是刷新所有FILE结构体的输出缓冲区，执行这个程序的时候会沿着`fp->chain`执行`overflow`程序。

```c
int \_IO\_flush\_all\_lockp (int do\_lock)
{
int result = 0;
struct \_IO\_FILE \*fp;
int last\_stamp;
fp = (\_IO\_FILE \*) \_IO\_list\_all;
while (fp != NULL)
{
...
if (((fp->\_mode <= 0 && fp->\_IO\_write\_ptr > fp->\_IO\_write\_base)
#if defined \_LIBC || defined \_GLIBCPP\_USE\_WCHAR\_T
|| (\_IO\_vtable\_offset (fp) == 0
&& fp->\_mode > 0 && (fp->\_wide\_data->\_IO\_write\_ptr
> fp->\_wide\_data->\_IO\_write\_base))
#endif
)
&& \_IO\_OVERFLOW (fp, EOF) == EOF) // 如果输出缓冲区有数据，刷新输出缓冲区
result = EOF;
fp = fp->\_chain; //遍历链表
}
...
}
```

`\_IO\_flush\_all\_lockp`调用函数的时机包括：

> - 执行`abort`函数时。（2.27之后不再刷新）
> - `\_\_malloc\_assert`（仅刷新 `stderr` ，2.36后不再刷新）
> - 执行`exit`函数时。
> - 从`main`函数返回时。（也是执行`exit`）

首先是`abort`函数的流程，利用的`double free`漏洞触发，栈回溯为：

```shell
\_IO\_flush\_all\_lockp (do\_lock=do\_lock@entry=0x0)
\_\_GI\_abort ()
\_\_libc\_message (do\_abort=do\_abort@entry=0x2, fmt=fmt@entry=0x7ffff7ba0d58 "\*\*\* Error in `%s': %s: 0x%s \*\*\*\n")
malloc\_printerr (action=0x3, str=0x7ffff7ba0e90 "double free or corruption (top)", ptr=<optimized out>, ar\_ptr=<optimized out>)
\_int\_free (av=0x7ffff7dd4b20 <main\_arena>, p=<optimized out>,have\_lock=0x0)
main ()
\_\_libc\_start\_main (main=0x400566 <main>, argc=0x1, argv=0x7fffffffe578, init=<optimized out>, fini=<optimized out>, rtld\_fini=<optimized out>, stack\_end=0x7fffffffe568)
\_start ()
```

exit 函数，栈回溯为：

```shell
\_IO\_flush\_all\_lockp (do\_lock=do\_lock@entry=0x0)
\_IO\_cleanup ()
\_\_run\_exit\_handlers (status=0x0, listp=<optimized out>, run\_list\_atexit=run\_list\_atexit@entry=0x1)
\_\_GI\_exit (status=<optimized out>)
main ()
\_\_libc\_start\_main (main=0x400566 <main>, argc=0x1, argv=0x7fffffffe578, init=<optimized out>, fini=<optimized out>, rtld\_fini=<optimized out>, stack\_end=0x7fffffffe568)
\_start ()
```

程序正常退出，栈回溯为：

```shell
\_IO\_flush\_all\_lockp (do\_lock=do\_lock@entry=0x0)
\_IO\_cleanup ()
\_\_run\_exit\_handlers (status=0x0, listp=<optimized out>, run\_list\_atexit=run\_list\_atexit@entry=0x1)
\_\_GI\_exit (status=<optimized out>)
\_\_libc\_start\_main (main=0x400526 <main>, argc=0x1, argv=0x7fffffffe578, init=<optimized out>, fini=<optimized out>, rtld\_fini=<optimized out>, stack\_end=0x7fffffffe568)
\_start ()
```

## 21.猜想

从上面可以看出，很多函数并没有用，但为什么还要设置这些呢？以下是我的猜想。

> 1. `glibc`的文件操作经历了很多版本迭代，为了兼容之前的版本，保留了很多没有用的操作。
> 2. 跳表种类很多，我们目前看到的是`file`操作，还有字符操作（str）、宽字符（wdate）操作、帮助文档操作（help）等等，有一些操作是独有的，类似于`\_IO\_new\_file\_xsputn`。有一些是通用的操作，类似于`\_IO\_default\_uflow`。
> 3. 现将框架搭起来，如果以后有需要的时候可以方便进行扩展。

对这些清楚了之后，我们就可以看看其他的`house`到底是干什么的了。

# 四、虚表检测

虚表检测是2.24之后加入的内容，`IO\_validate\_vtable`检测如果虚表超出范围就进入`\_IO\_vtable\_check`函数。各路大神找到的`house`很多都不是打`file`的跳表，而是其他处理跳表，但都差不太多。简要梳理如下。

> 1. 2.23 的没有任何限制，可以将`vtable` 劫持在堆上并修改其内容，然后触发FSOP，
> 2. 2.24 引入了`vtable check`，使得将`vtable` 整体劫持到堆上已不可能，大佬发现可以使用内部的`vtable`中`\_IO\_str\_jumps`或`\_IO\_wstr\_jumps`来进行利用。
> 3. 2.31 中将`\_IO\_str\_finish`函数中强制执行`free`函数，导致无法使用上述问题，因而催生出其他调用链。

## 虚表范围

虚表位置判断主要在`IO\_validate\_vtable`函数，**2.37以前**判断区间为`\_IO\_helper\_jumps - \_IO\_str\_jumps`之间的区域 0xd60，里面有以下虚表。
