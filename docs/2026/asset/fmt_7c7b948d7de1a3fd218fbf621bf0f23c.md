
# [翻译][注意]JVM 虚拟机创建对象的过程分析（下）

[上篇见这](https://bbs.pediy.com/thread-255308.htm)

## C1中的内存分配

从TLAB的角度来思考这个问题。为了不跑偏，我们深入挖掘源码，看看TLAB在快、慢和非常慢三种场景下的内存分配。你需要看看`operator new`是怎么编译的，为了少死点脑细胞，我们来看看客户端的编译器代码（C1）：它比服务端的更简单更容易理解，麻雀虽小，五脏俱全，因为`new`在`java`中非常常见，所以它里面有非常多的优化。

我们对两个方法感兴趣：

- `C1_MacroAssembler :: allocate_object`。包含了TLAB中对象分配和初始化相关的代码
- `Runtime1 :: generate_code_for`。当分配操作不能在短时间内完成时调用。

JVM如何判断对象是否总是能在短时间内被创建值得我们关注，通过寻找方法的调用处（find usages）,我们找到了`instanceKlass.hpp`中的这个注释：

```
// This bit is initialized in classFileParser.cpp.
// It is false under any of the following conditions:
// - the class is abstract (including any interface)
// - the class has a finalizer (if !RegisterFinalizersAtInit)
// - the class size is larger than FastAllocateSizeLimit
// - the class is java/lang/Class, which cannot be allocated directly
bool can_be_fastpath_allocated() const {
return !layout_helper_needs_slow_path(layout_helper());
}
```

显而易见，大对象（超过128kB）和具有`finalize`方法的类在JVM中分配会非常耗时（猜一猜 - 抽象类的分配代码在哪？）。让我们记住这一点，然后返回去看看内存分配进程。

`tlab_allocate`会尝试快速分配对象，它的代码我们刚好在`PrintAssembly`中看到过了。当它结束之后，我们的内存分配也就结束了，之后转入对象的初始化操作。

`tlab_refill` 会尝试分配一个新的TLAB。我们进行了一个有趣的测试，这一方法会确定是分配一个新的TLAB（抛弃旧的）还是保留旧的TLAB，直接在`eden`区分配一个对象

```
// Retain tlab and allocate object in shared space if
// the amount free in the tlab is too large to discard.
cmpptr(t1, Address(thread_reg, in_bytes(JavaThread::tlab_refill_waste_limit_offset())));
jcc(Assembler::lessEqual, discard_tlab);
```

`tlab_refill_waste_limit` 负责控制TLABs的大小，以免我们在为对象分配内存时造成过多的浪费。默认情况下，它的值是当前TLAB大小的1.5%（当然，也可以通过设置`TLABRefillWasteFraction`的值来确定这个百分比, `TLABRefillWasteFraction`的默认值是64，TLAB的大小除以这一参数值即为浪费的百分比）。在每次进行慢分配的时候，这一限制都会生效以避免分配失败。它的值在每次GC循环之后会被重置。

`eden_allocate`- 会尝试在`eden`区分配内存(对象或是TLAB)。这一部分与TLAB中的分配非常相似：我们会检查是否还有空闲空间，如果有，使用原子操作（`cmpxchg lock clause`）获取内存，如果没有，则进行慢分配。`eden`区内的分配不是`wait-free`：如果两个线程同时在`eden`区进行内存分配，有可能其中一个的分配并不起效，不得不重新进行分配。

## JVM upcall

如果你不能在`eden`区分配内存，可以尝试使用`InstanceKlass :: allocate_instance`方法。在调用这一方法之前，需要做大量的辅助工作。为GC设置特殊的结构体，创建必要的栈桢以适应[调用约定](<a href="elink@8d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5z5o6k6Q4y4h3k6U0j5h3I4D9K9h3&6Y4i4K6g2X3j5$3!0F1N6X3g2F1N6r3W2G2L8Y4x3`."><mark class="encrypted">88bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5z5o6k6Q4y4h3k6U0j5h3I4D9K9h3&6Y4i4K6g2X3j5$3!0F1N6X3g2F1N6r3W2G2L8Y4x3`.</mark></a>)，因此这一操作并不快。它里面有大量的代码，如果没有一份简单的描述的话，看起来会非常吃力，所以我不会详细讲解里面的代码，只会给出一个大概的框架：

首先，JVM会尝试通过指定的接口为当前的垃圾回收分配内存。在调用发生的时候也是一样的，正如上面所说的：首先尝试在TLAB中进行分配，然后尝试在堆中分配TLAB并创建对象。如果失败了，就会调用垃圾回收。还有些其他地方涉及到GC overhead limt exceed这一`error`，包括各种与GC有关的通知，日志和与内存分配无关的检查。

如果垃圾回收之后空间还是不够，就会尝试在老年代进行分配（这取决于选择的GC算法），在失败的时候，再进行一次垃圾回收，并尝试创建对象，如果不起作用，就会抛出`OutOfMemoryError`。

当对象被成功创建后，会检查该对象是否有`finzlize`方法，如果是的话，就会注册它的`Finilizer`方法（这下你就明白了为什么这个类在标准库中，但从来没有显式调用过）。这个方法本身在很久以前就写好了：`Finilizer`对象创建为一个全局变量，并添加到链表中（这意味着它后面会被析构并回收）。这刚好证明了JVM的无条件调用以及这条忠告：“不要使用`finalize`方法，尽管你非常想要用”

最后，我们几乎已经知道了关于分配的所有事情：对象被快速分配，TLAB快速填充，对象有时候马上被分配到`eden`区，有时候他们会触发JVM中的一些耗时的调用。

## 监视慢分配

我把内存分配的时候相关的所有数据记录下来（慢分配，`refilles`的平均数，分配线程的数量，内部碎片造成的损失），那么用这些信息我们可以干什么呢？

![图片描述](upload/attach/202001/93_CVCXW5E3FSS93TF.png)

上述是一些性能相关的数据，最终存储到 `hsperfdata` 文件中，你可以使用`jcmd`或`sun.jvmstat.monitor` API编程来查看它。

没有其他方法能获取到这些信息，但如果你用的是Oracle JDK，使用JFR，你就可以在栈追踪区域看到它。（这一`private` API在OpenJDK中不可用）。

这些信息重要吗？在大多数情况下，它不重要，但在某些情况下它很重要。比如，Twitter JVM团队写了一份[报告](elink@2a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5e0e0W2G2x3f1I4h3k6V1N6H3x3V1q4Q4x3U0k6@1i4K6y4p5x3U0b7H3x3l9`.`.`)，在慢分配中，调整必要的参数后，能够降低几个百分点的时间开销。

## 预载入

当我们查看代码的时候，经常能看到一些预载入的额外检查，这些我通常都手动忽略了。

> 预载入是一种提高性能的技术，它把一些我们马上要（但不是现在）读取的数据加载进处理器缓存中。在硬件层面，当处理器认为下一部分数据是顺序排列的，就会载入它。在软件层面，（编译器，虚拟机的）程序员生成一些特殊的指令给处理器一些提示，告诉它把某个地址的内存放进缓存中效果更好。

在`hotspot`中，预载入是一个C2专有的优化，所以在C1的代码中我们没看到它。优化的具体过程是这样的：当在TLAB中分配内存时，生成一条指令，用于要分配的内存位置后面的数据载入缓存。平均而言，Java应用会进行很多很多次分配，所以，把后续的数据预告载入缓存是个不错的主意：下一次创建对象的时候，不需要再等待，因为它已经被加载进缓存中了。

![图片描述](upload/attach/202001/93_9TVKKSN28TAGSBQ.png)

根据`**AllocatePrefetchStyle**`选项的不同，预载入有几种不同的模式：你可以在每次内存分配之后进行一次预载入，或是进行多次预载入。此外，可以通过`AllocatePrefetchStyle`改变预载入时的指令：你可以只把数据加载进L1缓存中（例如，当你加载完之后马上就不再使用的数据），只加载进L3中，或是一次性加载加入：这些可选项取决于处理器的架构，对应的值和指令可以在这个[.ad文件](elink@bbdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2Y4i4K6u0W2L8%4m8W2L8X3A6V1K9#2)9J5k6h3A6S2N6X3q4Q4x3X3g2F1k6i4c8Q4x3V1k6B7k6r3D9&6i4K6u0r3K9X3c8C8z5g2)9J5c8X3S2G2N6s2y4H3L8%4c8Q4x3V1k6X3K9h3I4W2i4K6u0r3z5h3c8S2j5h3y4W2j5U0k6U0y4U0c8U0i4K6u0r3M7%4u0U0i4K6u0r3j5%4m8#2i4K6u0r3P5o6R3$3i4K6u0r3N6X3#2Q4x3V1k6^5z5o6k6Q4y4h3j5$3y4q4)9J5k6h3q4V1i4K6t1K6L8o6f1#2z5e0x3`.)中找到。

在生产中，我们不建议你去碰这些值，除非你是JVM工程师，想要在SPECjbb-benchmark中拿个比较好的名次，或是写一些性能非常高的Java代码，你进行的所有修改都经过了再三地确认（那你可能不需要看这篇文章了，因为你几乎对这个已经有深入的了解了）

## 初始化

在内存分配的时候，会清除所有东西，只会留下一些，在调用对象构造函数之前，需要初始化的数据。在C1-编译器中也可以看到这部分内容，但是在ARM中，代码更简洁，也有一些更有趣的东西。

请求的方法是：

`C1_MacroAssembler :: initialize_object`

这个方法并不复杂

首先，设置对象的`header`。`header`由两部分组成 - [mark word](elink@affK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2Y4i4K6u0W2L8%4m8W2L8X3A6V1K9#2)9J5k6h3A6S2N6X3q4Q4x3X3g2F1k6i4c8Q4x3V1k6B7k6r3D9&6i4K6u0r3K9X3c8C8z5g2)9J5c8X3S2G2N6s2y4H3L8%4c8Q4x3V1k6X3K9h3I4W2i4K6u0r3z5h3c8S2j5h3y4W2j5U0k6U0y4U0c8U0i4K6u0r3M7%4u0U0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6$3L8g2)9J5c8X3!0G2M7s2y4Q4x3V1k6E0j5i4u0C8e0$3!0H3i4K6u0W2K9s2m8H3i4K6t1K6L8o6x3H3">mark word</a>，其中包括关于锁，身份`hash`码（或偏向锁），垃圾回收以及指向对象类的类指针，它的同名`native`类表示它在`metaspace`中，使用它可以获取到`java.lang.class`。

![图片描述](upload/attach/202001/93_JDEDT666C9FUZSS.png)

这一指向类的指针通常是32位的，并不是64位。它的最小可能大小12字节（强制对齐会使它增长在16字节）

如果`**ZeroTLAB**`选项没有启用，所有的内存会被清空。默认情况下它是关闭的。将一大片内存区域清0也会导致缓存清空，更有效的释放一片内存的方法是，使用一小段马上会被覆写的内存来进行释放。此外，聪明的C2编译器只会做必要的工作，也不会清空内存，而是在后面紧跟一个设计好的参数。这是另一种做法。

最后，设置`StoreStore barrier`（关于`barrier`的更多细节，可以看这篇[文章](elink@661K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5h3u0J5i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6H3L8%4y4@1i4K6u0r3x3U0l9&6x3e0t1^5i4K6u0r3"><mark class="encrypted">e59K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5h3u0J5i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6H3L8%4y4@1i4K6u0r3x3U0l9&6x3e0t1^5i4K6u0r3</mark></a>)）。`barrier`的作用是阻止处理器执行后续搜集，直到当前指令执行完。

```c++
// StoreStore barrier required after complete initialization
// (headers + content zeroing), before the object may escape.
membar(MacroAssembler::StoreStore, tmp1);
```

对于一些设置为`public`的对象，这是必要的：如果代码中有错误，而这一对象被设置为`public`，你会想要看到它内部的`fields`（语言特性允许你这样做）的默认值或是开发者预设好的值，而不是一个随机的值，虚拟机希望获取到一个正确的`header`。`x86`架构的内存模型更加强大，它不需要这一指令，所以我们看到的是ARM的实现。

## 在实践中检验

> 注意上述代码可能有`bug`，我只是用这些代码来证明文章的正确性，并没有实际运行过

到目前为止，所有的一切看起来都棒极了：我们阅读了源代码，找到了一些有意思的东西，但是对编译器实际做的情况我们并不清楚，也许我们的理解都是错误的。再回头看看`PrintAssembly`中`new Long(1023)`的相关代码：
