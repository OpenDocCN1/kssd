

# vfp&exe v1.85

> 原文：[`www.pediy.com/kssd/pediy03/forum1294.htm`](https://www.pediy.com/kssd/pediy03/forum1294.htm)

vfp&exe v1.85 demo 又一简单破解法   
1、运行加密的 VFP 文件，最小化该进程   
2、运行 filemon.exe 找出 VFP 最小化进程文件名（在一些特殊目录中）   
  （或者开始就运行 trw2000 调入加密后的 vfp 文件，BPX 401760 后 F5，   
  中间横线上出现新的进程名后记下该文件名，退出 trw2000 和 vfp 加密文件）   
3、重新运行 vfp 加密文件，最小化后用 win98 下的查找文件找到先记下的那一   
  文件后，按右键将该文件复制出来的文件即是去壳后的文件   
4、改入口点为 401760   
5、删除第二次找到的 FEF2 后的所有字节，存盘后的文件即是可用 unfoxall 反编译的文件了  4dd466d0e1e0cc123afbe504da2462ca