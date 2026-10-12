
# asm 操作数组【回答一个朋友的问题】

> 原文：[`www.pediy.com/kssd/pediy10/98815.html`](https://www.pediy.com/kssd/pediy10/98815.html)

## Asm 中的数组研究

### 前言

这个不算是 asm 魅力系列的文章，只是回答一个朋友的问题！因为写的长了就发出来，希望对大家有帮助！

### 问题

`http://bbs.pediy.com/showthread.php?p=694325#poststop`

任何语言中的数组实际上都是指针的概念，你这样想，一些数据放在内存的一个区域里面，我们要怎么访问他们呢？

步骤：
首先我们得知道这些数据在哪个内存块。其次要知道怎么在这个内存块里面找到我们要的那个指定的数据！

`C` 或者是其他的高级语言把这些东西格式化了，比如 `char a[]={1,2,3,4};` 然后访问的时候 `a[0]  a[1]  a[2]   a[3]` 实际上还是以前的老话，你要是理解了这种抽象过的符号表示的话，其实是很好理解的。但是 `asm` 比较接近人的思维，什么思维呢？就是我在步骤里提到的这种人人都能立刻想到的思路！

### （一）

我先就用 `masm` 来说把！那么 `masm` 也做了一些格式化，所以我们操作数组也可以比较方便！

来看一小段：

```
.data
Array BYTE 1,2,3
.code
Mov esi,OFFSET Array
```

那么要访问第一个数组元素的话，就是 `mov eax,[esi]`, 第二个呢就是 `inc esi , mov eax,[esi]`

是和 `C` 一样简单的。

关键就是 `offset`，他是干什么呢？就是相当于是把当前的指针指向 `Array` 这个内存区域的起始。OK。

那么来看这样的两个定义

来看两段定义：

```
arrayA BYTE 1h,2h,3h,4h
arrayB WORD 1000h,2000h,3000h,4000h
```

我们怎么访问？

我举个例子：

```
.data
arrayB WORD 1000h,2000h,3000h,4000h
.code
Mov esi,offset arrayB
Mov eax,[esi]
Add esi,4
```

知道该注意什么地方了把！

### （二）

上面看到了，实际上 `inc esi` 这样是很麻烦的。那么好呢容易想到变址操作数。

那么
变址操作数的话，我们可以这样写：

```
.data
arrayB WORD 1000h,2000h,3000h,4000h
.code
Mov esi,offset arrayB
Mov eax,[esi]
Mov ebx,[esi+2]
Mov ecx,[esi+4]
```

那么我们来阐述清楚一个小细节，`c` 语言里比如 `char aa[2]={0,1};` 那么 `printf("%x",aa);` 应该是输出 `aa` 的内存地址的，`printf("%c",*aa)`，这样输出了第一个元素 `0`. 为什么？因为实际上 `aa` 这个名字实际上是 `aa[0]`，那么同样 `asm` 里面我们也可以这样考虑。

那么我们这样访问元素：

```
.data
arrayB WORD 1000h,2000h,3000h,4000h
.code
Mov esi,0h
Mov eax,[arrayB+esi]
Mov ebx,[arrayB+esi+2]
Mov ecx,[arrayB+esi+4]
```

这样就更方便了，实际上跟 `C` 就没有多大的区别了！

### （三）

我们先看个 `c` 语言的小例子：

```
#include "stdio.h"
#include "stdlib.h"

int main()
{
  char a[2]={1,2};
  for(int i=0;i<sizeof(a);i++)
  {
  printf("%d\n",a[i]);
  }
  return 0;
}
```

输出的结果可想而知：`1 2`

那么我们也可以这样：

```
#include "stdio.h"
#include "stdlib.h"

int main()
{
  char a[2]={1,2};
  for(int i=0;i<sizeof(a);i++)
  {
  printf("%d\n",*a+i);
  }
  return 0;
}
```

结果一样，这是什么意思呢？就是我说过的，数组就是个指针的操作，指针更灵活，也不要一提指针就想起 `C` 语言里的这个那个，记住，指针是人的日常生活里广大人名群众喜闻乐见的一种思维方式！------倒

那么用指针的概念，来操作数组的话，`asm` 里面同样可以写：

```
.data
Array db 1h,2h,3h,4h
.code
Point PTR BYTE Array

Mov esi,point
Mov eax,[esi]
```

好了，实际上是很简单的！

对于 `c` 里面的 `a[i]` 这样的东西你反汇编看下，实际上用的就是变址：

```
9:        printf("%d\n",a[i]);
0040D738  mov         ecx,dword ptr [ebp-8]
0040D73B  movsx       edx,byte ptr [ebp+ecx-4]
```

好了，基本就这些东西！

建议你遇到这样的问题要多思考，不要只想着问，因为这些都是基本的程序层面的问题！

用我师傅 `achillis` 的话说：这叫编码！

我很赞同这个说法，这些层面的东西都只能叫做编码，只是在遵循一些规则。大牛是什么？

大牛就是充分的了解了这些规则，熟练的变通这些规则，甚至突破规则！
