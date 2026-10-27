
`BitPlainText[i] = (Base64Index[j] << 2) + ((Base64Index[j + 1] & 0xF0) >> 4);`//将第一个4*6Bit组数据左移2位去除高位补的0x00得到6个有效Bit位，从第二个4*6Bit组取得4Bit数据右移4位(包含两个有效位)，两个部分相加得到第一个3*8Bit组数据。

![剖析5](https://bbs.kanxue.com/upload/attach/201907/813468_EKFATYSUWX8ZXVT.png)

`BitPlainText[i + 1] = ((Base64Index[j + 1] & 0x0F) << 4) + ((Base64Index[j + 2] & 0x3C) >> 2);`//从第二个4*6Bit组取得4Bit数据左移4位得到4个有效Bit位，从第三个4*6Bit组取得6Bit数据右移2位得到4个有效Bit位，两个部分相加得到第二个3*8Bit组数据。

![剖析6](https://bbs.kanxue.com/upload/attach/201907/813468_VRJABRYDRNXQZWG.png)

`BitPlainText[i + 2] = ((Base64Index[j + 2] & 0x03) << 6) + Base64Index[j + 3];`//从第三个4*6Bit组取得2Bit数据左移6位得到2个有效Bit位，从第四个4*6Bit组取得8Bit数据得到6个有效Bit位，两个部分相加得到第三个3*8Bit组数据。

![剖析7](https://bbs.kanxue.com/upload/attach/201907/813468_T56GXHR3SXNVZ2T.png)

# 本篇总结：

> 写到这里，我对Base64算法的理解也就分享完了。其实Base64的原理非常简单，但是实现它的过程却十分的有意思，它展现了C语言中位操作的魅力。我感觉只有对位操作有着深入的理解，才能更好的进行密码学的学习。

不知道是为什么在核心代码讲解部分，完全不知道要写些什么。。。。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-8-21 20:32

被QiuJYu编辑，原因： markdown *好转义bug

上传的附件：

- [Base64参考代码.zip](javascript:void(0);)
  
  （1.41MB，80次下载）

- [Base64图片.zip](javascript:void(0);)
  
  （109.27kb，70次下载）
