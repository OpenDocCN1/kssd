
- [交叉编译工具](https://bbs.kanxue.com/elink@646K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5k6h3I4A6L8X3q4J5L8#2)9J5k6h3!0J5k6#2)9J5c8X3y4G2L8i4m8G2L8X3g2F1N6s2y4Q4x3V1k6@1L8$3!0D9j5$3S2S2K9h3&6Q4x3V1k6T1K9h3&6S2M7X3W2W2M7#2)9J5c8U0c8Q4x3X3f1&6i4K6u0V1x3U0l9I4y4#2)9J5k6e0l9I4i4K6u0r3j5i4u0E0i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8Y4L8Y4g2W2j5h3u0A6K9r3k6Q4x3V1j5`.)

> 补充：笔者的操作系统环境
>
> Windows11+WSL2

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `┌──(root㉿W1sh)``-``[~]`  `└─``# lsb_release -a`  `No LSB modules are available.`  `Distributor` `ID``: Kali`  `Description:    Kali GNU``/``Linux Rolling`  `Release:` `2024.1`  `Codename:       kali``-``rolling`    `┌──(root㉿W1sh)``-``[~]`  `└─``# uname -a`  `Linux W1sh` `5.15``.``146.1``-``microsoft``-``standard``-``WSL2` `#1 SMP Thu Jan 11 04:09:03 UTC 2024 x86_64 GNU/Linux` |

# 1.2 编译Linux内核

## 1.2.1 安装交叉编译工具

> 如果您的apt工具可以下载到低版本的gcc↓

|  |  |
| --- | --- |
| 1  2 | `apt install gcc``-``arm``-``linux``-``gnueabi` `#armel`  `apt install gcc``-``arm``-``linux``-``gnueabihf` `#armhf` |

本篇我们需要用到的架构是`armhf`，所以需要安装`gcc-arm-linux-gnueabihf`

> 由于笔者的内核版本较高，最低只能下载到`gcc-11`，所以需要手动[下载编译工具](https://bbs.kanxue.com/elink@e8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5k6h3I4A6L8X3q4J5L8#2)9J5k6h3!0J5k6#2)9J5c8X3y4G2L8i4m8G2L8X3g2F1N6s2y4Q4x3V1k6@1L8$3!0D9j5$3S2S2K9h3&6Q4x3V1k6T1K9h3&6S2M7X3W2W2M7#2)9J5c8U0c8Q4x3X3f1&6i4K6u0V1x3U0l9I4y4#2)9J5k6e0l9I4i4K6u0r3j5i4u0E0i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8Y4L8Y4g2W2j5h3u0A6K9r3k6Q4x3V1j5`.)交叉编译工具
>
> 具体的`GCC`版本请根据您要编译的Linux内核进行区分(检查文件`linux-x.x.x/include/linux/compiler-gccX.h`)

## 1.2.2 下载Linux源码

本篇使用的Linux内核版本是`3.2.1`，如有需要请自行[下载](https://bbs.kanxue.com/elink@546K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2k6h3c8Y4k6g2)9J5k6h3E0W2M7X3&6W2L8q4)9J5k6h3!0J5k6#2)9J5c8Y4m8#2j5W2)9J5c8X3I4A6L8Y4g2^5i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3)其它版本

## 1.2.3 编译Linux内核

### 准备配置文件

内核源码中针对不同架构和开发板型号已经提供了不同的模板配置文件(`arch/${arch}/configs`)，我们可选择一项先生成一份`.config`，这里我们选择的处理器型号是`vexpress`

|  |  |
| --- | --- |
| 1 | `make CROSS_COMPILE``=``arm``-``linux``-``gnueabihf``-` `ARCH``=``arm vexpress_defconfig` |

### 修改配置文件

针对本篇的固件模拟，编译内核前我们需要对配置文件进行一些修改

- 避免文件系统只读：`CONFIG_LBDAF=y`
- 确保web服务正常启动：`CONFIG_IPV6=y`

#### 设置内核启动参数（内核版本小于4请指定！）

由于`v3.2.1`的内核不支持通过启用配置项`CONFIG_ATAGS`来从命令行中获取内核启动参数，所以我们需要在配置文件中通过字段`CONFIG_CMDLINE`来提前设置（使用qemu -append参数无效）

这是笔者的参数设置

|  |  |
| --- | --- |
| 1  2  3 | `CONFIG_CMDLINE``=``"root=/dev/mmcblk0p2 rw console=ttyAMA0 mem=128M"`  `CONFIG_IPV6``=``y`  `CONFIG_LBDAF``=``y` |

##### 关于mem=128M（内核版本小于4）

如果不指定此项在启动服务时系统会因为内存溢出而崩溃

### 编译

|  |  |
| --- | --- |
| 1 | `make CROSS_COMPILE``=``arm``-``linux``-``gnueabi``-` `ARCH``=``arm` |

#### 关于编译时的报错

> 提前声明，下述解决办法并不一定能100%让您的内核被成功编译**并且启动**，如遇报错，请第一时间检查您的GCC版本与内核版本是否匹配

##### [libz.so.1: cannot open shared object file](https://bbs.kanxue.com/elink@b30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3U0p5J5y4e0j5^5y4U0k6Q4x3V1k6D9K9h3u0*7i4K6u0V1M7$3!0Q4x3X3b7I4i4K6u0V1j5$3q4F1L8X3!0@1i4K6u0V1L8%4m8W2L8W2)9J5k6s2y4Z5j5i4u0W2k6q4)9J5k6r3!0T1K9X3g2U0N6q4)9J5k6r3k6A6L8r3f1`.)

|  |  |
| --- | --- |
| 1  2  3 | `apt``-``get install zlib1g`  `apt``-``get install zlib1g:i386`  `apt``-``get install libc6``-``i386 lib32stdc``+``+``6` `lib32gcc1 lib32ncurses5` |

##### [arch/arm/kernel/return_address.c:66:7: error: redefinition of ‘return_address’](https://bbs.kanxue.com/elink@60fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0b7#2x3U0f1@1z5e0c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1K6y4U0l9^5x3U0M7$3)

|  |  |
| --- | --- |
| 1  2  3 | `@arch``/``arm``/``include``/``asm``/``ftrace.`  `-` `extern inline void` `*``return_address(unsigned` `int` `level)`  `+` `static inline void` `*``return_address(unsigned` `int` `level)` |

|  |  |
| --- | --- |
| 1  2  3  4  5 | `@arch``/``arm``/``kernel``/``return_address.c`  `-` `void` `*``return_address(unsigned` `int` `level)`  `-` `{`  `-`   `return` `NULL;`  `-` `}` |

##### [Can’t use ‘defined(@array)’ (Maybe you should just omit the defined()?)](https://bbs.kanxue.com/elink@b2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7@1y4K6b7^5x3e0t1%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6R3#2x3K6b7H3x3o6t1`.)

|  |  |
| --- | --- |
| 1  2  3 | `@kernel``/``timeconst.pl`  `-` `if``(!defined(@val))`  `+` `if``(!@val)` |

## 1.2.4 启动虚拟机

编译完成后我们得到了内核文件(`arch/${arch}/boot/zImage`)，另外我们还需要设备树文件，Linux源码中提供了部分这样的文件(`arch/${arch}/boot/dts`)，若未找到可自行[下载](https://bbs.kanxue.com/elink@1a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3k6X3c8W2N6W2)9J5k6o6g2Q4x3V1k6I4k6h3#2#2i4K6u0V1M7Y4m8A6x3W2)9J5k6s2k6W2P5s2m8J5k6i4y4K6)

### 准备文件系统

这里我们使用从固件中解包得到的文件系统，首先创建一个磁盘镜像

|  |  |
| --- | --- |
| 1 | `qemu``-``img create` `-``f raw disk.img` `512M` |

然后将其格式化，这里注意您的内核版本支持的文件系统格式，笔者使用的是`3.2.1`版本内核且编译时没有开启`ext4`文件系统支持，所以使用`ext3`

|  |  |
| --- | --- |
| 1 | `mkfs` `-``t ext3 .``/``disk.img` |

创建一个临时文件夹作为挂载点，并将解包得到的文件系统复制进其中

|  |  |
| --- | --- |
| 1  2  3  4 | `mkdir tmpfs`  `sudo mount` `-``o loop .``/``disk.img tmpfs``/`  `sudo cp` `-``r rootfs``/``*` `tmpfs``/`  `sudo umount tmpfs` |

### 启动

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `sudo qemu``-``system``-``arm \`  `-``M` `"vexpress-a9"` `\` `#指定开发板型号`  `-``kernel` `"/kernel/linux/3.1.2/arch/armhf/zImage"` `\` `#指定内核文件`  `-``dtb` `"/kernel/dtb/vexpress-v2p-ca9.dtb"` `\` `#指定设备树文件`  `-``sd` `"./disk.img"` `\` `#指定根文件系统`  `-``net nic \`  `-``net tap,ifname``=``tap0,script``=``no,downscript``=``no \` `#在主机上添加一个网络接口tap0`  `-``nographic \` `#无界面启动`  `-``smp` `4` `#指定cpu核心数量（根据具体的开发板型号支持数量定，太低会很卡` |

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_MQTKJMY3MNFBQFN.webp)

#### 网络配置

##### 主机

|  |  |
| --- | --- |
| 1 | `ifconfig tap0` `192.168``.``11.1``/``24` `up` |

##### 虚拟机

|  |  |
| --- | --- |
| 1  2 | `ifconfig lo` `127.0``.``0.1` `up`  `ifconfig eth0` `192.168``.``11.2` `up` |

#### 远程连接

##### Telnet

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_AUXEN7WXTC59ZF5.webp)

##### SSH

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_MNPQSUPX2J8SMXR.webp)

本篇的固件文件系统默认启动了telnet以供外部进行访问，若需要从内向外的ssh连接操作则可能遇见以下问题

##### SSH错误

###### exited: No matching algo kex

解决(主机)：

|  |  |
| --- | --- |
| 1 | `echo` `"KexAlgorithms diffie-hellman-group1-sha1"` `>>` `/``etc``/``ssh``/``sshd_config` |

重启ssh服务

###### exited: No matching algo hostkey

解决(主机)：

|  |  |
| --- | --- |
| 1 | `echo` `"HostKeyAlgorithms +ssh-rsa"` `>>` `/``etc``/``ssh``/``sshd_config` |

重启ssh服务

## 1.3 启动固件服务

#### 1.3.1 init.d/boot

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_JBJTCEMAYHVVKZM.webp)

#### 1.3.2 generate_default_cert

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_GBUGDHZRE2ZJZHX.webp)

#### 1.3.3 init.d/confd

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_SM7S5G4J9MXYWVG.webp)

注意看到最后一行`PnP Agent is starting!`，它启动了另一个服务，前提是您的操作系统支持`IPv6`，否则最后会启动失败，这就是为什么在内核编译的时候我强调了要开启`IPv6`的配置选项

#### 1.3.4 init.d/nginx

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_NU8E9UPB34MAVBX.webp)

nginx最后会根据几个配置文件来启动`uwsgi`来负责监听几个端口以处理数据

#### 1.3.5 访问路由页面

![图片描述](https://bbs.kanxue.com/upload/attach/202404/952954_C37YA8UQJJGBP3G.webp)

访问成功

# 2. 漏洞分析

已知是未授权的文件上传漏洞

## 2.1 nginx配置文件

### init.d/nginx

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `NGINX_BIN``=``/``usr``/``sbin``/``nginx`  `UPLOAD``=``/``var``/``upload`  `NOTIFYD``=``/``usr``/``bin``/``notifyd`  `UWSGI``=``/``usr``/``bin``/``uwsgi``-``launcher`  `#...`  `start(){`  `#...`  `$NGINX_BIN`  `$NOTIFYD` `-``i` `127.0``.``0.1` `&`    `$UWSGI start`  `}` |

这里最后启动了`uwsgi-launcher`

|  |  |
| --- | --- |
