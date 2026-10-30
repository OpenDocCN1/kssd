
# LSAN，Leak Sanitizer

用于检测程序中的内存泄露。内存泄露通常并不会导致程序crash，但它是一个不稳定的因素，可能会被利用、也可能没办法被利用，这不是一个严格意义上的漏洞。与其他Sanitizer的使用不同，需要将 `__AFL_LEAK_CHECK();` 添加到你想要进行内存泄露检查的目标源代码的所有区域。在编译之前启用， `export AFL_USE_LSAN=1` 。要忽略某些分配的内存泄漏检查， `__AFL_LSAN_OFF();` 可以在分配内存之前和 `__AFL_LSAN_ON();` 之后使用，LSAN不会检查这两个宏之间区域。

> Note：
>
> 1. 一些Sanitizer不能混用，而即使有些可以同时允许的Santizier也可能导致意想不到的行为影响fuzzing，这需要结合你fuzzing的目标情况而定。如果你不熟悉Sanitizer的原理，最好一个编译实例中只启用一个Sanitizer，这样通常不会出问题，而且组合Sanitizer不见得会有好效果，基于对目标的了解正确的使用Sanitizer才是最佳的实践。
> 2. 有些Sanitizer提供了参数设置的环境变量，如 `ASAN_OPTIONS`，如果你有很明确的需求可以设置该变量进一步限制Sanitizer的检测行为，这可能会提高你fuzzing的效率。如果你不熟悉、也没有明确的需求，那么保持默认即可，这通常是最实用的。
> 3. 启用CFISAN的实例，可能会检测出很多crash(成百上千)，这是正常的，但大多数是无用的，甚至全是无用的，你需要注意甄别。
> 4. 如果你对目标内部结构足够熟悉，你确定那些区域是线程并发的高发区域，那么你可以结合TSAN与 **partial instrumentation** 功能提高TSAN的检测效率，因为启用TSAN的实例通常fuzzing速度会大幅减慢。
> 5. 通常启Sanitizer后，会大幅减慢fuzzing的速度，CPU每秒执行次数会减少，内存也会被大量消耗(AddressSanitizer会大量消耗内存，甚至可能导致计算机内存耗尽)。如果你的计算机配置不行，请斟酌一个合理的搭配。
> 6. 一种Sanitizer只应该允许一个实例。在两个实例上允许两个同样的Sanitizer是一种浪费，因为AFL++会同步所有实例的testcase，其他实例的testcase无论如何都会被该实例上的Sanitizer检测一遍，不应该启用两个相同的Sanitizer检测两遍，这会减慢效率。

暂时只想到这些，以后想到了再补充。

## LLVM Persistent Mode

In-process fuzzing是一个强大功能，通常比默认常规编译fuzzing的速度快得多，大概快10-20倍，并且基本没有任何缺点。如果可以，请毫不犹豫的使用Persistent mode。

众所周知，AFL使用ForkServer来进行每次fuzzing，然而即便不用execve这种巨大的开销，但fork仍然是一笔不小的开。而Persistent fuzzing即一次fork进程种进行多次fuzzing，而无需每次都fork。

Persistent mode提供一组AFL++的函数和宏，我们使用下面的形式,用一个while包含我们要进行Persistent fuzzing的区域。请注意，该区域的代码必须要是无状态的，要么是可以手动可靠的重置为初始状态！这样我们才能再每次fuzzing时重置进而再次fuzzing。

`afl-clang-fast`/`lto`编译的情况下，只需要使用下面的形式即可，但若不是，则复杂一些。

AFL++官方的仓库对Persistent Mode花了不小的篇幅讲诉，讲的也比较全面，请在此处[Persistent Mode](https://bbs.kanxue.com/elink@34cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8c8V1I4H3L8s2g2K6M7r3I4#2M7#2)9J5c8V1q4r3e0s2m8D9N6i4y4H3L8s2g2K6i4K6u0r3j5X3I4G2j5W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8X3W2F1M7%4c8J5N6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8W2u0q4b7f1c8y4c8g2)9J5k6i4m8W2M7Y4y4A6M7%4c8W2L8Y4c8Q4y4h3k6E0L8$3c8W2i4K6u0W2L8h3b7`.)中查阅，我就不做过多描述了。

```cpp
#include "what_you_need_for_your_target.h"
__AFL_FUZZ_INIT();
int main() {
// anything else here, e.g. command line arguments, initialization, etc.
#ifdef __AFL_HAVE_MANUAL_CONTROL
__AFL_INIT();
#endif
unsigned char *buf = __AFL_FUZZ_TESTCASE_BUF; // must be after __AFL_INIT
// and before __AFL_LOOP!
while (__AFL_LOOP(10000)) {
int len = __AFL_FUZZ_TESTCASE_LEN; // don't use the macro directly in a
// call!
if (len < 8) continue; // check for a required/useful minimum input length
/* Setup function call, e.g. struct target *tmp = libtarget_init() */
/* Call function to be fuzzed, e.g.: */
target_function(buf, len);
/* Reset state. e.g. libtarget_free(tmp) */
}
return 0;
}
```

## Patch

大多数时候，我们fuzzing一个目标想要其达到我们预期的效果，都需要Patch。并且我们在后续fuzzing流程的持续改进中可能还会发现一些影响fuzzing效率的地方，我们又会倒回来patch，编译重新启动fuzzing。

此外，有时候一些校验、检查，它们往往对于fuzzing的结果没有什么影响，但是却严重影响fuzzing的效率。此时我们通常会审查目标内部代码，将这些严重性fuzzing效率的地方Patch，或者是删除。

我们都知道Persistent Mode收益十分巨大，但却要求Persistent循环区域内的代码是无状态的，有时候区域会有一些有状态的函数，但他们却并不重要，这时你可以Patch它们，使它们返回诸如硬编码之类可以，这样就变成无状态的，我们就可以使用Persistent Mode了。

例如一个区域的输入可能依赖于socket IO读入，而处理socket IO是很麻烦的，因此我们可以考虑将socket fd替换为文件 fd，并patch那些受socket fd受影响区域，以便我们fuzzing正确运行。

简言之，Patch最好有明确的理由，随意的Patch对模糊测试来说可能会导致很糟糕的现象，要么你对此处的Patch是基于改进fuzzing效率，要么是为了启用某些有益的fuzzing功能....总之，最好清楚自己的Patch是为了什么。

但请注意，对于一次模糊测试来说Patch只是可选的，如果你对自己的工具、目标不甚了解，那么Patch对你而言可能不重要。如果你清楚目标的内部结构，并且明确知道要改进fuzzing的流程和目的，那么Patch可能是你定制化自己fuzzing的一个重要手段。

# 后续

目前就先写到着，后面的内容，包括build、fuzzing、评估、流程改进等等就放到下篇，最近的工作可能要忙一些其他的。

不知道为什么，看雪不支持md语法了，我也没找到怎么支持md，如果你对我的文章感兴趣可以去我的blog上查阅：

[070K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5%4R3I4L8X3N6Q4x3V1k6H3i4K6u0r3x3e0M7#2z5o6V1%4z5e0S2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@20eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5%4R3I4L8X3N6Q4x3V1k6H3i4K6u0r3x3e0M7#2z5o6V1%4z5e0S2Q4x3X3g2Z5N6r3#2D9)

> 适用于白盒fuzzing

# input corpus

## 收集语料库

对于模糊测试工具而言，我们需要为其准备一个或多个起始的输入案例，这些案例通常能够很好的测试目标程序的预期功能，这样我们就可以尽可能多的覆盖目标程序。

收集语料的来源多种多样。通常目标程序会包含一些测试用例，我们可以将其做位我们初始语料的一部分，此外互联网上也有些公开的语料库你可以收集他们做为你的需要。

关于语料库的主动性选择，这个更多需要你对fuzzing 目标内部结构的了解。例如你当你要fuzzing的目标对随着输入的规模内存变化非常敏感，那么制作一批很大的文件与较小的文件可能是一个策略，具体是否是否有效取决于你经验、以及对目标的理解。

此外，需要注意控制语料库的规模，太过庞大的语料库并不是好的选择，太过潘达的语料库会拖慢fuzzing的效率，尽可能用相对较小的语料覆盖更多目标代码的预期功能即可。

## 语料库唯一化

我们在上一小节最后提到一点，太过庞大的语料库会因为有太多的测试用例重复相同的路径覆盖，这会减慢fuzzing的效率。因此人们制作了一个工具，能够使语料库覆盖的路径唯一化，简单的说就算去除重复的种子输入，缩减语料库的规模，同时保持相当的测试路径效果。

在AFL++中可以使用工具 `afl-cmin` 从语料库中去除不会产生新路径和覆盖氛围的重复输入，并且AFL++官方提示强烈建议我们对语料库唯一化，这是一个几乎不会产生坏处的友谊操作。

具体的使用如下：

1. 将收集到的所有种子文件放入一个目录中，例如 INPUTS
2. 运行 afl-cmin：

|   |   |
| --- | --- |
| 1 2 3 4 5 6 7 | `# 如果要通过模糊测试来调用目标程序 bin/target INPUTFILE ，请将目标程序将从中读取的 INPUTFILE 参数替换为 @@` `afl-cmin -i INPUTS -o INPUTS_UNIQUE -- bin` `/target` `-someopt @@` `# 如果目标从 stdin（标准输入）读取，只需省略 ， @@ 因为这是默认值：` `afl-cmin -i INPUTS -o INPUTS_UNIQUE -- bin` `/target` `-someopt` |

## 字典

其实将字典放到这一个大节下面不是很合适，因为字典可以归类为一种辅助技巧，不过因为字典影响输入，所以我就将其划到这里了。

关于是否使用字典，取决于fuzzing的目的与目标。例如fuzzing的目标是ftp服务器，我们fuzzing的目的是站在用户的视角仅能输入命令，因此我们的输入其中很大一部分可以规范到ftp提供的命令，我们更多的是通过重复测试各种命令的组合来测试目标ftp服务器在各种场景都能正确运行。

又比如，当你fuzzing一个很复杂的目标时，它通常提供一个非常非常丰富的命令行参数，每一次运行时组合不同的参数可能会有更好的覆盖效果，因此可以将你需要启用的参数标记为字典添加进命令行参数列表中。

最后，目标程序可能经常有常量的比较和验证，而这些环节通常会使得fuzzing停滞在此，因为模糊器的变异策略通常对应常量的猜测是非常低效的。我们可以收集目标程序中使用到的常量，定义为一个字典提供给模糊器。但目前对于AFL++来说有更好的方法解决这种需求，而无需定义字典，后面我们会介绍这些方法。

|   |   |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 | `# 模糊器默认的变异策略通常难以命中if分支为true的情况，因为input做为64位，其值的空间太大了，根本难以猜测。` `if` `(input = 0x1122336644587) {` `crash();` `}` `else` `{` `OK();` `}` |

# 编译前的准备

## 选择最佳的编译器

如我们上一节中谈到收集程序常量定义字典时，事实上收集常量并生成字典这个事情，在编译时完全可以顺便将其解决。没错，功能强大的编译器可以使我们在编译期间获得非常多有用的功能。对于AFL++的编译器选择，官方提供了一个简单的选择流程，如下

|   |   |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 | `+--------------------------------+` `| clang` `/clang` `++ 11+ is available | --> use LTO mode (afl-clang-lto` `/afl-clang-lto` `++)` `+--------------------------------+ see [https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.lto.md](https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.lto.md)` `|` `|` `if` `not, or` `if` `the target fails with LTO afl-clang-lto/++` `|` `v` `+---------------------------------+` `| clang` `/clang` `++ 3.8+ is available | --> use LLVM mode (afl-clang-fast` `/afl-clang-fast` `++)` `+---------------------------------+ see [https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.llvm.md](https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.llvm.md)` `|` `|` `if` `not, or` `if` `the target fails with LLVM afl-clang-fast/++` `|` `v` `+--------------------------------+` `| gcc 5+ is available | -> use GCC_PLUGIN mode (afl-gcc-fast` `/afl-g` `++-fast)` `+--------------------------------+ see [https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.gcc_plugin.md](https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.gcc_plugin.md) and` `[https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.instrument_list.md](https://github` `.com` `/AFLplusplus/AFLplusplus/blob/stable/instrumentation/README` `.instrument_list.md)` `|` `|` `if` `not, or` `if` `you` `do` `not have a gcc with plugin support` `|` `v` `use GCC mode (afl-gcc` `/afl-g` `++) (or afl-clang` `/afl-clang` `++` `for` `clang)` |

若你的LLVM和clang版本大于等于11，那么你可以启用LLVM LTO模式，使用`afl-clang-lto`/`afl-clang-lto++`，该模式通常是最佳的。随后依次是`afl-clang-fast`/`afl-clang-fast++`和`afl-gcc-fast`/`afl-g++-fast`。
