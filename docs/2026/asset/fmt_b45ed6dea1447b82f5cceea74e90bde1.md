
提前返回就不会解压initramfs和initrd到rootfs，而是直接调用`default_rootfs()`在rootfs中创建出`/dev/console`文件，除此之外没有其他文件。
因此接下来的流程和启动方式1一样会进入到`prepare_namespace()`函数，只不过由于设置了`dm=`命令行参数会进入到`dm_setup`函数，以及调用后边的`dm_run_setup()`函数。接下来的流程会利用device mapper机制创建出虚拟块设备`dm-0`，将这个虚拟块设备文件做为根文件系统挂载，除此之外启动流程和上面所述的linux启动方式1是差不多的。

查看`/sys/block/dm-0/slaves`目录就可以看到它所对应的设备文件:

```
ls -l /sys/block/dm-0/slaves

lrwxrwxrwx 1 root root 0 2017-01-13 09:40 sda34 -> ../../../../soc/624000.ufshc/host0/target0:0:0/0:0:0:0/block/sda/sda34
```

如果内核命令行参数中没有设置`skip_initramfs`，那么`boot.img`中的ramdisk会被解压到rootfs中，就不会走到`prepare_namespace()`函数，而是直接跳到ramdisk中执行，也就是进入到recovery模式。

那么在system-as-root的情况下如何重新挂载system目录所在的分区呢?

具体的逻辑在`system/core/adb/remount_service.cpp`的`remount_partition()`函数中，由于现在`/proc/mounts`无法体现出`/`所在设备情况:

```
sailfish:/ # cat /proc/mounts

rootfs / rootfs rw,seclabel 0 0

/dev/root / ext4 ro,seclabel,relatime,data=ordered 0 0
```

需要进一步分析厂商的fstab文件才能得到`/`目录所在的设备，这里就是`/fstab.sailfish`文件：

```
#<src> <mnt_point> <type> <mnt_flags and options> <fs_mgr_flags>
/dev/block/bootdevice/by-name/system / ext4 ro,barrier=1,discard wait,slotselect,verify
```

知道`/`对应的设备以后执行重新挂载就可以了。

### Method C:

这种启动方式bootloader仍然会传递initrd给内核，这样initrd就会解压至rootfs中做为初始的根文件系统，并在上执行`/init`进程，而`/init`进程负责挂载system分区并且把它做为新的rootdir，并且进一步执行`/system/bin/init`程序完成系统启动的剩余步骤。

这种方式又被magisk作者称为两阶段的`ramdisk system-as-root`。在谷歌的角度只有Method B才被称为system-as-root，对于Method C它虽然也是将"system挂载为(as)root"，但是从启动的角度来说和Method B有本质的区别。所以在谷歌的官方文档中如果看到这样的描述，里边的system-as-root指的就是启动方式Method B:

> 搭载 Android 10 的设备不得使用 system-as-root。

从Method B进化到Method C，有部分原因是因为动态分区这一新特性的引入:

Android 10引入了动态分区的概念，因为A/B分区的存在，导致可用分区空间减少了很多，如果system,vendor,product等分区大小没有规划好，很有可能会影响到后边系统的升级。而动态分区会分配出super逻辑分区将system,vendor,product分区纳入其中，子分区可动态调整大小，各个分区映像不再需要为将来的 OTA 预留空间。

动态分区是通过Linux内核device-mapper框架的`dm-linear`插件实现的，bootloader和Linux内核无法解读super分区，因此无法自行装载system本身。所以Method B启动方式不再适用，需要新的启动方式就是Method C。

由于bootloader无法理解动态分区，因此无法对其进行刷写，安卓引入了另外一个新的功能:用户空间`fastbootd`，原来由闭源bootloader实现的功能，现在放在了recovery模式下的`fastbootd`实现，阅读`fastbootd`的代码还可以学习到`fastboot`协议实现的细节，`fastbootd`可以通过hal接口调用厂商特有的功能，用户空间`fastbootd`也算是Project Treble目标的一种体现。

以运行android 10的pixel 3xl举例，下载厂商二进制包放在android 10源码中编译以后，会生成`boot.img`和`ramdisk-recovery.img`，其中`boot.img`中的ramdisk正是`ramdisk-recovery.img`文件，新的内核可以在`/proc`文件系统中查看设备树的信息:

```
cat /proc/device-tree/chosen/linux,initrd-start | xxd

00000000: 0000 0000 8470 0000 .....p..

cat /proc/device-tree/chosen/linux,initrd-end | xxd

00000000: 0000 0000 8515 3be9
```

`0x85153be9 - 0x84700000 = 10828777`

而编译出来的`ramdisk-recovery.img`大小也正是10828777字节。

`ramdisk-recovery.img`中也有个`init`程序，它里边的`init`程序和编译到`system.img`中的`init`程序有什么区别？

`ramdisk-recovery.img`中的`init`是个指向`/system/bin/init`的软链接，`system/core/init/Android.bp`文件中有如下语句:

```
cc_binary {
    name: "init_second_stage",
    recovery_available: true,
    stem: "init",
    defaults: ["init_defaults"],
    static_libs: ["libinit"],
    required: [
        "e2fsdroid",
        "mke2fs",
        "sload_f2fs",
        "make_f2fs",
    ],
    srcs: ["main.cpp"],
    symlinks: ["ueventd"],
    target: {
        recovery: {
            cflags: ["-DRECOVERY"],
            exclude_shared_libs: ["libbinder", "libutils"],
        },
    },
    ldflags: ["-Wl,--rpath,/system/${LIB}/bootstrap"],
}
```

注意上面的`Android.bp`中声明了`recovery_available: true`。只要声明了`recovery_available: true`或者`recovery: true`的都会编译到`recovery`目录最终打包成`ramdisk-recovery.img`镜像:

> **Shared library support in recovery mode**
>
> In Android 10, shared libraries are available in the recovery partition, which eliminates the need for all recovery mode executables to be static. The shared libraries are located under the `/system/lib` (or `/system/lib64` for 64-bit devices) directory in the partition.
>
> To add a new shared library to the recovery partition, add `recovery_available: true` or `recovery: true` to `Android.bp` of the shared library. The former installs the library to both the system and recovery partitions, while the latter installs it only to the recovery partition.
>
> Shared library support can't be built with Android's make-based build system. To convert an existing static executable for the recovery mode to a dynamic one, remove `LOCAL_FORCE_STATIC_EXECUTABLE := true` in `Android.mk` or `static_executable: true` (in `Android.bp`).

由于指定了`stem`属性，所以生成的目标产物名称为`init`，而且`recovery`目录的`system/bin/init`程序它指定了`exclude_shared_libs`从而排除了`libbinder`和`libutils`库，因为不需要这两个库。

和启动方式2不同的地方是现在必须从initrd启动，所以之前用于指示是否进入recovery模式的`do_skip_initramfs`内核命令行参数不再使用，android 10引入了新的`androidboot.force_normal_boot`参数，它的值为1时表示进入正常的系统，否则才会进入recovery模式。和`init`第一阶段挂载相关还有新的内核命令行参数`androidboot.boot_devices`。

看一下启动流程，`ramdisk-recovery.img`会被解压到rootfs中并且执行`/init`，根据上面的描述`/init`的代码仍然编译自`system/core/init`目录，因此执行函数从`system/core/init/main.cpp`的`main()`函数开始:

```
FirstStageMain(){

....

if (ForceNormalBoot()) {

mkdir("/first_stage_ramdisk", 0755);

// SwitchRoot() must be called with a mount point as the target, so we bind mount the

// target directory to itself here.

if (mount("/first_stage_ramdisk", "/first_stage_ramdisk", nullptr, MS_BIND, nullptr) != 0) {

LOG(FATAL) << "Could not bind mount /first_stage_ramdisk to itself";

}

SwitchRoot("/first_stage_ramdisk");

}

....

if (!DoFirstStageMount()) {

LOG(FATAL) << "Failed to mount required partitions early ...";

}

...

const char* path = "/system/bin/init";

const char* args[] = {path, "selinux_setup", nullptr};

execv(path, const_cast<char**>(args));

...

}
```

`ForceNormalBoot()`用于判断是不是正常启动系统:

```
bool ForceNormalBoot() {

std::string cmdline;

android::base::ReadFileToString("/proc/cmdline", &cmdline);

return cmdline.find("androidboot.force_normal_boot=1") != std::string::npos;

}
```

如果是正常启动系统则将根切换到`/first_stage_ramdisk`目录:

`SwitchRoot`是通过move bind和`chroot`来实现切换根操作的。

`/first_stage_ramdisk`目录有如下文件:

```
drwxrwxr-x 4096 7月 18 2022 avb
-rw-rw-r-- 2015 7月 18 2022 fstab.sdm845
```

`avb`目录下存放着验证启动相关的公钥文件，是通过源码`system/core/rootdir/avb/`生成出来的。

`fstab.sdm845`是厂商相关的文件系统挂载表，由`device/google/crosshatch/device.mk`文件拷贝生成:

```
PRODUCT_COPY_FILES += \
device/google/crosshatch/fstab.hardware:$(TARGET_COPY_OUT_RECOVERY)/root/first_stage_ramdisk/fstab.$(PRODUCT_PLATFORM)
```

接下来执行`DoFirstStageMount()`函数进行第一阶段挂载，如果是recovery模式则跳过此阶段。

第一阶段挂载执行完以后就执行`/system/bin/init`程序，跳到下一阶段`selinux_setup`。此时的`/system/bin/init`已经不再是`ramdisk-recovery.img`中的`/system/bin/init`，实际上它执行的是`system.img`中的`/system/bin/init`程序。

看一下`DoFirstStageMount()`到底做了什么:

首先寻找挂载所需fstab文件，文件内容可以通过设备树传递，对于pixel3xl来说对应的路径为`/first_stage_ramdisk/fstab.sdm845`，只有标记为`first_stage_mount`的挂载点才会在这一阶段处理，以下是具有`first_stage_mount`标记的项:

```
# Android fstab file.
#<src> <mnt_point> <type> <mnt_flags and options> <fs_mgr_flags>
system /system ext4 ro,barrier=1 wait,slotselect,avb=vbmeta,logical,first_stage_mount
vendor /vendor ext4 ro,barrier=1 wait,slotselect,avb,logical,first_stage_mount
product /product ext4 ro,barrier=1 wait,slotselect,avb,logical,first_stage_mount
/dev/block/by-name/metadata /metadata ext4 noatime,nosuid,nodev,discard,sync wait,formattable,first_stage_mount
```

第一阶段装载的目标就是挂载`/system`、`/vendor`、`/product`和`/metadata`。

然后进入`FirstStageMount::DoFirstStageMount()`逻辑:

```
bool FirstStageMount::DoFirstStageMount() {

if (!IsDmLinearEnabled() && fstab_.empty()) {

// Nothing to mount.

LOG(INFO) << "First stage mount skipped (missing/incompatible/empty fstab in device tree)";

return true;

}

if (!InitDevices()) return false;

if (!CreateLogicalPartitions()) return false;

if (!MountPartitions()) return false;

return true;

}
```

`InitDevices()`主要做的事情是初始化一些数据结构，创建出`/dev/device-mapper`设备节点以及触发`uevent`事件创建出subsystem为`block`的设备节点。

触发`uevent`事件的逻辑是遍历`/sys/class`、`/sys/block`、`/sys/devices`目录下的`uevent`文件，向其中写入`"add\n"`，并且利用`netlink`机制获取内核传递过来的`uevent`消息。

这个过程处理完就会在`/dev/block`下创建出设备及分区的设备节点，以及一些符号链接，比如`system_a`分区的设备名为`sda5`，那么就会创建出`/dev/block/sda5`及如下符号链接:

```
/dev/block/platform/soc/1d84000.ufshc/by-name/system_a -> /dev/block/sda5
/dev/block/by-name/system_a  -> /dev/block/sda5
/dev/block/platform/soc/1d84000.ufshc/sda5   -> /dev/block/sda5
```
