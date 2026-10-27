
```
0: kd> u fffff803`7520d400 nt!KiSystemCall64:
```

那下面我们看一下先来看一下`KiSystemCall64Shadow`的代码是什么。

```
ntoskrnl - 单处理器，不支持PAE
ntkrnlpa - 单处理器，支持PAE
ntkrnlmp - 多处理器，不支持PAE
ntkrpamp - 多处理器，支持PAE
```

上面是内核文件名字的区别，大家根据自己的环境去找内核文件进行逆向分析。关于符号的话，可以设置内核符号表地址，或者直接 `load pdb` 文件都可以。

然后 `alt+t` 全局搜索`KiSystemCall64`，就可以找到跳板函数和真正的函数了。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_5EV7AHQTV3MJEQZ.webp)

## `KiSystemCall64Shadow`

进来之后就看到切换 `gs`，然后保存现场到代码。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_RPDUGRPXC8ASTEQ.webp)
![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_GPEVRU8Z8229MPA.webp)

然后在往下看就会跳到`KiSystemServiceUser`这个函数里。注意到这个函数的时候已经不是`KVASCODE`段了，而是跳到了`.text`段，说明已经进入到了真正的内核中，而不是跳板代码。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_G5667JYZXJ4HGE6.webp)

我们再来看一下这个函数。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_MGX2Z3E9FVSUN3P.webp)

下面的过程就跟 xp 系统里的流程差不多了：判断是不是 GDI 线程，如果是的话用 Shdow 表，并且要转换线程位 GDI 线程。这部分我们不看，我们现在直接看调用函数的位置。

在中间过程里，`KiSystemServiceCopyStart`这个函数负责复制参数，有兴趣的可以看一下。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_JT3J8GDRSE7EWHN.webp)

大家可以再看一下`KiSystemCall64`函数，会发现这两个函数长得差不多，而且也是走的`KiSystemServiceUser`这个函数，我这边就不带大家看了。

可以发现系统 API 的过程大致如下：

1. 三环调用 API
2. `ntdll` 调用 `syscall` 进入 0 环
3. 如果开启了内核页表隔离机制的话，就调用`KiSystemCall64Shadow`，否则调用`KiSystemCall64`；两个的区别就是`KiSystemCall64Shadow`需要切换 `cr3`，而`KiSystemCall64`不需要。
4. 在`KiSystemCall64Shadow`函数里切换栈并保存现场到 0 环栈，把 0 环栈底作为`KTRAP_FRAME`结构体用。
5. 保存完现场就是找到 `ssdt` 表，然后调用对应序号的函数即可。

## 阶段实验

实验内容：驱动实现遍历 `ssdt` 表并打印出来。目前我们还没办法 hook `ssdt` 表，因为 Windows 系统有 pg 机制。

（关于 `ssdt` 表的结构等知识，大家到网上搜一下就有了，跟 xp 时候基本一样。）

驱动代码我过两天会贴出来，国庆结束了，这几天会比较忙。

另外说一下，我其实本来是想着在页机制的研究那章写页表隔离机制和如何找自映射的，不过后来在做这套教程的时候又觉得不太合适。我也是一边找自己之前自学时候的笔记，一边回顾去整理出来，可能发出来的文章并没有跟之前说的文章顺序一致，不过最后所有的内容肯定都会有的。

这是我提前整理出的目录结构，现在在一点一点填充内容，后面会慢慢发出来的。

![图片描述](https://bbs.kanxue.com/upload/attach/202310/987725_73CMAD4VK894E2S.webp)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)  
[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)  
[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)
