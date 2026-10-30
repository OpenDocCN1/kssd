
# CVE-2021-4145 dirty cred漏洞复现

复现一下 dirty cred 漏洞

<!--more-->

同样本篇文章采用的还是 环境配置——漏洞验证——源码分析——代码调试 这四部分。

## 环境配置

### 内核编译

选用一个漏洞存在的版本，例如 `5.13.2`。

下面就是编译内核会踩得一些坑，我将完整复述一遍：

源码下载好之后，先 `make menuconfig` 开启调试符号，`kernel hacking->kernel debugging` 勾选，`kernel hacking->Compile-time checks and compiler options->Compile the kernel with debug info` 勾选。

保存退出之后还需要加上两个选项。

`vim .config`，打开之后找到两个选项，一个是 `CONFIG_FUSE_FS` 另一个是 `CONFIG_USER_NS`，这两个选项都需要启动，默认生成的 `config` 应该是没有启用这两个选项的。

配置完成之后就可以开始编译了。

编译完成之后，在本目录下得到带完整符号的 `vmlinux`，在 `arch/x86/boot/` 得到启动内核 `bzImage`

### 文件系统编译

依然是采用 busybox，方法和之前是一致的，看我最开始的 环境搭建 即可，这里可以提前把 EXP 编译进去然后打包文件系统。

### 启动脚本

就是传说中的 `start.sh`

这里给大家参考一下我的 qemu 启动参数。

```shell
qemu-system-x86_64 \
-m 256M \
-smp 2,cores=2,threads=1\
-kernel ./bzImage \
-initrd ./rootfs.img \
-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 quiet nokaslr"\
-cpu qemu64 \
-netdev user,id=t0, \
-device e1000,netdev=t0,id=nic0 \
-nographic \
#-s -S\
```

最后一行用于调试，大家不需要调试可以先注释掉，其它参数解释如下：

1. `-m 256M`: 指定虚拟机的内存大小为 256MB。

2. `-smp 2,cores=2,threads=1`: 指定使用 2 个 CPU，每个 CPU 拥有 2 个核心，每个核心只有一个线程。

3. `-kernel ./bzImage`: 指定了内核文件。

4. `-initrd ./rootfs.img`: 指定我们制作的 Linux 文件系统。

5. `-append "root=/dev/ram rw console=ttyS0 oops=panic panic=1 quiet nokaslr"`:指定了传递给内核的启动参数。这里的 quiet 可以让内核不输出很多信息直接启动，`nokaslr` 一定要加，否则断点无法命中。

6. `-cpu qemu64`: 指定使用 QEMU 的默认 x86_64 CPU 模拟器。

7. `-netdev user,id=t0,`: 指定了用户模式网络设备。

8. `-device e1000,netdev=t0,id=nic0`: 指定了要添加到虚拟机的网络设备。

9. `-nographic`: 无需图形界面的情况下运行 QEMU。

---

现在在目录下应该有了 `start.sh`，`bzImage` 和 `rootfs.img`，文件系统可以提前打包 exp 进去。

## 漏洞验证

### EXP验证

用网上通用的一个 EXP。
