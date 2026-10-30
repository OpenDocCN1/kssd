
### AFL persistent mode handler

```c
/* A simplified persistent mode handler, used as explained in README.llvm. */
int __afl_persistent_loop(unsigned int max_cnt) {
  static u8  first_pass = 1;
  static u32 cycle_cnt;
  if (first_pass) {
    if (is_persistent) {
      memset(__afl_area_ptr, 0, MAP_SIZE);
      __afl_area_ptr[0] = 1;
      __afl_prev_loc = 0;
    }
    cycle_cnt = max_cnt;
    first_pass = 0;
    return 1;
  }
  if (is_persistent) {
    if (--cycle_cnt) {
      raise(SIGSTOP);
      __afl_area_ptr[0] = 1;
      __afl_prev_loc = 0;
      return 1;
    } else {
      __afl_area_ptr = __afl_area_initial;
    }
  }
  return 0;
}
```

- 如果是第一次执行那么 `first_pass = 1`
  - 如果处于 `persistent` mode 下。
    - 清空 `__afl_area_ptr`。
    - 然后令 `__afl_area_ptr[0] = 1`
    - 令 `__afl_prev_loc = 0`
  - 设置循环次数 `cycle_cnt` 为 `max_cnt`
  - 设置 `first_pass = 0` 初次循环已经结束。`return 1`.
- 如果在 `persistent` mode 下，且 `--cycle_cnt` 大于 `1`。
  - 发出信号 `SIGSTOP` 暂停当前进程。
  - 设置 `__afl_area_ptr[0] = 1` 与 `__afl_prev_loc = 0`
  - `return 1`
- 如果在 `persistent` mode 下，且 `--cycle_cnt` 为 `0`
  - 让 `__afl_area_ptr` 指向 `__afl_area_initial`
- 最后 `return 0`

整体过程大致如下：

当第一次运行到 `AFL_LOOP` 时，进行初始化然后 `return 1`，此时满足 `while (AFL_LOOP(1000))`，于是执行一次 `fuzz`。

当我们再次进入 `fuzz` loop 时，计数减 `1`，触发：`raise(SIGSTOP)` 暂停进程，而 `forkserver` 收到了此时的暂停信号，设置 `child_stopped = 1`，通知 `afl-fuzzer`。

当 `afl-fuzzer` 再进行一次 `fuzz` 时，恢复之前的子进程继续执行，并设置 `child_stopped` 为 `0`。

此时相当于重新执行了一遍程序，重新对 `__afl_prev_loc` 设置，随后直接 `return 1`，此时又进入 `while (AFL_LOOP(1000))` 执行一次，接下来下一次触发 `raise(SIGSTOP)` 暂停进程。

以上的步骤重复执行，直到第 `1000` 次，`cnt` 被减到 `0`，不会再暂停子进程，而是令 `__afl_area_ptr` 指向无关的 `__afl_area_initial`，随后子进程结束。指向一个无关值主要是因为程序仍然会进行插桩，导致向 `__afl_area_ptr` 中写值。我们选择向一个无关的位置写值而不影响到共享内存等。

## trace-pc-guard mode

`954K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3q4F1k6#2)9J5k6h3I4D9N6X3#2Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4K6i4K6u0r3f1$3q4F1K9i4c8A6P5X3g2J5b7$3!0$3k6i4u0S2k6$3g2Q4x3X3g2Z5N6r3#2D9i4K6t1K6N6s2u0S2j5$3W2F1k6#2)9J5k6s2m8U0M7#2)9J5k6s2N6A6N6r3S2Q4x3X3c8Y4N6h3q4J5k6s2x3`.

如果想使用这个功能需要设置：

|  |  |
| --- | --- |
| 1 | `AFL_TRACE_PC=1` |

传入：`With -fsanitize-coverage=trace-pc-guard`

*the compiler will insert the following code on every edge*

然后重新编译。

```c
/* The following stuff deals with supporting -fsanitize-coverage=trace-pc-guard.
It remains non-operational in the traditional, plugin-backed LLVM mode.
For more info about 'trace-pc-guard', see README.llvm.
The first function (__sanitizer_cov_trace_pc_guard) is called back on every
edge (as opposed to every basic block). */
void __sanitizer_cov_trace_pc_guard(uint32_t* guard) {
  __afl_area_ptr[*guard]++;
}
```

此函数 `__sanitizer_cov_trace_pc_guard` 将在每个 `basic block` `edge`（边界）被调用，其实就是通过（`*guard`）索引到共享内存对应的计数位置，然后计数加一。

而 `guard` 的初始化如下：

```c
/* Init callback. Populates instrumentation IDs. Note that we're using
ID of 0 as a special value to indicate non-instrumented bits. That may
still touch the bitmap, but in a fairly harmless way. */
void __sanitizer_cov_trace_pc_guard_init(uint32_t* start, uint32_t* stop) {
  u32 inst_ratio = 100;
  u8* x;

  if (start == stop || *start) return;

  x = getenv("AFL_INST_RATIO");
  if (x) inst_ratio = atoi(x);

  if (!inst_ratio || inst_ratio > 100) {
    fprintf(stderr, "[-] ERROR: Invalid AFL_INST_RATIO (must be 1-100).\n");
    abort();
  }

  /* Make sure that the first element in the range is always set - we use that
  to avoid duplicate calls (which can happen as an artifact of the underlying
  implementation in LLVM). */
  *(start++) = R(MAP_SIZE - 1) + 1;

  while (start < stop) {
    if (R(100) < inst_ratio) *start = R(MAP_SIZE - 1) + 1;
    else *start = 0;
    start++;
  }
}
```

首先获取了插桩密度。

然后从第一个 `guard` 遍历。`llvm` 设置 `guard` 的收尾为 `start` 与 `stop`。并设置 `guard` 指向的值。

注意有一定的概率不进行插桩（随机插桩）。

特别的，若此时的 `basic block` 因为概率选择的原因没有进行插桩，那么设置这里的 `guard` 指向的值为 `0`，这里的 `0` 是作为一个特殊值，代表不进行插桩。

```c
*(start++) = R(MAP_SIZE - 1) + 1;
while (start < stop) {
  if (R(100) < inst_ratio) *start = R(MAP_SIZE - 1) + 1;
  else *start = 0;
  start++;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-2-14 15:13 被 Roland_ 编辑，原因：

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)
