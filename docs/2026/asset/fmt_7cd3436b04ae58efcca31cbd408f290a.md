
# [原创]这个崩溃有点意思，你中过招吗

## 缘起

前几天，在加班赶进度时遇到了一个意想不到的崩溃。由于是新加的代码导致的问题，所以很快就定位到了问题代码。但是，看了好几遍也没看出问题在哪？虽然代码在逻辑上有漏洞——某些情况下没有返回值，但是在我的认知里，应该不会导致崩溃。本文记录了使用 `IDA` 静态分析反汇编代码定位这个问题的过程。

## 示例代码

因为整个定位过程非常简单，就不在这里啰嗦了。定位到问题后，我特意建了一个简单的测试工程。关键代码不多，就几行，我把测试代码粘贴如下：

```c++
#include "stdafx.h"
#include <string>
class ConfigParam
{
public:
int option;
std::wstring strValue;
};
ConfigParam GetParam(int option)
{
ConfigParam result;
result.option = option;
result.strValue = L"default";
if (option == 0)
{
return result;
}
}
int _tmain(int argc, _TCHAR* argv[])
{
ConfigParam param = GetParam(1);
return 0;
}
```

在开始分析之前，请先停下来思考一下，上面的代码有问题吗？会导致崩溃吗？

如果之前看到这段代码，你问我会不会崩溃。我的回答是：**不会**。但是现在我的回答是：**会**。多么痛的领悟。

## 残酷的崩溃

在 `vs 2010` 中按 F5 调试启动，无情的中断下来了。入下图：

![crash-when-destrcutor-called](upload/attach/202209/873494_QUPF58C875YWQ72.jpg)

惊不惊喜，意不意外？

`GetParam()` 反回一个 `ConfigParam` 类型的对象，这个反回的对象在析构的时候却崩溃了。如果仔细观察 `GetParam()` 的实现，可以发现 `GetParam()` 并不是所有分支上都有返回值，但是编译器应该会返回一个临时对象。难道这个反回的临时对象有问题？对于这种问题，唯有通过反汇编才能找到答案。

## 请出 `IDA`

使用 `IDA` 打开对应的程序，找到 `GetParam()` 的反汇编，可以发现一个有意思的事情是 `GetParam()` 的形式。本来声明的是

`ConfigParam GetParam(int option)`，在 `IDA` 中看到的却是 `ConfigParam *__fastcall GetParam(ConfigParam *result, int option)`。如下图：

![GetParam-real-type](upload/attach/202209/873494_2Y8U9EDKJKNKJZR.jpg)

依稀记得多年前接触汇编的时候，了解到一种说法：如果返回值类型比较大（大家应该知道在 `32` 位程序中，函数的返回值基本是通过 `EAX` 反回的），那么会把返回值的地址当作第一个参数传递给函数，`EAX` 指向的是返回值的地址。正好跟 `IDA` 对应上了。

## 查看关键逻辑

代码中的 `GetParam()` 函数，当 `option` 是 `0` 的时候，会反回局部的 `result`，否则什么都不做。看看编译器帮我们做了什么吧。编译器做的事情也是，当 `option` 为 `0` 的时候，执行拷贝构造函数把局部的 `result` 返回出去，否则不会对参数中的 `result` 做任何操作。关键代码如下图所示：

![key-logic-not-assign-result](upload/attach/202209/873494_THFGPWJV43H6GKK.jpg)

那么在调用 `GetParam()` 函数的地方，会对 `result` 做什么初始化的工作吗？

## 查看 `main()` 函数逻辑

从下图可以清楚的看到，`main()` 函数并没有对 `result` 做任何初始化就传递给了 `GetParam()` 函数。

![result-not-initialized-in-main](upload/attach/202209/873494_JBD2FAKR3PNHJE6.jpg)

所以，调用完 `GetParam()` 后，`main()` 函数中的 `result` 是一个未初始化的对象。而不是一个调用过构造函数的对象。所以后面再调用其析构函数的时候，发生什么事情都是正常的了。我在遇到这个问题之前，一直以为 `GetParam()` 函数返回来的是一个初始化过的对象，因为根据之前的认知，在对象产生的时候一定会调用构造函数。这里既没有调用构造函数，也没有调用拷贝构造函数。

## vs 的 bug ？

这个问题最先是在 `vs2019` 上发现的，我还以为是 `vs2019` 的 `bug`，于是试了 `vs2017`、`vs2013`、`vs2010`，发现都会崩溃。但是每个版本的 `vs` 都会给出一个警告：`warning C4715: 'GetParam' : not all control paths return a value`。

![not-all-control-paths-return-a-value-warning](upload/attach/202209/873494_TZZGHZNS9E8CWDQ.jpg)

虽然给了警告，但是多少还是觉得 `vs` 的处理不太合理，难道所有编译器都是这个行为吗？试试 `gcc` 中的行为。

## `gcc`

不知道大家是否还记得我之前分享过的一个宝藏网址（<mark class="encrypted">3ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8$3c8T1L8$3I4@1i4K6u0W2L8%4u0Y4i4K6u0r3i4@1g2r3i4@1u0o6i4K6R3&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1$3i4K6W2r3i4@1p5#2i4@1f1%4i4K6W2o6i4K6S2n7i4@1f1#2i4K6V1H3i4K6R3@1i4@1f1%4i4@1p5%4i4K6S2p5i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1f1#2i4K6V1&6i4@1p5^5i4@1f1#2i4@1q4r3i4@1t1&6i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1$3i4@1q4q4i4@1t1#2i4@1f1@1i4@1u0n7i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1f1%4i4@1u0n7i4K6V1K6i4@1f1$3i4K6W2q4i4K6W2o6i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1#2i4K6W2n7i4@1u0q4i4@1f1$3i4K6V1^5i4@1q4r3</mark> `gcc5.2` 中 `GetParam()` 函数的反汇编代码。

![view-disassembly-code-of-getparam-in-gcc5.2](upload/attach/202209/873494_GXG995N2YGDC45F.jpg)

可见，逻辑十分清晰， `56` 行中的 `rdi` 指向的是返回值地址，第 `60` 行会先调用构造函数，传递的对象地址就是 `56` 行的 `rdi` （虽然中间经过 `[rbp-24]` 及 `rax` 倒了两手）。第 `69` 行判断 `option` 是否为 `0`，但是第 `70` 行直接来了个强制跳转（并没有根据比较结果跳转，这个编译器有点屌），跳转到了 `.L8` 的位置，后面几行是函数返回的处理。

可见，`gcc` 生成的代码会在 `GetParam()` 内部会先初始化，再返回。这样就避免了崩溃问题。

再看看 `main()` 函数的反汇编代码，入下图：

![view-disassembly-code-of-main-in-gcc5.2](upload/attach/202209/873494_XUZZ9K7FHMZHCJH.jpg)

逻辑非常清晰易懂。第 `89` 行把局部变量的地址加载到 `rax` 中，第 `90` 行把 `1` 赋值到 `esi` 中，第 `91` 行把 `rax` 的值放到 `rdi` 中，第 `92` 行调用 `GetParam()` 函数。

> **扩展：** 感觉 `gcc` 生成的反汇编对应的调用约定是这样的：函数的第一个参数通过 `rdi` 传递，第二个参数通过 `rsi` 传递。
>
> 简单搜了一下，`linux` 平台 `x64` 应用程序的调用约定还真是这样的，具体可以参考这篇文章 <mark class="encrypted">e20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2A6L8X3g2K6y4K6N6Q4x3V1k6H3i4K6u0r3x3K6M7^5z5o6f1I4y4q4)9J5k6h3S2@1L8h3I4Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

综上分析，同样的代码在 `gcc 5.2` 中的结果是正确的。

函数有返回值但是却不反回，这应该不算是正常情况，也许在标准中对这种行为有描述？是未定义行为？编译器可以根据自己的喜好发挥？一切还要到标准中找答案。

## 翻看标准

在网站 <mark class="encrypted">b1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7r3g2F1i4K6u0V1M7%4c8V1i4K6u0W2L8%4u0Y4i4K6u0r3d9W2c8o6x3g2)9J5c8W2y4o6x3U0u0Q4x3V1k6i4c8K6t1I4i4K6u0r3k6r3!0U0M7#2)9J5c8Y4y4@1j5h3&6V1j5i4u0V1M7H3`.`.</mark> 上找到了 `c++` 标准的草稿。我参考的版本是 `N3242`。这个是 2011 版的草稿。网站上的原话是

> A draft for the 2011 edition is available in <a href="elink@846K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7r3g2F1i4K6u0V1M7%4c8V1i4K6u0W2L8%4u0Y4i4K6u0r3d9W2c8o6x3g2)9J5c8W2y4o6x3U0u0Q4x3V1k6i4c8K6t1I4i4K6u0r3k6r3!0U0M7#2)9J5c8Y4m8S2M7r3g2J5M7#2)9J5c8U0t1H3x3e0q4Q4x3V1k6F1x3K6t1@1x3W2)9J5k6i4m8V1k6R3`.`.">N3242</a>.

在第 `6.6.3` 节中有一段简单的描述：有返回值却不返回值的情况是未定义的行为。原文截图如下：

![undefined-behaviour-in-cpp-standard-draft](upload/attach/202209/873494_9JUP5ASH8PRT5JJ.jpg)

## 总结

如果一个函数是有返回值的，但是却不返回值，这个行为是未定义的。每个编译器可以自由发挥。很多版本的 `vs` 会給警告。一定要重视编译器的警告！！！

## 参考资料

N3242 <mark class="encrypted">050K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7r3g2F1i4K6u0V1M7%4c8V1i4K6u0W2L8%4u0Y4i4K6u0r3d9W2c8o6x3g2)9J5c8W2y4o6x3U0u0Q4x3V1k6i4c8K6t1I4i4K6u0r3k6r3!0U0M7#2)9J5c8Y4m8S2M7r3g2J5M7#2)9J5c8U0t1H3x3e0q4Q4x3V1k6F1x3K6t1@1x3W2)9J5k6i4m8V1k6R3`.`.</mark>

调用约定 <mark class="encrypted">c30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2A6L8X3g2K6y4K6N6Q4x3V1k6H3i4K6u0r3x3K6M7^5z5o6f1I4y4q4)9J5k6h3S2@1L8h3I4Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

查看反汇编代码的宝藏网址 <mark class="encrypted">4d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8$3c8T1L8$3I4@1i4K6u0W2L8%4u0Y4i4K6u0r3</mark>
