
> bbbbbbbqQXpb1bbbbbbbz0KkbbbbaMdqG2bbbbbbbapb1bbTsbBbbbbbbbS1PxdqG2bbbbbqQXpb1bb4obska!bqkbkb\*MdqG2b
> bzbzqQXpb1bbaQbbbalb36bUbbbbbbbbb2bQqQXpb1bbUbU6CHsbGbb1bbbbbbbbbbbbbbbbbbbbbGbbbbbbbbbUbbbbbbbbbbb
> bbbbx4HRe0E!A4xWspY!A3Mh

该段序号是我构造的在`launches`递增到5时候(就凑巧产生了碰撞)的注册信息。

用户注册信息的二进制格式不再赘述, 只有"关键"的五个时间字段进行了交织编码, 可能是为了让字段的变化尽可能分散避免碰撞吧。

`regtime` 19,17,15,13
`time1` 7,5,3,1
`time0` 6,4,2,0
`time8` 18,16,10,8
`launches` 14,12,11,9

`regtime` 不能小于2020年3月, 注册标志是限期订阅的话, `time0`从当地时间一天前算过期. `time8`是`time0`的镜像。

我又收集了一下其他版本的15字节后缀和base64表, 回头优化下注册机发上来, 有时候生活就像一部老式游戏, 只是为了清掉一个红点就要往返于各个大陆最后开始荒野求生~~~

老版本的算法也是类似, 网上破解都是爆破, 可能是大佬们觉得md5原像攻击不现实吧!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-8-22 18:54 被曾半仙编辑，原因：[#加密算法](https://bbs.kanxue.com/forum-4-1-5.htm)
