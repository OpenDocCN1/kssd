
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

`//AFL/llvm_mode/afl-llvm-rt.o.c`
```
void __sanitizer_cov_trace_pc_guard(uint32_t* guard) {
  __afl_area_ptr[*guard]++;
}
```
/* Init callback. Populates instrumentation IDs. Note that we're using
ID of 0 as a special value to indicate non-instrumented bits. That may
still touch the bitmap, but in a fairly harmless way. */
```
void __sanitizer_cov_trace_pc_guard_init(uint32_t* start, uint32_t* stop) {
  u32 inst_ratio = 100;
  u8* x;
  if (start == stop || *start) return;
  x = getenv("AFL_INST_RATIO");
  if (x) inst_ratio = atoi(x);
  if (!inst_ratio || inst_ratio > 100) {
    fprintf(stderr,"[-] ERROR: Invalid AFL_INST_RATIO (must be 1-100).\n");
    abort();
  }
  /* Make sure that the first element in the range is always set - we use that
  to avoid duplicate calls (which can happen as an artifact of the underlying
  implementation in LLVM). */
  *(start++) = R(MAP_SIZE - 1) + 1; //初始化初始值
  while (start < stop) {
    //为每一个*guard赋初始值*start = R(MAP_SIZE - 1) + 1;
    if (R(100) < inst_ratio) *start = R(MAP_SIZE - 1) + 1;
    //  #  define R(x) (random() % (x))【random随机数】
    else *start = 0;
    start++;
    //指针遍历（uint32_t）
  }
}
```

> 但是不可避免的，会有概率出现冲突的情况，因此Sakura师傅对其进行了替换：

```
void __sanitizer_cov_trace_pc_guard_init(uint32_t* start, uint32_t* stop) {
  unsigned int N = 0 ;
  printf("@sakura in __sanitizer_cov_trace_pc_guard_init\n");
  if(start == stop || *start) return;
  for (uint32* x = start ; x < stop ; x++){
    *x = (++N) % MAP_SIZE;
    //mod map_size（1 << 16 == 65536）
  }
  printf("@sakura there are %u guards in total\n",N);
}
```

要想对每条edge进行插桩，需要先在`make afl`时传入`AFL_TRACE_PC=1`来定义`DUSE_TRACE_PC`宏（`make AFL_TRACE_PC=1`），然后在执行`afl-clang-fast`的时候传入`-fsanitize-coverage=trace-pc-guard`参数即可。正常`make`会启用`afl-llvm-pass`而非`afl-llvm-rt`

### GCOV、LCOV

覆盖率记录的实现除了前两种方式之外还可以使用GCOV、LCOV这两个东西可视化展示代码覆盖率，但是不能使用到fuzzer上。

**一些fuzz的覆盖率反馈和引导变异的一些概念：**

fuzz有`生成式fuzz`和`变异式fuzz`两种，生成式fuzz就是一开始什么种子也没有，纯根据fuzzer的逻辑来生成种子（输入），fuzz过程中没有任何的反馈，也不会根据某时某刻的代码覆盖率来把interesting case变异发现新的路径。 AFL采用的是变异式fuzz

进行fuzz时，对样本进行变异，然后在找到新的路径时，将当前变异后数据当成一个新的样本继续变异。经过逐代的变异杂交选优，直到达到一个局部最优解为止（已发现的路径达到max），这就是所谓的**遗传算法**。

因为遗传算法的特征总是来自于**初始种子样本**和**变异策略**，所以改进也主要在这两方面。种子也可以是一些代码语句：

```
select * from cyberangel;
```

如果变异器不会引入新的token（特征），那永远就只会生成`select`这一种特征，而不会生成如SQL语句中的`insert`、`delete`等。所以种子的变异策略也十分的重要，它极大的影响着fuzz的代码覆盖率。

除了对种子的不断变异，我们还需要了解AFL如何将自动或半自动生成的随机数据输入到被fuzzer的程序中并监视如崩溃，断言（assertion）等程序异常这个过程，也就是进行输入和捕获crash。AFL采用的是`fork server`这种机制，具体方式如下：

1. fuzzer变异生成新样本后会将它们写入到执行文件目录的`.cur_input`隐藏文件夹中
2. `afl-fuzz`会`fork`出一个子进程作为`fork server`，然后fuzzer通过管道发送4字节数据通知`fork server`去`fork`一个进程来测试。
3. 无论被fuzz的进程结果如何，`fork server`都会通过管道返回子进程的执行结果到`afl-fuzz`
4. `afl-fuzz`根据测试生成的覆盖率信息来引导后续的测试

> 进程关系：`fuzzer进程 -> fork server -> 被fuzz的程序`

常用的AFL魔改技巧有哪些？

- 因为AFL中有覆盖率反馈和crash捕获的功能，所以可以替换AFL原本的`mutate`【变异器】为自己的代码，说白了就是借了AFL的一个壳。
- 在写入`testcase`之前可以对写入的内容进行封装和映射，`afl-fuzz`的这部分原本的代码如下：

```
//afl-fuzz.c
/* Write modified data to file for testing. If out_file is set, the old file
is unlinked and a new one is created. Otherwise, out_fd is rewound and
truncated. */
static void write_to_testcase(void* mem, u32 len) {
  s32 fd = out_fd;
  if (out_file) {
    unlink(out_file); /* Ignore errors. */
    fd = open(out_file, O_WRONLY | O_CREAT | O_EXCL, 0600);
    if (fd < 0) PFATAL("Unable to create '%s'", out_file);
  } else lseek(fd, 0, SEEK_SET);
  ck_write(fd, mem, len, out_file);
  if (!out_file) {
    if (ftruncate(fd, len)) PFATAL("ftruncate() failed");
    lseek(fd, 0, SEEK_SET);
  } else close(fd);
}
```

`write_to_testcase`函数会将变异生成的新样本写入到`.cur_input`隐藏文件夹中，所以我们在文件写入之前可以在外面套上一层函数以便让AFL的原始字节码的变异适用于更多的场景。`afl-fuzz`适合基于字节粒度变异的fuzz（如fuzz一个简单的输入输出程序），但并不是所有目标都可以直接进行字节粒度的fuzz。有些是因为文件解析部分的代码不能单独抽出来，有些是因为文件解析仅仅是逻辑的开始。那么为变异器加层就是在这方面扩展`afl-fuzz`的最简单方法，最经典的例子如`webassembly`。

为了fuzz这些结构化的东西（类似SQL）我们需要进行`结构感知（structure-aware）`，即针对特定输入类型的语法进行感知

**如何使用AFL fuzz client-server模式的程序，或者如何使用AFL去fuzz网络协议呢？**

`client-server`模式中的`client -> server`大致流程如下：

```
client ---> server(){
接收包();
处理包();
返回响应包();
}
```

这篇文章值得参考：[475K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3j5i4y4@1L8s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9r3!0%4i4K6u0V1k6Y4g2*7P5W2)9J5k6s2y4W2M7Y4k6W2M7W2)9J5k6r3q4E0k6i4u0A6j5$3q4F1i4K6u0V1k6Y4g2*7P5Y4W2Q4x3X3c8D9L8%4l9`.](https://bbs.kanxue.com/elink@6cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3j5i4y4@1L8s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9r3!0%4i4K6u0V1k6Y4g2*7P5W2)9J5k6s2y4W2M7Y4k6W2M7W2)9J5k6r3q4E0k6i4u0A6j5$3q4F1i4K6u0V1k6Y4g2*7P5Y4W2Q4x3X3c8D9L8%4l9`.`)

文章中提到一个为`Persistent mode（持久模式）`的概念，如果要向AFL集成这种功能，我们可以按照如下流程编写自己的代码：

```
while (go) //while loop
put_request(read(file)) //将client要发送的请求包写入到文件中
req = get_request() //并让服务端从文件中获取请求
process(req) //服务端处理请求
notify_fuzzer() //通知fuzzer（AFL）
```

该方法需要用`afl`直接启动`server`程序，`patch` `server`程序接收请求包的代码，改为直接从标准输入里读取，`server`执行足够多次请求后退出，AFL再启动一个新的`server`再次`fuzz`。

**fuzz要真正解决的问题和一些Sakura师傅的建议**

- 要对某一个项目进行fuzz就要首先考虑如何将项目跑起来，如何在插桩和`asan`之后跑起来。所以要先学会`Makefile`和`CMake`，必要时要学会使用`unicorn`、`qiling`等固件仿真工具进行模拟运行。

> `Address Sanitizer`又名`ASan`，是一个快速的C/C++内存错误检查器。它可以检测到：`UAF`、`Heap buffer overflow`、`Stack buffer overflow`、`Global buffer overflow`、`Use after return`、`Use after scpoe`、`Initialization order bugs`、`Memory leak`等漏洞类型。`asan`可以简单理解成对`malloc`和`free`以及存取指令等的`hook`，从而在发现分配出来的内存的大小，小于要存取的`index`的大小时，检测出越界读写问题。

- 要理解被测试程序的代码，并不是说随便拿到一个程序拿fuzz跑起来是有意义的，例如传入数据的`parser`（解析器、语法分析器），必要时通过逆向来单独取出一部分代码来测试功能。比如Windows的`media player`，这是一个图形化的应用程序，我们不能通过命令行窗口来对该程序进行流程的控制与输入，但是这些功能的实现肯定都是存在于某一个`dll`（动态链接库）中的，所以可以利用`dlopen`函数写`harness`来加载`dll`：

> `harness`作用：如果我们想fuzz `dll` (加载的库)中的函数输入，因为没有定义入口点，我们需要编写测试工具来将输入从命令行传递到 DLL 函数。

- 善于根据场景改进fuzz，以将只用与文件格式fuzz的AFL利用加层或映射让其扩展到更多的场景中，实现自己的自定义编译。
- 根据不同的目标掌握不同的知识，找到主流使用的fuzzer并进行改进
  - `js fuzz`需要掌握编译原理，理解文法
  - 内核`fuzz`需要掌握内核知识和驱动盒编写，理解如何构建和生成有关联性的系统调用集合。
- 自定义检测工具

> `Address Sanitizer`又名`ASan`，是一个快速的C/C++内存错误检查器。它可以检测到：`UAF`、`Heap buffer overflow`、`Stack buffer overflow`、`Global buffer overflow`、`Use after return`、`Use after scpoe`、`Initialization order bugs`、`Memory leak`等漏洞类型。`asan`可以简单理解成对`malloc`、`free`等指令的`hook`，但是如果像`js`引擎这种直接`mmap`一块大内存然后自己来管理内存的情况，那么如何检测出潜在的越界读写问题呢？这种时候就需要理解代码，然后自己实现一套针对性的自定义`asan`了。

## afl-gcc源码阅读
