
```
// StoreStore barrier required after complete initialization

// (headers + content zeroing), before the object may escape.

membar(MacroAssembler::StoreStore, tmp1);
```

对于一些设置为 `public` 的对象，这是必要的：如果代码中有错误，而这一对象被设置为 `public`，你会想要看到它内部的 fields（语言特性允许你这样做）的默认值或是开发者预设好的值，而不是一个随机的值，虚拟机希望获取到一个正确的 `header`。x86 架构的内存模型更加强大，它不需要这一指令，所以我们看到的是 ARM 的实现。

## 在实践中检验

> 注意上述代码可能有 bug，我只是用这些代码来证明文章的正确性，并没有实际运行过

到目前为止，所有的一切看起来都棒极了：我们阅读了源代码，找到了一些有意思的东西，但是对编译器实际做的情况我们并不清楚，也许我们的理解都是错误的。再回头看看 `PrintAssembly` 中 `new Long(1023)` 的相关代码：

```
0x0000000105eb7b3e: mov 0x60(%r15),%rax
0x0000000105eb7b42: mov %rax,%r10
0x0000000105eb7b45: add $ 0x18,% r10; Allotsiruem 24 bytes: 8 byte header,
; 4 bytes pointer to a class,
; 4 bytes for alignment,
; 8 bytes on a long field
0x0000000105eb7b49: cmp 0x70(%r15),%r10
0x0000000105eb7b4d: jae 0x0000000105eb7bb5
0x0000000105eb7b4f: mov %r10,0x60(%r15)
0x0000000105eb7b53: prefetchnta 0xc0(%r10) ; prefetch
0x0000000105eb7b5b: movq $ 0x1, (% rax); Set the title
0x0000000105eb7b62: movl $ 0xf80022ab, 0x8 (% rax); We set the pointer to the Long class
0x0000000105eb7b69: mov %r12d,0xc(%rax)
0x0000000105eb7b6d: movq $ 0x3ff, 0x10 (% rax); We put 1023 in the object field
```

棒极了，看起来确实和我们预期的一样。

总而言之，创建一个新的对象的过程如下所示：

1. 首先尝试在 `TLAB` 中分配对象。
2. 如果 `TLAB` 中空间不够，要么从 `eden` 区中再分配一块 `TLAB`，要么直接在 `eden` 区创建该对象，整个过程是一个原子操作。
3. 如果 `eden` 区空间也不够，就会进行垃圾回收。
4. 如果垃圾回收之后，空间还是不够，则会尝试在老年代进行分配。
5. 如果还是不够，则会抛出 `OOM(out of memory)` 异常。
6. 对象创建成功后，设置对象的 `header`，并调用它的构造方法。

至此，理论部分就结束了，接下来开始实战：是否上述操作能够提高性能，是否需要预载入，是否 `TLAB` 的大小可以通过一些值进行控制。

## 实验

现在我们已经知道了有多少个对象会被创建，哪一个参数能够控制这一过程，是时候在实战在校验这些理论的正确性了。首先让我们写一个简单的 benchmark，它仅仅在几个线程中创建 `java.lang.Object`，以及修改一些 `JVM` 参数。

这一实验运行在 `Java 1.8.0_121`，Debian 3.16, Intel Xeon X5675 的实验环境中。横坐标是线程的数量，纵坐标是每微秒进行的空间分配的次数。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_3DTUHZCEP96KC4H.png)

结果正如预期的一样：

默认情况下，分配次数基本上和线程数成正比，这和我们预期的一样。随着线程数的增长，斜率稍微降低了一点，这很正常：如果在两次分配之间做一些操作（比如，使用 `Blakehole #consume CPU`），那么分配的碎片会减少，比率会重新回到线性。

关闭预载入使得所有的分配操作都有一些变慢。在这次 benchmark 中，我们仅仅使用分配来增加 `JVM` 的工作负荷，在实际中程序的实现是多种多样的，所以我们不能使用实际程序来总结出一个特定的结论。最后，虽说没有特定的方法，但你仍然可以关掉预载入以验证这一操作对你的程序的影响。

在 `TLAB` 分配停止之后，情况变得很糟糕：每次调用 `JIT->JVM` 的开销基本上都达到了原来的两倍半，随着线程数的增长，对单一指针的竞争也会增长，结果也不具有普适性。

最后，我们来看看对析构的影响，比较 `eden` 区对象的析构的性能：

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_QRPG2HKNHC7RBQK.png)

相较于快速分配，性能下降了一至两个数量级

## 结论

为了又快又好地分配一个对象，`JVM` 做了大量的的工作，`TLAB` 是进行分配工作的主力。同时，`TLAB` 还会和垃圾回收紧密合作：在垃圾回收将内存释放之后，分配就能够自由地进行了。

这些知识有用吗？可能有，也可能没有。不过它用助于你理解你的机器是内部的内存是怎么分布的，以及它使用了哪些好的方法。

原文链接：[398K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8i4g2E0j5X3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6B7j5i4k6S2i4K6u0r3K9r3!0%4i4K6u0V1k6r3!0W2M7#2)9J5k6r3A6$3L8g2)9J5k6r3q4D9L8r3!0U0j5i4c8W2i4K6u0V1L8$3u0B7k6h3y4@1M7#2)9J5y4e0y4r3i4K6u0r3](https://bbs.kanxue.com/elink@2fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8i4g2E0j5X3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6B7j5i4k6S2i4K6u0r3K9r3!0%4i4K6u0V1k6r3!0W2M7#2)9J5k6r3A6$3L8g2)9J5k6r3q4D9L8r3!0U0j5i4c8W2i4K6u0V1L8$3u0B7k6h3y4@1M7#2)9J5y4e0y4r3i4K6u0r3)

编译： 看雪翻译小组 梦野间

校对： 看雪翻译小组 lumou

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-1-28 22:15 被 kanxue 编辑，原因：
