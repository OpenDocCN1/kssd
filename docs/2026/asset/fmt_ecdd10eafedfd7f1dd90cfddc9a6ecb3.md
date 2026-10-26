
# [旧帖][原创]WinAudio Recorder破解手记0.00雪花

【文章标题】: WinAudio Recorder破解手记
【文章作者】: 阿蔡
【软件名称】: WinAudio Recorder2.2.2
【下载地址】: 自行搜索
【编写语言】: Microsoft Visual C++ 7.0(`PEID`识别)
【使用工具】: `od` & `ida` & `peid`
【作者声明】: 第一次写技术文章,错误在所难免,敬请诸位大侠赐教!

第一次成功破解程序(简单的CrackMe不算程序吧,不过此程序也很简单),并写出注册机,兴奋不已,写下笔记与大家分享(文笔不好敬请原谅,排版也不好![](https://bbs.kanxue.com/view/img/face/50.gif)),错误之处欢迎大家指正.

我并没有尝试在网上搜索此软件的破解教程,不知道网上有没有,不过我的目的在于学习,所以这也就没关系了.

首先用`PEID`查壳,如图:

![](https://bbs.kanxue.com/upload/attach/200910/292177_p64ow77lf2tvf85.jpg)

没有加壳,正好适合我等菜鸟.运行程序输入假注册码注册,结果如图:

![](https://bbs.kanxue.com/upload/attach/200910/292177_f3p4gx1ljzu5joz.jpg)

有提示信息,记之.`OD`载入,F9运行.出来注册窗口,用`Ultra String Reference`插件搜索字符串信息,找到刚才提示的字符串,如图:

![](https://bbs.kanxue.com/upload/attach/200910/292177_id4ax654luwisv1.jpg)

双击字符串来到下面的地方:

```
0040F19C   .  6A 00         PUSH 0
0040F19E   .  6A 00         PUSH 0
0040F1A0   .  68 D4E04300   PUSH Recorder.0043E0D4                   ;  please input correct registration code!
0040F1A5   .  E8 6B160200   CALL Recorder.00430815
0040F1AA   .  5F            POP EDI
0040F1AB   .  5E            POP ESI
0040F1AC   .  5D            POP EBP
```

通过`IDA`对比,向上找,在不远处发现调用了`CWnd::UpdateData`(取得用户名和密码)这个函数,在这个函数的后面下断进行分析,下面为分析结果:
