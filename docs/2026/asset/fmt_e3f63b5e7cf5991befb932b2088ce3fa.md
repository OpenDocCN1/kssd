
## 最终思路
再次审查源码后，发现 `cmd_rip` 函数中：

```
void  cmd_rip() { char *fname = strtok(NULL, delim);##this! char *rbuff = xmalloc(SBSIZE); /* Redirect input to stdout */ if (!fname) { read_max(rbuff, SBSIZE); write_str(rbuff); write_str("\n"); xfree(rbuff); return; } /* Redirect input to a file */ remove_slash(fname);#that! if (strlen(fname) == 0) { write_str("rip: flag = ∅\n"); xfree(rbuff); return; } /* Special case: flag1 cannot be altered */ if (!strcmp(fname,"flag1")) { write_str("rip: ¬ perm\n"); xfree(rbuff); return; } for (int i = 1 /* ignore flag1 */; i < FILEMAX; i++) { if (gfiles[i] && gfiles[i]->fname && !strcmp(gfiles[i]->fname, fname)) { /* The file's owner must be root or the current user, and the file must be writable */ if ((curr_uid != 0 && gfiles[i]->fuid != curr_uid) || !(gfiles[i]->fflag & WRPERM)) { write_str("rip: ¬ perm\n"); xfree(rbuff); return; } read_max(rbuff, SBSIZE); if (gfiles[i]->fdata) { /* File is not empty -> rewrite the file content */ if (strlen(rbuff) > 0) { gfiles[i]->fdata = xrealloc(gfiles[i]->fdata, strlen(gfiles[i]->fdata) + strlen(rbuff) + 1); /* Remember the extra null byte */ } strcat(gfiles[i]->fdata, rbuff); } else { /* File is empty -> write the content directly */ gfiles[i]->fdata = strdup(rbuff); } xfree(rbuff); return; } } write_str("rip: \""); write_str(fname); write_str("\" ∉ ℱ\n"); xfree(rbuff); }
```

注意代码中我注释的 `this` 和 `that` 部分。

关注到 `remove_slash` 函数，会将字符串中含 `\` 的部分置为 `\x00`：

```
void remove_slash(char *fname) { int fname_len = strlen(fname); for (int i = 0; i < fname_len; i++) { if (fname[i] == '/') { fname[i] = '\0'; } } }
```

同时也是通过 `strlen` 来判断长度。

`menu` 与 `cmd_rip` 函数中，按先后顺序调用了如下：

```
char *token = strtok(gbuff, delim); char *fname = strtok(NULL, delim);##this! remove_slash(fname);#that!
```

如果我们构造这样的 payload：

```
b'rip.'+b'a'*(0x400-4)
```

同时，在之前堆风水写 `gbuff` 的第 `0x400-0x408` 字节，若我们填入如下内容：

```
p16(prev_size)+b'\ '*6
```

那么在执行第一句语句之后，返回的指针已经指向

```
b'a'*(0x400-4)+p16(prev_size)+b'\ '*6
```

注意 `strtok` 第一个参数为 `NULL` 时，继承的是上一次 `strtok` 返回的指针。

因此执行第二句语句之后，能够将 `off-by-null` 的 size 头低位 `0x21` 置为 `0`，触发 `off-by-null`

同时，执行第三句语句，将调整我们的 `prev_size` 为正常。

因此至此，我们已经构造好 `prev_size`。

### fake_size

我们知道，`free` 一个堆块，会去检测下一个堆块的 `next_size` 是否合法：

```
if (__glibc_unlikely (chunksize_nomask (next) < CHUNK_HDR_SZ) || __glibc_unlikely (chunksize_nomask (next) > av->system_mem)) malloc_printerr ("malloc(): invalid next size (unsorted)");
```

我们 `free` 掉 `off-by-null-chunk` 的时候，除非 `off-by-null-chunk` 的 `size` 本身为 `0x501` 这种，`off by null` 之后不用伪造 `next_size`。否则需要伪造合法的 `next_size`，不然 `free` 会触发错误。

伪造 `next_size` 的过程几乎没有希望的，之前看来：

首先 `off-by-null-chunk` 的 `size` 需要大于 `0x400`（不能在 `tcache` 范围），在 `tcache` 范围内的 `tcache_off-by-null` 是没有作用的。

这就意味着我们申请的 `off-by-null-chunk` 的 `size` 需要大于 `0x400`，那就只能调用 `cmd_rip` 来申请，其他都不能申请大于 `0x400` size 的 chunk。而 `cmd_rip` 的操作是根据 `strlen` 函数返回的长度来进行 `realloc` 的，也就是说，我们申请出某个 size 的堆块，就必须填满他。

`cmd_rip` 中部分代码：

```
for (int i = 1 /* ignore flag1 */; i < FILEMAX; i++) { if (gfiles[i] && gfiles[i]->fname && !strcmp(gfiles[i]->fname, fname)) { /* The file's owner must be root or the current user, and the file must be writable */ if ((curr_uid != 0 && gfiles[i]->fuid != curr_uid) || !(gfiles[i]->fflag & WRPERM)) { write_str("rip: ¬ perm\n"); xfree(rbuff); return; } read_max(rbuff, SBSIZE); if (gfiles[i]->fdata) { /* File is not empty -> rewrite the file content */ if (strlen(rbuff) > 0) { gfiles[i]->fdata = xrealloc(gfiles[i]->fdata, strlen(gfiles[i]->fdata) + strlen(rbuff) + 1); /* Remember the extra null byte */ } strcat(gfiles[i]->fdata, rbuff); } else { /* File is empty -> write the content directly */ gfiles[i]->fdata = strdup(rbuff); } xfree(rbuff); return; } }
```

但我们要伪造 `fake_next_size`，`fake_next_size` 中肯定带 `\x00` 截断，同时由于 `off-by-null-chunk` 原本 `size` 低字节为 `0x21`，被改成 `\x00` 后，`fake_next_size` 要放在 `off-by-null-chunk` 的 `+0x500` 处，例如如下结构：

```
off-by-null-chunk：
0 0x500(0x521)
........
........
padding,fake_next_size
padding,padding。
```

其中 `padding` 为 `8` 非零字节。可以注意到 `fake_next_size` 中带 `\x00` 截断，因此我们不能申请到 `0x521` 大小的 chunk。

想过很多种方法，例如利用 `realloc` 去进行一些 trick，例如：

先申请出 `0x521` 大小的 chunk，然后利用 `cmd_wtf` 里的 `strcpy` 往里填入零字节，再利用 `cmd_rip` 来 `realloc` 到适当字节，恰好写进 `fake_next_size`，但是对一个内存大于申请 size 的堆块进行 `realloc`，会 `free` 掉剩余 size 的小 chunk：

例如对 `0x521` chunk 进行 `reallc(ptr,0x410)`，他会 `free` 掉剩余的 `0x100` 字节。这样会破坏掉原来 chunk 的 `size` 头，导致我们不能触发 `off-by-null`。

可见对大于 `0x400` size 的 chunk 去写入一个 `fake_next_size` 是非常困难的。

转换思路呢？可能一开始就想错了，为什么 `off-by-null-chunk` size 一定要大于 `0x400` 呢？他的 size 可以是 `0x321`，只要我们提前填满对应 size 的 `tcache_list` 即可。

而如果 size 小于 `0x400`，我们观察 `cmd_wtf` 函数：

```
void cmd_wtf() { char *fdata = strtok(NULL, delim); if (!fdata) { write_str("wtf: data ∈ ∅\n"); return; } char *fname = strtok(NULL, delim); if (!fname) { write_str("wtf: file ∈ ∅\n"); return; } remove_slash(fname); if (strlen(fname) == 0) { write_str("wtf: file = ∅\n"); return; } /* Special case: flag1 cannot be altered */ if (!strcmp(fname,"flag1")) { write_str("wtf: ¬ perm\n"); return; } for (int i = 1 /* ignore flag1 */; i < FILEMAX; i++) { if (gfiles[i] && gfiles[i]->fname && !strcmp(gfiles[i]->fname, fname)) { /* The file's owner must be root or the current user, and the file must be writable */ if ((curr_uid != 0 && gfiles[i]->fuid != curr_uid) || !(gfiles[i]->fflag & WRPERM)) { write_str("wtf: ¬ perm\n"); return; } if (gfiles[i]->fdata) { /* File is not empty -> rewrite the file content */ if (strlen(fdata) > strlen(gfiles[i]->fdata)) { gfiles[i]->fdata = xrealloc(gfiles[i]->fdata, strlen(fdata) + 1); /* Remember the extra null byte */ } strcpy(gfiles[i]->fdata, fdata); } else { /* File is empty -> write the content directly */ gfiles[i]->fdata = strdup(fdata); } return; } } write_str("wtf: \""); write_str(fname); write_str("\" ∉ ℱ\n"); }
```

其中关键的即为：
