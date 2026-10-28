
在 vt 上查找下载地址：`099K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6W2M7%4y4#2j5h3g2Q4x3X3g2S2k6g2)9J5c8Y4N6H3i4K6u0V1K9h3&6U0L8s2g2V1k6i4y4Q4x3V1k6X3L8$3&6@1M7#2)9J5c8X3I4S2N6W2)9J5k6h3A6H3k6H3`。

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_YCWQ67BUVA7Q867.png)

成功找到名为 `lav.jpg` 的攻击文件，且这里可以看到该文件是 .net 平台的。

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_66SDFZK2DPW53UE.png)

下载到本地查壳可以发现，样本由 C# 编写。

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_4S8K5TZU22DF8BA.png)

该样本结构如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_H37NY4FU9RPNCV2.png)

通过对 `main` 函数的分析可以得知：

1. 程序首先会尝试通过 `Assembly.GetExecutingAssembly().GetManifestResourceStream` 函数获取名为 `compressed` 的资源
2. 成功获取之后会复制资源到 `arrany`
3. 将 `arrany` 传入到 `Decompress` 函数
4. 创建新线程，`start` 参数为经过 `Decompress` 函数处理过的 `array`

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_N95QZVGRRRCAQZF.png)

我们查看一下 `Decompress` 函数的内容可以发现该函数是一个解密函数：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_Y45STEUDU5AK5GZ.png)

在调用 `Decompress` 函数处设置断点：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_C8VQYNQ6GAP67WS.png)

查看一下当前 `array`，是一个大的字节流

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_TUN8Q2UBVUQ53GM.png)

`F10` 单步往下走，得到返回值

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_RPH68FVX43MT4R4.png)

展开返回值，很明显解密出来的内容是个 PE 文件

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_4M6JAVKBWA9M8JU.png)

此时可以选中该变量，然后鼠标右键 -> 在内存窗口中显示：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_Y2TCXBA49ZTP92V.png)

内存如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_TY3ZSCG25P67FJ8.png)

保存为 `dump.exe`，可以看到该文件依旧是由 C# 编写：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_EJEXEVFS2BF58QZ.png)

`dump.exe` 的入口如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_YQ78A876G9K7GV8.png)

可以看到这里 `dump.exe` 是经过混淆的，尝试使用 `de4dot` 去混淆：

去混淆之后函数结构如下，比之前好了一点点，然后尝试在 `main` 函数设置断点：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_3G6JHAQTJ9DXXPE.png)

提示异常

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_74D2TJPZ25GVUMX.png)

这里其实不应该直接在 `main` 函数设置断点的，我偷了个懒，尝试直接在 `main` 函数设置断点

但其实在 `main` 函数之前，可能还会运行其他的内容，比如 `public static` 属性的变量赋值，比如下面的代码：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_C48D3NPJ75HW8VX.png)

这里可以看到，`main` 函数中只有一个输出语句

在最后声明了一个 `static` 的变量 `str1`，但是并未引用过 `str1`

但是运行代码之后可以发现，由于 `static` 变量 `str1` 的声明，会导致 `str1` 的赋值会在 `main` 函数之前执行，这是由 C# 的编译顺序决定的。

所以在该样本中，应该也是有类似的操作，导致代码还没有跑到 `main` 函数就抛出异常了。

终于在 4970 处找到了异常的原因

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_PBE2N3E9D36FSHV.png)

并且在一个长语句中，代码多次调用了 `smethod_0` 函数：

这里可以看到 `smethod_0` 函数是 ECB 模式 AES 的解密函数，参数 1 是 base64 编码的字符串，参数 2 是 AES 解密所使用到的 key

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_QSDDQ2FX3UH2CY2.png)

现在在 4970 行地方设置断点，代码成功断下来：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_ZVA2DS8KTJE83CV.png)

往后跟两步之后定位到了异常原因，原来是因为我这个虚拟机卸载了网卡，所以在 `GetExternalIP` 的时候导致异常

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_EF9UA288TE78T7A.png)

尝试通过 `DownloadString` 函数访问 `"857K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2X3j5$3!0F1k6X3W2Y4i4K6u0W2L8h3g2Q4x3V1k6A6M7l9."`.

这里应该是通过该地址获取本机出口 IP 地址。

这里直接将 `DowinloadString` 方法替换试试：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_2923AKY86FMMD4G.png)

过了之后直接 F10 运行，组装出来变量如下：

```text
"+------------- Client INFO -------------+\r\nIP: 192.168.1.1\r\nHWID: 0F8BXXXXXX0906EA\r\nOwner Name: WIN-IHXXXXXXIMB\r\nFull OS Name: Microsoft Windows 7 家庭普通版 \r\nOS Platform: Win32NTOS Version: 6.1.7601.65536\r\nSystem Boot Mode: Normal\r\nPhysical Memory: 2.80 GB Available Of 4.09 GB \r\nVirtual Memory: 1.90 GB Available Of 2.04 GB \r\nDate: 2019/12/27 12:01:46\r\n-----------------------------------------"
```

可以看到，样本首先会获取的信息有：

1. 计算机出网 IP
2. 计算机 HWID
3. OwnerName
4. Full OS Name
5. OS Platform
6. OS version
7. System Boot Mode
8. Physical Memory
9. Virtual Memory

接着就会运行到 `Main` 函数：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_7KHACZVUCJEPZF9.png)

`Smethod_14` 用于获取 Chrome 浏览器的隐私信息：

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_JP7UPUCG9SVT5HU.png)

`Smethod_16` 用于获取 `"C:\Users\Shyt\AppData\Roaming.purple\accounts.xml"` 的信息，通过查询可以的知，该目录对应程序 pidgin

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_DJASFAAECG463QB.png)

`Smethod_4` 用于获取 `@"C:\Users\Shyt\AppData\Local\Vivaldi\User Data\Default\login data"`，也就是 Vivaldi 软件

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_H2PSXX3A3CVZTQ9.png)

后面的内容都差不多，就不截图了，样本还会获取的信息有：

- FTP 凭证
- Opera 浏览器隐私信息
- Outlook 隐私信息
- UC 浏览器隐私信息
- 360 浏览器隐私信息
- 猎豹浏览器隐私信息
- Firefox 隐私信息

最后会将收集到的信息打包，通过 FTP 或者 SMTP 的形式上传

![图片描述](https://bbs.kanxue.com/upload/attach/201912/757351_YYBQP6YW52ZRN4E.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-12-27 15:10 被 jux1a 编辑，原因：

[#.NET平台](https://bbs.kanxue.com/forum-4-1-7.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
