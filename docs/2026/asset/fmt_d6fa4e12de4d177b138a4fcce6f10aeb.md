
```c
acct_update_integrals(current);
task_numa_free(current, false);
return retval;

out:
if (bprm->point_of_no_return && !fatal_signal_pending(current))
    force_fatal_sig(SIGSEGV);
sched_mm_cid_after_execve(current);
current->in_execve = 0;
return retval;
}

static int exec_binprm(struct linux_binprm *bprm)
{
    pid_t old_pid, old_vpid;
    int ret, depth;

    old_pid = current->pid; //保存执行前的 PID 信息
    rcu_read_lock();
    old_vpid = task_pid_nr_ns(current, task_active_pid_ns(current->parent));
    rcu_read_unlock();

    //解析主循环
    for (depth = 0;; depth++) {
        struct file *exec;

        if (depth > 5)
            return -ELOOP;

        ret = search_binary_handler(bprm); //根据文件内容选择合适的处理器
        if (ret < 0)
            return ret;

        if (!bprm->interpreter)
            break;

        exec = bprm->file; // 解释器替换，下一轮执行的主角从原文件变成解释器本身
        bprm->file = bprm->interpreter;
        bprm->interpreter = NULL;

        allow_write_access(exec);
        if (unlikely(bprm->have_execfd)) {
            if (bprm->executable) {
                fput(exec);
                return -ENOEXEC;
            }
            bprm->executable = exec;
        } else
            fput(exec);
    }

    //执行成功后的系统通知
    audit_bprm(bprm);
    trace_sched_process_exec(current, old_pid, bprm);
    ptrace_event(PTRACE_EVENT_EXEC, old_vpid);
    proc_exec_connector(current);

    return 0;
}

static int exec_binprm(struct linux_binprm *bprm)
{
    pid_t old_pid, old_vpid;
    int ret, depth;

    old_pid = current->pid; //保存执行前的 PID 信息
    rcu_read_lock();
    old_vpid = task_pid_nr_ns(current, task_active_pid_ns(current->parent));
    rcu_read_unlock();

    //解析主循环
    for (depth = 0;; depth++) {
        struct file *exec;

        if (depth > 5)
            return -ELOOP;

        ret = search_binary_handler(bprm); //根据文件内容选择合适的处理器
        if (ret < 0)
            return ret;

        if (!bprm->interpreter)
            break;

        exec = bprm->file; // 解释器替换，下一轮执行的主角从原文件变成解释器本身
        bprm->file = bprm->interpreter;
        bprm->interpreter = NULL;

        allow_write_access(exec);
        if (unlikely(bprm->have_execfd)) {
            if (bprm->executable) {
                fput(exec);
                return -ENOEXEC;
            }
            bprm->executable = exec;
        } else
            fput(exec);
    }

    //执行成功后的系统通知
    audit_bprm(bprm);
    trace_sched_process_exec(current, old_pid, bprm);
    ptrace_event(PTRACE_EVENT_EXEC, old_vpid);
    proc_exec_connector(current);

    return 0;
}

static int search_binary_handler(struct linux_binprm *bprm)
{
    bool need_retry = IS_ENABLED(CONFIG_MODULES);
    struct linux_binfmt *fmt;
    int retval;

    retval = prepare_binprm(bprm); //读取文件前 128 字节
    if (retval < 0)
        return retval;

    retval = security_bprm_check(bprm); //安全检查
    if (retval)
        return retval;

    retval = -ENOENT;

retry:
    read_lock(&binfmt_lock);

    // 解析器匹配循环
    list_for_each_entry(fmt, &formats, lh) {
        if (!try_module_get(fmt->module))
            continue;
        read_unlock(&binfmt_lock);
        retval = fmt->load_binary(bprm); //检查文件魔数，判断文件格式
        read_lock(&binfmt_lock);
        put_binfmt(fmt);
        if (bprm->point_of_no_return || (retval != -ENOEXEC)) {
            read_unlock(&binfmt_lock);
            return retval;
        }
    }

    read_unlock(&binfmt_lock);

    //没有找到解析器
    if (need_retry) {
        if (printable(bprm->buf[0]) && printable(bprm->buf[1]) &&
            printable(bprm->buf[2]) && printable(bprm->buf[3]))
            return retval;
        if (request_module("binfmt-%04x", *(ushort *)(bprm->buf + 2)) < 0)
            return retval;
        need_retry = false;
        goto retry;
    }

    return retval;
}

static int search_binary_handler(struct linux_binprm *bprm)
{
    bool need_retry = IS_ENABLED(CONFIG_MODULES);
    struct linux_binfmt *fmt;
    int retval;

    retval = prepare_binprm(bprm); //读取文件前 128 字节
    if (retval < 0)
        return retval;

    retval = security_bprm_check(bprm); //安全检查
    if (retval)
        return retval;

    retval = -ENOENT;

retry:
    read_lock(&binfmt_lock);

    // 解析器匹配循环
    list_for_each_entry(fmt, &formats, lh) {
        if (!try_module_get(fmt->module))
            continue;
        read_unlock(&binfmt_lock);
        retval = fmt->load_binary(bprm); //检查文件魔数，判断文件格式
        read_lock(&binfmt_lock);
        put_binfmt(fmt);
        if (bprm->point_of_no_return || (retval != -ENOEXEC)) {
            read_unlock(&binfmt_lock);
            return retval;
        }
    }

    read_unlock(&binfmt_lock);

    //没有找到解析器
    if (need_retry) {
        if (printable(bprm->buf[0]) && printable(bprm->buf[1]) &&
            printable(bprm->buf[2]) && printable(bprm->buf[3]))
            return retval;
        if (request_module("binfmt-%04x", *(ushort *)(bprm->buf + 2)) < 0)
            return retval;
        need_retry = false;
        goto retry;
    }

    return retval;
}

retval = fmt->load_binary(bprm); //检查文件魔数，判断文件格式
retval = fmt->load_binary(bprm); //检查文件魔数，判断文件格式

//检查是否为 ELF 魔数 \x7FELF
if (memcmp(elf_ex->e_ident, ELFMAG, SELFMAG) != 0)
    goto out;

//检查是否为 ELF 魔数 \x7FELF
if (memcmp(elf_ex->e_ident, ELFMAG, SELFMAG) != 0)
    goto out;

if (request_module("binfmt-%04x", *(ushort *)(bprm->buf + 2)) < 0)
    return retval;
if (request_module("binfmt-%04x", *(ushort *)(bprm->buf + 2)) < 0)
    return retval;

#define request_module(mod...) __request_module(true, mod)
#define request_module(mod...) __request_module(true, mod)

int __request_module(bool wait, const char *fmt, ...)
{
    va_list args;
    char module_name[MODULE_NAME_LEN];
    int ret, dup_ret;

    WARN_ON_ONCE(wait && current_is_async());
    if (!modprobe_path[0])
        return -ENOENT;

    va_start(args, fmt);
    ret = vsnprintf(module_name, MODULE_NAME_LEN, fmt, args);
    va_end(args);

    if (ret >= MODULE_NAME_LEN)
        return -ENAMETOOLONG;

    ret = security_kernel_module_request(module_name);
    if (ret)
        return ret;

    ret = down_timeout(&kmod_concurrent_max, MAX_KMOD_ALL_BUSY_TIMEOUT * HZ);
    if (ret) {
        pr_warn_ratelimited("request_module: modprobe %s cannot be processed, kmod busy with %d threads for more than %d seconds now",
                            module_name, MAX_KMOD_CONCURRENT, MAX_KMOD_ALL_BUSY_TIMEOUT);
        return ret;
    }

    trace_module_request(module_name, wait, _RET_IP_);

    if (kmod_dup_request_exists_wait(module_name, wait, &dup_ret)) {
        ret = dup_ret;
        goto out;
    }

    ret = call_modprobe(module_name, wait ? UMH_WAIT_PROC : UMH_WAIT_EXEC);

out:
    up(&kmod_concurrent_max);
    return ret;
}
EXPORT_SYMBOL(__request_module);

int __request_module(bool wait, const char *fmt, ...)
{
    va_list args;
    char module_name[MODULE_NAME_LEN];
    int ret, dup_ret;

    WARN_ON_ONCE(wait && current_is_async());
    if (!modprobe_path[0])
        return -ENOENT;

    va_start(args, fmt);
    ret = vsnprintf(module_name, MODULE_NAME_LEN, fmt, args);
    va_end(args);

    if (ret >= MODULE_NAME_LEN)
        return -ENAMETOOLONG;

    ret = security_kernel_module_request(module_name);
    if (ret)
        return ret;

    ret = down_timeout(&kmod_concurrent_max, MAX_KMOD_ALL_BUSY_TIMEOUT * HZ);
    if (ret) {
        pr_warn_ratelimited("request_module: modprobe %s cannot be processed, kmod busy with %d threads for more than %d seconds now",
                            module_name, MAX_KMOD_CONCURRENT, MAX_KMOD_ALL_BUSY_TIMEOUT);
        return ret;
    }

    trace_module_request(module_name, wait, _RET_IP_);

    if (kmod_dup_request_exists_wait(module_name, wait, &dup_ret)) {
        ret = dup_ret;
        goto out;
    }

    ret = call_modprobe(module_name, wait ? UMH_WAIT_PROC : UMH_WAIT_EXEC);

out:
    up(&kmod_concurrent_max);
    return ret;
}
EXPORT_SYMBOL(__request_module);

static int call_modprobe(char *orig_module_name, int wait)
{
    struct subprocess_info *info;
    static char *envp[] = {
        "HOME=/",
        "TERM=linux",
        "PATH=/sbin:/usr/sbin:/bin:/usr/bin",
        NULL
    };
    char *module_name;
    int ret;
    char **argv = kmalloc(sizeof(char *[5]), GFP_KERNEL);
    if (!argv)
        goto out;
    module_name = kstrdup(orig_module_name, GFP_KERNEL);
```
