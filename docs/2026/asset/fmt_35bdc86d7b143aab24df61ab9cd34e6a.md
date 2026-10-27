
* 上面这两点算是一点小遗憾，而且不是那么好修复

### 插件出炉

* 上述的所有过程手工起来还是比较麻烦的，我们做成了插件，你可以在[](<mark class="encrypted">60dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9f1p5W2e0c8f1y4Q4x3V1k6D9L8s2k6E0i4K6u0V1k6r3g2G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6t1&6i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1$3i4K6R3&6i4@1u0q4i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1$3i4@1u0m8i4K6V1H3i4@1f1%4i4@1p5H3i4K6R3I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1q4r3i4@1t1%4i4@1f1$3i4@1t1K6i4@1p5^5i4@1f1$3i4K6R3@1i4K6S2r3i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1&6i4K6R3%4i4K6S2o6b7X3W2F1j5i4u0&6</mark> Ninja插件，直接clone下来就可以用了。

* 使用插件来还愿代码只需要2步：

1. 选择一条更新`state_var`的指令
2. 执行插件

* 下面是一个演示视频：

![](upload/attach/201902/3_E2J626T3VRDHKV8.gif)

### 译者

* 原文地址：<a href="elink@54fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7Y4m8A6M7#2)9J5k6h3g2U0i4K6u0r3x3U0l9I4y4#2)9J5c8U0p5J5i4K6u0r3k6r3W2K6M7$3g2U0N6r3W2G2L8W2)9J5k6r3I4D9N6X3#2Q4x3X3c8G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1M7o6q4Q4x3X3g2Z5N6r3#2D9"><mark class="encrypted">84dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7Y4m8A6M7#2)9J5k6h3g2U0i4K6u0r3x3U0l9I4y4#2)9J5c8U0p5J5i4K6u0r3k6r3W2K6M7$3g2U0N6r3W2G2L8W2)9J5k6r3I4D9N6X3#2Q4x3X3c8G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1M7o6q4Q4x3X3g2Z5N6r3#2D9</mark></a>

* 本文由 看雪翻译小组 `freakish@敌法` 提供翻译

* 感谢 看雪 `Eddie` 提供原文

* 翻译理解过程中参考了一些csdn相关文章， 一并感谢各位

## LLVM代码混淆分析及逻辑还原

### 概述

- LLVM Obfuscator是一款工业级别的代码混淆器，在过去几年的CTF里我们经常会遇到经过代码经过它混淆的情况。这片博文记录了我们对混淆器原理的研究以及从中发现的有关混淆器的设计实现的脆弱之处。基于我们的研究结果，我们在Binary Ninja平台上写了一个插件，通过这个插件可以自动化的解决掉由于代码混淆带来的逆向分析困难。

### LLVM Obfuscator简介

- LLVM Obfuscator是一个基于LLVM框架实现的一个开源代码混淆器，整个项目包含了三个相对独立的LLVM pass, 每个pass实现了一种混淆方式，通过这些混淆手段，可以模糊原程序的流程或者某一部分的算法，给逆向分析带来一些困难。

- 由于上述的三个pass是基于LLVM IR实现的, 因此从理论上来说， 这种混淆器是支持世界上任何一种语言和机器架构的。

- 关于每种pass的详细文档，可以查看下面的这三个链接：

  1. [Instructions Substitution(指令变换)](https://bbs.kanxue.com/elink@6eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6%4K9h3E0A6i4K6u0r3d9h3&6K6N6s2u0#2j5%4c8A6L8$3&6K6i4K6u0V1f1%4g2T1M7%4c8A6N6s2g2@1K9h3!0F1)

  2. [Bogus Control Flow(流程伪造)](https://bbs.kanxue.com/elink@99cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6%4K9h3E0A6i4K6u0r3b7X3!0Y4N6i4y4Q4x3X3c8o6L8$3&6@1M7X3!0D9i4K6u0V1c8X3I4G2N6H3`.`.)

  3. [Control Flow Flattening(流程平坦化)](https://bbs.kanxue.com/elink@c99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6%4K9h3E0A6i4K6u0r3b7$3!0F1N6s2u0G2L8q4)9J5k6p5k6D9L8%4N6Q4x3X3c8r3L8r3q4@1N6r3g2F1K9h3&6Y4)

- 上面的这几个链接里面是各个pass的作者维护的一份简单文档，如果你觉得文档不够详尽，建议直接参考相应的源码即可，可能对你来说会又直观又准确。

- 如果说看代码，其实是比较费劲的一个事情，主要是LLVM Obfuscator的工程代码结构的原因。现在github上，LLVM Obfuscator是按分支来维护的，每个版本一个分支，也就是说你Clone下来的代码都杂在一起，直接上来就看代码很容易迷失在代码的海洋中。不过我们可以有目的的挑着看，比如我们Clone一份[4.0](https://bbs.kanxue.com/elink@c5eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3i4K6u0r3L8r3W2T1i4K6u0r3g2s2u0S2L8Y4y4X3L8%4u0E0M7#2)9J5c8V1!0T1k6Y4g2K6j5$3q4@1K9h3!0F1)的代码，然后直接在 `lib/Transforms`目录下的代码， 这里都是自定义的LLVM pass。

- **在我们这篇博文里面，我们只关注流程平坦化**这一个主题，这个特性在我看来是比较有趣，并且混淆效果也是比较理想的一个特性。

### 控制流程平坦化

- 总体来说，控制流程平坦化这个特性，抽象下来，主要是通过这几个步骤来实现的：

  1. 在整个代码流程中，分析搜集出所有的基本代码块(Basic Block)（译者注：遇到条件分支就算是一个新的代码块了）
  2. 把基本代码块放到控制流图的最底部，然后删除掉原来的基本块之间的跳转关系
  3. 添加混淆器的流程控制分发逻辑，通过新的复杂分发逻辑还原原来程序块之间的逻辑关系

- 还是举个例子吧，为了形象一点，我这里给出两幅图来进行左右对比。

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_C7D67HCK7KYTFFE.png)

- 左边的图是IDA7.0（Demo版就行）对未混淆程序生成的代码流程图，右图是同一个程序经过LLVM Obfuscator的流程平坦化处理之后IDA7.0分析出的代码流程图。

- 在这两幅图里面，绿色的块表示函数里面的代码基本块，图中的蓝色的块就是混淆器为了达到混淆效果和保持原程序逻辑而添加的粘合代码，这里我们给这些蓝色块的代码起个名字好了，叫它 **backbone**(混淆器运行框架)

- 对于右边这幅图，为了看起来更加的直观，我们可以使用IDA的node分组的功能把流程图的显示方式优化一下，这里我直接把backbone代码合并成一个node，这样看起来就清晰了，看图：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_XG7PXBQ5TZYVC73.png)

- 虽然现在流程图简单了不少，但是通过和上面的左图进行对比， 整个程序流程还是发生了很大的变化，并且各个基本块之间的逻辑关系也很难判断了，整个代码流程看上去更像是一个switch...case结构，每个基本块是case分支逻辑。

- 由此我们也可以这样想，整个逻辑流程变成了一个状态机架构，每次执行哪个代码块由状态机的值来决定，而每个代码块最后会更新状态机的值，然后backbone框架代码根据这个值，再来决定执行哪个基本代码块，**所以一个代码块肯定要对应一个固定的状态机的值。**

### 流程平坦化的弱点

- 从现在开始，我们开始借助 Binary Ninja这个平台来进行后续的分析，选择这个平台主要是基于这个平台里的几个特性（IDA中没有）：

  1. Medium-level IL
  2. SSA Form
  3. Value-set analysis

#### 确定Backbone块（确定骨架代码）

- 为了搞清楚流程平坦化的弱点，我们通过一个例子来详细的分析一下Backbone的代码，先看下我们的例子：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_33WJBR9ATCMDV24.png)

  这个例子就是我们上面的那个经过混淆处理的程序的一部分，其他部分的代码基本是相似的，因此这里我们就截取其中一部分代表就可以了。我们仔细观察这段代码，**这段代码会读取状态变量，然后把变量和某个值进行比较，如果比较相等，就跳转到某个基本块执行，如果不等，就跳转到下一个Backbone里面继续上述的过程**

- 注意，这里我们就发现了一个关键的脆弱点：**给定一个状态变量，记为`state_var`，我们发现每个Backbone代码块至少包含一次对这个变量的引用，如果遍历出所有引用到这个变量的代码块，那我们就可以得到所有的Backbone块**，下面我们通过Binary Ninja的medium-level IL特性来搜集所有的块，这里我直接给出代码：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_2M6664X7UZNZQE3.png)

- 这个算法可以找出所有对`state_var`进行过引用的Backbone块，包括程序的起始块（这个块是定义这个变量的块），起始块一般是这样的：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_6C8QJUPB6QGTZP9.png)

- 从起始块我们很容易找到这个状态变量，然后通过def-use和use-def调用链，就能比较顺利的找到剩余的Backbone块了。

#### 确定真实的程序逻辑块

- 通过类似的思想，我们看看能不能找到什么特征，通过这个特征来找到所有的逻辑块。在我们的这个例子里面，一个真实基本块会包含一个或者多个执行出口，而执行出口一般都是以一个无条件跳转实现的，一个比较典型的真实块看起来大致是这样的：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_7HDWP7V4M5TVMDY.png)

- 看代码可以知道，先是修改一下状态变量`state_var`的值，然后跳转到骨架代码。看上面的代码，我们基本可以确定，下次执行的真实代码块对应的case值是`0xfcbce33c`，对于那种有多个出口的真实块会被拆分成多个块，看起来会大致像下面这样：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_WTQ3BWJBAP629BC.png)

- 这里，原程序的一个条件语句其实被转换成了一个赋值语句，然后根据赋值的结果决定是不是要执行某个代码块，举个例子来说，比如原程序是这样的(^_^一起截图了，下面的是变化后的结果)：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_NWYJ7CQQ4RSZMHQ.png)

- 但是，我们的目标是找到所有的真实代码块，为了达到这个目标，我们需要利用LLVM Obfuscator的另一个关键弱点：**所有逻辑基本块中，每个块至少包含一次`state_var`的定义动作（注意是定义不是引用）**，就跟起始块有点类似。

- 乍一看，可能我们要基于深度优先来进行一次def-use类型的搜索，不过在Ninja上，这个工作被简化了不少，前面一个小节里面，我们查找了所有使用到了`state_var`的代码块，但是在这里我们只查找定义了这个变量的代码块就好了，代码如下：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_KYQERVPVVRE4MZ5.png)

- 上面的代码里面找到的每个包含了`state_var`变量定义的代码块都被认为是一个基本的逻辑代码块，包含起始块。后面的章节里面会发现，这种特征方式得到的结果还是很令人满意的。

#### 还原代码流程

- 到现在为止，我们基本上已经有了重构代码流程的所有信息，再梳理一下的话，我们现在对于一个经过混淆过的程序，目前掌握了以下两点信息：

  1. 所有的代码基本块
  2. 状态机值和基本代码块的映射关系

- 目前我们还差一步，那就是我们目前还不知道对于一个给定的基本代码块，它的下一个执行节点是什么，也就是我们需要确定一个基本代码块执行完毕的时候，它把状态机的值改成了什么。为了完成这个目标，我们就要借助Binary Ninja的另一个很重要的特性，Value-Set分析，通过这个分析，可以知道某个寄存器或者内存位置里面的值是什么（译者注：相当于一个值跟踪系统，有点模拟执行的味道），有了这个，我们也可以确定出来最后状态机的值了。

- 前面提到过，一个基本代码块最后会把状态机`state_var`更新成一个固定值，现在我们就把这些值都找出来，这样整条链就串起来了：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_E8WNMW94NDWDAPZ.png)

- 对于有条件跳转的情况，处理的方式有点trick的味道，由于我们的目标是要确定基本块执行完毕的时候，出口的`state_var`的值是什么，也就是要确定条件跳转的时候，哪个是true, 哪个是false, 为了方便，我们使用Ninja的 SSA 图形视图来观察一下，看下面这个例子：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_SAJ9ZFQ4UTDY476.png)

- 在这个例子里面，函数执行完毕的时候，是有两种情况的的`state_var`的，在上图里面，凡事会影响到`state_var`相关的语句全都高亮了。为了更加的直观，这里把上述的逻辑用 LLVM-IR再描述一下：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_Z3HJVHHFBU8GVVA.png)

- 大致逻辑就是：

  1. 先把`%next_state`设置成`%false_branch`
  2. 如果`%original_condition`的结果为1， 在把`%next_state`设置成`%true_branch`
  3. 最后再把state结果保存到`%state`变量

- 回头观察上图中的SSA-MLIL, 我们看下高亮的语句部分，其实就是在两个不同版本的nextState变量之间进行选择，而且每个不同版本的nextState后面跟着一个数字作为版本号标志，再根据我们分析的逻辑，版本号小的那个就是false，版本号大的是true.最后借助于Ninja的Value-Set分析，我们就可以得出最后的nextState的最终值，所以就能确定下一个要执行的基本块是哪个了，还是上代码：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_ZRH5B336YFN9VEB.png)

- 现在版本的API还不太方便，但是结果还是准确的，把上面的东西综合起来，就形成了一个比较完整的脚本了：

  ![图片描述](https://bbs.kanxue.com/upload/attach/201902/3_A29GU2W93KTXU83.png)

### 重构干净的二进制文件

- 目前来说，在Binary Ninja平台上patch程序还是比较麻烦的，但是好像也没有什么更好的替代品了。到现在为止，我们已经能够重构原始的控制流程了，剩下的工作就是根据掌握的信息来patch二进制代码了，让真实逻辑代码块之间直接相连，忽略掉Backbone代码。

#### 构建代码原型
