
# [原创]r4j0x00发布的V8的1DAY漏洞分析

## 一．漏洞简介

漏洞来源<mark class="encrypted">1e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3M7U0c8B7x3s2R3H3x3q4)9J5c8Y4y4@1j5i4c8#2M7#2)9J5c8U0p5J5z5e0R3$3z5o6t1#2y4K6l9@1y4o6R3$3y4K6b7^5x3e0M7`.</mark>

作者发布了自己的exploit以及google的修改记录：

<mark class="encrypted">5c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5y4r3Z5H3P5o6l9H3i4K6u0r3k6i4S2H3L8r3!0A6N6s2y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3j5$3S2J5L8$3#2W2i4K6u0V1k6i4S2H3L8r3!0A6N6l9`.`.</mark>

<mark class="encrypted">1beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3z5q4)9J5c8Y4j5^5i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3z5o6g2T1j5K6q4T1x3r3y4S2j5U0x3I4j5$3x3H3y4U0c8W2k6X3x3$3y4h3f1H3y4h3q4V1j5U0R3I4k6X3g2W2z5o6p5@1x3U0j5I4j5W2)9J5x3$3c8A6k6X3k6Q4x3X3b7J5k6e0u0U0y4e0j5@1y4h3b7^5y4$3c8S2j5X3g2U0k6o6x3%4z5e0y4T1x3h3j5I4x3o6x3H3x3o6V1%4y4l9`.`.</mark>

## 二．漏洞初步分析

漏洞的核心触发点为这个被优化的函数：

```js
function trigger(array) {
  var x = array.length;
  x -= 67108861;
  x = Math.max(x, 0);
  x *= 6;
  x -= 5;
  x = Math.max(x, 0);
  let corrupting_array = [0.1, 0.1];
  let corrupted_array = [0.1];
  corrupting_array[x] = length_as_double;
  return [corrupting_array, corrupted_array];
}
```

字面意思可以推测出`corrupting_array`越界踩踏了`corrupted_array`，于是来调试证明一下。

在优化完成后用`%DebugPrint`先后打印出`corrupting_array`和`corrupted_array`的地址：

```
DebugPrint: 0x20859364f19: [JSArray]
- map: 0x0208082818b9 <Map(HOLEY_DOUBLE_ELEMENTS)> [FastProperties]
- prototype: 0x020808248f7d <JSArray[0]>
- elements: 0x020859364f01 <FixedDoubleArray[2]> [HOLEY_DOUBLE_ELEMENTS]
..................
..................
DebugPrint: 0x20859364f39: [JSArray]
- map: 0x020808281891 <Map(PACKED_DOUBLE_ELEMENTS)> [FastProperties]
- prototype: 0x020808248f7d <JSArray[0]>
- elements:
```

可以看到`corrupting_array`的地址正好在`corrupted_array`前面一点，gdb打印出内存：

![图片描述](upload/attach/202010/782331_3G43K62P7RM5C8D.png)

首先，`corrupting_array`首地址偏移8的位置，也就是最右侧上面那个红框里，`59364f01`，最后个bit为1，代表这是个指针，和左侧红框里isolate指针`0x20800000000`相加再减去那个1，指向`0x20859364f00`。这个位置存放的就是`elements`，这在`DebugPrint`的打印里也可以找到，同理可以对应找到`map`，`prototype`等信息。

而`59364f01`左边，也就是`corrupting_array`首地址偏移12的位置，存放的是`corrupting_array`数组的大小`0x4`，这个数据最后个bit是0，因此是个smi，也就是整形数据，需要右移1bit，因此真正的大小是2，这也的确是`corrupting_array`最初定义时的大小。

关于V8的isolate，指针和SMI知识，可查阅其它文档，但需要说下的是，这部分貌似最近有了变化，因为从其它文档看，指针和SMI都占位了64bit，但现在却缩到了32bit，估计是为了节约内存吧。

接下来再看`elements`，也就是刚才计算出来的`0x20859364f00`，首地址存储的是`elements`的属性信息，略过不关注，再往后就是连续的两个浮点数`0x3fb999999999999a`，也就是真正的数组数据，两个`0.1`。这样`corrupting_array`的解析就基本完成了。

这时，再看看`corrupting_array`是怎么越界的，从`trigger`函数里分析，`x`的值计算出来必定是7，也就是`corrupting_array`的大小理应扩展为8，但实际上从调试来看，它的大小只有2，那么越界写就这样产生了，右边第2个红框处就是越界写的位置，这个位置正好是`corrupted_array`的首地址偏移8的位置，也就是存放它数组大小以及`elements`首指针的位置。

这样的话，`corrupted_array`数据的数组访问范围就变成了从isolate首地址+8开始的（`0x24242424/2`）这么大的空间，基本涵盖了整个isolate，数据泄露就这么产生了。

要了解这里面的代码流程，可以从`DebugPrint`的实现入手，断点断在`JSArray::JSArrayPrint`，一步步看下去。

后面的利用流程大致说下，在`search_space`中，作者把isolate中可能的空间范围分段搜索他感兴趣的数据，之所以分段，是因为这段内存并不是连续可读的，这里不同版本的浏览器是不一样的，甚至同一版本浏览器每次启动都可能发生变化，这里需要自行调整出一个合适适度的范围。

此外，由于`corrupted_array`是个浮点数组，只能以U64的方式进行读取，而内存数据有可能并不是刚好按作者想象的16进制对齐，需要自己设法调整。

## 三．漏洞原理分析

显然，和以前大多数漏洞一样，这个越界之所以产生，是优化的过程出了问题。但在分析优化流程之前，首先还是看看google给出的修改方案，关键点在这里：

![图片描述](upload/attach/202010/782331_7DA57APQQ3MTTB6.png)

这个`xx.tq`代码将会在`xx-tq-csa.cc`中被翻译成一段冗长的C++代码。

可以看出，这里将会对数组的最大值做判断，超过了就会报OOM。这个地方是否就是针对优化流程做的修改呢？

找一个chrome85版本做实验，这个版本已经修改了这个bug，通过插入日志可以发现，根本走不到`trigger`函数，在前面的`giant_array.splice`那一步就发生了OOM。

（图略）

再分析前面的js代码，可以发现`splice`以及前面的流程，强行把`giant_array`数组撑破了，超出了最大值`67108862`，达到了`67108863`。因此fix的代码才会阻止这个步骤。

那么这和之后的`trigger`优化流程有什么联系呢？

感谢谷歌的turbolizer优化流程分析工具，使得这个问题的分析变得简单了，甚至都不需要去看代码（这貌似不是什么好事）。

![图片描述](upload/attach/202010/782331_FM9HP66DRYFWFKX.png)

这个“节点之海”（sea of nodes）看起来深不可测，但如果只针对这个问题，还是比较好理解的。

绿色部分代表了优化后机器指令，20，38，40分别是减，乘，减，正好对应`trigger`函数里的三次算术操作。

蓝色的97，104对应`trigger`函数里调用`Math.max`的操作。

所有的操作下面都有个Range，代表这步做完后，这个值可能所在的范围，“节点之海”可以根据这个Range决定是否优化掉115节点，也就是边界检查这步，它检查是否超出了数组的最大范围。

如果最后计算出来的Range都没有超出数组的范围，那显然是可以不检查的，这个逻辑并没有问题。

但问题在于最开始的87节点，它推测的是`var x = array.length`的范围，按照数组的最大值，这理所当然被推测成了（0，67108862），但却没想到实际传进来的是个非法的被撑破的数组，它的大小刚好超过了67108862，是67108863。

所以，失去边界检查后，`corrupting_array[x] = length_as_double;`也就没人能阻止越界了。

## 四．参考文档：

1.[原创] 手把手教你入门V8漏洞利用：

https://bbs.pediy.com/thread-258431.htm

2.TurboFan TechTalk presentation

<mark class="encrypted">4e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0W2M7$3g2F1N6r3q4@1K9h3!0F1i4K6u0r3k6q4)9J5c8U0q4K6e0@1g2r3y4p5#2D9c8U0N6x3k6f1)9%4N6i4q4Q4x3X3c8#2g2r3S2v1f1%4g2D9d9X3I4f1K9q4)9J5k6q4)9J5k6s2N6Y4e0r3g2S2g2X3W2T1M7$3u0T1x3%4c8U0i4K6u0r3k6h3c8A6N6q4)9J5x3%4y4D9K9h3c8W2i4K6y4p5K9h3c8Q4x3X3g2Y4y4e0b7&6z5h3t1&6j5K6b7J5i4K6g2X3x3o6p5%4y4K6R3`.</mark>

3.introduction-to-turbofan

<mark class="encrypted">b1eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3q4J5i4K6u0V1k6g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5X3I4G2k6#2)9J5c8U0t1H3x3e0W2Q4x3V1j5H3x3g2)9J5c8U0t1^5i4K6u0r3K9h3&6@1M7X3!0V1N6h3y4@1K9h3!0F1i4K6u0V1N6r3!0Q4x3X3c8@1N6i4u0T1L8$3k6S2L8W2)9J5c8R3`.`.</mark>

4.从漏洞利用角度介绍Chrome的V8安全研究

<mark class="encrypted">d02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1@1K9r3!0#2i4K6u0W2j5$3!0E0i4K6u0r3M7r3!0K6N6s2y4Q4x3V1j5J5x3g2V1I4</mark>
