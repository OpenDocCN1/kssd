
# [原创]解决多进程ptrace反调试保护的一种方法

## 前言

我们经常看到很多加固都采用多进程`ptrace`的方式来反调试，这里想到一个小技巧绕过这种保护，测试目标是一款娜迦加固的APK，效果如下：

![](https://bbs.kanxue.com/upload/attach/201607/592070_0bosu26riudhcx8.png)

## 原理

多进程都是通过`fork`出来的，因此我们修改`/bionic/libc/bionic/fork.c`里面的`fork`函数来使得目标进程`fork`失败，代码如下：

![](https://bbs.kanxue.com/upload/attach/201607/592070_7pmssoeyxjigy32.png)

## 结果

修改`fork`函数后，目标APK只剩下一个进程了，使用`gdb` `attach`正常，效果如下：

![](https://bbs.kanxue.com/upload/attach/201607/592070_bhm4zuhhdnh2kzn.png)

## 结束语

有兴趣的同学可以试试其他的加固系统。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [source.png](javascript:void(0);)
  （24.50kb，56次下载）
- [single.png](javascript:void(0);)
  （7.52kb，25次下载）
- [multi.png](javascript:void(0);)
  （7.14kb，28次下载）
