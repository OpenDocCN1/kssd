
# [原创] linux中的信号处理与SROP

> 版权声明：本文为CSDN博主「ashimida@」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：195K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4A6k6r3q4F1x3e0p5K6L8r3W2V1j5h3&6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3U0f1@1y4K6R3#2y4l9`.
> 更多内容可关注微信公众号 ![](https://bbs.kanxue.com/upload/attach/202201/MYKVSVXH57H9873.jpg)

# 一、基本概念

信号是事件发生时对进程的通知机制,其与中断类似,在到达时都会打断程序的正常执行流程。一个进程(若具有权限则)可以向另一个进程或向自身发送信号，其可以作为一种同步技术或进程间通信的原始形式。发往进程的诸多信号通常都源于内核，引发内核为进程产生信号的事件包括:
- **硬件异常:** 如用户态的访问异常/除零异常等，其首先都是由硬件捕获并通知内核的，再由内核通过信号传递给用户态。
- **用户输入的中断字符**: 如`ctrl-c`, `ctrl-z`。
- **软件事件的发生**: 如针对文件描述符的输出变为有效,终端大小调整,定时器到期,cpu执行时间到期,子进程退出等。

每个信号在系统中都有唯一的编码，其编号随着系统的不同而不同，故程序中应该总是使用符号名来代表这些信号。

信号分为标准信号和实时信号, **在linux中编号1~31为标准信号 >31(<=64)的为实时信号**。

信号在产生后可能会经历一段时间才会真正被处理(到达),在此过程中信号则处于`pending`(等待状态), **在内核返回用户态时才会检查信号是否到来,**故:
- **若进程向其他进程发送信号**，则通常总要有一段(极短的)`pending`的时间, 直到目标进程被调度到或目标进程正在运行时产生了`el0`异常.
- **若进程向自身发送信号**，则通常在如`*kill`系统调用返回时此信号立即被处理。

**有时为了确保一段代码不被打断，可以通过掩码来屏蔽部分信号，被屏蔽的信号会一直处于等待状态,直到接触屏蔽**。

通过`/proc/pid/status`接口可以查看当前进程的信号:

```
## 实现代码参考内核 ./fs/proc/array.c task_sig

tangyuan@ubuntu:~/tests/namespace$ cat /proc/xxx/status

......

SigQ:   1/31451								## 当前进程信号队列中总共收到了多少个信号

SigPnd: 0000000000000000			## 当前线程收到过哪些信号,SigPnd(signal pending)是收到的信号掩码

ShdPnd: 0000000000000200			## 当前*线程组*共享队列中收到了哪些信号, ShdPnd(shared pending)是共享队列中收到的掩码,这里0x200代表收到信号为SIGUSR1

SigBlk: 0000000000000a00			## 当前线程阻塞的信号掩码,当前SIGUSR1/2信号均被阻塞

SigIgn: 0000000000000000			## 当前*线程组*忽略的信号,信号忽略是以线程组为单位的

SigCgt: 0000000000000a00		  ## 当前*线程组* 捕获的信号,也就是自定义了信号处理函数的信号,当前SIGUSR1/2均自定义了信号处理函数

......
```

linux中各信号的定义可参考[0], 这里需要注意的是:

**1. 标准信号不排队，实时信号需排队处理**
- 标准信号不做排队处理: 即内核某线程/线程组若收到某个标准信号，则在其被处理前是不会再次在`pending`队列中加入同一个标准信号的。**这意味着若一个标准信号多次到达，其信号处理函数有可能只被调用了一次**。
- 实时信号需要排队处理: 即内核某线程/线程组若收到实时信号，则不论之前是否有收到过此信号，都会在`pending`队列新增此信号。**这也意味着每发生一次实时信号其信号处理函数都会被调用一次**。

**2. 内核线程也可以接收信号**
虽然信号处理是为用户进程设计的，但在linux中内核线程也是可以接受信号的。和用户进程不同的是:
- 内核线程中只可以确定要接受哪个信号，但不能为信号指定具体处理函数
- 内核线程不会主动触发信号处理函数，若想要查看自身收到的信号，内核线程中需要使用循环来判断自身是否收到了信号(默认的信号处理发生在内核返回到用户态,内核线程不会触发此流程)
- 内核线程可以指定其接受的某个信号只能由内核态发出，即其可以指定自身不接受用户态发送的信号。
- 内核线程不接受`SIGKILL`信号
内核线程的信号处理可以参考内核线程函数 `jffs2_garbage_collect_thread`.

**3. init进程不接受SIGKILL/SIGSTOP信号**
见内核`sig_task_ignored`函数

# 二、信号处理函数的设置

从内核角度看，一个线程的信号可能保存在两个队列中:
- 一个是线程组共享的信号队列(`task_struct->signal->shared_pending`)
- 一个是线程自身私有的信号队列(`task_struct->pending`)

通常信号是发送到线程组共享的信号队列的，此队列中的信号被线程组中的任意线程处理(一次)即可，而在用户态看来则是一个信号可能被线程组中的任一线程处理。而通过如`tkill`系统调用也可以将信号直接发送给线程的私有信号队列，此时虽然线程组中所有线程的信号处理函数是同一个，但可以确保此信号只会由某个具体的线程来处理。在内核中信号相关的结构体定义如下:

```
// task_struct中信号相关字段

struct task_struct {

    ......

    /* Signal handlers: */

	struct signal_struct		*signal;		/* 指向线程组共享的信号描述信息的指针 */

	struct sighand_struct __rcu		*sighand;	/* 指向线程组共享的信号处理函数描述信息的指针 */

    /*

       线程私有的被阻塞信号结构体, sigset_t是一个信号掩码, 每个信号在其中占一个bit位,需要注意block和ignore不同:

       * 被设置为ignore后再接收到此信号则会被直接忽略,设置时若发现有已到达的ignore信号也会丢弃。

       * 被设置为blocked后再接收到此信号同样还需要加入到信号队列，只是此时不再向县城发送TFI_SIGPENDING到达信号了,

         后续unblock之后此信号还是会被处理的.

    */

	sigset_t			blocked;		

	sigset_t			real_blocked;

	sigset_t			saved_sigmask;			/* 在sigsuspend等函数等待信号期间临时保存之前的 block 掩码 */

	struct sigpending		pending;			/* 线程私有的信号队列 */

    

	unsigned long			sas_ss_sp;			/* 若线程有单独的信号栈则记录在这里 */

	size_t					sas_ss_size;

	unsigned int			sas_ss_flags;

    ......

}



//只记录部分相关结构体

struct signal_struct {

	......

	struct list_head	thread_head;	/* 指向线程组组长的task_struct */

	......

	struct sigpending	shared_pending;	/* 线程组的共享信号队列 */

    

	/* thread group exit support */

	int			group_exit_code;		/* 整个线程组是因为哪个信号退出的,见 complete_signal */

	int			group_stop_count;		/* thread group stop support, overloads group_exit_code too */

	unsigned int		flags; 			/* see SIGNAL_* flags below */



	struct pid *pids[PIDTYPE_MAX];		/* 信号处理时有时会向进程组，会话发送信号，这里记录进程组和会话pid等相关信息 */

    ......

}



struct sighand_struct {

	spinlock_t		siglock;       

	refcount_t		count;				/* 引用计数 */

	wait_queue_head_t	signalfd_wqh;	/* 等待此signal 的signalfd 队列,见 signalfd_read */

	struct k_sigaction	action[_NSIG];	/* 记录每个信号的信号处理函数等相关信息 */

};



struct k_sigaction {

	struct sigaction sa;

	......

};



struct sigaction {

    /* 信号处理函数指针,

       * 若为0(SIG_DFL)则代表使用默认信号处理函数

       * 若为1(SIG_IGN)则代表此信号被忽略

       * 若为2(SIG_KTHREAD)则代表当前内核线程可以接受用户态/内核态向其发送此信号

       * 若为3(SIG_KTHREAD_KERNEL)则代表当前内核线程只可以接受内核态向其发送此信号

    */

	__sighandler_t	sa_handler;

	unsigned long	sa_flags;		/* 某些信号会有一些细节控制flag，如SIGCHLD可以指定SA_NOCLDSTOP */

	sigset_t sa_mask;				/* 当当前信号正在处理时需屏蔽的其他信号，其可以用于防止信号处理函数被再次中断 */

};
```

各结构体关系如下图:

![](https://bbs.kanxue.com/upload/attach/202201/490870_8BR4T46PXSMS9WK.jpg)

linux 用户态可以通过`signal`/`sigaction`函数设置信号处理函数,二者系统调用接口如下:

```
SYSCALL_DEFINE3(sigaction, int, sig, const struct old_sigaction __user *, act, struct old_sigaction __user *, oact);

SYSCALL_DEFINE2(signal, int, sig, __sighandler_t, handler);
```

二者最终均调用了`do_sigaction`函数,这里以简单的**`sys_signal`**函数为例:

```
/* 将当前线程组信号sig的处理函数设置为handler, 并返回旧的信号处理函数指针 */

SYSCALL_DEFINE2(signal, int, sig, __sighandler_t, handler)

{

	struct k_sigaction new_sa, old_sa;

	int ret;



	new_sa.sa.sa_handler = handler;		            /* 构建一个 sigaction结构体并指定用户态的信号处理函数 handler */

	new_sa.sa.sa_flags = SA_ONESHOT | SA_NOMASK;	    /* signal 函数默认是单次触发 */

	

	sigemptyset(&new_sa.sa.sa_mask);				/* 清空/重置当前信号处理时的掩码 */

	

        /* 设置当前线程组信号sig的处理函数，并返回此信号旧的处理函数指针; 若信号设置为被忽略则需删除已收到的所有此信号 */

	ret = do_sigaction(sig, &new_sa, &old_sa);		



	return ret ? ret : (unsigned long)old_sa.sa.sa_handler;	/* 出错返回错误码,否则返回原有的handler */

}
```

**`do_sigaction`**:

```
/*

  此函数负责为当前线程组设置信号sig的信号处理函数(记录在act中),并通过oact返回之前的信号处理函数.

  如果act中指定信号sig会被忽略，那么会删除此线程组信号队列(包括线程组所有线程私有信号队列)中已经接受到的此信号.

*/

int do_sigaction(int sig, struct k_sigaction *act, struct k_sigaction *oact)

{

	struct task_struct *p = current, *t;

	struct k_sigaction *k;

	sigset_t mask;



    /* 若非有效信号([1,64]之外的信号),或是不可屏蔽信号则返回错误。不可屏蔽信号(SIG_KILL/SIG_STOP)不能设置handler */

	if (!valid_signal(sig) || sig < 1 || (act && sig_kernel_only(sig)))

		return -EINVAL;



	k = &p->sighand->action[sig-1];		/* 获取当前线程组中此信号的 action 数组 */

	

	if (oact)							/* 如果需要获取旧的信号处理信息,则通过oact 返回 */

		*oact = *k;

	.......

	if (act) {

	    /*  传入的sa_mask为此信号处理函数执行过程中需要屏蔽的其他信号, SIGKILL/SIGSTOP总是不可屏蔽信号,需要去除 */

		sigdelsetmask(&act->sa.sa_mask, sigmask(SIGKILL) | sigmask(SIGSTOP));

			      

		*k = *act;		/* 将新的action结构体复制到线程组此信号的action结构体中, 信号处理函数设置完毕 */

		 

	    /*

	       如果当前信号被设置为要忽略(此信号handler设置为SIG_IGN,或设置为SIG_DFL且此信号默认行为是忽略), 则需要同时删除此线程所

	      在线程组中所有信号队列中已经收到的所有此信号. 这包括线程组的shared_pending和各个线程自身的pending队列中:

	        * 已经加入队列的信号结构体(sigqueue)的删除

	        * 清除这些队列自身sigpending->signal中此sig的掩码

	     */

		if (sig_handler_ignored(sig_handler(p, sig), sig)) {

			sigemptyset(&mask);			/* 生成此信号对应的掩码 */

			sigaddset(&mask, sig);

			

			flush_sigqueue_mask(&mask, &p->signal->shared_pending);		/* 删除shared_pending中已有的所有此信号 */

			for_each_thread(p, t)					

				flush_sigqueue_mask(&mask, &t->pending);				/* 同时删除线程组各个线程中的此信号 */

		}

	}



	return 0;

}
```

# 三、信号的发送

这里以用户态入口系统调用`sys_kill`为例,其定义如下:

```
SYSCALL_DEFINE2(kill, pid_t, pid, int, sig)

{

	struct kernel_siginfo info;



        /* 为signal 准备 kernel_siginfo 结构体*/

	prepare_kill_siginfo(sig, &info);



	return kill_something_info(sig, &info, pid);

}



static int kill_something_info(int sig, struct kernel_siginfo *info, pid_t pid)

{

	int ret;



	if (pid > 0)

		return kill_proc_info(sig, info, pid);			/* 若pid > 0 ，则向此pid对应的线程组发送信号 */

	......

	return ret;

}
```
