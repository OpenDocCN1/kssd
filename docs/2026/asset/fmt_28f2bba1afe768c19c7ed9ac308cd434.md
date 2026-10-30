
```cpp
queue_cur = queue_cur->next;
current_entry++;
```

移动到下一个队列

```cpp
if (stop_soon == 2) {
if (child_pid > 0) kill(child_pid, SIGKILL);
if (forksrv_pid > 0) kill(forksrv_pid, SIGKILL);
}
if (waitpid(forksrv_pid, NULL, 0) <= 0) {
WARNF("error waitpid\n");
}
```

- `stop_soon == 2`：程序化停止（非用户手动停止）

- 终止子进程和fork服务器进程

- 等待fork服务器进程结束以获取资源使用统计

```cpp
write_bitmap();
write_stats_file(0, 0, 0);
save_auto();
```

记录位图，状态文件和自动提取的字典

## fuzz_one

### 初始化

```cpp
// 优先处理favored条目
if (pending_favored) {
if ((queue_cur->was_fuzzed || !queue_cur->favored) &&
UR(100) < SKIP_TO_NEW_PROB) return 1;
}
// 概率性跳过非favored条目
else if (!dumb_mode && !queue_cur->favored && queued_paths > 10) {
if (queue_cycle > 1 && !queue_cur->was_fuzzed) {
if (UR(100) < SKIP_NFAV_NEW_PROB) return 1;
} else {
if (UR(100) < SKIP_NFAV_OLD_PROB) return 1;
}
}
```

```cpp
// 将测试用例映射到内存
fd = open(queue_cur->fname, O_RDONLY);
orig_in = in_buf = mmap(0, len, PROT_READ | PROT_WRITE, MAP_PRIVATE, fd, 0);
out_buf = ck_alloc_nozero(len);
```

将测试用例映射到内存

```cpp
if (queue_cur->cal_failed) {
u8 res = FAULT_TMOUT;
if (queue_cur->cal_failed < CAL_CHANCES) {
queue_cur->exec_cksum = 0;
res = calibrate_case(argv, queue_cur, in_buf, queue_cycle - 1, 0);
if (res == FAULT_ERROR)
FATAL("Unable to execute target application");
}
if (stop_soon || res != crash_mode) {
cur_skipped_paths++;
goto abandon_entry;
}
}
```

重新校准之前失败的测试用例，确保测试用例能正常运行，超过重试次数就直接放弃

```cpp
if (!dumb_mode && !queue_cur->trim_done) {
u8 res = trim_case(argv, queue_cur, in_buf);
if (res == FAULT_ERROR)
FATAL("Unable to execute target application");
if (stop_soon) {
cur_skipped_paths++;
goto abandon_entry;
}
queue_cur->trim_done = 1;
if (len != queue_cur->len) len = queue_cur->len;
}
```

修剪测试用例，移除不影响执行路径的冗余字节，减少文件大小，提高变异效率，每个测试用例只修剪一次

> `trim_case`函数之后分析

```cpp
orig_perf = perf_score = calculate_score(queue_cur);
// 跳过确定性变异的条件
if (skip_deterministic || queue_cur->was_fuzzed || queue_cur->passed_det)
goto havoc_stage;
// 分布式模糊测试中的分工
if (master_max && (queue_cur->exec_cksum % master_max) != master_id - 1)
goto havoc_stage;
```

计算修剪之后的性能，直接到havoc变异的条件判断

### 确定性变异

#### 位翻转

```cpp
#define FLIP_BIT(_ar, _b) do { \
u8* _arf = (u8*)(_ar); \
u32 _bf = (_b); \
_arf[(_bf) >> 3] ^= (128 >> ((_bf) & 7)); \
} while (0)
```

精确翻转任意位置的单个位（`_ar`是字符串指针，`_b`是要翻转的位）

```cpp
stage_name = "bitflip 1/1";
stage_max = len << 3; // 总位数
for (stage_cur = 0; stage_cur < stage_max; stage_cur++) {
FLIP_BIT(out_buf, stage_cur);
if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry;
FLIP_BIT(out_buf, stage_cur); // 恢复
// 自动字典构建逻辑
if (!dumb_mode && (stage_cur & 7) == 7) {
u32 cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);
// ... 检测魔术字符串
}
}
```

会对该测试用例的每一个位都进行翻转，然后调用`common_fuzz_stuff`函数

```cpp
EXP_ST u8 common_fuzz_stuff(char** argv, u8* out_buf, u32 len) {
u8 fault;
if (post_handler) {
out_buf = post_handler(out_buf, &len);
if (!out_buf || !len) return 0;
}
write_to_testcase(out_buf, len);
fault = run_target(argv, exec_tmout);
if (stop_soon) return 1;
if (fault == FAULT_TMOUT) {
if (subseq_tmouts++ > TMOUT_LIMIT) {
cur_skipped_paths++;
return 1;
}
} else subseq_tmouts = 0;
/* Users can hit us with SIGUSR1 to request the current input
to be abandoned. */
if (skip_requested) {
skip_requested = 0;
cur_skipped_paths++;
return 1;
}
/* This handles FAULT_ERROR for us: */
queued_discovered += save_if_interesting(argv, out_buf, len, fault);
if (!(stage_cur % stats_update_freq) || stage_cur + 1 == stage_max)
show_stats();
return 0;
}
```

运行，然后保存有趣的测试用例

```cpp
queued_discovered += save_if_interesting(argv, out_buf, len, fault);
```

- 检查执行结果是否发现了新的代码路径或崩溃

- 如果有价值，将测试用例保存到队列中供后续分析

- 更新发现的队列项目计数

运行完这个函数如果超时多次或者跳过就会直接放弃这次fuzz

之后就是一样的多次翻转

#### 自动字典构建逻辑

```cpp
if (!dumb_mode && (stage_cur & 7) == 7) {
u32 cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);
if (stage_cur == stage_max - 1 && cksum == prev_cksum) {
/* If at end of file and we are still collecting a string, grab the
final character and force output. */
if (a_len < MAX_AUTO_EXTRA) a_collect[a_len] = out_buf[stage_cur >> 3];
a_len++;
if (a_len >= MIN_AUTO_EXTRA && a_len <= MAX_AUTO_EXTRA)
maybe_add_auto(a_collect, a_len);
} else if (cksum != prev_cksum) {
/* Otherwise, if the checksum has changed, see if we have something
worthwhile queued up, and collect that if the answer is yes. */
if (a_len >= MIN_AUTO_EXTRA && a_len <= MAX_AUTO_EXTRA)
maybe_add_auto(a_collect, a_len);
a_len = 0;
prev_cksum = cksum;
}
}
```

不在`dumb_mode`下且只在每个字节的**最低位**翻转时检查（`stage_cur & 7 == 7`）

```cpp
u32 cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);
```

计算当前程序执行路径的哈希值

```cpp
if (stage_cur == stage_max - 1 && cksum == prev_cksum) {
if (a_len < MAX_AUTO_EXTRA) a_collect[a_len] = out_buf[stage_cur >> 3];
a_len++;
if (a_len >= MIN_AUTO_EXTRA && a_len <= MAX_AUTO_EXTRA)
maybe_add_auto(a_collect, a_len);
}
```

当到达文件末尾且仍在收集字符串，且长度在3个字节到32个字节之间，直接添加进字典

```cpp
else if (cksum != prev_cksum) {
if (a_len >= MIN_AUTO_EXTRA && a_len <= MAX_AUTO_EXTRA)
maybe_add_auto(a_collect, a_len);
a_len = 0;
prev_cksum = cksum;
}
```

路径哈希发生变化，说明找到了一个完整的语法标识，加入字典

```cpp
if (cksum != queue_cur->exec_cksum) {
if (a_len < MAX_AUTO_EXTRA) a_collect[a_len] = out_buf[stage_cur >> 3];
a_len++;
}
```

当位翻转真正产生差异时才收集字符

#### 多位翻转

- `bitflip 2/1`: 连续翻转2位

- `bitflip 4/1`: 连续翻转4位

- `bitflip 8/8`: 整字节翻转（构建效应器映射）

- `bitflip 16/8`: 连续2字节翻转

- `bitflip 32/8`: 连续4字节翻转

其中在8/8的整字节翻转中有 AFL 中 重要的 Effective Map

```cpp
if (!eff_map[EFF_APOS(stage_cur)]) {
u32 cksum;
/* If in dumb mode or if the file is very short, just flag everything
without wasting time on checksums. */
if (!dumb_mode && len >= EFF_MIN_LEN)
cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);
else
cksum = ~queue_cur->exec_cksum;
if (cksum != queue_cur->exec_cksum) {
eff_map[EFF_APOS(stage_cur)] = 1;
eff_cnt++;
}
}
out_buf[stage_cur] ^= 0xFF;
}
```

在个`eff_map`中，如果该字节整个翻转不会造成程序的执行路径的改变，则将其标记，在之后的确定性变异中直接跳过该字节

```cpp
#define EFF_APOS(_p) ((_p) >> EFF_MAP_SCALE2)
#define EFF_REM(_x) ((_x) & ((1 << EFF_MAP_SCALE2) - 1))
#define EFF_ALEN(_l) (EFF_APOS(_l) + !!EFF_REM(_l))
#define EFF_SPAN_ALEN(_p, _l) (EFF_APOS((_p) + (_l) - 1) - EFF_APOS(_p) + 1)
```

为了节省内存，每8个连续字节共享一个映射位，这意味着如果8字节块中任何一个字节有效，整个块都被标记为有效

```cpp
if (eff_cnt != EFF_ALEN(len) &&
eff_cnt * 100 / EFF_ALEN(len) > EFF_MAX_PERC) {
memset(eff_map, 1, EFF_ALEN(len));
}
```

如果有效字节密度超过90%，则标记全部都是有效的

#### 算数运算

```cpp
for (i = 0; i < len; i++) {
u8 orig = out_buf[i];
if (!eff_map[EFF_APOS(i)]) { // 跳过无效字节
stage_max -= 2 * ARITH_MAX;
continue;
}
for (j = 1; j <= ARITH_MAX; j++) {
// 加法变异
u8 r = orig ^ (orig + j);
if (!could_be_bitflip(r)) { // 避免重复
out_buf[i] = orig + j;
if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry;
}
// 减法变异
r = orig ^ (orig - j);
if (!could_be_bitflip(r)) {
out_buf[i] = orig - j;
if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry;
}
}
out_buf[i] = orig; // 恢复原值
}
```

```cpp
static u8 could_be_bitflip(u32 xor_val) {
u32 sh = 0;
if (!xor_val) return 1;
/* Shift left until first bit set. */
while (!(xor_val & 1)) { sh++; xor_val >>= 1; }
/* 1-, 2-, and 4-bit patterns are OK anywhere. */
if (xor_val == 1 || xor_val == 3 || xor_val == 15) return 1;
/* 8-, 16-, and 32-bit patterns are OK only if shift factor is
divisible by 8, since that's the stepover for these ops. */
if (sh & 7) return 0;
if (xor_val == 0xff || xor_val == 0xffff || xor_val == 0xffffffff)
return 1;
return 0;
}
```

判断是否是位翻转后的结果，避免重复fuzz

就是对每一个位进行加减法的变异

之后就是16和32位的加减法变异

#### 有趣值变异
