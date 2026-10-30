
```c
static u8 run_target(char** argv, u32 timeout) {
static struct itimerval it;
static u32 prev_timed_out = 0;
static u64 exec_ms = 0;
int status = 0;
u32 tb4;
child_timed_out = 0;
/* After this memset, trace_bits[] are effectively volatile, so we
must prevent any earlier operations from venturing into that
territory. */
memset(trace_bits, 0, MAP_SIZE); // 将trace_bits全部置0，清空共享内存
MEM_BARRIER();
/* If we're running in "dumb" mode, we can't rely on the fork server
logic compiled into the target program, so we will just keep calling
execve(). There is a bit of code duplication between here and
init_fork server(), but c'est la vie. */
if (dumb_mode == 1 || no_fork server) { // 如果是dumb_mode模式且没有fork server
child_pid = fork(); // 直接fork出一个子进程
if (child_pid < 0) PFATAL("fork() failed");
if (!child_pid) {
... ...
/* Isolate the process and configure standard descriptors. If out_file is
specified, stdin is /dev/null; otherwise, out_fd is cloned instead. */
setsid();
dup2(dev_null_fd, 1);
dup2(dev_null_fd, 2);
if (out_file) {
dup2(dev_null_fd, 0);
} else {
dup2(out_fd, 0);
close(out_fd);
}
/* On Linux, would be faster to use O_CLOEXEC. Maybe TODO. */
close(dev_null_fd);
close(out_dir_fd);
close(dev_urandom_fd);
close(fileno(plot_file));
/* Set sane defaults for ASAN if nothing else specified. */
setenv("ASAN_OPTIONS", "abort_on_error=1:"
"detect_leaks=0:"
"symbolize=0:"
"allocator_may_return_null=1", 0);
setenv("MSAN_OPTIONS", "exit_code=" STRINGIFY(MSAN_ERROR) ":"
"symbolize=0:"
"msan_track_origins=0", 0);
execv(target_path, argv); // 让子进程execv执行目标程序
/* Use a distinctive bitmap value to tell the parent about execv()
falling through. */
*(u32*)trace_bits = EXEC_FAIL_SIG; // execv执行失败，写入 EXEC_FAIL_SIG
exit(0);
}
} else {
s32 res;
/* In non-dumb mode, we have the fork server up and running, so simply
tell it to have at it, and then read back PID. */
// 如果并不是处在dumb_mode模式，说明fork server已经启动了，我们只需要进行
// 控制管道的写和状态管道的读即可
if ((res = write(fsrv_ctl_fd, &prev_timed_out, 4)) != 4) {
if (stop_soon) return 0;
RPFATAL(res, "Unable to request new process from fork server (OOM?)");
}
if ((res = read(fsrv_st_fd, &child_pid, 4)) != 4) {
if (stop_soon) return 0;
RPFATAL(res, "Unable to request new process from fork server (OOM?)");
}
if (child_pid <= 0) FATAL("Fork server is misbehaving (OOM?)");
}
/* Configure timeout, as requested by user, then wait for child to terminate. */
// 配置超时，等待子进程结束
it.it_value.tv_sec = (timeout / 1000);
it.it_value.tv_usec = (timeout % 1000) * 1000;
setitimer(ITIMER_REAL, &it, NULL);
/* The SIGALRM handler simply kills the child_pid and sets child_timed_out. */
if (dumb_mode == 1 || no_fork server) {
if (waitpid(child_pid, &status, 0) <= 0) PFATAL("waitpid() failed");
} else {
s32 res;
if ((res = read(fsrv_st_fd, &status, 4)) != 4) {
if (stop_soon) return 0;
RPFATAL(res, "Unable to communicate with fork server (OOM?)");
}
}
if (!WIFSTOPPED(status)) child_pid = 0;
getitimer(ITIMER_REAL, &it);
exec_ms = (u64) timeout - (it.it_value.tv_sec * 1000 +
it.it_value.tv_usec / 1000); // 计算执行时间
it.it_value.tv_sec = 0;
it.it_value.tv_usec = 0;
setitimer(ITIMER_REAL, &it, NULL);
total_execs++;
/* Any subsequent operations on trace_bits must not be moved by the
compiler below this point. Past this location, trace_bits[] behave
very normally and do not have to be treated as volatile. */
MEM_BARRIER();
tb4 = *(u32*)trace_bits;
// 分别执行64和32位下的classify_counts，设置trace_bits所在的mem
#ifdef WORD_SIZE_64
classify_counts((u64*)trace_bits);
#else
classify_counts((u32*)trace_bits);
#endif /* ^WORD_SIZE_64 */
prev_timed_out = child_timed_out;
/* Report outcome to caller. */
if (WIFSIGNALED(status) && !stop_soon) {
kill_signal = WTERMSIG(status);
if (child_timed_out && kill_signal == SIGKILL) return FAULT_TMOUT;
return FAULT_CRASH;
}
/* A somewhat nasty hack for MSAN, which doesn't support abort_on_error and
must use a special exit code. */
if (uses_asan && WEXITSTATUS(status) == MSAN_ERROR) {
kill_signal = 0;
return FAULT_CRASH;
}
if ((dumb_mode == 1 || no_fork server) && tb4 == EXEC_FAIL_SIG)
return FAULT_ERROR;
/* It makes sense to account for the slowest units only if the testcase was run
under the user defined timeout. */
if (!(timeout > exec_tmout) && (slowest_exec_ms < exec_ms)) {
slowest_exec_ms = exec_ms;
}
return FAULT_NONE;
}
```

## 2.6 `update_bitmap_score` 函数

当我们发现一个新路径时，需要判断发现的新路径是否更“favorable”，也就是是否包含最小的路径集合能遍历到所有bitmap中的位，并在之后的fuzz过程中聚焦在这些路径上。

以上过程的第一步是为bitmap中的每个字节维护一个 `top_rated[]` 的列表，这里会计算究竟哪些位置是更“合适”的，该函数主要实现该过程。

函数的控制流程图如下：

![image-20210909111119930](upload/attach/202109/779730_8MEFWMMRJX7S2QJ.jpg)

结合源码进行解释：

```c
static void update_bitmap_score(struct queue_entry* q) {
u32 i;
u64 fav_factor = q->exec_us * q->len;
// 首先计算case的fav_factor，计算方法是执行时间和样例大小的乘积
/* For every byte set in trace_bits[], see if there is a previous winner,
and how it compares to us. */
for (i = 0; i < MAP_SIZE; i++) // 遍历trace_bits数组
if (trace_bits[i]) { // 不为0，表示已经被覆盖到的路径
if (top_rated[i]) { // 检查top_rated是否存在
/* Faster-executing or smaller test cases are favored. */
if (fav_factor > top_rated[i]->exec_us * top_rated[i]->len) continue; // 判断哪个计算结果更小
// 如果top_rated[i]的更小，则代表它的更优，不做处理，继续遍历下一个路径；
// 如果q的更小，就执行以下代码：
/* Looks like we're going to win. Decrease ref count for the
previous winner, discard its trace_bits[] if necessary. */
if (!--top_rated[i]->tc_ref) {
ck_free(top_rated[i]->trace_mini);
top_rated[i]->trace_mini = 0;
}
}
/* Insert ourselves as the new winner. */
top_rated[i] = q; // 设置为当前case
q->tc_ref++;
if (!q->trace_mini) { // 为空
q->trace_mini = ck_alloc(MAP_SIZE >> 3);
minimize_bits(q->trace_mini, trace_bits);
}
score_changed = 1;
}
}
```

## 3. 主循环

### 3.1 主循环之前

#### 3.1.1 `cull_queue` 函数

在前面讨论的关于case的 `top_rated` 的计算中，还有一个机制是检查所有的 `top_rated[]` 条目，然后顺序获取之前没有遇到过的byte的对比分数低的“获胜者”进行标记，标记至少会维持到下一次运行之前。在所有的fuzz步骤中，“favorable”的条目会获得更多的执行时间。

函数的控制流程图如下：

![image-20210909141024743](upload/attach/202109/779730_39HN95B9JY9HV24.jpg)

结合源码解析如下：

```c
static void cull_queue(void) {
struct queue_entry* q;
static u8 temp_v[MAP_SIZE >> 3];
u32 i;
if (dumb_mode || !score_changed) return; // 如果处于dumb模式或者score没有发生变化（top_rated没有发生变化），直接返回
score_changed = 0;
memset(temp_v, 255, MAP_SIZE >> 3);
// 设置temp_v大小为MAP_SIZE>>3，初始化为0xff，全1，表示还没有被覆盖到，为0表示被覆盖到了。
queued_favored = 0;
pending_favored = 0;
q = queue;
while (q) { // 进行队列遍历
q->favored = 0; // 所有元素的favored均设置为0
q = q->next;
}
/* Let's see if anything in the bitmap isn't captured in temp_v.
If yes, and if it has a top_rated[] contender, let's use it. */
// i从0到MAP_SIZE进行迭代，筛选出一组队列条目，它们可以覆盖到所有现在已经覆盖的路径
for (i = 0; i < MAP_SIZE; i++)
if (top_rated[i] && (temp_v[i >> 3] & (1 << (i & 7)))) {
u32 j = MAP_SIZE >> 3;
/* Remove all bits belonging to the current entry from temp_v. */
// 从temp_v中，移除所有属于当前current-entry的byte，也就是这个testcase触发了多少path就给tempv标记上
while (j--)
if (top_rated[i]->trace_mini[j])
temp_v[j] &= ~top_rated[i]->trace_mini[j];
top_rated[i]->favored = 1;
queued_favored++;
if (!top_rated[i]->was_fuzzed) pending_favored++;
}
q = queue;
while (q) { // 遍历队列，不是favored的case（冗余的测试用例）被标记成redundant_edges
mark_as_redundant(q, !q->favored); // 位置在/queue/.state/redundent_edges中
q = q->next;
}
}
```

这里根据网上公开的一个例子来理解该过程：

现假设有如下tuple和seed信息：

- **`tuple`**: `t0`, `t1`, `t2`, `t3`, `t4`

- **`seed`**: `s0`, `s1`, `s2`

- 初始化 `temp_v = [1,1,1,1,1]`

- `s1` 可覆盖 `t2`, `t3`，`s2` 覆盖 `t0`, `t1`, `t4`，并且 `top_rated[0] = s2`，`top_rated[2]=s1`

将按照如下过程进行筛选和判断：

1. 首先判断 `temp_v[0]=1`，说明 `t0` 没有被覆盖；

2. `top_rated[0]` 存在 (`s2`) -> 判断 `s2` 可以覆盖的范围 -> `trace_mini = [1,1,0,0,1]`；

3. 更新 `temp_v=[0,0,1,1,0]`， 标记 `s2` 为 "favored"；

4. 继续判断 `temp_v[1]=0`，说明 `t1` 此时已经被覆盖，跳过；

5. 继续判断 `temp_v[2]=1`，说明 `t2` 没有被覆盖；

6. `top_rated[2]` 存在 (`s1`) -> 判断 `s1` 可以覆盖的范围 -> `trace_mini=[0,0,1,1,0]`；

7. 更新 `temp_v=[0,0,0,0,0]`，标记 `s1` 为 "favored"；
