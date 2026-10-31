
# [原创] 对某款摄像头固件的简单分析！

前段时间搬了新家由于没有装宽带就用某万能钥匙蹭了个网。

可能是习惯了吧，连上wifi以后先用工具扫描了下内网设备，发现有个摄像头。

扫描端口显示开放了23，80，554，555，3000等端口，先对`telnetd`服务登陆试试。

用网上能找到的密码，还有loT病毒使用的密码都试过了，没办法只能放弃。

然后尝试登陆web服务，用`admin:123456`直接登陆进去了。而且在web界面还提供了刷机功能，包括固件信息。

这就好办了，直接从网上找相同的固件研究好了。

不过没有找到相同版本的，倒是找到一个高版本的固件，没办法凑合着用吧。

拿到固件当然是先解包了，用`c32asm`打开固件搜索了下关键字`hsqs`居然找到了，说明固件没有加密。

当时也没想分析固件结构只是想看看root密码，打算用`hashcat`跑出来登陆`telnetd`服务，就直接用`7zip`对`squashfs`文件系统解压缩了。

理所当然在`/etc/passwd`文件里找到了用`des`加密的root密码：

`root:0.IQvJd8bXSWU:0:0::/root:/bin/sh`（`password:hdipc%No`）

`root:btJy1tzjqgK3g:0:0::/root:/bin/sh` (???)

可能是电脑太渣跑了一个下午楞没跑出来直接放弃了。当时就想看看摄像头运行了哪些进程与服务与flash的布局。

既然跑不出密码那就把root密码删除 `root::0:0::/root:/bin/sh` 免密码登陆不就行了。

其实摄像头功能是由`/opt/topsee/ts_mgr`这个程序提供的，就没有分析`web_server`。不多说了直接上图。![](https://bbs.kanxue.com/upload/attach/201901/194991_5ZPJ2R39TXFPTZK.png)![](https://bbs.kanxue.com/upload/attach/201901/194991_4ERZQSCFXHEKFHJ.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/194991_2HNQCZX8YQGJZ2U.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-11-1 07:56

被jioushizhu编辑

，原因：

上传的附件：

- [`firmware_TH38A12_V1.0.0.15_20171011200225.bin`](javascript:void(0);)

  （7.10MB，167次下载）

- [`fwtools.rar`](javascript:void(0);)

  （1.43MB，177次下载）
