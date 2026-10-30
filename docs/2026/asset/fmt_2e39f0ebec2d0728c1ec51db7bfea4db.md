
- 首先扫描`dir`中是否有`@`，有的话替换成空字节。提取下一字节转成`int`赋值到`dict_level`

- 尝试`d = opendir(dir)`打开对应`dir`。

  - 打开失败且 `errno == ENOTDIR`（不是一个目录文件）。调用 `load_extras_file(dir, &min_len, &max_len, dict_level);`然后跳到 `check_and_sort`。对这个`file`进行检查和排序（by size）

  - 打开失败且`errno != ENOTDIR` 直接`abort`。

- 接下来一个大`While`循环扫描并读取目录下的文件到 `extras[]` 中。

- 最后在 `check_and_sort:` 中对 `extras[]`（extra token） 进行检测与排序

### find_timeout(void)

本函数的作用是当恢复本地会话时没有使用`-t`进行设置时防止不停的调整超时时间。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `static u8 tmp[4096];` `/* Ought to be enough for anybody. */` `u8 *fn, *off;` `s32 fd, i;` `u32 ret;` |

- 如果没有设置`resuming_fuzz`，直接`return`。

- 如果设置了`in_place_resume`，`fn = alloc_printf("%s/fuzzer_stats", out_dir)` 拼接路径

  - 否则`fn = alloc_printf("%s/../fuzzer_stats", in_dir)` 拼接路径。

- 打开`fn`，返回`fd`。

- 从`fd`读取`0xfff`字节到缓冲区`tmp`

- 匹配`tmp`中是否有子串 `exec_timeout :`

  - 如果没有则`return`

  - 如果有 `ret = atoi(off + 20)`，如果`ret > 4`那么设置 `exec_tmout = ret` 并且 `timeout_given = 3`。否则`return`，不进行设置。

### detect_file_args(char **argv)

本函数负责扫描每一个`argv[i]`是否有`@@` 子串，如果有就进行替换。

|  |  |
| --- | --- |
| 1  2 | `u32 i = 0;` `u8 *cwd = getcwd(NULL, 0);` |

- 首先获取当前路径`cwd`，然后进入一个大 `while (argv[i])` 扫描`argv`，查找是否有`"@@"`子串

  - 如果找到了 `"@@"` 子串。位置存在`aa_loc`中。

    - 若没设置`out_file`：拼接产生`out_file`：`out_file = alloc_printf("%s/.cur_input", out_dir)`

    - 若果 `out_file[0] == '/'` 那么直接令：`aa_subst = out_file`。否则拼接产生`out_file` `aa_subst = alloc_printf("%s/%s", cwd, out_file)`

    - 接下来将当前位置（`*aa_loc = 0`）的 '@' 换成空字节。然后跳过`aa_loc`两个字节，拼接产生`n_arg`。

      `n_arg = alloc_printf("%s%s%s", argv[i], aa_subst, aa_loc + 2)`

    - 然后将`argv[i]`这里的参数换成`n_arg`。

    - 最后重新将当前位置换成 '@'，（`*aa_loc = '@'`）

### perform_dry_run(use_argv)

对所有测试用例执行试运行，以确认该应用程序按照预期正常运行。仅对初始输入执行此操作，并且仅执行一次。

|  |  |
| --- | --- |
| 1  2  3 | `struct queue_entry *q = queue;` `u32 cal_failures = 0;` `u8 *skip_crashes = getenv("AFL_SKIP_CRASHES");` |

- 进入一个大循环 `while (q)` 扫描输入队列的每一项。

  - 取文件名放入`fn`。如："`id:000000,orig:a.out`"

  - 打开文件返回`fd`。接着以`q->len`为大小分配空间，返回`use_mem`

  - 将此文件的内容读入`use_mem`中。

  - `close(fd)`

  - 调用`calibrate_case(argv, q, use_mem, 0, 1)` 进行测试用例的校准。返回值为`res`

  - 释放`use_mem`

  - 如果设置了`stop_soon`那就立刻停止

  - 返回值为 `crash_mode` 或者 `FAULT_NOBIT`时

  - 打印提示信息后进入一个大`switch`。`switch (res)`

    - `case FAULT_NONE:` 

      - 如果`q`此时是`queue`头，那么调用`check_map_coverage()`检测`map`覆盖率。

      - 接下来如果设置了`crash_mode`，那么输出提示信息后`exit`。

    - `case FAULT_TMOUT:` 

      - 如果设置了`timeout_given`

        - `timeout_given > 1`：设置`queue`当前项 `q->cal_failed = CAL_CHANCES`，然后 `cal_failures++`

        - 否则输出信息后`abort`（处理初始测试用例超时）。

      - 如果未设置则直接`abort`。

    - `case FAULT_CRASH:` 

      - 如果设置了`crash_mode`，那么`break`。

      - 如果设置了`skip_crashes`，那么 `q->cal_failed = CAL_CHANCES` 然后 `cal_failures++` 之后直接`break`。

      - 如果设置了`mem_limit`，输出提示信息（内存不够）后`abort`

      - 否则直接`abort`

    - `case FAULT_ERROR:` 

    - `case FAULT_NOINST:` 

    - `case FAULT_NOBITS:` 

      - `useless_at_start++`

      - 如果 `!in_bitmap && !shuffle_queue` 都未设置，提示用户用例可能无效后`break`

  - 如果设置了 `q->var_behavior` 输出提示信息：*Instrumentation output varies across runs.*

  - `q`顺着队列后移一个元素。

- 如果设置了`cal_failures`

  - `cal_failures == queued_paths`：所有用例均超时。

  - 否则告诉用户我们跳过了 `cal_failures` 由于超时

  - 计算`cal_failures * 5` 是否大于 `queued_paths`

    - 如果大于，则说明测试用例的问题比例太高，可能需要重新检查设置。

- 结束。

### calibrate_case(char * *argv, struct queue_entry* q, u8 *use_mem,u32 handicap, u8 from_queue)

本函数对我们所有的输入文件做了评估并将`testcase`多次运行，并且当发现有新的`bits`/路径产生时评估此文件是否是`variable`（通过多次运行看是否产生路径变化？）

测试用例校准 `calibrate_case(argv, q, use_mem, 0, 1)` `use_mem`为测试用例的内容（可执行文件）

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `static u8 first_trace[MAP_SIZE];` `u8 fault = 0, new_bits = 0, var_detected = 0, hnb = 0, first_run = (q->exec_cksum == 0);` `u64 start_us, stop_us;` `s32 old_sc = stage_cur, old_sm = stage_max;` `u32 use_tmout = exec_tmout;` `u8 *old_sn = stage_name;` |

- 如果没有设置`from_queue`，或者设置了`resuming_fuzz`（恢复会话或尝试校准已添加的发现时），那么设置：

  `use_tmout = MAX(exec_tmout + CAL_TMOUT_ADD,exec_tmout * CAL_TMOUT_PERC / 100);`其实就是对于此时`timeout`进行适当的宽松。

- `q->cal_failed++`

- 设置此时的状态信息 `stage_name = "calibration";stage_max = fast_cal ? 3 : CAL_CYCLES;`

- 判断此时的`forkserver`是否已经启动。

  - 若未启动则调用`init_forkserver(argv)`

- 如果设置了`q->exec_cksum`。

  - 首先拷贝 `memcpy(first_trace, trace_bits, MAP_SIZE);`

  - 接着调用`hnb = has_new_bits(virgin_bits)`，查看是否有新的`tuple`产生还是仅有`hit-count`改变。

  - 如果返回的`hnb > new_bits`（一开始是0），那么更新`new_bits`。`new_bits = hnb;`

- 获取当前时间：`start_us = get_cur_time_us()`

- 接着进入一个大`for`循环中：`for (stage_cur = 0; stage_cur < stage_max; stage_cur++)`

  - 如果不是`first_run` 并且 `stage_cur % stats_update_freq == 0`也就是经过了一个周期，那么调用`show_stats()`输出`fuzz`信息。

  - 调用`write_to_testcase(use_mem, q->len)`将读取的内容写入我们的`outfile`中。

  - 接下来调用`run_target(argv, use_tmout)`运行文件，返回结果存储在`fault`中。

  - 如果设置了`stop_soon`或者`fault != crash_mode`。

    `goto abort_calibration:`

  - 如果非`dumb_mode`，并且`stage_cur==0`（第一次运行）；并且调用`count_bytes(trace_bits)`查找`trace_bit`中是否有被设置为1的`bit`，若没有，则设置 `fault = FAULT_NOINST` 最后跳转到`abort_calibration`

  - 接着根据当前的`trace_bits`计算`cksum`。（`checksum`？）

  - 如果当前的`q->exec_cksum != cksum`。

    - 重新调用`has_new_bits(virgin_bits)`，返回`hnb`若大于`new_bits`，更新`new_bits = hnb`。

    - 如果`q->exec_cksum`不为零（不是第一次执行这个`queue entry`）

      - `i`为索引，扫描`var_bytes`、`first_trace`、`trace_bits`。

      - 当满足`var_bytes[i]==0`，`first_trace[i] != trace_bits[i]`时，代表发现了可变的`entry`，设置对应的`var_bytes[i] = 1`，令`stage_max = CAL_CYCLES_LONG`。

      - 最后设置`var_detected = 1`。

    - 如果`q->exec_cksum`为零（第一次执行这一队列项）

      - 更新`q->exec_cksum = cksum`（`cksum`为之前通过`trace_bits`计算的）

      - 将`trace_bits`用`memcpy`拷贝到`first_trace`。

- 获取当前时间，计算总时间和轮数。

- 计算当前这一`queue entry`的`q->exec_us`。

- `q->bitmap_size = count_bytes(trace_bits)`计算执行完毕后`trace_bits`中的路径信息。

- `q->handicap = handicap; q->cal_failed = 0;`

- 累加`q->bitmap_size;`到`total_bitmap_size`，即累加了当前发生变化的`bits`的个数。

- `total_bitmap_entries++;`

- 调用`update_bitmap_score(q);`更新一些比如偏好因子的信息，包括进行对应的`trace_bits`压缩，以判断此判断此路径是否是更有利的。

- 如果当前的`case`没有产生一个新的结果，那么通知用户这是一个不重要的问题。

- 满足`!dumb_mode && first_run && !fault && !new_bits`（第一次运行，`fault`未设置，`newbits`未设置）设置`fault = FAULT_NOBITS;`

- 接下来是 `abort_calibration:`

  - 如果`new_bits == 2`（产生了新路径）且`q->has_new_cov`未设置。

    - 设置`q->has_new_cov = 1;`

    - `queued_with_cov++;`计数++

  - 如果设置了`var_detected`

    - 调用`count_bytes(var_bytes);`获取变化的`bytes`的数量存在`var_byte_count`。

    - 如果`q->var_behavior`未设置。

      - 调用`mark_as_variable(q)`，然后`queued_variable++`计数增加。

        实际此时是将此`entry`标记为可变。在`mark_as_variable(q)`中创建了符号链接`/queue/.state/variable_behavior/q->fname`

  - 接下来恢复之前的`stage`相关数据

  - 如果不是第一轮，那么输出相关信息。

  - `return fault`

### init_forkserver(char **argv)

[Linux 的进程间通信：管道](https://bbs.kanxue.com/elink@0f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0f1^5y4o6R3&6z5o6M7K6)

本函数用来初始化/启动`forkserver`。首先有如下定义：

|  |  |
| --- | --- |
| 1  2  3  4 | `static struct itimerval it;` `int st_pipe[2], ctl_pipe[2];` `int status;` `s32 rlen;` |

- 首先创建两个管道：`pipe(st_pipe) || pipe(ctl_pipe)`，其中`st_pipe`：状态管道。`ctl_pipe`：控制管道

- 然后产生一个`forkserver`：`forksrv_pid = fork()`

- 在子进程中（`forkserver`）

  - 首先进行 `setsid()`创建新的会话，使子进程完全独立，脱离控制。

  - 接下来重新配置`fd`。关闭子进程中的`stdout`和`stderr`，然后将其重定向到`/dev/null`中。（相当于关闭了子进程的全部输出）

  - 如果设置了`out_file`，那么用`dup2`再关掉子进程的`stdin`，重定向到`/dev/null`

  - 否则关闭`stdin`，重定向到`out_fd`。最后关闭`out_fd`

  - 将`FORKSRV_FD`重定向到`ctl_pipe[0]`。将`FORKSRV_FD + 1`重定向到`st_pipe[1]`。

    - 此时，**子进程只能从控制管道中读命令；向状态管道中写。**

  - 接下来关闭子进程中一些描述符。

    |  |  |
    | --- | --- |
    | 1  2  3  4  5  6  7  8  9 | `close(ctl_pipe[0]);` `close(ctl_pipe[1]);` `close(st_pipe[0]);` `close(st_pipe[1]);` `close(out_dir_fd);` `close(dev_null_fd);` `close(dev_urandom_fd);` `close(fileno(plot_file));` |

  - 如果没有设置`LD_BIND_LAZY`，那么设置 `setenv("LD_BIND_NOW", "1", 0)` 防止`linker`在`fork`之后做额外的工作。

  - 设置`asan`与`msan`选项。

  - **最后启动`execv(target_path, argv);`**
