
```cpp
#define \_IO\_blen(fp) ((fp)->\_IO\_buf\_end - (fp)->\_IO\_buf\_base)
int
\_IO\_str\_overflow (FILE \*fp, int c)
{
int flush\_only = c == EOF;
size\_t pos;
if (fp->\_flags & \_IO\_NO\_WRITES)
return flush\_only ? 0 : EOF;
if ((fp->\_flags & \_IO\_TIED\_PUT\_GET) && !(fp->\_flags & \_IO\_CURRENTLY\_PUTTING))
{
fp->\_flags |= \_IO\_CURRENTLY\_PUTTING;
fp->\_IO\_write\_ptr = fp->\_IO\_read\_ptr;
fp->\_IO\_read\_ptr = fp->\_IO\_read\_end;
}
pos = fp->\_IO\_write\_ptr - fp->\_IO\_write\_base;
if (pos >= (size\_t) (\_IO\_blen (fp) + flush\_only))
{
if (fp->\_flags & \_IO\_USER\_BUF) /\* not allowed to enlarge \*/
return EOF;
else
{
char \*new\_buf;
char \*old\_buf = fp->\_IO\_buf\_base; // 覆盖到这里
size\_t old\_blen = \_IO\_blen (fp);
size\_t new\_size = 2 \* old\_blen + 100;
if (new\_size < old\_blen)
return EOF;
new\_buf = malloc (new\_size); // 调用malloc
if (new\_buf == NULL)
{
/\* \_\_ferror(fp) = 1; \*/
return EOF;
}
if (old\_buf)
{
memcpy (new\_buf, old\_buf, old\_blen);// 调用memecpy，覆盖
free (old\_buf); // 调用free
/\* Make sure \_IO\_setb won't try to delete \_IO\_buf\_base. \*/
fp->\_IO\_buf\_base = NULL;
}
memset (new\_buf + old\_blen, '\0', new\_size - old\_blen);
...
}
}
```

利用流程如下：

- 将`\_IO\_buf\_base` 指向 `/bin/sh\x00` 地址。

- 控制`\_IO\_buf\_end-\_IO\_buf\_base` 的值也就是 `new\_size` 的值，进而控制分配的 `chunk` 的大小，分配到布局好的地址。

- 利用 `memcpy` 中覆盖地址，也就是 `malloc` 出来的 `new\_buf`， 可以覆盖`\_\_malloc\_hook/\_\_free\_hook` 等

- 最后调用 `free(old\_buf) -> system("/bin/sh")`，

在 `glibc-2.34` 后 `ptmalloc` 取消了各种 `hook`，但依然可以用 `house of pig` 实现任意地址写任意值，借助其他手段完成权限获取，后面有时间会做补充。

`House of Pig` 是一个将 `Tcache Statsh Unlink Attack` 和 `FSOP` 结合的攻击，同时使用到了 `Largebin Attack` 进行辅助。主要适用于 `libc 2.31` 及以后的新版本 `libc` 并且程序中仅有 `calloc` 时。

利用条件为

- 存在 `UAF`

- 能执行 `abort` 流程或程序显式调用 `exit` 或程序能通过主函数返回。

## <a href="elink@d74K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6t1L8%4u0F1L8%4x3K6i4K6u0r3M7s2N6F1k6X3W2D9k6b7`.`.">POC</a>
