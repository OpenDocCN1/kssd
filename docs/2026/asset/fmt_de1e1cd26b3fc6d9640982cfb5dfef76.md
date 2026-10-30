
```cpp
while (1) {
// 1. 等待AFL的fork请求
if (read(198, &_afl_temp, 4uLL) != 4) break;
// 2. 创建子进程
LODWORD(v13) = fork();
if (!v13) {
// 子进程：跳转到正常程序执行
goto __afl_fork_resume;
}
// 3. 父进程：发送子进程PID给AFL
_afl_fork_pid = v13;
write(199, &_afl_fork_pid, 4uLL);
// 4. 等待子进程结束并发送退出状态
waitpid(_afl_fork_pid, &_afl_temp, 0);
write(199, &_afl_temp, 4uLL);
}
```

基于边的覆盖位图跟踪：

```cpp
v7 = _afl_prev_loc ^ a4; // 计算边的哈希值
_afl_prev_loc ^= v7; // 等效于：_afl_prev_loc = a4
_afl_prev_loc = _afl_prev_loc >> 1; // 右移1位，作为下次的prev_loc
++*(_BYTE *)(v6 + v7); // 在位图中增加计数
```

## `cull_queue`

```cpp
static void cull_queue(void) {
struct queue_entry* q;
static u8 temp_v[MAP_SIZE >> 3];
u32 i;
if (dumb_mode || !score_changed) return;
score_changed = 0;
memset(temp_v, 255, MAP_SIZE >> 3);
queued_favored = 0;
pending_favored = 0;
q = queue;
while (q) {
q->favored = 0;
q = q->next;
}
/* Let's see if anything in the bitmap isn't captured in temp_v.
If yes, and if it has a top_rated[] contender, let's use it. */
for (i = 0; i < MAP_SIZE; i++)
if (top_rated[i] && (temp_v[i >> 3] & (1 << (i & 7)))) {
u32 j = MAP_SIZE >> 3;
/* Remove all bits belonging to the current entry from temp_v. */
while (j--)
if (top_rated[i]->trace_mini[j])
temp_v[j] &= ~top_rated[i]->trace_mini[j];
top_rated[i]->favored = 1;
queued_favored++;
if (!top_rated[i]->was_fuzzed) pending_favored++;
}
q = queue;
while (q) {
mark_as_redundant(q, !q->favored);
q = q->next;
}
}
```

```cpp
if (dumb_mode || !score_changed) return;
score_changed = 0;
memset(temp_v, 255, MAP_SIZE >> 3); // 将所有位设为1
queued_favored = 0;
pending_favored = 0;
```

如果是盲fuzz模式或者评分没有改变（改变就是出现了新的路径或者有一个更好的抵达某个路径的测试用例的出现）就会直接`return`。

将`temp_v`初始化为全1，表示所有覆盖率位置都需要被覆盖，重置所有测试用例的`favored`标记。

```cpp
for (i = 0; i < MAP_SIZE; i++)
if (top_rated[i] && (temp_v[i >> 3] & (1 << (i & 7)))) {
// 选择这个测试用例
top_rated[i]->favored = 1;
// 从temp_v中移除这个测试用例覆盖的所有位
while (j--)
if (top_rated[i]->trace_mini[j])
temp_v[j] &= ~top_rated[i]->trace_mini[j];
}
```

贪心选择算法，如果`bitmap`上的某一个位置有最佳的测试用例而且该位置也未被覆盖，将该测试用例标记位优选，然后从`temp_v`中移除该测试用例能覆盖到的所有位置。

```cpp
q = queue;
while (q) {
mark_as_redundant(q, !q->favored);
q = q->next;
}
```

将未被标记为优选的测试用例标记为冗余。

> 被标记为优选的测试样例会被优先变异

## `show_init_stats`

```cpp
static void show_init_stats(void) {
struct queue_entry* q = queue;
u32 min_bits = 0, max_bits = 0;
u64 min_us = 0, max_us = 0;
u64 avg_us = 0;
u32 max_len = 0;
if (total_cal_cycles) avg_us = total_cal_us / total_cal_cycles;
while (q) {
if (!min_us || q->exec_us < min_us) min_us = q->exec_us;
if (q->exec_us > max_us) max_us = q->exec_us;
if (!min_bits || q->bitmap_size < min_bits) min_bits = q->bitmap_size;
if (q->bitmap_size > max_bits) max_bits = q->bitmap_size;
if (q->len > max_len) max_len = q->len;
q = q->next;
}
SAYF("\n");
if (avg_us > (qemu_mode ? 50000 : 10000))
WARNF(cLRD "The target binary is pretty slow! See %s/perf_tips.txt.",
doc_path);
/* Let's keep things moving with slow binaries. */
if (avg_us > 50000) havoc_div = 10; /* 0-19 execs/sec */
else if (avg_us > 20000) havoc_div = 5; /* 20-49 execs/sec */
else if (avg_us > 10000) havoc_div = 2; /* 50-100 execs/sec */
if (!resuming_fuzz) {
if (max_len > 50 * 1024)
WARNF(cLRD "Some test cases are huge (%s) - see %s/perf_tips.txt!",
DMS(max_len), doc_path);
else if (max_len > 10 * 1024)
WARNF("Some test cases are big (%s) - see %s/perf_tips.txt.",
DMS(max_len), doc_path);
if (useless_at_start && !in_bitmap)
WARNF(cLRD "Some test cases look useless. Consider using a smaller set.");
if (queued_paths > 100)
WARNF(cLRD "You probably have far too many input files! Consider trimming down.");
else if (queued_paths > 20)
WARNF("You have lots of input files; try starting small.");
}
OKF("Here are some useful stats:\n\n"
cGRA " Test case count : " cRST "%u favored, %u variable, %u total\n"
cGRA " Bitmap range : " cRST "%u to %u bits (average: %0.02f bits)\n"
cGRA " Exec timing : " cRST "%s to %s us (average: %s us)\n",
queued_favored, queued_variable, queued_paths, min_bits, max_bits,
((double)total_bitmap_size) / (total_bitmap_entries ? total_bitmap_entries : 1),
DI(min_us), DI(max_us), DI(avg_us));
if (!timeout_given) {
/* Figure out the appropriate timeout. The basic idea is: 5x average or
1x max, rounded up to EXEC_TM_ROUND ms and capped at 1 second.
If the program is slow, the multiplier is lowered to 2x or 3x, because
random scheduler jitter is less likely to have any impact, and because
our patience is wearing thin =) */
if (avg_us > 50000) exec_tmout = avg_us * 2 / 1000;
else if (avg_us > 10000) exec_tmout = avg_us * 3 / 1000;
else exec_tmout = avg_us * 5 / 1000;
exec_tmout = MAX(exec_tmout, max_us / 1000);
exec_tmout = (exec_tmout + EXEC_TM_ROUND) / EXEC_TM_ROUND * EXEC_TM_ROUND;
if (exec_tmout > EXEC_TIMEOUT) exec_tmout = EXEC_TIMEOUT;
ACTF("No -t option specified, so I'll use exec timeout of %u ms.",
exec_tmout);
timeout_given = 1;
} else if (timeout_given == 3) {
ACTF("Applying timeout settings from resumed session (%u ms).", exec_tmout);
}
/* In dumb mode, re-running every timing out test case with a generous time
limit is very expensive, so let's select a more conservative default. */
if (dumb_mode && !getenv("AFL_HANG_TMOUT"))
hang_tmout = MIN(EXEC_TIMEOUT, exec_tmout * 2 + 100);
OKF("All set and ready to roll!");
}
```

```cpp
if (total_cal_cycles) avg_us = total_cal_us / total_cal_cycles;
while (q) {
if (!min_us || q->exec_us < min_us) min_us = q->exec_us;
if (q->exec_us > max_us) max_us = q->exec_us;
if (!min_bits || q->bitmap_size < min_bits) min_bits = q->bitmap_size;
if (q->bitmap_size > max_bits) max_bits = q->bitmap_size;
if (q->len > max_len) max_len = q->len;
q = q->next;
}
```

- 最小/最大执行时间 `min_us`/`max_us`
- 最小/最大位图大小 `min_bits`/`max_bits`
- 最大输入长度 `max_len`
- 平均执行时间 `avg_us = total_cal_us / total_cal_cycles`

```cpp
if (avg_us > (qemu_mode ? 50000 : 10000))
WARNF(cLRD "The target binary is pretty slow! See %s/perf_tips.txt.",
doc_path);
```

qemu模式下的平均执行时间给的更加宽松，也能看出qemu模式下的fuzz效率更低。

```cpp
if (avg_us > 50000) havoc_div = 10; /* 0-19 execs/sec */
else if (avg_us > 20000) havoc_div = 5; /* 20-49 execs/sec */
else if (avg_us > 10000) havoc_div = 2; /* 50-100 execs/sec */
```

平均执行时间更慢则降低变异强度（增大`havoc_div`）。

```cpp
if (!resuming_fuzz) {
if (max_len > 50 * 1024)
WARNF(cLRD "Some test cases are huge (%s) - see %s/perf_tips.txt!",
DMS(max_len), doc_path);
else if (max_len > 10 * 1024)
WARNF("Some test cases are big (%s) - see %s/perf_tips.txt.",
DMS(max_len), doc_path);
if (useless_at_start && !in_bitmap)
WARNF(cLRD "Some test cases look useless. Consider using a smaller set.");
if (queued_paths > 100)
WARNF(cLRD "You probably have far too many input files! Consider trimming down.");
else if (queued_paths > 20)
WARNF("You have lots of input files; try starting small.");
}
```

在非恢复对话下，给出一些样本的可读性建议。

```cpp
if (avg_us > 50000) exec_tmout = avg_us * 2 / 1000;
else if (avg_us > 10000) exec_tmout = avg_us * 3 / 1000;
else exec_tmout = avg_us * 5 / 1000;
exec_tmout = MAX(exec_tmout, max_us / 1000);
exec_tmout = (exec_tmout + EXEC_TM_ROUND) / EXEC_TM_ROUND * EXEC_TM_ROUND;
if (exec_tmout > EXEC_TIMEOUT) exec_tmout = EXEC_TIMEOUT;
```

根据平均执行时间/最大执行时间推导出较佳的`exec_tmout`。

```cpp
if (dumb_mode && !getenv("AFL_HANG_TMOUT"))
hang_tmout = MIN(EXEC_TIMEOUT, exec_tmout * 2 + 100);
```

如果是盲fuzz且未设环境变量则设定更保守的挂起时间。

## `find_start_position`
