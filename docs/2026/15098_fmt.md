
# 企业版代码分析记录 之 反调试和资源一致性校验

## 0x00 序1
随着移动安全越来越火，各种调试工具也都层出不穷，最近脱360壳的人满天飞，我今天来调一下棒棒企业版。

## 0x00 序2
现在脱壳动态调试，静态分析，基本上都是只使用`ida`，但很少有人结合`ida_python`动态调试，使用`ida_python`能很大的提高工作效率，就比如脱壳需要过反调试，或者寻找`dump`点，所以不是打开一次`ida`，附加一次进程就能脱下壳的，所以笔者决定结合`ida_python`将调试中使用到的小技巧总结归纳一下。

## 0x01 过反调试
想要脱壳第一步就需要过反调试，众所周知，反调试有好多种，最基本的一个是读取`proc`文件状态，所以肯定会调用`fopen`，所以使用`ida_python`下一个断点 `AddBpt(LocByName('fopen'))`；一路`f9`走到这里

![图片描述](upload/attach/202002/630232_zgy7fgl5ug69k79.png)

![图片描述](upload/attach/202002/630232_vf3587xrbamd5bz.png)

![图片描述](upload/attach/202002/630232_zo1dwbflz451d18.png)

然后到`lr`的地方

![图片描述](upload/attach/202002/630232_lh9ewtvkzwrea54.png)

`f5`

![图片描述](upload/attach/202002/630232_8ri4qgwtp14ew8r.png)

跟进来好多函数和变量都看不懂什么意思，分析之后使用`ida_python`重命名，就好分析了

![图片描述](upload/attach/202002/630232_1gh4e4epzee5v68.png)

使用之后是不是豁然开朗

![图片描述](upload/attach/202002/630232_2kj6ls0dmoyt6da.png)

![图片描述](upload/attach/202002/630232_g3ynn8qitk82be7.png)

这个函数是检测了这些东西然后判断到正在调试调试，返回`1`

![图片描述](upload/attach/202002/630232_rny67t3c33c71u4.png)

那我们直接把这个返回值改成`0`

![图片描述](upload/attach/202002/630232_rp61u2v5ab3b7rn.png)

![图片描述](upload/attach/202002/630232_cq92zsku64u7wzb.png)

走出这个函数来到这里

![图片描述](upload/attach/202002/630232_pitlbtx70pbaxxw.png)

这两个函数都是调试检测如果检测到调试会走到下面的异常函数

![图片描述](upload/attach/202002/630232_vqanj1f6kcp377f.png)

程序就会挂掉

同样修改返回值即可。

## 0x02 资源完整性代码分析
过掉反调试，断点断在 `pthread_create` 一路`f9`

![图片描述](upload/attach/202002/630232_7c2vezz4a96skfq.png)

确定`lr`是`libDexHelper.so`，然后在`r2`和`lr`的地址都下断点，继续`f9`

![图片描述](upload/attach/202002/630232_ilu1mbfbkq7botu.png)

走到`lr`以后把主线程暂停

![图片描述](upload/attach/202002/630232_6j4vdj7m05dkchl.png)

继续`f9`就可以到资源完整性检测的线程

![图片描述](upload/attach/202002/630232_wq81ywlomuzqgsd.png)

这个函数会传入一个`apk`的安装路径。

然后她会分布操作

1. 读取 `assets/meta-data/manifest.mf` 内容

![图片描述](upload/attach/202002/630232_32okzs6defupqi3.png)

2. 读取 `META-INF/MANIFEST.MF` 文件内容

![图片描述](upload/attach/202002/630232_mwop689hmfyclpz.png)

3. 这个函数就是拿本地预存的值和读取`apk`签名文件里面的`hash`比较

![图片描述](upload/attach/202002/630232_6ye1tsc2h0857cv.png)

4. 进入函数分析代码：它先通过`META-INF/MANIFEST.MF`把文件名和`hash` 存到一个链表

5. 他把`manifest.mf`这个文件 读成下面的结构。存到数组

```
i4

Kg==

RYeUYd+yP3JShXAmdbN3b8xD1WE=

opzvAD5X42gY13rlJ8HQWfchSj4=

yeA+pIPECBn2xj16EBDWyU9mjwA=

DoiscxMD7gK0Al1/5z93eaQ4N7A=

C+KBdBCnBWNM4LIAYy9d7sJhBek=
```

6. 读取这个文件里面记录的每个文件的`sha1`的前`4`个字节拼接做一次`base64`再做一次`sha1`

![图片描述](upload/attach/202002/630232_io19bgin1c7w20p.png)

然后和`manifest.mf` 里面的最后一组比较

![图片描述](upload/attach/202002/630232_9bfamfcz4xdln4o.png)

到此为止梆梆企业版的反调试，和文件完整性校验已经分析完毕。

其实`ida_python` 还能做好多事情，比如把一段代码设置成函数，加注释，设置条件断点，打印数据，我这里就不一一列举了。

后续有时间在用`ida_pyton`去分析加壳代码去脱壳。。。。
