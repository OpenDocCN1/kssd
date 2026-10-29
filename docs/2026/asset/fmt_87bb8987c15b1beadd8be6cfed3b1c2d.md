
补丁版本 `iOS 12.4.9 (16H5)` for `iPhone 6`

这两个固件都是 `ZIP` 压缩文件

|  |  |

| --- | --- |

| 1  2  3 | `➜ ~` `file` `*.ipsw` `iPhone_4.7_12.4.8_16G201_Restore.ipsw:` `Zip` `archive data, at least v2.0` `to extract` `iPhone_4.7_12.4.9_16H5_Restore.ipsw:` `Zip` `archive data, at least v2.0` `to extract` |

解压缩出来，`kernelcache.release.iphone7` 就是压缩后的内核二进制文件

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `➜ iPhone_4.7_12.4.8_16G201_Restore` `ls -al` `total 6012560` `drwxr-xr-x@ 9 wnagzihxa1n staff 288 Jan 2 19:41 .` `drwxr-xr-x 7 wnagzihxa1n staff 224 Jan 2 19:42 ..` `-rw-r--r--@ 1 wnagzihxa1n staff 2874835794 Jan 9 2007 038-60223-004.dmg` `-rw-r--r--@ 1 wnagzihxa1n staff 93846555 Jan 9 2007 038-60285-004.dmg` `-rw-r--r--@ 1 wnagzihxa1n staff 91602971 Jan 9 2007 038-60305-004.dmg` `-rw-r--r--@ 1 wnagzihxa1n staff 128367 Jan 9 2007 BuildManifest.plist` `drwxr-xr-x@ 10 wnagzihxa1n staff 320 Jan 9 2007 Firmware` `-rw-r--r--@ 1 wnagzihxa1n staff 985 Jan 9 2007 Restore.plist` `-rw-r--r--@ 1 wnagzihxa1n staff 14061377 Jan 9 2007 kernelcache.release.iphone7` |

`iPhone 6` 使用的是 `LZSS` 压缩算法

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `➜ iPhone_4.7_12.4.8_16G201_Restore` `xxd -a kernelcache.release.iphone7 | head -n 10` `00000000: 3083 d68f 3c16 0449 4d34 5016 046b 726e  0..<..IM4P..krn` `00000010: 6c16 1e4b 6572 6e65 6c43 6163 6865 4275  l..KernelCacheBu` `00000020: 696c 6465 722d 3134 3639 2e32 3630 2e31  ilder-1469.260.1` `00000030: 3504 83d6 8f0b 636f 6d70 6c7a 7373 025a  5....complzss.Z` `00000040: b99c 01ae f208 00d5 cd8b 0000 0001 0000  ................` `00000050: 0000 0000 0000 0000 0000 0000 0000 0000  ................` `*` `000001b0: 0000 0000 0000 ffcf faed fe0c 0000 01d5  ................` `000001c0: 00f6 f002 f6f0 16f6 f058 115a f3f1 20f6  .........X.Z.. .` `000001d0: f100 19f6 f028 faf0 3f5f 5f54 4558 5409  .....(..?__TEXT.` |

我们对其进行解压缩，使用的工具是 `lzssdec`

下载编译

|  |  |

| --- | --- |

| 1  2 | `➜ lzssdec wget http://nah6.com/~itsme/cvs-xdadevtools/iphone/tools/lzssdec.cpp` `➜ lzssdec g++ lzssdec.cpp -o lzssdec` |

解压缩 `kernelcache` 文件，现在我们获得了一个存在漏洞的固件版本，同理获取打补丁后的固件版本

|  |  |

| --- | --- |

| 1 | `➜ iPhone_4.7_12.4.8_16G201_Restore ./lzssdec -o 0x1b6 < kernelcache.release.iphone7 > kernelcache.release.iphone7.bin` |

现在漏洞版本和补丁版本的 `kernelcache` 文件都准备好了

|  |  |

| --- | --- |

| 1  2  3  4 | `➜ iPhone_4.7_12.4.8_16G201_Restore` `file kernelcache.release.iphone7.bin` `kernelcache.release.iphone7.bin: Mach-O 64-bit executable arm64` `➜ iPhone_4.7_12.4.9_16H5_Restore` `file kernelcache.release.iphone7.bin` `kernelcache.release.iphone7.bin: Mach-O 64-bit executable arm64` |

通过 `bindiff` 进行补丁对比，`bindiff` 现在已经更新到了 `6`

因为一些大家都懂得的原因，Windows 的 `IDA` 目前有最新的 `7.5`，而 `macOS` 只有 `7.0`，如果是 `IDA 7.0`，目前只能使用 `bindiff 5`，如果是 `7.5`，开心的使用 `bindiff 6` 吧

`macOS` 版本有一个错误需要提前解决掉

|  |  |

| --- | --- |

| 1 | `➜ ~ sudo ln -s /Applications/BinDiff/BinDiff.app/Contents/MacOS/bin/bindiff /Applications/BinDiff/BinDiff.app/Contents/app/bindiff` |

为了使用更多的特性以及更准确的分析结果，我决定使用 `IDA 7.5`

将两个文件载入 `IDA 7.5` 进行分析，生成 `idb` 文件，再通过 `bindiff` 分析这两个 `idb` 文件

![图片描述](https://bbs.kanxue.com/upload/attach/202102/915237_DZQUZ885JYM2SM9.png)

关于符号恢复这部分的一波三折大家可以看这篇文章《[关于恢复kernelcache符号的问题](https://bbs.pediy.com/thread-265831.htm)》，记录了我这几天是如何踩坑的

首先比对 `kc_12.4.8` 和 `kc_12.4.9`，得到八个差异函数，再逐个反编译查看代码，发现有五个函数是添加了同一段代码

|  |  |

| --- | --- |

| 1  2  3  4 | `__TEXT_EXEC:__text:FFFFFFF00768E4C4 MOV W1, #0x44 ; 'D'` `__TEXT_EXEC:__text:FFFFFFF00768E4C8 MOV X0, X20` `__TEXT_EXEC:__text:FFFFFFF00768E4CC BL sub_FFFFFFF00766D6C0` `__TEXT_EXEC:__text:FFFFFFF00768E4D0 MOV W23, #0` |

现在记录者五个跟漏洞有关的函数，再用 `kc_12.4.8` 和一个泄露符号的 `kc_symbols`，分别搜索前面记录的五个函数，通过 `bindiff` 的方式恢复出了两个正确的符号

剩下三个函数其中一个具有字符串，搜索源码发现是 `ipc_kobject_server()`，此时剩下两个函数找不到符号

| kc_12.4.8_func_name | kc_12.4.9_func_name | Similarity | bindiff_symbol | true_symbol |
| --- | --- | --- | --- | --- |
| sub_FFFFFFF00768E3AC | sub_FFFFFFF00768E3BC | 0.58 | _ipc_kmsg_get_from_kernel | `ipc_kmsg_get_from_kernel` |
| sub_FFFFFFF00768E164 | sub_FFFFFFF00768E164 | 0.96 | _ipc_kmsg_get | `ipc_kmsg_get` |
| sub_FFFFFFF0076A7824 | sub_FFFFFFF0076A7840 | 0.13 | _mach_gss_accept_sec_context_v2 |  |
| sub_FFFFFFF0076BE438 | sub_FFFFFFF0076BE470 | 0.11 | _ipc_port_send_turnstile_prepare | `ipc_kobject_server` |
| sub_FFFFFFF0076BF8C8 | sub_FFFFFFF0076BF90C | 0.28 | _ptmx_get_ioctl |  |

同时搜索补丁代码中的 `sub_FFFFFFF00766D6C0`，确定是函数 `bzero()`

![图片描述](https://bbs.kanxue.com/upload/attach/202102/915237_AXZHMVRUXJNUPDF.png)

到这一步为止，我们勉强和作者拥有了同样的漏洞分析起点

我们开始分析 `XNU` 源码，先从三个有符号的函数任选一个进行分析，我选择了函数 `ipc_kmsg_get()`

前两天刚好开源了最新的 `xnu-7195.50.7.100.1`

再来一个早一点的版本 `xnu-6153.141.1`

源码一对比，果然多了函数 `bzero()` 的调用

|  |  |

| --- | --- |

| 1 | `bzero(trailer, sizeof(*trailer));` |

左边是漏洞版本，右边是补丁版本

![图片描述](https://bbs.kanxue.com/upload/attach/202102/915237_BRR8NHZAN7XGRUS.png)

补丁操作的变量 `trailer` 类型是 `mach_msg_max_trailer_t`，而 `mach_msg_max_trailer_t` 是由 `mach_msg_mac_trailer_t` 定义而来

|  |  |

| --- | --- |

| 1  2 | `typedef mach_msg_mac_trailer_t mach_msg_max_trailer_t;` `mach_msg_max_trailer_t *trailer;` |

`mach_msg_mac_trailer_t` 是一个结构体，在这个结构体定义附近发现了两个宏：`MACH_MSG_TRAILER_MINIMUM_SIZE`，`MAX_TRAILER_SIZE`，分别代表最大的 `trailer` 和最小的 `trailer` 长度，由此我们可以找到结构体 `mach_msg_trailer_t` 的定义

|  |  |

| --- | --- |
