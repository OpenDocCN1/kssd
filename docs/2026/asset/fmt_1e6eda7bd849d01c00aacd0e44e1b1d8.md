
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```c
/**
* ptrace_event - possibly stop for a ptrace event notification
* @event: %PTRACE_EVENT_* value to report
* @message: value for %PTRACE_GETEVENTMSG to return
*
* Check whether @event is enabled and, if so, report @event and @message
* to the ptrace parent.
*
* Called without locks.
*/
static inline void ptrace_event(int event, unsigned long message)
{
if (unlikely(ptrace_event_enabled(current, event))) {
current->ptrace_message = message;
ptrace_notify((event << 8) | SIGTRAP);
} else if (event == PTRACE_EVENT_EXEC) {
/* legacy EXEC report via SIGTRAP */
if ((current->ptrace & (PT_PTRACED|PT_SEIZED)) == PT_PTRACED)
send_sig(SIGTRAP, current, 0);
}
}
```

在`exec.c`中对该函数的调用如下：

```c
static int exec_binprm(struct linux_binprm *bprm)
{
pid_t old_pid, old_vpid;
int ret, depth;
/* Need to fetch pid before load_binary changes it */
old_pid = current->pid;
rcu_read_lock();
old_vpid = task_pid_nr_ns(current, task_active_pid_ns(current->parent));
rcu_read_unlock();
.......
audit_bprm(bprm);
trace_sched_process_exec(current, old_pid, bprm);
// 调用ptrace_event,传入的event为PTRACE_EVENT_EXEC
// 直接走发送SIGTRAP的逻辑
ptrace_event(PTRACE_EVENT_EXEC, old_vpid);
proc_exec_connector(current);
return 0;
}
```

`SIGTRAP`信号的值为5，专门为调试设计。当kernel发生`int 3`时，触发回掉函数`do_trap()`，其代码如下：

```
asmlinkage void do_trap(struct pt_regs *regs, unsigned long address)
{
force_sig_fault(SIGTRAP, TRAP_TRACE, (void __user *)address);
regs->pc += 4;
}
int force_sig_fault(int sig, int code, void __user *addr
_ARCH_SI_TRAPNO(int trapno)
_ARCH_SI_IA64(int imm, unsigned int flags, unsigned long isr))
{
return force_sig_fault_to_task(sig, code, addr
_ARCH_SI_TRAPNO(trapno)
_ARCH_SI_IA64(imm, flags, isr), current);
}
```

父进程唤醒`wait`对子进程进行监控，`wait`有3种退出情况（子进程正常退出、收到信号退出、收到信号暂停），对于`PTRACE_TRACEME`来说，对应的是第三种情况--收到信号后暂停。

`PTRACE_TRACEME`只是表明了子进程可以被trace，如果进程调用了`PTRACE_TRACEME`，那么该进程处理信号的方式会发生改变。例如一个进程正在运行，此时输入`ctrl+c(SIGINT)`,则进程会直接退出；如果进程中有`ptrace (PTRACE_TRACEME,0，NULL,NULL)`，当输入`CTRL+C`时，该进程将会处于stopped的状态。

在`sys_ptrace`函数中，该部分的处理流程如下：

![image-20210205142516160](upload/attach/202102/779730_BS4TBTCMB925VUY.jpg)

在5.9版中，单独写成了`ptrace_traceme()`函数，而在2.6版本中，直接在`sys_ptrace`的逻辑中进行实现：

![image-20210205142732193](upload/attach/202102/779730_B8AXBAK2XB77VJB.jpg)

虽然2个版本的核心功能相同，但是5.9版本的处理逻辑和情况考量相比2.6版本上升了很大高度。

#### 2. PTRACE_ATTACH

1. 描述

attach到pid指定的进程，使其成为调用进程的`tracee`。`tracer`会向`tracee`发送一个`SIGSTOP`信号，但不一定已通过此调用完成而停止；`tracer`使用`waitpid()`等待`tracee`停止。

2. 定义

```c
static int ptrace_attach(struct task_struct *task, long request,
unsigned long addr,
unsigned long flags)
{
bool seize = (request == PTRACE_SEIZE);
int retval;
retval = -EIO; /* I/O error*/
/*
* 判断request是PTRACE_SEIZE还是PTRACE_ATTACH。
* 如果request为PTRACE_SEIZE，则进行必要的参数检查，错误时退出。
*/
if (seize) {
if (addr != 0)
goto out;
if (flags & ~(unsigned long)PTRACE_O_MASK)
goto out;
flags = PT_PTRACED | PT_SEIZED | (flags << PT_OPT_FLAG_SHIFT);
} else {
flags = PT_PTRACED;
}
audit_ptrace(task);
/*
* 判断task进程是否为kernel thread（PF_KTHREAD），
* 调用same_thread_group(task, current)，判断task是否和current进程在同一个线程组，查看current进程是否有权限trace task进程。
* 如果不符合要求，则直接退出。
*/
retval = -EPERM; /* Operation not permitted, retval = -1 */
if (unlikely(task->flags & PF_KTHREAD))
goto out;
if (same_thread_group(task, current))
goto out;
/*
* Protect exec's credential calculations against our interference;
* SUID, SGID and LSM creds get determined differently
* under ptrace.
*/
retval = -ERESTARTNOINTR;
if (mutex_lock_interruptible(&task->signal->cred_guard_mutex))
goto out;
task_lock(task);
retval = __ptrace_may_access(task, PTRACE_MODE_ATTACH_REALCREDS);
task_unlock(task);
if (retval)
goto unlock_creds;
write_lock_irq(&tasklist_lock);
retval = -EPERM;
if (unlikely(task->exit_state))
goto unlock_tasklist;
if (task->ptrace)
goto unlock_tasklist;
/*
* 设置子进程task->ptrace = PT_TRACED，被跟踪状态
*/
if (seize)
flags |= PT_SEIZED;
task->ptrace = flags;
/*
* 调用__ptrace_link(task, current)，将task->ptrace_entry链接到current->ptraced链表中。
*/
ptrace_link(task, current);
/* SEIZE doesn't trap tracee on attach */
/*
* 如果是PTRACE_ATTACH请求（PTRACE_SEIZE请求不会停止被跟踪进程），
* 则调用send_sig_info(SIGSTOP,SEND_SIG_PRIV, task);
* 发送SIGSTOP信号，中止task运行，设置task->state为TASK_STOPPED
*/
if (!seize)
send_sig_info(SIGSTOP, SEND_SIG_PRIV, task);
spin_lock(&task->sighand->siglock);
/*
* If the task is already STOPPED, set JOBCTL_TRAP_STOP and
* TRAPPING, and kick it so that it transits to TRACED. TRAPPING
* will be cleared if the child completes the transition or any
* event which clears the group stop states happens. We'll wait
* for the transition to complete before returning from this
* function.
*
* This hides STOPPED -> RUNNING -> TRACED transition from the
* attaching thread but a different thread in the same group can
* still observe the transient RUNNING state. IOW, if another
* thread's WNOHANG wait(2) on the stopped tracee races against
* ATTACH, the wait(2) may fail due to the transient RUNNING.
*
* The following task_is_stopped() test is safe as both transitions
* in and out of STOPPED are protected by siglock.
*
*
*
* 等待task->jobctl的JOBCTL_TRAPPING_BIT位被清零，
* 阻塞时进程状态被设置为TASK_UNINTERRUPTIBLE，引发进程调度
*/
if (task_is_stopped(task) &&
task_set_jobctl_pending(task, JOBCTL_TRAP_STOP | JOBCTL_TRAPPING))
signal_wake_up_state(task, __TASK_STOPPED);
spin_unlock(&task->sighand->siglock);
retval = 0;
unlock_tasklist:
write_unlock_irq(&tasklist_lock);
unlock_creds:
mutex_unlock(&task->signal->cred_guard_mutex);
out:
if (!retval) {
/*
* We do not bother to change retval or clear JOBCTL_TRAPPING
* if wait_on_bit() was interrupted by SIGKILL. The tracer will
* not return to user-mode, it will exit and clear this bit in
* __ptrace_unlink() if it wasn't already cleared by the tracee;
* and until then nobody can ptrace this task.
*/
wait_on_bit(&task->jobctl, JOBCTL_TRAPPING_BIT, TASK_KILLABLE);
proc_ptrace_connector(task, PTRACE_ATTACH);
}
return retval;
}
```

3. 分析

代码上可以看出，`PTRACE_ATTACH`处理的方式与`PTRACE_TRACEME`处理的方式不同。`PTRACE_ATTACH`会使父进程直接向子进程发送`SIGSTOP`信号，如果子进程停止，那么父进程的`wait`操作被唤醒，从而成功attach。一个进程不能attach多次。

在2.6版本中的实现如下(`kernel/ptrace.c`)：

```c
int ptrace_attach(struct task_struct *task)
{
int retval;
task_lock(task);
retval = -EPERM;
if (task->pid <= 1) // 不能调试init
goto bad;
if (task == current) // 不能调试自身
goto bad;
if (!task->mm)
goto bad;
/* 鉴权 */
if(((current->uid != task->euid) ||
(current->uid != task->suid) ||
(current->uid != task->uid) ||
(current->gid != task->egid) ||
(current->gid != task->sgid) ||
(current->gid != task->gid)) && !capable(CAP_SYS_PTRACE))
goto bad;
rmb();
if (!task->mm->dumpable && !capable(CAP_SYS_PTRACE))
goto bad;
if (task->ptrace & PT_PTRACED) // 一个进程不能被attach多次
goto bad;
retval = security_ptrace(current, task);
if (retval)
goto bad;
/* Go */
task->ptrace |= PT_PTRACED;
if (capable(CAP_SYS_PTRACE))
task->ptrace |= PT_PTRACE_CAP;
task_unlock(task);
write_lock_irq(&tasklist_lock);
__ptrace_link(task, current); // 调用__ptrace_link(task, current)，将task->ptrace_entry链接到current->ptraced链表中
write_unlock_irq(&tasklist_lock);
force_sig_specific(SIGSTOP, task); // 发送SIGSTOP，终止运行
return 0;
bad:
task_unlock(task);
return retval;
}
```

（未完待续）

承接上文 [Linux ptrace详细分析系列（一）](https://bbs.pediy.com/thread-265812.htm)，本次主要介绍2个不同版本下的源码分析和2个关键参数解析。

目录

- [三、sys_ptrace函数源码分析](https://bbs.kanxue.com/#三、sys_ptrace函数源码分析)

- 1. Linux-2.6版本的源码分析

- 1. 源码分析

- 2. 流程梳理

- 3. 其他

- 2. Linux-5.9版本的源码分析

- 1. 源码分析

- 2. 流程梳理

- 3. 其他

- [四、Request参数详解](https://bbs.kanxue.com/#四、request参数详解)

- 1. 参数简述

- 2. 重要参数详解

- 1. PTRACE_TRACEME

- 2. PTRA CE_ATTACH

## 三、sys_ptrace函数源码分析

### 1. Linux-2.6版本的源码分析

#### 1. 源码分析

首先看一下linux-2.6.0的`sys_ptrace`的处理流程（以`/arch/i386/kernel/ptrace.c`为例）：

|  |  |

| --- | --- |
