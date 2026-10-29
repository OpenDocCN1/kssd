
# [原创]ollvm快速学习

近来，ollvm在国内移动安全，尤其是安全加固上的使用越来越广泛，ollvm的混淆和反混淆也被视为比较高等的知识之一，让很多人感到无从下手，望尘莫及。如果你在google上搜索ollvm，你会发现第一页都是中文的搜索结果。其实，llvm和ollvm在国外是比较传统的东西，说到底也只是C++代码，难度大概等同于ART系统源码的程度。

本篇文章目地是为了从另一个从未出现的角度来让一个完全不懂llvm的新手快速上手ollvm，大神请直接跳过。

注意：本文因角度不同，若引起误会，纯属个人理解不同，本人不会作出任何解释，请谅解。

## 1. 快速理解llvm、clang和ollvm的概念

我们不使用网上那些冗杂的阐述，一句话概括，llvm是一个完整的编译器架构，作用可以理解为制作一个编译器，llvm先将源码生成为与目标机器无关的`LLVMIR`代码，然后把`LLVMIR`代码先优化，再向目标机器的汇编语言而努力。经典编译器都可以分为前端、中层优化和后端：

![](https://bbs.kanxue.com/upload/attach/201901/3_EAE47Q4U4D7NU5H.jpg)

我们从上图也理解了clang，是前端的一个套件，但在实际使用时，你会感觉到，我们只可以感受到clang，也只是在使用clang，因为编译的时候，是调用`clang`或`clang++`来编译源码。Ollvm呢？是基于LLVM的代码分支的代码混淆，对谁混淆？什么时候混淆？在中间表示`IR`层，通过编写`pass`（英文翻译：经过，自己理解，也不需要知道其正规的概念）来混淆`IR`，这样目标机器的汇编语言也就被混淆了：

![](https://bbs.kanxue.com/upload/attach/201901/3_WV6QCZ8HVUYUUP9.jpg)

当然，这里需要知道的是，即使不用ollvm，LLVM本身也是有很多`pass`的，我们这样简单的理解，`LLVMIR`本身是一种与目标机器无关的虚拟化代码，而在转化为真实汇编代码时，肯定要删除一些虚拟的东西，自然也需要`pass`。这样，我们就用不到200字的阐述完成了网上很多长篇大论才能达到的理解。

## 2、llvm与ollvm在移动加固的发展

在开始制作安卓vmp时，llvm其实是被我们暂时放弃的一个方案。因为从`smali`上想可以使用llvm，还必须先克服`smali`到底怎么成为C/C++，当时他被提及，很大原因是安卓系统后端使用了llvm，但其实ART下和llvm关系已经不大了。当时都急于上架vmp，因此大部分都是采用折衷策略，如自定义`dex`结构、置换指令等。随着发展，`smali2c`渐渐成熟，可以`smali2c`了，自然ollvm是必然的一个选择。但一个新的问题出现了，就像很多加固网站，需要提供源码，或编译过程中的中间文件，说到底还是源码加固。

所以想做二进制加固的ollvm分支，需要注意一下。想做到二进制加固，没有一个自己的反汇编解析引擎是不可能的，目前用`capstone`比较多。但要真正做自己的二进制VMP加固，首先你得有一个反汇编引擎能把指令抽出来，同时转为了自己的虚拟指令，如果接上了llvm，最完美的方案就是把`LLVMIR`给虚拟为自己的虚拟指令，这个难度相当于把.netframwork的`IL`指令给虚拟了。可以先逃避了这两个问题，ARM指令得到后，利用简单的函数式指令解析来完成这个虚拟过程。我们认为，如果只有`LLVMIR`，如ollvm，是没有虚拟的，只是混淆。

## 3、利用beyondcompare+sourceinsight4的ollvm快速上手

Ollvm如果你去网上搜索资料学习，大概是这么几种文章：“LLVM编译器架构王者-编写简单的C虚拟编译器”、“ollvm+ndk编译环境的搭建”、“从0开始学习LLVM”、“LLVM PASS的完整编写”，当然这些文章都很不错。但对于一个新手来说，看了和没看其实区别不大。因为我们要学习ollvm，必须先抓住关键，到底一个llvm是如何变为ollvm，这是最简单，最直观的学习方式。

我们去下载一份llvm4.0源码（官网d19K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0W2L8r3I4$3L8g2)9J5k6h3!0J5k6#2)9J5c8U0c8Q4x3X3f1H3i4K6u0W2x3g2)9J5c8X3I4D9N6X3#2Q4x3X3b7@1i4K6u0W2x3q4)9J5k6e0q4Q4x3X3g2K6M7X3y4Q4x3X3g2@1j5i4u0Q4x3X3g2^5P5W2!0q4c8W2!0n7b7#2)9^5z5g2!0q4y4g2)9&6x3W2)9^5b7$3!0D9L8s2k6E0i4@1f1$3i4@1u0m8i4K6V1H3i4@1f1%4i4@1p5H3i4K6R3I4i4@1g2r3i4@1u0o6i4K6R3^5[04aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3](https://bbs.kanxue.com/elink@4eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3)），之所以使用4.01，因为4.0.0是2017.03的，而4.01相对时期接近一点。然后拉入beyondcompare，这里把会话设置取消勾选“比较时间戳”，以及把比较设置为“仅文件”，时间戳因为比较时间不是我们的重点，而之所以不采用原先的文件结构比较，是因为两者都有一些空文件夹，没有比较的意义。如图：

![](https://bbs.kanxue.com/upload/attach/201901/3_9NZQ2Q4MZE3WTYT.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/3_M7UH3DT8EPACJN7.jpg)

看来主要的差别就在这些文件夹里了。先看最外层的`CMakeLists.txt`（整个工程使用`cmake`编译）等3个文件：基本都是加入了产品的一些信息和协议：

![](https://bbs.kanxue.com/upload/attach/201901/3_YZNUYKT24NZD4MC.jpg)

主要是其他四个文件夹：

`utils`文件夹：主要是Revision的细微差别

![](https://bbs.kanxue.com/upload/attach/201901/3_EKCMKJR6M6WHX57.jpg)

`tools`文件夹：ollvm中有`clang`文件夹，也可以看出`clang`是作为工具在llvm中存在，而llvm中并没有此文件夹，在llvm官网中`clang`是作为单独源码而存在

![](https://bbs.kanxue.com/upload/attach/201901/3_TJGWJK2EK7RUV6U.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/3_HAU5JJ7DVFYPU5W.jpg)

`include`文件夹（整个文件夹内都是.h文件夹）：

Ollvm多了`llvm\Transforms\obfuscation`文件夹，可以看出ollvm添加了一些功能头文件，一共6个文件。其中有5个我们可以理解为具体参数功能，`fla` 参数表示使用控制流平展（Control Flow Flattening）模式，`sub`参数表示使用指令替换（Instructions Substitution）模式，`bcf`参数表示使用控制流伪造（Bogus Control Flow）模式，`aesSeed`参数表示aes加密随机种子，`split`参数表示分离代码块，我们常用的是`sub`、`bcf`、`fla`。混淆参数代码我们在接下来分析，先看`Utils.h`文件

`Utils.h`（功能箱）：

![](https://bbs.kanxue.com/upload/attach/201901/3_7KZ6QSFU5BC39BZ.jpg)

其实这里也可以看出来，ollvm基于了llvm，使用了llvm的头文件，这些头文件也在该`include`文件夹。

`lib`文件夹（`\lib\Transforms`）：

![](https://bbs.kanxue.com/upload/attach/201901/3_K86TKY6KVZ3STRF.jpg)

这里稍微麻烦点，我们还是从最外层的两个文件分析，主要添加了`obfuscation`目录的编译和构建：

`CMakeLists.txt`：

![](https://bbs.kanxue.com/upload/attach/201901/3_E5UJE8XPJVS7EBU.jpg)

`LLVMBuild.txt`：

![](https://bbs.kanxue.com/upload/attach/201901/3_WTJU7ZXFMJJKH4E.jpg)

再来看`IPO`文件夹的两个文件：

`LLVMBuild.txt`：

![](https://bbs.kanxue.com/upload/attach/201901/3_D7JEMCKDYQRBFPE.jpg)

`PassManagerBuilder.cpp`（从文件名可以看出，`pass`管理生成，ollvm就是写`pass`），我们分三部分解析：

导入ollvm特有的头文件

![](https://bbs.kanxue.com/upload/attach/201901/3_C37QJD2PBGPQ888.jpg)

添加混淆参数`flag`：

![](https://bbs.kanxue.com/upload/attach/201901/3_W26XVCUY9YAGF75.jpg)

首先进行全局`aes`（`aesSeed`）随机种子密码初始化，接着把ollvm混淆功能函数（`split`、`fla`、`bcf`、`sub`）添加进去，而添加的这些函数，都可以在上面`include`文件夹的头文件找到，`BogusControlFlow.h`、`Flattening.h`、`Substitution.h`、`CryptoUtils.h`等。

![](https://bbs.kanxue.com/upload/attach/201901/3_XQY4HGA2BZ7RTZ7.jpg)

以`BogusControlFlow`为例来看：

![](https://bbs.kanxue.com/upload/attach/201901/3_PY3A6GPTTV3DM42.jpg)

再以`CryptoUtils`为例来看：

![](https://bbs.kanxue.com/upload/attach/201901/3_P3JWVT3RET242HJ.jpg)

最后以`Split`为例来看：

`Split.h`（分离基础块，这里可能对于`BasicBlock`基础块不太明白，就简单理解为代码块，也可以推测LLVM基于的是模块）：

![](https://bbs.kanxue.com/upload/attach/201901/3_BMKR3MVHTTQX8NW.jpg)

如果存在一些疑惑，请仔细对比一看，是不是全部对上了？

最后一个文件夹，`obfuscation`文件夹，里面就是具体的CPP代码了，是不是要加点，然后，，，

![](https://bbs.kanxue.com/upload/attach/201901/3_NRZK2RRT2737P8K.jpg)

当然我们现在还是没有接触到具体的代码，但对于以上的逻辑分析清楚特别重要，为以后我们自己的`pass`添加铺路。下面以`BogusControlFlow.cpp`为例，来看看`pass`的编写。

---

我们使用sourceinsight4可以非常清晰的看到该代码的逻辑组成

先回到`BogusControlFlow.h`，看到了`pass.h`

![](https://bbs.kanxue.com/upload/attach/201901/3_YHTP7XGQQT4VTYF.jpg)

接着来到`BogusControlFlow.cpp`的llvm `pass`代码处，而`createBogus`函数，我们在前面的`PassManagerBuilder.cpp`是见到过的。

![](https://bbs.kanxue.com/upload/attach/201901/3_PQBNYECH2A2NNJM.jpg)

往下跟，`BogusControlFlow`是一个结构体，跟到`runOnFunction`，

![](https://bbs.kanxue.com/upload/attach/201901/3_XPTSD5KV5HPGRPH.jpg)

`runOnFunction`调用了`bogus`函数

![](https://bbs.kanxue.com/upload/attach/201901/3_Z8ENJJRUN8PN2M7.jpg)

`bogus`调用了`addBogusFlow`函数

![](https://bbs.kanxue.com/upload/attach/201901/3_A9A6P8VTS6DCXBU.jpg)

`addBogusFlow`函数意为添加假流程，看来这里就是比较核心的函数实现了

![](https://bbs.kanxue.com/upload/attach/201901/3_KKYKDRJ4RUP22CR.jpg)

其实英文写的都是非常清楚的，现在翻译挺智能的，意思为：

![](https://bbs.kanxue.com/upload/attach/201901/3_JZ9Y3A4EJXKT5KH.jpg)

这里意思大概应该结合着前面学习都理解了，除了`phi`节点，对数据结构有点了解的朋友都知道节点，节点就有前驱和后续节点，直接看代码理解，

![](https://bbs.kanxue.com/upload/attach/201901/3_9AUGM9UDQKHS7C7.jpg)

但在真实代码中，`phi`并不存在，因此我们要继续消除`phi`，这也是转为真实指令的重要一步，对于以上代码，我们可以如下消除：

![](https://bbs.kanxue.com/upload/attach/201901/3_9MUQK8QYBYZVSP6.jpg)

`bb`就是我们刚刚见到的`BasicBlock`，这样我们就消除了`phi`节点，同时更进一步理解了`addBogusFlow`函数的意思，LLVM中使用`reg2mem` `pass`来对`phi`进行消除。至此，一切又回到了我们最初对LLVM的介绍。

七少月 2018.04

图片处理真麻烦，直接上网盘：链接:e50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0K6q4&6z5f1E0V1j5i4R3&6N6#2)9#2k6U0g2E0y4V1y4W2K9g2R3%4j5#2A6%4i4@1f1#2i4@1q4r3i4K6R3$3i4@1f1%4i4@1p5H3i4K6R3I4i4K6y4m8 kh96

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-1-24 15:28 被admin编辑，原因：

上传的附件：

- [ollvm快速学习.doc](javascript:void(0);)（627.50kb，395次下载）

