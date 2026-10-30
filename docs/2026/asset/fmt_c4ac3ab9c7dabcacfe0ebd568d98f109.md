
`Exp`中调用`AddLogContainer`，对应内核中的函数为`CLFS!CClfsLogFcbPhysical::AllocContainer`，该函数的`this`指针指向对象`CClfsLogFcbPhysical`。

`CClfsLogFcbPhysical`对象`0x2b0`的位置指向`CClfsLogFcbPhysical`对象，该对象`0x30`的位置保存一个指针，该指针指向一段`0x90`大小的heap内存，这里称之为`clfsheap`，`clfsheap` `0x30`保存指向base block的指针。

![图片描述](https://bbs.kanxue.com/upload/attach/202308/865065_CYWGCQX3XBUHJE6.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202308/865065_FXMTMCUW5BC23RY.png)

这里分别对trigger clfs base block这两个偏移下读写断点`398`下写断点`058`下读断点。

![图片描述](https://bbs.kanxue.com/upload/attach/202308/865065_YESFC8NKYAPCBZA.png)

正常情况下

![图片描述](https://bbs.kanxue.com/upload/tmp/865065_JBHAJQ8A99BGPUS.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-11-6 15:36 被kanxue编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
