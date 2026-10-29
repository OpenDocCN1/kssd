
`void qemu_init(int argc, char **argv, char **envp)` `{` `//...` `// 对参数进行解析` `for(;;) {` `if (optind >= argc)` `break;` `if (argv[optind][0] != '-') {` `loc_set_cmdline(argv, optind, 1);` `drive_add(IF_DEFAULT, 0, argv[optind++], HD_OPTS);` `}` `else {` `const QEMUOption *popt;` `popt = lookup_opt(argc, argv, &optarg, &optind);` `if (!(popt->arch_mask & arch_type)) {` `error_report("Option not supported for this target");` `exit(1);` `}` `switch(popt->index) {` `case QEMU_OPTION_cpu:` `/* hw initialization will check this */` `cpu_option = optarg;` `break;` `//...` `// 主要关注下面几个参数` `case QEMU_OPTION_m:` `opts = qemu_opts_parse_noisily(qemu_find_opts("memory"), optarg, true);` `if (!opts) {` `exit(EXIT_FAILURE);` `}` `break;` `case QEMU_OPTION_enable_kvm:` `qdict_put_str(machine_opts_dict, "accel", "kvm");` `break;` `case QEMU_OPTION_M:` `case QEMU_OPTION_machine:` `{` `bool help;` `keyval_parse_into(machine_opts_dict, optarg, "type", &help, &error_fatal);` `if (help) {` `machine_help_func(machine_opts_dict);` `exit(EXIT_SUCCESS);` `}` `break;` `}` `case QEMU_OPTION_smp:` `machine_parse_property_opt(qemu_find_opts("smp-opts"), "smp", optarg);` `break;` `}` `}` `}` `//...` `// 根据accel设置accelerators = kvm` `qemu_apply_legacy_machine_options(machine_opts_dict);` `qemu_apply_machine_options(machine_opts_dict);` `// 会根据进程名判断可用的加速类型` `configure_accelerators(argv[0]);` `// 内部调用了do_configure_accelerator --> accel_init_machine` `// accel_init_machine --> kvm_init` `// 初始化具体的accel类（这里是kvm）` `// 在qemu-6.2.0/accel/kvm/kvm-all.c line 3629` `// 函数kvm_accel_class_init内部找到真正的初始化函数` `// ac->init_machine = kvm_init;` `//...` `// 在qmp_x_exit_preconfig与虚拟cpu创建有关` `if (!preconfig_requested) {` `qmp_x_exit_preconfig(&error_fatal);` `}` `qemu_init_displays();` `// 设置accel` `accel_setup_post(current_machine);` `os_setup_post();` `resume_mux_open();` `}`

| 参数 | 描述 |
| --- | --- |
| `QEMU_OPTION_m` | 虚拟机内存大小 |
| `QEMU_OPTION_enable_kvm` | 启用kvm加速 |
| `QEMU_OPTION_machine` | 虚拟机机器类型 |
| `QEMU_OPTION_smp` | 虚拟机cpu数量 |

### 2.2 虚拟机创建

`qemu-6.2.0/accel/kvm/kvm-all.c` line 2306

在`kvm-all.c`文件2306行是`kvm_init`

调用栈：

|  |  |
| --- | --- |
| `kvm_init(MachineState * ms) (qemu-6.2.0\accel\kvm\kvm-all.c:2308)` |  |
| `accel_init_machine(AccelState * accel, MachineState * ms) (qemu-6.2.0\accel\accel-softmmu.c:39)` |  |
| `do_configure_accelerator(void * opaque, QemuOpts * opts, Error ** errp) (qemu-6.2.0\softmmu\vl.c:2348)` |  |
| `qemu_opts_foreach(QemuOptsList * list, qemu_opts_loopfunc func, void * opaque, Error ** errp) (qemu-6.2.0\util\qemu-option.c:1135)` |  |
| `configure_accelerators(const char * progname) (qemu-6.2.0\softmmu\vl.c:2414)` |  |
| `qemu_init(int argc, char ** argv, char ** envp) (qemu-6.2.0\softmmu\vl.c:3724)` |  |
| `main(int argc, char ** argv, char ** envp) (qemu-6.2.0\softmmu\main.c:49)` |  |

主要函数`kvm_init`

```
static int kvm_init(MachineState *ms)
{
MachineClass *mc = MACHINE_GET_CLASS(ms);
static const char upgrade_note[] =
"Please upgrade to at least kernel 2.6.29 or recent kvm-kmod\n"
"(see 423K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4G2N6i4u0U0k6h3k6G2M7X3N6W2i4K6u0W2L8X3g2@1i4K6u0r3M7s2u0G2K9X3g2U0N6s2y4Q4x3V1k6C8N6X3#2Q4x3U0W2Q4x3X3g2Q4y4f1y4F1";
//...
QLIST_INIT(&s->kvm_parked_vcpus);
// 开始使用kvm之前的标准流程
// 打开设备/dev/kvm，检查kvm API版本
// 保存了kvm设备描述符s->fd
s->fd = qemu_open_old("/dev/kvm", O_RDWR);
if (s->fd == -1) {
fprintf(stderr, "Could not access KVM kernel module: %m\n");
ret = -errno;
goto err;
}
ret = kvm_ioctl(s, KVM_GET_API_VERSION, 0);
if (ret < KVM_API_VERSION) {
if (ret >= 0) {
ret = -EINVAL;
}
fprintf(stderr, "kvm version too old\n");
goto err;
}
if (ret > KVM_API_VERSION) {
ret = -EINVAL;
fprintf(stderr, "kvm version not supported\n");
goto err;
}
//...
// 创建虚拟机，保存虚拟机描述符s->vmfd
do {
ret = kvm_ioctl(s, KVM_CREATE_VM, type);
} while (ret == -EINTR);
//...
s->vmfd = ret;
//...
}
```

其主要功能是保存了kvm设备描述符`s->fd`，创建的虚拟机的描述符`s->vmfd`。

### 2.3 虚拟cpu创建

在2.1节中有提到，`qmp_x_exit_preconfig`函数与虚拟cpu的创建有关。

动态调试跟踪分析

`qmp_x_exit_preconfig` `qemu-6.2.0\softmmu\vl.c:2740`

--> `qemu_init_board` `qemu-6.2.0\softmmu\vl.c:2652`

--> `machine_run_board_init` `qemu-6.2.0\hw\core\machine.c:1181`

--> `pc_q35_init` `qemu-6.2.0\hw\i386\pc_q35.c:182`

--> `x86_cpus_init` `qemu-6.2.0\hw\i386\x86.c:141`

--> `x86_cpu_new` `qemu-6.2.0\hw\i386\x86.c:114`

在`machine_run_board_init`函数中根据参数中给的机器类型调用不同的`pc_machine_init`函数

`machine_class->init(machine)`----`pc_q35_init`

```
void x86_cpus_init(X86MachineState *x86ms, int default_cpu_version)
{
/...
/ 根据参数smp的值，创建对应数量的虚拟cpu
for (i = 0; i < ms->smp.cpus; i++) {
x86_cpu_new(x86ms, possible_cpus->cpus[i].arch_id, &error_fatal);
}
}
```

在`x86_cpu_new`中继续虚拟cpu的创建

`x86_cpu_new`

--> `qdev_realize` `qemu-6.2.0\hw\core\qdev.c:333`

--> `device_set_realized` `qemu-6.2.0\hw\core\qdev.c:531`

--> `x86_cpu_realizefn` `qemu-6.2.0\target\i386\cpu.c:6447`

--> `qemu_init_vcpu` `qemu-6.2.0\softmmu\cpus.c:613`

在`x86_cpu_realizefn`中调用`qemu_init_vcpu`对创建的虚拟cpu进行初始化

### 2.4 虚拟机运行

`qemu-6.2.0/softmmu/cpus.c` line 611

在`cpus.c`文件611行`qemu_init_vcpu`中初始化虚拟cpu，创建执行线程。

|  |  |
| --- | --- |
