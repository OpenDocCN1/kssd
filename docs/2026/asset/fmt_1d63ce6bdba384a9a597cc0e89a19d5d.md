
[![2fQiuT.png](upload/attach/202106/865862\_2KSC6SVBWXUKSGS.jpg)](<mark class="encrypted">565K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6W2q4A6N6g2c8Q4x3U0V1`.</mark>

`bck->fd=bin`，即`&victim-0x10+0x10=&victim`被设置为了`bin`头，解链后结构如下：

[![2f1K0O.png](upload/attach/202106/865862\_QTRRQA553SVJC6R.jpg)](<mark class="encrypted">eabK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0q4w2x3p5!0Q4x3U0V1`.</mark>

之后再将`tc_victim`放入`tcache`。

### 放入前

[![2f1hEF.png](upload/attach/202106/865862\_4R8RFEVQNJADQ4V.jpg)](<mark class="encrypted">4bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0q4Z5c8f1k6Q4x3U0V1`.</mark>

### 放入后

[![2f1oC9.png](upload/attach/202106/865862\_PXH7R8Z2QEGZWWF.jpg)](<mark class="encrypted">fdeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0q4G2b7K6W2Q4x3U0V1`.</mark>

看到这里相信大家就大概能明白为什么这个攻击手法可以任意地址分配一个`chunk`了。此时`tcache`中还剩一个空位，程序会继续从`smallbin`中取`chunk`放入`tcache`，此时`smallbin`中只剩下`victim`，我们继续调试：

[![2f8Yef.png](upload/attach/202106/865862\_4DPDQBTP2AZKJN8.jpg)](<mark class="encrypted">164K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0S2k6k6h3k6Q4x3U0V1`.</mark>

新一轮的`tc_victim`为`&victim-0x10`。

接下来会找到倒数第二个`chunk`：

[![2f8bTO.png](upload/attach/202106/865862\_Y245X9SNWPX4DPN.jpg)](<mark class="encrypted">6feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0S2T1g2p5!0Q4x3U0V1`.</mark>

因此我们需要将`&victim-0x10+0x18`处设置为一个可写地址`x`，方便后面往其中写`bin`头地址，而`demo`一开始就将其设置好了。

[![2fGoCQ.png](upload/attach/202106/865862\_YE3FHC769GP8FTE.jpg)](<mark class="encrypted">c45K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6V1N6G2b7#2q4Q4x3U0V1`.</mark>

```c
victim[1] = (uint64_t)(&victim);
```

后续的就是继续解链，然后将`victim`加入到`tcache`。

[![2fJiK1.png](upload/attach/202106/865862\_U8JPKBHYGWPF5KD.jpg)](<mark class="encrypted">21cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6V1A6A6d9K6q4Q4x3U0V1`.</mark>

至此攻击完成。

## 0x3. 高版本下的 `_IO_FILE` 利用

在`2.23`下一般攻击`FILE`结构体就是劫持`IO`函数的`_chain`字段为我们伪造的`IO_FILE_plus`，然后修改`vtable`表中的`io_str_overflow`为`system`。在高版本`libc`下，如`libc2.31`下也依然是利用`io_str_overflow`这个函数，但`io_str_overflow`函数的实现发生了变化：

```c
int
_IO_str_overflow (FILE *fp, int c)
{
int flush_only = c == EOF;
size_t pos;
if (fp->_flags & _IO_NO_WRITES)
return flush_only ? 0 : EOF;
if ((fp->_flags & _IO_TIED_PUT_GET) && !(fp->_flags & _IO_CURRENTLY_PUTTING))
{
fp->_flags |= _IO_CURRENTLY_PUTTING;
fp->_IO_write_ptr = fp->_IO_read_ptr;
fp->_IO_read_ptr = fp->_IO_read_end;
}
pos = fp->_IO_write_ptr - fp->_IO_write_base;
if (pos >= (size_t) (_IO_blen (fp) + flush_only))
{
if (fp->_flags & _IO_USER_BUF) /* not allowed to enlarge */
return EOF;
else
{
char *new_buf;
char *old_buf = fp->_IO_buf_base;
size_t old_blen = _IO_blen (fp);
size_t new_size = 2 * old_blen + 100;
if (new_size < old_blen)
return EOF;
new_buf = malloc (new_size);
if (new_buf == NULL)
{
/* __ferror(fp) = 1; */
return EOF;
}
if (old_buf)
{
memcpy (new_buf, old_buf, old_blen);
free (old_buf);
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

可以看到`io_str_overflow`调用了`malloc`、`memcpy`、`free`等函数。

我们回溯一下`malloc`的参数`new_size`的来源：

```c
size_t new_size = 2 * old_blen + 100;
size_t old_blen = _IO_blen (fp);
#define _IO_blen(fp) ((fp)->_IO_buf_end - (fp)->_IO_buf_base)
```

如果我们能控制`(fp)->_IO_buf_end - (fp)->_IO_buf_base`，就能够控制`malloc`申请的`chunk`大小，再看到后面这一段：

```c
if (old_buf)//char *old_buf = fp->_IO_buf_base;
{
memcpy (new_buf, old_buf, old_blen);
free (old_buf);
/* Make sure _IO_setb won't try to delete _IO_buf_base. */
fp->_IO_buf_base = NULL;
}
```

如果`old_buf`指向的内存空间有数据，则将使用`memcpy`将`old_buf`中的数据拷贝到`new_buf`中，拷贝的长度为`old_blen`，然后再`free (old_buf)`。

再看到`_IO_str_overflow`的汇编：

[![2fBjyD.png](upload/attach/202106/865862\_QF63SXWBAAUK9V2.jpg)](<mark class="encrypted">980K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6V1u0B7P5f1c8Q4x3U0V1`.</mark>

在`+53`这一行，会将`[rdi+0x28]`处的值送到`rdx`中，而自从`libc2.29`开始，`setcontext`中的`gadget`索引由`rdi`变为了`rdx`，需要先控制`rdx`的值才能够进行后续的`srop`。

而`_IO_str_overflow`中的这条汇编正好可以将`rdx`进行赋值，且来源还是`rdi`。`rdi`正好是`FILE`结构体的首地址，只需要将`fp+0x28`设置为我们可以控制的地址就可以进行`srop`，且这条语句是在`malloc`之前执行的，所以利用方法就是：首先将`malloc_hook`设置为`setcontext+61`，然后触发`_IO_str_overflow`，事先在我们伪造的`FILE`结构体中设置好相应的数据，从而将`rdx`赋值为我们可以控制的地址，接着`_IO_str_overflow`调用`malloc`触发`setcontext`，进行`srop`。

触发`malloc`的条件如下：

```c
fp->_flags=0;//if (fp->_flags & _IO_USER_BUF) return EOF;/* not allowed to enlarge */
fp->_IO_write_ptr=srop_addr
/*
0x0 _flags
0x8 _IO_read_ptr
0x10 _IO_read_end
0x18 _IO_read_base
0x20 _IO_write_base
0x28 _IO_write_ptr
0x00007ffff7e2f0dd <+61>: mov rsp,QWORD PTR [rdx+0xa0]
*/
new_buf = malloc (2 * ((fp)->_IO_buf_end - (fp)->_IO_buf_base) + 100);
memcpy (new_buf, fp->_IO_buf_base, (fp)->_IO_buf_end - (fp)->_IO_buf_base);
free (fp->_IO_buf_base);
```

## 0x4. House of Pig

`libc`版本为`2.31`，用`C++`写的。

### 0x1. 修复 switch table

`IDA`反编译，查看`main`函数：

[![2fcGdg.png](upload/attach/202106/865862\_CEX5YZE25DHJ4HG.jpg)](<mark class="encrypted">e82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6X3y4s2k6r3N6Q4x3U0V1`.</mark>

看到一条`__asm { jmp rax }`，这是因为`IDA`没能正确识别`switch`跳转，修复一下，看到汇编：

[![2fWMtg.png](upload/attach/202106/865862\_MKZYS2RBWJJZ7WU.jpg)](<mark class="encrypted">c58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6W2N6y4N6r3N6Q4x3U0V1`.</mark>

`jmp rax`的地址在`0x3794`，跳转表在`0x69e0`。

[![2fWtBV.png](upload/attach/202106/865862\_7HWKWYJF9WMQF3F.jpg)](<mark class="encrypted">b63K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6W2N6@1b7W2k6Q4x3U0V1`.</mark>

跳转表是一字节一字节的，连续按`d`修改成四字节一组，一共有`6`个跳转。

[![2fW6nx.png](upload/attach/202106/865862\_BE8T2PERB869HJA.jpg)](<mark class="encrypted">54fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6W2M7$3L8Y4S2Q4x3U0V1`.</mark>

接下来开始修复跳转表，首先选中`jmp rax`那条汇编，再按如下路径选取`specify switch idiom`：

[![2f7GwR.png](upload/attach/202106/865862\_XBDD7KUBBQ2CU6R.jpg)](<mark class="encrypted">d24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6U0N6s2N6#2u0Q4x3U0V1`.</mark>

配置如下：

[![2fLdUA.png](upload/attach/202106/865862\_WXXRZUX65CP5HNR.jpg)](<mark class="encrypted">489K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6V1I4V1g2f1q4Q4x3U0V1`.</mark>

由上至下分别为：

跳转表的地址；跳转的数量；每一个跳转值的长度；基值；跳转开始的位置；跳转使用的寄存器；`/`；默认跳转位置。

修复成功后如下所示：

[![2fXMSx.png](upload/attach/202106/865862\_R9MZKVER7TSPM92.jpg)](<mark class="encrypted">7b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1j5J5k6W2S2y4f1%4S2Q4x3U0V1`.</mark>

舒服了许多。

### 0x2. 恢复结构体

开始分析程序。

随便点开程序开头的一个函数：

[![RnZHmT.png](upload/attach/202106/865862\_QS6UZQSH5QU4WE3.jpg)](<mark class="encrypted">22eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6d9L8W2A6t1L8g2c8Q4x3U0V1`.</mark>

一串赋值`0`的操作，不明白什么意思，继续往下。

找到菜单。
