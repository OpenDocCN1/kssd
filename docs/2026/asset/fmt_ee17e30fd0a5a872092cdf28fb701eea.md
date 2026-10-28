
# [原创]Petya勒索病毒的一次详细分析之旅（附上样本原文件、`idb`文件、`mbrdump`等）

## 0x0 写在前面

新人一枚，注册看雪好多年了，第一次发分析贴，有很多不足，如果发现错误，请大家多加指点。

Petya勒索病毒在6月底爆发，然后各大安全公司都发出了相关的报告，但是出于对该样本的兴趣，以及想了解更多关于该样本的技术细节，就尝试进行了一次详细分析，希望这个报告可以帮助到一些想自己动手分析该样本的同学们。

下面就是详细的分析流程：

## 0x1 概述

样本名：`PetYa`

MD5：`71b6a493388e7d0b40c83ce903bc6b04`

分析环境及工具：winXp sp3、[IDA](https://bbs.kanxue.com/elink@46cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5M7r3!0B7K9h3g2Q4x3X3g2U0L8W2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5k6o6b7@1x3U0M7H3x3W2)9J5k6o6q4Q4x3X3b7I4i4K6u0W2K9s2c8E0L8l9`.`.)6.8、吾爱OD1.1

## 0x2 相关文件

- `PetYa.dll`：样本主体
- `X.tmp`：样本释放的PE文件，用于提取本机账户密码的工具
- `Dllhost.dat`：样本释放的PE文件，用于远程执行命令

## 0x3 行为预览

1. 进程自身提权、检测杀软
2. 拷贝自身代码到堆内存执行，并卸载自身主模块，躲避内存扫描
3. 加密MBR引导扇区数据，篡改MBR代码
4. 创建重启计划任务
5. 释放密码抓取以及远程命令执行工具
6. 通过多种方式进行传播自身以及远程执行
7. 加密磁盘指定格式的文件数据，只加密固定磁盘，不对可移动存储设备加密，通过生成AES128密钥加密文件数据、然后用样本内置的RSA公钥加密AES密钥
8. 系统重启后加密NTFS文件系统的MFT表

## 0x4 报告正文

### 样本介绍

该样本为一个DLL文件，有一个导出序号为1的函数，该函数为样本整个行为的入口。

### 分析过程

#### 1. 提升进程权限、检查当前运行的进程、打开自身PE，读取自身PE数据以及保存自身文件信息

（1）首先需要写个`dllload`加载该样本并且调用1号导出函数：

![](https://bbs.kanxue.com/upload/attach/201707/595575_4bjtag10fgsxf06.png)

（2）提升进程权限，如果提升成功则保存到一个全局变量作为标记：

![](https://bbs.kanxue.com/upload/attach/201707/595575_j8z4aib0bfitt5s.png)

（3）遍历当期进程列表，判断是否有预期的3个进程在运行，该样本通过自定义的算法将进程字符串计算后得出一个`DWORD`数值，然后与事先计算好的3个进程名对应的`DWORD`数值进行比较，判断是否有这三个进程在运行：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_aoo5ubq93y3b3ph.png)](https://bbs.kanxue.com/upload/attach/201707/595575_aoo5ubq93y3b3ph.png)

经过分析这3个`DWORD`数值分别对应杀软：

`0x2E214B44`：卡巴斯基

`0x6403527E/651B3005`：诺顿

（4）读取自身数据到堆`buffer`，并且保存`buffer`首地址以及文件大小到全局变量：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_wtnc4bu4erh4zn3.png)](https://bbs.kanxue.com/upload/attach/201707/595575_wtnc4bu4erh4zn3.png)

#### 2. 拷贝自身PE数据到堆`buffer`中，并且修复重定位，然后流程转移到堆中执行代码，接着卸载掉样本自身的主模块，删除自身PE文件，然后再次调用导出函数1，这么做是为了躲避杀软的内存扫描

（1）申请堆空间，拷贝自身代码，修复重定位，流程转移：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_p7brtytbsg2u9od.png)](https://bbs.kanxue.com/upload/attach/201707/595575_p7brtytbsg2u9od.png)

（2）卸载自身主模块，删除自身文件然后调用导出函数1：

![](https://bbs.kanxue.com/upload/attach/201707/595575_60ljqzkuvj3c73n.png)

#### 3. 调用`WSAStartUp`初始化，检查`c:\windows`目录是否存在自身样本文件，如果存在则退出进程：

（1）初始化全局socket：

![](https://bbs.kanxue.com/upload/attach/201707/595575_8b9cb94f9czihbo.png)

（2）检查`c:\windows`目录下是否有样本自身的文件，如果存在则结束进程，不存在则创建文件：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_3ztz2pbl4hpaulm.png)](https://bbs.kanxue.com/upload/attach/201707/595575_3ztz2pbl4hpaulm.png)

#### 4. 篡改硬盘MBR代码：

（1）如果debug权限提升成功，则进行篡改MBR：

![](https://bbs.kanxue.com/upload/attach/201707/595575_11x67wugeabxt38.png)

（2）打开c盘驱动设备，获取磁盘信息得到扇区大小，然后根据该大小乘以10申请`buffer`，将`buffer`写入到第二个扇区中：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_8bu5cjpvflhjpcl.png)](https://bbs.kanxue.com/upload/attach/201707/595575_8bu5cjpvflhjpcl.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_z7zab59nw7s8o7z.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_jta52xnvhx7dors.png)

（3）判断卡巴斯基进程是否存在，如果存在则不执行MBR篡改：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_f0q3ble862fkg6b.png)](https://bbs.kanxue.com/upload/attach/201707/595575_f0q3ble862fkg6b.png)

（4）获取系统所在的盘符，然后查询该盘符对应的物理磁盘驱动器编号：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_0azzcg4jhlusuda.png)](https://bbs.kanxue.com/upload/attach/201707/595575_0azzcg4jhlusuda.png)

4.1）获取系统路径：

![](https://bbs.kanxue.com/upload/attach/201707/595575_2gs11uqi6geqsvt.png)

4.2）覆盖后得到系统所在盘符的符号链接：

![](https://bbs.kanxue.com/upload/attach/201707/595575_jpsaa384af2uewo.png)

4.3）打开系统盘符号链接，获取该盘所在的物理磁盘编号，然后将该编号数值转为字符串：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_ehi7cl89c5ercwn.png)](https://bbs.kanxue.com/upload/attach/201707/595575_ehi7cl89c5ercwn.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_mbigqln4cm4fo8f.png)

4.4）拼接出完整的物理磁盘符号链接。拷贝到参数指向的`buffer`内，作为传出数据：

![](https://bbs.kanxue.com/upload/attach/201707/595575_xm9qceclf1mc85t.png)

（5）检查`\\.\PhysicalDrive0`磁盘设备的分区类型是否为MBR，如果不是则不执行篡改代码：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_xex764telqrue5e.png)](https://bbs.kanxue.com/upload/attach/201707/595575_xex764telqrue5e.png)

磁盘分区类型：

![](https://bbs.kanxue.com/upload/attach/201707/595575_zj3q429k8neuss2.png)

[![](https://bbs.kanxue.com/upload/attach/201707/595575_e1ht430plrpkm7t.png)](https://bbs.kanxue.com/upload/attach/201707/595575_e1ht430plrpkm7t.png)

（6）生成60个字节的随机数，然后按字节取余`0x3A`，得出的值作为一个全局数组的下标，用该下标访问，拼接出一个疑似[序列号](https://bbs.kanxue.com/elink@543K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5M7r3!0B7K9h3g2Q4x3X3g2U0L8W2)9J5c8R3`.`.)的字符串：

![](https://bbs.kanxue.com/upload/attach/201707/595575_s0hnapvvwkhy3yr.png)

[![](https://bbs.kanxue.com/upload/attach/201707/595575_38h7y00zth7bad5.png)](https://bbs.kanxue.com/upload/attach/201707/595575_38h7y00zth7bad5.png)

最终生成的序列号：

![](https://bbs.kanxue.com/upload/attach/201707/595575_xpy79icqx2etvd5.png)

（7）读取`PhysicalDrive`起始扇区的MBR引导代码进行异或加密：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_wjuda81f948iwng.png)](https://bbs.kanxue.com/upload/attach/201707/595575_wjuda81f948iwng.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_vwc15kdfk5tg8ht.png)

加密前的起始扇区数据：

![](https://bbs.kanxue.com/upload/attach/201707/595575_7ugx16tqmw0fjdg.png)

异或加密后的数据：

![](https://bbs.kanxue.com/upload/attach/201707/595575_ss7rjsfsz6vqv1y.png)

接着将一个大小为`0x200`的栈`buffer`全部初始化为`0x7`：

![](https://bbs.kanxue.com/upload/attach/201707/595575_ijibsu66xqi8jy1.png)

然后分别生成了两个32个字节和8字节的随机数，这两个随机数分别是`salsa20`算法的`key`以及`iv`，在MBR代码里面会运用到`Key`和`iv`来进行MFT加密：

![](https://bbs.kanxue.com/upload/attach/201707/595575_gagup49yabkgjg1.png)

接着拷贝了一串字符串到栈`buffer`里，这段字符串为比特币钱包地址：

![](https://bbs.kanxue.com/upload/attach/201707/595575_mlfg70nh1vpfu2j.png)

（8）申请两块内存，大小分别是`0x200`和`0x22B1`，然后分别拷贝样本自身的MBR代码到这两个`buffer`内：

![](https://bbs.kanxue.com/upload/attach/201707/595575_4rxstvl2clgx1it.png)

（9）循环写入样本的MBR代码，长度为19个扇区：

![](https://bbs.kanxue.com/upload/attach/201707/595575_s6tdxbt6ev4xziz.png)

[![](https://bbs.kanxue.com/upload/attach/201707/595575_7tagte5xdulsj3n.png)](https://bbs.kanxue.com/upload/attach/201707/595575_7tagte5xdulsj3n.png)

篡改前的代码：

![](https://bbs.kanxue.com/upload/attach/201707/595575_xclxu7f2rex8239.png)

篡改后的代码：

![](https://bbs.kanxue.com/upload/attach/201707/595575_x8fzz3cf4pjrq4w.png)

（10）然后将之前通过60字节随机数当做下标生成的序列号字符串、以及用来加密MFT的32字节的`key`和8字节的`iv`、还有样本内置的字符串数据（经后面的分析得知该串为比特币钱包）写入第32个扇区：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_5dckf76zb9t2uyj.png)](https://bbs.kanxue.com/upload/attach/201707/595575_5dckf76zb9t2uyj.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_ge0j1pcyke66xwi.png)

（11）将之前初始化全是`0x7`的`buffer`写入第33个扇区：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_59aio6z6v9vp0lr.png)](https://bbs.kanxue.com/upload/attach/201707/595575_59aio6z6v9vp0lr.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_2su35991kzq74w7.png)

（12）将加密后的起始扇区MBR引导代码写入第34个扇区：

![](https://bbs.kanxue.com/upload/attach/201707/595575_l5c96v7bic219qm.png)

![](https://bbs.kanxue.com/upload/attach/201707/595575_rs804lzagj5881g.png)

#### 5. 创建重启的任务计划：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_ttp1wgsh1uuyp1l.png)](https://bbs.kanxue.com/upload/attach/201707/595575_ttp1wgsh1uuyp1l.png)

该任务计划是在样本运行之后一个小时进行重启：

![](https://bbs.kanxue.com/upload/attach/201707/595575_n7gymjy50ry31ax.png)

#### 6. 样本自身的传播与远程执行

首先该样本通过三种途径传播自身：

1. 通过局域网勘测，检测一些可以访问的内网ip并且保存起来，然后对这些ip进行传播自身并且远程执行自身
2. 通过windows远程桌面的登录凭据记录，获取这些凭据来进行传播自身以及远程执行
3. 通过（1）步骤获取到的ip地址，利用永恒之蓝漏洞进程传播

远程执行的方式：

样本通过`WNetAddConnection2`函数连接远程主机，然后将自身PE写入到远程主机的`c:\windows`目录接下来通过两种方式执行：

通过资源中释放出来的`psexec`的远程命令执行工具去运行`rundll32.exe`，通过该系统exe加载自身DLL并且执行导出函数1

通过`wimc`运行`rundll32.exe`，通过该系统exe加载自身DLL并且执行导出函数1

（1）创建线程，进行可攻击ip的勘测，首先，样本会调用同一个函数初始化两个`0x34`大小的结构，接下来会对该结构进行分析说明：

![](https://bbs.kanxue.com/upload/attach/201707/595575_kahnydjxur47my5.png)

结构初始化函数：

[![](https://bbs.kanxue.com/upload/attach/201707/595575_7854fyyymif8lt8.png)](https://bbs.kanxue.com/upload/attach/201707/595575_7854fyyymif8lt8.png)

经过分析还原出如下结构体：
