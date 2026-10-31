
# [原创]关于010Editor V9.0的暴力破解

`010Editor`是一款快速且强大的十六进制编辑器，用来编辑二进制文件，功能非常强大，由于是一款收费软件，所以对它尝试进行破解。

## 使用工具
`OD`

首先安装`010Editor V9.0`，安装之后，查看帮助，发现有试用期30天

![](https://bbs.kanxue.com/upload/attach/201811/829513_24UB8Q9D4G3UCSR.jpg)

想要破解一款软件，重要的是先收集信息，要了解到它是用的什么编译器，用的什么图形界面库，知己知彼，才能百战百胜。

打开文件的安装目录：

![](https://bbs.kanxue.com/upload/attach/201811/829513_3ARMSTY74XYQ9ND.jpg)

发现用的是`QT`的图形界面库，再用`Exeinfo PE`扫描一下：

![](https://bbs.kanxue.com/upload/attach/201811/829513_XESPZ537ZCZJC5J.jpg)

没有加壳，可能用的编译器是`VS2013`，接下来，开始破解

## 定位注册校验

找到`010Editor`的注册界面

![](https://bbs.kanxue.com/upload/attach/201811/829513_KVBY8XQKFHTG4SJ.jpg)

点进去，输入一个假的用户名和序列号，会弹出一个用户名或序列号错误的窗口

![](https://bbs.kanxue.com/upload/attach/201811/829513_H2UTHFRYTDYNBQY.jpg)

看到这样的界面，我们这时候就有了多种选择：

1. 根据给出的错误信息，搜索对应的字符串
2. 下 API 断点，栈回溯分析

首先就用简单的方法对字符串进行搜索，如果搜索不到，可能字符串已经加密，再改用第二种方法

打开`OD`，将`010Editor`拖入`OD`，右键中文搜索引擎，搜索`ASCII`

![](https://bbs.kanxue.com/upload/attach/201811/829513_3VK6XKTRHNPB66E.jpg)

如图，已经搜索到注册失败的字符串

![](https://bbs.kanxue.com/upload/attach/201811/829513_5SUMZUAXX9EVRPM.jpg)

双击进入

![](https://bbs.kanxue.com/upload/attach/201811/829513_ADNND9SX2RAV7HF.jpg)

可以得知这就是我们要找的地方

![](https://bbs.kanxue.com/upload/attach/201811/829513_UB4E97325PXHJTV.jpg)

分析上下文，找到了可能是注册成功的字符串

![](https://bbs.kanxue.com/upload/attach/201811/829513_DVQ3FWMGERMH7QV.jpg)

这里可能是我们要找的关键跳转

![](https://bbs.kanxue.com/upload/attach/201811/829513_2UGW96T3X6VY9Z9.jpg)

我们把它`nop`掉试一试，同时复制到可执行文件，保存。

我们打开修改后的`010Editor`

![](https://bbs.kanxue.com/upload/attach/201811/829513_CZ97EPJNDZCEGH4.jpg)

输入同样的信息，显示已经授权，但是我们发现之后的每次打开都需要重新授权一下

说明在其他的地方一直还是有检查，虽然已经破解成功，可以正常使用，但是还是不是很完美，每次弹框都很烦，所以我们进一步分析

![](https://bbs.kanxue.com/upload/attach/201811/829513_KP37X9W2AZAZ2SD.jpg)

我们`nop`掉的跳转的比较`cmp dei, 0xDB`

转到跳转过来的位置

![](https://bbs.kanxue.com/upload/attach/201811/829513_RQFQ4GPCJ459SY2.jpg)

发现上下两个位置都调用了这个`call`，更确切说明了这是一个关键`call`，要想注册成功，需要这个`call`返回的结果是`0xDB`，我们进入这个`call`里面进行分析。

![](https://bbs.kanxue.com/upload/attach/201811/829513_44T96BDV3GE8SZ2.jpg)

在这个`call`里面我们可以看到有多种返会结果，用方框圈的是我们想要的返回结果，`0Xdb`

![](https://bbs.kanxue.com/upload/attach/201811/829513_VP89HEGSTMGSSBY.jpg)

光标移动到`pop edi`，发现是通过跳转到达的这里，转到跳转处

![](https://bbs.kanxue.com/upload/attach/201811/829513_A2H67KKSMBMFTR2.jpg)

这回判断的是需要让这个`call`的返回值为`0x2D`

进入这个`call`

看这个`call`里面的汇编代码，这应该是关于序列号的算法，由于代码量很大，不详细分析，直接将返回值改为`0x2D`

![](https://bbs.kanxue.com/upload/attach/201811/829513_7QXHA8XMABY826J.jpg)

保存文件，重新启动修改后的`010Editor`，正常进入，点击`help`

![](https://bbs.kanxue.com/upload/attach/201811/829513_CGQMBVEDXFPF73S.jpg)

已经注册成功，破解完成

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
