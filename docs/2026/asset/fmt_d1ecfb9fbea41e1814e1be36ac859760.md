
```c
ssize_t generic_perform_write(struct kiocb *iocb, struct iov_iter *i)
{
	struct file *file = iocb->ki_filp;
	loff_t pos = iocb->ki_pos;
	struct address_space *mapping = file->f_mapping;
	const struct address_space_operations *a_ops = mapping->a_ops;
	size_t chunk = mapping_max_folio_size(mapping);
	long status = 0;
	ssize_t written = 0;
	do {
		struct folio *folio;
		size_t offset; /* Offset into folio */
		size_t bytes; /* Bytes to write to folio */
		size_t copied; /* Bytes copied from user */
		void *fsdata = NULL;
		bytes = iov_iter_count(i);
	retry:
		offset = pos & (chunk - 1);
		bytes = min(chunk - offset, bytes);
		balance_dirty_pages_ratelimited(mapping);
		/*
		 * Bring in the user page that we will copy from _first_.
		 * Otherwise there's a nasty deadlock on copying from the
		 * same page as we're writing to, without it being marked
		 * up-to-date.
		 */
		if (unlikely(fault_in_iov_iter_readable(i, bytes) == bytes)) {
			status = -EFAULT;
			break;
		}
		if (fatal_signal_pending(current)) {
			status = -EINTR;
			break;
		}
		status = a_ops->write_begin(file, mapping, pos, bytes,
					    &folio, &fsdata);
		if (unlikely(status < 0))
			break;
		offset = offset_in_folio(folio, pos);
		if (bytes > folio_size(folio) - offset)
			bytes = folio_size(folio) - offset;
		if (mapping_writably_mapped(mapping))
			flush_dcache_folio(folio);
		copied = copy_folio_from_iter_atomic(folio, offset, bytes, i);
		flush_dcache_folio(folio);
		status = a_ops->write_end(file, mapping, pos, bytes, copied,
					  folio, fsdata);
		if (unlikely(status != copied)) {
			iov_iter_revert(i, copied - max(status, 0L));
			if (unlikely(status < 0))
				break;
		}
		cond_resched();
		if (unlikely(status == 0)) {
			/*
			 * A short copy made ->write_end() reject the
			 * thing entirely. Might be memory poisoning
			 * halfway through, might be a race with munmap,
			 * might be severe memory pressure.
			 */
			if (chunk > PAGE_SIZE)
				chunk /= 2;
			if (copied) {
				bytes = copied;
				goto retry;
			}
		} else {
			pos += status;
			written += status;
		}
	} while (iov_iter_count(i));
	if (!written)
		return status;
	iocb->ki_pos += written;
	return written;
}
EXPORT_SYMBOL(generic_perform_write);
```

### `write_begin`（写入开始）

- 使用`FGP_WRITEBEGIN`标志获取或创建页面（已加锁）
- 如果页面不在`PageCache`或非最新，先读取
- 如果写入整个页面或超出`EOF`，可以跳过读取
- 返回页面和文件系统私有数据（`fsdata`）

```c
int bch2_write_begin(struct file *file, struct address_space *mapping,
		     loff_t pos, unsigned len,
		     struct folio **foliop, void **fsdata)
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
			/*
			 * If the folio hasn't been read in, we won't know if we
			 * actually need a reservation - we don't actually need
			 * to read here, we just need to check if the folio is
			 * fully backed by uncompressed data:
			 */
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

### `write_end` 写入结束

- 如果未复制数据，直接解锁返回
- 将页面标记为最新（`uptodate`）
- 更新文件大小（如果扩展）
- 标记页面为脏（`dirty`），触发后续写回
- 解锁页面并返回实际复制的字节数

```c
static int fuse_write_end(struct file *file, struct address_space *mapping,
			  loff_t pos, unsigned len, unsigned copied,
			  struct folio *folio, void *fsdata)
{
	struct inode *inode = folio->mapping->host;
	/* Haven't copied anything? Skip zeroing, size extending, dirtying. */
	if (!copied)
		goto unlock;

	pos += copied;

	if (!folio_test_uptodate(folio)) {
		/* Zero any unwritten bytes at the end of the page */
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

```text
用户写入mmap内存
└─> 触发写保护缺页中断
└─> handle_mm_fault()
└─> do_wp_page()
└─> do_page_mkwrite()
└─> vma->vm_ops->page_mkwrite()
└─> filemap_page_mkwrite()
└─> folio_mark_dirty() [标记脏页]
```

## Page Caches 写回（脏数据回写）

### 调用链

#### 1. 脏页达到阈值触发后台写回

- 脏页超过`background_thresh`时，唤醒后台写回线程
- 脏页超过`dirty_thresh`时，限流写入进程并强制写回
- 通过`balance_dirty_pages()`检测并触发

```text
写入进程产生脏页
└─> folio_mark_dirty()
└─> __mark_inode_dirty()
└─> inode_io_list_move_locked()
└─> wb_wakeup_delayed()
└─> mod_delayed_work()
└─> wb_workfn() [workqueue异步执行]
└─> wb_do_writeback()
└─> wb_writeback()
├─> queue_io() [将脏inode加入b_io队列]
└─> __writeback_inodes_wb() ⭐
└─> writeback_sb_inodes() ⭐
└─> writeback_single_inode() ⭐
└─> __writeback_single_inode() ⭐
└─> do_writepages() ⭐
└─> a_ops->writepages() ⭐
└─> write_cache_pages()
└─> a_ops->writepage() ⭐
```

#### 2. 显式同步操作

- `fsync()`/`fdatasync()`：同步单个文件
- `sync()`/`syncfs()`：同步整个文件系统
- `sync_file_range()`：同步文件指定范围

```text
fsync() / fdatasync()
└─> vfs_fsync()
└─> file->f_op->fsync()
└─> file_write_and_wait_range()
└─> __filemap_fdatawrite_range()
└─> filemap_fdatawrite_wbc()
└─> do_writepages() ⭐ [直接调用，同步执行]
└─> a_ops->writepages() ⭐
└─> write_cache_pages()
└─> a_ops->writepage() ⭐
└─> filemap_fdatawait_range() [等待写回完成]
└─> folio_wait_writeback()
```

#### 3. 内存压力触发写回

- 内存回收时，先写回脏页再回收
- 通过`wakeup_flusher_threads(WB_REASON_VMSCAN)`触发

```text
内存回收 (vmscan)
└─> shrink_inode_list()
└─> wakeup_flusher_threads(WB_REASON_VMSCAN)
└─> __wakeup_flusher_threads_bdi()
└─> wb_start_writeback()
└─> wb_wakeup()
└─> mod_delayed_work()
└─> wb_workfn() [workqueue异步执行]
└─> wb_do_writeback()
└─> wb_writeback()
├─> queue_io()
└─> __writeback_inodes_wb() ⭐
└─> writeback_sb_inodes() ⭐
└─> writeback_single_inode() ⭐
└─> __writeback_single_inode() ⭐
└─> do_writepages() ⭐
└─> a_ops->writepages() ⭐
└─> write_cache_pages()
└─> a_ops->writepage() ⭐
```

#### 4. 定时写回（Periodic Writeback）

- 定期写回旧数据（`kupdate`风格）
- 默认每`dirty_writeback_interval`秒执行一次

```text
定时器触发
└─> wb_check_old_data_flush()
└─> wb_writeback()
├─> queue_io() [选择旧脏页，dirtied_before参数]
└─> __writeback_inodes_wb() ⭐
└─> writeback_sb_inodes() ⭐
└─> writeback_single_inode() ⭐
└─> __writeback_single_inode() ⭐
└─> do_writepages() ⭐
└─> a_ops->writepages() ⭐
└─> write_cache_pages()
└─> a_ops->writepage() ⭐
```

#### 5. 文件系统卸载

- 卸载前必须写回所有脏数据

```text
文件系统卸载
└─> sync_filesystem()
└─> sync_inodes_sb()
└─> writeback_inodes_sb()
└─> writeback_sb_inodes() ⭐
└─> writeback_single_inode() ⭐
└─> __writeback_single_inode() ⭐
└─> do_writepages() ⭐
└─> a_ops->writepages() ⭐
└─> write_cache_pages()
└─> a_ops->writepage() ⭐
```

#### 6. 页面回收（Page Reclaim）

- 回收脏页前先写回
