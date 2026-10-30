
- 如果`execv`启动失败，那么会运行到：`*(u32 *) trace_bits = EXEC_FAIL_SIG`。通过在bitmap中设置签名来通知父进程执行`execv`失败。

  参考链接：

  - [setsid的作用](https://bbs.kanxue.com/elink@6e5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4%4k6h3g2@1k6X3q4@1K9r3g2J5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5e0b7#2y4K6t1$3x3b7`.`.)
  - [Linux--setsid() 与进程组、会话、守护进程](https://bbs.kanxue.com/elink@281K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6%4S2Q4x3X3b7K6x3o6x3^5y4o6p5#2y4o6q4Q4x3V1k6H3i4K6u0r3x3K6x3$3x3o6l9%4x3g2)9J5k6h3S2@1L8h3H3`.)
  - [进程间通信管道进阶篇：linux下dup/dup2函数的用法](https://bbs.kanxue.com/elink@208K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3c8@1!0p5h3f1y4m8i4K6u0r3j5i4u0U0K9r3W2$3k6g2)9J5c8U0t1H3x3e0y4Q4x3V1j5H3x3g2)9J5c8U0l9#2i4K6u0r3x3U0R3@1y4U0p5&6y4#2)9J5k6h3S2@1L8h3H3`.)

  ![](https://bbs.kanxue.com/upload/attach/202102/876323_RK376B8658AH5MS.jpg)

- 在父进程中关闭`ctl_pipe[0]`与`st_pipe[1]`，即：控制管道读与状态管道写。（父进程不需要）

- 然后对于父进程需要的进行如下赋值：`fsrv_ctl_fd = ctl_pipe[1]; fsrv_st_fd = st_pipe[0];`

- 接下来等待forkserver启动。（等待很短的一段时间）

- 然后从`st_pipe[0]`读取forkserver的状态信息。

  - 如果我们成功读到了4 bytes的状态信息，那么说明一切就绪，直接`return`。

  - 如果设置了`child_timed_out`，则通知用户调整`-t`参数。

  - 否则`waitpid(forksrv_pid, &status, 0)`等待forksrv返回status。

    - 调用`WIFSIGNALED(status)`判断forksrv是否返回的异常退出的信号status。

      ![](https://bbs.kanxue.com/upload/attach/202102/876323_ACYKSNX5QKZWNT3.jpg)

    - 如果是（异常退出）。

      - 是否设置`mem_limit && mem_limit < 500 && uses_asan`（`mem_limit`过小）。告知用户是由于未收到任何input就crash，可能是由于asan和`mem_limit`的原因。查看`notes_for_asan.txt`。

      - 如果没有设置`mem_limit`，告知用户可能的原因。

      - 否则通知用户其他可能的原因。（比如触发了oom之类的）

      - 最后直接通过`WTERMSIG(status)`获取信号告知用户single值。

    - 如果否（非异常退出）。

      - 检查是否设置了`trace_bits == EXEC_FAIL_SIG`，如果设置了说明execv没正常执行。

      - 检查是否设置`mem_limit && mem_limit < 500 && uses_asan`，可能是由于配置的内存限制导致的。

      - 如果没有设置`mem_limit`，可能是fuzzer的bug。 ：-）

      - 否则告知用户其他可能的原因，比如oom之类的。

- 结束。

### `has_new_bits(u8 *virgin_map)`

Check **if the current execution path brings anything new to the table.**

Update `virgin` bits to reflect the finds. **Returns 1 if the only change is the hit-count for a particular tuple; 2 if there are new tuples seen.**

Updates the map, so subsequent calls will always return 0.

首先有如下定义：

```c
#ifdef WORD_SIZE_64
    u64 *current = (u64 *) trace_bits;
    u64 *virgin = (u64 *) virgin_map;

    u32 i = (MAP_SIZE >> 3);
    // 这里 MAP_SIZE>>3 代表 MAP_SIZE/8 也就是按照8个字节一组，一共分了i组 （64位实现）
#else
    u32 *current = (u32 *) trace_bits;
    u32 *virgin = (u32 *) virgin_map;

    u32 i = (MAP_SIZE >> 2);
    // 这里 MAP_SIZE>>3 代表 MAP_SIZE/4 也就是按照4个字节一组，一共分了i组 （32位实现）
#endif /* ^WORD_SIZE_64 */
```

- `current`指向`trace_bits`首地址，`virgin`指向`virgin_map`首地址。

- 进入一个大`while (i--)`循环分组扫描。

  - 如果`*current`不为0（命中了相应路径），且`*current & *virgin`不为0。（代表我们命中的某个bit之前是没有碰到过的新的）

    - 如果`ret < 2`：

      ```c
      u8 *cur = (u8 *) current;
      u8 *vir = (u8 *) virgin;
      ```

      （指向第一个字节，注意这里是`u8`）

    - 接着进行如下判断：

      ```c
      if ((cur[0] && vir[0] == 0xff) || (cur[1] && vir[1] == 0xff) ||
          (cur[2] && vir[2] == 0xff) || (cur[3] && vir[3] == 0xff) ||
          (cur[4] && vir[4] == 0xff) || (cur[5] && vir[5] == 0xff) ||
          (cur[6] && vir[6] == 0xff) || (cur[7] && vir[7] == 0xff))
      ```

      即查看每组中的每一个字节是否满足`cur[i] && vir[i] == 0xff`。（即检查是否覆盖到了新的路径）

      若有一对儿成立，则`ret = 2`（new tuple出现了新路径）；否则`ret = 1`（only hit-count changed，仅仅是命中次数更新）

    - 进行`*virgin &= ~*current`

  - 这一组处理完了，进行`current++; virgin++;`准备处理下一组

- 最后如果`ret && virgin_map == virgin_bits`，那么令`bitmap_changed = 1`

- 返回`ret`。

### `write_to_testcase(void *mem, u32 len)`

Write modified data to file for testing. If `out_file` is set, the old file is unlinked and a new one is created. Otherwise, `out_fd` is rewound and truncated.

- 令`fd = out_fd`

- 首先检测是否设置`out_file`。

  - 若设置了：打开`outfile`，然后把`mem`的内容写到`outfile`。

  - 若没设置：调整大小，`ftruncate(fd, len)`。

    `878K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8X3k6@1M7Y4g2F1j5$3q4@1k6b7`.`.

### `run_target(char **argv, u32 timeout)`

Execute target application, monitoring for timeouts. Return status information. The called program will update `trace_bits[]`.

调用如下：

```c
fault = run_target(argv, use_tmout);
```

- 首先`memset`清零`trace_bits`。

- 接下来如果设置了`dumb_mode == 1`或者满足`no_forkserver`，会执行一段与`init_forkserver`中非常相似的代码。（没有forkserver时，在子进程中执行`targetfile`）

  - `fork()`出一个子进程，在子进程中执行`target_path`，如果失败，则`*(u32 *) trace_bits = EXEC_FAIL_SIG`。

- 否则代表此时已经有一个fork server up and running了。（此时代表有forkserver）

  - 此时我们向控制管道写4个字节的`prev_timed_out`，再从状态管道读取4个字节的`child_pid`。

    - 如果读取失败则说明forkserver异常。

- 接下来设置timer：`setitimer`。

- 如果满足`dumb_mode == 1 || no_forkserver`（无forkserver，以子进程的方式execve）：

  - `waitpid`等待子进程（`child_pid`）的status。

- 否则直接从forkserver的状态管道中读status。

- 计算执行时间`exec_ms`。

- 重置timer，`total_execs++`；执行次数的计数+1。

- `prev_timed_out = child_timed_out;`

- 判断子进程是否是异常退出，如果是，判断异常退出的原因后`return`。

- 判断退出状态是否是`uses_asan && WEXITSTATUS(status) == MSAN_ERROR`，如果是，设置`kill_signal = 0;`后`return FAULT_CRASH`。

- 接下来判断是否是无forkserver情况下`execve`执行失败。

- 如果最慢执行时间小于当前执行时间，并且`timeout < exec_tmout`，则更新`slowest_exec_ms = exec_ms;`。

- `return FAULT_NONE`

结束。

### `classify_counts(u64 *mem)`

调用：`classify_counts((u64 *) trace_bits)`

源码：

```c
static inline void classify_counts(u64 *mem) {
    u32 i = MAP_SIZE >> 3; // 八个一组，一共i组
    while (i--) { // 每一组扫描
        /* Optimize for sparse bitmaps. */
        if (unlikely(*mem)) { // 如果对应的mem中的值不为0
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
#define FF(_b)  (0xff << ((_b) << 3))

/* Count the number of bytes set in the bitmap. Called fairly sporadically,
   mostly to update the status screen or calibrate and examine confirmed
   new paths. */

static u32 count_bytes(u8 *mem) {
    u32 *ptr = (u32 *) mem; // ptr指向trace bits的首位
    u32 i = (MAP_SIZE >> 2); // 四个字节一组
    u32 ret = 0; // ret计数初始化为0

    while (i--) { // 一组一组扫描
        u32 v = *(ptr++); // v每次取4个字节
        if (!v) continue; // 如果4个字节全部是0的话直接跳过这四个字节取下一组。
        if (v & FF(0)) ret++; // 0xff
        if (v & FF(1)) ret++; // 0xff00
        if (v & FF(2)) ret++; // 0xff0000
        if (v & FF(3)) ret++; // 0xff000000
    }

    return ret;
}
```

其实就是四个一组，扫描哪个字节不为0。若扫描到不为0的就`ret++`；最后返回`ret`。

### `update_bitmap_score(struct queue_entry *q)`

当我们碰到一个新路径时，判断此路径是否是更有利的。即是否是能遍历到bitmap中的bit的最小的路径集合。

```c
u32 i;
u64 fav_factor = q->exec_us * q->len;
```

- 首先在开始时定义了一个偏好因子`fav_factor = q->exec_us * q->len;`

- 接着进入一个for循环：`for (i = 0; i < MAP_SIZE; i++)`

  - 扫描`trace_bits`中每一位`trace_bits[i]`

    - 如果`trace_bits[i] = 1`（代表这是已经被覆盖到的路径）

      - 若设置了`top_rated[i]`

        - 判断当前的`fav_factor`是否大于`top_rated[i]`列表中维护的因子的值。

          - 如果大于则说明`top_rated[i]`中存的是最优的`fav_factor`，则直接`continue`掉此次循环。
