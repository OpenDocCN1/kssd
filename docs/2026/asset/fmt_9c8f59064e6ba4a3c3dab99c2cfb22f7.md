
```c
int main(int argc, char** argv) {
......
setup_signal_handlers();
check_asan_opts();
if (sync_id) fix_up_sync();
if (!strcmp(in_dir, out_dir))
FATAL("Input and output directories can't be the same");
if (dumb_mode) {
if (crash_mode) FATAL("-C and -n are mutually exclusive");
if (qemu_mode) FATAL("-Q and -n are mutually exclusive");
}
if (getenv("AFL_NO_FORKSRV")) no_forkserver = 1;
if (getenv("AFL_NO_CPU_RED")) no_cpu_meter_red = 1;
if (getenv("AFL_NO_ARITH")) no_arith = 1;
if (getenv("AFL_SHUFFLE_QUEUE")) shuffle_queue = 1;
if (getenv("AFL_FAST_CAL")) fast_cal = 1;
if (getenv("AFL_HANG_TMOUT")) {
hang_tmout = atoi(getenv("AFL_HANG_TMOUT"));
if (!hang_tmout) FATAL("Invalid value of AFL_HANG_TMOUT");
}
if (dumb_mode == 2 && no_forkserver)
FATAL("AFL_DUMB_FORKSRV and AFL_NO_FORKSRV are mutually exclusive");
if (getenv("AFL_PRELOAD")) {
setenv("LD_PRELOAD", getenv("AFL_PRELOAD"), 1);
setenv("DYLD_INSERT_LIBRARIES", getenv("AFL_PRELOAD"), 1);
}
if (getenv("AFL_LD_PRELOAD"))
FATAL("Use AFL_PRELOAD instead of AFL_LD_PRELOAD");
save_cmdline(argc, argv);
fix_up_banner(argv[optind]);
check_if_tty();
get_core_count();
#ifdef HAVE_AFFINITY
bind_to_free_cpu();
#endif /* HAVE_AFFINITY */
check_crash_handling();
check_cpu_governor();
setup_post();
setup_shm();
init_count_class16();
setup_dirs_fds();
read_testcases();
load_auto();
pivot_inputs();
if (extras_dir) load_extras(extras_dir);
if (!timeout_given) find_timeout();
detect_file_args(argv + optind + 1);
if (!out_file) setup_stdio_file();
check_binary(argv[optind]);
start_time = get_cur_time();
if (qemu_mode)
use_argv = get_qemu_argv(argv[0], argv + optind, argc - optind);
else
use_argv = argv + optind;
perform_dry_run(use_argv);
cull_queue();
show_init_stats();
seek_to = find_start_position();
write_stats_file(0, 0, 0);
save_auto();
if (stop_soon) goto stop_fuzzing;
/* Woop woop woop */
if (!not_on_tty) {
sleep(4);
start_time += 4000;
if (stop_soon) goto stop_fuzzing;
}
while (1) {
u8 skipped_fuzz;
cull_queue();
if (!queue_cur) {
queue_cycle++;
current_entry = 0;
cur_skipped_paths = 0;
queue_cur = queue;
while (seek_to) {
current_entry++;
seek_to--;
queue_cur = queue_cur->next;
}
show_stats();
if (not_on_tty) {
ACTF("Entering queue cycle %llu.", queue_cycle);
fflush(stdout);
}
/* If we had a full queue cycle with no new finds, try
recombination strategies next. */
if (queued_paths == prev_queued) {
if (use_splicing) cycles_wo_finds++; else use_splicing = 1;
} else cycles_wo_finds = 0;
prev_queued = queued_paths;
if (sync_id && queue_cycle == 1 && getenv("AFL_IMPORT_FIRST"))
sync_fuzzers(use_argv);
}
skipped_fuzz = fuzz_one(use_argv);
if (!stop_soon && sync_id && !skipped_fuzz) {
if (!(sync_interval_cnt++ % SYNC_INTERVAL))
sync_fuzzers(use_argv);
}
if (!stop_soon && exit_1) stop_soon = 2;
if (stop_soon) break;
queue_cur = queue_cur->next;
current_entry++;
}
if (queue_cur) show_stats();
/* If we stopped programmatically, we kill the forkserver and the current runner.
If we stopped manually, this is done by the signal handler. */
if (stop_soon == 2) {
if (child_pid > 0) kill(child_pid, SIGKILL);
if (forksrv_pid > 0) kill(forksrv_pid, SIGKILL);
}
/* Now that we've killed the forkserver, we wait for it to be able to get rusage stats. */
if (waitpid(forksrv_pid, NULL, 0) <= 0) {
WARNF("error waitpid\n");
}
write_bitmap();
write_stats_file(0, 0, 0);
save_auto();
stop_fuzzing:
SAYF(CURSOR_SHOW cLRD "\n\n+++ Testing aborted %s +++\n" cRST,
stop_soon == 2 ? "programmatically" : "by user");
/* Running for more than 30 minutes but still doing first cycle? */
if (queue_cycle == 1 && get_cur_time() - start_time > 30 * 60 * 1000) {
SAYF("\n" cYEL "[!] " cRST
"Stopped during the first cycle, results may be incomplete.\n"
" (For info on resuming, see %s/README.)\n", doc_path);
}
fclose(plot_file);
destroy_queue();
destroy_extras();
ck_free(target_path);
ck_free(sync_id);
alloc_report();
OKF("We're done here. Have a nice day!\n");
exit(0);
}
```

### `setup_signal_handlers`

设置一些信号处理函数，比如说退出信号时要主动释放子进程、窗口大小调整时要跟踪变化等。

### `check_asan_opts`

读取环境变量`ASAN_OPTIONS`和`MSAN_OPTIONS`，做一些检查

### `fix_up_sync`

略

### `save_cmdline`

保存当前的命令

### `fix_up_banner`

创建一个 banner

### `check_if_tty`

检查是否在tty终端上面运行。

### `get_core_count`

计数logical CPU cores。

### `check_crash_handling`

检查崩溃处理函数，确保崩溃后不会进入程序。

```c
s32 fd = open("/proc/sys/kernel/core_pattern", O_RDONLY);
u8 fchar;
if (fd < 0) return;
ACTF("Checking core_pattern...");
if (read(fd, &fchar, 1) == 1 && fchar == '|') {
SAYF("\n" cLRD "[-] " cRST
"Hmm, your system is configured to send core dump notifications to an\n"
" external utility. This will cause issues: there will be an extended delay\n"
" between stumbling upon a crash and having this information relayed to the\n"
" fuzzer via the standard waitpid() API.\n\n"
" To avoid having crashes misinterpreted as timeouts, please log in as root\n"
" and temporarily modify /proc/sys/kernel/core_pattern, like so:\n\n"
" echo core >/proc/sys/kernel/core_pattern\n");
if (!getenv("AFL_I_DONT_CARE_ABOUT_MISSING_CRASHES"))
FATAL("Pipe at the beginning of 'core_pattern'");
}
close(fd);
```

笔者在 Ubuntu20 上跑 AFL 就会遇到这个问题，因为在默认情况下，系统会将崩溃信息通过管道发送给外部程序，由于这会影响到效率，因此通过 `echo core >/proc/sys/kernel/core_pattern` 修改保存崩溃信息的方式，将它保存为本地文件。

### `check_cpu_governor`

检查`cpu`的调节器，来使得`cpu`可以处于高效的运行状态。

### `setup_post`

如果用户指定了环境变量 `AFL_POST_LIBRARY` ，那么就会从对应的路径下加载动态库并加载 `afl_postprocess` 函数并保存在 `post_handler` 中。

### `setup_shm`

```c
EXP_ST void setup_shm(void) {
u8* shm_str;
if (!in_bitmap) memset(virgin_bits, 255, MAP_SIZE);
memset(virgin_tmout, 255, MAP_SIZE);
memset(virgin_crash, 255, MAP_SIZE);
shm_id = shmget(IPC_PRIVATE, MAP_SIZE, IPC_CREAT | IPC_EXCL | 0600);
if (shm_id < 0) PFATAL("shmget() failed");
atexit(remove_shm);
shm_str = alloc_printf("%d", shm_id);
/* If somebody is asking us to fuzz instrumented binaries in dumb mode,
we don't want them to detect instrumentation, since we won't be sending
fork server commands. This should be replaced with better auto-detection
later on, perhaps? */
if (!dumb_mode) setenv(SHM_ENV_VAR, shm_str, 1);
ck_free(shm_str);
trace_bits = shmat(shm_id, NULL, 0);
if (trace_bits == (void *)-1) PFATAL("shmat() failed");
}
```

初始化 `virgin_bits` 数组用于保存后续模糊测试中覆盖的路径，`virgin_tmout` 保存超时的路径，`virgin_crash` 保存崩溃的路径。

同时建立共享内存 `trace_bits`，该变量用于储存样例运行时的路径。

同时将共享内存的唯一标识符 `shm_id` 转为字符串后保存在环境变量 `SHM_ENV_VAR` 中。

### `init_count_class16`

初始化`count_class_lookup16`数组，帮助快速归类统计路径覆盖的数量。

### `setup_dirs_fds`

创建输出目录。

### `read_testcases`

读取测试样例。
