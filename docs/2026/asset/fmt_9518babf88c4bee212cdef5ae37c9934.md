
# [原创]Teaser Dragon CTF 2018 pwn

MY BLOG: <a href="elink@b63K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8Y4N6i4t1I4i4K6u0W2j5$3y4Q4x3V1j5J5x3o6p5^5i4K6u0r3x3e0m8Q4x3V1j5I4x3W2)9J5c8W2c8W2j5i4y4W2M7W2)9J5k6p5c8J5j5h3N6G2L8W2)9J5k6p5y4f1c8W2)9J5k6o6t1H3x3e0S2Q4x3V1j5`.">0gur1.cc</a>

## Production

这题据说是远程没有启用`assert`，导致文件没有`close`。但我当时没看出来，现在平台关闭了也试不了了TAT，就看了些大佬（<a href="elink@4a7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1k6$3!0U0K9r3g2F1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6W2Q4x3V1j5J5z5g2)9J5c8W2c8W2j5i4y4W2M7W2)9J5k6p5c8J5j5h3N6G2L8W2)9J5k6p5y4f1c8W2)9J5k6o6t1H3x3e0S2Q4x3V1j5`.">Ne0</a> 和<a href="elink@d80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6@1K9h3#2W2i4K6u0W2L8%4u0Y4i4K6u0r3N6%4u0A6N6r3g2#2M7q4)9J5c8U0p5I4y4o6f1@1">dcua</a> ）的wp，了解这个漏洞是怎么用的。duca的wp更细一点，两者结合更容易理解。

### 相关知识

#### 关于`assert`

> **DESCRIPTION** If the macro `NDEBUG` was defined at the moment `<assert.h>` was last included, the macro `assert()` generates no code, and hence does nothing at all. Otherwise, the macro `assert()` prints an error message to standard error and terminates the program by calling `abort(3)` if expression is false (i.e., compares equal to zero).

如果`assert.h`中包含了宏`NDEBUG`，`assert`将不生成代码，不执行任何操作；否则当`assert`中的条件不满足时，`assert`会输出错误并终止程序。

也就是说，如果远程`assert.h`中有`NDEBUG`，那么在`read_lyrics`中的关闭文件操作就不会执行：

```cc
if (strstr(buffer, "DrgnS")) {
printf("[-] Attack detected and stopped!\n");
assert(close(globals::records[idx]) == 0);
memmove(&globals::records[idx], &globals::records[idx + 1],
(globals::records.size() - idx - 1) * sizeof(int));
globals::records.pop_back();
return true;
}
```

`memmove`通过数组的前移实现删除，因此当读到的内容包含“DrgnS”时，只会在数组中删除元素，但文件仍处于打开状态，即`fd`的数量没有减少。

#### 关于`setrlimit`

程序进行了一些限制，其中就包含能够打开文件描述符的最大值：

```cc
rlim.rlim_cur = rlim.rlim_max = 32;
setrlimit(RLIMIT_NOFILE, &rlim);
```

打开的文件个数超过32个时，再通过`open`打开文件就会失败。观察`open_lyrics`的检查顺序，有两个用到`open`的操作位于`flag`之前，因此可以通过这一操作绕过`flag`的读取。

另外，程序中限制输入的`band`或者`song`中不能包含`../`，但是可以输入`..`绕过。

### 利用方法

参考<a href="elink@46dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1k6$3!0U0K9r3g2F1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6W2Q4x3V1j5J5z5g2)9J5c8W2c8W2j5i4y4W2M7W2)9J5k6p5c8J5j5h3N6G2L8W2)9J5k6p5y4f1c8W2)9J5k6o6t1H3x3e0S2Q4x3V1j5`.">Ne0大佬</a> 的思路。

1. 在`lyrics`二进制文件中有`DrgnS`字符串，因此打开16次该文件（`./data/../lyrics`），然后读取内容，每次都读到`DrgnS`处，此时`fd`为18（包含0,1,2标准的输入输出），而`record`数组因为`memmove`，元素都被删除了，`size`为0。

2. 打开12个任意的文件，此时`fd`为30.

3. `open_lyrics ./data/../flag`，在`read_lyrics`先是检查`path`是否为一个文件时，`open`文件，这时`fd1`为31，文件描述符有32个，把文件放入数组后，又检查`path`是否为链接文件，再次`open`，这时`fd2`=-1，并返回`true`。由此绕过了`flag`的检查

```cc
// Open the path, make sure that it's a file (and not e.g. directory), and
// save the file descriptor.
int fd1 = open(path, O_RDONLY);
if (fd1 == -1) {
return false;
}
struct stat st;
if (fstat(fd1, &st) != 0 || !S_ISREG(st.st_mode)) {
return false;
}
globals::records.push_back(fd1);
// Better safe then sorry. Make sure that the path also doesn't point to a
// symbolic link.
int fd2 = open(path, O_RDONLY | O_NOFOLLOW);
if (fd2 == -1) {
printf("[-] Detected attempt to open a symbolic link!\n");
// Some kind of attack detected?
return true;
}
close(fd2);
// Extra check to protect the flag.
if (strstr(path, "flag") != NULL) {
printf("[-] Not today\n");
close(globals::records.back());
globals::records.pop_back();
return false;
}
```

4. 由于`flag`中也包含`DrgnS`字符串，因此无法直接打印出来。`read_line_buffered`中，如果读到文件结尾处会返回0，`buffer`中的内容不会改变。并且如果连续调用`read_line_buffered`时，`buffer`在栈上的地址保持不变，内容也就不变。因此可以先对某个文件读到结尾，再读`flag`，再返回读那个文件，`buffer`中就会保持`flag`中的内容。

## Fast Storage

这题用到了一个奇怪的点：`abs(0x80000000)==0x80000000`。从开始看大佬的wp到自己捋顺一共花了两天时间ORZ。

### 基本逻辑

`add`，`print`和`edit`功能。

`add`函数输入`name`、`size`和`value`，并对`size`和`value`地址做简单的处理后一起存放。题目中实现了一个类似于哈希表的简单结构：对`name`进行哈希作为索引，出现碰撞后用`next`字段链接起来。

```c++
_DWORD *__fastcall add(__int64 a1, __int64 a2)
{
int v2; // eax@1
size_t v3; // rbx@7
int v4; // eax@7
char *name; // rax@9
size_t size; // [sp+8h] [bp-128h]@1
char buf; // [sp+10h] [bp-120h]@1
char v9; // [sp+10Fh] [bp-21h]@3
void *value_addr; // [sp+110h] [bp-20h]@1
__int64 v11; // [sp+118h] [bp-18h]@1
memset(&buf, 0, 0x100uLL);
v11 = 0LL;
size = 0LL;
value_addr = 0LL;
printf("Name: ", a2, &buf);
v2 = fileno(stdin);
v11 = read(v2, &buf, 0x100uLL);
if ( v11 <= 0 )
ouch();
v9 = 0;
printf("Size: ", &buf);
_isoc99_scanf("%lu", &size);
fgetc(stdin);
if ( size > 0x400 )
ouch();
value_addr = malloc(size);
if ( !value_addr )
ouch();
printf("Value: ", &size);
v3 = size;
v4 = fileno(stdin);
v11 = read(v4, value_addr, v3);
if ( v11 <= 0 )
ouch();
value_addr = (void *)((size << 48) | (unsigned __int64)value_addr);
name = strdup(&buf);
return insert_item((__int64)name, (__int64)value_addr);
}
```

其种`insert_item`函数是对`name`作一些运算并进行插入操作。我用的IDA6.8版本的就看不出来用了`abs`函数（当然即使看出来也不觉得有问题），据说7.0的可以直接显示。

```c++
_DWORD *__fastcall insert_item(__int64 name, __int64 content)
{
signed int v2; // ST1C_4@1
int v3; // ST18_4@1
int v4; // ST14_4@1
int idx; // ST1C_4@1
v2 = h1((_BYTE *)name);
v3 = h2(name);
v4 = h3((_BYTE *)name);
idx = ((v2 ^ (v2 >> 31)) - (v2 >> 31)) % 62; // abs
insert_idx(idx, name, content);
return set_hash_bits(idx, v3, v4);
}
```

通过`insert_idx`函数了解到哈希链表结点的结构。

```c++
_QWORD *__fastcall insert_idx(int idx, __int64 name, __int64 content)
{
_QWORD *result; // rax@3
__int64 v4; // [sp+8h] [bp-28h]@1
_QWORD *v5; // [sp+28h] [bp-8h]@1
v4 = content;
v5 = malloc(0x18uLL);
if ( !v5 )
ouch();
v5[1] = name;
v5[2] = v4;
*v5 = entry[idx];
result = entry;
entry[idx] = v5;
return result;
}
struct node{
struct node * next;
char * name;
char * value;
};
```

`set_hash_bits`函数用位运算记录`h2`和`h3`的结果：

```c++
_DWORD *__fastcall set_hash_bits(int idx, char a2, char a3)
{
_DWORD *result; // rax@1
result = bits;
bits[idx] |= (1 << a2) | (1 << a3);
return result;
}
```

`print`和`edit`都是通过`name`定位到结点，输出或修改`value`的值。

`find_by_name`函数中同样用到了`h1`，`h2`，`h3`函数，并在遍历链表前先对`bits[idx]`进行检查，查看`bits[idx]`是否有对应的几个`bit`。

```c++
signed __int64 __fastcall find(__int64 a1, __int64 a2)
{
int v2; // eax@1
signed __int64 result; // rax@4
unsigned __int64 *size; // [sp+0h] [bp-120h]@1
__int64 *content_addr; // [sp+8h] [bp-118h]@1
char name; // [sp+10h] [bp-110h]@1
char v7; // [sp+10Fh] [bp-11h]@3
unsigned __int64 v8; // [sp+110h] [bp-10h]@1
__int64 v9; // [sp+118h] [bp-8h]@1
memset(&name, 0, 0x100uLL);
v9 = 0LL;
v8 = 0LL;
printf("Name: ", a2, a2, a1);
v2 = fileno(stdin);
v9 = read(v2, &name, 0x100uLL);
if ( v9 <= 0 )
ouch();
v7 = 0;
v8 = find_by_name(&name);
if ( v8 )
{
*size = v8 >> 0x30;
*content_addr = v8 & 0xFFFFFFFFFFFFLL;
result = 1LL;
}
else
{
result = 0LL;
}
return result;
}
__int64 __fastcall find_by_name(char *name)
{
signed int v1; // ST24_4@1
int v2; // ST20_4@1
char v3; // al@1
__int64 result; // rax@2
int idx; // [sp+24h] [bp-Ch]@1
__int64 i; // [sp+28h] [bp-8h]@3
v1 = h1(name);
v2 = h2((__int64)name);
v3 = h3(name);
idx = ((v1 ^ (v1 >> 31)) - (v1 >> 31)) % 62; //abs
if ( (unsigned int)check(idx, v2, v3) )
{
for ( i = entry[idx]; i && strcmp(*(const char **)(i + 8), name); i = *(_QWORD *)i )
;
result = *(_QWORD *)(i + 16);
}
else
{
result = 0LL;
}
return result;
}
__int64 __fastcall check(int idx, char a2, char a3)
{
return (((1 << a2) | (1 << a3)) & bits[idx]) == ((1 << a2) | (1 << a3));
}
```

### bugs

大佬们究竟是怎么想到的？？

`abs(0x80000000)==0x80000000`。验证一下：

```c
#include<stdio.h>
#include<math.h>
#include<stdlib.h>
int main()
{
int a = 0x80000000;
printf("%d\n",a);
printf("%#x\n",a^(a>>31)-a>>31);
printf("%#x\n",abs(a));
}
```

输出结果：

```
./test
-2147483648
0x80000000
0x80000000
```

如果`h1(name)`为`0x80000000`，`idx = abs(0x80000000)%62=-2`，在`insert_idx`时进行`entry[idx] = node_addr`时，实际上是对`entry`之前的数据进行操作。查看`bss`结构，`entry[-2]`即为`bits[60]`。

```
.bss:0000000000202040 ; _DWORD bits[64]
.bss:0000000000202040 bits dd 40h dup(?) ; DATA XREF: set_hash_bits+Do
.bss:0000000000202040 ; set_hash_bits+3Do ...
.bss:0000000000202140 ; _QWORD entry[62]
.bss:0000000000202140 entry dq 3Eh dup(?) ; DATA XREF: insert_idx+4Ao
.bss:0000000000202140
```

### 漏洞利用
