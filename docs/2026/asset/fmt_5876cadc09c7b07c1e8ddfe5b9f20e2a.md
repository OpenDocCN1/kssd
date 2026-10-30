
- 如果满足 `dumb_mode == 1 || no_forkserver`（无 forkserver，以子进程的方式 `execve`）。
- `waitpid` 等待子进程 `child_pid` 的 status。
- 否则直接从 forkserver 的状态管道中读 status。
- 计算执行时间 `exec_ms`。

- 重置 timer，`total_execs++;`，执行次数的计数加 1。
- `prev_timed_out = child_timed_out;`

- 判断子进程是否是异常退出。如果是，判断异常退出的原因后 `return`。

- 判断退出状态是否是 `uses_asan && WEXITSTATUS(status) == MSAN_ERROR`。如果是，设置 `kill_signal = 0;` 后 `return FAULT_CRASH`。
- 接下来判断是否是在无 forkserver 情况下 `execve` 执行失败。
- 如果最慢执行时间小于当前执行时间，并且 `timeout < exec_tmout`，则更新 `slowest_exec_ms = exec_ms;`。

- `return FAULT_NONE`
- 结束

### `classify_counts(u64 *mem)`

`classify_counts((u64 *) trace_bits)`

```c
static inline void classify_counts(u64 *mem) {
  u32 i = MAP_SIZE >> 3; // 八个一组，一共 i 组

  while (i--) { // 每一组扫描
    /* Optimize for sparse bitmaps. */
    if (unlikely(*mem)) { // 如果对应的 mem 中的值不为 0
      u16 *mem16 = (u16 *) mem;
      mem16[0] = count_class_lookup16[mem16[0]];
      mem16[1] = count_class_lookup16[mem16[1]];
      mem16[2] = count_class_lookup16[mem16[2]];
      mem16[3] = count_class_lookup16[mem16[3]];
    }

    mem++;
  }
}
```

### `count_bytes(u8 *mem)`

调用：`count_bytes(trace_bits)`

实现如下：

```c
#define FF(_b) (0xff << ((_b) << 3))

/* Count the number of bytes set in the bitmap. Called fairly sporadically,

mostly to update the status screen or calibrate and examine confirmed

new paths. */

static u32 count_bytes(u8 *mem) {
  u32 *ptr = (u32 *) mem; // ptr 指向 trace bits 的首位
  u32 i = (MAP_SIZE >> 2); // 四个字节一组
  u32 ret = 0; // ret 计数初始化为 0

  while (i--) { // 一组一组扫描
    u32 v = *(ptr++); // v 每次取 4 个字节

    if (!v) continue; // 如果 4 个字节全部是 0，直接跳过这四个字节，取下一组。

    if (v & FF(0)) ret++; // 0xff
    if (v & FF(1)) ret++; // 0xff00
    if (v & FF(2)) ret++; // 0xff0000
    if (v & FF(3)) ret++; // 0xff000000
  }

  return ret;
}
```

其实就是四个一组，扫描哪个字节不为 0。若扫描到不为 0 的就 `ret++`；最后返回 `ret`。

### `update_bitmap_score(struct queue_entry *q)`

当我们碰到一个新路径时，判断此路径是否是更有利的，即是否是能遍历到 bitmap 中的 bit 的最小路径集合。

```c
u32 i;

u64 fav_factor = q->exec_us * q->len;
```

- 首先在开始时定义了一个偏好因子 `fav_factor = q->exec_us * q->len`。
- 接着进入一个 for 循环：`for (i = 0; i < MAP_SIZE; i++)`
- 扫描 `trace_bits` 中每一位 `trace_bits[i]`。
- 如果 `trace_bits[i] = 1`（代表这是已经被覆盖到的路径）。
- 若设置了 `top_rated[i]`。
- 判断当前的 `fav_factor` 是否大于 `top_rated[i]` 列表中维护的因子的值。
- 如果大于，则说明 `top_rated[i]` 中存的是最优的 `fav_factor`，直接 `continue` 掉此次循环。
- 否则 `top_rated[i]->tc_ref` 计数减 1，并 `free` 掉对应的 `top_rated[i]->trace_mini`，然后置空。
- 接下来相当于重新更新对应的 `top_rated[i]` 中维护的相关数据。
- 令 `top_rated[i] = q`。
- `q->tc_ref++`，计数增加。
- 如果 `q->trace_mini` 为 0：
  - 那么令 `q->trace_mini = ck_alloc(MAP_SIZE >> 3)`（分配 8192 字节）。
  - 调用 `minimize_bits(q->trace_mini, trace_bits)` 对原 `trace_bits` 进行压缩，放到只有原先 1/8 大小的 `q->trace_mini` 中。
- 最后令 `score_changed = 1`。
- 结束。

### `minimize_bits(u8 *dst, u8 *src)`

对 `src` 进行压缩，压缩到原来的 1/8。

```c
/* Compact trace bytes into a smaller bitmap. We effectively just drop the

count information here. This is called only sporadically, for some

new paths. */

static void minimize_bits(u8 *dst, u8 *src) {
  u32 i = 0;

  while (i < MAP_SIZE) {
    if (*(src++)) dst[i >> 3] |= 1 << (i & 7);

    i++;
  }
}
```

- `*(src++)` 每次取 `trace_bits[]` 数组中一个元素的值。
- `i >> 3` 得到了要压缩到的 `dst` 的 index，即这个数据会被压缩到 `dst[index]` 的 bits 中。
- `i & 7` 相当于 `i % 8`（即不考虑高位，只保留低三 bits），相当于计算要放到 `dst[]` 的一个 byte 中的哪个 bit。
- 最后将 `1 << (i % 8)` 把 1 放到对应的 bits 上。
- 压缩结束。

### `check_map_coverage()`

```c
/* Examine map coverage. Called once, for first test case. */

static void check_map_coverage(void) {
  u32 i;

  if (count_bytes(trace_bits) < 100) return;

  for (i = (1 << (MAP_SIZE_POW2 - 1)); i < MAP_SIZE; i++)
    if (trace_bits[i]) return;

  WARNF("Recompile binary with newer version of afl to improve coverage!");
}
```

- 利用 `count_bytes(trace_bits)` 统计发现的路径数量（不为 0 的字节），小于 100 则 `return`。
- 扫描 `trace_bits` 后半部分，如果有不为零的就返回。
- 否则告诉用户改进覆盖率。

### `cull_queue()`

用于精简队列。

```c
struct queue_entry *q;
static u8 temp_v[MAP_SIZE >> 3]; // temp_v[8192]
u32 i;
```

- 如果设置了 `dumb_mode` 或者未设置 `score_changed`，直接 `return`。
- 设置 `score_changed = 0`，初始化 `temp_v` 数组，`queued_favored = 0; pending_favored = 0;`。
- 从 queue 队头开始扫描，设置队列中每一项的 `q->favored = 0`。
- 接下来进入一个 `for (i = 0; i < MAP_SIZE; i++)`。
- 如果对于当前 queue entry，`top_rated[i]` 被设置了，并且对应的 `temp_v[i >> 3]` 的第 `(1 << (i & 7))` bit 被置位（即对应的 path 是否置位）。
- 在 `temp_v` 中清除掉对应的 path（将此 bit 位置 0）。
- 令 `top_rated[i]->favored = 1`，然后 `queued_favored` 计数加一。
- 如果当前的 `top_rated[i]->was_fuzzed` 为 0（没有被 fuzz 过），那么 `pending_favored` 计数加一。
- 接着扫描 queue 队列。
- 对每一项调用：`mark_as_redundant(q, !q->favored)`。
- 若 state 设置了，尝试创建并打开：`out_dir/queue/.state/redundant_edges/q->fname`。
- 否则删除对应的 `out_dir/queue/.state/redundant_edges/q->fname`。

### `mark_as_redundant(struct queue_entry *q, u8 state)`

- 如果当前 `state == q->fs_redundant`，那么直接返回。
- 否则将 `q->fs_redundan` 标记为 `state`。

### `write_stats_file(double bitmap_cvg, double stability, double eps)`

- 创建或打开对应的：`out_dir/fuzzer_stats`。
- 然后将一些状态信息写入。

```c
"start_time : %llu\n"
"last_update : %llu\n"
"fuzzer_pid : %u\n"
"cycles_done : %llu\n"
"execs_done : %llu\n"
"execs_per_sec : %0.02f\n"
"paths_total : %u\n"
"paths_favored : %u\n"
"paths_found : %u\n"
"paths_imported : %u\n"
"max_depth : %u\n"
"cur_path : %u\n" /* Must match find_start_position() */
"pending_favs : %u\n"
"pending_total : %u\n"
"variable_paths : %u\n"
"stability : %0.02f%%\n"
"bitmap_cvg : %0.02f%%\n"
"unique_crashes : %llu\n"
"unique_hangs : %llu\n"
"last_path : %llu\n"
"last_crash : %llu\n"
"last_hang : %llu\n"
"execs_since_crash : %llu\n"
"exec_timeout : %u\n" /* Must match find_timeout() */
"afl_banner : %s\n"
"afl_version : " VERSION "\n"
"target_mode : %s%s%s%s%s%s%s\n"
"command_line : %s\n"
"slowest_exec_ms : %llu\n",
```

# 附录

## 一些参数/名词

### GCC 之 `fsanitize`

gcc 从 4.8 版本起，集成了 Address Sanitizer 工具，可以用来检查内存访问的错误（编译时指定 `-fsanitize=address`）。

### GCC 之 `FORTIFY_SOURCE`

### ASAN

AddressSanitizer（ASan）是一个快速的内存错误检测器，基于编译器插桩（LLVM）。

基于 LLVM 的一个内存错误快速检测器。

## 一些函数/宏

### `UR(u32 limit)`

```c
/* Generate a random number (from 0 to limit - 1). This may
have slight bias. */
static inline u32 UR(u32 limit) {
  if (unlikely(!rand_cnt--)) {
    u32 seed[2];

    ck_read(dev_urandom_fd, &seed, sizeof(seed), "/dev/urandom");
    srandom(seed[0]);
    rand_cnt = (RESEED_RNG / 2) + (seed[1] % RESEED_RNG);
  }

  return random() % limit;
}
```

### `alloc_printf`

```c
/* User-facing macro to sprintf() to a dynamically allocated buffer. */
#define alloc_printf(_str...) ({ \
  u8* _tmp; \
  s32 _len = snprintf(NULL, 0, _str); \
  if (_len < 0) FATAL("Whoa, snprintf() fails?!"); \
  _tmp = ck_alloc(_len + 1); \
  snprintf((char*)_tmp, _len + 1, _str); \
  _tmp; \
})
```

`alloc_printf` 通过 `snprintf` 的一个小陷阱来做动态分配，说起来去年校赛好像学长还出了个 `snprintf` 的题（笑）。大概就是说，第一个 `snprintf` 那里 `size = 0`，那么返回值此时是“如果在理想情况下，字符串需要的长度”。相当于获取了一下 length，接下来再通过获取的 length 分配空间，然后二次调用 `snprintf` 把字符串放过去。

### `strrchr`
