
# [原创]屠龙技之ODbgScript2.25脚本编写

可以忽略前三个字，只是个噱头。无他，仅仅是展示 [ODbgScript2.25脚本](https://bbs.pediy.com/thread-253847.htm) 如何编写。

以 2019看雪CTF > 晋级赛Q3 > 第三题：庄周梦蝶为例，编写了一个穷举破解脚本，能不能出结果，看运气了。不过在这里不重要，主要是展示一下脚本写法。屠龙刀耍起！

原程序的逻辑是个开放式的判断分支，如果输入正确，走对的分支，否则走错误的分支。而穷举破解则是封闭式的循环判断并有一个正确的出口，有点像字母Q。

![](https://bbs.kanxue.com/upload/attach/201910/14494_FMM5V5JYTMH5CP5.png)

思路有了，剩下的就是先找到源程序存放初始序列号的内存地址1，以及算法判断正确与否的 `jxx` 位置2，然后由位置2的跳向错误改为 `jmp` 到验证算法的开始处，在内存地址1处写入由 `js` 产生的随机32位序列号，进行 `loop` 循环，正确则跳出大循环。通过OD简单分析便可得知以上信息：

![](https://bbs.kanxue.com/upload/attach/201910/14494_AMKMZYYAFC5RCC5.png)

![](https://bbs.kanxue.com/upload/attach/201910/14494_7A64MVUCH8K4Q4Q.png)

下面该上屠龙刀了，我去，刀呢？还好倚天安在。

![](https://bbs.kanxue.com/upload/attach/201910/14494_N8RRXFCZT5ZW8DB.png)

![](https://bbs.kanxue.com/upload/attach/201910/14494_S5KFQ75BENXZGJF.png)

上面注释已经很清楚了，并可以看到 `scjs`（`ss`）命令中对 `js` 语句随机产生32位数字序列号的支持和强大处理。

附件是修改过的 `gene` 程序，主要是把字符输入过程和中间错误提示去掉了，这样保证程序可以不中断循环，对比下原程序就可知道修改了哪些，有部分语句需要在脚本中动态修改才行。其实所有修改都可在脚本中动态修改，只不过有些麻烦。

下图是脚本运行结果：

![](https://bbs.kanxue.com/upload/attach/201910/14494_BE8HAKJM6XX23XW.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-11-2 18:23

被 `ssarg` 编辑，原因：

上传的附件：

- [`Gene22.exe`](javascript:void(0);)
  （83.50kb，19次下载）

- [`ckgenesj1.osc`](javascript:void(0);)
  （1.07kb，17次下载）
