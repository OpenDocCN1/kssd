
### squashfs-tools

`squashfs-tools`的源码路径是`./build_dir/host/squashfskit-v4.14/squashfs-tools`

修改`squashfs-tools`的`Makefile`，将`XZ_SUPPORT = 1`，引入`liblzma.a`的静态库路径。

```bash
XZ_SUPPORT = 1
LZMA_LIB := ../../xz-5.2.5/src/liblzma/.libs/liblzma.a
```

修改`squashfs_fs.h`头文件的`SQUASHFS_MAGIC`为`0x6563696E`。

```c++
#define SQUASHFS_CACHED_FRAGMENTS CONFIG_SQUASHFS_FRAGMENT_CACHE_SIZE
#define SQUASHFS_MAJOR 4
#define SQUASHFS_MINOR 0
//#define SQUASHFS_MAGIC 0x73717368
#define SQUASHFS_MAGIC 0x6563696E
//#define SQUASHFS_MAGIC_SWAP 0x68737173
#define SQUASHFS_MAGIC_SWAP 0x6E696365
#define SQUASHFS_START 0
```

## 固件解压

首先编译`xz-5.2.5`的静态库文件。

```bash
cd ./build_dir/host/xz-5.2.5
./configure
make
```

然后编译生成`unsquashfs`程序。

```bash
cd ./build_dir/host/squashfskit-v4.14/squashfs-tools
make
```

使用`dd`命令从`0x380040`开始导出固件文件`US_AX3000.squashfs`，然后就可以使用编译好的`unsquashfs`程序进行解压了。

```bash
./unsquashfs US_AX3000.squashfs
Parallel unsquashfs: Using 10 processors
879 inodes (998 blocks) to write
create_inode: could not create character device squashfs-root/dev/console, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyAMA0, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyS0, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyS1, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyS2, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyS3, because you're not superuser!
create_inode: could not create character device squashfs-root/dev/ttyS4, because you're not superuser!
[==========================================================================================================================================| ] 991/998 99%
created 697 files
created 97 directories
created 175 symlinks
created 0 devices
created 0 fifos
```

版本 : [AX3000_Pro_V16.03.49.26](https://bbs.kanxue.com/elink@ee4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1k6h3&6V1j5g2)9J5k6h3y4G2L8g2)9J5k6h3y4F1i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4Q4x3V1k6K6K9r3!0%4i4K6u0r3y4U0x3$3y4e0p5H3y4U0t1@1x3K6M7@1z5o6f1K6)

源码 : [openwrt](https://bbs.kanxue.com/elink@5c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7r3g2F1N6%4u0@1i4K6u0r3L8%4m8W2L8Y4N6J5N6l9`.`.)

首先使用`binwalk -E`分析固件的熵，虽然已经解密了，但是固件的格式`binwalk`无法有效的识别。

![AX3000固件的熵](https://bbs.kanxue.com/upload/attach/202511/920134_G5DFM9G23KGCRBK.webp)

但是使用`file`命令，可以识别出`u-boot`和`Linux`的内核信息。

使用`binwalk -Me US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin`解压固件后，得到`Linux`的内核文件。

`IDA` `ARM64`位的方式打开`128`固件，虽然可以搜索到大量的`squashf`函数字符串，但是都无法查看调用地址和引用地址。

![AX3000函数分析](https://bbs.kanxue.com/upload/attach/202511/920134_TWB4YHGSU8QVC4P.webp)

这时就可以使用`openwrt`的源码来编译生成带符号链接的内核文件，然后生成签名文件，导入到`IDA`进行函数重命名。

首先收集内核配置信息，发现固件是基于`f86f8e568461681fc85caf3c972489045367030e`版本生成的。

下载`openwrt`源码

安装编译环境

下载安装软件包

生成`.config`配置文件，然后进入配置选项。

![openwrt menuconfig](https://bbs.kanxue.com/upload/attach/202511/920134_4NFSE9FG92MD8WK.webp)

在`Global build settings`中，选择`Binary stripping method`为`none`来保留符号编译。

![openwrt strip](https://bbs.kanxue.com/upload/attach/202511/920134_9D2V5RR3RK5KYMY.webp)

保存退出后运行`make`来编译固件。

将`vmlinux.debug`导出到`Windows`系统，使用`IDA`打开生成`vmlinux.sig`签名文件,即可还原函数名称。

![openwrt sig](https://bbs.kanxue.com/upload/attach/202511/920134_DUYH8324RST3FS5.webp)

跳转到固件地址`0x380040`,原来的`squashfs`的`SQUASHFS_MAGIC`由`hsqs`更变为了`nice`，`xz`的`lzma_header_magic`由`ý7zXZ`更变为了`Tenda`。

![squashfs](https://bbs.kanxue.com/upload/attach/202511/920134_CTRMXQUGG8SPK5C.webp)

`squashfs`的`SQUASHFS_MAGIC`判断是在`mtd_check_rootfs_magic`函数中，但是因为源码经过了修改，找不到`mtd_check_rootfs_magic`函数，可以通过`mtd_check_oob_ops`的引用找到`sub_3AC088`。

![nice flag](https://bbs.kanxue.com/upload/attach/202511/920134_BHCJ2N7FD2UBE8R.webp)

`xz`的`lzma_header_magic`是在`xz_dec_run`中判断的，源码经过了修改，可以通过`xz_dec_reset`的引用找到，是`sub_2B0DC0`函数。

![xz flag](https://bbs.kanxue.com/upload/attach/202511/920134_3SDN8DBEU9EM75K.webp)

进入`crc32_le_0_0`,发现`crc32_table`的值发生了改变。

![crc32 flag](https://bbs.kanxue.com/upload/attach/202511/920134_YFDCYBUNPUDN8SC.webp)

![crc32 flag](https://bbs.kanxue.com/upload/attach/202511/920134_W7S69XSPE28MMTJ.webp)

`xz-5.2.5`的源码目录是`./build_dir/host/xz-5.2.5`

`coder.c`

`stream_flags_common.c`

`crc32_table_le.h`

`squashfs-tools`的源码路径是`./build_dir/host/squashfskit-v4.14/squashfs-tools`

修改`squashfs-tools`的`Makefile`，将`XZ_SUPPORT = 1`，引入`liblzma.a`的静态库路径。

修改`squashfs_fs.h`头文件的`SQUASHFS_MAGIC`为`0x6563696E`。

首先编译`xz-5.2.5`的静态库文件。

然后编译生成`unsquashfs`程序。

使用`dd`命令从`0x380040`开始导出固件文件`US_AX3000.squashfs`，然后就可以使用编译好的`unsquashfs`程序进行解压了。

```bash
binwalk US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
64            0x40            Flattened device tree, size: 3625955 bytes, version: 17
296           0x128           LZMA compressed data, properties: 0x6D, dictionary size: 8388608 bytes, uncompressed size: 10719240 bytes
3607560       0x370C08        Flattened device tree, size: 16962 bytes, version: 17
5273660       0x50783C        PGP RSA encrypted session key - keyid: 3B592D4E D1353F41 RSA (Encrypt or Sign) 1024b
```

```bash
binwalk US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
64            0x40            Flattened device tree, size: 3625955 bytes, version: 17
296           0x128           LZMA compressed data, properties: 0x6D, dictionary size: 8388608 bytes, uncompressed size: 10719240 bytes
3607560       0x370C08        Flattened device tree, size: 16962 bytes, version: 17
5273660       0x50783C        PGP RSA encrypted session key - keyid: 3B592D4E D1353F41 RSA (Encrypt or Sign) 1024b
```

```bash
file US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin
US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin: u-boot legacy uImage, \002, Linux/ARM, OS Kernel Image (lzma), 14553088 bytes, Fri Jan 10 06:24:02 2025, Load Address: 0X46000000, Entry Point: 0XFFFFFFFF, Header CRC: 0XB19671AF, Data CRC: 0XC47DB558
```

```bash
file US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin
US_AX3000.Pro_V16.03.49.26_cn_JAX01.bin: u-boot legacy uImage, \002, Linux/ARM, OS Kernel Image (lzma), 14553088 bytes, Fri Jan 10 06:24:02 2025, Load Address: 0X46000000, Entry Point: 0XFFFFFFFF, Header CRC: 0XB19671AF, Data CRC: 0XC47DB558
```

```bash
binwalk 128
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Linux kernel ARM64 image, load offset: 0x80000, image size: 11046912 bytes, little endian, 4k page size,
5498800       0x53E7B0        MPEG transport stream data
7561216       0x736000        ELF, 64-bit LSB shared object, version 1 (SYSV)
7580200       0x73AA28        gzip compressed data, maximum compression, from Unix, last modified: 1970-01-01 00:00:00 (null date)
7719424       0x75CA00        CRC32 polynomial table, little endian
7988227       0x79E403        Neighborly text, "neighbor get requestrequest"
7988277       0x79E435        Neighborly text, "neighbor get request get request"
7988332       0x79E46C        Neighborly text, "neighbor get request"
7988601       0x79E579        Neighborly text, "neighbor get requestrequest"
7988651       0x79E5AB        Neighborly text, "neighbor get requestest"
7988697       0x79E5D9        Neighborly text, "neighbor get request"
7988739       0x79E603        Neighborly text, "neighbor dump requestbor dump request"
7988797       0x79E63D        Neighborly text, "neighbor dump request dump request"
7988852       0x79E674        Neighborly text, "neighbor dump request attribute in neighbor dump request"
7988929       0x79E6C1        Neighborly text, "neighbor dump requeste dump request"
```
