
已经设置完成的具体注册表键值，如下，显示的名称会让人以为是一个微软的合法服务。

![](https://bbs.kanxue.com/upload/attach/202002/678001_HE5QYY4ZSRQ68PP.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_EFQ4ARTNU7UMHV8.jpg)

整体一个攻击与传播感染流程与之前安天分析的差不多，只有个别存在不同，图来源于安天的分析报告，此处为引用。

![](https://bbs.kanxue.com/upload/attach/202002/678001_A48MB46SYZD6HQQ.jpg)

# 5 样本分析

## 5.1 `spoolsv.exe`（大）分析

对`spoolsv.exe`（大）`bf98a0555212d2adbb7c020cb39084b6`进行下分析，无壳，64 位 C++程序，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_YC5E346SWMCNND9.jpg)

查找时间戳为 2018 年 6 月 26 日，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_FM2E27SQNXG32PF.jpg)

调试时可去掉 ASLR，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_PRDZNJYKP32YDTG.jpg)

IDA 对其分析后，发现主体功能如下，注释为已经分析过的结论，是一个多线程的 C++编写的程序，无 GUI，与 32 位汇编不同的是，在 x64 下，是寄存器传参。前 4 个参数分别是`rcx`、`rdx`、`r8`、`r9`进行传参，多余的通过栈传参从右向左入栈，如下是整个主程序的大致流程。

![](https://bbs.kanxue.com/upload/attach/202002/678001_CMAQZ3XEYXUTZ6H.jpg)

### 5.1.1 x64dbg 调试多线程

针对多线程的调试，之前是使用的是`ollydbg`，这次使用`x64dbg`对其调试。那么在`x64dbg`中如何调试多线程？这里就举当前分析的挖矿蠕虫恶意样本（`bf98a0555212d2adbb7c020cb39084b6`）。首先进入`main`函数位置，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_53GDAJAYW89QDGQ.jpg)

接下来会看到存在连续几个线程的创建，就以调试`thread1`（已经在上述的 IDA 中对这些线程进行了重命名）为例。

![](https://bbs.kanxue.com/upload/attach/202002/678001_WHX682B69U3PMQ8.jpg)

先在`thread1`线程入口设置断点，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_VX87734M427ZU9G.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_K9Z6BFVKRQV23NJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_HY92W5HUPQRH6MU.jpg)

在执行创建`thread2`线程时会进行跳转，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_EGZ47UX68X2EXYA.jpg)

此时来到线程列表窗口，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_4YNGEJKVMCEKJZJ.jpg)

将主线程挂起，避免影响`thread1`线程调试。

![](https://bbs.kanxue.com/upload/attach/202002/678001_Z9S5U8BDJYBW99K.jpg)

此时回到之前的汇编窗口，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_4W9NPQUZAYHTFQU.jpg)

一步步调试，就会切换到`thread1`线程入口。

![](https://bbs.kanxue.com/upload/attach/202002/678001_HV6EXY7NFPG6CRC.jpg)

此时就可以对`thread1`线程进行调试了，针对需要调试其中一些线程的时候，其余线程都可以进行挂起，避免对正在调试的线程造成影响。

![](https://bbs.kanxue.com/upload/attach/202002/678001_H4FZHQYYRRNGZF4.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_7ADTQKX5NM6627W.jpg)

现在演示下调试第二个`thread2`线程，如下，先恢复主线程。

![](https://bbs.kanxue.com/upload/attach/202002/678001_HP2YEYRYFA6P57U.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_FESKXCJB2228SPD.jpg)

在`thread1`线程里进行单步调试，不一会就立马跳转到了主线程区域。

![](https://bbs.kanxue.com/upload/attach/202002/678001_WMRC6PDKCXRC79R.jpg)

此时接着查看线程列表窗口，将之前调试的`thread1`线程进行挂起，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_JYFK8S7CP82V8W7.jpg)

在需要调试的`thread2`线程入口下断点，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_KKHXJGGFVYXHDA4.jpg)

在主线程进行单步调试，会跳转到线程处理部分，此时再将主线程进行挂起，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_ET63HCRKCK8BTB3.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_EBUYV92UZBAXSAP.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_R5F87J4M9CH7T4G.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_J5RR8D8GGDBXSWV.jpg)

接着调试`thread3`，此时需要在线程窗口将主线程给恢复，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_9JGSQP4TEX7PEAA.jpg)

此时处于`thread2`区域，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_2RJHC64YCDZB4WA.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_NEPY4GDZC93EXUZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_BCS9JBYS99EBAZT.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_737V965E53YHBMM.jpg)

将`thread2`线程给挂起，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_7ZVAN4TYX2QHTEB.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_DYANC8MUPVA5HGM.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_PSD94N8QX37UDU4.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_BX8TBHGEMKXWAUK.jpg)

在主线程区域执行完创建`thread3`线程的函数，如下，此时还无法跳转。

![](https://bbs.kanxue.com/upload/attach/202002/678001_NBRQ6RD3CUZPHW3.jpg)

此时来到线程列表窗口，在之前刚调试的`thread2`右键选择【转到线程】，此时回到反汇编窗口，查看是否已经是`thread2`线程区域，这时再在线程列表窗口里将主线程右键选择【挂起线程】，目前就可以继续进入`thread2`线程区域调试，目的是为了调用`Sleep`函数，达到线程切换。当在`thread2`里调用`Sleep`函数后，此时就切换到`thread3`线程区域，然后将`thread2`再次进行挂起，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_HKSTEG727W3Y362.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_YPEM97ZUBZB87F4.jpg)

运行之后就会切换到`thread3`线程区域，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_XSUFD9PAZ8CV24N.jpg)

成功跳转到`thread3`，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_BJS4FEPEKP7EEDC.jpg)

重新调试程序，一直单步调试，直到进入到`thread3`区域后将其余线程给挂起，当然如果线程之间有相互的数据共享，挂起其余线程后也影响当前调试的线程，所以调试时还是具体进行决定是否全部挂起线程。

![](https://bbs.kanxue.com/upload/attach/202002/678001_R3WZXUD3APE7MNE.jpg)

### 5.1.2 具体动态调试

首先出现的是硬编码的互斥量`{F5395176-40C2-0218-278D6EE}`，已感染的系统最早的恶意文件为`C:\Windows\System32\EnrollCertXaml.dll`与`spoolsv.exe`（498k 大小）。

#### 5.1.2.1 `thread1`分析

拼接所需的路径，创建以下目录：`C:\Windows\SpeechsTracing\`与`C:\Windows\SpeechsTracing\Microsoft\`，之后`spoolsv.exe`（498k 大小）进程会从`EnrollCertXaml.dll`提取并释放出`Crypt`文件，该文件为一个压缩文件。之后对其解压释放到`C:\Windows\SpeechsTracing\Microsoft\`目录下，实际查看后发现为 NSA 漏洞利用工具。再接着从`EnrollCertXaml.dll`分别释放出`x86.dll`与`x64.dll`到`C:\Windows\SpeechsTracing\Microsoft\`目录下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_NQATBW4PMG35HBN.jpg)

会释放出`Crypt`文件，后续会被进行解压到目录下成为 NSA 工具利用包，最后会删除`Crypt`文件。

![](https://bbs.kanxue.com/upload/attach/202002/678001_79GCCRFX9KBGM5E.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_HFCTF6WRZS3KYRQ.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_7BM33ERYJ4ZBWFF.jpg)

会删除`Crypt`文件，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_D8ZAKFG7R7FR6YG.jpg)

释放出 NSA 工具包后，第一步会先执行永恒之蓝漏洞工具并记录攻击日志，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_EA8CYP3J8HZPRJ8.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_6DMETFXZ42YTB26.jpg)

永恒之蓝利用时用到的相关配置文件`svchost.xml`会从程序的资源文件区域`BIN`进行读取创建，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_TEYAB4WBX4SBWJE.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_XYWA9SHHWBNANJP.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_J86NMMAHT7TNY5K.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_5TUZNJ9XS535MHQ.jpg)

第一阶段记录的攻击日志，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_JTXJDNRWBVS66XN.jpg)

永恒之蓝漏洞利用成功后，接着会使用双星漏洞进行利用并进行日志记录，来注入恶意`dll`，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_BQUZPFJNH5VW3HX.jpg)

其中利用的双星漏洞的配置文件，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_RRED77EWHKR6574.jpg)

对应的攻击日志，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_H2DZUCFZA693RF3.jpg)

永恒之蓝的漏洞利用配置文件，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_N4N3PB3NVYK3R4M.jpg)

获取到相应的 IP 地址时就会写入刚刚的配置文件内，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_FTAPCC4W5C8K8DE.jpg)

开始进行漏洞利用，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_B6GUZD2R2Y2NC2M.jpg)

整体的一个利用过程，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_XJB2VCDHR69Z6FB.jpg)

#### 5.1.2.2 `thread2`分析

`thread2`每隔`500ms`最终启动 64 个线程，加快扫描速度。

![](https://bbs.kanxue.com/upload/attach/202002/678001_SV77VR4MZFD9Z6D.jpg)

再次创建线程，进行端口的扫描侦查，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_GVQAEDEDFVPJYU3.jpg)

尝试连接 445 与 63257 端口，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_R77CCUU2QKY5ZV4.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_Y3UNG7F5YH58YVB.jpg)

最终开启的大量线程进行扫描，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_HF6MCKH5HBT498J.jpg)

#### 5.1.2.3 `thread3`分析

`thread3`主要是获取本地 IP 地址，并排除本地环回地址。

![](https://bbs.kanxue.com/upload/attach/202002/678001_UBDEMD8KYSW5VTU.jpg)

之后将获取到的 IP 共享给其余线程使用。

#### 5.1.2.4 `thread4`分析

`thread4`由之前两个线程来获取有效的 IP 地址后并采用`TcpTable`进行管理，如下。

![](https://bbs.kanxue.com/upload/attach/202002/678001_26V3Z4X2RK339B9.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/678001_THVKTE4C64BACY7.jpg)
