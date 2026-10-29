
第二条协议其实还是这个接口，就是参数多了一点，多的那点就是第一条协议的返回值。虽然看懂了，但是不知道她是怎么解析返回值的啊。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_l8hwqvwv2ufharg.png)

找到`install_channel`这个参数`hook`赋值的地方，打印堆栈。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_rlz5kz9bhqwzuul.png)

发现是这里读的。

![图片描述](https://bbs.kanxue.com/upload/attach/201712/630232_zj5vv7pvze6de83.png)

## 0x07 后续

嗯。加密、解密逻辑都找到了，模拟代码吧。直接拿它的代码就可以，稍微修改一下，根据`hook`下来的参数，多比较几次。测试，请求参数一模一样地还原……

由于不会排版，排版是参考了别人的。

最后，感谢论坛中各大神的分享，让我学习到了很多东西。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
