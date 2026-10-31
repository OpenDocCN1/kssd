
```c
/* 行号: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 */
static int wb_init(struct bdi_writeback *wb, struct backing_dev_info *bdi, gfp_t gfp)
{
    int err;

    memset(wb, 0, sizeof(*wb));

    wb->bdi = bdi;
    wb->last_old_flush = jiffies;
    INIT_LIST_HEAD(&wb->b_dirty);
    INIT_LIST_HEAD(&wb->b_io);
    INIT_LIST_HEAD(&wb->b_more_io);
    INIT_LIST_HEAD(&wb->b_dirty_time);
    spin_lock_init(&wb->list_lock);

    atomic_set(&wb->writeback_inodes, 0);
    wb->bw_time_stamp = jiffies;
    wb->balanced_dirty_ratelimit = INIT_BW;
    wb->dirty_ratelimit = INIT_BW;
    wb->write_bandwidth = INIT_BW;
    wb->avg_write_bandwidth = INIT_BW;

    spin_lock_init(&wb->work_lock);
    INIT_LIST_HEAD(&wb->work_list);
    INIT_DELAYED_WORK(&wb->dwork, wb_workfn);
    INIT_DELAYED_WORK(&wb->bw_dwork, wb_update_bandwidth_workfn);

    err = fprop_local_init_percpu(&wb->completions, gfp);
    if (err)
        return err;

    err = percpu_counter_init_many(wb->stat, 0, gfp, NR_WB_STAT_ITEMS);
    if (err)
        fprop_local_destroy_percpu(&wb->completions);

    return err;
}
```

#### `wb_do_writeback` 执行写回操作

- 设置`WB_writeback_running`标志
- 从`work_list`获取工作项并执行
- 检查全量刷新、定时写回、后台写回
- 清除运行标志并返回已写回页面数

```c
/* 行号: 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 */
/*
 * Retrieve work items and do the writeback they describe
 */
static long wb_do_writeback(struct bdi_writeback *wb)
{
    struct wb_writeback_work *work;
    long wrote = 0;

    set_bit(WB_writeback_running, &wb->state);
    while ((work = get_next_work_item(wb)) != NULL) {
        trace_writeback_exec(wb, work);
        wrote += wb_writeback(wb, work);
        finish_writeback_work(work);
    }

    /*
     * Check for a flush-everything request
     */
    wrote += wb_check_start_all(wb);

    /*
     * Check for periodic writeback, kupdated() style
     */
    wrote += wb_check_old_data_flush(wb);
    wrote += wb_check_background_flush(wb);
    clear_bit(WB_writeback_running, &wb->state);

    return wrote;
}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2025-12-4 17:05 被Elenia编辑，原因：修正

[#内核](https://bbs.kanxue.com/forum-171-1-185.htm)  
[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)
