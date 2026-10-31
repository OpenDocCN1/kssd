
# [原创]【NKCTF】babyHeap-Off by one&Tcache Attack

## 目录

[TOC]

## 程序分析

### IDA静态分析

#### 伪代码分析

##### main函数

```C
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
int choice; // [rsp+0h] [rbp-10h]
char buf[4]; // [rsp+4h] [rbp-Ch] BYREF
unsigned \_\_int64 v6; // [rsp+8h] [rbp-8h]
v6 = \_\_readfsqword(0x28u);
init(argc, argv, envp);
while ( 1 )
{
while ( 1 )
{
menu();
read(0, buf, 4uLL);
choice = strtol(buf, 0LL, 10);
if ( choice <= 5 && choice > 0 )
break;
puts("Index error.");
}
if ( choice == 5 )
break;
switch ( choice )
{
case 1:
add();
break;
case 2:
delete();
break;
case 3:
edit();
break;
default:
show();
break;
}
}
puts("Goodbye and welcome to use it next time.");
return 0;
}
```

##### add函数

```
unsigned \_\_int64 add()
{
signed int index; // [rsp+Ch] [rbp-14h]
int size; // [rsp+10h] [rbp-10h]
char buf[4]; // [rsp+14h] [rbp-Ch] BYREF
unsigned \_\_int64 v4; // [rsp+18h] [rbp-8h]
v4 = \_\_readfsqword(0x28u);
printf("Enter the index: ");
read(0, buf, 4uLL);
index = strtol(buf, 0LL, 10);
if ( (unsigned int)index > 0xF )
{
puts("Up to 16 nkctf notes can be created.");
}
else if ( note\_array[index] )
{
puts("Sorry, this nkctf note has already been used.");
}
else
{
printf("Enter the Size: ");
read(0, buf, 4uLL);
size = strtol(buf, 0LL, 10);
if ( size <= 256 )
{
note\_size[index] = size;
note\_array[index] = malloc(note\_size[index]);
if ( !note\_array[index] || !note\_size[index] )
my\_error("malloc()", 0xFFFFFFFFLL);
}
else
{
puts("This nkctf note is too big.");
}
}
return \_\_readfsqword(0x28u) ^ v4;
}
```

##### delete函数

```C
unsigned \_\_int64 delete()
{
unsigned int v1; // [rsp+0h] [rbp-10h]
char buf[4]; // [rsp+4h] [rbp-Ch] BYREF
unsigned \_\_int64 v3; // [rsp+8h] [rbp-8h]
v3 = \_\_readfsqword(0x28u);
printf("Enter the index: ");
read(0, buf, 4uLL);
v1 = strtol(buf, 0LL, 10);
if ( v1 > 0xF )
{
puts("Index error.");
}
else if ( note\_array[v1] )
{
free((void \*)note\_array[v1]);
note\_array[v1] = 0LL;
note\_size[v1] = 0;
if ( note\_array[v1] || note\_size[v1] )
my\_error("free()", 4294967294LL);
}
else
{
puts("The nkctf note to be freed does not exist.");
}
return \_\_readfsqword(0x28u) ^ v3;
}
```

##### edit函数

```C
unsigned \_\_int64 edit()
{
unsigned int v1; // [rsp+0h] [rbp-10h]
char buf[4]; // [rsp+4h] [rbp-Ch] BYREF
unsigned \_\_int64 v3; // [rsp+8h] [rbp-8h]
v3 = \_\_readfsqword(0x28u);
printf("Enter the index: ");
read(0, buf, 4uLL);
v1 = strtol(buf, 0LL, 10);
if ( v1 > 0xF )
{
puts("Index error.");
}
else if ( note\_array[v1] )
{
printf("Enter the content: ");
my\_read(note\_array[v1], (unsigned int)note\_size[v1]);
}
else
{
puts("The nkctf note to be filled was not created.");
}
return \_\_readfsqword(0x28u) ^ v3;
}
```

##### show函数

```C
unsigned \_\_int64 show()
{
unsigned int v1; // [rsp+0h] [rbp-10h]
char buf[4]; // [rsp+4h] [rbp-Ch] BYREF
unsigned \_\_int64 v3; // [rsp+8h] [rbp-8h]
v3 = \_\_readfsqword(0x28u);
printf("Enter the index: ");
read(0, buf, 4uLL);
v1 = strtol(buf, 0LL, 10);
if ( v1 > 0xF )
{
puts("Index error.");
}
else if ( note\_array[v1] )
{
puts((const char \*)note\_array[v1]);
}
else
{
puts("The nkctf note to be printed was not created.");
}
return \_\_readfsqword(0x28u) ^ v3;
}
```

##### my_read函数【漏洞】

```C
\_\_int64 \_\_fastcall my\_read(\_\_int64 a1, int a2)
{
unsigned int v3; // [rsp+10h] [rbp-10h]
int i; // [rsp+14h] [rbp-Ch]
for ( i = 0; i <= a2; ++i )//逻辑错误导致1个字节的溢出
{
v3 = read(0, (void \*)(i + a1), 1uLL);
if ( \*(\_BYTE \*)(i + a1) == 10 )
break;
}
return v3;
}
```

### 分析总结

**保护全开**，CRUD全都提供了，但是没有明显可利用的地方，问题出在`edit()`函数输入内容时使用的`my_read()`函数，看名字都很可移:)，其中根据创建时输入的`chunkSize`进行循环，由于逻辑错误导致会多读取一个字节，可以进行溢出

## 漏洞利用及原理

### 可利用漏洞

- 堆溢出Off by one

### 利用分析

已知在向堆输入内容时可进行一个字节的溢出，所以当我们申请`0x?8`个字节时便可实际输入`0x?9`篡改下一个`chunkSize`，于是我们可以进行如下利用

- 申请3个大小为`0x68`的chunk
- 通过溢出篡改`chunk#1->size=0xF1`
- 释放`chunk#1`使其进入`Tcache#0xf0`
- 重新申请`chunk#1`，大小为`0xE8`

此时`chunk#1`被重新至于原位且此时记录大小为`0xE8`，真实大小依旧为`0x68`，可随意篡改，泄露`chunk#2`，于是进行以下利用

- 篡改`chunk#2->size=0x4b1`
- 申请4个大小为`0x100`的chunk
- 释放`chunk#2`使其进入`Unsorted Bin`
- 向`chunk#1`中填充大小为`0x70`的字符
- 使用`show()`函数打印`chunk#1`以泄露`main_arena`

此时我们已经实现了 **libcBase Leak** 和 **Chunk Extends** ，题目给出了的ibc版本为`glibc-2.32`，由于应用了`Tcache`，所以决定使用`Tcache poisoning`进行任意地址写

### 利用原理和注意事项

`Tcache`整体和`FastBin`较相似，同采取 **单链表 LIFO** 进行管理，也既其`FreeChunk`仅有`fd`字段，区别是在利用时，`FastBin`等都将对`ChunkHeader`进行检查，而`Tcache`的检查极其少，导致安全性低，其中值得注意的一项检查和一项保护措施分别如下

- 针对`fd`字段的混淆
- 针对被申请地址的对齐检查
- `tcache_perthread_struct`结构体中的`counts`字段记录了当前分类中存在多少个可分配`FreeChunk`

在`glibc-2.31`后，对`FastBin/Tcache`的`fd`字段进行了混淆保护，当第一个`FreeChunk`进入分类中时，`&FreeChunk#0 >> 3`将作为`key`被保存并写入`FreeChunk#0->fd`，而后该分类的每个`FreeChunk->fd`在存取时都将与`key`进行异或，所以若是我们要篡改`fd`字段，则需要泄露`key`

在 `glibc 2.29` 之前，`TcacheChunk` 以 `16 bytes` 进行对齐，而在这之后，当申请的大小`64<=size<=128`，则以`16 bytes`进行对齐，其它情况下则以 `8 bytes` 进行对齐，所以若是申请的`&FakeChunk`不符合对齐条件，需要`-=8`以绕过检查

当`tcache_perthread_struct->counts=0`时，则会直接跳过`Tcache`从而去别处分配

#### 注意事项的解决方案：

1. 先泄露并保存`Tcache[X]->key`后在篡改`fd`时与真实`Address`进行异或
2. 若是报错`malloc(): unaligned tcache chunk detected`，则将`FakeFd-8`
3. 篡改使`tcache_perthread_struct->counts+1`或者篡改`FreeChunk#1`或之后`FreeChunk`的`fd`以保证在申请到`FakeChunk`前`tcache_perthread_struct->counts!=0`

## Exploit
