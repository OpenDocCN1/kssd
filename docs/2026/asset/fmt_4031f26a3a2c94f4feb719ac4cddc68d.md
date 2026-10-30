
因为是整个AFL的核心区域，函数长度有点长，我们从以下几个部分分析

### 主要数据结构、变量和初始化

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `s32 opt;` `// getopt选项解析`  `u64 prev_queued = 0;` `// 上一轮队列中的测试用例数`  `u32 sync_interval_cnt = 0,` `// 同步间隔计数器`  `seek_to;` `// 队列中的起始位置`  `u8  *extras_dir = 0;` `// 额外字典目录`  `u8  mem_limit_given = 0;` `// 内存限制是否设置`  `u8  exit_1 = !!``getenv``(``"AFL_BENCH_JUST_ONE"``);` `// 基准测试模式`  `char``** use_argv;` `// 实际使用的命令行参数`    `struct` `timeval tv;` `// 时间结构`  `struct` `timezone tz;` `// 时区结构` |
|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `// 1. 显示版本信息`  `SAYF(cCYA` `"afl-fuzz "` `cBRI VERSION cRST` `" by <lcamtuf@google.com>\n"``);`    `// 2. 设置文档路径`  `doc_path = access(DOC_PATH, F_OK) ?` `"docs"` `: DOC_PATH;`    `// 3. 初始化随机数生成器`  `gettimeofday(&tv, &tz);`  `srandom(tv.tv_sec ^ tv.tv_usec ^ getpid());` |

跟`afl-as`使用相同的策略，确保整个AFL工具的随机性一致

### 命令行解析

|  |  |
| --- | --- |
| 1  2  3  4 | `while` `((opt = getopt(argc, argv,` `"+i:o:f:m:b:t:T:dnCB:S:M:x:QV"``)) > 0)`  `switch` `(opt) {`  `// 各种选项处理`  `}` |
|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121  122  123  124  125  126  127  128  129  130  131  132  133  134  135  136  137  138  139  140  141  142  143  144  145  146  147  148  149  150  151  152  153  154  155  156  157  158  159  160  161  162  163  164  165  166  167  168  169  170  171  172  173  174  175  176  177  178  179  180  181  182  183  184 | `case` `'i'``:` `/* input dir */`    `if` `(in_dir) FATAL(``"Multiple -i options not supported"``);`  `in_dir = optarg;`    `if` `(!``strcmp``(in_dir,` `"-"``)) in_place_resume = 1;`    `break``;`    `case` `'o'``:` `/* output dir */`    `if` `(out_dir) FATAL(``"Multiple -o options not supported"``);`  `out_dir = optarg;`  `break``;`    `case` `'M'``: {` `/* master sync ID */`    `u8* c;`    `if` `(sync_id) FATAL(``"Multiple -S or -M options not supported"``);`  `sync_id = ck_strdup(optarg);`    `if` `((c =` `strchr``(sync_id,` `':'``))) {`    `*c = 0;`    `if` `(``sscanf``(c + 1,` `"%u/%u"``, &master_id, &master_max) != 2 ||`  `!master_id || !master_max || master_id > master_max ||`  `master_max > 1000000) FATAL(``"Bogus master ID passed to -M"``);`    `}`    `force_deterministic = 1;`    `}`    `break``;`    `case` `'S'``:`    `if` `(sync_id) FATAL(``"Multiple -S or -M options not supported"``);`  `sync_id = ck_strdup(optarg);`  `break``;`    `case` `'f'``:` `/* target file */`    `if` `(out_file) FATAL(``"Multiple -f options not supported"``);`  `out_file = optarg;`  `break``;`    `case` `'x'``:` `/* dictionary */`    `if` `(extras_dir) FATAL(``"Multiple -x options not supported"``);`  `extras_dir = optarg;`  `break``;`    `case` `'t'``: {` `/* timeout */`    `u8 suffix = 0;`    `if` `(timeout_given) FATAL(``"Multiple -t options not supported"``);`    `if` `(``sscanf``(optarg,` `"%u%c"``, &exec_tmout, &suffix) < 1 ||`  `optarg[0] ==` `'-'``) FATAL(``"Bad syntax used for -t"``);`    `if` `(exec_tmout < 5) FATAL(``"Dangerously low value of -t"``);`    `if` `(suffix ==` `'+'``) timeout_given = 2;` `else` `timeout_given = 1;`    `break``;`    `}`    `case` `'m'``: {` `/* mem limit */`    `u8 suffix =` `'M'``;`    `if` `(mem_limit_given) FATAL(``"Multiple -m options not supported"``);`  `mem_limit_given = 1;`    `if` `(!``strcmp``(optarg,` `"none"``)) {`    `mem_limit = 0;`  `break``;`    `}`    `if` `(``sscanf``(optarg,` `"%llu%c"``, &mem_limit, &suffix) < 1 ||`  `optarg[0] ==` `'-'``) FATAL(``"Bad syntax used for -m"``);`    `switch` `(suffix) {`    `case` `'T'``: mem_limit *= 1024 * 1024;` `break``;`  `case` `'G'``: mem_limit *= 1024;` `break``;`  `case` `'k'``: mem_limit /= 1024;` `break``;`  `case` `'M'``:` `break``;`    `default``:  FATAL(``"Unsupported suffix or bad syntax for -m"``);`    `}`    `if` `(mem_limit < 5) FATAL(``"Dangerously low value of -m"``);`    `if` `(``sizeof``(rlim_t) == 4 && mem_limit > 2000)`  `FATAL(``"Value of -m out of range on 32-bit systems"``);`    `}`    `break``;`    `case` `'b'``: {` `/* bind CPU core */`    `if` `(cpu_to_bind_given) FATAL(``"Multiple -b options not supported"``);`  `cpu_to_bind_given = 1;`    `if` `(``sscanf``(optarg,` `"%u"``, &cpu_to_bind) < 1 ||`  `optarg[0] ==` `'-'``) FATAL(``"Bad syntax used for -b"``);`    `break``;`    `}`    `case` `'d'``:` `/* skip deterministic */`    `if` `(skip_deterministic) FATAL(``"Multiple -d options not supported"``);`  `skip_deterministic = 1;`  `use_splicing = 1;`  `break``;`    `case` `'B'``:` `/* load bitmap */`    `/* This is a secret undocumented option! It is useful if you find`  `an interesting test case during a normal fuzzing process, and want`  `to mutate it without rediscovering any of the test cases already`  `found during an earlier run.`    `To use this mode, you need to point -B to the fuzz_bitmap produced`  `by an earlier run for the exact same binary... and that's it.`    `I only used this once or twice to get variants of a particular`  `file, so I'm not making this an official setting. */`    `if` `(in_bitmap) FATAL(``"Multiple -B options not supported"``);`    `in_bitmap = optarg;`  `read_bitmap(in_bitmap);`  `break``;`    `case` `'C'``:` `/* crash mode */`    `if` `(crash_mode) FATAL(``"Multiple -C options not supported"``);`  `crash_mode = FAULT_CRASH;`  `break``;`    `case` `'n'``:` `/* dumb mode */`    `if` `(dumb_mode) FATAL(``"Multiple -n options not supported"``);`  `if` `(``getenv``(``"AFL_DUMB_FORKSRV"``)) dumb_mode = 2;` `else` `dumb_mode = 1;`    `break``;`    `case` `'T'``:` `/* banner */`    `if` `(use_banner) FATAL(``"Multiple -T options not supported"``);`  `use_banner = optarg;`  `break``;`    `case` `'Q'``:` `/* QEMU mode */`    `if` `(qemu_mode) FATAL(``"Multiple -Q options not supported"``);`  `qemu_mode = 1;`    `if` `(!mem_limit_given) mem_limit = MEM_LIMIT_QEMU;`    `break``;`    `case` `'V'``:` `/* Show version number */`    `/* Version number has been printed already, just quit. */`  `exit``(0);`    `default``:`    `usage(argv[0]);` |

支持的主要选项：

| 选项 | 参数 | 功能 | 重要性 |
| --- | --- | --- | --- |
| -i | 目录 | 输入测试用例目录 | 必需 |
| -o | 目录 | 输出结果目录 | 必需 |
| -f | 文件 | 目标程序输入文件 | 可选 |
| -m | 内存 | 内存限制 | 重要 |
| -t | 时间 | 执行超时 | 重要 |
| -M/-S | ID | 主/从同步模式 | 并行 |
| -x | 目录 | 字典目录 | 优化 |
| -Q | - | QEMU模式 | 特殊 |
| -C | - | 崩溃模式 | 调试 |
| -n | - | 哑模式 | 特殊 |
| -B | bitmap | 基于已有覆盖率 |  |

`extern char *optarg;`

`optarg`定义在`<unistd.h>`头文件中，由 `getopt()` 函数维护，当 `getopt()` 解析到一个需要参数的选项时，会将该参数的地址赋值给 `optarg`，例如此时有`-i /tmp/test`这个选项，如果getopt的第三个参数有`i:`的话，那么此时`*optarg="/tmp/test"`

#### "M" & "S"选项

可以看到是主/从同步模式，`afl-fuzz`的单个fuzzer只占用一个进程，因此只能占用一个核心，而在多核系统中可以同时运行多个fuzzer实例

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `# 启动主实例`  `.``/afl-fuzz` `-i input -o sync_dir -M fuzzer01 .``/target` `@@`    `# 启动从实例`  `.``/afl-fuzz` `-i input -o sync_dir -S fuzzer02 .``/target` `@@`  `.``/afl-fuzz` `-i input -o sync_dir -S fuzzer03 .``/target` `@@` |

实例与实例之间的共享方式是通过将信息输入到同一个文件夹下，每个fuzzer周期性的check文件夹下其他fuzzer生成的用例，将有用的用例加入到自己的用例集中，这里可以详细分析一下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `sync_dir/`  `├── fuzzer01/           # 主实例 (-M)`  `│   ├── queue/          # 测试用例队列`  `│   ├── crashes/        # 崩溃样本`  `│   ├── hangs/         # 挂起样本`  `│   └── .synced/       # 同步状态记录`  `│       ├── fuzzer02   # 记录从fuzzer02同步到的位置`  `│       └── fuzzer03   # 记录从fuzzer03同步到的位置`  `├── fuzzer02/          # 从实例 (-S)`  `│   ├── queue/`  `│   └── .synced/`  `└── fuzzer03/          # 从实例 (-S)`  `├── queue/`  `└── .synced/` |

目录结构大致会是这样的

|  |  |
| --- | --- |
