
# [原创]关于绕过NGS(nexon game security)

NGS 采用双进程检测：

- 检测游戏代码段是否被 hook
- 代码段内存属性是否被修改
- 游戏 DR 寄存器是否被修改（存疑）
- 枚举遍历所有进程，查看是否有可疑进程（CE、OD）
- 检查可疑文件，有上传行为

## 关于调试

我的方法是写个驱动不给 NGS 打开游戏句柄权限。

五天前还能调试，今天不行了，游戏没更新，检测更新了，卧槽...

又测试了一阵，发现可以先给打开游戏进程句柄权限，等调试时再关闭 NGS 权限，可行。

（估计过几天这方法又得和谐掉，方法多的很，慢慢绕就是了。）

附件是外网绕过 NGS 方法。hook NGS `openProcess` 跨进程读写内存函数（我觉得不太保险，容易被检测）。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [NexonGameSecurity-bypass-alternative-master.zip](javascript:void(0);)（13.65kb，473次下载）
