
### 本函数用来初始化 `u16 count_class_lookup16[ 65536 ]` 这个数组。

- 将整个 `count_class_lookup16` 分成 `256` 段，每一段 `256` 份儿。初始化的时候利用了 `count_class_lookup8`。
- `count_class_lookup8` 中对于执行次数进行了规整，比如执行了 `4`-`7` 次的其计数为 `8`，比如 `32` 次到 `127` 次都会认为是 `64` 次。
- 变量 `trace_bits` 来记录分支执行次数，而 `count_class_lookup8` 实际就是对于 `trace_bits` 的规整。
- 而初始化 `count_class_lookup16` 实际是因为 AFL 中对于一条分支径的表示是由一个二元组来表示的。
- 例如： `A -->B-->C-->D-->A--> B` ，可以用 `[A,B]` `[B,C]` `[C,D]` `[D,A]` 四个二元组表示，只需要记录跳转的源地址和目标地址。并且 `[A,B]` 执行了两次，其余执行了一次，这里用 `hash` 映射在一张 `map` 中。
- 而基于这种二元组的表示的效率考虑，又使用了 `u16 count_class_lookup16[ 65536 ]` 这个数组，并在此初始化。

在网上找到这样一段解释我觉得很好：
*这样处理之后，对分支执行次数就会有一个简单的归类。例如，如果对某个测试用例处理时，分支A执行了 `32` 次；对另外一个测试用例，分支A执行了 `33` 次，那么AFL就会认为这两次的 **代码覆盖** 是相同的。当然，这样的简单分类肯定不能区分所有的情况，不过在某种程度上，处理了一些因为循环次数的微小区别，而误判为不同执行结果的情况。*

### setup_dirs_fds()

`https://647K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3@1`. `/xiaofeiIDO/p/6695459.html`

设置输出的文件夹与fd。
- 如果设置了 `sync_id` 且通过 `mkdir(sync_dir, 0700)` 创建了对应的文件夹失败，并且 `errno != EEXIST` 时，`abort`
- 如果没设置 `sync_id`，尝试创建了 `sync_dir`，创建失败，且 `errno != EEXIST` 时 `abort`。
- 如果创建 `sync_dir` 失败，且 `errno = EEXIST`。调用 `maybe_delete_out_dir()` 收尾。
- 如果创建成功。
- 设置了 `in_place_resume`，则 `abort`
- 否则调用 `open` 以读权限打开，返回 `fd`，赋值给 `out_dir_fd`。
- 接下来将 `out_dir` 与 `/queue` 拼接为 `tmp`。（`output/queue`）
- 创建 `tmp` 文件夹。
- 创建 `output/queue/.state/` （Top-level directory for queue metadata used for session resume and related tasks）
- 创建 `output/queue/.state/deterministic_done/` （Directory for flagging queue entries that went through deterministic fuzzing in the past.）
- 创建 `output/queue/.state/auto_extras/` （Directory with the auto-selected dictionary entries）
- 创建 `output/queue/.state/redundant_edges/` （The set of paths currently deemed redundant）
- 创建 `output/queue/.state/variable_behavior/` （The set of paths showing variable behavior）
- 接下来做目录的同步，用于追踪共同工作的 `fuzzer`
- 如果设置了 `sync_id`。那么创建 `output/.synced/`
- **创建 `output/crashes`，记录 `crash` 样本。**（All recorded crashes）
- 创建 `output/hangs`（ All recorded hangs.）
- 打开 `/dev/null` 与 `/dev/urandom`，`fd` 为 `dev_null_fd` 和 `dev_urandom_fd`。
- 创建 `output/plot_data`，通过 `open(tmp, O_WRONLY | O_CREAT | O_EXCL, 0600)` 打开返回 `fd`。
- 通过 `fdopen` 将 `fd` 转为文件指针 `plot_file`，通过 `fprintf` 将一些信息写入文件。忽略错误。

```c
fprintf(plot_file,"# unix_time, cycles_done, cur_path, paths_total, "
"pending_total, pending_favs, map_size, unique_crashes, "
"unique_hangs, max_depth, execs_per_sec\n");
```

**注意，以上的操作除了特殊说明，如果创建，打开失败，均 `abort`**

### read_testcases()

本函数从输入（input）读取测试用例并入队，在启动时调用。
在函数开头首先定义了

```
struct dirent **nl;
s32 nl_cnt;
```

而 `struct dirent` 如下：

```
// Data structure for a directory entry
struct dirent
{
  long d_ino; /* inode number 索引节点号 */
  off_t d_off; /* offset to this dirent 在目录文件中的偏移 */
  unsigned short d_reclen; /* length of this d_name 文件名长 */
  unsigned char d_type; /* the type of d_name 文件类型 */
  char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长255字符 */
}
```

- 首先测试 `iutput/queue` 是否可访问。如果可访问，那么将 `in_dir` 赋值为 `iutput/queue`。
- 接下来通过调用 `scandir(in_dir, &nl, NULL, alphasort)` 扫描对应的 `in_dir` 目录。
  - 函数原型：`int scandir(const char *dir, struct dirent **namelist, nt (*select) (const struct dirent *), nt(*compar) (const struct dirent **, const struct dirent**));`
  - 第一参数 `dir` 代表要扫描的路径。
  - 第三参数 `select` 适用于过滤的函数，对于第一参数指定的进行过滤
  - 第四参数 `compar` 将过滤后的结果进行排序。
  - 最终储存在第二参数 `namelist` 列表中。每一项都是一个 `dirent` 。
  - 那么本函数在这里的作用就是扫描目录 `in_dir` 中的每一项，将结果排序后放入 `nl` 这一列表中。
  - 返回值为函数成功执行时返回找到匹配模式文件的个数，存储在 `nl_cnt` 中。
    - 此时如果 `nl_cnt<0` 说明匹配失败，`errno` 若是 `ENOENT`、`ENOTDIR`，则输出路径无效提示信息后 `abort`。否则直接 `abort`
- 当匹配成功后，若设置了 `shuffle_queue`，那么调用 `shuffle_ptrs` 对此时的 `input-qeue` 进行改组（Shuffle）
- 接下来进入一个 `for` 循环：`for (i = 0; i < nl_cnt; i++)` 扫描数组 `nl` 中的每一项。
  - 首先将扫描到的文件名（`nl[i]->d_name`）与 `in_dir` 拼接形成路径 `fn` 。（`in_dir`, `nl[i]->d_name`）
  - 接下来将（`in_dir`, `"/.state/deterministic_done/"`, `nl[i]->d_name`）拼接形成 `dfn`。
  - 设置 `passed_det = 0`
  - 随后通过 `lstat(fn, &st) || access(fn, R_OK)` 检测目录是否可访问。不可访问则 `abort`。将从 `lstat(fn, &st)` 获取的文件信息保存在 `struct stat st;` 中。
  - 下一步检查我们扫描到的文件是否是有效的常规文件？
    - 通过 `S_ISREG` 查看 `st.st_mode` 确定文件的性质；`st.st_size` 确定大小；`strstr` 查找是否是 `README.txt` 文件。
      - 这一步主要是排除一些无关文件的干扰，比如 `README.txt`、`.` 和 `..`
  - 若是有效的常规文件，那么检测大小是否超界限。
  - 接下来通过 `if (!access(dfn, F_OK))` 检测 `dfn` 判断此文件是否已经 `fuzz` 过了，防止多余的时间消耗。（因为若 `fuzz` 过了，会被放到 `in_dir/.state/deterministic_done/` 下）
    - 如果没有 `fuzz` 过，设置 `passed_det = 1`。
    - 如果 `fuzz` 过了，保持 `passed_det = 0`。
  - 至此，一切检查完成，我们将调用 `add_to_queue(fn, st.st_size, passed_det)` 将这个文件入队。同时也带上了他的大小和 `passed_det` 信息。
- `for` 循环结束后，到达函数最后收尾的位置。首先检查是否设置 `queued_paths`，没设置则代表没有测试用例。`abort` 掉。
- 通过了 `queued_paths` 后，设置 `last_path_time = 0`、以及 `queued_at_start = queued_paths`。
  至此，整个 `read_testcases()` 结束。

### shuffle_ptrs()

进行队列改组。
调用：`shuffle_ptrs((void **) nl, nl_cnt)`

```c
/* Shuffle an array of pointers. Might be slightly biased. */
static void shuffle_ptrs(void **ptrs, u32 cnt) {
  u32 i;
  for (i = 0; i < cnt - 2; i++) {
    u32 j = i + UR(cnt - i);
    void *s = ptrs[i];
    ptrs[i] = ptrs[j];
    ptrs[j] = s;
  }
}
```

其中 `UR` 是产生一个 `0 - (limit-1)` 的随机数。
在 `i` 到 `cnt-1` 内随机产生一个索引下标 `j`。将数组元素 `nl[i]` 与 `nl[j]` 位置互换。
个人理解是对于 `input-qeue` 做了一个随机化 `or` 扰动。

### load_auto()

- 首先通过 `alloc_printf` 拼接出形如：`("%s/.state/auto_extras/auto_%06u", in_dir, i)` 的路径，赋值给 `fn` 。如：`input/.state/auto_extras/auto_000000`
- 接下来进入一个 `50` 次的 `for` 循环
  - 接着以 `0600` 权限打开 `fn` 对应的文件，返回 `fd`。如果 `open` 失败则 `abort` 掉。
  - 从 `fd` 读取 `MAX_AUTO_EXTRA + 1` 大小的 `bytes`，放入 `tmp` 数组中。`MAX_AUTO_EXTRA + 1 = 33`。注意这里多读取了一个字节来判断是否读取的 `token` 过长。`read` 的返回值为 `len`。
    - 如果读取失败则 `abort`
  - 如果 `len` 在：`[ MIN_AUTO_EXTRA, MAX_AUTO_EXTRA ]` 之间的话，调用 `maybe_add_auto(tmp, len)`，将我们此时的语料加入 `a_extras[]` 数组中。
  - 关闭 `fd`。
- 如果 `for` 循环正常进行了，输出信息：`OKF("Loaded %u auto-discovered dictionary tokens.", i)`
- 否则输出相反的信息。

### maybe_add_auto(u8 *mem, u32 len)

`maybe_add_auto(tmp, len)`，此时 `tmp(mem)` 对应的是读取的 `auto extra` 文件，`len` 对应长度。

```c
/* Interesting values, as per config.h */
static s8 interesting_8[] = {INTERESTING_8};
static s16 interesting_16[] = {INTERESTING_8, INTERESTING_16};
static s32 interesting_32[] = {INTERESTING_8, INTERESTING_16, INTERESTING_32};

static struct extra_data *extras; /* Extra tokens to fuzz with */
static u32 extras_cnt; /* Total number of tokens read */
static struct extra_data *a_extras; /* Automatically selected extras */
static u32 a_extras_cnt; /* Total number of tokens available */
```

- 如果 `MAX_AUTO_EXTRAS` 和 `USE_AUTO_EXTRAS` 没有设置的话直接 `return`。
  - 前者代表最大的 `auto_extras token` 数量，值为 `USE_AUTO_EXTRAS * 10`；后者代表用户指定的 `auto_extras token` 数量，默认 `50`。
- 接下来从 `mem[1]` 开始扫描，跳过与 `mem[0]` 相同的 `byte`。最后索引 `i` 停止在第一个与 `mem[0]` 不相同的位置。
- 如果 `i==len`，代表所有的 `mem` 中的 `byte` 都相同，那么 `return`。
- 如果 `len==2`：与 `interesting_8[]` 中每一项比较，如果相等就 `return`
- 如果 `len==4`：与 `interesting_16[]` 中每一项比较，如果相等就 `return`
- 接下来与 `extras[]` 数组中已经存在的 `extras` 相比，如果相等就 `return`。（样例敏感）
- 设置 `auto_changed=1`
- 扫描 `a_extras[]` 数组，如果遇到和 `mem` 相等的，那么 `a_extras[i].hit_cnt++` 命中计数 `+1`，跳转到 `sort_a_extras:`
  - 首先，按使用次数，降序对所有 `a_extras[i]` 进行排序。
  - 然后，按大 `size` 对 `a_extras` 中前 `USE_AUTO_EXTRAS` 个进行排序。
- 如果当前的 `a_extras_cnt < MAX_AUTO_EXTRAS`，则说明 `a_extras` 还没填满。
  - 此时我们首先用 `realloc` 调整 `a_extras_cnt` 的空间为 `a_extras_cnt + 1`。
  - 将 `a_extras[a_extras_cnt].data` 赋值为 `mem`，`.len=len`，然后 `a_extras_cnt++`，相当于加了新的一项。
  - 如果已经填满了，那么取一个随机数 `i`，将 `a_extras[i]` 换成 `mem`。

### pivot_inputs()

首先定义：

```c
struct queue_entry *q = queue; // 指向 input case queue 队头
u32 id = 0;
#ifndef SIMPLE_FILES
#  define CASE_PREFIX "id:"
#else
#  define CASE_PREFIX "id_"
#endif /* ^!SIMPLE_FILES */
```

- 接下来进入一个遍历 `input` 队列的大 `While` 循环：`while (q)`
  - 首先通过 `*rsl = strrchr(q->fname, '/')` 扫描 `queue` 中第一个元素的 `fname`，并从取出最后一个 `'/'` 以及其后的内容。
  - 接下来判断是否取成功。
    - 如果成功，则 `rsl++` 跳过 `'/'`
    - 如果失败，说明没有 `'/'`，直接令 `rsl = q->fname`
  - 如果 `rsl` 以 `CASE_PREFIX` 开头，尝试将 `CASE_PREFIX` 后的数字以 `"%06u"` 格式化后存入 `orig_id`。
    - 若此时的 `orig_id` 与 `id` 相等，设置 `resuming_fuzz = 1`。使用 `alloc_printf("%s/queue/%s", out_dir, rsl)`，拼接产生 `nfn`。接下来使用 `strchr(rsl + 3, ':')` 跳过 `CASE_PREFIX` 查找下一个 `":"` 的位置，存储在 `src_str` 中。
    - 如果 `src_str` 存在，用 `sscanf` 将其后的数字以 `"%06u"` 格式化后存入 `src_id`。
      - 接下来让指针 `s` 从队头开始扫描，每扫描过一个元素，`src_id--`；`s` 后移。若扫描结束后 `s` 还没有移动到队尾（此时 `src_id==0`），那么令队列深度为 `s` 的位置 `+1`。`q->depth = s->depth + 1`
      - 然后判断队列深度是否超过最大深度，如果超过最大深度，则指定为最大深度。`if (max_depth < q->depth) max_depth = q->depth`
  - 如果不以 `CASE_PREFIX` 开头。
    - 当没有定义 `SIMPLE_FILES` 时。（非单文件）
      - 检测 `rsl` 是否有 `,orig:` 为前缀的子串，如果是的话跳过前缀；如果不是直接令 `use_name = rsl`
      - 然后通过 `nfn = alloc_printf("%s/queue/id:%06u,orig:%s", out_dir, id, use_name)` 拼接产生 `nfn`。如：`"output/queue/id:000000,orig:a.out"`
    - 当定义了 `SIMPLE_FILES` 时（单文件）
      - 直接拼接 `nfn = alloc_printf("%s/queue/id_%06u", out_dir, id)`，并且不考虑 `id` 了。
  - 至此 `nfn` 获取完毕，这个 `nfn` 个人理解实际就是当前队列中测试用例的一个编号。
  - 调用 `link_or_copy(q->fname, nfn)` 创建硬链接（`q->fname` 到 `nfn`）并将 `input/a.out` 文件中的内容写入 `output/queue/id:000000,orig:a.out`（以我这里为例）
  - 重新对队列中这一元素的 `fname` 赋值： `q->fname = nfn`
  - 如果设置了 `q->passed_det=1` ，那么调用 `mark_as_det_done(q)` 标记 `queue` 这一项已经 `fuzz` 过了，并保持 `q->passed_det=1`
  - 接下来 `q` 指针后移。`id++`。
- 遍历结束后检测是否设置 `in_place_resume`，若设置了调用 `nuke_resume_dir()` 删除 `output/_resume/*` 临时目录。这个目录主要用于本地临时恢复。

### link_or_copy(u8 *old_path, u8* new_path)

调用如下：

```c
link_or_copy(q->fname, nfn)
// q->fname: "input/a.out"
// nfn: "output/queue/id:000000,orig:a.out"
```

- 首先调用 `link(old_path, new_path)` 将 `new_path` 简历为一个 `old_path` 的硬链接。
  `i = link(old_path, new_path)`
- 尝试以只读打开 `old_path`：`sfd = open(old_path, O_RDONLY)`
- 尝试以只读 `+0600` 权限打开 `new_path`，同时使用 `O_CREAT | O_EXCL` 测试文件是否存在。打开成功返回 `dfd`
- 接着分配 `64k` 的空间为缓冲区，通过缓冲区中转，将 `old_path` 中的内容写入 `new_path`。

```c
while ((i = read(sfd, tmp, 64 * 1024)) > 0)
  ck_write(dfd, tmp, i, new_path);
```

### mark_as_det_done(struct queue_entry *q)

/* 
Mark deterministic checks as done for a particular queue entry. We use the
.state file to avoid repeating deterministic fuzzing when resuming aborted
scans.
*/
```c
static void mark_as_det_done(struct queue_entry *q) {
  u8 *fn = strrchr(q->fname,'/');
  s32 fd;
  fn = alloc_printf("%s/queue/.state/deterministic_done/%s", out_dir, fn + 1);
  fd = open(fn, O_WRONLY | O_CREAT | O_EXCL, 0600); // 打开 fn 对应的文件，若没有则创建。
  if (fd < 0) PFATAL("Unable to create '%s'", fn);
  close(fd);
  ck_free(fn);
  q->passed_det = 1; // 设置 queue 中这一项的 passed_det=1 代表已经 fuzz 过了
}
```

### load_extras(u8 *dir)

首先定义：

```c
DIR *d;
struct dirent *de;
u32 min_len = MAX_DICT_FILE, max_len = 0, dict_level = 0;
u8 *x;
```
