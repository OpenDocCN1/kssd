
# [原创]不用`inf`安装ndis filter驱动

killvxk之前分享了一个代码：

https://bbs.pediy.com/thread-226472.htm

可惜发现晚了，那个连接无效了。去老v的gitee仓库转悠好几圈，没发现那个安装ndis filter的代码在哪个仓库，可能是未公开的仓库。我还去问了gitee那边，回复说是gitee之前有一次升级，`username/codes` 这种的URL仅对用户自己可见，不能公开访问了。

老v神出鬼没…联系不上…只好自己撸了一个。

调试上比较简单，拿着 processmonitor 配合 vm 建一些快照，看看日志，就完事了。

然后就是撸代码。

对于 `xx` 项目来说，NDIS Filter 确实不好用，还是上 WFP 吧，弃坑。有个 bug，先不解决了。
