
可以看到已经被插桩了。这里也就是我们一开始看到的：`__afl_maybe_log`

## 收尾工作

在插桩结束后，我们把参数打印一下：

![](upload/attach/202102/876323_TDE3RSQA4PTZW2E.jpg)

可以看到这里在用汇编器`as`来将我们插桩好的`.s`文件生成可执行文件。

而真正的汇编过程是`fork()`出一个子进程来执行的。

```c
if (!(pid = fork())) {
    execvp(as_params[0], (char **) as_params);
    FATAL("Oops, failed to execute '%s' - check your PATH", as_params[0]);
}
```

`main()`函数中等待子进程执行完毕后退出。

**至此整个插桩过程就结束了。**

# instrumentation trampoline究竟是什么？

在上一部分我们已经知道了，64位下AFL将`trampoline_fmt_64`写入`.s`文件的指定位置作为插桩。

本部分主要来讨论AFL究竟插进去了什么东西。

## `trampoline_fmt_64`

我们直接看ida中的内容，非常直观，`trampoline_fmt_64`就是如下汇编：

```c
lea rsp, [rsp-98h]
mov [rsp+98h+var_98], rdx
mov [rsp+98h+var_90], rcx
mov [rsp+98h+var_88], rax
mov rcx, 46A1h ;注意这里46A1h就是在fptintf插桩的时候R(MAP_SIZE)产生的随机数，可以用于区分每个桩
call __afl_maybe_log
mov rax, [rsp+98h+var_88]
mov rcx, [rsp+98h+var_90]
mov rdx, [rsp+98h+var_98]
lea rsp, [rsp+98h]
```

## `__afl_maybe_log`

大体流程如下：

![](upload/attach/202102/876323_4YXG58WPAWUZSE6.jpg)

在这之前我们首先要关注几个位于bss段的变量：

```c
.AFL_VARS:
.comm __afl_area_ptr, 8
.comm __afl_prev_loc, 8
.comm __afl_fork_pid, 4
.comm __afl_temp, 4
.comm __afl_setup_failure, 1
.comm __afl_global_area_ptr, 8, 8
```

- `__afl_area_ptr`：共享内存的地址。
- `__afl_prev_loc`：上一个插桩位置（`R(100)`随机数的值）
- `__afl_fork_pid`：由`fork()`产生的子进程pid
- `__afl_temp`：缓冲区
- `__afl_setup_failure`：标志位，如果置位则直接退出。
- `__afl_global_area_ptr`：一个全局指针。

```c
__afl_maybe_log:
lahf
seto %al

/* Check if SHM region is already mapped. */
movq __afl_area_ptr(%rip), %rdx
testq %rdx, %rdx
je __afl_setup
```

首先`lahf`用于将标志寄存器的低八位送入`AH`，即将标志寄存器`FLAGS`中的`SF`、`ZF`、`AF`、`PF`、`CF`五个标志位分别传送到累加器`AH`的对应位（八位中有三位是无效的）。

![](upload/attach/202102/876323_D89FYYC4YHDYP99.jpg)

接下来`seto`溢出置位。

然后检查共享内存是否已经被设置了。即`__afl_area_ptr`是否为空？

- 如果为`NULL`则说明还没有被设置，跳转到`__afl_setup`进行设置。
- 否则继续运行。

## `__afl_setup`

在`__afl_setup:`中用于初始化`__afl_area_ptr`，只有在运行到第一个桩时会进行本次初始化。

```c
__afl_setup:
/* Do not retry setup if we had previous failures. */
cmpb $0, __afl_setup_failure(%rip)
jne __afl_return
/* Check out if we have a global pointer on file. */
movq __afl_global_area_ptr(%rip), %rdx
testq %rdx, %rdx
je __afl_setup_first
movq %rdx, __afl_area_ptr(%rip)
jmp __afl_store
```

如果`__afl_setup_failure`不为0的话，直接跳转到`__afl_return`返回。

接下来检查`__afl_global_area_ptr`文件指针是否为`NULL`，如果为空则跳转到`__afl_setup_first`。

否则将`__afl_global_area_ptr`的值赋给`__afl_area_ptr`后跳转到`__afl_store`

## `__afl_setup_first`

```c
__afl_setup_first:
/* Save everything that is not yet saved and that may be touched by
   getenv() and several other libcalls we'll be relying on. */
leaq -352(%rsp), %rsp
movq %rax, 0(%rsp)
movq %rcx, 8(%rsp)
movq %rdi, 16(%rsp)
movq %rsi, 32(%rsp)
movq %r8, 40(%rsp)
movq %r9, 48(%rsp)
movq %r10, 56(%rsp)
movq %r11, 64(%rsp)
movq %xmm0, 96(%rsp)
movq %xmm1, 112(%rsp)
movq %xmm2, 128(%rsp)
movq %xmm3, 144(%rsp)
movq %xmm4, 160(%rsp)
movq %xmm5, 176(%rsp)
movq %xmm6, 192(%rsp)
movq %xmm7, 208(%rsp)
movq %xmm8, 224(%rsp)
movq %xmm9, 240(%rsp)
movq %xmm10, 256(%rsp)
movq %xmm11, 272(%rsp)
movq %xmm12, 288(%rsp)
movq %xmm13, 304(%rsp)
movq %xmm14, 320(%rsp)
movq %xmm15, 336(%rsp)
/* Map SHM, jumping to __afl_setup_abort if something goes wrong. */
/* The 64-bit ABI requires 16-byte stack alignment. We'll keep the
   original stack ptr in the callee-saved r12. */
pushq %r12
movq %rsp, %r12
subq $16, %rsp
andq $0xfffffffffffffff0, %rsp
leaq .AFL_SHM_ENV(%rip), %rdi
call _getenv
testq %rax, %rax
je __afl_setup_abort
movq %rax, %rdi
call _atoi
xorq %rdx, %rdx /* shmat flags */
xorq %rsi, %rsi /* requested addr */
movq %rax, %rdi /* SHM ID */
call _shmat
cmpq $-1, %rax
je __afl_setup_abort
/* Store the address of the SHM region. */
movq %rax, %rdx
movq %rax, __afl_area_ptr(%rip)
movq %rax, __afl_global_area_ptr(%rip)
movq %rax, %rdx
```

1. 在`__afl_setup_first`中，首先保存寄存器的值（包括`xmm`寄存器组）
2. 接下来进行`rsp`对齐操作。
3. 获取环境变量`"__AFL_SHM_ID"`的值（共享内存的id）。如果获取失败，那么跳转到`__afl_setup_abort`，说明获取失败。
4. 获取成功后调用`shmat`启用对共享内存的访问。如果启用失败，跳转到`__afl_setup_abort`。
5. 将`shmat`返回的共享内存的地址存储在 `__afl_area_ptr` 与 `__afl_global_area_ptr`全局变量中。
6. 一切顺利的话，接下来运行 `__afl_forkserver`

## `__afl_forkserver`

```c
/* Enter the fork server mode to avoid the overhead of execve() calls. We
   push rdx (area ptr) twice to keep stack alignment neat. */
pushq %rdx
pushq %rdx

/* Phone home and tell the parent that we're OK. (Note that signals with
   no SA_RESTART will mess it up). If this fails, assume that the fd is
   closed because we were execve()d from an instrumented binary, or because
   the parent doesn't want to use the fork server. */
movq $4, %rdx /* length */
leaq __afl_temp(%rip), %rsi /* data */
movq $(198 + 1), %rdi /* file desc */
call _write
cmpq $4, %rax
jne __afl_fork_resume
```

首先向`FORKSRV_FD+1`即199号描述符（即状态管道）中写出`__afl_temp`中的四个字节，来通知afl我们的fork server已经启动成功。

顺带一提，这里的向状态管道中写的值，在`afl-fuzz.c`中的这个位置被读出来：

![](upload/attach/202102/876323_VJTGAMXRZD39SGU.jpg)

**这样我们整个过程就串连起来了。**

接下来进入：`__afl_fork_wait_loop:`

## `__afl_fork_wait_loop`:

```c
__afl_fork_wait_loop:
/* Wait for parent by reading from the pipe. Abort if read fails. */
movq $4, %rdx /* length */
leaq __afl_temp(%rip), %rsi /* data */
movq $198, %rdi /* file desc */
call _read
cmpq $4, %rax
jne __afl_die
/* Once woken up, create a clone of our process. This is an excellent use
   case for syscall(__NR_clone, 0, CLONE_PARENT), but glibc boneheadedly
   caches getpid() results and offers no way to update the value, breaking
   abort(), raise(), and a bunch of other things :-( */
call _fork
cmpq $0, %rax
jl __afl_die
je __afl_fork_resume
/* In parent process: write PID to pipe, then wait for child. */
movl %eax, __afl_fork_pid(%rip)
movq $4, %rdx /* length */
leaq __afl_fork_pid(%rip), %rsi /* data */
movq $(198 + 1), %rdi /* file desc */
call _write
movq $0, %rdx /* no flags */
leaq __afl_temp(%rip), %rsi /* status */
movq __afl_fork_pid(%rip), %rdi /* PID */
call _waitpid
cmpq $0, %rax
jle __afl_die
/* Relay wait status to pipe, then loop back. */
movq $4, %rdx /* length */
leaq __afl_temp(%rip), %rsi /* data */
movq $(198 + 1), %rdi /* file desc */
call _write
jmp __afl_fork_wait_loop
```

1. 首先我们等待parent（fuzz）通过控制管道发来的命令，读入`__afl_temp`中。
2. 如果读取失败，那么跳到 `__afl_die`，break出循环。
3. `_fork`出一个子进程，子进程跳入执行：`__afl_fork_resume`
4. 将`fork()`出来的子进程pid赋值给`__afl_fork_pid`
5. 向状态管道中写出子进程pid，告知parent。此时`__afl_maybe_log`中的父进程作为forksrever与我们的fuzz进行通信。
6. 等待我们`fork()`出的子进程执行完毕。然后写入状态管道告知fuzz。
7. 重新执行下一轮 `__afl_fork_wait_loop`进行测试。

## `__afl_fork_resume` 与 `__afl_store`

```c
/* In child process: close fds, resume execution. */
movq $198, %rdi
call _close
movq $(198 + 1), %rdi
call _close
popq %rdx
popq %rdx
movq %r12, %rsp
popq %r12
movq 0(%rsp), %rax
movq 8(%rsp), %rcx
movq 16(%rsp), %rdi
movq 32(%rsp), %rsi
movq 40(%rsp), %r8
movq 48(%rsp), %r9
movq 56(%rsp), %r10
movq 64(%rsp), %r11
movq 96(%rsp), %xmm0
movq 112(%rsp), %xmm1
movq 128(%rsp), %xmm2
movq 144(%rsp), %xmm3
movq 160(%rsp), %xmm4
movq 176(%rsp), %xmm5
movq 192(%rsp), %xmm6
movq 208(%rsp), %xmm7
movq 224(%rsp), %xmm8
movq 240(%rsp), %xmm9
movq 256(%rsp), %xmm10
movq 272(%rsp), %xmm11
movq 288(%rsp), %xmm12
movq 304(%rsp), %xmm13
movq 320(%rsp), %xmm14
movq 336(%rsp), %xmm15
leaq 352(%rsp), %rsp
jmp __afl_store
```

1. 首先关闭子进程中的文件描述符。
2. 恢复子进程的寄存器状态。
3. 跳转执行`__afl_store`

### `__afl_store`:

```c
__afl_store:
/* Calculate and store hit for the code location specified in rcx. */
xorq __afl_prev_loc(%rip), %rcx
xorq %rcx, __afl_prev_loc(%rip)
shrq $1, __afl_prev_loc(%rip)
incb (%rdx, %rcx, 1)
```

这一部分反编译出来如下：
