
```
\_int\_malloc
-> for (;;)
-> while ((victim = unsorted\_chunks (av)->bk) != unsorted\_chunks (av))
-> bck = victim->bk;
-> size = chunksize (victim);
-> ......
-> unsorted\_chunks (av)->bk = bck;
-> bck->fd = unsorted\_chunks (av);
-> if (size == nb)
-> ......
-> return p;
-> if (in\_smallbin\_range (size))
-> ......
-> else
-> ......
```

进入`small bin`链表时的操作相对简单，可以自行阅读源代码，但当`victim`进入链表`large bin`中时，操作就会变得复杂起来。

变得复杂是因为`large bin`链表可以存储不同大小chunk的特殊性导致的。

当`victim`的大小小于`large bin`链表最早入链的`bck->bk`时，GLibC会选择直接将`victim`插入`large bin`链表，作为最晚入链的chunk存在。

但当`victim`的大小大于等于`large bin`链表中原最早入链chunk时，处理操作就会变得复杂起来，主要原因是`\_int\_malloc`函数需要遍历`large bin`链表，找到一个比`victim`更小的chunk。

从`while`循环语句中可以看到，`fwd`不断的根据`fd_nextsize`去更新，而循环中止的判断条件只是`fwd`小于等于`victim`的`size`时才会停止，那么GLibC就不怕找不到合适的chunk，导致这个循环一直进行产生死循环吗？

当然不用考虑！因为`if ((size) < chunksize_nomask (bck->bk))`已经告诉我们一个事情，那就是`victim`比`large bin`链表中最小的chunk要大，所以在遍历字段`fd_nextsize`的过程中至少也会找到一个chunk小于`victim`。

当根据`fd_nextsize`遍历`large bin`链表找到的`fwd`大小刚好等于`victim`的时候，GLibC会直接让`fwd`等于`fwd->fd`，`fwd->fd`相当于找到比`fwd`更晚入链的chunk，这么做是因为GLibC会固定将`victim`插入第二的位置。

直接往`fwd`的前面插入不行吗，为什么是`fwd`维护的`fd`链表中第二的位置呢？

GLibC主要是为了避免给`victim`更新`xx\_nextsize`，这是因为相同大小的chunk所在的链表中，只有头成员的`xx\_nextsize`是起作用的。

如果`fwd`小于`victim`，由于前面也没有找到和`victim`相等chunk，所以可以知道`victim`对应的大小在链表中还是唯一的，所以接下来GLibC会更新`fwd`和`victim`的`xx\_nextsize`。

```
if (in\_smallbin\_range (size))
-> ......
else
-> victim\_index = largebin\_index (size);
-> bck = bin\_at (av, victim\_index);
-> fwd = bck->fd;
-> if (fwd != bck)
-> if ((size) < chunksize\_nomask (bck->bk))
-> fwd = bck;
-> bck = bck->bk;
-> xxx->xx\_nextsize = xxxx
-> else
-> while (size < chunksize\_nomask (fwd))
-> fwd = fwd->fd\_nextsize;
-> if (size == chunksize\_nomask (fwd))
-> fwd = fwd->fd;
-> else
-> victim->fd\_nextsize = fwd;
-> victim->bk\_nextsize = fwd->bk\_nextsize;
-> fwd->bk\_nextsize = victim;
-> victim->bk\_nextsize->fd\_nextsize = victim;
-> bck = fwd->bk;
-> else
-> victim->fd\_nextsize = victim->bk\_nextsize = victim;
-> victim->bk = bck;
-> victim->fd = fwd;
-> fwd->bk = victim;
-> bck->fd = victim;
```

从上面可以看到，当程序还握有已释放`fwd`的内存地址且可以修改它时，一共会产生三处风险代码，而这些风险代码就是与`fwd->xx`相关的赋值语句。

```
set fwd->fd = evil\_address\_1
set fwd->bk\_nextsize = evil\_address\_2
set fwd->bk = evil\_address\_3
fwd = fwd->fd;
-> fwd = \*(fwd + offset(fd)) = evil\_address\_1
fwd->bk = victim;
-> \*(evil\_address\_1 + offset(bk)) = victim
victim->bk\_nextsize = fwd->bk\_nextsize;
victim->bk\_nextsize->fd\_nextsize = victim;
-> \*(evil\_address\_2 + offset(fd\_nextsize)) = victim
bck = fwd->bk;
bck->fd = victim;
-> \*(evil\_address\_3 + offset(fd)) = victim
```

上面风险代码诞生于`unsorted bin`中的`large chunk`移动到`large bin`链表的时候，当我们篡改`fwd`上`malloc\_chunk`结构体占用字段的数值为恶意地址时，会导致GLibC向恶意地址上写入`victim`的地址。

这个恶意地址当然是可以随意指定的，只要这片内存区域是可读可写的就可以，只不过与任意地址申请为chunk的场景有所不同，这里只能写入特定的内容。

### largebin链表更特殊的属性

前面提到过`large bin`链表和`unsorted bin`链表与其他链表有一个显著的区别，那就是`large bin`和`unsorted bin`允许链表中存放大小不相同的chunk。

`unsorted bin`是`small chunk`和`large chunk`被释放时的第一去处，该链表并不会对chunk按照大小进行排序。

那么`large bin`也是这样的吗？

当然不是，`large bin`会将入链的chunk按照大小进行排序。

`large chunk`从`unsorted bin`链表移动到`large bin`链表的时机，其实就发生在函数`\_int\_malloc`的内部，当函数遍历`unsroted bin`链表，发现chunk不能匹配申请者要求的时候，就会将chunk移入`small bin`或`large bin`中。

下面列出了`large chunk`进入`large bin`链表时的全部分支，从分支的判断条件上，我们可以窥探出GLibC是如何排列`large bin`中的chunk的。

```
victim\_index = largebin\_index (size);
bck = bin\_at (av, victim\_index);
fwd = bck->fd;
if (fwd != bck) {
if ((size) < chunksize\_nomask (bck->bk)) {
fwd = bck;
}
else {
while (size < chunksize\_nomask (fwd)) {
fwd = fwd->fd\_nextsize;
}
if (size == chunksize\_nomask (fwd)) { ...... }
else { ..... }
}
bck = fwd->bk;
}
else {
victim->fd\_nextsize = victim->bk\_nextsize = victim;
}
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

最先面临的判断是`fwd != bck`，因为`fd`和`bk`两个字段所维护的都是循环链表，所以GLibC用了一个特殊的数据作为链表头，即`&bins[i] - offset(fd)`，这个特殊的数据可以保证`xx->fd`和`xx->bk`分别指向`fd`的链表头成员`bins[i]`和`bk`的链表头成员`bins[i + 1]`，所以当`fwd == bck`时，就说明链表是空的。

`large bin`链表为空时，直接将chunk插入就可以，但当链表不为空时，GLibC会检查`bck->bk`指向的链表头chunk是否大于待移入`victim`，如果是，那就更新`fwd`为`bck`，此时的`bck`等于`&bins[i] - offset(fd)`，后面插入`victim`时，会通过`fwd->bk`将`victim`插入`bk`链表头的位置。

从这里可以看出，`fd`链表和`bk`链表已经不止根据chunk入链时间维护链表了，还会根据chunk的大小进行维护，并且`bk`链表头上保存的永远都是链表中的最小chunk。

如果`victim`的大小比`bk`链表头成员要大，那就好遍历`fd\_nextsize`字段，在链表中找到小于等于`victim`的chunk，然后完成插入。

至于`bins`数组中`fd`字段所在的位置，它与`bk`位置保存最小chunk的属性有所不同，该字段会保存链表中最大的chunk。

从`if (size == chunksize\_nomask(fwd))`对应的`else`分支可以知道，找到一个小于待插入的`victim`时就会进入`else`分支，如果`victim`是链表中最大的一个，那么`bck = fwd->bk`语句运行完后，`bck`就等于`&bins[i] - offset(fd)`，从而使得最后插入时`bck->fd`的值等于`bins[i]`，使得`fd`所在的位置被插入整个链表中最大的chunk。

### 传统攻击方式的消失

向任意地址写入特定chunk地址的攻击，依赖于`fd`、`bk`、`bk\_nextsize`三个属于结构体`malloc\_chunk`的字段，将它们篡改后，`\_int\_malloc`函数的流程内部，就可能将它们改写。

不管是修改哪一个字段，都要求黑客迫使`\_int\_malloc`函数的执行流程进入chunk从链表`unsorted bin`中迁移到`large bin`链表的分支中。

当`unsorted bin`中无法与申请大小匹配的`victim`的超过`large bin`链表中的空闲chunk时，就会开启基于`bk`和`bk\_nextsize`的利用，而当大小是相等时，则会轮到`fd`字段发挥可利用的作用。

#### 传统攻击方式的消失

在任意地址写入特定数据的攻击方式是一种较为古老的攻击方式，既然是古老的，那就代表它已经被GLibC修复掉了。

下面是GLibC的修复手法，依旧是我们非常熟悉的根据相邻chunk确认数据有效性。

```
if (fwd->bk\_nextsize->fd\_nextsize != fwd)
malloc\_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
if (bck->fd != fwd)
malloc\_printerr ("malloc(): largebin double linked list corrupted (bk)");
```

### 天无绝人之路

上面介绍的向任意地址写入特定数据的漏洞，其实是忽略了GLibC的部分的代码的，只注意关注了`else`的部分，而忽略了`if`分支下的代码。

那么`if`分支下的代码是不是极其稳固呢？

答案是否定的。

在下方代码中可以看到，`fwd->fd`对应`large bin`链表中的最晚入链chunk，另一个`fwd->fd->bk\_nextsize`则对应最晚入链chunk的`bk\_nextsize`字段。

当程序可以控制`large bin`链表中最晚入链chunk的`bk\_nextsize`字段时，赋值语句`victim->bk\_nextsize`就会指向我们所设置的恶意地址。

当`victim->bk\_nextsize->fd\_nextsize = victim`语句运行完后，恶意地址所指向内存区域偏移`offset(fd\_nextsize)`的地址就会被写上`victim`存储的数据。

```
bck = bin\_at (av, victim\_index);
if ((size) < chunksize\_nomask (bck->bk))
{
fwd = bck;
bck = bck->bk;
victim->fd\_nextsize = fwd->fd;
victim->bk\_nextsize = fwd->fd->bk\_nextsize;
fwd->fd->bk\_nextsize = victim->bk\_nextsize->fd\_nextsize = victim;
}
else { ...... }
```

至于怎么样才能进入`if`所在的分支，那就需要做到以下几点。

首先`unsorted bin`链表中必须存在一个`large chunk`，且这个`large chunk`需要比`large bin`链表中的最早入链chunk小，其次就是程序发出的堆内存申请的大小不能和`unsorted bin`链表中`large chunk`相等。

做到上面的两点，就可以顺利的进入`if`分支了。

那么问题就又来了，类似的问题之前并非没有先例，作为业界顶级项目GLibC的维护者，难道他们就没有注意到这个问题吗？

是难以加固代码，还是根本就没有察觉呢？

`fwd->fd->bk\_nextsize`指向比`fwd->fd`更小的chunk，而`fd\_nextsize`则会指向比它更大的chunk，按照道理来讲，`fwd->fd->bk\_nextsize->fd\_nextsize`会重新指向`fwd->fd`，只要检查这个结果是否成立，就可以验证数据的正确性了。

但是GLibC为什么不进行检查呢？

而且GLibC中也不是没有这样的检查啊！

```
victim->bk\_nextsize = fwd->bk\_nextsize;
if ((fwd->bk\_nextsize->fd\_nextsize != fwd))
malloc\_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
```

### 可以做到任意地址申请吗？

负责维护链表的`fd\_xxxx`和`bk\_xxxx`的字段被篡改后，按照道理来讲，是可以影响程序申请chunk的流程的，但是事情真的能那么顺利吗？

程序想要从`large bin`中获取chunk，只有一个要求，那就是`unsorted bin`链表不能提供给申请者提供chunk，在结束`unsorted bin`链表的遍历之后，只要GLibC通过`in\_smallbin\_range`宏判断出申请大小不属于`small chunk`的范围，就会把它视为针对`large chunk`的申请。

此时会正式开始从`large bin`链表中获取`large chunk`。

能不能从`large bin`链表中取出chunk，可不是申请大小符合`large chunk`的范围就可以了，GLibC有两点要求，一是`large bin`链表不为空，二是根据`first`取出链表中最大的`victim`跟申请大小`nb`进行判断，如果申请大小小于`victim`，才会从`large bin`链表中取出chunk，否则的话，也找不到能匹配的chunk。

```
\_int\_malloc
{
for (;;) {
while ((victim = unsorted\_chunks (av)->bk) != unsorted\_chunks (av)) { ...... }
if (!in\_smallbin\_range (nb)) {
if ((victim = first (bin)) != bin && chunksize\_nomask (victim) >= (nb)) { ...... }
}
}
}
```

在可以从`large bin`链表申请`large chunk`的时候，GLibC会获取当前链表中最小的chunk，如果该chunk大于申请大小就会拿来使用，否则则会继续按照从小到大的顺序遍历链表，直到找到大于申请大小`nb`的chunk。
