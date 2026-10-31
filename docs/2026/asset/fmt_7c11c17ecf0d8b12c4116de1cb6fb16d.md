
# 页面缓存与写回机制

## 写操作

### `bch2_write_begin`

```c
int bch2_write_begin(struct file *file, struct address_space *mapping, loff_t pos, unsigned len, struct folio **foliop, void **fsdata)
{
struct bch_inode_info *inode = to_bch_ei(mapping->host);
struct bch_fs *c = inode->v.i_sb->s_fs_info;
struct bch2_folio_reservation *res;
struct folio *folio;
unsigned offset;
int ret = -ENOMEM;

res = kmalloc(sizeof(*res), GFP_KERNEL);
if (!res)
return -ENOMEM;

bch2_folio_reservation_init(c, inode, res);
*fsdata = res;

bch2_pagecache_add_get(inode);

folio = __filemap_get_folio(mapping, pos >> PAGE_SHIFT,
FGP_WRITEBEGIN | fgf_set_order(len),
mapping_gfp_mask(mapping));
if (IS_ERR_OR_NULL(folio))
goto err_unlock;

offset = pos - folio_pos(folio);
len = min_t(size_t, len, folio_end_pos(folio) - pos);

if (folio_test_uptodate(folio))
goto out;

/* If we're writing entire folio, don't need to read it in first: */
if (!offset && len == folio_size(folio))
goto out;

if (!offset && pos + len >= inode->v.i_size) {
folio_zero_segment(folio, len, folio_size(folio));
flush_dcache_folio(folio);
goto out;
}

if (folio_pos(folio) >= inode->v.i_size) {
folio_zero_segments(folio, 0, offset, offset + len, folio_size(folio));
flush_dcache_folio(folio);
goto out;
}
readpage:
ret = bch2_read_single_folio(folio, mapping);
if (ret)
goto err;
out:
ret = bch2_folio_set(c, inode_inum(inode), &folio, 1);
if (ret)
goto err;

ret = bch2_folio_reservation_get(c, inode, folio, res, offset, len);
if (ret) {
if (!folio_test_uptodate(folio)) {
goto readpage;
}
goto err;
}

*foliop = folio;
return 0;
err:
folio_unlock(folio);
folio_put(folio);
err_unlock:
bch2_pagecache_add_put(inode);
kfree(res);
*fsdata = NULL;
return bch2_err_class(ret);
}
```

#### write_end 写入结束

- 如果未复制数据，直接解锁返回
- 将页面标记为最新（`uptodate`）
- 更新文件大小（如果扩展）
- 标记页面为脏（`dirty`），触发后续写回
- 解锁页面并返回实际复制的字节数

### `fuse_write_end`

```c
static int fuse_write_end(struct file *file, struct address_space *mapping, loff_t pos, unsigned len, unsigned copied, struct folio *folio, void *fsdata)
{
struct inode *inode = folio->mapping->host;

/* Haven't copied anything? Skip zeroing, size extending, dirtying. */
if (!copied)
goto unlock;

pos += copied;
if (!folio_test_uptodate(folio)) {
size_t endoff = pos & ~PAGE_MASK;
if (endoff)
folio_zero_segment(folio, endoff, PAGE_SIZE);
folio_mark_uptodate(folio);
}

if (pos > inode->i_size)
i_size_write(inode, pos);

folio_mark_dirty(folio);

unlock:
folio_unlock(folio);
folio_put(folio);

return copied;
}
```

### Mmap 路径

```
用户写入mmap内存
└─> 触发写保护缺页中断
└─> handle_mm_fault()
└─> do_wp_page()
└─> do_page_mkwrite()
└─> vma->vm_ops->page_mkwrite()
└─> filemap_page_mkwrite()
└─> folio_mark_dirty()    [标记脏页]
```

## Page Caches写回(脏数据回写)

### 调用链

#### 1. 脏页达到阈值触发后台写回

- 脏页超过`background_thresh`时，唤醒后台写回线程
- 脏页超过`dirty_thresh`时，限流写入进程并强制写回
- 通过`balance_dirty_pages()`检测并触发

```
写入进程产生脏页
└─> folio_mark_dirty()
└─> __mark_inode_dirty()
└─> inode_io_list_move_locked()
└─> wb_wakeup_delayed()
└─> mod_delayed_work()
└─> wb_workfn()                    [workqueue异步执行]
└─> wb_do_writeback()
└─> wb_writeback()
├─> queue_io()         [将脏inode加入b_io队列]
└─> __writeback_inodes_wb()  ⭐
└─> writeback_sb_inodes()  ⭐
└─> writeback_single_inode()  ⭐
└─> __writeback_single_inode()  ⭐
└─> do_writepages()  ⭐
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
```

#### 2. 显式同步操作

- `fsync()`/`fdatasync()`：同步单个文件
- `sync()`/`syncfs()`：同步整个文件系统
- `sync_file_range()`：同步文件指定范围

```
fsync() / fdatasync()
└─> vfs_fsync()
└─> file->f_op->fsync()
└─> file_write_and_wait_range()
└─> __filemap_fdatawrite_range()
└─> filemap_fdatawrite_wbc()
└─> do_writepages()  ⭐  [直接调用，同步执行]
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
└─> filemap_fdatawait_range()  [等待写回完成]
└─> folio_wait_writeback()
```

#### 3. 内存压力触发写回

- 内存回收时，先写回脏页再回收
- 通过`wakeup_flusher_threads(WB_REASON_VMSCAN)`触发

```
内存回收 (vmscan)
└─> shrink_inode_list()
└─> wakeup_flusher_threads(WB_REASON_VMSCAN)
└─> __wakeup_flusher_threads_bdi()
└─> wb_start_writeback()
└─> wb_wakeup()
└─> mod_delayed_work()
└─> wb_workfn()                    [workqueue异步执行]
└─> wb_do_writeback()
└─> wb_writeback()
├─> queue_io()
└─> __writeback_inodes_wb()  ⭐
└─> writeback_sb_inodes()  ⭐
└─> writeback_single_inode()  ⭐
└─> __writeback_single_inode()  ⭐
└─> do_writepages()  ⭐
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
```

#### 4. 定时写回（Periodic Writeback）

- 定期写回旧数据（kupdate风格）
- 默认每`dirty_writeback_interval`秒执行一次

```
定时器触发
└─> wb_check_old_data_flush()
└─> wb_writeback()
├─> queue_io()                    [选择旧脏页，dirtied_before参数]
└─> __writeback_inodes_wb()  ⭐
└─> writeback_sb_inodes()  ⭐
└─> writeback_single_inode()  ⭐
└─> __writeback_single_inode()  ⭐
└─> do_writepages()  ⭐
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
```

#### 5. 文件系统卸载

- 卸载前必须写回所有脏数据

```
文件系统卸载
└─> sync_filesystem()
└─> sync_inodes_sb()
└─> writeback_inodes_sb()
└─> writeback_sb_inodes()  ⭐
└─> writeback_single_inode()  ⭐
└─> __writeback_single_inode()  ⭐
└─> do_writepages()  ⭐
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
```

#### 6. 页面回收（Page Reclaim）

- 回收脏页前先写回

```
页面回收
└─> shrink_folio_list()
└─> folio_writeback()
└─> writeback_inodes_wb()
└─> __writeback_inodes_wb()  ⭐
└─> writeback_sb_inodes()  ⭐
└─> writeback_single_inode()  ⭐
└─> __writeback_single_inode()  ⭐
└─> do_writepages()  ⭐
└─> a_ops->writepages()  ⭐
└─> write_cache_pages()
└─> a_ops->writepage()  ⭐
```

### PerBDI

> 本质上这些不同路径都是在向 `BDI` 提交任务，或者直接调用 `bdi` 的写回函数

#### 工作队列初始化

> 功能：创建全局写回工作队列`bdi_wq`，用于执行所有写回工作。

```c
static int __init default_bdi_init(void)
{
    bdi_wq = alloc_workqueue("writeback", WQ_MEM_RECLAIM | WQ_UNBOUND |
                 WQ_SYSFS, 0);
    if (!bdi_wq)
        return -ENOMEM;
    return 0;
}
subsys_initcall(default_bdi_init);
```

#### bdi_writeback 初始化

- 初始化inode列表（`b_dirty`、`b_io`、`b_more_io`、`b_dirty_time`）
- 初始化工作列表（`work_list`）
- 初始化延迟工作项（`dwork`绑定到`wb_workfn`）
- 初始化写回带宽和限流参数
