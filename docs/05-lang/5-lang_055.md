
# mscorjit 的介绍，以及 maxtocode 2007 企业版运行库分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-197.htm`](https://www.pediy.com/kssd/pediy09/pediy09-197.htm)

在前面介绍 `mscorwks` 时提到了，.NET 的程序是以函数为单位编译。而在 `mscorjit` 中提供了一个函数 `compileMethod()`。`mscorwks` 就是通过调用这个函数来编译 .NET 方法的。

对于 EE 层，或者虚拟机预处理层的加密壳，只需要 hook 这个函数就可以 dump 出方法体的代码了。

需要注意一点，这个函数是 `thiscall` 调用约定的。

.NET 方法体进入 `compileMethod()` 之后是怎么样的处理流程呢？

限于篇幅，这里只能简单介绍一下，具体详情可以参考 `sscli` 2.0 的源代码。

`compileMethod()` 实际上只是一个接口函数，它没有做实际什么工作，只是简单地调用另一个函数：`jitNativeCode()`。

在 `jitNativeCode()` 函数是一个线程安全的函数，在它里面实际上只是做了一些准备工作。安装异常处理，然后实例化一个 `Complier` 对象。初始化 `Complier` 类，再调用这个类的一个成员函数 `compCompile()`。

在 `compCompile()` 就开始了实际的编译工作。

所以 hook `jit` 来实现脱壳的话，我们有三个选项：

- hook `compileMethod()` (`thiscall`)
- hook `jitNativeCode()` (`fastcall`)
- hook `compCompile()` (`thiscall`)

对于这三个方案，实现脱壳是完全一样的。在这三个函数里面我们都能够获取到脱壳所需要的结构体。实际上脱壳需要的结构体是 `CORINFO_METHOD_STRUCT`、`CORINFO_MODULE_STRUCT`、`CORINFO_METHOD_INFO` 这三个。

我在脱某壳个人版加密的程序时是采用的方法 2，hook `jitNativeCode()` 函数。

hook 方法：采用的替换方式，即直接修改 `compileMethod()` 函数，将 `call jitNativeCode()` 改为 `call myhook()`。在 `myhook()` 函数里面处理脱壳工作，然后再调用 `call jitNativeCode()`。

实际上有一个技巧。一般我们通过反射 `invoke()` 让一个方法体进入 `jit` 处理过程的，然后在 `myhook()` 里面截取 dump 方法体。

假设我们在 dump 一个 `Exit()` 函数时，如果让它执行了会导致进程退出，因为 `invoke()` 会执行方法，而我们需要的只是让方法体进入 `jit` 处理过程，以方法 dump，而不希望这个方法被执行。所以我们可以不用回调 `jitNativeCode()` 函数，直接返回一个 `nop` 地址。

`maxtocode` 2007 企业版具称是虚拟机处理层的壳，那它在虚拟机处理层做了什么工作呢？

具分析 `mscorjit.dll` 的内存镜像，它实际上做了和 `jit` 层脱壳差不多的工作，同时使用了方案 2 和方案 3。显然它 hook `jit` 的目的不是脱壳，而是实现方法体的还原。

我们可以很容易确定，在程序运行到 `compCompile()` 中后，加密壳的运行库已经完成了所有解密工作，因此在 `compCompile()` 函数里面，或者它下级的函数里面是脱壳的好时机。

如是，我修改了一下 `jithook` 程序，只改了 hook 位置，即 hook `compCompile()` 的下级函数。然后测试 dump。效果如下：

![](img/c3a26660b3fe3b24b318bfe3e43b2f09.jpg)

可以 dump 出源代码。

看来企业版相对个人版来说只是 hook 位置的变化，对于脱壳来说，基本上没有增加任何难度，只需要更改一下 hook 位置，就可以 dump 了。

另外发现企业版运行库的一个 bug，可以用简单的方法脱壳。

如果只是分析程序注册算法，我们只需要 `CORINFO_METHOD_INFO` 就够。在 `tankaiha` 大以前的文章介绍过。直接获取这个结构体中的 `ILCode`，而后用 `ilbytedecoder` 取得 `il` 汇编代码。异常处理表不用修复也不影响分析代码。

4dd466d0e1e0cc123afbe504da2462ca
