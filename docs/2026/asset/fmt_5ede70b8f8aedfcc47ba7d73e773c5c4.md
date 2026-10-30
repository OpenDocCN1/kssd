
- `cycles_wo_finds`计数加一。
- 否则设置`use_splicing`为1。（代表我们要通过splicing进行队列重组。）

- 否则设置`cycles_wo_finds`为0。
- 令`prev_queued`等于`queued_paths`
- 如果设置了`sync_id`并且`queue_cycle == 1`，并且环境变量中设置了`AFL_IMPORT_FIRST`

- 调用`sync_fuzzers(use_argv)`
- 调用`fuzz_one(use_argv)`对于我们的样本进行变换后fuzz，返回`skipped_fuzz`

- 若`skipped_fuzz`为0，并且`stop_soon`为0，并且设置了`sync_id`

- 若`sync_interval_cnt`没有到一个周期（`% SYNC_INTERVAL`）

- 调用`sync_fuzzers(use_argv)`同步其他fuzzer
- 如果没有设置`stop_soon`，且 `exit_1`不为0，那么设置`stop_soon=2`后`break`出fuzz主循环。
- 否则准备fuzz队列中下一个样本。
- 主循环结束后，销毁内存空间，关闭描述符，输出/更新一些状态，至此整个afl的fuzz过程就结束了。

## fuzz主循环中的关键函数

### `fuzz_one(char **argv)`

**从队列中取出当前的一项，然后进行fuzz，返回0如果fuzz成功。返回1如果跳过或者bailed out**

- 如果设置了`pending_favored`

- 查看（当前queue中的这一项是否已经fuzz过了或者不是favored）并且打一个100以内的随机数，如果小于`SKIP_TO_NEW_PROB`（百分之99）
- 直接`return 1`。
- 如果非`dumb_mode`，且当前的不是favored，并且`queued_paths > 10`
- 若`queue_cycle > 1`，并且当前的`queue_cur`还没有fuzz过。
- 打一个100以内的随机数，如果小于`SKIP_NFAV_NEW_PROB`，直接`return 1`。（百分之75）

- 否则打一个100以内的随机数，如果小于`SKIP_NFAV_OLD_PROB`，直接`return 1`。（百分之95）
- 如果不是`tty`模式。
- 输出`current_entry, queued_paths, unique_crashes`提示信息，刷新`stdout`缓冲区

- 将当前的test case映射进入内存。

```
orig_in = in_buf = mmap(0, len, PROT_READ | PROT_WRITE, MAP_PRIVATE, fd, 0)
```

- 分配`len`大小的空间，初始化为0，返回给`out_buf`。
- 设置`subseq_tmouts`为0

- 设置`cur_depth`为`queue_cur->depth`

**CALIBRATION (only if failed earlier on)阶段**

- 如果当前的`queue_cur->cal_failed`不为0（存在校准错误）

- 如果校准错误的次数小于三次。

- 重制`exec_cksum`来告诉`calibrate_case`重新执行testcase来避免对于无效`trace_bits`的使用。
- 设置`queue_cur->exec_cksum`为0。

- 对于`queue_cur`重新执行`calibrate_case`。

```
res = calibrate_case(argv, queue_cur, in_buf, queue_cycle - 1, 0)
```

- 如果返回值为`FAULT_ERROR`，那么直接`abort`

- 如果设置了`stop_soon`，或者`res`不等于`crash_mode`。

- `cur_skipped_paths`计数加一。

- 跳转到`abandon_entry`

**TRIMMING修剪阶段**

- 如果不是`dumb_mode`且当前的没有经过trim（`!queue_cur->trim_done`）
- 调用`trim_case`对当前项进行修建。返回`res`

```
trim_case(argv, queue_cur, in_buf)
```

- 如果`res`为`FAULT_ERROR`，直接`abort`
- 如果设置了`stop_soon`
- `cur_skipped_paths`计数加一。
- 跳转到`abandon_entry`
- 设置当前`queue_cur`为已经trim过。

```
queue_cur->trim_done = 1
```

- 如果`len`不等于`queue_cur->len`。

- 令`len` = `queue_cur->len`。

- 将`in_buf`中的内容拷贝`len`到`out_buf`。

**PERFORMANCE SCORE阶段**

- 调用`calculate_score(queue_cur)`计算当前`queue_cur`的score
- 如果设置了`skip_deterministic`或者`queue_cur->was_fuzzed`（被fuzz过了）或者`queue_cur->passed_det=1`
- 直接`goto havoc_stage`
- 如果当前的`queue_cur->exec_cksum % master_max`不等于`master_id - 1`，那么`goto havoc_stage`

*Skip deterministic fuzzing if exec path checksum puts this out of scope for this master instance.*

- 设置`doing_det = 1`

**SIMPLE BITFLIP (+dictionary construction)阶段**

```c
stage_short = "flip1";
stage_name = "bitflip 1/1";
```

- 定义`stage_max`为`len << 3`
- `stage_val_type`为`STAGE_VAL_NONE`
- `orig_hit_cnt`为`queued_paths + unique_crashes`

- 而`prev_cksum`为`queue_cur->exec_cksum`

- 接下来进入一个for循环

```c
for (stage_cur = 0; stage_cur < stage_max; stage_cur++) {
    stage_cur_byte = stage_cur >> 3;
    FLIP_BIT(out_buf, stage_cur);
    if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry;
    FLIP_BIT(out_buf, stage_cur);
    .......
}
```

- 这个循环中调用了`FLIP_BIT(out_buf, stage_cur)`

```c
#define FLIP_BIT(_ar, _b) do { \
    u8* _arf = (u8*)(_ar); \
    u32 _bf = (_b); \
    _arf[(_bf) >> 3] ^= (128 >> ((_bf) & 7)); \
} while (0)
```

- `((_bf) & 7)`相当于模8，产生了（0、1、2、3、4、5、6、7）
- 128是二进制的10000000.
- 等式的右边相当于将128右移动0-7个单位产生了二进制从（10000000 - 1）
- `((_bf) >> 3)`相当于`_bf/8`
- `stage_cur`最大为`stage_max`相当于`len << 3`
- 所以对于`FLIP_BIT(_ar, _b)`来说，`_bf`最大为`(len << 3)>>3`还是`len`

- 也就是说，对于这个for循环来说，每运行8次循环`_arf[i]`（大小为一个字节）的下标`i`就会加一，`i`最大为`len`。
- 同时在每8次为一组的循环中，128分别右移0、1、2、3、4、5、6、7位，将右移后产生的数字与`_arf[i]`进行异或翻转，而`_arf[i]`大小为一个字节，等价于对这个字节的每一位都做一次翻转异或

![](upload/attach/202102/876323_YJJ34SRMKCQP5TE.jpg)

- 当这一位被异或完毕后，调用`common_fuzz_stuff(argv, out_buf, len)`进行fuzz。
- 如果返回1，`goto abandon_entry`

- 最后再调用一次`FLIP_BIT(out_buf, stage_cur)`异或翻转回来。
- 这一部分代码中给出了注释进行解释：
- 比如说对于一串二进制：xxxxxxxxIHDRxxxxxxxx
- 当我们改变IHDR中的任意一个都会导致路径的改变or破坏， "IHDR"就像在二进制串中的一整体的具有原子性的可检查的特殊值（原语？）。*"IHDR" is an atomically-checked magic value of special significance to the fuzzed format.*，afl希望能找到这些值。
- 如果不是`dumb_mode`且`stage_cur & 7`不等于7

- 计算当前`trace_bits`的`hash32`为`cksum`
- 如果当前到达最后一轮循环并且`cksum == prev_cksum`
- 如果`a_len`小于`MAX_AUTO_EXTRA`
- 令`a_collect[a_len]`为`out_buf[stage_cur >> 3]`

- `a_len`递增1

- 如果`a_len`在`MIN_AUTO_EXTRA`与`MAX_AUTO_EXTRA`之间

- 调用`maybe_add_auto(a_collect, a_len)`
- 如果`cksum != prev_cksum`
- 如果`a_len`在`MIN_AUTO_EXTRA`与`MAX_AUTO_EXTRA`之间
- 调用`maybe_add_auto(a_collect, a_len)`将发现的新token加入`a_extra[]`

- `a_len`归零

- 令`prev_cksum = cksum`

- 如果`cksum != queue_cur->exec_cksum`

- 若`a_len < MAX_AUTO_EXTRA`

- `a_collect[a_len] = out_buf[stage_cur >> 3]`

- `a_len`递增1.

- 更新`new_hit_cnt`为`queued_paths + unique_crashes`

- 更新`stage_finds[STAGE_FLIP1]`

```
stage_finds[STAGE_FLIP1] += new_hit_cnt - orig_hit_cnt
```

- 加上所有的新的路径和新crashs

- 更新`stage_cycles[STAGE_FLIP1]`

```
stage_cycles[STAGE_FLIP1] += stage_max
```

- 加上bitflip 1/1阶段for循环中执行的次数（`common_fuzz_stuff`）

- 接下来进入`bitflip 2/1`
- 保存当前`new_hit_cnt`到`orig_hit_cnt`。
- 经过一个for循环做bit flip和样例运行。

```c
stage_name = "bitflip 2/1";
stage_short = "flip2";
stage_max = (len << 3) - 1;
orig_hit_cnt = new_hit_cnt;

for (stage_cur = 0; stage_cur < stage_max; stage_cur++) {
    stage_cur_byte = stage_cur >> 3;
    FLIP_BIT(out_buf, stage_cur);
    FLIP_BIT(out_buf, stage_cur + 1);
    if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry;
    FLIP_BIT(out_buf, stage_cur);
    FLIP_BIT(out_buf, stage_cur + 1);
}
```

- 这一次唯一的不同是是每次连续异或翻转两个bit。
- 翻转结束后更新`new_hit_cnt`
- 更新`stage_finds[STAGE_FLIP2]`与`stage_cycles[STAGE_FLIP2]`

- 接下来同样的进入`bitflip 4/1`，连续翻转4次
- 生成Effector map

```c
/* Effector map setup. These macros calculate:
   EFF_APOS - position of a particular file offset in the map.
   EFF_ALEN - length of a map with a particular number of bytes.
   EFF_SPAN_ALEN - map span for a sequence of bytes.
*/

#define EFF_APOS(_p) ((_p) >> EFF_MAP_SCALE2)
#define EFF_REM(_x) ((_x) & ((1 << EFF_MAP_SCALE2) - 1))
#define EFF_ALEN(_l) (EFF_APOS(_l) + !!EFF_REM(_l))
#define EFF_SPAN_ALEN(_p, _l) (EFF_APOS((_p) + (_l) - 1) - EFF_APOS(_p) + 1)
```

- 首先分配`len`大小的空间`eff_map`
- 将`eff_map[0]`初始化为1；将`eff_map[(len - 1)>>3]`初始化为1（及第一项和最后一项）
- 进入`"bitflip 8/8"`阶段

- 本阶段有一个很重要的思想：我们通过对于 `out_buf`所有bit进行异或翻转，如果产生了不一样的路径，就在`eff_map`中标记为1，否则为0。因为如果对所有bit都做翻转还无法带来相关的路径变化，afl认为在后续的一些开销更大的阶段，参考`eff_map`，可以对这些无效的byte进行跳过。减小开销。

- 这个阶段中不是通过FILP宏来做翻转，而是直接与`0xff`做异或。

```c
for (stage_cur = 0; stage_cur < stage_max; stage_cur++) {
    stage_cur_byte = stage_cur;
    out_buf[stage_cur] ^= 0xFF; //直接通过对于out_buf的每一个字节中的每一个bit做异或翻转。
    if (common_fuzz_stuff(argv, out_buf, len)) goto abandon_entry; //运行对应的test case（翻转后）
    if (!eff_map[EFF_APOS(stage_cur)]) { //如果eff_map[stage_cur>>3]为0的话
        //EFF_APOS宏也起到了一个将stage_cur>>3的效果
        u32 cksum;
        /* If in dumb mode or if the file is very short, just flag everything
           without wasting time on checksums. */
        if (!dumb_mode && len >= EFF_MIN_LEN)//如果不是dumb_mode且len大于最小的EFF_MIN_LEN
            cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST);//计算hash32
        else //否则，如果是dumb mode或者len过小
            cksum = ~queue_cur->exec_cksum;
        if (cksum != queue_cur->exec_cksum) {
            eff_map[EFF_APOS(stage_cur)] = 1;//产生新的路径，发生了变化，此时直接将对应的eff_map中的项标记为1
            eff_cnt++;
        }
    }
    out_buf[stage_cur] ^= 0xFF;//从新异或回来
}
```

- 如果`eff_map`的密度超过了`EFF_MAX_PERC`，那么将整个`eff_map`都标记为1（即使不这样做，我们也不会省很多时间）

- 更新`new_hit_cnt`、`stage_finds[STAGE_FLIP8]`、`stage_cycles[STAGE_FLIP8]`

- 如果`len<2`，直接跳到`skip_bitflip`
- 进入`"bitflip 16/8"`

- 唯一不同的是，在异或变异之前先检查了对应的`eff_map`的对应两个字节是否为0
