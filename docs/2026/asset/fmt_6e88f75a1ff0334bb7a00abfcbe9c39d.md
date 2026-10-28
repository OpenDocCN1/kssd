
# [原创]给任务管理器添加拷贝命令的功能

- 【文章标题】：【原创】给任务管理器添加拷贝命令的功能
- 【文章作者】：nickwu
- 【作者邮箱】：wuayi@qq.com
- 【软件名称】：`taskmgr.exe`
- 【下载地址】：附件
- 【使用工具】：`eXeScope`、`OD`、`LoadPE`、`ZeroAdd`
- 【操作平台】：Windows XP SP2

要检验自己学到的知识到底有多少，或者自己还有哪些知识没掌握，最好的办法就是多实践，从实践中获得的成就感会让你更拼命的为之奋斗，这也是为什么我喜欢PEDIY的原因。

在这之前先得感谢那些令人敬仰的前辈们，没有他们的文章，我碰到的困难会更多，所以很多时候，当我们欢呼雀跃于最终的胜利时，有必要回过头来想一下，我们之所以成功，是因为站在巨人的肩膀上，而且那巨人还往往不止一个。

## 参考文献

- stalker大侠写的《给任务管理器增加显示程序完整路径功能》
  链接：<http://bbs.pediy.com/showthread.php?t=82582>
- slore大侠写的《第一次PEDIY---让Windows 任务管理器更强,更人性!》
  链接：<http://bbs.pediy.com/showthread.php?t=95905>

下面开始动工：

1. 用 `LoadPE` 给 `taskmgr.exe` 添加输入函数：`USER32.dll` 中的几个 API（`OpenClipboard`、`EmptyClipboard`、`SetClipboardData` 和 `CloseClipboard`），等一下补丁代码要用到的，直接上图。

   ![](https://bbs.kanxue.com/upload/attach/201007/177817_toc12lfhuy9uc00.jpg)

   `AddImport.jpg`

2. 用 `eXeScope` 添加相关菜单项，资源 --> 菜单 --> 111，插入一项菜单“50128,拷贝命令(&K)”，上图。

   ![](https://bbs.kanxue.com/upload/attach/201007/177817_qlfidff266bbuk4.jpg)

   `AddMenu.jpg`

3. 用 `ZeroAdd` 软件给它添加一个 200KB 的区段，用来写补丁代码。

   ![](https://bbs.kanxue.com/upload/attach/201007/177817_kw5nl1jbynemu3d.jpg)

   `ZeroAdd.jpg`

用 `LoadPE` 可以看到最后一个区段的区段名为 `myCode`，开始地址：`1060000`（虚拟地址）`3CA00`（物理偏移）。

![](https://bbs.kanxue.com/upload/attach/201007/177817_4d7b6w397b8txru.jpg)

`myCode.jpg`

自己总结的一个经验，我认为修改程序的最佳步骤应该为：`LoadPE`（添加输入函数）--> `eXeScope`（添加资源项）--> `ZeroAdd`（添加区段），不按这个次序来可能会出错。

接下来启用强大的 `OD` 在新增区段中添加消息响应代码。

一步一步来，我先在新增的区段中加入以下代码：
