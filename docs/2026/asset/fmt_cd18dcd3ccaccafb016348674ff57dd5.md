
# [原创] Android10 aarch64 dlopen Hook

## 前言

对于自动化hook `Il2cpp` 的模块来说, `dlopen` 的hook相当于一个大门, 没有该大门口, 一切都是纸上谈兵

在 **armabi-v7a** 上hook `dlopen`, 轻松的不要不要的, 甚至借用一下 `virtual` 系列app的 `va++` 核心提供的 `hook_dlopen` 函数的接口都行

可是到了 **aarch64** 这里, 找了一圈, 能用的 `hook构件` 也就一枚 [And64InlineHook](https://bbs.kanxue.com/elink@b54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9M7s2u0G2M7q4)9J5c8V1q4F1k6o6j5@1d9h3&6D9K9h3&6W2d9r3!0G2K9H3`.`.), 而且源项目好像还得自己手动修一下才能使用.... ~~(可惜whale不支持安卓10, 原hookzz的Dobby也木大)~~

好了有 `hook构件` 了, 但是我看了一圈主流的 `VirtualApp` 商业授权做出来的 `虚拟多开` 系列软件, 基本上都没实现 **aarch64** 的 `hook_dlopen`, 有的实现了却没有给调用, 自己去调用的话就会卡死 ~~(还想 **借鉴** 一下方案, 逃个课先)~~

![就你妈离谱](https://bbs.kanxue.com/upload/attach/202105/855290_Z4GUBB43EK89XFH.jpg)

## 环境 & 预习

```
测试平台
机型: 红米K30-4G, 已Root
系统: miui11, android-10
```

看雪上看到的一篇文章: [Android9.0 hook dlopen问题](https://bbs.pediy.com/thread-257022.htm)

虽然半年前就看到这篇文章,但是基本上在看天书,现在拿出来看了一圈, 关键实现就是这里

![](https://bbs.kanxue.com/upload/attach/202105/855290_QJHJAXGHHA7CMUF.jpg)

虽然我看不懂什么 `LR寄存器` 什么的, 但是唯独看懂这第三个参数强制赋值为 `dlerror` 函数的函数地址, 以达到越过系统的限制

而关于那所谓的系统限制, 就是在安卓7开始, 系统就禁止 `用户APP` **打开/读取** 部分 `系统库文件` , 所以直接 `dlopen linker` 会直接失败, 具体可以使用 `dlerror()` 函数查看原因

## 初步尝试Hook

### 反汇编 libdl.so

貌似安卓9开始, `dlopen`就由 `libdl.so` 库进行导入, 反汇编看看

看了一下导出函数, 也就只有两个函数涉及到`dlopen`

![](https://bbs.kanxue.com/upload/attach/202105/855290_Y2J5UB7JKZ39HA4.jpg)

让我康康这个 `dlopen` 长什么样!

![](https://bbs.kanxue.com/upload/attach/202105/855290_YS66MAHEJG7QGPT.jpg)

![](https://bbs.kanxue.com/upload/attach/202105/855290_QDKU23BMWE2D2AU.jpg)

中间的 `.__loader_dlopen` 是个跳板
