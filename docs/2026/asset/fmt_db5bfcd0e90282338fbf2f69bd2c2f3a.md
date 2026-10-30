
```c
static void read_testcases(void) {
struct dirent **nl;
s32 nl_cnt;
u32 i;
u8* fn;
/* Auto-detect non-in-place resumption attempts. */
fn = alloc_printf("%s/queue", in_dir);
if (!access(fn, F_OK)) in_dir = fn; else ck_free(fn);
ACTF("Scanning '%s'...", in_dir);
/* We use scandir() + alphasort() rather than readdir() because otherwise,
the ordering of test cases would vary somewhat randomly and would be
difficult to control. */
nl_cnt = scandir(in_dir, &nl, NULL, alphasort);
if (nl_cnt < 0) {
if (errno == ENOENT || errno == ENOTDIR)
SAYF("\n" cLRD "[-] " cRST
"The input directory does not seem to be valid - try again. The fuzzer needs\n"
" one or more test case to start with - ideally, a small file under 1 kB\n"
" or so. The cases must be stored as regular files directly in the input\n"
" directory.\n");
PFATAL("Unable to open '%s'", in_dir);
}
if (shuffle_queue && nl_cnt > 1) {
ACTF("Shuffling queue...");
shuffle_ptrs((void**)nl, nl_cnt);
}
for (i = 0; i < nl_cnt; i++) {
struct stat st;
u8* fn = alloc_printf("%s/%s", in_dir, nl[i]->d_name);
u8* dfn = alloc_printf("%s/.state/deterministic_done/%s", in_dir, nl[i]->d_name);
u8 passed_det = 0;
free(nl[i]); /* not tracked */
if (lstat(fn, &st) || access(fn, R_OK))
PFATAL("Unable to access '%s'", fn);
/* This also takes care of . and .. */
if (!S_ISREG(st.st_mode) || !st.st_size || strstr(fn, "/README.testcases")) {
ck_free(fn);
ck_free(dfn);
continue;
}
if (st.st_size > MAX_FILE)
FATAL("Test case '%s' is too big (%s, limit is %s)", fn,
DMS(st.st_size), DMS(MAX_FILE));
/* Check for metadata that indicates that deterministic fuzzing
is complete for this entry. We don't want to repeat deterministic
fuzzing when resuming aborted scans, because it would be pointless
and probably very time-consuming. */
if (!access(dfn, F_OK)) passed_det = 1;
ck_free(dfn);
add_to_queue(fn, st.st_size, passed_det);
}
```

- 首先获取输入样例的文件夹路径`in_dir`
- 扫描`in_dir`，如果目录下文件的数量少于等于 0 则报错
- 如果设置了`shuffle_queue`就打乱顺序
- 遍历所有文件名，保存在`fn`中
- 过滤掉`.`和`..`这样的路径
- 如果文件的大小超过了`MAX_FILE`则终止
- `add_to_queue`

### `add_to_queue`

```c
static void add_to_queue(u8* fname, u32 len, u8 passed_det) {
struct queue_entry* q = ck_alloc(sizeof(struct queue_entry));
q->fname = fname;
q->len = len;
q->depth = cur_depth + 1;
q->passed_det = passed_det;
if (q->depth > max_depth) max_depth = q->depth;
if (queue_top) {
queue_top->next = q;
queue_top = q;
} else q_prev100 = queue = queue_top = q;
queued_paths++;
pending_not_fuzzed++;
cycles_wo_finds = 0;
/* Set next_100 pointer for every 100th element (index 0, 100, etc) to allow faster iteration. */
if ((queued_paths - 1) % 100 == 0 && queued_paths > 1) {
q_prev100->next_100 = q;
q_prev100 = q;
}
last_path_time = get_cur_time();
}
```

`afl-fuzz`维护一个`queue_entry`的链表，该链表用来保存测试样例，每次调用`add_to_queue`都会将新样例储存到链表头部。

另外还有一个`q_prev100`也是`queue_entry`的链表，但它每 100 个测试样例保存一次。

### `load_auto`

尝试在输入目录下寻找自动生成的字典文件，调用`maybe_add_auto`将相应的字典加入到全局变量`a_extras`中，用于后续字典模式的变异当中。

### `pivot_inputs`

在输出文件夹中创建与输入样例间的硬链接，称之为`orignal`。

### `load_extras`

如果指定了`-x`参数（字典模式），加载对应的字典到全局变量`extras`当中，用于后续字典模式的变异当中。

### `find_timeout`

如果指定了`resuming_fuzz`，即从输出目录当中恢复模糊测试状态，会从之前的模糊测试状态`fuzzer_stats`文件中计算中`timeout`值，保存在`exec_tmout`中。

### `detect_file_args`

检测输入的命令行中是否包含`@@`参数，如果包含的话需要将`@@`替换成目录文件`"%s/.cur_input", out_dir`，使得模糊测试目标程序的命令完整；同时将目录文件`"%s/.cur_input"`路径保存在`out_file`当中，后续变异的内容保存在该文件路径中，用于运行测试目标文件。

### `setup_stdio_file`

如果目标程序的输入不是来源于文件而是来源于标准输入的话，则将目录文件`"%s/.cur_input"`文件打开保存在`out_fd`文件句柄中，后续将标准输入重定向到该文件中；结合`detect_file_args`函数实现了将变异的内容保存在`"%s/.cur_input"`文件中，运行目标测试文件并进行模糊测试。

### `check_binary`

检查二进制文件是否合法。

### `perform_dry_run`

将每个测试样例作为输入去运行目标程序，检查程序是否能够正常工作：

```c
static void perform_dry_run(char** argv) {
struct queue_entry* q = queue;
u32 cal_failures = 0;
u8* skip_crashes = getenv("AFL_SKIP_CRASHES");
while (q) {
u8* use_mem;
u8 res;
s32 fd;
u8* fn = strrchr(q->fname, '/') + 1;
ACTF("Attempting dry run with '%s'...", fn);
fd = open(q->fname, O_RDONLY);
if (fd < 0) PFATAL("Unable to open '%s'", q->fname);
use_mem = ck_alloc_nozero(q->len);
if (read(fd, use_mem, q->len) != q->len)
FATAL("Short read from '%s'", q->fname);
close(fd);
res = calibrate_case(argv, q, use_mem, 0, 1);
ck_free(use_mem);
if (stop_soon) return;
if (res == crash_mode || res == FAULT_NOBITS)
SAYF(cGRA " len = %u, map size = %u, exec speed = %llu us\n" cRST,
q->len, q->bitmap_size, q->exec_us);
......
if (q->var_behavior) WARNF("Instrumentation output varies across runs.");
q = q->next;
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

对每个测试样例使用`calibrate_case`进行测试，并返回运行结果，然后处理其中异常的情况，比如说程序崩溃或运行超时等。

### `calibrate_case`
