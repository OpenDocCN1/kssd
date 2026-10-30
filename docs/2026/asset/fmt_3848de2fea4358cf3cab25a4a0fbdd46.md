
AFL实现的这套 `fork server` 机制就是为了提高效率，把 `fork` 相关的操作集成起来，`fuzzer`不需要对`fork`负责，只需要与`fork server`交互即可。关于这部分的具体运行原理在之后会有详细说明。

#### 3. 目录处理 && 环境检查

对输入目录输出目录进行检查，检查cpu，检查内核数量等等。

### 【四】[文件/数据]变异

正好这里涉及了对输入文件的变异策略，可以单独先提一下。在AFL的fuzz过程中，维护了一个 testcase 队列，每次把队列里的文件取出来之后，对其进行变异，下面就讲一下各个阶段的变异是怎样的。

注：`bitflip`、`arithmetic`、`interest`、`dictionary` 是 `deterministic fuzzing` 过程，属于 dumb mode（`-d`）和主 fuzzer（`-M`）会进行的操作；`havoc`、`splice` 与前面不同是存在随机性，是所有 fuzz 都会进行的变异操作。文件变异是具有启发性判断的，应注意“避免浪费，减少消耗”的原则，即之前变异应该尽可能产生更大的效果，比如 `eff_map` 数组的设计；同时减少不必要的资源消耗，变异可能没啥好效果的话要及时止损。

#### 1. `bitflip`，位反转，顾名思义按位进行翻转，0变1，1变0。

> `STAGE_FLIP1` 每次翻转一位（`1 bit`），按一位步长从头开始。

> `STAGE_FLIP2` 每次翻转相邻两位（`2 bit`），按一位步长从头开始。

> `STAGE_FLIP4` 每次翻转相邻四位（`4 bit`），按一位步长从头开始。

> `STAGE_FLIP8` 每次翻转相邻八位（`8 bit`），按八位步长从头开始，也就是说，每次对一个byte做翻转变化。

> `STAGE_FLIP16` 每次翻转相邻十六位（`16 bit`），按八位步长从头开始，每次对一个word做翻转变化。

> `STAGE_FLIP32` 每次翻转相邻三十二位（`32 bit`），按八位步长从头开始，每次对一个dword做翻转变化。

这一部分在 `5135` 行的 `#define FLIP_BIT(_ar, _b) do {***}` 有详细的代码实现。

##### `token` - 自动检测

源码中有一段关于这部分的注释，意思是说在进行为翻转的时候，程序会随时注意翻转之后的变化。比如说，对于一段 `xxxxxxxxIHDRxxxxxxxx` 的文件字符串，当改变 `IHDR` 任意一个都会导致奇怪的变化，这个时候，程序就会认为 `IHDR` 是一个可以让fuzzer很激动的“神仙值”--token。

> / *While flipping the least significant bit in every byte, pull of an extra trick to detect possible syntax tokens. In essence, the idea is that if you have a binary blob like this:
>
> `xxxxxxxxIHDRxxxxxxxx`
>
> ...and changing the leading and trailing bytes causes variable or no changes in program flow, but touching any character in the "IHDR" string always produces the same, distinctive path, it's highly likely that "IHDR" is an atomically-checked magic value of special significance to the fuzzed format.
>
> We do this here, rather than as a separate stage, because it's a nice way to keep the operation approximately "free" (i.e., no extra execs).
>
> Empirically, performing the check when flipping the least significant bit is advantageous, compared to doing it at the time of more disruptive changes, where the program flow may be affected in more violent ways.
>
> The caveat is that we won't generate dictionaries in the `-d` mode or `-S` mode - but that's probably a fair trade-off.
>
> This won't work particularly well with paths that exhibit variable behavior, but fails gracefully, so we'll carry out the checks anyway.*/

其实token的长度和数量都是可以控制的，在 `config.h` 中有定义，但是因为是在头文件宏定义的，修改之后需要重新编译使用。

> / *Maximum number of auto-extracted dictionary tokens to actually use in fuzzing (first value), and to keep in memory as candidates. The latter should be much higher than the former.* /

```
#define USE_AUTO_EXTRAS 50
#define MAX_AUTO_EXTRAS (USE_AUTO_EXTRAS * 10)
```

##### `effector map` - 生成

在这里值得一提的是 `effector map`，在看源码数据变异这一部分的时候，一定会注意的是在 bitflip 8/8 的时候遇到一个叫 `eff_map` 的数组，这个数组的大小是 `EFF_ALEN(len)`（也就是【？？】），数组元素只有 0/1 两种值，很明显是标记的意思，到底是标记什么呢？

要想明白 `effector map` 的原理需要了解三个点：

1. 为什么是 8/8 的时候出现？因为 `8bit`（比特）的时候是 `1byte`（字节），如果一个字节的翻转都无法带来路径变化，此byte极有可能是不会导致crash的数据，所以之后应该用一种思路避开无效byte。

2. 标记是干什么用的？根据上面的分析，就很好理解了，标记好的数组可以为之后的变异服务，相当于提前“踩雷（踩掉无效byte的雷）”，相当于进行了启发式的判断。无效为0，有效为1。

3. 达到了怎样的效果？要知道判断的时间开销，对不停循环的fuzzing过程来说是致命的，所以 `eff_map` 利用在这一次8/8的判断中，通过不大的空间开销，换取了可观的时间开销。(暂时是这样分析的，具体是否真的节约很多，不得而知)

#### 2. `arithmetic`，算术，实际操作就是加加减减。

`bitflip` 结束之后，就进入 `arithmetic` 阶段，目标大小和阶段与 `bitflip` 非常类似：

> arith 8/8，每次8bit进行加减运算，8bit步长从头开始，即对每个byte进行整数加减变异；

> arith 16/8，每次16bit进行加减运算，8bit步长从头开始，即对每个word进行整数加减变异；

> arith 32/8，每次32bit进行加减运算，8bit步长从头开始，即对每个dword进行整数加减变异；

其中对于加减变异的上限，在 `config.h` 中有所定义：

> / *Maximum offset for integer addition / subtraction stages:* /

```
#define ARITH_MAX 35
```

注：跟 `bitflip` 相同的，如果需要修改此值，在头文件中修改完之后，要进行编译才会生效。

在这里对整数目标进行+1，+2，+3...+35，-1，-2，-3...-35的变异。由于整数存在大端序和小端序两种表示，AFL会对这两种表示方式都进行变异。

前面也提到过AFL设计的巧妙之处，AFL尽力不浪费每一个变异，也会尽力让变异不冗余，从而达到快速高效的目标。AFL会跳过某些arithmetic变异：

1. 在 `eff_map` 数组中对byte进行了 0/1 标记，如果一个整数的所有 bytes 都被判为无效，那么就认为整数无效，跳过此数的变异；

2. 【？？】如果加减某数之后效果与之前某bitflip效果相同，认为此次变异在上一阶段已经执行过，此次不再执行；

#### 3. `interest`，把一些“有意思”的特殊内容替换到原文件中。

`interest` 的三个步骤跟 `arithmetic` 相同：

> interest 8/8，每次8bit进行加减运算，8bit步长从头开始，即对每个byte进行替换；

> interest 16/8，每次16bit进行加减运算，8bit步长从头开始，即对每个word进行替换；

> interest 32/8，每次32bit进行加减运算，8bit步长从头开始，即对每个dword进行替换；

用于替换的叫做 `interesting values`，是AFL预设的特殊数：

> / *Interesting values, as per config.h* /

```
static s8 interesting_8[] = { INTERESTING_8 };
static s16 interesting_16[] = { INTERESTING_8, INTERESTING_16 };
static s32 interesting_32[] = { INTERESTING_8, INTERESTING_16, INTERESTING_32 };
```

其中 `interesting values` 在 `config.h` 中的设定：

```
/* List of interesting values to use in fuzzing. */

#define INTERESTING_8 \
  -128,          /* Overflow signed 8-bit when decremented  */ \
  -1,            /*                                         */ \
   0,            /*                                         */ \
   1,            /*                                         */ \
   16,           /* One-off with common buffer size         */ \
   32,           /* One-off with common buffer size         */ \
   64,           /* One-off with common buffer size         */ \
   100,          /* One-off with common buffer size         */ \
   127           /* Overflow signed 8-bit when incremented  */

#define INTERESTING_16 \
  -32768,        /* Overflow signed 16-bit when decremented */ \
  -129,          /* Overflow signed 8-bit                   */ \
   128,          /* Overflow signed 8-bit                   */ \
   255,          /* Overflow unsig 8-bit when incremented   */ \
   256,          /* Overflow unsig 8-bit                    */ \
   512,          /* One-off with common buffer size         */ \
   1000,         /* One-off with common buffer size         */ \
   1024,         /* One-off with common buffer size         */ \
   4096,         /* One-off with common buffer size         */ \
   32767         /* Overflow signed 16-bit when incremented */

#define INTERESTING_32 \
  -2147483648LL, /* Overflow signed 32-bit when decremented */ \
  -100663046,    /* Large negative number (endian-agnostic) */ \
  -32769,        /* Overflow signed 16-bit                  */ \
   32768,        /* Overflow signed 16-bit                  */ \
   65535,        /* Overflow unsig 16-bit when incremented  */ \
   65536,        /* Overflow unsig 16 bit                   */ \
   100663045,    /* Large positive number (endian-agnostic) */ \
   2147483647    /* Overflow signed 32-bit when incremented */
```

可以看到，基本是些会造成溢出的数值。与前面的思想相同的，本着“避免浪费，减少消耗”的原则，`eff_map`数组中已经判定无效的就此轮跳过；如果 `interesting value` 达到的效果跟 `bitflip` 或者 `arithmetic` 效果相同，也被认为是重复消耗，跳过。

#### 4. `distionary`，字典，会把自动生成或者用户提供的token替换、插入到原文件中。

此阶段已经是确定性变异 `deterministic fuzzing` 的结尾：

> user extras (over)，从头开始，将用户提供的tokens依次替换到原文件中

> user extras (insert)，从头开始，将用户提供的tokens依次插入到原文件中

> auto extras (over)，从头开始，将自动检测的tokens依次替换到原文件中

其中 “用户提供的tokens” 是一开始通过 `-x` 选项指定的，如果没有则跳过对应的子阶段；“自动检测的tokens” 是第一个阶段 `bitflip` 生成的。

#### 5. `havoc`，“大破坏”，对原文件进行大量变异。

`havoc` 意味着随机的开始，与后面的 `splice` 是任何模式下都要进行的变异，具体来说 `havoc` 包含了多轮变异，每一轮都是组合拳：

> 随机选取某个bit进行翻转

> 随机选取某个byte，将其设置为随机的interesting value

> 随机选取某个word，并随机选取大、小端序，将其设置为随机的interesting value

> 随机选取某个dword，并随机选取大、小端序，将其设置为随机的interesting value

> 随机选取某个byte，对其减去一个随机数

> 随机选取某个byte，对其加上一个随机数

> 随机选取某个word，并随机选取大、小端序，对其减去一个随机数

> 随机选取某个word，并随机选取大、小端序，对其加上一个随机数

> 随机选取某个dword，并随机选取大、小端序，对其减去一个随机数

> 随机选取某个dword，并随机选取大、小端序，对其加上一个随机数

> 随机选取某个byte，将其设置为随机数

> 随机删除一段bytes

> 随机选取一个位置，插入一段随机长度的内容，其中75%的概率是插入原文中随机位置的内容，25%的概率是插入一段随机选取的数

> 随机选取一个位置，替换为一段随机长度的内容，其中75%的概率是替换成原文中随机位置的内容，25%的概率是替换成一段随机选取的数

> 随机选取一个位置，用随机选取的token（用户提供的或自动生成的）替换

> 随机选取一个位置，用随机选取的token（用户提供的或自动生成的）插入

之后AFL会生成一个随机数，作为变异组合的数量，每次从上面随机选取作用于当前文件。

#### 6. `splice`，“拼接”，两个文件拼接到一起得到一个新文件。

具体来说，AFL会在文件队列中随机选择一个文件与当前文件进行对比，如果差别不大就重新再选；如果差异明显，就随机选取位置两个文件都一切两半。最后将当前文件的头与随机文件的尾拼接起来得到新文件【为什么不是当前的尾拼接随机文件的头【？？】】。当然了本着“减少消耗”的原则拼接后的文件应该与上一个文件对比，如果未发生变化应该过滤掉。
