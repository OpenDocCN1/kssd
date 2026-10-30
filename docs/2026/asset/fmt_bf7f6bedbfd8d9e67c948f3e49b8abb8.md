
将新的测试用例插入队列，并初始化`fname`文件名称，增加`cur_depth`深度++ `queued_paths`测试用例数量++，`pending_not_fuzzed`没被fuzzed测试用例数量++，更新`last_path_time = get_cur_time()`

```struct queue_entry {

u8* fname;        /* File name for the test case    文件名  */

u32 len;          /* Input length                   testcase大小 */



u8  cal_failed,   /* Calibration failed?    校准失败？          */

    trim_done,    /* Trimmed?              该testcase是否被修剪过      */

    was_fuzzed,   /* historical, but needed for MOpt  */

    passed_det,   /* Deterministic stages passed?     */

    has_new_cov,  /* Triggers new coverage?           */

    var_behavior, /* Variable behavior?               */

    favored,      /* Currently favored?          当前是否被标记位favored(更多的fuzz机会)*/

    fs_redundant; /* Marked as redundant in the fs?   */



u32 bitmap_size,  /* Number of bits set in bitmap     bitmap中bit的数量*/

    fuzz_level,   /* Number of fuzzing iterations     */

    exec_cksum;   /* Checksum of the execution trace  trace_bits的checksum*/



u64 exec_us,      /* Execution time (us)              执行时间延迟*/

    handicap,     /* Number of queue cycles behind    */

    n_fuzz,       /* Number of fuzz, does not overflow */

    depth;        /* Path depth                       路径深度*/



u8* trace_mini;   /* Trace bytes, if kept  1个bit存一个byte的trace_mini     */
u32 tc_ref;       /* Trace bytes ref count  top_rate[]中该testcase入选的次数 */



struct queue_entry *next,  /* Next element, if any             */
                   *next_100;  /* 100 elements ahead    */
};
```

---

### 1.20 load_auto自动生成附加负载

`static void load_auto(void)`该函数有点不太明白，可能是将一些自己定义的规则token添加`extra_data`数组中。结构体如下：

调用函数1.21 `maybe_add_auto`将testcase添加到数组中

---

### 1.21 maybe_add_auto 添加token的函数

`static void maybe_add_auto(u8* mem, u32 len)`

作用：该函数会将传入的token添加到数组中，如果数组还有空间则，添加进来。没有的话那就在数组的下半部分随机删除一个token，然后将新的添加进来。数组最大`MAX_AUTO_EXTRAS` 50x10=500个。影响以下队列`extra[]`结构体变量，`a_extras_cnt` 当前token总数量++（添加成功的话）。

```struct extra_data {

  u8* data;     /* Dictionary token data    */
  u32 len;      /* Dictionary token length  */
  u32 hit_cnt;  /* Use count in the corpus  */
};
```

---

### 1.22 pivot_inputs

`static void pivot_inputs(void)`在输出目录中为输入测试用例创建硬链接，选择好名称并相应地旋转。

使用函数`link_or_copy`重新命名并且拷贝；使用函数`mark_as_det_done`为已经经过确定性变异（deterministic）阶段的testcase文件放入deterministic_done文件夹。这样经过deterministic的testcase就不用浪费时间进行重复。

---

### 1.23 load_extras 依然是加载token

`static void load_extras(u8* dir)`从extras目录中读取extras并按大小排序

作用：如果有token的目录，则将目录下的token加载到extra队列中。

其中函数`load_extras_file`从文件中加载extra_file并且排序，将token添加到extra数组中

影响参数和1.20 `load_auto`差不多。

---

### 1.24 find_timeout 超时函数

`static void find_timeout(void)`如果有`-t`的设置了自己的超时，那么会触发这个函数。

---

### 1.25 setup_stdio_file 设置文件输出目录

前面同样也有一系列判断输出路径。

`EXP_ST void setup_stdio_file(void)`如果没有使用`-f`，则为fuzzed data设置输出目录。

---

### 1.26 check_binary 检查二进制文件

`void check_binary(u8* fname)` 搜索路径，找到目标二进制文件，检查文件是否存在，是否为shell脚本，同时检查ELF头以及程序是否被插桩。

---

## 2 开始第一遍fuzz--dry run

---

### 2.1 检查

用`start_time=get_cur_time()` 获取开始时间；

检查是不是`QEMU_MODE`

### 2.2 ★★perform_dry_run

AFL关键函数：`static void perform_dry_run(char** argv)`

作用：执行input文件夹下的预先准备的所有testcase（perform_dry_run），生成初始化的queue和bitmap。这只对初始输入执行一次，所以叫：dry run。

1. 第一个是个while循环，遍历之前生成的input_queue 也就是queue链表。该`while(q) loop` 的前面，准备工作：从队列中取出`q->fname` 读取该文件`q->len` 大小到`use_mem` 中，关闭`fd`

2. 接着调用`calibrate_case`函数对该case进行校准。该函数内调用方式`res = calibrate_case(argv, q, use_mem, 0, 1);`

3. 根据校准的返回值`res` ，查看是哪种错误并进行判断。一共有一下几种错误类型。

```
enum {

/* 00 */ FAULT_NONE,

/* 01 */ FAULT_TMOUT,

/* 02 */ FAULT_CRASH,

/* 03 */ FAULT_ERROR,

/* 04 */ FAULT_NOINST,

/* 05 */ FAULT_NOBITS  };
```

4. 打印一些错误信息，退出函数

---

### 2.3 ★★calibratecase 校准testcase

AFL关键函数:`calibrate_case(char** argv, struct queue_entry* q, u8* use_mem,u32 handicap, u8 from_queue)`校准一个新的测试用例。这是在处理输入目录时完成的，以便在早期就警告有问题的测试用例;当发现新的路径来检测变量行为等等。

这个函数是AFL的重点函数之一，在`perform_dry_run，save_if_interesting，fuzz_one，pilot_fuzzing,core_fuzzing`函数中均有调用。

**步骤：**

1. 进行一系列参数设置，包括当前阶段`stage_cur`，阶段名称`stage_name`，新比特`new_bit`等初始化设置。

2. 最后一个参数`from_queue`，判断是否是为队列中的||刚恢复fuzz 以此设置较长的时间延迟。testcase参数`q->cal_failed++` 是否校准失败参数++

3. 判断是否已经启动forkserver ,调用函数`init_forkserver()`启动fork服务。如果是第一次接触linux
中fork()函数，不妨看一下 69bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0F1k6$3N6#2L8$3I4W2K9g2)9J5c8Y4m8Q4x3V1j5^5x3o6R3$3x3K6b7$3i4K6u0W2K9s2c8E0L8l9`.`. 该函数的理解，`init_forkserver()`的详细内容见2.4

4. 拷贝`trace_bits`到`first_trace`,并获取开始时间`start_us`；

5. -loop- 该loop循环多次执行这个testcase，循环的次数 8次或者3次，取决于是否快速校准。*对同一个初始testcase多次运行的意义可能是，觉得有些targetApp执行同一个testcase可能也会出现不同的路径（这是我的猜测*）

6. `static void write_to_testcase(void* mem, u32 len)` 将修改后的数据写入文件进行测试。如果`use_stdin`被清除了，那么取消旧文件链接并创建一个新文件。否则，`prog_in_fd`将被缩短。将testcase写入到文件中去。该函数较简单，不做单独解释。

7. `run_target`详细内容见2.5 主要作用是通知forkserver可以开始fork并且fuzz了。

8. `cksum = hash32(trace_bits, MAP_SIZE, HASH_CONST)`校验此次运行的`trace_bits`，检查是否出现新的情况。`hash32`函数较为简单，不多做分析。

9. 这段代码的主要意思是先用`cksum`也就是本次运行的出现`trace_bits`哈希和本次testcase `q->exec_cksum`对比。如果发现不同，则调用`has_new_bits`函数(见2.6)和我们的总表`virgin_bits` 对比。

10. 判断`q->exec_cksum` 是否为0，不为0那说明不是第一次执行。后面运行的时候如果，和前面第一次`trace_bits`结果不同，则需要多运行几次。这里把校准次数设为40...

11. -loop-end-

12. 接着收集一些关于这个测试用例性能的统计数据。比如执行时间延迟，校准错误？，bitmap大小等等。

13. `update_bitmap_score(q)` 2.7 对这个测试用例的每一个byte进行排序，用一个`top_rate[]`来维护它的最佳入口。维护完成之后，我们这个函数在

14. 如果这种情况没有从检测中得到`new_bit`，则告诉父程序。这是一个无关紧要的问题，但是需要提醒用户注意。

**总结：**`calibratecase`函数到此为止，该函数主要用途是`init_forkserver`；将testcase运行多次；用`update_bitmap_score`进行初始的byte排序。

---

### 2.4 ★★ init_forkserver()

`static void init_forkserver(char **argv)`启动APP和它的forkserver；

```
//检查输入输出管道是否存在。
if (pipe(st_pipe) || pipe(ctl_pipe)) PFATAL("pipe() failed");

//pid=0的话就是fork出来的子进程；！=0的话就是父进程 ,＜0就是fork失败了
forksrv_pid = fork();

//fork失败就打印关键词弹出失败并退出  （这里是在干什么？）
if (forksrv_pid < 0) PFATAL("fork() failed");

//如果是子进程的话，就执行下面的：
if (!forksrv_pid) {
  struct rlimit r;
  //dup2函数：复制一个文件的描述符。它们经常用来重定向进程的stdin、stdout和stderr
  if (dup2(use_stdin ? prog_in_fd : dev_null_fd, 0) < 0 ||
      dup2(dev_null_fd, 1) < 0 ||
      dup2(dev_null_fd, 2) < 0) {
    *(u32*)trace_bits = EXEC_FAIL_SIG;
    PFATAL("dup2() failed");
  }
  close(dev_null_fd);
  close(prog_in_fd);
 //该函数用于创建一个守护进程。在这个程序里面意思就是让该进程成为这个进程组的组长。
  setsid();
  ...
//如果成功，执行execv用以开始执行程序，这个函数除非出错不然不会返回。
 execv(target_path, argv);
//此处使用一个EXEC_FAIL_SIG 来告诉父进程执行失败。
*(u32*)trace_bits = EXEC_FAIL_SIG;
    exit(0);
}
...
//fuzzer从server中读取了四个字节的hello ，那么forkserver程序就设置成功了，如果没有，接下来的代码就是检查错误。
if (rlen == 4) {//判断读取是否成功
    OKF("All right - fork server is up.");
    return;}
```

fuzzer程序则等待forkprocess程序`waitpid(forksrv_pid, &status, 0) <= 0`函数返回。

**疑问：**fuzzer程序fork了一个进程，然后这个进程execv targetBinary ，targetBinary中也启动的了fork（），相当于fuzzer程序是实际被fork程序的祖祖父进程。Fuzzer->Fuzzer forkpoc()->execv targetApp->targetApp forkpoc。然后两个之间使用pipe进行通讯？

**解答：**我本来是学Windows的，以为execv()函数类似于windows中的createprocess，但实际上完全不一样。execv()函数执行之后永远不会返回(除非出错)，且运行程序会替换当前进程，pid不变。这样fuzzer先fork出来，然后再调用execv()函数，也就是说子进程成为了targetApp。这样下面的一系列代码就能说的通了。

**实际流程：**fuzzer(调用init_forkserver)->fork fueezer--targetApp(forkserver) call fork ->targetAppfork(实际fuzz的程序)。fuzzer还是实际fuzzer程序的祖父进程。这样接下来的代码就很容易理解了。

![forkserver设计](https://bbs.kanxue.com/upload/attach/201909/760932_S4K5TSQ8M643XMM.png)

下图可以很好的总结forkserver和被fuzzed程序之间的状态。

![forkserver设计](https://bbs.kanxue.com/upload/attach/201909/760932_JK7M8CRFBTCZAH7.png)

> 首先alf-fuzz会创建两个管道(init_forkserver())，然后会去执行afl-gcc编译出来的目标程序。结合之前的分析，目标程序的main函数位置已经被插桩，程序的控制流会交到_afl_maybe_log手中。如fuzz是第一次运行，则此时的程序便成为了fuzz server，之后运行的目标程序都是由该server fork出来的子进程。fuzz进行的时候，fuzz server会一直fork()新的进程，并且将子进程的结束状态通过pipe传递给afl-fuzz。
> 这里有几点需要注意：afl在这里利用了fork()的特性(creates a new process by duplicating the calling process)来实现目标程序反复执行。实际的fuzz server(_afl_maybe_log)由afl事先插桩在目标程序中，在进入main函数之前，fuzz server便会fork()新的进程，进行fuzz。
