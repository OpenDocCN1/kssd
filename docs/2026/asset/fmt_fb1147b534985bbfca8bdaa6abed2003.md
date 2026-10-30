
```
// 主循环中的同步调用
if (!stop_soon && sync_id && !skipped_fuzz) {
    if (!(sync_interval_cnt++ % SYNC_INTERVAL))
        sync_fuzzers(use_argv);
}
#define SYNC_INTERVAL 5  // 每5次fuzz_one()调用后同步一次
```

可以看到同步时机是5次

##### sync_fuzzers

```
static void sync_fuzzers(char** argv) {
    DIR* sd;
    struct dirent* sd_ent;
    u32 sync_cnt = 0;

    // 1. 打开同步目录
    sd = opendir(sync_dir);
    if (!sd) PFATAL("Unable to open '%s'", sync_dir);

    // 2. 遍历同步目录中的所有fuzzer实例
    while ((sd_ent = readdir(sd))) {
        // 跳过自己和隐藏文件
        if (sd_ent->d_name[0] == '.' || !strcmp(sync_id, sd_ent->d_name))
            continue;

        // 3. 检查是否有queue子目录
        qd_path = alloc_printf("%s/%s/queue", sync_dir, sd_ent->d_name);
        if (!(qd = opendir(qd_path))) {
            ck_free(qd_path);
            continue;
        }

        // 4. 读取同步状态
        qd_synced_path = alloc_printf("%s/.synced/%s", out_dir, sd_ent->d_name);
        id_fd = open(qd_synced_path, O_RDWR | O_CREAT, 0600);

        // 读取上次同步到的测试用例ID
        if (read(id_fd, &min_accept, sizeof(u32)) > 0)
            lseek(id_fd, 0, SEEK_SET);

        // 遍历其他fuzzer的队列文件
        while ((qd_ent = readdir(qd))) {
            // 解析文件名，提取用例ID
            if (qd_ent->d_name[0] == '.' ||
                sscanf(qd_ent->d_name, CASE_PREFIX "%06u", &syncing_case) != 1 ||
                syncing_case < min_accept)
                continue;

            // 只处理新的测试用例
            if (syncing_case >= next_min_accept)
                next_min_accept = syncing_case + 1;

            path = alloc_printf("%s/%s", qd_path, qd_ent->d_name);
            fd = open(path, O_RDONLY);

            if (fd < 0) {
                ck_free(path);
                continue;
            }

            // 读取测试用例内容
            if (fstat(fd, &st)) PFATAL("fstat() failed");

            if (st.st_size && st.st_size <= MAX_FILE) {
                u8* mem = mmap(0, st.st_size, PROT_READ, MAP_PRIVATE, fd, 0);

                // 执行测试用例
                write_to_testcase(mem, st.st_size);
                fault = run_target(argv, exec_tmout);

                // 检查是否有趣并保存
                syncing_party = sd_ent->d_name;
                queued_imported += save_if_interesting(argv, mem, st.st_size, fault);
                syncing_party = 0;

                munmap(mem, st.st_size);
            }
        }

        // 更新同步状态
        ck_write(id_fd, &next_min_accept, sizeof(u32), qd_synced_path);
    }
}
```

可以看到，保留用例来自于`save_if_interesting`函数：

```
static u8 save_if_interesting(char** argv, void* mem, u32 len, u8 fault) {
    u8 hnb;
    u8 keeping = 0, res;

    if (fault == crash_mode) {
        // 关键：检查是否有新的覆盖率位
        if (!(hnb = has_new_bits(virgin_bits))) {
            if (crash_mode) total_crashes++;
            return 0; // 没有新覆盖率，不保存
        }

        // 有新覆盖率，保存到队列
        add_to_queue(fn, len, 0);

        if (hnb == 2) {
            queue_top->has_new_cov = 1; // 标记为有新覆盖率
            queued_with_cov++;
        }

        keeping = 1;
    }

    return keeping;
}
```

```
static inline u8 has_new_bits(u8* virgin_map) {
    u64* current = (u64*)trace_bits; // 当前执行的覆盖率
    u64* virgin  = (u64*)virgin_map; // 全局virgin位图
    u32  i = (MAP_SIZE >> 3);
    u8   ret = 0;

    while (i--) {
        // 检查当前执行是否触及了virgin位图中的新位
        if (unlikely(*current) && unlikely(*current & *virgin)) {
            if (likely(ret < 2)) {
                u8* cur = (u8*)current;
                u8* vir = (u8*)virgin;

                // 检查是否有完全新的路径 (virgin位为0xff)
                if ((cur[0] && vir[0] == 0xff) || (cur[1] && vir[1] == 0xff) ||
                    (cur[2] && vir[2] == 0xff) || (cur[3] && vir[3] == 0xff) ||
                    (cur[4] && vir[4] == 0xff) || (cur[5] && vir[5] == 0xff) ||
                    (cur[6] && vir[6] == 0xff) || (cur[7] && vir[7] == 0xff))
                    ret = 2; // 发现新路径
                else
                    ret = 1; // 只是命中次数变化
            }

            // 更新virgin位图
            *virgin &= ~*current;
        }
        current++;
        virgin++;
    }

    return ret;
}
```

`ret=0`的话代表没有新覆盖率，`ret=1`代表发现已知路径的新的命中次数，`ret=2`代表发现全新的执行路径

#### bitmap

```
#define MAP_SIZE_POW2       16          // 2^16 = 65536
#define MAP_SIZE            (1 << MAP_SIZE_POW2)  // 65536字节

EXP_ST u8  virgin_bits[MAP_SIZE], // 未触及的覆盖率区域
       virgin_tmout[MAP_SIZE], // 超时情况下的virgin位图
       virgin_crash[MAP_SIZE]; // 崩溃情况下的virgin位图
```

```
EXP_ST void setup_shm(void) {
    // 关键：如果没有加载bitmap (-B选项)，则初始化为全1
    if (!in_bitmap) memset(virgin_bits, 255, MAP_SIZE);

    memset(virgin_tmout, 255, MAP_SIZE);
    memset(virgin_crash, 255, MAP_SIZE);

    // ... 共享内存设置
}
```

```
// Virgin位图的值含义：
// 0xFF (11111111) - 完全未触及的路径
// 0xFE (11111110) - 触及过1次
// 0xFC (11111100) - 触及过1-2次
// 0xF8 (11111000) - 触及过1-4次
// 0xF0 (11110000) - 触及过1-8次
// 0xE0 (11100000) - 触及过1-16次
// 0xC0 (11000000) - 触及过1-32次
// 0x80 (10000000) - 触及过1-128次
// 0x00 (00000000) - 触及过128+次
```

#### Bitmap文件的十六进制示例

```bash
# 查看bitmap文件内容（前32字节）

hexdump -C output_dir/fuzz_bitmap | head -5

00000000 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff |................|
00000010 ff ff ff fe ff ff fc ff ff f8 ff ff f0 ff ff e0 |................|
00000020 ff ff c0 ff ff 80 ff ff 00 ff ff ff ff ff ff ff |................|
00000030 ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff |................|
```

解读：

- `0x0000-0x000F`: 全部为`0xFF`，表示这些路径完全未被触及
- `0x0010`: `0xFE`，表示这个路径被触及过1次
- `0x0012`: `0xFC`，表示这个路径被触及过2-3次
- `0x0014`: `0xF8`，表示这个路径被触及过4-7次
- 以此类推...

### 系统环境准备和初始化

接下来就是一些参数的验证和环境的检查

```
// 1. 必需参数检查
if (optind == argc || !in_dir || !out_dir) usage(argv[0]);

// 2. 信号处理器设置
setup_signal_handlers();

// 3. ASAN选项检查
check_asan_opts();

// 4. 同步设置
if (sync_id) fix_up_sync();

// 5. 目录冲突检查
if (!strcmp(in_dir, out_dir))
    FATAL("Input and output directories can't be the same");
```

```
if (dumb_mode) {
    if (crash_mode) FATAL("-C and -n are mutually exclusive");
    if (qemu_mode)  FATAL("-Q and -n are mutually exclusive");
}
```

同时`dumb_mode`与`crash`和`qemu`模式不兼容

```
if (getenv("AFL_NO_FORKSRV"))    no_forkserver    = 1;
if (getenv("AFL_NO_CPU_RED"))    no_cpu_meter_red = 1;
if (getenv("AFL_NO_ARITH"))      no_arith         = 1;
if (getenv("AFL_SHUFFLE_QUEUE")) shuffle_queue    = 1;
if (getenv("AFL_FAST_CAL"))      fast_cal         = 1;
```

重要环境变量：

| 变量 | 功能 | 影响 |
| --- | --- | --- |
| `AFL_NO_FORKSRV` | 禁用fork server | 性能下降 |
| `AFL_NO_CPU_RED` | 禁用CPU红色警告 | 界面优化 |
| `AFL_NO_ARITH` | 禁用算术变异 | 策略简化 |
| `AFL_SHUFFLE_QUEUE` | 随机化队列顺序 | 探索策略 |
| `AFL_FAST_CAL` | 快速校准 | 启动优化 |
