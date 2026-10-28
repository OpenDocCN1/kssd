
![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_GNDWEG2QETYVX3S.png)

接着运行`777.exe`

发现木马首先会将自身拷贝到`%LOCALAPPDATA%`路径中

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_4QWCJPRNBH2EZBW.png)

然后将自身写入到开机自启动

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_BW93YJC7SHF932G.png)

生成`neuser.ini`文件

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_ZYPTMBV6AC6JDET.png)

生成勒索提示文件：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_YK9RVFQJZBYQAYF.png)

然后就是文件加密，以及最后删除自身，由于该勒索行为实在太多太多，不能进行详细分析，所以只暂时分析了一个大概。

## IDA静态分析

从`start`函数进来：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_ST48D3PGB5TFR73.png)

我们可以看到程序首先会通过`GetModuleFileNameW`获取当前的路径

然后调用了`sub_408B19`函数，参数分别为`MultiByteStr`和`sub_4017E8`

查看了一下`sub_408B19`函数，貌似是解密函数。

通过动态调试发现的确是解密函数，`MultiByteStr`解密出来就是加密后缀名`.Snake4444`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_XJ3FVTM6YF84BRC.png)

然后尝试获取`LOCALAPPDATA`，如果`LOCALAPPDATA`不存在则获取`APPDATA`环境变量，如果获取失败则退出进程，如果成功，则调用`PathAddBackslashW`尝试在路径最后加上反斜杠。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_BKC7C38H26R2PPZ.png)

接着往下看，我们可以知道程序获取了环境变量路径之后，是尝试将文件拷贝过去。

根据`if`判定条件的`&&`优先级可以知道，如果`LOCALAPPDATA`获取成功，后面部分则不会运行，所以直接查看靶机上面的`LOCALAPPDATA`路径，可以看到`777.exe`已经拷贝过来。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_SGAEGK282VGCVV9.png)

根据上面一张代码清单还可以得知，当路径`v0`不为空的时候会执行`sub_409624`，跟进到该函数：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_7F2NKEQG3Q9TXTG.png)

可以看到，`sub_409624`是注册表操作函数，将当前的`exe`写入到开机自启动。

接着往下走

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_ZTYB3EK8QSNSP2A.png)

程序会尝试获取环境变量`public`或环境变量`ALLUSERSPROFILE`存放到`v3`，有了上面那部分的经验，我们直接查看运行后的`public`路径：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_Q344M744AN5Q4NF.png)

可以看到多了两个文件，一个是一串十六进制文本，还有一个是勒索的提示信息，之后我们再分析这个十六进制文件名到底是什么。

路径获取完成之后，会将路径`v6`作为参数，传递到一个循环中

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_3TJ3S2K95V92MTJ.png)

通过调试得到`v6`路径

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_6GQK5MGZM6YJNCD.png)

可以看到，该循环应该会执行10次，但是由于循环次数还由`v8`决定，而`v8`的值由`sub_409B4B`决定，所以跟进到`sub_409B4B`函数。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_DXMFYQHJM9CMRD2.png)

该函数可以得知，程序首先会创建一个文件，路径为传入进来的`v6`，`if`分支是`v4`获取到`GetLastError`的时候，所以我们直接看`else`分支。

在`else`分支中，可以看到最后是一个关键的`wireFile`。而前面的函数通过静态分析暂时还没有得到有用的信息，于是使用`x64dbg`调试。

成功创建文件，文件名是黑客的公钥信息。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_XWVFBYQ8QPD2HGR.png)

调用`sub_40A116`时文件参数如下：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_HXMFZCGXSKNYJSX.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_Z9U7VDPQ9VK8UVN.png)

这里还有一个关键参数，`nullsub_1`，查看该参数对应的内存地址：`401148`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_QYVKXCET7W72URP.png)

发现暂时不知道是啥玩意儿，查看一下`nullsub_1`的调用

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_G2NSC24TWWDKVZP.png)

看地址应该是第二条，跳过去：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_AVVK8HNSZ4SQN6J.png)

是在函数最开的地方，我当时没有分析这两部分，现在回过头去看发现是计算用户ID的，所以这个`nullsub_1`存储的是用户ID信息。

所以这里是将用户ID、`public`路径名、加密后缀、勒索提示文件名作为参数传进来了

跟进到`40A116`函数，根据`rsa_genkey`关键字可以大概得知，该函数是用于生成RSA秘钥对的。

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_BWHUPNUWM273X5U.png)

可以看到，该函数最后返回的是`V9`，而`v9`的值是由`sub_409FDE`函数决定的，动态调试得到参数就是用户ID

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_D48SREJN3RJG7XN.png)

跟进到该函数，发现是`rsa`加密：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_VPBSZYUXJMHE47Q.png)

经过分析，这部分（`sub_409B4B`）的功能是：在`public`路径下写入文件，文件名为黑客公钥，写入内容为经过RSA加密的用户ID。

外层`wireFile`写入的时候，参数2的`ebp`指向的就是经过加密的用户ID的内存

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_2K5YB2EZAVQ582T.png)

`CloseHandle`后查看文件，发现有两行内容

第一行是内层函数里面写入的随机生成RSA公钥信息，第二行是刚才写入的加密后的用户ID

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_35VFTPBT44PPW3V.png)

接着往下走，跟进到这个`call`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_NRT38BSVAHRFAC7.png)

看到遍历磁盘操作

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_QMGAMGYXHU3UCBY.png)

然后根据遍历的结果创建线程：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_YMMKE4GVJQECKQK.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_6TBBHKP92QV5ND9.png)

这里面是一个`dowhile`大循环，循环条件是`FindNextFile`，很明显这里是遍历所有文件加密了，根据分支结构来看，加密函数应该是`sub_408D88`，进入到该函数，函数非常长，在中间的位置找到了之前对之前RSA加密的调用：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_VAUSC4X3ND9H8RB.png)

接下来动态调试验证一下：

直接在创建线程的地方下断点，`F9`过来：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_Z2U73UD6D7GMQUZ.png)

然后直接在我们之前看好的RSA加密函数处下断点，然后`F9`运行，跑过来

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_VUVX9YHPJASB4GQ.png)

第二次断

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_M96DDM9E2ZDAH58.png)

最开始是加密桌面的`link`

先看一下原始桌面：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_8X2MT4CQ9ZP58JV.png)

加密后

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_YD7WMQUHGRXHCD6.png)

多断几次，我的`7z`安装包也被加密了

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_4KQA752FQEJ73J2.png)

现在已经摸清楚循环加密的流程了，接着往后面走，返回到最外层的函数调用，也就是`start`进来的`sub_409C6B`，现在已经分析完了`sub_4099A3`的功能，接着看`sub_409449`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_4CND29TQU9XT2U8.png)

`40449`这个函数功能不复杂

看样子就是创建并执行了一个`bat`文件，动态调试去拿到这个`bat`文件

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_VMBTYH9CC6UJAFE.png)

直接在`sub_409449`入口处下断点，等了一段时间断下来的时候已经加密完成了

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_7EU6JSZN53WH64B.png)

创建路径：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_96KS6XSRYHAJR9X.png)

创建成功，还未写入数据：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_SKTMTX4WXEAP9ET.png)

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_E49YWPC3ERUM8CS.png)

这里因为`sub_409305`里面有`CreateProcess`，所以猜测`sub_409305`是执行`bat`，`sub_408B19`是填充`bat`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_3YDQFPZYRHYTDA5.png)

这里填充了一部分，还不能打开，显示正在占用，估计还没处理完，接着往下走，该函数（`409449`）执行完，程序最后还会`jmp`到`409509`，跳转过去：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_JRNE3SZYD3P3UVP.png)

`409509`

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_B5F5EV9WW6F9S3M.png)

可以看到`409509`的确还在对`bat`进行处理

然后`shellExecuuteExw`之后，删除了勒索文件：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_X4Z8VAKF72UHSBV.png)

最后进程结束，`bat`可以打开了：

![图片描述](https://bbs.kanxue.com/upload/attach/201911/757351_E3CZ63QB9TY39UQ.png)

可以看出来，这个`bat`的功能主要是删除卷影副本，清除远程登入日志。

### 0x03 分析总结

木马加密流程：

1. 拷贝自身到`LOCALAPPDATA`路径

2. 将自身写入到开机自启动

3. 解密勒索后缀、获取用户ID，RSA加密之后存放到`public`，文件名为黑客的公钥信息

4. 遍历用户磁盘

5. 对除了自身以外的所有文件进行加密并生成勒索提示文件

6. 生成`bat`文件，功能为删除卷影备份，删除登录信息

7. 删除原始木马

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
