
# [原创]360第5题 -- 简单方法暴删文件

第5题 -- 简单方法暴删文件

write by [327K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6%4k6h3!0D9j5i4u0Q4x3V1j5](<https://bbs.kanxue.com/elink@79cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6%4k6h3!0D9j5i4u0Q4x3V1j5>).

（解答的很仓促，有错在所难免，欢迎大家拍砖）

## 第五题解答思路

本题有多种解法，由于时间原因我只实现了个比较不好的方法。

不过本方法实现简单，完全不依赖系统ntfs，基本磁盘驱动以上的hook都能无视并暴力删除。

用`XueTr`扫一下可以发现驱动加载后hook了`NtfsCheckValidAttributeAccess`和`NtfsOpenAttributeInExistingFile`这两个ntfs内部的函数。这两个hook点选的还是比较好的，一是够深入够隐蔽，二是到文件系统这层基本就能杜绝那些文件名绕过等技巧了。

能首先想到的是反hook。虽然题目要求是不能干扰钩子，但可以钻题目的空子：`NtfsCheckValidAttributeAccess`是函数头`jmp`的hook，在NTFS里查找引用这个函数的地方，只会在`NtfsOpenFcbById`、`NtfsOpenExistingPrefixFcb`、`NtfsOpenFile`、`NtfsCreateNewFile`这几个地方用到，所以我们可以在这几个函数用到的地方再hook一下，直接跳转到`NtfsOpenAttributeInExistingFile + 5`地址。

而绕过`NtfsCheckValidAttributeAccess`就更好办了，这里的hook只是把`call`的地址改了。我们可以在`NtfsCheckValidAttributeAccess`的函数头hook一下。这样就能完美绕过`hijackFile`的钩子了。不够这种方法太取巧……我放弃了。

（事实证明放弃是对的，因为这招不给分，呵呵……）

第二种方式是直接操作ntfs的文件系统，自己写扇区实现。我把`ntfs-3g`的代码移植到vc上，不过倒楣的是移植的版本有点老，使用起来貌似有bug，要改还是有点麻烦……我又放弃了。

第三种方式是黑盒分析后得出的。首先可以确定`hijackFile`的钩子不会对D盘的相同路径的`360safe`过滤，也就是说它的钩子有检测关于卷的信息。我把卷设备对象和相关的`VPB`、ntfs的设备对象copy了一份分别测试，发现只有当ntfs的设备对象不同的时候才能删除文件。所以可以得出结论，`hijackFile`的钩子有验证ntfs的设备对象（暂时称`pNtfsDev`）的某个域。但其实`pNtfsDev`是个很大的结构体，结构体的开始部分才是`DEVICE_OBJECT`类型。其余的是文件系统自带的结构（`VCB`等）。

如果能找出钩子验证的是哪个域就能伪造一份。由于驱动被加密，逆向很麻烦，可以这样分析：先获取`pNtfsDev`的地址，当运行到钩子的时候填充一部分`pNtfsDev`所指向的内存为0。如果系统崩溃的话说明钩子会访问相关的域。这样几次测试后就能确定是哪个域，然后根据进行伪造。但`VCB`的结构是不公开的，这个就麻烦了，很难确定这个域是干嘛用的。所以我又放弃了。

（不过现在知道了，是MFT号）

最后找到一种比较简单的绕过方式：重加载文件系统方式。可以这样：预先加载我的山寨版ntfs，这个ntfs被我修改过，比如设备名是`MyNtfs`。加载后`myntfs`会通过`IoRegisterFileSystem`注册一个文件系统的设备对象（实际上就是放入一个链表）。这个设备对象是专门用来被系统在`IopMountVolume`的时候在一个链表里检测到然后用来操作挂载的。

然后将卷设备、`VPB`、各copy一份，把`vpb`的`flag`位清空，随后手动挂载`MyNtfs`。这样就能获取能用来操作文件的`pNtfsDev`（实际上也就是`PVOLUME_DEVICE_OBJECT`）。由于是自己的文件系统，所以想删什么文件都没问题了。

（当然，如果是直接替换系统原来的，可以直接右键删除，我的代码删了一句话，实际实现也是这个效果。不过这个效果有点bug，后面会提到。）

这样有个问题，系统原来open过的文件再用`MyNtfs`能进行操作么？事实是可以的，因为open（也就是create）操作后系统保存的是`file_object`，这个是会保存原ntfs的`PVOLUME_DEVICE_OBJECT`。所以以前的open操作后要继续操作，走的是原ntfs路线，新open操作就走的是`MyNtfs`了。

系统重启后可能有`dskchk`。这是由于`MyNtfs`在某个扇区里写了个标志位，本来想去掉，想想不管了……

代码见附件。

注意，运行代码前先用工具加载`MyNtfs.sys`。

`MyNtfs`的代码见本论坛或debugman。附件提供了个`bin`，改动不大。

## 又及

当时匆忙搞完后没仔细跟。现在又看了一遍，发现了个疏忽，实际上代码里面的挂载ntfs的代码没起作用，真正挂载到系统的是系统的`IopMountVolume`。具体原因还不明，貌似是`NtfsReadMftRecord`里读缓存失败，读的是个空的。不过系统的偏偏能成功……暂时交给各位来搞吧。

所以这题我做的很失误，难怪`dskchk`检查ntfs有错误。因为和系统原版NTFS有冲突。不过思路是对的，欢迎大家继续研究。

谢谢观看，各位新年愉快～

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [`第5题.zip`](javascript:void(0);)

  （272.17kb，195次下载）
