
结果如下：

```bash
$ ./exploit
[\*] self path /home/hi/./exploit
[\*] prepare evil data
Old limits -> soft limit= 14096 hard limit= 14096
[\*] starting exploit, num of cores: 4
[1] defragmentation - spray 10000 `file` to exhaust all file slabs for cross cache
[2] spray (38+3)\*32 kmalloc-192 & kmalloc-256
[3] allocate the vulnerable filter
[4] spray kmalloc-192 & kmalloc-256
[5] free (end-24)\*32 kmalloc-192 & kmalloc-256
[6] 1st freed the filter object
[7] free (end-middle+1)\*32 kmalloc-192 & kmalloc-256
[8] spray 4000 uprivileged `file`
[9] 2nd free the filter object
pause after 2nd free
[10] spraying 5000 unprivileged `file`
[10-1] found overlapped file, id : 22, 1930
[11] start 2 threads compete to write
[11-1] start slow write
[11-2] write evil data after the slow write
[12] got cmd, start spraying 4096\*2 `file` by opening /etc/passwd
[\*] spray done
[\*] write done, spent 10.233072 s
[13] check whether we overwrite the privileged file
[+] succeed
$ su hi
Password:
# id
uid=0(hi) gid=0(root) groups=0(root)
# cat /etc/passwd
hi:x:0:0:root:/:/bin/sh
root::0:0:root:/root:/bin/bash
```

<a href="elink@923K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1M7$3q4#2j5$3g2Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0u0Q4x3V1j5I4x3q4)9J5c8U0t1I4i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3W2)9J5k6o6t1#2z5o6S2Q4x3V1j5`.">CVE-2022-2588 Double-free 漏洞 DirtyCred 利用</a>

<a href="elink@56eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8K9i4m8J5k6i4W2Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0u0Q4x3V1j5I4x3q4)9J5c8X3c8A6M7Y4c8&6i4K6u0V1j5%4u0W2k6q4)9J5c8R3`.`.">浅析 Linux Dirty Cred 新型漏洞利用方式</a>

# CVE-2022-2588 Dirty Cred

## 一、漏洞信息

### 1. 漏洞简述

- 漏洞名称：Dirty Cred
- 漏洞编号：CVE-2022-2588
- 漏洞类型：Double Free
- 漏洞影响：Linux 本地提权
- CVSS评分：7.8
- 基础权限：需要

### 2. 组件概述

漏洞函数是 `route4_change()`，用于初始化和替换 `route4_filter` 对象。使用 `handle` 作为id来区分不同的 `route4_filter`，如果存在某个 `handle` 之前已被初始化过（`fold` 变量非空），就会移除旧的 `filter`，添加新的 `filter`；否则直接添加新的 `filter`。

### 3. 漏洞利用

由于将 `route4_filter` 对象从链表中删除和释放时的检查条件不一致，导致该对象被释放后仍存于链表中，后面可以触发 Double Free，本地攻击者利用该漏洞会导致系统崩溃，可能会造成本地特权升级问题。

### 4. 漏洞影响

影响Linux Kernel版本：

- Linux Kernel版本 >= 2.6.12
- Linux Kernel版本 <= 5.19.1

### 5. 解决方案

[f2fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6H3N6h3u0Q4x3V1k6K6j5$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8X3N6A6N6q4)9J5c8Y4c8G2M7Y4k6S2L8r3c8K6i4K6u0r3L8r3W2F1N6i4S2Q4x3X3g2Y4K9i4c8Q4x3V1k6U0L8$3#2E0K9i4c8Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6W2S2k6o6x3$3x3K6l9&6k6e0t1%4x3e0W2S2z5o6R3@1k6U0V1@1y4U0j5%4z5r3f1H3x3U0V1$3j5X3f1I4x3r3j5`.](https://bbs.kanxue.com/elink@6c4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6H3N6h3u0Q4x3V1k6K6j5$3#2Q4x3V1k6D9K9h3&6#2P5q4)9J5c8X3E0W2M7X3&6W2L8q4)9J5c8X3N6A6N6q4)9J5c8Y4c8G2M7Y4k6S2L8r3c8K6i4K6u0r3L8r3W2F1N6i4S2Q4x3X3g2Y4K9i4c8Q4x3V1k6U0L8$3#2E0K9i4c8Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6W2S2k6o6x3$3x3K6l9&6k6e0t1%4x3e0W2S2z5o6R3@1k6U0V1@1y4U0j5%4z5r3f1H3x3U0V1$3j5X3f1I4x3r3j5`.)

## 二、环境搭建

### 安装 Kernel

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/kernel/v5.x/linux-5.19.1.tar.xz
tar -xvf linux-5.19.1.tar.xz
make menuconfig
make x86_64_defconfig
make bzImage -j32
```

怕麻烦的可以在bsauce大佬的仓库去取：[a40K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7$3q4#2j5$3g2Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1k6X3q4U0N6r3!0J5P5g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5h3W2F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3W2)9J5k6o6t1#2z5o6R3`.](https://bbs.kanxue.com/elink@359K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7$3q4#2j5$3g2Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1k6X3q4U0N6r3!0J5P5g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5h3W2F1i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3W2)9J5k6o6t1#2z5o6R3`.)

### 编译选项

- `CONFIG_BINFMT_MISC=y` （否则启动VM时报错）
- `CONFIG_USER_NS=y` （触发漏洞需要 User Namespace）
- `CONFIG_NET_CLS_ROUTE4=y` （漏洞函数所在的模块）
- `CONFIG_DUMMY=y` `CONFIG_NET_SCH_QFQ=y` （breezeO_o 提供的两个编译选项，触发 poc 需要用到）
- `CONFIG_NET_CLS_ACT=y` / `CONFIG_NET_CLS_BASIC=y` （默认已开启）
- `CONFIG_NET_SCH_SFQ=y` （exploit 中触发漏洞需用到 `sfq` 随机公平队列）
- `CONFIG_NET_EMATCH_META=y` （exploit 中堆喷对象时需要用到）

## 三、漏洞分析

### 1. 前置知识

#### 内核凭证 Credential

Kernel 凭证是 kernel 文档中定义的 kernel 中携带特权信息的特征，表示权限和对应的能力，主要分为：

- task 凭证（`struct cred`）：其中存放了一个 task 的权限信息，例如 GID、UID 等等。如果能任意修改一个低权限进程的 `cred` 结构体，那么我们就可以将该进程提权至高权限（例如 root）

  ```c
  struct cred {
      atomic_t    usage;
  #ifdef CONFIG_DEBUG_CREDENTIALS
      atomic_t    subscribers; /* number of processes subscribed */
      void        *put_addr;
      unsigned    magic;
  #define CRED_MAGIC  0x43736564
  #define CRED_MAGIC_DEAD 0x44656144
  #endif
      kuid_t      uid; /* real UID of the task */
      kgid_t      gid; /* real GID of the task */
      kuid_t      suid; /* saved UID of the task */
      kgid_t      sgid; /* saved GID of the task */
      kuid_t      euid; /* effective UID of the task */
      kgid_t      egid; /* effective GID of the task */
      kuid_t      fsuid; /* UID for VFS ops */
      kgid_t      fsgid; /* GID for VFS ops */
      unsigned    securebits; /* SUID-less security management */
      kernel_cap_t    cap_inheritable; /* caps our children can inherit */
      kernel_cap_t    cap_permitted; /* caps we're permitted */
      kernel_cap_t    cap_effective; /* caps we can actually use */
      kernel_cap_t    cap_bset; /* capability bounding set */
      kernel_cap_t    cap_ambient; /* Ambient capability set */
  #ifdef CONFIG_KEYS
      unsigned char   jit_keyring; /* default keyring to attach requested
                       * keys to */
      struct key  *session_keyring; /* keyring inherited over fork */
      struct key  *process_keyring; /* keyring private to this process */
      struct key  *thread_keyring; /* keyring private to this thread */
      struct key  *request_key_auth; /* assumed request_key authority */
  #endif
  #ifdef CONFIG_SECURITY
      void        *security; /* subjective LSM security */
  #endif
      struct user_struct *user; /* real user ID subscription */
      struct user_namespace *user_ns; /* user_ns the caps and keyrings are relative to. */
      struct group_info *group_info; /* supplementary groups for euid/fsgid */
      /* RCU deletion */
      union {
          int non_rcu; /* Can we skip RCU deletion? */
          struct rcu_head rcu; /* RCU deletion hook */
      };
  } __randomize_layout;
  ```
- open file 凭证（`struct file`）：存放一个文件的部分权限信息，例如 read & write 权限等。如果一个低权限用户可以任意修改高权限文件（例如 `/etc/passwd`），那么同样也能造成提权的目的

