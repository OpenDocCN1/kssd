
调用方式参考AlleyWind在其它进程空间中调用`SetWindowDisplayAffinity`给其它进程的窗口设置显示掩码实现防捕获（[AlleyWind - `Operation.c`](https://bbs.kanxue.com/elink@5d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6m8L8r3I4W2P5g2N6A6L8X3c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3f1$3!0#2M7X3y4W2i4K6u0r3b7h3I4D9k6i4W2i4K9h3&6V1i4K6u0r3e0%4m8W2M7X3q4@1K9h3!0F1i4K6u0W2j5H3`.`.)）：

![](https://bbs.kanxue.com/upload/attach/202207/853701_WB4YA2TXVUPN2ZU.jpg)

可以看到，经过封装，让目标进程调用任意API已经变得很容易了，用AlleyWind让记事本反截屏：

![](https://bbs.kanxue.com/upload/attach/202207/853701_CNPWFUWPW8MJMS9.gif)

可以看到，AlleyWind中勾选防捕获（Anti Capture）后，我们的远端线程注入到记事本里，让记事本调用`SetWindowDisplayAffinity`，为自己的窗口开启了防捕获功能。于是截图工具一开始截图，记事本窗口就立即消失了。

# 五、总结

经过封装，像上面的代码仅需几行便能实现。C语言编写`ShellCode` -> 在目标进程中调用`ShellCode`并支持花式入参出参 -> 远端线程执行任意API，一路走来不容易。这样的技术可以“借花献佛”，也可以“借刀杀人”——**让其它进程调用API执行进行恶意操作，一切审计结果都会算到其它进程头上，毕竟我们连DLL也没注入。唯一能让远端线程留痕的，只有安装的第三方安防软件了，如`SysMon`的审计。**

本文实现的`Hijack_LoadProcAddr`（获取目标进程函数地址）与`Hijack_CallProc`（调用目标进程函数）都是[NTAssassin](https://bbs.kanxue.com/elink@d72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6z5g2p5q4K6M7$3q4K6M7$3W2F1)导出的函数，GitHub源码和`lib`都有，欢迎一键三连，更欢迎指正与交流。

Ratin/[ratin@knsoft.org](https://bbs.pediy.com/thread-267416.htm)

国家认证 系统架构设计师

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)
