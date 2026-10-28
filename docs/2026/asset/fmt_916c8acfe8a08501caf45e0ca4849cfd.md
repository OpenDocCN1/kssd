
![](https://bbs.kanxue.com/upload/attach/202101/860119_TPNV26BATATU6G9.png)

![](https://bbs.kanxue.com/upload/attach/202101/860119_624HB83YBQNK6W4.png)

使用 `InternetOpen` 来初始化应用程序

![](https://bbs.kanxue.com/upload/attach/202101/860119_AY25YFEBKFGHK6J.png)

使用 `InternetConnectW` 来连接恶意IP地址

![](https://bbs.kanxue.com/upload/attach/202101/860119_GWGUET2RDUEQ7HF.png)

使用 `HttpOpenRequestW` 发送本机信息等加密数据

![](https://bbs.kanxue.com/upload/attach/202101/860119_A4WCM4U4XADWKKZ.png)

利用 `InternetReadFile` 函数从打开的句柄中读取数据

![](https://bbs.kanxue.com/upload/attach/202101/860119_6RJJTXDX94876MG.png)

发送完数据之后，遍历 `C:\Windows\System32\Ryqsnymjzf\` 目录，将目录下的所有文件都删除

![](https://bbs.kanxue.com/upload/attach/202101/860119_QS638N8YF8JSQTJ.png)

![](https://bbs.kanxue.com/upload/attach/202101/860119_ENX5T82WPBNGJ49.png)

## 现网监控思路

如果想要对Emotet家族进行监控，可以从流量方面入手。对捕获到的流量进行分析，在调试中发现了一些硬编码的东西，通过这些可以对HTTP的header及body部分进行检测。

在流量中发现一个流具有这样的特征，则说明该机器已感染Emotet木马。

![](https://bbs.kanxue.com/upload/attach/202101/860119_5F5DA5SCFT46VCE.png)

也可以使用Yara规则来匹配样本，只是在编写Yara规则时，要在精确与全覆盖两方面有所取舍，需要大量的样本做支撑。

## 总结

有时候就会发现，逆向就是要大胆假设，小心求证。

分析前会觉得样本好难分析啊，分析完成之后就会觉得索然无味。。。

密码： `infected`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [a268e9e152c260a0e80431aa8d6df187d9f24a1b6be71328ea14320436083f51.doc.zip](javascript:void(0);)
  （84.08kb，37次下载）
