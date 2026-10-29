
```c
/** Note that this implementation of ptrace behaves differently from vanilla ptrace.  Contrary to what the man page says, in the PTRACE_PEEKTEXT, PTRACE_PEEKDATA, and PTRACE_PEEKUSER requests the data variable is not ignored.  Instead, the data variable is expected to point at a location (in user space) where the result of the ptrace call is written (instead of being returned). */
asmlinkage int sys_ptrace(long request, long pid, long addr, long data)
{
    struct task_struct *child;
    struct user * dummy = NULL;
    int i, ret;

    lock_kernel();
    ret = -EPERM;
    if (request == PTRACE_TRACEME) { // 请求为PTRACE_TRACEME
        /* 检查是否做好被跟踪的准备 */
        if (current->ptrace & PT_PTRACED)
            goto out;
        ret = security_ptrace(current->parent, current);
        if (ret)
            goto out;
        /* 检查通过，在process flags中设置ptrace位*/
        current->ptrace |= PT_PTRACED;
        ret = 0;
        goto out;
    }

    /* 非PTRACE_TRACEME的请求*/
    ret = -ESRCH; // 首先设置返回值为ESRCH，表明没有该进程，宏定义在errno-base.h头文件中
    read_lock(&tasklist_lock);
    child = find_task_by_pid(pid); // 查找task结构
    if (child)
        get_task_struct(child);
    read_unlock(&tasklist_lock);
    if (!child) // 没有找到task结构，指明所给pid错误
        goto out;

    ret = -EPERM; // 返回操作未授权
    if (pid == 1) // init进程不允许被调试
        goto out_tsk;

    /* 请求为 PTRACE_ATTACH 时*/
    if (request == PTRACE_ATTACH) {
        ret = ptrace_attach(child); // 进行attach
        goto out_tsk;
    }

    /* 检查进程是否被跟踪，没有的话不能执行其他功能；
     * 当不是PTRACE_KILL时，要求进程状态为TASK_STOPPED；
     * 被跟踪进程必须为当前进程的子进程
     * 在之前是直接在该代码处实现以上逻辑，现在重新将以上功能封装成了ptrace_check_attach函数
     */
    ret = ptrace_check_attach(child, request == PTRACE_KILL);
    if (ret < 0)
        goto out_tsk;

    /* 以下就为根据不同的request参数进行对应的处理了，用一个switch来总括，流程比较简单。*/
    switch (request) {
    /* when I and D space are separate, these will need to be fixed. 这算预告吗？23333*/
    case PTRACE_PEEKTEXT: /* read word at location addr. */
    case PTRACE_PEEKDATA: {
        unsigned long tmp;
        int copied;

        copied = access_process_vm(child, addr, &tmp, sizeof(tmp), 0);
        ret = -EIO; // 返回I/O错误
        if (copied != sizeof(tmp))
            break;
        ret = put_user(tmp,(unsigned long *) data);
        break;
    }

    /* read the word at location addr in the USER area. */
    case PTRACE_PEEKUSR: {
        unsigned long tmp;

        ret = -EIO;
        if ((addr & 3) || addr < 0 ||
            addr > sizeof(struct user) - 3)
            break;

        tmp = 0; /* Default return condition */
        if(addr < FRAME_SIZE*sizeof(long))
            tmp = getreg(child, addr);
        if(addr >= (long) &dummy->u_debugreg[0] &&
            addr <= (long) &dummy->u_debugreg[7]){
            addr -= (long) &dummy->u_debugreg[0];
            addr = addr >> 2;
            tmp = child->thread.debugreg[addr];
        }
        ret = put_user(tmp,(unsigned long *) data);
        break;
    }

    /* when I and D space are separate, this will have to be fixed. */
    case PTRACE_POKETEXT: /* write the word at location addr. */
    case PTRACE_POKEDATA:
        ret = 0;
        if (access_process_vm(child, addr, &data, sizeof(data), 1) == sizeof(data))
            break;
        ret = -EIO;
        break;

    case PTRACE_POKEUSR: /* write the word at location addr in the USER area */
        ret = -EIO;
        if ((addr & 3) || addr < 0 ||
            addr > sizeof(struct user) - 3)
            break;

        if (addr < FRAME_SIZE*sizeof(long)) {
            ret = putreg(child, addr, data);
            break;
        }
        /* We need to be very careful here.  We implicitly
           want to modify a portion of the task_struct, and we
           have to be selective about what portions we allow someone
           to modify. */

        ret = -EIO;
        if(addr >= (long) &dummy->u_debugreg[0] &&
            addr <= (long) &dummy->u_debugreg[7]){

            if(addr == (long) &dummy->u_debugreg[4]) break;
            if(addr == (long) &dummy->u_debugreg[5]) break;
            if(addr < (long) &dummy->u_debugreg[4] &&
                ((unsigned long) data) >= TASK_SIZE-3) break;

            if(addr == (long) &dummy->u_debugreg[7]) {
                data &= ~DR_CONTROL_RESERVED;
                for(i=0; i<4; i++)
                    if ((0x5f54 >> ((data >> (16 + 4*i)) & 0xf)) & 1)
                        goto out_tsk;
            }

            addr -= (long) &dummy->u_debugreg;
            addr = addr >> 2;
            child->thread.debugreg[addr] = data;
            ret = 0;
        }
        break;

    case PTRACE_SYSCALL: /* continue and stop at next (return from) syscall */
    case PTRACE_CONT: { /* restart after signal. */
        long tmp;

        ret = -EIO;
        if ((unsigned long) data > _NSIG)
            break;
        if (request == PTRACE_SYSCALL) {
            set_tsk_thread_flag(child, TIF_SYSCALL_TRACE);
        } else {
            clear_tsk_thread_flag(child, TIF_SYSCALL_TRACE);
        }
        child->exit_code = data;
        /* make sure the single step bit is not set. */
        tmp = get_stack_long(child, EFL_OFFSET) & ~TRAP_FLAG;
        put_stack_long(child, EFL_OFFSET,tmp);
        wake_up_process(child);
        ret = 0;
        break;
    }

    /*
     * make the child exit.  Best I can do is send it a sigkill.
     * perhaps it should be put in the status that it wants to
     * exit.
     */
    case PTRACE_KILL: {
        long tmp;

        ret = 0;
        if (child->state == TASK_ZOMBIE) /* already dead */
            break;
        child->exit_code = SIGKILL;
        /* make sure the single step bit is not set. */
        tmp = get_stack_long(child, EFL_OFFSET) & ~TRAP_FLAG;
        put_stack_long(child, EFL_OFFSET, tmp);
        wake_up_process(child);
        break;
    }

    case PTRACE_SINGLESTEP: { /* set the trap flag. */
        long tmp;

        ret = -EIO;
        if ((unsigned long) data > _NSIG)
            break;
        clear_tsk_thread_flag(child, TIF_SYSCALL_TRACE);
        if ((child->ptrace & PT_DTRACE) == 0) {
            /* Spurious delayed TF traps may occur */
            child->ptrace |= PT_DTRACE;
        }
        tmp = get_stack_long(child, EFL_OFFSET) | TRAP_FLAG;
        put_stack_long(child, EFL_OFFSET, tmp);
        child->exit_code = data;
        /* give it a chance to run. */
        wake_up_process(child);
        ret = 0;
        break;
    }

    case PTRACE_DETACH:
        /* detach a process that was attached. */
        ret = ptrace_detach(child, data);
        break;

    case PTRACE_GETREGS: { /* Get all gp regs from the child. */
        if (!access_ok(VERIFY_WRITE, (unsigned *)data, FRAME_SIZE*sizeof(long))) {
            ret = -EIO;
            break;
        }
        for ( i = 0; i < FRAME_SIZE*sizeof(long); i += sizeof(long) ) {
            __put_user(getreg(child, i),(unsigned long *) data);
            data += sizeof(long);
        }
        ret = 0;
        break;
    }

    case PTRACE_SETREGS: { /* Set all gp regs in the child. */
        unsigned long tmp;
        if (!access_ok(VERIFY_READ, (unsigned *)data, FRAME_SIZE*sizeof(long))) {
            ret = -EIO;
            break;
        }
        for ( i = 0; i < FRAME_SIZE*sizeof(long); i += sizeof(long) ) {
            __get_user(tmp, (unsigned long *) data);
            putreg(child, i, tmp);
            data += sizeof(long);
        }
        ret = 0;
        break;
    }

    case PTRACE_GETFPREGS: { /* Get the child FPU state. */
        if (!access_ok(VERIFY_WRITE, (unsigned *)data,
                       sizeof(struct user_i387_struct))) {
            ret = -EIO;
            break;
        }
        ret = 0;
        if (!child->used_math)
            init_fpu(child);
        get_fpregs((struct user_i387_struct __user *)data, child);
        break;
    }

    case PTRACE_SETFPREGS: { /* Set the child FPU state. */
        if (!access_ok(VERIFY_READ, (unsigned *)data,
                       sizeof(struct user_i387_struct))) {
            ret = -EIO;
            break;
        }
        child->used_math = 1;
        set_fpregs(child, (struct user_i387_struct __user *)data);
        ret = 0;
        break;
    }

    case PTRACE_GETFPXREGS: { /* Get the child extended FPU state. */
        if (!access_ok(VERIFY_WRITE, (unsigned *)data,
                       sizeof(struct user_fxsr_struct))) {
            ret = -EIO;
            break;
        }
        if (!child->used_math)
            init_fpu(child);
        ret = get_fpxregs((struct user_fxsr_struct __user *)data, child);
        break;
    }

    case PTRACE_SETFPXREGS: { /* Set the child extended FPU state. */
        if (!access_ok(VERIFY_READ, (unsigned *)data,
                       sizeof(struct user_fxsr_struct))) {
            ret = -EIO;
            break;
        }
        child->used_math = 1;
        ret = set_fpxregs(child, (struct user_fxsr_struct __user *)data);
        break;
    }

    case PTRACE_GET_THREAD_AREA:
        ret = ptrace_get_thread_area(child,
                                     addr, (struct user_desc __user *) data);
        break;

    case PTRACE_SET_THREAD_AREA:
        ret = ptrace_set_thread_area(child,
                                     addr, (struct user_desc __user *) data);
        break;

    default:
        ret = ptrace_request(child, request, addr, data);
        break;
    }
out_tsk:
    put_task_struct(child);
out:
    unlock_kernel();
    return ret;
}
```
