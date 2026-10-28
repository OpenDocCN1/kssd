
# [原创] 萌新对 Wanna Cry 的分析（初次尝试请多指教...）

## 0x00 前言

前辈们你们好我是萌新(•ᴗ•)

第一次尝试病毒分析，想挑一个资料多的来写（免得我找不着北...），于是决定分析之前非常出名的WannaCry

（过程中几次卡壳都从前辈们的报告里得到了思路，感谢每一位前辈！）

文章中问题肯定不少，就麻烦各位多多指正帮助了！

这个样本是我从B站一个测试病毒的视频里给的网址下的（叫1500个病毒啥的），拖进IDA之后直接到了`WinMain`，因为之前已经看过各大厂商写的分析报告，大概观察了一下于是得出结论：我的样本病毒是主程序生成的负责进行加密文件勒索用户的模块（主程序负责释放勒索模块和传播），不过我现在还在学习阶段，就先只研究一下病毒的恶意行为也是极好的 ⊙﹏⊙‖ ...

## 0x01 基本信息

文件名称：`wcry.exe`

MD5：`84C82835a5d21bbcf75a61706d8ab549`

文件类型：EXE

出品公司：MicrosoftCorporation

壳或编译器信息：MicrosoftVisualC++6.0

## 0x02 行为分析

### 一：勒索模块主体分析

首先样本向注册表写入了两个键值：

`RegCreateKeyW`

![](https://bbs.kanxue.com/upload/attach/201806/792131_H2Q58BCQ75PG5A8.jpg)

![](https://bbs.kanxue.com/upload/attach/201806/792131_WBX539ZRJNPP34Z.jpg)

然后使用内置的解压软件，以`WNcry@2ol7`为解压密码，释放了`b`、`c`、`r`、`s`、`t.wnry`文件，`u.wnry`、`taskdl`/`se.exe`是PE32文件，一个多国语言支持包...

果然 “制作恶意程序的人往往也是出色的开发者” 一点儿没错...

![](https://bbs.kanxue.com/upload/attach/201806/792131_BKMKHRUU5S3ZURM.jpg)

解压缩资源文件的代码，`Str`是压缩密码。

![](https://bbs.kanxue.com/upload/attach/201806/792131_HTGZRYBFDZW3RXZ.jpg)

上面几个函数里面实在是复杂，看的眼睛都瞎了...大体上都是循环解压的函数。

释放的文件的字节码：

`b.wnry`字节码：查资料之后得知是BMP文件。

![](https://bbs.kanxue.com/upload/attach/201806/792131_3RUTSQX5CJFMTD6.jpg)

`c.wnry`字节码：查资料之后得知上面的比特币钱包地址，下面的像是木马的下载地址

![](https://bbs.kanxue.com/upload/attach/201806/792131_YDHFPYTQWHAUYE7.jpg)

`r.wnry`字节码：这个就很明显了，是勒索窗口里显示的文字。

![](https://bbs.kanxue.com/upload/attach/201806/792131_JMZNB6PRGDXGQEM.jpg)

`s.wnry`字节码：查资料后得知`PK`是zip文件头，解压之后是`TaskData`（这么乱果然是代码...ヾ(￣▽￣)~）

![](https://bbs.kanxue.com/upload/attach/201806/792131_TX2JKW7FNHA4FK2.jpg)

`t.wnry`字节码：看不出来是什么，这么乱应该也是代码吧

![](https://bbs.kanxue.com/upload/attach/201806/792131_HGYGBWQ8U2AXP9G.jpg)

接下来到这里：三个字符串是比特币钱包。但是之后这里我没看懂，从`c.wnry`里读取了`30C`字节数据，然后好像也没往里写什么呀？

就又写回去了，函数的第二参数为`1`是读文件，`0`是写文件

![](https://bbs.kanxue.com/upload/attach/201806/792131_V4VQJQFUSSYWMHW.jpg)

把程序所在目录的文件属性设为了隐藏。但是我没找到对应代码......

接下来：无窗口创建进程，命令行参数：`attrib +h .`

无窗口创建进程，命令行参数：`icacls . /grant Everyone:F /T /C /Q`

![](https://bbs.kanxue.com/upload/attach/201806/792131_TMM4S3G7BTD6HC2.jpg)

然后获得要用的API

![](https://bbs.kanxue.com/upload/attach/201806/792131_WESHFQ6K39W4F93.jpg)

这里是导入一个密钥，程序里的加密操作使用`Crypt`系列函数做的。

（密钥相关的知识我现在力所不能及...等以后有时间了再好好研究一下吧）

![](https://bbs.kanxue.com/upload/attach/201806/792131_DYXQBJ8JNBHAW79.jpg)

然后到了主体中最重要的部分：动态加载`t.wnry`作为dll并调用其中的`TaskStart`函数 （注：这是整理的时候加上的）

可以看到打开`t.wnry`，`ReadFile`。

![](https://bbs.kanxue.com/upload/attach/201806/792131_GBFHCDWSTTTA8WR.jpg)![](https://bbs.kanxue.com/upload/attach/201806/792131_KU2QEAFGGMM8W75.jpg)

现在我可以确定文件里的东西很重要...应该就是代码

![](https://bbs.kanxue.com/upload/attach/201806/792131_Q7UABW5UCFD2R69.jpg)

下面又到了一处：解密了`t.wnry`之后把数据映射到了内存中，可以看到PE头。

因此得出结论：`t.wnry`是一个dll文件，并且病毒自己实现了`LoadLibrary`，里面应该才是核心代码。

![](https://bbs.kanxue.com/upload/attach/201806/792131_28JDCAJZTB93RMG.jpg)

节区信息

![](https://bbs.kanxue.com/upload/attach/201806/792131_DEPVQUNXRPB2RJJ.jpg)

终于做完了准备工作，要到关键函数了：

![](https://bbs.kanxue.com/upload/attach/201806/792131_YJ4TD88XZRNR2EQ.jpg)

`FindTaskStart`函数：

这里因为看过报告...所以知道是密钥相关的文件

![](https://bbs.kanxue.com/upload/attach/201806/792131_BH3RYS248BH6C4U.jpg)

释放密钥文件之后函数去`t.wnry`里搜索导出函数，可以看到里面只有一个函数：`TaskStart`。

`t.wnry`的导出表：

![](https://bbs.kanxue.com/upload/attach/201806/792131_UE2N8VKBW2JN7CP.jpg)

到这里勒索模块主体的静态分析就结束了，于是我试着把释放出来的Dll dump下来继续分析，但是这里因为内存的类型是`Private`，`OllyDump`和`LordPE`都无法访问内存ヾ(ﾟ∀ﾟゞ)

![](https://bbs.kanxue.com/upload/attach/201806/792131_YVJ5WCQTFHVVKKP.jpg)

纠结了一阵子之后悟出方法如下：内存窗口打开这块内存，全选二进制复制，`WinHex`新建文件，粘贴进去选择`ASC Hex`

（作为萌新，我猜除了我一定还有其他的人以前没想到这么简单也可以dump的吧…(⊙＿⊙；)…）

于是愉快的继续分析，IDA打开可以看到代码比之前清楚得多，应该是因为准备工作都做好了吧

### 二：主体动态加载的dll的分析

首先创建了一个互斥体

![](https://bbs.kanxue.com/upload/attach/201806/792131_YEWZERJ85WBMK57.jpg)

获得PC的SID（安全标识符，用于标识用户，组和计算机帐户）

![](https://bbs.kanxue.com/upload/attach/201806/792131_4YZVF5HDKHAMDTY.jpg)

这段代码是：查找文件对象的所有者，取得该对象的所有权来更改文件对象（查资料得知的，没太理解）

![](https://bbs.kanxue.com/upload/attach/201806/792131_KNKEBKMAGR68SUZ.jpg)

获得文件属性，但是`GetLastError`说文件不存在，直接返回了

![](https://bbs.kanxue.com/upload/attach/201806/792131_2ZFYTVYE6PSS7BC.jpg)

于是我猜了一下：这个文件应该是交了钱之后装着密钥的，如果存在就会开始解密了而不是加密文件。于是就跳过了一个超复杂的函数

（确认后的确是这样，猜对了就很开心...ヽ(✿ﾟ▽ﾟ)ノ）

![](https://bbs.kanxue.com/upload/attach/201806/792131_V5H7ET9TC62CNWP.jpg)

释放互斥体

![](https://bbs.kanxue.com/upload/attach/201806/792131_4RBYZ5N6WMTVNX7.jpg)

创建了五个线程：

第一个线程：每隔一段时间就获得时间，打开`00000000.res`然后写入数据。

![](https://bbs.kanxue.com/upload/attach/201806/792131_XXJAFBDRCSTFYWP.jpg)![](https://bbs.kanxue.com/upload/attach/201806/792131_PR59YZZGPHCMFRT.jpg)

第二个线程：

每隔一段时间检测解密密钥文件是否存在。存在应该就会解密。

![](https://bbs.kanxue.com/upload/attach/201806/792131_F83JEEEEXESKZS4.jpg)

第三个线程：

隔一段时间检测一次磁盘数量有没有变化，有的话应该会加密里面的文件。

![](https://bbs.kanxue.com/upload/attach/201806/792131_SN75NA365PV3HHQ.jpg)

第四个线程：每隔一段时间就无窗口创建`taskdl.exe`进程（看了前辈们的报告说是清除文件的）

![](https://bbs.kanxue.com/upload/attach/201806/792131_MU86VWB8VDSPXJ5.jpg)![](https://bbs.kanxue.com/upload/attach/201806/792131_MR462T4KXTH6H7Z.jpg)

第五个线程：

每隔一段时间创建`taskse.exe`和`@WanaDecryptor@.exe`进程（勒索窗口）

![](https://bbs.kanxue.com/upload/attach/201806/792131_2VHTR8QK4J6CBX3.jpg)![](https://bbs.kanxue.com/upload/attach/201806/792131_6AVEQ4CNTA4CKPC.jpg)

里面还存有一个注册表键值，但是没有进行操作，我猜测应该是创建的子进程里使用键值写注册表然后实现自启动？

![](https://bbs.kanxue.com/upload/attach/201806/792131_289GKUEC3NPAZ5V.jpg)

五个线程创建完毕之后终于到了最后一个函数

样本里还涉及对密钥进行操作，这类函数我实在是看不懂，贴一张安天的前辈的报告的图吧......

（密码学没好好学，下周还考试呢啥都不会emmm）

![](https://bbs.kanxue.com/upload/attach/201806/792131_34Y563F4GV8UBQE.jpg)

![](https://bbs.kanxue.com/upload/attach/201806/792131_HZ24SJQQF8CPDFN.jpg)

![](https://bbs.kanxue.com/upload/attach/201806/792131_8CRSMSCX94AEB42.jpg)

### 三：TaskStart中的加密文件部分分析

![](https://bbs.kanxue.com/upload/attach/201806/792131_BARCRXX498AYD3N.jpg)

就这个函数...我在IDA里分析完了之前的代码之后，因为它没有参数于是居然完全没有发现它的存在...

一度心态崩溃（脑子里一直循环：卧槽我到底在哪漏了啥？？）

于是给他起个名字叫：明明最终要但居然连个参数都没有的鬼函数...

首先判断`@WanaDecryptor@.exe`，`@WanaDecryptor@.exe.lnk`，`ReadMe`是否存在

![](https://bbs.kanxue.com/upload/attach/201806/792131_PW3UZQ2XWSFPPUA.jpg)

然后开始加密所有用户的桌面和文档目录（可能是为了让所有用户都立即感受到自己被勒索了emmm）

获得桌面的路径

![](https://bbs.kanxue.com/upload/attach/201806/792131_YVAPF6QNGFPX9DV.jpg)

根据路径找到目录

![](https://bbs.kanxue.com/upload/attach/201806/792131_USDP8ZD6ATF8GMX.jpg)

然后创建临时文件并打开，但是什么都没做就删除了？...

![](https://bbs.kanxue.com/upload/attach/201806/792131_EE5N2VS3PFAHK83.jpg)

![](https://bbs.pediy.com/upload/attach/201806/792131_6EKUTRC2XCJQ4CZ.jpg) ![](https://bbs.kanxue.com/upload/attach/201806/792131_PF5YWP7AFTA4ZPH.jpg)

在目录下排除这三个文件

![](https://bbs.kanxue.com/upload/attach/201806/792131_VTK5PMY4B3U9E47.jpg)

然后遍历目录下所有的文件，如果文件格式不是`exe`和`dll`就判断文件格式是不是要加密的

![](https://bbs.kanxue.com/upload/attach/201806/792131_H528USPC36ZPSSE.jpg)

是的话就打开文件，获得文件大小，然后加密

![](https://bbs.kanxue.com/upload/attach/201806/792131_P77NK2PFJ7ZEQZF.jpg)

加密当前用户文档目录下的文件

![](https://bbs.kanxue.com/upload/attach/201806/792131_HRK5KFCASJWA3FK.jpg)

对其他用户的这两个目录都进行加密

![](https://bbs.kanxue.com/upload/attach/201806/792131_5UJ4KKWNGCH2CRR.jpg)

接下来是使用`taskkill.exe`终止下面五个进程

![](https://bbs.kanxue.com/upload/attach/201806/792131_3QSSZ7YMY8JU3UB.jpg)

获得磁盘卷信息，判断磁盘是否有效（？）到磁盘内第一个目录下开始遍历

找到临时文件目录，使用和之前一样的方式遍历加密文件。

![](https://bbs.kanxue.com/upload/attach/201806/792131_PVZC7VAPSGR4UPS.jpg)

完毕后创建勒索窗口进程，使用线程1的线程函数写入`res`文件。

![](https://bbs.kanxue.com/upload/attach/201806/792131_99KD7WAYNC7YMWP.jpg)

创建`cmd`，用命令行启动勒索窗口，会每隔一段时间就把原文件移动到临时目录下。

![](https://bbs.kanxue.com/upload/attach/201806/792131_2ZSZKZM62S57MRJ.jpg)

移动到临时目录下

![](https://bbs.kanxue.com/upload/attach/201806/792131_DPU4AT2EPVKE6FJ.jpg)

每隔一段时间删除临时目录下的文件

![](https://bbs.kanxue.com/upload/attach/201806/792131_4A3CFSK2UVMAE5N.jpg)

文件遍历加密部分的详细过程我现在也有点力所不能及，希望等修炼一段时间之后再重新尝试一下吧！

## 0x03 总结

作为萌新我一定是脑子坏了才会第一次分析就来看WannaCry (┬＿┬) 感觉样本最核心的部分应该是对密钥进行的操作和遍历磁盘文件时的详细操作，但是目前我能力还有限，暂时告一段落（核心部分之外的代码都已经让我分析的很吃力了…）

样本的行为流程总结如下：主程序写注册表之后使用内置压缩软件解压资源文件，动态加载资源文件中的模块，然后转到模块导出函数`TaskStart`函数里

`TaskStart`函数先为之后的文件操作获得了一些安全权限，然后检测是否存在解密密钥，存在应该就会进行解密操作。接下来创建五个线程分别执行不同的操作。（上面分析了）
