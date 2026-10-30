
# AFL源码阅读

```cpp
static void locate_diffs(u8* ptr1, u8* ptr2, u32 len, s32* first, s32* last) {
    s32 f_loc = -1;
    s32 l_loc = -1;
    u32 pos;
    for (pos = 0; pos < len; pos++) {
        if (*(ptr1++) != *(ptr2++)) {
            if (f_loc == -1) f_loc = pos;
            l_loc = pos;
        }
    }
    *first = f_loc;
    *last = l_loc;
    return;
}
```

逐字节比较两个缓冲区，`f_loc`记录第一个不同字节的位置，`l_loc`记录最后一个不同字节的位置

```cpp
if (f_diff < 0 || l_diff < 2 || f_diff == l_diff) {
    ck_free(new_buf);
    goto retry_splicing;
}
/* Split somewhere between the first and last differing byte. */
split_at = f_diff + UR(l_diff - f_diff);
```

拼接点的选择：

- 如果文件完全相同(`f_diff < 0`)，重新选择目标
- 如果差异太小(`l_diff < 2`或`f_diff == l_diff`)，重新选择
- 在首尾差异之间随机选择拼接点，确保拼接有意义

```cpp
/* Do the thing. */
len = target->len;
memcpy(new_buf, in_buf, split_at);
in_buf = new_buf;
ck_free(out_buf);
out_buf = ck_alloc_nozero(len);
memcpy(out_buf, in_buf, len);
goto havoc_stage;
```

拼接操作开始：

- **前半部分**: 复制当前输入的前`split_at`字节到新缓冲区
- **后半部分**: 目标文件的后半部分已经在`new_buf`中
- **缓冲区切换**: 将`in_buf`指向新的拼接结果
- **重新进入havoc**: 对拼接结果进行随机变异

```cpp
abandon_entry:
splicing_with = -1;
/* Update pending_not_fuzzed count if we made it through the calibration
   cycle and have not seen this entry before. */
if (!stop_soon && !queue_cur->cal_failed && !queue_cur->was_fuzzed) {
    queue_cur->was_fuzzed = 1;
    pending_not_fuzzed--;
    if (queue_cur->favored) pending_favored--;
}
munmap(orig_in, queue_cur->len);
if (in_buf != orig_in) ck_free(in_buf);
ck_free(out_buf);
ck_free(eff_map);
return ret_val;
```

最后清理资源就能退出函数了

## `main`函数剩余部分

```cpp
if (!stop_soon && sync_id && !skipped_fuzz) {
    if (!(sync_interval_cnt++ % SYNC_INTERVAL))
        sync_fuzzers(use_argv);
}
```

同步实例

```cpp
if (!stop_soon && exit_1) stop_soon = 2;
if (stop_soon) break;
queue_cur = queue_cur->next;
current_entry++;
```

```cpp
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
```

进程清理

```cpp
write_bitmap();
write_stats_file(0, 0, 0);
save_auto();
```

最后统计信息以及数据的保存

```cpp
SAYF(CURSOR_SHOW cLRD "\n\n+++ Testing aborted %s +++\n" cRST,
     stop_soon == 2 ? "programmatically" : "by user");
/* Running for more than 30 minutes but still doing first cycle? */
if (queue_cycle == 1 && get_cur_time() - start_time > 30 * 60 * 1000) {
    SAYF("\n" cYEL "[!] " cRST
         "Stopped during the first cycle, results may be incomplete.\n"
         " (For info on resuming, see %s/README.)\n", doc_path);
}
```

如果运行了30min还在第一轮，则提示结果不完整

```cpp
fclose(plot_file);
destroy_queue();
destroy_extras();
ck_free(target_path);
ck_free(sync_id);
alloc_report();
```

资源销毁和内存清理

```cpp
OKF("We're done here. Have a nice day!\n");
exit(0);
```

最后优雅的退出

## `perform_dry_run(use_argv);`

```cpp
static void perform_dry_run(char** argv) {
    struct queue_entry* q = queue;
    u32 cal_failures = 0;
    u8* skip_crashes = getenv("AFL_SKIP_CRASHES");

    while (q) {
        u8* use_mem;
        u8  res;
        s32 fd;

        u8* fn = strrchr(q->fname, '/') + 1;

        ACTF("Attempting dry run with '%s'...", fn);

        fd = open(q->fname, O_RDONLY);
        if (fd < 0) PFATAL("Unable to open '%s'", q->fname);

        use_mem = ck_alloc_nozero(q->len);

        if (read(fd, use_mem, q->len) != q->len)
            FATAL("Short read from '%s'", q->fname);

        close(fd);

        res = calibrate_case(argv, q, use_mem, 0, 1);
        ck_free(use_mem);

        if (stop_soon) return;

        if (res == crash_mode || res == FAULT_NOBITS)
            SAYF(cGRA "    len = %u, map size = %u, exec speed = %llu us\n" cRST,
                 q->len, q->bitmap_size, q->exec_us);
        .....
    }
    .........
}
```

首先从`queue`获取测试样例（`fn == "id:000000,orig:case"`），然后读取之后调用`calibrate_case`函数
