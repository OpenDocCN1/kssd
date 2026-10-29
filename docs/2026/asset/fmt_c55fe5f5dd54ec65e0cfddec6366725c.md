
当 `BFrontendAction` 在处理 `ast` 的时候扫描到编译属性 `__attribute__((section("maps/stackrace")))` 时,就会创建出类型为 `BPF_MAP_TYPE_STACK_TRACE` 的 `bpf` `maps`,这个 `maps` 的 `key` 大小为 `probe_do_sys_open_1_stacks_table_t` 结构的 `key` 大小 `int`, `value` 大小为 `probe_do_sys_open_1_stacks_table_t` 结构的 `leaf` 大小 `struct bpf_stacktrace`。

`struct bpf_stacktrace` 结构定义为:

```
struct bpf_stacktrace {
  u64 ip[BPF_MAX_STACK_DEPTH];
};
```

可以看到它其实就是个 `u64` 数组,事实上每一个 `u64` 都对应着一个调用栈的 `pc` 指针,`bcc` 就是靠这个数据来进行栈回溯的。

当 `ebpf` 程序执行的时候,它会通过 `bpf_perf_event_output()` 往用户空间输出 `bpf_get_stackid()` 的返回值 `long stack_id`,用户空间程序拿到这个 `stack_id` 以后把它作为 `bpf maps` 中的 `key`,调用 `bpf(BPF_MAP_LOOKUP_ELEM)` 得到类型为 `struct bpf_stacktrace` 结构的调用栈数据,然后 `bcc` 就可以按照自己的方式进行符号化了。具体的代码在 `table.py` 的 `class StackTrace` 类中。

打印调用栈的逻辑为:

```
for addr in stack:
    stackstr += '        '
    if Probe.print_address:
        stackstr += ("%16x " % addr)
    symstr = bpf.sym(addr, tgid, show_module=True, show_offset=True)
    stackstr += ('%s\n' % (symstr.decode('utf-8')))
```

其中 `addr` 就是通过 `stack_id` 调用 `bpf(BPF_MAP_LOOKUP_ELEM)` 得到的地址列表。

`bpf_get_stackid()` 是怎么实现的?直接给出结论:通过基于 `fp` 的栈回溯来实现的,感兴趣的话可以查看一下内核的 `perf_callchain_user` 函数,这里就不细述了。

到这里,上面使用 `trace.py` 打印出程序的堆栈信息缺失的原因就揭晓了,基于 `fp` 的栈回溯虽然比较快,但是编译时如果指定了 `"-fomit-frame-pointer"` 就不会生成 `fp` 指针,堆栈信息就会缺失.如果用 `bcc` 分析某些大厂 `app`,很多调用除了第一行以外其他全是 `unknown`。

而对于 `func_a` 调用的堆栈信息符号化缺失是因为 `bcc` 它的符号化是读取 `/proc/pid/maps` 并解析其中的 `so` 的 `elf` 符号表来实现的,由于 `zipalign` 的原因：[4d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3j5$3!0E0L8h3q4F1k6q4)9J5k6r3I4A6L8X3g2Q4x3V1k6*7K9i4m8S2L8r3W2Y4L8W2)9K6c8X3S2D9i4K6y4p5P5X3S2Q4x3X3c8U0L8R3`.`](https://bbs.kanxue.com/elink@d04K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3j5$3!0E0L8h3q4F1k6q4)9J5k6r3I4A6L8X3g2Q4x3V1k6*7K9i4m8S2L8r3W2Y4L8W2)9K6c8X3S2D9i4K6y4p5P5X3S2Q4x3X3c8U0L8R3`.`.)

如果 `apk` 中所有未归档文件相对于文件开头是对齐的,那么 `so` 文件就不会被解压至 `/data/app` 目录下,而是直接在 `apk` 文件中进行 `mmap`。

`bcc` 对 `apk` 的支持不好,所以无法正确处理这种情况.而且 `bcc` 读取 `/proc/pid/maps` 只读取一次,除非整个可执行文件被替换不然 `bcc` 不会重新读取 `/proc/pid/maps`,这也就导致了无法更新到后加载的 `so` 的情况。

## 五.问题的解决

在 `bcc` 的 `issue` 列表中其实已经有人提了这样的问题:

[27cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8%4k6A6M7$3!0J5i4K6u0r3j5X3y4U0i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3x3K6f1I4y4b7`.`](https://bbs.kanxue.com/elink@020K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8%4k6A6M7$3!0J5i4K6u0r3j5X3y4U0i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3x3K6f1I4y4b7`.`)

上面提到了一个工具 `simpleperf`

`trace.py` 无法打印的, `simpleperf` 却可以,这是因为 `simpleperf` 是基于 `dwarf` 的 `.eh_frame` 节进行栈回溯的, 就算 `so` 编译的时候指定了 `"-fomit-frame-pointer"` 也能正确打印出堆栈信息.那么如果将 `simpleperf` 的栈回溯机制和 `bcc` 结合不就可以解决问题了吗?

先来看一下 `simpleperf` 的使用:

`simpleperf` 是 `android` 提供的性能分析工具,它也是通过调用 `perf_event_open` 来实现的,假设我们想追踪 `4665` 这个进程的 `sys_enter_clone` 系统调用.我们在手机上这么执行持续 `10` 秒采样:

```
$ simpleperf record -p 4665 --call-graph dwarf -e syscalls:sys_enter_clone --duration 10 -o /data/local/tmp/perf.data
```

然后将 `/data/local/tmp/perf.data` 拷贝至 `aosp/system/extras/simpleperf/scripts` 目录下,执行 `report_sample.py` 即可看到期间的所有 `sys_enter_clone` 调用以及堆栈。

虽然 `simpleperf` 有时用来做逆向分析的备用工具来使用也凑合,但是它最大的缺陷是无法打印出系统调用的参数,更无法采用一些 `hack` 的技巧来修改系统调用参数,而 `ebpf` 是可以直接执行定制化代码的,因此如果 `bcc` 可以结合 `simpleperf` 的栈回溯机制将是完美的组合。

从 `bcc` 的官方开发人员的回复来看,短时间内应该不会有基于 `dwarf` 的官方实现.而且安卓运行环境比较特殊,感觉 `bcc` 更多的是面向 `linux pc` 领域,只能自己来动手了。

首先我们需要理解一下 `simpleperf` 的原理,对于上面的命令来说,`simpleperf` 其实是构建了以下的 `perf_event_attr`:

```
struct perf_event_attr attr;
attr.size = sizeof(perf_event_attr);
attr.type = PERF_TYPE_TRACEPOINT;
attr.config = id; //cat /sys/kernel/debug/tracing/events/syscalls/sys_enter_openat/id
attr.disabled = 0;
attr.read_format = PERF_FORMAT_TOTAL_TIME_ENABLED | PERF_FORMAT_TOTAL_TIME_RUNNING | PERF_FORMAT_ID;
attr.sample_type |= PERF_SAMPLE_IP | PERF_SAMPLE_TID | PERF_SAMPLE_TIME | PERF_SAMPLE_PERIOD | PERF_SAMPLE_CPU | PERF_SAMPLE_ID | PERF_SAMPLE_RAW;
attr.sample_type |= PERF_SAMPLE_CALLCHAIN | PERF_SAMPLE_REGS_USER | PERF_SAMPLE_STACK_USER;
attr.sample_type &= ~PERF_SAMPLE_BRANCH_STACK;
attr.exclude_callchain_user = 1;
attr.sample_regs_user = ((1ULL << PERF_REG_ARM64_MAX) - 1);
attr.sample_stack_user = 65528;
attr.inherit = 0;
attr.sample_id_all = 1; //SampleIdAll
attr.freq = 0;
attr.sample_period = DEFAULT_SAMPLE_PERIOD_FOR_TRACEPOINT_EVENT; //1
attr.mmap = 1;
attr.comm = 1;
```

可以看到 `attr.sample_type` 指定的类型非常多,其中最重要的类型为 `PERF_SAMPLE_REGS_USER | PERF_SAMPLE_STACK_USER`

看一下 `perf` 对 `dwarf` 栈回溯支持的提交文件:

[a41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8U0f1H3y4K6M7#2x3#2)9J5c8R3`.`](https://bbs.kanxue.com/elink@ca4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8U0f1H3y4K6M7#2x3#2)9J5c8R3`.`)

[[b83K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8%4u0W2i4K6u0W2K9$3g2J5L8X3g2D9i4K6u0W2L8%4u0Y4i4K6u0r3j5h3I4D9i4K6u0r3x3e0x3@1x3K6x3&6x3e0R3K6y4q4)9J5k6o6p5H3z5o6f1I4i4K6u0V1y4#2)9J5k6r3N6A6N6q4)9J5k6s2y4W2L8X3c8Q4x3X3c8W2L8h3q4A6L8q4)9J5k6r3A6G2L8s2y4S2i4K6b7H3M7X3g2V1K9r3q4@1i4K6u0W2j5$3!0E0i4K6u0r3](https://bbs.kanxue.com/elink@e8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8%4u0W2i4K6u0W2K9$3g2J5L8X3g2D9i4K6u0W2L8%4u0Y4i4K6u0r3j5h3I4D9i4K6u0r3x3e0x3@1x3K6x3&6x3e0R3K6y4q4)9J5k6o6p5H3z5o6f1I4i4K6u0V1y4#2)9J5k6r3N6A6N6q4)9J5k6s2y4W2L8X3c8Q4x3X3c8W2L8h3q4A6L8q4)9J5k6r3A6G2L8s2y4S2i4K6b7H3M7X3g2V1K9r3q4@1i4K6u0W2j5$3!0E0i4K6u0r3)]

1. `PERF_SAMPLE_REGS_USER` 用于指示内核将用户空间发生事件时(这里为系统调用)的寄存器信息以 `PERF_RECORD_SAMPLE` 记录类型传送给用户空间。

2. `PERF_SAMPLE_STACK_USER` 用于指示内核将用户空间发生事件时(这里为系统调用)的栈空间片段数据以 `PERF_RECORD_SAMPLE` 记录类型传送给用户空间。

`simpleperf` 有了这两个信息以后再加上一个目标进程的 `maps` 信息即可指示 `libunwindstack` 库进行 `dwarf` 的栈回溯:
