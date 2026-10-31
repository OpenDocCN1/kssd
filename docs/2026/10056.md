
# [原创]使用键盘钩子记录按键

大三的学生哈，很多知识都处于入门阶段，勿见笑![](https://bbs.kanxue.com/view/img/face/12.gif) 上学期用 vc 写了一个截获键盘消息的程序，希望能和新手一起学习![](https://bbs.kanxue.com/view/img/face/28.gif)

我有个问题想请教一下，hook是写在DLL里的，动态加载，当程序具有输入焦点时候，能记录按键，但是程序没有焦点时候，就没法了，查了一下，写在独立DLL中的hook，不就是全局的钩子吗？恳请指教一下。。。。

> A global system hook is a system hook that is called when the specified messages are processed by any application on the entire system.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`KeyboardHook.zip`](javascript:void(0);)

  （3.32kb，62次下载）
