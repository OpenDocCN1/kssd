
| 1 | `-` ````如果设置了```no_arith```` |

- goto `skip_arith`

## ARITHMETIC INC/DEC 阶段
本阶段主要做加减变异

|  |  |
| --- | --- |
| 1 | `#define ARITH_MAX 35` |

- `"arith 8/8"`以byte为单元变异阶段

  - 首先扫描`out_buf`。
    `u8 orig = out_buf[i]`（此时一个`orig`是一个字节，此阶段是**按字节**扫描）
  - 如果对应的`eff_map`中的项为0，则`stage_max`减去2倍的`ARITH_MAX`，然后continue跳过此次变异
  - 否则进入一个for循环进行变异->运行

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 | `u8 orig` `=` `out_buf[i];` `.......` `for` `(j` `=` `1``; j <``=` `ARITH_MAX; j``+``+``) {` `/``/``依次扫描orig到orig``+``35` `u8 r` `=` `orig ^(orig` `+` `j);` `/``/``将orig与orig``+``j（j最大为``35``）进行异或翻转` `/``*` `Do arithmetic operations only` `if` `the result couldn't be a product` `of a bitflip.` `*``/` `if` `(!could_be_bitflip(r)) {` `/``/``判断是否为可以通过上一阶段bitfilp得到的（这一步是为了防止相同的冗余变异，节省时间）` `stage_cur_val` `=` `j;` `out_buf[i]` `=` `orig` `+` `j;` `/``/``将out_buf[i]本身加j变异` `if` `(common_fuzz_stuff(argv, out_buf,` `len``)) goto abandon_entry;``/``/``进行fuzz` `stage_cur``+``+``;` `}` `else` `stage_max``-``-``;` `/``/``否则stage_max减``1` `r` `=` `orig ^ (orig` `-` `j);` `/``/``将orig与orig``-``j（j最大为``35``）进行异或翻转` `if` `(!could_be_bitflip(r)) {``/``/``如果判断为可以bitfilp` `stage_cur_val` `=` `-``j;` `out_buf[i]` `=` `orig` `-` `j;``/``/``将out_buf[i]本身减j变异` `if` `(common_fuzz_stuff(argv, out_buf,` `len``)) goto abandon_entry;``/``/``进行fuzz` `stage_cur``+``+``;` `}` `else` `stage_max``-``-``;` `out_buf[i]` `=` `orig;` `}` |
- `"arith 16/8"`阶段

  - 此时以2个字节（word）为单位进行加减变异，并且对于大端序与小端序都进行变异。
- `"arith 32/8"`阶段

  - 以4个字节（32bits）为单位进行加减变异，并且对于大端序与小端序都进行变异。

## INTERESTING VALUES阶段
本阶段主要做替换变异

- 首先是`"interest 8/8"`阶段，以一个字节为单位进行**替换**变异
  - 本阶段首先通过`could_be_bitflip(orig ^ (u8) interesting_8[j])||could_be_arith(orig, (u8) interesting_8[j], 1))`
    保证替换不会由前面的异或和加减变异阶段得到（本质是在防止冗余变换，减小开销）
  - 然后通过`out_buf[i] = interesting_8[j]`进行一个字节的替换。
  - 之后调用`common_fuzz_stuff(argv, out_buf, len)`进行fuzz
- `"interest 16/8"`阶段，以两个字节为单位进行替换变异，并且去除异或、加减、与单字节变异阶段的冗余，同时考虑大小端序。
- `"interest 32/8"`阶段（4字节为单位替换变异）与前面类似。

## DICTIONARY STUFF阶段
本阶段主要基于用户提供的extra来进行一定的变异

- `"user extras (over)"`替换阶段
  - 在满足一定大小的条件下（同时有一定随机性），将用户的extra token以`memcpy`的方式替换/覆写（over）进去，然后进行fuzz

|  |  |
| --- | --- |
| 1 2 | `memcpy(out_buf` `+` `i, extras[j].data, last_len);` `if` `(common_fuzz_stuff(argv, out_buf,` `len``)) goto abandon_entry;` |
  - 最后在进行恢复。
- `"user extras (insert)"`插入阶段

  - 插入（insert）用户的`extras[j]`，然后产生一个新的`ex_tmp`，对于这个`ex_tmp`进行fuzz。
- `"auto extras (over)"`替换阶段2
- 本阶段类似于over，只不过用于替换的变成了`a_extras[j]`而非`extras[j]`
- 接下来到达标签：`skip_extras:`，如果我们在不跳至`havoc_stage`或`abandon_entry`的情况下来到这里，说明我们已经正确的完成了确定的fuzz（deterministic steps）步骤，我们可以对其进行标记如 .state/ 目录

  - 如果没有设置`queue_cur->passed_det`
    - 调用`mark_as_det_done(queue_cur)`进行标记。

## RANDOM HAVOC（随机毁灭）阶段
本阶段做大范围的随机变异。

- 首先检测如果没有设置`splice_cycle`
  - 那么标记此阶段为`"havoc"`
- 否则标记此阶段为`"splice"`
- 设置`stage_max`
  - 在每一轮stage中首先产生随机数`use_stacking`
  - 根据产生的`use_stacking`做相应次数的变换。相当于每一轮stage中具体的变换由多次小变化叠加产生。
  - 每次变换具体的内容也由一个随机数决定。
    `UR(15 + ((extras_cnt + a_extras_cnt) ? 2 : 0))`
    - 随机替换一个`interesting_8[]`中的byte进来
    - 随机替换`interesting_16[]`中的某个word进来（大小端序随机选择）
    - 随机替换`interesting_32[]`中的某个dword进来（大小端序随机选择）
    - 随机选取`out_buf[]`中某个byte进行减变异（减随机数）
    - 随机选取`out_buf[]`中某个byte进行加变异（加随机数）
    - 随机选取`out_buf[]`中某个word进行减变异（减随机数，大小端序随机选择）
    - 随机选取`out_buf[]`中某个word进行加变异（加随机数，大小端序随机选择）
    - 随机选取`out_buf[]`中某个dword进行减变异（减随机数，大小端序随机选择）
    - 随机选取`out_buf[]`中某个dword进行加变异（加随机数，大小端序随机选择）
    - 随机选取`out_buf[]`中某个byte进行异或翻转变异
    - 随机选取`out_buf[]`中某个byte进行删除
    - 随机选取`out_buf[]`中某个位置插入一段随机长度`clone_to = UR(temp_len)`的内容。这段内容有75%的概率是原来`out_buf[]`中的内容；有25%的概率是一段相同的随机选取的数字。（这串随机选取的数字有50%的几率随机生成，有50%的几率从`out_buf`中选一个字节）
    - 随机选取`out_buf[]`中某个位置覆写一段随机长度的内容。这段内容有75%的概率是原来`out_buf[]`中的内容；有25%的概率是一段相同的随机选取的数字。（这串随机选取的数字有50%的几率随机生成，有50%的几率从`out_buf`中选一个字节）
    - 随机选取一段内容覆写成extra token
      `a_extras[use_extra].data`或者`extras[use_extra].data`
    - 随机选取一段内容插入extra token
      `a_extras[use_extra].data`或者`extras[use_extra].data`
  - 至此，叠加变化结束，调用`common_fuzz_stuff(argv, out_buf, temp_len)`对进行这些随机大变换后的进行fuzz。
  - 如果fuzz后的`queued_paths`与`havoc_queued`不一样了，说明发现了新路径，更新`stage_max`、`perf_score`、`havoc_queued`。

## SPLICING阶段

- 当没有`define IGNORE_FINDS`时。如果我们经过了一整轮什么都没有发现，那么afl会进入`retry_splicing:`这里进一步的对于输入样本进行变换，通过拼接另一个输入样本来完成此变换，最后又跳回`havoc_stage`上一阶段进行大范围的随机变换。
- 否则设置`ret_val = 0`
- 到达`abandon_entry:`
- 设置`splicing_with = -1`
- 对于队列当前项信息更新。
  - 如果未设置`stop_soon`且`queue_cur->cal_failed`为0，`queue_cur->was_fuzzed`未被标记已经fuzz过。
    - 标记`queue_cur->was_fuzzed`为已经fuzz过了
    - `pending_not_fuzzed`计数减1
    - 如果当前对象是favored，那么`pending_favored`计数也减1
- return `ret_val`
- 至此，`fuzz_one`结束。

### sync_fuzzers(char **argv)

本函数用于抓取其他fuzzer的case，读取其他fuzz文件夹下的文件调用，记录下来我们最后一个打开的文件的`qd_ent->d_name`，最后将其写到`out_dir/.synced/sd_ent->d_name`中
- 首先打开目录`sync_dir`
- 然后通过`readdir(sd)`返回一个指向`struct dirent`的指针`sd_ent`。
- 接下来通过一个while循环遍历`sync_dir`目录下的所有文件。（由其他的fuzzer创建的）
  - 首先跳过`.`还有我们自己本fuzzer创建的文件。
  - 接着打开`"sync_dir/sd_ent->d_name/queue"`目录。
  - 检索最后看到的测试用例的ID。
  - 打开`out_dir/.synced/sd_ent->d_name`，返回到`id_fd`。
  - 接着从打开的`id_fd`读取一个`sizeof(u32)`到`min_accept`。
  - 然后`lseek`从新调整文件内指针到开头。
  - 设置当前的`next_min_accept`为我们刚刚读取的`min_accept`。
  - `sync_cnt`计数加一，由`"sync %u"`格式化到`stage_tmp`中。
  - 设置`stage_cur = stage_max = 0`
  - 接下来利用一个while循环对于此fuzzer排队的每个文件，解析ID并查看我们之前是否曾看过它； 如果没有，执行此个测试用例。
  - 我们利用`readdir(qd)`进一步取出目录中的文件。
    `while ((qd_ent = readdir(qd)))`
    - 若文件以'.'开头，或者`syncing_case < min_accept`或者；或者我们使用
      `sscanf(qd_ent->d_name, CASE_PREFIX "%06u", &syncing_case)`失败返回-1。直接跳过此次case的扫描。
    - 如果`syncing_case >= next_min_accept`。
      - 设置`next_min_accept`为`syncing_case + 1`
    - 打开`qd_path/qd_ent->d_name`返回为`fd`。
    - 忽略大小为0和超过大小的文件。
    - 将`fd`对应的文件映射到进程空间中，返回`u8 *mem`
    - 调用`write_to_testcase(mem, st.st_size)`将其写到`outfile`中。
    - 接着`run_target`运行对应文件，返回`fault`。
    - 如果设置了`stop_soon`，直接返回。
    - 将`syncing_party`置为`sd_ent->d_name`
    - 调用`save_if_interesting(argv, mem, st.st_size, fault)`将感兴趣的样本保存。
    - 设置`syncing_party = 0`
    - 调用 `munmap(mem, st.st_size)` 接触映射。
    - 然后 `stage_cur++ % stats_update_freq` 如果是0即循环到一个周期，那么输出对应的fuzz信息。
  - 将`&next_min_accept`对应文件中的内容写到`id_fd`对应的文件中。
  - 关闭对应的文件/目录描述等。

### save_if_interesting(char * *argv, void* mem, u32 len, u8 fault)

本函数用于检测我们在`run_target`中运行的文件返回的结果是否是“有趣的”，进而确定是否要在未来的分析时保存或者插入队列中。若需要返回1，否则返回0.
- 如果`fault`等于`crash_mode`。
  - 查看此时是否出现了`newbits`。
  - 如果没有的话若设置了`crash_mode`，则`total_crashes`计数加一。return 0；
    - 否则直接return 0；
  - 若出现了`newbits`则调用 `fn = alloc_printf("%s/queue/id:%06u,%s", out_dir, queued_paths,describe_op(hnb));`拼接出路径`fn`
  - 通过调用`add_to_queue(fn, len, 0)`将其插入队列。
  - 如果`hnb==2`成立。（有新路径发现）
    - 设置`queue_top->has_new_cov`为1。同时`queued_with_cov`计数加一。
  - 利用`hash32`从新计算`trace_bits`的哈希值，将其设置为`queue_top->exec_cksum`
  - 调用`calibrate_case`进行用例校准，评估当前队列。
  - 打开`fn`，将`mem`的内容写入文件`fn`。
  - 设置`keeping = 1`.
- 接下来通过switch来判断`fault`类型。
  - |  |  |
    | --- | --- |
    | 1 2 3 | `-` ```` 首先```total_tmouts```计数加一。 ```` `-` ```` 如果```nique_hangs > `````=` ```` KEEP_UNIQUE_HANG```那么直接返回keeping ```` `-` ```` 如果不是```dumb_mode ```` |

    - 调用`simplify_trace`对`trace_bits`进行调整。

    - 若没有新的超时路径，直接return keeping。

      `if (!has_new_bits(virgin_tmout))`

      - `unique_tmouts`计数加一。

      - 如果`exec_tmout小于hang_tmout`

    - 将`mem`的内容写到`outfile`。

    - 然后再次调用`run_target`运行一次，返回`new_fault`。

    - 如果未设置`stop_soon`，并且`new_fault`为`FAULT_CRASH`，那么跳转到`keep_as_crash`

    - 如果设置了`stop_soon`，或者`new_fault != FAULT_TMOUT`，直接return keeping。

      - 拼接出路径：

        `fn = alloc_printf("%s/hangs/id:%06llu,%s", out_dir,unique_hangs, describe_op(0));`

      - `unique_hangs`计数加一。

      - 通过`get_cur_time()`获取`last_hang_time`

      - break；

  - |  |  |

    | --- | --- |
