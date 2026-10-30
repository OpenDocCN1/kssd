
# [原创]BCTF2018-houseofatum-Writeup题解

先把`ld`和`Libc`给换成题目给的

```
patchelf --set-interpreter ./glibc-all-in-one/libs/2.26-0ubuntu2\_amd64/ld-2.26.so --replace-needed ./glibc-all-in-one/libs/2.27-3ubuntu1\_amd64/libc.so.6 ./glibc-all-in-one/libs/2.26-0ubuntu2\_amd64/libc-2.26.so houseofAtum
```

## 程序分析

这里只进行一些简单的分析，其他的博客分析的很详细了

```
bigeast@ubuntu:~/Desktop/ctf$ ./houseofAtum
1. new
2. edit
3. delete
4. show
```

```
int \_\_cdecl \_\_noreturn main(int argc, const char \*\*argv, const char \*\*envp)
{
int v3; // eax
initialize(argc, argv, envp);
while ( 1 )
{
while ( 1 )
{
while ( 1 )
{
v3 = menu();
if ( v3 != 2 )
break;
edit();
}
if ( v3 > 2 )
break;
if ( v3 != 1 )
goto LABEL\_13;
alloc();
}
if ( v3 == 3 )
{
del();
}
else
{
if ( v3 != 4 )
LABEL\_13:
exit(0);
show();
}
}
}
```

```
int alloc()
{
int i; // [rsp+Ch] [rbp-4h]
for ( i = 0; i <= 1 && notes[i]; ++i )
;
if ( i == 2 )
return puts("Too many notes!");
printf("Input the content:");
notes[i] = malloc(0x48uLL);
readn(notes[i], 72LL);
return puts("Done!");
}
```

这里`ull`表示无符号长整形，`ll`表示长整型，就是8字节。

这里`72=0x48`,`72LL`表示用8字节来存储72。没有在字符串末尾添加`/x00`，而且没有初始化，可能存在泄漏。利用`visit`或者`show`函数打印的时候就能泄漏了。

```
unsigned \_\_int64 del()
{
int v1; // [rsp+0h] [rbp-10h]
char v2[2]; // [rsp+6h] [rbp-Ah] BYREF
unsigned \_\_int64 v3; // [rsp+8h] [rbp-8h]
v3 = \_\_readfsqword(0x28u);
printf("Input the idx:");
v1 = getint();
if ( v1 >= 0 && v1 <= 1 && notes[v1] )
{
free((void \*)notes[v1]);
printf("Clear?(y/n):");
readn(v2, 2uLL);
if ( v2[0] == 121 )
notes[v1] = 0LL;
puts("Done!");
}
else
{
puts("No such note!");
}
return \_\_readfsqword(0x28u) ^ v3;
}
```

当`clear`选择`n`的时候，不会清空`note`数组的指针，而`edit`和`show`都是通过这个来判断一个`note`是否存在。

## 漏洞利用的参考程序

参考链接，<mark class="encrypted">fccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1k6$3!0U0K9r3g2F1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5^5i4K6u0V1x3e0q4Q4x3X3b7J5y4W2)9J5k6r3u0U0N6r3k6Q4x3X3b7J5x3o6p5^5i4K6u0W2K9s2c8E0L8l9`.`.</mark>

受上文的启发，虽然他的图画错了（头节点不应该指向其`fd`而应该指向`chunk`头）

### 实验该参考程序过程发现了一个小现象：

当`malloc(0x20)`，分配的`chunk`的`size`为`0x21`

当`malloc(0x28)`，分配的`chunk`的`size`为`0x21`

当`malloc(0x29)`，分配的`chunk`的`size`为`0x41`

`0x20=32`字节，是分配一个`chunk`的最小空间=`presize+size+fd+bk`=32字节。`size`后面多1表示上一个`chunk`的状态。可以看到当`malloc(28)`，显示的`size`仍然为`0x21`，肯定是和后一个`chunk`的`presize`复用了。

```
#include <unistd.h>
#include <stdlib.h>
#include <malloc.h>
void main(){
void \*a = malloc(0x28);
void \*b = malloc(0x28);
// fill the tcache
for(int i=0; i<7 ;i++){
free(a);
}
sleep(0);
free(b);//fast bin
//What will happen with this:
free(a);// fast bin
}
```

free b后：

```
pwndbg> heap
Allocated chunk | PREV\_INUSE
Addr: 0x555555757000
Size: 0x251
Free chunk (tcache) | PREV\_INUSE
Addr: 0x555555757250
Size: 0x31
fd: 0x555555757260
Free chunk (fastbins) | PREV\_INUSE
Addr: 0x555555757280
Size: 0x31
fd: 0x00
Top chunk | PREV\_INUSE
Addr: 0x5555557572b0
Size: 0x20d51
pwndbg> bins
tcachebins
0x30 [ 7]: 0x555555757260 ◂— 0x555555757260 /\* '`ruUUU' \*/
fastbins
0x20: 0x0
0x30: 0x555555757280 ◂— 0x0
0x40: 0x0
0x50: 0x0
0x60: 0x0
0x70: 0x0
0x80: 0x0
unsortedbin
all: 0x0
smallbins
empty
largebins
empty
```

free a之后：

发现 `a`也进了`fast bin`里面，而且其`fd`指向了`b`的`presize`字段。这样我们就可以通过`malloc`从`tcache bin`里得到`a`的`fd`，从而修改`b`的`presize`，甚至`presize`后面的`size`等内容

```
pwndbg> heap
Allocated chunk | PREV\_INUSE
Addr: 0x555555757000
Size: 0x251
Free chunk (fastbins) | PREV\_INUSE
Addr: 0x555555757250
Size: 0x31
fd: 0x555555757280
Free chunk (fastbins) | PREV\_INUSE
Addr: 0x555555757280
Size: 0x31
fd: 0x00
Top chunk | PREV\_INUSE
Addr: 0x5555557572b0
Size: 0x20d51
pwndbg> bins
tcachebins
0x30 [ 7]: 0x555555757260 —▸ 0x555555757280 ◂— 0x0
fastbins
0x20: 0x0
0x30: 0x555555757250 —▸ 0x555555757280 ◂— 0x0
0x40: 0x0
0x50: 0x0
0x60: 0x0
0x70: 0x0
0x80: 0x0
unsortedbin
all: 0x0
smallbins
empty
largebins
empty
```

## 漏洞利用思路

我们的目标是最终执行`system('/bin/sh')`，而且是通过堆来完成。在《HITB CTF 2018 gundam》中，我们通过动态获得`libc`基地址，进而计算出`_free_hook`地址和`system`地址，想方设法在`_free_hook`地址处写入`system`地址，再创建1个内容为`'/bin/sh'`的`chunk`，然后释放，就可以触发`_free_hook`，最终执行`system('/bin/sh')`。这道题同样可以采取这种思路。1.要动态获得`libc`基地址，就要用到`unsorted bin`，在`tcache`的`count`为7的情况下，将符合`unsorted bin`大小的`chunk`释放到`unsorted bin`中。该`chunk`前向指针`fd`和后向指针`bk`的值就是要泄露的地址（详细分析见上一篇《HITB CTF 2018 gundam分析》）。

2.题目中创建的`house of Atum` `chunk`的大小为`0x51`。显然，这样`chunk`释放后只能进入`tcache bin`和`fast bin`，要想使其进入`unsorted bin`，就得改变指定`chunk`的大小。

3.要在`_free_hook`地址处写入`system`地址，就得构建1个以`_free_hook`地址为数据区地址的`chunk`，即`_free_hook-0x10`为起始地址的`chunk`，以`system`地址作为内容参数。

以下分析和调试过程基于以上3点考虑，通过对堆块`chunk`的灵活操作，成功执行获得`shell`。

## 泄露Chunk的fd地址

```
连续释放同一个chunk7次后,此时通过show即可获得chunk 0的fd的地址，书本中记为heap\_addr
--------
tcachebin[7] -> chunk 0.fd <- chunk 0.fd
fastbin[] ：null
--------
```

## 伪造chunk

动态获得了`heap_addr`，即`chunk0`的`next`指针，后面该如何利用?

根据前面分析，要获得`libc`的基地址，就要改变`chunk0`的大小为`0x91`。`chunk0`的`size`域位于`chunk0`头部16个字节的后半部分中，可以考虑创建1个以`chunk0-0x10`为起始地址的`chunk1`，将`chunk0`的新的`size`值（`0x91`）作为`chunk1`的内容。要从`tcachebin`中分配`chunk1`时，前提是`chunk1`在`tcachebin`中，有两种方式可以使`chunk1`(`chunk0-0x10`为起始地址）进入`tcachebin`：一种是构造`tcache bin`的`double free` 然后构造`chunk0-0x10`为起始地址的`fake chunk`，另一种是将`chunk1`链接到`fastbin`中某个`chunk`后面，这样当`chunk`被从`fastbin`中分配时，其后面的`chunk1`就会被移到`tcache`中。我们首先分析第一种方法：

因为题目限制只能创建2个`chunk`，所以构造了`faka chunk`后，已经创建了2个`chunK`了，此时这两个`chunk`都是指向`chunk 0`的，无论释放哪一个，都会导致`faka chunk`丢失。

如下，两个已经创建的`chunk`都是指向`0x5616cec43260`的，无论释放哪一个都会导致`fake chunk` `0x5616cec43250`的丢失。

所以本题用了两次把`fastbin`中的`fake chunk`转移到`tcache bin`

## 泄露libc地址

连续释放`chunk0` 7次，将会使`chunk0`进入`0x90`大小的`tcachebin`中，再释放1次，`chunk0`将会进入`unsortedbin`。就可以按像`gundam`那样泄漏`glibc`地址，就是先

分析`tcache`的结构体是位于堆的低地址的最开头，也是一个`chunk`。

## 为什么创建的是0x50大小的Chunk，而不是0x48?

我们看到代码中是`malloc(0x48)`,`0x48`是`userdata`的大小，还需要加上`presize`和`size`的大小，也就是`0x48+0x10=0x58`,然后由于该`chunk`被使用，所以会占用下一个`chunk`的`presize`字段，所以`0x58-0x8=0x50`。所以每次只用分配`0x50`大小的`chunk`.

## 带详细注释的代码
