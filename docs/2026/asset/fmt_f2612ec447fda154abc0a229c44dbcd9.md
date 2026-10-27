
此时图更新为：

![](https://bbs.kanxue.com/upload/attach/202108/847490_WXH7GZP6EYEZZYZ.jpg)

## 实验验证：

先运行一个`exe`一次，然后记录其基址：

![](https://bbs.kanxue.com/upload/attach/202108/847490_VEDSUHW933ZS8NB.jpg)

然后在`nt!NtCreateUserProcess`中下断，再次启动该程序，跟踪上述流程，并寻找`_Segment.BaseAddress`。

![](https://bbs.kanxue.com/upload/attach/202108/847490_8CEGRY7WKRWHBE9.jpg)

不难发现，第二次启动程序使用了第一次启动后缓存的`_SEGMENT`，所以基址保持不变。并且查看`BeingCreated`标志，也会发现此时该标志为0。

![](https://bbs.kanxue.com/upload/attach/202108/847490_EG8WZC85M9BXSRM.jpg)

接下来再次启动该程序，并将其`BeingCreated`标志置为0，查看其基址又是如何的情景呢？（此时我是将提出的问题1,2合并来进行测试）。将`exe`剪切移动到一个位置后，再移动回来，查看其`BeingCreated`标志。

![](https://bbs.kanxue.com/upload/attach/202108/847490_F7XDC6UTSDCH9WG.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/847490_C4GFS3H33U7TSBM.jpg)

为了更加了解`_SEGMENT.BaseAddress`如何来的，所以继续顺着流程往下跟踪。（注解：`ImageBase`来源于`_SEGMENT.BaseAddress`，所以我一般会混合称呼。 ![](https://bbs.kanxue.com/upload/attach/202108/847490_93MVEYA9MFE422G.jpg)

对于新创建的`SEGMENT`流程来说：

`nt!MiCreateNewSection`->`nt!MiRelocateImage`->`nt!MiSelectImageBase`最终生成`_SEGMENT.BaseAddress`。

其中生成的算法为： ![](https://bbs.kanxue.com/upload/attach/202108/847490_CNADMGETMK2WFBE.jpg)

充满好奇心的同学，就如我一样，肯定会心中产生一个疑问，那么便是为什么，这里会有两种产生随机地址的流程呢？那么让我们继续顺藤摸瓜，向上查找

接着，就会寻找到如下图的关系：

在`nt! MiSelectImageBase`会判断`_CONTROL_AREA.u2.e2.ImageBaseOkToReuse`字段是否为1，说明当这个字段为1的时候，那么也会进行重定位。这个情况是发生在`_CONTROL_AREA`“缓存的情况”下（为什么是这个情况呢？因为另一种情况会重新创建`Segment`），即发生下： ![](https://bbs.kanxue.com/upload/attach/202108/847490_QHGZ5AAE5F9HJ3G.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/847490_YU6RR9R4M8GAUMT.jpg)

这就得出了一个结论，如果`ImageBaseOkToReuse`标志在置位的情况下，即时存在`CONTROL_AREA`的“缓存”，那么也会进行**重新计算`BaseAddress`**，所以可以看到在这个分支中会出现一个独特的函数，即：`nt!MiSwitchBaseAddress`。

当`nt!MiSelectImageBase`返回后，会返回到`nt!MiRelocateImageAgain`函数中，

`nt!MiRelocateImageAgain`函数会判断返回的`BaseAddress`值与原来的`_SEGMENT.BaseAddress`中的值是否一致，如果不一致的话，那么就会调用`nt!MiSwitchBaseAddress` 函数更新`_SEGMENT`。

![](https://bbs.kanxue.com/upload/attach/202108/847490_MC3CUAHMZRTFF8B.jpg)

但是这里还有一个小细节，就是它会判断`ImageActive`标志位，这个位顾名思义(我猜的)，代表的是当前进程是否存活。为了突出重点，所以我标出了流程中关键的部分，具体图如下：

![](https://bbs.kanxue.com/upload/attach/202108/847490_VJC4HQQ9KNQCFCY.jpg)

细分析如下：

在`nt!MiRelocateImageAgain`函数中，首先判断 `_control_area .u2.e2.ImageActive`。是否为1（即）当前进程是否存活。如果不是退出状态，则会调用`nt!MiSelectImageBase`函数来获取基址。 ![](https://bbs.kanxue.com/upload/attach/202108/847490_74Y3TDTDHKU6888.jpg)

如果获取到**新的`BaseAddress`**后，则将其和老的进行比较，不相同，则调用 `nt!MiSwitchBaseAddress`来更新`_SEGMENT`。

![](https://bbs.kanxue.com/upload/attach/202108/847490_NKTQHQ39XDDRYMJ.jpg)

对于其他的流程，后续总结的时候简要说明一下，`nt!PspAllocateProcess`函数流程可以参考《64位Windows创建64位进程逆向分析》。

![](https://bbs.kanxue.com/upload/attach/202108/847490_QYXANH7WUWJ2D7N.jpg)

# 总结：

对于复用`CONTROL_AREA`“缓存”的情况图：

![](https://bbs.kanxue.com/upload/attach/202108/847490_RM88J6URM89G74V.jpg)

对于重新创建`SEGMENT`的图：

![](https://bbs.kanxue.com/upload/attach/202108/847490_JY5ESNGNUU55A5T.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/847490_CW654WFH2WS9XH7.jpg)

简单说一句，如果想让内存在用户层可见，都需要调用`nt!MiMapViewOfImageSection`函数，这个函数主要作用就是创建`_SECTION`对象，并将`_SEGMENT.BaseAddress`函数映射到用户层可见部分。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-8-10 22:29

被烟花易冷丶编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
