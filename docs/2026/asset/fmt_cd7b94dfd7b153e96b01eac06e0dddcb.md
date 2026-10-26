
![](https://bbs.kanxue.com/upload/attach/202106/394321_53WBYHK8982VNZQ.png)

## 总结

经过一番折腾，目前满足我当前需求，可以正常`dump`出错的软件，但是`LordPE`不支持64进程，所有可能某些情况下还是鸡肋。

很少这么手工修改软件，思路还是挺重要的，要不是搜到那篇帖子，我可能就会采用勾上`procs.dll`的导出接口和`dump`函数，但随便几点某个进程，选择`dump`时，会弹出一个我的界面选择真实的目标进程，然后返回调用`dump`函数，也同样是可以同样的功能。

最后立个flag，有时间还是想写个支持64进程的增强版`LordPE`。或者我孤陋寡闻，目前已经有类似程序，有知情大佬，请不吝告知。

[有兴趣移步公众号](https://bbs.kanxue.com/elink@c29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4\)9J5k6i4N6W2K9i4S2A6L8W2\)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2\)9J5c8V1N6C8d9V1y4H3y4g2A6Z5L8r3c8X3y4$3k6c8K9h3k6#2f1V1W2v1x3@1p5`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
