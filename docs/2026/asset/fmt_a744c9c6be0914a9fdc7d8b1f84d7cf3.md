
如果有输出，则表示 overlay 文件系统已经被内核模块加载；如果输出为空，则表示没有加载 overlay 模块。

### 检查内核配置

```bash
grep CONFIG_OVERLAY_FS /boot/config-$(uname -r)
```

```text
CONFIG_OVERLAY_FS=m
# CONFIG_OVERLAY_FS_REDIRECT_DIR is not set
CONFIG_OVERLAY_FS_REDIRECT_ALWAYS_FOLLOW=y
# CONFIG_OVERLAY_FS_INDEX is not set
CONFIG_OVERLAY_FS_XINO_AUTO=y
# CONFIG_OVERLAY_FS_METACOPY is not set
```

根据输出的结果，可以看到 `CONFIG_OVERLAY_FS` 的值为 `m`，表示该文件系统功能已编译到模块中，可以在需要时加载该模块来使用 Overlay 文件系统，该功能已经开启。（`m` 或者 `y` 都代表已开启）

# 总结

通过学习和复现此漏洞的细节，希望大家能够通过本文了解学习 `fuseFS` 和 `overlayFS` 的操作机制并懂得如何利用此漏洞提权。在写本文时我也发现了自身很多不足之处。如果在文中发现哪处存在问题，也欢迎大家在留言区指出。

# 参考链接

<mark class="encrypted">fffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3g2F1j5h3!0@1K9h3q4F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6l9K6z5o6j5`.</mark>

<mark class="encrypted">fe5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9$3q4F1k6h3W2C8K9g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0y4Q4x3X3b7H3x3K6R3$3</mark>

<mark class="encrypted">6c1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4#2j5$3E0&6j5i4m8H3L8r3f1I4x3o6t1^5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5o6l9%4y4e0x3#2z5l9`.`.</mark>

# 概述

# 漏洞信息

- CVE ID：`CVE-2023-0386`（`overlay`）
- 影响版本：`Linux 5.11 ~ 5.19`，不包括 `5.15`
- 漏洞类型：`LPE`
- 补丁文件：[fs/overlayfs/copy_up.c](https://bbs.kanxue.com/elink@076K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6H3N6h3u0Q4x3V1k6K6j5$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8X3N6A6N6q4)9J5c8Y4c8G2M7Y4k6S2L8r3c8K6i4K6u0r3L8r3W2F1N6i4S2Q4x3X3g2Y4K9i4c8Q4x3V1k6V1K9h3k6X3i4K6u0r3k6Y4y4Q4x3V1k6G2N6X3g2J5L8r3q4&6k6Y4y4Q4x3V1k6U0L8%4m8&6i4K6g2X3N6i4m8Q4x3X3g2U0i4K6y4r3K9h3c8Q4x3@1b7@1k6U0p5I4j5h3c8S2x3e0m8V1x3r3p5`.)

> **Description:** A flaw was found in the Linux kernel, where unauthorized access to the execution of the setuid file with capabilities was found in the Linux kernel’s OverlayFS subsystem in how a user copies a capable file from a nosuid mount into another mount. This uid mapping bug allows a local user to escalate their privileges on the system.

这个漏洞是 `overlay` 内核组件存在逻辑问题，导致具有 `suid` 权限的文件可以未授权访问，最后实现本地提权。

# 漏洞分析

上篇文章我将内核版本切换到了 `5.16.12`（看 `chenaotian` 师傅的帖子说是 `5.15` 存在问题无法复现，不巧的是我一直用的都是 `5.15`，的确无法使用）。

分析漏洞之前先来看看组件是做什么的：Overlay 是一种 Linux 文件系统组件，用于实现一种可写的文件系统层（upper layer），并通过挂载两个文件系统来实现文件层次结构的组合。具体来说，它是一种用于容器和虚拟化环境的轻量级文件系统，通常用于在只读根文件系统之上创建一个可写层，以便应用程序可以在其中进行更改而不会影响到根文件系统。下图是 overlay 的分层结构：

![](https://bbs.kanxue.com/upload/attach/202305/951654_6RFEU7XGRDXJ9JJ.png)

1. **Merged 层**：这是一个虚拟的、合成的视图，它将 lower 层和 upper 层合并在一起。当用户访问 Overlay 文件系统时，他们看到的是这个 merged 层。在这个层中，来自 upper 层的更改会覆盖 lower 层的相应文件。对于同名文件，upper 层中的文件优先级更高。对于同名目录，则合并，只判断目录中的文件是否有上下层覆盖屏蔽关系。【用户最终看到的是这个视图】

2. **Upper 层**：这是一个可写的文件系统层，它存储所有对 lower 层文件的更改。这包括文件修改、创建和删除操作。

3. **Workdir 目录**：这是一个与 upper 层在同一文件系统中的可写目录，用于存储一些中间数据和元数据，以支持 OverlayFS 的正常运行。

4. **Lower 层**：这是基础文件系统层，只读。一个 Overlay 文件系统可以有一个或多个 lower 层。此层拥有写时复制特性（copy-up），当想要修改 lower 层不可修改的 `fileC` 时，先拷贝副本到 upper 层，修改后呈现到 merged 视图。

### OverlayFS 操作流程

#### 挂载 OverlayFS 文件系统

```bash
mkdir lower upper work merged
sudo mount -t overlay -o lowerdir=lower,upperdir=upper,workdir=work overlay merged
```

```text
# 挂载 OverlayFS
# mount 需要 root 用户权限；需要创建两个 lower 时：lowerdir=lower1:lower2
# 卸载 OverlayFS：sudo umount merged
```

![](https://bbs.kanxue.com/upload/attach/202305/951654_FGRYH8KSEHAFMU6.png)

上面的命令可以将 `lowerdir` 和 `upper` 目录堆叠到 `merged` 目录，`workdir` 工作目录要求是和 `upperdir` 目录同一类型文件系统的空目录，`workdir` 是 OverlayFS 内部使用的工作目录，不需要用户自己创建。`lower`、`upper` 和 `merged` 等路径需要替换为实际存在的路径，我们需要在根目录下创建这三个目录。

#### 各层新建文件对比

在 lower 层新建文件：

```bash
echo "This is a file in lower." > lower/test.txt
```

当我们在 lower 层新建文件后，可以看到新文件也出现在 merged 目录，upper 目录无变化。

```text
[+] lower
[+] merged
[ ] upper
```

![](https://bbs.kanxue.com/upload/attach/202305/951654_7ZQEFUV9TVAUNZJ.png)

在 merged 层新建文件：

```bash
echo "This is a file in merged." > merged/merged.txt
```

当我们在 merged 层新建文件后，可以看到新文件也出现在 upper 目录，lower 目录无变化。

```text
[+] merged
[+] upper
[ ] lower
```

![](https://bbs.kanxue.com/upload/attach/202305/951654_VMNCE7JR94V672S.png)

在 upper 层新建文件：

```bash
echo "This is a file in upper." > upper/upper.txt
```

当我们在 upper 层新建文件后，可以看到新文件也出现在 merged 目录，lower 目录无变化。

```text
[+] merged
[+] upper
[ ] lower
```

![](https://bbs.kanxue.com/upload/attach/202305/951654_3MUYPJHUTM9ATXX.png)

#### 在 merged 层修改文件的变化

merged 层是一个统一的视图，当在 merged 新建文件时，可以看到新文件也出现在 upper 目录，lower 目录无变化。也就是 merged 和 upper 是相互作用的。

**在 merged 层中：**

来自 upper 层的更改会覆盖 lower 层的相应文件。对于同名文件，upper 层中的文件优先级更高。对于同名目录，则合并，只判断目录中的文件是否有上下层覆盖屏蔽关系。

#### 注意事项

需要注意以下卸载 OverlayFS 后各层文件的变化，这也是后续漏洞利用的重要信息。

挂载 OverlayFS 后，在 lower 层新增文件，merged 也会新增文件，upper 无变化；从 merged 层修改 lower 层文件，upper 新增修改后的文件，接着卸载 OverlayFS，merged 文件会消失，upper 保留修改后的文件。

挂载 OverlayFS 后，在 lower 层新增文件，merged 也会新增文件，upper 无变化：

```bash
ubuntu@ubuntu:~/Downloads/fuse/change$ touch lower/lower.txt
ubuntu@ubuntu:~/Downloads/fuse/change$ tree
.
├── lower
│   └── lower.txt
├── merged
│   └── lower.txt
├── upper
└── work
    └── work [error opening dir]
```

从 merged 层修改 lower 层文件，upper 新增修改后的文件，接着卸载 OverlayFS，merged 文件会消失，upper 保留修改后的文件：

```bash
ubuntu@ubuntu:~/Downloads/fuse/change$ tree
.
├── lower
│   └── lower.txt
├── merged
├── upper
│   └── lower.txt
└── work
    └── work [error opening dir]
```

保留下来的文件权限跟 lower 层是一样的：

```bash
ubuntu@ubuntu:~/Downloads/fuse/change$ ls -al lower/lower.txt
-rw-rw-r-- 1 ubuntu ubuntu 4 May 12 03:29 lower/lower.txt

ubuntu@ubuntu:~/Downloads/fuse/change$ ls -al upper/lower.txt
-rw-rw-r-- 1 ubuntu ubuntu 10 May 12 03:47 upper/lower.txt
```

### 漏洞触发原理

#### 漏洞的逻辑

OverlayFS 所有文件都可以正常拷贝，包括没有在当前用户命名空间中映射的文件（也就是 nobody 文件）。

拷贝文件并不只是拷贝文件的内容，包括文件的元数据，也就是文件的属主信息、时间戳、权限信息，还有扩展信息如 capabilities 等都会一起拷贝过来。引发的风险就是，如果下层文件系统是一个用户文件系统（如 FUSE），用户高度可控，可以自定义任何文件，但该文件系统存在限制（如 nosuid），那么本漏洞就允许将下层用户自定义的 suid 文件从一个 nosuid 文件系统拷贝到一个正常文件系统中，导致非法的 suid 文件获得 suid 特权，进而造成提权。

> `nosuid` 是 Linux 文件系统中的一种标志，它是在挂载文件系统时使用的选项之一。如果一个文件系统被标记为 `nosuid`，则在该文件系统上的任何文件或目录上设置的 `setuid` 或 `setgid` 位都会被忽略，这意味着即使一个可执行文件被设置为具有特权用户的权限，也不会在执行该文件时获得特权，从而提高了系统的安全性。

`nosuid` 是在挂载时对文件系统进行了限制，但是文件系统可以生成一个 `suid` 的文件，即使因为限制不起作用。

#### 用户命名空间

什么是用户命名空间？

> 用户命名空间：用户命名空间（User Namespace）用于隔离用户 ID（UID）和组 ID（GID）。可以思考一下 C++ 中的命名空间，假设一个程序中用了两个相同函数名的函数（不是指重载），就会出现函数名冲突的问题，用命名空间就可以调用时避免冲突。

> 具体来说，假设在主机上存在一个名为 `james` 的用户，其 UID 为 0。如果直接使用 Overlay 进行文件系统叠加，并在容器中以 `james` 用户身份启动进程，该进程将会在容器中使用 UID 为 0 的用户，与主机中的 `james` 用户相同，容易造成权限混淆的问题。

> 使用命名空间后，一个容器中的 root 用户（UID 0）可能在主系统中被映射为一个非特权用户，起到了权限隔离的作用。

> 主要使用命令：`unshare`

**动手实践：**

- 创建 nobody 新用户命名空间并查看权限。

```bash
unshare -U /bin/bash
```

执行该命令后，将会创建一个新的用户命名空间，并在其中启动一个新的 Bash shell。在新的用户命名空间中，用户 ID 和组 ID 会重新映射，以便用户在该命名空间中获得新的权限。

输出如下：

```text
ubuntu@ubuntu:~/Downloads/test_overlay$ sudo unshare -U /bin/bash
[sudo] password for ubuntu:
nobody@ubuntu:/home/ubuntu/Downloads/test_overlay$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
nobody@ubuntu:/home/ubuntu/Downloads/test_overlay$
```

此处看到新用户命名空间中的 UID 为 nobody。

> nobody 是一个特殊的系统用户，通常被用作一个不需要拥有系统特权的用户来运行某些服务或进程。在 Linux 中，每个用户都有一个对应的 UID（User ID），而 nobody 用户的 UID 通常是 65534。当一个进程运行在一个不同的用户命名空间时，由于该命名空间中没有与该 UID 相关联的用户名，所以该 UID 会被识别为 nobody 用户。因此，当您在一个新的用户命名空间中使用 `unshare` 命令创建一个新的 shell 时，该 shell 中的 UID 会被识别为 nobody。（nobody 权限比普通用户低）

- 创建 root 权限的新用户命名空间，获取的 root 权限只限于新用户命名空间。
