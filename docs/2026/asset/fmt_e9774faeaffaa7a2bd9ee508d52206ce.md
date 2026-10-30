
```c
/* Append new test case to the queue. */
static void add_to_queue(u8* fname, u32 len, u8 passed_det) {
struct queue_entry* q = ck_alloc(sizeof(struct queue_entry));
// 通过ck_alloc分配一个 queue_entry 结构体，并进行初始化
q->fname = fname;
q->len = len;
q->depth = cur_depth + 1;
q->passed_det = passed_det;
if (q->depth > max_depth) max_depth = q->depth;
if (queue_top) {
queue_top->next = q;
queue_top = q;
} else q_prev100 = queue = queue_top = q;
queued_paths++; // queue计数器加1
pending_not_fuzzed++; // 待fuzz的样例计数器加1
cycles_wo_finds = 0;
/* Set next_100 pointer for every 100th element (index 0, 100, etc) to allow faster iteration. */
if ((queued_paths - 1) % 100 == 0 && queued_paths > 1) {
q_prev100->next_100 = q;
q_prev100 = q;
}
last_path_time = get_cur_time();
}
```

### 1.12 pivot_inputs 函数

在输出目录中为输入测试用例创建硬链接。

### 1.13 find_timeout 函数

变量 `timeout_given` 没有被设置时，会调用到该函数。该函数主要是在没有指定 `-t` 选项进行 resuming session 时，避免一次次地自动调整超时时间。

### 1.14 detect_file_args

识别参数中是否有`@@`，如果有，则替换为 `out_dir/.cur_input` ，没有则返回：

```c
/* Detect @@ in args. */
EXP_ST void detect_file_args(char** argv) {
u32 i = 0;
u8* cwd = getcwd(NULL, 0);
if (!cwd) PFATAL("getcwd() failed");
while (argv[i]) {
u8* aa_loc = strstr(argv[i], "@@"); // 查找@@
if (aa_loc) {
u8 *aa_subst, *n_arg;
/* If we don't have a file name chosen yet, use a safe default. */
if (!out_file)
out_file = alloc_printf("%s/.cur_input", out_dir);
/* Be sure that we're always using fully-qualified paths. */
if (out_file[0] == '/') aa_subst = out_file;
else aa_subst = alloc_printf("%s/%s", cwd, out_file);
/* Construct a replacement argv value. */
*aa_loc = 0;
n_arg = alloc_printf("%s%s%s", argv[i], aa_subst, aa_loc + 2);
argv[i] = n_arg;
*aa_loc = '@';
if (out_file[0] != '/') ck_free(aa_subst);
}
i++;
}
free(cwd); /* not tracked */
}
```

### 1.15 check_binary 函数

检查指定路径要执行的程序是否存在，是否为shell脚本，同时检查elf文件头是否合法及程序是否被插桩。

## 2. 第一遍fuzz

### 2.1 检查

调用 `get_cur_time()` 函数获取开始时间，检查是否处于 `qemu_mode`。

### 2.2 perform_dry_run 函数

该函数是AFL中的一个关键函数，它会执行 `input` 文件夹下的预先准备的所有测试用例，生成初始化的 queue 和 bitmap，只对初始输入执行一次。函数控制流程图如下：

![image-20210907201943574](upload/attach/202109/779730_ZKDYB9DDX9WG9BB.jpg)

下面将结合函数源码进行解析（删除部分非关键代码）：

```c
/* Perform dry run of all test cases to confirm that the app is working as
expected. This is done only for the initial inputs, and only once. */
static void perform_dry_run(char** argv) {
struct queue_entry* q = queue; // 创建queue_entry结构体
u32 cal_failures = 0;
u8* skip_crashes = getenv("AFL_SKIP_CRASHES"); // 读取环境变量 AFL_SKIP_CRASHES
while (q) { // 遍历队列
u8* use_mem;
u8 res;
s32 fd;
u8* fn = strrchr(q->fname, '/') + 1;
ACTF("Attempting dry run with '%s'...", fn);
fd = open(q->fname, O_RDONLY);
if (fd < 0) PFATAL("Unable to open '%s'", q->fname);
use_mem = ck_alloc_nozero(q->len);
if (read(fd, use_mem, q->len) != q->len)
FATAL("Short read from '%s'", q->fname); // 打开q->fname，读取到分配的内存中
close(fd);
res = calibrate_case(argv, q, use_mem, 0, 1); // 调用函数calibrate_case校准测试用例
ck_free(use_mem);
if (stop_soon) return;
if (res == crash_mode || res == FAULT_NOBITS)
SAYF(cGRA " len = %u, map size = %u, exec speed = %llu us\n" cRST,
q->len, q->bitmap_size, q->exec_us);
switch (res) { // 判断res的值
case FAULT_NONE:
if (q == queue) check_map_coverage(); // 如果为头结点，调用check_map_coverage评估覆盖率
if (crash_mode) FATAL("Test case '%s' does *NOT* crash", fn); // 抛出异常
break;
case FAULT_TMOUT:
if (timeout_given) { // 指定了 -t 选项
/* The -t nn+ syntax in the command line sets timeout_given to '2' and
instructs afl-fuzz to tolerate but skip queue entries that time
out. */
if (timeout_given > 1) {
WARNF("Test case results in a timeout (skipping)");
q->cal_failed = CAL_CHANCES;
cal_failures++;
break;
}
SAYF(... ...);
FATAL("Test case '%s' results in a timeout", fn);
} else {
SAYF(... ...);
FATAL("Test case '%s' results in a timeout", fn);
}
case FAULT_CRASH:
if (crash_mode) break;
if (skip_crashes) {
WARNF("Test case results in a crash (skipping)");
q->cal_failed = CAL_CHANCES;
cal_failures++;
break;
}
if (mem_limit) { // 建议增加内存
SAYF(... ...);
} else {
SAYF(... ...);
}
FATAL("Test case '%s' results in a crash", fn);
case FAULT_ERROR:
FATAL("Unable to execute target application ('%s')", argv[0]);
case FAULT_NOINST: // 测试用例运行没有路径信息
FATAL("No instrumentation detected");
case FAULT_NOBITS: // 没有出现新路径，判定为无效路径
useless_at_start++;
if (!in_bitmap && !shuffle_queue)
WARNF("No new instrumentation output, test case may be useless.");
break;
}
if (q->var_behavior) WARNF("Instrumentation output varies across runs.");
q = q->next; // 读取下一个queue
}
if (cal_failures) {
if (cal_failures == queued_paths)
FATAL("All test cases time out%s, giving up!",
skip_crashes ? " or crash" : "");
WARNF("Skipped %u test cases (%0.02f%%) due to timeouts%s.", cal_failures,
((double)cal_failures) * 100 / queued_paths,
skip_crashes ? " or crashes" : "");
if (cal_failures * 5 > queued_paths)
WARNF(cLRD "High percentage of rejected test cases, check settings!");
}
OKF("All test cases processed.");
}
```

总结以上流程：

1. 进入 `while` 循环，遍历 `input` 队列，从队列中取出 `q->fname`，读取文件内容到分配的内存中，然后关闭文件；
2. 调用 `calibrate_case` 函数校准该测试用例；
3. 根据校准的返回值 `res` ，判断错误类型；
4. 打印错误信息，退出。

### 2.3 calibrate_case 函数

该函数同样为AFL的一个关键函数，用于新测试用例的校准，在处理输入目录时执行，以便在早期就发现有问题的测试用例，并且在发现新路径时，评估新发现的测试用例的是否可变。该函数在 `perform_dry_run`，`save_if_interesting`，`fuzz_one`，`pilot_fuzzing`，`core_fuzzing`函数中均有调用。该函数主要用途是初始化并启动fork server，多次运行测试用例，并用 `update_bitmap_score` 进行初始的byte排序。

函数控制流程图如下：

![image-20210907203020918](upload/attach/202109/779730_MBE655SNXYUZVN6.jpg)

结合源码进行解读如下：
