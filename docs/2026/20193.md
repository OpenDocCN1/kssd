
# [分享] pwnable.kr random day5

## 题目

![](upload/attach/202101/760268_UM8JXFBFEVPRREZ.png)

## 解题过程

### 1. 查看文件列表

![](upload/attach/202101/760268_855R2ZH2CZJ379V.png)

`flag` 只对创建者 `random_pwn` 和 `root` 可读，而我们登录的用户是 `random`，无读权限。`random` 对 `random_pwn` 和 `random` 开放读和执行权限，而且权限里面有 `s`，因此 `random` 用户在执行这个文件时会被赋予 `root` 权限。

### 2. 查看 random.c 文件

```c
#include <stdio.h>
int main(){
unsigned int random;
random = rand(); // random value!
unsigned int key=0;
scanf("%d", &key);
if( (key ^ random) == 0xdeadbeef ){
printf("Good!\n");
system("/bin/cat flag");
return 0;
}
printf("Wrong, maybe you should try 2^32 cases.\n");
return 0;
}
```

我们的目标是执行 `system("/bin/cat flag")`，需满足 `key ^ random == 0xdeadbeef`

`key` 的值手动输入，我们可控

`random` 的值是由 `rand()` 产生，`rand()` 产生的随机数在每次运行的时候都是与上一次相同的。若要不同, 用函数 `srand()` 初始化它。此处没有使用 `srand()` 所以我们可以先执行一次程序查看 `random` 的值。

### 3. GDB 调试 random

#### 1. `disassaemble main` 查看 main 函数的汇编代码

![](upload/attach/202101/760268_2J7H74U9FPY7ARU.png)

与源码对照得知，`random` 的地址是 `%rbp - 0x4`，`key` 的地址是 `%rbp - 0x8` 我们这次只用到了 `random` 的地址

#### 2. 下断点，查看 random 值

`0x400606` 处完成了对 `random` 的赋值，在其后面找一地址下断点，执行后查看 `random` 的值。

```
b *0x0000000000400610
x/10x $rbp-4
# 第一个 x 是 examine 的简写，用于查看内存地址的值。
# 用法：x/<n/f/u> <addr>
# n,f,u 都是可选参数
# n是一个正整数，表示需要显示的内存单元的个数，一个内存单元的大小由后面的u定义
# f 表示显示的格式
# x 按十六进制格式显示变量。
# d 按十进制格式显示变量。
# u 按十六进制格式显示无符号整型。
# o 按八进制格式显示变量。
# t 按二进制格式显示变量。
# a 按十六进制格式显示变量。
# c 按字符格式显示变量。
# s 按字符串格式显示变量。
# f 按浮点数格式显示变量。
# u 表示从当前地址往后请求的字节数，GDB 默认是 4 个 bytes。
# b 表示单字节。
# h 表示双字节。
# w 表示四字节。
# g 表示八字节。
# 当我们指定了字节长度后，GDB会从指内存定的内存地址开始，读写指定字节，并把其当作一个值取出来。
# <addr>表示一个内存地址，也可以用寄存器（偏移）表示地址，如此次用的是 $rbp-4
```

![](upload/attach/202101/760268_6XCY9Y8G8B3HB45.png)

读得 `random` 的值为 `0x6b8b4567`

#### 3. 计算 key 的值

由 `key ^ random == 0xdeadbeef` 得 `0xdeadbeef ^ random == key`

利用 python 求得 `key` 的值 `print 0x6b8b4567^0xdeadbeef`

![](upload/attach/202101/760268_CZCEYUQNKEEKPHQ.png)

### 4. 输入 key 值，获得 flag

因为 `scanf()` 要求格式为 `%d`，所以我们应输入十进制

红框内为 `flag`

![](upload/attach/202101/760268_MAP9TRKNN5SH4UQ.png)

## 参考链接

- <mark class="encrypted">a35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5N6h3&6G2L8$3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4x3$3y4F1L8%4c8W2i4K6u0r3j5%4m8H3i4K6u0V1M7X3q4F1k6q4)9J5k6s2y4J5j5h3&6V1i4K6u0W2K9s2c8E0L8l9`.`.</mark>

- <mark class="encrypted">63eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4D9L8r3g2F1L8r3W2F1M7Y4g2A6i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2z5e0j5@1x3o6b7$3</mark>
