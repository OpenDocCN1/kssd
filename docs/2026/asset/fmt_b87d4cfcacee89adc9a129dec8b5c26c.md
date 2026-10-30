
```bash
afl-clang -O0 int\_check.c -o afl\_int\_check
../symcc\_build\_qsym/symcc -O0 int\_check.c -o symcc\_int\_check
mkdir corpus
echo "AAAAAAAAAAAAAAAAAAAAAAAA" > corpus/seed
```

![image-20210316201931648](upload/attach/202103/832012\_5FDJC5ZDPNT6PN4.jpg)

之后就开始运行吧，我们使用`afl`的并行模式：

```bash
afl-fuzz -M fuzz1 -i corpus/ -o out -m none -- ./afl\_int\_check @@
afl-fuzz -S fuzz2 -i corpus/ -o out -m none -- ./afl\_int\_check @@
~/.cargo/bin/symcc\_fuzzing\_helper -o out -a fuzz2 -n symcc -- ./symcc\_int\_check @@
```

这里`symcc_fuzzing_helper`的参数中`-o`指向`afl`的`out`目录，`-a`指向要辅助的`afl`进程名字，如上所示这里我就是`fuzz2`。

`symcc_fuzzing_helper`其实就是在使用`afl`生产的testcase进行符号执行，如果它认为样例有趣就会生成新的，并将它传送到正在执行的`afl`队列里，`afl`就能够使用该新生成样例进行测试，这一流程辅助了`afl`能够到达一些之前无法到达的路径。

这里有个小坑，单纯执行`afl`时`afl-fuzz`命令后面的`--`可带也可不带，但是`symcc_fuzzing_helper`是通过检测`afl-fuzz`里`--`后面命令执行的，所以如果我们在`afl-fuzz`后习惯性不加`--`就可能导致`symcc`无有效的输出结果。

一切准备就绪，开始运行吧！

![ddf22164-1a3a-45dc-b137-c7cebffdaab6](upload/attach/202103/832012\_5WEKNUQ8DWABQVH.jpg)

我们惊喜的发现单纯fuzz难以找到的漏洞，在`symcc_fuzzing_helper`的协助下，直接就找到了crash。

![81de9aff-f7eb-449c-984c-72881a98fac1](upload/attach/202103/832012\_Y7EPZ2VXJTYC3CU.jpg)

简单分析一下，用`Hexdump`看到crash真的是`0xdeadbeef`，`symcc`辅助`AFL`找到了使目标程序崩溃的testcase。

## 总结

使用符号执行`Symcc`与模糊测试引擎`Afl`的简单验证成功了，之后可以就试一试现实中的一些项目，比如论文中提的`Tcpdump`之类的；还可以考虑将`Symcc`与其他模糊测试引擎相结合，比如`Afl++`，在各种算法加持下的`Afl++`再结合符号执行可能会有更好的效果。

最后特别感谢`Symcc`发明人和论文的原作者 Aurélien Francillon 与 ADALogics 的安全研究员David Korczynski，它们在推特上帮助了我很多，同时感谢Discord的Fuzzing板块讨论让我有兴趣进行各种新的尝试，我仍有很多需要学习的地方。

![image-20210319120238731](upload/attach/202103/832012\_EA3KCME24TS72PW.jpg)

## 参考资料

带你搞懂符号执行的前世今生与最近技术

<mark class="encrypted">20eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6x3e0b7I4x3H3`.`.</mark>

G.O.S.S.I.P 学术论文推荐

<mark class="encrypted">676K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3#2H3i4K6u0W2j5i4m8H3i4K6u0r3M7r3!0K6N6s2y4Q4x3V1j5@1x3r3p5I4y4U0t1J5z5q4)9J5k6o6N6S2z5o6k6Q4x3X3b7@1z5e0R3#2i4K6u0V1j5U0N6U0x3W2)9J5k6r3t1K6y4e0l9%4k6e0y4X3j5K6p5$3x3b7`.`.</mark>

Symcc源码

<mark class="encrypted">297K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6i4u0W2j5$3!0E0i4K6u0V1M7K6y4Q4x3V1k6K6P5h3#2U0j5#2)9J5c8R3`.`.</mark>

入门`afl`

<mark class="encrypted">7ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0V1L8g2)9J5k6h3c8W2N6W2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3x3U0l9I4z5e0p5H3x3o6q4Q4x3X3b7J5x3U0f1%4y4o6k6Q4x3X3g2Z5N6r3#2D9</mark>

上个月末无聊的划水时间段内，在推上看到有人发了一篇关于如何结合去年新发布的符号执行`Symcc`与模糊测试引擎`AFL`，以提升Fuzz效率的视频贴。打开这个链接后才发现是个卖课的，emmm.... , 看价格**£1499**果断打扰了，果然没钱的都不配学安全吗23333；原技术文章看来是看不到了，只能靠公开的一段视频还原操作。

## 背景介绍

虽然有很多大佬肯定已经十分熟悉模糊测试和符号执行了，还是简单介绍一下背景知识，这样更有连贯性。

### 模糊测试 `AFL`

模糊测试（fuzz testing, fuzzing）是一种软件测试技术。 其核心思想是將自动或半自动生成的随机数据输入到一个程序中，并监视程序异常，如崩溃，断言（assertion）失败，以发现可能的程序错误，比如内存泄漏。 模糊测试常常用于检测软件或计算机系统的安全漏洞。

模糊测试诞生于1988年秋季的一个黑暗暴风雨之夜 [[Takanen et al, 2008.](https://bbs.kanxue.com/elink@37aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2K6i4K6u0W2j5i4u0@1k6h3y4Z5K9r3!0#2M7$3g2Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6*7K9h3&6Y4i4K6u0V1k6X3!0J5i4K6u0V1f1$3!0X3N6s2N6S2M7X3g2Q4x3X3c8e0k6h3y4#2M7X3W2@1P5g2)9J5k6q4c8W2M7%4c8A6L8X3N6Q4x3X3c8S2L8X3c8Q4x3X3c8c8N6h3q4D9K9i4c8&6i4K6u0V1b7i4y4K6N6i4u0S2L8X3y4W2i4K6u0V1f1$3g2U0L8$3&6V1i4K6u0V1c8h3c8A6N6r3W2G2L8W2)9J5k6q4l9I4z5e0x3H3i4K6u0W2j5i4y4H3P5l9`.`.)]。巴顿·米勒教授坐在麦迪逊威斯康星州的公寓里，通过一条1200波特的电话线连接到他所属大学的计算机。阵阵的雷暴在线路上造成噪音，这些噪音又导致两端的UNIX命令获得错误的输入，并导致崩溃。频繁的崩溃使他感到惊讶—我们编写的程序不是应该十分强大吗？作为一名科学家，他想探究该问题的严重程度及其原因。因此，他为威斯康星大学麦迪逊分校的学生编写了一个编程练习，而该练习将使他的学生创建第一个模糊测试器。

这项作业的原文描述是这样的：

> The goal of this project is to evaluate the robustness of various UNIX utility programs, given an unpredictable input stream. [...] First, you will build a fuzz generator. This is a program that will output a random character stream. Second, you will take the fuzz generator and use it to attack as many UNIX utilities as possible, with the goal of trying to break them.
>
> 该项目的目标是在给定不可预测的输入流的情况下评估各种UNIX实用程序的健壮性。[...]首先，您将构建一个模糊发生器。这是一个将输出随机字符流的程序。其次，您将使用模糊发生器，并使用它来攻击尽可能多的UNIX实用程序，以试图破坏它们。

这个作业在不经意间抓住了模糊测试的本质：创建随机的输入，并持续性观察它是否会破坏目标应用程序，理论上只要运行足够长的时间，我们就会看到错误的发生。

`AFL`（american fuzzy lop）最初由Michał Zalewski开发，和`libFuzzer`等一样是基于覆盖引导（Coverage-guided）的模糊测试工具，它通过记录输入样本的代码覆盖率，从而调整输入样本以提高覆盖率，增加发现漏洞的概率。其工作流程大致如下：

1. 从源码编译程序时进行插桩，以记录代码覆盖率（Code Coverage）
2. 选择一些输入文件，作为初始测试集加入输入队列（queue）
3. 将队列中的文件按一定的策略进行“突变”
4. 如果经过变异文件更新了覆盖范围，则将其保留添加到队列中
5. 上述过程会一直循环进行，期间触发了crash的文件会被记录下来

![image-20210319132122277](https://bbs.kanxue.com/upload/attach/202103/832012_ZNDDRHP7CFBXAPT.jpg)

### 符号执行 `Symcc`

符号执行简单来说是在目标程序的执行过程中跟踪中间值是如何计算的，每一个中间值都可以表示为程序输入的一个公式。在任何点，系统都会使用这个公式查看这个点是否可达，这个指针是否为空等。如果答案是确定的，那么符号执行引擎将会提供测试用例，一个新的输入例子来触发对应的行为。所以符号执行可以被方便的用来探测程序路径以及触发bug。

新的符号执行`Symcc`发表于去年顶会USENIX’20，论文名称为Symbolic execution with SYMCC: Don’t interpret, compile!

那么`Symcc`和它的前辈们又有什么不同呢？论文作者Aurélien Francillon认为传统符号执行主要分为两类：`IR-based`和`IR-less`。

`IR-based`是指无论测试对象如何，先把目标的二进制程序给转换到IR层（中间表达形式）再进行抽象解释，其缺点是特别容易路径爆炸。常见用此方法的符号执行有`angr`、`KLEE`和`Mayhem`，主要过程如下图所示：

![image-20210319094028131](https://bbs.kanxue.com/upload/attach/202103/832012_AA2F7YY6BJKRZB3.jpg)

`IR-less`是指符号执行引擎通过动态插桩技术（利用`PIN`插桩框架等）先对二进制程序插桩，执行部分指令后再构造符号表达式。其优点是快，缺点是插入的代码函数可能无法生成正确的符号表达式，且较依赖于指令集。常见用此方法的符号执行有`Triton`、`QSYM`、`SAGE`和`Driller`，主要过程如下图所示：

![image-20210319094624987](https://bbs.kanxue.com/upload/attach/202103/832012_CY3KN6ATJYGX8UX.jpg)

作者提出的`SymCC`不同点在于，直接在编译期就开始在生成的IR上植入符号执行相关代码，进一步提升性能；其流程大致如下：

![image-20210319094919700](https://bbs.kanxue.com/upload/attach/202103/832012_PB89A8HX8YBZ883.jpg)

## 开始结合

整体的结合流程没什么新花样，基本是按照 安装各种包环境—> 编译简单的后端/功能更强的`qsym`后端 —> 用`symcc`编译`llvm`的`libcxx` —> 开始尝试fuzz 的过程循序渐进。

### 安装各种环境

先安装各种包和依赖：

```bash
sudo apt-get install -y cargo \
  clang-10 \
  cmake \
  g++ \
  git \
  libz3-dev \
  llvm-10-dev \
  llvm-10-tools \
  ninja-build \
  python3-pip \
  zlib1g-dev
pip3 install lit
```

这里的`LLVM`要求是 8, 9, 10 或者11 ，C++编译器要支持 C++17。实际安装过程中有些unbuntu版本可能无法直接`apt-get`，使用`llvm`官方0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7s2c8Q4x3X3g2D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3 的包支持。

随后安装`Z3`，要求版本号大于4.5：

```bash
git clone https://github.com/Z3Prover/z3
cd z3
python scripts/mk_make.py
cd build
make
sudo make install
```

这里有点小坑，用该方式安装的`z3`可能在后面编译时`llvm`无法找到路径。如果报此类似错误的，该步可用`cmake`的`Ninja`进行编译，指路链接：bfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K9x3#2m8J5L8%4k6W2M7W2)9J5c8Y4Z5K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8W2u0q4b7f1c8y4c8g2)9J5k6p5y4y4j5h3E0W2i4K6u0W2L8h3b7`.

然后安装`afl`，没什么可说的先安原版：

```bash
git clone -b v2.56b https://github.com/google/AFL.git afl
cd afl && make
```

下载`symcc`的源码做好之后编译backend的准备：

```bash
git clone https://github.com/eurecom-s3/symcc symcc_source
cd symcc_source
git submodule init
git submodule update
```

### 编译backend

`symcc`带了两种后端，一种是功能简单（simple）的；另一种是`qsym`的后端，与前者相比功能性更强，我们两种都编译一下。在配置构建时，选项都将通过“`-D`”传递给`CMake`，常见选项如下表所示：

| 选项 | 作用 |
| --- | --- |
| `QSYM_BACKEND=ON/OFF` (default OFF) | 是否编译`QSYM`后端，若否即为编译`simple`后端；每次执行时可以使用`LD_LIBRARY_PATH`在后端之间切换 |
| `TARGET_32BIT=ON/OFF` (default OFF) | 启用64位主机对32位编译的支持 |
| `LLVM_DIR`/`LLVM_32BIT_DIR` (default empty) | 提供编译时`llvm`的位置 |
| `Z3_DIR`/`Z3_32BIT_DIR` (default empty) | 提供编译时`Z3`的位置 |
| `Z3_TRUST_SYSTEM_VERSION` (default OFF) | 信任系统版本的`Z3`，不检查`Z3`版本的兼容性；如果`Z3`的安装版本太旧，则可能发生编译错误。 |

首先编译简单的后端，这里的`LLVM_DIR`指向你所用版本`llvm`位置：

```bash
mkdir symcc_build_simple
cd symcc_build_simple
CC=clang-10 CXX=clang++-10 cmake -G Ninja \
  -DQSYM_BACKEND=OFF \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DZ3_TRUST_SYSTEM_VERSION=on \
  -DLLVM_DIR=/usr/lib/llvm-10/cmake \
  ../symcc_source \
  && ninja check
```
