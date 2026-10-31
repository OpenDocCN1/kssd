
# [原创]IO-FILE 中的 stdin 介绍

本文为科普文，便于新手更好的了解 `IO-FILE` 中的 `stdin`。

网上的很多教程对 `IO-FILE` 其实讲的都比较详细了，尤其是[这里](<https://bbs.kanxue.com/elink@4fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5%4y4K6V1#2z5l9`.`.)>)，从源码的角度看 `fread()` 函数，讲的非常透彻。本文换一个角度，从几个关键指针的角度来看 `IO-FILE` 中的 `stdin`，让大家搞明白 `IO-FILE` 中的 `stdin` 大概是怎么回事。`IO-FILE` 中的 `stdin`，一些常见的输入函数都会涉及，如 `scanf()`、`fread()` 函数，这些函数最关键的地方在于有一块输入缓冲区，用户输入的数据会先放到输入缓冲区中，函数再根据需要从缓冲区内读取数据（如 `fread()` 的 `size` 大小，`scanf()` 中的格式化字符串等）。

![](https://bbs.kanxue.com/upload/attach/202009/853476_WRG5PQSERHSG39F.png)

直接看一段代码便于理解：

```
#include<unistd.h>
#include<stdio.h>

int main() {
    char buf[10];
    int number1;
    int number2;
    int number3;
    int number4;
    int number5;

    fread(buf, 1, 10, stdin);
    puts("buf:");
    write(1, buf, 10);
    printf("\n");
    puts("----------");

    scanf("%d", &number1);
    printf("number1:%d\n", number1);
    printf("----------------\n");

    scanf("%d", &number2);
    printf("number2:%d\n", number2);
    printf("----------------\n");

    scanf("%d", &number3);
    printf("number3:%d\n", number3);
    printf("----------------\n");

    getc(stdin);
    scanf("%d", &number4);
    printf("number4:%d\n", number4);

    getc(stdin);
    scanf("%d", &number5);
    printf("number5:%d\n", number5);
}
```

程序首先调用 `fread()` 函数，在这个函数中，程序会调用系统函数读取终端输入。在读取终端输入之前，系统会开辟一个输入缓冲区，`_IO_buf_base` 指向这个输入缓冲区的起始位置，`_IO_buf_end` 指向输入缓冲区的末尾。此时 `_IO_read_ptr` 和 `_IO_read_end` 都为 0。然后系统从终端中读入数据，我这边输入的数据是 `aaaaaaaaaa10a20`，来看一下执行完 `fread()` 之后指针的变化。

![](https://bbs.kanxue.com/upload/attach/202009/853476_DVA2XPP3JVAD4N8.png)

可以看出，`_IO_read_ptr` 指向的是缓冲区内还未被读取的数据起始位置。`_IO_read_end` 指向的是缓冲区内还未被读取的数据末尾位置。`_IO_read_base` 指向读入数据的起始位置。

程序继续执行，执行第一个 `scanf()` 函数，此时会将缓冲区内的数字 10 读入。`_IO_read_ptr` 向后移动，继续指向缓冲区内还未被读取的数据起始位置。

![](https://bbs.kanxue.com/upload/attach/202009/853476_CQ2NZ7RFWYQHBMM.png)

![](https://bbs.kanxue.com/upload/attach/202009/853476_WM4WBHV929K2RCU.png)

此时程序继续执行，执行第二个 `scanf()` 函数，但是由于此时缓冲区内存放的是一个字符 `a`，因此 `scanf()` 并不会将其读取（读取失败情况下，`scanf()` 函数会返回 0），因此 `_IO_read_ptr` 并不会改变。

![](https://bbs.kanxue.com/upload/attach/202009/853476_4VXMF5J49NEKXHP.png)

![](https://bbs.kanxue.com/upload/attach/202009/853476_6QZCH8HX5B8P32V.png)

继续执行程序，执行第三个 `scanf()` 函数，显然由于 `_IO_read_ptr` 仍然指向字符 `a`，因此依然会读取失败。

![](https://bbs.kanxue.com/upload/attach/202009/853476_R9GY4QMMZJ5JDXZ.png)

![](https://bbs.kanxue.com/upload/attach/202009/853476_8A39QUZWYRFD5RZ.png)

继续执行程序，执行 `getc(stdin)`，该函数会从缓冲区中读一个字节，因此会将 `_IO_read_ptr` 加 1，执行完该函数之后：

![](https://bbs.kanxue.com/upload/attach/202009/853476_2PWXERNBZGZFEC9.png)

程序继续执行，执行第四个 `scanf()`，由于此时 `_IO_read_ptr` 又指向了一个数字，因此 `scanf()` 将该数据读了出来。

![](https://bbs.kanxue.com/upload/attach/202009/853476_E85QGCPXN8WHZC2.png)

此时 `_IO_read_ptr` 指向换行符，我们再次调用 `getc(stdin)` 让其加一，此时 `_IO_read_ptr` 便与 `_IO_read_end` 相等了。

![](https://bbs.kanxue.com/upload/attach/202009/853476_ZP44N8X2QEFWAP5.png)

也就是说，缓冲区内的数据都被我们读完了，那么我们调用第五次 `scanf()` 的时候，便又要从终端读入数据了。

这里我输入 `666bbbbbbbb`，第五次 `scanf()` 结束之后的指针情况：

![](https://bbs.kanxue.com/upload/attach/202009/853476_78KR5EJ59NR5NKB.png)

可以看到数据重新从输入缓冲区的起始位置开始存放，但是缓冲区内的老数据并未清零，只是被覆盖了。

来一个非调试版的：

![](https://bbs.kanxue.com/upload/attach/202009/853476_M43NSZU9QGEFF9Z.png)

## 关于任意地址写

当我们篡改了 `_IO_buf_base` 与 `_IO_buf_end`，将其改为目标地址开始与结尾时，那么我们下一次的终端输入就会被写到目标地址处了。

写此文的灵感主要来源于做 whctf2017 的 stackoverflow 的时候，当我们篡改了 `_IO_buf_base` 与 `_IO_buf_end` 到 `malloc_hook` 处的时候，还需要经过数次循环（多次调用 `sub_4008c8()`），原因就是 `_IO_read_ptr` 还小于 `_IO_read_end`，此时如果不调用 `getc(stdin)` 的话，`_IO_read_ptr` 是没法等于 `_IO_read_end`，那么程序也就不会从终端去读取输入了。当然有细心的人会发现这里存在 `scanf()` 函数，`scanf()` 函数其实也是可以从缓冲区内读取输入的，但是由于我们缓冲区内的数据都是不可见字符，因此 `scanf()` 是会读取失败的，因此并不能增加 `_IO_read_ptr`。

![](https://bbs.kanxue.com/upload/attach/202009/853476_57TCH6Z29TPRCKR.png)

对 `stdin` 的介绍就到这里了，写的不对的地方，欢迎大家指正。
