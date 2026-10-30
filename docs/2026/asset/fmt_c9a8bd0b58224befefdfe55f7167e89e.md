
```c
use_tmout = MAX(exec_tmout + CAL_TMOUT_ADD, exec_tmout * CAL_TMOUT_PERC / 100);
```

其实就是对于此时 timeout 进行适当的宽松。

- `q->cal_failed++`
- 设置此时的状态信息：
  ```c
  stage_name = "calibration";
  stage_max = fast_cal ? 3 : CAL_CYCLES;
  ```
- 判断此时的 `forkserver` 是否已经启动。
- 若未启动，则调用 `init_forkserver(argv)`。
- 如果设置了 `q->exec_cksum`：
  - 首先拷贝 `memcpy(first_trace, trace_bits, MAP_SIZE)`。
  - 接着调用 `hnb = has_new_bits(virgin_bits)`，查看是否有新的 tuple 产生，还是仅有 hit-count 改变。
- 如果返回的 `hnb > new_bits`（一开始是 0），那么更新 `new_bits`：
  ```c
  new_bits = hnb;
  ```
- 获取当前时间：
  ```c
  start_us = get_cur_time_us();
  ```
- 接着进入一个大 `for` 循环中：
  ```c
  for (stage_cur = 0; stage_cur < stage_max; stage_cur++)
  ```
- 如果不是 `first_run`，并且 `stage_cur % stats_update_freq == 0`，也就是经过了一个周期，那么调用 `show_stats()` 输出 fuzz 信息。
- 调用 `write_to_testcase(use_mem, q->len)`，将读取的内容写入我们的 outfile 中。
- 接下来调用 `run_target(argv, use_tmout)` 运行文件，返回结果存储在 `fault` 中。
- 如果设置了 `stop_soon` 或者 `fault != crash_mode`：
  ```c
  goto abort_calibration;
  ```
- 如果非 `dumb_mode`，并且 `stage_cur == 0`（第一次运行），调用 `count_bytes(trace_bits)` 查找 `trace_bit` 中是否有被设置为 1 的 bit。若没有，则设置：
  ```c
  fault = FAULT_NOINST;
  ```
  最后跳转到 `abort_calibration`。
- 接着根据当前的 `trace_bits` 计算 cksum。
- 如果当前的 `q->exec_cksum != cksum`：
  - 重新调用 `has_new_bits(virgin_bits)`，返回 `hnb`。若大于 `new_bits`，则更新 `new_bits = hnb`。
  - 如果 `q->exec_cksum` 不为零（不是第一次执行这个 queue entry）：
    - `i` 为索引，扫描 `var_bytes`、`first_trace`、`trace_bits`。
    - 当满足 `var_bytes[i] == 0`、`first_trace[i] != trace_bits[i]` 时，代表发现了可变的 entry，设置对应的 `var_bytes[i] = 1`，令 `stage_max = CAL_CYCLES_LONG`。
  - 最后设置 `var_detected = 1`。
- 如果 `q->exec_cksum` 为零（第一次执行这一队列项）：
  - 更新：
    ```c
    q->exec_cksum = cksum;
    ```
    `cksum` 为之前通过 `trace_bits` 计算的值。
  - 将 `trace_bits` 用 `memcpy` 拷贝到 `first_trace`。
- 获取当前时间，计算总时间和轮数。
- 计算当前这一 queue entry 的 `q->exec_us`。
- `q->bitmap_size = count_bytes(trace_bits)`，计算执行完毕后 `trace_bits` 中的路径信息。
- 设置：
  ```c
  q->handicap = handicap;
  q->cal_failed = 0;
  ```
- 累加 `q->bitmap_size` 到 `total_bitmap_size`，即累加当前发生变化的 bits 的个数。
- `total_bitmap_entries++`。
- 调用 `update_bitmap_score(q)`，更新一些比如偏好因子的信息，包括进行对应的 `trace_bits` 压缩，以判断此路径是否是更有利的。
- 如果当前的 case 没有产生一个新的结果，那么通知用户这是一个不重要的问题。
- 满足 `!dumb_mode && first_run && !fault && !new_bits`（第一次运行，fault 未设置，newbits 未设置）时，设置：
  ```c
  fault = FAULT_NOBITS;
  ```
- 接下来是：
  ```c
  abort_calibration:
  ```
- 如果 `new_bits == 2`（产生了新路径）且 `q->has_new_cov` 未设置：
  - 设置：
    ```c
    q->has_new_cov = 1;
    ```
  - `queued_with_cov++`，计数增加。
- 如果设置了 `var_detected`：
  - 调用 `count_bytes(var_bytes)`，获取变化的 bytes 数量，存储在 `var_byte_count` 中。
  - 如果 `q->var_behavior` 未设置：
    - 调用 `mark_as_variable(q)`，然后 `queued_variable++`，计数增加。
    - 实际此时是将此 entry 标记为可变。在 `mark_as_variable(q)` 中创建了符号链接：
      ```text
      /queue/.state/variable_behavior/q->fname
      ```
- 接下来恢复之前的 stage 相关数据。
- 如果不是第一轮，那么输出相关信息。
- `return fault`

## `init_forkserver(char **argv)`

[Linux 的进程间通信：管道](https://zhuanlan.zhihu.com/p/58489873)

本函数用来初始化/启动 forkserver。首先有如下定义：

```c
static struct itimerval it;
int st_pipe[2], ctl_pipe[2];
int status;
s32 rlen;
```

- 首先创建两个管道：
  ```c
  pipe(st_pipe) || pipe(ctl_pipe)
  ```
  其中，`st_pipe` 是状态管道，`ctl_pipe` 是控制管道。
- 然后产生一个 forkserver：
  ```c
  forksrv_pid = fork()
  ```
- 在子进程中（forkserver）：
  - 首先进行 `setsid()`，创建新的会话，使子进程完全独立，脱离控制。
  - 接下来重新配置 fd。关闭子进程中的 stdout 和 stderr，然后将其重定向到 `/dev/null` 中（相当于关闭了子进程的全部输出）。
  - 如果设置了 `out_file`，那么用 `dup2` 再关掉子进程的 stdin，重定向到 `/dev/null`。
  - 否则关闭 stdin，重定向到 `out_fd`。最后关闭 `out_fd`。
  - 将 `FORKSRV_FD` 重定向到 `ctl_pipe[0]`。将 `FORKSRV_FD + 1` 重定向到 `st_pipe[1]`。
  - 此时，**子进程只能从控制管道中读命令，向状态管道中写。**
  - 接下来关闭子进程中一些描述符：

```c
close(ctl_pipe[0]);
close(ctl_pipe[1]);
close(st_pipe[0]);
close(st_pipe[1]);
close(out_dir_fd);
close(dev_null_fd);
close(dev_urandom_fd);
close(fileno(plot_file));
```

- 如果没有设置 `LD_BIND_LAZY`，那么设置：
  ```c
  setenv("LD_BIND_NOW", "1", 0)
  ```
  防止 linker 在 fork 之后做额外的工作。
- 设置 asan 与 msan 选项。
- **最后启动 `execv(target_path, argv)`。**
- 如果 execv 启动失败，那么会运行到：
  ```c
  *(u32 *)trace_bits = EXEC_FAIL_SIG
  ```
  通过在 bitmap 中设置签名来通知父进程执行 execv 失败。

[setsid 的作用](https://blog.csdn.net/sweetfather/article/details/79457261)

[Linux--setsid() 与进程组、会话、守护进程](https://www.cnblogs.com/gx-303841541/p/3360071.html)

[进程间通信管道进阶篇：linux 下 dup/dup2 函数的用法](https://www.cnblogs.com/GODYCA/archive/2013/01/05/2846197.html)

![](upload/attach/202102/876323_RK376B8658AH5MS.jpg)

- 在父进程中关闭 `ctl_pipe[0]` 与 `st_pipe[1]`，即控制管道读与状态管道写（父进程不需要）。
- 然后对于父进程需要的进行如下赋值：
  ```c
  fsrv_ctl_fd = ctl_pipe[1];
  fsrv_st_fd = st_pipe[0];
  ```
- 接下来等待 forkserver 启动（等待很短的一段时间）。
- 然后从 st 中读取 forkserver 的状态信息。
- 如果我们成功读到了 4 bytes 的状态信息，那么说明一切就绪，直接 return。
- 如果设置了 `child_timed_out`，则通知用户调整 `-t` 参数。
- 否则：
  ```c
  waitpid(forksrv_pid, &status, 0)
  ```
  等待 forksrv 返回 status。
- 调用 `WIFSIGNALED(status)`，判断 forksrv 是否返回异常退出的信号 status。

![](upload/attach/202102/876323_ACYKSNX5QKZWNT3.jpg)

- 如果是（异常退出）：
  - 是否设置 `mem_limit && mem_limit < 500 && uses_asan`（`mem_limit` 过小）。告知用户是由于未收到任何 input 就 crash，可能是由于 asan 和 `mem_limit` 的原因。查看 `notes_for_asan.txt`。
  - 如果没有设置 `mem_limit`，告知用户可能的原因。
  - 否则通知用户其他可能的原因（比如触发了 oom 之类的）。
  - 最后直接通过 `WTERMSIG(status)` 获取信号，告知用户 signal 值。
- 如果否（非异常退出）：
  - 检查是否设置了：
    ```c
    trace_bits == EXEC_FAIL_SIG
    ```
    如果设置了，说明 execv 没正常执行。
  - 检查是否设置：
    ```c
    mem_limit && mem_limit < 500 && uses_asan
    ```
    可能是由于配置的内存限制导致的。
  - 如果没有设置 `mem_limit`，可能是 fuzzer 的 bug。:-）
  - 否则告知用户其他可能的原因，比如 oom 之类的。
- 结束。

## `has_new_bits(u8 *virgin_map)`

*Check **if the current execution path brings anything new to the table.** Update virgin bits to reflect the finds. **Returns 1 if the only change is the hit-count for a particular tuple; 2 if there are new tuples seen.** Updates the map, so subsequent calls will always return 0.*

首先有如下定义：

```c
#ifdef WORD_SIZE_64

u64 *current = (u64 *)trace_bits;
u64 *virgin = (u64 *)virgin_map;
u32 i = (MAP_SIZE >> 3);

这里 MAP_SIZE >> 3 代表 MAP_SIZE / 8，也就是按照 8 个字节一组，一共分了 i 组（64 位实现）。

#else

u32 *current = (u32 *)trace_bits;
u32 *virgin = (u32 *)virgin_map;
u32 i = (MAP_SIZE >> 2);

这里 MAP_SIZE >> 3 代表 MAP_SIZE / 4，也就是按照 4 个字节一组，一共分了 i 组（32 位实现）。

#endif /* ^WORD_SIZE_64 */
```

- `current` 指向 `trace_bits` 首地址。`virgin` 指向 `virgin_map` 首地址。
- 进入一个大 `while (i--)` 循环分组扫描。
- 如果 `*current` 不为 0（命中了相应路径），且 `*current & *virgin` 不为 0（代表我们命中的某个 bit 之前是没有碰到过的新的）：
  - 如果 `ret < 2`：
    ```c
    u8 *cur = (u8 *)current;
    u8 *vir = (u8 *)virgin;
    ```
    （指向第一个字节，注意这里是 `u8`。）
- 接着进行如下判断：

```c
if ((cur[0] && vir[0] == 0xff) || (cur[1] && vir[1] == 0xff) ||
(cur[2] && vir[2] == 0xff) || (cur[3] && vir[3] == 0xff) ||
(cur[4] && vir[4] == 0xff) || (cur[5] && vir[5] == 0xff) ||
(cur[6] && vir[6] == 0xff) || (cur[7] && vir[7] == 0xff))
```

即查看每组中的每一个字节是否满足 `cur[i] && vir[i] == 0xff`（即检查是否覆盖到了新的路径）。

若有一对儿成立，则 `ret = 2`（new tuple 出现了新路径）；否则 `ret = 1`（only hit-count changed，仅仅是命中次数更新）。

- 进行：
  ```c
  *virgin &= ~*current
  ```
- 这一组处理完了，进行：
  ```c
  current++;
  virgin++;
  ```
  准备处理下一组。
- 最后如果 `ret && virgin_map == virgin_bits`，那么令 `bitmap_changed = 1`。
- 返回 `ret`。

## `write_to_testcase(void *mem, u32 len)`

Write modified data to file for testing. If `out_file` is set, the old file is unlinked and a new one is created. Otherwise, `out_fd` is rewound and truncated.

- 令 `fd = out_fd`。
- 首先检测是否设置 `out_file`。
- 若设置了，打开 outfile，然后把 mem 的内容写到 outfile。
- 若没设置，调整大小：
  ```c
  ftruncate(fd, len)
  ```

<mark class="encrypted">7c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8X3k6@1M7Y4g2F1j5$3q4@1k6b7`.`.</mark>

## `run_target(char **argv, u32 timeout)`

Execute target application, monitoring for timeouts. Return status information. The called program will update `trace_bits[]`.

调用如下：

```c
fault = run_target(argv, use_tmout);
```

- 首先 `memset` 清零 `trace_bits`。
- 接下来如果设置了 `dumb_mode == 1` 或者满足 `no_forkserver`，会执行一段与 `init_forkserver` 中非常相似的代码（没有 forkserver 时，在子进程中执行 targetfile）。
- fork 出一个子进程，在子进程中执行 `target_path`。如果失败，则：
  ```c
  *(u32 *)trace_bits = EXEC_FAIL_SIG
  ```
- 否则代表此时已经有一个 fork server up and running 了（此时代表有 forkserver）。
- 此时我们向控制管道写 4 个字节的 `prev_timed_out`，再从状态管道读取 4 个字节的 `child_pid`。
- 如果读取失败，则说明 forkserver 异常。
- 接下来设置 timer：
  ```c
  setitimer
  ```
