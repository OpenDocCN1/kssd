
| `use_counters` | 1 | 使用[覆盖率计数器](https://bbs.kanxue.com/elink@880K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4D9j5h3&6Y4i4K6u0W2L8r3I4$3L8g2)9J5k6h3!0J5k6#2)9J5c8X3c8G2j5%4y4Q4x3V1k6e0j5h3&6A6N6r3W2*7k6i4u0o6L8%4k6W2M7X3q4Y4k6g2)9J5k6h3S2@1L8h3I4Q4x3U0y4U0L8%4k6W2M7X3q4Y4k6g2)9J5k6r3y4G2N6h3&6@1k6i4u0K6)生成命中代码块的频率的近似计数；默认为1。 |

| `help` | 0 | 打印帮助并退出 |

| `merge` | 0 | 不损失覆盖率前提下，将第2/3/4/…个语料库合并到第一个中去 |

| `merge_control_file` | 0 | 指定合并进程的控制文件，用于恢复合并状态 |

| `jobs` | 0 | 运行的作业数量。所有作业的输出会被重定向到`fuzz-JOB.log`。 |

| `workers` | 0 | 运行作业的并发进程数。若为0，实验CPU核心数一半 |

| `reload` | 1 | 每N秒载主语料库，以知悉其他进程发现的单元 |

| `only_ascii` | 0 | 仅生成ASCII（isprint + isspace）输入 |

| `artifact_prefix` | 0 | 提供将模糊处理工件（崩溃，超时或缓慢的输入）另存为`$(artifact_prefix)file`时要使用的前缀。默认为空。 |

| `exact_artifact_path` | 0 | 如果为空则忽略（默认）。如果为非空，则将失败（崩溃，超时）时的单个工件写为`$(exact_artifact_path)`。这将覆盖 `-artifact_prefix，`并且不会在文件名中使用校验和。请勿将相同的路径用于多个并行进程。 |

| `detect_leaks` | 1 | 如果为1（默认值）并且启用了LeakSanitizer，则尝试在模糊测试期间检测内存泄漏（即，不仅在关闭时）。 |

| `print_final_stats` | 0 | 退出时打印统计信息 |

| `print_corpus_stats` | 0 | 退出时打印语料库元素统计信息 |

| `print_coverage` | 0 | 退出时打印覆盖率信息 |

| `close_fd_mask` | 0 | 为1则在关闭stdout，为2则关闭stderr，为3则关闭二者 |

## 重运行模式

|  |  |

| --- | --- |

| 1 | `./your-fuzzer -flag1=val1 -flag2=val2 ... file1 file2 ...` |

与上面一样，但是选项后面接的是文件列表而非文件夹列表，这些输入样例将会重新读取并输入运行，不会产生新样例，在回归测试时十分有用。

这里有几个选项功能是值得单独说一下的

### Seed corpus 种子语料库

`corpus`语料库就是给目标程序的各种各样的输入

|  |  |

| --- | --- |

| 1  2 | `mkdir MY_CORPUS`  `./your-fuzzer MY_CORPUS/` `seeds/` |

当基于`libFuzzer`的模糊器以另一个目录作为参数执行时，它将首先递归地从每个目录中读取文件（在本例中`MY_CORPUS/`和`seeds/`都读），并对所有目录执行目标函数。然后，任何触发感兴趣的代码路径的输入将被写回到第一个语料库目录（在本例中为`MY_CORPUS`）。一般情况下我们将相关文件放在`seeds`的位置下，`MY_CORPUS/`目录为空，这样运行后生成的样本就存在`MY_CORPUS/`中了。

### 精简语料库样本集

在模糊测试期间，测试语料库可能会增长到很大容量。如果希望最小化语料库，即创建具有相同覆盖率的语料库子集但容量却小很多，这就是一件性价比十分高的事情了。

|  |  |

| --- | --- |

| 1  2 | `mkdir corpus1_min`  `./your-fuzzer -merge=1 corpus1_min corpus1` |

- `corpus1_min`: 精简后的样本集存放的位置
- `corpus1`: 原始样本集存放的位置

### 并行运行

提高模糊测试效率的另一种方法是使用更多的CPU。如果您使用`-jobs=N`它运行模糊器，它将产生N个独立的作业，但最多不超过拥有的内核数的一半。用于`-workers=M`设置允许的并行作业数。

当指定了多个任务时，程序启动时会先产生一个`master`进程，同时并发启动相应数量个`worker`进程，`master`会把`jobs`分配到`workers`上去执行，当某个任务结束后，相应进程终止，同时`master`会启动一个新的任务进程分配到对应的`worker`上，平均每个`worker`上会分配`*jobs/workers*`个任务。

|  |  |

| --- | --- |

| 1 | `./your-fuzzer MY_CORPUS/` `seeds/` `-jobs=8` |

在8核计算机上，这将产生4个并行工作器。如果其中一个被退出，将自动创建另一个，最多8个。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `fuzzer -jobs=8`  `├─sh -c ./fuzzer >fuzz-0.log 2>&1`  `│   └─fuzzer`  `│       └─{fuzzer}`  `├─sh -c ./fuzzer >fuzz-1.log 2>&1`  `│   └─fuzzer`  `│       └─{fuzzer}`  `├─sh -c ./fuzzer >fuzz-2.log 2>&1`  `│   └─fuzzer`  `│       └─{fuzzer}`  `├─sh -c ./fuzzer >fuzz-3.log 2>&1`  `│   └─fuzzer`  `│       └─{fuzzer}` |

### Dictionaries 字典

字典最早是`afl`在2015年一篇博客上提出的[`afl-fuzz: making up grammar with a dictionary in hand`](https://bbs.kanxue.com/elink@7b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5$3q4E0N6s2g2X3i4K6u0W2j5X3I4G2k6%4y4H3L8%4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6q4Q4x3V1k6S2k6X3I4Q4x3X3c8X3N6i4A6*7i4K6u0V1L8h3q4C8K9h3&6Y4i4K6u0V1N6i4m8Q4x3X3c8Y4M7X3q4E0L8h3q4J5i4K6u0V1N6$3W2@1K9q4)9J5k6h3S2@1L8h3H3`.)，

基本思路就是应用程序都是都是**处理具有一定格式的数据**，比如 `xml` 文档， `png`图片等等。 这些数据中会有一些**特殊字符序列 （或者说关键字）**， 比如 在 `xml` 文档中 就有 `CDATA`, 等，**png图片** 就有 **png 图片头**。

如果我们事先就把这些 **字符序列** 列举出来， `fuzz` 直接使用这些关键字去 组合，就会就可以减少很多没有意义的 尝试，同时还有可能会走到更深的程序分支中去。

`Dictionary` 就是实现了这种思路。 `libfuzzer` 和 `afl` 使用的 `dictionary` 文件的语法是一样的， 所以可以直接拿 `afl` 里面的 `dictionary` 文件来给 `libfuzzer` 使用。

如下是`libFuzzer`官方文档中的字典示例

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `# Lines starting with '#' and empty lines are ignored.`    `# Adds "blah" (w/o quotes) to the dictionary.`  `kw1="blah"`  `# Use \\ for backslash and \" for quotes.`  `kw2="\"ac\\dc\""`  `# Use \xAB for hex values`  `kw3="\xF7\xF8"`  `# the name of the keyword followed by '=' may be omitted:`  `"foo\x0Abar"` |

- `#` 开头的行 和 空行会被忽略
- `kw1=` 这些就类似于注释， 没有意义
- 真正有用的是由 `"` 包裹的**字串**，这些 **字串** 就会作为一个个的关键字， `libfuzzer` 会用它们进行组合来生成样本。

`libfuzzer` 使用 `-dict` 指定 `dict` 文件，下面使用 `xml.dict` 为 `dictionary` 文件，进行 `fuzz`。

|  |  |

| --- | --- |

| 1 | `./your-fuzzer -dict=DICTIONARY_FILE` |

### 输出

当`fuzzer`成功运行之后，信息会输出在屏幕上。输出行具有事件代码和统计信息的形式。常见的事件代码是：

- `READ` `fuzzer`已从语料库目录中读取了所有提供的输入样本。
- `INITED` `fuzzer`已完成初始化，其中包括通过被测代码运行每个初始输入样本。
- `NEW` `fuzzer`创建了一个测试输入，该输入涵盖了被测代码的新区域。此输入将保存到主要语料库目录。
- `pulse` `fuzzer`已生成 2的n次方个输入（定期生成以使用户确信`fuzzer`仍在工作）。
- `DONE` `fuzzer`已完成操作，因为它已达到指定的迭代限制（`-runs`）或时间限制（`-max_total_time`）。
- `RELOAD` `fuzzer`在定期从语料库目录中重新加载输入；这使它能够发现其他`fuzzer`进程发现的任何输入（请参阅[并行模糊化](https://bbs.kanxue.com/elink@799K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5k6h3I4D9N6X3#2Q4x3X3g2G2M7X3N6Q4x3V1j5@1i4K6u0W2x3q4)9J5k6e0m8Q4x3V1k6V1L8$3y4K6i4K6u0r3e0r3W2T1c8Y4g2*7P5X3g2J5i4K6u0W2K9s2c8E0L8q4)9J5x3%4m8S2M7X3q4D9L8r3g2D9i4K6u0V1k6Y4g2*7P5X3W2F1k6H3`.`.)）。

每条输出行还报告以下统计信息（非零时）：

- `cov：` 执行当前语料库所覆盖的代码块或边的总数。
- `ft：` `libFuzzer`使用不同的信号来评估代码覆盖率：边缘覆盖率，边缘计数器，值配置文件，间接调用方/被调用方对等。这些组合的信号称为*功能*（`ft：`）。
- `corp：` 当前内存中测试语料库中的条目数及其大小（以字节为单位）。
- `exec/s：` 每秒模糊器迭代的次数。
- `rss：` 当前的内存消耗。

对于`NEW`事件，输出行还包含有关产生新输入的变异操作的信息：

- `L：` 新输入的大小（以字节为单位）。
- `MS: <n> <operations>` 用于生成输入的变异操作的计数和列表。

我们以如下样例作解释：

|  |  |

| --- | --- |
