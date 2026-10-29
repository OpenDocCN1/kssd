
```
_IO_helper_jumps
_IO_helper_jumps
_IO_cookie_jumps
_IO_proc_jumps
_IO_str_chk_jumps
_IO_wstrn_jumps
_IO_wstr_jumps
_IO_wfile_jumps_maybe_mmap
_IO_wfile_jumps_mmap
__GI__IO_wfile_jumps
_IO_wmem_jumps
_IO_mem_jumps
_IO_strn_jumps
_IO_obstack_jumps
_IO_file_jumps_maybe_mmap
_IO_file_jumps_mmap
__GI__IO_file_jumps
_IO_str_jumps
```

## 攻击`_IO_vtable_check`

在`IO_validate_vtable`函数检查如果虚表超出范围，会进入`_IO_vtable_check`函数，

```c
void attribute_hidden _IO_vtable_check (void)
{
#ifdef SHARED
/* Honor the compatibility flag. */
void (*flag) (void) = atomic_load_relaxed (&IO_accept_foreign_vtables);
#ifdef PTR_DEMANGLE
PTR_DEMANGLE (flag);
#endif
if (flag == &_IO_vtable_check) //检查是否是外部重构的vtable
return;
```

这里就很有意思，也就是说GNU其实也同意你能够外部重构`vtable`，只是要满足一定条件。那么我们还是可以绕过虚表检测的。

> 1. 泄露`ptr_guard`，反算`IO_accept_foreign_vtables`然后修改。
> 2. 因为`IO_accept_foreign_vtables`中基本都是0，直接将`ptr_guard`修改为`&_IO_vtable_check`也可以。

但无论如何我们都需要有ld文件。

## 外置虚表

`check_stdfiles_vtables`函数是设置外置虚表的函数，如果能执行这个函数，也可以绕过虚表检测。

```c
static void check_stdfiles_vtables (void)
{
if (_IO_2_1_stdin_.vtable != &_IO_file_jumps
|| _IO_2_1_stdout_.vtable != &_IO_file_jumps
|| _IO_2_1_stderr_.vtable != &_IO_file_jumps)
IO_set_accept_foreign_vtables (&_IO_vtable_check);
}
```

# 五、宽字符跳表

将宽字符函数调用单独拿出来主要是因为，目前（2.36及以前，2.37也没有修订）宽字符跳表的引用没有加入保护，`house_of_apple house_of_cat`都是利用这一点。

以2.36为例，目前，涉及到宽字符跳转的函数一共有19个，也就是说跳表中的都定义了。

```
#define _IO_WFINISH(FP) WJUMP1 (__finish, FP, 0)
#define _IO_WOVERFLOW(FP, CH) WJUMP1 (__overflow, FP, CH)
#define _IO_WUNDERFLOW(FP) WJUMP0 (__underflow, FP)
#define _IO_WUFLOW(FP) WJUMP0 (__uflow, FP)
#define _IO_WPBACKFAIL(FP, CH) WJUMP1 (__pbackfail, FP, CH)
#define _IO_WXSPUTN(FP, DATA, N) WJUMP2 (__xsputn, FP, DATA, N)
#define _IO_WXSGETN(FP, DATA, N) WJUMP2 (__xsgetn, FP, DATA, N)
#define _IO_WSEEKOFF(FP, OFF, DIR, MODE) WJUMP3 (__seekoff, FP, OFF, DIR, MODE)
#define _IO_WSEEKPOS(FP, POS, FLAGS) WJUMP2 (__seekpos, FP, POS, FLAGS)
#define _IO_WSETBUF(FP, BUFFER, LENGTH) WJUMP2 (__setbuf, FP, BUFFER, LENGTH)
#define _IO_WSYNC(FP) WJUMP0 (__sync, FP)
#define _IO_WDOALLOCATE(FP) WJUMP0 (__doallocate, FP)
#define _IO_WSYSREAD(FP, DATA, LEN) WJUMP2 (__read, FP, DATA, LEN)
#define _IO_WSYSWRITE(FP, DATA, LEN) WJUMP2 (__write, FP, DATA, LEN)
#define _IO_WSYSSEEK(FP, OFFSET, MODE) WJUMP2 (__seek, FP, OFFSET, MODE)
#define _IO_WSYSCLOSE(FP) WJUMP0 (__close, FP)
#define _IO_WSYSSTAT(FP, BUF) WJUMP1 (__stat, FP, BUF)
#define _IO_WSHOWMANYC(FP) WJUMP0 (__showmanyc, FP)
#define _IO_WIMBUE(FP, LOCALE) WJUMP1 (__imbue, FP, LOCALE)
```

但实际上有引用的仅为以下4个

```c
#define _IO_WOVERFLOW(FP, CH) WJUMP1 (__overflow, FP, CH)
#define _IO_WUFLOW(FP) WJUMP0 (__uflow, FP)
#define _IO_WSETBUF(FP, BUFFER, LENGTH) WJUMP2 (__setbuf, FP, BUFFER, LENGTH)
#define _IO_WDOALLOCATE(FP) WJUMP0 (__doallocate, FP)
```

其中，`_IO_WSETBUF`仅用在`_IO_setbuffer`中，也就是我们经常用的`setbuf`

|  |  |
| --- | --- |
| 1  2 | `歌舞为谁演、拨珠为谁算，执戟儿郎为谁站，流血为谁干。`  `苏女庆生平，卫商秦问鼎，八千江东子弟兵，明总建大清。` |

昨天，突然有一个师傅突然提醒我GLIBC-2.37已经发布，就重新看了一下，瞬间感觉背后发凉，`largebin`没有什么修改，IO修改很大，针对性强的一逼，以前一些利用方式已无法使用，比较庆幸的是还有有一些新的利用方式可以继续挖掘。而且我认为GNU下一步会按2.37的思路不断修改IO，以前很多方法都会渐渐失效，在此我也决定将几个2.37以后失效的攻击链公布出来。本篇的内容主要是开篇，简单介绍一下IO，老手就可以无视。

# 零、入门

既然是IO操作，那么就有必要知道计算机到底是怎么读写硬盘或者其他设备的，此处主要以块设备为主。以硬盘为例，要想和硬盘交互，说白了也是和硬盘上的芯片+系统交互，在x86的体系下，其实就是和硬盘上的寄存器进行交互，为了让这个过程变的简单，硬盘上的寄存器被映射成了端口，硬盘的端口说明如下表。而能够交互端口的操作就是汇编指令中的`in out`，这些指令是特权指令，三环无法使用，像我们的常说的底层`read`或`write`函数其实也是靠操作系统执行`in，out`指令来实现外部设备读写的。具体的执行细节不是我们的重点，不再细写。需要知道的是，对于LBA硬盘来说，读写数据都必须一块一块的读，这也就是常说的块设备。

| I/O地址 | 读(主机从硬盘读数据) | 写(主机数据写入硬盘) |
| --- | --- | --- |
| 1F0H | 数据寄存器 | 数据寄存器 |
| 1F1H | 错误寄存器(只读寄存器) | 特征寄存器 |
| 1F2H | 扇区计数寄存器 | 扇区计数寄存器 |
| 1F3H | 扇区号寄存器或 LBA 块地址 0~7 | 扇区号或 LBA 块地址 0~7 |
| 1F4H | 磁道数低 8 位或 LBA 块地址 8~15 | 磁道数低 8 位或 LBA 块地址 8~15 |
| 1F5H | 磁道数高 8 位或 LBA 块地址 16~23 | 磁道数高 8 位或 LBA 块地址 16~23 |
| 1F6H | 驱动器/磁头或 LBA 块地址 24~27 | 驱动器/磁头或 LBA 块地址 24~27 |
| 1F7H | 命令寄存器或状态寄存器 | 命令寄存器 |

# 一、IO数据结构

对于LBA硬盘来说，读写数据都必须一块一块的读，如果我们每次执行`read，write`时都是操作很少的数据，则对系统消耗非常大，因此，C库就想了一个好办法——缓冲区。所以，就比较好理解了，缓冲区是为了减少3坏操作外部硬件时的消耗产生的，一切都是以外部硬件为服务对象。

> 1.**从外部硬件读取时。**为了减少消耗，会一次从外部硬件读取一“块”数据，并放入缓冲区，然后当`target`需要时，再从头部慢慢读取，只到读完才再次从硬件读取。这个缓冲区叫输入缓冲区。
> 2.**向外部硬件写入时。**为了减少消耗，不会一有东西就写入，而是先将内容从`source`写入缓冲区，当缓冲区满了时候再将内存一起写入硬件。这个缓冲区叫输出缓冲区。

为了更好的定位，对每个操作我们肯定至少要有3个基础数据。首先，以从外部硬件读取为例，我们要有输入缓冲区开始（base）、结尾（end）和当前（ptr）已经用了多少的指针。很明显当`ptr == end`时，说明输入缓冲区里的东西已经全部读完，需要重新从硬件读入。

同样，对于向外部硬件写入为例，我们要有输出缓冲区开始（base）、结尾（end）和当前（ptr）已经写了多少的指针。很明显当`ptr == end`时，说明输出缓冲区已经写满，可以向硬件写入了。

上面的内容看似非常清楚，但这里其实有一些比较容易混乱的地方。因为缓冲区内存储的是数据，输入、输出两者数据流动方向不同，但保护主体都一样，都是外部设备，所以**有用的数据部分**就有所不相同。

> 1. 对于输入缓冲区`ptr-end`是有用的数据，`base-ptr`为已使用的数据。
> 2. 对于输出缓冲区`base-ptr`是要写入硬件的内容（有用数据），`ptr-end`为空闲区域。
> 3. 两者结尾有所不同。
>    1. 对于输入缓冲区，因为从硬盘中读取的数据可能无法填满整个缓冲区的块，所以`_IO_buf_end != _IO_read_end`。输入缓冲区要使用`_IO_read_end`判断结束。
>    2. 对于输出缓冲区，缓冲区的结束就是输出缓冲区结束，`_IO_buf_end == _IO_write_end`。输出缓冲区往往使用`_IO_buf_end`判断结束。

虽然，输入、输出缓冲区作用不同，但原理上都是一块内存。一块外部设备可能既可以写入也可以读取，为了节省空间，我们可以定义一块缓冲区，需要输入的时候就做输入缓冲区，需要输出就做输出缓冲区。那么我们就有了8个指针。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `char` `*``_IO_buf_base;` `/``/` `缓冲区的基地址`  `char` `*``_IO_buf_end;` `/``/` `缓冲区的结束地址`  `char` `*``_IO_read_base;` `/``/` `输入缓冲区基地址`  `char` `*``_IO_read_ptr;` `/``/` `输入当前位置`  `char` `*``_IO_read_end;` `/``/` `输入缓冲区结尾地址`  `char` `*``_IO_write_base;``/``/` `输出缓冲区基地址`  `char` `*``_IO_write_ptr;` `/``/` `输出当前位置`  `char` `*``_IO_write_end;` `/``/` `输出缓冲区结尾地址` |

那么到现在，基本思路理清了，其他就方便了.

> **从文件中读取** 程序是从`fd`中读取一批数据到缓冲区中（`_IO_buf_base` 至 `_IO_buf_end`），`_IO_read_ptr` 指向已向`target`中写完的位置，既 `_IO_read_ptr` 至 `_IO_read_end` 为还没有写入`target`中的数据。当`_IO_read_ptr == _IO_read_end`时，说明输入缓冲区内已经没有可用数据，需要再次从文件中读入数据。
> **向文件输出** 程序是先将`source`中的数据写入到缓冲区中，`_IO_write_ptr` 指向已从`source`中写到的位置，既 `_IO_write_ptr` 至 `_IO_write_pend` 为还剩余的空间。当`_IO_write_ptr == _IO_buf_end`时，再全部写入`fd`中。

# 二、数据操作

既然有了数据结构，我们就可以简单定义一些操作来进行操作。

## 1.从硬盘中读入数据

这个逻辑前面已经说的非常清楚，简单逻辑如下。

> 1. 从`fd`中读取一批（一块）数据到输入缓冲区中（`_IO_buf_base` 至 `_IO_buf_end`），同时对`_IO_read_base` `_IO_read_ptr` `_IO_read_end` 设置初始值。（`_IO_read_ptr == _IO_read_base` ，当然也可能不同）
> 2. 从`_IO_read_ptr` 处向需要的内存中复制数据，同时把`_IO_read_ptr` 向后移位。
> 3. 当`_IO_read_ptr == _IO_read_end`时，说明缓冲区内已经没有可用数据，需要再次从文件中读入数据。冲入第一步。

## 2.向硬盘中写入数据

同理，操作逻辑如下。

> 1. 先将`source`中的数据复制到输出缓冲区中，`_IO_write_ptr` 指向已写到的位置。
> 2. 当`_IO_write_ptr == _IO_buf_end`时，将缓冲区中的内容全部写入`fd`中，并将`_IO_write_ptr`设置为 `_IO_write_base`，重复第一步。

上面的操作中，我们还忘了一个基本的问题：**缓冲区从哪里来？**其实缓冲区就是一块内存，可以在栈上、堆上、`libc`中，甚至随便`mmap`一块内存都可以，但不论怎么来，我们都需要这样一块区域，在此，我们借用`glibc`中在`malloc`的方法来申请缓冲区。那么我们还需要第三个操作。

## 3.申请缓冲区

这个操作非常简单。

> 申请一块缓冲区，并设置`_IO_buf_base`为开头，`_IO_buf_end`为结尾。

到此为止，IO的所有基本操作就已经算是完成了。当然，操作中还需要一些安全检测，例如判断缓冲区是否存在、`malloc`是否成功等内容，这里就不再赘述。下面，我以`glibc`中`_IO_file_jumps`为例，梳理一下函数操作的意图。

# 三、`_IO_file_jumps` 函数操作

说明顺序根据`_IO_file_jumps`中的操作顺序来，因为里面的互相调用还是挺多的，就不说具体写过程，主要说明操作的意图。

## 1.`_IO_new_file_finish`

这个看名字就非常简单，是文件结束的操作，所以它的操作如下

> 1. 清空所有缓冲区
> 2. 关闭（close）文件

## 2.`_IO_new_file_overflow`

这个函数意图比较简单，主要是处理当输出缓冲区用完时，向硬盘写入数据。当然，其实这个函数内部非常复杂，加入了一些检测。例如，如果缓冲区不存在则要**初始化缓冲区**。并且，这个函数的参数中有一个标志位。

> 1. 如果 `ch == EOF`，则输出`f->_IO_write_ptr - f->_IO_write_base`的区间。
> 2. 如果 `ch != EOF`，并且`f->_IO_write_ptr == f->_IO_buf_end`，则将缓冲区全部输出。
> 3. 如果 `ch == '\n'`，则输出 `f->_IO_write_ptr - f->_IO_write_base`加一个换行符。
> 4. 以上都不满足就返回 ch。

## 3.`_IO_new_file_underflow`

这个函数与`_IO_new_file_overflow`差不多，主要是用于从硬盘中读取数据，每次读取都是`_IO_buf_base 至 _IO_buf_end`。为了防止硬盘中没有这么多数据，设置`_IO_read_end`为读取的总数。如果，缓冲区不存在则要**初始化缓冲区**。程序返回`_IO_read_ptr`指针。

## 4.`__GI__IO_default_uflow`（`_IO_default_uflow`）

这个函数就是调用`_IO_new_file_underflow`，并简单做了一些检测。

## 5.`__GI__IO_default_pbackfail`（`_IO_default_pbackfail`）

设置存储的函数，暂不重要。

## 6.`_IO_new_file_xsputn`

这个函数是主要目的是将数据从`source`放入输出输出缓冲区。显然，放入过程中还有几种情况。

> 1. 如果要写入的数据**小于**剩余的空间`_IO_write_ptr - _IO_buf_end`，那么就直接将数据写入输出缓冲区即可。
