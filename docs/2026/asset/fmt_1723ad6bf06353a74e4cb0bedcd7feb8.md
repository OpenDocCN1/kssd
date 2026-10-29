
`asmlinkage void el0_svc_handler(struct pt_regs *regs)` `{`
`sve_user_discard();`
`el0_svc_common(regs, regs->regs[8], __NR_syscalls, sys_call_table);`
`}`
`static void el0_svc_common(struct pt_regs *regs, int scno, int sc_nr, const syscall_fn_t syscall_table[])` `{`
`unsigned long flags = current_thread_info()->flags;`

`regs->orig_x0 = regs->regs[0];`
`regs->syscallno = scno;`

`...`
`invoke_syscall(regs, scno, sc_nr, syscall_table);`
`...`
`}`
`static long __invoke_syscall(struct pt_regs *regs, syscall_fn_t syscall_fn)` `{`
`return syscall_fn(regs);`
`}`

`static void invoke_syscall(struct pt_regs *regs, unsigned int scno, unsigned int sc_nr, const syscall_fn_t syscall_table[])` `{`
`long ret;`
`if (scno < sc_nr) {`
`syscall_fn_t syscall_fn;`
`syscall_fn = syscall_table[array_index_nospec(scno, sc_nr)];`
`ret = __invoke_syscall(regs, syscall_fn);`
`} else {`
`ret = do_ni_syscall(regs, scno);`
`}`

`regs->regs[0] = ret;`
`}`

总结起来就一句话，

`syscall_table[scno](regs);`

编译时生成`syscall_table`,根据`id`索引到`handler`的地址（所以`hook syscall`函数可以直接替换这个`syscall_table[id]`寻址得到的函数地址，主要是要找到`syscall_table`的地址，和`call`回原函数处理）

所以要添加个`syscall`，就需要做两件事，定义个`handler`（`SYSCALL_DEFINE`x 宏包起来的声明，`x`是参数数量，具体看其他函数实现就知道了），增加个`NR_ID`（如果上面第1点也做了需要`bionic`的`NR_ID`和`linux`的`NR_ID`一致）并声明`syscall`。

### 1. 定义handler

图方便还是用了先有文件`kernel\xiaomi\sm8250\kernel\sys.c`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/792494_TD3Z4KBV77J4U75.png)

### 2. 增加NR_ID，看好路径，因为我再bionic也新增了syscall，这里要对应上

`sys_antidebug_update`，这个前缀`sys`是`SYSCALL_DEFINE`x 宏添加的，所以这里面要加上`sys`，看其他`syscall`声明或者展开下宏就知道了。

![图片描述](https://bbs.kanxue.com/upload/attach/202211/792494_HYHGYX9NYHWPTFB.png)

添加完后`build`编译不报错，刷机，这里可以直接写个`demo`验证下

![图片描述](https://bbs.kanxue.com/upload/attach/202211/792494_2A9MR88Y72JFCV4.png)

用 `aarch64-linux-gnu-gcc` 编译（`ubuntu` 可以直接`apt`装`arm`的交叉编译工具），不要忘记`-static`选项，刷机到手机上运行下，可以看到函数逻辑是走了的。

传入参数`123`，`return 123+100` ，所以`syscall`没有问题。

![图片描述](https://bbs.kanxue.com/upload/attach/202211/792494_R4SM7TSH37NCCHF.png)

但我就是看不到`printk`打的日志去哪了，打算封装个函数把日志写到文件去了。

`unistd.h`到处都是，看的人都晕了...

修改源码重新编译，时间成本比较大，安装驱动（`.ko`）是比较常用的了，有需要就`syscall hook`下，毕竟扩展性比较好，我也试过，内核编译选项把强制校验签名关了，可以随便安装未签名的驱动了，但同样是`printk`打不出来日志。不过自己的环境定制内核还是最方便的，本着目的就是“一劳永逸”。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-11-25 16:20

被`xxxlion`编辑，原因：

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
