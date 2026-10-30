
- 如果是 `0`，`stage_max` 计数减 `1`，然后 `continue` 跳过。
- 否则进行异或翻转后运行。
- 更新 `new_hit_cnt`、`stage_finds[STAGE_FLIP16]`、`stage_cycles[STAGE_FLIP16]`。
- 如果 `len < 4`，跳转到 `skip_bitflip`。
- 接下来是 `"bitflip 32/8"`，与上述基本相同。

- `skip_bitflip:`
- 如果设置了 `no_arith`，则执行 `goto skip_arith`。

## ARITHMETIC INC/DEC 阶段

本阶段主要做加减变异。

```c
#define ARITH_MAX 35
```

### `"arith 8/8"` 阶段

以 `byte` 为单元进行变异阶段。

- 首先扫描 `out_buf`。

```c
u8 orig = out_buf[i];
```

此时一个 `orig` 是一个字节，此阶段是**按字节**扫描。

- 如果对应的 `eff_map` 中的项为 `0`，则 `stage_max` 减去 `2` 倍的 `ARITH_MAX`，然后 `continue` 跳过此次变异。
- 否则进入一个 `for` 循环进行变异并运行。

```c
u8 orig = out_buf[i];

.......

for (j = 1; j <= ARITH_MAX; j++) { // 依次扫描 orig 到 orig + 35

  u8 r = orig ^ (orig + j); // 将 orig 与 orig + j（j 最大为 35）进行异或翻转

  /*
   * Do arithmetic operations only if the result couldn't be a product
   * of a bitflip.
   */

  if (!could_be_bitflip(r)) { // 判断是否可以通过上一阶段 bitflip 得到，防止相同的冗余变异，节省时间

    stage_cur_val = j;

    out_buf[i] = orig + j; // 将 out_buf[i] 本身加 j 变异

    if (common_fuzz_stuff(argv, out_buf, len))
      goto abandon_entry; // 进行 fuzz

    stage_cur++;

  } else {
    stage_max--; // 否则 stage_max 减 1
  }

  r = orig ^ (orig - j); // 将 orig 与 orig - j（j 最大为 35）进行异或翻转

  if (!could_be_bitflip(r)) { // 如果判断为可以 bitflip

    stage_cur_val = -j;

    out_buf[i] = orig - j; // 将 out_buf[i] 本身减 j 变异

    if (common_fuzz_stuff(argv, out_buf, len))
      goto abandon_entry; // 进行 fuzz

    stage_cur++;

  } else {
    stage_max--;
  }

  out_buf[i] = orig;
}
```

### `"arith 16/8"` 阶段

此时以 2 个字节（`word`）为单位进行加减变异，并且对于大端序与小端序都进行变异。

### `"arith 32/8"` 阶段

以 4 个字节（`32bits`）为单位进行加减变异，并且对于大端序与小端序都进行变异。

## INTERESTING VALUES 阶段

本阶段主要做替换变异。

### `"interest 8/8"` 阶段

以一个字节为单位进行**替换**变异。

- 本阶段首先通过 `could_be_bitflip(orig ^ (u8) interesting_8[j]) || could_be_arith(orig, (u8) interesting_8[j], 1)`，保证替换不会由前面的异或和加减变异阶段得到。本质上是在防止冗余变换，减小开销。
- 然后通过 `out_buf[i] = interesting_8[j]` 进行一个字节的替换。
- 之后调用 `common_fuzz_stuff(argv, out_buf, len)` 进行 fuzz。

### `"interest 16/8"` 阶段

以两个字节为单位进行替换变异，并且去除异或、加减与单字节变异阶段的冗余，同时考虑大小端序。

### `"interest 32/8"` 阶段

以 4 个字节为单位进行替换变异，与前面类似。

## DICTIONARY STUFF 阶段

本阶段主要基于用户提供的 `extra` 来进行一定的变异。

### `"user extras (over)"` 替换阶段

在满足一定大小的条件下（同时有一定随机性），将用户的 `extra token` 以 `memcpy` 的方式替换或覆写（`over`）进去，然后进行 fuzz。

```c
memcpy(out_buf + i, extras[j].data, last_len);
if (common_fuzz_stuff(argv, out_buf, len))
  goto abandon_entry;
```

- 最后进行恢复。

### `"user extras (insert)"` 插入阶段

插入（`insert`）用户的 `extras[j]`，然后产生一个新的 `ex_tmp`，并对这个 `ex_tmp` 进行 fuzz。

### `"auto extras (over)"` 替换阶段 2

本阶段类似于 `over`，只不过用于替换的变成了 `a_extras[j]`，而非 `extras[j]`。

- 接下来到达标签：`skip_extras:`。
- 如果我们在不跳至 `havoc_stage` 或 `abandon_entry` 的情况下来到这里，说明我们已经正确完成了确定的 fuzz（deterministic steps）步骤，可以对其进行标记，如 `.state/` 目录。
- 如果没有设置 `queue_cur->passed_det`，则调用 `mark_as_det_done(queue_cur)` 进行标记。

## RANDOM HAVOC（随机毁灭）阶段

本阶段做大范围的随机变异。

- 首先检测是否设置了 `splice_cycle`。
- 如果没有设置，则标记此阶段为 `"havoc"`。
- 否则标记此阶段为 `"splice"`。
- 设置 `stage_max`。
- 在每一轮 `stage` 中首先产生随机数 `use_stacking`。
- 根据产生的 `use_stacking` 做相应次数的变换。相当于每一轮 `stage` 中，具体的变换由多次小变化叠加产生。
- 每次变换具体的内容也由一个随机数决定：

```c
UR(15 + ((extras_cnt + a_extras_cnt) ? 2 : 0))
```

随机变换包括：

- 随机替换一个 `interesting_8[]` 中的 `byte`。
- 随机替换 `interesting_16[]` 中的某个 `word`，大小端序随机选择。
- 随机替换 `interesting_32[]` 中的某个 `dword`，大小端序随机选择。
- 随机选取 `out_buf[]` 中某个 `byte` 进行减变异，减去随机数。
- 随机选取 `out_buf[]` 中某个 `byte` 进行加变异，加上随机数。
- 随机选取 `out_buf[]` 中某个 `word` 进行减变异，减去随机数，大小端序随机选择。
- 随机选取 `out_buf[]` 中某个 `word` 进行加变异，加上随机数，大小端序随机选择。
- 随机选取 `out_buf[]` 中某个 `dword` 进行减变异，减去随机数，大小端序随机选择。
- 随机选取 `out_buf[]` 中某个 `dword` 进行加变异，加上随机数，大小端序随机选择。
- 随机选取 `out_buf[]` 中某个 `byte` 进行异或翻转变异。
- 随机选取 `out_buf[]` 中某个 `byte` 进行删除。
- 随机选取 `out_buf[]` 中某个位置，插入一段随机长度为 `clone_to = UR(temp_len)` 的内容。这段内容有 75% 的概率是原来 `out_buf[]` 中的内容，25% 的概率是一段相同的随机选取的数字。随机选取的数字有 50% 的几率随机生成，有 50% 的几率从 `out_buf` 中选一个字节。
- 随机选取 `out_buf[]` 中某个位置，覆写一段随机长度的内容。这段内容有 75% 的概率是原来 `out_buf[]` 中的内容，25% 的概率是一段相同的随机选取的数字。随机选取的数字有 50% 的几率随机生成，有 50% 的几率从 `out_buf` 中选一个字节。
- 随机选取一段内容覆写成 `extra token`：

```c
a_extras[use_extra].data
```

或者：

```c
extras[use_extra].data
```

- 随机选取一段内容插入 `extra token`：

```c
a_extras[use_extra].data
```

或者：

```c
extras[use_extra].data
```

至此，叠加变化结束，调用 `common_fuzz_stuff(argv, out_buf, temp_len)` 对进行这些随机大变换后的内容进行 fuzz。

- 如果 fuzz 后的 `queued_paths` 与 `havoc_queued` 不一样了，说明发现了新路径，更新 `stage_max`、`perf_score`、`havoc_queued`。

## SPLICING 阶段

- 当没有 `define IGNORE_FINDS` 时，如果我们经过了一整轮什么都没有发现，那么 AFL 会进入 `retry_splicing:`，进一步对输入样本进行变换，通过拼接另一个输入样本来完成此变换，最后又跳回 `havoc_stage`，进行大范围的随机变换。
- 否则设置 `ret_val = 0`。
- 到达 `abandon_entry:`。
- 设置 `splicing_with = -1`。
- 更新队列当前项信息。
- 如果未设置 `stop_soon`，且 `queue_cur->cal_failed` 为 `0`，`queue_cur->was_fuzzed` 未被标记为已经 fuzz 过，则：
  - 标记 `queue_cur->was_fuzzed` 为已经 fuzz 过。
  - `pending_not_fuzzed` 计数减 `1`。
  - 如果当前对象是 `favored`，则 `pending_favored` 计数也减 `1`。
- `return ret_val`。
- 至此，`fuzz_one` 结束。

### `sync_fuzzers(char **argv)`

本函数用于抓取其他 fuzzer 的 case，读取其他 fuzz 文件夹下的文件并调用，记录下来我们最后一个打开的文件的 `qd_ent->d_name`，最后将其写到 `out_dir/.synced/sd_ent->d_name` 中。

- 首先打开目录 `sync_dir`。
- 然后通过 `readdir(sd)` 返回一个指向 `struct dirent` 的指针 `sd_ent`。
- 接下来通过一个 `while` 循环遍历 `sync_dir` 目录下的所有文件，即由其他 fuzzer 创建的文件。
- 首先跳过 `.` 以及我们自己这个 fuzzer 创建的文件。
- 接着打开 `sync_dir/sd_ent->d_name/queue` 目录。
- 检索最后看到的测试用例 ID。
- 打开 `out_dir/.synced/sd_ent->d_name`，返回到 `id_fd`。
- 接着从打开的 `id_fd` 读取一个 `sizeof(u32)` 到 `min_accept`。
- 然后通过 `lseek` 重新调整文件内指针到开头。
- 设置当前的 `next_min_accept` 为刚刚读取的 `min_accept`。
- `sync_cnt` 计数加 `1`，由 `"sync %u"` 格式化到 `stage_tmp` 中。
- 设置 `stage_cur = stage_max = 0`。
- 接下来利用一个 `while` 循环遍历此 fuzzer 排队的每个文件，解析 ID 并查看我们之前是否曾看过它；如果没有，则执行此测试用例。
- 利用 `readdir(qd)` 进一步取出目录中的文件：

```c
while ((qd_ent = readdir(qd)))
```

- 若文件以 `.` 开头，或者 `syncing_case < min_accept`，或者使用 `sscanf(qd_ent->d_name, CASE_PREFIX "%06u", &syncing_case)` 失败并返回 `-1`，则直接跳过此次 case 的扫描。
- 如果 `syncing_case >= next_min_accept`，则设置 `next_min_accept` 为 `syncing_case + 1`。
- 打开 `qd_path/qd_ent->d_name`，返回为 `fd`。
- 忽略大小为 `0` 和超过大小限制的文件。
- 将 `fd` 对应的文件映射到进程空间中，返回 `u8 *mem`。
- 调用 `write_to_testcase(mem, st.st_size)` 将其写到 outfile 中。
- 接着通过 `run_target` 运行对应文件，返回 `fault`。
- 如果设置了 `stop_soon`，直接返回。
- 将 `syncing_party` 置为 `sd_ent->d_name`。
- 调用 `save_if_interesting(argv, mem, st.st_size, fault)` 保存感兴趣的样本。
- 设置 `syncing_party = 0`。
- 调用 `munmap(mem, st.st_size)` 接触映射。
- 然后执行 `stage_cur++ % stats_update_freq`。如果结果为 `0`，即循环到一个周期，则输出对应的 fuzz 信息。
- 将 `&next_min_accept` 对应文件中的内容写到 `id_fd` 对应的文件中。
- 关闭对应的文件、目录描述符等。

### `save_if_interesting(char **argv, void *mem, u32 len, u8 fault)`

本函数用于检测在 `run_target` 中运行的文件返回的结果是否是“有趣的”，进而确定是否要在未来的分析时保存或者插入队列中。若需要则返回 `1`，否则返回 `0`。

- 如果 `fault` 等于 `crash_mode`：
  - 查看此时是否出现了 `newbits`。
  - 如果没有，且设置了 `crash_mode`，则 `total_crashes` 计数加 `1`，`return 0`。
  - 否则直接 `return 0`。
- 若出现了 `newbits`，则调用以下表达式拼接出路径 `fn`：

```c
fn = alloc_printf("%s/queue/id:%06u,%s", out_dir, queued_paths, describe_op(hnb));
```

- 通过调用 `add_to_queue(fn, len, 0)` 将其插入队列。
- 如果 `hnb == 2` 成立，即发现了新路径：
  - 设置 `queue_top->has_new_cov` 为 `1`。
  - `queued_with_cov` 计数加 `1`。
  - 利用 `hash32` 重新计算 `trace_bits` 的哈希值，将其设置为 `queue_top->exec_cksum`。
- 调用 `calibrate_case` 进行用例校准，评估当前队列。
- 打开 `fn`，将 `mem` 的内容写入文件 `fn`。
- 设置 `keeping = 1`。
- 接下来通过 `switch` 判断 `fault` 类型。

#### `FAULT_TMOUT`

- 首先 `total_tmouts` 计数加 `1`。
- 如果 `nique_hangs >= KEEP_UNIQUE_HANG`，则直接返回 `keeping`。
- 如果不是 `dumb_mode`，调用 `simplify_trace` 对 `trace_bits` 进行调整。
- 若没有新的超时路径，则直接 `return keeping`：

```c
if (!has_new_bits(virgin_tmout))
```

- `unique_tmouts` 计数加 `1`。
- 如果 `exec_tmout` 小于 `hang_tmout`：
  - 将 `mem` 的内容写到 outfile。
  - 然后再次调用 `run_target` 运行一次，返回 `new_fault`。
  - 如果未设置 `stop_soon`，并且 `new_fault` 为 `FAULT_CRASH`，则跳转到 `keep_as_crash`。
- 如果设置了 `stop_soon`，或者 `new_fault != FAULT_TMOUT`，则直接 `return keeping`。
- 拼接出路径：

```c
fn = alloc_printf("%s/hangs/id:%06llu,%s", out_dir, unique_hangs, describe_op(0));
```

- `unique_hangs` 计数加 `1`。
- 通过 `get_cur_time()` 获取 `last_hang_time`。
- `break`。

#### `FAULT_CRASH`

- `keep_as_crash:`。
- `total_crashes` 计数加 `1`。
