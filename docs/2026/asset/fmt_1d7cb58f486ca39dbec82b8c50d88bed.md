
```
#!/bin/sh

exec tail -n +3 $0

# This file provides an easy way to add custom menu entries.  Simply type the

# menu entries you want to add after this comment.  Be careful not to change

# the 'exec tail' line above.

menuentry 'Ubuntu, KGDB with nokaslr' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-bf306d0a-28c8-49c6-bffc-446be272ddcf' {

    recordfail

    load_video

    gfxmode $linux_gfx_mode

    insmod gzio

    if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi

    insmod part_msdos

    insmod ext2

    set root='hd0,msdos1'

    if [ x$feature_platform_search_hint = xy ]; then

      search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  bf306d0a-28c8-49c6-bffc-446be272ddcf

    else

      search --no-floppy --fs-uuid --set=root bf306d0a-28c8-49c6-bffc-446be272ddcf

    fi

    echo 'Loading Linux 4.10.0-19 with KGDB built by GEDU lab...'

    linux /boot/vmlinuz-4.10.0-19-generic root=UUID=bf306d0a-28c8-49c6-bffc-446be272ddcf ro find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US quiet kgdbwait kgdb8250=io,03f8,ttyS0,115200,4 kgdboc=ttyS0,115200 kgdbcon nokaslr

    echo 'Loading initial ramdisk ...'

    initrd    /boot/initrd.img-4.10.0-19-generic

}
```

更新grub后，重启按住SHIFT 进入grub选择刚才的menu即可等到进入被调试模式。

```
sudo update-grub 

reboot
```

一开始并没有加入`nokaslr`选项时。会存在不能下断点，`bt`栈回溯无法获取地址对于的文件信息。本来以为是linux内核的编译选项导致，这里也有人遇到了同样的情况（8a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6#2P5q4)9J5k6h3y4F1i4K6u0r3k6X3!0J5N6h3#2Q4x3X3g2H3K9s2m8Q4x3@1k6E0L8$3c8Q4x3@1c8$3K9h3g2%4N6r3S2J5k6h3q4V1i4K6t1$3j5h3#2H3i4K6y4n7N6r3W2V1i4K6y4p5x3e0j5J5y4o6y4Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

## 配置debugging

1. 设置串口通信波特率：

   ```
   stty -F /dev/ttyS0 115200
   ```

2. 编写`gdb` config，在`vmlinux`时，可以通过`source`加载。

   ```
   set architecture i386:x86-64:intel

   target remote /dev/ttyS0
   ```

3. 使用`gdb`来调试带符号的`vmlinux`，这时`gdb`就会尝试使用串口`/dev/ttyS0`来调试。

   ```
   gdb -s /usr/lib/debug/boot/vmlinux-4.10.0-19-generic

   gdb> source config
   ```

符号加载完成，`bt`查看当前栈帧，`c`运行内核。

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_RE25HMZBBA4JUWX.png)

### 关于源码的查看

在`gdb`中使用`l`可以查看相应的源码，但是由于`vmlinux`编译时似乎是使用绝对路径编译的，因此可以建立相应的绝对路径把source code放进去。还有一种方法是，设置`set substitute-path PATH1 PATH2`，`PATH1`是`vmlinux`中的路径信息，`PATH2`是source code存放的真实路径。

### 单步调试

`c`运行内核后，debuggee就跑起来了。这时，基本的`gdb`命令都可以使用，但是如果用`n`单步运行时，可能过了很久都没有反应。举个栗子：

1. 在debugging随便下一个断`break ip_mc_leave_src`，后`c`运行debuggee。

2. 在debuggee中运行poc，在断点处停了。

3. 在debugging中，使用`n`单步，则发现`gdb`处于等待状态。debugeee也处于阻塞状态：

   ![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_U9WPCP6NF4YV7QB.png)

- debugging

  ![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_55VUCSR85RKNGE3.png)

经过一番探索之后，找到了解决方案：

1. debuggee> 使用`echo g > "/proc/sysrq-trigger"` 进入kgdb的被调试状态；

   `sudo su && echo g > "/proc/sysrq-trigger"`

2. debugging> 下断点后`c`继续运行debuggee；

3. debuggee> 运行poc触发断点；

4. debugging> 可以使用`n`单步调试。

注：debuggee如果不能进入kgdb中断，可用`cat /sys/module/kgdboc/parameters/kgdboc`检查的`kgdboc`参数是否设置正确。

sysrq还有很多强大的功能，具体可以参考这里：e65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3K9Y4g2K6N6r3W2F1i4K6u0V1P5g2)9J5k6r3I4A6L8W2)9J5c8Y4m8Q4x3V1j5#2y4o6t1@1y4e0f1#2i4K6u0W2K9s2c8E0L8l9`.`.

```
#查看

cat /proc/sys/kernel/sysrq 

#配置

echo X >/proc/sys/kernel/sysrq 

#或修改/etc/sysctl.conf中设置kernel.sysrq=1

echo m > /proc/sysrq-trigger 导出内存分配信息

echo t > /proc/sysrq-trigger 导出当前任务状态信息

echo p > /proc/sysrq-trigger 导出当前CPU寄存器和标志位信息

echo c > /proc/sysrq-trigger 产生空指针panic事件，人为导致系统崩溃

echo s > /proc/sysrq-trigger 即时同步所有挂载的文件系统

echo u > /proc/sysrq-trigger 即时重新挂载所有的文件系统为只读

echo w > /proc/sysrq-trigger转储处于uninterruptable阻塞状态的任务
```

### 转向kdump

调试后期，由于借助kgdb来调试时，一旦执行到用户态的shellcode时，`gdb`中就会输出大量的垃圾信息，这时可以借助kdump来看oops信息，使用过程如下：

```
#安装kdump

sudo apt-get install  linux-crashdump

service kdump-tools start

#检查kdump是否可用

查看/boot/grub/grub.cfg是否含有crashkernel

如果是自定义启动项，手工修改/etc/grub.d/40-custom中增加crashkernel

#检查另一个kernel是否启动

/sys/kernel/kexec_crash_loaded 1表示已经加载 0表示没有加载

#测试

echo c > "/etc/sysrq-trigger"

....等待重启后在/var/crash下生成了日志

解压 crash  /usr/lib/debug/boot/vmlinux-4.10.0-19-generic  /tmp/kernel-crash

查看/tmp/kernel-crash/VmCoreDmesg信息即可。
```

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_NVFCSD3KDQZKSGW.png)

### success

至此调试环境搭建完成。（由于现在换到windows下调试了，补一张windows的vmware截图：

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_5HETA6WCD9C63ZX.png)

## 0xFF 运行poc查看崩溃信息

内核跑起来后，在debuggee中运行[CVE-2017-8890的poc](https://bbs.kanxue.com/elink@85eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2A6L8X3E0&6j5%4S2Q4x3V1j5H3k6X3j5H3x3h3g2S2y4e0c8T1j5$3b7#2k6U0j5K6y4K6W2S2k6h3g2S2z5e0b7^5x3$3k6S2x3U0k6W2z5b7`.`.)，发现程序崩溃了。oops信息可以在`gdb`中看到，如果在`gdb`中看不到，可以尝试用CTRL+SHIFT+F1/F7 切换到命令行模式，可以看到很完整的oops信息。

![Alt text](https://bbs.kanxue.com/./upload/attach/201901/730610_BD9EE5F8PAP9C24.png)

查看崩溃点`<ip_mc_leave_src+37>: mov ecx,DWORD PTR [rbx+0x4]`，此时`rbx`为0x2，因此从0x6中取值失败，存在对空指针引用。
