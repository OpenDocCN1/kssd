
# [原创]QQ记住密码MD5读取器(源码)

这不是腾讯的漏洞，也不是最近才有的，而是小菜无意中逆向出来的，我想相信很多大牛都知道这个。小菜觉得没什么用，就发出来给大家玩玩

注意：本程序仅供学习，要是有侵权什么的，请大家告诉我，我会马上删除的

先发张我的使用截图，本人在很多人电脑上都测试成功

![](https://bbs.kanxue.com/upload/attach/201211/452670_moarj40tpsjqb9y.jpg)

说下实现原理，在腾讯的用户数据目录下，有可能是`C:\Program Files\Tencent\QQ\Users\`，也有可能在我的文档下，在`Tencent Files\All Users\QQ`目录下有一个文件`Registry.db`，里面的`Main_LoginAccountList`流包含QQ帐号列表

而在`Tencent Files\QQ号\QQ`目录下，也有一个`Registry.db`文件，里面`LoginInfo`流里面有一个字段(暂且叫字段)，名为`bufPassword`，数据是加密的，如果没记住密码的数据是空的。

解密算法小菜只逆向出了一点点，虽然只有这一点点，但也能解密出具体数据，但可能有一小部分人无法解密。解密后的数据包含两个字段，其中一个就是`bufSavedHashOne`，这是记住密码的MD5

在运行本程序之前，请退出所有QQ，这是因为没退出的QQ会占用`Registry.db`。另外请将程序放在和腾讯QQ相同的磁盘，不需要放在QQ目录，只需要在同一个磁盘就行了

反正没什么用，就把源码发下来

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`Test.jpg`](javascript:void(0);)（53.22kb，130次下载）

- [`QQRPwMD5.zip`](javascript:void(0);)（336.63kb，1039次下载）
