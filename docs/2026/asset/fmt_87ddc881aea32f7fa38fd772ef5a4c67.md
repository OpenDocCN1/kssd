
# [原创]Linux Kprobe原理探究

之前在分析其他安全厂商App的防护策略时，想要设计个风控分析沙盒来实现对于App行为的全面监控，包括：

- App访问、操作了哪些文件
- 执行了哪些操作
- 对于相关操作进行针对性的修改等等

其中很棘手的问题在于如何应对App中越来越常见的内联系统调用，对于内联系统调用的监控我不希望通过`ptrace`这类进程注入的方式来实现，而是想寻求通过定制系统或者相关的方式来实现以达到无侵入App的目的。

另一方面来说，通过定制系统的方式完成相关系统函数的修改确实是一种方式，但是定制系统在生产环境使用中会存在两个问题：

1. 调试测试：通常流程上都是相关函数修改->编译内核->借助AnyKernel3或者`Android_boot_image_editor`等工具完成`boot.img`重打包->刷入这些步骤，整体测试流程还是很繁琐的，其中还可能遇到代码bug导致系统无法启动等棘手问题，这些都对于实际开发来说很是崩溃。
2. 线上部署：和App一样，当本地测试好的内核遇到线上环境时可能会出现各式各样的问题，包括内核更新失败、内核文件传输、下载失败等等问题，直接导致系统无法启动，需要人工修复，想象下部署在遥远郊区的大规模设备集群大批量系统无法启动的场景，要靠人工一一修复是什么体验。

综上，最最贴合真实场景的是一种无侵入App且不阻断内核启动的方案，经过一顿搜索，最终定位到了Linux Kprobe这类内核监控方案。

第一次了解到kprobe技术是在evilpan的文章[Linux 内核监控在 Android 攻防中的应用](<mark class="encrypted">199K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9I4i4K6u0r3x3o6y4Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8@1M7X3q4U0K9h3&6Y4i4K6u0r3i4K6t1&6i4@1f1@1i4@1t1^5i4@1q4p5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1$3i4K6W2o6i4K6R3&6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1$3i4@1p5H3i4@1t1^5i4@1f1%4i4K6W2n7i4K6V1I4i4@1f1$3i4K6S2q4i4@1p5%4i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1$3i4@1p5I4i4K6R3^5i4@1f1@1i4@1t1^5i4@1q4p5i4@1f1#2i4K6R3^5i4K6R3$3i4@1f1@1i4@1t1^5i4@1u0m8i4@1f1$3i4K6V1#2i4@1t1H3i4@1f1$3i4K6S2p5i4@1q4q4i4@1f1K6i4K6R3H3i4K6R3I4i4@1f1&6i4K6R3%4i4K6R3%4i4@1f1&6i4K6W2n7i4K6R3$3i4@1f1K6i4K6R3H3i4K6R3I4i4@1f1#2i4K6R3&6i4K6S2p5i4@1f1%4i4@1q4n7i4@1q4r3i4@1f1@1i4@1t1^5i4K6R3&6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1#2i4@1t1I4i4K6R3J5i4@1f1%4i4@1u0m8i4@1p5%4</mark>）

![](upload/attach/202404/886627\_FZGT75Y4GXYCQG6.jpg)

而作为最底层的数据来源，kprobe、uprobe等是我们在做内核监控时需要重点关注的点，相比较于其他几种实现方式，kprobe无论从可扩展性、影响范围上都是最适合做二次开发的，参考作者给出的对比表：

|监控方案 |静态 |动态 |内核 |用户|
|----------- | ----------- | ----------- |----------- |----------- |
|Kprobes | |✔| ✔| |
|Uprobes | ✔|| |✔|
|Tracepoints |✔|| ✔ | |
|USDT |✔ ||| ✔|

因此最终确定了使用Linux Kprobe来作为内核系统函数的监控方案。

### 一、Kprobe基本知识

kprobe可以认为是一种kernel hook手段，它基于内核中断的方式实现，可以想象它是内核层的异常hook（参考SandHook），既然是异常hook，那么它所能hook的范围就没有限制了，可以针对函数、也可以针对单条指令。

简单理解就是把指定地址的指令替换成一个可以让cpu进入debug模式的指令（不同架构上指令不同），跳转到probe处理函数上进行数据收集、修改，再跳转回来继续执行。

X86中使用的是`int3`指令，ARM64中使用的是`BRK`指令进入debug monitor模式。

参考HPYU的Kprobe执行流程示意图：

![](upload/attach/202404/886627\_S3SXTYDSFQXCCVJ.jpg)

### 二、使用

kprobe主要有两种使用方法，一是通过模块加载；二是通过`debugfs`接口。从可扩展性和工程化的角度来看，模块加载是更优的选择，`debugfs`在某些特殊场景下（快速验证某些函数）可能会适合。

#### 基于内核模块加载

首先了解下动态内核模块（Loadable kernel module），LKM可以看出是内核向外提供的一个接口，通常是我们基于已编译好的内核产物+自定义的模块代码编译得到的`ko`文件，通过`insmod`的方式来实现动态新增定制功能，这种做法的好处是无需修改内核，需要新增功能时只需要变动相关LKM即可，它的作用域和静态编译的内核其他模块是完全等价的，而缺点是会带来些许性能上的损失，不过相比易用性来说这点可以忽略不计。

#### 2.1 案例

参考Linux源码下的`samples/kprobes`，里面包含kprobe、kretprobe等案例。

```c++
// samples/kprobes/kprobe\_example.c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>
/\* For each probe you need to allocate a kprobe structure \*/
static struct kprobe kp = {
.symbol\_name = "\_do\_fork",
};
/\* kprobe pre\_handler: called just before the probed instruction is executed \*/
static int handler\_pre(struct kprobe \*p, struct pt\_regs \*regs)
{
#ifdef CONFIG\_X86
printk(KERN\_INFO "pre\_handler: p->addr = 0x%p, ip = %lx,"
" flags = 0x%lx\n",
p->addr, regs->ip, regs->flags);
#endif
#ifdef CONFIG\_PPC
printk(KERN\_INFO "pre\_handler: p->addr = 0x%p, nip = 0x%lx,"
" msr = 0x%lx\n",
p->addr, regs->nip, regs->msr);
#endif
#ifdef CONFIG\_MIPS
printk(KERN\_INFO "pre\_handler: p->addr = 0x%p, epc = 0x%lx,"
" status = 0x%lx\n",
p->addr, regs->cp0\_epc, regs->cp0\_status);
#endif
#ifdef CONFIG\_TILEGX
printk(KERN\_INFO "pre\_handler: p->addr = 0x%p, pc = 0x%lx,"
" ex1 = 0x%lx\n",
p->addr, regs->pc, regs->ex1);
#endif
/\* A dump\_stack() here will give a stack backtrace \*/
return 0;
}
/\* kprobe post\_handler: called after the probed instruction is executed \*/
static void handler\_post(struct kprobe \*p, struct pt\_regs \*regs,
unsigned long flags)
{
#ifdef CONFIG\_X86
printk(KERN\_INFO "post\_handler: p->addr = 0x%p, flags = 0x%lx\n",
p->addr, regs->flags);
#endif
#ifdef CONFIG\_PPC
printk(KERN\_INFO "post\_handler: p->addr = 0x%p, msr = 0x%lx\n",
p->addr, regs->msr);
#endif
#ifdef CONFIG\_MIPS
printk(KERN\_INFO "post\_handler: p->addr = 0x%p, status = 0x%lx\n",
p->addr, regs->cp0\_status);
#endif
#ifdef CONFIG\_TILEGX
printk(KERN\_INFO "post\_handler: p->addr = 0x%p, ex1 = 0x%lx\n",
p->addr, regs->ex1);
#endif
}
/\*
\* fault\_handler: this is called if an exception is generated for any
\* instruction within the pre- or post-handler, or when Kprobes
\* single-steps the probed instruction.
\*/
static int handler\_fault(struct kprobe \*p, struct pt\_regs \*regs, int trapnr)
{
printk(KERN\_INFO "fault\_handler: p->addr = 0x%p, trap #%dn",
p->addr, trapnr);
/\* Return 0 because we don't handle the fault. \*/
return 0;
}
static int \_\_init kprobe\_init(void)
{
int ret;
kp.pre\_handler = handler\_pre;
kp.post\_handler = handler\_post;
kp.fault\_handler = handler\_fault;
ret = register\_kprobe(&kp);
if (ret < 0) {
printk(KERN\_INFO "register\_kprobe failed, returned %d\n", ret);
return ret;
}
printk(KERN\_INFO "Planted kprobe at %p\n", kp.addr);
return 0;
}
static void \_\_exit kprobe\_exit(void)
{
unregister\_kprobe(&kp);
printk(KERN\_INFO "kprobe at %p unregistered\n", kp.addr);
}
module\_init(kprobe\_init)
module\_exit(kprobe\_exit)
MODULE\_LICENSE("GPL");
// include/linux/kprobes.h
struct kprobe {
// 所有注册过的kprobe都会加入到kprobe\_table哈希表中，hlist指向哈希表的位置
struct hlist\_node hlist;
/\* list of kprobes for multi-handler support \*/
struct list\_head list;
/\*count the number of times this probe was temporarily disarmed \*/
unsigned long nmissed;
/\* location of the probe point \*/
kprobe\_opcode\_t \*addr;
/\* Allow user to indicate symbol name of the probe point \*/
// 地址和name不能同时出现，之前提过kprobe可以hook函数和地址
const char \*symbol\_name;
/\* Offset into the symbol \*/
unsigned int offset;
/\* Called before addr is executed. \*/
// 在单步执行原始指令前被调用
kprobe\_pre\_handler\_t pre\_handler;
/\* Called after addr is executed, unless... \*/
// 在单步执行原始指令后被调用
kprobe\_post\_handler\_t post\_handler;
/\* Saved opcode (which has been replaced with breakpoint) \*/
kprobe\_opcode\_t opcode;
// 保存平台相关的被探测指令和下一条指令
/\* copy of the original instruction \*/
struct arch\_specific\_insn ainsn;
/\*
\* Indicates various status flags.
\* Protected by kprobe\_mutex after this kprobe is registered.
\*/
u32 flags;
};
```

整个案例可以拆分成几个部分来看：

1. LKM的定义

一个完整的LKM包含`module_init`、`module_exit`、`MODULE_LICENSE`三个部分。

- `module_init`初始化kprobe、注册相关hook
- `module_exit`删除已有的注册函数、释放指针

2. kprobe结构体定义

首先初始化了`kprobe`结构体，参考上文，这里赋值了`symbol_name`字段为`do_fork`，也就需要hook`do_fork`函数。

3. hook函数的处理

指定`pre_handler`、`post_handler`以及异常处理`handler_fault`。

4. kprobe注册

注册初始化完成的`kp`指针（感观上和xhook很像）。

这样就完成了对于`do_for`函数的hook，整体使用流程很清晰简单，初始化kprobes结构体（设置`symbol_name`、handler）->注册kprobes->LKM封装。

#### 2.2 编译

- 在Android端LKM单独编译是无法生效的，需要借助于内核编译产物来完成编译
- 现在市面上的设备大部分都是没有开启kprobe选项的，需要在内核源码中额外添加，这里可参考[Pixel 3 Kernel打开KPROBES编译选项](<mark class="encrypted">790K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2k6Y4g2@1N6i4u0W2M7%4q4D9i4K6u0W2N6r3!0H3i4K6u0r3M7r3!0K6N6q4)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6C8k6i4u0F1k6h3I4Q4y4h3k6J5k6h3y4G2L8i4m8A6L8r3g2Q4x3V1k6Q4x3U0W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4b7U0m8Q4b7U0q4Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0W2Q4z5f1y4Q4z5o6m8Q4c8e0S2Q4b7e0k6Q4z5o6q4Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0g2Q4b7f1k6Q4b7U0W2Q4c8e0g2Q4b7V1q4Q4z5e0c8Q4c8e0N6Q4z5o6W2Q4z5o6S2Q4c8e0k6Q4z5f1y4Q4b7f1y4Q4c8e0N6Q4z5f1q4Q4z5o6c8U0L8$3&6X3K9h3N6Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4b7f1c8S2M7X3y4Z5i4K6u0r3j5i4u0E0y4U0c8Q4x3V1k6U0L8$3&6X3K9h3N6K6i4K6u0r3P5s2S2^5P5s2S2Q4y4h3k6V1k6h3k6U0L8$3&6X3K9h3N6Q4c8e0g2Q4b7e0u0Q4z5f1g2Q4c8e0g2Q4z5p5q4Q4b7e0m8o6e0@1&6r3d9f1N6Q4y4h3k6w2f1q4u0a6b7V1g2Q4x3@1c8k6i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1$3i4@1p5H3i4@1t1%4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1f1&6i4K6R3H3i4K6R3&6i4@1f1&6i4@1p5I4i4@1t1&6</mark>)

到目前为止，对于kprobe的使用是比较清晰了，下面从其原理角度来探究它是如何实现这套hook机制的。

### 三、实现原理
