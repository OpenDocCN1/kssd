
```c
typedef struct {
uint32 next;
uint32 len;
} LG\_mem\_free\_t;
uint32 LG\_mem\_min;
uint32 LG\_mem\_top;
LG\_mem\_free\_t LG\_mem\_free;
char \*LG\_mem\_base;
uint32 LG\_mem\_len;
char \*Origin\_LG\_mem\_base;
uint32 Origin\_LG\_mem\_len;
char \*LG\_mem\_end;
uint32 LG\_mem\_left;
#define realLGmemSize(x) (((x) + 7) & (0xfffffff8))
// 初始化内存管理器
// baseAddress: 托管的内存的首地址，是一个模拟器内的地址
// len: 内存的总长度
void initMemoryManager(uint32 baseAddress, uint32 len) {
printf("initMemoryManager: baseAddress:0x%X len: 0x%X\n", baseAddress, len);
Origin\_LG\_mem\_base = toPtr(baseAddress);
Origin\_LG\_mem\_len = len;
LG\_mem\_base = (char \*)((uint32)(Origin\_LG\_mem\_base + 3) & (~3));
LG\_mem\_len = (Origin\_LG\_mem\_len - (LG\_mem\_base - Origin\_LG\_mem\_base)) & (~3);
LG\_mem\_end = LG\_mem\_base + LG\_mem\_len;
LG\_mem\_free.next = 0;
LG\_mem\_free.len = 0;
((LG\_mem\_free\_t \*)LG\_mem\_base)->next = LG\_mem\_len;
((LG\_mem\_free\_t \*)LG\_mem\_base)->len = LG\_mem\_len;
LG\_mem\_left = LG\_mem\_len;
#ifdef MEM\_DEBUG
LG\_mem\_min = LG\_mem\_len;
LG\_mem\_top = 0;
#endif
}
void \*my\_malloc(uint32 len) {
LG\_mem\_free\_t \*previous, \*nextfree, \*l;
void \*ret;
len = (uint32)realLGmemSize(len);
if (len >= LG\_mem\_left) {
printf("my\_malloc no memory\n");
goto err;
}
if (!len) {
printf("my\_malloc invalid memory request");
goto err;
}
if (LG\_mem\_base + LG\_mem\_free.next > LG\_mem\_end) {
printf("my\_malloc corrupted memory");
goto err;
}
previous = &LG\_mem\_free;
nextfree = (LG\_mem\_free\_t \*)(LG\_mem\_base + previous->next);
while ((char \*)nextfree < LG\_mem\_end) {
if (nextfree->len == len) {
previous->next = nextfree->next;
LG\_mem\_left -= len;
#ifdef MEM\_DEBUG
if (LG\_mem\_left < LG\_mem\_min)
LG\_mem\_min = LG\_mem\_left;
if (LG\_mem\_top < previous->next)
LG\_mem\_top = previous->next;
#endif
ret = (void \*)nextfree;
goto end;
}
if (nextfree->len > len) {
l = (LG\_mem\_free\_t \*)((char \*)nextfree + len);
l->next = nextfree->next;
l->len = (uint32)(nextfree->len - len);
previous->next += len;
LG\_mem\_left -= len;
#ifdef MEM\_DEBUG
if (LG\_mem\_left < LG\_mem\_min)
LG\_mem\_min = LG\_mem\_left;
if (LG\_mem\_top < previous->next)
LG\_mem\_top = previous->next;
#endif
ret = (void \*)nextfree;
goto end;
}
previous = nextfree;
nextfree = (LG\_mem\_free\_t \*)(LG\_mem\_base + nextfree->next);
}
printf("my\_malloc no memory\n");
err:
return 0;
end:
return ret;
}
void my\_free(void \*p, uint32 len) {
LG\_mem\_free\_t \*free, \*n;
len = (uint32)realLGmemSize(len);
#ifdef MEM\_DEBUG
if (!len || !p || (char \*)p < LG\_mem\_base || (char \*)p >= LG\_mem\_end || (char \*)p + len > LG\_mem\_end || (char \*)p + len <= LG\_mem\_base) {
printf("my\_free invalid\n");
printf("p=%d,l=%d,base=%d,LG\_mem\_end=%d\n", (int32)p, len, (int32)LG\_mem\_base, (int32)LG\_mem\_end);
return;
}
#endif
free = &LG\_mem\_free;
n = (LG\_mem\_free\_t \*)(LG\_mem\_base + free->next);
while (((char \*)n < LG\_mem\_end) && ((void \*)n < p)) {
free = n;
n = (LG\_mem\_free\_t \*)(LG\_mem\_base + n->next);
}
#ifdef MEM\_DEBUG
if (p == (void \*)free || p == (void \*)n) {
printf("my\_free:already free\n");
return;
}
#endif
if ((free != &LG\_mem\_free) && ((char \*)free + free->len == p)) {
free->len += len;
} else {
free->next = (uint32)((char \*)p - LG\_mem\_base);
free = (LG\_mem\_free\_t \*)p;
free->next = (uint32)((char \*)n - LG\_mem\_base);
free->len = len;
}
if (((char \*)n < LG\_mem\_end) && ((char \*)p + len == (char \*)n)) {
free->next = n->next;
free->len += n->len;
}
LG\_mem\_left += len;
}
```

由于上面的`my_free()`在释放内存时要求传入释放内存的长度，与c语言的`free()`用法不同，因此还需要增加两个包装后的函数：

```c
void \*my\_mallocExt(uint32 len) {
uint32 \*p;
if (len == 0) {
return NULL;
}
p = my\_malloc(len + sizeof(uint32));
if (p) {
\*p = len;
return (void \*)(p + 1);
}
return p;
}
void my\_freeExt(void \*p) {
if (p) {
uint32 \*t = (uint32 \*)p - 1;
my\_free(t, \*t + sizeof(uint32));
}
}
```

`my_mallocExt()`和`my_freeExt()`直接替换系统的`malloc()`和`free()`进行内存管理，通过这种方式获得的内存只需要通过 **地址转换函数** 处理一下就可以在模拟器内外自由使用了。

一个常见的用法是传递字符串：

```c
uint32\_t copyStrToEmu(char \*str) {
if (!str) return 0;
uint32\_t len = strlen(str) + 1;
void \*p = my\_mallocExt(len);
memcpy(p, str, len);
return toAddr(p);
}
// str 将是一个模拟器内的地址，直接传递给模拟器
uint32\_t str = copyStrToEmu("test.txt");
// str2 将是一个本地指针，可以直接使用
char \*str2 = toPtr(str);
printf("%s\n", str2);
```

## 结束

以上就是我在使用Unicorn用到的最核心技术，如果对实际应用的真实案例感兴趣可以研究下我的开源模拟器 <a href="elink@d5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7k6h3&6Y4L8h3W2F1k6K6l9H3i4K6u0r3N6X3#2J5M7l9`.`.">vmrp</a>

2021/04/05 23:11 中国标准时间

原文地址<mark class="encrypted">033K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7k6h3&6Y4L8h3W2F1k6K6l9H3i4K6u0r3N6h3&6A6j5$3!0J5L8W2)9#2k6X3g2F1k6$3W2F1k6g2)9#2k6Y4c8#2N6r3!0J5K9h3q4D9</mark>

# 使用unicorn-engine开发模拟器

## 什么是unicorn引擎

Unicorn是基于qemu开发的一个CPU模拟器，支持常见的各种指令集，能在各种系统上运行。

GITHUB项目地址：18eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2L8X3W2U0L8%4u0F1i4K6u0V1k6h3&6Y4K9h3&6W2i4K6u0r3N6h3&6A6j5$3!0J5L8R3`.`.

官网地址：222K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2L8X3W2U0L8%4u0F1i4K6u0V1k6h3&6Y4K9h3&6W2i4K6u0W2L8%4u0Y4i4K6u0r3

一个中文API文档：173K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5h3u0W2L8%4u0Q4x3V1k6y4K9h3y4J5L8#2)9J5k6q4g2F1K9h3y4G2M7X3&6Q4x3X3c8q4L8X3N6A6L8X3g2Q4x3X3c8m8f1p5W2Q4x3X3c8p5L8$3y4#2L8h3g2F1N6r3q4@1K9h3!0F1

它只是一个CPU模拟器，所以它的API非常简洁，它提供了各种编程语言的绑定，你可以选择喜欢编程语言进行开发，被加载到unicorn中执行的程序对内存的每一次读写，每一条指令的执行都在你的掌控之中，并且被unicorn加载运行的程序对这一切是完全无感知的

## 准备工作

写这篇文章的目的是帮助理解我的开源模拟器 [vmrp](https://bbs.kanxue.com/elink@c16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7k6h3&6Y4L8h3W2F1k6K6l9H3i4K6u0r3N6X3#2J5M7l9`.`.) 的工作原理，因此这里介绍的是windows下用c语言开发arm模拟器，网络上其它的unicorn文章也仅仅只描述了简单的应用，对于一些关键的核心问题并没有给出答案。

此文列举的几个案例都是为了学习单一知识点而精心设计的简单案例，刻意省去了堆栈操作，因此和实战是不一样的，在最后会介绍到堆栈操作，因此想要应用到实战，必需学习完此文的所有内容。

使用到的GCC编译器：

0e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2X3L8%4u0Y4k6g2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3A6W2j5%4c8K6i4K6u0r3L8h3W2F1k6%4N6Q4x3X3c8%4y4U0c8Q4x3V1k6X3K9h3I4W2M7#2)9J5c8W2c8G2L8$3I4U0K9r3q4A6L8Y4y4Q4x3U0f1J5x3s2c8S2M7X3N6W2N6s2c8A6L8X3N6Q4x3U0f1J5x3q4N6A6L8U0j5@1i4K6u0r3f1r3g2J5M7$3!0F1j5h3I4Q4x3U0f1J5x3p5u0#2K9h3I4V1M7#2)9J5c8X3#2A6L8X3N6%4i4K6u0V1j5Y4g2A6L8r3c8K6i4K6u0r3z5q4)9J5k6e0q4Q4x3X3f1H3i4K6u0r3N6r3S2J5k6h3q4V1M7#2)9J5k6s2m8G2M7$3W2^5i4K6u0r3M7$3A6D9K9W2)9J5c8Y4R3^5y4W2)9#2k6U0j5@1i4K6u0V1z5q4)9J5k6e0q4Q4x3X3f1H3i4K6u0V1M7X3g2D9k6h3q4K6k6g2)9J5k6s2m8G2M7$3W2^5i4K6u0V1M7$3A6D9K9W2)9J5k6s2u0@1i4K6g2X3N6U0k6Q4x3X3c8J5k6i4j5H3i4K6u0W2y4%4Z5`.

使用的unicorn下载地址：

5b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2L8X3W2U0L8%4u0F1i4K6u0V1k6h3&6Y4K9h3&6W2i4K6u0r3N6h3&6A6j5$3!0J5L8W2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1j5I4i4K6u0W2x3q4)9J5k6e0u0Q4x3V1k6#2L8X3W2U0L8%4u0F1i4K6u0V1x3g2)9J5k6e0m8Q4x3X3f1J5i4K6u0V1N6$3W2F1x3K6u0Q4x3X3g2\*7K9i4l9`.

需要特别注意的是必需使用1.0.2以上版本的unicorn，因为我在使用unicorn开发模拟器时unicorn还处于1.0.1版本，1.0.1版本存在一个奇怪的BUG导致模拟器出现程序跑飞的情况，在1.0.2版本发布后再次尝试才终于成功。

另外使用unicorn一定会涉及到汇编和反汇编知识，如果有这方面的知识理解起来会更容易

## helloworld

高级编程语言会有类、对象等概念，这些概念是为了方便人脑的思考，总结出来的一种编程习惯，CPU只知道如何处理最简单的指令，无论程序有多复杂编译后给CPU执行的都是最简单的一条条指令，通常一个CPU指令集也就几十条指令而已，计算机所有的功能都是通过这几十条指令完成的，汇编语言其实就是每一条指令的直接使用和一些约定的编程规范

你可以简单的把一条指令理解为编程语言中的一个函数

让我们通过几条指令来做一个加法运算

|  |  |

| --- | --- |

| 1  2  3 | `mov r0,``1`       `/``/` `将数字``1``送入寄存器r0`  `mov r1,``2`       `/``/` `将数字``2``送入寄存器r1`  `add r2,r0,r1` `/``/` `将r0``+``r1的运算结果送入寄存器r2` |

这里推荐一个学习arm汇编的工具：496K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3&6G2N6i4S2A6M7K6W2Q4x3V1k6m8f1V1#2e0N6s2u0G2L8X3M7`.

为了方便演示，简化代码，下面的代码假设每一步都是成功的，所以没有错误处理的部分，实际应用时应该对unicorn api的返回值做判断

|  |  |

| --- | --- |
