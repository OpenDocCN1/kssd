
# \[原创\]分享我的PE工具`PEViewer`\[Delphi含源码\]

自己写的一个PE工具，很不完善，包括基本的查看功能，进程列举，添加区块，Delphi7编写，个人Pascal只是业余水平，未深入学习，部分组件用了`Raize_v4.02_D7`版，可去盒子和园子下载

在看雪潜了5年的水了，很少逛论坛，来也是来这里！总学习前辈同仁的代码！可还从来没分享过什么！将自己写的一个小程序发出来，供和我一般的新手学习，也请各位多指教！

发这个，其实我有个小小的愿望！还请斑竹大人特批！嘻嘻！

虽然自己依然很菜，但是还是会努力的向各位大牛学习！

- 程序名称：`PEViewer`
- 程序作者：shaoyun
- 开发平台：Delphi7、Windows XP SP3 测试通过
- 支持平台：Win9x/WinNT/Win2000/WinXP/WinVista/Win7
- 完成时间：2009年10月27日10:01:46
- Email: shaoyun at yeah.net
- Blog: c56K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5j5h3!0&6N6h3&6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0

## 程序简介

可执行文件格式分析工具，用于了解和学习Windows平台的可执行文件。

结合作者以前写的`PEViewer`、`ProcessViewer`的VC、Delphi两种版本所有功能，重新组织了界面。

可查看进程PE文件的信息。

系统进程枚举提供简单的进程查杀功能，详细的列出了进程的路径、模块、基址、进程内存占用等信息。

右键菜单，可快捷刷新和结束进程，F5为进程刷新快捷键，还包含打开所在目录、复制进程/模块路径功能，可直接打开进程/模块文件所在目录并选中文件

状态栏用于显示系统版本、进程总数、内存占用信息，通过提升进程权限，解决了`csrss.exe`进程无法显示的问题。

## 更新日志

- **2009-11-3**
  - 开发`TSYEdit`组件替代`TRzEdit`组件，增加16进制模式和数字模式，对齐属性的支持
  - 增加文件可读写状态提示
- **2009-11-6**
  - 增加区块添加功能，自动清理绑定数据，自动备份
  - 修正`TSYEdit`组件设计模式赋值BUG
- **2009-11-7**
  - 增加`FLC`(`VA`、`RVA`、`FileOffset`计算)功能
  - 修正区块是否修改的错误判断BUG

附件中包含源码和Edit组件的源码！

[`PEViewer20100106.rar`](https://bbs.kanxue.com/attach-download-37139.htm)

![](https://bbs.kanxue.com/upload/attach/201001/24036_r9ceu1qhtuxzx2s.png)

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`PEViewer20100106.rar`](javascript:void(0);)（351.87kb，285次下载）
- [`PEViewer_Snapshot.png`](javascript:void(0);)（47.11kb，459次下载）
