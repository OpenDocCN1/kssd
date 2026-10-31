
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
&& \_IO\_OVERFLOW (fp, EOF) == EOF)
result = EOF;
fp = fp->\_chain; //遍历链表
}
[...]
}
```

可以看到，在经历一系列条件判断后，程序会调用虚表中的`\_IO\_OVERFLOW` 函数，而虚表指针是可以由我们伪造的。所以合理布局，触发该函数，就能进入我们伪造的执行流。

下面讲如何达成调用`\_IO\_OVERFLOW`函数的条件：

```c
if (((fp->\_mode <= 0 && fp->\_IO\_write\_ptr > fp->\_IO\_write\_base)
#if defined \_LIBC || defined \_GLIBCPP\_USE\_WCHAR\_T
|| (\_IO\_vtable\_offset (fp) == 0
&& fp->\_mode > 0 && (fp->\_wide\_data->\_IO\_write\_ptr
> fp->\_wide\_data->\_IO\_write\_base))
#endif
)
&& \_IO\_OVERFLOW (fp, EOF) == EOF)
```

根据短路原则，执行`\_IO\_OVERFLOW`需要满足前面的条件为真，这里有一个或语句，故有两种达成条件，满足任一即可：

1. ```c
fp->\_mode = 0
fp->\_IO\_write\_ptr > fp->\_IO\_write\_base
```

这是较为常用的达成条件，只需伪造`\_IO\_FILE`中的`\_IO\_write\_ptr`、`\_IO\_write\_base`和`\_mode`即可达成

2. ```c
\_IO\_vtable\_offset (fp) == 0
fp->\_mode > 0
fp->\_wide\_data->\_IO\_write\_ptr > fp->\_wide\_data->\_IO\_write\_base
```

该条件可以结合**house of cat**使用。其中`\_IO\_vtable\_offset (fp) == 0` 即 `\_vtable\_offset`为0，另外需要关注`\_wide\_data`结构体，该手法会在house of cat中讲解

条件满足后，会从`\_IO\_list\_all`开始沿着`fp -> chain`遍历`FILE`结构体，执行对应虚表中的`\_IO\_OVERFLOW`，即`<\_IO\_file\_jumps+24>`

## house of kiwi - \_\_malloc\_assert

该手法用于在程序无`exit`时触发IO，且能控制`rdx`

以下为`\_\_malloc\_assert`的代码片段

```c
static void
\_\_malloc\_assert (const char \*assertion, const char \*file, unsigned int line,
const char \*function)
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

该函数调用的`\_\_fxprintf`、`fflush`都能调用虚表中函数，有劫持的可能。

以下是进入`\_\_malloc\_assert`的方法

```c
assert ((old\_top == initial\_top (av) && old\_size == 0) ||
((unsigned long) (old\_size) >= MINSIZE &&
prev\_inuse (old\_top) &&
((unsigned long) old\_end & (pagesize - 1)) == 0));
```

不满足以下条件任一即可进入：

1. `old\_size >=MINSIZE`

2. `old\_top.pre\_inuse = 0`

3. `old\_top页对齐`

通常用的比较多的是第三种，因为页对齐条件较苛刻，通常修改topchunk的size都能不对齐。注意**不能改的太大**，否则会进入别的assert。这里修改的方式一般是构造**堆重叠**或**largebin attack**

接下来讲函数的调用链，这里分`\_\_fxprintf`和`fflush`两种方式

### \_\_fxprintf

调用链为

`\_\_fxprintf -> \_\_vfxprintf -> locked\_vfxprintf -> \_\_vfprintf\_internal -> \_IO\_new\_file\_xsputn `

需要满足的条件为：

1. `\_IO\_write\_ptr` > `\_IO\_write\_base`

2. `\_lock`指向可写地址

调用后，仅刷新`stderr`，调用虚表`\_IO\_file\_jumps`中的`\_IO\_new\_file\_xsputn`，即`<\_IO\_file\_jumps+56>`

### fflush

`fflush(stderr) => \_IO\_file\_jumps => sync`

用的比较少，没什么条件。但是因为在`\_\_fxprinf`之后，所以需要保证`\_\_fxprinf`不会挂掉。

好处是通过这条调用链，在调用`sync`时的`rdx`固定，都是`IO\_helper\_jumps`，可以结合后面讲到的`<setcontext+61>`利用。不过高版本中加入虚表检测，这种利用变得困难

## 伪造`FILE`结构体

如何让我们伪造的`FILE`结构体在触发IO时被刷新？

如果使用**FSOP**的方式，程序从`\_IO\_list\_all`开始沿着`fp -> chain`遍历。故我们可以修改`\_IO\_list\_all`指针，指向我们伪造的结构体，如果使用largebin attack就可以一步到位指向布置在chunk上的伪造结构体。同时还可以修改`\_chain`指针，劫持到多个伪造的结构体的利用链

如果使用**house of kiwi**的方式，程序仅刷新`stderr`，可以修改`stderr`指针（如果可写），甚至直接修改`\_IO\_2\_1\_stderr`的内容

伪造`FILE`的手法繁多，这里举出最原始的一种手法作为例子

### 上古时期的FSOP(Glibc<2.24)

因为触发FSOP时，会调用虚表中`<\_IO\_file\_jumps+24>`处的`\_\_overflow`函数，我们只需把该位置填入`system`的地址。当然，虚表不可写，我们可以将`vtable`处改为我们伪造的虚表。同时此时的`rdi`指向`FILE`结构体的`\_flags`，所以可以将该位置填上`/bin/sh`

构造方式如下：

```c
.\_chain => chunk\_addr1
chunk\_addr1
{
file = {
\_flags = "/bin/sh\x00", //对应此结构体首地址(fp)
\_IO\_read\_ptr = 0x0,
\_IO\_read\_end = 0x0,
\_IO\_read\_base = 0x0,
\_IO\_write\_base = 0x0,
\_IO\_write\_ptr = 0x1,
...
\_mode = 0x0, //一般不用特意设置
\_unused2 = '\000' <repeats 19 times>
},
vtable = chunk\_addr2
}
chunk\_addr2
{
\_\_dummy = 0x0,
\_\_dummy2 = 0x0,
\_\_finish = 0x0,
\_\_overflow = system\_addr,
...
}
```

干净又卫生

### 虚表偏移时代(Glibc>=2.24)

加入了对虚表的检查`IO\_validate\_vtable()`与`IO\_vtable\_check()`，若无法通过检查，则会报错：`Fatal error: glibc detected an invalid stdio handle`

```c
static inline const struct \_IO\_jump\_t \* IO\_validate\_vtable (const struct \_IO\_jump\_t \*vtable)
{
uintptr\_t section\_length = \_\_stop\_\_\_libc\_IO\_vtables - \_\_start\_\_\_libc\_IO\_vtables;
const char \*ptr = (const char \*) vtable;
uintptr\_t offset = ptr - \_\_start\_\_\_libc\_IO\_vtables;
if (\_\_glibc\_unlikely (offset >= section\_length)) //检查vtable指针是否在glibc的vtable段中。
\_IO\_vtable\_check ();
return vtable;
}
```

glibc中有一段完整的内存存放着各个`vtable`，其中`\_\_start\_\_\_libc\_IO\_vtables`指向第一个`vtable`地址`\_IO\_helper\_jumps`，而`\_\_stop\_\_\_libc\_IO\_vtables`指向最后一个`vtable` `\_IO\_str\_chk\_jumps`结束的地址。若指针不在glibc的`vtable`段，会调用`\_IO\_vtable\_check()`做进一步检查，以判断程序是否使用了外部合法的`vtable`（重构或是动态链接库中的`vtable`），如果不是则报错。

2.37以前有以下虚表

```c
\_IO\_helper\_jumps
\_IO\_helper\_jumps
\_IO\_cookie\_jumps
\_IO\_proc\_jumps
\_IO\_str\_chk\_jumps
\_IO\_wstrn\_jumps
\_IO\_wstr\_jumps
\_IO\_wfile\_jumps\_maybe\_mmap
\_IO\_wfile\_jumps\_mmap
\_\_GI\_\_IO\_wfile\_jumps
\_IO\_wmem\_jumps
\_IO\_mem\_jumps
\_IO\_strn\_jumps
\_IO\_obstack\_jumps
\_IO\_file\_jumps\_maybe\_mmap
\_IO\_file\_jumps\_mmap
\_\_GI\_\_IO\_file\_jumps
\_IO\_str\_jumps
```

从此，虚表指针不能随意伪造了。好在这个检测依旧比较宽泛，我们依旧可以修改`vtable`为虚表内的其他指针，通过一定的**偏移**调用**其他虚表**内的函数

例如，原本house of kiwi会触发`<\_IO\_file\_jumps+56>`处的`\_IO\_new\_file\_xsputn`函数，我们希望调用`<\_IO\_cookie\_jumps+120>`处的`\_IO\_cookie\_write`函数。原本`vtable`指向`\_IO\_file\_jumps`，调用函数时则会将指针加上56。我们把`vtable`设置为`<\_IO\_cookie\_jumps+64>`，那么在实际调用时，就会调用到`<\_IO\_cookie\_jumps+64+56>`处的目标函数。这就是**虚表偏移**的思想，将执行流劫持到已有的函数上。

# 新时代的高版本IOFILE利用

高版本下的IOFILE利用方式很多，但大多不外乎都是利用**虚表偏移**的思想伪造`FILE`结构体，并通过以上的方式**触发IO**，从而getshell或者orw

这里介绍几种~~我会的~~比较好用的

## house of kiwi

相较于他的实战意义，kiwi更大的价值可能在于它的两条思路，一条是上文提到的`\_\_malloc\_assert`触发IO，另一条则是`setcontext<+61>`

这是一个函数的片段，兼具**布置寄存器、栈迁移**的功能，常常在沙盒限制`execve`的情况下用来打**orw链**，汇编代码如下

```assembly
0x00007fa70ef04a6d <+61>: mov rsp,QWORD PTR [rdx+0xa0]
0x00007fa70ef04a74 <+68>: mov rbx,QWORD PTR [rdx+0x80]
0x00007fa70ef04a7b <+75>: mov rbp,QWORD PTR [rdx+0x78]
0x00007fa70ef04a7f <+79>: mov r12,QWORD PTR [rdx+0x48]
0x00007fa70ef04a83 <+83>: mov r13,QWORD PTR [rdx+0x50]
0x00007fa70ef04a87 <+87>: mov r14,QWORD PTR [rdx+0x58]
0x00007fa70ef04a8b <+91>: mov r15,QWORD PTR [rdx+0x60]
0x00007fa70ef04a8f <+95>: test DWORD PTR fs:0x48,0x2
0x00007fa70ef04a9b <+107>: je 0x7fa70ef04b56 <setcontext+294>#基本上都能跳
--->
0x00007fa70ef04b56 <+294>: mov rcx,QWORD PTR [rdx+0xa8]
0x00007fa70ef04b5d <+301>: push rcx
0x00007fa70ef04b5e <+302>: mov rsi,QWORD PTR [rdx+0x70]
0x00007fa70ef04b62 <+306>: mov rdi,QWORD PTR [rdx+0x68]
0x00007fa70ef04b66 <+310>: mov rcx,QWORD PTR [rdx+0x98]
0x00007fa70ef04b6d <+317>: mov r8,QWORD PTR [rdx+0x28]
0x00007fa70ef04b71 <+321>: mov r9,QWORD PTR [rdx+0x30]
0x00007fa70ef04b75 <+325>: mov rdx,QWORD PTR [rdx+0x88]
0x00007fa70ef04b7c <+332>: xor eax,eax
0x00007fa70ef04b7e <+334>: ret
```

围绕`rdx`进行参数的布置，就能设置好各个寄存器

这里最重要的两条是

`[rdx+0xa0]` => `rsp`

`[rdx+0xa8]` => `rcx` => `ret`

我们通常将`rcx`布置为`ret`的gadget，这样在最后`ret`的时候就能实现栈迁移到赋给`rsp`的地址上

## house of emma

可以理解为kiwi衍生出的攻击手法，需要以下三个条件

1. 可以**任意写**一个可控地址（LargeBin Attack、Tcache Stashing Unlink Attack...）

2. 一次**任意地址读** 或 再来一次**任意写**

3. 可以**触发 IO 流**（FSOP、house of kiwi）

利用的虚表为`\_IO\_cookie\_jumps`，存在拓展结构`\_IO\_cookie\_file`（还是`\_IO\_FILE\_plus`加长版），多了一个函数表，存在劫持的可能

```c
/\* Special file type for fopencookie function. \*/
struct \_IO\_cookie\_file
{
struct \_IO\_FILE\_plus \_\_fp;
void \*\_\_cookie;
cookie\_io\_functions\_t \_\_io\_functions;
};
typedef struct \_IO\_cookie\_io\_functions\_t
{
cookie\_read\_function\_t \*read; /\* Read bytes. \*/
cookie\_write\_function\_t \*write; /\* Write bytes. \*/
cookie\_seek\_function\_t \*seek; /\* Seek/tell file position. \*/
cookie\_close\_function\_t \*close; /\* Close file. \*/
} cookie\_io\_functions\_t;
```
