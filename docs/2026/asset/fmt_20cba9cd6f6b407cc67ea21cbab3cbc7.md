
```c
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

#### `ext4_punch_hole`（核心函数）

- 边界检查
  - 如果`offset >= inode->i_size`，直接返回
  - 如果超出文件大小，调整长度
- 对齐处理
  - 将偏移量和长度对齐到块边界（`round_up`/`round_down`）
- 清除页面缓存
