
```C++
vm\_fault\_t filemap\_fault(struct vm\_fault \*vmf)
{
int error;
struct file \*file = vmf->vma->vm\_file;
struct file \*fpin = NULL;
struct address\_space \*mapping = file->f\_mapping;
struct inode \*inode = mapping->host;
pgoff\_t max\_idx, index = vmf->pgoff;
struct folio \*folio;
vm\_fault\_t ret = 0;
bool mapping\_locked = false;
max\_idx = DIV\_ROUND\_UP(i\_size\_read(inode), PAGE\_SIZE);
if (unlikely(index >= max\_idx))
return VM\_FAULT\_SIGBUS;
trace\_mm\_filemap\_fault(mapping, index);
/*
 * Do we have something in the page cache already?
 */
folio = filemap\_get\_folio(mapping, index);
if (likely(!IS\_ERR(folio))) {
/*
 * We found the page, so try async readahead before waiting for
 * the lock.
 */
if (!(vmf->flags & FAULT\_FLAG\_TRIED))
fpin = do\_async\_mmap\_readahead(vmf, folio);
if (unlikely(!folio\_test\_uptodate(folio))) {
filemap\_invalidate\_lock\_shared(mapping);
mapping\_locked = true;
}
} else {
ret = filemap\_fault\_recheck\_pte\_none(vmf);
if (unlikely(ret))
return ret;
/* No page in the page cache at all */
count\_vm\_event(PGMAJFAULT);
count\_memcg\_event\_mm(vmf->vma->vm\_mm, PGMAJFAULT);
ret = VM\_FAULT\_MAJOR;
fpin = do\_sync\_mmap\_readahead(vmf);
retry\_find:
/*
 * See comment in filemap\_create\_folio() why we need
 * invalidate\_lock
 */
if (!mapping\_locked) {
filemap\_invalidate\_lock\_shared(mapping);
mapping\_locked = true;
}
folio = \_\_filemap\_get\_folio(mapping, index,
FGP\_CREAT|FGP\_FOR\_MMAP,
vmf->gfp\_mask);
if (IS\_ERR(folio)) {
if (fpin)
goto out\_retry;
filemap\_invalidate\_unlock\_shared(mapping);
return VM\_FAULT\_OOM;
}
}
if (!lock\_folio\_maybe\_drop\_mmap(vmf, folio, &fpin))
goto out\_retry;
/* Did it get truncated? */
if (unlikely(folio->mapping != mapping)) {
folio\_unlock(folio);
folio\_put(folio);
goto retry\_find;
}
VM\_BUG\_ON\_FOLIO(!folio\_contains(folio, index), folio);
/*
 * We have a locked folio in the page cache, now we need to check
 * that it's up-to-date. If not, it is going to be due to an error,
 * or because readahead was otherwise unable to retrieve it.
 */
if (unlikely(!folio\_test\_uptodate(folio))) {
/*
 * If the invalidate lock is not held, the folio was in cache
 * and uptodate and now it is not. Strange but possible since we
 * didn't hold the page lock all the time. Let's drop
 * everything, get the invalidate lock and try again.
 */
if (!mapping\_locked) {
folio\_unlock(folio);
folio\_put(folio);
goto retry\_find;
}
/*
 * OK, the folio is really not uptodate. This can be because the
 * VMA has the VM\_RAND\_READ flag set, or because an error
 * arose. Let's read it in directly.
 */
goto page\_not\_uptodate;
}
/*
 * We've made it this far and we had to drop our mmap\_lock, now is the
 * time to return to the upper layer and have it re-find the vma and
 * redo the fault.
 */
if (fpin) {
folio\_unlock(folio);
goto out\_retry;
}
if (mapping\_locked)
filemap\_invalidate\_unlock\_shared(mapping);
/*
 * Found the page and have a reference on it.
 * We must recheck i\_size under page lock.
 */
max\_idx = DIV\_ROUND\_UP(i\_size\_read(inode), PAGE\_SIZE);
if (unlikely(index >= max\_idx)) {
folio\_unlock(folio);
folio\_put(folio);
return VM\_FAULT\_SIGBUS;
}
vmf->page = folio\_file\_page(folio, index);
return ret | VM\_FAULT\_LOCKED;
page\_not\_uptodate:
/*
 * Umm, take care of errors if the page isn't up-to-date.
 * Try to re-read it \_once\_. We do this synchronously,
 * because there really aren't any performance issues here
 * and we need to check for errors.
 */
fpin = maybe\_unlock\_mmap\_for\_io(vmf, fpin);
error = filemap\_read\_folio(file, mapping->a\_ops->read\_folio, folio);
if (fpin)
goto out\_retry;
folio\_put(folio);
if (!error || error == AOP\_TRUNCATED\_PAGE)
goto retry\_find;
filemap\_invalidate\_unlock\_shared(mapping);
return VM\_FAULT\_SIGBUS;
out\_retry:
/*
 * We dropped the mmap\_lock, we need to return to the fault handler to
 * re-find the vma and come back and find our hopefully still populated
 * page.
 */
if (!IS\_ERR(folio))
folio\_put(folio);
if (mapping\_locked)
filemap\_invalidate\_unlock\_shared(mapping);
if (fpin)
fput(fpin);
return ret | VM\_FAULT\_RETRY;
}
EXPORT\_SYMBOL(filemap\_fault);
```

#### `filemap_read_folio` 读取单个页面

- 调用文件系统的`read_folio`读取页面
- 等待I/O完成
- 检查页面是否最新
- 记录内存压力统计

```Go
static int filemap\_read\_folio(struct file \*file, filler\_t filler,
struct folio \*folio)
{
bool workingset = folio\_test\_workingset(folio);
unsigned long pflags;
int error;
/* Start the actual read. The read will unlock the page. */
if (unlikely(workingset))
psi\_memstall\_enter(&pflags);
error = filler(file, folio);
if (unlikely(workingset))
psi\_memstall\_leave(&pflags);
if (error)
return error;
error = folio\_wait\_locked\_killable(folio);
if (error)
return error;
if (folio\_test\_uptodate(folio))
return 0;
if (file)
shrink\_readahead\_size\_eio(&file->f\_ra);
return -EIO;
}
```

#### `__filemap_get_folio`(没有命中时创建)

```C
┌─────────────────────────────────────────┐
│ \_\_filemap\_get\_folio(mapping, index, │
│ fgp\_flags, gfp) │
└─────────────────────────────────────────┘
│
▼
┌─────────────────────┐
│ filemap\_get\_entry() │ ← 从radix tree查找
└──────────┬──────────┘
│
┌──────────┴──────────┐
│ │
找到 未找到
│ │
▼ ▼
┌──────────┐ ┌──────────────────┐
│检查类型 │ │ 检查FGP\_CREAT? │
│xa\_is\_ │ └────────┬─────────┘
│value()? │ │
└────┬─────┘ │
│ ┌────┴────┐
否 是 否
│ │ │
▼ ▼ ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│处理已存在 │ │创建新页面 │ │返回ENOENT│
│页面 │ │ │ └──────────┘
└────┬─────┘ └────┬─────┘
│ │
│ ┌────────┴────────┐
│ │ 1. 确定order │
│ │ 2. 对齐index │
│ │ 3. 调整gfp标志 │
│ │ 4. 降级分配循环 │
│ │ - 分配folio │
│ │ - 加入PageCache│
│ │ - 失败则降级 │
│ └────────┬────────┘
│ │
│ ┌────────┴────────┐
│ │ filemap\_add\_ │
│ │ folio()返回 │
│ └────────┬────────┘
│ │
│ ┌────────┴────────┐
│ │ -EEXIST? │
│ │ 是→goto repeat │
│ │ 否→继续 │
│ └────────┬────────┘
│ │
│ ┌────────┴────────┐
│ │ FGP\_FOR\_MMAP? │
│ │ 是→解锁页面 │
│ └────────┬────────┘
│ │
└─────────────┴─────────┐
│
▼
┌────────────────┐
│ 返回folio │
│ (引用计数+1) │
└────────────────┘
```

- `mapping`: 地址空间（`address_space`），对应文件的PageCache
- `index`: 页面索引（页号）
- `fgp_flags`: 控制行为的标志位（见下文）
- `gfp`: 内存分配标志（仅在创建页面时使用）
