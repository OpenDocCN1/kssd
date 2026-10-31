
### C 代码示例

```c
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
    if (!module_name)
        goto free_argv;
    argv[0] = modprobe_path;
    argv[1] = "-q";
    argv[2] = "--";
    argv[3] = module_name;
    argv[4] = NULL;
    info = call_usermodehelper_setup(modprobe_path, argv, envp, GFP_KERNEL,
            NULL, free_modprobe_argv, NULL);
    if (!info)
        goto free_module_name;
    ret = call_usermodehelper_exec(info, wait | UMH_KILLABLE);
    kmod_dup_request_announce(orig_module_name, ret);
    return ret;
free_module_name:
    kfree(module_name);
free_argv:
    kfree(argv);
out:
    kmod_dup_request_announce(orig_module_name, -ENOMEM);
    return -ENOMEM;
}
```

```c
char modprobe_path[KMOD_PATH_LEN] = CONFIG_MODPROBE_PATH;
```

```c
#ifdef CONFIG_STATIC_USERMODEHELPER
sub_info->path = CONFIG_STATIC_USERMODEHELPER_PATH;
#endif
```

```c
CONFIG_STATIC_USERMODEHELPER_PATH = "/sbin/usermode-helper";
```

### QEMU 启动命令

```bash
qemu-system-x86_64 \
-m 128M \
-kernel ./bzImage \
-initrd ./rootfs.cpio \
-append "root=/dev/ram rdinit=/init console=ttyS0 oops=panic panic=1 quiet rodata=off" \
-cpu qemu64,+smep,+smap \
-smp cores=2,threads=1 \
-nographic
```

### 文件系统操作

```bash
mkdir rootfs
cd rootfs
cpio -idvm < ../rootfs.cpio
```

### 初始化脚本

```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs devtmpfs /dev
echo "[*] Welcome to Kernel PWN Environment"
echo "[*] Starting shell..."
insmod /home/babydev.ko
chmod 777 /dev/noc /tmp
cp /proc/kallsyms /tmp/coresysms.txt
/home/eatFlag &
exec /bin/sh
# exec su -s /bin/sh ctf
```

[回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2026-1-3 02:50

被南行编辑

，原因：

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#内核](https://bbs.kanxue.com/forum-171-1-185.htm)
