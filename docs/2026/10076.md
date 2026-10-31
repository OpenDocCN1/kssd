
# [旧帖][原创]读写mbr汇编源码(申请邀请码)0.00雪花

最近开始学汇编，就写了一个简单的程序。通过打开`\\.\physicaldrive0`，即第一块硬盘，然后写入自己预先写好的mbr引导程序。成功写入后，下次开机会显示“Chinese hackers have already invaded your computer”。

用`Ghost`还原系统的话是没有效果的！还原后还是有的。如果用`od`加载进这个程序的话，还是会成功写入。

解决方法：重建mbr分区表。

以下是我写的简单的汇编程序，用`Fasm`编译，有注解。`BOOT.ASM`是mbr引导程序，目的是显示字符串；`PE.ASM`是Windows下写入mbr的程序。程序很简单，高手就路过了。

建议不要用这个方法干坏事。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`MBR.rar`](javascript:void(0);) （1.86kb，125次下载）
