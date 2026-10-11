
# ThreadMast 逆向（C 实现）

> 原文：[`www.pediy.com/kssd/pediy10/101161.html`](https://www.pediy.com/kssd/pediy10/101161.html)

`ThreadMast` 是一款 CPU 管理程序。官方网址：`http://threadmaster.tripod.com`（需翻墙）。

它可以实时监控各进程的 CPU 使用率，若发现有进程使用率超过用户设置的配额，则对此进程进行限制。使目标进程的 CPU 占用率始终处在用户设置的配额范围内。

软件没有图像化界面，是一个服务程序，用户可以通过注册表来设置 CPU 配额，取样时间，白名单进程等信息。还可以设置全局 CPU 配额和单进程 CPU 配额等等，十分灵活。

实际使用中 `ThreadMast` 运行很稳定，而且自身占用 CPU 很小。基本实现了 CPU 的限制功能。

把核心逆向成 C 语言实现。

没什么技术含量，不说啥啦，有兴趣看代码吧。

`blog`: `http://hi.baidu.com/index09`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `threadMast.rar` |

关注，找了个 `threadmaster`，

最好把对应的 `pdb` 文件也放上来~

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) | `tminst112.zip` |
