
```
|pack_method 1 //pack_method, D R H W
|unpacked_size 4 //may be different than the value in BIN index, often 4 byte asign
```

## 0x2 解包算法定位

上面文件数据结构分析，我们发现有是否压缩的flag和压缩后和解压的体积，再加上来查看`*.ARC`文件，内部比较紧凑，因此判定是压缩或者加密了。我们观察得到`pack_method`字段有"D,R,H,W"这四个很醒目的标识符，这里可以作为突破口。必须用ida6.8安装[VitaLoaderIDA](https://bbs.kanxue.com/elink@e27K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6q4L8r3g2U0N6s2u0&6i4K6u0r3g2X3W2@1j5f1I4G2j5h3c8W2M7V1W2p5b7b7`.`.)载入`eboot.bin`， `search immediate`搜索`H`等标志位即可找到。

同时我们注意到`GARBRO`源码中已经有了框架分析了一部分，但是最难的一个函数`UnpackH`仍是`throw new NotImplementedException();`我们需要自己分析一下补全。

![unpack_flag](https://bbs.kanxue.com/upload/attach/202011/617776_VP3Q5XPDBSBAVMW.jpg)

同时，我们在定位处上下翻翻，还能看到`huffman`和`runleng`的字符串，因此压缩算法很可能是基于这两种算法的魔改。

![huffman_decode](https://bbs.kanxue.com/upload/attach/202011/617776_57HRUFYBCJRKFBM.jpg)

## 0x3 解包算法伪代码分析

下面以`UnpackH`函数为例（其他函数`UnpackD`, `UnpackW`等也同理），来谈谈如何分析。伪代码层面上的分析一般有下列方法：

- 猜每个变量的含义并改名（字符串缓冲区，当前位置指针等比较容易识别）
- 修改到合适数据类型，有些是数组和结构的，添加结构体看着舒服一些。还有一些情况，地址显示的是负号，我们需要手动`invert signed`。
- 合并相同变量（注意不能乱合并，尤其是临时赋值的）

用上面方法初步分析整理，可得到如下的伪代码。
