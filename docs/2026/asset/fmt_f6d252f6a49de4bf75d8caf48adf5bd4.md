
| 偏移范围 | 长度 | 说明 |
| --- | --- | --- |
| 02h..03h | 2 bytes | 标志位 `flag`：0 - DMA boundary errors handled transparently<br>1 - C/H/S Information is valid.<br>2 - Removable Drive.<br>3 - Write with verify supported.<br>4 - Drive has change-line support (Required if Drive is Removable!)<br>5 - Drive can be locked (Required if Drive is Removable!)<br>6 - CHS Information set to maximum supported values; not current media.<br>15-7: Reserved (0). |
| 04h..07h | 4 bytes | 柱面数 |
| 08h..0Bh | 4 bytes | 磁头数 |
| 0Ch..0Fh | 4 bytes | 每个磁道的扇区数 |
| 10h..17h | 8 bytes | 绝对扇区数， |
| 18h..19h | 2 bytes | 每个扇区所占字节，通常是 `0x200` |

下面就要循环拷贝扇区到内存。

![](https://bbs.kanxue.com/upload/attach/202003/845934_NAM6XNS6B2YVD49.jpg)

函数中的代码（`byte_E` 表引导标志）

`int 0x13` 的 `0x42` 号功能从磁盘读 `n` 个 sectors 到 buffer 中。

![](https://bbs.kanxue.com/upload/attach/202003/845934_FKM6FKCDUQW8Q6K.jpg)

把 IPL 的数据全部都从硬盘读到内存后，调用 `Int 1a` 中断，`ah` 为 `0xbb`，检测是否支持 TCG。

![](https://bbs.kanxue.com/upload/attach/202003/845934_WUFMR8EY5MP6XJY.jpg)

跳转到 IPL 执行。

![](https://bbs.kanxue.com/upload/attach/202003/845934_2SEAW326V8HTAMB.jpg)

IPL 的扇区复制完毕后，接着便是解析 NTFS 文件系统，找到 `bootmgr` 文件。

![](https://bbs.kanxue.com/upload/attach/202003/845934_W7T36922BM2T3JP.jpg)

再往下接比较复杂了，暂时分析到这里就已经能够非常快速的分析 MBR 病毒查询逻辑。这个是去年分析的，现在已经有点记不清所有的点了，所以拿出来记一下，方便回阅。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-3-28 15:51

被 Cc28256 编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
