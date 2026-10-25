
# 六一献小礼：完整可编译 NT4's NTFS 源码（可稳定替换 xp 原版 ntfs.sys）

> 原文：[`www.pediy.com/kssd/pediy10/90393.html`](https://www.pediy.com/kssd/pediy10/90393.html)

（首先感谢珠海的同事，给了我很多学习的好机会～）  

write by [`hi.baidu.com/weolar/blog`](http://hi.baidu.com/weolar/blog)  

大家知道，文件系统在操作系统中应该属于比较独立的一块，只需要提供相应接口给上层使用。  

Windows 的 NTFS 文件系统也是一样，  

在实际编程中，Windows 以 dispatch routing 的形式为上层的 io 管理器、缓存管理器等提供读写的接口，  

甚至文件系统本身的缺页中断也是通过其自身的读写例程来实现换页操作。  

所以在设计上，其相对的独立性为我们“山寨”其驱动提供了不少方便。  

相信大家也都看过 NT4 的 NTFS 源码。  

可那份 源码与现在 WIN 5.1 下的相差甚远（NT4 应该写于 91 年）。  

但由于文件系统的独立性，使我想到也许 NTFS 自身结构的变化也许并不妨碍它的移植。  

事实证明也是如此，我将 NT4 的代码扣出后，只经过少许修改，便能成功运行在 XP 下，这也许对广大操作系统爱好者能提供不小的便捷吧～  

本来以为想移植到 xp 中会有不少困难，  

但实际过程中，我只是修改了很小一部分。  

其中包括 read 操作的一个死锁 bug。这个 bug 的起因是：  

NtfsCommonRead 中如果是 pageio 的时候，  

且是异步 read，MS 忘记把锁放进完成函数的 context 里了。即无处释放这个锁。  

另外 NT4 的代码少了 NtfsFsdPnp 例程。  

这个例程在 NtCreatePagingFile-》PpPagePathAssign 中会调用。  

本来我想参照 xp 的，不过似乎没成功，所以干脆直接返回 STATUS\_SUCCESS，  

貌似也没啥大问题，也能成功换页～![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  

其他方面，NTFS 的文件结构啥的，  

基本无需修改。所以各位可以尽管鄙视我，毕竟我没做啥大改动，  

就拿这么个题目来哗众取宠。不过我觉得比那个啥淫 cracker（yingcracker）好点吧，  

这娃直接把我的 ksbinsword 改个标题就说是他的了![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")  

至于这个可源码调试的 NTFS 有啥用，我想不必多说了吧，  

比如你想研究怎么强行删除文件，调试一下 NtfsDeleteFile 就知道了。  

有了代码方便很多！想看哪个结构就看哪个结构。或者你还可以改动一下，想隐藏哪个文件就隐藏哪个文件。  

附件中的 sys 不能直接运行，因为入口函数有 int 3 断点  

下面是截图：  

可以看出硬链接等 NTFS 特有的功能也支持的好好的  

![](img/33b9ab3dbae01195886f8731bebf248c.jpg)  

uneshell.org 帐号：u10365n

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part05.rar`\[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=90393`](http://bbs.pediy.com/showthread.php?t=90393) \]  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part06.rar`  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part01.rar`  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part02.rar`  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part03.rar`  

 |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `MyNtfs.part04.rar`  

 |
