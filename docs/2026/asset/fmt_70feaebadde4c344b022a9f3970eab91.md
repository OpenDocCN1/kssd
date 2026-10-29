
Linux CGroup 全称 Linux Control Group， 是 Linux 内核的一个功能，用来限制，控制与分离一个进程组群的资源（如 CPU、内存、磁盘输入输出等）。这个项目最早是由 Google 的工程师在 2006 年发起（主要是 Paul Menage 和 Rohit Seth），最早的名称为进程容器（process containers）。在 2007 年时，因为在 Linux 内核中，容器（container）这个名词太过广泛，为避免混乱，被重命名为 cgroup，并且被合并到 2.6.24 版的内核中去。

Linux Cgroup 主要提供以下功能

1、Resource limitation：限制资源的使用
- 例如：内存使用上限以及文件系统的缓存限制

2、Prioritization：优先级控制
- 例如：CPU 利用和磁盘的 IO 吞吐

3、Accounting 一些审计和一些统计

4、Control
- 挂起进程，恢复执行进程

**Cgroup 主要限制的资源**
- CPU
- 内存
- 网络
- 磁盘 I/O

![08c3da672eef1905666a05a566fcc4fa.png](https://bbs.kanxue.com/upload/attach/202204/809626_276KARRXBEJFASU.jpg)

## Cgroup 子系统

`cgroups` 的全称是 `control groups`，`cgroups` 为每种可以控制的资源定义了一个子系统。典型的子系统介绍如下：

1. `cpu` 子系统，主要限制进程的 cpu 使用率。
2. `cpuacct` 子系统，可以统计 `cgroups` 中的进程的 cpu 使用报告。
3. `cpuset` 子系统，可以为 `cgroups` 中的进程分配单独的 cpu 节点或者内存节点。
4. `memory` 子系统，可以限制进程的 memory 使用量。
5. `blkio` 子系统，可以限制进程的块设备 io。
6. `devices` 子系统，可以控制进程能够访问某些设备。
7. `net_cls` 子系统，可以标记 `cgroups` 中进程的网络数据包，然后可以使用 `tc` 模块（traffic control）对数据包进行控制。
8. `freezer` 子系统，可以挂起或者恢复 `cgroups` 中的进程。
9. `ns` 子系统，可以使不同 `cgroups` 下面的进程使用不同的 `namespace`。

## Cgroup 在什么时候创建

Linux 内核通过一个叫做 `cgroupfs` 的伪文件系统来提供管理 `cgroup` 的接口，我们可以通过 `lscgroup` 命令来列出系统中已有的 `cgroup`，该命令实际上遍历了 `/sys/fs/cgroup/` 目录中的文件:

|  |  |
| --- | --- |
| 1 | `lscgroup | tee cgroup.a` |

ps: 如果没有 `lscgroup` 的话，需要使用 `sudo apt install cgroup-tools`

![12b4edab5025c790f6b891eb6406a040.png](https://bbs.kanxue.com/upload/attach/202204/809626_GPGMAC5ZGS65YQH.jpg)

|  |  |
| --- | --- |
| 1 | `lssubsys -m` |

![94ab0d6ff962f5c49e39337a5a1092c6.png](https://bbs.kanxue.com/upload/attach/202204/809626_GFE4U533APTTWF9.jpg)

## Cgroup 如何限制资源

举个例子

![8eefa26950c917a2b3f69c321d9fe2e5.png](https://bbs.kanxue.com/upload/attach/202204/809626_786UCJ4ZQXT9P9E.jpg)

如果安装 `docker` 之后，在每个子系统下都会有一个 `docker` 的目录

![8ec11d6bb290716fbbce7c69b6a1178e.png](https://bbs.kanxue.com/upload/attach/202204/809626_F5ZCEHRDN7EG343.jpg)

其中的 4355 等都是 `docker` 容器，启动这个容器时，Docker 会为这个容器创建一个与容器标识符相同的 CGroup，在当前的主机上 CGroup 就会有以下的层级关系：

![1bf68aab2e78dfa6ab6df817e1d83e3c.png](https://bbs.kanxue.com/upload/attach/202204/809626_DX4M7ACMS6WHDYE.jpg)

每一个 CGroup 下面都有一个 `tasks` 文件，其中存储着属于当前控制组的所有进程的 pid，作为负责 cpu 的子系统，`cpu.cfs_quota_us` 文件中的内容能够对 CPU 的使用作出限制，如果当前文件的内容为 50000，那么当前控制组中的全部进程的 CPU 占用率不能超过 50%。

![3ba68dd272ee8dddde1d8685a0255ae2.png](https://bbs.kanxue.com/upload/attach/202204/809626_DJZVREQSKXF2V8A.jpg)

```
root@VM-0-14-ubuntu:/sys/fs/cgroup/cpu,cpuacct/docker# docker run -itd --cpu-quota=50000 alpine:latest
42e834d55ff2287aceb0d80bdf8a47a417f732e71fdb2419301c09ac6154e05d
root@VM-0-14-ubuntu:/sys/fs/cgroup/cpu,cpuacct/docker# cd 42e834d55ff2287aceb0d80bdf8a47a417f732e71fdb2419301c09ac6154e05d
root@VM-0-14-ubuntu:/sys/fs/cgroup/cpu,cpuacct/docker/42e834d55ff2287aceb0d80bdf8a47a417f732e71fdb2419301c09ac6154e05d# ls -alh
total 0
drwxr-xr-x 2 root root 0 Mar 31 17:53 .
drwxr-xr-x 13 root root 0 Mar 31 17:53 ..
-rw-r--r-- 1 root root 0 Mar 31 17:53 cgroup.clone_children
-rw-r--r-- 1 root root 0 Mar 31 17:53 cgroup.procs
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.stat
-rw-r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_all
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_percpu
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_percpu_sys
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_percpu_user
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_sys
-r--r--r-- 1 root root 0 Mar 31 17:53 cpuacct.usage_user
-rw-r--r-- 1 root root 0 Mar 31 17:53 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Mar 31 17:53 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Mar 31 17:53 cpu.shares
-r--r--r-- 1 root root 0 Mar 31 17:53 cpu.stat
-rw-r--r-- 1 root root 0 Mar 31 17:53 notify_on_release
-rw-r--r-- 1 root root 0 Mar 31 17:53 tasks
root@VM-0-14-ubuntu:/sys/fs/cgroup/cpu,cpuacct/docker/42e834d55ff2287aceb0d80bdf8a47a417f732e71fdb2419301c09ac6154e05d# cat cpu.cfs_quota_us
50000
root@VM-0-14-ubuntu:/sys/fs/cgroup/cpu,cpuacct/docker/42e834d55ff2287aceb0d80bdf8a47a417f732e71fdb2419301c09ac6154e05d#
```

## Cgroup 支持的文件种类

| 文件名 | R/W | 用途 |
| --- | --- | --- |
| Release_agent | RW | 删除分组时执行的命令，这个文件只存在于根分组 |
| Notify_on_release | RW | 设置是否执行 release_agent。为 1 时执行 |
| Tasks | RW | 属于分组的线程 TID 列表 |
| Cgroup.procs | R | 属于分组的进程 PID 列表。仅包括多线程进程的线程 leader 的 TID，这点与 tasks 不同 |
| Cgroup.event_control | RW | 监视状态变化和分组删除事件的配置文件 |

# 总结

## docker 的实现原理

其实 `docker` 就是一个 linux 下的进程，通过 Linux NameSpaces 对不同的容器进行隔离，为了保证宿主机与容器资源上的隔离，与资源占用的比例，所有使用 Cgroup 对进程进行资源上的限制或者控制

## docker 的实现原理(互联网)

1、docker 就是一个 linux 系统的进程, 它 **通过 Linux 的 namespaces 对不同的容器实现了资源隔离**，然后上面再跑一 rootfs 文件系统 **当容器使用的时候为了单个容器不榨干系统资源，所以就使用 cgroup 来做控制**。

## Linux NameSpace 和 Cgroup 解决了什么？

Linux 的 NameSpace 和 Cgroup 分别解决了不同资源隔离的问题，前者解决了进程、网络以及文件系统的隔离，后者实现了 CPU、内存等资源的隔离。

# 参考与引用

- [212K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8U0f1K6x3e0p5I4y4q4)9J5c8R3`.`.](https://bbs.kanxue.com/elink@3b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8U0f1K6x3e0p5I4y4l9`.`.)

- [6aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8%4m8@1j5h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8Y4y4W2M7r3q4J5j5i4c8A6L8$3&6Q4x3X3c8S2L8Y4S2A6k6i4c8&6i4K6u0V1K9i4y4G2L8r3q4@1K9h3&6Y4i4K6u0V1P5h3!0#2M7W2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2N6A6N6r3S2Q4x3X3c8D9K9h3&6#2P5q4)9J5k6r3&6S2L8h3g2K6M7r3q4U0k6i4x3`.](https://bbs.kanxue.com/elink@e8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8%4m8@1j5h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8Y4y4W2M7r3q4J5j5i4c8A6L8$3&6Q4x3X3c8S2L8Y4S2A6k6i4c8&6i4K6u0V1K9i4y4G2L8r3q4@1K9h3&6Y4i4K6u0V1P5h3!0#2M7W2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6s2N6A6N6r3S2Q4x3X3c8D9K9h3&6#2P5q4)9J5k6r3&6S2L8h3g2K6M7r3q4U0k6i4x3`.)

- [afbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6#2P5q4)9J5k6h3y4F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5k6o6x3H3y4U0S2Q4x3X3b7I4i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@885K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6#2P5q4)9J5k6h3y4F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5k6o6x3H3y4U0S2Q4x3X3b7I4i4K6u0W2K9s2c8E0L8l9`.`.)

- [b9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3!0D9M7$3S2W2L8r3I4Q4x3X3g2U0L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3x3e0M7H3x3e0m8Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@8c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3!0D9M7$3S2W2L8r3I4Q4x3X3g2U0L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3x3e0M7H3x3e0m8Q4x3X3g2Z5N6r3#2D9)

- [495K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3!0D9M7$3S2W2L8r3I4Q4x3X3g2U0L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3x3e0M7H3x3U0W2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@257K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3!0D9M7$3S2W2L8r3I4Q4x3X3g2U0L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3x3e0M7H3x3U0W2Q4x3X3g2Z5N6r3#2D9)
