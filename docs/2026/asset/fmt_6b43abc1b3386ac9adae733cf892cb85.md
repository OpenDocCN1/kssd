
# [原创]如何调试xxProtect

## 0x1

今天终于拿到了萌妹以求的xxprotect的demo，

![](https://bbs.kanxue.com/upload/attach/201901/99882_EZJHAPT6RGJBJKJ.jpg)

![](https://bbs.kanxue.com/upload/attach/201901/99882_Z2AG94UCBB3TZD7.jpg)

好久没搞反调试这一块了。心痒痒，开搞。

![](https://bbs.kanxue.com/upload/attach/201901/99882_96RN4U4UGTPS94P.jpg)

## 0x2

打开demo,长这样。

![](https://bbs.kanxue.com/upload/attach/201901/99882_UUAKNCQ2GKCF492.jpg)

是个64的程序，直接上x64dbg ,发现竟然找不到进程.尝试直接`OpenProcess`，报错

`STATUS_INVALID_CID`

![](https://bbs.kanxue.com/upload/attach/201901/99882_MQVS6XFXYG5X4E8.jpg)

根据msdn描述，很清楚了

![](https://bbs.kanxue.com/upload/attach/201901/99882_3SZDMYTBE7SVCBT.jpg)

做个实验测试一下

![](https://bbs.kanxue.com/upload/attach/201901/99882_6VES7XTGZEXRZHM.jpg)

系统根据cid拿不到eprocess信息，看看`PsLookupProcessByProcessId`的实现

![](https://bbs.kanxue.com/upload/attach/201901/99882_949YTX56B96F2JB.jpg)

可以考虑自己维护下这个东西`cid->cidtableentry`。

## 0x3

![](https://bbs.kanxue.com/upload/attach/201901/99882_YEXYNKK5N9CS2PA.jpg)

hook一哈`PspReferenceCidTableEntry`

![](https://bbs.kanxue.com/upload/attach/201901/99882_DZU7W9QDQXADSP9.jpg)

在进程创建的时候调用下`ProcessStart`加入当前的table.结束的时候调用下`ProcessDelete`移除掉table

然后跑起来..发现有点问题。在进程退出的时候，系统会GG掉。那只能换个办法了，hook吧`PsLookupProcessByProcessId`

## 0x4

代码稍微改一下，写一个维护类

![](https://bbs.kanxue.com/upload/attach/201901/99882_5WWAYBJ6VR9MU5G.jpg)

然后进程创建/结束的逻辑一样,不过table变成eprocess了

![](https://bbs.kanxue.com/upload/attach/201901/99882_FMRUKXZ73Y4YRU8.jpg)

加载hook

![](https://bbs.kanxue.com/upload/attach/201901/99882_88BBQH3PKBPMDEU.jpg)

一波操作后，我们就可以让调试器`OpenProcess`这个demo了

## 0x5

x64dbg虽然能打开进程句柄了，但是还是不能附加调试。在`DebugActiveProcess`的时候失败了。稍微跟进去看看，发现问题出在了`DebugActiveProcess`的`DbgUiDebugActiveProcess`的`DbgUiIssueRemoteBreakin`创建附加线程上

![](https://bbs.kanxue.com/upload/attach/201901/99882_S64C8ES3YS6WWHQ.jpg)

创建线程失败，导致x64dbg直接脱离调试状态。其实说到底这个线程要不要都可以，我们直接把这里PATCH掉。再附加.

嗯?直接可以调试了？

`MessageBox`下个断点试试，也能正常触发。

![](https://bbs.kanxue.com/upload/attach/201901/99882_HADQW26R69MQXG5.jpg)

下个硬件断点试试

![](https://bbs.kanxue.com/upload/attach/201901/99882_2SJERBD76HTWD5J.jpg)

好吧。这就完事了

## 0x6

1. hook `PsLookupProcessByProcessId`.自己维护`eprocess`和`cid`的关系

2. 去掉`DebugActiveProcess`的`DbgUiDebugActiveProcess`的`DbgUiIssueRemoteBreakin`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
