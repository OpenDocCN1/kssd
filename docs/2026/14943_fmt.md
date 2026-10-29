
# [原创]某房产网混淆 JavaScript 分析与还原

> 仅供学习研究，请勿用于非法用途，本人将不承担任何法律责任。

## Chrome Network 面板分析网络请求：返回的数据是一个加密文本

![](upload/attach/202112/938036_B7V3AZDVZW6WRZH.png)

前面有三组数字和加密数据。

通过分析发现，解密位置都是混淆代码。

![](upload/attach/202112/938036_MVS2858U75XZWY5.png)

![](upload/attach/202112/938036_8WW9RHA82PHEH37.png)

原本想把 JavaScript 抠下来，然后用 Python 调用 Node.js 进行解密。抠下来后发现 Node.js 运行报错，而且试用“代理 IP”时，返回的数据是解密失败的，好像和 IP 有关系。所以使用了 Sekiro JS RPC 调用，今天没事具体研究了它的解密方式。

![图片描述](upload/attach/202112/938036_Y9YBWPREAM6BGU5.png)

### 1、混淆还原

![图片描述](upload/attach/202112/938036_65U8VRYKMU7N4PP.png)

![图片描述](upload/attach/202112/938036_H2KT6PEPGUBG9PD.png)

所有变量名都是通过`_0x2e12`方法进行计算得出，所以第一步手动把所有混淆替换掉。

![图片描述](upload/attach/202112/938036_ARMYP6Z56FP9SKK.png)

![图片描述](upload/attach/202112/938036_8P8MU5UPVZCGUY5.png)

### 2、再替换掉一些固定的执行方法和参数

![图片描述](upload/attach/202112/938036_A4WVX4PM653J5B7.png)

### 3、重点分析 `deBase64` 和 `deBase64_cutStr` 方法

其实重点就在`deBase64`和`deBase64_cutStr`方法上，通过这两个方法就大概看得差不多了。

具体逻辑就是通过返回的前三组数字和`ip.split('.')`进行运算，然后计算 IP 的位数，传递给`deBase64_cutStr`方法进行`subString`后重新组合，重复 3 次。组合后就是一个 Base64 加密字符串，可以直接解密。

![图片描述](upload/attach/202112/938036_JWPBVVYJWKU639S.png)

通过 3 次`deBase64_cutStr`方法可以把 Base64 参数内插入的随机值去除掉，这样就能解密了。

![图片描述](upload/attach/202112/938036_259C6WWC6C7DV5G.png)

> 所以解密代码没变，如果请求数据的 IP 变了，这个代码是解密不出来的。

## Python 还原

![图片描述](upload/attach/202112/938036_G5JB96CMHY7QB5E.png)

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-12-23 17:53

被`wx_彩旗`编辑。

原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)
