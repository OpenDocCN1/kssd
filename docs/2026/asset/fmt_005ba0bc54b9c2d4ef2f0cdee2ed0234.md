
# [原创] Xposed 监听微信文本消息并关键字拦截

## 严重声明

本文的意图只有一个，就是通过分析 App 学习更多的逆向技术。如果有人利用本文知识和技术进行非法操作牟利，带来的任何法律责任都由操作者本人承担，与本文作者无任何关系。最终还是希望大家能够秉着学习的心态阅读此文。

## 功能说明

最近别人要我做个小功能，就是关键字检测：当他在微信发送我们设置的关键字时，就给他屏蔽掉，不让他发送。我这里设置的关键字是“中奖”，我们先看看效果图。

![](https://bbs.kanxue.com/upload/attach/201902/3_44DYA34QNZCBMPE.jpg)

## 分析

我们肯定要先监听到微信的发送内容，然后再屏蔽发送的内容。那我们如何入手呢？老规矩，我们用 `ddms` 轨迹录制，录制点击发送的事件。

## 开工

我们点击录制，点击发送的按钮，看做了哪些事情。录制完成之后搜索 `click` 点击事件，我们看到这样一个类很可疑。

![](https://bbs.kanxue.com/upload/attach/201902/3_W4CEWWMDYF9YBSF.jpg)

我们 Hook 这个类的这个方法，拿到传进来的参数。我们打印传进来的参数看看。

![](https://bbs.kanxue.com/upload/attach/201902/3_F5R4MZ2QGSAZ4PM.jpg)

![](https://bbs.kanxue.com/upload/attach/201902/3_QHEWZTQM5BYD3BX.jpg)

和我们想象的一样，这里就是监听文字消息的方法了，我们完成了百分之 90，文字监听成功了。

## 拦截关键字

那么我们怎么让关键字不发送出去呢？

看了编译出来的源码就能懂了，它有个返回值，那我们尝试把返回值改变看看：如果包含“Test”关键字，就把返回值改为 `true`。

![](https://bbs.kanxue.com/upload/attach/201902/3_WCM93RMNHMSA4FS.jpg)

![](https://bbs.kanxue.com/upload/attach/201806/SWVRMSZCT97XN5Y.jpg)

到这里为止，很明显了，监听消息和屏蔽关键字都已经成功。

有很多同学想要 demo，我就传 GitHub 一份，方便大家学习，下载地址回复可见。

地址：`739K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7K9h3!0F1k6h3y4U0i4K6u0r3g2$3g2o6K9r3q4@1e0i4y4Y4d9r3!0G2K9@1c8W2L8h3)9`.

大家喜欢可以 Star 一下哦！

版权声明：本文为博主原创文章，转载请声明出处。

---

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-2-2 13:46 被 admin 编辑，原因：图片本地化
