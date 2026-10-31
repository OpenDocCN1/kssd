
```C
/*
 * preallocate space for a file. This implements ext4's fallocate file
 * operation, which gets called from sys_fallocate system call.
 * For block-mapped files, posix_fallocate should fall back to the method
 * of writing zeroes to the required new blocks (the same behavior which is
 * expected for file systems which do not support fallocate() system call).
 */
long ext4_fallocate(struct file *file, int mode, loff_t offset, loff_t len)
{
struct inode *inode = file_inode(file);
loff_t new_size = 0;
unsigned int max_blocks;
int ret = 0;
int flags;
ext4_lblk_t lblk;
unsigned int blkbits = inode->i_blkbits;
/*
 * Encrypted inodes can't handle collapse range or insert
 * range since we would need to re-encrypt blocks with a
 * different IV or XTS tweak (which are based on the logical
 * block number).
 */
if (IS_ENCRYPTED(inode) &&
(mode & (FALLOC_FL_COLLAPSE_RANGE | FALLOC_FL_INSERT_RANGE)))
return -EOPNOTSUPP;
/* Return error if mode is not supported */
if (mode & ~(FALLOC_FL_KEEP_SIZE | FALLOC_FL_PUNCH_HOLE |
FALLOC_FL_COLLAPSE_RANGE | FALLOC_FL_ZERO_RANGE |
FALLOC_FL_INSERT_RANGE))
return -EOPNOTSUPP;
inode_lock(inode);
ret = ext4_convert_inline_data(inode);
inode_unlock(inode);
if (ret)
goto exit;
if (mode & FALLOC_FL_PUNCH_HOLE) {
ret = ext4_punch_hole(file, offset, len);
goto exit;
}
if (mode & FALLOC_FL_COLLAPSE_RANGE) {
ret = ext4_collapse_range(file, offset, len);
goto exit;
}
if (mode & FALLOC_FL_INSERT_RANGE) {
ret = ext4_insert_range(file, offset, len);
goto exit;
}
if (mode & FALLOC_FL_ZERO_RANGE) {
ret = ext4_zero_range(file, offset, len, mode);
goto exit;
}
trace_ext4_fallocate_enter(inode, offset, len, mode);
lblk = offset >> blkbits;
max_blocks = EXT4_MAX_BLOCKS(len, offset, blkbits);
flags = EXT4_GET_BLOCKS_CREATE_UNWRIT_EXT;
inode_lock(inode);
/*
 * We only support preallocation for extent-based files only
 */
if (!(ext4_test_inode_flag(inode, EXT4_INODE_EXTENTS))) {
ret = -EOPNOTSUPP;
goto out;
}
if (!(mode & FALLOC_FL_KEEP_SIZE) &&
(offset + len > inode->i_size ||
offset + len > EXT4_I(inode)->i_disksize)) {
new_size = offset + len;
ret = inode_newsize_ok(inode, new_size);
if (ret)
goto out;
}
/* Wait all existing dio workers, newcomers will block on i_rwsem */
inode_dio_wait(inode);
ret = file_modified(file);
if (ret)
goto out;
ret = ext4_alloc_file_blocks(file, lblk, max_blocks, new_size, flags);
if (ret)
goto out;
if (file->f_flags & O_SYNC && EXT4_SB(inode->i_sb)->s_journal) {
ret = ext4_fc_commit(EXT4_SB(inode->i_sb)->s_journal,
EXT4_I(inode)->i_sync_tid);
}
out:
inode_unlock(inode);
trace_ext4_fallocate_exit(inode, offset, max_blocks, ret);
exit:
return ret;
}
```

#### `ext4_punch_hole`(核心函数)

- 边界检查
- 如果 `offset >= inode->i_size`，直接返回
- 如果超出文件大小，调整长度
- 对齐处理
- 将偏移量和长度对齐到块边界（`round_up`/`round_down`）
- 清除页面缓存

```C++
int ext4_punch_hole(struct file *file, loff_t offset, loff_t length)
{
struct inode *inode = file_inode(file);
struct super_block *sb = inode->i_sb;
ext4_lblk_t first_block, stop_block;
struct address_space *mapping = inode->i_mapping;
loff_t first_block_offset, last_block_offset, max_length;
struct ext4_sb_info *sbi = EXT4_SB(inode->i_sb);
handle_t *handle;
unsigned int credits;
int ret = 0, ret2 = 0;
trace_ext4_punch_hole(inode, offset, length, 0);
inode_lock(inode);
/* No need to punch hole beyond i_size */
if (offset >= inode->i_size)
goto out_mutex;
/*
 * If the hole extends beyond i_size, set the hole
 * to end after the page that contains i_size
 */
if (offset + length > inode->i_size) {
length = inode->i_size +
PAGE_SIZE - (inode->i_size & (PAGE_SIZE - 1)) -
offset;
}
/*
 * For punch hole the length + offset needs to be within one block
 * before last range. Adjust the length if it goes beyond that limit.
 */
max_length = sbi->s_bitmap_maxbytes - inode->i_sb->s_blocksize;
if (offset + length > max_length)
length = max_length - offset;
if (offset & (sb->s_blocksize - 1) ||
(offset + length) & (sb->s_blocksize - 1)) {
/*
 * Attach jinode to inode for jbd2 if we do any zeroing of
 * partial block
 */
ret = ext4_inode_attach_jinode(inode);
if (ret < 0)
goto out_mutex;
}
// 等待所有直接 I/O 完成
/* Wait all existing dio workers, newcomers will block on i_rwsem */
inode_dio_wait(inode);
ret = file_modified(file);
if (ret)
goto out_mutex;
/*
 * Prevent page faults from reinstantiating pages we have released from
 * page cache.
 */
filemap_invalidate_lock(mapping);
ret = ext4_break_layouts(inode);
if (ret)
goto out_dio;
first_block_offset = round_up(offset, sb->s_blocksize);
last_block_offset = round_down((offset + length), sb->s_blocksize) - 1;
/* Now release the pages and zero block aligned part of pages*/
if (last_block_offset > first_block_offset) {
// 释放指定范围内的页面缓存
ret = ext4_update_disksize_before_punch(inode, offset, length);
if (ret)
goto out_dio;
ret = ext4_truncate_page_cache_block_range(inode,
first_block_offset, last_block_offset + 1);
if (ret)
goto out_dio;
}
if (ext4_test_inode_flag(inode, EXT4_INODE_EXTENTS))
credits = ext4_writepage_trans_blocks(inode);
else
credits = ext4_blocks_for_truncate(inode);
handle = ext4_journal_start(inode, EXT4_HT_TRUNCATE, credits);
if (IS_ERR(handle)) {
ret = PTR_ERR(handle);
ext4_std_error(sb, ret);
goto out_dio;
}
// 零化部分块
ret = ext4_zero_partial_blocks(handle, inode, offset,
length);
if (ret)
goto out_stop;
first_block = (offset + sb->s_blocksize - 1) >>
EXT4_BLOCK_SIZE_BITS(sb);
stop_block = (offset + length) >> EXT4_BLOCK_SIZE_BITS(sb);
/* If there are blocks to remove, do it */
if (stop_block > first_block) {
ext4_lblk_t hole_len = stop_block - first_block;
down_write(&EXT4_I(inode)->i_data_sem);
ext4_discard_preallocations(inode);
ext4_es_remove_extent(inode, first_block, hole_len);
// 删除块 从 extent 树或间接块中删除块
if (ext4_test_inode_flag(inode, EXT4_INODE_EXTENTS))
ret = ext4_ext_remove_space(inode, first_block,
stop_block - 1);
else
ret = ext4_ind_remove_space(handle, inode, first_block,
stop_block);
// 标记为 HOLE 在 extent 状态树中标记为 EXTENT_STATUS_HOLE
ext4_es_insert_extent(inode, first_block, hole_len, ~0,
EXTENT_STATUS_HOLE, 0);
up_write(&EXT4_I(inode)->i_data_sem);
}
ext4_fc_track_range(handle, inode, first_block, stop_block);
if (IS_SYNC(inode))
ext4_handle_sync(handle);
inode_set_mtime_to_ts(inode, inode_set_ctime_current(inode));
ret2 = ext4_mark_inode_dirty(handle, inode);
if (unlikely(ret2))
ret = ret2;
if (ret >= 0)
ext4_update_inode_fsync_trans(handle, inode, 1);
out_stop:
ext4_journal_stop(handle);
out_dio:
filemap_invalidate_unlock(mapping);
out_mutex:
inode_unlock(inode);
return ret;
}
```

### shmem/tmpfs 文件 实现

> tmpfs 文件就是 `/tmp` 目录下的文件
>
> Shmem 文件就是 `memfd_create()` 创建的 `fd`

#### 流程图
