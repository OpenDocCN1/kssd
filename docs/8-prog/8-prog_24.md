
# projectoz 编译

> 原文：[`www.pediy.com/kssd/pediy11/116236.html`](https://www.pediy.com/kssd/pediy11/116236.html)

高手飘过...

前几天下了个 `WRK`, 编译后想再编译 `projectoz` 学习下, 本人比较懒, 直接 `GOOGLE` 了下, 只找到了 `readme` 中的下面几行帮助,

```
    set  wrk=the WRK directory
    set  projectoz=the ProjectOZ directory
    path %wrk%\tools\x86;%path%
    nmake -nologo x86=
```

刚开始做了个批处理, 把这几行放进去, 编译时报没有找到 `"nt.h"`, 又到网上一顿好找, 只找到 2 条, 也没有什么帮助, 书店也转了下, 最后没办法, 觉得还是自己尝试解决吧.

打开 `projectoz` 目录下的 `makefile` 文件, 一开始就发现 `wrk` 目录设置与自己机器不符, 马上备份一份原文件, 修改为正确路径。我最后编译做的工作为：

在环境变量里添加了 2 个变量：

```
wrk=E:\WRK-v1.2
projectoz=E:\ProjectOZ
```

在 `path` 路径前面加上 `path %wrk%\tools\x86;%path%`；

做了个批处理文件，内容就 2 行：

```
    path %wrk%\tools\x86;%path%
    nmake -nologo x86=
```

然后运行这个批处理，没想到编译成功，而且可以使用。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

![](img/e53a4cb8715f5b504c1c1e3652a16818.jpg)

![](img/d997ec5af0680e9d4d4eadcb5f94c935.jpg)

因自己在网上没有找到相关信息，避免同学们也浪费时间，其实没有什么技术含量。
