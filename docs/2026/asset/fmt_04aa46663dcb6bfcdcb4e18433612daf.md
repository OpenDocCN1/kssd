
# [原创]pwnable.tw新手向write up(四) hacknote-Use After Free

## 往期

[pwnable.tw新手向write up(一)](https://bbs.pediy.com/thread-259290.htm)

[pwnable.tw新手向write up(二) 3×17-x64静态编译程序的`fini_array`劫持](https://bbs.pediy.com/thread-259298.htm)

[pwnable.tw新手向write up(三) dubblesort-多重保护下的栈溢出](https://bbs.pediy.com/thread-259341.htm)

## 前置知识

如果你没做过堆类型的题目,或者对chunk,fast bin这些内容还不怎么清楚的话,可以先看看这个:《glibc内存管理ptmalloc源代码分析.pdf》,作者是华庭,具体文件我会上传到附件的.

### chunk 的实际大小与申请的内存大小并不相同

我们所有使用`malloc`或者其他方式申请的内存空间,ptmalloc都使用chunk来表示,释放掉的内存也不是直接归还给操作系统,而是依旧用chunk来管理.所以chunk在使用中和释放后是两种不同的结构.首先是使用中:

```
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of chunk, in bytes |A|M|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| User data starts here... .
. .
. (malloc_usable_size() bytes) .
next . |
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| (size of chunk, but used for application data) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

前两个字节为chunk header,第三个字节开始,也就是`mem`指针指向的地方开始,就是我们申请的内存(大小等于我们申请的大小),`malloc`函数返回值就是`mem`指针的值.接着看看被释放的chunk:

```
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`head:' | Size of chunk, in bytes |A|0|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Forward pointer to next chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Back pointer to previous chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Unused space (may be 0 bytes long) .
. .
next . |
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`foot:' | Size of chunk, in bytes |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

user data被替换为了两个指针,`fd`指向前一个空间的chunk,`bk`指向后一个,如果是large bin的话还会多两个指针,不过这里就不提了.

现在我们已经知道了chunk的构造,那么如何通过申请内存的大小来计算chunk的实际大小呢?以32位系统为例,一个free掉的chunk有四个字段,分别是`prev_size`,`size`,`fd`,`bk`,也就是16个字节,所以一个chunk的大小最小为16字节.再看看使用中的chunk,因为是使用状态,所以下一个chunk的`prev_size`就是无效的,所以这个地方也可以被当前chunk使用.

这样就可以得知,chunk的`in_use_size=(用户请求大小+8-4) align 8B`.稍微解释一下,从左到右,用户请求大小就是我们申请的大小,+8是因为需要两个字段来保存chunk header,-4就是因为当前chunk可以使用下一个chunk的`prev_size`,节省了4字节的大小,对齐8B则是ptmalloc的要求.

所以,最终的分配空间大小`chunk_size = max(in_use_size,16)`.下文中及`exp`中用到的大小都是chunk的最终分配大小.

### Fast Bins 简介

glibc采用单向链表对其中的每个 bin 进行组织,并且每个bin采取LIFO策略,也就是和栈一样,最近释放的chunk会被更早的分配.不同大小的chunk也不会被链接在一起,fastbin支持从16字节开始的10个相应大小的bin.

### glibc 中的首次适应（First Fit）算法

空间分区以地址递增的次序链接,分配内存时顺序查找,找到大小能满足要求的第一个空闲分区.

如果分配内存时存在一个大于或等于所需大小的空间chunk,glibc就会选择这个chunk.举个例子:

```
a = malloc(512) = 0x2490010
b = malloc(256) = 0x2490220
free(a)
c = malloc(500) = 0x2490010
```

### Use After Free

当一个内存块在被释放之后,执行他的指针应该置0,不然就会变成危险的悬挂指针,使得这个内存块被再次申请出去之后(比如刚刚说的First Fit),我们还可以对内容进行操作,比如我们将数据进行构造之后,再次利用这个不合法的悬挂指针,就可能造成任意地址读写.

## 具体题目 hacknote

### 看一下防护,只开了NX和Canary

```text
[0] % checksec hacknote
[*] '/home/dylan/ctfs/pwnable_tw/hacknote/hacknote'
Arch: i386-32-little
RELRO: Partial RELRO
Stack: Canary found
NX: NX enabled
PIE: No PIE
```

### IDA分析一下程序结构,比较短小.

`main()`函数没什么好说的,常规的堆题目,打印一个菜单,分别提供创建,删除和打印笔记的功能,先进入`add_func()`查看一下:

```c
void __cdecl __noreturn main()
{
int choice_int; // eax
char choice_char; // [esp+8h] [ebp-10h]
unsigned int canary; // [esp+Ch] [ebp-Ch]
canary = __readgsdword(0x14u);
setvbuf(stdout, 0, 2, 0);
setvbuf(stdin, 0, 2, 0);
while ( 1 )
{
while ( 1 )
{
ui_func();
read(0, &choice_char, 4u);
choice_int = atoi(&choice_char);
if ( choice_int != 2 )
break;
delete_func();
}
if ( choice_int > 2 )
{
if ( choice_int == 3 )
{
show_func();
}
else
{
if ( choice_int == 4 )
exit(0);
LABEL_13:
puts("Invalid choice");
}
}
else
{
if ( choice_int != 1 )
goto LABEL_13;
add_func();
}
}
}
```

`chunk_count`限制使用`add_func()`的次数.申请`note`的时候,程序先申请8字节内存,并且把地址保存在`chunk_ptr[]`中.这8个字节用来保存note结构,前四个字节保存一个函数指针,姑且命名为`puts_func()`,这个函数将传入指针的下一个偏移地址的内容打印出来

```c
int __cdecl puts_func(int *a1)
{
return puts((const char *)a1[1]);
}
```

后四个字保存一个指针,指向大小为`size`的内存,用来保存note的`content`.这样可能不是很直观,我用C语言展示一下note的结构:

```c
struct manage_note
{
int *puts_func_ptr;
int *content_ptr;
};
```

`puts_func_ptr`指向`puts_func()`这个函数,`content_ptr`则指向note的`content`,用来保存我们的输入.如果还是不直观的话,可以看看图:

```
note
+-----------------+ +----------------+
| *puts_func_ptr |------------------->| puts_func |
+-----------------+ +----------------+
| *content_ptr |------------------->+----------------+
+-----------------+ | real |
| content |
+----------------+
```

这样应该很清晰了,假设我们创建一个大小为10字节的note,程序会先申请八个字节来保存note结构,接着再申请0x10字节作为`real_content`保存我们的输入.

```c
unsigned int delete_func()
{
int index; // [esp+4h] [ebp-14h]
char buf; // [esp+8h] [ebp-10h]
unsigned int canary; // [esp+Ch] [ebp-Ch]
canary = __readgsdword(0x14u);
printf("Index :");
read(0, &buf, 4u);
index = atoi(&buf);
if ( index < 0 || index >= chunk_count )
{
puts("Out of bound!");
_exit(0);
}
if ( chunk_ptr[index] ) // UAF
{
free(*((void **)chunk_ptr[index] + 1));
free(chunk_ptr[index]);
puts("Success");
}
return __readgsdword(0x14u) ^ canary;
}
```

`delete_func()`函数先读取一个下标`index`,然后对相应的note进行`free()`.这两个`free()`比较晕,第一个`free()`释放我们的`real_content`,第二个`free()`释放note结构,但是释放之后并没有对指针进行置0,造成了两个悬挂指针,我们可以对释放后的指针继续进行操作,所以这个地方造成了UAF.

```c
unsigned int show_func()
{
int index; // [esp+4h] [ebp-14h]
char buf; // [esp+8h] [ebp-10h]
unsigned int canary; // [esp+Ch] [ebp-Ch]
canary = __readgsdword(0x14u);
printf("Index :");
read(0, &buf, 4u);
index = atoi(&buf);
if ( index < 0 || index >= chunk_count )
{
puts("Out of bound!");
_exit(0);
}
if ( chunk_ptr[index] )
(*(void (__cdecl **)(void *))chunk_ptr[index])(chunk_ptr[index]);
return __readgsdword(0x14u) ^ canary;
}
```

`show_func()`函数先读取一个下标`index`,接着调用`*chunk_ptr[index]`处的函数,也就是我们刚刚说到的`puts_func()`,参数则是`chunk_ptr[index]`,又因为`puts_func()`实际打印的是传入参数的下一个地址,也就是`note->content`的内容,那么就会打印出我们输入的`content`.

### 利用思路

唯一的漏洞是`delete_func()`函数里面存在UAF,哪怕一个`note`已经被我们删除,我们依然可以调用`show_func()`函数来调用`puts_func_ptr`指向的函数,如果我们可以修改`puts_func_ptr`的值,那么我们就可以劫持程序的控制流了.具体如何修改我写到`exp`里边了,这里你先假设你已经可以修改了.

但是这个程序并没有后门函数,并且还给了`libc.so`文件,所以我们需要先泄露`libc`加载的地址,然后计算得到`system()`函数和`'/bin/sh'`的地址.因为我们现在已经可以修改note中的两个指针,所以只要将`content_ptr`修改为`got['puts']`的地址就可以泄露`libc`基地址了.

通过`libc`基地址可以计算出`system()`函数的地址,然后继续修改一个note的`puts_func_ptr`就可以调用`system()`函数了.这里对于`system()`函数的参数设置还有一个坑,回顾一下`show_func()`函数的内容:

```c
if ( chunk_ptr[index] )
(*chunk_ptr[index])(chunk_ptr[index]);
```

此时`chunk_ptr[index]`保存的是note指针,`chunk_ptr[index]`也就是`*note->puts_func_ptr`,这会调用`system()`函数,和我们预想的一样,但是参数同样是note指针,`puts_func()`自身进行了四个字节的偏移,所以才能顺利打印出`content`的内容,`system()`函数并不会进行偏移,自然也就无法执行`system('/bin/sh')`.

我们可以考虑构造note指针本身,它本身作为参数,前四个字节无法被系统识别,只要在后四个字节加入一个`';'`,就可以自由执行命令了,举个例子:
