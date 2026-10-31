
# [原创开源]Windows下PATH环境变量优化工具

随着开发环境越装越多，`PATH`也越来越长。

于是在想有没有办法优化`PATH`系统环境变量，上网查了些资料，写出了下面这个玩意。

简单介绍下：

* 会自动备份当前的`PATH`到程序工作目录下的`backup_path.reg`文件。重复启动程序会导致文件覆盖！
* 会优化不能被展开的路径名为`DOS8.3`短路径名。不能展开是指不包含环境变量
* 会删除不存在的路径。
* 增加`/l`参数，支持还原回长路径名
* 增加`/n`参数，不对路径优化，仅移除无效路径

重新写入注册表时会询问是否保存优化后的`PATH`。

上图:

![](https://bbs.kanxue.com/upload/attach/201408/423318_6f2uj1qw19875bx.png)

![](https://bbs.kanxue.com/upload/attach/201408/423318_zb1s8nj62lghbym.png)

src And bin:

[百度云盘下载](https://bbs.kanxue.com/elink@8f4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0p5$3k6q4m8#2b7b7`.`.)

另外本人英语不好，看不懂的话，莫见怪![](https://bbs.kanxue.com/view/img/face/1.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [P0.png](javascript:void(0);)
  （14.37kb，5次下载）
- [P1.png](javascript:void(0);)
  （18.67kb，5次下载）
